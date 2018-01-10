User Guide
==========

This document provides information on how to use dojot from a user perspective. On that
regard, this should describe the steps required to install and operate the platform from a device
developer or application developer point of view. For documentation regarding the operation of the
platform itself, please refer to the `operations guide <ops_guide.html>`_

.. contents:: Table of Contents
  :local:

Who should read this
--------------------

- Users that want a deeper look at how dojot works;
- Application developers


Getting Started
---------------

To start, please follow dojot's installation guide. There you should find how to properly 
download a working copy of the components, how to minimally configure them, how to start 
them up and how to check whether they are working.

dojot basics
------------

Before using dojot, the user must be familiar with some basic operations and concepts. 
They are very simple to understand and use, but without them, all operations might become 
obscure and senseless. It is advisable to checkout our 
`architecture description <architecture.html>`_ to get acquainted with all internal components.


User authentication
*******************

All HTTP requests supported by dojot are sent to the API gateway. In order to control 
which user should access which endpoints and resources, dojot makes uses of JWT 
(`JSON Web Token <https://tools.ietf.org/html/rfc7519>`_. A useful tool 
is `jwt.io <https://jwt.io/>`_) which encodes things like (not limited to these):

- User identity
- Validation data
- Timestamp

The component responsible for user authentication is `auth <https://github.com/dojot/auth>`_.


Devices and templates
*********************

In dojot, a device is a digital representation of an actual device or gateway with one or 
more sensors or of a virtual one with sensors/attributes inferred from other devices. 
Throughout the documentation, this kind of device will be called simply as 'device'. 
If the actual device must be referenced, we'll be calling it as 'physical device'.

Consider, for instance, a physical device with temperature and humidity sensors; 
it can be represented into dojot as a device with two attributes (one for each sensor). 
We call this kind of device as regular device or by its communication protocol, for instance, 
MQTT device or CoAP device.

We can also create devices which don’t directly correspond to their physical counterparts, 
for instance, we can create one with higher level of information of temperature (is becoming 
hotter or is becoming colder) whose values are inferred from temperature sensors of other devices. 
This kind of device is called virtual device.

All devices are created based on a *template*, which can be thought as a model of a device. 
As "model" we could think of part numbers or product models - one *prototype* from which devices 
are created. Templates in dojot have one label (any alphanumeric sequence), a list of attributes 
which will hold all the device emitted information, and optionally a few special attributes which 
will indicate how the device communicates, including transmission methods (protocol, ports, etc.) 
and message formats.

In fact, templates can represent not only "device models", but it can also abstract a "class 
of devices". For instance, we could have one template to represent all thermometers that will 
be used in dojot. This template would have only one attribute called, let's say, "temperature". 
While creating the device, the user would select its "physical template", let's say 
*TexasInstr882*, and the 'thermometer' template. The user would have also to add translation 
instructions in order to map the temperature reading that will be sent from the device to a 
"temperature" attribute. 

In order to create a device, a user selects which templates are going to compose this new device. 
All their attributes are merged together and associated to it - they are tightly linked to 
the original template so that any template update will reflect all associated devices.

The component responsible for managing devices (both real and virtual) and templates is 
`DeviceManager <https://github.com/dojot/device-manager>`_. 
`DeviceManager documentation <https://dojot.github.io/device-manager>`_ lists all the supported messages.


Flows
*****

This section will explain what a flow is and how to use it. It will be filled as soon 
as `mashup <https://github.com/dojot/mashup>`_ documentation is ready.

First steps
-----------

As said in  `User authentication`_, all requests must contain a valid access token. 
You can generate a new token by sending the following request:

.. code-block:: bash
  :linenos:

  $ curl -X POST http://localhost:8000/auth \
         -H 'Content-Type:application/json' \
         -d '{"username": "admin", "passwd" : "admin"}'

  {"jwt": "eyJ0eXAiOiJKV1QiL..."}

If you want to generate a token for other user, just change the username and password in 
the request payload.
The token ("eyJ0eXAiOiJKV1QiL...") should be used in every HTTP request sent to dojot in 
a special header. Such request would look like:

.. code-block:: bash
   :linenos:

   $ curl -X GET http://localhost:8000/device -H "Authorization: Bearer eyJ0eXAiOiJKV1QiL..." 

Remember that the token must be set in the request header as a whole, not parts of it. 
In the example only the first characters are shown for the sake of simplicity.


Device Management
-----------------

In order to properly configure a physical device in dojot, you must first create a 
representation to it in the platform. 
`Device manager how-to <https://dojot.github.io/device-manager/using-device-manager.html>`_ 
contains a tutorial to how to do that.


Integrating physical devices
----------------------------

This section should detail how to integrate a new device with the system. That should encompass
the both the communication requirements imposed on the device in order to allow its usage with
the platform, as well as the steps (if any, depending on the protocol used) to configure this
new device within the platform.

This could also explain (if indeed implemented) the device management functionalities made available
by the platform to the device developer.

Regarding the requirements imposed on the devices, it is forseen that, for each communication scheme
(protocol/serialization format) offically supported by the platform, a step by step guide on
how to "develop" a device is supplied. Such guide can, if applicable, make use of a platform-provided
library or SDK.


Flow Management
---------------

Moving to the perspective of an aplication developer, this section should list and explain the usage
of the information flow configuration process within the platform - how to use the provided gui,
high level description of the APIs that can be used to configure such flows, available actions to
be used when building the flows, so on and so forth.

