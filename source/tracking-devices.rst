Simple device tracking
======================

In this tutorial we will focus on how to track devices and how to use flows to
perform specific actions when a device reaches a particular area. Furthermore,
we will see a few more GUI functionalities not covered by `Using web
interface`_.

.. NOTE::
   - Who is this for: entry-level users
   - Level: basic
   - Reading time: X m


The scenario we'll build in this tutorial will be as follows: a truck is
carrying a container from Campinas to Pouso Alegre and we want to be notified
when it exits Campinas, when passes by Extrema and when it reaches its
destination. The map is shown below to help you better visualize where
everything is located.

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

.. _Using web interface: ./using-web-interface.html
