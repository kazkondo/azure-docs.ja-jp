---
title: "チュートリアル: Azure Active Directory と Namely の統合 | Microsoft Docs"
description: "Azure Active Directory と Namely の間でシングル サインオンを構成する方法について確認します。"
services: active-directory
documentationcenter: 
author: jeevansd
manager: prasannas
editor: 
ms.assetid: 9541d5c4-4c82-4b5b-b01a-6a3f75a2b7a1
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/20/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 3c2f2665b217b2de1656e77dae0d9b6c2b439ff5


---
# <a name="tutorial-azure-active-directory-integration-with-namely"></a>チュートリアル: Azure Active Directory と Namely の統合
このチュートリアルの目的は、Namely と Azure Active Directory (Azure AD) を統合する方法を説明することです。

Namely と Azure AD の統合には、次の利点があります。 

* Namely にアクセスするユーザーを Azure AD で管理できます。 
* ユーザーが各自の Azure AD アカウントで Namely に自動的にサインオン (シングル サインオン) するように、設定が可能です。
* 1 つの中央サイト (Azure クラシック ポータル) でアカウントを管理できます。

SaaS アプリと Azure AD の統合の詳細については、「 [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](active-directory-appssoaccess-whatis.md)」を参照してください。

## <a name="prerequisites"></a>前提条件
Azure AD と Namely の統合を構成するには、次のものが必要です。

* Azure AD サブスクリプション
* Namely でのシングル サインオンが有効なサブスクリプション

> [!NOTE]
> このチュートリアルの手順をテストする場合、運用環境を使用しないことをお勧めします。
> 
> 

このチュートリアルの手順をテストするには、次の推奨事項に従ってください。

* 必要な場合を除き、運用環境は使用しないでください。
* Azure AD の評価環境がない場合は、 [こちら](https://azure.microsoft.com/pricing/free-trial/)から 1 か月の評価版を入手できます。 

## <a name="scenario-description"></a>シナリオの説明
このチュートリアルの目的は、テスト環境で Azure AD のシングル サインオンをテストできるようにすることです。 

このチュートリアルで説明するシナリオは、主に次の 2 つの要素で構成されています。

1. ギャラリーから Namely を追加する 
2. Azure AD シングル サインオンの構成とテスト

## <a name="adding-namely-from-the-gallery"></a>ギャラリーから Namely を追加する
Azure AD への Namely の統合を構成するには、ギャラリーから管理対象 SaaS アプリの一覧に Namely を追加する必要があります。

**ギャラリーから Namely を追加するには、次の手順を実行します。**

1. **Azure クラシック ポータル**の左側のナビゲーション ウィンドウで、**[Active Directory]** をクリックします。 
   
    ![Active Directory][1]
2. **[ディレクトリ]** の一覧から、ディレクトリ統合を有効にするディレクトリを選択します。
3. アプリケーション ビューを開くには、ディレクトリ ビューでトップ メニューの **[アプリケーション]** をクリックします。
   
    ![[アプリケーション]][2]
4. ページの下部にある **[追加]** をクリックします。
   
    ![アプリケーション][3]
5. **[実行する内容]** ダイアログで、**[ギャラリーからアプリケーションを追加します]** をクリックします。
   
    ![アプリケーション][4]
6. 検索ボックスに、「 **Namely**」と入力します。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/tutorial_namely_01.png)
7. 結果ウィンドウで **[Namely]** を選択し、**[完了]** をクリックしてアプリケーションを追加します。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/tutorial_namely_02.png)

## <a name="configuring-and-testing-azure-ad-single-sign-on"></a>Azure AD シングル サインオンの構成とテスト
このセクションの目的は、"Britta Simon" というテスト ユーザーをベースに、Namely での Azure AD のシングル サインオンを構成し、テストする方法について説明することです。

シングル サインオンを機能させるには、Azure AD ユーザーに対応する Namely ユーザーが Azure AD で認識されている必要があります。 つまり、Azure AD ユーザーと Namely の関連ユーザーの間でリンク関係が確立されている必要があります。

このリンク関係を確立するには、Azure AD の **[ユーザー名]** の値を Namely の **[Username (ユーザー名)]** の値として割り当てます。

Namely で Azure AD のシングル サインオンを構成してテストするには、次の手順を完了する必要があります。

