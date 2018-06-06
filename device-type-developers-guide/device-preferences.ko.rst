Preferences
===========
Device Handler는 사용자가 장치의 특정 속성을 구성할 수 있도록 간단한 기본 설정을 지정할 수 있습니다.

----

개요
--------

사용자가 SmartThings에 장치를 추가하면 기기 이름을 지정하고 필요한 공간을 선택할 수 있습니다.
사용자에게 노출시키려는 추가 구성 옵션이 있는 경우에는, ``preferences`` 을 사용하여 지정할 수 있습니다.
이것들은 장치 이름 환경 설정과 동일한 페이지에 나타날 겁니다.

----

Defining preferences
--------------------

Device preferences should be placed in the Device Handler's ``metadata``.
They can appear anywhere in the ``metadata`` definition.

.. code-block:: groovy

    metadata {
        definition(...) {...}
        tiles() {...}
        preferences {
            input "tempOffset", "number", title: "Degrees", description: "Adjust temperature by this many degrees",
                  range: "*..*", displayDuringSetup: false
        }
    }

----

Device preferences are flat
---------------------------

*Device preferences are static, and single-page.*

Multiple page preferences and dynamic preferences pages are not supported in Device Handlers.
Device Handler preferences are a simple list of inputs:

.. code-block:: groovy

    preferences {
        input name: "text", type: "text", title: "Text", description: "Enter Text", required: true
    }

----

Display on setup
----------------

Use ``displayDuringSetup: true`` to force the preference input to be displayed when the device is being added to SmartThings:

.. code-block:: groovy

    preferences {
        input name: "email", type: "email", title: "Email", description: "Enter Email Address", required: true,
              displayDuringSetup: true
    }

Preferences that do not specify this value, or specify ``displayDuringSetup: false``, will only appear when the user presses the Settings button on the Device in the mobile application.

----

Supported input types
---------------------

The following input types are supported in Device Handler preferences:

- ``bool``
- ``decimal``
- ``email``
- ``enum``
- ``number``
- ``password``
- ``phone``
- ``time``
- ``text``

----

Getting preference input values
-------------------------------

Just as with SmartApp preferences, the name of the preferences input is a reference to the preference value:

.. code-block:: groovy

    metadata {
        definition(...) {...}
        tiles() {...}
        preferences {
            input "tempOffset", "number", title: "Degrees", description: "Adjust temperature by this many degrees", range: "*..*", displayDuringSetup: false
        }
    }

    def someCommandMethod() {
        if (tempOffset) {
            // handle offset value
        }
    }

.. note::

    Preference values are only available to the Device Handler when it is executing in response to Events or commands.
    It is not possible to use preference values in other ``metadata`` definitions, including ``tiles()``.

----

Example
-------

.. code-block:: groovy

    metadata {
        simulator {
            // TODO: define status and reply messages here
        }

        tiles {
            // TODO: define your main and details tiles here
        }

        preferences {
            input name: "email", type: "email", title: "Email", description: "Enter Email Address", required: true, displayDuringSetup: true
            input name: "text", type: "text", title: "Text", description: "Enter Text", required: true
            input name: "number", type: "number", title: "Number", description: "Enter number", required: true
            input name: "bool", type: "bool", title: "Bool", description: "Enter boolean", required: true
            input name: "password", type: "password", title: "password", description: "Enter password", required: true
            input name: "phone", type: "phone", title: "phone", description: "Enter phone", required: true
            input name: "decimal", type: "decimal", title: "decimal", description: "Enter decimal", required: true
            input name: "time", type: "time", title: "time", description: "Enter time", required: true
            input name: "options", type: "enum", title: "enum", options: ["Option 1", "Option 2"], description: "Enter enum", required: true
        }
    }

    def someCommand() {
        log.debug "email: $email"
        log.debug "text: $text"
        log.debug "bool: $bool"
        log.debug "password: $password"
        log.debug "phone: $phone"
        log.debug "decimal: $decimal"
        log.debug "time: $time"
        log.debug "options: $options"
    }

----

Additional notes
----------------

- Setting a default value (``defaultValue: "foobar"``) for an input may render that selection in the mobile app, but the user still needs to enter data in that field. It's recommended to not use ``defaultValue`` to avoid confusion.
