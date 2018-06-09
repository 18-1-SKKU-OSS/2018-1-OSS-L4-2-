.. _calling_web_services:

동기식 외부 HTTP 요청하기
========================

SmartApp 또는 디바이스 처리기는 외부 웹 서비스를 호출해야 할 수 있습니다. 이러한 요청을 할 수 있도록 해주는 몇 가지 사용가능한 API가 있습니다.

다양한 API는 사용할 HTTP 요청 메소드에 따라 이름이 지정됩니다. 예를 들어 ``httpGet()`` 은 HTTP GET 요청을 합니다.

.. note::

	여기서 설명하는 API는 단일 SmartApp 또는 디바이스 처리기 실행 내에서 동기식으로 실행됩니다.

	비동기식 HTTP 요청에 대한 정보는 :문서:`async-http` 문서를 확인하시길 바랍니다.

----

HTTP 메소드
-----------

다음 메소드는 HTTP 요청을 하는 데에 사용가능합니다.
각 메소드에 대해 :참고:`smartapp_ref` API 문서에서 더 자세히 읽으실 수 있습니다.

이 메소드는 동기식으로 실행되고, 응답을 받는 데에 10초의 제한 시간이 있습니다.

============================== ================
메소드                          설명
============================== ================
:참고:`smartapp_http_delete`    HTTP DELETE 요청 실행
:참고:`smartapp_http_get`       HTTP GET 요청 실행
:참고:`smartapp_http_head`      HTTP HEAD 요청 실행
:참고:`smartapp_http_post`      HTTP POST 요청 실행
:참고:`smartapp_http_post_json` JSON Content-Type으로 HTTP POST 요청 실행
:참고:`smartapp_http_put_json`  JSON Content-Type으로 HTTP PUT 요청 실행
============================== ================

다음은 HTTP GET 요청을 하는 간단한 예제입니다.

.. code-block:: groovy

    def params = [
        uri: "http://httpbin.org",
        path: "/get"
    ]

    try {
        httpGet(params) { resp ->
            resp.headers.each {
               log.debug "${it.name} : ${it.value}"
            }
            log.debug "response contentType: ${resp.contentType}"
            log.debug "response data: ${resp.data}"
        }
    } catch (e) {
        log.error "something went wrong: $e"
    }

----

요청 구성하기
------------

HTTP 요청을 만드는 다양한 API는 모두 요청에 대한 다양한 정보를 정의하는 매개변수 지도를 허용합니다.

=================== ==============
매개변수             설명
=================== ==============
uri                 요청을 보내는 엔드포인트의 URI 또는 URL
path                URI와 병합될 요청 경로
query               URL 쿼리 매개변수의 지도
headers             HTTP 헤더의 지도
contentType         요청 컨텐츠 유형 및 허용 헤더 
requestContentType  예상된 응답 컨텐츠 유형과 다를 경우, 요청에 대한 컨텐츠 유형
body                지정된 contentType에 따라 인코딩 될 요청 본문
=================== ==============

.. note::

	``requestContentType`` 을 지정하면 호출하는 다양한 http API의 기본 동작을 오버라이드할 수 있습니다.
	예를 들어, ``httpPostJson()`` 은 ``requestContentType`` 을 기본 값으로 ``"application/json"`` 으로 설정합니다.

----

응답 처리하기
------------

HTTP API는 요청으로부터 응답 정보와 함께 호출 될 클로저를 허용합니다. 

클로저는 `HttpResponseDecorator <https://github.com/jgritman/httpbuilder/blob/855e1784be8585de81cc3c99fd19285798c7bc4f/src/main/java/groovyx/net/http/HttpResponseDecorator.java>`__ 의 인스턴스로 전달됩니다.
이 객체를 검사하여 응답에 대한 정보를 얻을 수 있습니다.

다음은 다양한 응답 정보를 얻어오는 예제입니다.

.. code-block:: groovy

    def params = [
        uri: "http://httpbin.org",
        path: "/get"
    ]

    try {
        httpGet(params) { resp ->
            // 모든 헤더에 대해 반복합니다
            // 각 헤더는 이름과 값을 가집니다
            resp.headers.each {
               log.debug "${it.name} : ${it.value}"
            }

            // 지정된 키를 가지는 모든 헤더 배열을 가져옵니다
            def theHeaders = resp.getHeaders("Content-Length")

            // 응답의 contentType을 가져옵니다
            log.debug "response contentType: ${resp.contentType}"

            // 응답의 상태 코드를 가져옵니다
            log.debug "response status code: ${resp.status}"

            // 응답 본문으로부터 데이터를 가져옵니다
            log.debug "response data: ${resp.data}"
        }
    } catch (e) {
        log.error "something went wrong: $e"
    }


