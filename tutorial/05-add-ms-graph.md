<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込みます。 このアプリケーションでは、 [httparty](https://github.com/jnunemaker/httparty) gem を使用して Microsoft Graph を呼び出すことになります。

## <a name="create-a-graph-helper"></a>Graph ヘルパーを作成する

1. すべての API 呼び出しを管理するヘルパーを作成します。 CLI で次のコマンドを実行して、ヘルパーを生成します。

    ```Shell
    rails generate helper Graph
    ```

1. **Graph_helper rb**を開き、コンテンツを次のように置き換えます。

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

このコードの内容を確認してください。 このメソッドは、要求された`httparty`エンドポイントに対して gem を使用して、簡単な GET 要求を作成します。 このメソッドは、 `Authorization`ヘッダーにアクセストークンを送信し、渡されたすべてのクエリパラメーターを含みます。

たとえば、 `make_api_call`メソッドを使用して GET を`https://graph.microsoft.com/v1.0/me?$select=displayName`実行するには、次のように呼び出すことができます。

```ruby
make_api_call `/v1.0/me`, access_token, { '$select': 'displayName' }
```

この後、より多くの Microsoft Graph 機能をアプリに実装する際に、これに基づいて構築します。

## <a name="get-calendar-events-from-outlook"></a>Outlook からカレンダー イベントを取得する

1. CLI で、次のコマンドを実行して新しいコントローラーを追加します。

    ```Shell
    rails generate controller Calendar index
    ```

1. 新しいルートを **./config/routes.rb**に追加します。

    ```ruby
    get 'calendar', to: 'calendar#index'
    ```

1. Graph ヘルパーに新しいメソッドを追加して、[ユーザーのイベントを一覧表示](/graph/api/user-list-events?view=graph-rest-1.0)します。 **Web.config/graph_helper**を開き、次のメソッドを`GraphHelper`モジュールに追加します。

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="GetCalendarSnippet":::

    このコードの実行内容を考えましょう。

    - 呼び出される URL は `/v1.0/me/events` です。
    - パラメーター `$select`は、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します。
    - パラメーター `$orderby`は、生成された日付と時刻で結果を並べ替えます。最新のアイテムが最初に表示されます。
    - 正常な応答の場合は、 `value`キーに含まれている項目の配列を返します。

1. **/App/controllers/calendar_controller rb**を開き、その内容全体を次のように置き換えます。

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

1. サーバーを再起動します。 サインインして、ナビゲーションバーの [**予定表**] リンクをクリックします。 すべてが正常に機能していれば、ユーザーのカレンダーにイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果の表示

これで、HTML を追加して、よりわかりやすい方法で結果を表示することができます。

1. **/App/views/calendar/index.html.erb**を開き、その内容を次のように置き換えます。

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/index.html.erb" id="CalendarSnippet":::

    これにより、イベントのコレクションがループされ、各イベントにテーブル行が追加されます。

1. **/App/controllers/calendar_controller rb**の`render json: @events` `index`アクションから行を削除します。

1. ページを更新すると、アプリがイベントの表を表示するようになります。

    ![イベント表のスクリーンショット](./images/add-msgraph-01.png)
