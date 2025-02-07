---
lab:
  title: Azure Resource Manager テンプレートを使用して Azure Cosmos DB SQL API コンテナーを作成する
  module: Module 12 - Manage an Azure Cosmos DB SQL API solution using DevOps practices
---

# <a name="create-an-azure-cosmos-db-sql-api-container-using-azure-resource-manager-templates"></a>Azure Resource Manager テンプレートを使用して Azure Cosmos DB SQL API コンテナーを作成する

Azure Resource Manager テンプレートは、Azure にデプロイするインフラストラクチャを宣言的に定義する JSON ファイルです。 Azure Resource Manager テンプレートは、Azure にサービスをデプロイするための一般的なコードとしてのインフラストラクチャ ソリューションです。 Bicep は、JSON テンプレートの作成に使用できる、読みやすいドメイン固有言語を定義することで、概念をさらに詳しく説明します。

このラボでは、Azure Resource Manager テンプレートを使用して、新しい Azure Cosmos DB アカウント、データベース、およびコンテナーを作成します。 最初に未加工の JSON からテンプレートを作成してから、Bicep ドメイン固有言語を使用してテンプレートを作成します。

## <a name="prepare-your-development-environment"></a>開発環境を準備する

このラボで作業する環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、これらの手順に従って行います。 それ以外の場合は、以前にクローンされたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

   > &#128221; Visual Studio Code インターフェイスについてまだよく理解していない場合は、[Visual Studio Code の入門ガイド][code.visualstudio.com/docs/getstarted]を参照してください。

1. コマンド パレットを開き、**Git: Clone** を実行して、選択したローカル フォルダーに `https://github.com/microsoftlearning/dp-420-cosmos-db-dev` GitHub リポジトリをクローンします。

   > &#128161; **Ctrl + Shift + P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリが複製されたら、**Visual Studio Code** で選択したローカル フォルダーを開きます。

## <a name="create-azure-cosmos-db-sql-api-resources-using-azure-resource-manager-templates"></a>Azure Resource Manager テンプレートを使用して Azure Cosmos DB SQL API リソースを作成する

Azure Resource Manager の **Microsoft.DocumentDB** リソース プロバイダーを使用すると、JSON ファイルを使用してアカウント、データベース、およびコンテナーをデプロイできます。 ファイルは複雑な場合がありますが、予測可能な形式に従っており、Visual Studio Code 拡張機能を使用して書き込むことができます。

> &#128161; スタックしてしまい、テンプレートの構文エラーを理解できない場合は、この[ソリューションの Azure Resource Manager テンプレート][github.com/arm-template-guide]をガイドとして使用してください。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**31-create-container-arm-template** フォルダーを参照します。

1. **deploy.json** ファイルを開きます。

1. 空の Azure Resource Manager テンプレートを確認します。

   ```
   {
       "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "resources": [
       ]
   }
   ```

1. **resources** 配列内に、新しい JSON オブジェクトを追加して、新しい Azure Cosmos DB アカウントを作成します。

   ```
   {
       "type": "Microsoft.DocumentDB/databaseAccounts",
       "apiVersion": "2021-05-15",
       "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
       "location": "[resourceGroup().location]",
       "properties": {
           "databaseAccountOfferType": "Standard",
           "locations": [
               {
                   "locationName": "westus"
               }
           ]
       }
   }
   ```

   オブジェクトは、次の設定で構成されます。

   |                      **設定** | **Value**                                                |
   | ----------------------------: | :------------------------------------------------------- |
   |            **リソースの種類** | _Microsoft.DocumentDB/databaseAccounts_                  |
   |            **API バージョン** | _2021-05-15_                                             |
   |              **アカウント名** | _csmsarm_ &amp; _アカウント名から生成された一意の文字列_ |
   |                  **Location** | _リソース グループの現在の場所_                          |
   | **アカウント オファーの種類** | _Standard_                                               |
   |                      **場所** | _米国西部のみ_                                           |

1. **deploy.json** ファイルを保存します。

1. **31-create-container-arm-template** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

   > &#128221; このコマンドを実行すると、開始ディレクトリが **31-create-container-arm-template** フォルダーに既に設定されているターミナルが開きます。

