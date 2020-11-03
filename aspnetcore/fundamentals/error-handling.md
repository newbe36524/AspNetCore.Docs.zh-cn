---
title: 处理 ASP.NET Core 中的错误
author: rick-anderson
description: 了解如何处理 ASP.NET Core 应用中的错误。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
no-loc:
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
uid: fundamentals/error-handling
ms.openlocfilehash: da7f50b27e447b86bd8a06851b767488d51b7050
ms.sourcegitcommit: a07f83b00db11f32313045b3492e5d1ff83c4437
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90592886"
---
# <a name="handle-errors-in-aspnet-core"></a>处理 ASP.NET Core 中的错误

::: moniker range=">= aspnetcore-5.0"

作者：[Kirk Larkin](https://twitter.com/serpent5)、[Tom Dykstra](https://github.com/tdykstra/) 和 [Steve Smith](https://ardalis.com/)

本文介绍了处理 ASP.NET Core Web 应用中常见错误的一些方法。 有关 Web API，请参阅 <xref:web-api/handle-errors>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples)。 （[下载方法](xref:index#how-to-download-a-sample)。）在测试示例应用时，F12 浏览器开发人员工具上的网络选项卡非常有用。

## <a name="developer-exception-page"></a>开发人员异常页

开发人员异常页显示请求异常的详细信息。 ASP.NET Core 模板会生成以下代码：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=3-6)]

当应用在[开发环境](xref:fundamentals/environments)中运行时，前面突出显示的代码启用开发人员异常页。

模板在中间件管道的前面部分放置 <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A>，以便它可以捕获在后面的中间件中引发的异常。

仅当应用程序在开发环境中运行时，前面的代码才启用开发人员异常页。 当应用在生产环境中运行时，不应公开显示详细的异常信息。 有关配置环境的详细信息，请参阅 <xref:fundamentals/environments>。

开发人员异常页包括关于异常和请求的以下信息：

* 堆栈跟踪
* 查询字符串参数（如果有）
* Cookie（如果有）
* 标头

## <a name="exception-handler-page"></a>异常处理程序页

若要为[生产环境](xref:fundamentals/environments)配置自定义错误处理页，请调用 <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A>。 此异常处理中间件：

* 捕获并记录异常。
* 使用指示的路径在备用管道中重新执行请求。 如果响应已启动，则不会重新执行请求。 模板生成的代码使用 `/Error` 路径重新执行请求。

在下面的示例中，<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 在非开发环境中添加异常处理中间件：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

Razor Pages 应用模板在 Pages 文件夹中提供了一个“错误”页面 (.cshtml) 和 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> 类 (`ErrorModel`) 。 对于 MVC 应用，项目模板包括 `Error` 操作方法和主控制器的错误视图。

不要使用 HTTP 方法属性（如 `HttpGet`）标记错误处理程序操作方法。 显式谓词可阻止某些请求访问操作方法。 如果未经身份验证的用户应看到错误视图，则允许匿名访问该方法。

### <a name="access-the-exception"></a>访问异常

使用 <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> 访问错误处理程序中的异常和原始请求路径。 以下代码将 `ExceptionMessage` 添加到由 ASP.NET Core 模板生成的默认 Pages/Error.cshtml.cs：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Error.cshtml.cs?name=snippet)]

> [!WARNING]
> 请勿向客户端提供敏感错误信息。 提供服务的错误是一种安全风险。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试异常，请执行以下操作：

* 将环境设置为生产环境。
* 从 Program.cs 中的 `webBuilder.UseStartup<Startup>();` 删除注释。
* 在主页上选择“触发异常”。

## <a name="exception-handler-lambda"></a>异常处理程序 lambda

[自定义异常处理程序页](#exception-handler-page)的替代方法是向 <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 提供 lambda。 使用 lambda，可以在返回响应前访问错误。

以下代码使用 lambda 处理异常：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupLambda.cs?name=snippet)]