1. **[Azure AD シングル サインオンの構成](#configuring-azure-ad-single-single-sign-on)** - ユーザーがこの機能を使用できるようにします。
2. **[Azure AD のテスト ユーザーの作成](#creating-an-azure-ad-test-user)** - Britta Simon で Azure AD のシングル サインオンをテストします。
3. **[Namely のテスト ユーザーの作成](#creating-a-namely-test-user)** - Azure AD の Britta Simon にリンクさせるために、対応するユーザーを Namely で作成します。
4. **[Azure AD テスト ユーザーの割り当て](#assigning-the-azure-ad-test-user)** - Britta Simon が Azure AD のシングル サインオンを使用できるようにします。
5. **[シングル サインオンのテスト](#testing-single-sign-on)** - 構成が機能するかどうかを確認します。

### <a name="configuring-azure-ad-single-sign-on"></a>Azure AD シングル サインオンの構成
このセクションの目的は、Azure クラシック ポータルで Azure AD のシングル サインオンを有効にすることと、Namely アプリケーションでシングル サインオンを構成することです。 

**Namely で Azure AD シングル サインオンを構成するには、次の手順を実行します。**

1. Azure クラシック ポータルの **Namely** アプリケーション統合ページで **[シングル サインオンの構成]** をクリックして、**[シングル サインオンの構成]** ダイアログを開きます。
   
    ![Configure Single Sign-On][6] 
2. **[ユーザーの Namely へのアクセスを設定してください]** ページで、**[Microsoft Azure AD シングル サインオン]** を選択し、**[次へ]** をクリックします。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_03.png) 
3. **[アプリケーション設定の構成]** ダイアログ ページで、次の手順に従います。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_04.png) 
   
    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが Yardi eLearning アプリケーションへのサインオンに使用する URL を入力します。 **[サインオン URL]** ボックスに、ユーザーが Namely アプリケーションへのサインオンに使用する URL (例: *https://fabrikam.Namely.com/*) を入力します。
   
    b. **[次へ]**をクリックします。
4. **[Namely でのシングル サインオンの構成]** ページで、次の手順を実行します。
   
    ![Configure Single Sign-On](./media/active-directory-saas-namely-tutorial/tutorial_namely_05.png) 
   
    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが Yardi eLearning アプリケーションへのサインオンに使用する URL を入力します。 **[証明書のダウンロード]** をクリックし、コンピューターにファイルを保存します。
   
    b. **[次へ]**をクリックします。
5. Web ブラウザーの別のウィンドウで、Namely の企業サイトに管理者としてサインオンします。
6. 上部のツール バーの **[Company]**をクリックします。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_06.png) 
7. **[設定]** タブをクリックします。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_07.png) 
8. **[SAML]**をクリックします。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_08.png) 
9. **[SAML Settings]** ページで、次の手順を実行します。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_09.png) 
   
    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが Yardi eLearning アプリケーションへのサインオンに使用する URL を入力します。 **[Enable SAML]**をクリックします。 
   
    b. Azure クラシック ポータルの **[Namely でのシングル サインオンの構成]** ダイアログ ページで **[シングル サインオン サービス URL]** の値をコピーし、**[Identity provider SSO url (ID プロバイダー SSO URL)]** ボックスに貼り付けます。 
   
    c. ダウンロードした証明書をメモ帳で開き、その内容をコピーして、**[Identity provider certificate (ID プロバイダー証明書)]** ボックスに貼り付けます。    
   
    d. [ **Save**] をクリックします。
10. Azure クラシック ポータルで、シングル サインオンの構成確認を選択し、 **[次へ]**をクリックします。 
    
     ![Azure AD のシングル サインオン][10]
11. **[シングル サインオンの確認]** ページで、**[完了]** をクリックします。  
    
     ![Azure AD のシングル サインオン][11]

### <a name="creating-an-azure-ad-test-user"></a>Azure AD のテスト ユーザーの作成
このセクションの目的は、Azure クラシック ポータルで Britta Simon というテスト ユーザーを作成することです。

![Azure AD ユーザーの作成][20]

**Azure AD でテスト ユーザーを作成するには、次の手順に従います。**

1. **Azure クラシック ポータル**の左側のナビゲーション ウィンドウで、**[Active Directory]** をクリックします。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_09.png)  
2. **[ディレクトリ]** の一覧から、ディレクトリ統合を有効にするディレクトリを選択します。
3. 上部のメニューで **[ユーザー]**をクリックして、ユーザーの一覧を表示します。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_03.png) 
4. 下部にあるツール バーで **[ユーザーの追加]** をクリックして、**[ユーザーの追加]** ダイアログ ボックスを開きます。 
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_04.png) 
5. **[このユーザーに関する情報の入力]** ダイアログ ページで、次の手順に従います。 
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_05.png)  
   
    a.[サインオン URL] ボックスに、ユーザーが Tidemark アプリケーションへのサインオンに使用する URL を入力します。 [ユーザーの種類] として [組織内の新しいユーザー] を選択します。
   
    b. [ユーザー名] **ボックス**に「**BrittaSimon**」と入力します。
   
    c. **[次へ]**をクリックします。
