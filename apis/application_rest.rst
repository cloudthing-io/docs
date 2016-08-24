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

Base URL
---------------
This API is accesible via ``/api/v1``.
The full path for example tenant ``vanilla-ice`` would be:
``https://vanilla-ice.cloudthing.io/api/v1``

Methods
------------

API uses following HTTP verbs (methods):

.. list-table::
  :widths: 30 30 40
  :header-rows: 1

  * - Method
    - Target
    - Action

  * - ``GET``
    - collection
    - Retrieves collection.

  * - ``GET``
    - item
    - Retrieves single item.

  * - ``POST``
    - collection
    - Creates new item in collection.

  * - ``POST``
    - item
    - Updates item.

  * - ``DELETE``
    - item
    - Deletes item.

.. note::

  API **does not** support **PUT** since it is not possible to replace objects and **PATCH** which as we believe is commonly misunderstood and being used in wrong way (for updates).

Serialization format
----------------------

Currently REST API uses only JSON as serialization format and requires proper ``Content-Type`` (``POST``) and ``Accept`` (``POST``, ``GET``) headers in every client request.


TLS
----------

HTTP API accepts only encrypted (TLS) requests. All HTTP non-TLS requests will be redirected.
The HTTP server listens on standard port 443 of tenant's virtual host.

.. warning::

  Note that due to redirection of HTTP request port 80 is open, so your credentials will still be exposed if you send request without TLS encryption.

Authentication & authorization
---------------------

CloudThing REST API supports Basic and Bearer (JWT) authentication. Since API can be used with not only API keys but also users' (official and end) credentials, one should provide following authorization data.

Basic auth
^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 25 25 25 25
  :header-rows: 1

  * - Requester
    - Username
    - Password
    - Header & query

  * - API key
    - ``key``
    - ``secret``
    - Header: ``Authorization: Basic base64({key}:{secret})``

  * - Official user
    - ``email``
    - ``password``
    - Header: ``Authorization: Basic base64({email}:{password})``

  * - User
    - ``email``
    - ``password``
    - Header: ``Authorization: Basic base64({email}:{password})``, Query: ``application=applicationId``


JWT token auth
^^^^^^^^^^^^^^^^^

To obtain JWT token one should send POST request to ``/api/v1/auth/token`` endpoint with Basic auth.

.. list-table::
  :widths: 30 30 40
  :header-rows: 1

  * - Requester
    - Token
    - Header & query

  * - API key/Official user/User
    - ``access_token``
    - Header: ``Authorization: Bearer {access_token}``


Collections
---------

When requesting collection you can add additional query parameters:

.. list-table::
  :widths: 15 10 20 60
  :header-rows: 1

  * - Attribute
    - Type
    - Default Value
    - Description

  * - ``limit``
    - Integer
    - 25
    - Requested maximum number of returned items in single response.

  * - ``page``
    - Integer
    - 1
    - Requested page of returned items.

API will return meta data in top level of object and items in ``items`` array.
Metadata is:

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

  * - ``size``
    - Integer
    - N/A
    - Number of all items in collection.

  * - ``limit``
    - Integer
    - N/A
    - Requested maximum number of returned items in single response.

  * - ``page``
    - Integer
    - N/A
    - Requested page of returned items in cuurent response.

  * - ``prev``
    - Link
    - N/A
    - The URL to previus page of collection.

  * - ``next``
    - Link
    - N/A
    - The URL to next page of collection.

Example Query
"""""""""""""

Retrieving collection:

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0ft3nAnT/applications?limit=2&page=1" \
  -H 'Accept: application/json'

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0ft3nAnT/applications?limit=2&page=1",
    "size": 3,
    "limit": 2,
    "page": 1,
    "items": [
      {
        "href: "https://vanilla-ice.cloudthing.io/api/v1/applications/aPp1iCaT10n01"
        // JSON continues here, trunked for readability
      },
      {
        "href: "https://vanilla-ice.cloudthing.io/api/v1/applications/ArFd87gf10n02"
        // JSON continues here, trunked for readability
      }
    ],
    "next": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0ft3nAnT/applications?limit=2&page=2"
    }
  }


