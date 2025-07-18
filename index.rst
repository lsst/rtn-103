#################################################################################
Procedure for creating a butler repository at FrDF for ComCam multisite campaigns
#################################################################################

.. abstract::

   In this note we document the required input datasets and the procedure we followed at the Rubin French Data Facility (FrDF) for creating and populating a butler repository for the needs of ComCam multisite campaigns. This note is based on `DM-48746 <https://rubinobs.atlassian.net/browse/DM-48746>`__.

Introduction
============

Input Datasets
==============

.. _import-sky-map:

SkyMap
------

Skymap used was `/pbs/throng/lsst/users/byanny/skymaps/lsst_cells_v1.skymap.config`.
More details on the skymap can be found in the issue `DM-46717 <https://rubinobs.atlassian.net/browse/DM-46717>`__.

.. _import-raw-exposures:

Raw images
----------

For the ComCam multisite butler repository we use the 16000 exposures raw images produced during the LSSTComCam campaign (about 16000 exposures).
Raw exposures are registered in Rucio in the `raw` scope, in a dataset named `Dataset/LSSTComCam/raw/<date>`, where `<date>` is the date where the exposure has been acquired.
They are automatically replicated at FrDF and are located in `davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument/raw/LSSTComCam/`.
To facilitate ingestion, a metadata file `_index.json` has been generated for each exposure using the `astrometadata` package, and uploaded in the same directory as the exposure files.

.. _import-calibration-data:

Calibration data
----------------

LSSTComCam calibration data are located at USDF in the `/repo/main` butler repository. The list of calibrations to ingest is the following:

* `DM-48520 <https://rubinobs.atlassian.net/browse/DM-48520>`__
* `DM-47365 <https://rubinobs.atlassian.net/browse/DM-47365>`__
* `DM-47741 <https://rubinobs.atlassian.net/browse/DM-47741>`__
* `DM-47547 <https://rubinobs.atlassian.net/browse/DM-47547>`__
* `DM-47499 <https://rubinobs.atlassian.net/browse/DM-47499>`__
* `DM-47447 <https://rubinobs.atlassian.net/browse/DM-47447>`__
* `DM-47197 <https://rubinobs.atlassian.net/browse/DM-47197>`__
* `DM-46360 <https://rubinobs.atlassian.net/browse/DM-46360>`__
* `DM-47498 <https://rubinobs.atlassian.net/browse/DM-47498>`__

Each item is a ticket (`$TICKET`) that corresponds to a calibration collection (`COLLECTION=$INSTRUMENT/calib/$TICKET`), and requires an `export.yaml` to be ingested. These files can be found at USDF in the directory `/sdf/data/rubin/shared/calibration_archive`:

.. prompt:: bash

    cd /sdf/data/rubin/shared/calibration_archive
    rg -l $TICKET TAXICAB-* | grep export.yaml |& head -1

For instance:

.. prompt:: bash

    rg -l DM-48520 TAXICAB-* | grep export.yaml |& head -1
    ./TAXICAB-23/LSSTComCam.calibs.20250213a/export.yaml

These files can be manually retrieved through ssh, although they will eventually be managed by Rucio.
Each collection is registered in Rucio in the `ancillary` scope using the following command:

.. prompt:: bash

    rucio-register data-products \
      -s 10 \
      -C /sdf/data/rubin/shared/calibration_archive/rucio/main-calib-config.yaml \
      -r /repo/main \
      -t $dstype \
      -c $COLLECTION \
      -d $DATASET

    rucio did update --close ancillary:$DATASET

where `$dstype` is the dataset type (`dstyps` in our case), `$COLLECTION` is the collection name as defined above, and `$DATASET` is the dataset name: `Dataset/LSSTComCam/$dstype/$TICKET`.

The registered data products can then be replicated at FrDF:

.. prompt:: bash

    rucio rule add --rses 'SLAC_BUTLER_DISK|IN2P3_RAW_DISK' --copies 2 ancillary:$DATASET

or

.. prompt:: bash

    rucio rule add --rses 'IN2P3_RAW_DISK' --copies 1 ancillary:$DATASET

They are located in `davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument/ancillary/LSSTComCam/calib/`.

.. _import-reference-catalog:

Reference catalogs
------------------

