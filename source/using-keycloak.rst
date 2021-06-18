Using Keycloak in dojot
=======================

As of version v0.8.0, dojot started using `keycloak`_ and
removed the use of the old *auth* service. Keycloak is an open source identity
and access management solution.

In this tutorial, we will explain some important points of setting
up and using keycloak with dojot.

.. note::
   - Audience: users and administrators
   - Level: basic to intermediate
   - Reading time: 20 m

.. contents:: Table of Contents
  :local:

.. attention::
   A dojot *tenant* is equivalent to a keycloak *realm*.

Required
--------

 - If you are starting a *deployment*, installing a dojot, see :ref:`Setting Passwords (Required) via Environment Variables`.

Recommendations
---------------

 - Always use HTTPs.
 - Use strong passwords, by default the minimum required are
   passwords with 8 characters, one number, one uppercase letter,
   one lowercase letter and one special character.
 - Configure SMTP


.. _Accessing the Administrative Panel:

Accessing the Administrative Panel
----------------------------------

**Replace <url> and <real> for your case**

Using the master realm
~~~~~~~~~~~~~~~~~~~~~~

.. attention::
   You can only access this page using **localhost** or **https**.

Using a user who belongs to the Realm master and can control all realms,
go to ``<url>/auth/``, eg ``https://dojot.com.br/auth``.


Using a user with role of *admin*
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using a user who belongs to a realm can only control this realm,
go to ``<url>/auth/admin/<realm>/console``,
eg ``https://dojot.com.br/auth/admin/admin/console``.

Checking selected realm
~~~~~~~~~~~~~~~~~~~~~~~

After accessing the keycloak administrative interface using one of the paths above,
make sure you have the realm you want to use selected, in the left sidebar you
can select the realm of interest, as highlighted in the image below:

   .. image:: images/tutorials/keycloak/keycloak1.png
      :scale: 40%
      :align: center
      :alt: Checking selected realm


.. _Accessing an Account Settings:

Accessing an Account Settings
-----------------------------

**Replace <url> and <real> for your case**

To access your account and make changes like enable 2-factor
authentication, change password, change email,
go to the ``<url>/auth/realms/<realm>/account``,
eg ``https://dojot.com.br/auth/realms/admin/account``.


Creating a new user
-------------------

Access the Administrative panel as explained in the topic :ref:`Accessing the Administrative Panel`.

In the left bar menu select the `Users` option and then click `Add User`.

Now let's create the user, first fill in **Username**. It is recommended to fill in
the **Email** field and in **Required User Actions** Add the ``Update Password`` and
``Verify Email`` options, but for ``Verify Email`` to make sense it is necessary
to have the SMTP configured for the Realm of interest, if you do not want to
configure SMTP do not put ``Verify Email``. Another point is that if an SMTP
has not been configured, it is necessary to create a temporary password
and provide the user, to do so, access the **Credentials** tab after creating in **Save**.

Furthermore, it is important to define the **Role** of this user in your Realm,
without this it cannot be used. After clicking ``Save`` one of the tabs will
be **Role Mapping**, click on it to define the role of the user.
In the **Role Mapping** tab in the **Available Roles** box there will
be the ``admin`` and ``user`` options (if no other *Roles* are created). A user with role
``admin`` has access to all APIs and also the Administrator Panel of your realm while a user with
Role ``user`` has access to the API but cannot access the Administrator Panel.
Select the Role that fits this new user and add it to the **Assigned Roles** Box.


Configuring SMTP through the graphical interface
------------------------------------------------

Access the Administrative panel as explained in the topic :ref:`Accessing the Administrative Panel`.

In the menu on the left bar select the option **Realm Settings** and
then select the tab **Email** and make the necessary settings.


Setting HTTPs as required by the graphical interface
----------------------------------------------------

.. note::
   You must be using a *deployment* with HTTPs configured.

Access the Administrative panel as explained in the
topic :ref:`Accessing the Administrative Panel`.

In the menu on the left bar select the **Realm Settings** option and
then select the **Login** tab, under **Require SSL** select the ``external requests``
option and click **Save**.


Creating a new realm (tenant in the dojot)
------------------------------------------

Access the administrative panel as explained in the topic :ref:`Accessing the Administrative Panel`,
but necessarily **using the master realm**.

On the left side menu, hover over the current Realm name and an **Add realm**
option will appear, click on it.

.. note::
   The realm created here will be based on the file passed as the base for the keycloak,
   see more in :ref:`Base keycloak configuration file`.

