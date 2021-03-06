.. _first-smartapp-tutorial:

첫 번째 SmartApp을 작성해보세요
===========================

.. role:: strike
    :class: strike

이 튜토리얼은 SmartApp을 처음으로 작성하는 법을 여러분에게 안내해줄겁니다.
:ref:`groovy-for-smartthings` 을 읽으셨다면 다음으로 이 튜토리얼을 읽으시면 됩니다.

----

목표
-----

튜토리얼을 다 보면 이것들을 알게 될 것입니다:

- 웹 기반 IDE를 사용하여 SmartApp를 만드는 방법
- SmartApp의 주요 구성 요소
- SmartApp을 구성하기 위해 사용자로부터 입력을 수집하는 방법
- 장치 상태의 변경 사항을 구독하는 방법.
- 장치를 제어하는 방법
- 향후 실행을 위해 SmartApp를 스케쥴링하는 방법
- 시뮬레이터로 SmartApp을 테스트하는 방법.
- SmartApp을 게시하고 휴대 전화에 설치하는 방법.
- :strike:`노력 없이 세계를 지배하는 법`

우리가 만들 SmartApp은 비교적 간단하지만 그 과정에서 SmartThings의 몇 가지 핵심 개념을 알게되고 개발 프로세스에 익숙해질 겁니다.

저희가 만들 SmartApp이 하는 일은 동작이 감지되면 스위치를 켜고 동작이 중지되면 스위치를 끄는 것입니다

----

.. _ide_requirements:

선행 조건
-------------

이 튜토리얼을 완료하기 전에, :ref:`get-started-overview` 를 읽고, :ref:`quick-start` 페이지에서 논의 된 것처럼 계정을 등록해야합니다.
최소한 :ref:`groovy-basics` 및 :ref:`groovy-for-smartthings` 튜토리얼에서 다뤄진 기본적인 Groovy 개념에 익숙해지는 것이 좋습니다.

https://graph.api.smartthings.com 에서 IDE에 로그인하면서 시작하세요.
그런 다음 *내 위치* 페이지로 이동하여 내가 만든 위치를 확인하세요.

.. image:: ../img/getting-started/first-smartapp/my-locations.png
   :width: 70%

일반적으로는 허브를 설치 한 위치만 표시됩니다.
맨 왼쪽 열 (예) *이름* 열)에 나타나는 위치 이름을 클릭하세요. SmartThings 사용자 아이디와 암호로 다시 로그인해야 할 수도 있습니다.

.. warning::

　　IDE가 https://graph.api.smartthings.com 에 있더라도 SmartApp 배포에 올바른 URL이 아닐 수도 있습니다.
　　명시적으로 위치 이름을 선택하면 SmartApp가 제대로 게시될 겁니다.

SmartApp는 모션 센서와 스마트 스위치를 활용합니다.
이러한 장치가 없거나 Hub가없는 경우에도 이 튜토리얼의 대부분을 완료할 수 있습니다.
하드웨어가 없는 경우 필요한 단계에 대해서도 안내하겠습니다.

----

SmartApp 만들기
-----------------

IDE에서 *My SmartApps* 페이지로 이동하세요.
이렇게하면 생성한 모든 SmartApp을 보여주는 페이지로 이동합니다.
또한 여기에서 새 SmartApp을 만들 수 있습니다. *New SmartApp* 버튼을 클릭하세요.

.. image:: ../img/getting-started/first-smartapp/new-smartapp.png
   :width: 70%

새로운 SmartApp 생성을 위해 세 가지 옵션이 제공됩니다: *From Form,* *From Code,* and *From Template.*

.. image:: ../img/getting-started/first-smartapp/smartapp-form.png
   :width: 70%

*From Form* 옵션에서는 SmartApp에 대한 세부 정보를 채워넣고 상용구 코드가 있는 SmartApp을 만듭니다.

*From Code* 옵션은 입력 상자에 입력된 코드로 새로운 SmartApp을 만듭니다.

마지막으로 *From Template* 옵션을 사용하면 이미 존재하는 SmartApp을 선택하여 해당 코드를 시작점으로 사용할 수 있습니다.
이미 존재하는 SmartApp를 변경하거나 향상하려는 경우 유용하며 예제를 보고싶을 때도 유용합니다.

우리는 *From Form* 옵션으로 해보겠습니다.

다음과 같이 양식을 작성하세요:

이름
　　작성될 SmartApp의 이름입니다. "My First SmartApp"과 같은 이름을 붙입니다.

네임 스페이스
　　이 필드는 다른 사람이 똑같은 이름으로 SmartApp을 작성한 경우에도 SmartApp을 고유하게 식별합니다.
　　이것은 GitHub 사용자 이름이어야합니다(또는 GitHub 계정이 없는 경우 다른 고유한 식별자를 사용해도 됩니다.).

저자
　　이것은 당신입니다. 해당하는 내용으로 필드를 채워주세요.

설명
　　SmartApp의 의도와 기능을 설명합니다.
　　이 설명은 SmartThings 모바일 애플리케이션의 SmartApp 마켓플레이스 부분에 작성되므로 명확하고 간결한 설명을 써주세요.

범주
　　SmartApp는 기능에 따라 분류됩니다.
　　이것은 SmartThings 모바일 애플리케이션에서 사용됩니다.
　　SmartApp는 마켓플레이스 용이나 혼자 쓸 용도로 출판될 수 있습니다.
　　혼자 쓸 용도로 SmartApp을 게시할 때(지금 저희가 할 것과 같이) 모든 SmartApp이 *My Apps* 범주에 나타납니다.

나머지 필드는 그대로 두고 하단의 *Create* 버튼을 클릭하세요.
이렇게 하면 SmartApp이 만들어지고 일부 스켈레톤 코드가 채워집니다.
다음 부분에서는 편집기를 사용하여 첫 번째 SmartApp을 만들어보겠습니다.

----

편집기
------

SmartApp를 만들면 편집기와 시뮬레이터로 이동합니다.
코드를 살펴보기 전에 기본 기능에 익숙해질 필요가 있습니다.

코드 창 위에는 다섯 개의 버튼이 있습니다.

.. image:: ../img/getting-started/first-smartapp/editor-buttons.png

저장
　　이 단추는 SmartThings 클라우드에 SmartApp을 저장합니다.

게시
　　SmartApp을 직접 게시할 수 있게 해줍니다. 그래서 SmartThings 모바일 앱에 SmartApp를 설치하고 SmartThings 팀에 제출하여 SmartThings 카탈로그에 게시할 수 있습니다.

IDE 설정
　　여기서 원하는 대로 편집기를 개인 설정할 수 있습니다.
　　여러 테마 중 원하는 테마를 고를 수 있고, 키 위치를 마음대로 지정하며 글꼴 크기를 설정할 수 있습니다.

앱 설정
　　앱 설정에서는 SmartApp을 만들 때 입력한 값들이 있는 양식을 보여줍니다. SmartApp의 속성을 편집할 수도 있습니다.

시뮬레이터
　　이 버튼은 온라인 시뮬레이터 화면을 토글합니다. 시뮬레이터에 대해서는 다음 부분에 더 자세히 설명하겠습니다.

.. tip::

　IDE 오른쪽 상단의 *Simulator* 메뉴에 *Browse SmartApp Templates* 라는 드롭 다운메뉴를 볼 수 있을겁니다.
　　클릭하면 참고할 수 있거나 SmartApp을 만들기 시작할 때 시작점으로 쓸 수 있는 다양한 SmartApp들을 볼 수 있습니다.

----

Simulator
---------

On the right side of the IDE is the Simulator.
This is where you can install your SmartApp to test it, either using physical devices, or simulated devices.
We will walk you through installing the SmartApp using this later in the tutorial.

.. image:: ../img/getting-started/first-smartapp/simulator-1.png

If you don't have a Location yet, the Simulator will show a message instructing you to create one. Follow the steps there to create a Location.

----

SmartApp basics
---------------

The first thing to know is that there are a few different types of SmartApps.

Some SmartApps, called *Service Manager SmartApps*, manage the connection of a Cloud-connected or LAN-connected device.

*Solution Module SmartApps* provide a dashboard-like user interface in the SmartThings mobile application [1]_.

The most common type of a SmartApp is one that monitors the user's devices for certain changes (or simply execute on a defined schedule), and then take certain action ("Turn a light on when motion is detected").
These SmartApps are called *Event-Handler SmartApps*.

This tutorial will walk you through building a simple Event-Handler SmartApp, but the core principles you will learn are applicable to all types of SmartApps.

Regardless of what type of SmartApp you are writing, there are a few core principles that apply to all SmartApps:

