---
title: 'Azure AD Connect 同期: Azure AD サービス アカウントを管理する方法 | Microsoft Docs'
description: このトピックでは、Azure AD サービス アカウントを復元する方法を説明します。
services: active-directory
keywords: AADSTS70002, AADSTS50054, Azure AD Connect 同期コネクタ サービス アカウントのパスワードをリセットする方法
documentationcenter: ''
author: andkjell
manager: femila
editor: ''

ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/01/2016
ms.author: andkjell

---
# Azure AD Connect 同期: Azure AD サービス アカウントを管理する方法
Azure AD コネクタで使用されるサービス アカウントは、無料のサービスであると想定されています。その資格情報をリセットする必要がある場合、このトピックが役立ちます。たとえば、グローバル管理者が PowerShell を使用してサービス アカウントのパスワードを誤ってリセットしてしまった場合などです。

## 資格情報をリセットする
Azure AD コネクタで定義されたサービス アカウントが認証の問題のために Azure AD に接続できない場合、パスワードをリセットすることができます。

1. Azure AD Connect 同期サーバーにサインインし、PowerShell を起動します。
2. `Add-ADSyncAADServiceAccount` を実行します。 ![PowerShell コマンドレット addadsyncaadserviceaccount](./media/active-directory-aadconnectsync-howto-azureadaccount/addadsyncaadserviceaccount.png)
3. Azure AD グローバル管理者の資格情報を指定します。

このコマンドレットは、Azure AD と同期エンジンの両方で、サービス アカウントのパスワードをリセットして更新します。

## 上記の手順で解決できる既知の問題
このセクションでは、ユーザーから報告された、Azure AD サービス アカウントの資格情報をリセットすることで修正されたエラーの一覧を示します。

- - -
イベント 6900 The server encountered an unexpected error while processing a password change notification (パスワード変更通知の処理中に、サーバーで予期しないエラーが発生しました): AADSTS70002: Error validating credentials. (資格情報の検証中にエラーが発生しました。)AADSTS50054: Old password is used for authentication. (古いパスワードが認証に使用されています。)

- - -
イベント 659 Error while retrieving password policy sync configuration. (パスワード ポリシーの同期構成を取得しているときにエラーが発生しました。)Microsoft.IdentityModel.Clients.ActiveDirectory.AdalServiceException: AADSTS70002: Error validating credentials. (資格情報の検証中にエラーが発生しました。)AADSTS50054: Old password is used for authentication. (古いパスワードが認証に使用されています。)

## 次のステップ
**概要トピック**

* [Azure AD Connect sync: 同期を理解してカスタマイズする](active-directory-aadconnectsync-whatis.md)
* [オンプレミス ID と Azure Active Directory の統合](active-directory-aadconnect.md)

<!---HONumber=AcomDC_0907_2016-->