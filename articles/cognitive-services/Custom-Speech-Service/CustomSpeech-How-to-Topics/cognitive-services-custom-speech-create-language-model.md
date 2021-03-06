---
title: Custom Speech Service を使用して言語モデルを作成するためのチュートリアル - Microsoft Cognitive Services | Microsoft Docs
description: このチュートリアルでは、Microsoft Cognitive Services の Custom Speech Service を使用して言語モデルを作成する方法について説明します。
services: cognitive-services
author: PanosPeriorellis
manager: onano
ms.service: cognitive-services
ms.component: custom-speech
ms.topic: tutorial
ms.date: 05/03/2017
ms.author: panosper
ms.openlocfilehash: 6af2da9ffc7678a58fcf1c647ba89c586066d2ad
ms.sourcegitcommit: 1aacea6bf8e31128c6d489fa6e614856cf89af19
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 10/16/2018
ms.locfileid: "49339098"
---
# <a name="tutorial-create-a-custom-language-model"></a>チュートリアル: カスタム言語モデルを作成する

[!INCLUDE [Deprecation note](../../../../includes/cognitive-services-custom-speech-deprecation-note.md)]

このチュートリアルでは、アプリケーションでユーザーが発声または入力することが予想されるテキスト クエリまたは発言のカスタム言語モデルを作成します。 このカスタム言語モデルを Microsoft が提供する既存の最先端の音声モデルと組み合わせて使用して、アプリケーションに音声操作を追加できます。

このチュートリアルでは、以下の内容を学習します。
> [!div class="checklist"]
> * データを準備する
> * 言語データ セットをインポートする
> * カスタム言語モデルを作成する

