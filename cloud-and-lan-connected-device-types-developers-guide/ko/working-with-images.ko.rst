.. _working_with_camera_photos:

===================
카메라 사진 캡쳐 및 표시
===================

SmartThings에 연결된 카메라는 Carousel타일과 함께 ref:"imageCapture"기능을 사용하여 사진을 캡쳐하고 볼 수 있습니다.
SmartThings-연결 카메라는 LAN 또는 클라우드로 연결됩니다. 이 문서는 두 경우에서 이미지를 캡쳐하고 표시하는 단계를 설명합니다.

----

사진 캡쳐 기능
-----------

:ref:`imageCapture` 기능을 지원 하기 위하여는 장치 핸들러의 메타 데이터에 추가해야 합니다.

.. code-block:: groovy

    metadata {
        definition(name: "My Camera Device", namespace: "MyNamespace", author: "My Name") {
            capability "Image Capture"
            // other definition metadata...
        }
    }

이미지 캡쳐 기능은 하나의 속성인 "image"와 하나의 명령인 ``take()``를 정의합니다.
Carousel 타일을 사용하여 이미지를 표시하고 다음에 설명하는 대로 사진을 수동으로 찍을 수 있습니다.

----

사진 촬영 및 보기 위한 Tiles
-----------------------

이미지를 보고 캡쳐할 수 있도록 Tile을 추가합니다.

.. code-block:: groovy

    tiles {
        standardTile("image", "device.image", width: 1, height: 1, canChangeIcon: false, inactiveLabel: true, canChangeBackground: true) {
            state "default", label: "", action: "", icon: "st.camera.dropcam-centered", backgroundColor: "#FFFFFF"
        }

        carouselTile("cameraDetails", "device.image", width: 3, height: 2) { }

        standardTile("take", "device.image", width: 1, height: 1, canChangeIcon: false, inactiveLabel: true, canChangeBackground: false) {
            state "take", label: "Take", action: "Image Capture.take", icon: "st.camera.dropcam", backgroundColor: "#FFFFFF", nextState:"taking"
            state "taking", label:'Taking', action: "", icon: "st.camera.dropcam", backgroundColor: "#00A0DC"
            state "image", label: "Take", action: "Image Capture.take", icon: "st.camera.dropcam", backgroundColor: "#FFFFFF", nextState:"taking"
        }

        main "image"
        details(["cameraDetails", "take"])
    }

``carouselTile`` 은 이미지들이 전시될 곳이며 "take"타일은 사용자들이 이미지를 캡쳐할 수 있게 해 준다.
두 타일 모두 ``"image"`` 속성과 연관되어 있다는 점에 유의하십시오; 이 연결을 통해 이미지를 적절하게 찍고 표시할 수 있습니다.

LAN연결 카메라
^^^^^^^^^^^

LAN연결 장치들은 :ref:`hubaction_ref` 를 통해서 이미지를 캡쳐하고, :ref:`device_handler_ref_store_temp_image` 를 통해서 저장하며, :ref:`tiles_carousel_tile` 를 통해 표시 가능합니다.

``take()`` 명령은 사진 촬영을 ``HubAction`` 통해 요청합니다.
장치의 응답은 장치 핸들러의 ``parse()`` method로 전달 되며 그곳에서 ``storeTemporaryImage()`` 를 통해 장기-저장 저장소로 이동할 수 있습니다.
``storeTemporaryImage()`` 또한 "image" 이벤트를 발생 시켜 Carousel Tile이 새로운 이미지로 업데이트 됩니다.

다음은 ``take()`` 의 예시입니다 (요청에 대한 세부 정보는 각 장치별로 다릅니다):

.. code-block:: groovy

    def take() {
        def host = getHostAddress()
        def port = host.split(":")[1]

        def path = "/some/path/"

        def hubAction = new physicalgraph.device.HubAction(
            method: "GET",
            path: path,
            headers: [HOST:host]
        )

        hubAction.options = [outputMsgToS3:true]

        return hubAction
    }

    /**
    * Utility method to get the host addresses
    */
    private getHostAddress() {
        def parts = device.deviceNetworkId.split(":")
        def ip = convertHexToIP(parts[0])
        def port = convertHexToInt(parts[1])
        return ip + ":" + port
    }

``take()`` 명령의 구현에 관하여 주의해야 할 사항들이 있습니다:

#.