.. tip::

	'실패' 응답은 예외를 발생시키므로 호출 구문을 try/catch 블럭으로 감싸시길 바랍니다.

응답이 JSON을 반환하는 경우, ``data`` 는 응답 데이터에 쉽게 접근할 수 있게 해주는 지도 같은 구조를 가집니다.

.. code-block:: groovy

    def makeJSONWeatherRequest() {
        def params = [
            uri:  'http://api.openweathermap.org/data/2.5/',
            path: 'weather',
            contentType: 'application/json',
            query: [q:'Minneapolis', mode: 'json']
        ]
        try {
            httpGet(params) {resp ->
                log.debug "resp data: ${resp.data}"
                log.debug "humidity: ${resp.data.main.humidity}"
            }
        } catch (e) {
            log.error "error: $e"
        }
    }

위 예제의 요청으로부터 얻은 ``resp.data`` 는 다음과 같습니다. (가독성을 위해 들여 쓰기를 했습니다.)

.. code-block:: bash

    resp data: [id:5037649, dt:1432752405, clouds:[all:0],
        coord:[lon:-93.26, lat:44.98], wind:[speed:4.26, deg:233.507],
        cod:200, sys:[message:0.012, sunset:1432777690, sunrise:1432722741,
            country:US],
        name:Minneapolis, base:stations,
        weather:[[id:800, icon:01d, description:Sky is Clear, main:Clear]],
        main:[humidity:73, pressure:993.79, temp_max:298.696, sea_level:1026.82,
            temp_min:298.696, temp:298.696, grnd_level:993.79]]

위에서 보이는 것과 같은 자료구조로부터 습도를 쉽게 얻을 수 있습니다.

.. code-block:: groovy

    resp.data.main.humidity

----

호스트와 시간초과 제한
--------------------

호스트와 IP 주소 제한
^^^^^^^^^^^^^^^^^^^^

공개적으로 접근 가능한 호스트로만 요청할 수 있습니다.
HTTP 요청을 실행할 때, 요청은 허브가 아닌 SmartThings 플랫폼 (즉, SmartThings 클라우드)로부터 생성된다는 점을 기억하시길 바랍니다.

로컬 또는 개인 호스트에 대한 요청은 허용되지 않으며 ``SecurityException`` 으로 실패합니다.

요청 시간초과 제한
^^^^^^^^^^^^^^^^

요청은 10초 후에 시간초과됩니다.

한 실행 내에서 동기식으로 요청이 실행되기 때문에, 새로운 (현재는 베타 버젼) :doc:`async-http` 기능을 확인하시는 것이 좋습니다.

----

시도해보세요
-----------

다양한 HTTP API를 사용해보고 싶다면, API 키를 등록하지 않고 API를 사용할 수 있는 몇 가지 툴이 있습니다.

`httpbin.org <http://httpbin.org/>`__ 을 이용해 간단한 요청을 해볼 수 있습니다.
위의 ``httpGet()`` 예제는 이곳을 이용합니다.

POST 요청을 해볼 때에는 `PostCatcher <http://postcatcher.in/>`__ 를 이용할 수 있습니다.
대상 URL을 생성한 후 요청 내용을 검사할 수 있습니다.
다음은 ``httpPostJson()`` 을 사용한 예제입니다.

.. code-block:: groovy

    def params = [
        uri: "http://postcatcher.in/catchers/<yourUniquePath>",
        body: [
            param1: [subparam1: "subparam 1 value",
                     subparam2: "subparam2 value"],
            param2: "param2 value"
        ]
    ]

    try {
        httpPostJson(params) { resp ->
            resp.headers.each {
                log.debug "${it.name} : ${it.value}"
            }
            log.debug "response contentType: ${resp.    contentType}"
        }
    } catch (e) {
        log.debug "something went wrong: $e"
    }

----

참고 항목
---------

SmartSense 온습도 센서를 사용자의 날씨 지하 개인 기상 관측소에 연결하는 ``httpGet()`` 를 사용한 예제는 `이곳 <https://github.com/SmartThingsCommunity/Code/blob/e8a6b6926fb32df1e8d79bfe09a1ad063682396a/smartapps/wunderground-pws-connect.groovy>`_ 에서 확인하실 수 있습니다.

다양한 HTTP를 사용하는 IDE 일부 템플릿을 검색할 수 있습니다. Ecobee Service Manager는 ``httpGet()`` 과 ``httpPost()`` 를 모두 사용하는 예제입니다.
