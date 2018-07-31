Installation Guide
==================

This page contains information about how to deploy dojot using Docker compose.
Kubernetes and Google Cloud Platform support is on track to be implemented.

.. contents:: Table of Contents
  :local:


Hardware requirements
---------------------

In order to properly run dojot, the minimum hardware requirements are:

- 4GB of RAM
- 10GB of free disk space
- Network access
- The following ports should be opened:
   - TCP (incoming connections): 1883 (MQTT), 8883 (Secure MQTT if used), 8000
     (web interface access)
   - TCP (outgoing connections): 25 (if send e-mail node is used in a flow)



Docker compose
--------------

This document provides instructions on how to create a trivial deployment
environment on single host for *dojot*, using docker-compose as the processes
orchestration platform.

While very simple, this deployment option is best suited to development and
assessment of the platform and should not be used for production environments.

This guide has been checked on an Ubuntu 16.04 LTS environment.

The following sections describe all Docker compose dependencies.

Docker engine
^^^^^^^^^^^^^

Up to date information and installation procedures for the docker engine can be
found at the project's documentation:

https://docs.docker.com/engine/installation/

.. note::

  An optional step on the installation and configuration process of docker on
  any given machine is the setting of who is eligible for creating/spawning
  docker instances.

  Should the post-installation steps (more specifically the "Manage docker as
  non-root user") have not been run, all docker and docker-compose commands
  should be run by the super user (root), or as sudo.

  https://docs.docker.com/engine/installation/linux/linux-postinstall/

Docker Compose
^^^^^^^^^^^^^^

Up to date information and installation procedures for the docker-compose can
be found at the project's documentation:

https://docs.docker.com/compose/install/


Installation
^^^^^^^^^^^^

To setup the environment, merely clone the deployment repository and run the
commands below.

The docker-compose enabled deployment scripts and configuration repository can
be found at:

https://github.com/dojot/docker-compose

or as git clone command:::

  git clone https://github.com/dojot/docker-compose.git
  # Let's move into the repo - all commands in this page should be executed
  # inside it.
  cd docker-compose

Once the repository is properly cloned, select the version to be used by
checking out the appropriate tag (do notice that the tagname has to be
replaced): ::

  # Must be run from within the deployment repo

  git checkout tag_name -b branch_name

For instance: ::

  git checkout 0.2.0 -b 0.2.0

Or if you're brave enough: ::

  git checkout master

After the repository is cloned, and a release (or branch) has been selected,
there are still a few external modules that must be gathered before using the
platform. These modules can be retrieved by executing the following command: ::

  git submodule update --init --recursive

That done, the environment can be brought up by: ::

  # Must be run from the root of the deployment repo.
  # May need sudo to work: sudo docker-compose up -d
  docker-compose up -d


To check individual container status, docker's commands may be used, for
instance: ::

  # Shows the list of currently running containers, along with individual info
  docker ps

  # Shows the list of all configured containers, along with individual info
  docker ps -a

.. note::

  All docker, docker-compose commands may need sudo to work.

  To allow non-root users to manage docker, please check docker's documentation:

  https://docs.docker.com/engine/installation/linux/linux-postinstall/

Usage
^^^^^

The web interface is available at ``http://localhost:8000``. The user is
``admin`` and the password is ``admin``. You also can interact with platform
using the :ref:`Components and APIs`.

Read the :doc:`tutorials/using-api-interface` and
:doc:`tutorials/using-web-interface` for more information about how to
interact with the platform.

Kubernetes
----------

This section provides instructions on how to create a simple dojot deployment
environment on a multi-node environment, using Kubernetes as the orchestration
platform.

This deployment option as presented in this document is best suited for testing
and platform assessment. With appropriate changes, this option can be also be
used in production environments.

This guide has been checked on a Kubernetes cluster with Ceph as the underlying
storage infrastructure and it has also been tested on a Kubernetes cluster over
the Google Cloud Platform

The following sections describe all Kubernetes dependencies.

Kubernetes Cluster
^^^^^^^^^^^^^^^^^^

For this guide it is advised that you already have a working cluster.

If you desire to prepare a Kubernetes cluster from scratch, up to date
information and installation procedures can be found at `Kubernetes setup
documentation`_.

Persistent Storage
^^^^^^^^^^^^^^^^^^

To make sure that all the data from the containers running databases is
persisted when containers fail or are moved to different nodes of the
Kubernetes environment it is necessary to attach persistent storage to the
database pods.

Kubernetes requires that an infrastructure for persistent storage already
exists on the cluster. As an example for how to configure your persistent
storage we provide files for two different kind of deployments, the first is
for a local deployment where a Ceph Cluster is used as storage backend, more
information on Ceph may be found at: http://ceph.com/. The second example is
based on a Google Cloud deployment and use the existing persistent storage
services that are provided by Google Cloud. If you're deploying dojot using
Kubernetes to a different cloud provider, some adjustments to fit the different
deployments might be necessary.

Information about the currently supported persistent storage for Kubernetes can
be found at `persistent-volumes page`_.

Kubernetes Client
^^^^^^^^^^^^^^^^^

To install the Kubernetes client on your machine before proceeding with this
guide, follow the proper instructions as presented on the `Kubernetes
documentation`_.

Also, verify that your client is capable of connecting to the cluster.

For providing access for a local cluster, follow the documentation below:

https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/

If the Kubernetes cluster is running on a specific cloud platform like Google
Cloud, follow the steps as presented by your cloud provider.

Deployment
^^^^^^^^^^

To deploy dojot to a Kubernetes environment, we provide a script for clusters
with Ceph as storage solution.

To download the required files using git, run the following command: ::

  git clone https://github.com/dojot/kubernetes.git

or, to download a compressed zip file containing the data, use the following
link: https://github.com/dojot/kubernetes/archive/master.zip

This repository contains all the scripts and deployment files necessary to
properly setup dojot's containers. There is one file that must be changed:
``config.yaml``, which contains all the parameters used by these scripts. An
example of such file is this:

.. code-block:: yaml
   :linenos:

    ---
    version: 0.2.0-nightly20180319
    namespace: dojot
    storage:
      type: ceph
      cephMonitors:
      - '10.0.0.1:6789'
      - '10.0.0.2:6789'
      - '10.0.0.3:6789'
      cephAdminId: admin
      cephAdminKey: AQD85Z5a/wnlJBAARNISUDpC6RHc8g/UkUcDLA==
      cephUserId: admin
      cephUserKey: AQD85Z5a/wnlJBAARNISUDpC6RHc8g/UkUcDLA==
      cephPoolName: kube
    externalAccess:
      type: publicIP
      ips:
      - '10.0.0.1'
      - '10.0.0.2'
      - '10.0.0.3'
      ports:
        httpPort: 80
        httpsPort: 443
        mqttPort: 1883
        mqttSecurePort: 8883
    services:
      zookeeper:
        clusterSize: 3
      postgres:
        clusterSize: 3
      mongodb:
        replicas: 2
      kafka:
        clusterSize: 3
      auth:
        emailHost: 'smtp.gmail.com'
        emailUser: 'test@test.com'
    emailPassword: 'password'

From line 5 to 14, we have Ceph configuration parameters. The ``cephMonitors``
attribute specifies how many monitors are going to be used and by which address
they can be accessed. For more information about this element, check `ceph
monitors documentation
<http://docs.ceph.com/docs/jewel/rados/configuration/mon-config-ref/>`_.
``cephAdminId``, ``cephAdminKey``, ``cephUserId`` and ``cephUserKey``
attributes refers to user information. These values are set/generated in user
creation.

In ``externalAccess`` section we have what addresses and ports should be
exposed for external access. In ``services`` section, we can configure how many
replicas we want to each service and a few other parameters to configure that
service (for instance, auth taks an ``emailHost`` and ``emailUser``
parameters).

To configure and start the kubernetes cluster, just install all python
requirements and start the deploy.py script:

.. code-block:: bash

    pip install -r ./requirements.txt
    python ./deploy.py

.. _persistent-volumes page: https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes

.. _Kubernetes documentation: https://kubernetes.io/docs/tasks/tools/install-kubectl/
.. _Kubernetes setup documentation: https://kubernetes.io/docs/setup/
