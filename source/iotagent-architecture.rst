**********************
IoT Agent architecture
**********************

This document describes the IoT agent architecture used by dojot. It defines a
set of basic features and choices that must be followed in order to be aligned
with dojot architecture.

TODO
====
- Sanitize "device" and "physical device" terms. Sometimes they are used interchangeably.
- Build a list of terms (so that I can just say "blingblong" meaning "physical devices or its representative elements)
- Find a better term for "representative elements"
- Fill the Device identity section
- Select what SHOULD be done, what MAY be implemented or SHALL be presented

Who should read this?
=====================
Developers that want to create new IoT agents to be used with dojot.

Introduction
============

Using dojot involves dealing with the following entities:

- **physical devices**: devices that sends messages to IoT agents. They
  might have sensors and might be configurable, but this is not mandatory.
  Also, they must have some kind of connectivity to other services so that they
  can send their readings to these services.
- **users**: whoever sends requests to dojot in order to manage resources,
  retrieve historical device data, create subscriptions, manage flows, and so
  on.
- **profiles**: //TODO
- **tenants**: logical separation between resources that might be associated
  with multiple users.
- **resources**: elements that are associated to a particular entity. They are:

   - *devices*: representation of a element which has attributes. This element
     can be a physical device or a virtual one - one that doesn't receive
     attribute updates directly by a device.
   - *templates*: device blueprints that contain a list of attributes
     associated to that class of devices. All devices are created based on a
     template, from which it will inherit attributes.
   - *topics*: Kafka communication channels that are used to send and receive
     messages between dojot services.
   - *flows*: Sequence of processing blocks that are created by a user or an
     application and are used to analyze and preprocess data.
- **subjects**: group of topics that share a common message flow. For instance,
  there might be many topics that are used to transmit device data. All of them
  belong to the same subject "device-data".


When a new IoT agent is created, all these entities must be taken into account
in a coordinated way. This document lists all basic requirements for a new IoT
agent and they are categorized in the following groups:

#. **Device security**: IoT agents must be able to check whether a device
   connection is valid or not. A valid device connection is defined as one
   originated by a trusted physical device (or any representative element, such
   as gateways) which is allowed to connect to the IoT agent. A device is
   deemed as trusted by: (1) creating a device associated with it (which may
   include security information such as cryptographic keys) or (2) indicating
   directly to IoT agent that a device or a representative element is allowed
   to connect to it (so that elements that serves as relay connections can be
   properly and securely used).
#. **Information context separation**: each resource (device, templates, topics
   and flows) is associated to a particular tenant and entities that don't
   belong to that tenant must not be allowed to access its resources. This is
   valid throughout dojot and it is no exception for IoT agents. Therefore, an
   IoT agent must treat separately all devices that belong to different tenants
   - including the fact that no one from one tenant should be able to know of
   the existence of other tenants. For instance, a MQTT IoT agent should not
   allow messages sent to its broker from devices associated to tenant A to be
   published to devices subscribed to the same topic belonging to tenant B.
#. **IoT agent information and management**: any IoT agent should publish its
   capabilities and information models. For instance, it should let other
   services know about what is the device template which it accepts in order to
   properly receive and send messages to a particular physical device. It
   should also offer a management interface so that a user can change and
   retrieve its behavior, such as logging options, statistics, quotas and so
   on.
#. **IoT agent operation**: IoT agents must be able to receive and send
   messages (if allowed by the protocol) to devices and, therefore, send
   updates to other dojot services based on received device messages. All
   messages received from a particular device and sent to other dojot services
   must be sent in the same order as it was received. IoT agents should also be
   able to enable or disable message processing from a particular device and
   detect device liveness.

   An extra feature that an IoT agent might implement is firmware updates.
   Depending on is underlying protocol, it might be possible to do such thing
   in a easy, secure and reliable way.

Each one of these groups is going to be detailed in the following sections.

Device security
===============

An IoT manager should take into account the following aspects of device
communication:

#. Device identity: it should only accept connections from authorized physical
   devices. The verification of whether a new connection was originated by an
   authorized device (which includes verifying whether a particular device is
   authorized or not) should rely on public keys and/or signed certificates.
#. Communication channel security: all messages exchanged with a physical
   device should be encrypted using well-known cryptographic standards, such as
   TLS. Any in-house security protocols should be avoided.
#. Certificate revocation: the IoT agent should be able to discard any messages
   from previously authorized device if its security data has been somehow
   compromised. For instance, if the private key associated to a particular
   device is leaked, then all its messages should be ignored as there is no
   guarantee that they came from that device.