- SmartApps are not continuously running. They are executed in response to various Events or schedules.
- SmartApps are installed into a user's Location, and a user may install multiple instances of a SmartApp into the same Location.
- With the exception of Solution Module SmartApps, SmartApps do not have any user interface, except for the preferences page that allows the user to configure the SmartApp (more on this in a bit).
- The code that defines a SmartApp does not run on the user's mobile phone. SmartApps may execute in the SmartThings cloud, or on the Hub. The mobile application uses some information from the SmartApp to drive the experience in the app.

In your editor, you can see that there is some code already written for you.
This defines the basic structure and skeleton for your SmartApp.
We will discuss each key component as we build our SmartApp.

----

Definition
----------

Every SmartApp must have a ``definition`` method call.
This provides metadata about the SmartApp itself.
The ``definition`` method simply expects a map of parameters.
If you look at the code in the editor, you'll see that these values are already set from the values you entered when creating your SmartApp:

.. code-block:: groovy

    definition(
        name: "My First SmartApp",
        namespace: "mygithubusername",
        author: "Peter Gregory",
        description: "This is my first SmartApp. Woot!",
        category: "My Apps",
        iconUrl: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience.png",
        iconX2Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png",
        iconX3Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png")

We don't need to change anything here, so let's move on to defining our preferences.
If you do need to change some of your SmartApp's metadata, you can change these values later.

----

Preferences
-----------

The ``preferences`` method is where we define what information our SmartApp needs from the user.
When a user installs a SmartApp on their mobile device, they will be taken to a screen (or screens) where they can configure the SmartApp.
The content of these screens are derived from our ``preferences`` definition.

Preferences can be displayed as a simple, single screen, or multiple screens.
This tutorial will use a simple preferences definition, with only one screen.

In the editor, there is a ``preferences`` definition stubbed in for us:

.. code-block:: groovy

    preferences {
    	section("Title") {
    		// TODO: put inputs here
    	}
    }

Recall that the purpose of our SmartApp is to turn a switch on when motion is detected.
Our SmartApp needs to know which switch and motion sensor to work with.
Update  ``preferences`` with this code:

.. code-block:: groovy

    preferences {
        section("Turn on when motion detected:") {
            input "themotion", "capability.motionSensor", required: true, title: "Where?"
        }
        section("Turn on this light") {
            input "theswitch", "capability.switch", required: true
        }
    }

Notice that we defined two ``section`` calls.
Sections allow us to group related inputs, and can have a text description ("Select a switch to turn on").

We use the ``input`` method to specify what types of devices we want the user to choose from.
Let's break down in detail the ``input`` for the switch:

.. code-block:: groovy

    input "theswitch", "capability.switch", required: true

The first argument to ``input`` is what we - inside our SmartApp - want to refer to the device as.
In this case, we use ``"theswitch"``.
This becomes the identifier for the device in our SmartApp, so that we can refer to the switch as ``theswitch`` (without the quotes).
We'll see this in action shortly.

The second argument is the type of device our SmartApp will work with.
``"capability.switch"`` states that our SmartApp is requesting the user to pick from *any* device that supports the Switch *capability*.
The concept of capabilities is core to SmartThings, and requires a bit more explanation.

First, consider that the catalog of connected devices is growing at a rapid pace.
New devices arrive on the market almost daily.
Many of these devices do similar things, and some do multiple things.

Capabilities
^^^^^^^^^^^^
SmartThings abstracts devices into their *capabilities* - that is, what the device is capable of.
This allows us to build SmartApps that can work with *any* device that supports a given capability.
In this way, we can build robust SmartApps that will work with any device integrated with SmartThings that supports a given capability.

Capabilities are broken down into *commands* and *attributes*.
*Commands* can be issued to a device, and *attributes* are what the device reports on.
Every capability defines its commands and attributes, and devices that support a given capability must support those commands and attributes.

.. note::

    A device may (and typically does) support multiple capabilities.
    For example, a Phillips Hue Bulb supports the Switch capability, because it can turn on and off.
    It also supports the Color Control capability, since the bulb can change colors.
    In our example, a Hue bulb could be selected by the user since it supports the Switch capability.

    But, our SmartApp is only requesting that a user select a device that supports the Switch capability, so even if the user selects a device that can do more (such as a Hue bulb), we cannot assume that in our SmartApp.
    All we can know is that the device supports the Switch capability.

With capabilities, we can be assured that even if a new device supporting the Switch capability is added after we've written and published our SmartApp, there's no need to update any code!

Capabilities are created and maintained by SmartThings.
You can view the reference documentation for capabilities in the  :ref:`capabilities_taxonomy`.

The last thing to note in our ``input`` method call is the ``required: true`` argument.
This specifies that the user must select a device in order to install the SmartApp.

.. important::

    By requiring users to select which devices the SmartApp will work with, SmartThings is providing a basic security feature - SmartThings can only control those devices which a user explicitly chooses.
    SmartApps cannot control devices which the user did not select, and this is by design.

To summarize, when the user selects and installs the SmartApp from within SmartThings mobile app, they will be prompted to select a device that supports the switch capability.
The SmartThings mobile app will provide them with a list of devices for this user's Location that support the switch capability.
The device chosen will then be identified within the SmartApp as ``theswitch``.

We covered a lot of information for such a small amount of code because it's important that you understand the importance of ``preferences`` and capabilities.

For additional information about preferences, see the :ref:`prefs_and_settings` chapter of the SmartApp guide.

Now that you've updated the ``preferences`` method, make sure to save your SmartApp by clicking the *Save* button.

----

Events and callback methods
---------------------------

Our SmartApp needs to turn a switch on when motion is detected.
To turn the switch on, we first need to know when motion is detected.

SmartApps can subscribe to various Events so that when that Event happens, the SmartApp will be notified.
For our SmartApp we do this by using the :ref:`smartapp_subscribe` method.

In your editor, below the ``preferences``, you'll see some methods already defined:

.. code-block:: groovy

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
    	// TODO: subscribe to attributes, devices, locations, etc.
    }

    // TODO: implement event handlers

