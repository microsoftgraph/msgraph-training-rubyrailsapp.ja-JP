<!-- markdownlint-disable MD002 MD041 -->

この演習では、アプリケーションに Microsoft Graphを組み込む必要があります。 このアプリケーションでは[、httparty](https://github.com/jnunemaker/httparty) gem を使用して Microsoft Graph。

## <a name="create-a-graph-helper"></a>カスタム ヘルパー Graphする

1. すべての API 呼び出しを管理するヘルパーを作成します。 CLI で次のコマンドを実行してヘルパーを生成します。

    ```Shell
    rails generate helper Graph
    ```

1. **./app/helpers/graph_helper.rb** を開き、内容を次に置き換えてください。

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

このコードの動作を確認してください。 Gem を介して要求されたエンドポイントに対して単純な GET 要求または POST `httparty` 要求を行います。 ヘッダーにアクセス トークンを送信 `Authorization` し、渡されるクエリ パラメーターが含まれます。

たとえば、メソッドを使用して GET を実行 `make_api_call` するには、 `https://graph.microsoft.com/v1.0/me?$select=displayName` 次のように呼び出します。

```ruby
make_api_call 'GET', '/v1.0/me', access_token, {}, { '$select': 'displayName' }
```

後で、アプリに Microsoft の機能を実装するGraphを構築します。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

1. CLI で、次のコマンドを実行して新しいコントローラーを追加します。

    ```Shell
    rails generate controller Calendar index new
    ```

1. 新しいルートを **./config/routes.rb に追加します**。

    ```ruby
    get 'calendar', :to => 'calendar#index'
    ```

1. カレンダー ビューを取得するには、Graphヘルパーに[新しいメソッドを追加します](https://docs.microsoft.com/graph/api/calendar-list-calendarview?view=graph-rest-1.0)。 **./app/helpers/graph_helper.rb** を開き、次のメソッドをモジュールに追加 `GraphHelper` します。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    このコードの実行内容を考えましょう。

    - 呼び出される URL は `/v1.0/me/calendarview` です。
        - ヘッダーによって、結果の開始時刻と終了時刻がユーザーのタイム ゾーン `Prefer: outlook.timezone` に合わせて調整されます。
        - and `startDateTime` パラメーター `endDateTime` は、ビューの開始と終了を設定します。
        - パラメーター `$select` は、各イベントに返されるフィールドを、ビューが実際に使用するフィールドに制限します。
        - パラメーター `$orderby` は、開始時刻によって結果を並べ替える。
        - パラメーター `$top` は、結果を 50 イベントに制限します。
    - 応答が成功した場合、キーに含まれるアイテムの配列を返 `value` します。

1. 新しいメソッドを Graphヘルパーに追加して、タイム ゾーン名に基づいて[IANA](https://www.iana.org/time-zones)タイム Windows参照します。 これは、Microsoft Graphがタイム ゾーン名としてWindows返し、Ruby **DateTime** クラスには IANA タイム ゾーン識別子が必要なので、これが必要です。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="ZoneMappingSnippet":::

1. **./app/controllers/calendar_controller.rb** を開き、その内容全体を次に置き換えてください。

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

1. サーバーを再起動します。 サインインして、ナビゲーション バー **の [予定表** ] リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

HTML を追加して、結果をユーザーフレンドリーに表示できます。

1. **./app/views/calendar/index.html.erb** を開き、その内容を次に置き換えます。

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    これにより、イベントのコレクションがループされ、各イベントにテーブル行が追加されます。

1. `render json: @events` `index` **./app/controllers/calendar_controller.rb のアクションから行を削除します**。

1. ページを更新すると、アプリはイベントのテーブルをレンダリングする必要があります。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
