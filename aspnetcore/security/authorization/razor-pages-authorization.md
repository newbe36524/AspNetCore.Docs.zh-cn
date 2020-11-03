---
title: Razor 页面中的授权约定 ASP.NET Core
author: rick-anderson
description: 了解如何使用用于授权用户和允许匿名用户访问页面或文件夹的约定来控制对页面的访问。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/12/2019
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
uid: security/authorization/razor-pages-authorization
ms.openlocfilehash: 69e1d639aeb55ae64cc54b1cda402ed6bcbb04ab
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060178"
---
# <a name="no-locrazor-pages-authorization-conventions-in-aspnet-core"></a>Razor 页面中的授权约定 ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

在页面应用中控制访问权限的一种方法 Razor 是在启动时使用授权约定。 这些约定允许用户授权用户，并允许匿名用户访问页面的各个页面或文件夹。 本主题中所述的约定会自动应用 [授权筛选器](xref:mvc/controllers/filters#authorization-filters) 来控制访问权限。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/razor-pages-authorization/samples)（[如何下载](xref:index#how-to-download-a-sample)）

示例应用[ cookie ASP.NET Core Identity 不使用身份验证](xref:security/authentication/cookie)。 本主题中所示的概念和示例同样适用于使用的应用 ASP.NET Core Identity 。 若要使用 ASP.NET Core Identity ，请按照中的指导进行操作 <xref:security/authentication/identity> 。

## <a name="require-authorization-to-access-a-page"></a>需要授权才能访问页面

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*> 约定将添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到页面中的指定路径：

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=3)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径，无扩展名，只包含正斜杠。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizePage 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*)：

```csharp
options.Conventions.AuthorizePage("/Contact", "AtLeast21");
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter>可应用于具有 filter 特性的页模型类 `[Authorize]` 。 有关详细信息，请参阅 [授权筛选器属性](xref:razor-pages/filter#authorize-filter-attribute)。

## <a name="require-authorization-to-access-a-folder-of-pages"></a>需要授权才能访问页的文件夹

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*> 约定将添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到文件夹中指定路径的所有页面：

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=4)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizeFolder 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*)：

```csharp
options.Conventions.AuthorizeFolder("/Private", "AtLeast21");
```

## <a name="require-authorization-to-access-an-area-page"></a>需要授权才能访问区域页

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*> 约定将添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到指定路径处的区域页：

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts");
```

页面名称是文件的路径，该文件的扩展名相对于指定区域的页根目录。 例如，文件 *区域/ Identity /Pages/Manage/Accounts.cshtml* 的页面名称为 */Manage/Accounts* 。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizeAreaPage 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*)：

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts", "AtLeast21");
```

## <a name="require-authorization-to-access-a-folder-of-areas"></a>需要授权才能访问区域文件夹

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*> 约定将添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到文件夹中指定路径的所有区域：

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage");
```

文件夹路径是文件夹相对于指定区域的页面根目录的路径。 例如， *区域/ Identity /Pages/Manage/* 下的文件的文件夹路径为 */Manage* 。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizeAreaFolder 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*)：

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage", "AtLeast21");
```

## <a name="allow-anonymous-access-to-a-page"></a>允许匿名访问页面

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToPage*> 约定将添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 到位于指定路径的页面：

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=5)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径，无扩展名，只包含正斜杠。

## <a name="allow-anonymous-access-to-a-folder-of-pages"></a>允许匿名访问页面文件夹

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToFolder*> 约定将添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 到文件夹中指定路径的所有页面：

[!code-csharp[](razor-pages-authorization/samples/3.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=6)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径。

## <a name="note-on-combining-authorized-and-anonymous-access"></a>合并授权访问和匿名访问时的注意事项

有效的方法是指定页面的文件夹需要授权，然后指定该文件夹中的页面允许匿名访问：

```csharp
// This works.
.AuthorizeFolder("/Private").AllowAnonymousToPage("/Private/Public")
```

但是，反之则无效。 不能声明用于匿名访问的页面文件夹，然后在该文件夹中指定需要授权的页面：

```csharp
// This doesn't work!
.AllowAnonymousToFolder("/Public").AuthorizePage("/Public/Private")
```

在专用页面上要求授权失败。 当 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 都应用于页面时，将 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 优先并控制访问。

## <a name="additional-resources"></a>其他资源

* <xref:razor-pages/razor-pages-conventions>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

在页面应用中控制访问权限的一种方法 Razor 是在启动时使用授权约定。 这些约定允许用户授权用户，并允许匿名用户访问页面的各个页面或文件夹。 本主题中所述的约定会自动应用 [授权筛选器](xref:mvc/controllers/filters#authorization-filters) 来控制访问权限。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/razor-pages-authorization/samples)（[如何下载](xref:index#how-to-download-a-sample)）

示例应用[ cookie ASP.NET Core Identity 不使用身份验证](xref:security/authentication/cookie)。 本主题中所示的概念和示例同样适用于使用的应用 ASP.NET Core Identity 。 若要使用 ASP.NET Core Identity ，请按照中的指导进行操作 <xref:security/authentication/identity> 。

## <a name="require-authorization-to-access-a-page"></a>需要授权才能访问页面

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*> 约定通过将 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到页面中的指定路径：

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,4)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径，无扩展名，只包含正斜杠。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizePage 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizePage*)：

```csharp
options.Conventions.AuthorizePage("/Contact", "AtLeast21");
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter>可应用于具有 filter 特性的页模型类 `[Authorize]` 。 有关详细信息，请参阅 [授权筛选器属性](xref:razor-pages/filter#authorize-filter-attribute)。

## <a name="require-authorization-to-access-a-folder-of-pages"></a>需要授权才能访问页的文件夹

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*> 约定通过将 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到文件夹中指定路径的所有页面：

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,5)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizeFolder 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeFolder*)：