.. _Getting Access Token JWT:

Getting Access Token (Access Token JWT)
---------------------------------------

.. _Getting Access Token JWT from *username* and *password*:

From *username* and  *password* in a realm **(Not recommended)**
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

We have a *client* created in keycloak configured and disabled with the
name ``dev-test-cli`` which allows getting a JWT from a login and password in a realm.

.. attention::
   For security reasons it is disabled by default,
   after use it is recommended to disable it again.

.. _How to enable and disable *client* `dev-test-cli`:

How to enable and disable client ``dev-test-cli``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Access the Administrative panel as explained in the topic :ref:`Accessing the Administrative Panel`.

Select the option **Clients** in the left side menu,
it will load a new screen, in it there will be the list of clients.
Look for the ``dev-test-cli`` client, click on it,
it will open a new screen with several options,
one of them will be the **Enabled** option which
will have the value ``OFF`` change to the value ``ON``,
go to the end of the page and click ``Save``.

Remember to disable this client after use by setting the **Enabled**
value to ``OFF`` and saving, for security reasons.

.. attention::
   To use the command below to obtain the JWT you must already
   have access through the graphical interface the keycloak once and defined a password.
   If you haven't done this yet, please follow the previous topic :ref:`Accessing an Account Settings`
   before continuing.
   Also it is necessary to have `curl`_ and `jq`_. In Debian-based Linux
   distributions you can run: ``sudo apt-get install curl jq``

After enabling the ``dev-test-cli`` client it will be possible
to obtain a token using login and password.
As in the curl command below using the keycloak API,
change the values of ``DOJOT_URL``, ``REALM``,
``USERNAME`` and ``PASSWORD`` according to your case:

.. code-block:: bash

    DOJOT_URL=http://localhost:8000
    REALM=admin
    USERNAME=admin
    PASSWORD=password

    JWT=$(curl --location --request POST ${DOJOT_URL}/auth/realms/${REALM}/protocol/openid-connect/token \
    --data-urlencode "username=${USERNAME}" \
    --data-urlencode "password=${PASSWORD}" \
    --data-urlencode "client_id=dev-test-cli" \
    --data-urlencode "grant_type=password" 2>/dev/null | jq -r ".access_token")

If everything works out, the token will be available in the JWT variable,
to get the variable's value use the command below:

.. code-block:: bash

    echo $JWT

.. attention::
   The lifetime of this access token is not very long,
   this process returns a *refresh token* which can be used to get a new access token.

For a backend application
~~~~~~~~~~~~~~~~~~~~~~~~~

To use the Access Token in a backend application
it is recommended to create a new `Client` for the realm of interest or
the realms of interest, remember to configure them in **Client Protocol**
with the value ``openid-connect``, **Access Type** with
the value ``Confidential`` and using the `Secret`
obtained from the **Credentials** tab, there are
several possible configurations.
It is important to follow the currently established security standards for OAuth 2
and OpenIDConnect.
There are several keycloak libraries in various languages
that can help with this development.

For a frontend application
~~~~~~~~~~~~~~~~~~~~~~~~~~

We have a microservice in order to help the development to ensure more security
for using keycloak with dojot, see more about `Backstage`_.
You can use keycloak directly with `PKCE`_ and OpenID Connect as well.
It is important to follow  the currently established security standards for
OAuth 2 and OpenIDConnect.

For a mobile application
~~~~~~~~~~~~~~~~~~~~~~~~

We don't have use cases, but it's important to follow the currently
established security standards for OAuth 2 and OpenIDConnect.
Some starting points to pay attention to would be to create a
new Client with the Access Type `public` and use `PKCE`_.

Deployment Settings
-------------------

.. _Setting Passwords (Required) via Environment Variables:

Setting Passwords (Required) via Environment Variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The dojot deployments (**Docker compose and Ansible - kubernetes**) do not come with
passwords set for the ``admin`` and ``master`` users and it is necessary to set these
passwords so that the *deployments* will start up correctly,
if not configured services will not start and dojot will be unavailable.
But beware these values are only applied when the keycloak starts
for the first time and the *realm* are created, it will not affect the existing realms.

.. attention::
   When configuring use strong passwords, by default the minimum required are passwords
   with 8 characters, one number, one uppercase letter,
   one lowercase letter and one special character.


Docker compose
^^^^^^^^^^^^^^

