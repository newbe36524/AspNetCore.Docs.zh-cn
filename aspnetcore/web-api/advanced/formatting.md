---
title: 设置 ASP.NET Core Web API 中响应数据的格式
author: ardalis
description: 了解如何设置 ASP.NET Core Web API 中响应数据的格式。
ms.author: riande
ms.custom: H1Hack27Feb2017
ms.date: 04/17/2020
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
uid: web-api/advanced/formatting
ms.openlocfilehash: 89e3e51373db5f7cff974b7a8c69d06bedf856ca
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052508"
---
# <a name="format-response-data-in-aspnet-core-web-api"></a>设置 ASP.NET Core Web API 中响应数据的格式

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Steve Smith](https://ardalis.com/)

ASP.NET Core MVC 支持设置响应数据的格式。 可以使用特定格式或响应客户端请求的格式，来设置响应数据的格式。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/formatting)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="format-specific-action-results"></a>特定于格式的操作结果

一些操作结果类型特定于特殊格式，例如 <xref:Microsoft.AspNetCore.Mvc.JsonResult> 和 <xref:Microsoft.AspNetCore.Mvc.ContentResult>。 操作可以返回使用特定格式设置格式的结果，而不考虑客户端首选项。 例如，返回 `JsonResult`，将返回 JSON 格式的数据。 返回 `ContentResult` 或字符串，将返回纯文本格式的字符串数据。

无需操作返回任意特定类型。 ASP.NET Core 支持任何对象返回值。  返回非 <xref:Microsoft.AspNetCore.Mvc.IActionResult> 类型对象的操作结果将使用相应的 <xref:Microsoft.AspNetCore.Mvc.Formatters.IOutputFormatter> 实现来进行序列化。 有关详细信息，请参阅 <xref:web-api/action-return-types>。

内置的帮助程序方法 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> 返回 JSON 格式的数据：[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_get)]

示例下载返回作者列表。 在 F12 浏览器开发人员工具或 [Postman](https://www.getpostman.com/tools) 中使用上述代码：

* 将显示包含内容类型的响应标头。  `application/json; charset=utf-8`
* 将显示请求标头。 例如 `Accept` 标头。 上述代码将忽略 `Accept` 标头。

若要返回纯文本格式数据，请使用 <xref:Microsoft.AspNetCore.Mvc.ContentResult> 和 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Content%2A> 帮助程序：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_about)]

在上述代码中，返回的 `text/plain` 为 `Content-Type`。 返回字符串，将提供 `text/plain` 类型的 `Content-Type`：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_string)]

对于包含多个返回类型的操作，将返回 `IActionResult`。 例如，基于执行的操作的结果返回不同的 HTTP 状态代码。

## <a name="content-negotiation"></a>内容协商

当客户端指定 [Accept 标头](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html)时，会发生内容协商。 ASP.NET Core 使用的默认格式是 [JSON](https://json.org/)。 内容协商有以下特点：

* 由 <xref:Microsoft.AspNetCore.Mvc.ObjectResult> 实现。
* 内置于从帮助程序方法返回的特定于状态代码的操作结果中。 操作结果帮助程序方法基于 `ObjectResult`。

返回模型类型后，返回类型为 `ObjectResult`。

以下操作方法使用 `Ok` 和 `NotFound` 帮助程序方法：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_search)]

默认情况下，ASP.NET Core 支持 `application/json`、`text/json` 和 `text/plain` 媒体类型。 [Fiddler](https://www.telerik.com/fiddler) 或 [Postman](https://www.getpostman.com/tools) 等工具可以设置 `Accept` 请求标头，来指定返回格式。 `Accept` 标头包含服务器支持的类型时，将返回该类型。 下一节将介绍如何添加其他格式化程序。

控制器操作可以返回 POCO（普通旧 CLR 对象）。 返回 POCO 时，运行时自动创建包装该对象的 `ObjectResult`。 客户端将获得已格式化和序列化的对象。 若将返回的对象为 `null`，将返回 `204 No Content` 响应。

返回对象类型：

[!code-csharp[](./formatting/sample/Controllers/AuthorsController.cs?name=snippet_alias)]

在前面的代码中，请求有效作者别名将返回具有作者数据的 `200 OK` 响应。 请求无效别名将返回 `204 No Content` 响应。

### <a name="the-accept-header"></a>Accept 标头

内容协商在 `Accept` 标头出现在请求中时发生  。 请求包含 Accept 标头时，ASP.NET Core 将执行以下操作：

* 按首选顺序枚举 Accept 标头中的媒体类型。
* 尝试找到可以生成某种指定格式的响应的格式化程序。

若未找到可以满足客户端请求的格式化程序，ASP.NET Core 将指定以下操作：

* 已设置 `406 Not Acceptable` 时，将返回 <xref:Microsoft.AspNetCore.Mvc.MvcOptions>，或者
* 尝试找到第一个可以生成响应的格式化程序。

如果没有配置实现所请求格式的格式化程序，那么使用第一个可以设置对象格式的格式化程序。 若请求中没有 `Accept` 标头：

* 将使用第一个可以处理对象的格式化程序来将响应序列化。
* 不执行任何协商。 服务器将决定要返回的格式。

如果 Accept 标头包含 `*/*`，则将忽略该标头，除非 `RespectBrowserAcceptHeader` 在 <xref:Microsoft.AspNetCore.Mvc.MvcOptions> 上设置为 true。

### <a name="browsers-and-content-negotiation"></a>浏览器和内容协商

与典型的 API 客户端不同的是，Web 浏览器提供 `Accept` 标头。 Web 浏览器指定多种格式，包括通配符。 默认情况下，当框架检测到请求来自浏览器时，将执行以下操作：

* 忽略 `Accept` 标头。
* 若未另行配置，将使用 JSON 返回内容。

这样，在使用 API 时，浏览器中的体验将更加一致。

若要将应用配置为采用浏览器 Accept 标头，请将 <xref:Microsoft.AspNetCore.Mvc.MvcOptions.RespectBrowserAcceptHeader> 设置为 `true`：

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupRespectBrowserAcceptHeader.cs?name=snippet)]
::: moniker-end

