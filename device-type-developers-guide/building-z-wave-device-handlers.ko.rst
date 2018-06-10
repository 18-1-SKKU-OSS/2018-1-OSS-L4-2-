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

``createEvent()`` 를 사용하여 이벤트를 작성할 때 이벤트를 전송하려면 ``parse()`` 에서 결과 맵을 반환해야 합니다. ``createEvent`` 에 대한 자세한 내용은 `Creating Events <parse.html#creating-events>`__ 섹션을 참조하십시오.

`Z-Wave Command Reference <https://graph.api.smartthings.com/ide/doc/zwave-utils.html>`__ 에 표시된 것처럼 많은 Z-Wave명령 클래스에는 여러 버전이 있습니다. 기본적으로 ``zwave.parse()`` 는 명령 클래스의 가장 높은 버전을 사용하여 명령을 구문 분석합니다. 장치가 이전 버전의 명령을 보내는 경우 일부 필드가 누락되었거나 명령이 구문 분석하지 못하고 ``null`` 을 반환할 수 있습니다. 이 문제를 해결하기 위해 맵을  ``zwave.parse()`` 에 두번째 인수로 전달하여 사용할 각 명령 클래스의 버전을 지정할 수 있습니다.    
    
.. code-block:: groovy

    zwave.parse(description, [0x26: 1, 0x70: 1])

이 예에서는 가장 높은 버전 대신 SwitchMultilevel (0x26)및 Configuration (0x70)의 버전 1을 사용합니다.

----

Sending commands
----------------

Z-Wave명령을 장치에 전송하려면 명령 개체를 생성하고 이 개체에 대해 ``format()``  을 호출하여 인코딩된 문자열 표현으로 변환한 후 명령 방법에서 반환해야 합니다.

.. code-block:: groovy

    def on() {
        return zwave.basicV1.basicSet(value: 0xFF).format()
    }

    명령 개체를 생성하기 위해 제공되는 속기는 다음과 같습니다. 즉, ``zwave.basicV1.basicSet(value: 0xFF)`` 은 ``new physicalgraph.zwave.commands.basicv1.BasicSet(value: 0xFF)`` 와 같습니다.
명령 클래스 이름에 명령 이름의 대문자화와 'V'가 서로 다르다는 점을 참고하십시오.

명령에 전달되는 값 0xFF는 16진수 숫자입니다.
대부분의 Z-Wave명령에서는 8비트 정수를 사용하여 디바이스 상태를 나타냅니다.
일반적으로 0은 "off" 또는 "비활성"을 의미하고 1-99는 가변 수준 속성의 백분율 값으로 사용되며, 255(가장 높은 값)은 "on"또는"detected"를 의미합니다.

Z-Wave명령을 두개 이상 전송하려면 형식이 지정된 명령 문자열 목록을 반환하면 됩니다.
종종 명령 사이에 지연을 추가하여 장치가 각 명령을 처리할 수 있도록 하고, 가능하면 다음 명령을 수신하기 전에 응답을 보내는 것이 좋습니다.
명령 간의 지연을 추가하려면 ``"delay N"`` 형식의 문자열을 포함합니다. 여기서 N은 지연 시간(밀리초)입니다.
``delayBetween()`` 같은 헬퍼 메소드로 명령 목록을 작성하고 지연 명령을 삽입할 수 있습니다.

.. code-block:: groovy

    def off() {
        delayBetween([
            zwave.basicV1.basicSet(value: 0).format(),
            zwave.switchBinaryV1.switchBinaryGet().format()
        ], 100)
    }

이 예에서는 ``delayBetween`` 의 출력을 반환하므로 BasicSet명령이 전송되고 100ms지연 (0.1초)후 SwitchBinaryGet 명령이 *set* 명령에 의해 즉시 변경됩니다.

----

Sending commands in response to Events
--------------------------------------

경우에 따라 사용자의 요청에 대한 응답으로 명령을 보내는 대신 Z-Wave명령을 수신할 때 자동으로 장치로 명령을 보내려고 합니다.

구문 분석 메소드에서 목록을 반환하면 목록의 각 항목이 별도로 평가됩니다.
맵인 항목은 평상시와 같이 이벤트로 처리되어 가입된 SmartApps및 모바일 클라이언트로 전송됩니다.
그러나  HubAction 항목이 반환되면 Hub를 통해 장치로 전송되며 이는 명령 메서드에서 반환된 형식의 명령과 매우 유사합니다.
이벤트에 대한 응답으로 장치에 명령을 보내는 가장 쉬운 방법은 Z-wave명령이나 인코딩된 문자열을 사용하여 다음과 같은 HubAction 문자열을 제공하는 ``response()`` 헬퍼 입니다.

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

위의 예에서는 WakeUpNotification이벤트가 수신될 때마다 Z-Wave명령과 지연 명령을 장치에 보내는 ``response()`` 헬퍼를 사용합니다.
절전 모드에서 명령을 일시적으로 수신 중임을 나타내는 이 이벤트의 수신입니다.
숨겨진 이벤트를 만드는 것 외에도 핸들러는 BatteryGet 요청을 전송하고 응답을 1.2초간 기다린 다음 절전 모드 정보 저장 명령을 실행하여 장치를 다시 시작할 수 있음을 알려 줍니다.
