<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5dc42-101">この演習では、前の演習のアプリケーションを拡張して、Azure AD での認証をサポートします。</span><span class="sxs-lookup"><span data-stu-id="5dc42-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="5dc42-102">これは、Microsoft Graph を呼び出すのに必要な OAuth アクセス トークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="5dc42-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="5dc42-103">この手順では [、omniauth-oauth2 gem](https://github.com/omniauth/omniauth-oauth2) をアプリケーションに統合し、カスタムの OmniAuth 戦略を作成します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="5dc42-104">アプリ ID とシークレットを保持する別のファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="5dc42-105">./config フォルダーに新しい `oauth_environment_variables.rb` **ファイルを** 作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="5dc42-106">アプリケーション `YOUR_APP_ID_HERE` 登録ポータルのアプリケーション ID に置き換え、生成したパスワード `YOUR_APP_SECRET_HERE` に置き換える。</span><span class="sxs-lookup"><span data-stu-id="5dc42-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="5dc42-107">git などのソース管理を使用している場合は、アプリ ID とパスワードが誤って漏洩しないように、ファイルをソース管理から除外する良い時期です `oauth_environment_variables.rb` 。</span><span class="sxs-lookup"><span data-stu-id="5dc42-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="5dc42-108">**./config/environment.rb** を開き、行の前に次のコードを追加 `Rails.application.initialize!` します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="5dc42-109">OmniAuth のセットアップ</span><span class="sxs-lookup"><span data-stu-id="5dc42-109">Setup OmniAuth</span></span>

<span data-ttu-id="5dc42-110">既に gem をインストールしていますが、Azure OAuth エンドポイントで動作するには `omniauth-oauth2` [、OAuth2](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)戦略を作成する必要があります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="5dc42-111">これは、Azure プロバイダーに OAuth 要求を行うパラメーターを定義する Ruby クラスです。</span><span class="sxs-lookup"><span data-stu-id="5dc42-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="5dc42-112">`microsoft_graph_auth.rb` \*\*./lib '\*\*\*\* フォルダー内に新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="5dc42-113">このコードの動作を確認してください。</span><span class="sxs-lookup"><span data-stu-id="5dc42-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="5dc42-114">Microsoft ID プラットフォーム `client_options` エンドポイントを指定する設定を行います。</span><span class="sxs-lookup"><span data-stu-id="5dc42-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="5dc42-115">認証フェーズ中 `scope` にパラメーターを送信するように指定します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="5dc42-116">ユーザーの `id` プロパティをユーザーの一意の ID としてマップします。</span><span class="sxs-lookup"><span data-stu-id="5dc42-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="5dc42-117">アクセス トークンを使用して Microsoft Graph からユーザーのプロファイルを取得し、ハッシュを入力 `raw_info` します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="5dc42-118">コールバック URL を上書きして、アプリ登録ポータルで登録されたコールバックと一致する必要があります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="5dc42-119">`omniauth_graph.rb` **./config/initializers** フォルダーで呼び出される新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="5dc42-120">このコードは、アプリの起動時に実行されます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-120">This code will execute when the app starts.</span></span> <span data-ttu-id="5dc42-121">これは、oauth_environment_variables.rb で設定された環境変数で構成されたプロバイダーを使用して `microsoft_graph_auth` **OmniAuth ミドルウェアを読み込みます**。</span><span class="sxs-lookup"><span data-stu-id="5dc42-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="5dc42-122">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="5dc42-122">Implement sign-in</span></span>

