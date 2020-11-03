---
title: 托管和部署 ASP.NET Core
author: rick-anderson
description: 了解如何设置托管环境和部署 ASP.NET Core 应用。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: host-and-deploy/index
ms.openlocfilehash: 19e888859cea35624491a516404c57e30aa9db05
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93057214"
---
# <a name="host-and-deploy-aspnet-core"></a>托管和部署 ASP.NET Core

::: moniker range=">= aspnetcore-2.2"

一般而言，向托管环境部署 ASP.NET Core 应用需执行以下操作：

* 将已发布应用部署到托管服务器上的文件夹。
* 设置进程管理器，该管理器在收到请求时启动应用，并在应用发生故障或服务器重启后重新启动应用。
* 对于反向代理配置，将反向代理设置为将请求转发到应用。

## <a name="publish-to-a-folder"></a>发布到文件夹

[dotnet publish](/dotnet/core/tools/dotnet-publish) 命令编译应用代码，并复制在“发布”文件夹中运行应用所需的文件。 使用 Visual Studio 进行部署时，自动先执行 `dotnet publish` 步骤，再将文件复制到部署目标。

### <a name="folder-contents"></a>文件夹内容

“发布”文件夹包含一个或多个应用程序集文件、依赖项以及（可选）.NET 运行时。

.NET Core 应用可以发布为“独立式部署”，也可以发布为“依赖框架的部署”。 如果应用是独立式，“发布”文件夹中有包含 .NET 运行时的程序集文件。 如果应用依赖于框架，.NET 运行时文件就不包含在内，因为应用包含对服务器上安装的 .NET 版本的引用。 默认部署模型是依赖于框架的模型。 有关详细信息，请参阅 [.NET Core 应用程序部署](/dotnet/core/deploying/)。

除了“.exe”和“.dll”文件以外，ASP.NET Core 应用的“发布”文件夹通常包含配置文件、静态资产和 MVC 视图。 有关详细信息，请参阅 <xref:host-and-deploy/directory-structure>。

## <a name="set-up-a-process-manager"></a>设置进程管理器

ASP.NET Core 应用是一个控制台应用，在服务器启动时必须启动该应用，并且在出现故障后必须重新启动它。 若要自动启动和重新启动，需要使用进程管理器。 用于 ASP.NET Core 的最常见进程管理器是：

* Linux
  * [Nginx](xref:host-and-deploy/linux-nginx)
  * [Apache](xref:host-and-deploy/linux-apache)
* Windows
  * [IIS](xref:host-and-deploy/iis/index)
  * [Windows 服务](xref:host-and-deploy/windows-service)

## <a name="set-up-a-reverse-proxy"></a>设置反向代理

如果应用使用 [Kestrel](xref:fundamentals/servers/kestrel) 服务器，[Nginx](xref:host-and-deploy/linux-nginx)、[Apache](xref:host-and-deploy/linux-apache) 或 [IIS](xref:host-and-deploy/iis/index) 可用作反向代理服务器。 反向代理服务器接收来自 Internet 的 HTTP 请求，并将这些请求转发到 Kestrel。

无论配置是否使用反向代理服务器，都是受支持的托管配置。 有关详细信息，请参阅[何时结合使用 Kestrel 和反向代理](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)。

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

对于托管在代理服务器和负载均衡器后方的应用，可能需要附加配置。 如不提供附加配置，应用可能无法访问方案 (HTTP/HTTPS) 和发出请求的远程 IP 地址。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="use-visual-studio-and-msbuild-to-automate-deployments"></a>使用 Visual Studio 和 MSBuild 自动执行部署

除将输出从 [dotnet publish](/dotnet/core/tools/dotnet-publish) 复制到服务器外，部署通常还需要其他任务。 例如，可能需要从“发布”文件夹获取或排除额外文件。 Visual Studio 使用 [MSBuild](/visualstudio/msbuild/msbuild) 进行 Web 部署，用户可以自定义 MSBuild 以在部署期间执行其他任务。 有关详细信息，请参阅 <xref:host-and-deploy/visual-studio-publish-profiles> 和[使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/) 一书。

