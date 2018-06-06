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

#. HubAction의 특정 경로, 메서드(method), 헤더는 각 장치 마다 다릅니다. 이 정보는 장치 제조 업체의 설명서를 참조하십시오.
#. 반드시 ``hubAction.options = [outputMsgToS3: true]``를 지정하십시오. 그러면 이미지가 저장됩니다 (일시적으로). 다음은 이미지를 장기-저장 저장소로 이동합니다.
#. 명령 메서드에서 HubAction을 반환(return)하는 것을 잊지 마십시오. 반환하지 않으면 작업이 실행되지않습니다!

일단 우리가 ``take()`` 명령으로 요청을 하면 장치의 응답이 장치 핸들러의 ``parse()`` 메서드로 보내집니다.
이 응답은 방금 찍은 사진의 key인 ``tempImageKey``를 포함할 것입니다.

.. code-block:: groovy

    def parse(String description) {

        def map = stringToMap(description)

        if (map.tempImageKey) {
            try {
                storeTemporaryImage(map.tempImageKey, getPictureName())
            } catch (Exception e) {
                log.error e
            }
        } else if (map.error) {
            log.error "Error: ${map.error}"
        }

        // parse other messages too
    }

    private getPictureName() {
        return java.util.UUID.randomUUID().toString().replaceAll('-', '')
    }

``parse()``는 다음 작업을 수행합니다:

#. 응답을 확인하여 ``tempImageKey`` 가 전송되었는지 확인합니다. 만약 그렇다면 이 응답은 우리의 ``take()`` 명령에 의한 이미지 응답이라는 것을 의미합니다.
#. ``tempImageKey``와 사진의 이름을 가지고 ``storeTemporaryImage()``를 부릅니다. 사진의 이름은 각 기기 인스턴스안에서 고유한 값이어야 하며 오직 알파벳,숫자,"-","_,"." 만이 허용됩니다. ``storeTemporaryImage()``는 사진을 임시 저장소에서 365일 동안 저장하고 지나면 삭제되는 위치로 이동합니다.

또한 ``storeTemporaryImage()``은 우리의 Carousel 타일이 연관 된 속성인 "image"이벤트를 생성합니다.
이를 통해 이미지는 타일에서 표시될 수 있습니다.

클라우드 연결 카메라
^^^^^^^^^^^^^^^

``take()`` 명령어는 사진을 찍고 찍 사진 바이트들을 :ref:`device_handler_ref_store_image`를 저장하기 위해 타사 서비스에 HTTP 요청을 보냅니다.

다음은 간단한 예시입니다 (실제 응용 프로그램은 타사 인증과 추가 오류를 처리해야 합니다):

.. code-block:: groovy

    def take() {
        def params = [
            uri: "https://some-uri",
            path: "/some/path"
        ]

        try {
            httpGet(params) { response ->
                // 우리는 이 경우에 제 3자로 부터 "image/jpeg" 내용 유형을 기대합니다.
                if (response.status == 200 && response.headers.'Content-Type'.contains("image/jpeg")) {
                    def imageBytes = response.data
                    if (imageBytes) {
                        def name = getImageName()
                        try {
                            storeImage(name, imageBytes)
                        } catch (e) {
                            log.error "Error storing image ${name}: ${e}"
                        }

                    }
                } else {
                    log.error "Image response not successful or not a jpeg response"
                }
            }
        } catch (err) {
            log.debug "Error making request: $err"
        }

    }

    def getImageName() {
        return java.util.UUID.randomUUID().toString().replaceAll('-','')
    }

.. warning::

    Only synchronous HTTP requests are supported when using the Carousel Tile.

위의 ``take()``명령어는 다음 작업을 수행합니다:

#. 이미지 응답을 반환할 URI에 요청합니다. 진정한 통합은 요청에 대한 승인 정보를 제공해야 할 것입니다. 이는 일반적으로 설치 (:ref:`here <cloud_service_manager_oauth>`에 나와 있습니다) 프로세스를 통해 얻은 OAuth 토큰입니다.
#. 만약 반응이 성공적이고 그 내용 유형(Content-Type)이 우리가 기대한 내용이라면, 그것은 ``response.data``로부터 이미지 바이트를 얻는다.
#. ``storeImage()``로 UUID에서 생성된 이름을 사용하여 사진을 저장합니다. 사진의 이름은 각 장치 인스턴스 마다 고유해야 합니다.

``storeImage()``는 "image" 이벤트를 발생시킵니다, 이를 통해 Carousel Tile이 새로운 사진으로 업데이트 됩니다.

.. tip::

    ``httpGet()``는 이미지에 대한 응답 데이터를 ``ByteArrayInputStream``으로 직렬화 (serialize) 것이며, 따라서 우리는 응답 본문을 ``storeImage()``로 전달할 수 있습니다.

----

사진 크기 제한
-----------

사진의 크기의 한도는 1메가 바이트 입니다.

``storeImage()``는 이 한도를 초과하면 ``InvalidParameterException`` Exception을 throw합니다.

이 한도를 넘는 사진을 ``HubAction``를 통해 찍으려 하는 경우에는 ``error`` 응답을 포함하는 메시지가 ``parse()``로 보내집니다:

.. code-block:: groovy

    def parse(String description) {
        def map = stringToMap(description)

        if (map.error) {
            log.error "error: ${map.error}"
        } else if (map.tempImageKey) {
            //...
        }
    }

----

.. _image_name_allowed_chars:

사진 이름에 허용되는 문자들
--------------------

사진 이름에는 알파벳과 숫자, "-", "_", 그리고 "."만이 허용됩니다.

만약 이름에 다른 문자가 포함되는 경우 ``storeTemporaryImage()`` 와 ``storeImage()``가 ``InvalidParameterException``를 throw합니다.

----

사진 저장 기한
-----------

``HubAction``을 통해 저장된 이미지는 24시간 동안 저장되며, 그 후 삭제됩니다 (따라서 ``storeTemporaryImage()`` 를 사용합니다)

``storeImage()`` 또는 ``storeTemporaryImage()``를 통해 저장된 이미지는 7일 동안 클라이언트가 사용할 수 있으며, SmartThings에 의해 365일 동안 저장됩니다.

----

지원되는 사진 포맷
--------------

``storeImage()``는 JPEG및 PNG이미지 형식을 모두 지원합니다.
컨텐츠 유형은 ``storeImage()``를 호출할 때 지정할 수 있습니다:

.. code-block:: groovy

    storeImage("some-image-name", imgBytes, "image/png")

기본적으로 ``"image/jpeg"``형식이 사용됩니다.

``HubAction``을 통해 캡처되고 ``storeTemporaryImage()``로 저장된 이미지는 JPEG형식이어야 합니다.

두 경우 모두 파일 확장명을 포함할 필요가 없습니다(예:이미지 이름에 ``".jpg"`` 또는 ``".png"``).

----

관련 문서
-------

- :ref:`storeTemporaryImage() reference documentation <device_handler_ref_store_temp_image>`
- :ref:`storeImage() reference documentation <device_handler_ref_store_image>`
- :ref:`HubAction reference documentation <hubaction_ref>`
- :ref:`Image Capture Capability reference documentation <imageCapture>`
- :ref:`Tiles documentation <device_handler_tiles>`


.. _ByteArrayInputStream: https://docs.oracle.com/javase/7/docs/api/java/io/ByteArrayInputStream.html
