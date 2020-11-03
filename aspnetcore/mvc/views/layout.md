---
title: ASP.NET Core 中的布局
author: ardalis
description: 了解如何在 ASP.NET Core 应用中呈现视图之前，使用通用布局、共享指令和运行常见代码。
ms.author: riande
ms.date: 07/30/2019
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
uid: mvc/views/layout
ms.openlocfilehash: 502df268e7f5f33acfffccd5ec0bd65267fa12da
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060971"
---
# <a name="layout-in-aspnet-core"></a>ASP.NET Core 中的布局

作者：[Steve Smith](https://ardalis.com/) 和 [Dave Brock](https://twitter.com/daveabrock)

页面和视图经常共享视觉对象和程序元素。 本文演示了以下内容的操作方法：

* 使用通用布局。
* 共享指令。
* 在呈现页面或视图之前运行通用代码。

本文档讨论了两种不同方法 ASP.NET Core MVC： Razor 页面和控制器与视图的布局。 在本主题中，差异很小：

* Razor 页面位于 " *页面* " 文件夹中。
* 具有视图的控制器使用视图的“视图”文件夹。 

## <a name="what-is-a-layout"></a>什么是布局

大多数 Web 应用都有一个通用布局，可在页面间切换时为用户提供一致体验。 该布局通常包括应用标头、导航或菜单元素以及页脚等常见的用户界面元素。

![页面布局示例](layout/_static/page-layout.png)

应用中的许多页面也经常使用常见的 HTML 结构，如脚本和样式表。 所有这些共享元素都可以在 *布局* 文件中定义，该文件随后可由应用中使用的任何视图引用。 布局可减少视图中的重复代码。

按照约定，ASP.NET Core 应用的默认布局名为 _Layout.cshtml。  使用模板创建的新 ASP.NET Core 项目的布局文件为：

* Razor 页面： *pages/Shared/_Layout cshtml*

  ![解决方案资源管理器中的页面文件夹](layout/_static/rp-web-project-views.png)

* 包含视图的控制器：Views/Shared/_Layout.cshtml 

  ![解决方案资源管理器中的视图文件夹](layout/_static/mvc-web-project-views.png)

布局定义应用中的视图的最高级别模板。 应用不需要布局。 应用可以定义多个布局，其中不同的视图指定不同的布局。

下面的代码演示了使用控制器和视图创建的项目模板的布局文件：

[!code-cshtml[](~/common/samples/WebApplication1/Views/Shared/_Layout.cshtml?highlight=44,72)]

## <a name="specifying-a-layout"></a>指定布局

Razor 视图具有 `Layout` 属性。 单个视图通过设置此属性来指定布局：

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewStart.cshtml?highlight=2)]