Cognitive Services アカウントをお持ちでない場合は、開始する前に [無料アカウント](https://cris.ai) を作成してください。

## <a name="prerequisites"></a>前提条件

[[Cognitive Services サブスクリプション]](https://cris.ai/Subscriptions) ページを開いて、Cognitive Services アカウントに接続されることを確認します。

サブスクリプションが表示されない場合は、**[Get free subscription]\(無料のサブスクリプションを取得する\)** ボタンをクリックして、Cognitive Services にアカウントを作成させることができます。 または、**[Connect existing subscription]\(既存のサブスクリプションに接続する\)** ボタンをクリックして、Azure Portal で作成される Custom Search Service サブスクリプションに接続できます。

Azure Portal での Custom Search Service サブスクリプションの作成については、「[Create a Cognitive Services APIs account in the Azure portal](../../cognitive-services-apis-create-account.md)」(Azure Portal で Cognitive Services API アカウントを作成する) を参照してください。

## <a name="prepare-the-data"></a>データを準備する

アプリケーション用のカスタム言語モデルを作成するには、以下のような発言用例の一覧をシステムに提供する必要があります。

*   "彼は先週じんましんが出ました。"
*   "患者には、完治したヘルニア手術の傷跡があります。"

文は、完全な文章である必要も、文法的に正しい文章である必要もありませんが、展開中にシステムに入力されると思われる発言を正確に反映している必要があります。 これらの用例は、ユーザーがアプリケーションを使用して実行するタスクのスタイルとコンテンツの両方を反映する必要があります。

言語モデル データは、ロケールに応じて、US-ASCII または UTF-8 を使用してプレーン テキスト ファイルに書き込む必要があります。 en-US の場合は、両方のエンコードがサポートされます。 zh-CN の場合は、UTF-8 のみがサポートされます (BOM は省略可能です)。 テキスト ファイルには、1 行につき 1 つの用例 (文、発言、またはクエリ) を含める必要があります。

特定の文の重み (重要度) を高くする場合は、その文をデータに数回追加できます。 適切な繰り返し回数は、10 から 100 です。 100 に正規化すれば、これに対する相対的な重みを文に簡単に設定できます。

言語データの主な要件を次の表にまとめます。

| プロパティ | 値 |
|----------|-------|
| テキストのエンコード | en-US: US-ACSII または UTF-8、zh-CN: UTF-8|
| 1 行あたりの発言の数 | 1 |
| ファイルの最大サイズ | 200 MB |
| 解説 | 文字の 4 回を超える繰り返し (例: 'aaaaa') は避けます|
| 解説 | '\t' や [Unicode 文字コード表](http://www.utf8-chartable.de/) の U+00A1 より上の UTF-8 文字は使用しません|
| 解説 | 一意に発音する方法がないため、URI も拒否されます|

テキストのインポート時に、システムで処理できるように、テキストの正規化が行われます。 ただし、"_データをアップロードする前_" にユーザーが行う必要があるいくつかの重要な正規化があります。 言語データを準備する際に、適切な言語を決定するには、[文字起こしガイドライン](cognitive-services-custom-speech-transcription-guidelines.md)を参照してください。

## <a name="import-the-language-data-set"></a>言語データ セットをインポートする

[音響データ セット] 行の [インポート] ボタンをクリックします。新しいデータ セットをアップロードするためのサイトのページが表示されます。

言語データ セットをインポートする準備ができたら、[Custom Speech Service Portal](https://cris.ai) にログインします。  次に、上部のリボンの [Custom Speech] ドロップダウン メニューをクリックし、[Adaptation Data]\(適応データ\) を選択します。 Custom Speech Service へのデータのアップロードを初めて行う場合は、[データセット] という名前の空のテーブルが表示されます。

新しいデータ セットをアップロードするには、[言語データ セット] 行の [インポート] ボタンをクリックします。新しいデータ セットをアップロードするためのサイトのページが表示されます。 今後データ セットを識別するために役立つ [名前]と [説明] を入力します。 次に、[ファイルの選択] ボタンを使用して、言語データのテキスト ファイルを探します。 その後、[インポート] をクリックしてデータ セットをアップロードします。 データ セットのサイズによっては、数分かかる場合があります。

![試す](../../../media/cognitive-services/custom-speech-service/custom-speech-language-datasets-import.png)

インポートが完了したら、言語データ テーブルに戻り、言語データ セットに対応するエントリがあることを確認します。 一意の ID (GUID) が割り当てられることに注意してください。 データには、現在の状態を反映するステータスもあります。 そのステータスは、処理を待っている間は [待機中] に、検証中は [処理中] に、使用する準備ができると [完了] になります。 データの検証では、ファイル内のテキストに対する一連のチェックと、データのテキストの正規化が行われます。

ステータスが [完了] のときに、[レポートの表示] をクリックして、言語データ検証レポートを表示できます。 失敗した発言の詳細と共に、検証に成功した発言の数と失敗した発言の数が表示されます。 次の例では、不適切な文字のために 2 つの用例が検証に失敗しています (このデータ セットでは、最初の用例で 2 つの顔文字が使用され、2 つ目の用例で、ASCII 印刷可能文字セットの外部にある複数の文字が使用されていました)。

![試す](../../../media/cognitive-services/custom-speech-service/custom-speech-language-datasets-report.png)

言語データ セットは、ステータスが [完了] のときに、それを使用してカスタム言語モデルを作成できます。

![試す](../../../media/cognitive-services/custom-speech-service/custom-speech-language-datasets.png)

## <a name="create-a-custom-language-model"></a>カスタム言語モデルを作成する

言語データが準備できたら、[メニュー] ドロップダウン メニューから [言語モデル] をクリックして、カスタム言語モデルの作成プロセスを開始します。 このページには、現在のカスタム言語モデルを持つ [言語モデル] という名前のテーブルが含まれています。 カスタム言語モデルをまだ作成していない場合、テーブルは空になります。 現在のロケールがテーブル タイトルに反映されます。 別の言語の言語モデルを作成する場合は、[ロケールの変更] をクリックします。 サポートされている言語に関する追加情報については、[ロケールの変更](cognitive-services-custom-speech-change-locale.md)に関する記事を参照してください。 新しいモデルを作成するには、テーブル タイトルの下の [新規作成] リンクをクリックします。

[言語モデルの作成] ページで、このモデルの関連情報 (使用されるデータ セットなど) を追跡するために役立つ [名前] と [説明] を入力します。 次に、ドロップダウン メニューから [Base Language Model]\(ベース言語モデル\) を選択します。 このモデルがカスタマイズの開始点となります。 2 つのベース言語モデルから選択できます。 _Microsoft Search and Dictation LM_ は、コマンド、検索クエリ、ディクテーションなどのアプリケーションに向けて発せられる音声に適しています。 _Microsoft Conversational LM_ は、話し言葉スタイルで話される音声の認識に適しています。 この種類の音声は、通常は他の人に向けて発せられるものであり、コール センターや会議で発生します。

ベース言語モデルを指定したら、[言語データ] ドロップダウン メニューを使用して、カスタマイズに使用する言語データ セットを選択します。

![試す](../../../media/cognitive-services/custom-speech-service/custom-speech-language-models-create2.png)

音響モデルの作成と同じように、処理が完了したら、必要に応じて、新しいモデルのオフライン テストを実行することを選択できます。 オフライン テストは音声からテキストへのパフォーマンスを評価するため、音響データ セットが必要です。

言語モデルのオフライン テストを実行するには、[オフライン テスト] の横にあるチェック ボックスを選択します。 次に、ドロップダウン メニューから音響モデルを選択します。 カスタム音響モデルを作成していない場合は、Microsoft ベースの音響モデルがメニューの唯一のモデルになります。 Conversational LM ベース モデルを選択した場合は、ここで Conversational AM を使用する必要があります。 Search and Dictation LM モデルを使用する場合は、Search and Dictate AM を選択する必要があります。

最後に、評価を実行するために使用する音響データ セットを選択します。

処理を開始する準備ができたら、[作成] を選択します。 言語モデル テーブルに戻ります。 このモデルに対応するテーブルに新しいエントリがあります。 ステータスは、モデルの状態を反映し、[待機中]、[処理中]、[完了] を含むさまざまな状態を通過します。

モデルが [完了] 状態に達したら、エンドポイントに展開できます。 [結果の表示] をクリックすると、オフライン テストの結果が表示されます (実行した場合)。

ある時点でモデルの [名前] または [説明] を変更する場合は、言語モデル テーブルの適切な行の [編集] リンクを使用できます。

## <a name="next-steps"></a>次の手順

このチュートリアルでは、テキストで使用するカスタム言語モデルを開発しました。 オーディオ ファイルと文字起こしで使用するカスタム音響モデルを作成するには、音響モデルを作成するためのチュートリアルに進みます。

> [!div class="nextstepaction"]
> [カスタム音響モデルを作成する](cognitive-services-custom-speech-create-acoustic-model.md)
