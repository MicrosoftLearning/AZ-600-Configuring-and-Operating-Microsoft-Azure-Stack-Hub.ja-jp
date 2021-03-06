---
lab:
  title: ラボ:Azure Stack Hub のストレージ アカウントを構成および管理する
  module: 'Module 5: Manage Infrastructure'
ms.openlocfilehash: 0c83afc42ddfb6e7f9c82efeb9921c2c80e832ad
ms.sourcegitcommit: f99a365741ad02181da3b4fc4a921d5437810bc2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/02/2022
ms.locfileid: "139252337"
---
# <a name="lab---configure-and-manage-azure-stack-hub-storage-accounts"></a>ラボ - Azure Stack Hub のストレージ アカウントを構成および管理する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- なし

## <a name="estimated-time"></a>推定所要時間

30 分

## <a name="lab-scenario"></a>ラボのシナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、 そのストレージ アカウントを構成して管理する必要があります。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure Stack Hub のストレージを構成する
- Azure Stack Hub のストレージ アカウントを管理する

## <a name="lab-environment"></a>ラボ環境 

ラボ環境は、次のコンポーネントから構成されています。

- 次のアクセス ポイントを持つ、**AzS-HOST1** サーバーで実行されている ASDK デプロイ。

  - 管理者ポータル: https://adminportal.local.azurestack.external
  - 管理者の ARM エンドポイント: https://adminmanagement.local.azurestack.external
  - ユーザー ポータル: https://portal.local.azurestack.external
  - ユーザーの ARM エンドポイント: https://management.local.azurestack.external

- 管理ユーザー:

  - ASDK クラウド オペレーターのユーザー名: **CloudAdmin@azurestack.local**
  - ASDK クラウド オペレーターのパスワード:**Pa55w.rd1234**
  - ASDK ホスト管理者のユーザー名: **AzureStackAdmin@azurestack.local**
  - ASDK ホスト管理者のパスワード:**Pa55w.rd1234**

このラボでは、追加のユーザー アカウントを作成します。

### <a name="exercise-1-configure-azure-stack-hub-storage"></a>演習 1:Azure Stack Hub のストレージを構成する

この演習では、Azure Stack Hub のストレージを構成します。

1. ストレージ アカウントの保有オプションを構成する
1. ストレージ サービスが含まれるプランを作成する
1. このプランに基づいてオファーを作成する

#### <a name="task-1-configure-storage-account-retention"></a>タスク 1:ストレージ アカウントの保有オプションを構成する

このタスクでは次のことを行います。

