<!-- markdownlint-disable MD002 MD041 -->

この演習では、 [Ruby On Rails](https://rubyonrails.org/)を使用して web アプリを構築します。

1. Rails がまだインストールされていない場合は、コマンドラインインターフェイス (CLI) から次のコマンドを使用してインストールできます。

    ```Shell
    gem install rails -v 6.0.2.2
    ```

1. CLI を開き、ファイルを作成する権限があるディレクトリに移動し、次のコマンドを実行して新しい Rails アプリを作成します。

    ```Shell
    rails new graph-tutorial
    ```

1. この新しいディレクトリに移動し、次のコマンドを入力してローカル web サーバーを開始します。

    ```Shell
    rails server
    ```

1. ブラウザーを開き、`http://localhost:3000` に移動します。 すべてが動作している場合は、"Yay!" と表示されます。 レールを使用しています。 メッセージ。 このメッセージが表示されない場合は、 [Rails の入門ガイド](http://guides.rubyonrails.org/)を確認してください。

## <a name="install-gems"></a>Gem をインストールする

に進む前に、後で使用する gem をいくつかインストールします。

- [omniauth-](https://github.com/omniauth/omniauth-oauth2)サインインおよび OAuth トークンフローを処理するための oauth2。
- [omniauth-](https://github.com/cookpad/omniauth-rails_csrf_protection) csrf 保護を omniauth に追加するために rails_csrf_protection します。
- Microsoft Graph に電話をかけるための[httparty](https://github.com/jnunemaker/httparty) 。
- の場合は、データベースにセッションを格納するための[session_store](https://github.com/rails/activerecord-session_store)ます。

1. **/Gemfile**を開き、次の行を追加します。

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. CLI で、次のコマンドを実行します。

    ```Shell
    bundle install
    ```

1. CLI で、次のコマンドを実行して、セッションを格納するようにデータベースを構成します。

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. `session_store.rb` **./Config/イニシャライザー**ディレクトリに新しいファイルを作成し、次のコードを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>アプリを設計する

このセクションでは、アプリの基本的な UI を作成します。

1. **/App/views/layouts/application.html.erb**を開き、その内容を次のように置き換えます。

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    このコードはシンプルなスタイルのために [Bootstrap](http://getbootstrap.com/) を追加し、シンプルなアイコンのために [Font Awesome](https://fontawesome.com/) を追加します。 また、ナビゲーションバーのあるグローバルレイアウトを定義します。

1. **/App/assets/stylesheets/application.css**を開き、ファイルの末尾に次のものを追加します。

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. 次のコマンドを使用して、ホームページコントローラーを生成します。

    ```Shell
    rails generate controller Home index
    ```

1. `Home`コントローラーの`index`アクションをアプリの既定のページとして構成します。 /Config/routes.rb を開き、そのコンテンツを次のように置き換え**ます。**

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. **/App/view/home/index.html.erb**を開き、その内容を次のように置き換えます。

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. 変更内容をすべて保存し、サーバーを再起動します。 この時点で、アプリの外観は大きく異なります。

    ![デザインが変更されたホーム ページのスクリーンショット](./images/create-app-01.png)
