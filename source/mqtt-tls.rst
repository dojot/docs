Using MQTT with security (TLS)
==============================

.. note::
   - Audience: users
   - Level: intermediate
   - Reading time: 15 m

This document describes how to configure the dojot and
publish/subscribe using MQTT over TLS (MQTTS)
when using the microservice Broker MQTT
`IotAgent-VerneMQ`_ or Legacy Broker MQTT `IotAgent-Mosca`_.

.. contents:: Table of Contents
  :local:



Components
----------

X.509 Identity Management
~~~~~~~~~~~~~~~~~~~~~~~~~

The purpose of this component is to provide x.509 identification for
final dojot entities, that is, IoT devices that communicate with the
dojot IoT platform. See more about in `X.509 Identity Management`_.

What is a certificate?
^^^^^^^^^^^^^^^^^^^^^^

A certificate contains the public key for an entity (a user, device, website),
along with information about this entity, about the CA which signs the
certificate, the allowed certificate usage and a checksum. When a entity wants
a certificate to be signed, the entity should create a CSR file and send it to
the desired CA. The CSR file is an 'intention of certification'. The file
contains the information required from the entity and some information about
the certificate use, hostnames and IPs where the certificate will reside,
alternative names for the entity, etc.

Brokers MQTT
~~~~~~~~~~~~

Broker MQTT IotAgent VerneMQ (Default)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The IotAgent VerneMQ extends `VerneMQ`_ with some features and services
for dojot case, see more in `IotAgent-VerneMQ`_.
This is the **default** Broker MQTT in dojot deployments.

To use IotAgent VerneMQ with TLS, you need to configure by environment variable:

 - EXTERNAL_SERVER_HOSTNAME: Server hostname/ip (the host to which the device connects externally).
   Example: mydomain.com. By default is *localhost*.

See others environment variables in `VerneMQ Dojot - Environment variables`_.

Broker MQTT IotAgent Mosca (Legacy)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The IotAgent Mosca uses `Mosca`_ that is a node.js mqtt
broker,
see more about this service in `IotAgent-VerneMQ`_.
This is the **not default** Broker MQTT in dojot deployments.

To use IotAgent Mosca with TLS, you need to configure by environment variable:

 - MOSCA_TLS_DNS_LIST: Server hostnames/ip,
   the host to which the device connects externally (separated by a comma).
   Example: localhost, mydomain.com


About using IotAgent VerneMQ or IotAgent Mosca
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You shouldn't use the two brokers together to avoid port conflicts,
or use `Broker MQTT IotAgent VerneMQ` or
use `Legacy Broker MQTT IotAgent Mosca`.
By default our deployments are using `IotAgent VerneMQ`.

In *ansible-dojot* (kubernetes deployment) there is a variable ``dojot_domain_name``,
if this variable is defined, you don't need to configure the environment variable
mentioned in the previous topics, this is valid in both Brokers.
Still in  *ansible-dojot* you can disable the `unsecured mode`
by changing the variable ``dojot_insecure_mqtt`` to ``false``,
this is valid in both Brokers too, we will explain about the `unsecured mode`
in the next topics.
Check the :doc:`./installation-guide` for more info.

In addition, you can choose between `IotAgent VerneMQ` or `IotAgent Mosca`
when configuring *ansible-dojot*.
In *docker-compose*, you need to uncomment and comment the services
in the ``docker-compose.yml``, there's a commented explanation about that in this file.
Check the :doc:`./installation-guide` for more info.

All the certificates will be created automatically,
not needing to configure manually the certificates into the brokers.

How to connect a device with the IotAgent VerneMQ or IotAgent Mosca with mutual TLS
------------------------------------------------------------------------------------

.. ATTENTION::
   First of all, you will need a running dojot environment
   And then you need created device on the dojot
   and gets its identifier (ID).

Retrieving certificate for a device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a device to connect using MQTT over TLS (MQTTS), it must possess:

-  A key pair (.key file);
-  A certificate signed by a Certificate Authority (CA) trusted by
   VerneMQ (.crt file);
-  The certificate of this CA (.crt file);

The final objective when retrieving the certificate for a device is to
obtain this three files,
certificates and key pair, mentioned above.

