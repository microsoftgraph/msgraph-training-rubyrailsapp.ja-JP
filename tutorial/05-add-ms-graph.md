<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="5f5c5-101">この演習では、Microsoft Graph をアプリケーションに組み込みます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="5f5c5-102">このアプリケーションでは、 [httparty](https://github.com/jnunemaker/httparty) gem を使用して Microsoft Graph を呼び出すことになります。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="5f5c5-103">Graph ヘルパーを作成する</span><span class="sxs-lookup"><span data-stu-id="5f5c5-103">Create a Graph helper</span></span>

1. <span data-ttu-id="5f5c5-104">すべての API 呼び出しを管理するヘルパーを作成します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="5f5c5-105">CLI で次のコマンドを実行して、ヘルパーを生成します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="5f5c5-106">**Graph_helper rb**を開き、コンテンツを次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(endpoint, token, params = nil)
        headers = {
          Authorization: "Bearer #{token}"
        }

        query = params || {}

        HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                     headers: headers,
                     query: query
      end
    end
    ```

<span data-ttu-id="5f5c5-107">このコードの内容を確認してください。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="5f5c5-108">このメソッドは、要求された`httparty`エンドポイントに対して gem を使用して、簡単な GET 要求を作成します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-108">It makes a simple GET request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="5f5c5-109">このメソッドは、 `Authorization`ヘッダーにアクセストークンを送信し、渡されたすべてのクエリパラメーターを含みます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="5f5c5-110">たとえば、 `make_api_call`メソッドを使用して GET を`https://graph.microsoft.com/v1.0/me?$select=displayName`実行するには、次のように呼び出すことができます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

<span data-ttu-id="5f5c5-111">この後、より多くの Microsoft Graph 機能をアプリに実装する際に、これに基づいて構築します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="5f5c5-112">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="5f5c5-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="5f5c5-113">CLI で、次のコマンドを実行して新しいコントローラーを追加します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index
    ```

1. <span data-ttu-id="5f5c5-114">新しいルートを **./config/routes.rb**に追加します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. <span data-ttu-id="5f5c5-115">Graph ヘルパーに新しいメソッドを追加して、[ユーザーのイベントを一覧表示](/graph/api/user-list-events?view=graph-rest-1.0)します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-115">Add a new method to the Graph helper to [list the user's events](/graph/api/user-list-events?view=graph-rest-1.0).</span></span> <span data-ttu-id="5f5c5-116">**Web.config/graph_helper**を開き、次のメソッドを`GraphHelper`モジュールに追加します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="5f5c5-117">このコードの実行内容を考えましょう。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="5f5c5-118">呼び出される URL は `/v1.0/me/events` です。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-118">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="5f5c5-119">パラメーター `$select`は、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-119">The `$select` parameter limits the fields returned for each events to just those our view will actually use.</span></span>
    - <span data-ttu-id="5f5c5-120">パラメーター `$orderby`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-120">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="5f5c5-121">正常な応答の場合は、 `value`キーに含まれている項目の配列を返します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-121">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="5f5c5-122">**/App/controllers/calendar_controller rb**を開き、その内容全体を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-122">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        @events = get_calendar_events access_token || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            message: 'Microsoft Graph returned an error getting events.',
            debug: e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="5f5c5-123">サーバーを再起動します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-123">Restart the server.</span></span> <span data-ttu-id="5f5c5-124">サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-124">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="5f5c5-125">すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-125">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="5f5c5-126">結果の表示</span><span class="sxs-lookup"><span data-stu-id="5f5c5-126">Display the results</span></span>

<span data-ttu-id="5f5c5-127">これで、HTML を追加して、よりわかりやすい方法で結果を表示することができます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-127">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="5f5c5-128">**/App/views/calendar/index.html.erb**を開き、その内容を次のように置き換えます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-128">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="5f5c5-129">これにより、イベントのコレクションがループされ、各イベントにテーブル行が追加されます。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-129">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="5f5c5-130">**/App/controllers/calendar_controller rb**の`render json: @events` `index`アクションから行を削除します。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-130">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="5f5c5-131">ページを更新すると、アプリがイベントの表を表示するようになります。</span><span class="sxs-lookup"><span data-stu-id="5f5c5-131">Refresh the page and the app should now render a table of events.</span></span>

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
