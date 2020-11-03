---
title: ASP.NET Core 中的区域
author: rick-anderson
description: 了解 ASP.NET MVC 的区域功能如何将相关功能以单独的名称空间（用于路由）和文件夹结构（用于视图）的形式组织到一个组中。
ms.author: riande
ms.date: 03/21/2019
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
uid: mvc/controllers/areas
ms.openlocfilehash: 42eec406813adce4d7edbc1ab66a1f689c4aca0e
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053522"
---
# <a name="areas-in-aspnet-core"></a>ASP.NET Core 中的区域

作者：[Dhananjay Kumar](https://twitter.com/debug_mode) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

区域是一种 ASP.NET 功能，用于将相关功能作为单独的组进行组织：

* 用于路由的命名空间。
* 视图和页面的文件夹结构 Razor 。

使用区域通过向 `area` `controller` 和 `action` 或 Razor 页添加另一个路由参数来创建用于路由目的的层次结构 `page` 。

区域提供了一种方法，用于将 ASP.NET Core Web 应用划分为较小的功能组，每个组都有自己的一组 Razor 页、控制器、视图和模型。 区域实际上是应用内的结构。 在 ASP.NET Core Web 项目中，Pages、模型、控制器和视图等逻辑组件保存在不同的文件夹中。 ASP.NET Core 运行时使用命名约定来创建这些组件之间的关系。 对于大型应用，将应用分区为独立的高级功能区域可能更有利。 例如，具有多个业务单位（如结账、计费、搜索等）的电子商务应用。 其中每个单位都有各自的区域来包含视图、控制器、 Razor 页面和模型。

如果发生以下情况，请考虑在项目中使用区域：

* 应用由可以进行逻辑分隔的多个高级功能组件组成。
* 想对应用进行分区，以便可以独立处理每个功能区域。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/31samples)（[如何下载](xref:index#how-to-download-a-sample)）。 下载示例提供了用于测试区域的基本应用。

如果使用的是 Razor 页面，请参阅本文档中 [包含 Razor 页面的区域](#areas-with-razor-pages) 。

## <a name="areas-for-controllers-with-views"></a>带视图的控制器区域

使用区域、控制器和视图的典型 ASP.NET Core Web 应用包含以下内容：

* [区域文件夹结构](#area-folder-structure)。
* 具有属性的控制器 [`[Area]`](#attribute) 用于将控制器与区域关联：

  [!code-csharp[](areas/31samples/MVCareas/Areas/Products/Controllers/ManageController.cs?name=snippet2)]

* [添加到启动的区域路由](#add-area-route)：

  [!code-csharp[](areas/31samples/MVCareas/Startup.cs?name=snippet2&highlight=3-6)]

### <a name="area-folder-structure"></a>区域文件夹结构

请考虑具有两个逻辑组（产品和服务）的应用  。 使用区域，文件夹结构类似于以下内容：

* 项目名称
  * Areas
    * 产品
      * Controllers
        * HomeController.cs
        * ManageController.cs
      * 视图
        * 主页
          * Index.cshtml
        * 管理
          * Index.cshtml
          * About.cshtml
    * 服务
      * Controllers
        * HomeController.cs
      * 视图
        * 主页
          * Index.cshtml

虽然前面的布局是使用区域时的典型布局，但只需要视图文件即可使用此文件夹结构。 视图发现按以下顺序搜索匹配的区域视图文件：

```text
/Areas/<Area-Name>/Views/<Controller-Name>/<Action-Name>.cshtml
/Areas/<Area-Name>/Views/Shared/<Action-Name>.cshtml
/Views/Shared/<Action-Name>.cshtml
/Pages/Shared/<Action-Name>.cshtml
```

<a name="attribute"></a>

### <a name="associate-the-controller-with-an-area"></a>将控制器与区域关联

区域控制器是用[ &lbrack; area &rbrack; ](xref:Microsoft.AspNetCore.Mvc.AreaAttribute)特性指定的：

[!code-csharp[](areas/31samples/MVCareas/Areas/Products/Controllers/ManageController.cs?highlight=5&name=snippet)]

### <a name="add-area-route"></a>添加区域路由

区域路由通常使用  [传统路由](xref:mvc/controllers/routing#cr) ，而不是 [属性路由](xref:mvc/controllers/routing#ar)。 传统路由依赖于顺序。 一般情况下，具有区域的路由应放在路由表中靠前的位置，因为它们比没有区域的路由更特定。

如果所有区域的 url 空间一致，则 `{area:...}` 可用作路由模板中的令牌：

[!code-csharp[](areas/31samples/MVCareas/Startup.cs?name=snippet&highlight=21-23)]

在前面的代码中，`exists` 应用了路由必须与区域匹配的约束。 使用 `{area:...}` `MapControllerRoute` ：

* 是将路由添加到区域的最不复杂的机制。
* 将所有控制器与属性进行匹配 `[Area("Area name")]` 。

以下代码使用 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute*> 创建两个命名区域路由：

[!code-csharp[](areas/31samples/MVCareas/StartupMapAreaRoute.cs?name=snippet&highlight=21-29)]

有关详细信息，请参阅[区域路由](xref:mvc/controllers/routing#areas)。

### <a name="link-generation-with-mvc-areas"></a>MVC 区域的链接生成

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/31samples)中的以下代码显示指定区域的链接生成：

[!code-cshtml[](areas/31samples/MVCareas/Views/Shared/_testLinksPartial.cshtml?name=snippet)]

示例下载包含 [部分视图](xref:mvc/views/partial) ，其中包含：

* 前面的链接。
* 与前面的链接相似，但 `area` 未指定。

在[布局文件](xref:mvc/views/layout)中引用部分视图，因此应用中的每个页面都显示生成的链接。 在未指定区域的情况下生成的链接仅在从同一区域和控制器中的页面引用时才有效。

如果未指定区域或控制器，路由取决于环境值[](xref:mvc/controllers/routing#ambient)。 当前请求的当前路由值被视为链接生成的环境值。 在许多情况下，对于示例应用程序，使用环境值时，将生成带有未指定区域的标记的错误链接。

有关详细信息，请参阅[路由到控制器操作](xref:mvc/controllers/routing)。

### <a name="shared-layout-for-areas-using-the-_viewstartcshtml-file"></a>使用 _ViewStart.cshtml 文件的共享区域布局

若要为整个应用共享公共布局，请在 [应用程序根文件夹](#arf)中保留 *_ViewStart。* 有关详细信息，请参阅<xref:mvc/views/layout>。

<a name="arf"></a>

### <a name="application-root-folder"></a>应用程序根文件夹

应用程序根文件夹是包含 ASP.NET Core 模板创建的 web 应用中的 *Startup.cs* 的文件夹。

### <a name="_viewimportscshtml"></a>_ViewImports.cshtml

 对于 MVC， *_ViewImports 的/Views/* 和 */Pages/_ViewImports* Razor ，不会导入到区域中的视图。 使用以下方法之一来提供所有视图的视图导入：

* 将 *_ViewImports* 添加到 [应用程序根文件夹](#arf)。 应用程序根文件夹中的 *_ViewImports* 将应用到应用中的所有视图。
* 将 *_ViewImports* 的文件复制到 "区域" 下的相应视图文件夹中。

*_ViewImports 的 cshtml* 文件通常包含 [标记帮助](xref:mvc/views/tag-helpers/intro)程序导入、 `@using` 和 `@inject` 语句。 有关详细信息，请参阅 [导入共享指令](xref:mvc/views/layout#importing-shared-directives)。

<a name="rename"></a>

### <a name="change-default-area-folder-where-views-are-stored"></a>更改存储视图的默认区域文件夹

以下代码将默认的区域文件夹从 `"Areas"` 改为`"MyAreas"`：

[!code-csharp[](areas/31samples/MVCareas/Startup2.cs?name=snippet)]

<a name="arp"></a>

## <a name="areas-with-no-locrazor-pages"></a>包含页面的区域 Razor

包含页面的区域 Razor `Areas/<area name>/Pages` 在应用的根目录中需要一个文件夹。 以下文件夹结构用于[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/31samples)：

* 项目名称
  * Areas
    * 产品
      * 页数
        * _ViewImports
        * 关于
        * 索引
    * 服务
      * 页数
        * 管理
          * 关于
          * 索引

### <a name="link-generation-with-no-locrazor-pages-and-areas"></a>与 Razor 页面和区域一起生成链接

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/samples/RPareas)中的以下代码显示指定区域（例如 `asp-area="Products"`）的链接生成：

[!code-cshtml[](areas/31samples/RPareas/Pages/Shared/_testLinksPartial.cshtml?name=snippet)]

示例下载包含[部分视图](xref:mvc/views/partial)，该视图包含以前的链接和相同的链接（未指定区域）。 在[布局文件](xref:mvc/views/layout)中引用部分视图，因此应用中的每个页面都显示生成的链接。 在未指定区域的情况下生成的链接仅在从同一区域中的页引用时才有效。

如果未指定区域，路由取决于环境  值。 当前请求的当前路由值被视为链接生成的环境值。 在许多情况下，对于示例应用，使用环境值会生成错误的链接。 例如，考虑从下面的代码生成的链接：

[!code-cshtml[](areas/31samples/RPareas/Pages/Shared/_testLinksPartial.cshtml?name=snippet2)]

对于上述代码：

* 只有当最后一个请求是针对 `Services` 区域的页时，从 `<a asp-page="/Manage/About">` 生成的链接才是正确的。 例如，`/Services/Manage/`、`/Services/Manage/Index` 或 `/Services/Manage/About`。
* 只有当最后一个请求是针对 `/Home` 中的页时，从 `<a asp-page="/About">` 生成的链接才是正确的。
* 代码摘自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/31samples/RPareas)。

### <a name="import-namespace-and-tag-helpers-with-_viewimports-file"></a>使用 _ViewImports 文件导入命名空间和标记帮助程序

可以将 *_ViewImports 的 cshtml* 文件添加到每个区域 *页面* 文件夹，以将命名空间和标记帮助程序导入到文件夹中的每一 Razor 页。

请考虑使用示例代码的“服务”区域，它不包含 _ViewImports.cshtml 文件  。 以下标记显示了 " */Services/Manage/About* " Razor 页：

[!code-cshtml[](areas/31samples/RPareas/Areas/Services/Pages/Manage/About.cshtml)]

在前面的标记中：

* 必须使用完全限定的域名来指定模型 (`@model RPareas.Areas.Services.Pages.Manage.AboutModel`)。
* [标记帮助](xref:mvc/views/tag-helpers/intro) 程序由启用 `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`

在示例下载中，“产品”区域包含下列 _ViewImports.cshtml 文件  ：

[!code-cshtml[](areas/31samples/RPareas/Areas/Products/Pages/_ViewImports.cshtml)]

以下标记显示了 " */Products/About* " Razor 页：

[!code-cshtml[](areas/31samples/RPareas/Areas/Products/Pages/About.cshtml)]

在前面的文件中，将 `@addTagHelper` 按 *区域/产品/页面/_ViewImports cshtml* 文件将命名空间和指令导入文件。

有关详细信息，请参阅[管理标记帮助程序范围](xref:mvc/views/tag-helpers/intro?view=aspnetcore-2.2#managing-tag-helper-scope)和[导入共享指令](xref:mvc/views/layout#importing-shared-directives)。

### <a name="shared-layout-for-no-locrazor-pages-areas"></a>页面区域的共享布局 Razor

要共享整个应用的常用布局，请将 _ViewStart.cshtml 移动到应用程序根文件夹  。

### <a name="publishing-areas"></a>发布区域

当 *.csproj 文件中包含 `<Project Sdk="Microsoft.NET.Sdk.Web">` 时，所有 *.cshtml 文件以及 wwwroot 目录中的文件都将发布到输出中  。
::: moniker-end

::: moniker range="< aspnetcore-3.0"

区域是 ASP.NET 功能，用于将相关功能以单独的名称空间（用于路由）和文件夹结构（用于视图）的形式组织到一个组中。 使用区域通过向 `area` `controller` 和 `action` 或 Razor 页添加另一个路由参数来创建用于路由目的的层次结构 `page` 。

区域提供了一种方法，用于将 ASP.NET Core Web 应用划分为较小的功能组，每个组都有自己的一组 Razor 页、控制器、视图和模型。 区域实际上是应用内的结构。 在 ASP.NET Core Web 项目中，Pages、模型、控制器和视图等逻辑组件保存在不同的文件夹中。 ASP.NET Core 运行时使用命名约定来创建这些组件之间的关系。 对于大型应用，将应用分区为独立的高级功能区域可能更有利。 例如，具有多个业务单位（如结账、计费、搜索等）的电子商务应用。 其中每个单位都有各自的区域来包含视图、控制器、 Razor 页面和模型。

如果发生以下情况，请考虑在项目中使用区域：

* 应用由可以进行逻辑分隔的多个高级功能组件组成。
* 想对应用进行分区，以便可以独立处理每个功能区域。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/samples)（[如何下载](xref:index#how-to-download-a-sample)）。 下载示例提供了用于测试区域的基本应用。

如果使用的是 Razor 页面，请参阅本文档中 [包含 Razor 页面的区域](#areas-with-razor-pages) 。

## <a name="areas-for-controllers-with-views"></a>带视图的控制器区域

使用区域、控制器和视图的典型 ASP.NET Core Web 应用包含以下内容：

* [区域文件夹结构](#area-folder-structure)。
* 具有属性的控制器 [`[Area]`](#attribute) 用于将控制器与区域关联：

  [!code-csharp[](areas/samples/MVCareas/Areas/Products/Controllers/ManageController.cs?name=snippet2)]

* [添加到启动的区域路由](#add-area-route)：

  [!code-csharp[](areas/samples/MVCareas/Startup.cs?name=snippet2&highlight=3-6)]

### <a name="area-folder-structure"></a>区域文件夹结构

请考虑具有两个逻辑组（产品和服务）的应用  。 使用区域，文件夹结构类似于以下内容：

* 项目名称
  * Areas
    * 产品
      * Controllers
        * HomeController.cs
        * ManageController.cs
      * 视图
        * 主页
          * Index.cshtml
        * 管理
          * Index.cshtml
          * About.cshtml
    * 服务
      * Controllers
        * HomeController.cs
      * 视图
        * 主页
          * Index.cshtml

虽然前面的布局是使用区域时的典型布局，但只需要视图文件即可使用此文件夹结构。 视图发现按以下顺序搜索匹配的区域视图文件：

```text
/Areas/<Area-Name>/Views/<Controller-Name>/<Action-Name>.cshtml
/Areas/<Area-Name>/Views/Shared/<Action-Name>.cshtml
/Views/Shared/<Action-Name>.cshtml
/Pages/Shared/<Action-Name>.cshtml
```

<a name="attribute"></a>

### <a name="associate-the-controller-with-an-area"></a>将控制器与区域关联

区域控制器是用[ &lbrack; area &rbrack; ](xref:Microsoft.AspNetCore.Mvc.AreaAttribute)特性指定的：

[!code-csharp[](areas/samples/MVCareas/Areas/Products/Controllers/ManageController.cs?highlight=5&name=snippet)]

### <a name="add-area-route"></a>添加区域路由

区域路由通常使用传统路由，而不使用属性路由。 传统路由依赖于顺序。 一般情况下，具有区域的路由应放在路由表中靠前的位置，因为它们比没有区域的路由更特定。

如果所有区域的 url 空间一致，则 `{area:...}` 可用作路由模板中的令牌：

[!code-csharp[](areas/samples/MVCareas/Startup.cs?name=snippet&highlight=18-21)]

在前面的代码中，`exists` 应用了路由必须与区域匹配的约束。 使用 `{area:...}` 是将路由添加到区域的最简单的机制。

以下代码使用 <xref:Microsoft.AspNetCore.Builder.MvcAreaRouteBuilderExtensions.MapAreaRoute*> 创建两个命名区域路由：

[!code-csharp[](areas/samples/MVCareas/StartupMapAreaRoute.cs?name=snippet&highlight=18-27)]

将 `MapAreaRoute` 与 ASP.NET Core 2.2 配合使用时，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore/issues/7772)。

有关详细信息，请参阅[区域路由](xref:mvc/controllers/routing#areas)。

### <a name="link-generation-with-mvc-areas"></a>MVC 区域的链接生成

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/samples)中的以下代码显示指定区域的链接生成：

[!code-cshtml[](areas/samples/MVCareas/Views/Shared/_testLinksPartial.cshtml?name=snippet)]

使用上述代码生成的链接在应用的任何位置都有效。

示例下载包含[部分视图](xref:mvc/views/partial)，该视图包含以前的链接和相同的链接（未指定区域）。 在[布局文件](xref:mvc/views/layout)中引用部分视图，因此应用中的每个页面都显示生成的链接。 在未指定区域的情况下生成的链接仅在从同一区域和控制器中的页面引用时才有效。

如果未指定区域或控制器，路由取决于环境值  。 当前请求的当前路由值被视为链接生成的环境值。 在许多情况下，对于示例应用，使用环境值会生成错误的链接。

有关详细信息，请参阅[路由到控制器操作](xref:mvc/controllers/routing)。

### <a name="shared-layout-for-areas-using-the-_viewstartcshtml-file"></a>使用 _ViewStart.cshtml 文件的共享区域布局

要共享整个应用的常用布局，请将 _ViewStart.cshtml 移动到应用程序根文件夹  。

### <a name="_viewimportscshtml"></a>_ViewImports.cshtml

在其标准位置，/Views/_ViewImports.cshtml 不适用于区域  。 若要在你的区域中使用常见 [标记帮助](xref:mvc/views/tag-helpers/intro)程序、或，请 `@using` `@inject` 确保适当的 *_ViewImports cshtml* 文件 [适用于你的区域视图](xref:mvc/views/layout#importing-shared-directives)。 如果希望所有视图都具有相同的行为，请将 /Views/_ViewImports.cshtml 迁移到应用程序根  。

<a name="rename"></a>

### <a name="change-default-area-folder-where-views-are-stored"></a>更改存储视图的默认区域文件夹

以下代码将默认的区域文件夹从 `"Areas"` 改为`"MyAreas"`：

[!code-csharp[](areas/samples/MVCareas/Startup2.cs?name=snippet)]

<a name="arp"></a>

## <a name="areas-with-no-locrazor-pages"></a>包含页面的区域 Razor

包含页面的区域 Razor `Areas/<area name>/Pages` 在应用的根目录中需要一个文件夹。 以下文件夹结构用于[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/samples)：

* 项目名称
  * Areas
    * 产品
      * 页数
        * _ViewImports
        * 关于
        * 索引
    * 服务
      * 页数
        * 管理
          * 关于
          * 索引

### <a name="link-generation-with-no-locrazor-pages-and-areas"></a>与 Razor 页面和区域一起生成链接

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/samples/RPareas)中的以下代码显示指定区域（例如 `asp-area="Products"`）的链接生成：

[!code-cshtml[](areas/samples/RPareas/Pages/Shared/_testLinksPartial.cshtml?name=snippet)]

使用上述代码生成的链接在应用的任何位置都有效。

示例下载包含[部分视图](xref:mvc/views/partial)，该视图包含以前的链接和相同的链接（未指定区域）。 在[布局文件](xref:mvc/views/layout)中引用部分视图，因此应用中的每个页面都显示生成的链接。 在未指定区域的情况下生成的链接仅在从同一区域中的页引用时才有效。

如果未指定区域，路由取决于环境  值。 当前请求的当前路由值被视为链接生成的环境值。 在许多情况下，对于示例应用，使用环境值会生成错误的链接。 例如，考虑从下面的代码生成的链接：

[!code-cshtml[](areas/samples/RPareas/Pages/Shared/_testLinksPartial.cshtml?name=snippet2)]

对于上述代码：

* 只有当最后一个请求是针对 `Services` 区域的页时，从 `<a asp-page="/Manage/About">` 生成的链接才是正确的。 例如，`/Services/Manage/`、`/Services/Manage/Index` 或 `/Services/Manage/About`。
* 只有当最后一个请求是针对 `/Home` 中的页时，从 `<a asp-page="/About">` 生成的链接才是正确的。
* 代码摘自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/areas/samples/RPareas)。

### <a name="import-namespace-and-tag-helpers-with-_viewimports-file"></a>使用 _ViewImports 文件导入命名空间和标记帮助程序

可以将 *_ViewImports 的 cshtml* 文件添加到每个区域 *页面* 文件夹，以将命名空间和标记帮助程序导入到文件夹中的每一 Razor 页。

请考虑使用示例代码的“服务”区域，它不包含 _ViewImports.cshtml 文件  。 以下标记显示了 " */Services/Manage/About* " Razor 页：

[!code-cshtml[](areas/samples/RPareas/Areas/Services/Pages/Manage/About.cshtml)]

在前面的标记中：

* 必须使用完全限定的域名来指定模型 (`@model RPareas.Areas.Services.Pages.Manage.AboutModel`)。
* [标记帮助](xref:mvc/views/tag-helpers/intro) 程序由启用 `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers`

在示例下载中，“产品”区域包含下列 _ViewImports.cshtml 文件  ：

[!code-cshtml[](areas/samples/RPareas/Areas/Products/Pages/_ViewImports.cshtml)]

以下标记显示了 " */Products/About* " Razor 页：

[!code-cshtml[](areas/samples/RPareas/Areas/Products/Pages/About.cshtml)]

在前面的文件中，将 `@addTagHelper` 按 *区域/产品/页面/_ViewImports cshtml* 文件将命名空间和指令导入文件。

有关详细信息，请参阅[管理标记帮助程序范围](xref:mvc/views/tag-helpers/intro?view=aspnetcore-2.2#managing-tag-helper-scope)和[导入共享指令](xref:mvc/views/layout#importing-shared-directives)。

### <a name="shared-layout-for-no-locrazor-pages-areas"></a>页面区域的共享布局 Razor

要共享整个应用的常用布局，请将 _ViewStart.cshtml 移动到应用程序根文件夹  。

### <a name="publishing-areas"></a>发布区域

当 *.csproj 文件中包含 `<Project Sdk="Microsoft.NET.Sdk.Web">` 时，所有 *.cshtml 文件以及 wwwroot 目录中的文件都将发布到输出中  。
::: moniker-end
