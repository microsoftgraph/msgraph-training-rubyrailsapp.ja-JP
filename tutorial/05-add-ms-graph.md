<!-- markdownlint-disable MD002 MD041 -->

この演習では、Microsoft Graph をアプリケーションに組み込みます。 このアプリケーションでは、 [httparty](https://github.com/jnunemaker/httparty) gem を使用して Microsoft Graph を呼び出すことになります。

## <a name="create-a-graph-helper"></a>Graph ヘルパーを作成する

すべての API 呼び出しを管理するヘルパーを作成します。 CLI で次のコマンドを実行して、ヘルパーを生成します。

```Shell
rails generate helper Graph
```

新しく作成`./app/helpers/graph_helper.rb`したファイルを開き、コンテンツを次のように置き換えます。

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

## <a name="get-calendar-events-from-outlook"></a>Outlook から予定表のイベントを取得する

まず、ユーザーの予定表でイベントを表示する機能を追加します。CLI で、次のコマンドを実行して新しいコントローラーを追加します。

```Shell
rails generate controller Calendar index
```

これでルートを使用できるようになりました。`./app/view/layouts/application.html.erb` 内のナビゲーションバーにある **Calendar** リンクでこれを使用するように更新します。`<a class="nav-link" href="#">Calendar</a>` と書かれている行を次のように書き換えます。

```html
<%= link_to "Calendar", {:controller => :calendar, :action => :index}, class: "nav-link#{' active' if controller.controller_name == 'calendar'}" %>
```

[ユーザーのイベントを一覧表示](https://developer.microsoft.com/en-us/graph/docs/api-reference/v1.0/api/user_list_events)できるよう、Graph ヘルパーに新しいメソッドを追加します。`./app/helpers/graph_helper.rb` を開き、`GraphHelper` モジュールに次のメソッドを追加します。

```ruby
def get_calendar_events(token)
  get_events_url = '/v1.0/me/events'

  query = {
    '$select': 'subject,organizer,start,end',
    '$orderby': 'createdDateTime DESC'
  }

  response = make_api_call get_events_url, token, query

  raise response.parsed_response.to_s || "Request returned #{response.code}" unless response.code == 200
  response.parsed_response['value']
end
```

このコードが何をしているかを考えてみてください。

- 呼び出し先の URL は `/v1.0/me/events` になります
- `$select` パラメーターは、各イベントに対して返されるフィールドを、ビューが実際に使用するものだけに制限します
- `$orderby` パラメーターは、最新の項目が先頭になるように、結果を作成日時でソートします
- 正常なレスポンスの場合、 `value` キーに項目の配列を格納して返します

これでテストをする事ができます。`./app/controllers/calendar_controller.rb` を開き、このメソッドを呼び出せるよう `index` アクションを更新し結果を表示します。

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

サーバーを再起動します。サインインして、ナビゲーションバーの **[予定表]** リンクをクリックします。すべてが動作する場合は、ユーザーの予定表にイベントの JSON ダンプが表示されます。

## <a name="display-the-results"></a>結果を表示する

これで、HTML と CSS を追加して、よりわかりやすい方法で結果を表示することができます。

を`./app/views/calendar/index.html.erb`開き、その内容を次のように置き換えます。

```html
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    <% @events.each do |event| %>
      <tr>
        <td><%= event['organizer']['emailAddress']['name'] %></td>
        <td><%= event['subject'] %></td>
        <td><%= event['start']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
        <td><%= event['end']['dateTime'].to_time(:utc).localtime.strftime('%-m/%-d/%y %l:%M %p') %></td>
      </tr>
    <% end %>
  </tbody>
</table>
```

これにより、イベントのコレクションをループ処理して、テーブル行を1つずつ追加します。 `index`の`./app/controllers/calendar_controller.rb`アクション`render json: @events`から行を削除すると、アプリがイベントの表を表示するはずです。

![イベントの表のスクリーンショット](./images/add-msgraph-01.png)
