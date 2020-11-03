---
title: ASP.NET Core 中的 Razor Pages 介绍
author: Rick-Anderson
description: 了解 ASP.NET Core 中的 Razor Pages 如何使基于页面的编码方式比使用 MVC 更简单高效。
monikerRange: '>= aspnetcore-2.0'
ms.author: riande
ms.date: 02/12/2020
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
uid: razor-pages/index
ms.openlocfilehash: ff045b24c351c696566dee6046fc4b76f8f88e1a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059138"
---
# <a name="introduction-to-no-locrazor-pages-in-aspnet-core"></a>ASP.NET Core 中的 Razor Pages 介绍

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Ryan Nowak](https://github.com/rynowak)

通过 Razor Pages 对基于页面的场景编码比使用控制器和视图更轻松、更高效。

若要查找使用模型视图控制器方法的教程，请参阅 [ASP.NET Core MVC 入门](xref:tutorials/first-mvc-app/start-mvc)。

本文档介绍 Razor Pages。 它并不是分步教程。 如果认为某些部分过于复杂，请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)。 有关 ASP.NET Core 的概述，请参阅 [ASP.NET Core 简介](xref:index)。

## <a name="prerequisites"></a>先决条件

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.0.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.0.md)]

---

<a name="rpvs17"></a>

## <a name="create-a-no-locrazor-pages-project"></a>创建 Razor Pages 项目

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)，获取关于如何创建 Razor Pages 项目的详细说明。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

在命令行中运行 `dotnet new webapp`。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)，获取关于如何创建 Razor Pages 项目的详细说明。

---

## <a name="no-locrazor-pages"></a>Razor Pages

Startup.cs 中已启用 Razor 页面：

[!code-csharp[](index/3.0sample/RazorPagesIntro/Startup.cs?name=snippet_Startup&highlight=12,36)]

请考虑一个基本页面：<a name="OnGet"></a>

[!code-cshtml[](index/3.0sample/RazorPagesIntro/Pages/Index.cshtml?highlight=1)]

前面的代码与具有控制器和视图的 ASP.NET Core 应用中使用的 [Razor 视图文件](xref:tutorials/first-mvc-app/adding-view)非常相似。 不同之处在于 [`@page`](xref:mvc/views/razor#page) 指令。 `@page` 使文件转换为一个 MVC 操作 ，这意味着它将直接处理请求，而无需通过控制器处理。 `@page` 必须是页面上的第一个 Razor 指令。 `@page` 会影响其他的 [Razor](xref:mvc/views/razor) 构造。 Razor Pages 文件名有 .cshtml 后缀。

将在以下两个文件中显示使用 `PageModel` 类的类似页面。 Pages/Index2.cshtml 文件：

[!code-cshtml[](index/3.0sample/RazorPagesIntro/Pages/Index2.cshtml)]

Pages/Index2.cshtml.cs 页面模型：

[!code-csharp[](index/3.0sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

按照惯例，`PageModel` 类文件的名称与追加 .cs 的 Razor Page 文件名称相同。 例如，前面的 Razor Page 的名称为 Pages/Index2.cshtml。 包含 `PageModel` 类的文件的名称为 Pages/Index2.cshtml.cs。

页面的 URL 路径的关联由页面在文件系统中的位置决定。 下表显示了 Razor Page 路径及匹配的 URL：

| 文件名和路径               | 匹配的 URL |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` 或 `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` 或 `/Store/Index` |

注意：

* 默认情况下，运行时在“Pages”文件夹中查找 Razor Pages 文件。
* URL 未包含页面时，`Index` 为默认页面。

## <a name="write-a-basic-form"></a>编写基本窗体

由于 Razor Pages 的设计，在构建应用时可轻松实施用于 Web 浏览器的常用模式。 [模型绑定](xref:mvc/models/model-binding)、[标记帮助程序](xref:mvc/views/tag-helpers/intro)和 HTML 帮助程序均只可用于 Razor Page 类中定义的属性。 请参考为 `Contact` 模型实现的基本的“联系我们”窗体页面：

在本文档中的示例中，`DbContext` 在 [Startup.cs](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/razor-pages/index/3.0sample/RazorPagesContacts/Startup.cs#L23-L24) 文件中进行初始化。

内存中数据库需要 `Microsoft.EntityFrameworkCore.InMemory` NuGet 包。

[!code-csharp[](index/3.0sample/RazorPagesContacts/Startup.cs?name=snippet)]

数据模型：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Models/Customer.cs)]

数据库上下文：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Data/CustomerDbContext.cs)]

Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml)]

Pages/Create.cshtml.cs 页面模型：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_ALL)]

按照惯例，`PageModel` 类命名为 `<PageName>Model`并且它与页面位于同一个命名空间中。

使用 `PageModel` 类，可以将页面的逻辑与其展示分离开来。 它定义了页面处理程序，用于处理发送到页面的请求和用于呈现页面的数据。 这种隔离可实现：

* 通过[依赖关系注入](xref:fundamentals/dependency-injection)管理页面依赖项。
* [单元测试](xref:test/razor-pages-tests)

页面包含 `OnPostAsync` 处理程序方法，它在 `POST` 请求上运行（当用户发布窗体时）。 可以添加任何 HTTP 谓词的处理程序方法。 最常见的处理程序是：

* `OnGet`，用于初始化页面所需的状态。 在上面的代码中，`OnGet` 方法显示 CreateModel.cshtml Razor Page。
* `OnPost`，用于处理窗体提交。

`Async` 命名后缀为可选，但是按照惯例通常会将它用于异步函数。 前面的代码通常用于 Razor Pages。

如果你熟悉使用控制器和视图的 ASP.NET 应用：

* 前面示例中的 `OnPostAsync` 代码类似于典型的控制器代码。
* 大多数 MVC 基元（例如[模型绑定](xref:mvc/models/model-binding)、[验证](xref:mvc/models/validation)和操作结果）通过控制器和通过 Razor Pages 的运作方式相同。 

之前的 `OnPostAsync` 方法：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_OnPostAsync)]

`OnPostAsync` 的基本流：

检查验证错误。

* 如果没有错误，则保存数据并重定向。
* 如果有错误，则再次显示页面并附带验证消息。 很多情况下，都会在客户端上检测到验证错误，并且从不将它们提交到服务器。

Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml)]

Pages/Create.cshtml 中呈现的 HTML：

[!code-html[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create4.html)]

在前面的代码中，发布窗体：

* 对于有效数据：

  * `OnPostAsync` 处理程序方法调用 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.RedirectToPage*> 帮助程序方法。 `RedirectToPage` 返回 <xref:Microsoft.AspNetCore.Mvc.RedirectToPageResult> 的实例。 `RedirectToPage`:

    * 是操作结果。
    * 类似于 `RedirectToAction` 或 `RedirectToRoute`（用于控制器和视图）。
    * 针对页面自定义。 在前面的示例中，它将重定向到根索引页 (`/Index`)。 [页面 URL 生成](#url_gen)部分中详细介绍了 `RedirectToPage`。

* 对于传递给服务器的验证错误：

  * `OnPostAsync` 处理程序方法调用 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageBase.Page*> 帮助程序方法。 `Page` 返回 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageResult> 的实例。 返回 `Page` 的过程与控制器中的操作返回 `View` 的过程相似。 `PageResult` 是处理程序方法的默认返回类型。 返回 `void` 的处理程序方法将显示页面。
  * 在前面的示例中，在 [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) 中的值结果不返回 false 的情况下发布窗体。 在此示例中，客户端上不显示任何验证错误。 本文档的后面将介绍验证错误处理。

  [!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=3-6)]

* 对于客户端验证检测到的验证错误：

  * 数据不会发布到服务器。
  * 本文档的后面将介绍客户端验证。

`Customer` 属性使用 [`[BindProperty]`](xref:Microsoft.AspNetCore.Mvc.BindPropertyAttribute) 特性来选择加入模型绑定：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_PageModel&highlight=15-16)]

`[BindProperty]` 不应 用于包含不应由客户端更改的属性的模型。 有关详细信息，请参阅[过度发布](xref:data/ef-rp/crud#overposting)。

Razor Pages 只绑定带有非 `GET` 谓词的属性。 如果绑定到属性，则无需通过编写代码将 HTTP 数据转换为模型类型。 绑定通过使用相同的属性显示窗体字段 (`<input asp-for="Customer.Name">`) 来减少代码，并接受输入。

[!INCLUDE[](~/includes/bind-get.md)]

查看 Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml?highlight=3,9)]

* 在前面的代码中，[输入标记帮助程序](xref:mvc/views/working-with-forms#the-input-tag-helper) `<input asp-for="Customer.Name" />` 将 HTML `<input>` 元素绑定到 `Customer.Name` 模型表达式。
* 使用 [`@addTagHelper`](xref:mvc/views/tag-helpers/intro#addtaghelper-makes-tag-helpers-available) 提供标记帮助程序。

### <a name="the-home-page"></a>主页

Index.cshtml 是主页：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml)]

关联的 `PageModel` 类 (Index.cshtml.cs)：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml.cs?name=snippet)]

Index.cshtml 文件包含以下标记：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml?range=21)]

`<a /a>` [定位点标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)使用 `asp-route-{value}` 属性生成“编辑”页面的链接。 此链接包含路由数据及联系人 ID。 例如 `https://localhost:5001/Edit/1`。 [标记帮助程序](xref:mvc/views/tag-helpers/intro)使服务器端代码可以在 Razor 文件中参与创建和呈现 HTML 元素。

Index.cshtml 文件包含用于为每个客户联系人创建删除按钮的标记：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml?range=22-23)]

呈现的 HTML：

```html
<button type="submit" formaction="/Customers?id=1&amp;handler=delete">delete</button>
```

删除按钮采用 HTML 呈现，其 [formaction](https://developer.mozilla.org/docs/Web/HTML/Element/button#attr-formaction) 包括参数：

* `asp-route-id` 属性指定的客户联系人 ID。
* `asp-page-handler` 属性指定的 `handler`。

选中按钮时，向服务器发送窗体 `POST` 请求。 按照惯例，根据方案 `OnPost[handler]Async` 基于 `handler` 参数的值来选择处理程序方法的名称。

因为本示例中 `handler` 是 `delete`，因此 `OnPostDeleteAsync` 处理程序方法用于处理 `POST` 请求。 如果 `asp-page-handler` 设置为其他值（如 `remove`），则选择名称为 `OnPostRemoveAsync` 的处理程序方法。

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Index.cshtml.cs?name=snippet2)]

