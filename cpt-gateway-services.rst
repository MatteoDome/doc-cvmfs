.. _cpt_gateway_services:

CernVM-FS Gateway Services and Release Managers
===============================================

This page details the installation and configuration of a repository setup
involving a gateway machine and separate release manager machines.

Glossary
^^^^^^^^

Gateway (GW)
  The machine running an instance of the `CVMFS gateway
  services <https://github.com/cvmfs/cmvfs_services.git>`_ and which
  has access to the authoritative storage of the managed
  repositories. There is a current limitation in the implementation of
  the gateway services that the authoritative storage must be a
  locally mounted partition on GW, however S3 back-ends should be
  supported in the future. The purpose of the GW is to manage access
  to a set of repositories by assigning exclusive leases to specific
  repository sub-paths to different release manager (RM) machines. The
  RM can publish files to the sub-path for which it currently holds a
  lease but sending object packs to the GW. Having received the
  published payload from the RM, the final task of the GW in the
  publication lifecycle is to rebuild the catalogs and repository
  manifests for the modified repositories.

Release manager (RM)
  A machine running the CVMFS server tools which can request leases
  from a GW and publish change to different repositories where it
  currently holds a valid lease.

  It is on the RM that the computationally heavy tasks of compressing
  and hashing the files which are to be added or modified as part of a
  publish operation. The processed files are finally packed together
  and send to the GW to be inserted into the repository and made
  available to clients.

Gateway Services Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Gateway services installation instructions will be added after the
deployment strategy for the gateway services application is clarified.

Release Manager Configuration
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This section describes the steps needed to set up a release manager for a specific CVMFS repository. The precondition is a working gateway machine where the repository has been created as a Stratum 0.

1. Create another instance of the repository on the release manager using ``cvmfs_server mkfs``.

2. Retrieve the public key of the repository and the GW API key from the GW and place them into ``/etc/cvmfs/keys``.

3. In the ``client.conf`` file for the repository, modify the ``CVMFS_SERVER_URL`` variable to point to the GW URL.

4. In the ``server.conf`` file for the repository, modify the ``CVMFS_STRATUM0`` variable to point to the GW URL. Set ``CVMFS_UPSTREAM_STORAGE`` to ``gw,<UNUSED_PATH>, <GW_API_URL>``. Set ``CVMFS_GATEWAY_KEYS`` to point to the gateway key file.

Example:
""""""""

* The gateway machine is ``gateway.cern.ch``.
* The release manager is ``rm.cern.ch``.
* The new repository's fully qualified name is ``test.cern.ch``.
* The repository's public key is ``test.cern.ch.pub``.
* The GW API key is ``gateway_key``.
* The GW services application is running on port 8080 at the URL ``http:://gateway.cern.ch:8080/api/v1``.

Steps:

1. ``cvmfs_server mkfs test.cern.ch``
2. Copy ``test.cern.ch.pub`` and ``gateway_key`` to ``/etc/cvmfs/keys/``.
3. In ``client.conf``, set ``CVMFS_SERVER_URL`` to ``http://gateway.cern.ch/cvmfs/test.cern.ch``.
4. In ``server.conf``, set ``CVMFS_STRATUM0`` to ``http://gateway.cern.ch/cvmfs/test.cern.ch``, ``CVMFS_UPSTREAM_STORAGE`` to ``gw,/srv/cvmfs/test.cern.ch/data/txt,http:://gateway.cern.ch:8080/api/v1`` and ``CVMFS_GATEWAY_KEYS`` to ``/etc/cvmfs/keys/gateway_key``

At this point, from the RM we can publish to the repository: ::

  $ cvmfs_server transaction -e test.cern.ch

  ... make changes to the repository ...

  $ cvmfs_server publish -e test.cern.ch
