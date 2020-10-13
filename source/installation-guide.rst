Installation Guide
==================

This page contains information about how to deploy dojot using Docker compose and Kubernetes.

.. contents:: Table of Contents
  :local:


Hardware requirements
---------------------

The estimated hardware requirements for 500 devices with updates every 15s are:

.. list-table:: Hardware requirements for 500 devices
   :header-rows: 1

   *  - Deployment
      -
      - CPU
      - RAM
      - Free disk space
   *  - **Docker-compose**
      -
      - 4 Cores
      - 4GB
      - 10GB
   *  - **Kubernetes**
      - Master
      - 2 Cores
      - 2GB
      - 2GB
   *  - **Kubernetes**
      - Worker
      - 4 Cores
      - 4GB
      - 10GB


In addition, you need:

- **Network access**
- The following ports should be opened:
   - **TCP**: 8000 *(web interface access)*; 1883 *(MQTT, If you are going to use MQTT)*; 5896 *(LW2M, If you are going to use LW2M file server via HTTP instead of coap, UDP)*.
   - **TLS**: 8883 *(MQTTS, If you are going to use MQTT with TLS, in secure mode.)*.
   - **UDP**: 5683 and 5693 *(LW2M, If you are going to use LW2M)*; 5684 and 5694 *(LW2M, If you are going to use LW2M with DTLS)*.

Note: The above cores are approximately 3.5 GHz (x86-64)

Docker compose
--------------

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/aZ-Wtcd_Ydw?rel=0" frameborder="0"
    allowfullscreen></iframe><br/>

In this video tutorial above, version v0.4.2 is used, but the same video is valid for the current version, it is only necessary to change to version v0.4.4.

This document provides instructions on how to create a trivial deployment
environment on single host for *dojot*, using docker-compose as the processes
orchestration platform.

While very simple, this deployment option is best suited to development and
assessment of the platform and should not be used for production environments.

This guide has been checked on an Ubuntu 16.04 and 18.04 LTS environment.

The following sections describe all Docker compose dependencies.

Docker engine
^^^^^^^^^^^^^

Up to date information and installation procedures for the docker engine can be
found at the project's documentation:

https://docs.docker.com/engine/install/ubuntu/

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

  git checkout v0.4.4 -b v0.4.4


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

Read the :doc:`using-api-interface` and :doc:`using-web-interface` for more
information about how to interact with the platform.

Kubernetes
----------

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/qGUlBCmmIao?rel=0" frameborder="0"
    allowfullscreen></iframe>
In this video tutorial above, version v0.4.2 is used, but the same video is valid for the current version, it is only necessary to change to version v0.4.4


This section provides instructions on how to create a dojot deployment
on a multi-node environment, using Kubernetes as the orchestration
platform.

This deployment option when properly configured can be used for
creating production environments.

The following sections describe all dependencies and steps required
for this deployment.


Kubernetes Cluster
^^^^^^^^^^^^^^^^^^

For this guide it is advised that you already have a working K8s cluster.

If you need to build a Kubernetes cluster from scratch, up to date
information and installation procedures can be found at `Kubernetes setup
documentation`_.

.. _Kubernetes setup documentation: https://kubernetes.io/docs/setup/

Kubernetes Requirements
^^^^^^^^^^^^^^^^^^^^^^^

- The Kubernetes supported version is between **v1.11** and **v1.15** .
- Access to Docker Hub repositories
- (optional) a storage class that will be used for persistent storage

dojot Deployment
^^^^^^^^^^^^^^^^

To deploy dojot to Kubernetes it is advised the use of
ansible playbooks developed for dojot. The playbooks and
all the related code can be found on the repository `Ansible dojot`_.

The following steps will describe how to use this repository and
its playbooks.

1. Cloning the repository
.........................

The first deployment step is cloning the repository. To do so,
execute the command: ::

  git clone -b v0.4.4 https://github.com/dojot/ansible-dojot

2. Installing dependencies
..........................

The next step is installing the dependencies for running the
ansible playbook, this dependencies include ansible itself with
other modules that will be used to parse templates and communicate
with kubernetes.

Enter the folder where the repository was downloaded and install
the pip packages with the following commands: ::

  cd ansible-dojot
  pip install -r requirements.txt

3. Configuring the inventory
............................

For deploying kubernetes with ansible, it is necessary to model your
desired environment on an ansible inventory.

In the repository there is an '*inventory*' folder containing an
example inventory called '*example_local*' that can be used as the
starting point to creating the real environment inventory.

The first file that requires changes is the hosts.yaml. This file
describes the nodes that will be accessed by ansible to perform
the deployment. As the dojot deployment is done directly to K8s,
only a node with access to the kubernetes cluster is actually required.

The node that will access the cluster might be a kubernetes cluster node
that is accessible via SSH or event your local machine if it can reach
the kubernetes cluster with a configuration file.

On the example file, the access is done via a local node, where
the ansible script is executed. This node is described as localhost
in the hosts item of the group **all**.

These same nodes must be added as children of the group dojot-k8s.

To configure a local access on the hosts file, follow the example below:

.. code:: yaml

  ---
  all:
    hosts:
      localhost:
        ansible_connection: local
        ansible_python.version.major: 3
    children:
      dojot-k8s:
        hosts:
          localhost:

To configure remote access via ssh to a node of the cluster, follow
this other example:

.. code:: yaml

  ---
  all:
    hosts:
      NODE_NAME:
        ansible_host: NODE_IP
    children:
      dojot-k8s:
        hosts:
          NODE_NAME:

The next step is configuring the mandatory and optional variables
required for deploying dojot.

There is a document describing each of the variables that can be
configured at `Ansible dojot variables`_.

This variables must be set for the group '*dojot-k8s*', to do so set
their values on the file dojot.yaml on the folder '**group_vars/dojot-k8s/**'

.. _Ansible dojot: https://github.com/dojot/ansible-dojot
.. _Ansible dojot variables: https://github.com/dojot/ansible-dojot/blob/master/docs/vars.md

4. Executing the deployment playbook
....................................

Now that the inventory is set, the next step is executing
the deployment playbook.

To do so, run the following command:

.. code:: bash

  ansible-playbook -K -k -i inventories/YOUR_INVENTORY deploy.yaml

Wait for the playbook execution to finish without errors.

5. Accessing the deployed dojot environment
...........................................

Dojot access will be set using NodePorts, to view the proper ports to access
the environment it is necessary to check service configuration.

.. code:: bash

  kubectl get service -n dojot kong iotagent-mosca

This command will return the port used for external access to both the
REST API and GUI via kong and the MQTT port via iotagent-mosca.

