Building the Device Handler
===========================

The Device Handler for a Cloud-connected device is generally the same as any other Device Handler.
The means in which
it handles sending and receiving messages from its device is a little bit different.
Let's walk through a Cloud-connected
Device Handler example.

The Parse method
----------------

The parse method for Cloud-connected devices will always be empty. In a
Cloud-connected device, Event data is passed down from the Service
Manager, not from the device itself, so the parsing is handled in a
separate method. The Device Handler doesn't interface directly with
a hardware device, which is what parse is used for.

Sending commands to the third-party cloud
-----------------------------------------

Usually the actual implementation of device methods are delegated to its Service Manager. This is because the Service
Manager is the entity that has the authentication information. To invoke a method on the parent Service Manager,
you simply need to call it in the following format:

.. code-block:: groovy

    parent.methodName()

As with any other device-type, you need to define methods for all of the
possible commands for the capabilities you'd like to support. Then when
a user calls this method, it will pass information up to the parent
Service Manager, who will make the direct connection to the third party
cloud. You might for example want to turn a switch on, so you would call
the following.

.. code-block:: groovy

    def on() {
      parent.on(this)
    }

Receiving Events from the third-party cloud
-------------------------------------------

The Device Handler continuously polls the third-party cloud through
the service manager to check on the status of devices. When an Event is
fired, they can then be passed to the child Device Handler. Note that
poll runs every 10 minutes for Service Manager SmartApps.

In the device-type handler:

.. code-block:: groovy

    def poll() {
        results = parent.pollChildren()
        parseEventData(results)
    }

    def parseEventData(Map results){
        results.each { name, value ->
            //Parse events and optionally create SmartThings events
        }
    }

In the service manager:

.. code-block:: groovy

    def pollChildren(){
        def pollParams = [
            uri: "https://api.thirdpartysite.com",
            path: "/device",
            requestContentType: "application/json",
            query: [format:"json",body: jsonRequestBody]
            ]

            httpGet(pollParams) { resp ->
                state.devices = resp.data.devices { collector, stat ->
                def dni = [ app.id, stat.identifier ].join('.')
                def data = [
                    attribute1: stat.attributeValue,
                    attribute2: stat.attribute2Value
                ]
                collector[dni] = [data:data]
                return collector
                }
            }
    }

Generating Events at the request of the Service Manager
-------------------------------------------------------

You won't generate events directly within the Service Manager, but
rather request that they are generated within the Device Handler.
For example:

In the service manager:

.. code-block:: groovy

    childName.generateEvent(data)

In the Device Handler:

.. code-block:: groovy

    def generateEvent(Map results) {
      results.each { name, value ->
        sendEvent(name: name, value: value)
      }
      return null
    }
