User Guide
==========

This document provides information on how to use dojot from a device developer
or application developer point of view. For documentation regarding the
operation of the platform itself, please refer to the :doc:`ops_guide`.

.. contents:: Table of Contents
  :local:

Who should read this
--------------------

- Users that want a deeper look at how dojot works;
- Application developers.


Getting Started
---------------

To start, please follow dojot's installation guide in
:doc:`install/compose_guide`. There you should find how to properly download a
working copy of the components, how to minimally configure them, how to start
them up and how to check whether they are working.

dojot basics
------------

Before using dojot, you should be familiar with some basic operations and
concepts. They are very simple to understand and use, but without them, all
operations might become obscure and senseless.

In the next section, there is an explanation of a few basic entities in dojot:
devices, templates and flows. With these concepts in mind, we present a small
tutorial to how to use them in dojot - it only covers API access. For a guided
tour on how to use the web interface, check dojot's `YouTube channel`_.

If you want more information on how dojot works internally, you should checkout
the :doc:`architecture` to get acquainted with all internal components.

User authentication
*******************

All HTTP requests supported by dojot are sent to the API gateway. In order to
control which user should access which endpoints and resources, dojot makes
uses of `JSON Web Token`_ (a useful tool is `jwt.io`_) which encodes things
like (not limited to these):

- User identity
- Validation data
- Token expiration date

The component responsible for user authentication is `auth`_. You can find a
tutorial of how to authenticate a user and how to get an access token in `auth
documentation`_.


Devices and templates
*********************

In dojot, a device is a digital representation of an actual device or gateway
with one or more sensors or of a virtual one with sensors/attributes inferred
from other devices. Throughout the documentation, this kind of device will be
called simply as 'device'. If the actual device must be referenced, we'll be
calling it as 'physical device'.

Consider, for instance, a physical device with temperature and humidity
sensors; it can be represented in dojot as a device with two attributes (one
for each sensor). We call this kind of device as regular device or by its
communication protocol, for instance, MQTT device or CoAP device.

We can also create devices which don’t directly correspond to their physical
counterparts, for instance, we can create one with higher level of information
of temperature (is becoming hotter or is becoming colder) whose values are
inferred from temperature sensors of other devices. This kind of device is
called virtual device.

All devices are created based on a *template*, which can be thought as a model
of a device. As "model" we could think of part numbers or product models - one
*prototype* from which devices are created. Templates in dojot have one label
(any alphanumeric sequence), a list of attributes which will hold all the
device emitted information, and optionally a few special attributes which will
indicate how the device communicates, including transmission methods (protocol,
ports, etc.) and message formats.

In fact, templates can represent not only "device models", but it can also
abstract a "class of devices". For instance, we could have one template to
represent all thermometers that will be used in dojot. This template would have
only one attribute called, let's say, "temperature". While creating the device,
the user would select its "physical template", let's say *TexasInstr882*, and
the 'thermometer' template. The user would have also to add translation
instructions in order to map the temperature reading that will be sent from the
device to a "temperature" attribute.

In order to create a device, a user selects which templates are going to
compose this new device. All their attributes are merged together and
associated to it - they are tightly linked to the original template so that any
template update will reflect all associated devices.

The component responsible for managing devices (both real and virtual) and
templates is `DeviceManager`_. `DeviceManager documentation`_ explains in more
depth all the available operations.


Flows
*****

A flow is a sequence of blocks that process a particular event or device
message. It contains:

- entry point: a block representing what is the trigger to start a particular
  flow;
- processing blocks: a set of blocks that perform operations using the event.
  These blocks may or may not use the contents of such event to further process
  it. The operations might be: testing content for particular values or ranges,
  geo-positioning analysis, changing message attributes, perform operations on
  external elements, and so on.
- exit point: a block representing where the resulting data should be forwarded
  to. This block might be a database, a virtual device, an external element,
  and so on.

The component responsible for dealing with such flows is `flowbroker`_.

Step-by-step device management
******************************

This section provides a complete step-by-step tutorial of how to create,
update, send messages to and check historical data of a device. We will create
a simple device with only one attribute, send a few messages emulating the
physical device and check the historical data for the only attribute this
device has.

Also, this tutorial assumes that you are using `docker-compose`_, which has all
the necessary components to properly run dojot (so all API requests will be
sent to localhost:8000).

Getting access token
++++++++++++++++++++

As said in `User authentication`_, all requests must contain a valid access
token. You can generate a new token by sending the following request:

