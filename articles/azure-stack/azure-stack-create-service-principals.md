---
title: Azure Stack のサービス プリンシパルを作成する | Microsoft Docs
description: Azure Resource Manager でロール ベースのアクセス制御と共に使用してリソースへのアクセスを管理できる、新しいサービス プリンシパルを作成する方法について説明します。
services: azure-resource-manager
documentationcenter: na
author: sethmanheim
manager: femila
ms.assetid: 7068617b-ac5e-47b3-a1de-a18c918297b6
ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/26/2018
ms.author: sethm
ms.openlocfilehash: a6d8ef698c005429c1184b5565b1a9387d05e062
ms.sourcegitcommit: fbdfcac863385daa0c4377b92995ab547c51dd4f
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/30/2018
ms.locfileid: "50230116"
---
# <a name="provide-applications-access-to-azure-stack"></a>Azure Stack へのアクセスをアプリケーションに提供する

*適用先: Azure Stack 統合システムと Azure Stack 開発キット*

Azure Stack 内の Azure Resource Manager を通じてリソースをデプロイまたは構成するためのアクセスがアプリケーションに必要な場合は、アプリケーションの資格情報であるサービス プリンシパルを作成します。 サービス プリンシパルには必要な権限のみを委任できます。  

たとえば、Azure Resource Manager を使用して Azure リソースのインベントリを管理する構成管理ツールがあります。 このシナリオでは、サービス プリンシパルを作成し、これに閲覧者のロールを付与して、構成管理ツールのアクセスを読み取り専用に制限します。 

サービス プリンシパルは、お客様自身の資格情報でアプリを実行するよりも推奨されますが、その理由は次のとおりです。

* お客様自身のアカウントのアクセス許可とは異なるアクセス許可を、サービス プリンシパルに割り当てることができます。 通常、こうしたアクセス許可は、アプリが行う必要があることに制限されます。
* お客様の責任が変わっても、アプリの資格情報を変更する必要はありません。
* 無人インストール用スクリプトを実行するときに、証明書を使用して認証を自動化できます。  

## <a name="getting-started"></a>使用の開始

