---
redirect_url: /azure/sql-data-warehouse/sql-data-warehouse-load-with-data-factory
title: "Azure Blob Storage から Azure SQL Data Warehouse へのデータの読み込み (Azure Data Factory) | Microsoft Docs"
description: "Azure Data Factory を使用してデータを読み込む方法"
services: sql-data-warehouse
documentationcenter: NA
author: barbkess
manager: jhubbard
editor: 
tags: azure-sql-data-warehouse
ms.assetid: 689fb02e-eb98-4f7c-81e6-6c1d22d53901
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: data-services
ms.date: 11/22/2016
ms.author: barbkess
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: bf9ea4bd9fde1cc534c58e32706b659e5d89e4f7


---
# <a name="load-data-from-azure-blob-storage-into-azure-sql-data-warehouse-azure-data-factory"></a>Azure Blob Storage から Azure SQL Data Warehouse へのデータの読み込み (Azure Data Factory)
> [!div class="op_single_selector"]
> * [Data Factory](sql-data-warehouse-load-from-azure-blob-storage-with-data-factory.md)
> * [PolyBase](sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md)
> 
> 

 このチュートリアルでは、Azure Storage BLOB から SQL Data Warehouse にデータを移動するパイプラインを Azure Data Factory で作成する方法について説明します。 以降の手順では、次の操作を実行します。

* Azure Storage Blob にサンプルのデータを設定する。
* Azure Data Factory にリソースを接続する。
* Storage の BLOB から SQL Data Warehouse にデータを移動するためのパイプラインを作成する。

