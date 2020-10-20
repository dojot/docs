Components and APIs
===================

Components
----------

.. list-table:: Components
  :header-rows: 1

  * - | Component
    - | Repository
      | / Main site
    - | Documentation
      | for Component
    - | Component API
      | Documentation
  * - MongoDB
    - `MongoDB site`_
    - `MongoDB doc.`_
    -
  * - Postgres
    - `PostgreSQL site`_
    - `PostgreSQL doc.`_
    -
  * - | Kong API gateway
      | (Community Edition)
    - `Kong site`_
    - `Kong doc.`_
    -
  * - Redis
    - `Redis site`_
    - `Redis doc.`_
    -
  * - Zookeeper
    - `Zookeeper site`_
    - `Zookeeper doc.`_
    -
  * - Kafka
    - `Kafka site`_
    - `Kafka doc.`_
    -
  * - VerneMQ
    - `VerneMQ site`_
    - `VerneMQ doc.`_
    -
  * - Leshan
    - `Leshan site`_
    - `Leshan doc.`_
    -
  * - Auth
    - `GitHub - auth`_
    -
    - `API - auth`_
  * - Dojot Kong
    - `GitHub - Dojot Kong`_
    -
    -
  * - History
    - `GitHub - history`_
    -
    - `API - history`_
  * - Device Manager
    - `GitHub - DeviceManager`_
    -
    - `API - DeviceManager`_
  * - Image Manager
    - `GitHub - image-manager`_
    -
    - `API - image-manager`_
  * - GUI
    - `GitHub - GUI`_
    -
    -
  * - Dashboard
    - `GitHub - V2`_
    -
    - 
  * - Flowbroker
    - `GitHub - flowbroker`_
    -
    - `API - flowbroker`_
  * - Databroker
    - `GitHub - data-broker`_
    -
    - `API - data-broker`_
  * - | IotAgent VerneMQ
      | (MQTT) - **Default**
    - `GitHub - iotagent-vernemq`_
    -
    -
  * - | IotAgent Mosca
      | (MQTT) - **Legacy**
    - `GitHub - iotagent-mosca`_
    -
    -
  * - | IotAgent Leshan
      | (LWM2M)
    - `GitHub - iotagent-leshan`_
    -
    -
  * - Data Manager
    - `GitHub - Data Manager`_
    -
    - `API - Data Manager`_
  * - Cron
    - `GitHub - Cron`_
    -
    - `API - Cron`_
  * - X.509 Identity Management
    - `GitHub - x509-identity-mgmt`_
    -
    - `API - x509-identity-mgmt`_
  * - Kafka2Ftp
    - `GitHub - Kafka2Ftp`_
    -
    -
  * - Kafka WS
    - `GitHub - Kafka WS`_
    -
    - `API - kafka-ws`_


Exposed APIs (API Gateway)
--------------------------

The API gateway used in dojot reroutes some of endpoints from component.
The following table shows which **Exposed endpoint
by the API gateway** is mapped to which **component endpoint**,
its  **component endpoint Documentation** and
whether the endpoint **needs authentication** when used via API Gateway.
See more about how using APIs in :doc:`./using-api-interface`.

