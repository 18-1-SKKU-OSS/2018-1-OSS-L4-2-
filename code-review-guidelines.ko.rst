=========================================
�ڵ� ���� ���̵� ���� �� ��� ���
=========================================

SmartApp �Ǵ� Device Handler�� �����ϱ� ���� ������ �ڵ尡 �� ������ ���� �� ��ħ�� �ؼ��ϴ��� Ȯ���ؾ��մϴ�.
�� ���̵� ������ �ؼ����� �ʴ� �ڵ�� ������ �� �ֽ��ϴ�.

�� ������ SmartThings ������ ���� ��� ��� �������̱⵵ �մϴ�.

----

�Ϲ����� ��Ģ
-------

�ڵ�� �б� �������մϴ�.
^^^^^^^^^^^^^^^^^^^^^^^

�ڵ�� ��谡 ����������, ����� �н��ϴ�.
�������� �ְ��� �� �� ������ ����� �� �� ���� �Ϲ����� ��ħ�� �ֽ��ϴ�:

- �ǹ� �ִ� ������ �� �޼ҵ� �̸��� ����ϼ���.
- :����:`review_guidelines_dry`
- :����:`review_guidelines_methods`
- :����:`review_guidelines_comments`

.. _review_guidelines_dry:

�ߺ����� ������
^^^^^^^^^^^^^^^^^^^^^