Linking & expanding
---------

CloudThing API implements great linking concept developed by awesome people at ``stormpath.com``.
Every response (collection or item) is identified by its URL in ``href`` field, there is no other *id* provided. All relations between objects use this method and embed ``href`` in JSON object named as relative. It is possible to expand relations in single request (one level only) by providing relation name to ``expand`` parameter in query. You can expand several relations by putting them comma-seprated and even paginate and limit results if relation is collection by adding ``(limit:{limit},offset:{offset})`` after relation name.

In short words:

Just hit ``/api/v1/tenants/current`` and start exploring!

Example Query
"""""""""""""

Link in resource body:

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA" \
  -H 'Accept: application/json'

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA",
    // JSON continues here, trunked for readability
    "tenant": {
      "href: "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0ft3nAnT"
    }
  }

Expanded link in resource body:

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA?expand=tenant" \
  -H 'Accept: application/json'

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA",
    // JSON continues here, trunked for readability
    "tenant": {
      "href: "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0ft3nAnT"
      "shortName": "vanilla-ice",
      // JSON continues here, trunked for readability
    }
  }

Expanded link to collection:

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA?expand=clusters(limit:2,page:1)" \
  -H 'Accept: application/json'

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA",
    // JSON continues here, trunked for readability
    "clusters": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/clusters",
      "size": 3,
      "limit": 2,
      "page": 1,
      "items": [
        {
          "href: "https://vanilla-ice.cloudthing.io/api/v1/clusters/cLusCaT10n01"
          // JSON continues here, trunked for readability
        },
        {
          "href: "https://vanilla-ice.cloudthing.io/api/v1/clusters/Ar76fya10n02"
          // JSON continues here, trunked for readability
        }
      ],
      "next": {
        "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/Som31D0faPpl1cA/clusters?limit=2&page=2"
      }
    }
  }

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

  * - ``updatedAt``
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

  * - ``updatedAt``
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

Usergroup
============

.. contents::
    :local:
    :depth: 2

**Description**

Usergroup is a container for User resources. Every user can belong to many Groups.

**Directory URL**

``/usergroups/{usergroupId}``

**Usergroup Attributes**

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
    - Name of Usergroup.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to the :ref:`Tenant <ref-tenant>` owning this Usergroup.

  * - ``directory``
    - Link
    - N/A
    - A link to the :ref:`Directory <ref-directory>` this Usergroup is stored in.

  * - ``users``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Users <ref-user>` assigned to this Usergroup.

  * - ``memberships``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Memberships <ref-membership>` of this Usergroup.

**Usergroup Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/usergroups/Som31D0fOoZ3rGru",
    "name": "Home owners",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "directory": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo"
    },
    "users": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/usergroups/Som31D0fOoZ3rGru/users"
    },
    "memberships": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/usergroups/Som31D0fOoZ3rGru/memberships"
    }
  }

Usergroup Operations
-----------------

Create A Usergroup
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /directories/{directoryId}/usergroups``
    - Required: ``name``. Optional: ``custom``.
    - Creates new usergroup.

Retrieve A Usergroup
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /usergroups/{usergroupId}``
    - ``expand``
    - Retrieves the Usergroup with the specified ID. Expandable links: ``users``, ``memberships``, ``directory``, ``tenant``.

Update A Usergroup
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /usergroups/{usergroupId}``
    - ``name``, ``custom``
    - Updates the Usergroup with the specified ID.

Delete A Usergroup
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /usergroups/{usergroupId}``
    - N/A
    - Deletes the Usergroup with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/usergroups/Som31D0fOoZ3rGru" \
  -H 'Accept: application/json'

