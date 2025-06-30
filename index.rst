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

Skympa was taken from `/pbs/throng/lsst/users/byanny/skymaps/lsst_cells_v1.skymap.config`.
More details on the skymap can be found in the issue `DM-46717 <https://rubinobs.atlassian.net/browse/DM-46717>`__

.. _ingest-raw-exposures:

Ingest raw exposures
--------------------

We ingest the raw exposures using:

.. prompt:: bash

    butler ingest-raws --fail-fast --transfer direct $REPO $DATA/raw/LSSTComCam


