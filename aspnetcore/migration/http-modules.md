---
title: 将 HTTP 处理程序和模块迁移到 ASP.NET Core 中间件
author: rick-anderson
description: ''
ms.author: riande
ms.date: 12/07/2016
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
uid: migration/http-modules
ms.openlocfilehash: 9664f49bd709d2c9e46130773211c339e391d1f6
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060698"
---
# <a name="migrate-http-handlers-and-modules-to-aspnet-core-middleware"></a>将 HTTP 处理程序和模块迁移到 ASP.NET Core 中间件

本文介绍如何将现有 [的 ASP.NET HTTP 模块和处理程序从 system.webserver](/iis/configuration/system.webserver/) 迁移到 ASP.NET Core [中间件](xref:fundamentals/middleware/index)。

## <a name="modules-and-handlers-revisited"></a>模块和处理程序被

在继续 ASP.NET Core 中间件之前，让我们先回顾一下 HTTP 模块和处理程序的工作原理：

![模块处理程序](http-modules/_static/moduleshandlers.png)

**处理程序是：**

* 实现[IHttpHandler](/dotnet/api/system.web.ihttphandler)的类

* 用于处理具有给定文件名或扩展名的请求，如 *. report*

* 在 *Web.config* 中 [配置](/iis/configuration/system.webserver/handlers/)

**模块为：**

* 实现[IHttpModule](/dotnet/api/system.web.ihttpmodule)的类

* 为每个请求调用

* 能够短路 (停止进一步处理请求) 

* 可以添加到 HTTP 响应，或创建自己的响应

* 在 *Web.config* 中 [配置](/iis/configuration/system.webserver/modules/)

**模块处理传入请求的顺序由确定：**

1. <https://docs.microsoft.com/previous-versions/ms227673(v=vs.140)>，它是 ASP.NET： [BeginRequest](/dotnet/api/system.web.httpapplication.beginrequest)、 [AuthenticateRequest](/dotnet/api/system.web.httpapplication.authenticaterequest)等引发的序列事件。每个模块都可以为一个或多个事件创建处理程序。

2. 对于同一事件，在 *Web.config* 中配置它们的顺序。

除了模块外，还可以将生命周期事件的处理程序添加到 *Global.asax.cs* 文件。 这些处理程序在配置的模块中的处理程序之后运行。

## <a name="from-handlers-and-modules-to-middleware"></a>从处理程序和模块到中间件

**中间件比 HTTP 模块和处理程序更简单：**

