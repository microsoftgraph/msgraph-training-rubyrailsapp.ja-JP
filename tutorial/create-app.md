<!-- markdownlint-disable MD002 MD041 -->

この演習では、 [Ruby On Rails](https://rubyonrails.org/)を使用して web アプリを構築します。 Rails がまだインストールされていない場合は、コマンドラインインターフェイス (CLI) から次のコマンドを使用してインストールできます。

```Shell
gem install rails
```

CLI を開き、ファイルを作成する権限があるディレクトリに移動し、次のコマンドを実行して新しい Rails アプリを作成します。

```Shell
rails new graph-tutorial
```

Rails アプリと呼ばれる`graph-tutorial`新しいディレクトリを作成し、スキャフォールディングを作成します。 この新しいディレクトリに移動し、次のコマンドを入力してローカル web サーバーを開始します。

```Shell
rails server
```

ブラウザーを開き、`http://localhost:3000` に移動します。 すべてが動作している場合は、"Yay!" と表示されます。 レールを使用しています。 メッセージ。 このメッセージが表示されない場合は、 [Rails の入門ガイド](http://guides.rubyonrails.org/)を確認してください。

に進む前に、後で使用する gem をいくつかインストールします。

- [omniauth-](https://github.com/omniauth/omniauth-oauth2)サインインおよび OAuth トークンフローを処理するための oauth2。
- Microsoft Graph に電話をかけるための[httparty](https://github.com/jnunemaker/httparty) 。
- 電子メールの HTML 本文を処理するための[noて giri](https://github.com/sparklemotion/nokogiri) 。
- [session_store](https://github.com/rails/activerecord-session_store)は、データベースにセッションを格納するためのものです。

CLI で次のコマンドを実行します。

```Shell
bundle add omniauth-oauth2
bundle add httparty
bundle add nokogiri
bundle add activerecord-session_store
rails generate active_record:session_migration
```

最後のコマンドでは、次のような出力が生成されます。

```Shell
create  db/migrate/20180618172216_add_sessions_table.rb
```

作成されたファイルを開き、次の行を探します。

```ruby
class AddSessionsTable < ActiveRecord::Migration
```

その行を次のように変更します。

```ruby
class AddSessionsTable < ActiveRecord::Migration[5.2]
```

> [!NOTE]
> これは、Rails 5.2 .x を使用していることを前提としています。 別のバージョンを使用している場合`5.2`は、をご使用のバージョンに置き換えます。

ファイルを保存し、次のコマンドを実行します。

```Shell
rake db:migrate
```

最後に、新しいセッションストアを使用するように Rails を構成します。 ディレクトリ内にという`session_store.rb`新しいファイルを作成し、次のコードを追加します。 `./config/initializers`

```ruby
Rails.application.config.session_store :active_record_store, key: '_graph_app_session'
```

## <a name="design-the-app"></a>アプリを設計する

最初に、アプリのグローバルレイアウトを更新します。 を`./app/views/layouts/application.html.erb`開き、その内容を次のように置き換えます。

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

このコードでは、単純なスタイル設定[](https://fontawesome.com/)のために[ブートストラップ](http://getbootstrap.com/)が追加されています。 また、ナビゲーションバーのあるグローバルレイアウトを定義します。

を開き`./app/assets/stylesheets/application.css` 、ファイルの末尾に次のように追加します。

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

次に、既定のページを新しいページに置き換えます。 次のコマンドを使用して、ホームページコントローラーを生成します。

```Shell
rails generate controller Home index
```

その後、 `index`アプリの既定`Home`のページとして、コントローラーのアクションを構成します。 を`./config/routes.rb`開いて、コンテンツを次のように置き換えます。

```ruby
Rails.application.routes.draw do
  get 'home/index'
  root 'home#index'

  # Add future routes here

end
```

`./app/view/home/index.html.erb`ファイルを開き、その内容を次のように置き換えます。

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

すべての変更を保存し、サーバーを再起動します。 この時点で、アプリの外観は大きく異なります。

![再設計されたホームページのスクリーンショット](./images/create-app-01.png)