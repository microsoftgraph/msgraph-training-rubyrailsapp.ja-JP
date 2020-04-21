<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a71ee-101">この演習では、Azure AD での認証をサポートするために、前の手順で作成したアプリケーションを拡張します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="a71ee-102">これは、Microsoft Graph を呼び出すために必要な OAuth アクセストークンを取得するために必要です。</span><span class="sxs-lookup"><span data-stu-id="a71ee-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="a71ee-103">この手順では、omniauth と[oauth2](https://github.com/omniauth/omniauth-oauth2)の gem をアプリケーションに統合し、カスタム omniauth 戦略を作成します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-103">In this step you will integrate the [omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) gem into the application, and create a custom OmniAuth strategy.</span></span>

1. <span data-ttu-id="a71ee-104">アプリ ID とシークレットを保持する個別のファイルを作成します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-104">Create a separate file to hold your app ID and secret.</span></span> <span data-ttu-id="a71ee-105">/Config フォルダーにという`oauth_environment_variables.rb`新しいファイル **./config**を作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-105">Create a new file called `oauth_environment_variables.rb` in the **./config** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/oauth_environment_variables.rb.example":::

1. <span data-ttu-id="a71ee-106">を`YOUR_APP_ID_HERE`アプリケーション登録ポータルからのアプリケーション ID に置き換え、生成し`YOUR_APP_SECRET_HERE`たパスワードに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-106">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_SECRET_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="a71ee-107">Git などのソース管理を使用している場合は、アプリ ID とパスワードを誤っ`oauth_environment_variables.rb`てリークしないように、ソース管理からファイルを除外することをお勧めします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-107">If you're using source control such as git, now would be a good time to exclude the `oauth_environment_variables.rb` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

1. <span data-ttu-id="a71ee-108">を開き、次のコードを行の`Rails.application.initialize!`前に追加し**ます。**</span><span class="sxs-lookup"><span data-stu-id="a71ee-108">Open **./config/environment.rb** and add the following code before the `Rails.application.initialize!` line.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/environment.rb" id="LoadOAuthSettingsSnippet" highlight="4-6":::

## <a name="setup-omniauth"></a><span data-ttu-id="a71ee-109">セットアップ OmniAuth</span><span class="sxs-lookup"><span data-stu-id="a71ee-109">Setup OmniAuth</span></span>

<span data-ttu-id="a71ee-110">既に gem が`omniauth-oauth2`インストールされていますが、Azure OAuth エンドポイントを使用できるようにするためには、 [OAuth2 戦略を作成](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy)する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a71ee-110">You've already installed the `omniauth-oauth2` gem, but in order to make it work with the Azure OAuth endpoints, you need to [create an OAuth2 strategy](https://github.com/omniauth/omniauth-oauth2#creating-an-oauth2-strategy).</span></span> <span data-ttu-id="a71ee-111">これは、Azure プロバイダーに OAuth 要求を作成するためのパラメーターを定義する Ruby クラスです。</span><span class="sxs-lookup"><span data-stu-id="a71ee-111">This is a Ruby class that defines the parameters for making OAuth requests to the Azure provider.</span></span>

1. <span data-ttu-id="a71ee-112">**./Lib**' \* \* `microsoft_graph_auth.rb`フォルダーに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-112">Create a new file called `microsoft_graph_auth.rb` in the **./lib**\`\*\* folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/lib/microsoft_graph_auth.rb" id="AuthStrategySnippet":::

    <span data-ttu-id="a71ee-113">このコードの内容を確認してください。</span><span class="sxs-lookup"><span data-stu-id="a71ee-113">Take a moment to review what this code does.</span></span>

    - <span data-ttu-id="a71ee-114">を設定し`client_options`て、Microsoft identity platform エンドポイントを指定します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-114">It sets the `client_options` to specify the Microsoft identity platform endpoints.</span></span>
    - <span data-ttu-id="a71ee-115">承認フェーズで`scope`パラメーターを送信する必要があることを指定します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-115">It specifies that the `scope` parameter should be sent during the authorize phase.</span></span>
    - <span data-ttu-id="a71ee-116">ユーザーの`id`プロパティをユーザーの一意の ID としてマップします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-116">It maps the `id` property of the user as the unique ID for the user.</span></span>
    - <span data-ttu-id="a71ee-117">アクセストークンを使用して、Microsoft Graph からユーザーのプロファイルを取得し、 `raw_info`ハッシュを入力します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-117">It uses the access token to retrieve the user's profile from Microsoft Graph to fill in the `raw_info` hash.</span></span>
    - <span data-ttu-id="a71ee-118">これは、コールバック URL を上書きして、アプリ登録ポータルで登録されているコールバックと一致するようにします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-118">It overrides the callback URL to ensure that it matches the registered callback in the app registration portal.</span></span>

1. <span data-ttu-id="a71ee-119">`omniauth_graph.rb` **./Config/イニシャライザー**フォルダーに新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-119">Create a new file called `omniauth_graph.rb` in the **./config/initializers** folder, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/omniauth_graph.rb" id="ConfigureOmniAuthSnippet":::

    <span data-ttu-id="a71ee-120">このコードは、アプリが開始するときに実行されます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-120">This code will execute when the app starts.</span></span> <span data-ttu-id="a71ee-121">**Oauth_environment_variables rb**で設定された環境`microsoft_graph_auth`変数で構成されたプロバイダーを使用して、OmniAuth ミドルウェアを読み込みます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-121">It loads up the OmniAuth middleware with the `microsoft_graph_auth` provider, configured with the environment variables set in **oauth_environment_variables.rb**.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="a71ee-122">サインインの実装</span><span class="sxs-lookup"><span data-stu-id="a71ee-122">Implement sign-in</span></span>

<span data-ttu-id="a71ee-123">OmniAuth ミドルウェアの構成が完了したので、アプリへのサインインの追加に進むことができます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-123">Now that the OmniAuth middleware is configured, you can move on to adding sign-in to the app.</span></span>

1. <span data-ttu-id="a71ee-124">CLI で次のコマンドを実行して、サインインおよびサインアウト用のコントローラーを生成します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-124">Run the following command in your CLI to generate a controller for sign-in and sign-out.</span></span>

    ```Shell
    rails generate controller Auth
    ```

1. <span data-ttu-id="a71ee-125">**/App/controllers/auth_controller rb**を開きます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-125">Open **./app/controllers/auth_controller.rb**.</span></span> <span data-ttu-id="a71ee-126">コールバックメソッドを`AuthController`クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-126">Add a callback method to the `AuthController` class.</span></span> <span data-ttu-id="a71ee-127">このメソッドは、OAuth フローの完了後に OmniAuth ミドルウェアによって呼び出されます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-127">This method will be called by the OmniAuth middleware once the OAuth flow is complete.</span></span>

    ```ruby
    def callback
      # Access the authentication hash for omniauth
      data = request.env['omniauth.auth']

      # Temporary for testing!
      render json: data.to_json
    end
    ```

    <span data-ttu-id="a71ee-128">ここでは、OmniAuth によって提供されるハッシュをレンダリングします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-128">For now all this does is render the hash provided by OmniAuth.</span></span> <span data-ttu-id="a71ee-129">これを使用して、サインインが機能していることを確認してから、に進みます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-129">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="a71ee-130">**/Config/routes.rb**にルートを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-130">Add the routes to **./config/routes.rb**.</span></span>

    ```ruby
    # Add route for OmniAuth callback
    match '/auth/:provider/callback', to: 'auth#callback', via: [:get, :post]
    ```

1. <span data-ttu-id="a71ee-131">サーバーを起動し、を`https://localhost:3000`参照します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-131">Start the server and browse to `https://localhost:3000`.</span></span> <span data-ttu-id="a71ee-132">[サインイン] ボタンをクリックすると、`https://login.microsoftonline.com` にリダイレクトされます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-132">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="a71ee-133">Microsoft アカウントを使用してログインし、要求されたアクセス許可に同意します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-133">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="a71ee-134">ブラウザーがアプリにリダイレクトし、OmniAuth によって生成されたハッシュが表示されます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-134">The browser redirects to the app, showing the hash generated by OmniAuth.</span></span>

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

## <a name="storing-the-tokens"></a><span data-ttu-id="a71ee-135">トークンの格納</span><span class="sxs-lookup"><span data-stu-id="a71ee-135">Storing the tokens</span></span>

<span data-ttu-id="a71ee-136">トークンを取得できるようになったので、トークンをアプリに格納する手順を実装します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-136">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="a71ee-137">これはサンプルアプリなので、わかりやすくするために、セッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-137">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="a71ee-138">実際のアプリでは、データベースのような、より信頼性の高い安全なストレージ ソリューションを使用します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-138">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="a71ee-139">**/App/controllers/application_controller rb**を開きます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-139">Open **./app/controllers/application_controller.rb**.</span></span> <span data-ttu-id="a71ee-140">次のメソッドを `ApplicationController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-140">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="SaveInSessionSnippet":::

    <span data-ttu-id="a71ee-141">このメソッドは、OmniAuth ハッシュをパラメーターとして取得し、関連する情報のビットを抽出して、セッションに格納します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-141">The method takes the OmniAuth hash as a parameter and extracts the relevant bits of information, then stores that in the session.</span></span>

1. <span data-ttu-id="a71ee-142">ユーザー名、電子メール`ApplicationController`アドレス、およびアクセストークンをセッションから取得するために、アクセサー関数をクラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-142">Add accessor functions to the `ApplicationController` class to retrieve the user name, email address, and access token back out of the session.</span></span>

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

1. <span data-ttu-id="a71ee-143">アクションが処理される前`ApplicationController`に実行されるクラスに、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-143">Add the following code to the `ApplicationController` class that will run before any action is processed.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="BeforeActionSnippet":::

    <span data-ttu-id="a71ee-144">このメソッドは、ナビゲーションバーにユーザーの情報を表示するために (**アプリケーション .html**で) レイアウトで使用する変数を設定します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-144">This method sets the variables that the layout (in **application.html.erb**) uses to show the user's information in the nav bar.</span></span> <span data-ttu-id="a71ee-145">ここで追加することで、単一のコントローラーアクションごとにこのコードを追加する必要はありません。</span><span class="sxs-lookup"><span data-stu-id="a71ee-145">By adding it here, you don't have to add this code in every single controller action.</span></span> <span data-ttu-id="a71ee-146">ただし、これは、最適化されてい`AuthController`ないののアクションにも実行されます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-146">However, this will also run for actions in the `AuthController`, which isn't optimal.</span></span>

1. <span data-ttu-id="a71ee-147">次のコードを`AuthController` **/app/controllers/auth_controller rb**のクラスに追加して、before アクションをスキップします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-147">Add the following code to the `AuthController` class in **./app/controllers/auth_controller.rb** to skip the before action.</span></span>

    ```ruby
    skip_before_action :set_user
    ```

1. <span data-ttu-id="a71ee-148">`AuthController`クラスの`callback`関数を更新して、トークンをセッションに格納し、メインページにリダイレクトします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-148">Update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span> <span data-ttu-id="a71ee-149">既存の `callback` 関数を、以下の関数で置き換えます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-149">Replace the existing `callback` function with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="CallbackSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="a71ee-150">サインアウトを実装する</span><span class="sxs-lookup"><span data-stu-id="a71ee-150">Implement sign-out</span></span>

<span data-ttu-id="a71ee-151">この新しい機能をテストする前に、サインアウトする方法を追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-151">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="a71ee-152">`AuthController`クラスに次のアクションを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-152">Add the following action to the `AuthController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/auth_controller.rb" id="SignOutSnippet":::

1. <span data-ttu-id="a71ee-153">このアクションを/config/routes.rb に追加します **。**</span><span class="sxs-lookup"><span data-stu-id="a71ee-153">Add this action to **./config/routes.rb**.</span></span>

    ```ruby
    get 'auth/signout'
    ```

1. <span data-ttu-id="a71ee-154">サーバーを再起動し、サインインプロセスを実行します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-154">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="a71ee-155">ホームページに戻る必要がありますが、UI は、サインインしていることを示すように変更する必要があります。</span><span class="sxs-lookup"><span data-stu-id="a71ee-155">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![サインイン後のホーム ページのスクリーンショット](./images/add-aad-auth-01.png)

1. <span data-ttu-id="a71ee-157">右上隅にあるユーザーアバターをクリックして、[**サインアウト**] リンクにアクセスします。</span><span class="sxs-lookup"><span data-stu-id="a71ee-157">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="a71ee-158">**[サインアウト]** をクリックすると、セッションがリセットされ、ホーム ページに戻ります。</span><span class="sxs-lookup"><span data-stu-id="a71ee-158">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![[サインアウト] リンクのドロップダウン メニューのスクリーンショット](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="a71ee-160">トークンの更新</span><span class="sxs-lookup"><span data-stu-id="a71ee-160">Refreshing tokens</span></span>

<span data-ttu-id="a71ee-161">OmniAuth によって生成されたハッシュをよく見ると、ハッシュには`token`と`refresh_token`の2つのトークンがあることがわかります。</span><span class="sxs-lookup"><span data-stu-id="a71ee-161">If you look closely at the hash generated by OmniAuth, you'll notice there are two tokens in the hash: `token` and `refresh_token`.</span></span> <span data-ttu-id="a71ee-162">の`token`値は、API 呼び出しの`Authorization`ヘッダーで送信されるアクセストークンです。</span><span class="sxs-lookup"><span data-stu-id="a71ee-162">The value in `token` is the access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="a71ee-163">これは、アプリがユーザーに代わって Microsoft Graph にアクセスできるようにするトークンです。</span><span class="sxs-lookup"><span data-stu-id="a71ee-163">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="a71ee-164">ただし、このトークンは一時的なものです。</span><span class="sxs-lookup"><span data-stu-id="a71ee-164">However, this token is short-lived.</span></span> <span data-ttu-id="a71ee-165">トークンが発行された後、有効期限が切れる時間になります。</span><span class="sxs-lookup"><span data-stu-id="a71ee-165">The token expires an hour after it is issued.</span></span> <span data-ttu-id="a71ee-166">この`refresh_token`値が有効になります。</span><span class="sxs-lookup"><span data-stu-id="a71ee-166">This is where the `refresh_token` value becomes useful.</span></span> <span data-ttu-id="a71ee-167">更新トークンを使用すると、ユーザーが再度サインインしなくても、アプリは新しいアクセス トークンを要求できます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-167">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="a71ee-168">トークンの更新を実装するために、トークン管理コードを更新します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-168">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="a71ee-169">**/App/controllers/application_controller rb**を開き、先頭に次`require`のステートメントを追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-169">Open **./app/controllers/application_controller.rb** and add the following `require` statements at the top:</span></span>

    ```ruby
    require 'microsoft_graph_auth'
    require 'oauth2'
    ```

1. <span data-ttu-id="a71ee-170">次のメソッドを `ApplicationController` クラスに追加します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-170">Add the following method to the `ApplicationController` class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="RefreshTokensSnippet":::

    <span data-ttu-id="a71ee-171">このメソッドは、 [oauth2](https://github.com/oauth-xx/oauth2) gem ( `omniauth-oauth2` gem の依存関係) を使用してトークンを更新し、セッションを更新します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-171">This method uses the [oauth2](https://github.com/oauth-xx/oauth2) gem (a dependency of the `omniauth-oauth2` gem) to refresh the tokens, and updates the session.</span></span>

1. <span data-ttu-id="a71ee-172">現在`access_token`のメソッドを次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-172">Replace the current `access_token` method with the following.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/application_controller.rb" id="AccessTokenSnippet":::

    <span data-ttu-id="a71ee-173">セッションからトークンを返すだけでなく、最初に有効期限が近づいているかどうかを確認します。</span><span class="sxs-lookup"><span data-stu-id="a71ee-173">Instead of just returning the token from the session, it will first check if it is close to expiration.</span></span> <span data-ttu-id="a71ee-174">その場合は、トークンを返す前に更新されます。</span><span class="sxs-lookup"><span data-stu-id="a71ee-174">If it is, then it will refresh before returning the token.</span></span>
