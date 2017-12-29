MQTT-TLS
==============

This document describes how to configure dojot to use MQTT over TLS.

.. contents:: Table of Contents
  :local:

tl;dr
---------

For a device to connect using TLS with Mosquitto, it must posses:
  - A key pair (.key file)
  - A certificate signed by a Certificate Authority (CA) trusted by Mosquitto (.crt file)
  - The certificate of this CA (.crt file)
  - An entry on Mosquitto Access Control List (ACL), allowing the device to publish on a specific topic.
  - (optional and not so necessary) A Certificate Revocation List (CRL)


When a device is created, Device-manager will (automatic) configure the following components:
  - IoTAgent -> will registry the new device on its internal cache.
  - MQTT-Manager -> Will create an entry on the ACL. Allowing the device to publish on a specific topic.
  - EJBCA -> Will create an end entity so a certificate can be created on the future.

By default, dojot use clear MQTT. To activate TLS, some changes must be done on docker-compose.

On docker-compose.yml
  - Change the image of the service 'mqtt' from 'ansi/mosquitto' to 'dojot/mqtt-manager'
  - Change the avalible door for 'mqtt' service from '1883:1883' to '8883:8883'
  - Change the value of the environment variable MQTT_TLS of the service service 'iotagent' to true (lowercase).

On the configuration file 'iotagent/config.json':
   - set the flag 'secure' to true

Components
----------

EJBCA-REST
***************

EJBCA (https://www.ejbca.org) is a complete Private Key Infrastructure (PKI) capable to manage  CAs, cryptography keys and certificates.
EJBCA provides a SOAP, web and a command line interface. EJBCA-REST is an wrapper on top of EJBCA that provides modern interfaces, like REST and Kafka.

What is a certificate?
======================


A certificate contains the public key for a entity (a user, device, website), together with a bunch of information about this entity, about the CA signing the certificate, the allowed certificate usage and a checksum.
When a entity want a certificate signed, the entity should create a CSR file and sent it to the desired CA.
The CSR file is an 'intention of certification'. The fie contains the information required from the entity and some information about the certificate use, hostnames and IPs where the certificate will reside, alternative names for the entity and etc.
EJBCA can decide, using its configured policies, what information to keep, to discard and to overwrite of the received CSR. EJBCA can refuse to sign a CSR not judged safe enough be it policies.

These configurable policies are called 'Certificate Profiles'. One Certificate profile named CFREE, specialized for MQTT TLS,  is provided out of the box.

In short, CFREE have the following configurations (and many more):
  - Cryptography keys must have between 2048 and 8192 bits.
  - Certificate expires is 730 days.
  - Entities can define hostnames and IPs
  - Key usage is marked as not critical (for now)
  - The hash algorithm is SHA256. The sign algorithm is RSA.


I have a CSR. How can I ask EJBCA to sign it for me?
====================================================

Calm down! EJBCA will not allow strangers to ask for certification. You need to authenticate yourself.
EJBCA use a username+password authentication system. The term 'end entity' will be used to refer to EJBCA users to follow the terms on EJBCA documentation and to avoid ambiguities between EJBCA users and dojot users.
An administrator should create the end entity. A just created end entity have the state 'New' an can generate a certificate. After sign a certificate for an entity, the end entity pass to the state 'Generated' and will no longer accept this username and password.
