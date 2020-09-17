Integrating physical devices
============================


.. note::
   - Who is this for: developers
   - Level: intermediate


If you want to integrate your device within dojot, it must be able to send
messages to the platform. There are two ways to do that:

- Use one of the available IoT agents: currently, there is support for
  MQTT-based devices. If your project is using (or allows changing to) this
  protocol, then it would suffice to check if the device is sending its data
  using a simple key/value JSON. If it isn't, then you might want to use
  iotagent-mosca (check `iotagent-mosca`_ documentation to check out how to do
  that). If it is indeed sending key/value JSON messages, then it can send its
  messages to dojot's broker and it will be recognized by the platform.

- Create a new IoT agent to support the protocol used by the device: if your
  device is using another protocol that is not yet supported, then it might be
  a good idea to implement a new IoT agent. It's not that hard, but there are a
  few details that must be taken into account. To help developers to do such
  thing, there is the `iotagent-nodejs`_ library which deals with most
  internal mechanisms and messages - check its documentation to know more.


See more about :doc:`mqtt-tls`.

.. _iotagent-mosca: https://github.com/dojot/iotagent-mosca
.. _iotagent-nodejs: https://github.com/dojot/iotagent-nodejs
