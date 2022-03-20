---
lab:
  title: ラボ:SQL Server リソース プロバイダーを Azure Stack Hub に実装する
  module: 'Module 2: Provide Services'
ms.openlocfilehash: 76aa2946e5adb7bc82368595813707e49c85088a
ms.sourcegitcommit: f99a365741ad02181da3b4fc4a921d5437810bc2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/02/2022
ms.locfileid: "139252340"
---
# <a name="lab---implement-sql-server-resource-provider-in-azure-stack-hub"></a>ラボ - SQL Server リソース プロバイダーを Azure Stack Hub に実装する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- なし

## <a name="estimated-time"></a>推定所要時間

150 分

## <a name="lab-scenario"></a>ラボのシナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、 テナントが SQL Server データベースをデプロイできるようにする必要があります。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- SQL Server リソース プロバイダーを Azure Stack Hub に実装する

## <a name="lab-environment"></a>ラボ環境 

このラボでは、Active Directory フェデレーション サービス (AD FS) (ID プロバイダーとしてバックアップされた Active Directory) に統合された ADSK インスタンスを使用します。 

ラボ環境は次のように構成されています。

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

このラボでは、PowerShell を介して Azure Stack Hub を管理するために必要なソフトウェアをインストールします。 


### <a name="exercise-1-install-the-sql-server-resource-provider-in-azure-stack-hub"></a>演習 1:SQL Server リソース プロバイダーを Azure Stack Hub にインストールする

この演習では、SQL Server リソース プロバイダーを Azure Stack Hub にインストールします。

1. SQL Server リソース プロバイダー バイナリをダウンロードする 
1. SQL Server リソース プロバイダーをインストールする
1. SQL Server リソース プロバイダーのインストールを確認する

>**注**:演習の所要時間を可能な限り短縮するため、Azure Stack Hub SQL Server リソース プロバイダーのインストールに必要な、以下をはじめとする一部のタスクはすでに完了した状態となっています。

- Azure Marketplace シンジケーションを実装する
- **Microsoft AzureStack Add-On RP Windows Server** を Azure Marketplace からダウンロードする

#### <a name="task-1-download-sql-server-resource-provider-binaries"></a>タスク 1:SQL Server リソース プロバイダー バイナリをダウンロードする

このタスクでは次のことを行います。

