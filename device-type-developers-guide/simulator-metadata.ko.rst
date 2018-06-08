시뮬레이터
=========

IDE시뮬레이터를 사용하면 물리적 장치 없이도 장치의 동작을 모델링 할 수 있습니다.

----

개요
--------

IDE의 오른쪽에 Device Handler를 설치한 후 시뮬레이터가 나타납니다. 아래 이미지는 "Z-Wave Switch" Device Handler를 설치한 후 표시되는 시뮬레이터입니다 (장치 템플릿 찾아 보기 메뉴를 통해 사용 가능).

어서 시도 해 보세요. IDE에 Device Handler를 설치하고 가상 스위치를 선택합니다. 이 과정을 읽으면서 시뮬레이터 메타 데이터의 일부를 수정하고 어떤 일이 일어나는지 확인하십시오.

.. figure:: ../img/device-types/simulator.png

시뮬레이터 메타 데이터의 목적은 물리적 장치의 동작을 모델링 하는 것입니다. 시뮬레이터를 사용하여 Device Handler로 메시지 및 명령 전송을 테스트할 수 있습니다.

Device Handler에서 정의할 시뮬레이터 선언에는 "상태"와 "응답"의 두가지 유형이 있습니다.

----

상태
------

"상태"선언은 사용자가 장치를 물리적으로 작동하게 만드는 작업을 지정합니다. 예를 들어 Z-Wave스위치의 경우는 다음과 같습니다.

.. code-block:: groovy

    status "on":  "command: 2003, payload: FF"
    status "off": "command: 2003, payload: 00"

``status()`` 는 map을 인자로 받습니다.
키 (위 예시에서의 "on") 은 그저 액션을 위한 이름일 뿐입니다.
값("command: 2003, payload: FF") 는 물리적 디바이스에서 작업이 수행될 때 장치가 Device Handler의 ``parse(message)`` 메소드로 전송할 메세지 입니다. 
시뮬레이터에서, 각 상태 키("on" or "off" in the example above)는 시뮬레이터에서 가능한 메세지들 입니다.

----

응답
-----

"응답"선언은 물리적 장치가 허브로부터 특정 메시지를 수신할 때 장치 핸들러에 보낼 응답을 지정합니다. 예를 들어 Z-Wave스위치의 경우 다음을 지정합니다.

.. code-block:: groovy

    reply "2001FF,delay 100,2502": "command: 2503, payload: FF"
    reply "200100,delay 100,2502": "command: 2503, payload: 00"

``status()`` 처럼, ``reply()`` 도 인자로 map을 받습니다.
키는 장치로 전송된 다듬어지지 않은 명령으로(즉, Device Handler 의 명령 방법에서 반환된 명령)쉼표로 구분된 목록입니다. 예를 들어 위의 방법을 전송하는 Z-Wave스위치 명령은 다음과 같습니다:

.. code-block:: groovy

    def on() {
        delayBetween([
            zwave.basicV1.basicSet(value: 0xFF).format(),
            zwave.switchBinaryV1.switchBinaryGet().format()
        ])
    }


    def off() {
        delayBetween([
            zwave.basicV1.basicSet(value: 0x00).format(),
            zwave.switchBinaryV1.switchBinaryGet().format()
        ])
    }

이러한 메서드는 응답 선언의 첫번째 인수에 있는 값을 반환합니다. 응답 선언의 두번째 인수는 상태 선언과 동일한 방식으로 작동하며, 이들 인수에서 구문 분석 방법으로 보낸 메시지를 정의합니다. 하지만 이 경우에는 물리적인 작동이 아니라 명령에 따라 움직입니다.

----

요약
-------
이 선언의 목적은 물리적 디바이스에 연결하지 않고 IDE시뮬레이터에서 가상 디바이스가 작동할 수 있도록 하는 것입니다.
``status()`` 메서드를 사용하면 물리적 작동을 시뮬레이션할 수 있으며, ``reply()`` 메서드를 사용하면 허브의 명령에 대응하여 장치로 메시지를 보내는 것을 시뮬레이션할 수 있습니다.
