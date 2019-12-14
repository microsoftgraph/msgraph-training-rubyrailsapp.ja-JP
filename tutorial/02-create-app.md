<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="ca7cd-101">この演習では、 [Ruby On Rails](https://rubyonrails.org/)を使用して web アプリを構築します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-101">In this exercise you will use [Ruby on Rails](https://rubyonrails.org/) to build a web app.</span></span> <span data-ttu-id="ca7cd-102">Rails がまだインストールされていない場合は、コマンドラインインターフェイス (CLI) から次のコマンドを使用してインストールできます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-102">If you don't already have Rails installed, you can install it from your command-line interface (CLI) with the following command.</span></span>

```Shell
gem install rails
```

<span data-ttu-id="ca7cd-103">CLI を開き、ファイルを作成する権限があるディレクトリに移動し、次のコマンドを実行して新しい Rails アプリを作成します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-103">Open your CLI, navigate to a directory where you have rights to create files, and run the following command to create a new Rails app.</span></span>

```Shell
rails new graph-tutorial
```

<span data-ttu-id="ca7cd-104">Rails アプリと呼ばれる`graph-tutorial`新しいディレクトリを作成し、スキャフォールディングを作成します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-104">Rails creates a new directory called `graph-tutorial` and scaffolds a Rails app.</span></span> <span data-ttu-id="ca7cd-105">この新しいディレクトリに移動し、次のコマンドを入力してローカル web サーバーを開始します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-105">Navigate to this new directory and enter the following command to start a local web server.</span></span>

```Shell
rails server
```

<span data-ttu-id="ca7cd-106">ブラウザーを開き、`http://localhost:3000` に移動します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-106">Open your browser and navigate to `http://localhost:3000`.</span></span> <span data-ttu-id="ca7cd-107">すべてが動作している場合は、"Yay!" と表示されます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-107">If everything is working, you will see a "Yay!</span></span> <span data-ttu-id="ca7cd-108">レールを使用しています。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-108">You're on Rails!"</span></span> <span data-ttu-id="ca7cd-109">メッセージ。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-109">message.</span></span> <span data-ttu-id="ca7cd-110">このメッセージが表示されない場合は、 [Rails の入門ガイド](http://guides.rubyonrails.org/)を確認してください。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-110">If you don't see that message, check the [Rails getting started guide](http://guides.rubyonrails.org/).</span></span>

<span data-ttu-id="ca7cd-111">に進む前に、後で使用する gem をいくつかインストールします。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-111">Before moving on, install some additional gems that you will use later:</span></span>

- <span data-ttu-id="ca7cd-112">[omniauth-](https://github.com/omniauth/omniauth-oauth2)サインインおよび OAuth トークンフローを処理するための oauth2。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-112">[omniauth-oauth2](https://github.com/omniauth/omniauth-oauth2) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="ca7cd-113">[omniauth-](https://github.com/cookpad/omniauth-rails_csrf_protection) csrf 保護を omniauth に追加するために rails_csrf_protection します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-113">[omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) for adding CSRF protection to OmniAuth.</span></span>
- <span data-ttu-id="ca7cd-114">Microsoft Graph に電話をかけるための[httparty](https://github.com/jnunemaker/httparty) 。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-114">[httparty](https://github.com/jnunemaker/httparty) for making calls to Microsoft Graph.</span></span>
- <span data-ttu-id="ca7cd-115">電子メールの HTML 本文を処理するための[noて giri](https://github.com/sparklemotion/nokogiri) 。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-115">[nokogiri](https://github.com/sparklemotion/nokogiri) to process HTML bodies of email.</span></span>
- <span data-ttu-id="ca7cd-116">の場合は、データベースにセッションを格納するための[session_store](https://github.com/rails/activerecord-session_store)ます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-116">[activerecord-session_store](https://github.com/rails/activerecord-session_store) for storing sessions in the database.</span></span>

<span data-ttu-id="ca7cd-117">CLI で次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-117">Run the following commands in your CLI.</span></span>

```Shell
bundle add omniauth-oauth2
bundle add omniauth-rails_csrf_protection
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

<span data-ttu-id="ca7cd-118">最後のコマンドでは、次のような出力が生成されます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-118">The last command generates output like the following:</span></span>

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

<span data-ttu-id="ca7cd-119">作成されたファイルを開き、次の行を探します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-119">Open the file that was created and locate the following line.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

<span data-ttu-id="ca7cd-120">その行を次のように変更します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-120">Change that line to the following.</span></span>

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> <span data-ttu-id="ca7cd-121">これは、Rails 5.2 .x を使用していることを前提としています。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-121">This assumes that you are using Rails 5.2.x.</span></span> <span data-ttu-id="ca7cd-122">別のバージョンを使用している場合`5.2`は、をご使用のバージョンに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-122">If you are using a different version, replace `5.2` with your version.</span></span>

<span data-ttu-id="ca7cd-123">ファイルを保存し、次のコマンドを実行します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-123">Save the file and run the following command.</span></span>

```Shell
rake db:migrate
```

<span data-ttu-id="ca7cd-124">最後に、新しいセッションストアを使用するように Rails を構成します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-124">Finally, configure Rails to use the new session store.</span></span> <span data-ttu-id="ca7cd-125">ディレクトリ内にという`session_store.rb`新しいファイルを作成し、次のコードを追加します。 `./config/initializers`</span><span class="sxs-lookup"><span data-stu-id="ca7cd-125">Create a new file called `session_store.rb` in the `./config/initializers` directory, and add the following code.</span></span>

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a><span data-ttu-id="ca7cd-126">アプリを設計する</span><span class="sxs-lookup"><span data-stu-id="ca7cd-126">Design the app</span></span>

<span data-ttu-id="ca7cd-127">最初に、アプリのグローバルレイアウトを更新します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-127">Start by updating the global layout for the app.</span></span> <span data-ttu-id="ca7cd-128">を`./app/views/layouts/application.html.erb`開き、その内容を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-128">Open `./app/views/layouts/application.html.erb` and replace its contents with the following.</span></span>

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Ruby Graph Tutorial</title>
    <%= csrf_meta_tags %>
    <%= csp_meta_tag %>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
      integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
      integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
      integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
      integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
    <%= stylesheet_link_tag    'application', media: 'all', 'data-turbolinks-track': 'reload' %>
    <%= javascript_include_tag 'application', 'data-turbolinks-track': 'reload' %>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <%= link_to "Ruby Graph Tutorial", root_path, class: "navbar-brand" %>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
          aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <%= link_to "Home", root_path, class: "nav-link#{' active' if controller.controller_name == 'home'}" %>
            </li>
            <% if @user_name %>
              <li class="nav-item" data-turbolinks="false">
                <a class="nav-link" href="#">Calendar</a>
              </li>
            <% end %>
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            <% if @user_name %>
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button" aria-haspopup="true" aria-expanded="false">
                  <% if @user_avatar %>
                    <img src=<%= @user_avatar %> class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  <% else %>
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  <% end %>
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0"><%= @user_name %></h5>
                  <p class="dropdown-item-text text-muted mb-0"><%= @user_email %></p>
                  <div class="dropdown-divider"></div>
                  <a href="#" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            <% else %>
              <li class="nav-item">
                <a href="#" class="nav-link">Sign In</a>
              </li>
            <% end %>
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      <% if @errors %>
        <% @errors.each do |error| %>
          <div class="alert alert-danger" role="alert">
            <p class="mb-3"><%= error[:message] %></p>
            <%if error[:debug] %>
              <pre class="alert-pre border bg-light p-2"><code><%= error[:debug] %></code></pre>
            <% end %>
          </div>
        <% end %>
      <% end %>
      <%= yield %>
    </main>
  </body>
</html>
```

<span data-ttu-id="ca7cd-129">このコードでは、単純なスタイル設定のために[ブートストラップ](http://getbootstrap.com/)が追加さ[れてい](https://fontawesome.com/)ます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-129">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="ca7cd-130">また、ナビゲーションバーのあるグローバルレイアウトを定義します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-130">It also defines a global layout with a nav bar.</span></span>

<span data-ttu-id="ca7cd-131">を開き`./app/assets/stylesheets/application.css` 、ファイルの末尾に次のように追加します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-131">Now open `./app/assets/stylesheets/application.css` and add the following to the end of the file.</span></span>

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

<span data-ttu-id="ca7cd-132">次に、既定のページを新しいページに置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-132">Now replace the default page with a new one.</span></span> <span data-ttu-id="ca7cd-133">次のコマンドを使用して、ホームページコントローラーを生成します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-133">Generate a home page controller with the following command.</span></span>

```Shell
rails generate controller Home index
```

<span data-ttu-id="ca7cd-134">その後、 `index`アプリの既定`Home`のページとして、コントローラーのアクションを構成します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-134">Then configure the `index` action on the `Home` controller as the default page for the app.</span></span> <span data-ttu-id="ca7cd-135">を`./config/routes.rb`開いて、コンテンツを次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-135">Open `./config/routes.rb` and replace the contents with the following</span></span>

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

<span data-ttu-id="ca7cd-136">`./app/view/home/index.html.erb`ファイルを開き、その内容を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-136">Now open the `./app/view/home/index.html.erb` file and replace its contents with the following.</span></span>

```html
<div class="jumbotron">
  <h1>Ruby Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from Ruby</p>
  <% if @user_name %>
    <h4>Welcome <%= @user_name %>!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  <% else %>
    <a href="#" class="btn btn-primary btn-large">Click here to sign in</a>
  <% end %>
</div>
```

<span data-ttu-id="ca7cd-137">すべての変更を保存し、サーバーを再起動します。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-137">Save all of your changes and restart the server.</span></span> <span data-ttu-id="ca7cd-138">この時点で、アプリの外観は大きく異なります。</span><span class="sxs-lookup"><span data-stu-id="ca7cd-138">Now, the app should look very different.</span></span>

![再設計されたホームページのスクリーンショット](./images/create-app-01.png)
