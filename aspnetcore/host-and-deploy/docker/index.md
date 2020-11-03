---
title: 在 Docker 容器中托管 ASP.NET Core
author: rick-anderson
description: 了解指向如何在 Docker 容器中托管 ASP.NET Core 应用的相关资源的链接。
ms.author: riande
ms.custom: mvc
ms.date: 01/08/2018
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
uid: host-and-deploy/docker/index
ms.openlocfilehash: 6b4b011314be2481e6e71d7782fff6ee99cedb9a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059775"
---
# <a name="host-aspnet-core-in-docker-containers"></a>在 Docker 容器中托管 ASP.NET Core

下面的文章可用于了解如何在 Docker 中托管 ASP.NET Core 应用：

[容器和 Docker 简介](/dotnet/standard/microservices-architecture/container-docker-introduction/index)  
容器化是软件开发的一种方法，通过该方法可将应用程序或服务、其依赖项及其配置一起打包为容器映像。了解相关内容。 可对该映像进行测试，然后将其部署到主机。

[什么是 Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-defined)  
了解如何将 Docker 作为一种开源项目，用于将应用自动部署为可在云或本地运行的便携式独立容器。

[Docker 术语](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-terminology)  
了解 Docker 技术的术语和定义。

[Docker 容器、映像和注册表](/dotnet/standard/microservices-architecture/container-docker-introduction/docker-containers-images-registries)  
了解如何将 Docker 容器映像存储在映像注册表中，以实现跨环境的一致部署。

<xref:host-and-deploy/docker/building-net-docker-images> 了解如何生成和 Docker 化 ASP.NET Core 应用。 了解由 Microsoft 维护的 Docker 映像并检查用例。

[Visual Studio 容器工具](xref:host-and-deploy/docker/visual-studio-tools-for-docker)  
了解 Visual Studio 如何支持在用于 Windows 的 Docker 上生成、调试和运行面向 .NET Framework 或 .NET Core 的 ASP.NET Core 应用。 Windows 和 Linux 容器均受支持。

[发布到 Azure 容器注册表](/azure/vs-azure-tools-docker-hosting-web-apps-in-docker)  
了解如何通过 Visual Studio 容器工具扩展使用 PowerShell 将 ASP.NET Core 应用部署到 Azure 上的 Docker 主机。

[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)  
对于托管在代理服务器和负载均衡器后方的应用，可能需要附加配置。 通过代理传递的请求通常会遮盖初始请求相关信息，例如方案和客户端 IP。 可能必须将请求相关的一些信息手动转发给应用。

[使用 Docker 和小型容器的 GC](xref:performance/memory#sc) 讨论了包含小型容器的 GC 选择。