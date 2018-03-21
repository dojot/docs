Components and APIs
===================

Components
----------

.. list-table:: Components
  :header-rows: 1

  * - Component
    - GitHub repository
    - Documentation
  * - History
    - `GitHub - history-ws`_
    - `readthedocs - history-ws`_
  * - mongodb
    -
    - `mongodb documentation`_
  * - Mosquitto (MQTT broker)
    -
    - `Mosquitto documentation`_
  * - GUI
    - `GitHub - GUI`_
    -
  * - iotagent-json
    - `GitHub - iotagent-json`_
    - `readthedocs - iotagent-json`_
  * - data-broker
    - `GitHub - data-broker`_
    -
  * - DeviceManager
    - `GitHub - DeviceManager`_
    - `readthedocs - DeviceManager`_
  * - auth
    - `GitHub - auth`_
    - `readthedocs - auth`_
  * - postgres
    -
    - `postgres documentation`_
  * - Kong API gateway
    -
    - `Kong documentation`_
  * - Orchestrator/Mashup
    - `GitHub - mashup`_
    -
  * - redis
    -
    - `Redis documentation`_
  * - zookeeper
    -
    - `Zookeeper documentation`_
  * - Kafka
    -
    - `Kafka documentation`_
  * - EJBCA-REST
    - `GitHub - EJBCA-REST`_
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
     -
     - `GitHub - mashup`_
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
     - `API - STH`_
     - `GitHub - STH`_
   * - /metric
     -  Context broker
     - `Orion v1`_, `Orion v2`_
     - `GitHub - Orion`_
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
   * - `Orion v1`_ or `Orion v2`_
     - host:1026/
     - host:8000/metric
   * - GUI
     - host/
     - host:8000/gui
   * - ejbca
     - host:5583/sign
     - host:8000/sign
   * - ejbca
     - host:5583/ca
     - host:8000/ca


.. _mongodb documentation: https://docs.mongodb.com/manual/
.. _Mosquitto documentation: https://www.eclipse.org/mosquitto/man/
.. _postgres documentation: https://www.postgresql.org/docs/
.. _Kong documentation: https://getkong.org/docs/
.. _Redis documentation: https://redis.io/documentation
.. _Zookeeper documentation: https://zookeeper.apache.org/documentation.html
.. _Kafka documentation: http://kafka.apache.org/documentation/

.. _GitHub - history-ws: https://github.com/dojot/history-ws
.. _API - STH: https://github.com/telefonicaid/fiware-sth-comet#api-walkthrough
.. _readthedocs - history-ws: https://github.com/dojot/history-ws

.. _GitHub - GUI: https://github.com/dojot/gui

.. _GitHub - iotagent-json: https://github.com/dojot/iotagent-json
.. _readthedocs - iotagent-json: http://dojotdocs.readthedocs.io/projects/iotagent-json/en/latest/

.. _GitHub - data-broker: https://github.com/dojot/data-broker

.. _Orion v1: http://telefonicaid.github.io/fiware-orion/api/v1/
.. _Orion v2: http://telefonicaid.github.io/fiware-orion/api/v2/stable/

.. _GitHub - DeviceManager: https://github.com/dojot/device-manager
.. _API - DeviceManager: http://dojot.github.io/device-manager/apiary_latest.html
.. _readthedocs - DeviceManager: http://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/

.. _GitHub - auth: https://github.com/dojot/auth
.. _readthedocs - auth: http://dojotdocs.readthedocs.io/projects/auth/en/latest/
.. _API - auth: http://dojot.github.io/auth/apiary_latest.html

.. _GitHub - mashup: https://github.com/dojot/mashup

.. _GitHub - EJBCA-REST: http://dojotdocs.readthedocs.io/projects/EJBCA-REST/en/latest/
.. _API - EJBCA-REST: http://dojot.github.io/ejbca-rest/apiary_latest.html

