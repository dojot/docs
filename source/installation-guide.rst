##################
Installation Guide
##################

This page contains information about how to deploy dojot using Docker compose,
Kubernetes and Google Cloud Platform.

.. contents:: Table of Contents
  :local:


Installation - Docker compose
=============================

This document provides instructions on how to create a trivial deployment
environment on single host for *dojot*, using docker-compose as the processes
orchestration platform.

While very simple, this deployment option is best suited to development and
assessment of the platform and should not be used for production environments.

This guide has been checked on an Ubuntu 16.04 LTS environment.

Dependencies
------------

This setup has two software requirements docker engine and docker-compose.

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
------------

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

  git checkout tag_name

For instance: ::

  git checkout 0.1.0-dojot

Or if you're brave enough: ::

  git checkout master

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
-----

The web interface is available at ``http://localhost:8000``. The user is
``admin`` and the password is ``admin``. You also can interact with platform
using the :doc:`REST API <../components-and-apis>`.

Read the :doc:`../user_guide` for more information about how to interact with
the platform.


Installation - Google Cloud Platform
====================================

This document provides instructions on how to prepare a Google Cloud
environment for a dojot deployment using Kubernetes as the orchestrator.

This document will provide steps for creating a test and assessment environment
for those who want to learn and experiment with the dojot platform but prefer
to run it on a cloud environment.

The steps as presented here can be evolved to real world deployments with
proper changes to fulfill your deployment use case


Creating a Project
------------------

To prepare an environment to deploy dojot on the Google Cloud Platform, the
first thing that must be done is to create a project for the deployment.

To create a project, go to the page
https://console.cloud.google.com/projectcreate and define the new project's
name.

Wait until the project creation is complete. Then, go to the page
https://console.cloud.google.com/projectselector/home/dashboard, click on the
select button and choose the recently created project.

Creating a Cluster
------------------

Having the desired project selected, got to the kubernetes page of the GCP at
the link https://console.cloud.google.com/kubernetes/.

Wait for the Kubernetes Engine to be ready and click on the create cluster
button.

In the cluster creation page, define a name for your cluster and select an
appropriate region for your deployment. To create an evaluation and testing
environment for dojot a cluster with 3 machines with 1 vCPU is enough for the
sake of experimenting with the kubernetes deployment.

With the options properly set, click on the create button and wait for the
cluster to be created.

Getting the credentials
-----------------------

With the kubernetes cluster created, the next step is obtaining the cluster
access credentials so your machine is able to access the cluster and proceed
with the deployment.

On the kubernetes page, on the list of created clusters, locate the cluster you
just created, on the right side of it click on the "Connect" button. Copy the
first command that is provided and run this on a terminal on your machine, this
command will install the credentials. To run this command it is required that
the gcloud client is installed and properly configured on your machine. To
install this client follow the instructions provided by the google
documentation at: https://cloud.google.com/sdk/docs/quickstarts

With the credential configured, proceed to the `Installation - Kubernetes`_

Installation - Kubernetes
=========================

This document provides instructions on how to create a simple dojot deployment
environment on a multi-node environment, using kubernetes as the orchestration
platform.

This deployment option as presented in this document is best suited to tests
and assessment of the platform, but with the appropriate changes might be
evolved for production environments.

This guide has been checked on a Kubernetes cluster with Ceph as the underlying
storage infrastructure and it has also been tested on a Kubernetes cluster over
the Google Cloud Platform


Dependencies
------------

This setup has as the first requirement a Kubernetes cluster that is properly
configured and running.

The second requirement is a Kubernetes client correctly installed on the
machine that will start the deployment process

Kubernetes Cluster
^^^^^^^^^^^^^^^^^^

For this guide it is advised that you already have a functioning cluster.

If you desire to prepare a Kubernetes cluster from scratch, up to date
information and installation procedures can be found at the project's
documentation:

https://kubernetes.io/docs/setup/

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
be found at `persistent-volumes page
<https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes>`_

