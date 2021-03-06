---
title: "Service Fabric とコンテナーのデプロイ | Microsoft Docs"
description: "Service Fabric と、コンテナーを使用してマイクロサービス アプリケーションをデプロイする方法。 この記事では、Service Fabric が提供するコンテナー用の機能と、クラスターに Windows コンテナー イメージをデプロイする方法について説明します。"
services: service-fabric
documentationcenter: .net
author: msfussell
manager: timlt
editor: 
ms.assetid: 799cc9ad-32fd-486e-a6b6-efff6b13622d
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/24/2016
ms.author: mfussell
translationtype: Human Translation
ms.sourcegitcommit: af9f761179896a1acdde8e8b20476b7db33ca772
ms.openlocfilehash: 1c5f3bc66c902c3b7186cad44728fa5237dd298a


---
# <a name="preview-deploy-a-windows-container-to-service-fabric"></a>プレビュー: Service Fabric への Windows コンテナーのデプロイ
> [!div class="op_single_selector"]
> * [Windows コンテナーのデプロイ](service-fabric-deploy-container.md)
> * [Docker コンテナーのデプロイ](service-fabric-deploy-container-linux.md)
> 
> 

この記事では、コンテナー化されたサービスを Windows コンテナーに構築する手順を紹介します。

> [!NOTE]
> この機能は Linux 向けのプレビューの段階にあり、現時点で Windows Server 2016 ではまだ使用できません。 この機能は、Azure Service Fabric の今後のリリースで、Windows Server 2016 向けのプレビュー版になります。 
> 
> 

Service Fabric には、コンテナー化されたマイクロサービスで構成されたアプリケーションの構築に役立つ、いくつかのコンテナー機能が用意されています。 

機能は次のとおりです。

* コンテナー イメージのデプロイとアクティブ化
* リソース ガバナンス
* リポジトリの認証
* コンテナー ポートからホスト ポートへのマッピング
* コンテナー間での検出と通信
* 環境変数の構成と設定の機能

ここで、コンテナー化されたサービスをパッケージ化してアプリケーションに組み込む際の各機能の動作を見ていきます。

