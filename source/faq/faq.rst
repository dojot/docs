.. _faq:

Frequently Asked Questions
==========================

Here are some answers to frequently-asked questions from users of dojot
platform.

Got a question that isn't answered here? Please, open an issue on `dojot's Github repository
<http://github.com/dojot/dojot>`_.

**Table of Contents**

.. contents::
  :local:

General
-------
.. _general:

What is dojot? Why should I use it? Why open source it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It’s a brazilian IoT platform launched as open source software with aims to
ease the development of solutions and the IoT ecosystem with local resources
geared towards brazilians needs.

The dojot acts as a facilitating platform with:

  - Open APIs which makes the access to the platform resources easy.
  - Capacity to store large volumes of data in different formats.
  - Connectors to different types of devices.
  - Graphical user interface with flow builder to quickly prototype IoT solutions.
  - Real time event processing with customizable rules.

Where can I get it?
^^^^^^^^^^^^^^^^^^^

All components are available in dojot's GitHub repositories: `<https://github.com/dojot>`_.

Which repository is the main one?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are 3 main ones:

- `<https://github.com/dojot/dojot>`_: this is where we keep track of all the
  things related to this project as issues and there are also some component codes here.

- `<https://github.com/dojot/docker-compose>`_: repository with the files and settings 
to perform dojot in a Docker environment using the *docker-compose* tool. This is the 
repository we recommend to start with dojot.

- `<https://github.com/dojot/ansible-dojot>`_: repository with files and settings to perform 
dojot in *Kubernetes* environment.

See how to use *docker-compose* and *ansible-dojot* repositories in :doc:`../installation-guide`.

So, I found this pesky bug. How can I inform you about it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We ask you to open an issue in `dojot's Github repository
<http://github.com/dojot/dojot>`_.

If you are able to analyze and fix this bug, please do so. Create a
pull-request with a quick description of what you've done.

Usage
-----
.. _usage:

How do I start it? Is it CLI-based or it has a graphical user interface?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Dojot can be accessed by a web-based interface and by REST APIs.
Considering that you installed ``docker`` and ``docker-compose`` and cloned the
``docker-compose`` repository, starting it up is done by just one command:

.. code-block:: bash

   $ docker-compose up -d

And that's it.

The web interface is available at ``http://localhost:8000``. The user is
``admin``, password ``admin``.

REST APIs are explained in the `Applications`_ section.

Ok, I started it and I logged in. Now what?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nice! Now you can add your templates and devices, described in `Devices`_,
build some flows and subscribing to device events, both described in `Data
Flows`_.

How can I update my deploy to dojot's latest version?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You need to follow some steps:

1 Update the docker-compose repository to the latest version.

.. code-block:: bash

  $ cd <path-to-your-clone-of-docker-compose>
  $ git checkout master && git pull


If you need another version, you could checkout a tag instead:

.. code-block:: bash

  $ git tag
    0.1.0-dojot
    0.1.0-dojot-RC1
    0.1.0-dojot-RC2
    0.2.0
    v0.3.0-beta.1
    v0.3.1
    v0.4.0
    v0.4.1
    v0.4.1_rc2
    v0.4.2
    v0.4.2-rc.1
    v0.4.3
    v0.4.3-rc.1
    v0.4.3-rc.2
    v0.5.0-alpha.1
    v0.5.0-alpha.2
    v0.5.0-alpha.3
    v0.5.0-alpha.4
    v0.5.0-rc.1
    v0.5.0


  $ git checkout v0.5.0


Devices
-------
.. _devices:

What are *devices* for dojot?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In dojot, a device is a digital representation of an actual device or gateway
with one or more sensors or of a virtual one with sensors/attributes inferred
from other devices.

Consider, for instance, an actual device with thermal and humidity sensors; it
can be represented inside dojot as a device with two attributes (one for each
sensor). We call this kind of device as *regular device* or by its
communication protocol, for instance, *MQTT device* or *CoAP device*.

We can also create devices which don’t directly correspond to their physical
counterparts, for instance, we can create one with a higher level of
temperature information (*is becoming hotter* or *is becoming colder*) whose
values are inferred from temperature sensors of other devices. This kind of
device is called *virtual device*.

What is the relationship between this *device* and my actual device?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is as simple as it seems: the *regular device* for dojot is a mirror
(digital twin) of your actual device. You can choose which attributes are
available for applications and other components by adding each one of them at
the device creation interface.

