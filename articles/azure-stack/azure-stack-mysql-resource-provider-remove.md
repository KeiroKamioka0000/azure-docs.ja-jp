---
title: Azure Stack 上の MySQL リソース プロバイダーを削除する | Microsoft Docs
description: Azure Stack のデプロイから MySQL リソース プロバイダーを削除する方法について説明します。
services: azure-stack
documentationCenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/14/2018
ms.author: jeffgilb
ms.reviewer: quying
ms.openlocfilehash: 7d3b0e179972464a1ed857c576ca8a7c8fc2e162
ms.sourcegitcommit: db2cb1c4add355074c384f403c8d9fcd03d12b0c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/15/2018
ms.locfileid: "51686808"
---
# <a name="remove-the-mysql-resource-provider"></a>MySQL リソースプロバイダーを削除する

MySQL リソース プロバイダーを削除する前に、プロバイダーの依存関係をすべて削除する必要があります。 また、リソース プロバイダーのインストールに使用したデプロイ パッケージのコピーも必要になります。

  |最小の Azure Stack バージョン|MySQL RP バージョン|
  |-----|-----|
  |バージョン 1808 (1.1808.0.97)|[MySQL RP バージョン 1.1.30.0](https://aka.ms/azurestacksqlrp11300)|
  |バージョン 1804 (1.0.180513.1)|[MySQL RP バージョン 1.1.24.0](https://aka.ms/azurestackmysqlrp11240)
  |     |     |

## <a name="dependency-cleanup"></a>依存関係のクリーンアップ

DeployMySqlProvider.ps1 スクリプトを実行してリソース プロバイダーを削除する前に、いくつかのクリーンアップ タスクを実行する必要があります。

次のクリーンアップ タスクは、Azure Stack テナント ユーザーが担当します。

* リソース プロバイダーからすべてのデータベースを削除する。 (テナント データベースを削除してもデータは削除されません。)
* プロバイダーの名前空間からの登録解除を行う。

次のクリーンアップ タスクは、Azure Stack オペレーターが担当します。

* MySQL アダプターからホスティング サーバーを削除する。
* MySQL アダプターを参照するすべてのプランを削除する。
* MySQL アダプターに関連付けられているすべてのクォータを削除する。

## <a name="to-remove-the-mysql-resource-provider"></a>MySQL リソースプロバイダーを削除するには

1. 既存の MySQL リソース プロバイダーの依存関係がすべて削除されていることを確認します。

   >[!NOTE]
   >依存リソースがリソース プロバイダーを現在使用している場合でも、MySQL リソース プロバイダーのアンインストールは続行されます。
  
2. MySQL リソース プロバイダー バイナリのコピーを入手し、自己展開形式ファイルを実行してコンテンツを一時ディレクトリに展開します。
3. SQL リソース プロバイダー バイナリのコピーを入手し、自己展開形式ファイルを実行してコンテンツを一時ディレクトリに展開します。
4. 新しい管理者特権の PowerShell コンソール ウィンドウを開き、MySQL リソース プロバイダー バイナリ ファイルを抽出したディレクトリに変更します。
5. 次のパラメーターを使用して、DeployMySqlProvider.ps1 スクリプトを実行します。
    - **Uninstall**。 リソース プロバイダーと関連付けられているすべてのリソースを削除します。
    - **PrivilegedEndpoint**。 特権エンドポイントの IP アドレスまたは DNS 名。
    - **AzureEnvironment**。 Azure Stack のデプロイに使用する Azure 環境。 Azure AD のデプロイでのみ必須です。
    - **CloudAdminCredential**。 特権エンドポイントへのアクセスに必要な、クラウド管理者の資格情報。
    - **DirectoryTenantID**
    - **AzCredential**。 Azure Stack サービス管理者アカウントの資格情報。 Azure Stack のデプロイに使用したのと同じ資格情報を使用します。

## <a name="next-steps"></a>次の手順

[App Services を PaaS として提供する](azure-stack-app-service-overview.md)
