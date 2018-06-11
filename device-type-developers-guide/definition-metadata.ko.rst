정의
==========

정의 메타 데이터는 장치 핸들러에 대한 핵심 정보를 정의합니다.
초기 값은 장치 핸들러를 작성할 때 입력한 값에서 설정됩니다.

정의 메타 데이터의 예:

.. code-block:: groovy

    metadata {
        definition(name: "test device", namespace: "yournamespace", author: "your name") {

            capability "Alarm"
            capability "battery"

            attribute "customAttribute", "string"

            command "customCommand"

            fingerprint profileId: "0104", inClusters: "0000,0003,0006",
                        outClusters: "0019"
        }

        ...
    }

정의 방법은 매개 변수 맵과 폐쇄를 사용합니다.

지원되는 매개 변수는 다음과 같습니다.

*name*
    Device Handler의 이름.
*namespace*
    이 장치 처리기의 네임 스페이스. 이것은 당신의 github사용자 이름이어야 합니다. 이는 다른 사용자가 동일한 이름을 사용한 경우에도 장치 핸드 북을 이름별로 조회하여 찾을 수 있는지 확인할 때 사용됩니다.
*author*
    Device Handler의 작성자.

폐쇄는 장치 핸들러의 기능, 특성, 명령 및 지문 정보를 정의합니다.

----

기능
------------

기기가 기능을 지원한다는 것을 정의하려면 폐쇄 상태에서``capability`` 메소드가 ``definition`` 로 전달되었다고 하면 됩니다.

 ``capability`` 메소드에 대한 주장은 능력 이름이다.

.. code-block:: groovy

    capability "Actuator"
    capability "Power Meter"
    capability "Refresh"
    capability "Switch"

----

특성
----------
장치 핸들러에 대한 사용자 지정 특성을 정의해야 하는 경우, 폐쇄에서 ``attribute()`` 메소드가 ``definition()`` 메소드로 전달됨을 호출합니다.

**attribute(String attributeName, String attributeType, List possibleValues = null)**

*attributeName*
    특성의 이름.

*attributeType*
    특성의 타입. 가능한 타입으로는 "string", "number", 그리고 "enum"가 있습니다.

*possibleValues*
    선택사항. 그 특성의 가능한 값들. 오직 "enum"과 쓸수 있다.

.. code-block:: groovy

    // 이름이 "someName"인 스트링 특성
    attribute "someName", "string"

    // "light" 그리고 "dark" 값이 가능한 enum 특성
    attribute "someOtherName", "enum", ["light", "dark"]


----

명령어들
--------

장치 핸들러에 대한 사용자 정의 명령을 정의하려면 종료 시 ``command()`` 메소드를 ``definition()`` 메소드로 호출하십시오.

**command(String commandName, List parameterTypes = [])**

*commandName*
    명령의 이름. Device Handler 내의 메소드도 반드시 같은 이름으로 해야합니다.
*parameterTypes*
    선택사항. 명령 메소드를 위한 파라미터 타입 리스트이며, 필요할 때만 쓰세요.

.. code-block:: groovy

    // 인자없는 "myCommand" 명령
    command "myCommand"

    // 스트링과 숫자를 인자로 받는 myCommandWithParams 명령
    command "myCommandWithParams", ["string", "number"]

    ...

    // 정의에 지정된 각 명령에는 해당 메서드가 있어야 합니다.

    def myCommand() {
        // 명령 쓰세요
    }

    // this command takes parameters as defined in the definition
    def myCommandWithParams(stringParam, numberParam) {
        // 명령 쓰세요
    }

----

Fingerprinting(번역 안하는게 더 맞는 거 같아 놔둡니다)
--------------

ZigBee또는 Z-Wave장치를 SmartThingsHub에 추가할 때 할당할 장치 유형을 결정할 방법이 필요합니다.
이 과정을 "조인"과정 또는 "fingerprinting"라고 합니다.

장치 취급자는 지원하는 장치 또는 장치 종류를 지정하기 위해 "fingerprints" 을 정의합니다.
그런 다음 장치가 추가되면 연결 정보가 기본 처리기의 모든 지문과
자체적으로 게시된 처리기를 사용하여 장치 유형을 확인합니다.

fingerprinting  과정은 ZigBee 그리고 Z-Wave 장치 사이에 차이가 있다.

.. _zigbee-fingerprinting-label:

ZigBee fingerprinting
^^^^^^^^^^^^^^^^^^^^^

ZigBee 기기의 경우 사용해야 할 주요 프로파일은 다음과 같습니다.

-HA: HomeAutomation(0104)
-SEP: Smart Energy Profile
-ZLL: ZigBee Light Link (C05E)

