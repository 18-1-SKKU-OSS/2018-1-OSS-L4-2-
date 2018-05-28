.. _automatic_LAN_device_discovery:

==============
자동 LAN장치 검색
==============

자동 LAN장치 검색은 LAN연결 장치를 검색하는데의 복잡성을 최소화합니다.

일반적으로 SmartThings 플랫폼은 특정 장치에 대한 서비스 관리자 SmartApp이 있을때만 클라우드 및 LAN연결 장치를 검색합니다.
즉, Wemo 모션 센서 및 Bose 스피커와 같은 여러 LAN장치를 통합하려면 여러개의 서비스 관리자 SmartApps (각 LAN연결 장치를 위한 서비스 관리자 SmartAPP)가 필요합니다.
이에 반해, 이 플랫폼은 ZigBee 또는 Z-Wave 장치에 대한 서비스 관리자 SmartApp 요구 사항이 없습니다.

자동 신규 LAN장치 검색 기능은 일부 LAN연결 장치에 대한 서비스 관리자 SmartApp 요구 사항을 제거하여 훨씬 부드럽고 빠른 LAN연결 장치 검색을 가능하게 합니다.
다음을 참고하십시오 :ref:`autodiscoverlan-devices-list`.

개발자에 미치는 영향
===============

참고 자료에 대하여 :ref:`autodiscoverlan-devices-list' 서비스 관리자 SmartApp 또는 장치 핸들러에 대해 수정을 가한 경우 LAN장치 통합에 영향을 미칩니다.
아래 표를 참조하십시오.

============ ====================== ====
맞춤 장치 핸들러 맞춤 서비스 관리자 SmartApp 영향
============ ====================== ====
네           아니오                   사용자 지정 LAN장치 핸들러는 SmartThings버전으로 덮어씁니다.
네           아니오                   영향 없음
============ ====================== ====

.. _autodiscoverlan-devices-list:

지원되는 LAN연결 장치
^^^^^^^^^^^^^^^^^

현재 자동 LAN장치 검색을 통해 제한된 수의 LAN연결 장치를 검색할 수 있습니다.
다음을 참고하십시오 `How to connect Wi-Fi devices <https://support.smartthings.com/hc/articles/115001164026>`_.
