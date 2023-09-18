# AzureResourceManager-knowledge

## AzureResourceManagerとは

Azureのデプロイおよび管理サービスで、 Azureのリソースのデプロイ及び管理をコードを用いて実現ができるサービスです。そのコードのことをARMテンプレートといい、今回はこのテンプレートの記述方法について学習していきます。

## ARMテンプレートの動作検証

動作検証は、CloudShellの環境を使って行っていきます。ここでテンプレートファイルの格納・デプロイの実施をします。

![AzureCloudShell](/img/CloudShell.png)

まずデプロイ処理ができることを確認します。以下の内容でテンプレートファイルを作成し、実行します。

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": []
}
```

![TemplateFile](/img/templatefile01.png)

指定したリソースグループにテンプレートを使用してリソースを作るためには以下のコマンドを使用します。

[公式ドキュメント](https://learn.microsoft.com/ja-jp/powershell/module/az.resources/new-azresourcegroupdeployment?view=azps-10.3.0)

```
$templatefile = "./clouddrive/template01.json"
$RG = "armtemplatetest"
New-AzResourceGroupDeployment -ResourceGroupName $RG -TemplateFile $templatefile
```

![Result01](/img/result01.png)

デプロイの結果をPortal上で確認するには、リソースグループ > デプロイから確認することができます。

![Log01](/img/log01.png)

今回はテンプレートファイル内の「resources」を空欄で実行したため、何も生成しないテンプレートを実行したということになります。

次のデプロイではストレージアカウントなどのリソースを生成するテンプレートを作成して検証を行います。

## ARMテンプレートによるストレージアカウントのデプロイ

ARMテンプレートを使用した動作確認は完了したので、実際に意図したリソースの作成が行えるテンプレートファイルを作成してみます。

テンプレート作成時には[Azureクイックスタートテンプレート](https://azure.microsoft.com/resources/templates/)から対象のリソースを検索します。

![codesample](/img/codesample.png)

リソース画面に遷移後に、GitHubのボタンを選択します。

![ResourceTemplate](/img/resourcetemplate.png)

フォルダ内のazuredeploy.jsonにリソースを作成するためのテンプレートが記載されているので、これを参考にARMテンプレートを作成します。

![AzureDeploy](/img/azuredeploy.png)

![AzureDeploy](/img/azuredeploy.png)

以下が参考にして記述したテンプレートファイルになります。

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-09-01",
      "name": "nakasetestjpeast01",
      "location": "japaneast",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
          "supportsHttpsTrafficOnly": true
      }
    }
  ]
}
```

![Result2](/img/result02.png)

デプロイの履歴から、テンプレートファイルによってストレージアカウントがデプロイできていることがわかります。

![StorageAccount01](/img/stgaccount.png)

デプロイは上記のテンプレートファイルで十分に行えますが、ストレージアカウントの名前は一意である必要があるため、このままではテンプレートファイルの使いまわしができず、コード化した意味がありません。

上記のテンプレートを下記の通りに修正します。

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageName": {
        "type": "string",
        "minLength": 3,
        "maxLength": 24
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-09-01",
      "name": "[parameters('StorageName')]",
      "location": "japaneast",
      "sku": {
        "name": "Standard_LRS"
      },
      "kind": "StorageV2",
      "properties": {
          "supportsHttpsTrafficOnly": true
      }
    }
  ]
}
```

新たにparametersプロパティを追加し、ストレージアカウント名をファイルの外から参照できるようになりました。

以下の様にコマンド実行時に指定することで参照することができます。

```
New-AzResourceGroupDeployment -ResourceGroupName $RG -TemplateFile $templatefile -StorageName "nakasetestjpeast02"
```

先程同様、リソースのデプロイが完了しました。

![StorageAccount02](/img/stgaccount02.png)

デプロイ履歴の「入力」からパラメータの入力内容について確認ができます。今回は「storageName」に「nakasetestjpeast02」を指定してデプロイを行ったことが確認できました。

![Input01](/img/input01.png)


## ARMテンプレートによるリソースの更新

前節まではストレージアカウントが生成できることを確認してきました。ここでは、ARMテンプレート内のリソース名を同じ名前でデプロイを実施した場合、リソース情報が更新されることを検証します。

以下のリソースの冗長性の設定を変更してみます。LRSからGRSに変更できるようにテンプレートファイルを修正します。

```
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "StorageName": {
        "type": "string",
        "minLength": 3,
        "maxLength": 24
    }
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-09-01",
      "name": "[parameters('StorageName')]",
      "location": "japaneast",
      "sku": {
        "name": "Standard_GRS"
      },
      "kind": "StorageV2",
      "properties": {
          "supportsHttpsTrafficOnly": true
      }
    }
  ]
}
```

skuの項目を「Standard_GRS」に変更しました。

**更新前**
![Update01](/img/update01.png)

**更新後**
![Update02](/img/update02.png)

デプロイ後にちゃんと冗長性の項目が更新されていることが確認できました。同じ名前のリソースでテンプレートファイルを実行すると、新たにリソースが生成されることはなく、リソース情報が更新されるようです。