通过[发布 Web 功能](xref:tutorials/publish-to-azure-webapp-using-vs)或[内置 Git 支持](xref:host-and-deploy/azure-apps/azure-continuous-deployment)，可以将应用从 Visual Studio 直接部署到 Azure 应用服务。 Azure DevOps Services 支持[对 Azure 应用服务进行持续部署](/azure/devops/pipelines/targets/webapp)。 有关详细信息，请参阅[通过 ASP.NET Core 和 Azure 实现 DevOps](xref:azure/devops/index)。

## <a name="publish-to-azure"></a>发布到 Azure

有关如何使用 Visual Studio 将应用发布到 Azure 的说明，请参阅 <xref:tutorials/publish-to-azure-webapp-using-vs>。 [在 Azure 中创建 ASP.NET Core Web 应用](/azure/app-service/app-service-web-get-started-dotnet)提供了其他示例。

## <a name="publish-with-msdeploy-on-windows"></a>在 Windows 中使用 MSDeploy 发布

请参阅 <xref:host-and-deploy/visual-studio-publish-profiles>，了解如何使用 Visual Studio 发布配置文件发布应用，其中包括在 Windows 命令提示符处使用 [dotnet msbuild](/dotnet/core/tools/dotnet-msbuild) 命令。

## <a name="internet-information-services-iis"></a>Internet 信息服务 (IIS)

要使用 web.config 文件提供的配置部署到 Internet Information Services (IIS)，请参阅 <xref:host-and-deploy/iis/index> 下的文章。

## <a name="host-in-a-web-farm"></a>在 Web 场中托管

有关如何配置在 Web 场环境中托管 ASP.NET Core 应用（例如，部署应用的多个实例以实现可伸缩性）的信息，请参阅 <xref:host-and-deploy/web-farm>。

## <a name="host-on-docker"></a>在 Docker 中托管

有关详细信息，请参阅 <xref:host-and-deploy/docker/index>。

## <a name="perform-health-checks"></a>执行运行状况检查