Two versions of "The Monster" catalog are used (see `DM-46370 <https://rubinobs.atlassian.net/browse/DM-46370>`__ and `DM-49042 <https://rubinobs.atlassian.net/browse/DM-49042>`__).
Both are located at USDF in `/sdf/data/rubin/shared/refcats`, and registered in Rucio, in datasets 

.. code-block:: yaml

    Dataset/refcats/the_monster_20240219_1
    Dataset/refcats/the_monster_20240219_2

and `Dataset/refcats/the_monster_20240904`. They are replicated at FRDF with:

.. prompt:: bash

    rucio rule add --rses 'IN2P3_RAW_DISK' --copies 1 raw:Dataset/refcats/the_monster_20240219_1

and are located in `davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument/raw/refcats/`.

.. _import-pretrained-models:

Pretrained-models catalog
-------------------------

Pretrained-models catalog is registered in Rucio in the `ancillary`, in dataset `Dataset/LSSTComCam/dstyps/pretrained-models`.
It is replicated at FrDF with:

.. prompt:: bash

    rucio rule add --rses IN2P3_RAW_DISK --copies 1 ancillary:Dataset/LSSTComCam/dstyps/pretrained-models

and is located in `davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument/ancillary/pretrained_models/`.

.. _import-fgcm:

FGCM calibration
----------------

FGCM lookup table (see `DM-48089 <https://rubinobs.atlassian.net/browse/DM-48089>`__) is registered in Rucio in the `ancillary`, in dataset `Dataset/LSSTComCam/dstyps/fgcmLookUpTable`.
It is replicated at FrDF with:

.. prompt:: bash

    rucio rule add --rses IN2P3_RAW_DISK --copies 1 ancillary:Dataset/LSSTComCam/dstyps/fgcmLookUpTable

and is located in `davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument/ancillary/LSSTComCam/calib/fgcmcal/`.

.. _import-sso:

Solar System Objects catalog
----------------------------

Solar System Objects catalog (see `DM-49977 <https://rubinobs.atlassian.net/browse/DM-49977>`__) is registered in Rucio in the `ancillary`, in dataset `Dataset/LSSTComCam/dstyps/DM-49977`.
It is replicated at FrDF with:

.. prompt:: bash

    rucio rule add --rses IN2P3_RAW_DISK --copies 1 ancillary:Dataset/LSSTComCam/dstyps/DM-49977

and is located in `davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument/ancillary/u/jkurla/dp1_ephem_2/`.

Creating and populating the repository
======================================

We present here the procedure we used for creating and populating the repository.

The location of the repository is referred using the environment variable ``$REPO``:

.. prompt:: bash

    export REPO='davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/butler/ccms1'

The location of data to be ingested is defined using the environment variable ``$DATA``:

.. prompt:: bash

    export DATA='davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/instrument'

.. _create-empty-repository:

Create an empty repository
--------------------------

We use the seed configuration file ``butler-seed_ccms1.yaml`` shown below to create a butler repository composed of a PostgreSQL registry database and a WebDAV datastore (the default):

.. code-block:: bash

    $ cat butler-seed_ccms1.yaml
    datastore:
      name: "ccms1"
      root: "davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/butler/ccms1"
    registry:
      db: postgresql://ccpglsstprod.in2p3.fr:6552/lsstprod
      namespace: ccms1

To create the repository at location ``$REPO`` we use the command:

.. prompt:: bash

    butler create --seed-config butler-seed_ccms1.yaml --override $REPO

.. _register-instrument:

Register instrument
-------------------

To register the instrument for this repository we use the command below:

.. prompt:: bash

    butler register-instrument $REPO lsst.obs.lsst.LsstComCam

.. _register-sky-map:

Register SkyMap
----------------

To register the skymap configuration we use the command below:

.. prompt:: bash

    butler register-skymap --config-file lsst_cells_v1.skymap.config $REPO

.. _ingest-raw-exposures:

Ingest raw exposures
--------------------

We ingest the raw exposures using:

.. prompt:: bash

    butler ingest-raws --fail-fast --transfer direct $REPO $DATA/raw/LSSTComCam

Note that parallel ingestion was performed to speedup the process.
One can then check that all visits / detectors have been ingested:

