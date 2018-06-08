.. _cloud_connected_service_manager:

Building the Service Manager
============================

The Service Manager's responsibilities are:

- To authenticate with the third-party cloud service.
- Device discovery.
- Add/Change/Delete device actions.
- Handle sending any messages that require the authentication obtained.

Below we will get into the details of what is outlined above.
First, let's see an illustration of it in a SmartThings application using the Ecobee Thermostat.

.. _cloud_service_manager_oauth:

Authentication using OAuth
--------------------------

Any Service Manager authenticated with a third party via OAuth must itself have OAuth enabled.
This is because eventually the third-party service will call back into the SmartApp and hit the ``/oauth/initialize`` and ``/oauth/callback`` endpoints.

End user experience
~~~~~~~~~~~~~~~~~~~

As an end user you start by selecting the Service Manager SmartApp for Ecobee Thermostat from the SmartApps screen of the SmartThings mobile app.

Authorization with the third party is the first part of the
configuration process.
You will be taken to a page that describes how the authorization process works.

.. figure:: ../../img/device-types/cloud-connected/building-cloud-connected-device-types/click-to-login.png

From this screen you will then be directed to the third-party site, i.e., the Ecobee Thermostat site in this case, embedded within the SmartThings mobile application.
Here you will enter your Ecobee Thermostat service username and password.

.. figure:: ../../img/device-types/cloud-connected/building-cloud-connected-device-types/ecobee-login.png

Next this third-party Ecobee server will show you what access permissions SmartThings will have to your Ecobee account.
It also gives you an opportunity to accept or decline.

.. figure:: ../../img/device-types/cloud-connected/building-cloud-connected-device-types/authorize-ecobee.png

After you accept, on the following screen you will finish the configuration by tapping on the *Done* icon on the top right.

.. figure:: ../../img/device-types/cloud-connected/building-cloud-connected-device-types/ecobee-authorization-complete.png

Next, you will be taken back to the initial configuration screen where you select your device to complete the installation.

Implementation
~~~~~~~~~~~~~~

OAuth is an industry standard for authentication. However, the third-party service may use a different standard.
In that case, consult their documentation and implement it.
The basic concepts will be similar to that of OAuth.
The below example will walk through what is necessary for OAuth authentication.

There are two endpoints that all Service Manager SmartApps must define.

.. code-block:: groovy

    mappings {
        path("/oauth/initialize") {action: [GET: "oauthInitUrl"]}
        path("/oauth/callback") {action: [GET: "callback"]}
    }

The ``/oauth/initialize`` endpoint will be called during initialization.
This endpoint will then forward the user to the third-party service so they can log in.

The third-party service will be redirected to the ``/oauth/callback`` endpoint after the authentication has been successful.
Usually this is where the call is made to the third-party service to exchange an authorization code for an access token.

The overall idea is this:

- You will create a page on Service Manager SmartApp that will call out to the third-party API to initiate the authentication.
- The end result is an access token that SmartThings platform will then use to communicate with the third-party API.

In your Service Manager SmartApp preferences you create a page for authorization.

.. code-block:: groovy

    preferences {
        page(name: "Credentials", title: "Sample Authentication", content: "authPage", nextPage: "sampleLoggedInPage", install: false)
        ...
    }

The ``authPage`` method will perform the following tasks:

* Create a SmartApp access token that will be sent to the third party so that the third party can call back into SmartThings SmartApp.
* Check to make sure an access token doesn't already exist for this particular third-party service.
* Initialize the OAuth flow with the third party service if there is no access token.

Let's take a look at how we would accomplish this with ``authPage()``.

.. code-block:: groovy

    def authPage() {
        // Check to see if SmartApp has its own access token and create one if not.
        if(!state.accessToken) {
            // the createAccessToken() method will store the access token in state.accessToken
            createAccessToken()
        }

        def redirectUrl = "https://graph.api.smartthings.com/oauth/initialize?appId=${app.id}&access_token=${state.accessToken}&apiServerUrl=${getApiServerUrl()}"
        // Check to see if SmartThings already has an access token from the third-party service.
        if(!state.authToken) {
            return dynamicPage(name: "auth", title: "Login", nextPage: "", uninstall: false) {
                section() {
                    paragraph "tap below to log in to the third-party service and authorize SmartThings access"
                    href url: redirectUrl, style: "embedded", required: true, title: "3rd Party product", description: "Click to enter credentials"
                }
            }
        } else {
            // SmartThings has the token, so we can just call the third-party service to list our devices and select one to install.
        }
    }

There are a few things worth noting here:

- First, we are using ``state`` to store our tokens. Your specific needs may be different depending on your implementation. To learn more about how ``state`` works and what your options are, visit the :ref:`storing-data` guide.
- If we do not have a token from the third-party service, we start the OAuth flow by calling the SmartThings ``initialize`` endpoint. This is a static endpoint that will store a few bits of information about your SmartApp, such as the ``id``, and forwards the request to the ``/oauth/initalize`` endpoint defined in the SmartApp.

Initialize endpoint
~~~~~~~~~~~~~~~~~~~

This endpoint is used to initialize the OAuth flow to a third-party service.
The ``/oauth/initialize`` endpoint will save all the query parameters passed to it, but requires the following three parameters:

- The SmartApp ID,
- The SmartApp's access token, and
- The installed URL of the SmartApp. The endpoint will then call the mapped ``/oauth/initialize`` endpoint defined in the SmartApp with all the query parameters passed to it.

.. code-block:: html

    https://graph.api.smartthings.com/oauth/initialize


=================== ===========
Required parameters Value
=================== ===========
appId               The SmartApp ID
access_token        The SmartApp's access token
apiServerUrl        The URL of the server that the SmartApp is installed on. This information can be retrieved with the ``getApiServerUrl()`` method call.
=================== ===========

**Example:**

.. code-block:: groovy

    def redirectUrl = "https://graph.api.smartthings.com/oauth/initialize?appId=${app.id}&access_token=${state.accessToken}&apiServerUrl=${getApiServerUrl()}"

The ``initialize`` endpoint will forward the mapping defined in SmartApp to the ``/oauth/initialize``.
This method will be responsible for redirecting the user to the third-party login page.
Below is an example of how it works:

.. code-block:: groovy

    def oauthInitUrl() {

        // Generate a random ID to use as a our state value. This value will be used to verify the response we get back from the third-party service.
        state.oauthInitState = UUID.randomUUID().toString()

        def oauthParams = [
            response_type: "code",
            scope: "smartRead,smartWrite",
            client_id: appSettings.clientId,
            client_secret: appSettings.clientSecret,
            state: state.oauthInitState,
            redirect_uri: "https://graph.api.smartthings.com/oauth/callback"
        ]

        redirect(location: "${apiEndpoint}/authorize?${toQueryString(oauthParams)}")
    }

    // The toQueryString implementation simply gathers everything in the passed in map and converts them to a string joined with the "&" character.
    String toQueryString(Map m) {
	    return m.collect { k, v -> "${k}=${URLEncoder.encode(v.toString())}" }.sort().join("&")
    }

The ``oauthInitUrl()`` method sets up a request used to present the user with the third-party login page.
Often the third-party service will require information passed along with this request as query parameters.
The actual parameters sent with the request will vary depending on what the third-party service expects, so consult their API documentation to find specifics.

We are expecting to get an authorization code as a result of this request.
We will later exchange this authorization code for an access token.
We will create the access token request in our callback handler as seen below.
But for now, let's look at some basic parameters usually associated with authorization code requests.

================= ===========
Common parameters Value
================= ===========
response_type     The type of authorization defined by third-party service. Usually ``code`` or ``token``.
scope             Defines the scope of the request, i.e., what actions will be performed.
client_id         The client ID issued by the third-party service when signing up for access to their API. A best practice is to configure this parameter as an app setting in your SmartApp.
client_secret     The client secret issued by the third-party service when signing up for access to their API. A best practice is to configure this parameter as an app setting in your SmartApp.
state             Usually the ``state`` is not required, but is used to track state across requests. We will use this to validate the response we get back from the third party.
redirect_uri      The URI to be redirected to after the user has successfully authenticated with the third-party service. Usually this information is requested when signing up with the third-party service. This parameter must match what was entered at that time. For SmartApp development, this should always be the static value: ``https://graph.api.smartthings.com/oauth/callback``.
================= ===========

Callback endpoint
~~~~~~~~~~~~~~~~~

The third-party service will redirect the user to the callback endpoint after the user has been successfully authenticated.
For SmartApp development, this should always be the static value: ``https://graph.api.smartthings.com/oauth/callback``.
The callback endpoint is typically where the authorization code--that was acquired from the initialization--will be used to request the access token.
Let's look at an example.