.. code-block:: bash

  curl -X POST http://localhost:8000/auth \
         -H 'Content-Type:application/json' \
         -d '{"username": "admin", "passwd" : "admin"}'

  {"jwt": "eyJ0eXAiOiJKV1QiL..."}

If you want to generate a token for other user, just change the username and
password in the request payload. The token ("eyJ0eXAiOiJKV1QiL...") should be
used in every HTTP request sent to dojot in a special header. Such request
would look like:

.. code-block:: bash

   curl -X GET http://localhost:8000/device \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiL..."

Remember that the token must be set in the request header as a whole, not parts
of it. In the example only the first characters are shown for the sake of
simplicity. All further requests will use a bash variable called ``bash
${JWT}``, which contains the token got from auth component.


Device creation
+++++++++++++++

In order to properly configure a physical device in dojot, you must first
create its representation in the platform. The example presented here is just a
small part of what is offered by DeviceManager. For more information, check the
`DeviceManager how-to`_ for more detailed instructions.

First of all, let's create a template for the device - all devices are based
off of a template, remember.

.. code-block:: bash

    curl -X POST http://localhost:8000/template \
    -H "Authorization: Bearer ${JWT}" \
    -H 'Content-Type:application/json' \
    -d ' {
      "label": "Thermometer Template",
      "attrs": [
        {
          "label": "temperature",
          "type": "dynamic",
          "value_type": "float"
        }
      ]
    }'

This request should give back this message:


.. code-block:: json
   :linenos:

    {
      "result": "ok",
      "template": {
        "created": "2018-01-25T12:30:42.164695+00:00",
        "data_attrs": [
          {
            "template_id": "1",
            "created": "2018-01-25T12:30:42.167126+00:00",
            "label": "temperature",
            "value_type": "float",
            "type": "dynamic",
            "id": 1
          }
        ],
        "label": "Thermometer Template",
        "config_attrs": [],
        "attrs": [
          {
            "template_id": "1",
            "created": "2018-01-25T12:30:42.167126+00:00",
            "label": "temperature",
            "value_type": "float",
            "type": "dynamic",
            "id": 1
          }
        ],
        "id": 1
      }
    }

Note that the template ID is 1 (line 27).

To create a template based on it, send the following request to dojot:

.. code-block:: bash
    :linenos:

    curl -X POST http://localhost:8000/device \
    -H "Authorization: Bearer ${JWT}" \
    -H 'Content-Type:application/json' \
    -d ' {
      "templates": [
        "1"
      ],
      "label": "device"
    }'


The template ID list on line 6 contains the only template ID configured so far.
To check out the configured device, just send a GET request to /device:

.. code-block:: bash

    curl -X GET http://localhost:8000/device -H "Authorization: Bearer ${JWT}"


Which should give back:

.. code-block:: bash

    {
      "pagination": {
        "has_next": false,
        "next_page": null,
        "total": 1,
        "page": 1
      },
      "devices": [
        {
          "templates": [
            1
          ],
          "created": "2018-01-25T12:36:29.353958+00:00",
          "attrs": {
            "1": [
              {
                "template_id": "1",
                "created": "2018-01-25T12:30:42.167126+00:00",
                "label": "temperature",
                "value_type": "float",
                "type": "dynamic",
                "id": 1
              }
            ]
          },
          "id": "0998",
          "label": "device_0"
        }
      ]
    }


Sending messages
++++++++++++++++

So far we got an access token and created a template and a device based on it.
In an actual deployment, the physical device would send messages to dojot with
all its attributes and their current values. For this tutorial we will send
MQTT messages by hand to the platform, emulating such physical device. For
that, we will use mosquitto_pub from Mosquitto project.

.. ATTENTION::
    Some Linux distributions, Ubuntu in particular, have two packages for
    `mosquitto`_ - one containing tools to access it (i.e. mosquitto_pub and
    mosquitto_sub for publishing messages and subscribing to topics) and
    another one containing the MQTT broker. In this tutorial, only the tools
    are going to be used. Please check if MQTT broker is not running before
    starting dojot (by running commands like ``ps aux | grep mosquitto``).


The dojot compatible format for messages sent by devices is a simple key-value
JSON, such as:

.. code-block:: json

    {
      "temperature" : 10.6
    }

Let's send this message to dojot:

.. code-block:: bash

  mosquitto_pub -t /admin/0998/attrs -m '{"temperature": 10.6}'

