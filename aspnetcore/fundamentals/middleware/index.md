---
title: ASP.NET Core 中间件
author: rick-anderson
description: 了解 ASP.NET Core 中间件和请求管道。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/15/2020
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
uid: fundamentals/middleware/index
ms.openlocfilehash: 06f55647ba2bb41152e16bb054e098abbe157cb8
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059359"
---
# <a name="aspnet-core-middleware"></a>ASP.NET Core 中间件

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Steve Smith](https://ardalis.com/)

中间件是一种装配到应用管道以处理请求和响应的软件。 每个组件：

* 选择是否将请求传递到管道中的下一个组件。
* 可在管道中的下一个组件前后执行工作。

请求委托用于生成请求管道。 请求委托处理每个 HTTP 请求。

使用 <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A><xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 和 <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> 扩展方法来配置请求委托。 可将一个单独的请求委托并行指定为匿名方法（称为并行中间件），或在可重用的类中对其进行定义。 这些可重用的类和并行匿名方法即为中间件，也叫中间件组件。 请求管道中的每个中间件组件负责调用管道中的下一个组件，或使管道短路。 当中间件短路时，它被称为“终端中间件”，因为它阻止中间件进一步处理请求。

<xref:migration/http-modules>介绍了 ASP.NET Core 和 ASP.NET 4.x 中请求管道之间的差异，并提供了更多的中间件示例。

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>使用 IApplicationBuilder 创建中间件管道

ASP.NET Core 请求管道包含一系列请求委托，依次调用。 下图演示了这一概念。 沿黑色箭头执行。

![请求处理模式显示请求到达、通过三个中间件进行处理以及响应离开应用。 每个中间件运行其逻辑，并在 next() 语句处将请求传递到下一个中间件。 在第三个中间件处理请求之后，请求按相反顺序返回通过前两个中间件，以进行离开应用前并在其 next() 语句后的其他处理，作为对客户端的响应。](index/_static/request-delegate-pipeline.png)

每个委托均可在下一个委托前后执行操作。 应尽早在管道中调用异常处理委托，这样它们就能捕获在管道的后期阶段发生的异常。

尽可能简单的 ASP.NET Core 应用设置了处理所有请求的单个请求委托。 这种情况不包括实际请求管道。 调用单个匿名函数以响应每个 HTTP 请求。

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

用 <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> 将多个请求委托链接在一起。 `next` 参数表示管道中的下一个委托。 可通过不调用 next 参数使管道短路。 通常可在下一个委托前后执行操作，如以下示例所示：

[!code-csharp[](index/snapshot/Chain/Startup.cs?highlight=5-10)]

当委托不将请求传递给下一个委托时，它被称为“让请求管道短路”。 通常需要短路，因为这样可以避免不必要的工作。 例如，[静态文件中间件](xref:fundamentals/static-files)可以处理对静态文件的请求，并让管道的其余部分短路，从而起到终端中间件的作用。 如果中间件添加到管道中，且位于终止进一步处理的中间件前，它们仍处理 `next.Invoke` 语句后面的代码。 不过，请参阅下面有关尝试对已发送的响应执行写入操作的警告。

> [!WARNING]
> 在向客户端发送响应后，请勿调用 `next.Invoke`。 响应启动后，针对 <xref:Microsoft.AspNetCore.Http.HttpResponse> 的更改将引发异常。 例如，[设置标头和状态代码更改将引发异常](xref:performance/performance-best-practices#do-not-modify-the-status-code-or-headers-after-the-response-body-has-started)。 调用 `next` 后写入响应正文：
>
> * 可能导致违反协议。 例如，写入的长度超过规定的 `Content-Length`。
> * 可能损坏正文格式。 例如，向 CSS 文件中写入 HTML 页脚。
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted%2A> 是一个有用的提示，指示是否已发送标头或已写入正文。

<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> 委托不会收到 `next` 参数。 第一个 `Run` 委托始终为终端，用于终止管道。 `Run` 是一种约定。 某些中间件组件可能会公开在管道末尾运行的 `Run[Middleware]` 方法：

[!code-csharp[](index/snapshot/Chain/Startup.cs?highlight=12-15)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

在前面的示例中，`Run` 委托将 `"Hello from 2nd delegate."` 写入响应，然后终止管道。 如果在 `Run` 委托之后添加了另一个 `Use` 或 `Run` 委托，则不会调用该委托。

<a name="order"></a>

## <a name="middleware-order"></a>中间件顺序

下图显示了 ASP.NET Core MVC 和 Razor Pages 应用的完整请求处理管道。 你可以在典型应用中了解现有中间件的顺序，以及在哪里添加自定义中间件。 你可以完全控制如何重新排列现有中间件，或根据场景需要注入新的自定义中间件。

![ASP.NET Core 中间件管道](index/_static/middleware-pipeline.svg)

上图中的“终结点”中间件为相应的应用类型（MVC 或 Razor Pages）执行筛选器管道。

![ASP.NET Core 筛选器管道](index/_static/mvc-endpoint.svg)

向 `Startup.Configure` 方法添加中间件组件的顺序定义了针对请求调用这些组件的顺序，以及响应的相反顺序。 此顺序对于安全性、性能和功能至关重要。

下面的 `Startup.Configure` 方法按照建议的顺序增加与安全相关的中间件组件：

[!code-csharp[](index/snapshot/StartupAll3.cs?name=snippet)]

在上述代码中：

* 在使用[单个用户帐户](xref:security/authentication/identity)创建新的 Web 应用时未添加的中间件已被注释掉。
* 并非所有中间件都需要准确按照此顺序运行，但许多中间件必须遵循这个顺序。 例如：
  * `UseCors`、`UseAuthentication` 和 `UseAuthorization` 必须按照上述顺序运行。
  * 由于[此错误](https://github.com/dotnet/aspnetcore/issues/23218)，`UseCors` 当前必须在 `UseResponseCaching` 之前运行。

以下 `Startup.Configure` 方法将为常见应用方案添加中间件组件：

1. 异常/错误处理
   * 当应用在开发环境中运行时：
     * 开发人员异常页中间件 (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>) 报告应用运行时错误。
     * 数据库错误页中间件报告数据库运行时错误。
   * 当应用在生产环境中运行时：
     * 异常处理程序中间件 (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>) 捕获以下中间件中引发的异常。
     * HTTP 严格传输安全协议 (HSTS) 中间件 (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts%2A>) 添加 `Strict-Transport-Security` 标头。
1. HTTPS 重定向中间件 (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>) 将 HTTP 请求重定向到 HTTPS。
1. 静态文件中间件 (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) 返回静态文件，并简化进一步请求处理。
1. Cookie 策略中间件 (<xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy%2A>) 使应用符合欧盟一般数据保护条例 (GDPR) 规定。
1. 用于路由请求的路由中间件 (<xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A>)。
1. 身份验证中间件 (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) 尝试对用户进行身份验证，然后才会允许用户访问安全资源。
1. 用于授权用户访问安全资源的授权中间件 (<xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A>)。
1. 会话中间件 (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A>) 建立和维护会话状态。 如果应用使用会话状态，请在 Cookie 策略中间件之后和 MVC 中间件之前调用会话中间件。
1. 用于将 Razor Pages 终结点添加到请求管道的终结点路由中间件（带有 <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages%2A> 的 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints%2A>）。

<!--

FUTURE UPDATE

On the next topic overhaul/release update, add API crosslink to "Database Error Page Middleware" in Item 1 of the list ...

Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage*

... when available via the API docs.

-->

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseRouting();
    app.UseAuthentication();
    app.UseAuthorization();
    app.UseSession();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

在前面的示例代码中，每个中间件扩展方法都通过 <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName> 命名空间在 <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> 上公开。

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 是添加到管道的第一个中间件组件。 因此，异常处理程序中间件可捕获稍后调用中发生的任何异常。

尽早在管道中调用静态文件中间件，以便它可以处理请求并使其短路，而无需通过剩余组件。 静态文件中间件不提供授权检查。 可公开访问由静态文件中间件服务的任何文件，包括 wwwroot 下的文件。 若要了解如何保护静态文件，请参阅 <xref:fundamentals/static-files>。

如果静态文件中间件未处理请求，则请求将被传递给执行身份验证的身份验证中间件 (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>)。 身份验证不使未经身份验证的请求短路。 虽然身份验证中间件对请求进行身份验证，但仅在 MVC 选择特定 Razor Page 或 MVC 控制器和操作后，才发生授权（和拒绝）。

以下示例演示中间件排序，其中静态文件的请求在响应压缩中间件前由静态文件中间件进行处理。 使用此中间件顺序不压缩静态文件。 可以压缩 Razor Pages 响应。

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseResponseCompression();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

对于单页应用程序 (SPA)，SPA 中间件 <xref:Microsoft.Extensions.DependencyInjection.SpaStaticFilesExtensions.UseSpaStaticFiles%2A> 通常是中间件管道中的最后一个。 SPA 中间件处于最后的作用是：

* 允许所有其他中间件首先响应匹配的请求。
* 允许具有客户端侧路由的 SPA 针对服务器应用无法识别的所有路由运行。

若要详细了解 SPA，请参阅 [React](xref:spa/react) 和 [Angular](xref:spa/angular) 项目模板的指南。

### <a name="forwarded-headers-middleware-order"></a>转接头中间件顺序

[!INCLUDE[](~/includes/ForwardedHeaders.md)]

## <a name="branch-the-middleware-pipeline"></a>对中间件管道进行分支

<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 扩展用作约定来创建管道分支。 `Map` 基于给定请求路径的匹配项来创建请求管道分支。 如果请求路径以给定路径开头，则执行分支。

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

下表使用前面的代码显示来自 `http://localhost:1234` 的请求和响应。

| 请求             | 响应                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Hello from non-Map delegate. |
| localhost:1234/map1 | Map Test 1                   |
| localhost:1234/map2 | Map Test 2                   |
| localhost:1234/map3 | Hello from non-Map delegate. |

使用 `Map` 时，将从 `HttpRequest.Path` 中删除匹配的路径段，并针对每个请求将该路径段追加到 `HttpRequest.PathBase`。

`Map` 支持嵌套，例如：

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

此外，`Map` 还可同时匹配多个段：

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> 基于给定谓词的结果创建请求管道分支。 `Func<HttpContext, bool>` 类型的任何谓词均可用于将请求映射到管道的新分支。 在以下示例中，谓词用于检测查询字符串变量 `branch` 是否存在：

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs?highlight=14-15)]

下表使用前面的代码显示来自 `http://localhost:1234` 的请求和响应：

| 请求                       | 响应                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Hello from non-Map delegate. |
| localhost:1234/?branch=master | Branch used = master         |

<xref:Microsoft.AspNetCore.Builder.UseWhenExtensions.UseWhen%2A> 也基于给定谓词的结果创建请求管道分支。 与 `MapWhen` 不同的是，如果这个分支不发生短路或包含终端中间件，则会重新加入主管道：

[!code-csharp[](index/snapshot/Chain/StartupUseWhen.cs?highlight=25-26)]

在前面的示例中，响应 "Hello from main pipeline." 是为所有请求编写的。 如果请求中包含查询字符串变量 `branch`，则在重新加入主管道之前会记录其值。

## <a name="built-in-middleware"></a>内置中间件

ASP.NET Core 附带以下中间件组件。 “顺序”列提供备注，以说明中间件在请求处理管道中的放置，以及中间件可能会终止请求处理的条件。 如果中间件让请求处理管道短路，并阻止下游中间件进一步处理请求，它被称为“终端中间件”。 若要详细了解短路，请参阅[使用 IApplicationBuilder 创建中间件管道](#create-a-middleware-pipeline-with-iapplicationbuilder)部分。

| 中间件 | 描述 | 顺序 |
| ---------- | ----------- | ----- |
| [身份验证](xref:security/authentication/identity) | 提供身份验证支持。 | 在需要 `HttpContext.User` 之前。 OAuth 回叫的终端。 |
| [授权](xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization%2A) | 提供身份验证支持。 | 紧接在身份验证中间件之后。 |
| [Cookie 策略](xref:security/gdpr) | 跟踪用户是否同意存储个人信息，并强制实施 cookie 字段（如 `secure` 和 `SameSite`）的最低标准。 | 在发出 cookie 的中间件之前。 示例：身份验证、会话、MVC (TempData)。 |
| [CORS](xref:security/cors) | 配置跨域资源共享。 | 在使用 CORS 的组件之前。 由于[此错误](https://github.com/dotnet/aspnetcore/issues/23218)，`UseCors` 当前必须在 `UseResponseCaching` 之前运行。|
| [诊断](xref:fundamentals/error-handling) | 提供新应用的开发人员异常页、异常处理、状态代码页和默认网页的几个单独的中间件。 | 在生成错误的组件之前。 异常终端或为新应用提供默认网页的终端。 |
| [转接头](xref:host-and-deploy/proxy-load-balancer) | 将代理标头转发到当前请求。 | 在使用已更新字段的组件之前。 示例：方案、主机、客户端 IP、方法。 |
| [运行状况检查](xref:host-and-deploy/health-checks) | 检查 ASP.NET Core 应用及其依赖项的运行状况，如检查数据库可用性。 | 如果请求与运行状况检查终结点匹配，则为终端。 |
| [标头传播](xref:fundamentals/http-requests#header-propagation-middleware) | 将 HTTP 标头从传入的请求传播到传出的 HTTP 客户端请求中。 |
| [HTTP 方法重写](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | 允许传入 POST 请求重写方法。 | 在使用已更新方法的组件之前。 |
| [HTTPS 重定向](xref:security/enforcing-ssl#require-https) | 将所有 HTTP 请求重定向到 HTTPS。 | 在使用 URL 的组件之前。 |
| [HTTP 严格传输安全性 (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | 添加特殊响应标头的安全增强中间件。 | 在发送响应之前，修改请求的组件之后。 示例：转接头、URL 重写。 |
| [MVC](xref:mvc/overview) | 用 MVC/Razor Pages 处理请求。 | 如果请求与路由匹配，则为终端。 |
| [OWIN](xref:fundamentals/owin) | 与基于 OWIN 的应用、服务器和中间件进行互操作。 | 如果 OWIN 中间件处理完请求，则为终端。 |
| [响应缓存](xref:performance/caching/middleware) | 提供对缓存响应的支持。 | 在需要缓存的组件之前。 `UseCORS` 必须在 `UseResponseCaching` 之前。|
| [响应压缩](xref:performance/response-compression) | 提供对压缩响应的支持。 | 在需要压缩的组件之前。 |
| [请求本地化](xref:fundamentals/localization) | 提供本地化支持。 | 在对本地化敏感的组件之前。 |
| [终结点路由](xref:fundamentals/routing) | 定义和约束请求路由。 | 用于匹配路由的终端。 |
| [SPA](xref:Microsoft.AspNetCore.Builder.SpaApplicationBuilderExtensions.UseSpa%2A) | 通过返回单页应用程序 (SPA) 的默认页面，在中间件链中处理来自这个点的所有请求 | 在链中处于靠后位置，因此其他服务于静态文件、MVC 操作等内容的中间件占据优先位置。|
| [会话](xref:fundamentals/app-state) | 提供对管理用户会话的支持。 | 在需要会话的组件之前。 | 
| [静态文件](xref:fundamentals/static-files) | 为提供静态文件和目录浏览提供支持。 | 如果请求与文件匹配，则为终端。 |
| [URL 重写](xref:fundamentals/url-rewriting) | 提供对重写 URL 和重定向请求的支持。 | 在使用 URL 的组件之前。 |
| [WebSockets](xref:fundamentals/websockets) | 启用 WebSockets 协议。 | 在接受 WebSocket 请求所需的组件之前。 |

## <a name="additional-resources"></a>其他资源

* [生存期和注册选项](xref:fundamentals/dependency-injection#lifetime-and-registration-options)包含已设置范围、临时性和单一实例生存期服务的中间件的完整示例  。
* <xref:fundamentals/middleware/write>
* <xref:test/middleware>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Steve Smith](https://ardalis.com/)

中间件是一种装配到应用管道以处理请求和响应的软件。 每个组件：

* 选择是否将请求传递到管道中的下一个组件。
* 可在管道中的下一个组件前后执行工作。

请求委托用于生成请求管道。 请求委托处理每个 HTTP 请求。

使用 <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A><xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 和 <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> 扩展方法来配置请求委托。 可将一个单独的请求委托并行指定为匿名方法（称为并行中间件），或在可重用的类中对其进行定义。 这些可重用的类和并行匿名方法即为中间件，也叫中间件组件。 请求管道中的每个中间件组件负责调用管道中的下一个组件，或使管道短路。 当中间件短路时，它被称为“终端中间件”，因为它阻止中间件进一步处理请求。

<xref:migration/http-modules>介绍了 ASP.NET Core 和 ASP.NET 4.x 中请求管道之间的差异，并提供了更多的中间件示例。

## <a name="create-a-middleware-pipeline-with-iapplicationbuilder"></a>使用 IApplicationBuilder 创建中间件管道

ASP.NET Core 请求管道包含一系列请求委托，依次调用。 下图演示了这一概念。 沿黑色箭头执行。

![请求处理模式显示请求到达、通过三个中间件进行处理以及响应离开应用。 每个中间件运行其逻辑，并在 next() 语句处将请求传递到下一个中间件。 在第三个中间件处理请求之后，请求按相反顺序返回通过前两个中间件，以进行离开应用前并在其 next() 语句后的其他处理，作为对客户端的响应。](index/_static/request-delegate-pipeline.png)

每个委托均可在下一个委托前后执行操作。 应尽早在管道中调用异常处理委托，这样它们就能捕获在管道的后期阶段发生的异常。

尽可能简单的 ASP.NET Core 应用设置了处理所有请求的单个请求委托。 这种情况不包括实际请求管道。 调用单个匿名函数以响应每个 HTTP 请求。

[!code-csharp[](index/snapshot/Middleware/Startup.cs)]

第一个 <xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> 委托终止了管道。

用 <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A> 将多个请求委托链接在一起。 `next` 参数表示管道中的下一个委托。 可通过不调用 next 参数使管道短路。 通常可在下一个委托前后执行操作，如以下示例所示：

[!code-csharp[](index/snapshot/Chain/Startup.cs)]

当委托不将请求传递给下一个委托时，它被称为“让请求管道短路”。 通常需要短路，因为这样可以避免不必要的工作。 例如，[静态文件中间件](xref:fundamentals/static-files)可以处理对静态文件的请求，并让管道的其余部分短路，从而起到终端中间件的作用。 如果中间件添加到管道中，且位于终止进一步处理的中间件前，它们仍处理 `next.Invoke` 语句后面的代码。 不过，请参阅下面有关尝试对已发送的响应执行写入操作的警告。

> [!WARNING]
> 在向客户端发送响应后，请勿调用 `next.Invoke`。 响应启动后，针对 <xref:Microsoft.AspNetCore.Http.HttpResponse> 的更改将引发异常。 例如，设置标头和状态代码更改将引发异常。 调用 `next` 后写入响应正文：
>
> * 可能导致违反协议。 例如，写入的长度超过规定的 `Content-Length`。
> * 可能损坏正文格式。 例如，向 CSS 文件中写入 HTML 页脚。
>
> <xref:Microsoft.AspNetCore.Http.HttpResponse.HasStarted%2A> 是一个有用的提示，指示是否已发送标头或已写入正文。

<a name="order"></a>

## <a name="middleware-order"></a>中间件顺序

向 `Startup.Configure` 方法添加中间件组件的顺序定义了针对请求调用这些组件的顺序，以及响应的相反顺序。 此顺序对于安全性、性能和功能至关重要。

下面的 `Startup.Configure` 方法按照建议的顺序增加与安全相关的中间件组件：

[!code-csharp[](index/snapshot/Startup22.cs?name=snippet)]

在上述代码中：

* 在使用[单个用户帐户](xref:security/authentication/identity)创建新的 Web 应用时未添加的中间件已被注释掉。
* 并非所有中间件都需要准确按照此顺序运行，但许多中间件必须遵循这个顺序。 例如，`UseCors` 和 `UseAuthentication` 必须按照上述顺序运行。

以下 `Startup.Configure` 方法将为常见应用方案添加中间件组件：

1. 异常/错误处理
   * 当应用在开发环境中运行时：
     * 开发人员异常页中间件 (<xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>) 报告应用运行时错误。
     * 数据库错误页中间件 (`Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage`) 报告数据库运行时错误。
   * 当应用在生产环境中运行时：
     * 异常处理程序中间件 (<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>) 捕获以下中间件中引发的异常。
     * HTTP 严格传输安全协议 (HSTS) 中间件 (<xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts%2A>) 添加 `Strict-Transport-Security` 标头。
1. HTTPS 重定向中间件 (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A>) 将 HTTP 请求重定向到 HTTPS。
1. 静态文件中间件 (<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>) 返回静态文件，并简化进一步请求处理。
1. Cookie 策略中间件 (<xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy%2A>) 使应用符合欧盟一般数据保护条例 (GDPR) 规定。
1. 身份验证中间件 (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>) 尝试对用户进行身份验证，然后才会允许用户访问安全资源。
1. 会话中间件 (<xref:Microsoft.AspNetCore.Builder.SessionMiddlewareExtensions.UseSession%2A>) 建立和维护会话状态。 如果应用使用会话状态，请在 Cookie 策略中间件之后和 MVC 中间件之前调用会话中间件。
1. MVC (<xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvc%2A>) 将 MVC 添加到请求管道。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseDatabaseErrorPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();
    app.UseMvc();
}
```

在前面的示例代码中，每个中间件扩展方法都通过 <xref:Microsoft.AspNetCore.Builder?displayProperty=fullName> 命名空间在 <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder> 上公开。

<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 是添加到管道的第一个中间件组件。 因此，异常处理程序中间件可捕获稍后调用中发生的任何异常。

尽早在管道中调用静态文件中间件，以便它可以处理请求并使其短路，而无需通过剩余组件。 静态文件中间件不提供授权检查。 可公开访问由静态文件中间件服务的任何文件，包括 wwwroot 下的文件。 若要了解如何保护静态文件，请参阅 <xref:fundamentals/static-files>。

如果静态文件中间件未处理请求，则请求将被传递给执行身份验证的身份验证中间件 (<xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A>)。 身份验证不使未经身份验证的请求短路。 虽然身份验证中间件对请求进行身份验证，但仅在 MVC 选择特定 Razor Page 或 MVC 控制器和操作后，才发生授权（和拒绝）。

以下示例演示中间件排序，其中静态文件的请求在响应压缩中间件前由静态文件中间件进行处理。 使用此中间件顺序不压缩静态文件。 可以压缩来自 <xref:Microsoft.AspNetCore.Builder.MvcApplicationBuilderExtensions.UseMvcWithDefaultRoute%2A> 的 MVC 响应。

```csharp
public void Configure(IApplicationBuilder app)
{
    // Static files aren't compressed by Static File Middleware.
    app.UseStaticFiles();

    app.UseResponseCompression();

    app.UseMvcWithDefaultRoute();
}
```

## <a name="use-run-and-map"></a>Use、Run 和 Map

使用 <xref:Microsoft.AspNetCore.Builder.UseExtensions.Use%2A>、<xref:Microsoft.AspNetCore.Builder.RunExtensions.Run%2A> 和 <xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 配置 HTTP 管道。 `Use` 方法可使管道短路（即不调用 `next` 请求委托）。 `Run` 是一种约定，并且某些中间件组件可公开在管道末尾运行的 `Run[Middleware]` 方法。

<xref:Microsoft.AspNetCore.Builder.MapExtensions.Map%2A> 扩展用作约定来创建管道分支。 `Map` 基于给定请求路径的匹配项来创建请求管道分支。 如果请求路径以给定路径开头，则执行分支。

[!code-csharp[](index/snapshot/Chain/StartupMap.cs)]

下表使用前面的代码显示来自 `http://localhost:1234` 的请求和响应。

| 请求             | 响应                     |
| ------------------- | ---------------------------- |
| localhost:1234      | Hello from non-Map delegate. |
| localhost:1234/map1 | Map Test 1                   |
| localhost:1234/map2 | Map Test 2                   |
| localhost:1234/map3 | Hello from non-Map delegate. |

使用 `Map` 时，将从 `HttpRequest.Path` 中删除匹配的路径段，并针对每个请求将该路径段追加到 `HttpRequest.PathBase`。

<xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> 基于给定谓词的结果创建请求管道分支。 `Func<HttpContext, bool>` 类型的任何谓词均可用于将请求映射到管道的新分支。 在以下示例中，谓词用于检测查询字符串变量 `branch` 是否存在：

[!code-csharp[](index/snapshot/Chain/StartupMapWhen.cs)]

下表使用前面的代码显示来自 `http://localhost:1234` 的请求和响应。

| 请求                       | 响应                     |
| ----------------------------- | ---------------------------- |
| localhost:1234                | Hello from non-Map delegate. |
| localhost:1234/?branch=master | Branch used = master         |

`Map` 支持嵌套，例如：

```csharp
app.Map("/level1", level1App => {
    level1App.Map("/level2a", level2AApp => {
        // "/level1/level2a" processing
    });
    level1App.Map("/level2b", level2BApp => {
        // "/level1/level2b" processing
    });
});
```

此外，`Map` 还可同时匹配多个段：

[!code-csharp[](index/snapshot/Chain/StartupMultiSeg.cs?highlight=13)]

## <a name="built-in-middleware"></a>内置中间件

ASP.NET Core 附带以下中间件组件。 “顺序”列提供备注，以说明中间件在请求处理管道中的放置，以及中间件可能会终止请求处理的条件。 如果中间件让请求处理管道短路，并阻止下游中间件进一步处理请求，它被称为“终端中间件”。 若要详细了解短路，请参阅[使用 IApplicationBuilder 创建中间件管道](#create-a-middleware-pipeline-with-iapplicationbuilder)部分。

| 中间件 | 描述 | 顺序 |
| ---------- | ----------- | ----- |
| [身份验证](xref:security/authentication/identity) | 提供身份验证支持。 | 在需要 `HttpContext.User` 之前。 OAuth 回叫的终端。 |
| [Cookie 策略](xref:security/gdpr) | 跟踪用户是否同意存储个人信息，并强制实施 cookie 字段（如 `secure` 和 `SameSite`）的最低标准。 | 在发出 cookie 的中间件之前。 示例：身份验证、会话、MVC (TempData)。 |
| [CORS](xref:security/cors) | 配置跨域资源共享。 | 在使用 CORS 的组件之前。 |
| [诊断](xref:fundamentals/error-handling) | 提供新应用的开发人员异常页、异常处理、状态代码页和默认网页的几个单独的中间件。 | 在生成错误的组件之前。 异常终端或为新应用提供默认网页的终端。 |
| [转接头](xref:host-and-deploy/proxy-load-balancer) | 将代理标头转发到当前请求。 | 在使用已更新字段的组件之前。 示例：方案、主机、客户端 IP、方法。 |
| [运行状况检查](xref:host-and-deploy/health-checks) | 检查 ASP.NET Core 应用及其依赖项的运行状况，如检查数据库可用性。 | 如果请求与运行状况检查终结点匹配，则为终端。 |
| [HTTP 方法重写](xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions) | 允许传入 POST 请求重写方法。 | 在使用已更新方法的组件之前。 |
| [HTTPS 重定向](xref:security/enforcing-ssl#require-https) | 将所有 HTTP 请求重定向到 HTTPS。 | 在使用 URL 的组件之前。 |
| [HTTP 严格传输安全性 (HSTS)](xref:security/enforcing-ssl#http-strict-transport-security-protocol-hsts) | 添加特殊响应标头的安全增强中间件。 | 在发送响应之前，修改请求的组件之后。 示例：转接头、URL 重写。 |
| [MVC](xref:mvc/overview) | 用 MVC/Razor Pages 处理请求。 | 如果请求与路由匹配，则为终端。 |
| [OWIN](xref:fundamentals/owin) | 与基于 OWIN 的应用、服务器和中间件进行互操作。 | 如果 OWIN 中间件处理完请求，则为终端。 |
| [响应缓存](xref:performance/caching/middleware) | 提供对缓存响应的支持。 | 在需要缓存的组件之前。 |
| [响应压缩](xref:performance/response-compression) | 提供对压缩响应的支持。 | 在需要压缩的组件之前。 |
| [请求本地化](xref:fundamentals/localization) | 提供本地化支持。 | 在对本地化敏感的组件之前。 |
| [终结点路由](xref:fundamentals/routing) | 定义和约束请求路由。 | 用于匹配路由的终端。 |
| [会话](xref:fundamentals/app-state) | 提供对管理用户会话的支持。 | 在需要会话的组件之前。 |
| [静态文件](xref:fundamentals/static-files) | 为提供静态文件和目录浏览提供支持。 | 如果请求与文件匹配，则为终端。 |
| [URL 重写](xref:fundamentals/url-rewriting) | 提供对重写 URL 和重定向请求的支持。 | 在使用 URL 的组件之前。 |
| [WebSockets](xref:fundamentals/websockets) | 启用 WebSockets 协议。 | 在接受 WebSocket 请求所需的组件之前。 |

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/middleware/write>
* <xref:test/middleware>
* <xref:migration/http-modules>
* <xref:fundamentals/startup>
* <xref:fundamentals/request-features>
* <xref:fundamentals/middleware/extensibility>
* <xref:fundamentals/middleware/extensibility-third-party-container>

::: moniker-end