Every SmartApp must define methods named :ref:`smartapp_installed` and :ref:`smartapp_updated`.
When a user installs a SmartApp by clicking on the *Install* button in the SmartThings mobile application (after filling out any required preferences inputs), the ``installed()`` method we define in our SmartApp will be called.
This is where SmartApps can subscribe to any device changes we are interested in, as well as set up any scheduled tasks we want our SmartApp to perform.

Similarly, the ``updated()`` method is called when a user updates their installation of the SmartApp by changing any of the preferences inputs.
For example, a user may want to change which switch is turned on after they have installed the SmartApp.
So, they open the SmartApp settings, select a different switch, and then update the SmartApp.
At this point, the ``updated()`` method is called.

In our ``updated()`` method, notice that the first thing we do (aside from some logging, which is discussed shortly), is to call a method called :ref:`smartapp_unsubscribe`.
This method is provided by the SmartThings platform, and simply removes any existing subscriptions this SmartApp has created.
This is important, since the user has just changed their preferences for this SmartApp.
If we didn't do this, we might still be subscribed to Events for devices that the user has removed from the SmartApp.

Also, note that both ``installed()`` and ``updated()`` call a method named ``initialize()``.
Since both ``installed()`` and ``updated()`` typically both create subscriptions or schedules, we can reduce code duplication by using a helper method.

We also use the built-in logger (``log``) to log information.
SmartThings does not currently have a debugger within the IDE, so use the ``log()`` method to log information that might be useful for debugging.
The logs are available by clicking *Live Logging* at the top of the IDE.

Finally, note that we reference a variable named ``settings`` in our log statement.
Remember the preference inputs we defined? Every preference input gets stored in a read-only map called ``settings``.
We can get the values of the various inputs by indexing into the ``settings`` map with the name of the input (e.g., ``settings.theswitch``).

Now that you understand the purpose and importance of the ``installed()`` and ``updated()`` methods, we need to subscribe to any Events that we are interested in.
In our case, we need to know when the motion sensor reports that it detected motion.

In the editor, update the ``initialize()`` method with this:

.. code-block:: groovy

    def initialize() {
        subscribe(themotion, "motion.active", motionDetectedHandler)
    }

The ``subscribe()`` method accepts three parameters: The thing we want to subscribe to (``themotion``), the specific attribute and its state we care about (``"motion.active"``), and the name of the method (``motionDetectedHandler``) that should be called when this Event happens.

How do you know what attribute and what state we can subscribe to?
We refer to the :ref:`capabilities_taxonomy` to find out the available attributes the capability supports.
In the case of the Motion Sensor capability, we see that it supports the ``"motion"`` attribute.
In this case, it has two discreet possible values - "active" and "inactive".

