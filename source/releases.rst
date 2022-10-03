Release history
===============

Gatka - 2022.09
----------------------

Services
+++++++++

New Services
************

Basic-Auth
^^^^^^^^^^
      - The basic-auth service is responsible for generating credentials to authenticate the devices when sending messages through the agents.

File-Mgmt
^^^^^^^^^
      - The file management is responsible for managing the resources of MinIO on the Dojot platform, allowing the storage and handling of files in various extensions.

Improvements and fixes
**********************

Http-Agent
^^^^^^^^^^

      - Insecure route mapped in kong

InfluxDB
^^^^^^^^

      - Support for influxdb chronograph

GUI
^^^

      - Feature device id creation

History
^^^^^^^

      - MongoDB - Allow changing data expiration time

Others
^^^^^^

      - Minor bug fixes

Deployments
+++++++++++

Docker-compose
***************

    - Add profile feature
    - Grafana and Prometeus Improvements
    - Suport for Self Signed Certificate Nginx
    - Suport for Loki
    - Minor bug fixes

Ansible-dojot
*************

    - Grafana and Prometeus Improvements
    - Suport for Self Signed Certificate Nginx
    - Suport for HTTPS in the NGINX
    - Suport for Keda
    - Suport for Loki
    - Suport for K3S
    - Suport for EKS
    - Suport for GKE
    - Suport for OKE
    - Minor bug fixes