Using A Usergroup for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Usergroup for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /usergroups/{usergroupId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a Usergroup's associated resources of the specified type. Possible resource types are: ``users`` and ``memberships``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a Usergroup**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/usergroups/Som31D0fOoZ3rGru/users" \
  -H 'Accept: application/json'

This query would retrieve a collection containing all the Users associated with the specified Usergroup.

User
============

.. contents::
    :local:
    :depth: 2

**Description**

User represents a real person's account - either managing CloudThing's Tenant ot using end solutions.

**User URL**

``/users/{userId}``

**User Attributes**

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

  * - ``email``
    - String
    - 1 < N < 256 characters
    - User's email address.

  * - ``firstName``
    - String
    - 1 < N < 256 characters
    - User's first name.

  * - ``surname``
    - String
    - 1 < N < 256 characters
    - User's surname.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``activated``
    - Boolean
    - N/A
    - Indicates wheter user activated the account (eg. by email verification).

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to the :ref:`Tenant <ref-tenant>` owning this User.

  * - ``directory``
    - Link
    - N/A
    - A link to the :ref:`Directory <ref-directory>` this User is stored in.

  * - ``applications``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Applications <ref-application>` this User has access to.

  * - ``usergroups``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Usergroups <ref-usergroup>` this User belongs to.

  * - ``memberships``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Memberships <ref-membership>` of this User.

  * - ``devices``
    - Link
    - N/A
    - A link to a Collection of all the :ref:`Devices <ref-device>` this User has rights to.

**User Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f",
    "email": "john.doe@cloudthing.io",
    "firstName": "John",
    "surname": "Doe",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "activated": true,
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "directory": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/directories/Som31D0fD1r3cTo"
    },
    "applications": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f/applications"
    },
    "usergroups": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f/usergroups"
    },
    "memberships": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f/memberships"
    },
    "devices": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f/devices"
    }
  }

User Operations
-----------------

Create A User
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /directories/{directoryId}/users``
    - Required: ``email``, ``password``. Optional: ``firstName``, ``surname``, ``custom``.
    - Creates new user.

Retrieve A User
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /users/current``
    - N/A
    - Retrieves the User associated with the current authorization data. The response will be a ``302 Redirect``. You will find the location of the User in a Location header and response body, although most REST libraries and web browsers will automatically issue a request for it.

  * - ``GET /users/{userId}``
    - ``expand``
    - Retrieves the Usergroup with the specified ID. Expandable links: ``applications``, ``usergroups``, ``memberships``, ``devices``, ``directory``, ``tenant``.

Update A User
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /users/{userId}``
    - ``email``, ``password``, ``firstName``, ``surname``, ``custom``
    - Updates the User with the specified ID.

Delete A User
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /users/{userId}``
    - N/A
    - Deletes the User with the specified ID.

Example Query
"""""""""""""

Retrieves link to current User:

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/users/current" \
  -H 'Accept: application/json'

Response::

  HTTP/1.1 302 Found
  Content-Type: application/json
  Location: https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f

  {
    "user": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f"
    }
  }

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f" \
  -H 'Accept: application/json'

Using A User for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the User for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /users/{userId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a User's associated resources of the specified type. Possible resource types are: ``applications``, ``devices``, ``usergroups`` and ``memberships``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a User**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f/applications" \
  -H 'Accept: application/json'

This query would retrieve a collection containing all the Applications associated with the specified User.

Membership
============

.. contents::
    :local:
    :depth: 2

**Description**

Membership represents assignment of User to Usergroup.

**Membership URL**

``/memberships/{membershipId}``

**Membership Attributes**

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

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``user``
    - Link
    - N/A
    - A link to the :ref:`User <ref-user>` this Membership is about.

  * - ``usergroup``
    - Link
    - N/A
    - A link to the :ref:`Usergroup <ref-usergroup>` this Membership is about.

**Membership Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/memberships/Som31D0fMeM83rSh1P",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "user": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/users/Som31DUuZ3R0f"
    },
    "usergroup": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/usergroups/Som31D0fOoZ3rGru"
    }
  }

Membership Operations
-----------------

Create A Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /usergroups/{usergroupId}/memberships``
    - Required: ``user``.
    - Assigns given user to usergroup.

  * - ``POST /users/{userId}/memberships``
    - Required: ``usergroup``.
    - Assigns user to given usergroup.

Retrieve A Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /memberships/{membershipId}``
    - ``expand``
    - Retrieves the Membership with the specified ID. Expandable links: ``user``, ``usergroup``.

Delete A Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /memberships/{membershipId}``
    - N/A
    - Deletes the Membership with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/memberships/Som31D0fMeM83rSh1P" \
  -H 'Accept: application/json'

API Key
============

.. contents::
    :local:
    :depth: 2

**Description**

API keys are used for authorization during CloudThing API operations.

**API Key URL**

``/apikeys/{apikeyId}``

**API Key Attributes**

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
    - Name of API key.

  * - ``key``
    - String
    - 25 characters
    - API key.

  * - ``secret``
    - String
    - 32 characters
    - API secret. May be obtained only once in response for API key create request.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``status``
    - String (enum)
    - ``ENABLED``, ``DISABLED``
    - Presents status of API key.

  * - ``description``
    - String
    - N/A
    - The description of API key which may describes it's purpose.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this API key.

**API key Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/apikeys/AP1k3y1D3XamP13",
    "name": "CRM key",
    "key": "cJyGHVM1yIKoGQZowZXQz934e",
    "secret": "sIe7QxDach1l4xNzmrKNTH5TVn9eV9PJ",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "status": "ENABLED",
    "description": "This API key is used in our custom CRM integration",
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    }
  }

.. warning::

  The API key secret will be returned only once as a part of response for API key create request. You will not be able to retrieve that secret again. You can also create API key and download secret through CloudThing Control Center web application or CloudThing CLI.

API key Operations
-----------------

Create An API key
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /tenants/{tenantId}/apikeys``
    - Optional: ``name``, ``description``, ``custom``, ``status``.
    - Creates new API key.

Retrieve An API key
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /apikeys/{apikeyId}``
    - ``expand``
    - Retrieves the API key with the specified ID. Expandable links: ``tenant``.

Update An API key
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /apikeys/{apikeyId}``
    - ``name``, ``description``, ``custom``, ``status``
    - Updates the API key with the specified ID.

Delete An API key
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /apikeys/{apikeyId}``
    - N/A
    - Deletes the API key with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/apikeys/AP1k3y1D3XamP13" \
  -H 'Accept: application/json'

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

  * - ``updatedAt``
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

  * - ``updatedAt``
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
============

.. contents::
    :local:
    :depth: 2

**Description**

Device is a single real-world node or other data source implementing Product model.

**Device URL**

``/devices/{deviceId}``

**Device Attributes**

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

  * - ``token``
    - String
    - 32 characters
    - Device's token. 

  * - ``activated``
    - Boolean
    - N/A
    - Indicates whether device has been activated (by connecting with CloudThing platform).

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``properties``
    - Object
    - N/A
    - Object with properties where key is a defined in Product property id.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Product.

  * - ``product``
    - Link
    - N/A
    - A link to a :ref:`Product <ref-product>` this Device implements.

**Device Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/devices/Som31D0fd3V1c3",
    "token": "STTPDGtpSNkGpHdasG1BznI0u7w9pK4p",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "activated": true,
    "custom": {

    },
    "properties": {
      "macaddr": "00:11:22:33:44:55"
    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "product": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpr0doocT"
    }
  }

Device Operations
-----------------

Create A Device
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /products/{productId}/devices``
    - Optional: ``properties``, ``custom``.
    - Creates new device.

Retrieve A Device
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /devices/{deviceId}``
    - ``expand``
    - Retrieves the Device with the specified ID. Expandable links: ``tenant``, ``product``.

Update A Device
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /devices/{deviceId}``
    - ``properties``, ``custom``
    - Updates the Device with the specified ID.

Delete A Device
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /devices/{deviceId}``
    - N/A
    - Deletes the Device with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/devices/Som31D0fd3V1c3" \
  -H 'Accept: application/json'

Associated endpoints
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to use associated endpoint for retrieving additional data.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /devices/{deviceId}/resources/{type}/{name}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a associated resource of the specified type. Possible resource types are: ``data``, ``events`` and ``commands``.

Example Queries
"""""""""""""""

**Retrieving a temperature measured by Device**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/devices/Som31D0fd3V1c3/resources/data/temp" \
  -H 'Accept: application/json'

This query would retrieve a collection containing lat measurments of ``temp`` resource by Device.

Cluster
============

.. contents::
    :local:
    :depth: 2

**Description**

Cluster is a container for devices which exists in separate context for every Application. Each Device can belong to only one Cluster per Application (eg. Cluster would represent a home or apartment in Smart Home project).

**Cluster URL**

``/clusters/{clusterId}``

**Cluster Attributes**

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
    - Cluster's name. 

  * - ``description``
    - String
    - N/A
    - Description of Cluster.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Product.

  * - ``application``
    - Link
    - N/A
    - A link to a :ref:`Application <ref-application>` this Cluster exists within.

  * - ``users``
    - Link
    - N/A
    - A link to a Collection of the :ref:`Users <ref-application>` who has rights to this Cluster.

  * - ``devices``
    - Link
    - N/A
    - A link to a Collection of the :ref:`Devices <ref-devices>` which belongs to this Cluster.

  * - ``groups``
    - Link
    - N/A
    - A link to a Collection of the :ref:`Groups <ref-group>` existing within this Cluster.

  * - ``memberships``
    - Link
    - N/A
    - A link to a Collection of the :ref:`Group Memberships <ref-groupMemberships>` associated with this Cluster.

**Cluster Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX",
    "name": "NYC apartment",
    "description": "The Does family's New York City apartment",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "application": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/AppL1CaT10n1D"
    },
    "users": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX/users"
    },
    "devices": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX/devices"
    },
    "groups": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX/groups"
    },
    "memberships": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX/memberships"
    }
  }

Cluster Operations
-----------------

Create A Cluster
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /applications/{applicationId}/clusters``
    - Optional: ``name``, ``description``, ``custom``.
    - Creates new Cluster.

Retrieve A Cluster
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /clusters/{clusterId}``
    - ``expand``
    - Retrieves the Cluster with the specified ID. Expandable links: ``tenant``, ``application``, ``users``, ``devices``, ``groups``, ``memberships``.

Update A Cluster
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /clusters/{clusterId}``
    -  ``name``, ``description``, ``custom``
    - Updates the Cluster with the specified ID.

Delete A Cluster
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /clusters/{clusterId}``
    - N/A
    - Deletes the Cluster with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX" \
  -H 'Accept: application/json'

Using A Cluster for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Cluster for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /clusters/{clusterId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a Cluster's associated resources of the specified type. Possible resource types are: ``devices``, ``users`, ``groups`` and ``memberships``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a Cluster**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX/devices" \
  -H 'Accept: application/json'

This query would retrieve a collection containing all the Devices associated with the specified Cluster.


Cluster Membership
============

.. contents::
    :local:
    :depth: 2

**Description**

Cluster Membership represents assignment of Device to Cluster.

**Cluster Membership URL**

``/clusterMemberships/{membershipId}``

**Cluster Membership Attributes**

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

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``device``
    - Link
    - N/A
    - A link to the :ref:`Device <ref-device>` this Cluster Membership is about.

  * - ``cluster``
    - Link
    - N/A
    - A link to the :ref:`Cluster <ref-cluster>` this Cluster Membership is about.

**Cluster Membership Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/clusterMemberships/Som31D0fMeM83rSh1P",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "device": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/devices/d3v1C31dExa"
    },
    "cluster": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/cLUzT3r1DS0m3"
    }
  }

