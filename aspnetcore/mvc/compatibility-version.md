---
title: ASP.NET Core MVC 的兼容性版本
author: rick-anderson
description: 了解 ASP.NET Core 中的 Startup 类如何配置服务和应用的请求管道。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 9/25/2019
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
uid: mvc/compatibility-version
ms.openlocfilehash: b7d14aa0371fcc01d568cf1e7b9908c4a8d984df
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722827"
---
# <a name="compatibility-version-for-aspnet-core-mvc"></a>ASP.NET Core MVC 的兼容性版本

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> 方法为 ASP.NET Core 3.0 应用的无操作方法。 也就是说，使用 <xref:Microsoft.AspNetCore.Mvc.CompatibilityVersion> 的任何值调用 `SetCompatibilityVersion` 都不会影响应用程序。

* ASP.NET Core 的下一个次版本可能提供新的 `CompatibilityVersion` 值。
* `CompatibilityVersion` 值 `Version_2_0` 到 `Version_2_2` 被标记为 `[Obsolete(...)]`。
* 请参阅[防伪、CORS、诊断、Mvc 和路由中的重大 API 变更](https://github.com/aspnet/Announcements/issues/387)。 此列表包含兼容性开关的重大变更。

若要查看 `SetCompatibilityVersion` 如何与 ASP.NET Core 2.x 应用协同工作，请选择[这篇文章的 ASP.NET Core 2.2 版本](?view=aspnetcore-2.2)。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> 方法允许 ASP.NET Core 2.x 应用选择加入或退出 ASP.NET Core MVC 2.1 或 2.2 中引入的潜在重大行为变更。 这些潜在的中断行为变更通常取决于 MVC 子系统的行为方式以及运行时调用“代码”的方式****。 通过选择加入，你将获取最新的行为以及 ASP.NET Core 的长期行为。

以下代码将兼容模式设置为 ASP.NET Core 2.2：

[!code-csharp[Main](compatibility-version/samples/2.x/CompatibilityVersionSample/Startup.cs?name=snippet1)]

建议使用最新版本 (`CompatibilityVersion.Latest`) 来测试应用。 我们预计大多数应用不会使用最新版本进行中断行为变更。

调用 `SetCompatibilityVersion(CompatibilityVersion.Version_2_0)` 的应用会被阻止进行 ASP.NET Core 2.1/2.2 MVC 版本中引入的潜在重大行为变更。 该阻止操作：

* 不适用于所有 2.1 和更高版本的更改，它的目标是潜在地中断 MVC 子系统中的 ASP.NET Core 运行时行为变更。
* 不扩展到 ASP.NET Core 3.0。

未**** 调用 `SetCompatibilityVersion` 的 ASP.NET Core 2.1 和 2.2 版本的应用的默认兼容性是 2.0 兼容性。 即，未调用 `SetCompatibilityVersion` 与调用 `SetCompatibilityVersion(CompatibilityVersion.Version_2_0)` 相同。

以下代码将兼容模式设置为 ASP.NET Core 2.2（以下行为除外）：

* <xref:Microsoft.AspNetCore.Mvc.MvcOptions.AllowCombiningAuthorizeFilters>
* <xref:Microsoft.AspNetCore.Mvc.MvcOptions.InputFormatterExceptionPolicy>

[!code-csharp[Main](compatibility-version/samples/2.x/CompatibilityVersionSample/Startup2.cs?name=snippet1)]

对于遇到中断行为变更的应用，请使用适当的兼容性开关：

* 允许使用最新版本并选择退出特定的中断行为变更。
* 请用些时间更新应用，以便其适用于最新更改。

<xref:Microsoft.AspNetCore.Mvc.MvcOptions> 文件很好地解释了更改的内容以及为什么更改对大多数用户来说是一种改进。

对于 ASP.NET Core 3.0，已删除兼容性开关支持的旧行为。 我们认为这些积极的变化几乎使所有用户受益。 通过在 2.1 和 2.2 中引入这些更改，大多数应用都可以受益，而其他应用则有时间更新。
::: moniker-end