<!-- 
In the preceding code, `await context.Response.WriteAsync(new string(' ', 512));` is added so the Internet Explorer browser displays the error message rather than an IE error message. For more information, see [this GitHub issue](https://github.com/dotnet/AspNetCore.Docs/issues/16144).
-->

> [!WARNING]
> 不要向客户端提供来自 <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> 或 <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> 的敏感错误信息。 提供服务的错误是一种安全风险。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试异常处理 lambda：

* 将环境设置为生产环境。
* 从 Program.cs 中的 `webBuilder.UseStartup<StartupLambda>();` 删除注释。
* 在主页上选择“触发异常”。

## <a name="usestatuscodepages"></a>UseStatusCodePages

默认情况下，ASP.NET Core 应用不会为 HTTP 错误状态代码（如“404 - 未找到”）提供状态代码页。 当应用遇到没有正文的 HTTP 400-499 错误条件时，它将返回状态代码和空响应正文。 若要提供状态代码页，请使用状态代码页中间件。 若要启用常见错误状态代码的默认纯文本处理程序，请在 `Startup.Configure` 方法中调用 <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A>：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupUseStatusCodePages.cs?name=snippet&highlight=13)]

<!-- Review: 
When you comment out // UseExceptionHandler("/Error");`
you get a browser dependant error, not the codepage as I expected.
call /index/2 -> return StatusCode(500); -> you get the codepage 
-->

在请求处理中间件之前调用 `UseStatusCodePages`。 例如，在静态文件中间件和端点中间件之前调用 `UseStatusCodePages`。

未使用 `UseStatusCodePages` 时，导航到没有终结点的 URL 会返回一条与浏览器相关的错误消息，指示找不到终结点。 例如，导航到 `Home/Privacy2`。 调用 `UseStatusCodePages` 时，浏览器返回：

```html
Status Code: 404; Not Found
```

`UseStatusCodePages` 通常不在生产中使用，因为它返回对用户没有用的消息。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试 `UseStatusCodePages`：

* 将环境设置为生产环境。
* 从 Program.cs 中的 `webBuilder.UseStartup<StartupUseStatusCodePages>();` 删除注释。
* 选择主页上的链接。

### <a name="usestatuscodepages-with-format-string"></a>包含格式字符串的 UseStatusCodePages

若要自定义响应内容类型和文本，请利用需要使用内容类型和格式字符串的 <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> 重载：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupFormat.cs?name=snippet&highlight=13-14)]

在前面的代码中，`{0}` 是错误代码的占位符。

具有格式字符串的 `UseStatusCodePages` 通常不在生产环境中使用，因为它返回对用户没有用的消息。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试 `UseStatusCodePages`，请从 Program.cs 中的 `webBuilder.UseStartup<StartupFormat>();` 删除注释。

### <a name="usestatuscodepages-with-lambda"></a>包含 lambda 的 UseStatusCodePages

若要指定自定义错误处理和响应写入代码，请利用需要使用 lambda 表达式的 <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> 重载：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupStatusLambda.cs?name=snippet&highlight=13-20)]

使用 lambda 的 `UseStatusCodePages` 通常不在生产中使用，因为它返回对用户没有用的消息。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试 `UseStatusCodePages`，请从 Program.cs 中的 `webBuilder.UseStartup<StartupStatusLambda>();` 删除注释。

### <a name="usestatuscodepageswithredirects"></a>UseStatusCodePagesWithRedirects

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A> 扩展方法：

* 向客户端发送“302 - 已找到”状态代码。
* 将客户端重定向到 URL 模板中提供的错误处理终结点。 错误处理终结点通常会显示错误信息并返回 HTTP 200。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCredirect.cs?name=snippet&highlight=13)]

URL 模板可能会包括状态代码的 `{0}` 占位符，如前面的代码中所示。 如果 URL 模板以波形符 `~`（代字号）开头，则 `~` 会替换为应用的 `PathBase`。 在应用中指定终结点时，请为终结点创建 MVC 视图或 Razor 页面。 有关 Razor Pages 示例，请参阅[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中的 [Pages/MyStatusCode.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages)。

使用此方法通常是当应用：

* 应将客户端重定向到不同的终结点（通常在不同的应用处理错误的情况下）。 对于 Web 应用，客户端的浏览器地址栏反映重定向终结点。
* 不应保留原始状态代码并通过初始重定向响应返回该代码。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试 `UseStatusCodePages`，请从 Program.cs 中的 `webBuilder.UseStartup<StartupSCredirect>();` 删除注释。

### <a name="usestatuscodepageswithreexecute"></a>UseStatusCodePagesWithReExecute

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A> 扩展方法：

* 向客户端返回原始状态代码。
* 通过使用备用路径重新执行请求管道，从而生成响应正文。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/StartupSCreX.cs?name=snippet&highlight=13)]

