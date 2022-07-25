---
lab:
  title: ラボ:Azure Stack Hub で Azure Resource Manager (ARM) テンプレートを検証する
  module: 'Module 2: Provide Services'
ms.openlocfilehash: ac5c390449e293632dba9858773d5a50f98fb23c
ms.sourcegitcommit: 3ce6441f824c1ac2b22159d6830eba55dba5ba66
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/01/2022
ms.locfileid: "139251646"
---
# <a name="lab---validate-azure-resource-manager-arm-templates-with-azure-stack-hub"></a>ラボ - Azure Stack Hub で Azure Resource Manager (ARM) テンプレートを検証する
# <a name="student-lab-manual"></a>受講生用ラボ マニュアル

## <a name="lab-dependencies"></a>ラボの依存関係

- なし

## <a name="estimated-time"></a>推定所要時間

30 分

## <a name="lab-scenario"></a>ラボのシナリオ

あなたは Azure Stack Hub 環境のオペレーターとして、 既存の Azure Resource Manager (ARM) テンプレートを使用して、Azure Stack Hub リソースのデプロイを自動化する必要があります。 

## <a name="objectives"></a>目標

このラボを完了すると、次のことができるようになります。

- Azure Stack Hub デプロイ用の ARM テンプレートを検証する

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


### <a name="exercise-1-validate-an-arm-template-with-azure-stack-hub"></a>演習 1:Azure Stack Hub を使用して ARM テンプレートを検証する

この演習では、Azure Stack Hub を使用して ARM テンプレートを検証します。

1. クラウド機能ファイルを構築する 
1. テンプレート検証の成功例を実行する
1. テンプレート検証の失敗例を実行する
1. テンプレートに関する問題を修正する


#### <a name="task-1-build-a-cloud-capabilities-file"></a>タスク 1:クラウド機能ファイルを構築する

このタスクでは次のことを行います。

- クラウド機能ファイルを構築する

1. 必要であれば、次の資格情報を使用して **AzS-HOST1** にサインインします。

    - ユーザー名: **AzureStackAdmin@azurestack.local**
    - パスワード: **Pa55w.rd1234**

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として PowerShell を開始します。
1. **[管理者: Windows PowerShell]** プロンプトから以下を実行して、PowerShell ギャラリーを信頼されたレポジトリとして構成します

    ```powershell
    Set-PSRepository -Name 'PSGallery' -InstallationPolicy Trusted
    ```

1. **[管理者: Windows PowerShell]** プロンプトから次のように実行して、PowerShellGet をインストールします。

    ```powershell
    Install-Module PowerShellGet -MinimumVersion 2.2.3 -Force
    ```

    >**注**:使用中のモジュールに関する警告メッセージはすべて無視してください。