- SQL Server リソース プロバイダー バイナリをダウンロードする

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザー ウィンドウを開いて [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示し、CloudAdmin@azurestack.local としてサインインします。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で、ハブ メニューの「**すべてのサービス**」をクリックします。
1. 「**すべてのサービス**」ブレードで、「**Marketplace Management**」を検索して選択します。
1. 「Marketplace Management」ブレードで、利用可能なサービスの一覧に **Microsoft AzureStack Add-On RP Windows Server**」が表示されていることを確認します。
1. **AzSHOST-1** へのリモート デスクトップ セッション内で、別の Web ブラウザー ウィンドウを開き、SQL リソース プロバイダーの自己展開型の実行可能ファイルを (https://aka.ms/azshsqlrp11931) からダウンロードし、そのコンテンツを **C:\\ Downloads\\ SQLRP** フォルダーに展開します (最初にフォルダーを作成する必要があります)。


#### <a name="task-2-install-the-sql-server-resource-provider"></a>タスク 2:SQL Server リソース プロバイダーをインストールする

このタスクでは次のことを行います。

- SQL Server リソース プロバイダーをインストールする

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、管理者として Windows PowerShell を起動します。

    > **注:**  必ず新しい PowerShell セッションを起動してください。

1. **[管理者:Windows PowerShell]** プロンプトから以下を実行して、PowerShell ギャラリーを信頼されたレポジトリとして構成します

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. **[管理者:Windows PowerShell]** プロンプトから以下を実行して、SQL Server リソース プロバイダーに必要な AzureRM.Bootstrapper モジュールのバージョンをインストールします。

    ```powershell
    Get-Module -Name Azs.* -ListAvailable | Uninstall-Module -Force -Verbose
    Get-Module -Name Azure* -ListAvailable | Uninstall-Module -Force -Verbose

    Install-Module -Name Az.BootStrapper -Force
    Install-AzProfile -Profile 2020-09-01-hybrid -Force
    ```

1. **[管理者:Windows PowerShell]** プロンプトで以下を実行して、Azure Stack Hub オペレーターの PowerShell 環境を登録します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
      -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
      -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```

1. **[管理者:Windows PowerShell]** プロンプトで以下を実行して、現在の環境を設定します。

    ```powershell
    Set-AzEnvironment -Name 'AzureStackAdmin'
    ```

1. **[管理者:Windows PowerShell]** プロンプトで以下を実行して、現在の環境に対して認証を行います (サインインを求められたら、 **CloudAdmin@azurestack.local** ユーザーとしてサインインし、パスワードに **Pa55w.rd1234** と入力します)。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. **[管理者:Windows PowerShell]** プロンプトで以下を実行し、認証に成功したことと、対応するコンテキストが設定されたことを確認します。

    ```powershell
    Get-AzContext
    ```

1. **[管理者:Windows PowerShell]** プロンプトで以下を実行して、SQL Server リソース プロバイダーのインストールに必要な変数を設定します。

    ```powershell
    $domain = 'azurestack.local'
    $privilegedEndpoint = 'AzS-ERCS01'
    $downloadDir = 'C:\Downloads\SQLRP'

    # Set the AzureStack\AzureStackAdmin credentials
    $serviceAdmin = 'AzureStackAdmin@azurestack.local'
    $serviceAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $serviceAdminCreds = New-Object System.Management.Automation.PSCredential ($serviceAdmin, $serviceAdminPass)

    # Set the AzureStack\CloudAdmin credentials
    $cloudAdminName = 'AzureStack\CloudAdmin'
    $cloudAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $cloudAdminCreds = New-Object PSCredential($cloudAdminName, $cloudAdminPass)

    # Set credentials for the new resource provider VM local admin account
    $vmLocalAdminPass = ConvertTo-SecureString 'Pa55w.rd1234' -AsPlainText -Force
    $vmLocalAdminCreds = New-Object System.Management.Automation.PSCredential ('sqlrpadmin', $vmLocalAdminPass)

    # Set a password that will protect the private key of a self-signed certificate generated to secure the SQL Server resource provider
    $pfxPass = ConvertTo-SecureString 'Pa55w.rd1234pfx' -AsPlainText -Force

    # Update the PowerShell module path environment variable to include the SQL Server resource provider modules
    $rpModulePath = Join-Path -Path $env:ProgramFiles -ChildPath 'SqlMySqlPsh'
    $env:PSModulePath = $env:PSModulePath + ';' + $rpModulePath 
    ```

1. **[管理者:Windows PowerShell]** プロンプトで、現在のディレクトリを、展開した SQL Server リソース プロバイダーのインストール ファイルの場所に変更し、DeploySQLProvider.ps1 スクリプトを実行します。

    ```powershell
    Set-Location -Path 'C:\Downloads\SQLRP'

    .\DeploySQLProvider.ps1 `
        -AzCredential $serviceAdminCreds `
        -VMLocalCredential $vmLocalAdminCreds `
        -CloudAdminCredential $cloudAdminCreds `
        -PrivilegedEndpoint $privilegedEndpoint `
        -DefaultSSLCertificatePassword $pfxPass
    ```

    > **注**: インストールが完了するまで待ちます。 1 時間ほどかかる場合があります。

#### <a name="task-3-verify-installation-of-the-sql-server-resource-provider"></a>タスク 3:SQL Server リソース プロバイダーのインストールを確認する

このタスクでは次のことを行います。

- SQL Server リソース プロバイダーのインストールを確認する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Azure Stack 管理者ポータルが表示されている Web ブラウザーの画面に切り替え、ハブ メニューの「**リソース グループ**」をクリックします。 
1. 「**リソース グループ**」ブレードで「**system.local.sqladapter**」をクリックします。
1. 「**system.local.sqladapter**」ブレードで「**デプロイ**」エントリを確認し、すべてのデプロイに成功したことを検証します。
1. Azure Stack 管理者ポータルで「**仮想マシン**」に移動し、SQL リソース プロバイダーが正常に作成され、実行されていることを確認します。
1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザー ウィンドウを開いて [Azure Stack Hub テナント ポータル](https://portal.local.azurestack.external/)を表示させ、サインインを求められたら CloudAdmin@azurestack.local としてサインインします。
1. Azure Stack Hub テナント ポータルのハブ メニューで、「**リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで「**データとストレージ**」を選択し、利用可能なリソースの種類の一覧に「**SQL データベース**」が表示されることを確認します。

>**レビュー**:この演習では、SQL Server リソース プロバイダーを Azure Stack Hub にインストールしました。


### <a name="exercise-2-configure-sql-server-resource-provider-in-azure-stack-hub"></a>演習 2:SQL Server リソース プロバイダーを Azure Stack Hub で構成する

この演習では、SQL Server リソース プロバイダーを Azure Stack Hub で構成します。

1. (クラウド オペレーターとして) SQL Server ホスティング サーバー用のプラン、オファー、サブスクリプションを作成する
1. (クラウド オペレーターとして) SQL Server ホスティング サーバーの役割を担う Azure Stack Hub VM をデプロイする
1. (クラウド オペレーターとして) SQL ホスティング サーバーを追加する
1. (クラウド オペレーターとして) ユーザーが SQL データベースを使用できるようにする
1. (ユーザーとして) SQL データベースを作成する

>**注**:演習の所要時間を可能な限り短縮するため、Azure Stack Hub SQL Server リソース プロバイダーの構成を促進することを目的とした、以下をはじめとする一部のタスクはすでに完了しています。

- Azure Marketplace の SQL Server イメージをダウンロードする
- Azure Marketplace から **Sql IaaS VM 拡張機能** をダウンロードする


#### <a name="task-1-create-a-plan-offer-and-subscription-for-a-sql-server-hosting-server-as-a-cloud-operator"></a>タスク 1:(クラウド オペレーターとして) SQL Server ホスティング サーバー用のプラン、オファー、サブスクリプションを作成する

このタスクでは次のことを行います。

- (クラウド オペレーターとして) SQL Server ホスティング サーバー用のプラン、オファー、サブスクリプションを作成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザー ウィンドウを開いて [Azure Stack Hub 管理者ポータル](https://adminportal.local.azurestack.external/)を表示し、CloudAdmin@azurestack.local としてサインインします。
1. Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「 **+ リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで、「**オファーとプラン**」をクリックしてから「**プラン**」をクリックします。
1. 「**新しいプラン**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-hosting-plan1**
    - リソース名: **sql-server-hosting-plan1**
    - リソース グループ: 新しいリソース グループの名前 **sql-server-hosting-plans-RG**

1. **[次へ: サービス >]** をクリックします。
1. 「**新しいプラン**」ブレードの「**サービス**」タブで、「**Microsoft.Compute**」、「**Microsoft.Storage**」、「**Microsoft.Network**」のチェックボックスをオンにします。
1. **[次へ: クォータ >]** をクリックします。
1. 「**新しいプラン**」ブレードの「**クォータ**」タブで、次のように設定を行います。

    - Microsoft.Compute:**既定のクォータ**
    - Microsoft.Network:**既定のクォータ**
    - Microsoft.Storage:**既定のクォータ**

1. **[Review + create]\(レビュー + 作成\)** をクリックし、 **[作成]** をクリックします。

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**新規作成**」ブレードに戻って「**オファー**」クリックします。
1. 「**新しいオファーの作成**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-hosting-offer1**
    - リソース名: **sql-server-hosting-offer1**
    - リソース グループ: **sql-server-hosting-offers-RG**
    - このオファーをパブリックに設定する:"**いいえ**"

1. **[次へ: 基本プラン >]** をクリックします。 
1. 「**新しいオファーの作成**」ブレードの「**基本プラン**」タブで、「**sql-server-hosting-plan1**」エントリの横にあるチェックボックスをオンにします。
1. **[次へ: アドオン プラン >]** をクリックします。
1. 「**アドオン プラン**」の設定は既定値にしたまま、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザー ウィンドウに Azure Stack Hub 管理者ポータルが表示されている状態で、 **[新規作成]** ブレードに戻り、 **[オファー + プラン]** セクションで **[サブスクリプション]** をクリックします。
1. 「**ユーザー サブスクリプションの作成**」ブレードで、次のように設定してから「**作成**」を選択します。

    - 名前: **sql-server-hosting-subscription1**
    - ユーザー: **cloudadmin@azurestack.local**
    - ディレクトリ テナント:**ADFS.azurestack.local**
    - オファー名: **sql-server-hosting-offer1**

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。

1. Azure Stack Hub 管理者ポータルのウィンドウは開いたままにします。


#### <a name="task-2-deploy-an-azure-stack-hub-vm-that-will-become-a-sql-server-hosting-server-as-a-cloud-operator"></a>タスク 2:(クラウド オペレーターとして) SQL Server ホスティング サーバーの役割を担う Azure Stack Hub VM をデプロイする

このタスクでは次のことを行います。

- (クラウド オペレーターとして) SQL Server ホスティング サーバーの役割を担う Azure Stack Hub VM をデプロイする

    >**注**:請求可能なユーザー サブスクリプションに、SQL Server ホスティング サーバーとして機能する Azure Stack Hub VM を作成する必要があります。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、[Azure Stack Hub テナント ポータル](https://portal.local.azurestack.external/)が表示されている Web ブラウザーの画面に切り替えます。
1. Azure Stack Hub テナント ポータルのハブ メニューで、「**リソースの作成**」をクリックします。
1. **[新規作成]** ブレードで **[コンピューティング]** を選択し、利用可能なリソースの種類の一覧で **{WS-BYOL} 無料の SQL Server ライセンス:Windows Server 2016 上の SQL Server 2017 Express** を選択します。
1. 「**仮想マシンの作成**」ブレードの「**基本**」ペインで、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - 名前: **sql-host-vm0**
    - VM ディスクの種類:**Premium SSD**
    - ユーザー名: **sqladmin**
    - パスワード: **Pa55w.rd1234**
    - サブスクリプション: **sql-server-hosting-subscription1**
    - リソース グループ: 新しいリソース グループの名前 **sql-server-hosting-RG**
    - 場所: **ローカル**

1. 「**サイズの選択**」ブレードで、「**DS1_v2**」を選択してから「**選択**」をクリックします。
1. 「**仮想マシンの作成**」ブレードの「**設定**」ペインで、「**ネットワーク セキュリティ グループ**」を「**詳細**」に設定してから、「**ネットワーク セキュリティ グループ (ファイアウォール)** 」をクリックします。
1. 「**ネットワーク セキュリティ グループの作成**」ブレードで、「 **+ 受信規則の追加**」をクリックします。
1. 「**受信セキュリティ規則の追加**」ブレードで次のように設定してから「**追加**」を選択します (他の設定は既定値のままにします)。

    - 宛先ポート範囲:**1433**
    - プロトコル: **TCP**
    - アクション: **許可**
    - 名前: **SQL**

1. 「**ネットワーク セキュリティ グループの作成**」ブレードに戻って「**OK**」をクリックします。
1. 「**仮想マシンの作成**」ブレードの「**設定**」タブに戻り、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - ブート診断:Disabled (無効)
    - ゲスト OS の診断:Disabled (無効)

1. 「**仮想マシンの作成**」ブレードの「**SQL Server の設定**」タブで、次のように設定してから「**OK**」をクリックします (他の設定は規定値のままにします)。

    - SQL への接続:**パブリック (インターネット)**
    - ポート:**1433**
    - SQL 認証:**有効にする**
    - ログイン名:**SQLAdmin**
    - パスワード: **Pa55w.rd**
    - ストレージの構成:**全般**
    - 自動修正:**無効化**
    - 自動バックアップ:**無効化**
    - Azure Key Vault の統合:**無効化**

1. 「**仮想マシンの作成**」ブレードの「**概要**」ペインで、「**OK**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。 20 分ほどかかる場合があります。

1. デプロイが完了したら、**sql-host-vm0** 仮想マシン ブレードに移動し、 **[概要]** セクションの **[DNS 名]** ラベルの真下にある **[構成]** をクリックします。
1. **[sql-host-vm0-ip \| 構成]** ブレードの **[DNS 名ラベル (オプション)]** テキスト ボックスに、「**sql-host-vm0**」と入力して **[保存]** をクリックします。

    >**注**:これにより、「**sql-host-vm0.local.cloudapp.azurestack.external**」DNS 名を介して「**sql-host-vm0**」が利用できるようになります。

1. **[sql-host-vm0-ip \| 構成]** ブレードで、 **[割り当て]** オプションを **[静的]** に設定して **[保存]** をクリックします。

    >**注**:これにより、「**sql-host-vm0**」仮想マシンの再起動がトリガーされます。


#### <a name="task-3-add-a-sql-hosting-server-as-a-cloud-operator"></a>タスク 3:(クラウド オペレーターとして) SQL ホスティング サーバーを追加する

このタスクでは次のことを行います。

- (クラウド オペレーターとして) SQL ホスティング サーバーを追加する

1. **AzSHOST-1** へのリモート デスクトップ セッション内で、Web ブラウザーに Azure Stack 管理者ポータルが表示されている状態で「**すべてのサービス**」をクリックし、「**管理リソース**」セクションで「**SQL ホスティング サーバー**」をクリックします。

    > **注:**  **SQL Hosting Servers** リソースの種類を表示させるには、Azure Stack 管理者ポータルが表示されているブラウザー ページを更新する必要があります。

1. 「**SQL ホスティング サーバー**」ブレードで「 **+ 追加**」をクリックします。
1. 「**SQL ホスティング サーバーの追加**」ブレードで、次のように設定を行います。

    - SQL サーバー名: **sql-host-vm0.local.cloudapp.azurestack.external**
    - ユーザー名: **sqladmin**
    - パスワード: **Pa55w.rd1234**
    - ホスティング サーバーのサイズ (GB):**50**
    - 常時接続可用性グループ: オフ
    - [サブスクリプション]:**既定のプロバイダー サブスクリプション**
    - リソース グループ: 新しいリソース グループの名前 **sql.resources-RG**
    - 場所: **ローカル**

1. 「**SQL ホスティング サーバーの追加**」ブレードで「**SKU**」をクリックし、「**SKU**」ブレードで「**新しい SKU の作成**」をクリックしてから、「**SKU の作成**」ブレードで以下のように設定を行います。

    - 名前: **MSSQL2017Exp**
    - ファミリ:**SQL Server 2017**
    - サービス レベル:**スタンドアロン**
    - エディション:**Express**

1. 「**SKU の作成**」ブレードで「**OK**」をクリックし、「**SQL ホスティング サーバー**」ブレードに戻って「**作成**」をクリックします。

    > **注:**  操作が完了するまで待ちます。 これにかかる時間は 1 分未満です。

1. 「**SQL ホスティング サーバー**」ブレードで「**更新**」をクリックし、サーバーの一覧に「**sqlhost1.local.cloudapp.azurestack.external**」が表示されていることを確認します。


#### <a name="task-4-make-sql-databases-available-to-users-as-a-cloud-operator"></a>タスク 4:(クラウド オペレーターとして) ユーザーが SQL データベースを使用できるようにする

このタスクでは次のことを行います。

- (クラウド オペレーターとして) ユーザーが SQL データベースを使用できるようにする

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「 **+ リソースの作成**」をクリックします。
1. 「**新規作成**」ブレードで、「**オファーとプラン**」をクリックしてから「**プラン**」をクリックします。
1. 「**新しいプラン**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-2017-express-db-plan1**
    - リソース名: **sql-server-2017-express-db-plan1**
    - リソース グループ: 新しいリソース グループの名前 **sqldb-plans-RG**

1. **[次へ: サービス >]** をクリックします。
1. 「**新しいプラン**」ブレードの「**サービス**」タブで、「**Microsoft.SQLAdapter**」チェックボックスをオンにします。
1. **[次へ: クォータ >]** をクリックします。
1. 「**新しいプラン**」ブレードの「**クォータ**」タブで、「**Microsoft.SQLAdapter**」エントリの横にある「**新規作成**」をクリックします。
1. * *[クォータの作成]* ブレードで、次のように設定してから **[作成]** をクリックします。

    - クォータ名: **sql-server-2017-express-db-quota1**
    - データベースの最大サイズ (GB):**2**
    - データベースの最大数:**20**

1. **[Review + create]\(レビュー + 作成\)** をクリックし、 **[作成]** をクリックします。

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの画面に Azure Stack Hub 管理者ポータルが表示されている状態で、「**新規作成**」ブレードに戻って「**オファー**」クリックします。
1. 「**新しいオファーの作成**」ブレードの「**基本**」タブで、次のように設定を行います。

    - 表示名: **sql-server-2017-express-db-offer1**
    - リソース名: **sql-server-2017-express-db-offer1**
    - リソース グループ: **sqldb-offers-RG**
    - このオファーをパブリックに設定する:**はい**

1. **[次へ: 基本プラン >]** をクリックします。 
1. 「**新しいオファーの作成**」ブレードの「**基本プラン**」タブで、「**sql-server-2017-express-db-plan1**」エントリの横にあるチェックボックスをオンにします。
1. **[次へ: アドオン プラン >]** をクリックします。
1. 「**アドオン プラン**」の設定は既定値にしたまま、「**確認して作成**」をクリックしてから「**作成**」をクリックします。

    >**注**: デプロイが完了するまで待ちます。 通常は数秒で完了します。


#### <a name="task-5-crate-a-sql-database-as-a-user"></a>タスク 5:(ユーザーとして) SQL データベースを作成する

このタスクでは次のことを行います。

- テスト用のユーザー アカウントを作成する
- 新たに作成されたユーザー アカウントを使用して SQL データベースを作成する

1. **AzS-HOST1** へのリモート デスクトップ セッション内でスタート メニューから「**スタート**」をクリックし、「**Windows 管理ツール**」をクリックしてから、管理ツールの一覧で「**Active Directory 管理センター**」をダブルクリックします。
1. 「**Active Directory 管理センター**」コンソールで「**azurestack (ローカル)** 」をクリックします。
1. 詳細ペインで「**ユーザー**」コンテナーをダブルクリックします。
1. 「**タスク**」ペインの「**ユーザー**」セクションで、 **「新規作成」>「ユーザー」** の順にクリックします。
1. 「**ユーザーの作成**」ウィンドウで、次のように設定してから「**OK**」をクリックします。 

    - 完全名:**T1U1**
    - ユーザー UPN ログオン: **t1u1@azurestack.local**
    - ユーザー SamAccountName: **azurestack\t1u1**
    - パスワード: **Pa55w.rd**
    - パスワード オプション: **[その他のパスワード オプション] -> [パスワードを無期限にする]**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーの InPrivate セッションを開始します。
1. Web ブラウザー ウィンドウで [Azure Stack Hub ユーザー ポータル](https://portal.local.azurestack.external)に移動し、 **t1u1@azurestack.local** とパスワード **Pa55w.rd** でサインインします。
1. Azure Stack Hub ユーザー ポータルのダッシュボードで、「**サブスクリプションの取得**」タイルをクリックします。
1. 「**サブスクリプションの取得**」ブレードの「**名前**」テキスト ボックスに「**t1u1-sqldb-subscription1**」と入力します。
1. オファーの一覧で「**sql-server-2017-express-db-offer1**」を選択し、「**作成**」をクリックします。
1. メッセージ "**サブスクリプションが作成されました。サブスクリプションの使用を開始するには、ポータルを更新する必要があります**" が表示されたら、 **[最新の情報に更新]** をクリックします。 
1. Azure Stack Hub テナント パネルのハブ メニューで、「**すべてのサービス**」をクリックします。
1. サービスの一覧で「**SQL データベース**」をクリックします。
1. 「**SQL データベース**」ブレードで「 **+ 追加**」をクリックします。
1. 「**データベースの作成**」ブレードで、次のように設定を行います。

    - データベース名: **sqldb1**
    - 照合順序:**SQL_Latin1_General_CP1_CI_AS**
    - 最大サイズ (MB):**200**
    - サブスクリプション: **t1u1-sqldb-subscription1**
    - リソース グループ: 新しいリソース グループの名前 **sqldb-RG**
    - 場所: **ローカル**
    - SKU:**MSSQL2017Exp**

    >**注**:新たに作成した SKU がテナント ポータルで利用できるようになるまで、しばらく待たなければならない場合があります。

1. 「**データベースの作成**」ブレードで「**ログイン**」をクリックします。
1. 「**ログインの選択**」ブレードで「**新しいログインの作成**」をクリックします。
1. 「**新しいログイン**」ブレードで、以下のように設定してから「**OK**」をクリックします。

    - データベース ログイン: **dbAdmin**
    - パスワード: **Pa55w.rd**

1. 「**データベースの作成**」ブレードに戻って「**作成**」をクリックします。

>**注**: デプロイが完了するまで待ちます。 これに要する時間は 1 分未満です。 

>**レビュー**:この演習では、SQL Server ホスティング サーバーを Azure Stack Hub に追加し、これをテナントが利用できるよう設定したほか、SQL データベースをテナント ユーザーとしてデプロイしました。