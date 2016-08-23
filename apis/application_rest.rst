.. CloudThing documentation master file, created by
   sphinx-quickstart on Sun May  8 19:31:11 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

 ******************
 HTTP REST API Reference
 ******************

REST API Core Concepts
==========================

The following information is essential to understanding how the CloudThing API works.

.. contents::
    :local:
    :depth: 2

Methods
------------

REST API uses JSON as serialization format and requires proper *Content-Type* and *Accept* headers in every client request.

API uses following HTTP verbs (methods):

- **GET** (collection) - retrieves collection,
- **GET** (item) - retrieves single item,
- **POST** (collection) - creates new item in collection,
- **POST** (item) - updates item,
- **DELETE** (item) - deletes item.

API **does not** support **PUT** since it is not possible to replace objects and **PATCH** which as we believe is commonly misunderstood and being used in wrong way (for updates).

Linking
---------
CloudThing API implements great linking concept developed by awesome people at stormpath.com.
Every response (collection or object) is identified by its URL in *"href"* field, there is no other *id* provided. All relations between objects use this method and embed *href* in JSON object named as relative. It is possible to expanding relations in single request (one level only) by providing relation name to *expand* parameter in query.

In short words:

Just hit /api/v1/tenants/current and start exploring!

Host
----------

HTTP API accept only encrypted (TLS) requests. All HTTP non-TLS requests will be redirected.
**Note that your credential still be exposed if you use plain HTTP!**

The HTTP server listens on standard port 443 of tenant's virtual host.

Authentication
---------------------

CloudThing API supports Basic and Bearer (JWT) authentication. Basic auth needs:

- Username: {apiKey} or {userEmail},
- Password: {apiSecret} or {userPassword}.

To obtain JWT token one should send POST request to */api/v1/auth/token* endpoint with Basic auth.

Tenant
============

.. contents::
    :local:
    :depth: 2

**Description**

Tenant is an organization on behalf of which user or API key requests data. When registering an account at CloudThing.io the organization and user within it are created.

**Tenant URL**

``/tenants/{tenantId}``

**Tenant Attributes**

.. list-table::
  :widths: 15 10 20 60
  :header-rows: 1

  * - Attribute
    - Type
    - Valid Value(s)
    - Description

  * - ``href``
    - Link
    - N/A
    - The resource's fully qualified location URL.

  * - ``name``
    - String
    - 1 < N < 256 characters
    - Full name of Tenant, eg. organization/company name.

  * - ``shortName``
    - String
    - 1 < N <= 63 characters
    - Human-readable unique key. This key is unique and assigned by CloudThing upo registration. If you would like to change it, please contact us.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``modifiedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``customData``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``limits``
    - Object
    - N/A
    - An embedded object containing information about available limits of tokens, SMSs and emails.

  * - ``directories``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Directories <ref-dierctory>` mapped to this Tenant.

  * - ``applications``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Applications <ref-application>` mapped to this Tenant.

  * - ``products``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Products <ref-product>` mapped to this Tenant.

  * - ``devices``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Devices <ref-device>` mapped to this Tenant.

  * - ``analytics``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Analytics <ref-analytics>` configured for this Tenant.

  * - ``users``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Users <ref-user>` mapped to this Tenant.

  * - ``statistics``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Statstics <ref-statistic>` available for this Tenant.


**Tenant Example**

.. code-block:: json

	{
	  "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT",
	  "name": "user@example.com's organization",
	  "shortName": "vanilla-ice",
	  "createdAt": "2016-05-15T11:18:33Z",
	  "updatedAt": "2016-05-15T11:18:33Z",
	  "limits": {
	  	"tokens": {
	  		"total": 5,
	  		"used": 1
	  	},
	  	"sms": {
	  		"total": 5,
	  		"used": 0
	  	},
	  	"emails": {
	  		"total": 5,
	  		"used": 0
	  	}
	  },
	  "directories": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/directories"
	  },
	  "applications": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/applications"
	  },
	  "products": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/products"
	  },
	  "devices": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/devices"
	  },
	  "analytics": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/analytics"
	  },
	  "users": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/users"
	  },
	  "custom": {

	  }
	}

Tenant Operations
-----------------

Retrieve A Tenant
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /tenants/current``
    - N/A
    - Retrieves the Tenant associated with the current API key. The response will be a ``302 Redirect``. You will find the location of the Tenant in a Location header and response body, although most REST libraries and web browsers will automatically issue a request for it.

  * - ``GET /tenants/{tenantId}``
    - N/A
    - Retrieves the Tenant with the specified ID.

Example Query
"""""""""""""

Retrieves link to current tenant:

.. code-block:: bash

	curl -u "user@example.com:password" \
	"https://vanilla-ice.cloudthing.io/api/v1/tenants/current" \
	-H 'Accept: application/json'

Response::

	HTTP/1.1 302 Found
	Content-Type: application/json
	Location: https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT

	{
		"tenant": {
			"href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
		}
	}

Retrieves tenant:

.. code-block:: bash

	curl -u "user@example.com:password" \
	"https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT" \
	-H 'Accept: application/json'

Using A Tenant for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Tenant for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - GET /tenants/{tenantId}/{resourceName}
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a Tenant's associated resources of the specified type. Possible resource types are: ``directories``, ``applications``, ``products``, ``devices``, ``analytics``, ``users``, and ``statistics``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a Tenant**

.. code-block:: bash
curl -u "user@example.com:password" \
"https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/products" \
-H 'Accept: application/json'

This query would retrieve a collection containing all the Products associated with the specified Tenant.

Device
=============

Device's data (resources) can be retrieved by hitting /api/v1/devices/{id}/resources{data,events,commands}/{key} (eg. */api/v1/devices/s0m31D/resources/data/temp*).

User
================

Password can be changed by making POST request to user endpoint. To obtain user ID, hit /api/v1/users/current. Example:

**GET**

Retrieves link to current user:

.. code-block:: bash

	curl -u "user@example.com:password" \
	https://vanilla-ice.cloudthing.io/api/v1/users/current

Response::

	HTTP/1.1 302 Found
	Content-Type: application/json
	Location: https://vanilla-ice.cloudthing.io/api/v1/users/Som31D0fuZ3R

	{
		"user": {
			"href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31D0fuZ3R"
		}
	}

**POST**

Change password:

.. code-block:: bash

	curl -u "user@example.com:password" \
	-H "Content-Type: application/json" \
	-d '{"password": "newpass"}' \
	https://vanilla-ice.cloudthing.io/api/v1/users/Som31D0fuZ3R
