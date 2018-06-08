.. _building_devicetype:

장치 유형 구성
===========

LAN에 연결된 장치의 장치 핸들러는 일반적으로 다른 장치 핸들러와 동일합니다. 하는 방법
그것은 그것의 장치로부터 메시지를 보내고 받는 것을 처리합니다 조금 다릅니다. LAN에 연결된 장치 핸들러 예제를 살펴보겠습니다.


HubAction을 통해 외향적인 HTTP call 만들기
-------------------------------------

사용 중인 장치 유형에 따라 다음으로 요청을 보냅니다.
REST또는 UPnP를 통해 허브를 통해 장치에 액세스 할 수 있습니다.
이 작업은 제공된 ``HubAction``클래스에서 제공하는 SmartThings를 사용하여 수행할 수 있습니다.

----

개요
----

클래스 ``physicalgraph.device.HubAction``은 장치와 통신하기 위한 요청 정보를 캡슐화합니다.

요청 방법, 헤더 및 경로와 같은 요청에 대한 세부 정보를 제공하는 인스턴스는 ``HubAction``입니다.
그 자체로 ``HubAction``은 이러한 요청 세부 사항에 대한 wrapper에 지나지 않는다.

이는 명령어가 유용하게 쓰일 수 있도록 하는 명령어에서 ``HubAction``의 인스턴스가 반환될 때이다.

장치 핸들러의 명령어 방법이 ``HubAction``인스턴스를 반환할 때 SmartThings플랫폼은 실제로 요청을 수행하기 위해 해당 요청 정보를 사용합니다. 그런 다음 응답 데이터를 사용하여 장치 취급자의 ``parse`` 방법을 호출합니다.

여기에 중요한 사항 - *명령 방법을 사용할 때 HubAction 인스턴스가 리턴되지 않으면 더 이상 요청이 없습니다.* 시스템 메모리를 할당하는 개체일 뿐입니다. 별로 유용하지 않습니다.

따라서 플랫폼에서 요청을 할 수 있도록 명령 방법에서 ``HubAction``인스턴스를 반환해야 합니다.

----

HubAction 개체 만들기
---------------------------

HubAction 개체를 만들려면 다음과 같이 요청 정보를 정의하는 생성자에게 매개 변수 맵을 전달하면 됩니다:

.. code-block:: groovy

    def result = new physicalgraph.device.HubAction(
        method: "GET",
        path: "/somepath",
        headers: [
            HOST: "device IP address"
        ],
        query: [param1: "value1", param2: "value2"]
    )

제공되어 있는 옵션에 대한 간략한 설명은 다음과 같습니다:

*method*
    요청에 사용할 HTTP 메소드.

*path*
    요청을 보낼 경로. 요청에 URL매개 변수를 직접 추가하거나 ``query``옵션을 사용할 수 있습니다.

*headers*
    HTTP헤더의 맵과 이 요청에 대한 값입니다.
    여기서 장치의 IP주소를 호스트로 제공합니다.

*query*
    이 요청에 사용할 쿼리 매개 변수의 맵입니다.
    이 옵션을 사용하는 대신 원하는 경우 경로에서 직접 URL매개 변수를 사용할 수 있습니다.

----

응답 파싱
--------

``HubAction``을 사용하여 장치에 요청하면 다른 장치 메시지와 마찬가지로 모든 응답이 장치 취급자의 ``parse``방법으로 전달됩니다.

``parseLanMessage`` 메소드를 사용하여 수신 메시지를 구문 분석할 수 있습니다.

``parseLanMessage`` 예시:

.. code-block:: groovy

    def parse(description) {
        ...
        def msg = parseLanMessage(description)

        def headersAsString = msg.header // => headers as a string
        def headerMap = msg.headers      // => headers as a Map
        def body = msg.body              // => request body as a string
        def status = msg.status          // => http status code of the response
        def json = msg.json              // => any JSON included in response body, as a data structure of lists and maps
        def xml = msg.xml                // => any XML included in response body, as a document tree structure
        def data = msg.data              // => either JSON or XML in response body (whichever is specified by content-type header in response)

        ...
    }

JSON 이나 XML 응답 포맷에 대한 더 자세한 정보는
Groovy `JsonSlurper <http://docs.groovy-lang.org/latest/html/gapi/groovy/json/JsonSlurper.html>`__ and `XmlSlurper <http://docs.groovy-lang.org/latest/html/api/groovy/util/XmlSlurper.html>`__ 문서를 참고하세요.

----

주소 받아오기
----------

HubAction을 수행하려면 장치의 IP주소와 허브가 필요합니다.

디바이스 IP및 포트가 저장되는 방법은 디바이스 유형에 따라 다릅니다.

현재 이 정보를 쉽게 얻을 수 있는 공용 API가 없으므로, 있을 때까지 장치 유형 처리기에서 이 작업을 처리해야 합니다.

다음과 같은 도우미 메서드를 사용하여 이 정보를 얻는 것이 좋습니다:

.. code-block:: groovy

    // gets the address of the Hub
    private getCallBackAddress() {
        return device.hub.getDataValue("localIP") + ":" + device.hub.getDataValue("localSrvPortTCP")
    }

    // gets the address of the device
    private getHostAddress() {
        def ip = getDataValue("ip")
        def port = getDataValue("port")

        if (!ip || !port) {
            def parts = device.deviceNetworkId.split(":")
            if (parts.length == 2) {
                ip = parts[0]
                port = parts[1]
            } else {
                log.warn "Can't figure out ip and port for device: ${device.id}"
            }
        }

        log.debug "Using IP: $ip and port: $port for device: ${device.id}"
        return convertHexToIP(ip) + ":" + convertHexToInt(port)
    }

    private Integer convertHexToInt(hex) {
        return Integer.parseInt(hex,16)
    }

    private String convertHexToIP(hex) {
        return [convertHexToInt(hex[0..1]),convertHexToInt(hex[2..3]),convertHexToInt(hex[4..5]),convertHexToInt(hex[6..7])].join(".")
    }

