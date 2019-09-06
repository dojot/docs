MQTT-TLS Tutorial
=================

This document describes how to configure dojot to use MQTT over TLS.

.. contents:: Table of Contents
  :local:

tl;dr
-----

For a device to connect using TLS with Mosca, it must possess:

-  A key pair (.key file);
-  A certificate signed by a Certificate Authority (CA) trusted by
   Mosca (.crt file);
-  The certificate of this CA (.crt file);
-  An entry on Mosca Access Control List (ACL), allowing the device
   to publish on a specific topic;

When a device is created, DeviceManager will automatically notify
the following components:

-  IoTAgent-Mosca: will register the new device on its internal cache and will create an entry 
   on the ACL, allowing the device to publish on a specific topic.
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

EJBCA provides SOAP, web and command line interfaces. EJBCA-REST is a wrapper
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
-  Certificate expires in 730 days;
-  Entities can define hostnames and IPs;
-  Key usage is marked as not critical (for now);
-  The hash algorithm is SHA256. The sign algorithm is RSA.

I have a CSR. How can I ask EJBCA to sign it for me?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Calm down! EJBCA will not allow strangers to ask for certification. You need to
authenticate yourself. EJBCA use a username+password authentication system. The
term 'end entity' will be used to refer to EJBCA users to follow the terms on
EJBCA documentation and to avoid ambiguities between EJBCA users and dojot
users. An administrator should create the end entity. An entity that was just
created has the state 'New' an can generate a certificate. After signing a
certificate for an entity, the end entity's state changes to 'Generated' and
will no longer accept this username and password. EJBCA 'End entities' can
create only one certificate.

So, how does EJBCA work in dojot?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When creating a new device, an associated end entity is created in EJBCA. Its
name will be the device's ID (like '8fa3') and its password will be always
'dojot'.

A certificate can be signed by sending a HTTP POST request to
host:1234/sign/<cname>/pkcs10. CName is the end entity's name (or device). The
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
Mosca is a node.js mqtt broker. To use the Mosca broker with TLS, you need to allow the broker
to use TLS as the secure mode of connection. To do this, you need to declare the credentials and
the secure attributes in the Mosca server conf object. You can enable the TLS via configuration variable. 

.. code-block:: JavaScript

    if (config.mosca_tls === 'true') {

    var SECURE_CERT = '/opt/mosca/certs/mosquitto.crt';
    var SECURE_KEY =  '/opt/mosca/certs/mosquitto.key';
    var CA_CERT = '/opt/mosca/certs/ca.crt';

    //Mosca with TLS
    moscaSettings = {
        backend: mosca_backend,
        persistence: {
        factory: mosca.persistence.Redis,
        host: mosca_backend.host
        },
        type : "mqtts", // important to only use mqtts, not mqtt
        credentials :
        { // contains all security information
            keyPath: SECURE_KEY,
            certPath: SECURE_CERT,
            caPaths : [ CA_CERT ],
            requestCert : true, // enable requesting certificate from clients
            rejectUnauthorized : true // only accept clients with valid certificate
        },
        secure : {
            port : 8883  // 8883 is the standard mqtts port
        }
    }
    
    ...

All the certificates will be created automatically, 
not needing to configure manually the certificates into the broker.

MQTT configuration files
-----------------------------

Checkout this commented MQTT configuration file:

.. code:: ini

    # network port on which MQTT will accept new connections
    port 8883

    # Trusted CA certificate
    cafile //opt/mosca/certs/ca.crt

    # MQTT certificate
    certfile /opt/mosca/certs/mosquitto.crt

    # MQTT key par
    keyfile /opt/mosca/certs/mosquitto.key

    # Permission list file
    acl_file /opt/mosca/certs/access.acl

    # CA CRL.
    crlfile /opt/mosca/certs/ca.crl

Note that for all configuration updates, it is mandatory to restart
Mosca broker or to send a SIGDUP signal to its process.

Certificate retriever
---------------------

This component is a helper script for device certificates creation. It
is available at `Certificate Retriever GitHub repository`_ and it
coded using Python 3.

A user can use it by executing:

.. code:: bash

    ./certificate-retriever.py HOST DEVICE-NAME CA [OPTIONS]

The mandatory parameters are:

-  HOST: where dojot is. Example: http://localhost:8000
-  DEVICE-NAME: device name that will get a new certificate. Example:
   ac32
-  CA: CA which will sign the certificate. Example: IOTmidCA (this is
   the CA name used in dojot)

Other options are:

-  -u or --username USERNAME: dojot's username. If this parameter is not
   specified here, it will be asked iteratively.
-  -w or --overwrite: overwrites any certificate files or criptographic
   keys if already existent.
-  -k or --key KEYLENGTH: size of the criptographic key being generated
   (in bits).
-  -d or --dns: Hostname where the certificate owner can be reached out.
   Note that this has no relation with DNS (Domain Name System) servers
   - this name was kept because x509 certificates have an attribute that
   is called DNS.
-  -i or --ip: same as -d, buto to specify IP address.
-  --skip-https-check: if dojot accepts HTTPS connections but it has no
   valid certificate, then this option will allow the connection to be
   made.

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

Quick Tutorial
--------------

To publish using the appropriated certificates, you must need to be 
with the Mosca Broker and the EJBCA running. After creating the dojot 
environment, the templates and the devices, use the mosquitto to publish 
in the desired topic:

.. code:: bash

     mosquitto_pub -t <topic> -i <admin:deviceId> -m <message> -p 8883 --cert <your .crt file> --key <your .key file> --cafile IOTmidCA.crt

The .crt, .key and the .cafile can be created with the `Certificate Retriever GitHub repository`_ script.


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


How to verify if an entity is created in EJBCA
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can check if a entity (device) is created in the EJBCA by checking the EJBCA log:

.. code:: bash

    user 3b987 created

in the example above, we created a device with id 3b987. After the device was created, 
the ejbca add the device has an entity.

How to verify if a device has published in the topic
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
You can check if your device has successfully published into Mosca broker by checking the Mosca log:

.. code:: bash

    Published /devices/termo { temperature: 62.4 } admin:87852f undefined undefined

if a message like this did not appear, there was probably a failure to authenticate the certificates. 
Try to recreate the certificates with the `Certificate Retriever GitHub repository`_ script.

.. _EJBCA: https://www.ejbca.org 
.. _User Guide: http://dojotdocs.readthedocs.io/en/latest/user_guide.html#first-steps
.. _Mosca repository: https://github.com/mcollina/mosca
.. _Certificate Retriever GitHub repository: https://github.com/dojot/certificate-retriever