입력 및 출력 클러스터는 장치에서 특별히 정의하며 장치 설명서를 통해 사용할 수 있어야 합니다.

ZigBee 지문 정의의 예:

.. code-block:: groovy

        fingerprint profileId: "C05E", inClusters: "0000,0003,0004,0005,0006,0008,0300,1000", outClusters: "0019"

또한 지문에 제조 업체 및 모델 이름을 포함하여 특정 제품으로 지문을 제한할 수도 있습니다.

.. code-block:: groovy

    fingerprint inClusters: "0000,0001,0003,0020,0406,0500", manufacturer: "NYCE", model: "3014"

.. _zwave-fingerprinting:

Z-Wave fingerprinting
^^^^^^^^^^^^^^^^^^^^^

Z-Wave 지문은 ZigBee에 사용된 형태를 기반으로 했지만, 이제는 선호되는 새로운 형태가 있다.
기존 장치 핸드 북에서 원본 지문을 볼 수 있습니다. 레거시 형식에 대한 자세한 내용은 아래를 참조하십시오.

가장 좋은 시작 위치는 스마트 링크에 장치를 추가하고 세부 정보 보기에서 *원형의 설명* 을 찾는 것입니다.
SmartThings개발자 도구에 포함되어 있습니다.

Z-Wave raw description
++++++++++++++++++++++

새로운 형식이 도입된 이후 추가된 Z-Wave디바이스에는 여러 키 값을 가진 원시 설명 문자열이 있습니다.
필드(예:다음) ::

    zw:Ss type:2101 mfr:0086 prod:0102 model:0064 ver:1.04 zwv:4.05 lib:03 cc:5E,86,72,98,84 ccOut:5A sec:59,85,73,71,80,30,31,70,7A role:06 ff:8C07 ui:8C07

일부 장치에는 일부 필드가 없습니다.

**zw:**
    이 필드는 듣기 장치의 경우'L', 잠 자기 장치의 경우'S', 빔 가능 장치의 경우'F'로 시작합니다. 이러한 용어의 의미는 '참고:'Z-wavePrimer<.ave-primer>를 참조하십시오. 기기가 Z-Wave보안 계층을 통해 네트워크에 안전하게 포함되는 경우 대문자 뒤에 소문자가 붙습니다.
**유형:**
    이 필드는 일반 장치와 특정 장치를 결합하는 16비트 16진수 숫자의 Z-Wave장치 클래스입니다.
    클래스 코드. [1]_
**mfr:**
    이 16비트 16진수 숫자는 장치 제조 업체를 나타냅니다. [1]__``mfr`` , ``prod`` 그리고 ``model`` 의 세가지 가치는 인증된 Z-Wave제품을 독특하게 식별한다.
**prod:**
    이 16비트 16진수 숫자는 장치에서 보고하는 제품 유형 ID입니다.
**모델:**
    이 16비트 16진수 숫자는 장치에서 보고하는 제품 ID입니다.
**ver:**
    장치에서 보고하는 애플리케이션 펌웨어 버전입니다.
**zwv:**
    장치에서 사용 중인 Z-Wave프로토콜 스택의 버전입니다.
**lib:**
    장치의 기반이 되는 Z-Wave프로토콜 라이브러리의 유형을 나타냅니다. '01'은 정적 컨트롤러이고,'02'는 원격 컨트롤러이며,'07'은 브리지 컨트롤러이며, 다른 값은 일반적인 비 컨트롤러 장치입니다.
**cc:**
    장치에서 지원하는 Z-Wave명령 클래스 목록입니다(보안 캡슐화 제외). 각 16진수 코드로 표시된 명령 클래스에 대해서는  `Z-Wave명령 참조 <https://graph.api.smartthings.com/ide/doc/zwave-utils.html>`__  를 참조하십시오.
**ccOut:**
    장치가 제어할 수 있는 Z-Wave명령 클래스 목록. 이는 다른 장치로 전송된 명령과 장치가 생성한 보고서를 나타냅니다.
**sec:**
    이러한 명령 클래스는 Z-Wave보안 캡슐화를 통해서만 장치에서 지원됩니다.
**secOut:**
    이러한 명령 클래스는 Z-Wave보안 캡슐화를 통해서만 디바이스에 의해*제어* 됩니다.
**역할:**
    Z-WavePlus역할 유형을 나타냅니다. [1]_
**ff:**
    이는 "폼 팩터"를 나타내며 Z-Wave+Installer아이콘 유형에 해당합니다(구현 이유로 0x8000오프셋이 추가됨). [1]_
**ui:**
    이는 Z-Wave+User아이콘 유형에 해당합니다.

