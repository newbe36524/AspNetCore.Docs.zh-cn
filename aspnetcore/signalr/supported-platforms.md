---
title: ASP.NET Core SignalR 支持的平台
author: bradygaster
description: 了解 ASP.NET Core SignalR 支持的平台。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc, devx-track-js
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
uid: signalr/supported-platforms
ms.openlocfilehash: ee6e263fb5bef7bfb84587c3b0f04175eb8073cd
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051013"
---
# <a name="aspnet-core-no-locsignalr-supported-platforms"></a>ASP.NET Core SignalR 支持的平台

## <a name="server-system-requirements"></a>服务器系统要求

SignalR 对于 ASP.NET Core 支持 ASP.NET Core 支持的任何服务器平台。

## <a name="javascript-client"></a>JavaScript 客户端

[JavaScript 客户端](xref:signalr/javascript-client)在 NodeJS 8 及更高版本以及以下浏览器上运行：

| 浏览者                          | Version         |
| -------------------------------- | --------------- |
| Apple Safari，包括 iOS      | 当前&dagger; |
| Google Chrome，包括 Android | 当前&dagger; |
| Microsoft Edge                   | 当前&dagger; |
| Mozilla Firefox                  | 当前&dagger; |

&dagger;最新指的是浏览器的最新版本。

## <a name="net-client"></a>.NET 客户端

[.Net 客户端](xref:signalr/dotnet-client)在 ASP.NET Core 支持的任何平台上运行。 例如， [xamarin 开发人员可以使用 SignalR ](https://github.com/aspnet/Announcements/issues/305)使用 xamarin 8.4.0.1 和更高版本以及使用 xamarin 11.14.0.4 和更高版本的 ios 应用来构建 Android 应用程序。

如果服务器运行 IIS，则 Websocket 传输要求在 Windows Server 2012 或更高版本上安装 IIS 8.0 或更高版本。 所有平台都支持其他传输。

## <a name="java-client"></a>Java 客户端

[Java 客户端](xref:signalr/java-client)支持 java 8 及更高版本。

## <a name="unsupported-clients"></a>不受支持的客户端

以下客户端可用，但为试验或非正式客户端。 它们目前不受支持，可能永远不会。

* [C + + 客户端](https://github.com/aspnet/SignalR-Client-Cpp)

* [Swift 客户端](https://github.com/moozzyk/SignalR-Client-Swift)