Since the ``"motion"`` attribute value is either active or inactive, we can subscribe to either of those specific changes by using the format ``"<attribute>.<value>"``.
This will cause the specified event handler method to be called any time the ``"motion"`` attribute value changes to ``"active"`` (motion is detected).

Now that we've created our subscription, we need to define the event handler method.

----

Event Handler methods
---------------------

Add the following method to your SmartApp.
We'll fill in the real meat of the method later.

.. code-block:: groovy

    def motionDetectedHandler(evt) {
        log.debug "motionDetectedHandler called: $evt"
    }


Every event handler method must accept a single parameter, which is an :ref:`event_ref` object that contains information about the Event, such as the Event's value, time it occurred, and other information.

Since we subscribed to the ``"active"`` state of the motion sensor, we know that our event handler method will only be called when the motion sensor changes from inactive to active.

Now that we know motion has been detected, we need to turn the light on!

----

Controlling devices
-------------------

Recall that capabilities support commands (things the device can do), as well as attributes (things the attribute knows).
To turn the switch on requires only one line of code to be added to our event handler:

.. code-block:: groovy
    :emphasize-lines: 3

    def motionDetectedHandler(evt) {
        log.debug "motionDetectedHandler called: $evt"
        theswitch.on()
    }

Simple, right?
But how do we know that we can call the ``on()`` method on the switch?
By looking at the :ref:`Switch Capability Reference <switch>`, we see that the Switch capability supports the ``on()`` and ``off()`` commands.
These turn the switch on and off, respectively.

Also note that we referred to the switch selected by the user by the name we provided in the ``input`` inside ``preferences`` (theswitch).

----

Using the Simulator
-------------------

Save your SmartApp by clicking the *Save* button at the top of the IDE.
Click *Simulator* and you will see a Location section on the right-hand side:

.. image:: ../img/getting-started/first-smartapp/ide-location.png
   :width: 35%

SmartApps are installed to a Location in your SmartThings account.
By clicking the *Set Location* button, you are telling the Simulator that you want to install this SmartApp into the chosen Location.

After you have selected the Location, you will see the *Preferences* section appear:

.. image:: ../img/getting-started/first-smartapp/ide-devices.png
   :width: 35%

This is where you can choose devices that the SmartApp will use.
Here we see that it asks for a motion sensor to monitor, and a switch.
These two inputs directly correspond to what we have in the ``preferences`` section in our SmartApp.
SmartThings will provide a "Virtual Device" when it can.
When you do not have a physical device to choose from this is a very useful option.
By default the virtual devices will be selected.
Click the *Install* button, and the SmartApp will be installed into the Location you selected above.

Now we see the Simulator section appear:

.. image:: ../img/getting-started/first-smartapp/ide-simulator-unactuated.png
   :width: 35%

We have two devices.
A motion sensor, and a switch.
We can manipulate the motion sensor by choosing *active* or *inactive* and clicking the play button.
The same with the switch, it can be *on* or *off*.
We wrote our SmartApp to turn the switch on when motion is detected, so let's give that a try.
Choose *active* if it's not already selected and then hit the play button.
You should see the switch should go on:

.. image:: ../img/getting-started/first-smartapp/ide-simulator-actuated.png
   :width: 35%

.. warning::

   The behavior of the Simulator is known to have inconsistencies.
   If you are unable to see the correct device status, or unable to actuate the device, you may just be experiencing issues with the Simulator.

   In that case, just skip ahead to the next section to install the SmartApp via the SmartThings mobile app.

----

.. _publish-install-smartapp:

Publishing and installing
-------------------------

We can now see our first SmartApp in action in the Simulator.
The next question is how can we use this SmartApp on our mobile devices in the SmartThings app?
To accomplish this, we need to publish the SmartApp.

.. image:: ../img/getting-started/first-smartapp/publish.png
   :width: 70%

When you press the *Publish* button, a *For Me* option will appear.
Select it.
This means that the SmartApp will only be published for your account and not be visible for everyone in the SmartThings community.

.. note::

    If you have a SmartApp that you do want to publish publicly, you can do that via the "My Publication Requests" link at the top of the page.
    For more information on this, see :ref:`publishing-for-distribution`.

Now you should be able to see your SmartApp in the mobile app if you browse to the *My Apps* category of the Marketplace:

==================================================================   =====================================================================
.. image:: ../img/getting-started/first-smartapp/mobile-myapps.png   .. image:: ../img/getting-started/first-smartapp/mobile-myfirstsmartapp.png
==================================================================   =====================================================================

