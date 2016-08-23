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

  * - ``GET /tenants/{tenantId}/{resourceName}``
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

Directory
============

.. contents::
    :local:
    :depth: 2

**Description**

Directory is a container for User and Usergroup resources. Every user is unique by email within Directory only. You can attach Directory to Application for storing end-users accounts. Your accounts for managing CloudThing are also stored in offcial Directory which cannot be deleted.

**Directory URL**

``/directories/{directoryId}``

**Directory Attributes**

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
    - Name of Directory.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``modifiedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``official``
    - Boolean
    - N/A
    - Indicates whether it;s the official Directory or not.

  * - ``description``
    - String
    - N/A
    - The description of Directory which may describes it's purpose.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Product.

  * - ``users``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Users <ref-user>` stored in this Directory.

  * - ``usergroups``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Usergroups <ref-usergroup>` stored in this Directory.

  * - ``applications``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Applications <ref-application>` mapped to this Directory.

**Directory Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo",
    "name": "Smart application directory",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "official": false,
    "description": "This directory is used for end-users of our Smart Home application",
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "users": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo/users"
    },
    "usergroups": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo/usergroups"
    },
    "applications": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo/applications"
    }
  }

Directory Operations
-----------------

Create A Directory
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /tenants/{tenantId}/directories``
    - Required: ``name``. Optional: ``description``, ``custom``.
    - Creates new directory.

Retrieve A Directory
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /directories/{directoryId}``
    - ``expand``
    - Retrieves the Directory with the specified ID. Expandable links: ``users``, ``usergroups``, ``applications``, ``tenant``.

Update A Directory
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /directories/{directoryId}``
    - ``name``, ``description``, ``custom``
    - Updates the Directory with the specified ID.

Delete A Directory
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /directories/{directoryId}``
    - N/A
    - Deletes the Directory with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo" \
  -H 'Accept: application/json'

Using A Directory for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Directory for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /directories/{directoryId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a Directory's associated resources of the specified type. Possible resource types are: ``users``, ``usergroups`` and ``applications``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a Directory**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo/users" \
  -H 'Accept: application/json'

This query would retrieve a collection containing all the Users associated with the specified Directory.

Application
============

.. contents::
    :local:
    :depth: 2

**Description**

Application is a resource representing real-world application or integration with it's own resources and limited access to tenants' data. You can attach a Directory of Users to Application and use it to limit scope of operations for them.

**Application URL**

``/applications/{applicationId}``

**Application Attributes**

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
    - Name of Application.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``modifiedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``official``
    - Boolean
    - N/A
    - Indicates whether it's the official Application or not. Visible only for Api Key or offical User.

  * - ``status``
    - String (enum)
    - ``ENABLED``, ``DISABLED``
    - Indicates whether Application is enabled or not. Visible only for Api Key or offical User.

  * - ``description``
    - String
    - N/A
    - The description of Application which may describes it's purpose.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Product.

  * - ``directory``
    - Link
    - N/A
    - A link to a :ref:`Directory <ref-directory>` attached to this Application if exists.

  * - ``devices``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Devices <ref-device>` available in this Application (if requester is Api Key or offical User) or assigned to current User (if requester is User).

  * - ``clusters``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Clusters <ref-cluster>` created in this Application (if requester is Api Key or official User) or owned by current User (if requester is User).

  * - ``exports``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Exports <ref-export>` created for this Application. Visible only for Api Key or offical User.

**Application Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA",
    "name": "Smart home application",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "official": false,
    "status": "ENABLED",
    "description": "This application is our Smart Home app for end-users",
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "directory": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo"
    },
    "devices": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/devices"
    },
    "clusters": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/clusters"
    },
    "exports": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/exports"
    }
  }

Application Operations
-----------------

Create An Application
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /tenants/{tenantId}/applications``
    - Required: ``name``. Optional: ``description``, ``custom``, ``status``, ``directory``.
    - Creates new application.

Retrieve An Application
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /applications/{applicationId}``
    - ``expand``
    - Retrieves the Application with the specified ID. Expandable links: ``devices``, ``clusters``, ``exports``, ``tenant``, ``directory``.

