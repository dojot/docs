.. _Components and APIs:

Components and APIs
===================

Components
----------

.. list-table:: Components
  :header-rows: 1

  * - Component
    - Repository / Main site
    - Documentation
  * - MongoDB
    - `MongoDB official site`_
    - `MongoDB documentation`_
  * - postgres
    - `PostgreSQL official site`_
    - `PostgreSQL documentation`_
  * - Kong API gateway (Community Edition)
    - `Kong official site`_
    - `Kong documentation`_
  * - redis
    - `Redis official site`_
    - `Redis documentation`_
  * - zookeeper
    - `Zookeeper official site`_
    - `Zookeeper documentation`_
  * - Kafka
    - `Kafka official site`_
    - `Kafka documentation`_
  * - auth
    - `GitHub - auth`_
    - `readthedocs - auth`_
  * - History
    - `GitHub - history`_
    -
  * - DeviceManager
    - `GitHub - DeviceManager`_
    - `readthedocs - DeviceManager`_
  * - Image manager
    - `GitHub - image-manager`_
    -
  * - GUI
    - `GitHub - GUI`_
    -
  * - Flow broker
    - `GitHub - flowbroker`_
    -
  * - Data broker
    - `GitHub - data-broker`_
    -
  * - iotagent-mosca
    - `GitHub - iotagent-mosca`_
    -
  * - EJBCA-REST
    - `GitHub - EJBCA-REST`_
    -
  * - Data Manager
    - `GitHub - Data Manager`_
    -
  * - Cron
    - `GitHub - Cron`_
    -



Exposed APIs
------------

.. list-table:: APIs
   :header-rows: 1

  * - Endpoint
    - Purpose
    - Component API
    - Repository
  * - /device
    -  Device management
    - `API - DeviceManager`_
    - `GitHub - DeviceManager`_
  * - /template
    -  Template management
    - `API - DeviceManager`_
    - `GitHub - DeviceManager`_
  * - /flows
    -  Flow management
    - `API - flowbroker`_
    - `GitHub - flowbroker`_
  * - /auth
    -  User authentication
    - `API - auth`_
    - `GitHub - auth`_
  * - /auth/revoke
    -  User authentication
    - `API - auth`_
    - `GitHub - auth`_
  * - /auth/user
    -  User authentication
    - `API - auth`_
    - `GitHub - auth`_
  * - /history
    -  Device historical data
    - `API - history`_
    - `GitHub - history`_
  * - /gui
    -  Graphical User Interface
    -
    - `GitHub - GUI`_
  * - /sign
    -  Public key signing
    - `API - EJBCA-REST`_
    - `GitHub - EJBCA-REST`_
  * - /ca
    -  Certification-Auth. functions
    - `API - EJBCA-REST`_
    - `GitHub - EJBCA-REST`_
  * - /image
    - Device image management
    - `API - image-manager`_
    - `GitHub - image-manager`_
  * - /import
    - Data Manager
    - `API - Data Manager`_
    - `GitHub - Data Manager`_
  * - /export
    - Data Manager
    - `API - Data Manager`_
    - `GitHub - Data Manager`_
  * - /cron
    - Data Manager
    - `API - Cron`_
    - `GitHub - Cron`_

The API gateway used in dojot reroutes some of these endpoints so that they
become uniform: all of them are accessible through the same port (default is
TCP port 8000) and have the same naming scheme. Each component, though, might
have something different in its configuration and API documentation. The
following table shows which endpoint exposed by the API gateway is mapped to
which component endpoint.

.. list-table:: Original endpoints
   :header-rows: 1

   * - Service
     - Original endpoint
     - Endpoint
   * - DeviceManager
     - host:5000/device
     - host:8000/device
   * - DeviceManager
     - host:5000/template
     - host:8000/template
   * - mashup
     - host:3000/
     - host:8000/flows
   * - auth
     - host:5000/
     - host:8000/auth
   * - auth
     - host:5000/auth/revoke
     - host:8000/auth/revoke
   * - auth
     - host:5000/user
     - host:8000/auth/user
   * - STH
     - host:8666/
     - host:8000/history
   * - GUI
     - host/
     - host:8000/gui
   * - ejbca
     - host:5583/sign
     - host:8000/sign
   * - ejbca
     - host:5583/ca
     - host:8000/ca
   * - DataManager
     - host:3000/import
     - host:8000/import
   * - DataManager
     - host:3000/export
     - host:8000/export
   * - Cron
     - host:5000/cron
     - host:8000/cron


