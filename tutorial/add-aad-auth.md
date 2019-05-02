<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="bfc95-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="bfc95-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="bfc95-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="bfc95-103">この手順では、omniauth と[oauth2](https://github.com/omniauth/omniauth-oauth2)の gem をアプリケーションに統合し、カスタム omniauth 戦略を作成します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

<span data-ttu-id="bfc95-104">最初に、アプリ ID とシークレットを保持する個別のファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-104">First, create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="bfc95-105">`./config`フォルダーにという`oauth_environment_variables.rb`新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-105">Create a new file called `oauth_environment_variables.rb` in the `./config` folder, and add the following code.</span></span>

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

<span data-ttu-id="bfc95-106">を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換え、生成し`YOUR_APP_SECRET_HERE`たパスワードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="bfc95-107">Git などのソース管理を使用している場合は、アプリ ID とパスワードを誤っ`oauth_environment_variables.rb`てリークしないように、ソース管理からファイルを除外することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="bfc95-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

<span data-ttu-id="bfc95-108">このファイルが存在する場合、このファイルを読み込むコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-108">Now add code to load this file if it's present.</span></span> <span data-ttu-id="bfc95-109">`./config/environment.rb`ファイルを開き、行の`Rails.application.initialize!`前に次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-109">Open the `./config/environment.rb` file and add the following code before the `Rails.application.initialize!` line.</span></span>

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a><span data-ttu-id="bfc95-110">セットアップ OmniAuth</span><span class="sxs-lookup"><span data-stu-id="bfc95-110">Setup OmniAuth</span></span>