如果在应用中指定终结点，请为终结点创建 MVC 视图或 Razor 页面。 确保将 `UseStatusCodePagesWithReExecute` 放置在 `UseRouting` 之前，以便可以将请求重新路由到状态页。 有关 Razor Pages 示例，请参阅[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中的 [Pages/MyStatusCode2.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages)。

使用此方法通常是当应用应：

* 处理请求，但不重定向到不同终结点。 对于 Web 应用，客户端的浏览器地址栏反映最初请求的终结点。
* 保留原始状态代码并通过响应返回该代码。

URL 模板和查询字符串模板可能包括状态代码的占位符 `{0}`。 URL 模板必须以 `/` 开头。

<!-- Review: removing this. The sample code doesn't use @page "{code?}"
If you want that, it should be @page "{code:int?}"
but that's not required. Original text follows:

When using a placeholder in the path, confirm that the endpoint can process the path segment. For example, a Razor Page for errors should accept the optional path segment value with the `@page` directive:

```cshtml
@page "{code?}"
```
-->

错误处理终结点可以获取生成错误的原始 URL，如下面的示例所示：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/MyStatusCode2.cshtml.cs?name=snippet)]

有关 Razor Pages 示例，请参阅[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中的 [Pages/MyStatusCode2.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x/ErrorHandlingSample/Pages)。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples/5.x)中测试 `UseStatusCodePages`，请从 Program.cs 中的 `webBuilder.UseStartup<StartupSCreX>();` 删除注释。

## <a name="disable-status-code-pages"></a>禁用状态代码页

若要禁用 MVC 控制器或操作方法的状态代码页，请使用 [[SkipStatusCodePages]](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) 特性。

若要禁用 Razor Pages 处理程序方法或 MVC 控制器中的特定请求的状态代码页，请使用 <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature>：

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Pages/Privacy.cshtml.cs?name=snippet)]

## <a name="exception-handling-code"></a>异常处理代码

异常处理页中的代码可能也会引发异常。 应彻底测试生产错误页面，并格外小心，避免引发其自己的异常。
<!-- Review: original, which is not realistic 
 > It's often a good idea for production error pages to consist of purely static content.

 comments: - after you catch the exception, you need code to log the details and perhaps dynamically create a string with an error message. 
-->

### <a name="response-headers"></a>响应头

在响应头发送后：

* 应用无法更改响应的状态代码。
* 任何异常页或处理程序都无法运行。 必须完成响应或中止连接。

## <a name="server-exception-handling"></a>服务器异常处理

除了应用中的异常处理逻辑外，[HTTP 服务器实现](xref:fundamentals/servers/index)还能处理一些异常。 如果服务器在发送响应标头之前捕获到异常，服务器将发送不包含响应正文的 `500 - Internal Server Error` 响应。 如果服务器在发送响应标头后捕获到异常，服务器会关闭连接。 应用程序无法处理的请求将由服务器进行处理。 当服务器处理请求时，发生的任何异常都将由服务器的异常处理进行处理。 应用的自定义错误页面、异常处理中间件和筛选器都不会影响此行为。

## <a name="startup-exception-handling"></a>启动异常处理