Kubernetes Client
^^^^^^^^^^^^^^^^^

To install the Kubernetes client on your machine before proceeding with this
guide, follow the proper instructions as presented on the `Kubernetes
documentation <https://kubernetes.io/docs/tasks/tools/install-kubectl/>`_

Also, verify that your client is capable of connecting to the cluster.

For providing access for a local cluster, follow the documentation below:

https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/

If the Kubernetes cluster is running on a specific cloud platform like Google
Cloud, follow the steps as presented by your cloud provider.

Deployment
----------

To deploy dojot to a Kubernetes environment, we provide sample scripts and
templates for two kinds of clusters. The examples are for an environment
comprised by Kubernetes with Ceph for storage, the second is a deployment to a
Kubernetes environment running on Google Cloud Platform.

For both environment it is necessary to download the scripts and templates
before performing the deployment.

To download the required files using git, run the following command: ::

  git clone https://github.com/dojot/kubernetes.git

or, to download a compressed zip file containing the data, use the following
link: https://github.com/dojot/kubernetes/archive/master.zip

Enter the downloaded folder and follow the instructions in the section that
corresponds to your specific environment.

All the instructions provided in the following sections assume that the
commands are being run on a linux terminal.

Google Cloud Platform
^^^^^^^^^^^^^^^^^^^^^

To deploy dojot to a Kubernetes cluster running over Google Cloud the only
requirements are that you have your cluster configured on Google Cloud and your
local Kubernetes client is properly configured to access that cloud.

To execute the script to deploy to Google Cloud just run the following command on the terminal: ::

  ./deploy.sh GCP LB

The selected parameters set the type of storage to be used as GCP persistent
storage and also set the external access to use the load balancers as provided
by the Google Cloud Platform.

Just wait until the script finishes running and then check for when all the
pods have finished starting, to check if all the pods are running correctly,
run the command below and verify that all pods have reached a "Running" state,
this may take a while and retries for some pods. ::

  kubectl get pods -n dojot

After all the pods are running, run the following command in order to obtain
the public ip address that is being used by the load balancer ::

  kubectl -n dojot get services external

The command will return the external ip used by the load balancer, with this IP
you can access that ip using any browser at http://EXTERNAL_IP

The initial user and password are admin and admin.

Cluster with Ceph
^^^^^^^^^^^^^^^^^

To deploy dojot to a Kubernetes Cluster where you have as persistent storage
infrastructure a Ceph Cluster you will need the configuration file for
accessing Ceph.

Also you will need to set some information regarding your Ceph cluster on the
manifest files.

Edit the file "manifests/STORAGE/CEPH/rbd-provisioner.yaml" and change the
values of the pool and the userId to match those of your specific environment.
Also it is necessary to get the key for the admin user and the client user.
With this keys at hand, convert then to base 64, this may be done at your
terminal running the command: ::

  echo "KEY" | base64

The value that is returned must be added to the
"manifests/STORAGE/CEPH/ceph-secret-admin.yaml" and
"manifests/STORAGE/CEPH/ceph-secret-user.yaml" respectively at the field key.

Also you may choose to deploy with a load balancer if your infrastructure
provide one, otherwise you may deploy selecting a public ip of one of the
kubernetes cluster nodes as the point of access for the environment.

To execute the script and deploy with Ceph and a public ip just run the following command on the terminal: ::

  ./deploy.sh CEPH PUBLIC_IP

Wait while the script starts the deployment, you will be prompted for two
parameters during the deployment, the path for the ceph configuration file and
the desired public ip. Enter this parameters and type enter when prompted.

Just wait until the script finishes running and then check for when all the
pods have finished starting, to check if all the pods are running correctly,
run the command below and verify that all pods have reached a "Running" state,
this may take a while and retries for some pods. ::

  kubectl get pods -n dojot

After all the pods are runninf, you can access your dojot deployment using the
public ip that was defined http://PUBLIC_IP

The initial user and password are admin and admin.
