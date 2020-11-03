---
title: ASP.NET Core MVC 概述
author: ardalis
description: 了解 ASP.NET Core MVC 这一丰富框架如何使用“模型-视图-控制器”设计模式构建 Web 应用和 API。
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
uid: mvc/overview
ms.openlocfilehash: c55421efa1fd559f462549bfaceb794d4a9c35b8
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059307"
---
# <a name="overview-of-aspnet-core-mvc"></a>ASP.NET Core MVC 概述

作者：[Steve Smith](https://ardalis.com/)

ASP.NET Core MVC 是使用“模型-视图-控制器”设计模式构建 Web 应用和 API 的丰富框架。

## <a name="what-is-the-mvc-pattern"></a>什么是 MVC 模式？

模型-视图-控制器 (MVC) 体系结构模式将应用程序分成 3 个主要组件组：模型、视图和控制器。 此模式有助于实现[关注点分离](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns)。 使用此模式，用户请求被路由到控制器，后者负责使用模型来执行用户操作和/或检索查询结果。 控制器选择要显示给用户的视图，并为其提供所需的任何模型数据。

下图显示 3 个主要组件及其相互引用关系：

![MVC 模式](overview/_static/mvc.png)

这种责任划分有助于根据复杂性缩放应用程序，因为这更易于编码、调试和测试包含单一作业的某个组成部分（模型、视图或控制器）。 但这会加大更新、测试和调试代码的难度，该代码在这 3 个领域的两个或多个领域间存在依赖关系。 例如，用户界面逻辑的变更频率往往高于业务逻辑。 如果将表示代码和业务逻辑组合在单个对象中，则每次更改用户界面时都必须修改包含业务逻辑的对象。 这常常会引发错误，并且需要在每次进行细微的用户界面更改后重新测试业务逻辑。

> [!NOTE]
> 视图和控制器均依赖于模型。 但是，模型既不依赖于视图，也不依赖于控制器。 这是分离的一个关键优势。 这种分离允许模型独立于可视化展示进行构建和测试。

### <a name="model-responsibilities"></a>模型责任

MVC 应用程序的模型 (M) 表示应用程序和任何应由其执行的业务逻辑或操作的状态。 业务逻辑应与保持应用程序状态的任何实现逻辑一起封装在模型中。 强类型视图通常使用 ViewModel 类型，旨在包含要在该视图上显示的数据。 控制器从模型创建并填充 ViewModel 实例。

### <a name="view-responsibilities"></a>视图责任

视图 (V) 负责通过用户界面展示内容。 它们使用[ Razor 视图引擎](#razor-view-engine)在 HTML 标记中嵌入 .net 代码。 视图中应该有最小逻辑，并且其中的任何逻辑都必须与展示内容相关。 如果发现需要在视图文件中执行大量逻辑以显示复杂模型中的数据，请考虑使用 [View Component](views/view-components.md)、ViewModel 或视图模板来简化视图。

### <a name="controller-responsibilities"></a>控制器职责

控制器 (C) 是处理用户交互、使用模型并最终选择要呈现的视图的组件。 在 MVC 应用程序中，视图仅显示信息；控制器处理并响应用户输入和交互。 在 MVC 模式中，控制器是初始入口点，负责选择要使用的模型类型和要呈现的视图（因此得名 - 它控制应用如何响应给定请求）。

> [!NOTE]
> 控制器不应由于责任过多而变得过于复杂。 要阻止控制器逻辑变得过于复杂，请将业务逻辑推出控制器并推入域模型。

>[!TIP]
> 如果发现控制器操作经常执行相同类型的操作，可将这些常见操作移入[筛选器](#filters)。

## <a name="what-is-aspnet-core-mvc"></a>什么是 ASP.NET Core MVC

ASP.NET Core MVC 框架是轻量级、开源、高度可测试的演示框架，并针对 ASP.NET Core 进行了优化。

ASP.NET Core MVC 提供一种基于模式的方式，用于生成可彻底分开管理事务的动态网站。 它提供对标记的完全控制，支持 TDD 友好开发并使用最新的 Web 标准。

## <a name="features"></a>功能

ASP.NET Core MVC 包括以下功能：

* [路由](#routing)
* [模型绑定](#model-binding)
* [模型验证](#model-validation)
* [依赖项注入](../fundamentals/dependency-injection.md)
* [筛选器](#filters)
* [Areas](#areas)
* [Web API](#web-apis)
* [Testability](#testability)
* [Razor 查看引擎](#razor-view-engine)
* [强类型视图](#strongly-typed-views)
* [标记帮助程序](#tag-helpers)
* [查看组件](#view-components)

### <a name="routing"></a>路由

ASP.NET Core MVC 建立在 [ASP.NET Core 的路由](../fundamentals/routing.md)之上，是一个功能强大的 URL 映射组件，可用于生成具有易于理解和可搜索 URL 的应用程序。 它可让你定义适用于搜索引擎优化 (SEO) 和链接生成的应用程序 URL 命名模式，而不考虑如何组织 Web 服务器上的文件。 可以使用支持路由值约束、默认值和可选值的方便路由模板语法来定义路由。

通过基于约定的路由，可以全局定义应用程序接受的 URL 格式以及每个格式映射到给定控制器上特定操作方法的方式  。 接收传入请求时，路由引擎分析 URL 并将其匹配到定义的 URL 格式之一，然后调用关联的控制器操作方法。

```csharp
routes.MapRoute(name: "Default", template: "{controller=Home}/{action=Index}/{id?}");
```

借助属性路由，可以通过用定义应用程序路由的属性修饰控制器和操作来指定路由信息  。 这意味着路由定义位于与之相关联的控制器和操作旁。

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
    [HttpGet("{id}")]
    public IActionResult GetProduct(int id)
    {
      ...
    }
}
```

### <a name="model-binding"></a>模型绑定

ASP.NET Core MVC [模型绑定](models/model-binding.md)将客户端请求数据（窗体值、路由数据、查询字符串参数、HTTP 头）转换到控制器可以处理的对象中。 因此，控制器逻辑不必找出传入的请求数据；它只需具备作为其操作方法的参数的数据。

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null) { ... }
```

### <a name="model-validation"></a>模型验证

ASP.NET Core MVC 通过使用数据注释验证属性修饰模型对象来支持[验证](models/validation.md)。 验证属性在值发布到服务器前在客户端上进行检查，并在调用控制器操作前在服务器上进行检查。

```csharp
using System.ComponentModel.DataAnnotations;
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; }

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; }

    [Display(Name = "Remember me?")]
    public bool RememberMe { get; set; }
}
```

控制器操作：

```csharp
public async Task<IActionResult> Login(LoginViewModel model, string returnUrl = null)
{
    if (ModelState.IsValid)
    {
      // work with the model
    }
    // At this point, something failed, redisplay form
    return View(model);
}
```

框架处理客户端和服务器上的验证请求数据。 在模型类型上指定的验证逻辑作为非介入式注释添加到呈现的视图，并使用 [jQuery 验证](https://jqueryvalidation.org/)在浏览器中强制执行。

### <a name="dependency-injection"></a>依赖项注入

ASP.NET Core 内置有对[依赖关系注入 (DI)](../fundamentals/dependency-injection.md) 的支持。 在 ASP.NET Core MVC 中，[控制器](controllers/dependency-injection.md)可通过其构造函数请求所需服务，使其能够遵循 [Explicit Dependencies Principle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)（显式依赖关系原则）。

应用还可通过 `@inject` 指令使用[视图文件中的依赖关系注入](views/dependency-injection.md)：

```cshtml
@inject SomeService ServiceName

<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ServiceName.GetTitle</title>
</head>
<body>
    <h1>@ServiceName.GetTitle</h1>
</body>
</html>
```

### <a name="filters"></a>筛选器

[筛选器](controllers/filters.md)帮助开发者封装横切关注点，例如异常处理或授权。 筛选器允许操作方法运行自定义预处理和后处理逻辑，并且可以配置为在给定请求的执行管道内的特定点上运行。 筛选器可以作为属性应用于控制器或操作（也可以全局运行）。 此框架中包括多个筛选器（例如 `Authorize`）。 `[Authorize]` 是用于创建 MVC 授权筛选器的属性。

```csharp
[Authorize]
public class AccountController : Controller
```

### <a name="areas"></a>Areas

[区域](controllers/areas.md)提供将大型 ASP.NET Core MVC Web 应用分区为较小功能分组的方法。 区域是应用程序内的一个 MVC 结构。 在 MVC 项目中，模型、控制器和视图等逻辑组件保存在不同的文件夹中，MVC 使用命名约定来创建这些组件之间的关系。 对于大型应用，将应用分区为独立的高级功能区域可能更有利。 例如，具有多个业务单位的电子商务应用程序，如结帐、计费和搜索等。其中每个单位都有自己的逻辑组件视图、控制器和模型。

### <a name="web-apis"></a>Web API

除了作为生成网站的强大平台，ASP.NET Core MVC 还对生成 Web API 提供强大的支持。 可以生成可连接大量客户端（包括浏览器和移动设备）的服务。

该框架包括对 HTTP 内容协商的支持，后者有允许[设置数据格式](xref:web-api/advanced/formatting)为 JSON 或 XML 的内置支持。 编写[自定义格式化程序](xref:web-api/advanced/custom-formatters)以添加对自己格式的支持。

使用链接生成启用对超媒体的支持。 轻松启用对[跨域资源共享 (CORS)](https://www.w3.org/TR/cors/) 的支持，以便 Web API 可以跨多个 Web 应用程序共享。

### <a name="testability"></a>Testability

框架对界面和依赖项注入的使用非常适用于单元测试，并且该框架还包括使得[集成测试](xref:test/integration-tests)快速轻松的功能（例如 TestHost 和实体框架的 InMemory 提供程序）。 详细了解[如何测试控制器逻辑](controllers/testing.md)。

### <a name="no-locrazor-view-engine"></a>Razor 查看引擎

[ASP.NET CORE MVC 视图](views/overview.md)使用[ Razor 视图引擎](views/razor.md)呈现视图。 Razor 是一种紧凑、富于表现力且流畅的模板标记语言，用于使用 embedded c # 代码定义视图。 Razor 用于在服务器上动态生成 web 内容。 可以完全混合服务器代码与客户端内容和代码。

```cshtml
<ul>
    @for (int i = 0; i < 5; i++) {
        <li>List item @i</li>
    }
</ul>
```

使用 Razor 视图引擎可以定义 [布局](views/layout.md)、 [分部视图](views/partial.md) 和可替换部分。

### <a name="strongly-typed-views"></a>强类型视图

Razor MVC 中的视图可以基于模型进行强类型化。 控制器可以将强类型化的模型传递给视图，使视图具备类型检查和 IntelliSense 支持。

例如，以下视图呈现类型为 `IEnumerable<Product>` 的模型：

```cshtml
@model IEnumerable<Product>
<ul>
    @foreach (Product p in Model)
    {
        <li>@p.Name</li>
    }
</ul>
```

### <a name="tag-helpers"></a>标记帮助程序

[标记帮助](views/tag-helpers/intro.md) 程序使服务器端代码可以在文件中参与创建和呈现 HTML 元素 Razor 。 可以使用标记帮助程序定义自定义标记（例如 `<environment>`），或者修改现有标记的行为（例如 `<label>`）。 标记帮助程序基于元素名称及其属性绑定到特定的元素。 它们提供了服务器端呈现的优势，同时仍然保留了 HTML 编辑体验。

有多种常见任务（例如创建表单、链接，加载资产等）的内置标记帮助程序，公共 GitHub 存储库和 NuGet 包中甚至还有更多可用标记帮助程序。 标记帮助程序使用 C# 创建，基于元素名称、属性名称或父标记以 HTML 元素为目标。 例如，内置 LinkTagHelper 可以用来创建指向 `AccountsController` 的 `Login` 操作的链接：

```cshtml
<p>
    Thank you for confirming your email.
    Please <a asp-controller="Account" asp-action="Login">Click here to Log in</a>.
</p>
```

可以使用 `EnvironmentTagHelper` 在视图中包括基于运行时环境（例如开发、暂存或生产）的不同脚本（例如原始或缩减脚本）：

```cshtml
<environment names="Development">
    <script src="~/lib/jquery/dist/jquery.js"></script>
</environment>
<environment names="Staging,Production">
    <script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-2.1.4.min.js"
            asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
            asp-fallback-test="window.jQuery">
    </script>
</environment>
```

标记帮助程序提供 HTML 友好的开发体验，并提供丰富的 IntelliSense 环境用于创建 HTML 和 Razor 标记。 大多数内置标记帮助程序以现有 HTML 元素为目标，为该元素提供服务器端属性。

### <a name="view-components"></a>视图组件

通过[视图组件](views/view-components.md)可以包装呈现逻辑并在整个应用程序中重用它。 这些组件类似于[分部视图](views/partial.md)，但具有关联逻辑。

## <a name="compatibility-version"></a>兼容性版本

<xref:Microsoft.Extensions.DependencyInjection.MvcCoreMvcBuilderExtensions.SetCompatibilityVersion*> 方法允许应用选择加入或退出 ASP.NET Core MVC 2.1 或更高版本中引入的潜在中断行为变更。

有关详细信息，请参阅 <xref:mvc/compatibility-version>。

## <a name="additional-resources"></a>其他资源

* [MyTested AspNetCore-ASP.NET CORE mvc 的流畅测试库](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc)：强类型单元测试库，提供用于测试 Mvc 和 web API 应用的流畅界面。 （ *不由 Microsoft 进行支持或维护* 。）
* <xref:blazor/components/integrate-components-into-razor-pages-and-mvc-apps>
