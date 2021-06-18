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

MQTT Broker
~~~~~~~~~~~

IotAgent VerneMQ
^^^^^^^^^^^^^^^^

The IotAgent VerneMQ extends `VerneMQ`_ with some features and services
for dojot case, see more in `IotAgent-VerneMQ`_.
This is the **default** Broker MQTT in dojot deployments.

All the certificates will be automatically created, so there is no need to manually configure the brokers' ones.

.. ATTENTION::
   In order for TLS to work correctly it is necessary
   to configure the domain in which the ``deployment`` is running.
   In `ansible (Kubernetes)`  this can be set in the ``dojot_domain_name`` variable
   and in `docker-compose` in the ``DOJOT_DOMAIN_NAME`` variable in *.env*.
   In `ansible (Kubernetes)` you can disable the *unsecured mode* by changing
   the ``dojot_insecure_mode`` variable to ``false``
   and in `docker-compose` unmapping to port 1883 in the `iotagent-mqtt` service.
   Check the :doc:`./installation-guide` for more info.


How to connect a device with the IotAgent VerneMQ
-------------------------------------------------

.. ATTENTION::
   First of all, you will need a running dojot's environment.
   Then you should create a new device (or use an existing one if you prefer)
   and retrieve its ID.

Retrieving certificate for a device
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For a device to connect using MQTT over TLS (MQTTS), it must possess:

-  A key pair (.key file);
-  A certificate signed by a Certificate Authority (CA) trusted by
   dojot (.crt file);
-  The certificate of this CA (.crt file);

The objective when retrieving the certificate for a device is to
obtain these three files: the device's certificate, the device's key pair and the CA certificate.

There are two tools to facilitate
obtaining certificates from the dojot platform:

- The `CertReq`_ script.
- GUI's embedded certificate generation utility (more details in :ref:`Generating certificates for devices`).

In addition, you can use `OpenSSL`_ to create
certificates and sign using the `API - x509-identity-mgmt`_,
see more  at `X.509 Identity Management`_.

A brief explanation of how to use `CertReq`_
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As prerequisites this uses `git`_, `OpenSSL`_, `curl`_ and `jq`_ .

.. NOTE::
   You need to enable the `dev-test-cli` client in the keycloak. For
   security reasons it is disabled by default, after use it is
   recommended to disable it again :ref:`How to enable and disable *client* `dev-test-cli``.


On Debian-based Linux distributions, you can install
these prerequisites by running:

.. code-block:: bash

  sudo apt install git curl jq openssl

Download `CertReq`_  on your machine directly from dojot repository and switch
to the corresponding version of your dojot environment:

.. code:: shell

  git clone https://github.com/dojot/dojot.git
  cd dojot
  git checkout v0.8.0

Enter in ``certreq`` directory:

.. code:: shell

  cd tools/certreq

Finally, you can run the script to generate
certificates and keys as follows:

.. code:: shell

      ./bin/certreq.sh \
         -h http://localhost \
         -p 8000 \
         -i 'a1998e' \
         -t 'admin' \
         -u 'admin' \
         -s 'admin'

Given a *username* ``admin``,  *password* ``admin`` and *tenant* ``admin``,
this command will request a certificate
with *device ID (identifier)* ``a1998e``
for the dojot platform *host* ``http://localhost`` on *port* ``8000``.


.. NOTE::   For more details about the CertReq parameters, check
            the `CertReq - Parameters`_ document.
            Other useful resources for this matter are
            the `How to connect a device with the IoTAgent-VerneMQ with mutual TLS`_
            tutorial and the `CertReq`_ documentation.

And in the end this tool will create the directories ``./ca`` and
``./cert_{DEVICE_ID}`` to store the certificates
and public/private keys.


Simulating a device with mosquitto
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We are going to use `mosquitto` to simulate a device;
it will publish and subscribe in dojot via MQTT.

Before continuing, install `mosquitto_pub` and `mosquitto_sub` from
the `mosquitto-clients` package on Debian-based Linux distributions:

.. ATTENTION::
    Some Linux distributions, Debian-based Linux distributions in particular,
    have two packages for
    `mosquitto`_, one containing tools to access it (i.e. mosquitto_pub and
    mosquitto_sub for publishing messages and subscribing to topics) and
    another one containing the MQTT broker too. In this tutorial, only the
    tools from package `mosquitto-clients` on Debian-based Linux distributions
    are going to be used.
    Please check if another MQTT broker is not running before starting dojot
    (by running commands like ``ps aux | grep mosquitto``) to avoid port conflicts.

On Debian-based Linux distributions you can install
``mosquitto-clients``  running:

.. code:: bash

   sudo apt-get install mosquitto-clients


IotAgent VerneMQ (Default)
^^^^^^^^^^^^^^^^^^^^^^^^^^

To publish and subscribe using the appropriate certificates,
you must have IotAgent VerneMQ Broker, V2K Bridge,
K2V Bridge and the X.509 Identity Management running,
see more in `IotAgent-VerneMQ`_.

Simulating a device publishing with mosquitto

.. code:: bash

   mosquitto_pub  -h localhost -p 8883 -t <tenant>:<deviceId>/attrs  -m '{"attr_example": 10}' --cert <device .crt file> --key <device .key file> --cafile <ca .crt file>

An example of publication with the certificates and
keys generated in the previous topic with `CertReq`_ tool.

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

IotAgent Mosca (Legacy)
^^^^^^^^^^^^^^^^^^^^^^^

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


.. NOTE::
   In these examples, the published message has the attribute `attrs_example`.
   You need to change its name to comply to your device's attribute.

.. _Unsecured mode MQTT:

Unsecured mode MQTT (without TLS)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. ATTENTION::
   MQTT without security is not recommended, use this for testing only.


How to read a certificate
-------------------------

A certificate file can be in two formats: PEM (base64 text) or DER
(binary). `OpenSSL`_ offers tools to read such formats:

Reading certificate:

.. code:: bash

    openssl x509 -noout -text -in certFile.crt

Getting certificate fingerprint:

.. code:: bash

   openssl x509 -in certFile.crt -noout -fingerprint -sha256


.. _OpenSSL: https://www.openssl.org/
.. _curl: https://curl.haxx.se/
.. _jq: https://stedolan.github.io/jq/
.. _git: https://git-scm.com/
.. _VerneMQ: https://vernemq.com/
.. _IotAgent-VerneMQ: https://github.com/dojot/dojot/tree/v0.8.0/connector/mqtt/vernemq
.. _X.509 Identity Management: https://github.com/dojot/dojot/tree/v0.8.0/x509-identity-mgmt
.. _VerneMQ Dojot - Environment variables: https://github.com/dojot/dojot/tree/v0.8.0/connector/mqtt/vernemq/broker#environment-variables
.. _mosquitto: https://projects.eclipse.org/projects/technology.mosquitto
.. _How to connect a device with the IoTAgent-VerneMQ with mutual TLS: https://github.com/dojot/dojot/tree/v0.8.0/connector/mqtt/vernemq#how-to-connect-a-device-with-the-iot-agent-mqtt-with-mutual-tls
.. _Simulating a device with mosquitto: https://github.com/dojot/dojot/tree/v0.8.0/connector/mqtt/vernemq#simulating-a-device-with-mosquitto
.. _Simulating a device with mosquitto with security: https://github.com/dojot/dojot/tree/v0.8.0/connector/mqtt/vernemq#with-security
.. _Simulating a device with mosquitto without security: https://github.com/dojot/dojot/blob/v0.8.0/connector/mqtt/vernemq/README.md#without-security
.. _API - x509-identity-mgmt: https://dojot.github.io/dojot/x509-identity-mgmt/apiary_v0.8.0.html
.. _CertReq: https://github.com/dojot/dojot/tree/v0.8.0/tools/certreq
.. _CertReq - Parameters: https://github.com/dojot/dojot/tree/v0.8.0/tools/certreq#parameters
.. _IotAgent-Mosca: https://github.com/dojot/iotagent-mosca/tree/v0.8.0
.. _Mosca: https://github.com/mcollina/mosca


