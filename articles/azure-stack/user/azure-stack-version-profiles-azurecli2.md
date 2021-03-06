---
title: CLI を使用した Azure Stack への接続 | Microsoft Docs
description: クロスプラットフォーム コマンドライン インターフェイス (CLI) を使用して、Azure Stack でリソースを管理およびデプロイする方法
services: azure-stack
documentationcenter: ''
author: sethmanheim
manager: femila
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/15/2019
ms.author: sethm
ms.reviewer: sijuman
ms.lastreviewed: 01/24/2019
ms.openlocfilehash: 40973fbdd1965eb84776fc9365718c65fa0149a7
ms.sourcegitcommit: 79038221c1d2172c0677e25a1e479e04f470c567
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/19/2019
ms.locfileid: "56416991"
---
# <a name="use-api-version-profiles-with-azure-cli-in-azure-stack"></a>Azure Stack での Azure CLI による API バージョンのプロファイルの使用

この記事の手順に従うと、Linux、Mac、Windows クライアントのプラットフォームから Azure Stack Development Kit のリソースを管理するように Azure コマンド ライン インターフェイス (CLI) を設定できます。

## <a name="install-cli"></a>CLI のインストール

開発ワークステーションにサインインし、CLI をインストールします。 Azure Stack には、Azure CLI のバージョン 2.0 以降が必要です。 このバージョンは、記事「[Azure CLI のインストール](/cli/azure/install-azure-cli)」で説明されている手順を使用してインストールできます。 インストールが正常に完了したことを確認するには、ターミナルまたはコマンド プロンプト ウィンドウを開いて次のコマンドを実行します。

```azurecli
az --version
```

お使いのコンピューターにインストールされている Azure CLI と依存するその他のライブラリのバージョンが表示されます。

## <a name="trust-the-azure-stack-ca-root-certificate"></a>Azure Stack の CA ルート証明書を信頼する

