.. _anatomy-and-lifecycle-of-a-smartapp:

Anatomy and Life Cycle of a SmartApp
====================================

SmartApps are applications that allow users to tap into the capabilities of
their devices to automate their lives.
Most SmartApps are installed by the user via the SmartThings mobile client application. In addition, a few pre-installed SmartApps are readily available in the SmartThings system out-of-the-box.

----

Types of SmartApps
------------------

Generally speaking, there are three different kinds of SmartApps: *Event-Handlers*, *Solution Modules*, and *Service Managers*.
If you are familiar with back-end web development, then you will be more than capable of developing SmartApps.

Event Handler SmartApps
^^^^^^^^^^^^^^^^^^^^^^^

Event Handler SmartApps are the most common apps developed by our
community.
They allow you to subscribe to Events from devices and call a
handler method upon their firing.
This method can then do a variety of
things, most commonly invoking a command on another device.


A very simple example of an Event-Handler SmartApp would involve you walking through a
door and having the lights turn on automatically.

Solution Module SmartApps
^^^^^^^^^^^^^^^^^^^^^^^^^

These apps exist within the dashboard of the SmartThings app interface,
and are containers for other SmartApps.
The idea behind Solution Module
SmartApps is to combine SmartApps that, in the real world, intuitively
go together.
One example of this would be the "Home & Family" section of
the dashboard which allows you to see the comings and goings of your
family.

Service Manager SmartApps
^^^^^^^^^^^^^^^^^^^^^^^^^

Service Manager SmartApps are used to connect to LAN or cloud devices,
such as a Sonos or a WeMo device.
These SmartApps are the connecting glue between the unique protocols of such LAN or cloud devices and a Device Handler you would create for such devices.
These Service Manager SmartApps discover LAN or cloud devices and then continue to maintain their connection.

The Service Manager SmartApp must be installed when a user utilizes a
device using LAN or the cloud. So, for example, there is a Sonos Service
Manager SmartApp that is installed when pairing with a Sonos device.

----

SmartApp structure
------------------

SmartApps take the form of a single `Groovy <http://groovy.codehaus.org/>`__ script.
A typical SmartApp script is composed of four sections: *Definition*, *Preferences*, *Predefined Callbacks*, and *Event Handlers*.
There is also a *Mappings* section that is required for Cloud-connected SmartApps that will be described later.

.. image:: ../img/smartapps/demo-app.png
    :class: with-border

Definition
^^^^^^^^^^

The *defintion* section of the SmartApp specifies the name of the app along with other information that identifies and describes it.

Preferences
^^^^^^^^^^^

The *preferences* section is responsible for defining the screens that appear in the mobile app when a SmartApp is installed or updated.
These screens allow the user to specify which devices the SmartApp interacts with along with other configuration options that affect its behavior.

Pre-defined callbacks
^^^^^^^^^^^^^^^^^^^^^

The following methods, if present, are automatically called at various times during the lifecycle of a SmartApp:

#. ``installed()`` - Called when a SmartApp is first installed.
#. ``updated()`` - Called when the preferences of an installed SmartApp is updated.
#. ``uninstalled()`` - Called when a SmartApp is uninstalled.
#. ``childUninstalled()`` - Called for the parent app when a child app is uninstalled (a SmartApp can have child SmartApps).

The ``installed()`` and ``updated()`` methods are commonly found in all apps.
Since the selected devices may have changed when an app is updated, both of these methods typically set up the same Event subscriptions, so it is common practice to put those calls in an ``initialize()`` method and call it from both the installed and updated methods.

The ``uninstalled()`` method is typically not needed since the system automatically removes subscriptions and schedules when a SmartApp is uninstalled.
However, they can be necessary in apps that integrate with other systems and need to perform cleanup on those systems.

Event Handlers
^^^^^^^^^^^^^^

The remainder of the SmartApp contains the event handler methods specified in the Event subscriptions and any other methods necessary for implementing the SmartApp.
Event handler methods must have a single argument, which contains the
:ref:`event_ref` object.

----

SmartApp execution
------------------

SmartApps aren't always running.
Their various methods are executed when external Events occur.
SmartApps execute when any of the following types of Events occur:

1. **Pre-defined callback** - Any of the predefined lifecycle Events described above occur.
2. **Device state change** - An attribute changes on a device, which
   creates an Event, which triggers a subscription, which calls a
   handler method within your SmartApp.
3. **Location state change** - A location attribute such as *Mode* changes. *Sunrise* and *sunset* are other examples of location events.
4. **User action on the app** - The user taps a SmartApp icon or shortcut in the mobile app UI.
5. **Scheduled event** - Using a method like runIn(), you call
   a method within your SmartApp at a particular time.
6. **Web services call** Using our `web services
   API <../../smartapp-web-services-developers-guide/overview.rst>`__, you
   create an endpoint accessible over the web that calls a method within
   your SmartApp.

----

Device preferences
------------------

The most common type of input in the preferences section specifies what kind of devices a SmartApp works with.
For example, to specify that an app requires one contact sensor:

.. code-block:: groovy

    input "contact1", "capability.contactSensor"

This will generate an input element in the mobile UI that prompts for the selection of a single contact sensor (``capability.contactSensor``).
``contact1`` is the name of a variable that provides access to the device in the SmartApp.

Device inputs can also prompt for more than one device. So to ask for the selection of one or more switches:

.. code-block:: groovy

    input "switch1", "capability.switch", multiple: true

You can find more information about SmartApp preferences `here <preferences-and-settings.html>`__.

----

Event subscriptions
-------------------

Subscriptions allow a SmartApp to listen for Events from devices, or from a Location, or from the SmartApp tile in the mobile UI.
Device subscriptions are the most common and take the form:

.. code-block:: groovy

    subscribe(<device>, "<attribute[.value]>", handlerMethod)

For example, to subscribe to all Events from a contact sensor you would write:

.. code-block:: groovy

    subscribe(contact1, "contact", contactHandler)

The ``contactHandler()`` method would then be called whenever the sensor opened or closed.
You can also subscribe to specific Event values, so to call a handler only when the contact sensor opens write:

.. code-block:: groovy

    subscribe(contact1, "contact.open", contactOpenHandler)

The ``subscribe()`` method call accepts either a device or a list of devices, so you don't need to explicitly iterate over each device in a list when you specify ``multiple: true`` in an input preference.

You can learn more about subscribing to device Events in the :ref:`events_and_subscriptions` section.

----

SmartApp sandboxing
-------------------

SmartApps are developed in a sandboxed environment.
The sandbox is a way to limit developers to a specific subset of the Groovy language for performance and security.
We have :ref:`documented <groovy-for-smartthings>` the main ways this should affect you.

----

Execution location
------------------

With the original SmartThings Hub, all SmartApps execute in the SmartThings cloud.
With the new Samsung SmartThings Hub, certain SmartApps may run locally on hub or in the SmartThings cloud.
Execution location varies depending on a variety of factors, and is managed by the SmartThings internal team.

As a SmartThings developer, you should write your SmartApps to satisfy their specific use cases, regardless of where the app executes.
There is currently no way to specify or force a certain execution location.
