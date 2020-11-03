---
title: SignalR将 ASP.NET Core 应用程序发布到 Azure App Service
author: bradygaster
description: 了解如何将 ASP.NET Core SignalR 应用程序发布到 Azure App Service。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/02/2020
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
uid: signalr/publish-to-azure-web-app
ms.openlocfilehash: 8e6d36fe0b38486f94078b8f9cf12b852da7e0d9
ms.sourcegitcommit: d64bf0cbe763beda22a7728c7f10d07fc5e19262
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2020
ms.locfileid: "93234500"
---
# <a name="publish-an-aspnet-core-no-locsignalr-app-to-azure-app-service"></a>SignalR将 ASP.NET Core 应用程序发布到 Azure App Service

作者： [Brady Gaster](https://twitter.com/bradygaster)

[Azure App Service](/azure/app-service/app-service-web-overview) 是一种 [Microsoft 云计算](https://azure.microsoft.com/) 平台服务，用于承载 web 应用，包括 ASP.NET Core。

> [!NOTE]
> 本文介绍如何 SignalR 从 Visual Studio 发布 ASP.NET Core 应用。 有关详细信息，请参阅[ SignalR Azure 服务](https://azure.microsoft.com/services/signalr-service)。

## <a name="publish-the-app"></a>发布应用

本文介绍如何使用 Visual Studio 中的工具进行发布。 Visual Studio Code 用户可以使用 [Azure CLI](/cli/azure) 命令将应用发布到 Azure。 有关详细信息，请参阅 [使用命令行工具将 ASP.NET Core 应用程序发布到 Azure](/azure/app-service/app-service-web-get-started-dotnet)。

1. 在“解决方案资源管理器”中右键单击该项目，然后选择“发布”。

1. 确认已在 " **选取发布目标** " 对话框中选择 " **应用服务** " 和 " **新建** "。

1. 从 " **发布** " 按钮下拉菜单中选择 " **创建配置文件** "。

   在 " **创建应用服务** " 对话框中，输入下表中所述的信息，然后选择 " **创建** "。

   | 项目               | 说明 |
   | ------------------ | ----------- |
   | **名称**           | 应用的唯一名称。 |
   | **订阅**   | 应用使用的 Azure 订阅。 |
   | **资源组** | 应用所属的一组相关资源。 |
   | **托管计划**   | Web 应用的定价计划。 |

1. 选择 " **服务依赖关系** " 部分中的 " **Azure SignalR 服务** "。 选择 **+** 按钮：

   !["依赖关系" 区域显示在 "添加" 下拉列表中选择的 Azure：：： no (SignalR) ：：： Service](publish-to-azure-web-app/_static/signalr-service-dependency.png)

1. 在 " **Azure SignalR 服务** " 对话框中，选择 " **创建新的 Azure SignalR 服务实例** "。

1. 提供 **名称** 、 **资源组** 和 **位置** 。 返回到 " **Azure SignalR 服务** " 对话框，然后选择 " **添加** "。

Visual Studio 完成以下任务：

* 创建包含发布设置的发布配置文件。
* 使用提供的详细信息创建 *Azure Web 应用* 。
* 发布应用程序。
* 启动加载 web 应用程序的浏览器。

应用的 URL 的格式为 `{APP SERVICE NAME}.azurewebsites.net` 。 例如，名为的应用程序 `SignalRChatApp` 具有的 URL `https://signalrchatapp.azurewebsites.net` 。

如果在部署面向预览版 .NET Core 版本的应用时发生 HTTP *502.2 错误的网关* 错误，请参阅 [部署 ASP.NET Core 预览版本 Azure App Service](xref:host-and-deploy/azure-apps/index#deploy-aspnet-core-preview-release-to-azure-app-service) 以解决此问题。

## <a name="configure-the-app-in-azure-app-service"></a>在 Azure App Service 中配置应用

> [!NOTE]
> *本部分仅适用于不使用 Azure 服务的应用 SignalR 。*
>
> 如果应用使用 Azure SignalR 服务，应用服务不需要配置应用程序请求路由 (ARR) 相关性和 Web 套接字。 客户端将其 Web 套接字连接到 Azure SignalR 服务，而不是直接连接到应用程序。

对于不使用 Azure 服务托管的应用 SignalR ，请启用：

* [ARR 相关性] (https://azure.github.io/AppService/2016/05/16/Disable-Session-affinity- cookie (ARR cookie) # B0 l) 将来自用户的请求路由回同一应用服务实例。 默认设置为 **"打开** "。
* 允许 Web 套接字传输正常工作的[Web 套接字](xref:fundamentals/websockets)。 默认设置为 " **关闭** "。

1. 在 Azure 门户中，导航到 **应用服务** 中的 web 应用。
1. 打开 **配置**  >  **常规设置** 。
1. 将 " **Web 套接字** " 设置为 **"开"** 。
1. 验证 **ARR 相关性** 是否设置为 **On** 。

## <a name="app-service-plan-limits"></a>应用服务计划限制

基于所选的应用服务计划，Web 套接字和其他传输受到限制。 有关详细信息，请参阅 azure [订阅和服务限制、配额和约束](/azure/azure-subscription-service-limits#app-service-limits)一文中的 *azure 云服务限制* 和 *应用服务限制* 部分。

## <a name="additional-resources"></a>其他资源

* [什么是 Azure SignalR 服务？](/azure/azure-signalr/signalr-overview)
* <xref:signalr/introduction>
* <xref:host-and-deploy/index>
* <xref:tutorials/publish-to-azure-webapp-using-vs>
* [使用命令行工具将 ASP.NET Core 应用发布到 Azure](/azure/app-service/app-service-web-get-started-dotnet)
* [在 Azure 上托管和部署 ASP.NET Core 预览应用](xref:host-and-deploy/azure-apps/index#deploy-aspnet-core-preview-release-to-azure-app-service)