If there is no output, the message was sent to MQTT broker. The topic is build
from the following information:

- admin: user tenant. This is retrieved from "service" attribute from user
  configuration.

- 0998: device ID. This is retrieved from the device itself. It is returned
  when the device is created or read from /device endpoint.

This topic scheme is customizable, depending on device configuration.

For more information on how dojot deals with data sent from devices, check the
`Integrating physical devices`_ section.

Checking historical data
++++++++++++++++++++++++

In order to check all values that were sent from a device for a particular
attribute, you could use the `history APIs`_. Let's first send a few other
values to dojot so we can get a few more interesting results:


.. code-block:: bash

  mosquitto_pub -t /admin/3bb9/attrs -m '{"temperature": 36.5}'
  mosquitto_pub -t /admin/3bb9/attrs -m '{"temperature": 15.6}'
  mosquitto_pub -t /admin/3bb9/attrs -m '{"temperature": 10.6}'


To retrieve all values sent for temperature attribute of this device:

.. code-block:: bash

  curl -X GET \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsIn...' \
    "http://localhost:8000/history/device/3bb9/history?lastN=3&attr=temperature"

The history endpoint is built from these values:

- ``.../device/3bb9/...``: the device ID is ``3bb9`` - this is retrieved from
  the ``id`` attribute from the device
- ``.../history?lastN=3&attr=temperature``: the requested attribute is
  temperature and it should get the last 3 values. More operators are available
  in `history APIs`_.

  The request should result in the following message:

.. code-block:: json

    [
      {
        "device_id": "3bb9",
        "ts": "2018-03-22T13:47:07.050000Z",
        "value": 10.6,
        "attr": "temperature"
      },
      {
        "device_id": "3bb9",
        "ts": "2018-03-22T13:46:42.455000Z",
        "value": 15.6,
        "attr": "temperature"
      },
      {
        "device_id": "3bb9",
        "ts": "2018-03-22T13:46:21.535000Z",
        "value": 36.5,
        "attr": "temperature"
      }
    ]


This message contains all previously sent values.


Integrating physical devices
----------------------------

If you want to integrate your device within dojot, it must be able to send
messages to the platform. There are two ways to do that:

- Use one of the available IoT agents: currently, there is support for
  MQTT-based devices. If your project is using (or allows changing to) this
  protocol, then it would suffice to check if the device is sending its data
  using a simple key/value JSON. If it isn't, then you might want to use
  iotagent-json and a few translation instructions while adding the device in
  dojot (check `iotagent-json`_ documentation to check out how to do that). If
  it is indeed sending key/value JSON messages, then it can send its messages
  to dojot's broker and it will be recognized by the platform.

- Create a new IoT agent to support the protocol used by the device: if your
  device is using another protocol that is not yet supported, then it might be
  a good idea to implement a new IoT agent. It's not that hard, but there are a
  few details that must be taken into account. To help developers to do such
  thing, there is the `iotagent-nodejs`_ library which deals with most
  internal mechanisms and messages - check its documentation to know more.

After your device is able to communicate with dojot, you can start using it as
described in `Step-by-step device management`_.


Flow Management
---------------

Moving to the perspective of an aplication developer, this section should list
and explain the usage of the information flow configuration process within the
platform - how to use the provided gui, high level description of the APIs that
can be used to configure such flows, available actions to be used when building
the flows, so on and so forth.



.. _YouTube channel: https://www.youtube.com/channel/UCK1iQ-d-K-O2mOLahPOoe6w
.. _JSON Web Token: https://tools.ietf.org/html/rfc7519
.. _jwt.io: https://jwt.io/
.. _auth: https://github.com/dojot/auth
.. _auth documentation: http://dojotdocs.readthedocs.io/projects/auth/
.. _docker-compose: https://github.com/dojot/docker-compose
.. _DeviceManager: https://github.com/dojot/device-manager
.. _DeviceManager documentation: http://dojotdocs.readthedocs.io/projects/DeviceManager/
.. _DeviceManager how-to: http://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/using-device-manager.html#using-devicemanager
.. _mashup: https://github.com/dojot/mashup
.. _mosquitto: https://projects.eclipse.org/projects/technology.mosquitto
.. _history APIs: https://dojot.github.io/history-ws/apiary_latest.html
.. _flowbroker: https://github.com/dojot/flowbroker
.. _iotagent-json: https://github.com/dojot/iotagent-json
.. _iotagent-nodejs: https://github.com/dojot/iotagent-nodejs