<span data-ttu-id="bfc95-111">既に gem が`omniauth-oauth2`インストールされていますが、Azure OAuth エンドポイントを使用できるようにするためには、 [OAuth2 戦略を作成](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-111">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="bfc95-112">これは、Azure プロバイダーに OAuth 要求を作成するためのパラメーターを定義する Ruby クラスです。</span><span class="sxs-lookup"><span data-stu-id="bfc95-112">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

<span data-ttu-id="bfc95-113">`./lib`フォルダーにという`microsoft_graph_auth.rb`新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-113">Create a new file called `microsoft_graph_auth.rb` in the `./lib` folder, and add the following code.</span></span>

```ruby
require 'omniauth-oauth2'

module OmniAuth
  module Strategies
    # Implements an OmniAuth strategy to get a Microsoft Graph
    # compatible token from Azure AD
    class MicrosoftGraphAuth < OmniAuth::Strategies::OAuth2
      option :name, :microsoft_graph_auth

      DEFAULT_SCOPE = 'openid email profile User.Read'.freeze

      # Configure the Azure v2 endpoints
      option  :client_options,
              site:          'https://login.microsoftonline.com',
              authorize_url: '/common/oauth2/v2.0/authorize',
              token_url:     '/common/oauth2/v2.0/token'

      # Send the scope parameter during authorize
      option :authorize_options, [:scope]

      # Unique ID for the user is the id field
      uid { raw_info['id'] }

      # Get additional information after token is retrieved
      extra do
        {
          'raw_info' => raw_info
        }
      end

      def raw_info
        # Get user profile information from the /me endpoint
        @raw_info ||= access_token.get('https://graph.microsoft.com/v1.0/me').parsed
      end

      def authorize_params
        super.tap do |params|
          params['scope'.to_sym] = request.params['scope'] if request.params['scope']
          params[:scope] ||= DEFAULT_SCOPE
        end
      end

      # Override callback URL
      # OmniAuth by default passes the entire URL of the callback, including
      # query parameters. Azure fails validation because that doesn't match the
      # registered callback.
      def callback_url
        options[:redirect_uri] || (full_host + script_name + callback_path)
      end
    end
  end
end
```

<span data-ttu-id="bfc95-114">このコードの内容を確認してください。</span><span class="sxs-lookup"><span data-stu-id="bfc95-114">Take a moment to review what this code does.</span></span>

- <span data-ttu-id="bfc95-115">は、 `client_options` Azure v2 エンドポイントを指定するように設定します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-115">It sets the `client_options` to specify the Azure v2 endpoints.</span></span>
- <span data-ttu-id="bfc95-116">承認フェーズで`scope`パラメーターを送信する必要があることを指定します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-116">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
- <span data-ttu-id="bfc95-117">ユーザーの`id`プロパティをユーザーの一意の ID としてマップします。</span><span class="sxs-lookup"><span data-stu-id="bfc95-117">It maps the `id` property of the user as the unique ID for the user.</span></span>
- <span data-ttu-id="bfc95-118">アクセストークンを使用して、Microsoft Graph からユーザーのプロファイルを取得し、 `raw_info`ハッシュを入力します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-118">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
- <span data-ttu-id="bfc95-119">これは、コールバック URL を上書きして、アプリ登録ポータルで登録されているコールバックと一致するようにします。</span><span class="sxs-lookup"><span data-stu-id="bfc95-119">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

<span data-ttu-id="bfc95-120">戦略を定義したので、これを使用するように OmniAuth を構成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-120">Now that we've defined the strategy, we need to configure OmniAuth to use it.</span></span> <span data-ttu-id="bfc95-121">`./config/initializers`フォルダーにという`omniauth_graph.rb`新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-121">Create a new file called `omniauth_graph.rb` in the `./config/initializers` folder, and add the following code.</span></span>

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

<span data-ttu-id="bfc95-122">このコードは、アプリが開始するときに実行されます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-122">This code will execute when the app starts.</span></span> <span data-ttu-id="bfc95-123">このメソッドは、で`microsoft_graph_auth` `oauth_environment_variables.rb`設定された環境変数で構成されたプロバイダーを使用して OmniAuth ミドルウェアを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-123">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in `oauth_environment_variables.rb`.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="bfc95-124">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="bfc95-124">Implement sign-in</span></span>

<span data-ttu-id="bfc95-125">OmniAuth ミドルウェアの構成が完了したので、アプリへのサインインの追加に進むことができます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-125">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span> <span data-ttu-id="bfc95-126">CLI で次のコマンドを実行して、サインインおよびサインアウト用のコントローラーを生成します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-126">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

```Shell
rails generate controller Auth
```

<span data-ttu-id="bfc95-127">`./app/controllers/auth_controller.rb` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-127">Open the `./app/controllers/auth_controller.rb` file.</span></span> <span data-ttu-id="bfc95-128">次に示すメソッドを `AuthController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-128">Add the following method to the `AuthController` class.</span></span>

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

<span data-ttu-id="bfc95-129">このメソッドはすべて、カスタム戦略を呼び出すことが期待される OmniAuth のルートにリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-129">All this method does is redirect to the route that OmniAuth expects to invoke our custom strategy.</span></span>

<span data-ttu-id="bfc95-130">次に、コールバックメソッドを`AuthController`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-130">Next, add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="bfc95-131">このメソッドは、OAuth フローの完了後に OmniAuth ミドルウェアによって呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-131">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

<span data-ttu-id="bfc95-132">ここでは、OmniAuth によって提供されるハッシュをレンダリングします。</span><span class="sxs-lookup"><span data-stu-id="bfc95-132">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="bfc95-133">これを使用して、サインインが機能していることを確認してから、に進みます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-133">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="bfc95-134">テストを開始する前に、に`./config/routes.rb`ルートを追加する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-134">Before we test, we need to add the routes to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

<span data-ttu-id="bfc95-135">`signin`アクションを使用するようにビューを更新します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-135">Now update the views to use the `signin` action.</span></span> <span data-ttu-id="bfc95-136">開き`./app/views/layouts/application.html.erb`ます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-136">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="bfc95-137">行`<a href="#" class="nav-link">Sign In</a>`を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-137">Replace the line `<a href="#" class="nav-link">Sign In</a>` with the following.</span></span>

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

<span data-ttu-id="bfc95-138">`./app/views/home/index.html.erb`ファイルを開き、 `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>`行を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-138">Open the `./app/views/home/index.html.erb` file and replace the `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>` line with the following.</span></span>

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

<span data-ttu-id="bfc95-139">サーバーを起動し、を`https://localhost:3000`参照します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-139">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="bfc95-140">[サインイン] ボタンをクリックすると、に`https://login.microsoftonline.com`リダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-140">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="bfc95-141">Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-141">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="bfc95-142">ブラウザーがアプリにリダイレクトし、OmniAuth によって生成されたハッシュが表示されます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-142">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

```json
{
  "provider": "microsoft_graph_auth",
  "uid": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
  "info": {
    "name": null
  },
  "credentials": {
    "token": "eyJ0eXAi...",
    "refresh_token": "OAQABAAA...",
    "expires_at": 1529517383,
    "expires": true
  },
  "extra": {
    "raw_info": {
      "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
      "id": "eb52b3b2-c4ac-4b4f-bacd-d5f7ece55df0",
      "businessPhones": [
        "+1 425 555 0109"
      ],
      "displayName": "Adele Vance",
      "givenName": "Adele",
      "jobTitle": "Retail Manager",
      "mail": "AdeleV@contoso.onmicrosoft.com",
      "mobilePhone": null,
      "officeLocation": "18/2111",
      "preferredLanguage": "en-US",
      "surname": "Vance",
      "userPrincipalName": "AdeleV@contoso.onmicrosoft.com"
    }
  }
}
```

## <a name="storing-the-tokens"></a><span data-ttu-id="bfc95-143">トークンの格納</span><span class="sxs-lookup"><span data-stu-id="bfc95-143">Storing the tokens</span></span>

<span data-ttu-id="bfc95-144">トークンを入手できるようになったので、これをアプリに保存する方法を実装します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-144">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="bfc95-145">これはサンプルアプリなので、わかりやすくするために、セッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-145">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="bfc95-146">実際のアプリケーションでは、データベースのような、より信頼性の高いセキュリティで保護されたストレージソリューションを使用します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-146">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="bfc95-147">`./app/controllers/application_controller.rb` ファイルを開きます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-147">Open the `./app/controllers/application_controller.rb` file.</span></span> <span data-ttu-id="bfc95-148">ここでは、すべてのトークン管理メソッドを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-148">You'll add all of our token management methods here.</span></span> <span data-ttu-id="bfc95-149">他のすべてのコントローラーは`ApplicationController`クラスを継承するため、これらのメソッドを使用してトークンにアクセスできるようになります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-149">Because all of the other controllers inherit the `ApplicationController` class, they'll be able to use these methods to access the tokens.</span></span>

<span data-ttu-id="bfc95-150">次に示すメソッドを `ApplicationController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-150">Add the following method to the `ApplicationController` class.</span></span> <span data-ttu-id="bfc95-151">このメソッドは、OmniAuth ハッシュをパラメーターとして取得し、関連する情報のビットを抽出して、セッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-151">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

```ruby
def save_in_session(auth_hash)
  # Save the token info
  session[:graph_token_hash] = auth_hash.dig(:credentials)
  # Save the user's display name
  session[:user_name] = auth_hash.dig(:extra, :raw_info, :displayName)
  # Save the user's email address
  # Use the mail field first. If that's empty, fall back on
  # userPrincipalName
  session[:user_email] = auth_hash.dig(:extra, :raw_info, :mail) ||
                         auth_hash.dig(:extra, :raw_info, :userPrincipalName)
end
```

<span data-ttu-id="bfc95-152">次に、アクセサー関数を追加して、ユーザー名、電子メールアドレス、およびアクセストークンをセッションから取得します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-152">Now add accessor functions to retrieve the user name, email address, and access token back out of the session.</span></span>

```ruby
def user_name
  session[:user_name]
end

def user_email
  session[:user_email]
end

def access_token
  session[:graph_token_hash][:token]
end
```

<span data-ttu-id="bfc95-153">最後に、アクションが処理される前に実行されるコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-153">Finally, add some code that will run before any action is processed.</span></span>

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

<span data-ttu-id="bfc95-154">このメソッドは、ナビゲーションバーでユーザーの情報`application.html.erb`を表示するためにレイアウトで使用される変数を設定します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-154">This method sets the variables that the layout (in `application.html.erb`) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="bfc95-155">ここで追加することで、単一のコントローラーアクションごとにこのコードを追加する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="bfc95-155">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="bfc95-156">ただし、これは、最適化されてい`AuthController`ないののアクションにも実行されます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-156">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span> <span data-ttu-id="bfc95-157">Before アクションをスキップするに`AuthController`は、 `./app/controllers/auth_controller.rb`のクラスに次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-157">Add the following code to the `AuthController` class in `./app/controllers/auth_controller.rb` to skip the before action.</span></span>

```ruby
skip_before_action :set_user
```

<span data-ttu-id="bfc95-158">次に、 `AuthController`クラス`callback`の関数を更新してトークンをセッションに格納し、メインページにリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="bfc95-158">Then, update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="bfc95-159">既存の `callback` 関数を、以下の関数で置換します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-159">Replace the existing `callback` function with the following.</span></span>

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a><span data-ttu-id="bfc95-160">サインアウトを実装する</span><span class="sxs-lookup"><span data-stu-id="bfc95-160">Implement sign-out</span></span>

<span data-ttu-id="bfc95-161">この新しい機能をテストする前に、サインアウトする方法を追加します。`AuthController`クラスに次のアクションを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-161">Before you test this new feature, add a way to sign out. Add the following action to the `AuthController` class.</span></span>

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

<span data-ttu-id="bfc95-162">このアクションをに`./config/routes.rb`追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-162">Add this action to `./config/routes.rb`.</span></span>

```ruby
get 'auth/signout'
```

<span data-ttu-id="bfc95-163">`signout`アクションを使用するようにビューを更新します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-163">Now update the view to use the `signout` action.</span></span> <span data-ttu-id="bfc95-164">開き`./app/views/layouts/application.html.erb`ます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-164">Open `./app/views/layouts/application.html.erb`.</span></span> <span data-ttu-id="bfc95-165">行`<a href="#" class="dropdown-item">Sign Out</a>`を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-165">Replace the line `<a href="#" class="dropdown-item">Sign Out</a>` with:</span></span>

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

<span data-ttu-id="bfc95-166">サーバーを再起動し、サインインプロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-166">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="bfc95-167">ホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-167">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![サインイン後のホームページのスクリーンショット](./images/add-aad-auth-01.png)

<span data-ttu-id="bfc95-169">右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="bfc95-169">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="bfc95-170">[**サインアウト**] をクリックすると、セッションがリセットされ、ホームページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-170">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![[サインアウト] リンクがあるドロップダウンメニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="bfc95-172">トークンの更新</span><span class="sxs-lookup"><span data-stu-id="bfc95-172">Refreshing tokens</span></span>

<span data-ttu-id="bfc95-173">OmniAuth によって生成されたハッシュをよく見ると、ハッシュには`token`と`refresh_token`の2つのトークンがあることがわかります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-173">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="bfc95-174">の`token`値は、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンです。</span><span class="sxs-lookup"><span data-stu-id="bfc95-174">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="bfc95-175">これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="bfc95-175">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="bfc95-176">ただし、このトークンは存続期間が短くなります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-176">However, this token is short-lived.</span></span> <span data-ttu-id="bfc95-177">トークンが発行された後、有効期限が切れる時間になります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-177">The token expires an hour after it is issued.</span></span> <span data-ttu-id="bfc95-178">この`refresh_token`値が有効になります。</span><span class="sxs-lookup"><span data-stu-id="bfc95-178">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="bfc95-179">更新トークンを使用すると、ユーザーが再度サインインする必要なく、新しいアクセストークンをアプリで要求できます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-179">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="bfc95-180">トークンの更新を実装するために、トークン管理コードを更新します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-180">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="bfc95-181">を`./app/controllers/application_controller.rb`開き、先頭に`require`次のステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-181">Open `./app/controllers/application_controller.rb` and add the following `require` statements at the top:</span></span>

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

<span data-ttu-id="bfc95-182">その後、次のメソッドを`ApplicationController`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-182">Then add the following method to the `ApplicationController` class.</span></span>

```ruby
def refresh_tokens(token_hash)
  oauth_strategy = OmniAuth::Strategies::MicrosoftGraphAuth.new(
    nil, ENV['AZURE_APP_ID'], ENV['AZURE_APP_SECRET']
  )

  token = OAuth2::AccessToken.new(
    oauth_strategy.client, token_hash[:token],
    refresh_token: token_hash[:refresh_token]
  )

  # Refresh the tokens
  new_tokens = token.refresh!.to_hash.slice(:access_token, :refresh_token, :expires_at)

  # Rename token key
  new_tokens[:token] = new_tokens.delete :access_token

  # Store the new hash
  session[:graph_token_hash] = new_tokens
end
```

<span data-ttu-id="bfc95-183">このメソッドは、 [oauth2](https://github.com/oauth-xx/oauth2) gem ( `omniauth-oauth2` gem の依存関係) を使用してトークンを更新し、セッションを更新します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-183">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

<span data-ttu-id="bfc95-184">では、このメソッドを使用します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-184">Now put this method to use.</span></span> <span data-ttu-id="bfc95-185">そのためには、 `access_token` `ApplicationController`クラスのアクセサーを少し賢いものにしてください。</span><span class="sxs-lookup"><span data-stu-id="bfc95-185">To do that, make the `access_token` accessor in the `ApplicationController` class a bit smarter.</span></span> <span data-ttu-id="bfc95-186">セッションからトークンを返すだけでなく、最初に有効期限が近づいているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="bfc95-186">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="bfc95-187">その場合は、トークンを返す前に更新されます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-187">If it is, then it will refresh before returning the token.</span></span> <span data-ttu-id="bfc95-188">現在`access_token`のメソッドを次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="bfc95-188">Replace the current `access_token` method with the following.</span></span>

```ruby
def access_token
  token_hash = session[:graph_token_hash]

  # Get the expiry time - 5 minutes
  expiry = Time.at(token_hash[:expires_at] - 300)

  if Time.now > expiry
    # Token expired, refresh
    new_hash = refresh_tokens token_hash
    new_hash[:token]
  else
    token_hash[:token]
  end
end
```