`OnPostDeleteAsync` 方法：

* 获取来自查询字符串的 `id`。
* 使用 `FindAsync` 查询客户联系人的数据库。
* 如果找到客户联系人，则会将其删除，并更新数据库。
* 调用 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel.RedirectToPage*>，重定向到根索引页 (`/Index`)。

### <a name="the-editcshtml-file"></a>Edit.cshtml 文件

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Edit.cshtml?highlight=1)]

第一行包含 `@page "{id:int}"` 指令。 路由约束 `"{id:int}"` 告诉页面接受包含 `int` 路由数据的页面请求。 如果页面请求未包含可转换为 `int` 的路由数据，则运行时返回 HTTP 404（未找到）错误。 若要使 ID 可选，请将 `?` 追加到路由约束：

 ```cshtml
@page "{id:int?}"
```

Edit.cshtml.cs 文件：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Edit.cshtml.cs?name=snippet)]

## <a name="validation"></a>验证

验证规则：

* 在模型类中以声明方式指定。
* 在应用中的所有位置强制执行。

<xref:System.ComponentModel.DataAnnotations> 命名空间提供一组内置验证特性，可通过声明方式应用于类或属性。 DataAnnotations 还包含 [`[DataType]`](xref:System.ComponentModel.DataAnnotations.DataTypeAttribute) 等格式特性，有助于格式设置但不提供任何验证。

请考虑 `Customer` 模型：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Models/Customer.cs)]

使用以下 Create.cshtml 视图文件：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create3.cshtml?highlight=3,8-9,15-99)]

前面的代码：

* 包括 jQuery 和 jQuery 验证脚本。
* 使用 `<div />` 和 `<span />` [标记帮助程序](xref:mvc/views/tag-helpers/intro)以实现：

  * 客户端验证。
  * 验证错误呈现。

* 则会生成以下 HTML：

  [!code-html[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create5.html)]

如果在不使用名称值的情况下发布“创建”窗体，则将显示错误消息“名称字段是必需的”。 窗体上。 如果客户端上已启用 JavaScript，浏览器会显示错误，而不会发布到服务器。