使用运行状况检查中间件，对应用及其依赖项执行运行状况检查。 有关详细信息，请参阅 <xref:host-and-deploy/health-checks>。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* [ASP.NET 托管](https://dotnet.microsoft.com/apps/aspnet/hosting)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

一般而言，向托管环境部署 ASP.NET Core 应用需执行以下操作：

* 将已发布应用部署到托管服务器上的文件夹。
* 设置进程管理器，该管理器在收到请求时启动应用，并在应用发生故障或服务器重启后重新启动应用。
* 对于反向代理配置，将反向代理设置为将请求转发到应用。

## <a name="publish-to-a-folder"></a>发布到文件夹

[dotnet publish](/dotnet/core/tools/dotnet-publish) 命令编译应用代码，并复制在“发布”文件夹中运行应用所需的文件。 使用 Visual Studio 进行部署时，自动先执行 `dotnet publish` 步骤，再将文件复制到部署目标。

### <a name="folder-contents"></a>文件夹内容

“发布”文件夹包含一个或多个应用程序集文件、依赖项以及（可选）.NET 运行时。

.NET Core 应用可以发布为“独立式部署”，也可以发布为“依赖框架的部署”。 如果应用是独立式，“发布”文件夹中有包含 .NET 运行时的程序集文件。 如果应用依赖于框架，.NET 运行时文件就不包含在内，因为应用包含对服务器上安装的 .NET 版本的引用。 默认部署模型是依赖于框架的模型。 有关详细信息，请参阅 [.NET Core 应用程序部署](/dotnet/core/deploying/)。

除了“.exe”和“.dll”文件以外，ASP.NET Core 应用的“发布”文件夹通常包含配置文件、静态资产和 MVC 视图。 有关详细信息，请参阅 <xref:host-and-deploy/directory-structure>。

## <a name="set-up-a-process-manager"></a>设置进程管理器

ASP.NET Core 应用是一个控制台应用，在服务器启动时必须启动该应用，并且在出现故障后必须重新启动它。 若要自动启动和重新启动，需要使用进程管理器。 用于 ASP.NET Core 的最常见进程管理器是：

* Linux
  * [Nginx](xref:host-and-deploy/linux-nginx)
  * [Apache](xref:host-and-deploy/linux-apache)
* Windows
  * [IIS](xref:host-and-deploy/iis/index)
  * [Windows 服务](xref:host-and-deploy/windows-service)

## <a name="set-up-a-reverse-proxy"></a>设置反向代理

如果应用使用 [Kestrel](xref:fundamentals/servers/kestrel) 服务器，[Nginx](xref:host-and-deploy/linux-nginx)、[Apache](xref:host-and-deploy/linux-apache) 或 [IIS](xref:host-and-deploy/iis/index) 可用作反向代理服务器。 反向代理服务器接收来自 Internet 的 HTTP 请求，并将这些请求转发到 Kestrel。

无论配置是否使用反向代理服务器，都是受支持的托管配置。 有关详细信息，请参阅[何时结合使用 Kestrel 和反向代理](xref:fundamentals/servers/kestrel#when-to-use-kestrel-with-a-reverse-proxy)。

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

对于托管在代理服务器和负载均衡器后方的应用，可能需要附加配置。 如不提供附加配置，应用可能无法访问方案 (HTTP/HTTPS) 和发出请求的远程 IP 地址。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="use-visual-studio-and-msbuild-to-automate-deployments"></a>使用 Visual Studio 和 MSBuild 自动执行部署

除将输出从 [dotnet publish](/dotnet/core/tools/dotnet-publish) 复制到服务器外，部署通常还需要其他任务。 例如，可能需要从“发布”文件夹获取或排除额外文件。 Visual Studio 使用 MSBuild 进行 Web 部署，用户可以自定义 MSBuild 以在部署期间执行其他任务。 有关详细信息，请参阅 <xref:host-and-deploy/visual-studio-publish-profiles> 和[使用 MSBuild 和 Team Foundation Build](http://msbuildbook.com/) 一书。

通过[发布 Web 功能](xref:tutorials/publish-to-azure-webapp-using-vs)或[内置 Git 支持](xref:host-and-deploy/azure-apps/azure-continuous-deployment)，可以将应用从 Visual Studio 直接部署到 Azure 应用服务。 Azure DevOps Services 支持[对 Azure 应用服务进行持续部署](/azure/devops/pipelines/targets/webapp)。 有关详细信息，请参阅[通过 ASP.NET Core 和 Azure 实现 DevOps](xref:azure/devops/index)。

## <a name="publish-to-azure"></a>发布到 Azure

有关如何使用 Visual Studio 将应用发布到 Azure 的说明，请参阅 <xref:tutorials/publish-to-azure-webapp-using-vs>。 [在 Azure 中创建 ASP.NET Core Web 应用](/azure/app-service/app-service-web-get-started-dotnet)提供了其他示例。

## <a name="publish-with-msdeploy-on-windows"></a>在 Windows 中使用 MSDeploy 发布

请参阅 <xref:host-and-deploy/visual-studio-publish-profiles>，了解如何使用 Visual Studio 发布配置文件发布应用，其中包括在 Windows 命令提示符处使用 [dotnet msbuild](/dotnet/core/tools/dotnet-msbuild) 命令。

## <a name="internet-information-services-iis"></a>Internet 信息服务 (IIS)

要使用 web.config 文件提供的配置部署到 Internet Information Services (IIS)，请参阅 <xref:host-and-deploy/iis/index> 下的文章。

## <a name="host-in-a-web-farm"></a>在 Web 场中托管

有关如何配置在 Web 场环境中托管 ASP.NET Core 应用（例如，部署应用的多个实例以实现可伸缩性）的信息，请参阅 <xref:host-and-deploy/web-farm>。

## <a name="host-on-docker"></a>在 Docker 中托管

有关详细信息，请参阅 <xref:host-and-deploy/docker/index>。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* [ASP.NET 托管](https://dotnet.microsoft.com/apps/aspnet/hosting)

::: moniker-end
