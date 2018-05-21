martApp의 구조와 생애 주기
=========================

SmartApp은 사용자가 디바이스의 기능을 이용하여 삶을 자동화하도록 하는 응용프로그램입니다. 사용자는 SmartThings 모바일 응용프로그램을 통해 SmartApp을 설치할 수 있습니다. 또한, 몇몇 SmartApp은 SmartThings 시스템에 미리 설치되어 즉시 사용할 수 있습니다.

----

SmartApp의 종류
---------------

일반적으로, *Event-Handlers*, *Solution Modules*, 그리고 *Service Manager*라는 세 가지 종류의 SmartApp이 있습니다: . 백엔드 웹 개발이 익숙하다면 SmartApp을 충분히 개발할 수 있을 것입니다.

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

