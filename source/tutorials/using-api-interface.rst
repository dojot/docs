.. _Using API interface:

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


Getting access token
--------------------

As said in :ref:`User authentication`, all requests must contain a valid access
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
simplicity. All further requests will use an evironment variable called ``bash
${JWT}``, which contains the token got from auth component.


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
----------------

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


The default message format used by dojot is a simple key-value JSON (you could
translate any message format to this scheme using flows, though), such as:

.. code-block:: json

    {
      "temperature" : 10.6
    }

Let's send this message to dojot:

.. code-block:: bash

  mosquitto_pub -t /admin/0998/attrs -m '{"temperature": 10.6}'


If there is no output, the message was sent to MQTT broker.

As noted in the :doc:`../faq/faq`, there are some considerations regarding MQTT
topics:

- You can set the device ID that originates the message using the ``client-id``
  MQTT parameter. It should follow the following pattern:
  ``<service>:<deviceid>``, such as ``admin:efac``.

- If you can't do such thing, then the device should set its ID using the topic
  used to publish messages. The topic should assume the pattern
  ``/<service-id>/<device-id>/attrs`` (for instance: ``/admin/efac/attrs``).

- If you do define a topic in device template, then your device should publish
  its data to it and set the client-id parameter.

- MQTT payload must be a JSON with each key being an attribute of the dojot
  device, such as:

.. code-block:: javascript

  { "temperature" : 10.5,"pressure" : 770 }

For more information on how dojot deals with data sent from devices, check the
:doc:`integrating-physical-devices` tutorial. There you can find how to deal
with devices that don't publish messages in such format and how to translate
them.

Checking historical data
------------------------

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
.. _iotagent-mosca: https://github.com/dojot/iotagent-mosca
.. _iotagent-nodejs: https://github.com/dojot/iotagent-nodejs
