ZigBee Device Handlers 만들기
===============================

.. note::

        만약 당신이 새로운 ZigBee 스위치나 전구를 SmartThings와 통합한다면, 아래의 :참조:`zigbee_device_form` 부분을 참조하여 어떻게 이러한 장치들을 코드 작성 없이 통합할 수 있는지 보세요.

----

명령어들
--------

SmartThings는 ZigBee 함께 일하는 것을 더 쉽게 하기 위해 라이브러리를 제공한다.
모든 Device Handler 에는 이 라이브러리에 주입된 이름이 ``zigbee`` 로 표시되어 있습니다.

이 라이브러리는 아래 예에서 사용됩니다.
자세한 내용은 :참조:`zigbee_ref` 를 참조하십시오.

ZigBee  기기와 SmartThings를 통합하는 데 사용할 수 있는 네가지 일반적인 ZigBee 명령이 있습니다.

읽기
^^^^

읽기는  다음과 같이 형식화된 디바이스의 최근 상태를 가져옵니다:

.. code-block:: groovy

    def refresh() {
        zigbee.readAttribute(0x0B04, 0x050B)
    }

이 예에서 장치 유형("CentraLiteSwitch"장치)
유형)에서 "refresh" 함수를 호출하고 있습니다.
현재 상태(유효 전력 인출)를 판독하기 위해 ``readAttribute()`` 메소드를 통해 ZigBee  읽기 속성 요청을 보내고 있다.
여기서 살펴볼 클러스터는 Electrical Measurement (0xB04), 특히 Active Power Attribute(0x50B)입니다.    

+-------------------------------+-----------------------------+
| 요소                          | 설명                        |
+===============================+=============================+
|0x0B04                         | Cluster                     |
+-------------------------------+-----------------------------+
|0x050B                         | Attribute                   |
+-------------------------------+-----------------------------+

쓰기
^^^^^

쓰기는 ZigBee 장치의 특성을 다음과 같이 세팅하고, 다음과 같이 형시이 정해져 있습니다:

.. code-block:: groovy

    def configure() {
            zigbee.writeAttribute(8, 0x10, 0x21, 0x0014)
        }

이 예에서는("ZimbeeDimmer" Device Handler에서)조명이 완전히 켜지고 꺼지는 데 걸리는 시간을 설정하기 위해 특성에 쓰고 있습니다.
여기서는 Level Control Cluster (8)를 사용하여 전환 시간 및 해제(0x10)을 정의하는 속성에 씁니다.
사용 중인 값은 서명되지 않은 16비트 정수(0x21)로 포맷되며 페이 로드는 1/10/초 단위입니다.
이 경우 페이 로드({0014})는 2초로 변환됩니다.
페이 로드를 분해하면 0x0014의 16진수 값이 10진수 값인 20이 됩니다. 20*1/10초는 2초와 같습니다.

각 속성에는 특정 데이터 유형이 있습니다.
이 데이터 유형에 해당하는 값은 `ZigBee Cluster Library <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__ 표 2.16에서 확인할 수 있다.        


.. note::
위 예의{0014}페이 로드는 16진수 문자열입니다. 페이 로드의 길이는 데이터 유형의 두배여야 합니다. 예를 들어, 데이터 유형이 16비트인 경우 페이 로드는 416진수여야 합니다.

+-------------------------------+-----------------------------+
| 요소                          | 설명                        |
+===============================+=============================+
|8                              |Cluster                      |
+-------------------------------+-----------------------------+
|0x10                           |Attribute Set                |
+-------------------------------+-----------------------------+
|0x21                           |Data Type                    |
+-------------------------------+-----------------------------+
|0x0014                         |Payload                      |
+-------------------------------+-----------------------------+

명령어
^^^^^^^

Commands는 ZigBee 장치로 명령을 보내고, 다릉과 같은 형식을 따릅니다:

.. code-block:: groovy

    def on() {
        zigbee.command(0x0006, 0x01)
    }

이 예에서는("ZipseeDimmer"장치 유형에서)장치를 켜기 위해 Zigbee명령을 보냅니다.
On/OffCluster(6)를 사용하여(1)을 켜는 명령을 전송합니다.
이 명령에는 페이 로드가 없으므로 전달된 매개 변수에서 제외합니다.

+-------------------------------+-----------------------------+
| Component                     | Description                 |
+===============================+=============================+
|0x0006                         |Cluster                      |
+-------------------------------+-----------------------------+
|0x01                           |Command                      |
+-------------------------------+-----------------------------+

Configure
^^^^^^^^^

Configure reporting instructs a device to notify us when an attribute changes and is formatted like this:

.. code-block:: groovy

    def configure() {
        configureReporting(0x0006, 0x0000, 0x10, 0, 600, null)
    }

In this example (using the "CentraLite Switch" Device Handler), the bind command is sent to the device using its Network ID which can be determined using ``0x${device.deviceNetworkId}``.
Then using source and destination endpoints for the device and Hub (1 1), we bind to the On/Off Clusters (6) to get Events from the device.
The last part of the message contains the Hub's ZigBee id which is set as the Location for the device to send callback messages to.
Note that not at all devices support binding for Events.

+-------------------------------+-----------------------------+
| Component                     | Description                 |
+===============================+=============================+
|0x0006                         |Cluster                      |
+-------------------------------+-----------------------------+
|0x0000                         |Attribute ID                 |
+-------------------------------+-----------------------------+
|0x10                           |Boolean data type            |
+-------------------------------+-----------------------------+
|0                              |Minimum report time          |
+-------------------------------+-----------------------------+
|600                            |Maximum report time          |
+-------------------------------+-----------------------------+
|null                           |Reportable change (discrete) |
+-------------------------------+-----------------------------+

----

ZigBee utilities
----------------

In order to work with ZigBee you will need to use the ZigBee Cluster Library extensively to look up the proper values to send back and forth to your device.
You can download this document `here <http://www.zigbee.org/download/standards-zigbee-cluster-library/>`__.

There is also a ZigBee utility class covered in the :ref:`zigbee_ref`.

----

Best practices
--------------

- The use of 'raw ...' commands is deprecated. Instead use the documented methods on the ZigBee library. If you need to do something that requires the use of a 'raw' command let us know and we will look at adding it to the ZigBee library.
- Do not use ``sendEvent()`` in command methods. Sending Events should be handled in the ``parse`` method.

----

.. _zigbee_device_form:

Using the ZigBee Device Form
----------------------------

To integrate a new ZigBee switch or bulb with SmartThings, you can use the *From ZigBee Device Form*.

.. image:: ../img/device-types/zigbee-form.png

What it does
^^^^^^^^^^^^

By entering the ZigBee information for the device in the form, the appropriate existing Device Handler will be updated with the device's fingerprint.

Use it if
^^^^^^^^^

- You are the device manufacturer, or otherwise have access to the required ZigBee device information requested on the form.
- The device is best described as one of the following:

    - ZigBee Switch
    - ZigBee Switch with Power
    - ZigBee Dimmer/Bulb
    - ZigBee Dimmer/Bulb with Power
    - ZigBee Color Temperature Bulb

How to use
^^^^^^^^^^

Simply fill out the required fields in the form with the information for the device, and click *Create.*

You will then see the updated Device Handler code in the IDE editor.
You can then test that your device pairs with SmartThings and functions as expected, and then make an update as a Publication Request.
