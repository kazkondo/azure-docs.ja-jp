---
title: "Azure AD Connect: ユーザー サインイン | Microsoft Docs"
description: "Azure AD Connect ユーザー サインインのカスタム設定"
services: active-directory
documentationcenter: 
author: billmath
manager: femila
editor: curtand
ms.assetid: 547b118e-7282-4c7f-be87-c035561001df
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/01/2016
ms.author: billmath
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 9c31cd4c26a9285d9fca42d24bb87c7633df005f


---
# <a name="azure-ad-connect-user-sign-on-options"></a>Azure AD Connect ユーザーのサインオン オプション
Azure AD Connect では、ユーザーは同じパスワードを使用して、クラウドとオンプレミス両方のリソースにサインオンできます。 この記事では、Azure AD へのサインインに使用する ID を選択できるようにするために、各 ID モデルの主要な概念について説明します。

既に Azure AD の ID モデルについての知識があり、特定の方法の詳細を知りたい場合は、以下の該当するトピックをクリックしてください。

* [パスワードの同期](#password-synchronization)
* [フェデレーション SSO (ADFS を使用)](#federation-using-a-new-or-existing-ad-fs-in-windows-server-2012-r2-farm)

## <a name="choosing-a-user-sign-in-method"></a>ユーザーのサインイン方法の選択
多くの組織においては Office 365、SaaS アプリケーション、およびその他の Azure AD ベースのリソースへのユーザー サインオンを可能にするのみなので、その場合は既定のパスワードの同期オプションをお勧めします。
ただし、組織によっては AD FS などフェデレーション サインオン オプションを使用する必要があります。  次のような理由が考えられます。

* 組織は既に AD FS またはサード パーティのフェデレーション プロバイダーをデプロイしている
* セキュリティ ポリシーで、クラウドへのパスワード ハッシュ同期を禁止している
* 企業ネットワーク上のドメイン参加マシンからクラウドのリソースにアクセスするときに、ユーザーにとってシームレスな SSO (パスワードの再入力を要求しない) を実現する必要がある
* 次のような AD FS の特定の機能が必要である
  * サード パーティ プロバイダーまたはスマート カードを使用するオンプレミスのMulti-Factor Authentication (Windows Server 2012 R2 の AD FS 用のサード パーティの MFA プロバイダーについての説明を参照)
  * ソフト アカウント ロックアウトや AD パスワードと作業時間のポリシーなどの Active Directory の統合機能
  * デバイス登録、Azure AD 参加、または Intune MDM ポリシーを使用するオンプレミスとクラウド両方のリソースへの条件付きアクセス

### <a name="password-synchronization"></a>パスワードの同期
パスワードの同期では、ユーザーのパスワードのハッシュが、オンプレミスの Active Directory から Azure AD に同期されます。  オンプレミスでパスワードが変更またはリセットされると、新しいパスワードは Azure AD にすぐに同期され、ユーザーは常にクラウドのリソースでもオンプレミスと同じパスワードを使用できます。  パスワードがクリア テキストの状態で Azure AD に送信されたり Azure AD に格納されたりすることはありません。
パスワードの同期とパスワードの書き戻しを組み合わせて使用すると、Azure AD でセルフ サービス パスワードのリセットを有効にすることができます。

<center>![クラウド](./media/active-directory-aadconnect-user-signin/passwordhash.png)</center>

[パスワードの同期の詳細について](active-directory-aadconnectsync-implement-password-synchronization.md)

### <a name="federation-using-a-new-or-existing-ad-fs-in-windows-server-2012-r2-farm"></a>Windows Server 2012 R2 ファームで新規または既存の AD FS を使用するフェデレーション
フェデレーション サインオンを設定すると、ユーザーは自分のオンプレミスのパスワードを使用して Azure AD ベースのサービスにサインオンすることができます。また、企業ネットワークにアクセスしているときはパスワードを再入力する必要はありません。  AD FS を使用するフェデレーション オプションでは、新しい AD FS をデプロイすることも、Windows Server 2012 R2 ファームの既存の AD FS を指定することもできます。  既存のファームを指定する場合は、ユーザーがサインオンできるように、Azure AD Connect によってファームと Azure AD の間の信頼が構成されます。

<center>![クラウド](./media/active-directory-aadconnect-user-signin/federatedsignin.png)</center>

#### <a name="to-deploy-federation-with-ad-fs-in-windows-server-2012-r2-you-will-need-the-following"></a>Windows Server 2012 R2 で AD FS を使用するフェデレーションをデプロイするには、以下が必要です。
新しいファームをデプロイする場合:

* フェデレーション サーバー用の Windows Server 2012 R2 サーバー。
* Web アプリケーション プロキシ用の Windows Server 2012 R2 サーバー。
* fs.contoso.com などの目的のフェデレーション サービス名に対する SSL 証明書を 1 つ含む .pfx ファイル。

新しいファームをデプロイするか、既存のファームを使用する場合:

* フェデレーション サーバー上のローカル管理者の資格情報。
* Web アプリケーション プロキシ ロールをデプロイするワークグループ (ドメインに参加していない) サーバー上のローカル管理者の資格情報。
* ウィザードを実行するコンピューターは、Windows リモート管理を使用して AD FS または Web アプリケーション プロキシをインストールする他のコンピューターに接続できる必要があります。

[AD FS とのフェデレーションの構成](connect/active-directory-aadconnect-get-started-custom.md#configuring-federation-with-ad-fs)

#### <a name="sign-on-using-an-earlier-version-of-ad-fs-or-a-third-party-solution"></a>以前のバージョンの AD FS またはサード パーティのソリューションを使用するサインオン
クラウド サインオンを以前のバージョンの AD FS (AD FS 2.0 など) またはサード パーティのフェデレーション プロバイダーを使用して既に構成している場合は、Azure AD Connect でのユーザーのサインインの構成をスキップすることができます。  これにより、既存のソリューションをサインオンに使用しながら、最新の同期と Azure AD Connect のその他の機能を使用することができます。

[Azure AD のサードパーティ フェデレーション互換性リスト](active-directory-aadconnect-federation-compatibility.md)

## <a name="user-sign-in-and-user-principal-name-upn"></a>ユーザーのサインインとユーザー プリンシパル名 (UPN)
### <a name="understanding-user-principal-name"></a>ユーザー プリンシパル名について
Active Directory では、既定の UPN サフィックスは、ユーザー アカウントが作成されたドメインの DNS 名です。 ほとんどの場合、これはインターネット上で企業ドメインとして登録したドメイン名です。 ただし、Active Directory ドメインと信頼関係を使用して、複数の UPN サフィックスを追加することができます。

ユーザーの UPN の形式は、 username@domai. たとえば、アクティブなディレクトリ contoso.com に対して、ユーザー John が UPN john@contoso.com. ユーザーの UPN は RFC 822 に基づいています。 UPN と電子メールは形式が同じですが、ユーザーの UPN の値は、ユーザーの電子メール アドレスと必ずしも一致しません。

### <a name="user-principal-name-in-azure-ad"></a>Azure AD のユーザー プリンシパル名
Azure AD Connect ウィザードでは、userPrincipalName 属性が使用されるか、Azure AD のユーザー プリンシパル名としてオンプレミスで使用する属性を指定できます (カスタム インストールの場合)。 これは、Azure AD へのサインインに使用される値です。 ユーザー プリンシパル名の属性値が Azure AD の検証済みドメインに対応していない場合、その値は Azure AD によって既定値の .onmicrosoft.com に置き換えられます。

Azure Active Directory のすべてのディレクトリには、Azure をはじめとする Microsoft の各種サービスを利用できる contoso.onmicrosoft.com という形式のドメイン名が最初から存在しています。 カスタム ドメインを使用して、サインイン エクスペリエンスを改善し、簡素化することができます。 Azure AD のカスタム ドメイン名およびドメインの確認方法については、「 [Azure Active Directory へのカスタム ドメイン名の追加](active-directory-add-domain.md#add-a-custom-domain-name-to-your-directory)

## <a name="azure-ad-sign-in-configuration"></a>Azure AD サインインの構成
### <a name="azure-ad-sign-in-configuration-with-azure-ad-connect"></a>Azure AD Connect による Azure AD サインインの構成
Azure AD サインイン エクスペリエンスは、Azure AD で、同期するユーザーのユーザー プリンパル名のサフィックスを、Azure AD ディレクトリで確認されたカスタム ドメインのいずれかに照合できるかどうかによって異なります。 クラウドでのユーザーのサインイン エクスペリエンスがオンプレミスのエクスペリエンスと同様になるように、Azure AD Connect には Azure AD サインイン設定の構成を支援する機能があります。 Azure AD Connect は、ドメインに定義されている UPN サフィックスを一覧表示し、Azure AD のカスタム ドメインと照合します。これにより、必要なアクションを適切に実行できるようになります。
Azure AD サインイン ページには、オンプレミス Active Directory に定義されている UPN サフィックスの一覧と、各サフィックスに対応した状態が表示されます。 状態値は、次のいずれかです。

| 状態 | 説明 | 必要な対処 |
|:--- |:--- |:--- |
| Verified |Azure AD Connect が、Azure AD で一致する確認済みのドメインを検出しました。 このドメインのすべてのユーザーは、オンプレミスの資格情報を使用してサインインできます |対処は必要ありません |
| 未確認 |Azure AD Connect が、Azure AD で一致するカスタム ドメインを検出しましたが、そのドメインは確認されていません。 ドメインが確認されない場合、このドメインのユーザーの UPN サフィックスは、同期後に既定の .onmicrosoft.com サフィックスに変更されます。 |Azure AD でカスタム ドメインを確認します。 [詳細情報](active-directory-add-domain.md#verify-the-domain-name-with-azure-ad) |
| 追加されていません |Azure AD Connect が、UPN サフィックスに対応するカスタム ドメインを検出できませんでした。 ドメインが Azure に追加されず、確認されない場合、このドメインのユーザーの UPN サフィックスは、既定の .onmicrosoft.com に変更されます。 |UPN サフィックスに対応するカスタム ドメインを追加および確認します [詳細情報](active-directory-add-domain.md) |

Azure AD サインイン ページには、オンプレミス Active Directory に対して定義されている UPN サフィックスの一覧と、対応する Azure AD のカスタム ドメインおよび現在の確認状態が表示されます。 カスタム インストールでは、 **[Azure AD のサインイン]** ページでユーザー プリンシパル名の属性を選択できるようになりました。

![Azure AD サインイン ページ](./media/active-directory-aadconnect-user-signin/custom_azure_sign_in.png)

更新ボタンをクリックして、Azure AD のカスタム ドメインの最新状態を再取得することができます。

### <a name="selecting-attribute-for-user-principal-name-in-azure-ad"></a>Azure AD でのユーザー プリンシパル名の属性の選択
UserPrincipalName - userPrincipalName 属性は、ユーザーが Azure AD と Office 365 にサインインするときに使用する属性です。 使用するドメイン (UPN サフィックス) は、ユーザーを同期する前に、Azure AD で検証する必要があります。 既定の userPrincipalName 属性のままにしておくことを強くお勧めします。 この属性がルーティング不可能で検証できない場合は、サインイン ID を保持する属性として、電子メールなどの別の属性を選択することができます。 これを代替 ID といいます。 代替 ID の属性値は、RFC822 標準に従う必要があります。 代替 ID は、サインイン ソリューションとして、パスワード シングル サインオン (SSO) とフェデレーション SSO の両方で使用できます。

> [!NOTE]
> 代替 ID の使用は、すべての Office 365 ワークロードと互換性があるわけではありません。 詳細については、 [代替ログイン ID の構成](https://technet.microsoft.com/library/dn659436.aspx)に関するページを参照してください。
> 
> 

#### <a name="different-custom-domain-states-and-effect-on-azure-sign-in-experience"></a>さまざまなカスタム ドメインの状態と Azure サインイン エクスペリエンスへの影響
Azure AD ディレクトリのカスタム ドメインの状態と、オンプレミスで定義された UPN サフィックスの間の関係を理解することは非常に重要です。 Azure AD Connect を使用して同期を設定する場合に選択できるさまざまな Azure サインイン エクスペリエンスについて見ていきましょう。

以下の説明では、UPN サフィックスを contoso.com と想定します。これは、オンプレミス ディレクトリで UPN の一部分として使用されます (例: user@contoso.com)。

###### <a name="express-settings--password-synchronization"></a>簡単設定とパスワード同期
| 状態 | ユーザーの Azure サインイン エクスペリエンスへの影響 |
|:---:|:--- |
| 追加されていません |この場合、Azure AD ディレクトリに contoso.com のカスタム ドメインは追加されていません。 オンプレミスでサフィックス @contoso.com, の UPN を持つユーザーは、オンプレミス UPN を使用して Azure にサインインすることはできません。 その代わりに、Azure AD によって提供される新しい UPN を使用する必要があります。そのためには、既定の Azure AD ディレクトリのサフィックスを追加します。 たとえば、ユーザーを Azure AD ディレクトリ azurecontoso.onmicrosoft.com に同期する場合、オンプレミス ユーザー user@contoso.com には UPN として user@azurecontoso.onmicrosoft.com が与えられます。 |
| 未確認 |この場合、カスタム ドメイン contoso.com を Azure AD ディレクトリに追加しましたが、そのドメインはまだ確認されていません。 ドメインを確認しないままユーザーの同期を進めると、"追加されていません" のシナリオの場合と同様に、Azure AD によってユーザーに新しい UPN が割り当てられます。 |
| Verified |この場合、カスタム ドメインとして contoso.com を追加し、UPN サフィックスを Azure AD で既に確認済みです。 ユーザーは Azure AD に同期された後、オンプレミスのユーザー プリンシパル名 (例: user@contoso.com,) を使用して Azure にサインインできます |

###### <a name="ad-fs-federation"></a>AD FS のフェデレーション
Azure AD の既定の .onmicrosoft.com ドメイン、または確認されていない Azure AD のカスタム ドメインとのフェデレーションは作成できません。 Azure AD Connect ウィザードを実行している場合、フェデレーションの作成先として未確認のドメインを選択すると、ドメインに使用する DNS がホストされる場所に作成する必要があるレコードが Azure AD Connect の確認メッセージに表示されます。 詳細については、 [こちら](connect/active-directory-aadconnect-get-started-custom.md#verify-the-azure-ad-domain-selected-for-federation)を参照してください。

ユーザー サインイン オプションとして "AD FS とのフェデレーション" を選択した場合、Azure AD へのフェデレーションの作成を続行するには、カスタム ドメインが必要です。 この例では、カスタム ドメイン contoso.com が Azure AD ディレクトリに追加されている必要があります。

| 状態 | ユーザーの Azure サインイン エクスペリエンスへの影響 |
|:---:|:--- |
| 追加されていません |この場合、Azure AD Connect は UPN サフィックス contoso.com と一致するカスタム ドメインを Azure AD ディレクトリで見つけることができませんでした。 ユーザーが AD FS を使用して、user@contoso.com のようなオンプレミス UPN でサインインするように設定する場合は、カスタム ドメイン contoso.com を追加する必要があります。 |
| 未確認 |この場合、Azure AD Connect の確認メッセージに、後の段階でドメインを確認する方法について適切な詳細情報が示されます |
| Verified |この場合、他のアクションを実行することなく、構成を続行できます |

## <a name="changing-user-sign-in-method"></a>ユーザーのサインイン方法の変更
ウィザードを使用して Azure AD Connect の最初の構成を実行した後、Azure AD Connect に表示されるタスクを使用して、ユーザーのサインイン方法をフェデレーションからパスワード同期に変更できます。 Azure AD Connect ウィザードを再実行すると、実行できるタスクの一覧が表示されます。 タスクの一覧から **[ユーザー サインインの変更]** を選択します。

![ユーザー サインインの変更"](./media/active-directory-aadconnect-user-signin/changeusersignin.png)

次のページで、Azure AD の資格情報の入力を求められます。

![Azure への接続](./media/active-directory-aadconnect-user-signin/changeusersignin2.png)

**[ユーザー サインイン]** ページで、**[パスワード同期]** を選択します。 これにより、フェデレーション ディレクトリから管理ディレクトリに変更されます。

![Azure への接続](./media/active-directory-aadconnect-user-signin/changeusersignin3.png)

> [!NOTE]
> 一時的にパスワード同期に切り替えるだけの場合は、 **[ユーザー アカウントを変換しない]**をオンにします。 このオプションをオンにしないと、各ユーザーがフェデレーション ディレクトリに変換されることになり、数時間かかることがあります。
> 
> 

## <a name="next-steps"></a>次のステップ
「 [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)」をご覧ください。

「 [Azure AD Connect: 設計概念](active-directory-aadconnect-design-concepts.md)




<!--HONumber=Nov16_HO3-->


