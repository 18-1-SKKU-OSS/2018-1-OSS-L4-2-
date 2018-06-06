.. _smartapp_as_web_service_part_1:

웹 서비스 튜토리얼--SmartApp
===============================

이것은 인증 흐름을 설명하는 웹 서비스 SmartApp과 웹 어플리케이션을 작성하는 법을 알려줄 2부 중 1부입니다.

----

개요
--------

이 튜토리얼의 1부에서, 다음과 같은 것을 배우게 될 것입니다:

- 엔드포인트를 노출하는 웹 서비스 SmartApp을 개발하는 방법
- 간단한 API 호출을 통해 웹 서비스 SmartApp을 호출하는 방법

이 튜토리얼의 소스 코드는 `여기 <https://github.com/SmartThingsCommunity/Code/tree/master/smartapps/tutorials/web-services-smartapps>`__.에서 볼 수 있습니다.

이 튜토리얼의 1부에서는 엔트포인트를 노출하여 스위치에 대한 정보를 얻고 제어하는 간단한 SmartApp을 작성합니다.

----

새 SmartApp 만들기
---------------------

IDE에서 새 SmartApp을 만드세요. 필수 입력란을 채우고, 자동 생성된 클라이언트 ID와 비밀번호를 받기 위해 *SmartApp에서 OAuth 허용* 을 클릭하세요.

인증 코드 요청의 유효성을 검사하는데 사용되므로 재전송 URL을 지정하세요.
이 튜토리얼의 의도를 위해, 간단히 ``http://localhost:4567/oauth/callback`` 을 입력하세요.

클라이언트 ID와 비밀번호는 나중에 사용되므로 기억해두세요. (잊어버리면, IDE의 "앱 설정"에서 얻을 수 있습니다.)

----

환경 설정 정의
------------------

SmartApps는 설치 및 구성 시 사용되는 환경 설정 메타 데이터를 선언하여, SmartApp이 접근할 장치를 사용자가 제어할 수 있도록 합니다.

이는 구성 단계일 뿐 아니라, SmartApp이 제어할 수 있는 장치를 사용자가 명시적으로 선택해야하는 보안 단계이기도 합니다.

웹 서비스 SmartApps는 별다른 차이점이 없으며, 이러한 접근 방식의 힘을 발휘합니다.
최종 사용자는 SmartApp이 접근할 장치를 정확히 제어하므로, 해당 웹 서비스를 사용하는 외부 시스템이 접근할 장치가 무엇인지도 제어합니다.

환경 설정 정의는 다음과 같아야 합니다:

.. code-block:: groovy

  preferences {
    section ("Allow external service to control these things...") {
      input "switches", "capability.switch", multiple: true, required: true
    }
  }

또한 ``installed()`` 와 ``updated()`` 메소드가 정의되어 있는지 확인하세요 (이것은 SmartApp을 생성할 때 기본적으로 생성되어야 합니다).
이 예에서 어떤 장치 이벤트도 구독하지 않기 때문에, 두 메소드는 비워둘 수 있습니다.

웹 서비스 SmartApp 환경 설정에 대해 `여기 <web_services_preferences>` 에서 더 자세히 알 수 있습니다.

----

엔드 포인트 지정
-----------------

``mappings`` 선언은 개발자가 HTTP 엔드포인트를 노출하고, 지원되는 다양한 HTTP 작업을 연관된 핸들러에 매핑할 수 있도록 합니다.

우리 SmartApp은 두 엔드포인트를 노출합니다:

- ``/switches`` 엔드포인트는 GET 요청을 지원합니다. 이 엔드포인트에 대한 GET 요청은 구성된 스위치에 대한 상태 정보를 반환합니다.

- ``/switches/:command`` 엔드포인트는 PUT 요청을 지원합니다. 이 엔드포인트에 대한 PUT 요청은 구성된 스위치에서 지정된 명령을 (``"on"`` 이나 ``"off"``) 실행합니다.

다음은 매핑 정의에 대한 코드입니다. 이는 SmartApp의 최상위 수준에서 정의됩니다. (즉, 다른 메소드에서 정의되지 않습니다.):

.. code-block:: groovy

    mappings {
      path("/switches") {
        action: [
          GET: "listSwitches"
        ]
      }
      path("/switches/:command") {
        action: [
          PUT: "updateSwitches"
        ]
      }
    }

PUT 엔드포인트에서 변수 파라미터를 사용합니다.
값이 가변적이도록 지정하려면 ``:`` 접두사를 사용하세요. 이 값을 얻는 방법은 나중에 살펴보겠습니다.

다양한 핸들러에 대해 빈 메소드를 추가하세요. 다음 단계에서 이를 채울 것입니다:

.. code-block:: groovy

  def listSwitches() {}

  def updateSwitches() {}

더 많은 정보를 보려면 :ref:`web_services_mapping_endpoints` 문서를 참고하세요.

----

GET 스위치 정보
----------------------

이제 엔드포인트를 정의했으므로, 위에서 정의한 핸들러 메소드에서의 요청을 처리해야 합니다.

``/switches`` 엔드포인트에 대한 GET 요청 핸들러부터 시작해보겠습니다.
``/switches`` 엔드포인트에 대한 GET 요청이 호출되면, 디스플레이 이름과 구성된 스위치의 현재 스위치 값을 (예로  on 또는 off) 반환하고자 합니다.

