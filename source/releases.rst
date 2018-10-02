Release history
===============

battojutsu - 2018.10.03
-----------------------


- IoT agents:

  - Support for `sigfox devices <http://github.com/dojot/iotagent-sigfox>`_
  - Support for `LoRa devices <http://github.com/dojot/iotagetn-lora-everynet>`_ to be
    used with EveryNet networks.
  - Many improvements for IoT agent MQTT - performance, stability and documentation.

- GUI:

  - Map overlays
  - Pin color configuration on maps
  - Support for more screen resolutions
  - Filters (devices, templates)
  - Improved pagination

- Flows:

  - Support for global contexts: a new service, called ContextManager, was created
    to deal with contexts within a flow. They can be thought as data chunks that
    can be stored and retrieved by ContextManager when invoked within a flow node.
    They are split into four different access levels: tenant, flow, node and node
    instance. Check `flowbroker node library <https://github.com/dojot/flowbroker/blob/master/lib/ContextManagerClient.js>`_
    to check how to use context within nodes or check `flowbroker's get-context node <https://github.com/dojot/flowbroker/tree/master/orchestrator/nodes/get-context>`_
    to use it directly from flowbroker GUI you could just open the new
    flowbroker UI and check it in the node palette)
  - New configuration options for device actuation: send actuation message to the
    same device that triggered the flow or set which is the targed device dynamically,
    set while the flow is being processed.
  - Support for device information caching (improving performance)

- History:

  - Support for queries that retrieve all attributes from a particular device (
    without explicitly selecting which one should be returned). Check `history API <https://dojot.github.io/history/apiary_v1.0.0.html>`_
    for more information.

- DeviceManager:

  - Now DeviceManager is able to generate a random key for devices (PSK)

- New libraries:

  - New library for `dojot modules <https://github.com/dojot/dojot-module-nodejs>`_ to accelerate development.
  - New `log library <https://github.com/dojot/dojot-module-logger-nodejs>`_ to standardize all service logs.
  




