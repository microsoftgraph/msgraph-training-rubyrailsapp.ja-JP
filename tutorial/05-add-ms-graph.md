<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="3af08-101">この演習では、アプリケーションに Microsoft Graphを組み込む必要があります。</span><span class="sxs-lookup"><span data-stu-id="3af08-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="3af08-102">このアプリケーションでは[、httparty](https://github.com/jnunemaker/httparty) gem を使用して Microsoft Graph。</span><span class="sxs-lookup"><span data-stu-id="3af08-102">For this application, you will use the [httparty](https://github.com/jnunemaker/httparty) gem to make calls to Microsoft Graph.</span></span>

## <a name="create-a-graph-helper"></a><span data-ttu-id="3af08-103">カスタム ヘルパー Graphする</span><span class="sxs-lookup"><span data-stu-id="3af08-103">Create a Graph helper</span></span>

1. <span data-ttu-id="3af08-104">すべての API 呼び出しを管理するヘルパーを作成します。</span><span class="sxs-lookup"><span data-stu-id="3af08-104">Create a helper to manage all of your API calls.</span></span> <span data-ttu-id="3af08-105">CLI で次のコマンドを実行してヘルパーを生成します。</span><span class="sxs-lookup"><span data-stu-id="3af08-105">Run the following command in your CLI to generate the helper.</span></span>

    ```Shell
    rails generate helper Graph
    ```

1. <span data-ttu-id="3af08-106">**./app/helpers/graph_helper.rb** を開き、内容を次に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="3af08-106">Open **./app/helpers/graph_helper.rb** and replace the contents with the following.</span></span>

    ```ruby
    require 'httparty'

    # Graph API helper methods
    module GraphHelper
      GRAPH_HOST = 'https://graph.microsoft.com'.freeze

      def make_api_call(method, endpoint, token, headers = nil, params = nil, payload = nil)
        headers ||= {}
        headers[:Authorization] = "Bearer #{token}"
        headers[:Accept] = 'application/json'

        params ||= {}

        case method.upcase
        when 'GET'
          HTTParty.get "#{GRAPH_HOST}#{endpoint}",
                       :headers => headers,
                       :query => params
        when 'POST'
          headers['Content-Type'] = 'application/json'
          HTTParty.post "#{GRAPH_HOST}#{endpoint}",
                        :headers => headers,
                        :query => params,
                        :body => payload ? payload.to_json : nil
        else
          raise "HTTP method #{method.upcase} not implemented"
        end
      end
    end
    ```

<span data-ttu-id="3af08-107">このコードの動作を確認してください。</span><span class="sxs-lookup"><span data-stu-id="3af08-107">Take a moment to review what this code does.</span></span> <span data-ttu-id="3af08-108">Gem を介して要求されたエンドポイントに対して単純な GET 要求または POST `httparty` 要求を行います。</span><span class="sxs-lookup"><span data-stu-id="3af08-108">It makes a simple GET or POST request via the `httparty` gem to the requested endpoint.</span></span> <span data-ttu-id="3af08-109">ヘッダーにアクセス トークンを送信 `Authorization` し、渡されるクエリ パラメーターが含まれます。</span><span class="sxs-lookup"><span data-stu-id="3af08-109">It sends the access token in the `Authorization` header, and it includes any query parameters that are passed.</span></span>

<span data-ttu-id="3af08-110">たとえば、メソッドを使用して GET を実行 `make_api_call` するには、 `https://graph.microsoft.com/v1.0/me?$select=displayName` 次のように呼び出します。</span><span class="sxs-lookup"><span data-stu-id="3af08-110">For example, to use the `make_api_call` method to do a GET to `https://graph.microsoft.com/v1.0/me?$select=displayName`, you could call it like so:</span></span>

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

<span data-ttu-id="3af08-111">後で、アプリに Microsoft の機能を実装するGraphを構築します。</span><span class="sxs-lookup"><span data-stu-id="3af08-111">You'll build on this later as you implement more Microsoft Graph features into the app.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="3af08-112">Outlook からカレンダー イベントを取得する</span><span class="sxs-lookup"><span data-stu-id="3af08-112">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="3af08-113">CLI で、次のコマンドを実行して新しいコントローラーを追加します。</span><span class="sxs-lookup"><span data-stu-id="3af08-113">In your CLI, run the following command to add a new controller.</span></span>

    ```Shell
    rails generate controller Calendar index new
    ```