应用程序启动期间发生的异常仅可在承载层进行处理。 可以将主机配置为，[捕获启动错误](xref:fundamentals/host/web-host#capture-startup-errors)和[捕获详细错误](xref:fundamentals/host/web-host#detailed-errors)。

仅当错误在主机地址/端口绑定后出现时，托管层才能显示捕获的启动错误的错误页。 如果绑定失败：

* 托管层将记录关键异常。
* dotnet 进程崩溃。
* 不会在 HTTP 服务器为 [Kestrel](xref:fundamentals/servers/kestrel) 时显示任何错误页。

在 [IIS](/iis)（或 Azure 应用服务）或 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 上运行应用时，如果无法启动进程，[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)将返回“502.5 - 进程失败”。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

## <a name="database-error-page"></a>数据库错误页

数据库开发人员页面异常筛选器 `AddDatabaseDeveloperPageExceptionFilter` 捕获可以使用 Entity Framework Core 迁移解决的与数据库相关的异常。 当这些异常出现时，便会生成 HTML 响应，其中包含用于解决问题的可能操作的详细信息。 仅在开发环境中启用此页。 在指定个人用户帐户时，ASP.NET Core Razor Pages 模板生成以下代码：

[!code-csharp[](error-handling/samples/5.x/StartupDBexFilter.cs?name=snippet&highlight=6)]

## <a name="exception-filters"></a>异常筛选器

在 MVC 应用中，可以全局配置异常筛选器，也可以为每个控制器或每个操作单独配置。 在 Razor Pages 应用中，可以全局配置异常筛选器，也可以为每个页面模型单独配置。 这些筛选器处理在执行控制器操作或其他筛选器时出现的任何未处理的异常。 有关详细信息，请参阅 <xref:mvc/controllers/filters#exception-filters>。

异常筛选器适合捕获 MVC 操作内发生的异常，但它们不如内置[异常处理中间件](https://github.com/dotnet/aspnetcore/blob/master/src/Middleware/Diagnostics/src/ExceptionHandler/ExceptionHandlerMiddleware.cs) `UseExceptionHandler` 灵活。 我们建议使用 `UseExceptionHandler`，除非你需要根据选择的 MVC 操作以不同的方式执行错误处理。

[!code-csharp[](error-handling/samples/5.x/ErrorHandlingSample/Startup.cs?name=snippet&highlight=9)]

## <a name="model-state-errors"></a>模型状态错误

若要了解如何处理模型状态错误，请参阅[模型绑定](xref:mvc/models/model-binding)和[模型验证](xref:mvc/models/validation)。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end

::: moniker range="< aspnetcore-5.0"

作者：[Tom Dykstra](https://github.com/tdykstra/) 和 [Steve Smith](https://ardalis.com/)

本文介绍了处理 ASP.NET Core Web 应用中常见错误的一些方法。 有关 Web API，请参阅 <xref:web-api/handle-errors>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples)。 （[下载方法](xref:index#how-to-download-a-sample)。）

## <a name="developer-exception-page"></a>开发人员异常页

开发人员异常页显示请求异常的详细信息。 ASP.NET Core 模板会生成以下代码：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=1-4)]

当应用在[开发环境](xref:fundamentals/environments)中运行时，前面的代码启用开发人员异常页。

模板将 <xref:Microsoft.AspNetCore.Builder.DeveloperExceptionPageExtensions.UseDeveloperExceptionPage%2A> 放在任何中间件之前，以便捕获后面的中间件中的异常。

仅当应用程序在开发环境中运行时，前面的代码才启用开发人员异常页。 当应用在生产环境中运行时，不应公开显示详细的异常信息。 有关配置环境的详细信息，请参阅 <xref:fundamentals/environments>。

开发人员异常页包括关于异常和请求的以下信息：

* 堆栈跟踪
* 查询字符串参数（如果有）
* Cookie（如果有）
* 标头

## <a name="exception-handler-page"></a>异常处理程序页

若要为生产环境配置自定义错误处理页，请使用异常处理中间件。 中间件：

* 捕获并记录异常。
* 在备用管道中为指定的页或控制器重新执行请求。 如果响应已启动，则不会重新执行请求。 模板生成的代码将请求重新执行到 `/Error`。

在下面的示例中，<xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 在非开发环境中添加异常处理中间件：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_DevPageAndHandlerPage&highlight=5-9)]

Razor Pages 应用模板在 Pages 文件夹中提供了一个“错误”页面 (.cshtml) 和 <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> 类 (`ErrorModel`) 。 对于 MVC 应用，项目模板包括错误操作方法和主控制器的错误视图。

不要使用 HTTP 方法属性（如 `HttpGet`）标记错误处理程序操作方法。 显式谓词可阻止某些请求访问方法。 如果未经身份验证的用户应看到错误视图，则允许匿名访问该方法。

### <a name="access-the-exception"></a>访问异常

使用 <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> 访问错误处理程序控制器或页中的异常和原始请求路径：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/MyFolder/Error.cshtml.cs?name=snippet_ExceptionHandlerPathFeature&3,7)]

> [!WARNING]
> 请勿向客户端提供敏感错误信息。 提供服务的错误是一种安全风险。

若要触发前面的异常处理页，请将环境设置为生产环境并强制引发异常。

## <a name="exception-handler-lambda"></a>异常处理程序 lambda

[自定义异常处理程序页](#exception-handler-page)的替代方法是向 <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 提供 lambda。 使用 lambda，可以在返回响应前访问错误。

下面的示例展示了如何使用 lambda 进行异常处理：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_HandlerPageLambda)]

