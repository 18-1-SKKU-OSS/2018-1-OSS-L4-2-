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

각 입력값은 기본 설정의 각 섹션에 해당합니다.

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

각 섹션에 대해 더 자세히 살펴보겠습니다.

*When all of these people leave home* 섹션에서는 사용자가 이 앱에서 사용할 센서를 설정할 수 있습니다.
사용자는 기본 제목이 "When all of these people leave home"인 섹션을 볼 수 있습니다.
사용자가 원하는 센서를 선택할 수 있도록 드롭다운은 감지센서의 기능( ``capability.presenceSensor`` )을 가진 디바이스로 채워집니다.
``multiple: true`` 은 사용자가 원하는 만큼 센서를 추가할 수 있도록 합니다.
선택된 센서는 ``people`` 이라는 변수에 저장됩니다.

*Change to this mode* 섹션에서는 모든 사람이 사라졌을 때 작동될 모드를 사용자가 지정할 수 있도록 합니다.
``mode`` 라는 입력 유형이 사용되므로 드롭다운은 사용자가 지정한 모든 모드로 채워집니다.
제목 속성은 ``"Mode?"`` 이라는 제목을 보여주는 데에 사용됩니다.
선택된 모드는 ``newMode`` 라는 변수에 저장됩니다.

*False alarm threshold (기본 값은 10분)* 섹션에서는 사용자가 거짓 알림 임계값을 지정할 수 있습니다.
사용자가 분을 나타내는 숫자 값을 입력할 수 있도록 입력 필드 유형으로는 십진수가 사용됩니다.
"Number of minutes"라는 제목이 지정되었고, ``required`` 속성을 거짓으로 설정해놓았습니다.
모든 필드를 필수로 하는 게 기본 값이므로, 필드를 필요하지 않은 경우에 명시해야 합니다. 
나중에 사용하기 위해 사용자의 입력값을 ``falseAlarmThreshold`` 라는 이름의 변수에 저장합니다.

마지막으로 "Notifications"라는 섹션입니다.
이 섹션에서 사용자는 모든 사람이 사라졌을 때 어떻게 알림 받을지 설정할 수 있습니다.
이 입력 값은 약간 다릅니다. 이 입력 값은 ``contact`` 라는 특별한 입력 유형을 사용합니다.
SmartApp에 알림을 보내는 것에 대한 자세한 정보는 :ref:`smartapp_sending_notifications` 에서 읽으실 수 있습니다.

----

모니터링과 반응
--------------

지금까지 사용자로부터 필요한 입력 값을 수집했으므로, 적절한 이벤트를 알림받고 이벤트가 발생할 때 조치를 취해야 합니다.

``installed()`` 메소드를 통해 이를 수행할 수 있습니다.

.. code-block:: groovy

    def installed() {
        log.debug "Installed with settings: ${settings}"
        log.debug "Current mode = ${location.mode}, people = ${people.collect{it.label + ': ' + it.currentPresence}}"
        subscribe(people, "presence", presence)
    }

설치가 끝나면, 사람들의 상태를 추적합니다. 
``subscribe()`` 메소드를 사용해 미리 정의된 감지 센서 집합, 즉 ``people`` 의 ``presence`` 속성을 알림받을 수 있습니다.
사람들의 존재 상태가 변하면, ``presence`` 메소드(위 예제에서 가장 마지막 매개변수)가 호출됩니다.

(또한 ``log`` 문장에 주의하시길 바랍니다. ``log`` 문장을 자세히 다루진 않을 것이나, 철저하게 기록을 제공하는 것은 SmartApp 개발자로서 유지해야 할 습관입니다.
앱을 디버깅하거나 문제를 해결하려할 때 매우 중요합니다!)

presence 메소드를 정의해봅시다.

.. code-block:: groovy

    def presence(evt) {
        log.debug "evt.name: $evt.value"
        if (evt.value == "not present") {
            if (location.mode != newMode) {
                log.debug "checking if everyone is away"
                if (everyoneIsAway()) {
                    log.debug "starting sequence"
                    runIn(findFalseAlarmThreshold() * 60, "takeAction", [overwrite: false])
                }
            }
            else {
                log.debug "mode is the same, not evaluating"
            }
        }
        else {
            log.debug "present; doing nothing"
        }
    }

    // 지정된 모든 센서가 존재를 감지하지 못 하는 경우, 참을 반환합니다
    // 그렇지 않으면 거짓을 반환합니다
    private everyoneIsAway() {
        def result = true
	// 기본 설정 메소드에서
        // 정의한 people 변수에 대해 반복합니다
        for (person in people) {
            if (person.currentPresence == "present") {
                // 누군가 존재하므로 result 변수를 거짓으로 설정하고 
                // 반복문을 종료합니다
                result = false
                break
            }
        }
        log.debug "everyoneIsAway: $result"
        return result
    }

    // 몇 분 이내에 거짓 알림 임계값을 받은 경우
    // 기본 설정에 정의되지 않은 경우, 기본 값은 10분입니다
    private findFalseAlarmThreshold() {
        // Groovy에서 반환문은 묵시적이며 필수가 아닙니다
        // 기본 설정에서 설정한 변수가 정의되어 있는지 확인하고,
        // 정의되어 있는 경우 그 변수를 반환합니다
        // 정의되지 않은 경우 기본 값인 10을 반환합니다
        (falseAlarmThreshold != null && falseAlarmThreshold != "") ? falseAlarmThreshold : 10
    }