* 除 IIS 配置) 和应用程序生命周期外，模块、处理程序、 *Global.asax.cs* 、 *Web.config* (

* 中间件已使用模块和处理程序的角色

* 中间件使用代码而不是 *Web.config* 来配置

::: moniker range=">= aspnetcore-3.0"

* 通过[管道分支](xref:fundamentals/middleware/index#branch-the-middleware-pipeline)，你可以将请求发送到特定的中间件，不仅可以基于 URL，还可以发送到请求标头、查询字符串等。

::: moniker-end
::: moniker range="< aspnetcore-3.0"

* 通过[管道分支](xref:fundamentals/middleware/index#use-run-and-map)，你可以将请求发送到特定的中间件，不仅可以基于 URL，还可以发送到请求标头、查询字符串等。

::: moniker-end

**中间件非常类似于模块：**

* 为每个请求按原则调用

* 无法将[请求传递给下一个中间件](#http-modules-shortcircuiting-middleware)来对请求进行短线路

* 能够创建自己的 HTTP 响应

**中间件和模块按不同的顺序进行处理：**

* 中间件的顺序取决于它们插入请求管道的顺序，而模块的顺序主要基于 <https://docs.microsoft.com/previous-versions/ms227673(v=vs.140)> 事件

* 响应的中间件顺序与请求的顺序相反，而对于请求和响应，模块的顺序是相同的。

* 请参阅 [使用 IApplicationBuilder 创建中间件管道](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder)

![中间件](http-modules/_static/middleware.png)

请注意，在上图中，身份验证中间件与请求短路。

## <a name="migrating-module-code-to-middleware"></a>将模块代码迁移到中间件

现有 HTTP 模块如下所示：

[!code-csharp[](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyModule.cs?highlight=6,8,24,31)]

如 [中间件](xref:fundamentals/middleware/index) 页中所示，ASP.NET Core 中间件是一个类，该类公开 `Invoke` 采用 `HttpContext` 并返回的方法 `Task` 。 新的中间件将如下所示：

<a name="http-modules-usemiddleware"></a>

[!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddleware.cs?highlight=9,13,20,24,28,30,32)]

前面的中间件模板取自 [编写中间件](xref:fundamentals/middleware/write)的部分。

*MyMiddlewareExtensions* helper 类使你可以更轻松地在类中配置中间件 `Startup` 。 `UseMyMiddleware`方法将中间件类添加到请求管道。 中间件的构造函数中插入了中间件所需的服务。

<a name="http-modules-shortcircuiting-middleware"></a>

如果用户未获得授权，则模块可能会终止请求：

[!code-csharp[](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Modules/MyTerminatingModule.cs?highlight=9,10,11,12,13&name=snippet_Terminate)]

中间件通过不调用 `Invoke` 管道中的下一个中间件来处理这种情况。 请记住，这并不完全终止请求，因为当响应通过管道返回以前的中间件时，仍然会调用以前的。

[!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyTerminatingMiddleware.cs?highlight=7,8&name=snippet_Terminate)]

当您将模块的功能迁移到新的中间件时，您可能会发现您的代码不会进行编译，因为 `HttpContext` 在 ASP.NET Core 中，类发生了重大更改。 [稍后](#migrating-to-the-new-httpcontext)，你将了解如何迁移到新的 ASP.NET Core HttpContext。

## <a name="migrating-module-insertion-into-the-request-pipeline"></a>将模块插入迁移到请求管道中

通常使用 *Web.config* 将 HTTP 模块添加到请求管道：

[!code-xml[](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32-33,36,43,50,101)]

通过在类中将 [新的中间件添加](xref:fundamentals/middleware/index#create-a-middleware-pipeline-with-iapplicationbuilder) 到请求管道来转换此项 `Startup` ：

[!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=16)]

插入新中间件的管道中的确切位置取决于它作为模块处理的事件 `BeginRequest` ， (、等 `EndRequest` ) 及其在 *Web.config* 的模块列表中的顺序。

如前所述，ASP.NET Core 中没有应用程序生命周期，中间件的处理响应顺序与模块使用的顺序不同。 这可能会使你的排序决策更具挑战性。

如果排序会成为一个问题，则可以将模块拆分为多个中间件组件，这些组件可以独立排序。

## <a name="migrating-handler-code-to-middleware"></a>将处理程序代码迁移到中间件

HTTP 处理程序如下所示：

[!code-csharp[](../migration/http-modules/sample/Asp.Net4/Asp.Net4/HttpHandlers/ReportHandler.cs?highlight=5,7,13,14,15,16)]

在 ASP.NET Core 项目中，将其转换为类似于下面的中间件：

[!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Middleware/ReportHandlerMiddleware.cs?highlight=7,9,13,20,21,22,23,40,42,44)]

此中间件与与模块对应的中间件非常类似。 唯一的区别在于，这里不会调用 `_next.Invoke(context)` 。 这样做很有意义，因为处理程序位于请求管道的末尾，因此没有要调用的下一个中间件。

## <a name="migrating-handler-insertion-into-the-request-pipeline"></a>将处理程序插入迁移到请求管道中

配置 HTTP 处理程序是在 *Web.config* 中完成的，如下所示：

[!code-xml[](../migration/http-modules/sample/Asp.Net4/Asp.Net4/Web.config?highlight=6&range=1-3,32,46-48,50,101)]

可以通过将新的处理程序中间件添加到类中的请求管道来转换此转换 `Startup` ，类似于从模块转换的中间件。 此方法的问题是，它会将所有请求发送到新的处理程序中间件。 但是，只需要具有给定扩展的请求来访问中间件。 这将为你提供与 HTTP 处理程序相同的功能。

一种解决方法是使用扩展方法分支具有给定扩展的请求的管道 `MapWhen` 。 可在 `Configure` 添加其他中间件的相同方法中执行此操作：

[!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=27-34)]

`MapWhen` 采用以下参数：

1. 一个采用的 lambda， `HttpContext` `true` 如果请求应向下分支，则返回。 这意味着，不仅可以根据请求的扩展来分支请求，还可以处理请求标头、查询字符串参数等。

2. 一个采用 `IApplicationBuilder` 并添加分支的所有中间件的 lambda。 这意味着，可以将其他中间件添加到处理程序中间件前面的分支。

将在所有请求上调用分支之前添加到管道的中间件;该分支不会对它们产生任何影响。

## <a name="loading-middleware-options-using-the-options-pattern"></a>使用 options 模式加载中间件选项

某些模块和处理程序具有存储在 *Web.config* 中的配置选项。但是，在 ASP.NET Core 新的配置模型用于代替 *Web.config* 。

新 [配置系统](xref:fundamentals/configuration/index) 提供以下选项来解决此类情况：

* 将选项直接注入中间件，如下 [一节](#loading-middleware-options-through-direct-injection)所示。

* 使用 [options 模式](xref:fundamentals/configuration/options)：

1. 创建用于保存中间件选项的类，例如：

   [!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Options)]

2. 存储选项值

   配置系统允许您将选项值存储在任何所需的位置。 但是，大多数站点 *appsettings.json* 都使用，因此我们将采取这种方法：

   [!code-json[](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,14-18)]

   *MyMiddlewareOptionsSection* 是部分名称。 它不必与 options 类的名称相同。

3. 将选项值与 options 类相关联

    Options 模式使用 ASP.NET Core 的依赖项注入框架将选项类型 (例如 `MyMiddlewareOptions`) 与 `MyMiddlewareOptions` 具有实际选项的对象相关联。

    更新你的 `Startup` 类：

   1. 如果使用的是 *appsettings.json* ，请将其添加到构造函数中的配置生成器 `Startup` ：

      [!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Ctor&highlight=5-6)]

   2. 配置 options 服务：

      [!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=4)]

   3. 将选项与 options 类相关联：

      [!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_ConfigureServices&highlight=6-8)]

