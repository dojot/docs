Internal communication
======================

This page describes how each service in dojot communicate with each other.


Components
----------

The main components current in dojot are shown in :numref:`dojot_components`.

.. _dojot_components:
.. uml::
    :caption: dojot components
    :align: center


    [Auth]
    [DeviceManager]
    [Persister]
    [History]
    [DataBroker]
    [FlowBroker]
    [x509-identity-mgmt]

    package "Databases" {
      [mongodb]
      [postgreSQL]
    }
    package "IoT agents" {
      [IoT MQTT]
      [IoT LoRa]
      [IoT sigfox]
      [IoT RabbitMQ]
    }

    [postgreSQL] <-- [Auth]
    [postgreSQL] <-- [DeviceManager]
    [postgreSQL] <- [Kong]
    [postgreSQL] <-- [x509-identity-mgmt]
    [mongodb] <- [Persister]
    [mongodb] <-- [FlowBroker]
    [mongodb] <-- [History]
    [mongodb] <-- [x509-identity-mgmt]


They are:

- Auth: authentication mechanism
- DeviceManager: device and template storage.
- Persister: component that stores all device-generated data.
- History: component that exposes all device-generated data.
- DataBroker: deals with subjects and Kafka topics, as well as socket.io
  connections.
- Flowbroker: handles flows (both CRUD and flow execution)
- IoT agents: agents for different protocols.


Each service will be briefly described in this page. More information can be
found in each component documentation.

Messaging and authentication
----------------------------

There are two methods through which dojot components can talk to each other:
via HTTP REST requests and via Kafka. They are intended for different purposes,
though.

HTTP requests can be sent at boot time when a component want, for instance,
information about particular resources, such as list of devices or tenants. For
that, they must know which component has which resource in order to retrieve
them correctly. This means - and this is a very important thing that drives
architectural choices in dojot - that only a single service is responsible for
retrieving data models for a particular resource (note that a service might
have multiple instances, though). For example, DeviceManager is responsible for
storing and retrieving information model for devices and templates, FlowBroker
for flow descriptions, History for historical data, and so on.

Kafka, in the other hand, allows loosely coupled communication between
instances of services. This means that a producer (whoever sends a message)
does not know which components will receive its message. Furthermore, any
consumer doesn't know who generated the message that it being ingested. This
allows data to be transmitted based on "interests": a consumer is interested in
ingesting messages with a particular `subject` (more on that later) and
producers will send messages to all components that are interested in it. Note
that this mechanism allows multiple services to emit messages with the same
"subject", as well as multiple services ingesting messages with the same
"subject" with no tricky workarounds whatsoever.


Sending HTTP requests
+++++++++++++++++++++

In order to send requests via HTTP, a service must create an access token,
described here. There is no further considerations beyond following the API
description associated to each service. This can be seen in figure
:numref:`initial_authentication`. Note that all interactions depicted here are
abstractions of the actual ones. Also, it should be noted that these interactions
are valid only for internal components. Any external service should use Kong
as entrypoint.

.. _initial_authentication:
.. uml::
   :caption: Initial authentication
   :align: center

   actor Client
   boundary Kong
   control Auth

   Client -> Kong: POST /auth \nBody={"admin", "p4ssw0rD"}
   activate Kong
   Kong -> Auth: POST /user \nBody={"admin", "p4ssw0rD"}
   Auth --> Kong: JWT="873927dab"
   Kong --> Client: JWT="873927dab"
   deactivate Kong

In this figure, a client retrieves an access token for user `admin` whose
password is `p4ssw0rd`. After that, a user can send a request to HTTP APIs
using it. This is shown in :numref:`sending_requests`. Note: the actual authorization
mechanism is detailed in `Auth + API gateway (Kong)`_.