After selecting your SmartApp, you will be brought to the preferences screen where you can select the devices to work with this SmartApp:

.. image:: ../img/getting-started/first-smartapp/installing-smartapp.png
    :width: 40%

You can see the sections and inputs we defined in the ``preferences`` here.
Notice how the inputs are marked in red, to indicate that the user must set values for these inputs in order to install the SmartApp.

Tap the fields to select a motion sensor and switch.
If you have devices that support the requested capability, you'll see an option to select them.

You'll also see that some other inputs were added for us.
For single page preferences, every SmartApp receives an input to allow the user to assign a name of their choosing for this installation.
The name that they choose will then be displayed as the name of the SmartApp.
Also by default, the user can select to only execute this SmartApp when the Location is in certain :ref:`modes`.
It also includes the ability for the user to uninstall this SmartApp.

.. note::

    A SmartApp may be installed into a Location multiple times.
    For example, a person may have multiple rooms for which they want a light to come on when motion is detected.

    Even though the code is the same, each installation is unique, and must also be removed by the user individually.


----

Turn off when motion inactive
-----------------------------

We now have a simple SmartApp that turns a switch on when motion is detected.
Let's extend this further, and turn the switch off when the motion stops.

In our SmartApp, we need to subscribe to not only the motion sensor being active, but also inactive.

Recall that our subscription looks like this:

.. code-block:: groovy

    subscribe(themotion, "motion.active", motionDetectedHandler)

We will also subscribe the ``"motion.inactive"`` Event in a similar way.
Add this subscription to the ``initialize()`` method:

.. code-block:: groovy

    subscribe(themotion, "motion.inactive", motionStoppedHandler)

.. note::

    We could also subscribe to *any* change in the motion sensor, by simply specifying the attribute we want to monitor (e.g., ``"motion"`` instead of ``"motion.active"``).
    This would then call the specified handler method when there is any reported change to the ``"motion"`` attribute.
    For attributes that don't have a discrete set of possible values (for example, temperature readings), this is how we subscribe to changes for that attribute.

    We can then get the value of the Event in the event handler by looking at the ``value`` of the passed-in Event.
    If we were to do this in our SmartApp, it would look like this:

    .. code-block:: groovy

        def initialize() {
            subscribe(themotion, "motion", motionHandler)
        }

        def motionHandler(evt) {
            if (evt.value == "active") {
                // motion detected
            } else if (evt.value == "inactive") {
                // motion stopped
            }
        }

    Our SmartApp will use separate subscriptions and event handlers, but you are free to modify it to use a single subscription and handle the different values in your event handler method.

We need to define the ``motionStoppedHandler`` event handler method - add this method to your SmartApp:

.. code-block:: groovy

    def motionStoppedHandler(evt) {
        log.debug "motionStoppedHandler called: $evt"
        theswitch.off()
    }

Save your SmartApp in the IDE, publish it again for yourself, and then install it again in the Simulator.
Now when you change the motion to "inactive", the switch will turn off.

----

Going further--adding flexibility
----------------------------------

Our SmartApp now turns a switch on when motion is detected, then turns it off when motion stops.
But consider this scenario:

- A person enters a room, the motion sensors reports that motion is active, and our SmartApp turns the light on.
- The person then sits down, or stands still enough for the motion sensor to report motion is inactive, and our SmartApp turns the light off.
- The person than moves again, causing the motion sensor to again report active motion, and our SmartApp turns the light on again.

As you can imagine, this could be quite annoying.
It would be better if we could allow the user to specify a number of minutes *after motion stops* to turn the light off.
Then, once motion stops, if no motion is detected within the specified number of minutes, the SmartApp will turn the light off.
If motion is detected within this time window, the switch will not turn off.

We can add this flexibility into our SmartApp easily.
The first thing we need to do is update our ``preferences`` to let the user specify the number of minutes to elapse without motion being detected, before the light is turned off.

Replace the ``preferences`` in our SmartApp with the following:

.. code-block:: groovy
    :emphasize-lines: 5-7

    preferences {
        section("Turn on when motion detected:") {
            input "themotion", "capability.motionSensor", required: true, title: "Where?"
        }
        section("Turn off when there's been no movement for") {
            input "minutes", "number", required: true, title: "Minutes?"
        }
        section("Turn on/off this light") {
            input "theswitch", "capability.switch", required: true
        }
    }