Cluster Membership Operations
-----------------

Create A Cluster Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /clusters/{clusterId}/memberships``
    - Required: ``device``.
    - Assigns given Device to Cluster.

  * - ``POST /devices/{deviceId}/clusterMemberships``
    - Required: ``cluster``.
    - Assigns Device to given Cluster.

Retrieve A Cluster Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /clusterMemberships/{membershipId}``
    - ``expand``
    - Retrieves the Cluster Membership with the specified ID. Expandable links: ``device``, ``cluster``.

Delete A Cluster Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /clusterMemberships/{membershipId}``
    - N/A
    - Deletes the Cluster Membership with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/clusterMemberships/Som31D0fMeM83rSh1P" \
  -H 'Accept: application/json'

Group
============

.. contents::
    :local:
    :depth: 2

**Description**

Group is a container for devices of the same Cluster. Each Device can belong to several Groups within one Cluster (eg. Group would represent a room in Smart Home project).

**Group URL**

``/groups/{groupId}``

**Group Attributes**

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
    - Group's name. 

  * - ``description``
    - String
    - N/A
    - Description of Group.

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``custom``
    - Object
    - N/A
    - A custom structure you can store your own custom fields in.

  * - ``tenant``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Group.

  * - ``application``
    - Link
    - N/A
    - A link to a :ref:`Application <ref-application>` this Group exists within.

  * - ``cluster``
    - Link
    - N/A
    - A link to a :ref:`Cluster <ref-cluster>` this Group exists within.

  * - ``devices``
    - Link
    - N/A
    - A link to a Collection of the :ref:`Devices <ref-devices>` which belongs to this Group.