There are two tools to facilitate
obtaining certificates from the dojot platform:

- There is a script, see more in `CertReq`_.
- And t there's a feature in the web interface (GUI) to generate
  certificates easily, see more in A_SER_CRIADO.

In addition, you can use `OpenSSL`_ to create
certificates and sign using the `API - x509-identity-mgmt`_,
see more  at `X.509 Identity Management`_.

A brief explanation of how to use `CertReq`_:
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As prerequisites this uses `git`_, `OpenSSL`_, `curl`_ and `jq`_ .

On Debian-based Linux distributions, you can install
these prerequisites by running:

.. code-block:: bash

  sudo apt install git
  sudo apt-get install curl
  sudo apt-get install jq
  sudo apt-get install openssl

Download `CertReq`_  on your machine directly from dojot repository and switch
to the corresponding version of your dojot environment:

.. code:: shell

  git clone https://github.com/dojot/dojot.git
  cd dojot
  git checkout v0.5.0

Enter in ``certreq`` directory:

.. code:: shell

  cd tools/certreq

Finally, you can run the script to generate
certificates and keys as follows:

.. code:: shell

      ./bin/certreq.sh \
         -h localhost \
         -p 8000 \
         -i 'a1998e' \
         -u 'admin' \
         -s 'admin'

Given a *username* ``admin`` and *password* ``admin``,
this command will request a certificate
with *device ID (identifier)* ``a1998e``
for the dojot platform *host* ``localhost`` on *port* ``8000``.

.. NOTE::  For more understanding about parameters above, check `CertReq - Parameters`_, and for
            see more details about this above process access
            `How to connect a device with the IoTAgent-VerneMQ with mutual TLS`_
            and `CertReq`_

And in the end this tool will create the directories ``./ca`` and
``./cert_{DEVICE_ID}`` to store the certificates
and public/private keys.


Simulating a device with mosquitto
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can use the mosquitto to emulate a device to publish and
subscribe in dojot via MQTT.

Before install `mosquitto_pub` and `mosquitto_sub` from
package `mosquitto-clients` on Debian-based Linux distributions:

.. ATTENTION::
    Some Linux distributions, Debian-based Linux distributions in particular,
    have two packages for
    `mosquitto`_, one containing tools to access it (i.e. mosquitto_pub and
    mosquitto_sub for publishing messages and subscribing to topics) and
    another one containing the MQTT broker too. In this tutorial, only the
    tools from package `mosquitto-clients` on Debian-based Linux distributions
    are going to be used.
    Please check if MQTT broker is not running before starting dojot
    (by running commands like ``ps aux | grep mosquitto``) to avoid port conflicts.

On Debian-based Linux distributions you can install
``mosquitto-clients``  running:

.. code:: bash

   sudo apt-get install mosquitto-clients


Broker MQTT IotAgent VerneMQ (Default)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To publish and subscribe using the appropriate certificates,
you must have IotAgent VerneMQ Broker, V2K Bridge,
K2V Bridge and the X.509 Identity Management running,
see more in `IotAgent-VerneMQ`_.

Simulating a device publishing with mosquitto

.. code:: bash

   mosquitto_pub  -h localhost -p 8883 -t <tenant>:<deviceId>/attrs  -m '{"attr_example": 10}' --cert <device .crt file> --key <device .key file> --cafile <ca .crt file>

An example of publication with the certificates and
keys generated in the previous topic with tool `CertReq`_.

.. code:: bash

   mosquitto_pub \
   -h localhost \
   -p 8883 \
   -t admin:a1998e/attrs \
   -m '{"attr_example": 10 }' \
   --cert './cert_a1998e/cert.pem' \
   --key './cert_a1998e/private.key' \
   --cafile './ca/ca.pem'

Simulating a device subscribing with mosquitto

.. code:: bash

   mosquitto_sub  -h localhost -p 8883 -t <tenant>:<deviceId>/config  --cert <device .crt file> --key <device .key file> --cafile <ca .crt file>

For more details about simulate a device see
in `Simulating a device with mosquitto`_
and more about simulate a device with security
in `Simulating a device with mosquitto with security`_.

