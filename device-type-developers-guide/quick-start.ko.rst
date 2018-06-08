.. _device-handler-quickstart:

빠른 시작
===========
Device Handler는 SmartThings 플랫폼의 물리적 장치를 나타냅니다. 이는 실제 장치와 SmartThings 플랫폼 간의 통신을 담당합니다.

또는, 물리적 디바이스를 아직 사용할 수 없는 경우 Device Handler를 가상 디바이스와 연결할 수도 있습니다. 이번 섹션에서는 당신을 사용자 지정 디바이스 핸들러를 생성하고 가상 디바이스로 테스트하는 과정으로 안내합니다.

.. 경고::
    
    계속하기 전에 IDE에서 올바른 위치에 있는지 확인하십시오.
    :참조:`ide_requirements`에 설명된 사전 요구 사항을 따르세요.

    SmartThings 개발에 익숙하지 않은 경우에는 다음 자료부터 살펴 보십시오.:ref:`Getting Started <get-started-overview>`

----

.. _create-device-handler:

Create a new Device Handler
---------------------------

IDE에서 상단 메뉴의 MyDeviceHandlers링크를 클릭합니다. 여기에 모든 기기 사용자가 표시됩니다.

.. figure:: ../img/device-types/ide-device-types.png
페이지 오른쪽 상단에 있는+CreateNewDeviceHandler 버튼을 클릭하여 새 장치 핸들러를 만듭니다.

새 장치 핸들러를 생성하기 위한 양식이 표시됩니다. 양식 상단에 있는 탭에 새 장치 처리기를 만드는 데 필요한 여러 옵션이 표시되어 있습니다.

.. figure:: ../img/device-types/new-device-type-form.png

*From Template* 탭을 선택하세요.

DimmerSwitch템플릿에서 새 장치 핸들러를 생성할 겁니다.
왼쪽 메뉴에서 *Dimmer Switch* 를 클릭합니다.

.. figure:: ../img/device-types/centralite-switch-from-template.png

이제 오른쪽에 DimmerSwitchDeviceHandler코드가 표시됩니다.

잠시 시간을 내어 코드와 코드의 구조를 살펴보겠습니다. 아직 자세한 내용은 걱정하지 마십시오. 장치 핸들러의 구조에 주목하십시오.

.. figure:: ../img/device-types/device-type-anatomy.png

----

다음으로는, 이 코드를 당신 것으로 만들 기 위해 약간의 변화를 줍시다. 
definition 메소드에서, ``name`` 을 "Dimmer Switch" 에서"My Dimmer Switch" 같은 것으로 바꾸고, ``namespace`` 를 당신의 github 계정으로 바꾸고(그냥 비워둘 수도 있습니다), 그리고 ``author`` 를 당신 이름으로 바꾸세요.

editor 아래의 *Create* 버튼을 누르고, 다음 화면의 *Publish* 와 *For Me* 를 클릭하세요.

.. figure:: ../img/device-types/device-handler-publish.png

----

.. _create-virtual-device:

Create a Virtual Device
-----------------------
그런 다음 가상 디바이스를 생성하여 위에서 만든 디바이스 핸들러와 연결합니다.

IDE의 상단 메뉴에서, *My Devices* 를 클릭하세요.

.. figure:: ../img/device-types/ide-my-devices.png

오른쪽 상단의 *+New Device* 를 누르세요.
*Create Device* 페이지로 갈겁니다.

.. figure:: ../img/device-types/create-virtual-device.png

아래 절차들을 밟아 *Create Device* 을 채우세요:

Name
  가상 장치, 가급적이면 "Virtual Dimmer Switch"와 같은 장치의 유형을 나타내는 것입니다.

Label
  해도 안해도 상관없지만,"virtual-dimmer-switch"같은것으로 해놓을 수 있습니다.

Zigbee Id
  공란으로 놔도 돼요.

Device Network Id
  가상 디바이스를 식별하는 고유 ID여야 합니다. 이 ID가 다른 장치 ID와 충돌하지 않는지 확인하십시오. "VIRTDIMMERS01"을 입력하십시오.

Type
  풀 다운 메뉴에는 사용 가능한 Device Handlers가 나열됩니다. 당신만의 Device Handlers는 pulldown 리스트의 밑에 나타납니다. 목록을 아래로 스크롤 하여 위에서 생성한 Device Handlers를 선택합니다.

Version
  옵션은 *Published* 여야 합니다.

Location
  당신의 Hub위치여야 합니다.

Hub
  위의 Location과 관련된 당신의 Hub 이름. 

Group
  선택할 수 없습니다.

*Create.* 를 누르세요. 

당신의 SmartThings 어플내의 "My Home"의 *Things* 화면에서 *virtual-dimmer-switch* 가 즉시 나타나는 것을 볼 수 있습니다.

.. figure:: ../img/device-types/virtual-device-ios.png

----

.. _test-virtual-device:

Test your Device Handler with Virtual Device
--------------------------------------------

방금 만든 Virtual Dimmer를 사용하여 Device Handler를 테스트할 수 있습니다.
SmartThings 어플에서, **virtual-dimmer-switch** 의 *OFF* 타일을 눌러 *ON* 으로 바꿉니다.

.. figure:: ../img/device-types/virtual-dimmer-on.png

다음으로, **virtual-dimmer-switch** 을 눌러 세부 사항을 보고 타일들을 테스트 해봅니다.

.. figure:: ../img/device-types/virtual-dimmer-detail.png

----

.. note::

 시뮬레이터는 Device Handler 가 수신 메시지를 처리하는 방법을 테스트하는 데 유용하고 필요하지만, 가능한 경우 가상 장치를 사용하여 모바일 앱에서 테스트하는 것이 좋습니다.

----

다음 단계 
----------
이제 첫번째 Device Handler를 만들어 가상 디바이스와 함께 설치했으므로, 이 가이드의 나머지 부분을 사용하여 자세히 알아보십시오.
