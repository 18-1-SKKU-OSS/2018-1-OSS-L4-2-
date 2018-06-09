예제: Bon Voyage
======================

SmartApp 작성 시 중요한 몇몇 개념을 설명하는 데 도움이 되도록 예제를 살펴봅시다.

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
SmartApp에 알림을 보내는 것에 대한 자세한 정보는 :참고:`smartapp_sending_notifications` 에서 읽으실 수 있습니다.

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
앱을 디버깅하거나 문제를 해결하려 할 때 매우 중요합니다!)

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

    // 지정된 모든 센서가 존재를 감지하지 못 한 경우, 참을 반환합니다
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

좀 더 자세히 살펴봅시다.

제일 먼저 어떤 이벤트가 발생되었는지 확인해봅시다.
이벤트 처리기에 전달된 ``evt`` 변수를 검사하여 확인합니다.
감지 기능은 ``"present"`` 또는 ``"not present"`` 둘 중 하나일 수 있습니다.

다음으로, 현재 모드가 바꾸고 싶은 모드로 이미 설정되어 있지 않은지 확인합니다. 이미 그 모드로 설정되어 있다면, 해야 할 작업은 없습니다!

이제 재밌어지기 시작합니다.

위의 예제에서 ``everyoneIsAway()`` 와 ``findFalseAlarmThreshold()`` 의 두가지 도우미 메소드를 정의했습니다.

``everyoneIsAway()`` 은 설정된 모든 센서에서 존재가 감지되지 않았을 경우 참을 반환하고, 그렇지 않으면 거짓을 반환합니다.
``people`` 변수에 설정되고 저장된 모든 센서에 대해 반복하고, ``currentPresence`` 속성을 확인합니다.
``currentPresence`` 가 ``"present"`` 일 경우, result를 거짓으로 설정하고 반복문을 종료합니다.
그 후 result 변수 값을 반환합니다.

``findFalseAlarmThreshold()`` 은 사용자로부터 분 단위로 설정된 시간 이내에 발생한 거짓 알림 임계값을 얻습니다.
임계값 설정이 아직 되지 않은 경우, 기본 값으로 10분을 반환합니다.

모든 사람이 떠났을 때, 내장된 :ref:`smartapp_run_in` 메소드를 호출합니다. 이 메소드는 지정된 시간 내에 ``takeAction()`` 메소드를 실행합니다. (곧 이 메소드를 정의할 것입니다.)
``findFalseAlarmThreshold()`` 에 60을 곱해 분 단위를 ``runIn()`` 메소드에서 요구하는 초 단위로 바꾸어 사용합니다.
``overwrite: false`` 를 명시해 이전에 예정된 ``takeAction()`` 호출을 덮어쓰지 않도록 합니다.
이 SmartApp의 맥락에서는 한 사용자가 떠난 후에 거짓 알림 임계값 시간 내에 또 다른 사용자가 떠나면 ``takeAction()`` 이 두 번 호출됩니다.
``overwrite`` 의 기본 값은 참이므로, 이전에 예정된 ``takeAction()`` 호출이 취소되고 최근 호출로 대체됩니다.

이제 ``takeAction()`` 메소드를 정의하겠습니다.

.. code-block:: groovy

    def takeAction() {
        if (everyoneIsAway()) {
            def threshold = 1000 * 60 * findFalseAlarmThreshold() - 1000
            def awayLongEnough = people.findAll { person ->
                def presenceState = person.currentState("presence")
                def elapsed = now() - presenceState.rawDateCreated.time
                elapsed >= threshold
            }
            log.debug "Found ${awayLongEnough.size()} out of ${people.size()} person(s) who were away long enough"
            if (awayLongEnough.size() == people.size()) {
                //def message = "${app.label} changed your mode to '${newMode}' because everyone left home"
                def message = "SmartThings changed your mode to '${newMode}' because everyone left home"
                log.info message
                send(message)
                setLocationMode(newMode)
            } else {
                log.debug "not everyone has been away long enough; doing nothing"
            }
        } else {
            log.debug "not everyone is away; doing nothing"
        }
    }

    private send(msg) {
        if ( sendPushMessage != "No" ) {
            log.debug( "sending push message" )
            sendPush( msg )
        }

        if ( phone ) {
            log.debug( "sending text message" )
            sendSms( phone, msg )
        }

        log.debug msg
    }

