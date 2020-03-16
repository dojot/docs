.. _Components and APIs:

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
  * - Auth
    - `GitHub - auth`_
    - `Auth  doc.`_
    - `API - auth`_
  * - History
    - `GitHub - history`_
    -
    - `API - history`_
  * - Device Manager
    - `GitHub - DeviceManager`_
    - `DeviceManager doc.`_
    - `API - DeviceManager`_
  * - Image Manager
    - `GitHub - image-manager`_
    -
    - `API - image-manager`_
  * - GUI
    - `GitHub - GUI`_
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
  * - | Iotagent Mosca
      | (MQTT)
    - `GitHub - iotagent-mosca`_
    -
    -
  * - EJBCA-REST
    - `GitHub - EJBCA-REST`_
    -
    - `API - EJBCA-REST`_
  * - Data Manager
    - `GitHub - Data Manager`_
    -
    - `API - Data Manager`_
  * - Cron
    - `GitHub - Cron`_
    -
    - `API - Cron`_



Exposed APIs (API Gateway)
--------------------------


The API gateway used in dojot reroutes some of endpoints from component.
The following table shows which **endpoint exposed
by the API gateway** is mapped to which **component endpoint**,
its  **component endpoint Documentation** and
whether the endpoint **needs authentication** when used via API Gateway.
See more about how using API in :ref:`Using API interface`.

.. list-table:: Exposed endpoints
   :header-rows: 1

   * - | Component
     - | Endpoint exposed
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
   * - Device Manager
     - /device
     - /device
     - `See more <https://dojot.github.io/device-manager/apiary_latest.html#devices>`_
     - Yes
   * - Device Manager
     - /template
     - /template
     - `See more <https://dojot.github.io/device-manager/apiary_latest.html#templates>`_
     - Yes
   * - Flowbroker
     - **/flows**
     - /
     - `See more <https://dojot.github.io/flowbroker/apiary_latest.html#flows>`_
     - Yes
   * - Auth
     - **/auth**
     - /
     - `See more <https://dojot.github.io/auth/apiary_latest.html#auth-session-management-post>`_
     - No
   * - Auth
     - **/auth**/revoke
     - /revoke
     -
     - No
   * - Auth
     - **/auth**/user
     - /user
     - `See more <https://dojot.github.io/auth/apiary_latest.html#auth-known-users-manipulation-get>`_
     - Yes
   * - Auth
     - **/auth**/pap
     - /pap
     - `See more <https://dojot.github.io/auth/apiary_latest.html#crud-permissions-and-group>`_
     - Yes
   * - History
     - **/history**
     - /
     - `See more <https://dojot.github.io/history/apiary_latest.html>`_
     - Yes
   * - EJBCA REST
     - /sign
     - /sign
     - `See more <https://dojot.github.io/ejbca-rest/apiary_teste_release.html#ejbca-user-management-sign-a-certificate-for-a-user-post>`_
     - Yes
   * - EJBCA REST
     - /ca
     - /ca
     - `See more <https://dojot.github.io/ejbca-rest/apiary_latest.html#ejbca-ca-management>`_
     - Yes
   * - EJBCA REST
     - /user
     - /user
     - `See more <https://dojot.github.io/ejbca-rest/apiary_latest.html#ejbca-user-management-user-crud>`_
     - Yes
   * - Data Manager
     - /import
     - /import
     - `See more <https://dojot.github.io/data-manager/apiary_latest.html#import-post>`_
     - Yes
   * - Data Manager
     - /export
     - /export
     - `See more <https://dojot.github.io/data-manager/apiary_latest.html#export-get>`_
     - Yes
   * - Cron
     - /cron
     - /cron
     - `See more <https://dojot.github.io/cron/apiary_latest.html>`_
     - Yes
   * - Image Manager
     - **/fw-image**
     - /
     - `See more <https://dojot.github.io/image-manager/apiary_latest.html>`_
     - Yes
   * - Data Broker
     - | /device/
       | {deviceID}
       | /latest
     - | /device/
       | {deviceID}
       | /latest
     -
     - Yes
   * - Data Broker
     - /subscription
     - /subscription
     -
     - Yes
   * - Data Broker
     - /stream
     - /stream
     -
     - Yes
   * - Data Broker
     - /socket.io
     - /socket.io
     - `See more <https://dojot.github.io/data-broker/apiary_latest.html#websockets-socket-io-based-realtime-events-get>`_
     - No

**NOTE: Some of the endpoints from component aren't exposed, but are used internally.**


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
     - host:5000/auth/revoke
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
   * - EJBCA REST
     - host:5583/sign
     - host:8000/sign
   * - EJBCA REST
     - host:5583/ca
     - host:8000/ca
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
.. _Kong doc.: https://getkong.org/docs/
.. _Redis site: https://redis.io/
.. _Redis doc.: https://redis.io/documentation
.. _Zookeeper site: https://zookeeper.apache.org/
.. _Zookeeper doc.: https://zookeeper.apache.org/documentation.html
.. _Kafka site: https://kafka.apache.org/
.. _Kafka doc.: http://kafka.apache.org/documentation/


.. _GitHub - auth: https://github.com/dojot/auth
.. _API - auth: https://dojot.github.io/auth/apiary_latest.html
.. _Auth  doc.: http://dojotdocs.readthedocs.io/projects/auth/en/latest/
.. _Messages - auth: https://dojotdocs.readthedocs.io/projects/auth/en/latest/kafka-messages.html

.. _GitHub - history: https://github.com/dojot/history
.. _API - history: https://dojot.github.io/history/apiary_latest.html


.. _GitHub - DeviceManager: https://github.com/dojot/device-manager
.. _API - DeviceManager: https://dojot.github.io/device-manager/apiary_latest.html
.. _DeviceManager doc.: http://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/
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

.. _GitHub - EJBCA-REST: https://github.com/dojot/ejbca-rest
.. _API - EJBCA-REST: https://dojot.github.io/ejbca-rest/apiary_latest.html

.. _GitHub - Data Manager: https://github.com/dojot/data-manager
.. _API - Data Manager: https://dojot.github.io/data-manager/apiary_latest.html

.. _GitHub - Cron: https://github.com/dojot/cron
.. _API - Cron: https://dojot.github.io/cron/apiary_latest.html

