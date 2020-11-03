---
title: ASP.NET Core 中的安全注意事项 SignalR
author: bradygaster
description: 了解如何在 ASP.NET Core 中使用身份验证和授权 SignalR 。
monikerRange: '>= aspnetcore-2.1'
ms.author: anurse
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
uid: signalr/security
ms.openlocfilehash: 5ecbf07b1527e9c68443870f7fce77adc29a5416
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93050831"
---
# <a name="security-considerations-in-aspnet-core-no-locsignalr"></a>ASP.NET Core 中的安全注意事项 SignalR

作者： [Andrew Stanton](https://twitter.com/anurse)

本文提供有关如何保护的信息 SignalR 。

## <a name="cross-origin-resource-sharing"></a>跨域资源共享

[跨域资源共享 (CORS) ](https://www.w3.org/TR/cors/) 可用于 SignalR 在浏览器中允许跨域连接。 如果 JavaScript 代码托管在应用的另一个域中 SignalR ，则必须启用 [CORS 中间件](xref:security/cors) ，以允许 JavaScript 连接到 SignalR 应用。 仅允许来自你信任或控制的域的跨域请求。 例如： 。

* 你的网站承载于 `http://www.example.com`
* 你的 SignalR 应用程序承载于 `http://signalr.example.com`

应在应用程序中配置 CORS，使其 SignalR 仅允许源 `www.example.com` 。

有关配置 CORS 的详细信息，请参阅 [ (CORS) 启用跨域请求 ](xref:security/cors)。 SignalR**需要** 以下 CORS 策略：

* 允许特定的预期来源。 允许任何来源是可行的，但不安全或 **不** 推荐使用。
* `GET` `POST` 必须允许使用 HTTP 方法。
* 为了使 cookie 基于的粘滞会话正常工作，必须允许使用凭据。 即使未使用身份验证，也必须启用它们。

::: moniker range=">= aspnetcore-5.0"

但是，在5.0 中，我们在 TypeScript 客户端中提供了一个不使用凭据的选项。
如果你100知道在 Cookie 你的应用 (程序中 cookie 使用多个服务器) 时，azure 应用服务使用多个服务器时，azure 应用服务使用多个凭据，则不使用凭据选项仅应使用。

::: moniker-end

例如，以下 CORS 策略允许 SignalR 中托管的浏览器客户端 `https://example.com` 访问 SignalR 托管在上的应用 `https://signalr.example.com` ：

::: moniker range=">= aspnetcore-3.0"

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    // ... other middleware ...

    // Make sure the CORS middleware is ahead of SignalR.
    app.UseCors(builder =>
    {
        builder.WithOrigins("https://example.com")
            .AllowAnyHeader()
            .WithMethods("GET", "POST")
            .AllowCredentials();
    });

    // ... other middleware ...
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chathub");
    });

    // ... other middleware ...
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[Main](security/sample/Startup.cs?name=snippet1)]

::: moniker-end

## <a name="websocket-origin-restriction"></a>WebSocket 源限制

::: moniker range=">= aspnetcore-2.2"

