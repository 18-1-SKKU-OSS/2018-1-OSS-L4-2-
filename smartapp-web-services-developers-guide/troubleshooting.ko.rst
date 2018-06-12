.. _web_services_smartapps_troubleshooting:

===============
문제 해결
===============

일반
-------

웹 서비스 SmartApps 개발 및 테스트는 OAuth 프로세스 특성 상 까다로울 수 있습니다.
성공을 돕기 위한 몇 가지 일반적인 팁과 전략은 다음과 같습니다:

- :ref:`Web Services Authorization <webservices_authorization>` 문서를 읽고 이해했는지 확인하세요.
- SmartThings에서 게시한 SmartApp만 일반 사용자 계정에 설치할 수 있음을 기억하세요. SmartApp을 자체 게시한 경우, *게시한 계정만 테스트 용도로 SmartApp을 설치할 수 있습니다.*
- 토큰을 수신할 호출 가능 URL을 노출시키지 않고 브라우저를 통해 OAuth 프로세스를 완료하려고 하면, 작동하지 않습니다. 이것을 어떻게 할 수 있는지 보려면 :ref:`smartapp_as_web_service_part_2` 를 읽어보세요.
- SmartApp에 API 호출을 하려면 먼저 :ref:`설치된 SmartApp에 특정한 URL을 얻기 위해 REST 호출을 해야합니다 <web_services_get_endpoints>`. *SmartApp이 설치된 특정 서버와 관계 없이* ``https://graph.api.smartthings.com/api/smartapps/endpoints`` 를 항상 만들어야 합니다.

----

설치 중 오류
--------------------------

위치를 선택하고 인증할 장치를 선택할 때 발생할 수 있는 몇 가지 일반적인 오류가 있습니다.

"<clientID> 는 위치에 있는 SmartApp과 연관되어 있지 않습니다
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

문제
    OAuth 흐름을 통해 웹 서비스 SmartApp을 설치하려고 시도하면, SmartThings는 해당 클라이언트 ID로 해당 위치의 특정 서버에 게시된 SmartApp을 찾습니다.
    이 오류는 SmartApp이 사용자가 설치 중인 서버에 게시되지 않거나 웹 서비스 SmartApp을 SmartApp을 게시하지 않은 계정에 설치하려고 시도할 때 발생합니다.

해결
    만약 SmartApp이 자체 게시 되었다면, 설치 시 계정과 같은 계정을 사용하고 있는지 확인하세요  (SmartThings에서 게시한 웹 서비스 SmartApps만 다른 사용자 계정에 설치될 수 있습니다.)
    동일한 계정인데 다른 위치에 설치하려고 하는 경우, SmartApp이 해당 위치에 게시되어 있는지 확인하세요 (다른 OAuth 클라이언트 ID 및 비밀번호 처리를 요청할 것입니다).

    SmartThings에서 발행한 SmartApp인 경우 support@smartthings.com에 문의하세요.

----

인증 클릭 후 "인증할 장치를 하나 이상 선택하세요" 오류
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

문제
    인증할 장치를 선택한 경우, 이 오류는 설치 과정 자체에서 (``installed()`` 또는 ``updated()`` 메소드에서) 예외가 발생했음을 나타냅니다.

해결
    라이브 로깅에서 예외가 있는지 확인하고, ``installed()`` 또는 ``updated()`` 메소드에서 실행 중인 코드를 찾아 가능한 버그를 찾으세요.
