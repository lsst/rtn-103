#################################################################################
Procedure for creating a butler repository at FrDF for ComCam multisite campaigns
#################################################################################

.. abstract::

   In this note we document the required input datasets and the procedure we followed at the Rubin French Data Facility (FrDF) for creating and populating a butler repository for the needs of ComCam multisite campaigns. Based on `DM-48746 <https://rubinobs.atlassian.net/browse/DM-48746>`__

Introduction
============

Input Datasets
==============

Raw images
----------

Creating and populating the repository
======================================

Wwe present here the procedure we used for creating and populating the repository using the `LSST Science Pipelines <https://pipelines.lsst.io>`__ 

The location of the repository is referred using the environment variable ``$REPO``:

.. prompt:: bash

    export REPO='davs://ccdavrubinint.in2p3.fr:2880/pnfs/in2p3.fr/lsst/butler/ccms1'

The location of data to be ingested are defined using the  environment variable ``$DATA``:

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

    butler --long-log create --seed-config butler-seed_ccms1.yaml --override $REPO

.. _register-instrument:

Register instrument
-------------------

To register the instrument for this repository we use the command below:

.. prompt:: bash

    butler --long-log register-instrument $REPO lsst.obs.lsst.LsstComCam

.. _register-sky-map:

Register SkyMap
----------------

To register the skymap configuration we use the command below:

.. prompt:: bash

    butler --long-log register-skymap --config-file lsst_cells_v1.skymap.config $REPO

Skymap was taken from `/pbs/throng/lsst/users/byanny/skymaps/lsst_cells_v1.skymap.config`.
More details on the skymap can be found in the issue `DM-46717 <https://rubinobs.atlassian.net/browse/DM-46717>`__

.. _ingest-raw-exposures:

Ingest raw exposures
--------------------

We ingest the raw exposures using:

.. prompt:: bash

    butler ingest-raws --fail-fast --transfer direct $REPO $DATA/raw/LSSTComCam

Note that parallel ingestion was performed to speedup the process.
One can then check that all visits / detectors have been ingested:

.. prompt:: bash

    butler query-datasets $REPO raw --collections  LSSTComCam/raw/all --limit 200000 |wc -l
    148849

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

.. _import-calibration-data:

Ingest calibration data
-----------------------

The list of calibrations to ingest is the following:

* `DM-48520 <https://rubinobs.atlassian.net/browse/DM-48520>`__
* `DM-47365 <https://rubinobs.atlassian.net/browse/DM-47365>`__
* `DM-47741 <https://rubinobs.atlassian.net/browse/DM-47741>`__
* `DM-47547 <https://rubinobs.atlassian.net/browse/DM-47547>`__ 
* `DM-47499 <https://rubinobs.atlassian.net/browse/DM-47499>`__
* `DM-47447 <https://rubinobs.atlassian.net/browse/DM-47447>`__
* `DM-47197 <https://rubinobs.atlassian.net/browse/DM-47197>`__
* `DM-46360 <https://rubinobs.atlassian.net/browse/DM-46360>`__
* `DM-47498 <https://rubinobs.atlassian.net/browse/DM-47498>`__
* `DM-48650 <https://rubinobs.atlassian.net/browse/DM-48650>`__

Each item corresponds to a calibration collection at USDF, and requires an `export.yaml` to be ingested. These files can be found at USF in the directory `/sdf/data/rubin/shared/calibration_archive`, for instance:

.. prompt:: bash

    cd /sdf/data/rubin/shared/calibration_archive
    rg -l DM-48520 . | grep export.yaml |& head -1
    ./TAXICAB-23/LSSTComCam.calibs.20250213a/export.yaml

These files can be manually retrieved through ssh, although they will eventually be managed by Rucio.
To ingest calibration data we use the command below, for each collection:

.. prompt:: bash

    butler import $REPO $DATA/ancillary --export-file export.yaml  -t direct

Once all calibrations have been ingested, a global calibration collection is defined:

.. prompt:: bash

    butler collection-chain $REPO LSSTComCam/calib LSSTComCam/calib/DM-48955,LSSTComCam/calib/DM-48520,LSSTComCam/calib/DM-47365,LSSTComCam/calib/DM-47741,LSSTComCam/calib/DM-47547,LSSTComCam/calib/DM-47499,LSSTComCam/calib/DM-47447,LSSTComCam/calib/DM-47197,LSSTComCam/calib/DM-46360,LSSTComCam/calib/DM-47498,LSSTComCam/calib/DM-48650,LSSTComCam/calib/DM-48650/unbounded

