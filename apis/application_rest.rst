.. CloudThing documentation master file, created by
   sphinx-quickstart on Sun May  8 19:31:11 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

HTTP REST API
======================================

General concept
--------------------------------

**Methods**

REST API uses JSON as serialization format and requires proper *Content-Type* and *Accept* headers in every client request.

API uses following HTTP verbs (methods):

- **GET** (collection) - retrieves collection,
- **GET** (item) - retrieves single item,
- **POST** (collection) - creates new item in collection,
- **POST** (item) - updates item,
- **DELETE** (item) - deletes item.

API **does not** support **PUT** since it is not possible to replace objects and **PATCH** which as we believe is commonly misunderstood and being used in wrong way (for updates).

**Linking**

CloudThing API implements great linking concept developed by awesome people at stormpath.com.
Every response (collection or object) is identified by its URL in *"href"* field, there is no other *id* provided. All relations between objects use this method and embed *href* in JSON object named as relative. It is possible to expanding relations in single request (one level only) by providing relation name to *expand* parameter in query.

In short words:

Just hit /api/v1/tenants/current and start exploring!

**Host**

HTTP API accept only encrypted (TLS) requests. All HTTP non-TLS requests will be redirected.
**Note that your credential still be exposed if you use plain HTTP!**

The HTTP server listens on standard port 443 of tenant's virtual host.

**Authentication**

CloudThing API supports Basic and Bearer (JWT) authentication. Basic auth needs:

- Username: {apiKey} or {userEmail},
- Password: {apiSecret} or {userPassword}.

To obtain JWT token one should send POST request to */api/v1/auth/token* endpoint with Basic auth.

Tenant
--------------------------------

Tenant is an organization on behalf of which user or API key requests data. When registering an account at CloudThing.io the organization and user within it are created.

/tenants/{id}
############################

**GET**

Reads details of user's organization:

.. code-block:: bash

	curl -u "user@example.com:password" \
	https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT

Response::

	{
	  "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT",
	  "name": "user@example.com's organization",
	  "shortName": "vanilla-ice",
	  "createdAt": "2016-05-15T11:18:33Z",
	  "updatedAt": "2016-05-15T11:18:33Z",
	  "directories": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/directories"
	  },
	  "applications": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/applications"
	  },
	  "products": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/products"
	  },
	  "custom": {

	  }
	}

**POST**

Updates details of user's organization:

.. code-block:: bash

	curl -u "user@example.com:password" \
	-H "Content-Type: application/json" -X POST \
	https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT \
	-d '{"name": "Example, Inc"}'

Response::

	{
	  "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT",
	  "name": "Example, Inc",
	  "shortName": "vanilla-ice",
	  "createdAt": "2016-05-15T11:18:33Z",
	  "updatedAt": "2016-05-22T14:12:02Z",
	  "directories": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/directories"
	  },
	  "applications": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/applications"
	  },
	  "products": {
	    "href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT/products"
	  },
	  "custom": {

	  }
	}

/tenants/current
############################

**GET**

Retrieves link to current tenant:

.. code-block:: bash

	curl -u "user@example.com:password" \
	https://vanilla-ice.cloudthing.io/api/v1/tenants/current

Response::

	HTTP/1.1 302 Found
	Content-Type: application/json
	Location: https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT

	{
		"tenant": {
			"href": "https://vanilla-ice.cloudthing.io/api/v1/tenants/Som31D0fT3NAnT"
		}
	}


Device
--------------------------------

Device's data (resources) can be retrieved by hitting /api/v1/devices/{id}/resources{data,events,commands}/{key} (eg. */api/v1/devices/s0m31D/resources/data/temp*).