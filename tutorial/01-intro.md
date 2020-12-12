<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a4de7-101">このチュートリアルでは、Microsoft Graph API を使用してユーザーの予定表情報を取得する Ruby on Rails Web アプリを作成する方法について説明します。</span><span class="sxs-lookup"><span data-stu-id="a4de7-101">This tutorial teaches you how to build a Ruby on Rails web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="a4de7-102">完成したチュートリアルをダウンロードするだけの場合は、2 つの方法でダウンロードできます。</span><span class="sxs-lookup"><span data-stu-id="a4de7-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="a4de7-103">Ruby クイック [スタートをダウンロードして、](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) 作業コードを数分で取得します。</span><span class="sxs-lookup"><span data-stu-id="a4de7-103">Download the [Ruby quick start](https://developer.microsoft.com/graph/quick-start?platform=option-ruby) to get working code in minutes.</span></span>
> - <span data-ttu-id="a4de7-104">GitHub リポジトリを [ダウンロードまたは複製します](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="a4de7-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a4de7-105">前提条件</span><span class="sxs-lookup"><span data-stu-id="a4de7-105">Prerequisites</span></span>

<span data-ttu-id="a4de7-106">このチュートリアルを開始する前に、開発用コンピューターに次のツールがインストールされている必要があります。</span><span class="sxs-lookup"><span data-stu-id="a4de7-106">Before you start this tutorial, you should have the following tools installed on your development machine.</span></span>

- [<span data-ttu-id="a4de7-107">Ruby</span><span class="sxs-lookup"><span data-stu-id="a4de7-107">Ruby</span></span>](https://www.ruby-lang.org/en/downloads/)
- [<span data-ttu-id="a4de7-108">SQLite3</span><span class="sxs-lookup"><span data-stu-id="a4de7-108">SQLite3</span></span>](https://sqlite.org/index.html)
- [<span data-ttu-id="a4de7-109">Node.js</span><span class="sxs-lookup"><span data-stu-id="a4de7-109">Node.js</span></span>](https://nodejs.org/en/)
- [<span data-ttu-id="a4de7-110">長さ</span><span class="sxs-lookup"><span data-stu-id="a4de7-110">Yarn</span></span>](https://yarnpkg.com/)

<span data-ttu-id="a4de7-111">また、メールボックスを持つ個人用の Microsoft アカウントが Outlook.com Microsoft の仕事用アカウントまたは学校アカウントである必要があります。</span><span class="sxs-lookup"><span data-stu-id="a4de7-111">You should also have either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span> <span data-ttu-id="a4de7-112">Microsoft アカウントをお持ちない場合は、無料アカウントを取得するためのオプションが 2 つ提供されています。</span><span class="sxs-lookup"><span data-stu-id="a4de7-112">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="a4de7-113">新しい [個人用 Microsoft アカウントにサインアップできます](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)。</span><span class="sxs-lookup"><span data-stu-id="a4de7-113">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="a4de7-114">Office [365 開発者プログラムにサインアップして、365](https://developer.microsoft.com/office/dev-program) サブスクリプションを無料Office取得できます。</span><span class="sxs-lookup"><span data-stu-id="a4de7-114">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

> [!NOTE]
> <span data-ttu-id="a4de7-115">このチュートリアルは、必要なツールの次のバージョンで記述されています。</span><span class="sxs-lookup"><span data-stu-id="a4de7-115">This tutorial was written with the following versions of the required tools.</span></span> <span data-ttu-id="a4de7-116">このガイドの手順は他のバージョンでも動作する可能性がありますが、テストは行ってはいではありません。</span><span class="sxs-lookup"><span data-stu-id="a4de7-116">The steps in this guide may work with other versions, but that has not been tested.</span></span>
>
> - <span data-ttu-id="a4de7-117">Ruby バージョン 2.6.6。</span><span class="sxs-lookup"><span data-stu-id="a4de7-117">Ruby version 2.6.6.</span></span>
> - <span data-ttu-id="a4de7-118">SQLite3 バージョン 3.31.1</span><span class="sxs-lookup"><span data-stu-id="a4de7-118">SQLite3 version 3.31.1</span></span>
> - <span data-ttu-id="a4de7-119">Node.js バージョン 14.15.0</span><span class="sxs-lookup"><span data-stu-id="a4de7-119">Node.js version 14.15.0</span></span>
> - <span data-ttu-id="a4de7-120">本バージョン 1.22.0</span><span class="sxs-lookup"><span data-stu-id="a4de7-120">Yarn version 1.22.0</span></span>

## <a name="feedback"></a><span data-ttu-id="a4de7-121">フィードバック</span><span class="sxs-lookup"><span data-stu-id="a4de7-121">Feedback</span></span>

<span data-ttu-id="a4de7-122">このチュートリアルに関するフィードバックは [、GitHub リポジトリで提供してください](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp)。</span><span class="sxs-lookup"><span data-stu-id="a4de7-122">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-rubyrailsapp).</span></span>
