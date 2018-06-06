.. _async_http_guide:

===============================
비동기식 외부 HTTP 요청 생성 (베타)
===============================

.. include:: ../../common/async-http-beta-note.txt

개요
----

SmartApp과 디바이스 처리기는 HTTP를 통해 타사 서비스와 통신해야 할 수 있습니다. :ref:`calling_web_services` 문서에 나온 것과 같이 ``httpGet()``, ``httpPost()``, ``httpPut()`` 등의 다양한 HTTP API를 이용해 통신할 수 있습니다. 하지만 이러한 API는 실행되고 있는 SmartApp 또는 디바이스 처리기는 타사 서비스로부터 응답을 기다려 동기화를 합니다. 이런 동기 실행은 SmartApp 또는 디바이스 처리기를 실행하는 현재 스레드를 차단하고, 실행 시간 초과로 인한 작업 중단의 가능성을 높입니다.

이런 문제를 해결하기 위해 SmartApp과 디바이스 처리기가 *비동기적으로* HTTP를 요청할 수 있는 새로운 API를 배포하고 있습니다. 응답을 수신하면 호출할 메소드(반드시 구현해야 합니다)의 이름과 함께 요청의 세부사항을 명시합니다. SmartThings는 요청을 실행하고, 그 후 응답을 수신하면 명시된 요청 처리기 메소드를 호출합니다.

비동기식 HTTP 요청을 사용하면, 느린 타사 서비스 때문에 발생하는 실행 시간 초과로 인한 작업 중단의 가능성이 훨씬 적어집니다.

----

빠른 예제
---------

바로 비동기식 HTTP 요청 예제를 살펴보겠습니다. 이 예제는 단순하게 깃허브 API에 ``GET`` 요청을 하고, 응답을 기록합니다. 이 문서의 나머지 부분에서 다룰테니 아직 자세한 내용은 걱정하지 마시길 바랍니다.

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

가장 먼저 인지할 것은 ``include`` 명령어일 것입니다. 이는 다양한 API를 기능에 따라 그룹화할 수 있도록 하는 SmartThings의 새로운 기능입니다. :ref:`below <include_statement>` 에서 자세히 다룰테니, 아직은 너무 걱정하지 마시길 바랍니다. 
지금은 그저 특정 이름공간 - 이 예제에서는 ``"asynchttp_v1"`` - 에 존재하는 API 집합을 가져오는 방법으로 생각하시길 바랍니다. 

비동기식 HTTP 요청을 하는 코드는 매우 간단합니다. 이 예제에서는 응답을 수신할 시 호출되길 원하는 메소드의 이름, 요청을 빌드하는 데에 사용되는 데이터 지도, 그리고 선택적으로 응답 처리기에 전달될 데이터들의 지도와 함께 ``asynchttp_v1.get()`` 을 호출합니다. 요청 빌더 매개변수에 대한 자세한 정보는 :ref:`async_http_configure_request` 에 설명되어 있습니다.

선택적으로 응답 처리기 메소드를 정의할 수 있는데, 이 메소드는 ``get()`` 메소드로 전달하는 선택적인 데이터 지도뿐만 아니라 요청에 대한 응답도 받아들입니다. 아무것도 제공되지 않을 경우, 요청은 실행 후 즉시 응답이 삭제되는 '화재 및 도난'모드에서 이루어집니다. 응답 처리에 대한 자세한 내용은 :ref:`async_http_response_handling` 에 설명되어 있습니다.

----

동기 vs 비동기
-------------

다음 도표는 동기식 HTTP 요청을 하는식 것과 새로운 비동기식 HTTP API를 사용하는 것의 차이를 보여줍니다.

*Synchronous HTTP Request Flow:*

.. image:: ../../img/smartapps/sync-http-sequence.png
    :alt: Synchronous HTTP Request Flow

*Asynchronous HTTP Request Flow:*

.. image:: ../../img/smartapps/async-http-sequence.png
    :alt: Asynchronous HTTP Request Flow

위 도표로부터 동기식 HTTP 요청은 요청을 하고, 응답을 기다렸다가 응답 처리까지 모두 하나의 실행에서 한다는 것을 알 수 있습니다.

