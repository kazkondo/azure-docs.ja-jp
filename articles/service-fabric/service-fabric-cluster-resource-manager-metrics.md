---
title: "Azure Service Fabric クラスター リソース マネージャーでメトリックを管理する | Microsoft Docs"
description: "Service Fabric 内でメトリックを構成し使用する方法について説明します。"
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: 
ms.assetid: 0d622ea6-a7c7-4bef-886b-06e6b85a97fb
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/19/2016
ms.author: masnider
translationtype: Human Translation
ms.sourcegitcommit: dcda8b30adde930ab373a087d6955b900365c4cc
ms.openlocfilehash: ad2588b75e41da25e5c76855849f0086ed3aa14f


---
# <a name="managing-resource-consumption-and-load-in-service-fabric-with-metrics"></a>Service Fabric のリソース使用量と負荷をメトリックで管理する
メトリックとは、サービスが動作するために必要であり、クラスター内のノードによって提供されるリソースを表す、Service Fabric の一般的な用語です。 一般的に、メトリックは、サービスのパフォーマンスを調整するために管理する必要があるすべての要素を指します。

メモリ、ディスク、CPU 使用率など、 これらはすべてのメトリックの例です。 これは物理的なメトリック、つまり管理する必要があるノード上の物理リソースに対応するリソースです。 メトリックには論理的なメトリックも (一般的に) あります。たとえば、アプリケーションで定義され、特定のレベルのリソース消費に対応する (ただし、アプリケーションからはそれを実際には認識できない、またはそれを測定するがない)、"MyWorkQueueDepth" のような要素が考えられます。 ユーザーが使うほとんどのメトリックは論理メトリックです。 これにはさまざまな理由が考えられますが、最も一般的な理由としては、今日、多くのお客様がマネージド コードでサービスを書き込んでおり、特定のステートレス サービス インスタンスやステートフル サービス レプリカ オブジェクトからは実際の物理リソースの消費の測定が困難なことが挙げられます。 独自のメトリックをレポートすることの複雑さも、構成済みの既定のメトリックを提供している理由の 1 つです。

## <a name="default-metrics"></a>Default metrics
たとえば、今後利用するリソース、または重要になりそうなリソースがわからないまま、作業を開始する必要があるとします。 そこで、メトリックを指定せずに、とりあえず実装してからサービスを作成することにします。 それでも、何も問題ありません。 メトリックは自動的に選択されます。 現在、ユーザーが独自のメトリックを指定しなかった場合に自動的に適用される既定のメトリックは、PrimaryCount、ReplicaCount、および Count です (Count という名前がまぎらわしいという問題は、こちらでも認識しています)。 次の表に、各メトリックの負荷がどの程度各サービス オブジェクトに関連付けられているかについて示します。

| メトリック | ステートレス インスタンス負荷 | ステートフル セカンダリ負荷 | ステートフル プライマリ負荷 |
| --- | --- | --- | --- |
| PrimaryCount |0 |0 |1 |
| ReplicaCount |0 |1 |1 |
| Count |1 |1 |1 |

では、この既定のメトリックを利用すると、どのようなメリットがあるでしょうか。 基本的なワークロードについては、適切に処理を分散できるようになります。 次の例では、3 つのパーティションを含む 1 つのステートフル サービス、大きさ 3 の対象レプリカ セット、3 つのインスタンスを含む 1 つのステートレス サービスを作成します。すると、次のような環境が得られます。

![Cluster Layout with Default Metrics][Image1]

この例を見ると、

* ステートフル サービスに対応するプライマリ レプリカが 1 つのノードにスタックされない
* 同じパーティションのレプリカが同じノードに置かれていない
* プライマリおよびセカンダリの数がクラスター内で均等に分散されている
* サービス オブジェクト (ステートレスおよびステートフル) がすべて各ノードに均等に割り当てられている

優秀な結果です。  

