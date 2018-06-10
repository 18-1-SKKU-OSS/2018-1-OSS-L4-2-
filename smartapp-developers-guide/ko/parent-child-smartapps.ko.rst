.. _parent_child_smartapps:

부모-자식 SmartApp
======================

SmartApp은 자식 SmartApp을 가질 수 있습니다. 개별 디바이스에서 독립적으로 작동하는 여러 자동화를 제공하고 싶을 때 이 개념은 유용합니다. 이것은 한 부모에서 여러 개별 자동화를 통합합니다.

----

개요
--------

Smart Lighting은 부모-자식 SmartApp의 한 예제입니다.
Smart Lighting을 설치하면 하나의 부모 SmartApp(Smart Lighting)가 설치되고, 실제로 생성하는 각 고유한 자동화 조명은 자식 SmartApp의 새로운 인스턴스입니다.
이 자식 SmartApp은 실제로 각 자동화 조명을 제어합니다.

아래 도표는 이 관계를 보여줍니다.

.. image:: ../../img/smartapps/smart-lighting-diagram.png

부모 SmartApp과 그의 자식 SmartApp 사이의 관계는 일대다 관계입니다.
SmartApp은 여러 자식 SmartApp을 가질 수 있고, 이러한 자식은 또 자신의 자식 SmartApp을 가질 수 있습니다.
자식 SmartApp은 오직 하나의 부모 SmartApp만 가질 수 있습니다.

----

.. _parent_child_smartapp_parent:

부모 SmartApp
-------------

다른 SmartApp의 부모 SmartApp으로 정의하려면, ``preferences`` 의 ``app`` 입력 값을 사용하세요. ``app`` 입력 값은 사용자가 자식 SmartApp 인스턴스를 설치 및 편집하고, 부모와 자식 사이의 관계를 설정할 수 있도록 합니다.

.. code-block:: groovy

    preferences {
        page(name: "mainPage", title: "Child Apps", install: true, uninstall: true) {
            section {
                app(name: "childApps", appName: "Child App", namespace: "mynamespace", title: "New Child App", multiple: true)
            }
        }
    }

``app`` 입력 값을 통해 설치된 모든 자식 SmartApp은 부모 SmartApp의 기본 설정 페이지에 목록화되며, 사용자는 이 인스턴스를 편집하거나 삭제할 수 있습니다.

``app`` 입력 값의 옵션은 다음과 같습니다.

========= ===========
Option    Description
========= ===========
name      입력 값의 이름. 입력 값의 식별자 역할을 합니다.
appName   자식의 메타데이터에 정의된 자식 SmartApp의 이름
namespace 자식의 메타데이터에 정의된 자식 SmartApp의 이름공간
title     사용자가 자식 SmartApp의 새로운 인스턴스를 설치하기 위해 누르는 버튼의 제목
multiple  ``true`` 일 경우, 사용자는 여러 자식 SmartApp을 설치할 수 있습니다. ``false`` 일 경우, 오직 하나의 자식 SmartApp만 설치됩니다. 기본 값은 ``false`` 입니다.
========= ===========

----

자식 SmartApp
-------------

자식으로 사용하고 싶은 SmartApp에서 자식 SmartApp의 ``definition`` 의 ``parent`` 옵션을 ``"namespace":"Parent App Name"`` 의 형태로 지정하세요.

.. code-block:: groovy

    definition(
        ...
        parent: "yourNameSpace:Parent App Name",
        ...
    )

.. note::

	자식 SmartApp을 저장하면 플랫폼은 ``parent`` 옵션에서 지정된 이름공간과 이름이 존재하는지 확인합니다. 존재하지 않으면 에러가 발생합니다.

	부모 SmartApp이 저장되어있는지 먼저 확인하거나 부모 SmartApp을 저장한 후에 돌아와서 ``parent`` 옵션을 추가하시길 바랍니다.

----

부모와 자식 사이 의사소통
-----------------------

부모와 자식 SmartApp은 서로 소통해야할 때가 있습니다. 부모 SmartApp에서 ``getChildeApps()`` 메소드를 사용해 자식 SmartApp을 얻을 수 있습니다.

.. code-block:: groovy

    def children = getChildApps()
    log.debug "$children.size() child apps installed"
    children.each { child ->
        log.debug "Child app id: $child.id"
    }

이후 자식 SmartApp에서 바로 메소드를 호출할 수 있습니다.

.. code-block:: groovy

    // 자식 SmartApp이 foo() 메소드를 정의했다 가정합니다
    child.foo()

SmartApp의 이름으로 특정 자식 SmartApp을 찾기 위해 ``findChildAppByName()`` 메소드를 사용할 수 있습니다.

.. code-block:: groovy

    def theChild = findChildAppByName("My Child App")

자식 SmartApp은 자신의 ``parent`` 속성을 사용해 부모 SmartApp과 소통할 수 있습니다.

.. code-block:: groovy

    // 부모 SmartApp이 bar() 메소드를 정의했다 가정합니다
    parent.bar()

----

