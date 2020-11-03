---
title: 使用 ASP.NET Core 创建 Web API
author: scottaddie
description: 了解在 ASP.NET Core 中创建 Web API 的基础知识。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: mvc
ms.date: 07/20/2020
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
uid: web-api/index
ms.openlocfilehash: 51f76bc33d2da6097d26c1ffe17b1c93046a7b88
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052456"
---
# <a name="create-web-apis-with-aspnet-core"></a>使用 ASP.NET Core 创建 Web API

作者：[Scott Addie](https://github.com/scottaddie) 和 [Tom Dykstra](https://github.com/tdykstra)

ASP.NET Core 支持使用 C# 创建 RESTful 服务，也称为 Web API。 若要处理请求，Web API 使用控制器。 Web API 中的 *控制器* 是派生自 `ControllerBase` 的类。 本文介绍了如何使用控制器处理 Web API 请求。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/index/samples)。 （[下载方法](xref:index#how-to-download-a-sample)）。

## <a name="controllerbase-class"></a>ControllerBase 类

Web API 包含一个或多个派生自 <xref:Microsoft.AspNetCore.Mvc.ControllerBase> 的控制器类。 Web API 项目模板提供了一个入门版控制器：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=3)]

::: moniker-end

不要通过从 <xref:Microsoft.AspNetCore.Mvc.Controller> 类派生来创建 Web API 控制器。 `Controller` 派生自 `ControllerBase`，并添加对视图的支持，因此它用于处理 Web 页面，而不是 Web API 请求。 此规则有一个例外：如果打算为视图和 Web API 使用相同的控制器，则从 `Controller` 派生控制器。

`ControllerBase` 类提供了很多用于处理 HTTP 请求的属性和方法。 例如，`ControllerBase.CreatedAtAction` 返回 201 状态代码：

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=10)]

下面是 `ControllerBase` 提供的方法的更多示例。

|方法   |说明    |
|---------|---------|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest%2A>| 返回 400 状态代码。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound%2A>|返回 404 状态代码。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.PhysicalFile%2A>|返回文件。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync%2A>|调用[模型绑定](xref:mvc/models/model-binding)。|
|<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryValidateModel%2A>|调用[模型验证](xref:mvc/models/validation)。|

有关可用方法和属性的列表，请参阅 <xref:Microsoft.AspNetCore.Mvc.ControllerBase>。

## <a name="attributes"></a>特性

<xref:Microsoft.AspNetCore.Mvc> 命名空间提供可用于配置 Web API 控制器的行为和操作方法的属性。 下述示例使用属性来指定受支持的 HTTP 操作谓词和所有可返回的已知 HTTP 状态代码：

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_400And201&highlight=1-3)]

以下是可用属性的更多示例。

|特性|说明|
|---------|-----|
|[`[Route]`](<xref:Microsoft.AspNetCore.Mvc.RouteAttribute>)      |指定控制器或操作的 URL 模式。|
|[`[Bind]`](<xref:Microsoft.AspNetCore.Mvc.BindAttribute>)        |指定要包含的前缀和属性，以进行模型绑定。|
|[`[HttpGet]`](<xref:Microsoft.AspNetCore.Mvc.HttpGetAttribute>)  |标识支持 HTTP GET 操作谓词的操作。|
|[`[Consumes]`](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>)|指定某个操作接受的数据类型。|
|[`[Produces]`](<xref:Microsoft.AspNetCore.Mvc.ProducesAttribute>)|指定某个操作返回的数据类型。|

有关包含可用属性的列表，请参阅 <xref:Microsoft.AspNetCore.Mvc> 命名空间。

## <a name="apicontroller-attribute"></a>ApiController 属性