.. code-block:: groovy

    def callback() {
        log.debug "callback()>> params: $params, params.code ${params.code}"

        def code = params.code
        def oauthState = params.state

        // Validate the response from the third party by making sure oauthState == state.oauthInitState as expected
        if (oauthState == state.oauthInitState){
            def tokenParams = [
                grant_type: "authorization_code",
                code      : code,
                client_id : appSettings.clientId,
                client_secret: appSettings.clientSecret,
                redirect_uri: "https://graph.api.smartthings.com/oauth/callback"
            ]

            // This URL will be defined by the third party in their API documentation
            def tokenUrl = "https://www.someservice.com/home/token?${toQueryString(tokenParams)}"

            httpPost(uri: tokenUrl) { resp ->
                state.refreshToken = resp.data.refresh_token
                state.authToken = resp.data.access_token
            }

            if (state.authToken) {
                // call some method that will render the successfully connected message
                success()
            } else {
                // gracefully handle failures
                fail()
            }

        } else {
            log.error "callback() failed. Validation of state did not match. oauthState != state.oauthInitState"
        }
    }

    // Example success method
    def success() {
	    def message = """
		    <p>Your account is now connected to SmartThings!</p>
		    <p>Click 'Done' to finish setup.</p>
	    """
	    displayMessageAsHtml(message)
    }

    // Example fail method
    def fail() {
        def message = """
            <p>There was an error connecting your account with SmartThings</p>
            <p>Please try again.</p>
        """
        displayMessageAsHtml(message)
    }

    def displayMessageAsHtml(message) {
        def html = """
            <!DOCTYPE html>
            <html>
                <head>
                </head>
                <body>
                    <div>
                        ${message}
                    </div>
                </body>
            </html>
        """
        render contentType: 'text/html', data: html
    }

In this callback we first check to make sure that the state returned from the authorization code request matches what we sent as the state.
This is how we know that the response is intended for us.
If it matches, we then set up the parameters for the access token request.
Common parameters are as follows:

================= ===========
Common parameters value
================= ===========
grant_type        This is the type of grant we are requesting. The third-party service will define the expected value.
code              The authorization code we obtained in the previous request.
client_id         The same client_id that we used in the previous request, which was issued by the third-party service.
client_secret     The same client_secret that we used in the previous request, which was issued by the third-party service.
redirect_uri      The same redirect_uri that we used in the previous request. This will usually be verified by the third-party service.
================= ===========

We issue an HTTP POST request to get the token.
If we receive a success response, we will save the access token that was issued by the third-party service, along with the refresh token, in ``state``.

Once we have acquired the access token, our authentication process is complete.
Usually the next step is to display some message to the end user about the success of the operation.

.. important::

    ``revokeAccessToken()`` should be called when the SmartApp's access token is no longer required.
    This is true when a user uninstalls the SmartApp.
    It is also a good practice to revoke the access token after successful authentication with the 3rd party, unless the token will be used to access other endpoints in your SmartApp.

Refreshing the OAuth token
~~~~~~~~~~~~~~~~~~~~~~~~~~

OAuth tokens are available for a finite amount of time, so you will
often need to account for this, and if needed, refresh your
``access_token``.
Above we illustrated how we initiate the request for the access and refresh tokens, and how we saved them in our SmartApp.
If we make a request to the third-party service API and get an "expired token" response, it is up to us to issue a new request to refresh the access token.
This is where the refresh token comes into play.

If you run an API request and your ``access_token`` is determined invalid, for example:

.. code-block:: groovy

    if (resp.status == 401 && resp.data.status.code == 14) {
        log.debug "Storing the failed action to try later"
        def action = "actionCurrentlyExecuting"
        log.debug "Refreshing your auth_token!"
        refreshAuthToken()
        // replay initial request from the action variable
        retryInitialRequest(action)
    }

you can use your ``refresh_token`` to get a new ``access_token``.
To do this, you just need to post to a specified endpoint and handle the response properly.

