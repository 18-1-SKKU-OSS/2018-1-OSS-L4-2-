.. _smartapp_as_web_service_part_2:

웹 서비스 SmartApp 튜토리얼--인증 절차
==================================================

이 튜토리얼의 `1부 <./tutorial-part1.html>`__ 에서 간단한 웹 서비스 SmartApp을 생성하고, IDE 시뮬레이터에 설치하고, 웹 요청을 하는 방법을 배웠습니다.

2부에서는, SmartThings와 1부에서 작성한 웹 서비스 SmartApp이 통합되는 간단한 웹 어플리케이션을 작성할 것입니다.

----

개요
--------

이 튜토리얼의 2부에서, 다음과 같은 것을 배우게 될 것입니다:

- API 토큰을 얻는 방법
- 웹 서비스 SmartApp의 엔드포인트를 찾는 방법
- 웹 서비스 SmartApp 호출 방법

이 튜토리얼의 소스코드는 `여기 <https://github.com/SmartThingsCommunity/Code/tree/master/smartapps/tutorials/web-services-smartapps>`__ 에서 볼 수 있습니다.

.. include:: ../common/oauth-install-restriction.txt

우리는 1부에서 작성한 웹 서비스 SmartApp을 호출할 간단한 Sinatra 어플리케이션을 작성할 것입니다.

Sinatra에 익숙하지 않다면 Sinatra를 사용해보세요.
하지만 어플리케이션은 API 토큰과 엔드포인트를 가져오기 위한 웹 요청을 하기 때문에 반드시 필요한 것은 아닙니다.

.. note::

  Node의 속도가 더 빠르다면, `여기 <https://github.com/schettj/SmartThings>`__ 에서 커뮤니티 회원인 John S (@schettj) 가 작성한 멋진 SmartThings OAuth Node 앱을 확인하세요. 이것은 Node를 사용하여 웹 서비스 SmartApp에 대한 OAuth 흐름을 사용하여 접근 토큰을 얻는 방법을 보여줍니다.

----

전제 조건
-------------

이 튜토리얼의 1부를 완료하는 것 외에도 Ruby와 Sinatra를 설치해야 합니다.

Ruby를 설치하려면 `Ruby <http://ruby-lang.org>`__ 웹 사이트를, Sinatra 설치에 관한 정보를 얻으려면 `Sinatra Getting 시작 페이지 <http://www.sinatrarb.com/intro.html>`__ 를 방문하세요.

----

Sinatra 앱 부트스트랩하기
-------------------------

Sinatra 앱을 위해 새 디렉토리를 만들고, 그 디렉토리로 디렉토리를 바꾸세요:

.. code-block:: bash

    mkdir web-app-tutorial
    cd web-app-tutorial

자신이 선호하는 텍스트 편집기*에서 ``server.rb`` 라는 새 파일을 만들고 다음을 붙여넣고 저장하세요.

\*(*선호하는 텍스트 편집기가 vim이나 emacs라면, 당신에게 경의를 표합니다. 저희는 감동받았습니다 - 저희가 살짝 주눅까지 들 정도 입니다. 만약 선호하는 텍스트 편집기가 메모장이라면, 음... 그 정도까지 감동받지는 않을 것 같습니다. :@)*)

