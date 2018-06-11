=========================
SmartApp의 구조와 생애 주기
=========================

SmartApp은 사용자가 디바이스의 기능을 이용하여 삶을 자동화하도록 하는 응용프로그램입니다. 사용자는 SmartThings 모바일 응용프로그램을 통해 SmartApp을 설치할 수 있습니다. 또한, 몇몇 SmartApp은 SmartThings 시스템에 미리 설치되어 즉시 사용할 수 있습니다.

----

SmartApp의 종류
---------------

일반적으로 *Event-Handlers*, *Solution Modules*, 그리고 *Service Manager* 라는 세 가지 종류의 SmartApp이 있습니다.
백엔드 웹 개발이 익숙하다면 SmartApp을 충분히 개발할 수 있을 것입니다.

Event Handler SmartApps
^^^^^^^^^^^^^^^^^^^^^^^

이 응용프로그램은 커뮤니티에서 개발한 가장 일반적인 앱입니다. 이 응용프로그램을 통해 디바이스의 이벤트 발생 정보를 받고, 해당 이벤트가 시작될 때 핸들러 메소드를 호출할 수 있습니다. 이 메소드로 다양한 작업을 할 수 있으며, 가장 일반적으로 다른 디바이스에 명령을 내릴 수 있습니다.

Event-Handler SmartApp의 아주 간단한 예는 누군가 문을 통과하면 자동으로 조명을 켜도록 하는 것입니다.

Solution Module SmartApps
^^^^^^^^^^^^^^^^^^^^^^^^^

이 응용프로그램은 SmartThings 앱 인터페이스의 대시보드에 있으며, 다른 SmartApp의 컨테이너입니다. Solution Module SmartApp는 실생활에서 함께 사용되는 SmartApp과 결합하는 것입니다. 예를 들어 대시보드의 "Home & Family" 부분을 통해 사용자가 가족의 출입을 확인할 수 있습니다.

Service Manager SmartApps
^^^^^^^^^^^^^^^^^^^^^^^^^

이 응용프로그램은 LAN이나 Sonos, WeMo 디바이스와 같은 클라우드 기기에 접속하는 데에 사용됩니다. 이 SmartApp은 LAN 또는 클라우드 기기의 고유 프로토콜과 해당 기기에 대해 사용자가 만들어내는 Device Handler 사이의 연결 고리입니다. 이러한 Service Manager SmartApp은 LAN 또는 클라우드 기기를 검색한 후 연결을 계속 유지합니다.

사용자가 LAN 또는 클라우드를 이용하는 디바이스를 사용하는 경우, Service Manager SmartApp를 반드시 설치해야 합니다. 예를 들어, Sonos 기기와 페어링할 때에는 Sonos Service Manager SmartApp를 설치해야 합니다. 

----

SmartApp 구조
-------------

SmartApp은 `Groovy <http://groovy.codehaus.org/>`__ 스크립트의 형식을 가집니다. 
일반적인 SmartApp 스크립트는 *Definition(정의)*, *Preferences(기본 설정)*, *Predefined Callbacks(미리 정의된 콜백)* 및 *Event Handlers(이벤트 처리기)* 의 4개의 영역으로 구성됩니다. 
또한 클라우드와 연결되는 SmartApp에 필요한 매핑 영역도 있으며, 이는 추후에 설명하겠습니다.

.. image:: ../../img/smartapps/demo-app.png
    :class: with-border

Definition(정의)
^^^^^^^^^^^^^^^^
*정의* 영역에서는 앱의 이름과 앱을 식별하고 설명하는 정보를 지정합니다.

Preferences(기본 설정)
^^^^^^^^^^^^^^^^^^^^^
*기본 설정* 영역에서는 SmartApp이 설치 및 업데이트될 때 모바일 앱에 나타나는 화면을 정의합니다. 이 화면에서 사용자는 디바이스 동작에 영향을 미치는 구성 옵션과 SmartApp과 상호작용할 디바이스를 지정합니다.


Pre-defined callbacks(미리 정의된 콜백)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

다음 메소드가 존재하는 경우, SmartApp의 생명주기 동안 다양한 순간에 자동으로 메소드가 호출됩니다:

#. ``installed()`` - SmartApp이 처음 설치되었을 때 호출됩니다.
#. ``updated()`` - 설치된 SmartApp의 기본 설정이 업데이트 되었을 때 호출됩니다.
#. ``uninstalled()`` - SmartApp이 삭제되었을 때 호출됩니다.
#. ``childUninstalled()`` - 하위 앱이 삭제되었을 때 상위 앱에 호출됩니다. (SmartApp은 여러 개의 하위 SmartApp을 가질 수 있습니다.)

``installed()`` 와 ``updated()`` 메소드는 일반적으로 모든 앱에 존재합니다. 
앱이 업데이트 될 때 선택된 디바이스가 바뀌었을 수도 있으므로 일반적으로 이 두 메소드는 동일한 이벤트 구독을 설정합니다. 따라서 ``initialize()`` 메소드 안에서 이 메소드들을 호출하도록 하고, 설치 및 업데이트된 메소드에서 ``initialize()`` 를 호출하는 것이 일반적인 관행입니다.

SmartApp이 삭제되면 시스템이 자동으로 구독 및 스케쥴 정보를 삭제하기 때문에 ``uninstalled()`` 메소드는 보통 필요하지 않습니다. 
하지만 다른 시스템과 통합되고 이 시스템에 대해 정리를 수행해야 하는 앱에서는 ``uninstalled()`` 메소드가 필요할 수 있습니다.

