---
title: ASP.NET Core 中的路由
author: rick-anderson
description: 了解 ASP.NET Core 路由如何负责匹配 HTTP 请求并将其发送到可执行终结点。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/1/2020
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
uid: fundamentals/routing
ms.openlocfilehash: e3dd7168e6974f63fa963d3732bc5df41814c70e
ms.sourcegitcommit: d5ecad1103306fac8d5468128d3e24e529f1472c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/23/2020
ms.locfileid: "92491610"
---
# <a name="routing-in-aspnet-core"></a>ASP.NET Core 中的路由

作者：[Ryan Nowak](https://github.com/rynowak)、[Kirk Larkinn](https://twitter.com/serpent5) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

路由负责匹配传入的 HTTP 请求，然后将这些请求发送到应用的可执行终结点。 [终结点](#endpoint)是应用的可执行请求处理代码单元。 终结点在应用中进行定义，并在应用启动时进行配置。 终结点匹配过程可以从请求的 URL 中提取值，并为请求处理提供这些值。 通过使用应用中的终结点信息，路由还能生成映射到终结点的 URL。

应用可以使用以下内容配置路由：

- Controllers
- Razor Pages
- SignalR
- gRPC 服务
- 启用终结点的[中间件](xref:fundamentals/middleware/index)，例如[运行状况检查](xref:host-and-deploy/health-checks)。
- 通过路由注册的委托和 Lambda。

本文档介绍 ASP.NET Core 路由的较低级别详细信息。 有关配置路由的信息，请参阅：

* 对于控制器，请参阅 <xref:mvc/controllers/routing>。
* 对于 Razor Pages 约定，请参阅 <xref:razor-pages/razor-pages-conventions>。

本文档中所述的终结点路由系统适用于 ASP.NET Core 3.0 及更高版本。 有关以前基于 <xref:Microsoft.AspNetCore.Routing.IRouter> 的路由系统信息，请使用以下方法之一选择 ASP.NET Core 2.1 版本：

* 以前版本的版本选择器。
* 选择 [ASP.NET Core 2.1 路由](?view=aspnetcore-2.1)。

[查看或下载示例代码](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples/3.x)（[如何下载](xref:index#how-to-download-a-sample)）

此文档的下载示例由特定 `Startup` 类启用。 若要运行特定的示例，请修改 Program.cs，以便调用所需的 `Startup` 类。

## <a name="routing-basics"></a>路由基础知识

所有 ASP.NET Core 模板都包括生成的代码中的路由。 路由在 `Startup.Configure` 中的[中间件](xref:fundamentals/middleware/index)管道中进行注册。

以下代码演示路由的基本示例：

[!code-csharp[](routing/samples/3.x/RoutingSample/Startup.cs?name=snippet&highlight=8,10)]

路由使用一对由 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting*> 和 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 注册的中间件：

* `UseRouting` 向中间件管道添加路由匹配。 此中间件会查看应用中定义的终结点集，并根据请求选择[最佳匹配](#urlm)。
* `UseEndpoints` 向中间件管道添加终结点执行。 它会运行与所选终结点关联的委托。

前面的示例包含使用 [MapGet](xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapGet*) 方法的单一路由到代码终结点：

* 当 HTTP `GET` 请求发送到根 URL `/` 时：
  * 将执行显示的请求委托。
  * `Hello World!` 会写入 HTTP 响应。 默认情况下，根 URL `/` 为 `https://localhost:5001/`。
* 如果请求方法不是 `GET` 或根 URL 不是 `/`，则无路由匹配，并返回 HTTP 404。

### <a name="endpoint"></a>终结点

<a name="endpoint"></a>

`MapGet` 方法用于定义终结点。 终结点可以：

* 通过匹配 URL 和 HTTP 方法来选择。
* 通过运行委托来执行。

可通过应用匹配和执行的终结点在 `UseEndpoints` 中进行配置。 例如，<xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapGet*>、<xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions.MapPost*> 和[类似的方法](xref:Microsoft.AspNetCore.Builder.EndpointRouteBuilderExtensions)将请求委托连接到路由系统。
其他方法可用于将 ASP.NET Core 框架功能连接到路由系统：
- [用于 Razor Pages 的 MapRazorPages](xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapRazorPages*)
- [用于控制器的 MapControllers](xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllers*)
- [用于 SignalR 的 MapHub\<THub>](xref:Microsoft.AspNetCore.SignalR.HubRouteBuilder.MapHub*) 
- [用于 gRPC 的 MapGrpcService\<TService>](xref:grpc/aspnetcore)

下面的示例演示如何使用更复杂的路由模板进行路由：

[!code-csharp[](routing/samples/3.x/RoutingSample/RouteTemplateStartup.cs?name=snippet)]

`/hello/{name:alpha}` 字符串是一个路由模板。 用于配置终结点的匹配方式。 在这种情况下，模板将匹配：

* 类似 `/hello/Ryan` 的 URL
* 以 `/hello/` 开头、后跟一系列字母字符的任何 URL 路径。  `:alpha` 应用仅匹配字母字符的路由约束。 [路由约束](#route-constraint-reference)将在本文档的后面详细介绍。

URL 路径的第二段 `{name:alpha}`：

* 绑定到 `name` 参数。
* 捕获并存储在 [HttpRequest. RouteValues](xref:Microsoft.AspNetCore.Http.HttpRequest.RouteValues*) 中。

本文档中所述的终结点路由系统是新版本 ASP.NET Core 3.0。 但是，所有版本的 ASP.NET Core 都支持相同的路由模板功能和路由约束。

下面的示例演示如何通过[运行状况检查](xref:host-and-deploy/health-checks)和授权进行路由：

[!code-csharp[](routing/samples/3.x/RoutingSample/AuthorizationStartup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

前面的示例展示了如何：

* 将授权中间件与路由一起使用。
* 将终结点用于配置授权行为。

<xref:Microsoft.AspNetCore.Builder.HealthCheckEndpointRouteBuilderExtensions.MapHealthChecks*> 调用添加运行状况检查终结点。 将 <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*> 链接到此调用会将授权策略附加到该终结点。

调用 <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> 和 <xref:Microsoft.AspNetCore.Builder.AuthorizationAppBuilderExtensions.UseAuthorization*> 会添加身份验证和授权中间件。 这些中间件位于 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting*> 和 `UseEndpoints` 之间，因此它们可以：

* 查看 `UseRouting` 选择的终结点。
* 将 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 发送到终结点之前应用授权策略。

<a name="metadata"></a>

### <a name="endpoint-metadata"></a>终结点元数据

前面的示例中有两个终结点，但只有运行状况检查终结点附加了授权策略。 如果请求与运行状况检查终结点 `/healthz` 匹配，则执行授权检查。 这表明，终结点可以附加额外的数据。 此额外数据称为终结点元数据：

* 可以通过路由感知中间件来处理元数据。
* 元数据可以是任意的 .NET 类型。

## <a name="routing-concepts"></a>路由概念

路由系统通过添加功能强大的终结点概念，构建在中间件管道之上。 终结点代表应用的功能单元，在路由、授权和任意数量的 ASP.NET Core 系统方面彼此不同。

<a name="endpoint"></a>

### <a name="aspnet-core-endpoint-definition"></a>ASP.NET Core 终结点定义

ASP.NET Core 终结点是：

* 可执行：具有 <xref:Microsoft.AspNetCore.Http.Endpoint.RequestDelegate>。
* 可扩展：具有[元数据](xref:Microsoft.AspNetCore.Http.Endpoint.Metadata*)集合。
* Selectable:可选择性包含[路由信息](xref:Microsoft.AspNetCore.Routing.RouteEndpoint.RoutePattern*)。
* 可枚举：可通过从 [DI](xref:fundamentals/dependency-injection) 中检索 <xref:Microsoft.AspNetCore.Routing.EndpointDataSource> 来列出终结点集合。

以下代码显示了如何检索和检查与当前请求匹配的终结点：

[!code-csharp[](routing/samples/3.x/RoutingSample/EndpointInspectorStartup.cs?name=snippet)]

如果选择了终结点，可从 `HttpContext` 中进行检索。 可以检查其属性。 终结点对象是不可变的，并且在创建后无法修改。 最常见的终结点类型是 <xref:Microsoft.AspNetCore.Routing.RouteEndpoint>。 `RouteEndpoint` 包括允许自己被路由系统选择的信息。

在前面的代码中，[app.Use](xref:Microsoft.AspNetCore.Builder.UseExtensions.Use*) 配置了一个内联[中间件](xref:fundamentals/middleware/index)。

<a name="mt"></a>

下面的代码显示，根据管道中调用 `app.Use` 的位置，可能不存在终结点：

[!code-csharp[](routing/samples/3.x/RoutingSample/MiddlewareFlowStartup.cs?name=snippet)]

前面的示例添加了 `Console.WriteLine` 语句，这些语句显示是否已选择终结点。 为清楚起见，该示例将显示名称分配给提供的 `/` 终结点。

使用 `/` URL 运行此代码将显示：

```txt
1. Endpoint: (null)
2. Endpoint: Hello
3. Endpoint: Hello
```

使用任何其他 URL 运行此代码将显示：

```txt
1. Endpoint: (null)
2. Endpoint: (null)
4. Endpoint: (null)
```

此输出说明：

* 调用 `UseRouting` 之前，终结点始终为 null。
* 如果找到匹配项，则 `UseRouting` 和 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 之间的终结点为非 null。
* 如果找到匹配项，则 `UseEndpoints` 中间件即为终端。 稍后会在本文档中定义[终端中间件](#tm)。
* 仅当找不到匹配项时才执行 `UseEndpoints` 后的中间件。

`UseRouting` 中间件使用 [SetEndpoint](xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.SetEndpoint*) 方法将终结点附加到当前上下文。 可以将 `UseRouting` 中间件替换为自定义逻辑，同时仍可获得使用终结点的益处。 终结点是中间件等低级别基元，不与路由实现耦合。 大多数应用都不需要将 `UseRouting` 替换为自定义逻辑。

`UseEndpoints` 中间件旨在与 `UseRouting` 中间件配合使用。 执行终结点的核心逻辑并不复杂。 使用 <xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint*> 检索终结点，然后调用其 <xref:Microsoft.AspNetCore.Http.Endpoint.RequestDelegate> 属性。

下面的代码演示中间件如何影响或响应路由：

[!code-csharp[](routing/samples/3.x/RoutingSample/IntegratedMiddlewareStartup.cs?name=snippet)]

前面的示例演示两个重要概念：

* 中间件可以在 `UseRouting` 之前运行，以修改路由操作的数据。
    * 通常，在路由之前出现的中间件会修改请求的某些属性，如 <xref:Microsoft.AspNetCore.Builder.RewriteBuilderExtensions.UseRewriter*>、<xref:Microsoft.AspNetCore.Builder.HttpMethodOverrideExtensions.UseHttpMethodOverride*> 或 <xref:Microsoft.AspNetCore.Builder.UsePathBaseExtensions.UsePathBase*>。
* 中间件可以在 `UseRouting` 和 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 之间运行，以便在执行终结点前处理路由结果。
    * 在 `UseRouting` 和 `UseEndpoints` 之间运行的中间件：
      * 通常会检查元数据以了解终结点。
      * 通常会根据 `UseAuthorization` 和 `UseCors` 做出安全决策。
    * 中间件和元数据的组合允许按终结点配置策略。

前面的代码显示了支持按终结点策略的自定义中间件示例。 中间件将访问敏感数据的审核日志写入控制台。 可以将中间件配置为审核具有 `AuditPolicyAttribute` 元数据的终结点。 此示例演示选择加入模式，其中仅审核标记为敏感的终结点。 例如，可以反向定义此逻辑，从而审核未标记为安全的所有内容。 终结点元数据系统非常灵活。 此逻辑可以以任何适合用例的方法进行设计。

前面的示例代码旨在演示终结点的基本概念。 该示例不应在生产环境中使用。 审核日志中间件的更完整版本如下：

* 记录到文件或数据库。
* 包括详细信息，如用户、IP 地址、敏感终结点的名称等。

审核策略元数据 `AuditPolicyAttribute` 定义为一个 `Attribute`，便于和基于类的框架（如控制器和 SignalR）结合使用。 使用路由到代码时：

* 元数据附有生成器 API。
* 基于类的框架在创建终结点时，包含了相应方法和类的所有特性。

对于元数据类型，最佳做法是将它们定义为接口或特性。 接口和特性允许代码重用。 元数据系统非常灵活，无任何限制。

<a name="tm"></a>

### <a name="comparing-a-terminal-middleware-and-routing"></a>比较终端中间件和路由

下面的代码示例对比使用中间件和使用路由：

[!code-csharp[](routing/samples/3.x/RoutingSample/TerminalMiddlewareStartup.cs?name=snippet)]

使用 `Approach 1:` 显示的中间件样式是终端中间件。 之所以称之为终端中间件，是因为它执行匹配的操作：

* 前面示例中的匹配操作是用于中间件的 `Path == "/"` 和用于路由的 `Path == "/Movie"`。
* 如果匹配成功，它将执行一些功能并返回，而不是调用 `next` 中间件。

之所以称之为终端中间件，是因为它会终止搜索，执行一些功能，然后返回。

比较终端中间件和路由：
* 这两种方法都允许终止处理管道：
    * 中间件通过返回而不是调用 `next` 来终止管道。
    * 终结点始终是终端。
* 终端中间件允许在管道中的任意位置放置中间件：
    * 终结点在 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 位置执行。
* 终端中间件允许任意代码确定中间件匹配的时间：
    * 自定义路由匹配代码可能比较复杂，且难以正确编写。
    * 路由为典型应用提供了简单的解决方案。 大多数应用不需要自定义路由匹配代码。
* 带有中间件的终结点接口，如 `UseAuthorization` 和 `UseCors`。
    * 通过 `UseAuthorization` 或 `UseCors` 使用终端中间件需要与授权系统进行手动交互。

[终结点](#endpoint)定义以下两者：

* 用于处理请求的委托。
* 任意元数据的集合。 元数据用于实现横切关注点，该实现基于附加到每个终结点的策略和配置。

终端中间件可以是一种有效的工具，但可能需要：

* 大量的编码和测试。
* 手动与其他系统集成，以实现所需的灵活性级别。

请考虑在写入终端中间件之前与路由集成。

与 [Map](xref:fundamentals/middleware/index#branch-the-middleware-pipeline) 或 <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen*> 相集成的现有终端中间件通常会转换为路由感知终结点。 [MapHealthChecks](https://github.com/aspnet/AspNetCore/blob/master/src/Middleware/HealthChecks/src/Builder/HealthCheckEndpointRouteBuilderExtensions.cs#L16) 演示了路由器软件的模式：
* 在 <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder> 上编写扩展方法。
* 使用 <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder.CreateApplicationBuilder*> 创建嵌套中间件管道。
* 将中间件附加到新管道。 在本例中为 <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>。
* <xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.Build*> 将中间件管道附加到 <xref:Microsoft.AspNetCore.Http.RequestDelegate>。
* 调用 `Map` 并提供新的中间件管道。
* 从扩展方法返回由 `Map` 提供的生成器对象。

下面的代码演示了 [MapHealthChecks](xref:host-and-deploy/health-checks) 的使用方法：

[!code-csharp[](routing/samples/3.x/RoutingSample/AuthorizationStartup.cs?name=snippet)]

前面的示例说明了为什么返回生成器对象很重要。 返回生成器对象后，应用开发者可以配置策略，如终结点的授权。 在此示例中，运行状况检查中间件不与授权系统直接集成。

元数据系统的创建目的是为了响应扩展性创建者使用终端中间件时遇到的问题。 对于每个中间件，实现自己与授权系统的集成都会出现问题。

<a name="urlm"></a>

### <a name="url-matching"></a>URL 匹配

* 是路由将传入请求匹配到[终结点](#endpoint)的过程。
* 基于 URL 路径中的数据和标头。
* 可进行扩展，以考虑请求中的任何数据。

当路由中间件执行时，它会设置 `Endpoint`，并将值从当前请求路由到 <xref:Microsoft.AspNetCore.Http.HttpContext> 上的[请求功能](xref:fundamentals/request-features)：

* 调用 [GetEndpoint](<xref:Microsoft.AspNetCore.Http.EndpointHttpContextExtensions.GetEndpoint*>) 获取终结点。
* `HttpRequest.RouteValues` 将获取路由值的集合。

在路由中间件之后运行的[中间件](xref:fundamentals/middleware/index)可以检查终结点并采取措施。 例如，授权中间件可以在终结点的元数据集合中询问授权策略。 请求处理管道中的所有中间件执行后，将调用所选终结点的委托。

终结点路由中的路由系统负责所有的调度决策。 中间件基于所选终结点应用策略，因此重要的是：

* 任何可能影响发送或安全策略应用的决定都应在路由系统中做出。

> [!WARNING]
> 对于后向兼容性，执行 Controller 或 Razor Pages 终结点委托时，应根据迄今执行的请求处理将 [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) 的属性设为适当的值。
>
> 在未来的版本中，会将 `RouteContext` 类型标记为已过时：
>
> * 将 `RouteData.Values` 迁移到 `HttpRequest.RouteValues`。
> * 迁移 `RouteData.DataTokens` 以从终结点元数据检索 [IDataTokensMetadata](xref:Microsoft.AspNetCore.Routing.IDataTokensMetadata)。

URL 匹配在可配置的阶段集中运行。 在每个阶段中，输出为一组匹配项。 下一阶段可以进一步缩小这一组匹配项。 路由实现不保证匹配终结点的处理顺序。 所有可能的匹配项一次性处理。 URL 匹配阶段按以下顺序出现。 ASP.NET Core：

1. 针对终结点集及其路由模板处理 URL 路径，收集所有匹配项。
1. 采用前面的列表并删除在应用路由约束时失败的匹配项。
1. 采用前面的列表并删除 [MatcherPolicy](xref:Microsoft.AspNetCore.Routing.MatcherPolicy) 实例集失败的匹配项。
1. 使用 [EndpointSelector](xref:Microsoft.AspNetCore.Routing.Matching.EndpointSelector) 从前面的列表中做出最终决定。

根据以下内容设置终结点列表的优先级：

* [RouteEndpoint.Order](xref:Microsoft.AspNetCore.Routing.RouteEndpoint.Order*)
* [路由模板优先顺序](#rtp)

在每个阶段中处理所有匹配的终结点，直到达到 <xref:Microsoft.AspNetCore.Routing.Matching.EndpointSelector>。 `EndpointSelector` 是最后一个阶段。 它从匹配项中选择最高优先级终结点作为最佳匹配项。 如果存在具有与最佳匹配相同优先级的其他匹配项，则会引发不明确的匹配异常。

路由优先顺序基于更具体的路由模板（优先级更高）进行计算。 例如，考虑模板 `/hello` 和 `/{message}`：

* 两者都匹配 URL 路径 `/hello`。
* `/hello` 更具体，因此优先级更高。

通常，路由优先顺序非常适合为实践操作中使用的各种 URL 方案选择最佳匹配项。 仅在必要时才使用 <xref:Microsoft.AspNetCore.Routing.RouteEndpoint.Order> 来避免多义性。

由于路由提供的扩展性种类，路由系统无法提前计算不明确的路由。 假设有一个示例，例如路由模板 `/{message:alpha}` 和 `/{message:int}`：

* `alpha` 约束仅匹配字母数字字符。
* `int` 约束仅匹配数字。
* 这些模板具有相同的路由优先顺序，但没有两者均匹配的单一 URL。
* 如果路由系统在启动时报告了多义性错误，则会阻止此有效用例。

> [!WARNING]
>
> <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 内的操作顺序并不会影响路由行为，但有一个例外。 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*> 和 <xref:Microsoft.AspNetCore.Builder.MvcAreaRouteBuilderExtensions.MapAreaRoute*> 会根据调用顺序，自动将顺序值分配给其终结点。 这会模拟控制器的长时间行为，而无需路由系统提供与早期路由实现相同的保证。
>
> 在路由的早期实现中，可以实现具有依赖于路由处理顺序的路由扩展性。 ASP.NET Core 3.0 及更高版本中的终结点路由：
> 
> * 没有路由的概念。
> * 不提供顺序保证。 同时处理所有终结点。

<a name="rtp"></a>

### <a name="route-template-precedence-and-endpoint-selection-order"></a>路由模板优先顺序和终结点选择顺序

[路由模板优先顺序](https://github.com/dotnet/aspnetcore/blob/master/src/Http/Routing/src/Template/RoutePrecedence.cs#L16)是一种系统，该系统根据每个路由模板的具体程度为其分配值。 路由模板优先顺序：

* 无需在常见情况下调整终结点的顺序。
* 尝试匹配路由行为的常识性预期。

例如，考虑模板 `/Products/List` 和 `/Products/{id}`。 我们可合理地假设，对于 URL 路径 `/Products/List`，`/Products/List` 匹配项比 `/Products/{id}` 更好。 这种假设合理是因为文本段 `/List` 比参数段 `/{id}` 具有更好的优先顺序。

优先顺序工作原理的详细信息与路由模板的定义方式相耦合：

* 具有更多段的模板则更具体。
* 带有文本的段比参数段更具体。
* 具有约束的参数段比没有约束的参数段更具体。
* 复杂段与具有约束的参数段同样具体。
* catch-all 参数是最不具体的参数。 有关 catch-all 路由的重要信息，请参阅 [路由模板参考](#rtr) 中的“catch-all”。

有关确切值的参考，请参阅 [GitHub 上的源代码](https://github.com/dotnet/aspnetcore/blob/master/src/Http/Routing/src/Template/RoutePrecedence.cs#L189)。

<a name="lg"></a>

### <a name="url-generation-concepts"></a>URL 生成概念

URL 生成：

* 是指路由基于一系列路由值创建 URL 路径的过程。
* 允许终结点与访问它们的 URL 之间存在逻辑分隔。

终结点路由包含 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API。 `LinkGenerator` 是 [DI](xref:fundamentals/dependency-injection) 中可用的单一实例服务。 `LinkGenerator` API 可在执行请求的上下文之外使用。 [Mvc.IUrlHelper](xref:Microsoft.AspNetCore.Mvc.IUrlHelper) 和依赖 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> 的方案（如[标记帮助程序](xref:mvc/views/tag-helpers/intro)、HTML 帮助程序和[操作结果](xref:mvc/controllers/actions)）在内部使用 `LinkGenerator` API 提供链接生成功能。

链接生成器基于“地址”和“地址方案”概念 。 地址方案是确定哪些终结点用于链接生成的方式。 例如，许多用户熟悉的来自控制器或 Razor Pages 的路由名称和路由值方案都是作为地址方案实现的。

链接生成器可以通过以下扩展方法链接到控制器或 Razor Pages：

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

这些方法的重载接受包含 `HttpContext` 的参数。 这些方法在功能上等同于 [Url.Action](xref:System.Web.Mvc.UrlHelper.Action*) 和 [Url.Page](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Page*)，但提供了更大的灵活性和更多选项。

`GetPath*` 方法与 `Url.Action` 和 `Url.Page` 最相似，因为它们生成包含绝对路径的 URI。 `GetUri*` 方法始终生成包含方案和主机的绝对 URI。 接受 `HttpContext` 的方法在执行请求的上下文中生成 URI。 除非重写，否则将使用来自执行请求的[环境](#ambient)路由值、URL 基路径、方案和主机。

使用地址调用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator>。 生成 URI 的过程分两步进行：

1. 将地址绑定到与地址匹配的终结点列表。
1. 计算每个终结点的 <xref:Microsoft.AspNetCore.Routing.RouteEndpoint.RoutePattern>，直到找到与提供的值匹配的路由模式。 输出结果会与提供给链接生成器的其他 URI 部分进行组合并返回。

对任何类型的地址，<xref:Microsoft.AspNetCore.Routing.LinkGenerator> 提供的方法均支持标准链接生成功能。 使用链接生成器的最简便方法是通过扩展方法对特定地址类型执行操作：

| 扩展方法 | 描述 |
| ---------------- | ----------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | 根据提供的值生成具有绝对路径的 URI。 |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | 根据提供的值生成绝对 URI。             |

> [!WARNING]
> 请注意有关调用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> 方法的下列含义：
>
> * 对于不验证传入请求的 `Host` 标头的应用配置，请谨慎使用 `GetUri*` 扩展方法。 如果未验证传入请求的 `Host` 标头，则可能以视图或页面中 URI 的形式将不受信任的请求输入发送回客户端。 建议所有生产应用都将其服务器配置为针对已知有效值验证 `Host` 标头。
>
> * 在中间件中将 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> 与 `Map` 或 `MapWhen` 结合使用时，请小心谨慎。 `Map*` 会更改执行请求的基路径，这会影响链接生成的输出。 所有 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API 都允许指定基路径。 指定一个空的基路径来撤消 `Map*` 对链接生成的影响。

### <a name="middleware-example"></a>中间件示例

在以下示例中，中间件使用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API 创建列出存储产品的操作方法的链接。 应用中的任何类都可通过将链接生成器注入类并调用 `GenerateLink` 来使用链接生成器：

[!code-csharp[](routing/samples/3.x/RoutingSample/Middleware/ProductsLinkMiddleware.cs?name=snippet)]

<a name="rtr"></a>

## <a name="route-template-reference"></a>路由模板参考

如果路由找到匹配项，`{}` 内的令牌定义绑定的路由参数。 可在路由段中定义多个路由参数，但必须用文本值隔开这些路由参数。 例如，`{controller=Home}{action=Index}` 不是有效的路由，因为 `{controller}` 和 `{action}` 之间没有文本值。  路由参数必须具有名称，且可能指定了其他特性。

路由参数以外的文本（例如 `{id}`）和路径分隔符 `/` 必须匹配 URL 中的文本。 文本匹配区分大小写，并且基于 URL 路径已解码的表示形式。 要匹配文字路由参数分隔符（`{` 或 `}`），请通过重复该字符来转义分隔符。 例如 `{{` 或 `}}`。

星号 `*` 或双星号 `**`：

* 可用作路由参数的前缀，以绑定到 URI 的其余部分。
* 称为 catch-all 参数。 例如，`blog/{**slug}`：
  * 匹配以 `/blog` 开头并在其后面包含任何值的任何 URI。
  * `/blog` 后的值分配给 [slug](https://developer.mozilla.org/docs/Glossary/Slug) 路由值。

[!INCLUDE[](~/includes/catchall.md)]

全方位参数还可以匹配空字符串。

使用路由生成 URL（包括路径分隔符 `/`）时，catch-all 参数会转义相应的字符。 例如，路由值为 `{ path = "my/path" }` 的路由 `foo/{*path}` 生成 `foo/my%2Fpath`。 请注意转义的正斜杠。 要往返路径分隔符，请使用 `**` 路由参数前缀。 `{ path = "my/path" }` 的路由 `foo/{**path}` 生成 `foo/my/path`。

尝试捕获具有可选文件扩展名的文件名的 URL 模式还有其他注意事项。 例如，考虑模板 `files/{filename}.{ext?}`。 当 `filename` 和 `ext` 的值都存在时，将填充这两个值。 如果 URL 中仅存在 `filename` 的值，则路由匹配，因为尾随 `.` 是可选的。 以下 URL 与此路由相匹配：

* `/files/myFile.txt`
* `/files/myFile`

路由参数可能具有指定的默认值，方法是在参数名称后使用等号 (`=`) 隔开以指定默认值。 例如，`{controller=Home}` 将 `Home` 定义为 `controller` 的默认值。 如果参数的 URL 中不存在任何值，则使用默认值。 通过在参数名称的末尾附加问号 (`?`) 可使路由参数成为可选项。 例如 `id?`。 可选值和默认路由参数之间的差异是：

* 具有默认值的路由参数始终生成一个值。
* 仅当请求 URL 提供值时，可选参数才具有值。

路由参数可能具有必须与从 URL 中绑定的路由值匹配的约束。 在路由参数后面添加一个 `:` 和约束名称可指定路由参数上的内联约束。 如果约束需要参数，将以在约束名称后括在括号 `(...)` 中的形式提供。 通过追加另一个 `:` 和约束名称，可指定多个内联约束。

约束名称和参数将传递给 <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> 服务，以创建 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 的实例，用于处理 URL。 例如，路由模板 `blog/{article:minlength(10)}` 使用参数 `10` 指定 `minlength` 约束。 有关路由约束详情以及框架提供的约束列表，请参阅[路由约束引用](#route-constraint-reference)部分。

路由参数还可以具有参数转换器。 参数转换器在生成链接以及将操作和页面匹配到 URI 时转换参数的值。 与约束类似，可在路由参数名称后面添加 `:` 和转换器名称，将参数变换器内联添加到路径参数。 例如，路由模板 `blog/{article:slugify}` 指定 `slugify` 转换器。 有关参数转换的详细信息，请参阅[参数转换器参考](#parameter-transformer-reference)部分。

下表演示了示例路由模板及其行为：

| 路由模板                           | 示例匹配 URI    | 请求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 仅匹配单个路径 `/hello`。                                     |
| `{Page=Home}`                            | `/`                     | 匹配并将 `Page` 设置为 `Home`。                                         |
| `{Page=Home}`                            | `/Contact`              | 匹配并将 `Page` 设置为 `Contact`。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | 映射到 `Products` 控制器和 `List` 操作。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | 映射到 `Products` 控制器和 `Details` 操作，并将 `id` 设置为 123。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | 映射到 `Home` 控制器和 `Index` 方法。 `id` 将被忽略。        |
| `{controller=Home}/{action=Index}/{id?}` | `/Products`         | 映射到 `Products` 控制器和 `Index` 方法。 `id` 将被忽略。        |

使用模板通常是进行路由最简单的方法。 还可在路由模板外指定约束和默认值。

### <a name="complex-segments"></a>复杂段

复杂段通过[非贪婪](#greedy)的方式从右到左匹配文字进行处理。 例如，`[Route("/a{b}c{d}")]` 是一个复杂段。
复杂段以一种特定的方式工作，必须理解这种方式才能成功地使用它们。 本部分中的示例演示了为什么复杂段只有在分隔符文本没有出现在参数值中时才真正有效。 对于更复杂的情况，需要使用 [regex](/dotnet/standard/base-types/regular-expressions)，然后手动提取值。

[!INCLUDE[](~/includes/regex.md)]

这是使用模板 `/a{b}c{d}` 和 URL 路径 `/abcd` 执行路由的步骤摘要。 `|` 有助于可视化算法的工作方式：

* 从右到左的第一个文本是 `c`。 因此从右侧搜索 `/abcd`，并查找 `/ab|c|d`。
* 右侧的所有内容 (`d`) 现在都与路由参数 `{d}` 相匹配。
* 从右到左的下一个文本是 `a`。 从我们停止的地方开始搜索 `/ab|c|d`，然后 `/|a|b|c|d` 找到 `a`。
* 右侧的值 (`b`) 现在与路由参数 `{b}` 相匹配。
* 没有剩余的文本，并且没有剩余的路由模板，因此这是一个匹配项。

下面是使用相同模板 `/a{b}c{d}` 和 URL 路径 `/aabcd` 的负面案例示例。 `|` 有助于可视化算法的工作方式。 该案例不是匹配项，可由相同算法解释：
* 从右到左的第一个文本是 `c`。 因此从右侧搜索 `/aabcd`，并查找 `/aab|c|d`。
* 右侧的所有内容 (`d`) 现在都与路由参数 `{d}` 相匹配。
* 从右到左的下一个文本是 `a`。 从我们停止的地方开始搜索 `/aab|c|d`，然后 `/a|a|b|c|d` 找到 `a`。
* 右侧的值 (`b`) 现在与路由参数 `{b}` 相匹配。
* 此时还有剩余的文本 `a`，但是算法已经耗尽了要解析的路由模板，所以这不是一个匹配项。

匹配算法是[非贪婪](#greedy)算法：

* 它匹配每个步骤中最小可能文本量。
* 如果分隔符值出现在参数值内，则会导致不匹配。

正则表达式可以更好地控制它们的匹配行为。

<a name="greedy"></a>

贪婪匹配（也称为[懒惰匹配](https://wikipedia.org/wiki/Regular_expression#Lazy_matching)）匹配最大可能字符串。 非贪婪匹配最小可能字符串。

## <a name="route-constraint-reference"></a>路由约束参考

路由约束在传入 URL 发生匹配时执行，URL 路径标记为路由值。 路径约束通常检查通过路径模板关联的路径值，并对该值是否为可接受做出对/错决定。 某些路由约束使用路由值以外的数据来考虑是否可以路由请求。 例如，<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> 可以根据其 HTTP 谓词接受或拒绝请求。 约束用于路由请求和链接生成。

> [!WARNING]
> 请勿将约束用于输入验证。 如果约束用于输入验证，则无效的输入将导致 `404`（找不到页面）响应。 无效输入可能生成包含相应错误消息的 `400` 错误请求。 路由约束用于消除类似路由的歧义，而不是验证特定路由的输入。

下表演示示例路由约束及其预期行为：

| 约束 | 示例 | 匹配项示例 | 说明 |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789` | 匹配任何整数 |
| `bool` | `{active:bool}` | `true`, `FALSE` | 匹配 `true` 或 `false`。 不区分大小写 |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | 在固定区域性中匹配有效的 `DateTime` 值。 请参阅前面的警告。 |
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | 在固定区域性中匹配有效的 `decimal` 值。 请参阅前面的警告。|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | 在固定区域性中匹配有效的 `double` 值。 请参阅前面的警告。|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | 在固定区域性中匹配有效的 `float` 值。 请参阅前面的警告。|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638` | 匹配有效的 `Guid` 值 |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | 匹配有效的 `long` 值 |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 字符串必须至少为 4 个字符 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `MyFile` | 字符串不得超过 8 个字符 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 字符串必须正好为 12 个字符 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 字符串必须至少为 8 个字符，且不得超过 16 个字符 |
| `min(value)` | `{age:min(18)}` | `19` | 整数值必须至少为 18 |
| `max(value)` | `{age:max(120)}` | `91` | 整数值不得超过 120 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 整数值必须至少为 18，且不得超过 120 |
| `alpha` | `{name:alpha}` | `Rick` | 字符串必须由一个或多个字母字符组成，`a`-`z`，并区分大小写。 |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 字符串必须与正则表达式匹配。 请参阅有关定义正则表达式的提示。 |
| `required` | `{name:required}` | `Rick` | 用于强制在 URL 生成过程中存在非参数值 |

[!INCLUDE[](~/includes/regex.md)]

可向单个参数应用多个用冒号分隔的约束。 例如，以下约束将参数限制为大于或等于 1 的整数值：

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> 验证 URL 的路由约束并将转换为始终使用固定区域性的 CLR 类型。 例如，转换为 CLR 类型 `int` 或 `DateTime`。 这些约束假定 URL 不可本地化。 框架提供的路由约束不会修改存储于路由值中的值。 从 URL 中分析的所有路由值都将存储为字符串。 例如，`float` 约束会尝试将路由值转换为浮点数，但转换后的值仅用来验证其是否可转换为浮点数。

### <a name="regular-expressions-in-constraints"></a>约束中的正则表达式

[!INCLUDE[](~/includes/regex.md)]

使用 `regex(...)` 路由约束可以将正则表达式指定为内联约束。 <xref:Microsoft.AspNetCore.Builder.ControllerEndpointRouteBuilderExtensions.MapControllerRoute*> 系列中的方法还接受约束的对象文字。 如果使用该窗体，则字符串值将解释为正则表达式。

下面的代码使用内联 regex 约束：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRegex.cs?name=snippet)]

下面的代码使用对象文字来指定 regex 约束：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRegex2.cs?name=snippet)]

ASP.NET Core 框架将向正则表达式构造函数添加 `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant`。 有关这些成员的说明，请参阅 <xref:System.Text.RegularExpressions.RegexOptions>。

正则表达式与路由和 C# 语言使用的分隔符和令牌相似。 必须对正则表达式令牌进行转义。 若要在内联约束中使用正则表达式 `^\d{3}-\d{2}-\d{4}$`，请使用以下某项：

* 将字符串中提供的 `\` 字符替换为 C# 源文件中的 `\\` 字符，以便对 `\` 字符串转义字符进行转义。
* [逐字字符串文本](/dotnet/csharp/language-reference/keywords/string)。

要对路由参数分隔符 `{`、`}`、`[`、`]` 进行转义，请将表达式（例如 `{{`、`}}`、`[[`、`]]`）中的字符数加倍。 下表展示了正则表达式及其转义的版本：

| 正则表达式    | 转义后的正则表达式     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

路由中使用的正则表达式通常以 `^` 字符开头，并匹配字符串的起始位置。 表达式通常以 `$` 字符结尾，并匹配字符串的结尾。 `^` 和 `$` 字符可确保正则表达式匹配整个路由参数值。 如果没有 `^` 和 `$` 字符，正则表达式将匹配字符串内的所有子字符串，而这通常是不需要的。 下表提供了示例并说明了它们匹配或匹配失败的原因：

| 表达式   | String    | 匹配 | 注释               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | 是   | 子字符串匹配     |
| `[a-z]{2}`   | 123abc456 | 是   | 子字符串匹配     |
| `[a-z]{2}`   | mz        | 是   | 匹配表达式    |
| `[a-z]{2}`   | MZ        | 是   | 不区分大小写    |
| `^[a-z]{2}$` | hello     | 否    | 参阅上述 `^` 和 `$` |
| `^[a-z]{2}$` | 123abc456 | 否    | 参阅上述 `^` 和 `$` |

有关正则表达式语法的详细信息，请参阅 [.NET Framework 正则表达式](/dotnet/standard/base-types/regular-expression-language-quick-reference)。

若要将参数限制为一组已知的可能值，可使用正则表达式。 例如，`{action:regex(^(list|get|create)$)}` 仅将 `action` 路由值匹配到 `list`、`get` 或 `create`。 如果传递到约束字典中，字符串 `^(list|get|create)$` 将等效。 已传递到约束字典且不匹配任何已知约束的约束也将被视为正则表达式。 模板内传递且不匹配任何已知约束的约束将不被视为正则表达式。

### <a name="custom-route-constraints"></a>自定义路由约束

可实现 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 接口来创建自定义路由约束。 `IRouteConstraint` 接口包含 <xref:System.Web.Routing.IRouteConstraint.Match*>，当满足约束时，它返回 `true`，否则返回 `false`。

很少需要自定义路由约束。 在实现自定义路由约束之前，请考虑替代方法，如模型绑定。

ASP.NET Core [Constraints](https://github.com/dotnet/aspnetcore/tree/master/src/Http/Routing/src/Constraints) 文件夹提供了创建约束的经典示例。 例如 [GuidRouteConstraint](https://github.com/dotnet/aspnetcore/blob/master/src/Http/Routing/src/Constraints/GuidRouteConstraint.cs#L18)。

若要使用自定义 `IRouteConstraint`，必须在服务容器中使用应用的 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 注册路由约束类型。 `ConstraintMap` 是将路由约束键映射到验证这些约束的 `IRouteConstraint` 实现的目录。 应用的 `ConstraintMap` 可作为 [services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 调用的一部分在 `Startup.ConfigureServices` 中进行更新，也可以通过使用 `services.Configure<RouteOptions>` 直接配置 <xref:Microsoft.AspNetCore.Routing.RouteOptions> 进行更新。 例如：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint.cs?name=snippet)]

前面的约束应用于以下代码：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/TestController.cs?name=snippet&highlight=6,13)]

[!INCLUDE[](~/includes/MyDisplayRouteInfo.md)]

实现 `MyCustomConstraint` 可防止将 `0` 应用于路由参数：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint.cs?name=snippet2)]

[!INCLUDE[](~/includes/regex.md)]

前面的代码：

* 阻止路由的 `{id}` 段中的 `0`。
* 显示以提供实现自定义约束的基本示例。 不应在产品应用中使用。

下面的代码是防止处理包含 `0` 的 `id` 的更好方法：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/TestController.cs?name=snippet2)]

前面的代码与 `MyCustomConstraint` 方法相比具有以下优势：

* 它不需要自定义约束。
* 当路由参数包括 `0` 时，它将返回更具描述性的错误。

## <a name="parameter-transformer-reference"></a>参数转换器参考

参数转换器：

* 使用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> 生成链接时执行。
* 实现 <xref:Microsoft.AspNetCore.Routing.IOutboundParameterTransformer?displayProperty=fullName>。
* 使用 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 进行配置。
* 获取参数的路由值并将其转换为新的字符串值。
* 在生成的链接中使用转换后的值的结果。

例如，路由模式 `blog\{article:slugify}`（具有 `Url.Action(new { article = "MyTestArticle" })`）中的自定义 `slugify` 参数转换器生成 `blog\my-test-article`。

请考虑以下 `IOutboundParameterTransformer` 实现：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint2.cs?name=snippet2)]

若要在路由模式中使用参数转换器，请在 `Startup.ConfigureServices` 中使用 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 对其进行配置：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupConstraint2.cs?name=snippet)]

ASP.NET Core 框架使用参数转化器来转换进行终结点解析的 URI。 例如，参数转换器转换用于匹配 `area`、`controller`、`action` 和 `page` 的路由值。

```csharp
routes.MapControllerRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

使用上述路由模板，可将操作 `SubscriptionManagementController.GetAll` 与 URI `/subscription-management/get-all` 相匹配。 参数转换器不会更改用于生成链接的路由值。 例如，`Url.Action("GetAll", "SubscriptionManagement")` 输出 `/subscription-management/get-all`。

对于结合使用参数转换器和所生成的路由，ASP.NET Core 提供了 API 约定：

* <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention?displayProperty=fullName> MVC 约定将指定的参数转换器应用于应用中的所有特性路由。 在替换属性路径令牌时，参数转换器将转换这些令牌。 有关详细信息，请参阅[使用参数转换器自定义标记替换](xref:mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement)。
* Razor Pages 使用 <xref:Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention> API 约定。 此约定将指定的参数转换器应用于所有自动发现的 Razor Pages。 参数转换器转换 Razor Pages 路由的文件夹和文件名段。 有关详细信息，请参阅[使用参数转换器自定义页面路由](xref:razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes)。

<a name="ugr"></a>

## <a name="url-generation-reference"></a>URL 生成参考

本部分包含 URL 生成实现的算法的参考。 在实践中，最复杂的 URL 生成示例使用控制器或 Razor Pages。 有关其他信息，请参阅[控制器中的路由](xref:mvc/controllers/routing)。

URL 生成过程首先调用 [GetPathByAddress](xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*) 或类似方法。 此方法提供了一个地址、一组路由值以及有关 `HttpContext` 中当前请求的可选信息。

第一步是使用地址解析一组候选终结点，该终结点使用与该地址类型匹配的 [`IEndpointAddressScheme<TAddress>`](xref:Microsoft.AspNetCore.Routing.IEndpointAddressScheme`1)。

地址方案找到一组候选项后，就会以迭代方式对终结点进行排序和处理，直到 URL 生成操作成功。 URL 生成不检查多义性，返回的第一个结果就是最终结果。

### <a name="troubleshooting-url-generation-with-logging"></a>使用日志记录对 URL 生成进行故障排除

对 URL 生成进行故障排除的第一步是将 `Microsoft.AspNetCore.Routing` 的日志记录级别设置为 `TRACE`。 `LinkGenerator` 记录有关其处理的许多详细信息，有助于排查问题。

有关 URL 生成的详细信息，请参阅 [URL 生成参考](#ugr)。

### <a name="addresses"></a>地址

地址是 URL 生成中的概念，用于将链接生成器中的调用绑定到一组终结点。

地址是一个可扩展的概念，默认情况下有两种实现：

* 使用终结点名称 (`string`) 作为地址：
    * 提供与 MVC 的路由名称相似的功能。
    * 使用 <xref:Microsoft.AspNetCore.Routing.IEndpointNameMetadata> 元数据类型。
    * 根据所有注册的终结点的元数据解析提供的字符串。
    * 如果多个终结点使用相同的名称，启动时会引发异常。
    * 建议用于控制器和 Razor Pages 之外的常规用途。
* 使用路由值 (<xref:Microsoft.AspNetCore.Routing.RouteValuesAddress>) 作为地址：
    * 提供与控制器和 Razor Pages 生成旧 URL 相同的功能。
    * 扩展和调试非常复杂。
    * 提供 `IUrlHelper`、标记帮助程序、HTML 帮助程序、操作结果等所使用的实现。

地址方案的作用是根据任意条件在地址和匹配终结点之间建立关联：

* 终结点名称方案执行基本字典查找。
* 路由值方案有一个复杂的子集算法，这是集算法的最佳子集。

<a name="ambient"></a>

### <a name="ambient-values-and-explicit-values"></a>环境值和显式值

从当前请求开始，路由将访问当前请求 `HttpContext.Request.RouteValues` 的路由值。 与当前请求关联的值称为环境值。 为清楚起见，文档是指作为显式值传入方法的路由值。

下面的示例演示了环境值和显式值。 它将提供当前请求中的环境值和显式值：`{ id = 17, }`：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet)]

前面的代码：

* 返回 `/Widget/Index/17`
* 通过 [DI](xref:fundamentals/dependency-injection) 获取 <xref:Microsoft.AspNetCore.Routing.LinkGenerator>。

下面的代码不提供环境值和显式值：`{ controller = "Home", action = "Subscribe", id = 17, }`：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet2)]

前面的方法返回 `/Home/Subscribe/17`

`WidgetController` 中的下列代码返回 `/Widget/Subscribe/17`：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/WidgetController.cs?name=snippet3)]

下面的代码提供当前请求中的环境值和显式值中的控制器：`{ action = "Edit", id = 17, }`：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/GadgetController.cs?name=snippet)]

在上述代码中：

* 返回 `/Gadget/Edit/17`。
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Url> 获取 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper>。
* <xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Action*>   
生成一个 URL，其中包含操作方法的绝对路径。 URL 包含指定的 `action` 名称和 `route` 值。

下面的代码提供当前请求中的环境值和显式值：`{ page = "./Edit, id = 17, }`：

[!code-csharp[](routing/samples/3.x/RoutingSample/Pages/Index.cshtml.cs?name=snippet)]

当“编辑 Razor”页包含以下页面指令时，前面的代码会将 `url` 设置为 `/Edit/17`：

 `@page "{id:int}"`

如果“编辑”页面不包含 `"{id:int}"` 路由模板，`url` 为 `/Edit?id=17`。

除了此处所述的规则外，MVC <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> 的行为还增加了一层复杂性：

* `IUrlHelper` 始终将当前请求中的路由值作为环境值提供。
* [IUrlHelper](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Action*) 始终将当前 `action` 和 `controller` 路由值复制为显式值，除非由开发者重写。
* [IUrlHelper](xref:Microsoft.AspNetCore.Mvc.UrlHelperExtensions.Page*) 始终将当前 `page` 路由值复制为显式值，除非重写。 <!--by the user-->
* `IUrlHelper.Page` 始终替代当前 `handler` 路由值，且 `null` 为显式值，除非重写。

用户常常对环境值的行为详细信息感到惊讶，因为 MVC 似乎不遵循自己的规则。 出于历史和兼容性原因，某些路由值（例如 `action`、`controller`、`page` 和 `handler`）具有其自己的特殊行为。

`LinkGenerator.GetPathByAction` 和 `LinkGenerator.GetPathByPage` 提供的等效功能与 `IUrlHelper` 的兼容性异常相同。

### <a name="url-generation-process"></a>URL 生成过程

找到候选终结点集后，URL 生成算法将：

* 以迭代方式处理终结点。
* 返回第一个成功的结果。

此过程的第一步称为路由值失效。  路由值失效是路由决定应使用环境值中的哪些路由值以及应忽略哪些路由值的过程。 将考虑每个环境值，要么与显式值组合，要么忽略它们。

考虑环境值角色的最佳方式是在某些常见情况下，尝试保存应用程序开发者的键入内容。 就传统意义而言，环境值非常有用的情况与 MVC 相关：

* 链接到同一控制器中的其他操作时，不需要指定控制器名称。
* 链接到同一区域中的另一个控制器时，不需要指定区域名称。
* 链接到相同的操作方法时，不需要指定路由值。
* 链接到应用的其他部分时，不需要在应用的该部分中传递无意义的路由值。

如果不了解路由值失效，通常就会导致对返回 `null` 的 `LinkGenerator` 或 `IUrlHelper` 执行调用。 显式指定更多路由值，对路由值失效进行故障排除，以查看是否解决了问题。

路由值失效的前提是应用的 URL 方案是分层的，并按从左到右的顺序组成层次结构。 请考虑基本控制器路由模板 `{controller}/{action}/{id?}`，以直观地了解实践操作中该模板的工作方式。 对值进行更改会使右侧显示的所有路由值都失效 。 这反映了关于层次结构的假设。 如果应用的 `id` 有一个环境值，则操作会为 `controller` 指定一个不同的值：

* `id` 不会重复使用，因为 `{controller}` 在 `{id?}` 的左侧。

演示此原则的一些示例如下：

* 如果显式值包含 `id` 的值，则将忽略 `id` 的环境值。 可以使用 `controller` 和 `action` 的环境值。
* 如果显式值包含 `action` 的值，则将忽略 `action` 的任何环境值。 可以使用 `controller` 的环境值。 如果 `action` 的显式值不同于 `action` 的环境值，则不会使用 `id` 值。  如果 `action` 的显式值与 `action` 的环境值相同，则可以使用 `id` 值。
* 如果显式值包含 `controller` 的值，则将忽略 `controller` 的任何环境值。 如果 `controller` 的显式值不同于 `controller` 的环境值，则不会使用 `action` 和 `id` 值。 如果 `controller` 的显式值与 `controller` 的环境值相同，则可以使用 `action` 和 `id` 值。

由于存在特性路由和专用传统路由，此过程将变得更加复杂。 控制器传统路由（例如 `{controller}/{action}/{id?}`）使用路由参数指定层次结构。 对于控制器和 Razor Pages 的[专用传统路由](xref:mvc/controllers/routing#dcr)和[特性路由](xref:mvc/controllers/routing#ar)：

* 有一个路由值层次结构。
* 它们不会出现在模板中。

对于这些情况，URL 生成将定义“所需值”这一概念。 而由控制器和 Razor Pages 创建的终结点将指定所需值，以允许路由值失效起作用。

路由值失效算法的详细信息：

* 所需值名称与路由参数组合在一起，然后从左到右进行处理。
* 对于每个参数，将比较环境值和显式值：
    * 如果环境值和显式值相同，则该过程将继续。
    * 如果环境值存在而显式值不存在，则在生成 URL 时使用环境值。
    * 如果环境值不存在而显式值存在，则拒绝环境值和所有后续环境值。
    * 如果环境值和显式值均存在，并且这两个值不同，则拒绝环境值和所有后续环境值。

此时，URL 生成操作就可以计算路由约束了。 接受的值集与提供给约束的参数默认值相结合。 如果所有约束都通过，则操作将继续。

接下来，接受的值可用于扩展路由模板。 处理路由模板：

* 从左到右。
* 每个参数都替换了接受的值。
* 具有以下特殊情况：
  * 如果接受的值缺少一个值并且参数具有默认值，则使用默认值。
  * 如果接受的值缺少一个值并且参数是可选的，则继续处理。
  * 如果缺少的可选参数右侧的任何路由参数都具有值，则操作将失败。
  * <!-- review default-valued parameters optional parameters --> 如果可能，连续的默认值参数和可选参数会折叠。

显式提供且与路由片段不匹配的值将添加到查询字符串中。 下表显示使用路由模板 `{controller}/{action}/{id?}` 时的结果。

| 环境值                     | 显式值                        | 结果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| 控制器 =“Home”                | 操作 =“About”                       | `/Home/About`           |
| 控制器 =“Home”                | 控制器 =“Order”，操作 =“About” | `/Order/About`          |
| 控制器 = “Home”，颜色 = “Red” | 操作 =“About”                       | `/Home/About`           |
| 控制器 =“Home”                | 操作 =“About”，颜色 =“Red”        | `/Home/About?color=Red` |

### <a name="problems-with-route-value-invalidation"></a>路由值失效的问题

从 ASP.NET Core 3.0 开始，早期 ASP.NET Core 版本中使用的一些 URL 生成方案不适用于 URL 生成。 ASP.NET Core 团队计划在未来的版本中添加功能来满足这些需求。 现在，最佳解决方案是使用旧路由。

下面的代码显示了路由不支持的 URL 生成方案示例。

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupUnsupported.cs?name=snippet)]

在前面的代码中，`culture` 路由参数用于本地化。 需要将 `culture` 参数始终作为环境值接受。 但由于所需值的工作方式，不会将 `culture` 参数作为环境值接受：

* 在 `"default"` 路由模板中，`culture` 路由参数位于 `controller` 的左侧，因此对 `controller` 的更改不会使 `culture` 失效。
* 在 `"blog"` 路由模板中，`culture` 路由参数应在 `controller` 的右侧，这会显示在所需值中。

## <a name="configuring-endpoint-metadata"></a>配置终结点元数据

以下链接提供有关配置终结点元数据的信息：

* [通过终结点路由启用 Cors](xref:security/cors#enable-cors-with-endpoint-routing)
* 使用自定义 `[MinimumAgeAuthorize]` 属性的 [IAuthorizationPolicyProvider 示例](https://github.com/dotnet/AspNetCore/tree/release/3.1/src/Security/samples/CustomPolicyProvider)
* [使用 [Authorize] 属性测试身份验证](xref:security/authentication/identity#test-identity)
* <xref:Microsoft.AspNetCore.Builder.AuthorizationEndpointConventionBuilderExtensions.RequireAuthorization*>
* [使用 [Authorize] 属性选择方案](xref:security/authorization/limitingidentitybyscheme#selecting-the-scheme-with-the-authorize-attribute)
* [使用 [Authorize] 属性应用策略](xref:security/authorization/policies#apply-policies-to-mvc-controllers)
* <xref:security/authorization/roles>

<a name="hostmatch"></a>

## <a name="host-matching-in-routes-with-requirehost"></a>路由中与 RequireHost 匹配的主机

<xref:Microsoft.AspNetCore.Builder.RoutingEndpointConventionBuilderExtensions.RequireHost*> 将约束应用于需要指定主机的路由。 `RequireHost` 或 [[Host]](xref:Microsoft.AspNetCore.Routing.HostAttribute) 参数可以是：

* 主机：`www.domain.com`匹配任何端口的 `www.domain.com`。
* 带有通配符的主机：`*.domain.com`，匹配任何端口上的 `www.domain.com`、`subdomain.domain.com` 或 `www.subdomain.domain.com`。
* 端口：`*:5000` 匹配任何主机的端口 5000。
* 主机和端口：`www.domain.com:5000` 或 `*.domain.com:5000`（匹配主机和端口）。

可以使用 `RequireHost` 或 `[Host]` 指定多个参数。 约束匹配对任何参数均有效的主机。 例如，`[Host("domain.com", "*.domain.com")]` 匹配 `domain.com`、`www.domain.com` 和 `subdomain.domain.com`。

以下代码使用 `RequireHost` 来要求路由上的指定主机：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupRequireHost.cs?name=snippet)]

以下代码使用控制器上的 `[Host]` 特性来要求任何指定的主机：

[!code-csharp[](routing/samples/3.x/RoutingSample/Controllers/ProductController.cs?name=snippet)]

当 `[Host]` 属性同时应用于控制器和操作方法时：

* 使用操作上的属性。
* 忽略控制器属性。

## <a name="performance-guidance-for-routing"></a>路由的性能指南

大多数路由在 ASP.NET Core 3.0 中都进行了更新，以提高性能。

当应用出现性能问题时，人们常常会怀疑路由是问题所在。 怀疑路由的原因是，控制器和 Razor Pages 等框架在其日志记录消息中报告框架内所用的时间。 如果控制器报告的时间与请求的总时间之间存在明显的差异：

* 开发者会将应用代码排除在问题根源之外。
* 他们通常认为路由是问题的原因。

路由的性能使用数千个终结点进行测试。 典型的应用不太可能仅仅因为太大而遇到性能问题。 路由性能缓慢的最常见根本原因通常在于性能不佳的自定义中间件。

下面的代码示例演示了一种用于缩小延迟源的基本方法：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupDelay.cs?name=snippet)]

时间路由：

* 使用前面代码中所示的计时中间件的副本交错执行每个中间件。
* 添加唯一标识符，以便将计时数据与代码相关联。

这是一种可在延迟显著的情况下减少延迟的基本方法，例如超过 `10ms`。  从 `Time 1` 中减去 `Time 2` 会报告 `UseRouting` 中间件内所用的时间。

下面的代码使用一种更紧凑的方法来处理前面的计时代码：

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupSW.cs?name=snippetSW)]

[!code-csharp[](routing/samples/3.x/RoutingSample/StartupSW.cs?name=snippet)]

### <a name="potentially-expensive-routing-features"></a>可能比较昂贵的路由功能

下面的列表提供了一些路由功能，这些功能相对于基本路由模板来说比较昂贵：

* 正则表达式：可以编写复杂的正则表达式，或具有少量输入的长时间运行时间。

* 复杂段 (`{x}-{y}-{z}`)： 
  * 比分析常规 URL 路径段贵得多。
  * 导致更多的子字符串被分配。
  * ASP.NET Core 3.0 路由性能更新中未更新的复杂段逻辑。

* 同步数据访问：许多复杂应用都将数据库访问作为其路由的一部分。 ASP.NET Core 2.2 及更低版本的路由可能未提供适当的扩展点，因此无法支持数据库访问路由。 例如，<xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 和 <xref:Microsoft.AspNetCore.Mvc.ActionConstraints.IActionConstraint> 是同步的。 扩展点（如 <xref:Microsoft.AspNetCore.Routing.MatcherPolicy> 和 <xref:Microsoft.AspNetCore.Routing.EndpointSelectorContext>）是异步的。

## <a name="guidance-for-library-authors"></a>库创建者指南

本部分包含基于路由构建的库创建者指南。 这些详细信息旨在确保应用开发者可以在使用扩展路由的库和框架时具有良好的体验。

### <a name="define-endpoints"></a>定义终结点

若要创建一个使用路由进行 URL 匹配的框架，请先定义在 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 之上进行构建的用户体验。

在 <xref:Microsoft.AspNetCore.Routing.IEndpointRouteBuilder> 之上进行构建。 由此，用户就可以用其他 ASP.NET Core 功能编写框架，而不会造成混淆。 每个 ASP.NET Core 模板都包含路由。 假定路由存在并且用户熟悉路由。

```csharp
app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...);

    endpoints.MapHealthChecks("/healthz");
});
```

从对实现 <xref:Microsoft.AspNetCore.Builder.IEndpointConventionBuilder> 的 `MapMyFramework(...)` 的调用返回密式具体类型。 大多数框架 `Map...` 方法都遵循此模式。 `IEndpointConventionBuilder` 接口：

* 允许元数据的可组合性。
* 面向多种扩展方法。

通过声明自己的类型，你可以将自己的框架特定功能添加到生成器中。 可以包装一个框架声明的生成器并向其转发调用。

```csharp
app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...).RequireAuthorization()
                                 .WithMyFrameworkFeature(awesome: true);

    endpoints.MapHealthChecks("/healthz");
});
```

请考虑编写自己的 <xref:Microsoft.AspNetCore.Routing.EndpointDataSource>。 `EndpointDataSource` 是用于声明和更新终结点集合的低级别基元。 `EndpointDataSource` 是控制器和 Razor Pages 使用的强大 API。

路由测试具有非更新数据源的[基本示例](https://github.com/aspnet/AspNetCore/blob/master/src/Http/Routing/test/testassets/RoutingSandbox/Framework/FrameworkEndpointDataSource.cs#L17)。

默认情况下，请不要尝试注册 `EndpointDataSource`。 要求用户在 <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseEndpoints*> 中注册你的框架。 路由的理念是，默认情况下不包含任何内容，并且 `UseEndpoints` 是注册终结点的位置。

### <a name="creating-routing-integrated-middleware"></a>创建路由集成式中间件

请考虑将元数据类型定义为接口。

请实现在类和方法上使用元数据类型。

[!code-csharp[](routing/samples/3.x/RoutingSample/ICoolMetadata.cs?name=snippet2)]

控制器和 Razor Pages 等框架支持将元数据特性应用到类型和方法。 如果声明元数据类型：

* 将它们作为[特性](/dotnet/csharp/programming-guide/concepts/attributes/)进行访问。
* 大多数用户都熟悉应用特性。

将元数据类型声明为接口可增加另一层灵活性：

* 接口是可组合的。
* 开发者可以声明自己的且组合多个策略的类型。

请实现元数据替代，如以下示例中所示：

[!code-csharp[](routing/samples/3.x/RoutingSample/ICoolMetadata.cs?name=snippet)]

遵循这些准则的最佳方式是避免定义标记元数据：

* 不要只是查找元数据类型的状态。
* 定义元数据的属性并检查该属性。

对元数据集合进行排序，并支持按优先级替代。 对于控制器，操作方法上的元数据是最具体的。

使中间件始终有用，不管有没有路由。

```csharp
app.UseRouting();

app.UseAuthorization(new AuthorizationPolicy() { ... });

app.UseEndpoints(endpoints =>
{
    // Your framework
    endpoints.MapMyFramework(...).RequireAuthorization();
});
```

作为此准则的一个示例，请考虑 `UseAuthorization` 中间件。 授权中间件允许传入回退策略。 <!-- shown where?  (shown here) --> 如果指定了回退策略，则适用于以下两者：

* 无指定策略的终结点。
* 与终结点不匹配的请求。

这使得授权中间件在路由上下文之外很有用。 授权中间件可用于传统中间件的编程。

[!INCLUDE[](~/includes/dbg-route.md)]

::: moniker-end

::: moniker range="= aspnetcore-2.2"

路由负责将请求 URI 映射到终结点并向这些终结点调度传入的请求。 路由在应用中定义，并在应用启动时进行配置。 路由可以选择从请求包含的 URL 中提取值，然后这些值便可用于处理请求。 通过使用应用中的路由信息，路由还能生成映射到终结点的 URL。

要在 ASP.NET Core 2.2 中使用最新路由方案，请在 `Startup.ConfigureServices` 中为 MVC 服务注册指定[兼容性版本](xref:mvc/compatibility-version)：

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

<xref:Microsoft.AspNetCore.Mvc.MvcOptions.EnableEndpointRouting> 选项确定路由是应在内部使用 ASP.NET Core 2.1 或更早版本的基于终结点的逻辑还是使用其基于 <xref:Microsoft.AspNetCore.Routing.IRouter> 的逻辑。 兼容性版本设置为 2.2 或更高版本时，默认值为 `true`。 将值设置为 `false` 以使用先前的路由逻辑：

```csharp
// Use the routing logic of ASP.NET Core 2.1 or earlier:
services.AddMvc(options => options.EnableEndpointRouting = false)
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

有关基于 <xref:Microsoft.AspNetCore.Routing.IRouter> 的路由的详细信息，请参阅本主题的 [ASP.NET Core 2.1 版本](?view=aspnetcore-2.1)。

> [!IMPORTANT]
> 本文档介绍较低级别的 ASP.NET Core 路由。 有关 ASP.NET Core MVC 路由的信息，请参阅 <xref:mvc/controllers/routing>。 有关 Razor Pages 中路由约定的信息，请参阅 <xref:razor-pages/razor-pages-conventions>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="routing-basics"></a>路由基础知识

大多数应用应选择基本的描述性路由方案，让 URL 有可读性和意义。 默认传统路由 `{controller=Home}/{action=Index}/{id?}`：

* 支持基本的描述性路由方案。
* 是基于 UI 的应用的有用起点。

开发者通常在专业情况下使用[特性路由](xref:mvc/controllers/routing#attribute-routing)或专用传统路由向应用的高流量区域添加其他简洁路由。 专用情况示例包括：博客和电子商务终结点。

Web API 应使用属性路由，将应用功能建模为一组资源，其中操作是由 HTTP 谓词表示。 也就是说，对同一逻辑资源执行的许多操作（例如，GET 和 POST）都使用相同 URL。 属性路由提供了精心设计 API 的公共终结点布局所需的控制级别。

Razor Pages 应用使用默认的传统路由从应用的“页面”文件夹中提供命名资源。 还可以使用其他约定来自定义 Razor Pages 路由行为。 有关详细信息，请参阅 <xref:razor-pages/index> 和 <xref:razor-pages/razor-pages-conventions>。

借助 URL 生成支持，无需通过硬编码 URL 将应用关联到一起，即可开发应用。 此支持允许从基本路由配置入手，并在确定应用的资源布局后修改路由。

路由使用“终结点”(`Endpoint`) 来表示应用中的逻辑终结点。

终结点定义用于处理请求的委托和任意元数据的集合。 元数据用于实现横切关注点，该实现基于附加到每个终结点的策略和配置。

路由系统具有以下特征：

* 路由模板语法用于通过标记化路由参数来定义路由。
* 可以使用常规样式和属性样式终结点配置。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 用于确定 URL 参数是否包含给定的终结点约束的有效值。
* 应用模型（如 MVC/Razor Pages）注册其所有终结点，这些终结点具有可预测的路由方案实现。
* 路由实现会在中间件管道中任何所需位置制定路由决策。
* 路由中间件之后出现的中间件可以检查路由中间件针对给定请求 URI 的终结点决策结果。
* 可以在中间件管道中的任何位置枚举应用中的所有终结点。
* 应用可根据终结点信息使用路由生成 URL（例如，用于重定向或链接），从而避免硬编码 URL，这有助于可维护性。
* URL 生成是基于支持任意可扩展性的地址：

  * 可以使用[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 在任意位置解析链接生成器 API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>) 以生成 URL。
  * 如果无法通过 DI 获得链接生成器 API，则 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> 会提供生成 URL 的方法。

> [!NOTE]
> 在 ASP.NET Core 2.2 中发布终结点路由后，终结点链接的适用范围限制为 MVC/Razor Pages 操作和页面。 将计划在未来发布的版本中扩展终结点链接的功能。

路由通过 <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> 类连接到[中间件](xref:fundamentals/middleware/index)管道。 [ASP.NET Core MVC](xref:mvc/overview) 向中间件管道添加路由，作为其配置的一部分，并处理 MVC 和 Razor Pages 应用中的路由。 要了解如何将路由用作独立组件，请参阅[使用路由中间件](#use-routing-middleware)部分。

### <a name="url-matching"></a>URL 匹配

URL 匹配是路由向终结点调度传入请求的过程。 此过程基于 URL 路径中的数据，但可以进行扩展以考虑请求中的任何数据。 向单独的处理程序调度请求的功能是缩放应用的大小和复杂性的关键。

终结点路由中的路由系统负责所有的调度决策。 由于中间件是基于所选终结点来应用策略，因此任何可能影响调度或安全策略应用的决策都应在路由系统内部制定，这一点很重要。

执行终结点委托时，将根据迄今执行的请求处理将 [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) 的属性设为适当的值。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) 是从路由中生成的路由值的字典。 这些值通常通过标记 URL 来确定，可用来接受用户输入，或者在应用内作出进一步的调度决策。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) 是一个与匹配的路由相关的其他数据的属性包。 提供 <xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> 以支持将状态数据与每个路由相关联，以便应用可根据所匹配的路由作出决策。 这些值是开发者定义的，不会影响通过任何方式路由的行为。 此外，存储于 [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) 中的值可以属于任何类型，与 [RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) 相反，后者必须能够转换为字符串，或从字符串进行转换。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) 是参与成功匹配请求的路由的列表。 路由可以相互嵌套。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> 属性可以通过导致匹配的逻辑路由树反映该路径。 通常情况下，<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> 中的第一项是路由集合，应该用于生成 URL。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> 中的最后一项是匹配的路由处理程序。

<a name="lg"></a>

### <a name="url-generation-with-linkgenerator"></a>使用 LinkGenerator 的 URL 生成

URL 生成是通过其可根据一组路由值创建 URL 路径的过程。 这需要考虑终结点与访问它们的 URL 之间的逻辑分隔。

终结点路由包含链接生成器 API (<xref:Microsoft.AspNetCore.Routing.LinkGenerator>)。 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> 是可从 [DI](xref:fundamentals/dependency-injection) 检索的单一实例服务。 该 API 可在执行请求的上下文之外使用。 MVC 的 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> 和依赖 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> 的方案（如 [Tag Helpers](xref:mvc/views/tag-helpers/intro)、HTML Helpers 和 [Action Results](xref:mvc/controllers/actions)）使用链接生成器提供链接生成功能。

链接生成器基于“地址”和“地址方案”概念 。 地址方案是确定哪些终结点用于链接生成的方式。 例如，许多用户熟悉的来自 MVC/Razor Pages 的路由名称和路由值方案都是作为地址方案实现的。

链接生成器可以通过以下扩展方法链接到 MVC/Razor Pages 操作和页面：

* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetPathByAction*>
* <xref:Microsoft.AspNetCore.Routing.ControllerLinkGeneratorExtensions.GetUriByAction*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetPathByPage*>
* <xref:Microsoft.AspNetCore.Routing.PageLinkGeneratorExtensions.GetUriByPage*>

这些方法的重载接受包含 `HttpContext` 的参数。 这些方法在功能上等同于 `Url.Action` 和 `Url.Page`，但提供了更大的灵活性和更多的选项。

`GetPath*` 方法与 `Url.Action` 和 `Url.Page` 最相似，因为它们生成包含绝对路径的 URI。 `GetUri*` 方法始终生成包含方案和主机的绝对 URI。 接受 `HttpContext` 的方法在执行请求的上下文中生成 URI。 除非重写，否则将使用来自执行请求的环境路由值、URL 基本路径、方案和主机。

使用地址调用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator>。 生成 URI 的过程分两步进行：

1. 将地址绑定到与地址匹配的终结点列表。
1. 计算每个终结点的 `RoutePattern`，直到找到与提供的值匹配的路由模式。 输出结果会与提供给链接生成器的其他 URI 部分进行组合并返回。

对任何类型的地址，<xref:Microsoft.AspNetCore.Routing.LinkGenerator> 提供的方法均支持标准链接生成功能。 使用链接生成器的最简便方法是通过扩展方法对特定地址类型执行操作。

| 扩展方法   | 描述                                                         |
| ------------------ | ------------------------------------------------------------------- |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetPathByAddress*> | 根据提供的值生成具有绝对路径的 URI。 |
| <xref:Microsoft.AspNetCore.Routing.LinkGenerator.GetUriByAddress*> | 根据提供的值生成绝对 URI。             |

> [!WARNING]
> 请注意有关调用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> 方法的下列含义：
>
> * 对于不验证传入请求的 `Host` 标头的应用配置，请谨慎使用 `GetUri*` 扩展方法。 如果未验证传入请求的 `Host`标头，则可能以视图/页面中 URI 的形式将不受信任的请求输入发送回客户端。 建议所有生产应用都将其服务器配置为针对已知有效值验证 `Host` 标头。
>
> * 在中间件中将 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> 与 `Map` 或 `MapWhen` 结合使用时，请小心谨慎。 `Map*` 会更改执行请求的基路径，这会影响链接生成的输出。 所有 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API 都允许指定基路径。 始终指定一个空的基路径来撤消 `Map*` 对链接生成的影响。

## <a name="differences-from-earlier-versions-of-routing"></a>与早期版本路由的差异

ASP.NET Core 2.2 或更高版本中的终结点路由与 ASP.NET Core 中早期版本的路由之间存在一些差异：

* 终结点路由系统不支持基于 <xref:Microsoft.AspNetCore.Routing.IRouter> 的可扩展性，包括从 <xref:Microsoft.AspNetCore.Routing.Route> 继承。

* 终结点路由不支持 [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim)。 使用 2.1 [兼容性版本](xref:mvc/compatibility-version) (`.SetCompatibilityVersion(CompatibilityVersion.Version_2_1)`) 以继续使用兼容性填充码。

* 在使用传统路由时，终结点路由对生成的 URI 的大小写具有不同的行为。

  请考虑以下默认路由模板：

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  假设使用以下路由生成某个操作的链接：

  ```csharp
  var link = Url.Action("ReadPost", "blog", new { id = 17, });
  ```

  如果使用基于 <xref:Microsoft.AspNetCore.Routing.IRouter> 的路由，此代码生成 URI `/blog/ReadPost/17`，该 URI 遵循所提供路由值的大小写。 ASP.NET Core 2.2 或更高版本中的终结点路由生成 `/Blog/ReadPost/17`（“Blog”大写）。 终结点路由提供 `IOutboundParameterTransformer` 接口，可用于在全局范围自定义此行为或应用不同的约定来映射 URL。

  有关详细信息，请参阅[参数转换器参考](#parameter-transformer-reference)部分。

* 在试图链接到不存在的控制器/操作或页面时，MVC/Razor Pages 通过传统路由执行的链接生成，其操作具有不同的行为。

  请考虑以下默认路由模板：

  ```csharp
  app.UseMvc(routes =>
  {
      routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
  });
  ```

  假设使用以下路由通过默认模板生成某个操作的链接：

  ```csharp
  var link = Url.Action("ReadPost", "Blog", new { id = 17, });
  ```

  如果使用基于 `IRouter` 的路由，即使 `BlogController` 不存在或没有 `ReadPost` 操作方法，结果也始终为 `/Blog/ReadPost/17`。 正如所料，如果操作方法存在，ASP.NET Core 2.2 或更高版本中的终结点路由会生成 `/Blog/ReadPost/17`。 但是，如果操作不存在，终结点路由会生成空字符串。 从概念上讲，如果操作不存在，终结点路由不会假定终结点存在。

* 与终结点路由一起使用时，链接生成环境值失效算法的行为会有所不同。

  *环境值失效* 是一种算法，用于决定当前正在执行的请求（环境值）中的哪些路由值可用于链接生成操作。 链接到不同操作时，传统路由会使额外的路由值失效。 ASP.NET Core 2.2 之前的版本中，属性路由不具有此行为。 在 ASP.NET Core 的早期版本中，如果有另一个操作使用同一路由参数名称，则该操作的链接会导致发生链接生成错误。 在 ASP.NET Core 2.2 或更高版本中，链接到另一个操作时，这两种路由形式都会使值失效。

  请考虑 ASP.NET Core2.1 或更高版本中的以下示例。 链接到另一个操作（或另一页面）时，路由值可能会按非预期的方式被重用。

  在 /Pages/Store/Product.cshtml 中：

  ```cshtml
  @page "{id}"
  @Url.Page("/Login")
  ```

  在 /Pages/Login.cshtml 中：

  ```cshtml
  @page "{id?}"
  ```

  如果 ASP.NET Core 2.1 或更早版本中的 URI 为 `/Store/Product/18`，则由 `@Url.Page("/Login")` 在 Store/Info 页面上生成的链接为 `/Login/18`。 即使链接目标完全是应用的其他部分，仍会重用 `id` 值 18。 `/Login` 页面上下文中的 `id` 路由值可能是用户 ID 值，而非存储产品 ID 值。

  在 ASP.NET Core 2.2 或更高版本的终结点路由中，结果为 `/Login`。 当链接的目标是另一个操作或页面时，不会重复使用环境值。

* 往返路由参数语法：使用双星号 (`**`) catch-all 参数语法时，不对正斜杠进行编码。

  在链接生成期间，路由系统对双星号 (`**`) catch-all 参数（例如，`{**myparametername}`）中捕获的除正斜杠外的值进行编码。 在 ASP.NET Core 2.2 或更高版本中，基于 `IRouter` 的路由支持双星号 catch-all 参数。

  ASP.NET Core (`{*myparametername}`) 早期版本中的单星号 catch-all 参数语法仍然受支持，并对正斜杠进行编码。

  | 路由              | 链接生成方式为<br>`Url.Action(new { category = "admin/products" })`&hellip; |
  | ------------------ | --------------------------------------------------------------------- |
  | `/search/{*page}`  | `/search/admin%2Fproducts`（对正斜杠进行编码）             |
  | `/search/{**page}` | `/search/admin/products`                                              |

### <a name="middleware-example"></a>中间件示例

在以下示例中，中间件使用 <xref:Microsoft.AspNetCore.Routing.LinkGenerator> API 创建列出存储产品的操作方法的链接。 应用中的任何类都可通过将链接生成器注入类并调用 `GenerateLink` 来使用链接生成器。

```csharp
using Microsoft.AspNetCore.Routing;

public class ProductsLinkMiddleware
{
    private readonly LinkGenerator _linkGenerator;

    public ProductsLinkMiddleware(RequestDelegate next, LinkGenerator linkGenerator)
    {
        _linkGenerator = linkGenerator;
    }

    public async Task InvokeAsync(HttpContext httpContext)
    {
        var url = _linkGenerator.GetPathByAction("ListProducts", "Store");

        httpContext.Response.ContentType = "text/plain";

        await httpContext.Response.WriteAsync($"Go to {url} to see our products.");
    }
}
```

### <a name="create-routes"></a>创建路由

大多数应用通过调用 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 或 <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 上定义的一种类似的扩展方法来创建路由。 任何 <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 扩展方法都会创建 <xref:Microsoft.AspNetCore.Routing.Route> 的实例并将其添加到路由集合中。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 不接受路由处理程序参数。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 仅添加由 <xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> 处理的路由。 要了解 MVC 中的路由的详细信息，请参阅 <xref:mvc/controllers/routing>。

以下代码示例是典型 ASP.NET Core MVC 路由定义使用的一个 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 调用示例：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

此模板与 URL 路径相匹配，并且提取路由值。 例如，路径 `/Products/Details/17` 生成以下路由值：`{ controller = Products, action = Details, id = 17 }`。

路由值是通过将 URL 路径拆分成段，并且将每段与路由模板中的路由参数名称相匹配来确定的。 路由参数已命名。 参数通过将参数名称括在大括号 `{ ... }` 中来定义。

上述模板还可匹配 URL 路径 `/`，并且生成值 `{ controller = Home, action = Index }`。 这是因为 `{controller}` 和 `{action}` 路由参数具有默认值，且 `id` 路由参数是可选的。 路由参数名称为参数定义默认值后，等号 (`=`) 后将有一个值。 路由参数名称后面的问号 (`?`) 定义了可选参数。

路由匹配时，具有默认值的路由参数始终会生成路由值。 如果没有相应的 URL 路径段，则可选参数不会生成路由值。 有关路由模板方案和语法的详细说明，请参阅[路由模板参考](#route-template-reference)部分。

在以下示例中，路由参数定义 `{id:int}` 为 `id` 路由参数定义[路由约束](#route-constraint-reference)：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

此模板与类似于 `/Products/Details/17` 而不是 `/Products/Details/Apples` 的 URL 路径相匹配。 路由约束实现 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 并检查路由值，以验证它们。 在此示例中，路由值 `id` 必须可转换为整数。 有关框架提供的路由约束的说明，请参阅[路由约束参考](#route-constraint-reference)。

其他 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 重载接受 `constraints`、`dataTokens` 和 `defaults` 的值。 这些参数的典型用法是传递匿名类型化对象，其中匿名类型的属性名匹配路由参数名称。

以下 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 示例可创建等效路由：

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 定义约束和默认值的内联语法对于简单路由很方便。 但是，存在内联语法不支持的方案，例如数据令牌。

以下示例演示了一些其他方案：

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{**article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上述模板与 `/Blog/All-About-Routing/Introduction` 等的 URL 路径相匹配，并提取值 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }`。 `controller` 和 `action` 的默认路由值由路由生成，即便模板中没有对应的路由参数。 可在路由模板中指定默认值。 根据路由参数名称前的双星号 (`**`) 外观，`article` 路由参数被定义为 catch-all。 全方位路由参数可捕获 URL 路径的其余部分，也能匹配空白字符串。

以下示例添加了路由约束和数据令牌：

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上述模板与 `/en-US/Products/5` 等 URL 路径相匹配，并且提取值 `{ controller = Products, action = Details, id = 5 }` 和数据令牌 `{ locale = en-US }`。

![本地 Windows 令牌](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>路由类 URL 生成

<xref:Microsoft.AspNetCore.Routing.Route> 类还可通过将一组路由值与其路由模板组合来生成 URL。 从逻辑上来说，这是匹配 URL 路径的反向过程。

> [!TIP]
> 为更好了解 URL 生成，请想象你要生成的 URL，然后考虑路由模板将如何匹配该 URL。 将生成什么值？ 这大致相当于 URL 在 <xref:Microsoft.AspNetCore.Routing.Route> 类中的生成方式。

以下示例使用常规 ASP.NET Core MVC 默认路由：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

使用路由值 `{ controller = Products, action = List }`，将生成 URL `/Products/List`。 路由值将替换为相应的路由参数，以形成 URL 路径。 由于 `id` 是可选路由参数，因此成功生成的 URL 不具有 `id` 的值。

使用路由值 `{ controller = Home, action = Index }`，将生成 URL `/`。 提供的路由值与默认值匹配，并且会安全地省略与默认值对应的段。

已生成的两个 URL 将往返以下路由定义 (`/Home/Index` 和 `/`)，并生成用于生成该 URL 的相同路由值。

> [!NOTE]
> 使用 ASP.NET Core MVC 应用应该使用 <xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> 生成 URL，而不是直接调用到路由。

有关 URL 生成的详细信息，请参阅 [Url 生成参考](#url-generation-reference)部分。

## <a name="use-routing-middleware"></a>使用路由中间件

引用应用项目文件中的 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)。

将路由添加到 `Startup.ConfigureServices` 中的服务容器：

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

必须在 `Startup.Configure` 方法中配置路由。 该示例应用使用以下 API：

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>：仅匹配 HTTP GET 请求。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下表显示了具有给定 URI 的响应。

| URI                    | 响应                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 请求失败，不匹配。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 请求失败，仅匹配 HTTP GET。 |
| `GET /hello/Joe/Smith` | 请求失败，不匹配。              |

框架可提供一组用于创建路由 (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) 的扩展方法：

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

`Map[Verb]` 方法将使用约束来将路由限制为方法名称中的 HTTP 谓词。 有关示例，请参阅 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> 和 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>。

## <a name="route-template-reference"></a>路由模板参考

如果路由找到匹配项，大括号 (`{ ... }`) 内的令牌定义绑定的路由参数。 可在路由段中定义多个路由参数，但必须由文本值隔开。 例如，`{controller=Home}{action=Index}` 不是有效的路由，因为 `{controller}` 和 `{action}` 之间没有文本值。 这些路由参数必须具有名称，且可能指定了其他属性。

路由参数以外的文本（例如 `{id}`）和路径分隔符 `/` 必须匹配 URL 中的文本。 文本匹配区分大小写，并且基于 URL 路径已解码的表示形式。 要匹配文字路由参数分隔符（`{` 或 `}`），请通过重复该字符（`{{` 或 `}}`）来转义分隔符。

尝试捕获具有可选文件扩展名的文件名的 URL 模式还有其他注意事项。 例如，考虑模板 `files/{filename}.{ext?}`。 当 `filename` 和 `ext` 的值都存在时，将填充这两个值。 如果 URL 中仅存在 `filename` 的值，则路由匹配，因为尾随句点 (`.`) 是可选的。 以下 URL 与此路由相匹配：

* `/files/myFile.txt`
* `/files/myFile`

可以使用星号 (`*`) 或双星号 (`**`) 作为路由参数的前缀，以绑定到 URI 的其余部分。 这些称为 catch-all 参数。 例如，`blog/{**slug}` 将匹配以 `/blog` 开头且其后带有任何值（将分配给 `slug` 路由值）的 URI。 全方位参数还可以匹配空字符串。

使用路由生成 URL（包括路径分隔符 (`/`)）时，catch-all 参数会转义相应的字符。 例如，路由值为 `{ path = "my/path" }` 的路由 `foo/{*path}` 生成 `foo/my%2Fpath`。 请注意转义的正斜杠。 要往返路径分隔符，请使用 `**` 路由参数前缀。 `{ path = "my/path" }` 的路由 `foo/{**path}` 生成 `foo/my/path`。

路由参数可能具有指定的默认值，方法是在参数名称后使用等号 (`=`) 隔开以指定默认值。 例如，`{controller=Home}` 将 `Home` 定义为 `controller` 的默认值。 如果参数的 URL 中不存在任何值，则使用默认值。 通过在参数名称的末尾附加问号 (`?`) 可使路由参数成为可选项，如 `id?` 中所示。 可选值和默认路径参数的区别在于具有默认值的路由参数始终会生成值 &mdash;，而可选参数仅当请求 URL 提供值时才会具有值。

路由参数可能具有必须与从 URL 中绑定的路由值匹配的约束。 在路由参数后面添加一个冒号 (`:`) 和约束名称可指定路由参数上的内联约束。 如果约束需要参数，将以在约束名称后括在括号 (`(...)`) 中的形式提供。 通过追加另一个冒号 (`:`) 和约束名称，可指定多个内联约束。

约束名称和参数将传递给 <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> 服务，以创建 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 的实例，用于处理 URL。 例如，路由模板 `blog/{article:minlength(10)}` 使用参数 `10` 指定 `minlength` 约束。 有关路由约束详情以及框架提供的约束列表，请参阅[路由约束引用](#route-constraint-reference)部分。

路由参数还可以具有参数转换器，用于在生成链接以及将操作和页面匹配到 URI 时转换参数的值。 与约束类似，可以通过在路由参数名称后面添加冒号 (`:`) 和转换器名称，将参数变换器内联添加到路径参数。 例如，路由模板 `blog/{article:slugify}` 指定 `slugify` 转换器。 有关参数转换的详细信息，请参阅[参数转换器参考](#parameter-transformer-reference)部分。

下表演示了示例路由模板及其行为。

| 路由模板                           | 示例匹配 URI    | 请求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 仅匹配单个路径 `/hello`。                                     |
| `{Page=Home}`                            | `/`                     | 匹配并将 `Page` 设置为 `Home`。                                         |
| `{Page=Home}`                            | `/Contact`              | 匹配并将 `Page` 设置为 `Contact`。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | 映射到 `Products` 控制器和 `List` 操作。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | 映射到 `Products` 控制器和 `Details` 操作（`id` 设置为 123）。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | 映射到 `Home` 控制器和 `Index` 方法（忽略 `id`）。        |

使用模板通常是进行路由最简单的方法。 还可在路由模板外指定约束和默认值。

> [!TIP]
> 启用[日志记录](xref:fundamentals/logging/index)以查看内置路由实现（如 <xref:Microsoft.AspNetCore.Routing.Route>）如何匹配请求。

## <a name="reserved-routing-names"></a>保留的路由名称

以下关键字是保留的名称，它们不能用作路由名称或参数：

* `action`
* `area`
* `controller`
* `handler`
* `page`

## <a name="route-constraint-reference"></a>路由约束参考

路由约束在传入 URL 发生匹配时执行，URL 路径标记为路由值。 路径约束通常检查通过路径模板关联的路径值，并对该值是否可接受做出是/否决定。 某些路由约束使用路由值以外的数据来考虑是否可以路由请求。 例如，<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> 可以根据其 HTTP 谓词接受或拒绝请求。 约束用于路由请求和链接生成。

> [!WARNING]
> 请勿将约束用于“输入验证”。 如果将约束用于“输入约束”，那么无效输入将导致“404 - 未找到”响应，而不是含相应错误消息的“400 - 错误请求” 。 路由约束用于消除类似路由的歧义，而不是验证特定路由的输入。

下表演示示例路由约束及其预期行为。

| 约束 | 示例 | 匹配项示例 | 说明 |
|------------|---------|-----------------|-------|
| `int` | `{id:int}` | `123456789`, `-123456789` | 匹配任何整数。|
| `bool` | `{active:bool}` | `true`, `FALSE` | 匹配 `true` 或 `false`。 不区分大小写。|
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | 在固定区域性中匹配有效的 `DateTime` 值。 请参阅前面的警告。|
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | 在固定区域性中匹配有效的 `decimal` 值。 请参阅前面的警告。|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | 在固定区域性中匹配有效的 `double` 值。 请参阅前面的警告。|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | 在固定区域性中匹配有效的 `float` 值。 请参阅前面的警告。|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 匹配有效的 `Guid` 值。|
| `long` | `{ticks:long}` | `123456789`, `-123456789` | 匹配有效的 `long` 值。|
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 字符串必须至少为 4 个字符。|
| `maxlength(value)` | `{filename:maxlength(8)}` | `MyFile` | 字符串最多包含 8 个字符。|
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 字符串必须正好为 12 个字符。|
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 字符串必须至少为 8 个字符，且最多包含 16 个字符。|
| `min(value)` | `{age:min(18)}` | `19` | 整数值必须至少为 18 个字符。|
| `max(value)` | `{age:max(120)}` | `91` | 整数值最多包含 120 个字符。|
| `range(min,max)` | `{age:range(18,120)}` | `91` | 整数值必须至少为 18 个字符，且最多包含 120 个字符。|
| `alpha` | `{name:alpha}` | `Rick` | 字符串必须由一个或多个字母字符组成，`a`-`z`。 不区分大小写。|
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 字符串必须与正则表达式匹配。 请参阅有关定义正则表达式的提示。|
| `required` | `{name:required}` | `Rick` | 用于强制在 URL 生成过程中存在非参数值。|

可向单个参数应用多个由冒号分隔的约束。 例如，以下约束将参数限制为大于或等于 1 的整数值：

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> 验证 URL 的路由约束并将转换为始终使用固定区域性的 CLR 类型（例如 `int` 或 `DateTime`）。 这些约束假定 URL 不可本地化。 框架提供的路由约束不会修改存储于路由值中的值。 从 URL 中分析的所有路由值都将存储为字符串。 例如，`float` 约束会尝试将路由值转换为浮点数，但转换后的值仅用来验证其是否可转换为浮点数。

## <a name="regular-expressions"></a>正则表达式

ASP.NET Core 框架将向正则表达式构造函数添加 `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant`。 有关这些成员的说明，请参阅 <xref:System.Text.RegularExpressions.RegexOptions>。

正则表达式与路由和 C# 语言使用的分隔符和令牌相似。 必须对正则表达式令牌进行转义。 在路由中使用正则表达式 `^\d{3}-\d{2}-\d{4}$`：

* 表达式必须将字符串中提供的单反斜杠 `\` 字符作为源代码中的双反斜杠 `\\` 字符。
* 正则表达式必须使用 `\\` 对 `\` 字符串转义字符进行转义。
* 使用[逐字字符串文本](/dotnet/csharp/language-reference/keywords/string)时，正则表达式不需要 `\\`。

要对路由参数分隔符 `{`、`}`、`[`、`]` 进行转义，请将表达式 `{{`、`}`、`[[`、`]]` 中的字符数加倍。 下表展示了正则表达式和转义版本：

| 正则表达式    | 转义后的正则表达式     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

路由中使用的正则表达式通常以脱字号 `^` 开头，并匹配字符串的起始位置。 表达式通常以美元符号 `$` 字符结尾，并匹配字符串的结尾。 `^` 和 `$` 字符可确保正则表达式匹配整个路由参数值。 如果没有 `^` 和 `$` 字符，正则表达式将匹配字符串内的所有子字符串，而这通常是不需要的。 下表提供了示例并说明了它们匹配或匹配失败的原因。

| 表达式   | String    | 匹配 | 注释               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | 是   | 子字符串匹配     |
| `[a-z]{2}`   | 123abc456 | 是   | 子字符串匹配     |
| `[a-z]{2}`   | mz        | 是   | 匹配表达式    |
| `[a-z]{2}`   | MZ        | 是   | 不区分大小写    |
| `^[a-z]{2}$` | hello     | 否    | 参阅上述 `^` 和 `$` |
| `^[a-z]{2}$` | 123abc456 | 否    | 参阅上述 `^` 和 `$` |

有关正则表达式语法的详细信息，请参阅 [.NET Framework 正则表达式](/dotnet/standard/base-types/regular-expression-language-quick-reference)。

若要将参数限制为一组已知的可能值，可使用正则表达式。 例如，`{action:regex(^(list|get|create)$)}` 仅将 `action` 路由值匹配到 `list`、`get` 或 `create`。 如果传递到约束字典中，字符串 `^(list|get|create)$` 将等效。 已传递到约束字典（不与模板内联）且不匹配任何已知约束的约束还将被视为正则表达式。

## <a name="custom-route-constraints"></a>自定义路由约束

除了内置路由约束以外，还可以通过实现 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 接口来创建自定义路由约束。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 接口包含一个方法 `Match`，当满足约束时，它返回 `true`，否则返回 `false`。

若要使用自定义 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint>，必须在应用的服务容器中使用应用的 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 注册路由约束类型。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 是将路由约束键映射到验证这些约束的 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 实现的目录。 应用的 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 可作为 [services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 调用的一部分在 `Startup.ConfigureServices` 中进行更新，也可以通过使用 `services.Configure<RouteOptions>` 直接配置 <xref:Microsoft.AspNetCore.Routing.RouteOptions> 进行更新。 例如：

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

然后，可以使用在注册约束类型时指定的名称，以常规方式将约束应用于路由。 例如：

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="parameter-transformer-reference"></a>参数转换器参考

参数转换器：

* 在为 <xref:Microsoft.AspNetCore.Routing.Route> 生成链接时执行。
* 实现 `Microsoft.AspNetCore.Routing.IOutboundParameterTransformer`。
* 使用 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 进行配置。
* 获取参数的路由值并将其转换为新的字符串值。
* 在生成的链接中使用转换后的值的结果。

例如，路由模式 `blog\{article:slugify}`（具有 `Url.Action(new { article = "MyTestArticle" })`）中的自定义 `slugify` 参数转换器生成 `blog\my-test-article`。

若要在路由模式中使用参数转换器，请先在 `Startup.ConfigureServices` 中使用 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 对其进行配置：

```csharp
services.AddRouting(options =>
{
    // Replace the type and the name used to refer to it with your own
    // IOutboundParameterTransformer implementation
    options.ConstraintMap["slugify"] = typeof(SlugifyParameterTransformer);
});
```

框架使用参数转化器来转换进行终结点解析的 URI。 例如，ASP.NET Core MVC 使用参数转换器来转换用于匹配 `area``controller``action` 和 `page` 的路由值。

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller:slugify=Home}/{action:slugify=Index}/{id?}");
```

使用上述路由，操作 `SubscriptionManagementController.GetAll` 与 URI `/subscription-management/get-all` 匹配。 参数转换器不会更改用于生成链接的路由值。 例如，`Url.Action("GetAll", "SubscriptionManagement")` 输出 `/subscription-management/get-all`。

对于结合使用参数转换器和所生成的路由，ASP.NET Core 提供了 API 约定：

* ASP.NET Core MVC 还具有 `Microsoft.AspNetCore.Mvc.ApplicationModels.RouteTokenTransformerConvention` API 约定。 该约定将指定的参数转换器应用于应用中的所有属性路由。 在替换属性路径令牌时，参数转换器将转换这些令牌。 有关详细信息，请参阅[使用参数转换器自定义标记替换](xref:mvc/controllers/routing#use-a-parameter-transformer-to-customize-token-replacement)。
* Razor Pages 具有 `Microsoft.AspNetCore.Mvc.ApplicationModels.PageRouteTransformerConvention` API 约定。 此约定将指定的参数转换器应用于所有自动发现的 Razor Pages。 参数转换器转换 Razor Pages 路由的文件夹和文件名段。 有关详细信息，请参阅[使用参数转换器自定义页面路由](xref:razor-pages/razor-pages-conventions#use-a-parameter-transformer-to-customize-page-routes)。

## <a name="url-generation-reference"></a>URL 生成参考

以下示例演示如何在给定路由值字典和 <xref:Microsoft.AspNetCore.Routing.RouteCollection> 的情况下生成路由链接。

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上述示例末尾生成的 <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> 为 `/package/create/123`。 字典提供“跟踪包路由”模板 `package/{operation}/{id}` 的 `operation` 和 `id` 路由值。 有关详细信息，请参阅[使用路由中间件](#use-routing-middleware)部分或[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)中的示例代码。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> 构造函数的第二个参数是环境值的集合。 由于环境值限制了开发人员在请求上下文中必须指定的值数，因此环境值使用起来很方便。 当前请求的当前路由值被视为链接生成的环境值。 在 ASP.NET Core MVC 应用 `HomeController` 的 `About` 操作中，无需指定控制器路由值即可链接到使用 `Home` 环境值的 `Index` 操作&mdash;。

忽略与参数不匹配的环境值。 在显式提供的值会覆盖环境值的情况下，也会忽略环境值。 在 URL 中将从左到右进行匹配。

显式提供但与路由片段不匹配的值将添加到查询字符串中。 下表显示使用路由模板 `{controller}/{action}/{id?}` 时的结果。

| 环境值                     | 显式值                        | 结果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| 控制器 =“Home”                | 操作 =“About”                       | `/Home/About`           |
| 控制器 =“Home”                | 控制器 =“Order”，操作 =“About” | `/Order/About`          |
| 控制器 = “Home”，颜色 = “Red” | 操作 =“About”                       | `/Home/About`           |
| 控制器 =“Home”                | 操作 =“About”，颜色 =“Red”        | `/Home/About?color=Red` |

如果路由具有不对应于参数的默认值，且该值以显式方式提供，则它必须与默认值相匹配：

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

当提供 `controller` 和 `action` 的匹配值时，链接生成仅为此路由生成链接。

## <a name="complex-segments"></a>复杂段

复杂段（例如，`[Route("/x{token}y")]`）通过非贪婪的方式从右到左匹配文字进行处理。 请参阅[此代码](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)以了解有关如何匹配复杂段的详细说明。 ASP.NET Core 无法使用[代码示例](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)，但它提供了对复杂段的合理说明。
<!-- While that code is no longer used by ASP.NET Core for complex segment matching, it provides a good match to the current algorithm. The [current code](https://github.com/dotnet/AspNetCore/blob/91514c9af7e0f4c44029b51f05a01c6fe4c96e4c/src/Http/Routing/src/Matching/DfaMatcherBuilder.cs#L227-L244) is too abstracted from matching to be useful for understanding complex segment matching.
-->

::: moniker-end

::: moniker range="< aspnetcore-2.2"

路由负责将请求 URI 映射到路由处理程序和调度传入的请求。 路由在应用中定义，并在应用启动时进行配置。 路由可以选择从请求包含的 URL 中提取值，然后这些值便可用于处理请求。 如果使用应用中配置的路由，路由能生成映射到路由处理程序的 URL。

要在 ASP.NET Core 2.1 中使用最新路由方案，请在 `Startup.ConfigureServices` 中为 MVC 服务注册指定[兼容性版本](xref:mvc/compatibility-version)：

```csharp
services.AddMvc()
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
```

> [!IMPORTANT]
> 本文档介绍较低级别的 ASP.NET Core 路由。 有关 ASP.NET Core MVC 路由的信息，请参阅 <xref:mvc/controllers/routing>。 有关 Razor Pages 中路由约定的信息，请参阅 <xref:razor-pages/razor-pages-conventions>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="routing-basics"></a>路由基础知识

大多数应用应选择基本的描述性路由方案，让 URL 有可读性和意义。 默认传统路由 `{controller=Home}/{action=Index}/{id?}`：

* 支持基本的描述性路由方案。
* 是基于 UI 的应用的有用起点。

开发人员通常在专业情况下（例如，博客和电子商务终结点）使用[属性路由](xref:mvc/controllers/routing#attribute-routing)或专用传统路由向应用的高流量区域添加其他简洁路由。

Web API 应使用属性路由，将应用功能建模为一组资源，其中操作是由 HTTP 谓词表示。 也就是说，对同一逻辑资源执行的许多操作（例如，GET、POST）都将使用相同 URL。 属性路由提供了精心设计 API 的公共终结点布局所需的控制级别。

Razor Pages 应用使用默认的传统路由从应用的“页面”文件夹中提供命名资源。 还可以使用其他约定来自定义 Razor Pages 路由行为。 有关详细信息，请参阅 <xref:razor-pages/index> 和 <xref:razor-pages/razor-pages-conventions>。

借助 URL 生成支持，无需通过硬编码 URL 将应用关联到一起，即可开发应用。 此支持允许从基本路由配置入手，并在确定应用的资源布局后修改路由。

路由使用 <xref:Microsoft.AspNetCore.Routing.IRouter> 的路由实现来执行以下操作：

* 将传入请求映射到路由处理程序。
* 生成响应中使用的 URL。

默认情况下，应用只有一个路由集合。 当请求到达时，将按照路由在集合中的存在顺序来处理路由。 框架试图通过在集合中的每个路由上调用 <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> 方法，将传入请求的 URL 与集合中的路由进行匹配。 响应可根据路由信息使用路由生成 URL（例如，用于重定向或链接），从而避免硬编码 URL，这有助于可维护性。

路由系统具有以下特征：

* 路由模板语法用于通过标记化路由参数来定义路由。
* 可以使用常规样式和属性样式终结点配置。
* <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 用于确定 URL 参数是否包含给定的终结点约束的有效值。
* 应用模型（如 MVC/Razor Pages）注册了其所有具有可预测的路由方案实现的路由。
* 响应可根据路由信息使用路由生成 URL（例如，用于重定向或链接），从而避免硬编码 URL，这有助于可维护性。
* URL 生成基于支持任意可扩展性的路由。 <xref:Microsoft.AspNetCore.Mvc.IUrlHelper> 提供生成 URL 的方法。
<!-- fix [middleware](xref:fundamentals/middleware/index) -->
路由通过 <xref:Microsoft.AspNetCore.Builder.RouterMiddleware> 类连接到[中间件](xref:fundamentals/middleware/index)管道。 [ASP.NET Core MVC](xref:mvc/overview) 向中间件管道添加路由，作为其配置的一部分，并处理 MVC 和 Razor Pages 应用中的路由。 要了解如何将路由用作独立组件，请参阅[使用路由中间件](#use-routing-middleware)部分。

### <a name="url-matching"></a>URL 匹配

URL 匹配是一个过程，通过该过程，路由可向处理程序调度传入请求。 此过程基于 URL 路径中的数据，但可以进行扩展以考虑请求中的任何数据。 向单独的处理程序调度请求的功能是缩放应用的大小和复杂性的关键。

传入请求将进入 <xref:Microsoft.AspNetCore.Builder.RouterMiddleware>，后者将对序列中的每个路由调用 <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> 方法。 <xref:Microsoft.AspNetCore.Routing.IRouter> 实例将选择是否通过将 [RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler*) 设置为非 NULL <xref:Microsoft.AspNetCore.Http.RequestDelegate> 来处理请求。 如果路由为请求设置处理程序，将停止路由处理，并调用处理程序来处理该请求。 如果未找到用于处理请求的路由处理程序，中间件会将请求传递给请求管道中的下一个中间件。

<xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> 的主要输入是与当前请求关联的 [RouteContext.HttpContext](xref:Microsoft.AspNetCore.Routing.RouteContext.HttpContext*)。 [RouteContext.Handler](xref:Microsoft.AspNetCore.Routing.RouteContext.Handler) 和 [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData*) 是路由匹配后设置的输出。

调用 <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> 的匹配还可根据迄今执行的请求处理将 [RouteContext.RouteData](xref:Microsoft.AspNetCore.Routing.RouteContext.RouteData) 的属性设为适当的值。

[RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values*) 是从路由中生成的路由值的字典。 这些值通常通过标记 URL 来确定，可用来接受用户输入，或者在应用内作出进一步的调度决策。

[RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) 是一个与匹配的路由相关的其他数据的属性包。 提供 <xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*> 以支持将状态数据与每个路由相关联，以便应用可根据所匹配的路由作出决策。 这些值是开发者定义的，不会影响通过任何方式路由的行为。 此外，存储于 [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) 中的值可以属于任何类型，与 [RouteData.Values](xref:Microsoft.AspNetCore.Routing.RouteData.Values) 相反，后者必须能够转换为字符串，或从字符串进行转换。

[RouteData.Routers](xref:Microsoft.AspNetCore.Routing.RouteData.Routers) 是参与成功匹配请求的路由的列表。 路由可以相互嵌套。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> 属性可以通过导致匹配的逻辑路由树反映该路径。 通常情况下，<xref:Microsoft.AspNetCore.Routing.RouteData.Routers> 中的第一项是路由集合，应该用于生成 URL。 <xref:Microsoft.AspNetCore.Routing.RouteData.Routers> 中的最后一项是匹配的路由处理程序。

<a name="lg"></a>

### <a name="url-generation"></a>URL 生成

URL 生成是通过其可根据一组路由值创建 URL 路径的过程。 这需要考虑处理程序与访问它们的 URL 之间的逻辑分隔。

URL 遵循类似的迭代过程，但开头是将用户或框架代码调用到路由集合的 <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> 方法中。 每个路由按顺序调用其 <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> 方法，直到返回非 NULL 的 <xref:Microsoft.AspNetCore.Routing.VirtualPathData>。

<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> 的主输入有：

* [VirtualPathContext.HttpContext](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext)
* [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values)
* [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues)

路由主要使用 <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> 和 <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> 提供的路由值来确定是否可能生成 URL 以及要包括哪些值。 <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues> 是通过匹配当前请求而生成的路由值集。 与此相反，<xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values> 是指定如何为当前操作生成所需 URL 的路由值。 如果路由应获取与当前上下文关联的服务或其他数据时，则提供 <xref:Microsoft.AspNetCore.Routing.VirtualPathContext.HttpContext>。

> [!TIP]
> 将 [VirtualPathContext.Values](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.Values*) 视为 [VirtualPathContext.AmbientValues](xref:Microsoft.AspNetCore.Routing.VirtualPathContext.AmbientValues*) 的一组替代。 URL 生成尝试重复使用当前请求中的路由值，以便使用相同路由或路由值生成链接的 URL。

<xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> 的输出是 <xref:Microsoft.AspNetCore.Routing.VirtualPathData>。 <xref:Microsoft.AspNetCore.Routing.VirtualPathData> 是 <xref:Microsoft.AspNetCore.Routing.RouteData> 的并行值。 <xref:Microsoft.AspNetCore.Routing.VirtualPathData> 包含输出 URL 的 <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> 以及路由应该设置的某些其他属性。

[VirtualPathData.VirtualPath](xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath*) 属性包含路由生成的虚拟路径。 你可能需要进一步处理路径，具体取决于你的需求。 如果要在 HTML 中呈现生成的 URL，请预置应用的基路径。

[VirtualPathData.Router](xref:Microsoft.AspNetCore.Routing.VirtualPathData.Router*) 是对已成功生成 URL 的路由的引用。

[VirtualPathData.DataTokens](xref:Microsoft.AspNetCore.Routing.VirtualPathData.DataTokens*) 属性是生成 URL 的路由的其他相关数据的字典。 这是 [RouteData.DataTokens](xref:Microsoft.AspNetCore.Routing.RouteData.DataTokens*) 的并性值。

### <a name="create-routes"></a>创建路由

路由提供 <xref:Microsoft.AspNetCore.Routing.Route> 类，作为 <xref:Microsoft.AspNetCore.Routing.IRouter> 的标准实现。 <xref:Microsoft.AspNetCore.Routing.Route> 使用 route template 语法来定义模式，以便在调用 <xref:Microsoft.AspNetCore.Routing.IRouter.RouteAsync*> 时匹配 URL 路径。 调用 <xref:Microsoft.AspNetCore.Routing.IRouter.GetVirtualPath*> 时，<xref:Microsoft.AspNetCore.Routing.Route> 使用同一路由模板生成 URL。

大多数应用通过调用 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 或 <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 上定义的一种类似的扩展方法来创建路由。 任何 <xref:Microsoft.AspNetCore.Routing.IRouteBuilder> 扩展方法都会创建 <xref:Microsoft.AspNetCore.Routing.Route> 的实例并将其添加到路由集合中。

<xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 不接受路由处理程序参数。 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 仅添加由 <xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*> 处理的路由。 默认处理程序是 `IRouter`，该处理程序可能无法处理该请求。 例如，ASP.NET Core MVC 通常被配置为默认处理程序，仅处理与可用控制器和操作匹配的请求。 要了解 MVC 中的路由的详细信息，请参阅 <xref:mvc/controllers/routing>。

以下代码示例是典型 ASP.NET Core MVC 路由定义使用的一个 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 调用示例：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

此模板与 URL 路径相匹配，并且提取路由值。 例如，路径 `/Products/Details/17` 生成以下路由值：`{ controller = Products, action = Details, id = 17 }`。

路由值是通过将 URL 路径拆分成段，并且将每段与路由模板中的路由参数名称相匹配来确定的。 路由参数已命名。 参数通过将参数名称括在大括号 `{ ... }` 中来定义。

上述模板还可匹配 URL 路径 `/`，并且生成值 `{ controller = Home, action = Index }`。 这是因为 `{controller}` 和 `{action}` 路由参数具有默认值，且 `id` 路由参数是可选的。 路由参数名称为参数定义默认值后，等号 (`=`) 后将有一个值。 路由参数名称后面的问号 (`?`) 定义了可选参数。

路由匹配时，具有默认值的路由参数始终会生成路由值。 如果没有相应的 URL 路径段，则可选参数不会生成路由值。 有关路由模板方案和语法的详细说明，请参阅[路由模板参考](#route-template-reference)部分。

在以下示例中，路由参数定义 `{id:int}` 为 `id` 路由参数定义[路由约束](#route-constraint-reference)：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id:int}");
```

此模板与类似于 `/Products/Details/17` 而不是 `/Products/Details/Apples` 的 URL 路径相匹配。 路由约束实现 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 并检查路由值，以验证它们。 在此示例中，路由值 `id` 必须可转换为整数。 有关框架提供的路由约束的说明，请参阅[路由约束参考](#route-constraint-reference)。

其他 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 重载接受 `constraints`、`dataTokens` 和 `defaults` 的值。 这些参数的典型用法是传递匿名类型化对象，其中匿名类型的属性名匹配路由参数名称。

以下 <xref:Microsoft.AspNetCore.Builder.MapRouteRouteBuilderExtensions.MapRoute*> 示例可创建等效路由：

```csharp
routes.MapRoute(
    name: "default_route",
    template: "{controller}/{action}/{id?}",
    defaults: new { controller = "Home", action = "Index" });

routes.MapRoute(
    name: "default_route",
    template: "{controller=Home}/{action=Index}/{id?}");
```

> [!TIP]
> 定义约束和默认值的内联语法对于简单路由很方便。 但是，存在内联语法不支持的方案，例如数据令牌。

以下示例演示了一些其他方案：

```csharp
routes.MapRoute(
    name: "blog",
    template: "Blog/{*article}",
    defaults: new { controller = "Blog", action = "ReadArticle" });
```

上述模板与 `/Blog/All-About-Routing/Introduction` 等的 URL 路径相匹配，并提取值 `{ controller = Blog, action = ReadArticle, article = All-About-Routing/Introduction }`。 `controller` 和 `action` 的默认路由值由路由生成，即便模板中没有对应的路由参数。 可在路由模板中指定默认值。 根据路由参数名称前的星号 (`*`) 外观，`article` 路由参数被定义为 catch-all。 全方位路由参数可捕获 URL 路径的其余部分，也能匹配空白字符串。

以下示例添加了路由约束和数据令牌：

```csharp
routes.MapRoute(
    name: "us_english_products",
    template: "en-US/Products/{id}",
    defaults: new { controller = "Products", action = "Details" },
    constraints: new { id = new IntRouteConstraint() },
    dataTokens: new { locale = "en-US" });
```

上述模板与 `/en-US/Products/5` 等 URL 路径相匹配，并且提取值 `{ controller = Products, action = Details, id = 5 }` 和数据令牌 `{ locale = en-US }`。

![本地 Windows 令牌](routing/_static/tokens.png)

### <a name="route-class-url-generation"></a>路由类 URL 生成

<xref:Microsoft.AspNetCore.Routing.Route> 类还可通过将一组路由值与其路由模板组合来生成 URL。 从逻辑上来说，这是匹配 URL 路径的反向过程。

> [!TIP]
> 为更好了解 URL 生成，请想象你要生成的 URL，然后考虑路由模板将如何匹配该 URL。 将生成什么值？ 这大致相当于 URL 在 <xref:Microsoft.AspNetCore.Routing.Route> 类中的生成方式。

以下示例使用常规 ASP.NET Core MVC 默认路由：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{id?}");
```

使用路由值 `{ controller = Products, action = List }`，将生成 URL `/Products/List`。 路由值将替换为相应的路由参数，以形成 URL 路径。 由于 `id` 是可选路由参数，因此成功生成的 URL 不具有 `id` 的值。

使用路由值 `{ controller = Home, action = Index }`，将生成 URL `/`。 提供的路由值与默认值匹配，并且会安全地省略与默认值对应的段。

已生成的两个 URL 将往返以下路由定义 (`/Home/Index` 和 `/`)，并生成用于生成该 URL 的相同路由值。

> [!NOTE]
> 使用 ASP.NET Core MVC 应用应该使用 <xref:Microsoft.AspNetCore.Mvc.Routing.UrlHelper> 生成 URL，而不是直接调用到路由。

有关 URL 生成的详细信息，请参阅 [Url 生成参考](#url-generation-reference)部分。

## <a name="use-routing-middleware"></a>使用路由中间件

引用应用项目文件中的 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)。

将路由添加到 `Startup.ConfigureServices` 中的服务容器：

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_ConfigureServices&highlight=3)]

必须在 `Startup.Configure` 方法中配置路由。 该示例应用使用以下 API：

* <xref:Microsoft.AspNetCore.Routing.RouteBuilder>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>：仅匹配 HTTP GET 请求。
* <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*>

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_RouteHandler)]

下表显示了具有给定 URI 的响应。

| URI                    | 响应                                          |
| ---------------------- | ------------------------------------------------- |
| `/package/create/3`    | Hello! Route values: [operation, create], [id, 3] |
| `/package/track/-3`    | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/-3/`   | Hello! Route values: [operation, track], [id, -3] |
| `/package/track/`      | 请求失败，不匹配。              |
| `GET /hello/Joe`       | Hi, Joe!                                          |
| `POST /hello/Joe`      | 请求失败，仅匹配 HTTP GET。 |
| `GET /hello/Joe/Smith` | 请求失败，不匹配。              |

如果要配置单个路由，请在 `IRouter` 实例中调用 <xref:Microsoft.AspNetCore.Builder.RoutingBuilderExtensions.UseRouter*> 传入。 无需使用 <xref:Microsoft.AspNetCore.Routing.RouteBuilder>。

框架可提供一组用于创建路由 (<xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions>) 的扩展方法：

* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareDelete*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareGet*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewarePut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapMiddlewareVerb*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPost*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapPut*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>
* <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>

一些列出的方法（如 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*>）需要 <xref:Microsoft.AspNetCore.Http.RequestDelegate>。 路由匹配时，<xref:Microsoft.AspNetCore.Http.RequestDelegate> 会用作路由处理程序。 此系列中的其他方法允许配置中间件管道，将其用作路由处理程序。 如果 `Map*` 方法不接受处理程序（例如 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapRoute*>），则它将使用 <xref:Microsoft.AspNetCore.Routing.RouteBuilder.DefaultHandler*>。

`Map[Verb]` 方法将使用约束来将路由限制为方法名称中的 HTTP 谓词。 有关示例，请参阅 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapGet*> 和 <xref:Microsoft.AspNetCore.Routing.RequestDelegateRouteBuilderExtensions.MapVerb*>。

## <a name="route-template-reference"></a>路由模板参考

如果路由找到匹配项，大括号 (`{ ... }`) 内的令牌定义绑定的路由参数。 可在路由段中定义多个路由参数，但必须由文本值隔开。 例如，`{controller=Home}{action=Index}` 不是有效的路由，因为 `{controller}` 和 `{action}` 之间没有文本值。 这些路由参数必须具有名称，且可能指定了其他属性。

路由参数以外的文本（例如 `{id}`）和路径分隔符 `/` 必须匹配 URL 中的文本。 文本匹配区分大小写，并且基于 URL 路径已解码的表示形式。 要匹配文字路由参数分隔符（`{` 或 `}`），请通过重复该字符（`{{` 或 `}}`）来转义分隔符。

尝试捕获具有可选文件扩展名的文件名的 URL 模式还有其他注意事项。 例如，考虑模板 `files/{filename}.{ext?}`。 当 `filename` 和 `ext` 的值都存在时，将填充这两个值。 如果 URL 中仅存在 `filename` 的值，则路由匹配，因为尾随句点 (`.`) 是可选的。 以下 URL 与此路由相匹配：

* `/files/myFile.txt`
* `/files/myFile`

可以使用星号 (`*`) 作为路由参数的前缀，以绑定到 URI 的其余部分。 这称为 catch-all 参数。 例如，`blog/{*slug}` 将匹配以 `/blog` 开头且其后带有任何值（将分配给 `slug` 路由值）的 URI。 全方位参数还可以匹配空字符串。

使用路由生成 URL（包括路径分隔符 (`/`)）时，catch-all 参数会转义相应的字符。 例如，路由值为 `{ path = "my/path" }` 的路由 `foo/{*path}` 生成 `foo/my%2Fpath`。 请注意转义的正斜杠。

路由参数可能具有指定的默认值，方法是在参数名称后使用等号 (`=`) 隔开以指定默认值。 例如，`{controller=Home}` 将 `Home` 定义为 `controller` 的默认值。 如果参数的 URL 中不存在任何值，则使用默认值。 通过在参数名称的末尾附加问号 (`?`) 可使路由参数成为可选项，如 `id?` 中所示。 可选值和默认路径参数的区别在于具有默认值的路由参数始终会生成值 &mdash;，而可选参数仅当请求 URL 提供值时才会具有值。

路由参数可能具有必须与从 URL 中绑定的路由值匹配的约束。 在路由参数后面添加一个冒号 (`:`) 和约束名称可指定路由参数上的内联约束。 如果约束需要参数，将以在约束名称后括在括号 (`(...)`) 中的形式提供。 通过追加另一个冒号 (`:`) 和约束名称，可指定多个内联约束。

约束名称和参数将传递给 <xref:Microsoft.AspNetCore.Routing.IInlineConstraintResolver> 服务，以创建 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 的实例，用于处理 URL。 例如，路由模板 `blog/{article:minlength(10)}` 使用参数 `10` 指定 `minlength` 约束。 有关路由约束详情以及框架提供的约束列表，请参阅[路由约束引用](#route-constraint-reference)部分。

下表演示了示例路由模板及其行为。

| 路由模板                           | 示例匹配 URI    | 请求 URI&hellip;                                                    |
| ---------------------------------------- | ----------------------- | -------------------------------------------------------------------------- |
| `hello`                                  | `/hello`                | 仅匹配单个路径 `/hello`。                                     |
| `{Page=Home}`                            | `/`                     | 匹配并将 `Page` 设置为 `Home`。                                         |
| `{Page=Home}`                            | `/Contact`              | 匹配并将 `Page` 设置为 `Contact`。                                      |
| `{controller}/{action}/{id?}`            | `/Products/List`        | 映射到 `Products` 控制器和 `List` 操作。                       |
| `{controller}/{action}/{id?}`            | `/Products/Details/123` | 映射到 `Products` 控制器和 `Details` 操作（`id` 设置为 123）。 |
| `{controller=Home}/{action=Index}/{id?}` | `/`                     | 映射到 `Home` 控制器和 `Index` 方法（忽略 `id`）。        |

使用模板通常是进行路由最简单的方法。 还可在路由模板外指定约束和默认值。

> [!TIP]
> 启用[日志记录](xref:fundamentals/logging/index)以查看内置路由实现（如 <xref:Microsoft.AspNetCore.Routing.Route>）如何匹配请求。

## <a name="route-constraint-reference"></a>路由约束参考

路由约束在传入 URL 发生匹配时执行，URL 路径标记为路由值。 路径约束通常检查通过路径模板关联的路径值，并对该值是否可接受做出是/否决定。 某些路由约束使用路由值以外的数据来考虑是否可以路由请求。 例如，<xref:Microsoft.AspNetCore.Routing.Constraints.HttpMethodRouteConstraint> 可以根据其 HTTP 谓词接受或拒绝请求。 约束用于路由请求和链接生成。

> [!WARNING]
> 请勿将约束用于“输入验证”。 如果将约束用于“输入约束”，那么无效输入将导致“404 - 未找到”响应，而不是含相应错误消息的“400 - 错误请求” 。 路由约束用于消除类似路由的歧义，而不是验证特定路由的输入。

下表演示示例路由约束及其预期行为。

| 约束 | 示例 | 匹配项示例 | 说明 |
| ---------- | ------- | --------------- | ----- |
| `int` | `{id:int}` | `123456789`, `-123456789` | 匹配任何整数 |
| `bool` | `{active:bool}` | `true`, `FALSE` | 匹配 `true`或 `false`（区分大小写） |
| `datetime` | `{dob:datetime}` | `2016-12-31`, `2016-12-31 7:32pm` | 在固定区域性中匹配有效的 `DateTime` 值。 请参阅前面的警告。|
| `decimal` | `{price:decimal}` | `49.99`, `-1,000.01` | 在固定区域性中匹配有效的 `decimal` 值。 请参阅前面的警告。|
| `double` | `{weight:double}` | `1.234`, `-1,001.01e8` | 在固定区域性中匹配有效的 `double` 值。 请参阅前面的警告。|
| `float` | `{weight:float}` | `1.234`, `-1,001.01e8` | 在固定区域性中匹配有效的 `float` 值。 请参阅前面的警告。|
| `guid` | `{id:guid}` | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 匹配有效的 `Guid` 值 |
| `long` | `{ticks:long}` | `123456789`, `-123456789` | 匹配有效的 `long` 值 |
| `minlength(value)` | `{username:minlength(4)}` | `Rick` | 字符串必须至少为 4 个字符 |
| `maxlength(value)` | `{filename:maxlength(8)}` | `Richard` | 字符串不得超过 8 个字符 |
| `length(length)` | `{filename:length(12)}` | `somefile.txt` | 字符串必须正好为 12 个字符 |
| `length(min,max)` | `{filename:length(8,16)}` | `somefile.txt` | 字符串必须至少为 8 个字符，且不得超过 16 个字符 |
| `min(value)` | `{age:min(18)}` | `19` | 整数值必须至少为 18 |
| `max(value)` | `{age:max(120)}` | `91` | 整数值不得超过 120 |
| `range(min,max)` | `{age:range(18,120)}` | `91` | 整数值必须至少为 18，且不得超过 120 |
| `alpha` | `{name:alpha}` | `Rick` | 字符串必须由一个或多个字母字符（`a`-`z`，区分大小写）组成 |
| `regex(expression)` | `{ssn:regex(^\\d{{3}}-\\d{{2}}-\\d{{4}}$)}` | `123-45-6789` | 字符串必须匹配正则表达式（参见有关定义正则表达式的提示） |
| `required` | `{name:required}` | `Rick` | 用于强制在 URL 生成过程中存在非参数值 |

可向单个参数应用多个由冒号分隔的约束。 例如，以下约束将参数限制为大于或等于 1 的整数值：

```csharp
[Route("users/{id:int:min(1)}")]
public User GetUserById(int id) { }
```

> [!WARNING]
> 验证 URL 的路由约束并将转换为始终使用固定区域性的 CLR 类型（例如 `int` 或 `DateTime`）。 这些约束假定 URL 不可本地化。 框架提供的路由约束不会修改存储于路由值中的值。 从 URL 中分析的所有路由值都将存储为字符串。 例如，`float` 约束会尝试将路由值转换为浮点数，但转换后的值仅用来验证其是否可转换为浮点数。

## <a name="regular-expressions"></a>正则表达式

ASP.NET Core 框架将向正则表达式构造函数添加 `RegexOptions.IgnoreCase | RegexOptions.Compiled | RegexOptions.CultureInvariant`。 有关这些成员的说明，请参阅 <xref:System.Text.RegularExpressions.RegexOptions>。

正则表达式与路由和 C# 计算机语言使用的分隔符和令牌相似。 必须对正则表达式令牌进行转义。 要在路由中使用正则表达式 `^\d{3}-\d{2}-\d{4}$`，表达式必须在字符串中提供 `\`（单反斜杠）字符，正如 C# 源文件中的 `\\`（双反斜杠）字符一样，以便对 `\` 字符串转义字符进行转义（除非使用[字符串文本](/dotnet/csharp/language-reference/keywords/string)）。 要对路由参数分隔符进行转义（`{`、`}`、`[`、`]`），请将表达式（`{{`、`}`、`[[`、`]]`）中的字符数加倍。 下表展示了正则表达式和转义版本。

| 正则表达式    | 转义后的正则表达式     |
| --------------------- | ------------------------------ |
| `^\d{3}-\d{2}-\d{4}$` | `^\\d{{3}}-\\d{{2}}-\\d{{4}}$` |
| `^[a-z]{2}$`          | `^[[a-z]]{{2}}$`               |

路由中使用的正则表达式通常以脱字号 (`^`) 开头，并匹配字符串的起始位置。 表达式通常以美元符号 (`$`) 字符结尾，并匹配字符串的结尾。 `^` 和 `$` 字符可确保正则表达式匹配整个路由参数值。 如果没有 `^` 和 `$` 字符，正则表达式将匹配字符串内的所有子字符串，而这通常是不需要的。 下表提供了示例并说明了它们匹配或匹配失败的原因。

| 表达式   | String    | 匹配 | 注释               |
| ------------ | --------- | :---: |  -------------------- |
| `[a-z]{2}`   | hello     | 是   | 子字符串匹配     |
| `[a-z]{2}`   | 123abc456 | 是   | 子字符串匹配     |
| `[a-z]{2}`   | mz        | 是   | 匹配表达式    |
| `[a-z]{2}`   | MZ        | 是   | 不区分大小写    |
| `^[a-z]{2}$` | hello     | 否    | 参阅上述 `^` 和 `$` |
| `^[a-z]{2}$` | 123abc456 | 否    | 参阅上述 `^` 和 `$` |

有关正则表达式语法的详细信息，请参阅 [.NET Framework 正则表达式](/dotnet/standard/base-types/regular-expression-language-quick-reference)。

若要将参数限制为一组已知的可能值，可使用正则表达式。 例如，`{action:regex(^(list|get|create)$)}` 仅将 `action` 路由值匹配到 `list`、`get` 或 `create`。 如果传递到约束字典中，字符串 `^(list|get|create)$` 将等效。 已传递到约束字典（不与模板内联）且不匹配任何已知约束的约束还将被视为正则表达式。

## <a name="custom-route-constraints"></a>自定义路由约束

除了内置路由约束以外，还可以通过实现 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 接口来创建自定义路由约束。 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 接口包含一个方法 `Match`，当满足约束时，它返回 `true`，否则返回 `false`。

若要使用自定义 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint>，必须在应用的服务容器中使用应用的 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 注册路由约束类型。 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 是将路由约束键映射到验证这些约束的 <xref:Microsoft.AspNetCore.Routing.IRouteConstraint> 实现的目录。 应用的 <xref:Microsoft.AspNetCore.Routing.RouteOptions.ConstraintMap> 可作为 [services.AddRouting](xref:Microsoft.Extensions.DependencyInjection.RoutingServiceCollectionExtensions.AddRouting*) 调用的一部分在 `Startup.ConfigureServices` 中进行更新，也可以通过使用 `services.Configure<RouteOptions>` 直接配置 <xref:Microsoft.AspNetCore.Routing.RouteOptions> 进行更新。 例如：

```csharp
services.AddRouting(options =>
{
    options.ConstraintMap.Add("customName", typeof(MyCustomConstraint));
});
```

然后，可以使用在注册约束类型时指定的名称，以常规方式将约束应用于路由。 例如：

```csharp
[HttpGet("{id:customName}")]
public ActionResult<string> Get(string id)
```

## <a name="url-generation-reference"></a>URL 生成参考

以下示例演示如何在给定路由值字典和 <xref:Microsoft.AspNetCore.Routing.RouteCollection> 的情况下生成路由链接。

[!code-csharp[](routing/samples/2.x/RoutingSample/Startup.cs?name=snippet_Dictionary)]

上述示例末尾生成的 <xref:Microsoft.AspNetCore.Routing.VirtualPathData.VirtualPath> 为 `/package/create/123`。 字典提供“跟踪包路由”模板 `package/{operation}/{id}` 的 `operation` 和 `id` 路由值。 有关详细信息，请参阅[使用路由中间件](#use-routing-middleware)部分或[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/routing/samples)中的示例代码。

<xref:Microsoft.AspNetCore.Routing.VirtualPathContext> 构造函数的第二个参数是环境值的集合。 由于环境值限制了开发人员在请求上下文中必须指定的值数，因此环境值使用起来很方便。 当前请求的当前路由值被视为链接生成的环境值。 在 ASP.NET Core MVC 应用 `HomeController` 的 `About` 操作中，无需指定控制器路由值即可链接到使用 `Home` 环境值的 `Index` 操作&mdash;。

忽略与参数不匹配的环境值。 在显式提供的值会覆盖环境值的情况下，也会忽略环境值。 在 URL 中将从左到右进行匹配。

显式提供但与路由片段不匹配的值将添加到查询字符串中。 下表显示使用路由模板 `{controller}/{action}/{id?}` 时的结果。

| 环境值                     | 显式值                        | 结果                  |
| ---------------------------------- | -------------------------------------- | ----------------------- |
| 控制器 =“Home”                | 操作 =“About”                       | `/Home/About`           |
| 控制器 =“Home”                | 控制器 =“Order”，操作 =“About” | `/Order/About`          |
| 控制器 = “Home”，颜色 = “Red” | 操作 =“About”                       | `/Home/About`           |
| 控制器 =“Home”                | 操作 =“About”，颜色 =“Red”        | `/Home/About?color=Red` |

如果路由具有不对应于参数的默认值，且该值以显式方式提供，则它必须与默认值相匹配：

```csharp
routes.MapRoute("blog_route", "blog/{*slug}",
    defaults: new { controller = "Blog", action = "ReadPost" });
```

当提供 `controller` 和 `action` 的匹配值时，链接生成仅为此路由生成链接。

## <a name="complex-segments"></a>复杂段

复杂段（例如，`[Route("/x{token}y")]`）通过非贪婪的方式从右到左匹配文字进行处理。 请参阅[此代码](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)以了解有关如何匹配复杂段的详细说明。 ASP.NET Core 无法使用[代码示例](https://github.com/dotnet/aspnetcore/blob/v2.2.13/src/Http/Routing/src/Patterns/RoutePatternMatcher.cs#L293)，但它提供了对复杂段的合理说明。

::: moniker-end
