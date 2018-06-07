.. _building_servicemanager:

==============
서비스 관리자 구축
==============

서비스 관리자의 책임은 다음과 같습니다:

- **Discovery** - SSDP를 통해 LAN으로 연결된 장치들을 탐색한다 (mDNS/DNS-SD 나 Bonjour는 아직 지원 되지 않습니다)
- **Verification** - UPnP장치 설명을 성공적으로 가져와서 검색된 각 장치 확인
- **Inclusion** - 서비스 관리자의 하위 항목으로 디바이스 추가
- **Health** - IP주소 및 포트 변경 사항을 추적하고 필요에 따라 하위 장치로 전달합니다.

Service Manager 구현의 주요 부분을 살펴보겠습니다.

이 문서 전체에서 참조된 예제 코드는 아래 SmartApp및 DeviceType에서 파생되었습니다:

:download:`Generic UPnP Service Manager <src/generic-upnp-service-manager.groovy>`
:download:`Generic UPnP Device <src/generic-upnp-device.groovy>`

위에서 언급한 Generic UPnP장치는 불완전합니다.

완성 된 가이드는 다음을 참고하세요: `Building the Device Type <building-the-device-type.html>`_.
