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

The "reply" declarations specify responses that the physical device will send to the Device Handler when it receives a certain message from the Hub.
For a Z-Wave switch, for example, we specify:

.. code-block:: groovy

    reply "2001FF,delay 100,2502": "command: 2503, payload: FF"
    reply "200100,delay 100,2502": "command: 2503, payload: 00"

Just like ``status()``, ``reply()`` accepts a map as a parameter.
The key is a comma-separate list of the raw commands sent to the device, i.e. what's returned from the Device Handler's command methods.
For example, the Z-Wave switch commands that send the above methods are:

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

Those methods will return the values in the first arguments of the reply declarations.
The second argument in the reply declarations works the same way as the status declarations - they define messages sent to the parse method.
But in this case it's in response to commands, not physical actuations.

----

Summary
-------

The purpose of these declarations is to allow a virtual device to function in the IDE Simulator, without being attached to a physical device.
The ``status()`` method allows us to simulate physical actuation, while the ``reply()`` method allows us to simulate sending messages to the device in response to a command from the Hub.