**Group Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/groups/gRoUp31xDeX",
    "name": "Living room",
    "description": "Living room in New York City apartment",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "custom": {

    },
    "tenant": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "application": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/AppL1CaT10n1D"
    },
    "cluster": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/c7UZt3Rs1DeX"
    },
    "devices": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/groups/gRoUp31xDeX/devices"
    }
  }

Group Operations
-----------------

Create A Group
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /clusters/{clusterId}/groups``
    - Optional: ``name``, ``description``, ``custom``.
    - Creates new Group.

Retrieve A Group
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /groups/{groupId}``
    - ``expand``
    - Retrieves the Group with the specified ID. Expandable links: ``tenant``, ``application``, ``cluster``, ``devices``.

Update A Group
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /groups/{groupId}``
    -  ``name``, ``description``, ``custom``
    - Updates the Group with the specified ID.

Delete A Group
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /groups/{groupId}``
    - N/A
    - Deletes the Group with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/groups/gRoUp31xDeX" \
  -H 'Accept: application/json'

Using A Group for Look-Up
^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to retrieve other independent resources using the Group for look-up.

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /groups/{groupId}/{resourceName}``
    - :ref:`Pagination <about-pagination>`, :ref:`Sorting <about-sorting>`
    - Retrieves a collection of all of a Group's associated resources of the specified type. Possible resource types are: ``devices``.