이 부분에서는 많은 작업이 수행되므로 흥미로운 몇 부분을 살펴보겠습니다.

가장 먼저 하는 작업은 모든 사람이 사라졌는지 다시 확인하는 것입니다.
``falesAlarmThreshold()`` 로 인해 이미 호출된 이후 무언가 변했을 수 있으므로 이 작업은 필수적입니다.

모든 사람이 사라졌을 경우, 거짓 알림 임계값을 이용해 얼마나 많은 사람이 충분히 오래 떠나있는지 알아야 합니다.
``awayLongEnough`` 라는 변수를 생성하고, Groovy의 `findAll() <http://docs.groovy-lang.org/latest/html/groovy-jdk/java/util/Collection.html#findAll(groovy.lang.Closure)>`__ 메소드를 통해 설정합니다.
``findAll()`` 메소드는 전달된 클로저의 논리에 기반하여 집합 일부를 반환합니다.
각 사람에 대해 사용할 수 있는 :참고:`device_current_state` 메소드를 이용해 이벤트가 발생된 이후 경과된 시간을 가져옵니다.
이벤트가 임계값을 초과한 이후 시간이 경과되었다면, 클로저에서 ``true`` 를 반환하여 ``awayLongEnough`` 집합에 추가합니다. (Groovy에선 묵시적이기 때문에 "return" 키워드를 생략할 수 있습니다)

``findAll()`` 메소드 또는 Groovy가 클로저를 사용하는 방법에 대한 더 많은 정보는 http://www.groovy-lang.org/documentation.html 에 있는 Groovy 문서를 참고하시길 바랍니다.

충분히 오래 나가있는 사람의 수가 이 앱에서 설정된 사람의 총 수와 동일한 경우, 메세지(다음에 이 메소드를 살펴볼 것입니다.)를 보내고 원하는 모드로 :참고:`smartapp_set_location_mode` 메소드를 호출합니다.
이를 통해 모드를 변경합니다.

``send()`` 메소드는 ``msg`` 라는 전송할 메세지인 문자열 매개변수를 취합니다.
여기서 앱이 사용자가 지정한 연락처로 알림을 보냅니다.

마지막으로 사용자가 기본 설정을 바꿀때마다 호출되는 ``update()`` 메소드를 작성하겠습니다.
이 메소드가 호출되었을 때, 효율적으로 앱을 재설정하기 위해 ``unsubscribe()`` 메소드를 호출한 후 ``subscribe()`` 을 호출해야 합니다.

.. code-block:: groovy

    def updated() {
        log.debug "Updated with settings: ${settings}"
        log.debug "Current mode = ${location.mode}, people = ${people.collect{it.label + ': ' + it.currentPresence}}"
        unsubscribe()
        subscribe(people, "presence", presence)
    }

----

관련된 문서
----------

- :참고:`prefs_and_settings`
- :참고:`events_and_subscriptions`
- :참고:`smartapp_working_with_devices`
- :참고:`modes`
- :참고:`smartapp-scheduling`
- :참고:`smartapp_sending_notifications`

----

완전한 소스코드
--------------

이 SmartApp의 완전한 소스코드는 SmartThingsPublic GitHub 저장소 `여기 <https://github.com/SmartThingsCommunity/SmartThingsPublic/blob/master/smartapps/smartthings/bon-voyage.src/bon-voyage.groovy>`__ 에서 찾으실 수 있습니다.

