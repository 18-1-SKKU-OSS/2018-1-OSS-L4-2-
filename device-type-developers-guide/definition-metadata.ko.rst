정의
==========

정의 메타 데이터는 장치 핸들러에 대한 핵심 정보를 정의합니다.
초기 값은 장치 핸들러를 작성할 때 입력한 값에서 설정됩니다.

정의 메타 데이터의 예:

.. code-block:: groovy

    metadata {
        definition(name: "test device", namespace: "yournamespace", author: "your name") {

            capability "Alarm"
            capability "battery"

            attribute "customAttribute", "string"

            command "customCommand"

            fingerprint profileId: "0104", inClusters: "0000,0003,0006",
                        outClusters: "0019"
        }

        ...
    }

정의 방법은 매개 변수 맵과 폐쇄를 사용합니다.

지원되는 매개 변수는 다음과 같습니다.

*name*
    Device Handler의 이름.
*namespace*
    이 장치 처리기의 네임 스페이스. 이것은 당신의 github사용자 이름이어야 합니다. 이는 다른 사용자가 동일한 이름을 사용한 경우에도 장치 핸드 북을 이름별로 조회하여 찾을 수 있는지 확인할 때 사용됩니다.
*author*
    Device Handler의 작성자.

폐쇄는 장치 핸들러의 기능, 특성, 명령 및 지문 정보를 정의합니다.

----

기능
------------

기기가 기능을 지원한다는 것을 정의하려면 폐쇄 상태에서``capability`` 메소드가 ``definition`` 로 전달되었다고 하면 됩니다.

 ``capability`` 메소드에 대한 주장은 능력 이름이다.

.. code-block:: groovy

    capability "Actuator"
    capability "Power Meter"
    capability "Refresh"
    capability "Switch"

----

특성
----------
장치 핸들러에 대한 사용자 지정 특성을 정의해야 하는 경우, 폐쇄에서 ``attribute()`` 메소드가 ``definition()`` 메소드로 전달됨을 호출합니다.

**attribute(String attributeName, String attributeType, List possibleValues = null)**

*attributeName*
    Name of the attribute.

*attributeType*
    Type of the attribute. Available types are "string", "number", and "enum".

*possibleValues*
    Optional. The possible values for this attribute. Only valid with the "enum" attributeType.

.. code-block:: groovy

    // String attribute with name "someName"
    attribute "someName", "string"

    // enum attribute with possible values "light" and "dark"
    attribute "someOtherName", "enum", ["light", "dark"]


----

Commands
--------

To define a custom command for your Device Handler, call the ``command()`` method in the closure passed to the ``definition()`` method:

**command(String commandName, List parameterTypes = [])**

*commandName*
    The name of the command. You must also define a method in your Device Handler with the same name.
*parameterTypes*
    Optional. An ordered list of the parameter types for the command method, if needed.

.. code-block:: groovy

    // command name "myCommand" with no parameters
    command "myCommand"

    // command name myCommandWithParams that takes a string and a number parameter
    command "myCommandWithParams", ["string", "number"]

    ...

    // each command specified in the definition must have a corresponding method

    def myCommand() {
        // handle command
    }

    // this command takes parameters as defined in the definition
    def myCommandWithParams(stringParam, numberParam) {
        // handle command
    }

----

Fingerprinting
--------------

When a ZigBee or Z-Wave device is added to the SmartThings Hub, we need a way to determine which device type to assign it.
This process is known as a "join" process, or "fingerprinting".

Device Handlers define "fingerprints" to specify which devices or what kinds of devices they support.
Then, when a device is added, its join information is compared to all fingerprints in the default handlers and your
self-published handlers to determine which type of device it is.

The fingerprinting process differs between ZigBee and Z-Wave devices.

.. _zigbee-fingerprinting-label:

ZigBee fingerprinting
^^^^^^^^^^^^^^^^^^^^^

For ZigBee devices, the main profileIds you will need to use are:

-  HA: Home Automation (0104)
-  SEP: Smart Energy Profile
-  ZLL: ZigBee Light Link (C05E)

The input and output clusters are defined specifically by your device and should be available via the device's documentation.

