Using MQTT with security (TLS)
==============================

.. note::
   - Audience: users
   - Level: intermediate
   - Reading time: 15 m


This document describes how to configure dojot to use MQTT over TLS when using the microservice `IotAgent-Mosca`_

.. contents:: Table of Contents
  :local:


For a device to connect using TLS with Mosca, it must possess:

-  A key pair (.key file);
-  A certificate signed by a Certificate Authority (CA) trusted by
   Mosca (.crt file);
-  The certificate of this CA (.crt file);


When a device is created, DeviceManager will automatically notify
the following components:

-  IoTAgent-Mosca: will register the new device on its internal cache and will create an entry, allowing the device to publish on a specific topic.
-  EJBCA: will create an end entity so a certificate can be created on
   the future.

Components
----------

EJBCA-REST
~~~~~~~~~~

`EJBCA`_ is a complete Private Key Infrastructure (PKI) capable to manage CAs,
cryptography keys and certificates. EJBCA provides a SOAP, web and a command
line interface. EJBCA-REST is an wrapper on top of EJBCA that provides modern
interfaces, like REST and Kafka.

EJBCA provides SOAP, web and command line interfaces. `EJBCA-Rest`_ is a wrapper
on top of EJBCA that complements those, allowing the CA to be configured using
REST. When used within dojot, it also listens to Kafka events, allowing its
automatic configuration.

What is a certificate?
^^^^^^^^^^^^^^^^^^^^^^

A certificate contains the public key for an entity (a user, device, website),
along with information about this entity, about the CA which signs the
certificate, the allowed certificate usage and a checksum. When a entity wants
a certificate to be signed, the entity should create a CSR file and send it to
the desired CA. The CSR file is an 'intention of certification'. The file
contains the information required from the entity and some information about
the certificate use, hostnames and IPs where the certificate will reside,
alternative names for the entity, etc. EJBCA can decide, using its configured
policies, what information to keep, to discard and to overwrite of the received
CSR. EJBCA can refuse to sign a CSR if it concludes that it is not safe enough
according to its policies.

These configurable policies are called 'Certificate Profiles'. One Certificate
profile named CFREE, specialized for MQTT TLS, is provided out of the box.

In short, CFREE have the following configurations (and many more):

-  Cryptography keys must have between 2048 and 8192 bits;
-  Entities can define hostnames and IPs;
-  Key usage is marked as not critical (for now);
-  The hash algorithm is SHA256. The sign algorithm is RSA.


So, how does EJBCA work in dojot?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When creating a new device, an associated end entity is created in EJBCA. Its
name will be the device's ID (like 'f60c28') and its password will be always
'dojot'.

A certificate can be signed by sending a HTTP POST request to
host:8000/sign/<cname>/pkcs10. CName is the end entity's name (or device). The
payload sent with this request should be a JSON containing the end entity
password and a CSR file (certificate intention) in base64 format.

Note that the URL is 'routed' by the API gateway. As in other APIs in dojot, a
JWT is needed. You can find how to generate and how to use such token in
:ref:`Using API interface`.

In order to create the CSR file and ask for a certificate signature, a user can
use a helper script called 'Certificate Retriever', which is detailed in
`Certificate retriever`_ section.

Mosca
~~~~~~~~~~~~
Mosca is a node.js mqtt broker. To using Mosca you need do some configurations by environment variable:

 - MOSCA_TLS_DNS_LIST: TLS DNS list, Servers hostnames, the host to connect external (separated by a comma). Example: localhost, mydomain.com

All the certificates will be created automatically,
not needing to configure manually the certificates into the broker.

Note: To use Mosca without TLS too, you need set the environment variable ALLOW_UNSECURED_MODE to 'true' and to use 1883 port. **It is not recommended!**

Certificate retriever
---------------------

This component is a helper script for device certificates creation. It
is available at `Certificate Retriever GitHub repository`_ and it
coded using Python 3.

A user can use it by executing:

.. code:: bash

   sudo apt install python3-pip #run on Debian-based Linux distributions, if necessary

   pip3 install crypto #or pip install crypto, run if necessary
   pip3 install pyOpenSSL #or pip install pyOpenSSL, run if necessary
   pip3 install requests #or pip install requests, run if necessary

   mkdir -p certs

