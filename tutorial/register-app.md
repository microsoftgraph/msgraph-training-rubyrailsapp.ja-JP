<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="29fc3-101">この演習では、Azure Active Directory 管理センターを使用して、新しい Azure AD web アプリケーション登録を作成します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-101">In this exercise, you will create a new Azure AD web application registration using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="29fc3-102">ブラウザーを開き、[Azure Active Directory 管理センター](https://aad.portal.azure.com)へ移動します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="29fc3-103">**個人用アカウント** (別名: Microsoft アカウント)、または**職場/学校アカウント**を使用してログインします。</span><span class="sxs-lookup"><span data-stu-id="29fc3-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="29fc3-104">左側のナビゲーションで [ **Azure Active Directory** ] を選択し、[**管理**] の下にある [**アプリの登録**] を選択します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-104">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="29fc3-105">アプリの登録のスクリーンショット</span><span class="sxs-lookup"><span data-stu-id="29fc3-105">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="29fc3-106">**[新規登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-106">Select **New registration**.</span></span> <span data-ttu-id="29fc3-107">**[アプリケーションを登録]** ページで、次のように値を設定します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-107">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="29fc3-108">`Ruby Graph Tutorial` に **[名前]** を設定します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-108">Set **Name** to `Ruby Graph Tutorial`.</span></span>
    - <span data-ttu-id="29fc3-109">**[サポートされているアカウントの種類]** を **[任意の組織のディレクトリ内のアカウントと個人用の Microsoft アカウント]** に設定します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-109">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="29fc3-110">**[リダイレクト URI]** で、最初のドロップダウン リストを `Web` に設定し、それから `http://localhost:3000/auth/microsoft_graph_auth/callback` に値を設定します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-110">Under **Redirect URI**, set the first drop-down to `Web` and set the value to `http://localhost:3000/auth/microsoft_graph_auth/callback`.</span></span>

    ![[アプリケーションの登録] ページのスクリーンショット](./images/aad-register-an-app.png)

1. <span data-ttu-id="29fc3-112">**[登録]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-112">Choose **Register**.</span></span> <span data-ttu-id="29fc3-113">**Ruby Graph のチュートリアル**ページで、**アプリケーション (クライアント) ID**の値をコピーして保存します。次の手順で必要になります。</span><span class="sxs-lookup"><span data-stu-id="29fc3-113">On the **Ruby Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![新しいアプリの登録のアプリケーション ID のスクリーンショット](./images/aad-application-id.png)

1. <span data-ttu-id="29fc3-115">**[管理]** で **[証明書とシークレット]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-115">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="29fc3-116">**[新しいクライアント シークレット]** ボタンを選択します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-116">Select the **New client secret** button.</span></span> <span data-ttu-id="29fc3-117">**[説明]** に値を入力して、**[有効期限]** のオプションのいずれかを選び、**[追加]** を選択します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-117">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![[クライアントシークレットの追加] ダイアログのスクリーンショット](./images/aad-new-client-secret.png)

1. <span data-ttu-id="29fc3-119">クライアント シークレットの値をコピーしてから、このページから移動します。</span><span class="sxs-lookup"><span data-stu-id="29fc3-119">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="29fc3-120">次の手順で行います。</span><span class="sxs-lookup"><span data-stu-id="29fc3-120">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="29fc3-121">このクライアント シークレットは今後表示されないため、今必ずコピーするようにしてください。</span><span class="sxs-lookup"><span data-stu-id="29fc3-121">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![新しく追加されたクライアントシークレットのスクリーンショット](./images/aad-copy-client-secret.png)