- ストレージ アカウントの保有オプションを構成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザー ウィンドウを開いて [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示し、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、**[すべてのサービス]** をクリックします。
1. **[すべてのサービス]** ブレードで **[管理]** セクションを選択してから、**[リージョンの管理]** をクリックします。
1. **[ローカル]** ブレードで、**[リソース プロバイダー]** をクリックします。
1. **[リソース プロバイダー]** ブレードで、**[ストレージ]** をクリックします。
1. **[ストレージ]** ブレードで、**[構成]** をクリックします。
1. **[ストレージ – 構成]** ブレードの **[削除したストレージ アカウントの保有期間 (日単位)]** テキスト ボックスに「10」と入力して、**[保存]** をクリックします。


#### <a name="task-2-create-a-plan-containing-storage-services"></a>タスク 2:ストレージ サービスが含まれるプランを作成する

このタスクでは次のことを行います。

- ストレージ サービスが含まれるプランを作成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、**[+ リソースの作成]** をクリックします。 
1. **[新規]** ブレードで **[オファーとプラン]** をクリックします。
1. **[オファーとプラン]** ブレードで **[プラン]** をクリックします。
1. **[新しいプラン]** ブレードの **[基本]** タブで、次のように設定を行います。

    - 表示名: **storage-plan1**
    - リソース名: **storage-plan1**
    - リソース グループ: 新しいリソース グループの名前 **storage-plans-RG**

1. **[次へ: サービス >]** をクリックします。
1. **[新しいプラン]** ブレードの **[サービス]** タブで、**[Microsoft.Storage]** チェックボックスをオンにします。
1. **[次へ: クォータ >]** をクリックします。
1. **[新しいプラン]** ブレードの **[クォータ]** タブで、**[Microsoft.Storage]** ドロップダウン リストの横にある **[新規作成]** をクリックします。
1. **[Create Storage quota]\(ストレージ クォータの作成\)** ブレードで、次の設定を指定して **[OK]** をクリックします。

    - 名前: **storage-plan1-storage-quota**
    - 容量 (GB):**200**
    - ストレージ アカウントの数:**5**

1. **[Review + create]\(レビュー + 作成\)** をクリックし、 **[作成]** をクリックします。

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。


#### <a name="task-3-create-an-offer-based-on-the-plan"></a>タスク 3:このプランに基づいてオファーを作成する

このタスクでは次のことを行います。

- このプランに基づいてオファーを作成する 

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、**[+ リソースの作成]** をクリックします。 
1. **[新規]** ブレードで **[オファーとプラン]** をクリックします。
1. **[オファーとプラン]** ブレードで **[オファー]** をクリックします。
1. **[新しいオファーの作成]** ブレードの **[基本]** タブで、次のように設定を行います。

    - 表示名: **storage-offer1**
    - リソース名: **storage-offer1**
    - リソース グループ: **storage-offers-RG**
    - このオファーをパブリックに設定する:**はい**

1. **[次へ: 基本プラン >]** をクリックします。 
1. **[新しいオファーの作成]** ブレードの **[基本プラン]** タブで、**[storage-plan1]** エントリの横にあるチェックボックスをオンにします。
1. **[次へ: アドオン プラン >]** をクリックします。
1. **[アドオン プラン]** の設定は既定値にしたまま、**[確認および作成]** をクリックしてから **[作成]** をクリックします。

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。

>**レビュー**:この演習では、削除したストレージ アカウントの保有期間を設定したほか、**Microsoft.Storage** サービス ベース プランが含まれるパブリック オファーを作成しました。 


### <a name="exercise-2-manage-azure-stack-hub-storage-accounts"></a>演習 2:Azure Stack Hub のストレージ アカウントを管理する

この演習では、Azure Stack Hub のストレージ アカウントを管理します。

1. (テナント ユーザーとして) Azure ストレージ アカウントを作成および削除する
1. (クラウド オペレーターとして) 削除したストレージ アカウントを復元する
1. (クラウド オペレーターとして) ストレージ容量を開放する

#### <a name="task-1-create-and-delete-azure-storage-accounts-as-a-tenant-user"></a>タスク 1:(テナント ユーザーとして) Azure ストレージ アカウントを作成および削除する

このタスクでは次のことを行います。

- (テナント ユーザーとして) Azure ストレージ アカウントを作成および削除する 

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの InPrivate セッションを開始します。
1. Web ブラウザー ウィンドウで [Azure Stack Hub ユーザー ポータル](https://portal.local.azurestack.external)に移動し、 **t1u1@azurestack.local** とパスワード **Pa55w.rd** でサインインします。
1. Azure Stack Hub ユーザー ポータルのダッシュボードで、**[サブスクリプションの取得]** タイルをクリックします。
1. **[サブスクリプションの取得]** ブレードの **[名前]** テキスト ボックスに「**t1u1-storage-subscription1**」と入力します。
1. オファーの一覧で **[storage-offer1]** を選択し、**[作成]** をクリックします。
1. メッセージ "**サブスクリプションが作成されました。サブスクリプションの使用を開始するには、ポータルを更新する必要があります**" が表示されたら、 **[最新の情報に更新]** をクリックします。 
1. Azure Stack Hub テナント パネルのハブ メニューで、**[すべてのサービス]** をクリックします。
1. サービスの一覧で、**[サブスクリプション]** をクリックします。
1. **[サブスクリプション]** ブレードで、**[t1u1-storage-subscription1]** をクリックします。
1. **[t1u1-storage-subscription1]** ブレードで **[リソース]** をクリックします。
1. **[t1u1-storage-subscription1 - リソース]** ブレードで **[+ 追加]** をクリックします。
1. **[新規作成]** ブレードで **[データとストレージ]** をクリックしてから **[ストレージ アカウント]** をクリックします。
1. **[ストレージ アカウントの作成]** ブレードの **[基本]** タブで、次の設定を指定します (他の設定は既定値のままにします)。

    - リソース グループ: 新しいリソース グループの名前 **storageRG**
    - 名前: 3～24 の小文字または数字から成る一意の名前
    - 場所: **ローカル**
    - パフォーマンス: **Standard**
    - アカウント種類:**ストレージ (汎用 v1)**
    - レプリケーション：**ローカル冗長ストレージ (LRS)**

1. **[ストレージ アカウントの作成]** ブレードの **[基本]** タブで、 **[次へ: 詳細 >]** をクリックします。
1. **[ストレージ アカウントの作成]** ブレードの **[詳細]** タブで、設定を規定値にしたまま **[確認および作成]** をクリックします。
1. **[ストレージ アカウントの作成]** ブレードの **[レビュー + 作成]** タブで、**[作成]** を選択します。

    >**注**:ストレージ アカウントがプロビジョニングされるまで待ちます。 これには 1 分ほどかかります。

1. 手順 11～16 に従って、同じ設定を持つもう 1 つのストレージ アカウントを同じリソース グループ内に作成します。 

    >**注**:次に、両方のストレージ アカウントを削除します。 必ず最初にこれらの名前を記録しておいてください。 これらは、このラボの後半で必要になります。

1. Azure Stack Hub ユーザー ポータルのハブ メニューで **[すべてのサービス]** をクリックしてから、**[すべてのサービス]** ブレードで **[ストレージ アカウント]** をクリックします。
1. **[ストレージ アカウント]** ブレードで、サブスクリプション フィルターに **"t1u1-storage-subscription1"** が含まれていることを確認してから、ストレージ アカウントの一覧で、このタスクの前半で作成した両方のストレージ アカウントを選択して **[削除]** をクリックします。
1. **[リソースの削除]** ブレードの **[削除の確認]** テキスト ボックスに「**Yes**」と入力して、**[削除]** をクリックします。

#### <a name="task-2-recover-a-deleted-storage-account-as-a-cloud-operator"></a>タスク 2:(クラウド オペレーターとして) 削除したストレージ アカウントを復元する 

このタスクでは次のことを行います。

- (クラウド オペレーターとして) 削除したストレージ アカウントを復元する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Azure Stack Hub 管理者ポータルが表示されている Web ブラウザーの画面に切り替え、**[すべてのサービス]** をクリックし、**[すべてのサービス]** ブレードで **[管理]** をクリックしてから、管理サービスの一覧で **[ストレージ]** をクリックします。
1. **[ストレージ]** ブレードで、**[ストレージ アカウント]** をクリックします。
1. **[ストレージ \| ストレージ アカウント]** ブレードの **[アカウント名]** フィルター テキスト ボックスに、前のタスクで作成した最初のストレージ アカウントの名前を入力し、そのストレージ アカウント名が **[削除済み]** 状態で一覧に表示されることを確認します。
1. 最初に削除したストレージ アカウントを表すエントリをクリックします。
1. **[ストレージ アカウント]** ブレードで、**[復元]** をクリックします。 確認メッセージが表示されたら、[**はい**] をクリックします。

    >**注**: 操作が完了するまで待ちます。 これに要する時間は 1 分未満です。 復元されたストレージ アカウントがユーザー ポータルに表示されるまで、さらに時間がかかる場合があることに留意してくだい。

1. **[ストレージ アカウント]** ブレードを閉じます。


#### <a name="task-3-reclaim-storage-capacity-as-a-cloud-operator"></a>タスク 3:(クラウド オペレーターとして) ストレージ容量を開放する 

このタスクでは次のことを行います。

- (クラウド オペレーターとして) ストレージ容量を開放する

1. **AzS-HOST1** へのリモート デスクトップ セッション内の Azure Stack Hub 管理者ポータルで、CloudAdmin@azurestack.local としてログオンし、 **[ストレージ \| ストレージ アカウント]** ブレードの **[最新の情報に更新]** をクリックします。
1. **[アカウント名]** フィルター テキスト ボックスに最初のストレージ アカウント名が入力されている状態では、そのストレージ アカウントが **"アクティブ"** 状態として表示されることを確認します。
1. **[ストレージ \| ストレージ アカウント]** ブレードの **[アカウント名]** フィルター テキスト ボックスに、前のタスクで作成した 2 番目のストレージ アカウントの名前を入力し、ストレージ アカウント名が **[削除済み]** 状態で一覧に表示されることを確認します。
1. **[ストレージ \| ストレージ アカウント]** ブレードで **[Reclaim space]\(領域の回収\)** をクリックし、確認メッセージが表示されたら **[はい]** をクリックします。

    >**注**: 操作が完了するまで待ちます。 これに要する時間は 1 分未満です。

1. ブラウザー ページを更新します。
1. **[ストレージ \| ストレージ アカウント]** ブレードに戻り、[**アカウント名]** フィルター テキスト ボックスに、前のタスクで作成した 2 番目のストレージ アカウントの名前を入力し、その名前が検索結果の一覧に表示されなくなることを確認します。 

>**レビュー**:この演習では、テナント ユーザーとしてストレージ アカウントを作成してから削除したほか、クラウド オペレーターとして削除したストレージ アカウントを復元し、ストレージ容量を開放しました。
