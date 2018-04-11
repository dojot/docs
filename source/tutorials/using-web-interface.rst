Using dojot's web interface
===========================

This tutorial will show how to do basic operations in dojot, such as creating
devices, checking its attributes and creating flows.

Simple use
----------

This section will show how to manage device. For this tutorial we will show
how to add two thermometers and a virtual device that will represent an alarm
system that will monitor both sensors.

As described in `User Guide`_, all devices are based on a template. To create
one, you should access the template tab at the left and then create one new
template, as shown below.

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/C-P_SfeNA_A?rel=0" frameborder="0"
    allowfullscreen></iframe>


Now we have one template from which devices can be "instantiated". All devices
based on it will accept messages via MQTT that are sent to
"/devices/thermometers" topic. To create new devices, you should go back to the
devices tab and create a new device, selecting the templates it will be based
on, as shown below.

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/RgeQPd7QSjg?rel=0" frameborder="0"
    allowfullscreen></iframe>

Note that, when you select the template in the right panel at device creation
screen, all attributes are inherited from that device. You could add more
templates as needed, keeping in mind that templates used to compose a device
must not share an attribute with the same name.

Now the physical devices can send messages to dojot. There a few things to pay
attention to: as we defined the MQTT topic (all devices will send to
``/devices/thermometer`` topic), the devices must identify themselves using the
``client-id`` parameter from MQTT protocol. Another way of doing that is to
just use the default topic scheme (which is ``/{SERVICE}/{DEVICE_ID}/attrs``.

Just for the sake of simplicity, we'll emulate one device using mosquitto_pub
tool. We set the ``client-id`` parameter by using the ``-i`` flag of
mosquitto_pub.

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/PwKMD02mH88?rel=0" frameborder="0"
    allowfullscreen></iframe>

Now that we've created the sensors, let's create a virtual one. This will be
the representation of a alarm system that will be triggered whenever something
bad is detected to these sensors. Let's say they are installed in a kitchen. So
it is expected that their temperature readings will be no more than 40˚C. If it
is more than that, our simple detection system will conclude that the kitchen is
on fire. This alarm representation will have two attributes: one for a severity
level for a particular alarm and another one for a textual message, so that the
user is properly informed of what's happening.

Just as for "regular devices", virtual devices also are based on templates. So,
let's create one, as shown below.

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/qb64f4PAhEo?rel=0" frameborder="0"
    allowfullscreen></iframe>

Once we've created the virtual device, we can add create a flow to implement
the logic behind the alarm generation. The idea is: if the temperature reading
is less than 40, then the alarm system will be updated with a notification of
severity 4 (mildly important) and a message indicating that the kitchen in OK.
Otherwise, if the temperature is higher than 40, then a notification is sent
with severity 1 (highest severity) and a message indicating that the kitchen is
on fire. This is done as shown belown.

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/7r5demA3rr0?rel=0" frameborder="0"
    allowfullscreen></iframe>

Note that the "change" nodes have a reference to an "output" entity. This can
be thought as an ordinary JavaScript variable - it will have a ``message`` and
a ``severity`` attributes that match those from the virtual device. This
"object" is referenced in the output node as a data source for the device to be
updated (in this case, the virtual device we've created).

So, let's send a few more messages and see what will happen to that virtual
device.

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/mXrgSfclxI0?rel=0" frameborder="0"
    allowfullscreen></iframe>


This is cool, but, how can it be used for real? How can it help develop your
application? That node palette, can new nodes be added to it? Well, a few of
these issues are detailed in the next tutorial.

Tracking devices
----------------

In this tutorial we will focus on how to track devices and how to use flows to
perform specific actions when a device reaches a particular area. Furthermore,
we will see a few more functionalities not covered by `Simple use`_.

The scenario we'll build in this tutorial will be as follows: a truck is
carrying a container with all sorts of packages from Campinas to Pouso Alegre
and we want to be notified when it exits Campinas, when passes by Extrema and
when it reaches its destination. The map is shown below to help you better
visualize where everything is located.

.. raw:: html

    <iframe
    src="https://www.google.com/maps/d/embed?mid=1l8wOtclvWyD0RpDezTTz-7nWNFUzAle7"
    width="640" height="480"></iframe>

So, as done before, let's create a template with all necessary parameters and a
new device based on it.


<< insert youtube video here >>

Note the ``position`` attribute, which will hold the GPS position informed by
the truck.

In order to properly generate the notification, let's create a flow that sends
a HTTP request to an external element and updates a virtual device which
represents one package that its is carrying. Let's create the virtual device
first.

<< insert youtube video here >>

Let's create the flow.

<< insert youtube video here >>

Note that we added ``geofence`` nodes which defines a circle that represents
the area of interest.



.. _User Guide: ../user_guide.html
.. _Flow Tutorial: ./flow.html