Azure Stack のデプロイ方法に応じて、サービス プリンシパルを作成して開始します。 このドキュメントでは、[Azure Active Directory (Azure AD)](#create-service-principal-for-azure-ad) および [Active Directory フェデレーション サービス (AD FS)](#create-service-principal-for-ad-fs) のサービス プリンシパルを作成する方法を説明します。 サービス プリンシパルを作成したら、AD FS と Azure Active Directory の両方に共通する一連の手順を実行して、ロールに[アクセス許可を委任](#assign-role-to-service-principal)します。     

## <a name="create-service-principal-for-azure-ad"></a>Azure AD のサービス プリンシパルを作成する

Azure AD を ID ストアとして使用して Azure Stack をデプロイした場合は、Azure での手順と同様の方法でサービス プリンシパルを作成できます。 このセクションでは、それらの手順をポータルで行う方法について説明します。 始める前に、[Azure AD で必要なアクセス許可](../active-directory/develop/howto-create-service-principal-portal.md#required-permissions)があることを確認してください。

### <a name="create-service-principal"></a>サービス プリンシパルの作成
このセクションでは、アプリケーションが表示される Azure AD にアプリケーション (サービス プリンシパル) を作成します。

1. [Azure Portal](https://portal.azure.com) で Azure アカウントにサインインします。
2. **[Azure Active Directory]** > **[アプリの登録]** > **[新しいアプリケーションの登録]** を選択します   
3. アプリケーションの名前と URL を指定します。 作成するアプリケーションの種類として、**[Web アプリ/API]** または **[ネイティブ]** を選択します。 値を設定したら、**[作成]** をクリックします。

アプリケーションのサービス プリンシパルが作成されました。

### <a name="get-credentials"></a>資格情報の取得
プログラムでログインする場合、アプリケーションに対しては ID を、Web アプリ/API に対しては認証キーを使用します。 これらの値を取得するには、次の手順に従います。

1. Active Directory の **[アプリの登録]** で、アプリケーションを選択します。

2. **アプリケーション ID** をコピーし、アプリケーション コードに保存します。 「[サンプル アプリケーション](#sample-applications)」の各アプリケーションでは、この値をクライアント ID と呼んでいます。

     ![[クライアント ID]](./media/azure-stack-create-service-principal/image12.png)
3. Web アプリ/API に対して認証キーを生成するには、**[設定]** > **[キー]** の順に選択します。 

4. キーの説明を入力し、キーの期間を指定します。 操作が完了したら、**[保存]** をクリックします。

キーを保存すると、キーの値が表示されます。 キーは後から取得できないため、この値をメモ帳などの一時的な場所にコピーします。 キー値は、アプリケーションとしてサインインする際にアプリケーション ID と共に入力します。 アプリケーションが取得できる場所にキー値を保存します。

![保存されたキー](./media/azure-stack-create-service-principal/image15.png)

完了したら、引き続き[アプリケーションにロールを割り当て](#assign-role-to-service-principal)ます。

## <a name="create-service-principal-for-ad-fs"></a>AD FS のサービス プリンシパルを作成する
AD FS を使用して Azure Stack をデプロイした場合は、PowerShell を使ってサービス プリンシパルを作成し、アクセスのロールを割り当てて、その ID を使用して PowerShell からサインインできます。

ERCS 仮想マシン上で、特権エンドポイントからスクリプトが実行されます。

要件:
- 証明書が必要です。

証明書の要件:
 - 暗号化サービス プロバイダー (CSP) は、従来のキー プロバイダーである必要があります。
 - 公開キーと秘密キーの両方が必要なため、証明書は PFX ファイル形式である必要があります。 Windows サーバーでは、公開キー ファイル (SSL 証明書ファイル) と関連付けられている秘密キー ファイルが含まれている .pfx ファイルが使用されます。
 - 運用環境では、証明書は、内部の証明機関または公的証明機関のどちらかから発行されている必要があります。 公的証明機関を使用している場合は、Microsoft の信頼されたルート機関プログラムの一部としてその機関を基本オペレーティング システム イメージに含める必要があります。 [Microsoft の信頼されたルート証明書プログラム: 参加者](https://gallery.technet.microsoft.com/Trusted-Root-Certificate-123665ca)に完全な一覧があります。
 - お使いの Azure Stack インフラストラクチャは、証明書において公開されている証明機関の証明書失効リスト (CRL) の場所にネットワーク アクセスできる必要があります。 この CRL は、HTTP エンドポイントである必要があります。


#### <a name="parameters"></a>parameters

自動化パラメーターの入力として、次の情報が必要です。


|パラメーター|説明|例|
|---------|---------|---------|
|Name|SPN アカウントの名前|MyAPP|
|ClientCertificates|証明書オブジェクトの配列|X509 証明書|
|ClientRedirectUris<br>(省略可能)|アプリケーションのリダイレクト URI|-|

#### <a name="example"></a>例

1. Windows PowerShell セッションを管理者特権で開き、次のコマンドを実行します。

   > [!NOTE]
   > この例では、自己署名証明書を作成します。 運用環境デプロイでこれらのコマンドを実行する場合、[Get-Item](/powershell/module/Microsoft.PowerShell.Management/Get-Item) を利用して、使用する証明書の証明書オブジェクトを取得します。

   ```PowerShell  
    # Credential for accessing the ERCS PrivilegedEndpoint, typically domain\cloudadmin
    $creds = Get-Credential

    # Creating a PSSession to the ERCS PrivilegedEndpoint
    $session = New-PSSession -ComputerName <ERCS IP> -ConfigurationName PrivilegedEndpoint -Credential $creds

    # This produces a self signed cert for testing purposes. It is preferred to use a managed certificate for this.
    $cert = New-SelfSignedCertificate -CertStoreLocation "cert:\CurrentUser\My" -Subject "CN=<yourappname>" -KeySpec KeyExchange

    $ServicePrincipal = Invoke-Command -Session $session -ScriptBlock { New-GraphApplication -Name '<yourappname>' -ClientCertificates $using:cert}
    $AzureStackInfo = Invoke-Command -Session $session -ScriptBlock { get-azurestackstampinformation }
    $session|remove-pssession

    # For Azure Stack development kit, this value is set to https://management.local.azurestack.external. This is read from the AzureStackStampInformation output of the ERCS VM.
    $ArmEndpoint = $AzureStackInfo.TenantExternalEndpoints.TenantResourceManager

    # For Azure Stack development kit, this value is set to https://graph.local.azurestack.external/. This is read from the AzureStackStampInformation output of the ERCS VM.
    $GraphAudience = "https://graph." + $AzureStackInfo.ExternalDomainFQDN + "/"

    # TenantID for the stamp. This is read from the AzureStackStampInformation output of the ERCS VM.
    $TenantID = $AzureStackInfo.AADTenantID

    # Register an AzureRM environment that targets your Azure Stack instance
    Add-AzureRMEnvironment `
    -Name "AzureStackUser" `
    -ArmEndpoint $ArmEndpoint

    # Set the GraphEndpointResourceId value
    Set-AzureRmEnvironment `
    -Name "AzureStackUser" `
    -GraphAudience $GraphAudience `
    -EnableAdfsAuthentication:$true

    Add-AzureRmAccount -EnvironmentName "azurestackuser" `
    -ServicePrincipal `
    -CertificateThumbprint $ServicePrincipal.Thumbprint `
    -ApplicationId $ServicePrincipal.ClientId `
    -TenantId $TenantID

    # Output the SPN details
    $ServicePrincipal

   ```

2. 自動化が完了すると、SPN を使用するために必要な詳細が表示されます。 

   例: 

   ```shell
   ApplicationIdentifier : S-1-5-21-1512385356-3796245103-1243299919-1356
   ClientId              : 3c87e710-9f91-420b-b009-31fa9e430145
   Thumbprint            : 30202C11BE6864437B64CE36C8D988442082A0F1
   ApplicationName       : Azurestack-MyApp-c30febe7-1311-4fd8-9077-3d869db28342
   PSComputerName        : azs-ercs01
   RunspaceId            : a78c76bb-8cae-4db4-a45a-c1420613e01b
   ```

### <a name="assign-a-role"></a>ロールの割り当て
サービス プリンシパルを作成したら、[ロールを割り当てる](#assign-role-to-service-principal)必要があります。

### <a name="sign-in-through-powershell"></a>PowerShell を使ったサインイン
ロールを割り当てたら、次のコマンドでサービス プリンシパルを使用して Azure Stack にサインインできます。

```powershell
Add-AzureRmAccount -EnvironmentName "<AzureStackEnvironmentName>" `
 -ServicePrincipal `
 -CertificateThumbprint $servicePrincipal.Thumbprint `
 -ApplicationId $servicePrincipal.ClientId ` 
 -TenantId $directoryTenantId
```

## <a name="assign-role-to-service-principal"></a>サービス プリンシパルにロールを割り当てる
サブスクリプション内のリソースにアクセスするには、アプリケーションをロールに割り当てる必要があります。 アプリケーションにとって適切なアクセス許可を表すのはどのロールであるかを判断します。 利用可能なロールについては、「[RBAC: 組み込みのロール](../role-based-access-control/built-in-roles.md)」を参照してください。

スコープは、サブスクリプション、リソース グループ、またはリソースのレベルで設定できます。 アクセス許可は、スコープの下位レベルに継承されます。 たとえば、アプリケーションをリソース グループの閲覧者ロールに追加すると、アプリケーションではリソース グループとそれに含まれているすべてのリソースを読み取ることができます。

1. Azure Stack ポータルで、アプリケーションを割り当てるスコープのレベルに移動します。 たとえば、サブスクリプション スコープでロールを割り当てるには、**[サブスクリプション]** を選択します。 リソース グループまたはリソースを選択することもできます。

2. アプリケーションを割り当てる特定のサブスクリプション (リソース グループまたはリソース) を選択します。

     ![割り当てのためのサブスクリプションの選択](./media/azure-stack-create-service-principal/image16.png)

3. **[アクセス制御 (IAM)]** を選択します。

     ![アクセスの選択](./media/azure-stack-create-service-principal/image17.png)

4. **[追加]** を選択します。

5. アプリケーションに割り当てるロールを選択します。

6. アプリケーションを検索して選択します。

7. **[OK]** をクリックして、ロールの割り当てを完了します。 該当のスコープのロールに割り当てられたユーザーの一覧にアプリケーションが表示されます。

サービス プリンシパルを作成してロールを割り当てたら、アプリケーション内でこれを使用して、Azure Stack リソースにアクセスできます。  

## <a name="next-steps"></a>次の手順

[AD FS のユーザーの追加](azure-stack-add-users-adfs.md)
[ユーザーのアクセス許可の管理](azure-stack-manage-permissions.md)
