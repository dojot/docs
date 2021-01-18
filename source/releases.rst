Release history
===============

Defendu - 2021.01
------------------

Services
++++++++++

New Services
************

Dojot Locust
^^^^^^^^^^^^

      - New tool using the Locust framework (https://locust.io/) for load testing in the dojot with the MQTT protocol

IotAgent MQTT VerneMQ
^^^^^^^^^^^^^^^^^^^^^

      - New scalable MQTT IotAgent using the VerneMQ broker (https://vernemq.com/) and auxiliary integration services (V2K and K2V)

Gui-V2
^^^^^^

      - New graphical interface service, still under development, providing:

        - Tool for creating a customizable dashboard with:

          - Graphics of various formats
          - Tables

Kafka-WS
^^^^^^^^

      - New Kafka “real time” data consumption service via websocket with filters and partial data return

Kafka2FTP
^^^^^^^^^

      - New Kafka messaging service to a FTP server

X.509 Identity Management
^^^^^^^^^^^^^^^^^^^^^^^^^

      - New dojot x509 certificate service using EJBCA (https://www.ejbca.org/)

InfluxDB Storer and Retriever
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

      - InfluxDB Storer is responsible for consuming Kafka data and writing it to InfluxDB
      - InfluxDB Retriever retrieves data that were written by InfluxDB Storer in InfluxDB via API REST

Improvements and fixes
**********************

Auth
^^^^

      - Documentation improvements

Cron
^^^^

      - Documentation improvements

Data-Broker
^^^^^^^^^^^

      - Documentation improvements
      - Several code improvements
      - Removal of topic mapping from kafka by id via Redis, it is now possible to use the topic name directly


Data-Manager
^^^^^^^^^^^^

      - Documentation improvements

Device-Manager
^^^^^^^^^^^^^^

      - Documentation improvements
      - Performance improvements in code

Flowbroker
^^^^^^^^^^

      - Documentation improvements
      - Various code performance improvements including better parallelization of processing queues
      - Minor bug fixes
      - Support for Kubernetes 17 with remote nodes
      - New “Publish FTP” node

Image-Manager
^^^^^^^^^^^^^

      - Documentation improvements

IotAgent-Leshan
^^^^^^^^^^^^^^^

      - Documentation improvements
      - Minor bug fixes

IotAgent Mosca
^^^^^^^^^^^^^^

      - Added CRL support
      - Added control of maximum active and inactivity connection time
      - Integration with the new X.509 Identity Management service
      - Minor bug fixes

GUI
^^^

      - Added historical reporting option by device
      - Added x509 certificate generation option for a device
      - Base url route customization
      - Minor bug fixes

History
^^^^^^^

      - Documentation improvements
      - New “First N” filter option that returns the first N data that is currently persisted
      - Improvements to data indexing in MongoDB
      - Minor bug fixes

Kong
^^^^

      - Migration to Kong Gateway version 2 (https://konghq.com/kong/)

Deployments
+++++++++++

Docker-compose
***************

    - Upgrade to docker-compose version 3.8
    - Update version of external services
    - Added new services

Ansible-dojot
*************

    - Several improvements mainly aimed at scalability and simplification of the installation process
    - Update to version 17 of kubernetes
    - Added Load Balancer - Nginx
    - Added Prometheus and Grafana to monitor part of the infrastructure (VMs, VerneMQ, Kubernetes, etc.)
    - Documentation improvements

Libraries
+++++++++

dojot-module-nodejs
*******************

   - Minor bug fixes

dojot-microservice-sdk-js
*************************

  - New dojot library in node.js with:

    - Kafka Handlers -  Module responsible for Consumer (can use regular expressions in topics) and Producer
    - Config Manager -  Module responsible for creating the standardized configuration file for the services
    - Service State Manager - Module to define graceful shutdown and health check for the service
    - WebUtils - Module for creating a server and a web structure (Express.js) to handle HTTP (S) requests.
    - Logger - Log module to be used in services in a standardized way
