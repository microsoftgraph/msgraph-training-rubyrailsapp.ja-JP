# <a name="contributing-to-microsoft-graph-training-repositories"></a><span data-ttu-id="83e66-101">Microsoft Graph トレーニング リポジトリへの貢献</span><span class="sxs-lookup"><span data-stu-id="83e66-101">Contributing to Microsoft Graph training repositories</span></span>

<span data-ttu-id="83e66-102">このプロジェクトに貢献してありがとうございます。</span><span class="sxs-lookup"><span data-stu-id="83e66-102">Thank you for contributing to this project!</span></span> <span data-ttu-id="83e66-103">プルリクエストを提出する前に、次の点を考慮してください。</span><span class="sxs-lookup"><span data-stu-id="83e66-103">Before submitting your pull request, be sure to consider the following.</span></span>

## <a name="overview"></a><span data-ttu-id="83e66-104">概要</span><span class="sxs-lookup"><span data-stu-id="83e66-104">Overview</span></span>

<span data-ttu-id="83e66-105">このリポジトリ内のコードは、次の 3 つの目的に使用されます。</span><span class="sxs-lookup"><span data-stu-id="83e66-105">The code in this repository serves three purposes:</span></span>

- <span data-ttu-id="83e66-106">チュートリアル フォルダー内の Markdown [ファイルは](/tutorial) 、Microsoft Graph のチュートリアル ページでチュートリアル [として公開](https://docs.microsoft.com/graph/tutorials) されます。</span><span class="sxs-lookup"><span data-stu-id="83e66-106">The Markdown files in the [tutorial](/tutorial) folder are published as a tutorial on the [Microsoft Graph tutorials](https://docs.microsoft.com/graph/tutorials) page.</span></span>
- <span data-ttu-id="83e66-107">デモ フォルダー内の [サンプル プロジェクト](/demo) は [、Microsoft Graph](https://developer.microsoft.com/graph/quick-start)クイック スタートのソースです。\* *\** _</span><span class="sxs-lookup"><span data-stu-id="83e66-107">The sample project in the [demo](/demo) folder is the source for a [Microsoft Graph quick start](https://developer.microsoft.com/graph/quick-start).\**\** _</span></span>
- <span data-ttu-id="83e66-108">デモ フォルダー内のサンプル プロジェクトは GitHub から直接ダウンロードすることもできます。簡単な構成の後は、この時点で実行する必要があります。</span><span class="sxs-lookup"><span data-stu-id="83e66-108">The sample project in the demo folder is also downloadable directly from GitHub and should run as-is after some simple configuration.</span></span>

> <span data-ttu-id="83e66-109">_*\**_ すべてのトレーニング リポジトリがクイック スタートとして利用できる (まだ) ではありません。</span><span class="sxs-lookup"><span data-stu-id="83e66-109">_*\**_ Not all training repositories are available as quick starts (yet).</span></span>

<span data-ttu-id="83e66-110">これは、ある場所での変更が同期を維持するために_may\* で変更が必要なので、念頭に置いておく必要があります。可能であれば、Markdown ファイルは (カスタム構文を使用して) ソース コード ファイルを直接参照し、ソース内のコードを更新すると Markdown のコードが自動的に `:::code` 更新されます。</span><span class="sxs-lookup"><span data-stu-id="83e66-110">This is important to keep in mind, because changes in one place _may\* require changes in another, to keep things in sync. Whereever possible, the Markdown files refer to the source code files directly (using a custom `:::code` syntax), so that updating code in source will automatically update the code in Markdown.</span></span>

## <a name="updating-code"></a><span data-ttu-id="83e66-111">コードの更新</span><span class="sxs-lookup"><span data-stu-id="83e66-111">Updating code</span></span>

<span data-ttu-id="83e66-112">`:::code`Markdown で使用される構文は、ソース コード ファイル内の特定のコメントによって異なります。</span><span class="sxs-lookup"><span data-stu-id="83e66-112">The `:::code` syntax used in Markdown depends on specific comments in the source code file.</span></span> <span data-ttu-id="83e66-113">これらのコメントは次のようになります。</span><span class="sxs-lookup"><span data-stu-id="83e66-113">These comments look like the following:</span></span>

```csharp
// <MySnippet>
Console.WriteLine("Hello World!");
// </MySnippet>
```

<span data-ttu-id="83e66-114">これらの "マーカー" コメントの間でコードを更新すると、Markdown ファイルは、Microsoft Graph ドキュメント サイトに公開された場合にそれらの変更を自動的に取得します。</span><span class="sxs-lookup"><span data-stu-id="83e66-114">If you update code between these "marker" comments, the Markdown files will automatically get those changes when published to the Microsoft Graph documentation site.</span></span> <span data-ttu-id="83e66-115">これらのコメント以外のコードを更新する場合は、対応する Markdown を更新する必要がある可能性が高い可能性があります。</span><span class="sxs-lookup"><span data-stu-id="83e66-115">If you update code outside of those comments, it's very likely that you'll need to update the corresponding Markdown.</span></span>

## <a name="adding-features"></a><span data-ttu-id="83e66-116">機能の追加</span><span class="sxs-lookup"><span data-stu-id="83e66-116">Adding features</span></span>

<span data-ttu-id="83e66-117">評価は高く評価される一方で、サンプルに新機能を追加するプル要求は送信しないでおください。</span><span class="sxs-lookup"><span data-stu-id="83e66-117">While the enthusiasm is appreciated, please don't send pull requests to add new features to the sample.</span></span> <span data-ttu-id="83e66-118">このリポジトリは主に「最初のアプリをビルドする」チュートリアルなので、機能セットは設計上制限されています。</span><span class="sxs-lookup"><span data-stu-id="83e66-118">Because this repository is primarily a "build your first app" tutorial, the feature set is limited, by design.</span></span>

## <a name="submitting-pull-requests"></a><span data-ttu-id="83e66-119">プル要求を送信する</span><span class="sxs-lookup"><span data-stu-id="83e66-119">Submitting pull requests</span></span>

<span data-ttu-id="83e66-120">すべてのプル要求をブランチに送信 `master` してください。</span><span class="sxs-lookup"><span data-stu-id="83e66-120">Please submit all pull requests to the `master` branch.</span></span>

## <a name="when-do-changes-get-published"></a><span data-ttu-id="83e66-121">変更が公開されるのは、いつですか?</span><span class="sxs-lookup"><span data-stu-id="83e66-121">When do changes get published?</span></span>

<span data-ttu-id="83e66-122">Microsoft Graph チュートリアル サイトへの [更新プログラムの発行](https://docs.microsoft.com/graph/tutorials) は自動ではありません。</span><span class="sxs-lookup"><span data-stu-id="83e66-122">Publishing of updates to the [Microsoft Graph tutorials](https://docs.microsoft.com/graph/tutorials) site is not automatic.</span></span> <span data-ttu-id="83e66-123">変更は最初に分岐に昇格し、次にサイト管理者がビルドをトリガー `live` する必要があります。</span><span class="sxs-lookup"><span data-stu-id="83e66-123">Changes must first be promoted to the `live` branch, then a build must be triggered by the site admins.</span></span> <span data-ttu-id="83e66-124">これは通常、"必要に応じて" 行われます。</span><span class="sxs-lookup"><span data-stu-id="83e66-124">This is typically done on an "as-needed" basis.</span></span>

## <a name="code-of-conduct"></a><span data-ttu-id="83e66-125">Code of conduct</span><span class="sxs-lookup"><span data-stu-id="83e66-125">Code of conduct</span></span>

<span data-ttu-id="83e66-p106">このプロジェクトでは、[Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/) が採用されています。詳細については、「[規範に関する FAQ](https://opensource.microsoft.com/codeofconduct/faq/)」を参照してください。または、その他の質問やコメントがあれば、[opencode@microsoft.com](mailto:opencode@microsoft.com) までにお問い合わせください。</span><span class="sxs-lookup"><span data-stu-id="83e66-p106">This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.</span></span>
