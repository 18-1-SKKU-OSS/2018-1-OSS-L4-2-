예제: Bon Voyage
======================

SmartApp 작성시 중요한 몇몇 개념을 설명하는 데 도움이 되도록 예제를 살펴봅시다.

----

Bon Voyage
------------

이 예제 SmartApp은 매우 간단합니다. 이 앱은 감지기를 모니터하고 모든 사람이 사라졌을 때 모드를 변경합니다.

이 작업을 수행하기 위해 앱에서는 다음을 수행해야 합니다.

- 어떤 센서를 모니터할지, 어떤 모드를 작동할지 및 다른 앱 기본 설정 등의 필요한 입력값을 사용자로부터 수집합니다.

- 적절한 이벤트를 구독하고, 이벤트가 발생할 때 조치를 취합니다.

기본 설정부터 구성해보겠습니다.

----

SmartApp 기본 설정
-----------------

Bon Voyage 앱을 설정하려면 사용자로부터 다음 정보를 수집해야 합니다.

- 모니터할 센서
- 모든 사람이 사라졌을 때 작동할 모드
- 거짓 알람 임계값
- 누구에게 어떻게 알림을 줄지

각 입력값은 기본 설정의 section에 해당합니다.

.. code-block:: groovy

    preferences {
        section("When all of these people leave home") {
            input "people", "capability.presenceSensor", multiple: true
        }
        section("Change to this mode") {
            input "newMode", "mode", title: "Mode?"
        }
        section("False alarm threshold (defaults to 10 min)") {
            input "falseAlarmThreshold", "decimal", title: "Number of minutes", required: false
        }
        section( "Notifications" ) {
            input("recipients", "contact", title: "Send notifications to", required: false) {
                input "sendPushMessage", "enum", title: "Send a push notification?", options: ["Yes", "No"], required: false
                input "phone", "phone", title: "Send a Text Message?", required: false
            }
        }

    }

각 section에 대해 더 자세히 살펴보겠습니다.

*When all of these people leave home* section에서는 사용자가 이 앱에서 사용할 센서를 설정할 수 있습니다.
사용자는 기본 제목이 "When all of these people leave home"인 section을 볼 수 있습니다.
사용자가 원하는 센서를 선택할 수 있도록 드롭다운은 감지센서의 기능(``capability.presenceSensor``)을 가진 디바이스로 채워집니다.
``multiple: true``은 사용자가 원하는 만큼 센서를 추가할 수 있도록 합니다.
선택된 센서는 ``people``이라는 변수에 저장됩니다.

*Change to this mode* section에서는 모든 사람이 사라졌을 때 작동될 모드를 사용자가 지정할 수 있도록 합니다.
``mode``라는 입력 유형이 사용되므로 드롭다운은 사용자가 지정한 모든 모드로 채워집니다.
제목 속성은 ``"Mode?"``이라는 제목을 보여주는 데에 사용됩니다.
선택된 모드는 ``newMode``라는 변수에 저장됩니다.

*False alarm threshold (기본 값은 10분)* section에서는 사용자가 거짓 알림 임계값을 지정할 수 있습니다.
사용자가 분을 나타내는 숫자 값을 입력할 수 있도록 입력 필드 유형으로는 십진수가 사용됩니다.
"Number of minutes"라는 제목이 지정되었고, ``required`` 속성을 거짓으로 설정해놓았습니다.
모든 필드를 필수로 하는 게 기본 값이므로, 필드를 필요하지 않은 경우에 명시해야 합니다. 
나중에 사용하기 위해 사용자의 입력값을 ``falseAlarmThreshold``라는 이름의 변수에 저장합니다.

마지막으로 "Notifications"라는 section입니다.
이 section에서 사용자는 모든 사람이 사라졌을 때 어떻게 알림 받을지 설정할 수 있습니다.
이 입력 값은 약간 다릅니다. 이 입력 값은 ``contact``라는 특별한 입력 유형을 사용합니다.
SmartApp에 알림을 보내는 것에 대한 자세한 정보는 :ref:`smartapp_sending_notifications`에서 읽으실 수 있습니다.
