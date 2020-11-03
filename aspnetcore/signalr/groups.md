---
title: 在中管理用户和组 SignalR
author: bradygaster
description: ASP.NET Core SignalR 用户和组管理的概述。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 05/17/2020
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
uid: signalr/groups
ms.openlocfilehash: a86408eaae8d3df32faef79453d9db0cdbd64a78
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93050948"
---
# <a name="manage-users-and-groups-in-no-locsignalr"></a>在中管理用户和组 SignalR

作者： [Brennan Conroy](https://github.com/BrennanConroy)

SignalR 允许将消息发送到与特定用户关联的所有连接以及命名连接组。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/groups/sample/)[（如何下载）](xref:index#how-to-download-a-sample)

## <a name="users-in-no-locsignalr"></a>用户 SignalR

中的单个用户 SignalR 可以有多个到应用的连接。 例如，用户可以连接到其桌面以及他们的手机上。 每个设备都有一个单独的 SignalR 连接，但它们都与同一用户关联。 如果向用户发送一条消息，则所有与该用户关联的连接都将收到该消息。 中心中的属性可以访问连接的用户标识符 `Context.UserIdentifier` 。

默认情况下， SignalR 将 `ClaimTypes.NameIdentifier` 从 `ClaimsPrincipal` 与连接关联的中用作用户标识符。 若要自定义此行为，请参阅 [使用声明自定义标识处理](xref:signalr/authn-and-authz#use-claims-to-customize-identity-handling)。

向特定用户发送消息，方法是将用户标识符传递到 `User` 中心方法中的函数，如以下示例中所示：

> [!NOTE]
> 用户标识符区分大小写。

[!code-csharp[Configure service](groups/sample/Hubs/ChatHub.cs?range=29-32)]

## <a name="groups-in-no-locsignalr"></a>组 SignalR

组是与某个名称关联的连接的集合。 可以将消息发送到组中的所有连接。 建议将组发送到一个或多个连接，因为这些组由应用程序管理。 连接可以是多个组的成员。 组是类似聊天应用程序的理想之选，其中每个会议室都可以表示为一个组。 通过和方法将连接添加到组或从组中删除连接 `AddToGroupAsync` `RemoveFromGroupAsync` 。

[!code-csharp[Hub methods](groups/sample/Hubs/ChatHub.cs?range=15-27)]

连接重新连接时，不会保留组成员身份。 重新建立组后，连接需要重新加入组。 不能对组的成员进行计数，因为如果将应用程序扩展到多台服务器，此信息不可用。

若要在使用组时保护对资源的访问，请在 ASP.NET Core 中使用 [身份验证和授权](xref:signalr/authn-and-authz) 功能。 如果用户仅当凭据对该组有效时才添加到组中，则发送到该组的消息将仅发送到授权用户。 但组不是一种安全功能。 身份验证声明具有组不具备的功能，如过期和吊销。 如果用户对组的访问权限被吊销，应用必须从组中显式删除用户。

> [!NOTE]
> 组名区分大小写。

## <a name="related-resources"></a>相关资源

* [入门](xref:tutorials/signalr)
* [中心](xref:signalr/hubs)
* [发布到 Azure](xref:signalr/publish-to-azure-web-app)