`[StringLength(10)]` 特性在呈现的 HTML 上生成 `data-val-length-max="10"`。 `data-val-length-max` 阻止浏览器输入超过指定最大长度的内容。 如果使用 [Fiddler](https://www.telerik.com/fiddler) 等工具来编辑和重播文章：

* 对于长度超过 10 的名称。
* 错误消息“字段名称必须是最大长度为 10 的字符串。” 将返回。

考虑下列 `Movie` 模型：

[!code-csharp[](~/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie30/Models/MovieDateRatingDA.cs?name=snippet1)]

验证特性指定要对应用这些特性的模型属性强制执行的行为：

* `Required` 和 `MinimumLength` 特性表示属性必须有值，但用户可输入空格来满足此验证。
* `RegularExpression` 特性用于限制可输入的字符。 在上述代码中，即“Genre”（分类）：

  * 只能使用字母。
  * 第一个字母必须为大写。 不允许使用空格、数字和特殊字符。

* `RegularExpression`“Rating”（分级）：

  * 要求第一个字符为大写字母。
  * 允许在后续空格中使用特殊字符和数字。 “PG-13”对“分级”有效，但对于“分类”无效。

* `Range` 特性将值限制在指定范围内。
* `StringLength` 特性可以设置字符串属性的最大长度，以及可选的最小长度。
* 从本质上来说，需要值类型（如 `decimal`、`int`、`float`、`DateTime`），但不需要 `[Required]` 特性。

`Movie` 模型的“创建”页面显示无效值错误：

![带有多个 jQuery 客户端验证错误的电影视图表单](~/tutorials/razor-pages/validation/_static/val.png)

有关详情，请参阅：

* [将验证添加到电影应用](xref:tutorials/razor-pages/validation)
* [ASP.NET Core 中的模型验证](xref:mvc/models/validation).

## <a name="handle-head-requests-with-an-onget-handler-fallback"></a>使用 OnGet 处理程序回退来处理 HEAD 请求

`HEAD` 请求可检索特定资源的标头。 与 `GET` 请求不同，`HEAD` 请求不返回响应正文。

通常，针对 `HEAD` 请求创建和调用 `OnHead` 处理程序：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Privacy.cshtml.cs?name=snippet)]

如果未定义 `OnHead` 处理程序，则 Razor Pages 会回退到调用 `OnGet` 处理程序。

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-no-locrazor-pages"></a>XSRF/CSRF 和 Razor Pages

Razor Pages 由 [防伪造验证](xref:security/anti-request-forgery)保护。 [FormTagHelper](xref:mvc/views/working-with-forms#the-form-tag-helper) 将防伪造令牌注入 HTML 窗体元素。

<a name="layout"></a>

## <a name="using-layouts-partials-templates-and-tag-helpers-with-no-locrazor-pages"></a>将布局、分区、模板和标记帮助程序用于 Razor Pages

页面可使用 Razor 视图引擎的所有功能。 布局、分区、模板、标记帮助程序、_ViewStart.cshtml 和 _ViewImports.cshtml 的工作方式与它们在传统的 Razor 视图中的工作方式相同。

让我们使用其中的一些功能来整理此页面。

向 Pages/Shared/_Layout.cshtml 添加[布局页面](xref:mvc/views/layout)：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Shared/_Layout2.cshtml?hightlight=12)]

[布局](xref:mvc/views/layout)：

* 控制每个页面的布局（页面选择退出布局时除外）。
* 导入 HTML 结构，例如 JavaScript 和样式表。
* 调用 `@RenderBody()` 时，呈现 Razor Page 的内容。

有关详细信息，请参阅[布局页面](xref:mvc/views/layout)。

在 Pages/_ViewStart.cshtml 中设置 [Layout](xref:mvc/views/layout#specifying-a-layout) 属性：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

布局位于“页面/共享”文件夹中。 页面按层次结构从当前页面的文件夹开始查找其他视图（布局、模板、分区）。 可以从“Pages”文件夹下的任意 Razor 页面使用“Pages/Shared”文件夹中的布局。

布局文件应位于 Pages/Shared 文件夹中。

建议不要将布局文件放在“视图/共享”文件夹中。 视图/共享 是一种 MVC 视图模式。 Razor Pages 旨在依赖文件夹层次结构，而非路径约定。

Razor Page 中的视图搜索包含“Pages”文件夹。 用于 MVC 控制器和传统 Razor 视图的布局、模板和分区可正常运行。

添加 Pages/_ViewImports.cshtml 文件：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

本教程的后续部分中将介绍 `@namespace`。 `@addTagHelper` 指令将[内置标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/Index)引入“页面”文件夹中的所有页面。

<a name="namespace"></a>

页面上设置的 `@namespace` 指令：

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

`@namespace` 指令将为页面设置命名空间。 `@model` 指令无需包含命名空间。

_ViewImports.cshtml 中包含 `@namespace` 指令后，指定的命名空间将为在导入 `@namespace` 指令的页面中生成的命名空间提供前缀。 生成的命名空间的剩余部分（后缀部分）是包含 _ViewImports.cshtml 的文件夹与包含页面的文件夹之间以点分隔的相对路径。

例如，`PageModel` 类 Pages/Customers/Edit.cshtml.cs 显式设置命名空间：

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

Pages/_ViewImports.cshtml 文件设置以下命名空间：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

为 Pages/Customers/Edit.cshtml Razor Page 生成的命名空间与 `PageModel` 类相同。

`@namespace`  *也适用于传统 Razor 视图。*

考虑 Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create3.cshtml?highlight=2-3)]

包含 _ViewImports.cshtml 的已更新的 Pages/Create.cshtml 视图文件和前面的布局文件 ：

[!code-cshtml[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create4.cshtml?highlight=2)]

在前面的代码中，_ViewImports.cshtml 导入了命名空间和标记帮助程序。 布局文件导入了 JavaScript 文件。

