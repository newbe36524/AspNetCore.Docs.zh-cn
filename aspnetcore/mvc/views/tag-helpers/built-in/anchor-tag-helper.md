---
title: ASP.NET Core 中的定位点标记帮助程序
author: pkellner
description: 了解 ASP.NET Core 定位点标记帮助程序属性以及每个属性在扩展 HTML 定位点标记的行为中所起的作用。
ms.author: scaddie
ms.custom: mvc
ms.date: 10/13/2019
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
uid: mvc/views/tag-helpers/builtin-th/anchor-tag-helper
ms.openlocfilehash: d39db59b0fc273fe4193a4864f302ecd3f4ad348
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060906"
---
# <a name="anchor-tag-helper-in-aspnet-core"></a>ASP.NET Core 中的定位点标记帮助程序

作者：[Peter Kellner](https://peterkellner.net) 和 [Scott Addie](https://github.com/scottaddie)

[定位点标记帮助程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper)可通过添加新属性来增强标准的 HTML 定位点 (`<a ... ></a>`) 标记。 按照约定，属性名称将使用前缀 `asp-`。 `asp-` 属性的值决定呈现的定位点元素的 `href` 属性值。

有关标记帮助程序的概述，请参阅 <xref:mvc/views/tag-helpers/intro>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/tag-helpers/built-in/samples)（[如何下载](xref:index#how-to-download-a-sample)）

本文档中的示例均使用 SpeakerController  ：

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?name=snippet_SpeakerController)]

## <a name="anchor-tag-helper-attributes"></a>定位点标记帮助程序属性

### <a name="asp-controller"></a>asp-controller

[asp-controller](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Controller*) 属性可分配用于生成 URL 的控制器。 下面的标记列出了所有发言人：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspController)]

生成的 HTML：

```html
<a href="/Speaker">All Speakers</a>
```

如果指定了 `asp-controller` 属性，而未指定 `asp-action` 属性，则默认的 `asp-action` 值为与当前正在执行的视图关联的控制器操作。 如果前面的标记中省略了 `asp-action`，并在 HomeController 的索引视图 (/Home) 中使用了定位点标记帮助程序，则生成的 HTML 为  ：

```html
<a href="/Home">All Speakers</a>
```

### <a name="asp-action"></a>asp-action

[asp-action](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Action*) 属性值表示生成的 `href` 属性中包含的控制器操作名称。 下面的标记可将生成的 `href` 属性值设置为发言人评估页：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAction)]

生成的 HTML：

```html
<a href="/Speaker/Evaluations">Speaker Evaluations</a>
```

如果未指定 `asp-controller` 属性，则使用默认控制器，该控制器调用执行当前视图的视图。

如果 `asp-action` 属性值为 `Index`，则不向 URL 追加任何操作，从而导致调用默认的 `Index` 操作。 `asp-controller` 引用的控制器中必须存在指定的（或默认的）操作。

### <a name="asp-route-value"></a>asp-route-{value}

[asp-route-{value}](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.RouteValues*) 属性可实现通配符路由前缀。 占用 `{value}` 占位符的所有值都解释为潜在的路由参数。 如果找不到默认路由，则将此路由前缀作为请求参数和值追加到生成的 `href` 属性。 否则，将在路由模板中替换它。

考虑以下控制器操作：

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/BuiltInTagController.cs?name=snippet_AnchorTagHelperAction)]

在 Startup.Configure 中定义默认路由模板  ：

[!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_UseMvc&highlight=8-10)]

MVC 视图使用操作提供的模型，如下所示：

```cshtml
@model Speaker
<!DOCTYPE html>
<html>
<body>
    <a asp-controller="Speaker"
       asp-action="Detail" 
       asp-route-id="@Model.SpeakerId">SpeakerId: @Model.SpeakerId</a>
</body>
</html>
```

默认路由的 `{id?}` 占位符得以匹配。 生成的 HTML：

