.. _smartapp_app_touch:

=======
앱 터치
=======

SmartApp의 옆에 있는 *재생* 아이콘을 클릭해 사용자가 동의한 경우 몇 가지 작업을 수행할 수 있습니다.

예를 들어, 사용자가 재생 아이콘을 누르면 음성 알림 SmartApp이 메시지를 재생하도록 할 수 있습니다.

----

.. _subscribe_to_app:

``app`` 구독
------------

이 기능을 활성화시키려면 다음과 같이 작성하여 ``app``을 구독하세요.

.. code-block:: groovy

    def initialize() {
        subscribe(app, appHandler)
    }

    def appHandler(evt) {
        log.debug "app event ${evt.name}:${evt.value} received"
    }

이벤트를 구독하면 다음과 같이 모바일 앱에서 재생 아이콘이 표시됩니다.

.. image:: ../../img/smartapps/app-touch.png
    :scale: 60

앱 이벤트 처리기 메소드에서 이 터치 이벤트에 대해 필요한 조치를 취할 수 있습니다.
