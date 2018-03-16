.. _faq:

Frequently Asked Questions
==========================

Here are some answers to frequently-asked questions from dojot platform.

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

It’s a brazilian IoT platform launched as open source software with aims to ease the development of solutions and the
IoT ecosystem with local resources geared towards brazilians needs.

It takes a role as an enabler platform with:

	- Open APIs which makes the access to the platform resources easy.

	- Capacity to store large volumes of data in different formats.

	- Connectors to different types of devices.

	- Graphical user interface with flow builder to prototype IoT solutions very quickly.

	- Real time event processing with customizable rules.

Where can I get it?
^^^^^^^^^^^^^^^^^^^

All components are available in dojot's GitHub repositories: `<https://github.com/dojot>`_.

Which repository is the main one?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are two main ones:

	- `<https://github.com/dojot/dojot>`_: this is where we keep track of all the things 
	  related to this project as a whole, such as architectural enhancements.
	
	- `<https://github.com/dojot/docker-compose>`_: repository for Docker compose files 
	  and configurations. This is what we would recommend to use to start with. 

So, I found this pesky bug. How can I inform you about it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

We ask you to open an issue in `dojot's Github repository <http://github.com/dojot/dojot>`_. If you know exactly which
component is failing, you could open the issue in its repository (it will work the same way).

If you are able to analyze and fix this bug, please do so. Create a pull-request with a quick description of what
you've done.

Usage
-----
.. _usage:

How do I start it? Is it CLI-based or it has a graphical user interface?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

dojot can be accessed by a nice web-based interface and by REST APIs. Considering that you installed ``docker`` and
``docker-compose`` and cloned the ``docker-compose`` repository, there are a few steps to start it up:

.. code-block:: bash

	$ docker-compose up -d

And that's it. 

The web interface is available at ``http://localhost:8000``. The user is ``admin``, password ``admin``.

REST APIs are explained in the `Applications`_ section.

Ok, I started it and I logged in. Now what?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Nice! Now you can add your first devices, described in `Devices`_, build some flows and subscribing to device events,
both described in `Data Flows`_.

How can I update my deploy to dojot's latest version?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You need to follow some steps:

1. Update the docker-compose repository to the latest version.
	
.. code-block:: bash	
	
	$ cd <path-to-your-clone-of-docker-compose>
	$ git checkout master && git pull
			
2. Deploy the latest docker images.

.. code-block:: bash
	
	$ docker-compose pull && docker-compose up -d --build
	
This procedure also applies to the available virtual machines once they do use docker-compose.
	

Devices
-------
.. _devices:

What are *devices* for dojot?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

In dojot, a device is a digital representation of an actual device or gateway with one or more sensors or of a virtual
one with sensors/attributes inferred from other devices.

Consider, for instance, an actual device with temperature and humidity sensors; it can be represented into dojot as a
device with two attributes (one for each sensor). We call this kind of device as *regular device* or by its
communication protocol, for instance, *MQTT device* or *CoAP device*.

We can also create devices which don’t directly correspond to their actual ones, for instance, we can create one with
higher level of information of temperature (*is becoming hotter* or *is becoming colder*) whose values are inferred
from temperature sensors of other devices. This kind of device is called *virtual device*.

What is the relationship between this *device* and my actual device?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is simple as it seems: the *regular device* for dojot is a mirror (digital twin) of your actual device. You can
choose which attributes are available for applications and other components by adding each one of them at the device
creation interface. If you don't want some attributes to be available to applications or other elements, just don't add
them in dojot.

What are *virtual devices*? How are they different from the other one?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

*Regular devices* are created to serve as a mirror (digital twin) for the actual devices and sensors. A *virtual
device* is an abstraction that models things that are not feasible in the real world. For instance, let's say that a
user has few smoke detectors in a laboratory, each one with different attributes.

Wouldn't it be nice if we had one device called *Laboratory* that has one attribute *isOnFire*? So, the applications
could rely only on this attribute to take an action.

Another difference is how virtual devices are populated. Regular ones will be filled with information sent by devices
or gateways to the platform and virtual ones will be filled by flows or by applications (they won't accept messages
addressed to them via MQTT, for example).


