---
title: Windows コンテナーを使用する Windows に Azure IoT Edge をインストールする方法 | Microsoft Docs
description: Windows コンテナーを使用する Windows への Azure IoT Edge のインストール手順
author: kgremban
manager: philmea
ms.reviewer: veyalla
ms.service: iot-edge
services: iot-edge
ms.topic: conceptual
ms.date: 08/27/2018
ms.author: kgremban
ms.openlocfilehash: e6edc9d6e98c03b1a5847dc08bbaa3ad029aa673
ms.sourcegitcommit: 6b7c8b44361e87d18dba8af2da306666c41b9396
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/12/2018
ms.locfileid: "51565039"
---
# <a name="install-azure-iot-edge-runtime-on-windows-to-use-with-windows-containers"></a>Windows に Azure IoT Edge をインストールして Windows コンテナーと共に使用する

Azure IoT Edge ランタイムを使用すると、デバイスを IoT Edge デバイスに変えることができます。 このランタイムは、Raspberry Pi のような小型デバイスにも、産業用サーバーのような大型デバイスにもデプロイすることができます。 IoT Edge ランタイムを使用してデバイスを構成すると、クラウドからデバイスへのビジネス ロジックのデプロイを開始できます。 

IoT Edge ランタイムの動作とランタイムに含まれるコンポーネントについては、「[Azure IoT Edge ランタイムとそのアーキテクチャの概要](iot-edge-runtime.md)」を参照してください。

この記事では、Windows コンテナーと共に Windows x64 (AMD/Intel) システムに Azure IoT Edge ランタイムをインストールする手順を示します。 

Windows のサポートは現在プレビューの段階です。

## <a name="supported-windows-versions"></a>サポートされている Windows バージョン
Windows コンテナーを使用する Azure IoT Edge は、以下と共に使用できます。
  * Windows 10/IoT Enterprise/IoT Core (2018 年 4 月の更新プログラム適用済み)(ビルド 17134)
  * Windows Server 1803

現在サポートされているオペレーティング システムの詳細については、「[Azure IoT Edge のサポート](support.md#operating-systems)」を参照してください。

## <a name="install-the-container-runtime"></a>コンテナー ランタイムをインストールする 

>[!NOTE]
>Windows IoT Core でのコンテナー エンジンのインストールについては、[IoT Core デバイスのプロビジョニングに関する記事](how-to-install-iot-core.md)の手順を実行してから、以下の手順に進みます。

Azure IoT Edge は、[OCI と互換性のある](https://www.opencontainers.org/)コンテナー ランタイム (Docker など) に依存します。 [Docker for Windows](https://www.docker.com/docker-windows) は、開発とテストに使用できます。 

[Windows コンテナーを使用するように](https://docs.docker.com/docker-for-windows/#switch-between-windows-and-linux-containers) Docker for Windows を構成してください。

## <a name="install-the-azure-iot-edge-security-daemon"></a>Azure IoT Edge セキュリティ デーモンのインストール

>[!NOTE]
>Azure IoT Edge ソフトウェア パッケージには、(LICENSE ディレクトリ内の) パッケージ内にあるライセンス条項が適用されます。 パッケージを使用する前に、ライセンス条項をお読みください。 インストールし、パッケージを使用すると、これらの条項に同意したものと見なされます。 ライセンス条項に同意しない場合は、パッケージを使用しないでください。

IoT Hub によって提供されるデバイス接続文字列を使用して、1 つの IoT Edge デバイスを手動でプロビジョニングすることもできますし、 Device Provisioning Service を使用して、複数のデバイスを自動的にプロビジョニングすることもできます。これは、プロビジョニングするデバイスが多数ある場合に便利です。 目的にプロビジョニング方法に応じて、適切なインストール スクリプトを選択してください。 

### <a name="install-and-manually-provision"></a>インストールして手動でプロビジョニングする

1. 「[新しい Azure IoT Edge デバイスを登録する](how-to-register-device-portal.md)」の手順に従って、デバイスを登録し、デバイス接続文字列を取得します。 

2. IoT Edge デバイスで、PowerShell を管理者として実行します。 

3. IoT Edge ランタイムをダウンロードしてインストールします。 

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Install-SecurityDaemon -Manual -ContainerOs Windows
   ```

4. **DeviceConnectionString** の入力を求められたら、IoT Hub から取得した接続文字列を入力します。 接続文字列の前後に引用符は含めないでください。 

### <a name="install-and-automatically-provision"></a>インストールして自動的にプロビジョニングする

1. 「[Windows 上のシミュレートされた TPM Edge デバイスの作成とプロビジョニング](how-to-auto-provision-simulated-device-windows.md)」の手順に従って、Device Provisioning Service を設定し、その**スコープ ID** を取得して、TPM デバイスをシミュレートし、その**登録 ID** を取得した後、個別の登録を作成します。 デバイスが IoT Hub に登録されたら、インストールを続行します。  

   >[!TIP]
   >インストールとテストを行っている間は、TPM シミュレーターを実行しているウィンドウを開いたままにしてください。 

2. IoT Edge デバイスで、PowerShell を管理者として実行します。 

3. IoT Edge ランタイムをダウンロードしてインストールします。 

   ```powershell
   . {Invoke-WebRequest -useb aka.ms/iotedge-win} | Invoke-Expression; `
   Install-SecurityDaemon -Dps -ContainerOs Windows
   ```

4. メッセージが表示されたら、プロビジョニング サービスとデバイスの **ScopeID** と **RegistrationID** を入力します。 

## <a name="verify-successful-installation"></a>インストールの成功を確認する

以下によって IoT Edge サービスの状態を確認できます。 

```powershell
Get-Service iotedge
```

以下を使用して、過去 5 分のサービス ログを調べます。

```powershell

# Displays logs from last 5 min, newest at the bottom.

Get-WinEvent -ea SilentlyContinue `
  -FilterHashtable @{ProviderName= "iotedged";
    LogName = "application"; StartTime = [datetime]::Now.AddMinutes(-5)} |
  select TimeCreated, Message |
  sort-object @{Expression="TimeCreated";Descending=$false} |
  format-table -autosize -wrap
```

また、以下を使用して、実行中のモジュールを一覧表示します。

```powershell
iotedge list
```

## <a name="tips-and-suggestions"></a>ヒントと検索候補

ネットワークにプロキシ サーバーがある場合は、「[Configure an IoT Edge device to communicate through a proxy server](how-to-configure-proxy-support.md)」(プロキシ サーバー経由で通信するように IoT Edge デバイスを構成する) の手順に従って IoT Edge ランタイムをインストールし、起動してください。

## <a name="next-steps"></a>次の手順

ランタイムがインストールされた IoT Edge デバイスがプロビジョニングされたら、次は [IoT Edge モジュールをデプロイ](how-to-deploy-modules-portal.md)できます。

Edge ランタイムを正常にインストールできない場合は、[トラブルシューティング](troubleshoot.md)のページを調べてください。
