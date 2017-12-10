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

.. TODO: Explain why persistence
.. TODO: CEPH for local cluster
.. TODO: Specific cloud provider storage

To make sure that all the data

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

Configuration
-------------

Deployment
----------
