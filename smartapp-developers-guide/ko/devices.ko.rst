.. _smartapp_working_with_devices:

============
디바이스 작동
============

SmartApp은 거의 항상 디바이스와 상호작용합니다.
특정 디바이스에 관한 정보를 얻거나(스위치가 켜져 있습니까?) 디바이스로 명령어(스위치 끄기)를 보내야 할 때가 있습니다.

----

디바이스 개요
------------

디바이스는 SmartApp와 상호작용하는 "물체"입니다.
디바이스는 하나 이상의 기능을 지원할 수도 있습니다.

기능은 디바이스가 아는 것(속성)과 디바이스가 할 수 있는 것(명령어)를 나타냅니다.
많은 제조업체의 디바이스와 투명하게 작동할 수 있도록 해주는 추상화입니다.

유연한 SmartApp을 만들기 위해서, SmartApp이 지정된 기능을 지원하는 어떠한 디바이스와도 작동할 수 있도록 개발해야 합니다.
예를 들어 특정 제조업체의 스위치와만 작동할 수 있는 SmartApp은 만들고 싶지 않을 겁니다.
스위치 기능을 지원하는 어떠한 디바이스와도 작동할 수 있는 앱을 만들고 싶을 겁니다.

----

기본 설정--디바이스 선택하기
-------------------------

사용자가 지정된 기능을 지원하는 디바이스를 선택할 수 있게 하기 위해선 preferences에서 input 요소를 사용합니다.

.. code-block:: groovy

    preferences {
        section {
            input "presenceSensors", "capability.presenceSensor"
        }
    }

위 예제에서는 사용자가 감지 센서 기능을 지원하는 디바이스를 선택할 수 있습니다.
휴대전화 또는 `SmartSense presence sensor <https://shop.smartthings.com/#!/products/smartsense-presence>`__ 을 선택할 수도 있습니다.
어떠한 디바이스든 상관 없습니다. 단지 감지 센서 기능을 지원하는 디바이스를 필요하다고 선언하면 됩니다.

지원되는 모든 기능에 대한 정보는 :참고:`capabilities_taxonomy` 를 참고하시길 바랍니다.
"기본 설정 참고" 열은 지정된 기능에 대해 기본 설정에서 사용해야 할 것을 알려줍니다.

----

디바이스와 상호작용하기
---------------------

SmartApp이 상호작용할 디바이스를 선언한 후에, Device 객체의 인스턴스는 지정했던 이름으로 SmartApp에서 사용할 수 있습니다.

.. code-block:: groovy

    preferences {
        section {
            input "theSwitch", "capability.switch"
        }
    }

    def someEventHandler(evt) {
        theSwitch.on()
    }

----

디바이스 속성
------------

속성은 디바이스의 상태를 나타냅니다. 예를 들어 "온도측정" 기능을 지원하는 디바이스는 "온도" 속성을 갖습니다.

속성은 상태를 갖습니다. "온도" 속성은 온도에 관련된 정보(값, 측정된 날짜 등)을 갖는 :참고:`state_ref` 객체와 연관되어 있습니다.

속성 데이터는 SmartThings 클라우드에 저장되고, 디바이스가 상태를 보고할 때 업데이트 됩니다.

----

디바이스 명령어
--------------

디바이스는 보통 하나 이상의 명령어를 받을 수 있습니다.
명령어는 디바이스가 할 수 있는 작업입니다.
스위치는 "on"과 "off" 명령어를 지원하고, 각각은 스위치를 "켜짐" 및 "꺼짐"으로 전환합니다.

모든 디바이스가 명령어를 받을 수 있는 건 아닙니다.
명령어는 일반적으로 일종의 물리적 작동(예를 들면 스위치를 켜기, 또는 잠금 해제)를 수행합니다.
예를 들어 습도 센서는 물리적으로 작동할 수 있는 작업이 없습니다.

----

디바이스의 현재 값 받기
---------------------

가장 최근 보고된 디바이스 상태 속성 정보는 두가지 방법으로 검색할 수 있습니다.

:참고:`device_current_state` 과 :참고:`device_attribute_state` 은 가장 최근 보고된 디바이스 상태를 캡슐화하는 :참고:`state_ref` 객체를 반환합니다.

.. code-block:: groovy

    preferences {
        section() {
            input "tempSensor", "capability.temperatureMeasurement"
        }
    }

    def someEventHandler(evt) {

        def currentState = tempSensor.currentState("temperature")
        log.debug "temperature value as a string: ${currentState.value}"
        log.debug "time this temperature record was created: ${currentState.date}"

        // 짧은 표기법 - 온도 측정 기능 지원
        // "온도" 속성에 "상태"를 덧붙입니다
        def anotherCurrentState = tempSensor.temperatureState
        log.debug "temperature value as an integer: ${anotherCurrentState.integerValue}"
    }

:참고:`device_latest_value`, :참고:`device_current_value` 및 :참고:`currentAttributeName` 은 가장 최근 보고된 속성 값을 반환합니다.
이들은 모두 같은 일을 하기 때문에 서로 구별없이 사용될 수 있습니다.

.. code-block:: groovy

    preferences {
        section() {
            input "myLock", "capability.lock"
        }
    }

    def someEventHandler(evt) {
        def currentValue = myLock.currentValue("lock")
        log.debug "the current value of myLock is $currentValue"

        def latestValue = myLock.latestValue("lock")
        log.debug "the latest value of myLock is $latestValue"

        // 잠금 기능은 "잠금" 속성을 갖습니다
        // <디바이스 이름>.current<대문자로 시작하는 속성 이름>:
        def anotherCurrentValue = myLock.currentLock
        log.debug "the current value of myLock using shortcut is: $anotherCurrentValue"
    }

