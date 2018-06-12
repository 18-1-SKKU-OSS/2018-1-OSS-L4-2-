.. _webservices_authorization:

인증
=============

SmartApp에 REST 요청을 하려면, 클라이언트가 OAuth2 인증 코드 흐름 구현을 사용하여 신뢰할 수 있는 관계를 설정해야 합니다.

----

개요
--------

일반적인 흐름은 다음과 같습니다:

#. 인증 코드를 요청하세요.
#. 코드를 사용해 접근 토큰을 요청하세요.
#. SmartApp의 엔드포인트 URI를 가져오세요.
#. 엔드포인트 URI를 사용해 SmartApp으로 REST 호출을 하세요.

인증 흐름의 일부로, SmartApp이 사용자가 선택한 위치에 설치됩니다.

.. include:: ../common/oauth-install-restriction.txt

.. note::

    SmartApp이 실제로 게시된 서버와 관계 없이 인증 코드, 접근 토큰, 엔드포인트를 얻으려면 ``https://graph.api.smartthings.com`` 을 사용해야 합니다.

----

인증 코드 얻기
----------------------

인증 URL: ``https://graph.api.smartthings.com/oauth/authorize``

인증 코드를 얻으려면, ``https://graph.api.smartthings.com/oauth/authorize`` 으로 ``GET`` 요청을 하세요:

.. code-block:: bash

    GET https://graph.api.smartthings.com/oauth/authorize?
            response_type=code&
            client_id=YOUR-SMARTAPP-CLIENT-ID&
            scope=app&
            redirect_uri=YOUR-SERVER-URI


다음 매개변수가 필요합니다:

============== ===========
매개변수      값
============== ===========
response_type  ``code`` 를 사용해 인증 코드를 얻습니다.
client_id      SmartApp의 OAuth 클라이언트 ID 입니다.
scope          이 인증 흐름의 경우 항상 "app" 이어야 합니다.
redirect_uri   인증 코드를 수신할 서버의 URI 입니다. 이 URI은 SmartApp 설정에 지정된 재전송 URI 중 하나와 일치해야 합니다. 그렇지 않으면 유효성 검사가 실패합니다.
============== ===========

이렇게 하려면 사용자가 SmartApp 계정 자격으로 로그인하고, 위치를 선택하고, 제 3자가 접근할 수 있는 장치를 선택해야 합니다.

인증 코드는 발급 후 24시간 후에 만료됩니다.

----

접근 토큰 얻기
----------------

토큰 URL: ``https://graph.api.smartthings.com/oauth/token``

접근 토큰을 얻으려면 받은 코드를 사용하세요:

.. code::

    POST https://graph.api.smartthings.com/oauth/token HTTP/1.1
    Host: graph.api.smartthings.com
    Content-Type: application/x-www-form-urlencoded

    grant_type=authorization_code&code=YOUR_CODE&client_id=YOUR_CLIENT_ID&client_secret=YOUR_CLIENT_SECRET&redirect_uri=YOUR_REDIRECT_URI

요청의  ``content-type`` 헤더는 ``application/x-www-form-urlencoded`` 형식이어야 합니다. 다음 형식의 매개변수가 필요합니다:

=================== ===========
매개변수           값
=================== ===========
grant_type          이 흐름의 경우 항상 "authorization_code" 입니다.
code                받은 코드입니다.
client_id           SmartApp의 클라이언트 ID 입니다.
client_secret       SmartApp의 클라이언트 비밀번호입니다.
redirect_uri        토큰을 수신할 서버의 URI 입니다. 이는 인증 코드를 얻는데 사용했던 URI와 일치해야 합니다.
=================== ===========

성공적인 응답은 다음과 같을 것입니다:

.. code::

    {
      "access_token": "XXXXXXXXXXXX",
      "expires_in": 1576799999,
      "token_type": "bearer"
    }

``expires_in`` 응답은 이 토큰이 만료될 시간 (지금으로부터의 초) 입니다.

토큰을 얻었다면 이는 어플리케이션에 안전하게 저장되어야 합니다.

----

.. _web_services_get_endpoints:

SmartApp 엔드포인트 얻기
----------------------

``https://graph.api.smartthings.com/api/smartapps/endpoints`` 로 ``GET`` 요청을 하여, 토큰을 사용해  SmartApp의 호출 가능한 엔드포인트를 요청할 수 있습니다.
접근 토큰은 ``Authorization: Bearer`` 헤더를 통해 제공되어야 합니다:

.. code-block:: bash

    GET -H "Authorization: Bearer ACCESS-TOKEN" "https://graph.api.smartthings.com/api/smartapps/endpoints"

성공적인 응답은 주어진 접근 토큰과 연결된 ``clientID`` 에 대해 설치된 모든 SmartApp 리스트를 반환합니다.

.. code-block:: javascript

    [
        {
            "oauthClient": {
                "clientId": "CLIENT-ID"
            },
            "location": {
                "id": ID,
                "name": "LOCATION-NAME"
            }
            "uri": "BASE-URL/api/smartapps/installations/INSTALLATION-ID",
            "base_url": "BASE-URL",
            "url": "/api/smartapps/installations/INSTALLATION-ID"
        },
        ...
    ]

.. important::

    ``base_url`` 과 ( ``uri`` 의 기본 URL) 은 SmartApp이 설치될 서버에 따라 달라집니다.

    SmartApps는 최종 사용자의 위치에 따라 여러 대의 서버에 설치할 수 있습니다.
    이 SmartApp이 도달할 수 있는 위치를 찾으려면 항상 ``uri`` 와 ``base_url`` 를 사용해야 합니다.

    SmartApp이 ``https://graph.api.smartthings.com`` 에 설치될 것이라 생각하지 마세요.

----

REST 호출하기
---------------

``/api/smartapps/endpoints`` 에서 반환된 ``uri`` 를 사용해 REST가 SmartApp을 호출하도록 할 수 있습니다.

간단히 SmartApp이 ``mapping`` 에서 선언한 어느 경로든 추가하여 적절한 요청을 하세요.

예를 들어, ``mappings`` 정의를 다음과 같이 가정합니다:

.. code-block:: groovy

    mappings {
        path("/switches") {
            action: [GET: "getSwitches"]
        }
    }

    def getSwitches() {
        // ...
    }

그리고 ``https://graph.api.smartthings.com/api/smartapps/installations/12345`` 의 URI로, 다음과 같이 ``/switches`` 엔드포인트로 요청할 수 있습니다:

.. code-block:: bash

    curl -H "Authorization: Bearer ACCESS-TOKEN" -X GET "https://graph.api.smartthings.com/api/smartapps/installations/12345/switches"