1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、Azure Stack Hub 用の PowerShell Az モジュールをインストールします。

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、管理者として Windows PowerShell を開始します。
1. **AzS-HOST1** へのリモート デスクトップ セッション内で、 **[管理者: Windows PowerShell]** プロンプトから次のように実行して、このラボに必要な Azure Stack Hub PowerShell モジュールをインストールします。 

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Install-Module -Name Az.BootStrapper -Force
    Install-AzProfile -Profile 2020-09-01-hybrid -Force
    Install-Module -Name AzureStack -RequiredVersion 2.2.0 
    ```

    >**注**:すでに利用可能となっているコマンドに関するエラー メッセージはすべて無視してください。

1. **[管理者: Windows PowerShell]** ウィンドウで以下を実行して、Azure Stack Hub オペレーターの PowerShell 環境を登録します。

    ```powershell
    Add-AzEnvironment -Name 'AzureStackAdmin' -ArmEndpoint 'https://adminmanagement.local.azurestack.external' `
       -AzureKeyVaultDnsSuffix adminvault.local.azurestack.external `
       -AzureKeyVaultServiceEndpointResourceId https://adminvault.local.azurestack.external
    ```   

1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、新しく登録した **AzureStackAdmin** 環境にサインインします。

    ```powershell
    Connect-AzAccount -EnvironmentName 'AzureStackAdmin'
    ```

1. メッセージが表示されたら、 **CloudAdmin@azurestack.local** アカウントとパスワード **Pa55w.rd1234** を使って認証を行います。
1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、Azure Stack ツールをダウンロードします。

    ```powershell
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
    Set-Location -Path 'C:\'
    Invoke-WebRequest https://github.com/Azure/AzureStack-Tools/archive/az.zip -OutFile az.zip
    Expand-Archive az.zip -DestinationPath . -Force
    Set-Location -Path '\AzureStack-Tools-az'
    ```

    >**注**:この手順により、Azure Stack Hub ツールをホストする GitHub リポジトリが含まれるアーカイブがローカル コンピューターにコピーされ、アーカイブが **C:\\AzureStack-Tools-master** フォルダーに展開されます。 このツールには PowerShell モジュールが含まれています。このモジュールによって、Azure Stack Hub Resource Manager テンプレートの検証をはじめとする、さまざまな機能が提供されます。 

1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、AzureRM.CloudCapabilities PowerShell モジュールをインポートします。

    ```powershell
    Import-Module .\CloudCapabilities\Az.CloudCapabilities.psm1 -Force
    ```

1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、クラウド機能の JSON ファイルを生成します。 

    ```powershell
    $path = 'C:\Templates'
    New-Item -Path $path -ItemType Directory -Force
    Get-AzCloudCapability -Location 'local' -OutputPath $path\AzureCloudCapabilities.Json -Verbose
    ```

1. **AzS-HOST1** へのリモート デスクトップ セッション内でエクスプローラーを開始し、**C:\\Templates** フォルダーに移動して、**AzureCloudCapabilities.Json** ファイルが正常に作成されたことを確認します。

#### <a name="task-2-run-a-successful-template-validation"></a>タスク 2:テンプレート検証の成功例を実行する

このタスクでは次のことを行います。

- Azure Stack Hub Quickstart テンプレートに対してテンプレート検証ツールを実行する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、Web ブラウザーを起動して Azure Stack Hub QuickStart Templates リポジトリの「[**AzureStack 用 Windows 上の MySql Server**](https://github.com/Azure/AzureStack-QuickStart-Templates/tree/master/mysql-standalone-server-windows)」ページに移動します。 
1. **[AzureStack 用 Windows 上の MySql Server]** ページで **[azuredeploy.json]** をクリックします。
1. [[AzureStack-QuickStart-Templates/mysql-standalone-server-windows/azuredeploy.json]](https://github.com/Azure/AzureStack-QuickStart-Templates/blob/master/mysql-standalone-server-windows/azuredeploy.json) ページで、テンプレートの内容を確認します。
1. **[管理者: Windows PowerShell]** ウィンドウに切り替え、次のように実行して azuredeploy.json ファイルをダウンロードし、**sampletemplate1.json** という名前で **C:\\Templates** フォルダーに保存します。

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/AzureStack-QuickStart-Templates/master/mysql-standalone-server-windows/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate1.json
    ```

1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、AzureRM.TemplateValidator PowerShell モジュールをインポートします。

    ```powershell
    Set-Location -Path 'C:\AzureStack-Tools-az\TemplateValidator'
    Import-Module .\AzureRM.TemplateValidator.psm1 -Force
    ```
  
1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、テンプレートを検証します。

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate1.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate1validationreport.html `
        -Verbose
    ```

1. 検証の出力に目を通し、問題がないことを確認します。

    >**注**:出力は次の形式でなければなりません。

    ```
    Validation Summary:
        Passed: 1
        NotSupported: 0
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate1validationreport.html
    ```

#### <a name="task-3-run-a-failed-template-validation"></a>タスク 3:テンプレート検証の失敗例を実行する

このタスクでは次のことを行います。

