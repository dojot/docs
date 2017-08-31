Architecture
============

This document describes the current architecture that guides the platform implementation, detailing
the components that comprise the solution, as well as their functionalities and how each of them
contribute to the platform as a whole.

While a brief explanation of each component is provided, this high level description does not
explain (or aims to explain) the minutia of each component's implementation. For that, please
refer to each component's own documentation.

.. contents:: Table of Contents
  :local:

Components
----------

This section should describe - in high level - the components that comprise the solution, giving
the reader enough detail to understand the responsabilities of the components, the technologies
involved in its design and implementation and its relation with its peers.

Infrastructure
--------------

This section should describe the components that are used as ready-made pieces of working software
that compose the solution, but have no implementation specific to the project. Relevant topics that
might be discussed here are:

 * The API gateway
 * Storage components (mongo, redis, HDFS, CEPH, etc.)
 * Processing libraries and environments (Spark, Flink, Storm, kafka-streaming, map-reduce, etc.)
 * Broker components (rabbitMQ, mosquitto, kafka, verneMQ, emqtt, etc.)

Communications
--------------

This section should provide the reader with the communication strategy used to bind together the
components that comprise the solution, as well as the interfaces (protocols, serialization formats)
available to the applications and devices developers.

Deployment strategies
---------------------

This section should list the deployment requirements and implementation decisions made to satisfy
those requirements. "Why orchestrator platform 'x'?", "How can this be deployed on commercial cloud
environments?", "How can this be deployed on stand-alone environments?" are all questions that
should be answered here.

Comparative analysis
--------------------

This section should detail the features that differenciate the platform from a "stock" deployment
of fiware, as well as a feature summary comparing the proposed solution with a reduced set of
third-party implementations of IoT platforms available.
