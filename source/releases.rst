Release history
===============

Full Contact - 2022.02
----------------------

Services
+++++++++

New Services
************

IotAgent HTTP
^^^^^^^^^^^^^^^
      - The IotAgent HTTP can receive messages from physical devices, directly
        or through a gateway. In it's case, the messages are sent via HTTP with
        JSON payloads.

Improvements and fixes
**********************

History
^^^^^^^

      - Support for getting data in CSV format.
      - Fix TTL error - Now data is stored for 7 days by default.

InfluxDB
^^^^^^^^

      - Support for getting data in CSV format.

GUI
^^^

      - Support for export data in CSV format.

GUI-V2
^^^^^^

      - Support for export data in CSV format.

Cron
^^^^

      - Support for using SDK library

Others
^^^^^^

      - Minor bug fixes

Deployments
+++++++++++

Docker-compose
***************

    - IotAgent HTTP Deployment
    - Monitoring feature with Prometheus and Grafana for services:
         - InfluxDB
         - MongoDB
         - Kong
         - Kafka
         - VerneMQ
    - Minor bug fixes

Ansible-dojot
*************

    - IotAgent HTTP Deployment
    - Change Kubernetes container runtime to Containerd
    - Minor bug fixes