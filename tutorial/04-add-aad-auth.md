<!-- markdownlint-disable MD002 MD041 -->

この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。 これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。 この手順では、omniauth と[oauth2](https://github.com/omniauth/omniauth-oauth2)の gem をアプリケーションに統合し、カスタム omniauth 戦略を作成します。

1. アプリ ID とシークレットを保持する個別のファイルを作成します。 /Config フォルダーにという`oauth_environment_variables.rb`新しいファイル **./config**を作成し、次のコードを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換え、生成し`YOUR_APP_SECRET_HERE`たパスワードに置き換えます。

    > [!IMPORTANT]
    > Git などのソース管理を使用している場合は、アプリ ID とパスワードを誤っ`oauth_environment_variables.rb`てリークしないように、ソース管理からファイルを除外することをお勧めします。

1. を開き、次のコードを行の`Rails.application.initialize!`前に追加し**ます。**

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a>セットアップ OmniAuth

既に gem が`omniauth-oauth2`インストールされていますが、Azure OAuth エンドポイントを使用できるようにするためには、 [OAuth2 戦略を作成](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)する必要があります。 これは、Azure プロバイダーに OAuth 要求を作成するためのパラメーターを定義する Ruby クラスです。

1. **./Lib**' * * `microsoft_graph_auth.rb`フォルダーに新しいファイルを作成し、次のコードを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    このコードの内容を確認してください。

    - を設定し`client_options`て、Microsoft identity platform エンドポイントを指定します。
    - 承認フェーズで`scope`パラメーターを送信する必要があることを指定します。
    - ユーザーの`id`プロパティをユーザーの一意の ID としてマップします。
    - アクセストークンを使用して、Microsoft Graph からユーザーのプロファイルを取得し、 `raw_info`ハッシュを入力します。
    - これは、コールバック URL を上書きして、アプリ登録ポータルで登録されているコールバックと一致するようにします。

1. `omniauth_graph.rb` **./Config/イニシャライザー**フォルダーに新しいファイルを作成し、次のコードを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    このコードは、アプリが開始するときに実行されます。 **Oauth_environment_variables rb**で設定された環境`microsoft_graph_auth`変数で構成されたプロバイダーを使用して、OmniAuth ミドルウェアを読み込みます。

## <a name="implement-sign-in"></a>サインインの実装

OmniAuth ミドルウェアの構成が完了したので、アプリへのサインインの追加に進むことができます。

1. CLI で次のコマンドを実行して、サインインおよびサインアウト用のコントローラーを生成します。

    ```Shell
    rails generate controller Auth
    ```

1. **/App/controllers/auth_controller rb**を開きます。 コールバックメソッドを`AuthController`クラスに追加します。 このメソッドは、OAuth フローの完了後に OmniAuth ミドルウェアによって呼び出されます。

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    ここでは、OmniAuth によって提供されるハッシュをレンダリングします。 これを使用して、サインインが機能していることを確認してから、に進みます。

1. **/Config/routes.rb**にルートを追加します。

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. サーバーを起動し、を`https://localhost:3000`参照します。 [サインイン] ボタンをクリックすると、`https://login.microsoftonline.com` にリダイレクトされます。 Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。 ブラウザーがアプリにリダイレクトし、OmniAuth によって生成されたハッシュが表示されます。

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

トークンを取得できるようになったので、トークンをアプリに格納する手順を実装します。 これはサンプルアプリなので、わかりやすくするために、セッションに格納します。 実際のアプリでは、データベースのような、より信頼性の高い安全なストレージ ソリューションを使用します。

1. **/App/controllers/application_controller rb**を開きます。 次のメソッドを `ApplicationController` クラスに追加します。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    このメソッドは、OmniAuth ハッシュをパラメーターとして取得し、関連する情報のビットを抽出して、セッションに格納します。

1. ユーザー名、電子メール`ApplicationController`アドレス、およびアクセストークンをセッションから取得するために、アクセサー関数をクラスに追加します。

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

1. アクションが処理される前`ApplicationController`に実行されるクラスに、次のコードを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    このメソッドは、ナビゲーションバーにユーザーの情報を表示するために (**アプリケーション .html**で) レイアウトで使用する変数を設定します。 ここで追加することで、単一のコントローラーアクションごとにこのコードを追加する必要はありません。 ただし、これは、最適化されてい`AuthController`ないののアクションにも実行されます。

1. 次のコードを`AuthController` **/app/controllers/auth_controller rb**のクラスに追加して、before アクションをスキップします。

    ```ruby
    skip_before_action :set_user
    ```

1. `AuthController`クラスの`callback`関数を更新して、トークンをセッションに格納し、メインページにリダイレクトします。 既存の `callback` 関数を、以下の関数で置き換えます。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a>サインアウトを実装する

この新しい機能をテストする前に、サインアウトする方法を追加します。

1. `AuthController`クラスに次のアクションを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. このアクションを/config/routes.rb に追加します **。**

    ```ruby
    get 'auth/signout'
    ```

1. サーバーを再起動し、サインインプロセスを実行します。 ホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. 右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。 **[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>トークンの更新

OmniAuth によって生成されたハッシュをよく見ると、ハッシュには`token`と`refresh_token`の2つのトークンがあることがわかります。 の`token`値は、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンです。 これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。

ただし、このトークンは一時的なものです。 トークンが発行された後、有効期限が切れる時間になります。 この`refresh_token`値が有効になります。 更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。 トークンの更新を実装するために、トークン管理コードを更新します。

1. **/App/controllers/application_controller rb**を開き、先頭に次`require`のステートメントを追加します。

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. 次のメソッドを `ApplicationController` クラスに追加します。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    このメソッドは、 [oauth2](https://github.com/oauth-xx/oauth2) gem ( `omniauth-oauth2` gem の依存関係) を使用してトークンを更新し、セッションを更新します。

1. 現在`access_token`のメソッドを次のように置き換えます。

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    セッションからトークンを返すだけでなく、最初に有効期限が近づいているかどうかを確認します。 その場合は、トークンを返す前に更新されます。
