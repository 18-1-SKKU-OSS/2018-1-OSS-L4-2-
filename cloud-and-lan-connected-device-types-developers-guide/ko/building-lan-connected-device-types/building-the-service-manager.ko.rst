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

----

.. _lan_device_discovery:

탐색
---

----

.. _lan_device_discovery:

Discovery
---------

SSDP(SimpleServiceDiscoveryProtocol)는 네트워크에서 장치를 찾는 데 사용되는 기본 프로토콜입니다.

UPnP(Universal Plug and Play)의 backbone 역할을 하므로 새 네트워크 장치를 시스템에 쉽게 연결할 수 있습니다.

자세한 내용은 다음을 참조하십시오 `UPnP Device Architecture 1.1 <http://upnp.org/specs/arch/UPnP-arch-DeviceArchitecture-v1.1.pdf>`__ .

새 장치를 검색하려면 먼저 올바른 **search target**을(를)사용하여 Location Event에 가입해야 합니다.

아래 예에서 **urn:schemas-upnp-org:device:ZonePlayer:1**는 Sonos를 검색하기 위한 것이지만, 검색 대상은 제조 업체마다 다릅니다.

UPnP의 경우 이 정보는 장치 설명서에 게시되어 있어야 하지만 제조 업체에 직접 문의해야 할 수도 있습니다.

Event 구독은 다음과 같습니다:

.. code-block:: groovy

    subscribe(location, "ssdpTerm.urn:schemas-upnp-org:device:ZonePlayer:1", ssdpHandler)

이는 언제든지 검색 타겟을 urn:schemas-upnp-org:device:ZonePlayer:1 (e.g. Sonos)로 가지는 SSDP **search response**가 현재 위치에서 Hub으로 부터 받아지면 ssdpHandler 메소드가 실행됩니다.

그런 다음 원하는 검색 대상에 대한 적절한 검색 명령을 전송해야 합니다:

.. code-block:: groovy

    void ssdpDiscover() {
        sendHubCommand(new physicalgraph.device.HubAction("lan discovery urn:schemas-upnp-org:device:ZonePlayer:1", physicalgraph.device.Protocol.LAN))
    }

.. note:: HubAction은 SmartThings플랫폼에서 제공하는 클래스입니다.

    The class ``physicalgraph.device.HubAction`` encapsulates request information
    for communicating with the device.

    When you create an instance of a ``HubAction``, you provide details about the
    request, such as the request method, headers, and path. By itself, ``HubAction`` is little more than a wrapper for these request details.
    In this case, it's a thin wrapper around discovery information.

In the above HubAction example, the main message to be sent through the Hub is:

.. code-block:: groovy

    lan discovery urn:schemas-upnp-org:device:ZonePlayer:1

This is converted by our device connectivity layer into an M-SEARCH multicast request that is sent to the LAN via the Hub, and
should look something like the following:

.. code-block:: bash

    M-SEARCH * HTTP/1.1
    HOST: 239.255.255.250:1900
    MAN: "ssdp:discover"
    MX: 4
    ST: urn:schemas-upnp-org:device:ZonePlayer:1

After the end device receives the multicast M-SEARCH, it is supposed to issue a unicast **search response**, delayed by a random number of seconds between 0 and MX (4 in this case).
The search response sent from the device back to the Hub should look something like this:

.. code-block:: bash

    HTTP/1.1 200 OK
    CACHE-CONTROL: max-age=100
    EXT:
    LOCATION: http://10.0.1.14:80/xml/device_description.xml
    SERVER: FreeRTOS/6.0.5, UPnP/1.0, IpBridge/0.1
    ST: urn:schemas-upnp-org:device:ZonePlayer:1
    USN: uuid:RINCON_000E58F0FFFFFF400::urn:schemas-upnp-org:device:ZonePlayer:1

This will get routed back to the cloud where it will be converted into an Event that will fire the ssdpHandler method with the following description:

.. code-block:: bash

    devicetype:04, mac:000E58F0FFFF, networkAddress:0A00010E, deviceAddress:0578, stringCount:04, ssdpPath:/xml/device_description.xml, ssdpUSN:uuid:RINCON_000E58F0FFFFFF400::urn:schemas-upnp-org:device:ZonePlayer:1, ssdpTerm:urn:schemas-upnp-org:device:ZonePlayer:1, ssdpNTS:

The ssdpHandler method should record the data from the search response, in preparation for verification.

.. code-block:: groovy

    def ssdpHandler(evt) {
        def description = evt.description
        def hub = evt?.hubId

        def parsedEvent = parseEventMessage(description)
        parsedEvent << ["hub":hub]

        def devices = getDevices()
        String ssdpUSN = parsedEvent.ssdpUSN.toString()
        if (!devices."${ssdpUSN}") {
            devices << ["${ssdpUSN}": parsedEvent]
        }
    }
