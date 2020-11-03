---
title: 处理 ASP.NET Core Web API 中的错误
author: pranavkm
description: 了解 ASP.NET Core Web API 的错误处理。
monikerRange: '>= aspnetcore-2.1'
ms.author: prkrishn
ms.custom: mvc
ms.date: 07/23/2020
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
uid: web-api/handle-errors
ms.openlocfilehash: 0efcf1bbeeb65cf7f4420f8c50fb4adf7d1d016d
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052521"
---
# <a name="handle-errors-in-aspnet-core-web-apis"></a>处理 ASP.NET Core Web API 中的错误

本文介绍如何处理和自定义 ASP.NET Core Web API 的错误处理。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/handle-errors/samples) ([如何下载](xref:index#how-to-download-a-sample)) 

## <a name="developer-exception-page"></a>开发人员异常页

[开发人员异常页](xref:fundamentals/error-handling)是一种用于获取服务器错误详细堆栈跟踪的有用工具。 它使用 <xref:Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware> 来捕获 HTTP 管道中的同步和异步异常，并生成错误响应。 为了进行说明，请考虑以下控制器操作：

[!code-csharp[](handle-errors/samples/3.x/Controllers/WeatherForecastController.cs?name=snippet_GetByCity)]

运行以下 `curl` 命令以测试前面的操作：

```bash
curl -i https://localhost:5001/weatherforecast/chicago
```

::: moniker range=">= aspnetcore-3.0"

在 ASP.NET Core 3.0 及更高版本中，如果客户端不请求 HTML 格式的输出，则开发人员异常页将显示纯文本响应。 随即显示以下输出：

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/plain
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:13:16 GMT

System.ArgumentException: We don't offer a weather forecast for chicago. (Parameter 'city')
   at WebApiSample.Controllers.WeatherForecastController.Get(String city) in C:\working_folder\aspnet\AspNetCore.Docs\aspnetcore\web-api\handle-errors\samples\3.x\Controllers\WeatherForecastController.cs:line 34
   at lambda_method(Closure , Object , Object[] )
   at Microsoft.Extensions.Internal.ObjectMethodExecutor.Execute(Object target, Object[] parameters)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ActionMethodExecutor.SyncObjectResultExecutor.Execute(IActionResultTypeMapper mapper, ObjectMethodExecutor executor, Object controller, Object[] arguments)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeActionMethodAsync>g__Logged|12_1(ControllerActionInvoker invoker)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.<InvokeNextActionFilterAsync>g__Awaited|10_0(ControllerActionInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Rethrow(ActionExecutedContextSealed context)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.Next(State& next, Scope& scope, Object& state, Boolean& isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ControllerActionInvoker.InvokeInnerFilterAsync()
--- End of stack trace from previous location where exception was thrown ---
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeFilterPipelineAsync>g__Awaited|19_0(ResourceInvoker invoker, Task lastTask, State next, Scope scope, Object state, Boolean isCompleted)
   at Microsoft.AspNetCore.Mvc.Infrastructure.ResourceInvoker.<InvokeAsync>g__Logged|17_1(ResourceInvoker invoker)
   at Microsoft.AspNetCore.Routing.EndpointMiddleware.<Invoke>g__AwaitRequestTask|6_0(Endpoint endpoint, Task requestTask, ILogger logger)
   at Microsoft.AspNetCore.Authorization.AuthorizationMiddleware.Invoke(HttpContext context)
   at Microsoft.AspNetCore.Diagnostics.DeveloperExceptionPageMiddleware.Invoke(HttpContext context)

HEADERS
=======
Accept: */*
Host: localhost:44312
User-Agent: curl/7.55.1
```

要改为显示 HTML 格式的响应，请将 `Accept` HTTP 请求头设置为 `text/html` 媒体类型。 例如： 。

```bash
curl -i -H "Accept: text/html" https://localhost:5001/weatherforecast/chicago
```

请考虑以下 HTTP 响应摘录：

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

在 ASP.NET Core 2.2 及更低版本中，开发人员异常页将显示 HTML 格式的响应。 例如，请考虑以下 HTTP 响应摘录：

::: moniker-end

```console
HTTP/1.1 500 Internal Server Error
Transfer-Encoding: chunked
Content-Type: text/html; charset=utf-8
Server: Microsoft-IIS/10.0
X-Powered-By: ASP.NET
Date: Fri, 27 Sep 2019 16:55:37 GMT

<!DOCTYPE html>
<html lang="en" xmlns="http://www.w3.org/1999/xhtml">
    <head>
        <meta charset="utf-8" />
        <title>Internal Server Error</title>
        <style>
            body {
    font-family: 'Segoe UI', Tahoma, Arial, Helvetica, sans-serif;
    font-size: .813em;
    color: #222;
    background-color: #fff;
}
```

::: moniker range=">= aspnetcore-3.0"

通过 Postman 等工具进行测试时，HTML 格式的响应会很有用。 以下屏幕截图显示了 Postman 中的纯文本和 HTML 格式的响应：

![Postman 中的开发人员异常页测试](handle-errors/_static/developer-exception-page-postman.gif)

::: moniker-end

> [!WARNING]
> 仅当应用程序在开发环境中运行时才启用开发人员异常页。 否则当应用程序在生产环境中运行时，详细的异常信息会向公众泄露 有关配置环境的详细信息，请参阅 <xref:fundamentals/environments>。

## <a name="exception-handler"></a>异常处理程序

在非开发环境中，可使用[异常处理中间件](xref:fundamentals/error-handling)来生成错误负载：

1. 在 `Startup.Configure` 中，调用 <xref:Microsoft.AspNetCore.Builder.ExceptionHandlerExtensions.UseExceptionHandler%2A> 以使用中间件：

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_UseExceptionHandler&highlight=9)]

    ::: moniker-end

1. 配置控制器操作以响应 `/error` 路由：

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorController)]

    ::: moniker-end

前面的 `Error` 操作向客户端发送符合 [RFC 7807](https://tools.ietf.org/html/rfc7807) 的负载。

异常处理中间件还可以在本地开发环境中提供更详细的内容协商输出。 使用以下步骤在开发和生产环境中生成一致的负载格式：

1. 在 `Startup.Configure` 中，注册特定于环境的异常处理中间件实例：

    ::: moniker range=">= aspnetcore-3.0"

    ```csharp
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    ```csharp
    public void Configure(IApplicationBuilder app, IHostingEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseExceptionHandler("/error-local-development");
        }
        else
        {
            app.UseExceptionHandler("/error");
        }
    }
    ```

    ::: moniker-end

    在上述代码中，通过以下方法注册了中间件：

    * 开发环境中 `/error-local-development` 的路由。
    * 非开发环境中 `/error` 的路由。
    
1. 将属性路由应用于控制器操作：

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

    ::: moniker range="<= aspnetcore-2.2"

    [!code-csharp[](handle-errors/samples/2.x/2.2/Controllers/ErrorController.cs?name=snippet_ErrorControllerEnvironmentSpecific)]

    ::: moniker-end

## <a name="use-exceptions-to-modify-the-response"></a>使用异常来修改响应

可以从控制器之外修改响应的内容。 在 ASP.NET 4.x Web API 中，执行此操作的一种方法是使用 <xref:System.Web.Http.HttpResponseException> 类型。 ASP.NET Core 不包括等效类型。 可以使用以下步骤来添加对 `HttpResponseException` 的支持：

1. 创建名为 `HttpResponseException` 的已知异常类型：

    [!code-csharp[](handle-errors/samples/3.x/Exceptions/HttpResponseException.cs?name=snippet_HttpResponseException)]

1. 创建名为 `HttpResponseExceptionFilter` 的操作筛选器：

    [!code-csharp[](handle-errors/samples/3.x/Filters/HttpResponseExceptionFilter.cs?name=snippet_HttpResponseExceptionFilter)]

    在上述筛选器中，将从最大整数值减去幻数10。 如果减去此数字，将允许其他筛选器在管道的最末尾运行。

1. 在 `Startup.ConfigureServices` 中，将操作筛选器添加到筛选器集合：

    ::: moniker range=">= aspnetcore-3.0"

    [!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.2"
    
    [!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

    ::: moniker range="= aspnetcore-2.1"

    [!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_AddExceptionFilter)]

    ::: moniker-end

## <a name="validation-failure-error-response"></a>验证失败错误响应

对于 Web API 控制器，如果模型验证失败，MVC 将使用 <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> 响应类型做出响应。 MVC 使用 <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.InvalidModelStateResponseFactory> 的结果来构造验证失败的错误响应。 下面的示例使用工厂在 `Startup.ConfigureServices` 中将默认响应类型更改为 <xref:Microsoft.AspNetCore.Mvc.SerializableError>：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](handle-errors/samples/3.x/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=4-13)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](handle-errors/samples/2.x/2.2/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=5-14)]

::: moniker-end

::: moniker range="= aspnetcore-2.1"

[!code-csharp[](handle-errors/samples/2.x/2.1/Startup.cs?name=snippet_DisableProblemDetailsInvalidModelStateResponseFactory&highlight=6-15)]

::: moniker-end

## <a name="client-error-response"></a>客户端错误响应

错误结果  定义为具有 HTTP 状态代码 400 或更高的结果。 对于 Web API 控制器，MVC 会将错误结果转换为具有 <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> 的结果。

::: moniker range="= aspnetcore-2.1"

> [!IMPORTANT]
> ASP.NET Core 2.1 生成一个基本符合 RFC 7807 的问题详细信息响应。 如果需要达到百分百的符合性，请将项目升级到 ASP.NET Core 2.2 或更高版本。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

可以通过以下方式之一配置错误响应：

1. [实现 ProblemDetailsFactory](#implement-problemdetailsfactory)
1. [使用 ApiBehaviorOptions.ClientErrorMapping](#use-apibehavioroptionsclienterrormapping)

### <a name="implement-problemdetailsfactory"></a>实现 `ProblemDetailsFactory`

MVC 使用 <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ProblemDetailsFactory?displayProperty=fullName> 生成 <xref:Microsoft.AspNetCore.Mvc.ProblemDetails> 和 <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> 的所有实例。 这包括客户端错误响应、验证失败错误响应以及 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Problem%2A?displayProperty=nameWithType> 和 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.ValidationProblem%2A?displayProperty=nameWithType> 帮助程序方法。

若要自定义问题详细信息响应，请在 `Startup.ConfigureServices` 中注册 <xref:Microsoft.AspNetCore.Mvc.Infrastructure.ProblemDetailsFactory> 的自定义实现：

```csharp
public void ConfigureServices(IServiceCollection serviceCollection)
{
    services.AddControllers();
    services.AddTransient<ProblemDetailsFactory, CustomProblemDetailsFactory>();
}
```

::: moniker-end

::: moniker range="= aspnetcore-2.2"

可以按照[使用 ApiBehaviorOptions.ClientErrorMapping](#use-apibehavioroptionsclienterrormapping) 部分所述的方式配置错误响应。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

### <a name="use-apibehavioroptionsclienterrormapping"></a>使用 ApiBehaviorOptions.ClientErrorMapping

使用 <xref:Microsoft.AspNetCore.Mvc.ApiBehaviorOptions.ClientErrorMapping%2A> 属性配置 `ProblemDetails` 响应的内容。 例如，`Startup.ConfigureServices` 中的以下代码会更新 404 响应的 `type` 属性：

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](index/samples/3.x/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=8-9)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[!code-csharp[](index/samples/2.x/2.2/Startup.cs?name=snippet_ConfigureApiBehaviorOptions&highlight=9-10)]

::: moniker-end