Kafka messages
--------------

These are the messages sent by components and their subjects. If you are
developing a new internal component (such as a new IoT agent), see `API -
data-broker`_ to check how to receive messages sent by other components in
dojot.

.. list-table:: Original endpoints
   :header-rows: 1

   * - Component
     - Message
     - Subject
   * - DeviceManager
     - Device CRUD (`Messages - DeviceManager`_)
     - ``dojot.device-manager.device``
   * - iotagent-mosca
     - Device data update (`Messages - iotagent-mosca`_)
     - ``device-data``
   * - auth
     - Tenants creation/removal (`Messages - auth`_)
     - ``dojot.tenancy``

.. _MongoDB documentation: https://docs.mongodb.com/manual/
.. _MongoDB official site: https://www.mongodb.com/
.. _PostgreSQL documentation: https://www.postgresql.org/docs/
.. _PostgreSQL official site: https://www.postgresql.org
.. _Kong official site: https://konghq.com/kong-community-edition/
.. _Kong documentation: https://getkong.org/docs/
.. _Redis official site: https://redis.io/
.. _Redis documentation: https://redis.io/documentation
.. _Zookeeper official site: https://zookeeper.apache.org/
.. _Zookeeper documentation: https://zookeeper.apache.org/documentation.html
.. _Kafka official site: https://kafka.apache.org/
.. _Kafka documentation: http://kafka.apache.org/documentation/


.. _GitHub - auth: https://github.com/dojot/auth
.. _API - auth: https://dojot.github.io/auth/apiary_latest.html
.. _readthedocs - auth: http://dojotdocs.readthedocs.io/projects/auth/en/latest/
.. _Messages - auth: https://dojotdocs.readthedocs.io/projects/auth/en/latest/kafka-messages.html

.. _GitHub - history: https://github.com/dojot/history
.. _API - history: https://dojot.github.io/history/apiary_latest.html


.. _GitHub - DeviceManager: https://github.com/dojot/device-manager
.. _API - DeviceManager: https://dojot.github.io/device-manager/apiary_latest.html
.. _readthedocs - DeviceManager: http://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/
.. _Messages - DeviceManager: http://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/kafka-messages.html

.. _GitHub - image-manager: https://github.com/dojot/image-manager
.. _API - image-manager: https://dojot.github.io/image-manager/apiary_latest.html


.. _GitHub - GUI: https://github.com/dojot/gui


.. _GitHub - flowbroker: https://github.com/dojot/flowbroker
.. _API - flowbroker: https://dojot.github.io/flowbroker/apiary_latest.html

.. _GitHub - data-broker: https://github.com/dojot/data-broker
.. _API - data-broker: https://dojot.github.io/data-broker/apiary_latest.html

.. _Messages - iotagent-mosca: http://dojotdocs.readthedocs.io/projects/iotagent-mosca/en/latest/operation.html#sending-messages-to-other-components-via-kafka

.. _GitHub - iotagent-mosca: https://github.com/dojot/iotagent-mosca

.. _GitHub - data-broker: https://github.com/dojot/data-broker

.. _GitHub - mashup: https://github.com/dojot/mashup

.. _GitHub - EJBCA-REST: https://github.com/dojot/ejbca-rest
.. _API - EJBCA-REST: https://dojot.github.io/ejbca-rest/apiary_latest.html

.. _GitHub - Data Manager: https://github.com/dojot/data-manager
.. _API - Data Manager: https://dojot.github.io/data-manager/apiary_v0.4.0.html

.. _GitHub - Cron: https://github.com/dojot/cron
.. _API - Cron: #

