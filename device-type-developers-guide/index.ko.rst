.. _device_type_dev_guide:

Device Handlers
===============
Device Handlers는 물리적 디바이스의 가상화된 표현입니다.

Device Handlers 작성에 익숙하지 않다면  :문서:`quick-start` 를 읽고 시작합시다.

그 후에 Device Handler와 SmartThings아키텍처의 어디에 맞는지에 대한 광범위한 설명을 보려면 :문서:`overview` 를 읽어 보십시오.

나머지 부분에서는 주로 허브 연결 장치(ZigBee 혹은 Z-Wave)를 대상으로 한 Device Handlers의 다양한 구성 요소에 대해 설명합니다(일반적인 Device Handler의 원칙 및 패턴이 다른 장치에도 적용됨).


.. note::
이 가이드에서는 허브 연결 Device Handlers에 대해 설명합니다. LAN 및 클라우드로 연결된 Device Handlers에 대한 내용은 `this guide <../cloud-and-lan-connected-device-types-developers-guide/index.html>`__ 를 참조하십시오.
 
목차:

.. toctree::
   :maxdepth: 1

   quick-start
   overview
   simulator-metadata
   definition-metadata
   tiles-metadata
   device-preferences
   parse
   z-wave-primer
   building-z-wave-device-handlers
   z-wave-example
   zigbee-primer
   building-zigbee-device-handlers
   zigbee-example
   other-available-apis
   device-integration
