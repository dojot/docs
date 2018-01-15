APIs
====

These are all APIs exposed by dojot.

.. list-table:: APIs
   :header-rows: 1

   * - Endpoint
     - Purpose
     - Component
     - Repository
   * - /device
     -  Device management
     - `DeviceManager`_
     - `GitHub - device-manager`_
   * - /template
     -  Template management
     - `DeviceManager`_
     - `GitHub - device-manager`_
   * - /flows
     -  Flow management
     - mashup
     - `GitHub - mashup`_
   * - /auth
     -  User authentication
     - `auth`_
     - `GitHub - auth`_
   * - /auth/revoke
     -  User authentication
     - `auth`_
     - `GitHub - auth`_
   * - /auth/user
     -  User authentication
     - `auth`_
     - `GitHub - auth`_
   * - /history
     -  Device historical data
     - `STH`_
     - `GitHub - STH`_
   * - /metric
     -  Context broker
     - `Orion v1`_, `Orion v2`_
     - `GitHub - Orion`_
   * - /gui
     -  Graphical User Interface
     - GUI
     - `GitHub - GUI`_
   * - /sign
     -  Public key signing
     - ejbca
     - `GitHub - ejbca-rest`_
   * - /ca
     -  Certification-Auth. functions
     - ejbca
     - `GitHub - ejbca-rest`_


The API gateway used in dojot reroutes some of these endpoints so that they become uniform: all of them are accessible through the same port (default is TCP port 8000) and have the same naming scheme. Each component, though, might have something different in its configuration and API documentation. The following table shows which endpoint exposed by the API gateway is mapped to which component endpoint.

.. list-table:: Original endpoints
   :header-rows: 1

   * - Service
     - Original endpoint
     - Endpoint
   * - `DeviceManager`_
     - host:5000/device
     - host:8000/device
   * - `DeviceManager`_
     - host:5000/template
     - host:8000/template
   * - mashup
     - host:3000/
     - host:8000/flows
   * - `auth`_
     - host:5000/
     - host:8000/auth
   * - `auth`_
     - host:5000/auth/revoke
     - host:8000/auth/revoke
   * - `auth`_
     - host:5000/user
     - host:8000/auth/user
   * - `STH`_
     - host:8666/
     - host:8000/history
   * - `Orion v1`_ or `Orion v2`_
     - host:1026/
     - host:8000/metric
   * - GUI
     - host/
     - host:8000/gui
   * - ejbca
     - host:5583/
     - host:8000/sign
   * - ejbca
     - host:5583/
     - host:8000/ca



.. _DeviceManager:  https://dojot.github.io/device-manager/apis.html
.. _GitHub - device-manager: https://github.com/dojot/device-manager
.. _GitHub - mashup: https://github.com/dojot/mashup
.. _auth: https://dojot.github.io/auth/apis.html
.. _GitHub - auth: https://github.com/dojot/auth
.. _STH: https://github.com/telefonicaid/fiware-sth-comet/blob/master/doc/manuals/raw-data-retrieval.md
.. _GitHub - STH: https://github.com/telefonicaid/fiware-sth-comet
.. _Orion v1: http://telefonicaid.github.io/fiware-orion/api/v1/
.. _Orion v2: http://telefonicaid.github.io/fiware-orion/api/v2/stable/
.. _GitHub - Orion: https://github.com/dojot/fiware-orion
.. _GitHub - GUI: https://github.com/dojot/gui
.. _GitHub - ejbca-rest: https://github.com/dojot/ejbca-rest