<span data-ttu-id="5dc42-123">OmniAuth ミドルウェアが構成されたので、次にアプリへのサインインの追加に進む必要があります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="5dc42-124">CLI で次のコマンドを実行して、サインインおよびサインアウト用のコントローラーを生成します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="5dc42-125">**./app/controllers/auth_controller.rb を開きます**。</span><span class="sxs-lookup"><span data-stu-id="5dc42-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="5dc42-126">コールバック メソッドをクラスに追加 `AuthController` します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="5dc42-127">このメソッドは、OAuth フローが完了すると OmniAuth ミドルウェアによって呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="5dc42-128">今のところ、OmniAuth によって提供されるハッシュをレンダリングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="5dc42-129">これを使用して、サインインが機能しているか確認してから、次に進む必要があります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="5dc42-130">ルートを **./config/routes.rb に追加します**。</span><span class="sxs-lookup"><span data-stu-id="5dc42-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', :to => 'auth#callback', :via => [:get, :post]
    ```

1. <span data-ttu-id="5dc42-131">サーバーを起動し、参照します `https://localhost:3000` 。</span><span class="sxs-lookup"><span data-stu-id="5dc42-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="5dc42-132">[サインイン] ボタンをクリックすると、`https://login.microsoftonline.com` にリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="5dc42-133">Microsoft アカウントでログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="5dc42-134">ブラウザーはアプリにリダイレクトされ、OmniAuth によって生成されたハッシュが表示されます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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
          "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users(displayName,mail,mailboxSettings,userPrincipalName)/$entity",
          "displayName": "Lynne Robbins",
          "mail": "LynneR@contoso.OnMicrosoft.com",
          "userPrincipalName": "LynneR@contoso.OnMicrosoft.com",
          "id": "d294e784-840e-4f9f-bb1e-95c0a75f2f18@2d18179c-4386-4cbd-8891-7fd867c4f62e",
          "mailboxSettings": {
            "archiveFolder": "AAMkAGI2...",
            "timeZone": "Pacific Standard Time",
            "delegateMeetingMessageDeliveryOptions": "sendToDelegateOnly",
            "dateFormat": "M/d/yyyy",
            "timeFormat": "h:mm tt",
            "automaticRepliesSetting": {
              "status": "disabled",
              "externalAudience": "all",
              "internalReplyMessage": "",
              "externalReplyMessage": "",
              "scheduledStartDateTime": {
                "dateTime": "2020-12-09T17:00:00.0000000",
                "timeZone": "UTC"
              },
              "scheduledEndDateTime": {
                "dateTime": "2020-12-10T17:00:00.0000000",
                "timeZone": "UTC"
              }
            },
            "language": {
              "locale": "en-US",
              "displayName": "English (United States)"
            },
            "workingHours": {
              "daysOfWeek": [
                "monday",
                "tuesday",
                "wednesday",
                "thursday",
                "friday"
              ],
              "startTime": "08:00:00.0000000",
              "endTime": "17:00:00.0000000",
              "timeZone": {
                "name": "Pacific Standard Time"
              }
            }
          }
        }
      }
    }
    ```

## <a name="storing-the-tokens"></a><span data-ttu-id="5dc42-135">トークンの格納</span><span class="sxs-lookup"><span data-stu-id="5dc42-135">Storing the tokens</span></span>

<span data-ttu-id="5dc42-136">トークンを取得できるようになったので、トークンをアプリに格納する手順を実装します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="5dc42-137">これはサンプル アプリのため、わかりやすくするために、セッションに保存します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="5dc42-138">実際のアプリでは、データベースのような、より信頼性の高い安全なストレージ ソリューションを使用します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="5dc42-139">**./app/controllers/application_controller.rb を開きます**。</span><span class="sxs-lookup"><span data-stu-id="5dc42-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="5dc42-140">次のメソッドを `ApplicationController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="5dc42-141">このメソッドは、OmniAuth ハッシュをパラメーターとして受け取り、関連する情報を抽出し、それをセッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="5dc42-142">アクセサー関数をクラスに追加して、ユーザー名、電子メール アドレス、およびアクセス トークンをセッション `ApplicationController` から取り出します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

    ```ruby
    def user_name
      session[:user_name]
    end

    def user_email
      session[:user_email]
    end

    def user_timezone
      session[:user_timezone]
    end

    def access_token
      session[:graph_token_hash][:token]
    end
    ```

