Using API interface
===================

This section provides a complete step-by-step tutorial of how to create,
update, send messages to and check historical data of a device. Also, this
tutorial assumes that you are using `docker-compose`_, which has all the
necessary components to properly run dojot.

.. note::
   - Audience: developers
   - Level: basic
   - Reading time: 15 m


Prerequisites
-------------

It uses:

- `curl`_  to access the dojot platform APIs;
- `jq`_  to process the return JSON from the dojot platform APIs.
- `mosquitto`_ to publish and subscribe in `iotagent-mosca`_ (or iotagent-mqtt) via MQTT.


In Debian-based Linux distributions you can run:

.. code-block:: bash

  sudo apt-get install curl
  sudo apt-get install jq
  sudo apt-get install mosquitto-clients


Getting access token
--------------------

All requests must contain a valid access
token. You can generate a new token by sending the following request:

.. code-block:: bash

  JWT=$(curl -s -X POST http://localhost:8000/auth \
  -H 'Content-Type:application/json' \
  -d '{"username": "admin", "passwd" : "admin"}' | jq -r ".jwt")

To check:

.. code-block:: bash

  echo $JWT

If you want to generate a token for other user, just change the username and
password in the request payload. The token ("eyJ0eXAiOiJKV1QiL...") should be
used in every HTTP request sent to dojot in a special header. Such request
would look like:

.. code-block:: bash

   curl -X GET http://localhost:8000/device \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiL..."

Remember that the token must be set in the request header as a whole, not parts
of it. In the example only the first characters are shown for the sake of
simplicity. All further requests will use an evironment variable called
``${JWT}``, which contains the token got from auth component.


Device creation
---------------

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
        },
        {
          "label": "fan",
          "type": "actuator",
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
          },
          {
            "template_id": "1",
            "created": "2018-01-25T12:30:42.167126+00:00",
            "label": "fan",
            "type": "actuator",
            "value_type": "float",
            "id": 2
          }
        ],
        "id": 1
      }
    }

Note that the template ID is 1 (line 35), if you have already created another template this id will be different.

To create a template based on it, send the following request to dojot:

.. code-block:: bash

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
  :linenos:

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
              },
              {
                "template_id": "1",
                "created": "2018-01-25T12:30:42.167126+00:00",
                "label": "fan",
                "value_type": "actuator",
                "type": "float",
                "id": 2
             }
            ]
          },
          "id": "0998",
          "label": "device_0"
        }
      ]
    }


Sending messages
----------------

So far we got an access token and created a template and a device based on it. In an actual
deployment, the physical device would send messages to dojot with all its attributes and their
current values. For this tutorial we will send MQTT messages by hand to the platform, emulating such
physical device. For that, we will use mosquitto_pub and mosquitto_sub from `mosquitto`_.

The default message format used by dojot is a simple key-value JSON (you could
translate any message format to this scheme using flows, though), such as:

.. code-block:: json

    {
      "temperature" : 10.6
    }


.. ATTENTION::
    Some Linux distributions, Debian-based Linux distributions in particular, have two packages for
    `mosquitto`_ - one containing tools to access it (i.e. mosquitto_pub and mosquitto_sub for
    publishing messages and subscribing to topics) and another one containing the MQTT broker too.
    In this tutorial, only the tools from package `mosquitto-clients` on Debian-based Linux
    distributions are going to be used. Please check  if another MQTT broker is not running before starting
    dojot (by running commands like ``ps aux | grep mosquitto``) to avoid port conflicts.


For simplicity's sake, we are not using TLS in the examples below. Check :doc:`mqtt-tls` for more
information on its usage.

.. Note::
    To run `mosquitto_pub` and `mosquitto_sub` without using TLS
    as in the examples below, you need to configure some settings
    (or for how to disable the mode without TLS). For more details on this topic, please refer to the :ref:`Unsecured mode MQTT` page.

As of **v0.5.0**, you can choose the between two MQTT brokers: Mosca or VerneMQ. By default, VerneMQ
is used, but you can use Mosca too. Check the :doc:`../installation-guide` for more information.

Using VerneMQ
^^^^^^^^^^^^^

Let's send a message to dojot:

.. code-block:: bash

  mosquitto_pub -h localhost -p 1883 -u admin:0998 -t admin:0998/attrs -m '{"temperature": 10.6}' -q 1


If there is no output, the message was sent to MQTT broker.

Note that we sent a message with the parameter ``-q 1``. This means that the message will use QoS 1,
i.e., the message is guaranteed to be sent at least one time.


**Also you can send a configuration message from dojot to the device to change some of its attributes.
The target attribute must be of type “actuator”.**

To simulate receiving the message on a device, we can use ``mosquitto_sub``:

.. code-block:: bash

  mosquitto_sub -h localhost -p 1883 -u admin:0998 -t admin:0998/config -q 1

Triggering the sending of the message from the dojot to the device.