6. **[ユーザー プロファイル]** ダイアログ ページで、次の手順に従います。 
   
   ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_06.png) 
   
   a.[サインオン URL] ボックスに、ユーザーが Tidemark アプリケーションへのサインオンに使用する URL を入力します。 **[名]** ボックスに「**Britta**」と入力します。  
   
   b. **[姓]** ボックスに「**Simon**」と入力します。
   
   c. **[表示名]** ボックスに「**Britta Simon**」と入力します。
   
   d. **[ロール]** 一覧で **[ユーザー]** を選択します。
   e. **[次へ]**をクリックします。
7. **[一時パスワードの取得]** ダイアログ ページで、**[作成]** をクリックします。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_07.png) 
8. **[一時パスワードの取得]** ダイアログ ページで、次の手順に従います。
   
    ![Azure AD のテスト ユーザーの作成](./media/active-directory-saas-namely-tutorial/create_aaduser_08.png) 
   
    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが Yardi eLearning アプリケーションへのサインオンに使用する URL を入力します。 **[新しいパスワード]** の値を書き留めます。
   
    b. ページの下部にある [完了]」を参照してください。   

### <a name="creating-a-namely-test-user"></a>Namely のテスト ユーザーの作成
このセクションの目的は、Namely で Britta Simon というユーザーを作成することです。

**Namely で Britta Simon というユーザーを作成するには、次の手順を実行します。**

1. Namely の企業サイトに管理者としてサインオンします。
2. 上部のツールバーの **[People]**をクリックします。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_10.png) 
3. **[ディレクトリ]** タブをクリックします。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_11.png) 
4. **[Add New Person]**をクリックします。
5. **[Add New Person]** ダイアログで、次の手順を実行します。
   
    a.[サインオン URL] ボックスに、次のパターンを使用して、ユーザーが Yardi eLearning アプリケーションへのサインオンに使用する URL を入力します。 **[First Name (名)]** ボックスに「**Britta**」と入力します。
   
    b. **[Last Name (姓)]** ボックスに「**Simon**」と入力します。
   
    c. **[Email]** ボックスに、Britta の Azure クラシック ポータルの電子メール アドレスを入力します。
   
    d. [ **Save**] をクリックします。

### <a name="assigning-the-azure-ad-test-user"></a>Azure AD テスト ユーザーの割り当て
このセクションの目的は、Namely へのアクセスを許可することで、Britta Simon が Azure のシングル サインオンを使用できるようにすることです。

![ユーザーの割り当て][200] 

**Britta Simon を Namely に割り当てるには、次の手順を実行します。**

1. Azure クラシック ポータルでアプリケーション ビューを開くために、ディレクトリ ビューでトップ メニューの **[アプリケーション]** をクリックします。
   
    ![ユーザーの割り当て][201] 
2. アプリケーションの一覧で **[Namely]**を選択します。
   
    ![[シングル サインオンの構成]](./media/active-directory-saas-namely-tutorial/tutorial_namely_50.png) 
3. 上部のメニューで **[ユーザー]**をクリックします。
   
    ![ユーザーの割り当て][203] 
4. ユーザーの一覧で **[Britta Simon]**を選択します。
5. 下部にあるツール バーで **[割り当て]**をクリックします。
   
    ![ユーザーの割り当て][205]

### <a name="testing-single-sign-on"></a>シングル サインオンのテスト
このセクションの目的は、アクセス パネルを使用して Azure AD のシングル サインオン構成をテストすることです。

アクセス パネルで [Namely] タイルをクリックすると、Namely アプリケーションに自動的にサインオンします。

## <a name="additional-resources"></a>その他のリソース
* [SaaS アプリと Azure Active Directory を統合する方法に関するチュートリアルの一覧](active-directory-saas-tutorial-list.md)
* [Azure Active Directory のアプリケーション アクセスとシングル サインオンとは](active-directory-appssoaccess-whatis.md)

<!--Image references-->

[1]: ./media/active-directory-saas-namely-tutorial/tutorial_general_01.png
[2]: ./media/active-directory-saas-namely-tutorial/tutorial_general_02.png
[3]: ./media/active-directory-saas-namely-tutorial/tutorial_general_03.png
[4]: ./media/active-directory-saas-namely-tutorial/tutorial_general_04.png

[6]: ./media/active-directory-saas-namely-tutorial/tutorial_general_05.png
[10]: ./media/active-directory-saas-namely-tutorial/tutorial_general_06.png
[11]: ./media/active-directory-saas-namely-tutorial/tutorial_general_07.png
[20]: ./media/active-directory-saas-namely-tutorial/tutorial_general_100.png

[200]: ./media/active-directory-saas-namely-tutorial/tutorial_general_200.png
[201]: ./media/active-directory-saas-namely-tutorial/tutorial_general_201.png
[203]: ./media/active-directory-saas-namely-tutorial/tutorial_general_203.png
[204]: ./media/active-directory-saas-namely-tutorial/tutorial_general_204.png
[205]: ./media/active-directory-saas-namely-tutorial/tutorial_general_205.png









<!--HONumber=Nov16_HO3-->


