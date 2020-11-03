---
title: 基于通过单独用户帐户创建的 ASP.NET Core 项目的项目
author: rick-anderson
description: 基于通过单独用户帐户创建的 ASP.NET Core 项目发现文章。
ms.author: riande
ms.date: 12/11/2019
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
uid: security/authentication/individual
ms.openlocfilehash: 656006396de120b7feae6f2e08b5dad3b5a170b5
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053340"
---
# <a name="articles-based-on-aspnet-core-projects-created-with-individual-user-accounts"></a>基于通过单独用户帐户创建的 ASP.NET Core 项目的项目

ASP.NET Core Identity 包含在 Visual Studio 中具有 "单独用户帐户" 选项的项目模板中。

.NET Core CLI 中提供了身份验证模板 `-au Individual` ：

::: moniker range=">= aspnetcore-2.1"

```dotnetcli
dotnet new mvc -au Individual
dotnet new webapp -au Individual
```

::: moniker-end

::: moniker range="= aspnetcore-2.0"

```dotnetcli
dotnet new mvc -au Individual
dotnet new razor -au Individual
```

::: moniker-end

对于 web API 身份验证，请参阅 [此 GitHub 问题](https://github.com/dotnet/AspNetCore/issues/5833) 。

<a name="no"></a>

## <a name="no-authentication"></a>无身份验证

在 .NET Core CLI 中通过选项指定了身份验证 `-au` 。 在 Visual Studio 中，" **更改身份验证** " 对话框可用于新的 web 应用程序。 Visual Studio 中新 web 应用的默认值为 " **无身份验证** "。

用无身份验证创建的项目：

* 不要包含用于登录和注销的网页和 UI。
* 不包含身份验证代码。

<a name="win"></a>

## <a name="windows-authentication"></a>Windows 身份验证

Windows 身份验证是通过选项在 .NET Core CLI 中为新 web 应用指定的 `-au Windows` 。 在 Visual Studio 中，" **更改身份验证** " 对话框提供 **Windows 身份验证** 选项。

如果选择了 "Windows 身份验证"，则会将应用程序配置为使用 [Windows 身份验证 IIS 模块](xref:host-and-deploy/iis/modules)。 Windows 身份验证适用于 Intranet 网站。

## <a name="dotnet-new-webapp-authentication-options"></a>dotnet new webapp authentication 选项

下表显示了可用于新 web 应用的身份验证选项：

| 选项 | 身份验证类型 | 有关详细信息的链接 |
 | ----------------- | ------------ | ---------- |
| None            |  无身份验证 | | 
| 个人      |  单个身份验证 | <xref:security/authentication/identity>
| IndividualB2C   |  Azure AD B2C 的云托管的单个身份验证 | [Azure AD B2C](/azure/active-directory-b2c/) |
| SingleOrg       |  对一个租户进行组织身份验证 | [Azure AD](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp) |
| MultiOrg        |  对多个租户进行组织身份验证 | [Azure AD](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp) |
| Windows         |  Windows 身份验证 | [Windows 身份验证](xref:security/authentication/windowsauth)

## <a name="visual-studio-new-webapp-authentication-options"></a>Visual Studio new webapp authentication 选项

下表显示了使用 Visual Studio 创建新的 web 应用时可用的身份验证选项：

| 选项 | 身份验证类型 | 有关详细信息的链接 |
 | ----------------- | ------------ | ---------- |
| None            |  无身份验证 | | 
| 应用中的单个用户帐户/存储用户帐户 |  单个身份验证 | <xref:security/authentication/identity> |
| 单个用户帐户/连接到云中的现有用户存储 |  Azure AD B2C 的云托管的单个身份验证 | [Azure AD B2C](/azure/active-directory-b2c/) |
| 工作或学校云/单个组织  |  对一个租户进行组织身份验证 | [Azure AD](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp) |
| 工作或学校云/多个组织 |  对多个租户进行组织身份验证 | [Azure AD](/azure/active-directory/develop/quickstart-v2-aspnet-core-webapp) |
| Windows         |  Windows 身份验证 | [Windows 身份验证](xref:security/authentication/windowsauth)

## <a name="additional-resources"></a>其他资源

以下文章演示了如何使用 ASP.NET Core 使用单个用户帐户的模板中生成的代码：

* [ASP.NET Core 中的帐户确认和密码恢复](xref:security/authentication/accconfirm)
* [使用授权保护的用户数据创建 ASP.NET Core 应用](xref:security/authorization/secure-data)