- Azure Quickstart テンプレートに対してテンプレート検証ツールを実行する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、AzureStack QuickStart テンプレート レポジトリが表示されている Web ブラウザーから、Azure QuickStart テンプレート レポジトリの「[**Ubuntu VM 上の MySQL Server 5.6**](https://github.com/Azure/azure-quickstart-templates/tree/master/application-workloads/mysql/mysql-standalone-server-ubuntu)」ページに移動します。
1. **[Ubuntu VM 上の MySQL Server 5.6]** ページで **[azuredeploy.json]** をクリックします。
1. [[azure-quickstart-templates/mysql-standalone-server-ubuntu/azuredeploy.json]](https://github.com/Azure/azure-quickstart-templates/blob/master/application-workloads/mysql/mysql-standalone-server-ubuntu/azuredeploy.json) ページで、テンプレートの内容を確認します。
1. **[管理者: Windows PowerShell]** ウィンドウに切り替え、次のように実行して azuredeploy.json ファイルをダウンロードし、**sampletemplate2.json** という名前で **C:\\Templates** フォルダーに保存します。

    ```powershell
    Invoke-WebRequest -Uri 'https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/application-workloads/mysql/mysql-standalone-server-ubuntu/azuredeploy.json' -UseBasicParsing -OutFile $path\sampletemplate2.json
    ```

1. **AzS-HOST1** へのリモート デスクトップ セッション内の Web ブラウザーで、[Virtual Machines](https://docs.microsoft.com/en-us/rest/api/compute/virtualmachines) 用の REST API リファレンスに移動し、Azure API が最新バージョンであることを確認します (これを作成している時点では **2020-12-01**)。 
1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、メモ帳で **sampletemplate2.json** ファイルを開きます。

    ```powershell
    notepad.exe $path\sampletemplate2.json
    ```

1. メモ帳ウィンドウに表示された **sampletemplate2.json** ファイルの内容で、テンプレートの `resources` セクションから仮想マシン リソースを表すセクションを探します。次のような形式になっています。

    ```json
    {
      "apiVersion": "2017-03-30",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

1. `apiVersion` キーの値を、仮想マシンの Azure REST API の最新バージョン (このタスクの前半で確認したバージョン) に設定して、変更を保存します。ファイルは開いたままにします。

    >**注**:変更を加える前に、元の値を記録しておいてください。 この値は、次のタスクで元に戻す必要がなります。

    >**注**:REST API のバージョンが **2020-12-01** の場合、変更を加えることで以下のような結果になります。

    ```json
    {
      "apiVersion": "2020-12-01",
      "type": "Microsoft.Compute/virtualMachines",
      "name": "[variables('vmName')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
        "[concat('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
      ],
    ```

    >**注**:これは、構成を意図的に実装するためのものですが、Azure Stack Hub ではまだサポートされていません。

1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、新しく変更したテンプレートを検証します。

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 検証の出力に目を通し、今回はサポートの問題が発生していることに確認します。

    >**注**:出力は次の形式でなければなりません。

    ```
    Validation Summary:
        Passed: 0
        NotSupported: 1
        Exception: 0
        Warning: 0
        Recommend: 0
        Total Templates: 1
    Report available at - C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html
    ```

1. **C:\AzureStack-Tools-az\TemplateValidator\sampletemplate2validationreport.html** ファイルを開いて、レポートを確認します。 

    >**注**:レポートには、次の形式のエントリが含まれている必要があります。**NotSupported: apiversion (Resource type:Microsoft.Compute/virtualMachines).Not Supported Values - 2020-12-01**.


#### <a name="task-4-remediate-template-issues"></a>タスク 4:テンプレートに関する問題を修正する

このタスクでは次のことを行います。

- テンプレートに関する問題を修正する

1. **AzS-HOST1** へのリモート デスクトップ セッション内で、エクスプローラーを使用して **C:\\Templates** フォルダーに移動し、**AzureCloudCapabilities.json** ファイルを開きます。
1. **AzureCloudCapabilities.json** ファイルで、`"ResourceTypeName": "virtualMachines",` セクションを探します。これは次の形式になっています。

    ```json
    {
      "ProviderNamespace": "Microsoft.Compute",
      "ResourceTypeName": "virtualMachines",
      "Locations": [
        "local"
      ],
      "ApiVersions": [
        "2020-06-01",
        "2019-12-01",
        "2019-07-01",
        "2019-03-01",
        "2018-10-01",
        "2018-06-01",
        "2018-04-01",
        "2017-12-01",
        "2017-03-30",
        "2016-08-30",
        "2016-03-30",
        "2015-11-01",
        "2015-06-15"
      ],
      "ApiProfiles": [
        "2017-03-09-profile",
        "2018-03-01-hybrid"
      ]
    },
    ```

1. **sampletemplate2.json** ファイルに切り替え、前のタスクで修正した REST API バージョンを元の値に戻します。 このバージョンが、上記の **AzureCloudCapabilities.json** の API バージョンと一致していることを確認してから、変更内容を保存します。
1. **[管理者: Windows PowerShell]** ウィンドウで次のように実行して、新しく変更したテンプレートを検証します。

    ```powershell
    Test-AzureRMTemplate `
        -TemplatePath $path `
        -TemplatePattern sampletemplate2.json `
        -CapabilitiesPath $path\AzureCloudCapabilities.json `
        -IncludeComputeCapabilities `
        -IncludeStorageCapabilities `
        -Report sampletemplate2validationreport.html `
        -Verbose
    ```

1. 検証の出力を調べて、出力の一覧に **NotSupported** エントリがまだ 1 つ以上ある場合は、PowerShell プロンプトで「`C:\AzureStack-Tools-az\TemplateValidator\sampletemplate1validationreport.html`」と入力して **Enter** キーを押し、検証レポートを開きます。
1. レポートを調べて残りの問題を特定し、**template2.json** ファイルを適切に変更します。テンプレートの検証をもう一度実行して、今度は問題がないことを確認します。

>**レビュー**:この演習では、クラウド機能ファイルを作成し、これを使用して Azure Resource Manager テンプレートを検証しました。 また、検証の結果をもとに、テンプレートの修正も行いました。
