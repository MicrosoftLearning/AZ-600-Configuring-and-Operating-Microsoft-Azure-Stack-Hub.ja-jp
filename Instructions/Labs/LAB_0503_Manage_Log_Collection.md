---
lab:
  title: ラボ:Azure Stack Hub でログ収集を管理する
  module: 'Module 5: Manage Infrastructure'
ms.openlocfilehash: 390643cfd94ddc4364e44c7c20d160d33b231eb7
ms.sourcegitcommit: 3ce6441f824c1ac2b22159d6830eba55dba5ba66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/01/2022
ms.locfileid: "139251631"
---
# <a name="lab---manage-log-collection-in-azure-stack-hub"></a>ラボ - Azure Stack Hub でログ収集を管理する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- なし

## <a name="estimated-time"></a>推定所要時間

30 分

## <a name="lab-scenario"></a>ラボのシナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、 ログ収集の実行方法を特定する必要があります。

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- 自動 Azure Stack Hub ログ収集を実行する

## <a name="lab-environment"></a>ラボ環境 

ラボ環境は、次のコンポーネントから構成されています。

- 次のアクセス ポイントを持つ、**AzSHOST-1** サーバーで実行されている ASDK デプロイ。

  - 管理者ポータル: https://adminportal.local.azurestack.external
  - 管理者の ARM エンドポイント: https://adminmanagement.local.azurestack.external
  - ユーザー ポータル: https://portal.local.azurestack.external
  - ユーザーの ARM エンドポイント: https://management.local.azurestack.external

- 管理ユーザー:

  - ASDK クラウド オペレーターのユーザー名: **CloudAdmin@azurestack.local**
  - ASDK クラウド オペレーターのパスワード:**Pa55w.rd1234**
  - ASDK ホスト管理者のユーザー名: **AzureStackAdmin@azurestack.local**
  - ASDK ホスト管理者のパスワード:**Pa55w.rd1234**


### <a name="exercise-1-explore-azure-stack-hub-diagnostic-logs-collection-capabilities"></a>演習 1:Azure Stack Hub の診断ログ収集機能について学習する

この演習では、診断ログの収集オプションを管理するためのさまざまな方法について学びます。

1. 診断ログのプロアクティブな収集を有効にする
1. 診断ログをオンデマンドで送信する
1. 診断ログをローカル ファイル共有にコピーする

#### <a name="task-1-enable-proactive-diagnostic-log-collection"></a>タスク 1:診断ログのプロアクティブな収集を有効にする

このタスクでは次のことを行います。

- ログのプロアクティブな収集を有効にする

>**注**:プロアクティブなログ収集では、ユーザーがサポート ケースを開く前に診断ログが自動的に収集され、Azure Stack Hub から Microsoft に送信されます。 これらのログは、システム正常性アラートが発生した場合にのみ収集され、サポート ケースのコンテキストで Microsoft サポートによってのみアクセスされます。

