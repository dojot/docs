Release history
===============

Full Contact - 2021.07
----------------------

Services
+++++++++

New Services
************

Certificate ACL
^^^^^^^^^^^^^^^
      - The certificate-acl is responsible for keeping in memory an association between certificates
        fingerprint and their owners so that dojot services that needs this information can query it
        instead of x509-identity-mgmt service, which keeps this information only on disk.

Cert-sidecar
^^^^^^^^^^^^
      - The Cert-Sidecar, certificate sidecar, is a service utility for managing
        x509-identity-mgmt certificates for use with TLS connections.


Improvements and fixes
**********************

IotAgent MQTT VerneMQ
^^^^^^^^^^^^^^^^^^^^^

      - Integration with Cert-sidecar
      - Integration with Certificate ACL

V2k-Bridge
~~~~~~~~~~

      - Integration with Cert-sidecar

K2v-Bridge
~~~~~~~~~~

      - Integration with Cert-sidecar

Kakfa-ws
^^^^^^^^

      - Improvements in the service's health check


X.509 Identity Management
^^^^^^^^^^^^^^^^^^^^^^^^^

      -  Support for external certificates


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

    - Documentation improvements
    - Volumes feature
    - Labels feature
    - Docker log files rotation
    - Update Kubernetes version to 1.19.8
    - Minor bug fixes
