---
title: " (CORS) 启用跨域请求 ASP.NET Core"
author: rick-anderson
description: 了解如何在 ASP.NET Core 应用中允许或拒绝跨源请求的标准。
ms.author: riande
ms.custom: mvc
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
uid: security/cors
ms.openlocfilehash: 03008f40fc1c4b323535d08a1bb4c4007bc145f7
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060815"
---
# <a name="enable-cross-origin-requests-cors-in-aspnet-core"></a> (CORS) 启用跨域请求 ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Kirk Larkin](https://twitter.com/serpent5)

本文介绍如何在 ASP.NET Core 的应用程序中启用 CORS。

浏览器安全性可防止网页向不处理网页的域发送请求。 此限制称为同域策略。 同域策略可防止恶意站点从另一站点读取敏感数据。 有时，你可能想要允许其他站点对你的应用进行跨域请求。 有关详细信息，请参阅 [MOZILLA CORS 一文](https://developer.mozilla.org/docs/Web/HTTP/CORS)。

[跨源资源共享](https://www.w3.org/TR/cors/) (CORS) ：

* 是一种 W3C 标准，可让服务器放宽相同的源策略。
* **不** 是一项安全功能，CORS 放宽 security。 API 不能通过允许 CORS 来更安全。 有关详细信息，请参阅 [CORS 的工作](#how-cors)原理。
* 允许服务器明确允许一些跨源请求，同时拒绝其他请求。
* 比早期的技术（如 [JSONP](/dotnet/framework/wcf/samples/jsonp)）更安全且更灵活。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/3.1sample/Cors/WebAPI)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="same-origin"></a>同一原点

如果两个 Url 具有相同的方案、主机和端口 ([RFC 6454](https://tools.ietf.org/html/rfc6454)) ，则它们具有相同的源。

这两个 Url 具有相同的源：

* `https://example.com/foo.html`
* `https://example.com/bar.html`

这些 Url 的起源不同于前两个 Url：

* `https://example.net`：不同的域
* `https://www.example.com/foo.html`：不同的子域
* `http://example.com/foo.html`：不同方案
* `https://example.com:9000/foo.html`：不同的端口

## <a name="enable-cors"></a>启用 CORS

有三种方法可启用 CORS：

* 使用 [命名策略](#np) 或 [默认策略](#dp)的中间件。
* 使用 [终结点路由](#ecors)。
* 带有 [[EnableCors]](#attr) 属性的。

通过命名策略使用 [[EnableCors]](#attr) 属性，可在限制支持 CORS 的终结点时提供最佳控制。

> [!WARNING]
> <xref:Owin.CorsExtensions.UseCors%2A> 使用时，必须调用 <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> `UseResponseCaching` 。

以下各节详细介绍了每种方法。

<a name="np"></a>

## <a name="cors-with-named-policy-and-middleware"></a>具有命名策略和中间件的 CORS

CORS 中间件处理跨域请求。 以下代码将 CORS 策略应用到具有指定来源的所有应用的终结点：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup.cs?name=snippet&highlight=3,9,32)]

前面的代码：

* 将策略名称设置为 `_myAllowSpecificOrigins` 。 策略名称为任意名称。
* 调用 <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors*> 扩展方法并指定  `_myAllowSpecificOrigins` CORS 策略。 `UseCors` 添加 CORS 中间件。 必须将对的调用 `UseCors` 置于之后 `UseRouting` 但在之前 `UseAuthorization` 。 有关详细信息，请参阅 [中间件顺序](xref:fundamentals/middleware/index#middleware-order)。
* <xref:Microsoft.Extensions.DependencyInjection.CorsServiceCollectionExtensions.AddCors*>使用[lambda 表达式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)调用。 Lambda 采用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> 对象。 本文稍后将介绍[配置选项](#cors-policy-options)，如 `WithOrigins` 。
* 启用 `_myAllowSpecificOrigins` 所有控制器终结点的 CORS 策略。 请参阅 [终结点路由](#ecors) ，将 CORS 策略应用到特定终结点。
* 使用 [响应缓存中间件](xref:performance/caching/middleware)时，请 <xref:Owin.CorsExtensions.UseCors%2A> 先调用 <xref:Microsoft.AspNetCore.Builder.ResponseCachingExtensions.UseResponseCaching%2A> 。

通过终结点路由，CORS 中间件 **必须** 配置为在对和的调用之间执行 `UseRouting` `UseEndpoints` 。

有关测试代码的说明，请参阅 [测试 CORS](#testc) ，如以上代码所示。

<xref:Microsoft.Extensions.DependencyInjection.MvcCorsMvcCoreBuilderExtensions.AddCors*>方法调用将 CORS 服务添加到应用的服务容器：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup.cs?name=snippet2)]

有关详细信息，请参阅本文档中的 [CORS 策略选项](#cpo) 。

这些 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> 方法可以链接在一起，如以下代码所示：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup2.cs?name=snippet)]

注意：指定的 URL **不** 能包含尾随斜杠 (`/`) 。 如果 URL 以结尾 `/` ，则比较返回， `false` 不返回任何标头。

<a name="dp"></a>

### <a name="cors-with-default-policy-and-middleware"></a>具有默认策略和中间件的 CORS

以下突出显示的代码将启用默认 CORS 策略：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupDefaultPolicy.cs?name=snippet2&highlight=7,29)]

前面的代码将默认的 CORS 策略应用到所有控制器终结点。

<a name="ecors"></a>

## <a name="enable-cors-with-endpoint-routing"></a>通过终结点路由启用 Cors

目前使用每个终结点启用 CORS 不 `RequireCors` 支持 [自动预检请求](#apf)。 **not** 有关详细信息，请参阅 [此 GitHub 问题](https://github.com/dotnet/aspnetcore/issues/20709) 和 [测试与终结点路由和 [HTTPOPTIONS] 的 CORS](#tcer)。

使用终结点路由，可以使用一组扩展方法在每个终结点上启用 CORS <xref:Microsoft.AspNetCore.Builder.CorsEndpointConventionBuilderExtensions.RequireCors*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupEndPt.cs?name=snippet2&highlight=3,7-15,32,40,43)]

在上述代码中：

* `app.UseCors` 启用 CORS 中间件。 由于尚未配置默认策略，因此 `app.UseCors()` 单独不启用 CORS。
* `/echo`和控制器端点允许使用指定策略的跨域请求。
* `/echo2`和 Razor Pages 终结点 **不** 允许跨源请求，因为未指定默认策略。

[[DisableCors]](#dc)特性 **不** 会禁用通过终结点路由启用的 CORS `RequireCors` 。

请参阅 [测试与终结点路由和 [HttpOptions] 的 CORS](#tcer) ，获取与前面类似的代码测试说明。

<a name="attr"></a>

## <a name="enable-cors-with-attributes"></a>使用属性启用 CORS

使用 [[EnableCors]](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute) 属性启用 CORS，并将命名策略应用到只有那些需要 CORS 的终结点提供了精细的控制。

[[EnableCors]](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute)属性提供了一种用于全局应用 CORS 的替代方法。 `[EnableCors]`特性启用所选终结点的 CORS，而不是所有终结点：

* `[EnableCors]` 指定默认策略。
* `[EnableCors("{Policy String}")]` 指定命名策略。

`[EnableCors]`特性可应用于：

* Razor 分页 `PageModel`
* 控制器
* 控制器操作方法

可以将不同的策略应用到具有属性的控制器、页面模型或操作方法 `[EnableCors]` 。 如果将 `[EnableCors]` 属性应用于控制器、页面模型或操作方法，并在中间件中启用了 CORS，则会应用 **这两种** 策略。 **建议不要结合策略。使用** `[EnableCors]` **特性或中间件，而不是在同一应用中。**

下面的代码将不同的策略应用于每个方法：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/WidgetController.cs?name=snippet&highlight=6,14)]

下面的代码创建两个 CORS 策略：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Startup3.cs?name=snippet&highlight=12-28,44)]

对于限制 CORS 请求的最佳控制：

* `[EnableCors("MyPolicy")]`与命名策略一起使用。
* 不要定义默认策略。
* 请勿使用 [终结点路由](#ecors)。

下一节中的代码满足前面的列表。

有关测试代码的说明，请参阅 [测试 CORS](#testc) ，如以上代码所示。

<a name="dc"></a>

### <a name="disable-cors"></a>禁用 CORS

[[DisableCors]](xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute) **特性不会禁用已** 通过 [终结点路由](#ecors)启用的 CORS。

以下代码定义 CORS 策略 `"MyPolicy"` ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupTestMyPolicy.cs?name=snippet)]

以下代码禁用操作的 CORS `GetValues2` ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/ValuesController.cs?name=snippet&highlight=1,23)]

前面的代码：

* 不会使用 [终结点路由](#ecors)启用 CORS。
* 不定义 [默认的 CORS 策略](#dp)。
* 使用 [[EnableCors ( "MyPolicy" ) ]](#attr) 启用控制器的 `"MyPolicy"` CORS 策略。
* 为方法禁用 CORS `GetValues2` 。

有关测试上述代码的说明，请参阅 [测试 CORS](#testc) 。

<a name="cpo"></a>

## <a name="cors-policy-options"></a>CORS 策略选项

本部分介绍可在 CORS 策略中设置的各种选项：

* [设置允许的来源](#set-the-allowed-origins)
* [设置允许的 HTTP 方法](#set-the-allowed-http-methods)
* [设置允许的请求标头](#set-the-allowed-request-headers)
* [设置公开的响应标头](#set-the-exposed-response-headers)
* [跨域请求中的凭据](#credentials-in-cross-origin-requests)
* [设置预检过期时间](#set-the-preflight-expiration-time)

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsOptions.AddPolicy*> 在中调用 `Startup.ConfigureServices` 。 对于某些选项，最好先阅读 [CORS 如何工作](#how-cors) 部分。

## <a name="set-the-allowed-origins"></a>设置允许的来源

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>：允许所有来源的 CORS 请求与任何方案 (`http` 或 `https`) 。 `AllowAnyOrigin` 是不安全的，因为 *任何网站* 都可以向应用程序发出跨域请求。

> [!NOTE]
> 指定 `AllowAnyOrigin` 和 `AllowCredentials` 是不安全的配置，并可能导致跨站点请求伪造。 使用这两种方法配置应用时，CORS 服务将返回无效的 CORS 响应。

`AllowAnyOrigin` 影响预检请求和 `Access-Control-Allow-Origin` 标头。 有关详细信息，请参阅 [预检请求](#preflight-requests) 部分。

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetIsOriginAllowedToAllowWildcardSubdomains*>：将 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.IsOriginAllowed*> 策略的属性设置为一个函数，当计算是否允许源时，此函数允许源匹配已配置的通配符域。

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet)]

### <a name="set-the-allowed-http-methods"></a>设置允许的 HTTP 方法

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyMethod*>:

* 允许任何 HTTP 方法：
* 影响预检请求和 `Access-Control-Allow-Methods` 标头。 有关详细信息，请参阅 [预检请求](#preflight-requests) 部分。

### <a name="set-the-allowed-request-headers"></a>设置允许的请求标头

若要允许在 CORS 请求中发送特定标头（称为 [作者请求标头](https://xhr.spec.whatwg.org/#request)），请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> 并指定允许的标头：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet2)]

若要允许所有 [作者请求标头](https://www.w3.org/TR/cors/#author-request-headers)，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet3)]

`AllowAnyHeader` 影响预检请求和 [访问控制请求标](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Request-Method) 头。 有关详细信息，请参阅 [预检请求](#preflight-requests) 部分。

`WithHeaders`仅当发送的标头 `Access-Control-Request-Headers` 与中所述的标头完全匹配时，才可以使用 CORS 中间件策略匹配指定的特定标头 `WithHeaders` 。

例如，考虑按如下方式配置的应用：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet4)]

CORS 中间件使用以下请求标头拒绝预检请求，因为 `Content-Language`) 中未列出 ([HeaderNames. ContentLanguage](xref:Microsoft.Net.Http.Headers.HeaderNames.ContentLanguage) `WithHeaders` ：

```
Access-Control-Request-Headers: Cache-Control, Content-Language
```

应用返回 *200 OK* 响应，但不会向后发送 CORS 标头。 因此，浏览器不会尝试跨域请求。

### <a name="set-the-exposed-response-headers"></a>设置公开的响应标头

默认情况下，浏览器不会向应用程序公开所有的响应标头。 有关详细信息，请参阅 [W3C 跨域资源共享 (术语) ：简单的响应标头](https://www.w3.org/TR/cors/#simple-response-header)。

默认情况下可用的响应标头包括：

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

CORS 规范将这些标头称为 *简单的响应标头* 。 若要使其他标头可用于应用程序，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet5)]
### <a name="credentials-in-cross-origin-requests"></a>跨域请求中的凭据

凭据需要在 CORS 请求中进行特殊处理。 默认情况下，浏览器不会使用跨域请求发送凭据。 凭据包括 cookie s 和 HTTP 身份验证方案。 若要使用跨域请求发送凭据，客户端必须设置 `XMLHttpRequest.withCredentials` 为 `true` 。

`XMLHttpRequest`直接使用：

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'https://www.example.com/api/test');
xhr.withCredentials = true;
```

使用 jQuery：

```javascript
$.ajax({
  type: 'get',
  url: 'https://www.example.com/api/test',
  xhrFields: {
    withCredentials: true
  }
});
```

使用 [提取 API](https://developer.mozilla.org/docs/Web/API/Fetch_API)：

```javascript
fetch('https://www.example.com/api/test', {
    credentials: 'include'
});
```

服务器必须允许凭据。 若要允许跨域凭据，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowCredentials*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet6)]

HTTP 响应包含一个 `Access-Control-Allow-Credentials` 标头，通知浏览器服务器允许跨源请求的凭据。

如果浏览器发送凭据，但响应不包含有效的 `Access-Control-Allow-Credentials` 标头，则浏览器不会向应用程序公开响应，而且跨源请求会失败。

允许跨域凭据会带来安全风险。 另一个域中的网站可以代表用户将登录用户的凭据发送给该应用程序，而无需用户的知识。 <!-- TODO Review: When using `AllowCredentials`, all CORS enabled domains must be trusted.
I don't like "all CORS enabled domains must be trusted", because it implies that if you're not using  `AllowCredentials`, domains don't need to be trusted. -->

CORS 规范还指出， `"*"` 如果 `Access-Control-Allow-Credentials` 标头存在，则 (所有源) 的设置源无效。

<a name="pref"></a>

## <a name="preflight-requests"></a>预检请求

对于某些 CORS 请求，浏览器会在发出实际请求之前发送额外的 [OPTIONS](https://developer.mozilla.org/docs/Web/HTTP/Methods/OPTIONS) 请求。 此请求称为 [预检请求](https://developer.mozilla.org/docs/Glossary/Preflight_request)。 如果满足以下 **所有** 条件，浏览器可以跳过预检请求：

* 请求方法为 GET、HEAD 或 POST。
* 应用不会设置、、、或以外的请求标头 `Accept` `Accept-Language` `Content-Language` `Content-Type` `Last-Event-ID` 。
* `Content-Type`标头（如果已设置）具有以下值之一：
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

为客户端请求设置的请求标头上的规则适用于应用通过在对象上调用来设置的标头 `setRequestHeader` `XMLHttpRequest` 。 CORS 规范调用这些标头 [作者请求标头](https://www.w3.org/TR/cors/#author-request-headers)。 此规则不适用于浏览器可以设置的标头，如 `User-Agent` 、 `Host` 或 `Content-Length` 。

下面是一个示例响应，它类似于在本文档的 " [测试 CORS](#testc) " 部分中通过 " **Put test** " 按钮发出的预检请求。

```
General:
Request URL: https://cors3.azurewebsites.net/api/values/5
Request Method: OPTIONS
Status Code: 204 No Content

Response Headers:
Access-Control-Allow-Methods: PUT,DELETE,GET
Access-Control-Allow-Origin: https://cors1.azurewebsites.net
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f8...8;Path=/;HttpOnly;Domain=cors1.azurewebsites.net
Vary: Origin

Request Headers:
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Method: PUT
Connection: keep-alive
Host: cors3.azurewebsites.net
Origin: https://cors1.azurewebsites.net
Referer: https://cors1.azurewebsites.net/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0
```

预检请求使用 [HTTP OPTIONS](https://developer.mozilla.org/docs/Web/HTTP/Methods/OPTIONS) 方法。 它可能包含以下标头：

* [访问控制-请求-方法](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Request-Method)：将用于实际请求的 HTTP 方法。
* [访问控制-请求标头](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Headers)：应用在实际请求上设置的请求标头的列表。 如前文所述，这不包含浏览器设置的标头，如 `User-Agent` 。
* [访问控制-允许-方法](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Methods)

如果预检请求被拒绝，应用将返回响应， `200 OK` 但不会设置 CORS 标头。 因此，浏览器不会尝试跨域请求。 有关拒绝的预检请求的示例，请参阅本文档的 [测试 CORS](#testc) 部分。

使用 F12 工具时，控制台应用会显示类似于以下内容之一的错误，具体取决于浏览器：

* Firefox：跨源请求被阻止：相同的源策略不允许读取上的远程资源 `https://cors1.azurewebsites.net/api/TodoItems1/MyDelete2/5` 。  (原因： CORS 请求未成功) 。 [了解详细信息](https://developer.mozilla.org/docs/Web/HTTP/CORS/Errors/CORSDidNotSucceed)
* 基于 Chromium：派生自源 "" 的 "" 访问权限已被 https://cors1.azurewebsites.net/api/TodoItems1/MyDelete2/5 https://cors3.azurewebsites.net CORS 策略阻止：响应预检请求未通过访问控制检查：请求的资源上没有 "访问控制-允许-源" 标头。 如果非跳转响应可满足需求，请将请求的模式设置为“no-cors”，以便在禁用 CORS 的情况下提取资源。

若要允许特定标头，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet2)]

若要允许所有 [作者请求标头](https://www.w3.org/TR/cors/#author-request-headers)，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet3)]

浏览器的设置方式并不一致 `Access-Control-Request-Headers` 。 如果：

* 标头设置为以外的任何内容 `"*"`
* <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.AllowAnyHeader*> 调用：至少包含 `Accept` 、 `Content-Type` 和 `Origin` ，以及要支持的任何自定义标头。

<a name="apf"></a>

### <a name="automatic-preflight-request-code"></a>自动预检请求代码

应用 CORS 策略的时间：

* 通过 `app.UseCors` 在中调用 `Startup.Configure` 。
* 使用 `[EnableCors]` 特性。

ASP.NET Core 对 "预检选项" 请求做出响应。

目前使用每个终结点启用 CORS 不 `RequireCors` 支持自动 **not** 预检请求。

本文档的 " [测试 CORS](#testc) " 部分说明了此行为。

<a name="pro"></a>

### <a name="httpoptions-attribute-for-preflight-requests"></a>用于预检请求的 [HttpOptions] 属性

如果为 CORS 启用了适当的策略，ASP.NET Core 通常会自动响应 CORS 预检请求。 在某些情况下，可能不会出现这种情况。 例如，将 [CORS 用于终结点路由](#ecors)。

下面的代码使用 [[HttpOptions]](xref:Microsoft.AspNetCore.Mvc.HttpOptionsAttribute) 特性为 OPTIONS 请求创建终结点：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems2Controller.cs?name=snippet&highlight=5-17)]

有关测试上述代码的说明，请参阅 [通过终结点路由测试 CORS 和 [HttpOptions]](#tcer) 。

### <a name="set-the-preflight-expiration-time"></a>设置预检过期时间

`Access-Control-Max-Age`标头指定可缓存对预检请求的响应的时间长度。 若要设置此标头，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetPreflightMaxAge*> ：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupAllowSubdomain.cs?name=snippet7)]
<a name="how-cors"></a>

## <a name="how-cors-works"></a>CORS 如何工作

本部分介绍 HTTP 消息级别的 [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) 请求中发生的情况。

* CORS **不** 是一种安全功能。 CORS 是一种 W3C 标准，可让服务器放宽相同的源策略。
  * 例如，恶意执行组件可能对站点使用 [跨站点脚本 (XSS) ](xref:security/cross-site-scripting) ，并向启用了 CORS 的站点执行跨站点请求来窃取信息。
* API 不能通过允许 CORS 来更安全。
  * 它由客户端 (浏览器) 来强制执行 CORS。 服务器执行请求并返回响应，这是返回错误并阻止响应的客户端。 例如，以下任何工具都将显示服务器响应：
    * [Fiddler](https://www.telerik.com/fiddler)
    * [Postman](https://www.getpostman.com/)
    * [.NET HttpClient](/dotnet/csharp/tutorials/console-webapiclient)
    * Web 浏览器，方法是在地址栏中输入 URL。
* 这是一种方法，使服务器能够允许浏览器执行跨源 [XHR](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) 或 [获取 API](https://developer.mozilla.org/docs/Web/API/Fetch_API) 请求，否则将被禁止。
  * 没有 CORS 的浏览器不能执行跨域请求。 在 CORS 之前，使用 [JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) 来绕过此限制。 JSONP 不使用 XHR，而是使用 `<script>` 标记接收响应。 允许跨源加载脚本。

[CORS 规范](https://www.w3.org/TR/cors/)介绍了几个新的 HTTP 标头，它们启用了跨域请求。 如果浏览器支持 CORS，则会自动为跨域请求设置这些标头。 若要启用 CORS，无需自定义 JavaScript 代码。

部署的[示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/3.1sample/Cors/WebAPI)上的 " [PUT 测试" 按钮](https://cors3.azurewebsites.net/test)

下面是一个从 " [值](https://cors3.azurewebsites.net/) " 测试按钮到的跨源请求的示例 `https://cors1.azurewebsites.net/api/values` 。 `Origin`标头：

* 提供发出请求的站点的域。
* 是必需的，并且必须与主机不同。

**常规标头**

```
Request URL: https://cors1.azurewebsites.net/api/values
Request Method: GET
Status Code: 200 OK
```

**响应标头**

```
Content-Encoding: gzip
Content-Type: text/plain; charset=utf-8
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f...;Path=/;HttpOnly;Domain=cors1.azurewebsites.net
Transfer-Encoding: chunked
Vary: Accept-Encoding
X-Powered-By: ASP.NET
```

**请求标头**

```
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: keep-alive
Host: cors1.azurewebsites.net
Origin: https://cors3.azurewebsites.net
Referer: https://cors3.azurewebsites.net/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0 ...
```

在 `OPTIONS` 请求中，服务器设置响应中的 **响应标头** `Access-Control-Allow-Origin: {allowed origin}` 标头。 例如，已部署的 [示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/3.1sample/Cors/WebAPI) [Delete [EnableCors]](https://cors1.azurewebsites.net/test?number=2) button `OPTIONS` 请求包含以下标头：

**常规标头**

```
Request URL: https://cors3.azurewebsites.net/api/TodoItems2/MyDelete2/5
Request Method: OPTIONS
Status Code: 204 No Content
```

**响应标头**

```
Access-Control-Allow-Headers: Content-Type,x-custom-header
Access-Control-Allow-Methods: PUT,DELETE,GET,OPTIONS
Access-Control-Allow-Origin: https://cors1.azurewebsites.net
Server: Microsoft-IIS/10.0
Set-Cookie: ARRAffinity=8f...;Path=/;HttpOnly;Domain=cors3.azurewebsites.net
Vary: Origin
X-Powered-By: ASP.NET
```

**请求标头**

```
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Headers: content-type
Access-Control-Request-Method: DELETE
Connection: keep-alive
Host: cors3.azurewebsites.net
Origin: https://cors1.azurewebsites.net
Referer: https://cors1.azurewebsites.net/test?number=2
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: cross-site
User-Agent: Mozilla/5.0
```

在前面的 **响应标头** 中，服务器设置响应中的 [访问控制允许源](https://developer.mozilla.org/docs/Web/HTTP/Headers/Access-Control-Allow-Origin) 标头。 `https://cors1.azurewebsites.net`此标头的值与 `Origin` 请求中的标头相匹配。

如果 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*> 调用了，则将 `Access-Control-Allow-Origin: *` 返回通配符值。 `AllowAnyOrigin` 允许任何源。

如果响应不包含 `Access-Control-Allow-Origin` 标头，则跨域请求会失败。 具体而言，浏览器不允许该请求。 即使服务器返回成功的响应，浏览器也不会将响应提供给客户端应用程序。

<a name="options"></a>

### <a name="display-options-requests"></a>显示选项请求

默认情况下，Chrome 和 Edge 浏览器不会在 F12 工具的 "网络" 选项卡上显示 "请求" 选项。 若要在这些浏览器中显示选项请求：

* `chrome://flags/#out-of-blink-cors` 或 `edge://flags/#out-of-blink-cors`
* 禁用标志。
* 重新启动.

默认情况下，Firefox 显示 "选项请求"。

## <a name="cors-in-iis"></a>IIS 中的 CORS

部署到 IIS 时，如果未将服务器配置为允许匿名访问，则必须在 Windows 身份验证之前运行 CORS。 若要支持此方案，需要为应用安装和配置 [IIS CORS 模块](https://www.iis.net/downloads/microsoft/iis-cors-module) 。

<a name="testc"></a>

## <a name="test-cors"></a>测试 CORS

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/3.1sample/Cors/WebAPI)包含测试 CORS 的代码。 请参阅[如何下载](xref:index#how-to-download-a-sample)。 该示例是一个 API 项目，其中 Razor 添加了页面：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupTest2.cs?name=snippet2)]

  > [!WARNING]
  > `WithOrigins("https://localhost:<port>");` 应仅用于测试示例应用程序，类似于 [下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/cors/3.1sample/Cors)。

下面 `ValuesController` 提供用于测试的终结点：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/ValuesController.cs?name=snippet)]

[MyDisplayRouteInfo](https://github.com/Rick-Anderson/RouteInfo/blob/master/Microsoft.Docs.Samples.RouteInfo/ControllerContextExtensions.cs) 由 [Rick.DocRouteInfo](https://www.nuget.org/packages/Rick.Docs.Samples.RouteInfo) NuGet 包提供，并显示路由信息。

使用以下方法之一测试前面的示例代码：

* 使用部署的示例应用 [https://cors3.azurewebsites.net/](https://cors3.azurewebsites.net/) 。 无需下载示例。
* `dotnet run`使用的默认 URL 运行示例 `https://localhost:5001` 。
* 从 Visual Studio 运行示例，并将端口设置为44398，查找的 URL `https://localhost:44398` 。

使用带有 F12 工具的浏览器：

* 选择 " **值** " 按钮，然后查看 " **网络** " 选项卡中的标头。
* 选择 " **放置测试** " 按钮。 请参阅 [显示选项请求](#options) ，以获取有关显示选项请求的说明。 **PUT 测试** 创建两个请求：一个选项预检请求和 PUT 请求。
* 选择此 **`GetValues2 [DisableCors]`** 按钮可触发失败的 CORS 请求。 如文档中所述，响应返回200成功，但不进行 CORS 请求。 选择 " **控制台** " 选项卡以查看 CORS 错误。 根据浏览器，将显示类似于以下内容的错误：

     `'https://cors1.azurewebsites.net/api/values/GetValues2'`CORS 策略已阻止从原始位置获取的访问权限 `'https://cors3.azurewebsites.net'` ：请求的资源上没有 "访问控制-允许" 标头。 如果非跳转响应可满足需求，请将请求的模式设置为“no-cors”，以便在禁用 CORS 的情况下提取资源。
     
可以使用[Fiddler](https://www.telerik.com/fiddler)或[Postman](https://www.getpostman.com/)等工具来测试启用了[CORS 的终结](https://curl.haxx.se/)点。 使用工具时，标头指定的请求源 `Origin` 必须与接收请求的主机不同。 如果请求不是基于标头值 *跨* 域的，则 `Origin` ：

* 不需要 CORS 中间件来处理请求。
* 不会在响应中返回 CORS 标头。

以下命令使用 `curl` 发出带有以下信息的选项请求：

```bash
curl -X OPTIONS https://cors3.azurewebsites.net/api/TodoItems2/5 -i
```

<!--
curl come with Git. Add to path variable
C:\Program Files\Git\mingw64\bin\
-->

<a name="tcer"></a>

### <a name="test-cors-with-endpoint-routing-and-httpoptions"></a>通过终结点路由和 [HttpOptions] 测试 CORS

目前使用每个终结点启用 CORS 不 `RequireCors` 支持 [自动预检请求](#apf)。 **not** 请考虑以下代码，它使用 [终结点路由启用 CORS](#ecors)：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/StartupEndPointBugTest.cs?name=snippet2)]

下面 `TodoItems1Controller` 提供用于测试的终结点：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems1Controller.cs?name=snippet2)]

从已部署[示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/3.1sample/Cors/WebAPI)的[测试页](https://cors1.azurewebsites.net/test?number=1)测试前面的代码。

**Delete [EnableCors]** 和 **GET [EnableCors]** 按钮成功，因为终结点具有 `[EnableCors]` 和响应预检请求。 其他终结点失败。 " **获取** " 按钮失败，因为 [JavaScript](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/3.1sample/Cors/WebAPI/wwwroot/js/MyJS.js) 发送：

```javascript
 headers: {
      "Content-Type": "x-custom-header"
 },
```

下面 `TodoItems2Controller` 提供了类似的终结点，但包含响应选项请求的显式代码：

[!code-csharp[](cors/3.1sample/Cors/WebAPI/Controllers/TodoItems2Controller.cs?name=snippet2)]

从已部署示例的 [测试页](https://cors1.azurewebsites.net/test?number=2) 测试前面的代码。 在 " **控制器** " 下拉列表中，选择 " **预检** "，然后 **设置 "控制器** "。 对终结点的所有 CORS 调用都将 `TodoItems2Controller` 成功。

## <a name="additional-resources"></a>其他资源

* [ (CORS) 跨域资源共享 ](https://developer.mozilla.org/docs/Web/HTTP/CORS)
* [IIS CORS 模块入门](https://blogs.iis.net/iisteam/getting-started-with-the-iis-cors-module)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

本文介绍如何在 ASP.NET Core 的应用程序中启用 CORS。

浏览器安全性可防止网页向不处理网页的域发送请求。 此限制称为同域策略。 同域策略可防止恶意站点从另一站点读取敏感数据。 有时，你可能想要允许其他站点对你的应用进行跨域请求。 有关详细信息，请参阅 [MOZILLA CORS 一文](https://developer.mozilla.org/docs/Web/HTTP/CORS)。

[跨源资源共享](https://www.w3.org/TR/cors/) (CORS) ：

* 是一种 W3C 标准，可让服务器放宽相同的源策略。
* **不** 是一项安全功能，CORS 放宽 security。 API 不能通过允许 CORS 来更安全。 有关详细信息，请参阅 [CORS 的工作](#how-cors)原理。
* 允许服务器明确允许一些跨源请求，同时拒绝其他请求。
* 比早期的技术（如 [JSONP](/dotnet/framework/wcf/samples/jsonp)）更安全且更灵活。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="same-origin"></a>同一原点

如果两个 Url 具有相同的方案、主机和端口 ([RFC 6454](https://tools.ietf.org/html/rfc6454)) ，则它们具有相同的源。

这两个 Url 具有相同的源：

* `https://example.com/foo.html`
* `https://example.com/bar.html`

这些 Url 的起源不同于前两个 Url：

* `https://example.net`：不同的域
* `https://www.example.com/foo.html`：不同的子域
* `http://example.com/foo.html`：不同方案
* `https://example.com:9000/foo.html`：不同的端口

比较来源时，Internet Explorer 不会考虑该端口。

## <a name="cors-with-named-policy-and-middleware"></a>具有命名策略和中间件的 CORS

CORS 中间件处理跨域请求。 以下代码通过指定源为整个应用启用 CORS：

[!code-csharp[](cors/sample/Cors/WebAPI/Startup.cs?name=snippet&highlight=8,14-23,38)]

前面的代码：

* 将策略名称设置为 " \_ myAllowSpecificOrigins"。 策略名称为任意名称。
* 调用 <xref:Microsoft.AspNetCore.Builder.CorsMiddlewareExtensions.UseCors*> 扩展方法，该方法启用 CORS。
* <xref:Microsoft.Extensions.DependencyInjection.CorsServiceCollectionExtensions.AddCors*>使用[lambda 表达式](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions)调用。 Lambda 采用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder> 对象。 本文稍后将介绍[配置选项](#cors-policy-options)，如 `WithOrigins` 。

<xref:Microsoft.Extensions.DependencyInjection.MvcCorsMvcCoreBuilderExtensions.AddCors*>方法调用将 CORS 服务添加到应用的服务容器：

[!code-csharp[](cors/sample/Cors/WebAPI/Startup.cs?name=snippet2)]

有关详细信息，请参阅本文档中的 [CORS 策略选项](#cpo) 。

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder>方法可以链接方法，如以下代码所示：

[!code-csharp[](cors/sample/Cors/WebAPI/Startup2.cs?name=snippet2)]

注意： URL **不** 能包含尾随斜杠 (`/`) 。 如果 URL 以结尾 `/` ，则比较返回， `false` 不返回任何标头。

以下代码通过 CORS 中间件将 CORS 策略应用到所有应用终结点：
```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseCors();

    app.UseHttpsRedirection();
    app.UseMvc();
}
```
注意： `UseCors` 必须先调用 `UseMvc` 。

请参阅 [在页面 Razor 、控制器和操作方法中启用 cors，](#ecors) 以在页面/控制器/操作级别应用 cors 策略。

有关测试代码的说明，请参阅 [测试 CORS](#test) ，如以上代码所示。

## <a name="enable-cors-with-attributes"></a>使用属性启用 CORS

[ &lbrack; EnableCors &rbrack; ](xref:Microsoft.AspNetCore.Cors.EnableCorsAttribute)属性提供了一种用于全局应用 CORS 的替代方法。 `[EnableCors]`属性为所选终结点启用 CORS，而不是所有终结点。

使用 `[EnableCors]` 指定默认策略并 `[EnableCors("{Policy String}")]` 指定策略。

`[EnableCors]`特性可应用于：

* Razor 分页 `PageModel`
* 控制器
* 控制器操作方法

您可以使用属性将不同的策略应用到控制器/页模型/操作  `[EnableCors]` 。 如果将 `[EnableCors]` 属性应用于控制器/页面模型/操作方法，并在中间件中启用了 CORS，则会应用 **这两种** 策略。 建议 **不要** 合并策略。 使用 `[EnableCors]` 特性或中间件， **不能同时使用两者** 。 使用时 `[EnableCors]` ，请 **不要** 定义默认策略。

下面的代码将不同的策略应用于每个方法：

[!code-csharp[](cors/sample/Cors/WebAPI/Controllers/WidgetController.cs?name=snippet&highlight=6,14)]

以下代码创建 CORS 默认策略和名为的策略 `"AnotherPolicy"` ：

[!code-csharp[](cors/sample/Cors/WebAPI/StartupMultiPolicy.cs?name=snippet&highlight=12-28)]

### <a name="disable-cors"></a>禁用 CORS

[ &lbrack; DisableCors &rbrack; ](xref:Microsoft.AspNetCore.Cors.DisableCorsAttribute)属性为控制器/页模型/操作禁用 CORS。

<a name="cpo"></a>

## <a name="cors-policy-options"></a>CORS 策略选项

本部分介绍可在 CORS 策略中设置的各种选项：

* [设置允许的来源](#set-the-allowed-origins)
* [设置允许的 HTTP 方法](#set-the-allowed-http-methods)
* [设置允许的请求标头](#set-the-allowed-request-headers)
* [设置公开的响应标头](#set-the-exposed-response-headers)
* [跨域请求中的凭据](#credentials-in-cross-origin-requests)
* [设置预检过期时间](#set-the-preflight-expiration-time)

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsOptions.AddPolicy*> 在中调用 `Startup.ConfigureServices` 。 对于某些选项，最好先阅读 [CORS 如何工作](#how-cors) 部分。

## <a name="set-the-allowed-origins"></a>设置允许的来源

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyOrigin*>：允许所有来源的 CORS 请求与任何方案 (`http` 或 `https`) 。 `AllowAnyOrigin` 是不安全的，因为 *任何网站* 都可以向应用程序发出跨域请求。

> [!NOTE]
> 指定 `AllowAnyOrigin` 和 `AllowCredentials` 是不安全的配置，并可能导致跨站点请求伪造。 对于安全应用，如果客户端必须授权自身访问服务器资源，请指定准确的来源列表。

`AllowAnyOrigin` 影响预检请求和 `Access-Control-Allow-Origin` 标头。 有关详细信息，请参阅 [预检请求](#preflight-requests) 部分。

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetIsOriginAllowedToAllowWildcardSubdomains*>：将 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.IsOriginAllowed*> 策略的属性设置为一个函数，当计算是否允许源时，此函数允许源匹配已配置的通配符域。

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=100-105&highlight=4-5)]

### <a name="set-the-allowed-http-methods"></a>设置允许的 HTTP 方法

<xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyMethod*>:

* 允许任何 HTTP 方法：
* 影响预检请求和 `Access-Control-Allow-Methods` 标头。 有关详细信息，请参阅 [预检请求](#preflight-requests) 部分。

### <a name="set-the-allowed-request-headers"></a>设置允许的请求标头

若要允许在 CORS 请求中发送特定标头（称为 *作者请求标头* ），请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> 并指定允许的标头：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=55-60&highlight=5)]

若要允许所有作者请求标头，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=64-69&highlight=5)]

此设置会影响预检请求和 `Access-Control-Request-Headers` 标头。 有关详细信息，请参阅 [预检请求](#preflight-requests) 部分。

CORS 中间件始终允许发送四个标头， `Access-Control-Request-Headers` 而不考虑在 CorsPolicy 中配置的值。 此标头列表包括：

* `Accept`
* `Accept-Language`
* `Content-Language`
* `Origin`

例如，考虑按如下方式配置的应用：

```csharp
app.UseCors(policy => policy.WithHeaders(HeaderNames.CacheControl));
```

CORS 中间件使用以下请求标头成功响应到预检请求，因为 `Content-Language` 始终允许：

```
Access-Control-Request-Headers: Cache-Control, Content-Language
```

### <a name="set-the-exposed-response-headers"></a>设置公开的响应标头

默认情况下，浏览器不会向应用程序公开所有的响应标头。 有关详细信息，请参阅 [W3C 跨域资源共享 (术语) ：简单的响应标头](https://www.w3.org/TR/cors/#simple-response-header)。

默认情况下可用的响应标头包括：

* `Cache-Control`
* `Content-Language`
* `Content-Type`
* `Expires`
* `Last-Modified`
* `Pragma`

CORS 规范将这些标头称为 *简单的响应标头* 。 若要使其他标头可用于应用程序，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithExposedHeaders*> ：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=73-78&highlight=5)]

### <a name="credentials-in-cross-origin-requests"></a>跨域请求中的凭据

凭据需要在 CORS 请求中进行特殊处理。 默认情况下，浏览器不会使用跨域请求发送凭据。 凭据包括 cookie s 和 HTTP 身份验证方案。 若要使用跨域请求发送凭据，客户端必须设置 `XMLHttpRequest.withCredentials` 为 `true` 。

`XMLHttpRequest`直接使用：

```javascript
var xhr = new XMLHttpRequest();
xhr.open('get', 'https://www.example.com/api/test');
xhr.withCredentials = true;
```

使用 jQuery：

```javascript
$.ajax({
  type: 'get',
  url: 'https://www.example.com/api/test',
  xhrFields: {
    withCredentials: true
  }
});
```

使用 [提取 API](https://developer.mozilla.org/docs/Web/API/Fetch_API)：

```javascript
fetch('https://www.example.com/api/test', {
    credentials: 'include'
});
```

服务器必须允许凭据。 若要允许跨域凭据，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowCredentials*> ：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=82-87&highlight=5)]

HTTP 响应包含一个 `Access-Control-Allow-Credentials` 标头，通知浏览器服务器允许跨源请求的凭据。

如果浏览器发送凭据，但响应不包含有效的 `Access-Control-Allow-Credentials` 标头，则浏览器不会向应用程序公开响应，而且跨源请求会失败。

允许跨域凭据会带来安全风险。 另一个域中的网站可以代表用户将登录用户的凭据发送给该应用程序，而无需用户的知识。 <!-- TODO Review: When using `AllowCredentials`, all CORS enabled domains must be trusted.
I don't like "all CORS enabled domains must be trusted", because it implies that if you're not using  `AllowCredentials`, domains don't need to be trusted. -->

CORS 规范还指出， `"*"` 如果 `Access-Control-Allow-Credentials` 标头存在，则 (所有源) 的设置源无效。

### <a name="preflight-requests"></a>预检请求

对于某些 CORS 请求，浏览器会在发出实际请求之前发送其他请求。 此请求称为 *预检请求* 。 如果满足以下条件，浏览器可以跳过预检请求：

* 请求方法为 GET、HEAD 或 POST。
* 应用不会设置、、、或以外的请求标头 `Accept` `Accept-Language` `Content-Language` `Content-Type` `Last-Event-ID` 。
* `Content-Type`标头（如果已设置）具有以下值之一：
  * `application/x-www-form-urlencoded`
  * `multipart/form-data`
  * `text/plain`

为客户端请求设置的请求标头上的规则适用于应用通过在对象上调用来设置的标头 `setRequestHeader` `XMLHttpRequest` 。 CORS 规范调用这些标头 *作者请求标头* 。 此规则不适用于浏览器可以设置的标头，如 `User-Agent` 、 `Host` 或 `Content-Length` 。

下面是预检请求的示例：

```
OPTIONS https://myservice.azurewebsites.net/api/test HTTP/1.1
Accept: */*
Origin: https://myclient.azurewebsites.net
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: accept, x-my-custom-header
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
Host: myservice.azurewebsites.net
Content-Length: 0
```

预航班请求使用 HTTP OPTIONS 方法。 它包括两个特殊标头：

* `Access-Control-Request-Method`：将用于实际请求的 HTTP 方法。
* `Access-Control-Request-Headers`：应用在实际请求上设置的请求标头的列表。 如前文所述，这不包含浏览器设置的标头，如 `User-Agent` 。

<!-- I think this needs to be removed, was put here accidently -->

如果为 CORS 启用了适当的策略，ASP.NET Core 一般会自动响应 CORS 预检请求。 请参阅 [[HttpOptions] 属性以获取预检请求](#pro)。

CORS 预检请求可能包含一个 `Access-Control-Request-Headers` 标头，该标头向服务器指示与实际请求一起发送的标头。

若要允许特定标头，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.WithHeaders*> ：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=55-60&highlight=5)]

若要允许所有作者请求标头，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.AllowAnyHeader*> ：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=64-69&highlight=5)]

浏览器在设置方式上并不完全一致 `Access-Control-Request-Headers` 。 如果将标头设置为 (以外的任何内容 `"*"` 或使用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicy.AllowAnyHeader*>) ，则应至少包含 `Accept` 、 `Content-Type` 、和 `Origin` ，以及要支持的任何自定义标头。

下面是 (假定服务器允许) 请求的预检请求的示例响应：

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Length: 0
Access-Control-Allow-Origin: https://myclient.azurewebsites.net
Access-Control-Allow-Headers: x-my-custom-header
Access-Control-Allow-Methods: PUT
Date: Wed, 20 May 2015 06:33:22 GMT
```

响应包含一个 `Access-Control-Allow-Methods` 标头，其中列出了允许的方法，还可以选择 `Access-Control-Allow-Headers` 标头，其中列出了允许的标头。 如果预检请求成功，则浏览器发送实际请求。

如果预检请求被拒绝，应用将返回 *200 OK* 响应，但不会向后发送 CORS 标头。 因此，浏览器不会尝试跨域请求。

### <a name="set-the-preflight-expiration-time"></a>设置预检过期时间

`Access-Control-Max-Age`标头指定可缓存对预检请求的响应的时间长度。 若要设置此标头，请调用 <xref:Microsoft.AspNetCore.Cors.Infrastructure.CorsPolicyBuilder.SetPreflightMaxAge*> ：

[!code-csharp[](cors/sample/CorsExample4/Startup.cs?range=91-96&highlight=5)]

<a name="how-cors"></a>

## <a name="how-cors-works"></a>CORS 如何工作

本部分介绍 HTTP 消息级别的 [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) 请求中发生的情况。

* CORS **不** 是一种安全功能。 CORS 是一种 W3C 标准，可让服务器放宽相同的源策略。
  * 例如，恶意执行组件可能会对站点使用 [阻止跨站点脚本 (XSS) ](xref:security/cross-site-scripting) ，并向启用了 CORS 的站点执行跨站点请求来窃取信息。
* API 不能通过允许 CORS 来更安全。
  * 它由客户端 (浏览器) 来强制执行 CORS。 服务器执行请求并返回响应，这是返回错误并阻止响应的客户端。 例如，以下任何工具都将显示服务器响应：
    * [Fiddler](https://www.telerik.com/fiddler)
    * [Postman](https://www.getpostman.com/)
    * [.NET HttpClient](/dotnet/csharp/tutorials/console-webapiclient)
    * Web 浏览器，方法是在地址栏中输入 URL。
* 这是一种方法，使服务器能够允许浏览器执行跨源 [XHR](https://developer.mozilla.org/docs/Web/API/XMLHttpRequest) 或 [获取 API](https://developer.mozilla.org/docs/Web/API/Fetch_API) 请求，否则将被禁止。
  * 没有 CORS) 的浏览器 (无法进行跨域请求。 在 CORS 之前，使用 [JSONP](https://www.w3schools.com/js/js_json_jsonp.asp) 来绕过此限制。 JSONP 不使用 XHR，而是使用 `<script>` 标记接收响应。 允许跨源加载脚本。

[CORS 规范](https://www.w3.org/TR/cors/)介绍了几个新的 HTTP 标头，它们启用了跨域请求。 如果浏览器支持 CORS，则会自动为跨域请求设置这些标头。 若要启用 CORS，无需自定义 JavaScript 代码。

下面是一个跨源请求的示例。 `Origin`标头提供发出请求的站点的域。 `Origin`标头是必需的，并且必须与主机不同。

```
GET https://myservice.azurewebsites.net/api/test HTTP/1.1
Referer: https://myclient.azurewebsites.net/
Accept: */*
Accept-Language: en-US
Origin: https://myclient.azurewebsites.net
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (compatible; MSIE 10.0; Windows NT 6.2; WOW64; Trident/6.0)
Host: myservice.azurewebsites.net
```

如果服务器允许该请求，则会 `Access-Control-Allow-Origin` 在响应中设置标头。 此标头的值可以与 `Origin` 请求中的标头相匹配，也可以是通配符值 `"*"` ，这意味着允许任何源：

```
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Type: text/plain; charset=utf-8
Access-Control-Allow-Origin: https://myclient.azurewebsites.net
Date: Wed, 20 May 2015 06:27:30 GMT
Content-Length: 12

Test message
```

如果响应不包含 `Access-Control-Allow-Origin` 标头，则跨域请求会失败。 具体而言，浏览器不允许该请求。 即使服务器返回成功的响应，浏览器也不会将响应提供给客户端应用程序。

<a name="test"></a>

## <a name="test-cors"></a>测试 CORS

测试 CORS：

1. [创建 API 项目](xref:tutorials/first-web-api)。 或者，您也可以 [下载该示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/cors/sample/Cors)。
1. 使用本文档中的方法之一启用 CORS。 例如： 。

  [!code-csharp[](cors/sample/Cors/WebAPI/StartupTest.cs?name=snippet2&highlight=13-18)]

  > [!WARNING]
  > `WithOrigins("https://localhost:<port>");` 应仅用于测试示例应用程序，类似于 [下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/cors/sample/Cors)。

1.  (Razor 页面或 MVC) 创建 web 应用项目。 该示例使用 Razor 页面。 可以在与 API 项目相同的解决方案中创建 web 应用。
1. 将以下突出显示的代码添加到 *索引 cshtml* 文件中：

  [!code-cshtml[](cors/sample/Cors/ClientApp/Pages/Index2.cshtml?highlight=7-99)]

1. 在上面的代码中，将替换 `url: 'https://<web app>.azurewebsites.net/api/values/1',` 为已部署应用的 URL。
1. 部署 API 项目。 例如， [部署到 Azure](xref:host-and-deploy/azure-apps/index)。
1. Razor从桌面运行页面或 MVC 应用，并单击 " **测试** " 按钮。 使用 F12 工具查看错误消息。
1. 从中删除 localhost 源 `WithOrigins` 并部署应用。 或者，使用其他端口运行客户端应用。 例如，在 Visual Studio 中运行。
1. 与客户端应用程序进行测试。 CORS 故障返回一个错误，但错误消息不能用于 JavaScript。 使用 F12 工具中的 "控制台" 选项卡查看错误。 根据浏览器，你会在 "F12 工具" 控制台 (出现错误，如下所示) ：

   * 使用 Microsoft Edge：

     **SEC7120： [CORS] 源在 `https://localhost:44375` `https://localhost:44375` 跨域资源的访问控制允许源响应标头中找不到 `https://webapi.azurewebsites.net/api/values/1`**

   * 使用 Chrome：

     **`https://webapi.azurewebsites.net/api/values/1`CORS 策略已阻止从原始位置访问 XMLHttpRequest `https://localhost:44375` ：请求的资源上没有 "访问控制-允许" 标头。**
     
可以使用工具（如 [Fiddler](https://www.telerik.com/fiddler) 或 [Postman](https://www.getpostman.com/)）测试启用 CORS 的终结点。 使用工具时，标头指定的请求源 `Origin` 必须与接收请求的主机不同。 如果请求不是基于标头值 *跨* 域的，则 `Origin` ：

* 不需要 CORS 中间件来处理请求。
* 不会在响应中返回 CORS 标头。

## <a name="cors-in-iis"></a>IIS 中的 CORS

部署到 IIS 时，如果未将服务器配置为允许匿名访问，则必须在 Windows 身份验证之前运行 CORS。 若要支持此方案，需要为应用安装和配置 [IIS CORS 模块](https://www.iis.net/downloads/microsoft/iis-cors-module) 。

## <a name="additional-resources"></a>其他资源

* [ (CORS) 跨域资源共享 ](https://developer.mozilla.org/docs/Web/HTTP/CORS)
* [IIS CORS 模块入门](https://blogs.iis.net/iisteam/getting-started-with-the-iis-cors-module)

::: moniker-end
