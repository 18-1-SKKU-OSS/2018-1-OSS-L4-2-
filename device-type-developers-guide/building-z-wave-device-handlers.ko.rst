.. _zwave-device-handlers:

Z-Wave Device Handlers 구성하기
===============================

Z-Wave의 공용 사양은 `여기 <http://z-wave.sigmadesigns.com/design-z-wave/z-wave-public-specification/>`__ 에서 확인할 수 있습니다.
SmartThings는 Z-Wave디바이스가 정보를 보내고 요청하는 데 사용하는 표준 명령과 메시지를 나타내는 사용자 정의 Z-Wave명령 개체를 제공합니다.

----

이벤트 분석하기
--------------

Z-Wave devices의 이벤트가 당신의 Device Handler 파싱 메소드로 전달될 때, 그 이벤트들은 인코딩된 문자열 형식으로 되어 있습니다.
가장 먼저 구문 분석 방법은 설명 문자열에서 ``zwave.parse`` 를 호출하여 Z-Wave명령 객체로 변환하는 것입니다.
이 오브젝트의 클래스는 Z-Wave명령에서 볼 수 있는 ``physicalgraph.zwave.Command`` 의 하위 분류 중 하나이다.
`<https://graph.api.smartthings.com/ide/doc/zwave-utils.html>`__ 를 참조하세요.
설명 문자열이 유효한 Z-Wave명령을 나타내지 않는 경우 ``zwave.parse`` 는 ``null`` 을 반환합니다.

.. code-block:: groovy

    def parse(String description) {
        def result = null
        def cmd = zwave.parse(description)
        if (cmd) {
            result = zwaveEvent(cmd)
            log.debug "Parsed ${cmd} to ${result.inspect()}"
        } else {
            log.debug "Non-parsed event: ${description}"
        }
        return result
    }

명령 객체가 있는 경우 이를 처리하는 권장 방법은 이 예에서 사용하는 ``zwaveEvent()`` 와 같은 과부하 기능에 너가 다루려고 하는 명령어들의 타입에 따라 다른 인자 타입을 전달하는 것입니다:

.. code-block:: groovy

    def zwaveEvent(physicalgraph.zwave.commands.basicv1.BasicReport cmd)
    {
        def result
        if (cmd.value == 0) {
            result = createEvent(name: "switch", value: "off")
        } else {
            result = createEvent(name: "switch", value: "on")
        }
        return result
    }

    def zwaveEvent(physicalgraph.zwave.commands.meterv3.MeterReport cmd) {
        def result
        if (cmd.scale == 0) {
            result = createEvent(name: "energy", value: cmd.scaledMeterValue, unit: "kWh")
        } else if (cmd.scale == 1) {
            result = createEvent(name: "energy", value: cmd.scaledMeterValue, unit: "kVAh")
        } else {
            result = createEvent(name: "power", value: cmd.scaledMeterValue, unit: "W")
        }
        return result
    }

    def zwaveEvent(physicalgraph.zwave.Command cmd) {
        // 이 명령은 다른 zwaveEvent 인스턴스에서 처리되지 않은 명령을 캡처하므로 장치에서 보내는 모든 명령을 볼 수 있습니다.
        return createEvent(descriptionText: "${device.displayName}: ${cmd}")
    }

Remember that when you use ``createEvent()`` to build an Event, the resulting map must be returned from ``parse()`` for the Event to be sent.
For information about ``createEvent``, see the `Creating Events <parse.html#creating-events>`__ section.

As the `Z-Wave Command Reference <https://graph.api.smartthings.com/ide/doc/zwave-utils.html>`__ shows, many Z-Wave command classes have multiple versions.
By default, ``zwave.parse()`` will parse a command using the highest version of the command class.
If the device is sending an earlier version of the command, some fields may be missing, or the command may fail to parse and return ``null``.
To fix this, you can pass in a map as the second argument to ``zwave.parse()`` to tell it which version of each command class to use:

.. code-block:: groovy

    zwave.parse(description, [0x26: 1, 0x70: 1])

This example will use version 1 of SwitchMultilevel (0x26) and Configuration (0x70) instead of the highest versions.

----

Sending commands
----------------

To send a Z-Wave command to the device, you must create the command object, call ``format()`` on it to convert it to the encoded string representation, and return it from the command method.

.. code-block:: groovy

    def on() {
        return zwave.basicV1.basicSet(value: 0xFF).format()
    }

There is a shorthand provided to create command objects: ``zwave.basicV1.basicSet(value: 0xFF)`` is the same as ``new physicalgraph.zwave.commands.basicv1.BasicSet(value: 0xFF)``.
Note the different capitalization of the command name and the 'V' in the command class name.

The value 0xFF passed in to the command is a hexadecimal number.
Many Z-Wave commands use 8-bit integers to represent device state.
Generally 0 means "off" or "inactive", 1-99 are used as percentage values for a variable level attribute, and 0xFF or 255 (the highest value) means "on" or "detected".

If you want to send more than one Z-Wave command, you can return a list of formatted command strings.
It is often a good idea to add a delay between commands to give the device an opportunity to finish processing each command and possibly send a response before receiving the next command.
To add a delay between commands, include a string of the form ``"delay N"`` where N is the number of milliseconds to delay.
There is a helper method ``delayBetween()`` that will take a list of commands and insert delay commands between them:

.. code-block:: groovy

    def off() {
        delayBetween([
            zwave.basicV1.basicSet(value: 0).format(),
            zwave.switchBinaryV1.switchBinaryGet().format()
        ], 100)
    }

This example returns the output of ``delayBetween``, and thus will send a BasicSet command, followed by a 100 ms delay (0.1 seconds), then a SwitchBinaryGet command in order to check immediately that the state of the switch was indeed changed by the *set* command.

----

Sending commands in response to Events
--------------------------------------

In some situations, instead of sending a command in response to a request by the user, you want to automatically send a command to the device on receipt of a Z-Wave command.

If you return a list from the parse method, each item of the list will be evaluated separately.
Items that are maps will be processed as Events as usual and sent to subscribed SmartApps and mobile clients.
Returned items that are HubAction items, however, will be sent via the Hub to the device, in much the same way as formatted commands returned from command methods.
The easiest way to send a command to a device in response to an Event is the ``response()`` helper, which takes a Z-Wave command or encoded string and supplies a HubAction:

.. code-block:: groovy

    def zwaveEvent(physicalgraph.zwave.commands.wakeupv1.WakeUpNotification cmd)
    {
        def event = createEvent(descriptionText: "${device.displayName} woke up", displayed: false)
        def cmds = []
        cmds << zwave.batteryV1.batteryGet().format()
        cmds << "delay 1200"
        cmds << zwave.wakeUpV1.wakeUpNoMoreInformation().format()
        [event, response(cmds)] // return a list containing the event and the result of response()
    }

The above example uses the ``response()`` helper to send Z-Wave commands and delay commands to the device whenever a WakeUpNotification Event is received.
The reception of this Event that indicates that the sleepy device is temporarily listening for commands.
In addition to creating a hidden Event, the handler will send a BatteryGet request, wait 1.2 seconds for a response, and then issue a WakeUpNoMoreInformation command to tell the device it can go back to sleep to save battery.

