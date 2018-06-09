.. _parent_child_smartapps:

부모-자식 SmartApp
======================

SmartApp은 자식 SmartApp을 가질 수 있습니다. 개별 디바이스에서 독립적으로 작동하는 여러 자동화를 제공하고 싶을 때 이 개념은 유용합니다. 이것은 한 부모에서 여러 개별 자동화를 통합합니다.

----

개요
--------

Smart Lighting은 부모-자식 SmartApp의 한 예제입니다.
Smart Lighting을 설치하면 하나의 부모 SmartApp(Smart Lighting)가 설치되고, 실제로 생성하는 각 고유한 자동화 조명은 자식 SmartApp의 새로운 인스턴스입니다.
이 자식 SmartApp은 실제로 각 자동화 조명을 제어합니다.

아래 도표는 이 관계를 보여줍니다.

.. image:: ../../img/smartapps/smart-lighting-diagram.png

부모 SmartApp과 그의 자식 SmartApp 사이의 관계는 일대다 관계입니다.
SmartApp은 여러 자식 SmartApp을 가질 수 있고, 이러한 자식은 또 자신의 자식 SmartApp을 가질 수 있습니다.
자식 SmartApp은 오직 하나의 부모 SmartApp만 가질 수 있습니다.

----

.. _parent_child_smartapp_parent:

부모 SmartApp
-------------

다른 SmartApp의 부모 SmartApp으로 정의하려면, ``preferences`` 의 ``app`` 입력 값을 사용하세요. ``app`` 입력 값은 사용자가 자식 SmartApp 인스턴스를 설치 및 편집하고, 부모와 자식 사이의 관계를 설정할 수 있도록 합니다.

.. code-block:: groovy

    preferences {
        page(name: "mainPage", title: "Child Apps", install: true, uninstall: true) {
            section {
                app(name: "childApps", appName: "Child App", namespace: "mynamespace", title: "New Child App", multiple: true)
            }
        }
    }

``app`` 입력 값을 통해 설치된 모든 자식 SmartApp은 부모 SmartApp의 기본 설정 페이지에 목록화되며, 사용자는 이 인스턴스를 편집하거나 삭제할 수 있습니다.

``app`` 입력 값의 옵션은 다음과 같습니다.

========= ===========
Option    Description
========= ===========
name      입력 값의 이름. 입력 값의 식별자 역할을 합니다.
appName   자식의 메타데이터에 정의된 자식 SmartApp의 이름
namespace 자식의 메타데이터에 정의된 자식 SmartApp의 이름공간
title     사용자가 자식 SmartApp의 새로운 인스턴스를 설치하기 위해 누르는 버튼의 제목
multiple  ``true`` 일 경우, 사용자는 여러 자식 SmartApp을 설치할 수 있습니다. ``false`` 일 경우, 오직 하나의 자식 SmartApp만 설치됩니다. 기본 값은 ``false`` 입니다.
========= ===========

----

자식 SmartApp
-------------

자식으로 사용하고 싶은 SmartApp에서 자식 SmartApp의 ``definition`` 의 ``parent`` 옵션을 ``"namespace":"Parent App Name"`` 의 형태로 지정하세요.

.. code-block:: groovy

    definition(
        ...
        parent: "yourNameSpace:Parent App Name",
        ...
    )

.. note::

	자식 SmartApp을 저장하면 플랫폼은 ``parent`` 옵션에서 지정된 이름공간과 이름이 존재하는지 확인합니다. 존재하지 않으면 에러가 발생합니다.

	부모 SmartApp이 저장되어있는지 먼저 확인하거나 부모 SmartApp을 저장한 후에 돌아와서 ``parent`` 옵션을 추가하시길 바랍니다.

----

부모와 자식 사이 의사소통
-----------------------

부모와 자식 SmartApp은 서로 소통해야할 때가 있습니다. 부모 SmartApp에서 ``getChildeApps()`` 메소드를 사용해 자식 SmartApp을 얻을 수 있습니다.

.. code-block:: groovy

    def children = getChildApps()
    log.debug "$children.size() child apps installed"
    children.each { child ->
        log.debug "Child app id: $child.id"
    }

이후 자식 SmartApp에서 바로 메소드를 호출할 수 있습니다.

.. code-block:: groovy

    // 자식 SmartApp이 foo() 메소드를 정의했다 가정합니다
    child.foo()

SmartApp의 이름으로 특정 자식 SmartApp을 찾기 위해 ``findChildAppByName()`` 메소드를 사용할 수 있습니다.

.. code-block:: groovy

    def theChild = findChildAppByName("My Child App")

자식 SmartApp은 자신의 ``parent`` 속성을 사용해 부모 SmartApp과 소통할 수 있습니다.

.. code-block:: groovy

    // 부모 SmartApp이 bar() 메소드를 정의했다 가정합니다
    parent.bar()

----

둘 이상의 부모 인스턴스 방지하기
-----------------------------

사용자가 자신의 위치에 둘 이상의 부모 SmartApp을 설치하는 것을 방지하려면, definition에 ``singleInstance: true`` 을 지정해줄 수 있습니다.

.. code-block:: groovy

    definition(
        ...
        singleInstance: true
        ...
    )

``singleInstance: true`` 을 지정해주면 사용자가 이미 설치되어 있는 부모 SmartApp을 설치하려 할 때 기존의 설치된 앱으로 이동합니다.
그곳에서 기존 자식 SmartApp을 설정하거나 새로운 자식 SmartApp을 추가할 수 있습니다.
이 방법은 부모 SmartApp이 하나만 필요할 때 여러 인스턴스를 생성되는 것을 방지할 수 있습니다.

----