Preferences inputs can be more than just devices - we can ask users to enter in numeric values, text values, booleans, enumerated lists, and more.
You can learn about the various options for preferences inputs :ref:`here <prefs_and_settings>`.

Now that the user can specify the number of minutes to wait without motion before turning the light off, we need to implement the logic to do so.

Our ``motionStoppedHandler()`` method will be called whenever the motion sensor reports that motion has stopped.
Before turning the light off, we need to check that there is no motion detected for the specified number of minutes in the future.
But since SmartApps are not continuously running, how can we handle checking for future states?
The answer is by using methods that allow us to schedule a SmartApp for future execution.

The first thing we need to do is update our ``motionStoppedHandler()`` to execute a method after the number of minutes specified by the user.
This method will then check to see if there has been motion reported within the time interval, and turn the light off if there has been no motion.

Let's write some skeleton code to do this, and we'll fill in the details later.
First, update the ``motionStoppedHandler()`` method and add a new method as shown below:

.. code-block:: groovy

    def motionStoppedHandler(evt) {
    	log.debug "motionStoppedHandler called: $evt"
        runIn(60 * minutes, checkMotion)
    }

    def checkMotion() {
        log.debug "In checkMotion scheduled method"
    }

We use the :ref:`smartapp_run_in` method to schedule our ``checkMotion()`` method to be called after the number of minutes specified by the user.
We pass ``runIn()`` the number of seconds (from the time of the call) to schedule the call, and the name of the method we want executed.

When motion stops, our ``checkMotion()`` method will be called after the number of minutes specified by the user.
Now, inside our ``checkMotion()`` method, we need to see if there has been any motion detected in the time window specified.
We can use some date/time utility methods, along with information about the device state, to determine if we should turn the switch off.

Here's the logic we need to implement:

- If the motion sensor is currently reporting active motion, do nothing.
- If the motion sensor is reporting inactive motion, check to see what time the motion sensor reported inactive motion.
- If the motion sensor reported that motion has been inactive for longer than the time specified by the user, turn the switch off.

And here's the full method definition for ``checkMotion()``.
Update your SmartApp with the code below:

.. code-block:: groovy

    def checkMotion() {
    	log.debug "In checkMotion scheduled method"

        // get the current state object for the motion sensor
    	def motionState = themotion.currentState("motion")

        if (motionState.value == "inactive") {
    		// get the time elapsed between now and when the motion reported inactive
            def elapsed = now() - motionState.date.time

            // elapsed time is in milliseconds, so the threshold must be converted to milliseconds too
            def threshold = 1000 * 60 * minutes

    		if (elapsed >= threshold) {
                log.debug "Motion has stayed inactive long enough since last check ($elapsed ms):  turning switch off"
                theswitch.off()
        	} else {
            	log.debug "Motion has not stayed inactive long enough since last check ($elapsed ms):  doing nothing"
            }
        } else {
        	// Motion active; just log it and do nothing
        	log.debug "Motion is active, do nothing and wait for inactive"
        }
    }

The first thing to note is that we get a :ref:`state_ref` object for the motion sensor, by using the ``currentState()`` method with ``"motion"`` as the attribute we're interested in.
This object encapsulates information about an attribute at a particular moment in time.
In our case, we want the current state.

From this object, we can determine when this state record was created.
This will be the time that the motion sensor reported it is inactive.
Using the :ref:`smartapp_now` method, we can get the current time (in milliseconds), and then see if the motion stopped within the threshold specified by the user.
If the time elapsed since the motion stopped exceeds the threshold, we turn the switch off.

Go ahead and save and publish your SmartApp again, and try it out!

----

Complete code listing
---------------------

Here is the entire code for our SmartApp:

.. code-block:: groovy

    definition(
        name: "My First SmartApp",
        namespace: "mygithubusername",
        author: "Peter Gregory",
        description: "This is my first SmartApp. Woot!",
        category: "My Apps",
        iconUrl: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience.png",
        iconX2Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png",
        iconX3Url: "https://s3.amazonaws.com/smartapp-icons/Convenience/Cat-Convenience@2x.png")

    preferences {
    	section("Turn on when motion detected:") {
            input "themotion", "capability.motionSensor", required: true, title: "Where?"
        }
        section("Turn off when there's been no movement for") {
            input "minutes", "number", required: true, title: "Minutes?"
        }
        section("Turn on this light") {
            input "theswitch", "capability.switch", required: true
        }
    }

    def installed() {
    	initialize()
    }

    def updated() {
    	unsubscribe()
    	initialize()
    }

    def initialize() {
    	subscribe(themotion, "motion.active", motionDetectedHandler)
        subscribe(themotion, "motion.inactive", motionStoppedHandler)
    }

    def motionDetectedHandler(evt) {
    	log.debug "motionDetectedHandler called: $evt"
        theswitch.on()
    }

    def motionStoppedHandler(evt) {
    	log.debug "motionStoppedHandler called: $evt"
        runIn(60 * minutes, checkMotion)
    }

    def checkMotion() {
    	log.debug "In checkMotion scheduled method"

    	def motionState = themotion.currentState("motion")

        if (motionState.value == "inactive") {
            // get the time elapsed between now and when the motion reported inactive
            def elapsed = now() - motionState.date.time

            // elapsed time is in milliseconds, so the threshold must be converted to milliseconds too
            def threshold = 1000 * 60 * minutes

            if (elapsed >= threshold) {
                log.debug "Motion has stayed inactive long enough since last check ($elapsed ms):  turning switch off"
                theswitch.off()
            } else {
            	log.debug "Motion has not stayed inactive long enough since last check ($elapsed ms):  doing nothing"
            }
        } else {
            // Motion active; just log it and do nothing
            log.debug "Motion is active, do nothing and wait for inactive"
        }
    }

----

How the switch turns on (or off)
--------------------------------

Now that we understand how to control devices in a SmartApp, you may be wondering how exactly the method ``switch.on()`` turns on the switch.
The answer is Device Handlers.

Device Handlers are software much the same way SmartApps are.
They define what actually happens when you call ``switch.on()``.
Let's look at an example to further understand this.

When you connect a new device to your SmartThings Hub, a Device Handler is picked for it based on the signature the device delivered to the Hub as part of its pairing communication.
The Device Handler will have methods defined in it that support that device.
So in our case, the Device Handler for the specific switch being used will have both ``on()`` and ``off()`` methods defined.
The actual implementation of these methods vary depending upon the underlying device protocols, but are typically low-level protocol-specific commands to send to the device (like Z-Wave or ZigBee).

So, when ``switch.on()`` is executed from your SmartApp, the SmartThings platform will look up the Device Handler associated with the device and call its ``on()`` method, which will in turn send the protocol and device-specific command through the Hub to the device.
Device Handlers are discussed in the :ref:`device_type_dev_guide` guide.

----

Summary
-------

In this tutorial, you learned how to write a SmartApp. To do this, we:

- Created a new SmartApp using the web-based IDE.
- Defined the ``preferences`` that specifies what input we need from the user.
- Subscribed to device Events and controlled a device. We used the :ref:`capabilities_taxonomy` to determine what attributes and commands a capability supports.
- Used the web-based Simulator to test our SmartApp with virtual devices.
- Published the SmartApp for yourself and installed it on your mobile phone.
- Extended our SmartApp by allowing a user to enter the number of minutes to wait before turning the switch off, and implemented this using the ``runIn()`` method.

----

Next steps
----------

Now that you've written your first SmartApp and have a basic understanding of the SmartThings developer tools, language, and workflow, here are some further topics for you to pursue.

More about SmartApps
^^^^^^^^^^^^^^^^^^^^

There is much more you can do with SmartApps than what this tutorial covered.
SmartApps can :ref:`send notifications <smartapp_sending_notifications>`, :ref:`execute routines <smartapp-routines>`, :ref:`define advanced schedules <smartapp-scheduling>` for which they execute, :ref:`call external web services <calling_web_services>`, and more.
You can learn more about developing SmartApps in the :ref:`smartapp_dev_guide` guide.

You can also make your SmartApp into a web service, capable of exposing its own REST endpoints.
You can read about them in the :ref:`smartapp_web_services_guide` guide.

Fork it!
^^^^^^^^

SmartThings SmartApps and Device Handlers are now hosted in GitHub.
Further, the IDE can integrate with GitHub, to provide a seamless developer experience.
Learn more about it in the :ref:`github_integration` chapter of the :ref:`tools_ide` guide. Happy forking!

Device Handler development
^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are interested in learning more about Device Handlers, and how to write one, head over to the :ref:`device_type_dev_guide` guide.

.. [1] Solution Module SmartApps are not currently available for developers, but support for this is planned in the near future.