둘 이상의 부모 인스턴스 방지하기
-----------------------------

사용자가 자신의 위치에 둘 이상의 부모 SmartApp을 설치하는 것을 방지하려면, definition에 ``singleInstance: true`` 을 지정해줄 수 있습니다.

.. code-block:: groovy

    definition(
        ...
        singleInstance: true
        ...
    )

``singleInstance: true`` 을 지정해주면 사용자가 이미 설치되어 있는 부모 SmartApp을 설치하려 할 때 기존의 설치된 앱으로 이동합니다.
그곳에서 기존 자식 SmartApp을 설정하거나 새로운 자식 SmartApp을 추가할 수 있습니다.
이 방법은 부모 SmartApp이 하나만 필요할 때 여러 인스턴스를 생성되는 것을 방지할 수 있습니다.

-----

예제
----

아래는 부모 SmartApp("Simple Lighting")이 여러 자식 SmartApp("Simple Automations")을 만드는 방법을 보여주는 간단한 예제입니다.

부모 SmartApp은 다음과 같습니다.

.. code-block:: groovy

    definition(
        name: "Simple Lighting",
        namespace: "mynamespace/parent",
        author: "Your Name",
        description: "An example of parent/child SmartApps (this is the parent).",
        category: "My Apps",
        iconUrl: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience.png",
        iconX2Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png",
        iconX3Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png")


    preferences {
    	// 부모 앱의 기본 설정은 상당히 간단합니다. 자식 앱을 app 입력 값으로 설정합니다.
        page(name: "mainPage", title: "Simple Automations", install: true, uninstall: true,submitOnChange: true) {
            section {
                app(name: "simpleAutomation", appName: "Simple Automation", namespace: "mynamespace/automations", title: "Create New Automation", multiple: true)
    		}
    	}
    }

    def installed() {
    	log.debug "Installed with settings: ${settings}"
    	initialize()
    }

    def updated() {
    	log.debug "Updated with settings: ${settings}"
    	unsubscribe()
    	initialize()
    }

    def initialize() {
    	// 자식 앱이 기본 설정 및 구독을 처리하기 때문에 이곳에서는 처리할 일이 없습니다.
        // 데모/정보 목적으로 메세지를 기록합니다.
        log.debug "there are ${childApps.size()} child smartapps"
        childApps.each {child ->
            log.debug "child app: ${child.label}"
        }
    }

다음은 자식 SmartApp입니다.

.. code-block:: groovy

    definition(
        name: "Simple Automation",
        namespace: "mynamespace/automations",
        author: "Your Name",
        description: "A simple app to control basic lighting automations. This is a child app.",
        category: "My Apps",

        // parent 옵션은 부모 앱을 <이름공간>/<앱 이름> 형태로 지정할 수 있습니다.
        parent: "mynamespace/parent:Simple Lighting",
        iconUrl: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience.png",
        iconX2Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png",
        iconX3Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png")


    preferences {
    	page name: "mainPage", title: "Automate Lights & Switches", install: false, uninstall: true, nextPage: "namePage"
    	page name: "namePage", title: "Automate Lights & Switches", install: true, uninstall: true
    }

    def installed() {
        log.debug "Installed with settings: ${settings}"
        initialize()
    }

    def updated() {
        log.debug "Updated with settings: ${settings}"
        unschedule()
        initialize()
    }

    def initialize() {
    	// 사용자가 이름을 오버라이드하지 않는 경우, 이름을 기본 값으로 설정합니다
    	if (!overrideLabel) {
            app.updateLabel(defaultLabel())
    	}
    	// 핸들러 켜짐 및 꺼짐을 예약합니다
    	schedule(turnOnTime, turnOnHandler)
        schedule(turnOffTime, turnOffHandler)
    }

    // 메인페이지에서 조명, 작업, 커짐 및 꺼짐 시간을 선택합니다
    def mainPage() {
        dynamicPage(name: "mainPage") {
            section {
                lightInputs()
                actionInputs()
            }
            timeInputs()
    	}
    }

    // 사용자가 자동화에 이름을 지정할 수 있는 페이지
    def namePage() {
        if (!overrideLabel) {
            // 사용자가 이름을 바꾸지 않은 경우, 기본 이름으로 설정합니다
            def l = defaultLabel()
            log.debug "will set default label of $l"
            app.updateLabel(l)
    	}
        dynamicPage(name: "namePage") {
            if (overrideLabel) {
                section("Automation name") {
                    label title: "Enter custom name", defaultValue: app.label, required: false
                }
            } else {
                section("Automation name") {
                    paragraph app.label
                }
            }
            section {
                input "overrideLabel", "bool", title: "Edit automation name", defaultValue: "false", required: "false", submitOnChange: true
            }
        }
    }

    // 조명을 선택하는 입력 값
    def lightInputs() {
        input "lights", "capability.switch", title: "Which lights do you want to control?", multiple: true, submitOnChange: true
    }

    // 조명에 대한 작업(조명을 키고 색을 지정, 조명을 키고 밝기를 조절 등)을 관리하는 입력 값
    def actionInputs() {
        if (lights) {
            input "action", "enum", title: "What do you want to do?", options: actionOptions(), required: true, submitOnChange: true
            if (action == "color") {
                input "color", "enum", title: "Color", required: true, multiple:false, options: [
                    ["Soft White":"Soft White - Default"],
                    ["White":"White - Concentrate"],
                    ["Daylight":"Daylight - Energize"],
                    ["Warm White":"Warm White - Relax"],
                    "Red","Green","Blue","Yellow","Orange","Purple","Pink"]

            }
            if (action == "level" || action == "color") {
                input "level", "enum", title: "Dimmer Level", options: [[10:"10%"],[20:"20%"],[30:"30%"],[40:"40%"],[50:"50%"],[60:"60%"],[70:"70%"],[80:"80%"],[90:"90%"],[100:"100%"]], defaultValue: "80"
            }
        }
    }

    // 선택된 스위치에 대해 사용가능한 작업의 지도를 가져오는 메소드
    def actionMap() {
        def map = [on: "Turn On", off: "Turn Off"]
        if (lights.find{it.hasCommand('setLevel')} != null) {
            map.level = "Turn On & Set Level"
        }
        if (lights.find{it.hasCommand('setColor')} != null) {
            map.color = "Turn On & Set Color"
        }
        map
    }

    // 작업 지도의 목록을 입력을 위해 지도로 모으는 메소드
    def actionOptions() {
        actionMap().collect{[(it.key): it.value]}
    }

    // 켜기 및 끄기 시간을 선택하는 입력 값
    def timeInputs() {
        if (settings.action) {
            section {
                input "turnOnTime", "time", title: "Time to turn lights on", required: true
                input "turnOffTime", "time", title: "Time to turn lights off", required: true
            }
        }
    }

    // 자동화 이름을 기본 값으로 설정하는 메소드
    // 선택되었던 조명과 작업을 사용하여 자동화 이름을 생성합니다
    def defaultLabel() {
        def lightsLabel = settings.lights.size() == 1 ? lights[0].displayName : lights[0].displayName + ", etc..."

        if (action == "color") {
            "Turn on and set color of $lightsLabel"
        } else if (action == "level") {
            "Turn on and set level of $lightsLabel"
        } else {
            "Turn $action $lightsLabel"
        }
    }

    // 조명을 켜고 지정된 경우 밝기와 색을 설정하는 메소드
    def turnOnHandler() {
        // 선택된 작업에 따라 실행합니다
        switch(action) {
            case "level":
                lights.each {
                    // 스위치가 setLevel 명령어를 가지고 있는지 확인합니다
                    if (it.hasCommand('setLevel')) {
                        log.debug("Not So Smart Lighting: $it.displayName setLevel($level)")
                        it.setLevel(level as Integer)
                    }
                    it.on()
                }
                break
            case "on":
                log.debug "on()"
                lights.on()
                break
            case "color":
                setColor()
                break
            }
    }

    // 사용자가 색을 지정한 경우, 지정된대로 색과 밝기를 설정합니다
    def setColor() {

    	def hueColor = 0
    	def saturation = 100

    	switch(color) {
    		case "White":
                hueColor = 52
                saturation = 19
                break;
            case "Daylight":
                hueColor = 53
                saturation = 91
                break;
            case "Soft White":
                hueColor = 23
                saturation = 56
                break;
            case "Warm White":
                hueColor = 20
                saturation = 80
                break;
            case "Blue":
                hueColor = 70
                break;
            case "Green":
                hueColor = 39
                break;
            case "Yellow":
                hueColor = 25
                break;
            case "Orange":
                hueColor = 10
                break;
            case "Purple":
                hueColor = 75
                break;
            case "Pink":
                hueColor = 83
                break;
            case "Red":
                hueColor = 100
                break;
    	}

    	def value = [switch: "on", hue: hueColor, saturation: saturation, level: level as Integer ?: 100]
    	log.debug "color = $value"

    	lights.each {
            if (it.hasCommand('setColor')) {
                log.debug "$it.displayName, setColor($value)"
                it.setColor(value)
            } else if (it.hasCommand('setLevel')) {
                log.debug "$it.displayName, setLevel($value)"
                it.setLevel(level as Integer ?: 100)
            } else {
                log.debug "$it.displayName, on()"
                it.on()
            }
    	}
    }

    // 조명 처리기를 간단하게 끕니다
    def turnOffHandler() {
    	lights.off()
    }

위 코드를 사용하여 부모와 자식 SmartApp을 만들어보고, 부모 SmartApp을 게시해보시길 바랍니다. (부모에 의해 발견되고 마켓플레이스에서 개별적으로 설치하고 싶지 않으실테니 자식 SmartApp는 게시할 필요 없습니다.)
그 후 마켓플레이스에서 "Simple Lighting"을 "내 앱"에 설치하십시오. 그러면 자식 SmartApp("Simple Automation")의 인스턴스가 될 자동화를 여러 개 추가할 수 있습니다.

----

