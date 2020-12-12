<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b98fd-101">このセクションでは、ユーザーの予定表にイベントを作成する機能を追加します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-101">In this section you will add the ability to create events on the user's calendar.</span></span>

1. <span data-ttu-id="b98fd-102">**./app/helpers/graph_helper.rb** を開き **、Graph** クラスに次のメソッドを追加します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-102">Open **./app/helpers/graph_helper.rb** and add the following method to the **Graph** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/helpers/graph_helper.rb" id="CreateEventSnippet":::

1. <span data-ttu-id="b98fd-103">**./app/controllers/calendar_controller** を開き **、CalendarController** クラスに次のルートを追加します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-103">Open **./app/controllers/calendar_controller** and add the following route to the **CalendarController** class.</span></span>

    :::code language="ruby" source="../demo/graph-tutorial/app/controllers/calendar_controller.rb" id="CreateEventRouteSnippet":::

1. <span data-ttu-id="b98fd-104">**./config/routes.rb を** 開き、新しいルートを追加します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-104">Open **./config/routes.rb** and add the new route.</span></span>

    ```ruby
    post 'calendar/new', :to => 'calendar#create'
    ```

1. <span data-ttu-id="b98fd-105">**./app/views/calendar/new.html.erb** を開き、その内容を次の内容に置き換えます。</span><span class="sxs-lookup"><span data-stu-id="b98fd-105">Open **./app/views/calendar/new.html.erb** and replace its contents with the following.</span></span>

    :::code language="html" source="../demo/graph-tutorial/app/views/calendar/new.html.erb" id="NewEventFormSnippet":::

1. <span data-ttu-id="b98fd-106">変更を保存し、アプリを更新します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-106">Save your changes and refresh the app.</span></span> <span data-ttu-id="b98fd-107">[予定表 **] ページ** で、[新しいイベント] **ボタンを選択** します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-107">On the **Calendar** page, select the **New event** button.</span></span> <span data-ttu-id="b98fd-108">フォームに入力し、[作成] **を選択** して新しいイベントを作成します。</span><span class="sxs-lookup"><span data-stu-id="b98fd-108">Fill in the form and select **Create** to create a new event.</span></span>

    ![新しいイベント フォームのスクリーンショット](images/create-event-01.png)