.. _sending_requests:
.. uml::
   :caption: Sending messages to HTTP API
   :align: center

   actor Client
   boundary Kong
   control Auth
   control DeviceManager
   database PostgreSQL

   Client -> Kong: POST /device \nHeaders="Authorization: Bearer JWT"\nBody={ device }
   activate Kong
   Kong -> Auth: POST /pep \nBody={"admin", "/device"}
   Auth --> Kong: OK 200
   Kong -> DeviceManager: POST /device \nHeaders="Authorization: JWT" \nBody={ "device" : "XYZ" }
   activate DeviceManager
   DeviceManager -> PostgreSQL: INSERT INTO ....
   PostgreSQL --> DeviceManager: OK
   DeviceManager --> Kong: OK 200
   deactivate DeviceManager
   Kong --> Client: OK 200
   deactivate Kong

In this figure, a client creates a new device using the token retrieved in
:numref:`initial_authentication`. This request is analyzed by Kong, which will
invoke Auth to check whether the user set in the token is allowed to ``POST``
to ``/device`` endpoint. Only after the approval of such request, Kong will
forward it to DeviceManager.


Sending Kafka messages
++++++++++++++++++++++

Kafka uses a quite different approach. Each message should be associated to a
subject and a tenant. This is show in :numref:`retrieving_topics`;

.. _retrieving_topics:
.. uml::
   :caption: Retrieving Kafka topics
   :align: center

   control DeviceManager
   control DataBroker
   database Redis
   control Kafka

   DeviceManager -> DataBroker: GET /topic/dojot.device-manager.devices \nHeaders="Authorization: Bearer JWT"
   note left
     JWT contains the
     service associated
     to the subject
     (admin, for instance).
   end note
   activate DataBroker
   DataBroker -> Redis: GET KEY\n"admin:dojot.device-manager.devices "
   note left
     If the key does
     not exist, then
     it will be
     created.
   end note
   Redis --> DataBroker: 9d0352b7-d195-4852...
   DataBroker -> Redis: GET KEY\n"profile-admin:dojot.device-manager.devices "
   Redis --> DataBroker: { "topic-profile": { ... } }
   DataBroker -> Kafka: CREATE TOPIC \n9d0352b7-d195-4852...\n{ "topic-profile": { ... } }
   note left
     There's no need
     to recreate this
     topic if it is
     already created.
   end note
   Kafka -> DataBroker: OK
   DataBroker --> DeviceManager: { "topic" : "9d0352b7-d195-4852..." }
   deactivate DataBroker
   DeviceManager -> Kafka: SEND MESSAGE\n topic:9d0352b7-d195-4852...\ndata: {"device": "XYZ", "event": "CREATE", ...}
   Kafka --> DeviceManager: OK

In this example, DeviceManager needs to publish a message about a new device.
In order to do so, it sends a request to DataBroker, indicating which tenant
(within JWT token) and which subject (``dojot.device-manager.devices``) it
wants to use to send the message. DataBroker will invoke Redis to check whether
this topic is already created and check whether dojot administrator had created
a profile to this particular tuple ``{tenant, subject}``.

The two profile schemes available are shown in :numref:`automatic_scheme` and
:numref:`assigned_scheme`.

.. _automatic_scheme:
.. uml::
   :caption: Automatic scheme profile
   :align: center

   class IAutoScheme <<interface>> {
     + num_partitions: number;
     + replication_factor: number;
   }

The automatic scheme set the number of Kafka partitions to be used to the topic
being created, as well as the replication factor (how many replicas will be
there for each topic partition). It's up to Kafka to decide which partition and
replica will be assigned to which broker instance. You can check `Kafka
partitions and replicas`_ in order to know a bit more about partition and
replicas. Of course you can check `Kafka's official documentation`_.

.. _assigned_scheme:
.. uml::
   :caption: Assigned scheme profile
   :align: center

   class IAssignedScheme <<interface>> {
     + replica_assignment: Map<number, number[]>;
   }


The assigned scheme indicates which partition will be allocated to which Kafka
instance. This includes also replicas (partitions with more than one associated
Kafka instance).

Bootstrapping tenants
+++++++++++++++++++++