4. 将选项注入中间件构造函数。 这类似于将选项注入控制器。

   [!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_MiddlewareWithParams&highlight=4,7,10,15-16)]

   将中间件添加到中的 [UseMiddleware](#http-modules-usemiddleware) 扩展方法 `IApplicationBuilder` 会处理依赖关系注入。

   这并不限于 `IOptions` 对象。 中间件所需的任何其他对象都可以通过这种方式注入。

## <a name="loading-middleware-options-through-direct-injection"></a>通过直接注入加载中间件选项

Options 模式的优点在于，它在选项值与其使用者之间产生松散耦合。 将选项类与实际选项值相关联后，任何其他类都可以通过依赖关系注入框架访问这些选项。 无需围绕选项值进行传递。

如果要使用不同的选项两次使用同一中间件，则会出现这种情况。 例如，在不同的分支中使用的授权中间件允许不同角色。 不能将两个不同的选项对象与一个 options 类相关联。

解决方法是在类中获取选项对象，并将其 `Startup` 直接传递给中间件的每个实例。

1. 将第二个键添加到 *appsettings.json*

   若要将第二组选项添加到 *appsettings.json* 文件中，请使用新密钥来唯一标识它：

   [!code-json[](http-modules/sample/Asp.Net.Core/appsettings.json?range=1,10-18&highlight=2-5)]

2. 检索选项值并将其传递给中间件。 `Use...`将中间件添加到管道的扩展方法 () 是要传入选项值的逻辑位置： 

   [!code-csharp[](http-modules/sample/Asp.Net.Core/Startup.cs?name=snippet_Configure&highlight=20-23)]

3. 启用中间件以采用 options 参数。 提供 `Use...` 扩展方法 (的重载，该重载采用 options 参数，并将其传递给 `UseMiddleware`) 。 当 `UseMiddleware` 用参数调用时，它会在实例化中间件对象时将参数传递给中间件构造函数。

   [!code-csharp[](../migration/http-modules/sample/Asp.Net.Core/Middleware/MyMiddlewareWithParams.cs?name=snippet_Extensions&highlight=9-14)]

   请注意这如何包装对象中的选项对象 `OptionsWrapper` 。 这实现 `IOptions` 了中间件构造函数的预期。

## <a name="migrating-to-the-new-httpcontext"></a>迁移到新的 HttpContext

你以前看到， `Invoke` 中间件中的方法采用类型为的参数 `HttpContext` ：

```csharp
public async Task Invoke(HttpContext context)
```

`HttpContext` 在 ASP.NET Core 中进行了重大更改。 本部分演示如何将 system.servicemodel 最 [常用的属性转换为新](/dotnet/api/system.web.httpcontext) 的 `Microsoft.AspNetCore.Http.HttpContext` 。

### <a name="httpcontext"></a>HttpContext

**HttpContext** 会转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Items)]

