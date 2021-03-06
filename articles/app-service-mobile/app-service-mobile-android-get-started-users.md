---
title: "Mobile Apps を使用した Android での認証の追加 | Microsoft Docs"
description: "Azure App Service の Mobile Apps 機能を使用して、Google、Facebook、Twitter、Microsoft などのさまざまな ID プロバイダーを通じて Android アプリのユーザーを認証する方法について説明します。"
services: app-service\mobile
documentationcenter: android
author: ysxu
manager: erikre
editor: 
ms.assetid: 1fc8e7c1-6c3c-40f4-9967-9cf5e21fc4e1
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-android
ms.devlang: java
ms.topic: article
ms.date: 10/01/2016
ms.author: yuaxu
translationtype: Human Translation
ms.sourcegitcommit: 817626dd3fc87db61280075b80cedf8b9ed77684
ms.openlocfilehash: e638495c10742388804e75f3277c50cf1e20c6a6


---
# <a name="add-authentication-to-your-android-app"></a>Android アプリに認証を追加する
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

## <a name="summary"></a>概要
このチュートリアルでは、サポートされている ID プロバイダーを使用して、Android で todolist クイック スタート プロジェクトに認証を追加します。 最初に、このチュートリアルの基になっている [Mobile Apps の使用] チュートリアルを完了しておく必要があります。

## <a name="a-nameregisteraregister-your-app-for-authentication-and-configure-azure-app-service"></a><a name="register"></a>アプリを認証に登録し、Azure App Services を構成する
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

## <a name="a-namepermissionsarestrict-permissions-to-authenticated-users"></a><a name="permissions"></a>アクセス許可を、認証されたユーザーだけに制限する
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

* Android Studio で、[Mobile Apps の使用]に関するチュートリアルで完成させたプロジェクトを開きます。 **[Run (実行)]** メニューの **[Run app (アプリの実行)]** をクリックし、アプリの開始後に、状態コード 401 (許可されていません) のハンドルされない例外が発生することを確認します。

     この例外は、認証されないユーザーとしてアプリがバックエンドにアクセスしようとしても、*TodoItem* テーブルでは認証が要求されるために発生します。

次に、Mobile Apps バックエンドのリソースを要求する前にユーザーを認証するようにアプリを更新します。 

## <a name="add-authentication-to-the-app"></a>アプリケーションに認証を追加する
[!INCLUDE [mobile-android-authenticate-app](../../includes/mobile-android-authenticate-app.md)]

## <a name="a-namecache-tokensacache-authentication-tokens-on-the-client"></a><a name="cache-tokens"></a>クライアントに認証トークンをキャッシュする
[!INCLUDE [mobile-android-authenticate-app-with-token](../../includes/mobile-android-authenticate-app-with-token.md)]

## <a name="next-steps"></a>次のステップ
これで基本的な認証チュートリアルは完了しましたので、引き続き次のいずれかのチュートリアルのご利用を検討してください。

* [プッシュ通知を Android アプリに追加する](app-service-mobile-android-get-started-push.md)。
  Azure Notification Hubs を使用してプッシュ通知を送信するように Mobile Apps バックエンドを構成する方法について説明します。
* [Android アプリのオフライン同期を有効にする](app-service-mobile-android-get-started-offline-data.md)。
  Mobile Apps バックエンドを使用してオフライン サポートをアプリに追加する方法について説明します。 オフライン同期を使用すると、ユーザーはネットワークにアクセスできなくても、データの表示、追加、変更など、モバイル アプリケーションとやり取りできます。

<!-- Anchors. -->
[Register your app for authentication and configure Mobile Services]: #register
[Restrict table permissions to authenticated users]: #permissions
[Add authentication to the app]: #add-authentication
[Store authentication tokens on the client]: #cache-tokens
[Refresh expired tokens]: #refresh-tokens
[Next Steps]:#next-steps


<!-- URLs. -->
[Mobile Apps の使用]: app-service-mobile-android-get-started.md



<!--HONumber=Dec16_HO2-->