반면 비동기식 HTTP 요청은 *개별 실행* 에서 응답을 처리합니다. SmartThings 플랫폼은 요청을 하고, 응답을 기다리고, 응답이 도착하면 지정된 응답 처리기를 호출하는 새로운 SmartApp(또는 디바이스 처리기) 실행하도록 합니다.

이러한 실행이 꼭 순차적일 필요는 없다는 점에 유의하시길 바랍니다. 요청을 하고 응답을 받는 사이에 예정된 실행이나 이벤트 콜백의 결과로 다른 실행이 발생할 수 있습니다.
동기식 HTTP 요청과 비교하여 비동기식 HTTP 요청을 사용하는 것에 대한 더 많은 정보는 :ref:`async_http_when_to_use`에서 확인해보세요.

비동기식 HTTP 요청은 ``GET``, ``POST``, ``PUT``, ``DELETE``, ``HEAD``, 그리고 ``PATCH`` 의 HTTP 요청 메소드를 지원합니다. 지원되는 메소드는 :ref:`below <async_http_supported_methods>` 에 요약되어 설명되어 있습니다.

----

.. _include_statement:

include 문
------------

모든 비동기식 HTTP API는 ``include`` 문을 사용해 SmartApp 또는 디바이스 처리기에 포함될 수 있는 객체에서 이름공간을 갖습니다.

.. code-block:: groovy

    include 'asynchttp_v1'

``asynchttp_v1`` 은 비동기식 HTTP API가 존재하는 객체에 대한 참조입니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        asynchttp_v1.get([uri: 'https://api.github.com'], handler)
    }

    def handler(response, data) {
        // handle response
    }

``include`` 문은 파일 상단에 위치해야 합니다.

.. note::

	비동기식 HTTP API는 이 기능을 사용하는 첫번째 기능입니다.

	이 기능은 SmartApp 또는 디바이스 처리기에서 사용가능한 API를 세밀하게 제어할 수 있도록 하고, 전역 이름공간이 침범되지 않도록 합니다.

	``include()`` 를 사용하면 SmartThings 플랫폼은 내부적으로 등록된 API중 이름이 일치하는 API를 찾습니다.
	일치하는 API를 찾으면, 해당 API를 나타내는 클래스의 인스턴스가 SmartApp 또는 디바이스 처리기에 생성됩니다.
	일치하는 API를 찾지 못하면, 예외가 발생하고 SmartApp 또는 디바이스 처리기는 저장에 실패합니다.

----

.. _async_http_configure_request:

요청 구성하기
------------

모든 비동기 HTTP 요청 메소드는 첫번째 인자로 응답과 함께 호출할 메소드의 이름을 필요로 합니다. 또한 URI, 선택적으로 경로, URL 쿼리 매개변수, HTTP 헤더, 요청의 내용 유형을 지정해주어야 합니다. 매개변수 지도를 넘겨줌으로써 이를 해결할 수 있습니다. 아래의 표는 지도에서 지원하는 키 목록입니다.

================== ===========
Key                Description
================== ===========
uri (required)     Either a URI or URL of the endpoint to make a request from.
path               Request path that is merged with the URI.
query              Map of URL query parameters.
headers            Map of HTTP headers.
requestContentType The value of the ``Content-Type`` request header. Defaults to ``'application/json'``.
contentType        The value of the ``Accept`` request header. Defaults to the value of the ``requestContentType`` parameter if not specified.
body               The request body to send. Can be a string, or if the ``requestContentType`` is ``"application/json"``, a Map or List (will be serialized to JSON). Only valid for ``PUT``, ``POST``, ``DELETE``, and ``PATCH`` requests.
================== ===========


URI와 경로
^^^^^^^^^^

``uri`` 는 모든 비동기 HTTP 요청 메소드에 필요합니다. ``path`` 가 지정될 경우, URI와 합쳐집니다.

.. code-block:: groovy

    // uri and path merged to form "https://someapi.com/some/path"
    def params = [
        uri: 'https://someapi.com',
        path: '/some/path'
    ]

HTTP 요청을 할 때에는 공개적으로 접근할 수 있는 (즉, 로컬이 아닌) 주소로만 지정할 수 있습니다. 더 자세한 정보는 아래의 :ref:`async_http_limits` 을 보시길 바랍니다.

