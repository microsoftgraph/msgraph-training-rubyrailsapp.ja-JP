<!-- markdownlint-disable MD002 MD041 -->

このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する Ruby on Rails Web アプリを作成する方法について説明します。

> [!TIP]
> 完成したチュートリアルをダウンロードするだけの場合は、2 つの方法でダウンロードできます。
>
> - Ruby クイック [スタートをダウンロードして](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) 、作業コードを数分で取得します。
> - リポジトリをダウンロードまたは複製[GitHubします](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。

## <a name="prerequisites"></a>前提条件

このチュートリアルを開始する前に、開発マシンに次のツールがインストールされている必要があります。

- [Ruby](https://www.ruby-lang.org/en/downloads/)
- [SQLite3](https://sqlite.org/index.html)
- [Node.js](https://nodejs.org/en/)
- [Yarn](https://yarnpkg.com/)

また、Outlook.com 上のメールボックスを持つ個人用 Microsoft アカウント、または Microsoft の仕事用または学校用のアカウントを持っている必要があります。 Microsoft アカウントをお持ちでない場合は、無料アカウントを取得するためのオプションが 2 つご利用できます。

- 新しい [個人用 Microsoft アカウントにサインアップできます](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。
- 開発者プログラム[にサインアップしてOffice 365無料](https://developer.microsoft.com/office/dev-program)のサブスクリプションをOffice 365できます。

> [!NOTE]
> このチュートリアルは、必要なツールの次のバージョンで記述されています。 このガイドの手順は、他のバージョンでも動作しますが、テストされていない場合があります。
>
> - Ruby バージョン 3.0.1
> - SQLite3 バージョン 3.35.5
> - Node.jsバージョン 14.15.0
> - Yarn バージョン 1.22.0

## <a name="feedback"></a>フィードバック

このチュートリアルに関するフィードバックは、GitHub[してください](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。
