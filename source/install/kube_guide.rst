.. _kubernetes:

Installation Guide - Kubernetes
===============================

This document provides instructions on how to create a simple dojot deployment
environment on a multi-node environment, using kubernetes as the orchestration platform.

This deployment option as presented in this document is best suited to tests and assessment
of the platform, but with the appropriate changes might be evolved for production environments.

This guide has been checked on a Kubernetes cluster with Ceph as the underlying storage infrastructure and
it has also been tested on a Kubernetes cluster over the Google Cloud Platform

.. contents:: Table of Contents
  :local:

Dependencies
------------

This setup has as the first requirement a Kubernetes cluster that is properly configured and running.

The second requirement is a Kubernetes client correctly installed on the
machine that will start the deployment process

Kubernetes Cluster
******************

For this guide it is advised that you already have a functioning cluster.

If you desire to prepare a Kubernetes cluster from scratch,
up to date information and installation procedures can be found at the project's documentation:

https://kubernetes.io/docs/setup/

Persistent Storage
******************

To make sure that all the data from the containers running databases is persisted when containers fail or are moved
to different nodes of the Kubernetes environment it is necessary to attach persistent storage to the database pods.

Kubernetes requires that an infrastructure for persistent storage already exists on the cluster. As an example for how to
configure your persistent storage we provide files for two different kind of deployments, the first is for a local deployment
where a Ceph Cluster is used as storage backend, more information on Ceph may be found at: http://ceph.com/. The second example
is based on a Google Cloud deployment and use the existing persistent storage services that are provided by Google Cloud.
If you're deploying dojot using Kubernetes to a different cloud provider, some adjustments to fit the different deployments might
be necessary.

Information about the currently supported persistent storage for Kubernetes can be found at:
https://kubernetes.io/docs/concepts/storage/persistent-volumes/#types-of-persistent-volumes

Kubernetes Client
*****************

To install the Kubernetes client on your machine before proceeding with this guide, follow the proper instructions
as presented on the Kubernetes documentation:

https://kubernetes.io/docs/tasks/tools/install-kubectl/

Also, verify that your client is capable of connecting to the cluster.

For providing access for a local cluster, follow the documentation below:

https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/

If the Kubernetes cluster is running on a specific cloud platform like Google Cloud,
follow the steps as presented by your cloud provider.

Deployment
----------

To deploy dojot to a Kubernetes environment, we provide sample scripts and templates for two kinds of clusters. The examples
are for an environment comprised by Kubernetes with Ceph for storage, the second is a deployment to a Kubernetes environment
running on Google Cloud Platform.

For both environment it is necessary to download the scripts and templates before performing the deployment.

To download the required files using git, run the following command: ::

  git clone https://github.com/dojot/kubernetes.git

or, to download a compressed zip file containing the data, use the following link: https://github.com/dojot/kubernetes/archive/master.zip

Enter the downloaded folder and follow the instructions in the section that corresponds to your specific environment.

All the instructions provided in the following sections assume that the commands are being run on a linux terminal.

Cluster with Ceph
*****************

  .. TODO: Explain how to create kube pool and user on ceph
  .. TODO: Explain this steps

Google Cloud Platform
*********************

To deploy dojot to a Kubernetes cluster running over Google Cloud the only requirements are that you have your cluster
configured on Google Cloud and your local Kubernetes client is properly configured to access that cloud.

To execute the script to deploy to Google Cloud just run the following command on the terminal: ::

  ./deploy.sh GCP LB

The selected parameters set the type of storage to be used as GCP persistent storage and also set the external access
to use the load balancers as provided by the Google Cloud Platform.

Just wait until the script finishes running and then check for when all the pods have finished starting, to check
if all the pods are running correctly, run the command below and verify that all pods have reached a "Running" state,
this may take a while and retries for some pods. ::

  kubectl get pods -n dojot

After all the pods are running, run the following command in order to obtain the public ip address that is being used by the load balancer ::

  kubectl -n dojot get services external

The command will return the external ip used by the load balancer, with this IP you can access that ip using any browser at
http://EXTERNAL_IP

The initial user and password are admin and admin.
