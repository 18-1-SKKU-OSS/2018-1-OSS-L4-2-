Building Cloud-connected Device Types
=====================================

Cloud-connected devices use a third-party service to accomplish device communication.
An example of such a device is the
Ecobee thermostat.

When developing a Device Handler for a Cloud-connected device, you must create a Service Manager SmartApp that will handle authenticating with the third-party service, communicating with the device, and reacting to any device changes that occur.

This guide overviews the concept of the Service Manager/Device Handler architecture and
also gives an example of both the Service Manager and Device Handler creation.

Table of Contents:

.. toctree::
   :maxdepth: 2

   division-of-labor
   building-the-service-manager
   building-the-device-type