.. list-table:: Exposed endpoints
   :header-rows: 1

   * - | Component
     - | Exposed endpoint
       | by the API gateway
     - | Component
       | Endpoint
     - | Component
       | Endpoint
       | Documentation
     - | Needs
       | Authentication
   * - GUI
     - /
     - /
     -
     - No
   * - Dashboard
     - /v2
     - /
     -
     - No
   * - Device Manager
     - /device
     - /device
     - `API - DeviceManager`_
     - Yes
   * - Device Manager
     - /template
     - /template
     - `API - DeviceManager`_
     - Yes
   * - Flowbroker
     - **/flows**
     - /
     - `API - flowbroker`_
     - Yes
   * - Auth
     - **/auth**
     - /
     - `API - auth`_
     - No
   * - Auth
     - **/auth**/revoke
     - /revoke
     - `API - auth`_
     - No
   * - Auth
     - **/auth**/user
     - /user
     - `API - auth`_
     - Yes
   * - Auth
     - **/auth**/pap
     - /pap
     - `API - auth`_
     - Yes
   * - History
     - **/history**
     - /
     - `API - history`_
     - Yes
   * - Data Manager
     - /import
     - /import
     - `API - Data Manager`_
     - Yes
   * - Data Manager
     - /export
     - /export
     - `API - Data Manager`_
     - Yes
   * - Cron
     - /cron
     - /cron
     - `API - Cron`_
     - Yes
   * - Image Manager
     - **/fw-image**
     - /
     - `API - image-manager`_
     - Yes
   * - Data Broker
     - | /device/
       | {deviceID}
       | /latest
     - | /device/
       | {deviceID}
       | /latest
     - `API - data-broker`_
     - Yes
   * - Data Broker
     - /subscription
     - /subscription
     - `API - data-broker`_
     - Yes
   * - Data Broker
     - /stream
     - /stream
     - `API - data-broker`_
     - Yes
   * - Data Broker
     - /socket.io
     - /socket.io
     - `API - data-broker`_
     - No
   * - X.509 Identity Management
     - **/x509/v1**
     - **/api/v1**
     - `API - x509-identity-mgmt`_
     - Yes
   * - Kafka WS
     - **/kafka-ws/v1/ticket**
     -  /v1/ticket
     -
     - Yes
   * - Kafka WS
     - **/kafka-ws/v1**
     - /v1
     -
     - No

**NOTE: Some of the components' endpoints aren't exposed, but are used internally.**


In addition, the API gateway reroutes the endpoints with their ports from component, so that they
become uniform: all of them are accessible through the same port (default is
TCP port 8000), see the following table.

.. list-table:: Original endpoints to The API gateway
   :header-rows: 1

   * - Component
     - Original endpoint
     - Gateway Endpoint
   * - GUI
     - host:80/
     - host:8000/
   * - Dashboard 
     - host:80/
     - host:8000/v2
   * - Device Manager
     - host:5000/device
     - host:8000/device
   * - Device Manager
     - host:5000/template
     - host:8000/template
   * - Flowbroker
     - host:80/
     - host:8000/flows
   * - Auth
     - host:5000/
     - host:8000/auth
   * - Auth
     - host:5000/revoke
     - host:8000/auth/revoke
   * - Auth
     - host:5000/user
     - host:8000/auth/user
   * - Auth
     - host:5000/pap
     - host:8000/auth/pap
   * - History
     - host:8000/
     - host:8000/history
   * - Data Manager
     - host:3000/import
     - host:8000/import
   * - Data Manager
     - host:3000/export
     - host:8000/export
   * - Cron
     - host:5000/cron
     - host:8000/cron
   * - Image Manager
     - host:5000/
     - host:8000/fw-image
   * - Data Broker
     - host:80/device/{{deviceID}}/latest
     - host:8000/device/{deviceID}/latest
   * - Data Broker
     - host:80/subscription
     - host:8000/subscription
   * - Data Broker
     - host:80/stream
     - host:8000/stream
   * - Data Broker
     - host:80/socket.io
     - host:8000/socket.io
   * - X.509 Identity Management
     - host:3000/api/v1
     - host:8000/x509/v1
   * - Kafka WS
     - host:8080/v1/ticket
     - host:8000/kafka-ws/v1/ticket
   * - Kafka WS
     - host:8080/v1/topics
     - host:8000/kafka-ws/v1/topics

     

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

