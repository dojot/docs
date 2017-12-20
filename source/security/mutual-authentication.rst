Mutual Authentication
=====================

Mutual authentication is the process in which two entities authenticate each other. In a client-server communication, the client must prove its identity to the server and the server must prove its identity to the client. Thus, each entity can ensure that they are communicating with a legitimate interlocutor.

Mutual authentication protects access to data the application accesses from *dojot* and therefore protects access to data of that application’s user. It is done by ensuring that only registered applications can access platform data and functionality. In addition, it ensures that the platform the application is accessing is legitimate, meaning that no attacker can pass themselves by the platform and get user or application data.

.. contents:: Table of Contents
  :local:

Using Mutual Authentication
---------------------------

Applications can access *dojot* functionality to interact with its components and connected devices. For an application to ensure that it is communicating with a legitimate platform (and vice versa), it must make use of the mutual authentication functionality *dojot* provides. This is a simple process and its use requires only three steps to follow:

* Application Registration. When an application is registered in *dojot*, it receives an identifier and a key that must be kept secret. This key is used to authenticate the application on the platform.

* Authentication. At the beginning of the communication between application and *dojot*, the application initiates a handshake in which the two entities will exchange information to ensure they are legitimate.

* Using the platform. When accessing *dojot* interfaces, the platform informs a session identifier that is obtained at the time of authentication. Thus, the platform can verify that the mutual authentication process was performed by the application.

Application Registration
------------------------

An application that is registered with *dojot* will receive an identifier and a key that must be kept secret. The registration indicates that an application will communicate and use platform features.

Currently, the method used to register an application is through a REST interface. After making the request for the registration, the application will receive a unique identifier and a key. The API is described below

**REGISTER COMPONENT** - Register new application

::

    POST /kerberos/registerComponent

    Response  200

    Headers
    Content-Type: application/json

    Body
    {
      "AppId": "0001020304050607",
      "AppKey": "000102030405060708090a0b0c0d0e0f000102030405060708090a0b0c0d0e0f"
    }

Received identifier and key will be used at the moment the application authenticates with *dojot*. In order to do this, a client library is provided to perform the authentication process (available in github.com/dojot/ma-client-libs) and therefore, the library should have knowledge about the values of the identifier and the key. The file https://github.com/dojot/ma-client-libs/kerberos/src/protocol/unique.h is used to store these values and will be used by the library at the moment of authentication.

Authentication
--------------

When communicating with *dojot*, the application must perform mutual authentication. This process is done through the library provided in github.com/dojot/ma-client-libs. By using the library, three steps should be followed:

1. Initialize the library with server addresses

2. Register the callback function

3. Call mutual authentication function

Library Initialization
~~~~~~~~~~~~~~~~~~~~~~

Initialization tells the library which URLs will be used to perform mutual authentication. The function to be used is described below:

**Initialize Kerberos**

.. code-block:: c

    errno_t initializeKerberos(uint8_t* host, uint8_t hostLength, uint8_t* uriRequestAS, uint8_t requestASLength, uint8_t* uriRequestAP, uint8_t requestAPLength)

The arguments used in the function are described below.

* host - Platform main URL

* hostLength - Host string size

* uriRequestAS - requestAS endpoint

* requestASLength - requestAS string size

* uriRequestAP - requestAP endpoint

* requestAPLength - requestAP string Size

The following code snippet shows an example of how the function can be used.

.. code-block:: c

    char* host = "http://localhost:8000/"; // dojot URL
    char* reqAS = "kerberos/requestAS";
    char* reqAP = "kerberos/requestAP";

    errno_t ret = initializeKerberos(host, strlen(host), reqAS, strlen(reqAS), reqAP, strlen(reqAP));

Register callback
~~~~~~~~~~~~~~~~~

While the mutual authentication process is carried out, the library communicates with the server and checks received data. If an error occurs during this process, the library will call a callback function.

This callback function is implemented by the library user and must be registered before the authentication process. The callback function can include code for error handling and logging, for example.

**Set Callback**

.. code-block:: c

    errno_t setCallback(void (*callback)(int))

The following code shows an example of how the callback function can be created and registered.

.. code-block:: c

    void errorCallback(int err){
        // Error handling and logging code
    }

    errno_t ret = setCallback(&errorCallback);

Call mutual authentication function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

After initializing the library with platform URL and registering the callback function, the library is ready to perform the mutual authentication process. The function that is used to perform the process is shown below.

.. code-block:: c

    errno_t executeKerberosHandshake()

The code below shows an example of how the function may be used.

.. code-block:: c

    errno_t ret = executeKerberosHandshake();

Accessing *dojot* APIs
----------------------

After the mutual authentication process completes, the application may send additional data in the calls to the platform interfaces. This data is the mutual authentication session identifier and is sent through an HTTP header.

The following is an example of a call to a *dojot* API where mutual authentication session identifier is also sent.

::

    GET /device HTTP/1.1
    Host: localhost:8000
    ma-session-id: a4cdad05441940c5c07ee9f55b8fafbdc0eba14afce449c9c9ec052bb20f50f4

