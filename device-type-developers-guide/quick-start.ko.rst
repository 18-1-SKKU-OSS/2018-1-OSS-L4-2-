.. _device-handler-quickstart:

빠른 시작
===========
Device Handler는 SmartThings 플랫폼의 물리적 장치를 나타냅니다. 이는 실제 장치와 SmartThings 플랫폼 간의 통신을 담당합니다.

또는, 물리적 디바이스를 아직 사용할 수 없는 경우 Device Handler를 가상 디바이스와 연결할 수도 있습니다. 이번 섹션에서는 당신을 사용자 지정 디바이스 핸들러를 생성하고 가상 디바이스로 테스트하는 과정으로 안내합니다.

.. 경고::
    
    계속하기 전에 IDE에서 올바른 위치에 있는지 확인하십시오.
    :ref:`ide_requirements`에 설명된 사전 요구 사항을 따르세요.

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

From the top menu of the IDE, click on the *My Devices*.

.. figure:: ../img/device-types/ide-my-devices.png

Click on *+New Device* on the top-right.
This will take you to *Create Device* page.

.. figure:: ../img/device-types/create-virtual-device.png

Follow below steps to fill the above *Create Device* form:

Name
  Your Virtual Device, preferably something that's indicative of the type of the device, such as "Virtual Dimmer Switch".

Label
  Optional, but you can have something like "virtual-dimmer-switch".

Zigbee Id
  Can be blank.

Device Network Id
  Should be a unique ID that identifies your Virtual Device. Make sure this ID doesn't conflict with any other device Ids. Put in "VIRTDIMMERS01".

Type
  Pulldown menu lists available Device Handlers.
  Note that all your custom Device Handlers are listed at the bottom of the pulldown list.
  Scroll down the list and select the customer Device Handler that you created above.

Version
  Option should be *Published*.

Location
  Must be your Hub Location.

Hub
  Your Hub name associated with the above Location.

Group
  Not selectable.

Click *Create.*

You will see *virtual-dimmer-switch* device appear instantly in your SmartThings mobile app, in the *Things* screen of the "My Home" view.

.. figure:: ../img/device-types/virtual-device-ios.png

----

.. _test-virtual-device:

Test your Device Handler with Virtual Device
--------------------------------------------

With the Virtual Dimmer you just created you can test your Device Handler.
From your SmartThings mobile app, tap on the *OFF* tile of **virtual-dimmer-switch** to turn it *ON*.

.. figure:: ../img/device-types/virtual-dimmer-on.png

Next, tap on the **virtual-dimmer-switch** to open the detail view and test the tiles.

.. figure:: ../img/device-types/virtual-dimmer-detail.png

----

.. note::

  While the Simulator is useful and necessary for testing how the Device Handler handles incoming messages, we recommended that you test on the mobile app with Virtual Devices wherever possible.

----

Next steps
----------

Now that you have created and installed your first Device Handler with a Virtual Device, use the rest of this guide to learn more.