### <a name="configure-formatters"></a>配置格式化程序

需要支持其他格式的应用可以添加相应的 NuGet 包，并配置支持。 输入和输出的格式化程序不同。 [模型绑定](xref:mvc/models/model-binding)使用输入格式化程序。 格式响应使用输出格式化程序。 有关创建自定义格式化程序的信息，请参阅[自定义格式化程序](xref:web-api/advanced/custom-formatters)。

::: moniker range=">= aspnetcore-3.0"

### <a name="add-xml-format-support"></a>添加 XML 格式支持

调用 <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> 来配置使用 <xref:System.Xml.Serialization.XmlSerializer> 实现的 XML 格式化程序：

[!code-csharp[](./formatting/3.0sample/Startup.cs?name=snippet)]

前面的代码将使用 `XmlSerializer` 将结果序列化。

使用前面的代码时，控制器方法会基于请求的 `Accept` 标头返回相应的格式。

### <a name="configure-systemtextjson-based-formatters"></a>配置基于 System.Text.Json 的格式化程序

可以使用 `Microsoft.AspNetCore.Mvc.JsonOptions.SerializerOptions` 配置基于 `System.Text.Json` 的格式化程序的功能。

```csharp
services.AddControllers().AddJsonOptions(options =>
{
    // Use the default property (Pascal) casing.
    options.JsonSerializerOptions.PropertyNamingPolicy = null;

    // Configure a custom converter.
    options.JsonSerializerOptions.Converters.Add(new MyCustomJsonConverter());
});
```