.. prompt:: bash

    butler query-datasets $REPO raw --collections LSSTComCam/raw/all --limit 0 | wc -l
    148849

Since there are 9 detectors in LSSTComCam, this corresponds to the approximate number of 16000 exposures in the LSSTComCam campaign.

.. _define-visits:

Define visits
-------------

To define visits from the exposures previously ingested into the repository we use the command below:

.. prompt:: bash
    
    butler define-visits $REPO LSSTComCam --collections LSSTComCam/raw/all

.. _add-instrument-calibrations:

Add instrument's curated calibrations
-------------------------------------

To ingest the known calibration data for LSSTComCam (see `DM-48650 <https://rubinobs.atlassian.net/browse/DM-48650>`__) we use the command below:

.. prompt:: bash

    butler write-curated-calibrations $REPO lsst.obs.lsst.LsstComCam --label DM-48650

.. _ingest-calibration-data:

Ingest calibration data
-----------------------

To ingest calibration data we use the command below, for each collection:

.. prompt:: bash

    butler import $REPO $DATA/ancillary --export-file export.yaml -t direct

Once all calibrations have been ingested, a global calibration collection is defined:

.. prompt:: bash

    butler collection-chain $REPO LSSTComCam/calib LSSTComCam/calib/DM-48955,LSSTComCam/calib/DM-48520,LSSTComCam/calib/DM-47365,LSSTComCam/calib/DM-47741,LSSTComCam/calib/DM-47547,LSSTComCam/calib/DM-47499,LSSTComCam/calib/DM-47447,LSSTComCam/calib/DM-47197,LSSTComCam/calib/DM-46360,LSSTComCam/calib/DM-47498,LSSTComCam/calib/DM-48650,LSSTComCam/calib/DM-48650/unbounded


.. _ingest-reference-catalog:

Ingest reference catalogs
-------------------------

For the first version of "The Monster" catalog, the corresponding dataset type is registered with:

.. prompt:: bash

    butler register-dataset-type $REPO the_monster_20240904 SimpleCatalog htm7

Then the ingestion is done:

.. prompt:: bash

    butler ingest-files $REPO the_monster_20240904 refcats/DM-46370/the_monster_20240904 --prefix $DATA/raw/refcats/the_monster_20240904/ -t direct the_monster_20240904.ecsv

where the file `the_monster_20240904.ecsv` has been provided by B. Yanny. Similarly, for the second version:

.. prompt:: bash

    butler register-dataset-type $REPO the_monster_20250219 SimpleCatalog htm7
    butler ingest-files $REPO the_monster_20250219 refcats/DM-49042/the_monster_20250219 --prefix $DATA/raw/refcats/the_monster_20250219/ -t direct the_monster_20250219.ecsv

A chained collection is then created:

.. prompt:: bash

    butler collection-chain $REPO refcats refcats/DM-46370/the_monster_20240904,refcats/DM-49042/the_monster_20250219

.. _ingest-pretrained-models:

Ingest Pretrained-models catalog
--------------------------------

Pretrained-models catalog is ingested with:

.. prompt:: bash

    butler import $REPO --export-file pretrained-models-export.yaml -t direct $DATA/ancillary/

where `pretrained-models-export.yaml` has the following content:

.. code-block:: yaml

    description: Butler Data Repository Export
    version: 1.0.2
    universe_version: 7
    universe_namespace: daf_butler
    data:
    - type: collection
      collection_type: RUN
      name: pretrained_models/tac_cnn_comcam_2025-02-18
      host: null
      timespan_begin: null
      timespan_end: null
    - type: dataset_type
      name: pretrainedModelPackage
      dimensions: []
      storage_class: NNModelPackagePayload
      is_calibration: false
    - type: dataset
      dataset_type: pretrainedModelPackage
      run: pretrained_models/tac_cnn_comcam_2025-02-18
      records:
      - dataset_id:
        - !uuid 'a83d850a-0094-417c-ac9c-64d0f7b98048'
        data_id:
        - {}
        path: pretrained_models/tac_cnn_comcam_2025-02-18/pretrainedModelPackage/pretrainedModelPackage_pretrained_models_tac_cnn_comcam_2025-02-18.zip
        formatter: lsst.meas.transiNet.modelPackages.formatters.NNModelPackageFormatter
    
