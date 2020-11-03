---
title: 在 ASP.NET Core 中使用中心 SignalR
author: bradygaster
description: 了解如何在 ASP.NET Core 中使用集线器 SignalR 。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/16/2020
no-loc:
- appsettings.json
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: signalr/hubs
ms.openlocfilehash: 4a31c16eb44e2244574d0df49c30e7a44b2bba6e
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93050935"
---
# <a name="use-hubs-in-no-locsignalr-for-aspnet-core"></a>使用中 SignalR 的中心进行 ASP.NET Core

作者： [Rachel Appel](https://twitter.com/rachelappel) 和 [古柯 Griffin](https://twitter.com/1kevgriff)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/hubs/sample/ )[（如何下载）](xref:index#how-to-download-a-sample)

## <a name="what-is-a-no-locsignalr-hub"></a>什么是 SignalR 中心

SignalR利用中心 API，你可以从服务器对连接的客户端调用方法。 在服务器代码中，您将定义由客户端调用的方法。 在客户端代码中，您将定义从服务器调用的方法。 SignalR 处理幕后的所有事情，使客户端到服务器和服务器到客户端的实时通信成为可能。

## <a name="configure-no-locsignalr-hubs"></a>配置 SignalR 中心

SignalR中间件需要一些服务，这些服务通过调用配置 `services.AddSignalR` 。

[!code-csharp[Configure service](hubs/sample/startup.cs?range=38)]

::: moniker range=">= aspnetcore-3.0"

向 SignalR ASP.NET Core 应用程序添加功能时， SignalR 通过 `endpoint.MapHub` 在 `Startup.Configure` 方法的回调中调用来设置路由 `app.UseEndpoints` 。

```csharp
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapHub<ChatHub>("/chathub");
});
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

将 SignalR 功能添加到 ASP.NET Core 的应用时， SignalR 通过 `app.UseSignalR` 在方法中调用来设置路由 `Startup.Configure` 。

[!code-csharp[Configure routes to hubs](hubs/sample/startup.cs?range=57-60)]

::: moniker-end

## <a name="create-and-use-hubs"></a>创建和使用集线器

通过声明从继承的类 `Hub` ，并向其添加公共方法来创建中心。 客户端可以调用定义为的方法 `public` 。

```csharp
public class ChatHub : Hub
{
    public Task SendMessage(string user, string message)
    {
        return Clients.All.SendAsync("ReceiveMessage", user, message);
    }
}
```

可以指定返回类型和参数（包括复杂类型和数组），就像在任何 c # 方法中一样。 SignalR 处理参数和返回值中的复杂对象和数组的序列化和反序列化。

> [!NOTE]
> 中心是暂时性的：
>
> * 不要将状态存储在 hub 类的属性中。 每个中心方法调用在新的中心实例上执行。
> * `await`调用依赖于中心保持活动状态的异步方法时使用。 例如， `Clients.All.SendAsync(...)` 如果在未调用的情况下调用，则方法会失败， `await` 并且在完成前集线器方法完成 `SendAsync` 。

## <a name="the-context-object"></a>上下文对象

`Hub`类具有一个 `Context` 属性，该属性包含有关连接的信息的以下属性：

| 属性 | 说明 |
| ------ | ----------- |
| `ConnectionId` | 获取由分配的连接的唯一 ID SignalR 。 每个连接都有一个连接 ID。|
| `UserIdentifier` | 获取 [用户标识符](xref:signalr/groups)。 默认情况下， SignalR 将 `ClaimTypes.NameIdentifier` 从 `ClaimsPrincipal` 与连接关联的中用作用户标识符。 |
| `User` | 获取 `ClaimsPrincipal` 与当前用户关联的。 |
| `Items` | 获取可用于在此连接的范围内共享数据的键/值集合。 数据可以存储在此集合中，它将在不同的集线器方法调用中持久保存。 |
| `Features` | 获取连接上的可用功能的集合。 目前，在大多数情况下不需要此集合，因此不会对其进行详细介绍。 |
| `ConnectionAborted` | 获取一个 `CancellationToken` ，它将在连接中止时通知。 |

`Hub.Context` 还包含以下方法：

| 方法 | 说明 |
| ------ | ----------- |
| `GetHttpContext` | 返回 `HttpContext` 连接的， `null` 如果连接不与 HTTP 请求关联，则为。 对于 HTTP 连接，可以使用此方法来获取 HTTP 标头和查询字符串等信息。 |
| `Abort` | 中止连接。 |

## <a name="the-clients-object"></a>Clients 对象

`Hub`类具有一个 `Clients` 属性，该属性包含服务器和客户端之间的通信的以下属性：

| 属性 | 说明 |
| ------ | ----------- |
| `All` | 在所有连接的客户端上调用方法 |
| `Caller` | 在调用集线器方法的客户端上调用方法 |
| `Others` | 在所有连接的客户端上调用方法，但调用方法的客户端除外 |

`Hub.Clients` 还包含以下方法：

| 方法 | 说明 |
| ------ | ----------- |
| `AllExcept` | 在所有连接的客户端（指定的连接除外）上调用方法 |
| `Client` | 在特定连接的客户端上调用方法 |
| `Clients` | 在特定连接的客户端上调用方法 |
| `Group` | 对指定组中的所有连接调用方法  |
| `GroupExcept` | 对指定组中的所有连接调用方法，指定的连接除外 |
| `Groups` | 在多组连接上调用方法  |
| `OthersInGroup` | 对一组连接调用方法，而不包括调用该集线器方法的客户端  |
| `User` | 对与特定用户关联的所有连接调用方法 |
| `Users` | 对与指定用户相关联的所有连接调用方法 |

前面的表中的每个属性或方法都返回一个包含方法的对象 `SendAsync` 。 `SendAsync`方法允许你提供要调用的客户端方法的名称和参数。

## <a name="send-messages-to-clients"></a>向客户端发送消息

若要调用特定的客户端，请使用对象的属性 `Clients` 。 在下面的示例中，有三种集线器方法：

* `SendMessage` 使用将消息发送到所有连接的客户端 `Clients.All` 。
* `SendMessageToCaller` 使用将消息发回给调用方 `Clients.Caller` 。
* `SendMessageToGroups` 向组中的所有客户端发送一条消息 `SignalR Users` 。

[!code-csharp[Send messages](hubs/sample/hubs/chathub.cs?name=HubMethods)]

## <a name="strongly-typed-hubs"></a>强类型中心

使用的缺点 `SendAsync` 是它依赖于幻字符串来指定要调用的客户端方法。 如果客户端中的方法名称拼写错误或缺失，则这会使代码对运行时错误开放。

使用的替代方法 `SendAsync` 是使用强类型 `Hub` <xref:Microsoft.AspNetCore.SignalR.Hub%601> 。 在下面的示例中， `ChatHub` 客户端方法已提取到名为的接口 `IChatClient` 。

[!code-csharp[Interface for IChatClient](hubs/sample/hubs/ichatclient.cs?name=snippet_IChatClient)]

此接口可用于重构前面的 `ChatHub` 示例。

[!code-csharp[Strongly typed ChatHub](hubs/sample/hubs/StronglyTypedChatHub.cs?range=8-18,36)]

通过使用 `Hub<IChatClient>` ，可以对客户端方法进行编译时检查。 这可以防止由于使用神奇字符串而导致的问题，因为 `Hub<T>` 只能提供对在接口中定义的方法的访问。

使用强类型 `Hub<T>` 禁用功能 `SendAsync` 。 接口上定义的任何方法仍可以定义为异步方法。 事实上，其中每个方法应返回 `Task` 。 由于它是一个接口，因此请勿使用 `async` 关键字。 例如： 。

```csharp
public interface IClient
{
    Task ClientMethod();
}
```

> [!NOTE]
> `Async`后缀不会从方法名称中去除。 除非您的客户端方法是使用定义的 `.on('MyMethodAsync')` ，否则不应使用 `MyMethodAsync` 作为名称。

## <a name="change-the-name-of-a-hub-method"></a>更改集线器方法的名称

默认情况下，服务器集线器方法名称是 .NET 方法的名称。 但是，可以使用 [HubMethodName](xref:Microsoft.AspNetCore.SignalR.HubMethodNameAttribute) 属性来更改此默认设置，并手动指定方法的名称。 调用方法时，客户端应使用此名称，而不是 .NET 方法名称。

[!code-csharp[HubMethodName attribute](hubs/sample/hubs/chathub.cs?name=HubMethodName&highlight=1)]

## <a name="handle-events-for-a-connection"></a>处理连接事件

SignalR中心 API 提供 `OnConnectedAsync` 和 `OnDisconnectedAsync` 虚拟方法来管理和跟踪连接。 重写 `OnConnectedAsync` 虚拟方法，以便在客户端连接到集线器时执行操作，如将其添加到组。

[!code-csharp[Handle connection](hubs/sample/hubs/chathub.cs?name=OnConnectedAsync)]

重写 `OnDisconnectedAsync` 虚拟方法，以便在客户端断开连接时执行操作。 如果客户端通过调用 (特意断开连接 `connection.stop()` ，例如) ，则 `exception` 参数将为 `null` 。 但是，如果客户端由于错误而断开连接 (例如，) 网络故障，则 `exception` 参数将包含描述失败的异常。

[!code-csharp[Handle disconnection](hubs/sample/hubs/chathub.cs?name=OnDisconnectedAsync)]

[!INCLUDE[](~/includes/connectionid-signalr.md)]

## <a name="handle-errors"></a>处理错误

在中心方法中引发的异常将发送到调用方法的客户端。 在 JavaScript 客户端上，该 `invoke` 方法返回 [javascript 承诺](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises)。 当客户端接收到使用附加到承诺的处理程序的错误时 `catch` ，它将作为 JavaScript 对象进行调用和传递 `Error` 。

[!code-javascript[Error](hubs/sample/wwwroot/js/chat.js?range=23)]

如果中心引发异常，则不会关闭连接。 默认情况下， SignalR 将向客户端返回一般性错误消息。 例如： 。

```
Microsoft.AspNetCore.SignalR.HubException: An unexpected error occurred invoking 'MethodName' on the server.
```

意外的异常通常包含敏感信息，例如数据库连接失败时触发的异常中的数据库服务器的名称。 SignalR 默认情况下，不会公开这些详细的错误消息作为安全措施。 有关抑制异常详细信息的原因的详细信息，请参阅 [安全注意事项一文](xref:signalr/security#exceptions) 。

如果 *要将* 异常情况传播到客户端，则可以使用 `HubException` 类。 如果 `HubException` 从中心方法引发，则 SignalR **会将** 整个消息发送到客户端（未修改）。

[!code-csharp[ThrowHubException](hubs/sample/hubs/chathub.cs?name=ThrowHubException&highlight=3)]

> [!NOTE]
> SignalR 仅将 `Message` 异常的属性发送到客户端。 异常的堆栈跟踪和其他属性不适用于客户端。

## <a name="related-resources"></a>相关资源

* [ASP.NET Core 简介 SignalR](xref:signalr/introduction)
* [JavaScript 客户端](xref:signalr/javascript-client)
* [发布到 Azure](xref:signalr/publish-to-azure-web-app)
