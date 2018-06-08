.. _building_servicemanager:

============================
Building the Service Manager
============================

The Service Manager's responsibilities are:

- **Discovery** - Discover devices on the LAN via SSDP (mDNS/DNS-SD or Bonjour is not currently supported)
- **Verification** - Verify each discovered device through successful fetching of the UPnP device description
- **Inclusion** - Add the device as a child of the service manager
- **Health** - Track IP address and port changes and allow these to make it down to the child device(s) as necessary

Let's take a look at some of the key parts of a Service Manager implementation. The example code referenced throughout
this document is derived from the below SmartApp and DeviceType:

:download:`Generic UPnP Service Manager <src/generic-upnp-service-manager.groovy>`
:download:`Generic UPnP Device <src/generic-upnp-device.groovy>`

The above referenced Generic UPnP Device is incomplete. For the complete guide, please see `Building the Device Type <building-the-device-type.html>`_.

----

.. _lan_device_discovery:

Discovery
---------

Simple Service Discovery Protocol (SSDP) is the main protocol used to find devices on your network. It serves as the backbone of Universal Plug and Play (UPnP), which
allows you to easily connect new network devices to a system. See `UPnP Device Architecture 1.1 <http://upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf>`__
for full specification details.

To discover new devices, you first need to subscribe to Location Events with the correct **search target** for the device. The
search target in the below example, **urn:schemas-upnp-org:device:ZonePlayer:1**, is for discovery of a Sonos, but search targets will vary by manufacturer and device.
For UPnP, this information should be published on documentation for the device, but you may
alternatively have to contact the manufacturer directly to obtain it. Here is the Event subscription:

.. code-block:: groovy

    subscribe(location, "ssdpTerm.urn:schemas-upnp-org:device:ZonePlayer:1", ssdpHandler)

This means that any time an SSDP **search response** with a search target of urn:schemas-upnp-org:device:ZonePlayer:1
(e.g. Sonos) is received from a Hub in this Location, it will fire the ssdpHandler method.

Next, you need to send an appropriate discovery command for the desired search target:

.. code-block:: groovy

    void ssdpDiscover() {
        sendHubCommand(new physicalgraph.device.HubAction("lan discovery urn:schemas-upnp-org:device:ZonePlayer:1", physicalgraph.device.Protocol.LAN))
    }

.. note:: HubAction is a class supplied by the SmartThings platform

    The class ``physicalgraph.device.HubAction`` encapsulates request information
    for communicating with the device.

    When you create an instance of a ``HubAction``, you provide details about the
    request, such as the request method, headers, and path. By itself, ``HubAction`` is little more than a wrapper for these request details.
    In this case, it's a thin wrapper around discovery information.

In the above HubAction example, the main message to be sent through the Hub is:

.. code-block:: groovy

    lan discovery urn:schemas-upnp-org:device:ZonePlayer:1

This is converted by our device connectivity layer into an M-SEARCH multicast request that is sent to the LAN via the Hub, and
should look something like the following:

.. code-block:: bash

    M-SEARCH * HTTP/1.1
    HOST: 239.255.255.250:1900
    MAN: "ssdp:discover"
    MX: 4
    ST: urn:schemas-upnp-org:device:ZonePlayer:1

After the end device receives the multicast M-SEARCH, it is supposed to issue a unicast **search response**, delayed by a random number of seconds between 0 and MX (4 in this case).
The search response sent from the device back to the Hub should look something like this:

.. code-block:: bash

    HTTP/1.1 200 OK
    CACHE-CONTROL: max-age=100
    EXT:
    LOCATION: http://10.0.1.14:80/xml/device_description.xml
    SERVER: FreeRTOS/6.0.5, UPnP/1.0, IpBridge/0.1
    ST: urn:schemas-upnp-org:device:ZonePlayer:1
    USN: uuid:RINCON_000E58F0FFFFFF400::urn:schemas-upnp-org:device:ZonePlayer:1

This will get routed back to the cloud where it will be converted into an Event that will fire the ssdpHandler method with the following description:

.. code-block:: bash

    devicetype:04, mac:000E58F0FFFF, networkAddress:0A00010E, deviceAddress:0578, stringCount:04, ssdpPath:/xml/device_description.xml, ssdpUSN:uuid:RINCON_000E58F0FFFFFF400::urn:schemas-upnp-org:device:ZonePlayer:1, ssdpTerm:urn:schemas-upnp-org:device:ZonePlayer:1, ssdpNTS:

The ssdpHandler method should record the data from the search response, in preparation for verification.

.. code-block:: groovy

    def ssdpHandler(evt) {
        def description = evt.description
        def hub = evt?.hubId

        def parsedEvent = parseEventMessage(description)
        parsedEvent << ["hub":hub]

        def devices = getDevices()
        String ssdpUSN = parsedEvent.ssdpUSN.toString()
        if (!devices."${ssdpUSN}") {
            devices << ["${ssdpUSN}": parsedEvent]
        }
    }

----

Verification
------------

Once we've recorded the presence of a device on the LAN with the desired SSDP search target, the next step is to verify the
availability of the device by fetching some more information about it. In UPnP, this is called the **device description**.
In the search response, there is a LOCATION header which shows the Location of the device description on the LAN. SmartThings
splits this into **networkAddress**, **deviceAddress**, and **ssdpPath** in the Event, which at this point should exist in app state.
This can be pulled out of state and put into a HubAction. Note that the HubAction has a **callback**, which means that
when an HTTP response is issued from the device to the Hub, it will fire the **deviceDescriptionHandler** method.

