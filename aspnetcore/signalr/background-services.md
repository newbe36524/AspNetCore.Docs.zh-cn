---
title: SignalR在后台服务中托管 ASP.NET Core
author: bradygaster
description: 了解如何 SignalR 通过 .Net Core BackgroundService 类将消息发送到客户端。
monikerRange: '>= aspnetcore-2.2'
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
uid: signalr/background-services
ms.openlocfilehash: 810eff7ccb08ecc22ea255bf0a9fe3d22637179f
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060100"
---
# <a name="host-aspnet-core-no-locsignalr-in-background-services"></a>SignalR在后台服务中托管 ASP.NET Core

作者： [Brady Gaster](https://twitter.com/bradygaster)

本文提供以下内容的指导：

* SignalR使用宿主 ASP.NET Core 的后台工作进程托管中心。
* 从 .NET Core [BackgroundService](xref:Microsoft.Extensions.Hosting.BackgroundService)中将消息发送到已连接的客户端。

::: moniker range=">= aspnetcore-3.0"

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/background-service/samples/3.x)[（如何下载）](xref:index#how-to-download-a-sample)

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/background-service/samples/2.2)[（如何下载）](xref:index#how-to-download-a-sample)

::: moniker-end

## <a name="enable-no-locsignalr-in-startup"></a>SignalR在启动时启用

::: moniker range=">= aspnetcore-3.0"

SignalR在后台工作进程的上下文中托管 ASP.NET Core 中心与在 ASP.NET Core 的 web 应用中托管集线器完全相同。 在 `Startup.ConfigureServices` 方法中，调用会 `services.AddSignalR` 将所需的服务添加到 ASP.NET Core 依赖关系注入 (DI) 层以支持 SignalR 。 在中 `Startup.Configure` ， `MapHub` 调用方法 `UseEndpoints` 以连接 ASP.NET Core 请求管道中的中心终结点。

[!code-csharp[Startup](background-service/samples/3.x/Server/Startup.cs?name=Startup)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

SignalR在后台工作进程的上下文中托管 ASP.NET Core 中心与在 ASP.NET Core 的 web 应用中托管集线器完全相同。 在 `Startup.ConfigureServices` 方法中，调用会 `services.AddSignalR` 将所需的服务添加到 ASP.NET Core 依赖关系注入 (DI) 层以支持 SignalR 。 在中 `Startup.Configure` ， `UseSignalR` 调用方法以将中心终结点连接到 ASP.NET Core 请求管道中的)  (。

[!code-csharp[Startup](background-service/samples/2.2/Server/Startup.cs?name=Startup)]

::: moniker-end

在前面的示例中， `ClockHub` 类实现 `Hub<T>` 类以创建强类型中心。 已 `ClockHub` 在类中配置， `Startup` 以响应终结点上的请求 `/hubs/clock` 。

有关强类型化集线器的详细信息，请参阅 [使用中 SignalR 的中心进行 ASP.NET Core](xref:signalr/hubs#strongly-typed-hubs)。

> [!NOTE]
> 此功能并不限于[集线器 \<T> ](xref:Microsoft.AspNetCore.SignalR.Hub`1)类。 从 [中心](xref:Microsoft.AspNetCore.SignalR.Hub)继承的任何类（如 [DynamicHub](xref:Microsoft.AspNetCore.SignalR.DynamicHub)）都适用。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/ClockHub.cs?name=ClockHub)]

::: moniker-end

强类型化所使用的接口 `ClockHub` 是 `IClock` 接口。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/HubServiceInterfaces/IClock.cs?name=IClock)]

::: moniker-end

## <a name="call-a-no-locsignalr-hub-from-a-background-service"></a>SignalR从后台服务调用中心

在启动过程中， `Worker` `BackgroundService` 使用启用类 `AddHostedService` 。

```csharp
services.AddHostedService<Worker>();
```

由于 SignalR 还会在 `Startup` 阶段启用，其中每个中心均附加到 ASP.NET CORE 的 HTTP 请求管道中的单个终结点，因此，每个中心都由 `IHubContext<T>` 服务器上的表示。 使用 ASP.NET Core 的 DI 功能，由宿主层实例化的其他类（如 `BackgroundService` 类、MVC 控制器类或 Razor 页模型）可以 `IHubContext<ClockHub, IClock>` 在构造过程中接受实例，从而获取对服务器端集线器的引用。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[Startup](background-service/samples/3.x/Server/Worker.cs?name=Worker)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Startup](background-service/samples/2.2/Server/Worker.cs?name=Worker)]

::: moniker-end

由于 `ExecuteAsync` 在后台服务中以迭代方式调用方法，服务器的当前日期和时间将使用发送到已连接的客户端 `ClockHub` 。

## <a name="react-to-no-locsignalr-events-with-background-services"></a>SignalR通过后台服务对事件做出响应

与使用适用于或 .NET 桌面应用程序的 JavaScript 客户端的单页面应用一样 SignalR <xref:signalr/dotnet-client> ，也可以使用、 `BackgroundService` 或 `IHostedService` 实现来连接到 SignalR 集线器并响应事件。

`ClockHubClient`类实现 `IClock` 接口和 `IHostedService` 接口。 这样一来，就可以在过程中将其启用 `Startup` ，并对来自服务器的中心事件做出响应。

```csharp
public partial class ClockHubClient : IClock, IHostedService
{
}
```

在初始化期间，会 `ClockHubClient` 创建一个实例 `HubConnection` ，并将该 `IClock.ShowTime` 方法作为中心事件的处理程序启用 `ShowTime` 。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[The ClockHubClient constructor](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

在 `IHostedService.StartAsync` 实现中，以 `HubConnection` 异步方式启动。

[!code-csharp[StartAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

在 `IHostedService.StopAsync` 方法中，将 `HubConnection` 异步释放。

[!code-csharp[StopAsync method](background-service/samples/3.x/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end
::: moniker range="<= aspnetcore-2.2"

[!code-csharp[The ClockHubClient constructor](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=ClockHubClientCtor)]

在 `IHostedService.StartAsync` 实现中，以 `HubConnection` 异步方式启动。

[!code-csharp[StartAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StartAsync)]

在 `IHostedService.StopAsync` 方法中，将 `HubConnection` 异步释放。

[!code-csharp[StopAsync method](background-service/samples/2.2/Clients.ConsoleTwo/ClockHubClient.cs?name=StopAsync)]

::: moniker-end

## <a name="additional-resources"></a>其他资源

* [入门](xref:tutorials/signalr)
* [中心](xref:signalr/hubs)
* [发布到 Azure](xref:signalr/publish-to-azure-web-app)
* [强类型中心](xref:signalr/hubs#strongly-typed-hubs)