And what are *templates*?
^^^^^^^^^^^^^^^^^^^^^^^^^

Templates, simply put, are "blueprints for devices" which serve as basis to create a new device. A single device is
built using a set of templates - its attributes will be inherited from each template (their names must not be exactly
the same, though). If one template is changed, then all associated devices will also be changed.


How can I send MQTT data to dojot so that it appears on the dashboard?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all, you create a digital representation for your actual device. Then, you configure it to send data to dojot
so that it matches its digital representation.

Let’s take as example a weather station which measures temperature and humidity, and publishes them periodically
through MQTT. First, you create a device of type MQTT with two attributes (temperature and humidity). Then you set your
actual device to push the data to dojot. Here, you need to follow some rules:

  - MQTT topic must follow the pattern ``/<service-id>/<device-id>/attrs``, where ``<service-id>`` is an identifier
    associated with the user account and the ``<device-id>`` is a unique identifier assigned by dojot. For example,
    topic ``/admin/882d/attrs`` must be used for user ``admin`` and device ID ``882d``. There are mechanisms to change 
    this default topic scheme - check `iotagent-json documentation`_ for that.
  
 	- MQTT payload must be a JSON with each key being an attribute of the dojot device, such as:
 	  
.. code-block:: javascript

	{ "temperature" : 10.5,"pressure" : 770 }
	
	
It’s worth to point out that we are relaxing these rules so that you’ll have more flexibility to configure both topic
and payload. This feature will be available in the next official release.

On the dashboard some attributes are shown as tables and others as charts. How are they chosen/set?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The type of an attribute determines how the data is shown on the dashboard as follows:

	- ``Geo``: geo map.

	- ``Boolean`` and ``Text``: table.

	- ``Integer`` and ``Float``: line chart.

I'm interested in integrating my super cool device with dojot. How can I do it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If your device is able to send messages using MQTT (with JSON payload), CoAP or HTTP, there is a good chance that your
device can be integrated with minor or no modifications whatsoever. The requirements for such integration is described
in the question `How can I send MQTT data to dojot so that it appears on the dashboard?`_.

Is there any restrictions about the message my device will send to dojot? Format, size, frequency?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

None but format, which is described in the question `How can I send MQTT data to dojot so that it appears on the
dashboard?`_.

How can I send some commands to my device through dojot?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For now, you can send HTTP requests to dojot containing a few instructions about which device should be configured and
the actuation payload itself. More details on that can be found in `Device-Manager how-to - sending actuation
messages`_.


I didn’t find the protocol supported by my device in the type list, is there anything I can do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are some possibilities. The first one is to develop a proxy to translate your protocol to one supported by dojot.
The second one is to develop a connector similar to the existing ones for MQTT, CoAP and HTTP.


I saved an attribute, but it disappeared from the device. Is it a bug?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You might have saved the attribute, but not the device. If you don’t click on the save button for the device, the added
attributes will be discarded. We’re improving the system messages to caveat the users and remember them to save their
configurations.

How can I retrieve historical data for a particular device?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can do this by sending a request to /history endpoint, such as:

.. code-block:: bash

	curl -X GET \
		-H 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsIn...' \
		http://localhost:8000/history/STH/v1/contextEntities/type/device/id/3ba9/attributes/temperature?lastN=10


which will retrieve the last 10 entries of `temperature` attribute from the device `3ba9`. There are more operators
that could be used to filter entries. Check `STH
<https://github.com/telefonicaid/fiware-sth-comet/blob/master/doc/manuals/raw-data-retrieval.md>`_ documentation to
check out all possible operators.


Data Flows
----------
.. _data_flows:

What is data flow?
^^^^^^^^^^^^^^^^^^

It’s a processing flow for income messages/data of a device. With a flow you can dynamically analyse each new message
in order to apply validations, infer information, and trigger actions or notifications.

The data flow UI… really looks like node-RED. Are they related in some way?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It’s based on the Node-RED frontend, but uses its own engine to process the messages. 
If you’re familiar with Node-Red, you won’t have any difficult to use it.

Why should I use it?
^^^^^^^^^^^^^^^^^^^^