すばらしい環境ですが、選択したパーティション分割構成によって、完全に均等にすべてのパーティションで使用されるようになるのでしょうか。 さらに、特定のサービスに対する負荷が長期間一定であるか、少なくとも今と同じである可能性はどれくらいあるのでしょうか。 現実的なワークロードの場合、同等であるすべてのレプリカのオッズは実際には低いと言えます。そのため、クラスターを最大限に活用するには、カスタム メトリックを検討することをお勧めします。

現実的には、確かに既定のメトリックのみを使用して実行することはできますが、そうした場合、通常はクラスターの使用率が想定よりも低くなります (レポートに適応性がなく、すべてが同等であると想定されるためです)。最悪の場合、一部のノードに負荷の割り当てが集中し、パフォーマンスが低下することもあります。 この問題は、次に説明する、カスタム メトリックと動的な負荷レポートを使用することで改善できます。

## <a name="custom-metrics"></a>カスタム メトリック
これまで、物理的なメトリックと論理的なメトリックの両方を使用できること、およびユーザーが独自のメトリックを定義できることについて説明してきました。 そこで、 では、独自のメトリックはどのように定義できるでしょうか。 難しいことではありません。 サービスを作成するときに、メトリックと既定の初期負荷を構成するだけです。 消費すると想定されるサービス量を表すメトリックと既定値のセットは、サービスを作成するときに、名前付きサービス インスタンスごとに構成できます。

サービスの負荷分散も行う必要がある場合には、カスタム メトリックの定義を開始するときに、既定のメトリックを明示的に再適用する必要があることに注意してください。 これは、既定のメトリックとカスタム メトリックの関係についてユーザーの意図を明確に示す必要があるからです。ユーザーによっては、プライマリの分散よりもメモリの消費や WorkQueueDepth を重視することがあります。

たとえば、既定のメトリックに加えて、"Memory" という名前のメトリックをレポートするようにサービスを構成するとします。 Memory については、基本的な測定を何度か行ったことがあり、通常はそのサービスのプライマリ レプリカが 20 MB、同じサービスのセカンダリ レプリカが 5 MB  のメモリを使用することを把握しているとします。 Memory はこの特定のサービスのパフォーマンスを管理するうえで最も重要なメトリックであることはわかっていますが、一部のノードまたは障害ドメインが失われたときに膨大な数のプライマリ レプリカが停止する事態を避けるために、プライマリ レプリカの負荷も調整する必要があるとします。 それ以外については、既定のメトリックを採用するとします。

このような場合、次のようにします。

コード:

```csharp
StatefulServiceDescription serviceDescription = new StatefulServiceDescription();
StatefulServiceLoadMetricDescription memoryMetric = new StatefulServiceLoadMetricDescription();
memoryMetric.Name = "MemoryInMb";
memoryMetric.PrimaryDefaultLoad = 20;
memoryMetric.SecondaryDefaultLoad = 5;
memoryMetric.Weight = ServiceLoadMetricWeight.High;

StatefulServiceLoadMetricDescription primaryCountMetric = new StatefulServiceLoadMetricDescription();
primaryCountMetric.Name = "PrimaryCount";
primaryCountMetric.PrimaryDefaultLoad = 1;
primaryCountMetric.SecondaryDefaultLoad = 0;
primaryCountMetric.Weight = ServiceLoadMetricWeight.Medium;

StatefulServiceLoadMetricDescription replicaCountMetric = new StatefulServiceLoadMetricDescription();
replicaCountMetric.Name = "ReplicaCount";
replicaCountMetric.PrimaryDefaultLoad = 1;
replicaCountMetric.SecondaryDefaultLoad = 1;
replicaCountMetric.Weight = ServiceLoadMetricWeight.Low;

StatefulServiceLoadMetricDescription totalCountMetric = new StatefulServiceLoadMetricDescription();
totalCountMetric.Name = "Count";
totalCountMetric.PrimaryDefaultLoad = 1;
totalCountMetric.SecondaryDefaultLoad = 1;
totalCountMetric.Weight = ServiceLoadMetricWeight.Low;

serviceDescription.Metrics.Add(memoryMetric);
serviceDescription.Metrics.Add(primaryCountMetric);
serviceDescription.Metrics.Add(replicaCountMetric);
serviceDescription.Metrics.Add(totalCountMetric);

await fabricClient.ServiceManager.CreateServiceAsync(serviceDescription);
```

