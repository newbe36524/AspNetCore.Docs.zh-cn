---
title: 通过 ASP.NET Core 和 Azure 实现 DevOps
author: CamSoper
description: 提供有关为托管在 Azure 中的 ASP.NET Core 应用构建 DevOps 管道的端到端指导的指南。
ms.author: casoper
ms.date: 08/07/2018
ms.custom: devx-track-csharp, mvc, seodec18
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
uid: azure/devops/index
ms.openlocfilehash: 10a6bae73743d3063ad2e1ab49fc418ad1d63756
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052287"
---
# <a name="devops-with-aspnet-core-and-azure"></a>通过 ASP.NET Core 和 Azure 实现 DevOps

[![封面图像](./media/cover-large.png)](https://aka.ms/devopsbook)

作者：[Cam Soper](https://twitter.com/camsoper)[Scott Addie](https://twitter.com/scottaddie)

本指南以[可下载 PDF 电子书](https://aka.ms/devopsbook)的形式提供。

## <a name="welcome"></a>欢迎使用 

欢迎使用适用于 .NET 的 Azure 开发生命周期指南！ 本指南介绍使用 .NET 工具和进程围绕 Azure 构建开发生命周期的基本概念。 读完本指南后，可以获得成熟 DevOps 工具链的优势。

## <a name="who-this-guide-is-for"></a>本指南适用对象

读者应是有经验的 ASP.NET Core 开发人员（200-300 级别）。 无须了解有关 Azure 的任何内容，因为本指南中将进行介绍。 本指南对于更侧重于操作（而不是开发）的 DevOps 工程师可能也很有用。

本指南面向 Windows 开发人员。 但是，.NET Core 完全支持 Linux 和 macOS。 若要针对 Linux/macOS 调整本指南，请注意有关 Linux/macOS 差异的标注。

## <a name="what-this-guide-doesnt-cover"></a>本指南未涵盖的内容

本指南专注于 .NET 开发人员的端到端持续部署体验。 这不是介绍 Azure 中所有内容的全面指南，并不特别着重于适用于 Azure 服务的 .NET API。 介绍的重点在于持续集成、部署、监视和调试。 本指南结尾附近提供了后续步骤建议。 建议包括对 ASP.NET Core 开发人员有用的 Azure 平台服务。

## <a name="whats-in-this-guide"></a>本指南内容

### <a name="tools-and-downloads"></a>[工具和下载](xref:azure/devops/tools-and-downloads)

了解在何处获取本指南中使用的工具。

### <a name="deploy-to-app-service"></a>[部署到应用服务](xref:azure/devops/deploy-to-app-service)

了解将 ASP.NET Core 应用部署到 Azure 应用服务的各种方法。

### <a name="continuous-integration-and-deployment"></a>[持续集成和部署](xref:azure/devops/cicd)

通过 GitHub、Azure DevOps Services 和 Azure，为 ASP.NET Core 应用生成端到端持续集成和部署解决方案。

### <a name="monitor-and-debug"></a>[监视和调试](xref:azure/devops/monitor)

使用 Azure 工具监视、故障排除和优化应用程序。

### <a name="next-steps"></a>[后续步骤](xref:azure/devops/next-steps)

ASP.NET Core 开发人员学习 Azure 的其他学习路径。

## <a name="additional-introductory-reading"></a>其他介绍性读物

如果这是你首次接触云计算，这些文章介绍了基础知识。

* [什么是云计算？](https://azure.microsoft.com/overview/what-is-cloud-computing/)
* [云计算示例](https://azure.microsoft.com/overview/examples-of-cloud-computing/)
* [什么是 IaaS？](https://azure.microsoft.com/overview/what-is-iaas/)
* [什么是 PaaS？](https://azure.microsoft.com/overview/what-is-paas/)