All components are interested in a set of subjects, which will be used to
either send messages or receive messages from Kafka. As dojot groups Kafka
topics and tenants into subjects (a subject will be composed by one or more
Kafka topics, each one transmitting messages for a particular tenant), the
component must bootstrap each tenant before sending or receiving messages. This
is done in two phases: component boot time and component runtime.

In the first phase, a component asks Auth in order to retrieve all currently
configured tenants. It is interested, let's say, in consuming messages from
`device-data` and `dojot.device-manager.devices` subjects. Therefore, it will
request DataBroker a topic for each tenant for each subject. With that list of
topics, it can create Producers and Consumers to send and receives messages
through those topics. This is shown by :numref:`Tenant bootstrapping startup`.

.. _Tenant bootstrapping startup:
.. uml::
   :caption: Tenant bootstrapping at startup
   :align: center

   control Component
   control Auth
   control DataBroker
   control Kafka

   Component -> DataBroker: GET /topic/dojot.tenancy \nHeaders="Authorization: JWT"
   DataBroker --> Component: {"topic" : "eca098e7f..."}
   Component-> Auth: GET /tenants
   Auth --> Component: {"tenants" : ["admin", "tenant1"]}
   loop each tenant
     Component -> DataBroker: GET /topic/device-data \nHeaders="Authorization: JWT[tenant]"
     DataBroker --> Component: {"topic" : "890874987ef..."}
     Component -> Kafka: SUBSCRIBE\ntopic: 890874987ef...
     Kafka --> Component: OK
     Component -> DataBroker: GET /topic/dojot.device-manager.devices \nHeaders="Authorization: JWT[tenant]"
     DataBroker --> Component: {"topic" : "890874987ef..."}
     Component -> Kafka: SUBSCRIBE\ntopic: 890874987ef...
     Kafka --> Component: OK
   end

The second phase starts after startup and its purpose is to process all
messages received through Kafka. This will include any tenant that is created
after all services are up and running. :numref:`Tenant bootstrapping` shows how
to deal with these messages.

.. _Tenant bootstrapping:
.. uml::
   :caption: Tenant bootstrapping
   :align: center


   control Kafka
   control Component
   control DataBroker

   Kafka -> Component: MESSAGE\ntopic:98797ce98af...\nmessage: {"tenant" : "new-tenant"}
   Component -> DataBroker: GET /topic/device-data\nHeaders: "Authorization: Bearer JWT"
   note left
     JWT contains
     new tenant
   end note
   DataBroker --> Component: OK {"topic" : "876ca876g7..."}
   Component -> Kafka: SUBSCRIBE\ntopic: 876ca876g7...
   Kafka --> Component: OK
   Component -> DataBroker: GET /topic/dojot.device-manager.devices\nHeaders: "Authorization: Bearer JWT"
   note left
     JWT contains
     new tenant
   end note
   DataBroker --> Component: OK {"topic" : "22432c4a..."}
   Component -> Kafka: SUBSCRIBE\ntopic: 22432c4a...
   Kafka --> Component: OK


All services that are somehow interested in using subjects should execute this
procedure in order to correctly receive all messages.


Auth + API gateway (Kong)
-------------------------

Auth is a service deeply connected to Kong. It is responsible for user
management, authentication and authorization. As such, it is invoked by Kong
whenever an request is received by one of its registered endpoints. This
section will detail how this is performed and how they work together.

Kong configuration
++++++++++++++++++

There are two configuration procedures when starting Kong within dojot:

#. Migrating existing data
#. Registering API endpoints and plugins.

The first task is performed by simply invoking Kong with a special flag.

The second one is performed by executing a configuration script
`kong.config.sh`. Its only purpose is to register endpoints in Kong, such as:

.. code-block:: bash


    (curl -o /dev/null ${kong}/apis -sS -X POST \
        --header "Content-Type: application/json" \
        -d @- ) <<PAYLOAD
    {
        "name": "data-broker",
        "uris": ["/device/(.*)/latest", "/subscription"],
        "strip_uri": false,
        "upstream_url": "http://data-broker:80"
    }
    PAYLOAD