## <a name="package-a-windows-container"></a>Windows コンテナーのパッケージ化
コンテナーをパッケージ化する際は、Visual Studio プロジェクト テンプレートを使用するか、それとも [アプリケーション パッケージを手動で作成](#manually)するかを選択できます。 Visual Studio を使用するときには、アプリケーション パッケージの構造とマニフェスト ファイルは新しいプロジェクト テンプレートによって作成されます。 VS テンプレートは、将来のリリースでリリースされます。

## <a name="use-visual-studio-to-package-an-existing-container-image"></a>Visual Studio を使用した既存コンテナ― イメージのパッケージ化
> [!NOTE]
> Visual Studio for Service Fabric の今後のリリースでは、現在ゲスト実行可能ファイルを追加するのと同じ方法で、アプリケーションにコンテナーを追加できます。 詳細については、「[Service Fabric へのゲスト実行可能ファイルのデプロイ](service-fabric-deploy-existing-app.md)」を参照してください。 現在は、次のセクションで説明するように、コンテナーを手動でパッケージ化する必要があります。
> 
> 

<a id="manually"></a>

## <a name="manually-package-and-deploy-a-container"></a>コンテナーの手動によるパッケージ化およびデプロイ
コンテナー化されたサービスを手動でパッケージ化するプロセスは、次の手順に基づいています。

1. リポジトリにコンテナーを発行します。
2. パッケージ ディレクトリ構造を作成します。
3. サービス マニフェスト ファイルを編集します。
4. アプリケーション マニフェスト ファイルを編集します。

## <a name="deploy-and-activate-a-container-image"></a>コンテナー イメージのデプロイおよびアクティブ化
Service Fabric の [アプリケーション モデル](service-fabric-application-model.md)では、コンテナーは複数のサービス レプリカが配置されたアプリケーション ホストを表します。 コンテナーをデプロイしてアクティブ化するには、コンテナー イメージ名をサービス マニフェストの `ContainerHost` 要素に入力します。

サービス マニフェストで、エントリ ポイントの `ContainerHost` を追加します。 次に、コンテナー リポジトリとイメージの名前になるように `ImageName` を設定します。 次のマニフェストの一部では、`myrepo` というリポジトリから `myimage:v1` というコンテナーをデプロイする場合の例を示しています。

```xml
    <CodePackage Name="Code" Version="1.0">
        <EntryPoint>
          <ContainerHost>
            <ImageName>myrepo/myimage:v1</ImageName>
            <Commands></Commands>
          </ContainerHost>
        </EntryPoint>
    </CodePackage>
```

コンテナー内で実行されるコンマ区切りの一連のコマンドによりオプションの `Commands` 要素を指定することで、入力コマンドを指定することができます。

## <a name="understand-resource-governance"></a>リソース ガバナンスについて
リソース ガバナンスは、コンテナー機能の 1 つです。コンテナーがホスト上で使用できるリソースを制限します。 アプリケーション マニフェストで指定された `ResourceGovernancePolicy` は、サービス コード パッケージのリソース制限を宣言するために使用します。 次のリソースのリソースの制限を設定できます。

* メモリ
* MemorySwap
* CpuShares (CPU の相対的な重み)
* MemoryReservationInMB  
* BlkioWeight (BlockIO の相対的な重み)

> [!NOTE]
> 今後のリリースで、IOPS、読み取り/書き込み BPS などの特定のブロック IO 制限の指定がサポートされる予定です。
> 
> 

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ResourceGovernancePolicy CodePackageRef="FrontendService.Code" CpuShares="500"
            MemoryInMB="1024" MemorySwapInMB="4084" MemoryReservationInMB="1024" />
        </Policies>
    </ServiceManifestImport>
```

## <a name="authenticate-a-repository"></a>リポジトリの認証
コンテナーをダウンロードするには、コンテナー リポジトリにサインイン資格情報を提供する必要がある場合があります。 アプリケーション マニフェストで指定されたサインイン資格情報は、イメージ リポジトリからコンテナー イメージをダウンロードする際に必要なサインイン情報または SSH キーを指定するのに使用されます。 次の例は、 *TestUser* というアカウントと、クリア テキスト (推奨*されません*) のパスワードを示しています。

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
                <RepositoryCredentials AccountName="TestUser" Password="12345" PasswordEncrypted="false"/>
            </ContainerHostPolicies>
        </Policies>
    </ServiceManifestImport>
```

コンピューターにデプロイされた証明書を使用してパスワードを暗号化することをお勧めします。

次の例は、*MyCert* という証明書を使用してパスワードが暗号化された *TestUser* というアカウントを示しています。 `Invoke-ServiceFabricEncryptText` PowerShell コマンドを使用して、パスワードのシークレット暗号化テキストを作成できます。 詳細については、「[Service Fabric アプリケーションでのシークレットの管理](service-fabric-application-secret-management.md)」を参照してください。

パスワードの暗号化を解除するための証明書の秘密キーは、ローカル コンピューターにアウトオブバンド方式でデプロイされる必要があります  (Azure では、Azure Resource Manager を使用します)。その後、Service Fabric によってコンピューターにサービス パッケージが展開されたときに、シークレットを暗号化解除できます。 アカウント名とシークレットを使用すると、コンテナーのリポジトリで認証できます。

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
                <RepositoryCredentials AccountName="TestUser" Password="[Put encrypted password here using MyCert certificate ]" PasswordEncrypted="true"/>
            </ContainerHostPolicies>
        </Policies>
    </ServiceManifestImport>
```

## <a name="configure-container-port-to-host-port-mapping"></a>コンテナー ポートからホスト ポートへのマッピングの構成
アプリケーション マニフェストで `PortBinding` を指定することで、コンテナーとの通信に使用されるホスト ポートを構成できます。 このポートのバインドでは、サービスがコンテナー内部でリッスンしているポートをホスト上のポートにマップします。

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
                <PortBinding ContainerPort="8905"/>
            </ContainerHostPolicies>
        </Policies>
    </ServiceManifestImport>
```

## <a name="configure-container-to-container-discovery-and-communication"></a>コンテナー間での検出と通信の構成
次の例に示すように、`PortBinding` ポリシーを使用して、コンテナー ポートをサービス マニフェストの `Endpoint` にマップすることができます。 エンドポイント `Endpoint1` は固定ポート (たとえば、ポート 80) を指定できます。 ポートをまったく指定せずにいることもでき、その場合はクラスターのアプリケーションのポート範囲からランダムにポートが選択されます。

エンドポイントを指定する場合、Service Fabric は、サービス マニフェストの `Endpoint` タグを使用して、このエンドポイントをネーム サービスに自動的に発行することができます。 これにより、クラスター内で実行されているその他のサービスが解決のための REST クエリを使用してこのコンテナーを検出できます。

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <Policies>
            <ContainerHostPolicies CodePackageRef="FrontendService.Code">
                <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
            </ContainerHostPolicies>
        </Policies>
    </ServiceManifestImport>
```

ネーム サービスに登録すると、[リバース プロキシ](service-fabric-reverseproxy.md)を使用して、コンテナー内のコードでコンテナー間の通信を簡単に行うことができます。 通信は、リバース プロキシ HTTP リスニング ポートおよび通信先のサービスの名前を環境変数として設定することで実行します。 詳細については、次のセクションを参照してください。 

## <a name="configure-and-set-environment-variables"></a>環境変数の構成と設定
環境変数は、サービス マニフェストのコード パッケージごとに指定できます。これは、コンテナーにデプロイされたサービスでも、プロセス実行可能ファイルまたはゲスト実行可能ファイルとしてデプロイされたサービスでも実施可能です。 これらの環境変数の値は、アプリケーション マニフェスト内で個別に上書きできるほか、デプロイの最中にアプリケーションのパラメーターとして指定することもできます。

次のサービス マニフェストの XML スニペットは、コード パッケージ用の環境変数を指定する方法の例です。

```xml
    <ServiceManifest Name="FrontendServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <Description>a guest executable service in a container</Description>
        <ServiceTypes>
            <StatelessServiceType ServiceTypeName="StatelessFrontendService"  UseImplicitHost="true"/>
        </ServiceTypes>
        <CodePackage Name="FrontendService.Code" Version="1.0">
            <EntryPoint>
            <ContainerHost>
                <ImageName>myrepo/myimage:v1</ImageName>
                <Commands></Commands>
            </ContainerHost>
            </EntryPoint>
            <EnvironmentVariables>
                <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
                <EnvironmentVariable Name="BackendServiceName" Value=""/>
            </EnvironmentVariables>
        </CodePackage>
    </ServiceManifest>
```

これらの環境変数は、アプリケーション マニフェスト レベルで上書きできます。

```xml
    <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
        <EnvironmentOverrides CodePackageRef="FrontendService.Code">
            <EnvironmentVariable Name="BackendServiceName" Value="[BackendSvc]"/>
            <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
        </EnvironmentOverrides>
    </ServiceManifestImport>
```

前の例では、`HttpGateway` 環境変数に対して明示的な値 (19000) を指定しています。一方、`BackendServiceName` パラメーターの値は `[BackendSvc]` アプリケーション パラメーターによって設定します。 これらの設定により、アプリケーションのデプロイ時に `BackendServiceName` の値を指定できるようになり、マニフェスト内の固定値もなくすことができます。

## <a name="complete-examples-for-application-and-service-manifest"></a>アプリケーション マニフェストとサービス マニフェストのすべてのコード例

アプリケーション マニフェスト ファイルの例を次に示します。

```xml
    <ApplicationManifest ApplicationTypeName="SimpleContainerApp" ApplicationTypeVersion="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <Description>A simple service container application</Description>
        <Parameters>
            <Parameter Name="ServiceInstanceCount" DefaultValue="3"></Parameter>
            <Parameter Name="BackEndSvcName" DefaultValue="bkend"></Parameter>
        </Parameters>
        <ServiceManifestImport>
            <ServiceManifestRef ServiceManifestName="FrontendServicePackage" ServiceManifestVersion="1.0"/>
            <EnvironmentOverrides CodePackageRef="FrontendService.Code">
                <EnvironmentVariable Name="BackendServiceName" Value="[BackendSvcName]"/>
                <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
            </EnvironmentOverrides>
            <Policies>
                <ResourceGovernancePolicy CodePackageRef="Code" CpuShares="500" MemoryInMB="1024" MemorySwapInMB="4084" MemoryReservationInMB="1024" />
                <ContainerHostPolicies CodePackageRef="FrontendService.Code">
                    <RepositoryCredentials AccountName="username" Password="****" PasswordEncrypted="true"/>
                    <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
                </ContainerHostPolicies>
            </Policies>
        </ServiceManifestImport>
    </ApplicationManifest>
```

次にサービス マニフェストの例 (上記のアプリケーション マニフェストで指定されたもの) を示します。

```xml
    <ServiceManifest Name="FrontendServicePackage" Version="1.0" xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
        <Description> A service that implements a stateless front end in a container</Description>
        <ServiceTypes>
            <StatelessServiceType ServiceTypeName="StatelessFrontendService"  UseImplicitHost="true"/>
        </ServiceTypes>
        <CodePackage Name="FrontendService.Code" Version="1.0">
            <EntryPoint>
            <ContainerHost>
                <ImageName>myrepo/myimage:v1</ImageName>
                <Commands></Commands>
            </ContainerHost>
            </EntryPoint>
            <EnvironmentVariables>
                <EnvironmentVariable Name="HttpGatewayPort" Value=""/>
                <EnvironmentVariable Name="BackendServiceName" Value=""/>
            </EnvironmentVariables>
        </CodePackage>
        <ConfigPackage Name="FrontendService.Config" Version="1.0" />
        <DataPackage Name="FrontendService.Data" Version="1.0" />
        <Resources>
            <Endpoints>
                <Endpoint Name="Endpoint1" Port="80"  UriScheme="http" />
            </Endpoints>
        </Resources>
    </ServiceManifest>
```

## <a name="next-steps"></a>次のステップ
コンテナー化サービスをデプロイしたので、ライフ サイクルを管理する方法について、「[Service Fabric アプリケーションのライフ サイクル](service-fabric-application-lifecycle.md)」を参照してください。




<!--HONumber=Nov16_HO3-->


