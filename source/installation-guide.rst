Installation Guide
==================

This page contains information about how to deploy dojot using Docker Compose and Kubernetes.

.. contents:: Table of Contents
  :local:


Hardware requirements
---------------------

When choosing hardware configurations for Dojot *deployment*, the number of devices and the message interval they will be 
configured with should be considered. As an example, the estimated hardware requirements for 500 devices with a message interval every 15s are:

.. list-table:: Hardware requirements for 500 devices
   :header-rows: 1

   *  - Deployment
      -
      - CPU
      - RAM
      - Free disk space
   *  - **Docker Compose**
      -
      - 4 Cores
      - 6GB
      - 20GB
   *  - **Kubernetes**
      - Master
      - 2 Cores
      - 2GB
      - 30GB
   *  - **Kubernetes**
      - Worker
      - 4 Cores
      - 6GB
      - 40GB
   *  - **Kubernetes**
      - Balancer
      - 1 Core
      - 1GB
      - 10GB


In addition, you need:

- **Network access**

- The following ports should be opened for Docker Compose deployment:
   - **TCP**: 8000 *(web interface access)*; 1883 *(MQTT, If you are going to use MQTT)*; 5896 *(LWM2M, If you are going to use LWM2M file server via HTTP instead of coap, UDP)*.
   - **TLS**: 8883 *(MQTTS, If you are going to use MQTT with TLS, in secure mode.)*.
   - **UDP**: 5683 and 5693 *(LWM2M, If you are going to use LWM2M)*; 5684 and 5694 *(LWM2M, If you are going to use LWM2M with DTLS)*.

- The following ports should be opened on load balancer for Kubernetes deployment:
   - **TCP**: 80 *(web interface access)*; 1883 *(MQTT, If you are going to use MQTT)*; 5896 *(LWM2M, If you are going to use LWM2M file server via HTTP instead of coap, UDP)*.
   - **TLS**: 8883 *(MQTTS, If you are going to use MQTT with TLS, in secure mode.)*.
   - **UDP**: 5683 and 5693 *(LWM2M, If you are going to use LWM2M)*; 5684 and 5694 *(LWM2M, If you are going to use LWM2M with DTLS)*.

Note: The above cores are approximately 3.5 GHz (x86-64)

Docker Compose
--------------

This document provides instructions on how to create a trivial deployment
environment on single host for *dojot*, using Compose as the processes
orchestration platform.

While very simple, this deployment option is best suited to development and
assessment of the platform and should not be used for production environments.

This guide has been checked on an Ubuntu 18.04 LTS environment.

The following sections describe all Docker Compose dependencies.

Docker Engine
^^^^^^^^^^^^^

This guide has been checked using Docker Engine 19.03

Up to date information and installation procedures for the Docker Engine can be
found at the project's documentation:

https://docs.docker.com/engine/install/ubuntu/

.. note::

  An optional step on the installation and configuration process of Docker on
  any given machine is the setting of who is eligible for creating/spawning
  Docker instances.

  Should the post-installation steps (more specifically the "Manage Docker as
  non-root user") have not been run, all Docker and Compose commands
  should be run by the super user (root), or as sudo.

  https://docs.docker.com/engine/installation/linux/linux-postinstall/

Docker Compose
^^^^^^^^^^^^^^

This guide has been checked using Compose 1.27

Up to date information and installation procedures for the Compose can
be found at the project's documentation:

https://docs.docker.com/compose/install/


Installation
^^^^^^^^^^^^

To setup the environment, merely clone the deployment repository and run the
commands below.

The Docker Compose enabled deployment scripts and configuration repository can
be found at:

https://github.com/dojot/docker-compose

or as git clone command: ::

  git clone https://github.com/dojot/docker-compose.git
  # Let's move into the repo - all commands in this page should be executed
  # inside it.
  cd docker-compose

Once the repository is properly cloned, select the version to be used by
checking out the appropriate tag (do notice that the tag_name has to be
replaced): ::

  # Must be run from within the deployment repo

  git checkout tag_name -b branch_name

For instance: ::

  git checkout v0.7.0 -b v0.7.0

.. note::
   For a guide on how to use **HTTPS** go to this link: https://github.com/dojot/docker-compose/tree/v0.7.0#how-to-secure-dojot-with-nginx-and-lets-encrypt


.. attention::
   Before running the command below, it is necessary to define your domain or IP in the ``.env`` file in the variable ``DOJOT_DOMAIN_NAME``.

That done, the environment can be brought up by: ::

  # Must be run from the root of the deployment repo.
  # May need sudo to work: sudo docker-compose up -d
  docker-compose up -d

.. note::
   To get completely ready, **healthy**, all services in this `docker-compose` take an average of at least 12 minutes.

To check individual container status, Docker's commands may be used, for
instance: ::

  # Shows the list of currently running containers, along with individual info
  docker ps

  # Shows the list of all configured containers, along with individual info
  docker ps -a

.. note::

  All Docker, Docker Compose commands may need sudo to work.

  To allow non-root users to manage Docker, please check Docker's documentation:

  https://docs.docker.com/engine/installation/linux/linux-postinstall/

Volumes
^^^^^^^

When we deploy dojot with the command 'docker-compose up -d' the volumes are enabled and created
by default.

The volumes of microservices that Dojot uses can be incompatible between dojot versions. This means
that you are unable to use dojot v0.4.x volumes in dojot v0.5.x or above and vice versa.

To use different versions of dojot in the same environment, you must first drop the volumes of the other version.

.. note::

  If you drop the dojot volumes you will also lose all data that you have collected on the platform
  so far.

To drop the volumes just pass the '-v' parameter in the 'docker-compose down' command as
displayed below: ::

  docker-compose down -v

That way volumes and dojot will be dropped and you will be able to deploy a different dojot version.

Usage
^^^^^

The web interface is available at ``http://localhost:8000``. The user is
``admin`` and the password is ``admin``. You also can interact with platform
using the :doc:`./components-and-apis`.

.. attention::
   Always change the ``admin`` user password to a suitable password and keep it safe.

Read the :doc:`using-api-interface` and :doc:`using-web-interface` for more
information about how to interact with the platform.

Kubernetes
----------

For simple installation with kubernetes please check the pdf below.

:download:`click here to access the dojot installation guide with kubernetes <pdf/Dojot-Installation-Guide.pdf>`

If you want to install a more robust Dojot that supports up to 100k devices, check the pdf below.

.. note::

  In the 100k environment, dojot does not process or store messages sent by devices.
  This environment will only work for load tests and only a few dojot components will be available.

:download:`click here to access the dojot 100k installation guide with kubernetes <pdf/Dojot-100k-Installation-Guide.pdf>`

.. note::

  Unfortunately in this tutorial we do not have support for the English language yet.