An example of a ZigBee fingerprint definition:

.. code-block:: groovy

        fingerprint profileId: "C05E", inClusters: "0000,0003,0004,0005,0006,0008,0300,1000", outClusters: "0019"

You can also include the manufacturer and model name in the fingerprint to limit the fingerprint to a specific product:

.. code-block:: groovy

    fingerprint inClusters: "0000,0001,0003,0020,0406,0500", manufacturer: "NYCE", model: "3014"

.. _zwave-fingerprinting:

Z-Wave fingerprinting
^^^^^^^^^^^^^^^^^^^^^

Z-Wave fingerprints used to be based on the format used for ZigBee, but there is now a new format that is preferred.
You may see the original fingerprints on older Device Handlers; see below for information on the legacy format.

The best place to start is to add your device to SmartThings and look for the *Raw Description* in its details view
in the SmartThings developer tools.

Z-Wave raw description
++++++++++++++++++++++

Z-Wave devices added since the introduction of the new format will have raw description strings with multiple key-value
fields, such as::

    zw:Ss type:2101 mfr:0086 prod:0102 model:0064 ver:1.04 zwv:4.05 lib:03 cc:5E,86,72,98,84 ccOut:5A sec:59,85,73,71,80,30,31,70,7A role:06 ff:8C07 ui:8C07

Not all fields will be present for every device.

**zw:**
    This field will start with 'L' for listening devices, 'S' for sleepy devices, and 'F' for beamable devices. See the
    :ref:`Z-Wave Primer <zwave-primer>` for the meaning of those terms. That capital letter will be followed by a
    lowercase 's' if the device is securely included into the network via the Z-Wave Security Layer.
**type:**
    This field is the Z-Wave Device Class as a 16-bit hexadecimal number that combines the Generic and Specific Device
    Class codes. [1]_
**mfr:**
    This 16-bit hexadecimal number identifies the device manufacturer. [1]_ The three values of ``mfr``, ``prod`` and
    ``model`` uniquely identify a certified Z-Wave product.
**prod:**
    This 16-bit hexadecimal number is the Product Type ID reported by the device.
**model:**
    This 16-bit hexadecimal number is the Product ID reported by the device.
**ver:**
    This is the application firmware version reported by the device.
**zwv:**
    This is the version of the Z-Wave protocol stack being used by the device.
**lib:**
    This indicates the type of Z-Wave protocol library the device is based on. '01' is a static controller, '02' is a
    remote controller, '07' is a bridge controller, and other values are normal non-controller devices.
**cc:**
    The list of Z-Wave command classes supported by the device (without security encapsulation). See the
    `Z-Wave Command Reference <https://graph.api.smartthings.com/ide/doc/zwave-utils.html>`__ for the command classes
    represented by each hex code.
**ccOut:**
    The list of Z-Wave command classes that the device can control. This refers to commands sent to other devices versus
    reports generated by the device.
**sec:**
    These command classes are supported by the device only via Z-Wave Security encapsulation.
**secOut:**
    These command classes are *controlled* by the device only via Z-Wave Security encapsulation.
**role:**
    This indicates the Z-Wave Plus Role Type. [1]_
**ff:**
    This stands for "form factor" and corresponds to the Z-Wave+ Installer Icon type (An offset of 0x8000 is added for
    implementation reasons). [1]_
**ui:**
    This corresponds to the Z-Wave+ User Icon type.

.. [1] See `this document <http://zwavepublic.com/files/sds13740-1-z-wave-plus-device-and-command-class-types-and-defines-specificationpdf>`_ for the values of identifiers defined by the Z-Wave standard.

New Z-Wave fingerprint format
+++++++++++++++++++++++++++++

If you're writing a Device Handler for a specific device, you can base the fingerprint on the manufacturer info.
For example, the fingerprint to match the raw description example above would be:

.. code-block:: groovy

    fingerprint mfr: "0086", prod: "0102", model: "0064"

No other parameters are required. Note that you need to add quotes and commas to the more concise raw description format
to make it valid Groovy code.

Sometimes related products are grouped under the same 'prod' ID. In that case you can use a fingerprint without the
'model' parameter.

