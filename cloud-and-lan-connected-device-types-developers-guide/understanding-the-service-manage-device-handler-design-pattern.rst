Service Manager Design Pattern
==============================

Basic overview
--------------

Devices that connect through the internet as a whole (cloud) or LAN
devices (on your local network) require a defined Service-Manager
SmartApp, in addition to the usually expected Device Handler.
The Service Manager makes the connection with the device, handling the input and output interactions, and the Device Handler parses messages.

Cloud-connected devices
-----------------------

When using a Cloud-connected device, the service manager is used to
discover and initiate a connection between the device and your Hub,
using OAuth connections to external third parties.
Then the Device Handler uses this connection to communicate between the Hub and device.

LAN-connected devices
---------------------

When using a LAN-connected device, the service manager is used to
discover and initiate a connection between the device and your Hub,
using the protocols SSDP or mDNS/DNS-SD.
Then the Device Handler uses UPnP/SOAP Calls or REST Calls to communicate outgoing messages between
the Hub and device.