`DRY ��Ģ <https://en.wikipedia.org/wiki/Don%27t_repeat_yourself>`__ (don't repeat yourself �ߺ����� ����)�� �����ּ���.

�ڵ带 ����/�ٿ��ֱ� ���� ������. ���� ���̴� �ڵ带 ���� ��ƿ��Ƽ �޼ҵ�� ���ּ���.

.. _review_guidelines_methods:

�޼ҵ�� �Ѱ��� ������ �������մϴ�
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

�޼ҵ�� �Ѱ��� �������� �������Ѿ��ϸ� �����ؾ��մϴ�.
�޼����� ���ǰ� ǥ�� ��ǻ�� ȭ���� �Ѿ�ٸ�, �ʹ� ��ٴ� �̴ϴ�.

�ڵ带 ��ƿ��Ƽ �޼���� �и��� �� �ִ��� �˾ƺ�����.
���� ���, ū �������� HTTP ������ ��ø��� �м��ϴ� �޼���� ����� �� ������, �� �۾��� ȣ���� �� �ִ� ���� �޼���� �и���Ű����.
�̷��� �ϸ�, �ڵ带 �� ���� ������ �� �ְ� �Ǹ� �� ���� `���ɻ��� �и� <https://en.wikipedia.org/wiki/Separation_of_concerns>`__�� �����մϴ�.

������� �ʴ� �ڵ带 �������� ������
^^^^^^^^^^^^^^^^^^^^^^^^^

������� �ʰų� �ּ�ó�� �� �ڵ�� �����ϱ� ���� ���� �ּž��մϴ�.

������� ��, ������ ��� �Ǵ� ����ϴ� �� ������� ������
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

���� ������ �ʿ� �������� ? ���� ����ϰ� �������̾���մϴ�.

.. _review_guidelines_comments:

������ �ּ��� �޾��ּ���
^^^^^^^^^^^^^^^^^^^^^

�ּ��� �����ϰ� ���� �� �ڵ忡 ��Ȯ���� ������ ���� �� �ֽ��ϴ�.
����ġ�� ���� ���Ǹ� �ڵ尡 ���������� ���� �������ϴ�.

����� �� �� ���� ��ħ�� �ֽ��ϴ�:
- �Ϲ������� �ڵ尡 �Ϲ����� ������ �ٸ��� ����� �� �ּ��� �ʿ��մϴ�.
- Device Handler ����� Ŀ�ǵ� �� �Ӽ����� �뵵, �Ű� ���� �� ���� ���� (���� ������ ���)�� �����ϴ� �ּ��� �־���մϴ�.
- �߿��� �޼ҵ�� �� �޼ҵ尡 �ϴ� ��, ��ȯ��, ���� ���� �� �Ű� ������ �����ϴ� �ּ��� �Բ� �ۼ��ؾ��մϴ�. `JavaDoc ���� �ּ� <https://en.wikipedia.org/wiki/Javadoc#Overview_of_Javadoc>`__�� ����� �� ������ �ҽ����� ������ ���� �� �� �ִ� ������ �����ϴ�.
- �ּ��� ��ġ�� ���ؾ��մϴ� - �ڵ��� ��� �࿡ �ּ��� ���ϸ� �ڵ尡 ȥ���������� �Ӵ��� ���ʿ��� ���Դϴ�.

�޼ҵ忡 �ּ��� ������ �ۼ��� �����Դϴ�:

.. code-block:: groovy

    def capabilityCommands = getDeviceCapabilityCommands(device.capabilities)

    /**
     * Builds a map of capability names to their supported commands.
     *
     * @param a list of Capabilities.
     * @return a map of device capability -> supported commands.
    */
    def getDeviceCapabilityCommands(deviceCapabilities) {
        def map = [:]
        deviceCapabilities.collect {
            map[it.name] = it.commands.collect{ it.name.toString() }
        }
        return map
    }

Here's an example of an in-line code comment explaining why the code is checking if a percentage value is within a certain hard-coded range:
������ �ۼ�Ʈ ���� �ش���� �ȿ� �ִ��� Ȯ���ϴ� ������ ������ �ζ��� �ּ��Դϴ�:

.. code-block:: groovy

    log.trace "stopDimmersHandler evt: ${evt.value}"
    def percentComplete = completionPercentage()

// ���� ��쿡 �츮�� ���� ���� �ϴ� ���� ������ �Ѱų� ���� ���Դϴ�.
     // �׷��� �������� ���� ������ �ʵ��� �ؾ��մϴ�.
    if (percentComplete > 2 && percentComplete < 98) {
        ...

    }

�������� �ּ��� ���� ������ �����ϴ�.
�ڵ常 �о �и��� ������ �ּ��� �ܼ��� �ݺ��ϰ� �ֽ��ϴ�: ��ġ�� �������� �ʰ� �ֽ��ϴ�.

.. code-block:: groovy

    // ��� �ڽ��� �޾ƿͶ�
    def children = pollChildren()
    // ��� �ڽ��� �湮�ض�
    children.each {child ->
        // �� �ڽ��� �α׷� �����
        log.debug "child: $child"
    }

��� ``if()``�� ``switch()`` ������ Ȯ���Ͻʽÿ�
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``if ()``�Ǵ�``switch ()``����� ��� ���� �Է��� ó���ϴ��� Ȯ���Ͻʽÿ�.
Ư�� ������ ó���ϴ� ���� �ؾ������ ����ġ ���� �� ������ �߻��� �� �ֽ��ϴ�.

���� ���``switch ()``���� ��ġ�ϴ� ������ ���� ��츦 ó���ϱ� ����``default :`` ���ǹ��� �������մϴ�

������ Ȯ���ϼ���
^^^^^^^^^^^^^^^^^^

�޼ҵ尡 �Ϻ� �Է¿� �۵��� �� ���� �Ǵ� ���� SmartApp �Ǵ� Device Handler���� ȣ��Ǵ� ��츦 �����ϴ� ��� �Է� ���� ó���� �� �־���մϴ�.

�ϰ��� ��ȯ �� ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Groovy�� ���� Ÿ���� ���α׷��� ����Դϴ�.
�̴� ���� ���� ����������, �糯�� �����μ� �ſ� ȿ���������� �Ǽ��ϱ⵵ �����ϴ�.

�޼ҵ� �ñ״�ó�� �Է� ���ο� ������� �޼ҵ�� ���� �ڷ����� ��ȯ�ؾ��մϴ�.
������ ������ �����Դϴ�:

.. code-block:: groovy

    def getSomeResult(input) {
        if (input == "option1") {
            return true
        }
        if (input == "option2") {
            return false
        }
        return [name: "someAttribute", value: input]
    }

���� ������ �ϰ��� �ڷ����� ��ȯ���� �ʽ��ϴ�.
�� �ڵ��� Ŭ���̾�Ʈ�� ȣ���ϸ� �Ҹ� ���� �� ��ȯ ���� ��� �޾ƾ��մϴ�.
�̿� �ٸ��� �޼���� �׻� ������ �ڷ����� ��ȯ�ؾ��մϴ�.

.. note::

    Ư���� ���, �޼ҵ尡 �ٸ� �ڷ����� ��ȯ�ϴ� �� *�ǹ� ���� ����* �ֽ��ϴ�.
    �̷��� ���� ���� �����̸�, ��ȯ�Ǵ� �ڷ������ � ��Ȳ���� �� �ڷ����� ��ȯ�Ǵ� ���� �ּ��� �ۼ��Ǿ� �־�� �մϴ�.


�迭 �ε����� �����ϼ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

�����͸� �Ľ� �� �� �迭�� ����Ѵٸ� �����ϼž��մϴ�.
�迭�� �ε����� �� ������ �迭�� �׸�ŭ�� ���Ұ� �����ϴ��� ���� Ȯ���ؾ��մϴ�.

������ `` ":"`���ڸ� �������� ���ڿ��� �����ϰ�`` ":"`` ���� ������ ���� ���� ��ȯ�ϴ� �ڵ��Դϴ�:

.. code-block:: groovy

    def getSplitString(input) {
        return input.split(":")[1]
    }

    // -> "123"
    getSplitString("abc:123")

    // -> ArrayIndexOutOfBounds exception!
    getSplitString("abc:")

``getSplitString ()``��``split ()`split�� ����� �ϳ� �̻��� ���Ҹ� ������ �ִ����� �������� �ʱ� ������, �Ľ� �� ������� �ι�° �׸� �����Ϸ� �� ��``ArrayIndexOutOfBounds`` ���ܰ� �߻��մϴ�.
�̿� ���� ��� �迭�� �׸��� �ִ��� Ȯ���ϴ� ����ó���� ������մϴ�.

.. code-block:: groovy

    def getSplitString(input) {
        def splitted = input?.split(":")
        if (splitted?.size() == 2) {
            return splitted[1]
        } else {
            return null
        }
    }

���� �����ڸ� �ٸ��� ����ϼ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

�׷��� ���� �����ڸ� �����մϴ�. ���� �����ڸ� ����ϸ� �� �����ϰ� ���ǹ��� �ۼ��� �� �ֽ��ϴ�.
�׷���, �̸� ȿ�������� ����Ϸ��� :ref:`Groovy truth <review_guidelines_groovy_truth>` �� �����ؾ��մϴ�.

����``bulbLevel``�� �������� ���� ��� �� ���� ``100``���� �����ϴ� �����Դϴ�:

.. code-block:: groovy

    def bulbLevel = settings.level ?: 100

�׷��� ���� ��������``settings.level``��``0``�̶�� ��� �ɱ��? ** �׷��� 0�� false�� �����ϱ� ������ **``bulbLevel`` **�� **``100`` **! **���� �����߽��ϴ�! **! **

���� �ڵ�� ������ ���� �ۼ��Ǿ���մϴ�:

.. code-block:: groovy

    def bulbLevel = settings.level == null ?: 100


Null �� ó��
^^^^^^^^^^^^^^^^^^

.. �߿�::

     NullPointerExceptions�� SmartThings �÷������� ���� ���� �߻��ϴ� ���� �� �ϳ��Դϴ�. �������ּ���!

   LAN�� SSDP ��ȣ �ۿ뿡�� * �ſ� * ���� �Ͼ�� ���̹Ƿ� �׻� �ڵ带 �ѹ� �� Ȯ�����ּ���.

``NullPointerException``�� SmartApp �Ǵ� Device Handler�� ������ ���� ��Ű���� `������ �׺���̼�<http://groovy-lang.org/operators.html#_safe_navigation_operator>`__ (``?`` ) �����ڷ� ���� ó���� �� �ֽ��ϴ�.
``null`` ���� ���� �� �ִ� ��� �ڵ�� �̸� �̸� ó���ؾ��մϴ�.

�Ʒ� ������``null``�� ������ �� ���� ���� �߻��ϴ� ���� ``?`` �����ڸ� ����Ͽ� �װ��� ó���ϴ� ����� �����ݴϴ� :

.. code-block:: groovy

    // LAN �̺�Ʈ�� ��� �Ǵ� "content-type"����� ���� ���, 
    // NullPointerException�� ������ ������!
    if (lanEvent.headers?."content-type"?.contains("xml")) { ... }

.. code-block:: groovy

    // ��ġ�� ��尡 ���� ��� �ڵ�� null�� ��ȯ�մϴ�.
    // �׷��� NullPointerException�� throw���� �ʽ��ϴ�. 
    if (location.modes?.find{it.name == newMode}) { ... }


.. _review_guidelines_groovy_truth:

�׷�� ���� ���� �ùٸ��� ����ϼ���
^^^^^^^^^^^^^^^^^^^^^^^^^^ 

Groovy�� �� �Ǵ� �������� �����ϴ� ���� �ϰ������� �����ϴ��� Ȯ���ϼ���.
�׷���� �� ���� ���� ������ `���� http://groovy-lang.org/semantics.html#Groovy-Truth>`__�� �ۼ��Ǿ��ֽ��ϴ�.

�˰� �־�� �� �� ���� �������� �Դϴ�:

- �� ���ڿ��� ``����``���� ���ֵ˴ϴ�; ��� ���� ���� ���ڿ��� ``��``���� ���ֵ˴ϴ�.
- �� �ʰ� ����Ʈ�� ``����``���� ���ֵ˴ϴ�; ��� ���� ���� �ʰ� ����� ``��``�� ���ֵ˴ϴ�.
- 0�� ``����``���� ���ֵ˴ϴ�. 0�� �ƴ� ���ڴ� ``��``���� ���ֵ˴ϴ�.

���ڰ� 0�� 100 ���̿� �����ϴ��� Ȯ���ϴ� �����Դϴ�:

.. code-block:: groovy

    def verifyLevel(level) {
        if (!level) {
            return false
        } else {
            return (level >= 0 && level <= 100)
        }
    }

�׷�񿡼� ``0``�� �����̱� ������``verifyLevel (0)``�� ȣ���ϸ� �����``false``�Դϴ�.
�׷��� �Ʒ��� ���� �ۼ��ؾ��մϴ�:

.. code-block:: groovy

    def verifyLevel(level) {
        return (level instanceof Number && level >= 0 && level <= 100)
    }

���� �Ͼ�� �����̱� ������ �׷���� ���� ���� �� �����ϰ� �����ϰ� ����ؾ��մϴ�. 

----

State �Լ��� ���
-----------

``state`` �� ������ �����ͺ��̽��� �ƴմϴ�
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

state�� ������ �� �ִ� �������� ���� :����:`limited <state_size_limit>`�Դϴ�.
�ֱ�������(�̺�Ʈ�� �������� ��������) ``state``�� ���Ҹ� �߰������� �������� �ʴ� �ڵ�� �����ϼ���. 

``state``�� �۵���� ����
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``state``�� ����� ��, :����:`����� ���� <state_how_it_works>�� ������ ��ĥ ������ ���ӵ��� �ʽ��ϴ�`.
���ÿ� ����Ǵ� �ٸ� SmartApp �ν��Ͻ��� state ���� �������̵��ϴ� ���ó��, �ǵ����� ���� ����� �߻��� �� �ֽ��ϴ�.

���� ``atomicState`` �� ``state``�� ����ؾ��ϴ��� �˾Ƶμ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``atomicState``��``state``�� :����:`���� <choose_between_state_atomicState>`�� �����Ͽ� �ʿ信 �°� �ùٸ� ���� ����ϰ� �� SmartApp�� �� ���� ��� ����ϴ� ���� �������ּ���.

Collection�� ``atomicState``�� ������ �� �����ϼ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Atomic State������ Collection ������ State������ ���������� ���� �ʽ��ϴ�.
Atomic State�� ����� collection�� ������ �۾� ����� ������ :����:`<atomic_state_collections> ������ �о��ּ���.`.

----

�� ����
------------

�ܺ� HTTP ��û ����ȭ
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

�ܺ� ���񽺷��� :����:`HTTP ��û <calling_web_services>`�� �ܺ� ��û�� ���� �ʿ伺, � �����͸� �����ߴ���, �� �����Ͱ� ��� ������ ���� ��ϵǾ���մϴ�.
�ش��ϴ� ��� ��3���� ���� ���� ��� ��ħ�� ���� ��ũ�� �ּ����� �޾��ּ���.

����� ��� ���� ����Ʈ ����ȭ 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

SmartApp �Ǵ� Device Handler�� :����:`���� ����Ʈ�� �ϳ��� �����ϴ� ��� <web_services_mapping_endpoints>`, API�� ���� ���, API���� �׼��� �ϴ� ������ �� ������ ��쿡 ���� �ּ��� �ۼ��ϰ� API�� ������ �� �ִ� ���� ������ ���� ���� ��ȣ ��å�� ���� ��ũ�� �Բ� �����ּ���.

----

�����ٸ�
----------

��Ǯ�̵Ǵ� ª�� �������� �����ϼ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

�����ٵǰų� �ֱ��� �Լ��� �߿��� ������ �ְų� �����ڰ� �������� �ʴ� �̻� 5�п� �ѹ� �̻� ������� ���ƾ� �մϴ�.

�ڵ尡 5�п� �ѹ� �̻� ����Ǵ� ��� �� �ڵ尡 �ʿ��� ������ �ּ��� �߰����ּ���.

``runIn()`` �޼��� ü�̴��� �����ϼ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

:����:`runIn() �޼��带 ü�̴����� ������ <scheduling_chained_run_in>`
 
�� �ʿ��ϴٸ� �� ������ �����ϴ� �ּ��� �߰����ּ���.

----

���� �������
-----------------------

������ ��Ȯ�ؾ��մϴ�
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

���ڿ� ������ ����Ͽ� �̺�Ʈ�� ������ �� �����Ƿ� SmartApp�� �����ϴ� ����� �ټ� ������ �� �� �ֽ��ϴ�.

��:

.. code-block:: groovy

    def myContactSubscription = "contact.open"

    ...

    subscribe(contact1, myContactSubscription, myContactHandler)

���� ���� ����� �Ӽ��� ��������� �����ϴ� ���Դϴ�:

.. code-block:: groovy

    subscribe(contact1, "contact.open", myContactHandler)

�׷��� SmartApp�� ����(���� ���, state����)�� �����ؾ��ϴ� ���, �����ڴ� ������ �����Ǵ� ��İ� ����Ǵ� Ư���� �������� ������ �� �־���մϴ�.

������ ��ü���̾�� �մϴ�
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

����ġ�� �������� ������ ������ ���ʽÿ�.

��� ��ġ�� �̺�Ʈ�� ������ SmartApp�� ����ġ�� ���� ����Ǹ� �̷� ���� ���� �ʿ����� �ʽ��ϴ�.
��� �����ִ� �̺�Ʈ�� ��ü���� ������ ����ʽÿ�.

LAN ���� ��ġ �� ���� �����ڸ� ����� ��� :����:`��ġ �˻� ��� <lan_device_discovery> ����'�� �� ���ּ���.

���� �޼���� �������� ������
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

�׷�񿡼��� ������ ���� ���ڿ��� ������� �Լ��� ������ �� �ֽ��ϴ�.

.. code-block:: groovy

    object."${mystring}"()

�ſ� �� �� �� ������, ``$ {mystring}``�� HTTP ��û, �� SmartThings �÷��� �ܺ� �Ǵ� �ٸ� SmartApp �Ǵ� Device Handler���� �� ��� �Է��� �����ؾ��մϴ�.

���� ���� ����� �Է��� ����ϱ� �� ``switch()``���� ����ϴ� ���Դϴ�:

.. code-block:: groovy

    switch(mystring) {
        case "cmd1":
            object.cmd1()
            break
        case "cmd2":
            object.cmd2()
            break
        case "cmd3":
            object.cmd3()
            break
        default:
            return "ERROR"
    }


SMS �޽����� �ϵ� �ڵ����� ������
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Notifications should never be sent to a hard-coded number.
They should always use a number provided by the user using the :ref:`contact input <contact_book>` (even though Contact Book is not enabled, the contact input type is available and contains a fall-back mechanism for non-Contact Book users. Using this future-proofs your SmartApp).
�˸��� ����� �ϵ� �ڵ�� ��ȣ�� �����ϸ� �ȵ˴ϴ�.
:����:`����ó �Է� <contact_book>`���� ����ڰ� ���� �� ��ȣ�� ����ؾ��մϴ� (Contact Book�� ����� �� ���, ����ó �Է����� ����� �� ������ ����ó ����ڰ� �ƴ� ������� ���� fall-back ��Ŀ������ �����մϴ�) �� ����� ����ϸ� SmartApp�� �����˴ϴ�.

----

����
-----------

�ݺ����� �ٻ� ��⸦ ����������
^^^^^^^^^^^^^^^^^^^^^

�ݺ������� �ٻ� ��⸦ �ɾ�� �� �ϸ��� ������ �����ϴ�.
�̷��� ���� ������:

.. code-block:: groovy

    def mywait(ms) {
        def start = now()
        while (now() < start + ms) {
            // do nothing, just wait
        }
    }

�� �ڵ��� ������ �� �и� �� ���� ������ ������Ű�� ���Դϴ�.
�̷��� �ϸ� ���ҽ��� ����ǰ� 20�� ���� ������ �ʰ��� ���ɼ��� �������ϴ�.

���� ���� ���, �ۿ��� ���Ŀ� ����� ���� :����:`�����ٸ� <smartapp-scheduling>`�ؾ��մϴ�.

``synchronized()``�� ���� ������
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``synchronized``�� ����ϸ� ���� ���� ��尡 �߻��ϰ� �ƹ��� ������ ���� �ʽ��ϴ�.
������� ������.

When a SmartApp or Device Handler executes, it is executing on one of *n* available servers assigned for that Location, where *n* is variable depending on Location, current load, and other factors.
Concurrent executions of the SmartApp or Device Handler are not guaranteed, or even likely, to be executing on the same server.
Because of this, trying to force synchronous behavior by using ``synchronized`` would only work in the rare occurrence that a concurrent execution happens on the same server, yet it always incurs overhead.
SmartApp �Ǵ� Device Handler�� ����Ǹ� �ش� ��ġ�� �Ҵ�� * n * ���� ��� ������ ���� �� �ϳ����� ����˴ϴ�. ���⼭ * n *�� ��ġ, ����ε� �� ��Ÿ ��ҿ� ���� �ٸ��ϴ�.
SmartApp �Ǵ� Device Handler�� ���� ������ ������ �������� ����Ǵ� ���� ������� �ʰų� ���ɼ��� �ֽ��ϴ�.
�� ������``synchronized``�� ����Ͽ� ���� ������ ���� �����Ϸ��� �ϸ� ������ �������� ���� ������ �߻��ϴ� �幮 ��쿡�� �۵������� ���� ��尡 �׻� �߻��մϴ�

----

LAN ����
------------

��ġ �� �˻� ���
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

LAN�� ����� ��ġ�� ���� �����ڴ� ��ġ �˻��� ���� :����: ` ��ġ �˻� ��� <lan_device_discovery>`�� �����ϼ���.

IP ���� ó��
^^^^^^^^^^^^^^^^

Service managers for LAN-connected devices should :ref:`handle any IP change <lan_device_health>`.
This can happen when the router power cycles and loses its DHCP mappings.
LAN�� ����� ��ġ�� ���� �����ڴ� :����:`IP ���� <lan_device_health>`�� ó���ؾ��մϴ�.
�̴� ����� ������ ��ȯ�ǰ� DHCP ������ �սǵ� �� �߻��� �� �ֽ��ϴ�.

----

.. _review_guidelines_parent_child:

�θ� - �ڽ� ����
--------------------------

������ ������ ����ϼ���
^^^^^^^^^^^^^^^^^^

�θ� - �ڽ� ���踦 ����ϴ� ��� �ڽ� ��ġ�� �ִ� �θ� SmartApp �̰ų� �ڽ� SmartApp�� �ִ� �θ� SmartApp�̾���մϴ�. �θ�� �ڽ��� ������ ���Ͽ� �־���մϴ�.

��, ���� �ڵ带 ���� ���Ͽ� �θ� ���� ũ�Ⱑ Ŀ���� �ڵ带 �����ϱ� ��ư� ����� ������ �߻��ϱ� ����� ������ϱⰡ ��ƽ��ϴ�.