[Razor Pages 初学者项目](#rpvs17)包含 Pages/_ValidationScriptsPartial.cshtml，它与客户端验证联合。

有关分部视图的详细信息，请参阅 <xref:mvc/views/partial>。

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>页面的 URL 生成

之前显示的 `Create` 页面使用 `RedirectToPage`：

[!code-csharp[](index/3.0sample/RazorPagesContacts/Pages/Customers/Create.cshtml.cs?name=snippet_PageModel&highlight=28)]

应用具有以下文件/文件夹结构：

* /Pages

  * *Index.cshtml*
  * Privacy.cshtml
  * /Customers

    * Create.cshtml
    * Edit.cshtml
    * *Index.cshtml*

成功后，Pages/Customers/Create.cshtml and Pages/Customers/Edit.cshtml 页面将重定向到 Pages/Customers/Index.cshtml  。 字符串 `./Index` 是用于访问前一页的相对页名称。 它用于生成 *Pages/Customers/Index.cshtml* 页面的 URL。 例如：

* `Url.Page("./Index", ...)`
* `<a asp-page="./Index">Customers Index Page</a>`
* `RedirectToPage("./Index")`

绝对页名称 `/Index` 用于生成 *Pages/Index. cshtml* 页面的 URL。 例如：

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">Home Index Page</a>`
* `RedirectToPage("/Index")`

页面名称是从根“/Pages”文件夹到页面的路径（包含前导 `/`，例如 `/Index`）。 与硬编码 URL 相比，前面的 URL 生成示例提供了改进的选项和功能。 URL 生成使用[路由](xref:mvc/controllers/routing)，并且可以根据目标路径定义路由的方式生成参数并对参数编码。

页面的 URL 生成支持相对名称。 下表显示了 Pages/Customers/Create.cshtml 中不同的 `RedirectToPage` 参数选择的索引页。

| RedirectToPage(x)| 页面 |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

<!-- Test via ~/razor-pages/index/3.0sample/RazorPagesContacts/Pages/Customers/Details.cshtml.cs -->

`RedirectToPage("Index")`、`RedirectToPage("./Index")` 和 `RedirectToPage("../Index")` 是相对名称。 结合`RedirectToPage` 参数与当前页的路径来计算目标页面的名称。

构建结构复杂的站点时，相对名称链接很有用。 如果使用相对名称链接文件夹中的页面：

* 重命名文件夹不会破坏相对链接。
* 链接不会中断，因为它们不包含文件夹名称。

若要重定向到不同[区域](xref:mvc/controllers/areas)中的页面，请指定区域：

```csharp
RedirectToPage("/Index", new { area = "Services" });
```

有关详细信息，请参阅 <xref:mvc/controllers/areas> 和 <xref:razor-pages/razor-pages-conventions>。

## <a name="viewdata-attribute"></a>ViewData 特性

可以通过 <xref:Microsoft.AspNetCore.Mvc.ViewDataAttribute> 将数据传递到页面。 具有 `[ViewData]` 特性的属性从 <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary> 保存和加载值。

在下面的示例中，`AboutModel` 将 `[ViewData]` 特性应用于 `Title` 属性：

```csharp
public class AboutModel : PageModel
{
    [ViewData]
    public string Title { get; } = "About";

    public void OnGet()
    {
    }
}
```

在“关于”页面中，以模型属性的形式访问 `Title` 属性：

```cshtml
<h1>@Model.Title</h1>
```

在布局中，从 ViewData 字典读取标题：

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

## <a name="tempdata"></a>TempData

ASP.NET Core 公开 <xref:Microsoft.AspNetCore.Mvc.Controller.TempData>。 此属性存储未读取的数据。 <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.TempDataDictionary.Keep*> 和 <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.TempDataDictionary.Peek*> 方法可用于检查数据，而不执行删除。 `TempData` 在多个请求需要数据的情况下对重定向很有用。

下面的代码使用 `TempData` 设置 `Message` 的值：

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

Pages/Customers/Index.cshtml 文件中的以下标记使用 `TempData` 显示 `Message` 的值。

```cshtml
<h3>Msg: @Model.Message</h3>
```

Pages/Customers/Index.cshtml.cs 页面模型将 `[TempData]` 属性应用到 `Message` 属性。

```csharp
[TempData]
public string Message { get; set; }
```

有关详细信息，请参阅 [TempData](xref:fundamentals/app-state#tempdata)。

<a name="mhpp"></a>

## <a name="multiple-handlers-per-page"></a>针对一个页面的多个处理程序

以下页面使用 `asp-page-handler` 标记帮助程序为两个处理程序生成标记：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

前面示例中的窗体包含两个提交按钮，每个提交按钮均使用 `FormActionTagHelper` 提交到不同的 URL。 `asp-page-handler` 是 `asp-page` 的配套属性。 `asp-page-handler` 生成提交到页面定义的各个处理程序方法的 URL。 未指定 `asp-page`，因为示例已链接到当前页面。

页面模型：

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

前面的代码使用已命名处理程序方法。 已命名处理程序方法通过采用名称中 `On<HTTP Verb>` 之后及 `Async` 之前的文本（如果有）创建。 在前面的示例中，页面方法是 OnPost **JoinList** Async 和 OnPost **JoinListUC** Async。 删除 OnPost 和 Async 后，处理程序名称为 `JoinList` 和 `JoinListUC`。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

使用前面的代码时，提交到 `OnPostJoinListAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH?handler=JoinList`。 提交到 `OnPostJoinListUCAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH?handler=JoinListUC`。

## <a name="custom-routes"></a>自定义路由

使用 `@page` 指令，执行以下操作：

* 指定页面的自定义路由。 例如，使用 `@page "/Some/Other/Path"` 将“关于”页面的路由设置为 `/Some/Other/Path`。
* 将段追加到页面的默认路由。 例如，可使用 `@page "item"` 将“item”段添加到页面的默认路由。
* 将参数追加到页面的默认路由。 例如，`@page "{id}"` 页面需要 ID 参数 `id`。

支持开头处以波形符 (`~`) 指定的相对于根目录的路径。 例如，`@page "~/Some/Other/Path"` 和 `@page "/Some/Other/Path"` 相同。

如果你不喜欢 URL 中的查询字符串 `?handler=JoinList`，请更改路由，将处理程序名称放在 URL 的路径部分。 可以通过在 `@page` 指令后面添加使用双引号括起来的路由模板来自定义路由。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

使用前面的代码时，提交到 `OnPostJoinListAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH/JoinList`。 提交到 `OnPostJoinListUCAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH/JoinListUC`。

`handler` 前面的 `?` 表示路由参数为可选。

## <a name="advanced-configuration-and-settings"></a>高级配置和设置

大多数应用不需要以下部分中的配置和设置。

要配置高级选项，请使用 <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages%2A> 重载，该重载配置 <xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions>：

[!code-csharp[](index/3.0sample/RazorPagesContacts/StartupRPoptions.cs?name=snippet)]

使用 <xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions> 设置页面的根目录，或者为页面添加应用程序模型约定。 有关约定的详细信息，请参阅 [Razor Pages 授权约定](xref:security/authorization/razor-pages-authorization).

若要预编译视图，请参阅 [Razor 视图编译](xref:mvc/views/view-compilation)。

### <a name="specify-that-no-locrazor-pages-are-at-the-content-root"></a>指定 Razor Pages 位于内容根目录中

默认情况下，Razor Pages 位于 /Pages 目录的根位置。 添加 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcBuilderExtensions.WithRazorPagesAtContentRoot*> 以指定 Razor Pages 位于应用的[内容根](xref:fundamentals/index#content-root) (<xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment.ContentRootPath>)：

[!code-csharp[](index/3.0sample/RazorPagesContacts/StartupWithRazorPagesAtContentRoot.cs?name=snippet)]

### <a name="specify-that-no-locrazor-pages-are-at-a-custom-root-directory"></a>指定 Razor Pages 位于自定义根目录中

添加 <xref:Microsoft.Extensions.DependencyInjection.MvcRazorPagesMvcCoreBuilderExtensions.WithRazorPagesRoot*>，以指定 Razor Pages 位于应用中自定义根目录位置（提供相对路径）：

[!code-csharp[](index/3.0sample/RazorPagesContacts/StartupWithRazorPagesRoot.cs?name=snippet)]

## <a name="additional-resources"></a>其他资源

* 请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)这篇文章以本文为基础撰写。
* [授权属性和 Razor Pages](xref:security/authorization/simple#aarp)
* [下载或查看示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/index/3.0sample)
* <xref:index>
* <xref:mvc/views/razor>
* <xref:mvc/controllers/areas>
* <xref:tutorials/razor-pages/razor-pages-start>
* <xref:security/authorization/razor-pages-authorization>
* <xref:razor-pages/razor-pages-conventions>
* <xref:test/razor-pages-tests>
* <xref:mvc/views/partial>
* <xref:blazor/components/integrate-components-into-razor-pages-and-mvc-apps>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Ryan Nowak](https://github.com/rynowak)

Razor Pages 是 ASP.NET Core MVC 的一个新特性，它可以使基于页面的编码方式更简单高效。

若要查找使用模型视图控制器方法的教程，请参阅 [ASP.NET Core MVC 入门](xref:tutorials/first-mvc-app/start-mvc)。

本文档介绍 Razor Pages。 它并不是分步教程。 如果认为某些部分过于复杂，请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)。 有关 ASP.NET Core 的概述，请参阅 [ASP.NET Core 简介](xref:index)。

## <a name="prerequisites"></a>先决条件

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---

<a name="rpvs17"></a>

## <a name="create-a-no-locrazor-pages-project"></a>创建 Razor Pages 项目

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)，获取关于如何创建 Razor Pages 项目的详细说明。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

在命令行中运行 `dotnet new webapp`。

在 Visual Studio for Mac 中打开生成的 .csproj 文件。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

在命令行中运行 `dotnet new webapp`。

---

## <a name="no-locrazor-pages"></a>Razor Pages

Startup.cs 中已启用 Razor 页面：

[!code-csharp[](index/sample/RazorPagesIntro/Startup.cs?name=snippet_Startup)]

请考虑一个基本页面：<a name="OnGet"></a>

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Index.cshtml)]