And to finally get the certificate for the device:

.. code:: bash

    python3 generateLoginPwd.py  ${DOJOT_HOST} ${DEVICE_ID} IOTmidCA #run every time

The mandatory parameters are:

-  ${DOJOT_HOST}: where dojot is (No / at the end). Example: http://localhost:8000
-  ${DEVICE_ID}: device id that will get a new certificate. Example: f60c28

Note that authentication is performed in dojot. The script will ask for user
credentials and will invoke user authentication automatically. The user needs
permission for certificate signing to be able to use this script.

An end entity must exist in EJBCA in 'New' state before asking for a new
certificate signature. When a new device is created, an end entity is
automatically created in EJBCA by DeviceManager. This new end entity's name is
the device ID itself. Its password is 'dojot'.

The script authenticates users with given username and password, retrieves CA
certificate, generates a key pair as well as a CSR file and asks for
certificate signature, in this order. Any error in any step will halt its
execution.

After successfully executed, all certificates can be found in './certs'
folder.

Simulating a device with mosquitto
----------------------------------

To publish and subscribe using the appropriated certificates, you must need to be
with the Mosca Broker and the EJBCA running. After creating the dojot
environment, the templates and the devices, use the mosquitto emulate
a device and to publish and subscribe in the desired topics:


Before install mosquitto_pub and mosquitto_sub (from package `mosquitto-clients` on Debian-based Linux distributions) and access the folder certs, if necessary:

.. ATTENTION::
    Some Linux distributions, Debian-based Linux distributions in particular, have two packages for
    `mosquitto`_ - one containing tools to access it (i.e. mosquitto_pub and
    mosquitto_sub for publishing messages and subscribing to topics) and
    another one containing the MQTT broker too. In this tutorial, only the tools from package `mosquitto-clients` on Debian-based Linux distributions are going to be used.
    Please check if MQTT broker is not running before starting dojot
    (by running commands like ``ps aux | grep mosquitto``) to avoid port conflicts.

.. code:: bash

   sudo apt-get install mosquitto-clients   #if necessary on Debian-based Linux distributions
   cd certs  #if necessary

How to publish:

.. code:: bash

   mosquitto_pub  -h localhost -p 8883 -t /<tenant>/<deviceId>/attrs -i <tenant>:<deviceId> -m '{"attr_example": 10}' --cert <your .crt file> --key <your .key file> --cafile IOTmidCA.crt

How to subscribe:

.. code:: bash

   mosquitto_sub  -h localhost -p 8883 -t /<tenant>/<deviceId>/config -i <tenant>:<deviceId> --cert <your .crt file> --key <your .key file> --cafile IOTmidCA.crt


The <your .crt file>, <your .key file> and the cafile can be created with the `Certificate Retriever GitHub repository`_ script.
Where <tenant> is a context identifier into dojot and <deviceId> is a identifier for the device in the corresponding context.

Note: In this case, the message is a publication with an attribute, this attribute has the label `attr_example` and a new value 10, you need to change this for your case.


Important Notes
---------------

These are a few but important notes related to device security and
associated subjects.

Debugging
~~~~~~~~~

TLS errors might be not so verbose as other problems. If an error occurrs, the
user might not know what went wrong because no component indicates any problem.
In this section there are some tips, frequent problems and debugging tools to
find out what's happening.

How to read a certificate
^^^^^^^^^^^^^^^^^^^^^^^^^

A certificate file can be in two formats: PEM (base64 text) or DER
(binary). OpenSSL offers tools to read such formats:

.. code:: bash

    openssl x509 -noout -text -in certFile.crt



.. _EJBCA: https://www.ejbca.org
.. _Mosca repository: https://github.com/mcollina/mosca
.. _Certificate Retriever GitHub repository: https://github.com/dojot/certificate-retriever/tree/v0.4.3
.. _IotAgent-Mosca: https://github.com/dojot/iotagent-mosca/tree/v0.4.3
.. _EJBCA-Rest: https://github.com/dojot/ejbca-rest/tree/v0.4.3
.. _mosquitto: https://projects.eclipse.org/projects/technology.mosquitto