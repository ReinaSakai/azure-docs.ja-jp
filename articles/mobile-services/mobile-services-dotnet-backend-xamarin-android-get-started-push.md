---
title: Xamarin Android アプリケーション用 Mobile Services の使用 | Microsoft Docs
description: Azure Mobile Services と Notification Hubs を使用して Xamarin Android アプリにプッシュ通知を送信する方法について説明します。
services: mobile-services
documentationcenter: xamarin
author: ggailey777
manager: dwrede
editor: mollybos

ms.service: mobile-services
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin-android
ms.devlang: dotnet
ms.topic: article
ms.date: 07/21/2016
ms.author: glenga

---
# Mobile Services アプリへのプッシュ通知の追加
[!INCLUDE [mobile-services-selector-get-started-push](../../includes/mobile-services-selector-get-started-push.md)]

&nbsp;

[!INCLUDE [mobile-service-note-mobile-apps](../../includes/mobile-services-note-mobile-apps.md)]

> このトピックの Mobile Apps バージョンについては、「[Xamarin.Android アプリへのプッシュ通知の追加](../app-service-mobile/app-service-mobile-xamarin-android-get-started-push.md)」を参照してください。
> 
> 

## 概要
このトピックでは、Azure Mobile Services を使用して Xamarin Android アプリにプッシュ通知を送信する方法について説明します。このチュートリアルでは、Google Cloud Messaging (GCM) を使用するプッシュ通知を [Mobile Services の使用]プロジェクトに追加します。完了すると、モバイル サービスは、レコードが挿入されるたびにプッシュ通知を送信します。

このチュートリアルには、次のものが必要です。

* アクティブな Google アカウント。
* [Google Cloud Messaging のクライアント コンポーネント]。このコンポーネントは、チュートリアル中に追加します。

Xamarin.Android コンポーネントと [Azure Mobile Services][Azure Mobile Services Component] コンポーネントは、「[Mobile Services の使用]」のチュートリアルを完了していれば、プロジェクトに既にインストールされています。

## <a id="register"></a>Google Cloud Messaging を有効にする
[!INCLUDE [GCM を有効にする](../../includes/mobile-services-enable-google-cloud-messaging.md)]

## <a id="configure"></a>プッシュ要求を送信するように Mobile Services を構成する
[!INCLUDE [mobile-services-android-configure-push](../../includes/mobile-services-android-configure-push.md)]

## <a id="update-server"></a>Mobile Services を更新してプッシュ通知を送信する
[!INCLUDE [mobile-services-dotnet-backend-android-push-update-service](../../includes/mobile-services-dotnet-backend-android-push-update-service.md)]

## <a id="configure-app"></a>プッシュ通知の既存のプロジェクトを構成する
[!INCLUDE [mobile-services-xamarin-android-push-configure-project](../../includes/mobile-services-xamarin-android-push-configure-project.md)]

## <a id="add-push"></a>アプリケーションにプッシュ通知コードを追加する
[!INCLUDE [mobile-services-xamarin-android-push-add-to-app](../../includes/mobile-services-xamarin-android-push-add-to-app.md)]

## <a name="test-app"></a>発行されたモバイル サービスに対してアプリケーションをテストする
Android フォンを USB ケーブルで直接接続するか、エミュレーターで仮想デバイスを使用する方法により、アプリケーションをテストできます。

### <a id="local-testing"></a>ローカル テストのためにプッシュ通知を有効にする
[!INCLUDE [mobile-services-dotnet-backend-configure-local-push](../../includes/mobile-services-dotnet-backend-configure-local-push.md)]

[!INCLUDE [mobile-services-android-push-notifications-test](../../includes/mobile-services-android-push-notifications-test.md)]

<!-- URLs. -->
[Mobile Services の使用]: mobile-services-dotnet-backend-xamarin-android-get-started.md
[Google Cloud Messaging のクライアント コンポーネント]: http://components.xamarin.com/view/GCMClient/
[Azure Mobile Services Component]: http://components.xamarin.com/view/azure-mobile-services/

<!---HONumber=AcomDC_0727_2016-->