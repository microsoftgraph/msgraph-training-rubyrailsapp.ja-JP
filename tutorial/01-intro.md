<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="4096e-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する Ruby on Rails Web アプリを作成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="4096e-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="4096e-102">完成したチュートリアルをダウンロードするだけの場合は、2 つの方法でダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="4096e-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="4096e-103">Ruby クイック [スタートをダウンロードして](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) 、作業コードを数分で取得します。</span><span class="sxs-lookup"><span data-stu-id="4096e-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="4096e-104">リポジトリをダウンロードまたは複製[GitHubします](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="4096e-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="4096e-105">前提条件</span><span class="sxs-lookup"><span data-stu-id="4096e-105">Prerequisites</span></span>

<span data-ttu-id="4096e-106">このチュートリアルを開始する前に、開発マシンに次のツールがインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="4096e-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="4096e-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="4096e-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="4096e-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="4096e-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="4096e-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="4096e-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="4096e-110">Yarn</span><span class="sxs-lookup"><span data-stu-id="4096e-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="4096e-111">また、Outlook.com 上のメールボックスを持つ個人用 Microsoft アカウント、または Microsoft の仕事用または学校用のアカウントを持っている必要があります。</span><span class="sxs-lookup"><span data-stu-id="4096e-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="4096e-112">Microsoft アカウントをお持ちでない場合は、無料アカウントを取得するためのオプションが 2 つご利用できます。</span><span class="sxs-lookup"><span data-stu-id="4096e-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="4096e-113">新しい [個人用 Microsoft アカウントにサインアップできます](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。</span><span class="sxs-lookup"><span data-stu-id="4096e-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="4096e-114">開発者プログラム[にサインアップしてOffice 365無料](https://developer.microsoft.com/office/dev-program)のサブスクリプションをOffice 365できます。</span><span class="sxs-lookup"><span data-stu-id="4096e-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="4096e-115">このチュートリアルは、必要なツールの次のバージョンで記述されています。</span><span class="sxs-lookup"><span data-stu-id="4096e-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="4096e-116">このガイドの手順は、他のバージョンでも動作しますが、テストされていない場合があります。</span><span class="sxs-lookup"><span data-stu-id="4096e-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="4096e-117">Ruby バージョン 3.0.1</span><span class="sxs-lookup"><span data-stu-id="4096e-117">Ruby version 3.0.1</span></span>
> - <span data-ttu-id="4096e-118">SQLite3 バージョン 3.35.5</span><span class="sxs-lookup"><span data-stu-id="4096e-118">SQLite3 version 3.35.5</span></span>
> - <span data-ttu-id="4096e-119">Node.jsバージョン 14.15.0</span><span class="sxs-lookup"><span data-stu-id="4096e-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="4096e-120">Yarn バージョン 1.22.0</span><span class="sxs-lookup"><span data-stu-id="4096e-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="4096e-121">フィードバック</span><span class="sxs-lookup"><span data-stu-id="4096e-121">Feedback</span></span>

<span data-ttu-id="4096e-122">このチュートリアルに関するフィードバックは、GitHub[してください](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="4096e-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
