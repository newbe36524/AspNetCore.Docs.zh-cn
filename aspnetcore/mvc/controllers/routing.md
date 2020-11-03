---
title: 在 ASP.NET Core 中路由到控制器操作
author: rick-anderson
description: 了解 ASP.NET Core MVC 如何使用路由中间件来匹配传入请求的 URL 并将它们映射到操作。
ms.author: riande
ms.date: 3/25/2020
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
uid: mvc/controllers/routing
ms.openlocfilehash: 9f64dd8f0ca026cec4b7ee4b5ea02523139eed4f
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93057149"
---
# <a name="routing-to-controller-actions-in-aspnet-core"></a>在 ASP.NET Core 中路由到控制器操作

作者：[Ryan Nowak](https://github.com/rynowak)、[Kirk Larkinn](https://twitter.com/serpent5) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core 控制器使用路由 [中间件](xref:fundamentals/middleware/index) 来匹配传入请求的 url，并将其映射到 [操作](#action)。  路由模板：

* 在启动代码或属性中定义。
* 描述 URL 路径如何与 [操作](#action)匹配。
* 用于生成链接的 Url。 生成的链接通常在响应中返回。

操作是按 [逆路由](#cr) 或按 [属性路由](#ar)。 在控制器或 [操作](#action) 上放置路由，使其属性路由。 有关详细信息，请参阅[混合路由](#routing-mixed-ref-label)。

本文档：

* 说明 MVC 与路由之间的交互：
  * 典型的 MVC 应用使用路由功能的方式。
  * 涵盖两种：
    * 通常用于控制器和视图的[传统路由](#cr)。
    * 用于 REST Api 的 *属性路由* 。 如果主要对 REST Api 的路由感兴趣，请跳转到 [Rest api 的属性路由](#ar) 部分。
  * 有关高级路由的详细信息，请参阅 [路由](xref:fundamentals/routing) 。
* 指 ASP.NET Core 3.0 中添加的默认路由系统，称为 "终结点路由"。 出于兼容性目的，可以将控制器用于以前版本的路由。 有关说明，请参阅 [2.2-3.0 迁移指南](xref:migration/22-to-30) 。 有关旧路由系统上的参考材料，请参阅 [本文档的2.2 版本](xref:mvc/controllers/routing?view=aspnetcore-2.2) 。

<a name="cr"></a>

## <a name="set-up-conventional-route"></a>设置传统路由

`Startup.Configure` 使用 [传统路由](#crd)时，通常具有如下所示的代码：

[!code-csharp[](routing/samples/3.x/main/StartupDefaultMVC.cs?name=snippet)]

在对的调用中 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> ， <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> 用于创建单个路由。 单路由命名为 `default` route。 大多数具有控制器和视图的应用都使用类似于路由的路由模板 `default` 。 REST Api 应使用 [属性路由](#ar)。

路由模板 `"{controller=Home}/{action=Index}/{id?}"` ：

* 匹配 URL 路径，例如 `/Products/Details/5`
* `{ controller = Products, action = Details, id = 5 }`通过词汇切分路径来提取路由值。 如果应用有一个名为的控制器 `ProductsController` 和一个操作，则提取路由值将导致匹配 `Details` ：

  [!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippetA)]

  [!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

* `/Products/Details/5` 模型将的值绑定 `id = 5` ，以将 `id` 参数设置为 `5` 。 有关更多详细信息，请参阅 [模型绑定](xref:mvc/models/model-binding) 。
* `{controller=Home}``Home`将定义为默认值 `controller` 。
* `{action=Index}``Index`将定义为默认值 `action` 。
*  `?`中的字符 `{id?}` 定义 `id` 为可选。
  * 默认路由参数和可选路由参数不必包含在 URL 路径中进行匹配。 有关路由模板语法的详细说明，请参阅[路由模板参考](xref:fundamentals/routing#route-template-reference)。
* 匹配 URL 路径 `/` 。
* 生成路由值 `{ controller = Home, action = Index }` 。

和的值 `controller` 将 `action` 使用默认值。 `id` 不会生成值，因为 URL 路径中没有相应的段。 `/` 仅当存在和操作时才匹配 `HomeController` `Index` ：

```csharp
public class HomeController : Controller
{
  public IActionResult Index() { ... }
}
```

使用上述控制器定义和路由模板，为 `HomeController.Index` 以下 URL 路径运行操作：

* `/Home/Index/17`
* `/Home/Index`
* `/Home`
* `/`

URL 路径 `/` 使用路由模板默认 `Home` 控制器和 `Index` 操作。 URL 路径 `/Home` 使用路由模板默认 `Index` 操作。

简便方法 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapDefaultControllerRoute%2A>：

```csharp
endpoints.MapDefaultControllerRoute();
```

代替

```csharp
endpoints.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

> [!IMPORTANT]
> 使用 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A> 和中间件配置路由 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A> 。 使用控制器：
>
> * <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers%2A>在内部调用 `UseEndpoints` ，以映射[属性路由](#ar)控制器。
> * 调用 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> 或 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> 以映射 [逆路由](#cr) 控制器和 [属性路由](#ar) 控制器。

<a name="routing-conventional-ref-label"></a>
<a name="crd"></a>

## <a name="conventional-routing"></a>传统路由

传统路由用于控制器和视图。 `default` 路由：

[!code-csharp[](routing/samples/3.x/main/StartupDefaultMVC.cs?name=snippet2)]

是一种 *传统路由* 。 它被称为 *传统路由* ，因为它建立了一个 URL 路径 *约定* ：

* 第一个路径段 `{controller=Home}` 映射到控制器名称。
* 第二段 `{action=Index}` 映射到 [操作](#action) 名称。
* 第三段 `{id?}` 用于可选 `id` 。 `?`中的 `{id?}` 使其成为可选的。 `id` 用于映射到模型实体。

使用此 `default` 路由，URL 路径：

* `/Products/List` 映射到 `ProductsController.List` 操作。
* `/Blog/Article/17` 映射到 `BlogController.Article` 和通常将参数绑定 `id` 到17。

此映射：

* **仅** 基于控制器和 [操作](#action)名称。
* 不基于命名空间、源文件位置或方法参数。

通过使用传统路由和默认路由，可以创建应用，而无需为每个操作都提供新的 URL 模式。 对于具有 [CRUD](https://wikipedia.org/wiki/Create,_read,_update_and_delete) 样式操作的应用，跨控制器的 url 保持一致：

* 有助于简化代码。
* 使 UI 更具可预测性。

> [!WARNING]
> `id`前面的代码中的将由路由模板定义为可选的。 无需作为 URL 的一部分提供的可选 ID 即可执行操作。 通常， `id` 从 URL 中省略时：
>
> * `id``0`由模型绑定设置为。
> * 在数据库匹配中找不到实体 `id == 0` 。
>
> [特性路由](#ar) 可提供精细的控制，以使某些操作（而不是其他操作）需要 ID。 按照约定，文档包含可选参数，如 `id` 它们可能出现在正确用法中的情况。

大多数应用应选择基本的描述性路由方案，让 URL 有可读性和意义。 默认传统路由 `{controller=Home}/{action=Index}/{id?}`：

* 支持基本的描述性路由方案。
* 是基于 UI 的应用的有用起点。
* 是许多 web UI 应用程序所需的唯一路由模板。 对于较大的 web UI 应用，使用 [区域](#areas) 的其他路由常常是必需的。

<xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> 和 <xref:Microsoft.AspNetCore.Builder.MvcAreaRouteBuilderExtensions.MapAreaRoute%2A> ：

* 根据调用的顺序，自动将 **订单** 值分配给其终结点。

ASP.NET Core 3.0 及更高版本中的终结点路由：

* 没有路由的概念。
* 不为执行扩展性提供排序保证，同时处理所有终结点。

启用[日志记录](xref:fundamentals/logging/index)以查看内置路由实现（如 <xref:Microsoft.AspNetCore.Routing.Route>）如何匹配请求。

[属性路由](#ar) 将在本文档的后面部分进行说明。

<a name="mr"></a>

### <a name="multiple-conventional-routes"></a>多个传统路由

[conventional routes](#cr) `UseEndpoints` 通过添加更多对和的调用，可以在内添加多个传统路由 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> 。 这样做允许定义多个约定，或者添加专用于特定 [操作](#action)的传统路由，例如：

[!code-csharp[](routing/samples/3.x/main/Startup.cs?name=snippet_1)]

<a name="dcr"></a>

`blog`上述代码中的路由是专用的 **传统路由** 。 这称为专用的传统路由，因为：

* 它使用 [传统路由](#cr)。
* 它专用于特定的 [操作](#action)。

因为 `controller` 和 `action` 不会以参数形式出现在路由模板中 `"blog/{*article}"` ：

* 它们只能具有默认值 `{ controller = "Blog", action = "Article" }` 。
* 此路由始终映射到操作 `BlogController.Article` 。

`/Blog`、 `/Blog/Article` 和 `/Blog/{any-string}` 是唯一与博客路由匹配的 URL 路径。

上面的示例：

* `blog` 路由具有比路由更高的优先级， `default` 因为它是首先添加的。
* 是一个 [缩略](https://developer.mozilla.org/docs/Glossary/Slug) 名称样式路由的示例，在此示例中，通常将项目名称作为 URL 的一部分。

> [!WARNING]
> 在 ASP.NET Core 3.0 及更高版本中，路由不会：
> * 定义名为 *route* 的概念。 `UseRouting` 向中间件管道添加路由匹配。 `UseRouting`中间件会查看应用中定义的终结点集，并根据请求选择最佳的终结点匹配。
> * 提供可扩展性（如或）的执行 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 顺序 <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> 。
>
>有关路由的参考材料，请参阅 [路由](xref:fundamentals/routing) 。

<a name="cro"></a>

### <a name="conventional-routing-order"></a>传统路由顺序

传统路由仅匹配应用定义的操作和控制器的组合。 这旨在简化传统路由重叠的情况。
使用、和添加路由时， <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute%2A> <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapDefaultControllerRoute%2A> 会根据 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> 调用顺序，自动将其分配给终结点。 与前面显示的路由的匹配具有更高的优先级。 传统路由依赖于顺序。 通常情况下，应将具有区域的路由置于更早的位置，因为它们比没有区域的路由更具体。 使用全部捕获路由参数的[专用传统路由](#dcr) `{*article}` 可以使路由过于[贪婪](xref:fundamentals/routing#greedy)，这意味着它与你打算与其他路由匹配的 url 相匹配。 将贪婪路由置于路由表中，以防止贪婪匹配。

[!INCLUDE[](~/includes/catchall.md)]

<a name="best"></a>

### <a name="resolving-ambiguous-actions"></a>解决不明确的操作

如果两个终结点通过路由匹配，则路由必须执行下列操作之一：

* 选择最佳的候选项。
* 引发异常。

例如：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet9)]

前面的控制器定义了两个匹配的操作：

* URL 路径 `/Products33/Edit/17`
* 路由数据 `{ controller = Products33, action = Edit, id = 17 }` 。

这是 MVC 控制器的典型模式：

* `Edit(int)` 显示用于编辑产品的窗体。
* `Edit(int, Product)` 处理已发布的窗体。

解析正确的路由：

* `Edit(int, Product)` 当请求为 HTTP 时选择 `POST` 。
* `Edit(int)` 当 [HTTP 谓词](#verb) 为其他任何内容时，将选择。 `Edit(int)` 通常通过调用 `GET` 。

<xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute>提供了 `[HttpPost]` 用于路由的，以便它可以根据请求的 HTTP 方法进行选择。 `HttpPostAttribute` `Edit(int, Product)` 比更好匹配 `Edit(int)` 。

了解属性的角色非常重要 `HttpPostAttribute` 。 为其他 [HTTP 谓词](#verb)定义了类似的属性。 在 [传统路由](#cr)中，当操作是显示形式的一部分时，操作通常使用相同的操作名称，即提交窗体工作流。 例如，请参阅 [检查两个编辑操作方法](xref:tutorials/first-mvc-app/controller-methods-views#get-post)。

如果路由无法选择最佳候选项，则 <xref:System.Reflection.AmbiguousMatchException> 会引发，并列出多个匹配的终结点。

<a name="routing-route-name-ref-label"></a>

### <a name="conventional-route-names"></a>传统路由名称

`"blog"` `"default"` 以下示例中的字符串和是传统的路由名称：

[!code-csharp[](routing/samples/3.x/main/Startup.cs?name=snippet_1)]

路由名称为路由指定逻辑名称。 命名路由可用于生成 URL。 当路由顺序使 URL 生成复杂化时，使用命名路由可简化 URL 创建。 路由名称必须是唯一的应用程序范围。

路由名称：

* 不会影响 URL 匹配或处理请求。
* 仅用于生成 URL。

路由名称概念在路由中表示为 [IEndpointNameMetadata](xref:Microsoft.AspNetCore.Routing.IEndpointNameMetadata)。 术语 **路由名称** 和 **终结点名称** ：

* 是可互换的。
* 文档和代码中使用哪一个取决于所述的 API。

<a name="attribute-routing-ref-label"></a>
<a name="ar"></a>

## <a name="attribute-routing-for-rest-apis"></a>REST Api 的属性路由

REST Api 应使用属性路由将应用功能建模为一组资源，其中的操作由 [HTTP 谓词](#verb)表示。

属性路由使用一组属性将操作直接映射到路由模板。 下面 `StartUp.Configure` 是 REST API 的典型代码，并在下一个示例中使用：

[!code-csharp[](routing/samples/3.x/main/StartupAPI.cs?name=snippet)]

在前面的代码中，在 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers%2A> 中调用， `UseEndpoints` 以映射属性路由控制器。

如下示例中：

* 使用前面的 `Configure` 方法。
* `HomeController` 匹配一组与默认传统路由匹配的 Url `{controller=Home}/{action=Index}/{id?}` 。

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet2)]

对 `HomeController.Index` 任何 URL 路径（、或）运行操作 `/` `/Home` `/Home/Index` `/Home/Index/3` 。

此示例突出显示了属性路由与 [传统路由](#cr)之间的主要编程区别。 属性路由需要更多输入才能指定路由。 传统的默认路由会更简洁地处理路由。 但是，特性路由允许和要求精确控制哪些路由模板适用于每个 [操作](#action)。

使用属性路由时，控制器和操作名称不会播放任何操作，除非使用 [令牌替换](#routing-token-replacement-templates-ref-label) 。 下面的示例匹配与上一示例相同的 Url：

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemoController.cs?name=snippet)]

下面的代码使用和的标记 `action` 替换 `controller` ：

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet22)]

以下代码适用于 `[Route("[controller]/[action]")]` 控制器：

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet24)]

在前面的代码中， `Index` 方法模板必须在 `/` 路由模板之前预置或 `~/` 。 应用于操作的以 `/` 或 `~/` 开头的路由模板不与应用于控制器的路由模板合并。

有关路由模板选择的信息，请参阅 [路由模板优先级](xref:fundamentals/routing#rtp) 。

## <a name="reserved-routing-names"></a>保留的路由名称

使用控制器或页面时，保留的路由参数名称如下 Razor ：

* `action`
* `area`
* `controller`
* `handler`
* `page`

使用 `page` 作为带有属性路由的路由参数是一个常见错误。 这样做会导致在 URL 生成时出现不一致的行为。

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemo2Controller.cs?name=snippet)]

URL 生成使用特殊参数名称来确定 URL 生成操作是指引用 Razor 页面还是引用控制器。

<a name="verb"></a>

## <a name="http-verb-templates"></a>HTTP 谓词模板

ASP.NET Core 具有以下 HTTP 谓词模板：

* [HttpGet](xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute)
* [HttpPost](xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute)
* [HttpPut](xref:Microsoft.AspNetCore.Mvc.HttpPutAttribute)
* [HttpDelete](xref:Microsoft.AspNetCore.Mvc.HttpDeleteAttribute)
* [[HttpHead]](xref:Microsoft.AspNetCore.Mvc.HttpHeadAttribute)
* [[HttpPatch]](xref:Microsoft.AspNetCore.Mvc.HttpPatchAttribute)

<a name="rt"></a>

### <a name="route-templates"></a>路由模板

ASP.NET Core 具有以下路由模板：

* 所有 [HTTP 谓词模板](#verb) 都是路由模板。
* [路由](xref:Microsoft.AspNetCore.Mvc.RouteAttribute)

<a name="arx"></a>

### <a name="attribute-routing-with-http-verb-attributes"></a>具有 Http 谓词特性的属性路由

请考虑以下控制器：

[!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet)]

在上述代码中：

* 每个操作都包含 `[HttpGet]` 特性，该特性仅将匹配限制为 HTTP GET 请求。
* `GetProduct`操作包括 `"{id}"` 模板，因此 `id` 附加到 `"api/[controller]"` 控制器上的模板。 方法模板为 `"api/[controller]/"{id}""` 。 因此，此操作仅匹配对窗体、、等的 GET 请求 `/api/test2/xyz` `/api/test2/123` `/api/test2/{any string}` 。
  [!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet2)]
* `GetIntProduct`操作包含 `"int/{id:int}")` 模板。 `:int`模板的部分将 `id` 路由值限制为可以转换为整数的字符串。 针对以下内容的 GET 请求 `/api/test2/int/abc` ：
  * 与此操作不匹配。
  * 返回 " [找不到 404](https://developer.mozilla.org/docs/Web/HTTP/Status/404) " 错误。
    [!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet3)]
* `GetInt2Product`操作包含 `{id}` 在模板中，但不会限制为 `id` 可转换为整数的值。 针对以下内容的 GET 请求 `/api/test2/int2/abc` ：
  * 与此路由匹配。
  * 模型绑定无法转换 `abc` 为整数。 该 `id` 方法的参数是整数。
  * 返回 [400 错误请求](https://developer.mozilla.org/docs/Web/HTTP/Status/400) ，因为模型绑定无法转换 `abc` 为整数。
      [!code-csharp[](routing/samples/3.x/main/Controllers/Test2Controller.cs?name=snippet4)]

特性路由可以使用 <xref:Microsoft.AspNetCore.Mvc.Routing.HttpMethodAttribute> <xref:Microsoft.AspNetCore.Mvc.HttpPostAttribute> 、和等属性 <xref:Microsoft.AspNetCore.Mvc.HttpPutAttribute> <xref:Microsoft.AspNetCore.Mvc.HttpDeleteAttribute> 。 所有 [HTTP 谓词](#verb) 特性都接受路由模板。 下面的示例演示两个匹配同一路由模板的操作：

[!code-csharp[](routing/samples/3.x/main/Controllers/MyProductsController.cs?name=snippet1)]

使用 URL 路径 `/products3` ：

* `MyProductsController.ListProducts`当[HTTP 谓词](#verb)为时，该操作将运行 `GET` 。
* `MyProductsController.CreateProduct`当[HTTP 谓词](#verb)为时，该操作将运行 `POST` 。

构建 REST API 时，很少需要 `[Route(...)]` 在操作方法上使用，因为该操作接受所有 HTTP 方法。 更好的方法是使用更具体的 [HTTP 谓词特性](#verb) 来精确了解 API 支持的内容。 REST API 的客户端需要知道映射到特定逻辑操作的路径和 Http 谓词。

REST Api 应使用属性路由将应用功能建模为一组资源，其中的操作由 HTTP 谓词表示。 这意味着，多个操作（例如，同一逻辑资源的 GET 和 POST）使用相同的 URL。 属性路由提供了精心设计 API 的公共终结点布局所需的控制级别。

由于属性路由适用于特定操作，因此，使参数变成路由模板定义中的必需参数很简单。 在下面的示例中， `id` 需要作为 URL 路径的一部分：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsApiController.cs?name=snippet2)]

`Products2ApiController.GetProduct(int)`操作：

* 是用 URL 路径（如）运行的 `/products2/3`
* 不以 URL 路径运行 `/products2` 。

使用 [[Consumes]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) 属性，操作可以限制支持的请求内容类型。 有关详细信息，请参阅 [使用 "使用" 属性定义支持的请求内容类型](xref:web-api/index#consumes)。

 请参阅[路由](xref:fundamentals/routing)了解路由模板和相关选项的完整说明。

有关的详细信息 `[ApiController]` ，请参阅 [ApiController 属性](xref:web-api/index##apicontroller-attribute)。

## <a name="route-name"></a>路由名称

以下代码定义  的`Products_List`路由名称：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsApiController.cs?name=snippet2)]

可以使用路由名称基于特定路由生成 URL。 路由名称：

* 不会影响路由的 URL 匹配行为。
* 仅用于生成 URL。

路由名称必须在应用程序范围内唯一。

对比前面的代码和常规默认路由，该路由将 `id` 参数定义为可选的 (`{id?}`) 。 精确指定 Api 的功能具有多种优点，例如允许 `/products` 和 `/products/5` 调度到不同的操作。

<a name="routing-combining-ref-label"></a>

## <a name="combining-attribute-routes"></a>组合特性路由

若要使属性路由减少重复，可将控制器上的路由属性与各个操作上的路由属性合并。 控制器上定义的所有路由模板均作为操作上路由模板的前缀。 在控制器上放置路由属性会使控制器中的 **所有** 操作都使用属性路由。

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsApiController.cs?name=snippet)]

在上面的示例中：

* URL 路径 `/products` 可以匹配 `ProductsApi.ListProducts`
* URL 路径 `/products/5` 可以匹配 `ProductsApi.GetProduct(int)` 。

这两个操作仅匹配 HTTP， `GET` 因为它们用特性标记 `[HttpGet]` 。

应用于操作的以 `/` 或 `~/` 开头的路由模板不与应用于控制器的路由模板合并。 下面的示例匹配一组类似于默认路由的 URL 路径。

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet)]

下表说明了 `[Route]` 上述代码中的属性：

| Attribute               | 结合 `[Route("Home")]` | 定义路由模板 |
| ----------------- | ------------ | --------- |
| `[Route("")]` | 是 | `"Home"` |
| `[Route("Index")]` | 是 | `"Home/Index"` |
| `[Route("/")]` | 否 | `""` |
| `[Route("About")]` | 是 | `"Home/About"` |

<a name="routing-ordering-ref-label"></a>
<a name="oar"></a>

### <a name="attribute-route-order"></a>属性路由顺序

路由构建树并同时匹配所有终结点：

* 路由条目的行为方式与置于理想排序中的行为相同。
* 最特定的路由在更通用的路由之前有机会执行。

例如，类似于属性路由的属性路由 `blog/search/{topic}` 更为具体 `blog/{*article}` 。 `blog/search/{topic}`默认情况下，路由具有更高的优先级，因为它更为具体。 使用 [传统路由](#cr)，开发人员负责按所需的顺序放置路由。

属性路由可以使用属性配置订单 <xref:Microsoft.AspNetCore.Mvc.RouteAttribute.Order> 。 提供的所有 [路由属性](xref:Microsoft.AspNetCore.Mvc.RouteAttribute) 都包括 `Order` 。 路由按 `Order` 属性的升序进行处理。 默认顺序为 `0`。 使用 `Order = -1` 不设置顺序的路由之前的运行设置路由。 使用 `Order = 1` 默认路由排序后的运行设置路由。

**避免** 依赖于 `Order` 。 如果应用的 URL 空间需要显式顺序值才能正确路由，则很可能会使客户端混淆。 通常，属性路由选择 URL 匹配的正确路由。 如果用于 URL 生成的默认顺序不起作用，则使用路由名称作为替代通常比应用属性更简单 `Order` 。

请考虑以下两个控制器，它们都定义了路由匹配 `/home` ：

[!code-csharp[](routing/samples/3.x/main/Controllers/HomeController.cs?name=snippet2)]

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemoController.cs?name=snippet)]

`/home`使用前面的代码请求将引发异常，如下所示：

```text
AmbiguousMatchException: The request matched multiple endpoints. Matches:

 WebMvcRouting.Controllers.HomeController.Index
 WebMvcRouting.Controllers.MyDemoController.MyIndex
```

添加 `Order` 到其中一个路由属性可解决歧义：

[!code-csharp[](routing/samples/3.x/main/Controllers/MyDemo3Controller.cs?name=snippet3& highlight=2)]

在前面的代码中， `/home` 运行 `HomeController.Index` 终结点。 若要转到 `MyDemoController.MyIndex` ，请请求 `/home/MyIndex` 。 **注意** ：

* 上面的代码是一个示例或不良路由设计。 它用于说明 `Order` 属性。
* `Order`属性只解析多义性，该模板无法匹配。 最好删除该 `[Route("Home")]` 模板。

请参阅[ Razor 页面路由和应用约定：路由](xref:razor-pages/razor-pages-conventions#route-order)顺序获取有关页面的路由顺序的信息 Razor 。

在某些情况下，将返回具有不明确路由的 HTTP 500 错误。 使用 [日志记录](xref:fundamentals/logging/index) 查看导致的终结点 `AmbiguousMatchException` 。

<a name="routing-token-replacement-templates-ref-label"></a>

## <a name="token-replacement-in-route-templates-controller-action-area"></a>路由模板中的标记替换 [控制器]，[操作]，[区域]

为方便起见，特性路由支持为保留路由参数替换标记，方法是将令牌括在以下其中一项：

* 方括号： `[]`
* 大括号： `{}`

令牌 `[action]` 、 `[area]` 和 `[controller]` 将替换为自定义路由的操作的 "操作名称"、"区域名称" 和 "控制器名称" 的值：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet)]

在上述代码中：

  [!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet10)]

  * 与 `/Products0/List`

  [!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet11)]

  * 与 `/Products0/Edit/{id}`

标记替换发生在属性路由生成的最后一步。 前面的示例与下面的代码具有相同的行为：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet20)]

[!INCLUDE[](~/includes/MTcomments.md)]

属性路由还可以与继承结合使用。 这与标记替换功能功能强大。 标记替换也适用于属性路由定义的路由名称。
`[Route("[controller]/[action]", Name="[controller]_[action]")]`为每个操作生成唯一的路由名称：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet5)]

标记替换也适用于属性路由定义的路由名称。
`[Route("[controller]/[action]", Name="[controller]_[action]")]`
 为每项操作生成一个唯一的路由名称。

若要匹配文本标记替换分隔符 `[` 或 `]`，可通过重复该字符（`[[` 或 `]]`）对其进行转义。

<a name="routing-token-replacement-transformers-ref-label"></a>

### <a name="use-a-parameter-transformer-to-customize-token-replacement"></a>使用参数转换程序自定义标记替换

使用参数转换程序可以自定义标记替换。 参数转换程序实现 <xref:Microsoft.AspNetCore.Routing.IOutboundParameterTransformer> 并转换参数值。 例如，自定义 `SlugifyParameterTransformer` 参数转换器将 `SubscriptionManagement` 路由值更改为 `subscription-management` ：

[!code-csharp[](routing/samples/3.x/main/StartupSlugifyParamTransformer.cs?name=snippet2)]

<xref:Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention> 是应用程序模型约定，可以：

* 将参数转换程序应用到应用程序中的所有属性路由。
* 在替换属性路由标记值时对其进行自定义。

[!code-csharp[](routing/samples/3.x/main/Controllers/SubscriptionManagementController.cs?name=snippet)]

前面的 `ListAll` 方法匹配 `/subscription-management/list-all` 。

`RouteTokenTransformerConvention` 在 `ConfigureServices` 中注册为选项。

[!code-csharp[](routing/samples/3.x/main/StartupSlugifyParamTransformer.cs?name=snippet)]

有关信息区的定义，请参阅 [网站上的 MDN web 文档](https://developer.mozilla.org/docs/Glossary/Slug) 。

[!INCLUDE[](~/includes/regex.md)]
<a name="routing-multiple-routes-ref-label"></a>

### <a name="multiple-attribute-routes"></a>多个属性路由

属性路由支持定义多个访问同一操作的路由。 此操作最常用于模拟默认传统路由的行为，如以下示例所示：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet6x)]

在控制器上放置多个路由属性意味着每个属性都与操作方法上的每个路由属性结合：

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet6)]

所有 [HTTP 谓词](#verb) 路由约束都实现 `IActionConstraint` 。

当实现的多个路由属性 <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> 放置在一个操作上时：

* 每个操作约束都与应用于控制器的路由模板结合。

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet7)]

在操作中使用多个路由可能看起来非常有用，更好的做法是让应用程序的 URL 空间基本且定义完善。 **仅** 在需要时对操作使用多个路由，例如支持现有客户端。

<a name="routing-attr-options"></a>

### <a name="specifying-attribute-route-optional-parameters-default-values-and-constraints"></a>指定属性路由的可选参数、默认值和约束

属性路由支持使用与传统路由相同的内联语法，来指定可选参数、默认值和约束。

[!code-csharp[](routing/samples/3.x/main/Controllers/ProductsController.cs?name=snippet8&highlight=3)]

在上面的代码中， `[HttpPost("product/{id:int}")]` 应用路由约束。 此 `ProductsController.ShowProduct` 操作仅由类似的 URL 路径进行匹配 `/product/3` 。 路由模板部分 `{id:int}` 仅限制整数。

有关路由模板语法的详细说明，请参阅[路由模板参考](xref:fundamentals/routing#route-template-reference)。

<a name="routing-cust-rt-attr-irt-ref-label"></a>

### <a name="custom-route-attributes-using-iroutetemplateprovider"></a>使用 IRouteTemplateProvider 的自定义路由属性

所有 [路由特性](#rt) 都实现 <xref:Microsoft.AspNetCore.Mvc.Routing.IRouteTemplateProvider> 。 ASP.NET Core 运行时：

* 应用启动时，在控制器类和操作方法上查找属性。
* 使用实现 `IRouteTemplateProvider` 以生成初始路由集的特性。

实现 `IRouteTemplateProvider` 以定义自定义路由属性。 每个 `IRouteTemplateProvider` 都允许定义一个包含自定义路由模板、顺序和名称的路由：

[!code-csharp[](routing/samples/3.x/main/Controllers/MyTestApiController.cs?name=snippet&highlight=1-10)]

上述 `Get` 方法返回 `Order = 2, Template = api/MyTestApi` 。

<a name="routing-app-model-ref-label"></a>

### <a name="use-application-model-to-customize-attribute-routes"></a>使用应用程序模型自定义属性路由

应用程序模型：

* 是在启动时创建的对象模型。
* 包含 ASP.NET Core 用来在应用程序中路由和执行操作的所有元数据。

应用程序模型包括从路由属性收集的所有数据。 路由属性中的数据由 `IRouteTemplateProvider` 实现提供。 规范

* 可以编写来修改应用程序模型，以自定义路由的行为方式。
* 在应用程序启动时读取。

本部分显示了使用应用程序模型自定义路由的基本示例。 下面的代码使路由大致与项目的文件夹结构对齐。

[!code-csharp[](routing/samples/3.x/nsrc/NamespaceRoutingConvention.cs?name=snippet)]

下面的代码阻止将 `namespace` 约定应用到已路由属性的控制器：

[!code-csharp[](routing/samples/3.x/nsrc/NamespaceRoutingConvention.cs?name=snippet2)]

例如，以下控制器不使用 `NamespaceRoutingConvention` ：

[!code-csharp[](routing/samples/3.x/nsrc/Controllers/ManagersController.cs?name=snippet&highlight=1)]

`NamespaceRoutingConvention.Apply` 方法：

* 如果控制器为属性路由，则不执行任何操作。
* 基于删除基的控制器模板 `namespace` `namespace` 。

`NamespaceRoutingConvention`可应用于 `Startup.ConfigureServices` ：

[!code-csharp[](routing/samples/3.x/nsrc/Startup.cs?name=snippet&highlight=1,14-18)]

例如，请考虑以下控制器：

[!code-csharp[](routing/samples/3.x/nsrc/Controllers/UsersController.cs)]

在上述代码中：

* 基数 `namespace` 是 `My.Application` 。
* 前面的控制器的全名为 `My.Application.Admin.Controllers.UsersController` 。
* 将 `NamespaceRoutingConvention` 控制器模板设置为 `Admin/Controllers/Users/[action]/{id?` 。

`NamespaceRoutingConvention`还可以应用为控制器上的属性：

[!code-csharp[](routing/samples/3.x/nsrc/Controllers/TestController.cs?name=snippet&highlight=1)]

<a name="routing-mixed-ref-label"></a>

## <a name="mixed-routing-attribute-routing-vs-conventional-routing"></a>混合路由：属性路由与传统路由

ASP.NET Core 应用可以混合使用传统路由和属性路由。 通常将传统路由用于为浏览器处理 HTML 页面的控制器，将属性路由用于处理 REST API 的控制器。

操作既支持传统路由，也支持属性路由。 通过在控制器或操作上放置路由可实现属性路由。 不能通过传统路由访问定义属性路由的操作，反之亦然。 控制器上的 **任何** 路由属性都使控制器属性中的 **所有操作都** 已路由。

属性路由和传统路由使用相同的路由引擎。

<a name="routing-url-gen-ref-label"></a>
<a name="ambient"></a>

## <a name="url-generation-and-ambient-values"></a>URL 生成和环境值

应用可以使用路由 URL 生成功能来生成指向操作的 URL 链接。 生成 Url 可消除硬编码 Url，使代码更可靠和更易于维护。 本部分重点介绍 MVC 提供的 URL 生成功能，仅介绍 URL 生成的工作原理的基础知识。 有关 URL 生成的详细说明，请参阅[路由](xref:fundamentals/routing)。

<xref:Microsoft.AspNetCore.Mvc.IUrlHelper>接口是 MVC 之间的基础结构和用于生成 URL 的路由的基础元素。 的实例 `IUrlHelper` 可通过 " `Url` 控制器"、"视图" 和 "视图" 组件中的属性使用。

在下面的示例中， `IUrlHelper` 通过属性使用接口 `Controller.Url` 生成其他操作的 URL。

[!code-csharp[](routing/samples/3.x/main/Controllers/UrlGenerationController.cs?name=snippet_1)]

如果应用使用默认传统路由，则变量的值 `url` 为 URL 路径字符串 `/UrlGeneration/Destination` 。 此 URL 路径由路由通过组合创建：

* 当前请求中的路由值，称为 " **环境值** "。
* 传递到的值 `Url.Action` ，并将这些值替换为路由模板：

``` text
ambient values: { controller = "UrlGeneration", action = "Source" }
values passed to Url.Action: { controller = "UrlGeneration", action = "Destination" }
route template: {controller}/{action}/{id?}

result: /UrlGeneration/Destination
```

路由模板中的每个路由参数都会通过将名称与这些值和环境值匹配，来替换掉原来的值。 不具有值的路由参数可以：

* 如果有一个默认值，则使用默认值。
* 如果是可选的，则跳过它。 例如， `id` 路由模板中的 `{controller}/{action}/{id?}` 。

如果任何所需的路由参数没有对应的值，URL 生成将失败。 如果某个路由的 URL 生成失败，则尝试下一个路由，直到尝试所有路由或找到匹配项为止。

前面的示例 `Url.Action` 假定 [传统路由](#cr)。 URL 生成的工作方式类似于 [属性路由](#ar)，但概念不同。 对于传统路由：

* 路由值用于扩展模板。
* 和的路由值 `controller` `action` 通常出现在该模板中。 这是因为由路由匹配的 Url 遵循约定。

下面的示例使用属性路由：

[!code-csharp[](routing/samples/3.x/main/Controllers/UrlGenerationAttrController.cs?name=snippet_1)]

`Source`上述代码中的操作生成 `custom/url/to/destination` 。

<xref:Microsoft.AspNetCore.Routing.LinkGenerator> 已添加到 ASP.NET Core 3.0 中作为的替代项 `IUrlHelper` 。 `LinkGenerator` 提供类似但更灵活的功能。 上的每个方法 `IUrlHelper` 也具有相应的一系列方法 `LinkGenerator` 。

### <a name="generating-urls-by-action-name"></a>根据操作名称生成 URL

[LinkGenerator、GetPathByAction](xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*)和所有相关重载都旨在通过指定控制器名称和操作名称来生成目标终结[点。](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*)

使用时 `Url.Action` ，和的当前路由值 `controller` `action` 由运行时提供：

* 和的值 `controller` `action` 都属于 [环境值](#ambient) 和值。 方法 `Url.Action` 始终使用和的当前值， `action` `controller` 并生成路由到当前操作的 URL 路径。

路由尝试使用环境值中的值来填充生成 URL 时未提供的信息。 请考虑类似于 `{a}/{b}/{c}/{d}` 环境值的路由 `{ a = Alice, b = Bob, c = Carol, d = David }` ：

* 路由具有足够的信息来生成 URL，无需任何其他值。
* 路由具有足够的信息，因为所有路由参数都具有值。

如果添加了值 `{ d = Donovan }` ：

* 值 `{ d = David }` 被忽略。
* 生成的 URL 路径为 `Alice/Bob/Carol/Donovan` 。

**警告** ： URL 路径是分层的。 在前面的示例中，如果添加了值 `{ c = Cheryl }` ：

* 这两个值 `{ c = Carol, d = David }` 都将被忽略。
* 不再存在的值 `d` ，URL 生成将失败。
* `c` `d` 若要生成 URL，必须指定和的所需值。  

你可能希望在默认路由中遇到此问题 `{controller}/{action}/{id?}` 。 此问题在实践中很罕见，因为 `Url.Action` 始终显式指定 `controller` 和 `action` 值。

多个 [Url 重载。操作](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*) 采用路由值对象为除和以外的路由参数提供值 `controller` `action` 。 路由值对象经常与一起使用 `id` 。 例如 `Url.Action("Buy", "Products", new { id = 17 })`。 路由值对象：

* 按约定通常是匿名类型的对象。
* 可以是 `IDictionary<>` 或 [POCO](https://wikipedia.org/wiki/Plain_old_CLR_object)) 。

任何与路由参数不匹配的附加路由值都放在查询字符串中。

[!code-csharp[](routing/samples/3.x/main/Controllers/TestController.cs?name=snippet)]

前面的代码生成 `/Products/Buy/17?color=red` 。

下面的代码生成一个绝对 URL：

[!code-csharp[](routing/samples/3.x/main/Controllers/TestController.cs?name=snippet2)]

若要创建绝对 URL，请使用以下项之一：

* 接受的重载 `protocol` 。 例如，前面的代码。
* 默认情况下， [LinkGenerator](xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*)生成绝对 uri。

<a name="routing-gen-urls-route-ref-label"></a>

### <a name="generate-urls-by-route"></a>按路由生成 Url

前面的代码演示了如何通过传入控制器和操作名称来生成 URL。 `IUrlHelper` 还提供了 [RouteUrl](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.RouteUrl*) 系列方法。 这些方法类似于 [Url. 操作](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*)，但它们不会将和的当前值 `action` 复制 `controller` 到路由值。 最常见的用法是 `Url.RouteUrl` ：

* 指定用于生成 URL 的路由名称。
* 通常不指定控制器或操作名称。

[!code-csharp[](routing/samples/3.x/main/Controllers/UrlGeneration2Controller.cs?name=snippet_1)]

下面的 Razor 文件生成一个到的 HTML 链接 `Destination_Route` ：

[!code-cshtml[](routing/samples/3.x/main/Views/Shared/MyLink.cshtml)]

<a name="routing-gen-urls-html-ref-label"></a>

### <a name="generate-urls-in-html-and-no-locrazor"></a>在 HTML 和中生成 Url Razor

<xref:Microsoft.AspNetCore.Mvc.Rendering.IHtmlHelper> 提供 <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.HtmlHelper> 方法 [Html.beginform](xref:Microsoft.AspNetCore.Mvc.Rendering.IHtmlHelper.BeginForm*) 和 [html.actionlink](xref:Microsoft.AspNetCore.Mvc.Rendering.IHtmlHelper.ActionLink*) 分别生成 `<form>` 和元素的方法 `<a>` 。 这些方法使用 [Url 操作](xref:Microsoft.AspNetCore.Mvc.IUrlHelper.Action*) 方法来生成 url，并接受类似参数。 `HtmlHelper` 的配套 `Url.RouteUrl` 为 `Html.BeginRouteForm` 和 `Html.RouteLink`，两者具有相似的功能。

TagHelper 通过 `form` TagHelper 和 `<a>` TagHelper 生成 URL。 两者均通过 `IUrlHelper` 来实现。 有关详细信息，请参阅 [窗体中的标记帮助](xref:mvc/views/working-with-forms) 程序。

在视图内，可通过 `Url` 属性将 `IUrlHelper` 用于前文未涵盖的任何临时 URL 生成。

<a name="routing-gen-urls-action-ref-label"></a>

### <a name="url-generation-in-action-results"></a>操作结果中的 URL 生成

前面的示例演示了如何 `IUrlHelper` 在控制器中使用。 控制器中最常见的用法是将 URL 生成为操作结果的一部分。

<xref:Microsoft.AspNetCore.Mvc.ControllerBase> 和 <xref:Microsoft.AspNetCore.Mvc.Controller> 基类为操作结果提供简便的方法来引用另一项操作。 一种典型用法是在接受用户输入后重定向：

[!code-csharp[](routing/samples/3.x/main/Controllers/CustomerController.cs?name=snippet)]

操作将生成工厂方法（如 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.RedirectToAction%2A> 和），并 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction%2A> 遵循中的方法 `IUrlHelper` 。

<a name="routing-dedicated-ref-label"></a>

### <a name="special-case-for-dedicated-conventional-routes"></a>专用传统路由的特殊情况

[传统路由](#cr) 可以使用一种特殊的路由定义，称为 [专用的传统路由](#dcr)。 在以下示例中，名为的路由 `blog` 是一个专用的传统路由：

[!code-csharp[](routing/samples/3.x/main/Startup.cs?name=snippet_1)]

使用前面的路由定义， `Url.Action("Index", "Home")` 使用路由生成 URL 路径 `/` `default` ，但为什么要这样做呢？ 用户可能认为使用 `blog`，路由值 `{ controller = Home, action = Index }` 就足以生成 URL，且结果为 `/blog?action=Index&controller=Home`。

[专用传统路由](#dcr) 依赖于默认值的特殊行为，这些默认值没有相应的路由参数可防止 [路由过于被](xref:fundamentals/routing#greedy) URL 生成。 在此例中，默认值是为 `{ controller = Blog, action = Article }`，`controller` 和 `action` 均未显示为路由参数。 当路由执行 URL 生成时，提供的值必须与默认值匹配。 使用的 URL 生成 `blog` 失败，因为这些值 `{ controller = Home, action = Index }` 不匹配 `{ controller = Blog, action = Article }` 。 然后，路由回退，尝试使用 `default`，并最终成功。

<a name="routing-areas-ref-label"></a>

## <a name="areas"></a>Areas

[区域](xref:mvc/controllers/areas) 是一项 MVC 功能，用于将相关功能作为一个单独的组组织到一个组中：

* 控制器操作的路由命名空间。
* 视图的文件夹结构。

通过使用区域，应用可以有多个具有相同名称的控制器，只要它们具有不同的区域即可。 通过向 `controller` 和 `action` 添加另一个路由参数 `area`，可使用区域为路由创建层次结构。 本部分讨论路由如何与区域交互。 有关如何将区域与视图结合使用的详细信息，请参阅 [区域](xref:mvc/controllers/areas) 。

下面的示例将 MVC 配置为使用默认传统路由，并为 `area` 命名的 `area` 指定路由 `Blog` ：

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet1)]

在前面的代码中， <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapAreaControllerRoute%2A> 将调用来创建 `"blog_route"` 。 第二个参数 `"Blog"` 为区域名称。

当匹配 URL 路径（如 `/Manage/Users/AddUser` ）时， `"blog_route"` 路由将生成路由值 `{ area = Blog, controller = Users, action = AddUser }` 。 `area`路由值由的默认值生成 `area` 。 创建的路由 `MapAreaControllerRoute` 等效于以下内容：

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup2.cs?name=snippet2)]

`MapAreaControllerRoute` 通过为使用所提供的区域名称（本例中为 `Blog`）的 `area` 提供默认值和约束，来创建路由。 默认值确保路由始终生成 `{ area = Blog, ... }`，约束要求在生成 URL 时使用值 `{ area = Blog, ... }`。

传统路由依赖于顺序。 通常情况下，应将具有区域的路由置于更早的位置，因为它们比没有区域的路由更具体。

使用前面的示例，路由值 `{ area = Blog, controller = Users, action = AddUser }` 与以下操作匹配：

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

[[Area]](xref:Microsoft.AspNetCore.Mvc.AreaAttribute)特性用于将控制器表示为区域的一部分。 此控制器在区域中 `Blog` 。 没有属性的控制器 `[Area]` 不是任何区域的成员，并且在 **not** `area` 路由值由路由提供时不匹配。 在下面的示例中，只有所列出的第一个控制器才能与路由值 `{ area = Blog, controller = Users, action = AddUser }` 匹配。

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Zebra/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Controllers/UsersController.cs)]

此处显示每个控制器的命名空间，以获取完整性。 如果前面的控制器使用相同的命名空间，则会生成编译器错误。 类命名空间对 MVC 的路由没有影响。

前两个控制器是区域成员，仅在 `area` 路由值提供其各自的区域名称时匹配。 第三个控制器不是任何区域的成员，只能在路由没有为 `area` 提供任何值时匹配。

<a name="aa"></a>

就 *不匹配任何值* 而言，缺少 `area` 值相当于 `area` 的值为 NULL 或空字符串。

在区域内执行操作时，的路由值 `area` 可用作路由用于生成 URL 的 [环境值](#ambient) 。 这意味着默认情况下，区域在 URL 生成中具有 *粘性* ，如以下示例所示。

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup3.cs?name=snippet3)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Duck/Controllers/UsersController.cs)]

下面的代码生成一个 URL，用于 `/Zebra/Users/AddUser` ：

[!code-csharp[](routing/samples/3.x/AreasRouting/Controllers/HomeController.cs?name=snippet)]

<a name="action"></a>

## <a name="action-definition"></a>操作定义

控制器上的公共方法（具有 [NonAction](xref:Microsoft.AspNetCore.Mvc.NonActionAttribute) 特性的方法除外）是操作。

## <a name="sample-code"></a>代码示例

 * [示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/routing/samples/3.x)中包含了[MyDisplayRouteInfo](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/routing/samples/3.x/main/Extensions/ControllerContextExtensions.cs)方法，用于显示路由信息。
* [查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/routing/samples/3.x)（[如何下载](xref:index#how-to-download-a-sample)）

[!INCLUDE[](~/includes/dbg-route.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core MVC 使用路由[中间件](xref:fundamentals/middleware/index)来匹配传入请求的 URL 并将它们映射到操作。 路由在启动代码或属性中定义。 路由描述应如何将 URL 路径与操作相匹配。 它还用于在响应中生成送出的 URL（用于链接）。

操作既支持传统路由，也支持属性路由。 通过在控制器或操作上放置路由可实现属性路由。 有关详细信息，请参阅[混合路由](#routing-mixed-ref-label)。

本文档将介绍 MVC 与路由之间的交互，以及典型的 MVC 应用如何使用各种路由功能。 有关高级路由的详细信息，请参阅[路由](xref:fundamentals/routing)。

## <a name="setting-up-routing-middleware"></a>设置路由中间件

在 *Configure* 方法中，可能会看到与下面类似的代码：

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

在对 `UseMvc` 的调用中，`MapRoute` 用于创建单个路由，亦称 `default` 路由。 大多数 MVC 应用使用带有模板的路由，与 `default` 路由类似。

路由模板 `"{controller=Home}/{action=Index}/{id?}"` 可以匹配诸如 `/Products/Details/5` 之类的 URL 路径，并通过对路径进行标记来提取路由值 `{ controller = Products, action = Details, id = 5 }`。 MVC 将尝试查找名为 `ProductsController` 的控制器并运行 `Details` 操作：

```csharp
public class ProductsController : Controller
{
   public IActionResult Details(int id) { ... }
}
```

请注意，在此示例中，当调用此操作时，模型绑定会使用值 `id = 5` 将 `id` 参数设置为 `5`。 有关更多详细信息，请参阅[模型绑定](../models/model-binding.md)。

使用 `default` 路由：

```csharp
routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

路由模板：

* `{controller=Home}` 将 `Home` 定义为默认 `controller`

* `{action=Index}` 将 `Index` 定义为默认 `action`

* `{id?}` 将 `id` 定义为可选参数

默认路由参数和可选路由参数不必包含在 URL 路径中进行匹配。 有关路由模板语法的详细说明，请参阅[路由模板参考](xref:fundamentals/routing#route-template-reference)。

`"{controller=Home}/{action=Index}/{id?}"` 可以匹配 URL 路径 `/` 并生成路由值 `{ controller = Home, action = Index }`。 `controller` 和 `action` 的值使用默认值，`id` 不生成值，因为 URL 路径中没有相应的段。 MVC 使用这些路由值选择 `HomeController` 和 `Index` 操作：

```csharp
public class HomeController : Controller
{
  public IActionResult Index() { ... }
}
```

通过使用此控制器定义和路由模板，将对以下任意 URL 路径执行 `HomeController.Index` 操作：

* `/Home/Index/17`

* `/Home/Index`

* `/Home`

* `/`

简便方法 `UseMvcWithDefaultRoute`：

```csharp
app.UseMvcWithDefaultRoute();
```

可用于替换：

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

`UseMvc` 和 `UseMvcWithDefaultRoute` 可向中间件管道添加 `RouterMiddleware` 的实例。 MVC 不直接与中间件交互，而是使用路由来处理请求。 MVC 通过 `MvcRouteHandler` 实例连接到路由。 `UseMvc` 内的代码与下面类似：

```csharp
var routes = new RouteBuilder(app);

// Add connection to MVC, will be hooked up by calls to MapRoute.
routes.DefaultHandler = new MvcRouteHandler(...);

// Execute callback to register routes.
// routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");

// Create route collection and add the middleware.
app.UseRouter(routes.Build());
```

`UseMvc` 不直接定义任何路由，它向 `attribute` 路由的路由集合添加占位符。 重载 `UseMvc(Action<IRouteBuilder>)` 则允许用户添加自己的路由，并且还支持属性路由。  `UseMvc` 及其所有变体都会为属性路由添加占位符：无论如何配置 `UseMvc`，属性路由始终可用。 `UseMvcWithDefaultRoute` 定义默认路由并支持属性路由。 [属性路由](#attribute-routing-ref-label)部分提供了有关属性路由的更多详细信息。

<a name="routing-conventional-ref-label"></a>

## <a name="conventional-routing"></a>传统路由

`default` 路由：

```csharp
routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

前面的代码是传统路由的示例。 此样式称为传统路由，因为它建立了一个 URL 路径 *约定* ：

* 第一个路径段映射到控制器名称。
* 第二个映射到操作名称。
* 第三段用于可选 `id` 。 `id` 映射到模型实体。

使用此 `default` 路由时，URL 路径 `/Products/List` 映射到 `ProductsController.List` 操作，`/Blog/Article/17` 映射到 `BlogController.Article`。 此映射 **仅** 基于控制器和操作名称，而不基于命名空间、源文件位置或方法参数。

> [!TIP]
> 使用默认路由进行传统路由时，可快速生成应用程序，无需为所定义的每项操作提供一个新的 URL 模式。 对于包含 CRUD 样式操作的应用程序，通过保持各控制器间 URL 的一致性，可帮助简化代码，使 UI 更易预测。

> [!WARNING]
> 路由模板将 `id` 定义为可选参数，意味着无需在 URL 中提供 ID 也可执行操作。 从 URL 中省略 `id` 通常会导致模型绑定将它设置为 `0`，进而导致在数据库中找不到与 `id == 0` 匹配的实体。 属性路由可以提供细化控制，使某些操作需要 ID，某些操作不需要 ID。 按照惯例，当可选参数（比如 `id`）有可能在正确的用法中出现时，本文档将涵盖这些参数。

## <a name="multiple-routes"></a>多个路由

通过添加对 `MapRoute` 的多次调用，可以在 `UseMvc` 内添加多个路由。 这样做可以定义多个约定，或添加专用于特定操作的传统路由，比如：

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("blog", "blog/{*article}",
            defaults: new { controller = "Blog", action = "Article" });
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

此处的 `blog` 路由是一个 *专用的传统路由* ，这表示它使用传统路由系统，但专用于特定的操作。 由于 `controller` 和 `action` 不会在路由模板中作为参数显示，它们只能有默认值，因此，此路由将始终映射到 `BlogController.Article` 操作。

路由集合中的路由会进行排序，并按添加顺序进行处理。 因此，在此示例中，将先尝试 `blog` 路由，再尝试 `default` 路由。

> [!NOTE]
> *专用传统路由* 通常使用全部 **捕获** 路由参数 `{*article}` 来捕获 URL 路径的其余部分。 这会使某个路由变得“太贪婪”，也就是说，它会匹配用户想要使用其他路由来匹配的 URL。 将“贪婪的”路由放在路由表中靠后的位置可解决此问题。

### <a name="fallback"></a>回退

在处理请求时，MVC 将验证路由值能否用于在应用程序中查找控制器和操作。 如果路由值与任何操作都不匹配，则将该路由视为不匹配，并尝试下一个路由。 这称为 *回退* ，其目的是简化传统路由重叠的情况。

### <a name="disambiguating-actions"></a>区分操作

当通过路由匹配到两项操作时，MVC 必须进行区分，以选择“最佳”候选项，否则会引发异常。 例如：

```csharp
public class ProductsController : Controller
{
   public IActionResult Edit(int id) { ... }

   [HttpPost]
   public IActionResult Edit(int id, Product product) { ... }
}
```

此控制器定义了两项操作，这两项操作均与 URL 路径 `/Products/Edit/17` 和路由数据 `{ controller = Products, action = Edit, id = 17 }` 匹配。 这是 MVC 控制器的典型模式，其中 `Edit(int)` 显示用于编辑产品的表单，`Edit(int, Product)` 处理已发布的表单。 为此，MVC 需要在请求为 HTTP `POST` 时选择 `Edit(int, Product)`，在 Http 谓词为任何其他内容时选择 `Edit(int)`。

`HttpPostAttribute` ( `[HttpPost]` ) 是 `IActionConstraint` 的实现，它仅允许执行当 Http 谓词为 `POST` 时选择的操作。 `IActionConstraint` 的存在使 `Edit(int, Product)` 成为比 `Edit(int)`“更好”的匹配项，因此会先尝试 `Edit(int, Product)`。

只需在特殊化方案中编写自定义 `IActionConstraint` 实现，但务必了解 `HttpPostAttribute` 等属性的角色 — 为其他 Http 谓词定义了类似的属性。 在传统路由中，当操作属于 `show form -> submit form` 工作流时通常使用相同的操作名称。 在阅读[了解 IActionConstraint](#understanding-iactionconstraint) 部分后，此模式的便利性将变得更加明显。

如果匹配多个路由，但 MVC 找不到“最佳”路由，则会引发 `AmbiguousActionException`。

<a name="routing-route-name-ref-label"></a>

### <a name="route-names"></a>路由名称

以下示例中的字符串 `"blog"` 和 `"default"` 都是路由名称：

```csharp
app.UseMvc(routes =>
{
   routes.MapRoute("blog", "blog/{*article}",
               defaults: new { controller = "Blog", action = "Article" });
   routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

路由名称为路由提供一个逻辑名称，以便使用命名路由来生成 URL。 路由排序会使 URL 生成复杂化，而这极大地简化了 URL 创建。 路由名称必须在应用程序范围内唯一。

路由名称不影响请求的 URL 匹配或处理；它们仅用于 URL 生成。 [路由](xref:fundamentals/routing)提供了有关 URL 生成（包括 MVC 特定帮助程序中的 URL 生成）的更多详细信息。

<a name="attribute-routing-ref-label"></a>

## <a name="attribute-routing"></a>属性路由

属性路由使用一组属性将操作直接映射到路由模板。 在下面的示例中，`Configure` 方法使用 `app.UseMvc();`，不传递任何路由。 `HomeController` 将匹配一组 URL，这组 URL 与默认路由 `{controller=Home}/{action=Index}/{id?}` 匹配的 URL 类似：

```csharp
public class HomeController : Controller
{
   [Route("")]
   [Route("Home")]
   [Route("Home/Index")]
   public IActionResult Index()
   {
      return View();
   }
   [Route("Home/About")]
   public IActionResult About()
   {
      return View();
   }
   [Route("Home/Contact")]
   public IActionResult Contact()
   {
      return View();
   }
}
```

将针对任意 URL 路径 `/`、`/Home` 或 `/Home/Index` 执行 `HomeController.Index()` 操作。

> [!NOTE]
> 此示例重点介绍属性路由与传统路由之间的主要编程差异。 属性路由需要更多输入来指定路由；传统的默认路由处理路由的方式则更简洁。 但是，属性路由允许（并需要）精确控制应用于每项操作的路由模板。

使用属性路由时，控制器名称和操作名称对于操作的选择 **没有** 影响。 此示例匹配的 URL 与上一示例相同。

```csharp
public class MyDemoController : Controller
{
   [Route("")]
   [Route("Home")]
   [Route("Home/Index")]
   public IActionResult MyIndex()
   {
      return View("Index");
   }
   [Route("Home/About")]
   public IActionResult MyAbout()
   {
      return View("About");
   }
   [Route("Home/Contact")]
   public IActionResult MyContact()
   {
      return View("Contact");
   }
}
```

> [!NOTE]
> 上述路由模板未定义 `action`、`area` 和 `controller` 的路由参数。 事实上，属性路由中不允许使用这些路由参数。 由于路由模板已与某项操作关联，因此，分析 URL 中的操作名称毫无意义。

## <a name="attribute-routing-with-httpverb-attributes"></a>使用 Http[Verb] 属性的属性路由

属性路由还可以使用 `Http[Verb]` 属性，比如 `HttpPostAttribute`。 所有这些属性都可采用路由模板。 此示例展示与同一路由模板匹配的两项操作：

```csharp
[HttpGet("/products")]
public IActionResult ListProducts()
{
   // ...
}

[HttpPost("/products")]
public IActionResult CreateProduct(...)
{
   // ...
}
```

对于诸如 `/products` 之类的 URL 路径，当 Http 谓词为 `GET` 时将执行 `ProductsApi.ListProducts` 操作，当 Http 谓词为 `POST` 时将执行 `ProductsApi.CreateProduct`。 属性路由首先将 URL 与路由属性定义的路由模板集进行匹配。 一旦某个路由模板匹配，就会应用 `IActionConstraint` 约束来确定可以执行的操作。

> [!TIP]
> 生成 REST API 时，很少会在操作方法上使用 `[Route(...)]`这是因为该操作将接受所有 HTTP 方法。 建议使用更特定的 `Http*Verb*Attributes` 来明确 API 所支持的操作。 REST API 的客户端需要知道映射到特定逻辑操作的路径和 Http 谓词。

由于属性路由适用于特定操作，因此，使参数变成路由模板定义中的必需参数很简单。 在此示例中，`id` 是 URL 路径中的必需参数。

```csharp
public class ProductsApiController : Controller
{
   [HttpGet("/products/{id}", Name = "Products_List")]
   public IActionResult GetProduct(int id) { ... }
}
```

将针对诸如 `/products/3`（而非 `/products`）之类的 URL 路径执行 `ProductsApi.GetProduct(int)` 操作。 请参阅[路由](xref:fundamentals/routing)了解路由模板和相关选项的完整说明。

## <a name="route-name"></a>路由名称

以下代码定义 `Products_List` 的 *路由名称* ：

```csharp
public class ProductsApiController : Controller
{
   [HttpGet("/products/{id}", Name = "Products_List")]
   public IActionResult GetProduct(int id) { ... }
}
```

可以使用路由名称基于特定路由生成 URL。 路由名称不影响路由的 URL 匹配行为，仅用于生成 URL。 路由名称必须在应用程序范围内唯一。

> [!NOTE]
> 这一点与传统的 *默认路由* 相反，后者将 `id` 参数定义为可选参数 (`{id?}`)。 这种精确指定 API 的功能可带来一些好处，比如允许将 `/products` 和 `/products/5` 分派到不同的操作。

<a name="routing-combining-ref-label"></a>

### <a name="combining-routes"></a>合并路由

若要使属性路由减少重复，可将控制器上的路由属性与各个操作上的路由属性合并。 控制器上定义的所有路由模板均作为操作上路由模板的前缀。 在控制器上放置路由属性会使控制器中的 **所有** 操作都使用属性路由。

```csharp
[Route("products")]
public class ProductsApiController : Controller
{
   [HttpGet]
   public IActionResult ListProducts() { ... }

   [HttpGet("{id}")]
   public ActionResult GetProduct(int id) { ... }
}
```

在此示例中，URL 路径 `/products` 可以匹配 `ProductsApi.ListProducts`，URL 路径 `/products/5` 可以匹配 `ProductsApi.GetProduct(int)`。 这两项操作仅匹配 HTTP `GET`，因为它们用 `HttpGetAttribute` 标记。

应用于操作的以 `/` 或 `~/` 开头的路由模板不与应用于控制器的路由模板合并。 此示例匹配一组与 *默认路由* 类似的 URL 路径。

```csharp
[Route("Home")]
public class HomeController : Controller
{
    [Route("")]      // Combines to define the route template "Home"
    [Route("Index")] // Combines to define the route template "Home/Index"
    [Route("/")]     // Doesn't combine, defines the route template ""
    public IActionResult Index()
    {
        ViewData["Message"] = "Home index";
        var url = Url.Action("Index", "Home");
        ViewData["Message"] = "Home index" + "var url = Url.Action; =  " + url;
        return View();
    }

    [Route("About")] // Combines to define the route template "Home/About"
    public IActionResult About()
    {
        return View();
    }   
}
```

<a name="routing-ordering-ref-label"></a>

### <a name="ordering-attribute-routes"></a>对属性路由排序

与按定义的顺序执行的传统路由不同，属性路由会生成一个树并同时匹配所有路由。 其行为就像路由条目是以理想排序方式放置的一样；最特定的路由有机会比较一般的路由先执行。

例如，像 `blog/search/{topic}` 这样的路由比像 `blog/{*article}` 这样的路由更特定。 从逻辑上讲，`blog/search/{topic}` 路由默认情况下先“运行”，因为这是唯一合理的排序。 使用传统路由时，开发人员负责按所需顺序放置路由。

属性路由可以使用框架提供的所有路由属性的 `Order` 属性来配置顺序。 路由按 `Order` 属性的升序进行处理。 默认顺序为 `0`。 使用 `Order = -1` 设置的路由比未设置顺序的路由先运行。 使用 `Order = 1` 设置的路由在默认路由排序后运行。

> [!TIP]
> 避免依赖 `Order`。 如果 URL 空间需要有显式顺序值才能正确进行路由，则同样可能使客户端混淆不清。 属性路由通常选择与 URL 匹配的正确路由。 如果用于 URL 生成的默认顺序不起作用，使用路由名称作为替代项通常比应用 `Order` 属性更简单。

Razor Pages 路由和 MVC 控制器路由共享一个实现。 页面上的路由顺序信息 Razor 主题中提供了[ Razor 页面路由和应用约定：路由顺序](xref:razor-pages/razor-pages-conventions#route-order)。

<a name="routing-token-replacement-templates-ref-label"></a>

## <a name="token-replacement-in-route-templates-controller-action-area"></a>路由模板中的标记替换（[controller]、[action]、[area]）

为方便起见，特性路由支持 *标记替换* ，方法是在方括号中包含一个标记 (`[` ， `]`) 。 标记 `[action]`、`[area]` 和 `[controller]` 替换为定义了路由的操作中的操作名称值、区域名称值和控制器名称值。 在接下来的示例中，操作与注释中所述的 URL 路径匹配：

[!code-csharp[](routing/samples/2.x/main/Controllers/ProductsController.cs?range=7-11,13-17,20-22)]

标记替换发生在属性路由生成的最后一步。 上述示例的行为方式将与以下代码相同：

[!code-csharp[](routing/samples/2.x/main/Controllers/ProductsController2.cs?range=7-11,13-17,20-22)]

属性路由还可以与继承结合使用。 与标记替换结合使用时尤为强大。

```csharp
[Route("api/[controller]")]
public abstract class MyBaseController : Controller { ... }

public class ProductsController : MyBaseController
{
   [HttpGet] // Matches '/api/Products'
   public IActionResult List() { ... }

   [HttpPut("{id}")] // Matches '/api/Products/{id}'
   public IActionResult Edit(int id) { ... }
}
```

标记替换也适用于属性路由定义的路由名称。 `[Route("[controller]/[action]", Name="[controller]_[action]")]` 为每项操作生成一个唯一的路由名称。

若要匹配文本标记替换分隔符 `[` 或 `]`，可通过重复该字符（`[[` 或 `]]`）对其进行转义。

::: moniker-end

::: moniker range="= aspnetcore-2.2"

<a name="routing-token-replacement-transformers-ref-label"></a>

### <a name="use-a-parameter-transformer-to-customize-token-replacement"></a>使用参数转换程序自定义标记替换

使用参数转换程序可以自定义标记替换。 参数转换程序实现 `IOutboundParameterTransformer` 并转换参数值。 例如，一个自定义 `SlugifyParameterTransformer` 参数转换程序可将 `SubscriptionManagement` 路由值更改为 `subscription-management`。

`RouteTokenTransformerConvention` 是应用程序模型约定，可以：

* 将参数转换程序应用到应用程序中的所有属性路由。
* 在替换属性路由标记值时对其进行自定义。

```csharp
public class SubscriptionManagementController : Controller
{
    [HttpGet("[controller]/[action]")] // Matches '/subscription-management/list-all'
    public IActionResult ListAll() { ... }
}
```

`RouteTokenTransformerConvention` 在 `ConfigureServices` 中注册为选项。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc(options =>
    {
        options.Conventions.Add(new RouteTokenTransformerConvention(
                                     new SlugifyParameterTransformer()));
    });
}

public class SlugifyParameterTransformer : IOutboundParameterTransformer
{
    public string TransformOutbound(object value)
    {
        if (value == null) { return null; }

        // Slugify value
        return Regex.Replace(value.ToString(), "([a-z])([A-Z])", "$1-$2").ToLower();
    }
}
```

::: moniker-end


::: moniker range="< aspnetcore-3.0"
<a name="routing-multiple-routes-ref-label"></a>

### <a name="multiple-routes"></a>多个路由

属性路由支持定义多个访问同一操作的路由。 此操作最常用于模拟 *默认传统路由* 的行为，如以下示例所示：

```csharp
[Route("[controller]")]
public class ProductsController : Controller
{
   [Route("")]     // Matches 'Products'
   [Route("Index")] // Matches 'Products/Index'
   public IActionResult Index()
}
```

在控制器上放置多个路由属性意味着，每个路由属性将与操作方法上的每个路由属性合并。

```csharp
[Route("Store")]
[Route("[controller]")]
public class ProductsController : Controller
{
   [HttpPost("Buy")]     // Matches 'Products/Buy' and 'Store/Buy'
   [HttpPost("Checkout")] // Matches 'Products/Checkout' and 'Store/Checkout'
   public IActionResult Buy()
}
```

当在某个操作上放置多个路由属性（可实现 `IActionConstraint`）时，每个操作约束将与定义它的属性中的路由模板合并。

```csharp
[Route("api/[controller]")]
public class ProductsController : Controller
{
   [HttpPut("Buy")]      // Matches PUT 'api/Products/Buy'
   [HttpPost("Checkout")] // Matches POST 'api/Products/Checkout'
   public IActionResult Buy()
}
```

> [!TIP]
> 在操作上使用多个路由可能看起来很强大，但更建议使应用程序的 URL 空间保持简洁且定义完善。 仅在需要时，例如为了支持现有客户端，才在操作上使用多个路由。

<a name="routing-attr-options"></a>

### <a name="specifying-attribute-route-optional-parameters-default-values-and-constraints"></a>指定属性路由的可选参数、默认值和约束

属性路由支持使用与传统路由相同的内联语法，来指定可选参数、默认值和约束。

```csharp
[HttpPost("product/{id:int}")]
public IActionResult ShowProduct(int id)
{
   // ...
}
```

有关路由模板语法的详细说明，请参阅[路由模板参考](xref:fundamentals/routing#route-template-reference)。

<a name="routing-cust-rt-attr-irt-ref-label"></a>

### <a name="custom-route-attributes-using-iroutetemplateprovider"></a>使用 `IRouteTemplateProvider` 的自定义路由属性

该框架中提供的所有路由属性（`[Route(...)]`、`[HttpGet(...)]` 等）都可实现 `IRouteTemplateProvider` 接口。 当应用启动时，MVC 会查找控制器类和操作方法上的属性，并使用可实现 `IRouteTemplateProvider` 的属性生成一组初始路由。

用户可以实现 `IRouteTemplateProvider` 来定义自己的路由属性。 每个 `IRouteTemplateProvider` 都允许定义一个包含自定义路由模板、顺序和名称的路由：

```csharp
public class MyApiControllerAttribute : Attribute, IRouteTemplateProvider
{
   public string Template => "api/[controller]";

   public int? Order { get; set; }

   public string Name { get; set; }
}
```

应用 `[MyApiController]` 时，上述示例中的属性会自动将 `Template` 设置为 `"api/[controller]"`。

<a name="routing-app-model-ref-label"></a>

### <a name="using-application-model-to-customize-attribute-routes"></a>使用应用程序模型自定义属性路由

*应用程序模型* 是一个在启动时创建的对象模型，MVC 可使用其中的所有元数据来路由和执行操作。 *应用程序模型* 包含从路由属性收集（通过 `IRouteTemplateProvider`）的所有数据。 可通过编写 *约定* 在启动时修改应用程序模型，以便自定义路由的行为方式。 此部分通过一个简单的示例说明了如何使用应用程序模型自定义路由。

[!code-csharp[](routing/samples/2.x/main/NamespaceRoutingConvention.cs)]

<a name="routing-mixed-ref-label"></a>

## <a name="mixed-routing-attribute-routing-vs-conventional-routing"></a>混合路由：属性路由与传统路由

MVC 应用程序可以混合使用传统路由与属性路由。 通常将传统路由用于为浏览器处理 HTML 页面的控制器，将属性路由用于处理 REST API 的控制器。

操作既支持传统路由，也支持属性路由。 通过在控制器或操作上放置路由可实现属性路由。 不能通过传统路由访问定义属性路由的操作，反之亦然。 控制器上的 **任何** 路由属性都会使控制器中的所有操作使用属性路由。

> [!NOTE]
> 这两种路由系统的区别在于 URL 与路由模板匹配后所应用的过程。 在传统路由中，将使用匹配项中的路由值，从包含所有传统路由操作的查找表中选择操作和控制器。 在属性路由中，每个模板都与某项操作关联，无需进行进一步的查找。

## <a name="complex-segments"></a>复杂段

复杂段（例如，`[Route("/dog{token}cat")]`）通过非贪婪的方式从右到左匹配文字进行处理。 有关说明，请参阅[源代码](https://github.com/aspnet/Routing/blob/9cea167cfac36cf034dbb780e3f783114ef94780/src/Microsoft.AspNetCore.Routing/Patterns/RoutePatternMatcher.cs#L296)。 有关详细信息，请参阅 [此问题](https://github.com/dotnet/AspNetCore.Docs/issues/8197)。

<a name="routing-url-gen-ref-label"></a>

## <a name="url-generation"></a>URL 生成

MVC 应用程序可以使用路由的 URL 生成功能，生成指向操作的 URL 链接。 生成 URL 可消除硬编码 URL，使代码更稳定、更易维护。 此部分重点介绍 MVC 提供的 URL 生成功能，并且仅涵盖 URL 生成工作原理的基础知识。 有关 URL 生成的详细说明，请参阅[路由](xref:fundamentals/routing)。

`IUrlHelper` 接口用于生成 URL，是 MVC 与路由之间的基础结构的基础部分。 在控制器、视图和视图组件中，可通过 `Url` 属性找到 `IUrlHelper` 的实例。

在此示例中，将通过 `Controller.Url` 属性使用 `IUrlHelper` 接口来生成指向另一项操作的 URL。

[!code-csharp[](routing/samples/2.x/main/Controllers/UrlGenerationController.cs?name=snippet_1)]

如果应用程序使用的是传统默认路由，则 `url` 变量的值将为 URL 路径字符串 `/UrlGeneration/Destination`。 此 URL 路径由路由创建，方法是将当前请求中的路由值（环境值）与传递到 `Url.Action` 的值合并，并将这些值替换到路由模板中：

```
ambient values: { controller = "UrlGeneration", action = "Source" }
values passed to Url.Action: { controller = "UrlGeneration", action = "Destination" }
route template: {controller}/{action}/{id?}

result: /UrlGeneration/Destination
```

路由模板中的每个路由参数都会通过将名称与这些值和环境值匹配，来替换掉原来的值。 没有值的路由参数如果有默认值，则可使用默认值；如果本身是可选参数（比如此示例中的 `id`），则可直接跳过。 如果任何所需路由参数没有对应的值，URL 生成将失败。 如果某个路由的 URL 生成失败，则尝试下一个路由，直到尝试所有路由或找到匹配项为止。

上面的 `Url.Action` 示例假定使用传统路由，但 URL 生成功能的工作方式与属性路由相似，只不过概念不同。 在传统路由中，路由值用于扩展模板，`controller` 和 `action` 的路由值通常出现在该模板中 — 这种做法可行是因为通过路由匹配的 URL 遵守某项 *约定* 。 在属性路由中，`controller` 和 `action` 的路由值不能出现在模板中，它们用于查找要使用的模板。

此示例使用属性路由：

[!code-csharp[](routing/samples/2.x/main/StartupUseMvc.cs?name=snippet_1)]

[!code-csharp[](routing/samples/2.x/main/Controllers/UrlGenerationControllerAttr.cs?name=snippet_1)]

MVC 生成一个包含所有属性路由操作的查找表，并匹配 `controller` 和 `action` 的值，以选择要用于生成 URL 的路由模板。 在上述示例中，生成了 `custom/url/to/destination`。

### <a name="generating-urls-by-action-name"></a>根据操作名称生成 URL

`Url.Action` (`IUrlHelper` . `Action`) 以及所有相关重载都基于这样一种想法：用户想通过指定控制器名称和操作名称来指定要链接的内容。

> [!NOTE]
> 使用 `Url.Action` 时，将为用户指定 `controller` 和 `action` 的当前路由值，`controller` 和 `action` 的值是环境值  的一部分。 `Url.Action` 方法始终使用 `action` 和 `controller` 的当前值，并将生成将路由到当前操作的 URL 路径。

路由尝试使用环境值中的值来填充生成 URL 时未提供的信息。 通过使用路由（比如 `{a}/{b}/{c}/{d}`）和环境值 `{ a = Alice, b = Bob, c = Carol, d = David }`，路由就具有足够的信息来生成 URL，而无需任何附加值，因为所有路由参数都有值。 如果添加了值 `{ d = Donovan }`，则会忽略值 `{ d = David }`，生成的 URL 路径将为 `Alice/Bob/Carol/Donovan`。

> [!WARNING]
> URL 路径是分层的。 在上述示例中，如果添加了值 `{ c = Cheryl }`，则会忽略 `{ c = Carol, d = David }` 这两个值。 在这种情况下，`d` 不再具有任何值，URL 生成将失败。 用户需要指定 `c` 和 `d` 所需的值。  使用默认路由 (`{controller}/{action}/{id?}`) 时可能会遇到此问题，但在实际操作中很少遇到此行为，因为 `Url.Action` 始终显式指定 `controller` 和 `action` 值。

较长的 `Url.Action` 重载还采用附加 *路由值* 对象，为 `controller` 和 `action` 以外的路由参数提供值。 此重载最常与 `id` 结合使用，比如 `Url.Action("Buy", "Products", new { id = 17 })`。 按照惯例， *路由值* 对象通常是匿名类型的对象，但它也可以是 `IDictionary<>` 或 *普通旧 .NET 对象* 。 任何与路由参数不匹配的附加路由值都放在查询字符串中。

[!code-csharp[](routing/samples/2.x/main/Controllers/TestController.cs)]

> [!TIP]
> 若要创建绝对 URL，请使用采用 `protocol` 的重载：`Url.Action("Buy", "Products", new { id = 17 }, protocol: Request.Scheme)`

<a name="routing-gen-urls-route-ref-label"></a>

### <a name="generating-urls-by-route"></a>根据路由生成 URL

上面的代码演示了如何通过传入控制器和操作名称来生成 URL。 `IUrlHelper` 还提供 `Url.RouteUrl` 系列的方法。 这些方法类似于 `Url.Action`，但它们不会将 `action` 和 `controller` 的当前值复制到路由值。 最常见的用法是指定一个路由名称，以使用特定路由来生成 URL，通常 *不* 指定控制器或操作名称。

[!code-csharp[](routing/samples/2.x/main/Controllers/UrlGenerationControllerRouting.cs?name=snippet_1)]

<a name="routing-gen-urls-html-ref-label"></a>

### <a name="generating-urls-in-html"></a>在 HTML 中生成 URL

`IHtmlHelper` 提供 `HtmlHelper` 方法 `Html.BeginForm` 和 `Html.ActionLink`，可分别生成 `<form>` 和 `<a>` 元素。 这些方法使用 `Url.Action` 方法来生成 URL，并且采用相似的参数。 `HtmlHelper` 的配套 `Url.RouteUrl` 为 `Html.BeginRouteForm` 和 `Html.RouteLink`，两者具有相似的功能。

TagHelper 通过 `form` TagHelper 和 `<a>` TagHelper 生成 URL。 两者均通过 `IUrlHelper` 来实现。 有关详细信息，请参阅[使用表单](../views/working-with-forms.md)。

在视图内，可通过 `Url` 属性将 `IUrlHelper` 用于前文未涵盖的任何临时 URL 生成。

<a name="routing-gen-urls-action-ref-label"></a>

### <a name="generating-urls-in-action-results"></a>在操作结果中生成 URL

以上示例展示了如何在控制器中使用 `IUrlHelper`，不过，控制器中最常见的用法是将 URL 生成为操作结果的一部分。

`ControllerBase` 和 `Controller` 基类为操作结果提供简便的方法来引用另一项操作。 一种典型用法是在接受用户输入后进行重定向。

```csharp
public IActionResult Edit(int id, Customer customer)
{
    if (ModelState.IsValid)
    {
        // Update DB with new details.
        return RedirectToAction("Index");
    }
    return View(customer);
}
```

操作结果工厂方法遵循与 `IUrlHelper` 上的方法类似的模式。

<a name="routing-dedicated-ref-label"></a>

### <a name="special-case-for-dedicated-conventional-routes"></a>专用传统路由的特殊情况

传统路由可以使用一种特殊的路由定义，称为 *专用传统路由* 。 在下面的示例中，名为 `blog` 的路由是一种专用传统路由。

```csharp
app.UseMvc(routes =>
{
    routes.MapRoute("blog", "blog/{*article}",
        defaults: new { controller = "Blog", action = "Article" });
    routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
});
```

使用这些路由定义，`Url.Action("Index", "Home")` 将通过 `default` 路由生成 URL 路径 `/`，但是，为什么会这样？ 用户可能认为使用 `blog`，路由值 `{ controller = Home, action = Index }` 就足以生成 URL，且结果为 `/blog?action=Index&controller=Home`。

专用传统路由依赖于不具有相应路由参数的默认值的特殊行为，以防止路由在 URL 生成过程中“太贪婪”。 在此例中，默认值是为 `{ controller = Blog, action = Article }`，`controller` 和 `action` 均未显示为路由参数。 当路由执行 URL 生成时，提供的值必须与默认值匹配。 使用 `blog` 的 URL 生成将失败，因为值 `{ controller = Home, action = Index }` 与 `{ controller = Blog, action = Article }` 不匹配。 然后，路由回退，尝试使用 `default`，并最终成功。

<a name="routing-areas-ref-label"></a>

## <a name="areas"></a>Areas

[区域](areas.md)是一种 MVC 功能，用于将相关功能整理到一个组中，作为单独的路由命名空间（用于控制器操作）和文件夹结构（用于视图）。 通过使用区域，应用程序可以有多个名称相同的控制器，只要它们具有不同的 *区域* 。 通过向 `controller` 和 `action` 添加另一个路由参数 `area`，可使用区域为路由创建层次结构。 此部分将讨论路由如何与区域交互；有关如何将区域与视图结合使用的详细信息，请参阅[区域](areas.md)。

下面的示例将 MVC 配置为使用默认传统路由和 *区域路由* （用于名为 `Blog` 的区域）：

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet1)]

与 URL 路径（比如 `/Manage/Users/AddUser`）匹配时，第一个路由将生成路由值 `{ area = Blog, controller = Users, action = AddUser }`。 `area` 路由值由 `area` 的默认值生成，事实上，通过 `MapAreaRoute` 创建的路由等效于以下路由：

[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet2)]

`MapAreaRoute` 通过为使用所提供的区域名称（本例中为 `Blog`）的 `area` 提供默认值和约束，来创建路由。 默认值确保路由始终生成 `{ area = Blog, ... }`，约束要求在生成 URL 时使用值 `{ area = Blog, ... }`。

> [!TIP]
> 传统路由依赖于顺序。 一般情况下，具有区域的路由应放在路由表中靠前的位置，因为它们比没有区域的路由更特定。

在上面的示例中，路由值将与以下操作匹配：

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

`AreaAttribute` 用于将控制器表示为某个区域的一部分，比方说，此控制器位于 `Blog` 区域中。 没有 `[Area]` 属性的控制器不是任何区域的成员，在路由提供 `area` 路由值时 **不** 匹配。 在下面的示例中，只有所列出的第一个控制器才能与路由值 `{ area = Blog, controller = Users, action = AddUser }` 匹配。

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Blog/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Zebra/Controllers/UsersController.cs)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Controllers/UsersController.cs)]

> [!NOTE]
> 出于完整性考虑，此处显示了每个控制器的命名空间，否则，控制器会发生命名冲突并生成编译器错误。 类命名空间对 MVC 的路由没有影响。

前两个控制器是区域成员，仅在 `area` 路由值提供其各自的区域名称时匹配。 第三个控制器不是任何区域的成员，只能在路由没有为 `area` 提供任何值时匹配。

> [!NOTE]
> 就 *不匹配任何值* 而言，缺少 `area` 值相当于 `area` 的值为 NULL 或空字符串。

在某个区域内执行某项操作时，`area` 的路由值将以 *环境值* 的形式提供，以便路由用于生成 URL。 这意味着默认情况下，区域在 URL 生成中具有 *粘性* ，如以下示例所示。
[!code-csharp[](routing/samples/3.x/AreasRouting/Startup.cs?name=snippet3)]

[!code-csharp[](routing/samples/3.x/AreasRouting/Areas/Duck/Controllers/UsersController.cs)]

<a name="iactionconstraint-ref-label"></a>

## <a name="understanding-iactionconstraint"></a>了解 IActionConstraint

> [!NOTE]
> 此部分深入介绍框架内部结构以及 MVC 如何选择要执行的操作。 典型的应用程序不需要自定义 `IActionConstraint`

即使不熟悉 `IActionConstraint`，也可能已经用过该接口。 `[HttpGet]` 属性和类似的 `[Http-VERB]` 属性可实现 `IActionConstraint` 来限制操作方法的执行。

```csharp
public class ProductsController : Controller
{
    [HttpGet]
    public IActionResult Edit() { }

    public IActionResult Edit(...) { }
}
```

假定使用默认传统路由，URL 路径 `/Products/Edit` 将生成值 `{ controller = Products, action = Edit }`，这将匹配此处所示的 **两项** 操作。 在 `IActionConstraint` 术语中，我们会说，这两项操作都被视为候选项，因为它们都与该路由数据匹配。

当 `HttpGetAttribute` 执行时，它认为 *Edit()* 是 *GET* 的匹配项，而不是任何其他 Http 谓词的匹配项。 `Edit(...)` 操作未定义任何约束，因此将匹配任何 Http 谓词。 因此，假定 Http 谓词为 `POST`，则仅 `Edit(...)` 匹配。 不过，对于 `GET`，这两项操作仍然都能匹配，只是具有 `IActionConstraint` 的操作始终被认为比没有该接口的操作 *更匹配* 。 因此，由于 `Edit()` 具有 `[HttpGet]`，则认为它更特定，在两项操作都能匹配的情况将选择它。

从概念上讲，`IActionConstraint` 是一种 *重载* 形式，但它并不重载具有相同名称的方法，而在匹配相同 URL 的操作之间重载。 属性路由也使用 `IActionConstraint`，这可能会导致将不同控制器中的操作都视为候选项。

<a name="iactionconstraint-impl-ref-label"></a>

### <a name="implementing-iactionconstraint"></a>实现 IActionConstraint

实现 `IActionConstraint` 最简单的方法是创建派生自 `System.Attribute` 的类，并将其置于操作和控制器上。 MVC 将自动发现任何应用为属性的 `IActionConstraint`。 可使用应用程序模型应用约束，这可能是最灵活的一种方法，因为它允许对其应用方式进行元编程。

在下面的示例中，约束基于路由数据中的 *国家/地区代码* 选择操作。 [GitHub 上的完整示例](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.ActionConstraintSample.Web/CountrySpecificAttribute.cs)。

```csharp
public class CountrySpecificAttribute : Attribute, IActionConstraint
{
    private readonly string _countryCode;

    public CountrySpecificAttribute(string countryCode)
    {
        _countryCode = countryCode;
    }

    public int Order
    {
        get
        {
            return 0;
        }
    }

    public bool Accept(ActionConstraintContext context)
    {
        return string.Equals(
            context.RouteContext.RouteData.Values["country"].ToString(),
            _countryCode,
            StringComparison.OrdinalIgnoreCase);
    }
}
```

用户负责实现 `Accept` 方法，并为要执行的约束选择“顺序”。 在此例中，当 `country` 路由值匹配时，`Accept` 方法返回 `true` 以表示该操作是匹配项。 它与 `RouteValueAttribute` 的不同之处在于，它允许回退到非属性化操作。 通过该示例可以了解到，如果定义 `en-US` 操作，则像 `fr-FR` 这样的国家/地区代码将回退到一个未应用 `[CountrySpecific(...)]` 的较通用的控制器。

`Order` 属性决定约束所属的 *阶段* 。 操作约束基于 `Order` 分组运行。 例如，该框架提供的所有 HTTP 方法属性均使用相同的 `Order` 值，以便在相同的阶段运行。 用户可以按需设置阶段数来实现所需的策略。

> [!TIP]
> 若要确定 `Order` 的值，请考虑是否应在 HTTP 方法前应用约束。 数值较低的先运行。

::: moniker-end
