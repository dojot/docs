Installation Guide - Google Cloud Platform
==========================================

This document provides instructions on how to prepare a Google Cloud environment for a dojot deployment
using Kubernetes as the orchestrator.

This document will provide steps for creating a test and assessment environment for those who want to learn and
experiment with the dojot platform but prefer to run it on a cloud environment.

The steps as presented here can be evolved to real world deployments with proper
changes to fulfill your deployment use case

.. contents:: Table of Contents
  :local:

Creating a Project
------------------

To prepare an environment to deploy dojot on the Google Cloud Platform, the first thing that must be done is to create a project
for the deployment.

To create a project, go to the page https://console.cloud.google.com/projectcreate and define the new project's name.

Wait until the project creation is complete. Then, go to the page https://console.cloud.google.com/projectselector/home/dashboard,
click on the select button and choose the recently created project.

Creating a Cluster
------------------

Having the desired project selected, got to the kubernetes page of the GCP at the link https://console.cloud.google.com/kubernetes/.

Wait for the Kubernetes Engine to be ready and click on the create cluster button.

In the cluster creation page, define a name for your cluster and select an appropriate region for your deployment.
To create an evaluation and testing environment for dojot a cluster with 3 machines with 1 vCPU is
enough for the sake of experimenting with the kubernetes deployment.
With the options properly set, click on the create button and wait for the cluster to be created.

Getting the credentials
-----------------------

With the kubernetes cluster created, the next step is obtaining the cluster access credentials so your machine is able
to access the cluster and proceed with the deployment.

On the kubernetes page, on the list of created clusters, locate the cluster you just created,
on the right side of it click on the "Connect" button. Copy the first command that is provided and run this on
a terminal on your machine, this command will install the credentials. To run this command it is required
that the gcloud client is installed and properly configured on your machine. To install this client follow
the instructions provided by the google documentation at: https://cloud.google.com/sdk/docs/quickstarts

With the credential configured, proceed to the :ref:`Kubernetes Deployment Guide <kubernetes>`