Each of these aspects will be detailed in the following sections.

Device identity
---------------

The device identity verification is the starting point when dealing with
communication security. This validation will indicate to the IoT agent if the
device that opened the connection is whoever it says it is. Furthermore, the
IoT agent must, once this validation succeeds, check whether this device can
connect to it by checking its ID. This section will show how to do that.

For connection-oriented protocols, the IoT agent should only accept connections
for devices that have a certificate that was signed by an authority that is
trusted by dojot. Once this certificate is valid, device identity can be
checked in two forms:

- Device ID encoded in certificate: although this is a less-reliable mechanism,
  it allows greater flexibility using many devices in a controlled deployment.
  This is based on setting the common name (CN certificate field) as dojot
  device ID. Therefore, IoT agent should check whether this device exists or
  not and allow or deny the connection right away depending on this
  verification. The weak points of this mechanisms is that the device
  certificate must be signed by dojot's internal CA (once there is a procedure
  to sign only one certificate per device) and, if this certificate is valid,
  then its ID must also be valid. If any other CA is used, then this mechanism
  has no valid use.

- IoT agent has all valid certificates: if an administrator wants to use an
  external CA to sign all device certificates, then there is no actual control
  of which device ID was used to generate a particular certificate. Therefore,
  IoT agent must have all valid certificates properly mapped onto a device list
  - this will guarantee that only one certificate is allowed to a particular
  device and vice-versa.

Using the first mechanism, the device (or an operator configuring a device for
the first time) must call dojot CA to generate a signed certificate for itself.
There is no further action for IoT agent to take as long as dojot CA is used.

The second mechanism, however, requires that an IoT agent offer methods to
manage certificates. The developer must take into account also that this IoT
agent must be able to scale - these certificates must be accessible to all IoT
agent instances, if allowed by deployment.

Communication security
----------------------

With a valid certificate, a device can create a communication channel with
dojot. For connection-oriented channels, this certificate should be used
alongside cryptographic keys in order to provide an encrypted channel. For
other channel types (such as channels for exchanging messages through a
gateway, such as LoRa or sigfox), it suffice to be sure that the connection
between dojot and the backend server is secure. The backend identity should be
asserted beforehand. Once it is known to be trusted, all its messages can be
processed with no major concern.

Certificate revocation
----------------------

An IoT agent should be able to be informed about revoked certificates. It
should expose an API or configuration messages to allow such thing. It should
not allow any communication with a particular device that uses a revoked
certificate.

Information context separation
==============================

A tenant could be thought simply as a group of users that share some resources.
But its meaning might go beyond that - it might implies that these resources
would not share any common infrastructure (considering anything that transmits,
processes or stores data) with resources belonging to other tenants. One might
want to have separate software instances to process data from different tenants
so that processing data from one tenant will not affect processing data from
the other, achieving a higher level of context separation.

Although this is desirable, some deployment scenarios might force using some of
the same infrastructure for different tenants (for instance, when the
deployment has as reduced numbers of processing units or network connections).
So, in order to have a minimum context separation among tenants, an IoT agent
should use everything it can to separate them, such as using different threads,
queues, sockets, etc., and should not rely solely in deployment scenarios
features (such as different IoT agents for different tenants). For instance,
for topic based protocols, such as MQTT, one might want to force different
topics for different tenants. Should a device publish data to a particular
topic that is owned by other tenant, this message is ignored or blocked
(sending an error back to the device might be an optional behavior). Therefore
no device from one tenant can send messages to any device from other tenant.

The mechanism through which context separation is implemented highly depends on
which protocol is used. A thorough analysis should be performed to properly
implement this feature.

IoT agent information and management
====================================

An IoT agent should expose all the necessary information to use it properly. It
should expose:

- **Device template**: an IoT agent should publish which is the data model it
  accepts for a valid device. This should be done by publishing a new device
  template to other dojot services. There should be a mechanism so that
  different instances of the same IoT agent publishes the same device template
  (including any template IDs). If the device template is updated in a newer
  version of an IoT agent, the device template ID should change. The messages
  used to publish its device template is detailed in `Device Template message`_
  section.
- **Management APIs**: an IoT agent should be manageable and should expose its
  APIs to do that. The minimum set of management APIs that an IoT agent should
  offer are:

  - *Logging*: there should be a way to change the log level of an IoT agent;
  - *Statistics*: an IoT agent may expose an API to let a user or application
    retrieve statistical information about its execution. An administrator
    might want to switch on or off the generation of a particular statistical
    variable, such as processing time.