What are *virtual devices*? How are they different from the other one?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*Regular devices* are created to serve as a mirror (digital twin) for the
actual devices and sensors. A *virtual device* is an abstraction that models
things that are not feasible in the real world. For instance, let's say that a
user has few smoke detectors in a laboratory, each one with different
attributes.

Wouldn't it be nice if we had one device called *Laboratory* that has one
attribute *isOnFire*? Therefore, the applications could rely only on this
attribute to take an action.

Another difference is how virtual devices are populated. Regular ones will be
filled with information sent by devices or gateways to the platform and virtual
ones will be filled by flows or by applications.


And what are *templates*?
^^^^^^^^^^^^^^^^^^^^^^^^^

Templates, simply put, are "blueprints for devices" which serve as basis to
create a new device. A single device is built using a set of templates - its
attributes will be inherited from each template (their names must not be
exactly the same, though). If one template is changed, then all associated
devices will also be changed.


How can I send MQTT data to dojot so that it appears on the dashboard?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all, you create a digital representation for your actual device. Then,
you configure it to send data to dojot so that it matches its digital
representation.

Let’s take as example a weather station which measures temperature and
humidity, and publishes them periodically through MQTT. First, you create a
device of type MQTT with two attributes (temperature and humidity). Then you
set your actual device to push the data to dojot.

.. ATTENTION::
  As of **v0.5.0**, you can choose the between two MQTT brokers: Mosca or VerneMQ. By default,
  VerneMQ is used, but you can use Mosca too. Check the :doc:`../installation-guide` for more information.

In order to send data to dojot via MQTT (using Mosca or VerneMQ), there are some
things to keep in mind:

- When using Mosca, the topic should look like ``/<tenant>/<device-id>/attrs`` (e.g.:
  ``/admin/efac/attrs``). Depending on how IoT agent MQTT was started (more strict), the client ID
  must also be set to ``<tenant>:<device-id>``, such as ``admin:efac``.

- When using VerneMQ, the topic should look like ``<tenant>:<device-id>/attrs`` (e.g.:
  ``admin:efac/attrs``). You must also set the username for the client as ``<tenant>:<device-id>``, such
  as ``admin:efac``, and it should match the same part in the topic. You can also set the client ID
  too (not required).

- MQTT payload must be a JSON with each key being an attribute of the dojot
  device, such as:

.. code-block:: javascript

  { "temperature" : 10.5, "pressure" : 770 }

You can use certificates with MQTT, check :doc:`../mqtt-tls` for more information.

On the dashboard some attributes are shown as tables and others as charts. How are they chosen/set?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The type of an attribute determines how the data is shown on the dashboard as
follows:

  - ``Geo``: geo map.
  - ``Boolean``, ``Text`` and  ``JSON``: table.
  - ``Integer`` and ``Float``: line chart.

I'm interested in integrating my super cool device with dojot. How can I do it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If your device is able to send messages using MQTT (with JSON payload), CoAP or
HTTP, there is a good chance that your device can be integrated with minor or
no modifications whatsoever.

Is there any restrictions about the message my device will send to dojot? Format, size, frequency?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

None but format, which is described in the question `How can I send MQTT data
to dojot so that it appears on the dashboard?`_.

How can I send some commands to my device through dojot?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For now, you can send HTTP requests to dojot containing a few instructions
about which device should be configured and the actuation payload itself. More
details on that can be found :ref:`Sending messages`.


I didn’t find the protocol supported by my device in the type list, is there anything I can do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are some possibilities. The first one is to develop a proxy to translate
your protocol to one supported by dojot. The second one is to develop a IotAgent, a
connector, similar to the existing ones
for MQTT, CoAP and HTTP. Take a look at https://github.com/dojot/iotagent-nodejs.


How can I retrieve historical data for a particular device?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can do this by sending a request to /history endpoint, such as:

.. code-block:: bash

  curl -X GET \
    -H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsIn...' \
    "http://localhost:8000/history/device/3bb9/history?lastN=3&attr=temperature"


which will retrieve the last 3 entries of `temperature` attribute from the
device `3bb9`:

.. code-block:: json

    [
      {
        "device_id": "3bb9",
        "ts": "2018-03-22T13:47:07.050000Z",
        "value": 29.76,
        "attr": "temperature"
      },
      {
        "device_id": "3bb9",
        "ts": "2018-03-22T13:46:42.455000Z",
        "value": 23.76,
        "attr": "temperature"
      },
      {
        "device_id": "3bb9",
        "ts": "2018-03-22T13:46:21.535000Z",
        "value": 25.76,
        "attr": "temperature"
      }
    ]