前面的代码与具有控制器和视图的 ASP.NET Core 应用中使用的 [Razor 视图文件](xref:tutorials/first-mvc-app/adding-view)非常相似。 不同之处在于 `@page` 指令。 `@page` 使文件转换为一个 MVC 操作 ，这意味着它将直接处理请求，而无需通过控制器处理。 `@page` 必须是页面上的第一个 Razor 指令。 `@page` 会影响其他 Razor 构造的行为。

将在以下两个文件中显示使用 `PageModel` 类的类似页面。 Pages/Index2.cshtml 文件：

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Index2.cshtml)]

Pages/Index2.cshtml.cs 页面模型：

[!code-csharp[](index/sample/RazorPagesIntro/Pages/Index2.cshtml.cs)]

按照惯例，`PageModel` 类文件的名称与追加 .cs 的 Razor Page 文件名称相同。 例如，前面的 Razor Page 的名称为 Pages/Index2.cshtml。 包含 `PageModel` 类的文件的名称为 Pages/Index2.cshtml.cs。

页面的 URL 路径的关联由页面在文件系统中的位置决定。 下表显示了 Razor Page 路径及匹配的 URL：

| 文件名和路径               | 匹配的 URL |
| ----------------- | ------------ |
| */Pages/Index.cshtml* | `/` 或 `/Index` |
| */Pages/Contact.cshtml* | `/Contact` |
| */Pages/Store/Contact.cshtml* | `/Store/Contact` |
| */Pages/Store/Index.cshtml* | `/Store` 或 `/Store/Index` |

注意：

* 默认情况下，运行时在“Pages”文件夹中查找 Razor Pages 文件。
* URL 未包含页面时，`Index` 为默认页面。

## <a name="write-a-basic-form"></a>编写基本窗体

由于 Razor Pages 的设计，在构建应用时可轻松实施用于 Web 浏览器的常用模式。 [模型绑定](xref:mvc/models/model-binding)、[标记帮助程序](xref:mvc/views/tag-helpers/intro)和 HTML 帮助程序均只可用于 Razor Page 类中定义的属性。 请参考为 `Contact` 模型实现的基本的“联系我们”窗体页面：

在本文档中的示例中，`DbContext` 在 [Startup.cs](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/razor-pages/index/sample/RazorPagesContacts/Startup.cs#L15-L16) 文件中进行初始化。

[!code-csharp[](index/sample/RazorPagesContacts/Startup.cs?highlight=15-16)]

数据模型：

[!code-csharp[](index/sample/RazorPagesContacts/Data/Customer.cs)]

数据库上下文：

[!code-csharp[](index/sample/RazorPagesContacts/Data/AppDbContext.cs)]

Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Create.cshtml)]

Pages/Create.cshtml.cs 页面模型：

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_ALL)]

按照惯例，`PageModel` 类命名为 `<PageName>Model`并且它与页面位于同一个命名空间中。

使用 `PageModel` 类，可以将页面的逻辑与其展示分离开来。 它定义了页面处理程序，用于处理发送到页面的请求和用于呈现页面的数据。 这种隔离可实现：

* 通过[依赖关系注入](xref:fundamentals/dependency-injection)管理页面依赖项。
* 对页面执行[单元测试](xref:test/razor-pages-tests)。

页面包含 `OnPostAsync` 处理程序方法，它在 `POST` 请求上运行（当用户发布窗体时）。 可以为任何 HTTP 谓词添加处理程序方法。 最常见的处理程序是：

* `OnGet`，用于初始化页面所需的状态。 [OnGet](#OnGet) 示例。
* `OnPost`，用于处理窗体提交。

`Async` 命名后缀为可选，但是按照惯例通常会将它用于异步函数。 前面的代码通常用于 Razor Pages。

如果你熟悉使用控制器和视图的 ASP.NET 应用：

* 前面示例中的 `OnPostAsync` 代码类似于典型的控制器代码。
* 多数 MVC 基元（例如[模型绑定](xref:mvc/models/model-binding)、[验证](xref:mvc/models/validation)、[验证](xref:mvc/models/validation)和操作结果）都是共享的。

之前的 `OnPostAsync` 方法：

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync)]