1. 必要であれば、次の資格情報を使用して **AzSHOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザー ウィンドウを開いて [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示し、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、ハブ メニューで「**ヘルプとサポート**」をクリックします。
1. 「**概要**」ブレードで「**ログの収集**」をクリックします
1. **[概要] \| [Log Collection]\(ログ収集\)** ブレードで、 **[Enable proactive log collection]\(ログのプロアクティブな収集を有効にする\)** をクリックします。

    >**注**:「**ログのプロアクティブな収集を有効にする**」オプションが利用できない場合は、次の手順に直接進みます。 

1. 「**設定**」ブレードで、次のように設定してから「**保存**」をクリックします。

    - ログのプロアクティブな収集:**有効にする**
    - ログの場所:**Azure (推奨)**

1. **[概要] \| [Log Collection]\(ログ収集\)** ブレードに戻り、 **[設定]** をクリックして、ログのプロアクティブな収集が指定した設定に従って構成されていることを確認します。


#### <a name="task-2-send-diagnostic-logs-on-demand"></a>タスク 2:診断ログをオンデマンドで送信する

このタスクでは次のことを行います。

- Azure Stack Hub 管理者ポータルを使用して、診断ログをオンデマンドで送信する
- Azure Stack Hub PowerShell を使用して、診断ログをオンデマンドで送信する 

>**注**:この機能は「*今すぐログを送信*」と呼ばれています。

>**注**:まずは、Azure Stack Hub 管理者ポータルを使用して診断ログをオンデマンドで送信することから始めます。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーのウィンドウに [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示し、ハブ メニューの **[ヘルプとサポート]** をクリックします。
1. 「**概要**」ブレードで「**ログの収集**」をクリックします
1. **[概要] \| [Log Collection]\(ログ収集\)** ブレードで、 **[Send logs now]\(今すぐログを送信\)** をクリックします。
1. 「**今すぐログを送信**」ブレードで、次のように設定してから「**収集してアップロード**」をクリックします。

    - 開始: 現在の日付と時刻 - 3 時間
    - 終了: 現在の日付と時刻

    >**注**:アップロードが完了するのを待たずに、次の手順に進みます。 ログ収集に失敗します。 これは、ラボ環境から対象の Azure サブスクリプションに直接アクセスできないためであり、予期された結果です。

    >**注**:ここからは、Azure Stack Hub PowerShell を使用して同等の機能を構成します。 これには特権エンドポイントへの接続が必要です。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、管理者として PowerShell ISE を開始します。
1. **[管理者: Windows PowerShell ISE]** コンソールで次のように実行して、特権エンドポイントが実行されているインフラストラクチャ VM の IP アドレスを特定します。

    ```powershell
    $ipAddress = (Resolve-DnsName -Name AzS-ERCS01).IPAddress
    ```

1. **[管理者: Windows PowerShell ISE]** ウィンドウで次のように実行して、特権エンドポイントが実行されているインフラストラクチャ VM の IP アドレスを、WinRM の信頼されたホストの一覧に追加します (すべてのホストが既に許可されているのでない場合)。

    ```powershell
    $trustedHosts = (Get-Item -Path WSMan:\localhost\Client\TrustedHosts).Value
    If ($trustedHosts -ne '*') {
    If ($trustedHosts -ne '') {
        $trustedHosts += ",ipAddress"
    } else {
    $trustedHosts = "$ipAddress"
    }
    }
    Set-Item WSMan:\localhost\Client\TrustedHosts -Value $TrustedHosts -Force
    ```

1. **[管理者: Windows PowerShell ISE]** ウィンドウで次のように実行して、Azure Stack Hub 管理者の資格情報を変数に格納します。

    ```powershell
    $adminUserName = 'CloudAdmin@azurestack.local'
    $adminPassword = 'Pa55w.rd1234' | ConvertTo-SecureString -Force -AsPlainText
    $adminCredentials = New-Object PSCredential($adminUserName,$adminPassword)
    ```

1. **[管理者: Windows PowerShell ISE]** ウィンドウで次のように実行して、特権エンドポイントへの PowerShell リモート処理セッションを確立します。

    ```powershell
    $session = New-PSSession -ComputerName $ipAddress -ConfigurationName PrivilegedEndpoint -Credential $adminCredentials
    Enter-PSSession -Session $session
    ```

    >**注**:PowerShell リモート セッションが正常に確立されたことを確認します。 これで「Windows PowerShell ISE」ウィンドウの「コンソール」ペインに、特権エンドポイントが実行されているインフラストラクチャ VM の IP アドレスから始まるプロンプトが表示されます (角かっこで囲まれています)。

1. **[管理者: Windows PowerShell ISE]** ウィンドウのコンソール ペインの PowerShell リモート処理セッションで次のように実行して、Azure Stack Hub ストレージ診断ログをオンデマンドで送信します。

    ```powershell
    Send-AzureStackDiagnosticLog -FilterByRole Storage
    ```

    >**注**:アップロードが完了するのを待たずに、次の手順に進みます。 対象の Azure サブスクリプションにラボ環境から直接アクセスできない場合、ログ収集が失敗する可能性があります。

1. **[管理者: Windows PowerShell ISE]** ウィンドウのコンソール ペインの PowerShell リモート処理セッションで、**Ctrl** + **C** キーを押してログ収集を終了し、次のように実行して対話型セッションを終了します (切断する必要はありません)。

    ```powershell
    Exit-PSSession
    ```

    >**注**:ロール別にフィルターをかけたり、ログ収集の時間枠を設定したりするオプションも用意されています。 詳細については、「[診断ログの収集](https://docs.microsoft.com/en-us/azure/azure-stack/azure-stack-diagnostics)」を参照してください。


#### <a name="task-3-copy-diagnostic-logs-to-a-local-file-share"></a>タスク 3:診断ログをローカル ファイル共有にコピーする

このタスクでは次のことを行います。

- Azure Stack Hub 管理者ポータルを使用して、診断ログをローカル ファイル共有にコピーする
- Azure Stack Hub PowerShell を使用して、診断ログをローカル ファイル共有にコピーする 

>**注**:まずは、ログを格納するためのファイル共有を作成することから始めます。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、エクスプローラーを開始します。 
1. エクスプローラーを使用して、新しいフォルダー **C:\\AzSHLogs** を作成します。
1. エクスプローラーで「**AzSHLogs**」フォルダーを右クリックし、右クリック メニューで「**プロパティ**」をクリックします。
1. 「**AzSHLogs プロパティ**」ウィンドウで、「**共有**」タブをクリックしてから「**詳細な共有**」をクリックします。
1. 「**詳細な共有**」ダイアログ ボックスで、「**このフォルダーを共有する**」をクリックしてから「**アクセス許可**」をクリックします。
1. 「**AzSHLogs のアクセス許可**」ウィンドウで、「**すべてのユーザー**」エントリが選択されていることを確認してから「**削除**」をクリックします。
1. 「**追加**」をクリックし、「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」ダイアログ ボックスに「**CloudAdmins**」と入力して「**OK**」をクリックします。
1. 「**CloudAdmins**」エントリが選択されていることを確認してから、「**許可**」列の「**フル コントロール**」チェックボックスをクリックします。
1. 「**追加**」をクリックし、「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」ダイアログ ボックスで「**場所**」をクリックします。
1. **[場所]** ダイアログ ボックスで、ローカル コンピューターを表すエントリ (**AzSHOST-1**) をクリックして、 **[OK]** をクリックします。
1. 「**選択するオブジェクト名を入力**」テキスト ボックスに「**Administrators**」と入力して「**OK**」をクリックします。
1. 「**Administrators**」エントリが選択されていることを確認してから、「**許可**」列の「**フル コントロール**」チェックボックスをクリックして、「**OK**」をクリックします。
1. 「**詳細な共有**」ダイアログ ボックスに戻り、「**OK**」をクリックします。
1. 「**AzSHLogs プロパティ**」ウィンドウに戻り、「**セキュリティ**」タブをクリックしてから「**編集**」をクリックします。
1. 「**追加**」をクリックし、「**ユーザー、コンピューター、サービス アカウント、またはグループの選択**」ダイアログ ボックスに「**CloudAdmins**」と入力して「**OK**」をクリックします。
1. 「**AzSHLogs のアクセス許可**」ダイアログ ボックスで、「**グループまたはユーザー名**」ペインの「**CloudAdmins**」をクリックし、「**CloudAdmins のアクセス許可**」ペインの「**許可**」列で「**フル コントロール**」をクリックしてから、「**OK**」をクリックします。
1. 「**AzSHLogs プロパティ**」ウィンドウに戻り、「**閉じる**」をクリックします。

    >**注**:これで、Azure Stack Hub 管理者ポータルまたは特権エンドポイントのいずれかを介して、診断ログ収集をファイル共有に構成できるようになります。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーのウィンドウに [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示し、ハブ メニューの **[ヘルプとサポート]** をクリックします。
1. 「**概要**」ブレードで「**ログの収集**」をクリックします
1. **[概要] \| [Log Collection]\(ログ収集\)** ブレードで、 **[設定]** をクリックします。
1. 「**設定**」ブレードで、「**ログの場所**」オプションを「**Azure (推奨)** 」から「**ローカル ファイル共有**」に変更し、以下のように設定します。

    - SMB ファイル共有パス: **\\\\AzSHOST-1.azurestack.local\\AzSHLogs**
    - ユーザー名:**AZURESTACK\\AzureStackAdmin**
    - パスワード: **Pa55w.rd1234**

1. 「**設定**」ブレードで「**保存**」をクリックします。
1. **[概要] \| [Log Collection]\(ログ収集\)** ブレードに戻り、 **[Send logs now]\(今すぐログを送信\)** をクリックします。
1. 「**今すぐログを送信**」ブレードで、次のように設定してから「**収集してアップロード**」をクリックします。

    - 開始: 現在の日付と時刻 - 3 時間
    - 終了: 現在の日付と時刻

    >**注**:ログの収集とアップロードの進行を確認するには、 **[概要] \| [Log Collection]\(ログ収集\)** ブレードで **[最新の情報に更新]** をクリックします。

    >**注**:アップロードが完了するのを待たずに、次の手順に進みます。 これでログは正常に収集されますが、完了するまで 15 分ほどかかる場合があります。 

    >**注**:ここからは、Azure Stack Hub PowerShell を使用して同等の機能を構成します。 これには特権エンドポイントへの接続が必要です。

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell ISE]** コンソール (前のタスクで特権エンドポイントへの PowerShell リモート処理セッションを開始したもの) に戻ります。
1. **[管理者: Windows PowerShell ISE]** ウィンドウのコンソール ペインで次のように実行して、Azure Stack Hub ストレージ診断ログをローカル ファイル共有にコピーします。

    ```powershell
    Invoke-Command -Session $session { Get-AzureStackLog -OutputSharePath '\\AzSHOST-1.azurestack.local\AzSHLogs' -OutputShareCredential $using:adminCredentials -FilterByRole Storage}
    ```

1. コマンドレットが完了するまで待ってから、エクスプローラーで **C:\\AzSHLogs** フォルダーの内容を調べて、ログがアップロードされていることを確認します。

    >**注**:このフォルダーには、上記で行った個々のコピーに対応するフォルダーが含まれているはずです。 フォルダーの名前は **AzureStackLogs-*YYYYMMDDHHMMSS*-AZS-ERCS01** という形式であり、**_YYYYMMDDHHMMSS_** はコピーのタイムスタンプを表します。

1. **[管理者: Windows PowerShell]** ウィンドウの PowerShell リモート処理セッションのプロンプトで次のように実行して、セッションを閉じます。

    ```powershell
    Enter-PSSession -$session
    Close-PrivilegedEndpoint -TranscriptsPathDestination '\\AzSHOST-1.azurestack.local\AzSHLogs' -Credential $using:adminCredentials
    ```

1. コマンドレットが完了するまで待ってから、エクスプローラーで **C:\\AzSHLogs** フォルダーの内容を調べて、セッション トランスクリプトがアップロードされていることを確認します。

>**レビュー**:この演習では特権エンドポイントへの PowerShell リモート セッションを確立し、そのエンドポイントで利用できる PowerShell コマンドレットを使用して診断ログを収集しました。