.. [1] See `this document <http://zwavepublic.com/files/sds13740-1-z-wave-plus-device-and-command-class-types-and-defines-specificationpdf>`_ for the values of identifiers defined by the Z-Wave standard.

New Z-Wave fingerprint format
+++++++++++++++++++++++++++++

If you're writing a Device Handler for a specific device, you can base the fingerprint on the manufacturer info.
For example, the fingerprint to match the raw description example above would be:

.. code-block:: groovy

    fingerprint mfr: "0086", prod: "0102", model: "0064"

No other parameters are required. Note that you need to add quotes and commas to the more concise raw description format
to make it valid Groovy code.

Sometimes related products are grouped under the same 'prod' ID. In that case you can use a fingerprint without the
'model' parameter.

If you are writing a general Device Handler that supports all devices of a certain type, you can still base the
fingerprint on command class support.

.. code-block:: groovy

    fingerprint type: "10", cc: "25,32"

That fingerprint would match all devices of the Binary Switch generic device class – i.e. their 'type' starts with
"10" – that support the Binary Switch (0x25) and Meter (0x32) command classes.

The supported parameters are:

**type:**
    Matches if it's equal to or a prefix of the device's 'type' value in the raw description. Aliased as 'deviceId'.
**mfr, prod, model:**
    Matches if 'mfr' matches the raw description and 'prod' and 'model' match as prefixes (if present).
**cc, ccOut:**
    Takes a list of command class codes as a string: comma-separated, uppercase hexadecimal. Matches if all listed
    command class codes are reported as supported or controlled respectively in the device's raw description.
**sec, secOut:**
    The same as the previous parameter, but only matches against command classes the device supports/controls only via
    Z-Wave Security encapsulation.
**ff/ui:**
    Either of these parameters can be used to match against the corresponding fields of the raw description. It is only
    possible to use one of the following in a single fingerprint: 'type', 'deviceId', 'ff', ui'.
**deviceJoinName:**
    Not used for matching. If the fingerprint matches, the device will appear to the user with this name.

When multiple device fingerprints match an added Z-Wave device, they are ranked first by number of 'mfr', 'prod', and
'model' parameters, then by the number of command classes listed, and finally by the length of the 'type', 'ff', or
'ui' parameter. When fingerprints have the same rank, self-published Device Handlers take precedence over the default
production ones.

Legacy Z-Wave fingerprint format
+++++++++++++++++++++++++++++++

Legacy fingerprints include the device class – or ``type`` value (see above) – in the ``deviceId`` parameter and the
command classes it supports in the ``inClusters`` parameter. So the fingerprint:

.. code-block:: groovy

    fingerprint deviceId:"0x1104", inClusters:"0x26, 0x2B, 0x2C, 0x27, 0x73, 0x70, 0x86, 0x72", outClusters: "0x20"

would be formatted in the new style as:

.. code-block:: groovy

    fingerprint type: "1104", cc: "26,2B,2C,27,73,70,86,72", ccOut: "20"

.. _device_handler_fingerprinting_best_practices:

Fingerprinting best practices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add multiple fingerprints
+++++++++++++++++++++++++

A Device Handler can have multiple fingerprints in order to work with multiple versions of a device.
Each fingerprint is independent. If any of them is the highest ranking match, the device will use your device type.

You can distinguish between the different devices that use the handler by adding the 'deviceJoinName' parameter.
For example:

.. code-block:: groovy

    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "0019", manufacturer: "sengled", model: "Z01-CIA19NAE26", deviceJoinName: "Sengled Element touch"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "000A, 0019", manufacturer: "Jasco Products", model: "45852", deviceJoinName: "GE Zigbee Plug-In Dimmer"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "000A, 0019", manufacturer: "Jasco Products", model: "45857", deviceJoinName: "GE Zigbee In-Wall Dimmer"

If an added device supports the inClusters in the first fingerprint but doesn't match all the extra info in any of the
next three, it will join with the name from the handler's definition metadata, in this case "ZigBee Dimmer Power."

Device pairing process
++++++++++++++++++++++

The order of the ``inClusters`` and ``outClusters`` lists is not important to the pairing process.
It is a best practice, however, to list the clusters in ascending order.

The device can have more clusters than the fingerprint specifies, and it will still pair.
If one of the clusters specified in the fingerprint is incorrect, the device will *not* pair.

Overly general fingerprints
+++++++++++++++++++++++++++

If you wish to publish or share a Device Handler, you must make sure that the fingerprints do not capture other devices
that aren't covered by your handler.

If you copied a working fingerprint from a default or template handler, it would be ambiguous which type should match if
yours was published. The easiest way to remedy this is to include manufacturer and model info in all fingerprints.