이 문서의 나머지 예제에서는 이러한 도우미 방법을 사용합니다.

----

Wake on LAN (WOL)
-----------------

``HubAction``는 `WOL <https://en.wikipedia.org/wiki/Wake-on-LAN>`__ 요청을 생성할때 이용될 수 있습니다.

다음은 예시입니다:

.. code-block:: groovy

    def myWOLCommand() {
        def result = new physicalgraph.device.HubAction (
            "wake on lan <your mac address w/o ':'>",
            physicalgraph.device.Protocol.LAN,
            null,
            [secureCode: "111122223333"]
        )
        return result
    }


``HubAction`` 에 대한 첫번째 인수는 HubAction 클래스에 WOL요청이 될 것이라고 알려 줍니다.

인수는 "wake on lan <mac address>" 형식이어야 합니다. 여기서 MAC주소는 ':'구분 문자가 없는 주소입니다.

예를 들어 NIC의 MAC주소가``01:23:45:67:89:ab``인 경우 ``HubAction``에 대한 첫번째 매개 변수는 ``"wake on lan 0123456789ab"``입니다.

두번째 파라미터는 단순히 LAN요청을 지정하기만 하면 됩니다.

WOL유형 요청 시에는 항상 이러한 상황이 발생합니다. 그래서 그 가치는 항상``physicalgraph.device.Protocol.LAN`` 입니다.

세번째 매개 변수는 Device Network ID, 또는 dni입니다. WOL요청의 경우 이 매개 변수는 ``null``이어야 합니다.

마지막 매개 변수는 요청에 대한 옵션을 나타내는 맵입니다.

WOL요청의 경우 이 맵은 매개 변수 ``secureCode`` 하나로만 구성됩니다.

일부 NIC는 유효한 MAC주소뿐만 아니라 유효한 암호도 입력해야 하는*SecureOn*기능을 지원합니다.

이 암호는 NIC에서 구성해야 합니다.

NIC가*SecureOn*을 지원하지 않거나 암호가 설정되지 않은 경우 옵션 맵을 생략하면 됩니다.

----

REST 요청
--------

``HubAction`` 은 `REST <http://en.wikipedia.org/wiki/Representational_state_transfer>`__ 장치와 통신하기 위한 콜을 만들때 이용됩니다.

다음은 간단한 예시입니다:

.. code-block:: groovy

    def myCommand() {
        def result = new physicalgraph.device.HubAction(
            method: "GET",
            path: "/yourpath?param1=value1&param2=value2",
            headers: [
                HOST: getHostAddress()
            ]
        )
        return result
    }

----

UPnP/SOAP 요청
-------------


다르게는 초기 연결 후 UPnP를 사용할 수 있습니다.

UPnP 는 `SOAP <http://en.wikipedia.org/wiki/SOAP_%28protocol%29>`__
(Simple Object Access Protocol)을 장치와 통신하기 위해 이용합니다.

SmartThings 는 ``HubSoapAction``를 이러한 목적으로 제공합니다.

HubAction 클래스와 유사하지만 (사실 이는 HubAction class를 extend한 것이다) 그것은 당신을 위해 soap envelope를 만드는 것을 다룰 것입니다.

다은은 ``HubSoapAction``를 이용하는 간단한 예시입니다:

.. code-block:: groovy

    def someCommandMethod() {
        return doAction("SetVolume", "RenderingControl", "/MediaRenderer/RenderingControl/Control", [InstanceID: 0, Channel: "Master", DesiredVolume: 3])
    }

    def doAction(action, service, path, Map body = [InstanceID:0, Speed:1]) {
        def result = new physicalgraph.device.HubSoapAction(
            path:    path,
            urn:     "urn:schemas-upnp-org:service:$service:1",
            action:  action,
            body:    body,
            headers: [Host:getHostAddress(), CONNECTION: "close"]
        )
        return result
    }

----

장치 이벤트 구독
------------

특정 Event에 대해 LAN에 연결된 장치에서 회신을 수신하고 싶은 경우 ``HubAction``을 사용하여 가입할 수 있습니다.

이 이벤트가 장치에서 발생하면 ``parse``방법이 호출됩니다.

다음은 UPnP 사용 사례입니다:

.. code-block:: groovy

    def someCommand() {
        subscribeAction("/path/of/event")
    }

    private subscribeAction(path, callbackPath="") {
        log.trace "subscribe($path, $callbackPath)"
        def address = getCallBackAddress()
        def ip = getHostAddress()

        def result = new physicalgraph.device.HubAction(
            method: "SUBSCRIBE",
            path: path,
            headers: [
                HOST: ip,
                CALLBACK: "<http://${address}/notify$callbackPath>",
                NT: "upnp:event",
                TIMEOUT: "Second-28800"
            ]
        )

        log.trace "SUBSCRIBE $path"

        return result
    }

----

참조 및 리소스
-----------

- `UPnP <http://en.wikipedia.org/wiki/Universal_Plug_and_Play>`__

- `SOAP <http://en.wikipedia.org/wiki/SOAP>`__

- `REST <http://en.wikipedia.org/wiki/Representational_state_transfer>`__