It allows one of the coolest things of IoT in an easy and intuitive way, which is to analyse data 
for extracting information, then take actions.

What can it do, exactly?
^^^^^^^^^^^^^^^^^^^^^^^^

You can do things such as:

	- Create virtual viewers of a device (rename attributes, aggregate attributes, 
	  change values, etc).

	- Infer information based on switch rules.

	- Infer information based on edge-detection rules.

	- Infer information based on geo-fence rules.
	
	- Notify through email.
	
	- Notify through HTTP.

The data flows component is in constantly development with new features being added every new 
release.

So, how can I use it?
^^^^^^^^^^^^^^^^^^^^^

It follows the basic usage flow as node-RED. You can check its `documentation <https://nodered.org>`_ for more details
about this.

Can I apply the same flow to multiple devices?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Multiple devices can be used both as input and output of data flows. It’s worth to point out that the flow is processed
individually for each new input message, i.e. for each input device.

Can I correlate data from different devices in the same flow?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As the data flow is processed individually for each message, you need to create a virtual device to aggregate all
attributes, then use this virtual device as the input of the flow.

I want to send an email, what should I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Basically, you need to add an email node and configure it. This node is pre-configured to use the Gmail server
``gmail-smtp-in.l.google.com``, but you’re free to choose your own. For writing an email body, you can use a template
before the email.

.. image:: df_email.gif
        :width: 95%
        :align: center
        
It is important to point out that dojot contains no e-mail server. It will generate SMTP commands and send them to the
specified e-mail server.


What about a HTTP POST request, how can I send it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is almost the same process as sending an e-mail. 

.. image:: df_http_request.gif
        :width: 95%
        :align: center

One important note: make sure that dojot can access your server. 

I want to rename the attributes of a device, what should I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all, you need to create a virtual device with the new attributes, then you build a data flow to rename them.
This can be done connecting a ‘change’ node after the input device to map the input attributes to the corresponding
ones into an output, and finally connecting the ‘change’ to the virtual device and assigning to it the output.

.. image:: df_attributes_renaming.gif
        :width: 95%
        :align: center

I want to aggregate the attributes of multiple devices, what should I do?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First of all, you need to create a virtual device to aggregate all attributes, then you build a data flow to map the
attributes of each device to the virtual one. This can be done connecting a ‘change’ node after each input device to
put the input values into an output, and finally connecting all changes to the virtual device and assigning to it the
output.

.. image:: df_attributes_aggregation.gif
        :width: 95%
        :align: center

It would be cool a WhatsApp node, is it in roadmap?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It’s under analysis. We intend to support other notifications systems besides email, including WhatsApp, Twitter and
Telegram. If you also have interest, please help us to develop them.

Applications
------------
.. _applications:

What APIs are available for applications?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can check all available APIs in the `API Listing page <../components-and-apis.html>`_

How can I use them?
^^^^^^^^^^^^^^^^^^^

There is a very quick and useful tutorial in the `User Guide <../user_guide.html#step-by-step-device-management>`_.
	
I'm interested in integrate my application with dojot. How can I do it?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This should be pretty straightforward. There are two ways that your application could be 
integrated with dojot:
      
  - **Retrieving historical data**: you might want to periodically read all historical data related to a device. This
    can be done by using this API (one side-note: all endpoints described in this apiary should be preceded by
    ``/history/``).

  - **Subscribing to events related to devices**: if your application is able to listen to events, you might rather use
    subscriptions, which can be created using this API (also, all endpoints should be preceded by ``/metrics/``).
   
  - **Using mashup to pre-process data**: if you want to do something more, you could use flows. They can help process
    and transform data so that they can be properly sent to your application via HTTP request, by e-mail or stored in a
    virtual device (which can be used to generate notifications as previously described).

All these endpoints should bear an access token, which is retrieved as described in the question `How can I use
them?`_.


.. _Device-Manager how-to - sending actuation messages: http://dojotdocs.readthedocs.io/projects/DeviceManager/en/latest/using-device-manager.html#sending-actuation-messages-to-devices
.. _iotagent-json documentation: http://dojotdocs.readthedocs.io/projects/iotagent-json/en/latest/
