Release history
===============

Gatka - 2021.08
------------------

Services
+++++++++

New Services
************

Keycloak
^^^^^^^^
 - The old service `auth` has been removed and we now use Keycloak.

Improvements and fixes
**********************

 - Changes were made to several services to make them work with keycloak.


Others
^^^^^^

      - Minor bug fixes

Deployments
+++++++++++

Docker-compose
***************
    - Documentation improvements
    - Minor bug fixes

Ansible-dojot
*************

    - Documentation improvements
    - Volumes feature
    - Labels feature
    - Docker log files rotation
    - Update Kubernetes version to 1.19.8
    - Minor bug fixes

Libraries
+++++++++

 - Changes were made to dojo-module-nodejs, iot agent-nodejs, dojot-module-python, dojot-module-java, iotagent-jav libs to work with keycloak.