.. code-block:: ruby

    require 'bundler/setup'
    require 'sinatra'
    require 'oauth2'
    require 'json'
    require "net/http"
    require "uri"

    # 접근 토큰을 얻는데 사용되는 클라이언트 ID와 비밀번호
    CLIENT_ID = ENV['ST_CLIENT_ID']
    CLIENT_SECRET = ENV['ST_CLIENT_SECRET']

    # 이 세션에서 접근 토큰을 저장할 것입니다
    use Rack::Session::Pool, :cookie_only => false

    # 이것은 우리 접근과 함께 호출될 URI입니다
    # 우리 SmartThings 계정으로 인증한 후 코드
    redirect_uri = 'http://localhost:4567/oauth/callback'

    # 이것은 토큰을 받으면 엔드포인트를 얻는데 사용할 URI입니다
    endpoints_uri = 'https://graph.api.smartthings.com/api/smartapps/endpoints'

    options = {
      site: 'https://graph.api.smartthings.com',
      authorize_url: '/oauth/authorize',
      token_url: '/oauth/token'
    }

    # OAuth 흐름 처리를 위한 OAuth2 모듈 사용
    client = OAuth2::Client.new(CLIENT_ID, CLIENT_SECRET, options)

    # 우리가 접근 토큰을 가지고 있는지 여부를 알려주는 헬퍼 메소드
    def authenticated?
      session[:access_token]
    end

    # 어플리케이션 루트에 대한 요청 처리
    get '/' do
      %(<a href="/authorize">Connect with SmartThings</a>)
    end

    # /authorize URL에 대한 요청 처리 
    get '/authorize' do
        'Not Implemented!'
    end

    # /oauth/callback URL에 대한 요청 처리
    # 인증이 완료되면 SmartThings에 인증 코드로
    # 이 URL을 호출하도록 알려드릴 것입니다.
    get '/oauth/callback' do
        'Not Implemented!'
    end

    # /getSwitch URL에 대한 요청 처리
    # 여기서 구성된 스위치에 대한 정보를 얻기 위해
    # 요청할 것입니다.
    get '/getswitch' do
        'Not Implemented!'
    end

Gemfile을 만드세요 - 편집기에서 새 파일을 열고, 아래 내용을 붙여넣고 ``Gemfile`` 로 저장하세요.

.. code-block:: ruby

    source 'https://rubygems.org'

    gem 'sinatra'
    gem 'oauth2'
    gem 'json'

우리는 앱을 설치하기 위해 번들러를 사용할 것입니다. 만약 없다면, `여기 <http://bundler.io/>`__ 에서 시작하는 법을 배울 수 있습니다.

명령 줄 위에서 번들을 실행하세요:

.. code-block:: bash

    bundle install

또한 ST_CLIENT_ID와 ST_CLIENT_SECRET에 대한 환경 변수를 설정하려고 합니다.

이제 로컬 머신에서 앱을 실행하세요::

    ruby server.rb

`http://localhost:4567 <http://localhost:4567>`__ 를 방문하세요.
"SmartThings와 연결" 링크가 있는 페이지가 나타날 것입니다.

우리는 OAuth2 흐름을 처리하기 위해 `OAuth2 모듈 <https://github.com/intridea/oauth2>`__ 을 사용하고 있습니다.
우리는 ``client_id`` 와 ``client_secret`` 를 사용해 새로운 ``client`` 객체를 만듭니다.
또한 이를 SmartThings OAuth 엔드포인트에 대한 정보를 정의하는 ``options`` 데이터 구조로 구성합니다.

우리는 서버의 ``/authorize`` URL을 가리키는 링크를 간단히 표시하기 위해 루트 URL을 처리했습니다. 다음에 그것을 채울 것입니다.

----

인증 코드 받기
-------------------------

사용자가 "SmartThings와 연결" 링크를 클릭하면 OAuth 인증 코드를 받아야 합니다.

이렇게 하려면, 사용자가 SmartThings로 인증하고, 이 어플리케이션에서 사용할 수 있는 장치를 승인해야 합니다.
일단 완료되면, 사용자는 OAuth 인증 코드와 함께 지정된 ``redirect_uri`` 로 다시 돌아가게 됩니다.
이 튜토리얼의 1부에서 SmartApp을 만들 때, 우리는 재전송 URI를 ``http://localhost:4567/oauth/callback`` 로 설정했습니다.
SmartApp의 재전송 URI와 Sinatra 앱의 ``redirect_uri`` 필드가 일치하는 것이 중요한데, 이는 이 두 URI가 일치하는지 확인하는 인증 코드 요청과 함께 유효성 검사가 실행되기 때문입니다.
이것은 (``client_id`` 와 ``client_secret`` 와 함께) 접근 토큰을 얻기 위해 사용될 것입니다.

.. important::

    SmartApp을 자체 게시하면, 게시된 위치에서 SmartApp이 게시되고 사용할 수 있습니다.
    SmartThings이 글로벌 공간으로 나아가고 있기 때문에, SmartApp을 게시한 위치는 특정 서버와 상응합니다.
    즉, 자체 게시된 SmartApp은 해당 서버에서만 사용할 수 있습니다.


``/authorize`` 경로를 다음으로 바꾸세요:

.. code-block:: ruby

    get '/authorize' do
      # 승인 URL을 얻기 위해 OAuth2를 사용합니다.
      # SmartThings로 인증한 후, 우리는
      # 토큰을 얻는데 사용된 접근 코드를 포함하여 redirect_uri로 재전송됩니다.
      url = client.auth_code.authorize_url(redirect_uri: redirect_uri, scope: 'app')
      redirect url
    end

서버가 실행중이면 서버를 종료하고 (CTRL+C), ``ruby server.rb`` 를 사용해 서버를 다시 시작하세요.

다시 `http://localhost:4567 <http://localhost:4567>`__ 을 방문하고, "SmartThings와 연결" 링크를 클릭하세요.

이렇게 하면 SmartThings 계정으로 인증 (아직 로그인하지 않은 경우) 하라는 메시지가 나타나야 하며, 이 어플리케이션을 인증하는 페이지로 이동하게 됩니다.
다음과 같이 보일 것입니다:

.. figure:: ../img/smartapps/web-services/preferences.png

*인증* 버튼을 클릭하면 서버로 재전송됩니다.

아직 이 URL을 처리하지 않았으므로 "구현되지 않음!"을 참조하세요.

----

접근 토큰 얻기
-------------------

SmartThings가 인증 후 어플리케이션으로 재전송하면, ``code`` 매개변수를 URL에 전달합니다.
이것은 웹 서비스 SmartApp에 요청하는데 필요한 API 토큰을 얻는데 사용할 코드입니다.

우리는 세션에 접근 토큰을 저장할 것입니다.
``server.rb`` 의 맨 위에 세션을 사용하도록 앱을 구성하고, 사용자가 인증했는지를 알 수 있도록 헬퍼 메소드를 추가합니다:

.. code-block:: ruby

    # 우리는 세션에 접근 토큰을 저장할 것입니다
    use Rack::Session::Pool, :cookie_only => false

    def authenticated?
        session[:access_token]
    end

``/oauth/callback`` 루트를 다음과 같이 변경하세요:

.. code-block:: ruby

    get '/oauth/callback' do
      # 콜백은 "code" URL 매개변수와 함께 호출됩니다
      # 우리의 접근 토큰을 얻기 위해 사용할 수 있는 코드입니다
      code = params[:code]

      # 토큰을 얻기 위한 코드
      response = client.auth_code.get_token(code, redirect_uri: redirect_uri, scope: 'app')

      # 이제 접근 토큰이 생겼으므로 세션에 저장합니다
      session[:access_token] = response.token

      # 디버그 - 실행 중인 콘솔을 (지금부터 몇 초후인) 만료 날짜로 검사하고,
      # 만료 날짜는 (에포크 시간)
      puts 'TOKEN EXPIRES IN ' + response.expires_in.to_s
      puts 'TOKEN EXPIRES AT ' + response.expires_at.to_s
      redirect '/getswitch'
    end

먼저 매개 변수에서 접근 코드를 검색합니다.
OAuth2 모듈을 사용해 토큰을 얻는데 이를 사용하고, 세션이 토큰을 저장합니다.

그러고 나서 서버의 ``/getswitch`` URL로 재전송합니다.
여기서 우리는 호출할 엔드포인트를 검색하고, 구성된 스위치의 상태를 가져옵니다.

서버를 다시 시작하고 사용해보세요.
인증되면, ``/getswitch`` URL로 재전송되어야 합니다. 다음에 그를 구현하는 것을 시작할 것입니다.

----

엔드포인트 발견
---------------------

이제 OAuth 토큰을 가지고 있으므로, 이를 사용해 우리 웹 서비스 SmartApp의 엔드포인트를 발견할 수 있습니다.

``/getswitch`` 경로를 다음과 같이 변경하세요:

