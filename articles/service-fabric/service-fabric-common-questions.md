---
title: "Microsoft Azure Service Fabric に関するよく寄せられる質問 |Microsoft ドキュメント"
description: "Service Fabric に関してよく寄せられる質問とその回答"
services: service-fabric
documentationcenter: .net
author: seanmck
manager: timlt
editor: 
ms.assetid: 5a179703-ff0c-4b8e-98cd-377253295d12
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/14/2016
ms.author: seanmck
translationtype: Human Translation
ms.sourcegitcommit: da356a95fc372c140e089a943e5fcb680f0c9fd7
ms.openlocfilehash: 009fde75bff1b7746ad0ae247a3b895366f54b84

---


# <a name="commonly-asked-service-fabric-questions"></a>Service Fabric に関してよく寄せられる質問

Service Fabric で実行できる内容とその使用方法に関してよく寄せられる多数の質問があります。 このドキュメントでは、これらのよく寄せられる質問とその回答を示します。

## <a name="cluster-setup-and-management"></a>クラスターのセットアップと管理

### <a name="can-i-create-a-cluster-that-spans-multiple-azure-regions"></a>複数の Azure リージョンにまたがるクラスターを作成することはできますか?

現時点ではできませんが、これは共通の要求であるため、引き続き調査が行われています。

Service Fabric クラスタリング テクノロジの核心は、Azure リージョンとは無関係に、相互にネットワーク接続されているのであれば、世界中で実行されているコンピューターを組み合わせて使用できることです。 ただし、Azure での Service Fabric クラスター リソースは、クラスターが構築される仮想マシン スケール セットと同じように、地域に限定されています。 さらに、遠く離れて分散しているコンピューター間に強い一貫性のあるデータのレプリケーションを提供することに伴う固有の課題があります。 マイクロソフトでは、予測可能で妥当なパフォーマンスが確保された後で、地域間クラスターをサポートすることを目指しています。

### <a name="do-service-fabric-nodes-automatically-receive-os-updates"></a>Service Fabric ノードでは、OS の更新は自動的に受信されますか?

現時点ではそうではありませんが、これも共通の要求であり、今後提供する予定です。

OS の更新に伴う課題は、それを行うには通常はコンピューターを再起動する必要があり、それによって可用性が一時的に失われることです。 Service Fabric では可用性が失われたサービスのトラフィックを他のノードに自動的にリダイレクトするため、再起動自体は問題ではありません。 ただし、OS の更新がクラスター全体で連係されていなかった場合、多数のノードが一度にダウンする可能性があります。 このような再起動の同時発生によって、サービスの可用性が完全に失われるか、少なくとも (ステートフル サービス用の) 特定のパーティションの可用性が失われる可能性があります。

今後、更新ドメイン間で連係する OS 更新ポリシーをサポートして、再起動やその他の予想外の障害が発生した場合でも可用性が維持されることを保証する予定です。

それまでの間、唯一の安全なオプションは、OS の更新を一度に 1 つのノードで手動で更新することです。

### <a name="what-is-the-minimum-size-of-a-service-fabric-cluster-why-cant-it-be-smaller"></a>Service Fabric クラスターの最小サイズとは何ですか?  もっと小さくできないのはなぜですか?

運用ワークロードを実行する Service Fabric クラスターでサポートされる最小サイズは、5 つのノードです。 開発/テスト シナリオでは、3 つのノードで構成されるクラスターがサポートされます。

これらの最小要件が存在する理由はは、Service Fabric クラスターでは、一連のステートフル システム サービス (ネーム サービスやフェールオーバー マネージャーなど) が実行されるためです。 クラスターにデプロイされているサービスと現在のホスト場所を追跡するこれらのサービスは、強い一貫性に依存しています。 この強い一貫性は、これらのサービスの状態に対する特定の更新の "*クォーラム*" を取得する能力に依存しています (クォーラムは特定のサービスに対するレプリカの strict majority (N/2 +1) を表します)。

これを背景として、考えられるいくつかのクラスター構成を検討してみます。

**1 つのノード**: 単一のノードが何らかの理由で失われることはクラスター全体が失われるためを意味するため、このオプションでは高可用性を提供できません。

**2 つのノード**: 2 つのノード (N = 2) 間でデプロイされるサービスのクォーラムは 2 です (2/2 + 1 = 2)。 1 つのレプリカが失われると、クォーラムを作成できなくなります。 サービスのアップグレードを実行するには、レプリカを一時的に停止させる必要があるため、これは有用な構成ではありません。

**3 つのノード**: 3 つのノード (N = 3) でも、クォーラムを作成するためのノードの要件は 2 つのままです (3/2 + 1 = 2)。 これは、個々のノードが失われた場合でも、クォーラムを保持できることを意味します。

3 つのノードのクラスター構成は、開発/テストでサポートされます。その理由は、アップグレードや個々のノードの障害が同時に発生しない限り、アップグレードを安全に実行でき、個々のノードの障害にも対応できるためです。 運用ワークロードでは、このような同時障害から回復する必要があるため、5 つのノードが必要です。

### <a name="can-i-turn-off-my-cluster-at-nightweekends-to-save-costs"></a>コストを節約するために夜間/週末にクラスターをオフにすることはできますか?

通常はできません。 Service Fabric は、一時的なローカル ディスクに状態を格納します。これは、仮想マシンが別のホストに移動されても、データは一緒に移動されないことを意味します。 通常の運用では、新しいノードは他のノードによって最新の状態になるため、これは問題にはなりません。 ただし、すべてのノードを停止し、後で再起動した場合は、ほとんどのノードが新しいホスト上で開始され、システムが回復できない状態になる可能性が非常に高くなります。

