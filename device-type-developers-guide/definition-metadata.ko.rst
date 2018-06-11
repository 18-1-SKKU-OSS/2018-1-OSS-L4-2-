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

특정 장치에 대한 장치 핸들러를 작성하는 경우 제조 업체 정보를 기반으로 지문을 작성할 수 있습니다.
예를 들어 위의 원시 설명 예제와 일치하는 지문은 다음과 같습니다.

.. code-block:: groovy

    fingerprint mfr: "0086", prod: "0102", model: "0064"

다른 매개 변수는 필요하지 않습니다. 인용 부호와 쉼표를 더 간결한 원시 설명 형식에 추가하여 유효한 Groov코드로 만들어야 합니다.

때로는 관련 제품이 동일한 'prod' ID로 그룹화되기도 합니다. 이 경우'mode'매개 변수 없이 지문을 사용할 수 있습니다.

특정 유형의 모든 장치를 지원하는 일반 장치 핸들러를 작성하는 경우에도 명령 클래스 지원을 기반으로 fingerprint를 사용할 수 있습니다.

.. code-block:: groovy

    fingerprint type: "10", cc: "25,32"

이 지문은 BinarySwitch일반 장치 클래스의 모든 장치와 일치합니다. 즉,'유형'이
"10"-BinarySwitch(0x25)및 Meter(0x32)명령 클래스를 지원합니다.

지원되는 매개 변수는 다음과 같습니다.

**유형:**
    초기 정보에서 장치의 '유형'값 접두사 또는 같은지 여부를 일치시킵니다. 별칭이 'deviceId'로 지정되었습니다.
**mfr, 주입, 모형:**
    '.r'가 초기 설명과 일치하면 일치하고 접두사( 있는 경우)로 'spot'및'Model'이 일치하면 일치합니다.
**참조인, cc:**
    명령 클래스 코드 목록을 쉼표로 구분된 대문자 16진수 문자열로 표시합니다. 모두 나열된 경우 일치
    명령 클래스 코드는 장치의 초기 설명에서 각각 지원되거나 제어되는 것으로 보고됩니다.
**초, SecOut:**
    이전 매개 변수와 동일하지만 디바이스 지원 via 제어는
    Z-Wave보안 캡슐화.
**ff /i:**
    이러한 매개 변수 중 하나를 사용하여 원시 설명의 해당 필드와 일치시킬 수 있습니다. 그것은 단지
    하나의 지문에 'type','deviceId','는 ', 음'중 하나를 사용할 수 있습니다.
**장치 이름:**
    일치시키는 데 사용되지 않습니다. 지문이 일치하는 경우 이 이름을 가진 사용자에게 장치가 나타납니다.

여러개의 장치 지문이 추가된 Z-Wave장치와 일치할 경우, 해당 지문은 'Z-war','stroke'및
'모델'매개 변수 다음에 나열된 명령 클래스 수를 기준으로 마지막으로 'type','는 '또는
'UI'매개 변수. 지문의 순위가 동일한 경우 자체 게시 장치 핸드 북이 기본 값보다 우선합니다.
생산용 제품

기존 Z-Wave지문 형식
+++++++++++++++++++++++++++++++

기존 지문에는 다음과 같은 장치 클래스(또는 ``type`` 값(위 참조)가 포함됩니다.
``deviceId`` 매개 변수 및
명령 클래스는 ``inClusters`` 매개 변수에서 지원합니다. 그래서 fingerprint는 :

.. code-block:: groovy

  지문 장치 ID:"0x1104", 클러스터 내에서 "0x26,0x2B, 0x2C, 0x27,0x73,0x86,0x72", 0x20"

새로운 형식으로 다음과 같이 포맷됩니다.

.. code-block:: groovy

  지문 유형:"1104", 참조:"26,2B, 2C, 27,73,70,86,72"

.. _device_handler_fingerprinting_best_practices:

핑거 프린트 모범 사례
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

여러개의 지문 추가
+++++++++++++++++++++++++

장치 핸들러는 장치의 여러 버전에서 작업하기 위해 여러개의 지문을 가질 수 있습니다.
각 지문은 독립적입니다. 가장 높은 순위 일치 항목이 있으면 장치 유형이 사용됩니다.

'deviceAcinName'매개 변수를 추가하여 처리기를 사용하는 여러 디바이스를 구분할 수 있습니다.
예:

.. code-block:: groovy

    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "0019", manufacturer: "sengled", model: "Z01-CIA19NAE26", deviceJoinName: "Sengled Element touch"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "000A, 0019", manufacturer: "Jasco Products", model: "45852", deviceJoinName: "GE Zigbee Plug-In Dimmer"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "000A, 0019", manufacturer: "Jasco Products", model: "45857", deviceJoinName: "GE Zigbee In-Wall Dimmer"

    추가된 장치가 첫번째 지문에서 클러스터를 지원하지만 모든 추가 정보와 일치하지 않는 경우
    다음 3단계로, 그것은 핸들러의 정의 메타 데이터의 이름과 결합할 것이다, 이 경우에 "ZigBee Dimmer Power."

장치 페어링 과정
++++++++++++++++++++++

 ``inClusters`` 그리고 ``outClusters`` 목록의 순서는 쌍 구성 과정에서 중요하지 않습니다.
그러나 클러스터를 오름차순으로 나열하는 것이 좋습니다.

장치에는 지문이 지정하는 것보다 많은 클러스터가 있을 수 있으며, 여전히 클러스터는 쌍으로 구성됩니다.
지문에 지정된 클러스터 중 하나가 잘못된 경우 디바이스는 페어링되지 *않을* 겁니다.

지나치게 일반적인 지문
+++++++++++++++++++++++++++

장치 핸들러를 게시하거나 공유하려면 지문이 다른 장치를 캡처하지 않는지 확인해야 합니다.
해당 사항은 귀하의 담당자가 다루지 않습니다.

기본 또는 템플릿 처리기에서 작업 지문을 복사한 경우
당신의 것이 출판되었습니다. 이 문제를 해결하는 가장 쉬운 방법은 제조 업체 및 모델 정보를 모든 지문에 포함시키는 것입니다.