1. 次のコマンドを使用して、Azure CLI の対話型ログイン プロシージャを開始します。

   ```
   az login
   ```

1. Azure CLI では、Web ブラウザーのウィンドウまたはタブが自動的に開き、ブラウザー インスタンス内で、サブスクリプションに関連付けられている Microsoft 資格情報を使用して Azure CLI にサインインします。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. 自分用のリソース グループをラボ プロバイダーが作成済みかどうかを確認します。それが済んでいる場合は、次のセクションで必要になるので、その名前をメモします。

   ```
   az group list --query "[].{ResourceGroupName:name}" -o table
   ```

   このコマンドを実行すると、複数のリソース グループ名を返すことができます。

1. (省略可能) **"リソース グループが作成されていない場合" は**、リソース グループ名を選んで作成します。\*\_ ラボ環境によってはロックされている場合があるので、自分用のリソース グループを管理者に作成してもらう必要があることに注意してください。

   i. この一覧から、自分に最も近い場所の名前を取得します。

   ```
   az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
   ```

   ii. リソース グループを作成します。 _ラボ環境によってはロックされている場合があるので、自分用のリソース グループを管理者に作成してもらう必要があることに注意してください。_

   ```
   az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
   ```

1. 次のコマンドを使用して、このラボで以前に作成または表示したリソース グループの名前を使用して、新しい変数名 **resourceGroup** を作成します。

   ```
   $resourceGroup="<resource-group-name>"
   ```

   > &#128221; たとえば、リソース グループの名前が **dp420** の場合、コマンドは **\$resourceGroup="dp420"** になります。

1. **echo** コマンドレットを使用して、次のコマンドを使用して **\$resourceGroup** 変数の値をターミナル出力に書き込みます。

   ```
   echo $resourceGroup
   ```

1. [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] コマンドを使用して Azure Resource Manager テンプレートをデプロイします。

   ```
   az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
   ```

1. 統合ターミナルを開いたままにして、**deploy.json** ファイルのエディターに戻ります。

1. **resources** 配列内に、別の新しい JSON オブジェクトを追加して、新しい Azure Cosmos DB SQL API データベースを作成します。

   ```
   ,
   {
       "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
       "apiVersion": "2021-05-15",
       "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
       "dependsOn": [
           "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
       ],
       "properties": {
           "resource": {
               "id": "cosmicworks"
           }
       }
   }
   ```

   オブジェクトは、次の設定で構成されます。

   |           **設定** | **Value**                                                                     |
   | -----------------: | :---------------------------------------------------------------------------- |
   | **リソースの種類** | _Microsoft.DocumentDB/databaseAccounts/sqlDatabases_                          |
   | **API バージョン** | _2021-05-15_                                                                  |
   |   **アカウント名** | _csmsarm_ &amp; _アカウント名から生成された一意の文字列_ &amp; _/cosmicworks_ |
   |    **リソース ID** | _cosmicworks_                                                                 |
   |       **依存関係** | _テンプレートで前に作成した databaseAccount_                                  |

1. **deploy.json** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して Azure Resource Manager テンプレートをデプロイします。

   ```
   az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
   ```

1. 統合ターミナルを開いたままにして、**deploy.json** ファイルのエディターに戻ります。

1. **resources** 配列内に、別の新しい JSON オブジェクトを追加して、新しい Azure Cosmos DB SQL API コンテナーを作成します。

   ```
   ,
   {
       "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
       "apiVersion": "2021-05-15",
       "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
       "dependsOn": [
           "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
           "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
       ],
       "properties": {
           "options": {
               "throughput": 400
           },
           "resource": {
               "id": "products",
               "partitionKey": {
                   "paths": [
                       "/categoryId"
                   ]
               }
           }
       }
   }
   ```

   オブジェクトは、次の設定で構成されます。

   |                **設定** | **Value**                                                                              |
   | ----------------------: | :------------------------------------------------------------------------------------- |
   |      **リソースの種類** | _Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers_                        |
   |      **API バージョン** | _2021-05-15_                                                                           |
   |        **アカウント名** | _csmsarm_ &amp; _アカウント名から生成された一意の文字列_ &amp; _/cosmicworks/products_ |
   |         **リソース ID** | _products_                                                                             |
   |        **スループット** | _400_                                                                                  |
   | **パーティション キー** | _/categoryId_                                                                          |
   |            **依存関係** | _テンプレートで先に作成したアカウントとデータベース_                                   |