```html
<a href="/Speaker/Detail/12">SpeakerId: 12</a>
```

假设路由前缀不属于匹配路由模板的一部分，如下面的 MVC 视图所示：

```cshtml
@model Speaker
<!DOCTYPE html>
<html>
<body>
    <a asp-controller="Speaker"
       asp-action="Detail"
       asp-route-speakerid="@Model.SpeakerId">SpeakerId: @Model.SpeakerId</a>
<body>
</html>
```

生成以下 HTML，因为匹配的路由中未找到 `speakerid`：

```html
<a href="/Speaker/Detail?speakerid=12">SpeakerId: 12</a>
```

如果 `asp-controller` 或 `asp-action` 均未指定，则会执行与 `asp-route` 属性中相同的默认处理。

### <a name="asp-route"></a>asp-route

[asp-route](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Route*) 属性用于创建直接链接到命名路由的 URL。 使用[路由属性](xref:mvc/controllers/routing#attribute-routing)，路由可以按 `SpeakerController` 中所示进行命名并用于其 `Evaluations` 操作：

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?range=22-24)]

在下列标记中，`asp-route` 属性引用命名路由：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspRoute)]

定位点标记帮助程序使用 URL /Speaker/Evaluations 生成直接指向该控制器操作的路由  。 生成的 HTML：

```html
<a href="/Speaker/Evaluations">Speaker Evaluations</a>
```

如果除了 `asp-route`，还指定了 `asp-controller` 或 `asp-action`，则可能不会生成预期的路由。 为了避免发生路由冲突，不应将 `asp-route` 与 `asp-controller` 和 `asp-action` 属性结合使用。

### <a name="asp-all-route-data"></a>asp-all-route-data

[asp-all-route-data](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.RouteValues*) 属性支持创建键值对字典。 键是参数名称，值是参数值。

在下面的示例中，将对字典进行初始化并将其传递给 Razor 视图。 或者，也可以使用模型传入数据。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAllRouteData)]

前面的代码生成以下 HTML：

```html
<a href="/Speaker/EvaluationsCurrent?speakerId=11&currentYear=true">Speaker Evaluations</a>
```

平展 `asp-all-route-data` 字典，以生成满足重载 `Evaluations` 操作要求的查询字符串：

[!code-csharp[](samples/TagHelpersBuiltIn/Controllers/SpeakerController.cs?range=26-30)]

如果字典中的任何键匹配路由参数，则将根据需要在路由中替换这些值。 其他不匹配的值作为请求参数生成。

### <a name="asp-fragment"></a>asp-fragment

[asp-fragment](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Fragment*) 属性可定义要追加到 URL 的 URL 片段。 定位点标记帮助程序添加哈希字符 (#)。 请考虑以下标记：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspFragment)]

生成的 HTML：

```html
<a href="/Speaker/Evaluations#SpeakerEvaluations">Speaker Evaluations</a>
```

生成客户端应用时，哈希标记很有用。 它们可用于在 JavaScript 中轻松地执行标记和搜索等操作。

### <a name="asp-area"></a>asp-area

[asp-area](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Area*) 属性可设置用来设置相应路由的区域名称。 以下示例展示了 `asp-area` 属性如何导致重新映射路由。

#### <a name="usage-in-no-locrazor-pages"></a>页面使用量 Razor

Razor ASP.NET Core 2.1 或更高版本中支持页区域。

考虑以下目录层次结构：

* **{项目名称}**
  * **wwwroot**
  * **Areas**
    * **会话**
      * **页**
        * *\_Viewstart.cshtml*
        * *Index.cshtml*
        * *Index.cshtml.cs*
  * **页**

用于引用 *会话* 区域 *索引* Razor 页的标记为：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspAreaRazorPages)]

生成的 HTML：

```html
<a href="/Sessions">View Sessions</a>
```

