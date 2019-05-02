<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="35f78-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する Ruby on Rails web アプリを作成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="35f78-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="35f78-102">完成したチュートリアルをダウンロードするだけで済む場合は、2つの方法でダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="35f78-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="35f78-103">[Ruby クイックスタート](https://developer.microsoft.com/graph/quick-start?platform=option-ruby)をダウンロードして、作業コードを分単位で取得します。</span><span class="sxs-lookup"><span data-stu-id="35f78-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="35f78-104">[GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)をダウンロードするか、クローンを作成します。</span><span class="sxs-lookup"><span data-stu-id="35f78-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="35f78-105">前提条件</span><span class="sxs-lookup"><span data-stu-id="35f78-105">Prerequisites</span></span>

<span data-ttu-id="35f78-106">このチュートリアルを開始する前に、開発用のコンピューターに[Ruby](https://www.ruby-lang.org/en/downloads/)がインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="35f78-106">Before you start this tutorial, you should have [Ruby](https://www.ruby-lang.org/en/downloads/) installed on your development machine.</span></span> <span data-ttu-id="35f78-107">Ruby を使用していない場合は、「ダウンロードオプション」の前のリンクを参照してください。</span><span class="sxs-lookup"><span data-stu-id="35f78-107">If you do not have Ruby, visit the previous link for download options.</span></span>

> [!NOTE]
> <span data-ttu-id="35f78-108">このチュートリアルは、Ruby バージョン2.4.4 を使用して作成されています。</span><span class="sxs-lookup"><span data-stu-id="35f78-108">This tutorial was written with Ruby version 2.4.4.</span></span> <span data-ttu-id="35f78-109">このガイドの手順は、他のバージョンでは動作しますが、テストされていません。</span><span class="sxs-lookup"><span data-stu-id="35f78-109">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="35f78-110">フィードバック</span><span class="sxs-lookup"><span data-stu-id="35f78-110">Feedback</span></span>

<span data-ttu-id="35f78-111">このチュートリアルに関するフィードバックは、 [GitHub リポジトリ](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)に記入してください。</span><span class="sxs-lookup"><span data-stu-id="35f78-111">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>