1. **deploy.json** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して最後の Azure Resource Manager テンプレートをデプロイします。

   ```
   az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
   ```

1. 統合ターミナルを閉じます。

## <a name="observe-deployed-azure-cosmos-db-resources"></a>デプロイされた Azure Cosmos DB リソースを確認する

Azure Cosmos DB SQL API リソースがデプロイされると、Azure portal のリソースに移動できます。 データ エクスプローラーを使用して、アカウント、データベース、およびコンテナーがすべて正しくデプロイおよび構成されていることを検証します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (`portal.azure.com`) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選び、このラボで作成した **csmsarm** プレフィックスの付いた **Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、 **products** コンテナー ノードを確認します。

1. **products** コンテナー ノードを選択し、 **[Scale & Settings]** を選択します。

1. **[Scale]** セクション内の値を確認します。 具体的には、 **[Throughput]** セクションで **[Manual]** オプションが選択されており、プロビジョニングされたスループットが **400** RU/秒に設定されていることを確認します。

1. **[Settings]** セクション内の値を確認します。 具体的には、**パーティション キー** の値が **/categoryId** に設定されていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## <a name="create-azure-cosmos-db-sql-api-resources-using-bicep-templates"></a>Bicep テンプレートを使用して Azure Cosmos DB SQL API リソースを作成する

Bicep は、効率的なドメイン固有言語であり、Azure Resource Manager テンプレートよりも単純かつ簡単に Azure リソースをデプロイできます。 違いを説明するために、Bicep と別の名前を使用してまったく同じリソースをデプロイします。\[\]

> &#128161; スタックしてしまい、テンプレートの構文エラーを理解できない場合は、この[ソリューションの Bicep テンプレート][github.com/bicep-template-guide]をガイドとして使用してください。

1. **Visual Studio Code** の **[エクスプローラー]** ペインで、**31-create-container-arm-template** フォルダーを参照します。

1. 空の **deploy.bicep** ファイルを開きます。

1. ファイル内に、新しいオブジェクトを追加して、新しい Azure Cosmos DB アカウントを作成します。

   ```
   resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
     name: 'csmsbicep${uniqueString(resourceGroup().id)}'
     location: resourceGroup().location
     properties: {
       databaseAccountOfferType: 'Standard'
       locations: [
         {
           locationName: 'westus'
         }
       ]
     }
   }
   ```

   オブジェクトは、次の設定で構成されます。

   |                      **設定** | **Value**                                                |
   | ----------------------------: | :------------------------------------------------------- |
   |                **エイリアス** | _Account_                                             |
   |                      **名前** | _csmsbicep_ &amp; _アカウント名から生成された一意の文字列_ |
   |            **リソースの種類** | _Microsoft.DocumentDB/databaseAccounts/sqlDatabases_     |
   |            **API バージョン** | _2021-05-15_                                             |
   |                  **Location** | _リソース グループの現在の場所_                          |
   | **アカウント オファーの種類** | _Standard_                                               |
   |                      **場所** | _米国西部のみ_                                           |

1. **deploy.bicep** ファイルを保存します。

1. **31-create-container-arm-template** フォルダーのコンテキスト メニューを開き、 **[統合ターミナルで開く]** を選択して新しいターミナル インスタンスを開きます。

1. 次のコマンドを使用して、このラボで以前に作成または表示したリソース グループの名前を使用して、新しい変数名 **resourceGroup** を作成します。

   ```
   $resourceGroup="<resource-group-name>"
   ```

   > &#128221; たとえば、リソース グループの名前が **dp420** の場合、コマンドは **\$resourceGroup="dp420"** になります。

1. **az deployment group create** コマンドを使用して Bicep テンプレートをデプロイします。

   ```
   az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
   ```

1. 統合ターミナルを開いたままにして、**deploy.bicep** ファイルのエディターに戻ります。

