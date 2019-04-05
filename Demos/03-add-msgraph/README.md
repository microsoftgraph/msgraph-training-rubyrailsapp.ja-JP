# <a name="how-to-run-the-completed-project"></a>完了したプロジェクトを実行する方法

## <a name="prerequisites"></a>前提条件

このフォルダーで完了したプロジェクトを実行するには、次のものが必要です。

- 開発用コンピューターにインストールされている[Ruby](https://www.ruby-lang.org/en/downloads/) 。 Ruby を使用していない場合は、「ダウンロードオプション」の前のリンクを参照してください。 (**注:** このチュートリアルは、Ruby バージョン2.4.4 を使用して作成されています。 このガイドの手順は、他のバージョンでは動作しますが、テストされていません。)
- Outlook.com 上のメールボックスを持つ個人の Microsoft アカウント、または microsoft 職場または学校のアカウントのいずれか。

Microsoft アカウントを持っていない場合は、無料のアカウントを取得するためのオプションがいくつかあります。

- [新しい個人用 Microsoft アカウントにサインアップ](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)することができます。
- [office 365 開発者プログラムにサインアップ](https://developer.microsoft.com/office/dev-program)して、無料の office 365 サブスクリプションを取得することができます。

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a>web アプリケーションを Azure Active Directory 管理センターに登録する

1. ブラウザーを開き、 [Azure Active Directory 管理センター](https://aad.portal.azure.com)に移動します。 **個人アカウント**(別名: Microsoft アカウント) または**職場または学校のアカウント**を使用してログインします。

1. 左側のナビゲーションで [ **Azure Active Directory** ] を選択し、[**管理**] で [**アプリの登録 (プレビュー)** ] を選択します。

    ![アプリの登録のスクリーンショット ](/tutorial/images/aad-portal-app-registrations.png)

1. [**新しい登録**] を選択します。 [**アプリケーションの登録**] ページで、次のように値を設定します。

    - **名前**をに`Ruby Graph Tutorial`設定します。
    - **任意の組織ディレクトリおよび個人の Microsoft アカウントのアカウント**に、**サポートされているアカウントの種類**を設定します。
    - [**リダイレクト URI**] で、最初のドロップダウンを`Web`に設定し、値`http://localhost:3000/auth/microsoft_graph_auth/callback`をに設定します。

    ![[アプリケーションの登録] ページのスクリーンショット](/tutorial/images/aad-register-an-app.png)

1. [**登録**] を選択します。 **Ruby Graph のチュートリアル**ページで、**アプリケーション (クライアント) ID**の値をコピーして保存します。次の手順で必要になります。

    ![新しいアプリの登録のアプリケーション ID のスクリーンショット](/tutorial/images/aad-application-id.png)

1. [**証明書の &** ] の [**管理**] で選択します。 [**新しいクライアントシークレット**] ボタンを選択します。 [**説明**] に値を入力して、[**有効期限**] のオプションの1つを選択し、[**追加**] を選択します。

    ![[クライアントシークレットの追加] ダイアログのスクリーンショット](/tutorial/images/aad-new-client-secret.png)

1. このページを終了する前に、クライアントシークレット値をコピーします。 次の手順で必要になります。

    > [!IMPORTANT]
    > このクライアントシークレットが再度表示されることはありません。そのため、ここでコピーしてください。

    ![新しく追加されたクライアントシークレットのスクリーンショット](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a>サンプルを構成する

1. ファイルの`./config/oauth_environment_variables.rb.example`名前を`oauth_environment_variables.rb`に変更します。
1. `oauth_environment_variables.rb`ファイルを編集し、次のように変更します。
    1. を`YOUR_APP_ID_HERE`アプリ登録ポータルで取得した**アプリケーション Id**に置き換えます。
    1. を`YOUR APP PASSWORD HERE`アプリ登録ポータルから取得したパスワードに置き換えます。
1. コマンドラインインターフェイス (CLI) で、このディレクトリに移動し、次のコマンドを実行して要件をインストールします。

    ```Shell
    bundle install
    ```

1. CLI で、次のコマンドを実行して、アプリのデータベースを初期化します。

    ```Shell
    rake db:migrate
    ```

## <a name="run-the-sample"></a>サンプルを実行する

1. CLI で次のコマンドを実行して、アプリケーションを起動します。

    ```Shell
    rails server
    ```

1. Web ブラウザーを開き、`http://localhost:3000` を参照します。