우리의 핸들러 메소드는 SmartThings 플랫폼에 의해 JSON으로 순차화되는 맵 목록을 반환합니다:

.. code-block:: groovy

  // 다음과 같은 리스트를 반환합니다:
  // [[name: "kitchen lamp", value: "off"], [name: "bathroom", value: "on"]]
  def listSwitches() {
      def resp = []
      switches.each {
        resp << [name: it.displayName, value: it.currentValue("switch")]
      }
      return resp
  }

웹 요청 응답에 대해 더 많은 정보를 보려면 :ref:`smartapp_web_services_response` 문서를 참고하세요.

----

스위치 업데이트
-------------------

``/switches/:command`` 엔드포인트에 대한 PUT 요청도 처리해야 합니다. ``/switches/on`` 는 스위치를 켜고, ``/switches/off`` 는 스위치를 끕니다.

어느 구성된 스위치도 지정된 명령을 지원하지 않으면, ``501`` HTTP 에러를 반환할 것입니다.

.. code-block:: groovy

    void updateSwitches() {
        // 명령 파라미터를 얻기 위해 내장 요쳥 오브젝트 사용
        def command = params.command

        // 모든 스위치는 명령을 가집니다
        // 모든 스위치의 명령 실행
        // (이것을 배열 상에서 할 수 있음을 기억하세요 - 명령은 모든 원소에서 호출됩니다)
        switch(command) {
            case "on":
                switches.on()
                break
            case "off":
                switches.off()
                break
            default:
                httpError(400, "$command is not a valid command for all switches specified")
        }
    }


이 예에서는 명령을 얻기 위해 엔드포인트 자체를 사용합니다.
요청에 대한 작업에 대해 더 알아볼 수 있습니다 :ref:`here <webservices_smartapp_request_handling>`.

----

SmartApp 자체 게시
-------------------------

*게시* 버튼을 클릭하고 *나를 위해* 를 골라 앱 자체를 위해 게시하세요.

----

.. _run_api_smartapp_simulator:

시뮬레이터에서 SmartApp 실행하기
---------------------------------

시뮬레이터를 사용해서, 우리 웹 서비스 SmartApp을 빠르게 시험해볼 수 있습니다.

시뮬레이터에서 *설치* 버튼을 누르고, SmartApp을 설치할 위치를 선택하고, 스위치를 선택하세요.

시뮬레이터의 오른쪽 아래에 API 토큰과 API 엔드포인트 URL이 있습니다:

.. image:: ../img/smartapps/web-services/web-services-smartapp-simulator-install.png

.. important::

    SmartAPp API 엔드포인트의 기본 URL은 설치될 위치에 따라 다릅니다.

    **올바른 URL을 가지기 위해 시뮬레이터에서 URL을 복사하세요!**

우리 SmartApp에 대한 요청을 시험하는데 이를 사용할 수 있습니다.

----

SmartApp으로의 API 호출 만들기
------------------------------

웹 요청을 하기 위해 선호하는 툴을 사용하여 (이 예에서는 컬을 사용하지만, `Apigee <http://apigee.com>`__ 은 요청을 하기에 좋은 UI 기반 툴입니다), SmartApp 엔드포인트 중 하나를 호출할 것입니다.

시뮬레이터에서, API 엔드포인트를 가져오세요. 그러면 다음과 같을 것입니다::

  https://<BASE-URL>/api/smartapps/installations/158ef595-3695-49ab-acc1-80e93288c0c8

설치 시 다른 고유 URL이 사용됩니다.

.. important::

    SmartAPp API 엔드포인트의 기본 URL은 설치될 위치에 따라 다릅니다.

    **올바른 URL을 가지기 위해 시뮬레이터에서 URL을 복사하세요!**

스위치에 대한 정보를 얻기 위해, GET 요청을 사용해 /switch 엔드포인트를 호출할 것입니다.
고유한 엔드포인트와 API 키를 대체해야 합니다.

.. code-block:: bash

  curl -H "Authorization: Bearer <api token>" "<api endpoint>/switches"

이는 아래와 같이 JSON 응답을 호출해야 합니다::

  [{"name":"Kitchen 2","value":"off"},{"name":"Living room window","value":"off"}]

스위치를 켜거나 끄려면, PUT 요청을 사용해 /switches 엔드포인트를 호출하세요.
다시, 고유한 엔드포인트와 API 키를 대체해야 합니다.

.. code-block:: bash

  curl -H "Authorization: Bearer <api token>" -X PUT "<api endpoint>/switches/on"

스위치를 끄려면 명령 값을 ``"off"`` 로 바꾸세요.
스위치를 켜고 끄고, 컬을 사용하여 상태가 변경되었는지 확인하세요.

----

SmartApp 삭제
----------------------

마지막으로, IDE 시뮬레이터의 *삭제* 버튼을 이용해 SmartApp을 삭제하세요.

----

요약
-------

이 튜토리얼에서, 엔드포인트가 정보를 얻고 제어할 수 있도록 하는 SmartApp을 만드는 법을 배웠습니다.
시뮬레이터에 SmartApp을 설치하고 엔드포인트에 API 호출을 하는 방법도 배웠습니다.

이 튜토리얼의 다음 부분에서는, (단순히 시뮬레이터와 생성된 접근 토큰을 사용하는 대신) OAuth2 흐름을 사용하여 외부 어플리케이션이 SmartThings와 상호 작용하는 방법을 살펴볼 것입니다.