.. code-block:: ruby

    get '/getswitch' do
      # 만약 접근 토큰 없이 이 URL에 접속했다면
      # 루트로 재전송하여 승인을 거칩니다
      if !authenticated?
        redirect '/'
      end

      token = session[:access_token]

      # 토큰을 사용해 SmartThings 엔드포인트 URI에 요청하여
      # 엔드포인트를 가져옵니다
      url = URI.parse(endpoints_uri)
      req = Net::HTTP::Get.new(url.request_uri)

      # "Authorization: Bearer <API Token>" 이라는 HTTP 헤더를 설정합니다
      req['Authorization'] = 'Bearer ' + token

      http = Net::HTTP.new(url.host, url.port)
      http.use_ssl = (url.scheme == "https")

      response = http.request(req)
      json = JSON.parse(response.body)

      # 디버그 문
      puts json

      # JSON에서 엔드포인트를 가져옵니다:
      uri = json[0]['uri']

      '<h3>JSON Response</h3><br/>' + JSON.pretty_generate(json) + '<h3>Endpoint</h3><br/>' + uri
    end

위의 코드는 단순히 ``https://graph.api.smartthings.com/api/smartapps/endpoints`` 의 SmartThings API 엔드포인트로 GET 요청을 하고, ``"Authorization"`` HTTP 헤더를 API 토큰으로 설정합니다.

응답은 (다른 것들 중에) SmartApp의 엔드포인트를 포함하는 JSON 입니다. 반환된 JSON는 엔드포인트 URL들을 작성하는데 사용할 `uri` 라고 불리는 키를 포함합니다.
JSON에는 다른 URL키들이 있지만, `uri` 키는 SmartApp이 있는 서버에만 해당됩니다.
엔드포인트에는 항상 `uri` 키나 `base_uri` 를 사용하세요.
이 단계에서 우리는 그저 JSON 응답과 페이지의 엔드포인트를 표시합니다.

이제 당신은 절차를 알고 있습니다. 서버를 다시 시작하고, 페이지를 새로 고친 다음, 링크를 클릭하세요 (재인증 해야 함).
그런 다음 페이지에서 JSON 응답과 엔드포인트가 표시되어야 합니다.

----

API 호출하기
--------------

이제 토큰과 엔드포인트를 가지고 있으므로, SmartApp에 API 호출을 할 수 있습니다.

URL 경로에서 짐작할 수 있듯이, 우리는 스위치의 이름과 현재 상태 (켜기 또는 끄기)를 표시할 것입니다.

응답 HTML을 출력하는 ``getswitch`` 경로 핸들러의 마지막 줄을 제거하고, 다음을 추가하세요:

.. code-block:: ruby

  # 이제 웹 서비스 SmartApp에 대한 URL을 작성할 수 있습니다
  # 우리는 스위치에 대한 정보를 얻기 위해 GET 요청을 할 것입니다
  switchUrl = uri + '/switches'

  # 디버그
  puts "SWITCH ENDPOINT: " + switchUrl

  getSwitchURL = URI.parse(switchUrl)
  getSwitchReq = Net::HTTP::Get.new(getSwitchURL.request_uri)
  getSwitchReq['Authorization'] = 'Bearer ' + token

  getSwitchHttp = Net::HTTP.new(getSwitchURL.host, getSwitchURL.port)
  getSwitchHttp.use_ssl = true

  switchStatus = getSwitchHttp.request(getSwitchReq)

  '<h3>Response Code</h3>' + switchStatus.code + '<br/><h3>Response Headers</h3>' + switchStatus.to_hash.inspect + '<br/><h3>Response Body</h3>' + switchStatus.body


위의 코드는 URL을 작성할 SmartApp에 대한 (위의 JSON 응답에서의 `uri` 키로 부터 얻은) 엔드포인트를 사용하고, ``/switches`` 엔드포인트로 GET 요청을 보냅니다.
이는 단순히 상태, 헤더, 그리고 웹 서비스 SmartApp로부터 반환된 응답 본문을 표시합니다.

서버를 재시작하고 다시 시도해보세요.
구성된 스위치의 상태가 표시되어야 합니다!

----

요약
-------

이 튜토리얼의 2부에서는, 외부 어플리케이션이 접근 토큰을 가져와 어떻게 SmartThings와 함께 작동하는지, 어떻게 엔드포인트를 찾는지, 어떻게 웹 서비스 SmartApp으로 API 호출을 하는지에 대해 배웠습니다.

구성된 스위치를 켜거나 끄기 위한 다른 API 호출 만들기를 포함해, 이 샘플을 더 자세히 살펴보는 것이 좋겠습니다.