.. _MongoDB doc.: https://docs.mongodb.com/manual/
.. _MongoDB site: https://www.mongodb.com/
.. _PostgreSQL doc.: https://www.postgresql.org/docs/
.. _PostgreSQL site: https://www.postgresql.org
.. _Kong site: https://konghq.com/kong-community-edition/
.. _Kong doc.: https://docs.konghq.com/2.0.x/
.. _Redis site: https://redis.io/
.. _Redis doc.: https://redis.io/documentation
.. _Zookeeper site: https://zookeeper.apache.org/
.. _Zookeeper doc.: https://zookeeper.apache.org/documentation.html
.. _Kafka site: https://kafka.apache.org/
.. _Kafka doc.: http://kafka.apache.org/documentation/
.. _VerneMQ site: https://vernemq.com/
.. _VerneMQ doc.: https://docs.vernemq.com/
.. _Leshan site: https://www.eclipse.org/leshan/
.. _Leshan doc.: https://github.com/eclipse/leshan/wiki

.. _GitHub - auth: https://github.com/dojot/auth/tree/v0.5.0
.. _API - auth: https://dojot.github.io/auth/apiary_v0.5.0.html
.. _Messages - auth: https://github.com/dojot/auth/tree/v0.5.0#kafka-messages

.. _GitHub - Dojot Kong: https://github.com/dojot/kong/tree/v0.5.0

.. _GitHub - history: https://github.com/dojot/history/tree/v0.5.0
.. _API - history: https://dojot.github.io/history/apiary_v0.5.0.html


.. _GitHub - DeviceManager: https://github.com/dojot/device-manager/tree/v0.5.0
.. _API - DeviceManager: https://dojot.github.io/device-manager/apiary_v0.5.0.html
.. _Messages - DeviceManager: https://github.com/dojot/device-manager/tree/v0.5.0#events

.. _GitHub - image-manager: https://github.com/dojot/image-manager/tree/v0.5.0
.. _API - image-manager: https://dojot.github.io/image-manager/apiary_v0.5.0.html


.. _GitHub - GUI: https://github.com/dojot/gui/tree/v0.5.0


.. _GitHub - flowbroker: https://github.com/dojot/flowbroker/tree/v0.5.0
.. _API - flowbroker: https://dojot.github.io/flowbroker/apiary_v0.5.0.html

.. _GitHub - data-broker: https://github.com/dojot/data-broker/tree/v0.5.0
.. _API - data-broker: https://dojot.github.io/data-broker/apiary_v0.5.0.html

.. _Messages - iotagent-mosca: http://dojotdocs.readthedocs.io/projects/iotagent-mosca/en/latest/operation.html#sending-messages-to-other-components-via-kafka
.. _GitHub - iotagent-mosca: https://github.com/dojot/iotagent-mosca/tree/v0.5.0

.. _GitHub - iotagent-vernemq: https://github.com/dojot/dojot/tree/v0.5.0/connector/mqtt/vernemq

.. _GitHub - iotagent-leshan: https://github.com/dojot/iotagent-leshan/tree/v0.5.0


.. _GitHub - Data Manager: https://github.com/dojot/data-manager/tree/v0.5.0
.. _API - Data Manager: https://dojot.github.io/data-manager/apiary_v0.5.0.html

.. _GitHub - Cron: https://github.com/dojot/cron/tree/v0.5.0
.. _API - Cron: https://dojot.github.io/cron/apiary_v0.5.0.html

.. _GitHub - x509-identity-mgmt: https://github.com/dojot/dojot/tree/v0.5.0/x509-identity-mgmt
.. _API - x509-identity-mgmt: https://dojot.github.io/dojot/x509-identity-mgmt/apiary_v0.5.0.html

.. _GitHub - Kafka2Ftp: https://github.com/dojot/dojot/tree/v0.5.0/connector/kafka2ftp

.. _GitHub - Kafka WS: https://github.com/dojot/dojot/tree/v0.5.0/subscription-engine/kafka-ws
.. _API - kafka-ws: https://dojot.github.io/dojot/kafka-ws/apiary_v0.5.0.html

.. _GitHub - V2: https://github.com/dojot/gui-v2
