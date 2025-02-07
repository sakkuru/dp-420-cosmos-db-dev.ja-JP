---
lab:
  title: Azure Cosmos DB SQL API アカウントを作成する
  module: Module 1 - Get started with Azure Cosmos DB SQL API
ms.openlocfilehash: 4afd40294fba387fcc1ef5dda3196cc623609227
ms.sourcegitcommit: b90234424e5cfa18d9873dac71fcd636c8ff1bef
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/12/2022
ms.locfileid: "138025122"
---
# <a name="create-an-azure-cosmos-db-sql-api-account"></a>Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB について深く掘り下げる前に、よく使用するリソースの作成の基本を理解することが重要です。 ほとんどのシナリオでは、アカウント、データベース、コンテナー、および項目の作成に慣れている必要があります。 実際のシナリオでは、すべてのリソースが正しく作成されていることをテストするために、いくつかの基本的なクエリを "手元に" 用意しておくことも必要があります。

このラボでは、SQL API を使用して、新しい Azure Cosmos DB アカウントを作成します。 次に、データ エクスプローラーを使用して、データベース、コンテナー、2 つの項目を作成します。 最後に、データベースに対してクエリを実行して、作成した項目を確認します。

## <a name="create-a-new-azure-cosmos-db-account"></a>新しい Azure Cosmos DB アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。 Azure Cosmos DB アカウントを初めてプロビジョニングするときに、そのアカウントでサポートする API を選択します (たとえば、**Mongo API** または **SQL API**)。

1. 新しい Web ブラウザー ウィンドウまたはタブ内で、Azure portal (``portal.azure.com``) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[Azure サービス]** カテゴリで、 **[リソースの作成]** を選択してから、 **[Azure Cosmos DB]** を選択します。

    > &#128161; または、 **[&#8801;]** メニューを展開し、 **[すべてのサービス]** を選択し、 **[データベース]** カテゴリで、 **[Azure Cosmos DB]** を選んでから **[作成]** を選択します。

1. **[API オプションの選択]** ペインで、 **[コア (SQL - 推奨)]** セクション内の **[作成]** オプションを選択します。

1. **[Azure Cosmos DB アカウントの作成]** ペイン内で、 **[基本]** タブを確認します。

1. **[基本]** タブで、各設定に対して次の値を入力します。

    | **設定** | **値** |
    | --: | :-- |
    | **サブスクリプション** | ''*すべてのリソースはリソース グループに属している必要があります。すべてのリソース グループはサブスクリプションに属している必要があります。ここでは、既存の Azure サブスクリプションを使用します。* '' |
    | **リソース グループ** | ''*すべてのリソースはリソース グループに属している必要があります。ここでは、既存のリソース グループを選択するか、新しいものを作成します。* '' |
    | **アカウント名** | ''*グローバルに一意のアカウント名。この名前は、要求の DNS アドレスの一部として使用されます。グローバルに一意の名前を入力します。ポータルでは、名前がリアルタイムで確認されます。* '' |
    | **Location** | ''*データベースが最初にホストされる地理的リージョンを選択します。使用可能なリージョンを選びます。* '' |
    | **容量モード** | ''*プロビジョニングされたスループットを選択します*'' |
    | **Apply Free Tier Discount (Free レベル割引の適用)** | *適用しない* |

1. **[確認と作成]** を選択して **[確認と作成]** タブに移動してから、 **[作成]** を選択します。

    > &#128221; Azure Cosmos DB SQL API アカウントが使用できる状態になるまで、10 分から 15 分かかる場合があります。

1. **[デプロイ]** ペインを確認します。 デプロイが完了すると、そのペインが更新され、 **[デプロイに成功しました]** というメッセージが表示されます。

1. 引き続き **[デプロイ]** ペインで、 **[リソースに移動]** を選択します。

## <a name="use-the-data-explorer-to-create-a-new-database-and-container"></a>データ エクスプローラーを使用して、新しいデータベースとコンテナーを作成する

Azure portal で Azure Cosmos DB SQL API データベースおよびコンテナーを管理する場合にはデータ エクスプローラーが主要なツールになります。 このラボで使用する基本的なデータベースとコンテナーを作成します。

1. **[Azure Cosmos DB アカウント]** ペインで、リソース メニューから **[データ エクスプローラー]** を選択します。

1. **[データ エクスプローラー]** ペインで、 **[New Container]** を選択します。

1. **[New Container]** ポップアップで、各設定に次の値を入力してから **[OK]** を選択します。

    | **設定** | **値** |
    | --: | :-- |
    | **データベース ID** | *cosmicworks* |
    | **コンテナー間でスループットを共有する** | *選択しない* |
    | **コンテナー ID** | *products* |
    | **パーティション キー** | */categoryId* |
    | **コンテナーのスループット (自動スケーリング)** | *[Manual]* |
    | **RU/秒** | *400* |

1. **[データ エクスプローラー]** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

## <a name="use-the-data-explorer-to-create-new-items"></a>データ エクスプローラーを使用して新しい項目を作成する

またデータ エクスプローラーには、Azure Cosmos DB SQL API コンテナー内で項目を照会、作成、および管理するための一連の SQL 機能が含まれています。 データ エクスプローラーで未加工の JSON を使用して 2 つの基本的な項目を作成します。

1. **[データ エクスプローラー]** ペインで、**cosmicworks** データベース ノード、**products** コンテナー ノードの順に展開してから、 **[Items]** を選択します。

1. **[データ エクスプローラー]** ペインのまま、コマンド バーから **[New Item]** を選択します。 エディターで、プレースホルダーの JSON 項目を次の内容に置き換えます。

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. コマンド バーから **[Save]** を選択して、最初の JSON 項目を追加します。

1. **[Items]** タブに戻り、コマンド バーから **[New Item]** を選択します。 エディターで、プレースホルダーの JSON 項目を次の内容に置き換えます。

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. コマンド バーから **[Save]** を選択して、2 つ目の JSON 項目を追加します。

1. **[Items]** タブの **[Items]** ペインで、2 つの新しい項目を確認します。

## <a name="use-the-data-explorer-to-issue-a-basic-query"></a>データ エクスプローラーを使用して基本クエリを発行する

最後に、データ エクスプローラーには組み込みのクエリ エディターが用意されています。これを使用して、クエリの発行、結果の確認、および 1 秒あたりの要求ユニット数 (RU/秒) 単位での影響の測定を行います。

1. **[データ エクスプローラー]** ペインで **[New SQL Query]** を選択します。

1. [Query] タブで、 **[Execute Query]** を選択して、フィルターなしですべての項目を選択する標準クエリを表示します。

1. エディター領域のコンテンツを削除します。

1. **[クエリ]** タブで、プレースホルダー クエリを次の内容に置き換えます。

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; このクエリを実行すると、**price** が $500 を超えるすべての項目が選択されます。

1. **[Execute Query]** を選択します。

1. クエリの結果を確認します。JSON 項目が 1 つと、そのすべてのプロパティが含まれている必要があります。

1. **[クエリ]** タブで、 **[Query Stats]** を選択します。

1. 引き続き、 **[クエリ]** タブで、 **[Query Stats]** セクション内の **[Request Charge]** フィールドの値を確認します。

    > &#128221; 通常、このシンプルなクエリの要求料金は、コンテナー サイズが小さい場合は 2 から 3 RU/s となります。

1. Web ブラウザーのウィンドウまたはタブを閉じます。