**唯一的请求 ID (不)**

提供每个请求的唯一 id。 在日志中包含非常有用。

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Trace)]

### <a name="httpcontextrequest"></a>Httpcontext.current 请求

**HttpMethod** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Method)]

**HttpContext** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Query)]

**Httpcontext.current** 转换为（ **RawUrl** ）：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Url)]

**IsSecureConnection** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Secure)]

**UserHostAddress** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Host)]

**Httpcontext.current Cookie 。s** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Cookies)]

**RequestContext RouteData** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Route)]

**Httpcontext.current** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Headers)]

**UserAgent** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Agent)]

**UrlReferrer** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Referrer)]

**HttpContext** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Type)]

**Httpcontext.current** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Form)]

> [!WARNING]
> 仅当 content 子类型为 *x-www-url 编码* 或 *窗体数据* 时才读取窗体值。

**InputStream** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Input)]

> [!WARNING]
> 仅在管道末尾的处理程序类型中间件中使用此代码。
>
>对于每个请求，可以读取上面所示的原始主体。 第一次读取后尝试读取正文的中间件将读取空正文。
>
>这并不适用于读取如上所示的窗体，因为这是从缓冲区中完成的。

### <a name="httpcontextresponse"></a>HttpContext 响应

**Httpcontext.current** 转换为（ **StatusDescription** ）：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Status)]

**ContentEncoding** 和 **httpcontext** 转换为以下内容：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespType)]

**Httpcontext.current** 还会转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_RespTypeOnly)]

**Httpcontext.current** 转换为：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_Output)]

**TransmitFile**

[中间件和请求功能](xref:fundamentals/request-features#middleware-and-request-features)中讨论了如何提供文件。

**Httpcontext.current 标头**

发送响应标头比较复杂，因为如果在将任何内容写入响应正文后设置这些标头，则不会发送这些标头。

解决方法是设置一个回调方法，该方法将在开始写入响应之前被调用。 最好在中间件中的方法的开头完成此操作 `Invoke` 。 这是设置响应标头的此回调方法。

下面的代码设置一个名为的回调方法 `SetHeaders` ：

```csharp
public async Task Invoke(HttpContext httpContext)
{
    // ...
    httpContext.Response.OnStarting(SetHeaders, state: httpContext);
```

`SetHeaders`回调方法将如下所示：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetHeaders)]

**HttpContext 响应。 Cookie些**

Cookie在 *设置 Cookie* 响应标头中传递到浏览器。 因此，发送需要与 cookie 用于发送响应标头的回调相同：

```csharp
public async Task Invoke(HttpContext httpContext)
{
    // ...
    httpContext.Response.OnStarting(SetCookies, state: httpContext);
    httpContext.Response.OnStarting(SetHeaders, state: httpContext);
```

`SetCookies`回调方法将如下所示：

[!code-csharp[](http-modules/sample/Asp.Net.Core/Middleware/HttpContextDemoMiddleware.cs?name=snippet_SetCookies)]

## <a name="additional-resources"></a>其他资源

* [HTTP 处理程序和 HTTP 模块概述](/iis/configuration/system.webserver/)
* [配置](xref:fundamentals/configuration/index)
* [应用程序启动](xref:fundamentals/startup)
* [中间件](xref:fundamentals/middleware/index)
