---
title: ASP.NET Core 的身份验证示例
author: rick-anderson
description: 提供指向 ASP.NET Core 存储库中的身份验证示例的链接。
ms.author: riande
ms.date: 01/31/2019
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
uid: security/authentication/samples
ms.openlocfilehash: 4153a443748dbff40be19e25fc1c719ee4e39609
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060334"
---
# <a name="authentication-samples-for-aspnet-core"></a>ASP.NET Core 的身份验证示例

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

[ASP.NET Core 存储库](https://github.com/dotnet/AspNetCore)包含 *AspNetCore/src/Security/samples* 文件夹中的以下身份验证示例：

* [声明转换](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/ClaimsTransformation)
* [Cookie 验证](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/Cookies)
* [自定义策略提供程序-IAuthorizationPolicyProvider](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/CustomPolicyProvider)
* [动态身份验证方案和选项](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/DynamicSchemes)
* [外部声明](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/Identity.ExternalClaims)
* [cookie基于请求选择和其他身份验证方案](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/PathSchemeSelection)
* [限制对静态文件的访问](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/StaticFilesAuth)

## <a name="run-the-samples"></a>运行示例

* 选择 [分支](https://github.com/dotnet/AspNetCore)。 例如： `release/3.1`
* 克隆或下载 [ASP.NET Core 存储库](https://github.com/dotnet/AspNetCore)。
* 验证是否已安装与 ASP.NET Core 存储库的克隆相匹配的 [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core) 版本。
* 导航到 *AspNetCore/src/Security/samples* 中的示例，并使用运行该示例 `dotnet run` 。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[ASP.NET Core 存储库](https://github.com/dotnet/AspNetCore)包含 *AspNetCore/src/Security/samples* 文件夹中的以下身份验证示例：

* [声明转换](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/ClaimsTransformation)
* [Cookie 验证](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/Cookies)
* [自定义策略提供程序-IAuthorizationPolicyProvider](https://github.com/dotnet/AspNetCore/tree/2.1.3/src/Security/samples/CustomPolicyProvider)
* [动态身份验证方案和选项](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/DynamicSchemes)
* [外部声明](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/Identity.ExternalClaims)
* [cookie基于请求选择和其他身份验证方案](https://github.com/dotnet/AspNetCore/tree/release/2.1/src/Security/samples/PathSchemeSelection)
* [限制对静态文件的访问](https://github.com/dotnet/AspNetCore/tree/2.1.3/src/Security/samples/StaticFilesAuth)

## <a name="run-the-samples"></a>运行示例

* 选择 [分支](https://github.com/dotnet/AspNetCore)。 例如： `release/2.1`
* 克隆或下载 [ASP.NET Core 存储库](https://github.com/dotnet/AspNetCore)。
* 验证是否已安装与 ASP.NET Core 存储库的克隆相匹配的 [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core) 版本。
* 导航到 *AspNetCore/src/Security/samples* 中的示例，并使用运行该示例 `dotnet run` 。

::: moniker-end