可以使用 `JsonResult` 配置基于每个操作的输出序列化选项。 例如： 。

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerOptions
    {
        WriteIndented = true,
    });
}
```

### <a name="add-newtonsoftjson-based-json-format-support"></a>添加基于 Newtonsoft.Json 的 JSON 格式支持

ASP.NET Core 3.0 之前的版本中，默认设置使用通过 `Newtonsoft.Json` 包实现的 JSON 格式化程序。 在 ASP.NET Core 3.0 或更高版本中，默认 JSON 格式化程序基于 `System.Text.Json`。 `Newtonsoft.Json`通过安装 [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) NuGet 包并在中进行配置，可获得对基于的格式化程序和功能的支持 `Startup.ConfigureServices` 。

[!code-csharp[](./formatting/3.0sample/StartupNewtonsoftJson.cs?name=snippet)]

在前面的代码中，对的调用将 `AddNewtonsoftJson` 配置以下 WEB API、MVC 和 Razor 页面功能以供使用 `Newtonsoft.Json` ：

* 用于读取和写入 JSON 的输入和输出格式化程序
* <xref:Microsoft.AspNetCore.Mvc.JsonResult>
* [JSON 修补程序](xref:web-api/jsonpatch)
* <xref:Microsoft.AspNetCore.Mvc.Rendering.IJsonHelper>
* [TempData](xref:fundamentals/app-state#tempdata)

某些功能可能不适用于基于 `System.Text.Json` 的格式化程序，而需要引用基于 `Newtonsoft.Json` 的格式化程序。 若应用符合以下情况，请继续使用基于 `Newtonsoft.Json` 的格式化程序：

* 使用 `Newtonsoft.Json` 属性。 例如，`[JsonProperty]` 或 `[JsonIgnore]`。
* 自定义序列化设置。
* 依赖 `Newtonsoft.Json` 提供的功能。
* 配置 `Microsoft.AspNetCore.Mvc.JsonResult.SerializerSettings`。 ASP.NET Core 3.0 之前的版本中，`JsonResult.SerializerSettings` 接受特定于 `Newtonsoft.Json` 的 `JsonSerializerSettings` 的实例。
* 生成 [OpenAPI](<xref:tutorials/web-api-help-pages-using-swagger>) 文档。

可以使用 `Microsoft.AspNetCore.Mvc.MvcNewtonsoftJsonOptions.SerializerSettings` 配置基于 `Newtonsoft.Json` 的格式化程序的功能：

```csharp
services.AddControllers().AddNewtonsoftJson(options =>
{
    // Use the default property (Pascal) casing
    options.SerializerSettings.ContractResolver = new DefaultContractResolver();

    // Configure a custom converter
    options.SerializerSettings.Converters.Add(new MyCustomJsonConverter());
});
```

可以使用 `JsonResult` 配置基于每个操作的输出序列化选项。 例如： 。

```csharp
public IActionResult Get()
{
    return Json(model, new JsonSerializerSettings
    {
        Formatting = Formatting.Indented,
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

### <a name="add-xml-format-support"></a>添加 XML 格式支持

XML 格式需要 [Microsoft.AspNetCore.Mvc.Formatters.Xml](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Formatters.Xml/) NuGet 包。

调用 <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcBuilderExtensions.AddXmlSerializerFormatters*> 来配置使用 <xref:System.Xml.Serialization.XmlSerializer> 实现的 XML 格式化程序：

[!code-csharp[](./formatting/sample/Startup.cs?name=snippet)]

前面的代码将使用 `XmlSerializer` 将结果序列化。

使用前面的代码时，控制器方法应基于请求的 `Accept` 标头返回相应的格式。

::: moniker-end

### <a name="specify-a-format"></a>指定格式

若要限制响应格式，请应用 [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) 筛选器。 与大多数 [筛选器](xref:mvc/controllers/filters)一样， `[Produces]` 可以在操作、控制器或全局范围内应用：

[!code-csharp[](./formatting/3.0sample/Controllers/WeatherForecastController.cs?name=snippet)]

上述 [`[Produces]`](xref:Microsoft.AspNetCore.Mvc.ProducesAttribute) 筛选器：

* 强制控制器内的所有操作返回 JSON 格式的响应。
* 若已配置其他格式化程序，并且客户端指定了其他格式，将返回 JSON。

有关详细信息，请参阅[筛选器](xref:mvc/controllers/filters)。

### <a name="special-case-formatters"></a>特例格式化程序

一些特例是使用内置格式化程序实现的。 默认情况下，`string` 返回类型的格式将设为 text/plain（如果通过 `Accept` 标头请求则为 text/html）  。 可以通过删除 <xref:Microsoft.AspNetCore.Mvc.Formatters.StringOutputFormatter> 删除此行为。 在 `ConfigureServices` 方法中删除格式化程序。 有模型对象返回类型的操作将在返回 `null` 时返回 `204 No Content`。 可以通过删除 <xref:Microsoft.AspNetCore.Mvc.Formatters.HttpNoContentOutputFormatter> 删除此行为。 以下代码删除 `StringOutputFormatter` 和 `HttpNoContentOutputFormatter`。

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](./formatting/3.0sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end
::: moniker range="< aspnetcore-3.0"
[!code-csharp[](./formatting/sample/StartupStringOutputFormatter.cs?name=snippet)]
::: moniker-end

如果没有 `StringOutputFormatter`，内置 JSON 格式化程序将设置 `string` 返回类型的格式。 如果删除了内置 JSON 格式化程序并提供了 XML 格式化程序，则 XML 格式化程序将设置 `string` 返回类型的格式。 否则，`string` 返回类型返回 `406 Not Acceptable`。

没有 `HttpNoContentOutputFormatter`，null 对象将使用配置的格式化程序来进行格式设置。 例如： 。

* JSON 格式化程序返回正文为 `null` 的响应。
* 设置属性 `xsi:nil="true"` 时，XML 格式化程序返回空 XML 元素。

## <a name="response-format-url-mappings"></a>响应格式 URL 映射

客户端可以在 URL 中请求特定格式，例如：

* 在查询字符串中，或在路径中。
* 使用格式特定的文件扩展名，如 .xml 或 .json。

请求路径的映射必须在 API 使用的路由中指定。 例如： 。

[!code-csharp[](./formatting/sample/Controllers/ProductsController.cs?name=snippet)]

上述路由将允许指定所请求格式为可选文件扩展名。 [`[FormatFilter]`](xref:Microsoft.AspNetCore.Mvc.FormatFilterAttribute)特性检查中的格式值是否存在 `RouteData` ，并在创建响应时将响应格式映射到适当的格式化程序。

|           路由        |             格式化程序              |
|------------------------|------------------------------------|
|   `/api/products/5`    |    默认输出格式化程序    |
| `/api/products/5.json` | JSON 格式化程序（如配置） |
| `/api/products/5.xml`  | XML 格式化程序（如配置）  |