1. ファイル内に、別の新しいオブジェクトを追加して、新しい Azure Cosmos DB データベースを作成します。

   ```
   resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
     parent: Account
     name: 'cosmicworks'
     properties: {
       resource: {
           id: 'cosmicworks'
       }
     }
   }
   ```

   オブジェクトは、次の設定で構成されます。

   |           **設定** | **Value**                                            |
   | -----------------: | :--------------------------------------------------- |
   |         **Parent** | _テンプレートで前に作成したアカウント_               |
   |     **エイリアス** | _Database_                                     |
   |           **名前** | _cosmicworks_                                        |
   | **リソースの種類** | _Microsoft.DocumentDB/databaseAccounts/sqlDatabases_ |
   | **API バージョン** | _2021-05-15_                                         |
   |    **リソース ID** | _cosmicworks_                                        |

1. **deploy.bicep** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して Bicep テンプレートをデプロイします。

   ```
   az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
   ```

1. 統合ターミナルを開いたままにして、**deploy.bicep** ファイルのエディターに戻ります。

1. ファイル内に、別の新しいオブジェクトを追加して、新しい Azure Cosmos DB コンテナーを作成します。

   ```
   resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
     parent: Database
     name: 'products'
     properties: {
       options: {
         throughput: 400
       }
       resource: {
         id: 'products'
         partitionKey: {
           paths: [
             '/categoryId'
           ]
         }
       }
     }
   }
   ```

   オブジェクトは、次の設定で構成されます。

   |                      **設定** | **Value**                                |
   | ----------------------------: | :--------------------------------------- |
   |                    **Parent** | _テンプレートで前に作成したデータベース_ |
   |                **エイリアス** | _Container_                             |
   |                      **名前** | _products_                               |
   |               **リソース ID** | _products_                               |
   |              **スループット** | _400_                                    |
   | **パーティション キーのパス** | _/categoryId_                            |

1. **deploy.bicep** ファイルを保存します。

1. 統合ターミナルに戻ります。

1. **az deployment group create** コマンドを使用して最後の Bicep テンプレートをデプロイします。

   ```
   az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
   ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

## <a name="observe-bicep-template-deployment-results"></a>Bicep テンプレートのデプロイ結果を確認する

Bicep デプロイは、Azure Resource Manager デプロイと同じ手法の多くを使用して検証できます。 アカウント、データベース、コンテナーが正常にデプロイされたことを検証するだけでなく、6 つのデプロイすべてにわたってデプロイ履歴も表示します。

1. 新しい Web ブラウザー ウィンドウまたはタブで、Azure portal (`portal.azure.com`) に移動します。

1. ご利用のサブスクリプションに関連付けられている Microsoft 資格情報を使用して、ポータルにサインインします。

1. **[リソース グループ]** を選択し、このラボで先ほど作成または表示したリソース グループを選択します。

1. リソース グループ内で、 **[デプロイ]** ペインに移動します。

1. Azure Resource Manager テンプレートと Bicep ファイルからの 6 つのデプロイを確認します。

1. 引き続きリソース グループ内で、 **[概要]** ペインに移動します。

1. 引き続きリソース グループ内で、このラボで作成した **csmsbicep** プレフィックスがついた**Azure Cosmos DB アカウント** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、 **[データ エクスプローラー]** ペインに移動します。

1. **[データ エクスプローラー]** で、**cosmicworks** データベース ノードを展開し、 **products** コンテナー ノードを確認します。

1. **products** コンテナー ノードを選択し、 **[Scale & Settings]** を選択します。

1. **[Scale]** セクション内の値を確認します。 具体的には、 **[Throughput]** セクションで **[Manual]** オプションが選択されており、プロビジョニングされたスループットが **400** RU/秒に設定されていることを確認します。

1. **[Settings]** セクション内の値を確認します。 具体的には、**パーティション キー** の値が **/categoryId** に設定されていることを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/cli/azure/deployment/group]: https://docs.microsoft.com/cli/azure/deployment/group
[github.com/arm-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.json
[github.com/bicep-template-guide]: https://raw.githubusercontent.com/Sidney-Andrews/acdbsad/solutions/31-create-container-arm-template/deploy.bicep