`OnPostAsync` 的基本流：

检查验证错误。

* 如果没有错误，则保存数据并重定向。
* 如果有错误，则再次显示页面并附带验证消息。 客户端验证与传统的 ASP.NET Core MVC 应用程序相同。 很多情况下，都会在客户端上检测到验证错误，并且从不将它们提交到服务器。

成功输入数据后，`OnPostAsync` 处理程序方法调用 `RedirectToPage` 帮助程序方法来返回 `RedirectToPageResult` 的实例。 `RedirectToPage` 是新的操作结果，类似于 `RedirectToAction` 或 `RedirectToRoute`，但是已针对页面进行自定义。 在前面的示例中，它将重定向到根索引页 (`/Index`)。 [页面 URL 生成](#url_gen)部分中详细介绍了 `RedirectToPage`。

提交的窗体存在（已传递到服务器的）验证错误时，`OnPostAsync` 处理程序方法调用 `Page` 帮助程序方法。 `Page` 返回 `PageResult` 的实例。 返回 `Page` 的过程与控制器中的操作返回 `View` 的过程相似。 `PageResult` 是处理程序方法的默认返回类型。 返回 `void` 的处理程序方法将显示页面。

`Customer` 属性使用 `[BindProperty]` 特性来选择加入模型绑定。

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_PageModel&highlight=10-11)]

Razor Pages 只绑定带有非 `GET` 谓词的属性。 绑定属性可以减少需要编写的代码量。 绑定通过使用相同的属性显示窗体字段 (`<input asp-for="Customer.Name">`) 来减少代码，并接受输入。

[!INCLUDE[](~/includes/bind-get.md)]

主页 (Index.cshtml)：

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml)]

关联的 `PageModel` 类 (Index.cshtml.cs)：

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs)]

Index.cshtml 文件包含以下标记来创建每个联系人项的编辑链接：

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=21)]

`<a asp-page="./Edit" asp-route-id="@contact.Id">Edit</a>` [定位点标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)使用 `asp-route-{value}` 属性生成“编辑”页面的链接。 此链接包含路由数据及联系人 ID。 例如 `https://localhost:5001/Edit/1`。 [标记帮助程序](xref:mvc/views/tag-helpers/intro)使服务器端代码可以在 Razor 文件中参与创建和呈现 HTML 元素。 标记帮助程序由 `@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers` 启动

Pages/Edit.cshtml 文件：

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Edit.cshtml?highlight=1)]

第一行包含 `@page "{id:int}"` 指令。 路由约束 `"{id:int}"` 告诉页面接受包含 `int` 路由数据的页面请求。 如果页面请求未包含可转换为 `int` 的路由数据，则运行时返回 HTTP 404（未找到）错误。 若要使 ID 可选，请将 `?` 追加到路由约束：

 ```cshtml
@page "{id:int?}"
```

Pages/Edit.cshtml.cs 文件：

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Edit.cshtml.cs)]

Index.cshtml 文件还包含用于为每个客户联系人创建删除按钮的标记：

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Index.cshtml?range=22-23)]

删除按钮采用 HTML 呈现，其 `formaction` 包括参数：

* `asp-route-id` 属性指定的客户联系人 ID。
* `asp-page-handler` 属性指定的 `handler`。

下面是呈现的删除按钮的示例，其中客户联系人 ID 为 `1`：

```html
<button type="submit" formaction="/?id=1&amp;handler=delete">delete</button>
```

选中按钮时，向服务器发送窗体 `POST` 请求。 按照惯例，根据方案 `OnPost[handler]Async` 基于 `handler` 参数的值来选择处理程序方法的名称。

因为本示例中 `handler` 是 `delete`，因此 `OnPostDeleteAsync` 处理程序方法用于处理 `POST` 请求。 如果 `asp-page-handler` 设置为其他值（如 `remove`），则选择名称为 `OnPostRemoveAsync` 的处理程序方法。 以下代码显示了 `OnPostDeleteAsync` 处理程序：

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Index.cshtml.cs?range=26-37)]

`OnPostDeleteAsync` 方法：

* 接受来自查询字符串的 `id`。 如果 Index.cshtml 页面指令包含路由约束 `"{id:int?}"`，则 `id` 来自路由数据。 `id` 的路由数据在 `https://localhost:5001/Customers/2` 等 URI 中指定。
* 使用 `FindAsync` 查询客户联系人的数据库。
* 如果找到客户联系人，则从客户联系人列表将其删除。 数据库将更新。
* 调用 `RedirectToPage`，重定向到根索引页 (`/Index`)。

## <a name="mark-page-properties-as-required"></a>按需标记页面属性

`PageModel` 上的属性可通过 [Required](/dotnet/api/system.componentmodel.dataannotations.requiredattribute) 特性进行标记：

[!code-csharp[](index/sample/Create.cshtml.cs?highlight=3,15-16)]

有关详细信息，请参阅[模型验证](xref:mvc/models/validation)。

## <a name="handle-head-requests-with-an-onget-handler-fallback"></a>使用 OnGet 处理程序回退来处理 HEAD 请求

`HEAD` 请求可检索特定资源的标头。 与 `GET` 请求不同，`HEAD` 请求不返回响应正文。

通常，针对 `HEAD` 请求创建和调用 `OnHead` 处理程序： 

```csharp
public void OnHead()
{
    HttpContext.Response.Headers.Add("HandledBy", "Handled by OnHead!");
}
```

在 ASP.NET Core 2.1 或更高版本中，如果未定义 `OnHead` 处理程序，则 Razor Pages 会回退到调用 `OnGet` 处理程序。 此行为是通过在 `Startup.ConfigureServices` 中调用 [SetCompatibilityVersion](xref:mvc/compatibility-version) 而启用的：

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

默认模板在 ASP.NET Core 2.1 和 2.2 中生成 `SetCompatibilityVersion` 调用。 `SetCompatibilityVersion` 有效地将 Razor Pages 选项 `AllowMappingHeadRequestsToGetHandler` 设置为 `true`。

可显式选择使用特定行为，而不是通过 `SetCompatibilityVersion` 选择使用所有行为。 通过下列代码，可选择允许将 `HEAD` 请求映射到 `OnGet` 处理程序：

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.AllowMappingHeadRequestsToGetHandler = true;
    });
