---
title: 用于 ASP.NET Core Blazor 的工具
author: guardrex
description: 了解可用于构建 Blazor 应用程序的工具。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/28/2020
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
uid: blazor/tooling
zone_pivot_groups: operating-systems
ms.openlocfilehash: d1626fe753782d524bf75c398c11235c3110633a
ms.sourcegitcommit: d7991068bc6b04063f4bd836fc5b9591d614d448
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91762147"
---
# <a name="tooling-for-aspnet-core-no-locblazor"></a>用于 ASP.NET Core Blazor 的工具

作者：[Daniel Roth](https://github.com/danroth27) 和 [Luke Latham](https://github.com/guardrex)

::: zone pivot="windows"

1. 安装带有“ASP.NET 和 Web 开发”工作负载的最新版本 [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)。

1. 创建新项目。

1. 选择“Blazor 应用”。 选择“下一步”。

1. 在“项目名称”字段提供项目名称，或接受默认项目名称。 确认“位置”条目正确无误或为项目提供位置。 选择“创建”。

1. 若要获得 Blazor WebAssembly 体验，请选择“Blazor WebAssembly 应用”模板。 若要获得 Blazor Server 体验，请选择“Blazor Server 应用”模板。 选择“创建”。

   有关 Blazor WebAssembly 和 Blazor Server 这两个 Blazor 托管模型的信息，请参阅 <xref:blazor/hosting-models>。 

1. 按 Ctrl+F5 运行应用<kbd></kbd><kbd></kbd>。

有关信任 ASP.NET Core HTTPS 开发证书的详细信息，请参阅 <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos>。

::: zone-end

::: zone pivot="linux"

1. 安装最新版本的 [.NET Core SDK](https://dotnet.microsoft.com/download)。 如果以前安装了该 SDK，可以通过在命令行界面中执行以下命令来确定已安装的版本：

   ```dotnetcli
   dotnet --version
   ```

1. 安装最新版本的 [Visual Studio Code](https://code.visualstudio.com)。

1. 安装最新的 [C# for Visual Studio Code 扩展](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)。

1. 若要获得 Blazor WebAssembly 体验，请在命令行界面中执行以下命令：

   ```dotnetcli
   dotnet new blazorwasm -o WebApplication1
   ```

   若要获得 Blazor Server 体验，请在命令行界面中执行以下命令：

   ```dotnetcli
   dotnet new blazorserver -o WebApplication1
   ```

   有关 Blazor WebAssembly 和 Blazor Server 这两个 Blazor 托管模型的信息，请参阅 <xref:blazor/hosting-models>。 

1. 在 Visual Studio Code 中打开 `WebApplication1` 文件夹，

1. IDE 要求添加资产以用于生成和调试项目。 选择 **“是”** 。

1. 按 Ctrl+F5 运行应用<kbd></kbd><kbd></kbd>。

## <a name="trust-a-development-certificate"></a>信任开发证书

Linux 上没有用于信任证书的集中途径。 通常采用以下方法之一：

* 在浏览器的排除列表中排除应用的 URL。
* 信任 `localhost` 的所有自签名证书。
* 将证书添加到浏览器的受信任证书列表。

有关详细信息，请参阅浏览器制造商和 Linux 发行版提供的指南。

::: zone-end

::: zone pivot="macos"

1. 安装 [Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/)。

1. 选择“文件” > “新建解决方案”或从“启动窗口”创建“新项目”   。

1. 在边栏中，选择“Web 和控制台” > “应用”。 

   若要获得 Blazor WebAssembly 体验，请选择“Blazor WebAssembly 应用”模板。 若要获得 Blazor Server 体验，请选择“Blazor Server 应用”模板。 选择“下一步”。

   有关 Blazor WebAssembly 和 Blazor Server 这两个 Blazor 托管模型的信息，请参阅 <xref:blazor/hosting-models>。 

1. 确认已将“身份验证”设置为“无身份验证”。 选择“下一步”。

1. 在“项目名称”字段中，将应用命名为 `WebApplication1`。 选择“创建”。

1. 选择“运行” > “启动而不调试”以不使用调试程序运行应用 。 使用“运行” > “开始调试”运行应用，或者使用“运行 (&#9654;)”按钮通过调试程序运行应用。 

如果出现信任开发证书的提示，请信任证书并继续操作。 信任证书需要使用用户密码和密钥链密码。 有关信任 ASP.NET Core HTTPS 开发证书的详细信息，请参阅 <xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos>。

::: zone-end
