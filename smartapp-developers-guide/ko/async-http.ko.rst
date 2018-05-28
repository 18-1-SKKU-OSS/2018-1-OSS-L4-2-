.. _async_http_guide:

===============================
비동기 외부 HTTP 요청 생성 (베타)
===============================

.. include:: ../../common/async-http-beta-note.txt

개요
----

SmartApp과 디바이스 처리기는 HTTP를 통해 타사 서비스와 통신해야 할 수 있습니다. :ref:`calling_web_services` 문서에 나온 것과 같이 ``httpGet()``, ``httpPost()``, ``httpPut()``등의 다양한 HTTP API를 이용해 통신할 수 있습니다. 하지만 이러한 API는 실행되고 있는 SmartApp 또는 디바이스 처리기는 타사 서비스로부터 응답을 기다려 동기화를 합니다. 이런 동기 실행은 SmartApp 또는 디바이스 처리기를 실행하는 현재 스레드를 차단하고, 실행 시간 초과로 인한 작업 중단의 가능성을 높입니다.

이런 문제를 해결하기 위해 SmartApp과 디바이스 처리기가 *비동기적으로* HTTP를 요청할 수 있는 새로운 API를 배포하고 있습니다. 응답을 수신하면 호출할 메소드(반드시 구현해야 합니다)의 이름과 함께 요청의 세부사항을 명시합니다. SmartThings는 요청을 실행하고, 그 후 응답을 수신하면 명시된 요청 처리기 메소드를 호출합니다.

비동기 HTTP 요청을 사용하면, 느린 타사 서비스 때문에 발생하는 실행 시간 초과로 인한 작업 중단의 가능성이 훨씬 적어집니다.

----

빠른 예제
---------

바로 비동기 HTTP 요청 예제를 살펴보겠습니다. 이 예제는 단순하게 깃허브 API에 ``GET`` 요청을 하고, 응답을 기록합니다. 이 문서의 나머지 부분에서 다룰테니 아직 자세한 내용은 걱정하지 마시길 바랍니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        def params = [
            uri:  'https://api.github.com',
            contentType: 'application/json'
        ]
        def data = [key1: "hello world"]

        asynchttp_v1.get('responseHandlerMethod', params, data)
    }

    def responseHandlerMethod(response, data) {
        log.debug "got response data: ${response.getData()}"
        log.debug "data map passed to handler method is: $data"
    }

가장 먼저 인지할 것은 ``include`` 명령어일 것입니다. 이는 다양한 API를 기능에 따라 그룹화할 수 있도록 하는 SmartThings의 새로운 기능입니다. :ref:`below <include_statement>`에서 자세히 다룰테니, 아직은 너무 걱정하지 마시길 바랍니다. 지금은 그저 특정 이름공간-이 예제에서는 "``asynchttp_v1"``-에 존재하는 API 집합을 가져오는 방법으로 생각하시길 바랍니다. 

비동기 HTTP 요청을 하는 코드는 매우 간단합니다. 이 예제에서는 응답을 수신할 시 호출되길 원하는 메소드의 이름, 요청을 빌드하는 데에 사용되는 데이터 지도, 그리고 선택적으로 응답 처리기에 전달될 데이터들의 지도와 함께 ``asynchttp_v1.get()``을 호출합니다. 요청 빌더 매개변수에 대한 자세한 정보는 :ref:`async_http_configure_request`에 설명되어 있습니다.

선택적으로 응답 처리기 메소드를 정의할 수 있는데, 이 메소드는 ``get()``메소드로 전달하는 선택적인 데이터 지도뿐만 아니라 요청에 대한 응답도 받아들입니다. 아무것도 제공되지 않을 경우, 요청은 실행 후 즉시 응답이 삭제되는 '화재 및 도난'모드에서 이루어집니다. 응답 처리에 대한 자세한 내용은 :ref:`async_http_response_handling`에 설명되어 있습니다.

----
