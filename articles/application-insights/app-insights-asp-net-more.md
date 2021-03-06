---
title: "Application Insights をさらに活用する | Microsoft Docs"
description: "Application Insights の利用開始後に使用できる機能の概要を示します。"
services: application-insights
documentationcenter: .net
author: alancameronwills
manager: douge
ms.assetid: 7ec10a2d-c669-448d-8d45-b486ee32c8db
ms.service: application-insights
ms.workload: tbd
ms.tgt_pltfrm: ibiza
ms.devlang: na
ms.topic: article
ms.date: 10/27/2016
ms.author: awills
translationtype: Human Translation
ms.sourcegitcommit: e1f9c62765e414e194330419f26c2b7437da21b3
ms.openlocfilehash: 79cfe90bb883b5cadf707272d31a990c8886dbe4


---
# <a name="more-telemetry-from-application-insights"></a>Application Insights からのテレメトリの追加
[Application Insights を ASP.NET コードに追加](app-insights-asp-net.md)した後、さらに多くのテレメトリを取得するためにできることがいくつかあります。 

## <a name="if-your-app-runs-on-your-iis-server-"></a>アプリが IIS サーバーで実行されている場合
管理下の IIS サーバーでアプリがホストされている場合は、そのサーバーに Application Insights Status Monitor をインストールしてください。 既にインストールされている場合は、何もする必要はありません。