> [!TIP]
> 若要在页面应用中支持区域 Razor ，请在中执行以下 `Startup.ConfigureServices` 操作之一：
>
> * 将[兼容性版本](xref:mvc/compatibility-version)设置为 2.1 或更高版本。
> * 将 AllowAreas 属性设置为[ Razor PagesOptions](xref:Microsoft.AspNetCore.Mvc.RazorPages.RazorPagesOptions.AllowAreas*) `true` ：
>
>   [!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_AllowAreas)]

#### <a name="usage-in-mvc"></a>MVC 中的使用情况

考虑以下目录层次结构：

* **{项目名称}**
  * **wwwroot**
  * **Areas**
    * **博客**
      * **Controllers**
        * HomeController.cs 
      * **Views**
        * **主页**
          * *AboutBlog.cshtml*
          * *Index.cshtml*
        * *\_Viewstart.cshtml*
  * **Controllers**

如果将 `asp-area` 设置为 “Blogs”，则会为此定位点标记的关联控制器和视图的路由添加目录 Areas/Blogs 作为前缀  。 用于引用 AboutBlog 视图的标记是  ：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspArea)]

生成的 HTML：

```html
<a href="/Blogs/Home/AboutBlog">About Blog</a>
```

> [!TIP]
> 若要在 MVC 应用中支持区域，路由模板必须包含对该区域（如果存在）的引用。 该模板由 `routes.MapRoute` *Startup.Configu)* 中的方法调用的第二个参数表示：
>
> [!code-csharp[](samples/TagHelpersBuiltIn/Startup.cs?name=snippet_UseMvc&highlight=5)]

### <a name="asp-protocol"></a>asp-protocol

[asp-protocol](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Protocol*) 属性用于在 URL 中指定协议（比如 `https`）。 例如： 。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspProtocol)]

生成的 HTML：

```html
<a href="https://localhost/Home/About">About</a>
```

示例中的主机名为 localhost。 生成 URL 时，定位点标记帮助程序会使用网站的公共域。

### <a name="asp-host"></a>asp-host

[asp-host](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Host*) 属性用于在 URL 中指定主机名。 例如： 。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspHost)]

生成的 HTML：

```html
<a href="https://microsoft.com/Home/About">About</a>
```

### <a name="asp-page"></a>asp-page

[Asp 页](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.Page*)属性用于 Razor 页面。 使用它向特定页设置定位点标记的 `href` 属性值。 通过在页面名称前面使用正斜杠 (“/”) 作为前缀，可创建 URL。

以下示例指向 "与会者" Razor 页：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPage)]

生成的 HTML：

```html
<a href="/Attendee">All Attendees</a>
```

`asp-page` 属性与 `asp-route`、`asp-controller` 和 `asp-action` 属性互斥。 但是，`asp-page` 可与 `asp-route-{value}` 结合使用以控制路由，如以下标记所示：

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPageAspRouteId)]

生成的 HTML：

```html
<a href="/Attendee?attendeeid=10">View Attendee</a>
```

### <a name="asp-page-handler"></a>asp-page-handler

[Asp 页处理程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.AnchorTagHelper.PageHandler*)属性与页一起使用 Razor 。 它用于链接到特定的页处理程序。

请考虑以下页处理程序：

[!code-csharp[](samples/TagHelpersBuiltIn/Pages/Attendee.cshtml.cs?name=snippet_OnGetProfileHandler)]

页模型的关联标记链接到 `OnGetProfile` 页处理程序。 注意，`asp-page-handler` 属性值中省略了页处理程序方法名称的 `On<Verb>` 前缀。 如果方法是异步的，也省略 `Async` 后缀。

[!code-cshtml[](samples/TagHelpersBuiltIn/Views/Home/Index.cshtml?name=snippet_AspPageHandler)]

生成的 HTML：

```html
<a href="/Attendee?attendeeid=12&handler=Profile">Attendee Profile</a>
```

## <a name="additional-resources"></a>其他资源

* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
