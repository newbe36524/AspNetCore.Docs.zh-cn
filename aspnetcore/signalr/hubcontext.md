---
title: SignalR HubContext
author: bradygaster
description: 了解如何使用 ASP.NET Core SignalR HubContext 服务向中心外部的客户端发送通知。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/12/2019
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
- IHubContext
uid: signalr/hubcontext
ms.openlocfilehash: 0b1940dc85634051e8a566c6859f51c130b69269
ms.sourcegitcommit: 1b7f2e1aabf43fa93b920cad36515d7336bfc2df
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93066728"
---
# <a name="send-messages-from-outside-a-hub"></a>从中心外发送消息

作者：[Mikael Mengistu](https://twitter.com/MikaelM_12)

SignalR中心是将消息发送到连接到服务器的客户端的核心抽象 SignalR 。 还可以使用服务从应用程序中的其他位置发送消息 `IHubContext` 。 本文介绍如何访问 SignalR `IHubContext` 以将通知从中心外发送到客户端。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/hubcontext/sample/)[（如何下载）](xref:index#how-to-download-a-sample)

## <a name="get-an-instance-of-no-locihubcontext"></a>获取实例 IHubContext

在 ASP.NET Core 中 SignalR ，可以 `IHubContext` 通过依赖关系注入访问的实例。 可以将的实例注入 `IHubContext` 控制器、中间件或其他 DI 服务。 使用实例将消息发送到客户端。

> [!NOTE]
> 这不同于 ASP.NET 4.x SignalR ，后者使用 GlobalHost 提供对的访问 `IHubContext` 。 ASP.NET Core 具有依赖关系注入框架，无需此全局单一实例。

### <a name="inject-an-instance-of-no-locihubcontext-in-a-controller"></a>IHubContext在控制器中注入实例

您可以 `IHubContext` 通过将实例添加到构造函数中，将实例注入控制器：

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=12-19,57)]

现在，有权访问的实例 `IHubContext` ，你可以调用中心方法，就像在中心内一样。

[!code-csharp[IHubContext](hubcontext/sample/Controllers/HomeController.cs?range=21-25)]

### <a name="get-an-instance-of-no-locihubcontext-in-middleware"></a>获取 IHubContext 中间件中的实例

访问 `IHubContext` 中间件管道中的，如下所示：

```csharp
app.Use(async (context, next) =>
{
    var hubContext = context.RequestServices
                            .GetRequiredService<IHubContext<ChatHub>>();
    //...
    
    if (next != null)
    {
        await next.Invoke();
    }
});
```

> [!NOTE]
> 当从类的外部调用中心方法时 `Hub` ，没有与调用关联的调用方。 因此，没有对 `ConnectionId` 、和属性的访问权限 `Caller` `Others` 。

### <a name="get-an-instance-of-no-locihubcontext-from-ihost"></a>从 IHost 获取的实例 IHubContext

`IHubContext`从 web 主机访问可用于与 ASP.NET Core 以外的区域进行集成，例如，使用第三方依赖关系注入框架：

```csharp
    public class Program
    {
        public static void Main(string[] args)
        {
            var host = CreateHostBuilder(args).Build();
            var hubContext = host.Services.GetService(typeof(IHubContext<ChatHub>));
            host.Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder => {
                    webBuilder.UseStartup<Startup>();
                });
    }
```

### <a name="inject-a-strongly-typed-hubcontext"></a>注入强类型 HubContext

若要注入强类型 HubContext，请确保中心继承自 `Hub<T>` 。 使用 `IHubContext<THub, T>` 接口而不是插入它 `IHubContext<THub>` 。

```csharp
public class ChatController : Controller
{
    public IHubContext<ChatHub, IChatClient> _strongChatHubContext { get; }

    public ChatController(IHubContext<ChatHub, IChatClient> chatHubContext)
    {
        _strongChatHubContext = chatHubContext;
    }

    public async Task SendMessage(string user, string message)
    {
        await _strongChatHubContext.Clients.All.ReceiveMessage(user, message);
    }
}
```

有关详细信息，请参阅 [强类型中心](xref:signalr/hubs#strongly-typed-hubs) 。

## <a name="related-resources"></a>相关资源

* [入门](xref:tutorials/signalr)
* [中心](xref:signalr/hubs)
* [发布到 Azure](xref:signalr/publish-to-azure-web-app)
