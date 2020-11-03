---
title: 将 ASP.NET Core 应用发布到 IIS
author: rick-anderson
description: 了解如何在 IIS 服务器上托管 ASP.NET Core 应用。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/03/2019
no-loc:
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
uid: tutorials/publish-to-iis
ms.openlocfilehash: 40c47da472257862414ba33be582eb19d3f0b29c
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91754549"
---
# <a name="publish-an-aspnet-core-app-to-iis"></a>将 ASP.NET Core 应用发布到 IIS

本教程介绍如何在 IIS 服务器上托管 ASP.NET Core 应用。

本教程涵盖以下主题：

> [!div class="checklist"]
> * 在 Windows Server 上安装.NET Core Hosting Bundle。
> * 在 IIS 管理器中创建 IIS 站点。
> * 部署 ASP.NET Core 应用。

## <a name="prerequisites"></a>先决条件

* [.NET Core SDK](/dotnet/core/sdk) 安装在开发计算机上。
* Windows Server 配置了“Web 服务器 (IIS)”服务器角色。 如果服务器未配置为托管具有 IIS 的网站，请按照 <xref:host-and-deploy/iis/index#iis-configuration> 文章中“IIS 配置”部分的指南操作，然后返回本教程。

> [!WARNING]
> IIS 配置和网站安全涉及到本教程未介绍的概念。 在 IIS 上托管生产应用之前，请先参阅 [Microsoft IIS 文档](https://www.iis.net/)中的 IIS 指南和[有关使用 IIS 进行托管的 ASP.NET Core 文章](xref:host-and-deploy/iis/index)。
>
> 本教程未介绍的 IIS 托管的重要方案包括：
>
> * [为 ASP.NET Core 数据保护创建注册表配置单元](xref:host-and-deploy/iis/index#data-protection)
> * [配置应用池的访问控制列表 (ACL)](xref:host-and-deploy/iis/index#application-pool-identity)
> * 为了重点介绍 IIS 部署概念，本教程部署了一个没有在 IIS 中配置 HTTPS 安全性的应用。 有关托管为 HTTPS 协议启用的应用的详细信息，请参阅本文[其他资源](#additional-resources)部分中的安全主题。 有关托管 ASP.NET 核心应用的更多指南，请参阅 <xref:host-and-deploy/iis/index> 文章。

## <a name="install-the-net-core-hosting-bundle"></a>安装 .NET Core 托管捆绑包

在 IIS 服务器上安装 .NET Core 托管捆绑包。 捆绑包可安装 .NET Core 运行时、.NET Core 库和 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 该模块允许 ASP.NET Core 应用在 IIS 后面运行。

使用以下链接下载安装程序：

[当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

1. 在 IIS 服务器上运行安装程序。

1. 重启服务器或在命令行界面中执行 `net stop was /y`，后跟 `net start w3svc`。

## <a name="create-the-iis-site"></a>创建 IIS 站点

1. 在 IIS 服务器上，创建一个文件夹以包含应用已发布的文件夹和文件。 在接下来的步骤中，文件夹路径作为应用程序的物理路径提供给 IIS。 有关应用程序部署文件夹和文件布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。

1. 在 IIS 管理器中，打开“连接”面板中的服务器节点。 右键单击“站点”文件夹。 选择上下文菜单中的“添加网站”。

1. 提供网站名称，并将“物理路径”设置为所创建应用的部署文件夹 。 提供“绑定”配置，并通过选择“确定”创建网站 。

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定可能会为应用带来安全漏洞。 此行为同时适用于强通配符和弱通配符。 使用显式主机名而不是通配符。 如果可控制整个父域（区别于易受攻击的 `*.com`），则子域通配符绑定（例如，`*.mysub.com`）不具有此安全风险。 有关详细信息，请参阅 [rfc7230 第 5.4 条](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 确认进程模型标识拥有适当的权限。

   如果将应用池的默认标识（“进程模型” > “Identity”）从 `ApplicationPoolIdentity` 更改为另一标识，请确保新标识拥有对应用文件夹、数据库和其他所需资源的必需访问权限 。 例如，应用池需要对文件夹的读取和写入权限，以便应用在其中读取和写入文件。

## <a name="create-an-aspnet-core-no-locrazor-pages-app"></a>创建 ASP.NET Core Razor Pages 应用

按照 <xref:getting-started> 教程创建 Razor Pages 应用。

## <a name="publish-and-deploy-the-app"></a>发布和部署应用

发布应用意味着生成可由服务器托管的编译应用。 部署应用意味着将发布的应用移动到托管系统。 发布步骤由 [.NET Core SDK ](/dotnet/core/sdk) 处理，而部署步骤可以通过各种方法处理。 本教程采用“文件夹”部署方法，即：
 
* 将应用发布到一个文件夹。
* 文件夹的内容将移动到 IIS 站点的文件夹（IIS 管理器中站点的物理路径）。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 在“解决方案资源管理器”中右键单击该项目，然后选择“发布”。
1. 在“选择发布目标”对话框中，选择“文件夹”发布选项 。
1. 设置“文件夹或文件共享”路径。
   * 如果为开发计算机上可用作网络共享的 IIS 站点创建了一个文件夹，请提供该共享的路径。 当前用户必须具有写入权限才能发布到共享。
   * 如果无法直接部署到 IIS 服务器上的 IIS 站点文件夹，请发布到可移动介质上的文件夹，并将已发布的应用物理移动到服务器上的 IIS 站点文件夹，该文件夹是该站点在 IIS 管理器中的物理路径。 将 `bin/Release/{TARGET FRAMEWORK}/publish` 文件夹的内容移动到服务器上的 IIS 站点文件夹，该文件夹是该站点在 IIS 管理器中的物理路径。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

1. 在命令 shell 中，使用 [dotnet publish](/dotnet/core/tools/dotnet-publish) 命令在发布配置中发布应用：

   ```dotnetcli
   dotnet publish --configuration Release
   ```

1. 将 `bin/Release/{TARGET FRAMEWORK}/publish` 文件夹的内容移动到服务器上的 IIS 站点文件夹，该文件夹是该站点在 IIS 管理器中的物理路径。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

1. 右键单击“解决方案”中的项目，然后选择“发布” > “发布到文件夹”  。
1. 设置“选择文件夹”路径。
   * 如果为开发计算机上可用作网络共享的 IIS 站点创建了一个文件夹，请提供该共享的路径。 当前用户必须具有写入权限才能发布到共享。
   * 如果无法直接部署到 IIS 服务器上的 IIS 站点文件夹，请发布到可移动介质上的文件夹，并将已发布的应用物理移动到服务器上的 IIS 站点文件夹，该文件夹是该站点在 IIS 管理器中的物理路径。 将 `bin/Release/{TARGET FRAMEWORK}/publish` 文件夹的内容移动到服务器上的 IIS 站点文件夹，该文件夹是该站点在 IIS 管理器中的物理路径。

---

## <a name="browse-the-website"></a>浏览网站

应用收到第一个请求后，可以在浏览器中访问该应用。 在站点的 IIS 管理器中创建的终结点绑定上发出对应用的请求。

## <a name="next-steps"></a>后续步骤

在本教程中，你将了解：

> [!div class="checklist"]
> * 在 Windows Server 上安装.NET Core Hosting Bundle。
> * 在 IIS 管理器中创建 IIS 站点。
> * 部署 ASP.NET Core 应用。

若要了解有关在 IIS 上托管 ASP.NET Core 应用的详细信息，请参阅 IIS 概述文章：

> [!div class="nextstepaction"]
> <xref:host-and-deploy/iis/index>

## <a name="additional-resources"></a>其他资源

### <a name="articles-in-the-aspnet-core-documentation-set"></a>ASP.NET Core 文档集中的文章

* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:test/troubleshoot-azure-iis>
* <xref:security/enforcing-ssl>

### <a name="articles-pertaining-to-aspnet-core-app-deployment"></a>有关 ASP.NET Core 应用部署的文章

* <xref:tutorials/publish-to-azure-webapp-using-vs>
* <xref:tutorials/publish-to-azure-webapp-using-vscode>
* <xref:host-and-deploy/visual-studio-publish-profiles>
* [使用 Visual Studio for Mac 将 Web 应用发布到文件夹](/visualstudio/mac/publish-folder)

### <a name="articles-on-iis-https-configuration"></a>有关 IIS HTTPS 配置的文章

* [在 IIS 管理器中配置 SSL](/iis/manage/configuring-security/configuring-ssl-in-iis-manager)
* [如何在 IIS 上设置 SSL](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis)

### <a name="articles-on-iis-and-windows-server"></a>有关 IIS 和 Windows Server 的文章

* [Microsoft IIS 官方网站](https://www.iis.net/)
* [Windows Server 技术内容库](/windows-server/windows-server)

### <a name="deployment-resources-for-iis-administrators"></a>面向 IIS 管理员的部署资源

* [IIS 文档](/iis)
* [IIS 中 IIS 管理器入门](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8)
* [.NET Core 应用程序部署](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