An IoT agent should also be able to gather statistics information related to
its execution. Furthermore, it should let an administrator set quotas on those
measured quantities. These quantities might include, but are not limited to:

- transmission statistics

  - number of received device messages from device (total, per device, per
    tenant)
  - number of published device messages to dojot (total, per device, per
    tenant)
  - number of messages sent to devices (total, per device, per tenant)
  - [optional] time taken between receiving a message from a physical device
    and publishing it (total - mean, per device - mean, per tenant - mean)

- IoT agent service health check
  - system statistics (memory, disk, etc.) used by the service

Many other values might be gathered. The list above is the minimum list that an
IoT agent is expected to expose to other services. Particularly for health
check, there is a document detailing how expose it.

IoT agent operation
===================

The main purpose of an IoT agent is to publish data from a particular device to
other dojot services. Its operation is two fold: receive and process messages
related to device management from other services as well as receive messages
from the devices themselves (or their representative elements) and publish
these data to other services. ~~This is shown in the figure below.~~ ??

//TODO
<< a figure should be here >>

The following sections describe how an IoT agent can send and receive messages
to/from other dojot services and what are the considerations it must take into
account when receiving messages from physical devices.

Messages
--------
//Aqui ele explica muita coisas q as libs ja fornecem, é interessante estar aqui mesmo?
//Nao deveria estar por exemplo no readme do databroker? Ou algo assim?

At start, all IoT agents (in fact, all services that need to receive or send
messages related to devices) must know the list of configured tenants. This is
the most basic piece of information that IoT agent needs to know in order to
work properly. The request that should be sent to Auth service is this (all
requests sent from dojot services to its own services should use the
"dojot-management" user):


+----------------------------------------------------------+
| Host: Auth                                               |
+========================+=+===============================+
| Endpoint: /admin/tenants | Method: GET                   |
+--------------------------+-------------------------------+
|                       Request                            |
+--------------------------+-------------------------------+
| Headers                  | Authorization: Bearer ${JWT}  |
+--------------------------+-------------------------------+
| Response                                                 |
+--------------------------+-------------------------------+
| Headers                  | Content-Type: application/json|
+--------------------------+-------------------------------+
|                          | ::                            |
|                          |                               |
|                          |   tenants => *tenant          |
| Body format              |     tenant => string          |
+--------------------------+-------------------------------+


A sample response for this request is:

.. code-block:: json

    {
      "tenants": [
        "admin",
        "users",
        "system"
      ]
    }

With this list, the IoT agent can request topics for receiving device and
tenant lifecycle events and for publishing new device attribute data. This is
done by sending the following request to DataBroker:

+-------------------------------------------------------------+
|                       Host: DataBroker                      |
+============================+================================+
| Endpoint: /topic/{subject} |           Method: GET          |
+----------------------------+--------------------------------+
|                           Request                           |
+----------------------------+--------------------------------+
|           Headers          |  Authorization: Bearer ${JWT}  |
+----------------------------+--------------------------------+
|                           Response                          |
+----------------------------+--------------------------------+
|           Headers          | Content-Type: application/json |
+----------------------------+--------------------------------+
|         Body format        | ::                             |
|                            |                                |
|                            |   topic => string              |
+----------------------------+--------------------------------+


A sample response for this request is:

.. code-block:: json

    {
      "topic": "c9b2c688-9e40-4032-877a-3d262acba9d0"
    }

Some subjects are "tenant-sensitive" (a different topic will be returned for
different tenants) and some are not (the same topic will be returned regardless
the tenant). DataBroker will use the tenant contained in the authorization
token when dealing with tenant-sensitive subjects.

The following subjects should be used by IoT agents:

- dojot.tenancy
- dojot.device-manager.device-template
- dojot.device-manager.device
- device-data

Each one will be detailed in the following sections

dojot.tenancy
^^^^^^^^^^^^^

The topic related to this subject will be used to receive tenant lifecycle
events. Whenever a new tenant is created or delete, the following message will
be published:

+---------------------------------------------------+
| Subject: dojot.tenancy                            |
+------------------------+--------------------------+
| Body format (JSON)     |                          |
|                        | ::                       |
|                        |                          |
|                        |   type="CREATE"/"DELETE" |
|                        |   tenant=>string         |
+------------------------+--------------------------+

This subject is not tenant-sensitive.
A sample message received by this topic is:

.. code-block:: json

    {
      "type": "CREATE",
      "tenant": "new_tenant"
    }

