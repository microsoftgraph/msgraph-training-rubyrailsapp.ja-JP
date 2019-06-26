<!-- markdownlint-disable MD002 MD041 -->

この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。 これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。 この手順では、omniauth と[oauth2](https://github.com/omniauth/omniauth-oauth2)の gem をアプリケーションに統合し、カスタム omniauth 戦略を作成します。

最初に、アプリ ID とシークレットを保持する個別のファイルを作成します。 `./config`フォルダーにという`oauth_environment_variables.rb`新しいファイルを作成し、次のコードを追加します。

```ruby
ENV['AZURE_APP_ID'] = 'YOUR_APP_ID_HERE'
ENV['AZURE_APP_SECRET'] = 'YOUR_APP_SECRET_HERE'
ENV['AZURE_SCOPES'] = 'openid profile email offline_access user.read calendars.read'
```

を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換え、生成し`YOUR_APP_SECRET_HERE`たパスワードに置き換えます。

> [!IMPORTANT]
> Git などのソース管理を使用している場合は、アプリ ID とパスワードを誤っ`oauth_environment_variables.rb`てリークしないように、ソース管理からファイルを除外することをお勧めします。

このファイルが存在する場合、このファイルを読み込むコードを追加します。 `./config/environment.rb`ファイルを開き、行の`Rails.application.initialize!`前に次のコードを追加します。

```ruby
# Load OAuth settings
oauth_environment_variables = File.join(Rails.root, 'config', 'oauth_environment_variables.rb')
load(oauth_environment_variables) if File.exist?(oauth_environment_variables)
```

## <a name="setup-omniauth"></a>セットアップ OmniAuth

既に gem が`omniauth-oauth2`インストールされていますが、Azure OAuth エンドポイントを使用できるようにするためには、 [OAuth2 戦略を作成](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)する必要があります。 これは、Azure プロバイダーに OAuth 要求を作成するためのパラメーターを定義する Ruby クラスです。

`./lib`フォルダーにという`microsoft_graph_auth.rb`新しいファイルを作成し、次のコードを追加します。

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

このコードの内容を確認してください。

- は、 `client_options` Azure v2 エンドポイントを指定するように設定します。
- 承認フェーズで`scope`パラメーターを送信する必要があることを指定します。
- ユーザーの`id`プロパティをユーザーの一意の ID としてマップします。
- アクセストークンを使用して、Microsoft Graph からユーザーのプロファイルを取得し、 `raw_info`ハッシュを入力します。
- これは、コールバック URL を上書きして、アプリ登録ポータルで登録されているコールバックと一致するようにします。

戦略を定義したので、これを使用するように OmniAuth を構成する必要があります。 `./config/initializers`フォルダーにという`omniauth_graph.rb`新しいファイルを作成し、次のコードを追加します。

```ruby
require 'microsoft_graph_auth'

Rails.application.config.middleware.use OmniAuth::Builder do
  provider :microsoft_graph_auth,
           ENV['AZURE_APP_ID'],
           ENV['AZURE_APP_SECRET'],
           scope: ENV['AZURE_SCOPES']
end
```

このコードは、アプリが開始するときに実行されます。 このメソッドは、で`microsoft_graph_auth` `oauth_environment_variables.rb`設定された環境変数で構成されたプロバイダーを使用して OmniAuth ミドルウェアを読み込みます。

## <a name="implement-sign-in"></a>サインインの実装

OmniAuth ミドルウェアの構成が完了したので、アプリへのサインインの追加に進むことができます。 CLI で次のコマンドを実行して、サインインおよびサインアウト用のコントローラーを生成します。

```Shell
rails generate controller Auth
```

`./app/controllers/auth_controller.rb` ファイルを開きます。 次に示すメソッドを `AuthController` クラスに追加します。

```ruby
def signin
  redirect_to '/auth/microsoft_graph_auth'
end
```

このメソッドはすべて、カスタム戦略を呼び出すことが期待される OmniAuth のルートにリダイレクトされます。

次に、コールバックメソッドを`AuthController`クラスに追加します。 このメソッドは、OAuth フローの完了後に OmniAuth ミドルウェアによって呼び出されます。

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Temporary for testing!
  render json: data.to_json
end
```

ここでは、OmniAuth によって提供されるハッシュをレンダリングします。 これを使用して、サインインが機能していることを確認してから、に進みます。 テストを開始する前に、に`./config/routes.rb`ルートを追加する必要があります。

```ruby
get 'auth/signin'

# Add route for OmniAuth callback
match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
```

`signin`アクションを使用するようにビューを更新します。 開き`./app/views/layouts/application.html.erb`ます。 行`<a href="#" class="nav-link">Sign In</a>`を次のように置き換えます。

```html
<%= link_to "Sign In", {:controller => :auth, :action => :signin}, :class => "nav-link" %>
```

`./app/views/home/index.html.erb`ファイルを開き、 `<a href="#" class="btn btn-primary btn-large">Click here to sign in</a>`行を次のように置き換えます。

```html
<%= link_to "Click here to sign in", {:controller => :auth, :action => :signin}, :class => "btn btn-primary btn-large" %>
```

サーバーを起動し、を`https://localhost:3000`参照します。 [サインイン] ボタンをクリックすると、に`https://login.microsoftonline.com`リダイレクトされます。 Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。 ブラウザーがアプリにリダイレクトし、OmniAuth によって生成されたハッシュが表示されます。

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

## <a name="storing-the-tokens"></a>トークンの格納

トークンを入手できるようになったので、これをアプリに保存する方法を実装します。 これはサンプルアプリなので、わかりやすくするために、セッションに格納します。 実際のアプリケーションでは、データベースのような、より信頼性の高いセキュリティで保護されたストレージソリューションを使用します。

`./app/controllers/application_controller.rb` ファイルを開きます。 ここでは、すべてのトークン管理メソッドを追加します。 他のすべてのコントローラーは`ApplicationController`クラスを継承するため、これらのメソッドを使用してトークンにアクセスできるようになります。

次に示すメソッドを `ApplicationController` クラスに追加します。 このメソッドは、OmniAuth ハッシュをパラメーターとして取得し、関連する情報のビットを抽出して、セッションに格納します。

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

次に、アクセサー関数を追加して、ユーザー名、電子メールアドレス、およびアクセストークンをセッションから取得します。

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

最後に、アクションが処理される前に実行されるコードを追加します。

```ruby
before_action :set_user

def set_user
  @user_name = user_name
  @user_email = user_email
end
```

このメソッドは、ナビゲーションバーでユーザーの情報`application.html.erb`を表示するためにレイアウトで使用される変数を設定します。 ここで追加することで、単一のコントローラーアクションごとにこのコードを追加する必要はありません。 ただし、これは、最適化されてい`AuthController`ないののアクションにも実行されます。 Before アクションをスキップするに`AuthController`は、 `./app/controllers/auth_controller.rb`のクラスに次のコードを追加します。

```ruby
skip_before_action :set_user
```

次に、 `AuthController`クラス`callback`の関数を更新してトークンをセッションに格納し、メインページにリダイレクトします。 既存の `callback` 関数を、以下の関数で置換します。

```ruby
def callback
  # Access the authentication hash for omniauth
  data = request.env['omniauth.auth']

  # Save the data in the session
  save_in_session data

  redirect_to root_url
end
```

## <a name="implement-sign-out"></a>サインアウトを実装する

この新しい機能をテストする前に、サインアウトする方法を追加します。`AuthController`クラスに次のアクションを追加します。

```ruby
def signout
  reset_session
  redirect_to root_url
end
```

このアクションをに`./config/routes.rb`追加します。

```ruby
get 'auth/signout'
```

`signout`アクションを使用するようにビューを更新します。 開き`./app/views/layouts/application.html.erb`ます。 行`<a href="#" class="dropdown-item">Sign Out</a>`を次のように置き換えます。

```html
<%= link_to "Sign Out", {:controller => :auth, :action => :signout}, :class => "dropdown-item" %>
```

サーバーを再起動し、サインインプロセスを実行します。 ホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。

![サインイン後のホームページのスクリーンショット](./images/add-aad-auth-01.png)

右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。 [**サインアウト**] をクリックすると、セッションがリセットされ、ホームページに戻ります。

![[サインアウト] リンクがあるドロップダウンメニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>トークンの更新

OmniAuth によって生成されたハッシュをよく見ると、ハッシュには`token`と`refresh_token`の2つのトークンがあることがわかります。 の`token`値は、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンです。 これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。

ただし、このトークンは存続期間が短くなります。 トークンが発行された後、有効期限が切れる時間になります。 この`refresh_token`値が有効になります。 更新トークンを使用すると、ユーザーが再度サインインする必要なく、新しいアクセストークンをアプリで要求できます。 トークンの更新を実装するために、トークン管理コードを更新します。

を`./app/controllers/application_controller.rb`開き、先頭に`require`次のステートメントを追加します。

```ruby
require 'microsoft_graph_auth'
require 'oauth2'
```

その後、次のメソッドを`ApplicationController`クラスに追加します。

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

このメソッドは、 [oauth2](https://github.com/oauth-xx/oauth2) gem ( `omniauth-oauth2` gem の依存関係) を使用してトークンを更新し、セッションを更新します。

では、このメソッドを使用します。 そのためには、 `access_token` `ApplicationController`クラスのアクセサーを少し賢いものにしてください。 セッションからトークンを返すだけでなく、最初に有効期限が近づいているかどうかを確認します。 その場合は、トークンを返す前に更新されます。 現在`access_token`のメソッドを次のように置き換えます。

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
