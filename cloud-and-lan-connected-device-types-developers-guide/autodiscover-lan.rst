.. _automatic_LAN_device_discovery:

==============================
Automatic LAN Device Discovery
==============================

Automatic LAN device discovery minimizes the complexity in discovering LAN-connected devices. 

Normally the SmartThings platform will discover a LAN-connected or a Cloud-connected device only when a Service Manager SmartApp for that specific device is present.
This means that if you want to integrate multiple LAN devices, such as a Wemo motion sensor and a Bose Speaker, then you will need multiple Service Manager SmartApps, i.e., a separate Service Manager SmartApp for each LAN-connected device.  
On the contrary, the platform does not have any such Service Manager SmartApp requirement for a ZigBee or a Z-Wave device.

The new automatic LAN device discovery eliminates the Service Manager SmartApp requirement for some LAN-connected devices, thereby making for a much smoother and quicker LAN-connected device discovery.
See :ref:`autodiscoverlan-devices-list`.

Impact on the developer
-----------------------

For the :ref:`autodiscoverlan-devices-list` if you have made any customizations to either your Service Manager SmartApp or your Device Handler, then your LAN-connected device integration will be impacted.
See the table below.

===================== =============================== ======
Custom Device Handler Custom Service Manager SmartApp Impact
===================== =============================== ======
Yes                   No                              Custom LAN Device Handler is overwritten with the SmartThings version.
Yes                   Yes                             No impact
===================== =============================== ======

.. _autodiscoverlan-devices-list:

Supported LAN-connected Devices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Currently a limited number of LAN-connected devices can be discovered with automatic LAN device discovery.
See `How to connect Wi-Fi devices <https://support.smartthings.com/hc/articles/115001164026>`_. 