アプリケーションをデプロイする前にテスト用のクラスターを作成する場合は、[継続的インテグレーション/継続的配置パイプライン](service-fabric-set-up-continuous-integration.md)の一部としてこれらのクラスターを作成することをお勧めします。

## <a name="application-design"></a>アプリケーションの設計

### <a name="whats-the-best-way-to-query-data-across-partitions-of-a-reliable-collection"></a>Reliable Collection のパーティション全体のデータを照会する最善の方法は何ですか?

Reliable collection は、通常は、パフォーマンスとスループットを高めるためのスケールアウトを実行できるように[パーティション分割](service-fabric-concepts-partitioning.md)されます。 これは、特定のサービスの状態が、数十から数百台のコンピューターに分散されることを意味します。 このデータ セット全体に対して操作を実行するには、いくつかのオプションがあります。

- 別のサービスのすべてのパーティションを照会して必要なデータを引き出すサービスを作成します。
- 別のサービスのすべてのパーティションからデータを受信できるサービスを作成します。
- 各サービスから外部ストアにデータを定期的にプッシュします。 この方法は、実行する照会が中核となるビジネス ロジックの一部ではない場合のみに適しています。


### <a name="whats-the-best-way-to-query-data-across-my-actors"></a>アクター全体のデータを照会する最善の方法は何ですか。

アクターは、状態とコンピューティングに依存しないように設計されているため、実行時にアクターの状態を広範に照会することは推奨されていません。 アクターの状態のフル セットを照会する必要がある場合は、次のいずれかを検討してください。

- アクター サービスをステートフルな信頼できるサービスに置き換えて、すべてのアクターからすべてのデータを収集するネットワーク要求の数がサービス内のパーティションの数と同じになるようにします。
- 状態を外部ストアに定期的にプッシュして簡単に照会できるようにアクターを設計します。 上記と同じように、この方法は、実行する照会が実行時の動作で必須でない場合のみに実行可能です。

### <a name="how-much-data-can-i-store-in-a-reliable-collection"></a>Reliable Collection には、どのくらいの量のデータを格納できますか?

Reliable Services は通常はパーティション分割されるため、格納できる量は、クラスター内のコンピューターの台数とこれらのコンピューターで使用できるメモリの量によってのみ制限されます。

たとえば、サービスに 100 個のパーティションと 3 つのレプリカがある Reliable Collection があり、平均サイズが 1 KB のオブジェクトを格納するとします。 ここで、クラスターが 10 台のコンピューターで構成され、各コンピューターのメモリが 16 GB であるとします。 単純かつ控えめに見積もるために、オペレーティング システム、システム サービス、Service Fabric ランタイム、および使用するサービスで 6 GB が消費され、各コンピューターで残りの 10 GB 、つまりクラスターで 100 GB を使用できるものと想定します。

各オブジェクトは 3 回格納される必要があること (1 回はプライマリに、2 回はレプリカに) に留意すると、容量全部を使用して運用した場合は、約 3,500 万個のオブジェクトをコレクションに格納するのに十分なメモリがあります。 ただし、障害ドメインとアップグレード ドメインが同時に失われた場合の回復力を備えておくことが推奨されます。これは容量の約 1/3 に当たるため、この数値は約 2,300 万個に減少します。

この計算では、以下も想定されていることに注意してください。

- パーティション間のデータの分散はほぼ一定であるか、クラスター リソース マネージャーに負荷メトリックを報告すること。 既定では、Service Fabric は、レプリカの数に基づいて負荷を分散します。 上の例では、クラスター内の各ノードに 10 個のプライマリ レプリカと 20 個のセカンダリ レプリが配置されます。 パーティション間で負荷が均等に分散される場合は問題はありません。 負荷が均等でない場合は、負荷をレポートして、小さなレプリカがリソース マネージャーによって 1 つにまとめられ、大きなレプリカが個々のノードでより多くのメモリを消費できるようにする必要があります。

- 問題の Reliable Service が、クラスターで格納状態にある唯一のサービスであること。 複数のサービスをクラスターにデプロイできるため、実行する必要があるリソースとその状態の管理を意識する必要があります。

- クラスター自体が拡大も縮小もしないこと。 コンピューターが追加された場合、Service Fabric は、追加された容量を活用するためにレプリカの再分散を実行します。個々のレプリカは複数のコンピューターにまたがることはできないため、この動作はコンピューターの数がサービス内のパーティションの数を上回るまで続けられます。 逆に、コンピューターを削除することでクラスターのサイズが減少した場合、レプリカはより緊密にパックされ、全体の容量が小さくなります。

### <a name="how-much-data-can-i-store-in-an-actor"></a>アクターには、どのくらいの量のデータを格納できますか?

Reliable Services と同じように、アクター サービスに格納できるデータの量は、ディスク領域の合計とクラスター内のノードで使用できるメモリによってのみ制限されます。 ただし、個々のアクターは、小さな分量の状態とそれに関連付けられたビジネス ロジックをカプセル化するために使用すると、最も効果があります。 原則として、個々のアクターには、キロバイト単位で測定される状態を格納してください。

### <a name="how-does-service-fabric-relate-to-containers"></a>Service Fabric はコンテナーとどのように関連していますか?

コンテナーは、サービスとそれに依存するものをパケージ化して、それらがすべての環境で一貫性をもって実行され、1 台のコンピューター上で隔離された方法で運用できるようにします。 Service Fabric には、[コンテナーにパッケージ化されたサービス](service-fabric-containers-overview.md)も含めて、サービスをデプロイして管理する方法が用意されています。

## <a name="next-steps"></a>次のステップ

- [Service Fabric の中心概念とベスト プラクティスを学習する](https://mva.microsoft.com/en-us/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=tbuZM46yC_5206218965)



<!--HONumber=Dec16_HO3-->


