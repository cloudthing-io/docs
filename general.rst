.. CloudThing documentation master file, created by
   sphinx-quickstart on Sun May  8 19:31:11 2016.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

General Concept
======================================

CloudThing Cloud Platform is a software layer between hardware devices and Internet of Things applications. It may be considered as middleware for end-to-end IoT solutions.

CloudThing features set of APIs for connecting and managing devices, as well as for services and apps development and management.

The CT cloud platform is multi-tenant application with strong separation between tenants' data. Every new tenant has his own virtual host created accessible by {shortName}.cloudthing.io. Short names are automatically generated bundles of two words, if you want custom one get in touch hello@cloudthing.io.

Example: if generated name was *vanilla-ice* then tenant's vhost would be exposed via *vanilla-ice.cloudthing.io* and services would ve accessible via:

- **Control Center:** *https://vanilla-ice.cloudthing.io*,
- **HTTP API:** *https://vanilla-ice.cloudthing.io/api/v1*,
- **MQTT API:** *mqtts://vanilla-ice.cloudthing.io:8883*,
- **MQTT Websockets API:** *wss://vanilla-ice.cloudthing.io:9001/mqtt*,
- **HTTP connector API:** *http://vanilla-ice.cloudthing.io:81/v1*,

etc.