```

<a name="xsrf"></a>

## <a name="xsrfcsrf-and-no-locrazor-pages"></a>XSRF/CSRF 和 Razor Pages

无需为[防伪验证](xref:security/anti-request-forgery)编写任何代码。 Razor Pages 自动将防伪标记生成过程和验证过程包含在内。

<a name="layout"></a>

## <a name="using-layouts-partials-templates-and-tag-helpers-with-no-locrazor-pages"></a>将布局、分区、模板和标记帮助程序用于 Razor Pages

页面可使用 Razor 视图引擎的所有功能。 布局、分区、模板、标记帮助程序、_ViewStart.cshtml 和 _ViewImports.cshtml 的工作方式与它们在传统的 Razor 视图中的工作方式相同。

让我们使用其中的一些功能来整理此页面。

向 Pages/Shared/_Layout.cshtml 添加[布局页面](xref:mvc/views/layout)：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_LayoutSimple.cshtml)]

[布局](xref:mvc/views/layout)：

* 控制每个页面的布局（页面选择退出布局时除外）。
* 导入 HTML 结构，例如 JavaScript 和样式表。

请参阅[布局页面](xref:mvc/views/layout)了解详细信息。

在 Pages/_ViewStart.cshtml 中设置 [Layout](xref:mvc/views/layout#specifying-a-layout) 属性：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewStart.cshtml)]

布局位于“页面/共享”文件夹中。 页面按层次结构从当前页面的文件夹开始查找其他视图（布局、模板、分区）。 可以从“Pages”文件夹下的任意 Razor 页面使用“Pages/Shared”文件夹中的布局。

布局文件应位于 Pages/Shared 文件夹中。

建议不要将布局文件放在“视图/共享”文件夹中。 视图/共享 是一种 MVC 视图模式。 Razor Pages 旨在依赖文件夹层次结构，而非路径约定。

Razor Page 中的视图搜索包含“Pages”文件夹。 要用于 MVC 控制器和传统 Razor 视图的布局、模板和分区可正常运行。

添加 Pages/_ViewImports.cshtml 文件：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml)]

本教程的后续部分中将介绍 `@namespace`。 `@addTagHelper` 指令将[内置标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/Index)引入“页面”文件夹中的所有页面。

<a name="namespace"></a>

页面上显式使用 `@namespace` 指令后：

[!code-cshtml[](index/sample/RazorPagesIntro/Pages/Customers/Namespace2.cshtml?highlight=2)]

此指令将为页面设置命名空间。 `@model` 指令无需包含命名空间。

_ViewImports.cshtml 中包含 `@namespace` 指令后，指定的命名空间将为在导入 `@namespace` 指令的页面中生成的命名空间提供前缀。 生成的命名空间的剩余部分（后缀部分）是包含 _ViewImports.cshtml 的文件夹与包含页面的文件夹之间以点分隔的相对路径。

例如，`PageModel` 类 Pages/Customers/Edit.cshtml.cs 显式设置命名空间：

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/Edit.cshtml.cs?name=snippet_namespace)]

Pages/_ViewImports.cshtml 文件设置以下命名空间：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/_ViewImports.cshtml?highlight=1)]

为 Pages/Customers/Edit.cshtml Razor Page 生成的命名空间与 `PageModel` 类相同。

`@namespace`  *也适用于传统 Razor 视图。*

原始的 Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/sample/RazorPagesContacts/Pages/Create.cshtml?highlight=2)]

更新后的 Pages/Create.cshtml 视图文件：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/Create.cshtml?highlight=2)]

[Razor Pages 初学者项目](#rpvs17)包含 Pages/_ValidationScriptsPartial.cshtml，它与客户端验证联合。

有关分部视图的详细信息，请参阅 <xref:mvc/views/partial>。

<a name="url_gen"></a>

## <a name="url-generation-for-pages"></a>页面的 URL 生成

之前显示的 `Create` 页面使用 `RedirectToPage`：

[!code-csharp[](index/sample/RazorPagesContacts/Pages/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=10)]

应用具有以下文件/文件夹结构：

* /Pages

  * *Index.cshtml*
  * /Customers

    * Create.cshtml
    * Edit.cshtml
    * *Index.cshtml*

成功后，Pages/Customers/Create.cshtml 和 Pages/Customers/Edit.cshtml 页面将重定向到 Pages/Index.cshtml。 字符串 `/Index` 是用于访问上一页的 URI 的组成部分。 可以使用字符串 `/Index` 生成 Pages/Index.cshtml 页面的 URI。 例如：

* `Url.Page("/Index", ...)`
* `<a asp-page="/Index">My Index Page</a>`
* `RedirectToPage("/Index")`

页面名称是从根“/Pages”文件夹到页面的路径（包含前导 `/`，例如 `/Index`）。 与硬编码 URL 相比，前面的 URL 生成示例提供了改进的选项和功能。 URL 生成使用[路由](xref:mvc/controllers/routing)，并且可以根据目标路径定义路由的方式生成参数并对参数编码。

页面的 URL 生成支持相对名称。 下表显示了 Pages/Customers/Create.cshtml 中不同的 `RedirectToPage` 参数选择的索引页：

| RedirectToPage(x)| 页面 |
| ----------------- | ------------ |
| RedirectToPage("/Index") | *Pages/Index* |
| RedirectToPage("./Index"); | *Pages/Customers/Index* |
| RedirectToPage("../Index") | *Pages/Index* |
| RedirectToPage("Index")  | *Pages/Customers/Index* |

`RedirectToPage("Index")`、`RedirectToPage("./Index")` 和 `RedirectToPage("../Index")` 是相对名称。 结合`RedirectToPage` 参数与当前页的路径来计算目标页面的名称。  <!-- Review: Original had The provided string is combined with the page name of the current page to compute the name of the destination page.  page name, not page path -->

构建结构复杂的站点时，相对名称链接很有用。 如果使用相对名称链接文件夹中的页面，则可以重命名该文件夹。 所有链接仍然有效（因为这些链接未包含此文件夹名称）。

若要重定向到不同[区域](xref:mvc/controllers/areas)中的页面，请指定区域：

```csharp
RedirectToPage("/Index", new { area = "Services" });
```

有关详细信息，请参阅 <xref:mvc/controllers/areas>。

## <a name="viewdata-attribute"></a>ViewData 特性

可以通过 [ViewDataAttribute](/dotnet/api/microsoft.aspnetcore.mvc.viewdataattribute) 将数据传递到页面。 控制器或 Razor Pages 模型上使用 `[ViewData]` 属性的属性将其值存储在 [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) 中并从此处进行加载。

在下面的示例中，`AboutModel` 包含使用 `[ViewData]` 标记的 `Title` 属性。 `Title` 属性设置为“关于”页面的标题：

```csharp
public class AboutModel : PageModel
{
    [ViewData]
    public string Title { get; } = "About";

