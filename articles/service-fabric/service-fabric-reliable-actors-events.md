---
title: Reliable Actors のイベント | Microsoft Docs
description: Service Fabric Reliable Actors のイベントの概要
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: ''

ms.service: service-fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/30/2016
ms.author: amanbha

---
# アクター イベント
アクター イベントは、アクターからクライアントにベスト エフォート通知を送信する方法を提供します。アクター イベントは、アクターとクライアントの通信用に設計されており、アクター間の通信には使用できません。

次のコード スニペットは、アプリケーションでアクター イベントを使用する方法を示しています。

アクターによって発行されたイベントを表すインターフェイスを定義します。このインターフェイスは、`IActorEvents` インターフェイスから派生する必要があります。メソッドの引数は、[データ コントラクト シリアル化可能](service-fabric-reliable-actors-notes-on-actor-type-serialization.md)である必要があります。イベント通知が一方向かつベスト エフォートであるため、メソッドは void を返す必要があります。

```csharp
public interface IGameEvents : IActorEvents
{
    void GameScoreUpdated(Guid gameId, string currentScore);
}
```

アクター インターフェイスで、アクターによって発行されたイベントを宣言します。

```csharp
public interface IGameActor : IActor, IActorEventPublisher<IGameEvents>
{
    Task UpdateGameStatus(GameStatus status);

    Task<string> GetGameScore();
}
```

クライアント側で、イベント ハンドラーを実装します。

```csharp
class GameEventsHandler : IGameEvents
{
    public void GameScoreUpdated(Guid gameId, string currentScore)
    {
        Console.WriteLine(@"Updates: Game: {0}, Score: {1}", gameId, currentScore);
    }
}
```

クライアント側で、イベントを発行するアクターに対してプロキシを作成し、そのイベントをサブスクライブします。

```csharp
var proxy = ActorProxy.Create<IGameActor>(
                    new ActorId(Guid.Parse(arg)), ApplicationName);

await proxy.SubscribeAsync<IGameEvents>(new GameEventsHandler());
```

フェールオーバーが発生した場合、アクターは別のプロセスまたはノードにフェールオーバーする可能性があります。アクター プロキシは、アクティブなサブスクリプションを管理し、自動的に再サブスクライブします。`ActorProxyEventExtensions.SubscribeAsync<TEvent>` API を介して、再サブスクリプションの間隔を制御できます。サブスクリプションを解除するには、`ActorProxyEventExtensions.UnsubscribeAsync<TEvent>` API を使用します。

アクターで、イベントが発生したときに単純に発行します。イベントをサブスクライブしているユーザーがいる場合、アクター ランタイムはこれらのユーザーに通知を送信します。

```csharp
var ev = GetEvent<IGameEvents>();
ev.GameScoreUpdated(Id.GetGuidId(), score);
```

## 次のステップ
* [アクターの再入](service-fabric-reliable-actors-reentrancy.md)
* [アクターの診断とパフォーマンスの監視](service-fabric-reliable-actors-diagnostics.md)
* [Actor API リファレンス ドキュメント](https://msdn.microsoft.com/library/azure/dn971626.aspx)
* [コード サンプル](https://github.com/Azure/servicefabric-samples)

<!---HONumber=AcomDC_0907_2016-->