1. Azure Stack の CA ルート証明書を [Azure Stack オペレーター](../azure-stack-cli-admin.md#export-the-azure-stack-ca-root-certificate)から取得して信頼します。 Azure Stack の CA ルート証明書を信頼するには、Python の既存の証明書に追加します。

1. マシンで証明書の場所を探します。 この場所は、Python をインストールした場所に応じて異なる場合があります。 [pip](https://pip.pypa.io) と [certifi](https://pypi.org/project/certifi/) モジュールをインストールしておく必要があります。 Bash プロンプトから次の Python コマンドを使用できます。

    ```bash  
    python -c "import certifi; print(certifi.where())"
    ```

    証明書の場所を書き留めておきます (例: `~/lib/python3.5/site-packages/certifi/cacert.pem`)。 このパスは、オペレーティング システムと、インストールされている Python のバージョンによって異なります。

### <a name="set-the-path-for-a-development-machine-inside-the-cloud"></a>クラウド内の開発用マシンのパスを設定する

Azure Stack 環境内に作成されている Linux マシンから CLI を実行する場合は、証明書のパスを指定して次の Bash コマンドを実行します。

```bash
sudo cat /var/lib/waagent/Certificates.pem >> ~/<yourpath>/cacert.pem
```

### <a name="set-the-path-for-a-development-machine-outside-the-cloud"></a>クラウドの外部の開発用マシンのパスを設定する

Azure Stack 環境の外部のマシンから CLI を実行する場合:  

1. [Azure Stack への VPN 接続](azure-stack-connect-azure-stack.md)を設定します。
1. Azure Stack オペレーターから取得した PEM 証明書をコピーし、ファイルの場所 (PATH_TO_PEM_FILE) を書き留めておきます。
1. 開発用ワークステーション上のオペレーティング システムに応じて、次のセクションのコマンドを実行します。

#### <a name="linux"></a>Linux

```bash
sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
```

#### <a name="macos"></a>macOS

```bash
sudo cat PATH_TO_PEM_FILE >> ~/<yourpath>/cacert.pem
```

#### <a name="windows"></a> Windows

```powershell
$pemFile = "<Fully qualified path to the PEM certificate Ex: C:\Users\user1\Downloads\root.pem>"

$root = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2
$root.Import($pemFile)

Write-Host "Extracting required information from the cert file"
$md5Hash    = (Get-FileHash -Path $pemFile -Algorithm MD5).Hash.ToLower()
$sha1Hash   = (Get-FileHash -Path $pemFile -Algorithm SHA1).Hash.ToLower()
$sha256Hash = (Get-FileHash -Path $pemFile -Algorithm SHA256).Hash.ToLower()

$issuerEntry  = [string]::Format("# Issuer: {0}", $root.Issuer)
$subjectEntry = [string]::Format("# Subject: {0}", $root.Subject)
$labelEntry   = [string]::Format("# Label: {0}", $root.Subject.Split('=')[-1])
$serialEntry  = [string]::Format("# Serial: {0}", $root.GetSerialNumberString().ToLower())
$md5Entry     = [string]::Format("# MD5 Fingerprint: {0}", $md5Hash)
$sha1Entry    = [string]::Format("# SHA1 Fingerprint: {0}", $sha1Hash)
$sha256Entry  = [string]::Format("# SHA256 Fingerprint: {0}", $sha256Hash)
$certText = (Get-Content -Path $pemFile -Raw).ToString().Replace("`r`n","`n")

$rootCertEntry = "`n" + $issuerEntry + "`n" + $subjectEntry + "`n" + $labelEntry + "`n" + `
$serialEntry + "`n" + $md5Entry + "`n" + $sha1Entry + "`n" + $sha256Entry + "`n" + $certText

Write-Host "Adding the certificate content to Python Cert store"
Add-Content "${env:ProgramFiles(x86)}\Microsoft SDKs\Azure\CLI2\Lib\site-packages\certifi\cacert.pem" $rootCertEntry

Write-Host "Python Cert store was updated to allow the Azure Stack CA root certificate"
```

## <a name="get-the-virtual-machine-aliases-endpoint"></a>仮想マシンのエイリアス エンドポイントを取得する

CLI を使用して仮想マシンを作成するには、Azure Stack のオペレーターに連絡し、仮想マシンのエイリアス エンドポイント URI を取得する必要があります。 たとえば、Azure では次の URI `https://raw.githubusercontent.com/Azure/azure-rest-api-specs/master/arm-compute/quickstart-templates/aliases.json` が使用されます。 クラウド管理者は Azure Stack Marketplace で使用可能なイメージを使用して、Azure Stack 用に同様のエンドポイントを設定する必要があります。 次のセクションで示すように、`endpoint-vm-image-alias-doc` パラメーターのエンドポイント URI を `az cloud register` コマンドに渡す必要があります。 
  
## <a name="connect-to-azure-stack"></a>Azure Stack への接続

次の手順を使用して Azure Stack に接続します。

1. `az cloud register` コマンドを実行して、Azure Stack 環境を登録します。 一部のシナリオでは、インターネットへの直接送信接続がプロキシまたはファイアウォール経由でルーティングされ、SSL インターセプトが適用されます。 このような場合は、`az cloud register` コマンドが、"クラウドからエンドポイントを取得できない" といったエラーで失敗する可能性があります。 このエラーを回避するには、次の環境変数を設定します。

   ```shell
   set AZURE_CLI_DISABLE_CONNECTION_VERIFICATION=1 
   set ADAL_PYTHON_SSL_NO_VERIFY=1
   ```
   
    a. *クラウド管理*環境を登録するには、次のコマンドを使用します。

      ```azurecli
      az cloud register \ 
        -n AzureStackAdmin \ 
        --endpoint-resource-manager "https://adminmanagement.local.azurestack.external" \ 
        --suffix-storage-endpoint "local.azurestack.external" \ 
        --suffix-keyvault-dns ".adminvault.local.azurestack.external" \ 
        --endpoint-vm-image-alias-doc <URI of the document which contains virtual machine image aliases>
      ```
    b. *ユーザー*環境を登録するには、次のコマンドを使用します。

      ```azurecli
      az cloud register \ 
        -n AzureStackUser \ 
        --endpoint-resource-manager "https://management.local.azurestack.external" \ 
        --suffix-storage-endpoint "local.azurestack.external" \ 
        --suffix-keyvault-dns ".vault.local.azurestack.external" \ 
        --endpoint-vm-image-alias-doc <URI of the document which contains virtual machine image aliases>
      ```
    c. マルチ テナント環境で*ユーザー*を登録するには、次のコマンドを使用します。

      ```azurecli
      az cloud register \ 
        -n AzureStackUser \ 
        --endpoint-resource-manager "https://management.local.azurestack.external" \ 
        --suffix-storage-endpoint "local.azurestack.external" \ 
        --suffix-keyvault-dns ".vault.local.azurestack.external" \ 
        --endpoint-vm-image-alias-doc <URI of the document which contains virtual machine image aliases> \
        --endpoint-active-directory-resource-id=<URI of the ActiveDirectoryServiceEndpointResourceID> \
        --profile 2018-03-01-hybrid
      ```
    d.[Tableau Server return URL]: Tableau Server ユーザーがアクセスする URL。 AD FS 環境にユーザーを登録するには、次のコマンドを使用します。

      ```azurecli
      az cloud register \
        -n AzureStack  \
        --endpoint-resource-manager "https://management.local.azurestack.external" \
        --suffix-storage-endpoint "local.azurestack.external" \
        --suffix-keyvault-dns ".vault.local.azurestack.external"\
        --endpoint-active-directory-resource-id "https://management.adfs.azurestack.local/<tenantID>" \
        --endpoint-active-directory-graph-resource-id "https://graph.local.azurestack.external/"\
        --endpoint-active-directory "https://adfs.local.azurestack.external/adfs/"\
        --endpoint-vm-image-alias-doc <URI of the document which contains virtual machine image aliases> \
        --profile "2018-03-01-hybrid"
      ```
1. 次のコマンドを使用して、アクティブな環境を設定します。
   
    a. *クラウド管理*環境の場合は、次のコマンドを使用します。

      ```azurecli
      az cloud set \
        -n AzureStackAdmin
      ```

    b. *ユーザー*環境の場合は、次のコマンドを使用します。

      ```azurecli
      az cloud set \
        -n AzureStackUser
      ```

1. Azure Stack 固有の API バージョンのプロファイルを使用するようにお使いの環境の構成を更新します。 構成を更新するには、次のコマンドを実行します。

    ```azurecli
    az cloud update \
      --profile 2018-03-01-hybrid
   ```

    >[!NOTE]  
    >1808 ビルドより前のバージョンの Azure Stack を実行している場合は、**2018-03-01-hybrid** の API バージョンのプロファイルではなく、**2017-03-09-profile** の API バージョンのプロファイルを使用する必要があります。

1. `az login` コマンドを使用して、Azure Stack 環境にサインインします。 Azure Stack 環境には、ユーザーまたは[サービス プリンシパル](../../active-directory/develop/app-objects-and-service-principals.md)としてサインインできます。 

    * Azure AD 環境
      * *ユーザー*としてサインインする場合: `az login` コマンド内で直接ユーザー名とパスワードを指定するか、ブラウザーを使用して認証できます。 多要素認証が有効になっているアカウントの場合は、後者を実行する必要があります。

      ```azurecli
      az login \
        -u <Active directory global administrator or user account. For example: username@<aadtenant>.onmicrosoft.com> \
        --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com>
      ```

      > [!NOTE]
      > お使いのユーザー アカウントで多要素認証が有効になっている場合は、`-u` パラメーターを指定しないで、`az login` コマンドを使用できます。 このコマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。
   
      * *サービス プリンシパル*を使ってサインインする｡サインインする前に、CLI または [Azure Portal でサービス プリンシパルを作成](azure-stack-create-service-principals.md)してロールに割り当てます。 次のコマンドを使用してサインインします。

      ```azurecli  
      az login \
        --tenant <Azure Active Directory Tenant name. For example: myazurestack.onmicrosoft.com> \
        --service-principal \
        -u <Application Id of the Service Principal> \
        -p <Key generated for the Service Principal>
      ```
    * AD FS 環境

        * Web ブラウザーとデバイス コードを使用してユーザーとしてサインインします。  
           ```azurecli  
           az login --use-device-code
           ```

           > [!NOTE]  
           >コマンドを実行すると、認証で使用する必要がある URL とコードを取得できます。

        * サービス プリンシパルとしてサインインします。
        
          1. サービス プリンシパルのログインに使用する .pem ファイルを用意します。

            * プリンシパルが作成されたクライアント マシン上でサービス プリンシパル証明書 (`cert:\CurrentUser\My` にあり､cert 名はプリンシパル名と同じ) を秘密キーを含む pfx としてエクスポートします。
        
            * pfx を pem に変換します (OpenSSL ユーティリティ を使用)。

          2.  CLI にサインインします。
            ```azurecli  
            az login --service-principal \
              -u <Client ID from the Service Principal details> \
              -p <Certificate's fully qualified name, such as, C:\certs\spn.pem>
              --tenant <Tenant ID> \
              --debug 
            ```

## <a name="test-the-connectivity"></a>接続のテスト

すべての設定が完了したら、Azure Stack 内で CLI を使ってリソースを作成してみましょう。 たとえば、アプリケーションのリソース グループを作成して仮想マシンを追加できます。 次のコマンドを使用して、"MyResourceGroup" という名前のリソース グループを作成します。

```azurecli
az group create \
  -n MyResourceGroup -l local
```

リソース グループが正常に作成されると、上記のコマンドによって、新たに作成されたリソースの次のプロパティが出力されます。

![リソース グループ作成の出力](media/azure-stack-connect-cli/image1.png)

## <a name="known-issues"></a>既知の問題

Azure Stack 内で CLI を使用する場合、次のような既知の問題があります。

 - CLI 対話モード。たとえば、Azure Stack では、`az interactive` コマンドはまだサポートされていません。
 - Azure Stack で使用できる仮想マシン イメージの一覧を取得するには、`az vm image list` コマンドの代わりに、`az vm image list --all` コマンドを使用します。 `--all` オプションを指定すると、Azure Stack 環境内で使用できるイメージのみが応答として返されます。
 - Azure で使用できる仮想マシン イメージの別名は、Azure Stack に適用できない場合があります。 仮想マシン イメージを使用する場合は、イメージの別名の代わりに、URN パラメーター全体 (Canonical:UbuntuServer:14.04.3-LTS:1.0.0) を使用する必要がありますします。 この URN は、`az vm images list` コマンドから派生したイメージ仕様と一致している必要があります。

## <a name="next-steps"></a>次の手順

- [Azure CLI を使用したテンプレートのデプロイ](azure-stack-deploy-template-command-line.md)
- [Azure Stack ユーザー (オペレーター) に対する Azure CLI の有効化](../azure-stack-cli-admin.md)
- [ユーザー アクセス許可の管理](azure-stack-manage-permissions.md) 
