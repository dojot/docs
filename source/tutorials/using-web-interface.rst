Using dojot's web interface
===========================

This tutorial will show how to do basic operations in dojot, such as creating
devices, checking its attributes and creating flows.

Device management
-----------------

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

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/qb64f4PAhEo?rel=0" frameborder="0"
    allowfullscreen></iframe>

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/7r5demA3rr0?rel=0" frameborder="0"
    allowfullscreen></iframe>

.. raw:: html

    <iframe id="ytplayer" type="text/html" width="720" height="405"
    src="https://www.youtube.com/embed/mXrgSfclxI0?rel=0" frameborder="0"
    allowfullscreen></iframe>


.. _User Guide: ../user_guide.html