```csharp
options.Conventions.AuthorizeFolder("/Private", "AtLeast21");
```

## <a name="require-authorization-to-access-an-area-page"></a>需要授权才能访问区域页

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*> 约定通过将 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到指定路径处的区域页：

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts");
```

页面名称是文件的路径，该文件的扩展名相对于指定区域的页根目录。 例如，文件 *区域/ Identity /Pages/Manage/Accounts.cshtml* 的页面名称为 */Manage/Accounts* 。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizeAreaPage 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaPage*)：

```csharp
options.Conventions.AuthorizeAreaPage("Identity", "/Manage/Accounts", "AtLeast21");
```

## <a name="require-authorization-to-access-a-folder-of-areas"></a>需要授权才能访问区域文件夹

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*> 约定通过将 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 到文件夹中指定路径的所有区域：

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage");
```

文件夹路径是文件夹相对于指定区域的页面根目录的路径。 例如， *区域/ Identity /Pages/Manage/* 下的文件的文件夹路径为 */Manage* 。

若要指定 [授权策略](xref:security/authorization/policies)，请使用 [AuthorizeAreaFolder 重载](xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AuthorizeAreaFolder*)：

```csharp
options.Conventions.AuthorizeAreaFolder("Identity", "/Manage", "AtLeast21");
```

## <a name="allow-anonymous-access-to-a-page"></a>允许匿名访问页面

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToPage*> 约定通过将 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 到位于指定路径的页面：

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,6)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径，无扩展名，只包含正斜杠。

## <a name="allow-anonymous-access-to-a-folder-of-pages"></a>允许匿名访问页面文件夹

使用 <xref:Microsoft.Extensions.DependencyInjection.PageConventionCollectionExtensions.AllowAnonymousToFolder*> 约定通过将 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.AddRazorPagesOptions*> 添加 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 到文件夹中指定路径的所有页面：

[!code-csharp[](razor-pages-authorization/samples/2.x/AuthorizationSample/Startup.cs?name=snippet1&highlight=2,7)]

指定的路径是视图引擎路径，它是 Razor 页面根相对路径。

## <a name="note-on-combining-authorized-and-anonymous-access"></a>合并授权访问和匿名访问时的注意事项

有效的方法是，指定需要授权的页面文件夹，并指定该文件夹中的页面允许匿名访问：

```csharp
// This works.
.AuthorizeFolder("/Private").AllowAnonymousToPage("/Private/Public")
```

但是，反之则无效。 不能声明用于匿名访问的页面文件夹，然后在该文件夹中指定需要授权的页面：

```csharp
// This doesn't work!
.AllowAnonymousToFolder("/Public").AuthorizePage("/Public/Private")
```

在专用页面上要求授权失败。 当 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Authorization.AuthorizeFilter> 都应用于页面时，将 <xref:Microsoft.AspNetCore.Mvc.Authorization.AllowAnonymousFilter> 优先并控制访问。

## <a name="additional-resources"></a>其他资源

* <xref:razor-pages/razor-pages-conventions>
* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageConventionCollection>

::: moniker-end
