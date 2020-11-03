---
title: 从 ClaimsPrincipal 迁移
author: mjrousos
description: 了解如何从 ClaimsPrincipal 迁移到，以检索当前经过身份验证的用户的标识和 ASP.NET Core 中的声明。
ms.author: scaddie
ms.custom: mvc
ms.date: 03/26/2019
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
uid: migration/claimsprincipal-current
ms.openlocfilehash: 3aa0adb299789efbb071cdb934d43832a84cf540
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059762"
---
# <a name="migrate-from-claimsprincipalcurrent"></a>从 ClaimsPrincipal 迁移

在 ASP.NET 4.x 项目中，通常使用 [ClaimsPrincipal](/dotnet/api/system.security.claims.claimsprincipal.current) 来检索当前经过身份验证的用户的标识和声明。 在 ASP.NET Core 中，不再设置此属性。 根据需要更新的代码，可以通过不同的方式获取当前经过身份验证的用户的标识。

## <a name="context-specific-state-instead-of-static-state"></a>上下文特定的状态而不是静态状态

使用 ASP.NET Core 时， `ClaimsPrincipal.Current` 不会设置和的值 `Thread.CurrentPrincipal` 。 这些属性都表示静态状态，这 ASP.NET Core 通常会避免这种情况。 相反，ASP.NET Core 使用 [依赖关系注入](xref:fundamentals/dependency-injection) (DI) 来提供当前用户的标识等依赖项。 从 DI 获取当前用户的标识也更易于测试，因为可以轻松注入测试标识。

## <a name="retrieve-the-current-user-in-an-aspnet-core-app"></a>在 ASP.NET Core 应用程序中检索当前用户

有几个选项可用于检索 ASP.NET Core 的当前已验证身份的用户的 `ClaimsPrincipal` 位置 `ClaimsPrincipal.Current` ：

* **ControllerBase** 。 MVC 控制器可以使用 [用户的用户](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.user) 属性来访问当前已经过身份验证的用户。
* **Httpcontext.current** 。 有权访问当前 `HttpContext` (中间件的组件，例如) 可以从 HttpContext 获取当前用户 `ClaimsPrincipal` 的[HttpContext.User](/dotnet/api/microsoft.aspnetcore.http.httpcontext.user)。
* **从调用方传入** 。 `HttpContext`通常从控制器或中间件组件调用不具有当前访问权限的库，并且可以将当前用户的标识作为参数传递。
* **IHttpContextAccessor** 。 正在迁移到 ASP.NET Core 的项目可能太大，无法轻松地将当前用户的标识传递到所有必要的位置。 在这种情况下，可以使用 [IHttpContextAccessor](/dotnet/api/microsoft.aspnetcore.http.ihttpcontextaccessor) 作为一种解决方法。 `IHttpContextAccessor` 如果) 存在，则能够访问当前 `HttpContext` (。 如果正在使用 DI，请参阅 <xref:fundamentals/httpcontext> 。 一种短期解决方案，用于在代码中获取当前用户的标识，但尚未更新为使用 ASP.NET Core 的 DI 驱动的体系结构：

  * `IHttpContextAccessor`在中调用[AddHttpContextAccessor](https://github.com/aspnet/Hosting/issues/793) ，使其在 DI 容器中可用 `Startup.ConfigureServices` 。
  * `IHttpContextAccessor`在启动过程中获取实例，并将其存储在静态变量中。 此实例可用于以前从静态属性中检索当前用户的代码。
  * 使用检索当前用户的 `ClaimsPrincipal` `HttpContextAccessor.HttpContext?.User` 。 如果在 HTTP 请求的上下文之外使用此代码， `HttpContext` 则为 null。

最后一种方法是使用 `IHttpContextAccessor` 存储在静态变量中的实例，而不是首选向静态依赖项注入的依赖项的 ASP.NET Core 原则。 计划最终 `IHttpContextAccessor` 从 DI 检索实例。 但在迁移使用的大型现有 ASP.NET 应用程序时，静态帮助器可能是一个有用的桥 `ClaimsPrincipal.Current` 。