[`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 属性可应用于控制器类，以启用下述 API 特定的固定行为：

::: moniker range=">= aspnetcore-2.2"

* [属性路由要求](#attribute-routing-requirement)
* [自动 HTTP 400 响应](#automatic-http-400-responses)
* [绑定源参数推理](#binding-source-parameter-inference)
* [Multipart/form-data 请求推理](#multipartform-data-request-inference)
* [错误状态代码的问题详细信息](#problem-details-for-error-status-codes)

必须有[兼容性版本](xref:mvc/compatibility-version) 2.2 或更高版本，才能使用“错误状态代码的问题详细信息”功能。 必须有兼容性版本 2.1 或更高版本，才能使用其他功能。

::: moniker-end

* [属性路由要求](#attribute-routing-requirement)
* [自动 HTTP 400 响应](#automatic-http-400-responses)
* [绑定源参数推理](#binding-source-parameter-inference)
* [Multipart/form-data 请求推理](#multipartform-data-request-inference)

这些功能需要[兼容性版本](xref:mvc/compatibility-version)为 2.1 或更高版本。

### <a name="attribute-on-specific-controllers"></a>特定控制器上的属性

`[ApiController]` 属性可应用于特定控制器，如项目模板中的以下示例所示：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2])]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=2)]

::: moniker-end

### <a name="attribute-on-multiple-controllers"></a>多个控制器上的属性

在多个控制器上使用该属性的一种方法是创建通过 `[ApiController]` 属性批注的自定义基控制器类。 下述示例展示了自定义基类以及从其派生的控制器：

[!code-csharp[](index/samples/2.x/2.2/Controllers/MyControllerBase.cs?name=snippet_MyControllerBase)]

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_Inherit)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="attribute-on-an-assembly"></a>程序集上的属性

如果将[兼容性版本](xref:mvc/compatibility-version)设置为 2.2 或更高版本，则 `[ApiController]` 属性可应用于程序集。 以这种方式进行注释，会将 web API 行为应用到程序集中的所有控制器。 无法针对单个控制器执行选择退出操作。 将程序集级别的属性应用于 `Startup` 类两侧的命名空间声明：

```csharp
[assembly: ApiController]
namespace WebApiSample
{
    public class Startup
    {
        ...
    }
}
```

::: moniker-end

## <a name="attribute-routing-requirement"></a>特性路由要求

`[ApiController]` 属性使属性路由成为要求。 例如：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_ControllerSignature&highlight=2)]

不能通过由 `Startup.Configure` 中的 `UseEndpoints`、<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> 或 <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> 定义的[传统路由](xref:mvc/controllers/routing#conventional-routing)访问操作。

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Controllers/ValuesController.cs?name=snippet_ControllerSignature&highlight=1)]

不能通过 <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A> 定义的[传统路由](xref:mvc/controllers/routing#conventional-routing)或通过 `Startup.Configure` 中的 <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> 访问操作。

::: moniker-end

## <a name="automatic-http-400-responses"></a>自动 HTTP 400 响应

`[ApiController]` 属性使模型验证错误自动触发 HTTP 400 响应。 因此，操作方法中不需要以下代码：

```csharp
if (!ModelState.IsValid)
{
    return BadRequest(ModelState);
}
```

ASP.NET Core MVC 使用 <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ModelStateInvalidFilter> 操作筛选器来执行上述检查。

### <a name="default-badrequest-response"></a>默认 BadRequest 响应

使用 2.1 的兼容性版本时，HTTP 400 响应的默认响应类型为 <xref:Microsoft.AspNetCore.Mvc.SerializableError>。 下述请求正文是序列化类型的示例：

```json
{
  "": [
    "A non-empty request body is required."
  ]
}
```

::: moniker range=">= aspnetcore-2.2"

使用 2.2 或更高版本的兼容性版本时，HTTP 400 响应的默认响应类型为 <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails>。 下述请求正文是序列化类型的示例：

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "traceId": "|7fb5e16a-4c8f23bbfc974667.",
  "errors": {
    "": [
      "A non-empty request body is required."
    ]
  }
}
```

`ValidationProblemDetails` 类型：

* 提供计算机可读的格式来指定 Web API 响应中的错误。
* 符合 [RFC 7807 规范](https://tools.ietf.org/html/rfc7807)。

::: moniker-end

要使自动和自定义响应保持一致，请调用 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A> 方法，而不是 <xref:System.Web.Http.ApiController.BadRequest%2A>。 `ValidationProblem` 返回 <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> 对象以及自动响应。

### <a name="log-automatic-400-responses"></a>记录自动 400 响应

请参阅[如何对模型验证错误记录自动 400 响应 (dotnet/AspNetCore.Docs#12157)](https://github.com/dotnet/AspNetCore.Docs/issues/12157)。

### <a name="disable-automatic-400-response"></a>禁用自动 400 响应

若要禁用自动 400 行为，请将 <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressModelStateInvalidFilter> 属性设置为 `true`。 将以下突出显示的代码添加到 `Startup.ConfigureServices`：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,5)]

::: moniker-end

## <a name="binding-source-parameter-inference"></a>绑定源参数推理

绑定源特性定义可找到操作参数值的位置。 存在以下绑定源特性：

|特性|绑定源 |
|---------|---------|
|[`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute)     | 请求正文 |
|[`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute)     | 请求正文中的表单数据 |
|[`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) | 请求标头 |
|[`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute)   | 请求查询字符串参数 |
|[`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute)   | 当前请求中的路由数据 |
|[`[FromServices]`](xref:mvc/controllers/dependency-injection#action-injection-with-fromservices) | 作为操作参数插入的请求服务 |

> [!WARNING]
> 当值可能包含 `%2f`（即 `/`）时，请勿使用 `[FromRoute]`。 `%2f` 不会转换为 `/`（非转义形式）。 如果值可能包含 `%2f`，则使用 `[FromQuery]`。

如果没有 `[ApiController]` 属性或诸如 `[FromQuery]` 的绑定源属性，ASP.NET Core 运行时会尝试使用复杂对象模型绑定器。 复杂对象模型绑定器按已定义顺序从值提供程序拉取数据。

在下面的示例中，`[FromQuery]` 特性指示 `discontinuedOnly` 参数值在请求 URL 的查询字符串中提供：

[!code-csharp[](index/samples/2.x/2.2/Controllers/ProductsController.cs?name=snippet_BindingSourceAttributes&highlight=3)]

`[ApiController]` 属性将推理规则应用于操作参数的默认数据源。 借助这些规则，无需通过将属性应用于操作参数来手动识别绑定源。 绑定源推理规则的行为如下：

* `[FromBody]` 针对复杂类型参数进行推断。 `[FromBody]` 不适用于具有特殊含义的任何复杂的内置类型，如 <xref:Microsoft.AspNetCore.Http.IFormCollection> 和 <xref:System.Threading.CancellationToken>。 绑定源推理代码将忽略这些特殊类型。
* `[FromForm]` 针对 <xref:Microsoft.AspNetCore.Http.IFormFile> 和 <xref:Microsoft.AspNetCore.Http.IFormFileCollection> 类型的操作参数进行推断。 该特性不针对任何简单类型或用户定义类型进行推断。
* `[FromRoute]` 针对与路由模板中的参数相匹配的任何操作参数名称进行推断。 当多个路由与一个操作参数匹配时，任何路由值都视为 `[FromRoute]`。
* `[FromQuery]` 针对任何其他操作参数进行推断。

### <a name="frombody-inference-notes"></a>FromBody 推理说明

对于简单类型（例如 `string` 或 `int`），推断不出 `[FromBody]`。 因此，如果需要该功能，对于简单类型，应使用 `[FromBody]` 属性。

当操作拥有多个从请求正文中绑定的参数时，将会引发异常。 例如，以下所有操作方法签名都会导致异常：

* `[FromBody]` 对两者进行推断，因为它们是复杂类型。

  ```csharp
  [HttpPost]
  public IActionResult Action1(Product product, Order order)
  ```

* `[FromBody]` 对一个进行归属，对另一个进行推断，因为它是复杂类型。

  ```csharp
  [HttpPost]
  public IActionResult Action2(Product product, [FromBody] Order order)
  ```

* `[FromBody]` 对两者进行归属。

  ```csharp
  [HttpPost]
  public IActionResult Action3([FromBody] Product product, [FromBody] Order order)
  ```

::: moniker range="= aspnetcore-2.1"

> [!NOTE]
> 在 ASP.NET Core 2.1 中，集合类型参数（如列表和数组）被不正确地推断为 `[FromQuery]`。 若要从请求正文中绑定参数，应对这些参数使用 `[FromBody]` 属性。 此行为在 ASP.NET Core 2.2 或更高版本中得到了更正，其中集合类型参数默认被推断为从正文中绑定。

::: moniker-end

### <a name="disable-inference-rules"></a>禁用推理规则

若要禁用绑定源推理，请将 <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressInferBindingSourcesForParameters> 设置为 `true`。 在 `Startup.ConfigureServices` 中添加下列代码：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,6)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,4)]

::: moniker-end

## <a name="multipartform-data-request-inference"></a>Multipart/form-data 请求推理

使用 [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) 属性批注操作参数时，`[ApiController]` 属性应用推理规则。 将推断 `multipart/form-data` 请求内容类型。

要禁用默认行为，请在 `Startup.ConfigureServices` 中将 <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressConsumesConstraintForFormFileParameters> 属性设置为 `true`：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,4)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,5)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](index/samples/2.x/2.1/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=1,3)]

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="problem-details-for-error-status-codes"></a>错误状态代码的问题详细信息

当兼容性版本为 2.2 或更高版本时，MVC 会将错误结果（状态代码为 400 或更高的结果）转换为状态代码为 <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> 的结果。 `ProblemDetails` 类型基于 [RFC 7807 规范](https://tools.ietf.org/html/rfc7807)，用于提供 HTTP 响应中计算机可读的错误详细信息。

考虑在控制器操作中使用以下代码：

[!code-csharp[](index/samples/2.x/2.2/Controllers/PetsController.cs?name=snippet_ProblemDetailsStatusCode)]

`NotFound` 方法会生成带 `ProblemDetails` 正文的 HTTP 404 状态代码。 例如：

```json
{
  type: "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  title: "Not Found",
  status: 404,
  traceId: "0HLHLV31KRN83:00000001"
}
```

### <a name="disable-problemdetails-response"></a>禁用 ProblemDetails 响应

当 <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.SuppressMapClientErrors%2A> 属性设置为 `true` 时，会禁止自动创建错误状态代码的 `ProblemDetails`。 在 `Startup.ConfigureServices` 中添加下列代码：

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=2,7)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=3,8)]

::: moniker-end

<a name="consumes"></a>

## <a name="define-supported-request-content-types-with-the-consumes-attribute"></a>使用 [Consumes] 属性定义支持的请求内容类型

默认情况下，操作支持所有可用的请求内容类型。 例如，如果应用配置为同时支持 JSON 和 XML [输入格式化程序](xref:mvc/models/model-binding#input-formatters)，那么操作支持多种内容类型，其中包括 `application/json` 和 `application/xml`。

使用 [[Consumes]](<xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute>) 属性，操作可以限制支持的请求内容类型。 将 `[Consumes]` 属性应用于操作或控制器，同时指定一个或多个内容类型：

```csharp
[HttpPost]
[Consumes("application/xml")]
public IActionResult CreateProduct(Product product)
```

在上面的代码中，`CreateProduct` 操作指定内容类型 `application/xml`。 路由到此操作的请求必须指定 `application/xml` 的 `Content-Type` 头。 如果请求未指定 `application/xml` 的 `Content-Type` 头，会生成 [415 不支持的媒体类型](https://developer.mozilla.org/docs/Web/HTTP/Status/415)响应。

使用 `[Consumes]` 属性，操作可以通过应用类型约束，根据传入请求的内容类型来影响它的选择。 请看下面的示例：

[!code-csharp[](index/samples/3.x/Controllers/ConsumesController.cs?name=snippet_Class)]

在上面的代码中，`ConsumesController` 配置为处理发送到 `https://localhost:5001/api/Consumes` URL 的请求。 控制器的两个操作（`PostJson` 和 `PostForm`）都使用相同的 URL 处理 POST 请求。 如果 `[Consumes]` 属性不应用类型约束，则会抛出不明确匹配异常。

`[Consumes]` 属性应用于两个操作。 `PostJson` 操作处理使用 `application/json` 的 `Content-Type` 头发送的请求。 `PostForm` 操作处理使用 `application/x-www-form-urlencoded` 的 `Content-Type` 头发送的请求。 

## <a name="additional-resources"></a>其他资源

* <xref:web-api/action-return-types>
* <xref:web-api/handle-errors>
* <xref:web-api/advanced/custom-formatters>
* <xref:web-api/advanced/formatting>
* <xref:tutorials/web-api-help-pages-using-swagger>
* <xref:mvc/controllers/routing>
