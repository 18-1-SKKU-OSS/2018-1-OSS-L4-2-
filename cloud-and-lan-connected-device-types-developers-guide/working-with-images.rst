.. _working_with_camera_photos:

========================================
Capturing and Displaying Camera Pictures
========================================

Cameras connected to SmartThings can use the :ref:`imageCapture` Capability, along with the Carousel Tile, to capture and view images.
SmartThings-connected cameras are either LAN- or Cloud-Connected; this document outlines the steps to capture and display images for both.

----

Image Capture Capability
------------------------

Add support for the :ref:`imageCapture` Capability by including it in the Device Handler's metadata:

.. code-block:: groovy

    metadata {
        definition(name: "My Camera Device", namespace: "MyNamespace", author: "My Name") {
            capability "Image Capture"
            // other definition metadata...
        }
    }

The Image Capture Capability defines one attribute, "image", and one command, ``take()``.
The Carousel Tile can be used to display images and allow the user to manually take a photo, as discussed next.

----

Tiles for taking and viewing pictures
-------------------------------------

Add tiles to allow the viewing and taking of images:

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


The ``carouselTile`` is where the images will be displayed, and the "take" tile allows users to capture images.
Note that both are associated with the ``"image"`` attribute; this association allows the images to be taken and displayed properly.

Capture and display images
--------------------------

The ``take()`` command of the Image Capture Capability is responsible for capturing the image.
Follow the protocol-specific instructions for implementing this command method below.

LAN-connected cameras
^^^^^^^^^^^^^^^^^^^^^

LAN-connected devices can capture images using :ref:`hubaction_ref`, store them using :ref:`device_handler_ref_store_temp_image`, and display them with the :ref:`tiles_carousel_tile`.

The ``take()`` command will issue a request to take a picture via a ``HubAction``.
The response from the device will be sent to the Device Handler's ``parse()`` method, where it can then be moved to longer-lasting storage using ``storeTemporaryImage()``.
``storeTemporaryImage()`` also emits the "image" event, causing the Carousel Tile to be updated with the new image.

Here's an example of the ``take()`` command (details of the request will be specific to each device):

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

Some things to note about the implementation of the ``take()`` command:

#. The specific path, method, and headers of the HubAction will vary for each device. Consult the device manufacturer's documentation for this information.
#. Make sure to specify ``hubAction.options = [outputMsgToS3: true]``. This will result in the image being stored (temporarily). We will move the image to  longer-lasting storage next.
#. Remember to return the HubAction from the command method, otherwise it will not be executed!

Once we've made the request in the ``take()`` command method, the response from the device will be sent to the Device Handler's ``parse()`` method.
This response will contain a ``tempImageKey``, which is the key of the photo just taken.

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

``parse()`` does the following:

#. Checks the response to see if ``tempImageKey`` was sent. If it was, this means that this is the image response from our ``take()`` command.
#. Calls ``storeTemporaryImage()`` with the ``tempImageKey`` and a name for the picture. The name must be unique per device instance, contain only alphanumeric, "-", "_", and "." characters. This will move the image from temporary storage to a location where the image will be stored for 365 days, before being permanently deleted.

``storeTemporaryImage()`` also emits the "image" event, which is the attribute our Carousel Tile is associated with.
This is what allows the image to be displayed in the tile.

Cloud-connected cameras
^^^^^^^^^^^^^^^^^^^^^^^

The ``take()`` command will issue an HTTP request to the third-party service to capture the image, and store the resulting image bytes using :ref:`device_handler_ref_store_image`.

Below is a simplified example (A real application will need to handle authentication with the third-party, as well as additional error handling):

.. code-block:: groovy

    def take() {
        def params = [
            uri: "https://some-uri",
            path: "/some/path"
        ]

        try {
            httpGet(params) { response ->
                // we expect a content type of "image/jpeg" from the third party in this case
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

The ``take()`` command above does the following:

#. Makes a request to a URI that will return an image response. A real integration would need to provide authorization information on the request. This would typically be an OAuth token obtained through the setup process, as documented :ref:`here <cloud_service_manager_oauth>`.
#. If the response is successful and its Content-Type is our expected content, it gets the image bytes from ``response.data``.
#. Stores the image using ``storeImage()``, using a name generated from a UUID. The name of the image is required to be unique for each device instance.

``storeImage()`` will emit the "image" event, which causes the Carousel Tile to be updated with the new image.

.. tip::

    ``httpGet()`` will serialize the response data for images into a ``ByteArrayInputStream``, which is why we can pass the response body to ``storeImage()``.

----

Retrieving an image
-------------------

If you need to retrieve the byte representation of an image stored with ``storeImage()`` or ``storeTemporaryImage()``, use :ref:`device_handler_ref_get_image`.
This will return the bytes of the image in a `ByteArrayInputStream`_.

.. code-block:: groovy

    // Image with "some-name" that was previously stored
    ByteArrayInputStream img = getImage("some-name")


----

Image size limits
-----------------

Images are limited to a maximum of one megabyte.

``storeImage()`` will throw an ``InvalidParameterException`` if this limit is exceeded.

Attempting to capture an image exceeding this maximum using ``HubAction`` will result in the message sent to ``parse()`` containing an ``error`` response:

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

Allowed image name characters
-----------------------------

Image names are restricted to alphanumeric, "-", "_", and "." characters.

An ``InvalidParameterException`` is thrown by ``storeTemporaryImage()`` and ``storeImage()`` if the name contains other characters.

----

Image storage duration
----------------------

Images stored via a ``HubAction`` are stored for 24 hours, after which it is deleted (this is why we use ``storeTemporaryImage()`` to move images captured by a ``HubAction``).

Images stored via ``storeImage()`` or ``storeTemporaryImage()`` are available to clients for seven days, and stored by SmartThings for 365 days, after which it is deleted.

----

Supported image formats
-----------------------

``storeImage()`` supports both JPEG and PNG image formats.
The content type can be specified when calling ``storeImage()``:

.. code-block:: groovy

    storeImage("some-image-name", imgBytes, "image/png")

The content type of ``"image/jpeg"`` is the default.

Images captured via a ``HubAction`` and stored with ``storeTemporaryImage()`` must be in JPEG format.

In either case, there is no need to include the file extension (e.g., ``".jpg"`` or ``".png"`` in the image name).

----

Related documentation
---------------------

- :ref:`storeTemporaryImage() reference documentation <device_handler_ref_store_temp_image>`
- :ref:`storeImage() reference documentation <device_handler_ref_store_image>`
- :ref:`HubAction reference documentation <hubaction_ref>`
- :ref:`Image Capture Capability reference documentation <imageCapture>`
- :ref:`Tiles documentation <device_handler_tiles>`


.. _ByteArrayInputStream: https://docs.oracle.com/javase/7/docs/api/java/io/ByteArrayInputStream.html
