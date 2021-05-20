Release history
===============

Eskrima - 2021.05
------------------

Services
++++++++++

Improvements and fixes
**********************

V2k-Bridge
^^^^^^^^^^

      - Added graceful shutdown and health check for the service

K2v-Bridge
^^^^^^^^^^

      - Added graceful shutdown and health check for the service

Kafka-WS
^^^^^^^^

      - Added graceful shutdown and health check for the service

Influxdb-Retriever
^^^^^^^^^^^^^^^^^^

      - Added graceful shutdown and health check for the service

Influxdb-Storer
^^^^^^^^^^^^^^^

      - Added graceful shutdown and health check for the service

Others
^^^^^^

      - Minor bug fixes

Deployments
+++++++++++

Docker-compose
***************

    - Minor bug fixes

Ansible-dojot
*************

    - The docker and containerd version was defined in the kubernetes playbook
    - Bug fixes in Kafka-WS Readiness and Liveness Probe
    - Bug fixes in Nginx
    - Documentation improvements