在前面的代码中，添加了 `await context.Response.WriteAsync(new string(' ', 512));`，以便 Internet Explorer 浏览器显示相应的错误消息，而非显示 IE 错误消息。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/16144)。

> [!WARNING]
> 不要向客户端提供来自 <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerFeature> 或 <xref:Microsoft.AspNetCore.Diagnostics.IExceptionHandlerPathFeature> 的敏感错误信息。 提供服务的错误是一种安全风险。

若要在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples)中查看异常处理 lambda 的结果，请使用 `ProdEnvironment` 和 `ErrorHandlerLambda` 预处理器指令，并选择主页上的“触发异常”。

## <a name="usestatuscodepages"></a>UseStatusCodePages

默认情况下，ASP.NET Core 应用不会为 HTTP 状态代码（如“404 - 未找到”）提供状态代码页。 应用返回状态代码和空响应正文。 若要提供状态代码页，请使用状态代码页中间件。

此中间件是通过 [Microsoft.AspNetCore.Diagnostics](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics/) 包提供。

若要启用常见错误状态代码的默认纯文本处理程序，请在 `Startup.Configure` 方法中调用 <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A>：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePages)]

在请求处理中间件（例如，静态文件中间件和 MVC 中间件）前面调用 `UseStatusCodePages`。

未使用 `UseStatusCodePages` 时，导航到没有终结点的 URL 会返回一条与浏览器相关的错误消息，指示找不到终结点。 例如，导航到 `Home/Privacy2`。 调用 `UseStatusCodePages` 时，浏览器返回：

```
Status Code: 404; Not Found
```

## <a name="usestatuscodepages-with-format-string"></a>包含格式字符串的 UseStatusCodePages

若要自定义响应内容类型和文本，请利用需要使用内容类型和格式字符串的 <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> 重载：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesFormatString)]

## <a name="usestatuscodepages-with-lambda"></a>包含 lambda 的 UseStatusCodePages

若要指定自定义错误处理和响应写入代码，请利用需要使用 lambda 表达式的 <xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePages%2A> 重载：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesLambda)]

## <a name="usestatuscodepageswithredirects"></a>UseStatusCodePagesWithRedirects

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithRedirects%2A> 扩展方法：

* 向客户端发送“302 - 已找到”状态代码。
* 将客户端重定向到 URL 模板中的位置。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithRedirect)]

URL 模板可能会包括状态代码的 `{0}` 占位符，如上面的示例所示。 如果 URL 模板以波形符 `~`（代字号）开头，则 `~` 会替换为应用的 `PathBase`。 如果在应用中指向终结点，请为终结点创建 MVC 视图或 Razor 页面。 有关 Razor Pages 示例，请参阅[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples)中的 Pages/StatusCode.cshtml。

使用此方法通常是当应用：

* 应将客户端重定向到不同的终结点（通常在不同的应用处理错误的情况下）。 对于 Web 应用，客户端的浏览器地址栏反映重定向终结点。
* 不应保留原始状态代码并通过初始重定向响应返回该代码。

## <a name="usestatuscodepageswithreexecute"></a>UseStatusCodePagesWithReExecute

<xref:Microsoft.AspNetCore.Builder.StatusCodePagesExtensions.UseStatusCodePagesWithReExecute%2A> 扩展方法：

* 向客户端返回原始状态代码。
* 通过使用备用路径重新执行请求管道，从而生成响应正文。

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Startup.cs?name=snippet_StatusCodePagesWithReExecute)]

如果在应用中指向终结点，请为终结点创建 MVC 视图或 Razor 页面。 确保将 `UseStatusCodePagesWithReExecute` 放置在 `UseRouting` 之前，以便可以将请求重新路由到状态页。 有关 Razor Pages 示例，请参阅[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/error-handling/samples)中的 Pages/StatusCode.cshtml。

使用此方法通常是当应用应：

* 处理请求，但不重定向到不同终结点。 对于 Web 应用，客户端的浏览器地址栏反映最初请求的终结点。
* 保留原始状态代码并通过响应返回该代码。

URL 模板和查询字符串模板可能包括状态代码的占位符 (`{0}`)。 URL 模板必须以斜杠 (`/`) 开头。 若要在路径中使用占位符，请确认终结点（页或控制器）能否处理路径段。 例如，错误的 Razor 页面应通过 `@page` 指令接受可选路径段值：

```cshtml
@page "{code?}"
```

错误处理终结点可以获取生成错误的原始 URL，如下面的示例所示：

