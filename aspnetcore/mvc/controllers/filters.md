---
title: ASP.NET Core 中的筛选器
author: Rick-Anderson
description: 了解筛选器的工作原理以及如何在 ASP.NET Core 中使用它们。
ms.author: riande
ms.custom: mvc
ms.date: 02/04/2020
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
uid: mvc/controllers/filters
ms.openlocfilehash: ecb4de3439656eb56507b920db704048d8f96759
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93058501"
---
# <a name="filters-in-aspnet-core"></a>ASP.NET Core 中的筛选器

::: moniker range=">= aspnetcore-3.0"

作者：[Kirk Larkin](https://github.com/serpent5)、[Rick Anderson](https://twitter.com/RickAndMSFT)、[Tom Dykstra](https://github.com/tdykstra/) 和 [Steve Smith](https://ardalis.com/)

通过使用 ASP.NET Core 中的筛选器，可在请求处理管道中的特定阶段之前或之后运行代码。 

内置筛选器处理任务，例如：

* 授权（防止用户访问未获授权的资源）。
* 响应缓存（对请求管道进行短路出路，以便返回缓存的响应）。

可以创建自定义筛选器，用于处理横切关注点。 横切关注点的示例包括错误处理、缓存、配置、授权和日志记录。  筛选器可以避免复制代码。 例如，错误处理异常筛选器可以合并错误处理。

本文档适用于 Razor 具有视图的页面、API 控制器和控制器。 筛选器不直接与[ Razor 组件](xref:blazor/components/index)一起使用。 筛选器只能在以下情况下间接影响组件：

* 该组件嵌入在页面或视图中。
* 页面或控制器/视图使用此筛选器。

[查看或下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample)（[如何下载](xref:index#how-to-download-a-sample)）。

## <a name="how-filters-work"></a>筛选器的工作原理

筛选器在 ASP.NET Core 操作调用管道  （有时称为筛选器管道  ）内运行。 筛选器管道在 ASP.NET Core 选择了要执行的操作之后运行。

![请求通过其他中间件、路由中间件、操作选择和操作调用管道进行处理。 请求处理继续往回通过操作选择、路由中间件和各种其他中间件，变成发送到客户端的响应。](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a>筛选器类型

每种筛选器类型都在筛选器管道中的不同阶段执行：

* [授权筛选器](#authorization-filters)最先运行，用于确定是否已针对请求为用户授权。 如果请求未获授权，授权筛选器可以让管道短路。

* [资源筛选器](#resource-filters)：

  * 授权后运行。  
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> 在筛选器管道的其余阶段之前运行代码。 例如，`OnResourceExecuting` 在模型绑定之前运行代码。
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> 在管道的其余阶段完成之后运行代码。

* [操作筛选器](#action-filters)：

  * 在调用操作方法之前和之后立即运行代码。
  * 可以更改传递到操作中的参数。
  * 可以更改从操作返回的结果。
  * 页面 **不** 支持 Razor 。

* [异常筛选器](#exception-filters)在向响应正文写入任何内容之前，对未经处理的异常应用全局策略。

* [结果筛选器](#result-filters)在执行操作结果之前和之后立即运行代码。 仅当操作方法成功执行时，它们才会运行。 对于必须围绕视图或格式化程序的执行的逻辑，它们很有用。

下图展示了筛选器类型在筛选器管道中的交互方式。

![请求通过授权过滤器、资源过滤器、模型绑定、操作过滤器、操作执行和操作结果转换、异常过滤器、结果过滤器和结果执行进行处理。 返回时，请求仅由结果过滤器和资源过滤器进行处理，变成发送到客户端的响应。](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a>实现

筛选器通过不同的接口定义支持同步和异步实现。

同步筛选器在其管道阶段之前和之后运行代码。 例如，<xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*> 在调用操作方法之前调用。 <xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*> 在操作方法返回之后调用。

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

在上面的代码中， [MyDebug](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs) 是 [示例下载](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/filters/3.1sample/FiltersSample/Helper/MyDebug.cs)中的实用工具函数。

异步筛选器定义 `On-Stage-ExecutionAsync` 方法。 例如，<xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*>：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

在前面的代码中，`SampleAsyncActionFilter` 具有执行操作方法的 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`)。

### <a name="multiple-filter-stages"></a>多个筛选器阶段

可以在单个类中实现多个筛选器阶段的接口。 例如，<xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> 类可实现：

* 同步：<xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter>
* 异步：<xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
* <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter>

筛选器接口的同步和异步版本任意实现一个，而不是同时实现 。 运行时会先查看筛选器是否实现了异步接口，如果是，则调用该接口。 如果不是，则调用同步接口的方法。 如果在一个类中同时实现异步和同步接口，则仅调用异步方法。 当使用抽象类（如 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> ）时，仅重写每个筛选器类型的同步方法或异步方法。

### <a name="built-in-filter-attributes"></a>内置筛选器属性

ASP.NET Core 包含许多可子类化和自定义的基于属性的内置筛选器。 例如，以下结果筛选器会向响应添加标头：

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

通过使用属性，筛选器可接收参数，如前面的示例所示。 将 `AddHeaderAttribute` 添加到控制器或操作方法，并指定 HTTP 标头的名称和值：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

使用 [浏览器开发人员工具](https://developer.mozilla.org/docs/Learn/Common_questions/What_are_browser_developer_tools) 等工具来检查标头。 在响应标头下，将显示 `author: Rick Anderson`。

以下代码实现了 `ActionFilterAttribute`：

* 从配置系统读取标题和名称。 与前面的示例不同，以下代码不需要将筛选器参数添加到代码中。
* 将标题和名称添加到响应标头。

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MyActionFilterAttribute.cs?name=snippet)]

使用[选项模式](xref:fundamentals/configuration/options)从[配置系统](xref:fundamentals/configuration/index)中提供配置选项。 例如，从 *appsettings.json* 文件：

[!code-json[](filters/3.1sample/FiltersSample/appsettings.json)]

在 `StartUp.ConfigureServices` 中：

* `PositionOptions` 类已通过 `"Position"` 配置区域添加到服务容器。
* `MyActionFilterAttribute` 已添加到服务容器。

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupAF.cs?name=snippet)]

以下代码显示 `PositionOptions` 类：

[!code-csharp[](filters/3.1sample/FiltersSample/Helper/PositionOptions.cs?name=snippet)]

以下代码将 `MyActionFilterAttribute` 应用于 `Index2` 方法：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet2&highlight=9)]

在 **Response Headers** `author: Rick Anderson` `Editor: Joe Smith` 调用终结点时，将显示 "响应标头"、和 `Sample/Index2` 。

下面的代码将 `MyActionFilterAttribute` 和应用于 `AddHeaderAttribute` Razor 页面：

[!code-csharp[](filters/3.1sample/FiltersSample/Pages/Movies/Index.cshtml.cs?name=snippet)]

筛选器不能应用于 Razor 页面处理程序方法。 它们可以应用于 Razor 页面模型或全局应用。

多种筛选器接口具有相应属性，这些属性可用作自定义实现的基类。

筛选器属性：

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a>筛选器作用域和执行顺序

可以将筛选器添加到管道中的以下三个 *范围* 之一：

* 在控制器操作上使用属性。 筛选器属性不能应用于 Razor 页面处理程序方法。
* 在控制器或页上使用特性 Razor 。
* 针对所有控制器、操作和页面全局 Razor 显示，如以下代码所示：

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder.cs?name=snippet)]

### <a name="default-order-of-execution"></a>默认执行顺序

当管道的某个特定阶段有多个筛选器时，作用域可确定筛选器执行的默认顺序。  全局筛选器涵盖类筛选器，类筛选器又涵盖方法筛选器。

在筛选器嵌套模式下，筛选器的 after  代码会按照与 before  代码相反的顺序运行。 筛选器序列：

* 全局筛选器的 before  代码。
  * 控制器 *before* 和 Razor 页面筛选器的前代码。
    * 操作方法筛选器的 before  代码。
    * 操作方法筛选器的 after  代码。
  * 控制器 *after* 和 Razor 页面筛选器后的代码。
* 全局筛选器的 after  代码。
  
下面的示例阐释了为同步操作筛选器调用筛选器方法的顺序。

| 序列 | 筛选器作用域 | 筛选器方法 |
|:--------:|:------------:|:-------------:|
| 1 | 全球 | `OnActionExecuting` |
| 2 | 控制器或 Razor 页面| `OnActionExecuting` |
| 3 | 方法 | `OnActionExecuting` |
| 4 | 方法 | `OnActionExecuted` |
| 5 | 控制器或 Razor 页面 | `OnActionExecuted` |
| 6 | 全球 | `OnActionExecuted` |

### <a name="controller-level-filters"></a>控制器级别筛选器

继承自 <xref:Microsoft.AspNetCore.Mvc.Controller> 基类的每个控制器包括 [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*)、[Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) 和 [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted` 方法。 这些方法：

* 覆盖为给定操作运行的筛选器。
* `OnActionExecuting` 在所有操作筛选器之前调用。
* `OnActionExecuted` 在所有操作筛选器之后调用。
* `OnActionExecutionAsync` 在所有操作筛选器之前调用。 `next` 之后的筛选器中的代码在操作方法之后运行。

例如，在下载示例中，启动时全局应用 `MySampleActionFilter`。

`TestController`：

* 将 `SampleActionFilterAttribute` (`[SampleActionFilter]`) 应用于 `FilterTest2` 操作。
* 重写 `OnActionExecuting` 和 `OnActionExecuted`。

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

[!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

<!-- test via  webBuilder.UseStartup<Startup>(); -->

导航到 `https://localhost:5001/Test/FilterTest2` 运行以下代码：

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

控制器级别筛选器将 [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) 属性设置为 `int.MinValue`。 控制器级别筛选器无法设置为在将筛选器应用于方法之后运行  。 在下一节对 Order 进行了介绍。

有关 Razor 页面，请 [参阅 Razor 通过重写筛选器方法实现页面筛选器](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods)。

### <a name="overriding-the-default-order"></a>重写默认顺序

可以通过实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter> 来重写默认执行序列。 `IOrderedFilter` 公开了 <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order> 属性来确定执行顺序，该属性优先于作用域。 具有较低的 `Order` 值的筛选器：

* 在具有较高的 `Order` 值的筛选器之前运行 before  代码。
* 在具有较高的 `Order` 值的筛选器之后运行 after  代码。

使用构造函数参数设置了 `Order` 属性：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test3Controller.cs?name=snippet)]

请考虑以下控制器中的两个操作筛选器：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test2Controller.cs?name=snippet)]

在 `StartUp.ConfigureServices` 中添加了全局筛选器：

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder.cs?name=snippet)]

3 个筛选器按下列顺序运行：

* `Test2Controller.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `MyAction2FilterAttribute.OnActionExecuting`
      * `Test2Controller.FilterTest2`
    * `MyAction2FilterAttribute.OnResultExecuting`
  * `MySampleActionFilter.OnActionExecuted`
* `Test2Controller.OnActionExecuted`

在确定筛选器的运行顺序时，`Order` 属性重写作用域。 先按顺序对筛选器排序，然后使用作用域消除并列问题。 所有内置筛选器实现 `IOrderedFilter` 并将默认 `Order` 值设为 0。 如前所述，控制器级别筛选器将 [Order](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/Filters/ControllerActionFilter.cs#L15-L17) 属性设置为 `int.MinValue`。对于内置筛选器，作用域会确定顺序，除非将 `Order` 设为非零值。

在前面的代码中，`MySampleActionFilter` 具有全局作用域，因此它在具有控制器作用域的 `MyAction2FilterAttribute` 之前运行。 若要首先运行 `MyAction2FilterAttribute`，请将顺序设置为 `int.MinValue`：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/Test2Controller.cs?name=snippet2)]

若要首先运行全局筛选器 `MySampleActionFilter`，请将 `Order` 设置为 `int.MinValue`：

[!code-csharp[](./filters/3.1sample/FiltersSample/StartupOrder2.cs?name=snippet&highlight=6)]

## <a name="cancellation-and-short-circuiting"></a>取消和设置短路

通过设置提供给筛选器方法的 <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> 参数上的 <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> 属性，可以使筛选器管道短路。 例如，以下资源筛选器将阻止执行管道的其余阶段：

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

在下面的代码中，`ShortCircuitingResourceFilter` 和 `AddHeader` 筛选器都以 `SomeResource` 操作方法为目标。 `ShortCircuitingResourceFilter`：

* 先运行，因为它是资源筛选器且 `AddHeader` 是操作筛选器。
* 对管道的其余部分进行短路处理。

这样 `AddHeader` 筛选器就不会为 `SomeResource` 操作运行。 如果这两个筛选器都应用于操作方法级别，只要 `ShortCircuitingResourceFilter` 先运行，此行为就不会变。 先运行 `ShortCircuitingResourceFilter`（考虑到它的筛选器类型），或显式使用 `Order` 属性。

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

## <a name="dependency-injection"></a>依赖项注入

可按类型或实例添加筛选器。 如果添加实例，该实例将用于每个请求。 如果添加类型，则将激活该类型。 激活类型的筛选器意味着：

* 将为每个请求创建一个实例。
* [依赖关系注入](xref:fundamentals/dependency-injection) (DI) 将填充所有构造函数依赖项。

如果将筛选器作为属性实现并直接添加到控制器类或操作方法中，则该筛选器不能由[依赖关系注入](xref:fundamentals/dependency-injection) (DI) 提供构造函数依赖项。 无法由 DI 提供构造函数依赖项，因为：

* 属性在应用时必须提供自己的构造函数参数。 
* 这是属性工作原理上的限制。

以下筛选器支持从 DI 提供的构造函数依赖项：

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* 在属性上实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>。

可以将前面的筛选器应用于控制器或操作方法：

可以从 DI 获取记录器。 但是，避免创建和使用筛选器仅用于日志记录。 [内置框架日志记录](xref:fundamentals/logging/index)通常提供日志记录所需的内容。 添加到筛选器的日志记录：

* 应重点关注业务域问题或特定于筛选器的行为。
* 不应记录操作或其他框架事件  。 内置筛选器记录操作和框架事件。

### <a name="servicefilterattribute"></a>ServiceFilterAttribute

在 `ConfigureServices` 中注册服务筛选器实现类型。 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> 可从 DI 检索筛选器实例。

以下代码显示 `AddHeaderResultServiceFilter`：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

在以下代码中，`AddHeaderResultServiceFilter` 将添加到 DI 容器中：

[!code-csharp[](./filters/3.1sample/FiltersSample/Startup.cs?name=snippet&highlight=4)]

在以下代码中，`ServiceFilter` 属性将从 DI 中检索 `AddHeaderResultServiceFilter` 筛选器的实例：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

使用 `ServiceFilterAttribute` 时，[ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable) 设置：

* 提供以下提示：筛选器实例可能在其创建的请求范围之外被重用。  ASP.NET Core 运行时不保证：

  * 将创建筛选器的单一实例。
  * 稍后不会从 DI 容器重新请求筛选器。

* 不应与依赖于生命周期不同于单一实例的服务的筛选器一起使用。

 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> 可实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>。 `IFilterFactory` 公开用于创建 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> 实例的 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> 方法。 `CreateInstance` 从 DI 中加载指定的类型。

### <a name="typefilterattribute"></a>TypeFilterAttribute

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> 与 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> 类似，但不会直接从 DI 容器解析其类型。 它使用 <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName> 对类型进行实例化。

因为不会直接从 DI 容器解析 `TypeFilterAttribute` 类型：

* 使用 `TypeFilterAttribute` 引用的类型不需要注册在 DI 容器中。  它们具备由 DI 容器实现的依赖项。
* `TypeFilterAttribute` 可以选择为类型接受构造函数参数。

使用 `TypeFilterAttribute` 时，[TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable) 设置：
* 提供提示：筛选器实例可能在其创建的请求范围之外被重用。  ASP.NET Core 运行时不保证将创建筛选器的单一实例。

* 不应与依赖于生命周期不同于单一实例的服务的筛选器一起使用。

下面的示例演示如何使用 `TypeFilterAttribute` 将参数传递到类型：

[!code-csharp[](filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a>授权筛选器

授权筛选器：

* 是筛选器管道中运行的第一个筛选器。
* 控制对操作方法的访问。
* 具有在它之前的执行的方法，但没有之后执行的方法。

自定义授权筛选器需要自定义授权框架。 建议配置授权策略或编写自定义授权策略，而不是编写自定义筛选器。 内置授权筛选器：

* 调用授权系统。
* 不授权请求。

不会在授权筛选器中引发异常  ：

* 不会处理异常。
* 异常筛选器不会处理异常。

在授权筛选器出现异常时请小心应对。

详细了解[授权](xref:security/authorization/introduction)。

## <a name="resource-filters"></a>资源筛选器

资源筛选器：

* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter> 接口。
* 执行会覆盖筛选器管道的绝大部分。
* 只有 [授权筛选器](#authorization-filters) 才会在资源筛选器之前运行。

如果要使大部分管道短路，资源筛选器会很有用。 例如，如果缓存命中，则缓存筛选器可以绕开管道的其余阶段。

资源筛选器示例：

* 之前显示的[短路资源筛选器](#short-circuiting-resource-filter)。
* [DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/2.0.0-preview2/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs)：

  * 可以防止模型绑定访问表单数据。
  * 用于上传大型文件，以防止表单数据被读入内存。

## <a name="action-filters"></a>操作筛选器

操作筛选器 **不适用于** Razor 页面。 Razor 页面支持 <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> 。 有关详细信息，请参阅 [Razor Pages 的筛选方法](xref:razor-pages/filter)。

操作筛选器：

* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> 接口。
* 它们的执行围绕着操作方法的执行。

以下代码显示示例操作筛选器：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> 提供以下属性：

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> - 用于读取操作方法的输入。
* <xref:Microsoft.AspNetCore.Mvc.Controller> - 用于处理控制器实例。
* <xref:System.Web.Mvc.ActionExecutingContext.Result> - 设置 `Result` 会使操作方法和后续操作筛选器的执行短路。

在操作方法中引发异常：

* 防止运行后续筛选器。
* 与设置 `Result` 不同，结果被视为失败而不是成功。

<xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> 提供 `Controller` 和 `Result` 以及以下属性：

* <xref:System.Web.Mvc.ActionExecutedContext.Canceled> - 如果操作执行已被另一个筛选器设置短路，则为 true。
* <xref:System.Web.Mvc.ActionExecutedContext.Exception> - 如果操作或之前运行的操作筛选器引发了异常，则为非 NULL 值。 将此属性设置为 null：

  * 有效地处理异常。
  * 执行 `Result`，从操作方法中将它返回。

对于 `IAsyncActionFilter`，一个向 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> 的调用可以达到以下目的：

* 执行所有后续操作筛选器和操作方法。
* 返回 `ActionExecutedContext`。

若要设置短路，可将 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> 分配到某个结果实例，并且不调用 `next` (`ActionExecutionDelegate`)。

该框架提供一个可子类化的抽象 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>。

`OnActionExecuting` 操作筛选器可用于：

* 验证模型状态。
* 如果状态无效，则返回错误。

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

> [!NOTE]
> 使用特性批注的控制器 `[ApiController]` 自动验证模型状态并返回400响应。 有关详细信息，请参阅[自动 HTTP 400 响应](xref:web-api/index#automatic-http-400-responses)。

`OnActionExecuted` 方法在操作方法之后运行：

* 可通过 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result> 属性查看和处理操作结果。
* 如果操作执行已被另一个筛选器设置短路，则 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> 设置为 true。
* 如果操作或后续操作筛选器引发了异常，则 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> 设置为非 NULL 值。 将 `Exception` 设置为 null：

  * 有效地处理异常。
  * 执行 `ActionExecutedContext.Result`，从操作方法中将它正常返回。

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a>异常筛选器

异常筛选器：

* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>。
* 可用于实现常见的错误处理策略。

下面的异常筛选器示例使用自定义错误视图，显示在开发应用时发生的异常的相关详细信息：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/CustomExceptionFilter.cs?name=snippet_ExceptionFilter&highlight=16-19)]

以下代码测试异常筛选器：

[!code-csharp[](filters/3.1sample/FiltersSample/Controllers/FailingController.cs?name=snippet)]

异常筛选器：

* 没有之前和之后的事件。
* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>。
* 处理在 Razor 页或控制器创建、 [模型绑定](xref:mvc/models/model-binding)、操作筛选器或操作方法中发生的未经处理的异常。
* 不要 **捕获资源** 筛选器、结果筛选器或 MVC 结果执行中发生的异常。

若要处理异常，请将 <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> 属性设置为 `true`，或编写响应。 这将停止传播异常。 异常筛选器无法将异常转变为“成功”。 只有操作筛选器才能执行该转变。

异常筛选器：

* 非常适合捕获发生在操作中的异常。
* 并不像错误处理中间件那么灵活。

建议使用中间件处理异常。 基于所调用的操作方法，仅当错误处理不同时，才使用异常筛选器  。 例如，应用可能具有用于 API 终结点和视图/HTML 的操作方法。 API 终结点可能返回 JSON 形式的错误信息，而基于视图的操作可能返回 HTML 形式的错误页。

## <a name="result-filters"></a>结果筛选器

结果筛选器：

* 实现接口：
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter>
* 它们的执行围绕着操作结果的执行。

### <a name="iresultfilter-and-iasyncresultfilter"></a>IResultFilter 和 IAsyncResultFilter

以下代码显示一个添加 HTTP 标头的结果筛选器：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

要执行的结果类型取决于所执行的操作。 返回视图的操作会将所有 Razor 处理作为要执行的 <xref:Microsoft.AspNetCore.Mvc.ViewResult> 的一部分。 API 方法可能会将某些序列化操作作为结果执行的一部分。 详细了解 [操作结果](xref:mvc/controllers/actions)。

仅当操作或操作筛选器生成操作结果时，才会执行结果筛选器。 不会在以下情况下执行结果筛选器：

* 授权筛选器或资源筛选器使管道短路。
* 异常筛选器通过生成操作结果来处理异常。

<xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> 方法可以将 <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> 设置为 `true`，使操作结果和后续结果筛选器的执行短路。 设置短路时写入响应对象，以免生成空响应。 如果在 `IResultFilter.OnResultExecuting` 中引发异常，则会导致：

* 阻止操作结果和后续筛选器的执行。
* 结果被视为失败而不是成功。

当 <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> 方法运行时，响应可能已发送到客户端。 如果响应已发送到客户端，则无法更改。

如果操作结果执行已被另一个筛选器设置短路，则 `ResultExecutedContext.Canceled` 设置为 `true`。

如果操作结果或后续结果筛选器引发了异常，则 `ResultExecutedContext.Exception` 设置为非 NULL 值。 将 `Exception` 设置为 NULL 可有效地处理异常，并防止在管道的后续阶段引发该异常。 处理结果筛选器中出现的异常时，没有可靠的方法来将数据写入响应。 如果在操作结果引发异常时标头已刷新到客户端，则没有任何可靠的机制可用于发送失败代码。

对于 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>，通过调用 <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> 上的 `await next` 可执行所有后续结果筛选器和操作结果。 若要设置短路，请将 [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) 设置为 `true`，并且不调用 `ResultExecutionDelegate`：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

该框架提供一个可子类化的抽象 `ResultFilterAttribute`。 前面所示的 [AddHeaderAttribute](#add-header-attribute) 类是一种结果筛选器属性。

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a>IAlwaysRunResultFilter 和 IAsyncAlwaysRunResultFilter

<xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> 接口声明了一个针对所有操作结果运行的 <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> 实现。 这包括由以下对象生成的操作结果：

* 设置短路的授权筛选器和资源筛选器。
* 异常筛选器。

例如，以下筛选器始终运行并在内容协商失败时设置具有“422 无法处理的实体”  状态代码的操作结果 (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>)：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

### <a name="ifilterfactory"></a>IFilterFactory

<xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> 可实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>。 因此，`IFilterFactory` 实例可在筛选器管道中的任意位置用作 `IFilterMetadata` 实例。 当运行时准备调用筛选器时，它会尝试将其转换为 `IFilterFactory`。 如果转换成功，则调用 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> 方法来创建将调用的 `IFilterMetadata` 实例。 这提供了一种很灵活的设计，因为无需在应用启动时显式设置精确的筛选器管道。

可以使用自定义属性实现来实现 `IFilterFactory` 作为另一种创建筛选器的方法：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

在以下代码中应用了筛选器：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/SampleController.cs?name=snippet3&highlight=21)]

通过运行[下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample)来测试前面的代码：

* 调用 F12 开发人员工具。
* 导航到 `https://localhost:5001/Sample/HeaderWithFactory`。

F12 开发人员工具显示示例代码添加的以下响应标头：

* **作者：**`Rick Anderson`
* **globaladdheader:** `Result filter added to MvcOptions.Filters`
* **内部：**`My header`

前面的代码创建 **internal:** `My header` 响应标头。

### <a name="ifilterfactory-implemented-on-an-attribute"></a>在属性上实现 IFilterFactory

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

实现 `IFilterFactory` 的筛选器可用于以下筛选器：

* 不需要传递参数。
* 具备需要由 DI 填充的构造函数依赖项。

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> 可实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>。 `IFilterFactory` 公开用于创建 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> 实例的 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> 方法。 `CreateInstance` 从服务容器 (DI) 中加载指定的类型。

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

以下代码显示应用 `[SampleActionFilter]` 的三种方法：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

在前面的代码中，使用 `[SampleActionFilter]` 修饰方法是应用 `SampleActionFilter` 的首选方法。

## <a name="using-middleware-in-the-filter-pipeline"></a>在筛选器管道中使用中间件

资源筛选器的工作方式与[中间件](xref:fundamentals/middleware/index)类似，即涵盖管道中的所有后续执行。 但筛选器又不同于中间件，它们是运行时的一部分，这意味着它们有权访问上下文和构造。

若要将中间件用作筛选器，可创建一个具有 `Configure` 方法的类型，该方法可指定要注入到筛选器管道的中间件。 下面的示例使用本地化中间件为请求建立当前区域性：

[!code-csharp[](./filters/3.1sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,22)]

使用 <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> 运行中间件：

[!code-csharp[](./filters/3.1sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

中间件筛选器与资源筛选器在筛选器管道的相同阶段运行，即，在模型绑定之前以及管道的其余阶段之后。

## <a name="next-actions"></a>后续操作

* 请参阅 [筛选 Razor 页面的方法](xref:razor-pages/filter)。
* 若要尝试使用筛选器，请[下载、测试并修改 GitHub 示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/3.1sample)。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Kirk Larkin](https://github.com/serpent5)、[Rick Anderson](https://twitter.com/RickAndMSFT)、[Tom Dykstra](https://github.com/tdykstra/) 和 [Steve Smith](https://ardalis.com/)

通过使用 ASP.NET Core 中的筛选器，可在请求处理管道中的特定阶段之前或之后运行代码。 

内置筛选器处理任务，例如：

* 授权（防止用户访问未获授权的资源）。
* 响应缓存（对请求管道进行短路出路，以便返回缓存的响应）。

可以创建自定义筛选器，用于处理横切关注点。 横切关注点的示例包括错误处理、缓存、配置、授权和日志记录。  筛选器可以避免复制代码。 例如，错误处理异常筛选器可以合并错误处理。

本文档适用于 Razor 具有视图的页面、API 控制器和控制器。

[查看或下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample)（[如何下载](xref:index#how-to-download-a-sample)）。

## <a name="how-filters-work"></a>筛选器的工作原理

筛选器在 ASP.NET Core 操作调用管道  （有时称为筛选器管道  ）内运行。  筛选器管道在 ASP.NET Core 选择了要执行的操作之后运行。

![请求通过其他中间件、路由中间件、操作选择和 ASP.NET Core 操作调用管道进行处理。 请求处理继续往回通过操作选择、路由中间件和各种其他中间件，变成发送到客户端的响应。](filters/_static/filter-pipeline-1.png)

### <a name="filter-types"></a>筛选器类型

每种筛选器类型都在筛选器管道中的不同阶段执行：

* [授权筛选器](#authorization-filters)最先运行，用于确定是否已针对请求为用户授权。 如果请求未获授权，授权筛选器可以让管道短路。

* [资源筛选器](#resource-filters)：

  * 授权后运行。  
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuting*> 可以在筛选器管道的其余阶段之前运行代码。 例如，`OnResourceExecuting` 可以在模型绑定之前运行代码。
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter.OnResourceExecuted*> 可以在管道的其余阶段完成之后运行代码。

* [操作筛选器](#action-filters)可以在调用单个操作方法之前和之后立即运行代码。 它们可用于处理传入某个操作的参数以及从该操作返回的结果。 页面 **不** 支持操作筛选器 Razor 。

* [异常筛选器](#exception-filters)用于在向响应正文写入任何内容之前，对未经处理的异常应用全局策略。

* [结果筛选器](#result-filters)可以在执行单个操作结果之前和之后立即运行代码。 仅当操作方法成功执行时，它们才会运行。 对于必须围绕视图或格式化程序的执行的逻辑，它们很有用。

下图展示了筛选器类型在筛选器管道中的交互方式。

![请求通过授权过滤器、资源过滤器、模型绑定、操作过滤器、操作执行和操作结果转换、异常过滤器、结果过滤器和结果执行进行处理。 返回时，请求仅由结果过滤器和资源过滤器进行处理，变成发送到客户端的响应。](filters/_static/filter-pipeline-2.png)

## <a name="implementation"></a>实现

筛选器通过不同的接口定义支持同步和异步实现。

同步筛选器可以在其管道阶段之前 (`On-Stage-Executing`) 和之后 (`On-Stage-Executed`) 运行代码。 例如，`OnActionExecuting` 在调用操作方法之前调用。 `OnActionExecuted` 在操作方法返回之后调用。

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

异步筛选器定义 `On-Stage-ExecutionAsync` 方法：

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleAsyncActionFilter.cs?name=snippet)]

在前面的代码中，`SampleAsyncActionFilter` 具有执行操作方法的 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> (`next`)。  每个 `On-Stage-ExecutionAsync` 方法采用执行筛选器的管道阶段的 `FilterType-ExecutionDelegate`。

### <a name="multiple-filter-stages"></a>多个筛选器阶段

可以在单个类中实现多个筛选器阶段的接口。 例如，<xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute> 类实现 `IActionFilter`、`IResultFilter` 及其异步等效接口。

筛选器接口的同步和异步版本任意实现一个，而不是同时实现 。 运行时会先查看筛选器是否实现了异步接口，如果是，则调用该接口。 如果不是，则调用同步接口的方法。 如果在一个类中同时实现异步和同步接口，则仅调用异步方法。 使用抽象类时（如 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>），将为每种筛选器类型仅重写同步方法或仅重写异步方法。

### <a name="built-in-filter-attributes"></a>内置筛选器属性

ASP.NET Core 包含许多可子类化和自定义的基于属性的内置筛选器。 例如，以下结果筛选器会向响应添加标头：

<a name="add-header-attribute"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderAttribute.cs?name=snippet)]

通过使用属性，筛选器可接收参数，如前面的示例所示。 将 `AddHeaderAttribute` 添加到控制器或操作方法，并指定 HTTP 标头的名称和值：

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1)]

<!-- `https://localhost:5001/Sample` -->

多种筛选器接口具有相应属性，这些属性可用作自定义实现的基类。

筛选器属性：

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ExceptionFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.Filters.ResultFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>

## <a name="filter-scopes-and-order-of-execution"></a>筛选器作用域和执行顺序

可以将筛选器添加到管道中的以下三个 *范围* 之一：

* 在操作上使用属性。
* 在控制器上使用属性。
* 所有控制器和操作的全局筛选器，如下面的代码所示：

[!code-csharp[](./filters/sample/FiltersSample/StartupGF.cs?name=snippet_ConfigureServices)]

前面的代码使用 [MvcOptions.Filters](xref:Microsoft.AspNetCore.Mvc.MvcOptions.Filters) 集合全局添加三个筛选器。

### <a name="default-order-of-execution"></a>默认执行顺序

当有同一类型的多个筛选器时，作用域可确定筛选器执行的默认顺序  。  全局筛选器涵盖类筛选器。 类筛选器涵盖方法筛选器。

在筛选器嵌套模式下，筛选器的 after  代码会按照与 before  代码相反的顺序运行。 筛选器序列：

* 全局筛选器的 before  代码。
  * 控制器筛选器的 before  代码。
    * 操作方法筛选器的 before  代码。
    * 操作方法筛选器的 after  代码。
  * 控制器筛选器的 after  代码。
* 全局筛选器的 after  代码。
  
下面的示例阐释了为同步操作筛选器调用筛选器方法的顺序。

| 序列 | 筛选器作用域 | 筛选器方法 |
|:--------:|:------------:|:-------------:|
| 1 | 全球 | `OnActionExecuting` |
| 2 | 控制器 | `OnActionExecuting` |
| 3 | 方法 | `OnActionExecuting` |
| 4 | 方法 | `OnActionExecuted` |
| 5 | 控制器 | `OnActionExecuted` |
| 6 | 全球 | `OnActionExecuted` |

此序列显示：

* 方法筛选器已嵌套在控制器筛选器中。
* 控制器筛选器已嵌套在全局筛选器中。

### <a name="controller-and-no-locrazor-page-level-filters"></a>控制器和 Razor 页级筛选器

继承自 <xref:Microsoft.AspNetCore.Mvc.Controller> 基类的每个控制器包括 [Controller.OnActionExecuting](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuting*)、[Controller.OnActionExecutionAsync](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecutionAsync*) 和 [Controller.OnActionExecuted](xref:Microsoft.AspNetCore.Mvc.Controller.OnActionExecuted*)
`OnActionExecuted` 方法。 这些方法：

* 覆盖为给定操作运行的筛选器。
* `OnActionExecuting` 在所有操作筛选器之前调用。
* `OnActionExecuted` 在所有操作筛选器之后调用。
* `OnActionExecutionAsync` 在所有操作筛选器之前调用。 `next` 之后的筛选器中的代码在操作方法之后运行。

例如，在下载示例中，启动时全局应用 `MySampleActionFilter`。

`TestController`：

* 将 `SampleActionFilterAttribute` (`[SampleActionFilter]`) 应用于 `FilterTest2` 操作。
* 重写 `OnActionExecuting` 和 `OnActionExecuted`。

[!code-csharp[](./filters/sample/FiltersSample/Controllers/TestController.cs?name=snippet)]

导航到 `https://localhost:5001/Test/FilterTest2` 运行以下代码：

* `TestController.OnActionExecuting`
  * `MySampleActionFilter.OnActionExecuting`
    * `SampleActionFilterAttribute.OnActionExecuting`
      * `TestController.FilterTest2`
    * `SampleActionFilterAttribute.OnActionExecuted`
  * `MySampleActionFilter.OnActionExecuted`
* `TestController.OnActionExecuted`

有关 Razor 页面，请 [参阅 Razor 通过重写筛选器方法实现页面筛选器](xref:razor-pages/filter#implement-razor-page-filters-by-overriding-filter-methods)。

### <a name="overriding-the-default-order"></a>重写默认顺序

可以通过实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter> 来重写默认执行序列。 `IOrderedFilter` 公开了 <xref:Microsoft.AspNetCore.Mvc.Filters.IOrderedFilter.Order> 属性来确定执行顺序，该属性优先于作用域。 具有较低的 `Order` 值的筛选器：

* 在具有较高的 `Order` 值的筛选器之前运行 before  代码。
* 在具有较高的 `Order` 值的筛选器之后运行 after  代码。

可以使用构造函数参数设置 `Order` 属性：

```csharp
[MyFilter(Name = "Controller Level Attribute", Order=1)]
```

请考虑前面示例中所示的 3 个相同操作筛选器。 如果控制器和全局筛选器的 `Order` 属性分别设置为 1 和 2，则会反转执行顺序。

| 序列 | 筛选器作用域 | `Order` 属性 | 筛选器方法 |
|:--------:|:------------:|:-----------------:|:-------------:|
| 1 | 方法 | 0 | `OnActionExecuting` |
| 2 | 控制器 | 1  | `OnActionExecuting` |
| 3 | 全球 | 2  | `OnActionExecuting` |
| 4 | 全球 | 2  | `OnActionExecuted` |
| 5 | 控制器 | 1  | `OnActionExecuted` |
| 6 | 方法 | 0  | `OnActionExecuted` |

在确定筛选器的运行顺序时，`Order` 属性重写作用域。 先按顺序对筛选器排序，然后使用作用域消除并列问题。 所有内置筛选器实现 `IOrderedFilter` 并将默认 `Order` 值设为 0。 对于内置筛选器，作用域会确定顺序，除非将 `Order` 设为非零值。

## <a name="cancellation-and-short-circuiting"></a>取消和设置短路

通过设置提供给筛选器方法的 <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext> 参数上的 <xref:Microsoft.AspNetCore.Mvc.Filters.ResourceExecutingContext.Result> 属性，可以使筛选器管道短路。 例如，以下资源筛选器将阻止执行管道的其余阶段：

<a name="short-circuiting-resource-filter"></a>

[!code-csharp[](./filters/sample/FiltersSample/Filters/ShortCircuitingResourceFilterAttribute.cs?name=snippet)]

在下面的代码中，`ShortCircuitingResourceFilter` 和 `AddHeader` 筛选器都以 `SomeResource` 操作方法为目标。 `ShortCircuitingResourceFilter`：

* 先运行，因为它是资源筛选器且 `AddHeader` 是操作筛选器。
* 对管道的其余部分进行短路处理。

这样 `AddHeader` 筛选器就不会为 `SomeResource` 操作运行。 如果这两个筛选器都应用于操作方法级别，只要 `ShortCircuitingResourceFilter` 先运行，此行为就不会变。 先运行 `ShortCircuitingResourceFilter`（考虑到它的筛选器类型），或显式使用 `Order` 属性。

[!code-csharp[](./filters/sample/FiltersSample/Controllers/SampleController.cs?name=snippet_AddHeader&highlight=1,9)]

## <a name="dependency-injection"></a>依赖项注入

可按类型或实例添加筛选器。 如果添加实例，该实例将用于每个请求。 如果添加类型，则将激活该类型。 激活类型的筛选器意味着：

* 将为每个请求创建一个实例。
* [依赖关系注入](xref:fundamentals/dependency-injection) (DI) 将填充所有构造函数依赖项。

如果将筛选器作为属性实现并直接添加到控制器类或操作方法中，则该筛选器不能由[依赖关系注入](xref:fundamentals/dependency-injection) (DI) 提供构造函数依赖项。 无法由 DI 提供构造函数依赖项，因为：

* 属性在应用时必须提供自己的构造函数参数。 
* 这是属性工作原理上的限制。

以下筛选器支持从 DI 提供的构造函数依赖项：

* <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute>
* <xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute>
* 在属性上实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>。

可以将前面的筛选器应用于控制器或操作方法：

可以从 DI 获取记录器。 但是，避免创建和使用筛选器仅用于日志记录。 [内置框架日志记录](xref:fundamentals/logging/index)通常提供日志记录所需的内容。 添加到筛选器的日志记录：

* 应重点关注业务域问题或特定于筛选器的行为。
* 不应记录操作或其他框架事件  。 内置筛选器记录操作和框架事件。

### <a name="servicefilterattribute"></a>ServiceFilterAttribute

在 `ConfigureServices` 中注册服务筛选器实现类型。 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> 可从 DI 检索筛选器实例。

以下代码显示 `AddHeaderResultServiceFilter`：

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

在以下代码中，`AddHeaderResultServiceFilter` 将添加到 DI 容器中：

[!code-csharp[](./filters/sample/FiltersSample/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

在以下代码中，`ServiceFilter` 属性将从 DI 中检索 `AddHeaderResultServiceFilter` 筛选器的实例：

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_ServiceFilter&highlight=1)]

使用 `ServiceFilterAttribute` 时，[ServiceFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute.IsReusable) 设置：

* 提供以下提示：筛选器实例可能在其创建的请求范围之外被重用。  ASP.NET Core 运行时不保证：

  * 将创建筛选器的单一实例。
  * 稍后不会从 DI 容器重新请求筛选器。

* 不应与依赖于生命周期不同于单一实例的服务的筛选器一起使用。

 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> 可实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>。 `IFilterFactory` 公开用于创建 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> 实例的 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> 方法。 `CreateInstance` 从 DI 中加载指定的类型。

### <a name="typefilterattribute"></a>TypeFilterAttribute

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> 与 <xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute> 类似，但不会直接从 DI 容器解析其类型。 它使用 <xref:Microsoft.Extensions.DependencyInjection.ObjectFactory?displayProperty=fullName> 对类型进行实例化。

因为不会直接从 DI 容器解析 `TypeFilterAttribute` 类型：

* 使用 `TypeFilterAttribute` 引用的类型不需要注册在 DI 容器中。  它们具备由 DI 容器实现的依赖项。
* `TypeFilterAttribute` 可以选择为类型接受构造函数参数。

使用 `TypeFilterAttribute` 时，[TypeFilterAttribute.IsReusable](xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute.IsReusable) 设置：
* 提供提示：筛选器实例可能在其创建的请求范围之外被重用。  ASP.NET Core 运行时不保证将创建筛选器的单一实例。

* 不应与依赖于生命周期不同于单一实例的服务的筛选器一起使用。

下面的示例演示如何使用 `TypeFilterAttribute` 将参数传递到类型：

[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_TypeFilter&highlight=1,2)]
[!code-csharp[](../../mvc/controllers/filters/sample/FiltersSample/Filters/LogConstantFilter.cs?name=snippet_TypeFilter_Implementation&highlight=6)]

<!-- 
https://localhost:5001/home/hi?name=joe
VS debug window shows 
FiltersSample.Filters.LogConstantFilter:Information: Method 'Hi' called
-->

## <a name="authorization-filters"></a>授权筛选器

授权筛选器：

* 是筛选器管道中运行的第一个筛选器。
* 控制对操作方法的访问。
* 具有在它之前的执行的方法，但没有之后执行的方法。

自定义授权筛选器需要自定义授权框架。 建议配置授权策略或编写自定义授权策略，而不是编写自定义筛选器。 内置授权筛选器：

* 调用授权系统。
* 不授权请求。

不会在授权筛选器中引发异常  ：

* 不会处理异常。
* 异常筛选器不会处理异常。

在授权筛选器出现异常时请小心应对。

详细了解[授权](xref:security/authorization/introduction)。

## <a name="resource-filters"></a>资源筛选器

资源筛选器：

* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IResourceFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResourceFilter> 接口。
* 执行会覆盖筛选器管道的绝大部分。
* 只有 [授权筛选器](#authorization-filters) 才会在资源筛选器之前运行。

如果要使大部分管道短路，资源筛选器会很有用。 例如，如果缓存命中，则缓存筛选器可以绕开管道的其余阶段。

资源筛选器示例：

* 之前显示的[短路资源筛选器](#short-circuiting-resource-filter)。
* [DisableFormValueModelBindingAttribute](https://github.com/aspnet/Entropy/blob/rel/2.0.0-preview2/samples/Mvc.FileUpload/Filters/DisableFormValueModelBindingAttribute.cs)：

  * 可以防止模型绑定访问表单数据。
  * 用于上传大型文件，以防止表单数据被读入内存。

## <a name="action-filters"></a>操作筛选器

> [!IMPORTANT]
> 操作筛选器 **不适用于** Razor 页面。 Razor 页面支持 <xref:Microsoft.AspNetCore.Mvc.Filters.IPageFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncPageFilter> 。 有关详细信息，请参阅 [Razor Pages 的筛选方法](xref:razor-pages/filter)。

操作筛选器：

* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IActionFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncActionFilter> 接口。
* 它们的执行围绕着操作方法的执行。

以下代码显示示例操作筛选器：

[!code-csharp[](./filters/sample/FiltersSample/Filters/MySampleActionFilter.cs?name=snippet_ActionFilter)]

<xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext> 提供以下属性：

* <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.ActionArguments> - 用于读取操作方法的输入。
* <xref:Microsoft.AspNetCore.Mvc.Controller> - 用于处理控制器实例。
* <xref:System.Web.Mvc.ActionExecutingContext.Result> - 设置 `Result` 会使操作方法和后续操作筛选器的执行短路。

在操作方法中引发异常：

* 防止运行后续筛选器。
* 与设置 `Result` 不同，结果被视为失败而不是成功。

<xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext> 提供 `Controller` 和 `Result` 以及以下属性：

* <xref:System.Web.Mvc.ActionExecutedContext.Canceled> - 如果操作执行已被另一个筛选器设置短路，则为 true。
* <xref:System.Web.Mvc.ActionExecutedContext.Exception> - 如果操作或之前运行的操作筛选器引发了异常，则为非 NULL 值。 将此属性设置为 null：

  * 有效地处理异常。
  * 执行 `Result`，从操作方法中将它返回。

对于 `IAsyncActionFilter`，一个向 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutionDelegate> 的调用可以达到以下目的：

* 执行所有后续操作筛选器和操作方法。
* 返回 `ActionExecutedContext`。

若要设置短路，可将 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutingContext.Result?displayProperty=fullName> 分配到某个结果实例，并且不调用 `next` (`ActionExecutionDelegate`)。

该框架提供一个可子类化的抽象 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionFilterAttribute>。

`OnActionExecuting` 操作筛选器可用于：

* 验证模型状态。
* 如果状态无效，则返回错误。

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet)]

`OnActionExecuted` 方法在操作方法之后运行：

* 可通过 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Result> 属性查看和处理操作结果。
* 如果操作执行已被另一个筛选器设置短路，则 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Canceled> 设置为 true。
* 如果操作或后续操作筛选器引发了异常，则 <xref:Microsoft.AspNetCore.Mvc.Filters.ActionExecutedContext.Exception> 设置为非 NULL 值。 将 `Exception` 设置为 null：

  * 有效地处理异常。
  * 执行 `ActionExecutedContext.Result`，从操作方法中将它正常返回。

[!code-csharp[](./filters/sample/FiltersSample/Filters/ValidateModelAttribute.cs?name=snippet2&higlight=12-99)]

## <a name="exception-filters"></a>异常筛选器

异常筛选器：

* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter>。 
* 可用于实现常见的错误处理策略。

下面的异常筛选器示例使用自定义错误视图，显示在开发应用时发生的异常的相关详细信息：

[!code-csharp[](./filters/sample/FiltersSample/Filters/CustomExceptionFilter.cs?name=snippet_ExceptionFilter&highlight=16-19)]

异常筛选器：

* 没有之前和之后的事件。
* 实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IExceptionFilter.OnException*> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncExceptionFilter.OnExceptionAsync*>。
* 处理在 Razor 页或控制器创建、 [模型绑定](xref:mvc/models/model-binding)、操作筛选器或操作方法中发生的未经处理的异常。
* 不要 **捕获资源** 筛选器、结果筛选器或 MVC 结果执行中发生的异常。

若要处理异常，请将 <xref:System.Web.Mvc.ExceptionContext.ExceptionHandled> 属性设置为 `true`，或编写响应。 这将停止传播异常。 异常筛选器无法将异常转变为“成功”。 只有操作筛选器才能执行该转变。

异常筛选器：

* 非常适合捕获发生在操作中的异常。
* 并不像错误处理中间件那么灵活。

建议使用中间件处理异常。 基于所调用的操作方法，仅当错误处理不同时，才使用异常筛选器  。 例如，应用可能具有用于 API 终结点和视图/HTML 的操作方法。 API 终结点可能返回 JSON 形式的错误信息，而基于视图的操作可能返回 HTML 形式的错误页。

## <a name="result-filters"></a>结果筛选器

结果筛选器：

* 实现接口：
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>
  * <xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> 或 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter>
* 它们的执行围绕着操作结果的执行。

### <a name="iresultfilter-and-iasyncresultfilter"></a>IResultFilter 和 IAsyncResultFilter

以下代码显示一个添加 HTTP 标头的结果筛选器：

[!code-csharp[](./filters/sample/FiltersSample/Filters/LoggingAddHeaderFilter.cs?name=snippet_ResultFilter)]

要执行的结果类型取决于所执行的操作。 返回视图的操作会将所有 Razor 处理作为要执行的 <xref:Microsoft.AspNetCore.Mvc.ViewResult> 的一部分。 API 方法可能会将某些序列化操作作为结果执行的一部分。 详细了解 [操作结果](xref:mvc/controllers/actions)。

仅当操作或操作筛选器生成操作结果时，才会执行结果筛选器。 不会在以下情况下执行结果筛选器：

* 授权筛选器或资源筛选器使管道短路。
* 异常筛选器通过生成操作结果来处理异常。

<xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuting*?displayProperty=fullName> 方法可以将 <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel?displayProperty=fullName> 设置为 `true`，使操作结果和后续结果筛选器的执行短路。 设置短路时写入响应对象，以免生成空响应。 如果在 `IResultFilter.OnResultExecuting` 中引发异常，则会导致：

* 阻止操作结果和后续筛选器的执行。
* 结果被视为失败而不是成功。

当 <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter.OnResultExecuted*?displayProperty=fullName> 方法运行时，响应可能已发送到客户端。 如果响应已发送到客户端，则无法再更改。

如果操作结果执行已被另一个筛选器设置短路，则 `ResultExecutedContext.Canceled` 设置为 `true`。

如果操作结果或后续结果筛选器引发了异常，则 `ResultExecutedContext.Exception` 设置为非 NULL 值。 将 `Exception` 设置为 NULL 可有效地处理异常，并防止 ASP.NET Core 在管道的后续阶段重新引发该异常。 处理结果筛选器中出现的异常时，没有可靠的方法来将数据写入响应。 如果在操作结果引发异常时标头已刷新到客户端，则没有任何可靠的机制可用于发送失败代码。

对于 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncResultFilter>，通过调用 <xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutionDelegate> 上的 `await next` 可执行所有后续结果筛选器和操作结果。 若要设置短路，请将 [ResultExecutingContext.Cancel](xref:Microsoft.AspNetCore.Mvc.Filters.ResultExecutingContext.Cancel) 设置为 `true`，并且不调用 `ResultExecutionDelegate`：

[!code-csharp[](./filters/sample/FiltersSample/Filters/MyAsyncResponseFilter.cs?name=snippet)]

该框架提供一个可子类化的抽象 `ResultFilterAttribute`。 前面所示的 [AddHeaderAttribute](#add-header-attribute) 类是一种结果筛选器属性。

### <a name="ialwaysrunresultfilter-and-iasyncalwaysrunresultfilter"></a>IAlwaysRunResultFilter 和 IAsyncAlwaysRunResultFilter

<xref:Microsoft.AspNetCore.Mvc.Filters.IAlwaysRunResultFilter> 和 <xref:Microsoft.AspNetCore.Mvc.Filters.IAsyncAlwaysRunResultFilter> 接口声明了一个针对所有操作结果运行的 <xref:Microsoft.AspNetCore.Mvc.Filters.IResultFilter> 实现。 这包括由以下对象生成的操作结果：

* 设置短路的授权筛选器和资源筛选器。
* 异常筛选器。

例如，以下筛选器始终运行并在内容协商失败时设置具有“422 无法处理的实体”  状态代码的操作结果 (<xref:Microsoft.AspNetCore.Mvc.ObjectResult>)：

[!code-csharp[](./filters/sample/FiltersSample/Filters/UnprocessableResultFilter.cs?name=snippet)]

### <a name="ifilterfactory"></a>IFilterFactory

<xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory> 可实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata>。 因此，`IFilterFactory` 实例可在筛选器管道中的任意位置用作 `IFilterMetadata` 实例。 当运行时准备调用筛选器时，它会尝试将其转换为 `IFilterFactory`。 如果转换成功，则调用 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> 方法来创建将调用的 `IFilterMetadata` 实例。 这提供了一种很灵活的设计，因为无需在应用启动时显式设置精确的筛选器管道。

可以使用自定义属性实现来实现 `IFilterFactory` 作为另一种创建筛选器的方法：

[!code-csharp[](./filters/sample/FiltersSample/Filters/AddHeaderWithFactoryAttribute.cs?name=snippet_IFilterFactory&highlight=1,4,5,6,7)]

可以通过运行[下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample)来测试前面的代码：

* 调用 F12 开发人员工具。
* 导航到 `https://localhost:5001/Sample/HeaderWithFactory`。

F12 开发人员工具显示示例代码添加的以下响应标头：

* **作者：**`Joe Smith`
* **globaladdheader:** `Result filter added to MvcOptions.Filters`
* **内部：**`My header`

前面的代码创建 **internal:** `My header` 响应标头。

### <a name="ifilterfactory-implemented-on-an-attribute"></a>在属性上实现 IFilterFactory

<!-- Review 
This section needs to be rewritten.
What's a non-named attribute?
-->

实现 `IFilterFactory` 的筛选器可用于以下筛选器：

* 不需要传递参数。
* 具备需要由 DI 填充的构造函数依赖项。

<xref:Microsoft.AspNetCore.Mvc.TypeFilterAttribute> 可实现 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory>。 `IFilterFactory` 公开用于创建 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterMetadata> 实例的 <xref:Microsoft.AspNetCore.Mvc.Filters.IFilterFactory.CreateInstance*> 方法。 `CreateInstance` 从服务容器 (DI) 中加载指定的类型。

[!code-csharp[](./filters/sample/FiltersSample/Filters/SampleActionFilterAttribute.cs?name=snippet_TypeFilterAttribute&highlight=1,3,7)]

以下代码显示应用 `[SampleActionFilter]` 的三种方法：

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet&highlight=1)]

在前面的代码中，使用 `[SampleActionFilter]` 修饰方法是应用 `SampleActionFilter` 的首选方法。

## <a name="using-middleware-in-the-filter-pipeline"></a>在筛选器管道中使用中间件

资源筛选器的工作方式与[中间件](xref:fundamentals/middleware/index)类似，即涵盖管道中的所有后续执行。 但筛选器又不同于中间件，它们是 ASP.NET Core 运行时的一部分，这意味着它们有权访问 ASP.NET Core 上下文和构造。

若要将中间件用作筛选器，可创建一个具有 `Configure` 方法的类型，该方法可指定要注入到筛选器管道的中间件。 下面的示例使用本地化中间件为请求建立当前区域性：

[!code-csharp[](./filters/sample/FiltersSample/Filters/LocalizationPipeline.cs?name=snippet_MiddlewareFilter&highlight=3,22)]

使用 <xref:Microsoft.AspNetCore.Mvc.MiddlewareFilterAttribute> 运行中间件：

[!code-csharp[](./filters/sample/FiltersSample/Controllers/HomeController.cs?name=snippet_MiddlewareFilter&highlight=2)]

中间件筛选器与资源筛选器在筛选器管道的相同阶段运行，即，在模型绑定之前以及管道的其余阶段之后。

## <a name="next-actions"></a>后续操作

* 请参阅 [筛选 Razor 页面的方法](xref:razor-pages/filter)。
* 若要尝试使用筛选器，请[下载、测试并修改 GitHub 示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/filters/sample)。

::: moniker-end
