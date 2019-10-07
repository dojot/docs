Release history
===============

carate - 2019.09.11
-----------------------

- IoT agents:

  - Support for
    `LWM2M devices <https://github.com/dojot/iotagent-leshan>`_


- GUI:

  - New interface for devices management
  - New interface for templates management
  - Attribute Metadata Management
  - Import and export
  - Firmware update
  - Profiles
  - Notifications
  - Internationalization
  - View details with Actuator Attributes
  - Devices Filter on Map


- Flows:

  - New node Event Device
  - New node Event Template Device
  - New node FTP
  - New node Multi Device Out
  - New node Notification
  - New node Multi Actuate
  - New node Cron
  - New node Cron Batch
  - New node Cumulative Sum
  - New node Merge data
  - Internacionalization
  - Support to handlebars template on template node


- History:

  - New endpoint to query notifications


- ImageManager:

  - Improvements to support Firmware Update

- DataBroker:

  - Support notifications in socket-io

- DataManager:

  - It's new a dojot's microservice that manages
    the dojot's data configuration,
    making possible to import and export configuration.


- Cron:

  - It's new a dojot's microservice that allows
    you to schedule events to be emitted
    to other microservices.


- New libraries:

  - To accelerate development:
    `Dojot Module Java <https://github.com/dojot/dojot-module-java>`_
    and `Dojot Module Python <https://github.com/dojot/dojot-module-python>`_
  - HealthCheck:
    `HealthCheck Python <https://github.com/dojot/healthcheck-python>`_
    e `Healthcheck NodeJs <https://github.com/dojot/healthcheck-nodejs>`_
  - IoTAgent:
    `IoTAgent Java <https://github.com/dojot/iotagent-java>`_


