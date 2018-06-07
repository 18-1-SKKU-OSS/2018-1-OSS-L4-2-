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
