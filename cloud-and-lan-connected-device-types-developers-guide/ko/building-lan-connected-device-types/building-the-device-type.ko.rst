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
    요청에 사용할 HTTP 메서드.

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

``parseLanMessage`` 메서드를 사용하여 수신 메시지를 구문 분석할 수 있습니다.

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