Event Handlers(이벤트 처리기)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SmartApp의 나머지 영역에는 이벤트 구독에서 지정된 이벤트 처리기 메소드와 SmartApp을 구현하는 데에 필요한 다른 모든 메소드가 포함되어 있습니다. 이벤트 처리기 메소드는 :참고:`event_ref` 객체를 포함하는 하나의 인자를 가져야 합니다.

----

SmartApp 실행
-------------

SmartApp이 항상 실행되고 있는 것은 아닙니다. 외부 이벤트가 발생할 때 다양한 메소드가 실행됩니다. SmartApp은 다음 유형의 이벤트가 발생할 때 실행됩니다.

1. **미리 정의된 콜백** - 위에서 기술된 미리 정의된 생명주기 동안의 이벤트 중 하나가 발생합니다.
2. **디바이스 상태 변경** - 디바이스에서 속성이 변경되어 이벤트를 생성하고, 이는 SmartApp에서 처리기 메소드를 호출하는 구독을 야기합니다.
3. **위치 상태 변경** - *Mode* 와 같은 위치 속성이 변경됩니다. *Sunrise* 와 *sunset* 은 위치 이벤트의 다른 예입니다.
4. **앱에서의 사용자 행동** - 사용자가 모바일 앱 UI에서 SmartApp의 아이콘 또는 바로 가기를 누릅니다.
5. **예정된 이벤트** - runIn()과 같은 메소드를 사용해서 사용자가 특정 시간에 SmartApp에 있는 메소드를 호출합니다.
6. **웹 서비스 호출** `웹 서비스 API <../../smartapp-web-services-developers-guide/overview.ko.rst>`__ 를 이용해 사용자는 SmartApp 내의 메소드를 호출하는 웹을 통해 접근할 수 있는 엔드 포인트를 만듭니다.

----

디바이스 기본설정
----------------

기본설정에서 가장 일반적인 입력 값은 SmartApp에서 작동할 디바이스 종류를 명시하는 값입니다. 예를 들어, 앱이 하나의 접촉 센서가 필요함을 명시하려면 다음과 같이 작성하세요.

.. code-block:: groovy

    input "contact1", "capability.contactSensor"

위 코드는 모바일 UI에서 하나의 접촉 센서를 선택하라는 입력 요소(``capability.contactSensor``)를 생성합니다. 
``contact1``은 SmartApp에서 디바이스에 대한 접근을 제공하는 변수의 이름입니다.

디바이스 입력 값은 둘 이상일 수 있습니다. 하나 이상의 스위치를 선택하도록 하기 위해선 다음과 같이 작성하세요.

.. code-block:: groovy

    input "switch1", "capability.switch", multiple: true

`여기 <preferences-and-settings.ko.rst>`__ 에서 SmartApp 기본설정에 관한 더 많은 정보를 얻으실 수 있습니다.

----

이벤트 구독
----------

구독을 통해 SmartApp에서 디바이스, 장소, 또는 모바일 UI의 SmartApp 타일로부터의 이벤트를 알 수 있습니다. 디바이스 구독은 가장 일반적이고, 다음과 같은 형식을 이용합니다.

.. code-block:: groovy

    subscribe(<device>, "<attribute[.value]>", handlerMethod)

예를 들어, 접촉 센서로부터 모든 이벤트를 구독하려면 다음과 같이 작성하세요.

.. code-block:: groovy

    subscribe(contact1, "contact", contactHandler)

``contactHandler()`` 메소드는 접촉 센서가 열리거나 닫 때마다 호출됩니다.
또한 특정 이벤트 값만 구독할 수 있어, 접촉 센서가 열릴 때에만 처리기를 호출하려면 다음과 같이 작성하세요.

.. code-block:: groovy

    subscribe(contact1, "contact.open", contactOpenHandler)


``subscribe()`` 메소드는 디바이 또는 디바이스의 목록을 허용하므로, 입력 기본설정에 ``multiple: true``를 명시하면 목록의 각 디바이스에 대해서 반복해서 명시하지 않아도 됩니다.

:참고:`events_and_subscriptions` 에서 디바이스 이벤트 구독에 대한 더 많은 정보를 얻으실 수 있습니다.

----

SmartApp 샌드박싱
----------------

SmartApp은 샌드박스 환경에서 개발됩니다. 샌드박스는 성능과 보안을 위해 개발자를 Groovy 언어의 특정 하위 집합으로 제한하는 방법입니다. SmartApp은 :참고:`documented <groovy-for-smartthings>` 을 주된 방법으로 하고 있으며, 다른 개발자도 이 방법을 따라야합니다.

----

실행 위치
--------

이전 버젼의 SmartThings Hub를 사용하는 경우, 모든 SmartApp이 SmartThings 크라우드에서 실행됩니다. 새로운 버젼의 SmartThings Hub를 사용하는 경우, 어떤 SmartApp은 로컬로 실행될 수 있습니다. 실행 위치는 다양한 요인에 따라 다르며, SmartThings 내부 팀이 관리합니다.

SmartThings 개발자는 앱의 실행 위치와 관계없이 특정 사용 경우를 충족시키도록 SmartApp을 개발해야합니다. 현재에는 실행 위치를 지정하거나 강제 적용하는 방법이 없습니다.