A chained collection is then created:

.. prompt:: bash	

    butler collection-chain $REPO pretrained_models pretrained_models/tac_cnn_comcam_2025-02-18

.. _ingest-fgcm:

Ingest FGCM calibration
-----------------------

FGCM calibration is ingested with:

.. prompt:: bash

    butler import $REPO --export-file DM-48089-fgcmLookupTable-export.yaml -t direct $DATA/ancillary/

where `DM-48089-fgcmLookupTable-export.yaml` has the following content:

.. code-block:: yaml

    description: Butler Data Repository Export
    version: 1.0.2
    universe_version: 7
    universe_namespace: daf_butler
    data:
    - type: dimension
      element: instrument
      records:
      - name: LSSTComCam
        visit_max: 7050123199999
        visit_system: 2
        exposure_max: 7050123199999
        detector_max: 1000
        class_name: lsst.obs.lsst.LsstComCam
    - type: collection
      collection_type: RUN
      name: LSSTComCam/calib/fgcmcal/DM-48089
      host: null
      timespan_begin: null
      timespan_end: null
    - type: dataset_type
      name: fgcmLookUpTable
      dimensions:
      - instrument
      storage_class: Catalog
      is_calibration: false
    - type: dataset
      dataset_type: fgcmLookUpTable
      run: LSSTComCam/calib/fgcmcal/DM-48089
      records:
      - dataset_id:
        - !uuid 'bb573ca3-6159-45d9-88e3-866e01da4882'
        data_id:
        - instrument: LSSTComCam
        path: LSSTComCam/calib/fgcmcal/DM-48089/fgcmLookUpTable/fgcmLookUpTable_LSSTComCam_LSSTComCam_calib_fgcmcal_DM-48089.fits
        formatter: lsst.obs.base.formatters.fitsGeneric.FitsGenericFormatter

A chained collection is then created:

.. prompt:: bash

    butler collection-chain $REPO LSSTComCam/calib/fgcmcal LSSTComCam/calib/fgcmcal/DM-48089  

.. _ingest-sso:

Ingest Solar System Objects catalog
-----------------------------------

Solar System Objects catalog (see `DM-49977 <https://rubinobs.atlassian.net/browse/DM-49977>`__) is ingested with:

.. prompt:: bash

    butler import $REPO --export-file export.yaml -t direct $DATA/ancillary/

where the file `export.yaml` has been provided by B. Yanny. A TAGGED collection is then created, including all datasets:

.. code-block:: python

    butler = Butler('$REPO',writeable=True)
    butler.registry.registerCollection("LSSTComCam/calib/DM-49977/DP1.0/preloaded_SsObjects.20250409", CollectionType.TAGGED)
    dataset_refs = butler.registry.queryDatasets("preloaded_DRP_SsObjects",collections="u/jkurla/dp1_ephem_2*",instrument="LSSTComCam")
    butler.registry.associate("LSSTComCam/calib/DM-49977/DP1.0/preloaded_SsObjects.20250409", dataset_refs)

.. _create-collection:

Create global collection
------------------------

Within the 16000 exposures ingested, about 2000 are Science exposures (each with 9 detectors):

.. prompt:: bash

    butler query-datasets $REPO raw --collections LSSTComCam/raw/all --where "exposure.observation_type='science'" --limit 0 |wc -l
    19205

From these ones, 1792 exposures have been selected to be processed (see `DM-49594 <https://rubinobs.atlassian.net/browse/DM-49594>`__). We define therefore a collection containing thse 1792 selected LSSTComCam exposures:

.. prompt:: bash

    python /pbs/throng/lsst/users/byanny/butler_associate_visits.py $REPO /pbs/throng/lsst/users/byanny/dp1_good_visits.txt LSSTComCam/raw/DP1-RC3/DM-49594 LSSTComCam/raw/all LSSTComCam 2000

Finally, we define a collection containg all input collections previously defined:

.. prompt:: bash

    butler collection-chain $REPO LSSTComCam/DP1/defaults LSSTComCam/raw/DP1-RC3/DM-49594,LSSTComCam/calib,refcats,skymaps,pretrained_models,LSSTComCam/calib/fgcmcal,LSSTComCam/calib/DM-49977/DP1.0/preloaded_SsObjects.20250409