Example Queries
"""""""""""""""

**Retrieving a Collection Associated with a Group**

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/groups/gRoUp31xDeX/devices" \
  -H 'Accept: application/json'

This query would retrieve a collection containing all the Devices associated with the specified Group.

Group Membership
============

.. contents::
    :local:
    :depth: 2

**Description**

Group Membership represents assignment of Device to Group.

**Group Membership URL**

``/groupMemberships/{membershipId}``

**Group Membership Attributes**

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

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``device``
    - Link
    - N/A
    - A link to the :ref:`Device <ref-device>` this Group Membership is about.

  * - ``group``
    - Link
    - N/A
    - A link to the :ref:`Group <ref-group>` this Group Membership is about.

**Group Membership Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/groupMemberships/Som31D0fMeM83rSh1P",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "device": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/devices/d3v1C31dExa"
    },
    "group": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/groups/gRooP1DS0m3"
    }
  }

Group Membership Operations
-----------------

Create A Group Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /groups/{groupId}/memberships``
    - Required: ``device``.
    - Assigns given Device to Group.

  * - ``POST /devices/{deviceId}/groupMemberships``
    - Required: ``group``.
    - Assigns Device to given Group.

Retrieve A Group Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /groupMemberships/{membershipId}``
    - ``expand``
    - Retrieves the Group Membership with the specified ID. Expandable links: ``device``, ``group``.

Delete A Group Membership
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /groupMemberships/{membershipId}``
    - N/A
    - Deletes the Group Membership with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/groupMemberships/Som31D0fMeM83rSh1P" \
  -H 'Accept: application/json'

Export
============

.. contents::
    :local:
    :depth: 2

**Description**

Export represents access level for different resources granted to Application .

**Export URL**

``/exports/{exportId}``

**Export Attributes**

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

  * - ``createdAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource was created.

  * - ``updatedAt``
    - String
    - RFC3339 Datetime
    - Indicates when this resource’s attributes were last modified.

  * - ``modelType``
    - String (enum)
    - ``DEVICE``, ``GROUP``, ``CLUSTER``
    - Type of exported resource owner.

  * - ``product``
    - Link
    - N/A
    - A link to a :ref:`Product <ref-tenant>` if ``modelType`` is ``DEVICE``.

  * - ``limitsType``
    - String (enum)
    - ``DEVICE``, ``GROUP``, ``CLUSTER``
    - Type of container limiting exporting resources.

  * - ``limits``
    - Link
    - N/A
    - A link to a :ref:`Device <ref-device>` if ``limitsType`` is ``DEVICE``, a :ref:`Group <ref-group>` if ``limitsType`` is ``GROUP`` or a :ref:`Cluster <ref-cluster>` if ``limitsType`` is ``CLUSTER``.

  * - ``export``
    - Array (Object)
    - N/A
    - An array of objects defining exported resources.

  * - ``tenantExportingPermission``
    - String (enum)
    - ``PRIMARY``, ``EXPORT``
    - Indicates wheter exporting Tenant has a primary permission for this resource or imported it.

  * - ``tenantExporting``
    - Link
    - N/A
    - A link to a :ref:`Tenant <ref-tenant>` owning this Product.

  * - ``application``
    - Link
    - N/A
    - A link to a :ref:`Application <ref-application>` importing this Export.


**Export Example**

.. code-block:: json

  {
    "href": "https://vanilla-ice.cloudthing.io/api/v1/exports/3xP0rT1D1234",
    "modelType": "DEVICE",
    "createdAt": "2016-05-15T11:18:33Z",
    "updatedAt": "2016-05-15T11:18:33Z",
    "product": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/products/Som31D0fpR0do0cT"
    },
    "limitsType": "CLUSTER",
    "limits": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/clusters/cLo0zt3rz1D"
    },
    "export": [
      {
        "type": "DATA",
        "name": "temp",
        "read": true,
        "write": true,
        "grantRead": false,
        "grantWrite": false
      },
      {
        "type": "COMMAND",
        "name": "turn",
        "read": true,
        "write": true,
        "grantRead": true,
        "grantWrite": false
      },
      {
        "type": "PROPERTY",
        "name": "macaddr",
        "read": true,
        "write": false,
        "grantRead": false,
        "grantWrite": false
      }     
    ],
    "tenantExportingPermission": "PRIMARY",
    "tenantExporting": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
    },
    "application": {
      "href": "https://vanilla-ice.cloudthing.io/api/v1/applications/aPpl1Cat10n1D"
    }
  }

Export Operations
-----------------

Create An Export
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /applications/{applicationId}/exports``
    - Required: ``modelType``, ``export``, ``tenantExportingPermission``. Required if ``modelType`` is ``DEVICE``: ``product``. Optional: ``limitsType``, ``limits``.
    - Creates new Export.

Retrieve An Export
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``GET /exports/{exportId}``
    - ``expand``
    - Retrieves the Export with the specified ID. Expandable links: ``product``, ``limits``, ``application``, ``tenantExporting``.

Update An Export
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Attributes
    - Description

  * - ``POST /exports/{exportId}``
    - ``modelType``, ``export``, ``product``, ``limitsType``, ``limits``
    - Updates the Export with the specified ID.

Delete An Export
^^^^^^^^^^^^^^^^^^

.. list-table::
  :widths: 40 20 40
  :header-rows: 1

  * - Operation
    - Optional Query Parameters
    - Description

  * - ``DELETE /exports/{exportId}``
    - N/A
    - Deletes the Export with the specified ID.

Example Query
"""""""""""""

.. code-block:: bash

  curl -u "user@example.com:password" \
  "https://vanilla-ice.cloudthing.io/api/v1/exports/3xP0rT1D1234" \
  -H 'Accept: application/json'


******************
MQTT Pub/Sub API Reference
******************