Update An Application
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /applications/{applicationId}``
    - ``name``, ``description``, ``custom``, ``status``, ``directory``
    - Updates the Application with the specified ID.

Delete An Application
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /applications/{applicationId}``
    - N/A
    - Deletes the Application with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA" \
  -H 'Accept: application/json'

Using An Application for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Application for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /applications/{applicationId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of an Application's associated resources of the specified type. Possible resource types are: ``devices``, ``clusters`` and ``exports``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with an Application**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/clusters" \
  -H 'Accept: application/json'

This query would retrieve a collection containing all the Clusters associated with the specified Application.

Associated subresources of Application
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve subresources of Application

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /applications/{applicationId}/availableResources``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a set of available in Application resources belonged to Application, Products, Clusters and Groups.

Example Queries
"""""""""""""""

**Retrieving an available resources in Application**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/resources" \
  -H 'Accept: application/json'


Product
============

.. contents::
    :local:
    :depth: 2

**Description**

Product is a model of your real-world product. You can create particular devices within it.

**Product URL**

``/products/{productId}``

**Product Attributes**

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
    - Name of Product.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``modifiedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``properties``
    - Array(Object)
    - N/A
    - List of objects, each containing: ``key``, ``name``, ``setOn`` and `unique`.

  * - ``resources``
    - Object
    - N/A
    - An embedded object containing information about resources, containd ``data``, ``events`` and ``commands`` arrays.

  * - ``extensions``
    - Object
    - N/A
    - An embedded object containing information available extensions and their configuration.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Product.

  * - ``devices``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Devices <ref-device>` mapped to this Product.

  * - ``functions``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Functions <ref-function>` mapped to this Product.

**Product Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpr0doocT",
    "name": "Smart washing machine",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "properties": [
      {
        "key": "macaddr",
        "name": "MAC address",
        "setOn": "MANUFACTURING",
        "unique": true
      }
    ],
    "resources": {
      "data": [
        {
          "id": "rpm",
          "name": "Revolutions per minute",
          "description": "Reports current RPM"
        }
      ],
      "events": [
        {
          "id": "dmg",
          "name": "Machine damage",
          "description": "Fired if machine damage occurred"
        }
      ],
      "commands": [
        {
          "id": "turn",
          "name": "Turn washing",
          "description": "Turns machine on/off",
          "payloads": [
            {
              "name": "on",
              "value": "ON"
            },
            {
              "name": "off",
              "value": "OFF"
            }
          ]
        }
      ]
    },
    "extensions": {
      "connectors": {
        "sigfox": {
          "autoGenerate": true,
          "contentType": "CUSTOM",
          "parser": {
            "href": "https://vanilla-ice.cloudthing.io/api/v1/functions/SgKSGoEETgSQ0dpNgdA5Qg"
          },
          "status": "ENABLED"
        }
      }
    },
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "devices": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpr0doocT/devices"
    },
    "functions": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpr0doocT/functions"
    }
  }

Product Operations
-----------------

Create A Product
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /tenants/{tenantId}/products``
    - Required: ``name``. Optional: ``properties``, ``resources``, ``custom``, ``extensions``.
    - Creates new product.

Retrieve A Product
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /products/{productId}``
    - ``expand``
    - Retrieves the Product with the specified ID. Expandable links: ``devices``, ``functions``, ``tenant``.

Update A Product
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /products/{productId}``
    - ``name``, ``properties``, ``resources``, ``custom``, ```extensions``
    - Updates the Product with the specified ID.

Delete A Product
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /products/{productId}``
    - N/A
    - Deletes the Product with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

	curl -u "user@example.com:password" \
	"https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpr0doocT" \
	-H 'Accept: application/json'

Using A Product for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Product for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /products/{productId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a Product's associated resources of the specified type. Possible resource types are: ``devices`` and ``functions``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a Product**

.. code-block:: bash

	curl -u "user@example.com:password" \
	"https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpr0doocT/devices" \
	-H 'Accept: application/json'

This query would retrieve a collection containing all the Devices associated with the specified Product.

Device
==================

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