    public void OnGet()
    {
    }
}
```

在“关于”页面中，以模型属性的形式访问 `Title` 属性：

```cshtml
<h1>@Model.Title</h1>
```

在布局中，从 ViewData 字典读取标题：

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

## <a name="tempdata"></a>TempData

ASP.NET 在[控制器](/dotnet/api/microsoft.aspnetcore.mvc.controller)上公开了 [TempData](/dotnet/api/microsoft.aspnetcore.mvc.controller.tempdata?view=aspnetcore-2.0#Microsoft_AspNetCore_Mvc_Controller_TempData) 属性。 此属性存储未读取的数据。 `Keep` 和 `Peek` 方法可用于检查数据，而不执行删除。 多个请求需要数据时，`TempData` 有助于进行重定向。

下面的代码使用 `TempData` 设置 `Message` 的值：

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateDot.cshtml.cs?highlight=10-11,25&name=snippet_Temp)]

Pages/Customers/Index.cshtml 文件中的以下标记使用 `TempData` 显示 `Message` 的值。

```cshtml
<h3>Msg: @Model.Message</h3>
```

Pages/Customers/Index.cshtml.cs 页面模型将 `[TempData]` 属性应用到 `Message` 属性。

```csharp
[TempData]
public string Message { get; set; }
```

有关详细信息，请参阅 [TempData](xref:fundamentals/app-state#tempdata)。

<a name="mhpp"></a>

## <a name="multiple-handlers-per-page"></a>针对一个页面的多个处理程序

以下页面使用 `asp-page-handler` 标记帮助程序为两个处理程序生成标记：

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?highlight=12-13)]

<!-- Review: the FormActionTagHelper applies to all <form /> elements on a Razor page, even when there's no `asp-` attribute   -->

前面示例中的窗体包含两个提交按钮，每个提交按钮均使用 `FormActionTagHelper` 提交到不同的 URL。 `asp-page-handler` 是 `asp-page` 的配套属性。 `asp-page-handler` 生成提交到页面定义的各个处理程序方法的 URL。 未指定 `asp-page`，因为示例已链接到当前页面。

页面模型：

[!code-csharp[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml.cs?highlight=20,32)]

前面的代码使用已命名处理程序方法。 已命名处理程序方法通过采用名称中 `On<HTTP Verb>` 之后及 `Async` 之前的文本（如果有）创建。 在前面的示例中，页面方法是 OnPost **JoinList** Async 和 OnPost **JoinListUC** Async。 删除 OnPost 和 Async 后，处理程序名称为 `JoinList` 和 `JoinListUC`。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateFATH.cshtml?range=12-13)]

使用前面的代码时，提交到 `OnPostJoinListAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH?handler=JoinList`。 提交到 `OnPostJoinListUCAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH?handler=JoinListUC`。

## <a name="custom-routes"></a>自定义路由

使用 `@page` 指令，执行以下操作：

* 指定页面的自定义路由。 例如，使用 `@page "/Some/Other/Path"` 将“关于”页面的路由设置为 `/Some/Other/Path`。
* 将段追加到页面的默认路由。 例如，可使用 `@page "item"` 将“item”段添加到页面的默认路由。
* 将参数追加到页面的默认路由。 例如，`@page "{id}"` 页面需要 ID 参数 `id`。

支持开头处以波形符 (`~`) 指定的相对于根目录的路径。 例如，`@page "~/Some/Other/Path"` 和 `@page "/Some/Other/Path"` 相同。

如果你不喜欢 URL 中的查询字符串 `?handler=JoinList`，请更改路由，将处理程序名称放在 URL 的路径部分。 可以通过在 `@page` 指令后面添加使用双引号括起来的路由模板来自定义路由。

[!code-cshtml[](index/sample/RazorPagesContacts2/Pages/Customers/CreateRoute.cshtml?highlight=1)]

使用前面的代码时，提交到 `OnPostJoinListAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH/JoinList`。 提交到 `OnPostJoinListUCAsync` 的 URL 路径为 `https://localhost:5001/Customers/CreateFATH/JoinListUC`。

`handler` 前面的 `?` 表示路由参数为可选。

## <a name="configuration-and-settings"></a>配置和设置

若要配置高级选项，请在 MVC 生成器上使用 `AddRazorPagesOptions` 扩展方法：

[!code-csharp[](index/sample/RazorPagesContacts/StartupAdvanced.cs?name=snippet_1)]

目前，可以使用 `RazorPagesOptions` 设置页面的根目录，或者为页面添加应用程序模型约定。 通过这种方式，我们在将来会实现更多扩展功能。

若要预编译视图，请参阅 [Razor 视图编译](xref:mvc/views/view-compilation)。

[下载或查看示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/razor-pages/index/sample).

请参阅 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)这篇文章以本文为基础撰写。

### <a name="specify-that-no-locrazor-pages-are-at-the-content-root"></a>指定 Razor Pages 位于内容根目录中

默认情况下，Razor Pages 位于 /Pages 目录的根位置。 向 [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) 添加 [WithRazorPagesAtContentRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvcbuilderextensions.withrazorpagesatcontentroot)，以指定 Razor Pages 位于应用的[根目录](xref:fundamentals/index#content-root) ([ContentRootPath](/dotnet/api/microsoft.aspnetcore.hosting.ihostingenvironment.contentrootpath))：

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesAtContentRoot();
```

### <a name="specify-that-no-locrazor-pages-are-at-a-custom-root-directory"></a>指定 Razor Pages 位于自定义根目录中

向 [AddMvc](/dotnet/api/microsoft.extensions.dependencyinjection.mvcservicecollectionextensions.addmvc#Microsoft_Extensions_DependencyInjection_MvcServiceCollectionExtensions_AddMvc_Microsoft_Extensions_DependencyInjection_IServiceCollection_) 添加 [WithRazorPagesRoot](/dotnet/api/microsoft.extensions.dependencyinjection.mvcrazorpagesmvccorebuilderextensions.withrazorpagesroot)，以指定 Razor Pages 位于应用中的自定义根目录（提供相对路径）：

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        ...
    })
    .WithRazorPagesRoot("/path/to/razor/pages");
```

## <a name="additional-resources"></a>其他资源

* [授权属性和 Razor Pages](xref:security/authorization/simple#aarp)
* <xref:index>
* <xref:mvc/views/razor>
* <xref:mvc/controllers/areas>
* <xref:tutorials/razor-pages/razor-pages-start>
* <xref:security/authorization/razor-pages-authorization>
* <xref:razor-pages/razor-pages-conventions>
* <xref:test/razor-pages-tests>
* <xref:mvc/views/partial>

::: moniker-end
