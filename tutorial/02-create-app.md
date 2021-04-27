<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="30c40-101">この演習では [、Ruby on Rails を使用して](https://rubyonrails.org/) Web アプリをビルドします。</span><span class="sxs-lookup"><span data-stu-id="30c40-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span>

1. <span data-ttu-id="30c40-102">Rails がインストールされていない場合は、次のコマンドを使用してコマンド ライン インターフェイス (CLI) からインストールできます。</span><span class="sxs-lookup"><span data-stu-id="30c40-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. <span data-ttu-id="30c40-103">CLI を開き、ファイルを作成する権限を持つディレクトリに移動し、次のコマンドを実行して新しい Rails アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="30c40-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

    ```Shell
    rails new graph-tutorial
    ```

1. <span data-ttu-id="30c40-104">この新しいディレクトリに移動し、次のコマンドを入力してローカル Web サーバーを起動します。</span><span class="sxs-lookup"><span data-stu-id="30c40-104">Navigate to this new directory and enter the following command to start a local web server.</span></span>

    ```Shell
    rails server
    ```

1. <span data-ttu-id="30c40-105">ブラウザーを開き、`http://localhost:3000` に移動します。</span><span class="sxs-lookup"><span data-stu-id="30c40-105">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="30c40-106">すべてが機能している場合は、"Yay!</span><span class="sxs-lookup"><span data-stu-id="30c40-106">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="30c40-107">You'on Rails!</span><span class="sxs-lookup"><span data-stu-id="30c40-107">You're on Rails!"</span></span> <span data-ttu-id="30c40-108">メッセージ。</span><span class="sxs-lookup"><span data-stu-id="30c40-108">message.</span></span> <span data-ttu-id="30c40-109">そのメッセージが表示できない場合は、「Rails の開始ガイド [」を参照してください](http://guides.rubyonrails.org/)。</span><span class="sxs-lookup"><span data-stu-id="30c40-109">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

## <a name="install-gems"></a><span data-ttu-id="30c40-110">gems のインストール</span><span class="sxs-lookup"><span data-stu-id="30c40-110">Install gems</span></span>

<span data-ttu-id="30c40-111">次に進む前に、後で使用する追加の gem をインストールします。</span><span class="sxs-lookup"><span data-stu-id="30c40-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="30c40-112">サインインおよび OAuth トークン フローを処理する[omniauth-oauth2。](https://github.com/omniauth/omniauth-oauth2)</span><span class="sxs-lookup"><span data-stu-id="30c40-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="30c40-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) に CSRF 保護を追加する場合に使用します。</span><span class="sxs-lookup"><span data-stu-id="30c40-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="30c40-114">[microsoft Graph](https://github.com/jnunemaker/httparty)を呼び出Graph。</span><span class="sxs-lookup"><span data-stu-id="30c40-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="30c40-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) セッションをデータベースに格納する場合に使用します。</span><span class="sxs-lookup"><span data-stu-id="30c40-115">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

1. <span data-ttu-id="30c40-116">**./Gemfile を開** き、次の行を追加します。</span><span class="sxs-lookup"><span data-stu-id="30c40-116">Open **./Gemfile** and add the following lines.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. <span data-ttu-id="30c40-117">CLI で、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="30c40-117">In your CLI, run the following command.</span></span>

    ```Shell
    bundle install
    ```

1. <span data-ttu-id="30c40-118">CLI で、次のコマンドを実行して、セッションを格納するためのデータベースを構成します。</span><span class="sxs-lookup"><span data-stu-id="30c40-118">In your CLI, run the following commands to configure the database for storing sessions.</span></span>

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. <span data-ttu-id="30c40-119">`session_store.rb` **./config/initializers** ディレクトリで呼び出される新しいファイルを作成し、次のコードを追加します。</span><span class="sxs-lookup"><span data-stu-id="30c40-119">Create a new file called `session_store.rb` in the **./config/initializers** directory, and add the following code.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a><span data-ttu-id="30c40-120">アプリを設計する</span><span class="sxs-lookup"><span data-stu-id="30c40-120">Design the app</span></span>

<span data-ttu-id="30c40-121">このセクションでは、アプリの基本的な UI を作成します。</span><span class="sxs-lookup"><span data-stu-id="30c40-121">In this section you'll create the basic UI for the app.</span></span>

1. <span data-ttu-id="30c40-122">**./app/views/layouts/application.html.erb** を開き、その内容を次に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="30c40-122">Open **./app/views/layouts/application.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    <span data-ttu-id="30c40-123">このコードでは、 [単純なスタイル](http://getbootstrap.com/) 設定用のブートストラップと、いくつかの簡単なアイコン [の Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) を追加します。</span><span class="sxs-lookup"><span data-stu-id="30c40-123">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) for some simple icons.</span></span> <span data-ttu-id="30c40-124">また、ナビゲーション バーを持つグローバル レイアウトも定義します。</span><span class="sxs-lookup"><span data-stu-id="30c40-124">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="30c40-125">**./app/assets/stylesheets/application.css** を開き、ファイルの末尾に次の項目を追加します。</span><span class="sxs-lookup"><span data-stu-id="30c40-125">Open **./app/assets/stylesheets/application.css** and add the following to the end of the file.</span></span>

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. <span data-ttu-id="30c40-126">次のコマンドを使用してホーム ページ コントローラーを生成します。</span><span class="sxs-lookup"><span data-stu-id="30c40-126">Generate a home page controller with the following command.</span></span>

    ```Shell
    rails generate controller Home index
    ```

1. <span data-ttu-id="30c40-127">コントローラーの `index` アクションを `Home` アプリの既定のページとして構成します。</span><span class="sxs-lookup"><span data-stu-id="30c40-127">Configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="30c40-128">**./config/routes.rb を** 開き、その内容を次の内容に置き換える</span><span class="sxs-lookup"><span data-stu-id="30c40-128">Open **./config/routes.rb** and replace its contents with the following</span></span>

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. <span data-ttu-id="30c40-129">**./app/view/home/index.html.erb** を開き、その内容を次に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="30c40-129">Open **./app/view/home/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. <span data-ttu-id="30c40-130">**./app/assets/images** **ディレクトリにno-profile-photo.png** という名前の PNG ファイルを追加します。</span><span class="sxs-lookup"><span data-stu-id="30c40-130">Add a PNG file named **no-profile-photo.png** in the **./app/assets/images** directory.</span></span>

1. <span data-ttu-id="30c40-131">変更内容をすべて保存し、サーバーを再起動します。</span><span class="sxs-lookup"><span data-stu-id="30c40-131">Save all of your changes and restart the server.</span></span> <span data-ttu-id="30c40-132">これで、アプリは非常に異なって見える必要があります。</span><span class="sxs-lookup"><span data-stu-id="30c40-132">Now, the app should look very different.</span></span>

    ![デザインが変更されたホーム ページのスクリーンショット](./images/create-app-01.png)