CORS 提供的保护不适用于 WebSocket。 对于 Websocket 上的源限制，请阅读 [websocket 源限制](xref:fundamentals/websockets#websocket-origin-restriction)。

::: moniker-end

::: moniker range="< aspnetcore-2.2"

CORS 提供的保护不适用于 WebSocket。 浏览器不会：

* 执行 CORS 预检请求。
* 在发出 WebSocket 请求时，遵守 `Access-Control` 标头中指定的限制。

但是，浏览器在发出 WebSocket 请求时会发送 `Origin` 标头。 应将应用程序配置为验证这些标头，以确保只允许来自预期来源的 WebSocket。

在 ASP.NET Core 2.1 及更高版本中，可以使用在中放置的自定义中间件和中的 **`UseSignalR` 身份验证中间** 件来实现标题验证 `Configure` ：

[!code-csharp[Main](security/sample/Startup.cs?name=snippet2)]

> [!NOTE]
> 与 `Referer` 标头一样，`Origin` 标头由客户端控制，并可以伪造。 这些标头 **不** 应用作身份验证机制。

::: moniker-end

## <a name="connectionid"></a>ConnectionId

`ConnectionId`如果 SignalR 服务器或客户端版本 ASP.NET Core 2.2 或更低版本，则公开可能会导致恶意模拟。 如果 SignalR 服务器和客户端版本 ASP.NET Core 3.0 或更高版本，则 `ConnectionToken` 而不是 `ConnectionId` 必须保持机密。 `ConnectionToken`特意不在任何 API 中公开。  很难确保较旧的 SignalR 客户端无法连接到服务器，因此即使 SignalR 服务器版本 ASP.NET Core 3.0 或更高版本，也不应 `ConnectionId` 公开。

## <a name="access-token-logging"></a>访问令牌日志记录

使用 Websocket 或 Server-Sent 事件时，浏览器客户端会在查询字符串中发送访问令牌。 使用标准标头，通过查询字符串接收访问令牌通常是安全的 `Authorization` 。 始终使用 HTTPS 确保客户端和服务器之间安全的端到端连接。 许多 web 服务器都记录每个请求的 URL，包括查询字符串。 记录 Url 可能会记录访问令牌。 默认情况下，ASP.NET Core 记录每个请求的 URL，其中将包括查询字符串。 例如： 。

```
info: Microsoft.AspNetCore.Hosting.Internal.WebHost[1]
      Request starting HTTP/1.1 GET http://localhost:5000/chathub?access_token=1234
```

如果你对将此数据与服务器日志进行日志记录有关，你可以通过将 `Microsoft.AspNetCore.Hosting` 记录器配置到级别或更高级别来完全禁用此日志记录 `Warning` (这些消息在 `Info` 级别) 写入。 有关详细信息，请参阅 [日志筛选](xref:fundamentals/logging/index#log-filtering) 。 如果仍想记录某些请求信息，可以 [编写中间件](xref:fundamentals/middleware/write) 来记录所需的数据，并筛选出 `access_token` 查询字符串值 (如果) 存在。

## <a name="exceptions"></a>异常

异常消息通常被认为是不应透露给客户端的敏感数据。 默认情况下， SignalR 不会将集线器方法引发的异常的详细信息发送到客户端。 客户端将收到指示出现错误的一般消息。 可以将异常消息传递给客户端 (例如，用 [EnableDetailedErrors](xref:signalr/configuration#configure-server-options)进行开发或测试) 。 不应在生产应用程序中向客户端公开异常消息。

## <a name="buffer-management"></a>缓冲区管理

SignalR 使用每个连接缓冲区管理传入消息和传出消息。 默认情况下， SignalR 将这些缓冲区限制为 32 KB。 客户端或服务器可以发送的最大消息为 32 KB。 消息连接使用的最大内存为 32 KB。 如果消息始终小于 32 KB，则可以减少限制：

* 禁止客户端发送更大的消息。
* 服务器绝不会需要分配大型缓冲区来接受消息。

如果消息大小超过 32 KB，则可以增加限制。 增加此限制意味着：

* 客户端可能会导致服务器分配大的内存缓冲区。
* 大缓冲区的服务器分配可能会减少并发连接的数量。

传入消息和传出消息有限制，可以在中配置的 [HttpConnectionDispatcherOptions](xref:signalr/configuration#configure-server-options) 对象上进行配置 `MapHub` ：

* `ApplicationMaxBufferSize` 表示客户端从服务器缓冲的最大字节数。 如果客户端尝试发送比此限制更大的消息，则可能会关闭该连接。
* `TransportMaxBufferSize` 表示服务器可以发送的最大字节数。 如果服务器尝试发送消息 (包括从集线器方法返回值) 大于此限制，则会引发异常。

将限制设置为以 `0` 禁用限制。 通过删除该限制，客户端可以发送任意大小的消息。 发送大消息的恶意客户端可能会导致分配额外的内存。 过度使用内存会显著减少并发连接数。
