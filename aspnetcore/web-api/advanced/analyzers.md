---
title: 使用 Web API 分析器
author: pranavkm
description: 了解有关 ASP.NET Core MVC Web API 分析器包的信息。
monikerRange: '>= aspnetcore-2.2'
ms.author: prkrishn
ms.custom: mvc
ms.date: 09/05/2019
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
uid: web-api/advanced/analyzers
ms.openlocfilehash: cf0415e7d72e21a48db8bbeb4540f05e0b0a4198
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93057916"
---
# <a name="use-web-api-analyzers"></a>使用 Web API 分析器

ASP.NET Core 2.2 和更高版本提供了用于 Web API 项目的 MVC 分析器包。 分析器使用带有 <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> 批注的控制器，同时构建 [Web API 约定](xref:web-api/advanced/conventions)。

分析器包会通知你执行以下操作的任何控制器操作：

* 返回未声明的状态代码。
* 返回未声明的成功结果。
* 记录不返回的状态代码。
* 包含显式模型验证检查。

::: moniker range=">= aspnetcore-3.0"

## <a name="reference-the-analyzer-package"></a>引用分析器包

在 ASP.NET Core 3.0 或更高版本中，分析器随附在 .NET Core SDK 中。 若要在项目中启用分析器，请在项目文件中包含 `IncludeOpenAPIAnalyzers` 属性：

```xml
<PropertyGroup>
 <IncludeOpenAPIAnalyzers>true</IncludeOpenAPIAnalyzers>
</PropertyGroup>
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

## <a name="package-installation"></a>包安装

通过以下方法之一安装 [Microsoft.AspNetCore.Mvc.Api.Analyzers](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Api.Analyzers) NuGet 包：

### <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

从“程序包管理器控制台”窗口：
  * 请参阅 **查看** > **其他 Windows** > **程序包管理器控制台** 。
  * 导航到 ApiConventions.csproj 文件所在的目录  。
  * 执行以下命令：

    ```powershell
    Install-Package Microsoft.AspNetCore.Mvc.Api.Analyzers
    ```

### <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 右键单击 *Packages* **Solution Pad** > **添加包 ...** "中的" 包 "文件夹。
* 将 " **添加包** " 窗口的 " **源** " 下拉箭头设置为 "nuget.org"。
* 在搜索框中输入“Microsoft.AspNetCore.Mvc.Api.Analyzers”。
* 从结果窗格中选择“Microsoft.AspNetCore.Mvc.Api.Analyzers”包，然后单击“添加包”  。

### <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

从“集成终端”中运行以下命令：

```dotnetcli
dotnet add ApiConventions.csproj package Microsoft.AspNetCore.Mvc.Api.Analyzers
```

### <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

运行下面的命令：

```dotnetcli
dotnet add ApiConventions.csproj package Microsoft.AspNetCore.Mvc.Api.Analyzers
```

---

::: moniker-end

## <a name="analyzers-for-web-api-conventions"></a>Web API 约定的分析器

OpenAPI 文档包含操作可能返回的状态代码和响应类型。 在 ASP.NET Core MVC 中，<xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute> 和 <xref:Microsoft.AspNetCore.Mvc.ProducesAttribute> 等属性用于记录操作。 <xref:tutorials/web-api-help-pages-using-swagger> 进一步介绍有关记录 Web API 的详细信息。

包中的其中一个分析器检查使用 <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> 进行批注的控制器，并标识不完全记录其响应的操作。 请考虑以下示例：

[!code-csharp[](conventions/sample/Controllers/ContactsController.cs?name=missing404docs&highlight=10)]

上述操作记录了 HTTP 200 成功返回类型，但未记录 HTTP 404 失败状态代码。 分析器将 HTTP 404 状态代码的缺失文档报告为警告。 提供了修复此问题的选项。

![报告警告的分析器](conventions/_static/Analyzer.gif)

## <a name="additional-resources"></a>其他资源

* <xref:web-api/advanced/conventions>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:web-api/index>
