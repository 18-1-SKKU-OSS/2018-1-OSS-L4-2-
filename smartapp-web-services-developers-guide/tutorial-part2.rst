.. _smartapp_as_web_service_part_2:

Web Services SmartApp Tutorial--Authorization Flow
==================================================

In `Part 1 <./tutorial-part1.html>`__ of this tutorial, you learned how to create a simple Web Services SmartApp, and install it in the IDE simulator, and make web requests to it.

In Part 2, we'll build a simple web application that will integrate with SmartThings and the WebServices SmartApp we created in Part 1.

----

Overview
--------

In Part 2 of this tutorial, you will learn:

- How to get the API token.
- How to discover the endpoints of a Web Services SmartApp.
- How to make calls to the Web Services SmartApp.

The source code for this tutorial is available `here <https://github.com/SmartThingsCommunity/Code/tree/master/smartapps/tutorials/web-services-smartapps>`__.

.. include:: ../common/oauth-install-restriction.txt

We will build a simple Sinatra application that will make calls to the Web Services SmartApp we built in Part 1.

If you're not familiar with Sinatra, you are encouraged to try it out.
It's not strictly necessary, however, as our application will simply make web requests to get the API token and the endpoint.

.. note::

  If Node is more your speed, check out the awesome SmartThings OAuth Node app written by community member John S (@schettj) `here <https://github.com/schettj/SmartThings>`__. It shows how you can get an access token using the OAuth flow for a WebServices SmartApp using Node.

----

Prerequisites
-------------

Aside from completing Part 1 of this tutorial, you should have Ruby and Sinatra installed.

Visit the `Ruby <http://ruby-lang.org>`__ website to install Ruby, and the `Sinatra Getting Started Page <http://www.sinatrarb.com/intro.html>`__ for information about installing Sinatra.

----

Bootstrap the Sinatra app
-------------------------

Create a new directory for the Sinatra app, and change directories to it:

.. code-block:: bash

    mkdir web-app-tutorial
    cd web-app-tutorial

In your favorite text editor*, create a new file called ``server.rb`` and paste the following into it, and save it.