This command will register the endpoint `/device/*/latest` and `/subscription`
and all requests to it are going to be forwarded to `http//data-broker:80`. You
can check the documentation on how to add endpoints in `Kong's documentation`_.

For some of its registered endpoints, `kong.config.sh` will add two plugins to
selected endpoints:

#. JWT generation. The documentation for this plugin is available at `Kong JWT
   plugin page`_.
#. Configuration a plugin which will forward all policies requests to Auth.
   will invoke Auth in order to authenticate requests. This plugin is available
   in `PEP-Kong repository`_.

The following request install these two plugins in data-broker API:


.. code-block:: bash

  curl -o /dev/null -sS -X POST ${kong}/apis/data-broker/plugins -d "name=jwt"
  curl -o /dev/null -sS -X POST ${kong}/apis/data-broker/plugins -d "name=pepkong" -d "config.pdpUrl=http://auth:5000/pdp"


Emitted messages
****************

Auth will emit just one message via Kafka for tenant creation:

.. code-block:: json

   {
     "type" : "CREATE",
     "tenant" : "XYZ"
   }

Device Manager
--------------

DeviceManager stores and retrieves information models for devices and templates
and a few static information about them as well. Whenever a device is created,
removed or just edited, it will publish a message through Kafka. It depends
only on DataBroker and Kafka for reasons already explained in this document.

All messages published by Device Manager to Kafka can be seen in `Device
Manager messages`_.

IoT agent
---------

IoT agents receive messages from devices and translate them into a default
message to be published to other components. In order to do that, they might
want to know which devices are created in order to properly filter messages
which are not allowed into dojot (using, for instance, security information to
block messages from unauthorized devices). It will use the ``device-data``
subject and bootstrap tenants as described in `Bootstrapping tenants`_.

After requesting the topics for all tenants within `device-data` subject, IoT
agent will start receiving data from devices. As there are a plethora of ways
by which devices can do that, this step won't be detailed in this section (this
is highly dependent on how each IoT agent works). It must, though, send a
message to Kafka to inform other components of all new data that the device
just sent. This is shown in :numref:`IoT agent - kafka`.

.. _IoT agent - kafka:
.. uml::
   :caption: IoT agent message to Kafka
   :align: center

   control Kafka

   IoTAgent -> Kafka: SEND MESSAGE\n topic:890874987ef...\ndata: IoTAgentMessage
   Kafka -> IoTAgent: OK


The data sent by IoT agent has the structure shown in :numref:`IoT agent
message`.

.. _IoT agent message:
.. uml::
   :caption: IoT agent message structure
   :align: center


   class Metadata {
     + deviceid: string
     + tenant: string
     + timestamp: long int
    }

    class IoTAgentMessage {
      + metadata: Metadata
      + attrs: Dict<string, any>
    }

    IoTAgentMessage::metadata -> Metadata

Such message would be:

.. code-block:: json

    {
        "metadata": {
            "deviceid": "c6ea4b",
            "tenant": "admin",
            "timestamp": 1528226137452
        },
        "attrs": {
            "humidity": 60,
            "temperature" : 23
        }
    }



Persister
---------

Persister is a very simple service which only purpose is to receive messages
from devices (using ``device-data`` subject) and store them into MongoDB. For
that, the bootstrapping procedure (detailed in `Bootstrapping tenants`_) is
performed and, whenever a new message is received, it will create a new Mongo
document and store it into the device's collection. This is shown in
:numref:`Persister`.

.. _Persister:
.. uml::
   :caption: Persister
   :align: center

   control Kafka
   control Persister
   database MongoDB

   Kafka -> Persister: MESSAGE\ntopic:98797ce98af...\nmessage: IoTAgentMessage
   Persister -> MongoDB: NEW DOC { IoTAgentMessage }
   MongoDB --> Persister: OK
   Persister --> Kafka: COMMIT

This service is simple as it is by design.

History
-------

History is also a very simple service: whenever a user or application sends a
request to it, it will query MongoDB and build a proper message to send back
to the user/application. This is shown in :numref:`History`.

.. _History:
.. uml::
   :caption: History
   :align: center

   actor User
   boundary Kong
   control History
   database MongoDB

   User -> Kong: GET /device/history/efac?attr=temperature\nHeaders="Authorization: JWT"
   activate Kong
   Kong -> Kong: authorize
   Kong -> History: GET /history/efac?attr=temperature\nHeaders="Authorization: JWT"
   activate History
   History -> MongoDB: db.efac.find({attr=temperature})
   MongoDB --> History: doc1, doc2
   History -> History: processDocs([doc1, doc2])
   History --> Kong: OK\n{"efac":[\n\t{"temperature" : 10},\n\t{"temperature": 20}\n]}
   deactivate History
   Kong -> User: OK\n{"efac":[\n\t{"temperature" : 10},\n\t{"temperature": 20}\n]}
   deactivate Kong

Data Broker
-----------

DataBroker has a few more functionalities than only generating topics for
``{tenant, subject}`` pairs. It will also serve socket.io connections to emit
messages in real time. In order to do so, it retrieves all topics for
`device-data` subject, just as in any other component interested in data
received from devices. As soon as it receives a message, it will then forward
it to a 'room' (using socket.io vocabulary) associated to the device and to the
associated tenant. Thus, all client connected to it (such as graphical user
interfaces) will receive a new message containing all the received data. For
more information about how to open a socket.io connection with DataBroker,
check `DataBroker documentation`_.


Certificate authority
---------------------

The dojot has an internal *Certificate Authority* (`CA`_) capable of issuing
x.509 certificates so that devices can communicate with the platform through
a secure channel (using the TLS protocol).
When requesting a certificate for the platform, it is necessary to inform a
`CSR`_, which will go through a series of validations until arriving at the
internal Certificate Authority, which, in turn, if all checks pass successfully,
will sign a certificate and link this certificate to the device registration.
The ``x509-identity-mgmt`` component is responsible for providing
certificate-related services for devices.

Kafka-WS
---------------------

*Kafka WebSocket* allows the users to retrieve data from a given dojot 
topic in a Kafka cluster, this retrieval can be conditional and/or partial. 
It works with pure websocket connections, so you can create websocket 
clients in any language you want as long as they support RFC 6455.

Connecting to the service
++++++++++++++++++++++++++

The connection is done in two steps, you must first obtain a single-use ticket
through a REST request, then, be authorized to connect to the service through
a websocket.

First step: Get the single-use ticket
*************************************
A ticket allows the user to subscribe to a dojot topic. To obtain it is necessary 
to have a JWT access token that is issued by the platform's Authentication/Authorization 
service. Ticket request must be made by REST at the endpoint <base-url>/kafka-ws/v1/ticket 
using the HTTP GET verb. The request must contain the header Authorization and the
JWT token as value, according to the syntax:

| `POST <base-url>/v1/ticket`
| `Authorization: Bearer [Encoded JWT]`

The component responds with the following syntax:

| `HTTP/1.1 200 OK`
| `Content-type: application/json`
.. code-block:: json

  {
    "ticket": "[an opaque ticket of 64 hexadecimal characters]"
  }

Note: In the context of a dojot deployment the JWT Token is provided by the Auth service, 
and is validated by the API Gateway before redirecting the connection to the Kafka-WS. 
So, no validations are done by the Kafka WS.

Second step: Establish a websocket connection
**********************************************
The connection is done via pure websockets using the URI <base-url>/kafka-ws/v1/topics/:topic.
You must pass the previously generated ticket as a parameter of this URI. It is also possible 
to pass conditional and filter options as parameters of the URI.

Behavior when requesting a ticket and a websocket connection
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

Below we can understand the behavior of the Kafka-WS service when a user
(through a `user agent`_) requests a ticket in order to establish a
communication via websocket with Kafka-WS.

Note that when the user requests a new ticket, Kafka-WS extracts some
information from the *user's access token* (`JWT`_) and generates a
*signed payload*, to be used later in the decision to authorize (or not)
the websocket connection. From the payload a *ticket* is generated and
both are stored in Redis, where the ticket is the key to obtain the payload.
A `TTL`_ is defined by Kafka-WS, so the user has to use the ticket within the
established time, otherwise, Redis automatically deletes the ticket and payload.

After obtaining the ticket, the user makes an HTTP request to Kafka-WS
requesting an upgrade to communicate via *websocket*. As the specification of
this HTTP request limits the use of additional headers, it is necessary to send
the ticket through the URL, so that it can be validated by Kafka-WS before
authorizing the upgrade.

Since the ticket is valid, that is, it corresponds to an entry on Redis,
Kafka-WS retrieves the payload related to the ticket, verifies the integrity
of the payload and deletes that entry on Redis so that the ticket cannot be
used again.

With the payload it is possible to make the decision to authorize the upgrade
to websocket or not. If authorization is granted, Kafka-WS opens a subscription
channel based on a specific topic in Kafka. From there, the upgrade to websocket
is established and the user starts to receive data as they are being published
in Kafka.

.. uml::
    :caption: Obtaining a ticket and connecting via websocket
    :align: center

    actor User
    boundary Kong
    control "Kafka-WS"
    database Redis
    control Kafka

    group Get Ticket
        User -> Kong: GET /kafka-ws/v1/ticket\nHeaders="Authorization: JWT"
        Kong -> Kong: Checks JWT
        Kong -> "Kafka-WS" : Request a ticket
        "Kafka-WS" -> "Kafka-WS" : Sign the payload and\ngenerate a ticket for it
        "Kafka-WS" -> Redis : Register the ticket and\npayload with a TTL
        "Kafka-WS"<-- Redis : Sucess
        User <-- "Kafka-WS" : Returns the newly generated ticket
    end

    group Connect via websocket
        User -> Kong: Upgrade HTTP to websocket\n(ticket in the URL)
        Kong -> "Kafka-WS" : Forward the ticket
        "Kafka-WS" -> Redis : Recovers payload (if any)
        "Kafka-WS"<-- Redis : Payload found
        "Kafka-WS" -> "Kafka-WS" : Checks the payload
        "Kafka-WS" -> Kafka : Subscrive to kafka topic\n(Using the payload)
        "Kafka-WS" <-- Kafka : Sucess
        User <-- "Kafka-WS" : Upgrade to websocket accepted\nConnected!
        "Kafka-WS" <-- Kafka : New data in the topic
        User <-- "Kafka-WS" : Returns data
        "Kafka-WS" <-- Kafka : [...]
        User <-- "Kafka-WS" : [...]
        "Kafka-WS" <-- Kafka : [...]
        User <-- "Kafka-WS" : [...]
    end



.. _API - data-broker: https://dojot.github.io/data-broker/apiary_latest.html
.. _Kafka partitions and replicas: https://sookocheff.com/post/kafka/kafka-in-a-nutshell/#what-is-kafka
.. _DataBroker documentation: https://dojot.github.io/data-broker/apiary_latest.html
.. _Device Manager messages: https://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/kafka-messages.html
.. _CA: https://en.wikipedia.org/wiki/Certificate_authority
.. _CSR: https://en.wikipedia.org/wiki/Certificate_signing_request
.. _user agent: https://en.wikipedia.org/wiki/User_agent
.. _TTL: https://en.wikipedia.org/wiki/Time_to_live
.. _JWT: https://en.wikipedia.org/wiki/JSON_Web_Token
.. _Kafka's official documentation: https://kafka.apache.org/documentation/
.. _Kong's documentation: https://docs.konghq.com/0.14.x/getting-started/configuring-a-service/
.. _Kong JWT plugin page: https://docs.konghq.com/hub/kong-inc/jwt/
.. _PEP-Kong repository: https://github.com/dojot/pep-kong
