---
title: ASP.NET Core SignalR 简介
author: bradygaster
description: 了解 ASP.NET Core 库如何 SignalR 简化将实时功能添加到应用程序。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/27/2019
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
uid: signalr/introduction
ms.openlocfilehash: 1810fef903362addcef4a6c9ec53264604f58d2b
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051468"
---
# <a name="introduction-to-aspnet-core-no-locsignalr"></a>ASP.NET Core SignalR 简介

## <a name="what-is-no-locsignalr"></a>什么是 SignalR？

ASP.NET Core SignalR 是一种开放源代码库，可简化将实时 web 功能添加到应用程序的功能。 实时 web 功能使服务器端代码可以立即将内容推送到客户端。

适用于 SignalR ：

* 需要从服务器进行高频率更新的应用。 示例包括游戏、社交网络、投票、拍卖、地图和 GPS 应用。
* 仪表板和监视应用。 示例包括公司仪表板、即时销售更新或旅行警报。
* 协作应用。 协作应用的示例包括白板应用和团队会议软件。
* 需要通知的应用。 社交网络、电子邮件、聊天、游戏、旅行警报和很多其他应用都需使用通知。

SignalR 提供一个 API，用于创建 (RPC) 的服务器到客户端 [远程过程调用 ](https://wikipedia.org/wiki/Remote_procedure_call)。 Rpc 通过服务器端 .NET Core 代码从客户端调用 JavaScript 函数。

下面是的某些功能 SignalR ASP.NET Core：

* 自动处理连接管理。
* 将消息同时发送到所有连接的客户端。 例如，聊天室。
* 向特定客户端或客户端组发送消息。
* 可缩放以处理不断增加的流量。

源托管在[ SignalR GitHub 上的存储库](https://github.com/dotnet/AspNetCore/tree/master/src/SignalR)中。

## <a name="transports"></a>传输

SignalR 支持以下用于按正常回退)  (处理实时通信的技术：

* [WebSockets](https://tools.ietf.org/html/rfc7118)
* Server-Sent 事件
* 长轮询

SignalR 自动选择服务器和客户端功能内的最佳传输方法。

## <a name="hubs"></a>中心

SignalR 使用 *集线器* 在客户端和服务器之间进行通信。

中心是一种高级管道，它允许客户端和服务器分别调用方法。 SignalR 自动处理跨计算机边界的调度，使客户端能够在服务器上调用方法，反之亦然。 可以将强类型参数传递给方法，从而启用模型绑定。 SignalR 提供了两个内置的集线器协议：基于 JSON 的文本协议和基于 [MessagePack](https://msgpack.org/)的二进制协议。  与 JSON 相比，MessagePack 通常会创建较小的消息。 较早的浏览器必须支持 [XHR 级别 2](https://caniuse.com/#feat=xhr2) ，才能提供 MessagePack 协议支持。

中心通过发送包含客户端方法的名称和参数的消息来调用客户端代码。 作为方法参数发送的对象将使用配置的协议进行反序列化。 客户端尝试将名称与客户端代码中的方法匹配。 当客户端找到匹配项时，它将调用方法并向其传递反序列化的参数数据。

## <a name="additional-resources"></a>其他资源

* [SignalRASP.NET Core 入门](xref:tutorials/signalr)
* [支持的平台](xref:signalr/supported-platforms)
* [中心](xref:signalr/hubs)
* [JavaScript 客户端](xref:signalr/javascript-client)
