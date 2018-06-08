Building LAN-connected Device Types
===================================

LAN-connected devices communicate with the SmartThings Hub over the LAN.
An example of such a device is the Sonos system.

When developing a Device Handler for a LAN device, you must create a service manager
SmartApp that will handle discovery of devices on the LAN, in some cases communicate with the device, and react to any
device changes that occur via Events.

This guide overviews the concept of the Service Manager/Device Handler architecture
and also gives an example of both the Service Manager and Device Handler creation.

Table of Contents:

.. toctree::
   :maxdepth: 2

   division-of-labor
   building-the-service-manager
   building-the-device-type
