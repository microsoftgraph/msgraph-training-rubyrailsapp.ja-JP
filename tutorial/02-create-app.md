<!-- markdownlint-disable MD002 MD041 -->

この演習では [、Ruby on Rails を使用して](https://rubyonrails.org/) Web アプリをビルドします。

1. Rails がインストールされていない場合は、次のコマンドを使用してコマンド ライン インターフェイス (CLI) からインストールできます。

    ```Shell
    gem install rails -v 6.1.3.1
    ```

1. CLI を開き、ファイルを作成する権限を持つディレクトリに移動し、次のコマンドを実行して新しい Rails アプリを作成します。

    ```Shell
    rails new graph-tutorial
    ```

1. この新しいディレクトリに移動し、次のコマンドを入力してローカル Web サーバーを起動します。

    ```Shell
    rails server
    ```

1. ブラウザーを開き、`http://localhost:3000` に移動します。 すべてが機能している場合は、"Yay! You'on Rails! メッセージ。 そのメッセージが表示できない場合は、「Rails の開始ガイド [」を参照してください](http://guides.rubyonrails.org/)。

## <a name="install-gems"></a>gems のインストール

次に進む前に、後で使用する追加の gem をインストールします。

- サインインおよび OAuth トークン フローを処理する[omniauth-oauth2。](https://github.com/omniauth/omniauth-oauth2)
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection) に CSRF 保護を追加する場合に使用します。
- [microsoft Graph](https://github.com/jnunemaker/httparty)を呼び出Graph。
- [activerecord-session_store](https://github.com/rails/activerecord-session_store) セッションをデータベースに格納する場合に使用します。

1. **./Gemfile を開** き、次の行を追加します。

    :::code language="ruby" source="../demo/graph-tutorial/Gemfile" id="GemFileSnippet":::

1. CLI で、次のコマンドを実行します。

    ```Shell
    bundle install
    ```

1. CLI で、次のコマンドを実行して、セッションを格納するためのデータベースを構成します。

    ```Shell
    rails generate active_record:session_migration
    rake db:migrate
    ```

1. `session_store.rb` **./config/initializers** ディレクトリで呼び出される新しいファイルを作成し、次のコードを追加します。

    :::code language="ruby" source="../demo/graph-tutorial/config/initializers/session_store.rb" id="SessionStoreSnippet":::

## <a name="design-the-app"></a>アプリを設計する

このセクションでは、アプリの基本的な UI を作成します。

1. **./app/views/layouts/application.html.erb** を開き、その内容を次に置き換えます。

    :::code language="html" source="../demo/graph-tutorial/app/views/layouts/application.html.erb" id="LayoutSnippet":::

    このコードでは、 [単純なスタイル](http://getbootstrap.com/) 設定用のブートストラップと、いくつかの簡単なアイコン [の Fabric Core](https://developer.microsoft.com/fluentui#/get-started#fabric-core) を追加します。 また、ナビゲーション バーを持つグローバル レイアウトも定義します。

1. **./app/assets/stylesheets/application.css** を開き、ファイルの末尾に次の項目を追加します。

    :::code language="css" source="../demo/graph-tutorial/app/assets/stylesheets/application.css" id="CssSnippet":::

1. 次のコマンドを使用してホーム ページ コントローラーを生成します。

    ```Shell
    rails generate controller Home index
    ```

1. コントローラーの `index` アクションを `Home` アプリの既定のページとして構成します。 **./config/routes.rb を** 開き、その内容を次の内容に置き換える

    ```ruby
    Rails.application.routes.draw do
      get 'home/index'
      root 'home#index'

      # Add future routes here

    end
    ```

1. **./app/view/home/index.html.erb** を開き、その内容を次に置き換えます。

    :::code language="html" source="../demo/graph-tutorial/app/views/home/index.html.erb" id="HomeSnippet":::

1. **./app/assets/images** **ディレクトリにno-profile-photo.png** という名前の PNG ファイルを追加します。

1. 変更内容をすべて保存し、サーバーを再起動します。 これで、アプリは非常に異なって見える必要があります。

    ![デザインが変更されたホーム ページのスクリーンショット](./images/create-app-01.png)