1. 各 IIS Web サーバーで、管理者の資格情報を使用してサインインします。
2. [Status Monitor インストーラー](http://go.microsoft.com/fwlink/?LinkId=506648)をダウンロードし、実行します。
3. インストール ウィザードで、Microsoft Azure にサインインします。

それ以外に何もする必要はありませんが、アプリで監視が有効になっていることを確認できます。

![Extend in Azure](./media/app-insights-asp-net-more/025.png)

(Visual Studio でアプリをインストルメント化していない場合でも、Status Monitor を使用して、[実行時の監視を有効にする](app-insights-monitor-performance-live-website-now.md)こともできます。)

### <a name="what-do-you-get"></a>取得できるもの
Status Monitor をサーバー コンピューターにインストールすると、いくつかの追加のテレメトリを取得できます。

* .NET 4.5 アプリの依存関係テレメトリ (アプリによって行われた SQL 呼び出しと REST 呼び出し)  (.NET の以降のバージョンでは、依存関係テレメトリに Status Monitor は必要ありません)。 
* 例外スタック トレースで、詳細情報が表示されます。
* パフォーマンス カウンター。 Application Insights では、これらのカウンターは、[サーバー] ブレードに表示されます。 

![Extend in Azure](./media/app-insights-asp-net-more/070.png)

表示されるカウンターを増減するには、 [グラフを編集](app-insights-metrics-explorer.md)します。 必要なパフォーマンス カウンターが使用可能なセットに含まれていない場合は、[パフォーマンス カウンター モジュールによって収集されるセットに追加](app-insights-performance-counters.md)できます。

## <a name="if-its-an-azure-web-app-"></a>Azure Web アプリの場合
アプリが Azure Web アプリとして実行される場合は、アプリまたは VM の Azure コントロール パネルに移動し、Application Insights ブレードを開きます。 

### <a name="what-do-you-get"></a>取得できるもの
* 例外スタック トレースで、詳細情報が表示されます。
* .NET 4.5 アプリの依存関係テレメトリ (アプリによって行われた SQL 呼び出しと REST 呼び出し)  (.NET の以降のバージョンでは、依存関係テレメトリに拡張機能は必要ありません)。 

![Extend in Azure](./media/app-insights-asp-net-more/080.png)

(Visual Studio でアプリをインストルメント化していない場合でも、この方法を使用して、[実行時のパフォーマンス監視を有効にする](app-insights-monitor-performance-live-website-now.md)こともできます。)

## <a name="client-side-monitoring"></a>クライアント側の監視
アプリケーションのサーバー側 (バックエンド) からテレメトリ データを送信する SDK を既にインストールしています。 このため、クライアント側の監視を追加することができます。 これにより、ユーザー、セッション、ページ ビュー、およびブラウザーで発生する例外やクラッシュに関するデータを入手できます。 また、独自のコードを記述して、ユーザーのアプリの操作をクリックやキーボード操作までの細部にわたって追跡できます。

クライアントのブラウザーからテレメトリを取得するには、それぞれの Web ページに Application Insights の JavaScript スニペットを追加します。

1. Azure で、アプリの Application Insights リソースを開きます。
2. [作業の開始]、[MONITOR AND DIAGNOSE CLIENT SIDE APPLICATION (クライアント側アプリケーションの監視と診断)] の順に開き、スニペットをコピーします。
3. コピーしたスニペットを、各 Web ページの先頭に表示されるように貼り付けます。通常、これを行うには、マスター レイアウト ページに貼り付けます。

![Extend in Azure](./media/app-insights-asp-net-more/100.png)

コードにはアプリケーション リソースを識別するインストルメンテーション キーが含まれています。

### <a name="what-do-you-get"></a>取得できるもの
* ボタンのクリックの追跡など、 [Web ページからカスタム テレメトリ](app-insights-api-custom-events-metrics.md)を送信する JavaScript を記述できます。
* [Analytics](app-insights-analytics.md) では、`pageViews` のデータと `dependencies` の AJAX データ。 
* [クライアント パフォーマンスと使用状況データ](app-insights-javascript.md) 。

![Extend in Azure](./media/app-insights-asp-net-more/090.png)

[Web ページの追跡についてはこちらをご覧ください。](app-insights-web-track-usage.md)

## <a name="track-application-version"></a>アプリケーションのバージョンを追跡する
MSBuild プロセスで `buildinfo.config` が生成されていることを確認します。 .csproj ファイルに、次のコードを追加します。  

```XML

    <PropertyGroup>
      <GenerateBuildInfoConfigFile>true</GenerateBuildInfoConfigFile>    <IncludeServerNameInBuildInfo>true</IncludeServerNameInBuildInfo>
    </PropertyGroup> 
```

ビルド情報がある場合、Application Insights Web モジュールは、 **アプリケーションのバージョン** をプロパティとしてテレメトリのすべての項目に自動的に追加します。 これにより、[診断の検索](app-insights-diagnostic-search.md)を実行するとき、または[メトリックを調べる](app-insights-metrics-explorer.md)ときに、バージョンによってフィルター処理できます。 

ただし、Visual Studio で開発者向けのビルドではなく、MS ビルドでのみビルド バージョン番号が生成されることに注意してください。

## <a name="availability-web-tests"></a>可用性 Web テスト
世界中から定期的に Web アプリの HTTP 要求を送信してください。 応答が低速または低い信頼性の場合は、アラートが発行されます。

アプリの Application Insights リソースで Web テストを追加、編集、および表示するには、[可用性] タイルをクリックします。

複数の場所で実行される複数のテストを追加することができます。

![Extend in Azure](./media/app-insights-asp-net-more/110.png)

[詳細情報](app-insights-monitor-web-app-availability.md)

## <a name="custom-telemetry-and-logging"></a>カスタム テレメトリとログ記録
コードに追加した Application Insights パッケージには、アプリケーションから呼び出すことができる API が用意されています。

* [独自のイベントとメトリックを生成](app-insights-api-custom-events-metrics.md)し、ビジネス イベントのカウントやパフォーマンスの監視などを実行します。
* [ログ トレースをキャプチャします](app-insights-asp-net-trace-logs.md) 。
* [フィルター、変更、または強化します](app-insights-api-filtering-sampling.md) 。 

## <a name="powerful-analysis-and-presentation"></a>強力な分析とプレゼンテーション
データを探索する方法は多数あります。 Application Insights を最近使い始めた場合は、以下の記事を参照してください。

|  |  |
| --- | --- |
| [**インスタンスのデータの診断検索**](app-insights-visual-studio.md)<br/>要求、例外、依存関係の呼び出し、ログ トレースおよびページ ビューなどのイベントを検索およびフィルター処理します。 Visual Studio では、スタック トレースからコードに移動します。 |![Visual studio](./media/app-insights-asp-net-more/61.png) |
| [**集計データのメトリックス エクスプ ローラー**](app-insights-metrics-explorer.md)<br/>要求、失敗、および例外の比率、応答時間、ページの読み込み時間などの集計データを調査、フィルター処理、およびセグメント分割します。 |![Visual studio](./media/app-insights-asp-net-more/060.png) |
| [**ダッシュボード**](app-insights-dashboards.md#dashboards)<br/>複数のリソースからのデータをマッシュアップし、他のユーザーと共有します。 複数コンポーネントのアプリケーションと、チーム ルームでの継続的な表示に最適です。 |![ダッシュボードのサンプル](./media/app-insights-asp-net-more/62.png) |
| [**ライブ メトリック ストリーム**](app-insights-metrics-explorer.md#live-metrics-stream)<br/>新しいビルドをデプロイする場合、このほぼリアルタイムのパフォーマンス インジケーターを監視し、すべてが期待どおりに動作することを確認します。 |![分析のサンプル](./media/app-insights-asp-net-more/050.png) |
| [**分析**](app-insights-analytics.md)<br/>この強力なクエリ言語を使用して、アプリのパフォーマンスと使用状況に関する難しい質問に回答します。 |![分析のサンプル](./media/app-insights-asp-net-more/010.png) |
| [**自動および手動のアラート**](app-insights-alerts.md)<br/>アプリのテレメトリの通常パターンに対して自動アラートを適応し、通常とは異なるパターンがある場合にアラートをトリガーします。 カスタムまたは標準のメトリックスの特定レベルでアラートを設定することもできます。 |![アラートのサンプル](./media/app-insights-asp-net-more/020.png) |

## <a name="data-management"></a>データ管理
|  |  |
| --- | --- |
| [**連続エクスポート**](app-insights-export-telemetry.md)<br/>独自の方法で分析できるように、すべてのテレメトリをストレージにコピーします。 | |
| **データ アクセス API**<br/>近日対応予定。 | |
| [**サンプリング**](app-insights-sampling.md)<br/>データ速度を削減し、価格レベルの制限内で維持できます。 |![サンプリング タイル](./media/app-insights-asp-net-more/030.png) |




<!--HONumber=Dec16_HO3-->