指定的布局可以使用完整路径（例如 /Pages/Shared/_Layout.cshtml 或 /Views/Shared/_Layout.cshtml）或部分名称（示例：`_Layout`）。  如果提供了部分名称， Razor 视图引擎将使用其标准发现进程搜索布局文件。 首先搜索处理程序方法（或控制器）所在的文件夹，然后搜索 Shared 文件夹。  此发现过程与用于发现[分部视图](xref:mvc/views/partial#partial-view-discovery)的过程相同。

默认情况下，每个布局必须调用 `RenderBody`。 无论在何处调用 `RenderBody`，都会呈现视图的内容。

<a name="layout-sections-label"></a>
<!-- https://stackoverflow.com/questions/23327578 -->
### <a name="sections"></a>部分

布局可以通过调用 `RenderSection` 来选择引用一个或多个节  。 节提供一种方法来组织某些页面元素应当放置的位置。 每次调用 `RenderSection` 时都可指定该部分是必需还是可选：

```html
<script type="text/javascript" src="~/scripts/global.js"></script>

@RenderSection("Scripts", required: false)
```

如果找不到所需的节，将引发异常。 单个视图使用语法指定要在节中呈现的内容 `@section` Razor 。 如果某个页面或视图定义了一个部分，则必须呈现该部分（否则将发生错误）。

`@section`页面视图中的示例定义 Razor ：

```html
@section Scripts {
     <script type="text/javascript" src="~/scripts/main.js"></script>
}
```

在上面的代码中，scripts / main.js 添加到了页面或视图的 `scripts` 部分  。 相同应用中的其他页面或视图可能不需要此脚本且不会定义脚本部分。

以下标记使用 

```html
@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

上述标记是由[基架 Identity ](xref:security/authentication/scaffold-identity)生成的。

在页面或视图中定义的部分仅在其即时布局页面中可用。 不能从部分、视图组件或视图系统的其他部分引用它们。

### <a name="ignoring-sections"></a>忽略节

默认情况下，必须由布局页面呈现内容页中的正文和所有节。 Razor视图引擎通过跟踪正文和每个部分是否已呈现来强制执行此方法。

要让视图引擎忽略正文或节，请调用 `IgnoreBody` 和 `IgnoreSection` 方法。

页面中的正文和每个节 Razor 必须呈现或忽略。

<a name="viewimports"></a>

## <a name="importing-shared-directives"></a>导入共享指令

视图和页面可以使用 Razor 指令导入命名空间和使用 [依赖关系注入](dependency-injection.md)。 由多个视图共享的指令可以在通用 _ViewImports.cshtml 文件中进行指定。  `_ViewImports` 文件支持以下指令：

* `@addTagHelper`
* `@removeTagHelper`
* `@tagHelperPrefix`
* `@using`
* `@model`
* `@inherits`
* `@inject`

文件不支持其他 Razor 功能，例如函数和节定义。

示例 `_ViewImports.cshtml` 文件：

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewImports.cshtml)]

ASP.NET Core MVC 应用的 _ViewImports.cshtml 文件通常放在“页面”（或“视图”）文件夹中。  _ViewImports.cshtml 文件可以放在任何文件夹中，在这种情况下，它只会应用于该文件夹及其子文件夹中的页面或视图。  从根级别开始处理 `_ViewImports` 文件，然后处理在页面或视图本身的位置之前的每个文件夹。 可以在文件夹级别覆盖根级别指定的 `_ViewImports` 设置。

例如，假设：

* 根级别 _ViewImports.cshtml 文件包含 `@model MyModel1` 和 `@addTagHelper *, MyTagHelper1`。 
* 子文件夹 _ViewImports.cshtml 文件包含 `@model MyModel2` 和 `@addTagHelper *, MyTagHelper2`。 

子文件夹中的页面和视图将有权访问标记帮助程序和 `MyModel2` 模型。

如果在文件层次结构中找到多个 _ViewImports.cshtml 文件，指令的合并行为是： 

* `@addTagHelper``@removeTagHelper`：按顺序全部运行
* `@tagHelperPrefix`：最接近视图的文件会替代任何其他文件
* `@model`：最接近视图的文件会替代任何其他文件
* `@inherits`：最接近视图的文件会替代任何其他文件
* `@using`：全部包括在内；忽略重复项
* `@inject`：针对每个属性，最接近视图的属性会替代具有相同属性名的任何其他属性

<a name="viewstart"></a>

## <a name="running-code-before-each-view"></a>在呈现每个视图之前运行代码

需要在每个视图或页面之前运行的代码应置于 _ViewStart.cshtml 文件中。  按照约定，_ViewStart.cshtml 文件位于“页面”（或“视图”）文件夹。  在呈现每个完整视图（不是布局，也不是分部视图）之前运行 _ViewStart.cshtml 中列出的语句。  与  如果在视图或页面文件夹中定义了 _ViewStart.cshtml 文件，则它将在页面（或视图）的根目录中定义的文件之后运行）文件夹（如果有）。 

一个示例 _ViewStart.cshtml 文件： 

[!code-cshtml[](../../common/samples/WebApplication1/Views/_ViewStart.cshtml)]

上述文件指定所有视图都将使用 _Layout.cshtml 布局。 

_ViewStart.cshtml 和 _ViewImports.cshtml 通常不置于 /Pages/Shared（或 /Views/Shared）文件夹中。  这些应用级别版本的文件应直接放置在 /Pages（或 /Views）文件夹中。 
