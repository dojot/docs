.. _Concepts:

Concepts
========

This document provides information about dojot's concepts and abstractions.

.. contents:: Table of Contents
  :local:


.. note::

   - Audience
      - Users that want to take a look at how dojot works;
      - Application developers.
   - Level: basic


dojot basics
------------

Before using dojot, you should be familiar with some basic operations and
concepts. They are very simple to understand and use, but without them, all
operations might become obscure and senseless.

In the next section, there is an explanation of a few basic entities in dojot:
devices, templates and flows. With these concepts in mind, we present a small
tutorial to how to use them in dojot - it only covers API access. There a GUI
oriented tutorial in :doc:`using-web-interface` tutorial.


If you want more information on how dojot works internally, you should checkout
the :doc:`architecture` to get acquainted with all internal components.

.. _User authentication:

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
instructions (implemented in terms of data flows, build in flowbuilder) in
order to map the temperature reading that will be sent from the device to a
"temperature" attribute.

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