Powershell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,20,5”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)
```

(既定のメトリックのみを使用する場合、メトリックを収集したり、サービスを作成する場合に特別なことをしたりすることを考慮する必要性はまったくありません。)

ここまで、独自のメトリックを定義する方法を紹介しました。次に、メトリックのさまざまなプロパティについて説明します。 このようなプロパティは既に紹介しましたが、ここでは改めてその実際の意味について説明します。 現在、メトリックには 4 種類のプロパティを設定できます。

* Metric Name: メトリックの名前。 これは、リソース マネージャーから見たクラスター内のメトリックの一意の識別子です。
* Default Load: 既定の負荷は、サービスがステートレスまたはステートフルであるかによって異なる方法で表されます。
  * ステートレス サービスの場合、各メトリックには Default Load という名前のプロパティが 1 つだけあります。
  * ステートフルの場合、次のように定義します。
    * PrimaryDefaultLoad: このサービスがプライマリとして処理する、このメトリックの既定の負荷量。
    * SecondaryDefaultLoad: このサービスがセカンダリ レプリカとして処理する、このメトリックの既定の負荷量。  
* Weight: このサービスに対して構成されている他のメトリックに対する、このメトリックの相対的な重要度。

## <a name="load"></a>Load
負荷とは、一部のサービス インスタンスまたは特定のノード上のレプリカによって使用される特定のメトリックの量を表す一般的な概念です。

## <a name="default-load"></a>既定の負荷
既定の負荷とは、クラスター リソース マネージャーが想定する、このサービスの各サービス インスタンスまたはレプリカが実際のサービス インスタンスまたはレプリカから更新を受け取るまでに消費する負荷の量です。 単純なサービスの場合、最終的にこの数値は動的に更新されることがない静的な定義になるため、サービスの有効期間に使用されます。 既定の負荷は、特定のリソースを特定のワークロード専用に割り当てるという従来の手法とまったく同じ考え方であるため、単純な容量計画であれば非常に有効ですが、そのメリットは、少なくとも現時点では、マイクロサービスの考え方で運用する点にあります。マイクロサービスでは、リソースが実際には特定のワークロードに静的に割り当てられず、ユーザーが意思決定ループに参加しません。

ステートフル サービスでは、プライマリとセカンダリの両方に既定の負荷を指定できます。現実的には、多くのサービスではプライマリ レプリカとセカンダリ レプリカで実行されるワークロードが異なるため、この数値は異なることになります。また、通常はプライマリ レプリカが読み取りと書き込み (さらに計算負荷の大部分) を受け持つため、プライマリ レプリカの既定の負荷はセカンダリ  レプリカよりも大きくなります。

ただし、ここではユーザーがサービスを一定期間実行しており、サービスの一部のインスタンスまたはレプリカが他のものよりもリソースの消費量が多い、または消費量が時間によって変動することに気付いたとします。そのインスタンスまたはレプリカは、特定の顧客に関連付けられていたり、単純にメッセージング トラフィックや音声通話、株取引のような時間帯で変動するワークロードに対応したりしていることが考えられます。 いずれにしても、少なくともその時間の一部に対して大幅な負荷の低下がなければその負荷に使用できる "シングル ナンバー" が存在しないことがわかりました。 さらに、最初の推定値を軽減した場合、クラスター リソース マネージャーでサービスに割り当てられるリソース量が過剰または過小になり、その結果、ノードの使用率が過大または過小になることもわかりました。

どうすればよいでしょうか。 運用中にサービスの負荷のレポートを作成すればよいのです。

## <a name="dynamic-load"></a>動的な負荷
動的な負荷レポートを利用して、レプリカまたはインスタンスで有効期間内におけるクラスターのメトリックの割り当てとレポートされる使用量を調整できます。 通常、コールド状態で何の動作もしていないサービス レプリカまたはインスタンスは少量の特定のメトリックを使用しているとレポートし、負荷が高いレプリカやインスタンスは多くの特定のメトリックを使用しているとレポートします。 このようなクラスター内の一般的な負荷のレベルを使って、必要な量のリソースを確実に得られるように、運用中にクラスター内のサービス レプリカおよびインスタンスを再構成できます。その結果、負荷の高いサービスが、現在コールド状態または低負荷のレプリカやインスタンスからリソースを回収できるようになります。 運用中の負荷は、Reliable Services プログラミング モデルを介して、基本の StatefulService またはStatelessService クラスのプロパティである ServicePartition の ReportLoad メソッドを利用してレポートできます。 サービス内のコードは次のようになります。

コード:

```csharp
this.ServicePartition.ReportLoad(new List<LoadMetric> { new LoadMetric("Memory", 1234), new LoadMetric("metric1", 42) });
```

サービスのレプリカまたはインスタンスは、使用するように構成されているメトリックの負荷のみをレポートします。 メトリックの一覧は、各サービスの作成時に設定されますが、後で更新することもできます。 サービスのレプリカまたはインスタンスで現在使用するように構成されていないメトリックの負荷に関するレポートは、Service Fabric で記録されますが無視されます。つまり、クラスターの状態を分析したレポートを作成する際に、そのレポートが使用されません。 これにより大規模な分析が可能になるため、都合がよいのです。コードでは測定可能なすべてのメトリックを測定してレポートでき、ユーザーはコードを変更しなくても運用中にそのサービスのリソース割り当て規則を更新し、調整し、更新することができます。 たとえば、レポートのバグが多いときにメトリックを無効にしたり、動作に応じてメトリックの重み付けを再構成したり、他のメカニズムを使用してコードがデプロイされ検証された後でのみ新しいメトリックを有効にしたりできます。

## <a name="mixing-default-load-values-and-dynamic-load-reports"></a>既定の負荷値と動的負荷レポートの混在
負荷を動的にレポートするサービスに対して、既定の負荷を指定する必要性はあるでしょうか。 もちろんあります。 その場合、実際のサービスのレプリカまたはインスタンスから実際のレポートがレポートされるまで、既定の負荷は推定値として扱われます。 クラスター リソース マネージャーが処理するデータを渡すことができるため、これは非常に効果があります。既定の負荷の推定値により、最初から適切な位置にサービスのインスタンスやレプリカを配置することができるのです。 既定の負荷の情報がない場合は、作成時に事実上ランダムにサービスが配置されることになり、負荷が後で変更になれば、クラスター リソース マネージャーは、ほとんどの場合、配置をやり直すことになります。

では、前の例でカスタム負荷を追加したときと、サービスを作成後に動的に更新したときに何が起きるのか見てみましょう。 この例では、例として "Memory" を使用します。最初に次のコマンドを使ってステートフル サービスを作成していたとします。

Powershell:

```posh
New-ServiceFabricService -ApplicationName $applicationName -ServiceName $serviceName -ServiceTypeName $serviceTypeName –Stateful -MinReplicaSetSize 2 -TargetReplicaSetSize 3 -PartitionSchemeSingleton –Metric @("Memory,High,21,11”,"PrimaryCount,Medium,1,0”,"ReplicaCount,Low,1,1”,"Count,Low,1,1”)
```

この構文 (MetricName、MetricWeight、PrimaryDefaultLoad、SecondaryDefaultLoad) については前に説明しましたが、Weight の特定の値が意味する内容については、後ほど詳しく説明します。

考えられるクラスター レイアウトの 1 つは、次のようなものになります。

![Cluster Balanced with both Default and Custom metrics][Image2]

注目すべき点がいくつかあります。

* レプリカまたはインスタンスがそれぞれの独自の負荷をレポートするまで、レプリカまたはインスタンスではサービスの既定の負荷が使用されるため、ステートフル サービスのパーティション 1 の内部のレプリカがその独自の負荷をレポートしていないことがわかります。
* パーティション内のセカンダリ レプリカは独自の負荷を持つことができます。
* 全体的にメトリックは良好で、ノードの最大負荷と最小負荷 (ここで重視するカスタム メトリックである Memory) の差がわずか係数 1.75 です (Memory の負荷が最大のノードが N3、最小のノードが N2、その比が 28/16 = 1.75)。これは非常にバランスが取れている状態です。

さらに説明が必要な点がいくつかあります。

* 1.75 という比率が妥当であるかどうかを、何を根拠に決定するのか。 その比率が適切であるのか、さらに対処が必要なのかを、どのようにして判断するのか。
* 負荷分散はいつ実行されるのか。
* Memory の重みが "High" であるとは、どのような意味があるのか。

## <a name="metric-weights"></a>Metric weights
メトリックの重みを利用して、2 つの異なるサービスから同じメトリックをレポートさせながら、そのメトリックを別々に負荷分散する重要性を見ることができます。 たとえば、メモリ内分析エンジンと永続的なデータベースの場合を考えてみましょう。おそらく、どちらも "Memory" メトリックが重視されるでしょうが、メモリ内サービスであれば "Disk" メトリックはそれほど重視されないはずです。少量の "Disk" メトリックを消費するとしても、サービスのパフォーマンスに重要な影響を与えるわけではないため、それをレポートすることさえもしない場合があります。 複数のサービスを対象に同じメトリックを追跡できる機能は非常に有効です。クラスター リソース マネージャーでクラスター内の実際の消費量を追跡できるからです。これにより、ノードの容量オーバーなどの問題を確実に防止できます。

また、メトリックの重みは、クラスター リソース マネージャーでクラスターの負荷を分散する方法を決定するときに、完璧な答えが存在しない場合 (ほとんどの場合はそうですが) の判断材料となります。 メトリックには、Zero、Low、Medium、High という 4 種類の重みレベルを設定できます。 重み Zero のメトリックは負荷分散を判断するときにはまったく考慮されませんが、容量測定などのときにはその負荷が考慮されます。

クラスター内のさまざまなメトリックの重みの実際の影響は、特定のメトリックが他のメトリックよりも重要であることが集計時にクラスター リソース マネージャーに示されることで、サービスの配置が変わってくるという点に表れます。 これを認識するため、さまざまな重みを持つメトリックが他と競合する場合、クラスター リソース マネージャーはより高く重み付けされたメトリックをさらに負荷分散するソリューションを優先することができます。

それでは、いくつかの負荷レポートの簡単な例を使って、メトリックの重みが変わるとクラスター内の割り当てがどのように変化するか見てみましょう。 この例では、メトリックの相対的な重みを切り替えることで、リソース マネージャーでサービスの配置が変わり、特定のソリューションが優先されることがわかります。

![Metric Weight Example and Its Impact on Balancing Solutions][Image3]

この例には 4 つの異なるサービスがあります。すべてのサービスが、A と B という異なる 2 つのメトリックに対して別々の値をレポートします。1 番目のケースでは、すべてのサービスで MetricA が最も重要 (重み = High)、MetricB が比較的重要でない (重み = Low) と定義されており、実際にクラスター リソース マネージャーでは MetricA の方が MetricB よりもバランスが取れる (標準的な偏りが少ない) ようにサービスが配置されています。 2 番目のケースでは、メトリックの重みが逆になっています。クラスター リソース マネージャーは MetricA よりも MetricB のバランスを重視した配置にするために、サービス A とサービス B を入れ替える可能性が高いことがわかります。

### <a name="global-metric-weights"></a>メトリックの重み
では、ServiceA で MetricA を最も重要なメトリックとして定義する一方、ServiceB ではまったく考慮しなかった場合、実際に使用される重みはどうなるでしょうか。

すべてのメトリックについて追跡される重みは、実際には 2 種類あります。そのサービスで独自に定義される重みと、そのメトリックが考慮されるすべてのサービス全体のグローバルな平均重みです。 生成するソリューションのスコアを計算する際には、サービス独自の優先度に関してサービスのバランスを取ることと、クラスター全体で適切に割り当てることを両立することが重要であるため、この 2 種類の重みが両方とも使われます。

グローバルとローカルのどちらかのバランスを考慮しなかった場合、どうなるでしょうか。 グローバルでバランスの取れたソリューションを構築することは簡単ですが、その場合は個々のサービスのバランスが非常に悪くなり、偏ったリソース割り当てが生じることになります。 次の例では、ステートフル サービスに構成する既定のメトリックである PrimaryCount、ReplicaCount、Count について検討します。グローバルのバランスのみを考慮した場合、どうなるのか見てみましょう。

![The Impact of a Global Only Solution][Image4]

上の方の例では、グローバルのバランスのみを重視しています。実際にクラスター全体のバランスが取れており、すべてのノードに同じ数のプライマリが含まれ、レプリカの合計数も同じです。 ただし、この割り当ての実際の影響を見てみると、それほど良いとは言えません。ノードが 1 つでも失われると、その中のプライマリもすべて失われるため、特定のワークロードに偏りが生じます。 たとえば、1 番目のノードが失われるとします。 その場合、円のサービスの 3 つの異なるパーティションに対応する 3 つのプライマリが同時に失われます。 逆に、その他の 2 つのサービス (三角形と六角形) はそれぞれパーティションを持っており、レプリカが失われてもサービスの中断は生じません (ダウンしたレプリカを回復する必要がある場合を除く)。

下の方の例では、グローバルのバランスとサービス単位のバランスの両方を考慮してレプリカを分散しています。 ソリューションのスコアを計算するときに、重みの大部分をグローバル ソリューションに割り当てますが、可能な限り個々のサービス内部でバランスを取ることができるように (構成可能な) 部分を割り当てます。 その結果、同じ 1 番目のノードが失われても、失われるプライマリ (およびセカンダリ) の数はすべてのサービスのすべてのパーティションに分散され、各サービスへの影響が同じになります。

メトリックの重みを考慮することで、各サービス用に構成されたメトリックの平均重みに基づいてグローバルなバランスが算出されます。 サービス単位で定義されたメトリックの重みに応じて、サービスのバランスを取るのです。

## <a name="next-steps"></a>次のステップ
* サービスの構成に利用できるその他のオプションの詳細については、 [サービスの構成の詳細に関する記事](service-fabric-cluster-resource-manager-configure-services.md)
* 最適化メトリックの定義は、負荷を分散するのではなく、ノードで統合する方法の 1 つです。 最適化を構成する方法については、 [この記事](service-fabric-cluster-resource-manager-defragmentation-metrics.md)
* クラスター リソース マネージャーでクラスターの負荷を管理し、分散するしくみについては、 [負荷分散](service-fabric-cluster-resource-manager-balancing.md)
* 最初から開始して、 [Service Fabric クラスター リソース マネージャーの概要を確認するにはこちらを参照してください](service-fabric-cluster-resource-manager-introduction.md)
* 移動コストは、特定のサービスが他のサービスよりも高額になっていることをクラスター リソース マネージャーに警告する信号の 1 つです。 移動コストの詳細については、 [この記事](service-fabric-cluster-resource-manager-movement-cost.md)

[Image1]:./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-cluster-layout-with-default-metrics.png
[Image2]:./media/service-fabric-cluster-resource-manager-metrics/Service-Fabric-Resource-Manager-Dynamic-Load-Reports.png
[Image3]:./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-metric-weights-impact.png
[Image4]:./media/service-fabric-cluster-resource-manager-metrics/cluster-resource-manager-global-vs-local-balancing.png



<!--HONumber=Dec16_HO2-->