> [!VIDEO https://channel9.msdn.com/Blogs/Windows-Azure/Loading-Azure-SQL-Data-Warehouse-with-Azure-Data-Factory/player]
> 
> 

## <a name="before-you-begin"></a>開始する前に
Azure Data Factory を理解するには、[Azure Data Factory の概要][Azure Data Factory の概要]に関する記事を参照してください。

### <a name="create-or-identify-resources"></a>リソースを作成または識別する
このチュートリアルを開始する前に、次のリソースを用意する必要があります。

* **Azure Storage BLOB**: このチュートリアルでは、Azure Data Factory パイプラインのデータ ソースとして Azure Storage BLOB を使用します。そのため、サンプル データを格納できる Azure Storage BLOB が必要です。 まだお持ちでない場合は、「[Create a storage account (ストレージ アカウントの作成)][ストレージ アカウントの作成]」を参照してください。
* **SQL Data Warehouse**: このチュートリアルでは、Azure Storage BLOB から SQL Data Warehouse にデータを移動します。したがって、AdventureWorksDW サンプル データを読み込むデータ ウェアハウスをオンラインにする必要があります。 データ ウェアハウスがまだない場合は、[プロビジョニングする][SQL Data Warehouse の作成]方法を学習してください。 データ ウェアハウスはあっても、サンプル データでプロビジョニングしなかった場合は、[手動で読み込む][SQL Data Warehouse へのサンプル データのロード]ことができます。
* **Azure Data Factory**: Azure Data Factory によって実際の読み込みが実行されるので、データ移動パイプラインの作成に使用できるデータ ファクトリが必要です。まだデータ ファクトリがない場合は、[Azure Data Factory (Data Factory Editor) の概要][Data Factory Editor を使用した初めての Azure Data Factory パイプラインの作成] に関するページの手順 1. でその作成方法を確認してください。
* **AZCopy**: サンプル データをローカル クライアントから Azure Storage BLOB にコピーするには、AZCopy が必要です。 インストールの手順については、[AZCopy のドキュメント][AZCopy のドキュメント]を参照してください。

## <a name="step-1-copy-sample-data-to-azure-storage-blob"></a>手順 1: サンプル データを Azure Storage BLOB にコピーする
すべてのピースが揃ったら、サンプル データを Azure Storage BLOB にいつでもコピーすることができます。

1. [サンプル データをダウンロードします][サンプル データをダウンロードします]。 このデータにより、別の 3 年分の売上データが AdventureWorksDW サンプル データに追加されます。
2. 次の AZCopy コマンドを使用して、3 年分のデータを Azure Storage BLOB にコピーします。

````
AzCopy /Source:<Sample Data Location>  /Dest:https://<storage account>.blob.core.windows.net/<container name> /DestKey:<storage key> /Pattern:FactInternetSales.csv
````


## <a name="step-2-connect-resources-to-azure-data-factory"></a>手順 2: Azure Data Factory にリソースを接続する
これでデータの準備ができたので、Azure Data Factory パイプラインを作成して、データを Azure Storage BLOB から SQL Data Warehouse に移動することができます。

そのためには、まず、[Azure ポータル][Azure ポータル] を開き、左側のメニューで対象のデータ ファクトリを選択します。

### <a name="step-21-create-linked-service"></a>手順 2.1: リンクされたサービスを作成する
Azure ストレージ アカウントと SQL Data Warehouse をデータ ファクトリにリンクします。  

1. まず、データ ファクトリの [リンクされたサービス] セクション、[新しいデータ ストア] の順にクリックして、登録プロセスを開始します。 次に、Azure ストレージを登録する名前を選択します。種類として [Azure Storage] を選択し、アカウント名とアカウント キーを入力します。
2. SQL Data Warehouse を登録するには、[作成とデプロイ] セクションに移動して [新しいデータ ストア] を選択し、[Azure SQL Data Warehouse] を選択します。 コピーおよび貼り付けを行って、このテンプレートに特定の情報を入力します。

```JSON
{
    "name": "<Linked Service Name>",
    "properties": {
        "description": "",
        "type": "AzureSqlDW",
        "typeProperties": {
             "connectionString": "Data Source=tcp:<server name>.database.windows.net,1433;Initial Catalog=<server name>;Integrated Security=False;User ID=<user>@<servername>;Password=<password>;Connect Timeout=30;Encrypt=True"
         }
    }
}
```

### <a name="step-22-define-the-dataset"></a>手順 2.2: データ ソースを定義する
リンクされたサービスを作成した後は、データ セットを定義する必要があります。  これは、ストレージからデータ ウェアハウスに移動するデータの構造を定義することを意味します。  作成に関する詳しい情報を得ることができます。

1. データ ファクトリの [作成とデプロイ] セクションに移動し、このプロセスを開始します。
2. [新しいデータセット]、[Azure BLOB ストレージ] の順にクリックし、ストレージをデータ ファクトリにリンクします。  次のスクリプトを使用して、Azure BLOB ストレージのデータを定義することができます。

```JSON
{
    "name": "<Dataset Name>",
    "properties": {
        "type": "AzureBlob",
        "linkedServiceName": "<linked storage name>",
        "typeProperties": {
            "folderPath": "<containter name>",
            "fileName": "FactInternetSales.csv",
            "format": {
            "type": "TextFormat",
            "columnDelimiter": ",",
            "rowDelimiter": "\n"
            }
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval": 1
        },
        "policy": {
            "externalData": {
                "retryInterval": "00:01:00",
                "retryTimeout": "00:10:00",
                "maximumRetry": 3
            }
        }
    }
}
```


1. 次に、SQL Data Warehouse のデータセットも定義します。  前の手順と同様に、[新しいデータセット]、[Azure SQL Data Warehouse] の順にクリックします。

```JSON
{
    "name": "DWDataset",
    "properties": {
        "type": "AzureSqlDWTable",
        "linkedServiceName": "AzureSqlDWLinkedService",
        "typeProperties": {
            "tableName": "FactInternetSales"
        },
        "availability": {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

## <a name="step-3-create-and-run-your-pipeline"></a>手順 3: パイプラインを作成して実行する
最後に、Azure Data Factory でパイプラインを設定して実行します。  これは、実際のデータの移動を実行する操作です。  SQL Data Warehouse と Azure Data Factory で実行できるすべての操作については、[こちら][Azure Data Factory を使用した Azure SQL Data Warehouse との間でのデータの移動]を参照してください。

[作成とデプロイ] セクションで、[その他のコマンド]、[新しいパイプライン] の順にクリックします。  パイプラインを作成した後、次のコードを使用して、データ ウェアハウスにデータを転送できます。

```JSON
{
    "name": "<Pipeline Name>",
    "properties": {
        "description": "<Description>",
        "activities": [
          {
            "type": "Copy",
            "typeProperties": {
                "source": {
                    "type": "BlobSource",
                    "skipHeaderLineCount": 1
                },
                "sink": {
                    "type": "SqlDWSink",
                    "writeBatchSize": 0,
                    "writeBatchTimeout": "00:00:10"
                }
            },
            "inputs": [
              {
                "name": "<Storage Dataset>"
              }
            ],
            "outputs": [
              {
                "name": "<Data Warehouse Dataset>"
              }
            ],
            "policy": {
                "timeout": "01:00:00",
                "concurrency": 1
            },
            "scheduler": {
                "frequency": "Hour",
                "interval": 1
            },
            "name": "Sample Copy",
            "description": "Copy Activity"
          }
        ],
        "start": "<Date YYYY-MM-DD>",
        "end": "<Date YYYY-MM-DD>",
        "isPaused": false
    }
}
```

## <a name="next-steps"></a>次のステップ
詳細については、まず以下の情報を参照してください。

* [Azure Data Factory のラーニング パス][Azure Data Factory のラーニング パス]。
* [Azure SQL Data Warehouse コネクタ][Azure SQL Data Warehouse コネクタ]。 これは、Azure Data Factory と Azure SQL Data Warehouse を組み合わせて使用するための主要な参照トピックとなります。

これらのトピックでは、Azure Data Factory に関する詳細情報を提供します。 Azure SQL Database または HDinsight について説明しますが、この情報は Azure SQL Data Warehouse にも該当します。

* [チュートリアル: Azure Data Factory を使ってみる][チュートリアル: Azure Data Factory を使ってみる]。これは、Azure Data Factory を使用してデータを処理するための主要なチュートリアルです。 このチュートリアルでは、HDInsight を使用して Web ログの変換および分析を毎月行う初めてのパイプラインを作成します。 なお、このチュートリアルには、コピー アクティビティはありません。
* [チュートリアル: Azure Storage BLOB から Azure SQL Database へのデータのコピー][チュートリアル: Azure Storage BLOB から Azure SQL Database へのデータのコピー]。 このチュートリアルでは、Azure Data Factory でパイプラインを作成し、データを Azure Storage BLOB から Azure SQL Database にコピーします。

<!--Image references-->

<!--Article references-->
[AZCopy のドキュメント]: ../storage/storage-use-azcopy.md
[Azure SQL Data Warehouse コネクタ]: ../data-factory/data-factory-azure-sql-data-warehouse-connector.md
[BCP]: sql-data-warehouse-load-with-bcp.md
[SQL Data Warehouse の作成]: sql-data-warehouse-get-started-provision.md
[ストレージ アカウントの作成]: ../storage/storage-create-storage-account.md#create-a-storage-account
[Data Factory]: sql-data-warehouse-get-started-load-with-azure-data-factory.md
[Data Factory Editor を使用した初めての Azure Data Factory パイプラインの作成]: ../data-factory/data-factory-build-your-first-pipeline-using-editor.md
[Azure Data Factory の概要]: ../data-factory/data-factory-introduction.md
[SQL Data Warehouse へのサンプル データのロード]: sql-data-warehouse-load-sample-databases.md
[Azure Data Factory を使用した Azure SQL Data Warehouse との間でのデータの移動]: ../data-factory/data-factory-azure-sql-data-warehouse-connector.md
[PolyBase]: sql-data-warehouse-get-started-load-with-polybase.md
[チュートリアル: Azure Storage BLOB から Azure SQL Database へのデータのコピー]: ../data-factory/data-factory-copy-data-from-azure-blob-storage-to-sql-database.md
[チュートリアル: Azure Data Factory を使ってみる]: ../data-factory/data-factory-build-your-first-pipeline.md

<!--MSDN references-->

<!--Other Web references-->
[Azure Data Factory のラーニング パス]: https://azure.microsoft.com/documentation/learning-paths/data-factory
[Azure ポータル]: https://portal.azure.com
[サンプル データをダウンロードします]: https://migrhoststorage.blob.core.windows.net/adfsample/FactInternetSales.csv



<!--HONumber=Nov16_HO3-->


