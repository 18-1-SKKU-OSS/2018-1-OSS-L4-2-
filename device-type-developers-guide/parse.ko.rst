분석과 이벤트들 
================

``parse`` 메소드는 Device Handler의 핵심 메소드 입니다.

----

개요
--------

All messages from the device are passed to the ``parse()`` method.
It is responsible for turning those messages into something the SmartThings platform can understand.

Because the ``parse()`` method is responsible for handling raw device messages, their implementations vary greatly across different device types.
This document will not discuss all these different scenarios (see the `Z-Wave Device Handler Guide <building-z-wave-device-handlers.html>`__ or `ZigBee Device Handler guide <building-zigbee-device-handlers.html>`__ for protocol-specific information).

Consider an example of a simplified ``parse()`` method (modified from the CentraLite Switch):

.. code-block:: groovy

    def parse(String description) {
        log.debug "parse description: $description"

        def attrName = null
        def attrValue = null

        if (description?.startsWith("on/off:")) {
            log.debug "switch command"
            attrName = "switch"
            attrValue = description?.endsWith("1") ? "on" : "off"
        }

        def result = createEvent(name: attrName, value: attrValue)

        log.debug "Parse returned ${result?.descriptionText}"
        return result
    }

Our ``parse()`` method inspects the passed-in description, and creates an Event with name "switch" and a value of "on" or "off".
It then returns the created Event, where the SmartThings platform will handle firing the Event and notifying any SmartApps subscribed to that Event.

----

Parse, Events, and Attributes
-----------------------------

Recall that the "switch" capability specifies an attribute of "switch", with possible values "on" and "off".
*The* ``parse()`` *method is responsible for creating events for the attributes of that device's capabilities.*

That is a critical point to understand about Device Handlers - it is what allows SmartApps to respond to Event subscriptions!

.. note::

    Only events that constitute a state change are propagated through the SmartThings platform. A state change is when a particular attribute of the device changes. This is handled automatically by the platform, but should you want to override that behavior, you can do so by specifying the ``isStateChange`` parameter discussed below.

Creating Events
^^^^^^^^^^^^^^^

Use the ``createEvent()`` method to create events in your Device Handler.
It takes a map of parameters as an argument.
You should provide the ``name`` and ``value`` at a minimum.

.. important::

    The createEvent just creates a data structure (a Map) with information about the Event. *It does not actually fire an Event.*

    Only by returning that created map from your ``parse`` method will an Event be fired by the SmartThings platform.

The parameters you can pass to ``createEvent`` are:

*name* (required)
    String - The name of the Event. Typically corresponds to an attribute name of the device-handler's capabilities.
*value* (required)
    The value of the Event. The value is stored as a String, but you can pass in numbers or other objects. SmartApps will be responsible for parsing the Event's value into back to its desired form (e.g., parsing a number from a string)
*descriptionText*
    String - The description of this Event. This appears in the mobile application activity feed for the device. If not specified, this will be created using the Event name and value.
*displayed*
    boolean - ``true`` to display this Event in the mobile application activity feed. ``false`` to not display this Event. Defaults to ``true``.
*linkText*
    String - Name of the Event to show in the mobile application activity feed, if specified.
*isStateChange*
    boolean - ``true`` if this Event caused the device's attribute to change state. ``false`` otherwise. If not provided, ``createEvent`` will populate this based on the current state of the device.
*unit*
    String - a unit string, if desired. This will be used to create the descriptionText if it (the descriptionText parameter) is not specified.

Multiple Events
^^^^^^^^^^^^^^^

You are not limited to returning a single Event map from your ``parse`` method.

You can return a list of Event maps to tell the SmartThings platform to generate multiple events:

.. code-block:: groovy

    def parse(String description) {
        ...

        def evt1 = createEvent(name: "someName", value: "someValue")
        def evt2 = createEvent(name: "someOtherName", value: "someOtherValue")

        return [evt1, evt2]
    }

Generating Events outside of parse
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you need to generate an Event outside of the ``parse()`` method, you can use the ``sendEvent()`` method.
It simply calls ``createEvent()`` *and* fires the Event.
You pass in the same parameters as you do to ``createEvent()``.

----

Tips
----

When creating a Device Handler, determining what messages need to be handled by the ``parse()`` method varies by device.
A common practice to figure out what messages need to be handled is to simply log the messages in your ``parse()`` method (``log.debug "description: $description"``).
This allows you to see what the incoming message is for various actuations or states.