.. code-block:: groovy

    private refreshAuthToken() {
        def refreshParams = [
            method: 'POST',
            uri: "https://api.thirdpartysite.com",
            path: "/token",
            query: [grant_type:'refresh_token', code:"${state.sampleRefreshToken}", client_id:XXXXXXX],
        ]
        try{
            def jsonMap
            httpPost(refreshParams) { resp ->
                if(resp.status == 200)
                {
                    jsonMap = resp.data
                    if (resp.data) {
                        state.sampleRefreshToken = resp?.data?.refresh_token
                        state.sampleAccessToken = resp?.data?.access_token
                }
            }
        }
    }

There are some outbound connections in which we are using OAuth to
connect to a third party device cloud (Ecobee, Quirky, Jawbone, etc).
In these cases it is the third-party device cloud that issues an OAuth token to SmartThings so that SmartThings can call their APIs.

However, these same third-party device clouds also support webhooks and subscriptions that allow SmartThings to receive notifications when something changes in their cloud.

In this case, *and ONLY in this case*, the Service Manager SmartApp issues its own OAuth token and embeds it in the callback URL, as a way to authenticate the post backs from the external cloud.

Discovery
---------

Identifying devices in the third-party device cloud
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The techniques you will use to identify devices in the third-party
cloud will vary, because you are interacting with unique third-party
APIs which all have unique parameters.
Typically you will authenticate with the third-party API using OAuth; then call an API-specific method.
For example, it could be as simple as this:

.. code-block:: groovy

    def deviceListParams = [
        uri: "https://api.thirdpartysite.com",
        path: "/get-devices",
        requestContentType: "application/json",
        query: [token:"XXXX",type:"json" ]

    httpGet(deviceListParams) { resp ->
            //Handle the response here
    }

Creating child devices
~~~~~~~~~~~~~~~~~~~~~~

Within a Service Manager SmartApp, you create child devices for all your respective cloud devices.

.. code-block:: groovy

    settings.devices.each {deviceId->
        def device = state.devices.find{it.id==deviceId}
          if (device) {
            def childDevice = addChildDevice("smartthings", "Device Name", deviceId, null, [name: "Device.${deviceId}", label: device.name, completedSetup: true])
      }
    }

Getting initial device state
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Upon initial discovery of a device, you need to get the state of your device from the third-party API.
This would be the current status of various attributes of your device.
You need to have a method defined in your Service Manager that is responsible for connecting to the API and to check for the updates.
You set this method to be called from a poll method in your Device Handler, and in this case, it is called immediately on initialization.
Here is a very simple example which doesn't take into account error checking for the ``http`` request.

.. code-block:: groovy

    def pollParams = [
        uri: "https://api.thirdpartysite.com",
        path: "/device",
        requestContentType: "application/json",
        query: [format:"json",body: jsonRequestBody]

    httpGet(pollParams) { resp ->
        state.devices = resp.data.devices { collector, stat ->
        def dni = [ app.id, stat.identifier ].join('.')
        def data = [
            attribute1: stat.attributeValue,
            attribute2: stat.attribute2Value
        ]
        collector[dni] = [data:data]
        return collector
        }
    }

Handling adds, changes, deletes
-------------------------------

singleInstance Service Manager
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Adding the tag ``singleInstance: true`` to your Service Manager will ensure only one instance of the Service Manager will be installed.
All child devices will be installed under the single parent Service Manager.
This enforces a one-to-many relationship between the parent Service Manager SmartApp and any child devices.

.. code-block:: groovy
    :emphasize-lines: 9

    definition(
        name: "Ecobee (Connect)",
        namespace: "smartthings",
        author: "SmartThings",
        description: "Connect your Ecobee thermostat to SmartThings.",
        category: "SmartThings Labs",
        iconUrl: "https://s3.amazonaws.com/smartapp-icons/Partner/ecobee.png",
        iconX2Url: "https://s3.amazonaws.com/smartapp-icons/Partner/ecobee@2x.png",
        singleInstance: true)


Implicit creation of new child Devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you update your settings in a Service Manager to add additional
devices, the Service Manager needs to respond by adding a new device
in SmartThings.

.. code-block:: groovy

    updated(){
        initialize()
    }

    initialize(){
        settings.devices.each {deviceId ->
            try {
                def existingDevice = getChildDevice(deviceId)
                if(!existingDevice) {
                    def childDevice = addChildDevice("smartthings", "Device Name", deviceId, null, [name: "Device.${deviceId}", label: device.name, completedSetup: true])
                }
            } catch (e) {
                log.error "Error creating device: ${e}"
            }
        }
    }

Implicit removal of child Devices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Similarly when you remove devices in your Service Manager, they
need to be removed from SmartThings platform.

.. code-block:: groovy

    def delete = getChildDevices().findAll { !settings.devices.contains(it.deviceNetworkId) }

    delete.each {
        deleteChildDevice(it.deviceNetworkId)
    }

Also, when a Service Manager SmartApp is uninstalled, you need to remove its child devices.

.. code-block:: groovy

    def uninstalled() {
        removeChildDevices(getChildDevices())
    }

    private removeChildDevices(delete) {
        delete.each {
            deleteChildDevice(it.deviceNetworkId)
        }
    }

.. note::

    The ``addChildDevice``, ``getChildDevices``, and ``deleteChildDevice`` methods are a part of the :ref:`smartapp_ref` API.

Changes in Device name
~~~~~~~~~~~~~~~~~~~~~~

The device name is stored within the device and you need to monitor if it changes in the third-party cloud.

Explicit delete actions
~~~~~~~~~~~~~~~~~~~~~~~

When a user manually deletes a device in the Things screen on the client device, you need to delete the child device from within the Service Manager.
