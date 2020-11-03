---
title: 在 ASP.NET Core MVC 中使用控制器处理请求
author: ardalis
description: ''
ms.author: riande
ms.date: 12/05/2019
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
uid: mvc/controllers/actions
ms.openlocfilehash: a9319e74d0213b178c2a71be69a0332270d9446c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061452"
---
# <a name="handle-requests-with-controllers-in-aspnet-core-mvc"></a>在 ASP.NET Core MVC 中使用控制器处理请求

作者：[Steve Smith](https://ardalis.com/) 和 [Scott Addie](https://github.com/scottaddie)

控制器、操作和操作结果是开发人员如何使用 ASP.NET Core MVC 生成应用的一个基本组成部分。

## <a name="what-is-a-controller"></a>什么是控制器？

控制器用于对一组操作进行定义和分组。 操作（或操作方法  ）是控制器上一种用来处理请求的方法。 控制器按逻辑将类似的操作集合到一起。 通过这种操作的聚合，可以共同应用路由、缓存和授权等通用规则集。 请求通过[路由](xref:mvc/controllers/routing)映射到操作。

根据惯例，控制器类：

* 驻留在项目的根级 *控制器* 文件夹中。
* 继承自 `Microsoft.AspNetCore.Mvc.Controller`。

控制器是一个可实例化的类，其中下列条件至少某一个为 true：

* 类名称带有 `Controller` 后缀。
* 该类继承自带有 `Controller` 后缀的类。
* `[Controller]` 特性应用于该类。

控制器类不可含有关联的 `[NonController]` 属性。

控制器应遵循 [Explicit Dependencies Principle](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)（显式依赖关系原则）。 以下几种方法可以实现此原则。 如果多个控制器操作需要相同的服务，请考虑使用[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)来请求这些依赖关系。 如果该服务仅需要一个操作方法，请考虑使用[操作注入](xref:mvc/controllers/dependency-injection#action-injection-with-fromservices)来请求依赖关系。

在“模型-视图-控制器”模式中，控制器负责请求的初始处理和模型的实例化操作  。 通常情况下，应在模型中执行业务决策。

控制器获取模型处理的结果（如果有），并返回正确的视图及其关联的视图数据或 API 调用的结果。 请参阅 [ASP.NET Core MVC 概述](xref:mvc/overview)以及 [ASP.NET Core MVC 和 Visual Studio 入门](xref:tutorials/first-mvc-app/start-mvc)了解详细信息。

控制器是一个 UI 级别  的抽象。 它的职责是确保请求的数据有效，并选择应当返回的视图（或 API 的结果）。 在构造良好的应用程序中，它不会直接包括数据访问或业务逻辑。 相反，控制器会委托给处理这些责任的服务。

## <a name="defining-actions"></a>定义操作

控制器上的公共方法（除了那些带有 `[NonAction]` 属性的方法）均为操作。 操作上的参数会绑定到请求数据，并使用[模型绑定](xref:mvc/models/model-binding)进行验证。 所有模型绑定的内容都会执行模型验证。 `ModelState.IsValid` 属性值指示模型绑定和验证是否成功。

操作方法应包含用于将请求映射到某个业务关注点的逻辑。 业务关注点通常应当表示为控制器通过[依赖关系注入](xref:mvc/controllers/dependency-injection)来访问的服务。 然后，操作将业务操作的结果映射到应用程序状态。

操作可以返回任何内容，但是经常返回生成响应的 `IActionResult`（或异步方法的 `Task<IActionResult>`）的实例。 操作方法负责选择响应的类型  。 操作结果会做出响应  。

### <a name="controller-helper-methods"></a>控制器帮助程序方法

控制器通常继承自[控制器](/dotnet/api/microsoft.aspnetcore.mvc.controller)（尽管没有要求）。 派生自 `Controller` 会提供对三个帮助程序方法类别的访问：

#### <a name="1-methods-resulting-in-an-empty-response-body"></a>1. 导致空响应正文的方法

没有包含 `Content-Type` HTTP 响应标头，因为响应正文缺少要描述的内容。

该类别中有两种结果类型：重定向和 HTTP 状态代码。

* **HTTP 状态代码**

    此类型返回 HTTP 状态代码。 此类型的几种帮助程序方法是 `BadRequest`、`NotFound` 和 `Ok`。 例如，`return BadRequest();` 执行时生成 400 状态代码。 重载 `BadRequest``NotFound` 和 `Ok` 等方法时，它们不再符合 HTTP 状态代码响应方的资格，因为正在进行内容协商。

* **重定向**

    此类型（使用 `Redirect``LocalRedirect``RedirectToAction` 或 `RedirectToRoute`）返回一个到操作或目标的重定向。 例如，`return RedirectToAction("Complete", new {id = 123});` 重定向到 `Complete`，传递一个匿名对象。

    重定向结果类型与 HTTP 状态代码类型的不同之处主要在于 `Location` HTTP 响应标头的添加。

#### <a name="2-methods-resulting-in-a-non-empty-response-body-with-a-predefined-content-type"></a>2. 导致具有预定义内容类型的非空响应正文的方法

此类别中的大多数帮助程序方法都包含一个 `ContentType` 属性，通过它可以设置 `Content-Type` 响应标头来描述响应正文。

此类别中有两种结果类型：[视图](xref:mvc/views/overview)和[已格式化的响应](xref:web-api/advanced/formatting)。

* **视图**

    此类型返回一个使用模型呈现 HTML 的视图。 例如，`return View(customer);` 将模型传递给视图以进行数据绑定。

* **已格式化的响应**

    此类型返回 JSON 或类似的数据交换格式，从而以特定方式表示某个对象。 例如，`return Json(customer);` 将提供的对象串行化为 JSON 格式。
    
    此类型的其他常见方法包括 `File` 和 `PhysicalFile`。 例如，`return PhysicalFile(customerFilePath, "text/xml");` 返回 [PhysicalFileResult](/dotnet/api/microsoft.aspnetcore.mvc.physicalfileresult)。

#### <a name="3-methods-resulting-in-a-non-empty-response-body-formatted-in-a-content-type-negotiated-with-the-client"></a>3. 导致在与客户端协商的内容类型中设置非空响应正文的方法

此类别更为熟知的说法是“内容协商”  。 每当操作返回 [ObjectResult](/dotnet/api/microsoft.aspnetcore.mvc.objectresult) 类型或除 [IActionResult](/dotnet/api/microsoft.aspnetcore.mvc.iactionresult) 实现之外的任何实现时，会应用[内容协商](xref:web-api/advanced/formatting#content-negotiation)。 返回非 `IActionResult` 实现（例如 `object`）的操作也会返回已格式化的响应。

此类型的一些帮助程序方法包括 `BadRequest`、`CreatedAtRoute` 和 `Ok`。 这些方法的示例分别为 `return BadRequest(modelState);`、`return CreatedAtRoute("routename", values, newobject);` 和 `return Ok(value);`。 请注意，`BadRequest` 和 `Ok` 仅在传递了值的时候才执行内容协商；在没有传递值的情况下，它们充当 HTTP 状态码结果类型。 另一方面，`CreatedAtRoute` 方法始终执行内容协商，因为它的重载均要求传递一个值。

### <a name="cross-cutting-concerns"></a>横切关注点

应用程序通常会共享其部分工作流程。 示例包括需要身份验证才能访问购物车的应用，或者在某些页面上缓存数据的应用。 要在某个操作方法之前或之后执行逻辑，请使用筛选器  。 对横切关注点使用[筛选器](xref:mvc/controllers/filters)可以减少重复。

可在控制器或操作级别上应用大多数筛选器属性（例如 `[Authorize]`），具体取决于所需的粒度级别。

错误处理和响应缓存通常是横切关注点：
* [处理错误](xref:mvc/controllers/filters#exception-filters)
* [响应缓存](xref:performance/caching/response)

使用筛选器或自定义[中间件](xref:fundamentals/middleware/index)可处理许多横切关注点。