dojot.device-manager.device-template
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

+------------------------------------------------------+
| Subject: dojot.device-manager.device-template        |
+---------------------+--------------------------------+
| Body format (JSON)  |                                |
|                     | ::                             |
|                     |                                |
|                     |   event => "create"            |
|                     |   data => id label attrs       |
|                     |     id => string               |
|                     |     label => string            |
|                     |     attrs => [*template_attrs] |
+---------------------+--------------------------------+


dojot.device-manager.devices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

the topic related to this subject will be used to receive device lifecycle
events for a particular tenant. Its format is:

+-----------------------------------------------------------------+
| Subject: dojot.device-manager.device                            |
+====================+============================================+
| Body format (JSON) |                                            |
|                    | ::                                         |
|                    |                                            |
|                    |                                            |
|                    |   event => "create" / "update"             |
|                    |   meta => service                          |
|                    |     service => string                      |
|                    |   data => id label templates attrs created |
|                    |     id => string                           |
|                    |     label => string                        |
|                    |     templates => *number                   |
|                    |     attrs => [*template_attrs]             |
|                    |     created => iso_date                    |
+--------------------+--------------------------------------------+
| Body format (JSON) |                                            |
|                    | ::                                         |
|                    |                                            |
|                    |   event => "remove"                        |
|                    |   meta => service                          |
|                    |     service => string                      |
|                    |   data => id                               |
|                    |     id => string                           |
+--------------------+--------------------------------------------+
| Body format (JSON) |                                            |
|                    | ::                                         |
|                    |                                            |
|                    |   event => "actuate"                       |
|                    |   meta => service                          |
|                    |     service => string                      |
|                    |   data => id                               |
|                    |     id => string                           |
|                    |     attrs => *device_attrs                 |
+--------------------+--------------------------------------------+

The template_attrs is a simple key/value JSON with template ID as key and the
following structure as value:


.. code-block:: json

    {
      "template_id": "1",
      "created": "2018-01-05T15:41:54.840116+00:00",
      "label": "this-is-a-sample-attribute",
      "value_type": "float",
      "type": "dynamic",
      "id": 1
    }

The device_attrs attribute is a even simpler key/value JSON, such as:

.. code-block:: json

    {
      "temperature" : 10,
      "height" : 280
    }

This subject is tenant-sensitive.

A sample message received by this topic is:

.. code-block:: json

    {
      "event": "create",
      "meta": {
        "service": "admin"
      },
      "data": {
        "id": "efac",
        "label": "Device 1",
        "templates": [1, 2, 3],
        "attrs": {
          "1": [
            {
              "template_id": "1",
              "created": "2018-01-05T15:41:54.840116+00:00",
              "label": "this-is-a-sample-attribute",
              "value_type": "float",
              "type": "dynamic",
              "id": 1
            }
          ]
        },
        "created": "2018-02-06T10:43:40.890330+00:00"
      }
    }

device-data
^^^^^^^^^^^

The topic related to this subject will be used to publish data retrieved from a
physical device to other dojot services. Its format is:


+------------------------------------------------------------------------+
| Subject: device-data                                                   |
+--------------------+---------------------------------------------------+
| Body format (JSON) |                                                   |
|                    | ::                                                |
|                    |                                                   |
|                    |   metadata => deviceid tenant timestamp recv_time |
|                    |     deviceid => string                            |
|                    |     tenant => string                              |
|                    |     timestamp => unix_timestamp                   |
|                    |     recv_time => unix_timestamp                   |
|                    |   attrs => *device_attrs                          |
+--------------------+---------------------------------------------------+

This subject is tenant-sensitive. The timestamp is associated to when the
attribute values were gathered by the device (this could be done by the device
itself or by the IoT agent, if no timestamp was defined by the device). The
recv_time attribute indicates when the message was received.

A sample message received by this topic is:

.. code-block:: json

    {
      "metadata": {
        "deviceid": "c6ea4b",
        "tenant": "admin",
        "timestamp": 1528226137452,
        "recv_time": 1528226137462
      },
      "attrs": {
        "humidity": 60
      }
    }

Firmware update
---------------

An IoT agent might implement mechanisms in order to update firmware in devices.

The firmware update process should be:
//TODO
-


Behavior
========

The order in which a physical device sends its attributes must not be changed
when IoT agent publishes these data to other dojot services.

If the protocol imposes any unique ID to each device, the IoT agent must build
a correlation table to properly translate this unique ID into dojot device ID
and vice-versa.