요청 헤더
^^^^^^^^^

위의 표에서 볼 수 있듯이, 모든 요청에는 `Content-Type`` 과 ``Accept`` 라는 요청 헤더가 추가됩니다. 
다른 요청 헤더로 설정해야 한다면, 매개변수 지도에서 ``headers`` 키를 이용해 지정하시길 바랍니다.

.. code-block:: groovy

    def params = [
        uri: 'https://api.github.com',
        path: '/repos/SmartThingsCommunity/SmartThingsPublic/events',
        headers: ['If-None-Match': 'c873e724d02caa124de0884535c32acb']
    ]
    asynchttp_v1.get('someHandlerMethod', params)

위와 같이 구성했기 때문에 요청은 다음과 같은 형태를 갖습니다.

.. code-block:: http

    GET /repos/SmartThingsCommunity/SmartThingsPublic/events HTTP/1.1

    Host: api.github.com
    Content-Type: application/json
    Accept: application/json
    If-None-Match: c873e724d02caa124de0884535c32acb


쿼리 매개변수
^^^^^^^^^^^^

URL 쿼리 매개변수는 지도의 ``query`` 키 값을 지정함으로써 요청에 추가될 수 있습니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        // search for occurences of httpGet in the SmartThingsPublic repo
        def params = [
            uri: 'https://api.github.com',
            path: '/search/code',
            query: [q: "httpGet+repo:SmartThingsCommunity/SmartThingsPublic"],
            contentType: 'application/json'
        ]
        asynchttp_v1.get(processResponse, params)
    }

    def processResponse(response, data) { ... }


위의 코드로 만들어진 요청은 다음과 같은 형태를 갖습니다.

.. code-block:: http

    GET /search/code?q=httpGet+repo:SmartThingsCommunity/SmartThingsPublic HTTP/1.1

    Host: api.github.com
    Content-Type: application/json
    Accept: application/json

본문 요청
^^^^^^^^

본문을 가질 수 있는 HTTP 요청 메소드는 또한 매개변수 지도에 ``body`` 를 지정해줄 수 있습니다. 
``body`` 값은 문자열이 될 수 있으며, ``requestContentType`` 이 ``"application/json"`` 일 경우 JSON으로 직렬화 될 맵 또는 리스트가 될 수 있습니다.
:ref:`async_http_ref_put`, :ref:`async_http_ref_post`, :ref:`async_http_ref_delete`, :ref:`async_http_ref_patch` 메소드는 ``body`` 옵션을 지원합니다.

지도를 이용해 본문을 요청하는 ``POST`` 를 하는 예제입니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        def params = [
            uri: 'https://someapi.com',
            path: '/some/path',
            body: [key1: 'value 1']
        ]
        asynchttp_v1.post(processResponse, params)
    }

    def processResponse(response, data) { ... }

요청은 다음과 같은 형태를 갖습니다. (``Content-Type`` 와 ``Accept`` 헤더는 기본 값인 ``"application/json"`` 입니다.)

.. code-block:: bash

    POST /some/path

    Host: someapi.com
    Content-Type: application/json
    Accept: application/json

    {"key1": "value 1"}

본문으로 문자열을 사용해 ``PUT`` 을 하는 예제입니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        def params = [
            uri: 'https://someapi.com',
            path: '/some/path',
            body: "<entity><name>test</name></entity>",
            requestContentType: "application/xml"
        ]
        asynchttp_v1.put(processResponse, params)
    }

    def processResponse(response, data) { ... }

위의 예제로 만들어진 요청은 다음과 같은 형태를 갖습니다.

.. code-block:: bash

    PUT /some/path

    Host: someapi.com
    Content-Type: application/xml
    Accept: application/xml

    <entity><name>test</name></entity>

----

요청 처리
^^^^^^^^

SmartThings가 지정한 요청을 실행하고 제3자로부터 응답을 받으면, (지정된 경우) 요청 처리기 메소드가 (SmartApp 또는 디바이스 처리기의 새로운 실행에서) 호출됩니다. 
요청 처리기 메소드는 :ref:`AsyncResponse <async_http_response_ref>` 의 인스턴스로 호출될 것이고, 이를 통해 응답에 대한 정보를 얻을 수 있습니다.


응답 처리기 메소드는 또한 요청에서 지정된 데이터 지도를 받아야 합니다. 
이는 요청을 만들고 응답을 받기까지 동안 데이터를 전달할 때 유용합니다. 
요청을 할 때 (선택적인) 데이터를 지정하지 않는다면, 요청 처리기 메소드는 두번째 매개변수로 ``null`` 이 지정된 채 호출될 것입니다.
이러한 선택적인 데이터 매개변수는 이 문서의 뒷부분에서 설명하겠습니다.

응답 처리기 메소드의 시그니처는 다음과 같아야 합니다.

.. code-block:: groovy

    def someResponseHandler(response, data) {}



응답 상태 코드
^^^^^^^^^^^^^

다른 가능한 응답 코드를 처리해야 하는 경우, 응답 상태 코드를 얻을 수 있습니다.

.. code-block:: groovy

    def responseHandler(response, data) {
        def status = response.status
        switch (status) {
            case 200:
                log.debug "200 returned"
                break
            case 304:
                log.debug "304 returned"
                break
            default:
                log.warn "no handling for response with status $status"
                break
        }
    }


응답 헤더
^^^^^^^^

AsyncResponse 객체는 응답으로부터 전해받는 모든 헤더를 키-값 쌍의 지도로 포함합니다. (반환 유형은 ``Map<String, String>`` 입니다.)

.. code-block:: groovy

    def responseHandler(response, data) {
        def headers = response.headers
        headers.each { header, value ->
            log.debug "$header: $value"
        }
        // can use array notation to get specific header values
        def etagHeader = response.headers['ETag']
    }

응답 오류
^^^^^^^^

:ref:`async_response_ref_has_error` 을 이용해 응답에 오류를 있는지 확인하시길 바랍니다.
``hasError()`` 는 요청 도중 예외가 발생하면 true를 반환합니다.

**2XX가 아닌 응답도 오류로 간주됩니다.**

:ref:`async_response_ref_get_error_message` 메소드를 이용해 오류 메세지를 받을 수 있습니다.

.. code-block:: groovy

    def responseHandler(response, data) {
        if (response.hasError()) {
            log.debug "response received error: ${response.getErrorMessage()}"
        }
    }

응답 오류의 경우, :ref:`async_response_ref_get_error_data`, :ref:`async_response_ref_get_error_json`, 또는 :ref:`async_response_ref_get_error_xml` 을 이용해 응답 본문을 받을 수 있습니다.
이러한 메소드들은 성공적인 응답에 호출될 시 예외를 발생시킵니다.

.. code-block:: groovy

    def responseHandler(response, data) {
        if (response.hasError()) {
            log.debug "error response data: $response.errorData"
            try {
                // exception thrown if json cannot be parsed from response
                log.debug "error response json: $response.errorJson"
            } catch (e) {
                log.warn "error parsing json: $e"
            }
            try {
                // exception thrown if xml cannot be parsed from response
                log.debug "error response xml: $response.errorXml"
            } catch (e) {
                log.warn "error parsing xml: $e"
            }
        }
    }

JSON 응답
^^^^^^^^^

요청에 대한 응답이 JSON인 경우, :ref:`async_response_ref_get_json` 을 이용해 응답의 완전한 형식을 가진 JSONObject를 받을 수 있습니다.
아래 예제는 SmartThingsPublic 저장소의 "httpGet"을 발생시키는 GitHub API 호출로 JSON 응답을 받는 것을 보여줍니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        def params = [
            uri: 'https://api.github.com',
            path: '/search/code',
            query: [q: "httpGet+repo:SmartThingsCommunity/SmartThingsPublic"]
        ]
        asynchttp_v1.get(processResponse, params)
    }

    def processResponse(response, data) {
        if (response.hasError()) {
            log.error "response has error: $response.errorMessage"
        } else {
            def results
            try {
                // json response already parsed into JSONElement object
                results = response.json
            } catch (e) {
                log.error "error parsing json from response: $e"
            }
            if (results) {
                def total = results?.total_count

                log.debug "there are $total occurences of httpGet in the SmartThingsPublic repo"

                // for each item found, log the name of the file
                results?.items.each { log.debug "httpGet usage found in file $it.name" }
            } else {
                log.debug "did not get json results from response body: $response.data"
            }
        }
    }

``getJson()`` 은 응답 본문이 JSON으로 파싱될 수 없을 때, 요청이 응답을 얻는 데에 실패했을 때, 또는 응답 상태 코드가 2XX가 아닌 경우 예외를 발생시킵니다.
더 자세한 정보는 :ref:`async_response_ref_get_json` 을 참고하시길 바랍니다.

XML 응답
^^^^^^^^

XML 응답을 처리하는 방식은 JSON과 유사합니다. XML은 우리가 사용할 수 있는 자료구조로 파싱됩니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
      def params = [
          uri: 'https://httpbin.org',
            path: '/xml',
            requestContentType: 'application/xml'
        ]
        asynchttp_v1.get('xmlResultsHandler', params)
    }

    def xmlResultsHandler(response, data) {
        // results look like:
        // <slideshow title="Sample Slide Show" date="Date of publication" author="Yours Truly">
        //     <slide type="all">
        //         <title>Wake up to WonderWidgets!</title>
        //     </slide>
        // </slideshow>
        if (!response.hasError()) {
            def slideshow
            try {
                slideshow = response.xml
            } catch (e) {
                log.error "error parsing XML from response: $e"
            }
            if (slideshow) {
                log.debug "title: ${slideshow.slide.title.text()}" // -> Wake up to WonderWidgets!
            }
        } else {
            log.error "error making request: ${response.getErrorMessage()}"
        }
    }

``getJson()`` 과 비슷하게, ``getXml()`` 은 응답 본문이 XML로 파싱될 수 없을 때 예외를 발생시킵니다.
더 자세한 정보는 :ref:`async_response_ref_get_xml` 를 참고하시길 바랍니다.

원시 응답 받기
^^^^^^^^^^^^^

원시 응답 데이터를 받고 싶다면, :ref:`async_response_ref_get_data` 을 사용해 받을 수 있습니다.

.. code-block:: groovy

    def responseHandler(response, data) {
        log.debug "the raw response data is: $response.data"
    }

----

.. _passing_data_to_request_handler:

요청 처리기에 데이터 보내기
-------------------------

비동기식 HTTP 요청에 대한 응답은 별도의 SmartApp 또는 디바이스 처리기의 실행에서 처리되므로 요청을 할 때와 응답 처리기를 호출할 때 사이에서 데이터를 공유할 방법이 필요합니다.
:ref:`State <storing-data>` 에 데이터를 저장하는 대신, 비동기식 HTTP 메소드에 데이터 지도를 전달할 수 있고 이는 응답 처리기에도 전달됩니다.

.. note::

	모든 응답 처리기 메소드는 요청에 데이터가 지정되지 않은 경우에도 반드시 데이터 지도에 대한 두번째 매개변수를 받아야 합니다.
	요청에 데이터가 지정되지 않은 경우, 응답 처리기에 전달되는 값은 ``null`` 입니다.

	응답 처리기가 두번째 매개변수를 받지 않은 경우, 플랫폼이 응답 처리기를 호출하려 할 때``MethodMissingException`` 오류가 발생합니다.

.. code-block:: groovy

    include 'asynchttp_v1'

    def initialize() {
        def params = [uri: 'https://someapi.com']
        def data = [key1: "value 1", key2: "value 2"]
        asynchttp_v1.get(handler, params, data)
    }

    def handler(response, data) {
        // logs [key1: "value 1", key2: "value 2"]
        log.debug "data passed to response handler: $data"
    }

----

.. _async_http_supported_methods:

사용가능한 메소드
----------------

다음의 메소드는 ``asynchttp_v1`` 객체에서 사용할 수 있습니다.
HTTP 요청 메소드는 ``asynchttp_v1`` 메소드의 이름과 일치해야 합니다.--각 메소드에 대해 자세한 정보는 참고 문서를 확인하시길 바랍니다.

========= ======
HTTP 동사 메소드
========= ======
GET       :ref:`asynchttp_v1.get(String callbackMethod, Map params, Map data = null) <async_http_ref_get>`
PUT       :ref:`asynchttp_v1.put(String callbackMethod, Map params, Map data = null) <async_http_ref_put>`
POST      :ref:`asynchttp_v1.post(String callbackMethod, Map params, Map data = null) <async_http_ref_post>`
DELETE    :ref:`asynchttp_v1.delete(String callbackMethod, Map params, Map data = null) <async_http_ref_delete>`
PATCH     :ref:`asynchttp_v1.patch(String callbackMethod, Map params, Map data = null) <async_http_ref_patch>`
HEAD      :ref:`asynchttp_v1.head(String callbackMethod, Map params, Map data = null) <async_http_ref_head>`
========= ======

----

.. _async_http_limits:

호스트, 시간초과, 응답 및 데이터 크기 제한
--------------------------------------

.. _async_http_blacklisting:

호스트와 IP 주소 제한
^^^^^^^^^^^^^^^^^^^^

공개적으로 접근 가능한 호스트로만 요청할 수 있습니다.
HTTP 요청을 실행할 때, 요청은 허브가 아닌 SmartThings 플랫폼 (즉, SmartThings 클라우드)로부터 생성된다는 점을 기억하시길 바랍니다.

로컬 또는 개인 호스트에 대한 요청은 허용되지 않으며 ``SecurityException`` 으로 실패합니다.

요청 시간초과 제한
^^^^^^^^^^^^^^^^

요청은 40초 후에 시간초과됩니다.
요청이 시간초과되면 응답 처리기가 호출되며, 응답에 오류가 발생합니다.

.. code-block:: groovy

    def responseHandler(response, data) {
        if (response.hasError()) {
            log.error "response has error: $response.errorMessage"
        }
    }

응답 크기 제한
^^^^^^^^^^^^^

현재 응답 데이터의 제한은 500,000자 입니다.
이 제한은 베타 버젼 동안 고려되고 있으며, 필요에 따라 조정될 수 있습니다.

제한 글자 수를 초과하면, 응답 본문은 비어 있게 되지만 응답 상태는 실제 응답 상태를 반영합니다.
응답 크기가 제한 글자 수를 초과했다는 경고 메세지가 :ref:`async_response_ref_get_warning_messages` 에 추가됩니다.

데이터 크기 제한
^^^^^^^^^^^^^^^

응답 처리기에 전달되는 데이터 지도의 크기는 JSON으로 직렬화되었을 때 1000자로 제한됩니다.
제한 글자 수를 초과할 경우 요청을 하면 ``IllegalArugumentException`` 이 발생합니다.

----

부모-자식 관계에서 비동기식 HTTP 사용법
------------------------------------

비동기식 HTTP 요청을 할 때, 관련 응답 처리기 메소드는 요청한 SmartApp에서 호출됩니다.
이는 당연하지만, 부모-자식 관계의 SmartApp 또는 디바이스 처리기를 개발할 경우 이 사실을 염두에 두어야합니다.

예를 들어 부모에 응답 처리기가 존재하는 한, 자식 SmartApp 또는 디바이스 처리기는 부모에서 비동기식 HTTP 요청을 하는 메소드를 호출할 수있습니다. 

----

.. _async_http_when_to_use:

동기식 HTTP 요청을 사용하는 경우
-------------------------------

간단히 말해서 동기식 HTTP 요청이 필요하다는 게 분명하지 않는 한 비동기식 HTTP 요청을 선호하시길 바랍니다.
각 경우는 독자적으로 고려해야하지만, 동기식 HTTP 요청이 필요한 일반적인 경우가 있습니다.

- 클라우드 간 디바이스 통합 설치 동안 OAuth 흐름과 같이, 응답이 UI에서 사용되는 경우
- 응답이 다른 API로 즉시 반환되고, 해당 API를 수정할 수 없는 경우

다음 절에서는 동기식 HTTP 요청을 비동기식 HTTP 요청으로 수정하는 몇 가지 방법에 대해 설명하고, 비동기성이 요구하는 설계 변경 사항 중 일부를 강조합니다.

----

