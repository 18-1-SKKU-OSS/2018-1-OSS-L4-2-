.. _calling_web_services:

Making Synchronous External HTTP Requests
=========================================

SmartApps or Device Handlers may need to make calls to external web services. There are several APIs available to you to handle making these requests.

The various APIs are named for the underlying HTTP method they will use. ``httpGet()`` makes an HTTP GET request, for example.

.. note::

    The APIs discussed here are executed synchrously, within a single SmartApp or Device Handler execution.

    For information on making asynchronous HTTP requests, check out the :doc:`async-http` documentation.

----

HTTP methods
------------

The following methods are available for making HTTP requests.
You can read more about each of them in the :ref:`smartapp_ref` API documentation.

These methods execute synchrously, and there is a 10 second timeout limit for the response to be received.

============================== ================
Method                         Description
============================== ================
:ref:`smartapp_http_delete`    Executes an HTTP DELETE request
:ref:`smartapp_http_get`       Executes an HTTP GET request
:ref:`smartapp_http_head`      Executes an HTTP HEAD request
:ref:`smartapp_http_post`      Executes an HTTP POST request
:ref:`smartapp_http_post_json` Executes an HTTP POST request with JSON Content-Type
:ref:`smartapp_http_put_json`  Executes an HTTP PUT request with JSON Content-Type
============================== ================

Here's a simple example of making an HTTP GET request:

.. code-block:: groovy

    def params = [
        uri: "http://httpbin.org",
        path: "/get"
    ]

    try {
        httpGet(params) { resp ->
            resp.headers.each {
               log.debug "${it.name} : ${it.value}"
            }
            log.debug "response contentType: ${resp.contentType}"
            log.debug "response data: ${resp.data}"
        }
    } catch (e) {
        log.error "something went wrong: $e"
    }

----

Configuring the request
-----------------------

The various APIs for making HTTP requests all accept a map of parameters that define various information about the request:

=================== ==============
Parameter           Description
=================== ==============
uri                 Either a URI or URL of of the endpoint to make a request from.
path                Request path that is merged with the URI.
query               Map of URL query parameters.
headers             Map of HTTP headers.
contentType         Request content type and Accept header.
requestContentType  Content type for the request, if it is different from the expected response content-type.
body                Request body that will be encoded based on the given contentType.
=================== ==============

.. note::

    Specifying a ``requestContentType`` may override the default behavior of the various http API you are calling.
    For example, ``httpPostJson()`` sets the ``requestContentType`` to ``"application/json"`` by default.

----

Handling the response
---------------------

The HTTP APIs accept a closure that will be called with the response information from the request.

The closure is passed an instance of a `HttpResponseDecorator <https://github.com/jgritman/httpbuilder/blob/855e1784be8585de81cc3c99fd19285798c7bc4f/src/main/java/groovyx/net/http/HttpResponseDecorator.java>`__.
You can inspect this object to get information about the response.

Here's an example of getting various response information:

.. code-block:: groovy

    def params = [
        uri: "http://httpbin.org",
        path: "/get"
    ]

    try {
        httpGet(params) { resp ->
            // iterate all the headers
            // each header has a name and a value
            resp.headers.each {
               log.debug "${it.name} : ${it.value}"
            }

            // get an array of all headers with the specified key
            def theHeaders = resp.getHeaders("Content-Length")

            // get the contentType of the response
            log.debug "response contentType: ${resp.contentType}"

            // get the status code of the response
            log.debug "response status code: ${resp.status}"

            // get the data from the response body
            log.debug "response data: ${resp.data}"
        }
    } catch (e) {
        log.error "something went wrong: $e"
    }


.. tip::

    Any 'failed' response will generate an exception, so you should wrap your calls in a try/catch block.

If the response returns JSON, ``data`` will be in a map-like structure that allows you to easily access the response data:

.. code-block:: groovy

    def makeJSONWeatherRequest() {
        def params = [
            uri:  'http://api.openweathermap.org/data/2.5/',
            path: 'weather',
            contentType: 'application/json',
            query: [q:'Minneapolis', mode: 'json']
        ]
        try {
            httpGet(params) {resp ->
                log.debug "resp data: ${resp.data}"
                log.debug "humidity: ${resp.data.main.humidity}"
            }
        } catch (e) {
            log.error "error: $e"
        }
    }

The ``resp.data`` from the request above would look like this (indented for readability):

.. code-block:: bash

    resp data: [id:5037649, dt:1432752405, clouds:[all:0],
        coord:[lon:-93.26, lat:44.98], wind:[speed:4.26, deg:233.507],
        cod:200, sys:[message:0.012, sunset:1432777690, sunrise:1432722741,
            country:US],
        name:Minneapolis, base:stations,
        weather:[[id:800, icon:01d, description:Sky is Clear, main:Clear]],
        main:[humidity:73, pressure:993.79, temp_max:298.696, sea_level:1026.82,
            temp_min:298.696, temp:298.696, grnd_level:993.79]]

We can easily get the humidity from this data structure as shown above:

.. code-block:: groovy

    resp.data.main.humidity

----

Host and timeout limitations
----------------------------

Host and IP address restrictions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Requests can only be made to publicly accessible hosts.
Remember that when executing an HTTP request, the request originates from the SmartThings platform (i.e., the SmartThings cloud), not from the hub itself.

Requests made to local or private hosts are not allowed, and will fail with a ``SecurityException``.

Request timeout limit
^^^^^^^^^^^^^^^^^^^^^

Requests will timeout after 10 seconds.

Because the request is executed synchronously within a single execution, we encourage you to check out the new (currently beta) :doc:`async-http` feature.

----

Try it out
----------

If you're interested in experimenting with the various HTTP APIs, there are a few tools you can use to try out the APIs without signing up for any API keys.

You can use `httpbin.org <http://httpbin.org/>`__ to test making simple requests.
The ``httpGet()`` example above uses it.

For testing POST requests, you can use `PostCatcher <http://postcatcher.in/>`__.
You can generate a target URL and then inspect the contents of the request.
Here's an example using ``httpPostJson()``:

.. code-block:: groovy

    def params = [
        uri: "http://postcatcher.in/catchers/<yourUniquePath>",
        body: [
            param1: [subparam1: "subparam 1 value",
                     subparam2: "subparam2 value"],
            param2: "param2 value"
        ]
    ]

    try {
        httpPostJson(params) { resp ->
            resp.headers.each {
                log.debug "${it.name} : ${it.value}"
            }
            log.debug "response contentType: ${resp.    contentType}"
        }
    } catch (e) {
        log.debug "something went wrong: $e"
    }

----

See also
--------

A simple example using ``httpGet()`` that connects a SmartSense Temp/Humidity Sensor to your Weather Underground personal weather station can be found `here <https://github.com/SmartThingsCommunity/Code/blob/e8a6b6926fb32df1e8d79bfe09a1ad063682396a/smartapps/wunderground-pws-connect.groovy>`_.

You can browse some templates in the IDE that use the various HTTP APIs. The Ecobee Service Manager is an example that uses both ``httpGet()`` and ``httpPost()``.
