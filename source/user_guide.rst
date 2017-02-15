User Guide
==========

This document provides information on how to use the IoT Middleware from a user perspective. On that
regard, this should describe the steps required to install and operate the platform from a device
developer or application developer point of view. For documentation regarding the operation of the
Middleware itself, please refer to the [operations guide]()

.. contents:: Table of Contents
  :local:

Getting Started
---------------

The idea here is to present the reader with a sample end-to-end usage of the middleware, showcasing
a relevant set of the features implemented by the middleware, thus serving as an example of how to
use the middleware on a "real world" scenario.

To make things easier for the user, there may be a VM image with a pre-configured deployment of the
middleware. Should this be the case, instructions on how to deploy the solution might reside only on
the Operations Guide.

Device Management
-----------------

This section should detail how to integrate a new device with the system. That should encompass
the both the communication requirements imposed on the device in order to allow its usage with
the middleware, as well as the steps (if any, depending on the protocol used) to configure this
new device within the middleware.

This could also explain (if indeed implemented) the device management functionalities made available
by the middleware to the device developer.

Regarding the requirements imposed on the devices, it is forseen that, for each communication scheme
(protocol/serialization format) offically supported by the middleware, a step by step guide on
how to "develop" a device is supplied. Such guide can, if applicable, make use of a middleware-provided
library or SDK.

Flow Management
---------------

Moving to the perspective of an aplication developer, this section should list and explain the usage
of the information flow configuration process within the middleware - how to use the provided gui,
high level description of the APIs that can be used to configure such flows, available actions to
be used when building the flows, so on and so forth.
