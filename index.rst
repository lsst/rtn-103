#################################################################################
Procedure for creating a butler repository at FrDF for ComCam multisite campaigns
#################################################################################

.. abstract::

   In this note we document the required input datasets and the procedure we followed at the Rubin French Data Facility (FrDF) for creating and populating a butler repository for the needs of ComCam multisite campaigns.

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