You need to set a password value in the *.env* file for the
``KEYCLOAK_MASTER_PASSWORD`` and ``KEYCLOAK_ADMIN_PASSWORD_TEMP``
variables. The ``KEYCLOAK_ADMIN_PASSWORD_TEMP`` value will be the *admin*
user password of all realms when created.
See more at `settings required`_.


Ansible (kubernetes)
^^^^^^^^^^^^^^^^^^^^

*Work in progress*

Setting SMTP via environment variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is possible to configure SMTP via environment variables,
but beware these values are only applied when the *realm*
is created, it will not affect the existing realms.
To configure a *realm* that already exists,
see `Configuring SMTP through the graphical interface`.


Docker compose
^^^^^^^^^^^^^^

See more at `configuring SMTP`_.

Ansible (kubernetes)
^^^^^^^^^^^^^^^^^^^^

*Work in progress*

Setting HTTPs as required via environment variables
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Note: These values are only applied when the *realm* is created,
it will not affect existing realms.
For existing reals see `Configuring HTTPs as required`

Docker compose
^^^^^^^^^^^^^^

You need to set the **EXTERNAL** value in the *.env*
file to the ``KEYCLOAK_REALM_SSL_MODE`` variable. See more at `configuring HTTPs`_.

Ansible (kubernetes)
^^^^^^^^^^^^^^^^^^^^

*Work in progress*

.. _Base keycloak configuration file:

Base keycloak configuration file for creating realms
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Configuration file used as the basis for creating realms in keycloak.
It is the file exported from a realm by the full keycloak.
This file is configured using the ``DOJOT_CUSTOM_REALM_REP_FILE``
environment variable in the keycloak service.
When creating a new realm the identifiers (*ids*) and
realm name in this file are ignored. However, it is important
to point out that our *deployments* for both *docker-compose*
and *ansible (kubernetes)* already have a file with the necessary settings to use dojot.

Docker compose
^^^^^^^^^^^^^^

In the case of docker-compose this file is in `keycloak/customRealmRepresentation.json`

Ansible (kubernetes)
^^^^^^^^^^^^^^^^^^^^

*Work in progress*

Old libraries
~~~~~~~~~~~~~

To maintain compatibility with dojot it is necessary to provide the login
and password of the master or a user who can get the list of realms
available in the older libraries, namely `dojot-module-nodejs`_,
`iotagent-nodejs`_, `dojot-module-python`_,  `dojot-module-java`_,
`iotagent-java`_, see more in their respective documentation.
However, it is important to point out that our *deployments* for
both *docker-compose* and *ansible (kubernetes)* are already prepared
for this and it is only necessary to configure the passwords as described
in the topic `Setting Passwords (Required) via environment variables`.

Configuring new routes, services, resources,on keycloak to use authorization
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In `keycloak settings example`_ there is a basic case of configuration example, you can use it as basis for new settings.

Configuring keycloak with Nginx
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In `how to use keycloak with a nginx proxy`_
there is an example with the necessary settings.


.. _keycloak: https://www.keycloak.org/
.. _curl: https://curl.haxx.se/
.. _jq: https://stedolan.github.io/jq/
.. _Backstage: https://github.com/dojot/backstage/releases/tag/v0.8.0
.. _PKCE: https://oauth.net/2/pkce
.. _settings required: https://github.com/dojot/docker-compose/tree/v0.8.0#required-settings-and-recommended-settings
.. _configuring SMTP: https://github.com/dojot/docker-compose/tree/v0.8.0#keycloak-smtp
.. _configuring HTTPs: https://github.com/dojot/docker-compose/tree/v0.8.0#how-to-run-with-https-secure-dojot-with-lets-encrypt---recommended
.. _keycloak settings example: https://github.com/dojot/dojot/tree/v0.8.0/api-gateway/kong/examples#keycloak-settings
.. _how to use keycloak with a nginx proxy: https://github.com/dojot/dojot/tree/v0.8.0/iam/keycloak#how-to-use-keycloak-with-a-nginx-proxy
.. _Keycloak Dojot Provider: https://github.com/dojot/dojot/tree/v0.8.0iam/keycloak/dojot-provider#readme

.. _dojot-module-nodejs: https://github.com/dojot/dojot-module-nodejs
.. _iotagent-nodejs: https://github.com/dojot/iotagent-nodejs
.. _dojot-module-python: https://github.com/dojot/dojot-module-python
.. _dojot-module-java: https://github.com/dojot/dojot-module-java
.. _iotagent-java: https://github.com/dojot/iotagent-java