.. important::

	속성 값의 현재 또는 가장 최신 상태는 *가장 최근에 디바이스가 SmartThings에 보고한 값* 입니다.
	이 값은 폴링 또는 디바이스와 직접 통신하여 계산될 수 없습니다.

	예를 들어, ``someDevice.currentValue('someAttribute')`` 은 지정된 속성에 대해 가장 최근 보고된 값을 받아옵니다.
	디바이스가 오작동했거나 SmartThings 허브가 오프라인일 경우에는 반환된 값이 디바이스의 물리적 상태와 일치하지 않을 수 있습니다.

----

이벤트 기록에 대한 쿼리
---------------------

이벤트 목록을 시간 역순으로(최신 이벤트를 먼저) 가져오려면 ``events()`` 메소드를 사용하시길 바랍니다.

.. code-block:: groovy

    // 기본 값으로 최신 10개의 이벤트를 반환합니다
    myDevice.events()

    // 더 많은 결과 값을 얻기 위해서 max 옵션을 사용합니다
    myDevice.events(max: 30)

지정한 날짜 이후로 이벤트 목록을 시간 역순으로(최신 이벤트를 먼저) 가져오려면 ``eventsSince`` 메소드를 사용하시길 바랍니다.

.. code-block:: groovy

    // 어제부터 이 디바이스의 모든 이벤트를 가져옵니다 (최대 1000개의 이벤트)
    myDevice.eventsSince(new Date() - 1)

    // 어제부터 최근 20개 이벤트를 가져옵니다
    myDevice.eventsSince(new Date() - 1, [max: 20])

두 날짜 사이의 이벤트 목록을 가져오려면 ``eventsBetween`` 메소드를 사용하시길 바랍니다.

.. code-block:: groovy

    // 2일 전부터 어제까지 모든 이벤트를 가져옵니다 (최대 1000개의 이벤트)
    // 시간 역순(최신 이벤트가 먼저)로 정렬하여 이벤트를 반환합니다 
    myDevice.eventsBetween(new Date() - 2, new Date() - 1)

    // 지난 주 최근 50개의 이벤트를 가져옵니다
    myDevice.eventsBetween(new Date() - 7, new Date(), [max: 50])

디바이스의 상태 정보를 가져오는 데에도 비슷한 날짜 제약 메소드가 있습니다.

더 많은 정보는 :참고:`device_ref` API 문서를 참고하시길 바랍니다.

----

명령어 보내기
------------

SmartApp에서 디바이스로 스위치를 켜거나 잠금 해제와 같은 명령어를 보내야할 때가 있습니다.

사용자의 디바이스에 사용할 수 있는 명령어는 디바이스마다 다양합니다.
주어진 기능에 사용할 수 있는 명령어를 알고 싶다면 :참고:`capabilities_taxonomy` 을 참고하실 수 있습니다.

명령어를 보내는 것은 디바이스에서 명령 메소드를 호출하는 것만큼 간단합니다.

.. code-block:: groovy

    myLock.lock()
    myLock.unlock()

몇몇 명령어는 매개변수가 필요합니다.
모든 명령어는 마지막 인자로 선택적인 지도 매개변수를 가질 수 있으며, 이를 통해 명령어가 디바이스로 전송되기 전까지 지연 시간을 밀리 초 단위로 지정할 수 있습니다.

.. code-block:: groovy

    // 명령어를 전송하기 전 2초동안 기다립니다
    mySwitch.on([delay: 2000])


.. note::

	특정 디바이스는 지원되는 기능보다 더 많은 명령어를 제공할 수 있기 때문에, 선언한 기능보다 더 많은 명령어를 사용할 수 있습니다.
	가장 좋은 방법으로는 SmartApp을 특정 디바이스가 아닌 기능 사양으로 작성하는 것입니다.
	하지만 매우 특정한 경우에 대해 SmartApp을 개발하고 유연성을 포기할 의향이 있다면, 이 기능을 사용해도 됩니다.

----

여러 디바이스와 상호작용하기
-------------------------

디바이스 기본 설정에서 ``multiple:true`` 를 지정한다면, 사용자는 둘 이상의 디바이스를 선택할 수 있습니다.
이 경우, 디바이스 인스턴스는 객체 목록을 참조합니다.

각 디바이스에 대해서 반복하지 않고, 모든 디바이스로 명령어를 보낼 수 있습니다.

.. code-block:: groovy

    preferences {
        section {
            input "switches", "capability.switch", multiple: true
        }
    }

    def someEventHandler(evt) {
        log.debug "will send the on() command to ${switches.size()} switches"
        switches.on()
    }

위에서 언급된 메소드를 이용해 여러 디바이스의 상태와 이벤트 기록을 검색할 수도 있습니다.
단일 값 또는 객체 대신에 해당 메소드는 값 또는 객체의 목록을 반환합니다.

다음은 모든 스위치의 상태 값을 얻고, 켜져 있는 스위치를 기록하는 간단한 예제입니다.

.. code-block:: groovy

    preferences {
        section {
            input "switches", "capability.switch", multiple: true
        }
    }

    def someEventHandler(evt) {
        // 모든 스위치 값의 목록을 반환합니다
        def currSwitches = switches.currentSwitch

        def onSwitches = currSwitches.findAll { switchVal ->
            switchVal == "on" ? true : false
        }

        log.debug "${onSwitches.size()} out of ${switches.size()} switches are on"    
    }

----

추가 참고 목록
-------------

 - :참고:`capabilities_taxonomy`
 - :참고:`prefs_and_settings`
 - :참고:`events_and_subscriptions`
 - :참고:`device_ref` API Documentation
 - :참고:`event_ref` API Documentation
 - :참고:`state_ref` API Documentation


.. _Preferences and Settings: :doc:`preferences-and-settings`