For more details on data retrieval from the history, check the tutorial in :ref:`Checking historical data`.

In addition, there are more operators that could be used to filter entries.
Look for the History API in :doc:`./../components-and-apis`
documentation to check out all possible operators and other filters.


Data Flows
----------
.. _data_flows:

What is data flow?
^^^^^^^^^^^^^^^^^^

It’s a sequence of functional blocks to process incoming device messages. With
a flow you can dynamically analyze each new message in order to apply
validations, infer information and trigger actions or notifications.

The data flow UI… really looks like node-RED. Are they related in some way?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It’s based on the Node-RED frontend, but uses its own engine to process the
messages. If you’re familiar with Node-Red, it won't be difficult to use it.

Why should I use it?
^^^^^^^^^^^^^^^^^^^^

It allows one of the coolest things of IoT in an easy and intuitive way, which
is to analyze data for extracting information and then take actions.

What can it do, exactly?
^^^^^^^^^^^^^^^^^^^^^^^^

You can do things such as:

  - Create views from a particular device, by renaming, aggregating and
    changing values, etc).
  - Infer information based on switch, edge-detection and geo-fence rules.
  - Notify through many ways, like HTTP.

The data flows component is in constantly development with new features being
added every new release.

There are mechanisms to add new processing blocks to new flows. Check the `How
can I add a new node type to its menu?`_ question for more information on that.

So, how can I use it?
^^^^^^^^^^^^^^^^^^^^^

For more details on how to use flows, check the tutorial in :doc:`../flow`.

Can I apply the same flow to multiple devices?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can use a template as input to indicate that the flow should be applied to
all devices associated to that template. It’s worth to point out that the flow
is processed individually for each new input message, i.e. for each input
device.

Can I correlate data from different devices in the same flow?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As the data flow is processed individually for each message, you need to create
a virtual device to aggregate all attributes, then use this virtual device as
the input of the flow.

You can also create a node (or use an already existing one) that deals with contexts.

What about a HTTP POST request, how can I send it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^


.. image:: df_http_request.gif
        :width: 95%
        :align: center

One important note: make sure that dojot can access your server.

I want to rename the attributes of a device, what should I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all, you need to create a virtual device with the new attributes, then
you build a data flow to rename them. This can be done connecting a ‘change’
node after the input device to map the input attributes to the corresponding
ones into an output, and finally connecting the ‘change’ to the virtual device
and assigning to it the output.

.. image:: df_attributes_renaming.gif
        :width: 95%
        :align: center

I want to aggregate the attributes of multiple devices, what should I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all, you need to create a virtual device to aggregate all attributes,
then you build a data flow to map the attributes of each device to the virtual
one. This can be done connecting a ‘change’ node after each input device to put
the input values into an output, and finally connecting all changes to the
virtual device and assigning to it the output.

.. image:: df_attributes_aggregation.gif
        :width: 95%
        :align: center

How can I add a new node type to its menu?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There is a tutorial on how to add new nodes and two examples of node too,
check the `GitHub flowbroker repository`_ for more details.


Applications
------------
.. _applications:

What APIs are available for applications?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can check all available APIs in the :doc:`./../components-and-apis`.

How can I use them?
^^^^^^^^^^^^^^^^^^^

There is a very quick and useful tutorial in the :doc:`./../using-api-interface`.

I'm interested in integrating my application with dojot. How can I do it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This should be pretty straightforward. There are three ways that your application
could be integrated with dojot:

  - **Retrieving historical data**: you might want to periodically read all
    historical data related to a device. This can be done by using the *history* API
  - **Retrieving real time data**: you might want to read device-related real time data.
    This can be done by using *Kafka WS*, a *websocket* based implementation.
    To better understand how to use *Kafka WS* check :ref:`Kafka-WS Internal`.
  - **Using flowbroker to pre-process data**: for more flexible ways of data manipulation, you can use flows.
    They can process/transform data so they can
    be properly sent to your application via HTTP request, or stored
    in a virtual device (which can be used to generate notifications as
    previously described).


All these endpoints should bear an access token, see more :ref:`Getting access token`.

Check the documentation API for  *History* and *Kafka WS* in :doc:`./../components-and-apis`.
And to a tutorial on how to use the flow, check :doc:`../flow`.


.. _flowbroker library : https://github.com/dojot/flowbroker/tree/v0.5.0/lib