.. code-block:: bash

  curl -X PUT \
      http://localhost:8000/device/0998/actuate \
      -H "Authorization: Bearer ${JWT}" \
      -H 'Content-Type:application/json' \
      -d '{"attrs": {"fan" : 100}}'


As noted in the :doc:`../faq/faq`, there are some considerations regarding MQTT topics:

- You must set the username that originates the message using the ``username`` MQTT parameter. It
  should follow the following pattern: ``<tenant>:<device-id>``, such as ``admin:efac``. It must
  match the tenant and device ID set in the topic.

- The topic to publish messages has the pattern ``<tenant>:<device-id>/attrs``
  (e.g.: ``admin:efac/attrs``).

- The topic to subscribe should has the pattern ``<tenant>:<device-id>/config``
  (e.g.: ``admin:efac/config``).

- MQTT payload must be a JSON with each key being an attribute of the dojot device, such as:

.. code-block:: javascript

  { "temperature" : 10.5, "pressure" : 770 }

Using Mosca (legacy)
^^^^^^^^^^^^^^^^^^^^

.. ATTENTION::
    VerneMQ is the new default MQTT broker. Support for Mosca will be eventually dropped, so use
    VerneMQ if possible!

Let's send a message to dojot:

.. code-block:: bash

  mosquitto_pub -h localhost -t /admin/0998/attrs -p 1883 -m '{"temperature": 10.6}'


If there is no output, the message was sent to MQTT broker.


**Also you can send a configuration message from dojot to the device to change some of its attributes.
The target attribute must be of type “actuator”.**

To simulate receiving the message on a device, we can use ``mosquitto_sub``:

.. code-block:: bash

  mosquitto_sub -h localhost -p 1883 -t /admin/0998/config

Triggering the sending of the message from the dojot to the device.

.. code-block:: bash

  curl -X PUT \
      http://localhost:8000/device/0998/actuate \
      -H "Authorization: Bearer ${JWT}" \
      -H 'Content-Type:application/json' \
      -d '{"attrs": {"fan" : 100}}'


As noted in the :doc:`../faq/faq`, there are some considerations regarding MQTT topics:

- You can set the device ID that originates the message using the ``client-id`` MQTT parameter. It
  should follow the following pattern: ``<tenant>:<device-id>``, such as ``admin:efac``.

- If you can't do such thing, then the device should set its ID using the topic used to publish
  messages. The topic should assume the pattern ``/<tenant>/<device-id>/attrs``
  (e.g.: ``/admin/efac/attrs``).

- The topic to subscribe should assume the pattern ``/<tenant>/<device-id>/config``
  (e.g.: ``/admin/efac/config``).

- MQTT payload must be a JSON with each key being an attribute of the dojot
  device, such as:

.. code-block:: javascript

  { "temperature" : 10.5, "pressure" : 770 }


.. Note::
    For the rest of the tutorial we will treat as if you are using VerneMQ.

Checking historical data
------------------------

In order to check all values that were sent from a device for a particular
attribute, you could use the history api, see more in :doc:`components-and-apis`.
Let's first send a few other values to dojot so we can get a few more interesting results:

.. code-block:: bash

  mosquitto_pub -h localhost -p 1883 -u admin:0998 -t admin:0998/attrs -m '{"temperature": 36.5}' -q 1
  mosquitto_pub -h localhost -p 1883 -u admin:0998 -t admin:0998/attrs -m '{"temperature": 15.6}' -q 1
  mosquitto_pub -h localhost -p 1883 -u admin:0998 -t admin:0998/attrs -m '{"temperature": 10.6}' -q 1


To retrieve all values sent for temperature attribute of this device:

.. code-block:: bash

  curl -X GET \
    -H "Authorization: Bearer ${JWT}" \
    "http://localhost:8000/history/device/0998/history?lastN=3&attr=temperature"

The history endpoint is built from these values:

- ``.../device/0998/...``: the device ID is ``0998`` - this is retrieved from
  the ``id`` attribute from the device
- ``.../history?lastN=3&attr=temperature``: the requested attribute is
  temperature and it should get the last 3 values.

  The request should result in the following message:

.. code-block:: json

    [
      {
        "device_id": "0998",
        "ts": "2018-03-22T13:47:07.050000Z",
        "value": 10.6,
        "attr": "temperature"
      },
      {
        "device_id": "0998",
        "ts": "2018-03-22T13:46:42.455000Z",
        "value": 15.6,
        "attr": "temperature"
      },
      {
        "device_id": "0998",
        "ts": "2018-03-22T13:46:21.535000Z",
        "value": 36.5,
        "attr": "temperature"
      }
    ]


This message contains all previously sent values.


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
.. _curl: https://curl.haxx.se/
.. _jq: https://stedolan.github.io/jq/
.. _flowbroker: https://github.com/dojot/flowbroker
.. _iotagent-mosca: https://github.com/dojot/iotagent-mosca
.. _iotagent-nodejs: https://github.com/dojot/iotagent-nodejs