Broker MQTT IotAgent Mosca (Legacy)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To publish and subscribe using the appropriated certificates,
you must need to be with the IotAgent Mosca Broker and
the X.509 Identity Management running,
see more in `IotAgent-Mosca`_.
In addition, you need to use a **different topic** from VerneMQ
and pass the identifier to publish and subscribe, as follows:

How to publish:

.. code:: bash

   mosquitto_pub  -h localhost -p 8883 -t /<tenant>/<deviceId>/attrs -i <tenant>:<deviceId> -m '{"attr_example": 10}' --cert <device .crt file> --key <device .key file> --cafile <ca .crt file>

How to subscribe:

.. code:: bash

   mosquitto_sub  -h localhost -p 8883 -t /<tenant>/<deviceId>/config -i <tenant>:<deviceId> --cert <device .crt file> --key <device .key file> --cafile <ca .crt file>


Note: In all those examples above, the message is a publication
with an attribute, this attribute has the label `attr_example`
and a new value 10, you need to change this for your case.

Unsecured mode MQTT (without TLS)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**MQTT without security is not recommended, use this for testing only.**

In *ansible-dojot* (kubernetes deployment)  you can disable the `unsecured mode`
by changing the ``dojot_insecure_mqtt`` variable to ``false``,
this is valid in both Brokers.
Check the :doc:`./installation-guide` for more info.


Broker MQTT IotAgent VerneMQ (Default)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can disable the ``unsecured mode`` if you make port 1883 unavailable for external access.

See more about simulate a device without security
in `Simulating a device with mosquitto without security`_.

Broker MQTT IotAgent Mosca (Legacy)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can disable the ``unsecured mode`` in Mosca too
for that you need set the environment variable
ALLOW_UNSECURED_MODE to ``'false'`` or the port 1883 shouldn't to
be available for external access.
See more in `IotAgent-Mosca`_.

How to read a certificate
-------------------------

A certificate file can be in two formats: PEM (base64 text) or DER
(binary). `OpenSSL`_ offers tools to read such formats:

.. code:: bash

    openssl x509 -noout -text -in certFile.crt


.. _OpenSSL: https://www.openssl.org/
.. _curl: https://curl.haxx.se/
.. _jq: https://stedolan.github.io/jq/
.. _git: https://git-scm.com/
.. _VerneMQ: https://vernemq.com/
.. _IotAgent-VerneMQ: https://github.com/dojot/dojot/tree/v0.5.0/connector/mqtt/vernemq
.. _X.509 Identity Management: https://github.com/dojot/dojot/tree/v0.5.0/x509-identity-mgmt
.. _VerneMQ Dojot - Environment variables: https://github.com/dojot/dojot/tree/v0.5.0/connector/mqtt/vernemq/broker#environment-variables
.. _mosquitto: https://projects.eclipse.org/projects/technology.mosquitto
.. _How to connect a device with the IoTAgent-VerneMQ with mutual TLS: https://github.com/dojot/dojot/tree/v0.5.0/connector/mqtt/vernemq#how-to-connect-a-device-with-the-iot-agent-mqtt-with-mutual-tls
.. _Simulating a device with mosquitto: https://github.com/dojot/dojot/tree/v0.5.0/connector/mqtt/vernemq#simulating-a-device-with-mosquitto
.. _Simulating a device with mosquitto with security: https://github.com/dojot/dojot/tree/v0.5.0/connector/mqtt/vernemq#with-security
.. _Simulating a device with mosquitto without security: https://github.com/dojot/dojot/blob/v0.5.0/connector/mqtt/vernemq/README.md#without-security
.. _API - x509-identity-mgmt: https://dojot.github.io/dojot/x509-identity-mgmt/apiary_v0.5.0.html
.. _CertReq: https://github.com/dojot/dojot/tree/v0.5.0/tools/certreq
.. _CertReq - Parameters: https://github.com/dojot/dojot/tree/v0.5.0/tools/certreq#parameters
.. _IotAgent-Mosca: https://github.com/dojot/iotagent-mosca/tree/v0.5.0
.. _Mosca: https://github.com/mcollina/mosca


