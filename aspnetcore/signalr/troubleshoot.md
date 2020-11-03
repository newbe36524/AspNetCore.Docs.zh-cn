---
title: ASP.NET Core SignalR 连接故障排除
author: bradygaster
description: ASP.NET Core SignalR 连接故障排除。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 04/08/2020
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
uid: signalr/troubleshoot
ms.openlocfilehash: f1d9761267d7c6af76c0be6abb238742f40fb016
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059606"
---
# <a name="troubleshoot-connection-errors"></a>排查连接错误

本部分提供有关在尝试建立与 ASP.NET Core 集线器的连接时可能出现的错误的帮助 SignalR 。

### <a name="response-code-404"></a>响应代码404

使用 Websocket 和时 `skipNegotiation = true`
```log
WebSocket connection to 'wss://xxx/HubName' failed: Error during WebSocket handshake: Unexpected response code: 404
```

* 如果在没有粘滞会话的情况下使用多台服务器，则可以在一台服务器上启动连接，然后再切换到另一台服务器。 其他服务器无法识别上一个连接。
* 验证客户端是否正在连接到正确的终结点。 例如，服务器承载于 `http://127.0.0.1:5000/hub/myHub` 并且客户端正在尝试连接到 `http://127.0.0.1:5000/myHub` 。
* 如果连接使用该 ID，并且在协商后将请求发送到服务器的时间过长，则服务器会执行以下操作：

  * 删除 ID。
  * 返回404。

### <a name="response-code-400-or-503"></a>响应代码400或503

对于以下错误：

```log
WebSocket connection to 'wss://xxx/HubName' failed: Error during WebSocket handshake: Unexpected response code: 400

Error: Failed to start the connection: Error: There was an error with the transport.
```

此错误通常是由仅使用 Websocket 传输的客户端引起的，但该服务器上未启用 Websocket 协议。

### <a name="response-code-307"></a>响应代码307

使用 Websocket 和时 `skipNegotiation = true`
```log
WebSocket connection to 'ws://xxx/HubName' failed: Error during WebSocket handshake: Unexpected response code: 307
```

协商请求期间也会发生此错误。

常见原因：
* 应用配置为通过调用来强制使用 `UseHttpsRedirection` https `Startup` ，或通过 URL 重写规则强制使用 https。

可能的解决方案：
* 将客户端上的 URL 从 "http" 更改为 "https"。 `.withUrl("https://xxx/HubName")`

### <a name="response-code-405"></a>响应代码405

Http 状态代码 405-不允许的方法

* 应用未启用[CORS](xref:signalr/security#cross-origin-resource-sharing)

### <a name="response-code-0"></a>响应代码为0

Http 状态代码 0-通常为 [CORS](xref:signalr/security#cross-origin-resource-sharing) 问题，不提供状态代码

```log
Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:5000/default/negotiate?negotiateVersion=1. (Reason: CORS header 'Access-Control-Allow-Origin' missing).
```

* 将预期来源添加到 `.WithOrigins(...)`

```log
Cross-Origin Request Blocked: The Same Origin Policy disallows reading the remote resource at http://localhost:5000/default/negotiate?negotiateVersion=1. (Reason: expected 'true' in CORS header 'Access-Control-Allow-Credentials').
```

* 添加 `.AllowCredentials()` 到你的 CORS 策略。 不能将 `.AllowAnyOrigin()` 或 `.WithOrigins("*")` 与此选项一起使用

### <a name="response-code-413"></a>响应代码413

Http 状态代码 413-有效负载太大

这通常是由于访问令牌超过4k 而导致的。

* 如果使用 Azure SignalR 服务，请通过自定义通过服务发送的声明来减小令牌大小，方法是：
```csharp
.AddAzureSignalR(options =>
{
    options.ClaimsProvider = context => context.User.Claims;
});
```

### <a name="transient-network-failures"></a>暂时性网络故障

暂时性网络故障可能会关闭 SignalR 连接。 服务器可以将关闭的连接解释为正常的客户端断开连接。 若要获取有关在这些情况下客户端断开连接的原因的详细信息 [，请从客户端和服务器收集日志](xref:signalr/diagnostics)。