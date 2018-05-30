.. _modes:

=====
모드
=====

SmartThings를 사용하면 사용자는 특정 *모드*에서만 SmartApp이 실행될 수 있도록 지정할 수 있습니다.

----

개요
--------

모드는 스마트 홈을 위한 행동 필터로 여겨질 수 있습니다. 사용자는 사용하는 모드에 따라 어떻게 동작할지 바꿀 수 있습니다.

- "홈" 모드일 때, 조명이 켜지도록 한다.
- "부재 중" 모드일 때, 문자를 보내고 알람이 켜지도록 한다.

SmartThings는 "홈", "부재 중", "밤"과 같은 몇 가지 사전 구성 모드가 제공됩니다.
사용자는 각 장소에 대해 자신만의 모드를 만들 수 있습니다.

----

현재 모드 가져오기
----------------

SmartApp의 ``위치``에 있는 ``모드`` 또는 ``현재 모드`` 속성을 이용해 현재 모드를 가져올 수 있습니다.

.. code-block:: groovy

    def currMode = location.mode // "Home", "Away", etc.
    log.debug "current mode is $currMode"

    def anotherWay = location.currentMode
    log.debug "current mode is $anotherWay"

----

모든 모드 가져오기
-----------------

SmartApp이 설치된 위치에 대한 모든 모드 리스트를 가져올 수 있습니다.

.. code-block:: groovy

    def allModes = location.modes // ex: [Home, Away, Night]
    log.debug "all modes for this location: $allModes"

----

모드 설정
--------

``setLocationMode()`` 또는 ``location.setMode()``를 이용해 위치에 대한 모드를 설정할 수 있습니다.:

.. code-block:: groovy

    setLocationMode("Away")

.. code-block:: groovy

    location.setMode("Away")

이러한 메소드는 지정된 모드가 위치에 대해 존재하지 않을 경우 오류를 발생시킵니다.

----

사용자가 모드 선택하도록 하기
Allowing users to select Modes
------------------------------

SmartApp의 ``기본 설정`` 창에서 ``"mode"`` 입력창을 이용해 사용자가 모드를 선택할 수 있도록 지정할 수 있습니다.

.. code-block:: groovy

  input "modes", "mode", title: "select a mode(s)", multiple: true

위 코드는 사용자가 하나의 모드 (또는 여러 개의 모드)를 선택할 수 있도록 하고, SmartApp은 선택된 모드에 따라 동작을 달리할 수 있습니다.

``mode()`` 메소드를 이용해 사용자가 SmartApp이 실행할 모드를 선택할 수 있도록 할 수도 있습니다.


 .. code-block:: groovy

    mode(title: "Set for specific mode(s)")

개발자가 올바른 모드를 결정하는 데 필요한 작업 없이도 SmartApp은 선택된 모드에서만 실행됩니다.

사용자가 모드를 선택할 수 있도록 하는 다양한 방법에 대한 자세한 정보는 `here <mode_pref>`에서 확인할 수 있습니다.

----

모드 이벤트
-----------

``location`` 객체의 ``mode``를 구독함으로써 모드 변경에 대해 알림받을 수 있습니다.

.. code-block:: groovy

    def installed() {
        subscribe(location, "mode", modeChangeHandler)
    }

    def modeChangeHandler(evt) {
        log.debug "mode changed to ${evt.value}"
    }

위 예제에서 ``modeChangeHandler()``는 이 SmartApp이 설치된 위치에 대한 모드가 바뀔 때마다 호출됩니다.

----

예제
-------

다음 예제는 "예약 모드 변경" SmartApp의 단순화된 버전입니다. 전체 예제의 SmartApp은 IDE템플릿에서 볼 수 있습니다.

이 예제는 사용자가 모드를 선택하는 ``"mode"`` 입력창 사용법을 보여주고, (사용자가 정의한 예약에 따라) 지정된대로 모드를 바꿉니다.

.. code-block:: groovy

    preferences {
        section("At this time every day") {
		      input "time", "time", title: "Time of Day"
	    }
        section("Change to this mode") {
            input "newMode", "mode", title: "Mode?"
        }
    }

    def installed() {
        initialize()
    }

    def updated() {
        unschedule()
        initialize()
    }

    def initialize() {
        schedule(time, changeMode)
    }

    def changeMode() {
        log.debug "changeMode, location.mode = $location.mode, newMode = $newMode, location.modes = $location.modes"

        if (location.mode != newMode) {
            if (location.modes?.find{it.name == newMode}) {
                setLocationMode(newMode)
            }  else {
                log.warn "Tried to change to undefined mode '${newMode}'"
            }
        }
    }

위의 ``changeMode()`` 메소드에서 몇 가지 언급해야 할 점이 있습니다.

먼저, 이미 지정된 모드인지를 확인합니다. 만약 이미 지정된 모드라면, 아무 작업도 수행하지 않습니다.

.. code-block:: groovy

    if (location.mode != newMode)

모드를 변경해야 한다면, 먼저 모드가 실제로 존재하는지를 입증합니다.
이 작업은 현재 위치에서 존재하지 않는 모드에 대해 설정 하지 않도록 합니다.

.. code-block:: groovy

    if (location.modes?.find{it.name == newMode})

----

추가 참고 목록
-------------

- :ref:`Mode Input <mode_pref>`
- :ref:`Location Object <location_ref>`
- :ref:`Mode Object <mode_ref>`