1. <span data-ttu-id="5dc42-143">次のコードを、アクション `ApplicationController` が処理される前に実行するクラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="5dc42-144">このメソッドは、(application.htm **l.erb** 内の) レイアウトがナビゲーション バーにユーザーの情報を表示するために使用する変数を設定します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="5dc42-145">このコードをここに追加することで、コントローラーのすべての操作でこのコードを追加する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="5dc42-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="5dc42-146">ただし、これは最適ではない、内のアクション `AuthController` にも実行されます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="5dc42-147">`AuthController` **./app/controllers/auth_controller.rb** のクラスに次のコードを追加して、before アクションをスキップします。</span><span class="sxs-lookup"><span data-stu-id="5dc42-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="5dc42-148">クラス内 `callback` の関数を `AuthController` 更新して、トークンをセッションに格納し、メイン ページにリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="5dc42-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="5dc42-149">既存の `callback` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="5dc42-150">サインアウトを実装する</span><span class="sxs-lookup"><span data-stu-id="5dc42-150">Implement sign-out</span></span>

<span data-ttu-id="5dc42-151">この新機能をテストする前に、サインアウトする方法を追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="5dc42-152">次のアクションをクラスに追加 `AuthController` します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="5dc42-153">このアクションを **./config/routes.rb に追加します**。</span><span class="sxs-lookup"><span data-stu-id="5dc42-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="5dc42-154">サーバーを再起動し、サインイン プロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="5dc42-155">ホーム ページに戻る必要がありますが、サインイン中を示す UI が変更される必要があります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. <span data-ttu-id="5dc42-157">右上隅にあるユーザー アバターをクリックして、[サインアウト **] リンクにアクセス** します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="5dc42-158">**[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="5dc42-160">トークンの更新</span><span class="sxs-lookup"><span data-stu-id="5dc42-160">Refreshing tokens</span></span>

<span data-ttu-id="5dc42-161">OmniAuth によって生成されたハッシュを注意して見た場合、ハッシュには次の 2 つのトークンがあります `token` `refresh_token` 。</span><span class="sxs-lookup"><span data-stu-id="5dc42-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="5dc42-162">値はアクセス トークンで、API 呼び出しのヘッダー `token` `Authorization` で送信されます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="5dc42-163">これは、アプリがユーザーの代わりに Microsoft Graph にアクセスできるトークンです。</span><span class="sxs-lookup"><span data-stu-id="5dc42-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="5dc42-164">ただし、このトークンは一時的なものです。</span><span class="sxs-lookup"><span data-stu-id="5dc42-164">However, this token is short-lived.</span></span> <span data-ttu-id="5dc42-165">トークンは発行後 1 時間で期限切れになります。</span><span class="sxs-lookup"><span data-stu-id="5dc42-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="5dc42-166">ここで、値 `refresh_token` が役に立ちます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="5dc42-167">更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="5dc42-168">トークン更新を実装するためにトークン管理コードを更新します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="5dc42-169">**./app/controllers/application_controller.rb** を開き、上部に次のステートメント `require` を追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="5dc42-170">次のメソッドを `ApplicationController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="5dc42-171">このメソッドは [、oauth2](https://github.com/oauth-xx/oauth2) gem (gem の依存関係) を使用してトークンを更新し、 `omniauth-oauth2` セッションを更新します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="5dc42-172">現在のメソッドを `access_token` 次のメソッドに置き換える。</span><span class="sxs-lookup"><span data-stu-id="5dc42-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="5dc42-173">セッションからトークンを返すのではなく、最初に有効期限が近い場合に確認します。</span><span class="sxs-lookup"><span data-stu-id="5dc42-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="5dc42-174">更新されている場合は、トークンを返す前に更新されます。</span><span class="sxs-lookup"><span data-stu-id="5dc42-174">If it is, then it will refresh before returning the token.</span></span>