\*(*If your favorite text editor is vim or emacs, then our hat's off to you. We're impressed - maybe even a bit intimidated. If your favorite editor is notepad, well... we're not as impressed, or intimidated. :@)*)

.. code-block:: ruby

    require 'bundler/setup'
    require 'sinatra'
    require 'oauth2'
    require 'json'
    require "net/http"
    require "uri"

    # Our client ID and secret, used to get the access token
    CLIENT_ID = ENV['ST_CLIENT_ID']
    CLIENT_SECRET = ENV['ST_CLIENT_SECRET']

    # We'll store the access token in the session
    use Rack::Session::Pool, :cookie_only => false

    # This is the URI that will be called with our access
    # code after we authenticate with our SmartThings account
    redirect_uri = 'http://localhost:4567/oauth/callback'

    # This is the URI we will use to get the endpoints once we've received our token
    endpoints_uri = 'https://graph.api.smartthings.com/api/smartapps/endpoints'

    options = {
      site: 'https://graph.api.smartthings.com',
      authorize_url: '/oauth/authorize',
      token_url: '/oauth/token'
    }

    # use the OAuth2 module to handle OAuth flow
    client = OAuth2::Client.new(CLIENT_ID, CLIENT_SECRET, options)

    # helper method to know if we have an access token
    def authenticated?
      session[:access_token]
    end

    # handle requests to the application root
    get '/' do
      %(<a href="/authorize">Connect with SmartThings</a>)
    end

    # handle requests to /authorize URL
    get '/authorize' do
        'Not Implemented!'
    end

    # handle requests to /oauth/callback URL. We
    # will tell SmartThings to call this URL with our
    # authorization code once we've authenticated.
    get '/oauth/callback' do
        'Not Implemented!'
    end

    # handle requests to the /getSwitch URL. This is where
    # we will make requests to get information about the configured
    # switch.
    get '/getswitch' do
        'Not Implemented!'
    end

Create your Gemfile - open a new file in your editor, paste the contents below in, and save it as ``Gemfile``.

.. code-block:: ruby

    source 'https://rubygems.org'

    gem 'sinatra'
    gem 'oauth2'
    gem 'json'

We'll use bundler to install our app. If you don't have it, you can learn how to get started `here <http://bundler.io/>`__.

Back at the command line, run bundle:

.. code-block:: bash

    bundle install

You'll also want to set environment variables for your ST_CLIENT_ID and ST_CLIENT_SECRET.

Now, run the app on your local machine::

    ruby server.rb

Visit `http://localhost:4567 <http://localhost:4567>`__.
You should see a page with a link to "Connect with SmartThings".

We're using the `OAuth2 module <https://github.com/intridea/oauth2>`__ to handle the OAuth2 flow.
We create a new ``client`` object, using the ``client_id`` and ``client_secret``.
We also configure it with the ``options`` data structure that defines the information about the SmartThings OAuth endpoint.

We've handled the root URL to simply display a link that points to the ``/authorize`` URL of our server. We'll fill that in next.

----

Get an authorization code
-------------------------

When the user clicks on the "Connect with SmartThings" link, we need to get our OAuth authorization code.

To do this, the user will need to authenticate with SmartThings, and authorize the devices this application can work with.
Once that has been done, the user will be directed back to a specified ``redirect_uri``, with the OAuth authorization code.
When we created the SmartApp in the first part of this tutorial, we set the redirect URI to ``http://localhost:4567/oauth/callback``.
It is important that the redirect URI in the SmartApp and the ``redirect_uri`` field in this Sinatra app match, as validation will occur with the authorization code request that will make sure these two URIs match.
This will be used (along with the ``client_id`` and ``client_secret``), to get the access token.

.. important::

    When you self-publish a SmartApp, it is published and available in the Location that you published it.
    Since SmartThings is moving into the global space, the Location that you published your SmartApp corresponds to a specific server.
    This means your self-published SmartApp is only available on that server.


Replace the ``/authorize`` route with the following:

.. code-block:: ruby

    get '/authorize' do
      # Use the OAuth2 module to get the authorize URL.
      # After we authenticate with SmartThings, we will be redirected to the
      # redirect_uri, including our access code used to get the token
      url = client.auth_code.authorize_url(redirect_uri: redirect_uri, scope: 'app')
      redirect url
    end

Kill the server if it's running (CTRL+C), and start it up again using ``ruby server.rb``.

Visit `http://localhost:4567 <http://localhost:4567>`__ again, and click the "Connect with SmartThings" link.

This should prompt you to authenticate with your SmartThings account (if you are not already logged in), and bring you to a page where you must authorize this application.
It should look something like this:

.. figure:: ../img/smartapps/web-services/preferences.png

Click the *Authorize* button, and you will be redirected back your server.

You'll notice that we haven't implemented handling this URL yet, so we see "Not Implemented!".

----

Get an access token
-------------------

When SmartThings redirects back to our application after authorizing, it passes a ``code`` parameter on the URL.
This is the code that we will use to get the API token we need to make requests to our Web Servcies SmartApp.

We'll store the access token in the session.
Towards the top of ``server.rb``, we configure our app to use the session, and add a helper method to know if the user has authenticated:

.. code-block:: ruby

    # We'll store the access token in the session
    use Rack::Session::Pool, :cookie_only => false

    def authenticated?
        session[:access_token]
    end

Replace the ``/oauth/callback`` route with the following:

.. code-block:: ruby

    get '/oauth/callback' do
      # The callback is called with a "code" URL parameter
      # This is the code we can use to get our access token
      code = params[:code]

      # Use the code to get the token.
      response = client.auth_code.get_token(code, redirect_uri: redirect_uri, scope: 'app')

      # now that we have the access token, we will store it in the session
      session[:access_token] = response.token

      # debug - inspect the running console for the
      # expires in (seconds from now), and the expires at (in epoch time)
      puts 'TOKEN EXPIRES IN ' + response.expires_in.to_s
      puts 'TOKEN EXPIRES AT ' + response.expires_at.to_s
      redirect '/getswitch'
    end

We first retrieve the access code from the parameters.
We use this to get the token using the OAuth2 module, and store it in the session.

We then redirect to the ``/getswitch`` URL of our server.
This is where we will retrieve the endpoint to call, and get the status of the configured switch.

Restart your server, and try it out.
Once authorized, you should be redirected to the ``/getswitch`` URL. We'll start implementing that next.

----

Discover the endpoint
---------------------

Now that we have the OAuth token, we can use it to discover the endpoint of our WebServices SmartApp.

Replace the ``/getswitch`` route with the following:

.. code-block:: ruby

    get '/getswitch' do
      # If we get to this URL without having gotten the access token
      # redirect back to root to go through authorization
      if !authenticated?
        redirect '/'
      end

      token = session[:access_token]

      # make a request to the SmartThings endpoint URI, using the token,
      # to get our endpoints
      url = URI.parse(endpoints_uri)
      req = Net::HTTP::Get.new(url.request_uri)

      # we set a HTTP header of "Authorization: Bearer <API Token>"
      req['Authorization'] = 'Bearer ' + token

      http = Net::HTTP.new(url.host, url.port)
      http.use_ssl = (url.scheme == "https")

      response = http.request(req)
      json = JSON.parse(response.body)

      # debug statement
      puts json

      # get the endpoint from the JSON:
      uri = json[0]['uri']

      '<h3>JSON Response</h3><br/>' + JSON.pretty_generate(json) + '<h3>Endpoint</h3><br/>' + uri
    end

The above code simply makes a GET request to the SmartThings API endpoints service at ``https://graph.api.smartthings.com/api/smartapps/endpoints``, setting the ``"Authorization"`` HTTP header with the API token.

The response is JSON that contains (among other things), the endpoint of our SmartApp. The JSON that is returned contains a key called  `uri` that we will use to build our endpoint URLs.
There are other URL keys in the JSON, but the `uri` key is specific to the server that your SmartApp is on.
Always use the `uri` key or `base_uri` for your endpoints.
For this step, we just display the JSON response and endpoint in the page.

By now, you know the drill. Restart your server, refresh the page, and click the link (you'll have to reauthorize).
You should then see the JSON response and endpoint displayed on your page.

----

Make API calls
--------------

Now that we have our token and endpoint, we can make API calls to our SmartApp.

As you may have guessed by the URL path, we're just going to display the name of the switch, and it's current status (on or off).

Remove the line at the end of the ``getswitch`` route handler that outputs the response HTML, and add the following:

.. code-block:: ruby

  # now we can build a URL to our WebServices SmartApp
  # we will make a GET request to get information about the switch
  switchUrl = uri + '/switches'

  # debug
  puts "SWITCH ENDPOINT: " + switchUrl

  getSwitchURL = URI.parse(switchUrl)
  getSwitchReq = Net::HTTP::Get.new(getSwitchURL.request_uri)
  getSwitchReq['Authorization'] = 'Bearer ' + token

  getSwitchHttp = Net::HTTP.new(getSwitchURL.host, getSwitchURL.port)
  getSwitchHttp.use_ssl = true

  switchStatus = getSwitchHttp.request(getSwitchReq)

  '<h3>Response Code</h3>' + switchStatus.code + '<br/><h3>Response Headers</h3>' + switchStatus.to_hash.inspect + '<br/><h3>Response Body</h3>' + switchStatus.body


The above code uses the endpoint (obtained from the `uri` key in our JSON response above) for our SmartApp to build a URL, and then makes a GET request to the ``/switches`` endpoint.
It simply displays the the status, headers, and response body returned by our WebServices SmartApp.

Restart your server and try it out.
You should see status of your configured switches displayed!

----

Summary
-------

In the second part of this tutorial, we learned how an external application can work with SmartThings by getting an access token, discover endpoints, and make API calls to a WebServices SmartApp.

You are encouraged to explore further with this sample, including making different API calls to turn the configured switch on or off.
