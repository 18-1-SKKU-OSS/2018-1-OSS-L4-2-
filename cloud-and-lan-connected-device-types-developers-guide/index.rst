.. _cloud_lan_device_type_guide:

Cloud- and LAN-connected Devices
================================

Cloud- and LAN-connected devices are devices that use either a third-party service, like the Ecobee thermostat, or communicate over the LAN (local area network) like the Sonos system.
These devices require a unique implementation of their Device Handlers.
Cloud- and LAN-connected devices use a Service Manager SmartApp along with a Device Handler for authentication, maintaining connections, and device communications.
This guide will walk you through Service Manager and Device Handler creation for both of these scenarios.

Table of Contents:

.. toctree::
   :maxdepth: 3

   understanding-the-service-manage-device-handler-design-pattern
   building-cloud-connected-device-types/index
   building-lan-connected-device-types/index
   autodiscover-lan.rst
   working-with-images.rst