[!code-csharp[](error-handling/samples/2.x/ErrorHandlingSample/Pages/StatusCode.cshtml.cs?name=snippet_StatusCodeReExecute)]

## <a name="disable-status-code-pages"></a>禁用状态代码页

若要禁用 MVC 控制器或操作方法的状态代码页，请使用 [`[SkipStatusCodePages]`](xref:Microsoft.AspNetCore.Mvc.SkipStatusCodePagesAttribute) 特性。

若要禁用 Razor Pages 处理程序方法或 MVC 控制器中的特定请求的状态代码页，请使用 <xref:Microsoft.AspNetCore.Diagnostics.IStatusCodePagesFeature>：

```csharp
var statusCodePagesFeature = HttpContext.Features.Get<IStatusCodePagesFeature>();

if (statusCodePagesFeature != null)
{
    statusCodePagesFeature.Enabled = false;
}
```

## <a name="exception-handling-code"></a>异常处理代码

异常处理页中的代码可能会引发异常。 建议在生产错误页面中包含纯静态内容。

### <a name="response-headers"></a>响应头

在响应头发送后：

* 应用无法更改响应的状态代码。
* 任何异常页或处理程序都无法运行。 必须完成响应或中止连接。

## <a name="server-exception-handling"></a>服务器异常处理

除了应用中的异常处理逻辑外，[HTTP 服务器实现](xref:fundamentals/servers/index)还能处理一些异常。 如果服务器在发送响应标头之前捕获到异常，服务器将发送不包含响应正文的“500 - 内部服务器错误”响应。 如果服务器在发送响应标头后捕获到异常，服务器会关闭连接。 应用程序无法处理的请求将由服务器进行处理。 当服务器处理请求时，发生的任何异常都将由服务器的异常处理进行处理。 应用的自定义错误页面、异常处理中间件和筛选器都不会影响此行为。

## <a name="startup-exception-handling"></a>启动异常处理

应用程序启动期间发生的异常仅可在承载层进行处理。 可以将主机配置为，[捕获启动错误](xref:fundamentals/host/web-host#capture-startup-errors)和[捕获详细错误](xref:fundamentals/host/web-host#detailed-errors)。

仅当错误在主机地址/端口绑定后出现时，托管层才能显示捕获的启动错误的错误页。 如果绑定失败：

* 托管层将记录关键异常。
* dotnet 进程崩溃。
* 不会在 HTTP 服务器为 [Kestrel](xref:fundamentals/servers/kestrel) 时显示任何错误页。

在 [IIS](/iis)（或 Azure 应用服务）或 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 上运行应用时，如果无法启动进程，[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)将返回“502.5 - 进程失败”。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

## <a name="database-error-page"></a>数据库错误页

数据库错误页中间件捕获与数据库相关的异常，可使用实体框架迁移来解析这些异常。 当这些异常出现时，便会生成 HTML 响应，其中包含用于解决问题的可能操作的详细信息。 应仅在开发环境中启用此页。 通过向 `Startup.Configure` 添加代码来启用此页：

```csharp
if (env.IsDevelopment())
{
    app.UseDatabaseErrorPage();
}
```

<xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage%2A> 需要 [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore/) NuGet 包。

<!-- FUTURE UPDATE: On the next topic overhaul/release update, add API crosslink to this section for xref:Microsoft.AspNetCore.Builder.DatabaseErrorPageExtensions.UseDatabaseErrorPage* when available via the API docs. -->

## <a name="exception-filters"></a>异常筛选器

在 MVC 应用中，可以全局配置异常筛选器，也可以为每个控制器或每个操作单独配置。 在 Razor Pages 应用中，可以全局配置异常筛选器，也可以为每个页面模型单独配置。 这些筛选器处理在执行控制器操作或其他筛选器时出现的任何未处理的异常。 有关详细信息，请参阅 <xref:mvc/controllers/filters#exception-filters>。

> [!TIP]
> 异常筛选器适合捕获 MVC 操作内发生的异常，但它们不如异常处理中间件灵活。 建议使用中间件。 仅在需要根据选定 MVC 操作以不同方式执行错误处理时，才使用筛选器。

## <a name="model-state-errors"></a>模型状态错误

若要了解如何处理模型状态错误，请参阅[模型绑定](xref:mvc/models/model-binding)和[模型验证](xref:mvc/models/validation)。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

::: moniker-end