1. <span data-ttu-id="3af08-114">新しいルートを **./config/routes.rb に追加します**。</span><span class="sxs-lookup"><span data-stu-id="3af08-114">Add the new route to **./config/routes.rb**.</span></span>

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. <span data-ttu-id="3af08-115">カレンダー ビューを取得するには、Graphヘルパーに[新しいメソッドを追加します](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0)。</span><span class="sxs-lookup"><span data-stu-id="3af08-115">Add a new method to the Graph helper to [get a calendar view](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0).</span></span> <span data-ttu-id="3af08-116">**./app/helpers/graph_helper.rb** を開き、次のメソッドをモジュールに追加 `GraphHelper` します。</span><span class="sxs-lookup"><span data-stu-id="3af08-116">Open **./app/helpers/graph_helper.rb** and add the following method to the `GraphHelper` module.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    <span data-ttu-id="3af08-117">このコードの実行内容を考えましょう。</span><span class="sxs-lookup"><span data-stu-id="3af08-117">Consider what this code is doing.</span></span>

    - <span data-ttu-id="3af08-118">呼び出される URL は `/v1.0/me/calendarview` です。</span><span class="sxs-lookup"><span data-stu-id="3af08-118">The URL that will be called is `/v1.0/me/calendarview`.</span></span>
        - <span data-ttu-id="3af08-119">ヘッダーによって、結果の開始時刻と終了時刻がユーザーのタイム ゾーン `Prefer: outlook.timezone` に合わせて調整されます。</span><span class="sxs-lookup"><span data-stu-id="3af08-119">The `Prefer: outlook.timezone` header causes the start and end times in the results to be adjusted to the user's time zone.</span></span>
        - <span data-ttu-id="3af08-120">and `startDateTime` パラメーター `endDateTime` は、ビューの開始と終了を設定します。</span><span class="sxs-lookup"><span data-stu-id="3af08-120">The `startDateTime` and `endDateTime` parameters set the start and end of the view.</span></span>
        - <span data-ttu-id="3af08-121">パラメーター `$select` は、各イベントに返されるフィールドを、ビューが実際に使用するフィールドに制限します。</span><span class="sxs-lookup"><span data-stu-id="3af08-121">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
        - <span data-ttu-id="3af08-122">パラメーター `$orderby` は、開始時刻によって結果を並べ替える。</span><span class="sxs-lookup"><span data-stu-id="3af08-122">The `$orderby` parameter sorts the results by start time.</span></span>
        - <span data-ttu-id="3af08-123">パラメーター `$top` は、結果を 50 イベントに制限します。</span><span class="sxs-lookup"><span data-stu-id="3af08-123">The `$top` parameter limits the results to 50 events.</span></span>
    - <span data-ttu-id="3af08-124">応答が成功した場合、キーに含まれるアイテムの配列を返 `value` します。</span><span class="sxs-lookup"><span data-stu-id="3af08-124">For a successful response, it returns the array of items contained in the `value` key.</span></span>

1. <span data-ttu-id="3af08-125">新しいメソッドを Graphヘルパーに追加して、タイム ゾーン名に基づいて[IANA](https://www.iana.org/time-zones)タイム Windows参照します。</span><span class="sxs-lookup"><span data-stu-id="3af08-125">Add a new method to the Graph helper to lookup an [IANA time zone identifier](https://www.iana.org/time-zones) based on a Windows time zone name.</span></span> <span data-ttu-id="3af08-126">これは、Microsoft Graphがタイム ゾーン名としてWindows返し、Ruby **DateTime** クラスには IANA タイム ゾーン識別子が必要なので、これが必要です。</span><span class="sxs-lookup"><span data-stu-id="3af08-126">This is necessary because Microsoft Graph can return time zones as Windows time zone names, and the Ruby **DateTime** class requires IANA time zone identifiers.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. <span data-ttu-id="3af08-127">**./app/controllers/calendar_controller.rb** を開き、その内容全体を次に置き換えてください。</span><span class="sxs-lookup"><span data-stu-id="3af08-127">Open **./app/controllers/calendar_controller.rb** and replace its entire contents with the following.</span></span>

    ```ruby
    # Calendar controller
    class CalendarController < ApplicationController
      include GraphHelper

      def index
        # Get the IANA identifier of the user's time zone
        time_zone = get_iana_from_windows(user_timezone)

        # Calculate the start and end of week in the user's time zone
        start_datetime = Date.today.beginning_of_week(:sunday).in_time_zone(time_zone).to_time
        end_datetime = start_datetime.advance(:days => 7)

        @events = get_calendar_view access_token, start_datetime, end_datetime, user_timezone || []
        render json: @events
      rescue RuntimeError => e
        @errors = [
          {
            :message => 'Microsoft Graph returned an error getting events.',
            :debug => e
          }
        ]
      end
    end
    ```

1. <span data-ttu-id="3af08-128">サーバーを再起動します。</span><span class="sxs-lookup"><span data-stu-id="3af08-128">Restart the server.</span></span> <span data-ttu-id="3af08-129">サインインして、ナビゲーション バー **の [予定表** ] リンクをクリックします。</span><span class="sxs-lookup"><span data-stu-id="3af08-129">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="3af08-130">すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。</span><span class="sxs-lookup"><span data-stu-id="3af08-130">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="3af08-131">結果の表示</span><span class="sxs-lookup"><span data-stu-id="3af08-131">Display the results</span></span>

<span data-ttu-id="3af08-132">HTML を追加して、結果をユーザーフレンドリーに表示できます。</span><span class="sxs-lookup"><span data-stu-id="3af08-132">Now you can add HTML to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="3af08-133">**./app/views/calendar/index.html.erb** を開き、その内容を次に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="3af08-133">Open **./app/views/calendar/index.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    <span data-ttu-id="3af08-134">これにより、イベントのコレクションがループされ、各イベントにテーブル行が追加されます。</span><span class="sxs-lookup"><span data-stu-id="3af08-134">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="3af08-135">`render json: @events` `index` **./app/controllers/calendar_controller.rb のアクションから行を削除します**。</span><span class="sxs-lookup"><span data-stu-id="3af08-135">Remove the `render json: @events` line from the `index` action in **./app/controllers/calendar_controller.rb**.</span></span>

1. <span data-ttu-id="3af08-136">ページを更新すると、アプリはイベントのテーブルをレンダリングする必要があります。</span><span class="sxs-lookup"><span data-stu-id="3af08-136">Refresh the page and the app should now render a table of events.</span></span>

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
