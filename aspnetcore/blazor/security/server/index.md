---
title: 保护 ASP.NET Core Blazor Server应用
author: guardrex
description: 了解如何将 Blazor Server应用作为 ASP.NET Core 应用来保护。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/06/2020
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
uid: blazor/security/server/index
ms.openlocfilehash: 108fb3a8a24295cad43fd8c83303abd95a7ecd33
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93055472"
---
# <a name="secure-aspnet-core-no-locblazor-server-apps"></a>保护 ASP.NET Core Blazor Server应用

作者：[Luke Latham](https://github.com/guardrex)

Blazor Server应用的安全配置方式与 ASP.NET Core 应用相同。 有关详细信息，请参阅 <xref:security/index> 下的文章。 此“概述”下的主题特别适用于 Blazor Server。

## <a name="no-locblazor-server-project-template"></a>Blazor Server项目模板

创建项目后，可配置 Blazor Server项目模板来进行身份验证。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

请按照 <xref:blazor/tooling> 中的 Visual Studio 指南操作，新建具有身份验证机制的 Blazor Server项目。

在“新建 ASP.NET Core Web 应用”对话框中选择“Blazor Server应用”模板后，选择“身份验证”下的“更改”。

此时将打开一个对话框，为其他 ASP.NET Core 项目提供一组相同的身份验证机制：

* **无身份验证**
* **个人用户帐户** ：可存储用户帐户：
  * 在使用 ASP.NET Core 的 [Identity](xref:security/authentication/identity) 系统的应用中存储。
  * 使用 [Azure AD B2C](xref:security/authentication/azure-ad-b2c) 存储。
* **工作或学校帐户**
* **Windows 身份验证**

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

请按照 <xref:blazor/tooling> 中的 Visual Studio Code 指南操作，新建具有身份验证机制的 Blazor Server项目：

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

下表中显示了可能的身份验证值 (`{AUTHENTICATION}`)。

| 身份验证机制 | 描述 |
| ------------------------ | ----------- |
| `None`（默认值）         | 无身份验证 |
| `Individual`             | 使用 ASP.NET Core Identity 将用户存储在应用中 |
| `IndividualB2C`          | 用户存储在 [Azure AD B2C](xref:security/authentication/azure-ad-b2c) 中 |
| `SingleOrg`              | 对一个租户进行组织身份验证 |
| `MultiOrg`               | 对多个租户进行组织身份验证 |
| `Windows`                | Windows 身份验证 |

使用 `-o|--output` 选项，该命令使用为 `{APP NAME}` 占位符提供的值来执行以下操作：

* 为项目创建文件夹。
* 为该项目命名。

有关详细信息，请参阅 .NET Core 指南中的 [`dotnet new`](/dotnet/core/tools/dotnet-new) 命令。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

1. 请按照 <xref:blazor/tooling> 中的 Visual Studio for Mac 指南进行操作。

1. 在“配置新的 Blazor Server应用”步骤中，从“身份验证”下拉列表中选择“个人身份验证(应用内)”。

1. 此应用是使用 ASP.NET Core Identity 为应用中存储的个人用户创建的。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli/)

在命令行界面中使用以下命令，新建具有身份验证机制的 Blazor Server项目：

```dotnetcli
dotnet new blazorserver -o {APP NAME} -au {AUTHENTICATION}
```

下表中显示了可能的身份验证值 (`{AUTHENTICATION}`)。

| 身份验证机制 | 描述 |
| ------------------------ | ----------- |
| `None`（默认值）         | 无身份验证 |
| `Individual`             | 使用 ASP.NET Core Identity 将用户存储在应用中 |
| `IndividualB2C`          | 用户存储在 [Azure AD B2C](xref:security/authentication/azure-ad-b2c) 中 |
| `SingleOrg`              | 对一个租户进行组织身份验证 |
| `MultiOrg`               | 对多个租户进行组织身份验证 |
| `Windows`                | Windows 身份验证 |

使用 `-o|--output` 选项，该命令使用为 `{APP NAME}` 占位符提供的值来执行以下操作：

* 为项目创建文件夹。
* 为该项目命名。

更多相关信息：

* 请参阅 .NET Core 指南中的 [`dotnet new`](/dotnet/core/tools/dotnet-new) 命令。
* 在命令行界面中为 Blazor Server模板 (`blazorserver`) 执行 help 命令：

  ```dotnetcli
  dotnet new blazorserver --help
  ```

---

## <a name="scaffold-no-locidentity"></a>设置Identity的基架

将 Identity 架构到 Blazor Server项目中：

* [当前没有授权](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-without-existing-authorization)。
* [有授权](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-blazor-server-project-with-authorization)。

## <a name="additional-resources"></a>其他资源

* [快速入门：将 Microsoft 登录添加到 ASP.NET Core Web 应用](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp)
* [快速入门：使用 Microsoft 标识平台保护 ASP.NET Core Web API](/azure/active-directory/develop/quickstart-v2-aspnet-core-web-api)