If you are writing a general Device Handler that supports all devices of a certain type, you can still base the
fingerprint on command class support.

.. code-block:: groovy

    fingerprint type: "10", cc: "25,32"

That fingerprint would match all devices of the Binary Switch generic device class – i.e. their 'type' starts with
"10" – that support the Binary Switch (0x25) and Meter (0x32) command classes.

The supported parameters are:

**type:**
    Matches if it's equal to or a prefix of the device's 'type' value in the raw description. Aliased as 'deviceId'.
**mfr, prod, model:**
    Matches if 'mfr' matches the raw description and 'prod' and 'model' match as prefixes (if present).
**cc, ccOut:**
    Takes a list of command class codes as a string: comma-separated, uppercase hexadecimal. Matches if all listed
    command class codes are reported as supported or controlled respectively in the device's raw description.
**sec, secOut:**
    The same as the previous parameter, but only matches against command classes the device supports/controls only via
    Z-Wave Security encapsulation.
**ff/ui:**
    Either of these parameters can be used to match against the corresponding fields of the raw description. It is only
    possible to use one of the following in a single fingerprint: 'type', 'deviceId', 'ff', ui'.
**deviceJoinName:**
    Not used for matching. If the fingerprint matches, the device will appear to the user with this name.

When multiple device fingerprints match an added Z-Wave device, they are ranked first by number of 'mfr', 'prod', and
'model' parameters, then by the number of command classes listed, and finally by the length of the 'type', 'ff', or
'ui' parameter. When fingerprints have the same rank, self-published Device Handlers take precedence over the default
production ones.

Legacy Z-Wave fingerprint format
+++++++++++++++++++++++++++++++

Legacy fingerprints include the device class – or ``type`` value (see above) – in the ``deviceId`` parameter and the
command classes it supports in the ``inClusters`` parameter. So the fingerprint:

.. code-block:: groovy

    fingerprint deviceId:"0x1104", inClusters:"0x26, 0x2B, 0x2C, 0x27, 0x73, 0x70, 0x86, 0x72", outClusters: "0x20"

would be formatted in the new style as:

.. code-block:: groovy

    fingerprint type: "1104", cc: "26,2B,2C,27,73,70,86,72", ccOut: "20"

.. _device_handler_fingerprinting_best_practices:

Fingerprinting best practices
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Add multiple fingerprints
+++++++++++++++++++++++++

A Device Handler can have multiple fingerprints in order to work with multiple versions of a device.
Each fingerprint is independent. If any of them is the highest ranking match, the device will use your device type.

You can distinguish between the different devices that use the handler by adding the 'deviceJoinName' parameter.
For example:

.. code-block:: groovy

    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "0019", manufacturer: "sengled", model: "Z01-CIA19NAE26", deviceJoinName: "Sengled Element touch"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "000A, 0019", manufacturer: "Jasco Products", model: "45852", deviceJoinName: "GE Zigbee Plug-In Dimmer"
    fingerprint profileId: "0104", inClusters: "0000, 0003, 0004, 0005, 0006, 0008, 0702, 0B05", outClusters: "000A, 0019", manufacturer: "Jasco Products", model: "45857", deviceJoinName: "GE Zigbee In-Wall Dimmer"

If an added device supports the inClusters in the first fingerprint but doesn't match all the extra info in any of the
next three, it will join with the name from the handler's definition metadata, in this case "ZigBee Dimmer Power."

Device pairing process
++++++++++++++++++++++

The order of the ``inClusters`` and ``outClusters`` lists is not important to the pairing process.
It is a best practice, however, to list the clusters in ascending order.

The device can have more clusters than the fingerprint specifies, and it will still pair.
If one of the clusters specified in the fingerprint is incorrect, the device will *not* pair.

Overly general fingerprints
+++++++++++++++++++++++++++

If you wish to publish or share a Device Handler, you must make sure that the fingerprints do not capture other devices
that aren't covered by your handler.

If you copied a working fingerprint from a default or template handler, it would be ambiguous which type should match if
yours was published. The easiest way to remedy this is to include manufacturer and model info in all fingerprints.
