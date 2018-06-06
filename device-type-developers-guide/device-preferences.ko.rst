Preferences
===========
Device Handler는 사용자가 장치의 특정 속성을 구성할 수 있도록 간단한 기본 설정을 지정할 수 있습니다.

----

개요
--------

사용자가 SmartThings에 장치를 추가하면 기기 이름을 지정하고 필요한 공간을 선택할 수 있습니다.
사용자에게 노출시키려는 추가 구성 옵션이 있는 경우에는, ``preferences`` 을 사용하여 지정할 수 있습니다.
이것들은 장치 이름 환경 설정과 동일한 페이지에 나타날 겁니다.

----

우선권 정의
--------------------

기기 우선권은 Device Handler의 ``metadata`` 에 위치해야 합니다 .
우선권들은 ``metadata`` 정의의 아무데서나 나타날 수 있습니다.

.. code-block:: groovy

    metadata {
        definition(...) {...}
        tiles() {...}
        preferences {
            input "tempOffset", "number", title: "Degrees", description: "Adjust temperature by this many degrees",
                  range: "*..*", displayDuringSetup: false
        }
    }

----

디바이스 우선권은 단조롭다
---------------------------

*디바이스 우선권은 정적이고, 한 장이면 충분합니다*

여러 페이지 기본 설정 및 동적 환경 설정 페이지는 Device Handlers에서 지원되지 않습니다. Device Handler의 기본 설정은 다음과 같은 간단한 입력 목록입니다:

.. code-block:: groovy

    preferences {
        input name: "text", type: "text", title: "Text", description: "Enter Text", required: true
    }

----

환경 설정 화면
----------------

디바이스가 SmartThings에 추가될때 기본 설정 입력을 강제로 표시하려면, ``displayDuringSetup: true`` 를 사용하세요:

.. code-block:: groovy

    preferences {
        input name: "email", type: "email", title: "Email", description: "Enter Email Address", required: true,
              displayDuringSetup: true
    }

이 값을 지정하지 않거나  ``displayDuringSetup: false`` 를 지정하는 기본 설정은, 사용자가 모바일 응용 프로그램의 장치에서 설정 버튼을 누를 때만 나타납니다.
    
----

지원되는 입력 타입들
---------------------
다음 입력 타입 들은 Device Handler 기본 설정에서 가능한 것들 입니다. 

- ``bool``
- ``decimal``
- ``email``
- ``enum``
- ``number``
- ``password``
- ``phone``
- ``time``
- ``text``

----

기본 설정 입력 값 가져오기
-------------------------------

SmartApp기본 설정과 마찬가지로, 기본 설정 입력의 이름은 기본 설정 값에 대한 참조입니다.

.. code-block:: groovy

    metadata {
        definition(...) {...}
        tiles() {...}
        preferences {
            input "tempOffset", "number", title: "Degrees", description: "Adjust temperature by this many degrees", range: "*..*", displayDuringSetup: false
        }
    }

    def someCommandMethod() {
        if (tempOffset) {
            // offset 값을 처리합니다. 
        }
    }

.. note::
   
    기본 설정 값은 이벤트 또는 명령에 대한 응답으로 실행될 때 Device Handler에서만 사용할 수 있습니다.
    선호 가치를 ``tiles()`` 을 포함한 다른 ``metadata`` 정의에 사용하는 것은 불가능하다.

----

예시
-------

.. code-block:: groovy

    metadata {
        simulator {
            // 할 일: 여기서 상태와 대답 메세지를 정의하세요
        }

        tiles {
            // 할 일: 타일의 메인부분과 세부사항을 여기서 정의하세요
        }

        preferences {
            input name: "email", type: "email", title: "Email", description: "Enter Email Address", required: true, displayDuringSetup: true
            input name: "text", type: "text", title: "Text", description: "Enter Text", required: true
            input name: "number", type: "number", title: "Number", description: "Enter number", required: true
            input name: "bool", type: "bool", title: "Bool", description: "Enter boolean", required: true
            input name: "password", type: "password", title: "password", description: "Enter password", required: true
            input name: "phone", type: "phone", title: "phone", description: "Enter phone", required: true
            input name: "decimal", type: "decimal", title: "decimal", description: "Enter decimal", required: true
            input name: "time", type: "time", title: "time", description: "Enter time", required: true
            input name: "options", type: "enum", title: "enum", options: ["Option 1", "Option 2"], description: "Enter enum", required: true
        }
    }

    def someCommand() {
        log.debug "email: $email"
        log.debug "text: $text"
        log.debug "bool: $bool"
        log.debug "password: $password"
        log.debug "phone: $phone"
        log.debug "decimal: $decimal"
        log.debug "time: $time"
        log.debug "options: $options"
    }

----

추가 사항
----------------
-입력에 대한 기본 값으로 (``defaultValue: "foobar"``) 을 설정하면 모바일 앱에서 선택이 가능하지만 사용자는 여전히 해당 필드에 데이터를 입력해야 합니다. 혼란을 피하기 위해 ``defaultValue`` 를 사용하지 않는 것이 좋다.
