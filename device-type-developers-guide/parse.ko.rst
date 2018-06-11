분석과 이벤트들
================

``parse`` 메소드는 Device Handler의 핵심 메소드 입니다.

----

개요
--------

디바이스에서 나오는 모든 메세지들은 ``parse()`` 메소드로 전달됩니다.

이러한 메시지를 스마트 싱스 플랫폼이 이해할 수 있는 것으로 전환하는 것은 책임이 있다.

``parse()`` 메소드는 원시 기기 메시지 처리를 담당하기 때문에 구현 방식은 기기 유형에 따라 크게 다르다.
이 문서에서는 이러한 다양한 시나리오에 대해 설명하지는 않습니다 (`Z-Wave Device Handler Guide <building-z-wave-device-handlers.html>`__ or `ZigBee Device Handler guide <building-zigbee-device-handlers.html>`__ 를 참고하세요).

단순화된 ``parse()`` 메소드의 예를 들어 보겠습니다(CentraLiteSwitch에서 수정).

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

우리의 ``parse()`` 메소드는 통과된 설명을 검사하고, 이름"스위치"와 "on"또는"off"값을 가진 이벤트를 생성한다.
그런 다음 생성된 이벤트를 반환합니다. 여기서 SmartThings플랫폼은 이벤트 시작을 처리하고 해당 이벤트에 가입된 SmartApps에게 알립니다.

----

구문 분석, 이벤트 및 속성
-----------------------------

"스위치"기능은 가능한 값이 "on"및"off"인 "스위치"속성을 지정합니다.
*그* **"구문 분석()"*방법은 해당 장치의 기능 속성에 대한 이벤트를 생성하는 데 책임이 있습니다.*

이는 장치 핸드 북에 대해 이해하는 중요한 요점입니다. 즉, SmartApps가 이벤트 구독에 응답할 수 있도록 해 줍니다!

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