.. code-block:: groovy

    void verifyDevices() {
        def devices = getDevices().findAll { it?.value?.verified != true }
        devices.each {
            int port = convertHexToInt(it.value.port)
            String ip = convertHexToIP(it.value.ip)
            String host = "${ip}:${port}"
            sendHubCommand(new physicalgraph.device.HubAction("""GET ${it.value.ssdpPath} HTTP/1.1\r\nHOST: $host\r\n\r\n""", physicalgraph.device.Protocol.LAN, host, [callback: deviceDescriptionHandler]))
        }
    }

    void deviceDescriptionHandler(physicalgraph.device.HubResponse hubResponse) {
        def body = hubResponse.xml
        def devices = getDevices()
        def device = devices.find { it?.key?.contains(body?.device?.UDN?.text()) }
        if (device) {
            device.value << [name: body?.device?.roomName?.text(), model: body?.device?.modelName?.text(), serialNumber: body?.device?.serialNum?.text(), verified: true]
        }
    }

.. note:: HubResponse is a class supplied by the SmartThings platform. Here are some pieces of data that are included:

    * **description** - The raw message received by the device connectivity layer
    * **hubId** - The UUID of the SmartThings Hub that received the response
    * **status** - HTTP status code of the response
    * **headers** - Map of the HTTP headers of the response
    * **body** - String of the HTTP response body
    * **error** - Any error encountered during any automatic parsing of the body as either JSON or XML
    * **json** - If the HTTP response has a Content-Type header of application/json, the body is automatically parsed as JSON and stored here
    * **xml** - If the HTTP response has a Content-Type header of text/xml, the body is automatically parsed as XML and stored here

----

Inclusion
---------

Now that the device has been verified, we need to add it as a child device.

.. code-block:: groovy

    def addDevices() {
        def devices = getDevices()

        selectedDevices.each { dni ->
            def selectedDevice = devices.find { it.value.mac == dni }
            def d
            if (selectedDevice) {
                d = getChildDevices()?.find {
                    it.deviceNetworkId == selectedDevice.value.mac
                }
            }

            if (!d) {
                log.debug "Creating Generic UPnP Device with dni: ${selectedDevice.value.mac}"
                addChildDevice("smartthings", "Generic UPnP Device", selectedDevice.value.mac, selectedDevice?.value.hub, [
                    "label": selectedDevice?.value?.name ?: "Generic UPnP Device",
                    "data": [
                        "mac": selectedDevice.value.mac,
                        "ip": selectedDevice.value.ip,
                        "port": selectedDevice.value.port
                    ]
                ])
            }
        }
    }

.. note:: It's important to **not** use IP and port as the DNI (Device Network ID) of the device. This is because if/when the IP
    address changes, we do not want to update the device's DNI. Instead, we choose MAC address as DNI, which is guaranteed not
    to change.

----

.. _lan_device_health:

Health
------

Lastly, we need to handle the possibility of IP address or port changes. Unless you have setup a static DHCP reserveration in
your network router, there is a possibility that the IP address of the device will change, and the child device can be told
when this changes by the Service Manager. We'll start by modifying the above ssdpHandler method to handle changing IP and port data:

.. code-block:: groovy

    def ssdpHandler(evt) {
        def description = evt.description
        def hub = evt?.hubId

        def parsedEvent = parseEventMessage(description)
        parsedEvent << ["hub":hub]

        def devices = getDevices()
        String ssdpUSN = parsedEvent.ssdpUSN.toString()
        if (devices."${ssdpUSN}") {
            def d = devices."${ssdpUSN}"
            if (d.ip != parsedEvent.ip || d.port != parsedEvent.port) {
                d.ip = parsedEvent.ip
                d.port = parsedEvent.port
                def child = getChildDevice(parsedEvent.mac)
                if (child) {
                    child.sync(parsedEvent.ip, parsedEvent.port)
                }
            }
        } else {
            devices << ["${ssdpUSN}": parsedEvent]
        }
    }

This assumes that the DeviceType has a **sync** method that has the ability to alter the internally stored ip and port.

.. code-block:: groovy

    def sync(ip, port) {
        def existingIp = getDataValue("ip")
        def existingPort = getDataValue("port")
        if (ip && ip != existingIp) {
            updateDataValue("ip", ip)
        }
        if (port && port != existingPort) {
            updateDataValue("port", port)
        }
    }

Finally, we need to make sure that the M-SEARCH for our desired search target is periodically sent out over the LAN. We can
use the scheduler to do that from the Service Manager:

.. code-block:: groovy

    runEvery5Minutes("ssdpDiscover")

----

Best practices
--------------

For LAN Service Manager SmartApps, there are a couple items to keep in mind that might not be immediately apparent.

* Use something static as the DNI for the child device, such as MAC address.
* Avoid making calls from your child devices into the parent if possible, as this can lead to increased latency and unnecessary platform load. Instead, supply your child devices with enough information to make calls into the parent unnecessary, and use the Service Manager to manage any child device updates that need to happen based on network changes.

----

References and resources
------------------------

- `UPnP Device Architecture 1.1 <http://upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf>`__
