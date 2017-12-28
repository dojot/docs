Crypto Service
==============

Crypto Service provides data encryption and decryption functions to other *dojot* components. It is used only by internal services so they can protect data communication (both internally and externally) and data storage.

Available as a Docker image, Crypto Service can be instantiated easily and integrated in a short time. Encrypt and decrypt data functionalities are accessed through REST APIs.

.. contents:: Table of Contents
  :local:

REST APIs
---------

Encrypt and decrypt data APIs are described below.

**Decrypt**

POST /crypto/decrypt

Request

Headers

::

    Content-Type: application/json

Body

::

    {
      "data": "Clear or cipher data",
      "tagSize": 16,
      "key": "2034F6E32958647FDFF75D265B455EBF40C80E6D597092B3A802B3E5863F878C",
      "iv": "AD0ACC568C88C116D57B273D98FB92C0"
    }

Response  200

Headers

::

    Content-Type: application/json

Body

::

    {
      "data": "Cipher or clear data",
      "result": "SUCCESS"
    }


**Encrypt**

POST /crypto/encrypt

Request

Headers

::

    Content-Type: application/json

Body

::

    {
      "data": "Clear or cipher data",
      "tagSize": 16,
      "key": "2034F6E32958647FDFF75D265B455EBF40C80E6D597092B3A802B3E5863F878C",
      "iv": "AD0ACC568C88C116D57B273D98FB92C0"
    }

Response  200

Headers

::

    Content-Type: application/json

Body

::

    {
      "data": "Cipher or clear data",
      "result": "SUCCESS"
    }

Usage Examples
--------------

In order to use cryptographic functions provided by Crypto Service, one must access the available REST APIs through a HTTP request.

Examples of how those requests can be made are showed bellow using the command line tool curl.

**Encrypt**

::

    curl -X POST \
      http://localhost:8080/cryptointegration/rest/crypto/encrypt \
      -H 'content-type: application/json' \
      -d '{
      "data": "000102030405060708090A0B0C0D0F",
      "tagSize": 16,
      "key": "2034F6E32958647FDFF75D265B455EBF40C80E6D597092B3A802B3E5863F878C",
      "iv": "AD0ACC568C88C116D57B273D98FB92C0"
    }'

**Decrypt**

::

    curl -X POST \
      http://localhost:8080/cryptointegration/rest/crypto/decrypt \
      -H 'content-type: application/json' \
      -d '{
      "data": "C0FBC8DB5F72AD8DC04ECA2E32DA793F86D59D6",
      "tagSize": 16,
      "key": "2034F6E32958647FDFF75D265B455EBF40C80E6D597092B3A802B3E5863F878C",
      "iv": "AD0ACC568C88C116D57B273D98FB92C0"
    }'

