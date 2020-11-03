---
title: ASP.NET Core 中的集成测试
author: rick-anderson
description: 了解集成测试如何在基础结构级别（包括数据库、文件系统和网络）确保应用组件功能正常。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/14/2020
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
uid: test/integration-tests
ms.openlocfilehash: 9b36a77730a43c7515fcd2c56621412453784c9d
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722535"
---
# <a name="integration-tests-in-aspnet-core"></a>ASP.NET Core 中的集成测试

作者：[Javier Calvarro Nelson](https://github.com/javiercn)[Steve Smith](https://ardalis.com/) 和 [Jos van der Til](https://jvandertil.nl)

::: moniker range=">= aspnetcore-3.0"

集成测试可在包含应用支持基础结构（如数据库、文件系统和网络）的级别上确保应用组件功能正常。 ASP.NET Core 通过将单元测试框架与测试 Web 主机和内存中测试服务器结合使用来支持集成测试。

本主题假设读者基本了解单元测试。 如果不熟悉测试概念，请参阅 [.NET Core 和 .NET Standard 中的单元测试](/dotnet/core/testing/)主题及其链接内容。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)（[如何下载](xref:index#how-to-download-a-sample)）

示例应用是 Razor Pages 应用，假设读者基本了解 Razor Pages。 如果不熟悉 Razor Pages，请参阅以下主题：

* [Razor Pages 简介](xref:razor-pages/index)
* [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)
* [Razor Pages 单元测试](xref:test/razor-pages-tests)

> [!NOTE]
> 对于测试 SPA，建议使用可以自动执行浏览器的工具，如 [Selenium](https://www.seleniumhq.org/)。

## <a name="introduction-to-integration-tests"></a>集成测试简介

与[单元测试](/dotnet/core/testing/)相比，集成测试可在更广泛的级别上评估应用的组件。 单元测试用于测试独立软件组件，如单独的类方法。 集成测试确认两个或更多应用组件一起工作以生成预期结果，可能包括完整处理请求所需的每个组件。

这些更广泛的测试用于测试应用的基础结构和整个框架，通常包括以下组件：

* 数据库
* 文件系统
* 网络设备
* 请求-响应管道

单元测试使用称为 fake 或 mock 对象的制造组件，而不是基础结构组件。

与单元测试相比，集成测试：

* 使用应用在生产环境中使用的实际组件。
* 需要进行更多代码和数据处理。
* 需要更长时间来运行。

因此，将集成测试的使用限制为最重要的基础结构方案。 如果可以使用单元测试或集成测试来测试行为，请选择单元测试。

> [!TIP]
> 请勿为通过数据库和文件系统进行的数据和文件访问的每个可能排列编写集成测试。 无论应用中有多少位置与数据库和文件系统交互，一组集中式读取、写入、更新和删除集成测试通常能够充分测试数据库和文件系统组件。 将单元测试用于与这些组件交互的方法逻辑的例程测试。 在单元测试中，使用基础结构 fake/mock 会导致更快地执行测试。

> [!NOTE]
> 在集成测试的讨论中，测试的项目经常称为“测试中的系统”，简称“SUT”。
>
> 本主题中使用“SUT”来指代测试的 ASP.NET Core 应用。

## <a name="aspnet-core-integration-tests"></a>ASP.NET Core 集成测试

ASP.NET Core 中的集成测试需要以下内容：

* 测试项目用于包含和执行测试。 测试项目具有对 SUT 的引用。
* 测试项目为 SUT 创建测试 Web 主机，并使用测试服务器客户端处理 SUT 的请求和响应。
* 测试运行程序用于执行测试并报告测试结果。

集成测试后跟一系列事件，包括常规“排列”、“操作”和“断言”测试步骤：

1. 已配置 SUT 的 Web 主机。
1. 创建测试服务器客户端以向应用提交请求。
1. 执行“排列”测试步骤：测试应用会准备请求。
1. 执行“操作”测试步骤：客户端提交请求并接收响应。
1. 执行“断言”测试步骤：实际响应基于预期响应验证为通过或失败。
1. 该过程会一直继续，直到执行了所有测试。
1. 报告测试结果。

通常，测试 Web 主机的配置与用于测试运行的应用常规 Web 主机不同。 例如，可以将不同的数据库或不同的应用设置用于测试。

基础结构组件（如测试 Web 主机和内存中测试服务器 ([TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver))）由 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) 包提供或管理。 使用此包可简化测试创建和执行。

`Microsoft.AspNetCore.Mvc.Testing` 包处理以下任务：

* 将依赖项文件 (.deps) 从 SUT 复制到测试项目的 bin 目录中。
* 将[内容根目录](xref:fundamentals/index#content-root)设置为 SUT 的项目根目录，以便可在执行测试时找到静态文件和页面/视图。
* 提供 [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) 类，以简化 SUT 在 `TestServer` 中的启动过程。

[单元测试](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)文档介绍如何设置测试项目和测试运行程序，以及有关如何运行测试的详细说明与有关如何命名测试和测试类的建议。

> [!NOTE]
> 为应用创建测试项目时，请将集成测试中的单元测试分隔到不同的项目中。 这可帮助确保不会意外地将基础结构测试组件包含在单元测试中。 通过分隔单元测试和集成测试还可以控制运行的测试集。

Razor Pages 应用与 MVC 应用的测试配置之间几乎没有任何区别。 唯一的区别在于测试的命名方式。 在 Razor Pages 应用中，页面终结点的测试通常以页面模型类命名（例如，`IndexPageTests` 用于为索引页面测试组件集成）。 在 MVC 应用中，测试通常按控制器类进行组织，并以它们所测试的控制器来命令（例如 `HomeControllerTests` 用于为主页控制器测试组件集成）。

## <a name="test-app-prerequisites"></a>测试应用先决条件

测试项目必须：

* 引用 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing)包。
* 在项目文件中指定 Web SDK (`<Project Sdk="Microsoft.NET.Sdk.Web">`)。

可以在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)中查看这些先决条件。 检查 tests/RazorPagesProject.Tests/RazorPagesProject.Tests.csproj 文件。 示例应用使用 [xUnit](https://xunit.github.io/) 测试框架和 [AngleSharp](https://anglesharp.github.io/) 分析程序库，因此示例应用还引用：

* [xunit](https://www.nuget.org/packages/xunit)
* [xunit.runner.visualstudio](https://www.nuget.org/packages/xunit.runner.visualstudio)
* [AngleSharp](https://www.nuget.org/packages/AngleSharp)

还在测试中使用了 Entity Framework Core。 应用引用：

* [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore)
* [Microsoft.AspNetCore.Identity.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity.EntityFrameworkCore)
* [Microsoft.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore)
* [Microsoft.EntityFrameworkCore.InMemory](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.InMemory)
* [Microsoft.EntityFrameworkCore.Tools](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.Tools)

## <a name="sut-environment"></a>SUT 环境

如果未设置 SUT 的[环境](xref:fundamentals/environments)，则环境会默认为“开发”。

## <a name="basic-tests-with-the-default-webapplicationfactory"></a>使用默认 WebApplicationFactory 的基本测试

[WebApplicationFactory\<TEntryPoint>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) 用于为集成测试创建 [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)。 `TEntryPoint` 是 SUT 的入口点类，通常是 `Startup` 类。

测试类实现一个类固定例程接口 ([IClassFixture](https://xunit.github.io/docs/shared-context#class-fixture))，以指示类包含测试，并在类中的所有测试间提供共享对象实例。

以下测试类 `BasicTests` 使用 `WebApplicationFactory` 启动 SUT，并向测试方法 `Get_EndpointsReturnSuccessAndCorrectContentType` 提供 [HttpClient](/dotnet/api/system.net.http.httpclient)。 该方法检查响应状态代码是否成功（处于范围 200-299 中的状态代码），以及 `Content-Type` 标头是否为适用于多个应用页面的 `text/html; charset=utf-8`。

[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) 创建自动追随重定向并处理 cookie 的 `HttpClient` 的实例。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

默认情况下，在启用了 [GDPR 同意策略](xref:security/gdpr)时，不会在请求间保留非必要 cookie。 若要保留非必要 cookie（如 TempData 提供程序所使用的），请在测试中将它们标记为必要。 有关将 cookie 标记为必要的说明，请参阅[必要 cookie](xref:security/gdpr#essential-cookies)。

## <a name="customize-webapplicationfactory"></a>自定义 WebApplicationFactory

通过从 `WebApplicationFactory` 来创建一个或多个自定义工厂，可以独立于测试类创建 Web 主机配置：

1. 从 `WebApplicationFactory` 继承并替代 [ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost)。 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) 允许使用 [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices) 配置服务集合：

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)中的数据库种子设定由 `InitializeDbForTests` 方法执行。 [集成测试示例：测试应用组织](#test-app-organization)部分中介绍了该方法。

   SUT 的数据库上下文在其 `Startup.ConfigureServices` 方法中注册。 测试应用的 `builder.ConfigureServices` 回调在执行应用的 `Startup.ConfigureServices` 代码之后执行。 随着 ASP.NET Core 3.0 的发布，执行顺序是针对[泛型主机](xref:fundamentals/host/generic-host)的一个重大更改。 若要将与应用数据库不同的数据库用于测试，必须在 `builder.ConfigureServices` 中替换应用的数据库上下文。

   对于仍使用 [Web 主机](xref:fundamentals/host/web-host)的 SUT，测试应用的 `builder.ConfigureServices` 回调先于 SUT 的 `Startup.ConfigureServices` 代码。 之后执行测试应用的 `builder.ConfigureTestServices` 回调。

   示例应用会查找数据库上下文的服务描述符，并使用该描述符删除服务注册。 接下来，工厂会添加一个新 `ApplicationDbContext`，它使用内存中数据库进行测试。

   若要连接到与内存中数据库不同的数据库，请更改 `UseInMemoryDatabase` 调用以将上下文连接到不同数据库。 使用 SQL Server 测试数据库：

   * 在项目文件中引用 [Microsoft.EntityFrameworkCore.SqlServer](https://www.nuget.org/packages/Microsoft.EntityFrameworkCore.SqlServer/) NuGet 包。
   * 使用连接到数据库的连接字符串调用 `UseSqlServer`。

   ```csharp
   services.AddDbContext<ApplicationDbContext>((options, context) => 
   {
       context.UseSqlServer(
           Configuration.GetConnectionString("TestingDbConnectionString"));
   });
   ```

2. 在测试类中使用自定义 `CustomWebApplicationFactory`。 下面的示例使用 `IndexPageTests` 类中的工厂：

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   示例应用的客户端配置为阻止 `HttpClient` 追随重定向。 如稍后在[模拟身份验证](#mock-authentication)部分中所述，这允许测试检查应用第一个响应的结果。 第一个响应是在许多具有 `Location` 标头的测试中进行重定向。

3. 典型测试使用 `HttpClient` 和帮助程序方法处理请求和响应：

   [!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

对 SUT 发出的任何 POST 请求都必须满足防伪检查，该检查由应用的[数据保护防伪系统](xref:security/data-protection/introduction)自动执行。 若要安排测试的 POST 请求，测试应用必须：

1. 对页面发出请求。
1. 分析来自响应的防伪 cookie 和请求验证令牌。
1. 发出放置了防伪 cookie 和请求验证令牌的 POST 请求。

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)中的 `SendAsync` 帮助程序扩展方法 (Helpers/HttpClientExtensions.cs) 和 `GetDocumentAsync` 帮助程序方法 (Helpers/HtmlHelpers.cs) 使用 [AngleSharp](https://anglesharp.github.io/) 分析程序，通过以下方法处理防伪检查：

* `GetDocumentAsync`：接收 [HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage)，并返回 `IHtmlDocument`。 `GetDocumentAsync` 使用一个基于原始 `HttpResponseMessage` 准备虚拟响应的工厂。 有关详细信息，请参阅 [AngleSharp 文档](https://github.com/AngleSharp/AngleSharp#documentation)。
* `HttpClient` 的 `SendAsync` 扩展方法撰写 [HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage) 并调用 [SendAsync(HttpRequestMessage)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_)，以向 SUT 提交请求。 `SendAsync` 的重载接受 HTML 窗体 (`IHtmlFormElement`) 和以下内容：
  * 窗体的“提交”按钮 (`IHtmlElement`)
  * 窗体值集合 (`IEnumerable<KeyValuePair<string, string>>`)
  * 提交按钮 (`IHtmlElement`) 和窗体值 (`IEnumerable<KeyValuePair<string, string>>`)

> [!NOTE]
> [AngleSharp](https://anglesharp.github.io/) 是在本主题和示例应用中用于演示的第三方分析库。 ASP.NET Core 应用的集成测试不支持或不需要 AngleSharp。 可以使用其他分析程序，如 [Html Agility Pack (HAP)](https://html-agility-pack.net/)。 另一种方法是编写代码来直接处理防伪系统的请求验证令牌和防伪 cookie。

## <a name="customize-the-client-with-withwebhostbuilder"></a>使用 WithWebHostBuilder 自定义客户端

当测试方法中需要其他配置时，[WithWebHostBuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder) 可创建新 `WebApplicationFactory`，其中包含通过配置进一步自定义的 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)。

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples) 的 `Post_DeleteMessageHandler_ReturnsRedirectToRoot` 测试方法演示了 `WithWebHostBuilder` 的使用。 此测试通过在 SUT 中触发窗体提交，在数据库中执行记录删除。

由于 `IndexPageTests` 类中的另一个测试会执行删除数据库中所有记录的操作，并且可能在 `Post_DeleteMessageHandler_ReturnsRedirectToRoot` 方法之前运行，因此数据库会在此测试方法中重新进行种子设定，以确保存在记录供 SUT 删除。 在 SUT 中选择 `messages` 窗体的第一个删除按钮可在向 SUT 发出的请求中进行模拟：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a>客户端选项

下表显示在创建 `HttpClient` 实例时可用的默认 [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)。

| 选项 | 描述 | 默认 |
| ------ | ----------- | ------- |
| [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | 获取或设置 `HttpClient` 实例是否应自动追随重定向响应。 | `true` |
| [BaseAddress](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | 获取或设置 `HttpClient` 实例的基址。 | `http://localhost` |
| [处理 Cookie](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | 获取或设置 `HttpClient` 实例是否应处理 cookie。 | `true` |
| [MaxAutomaticRedirections](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | 获取或设置 `HttpClient` 实例应追随的重定向响应的最大数量。 | 7 |

创建 `WebApplicationFactoryClientOptions` 类并将它传递给 [CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) 方法（默认值显示在代码示例中）：

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="inject-mock-services"></a>注入模拟服务

可以通过在主机生成器上调用 [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices)，在测试中替代服务。 若要注入模拟服务，SUT 必须具有包含 `Startup.ConfigureServices` 方法的 `Startup` 类。

示例 SUT 包含返回引用的作用域服务。 向索引页面进行请求时，引用嵌入在索引页面上的隐藏字段中。

Services/IQuoteService.cs：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Services/IQuoteService.cs?name=snippet1)]

Services/QuoteService.cs：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Services/QuoteService.cs?name=snippet1)]

*Startup.cs*：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet2)]

Pages/Index.cshtml.cs：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml.cs?name=snippet1&highlight=4,9,20,26)]

Pages/Index.cs：

[!code-cshtml[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml?name=snippet_Quote)]

运行 SUT 应用时，会生成以下标记：

```html
<input id="quote" type="hidden" value="Come on, Sarah. We&#x27;ve an appointment in 
    London, and we&#x27;re already 30,000 years late.">
```

若要在集成测试中测试服务和引用注入，测试会将模拟服务注入到 SUT 中。 模拟服务会将应用的 `QuoteService` 替换为测试应用提供的服务，称为 `TestQuoteService`：

IntegrationTests.IndexPageTests.cs：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet4)]

调用 `ConfigureTestServices`，并注册作用域服务：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet5&highlight=7-10,17,20-21)]

在测试执行过程中生成的标记反映由 `TestQuoteService` 提供的引用文本，因而断言通过：

```html
<input id="quote" type="hidden" value="Something&#x27;s interfering with time, 
    Mr. Scarman, and time is my business.">
```

## <a name="mock-authentication"></a>模拟身份验证

`AuthTests` 类中的测试检查安全终结点是否：

* 将未经身份验证的用户重定向到应用的登录页面。
* 为经过身份验证的用户返回内容。

在 SUT 中，`/SecurePage` 页面使用 [AuthorizePage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage) 约定将 [AuthorizeFilter](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) 应用于页面。 有关详细信息，请参阅 [Razor Pages 授权约定](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

在 `Get_SecurePageRedirectsAnUnauthenticatedUser` 测试中，[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) 设置为禁止重定向，具体方法是将 [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) 设置为 `false`：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet2)]

通过禁止客户端追随重定向，可以执行以下检查：

* 可以根据预期 [HttpStatusCode.Redirect](/dotnet/api/system.net.httpstatuscode) 结果检查 SUT 返回的状态代码，而不是在重定向到登录页面之后的最终状态代码（这会是 [HttpStatusCode.OK](/dotnet/api/system.net.httpstatuscode)）。
* 检查响应标头中的 `Location` 标头值，以确认它以 `http://localhost/Identity/Account/Login` 开头，而不是最终登录页面响应（其中 `Location` 标头不存在）。

测试应用可以在 [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) 中模拟 <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>，以便测试身份验证和授权的各个方面。 最小方案返回一个 [AuthenticateResult.Success](xref:Microsoft.AspNetCore.Authentication.AuthenticateResult.Success*)：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet4&highlight=11-18)]

当身份验证方案设置为 `Test`（其中为 `ConfigureTestServices` 注册了 `AddAuthentication`）时，会调用 `TestAuthHandler` 以对用户进行身份验证。 `Test` 架构必须与应用所需的架构匹配，这一点很重要。 否则，身份验证将不起作用。

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet3&highlight=7-12)]

有关 `WebApplicationFactoryClientOptions` 的详细信息，请参阅[客户端选项](#client-options)部分。

## <a name="set-the-environment"></a>设置环境

默认情况下，SUT 的主机和应用环境配置为使用开发环境。 使用 `IHostBuilder` 时替代 SUT 的环境：

* 设置 `ASPNETCORE_ENVIRONMENT` 环境变量（例如，`Staging`、`Production` 或其他自定义值，例如 `Testing`）。
* 在测试应用中替代 `CreateHostBuilder`，以读取以 `ASPNETCORE` 为前缀的环境变量。

```csharp
protected override IHostBuilder CreateHostBuilder() =>
    base.CreateHostBuilder()
        .ConfigureHostConfiguration(
            config => config.AddEnvironmentVariables("ASPNETCORE"));
```

如果 SUT 使用 Web 主机 (`IWebHostBuilder`)，则替代 `CreateWebHostBuilder`：

```csharp
protected override IWebHostBuilder CreateWebHostBuilder() =>
    base.CreateWebHostBuilder().UseEnvironment("Testing");
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a>测试基础结构如何推断应用内容根路径

`WebApplicationFactory` 构造函数通过在包含集成测试的程序集中搜索键等于 `TEntryPoint` 程序集 `System.Reflection.Assembly.FullName` 的 [WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute)，来推断应用[内容根](xref:fundamentals/index#content-root)路径。 如果找不到具有正确键的属性，则 `WebApplicationFactory` 会回退到搜索解决方案文件 (.sln) 并将 `TEntryPoint` 程序集名称追加到解决方案目录。 应用根目录（内容根路径）用于发现视图和内容文件。

## <a name="disable-shadow-copying"></a>禁用卷影复制

卷影复制会导致在与输出目录不同的目录中执行测试。 若要使测试正常工作，必须禁用卷影复制。 [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)使用 xUnit 并通过包含具有正确配置设置的 xunit.runner.json 文件来对 xUnit 禁用卷影复制。 有关详细信息，请参阅[使用 JSON 配置 xUnit](https://xunit.github.io/docs/configuring-with-json.html)。

将包含以下内容的 xunit.runner.json 文件添加到测试项目的根：

```json
{
  "shadowCopy": false
}
```

## <a name="disposal-of-objects"></a>对象的处置

执行 `IClassFixture` 实现的测试之后，当 xUnit 处置 [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) 时，[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) 和 [HttpClient](/dotnet/api/system.net.http.httpclient) 会进行处置。 如果开发者实例化的对象需要处置，请在 `IClassFixture` 实现中处置它们。 有关详细信息，请参阅[实现 Dispose 方法](/dotnet/standard/garbage-collection/implementing-dispose)。

## <a name="integration-tests-sample"></a>集成测试示例

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)包含两个应用：

| 应用 | 项目目录 | 描述 |
| --- | ----------------- | ----------- |
| 消息应用 (SUT) | src/RazorPagesProject | 允许用户添加消息、删除一个消息、删除所有消息和分析消息。 |
| 测试应用 | tests/RazorPagesProject.Tests | 用于集成测试 SUT。 |

可使用 IDE 的内置测试功能（例如 [Visual Studio](https://visualstudio.microsoft.com)）运行测试。 如果使用 [Visual Studio Code](https://code.visualstudio.com/) 或命令行，请在 tests/RazorPagesProject.Tests 目录中的命令提示符处执行以下命令：

```console
dotnet test
```

### <a name="message-app-sut-organization"></a>消息应用 (SUT) 组织

SUT 是具有以下特征的 Razor Pages 消息系统：

* 应用的索引页面（Pages/Index.cshtml 和 Pages/Index.cshtml.cs）提供 UI 和页面模型方法，用于控制添加、删除和分析消息（每个消息的平均字词数）。
* 消息由 `Message` 类 (Data/Message.cs) 描述，并具有两个属性：`Id`（键）和 `Text`（消息）。 `Text` 属性是必需的，并限制为 200 个字符。
* 消息使用[实体框架的内存中数据库](/ef/core/providers/in-memory/)存储。&#8224;
* 应用在其数据库上下文类 `AppDbContext` (Data/AppDbContext.cs) 中包含数据访问层 (DAL)。
* 如果应用启动时数据库为空，则消息存储初始化为三条消息。
* 应用包含只能由经过身份验证的用户访问的 `/SecurePage`。

&#8224;EF 主题[使用 InMemory 进行测试](/ef/core/miscellaneous/testing/in-memory)说明如何将内存中数据库用于 MSTest 测试。 本主题使用 [xUnit](https://xunit.github.io/) 测试框架。 不同测试框架中的测试概念和测试实现相似，但不完全相同。

尽管应用未使用存储库模式且不是[工作单元 (UoW) 模式](https://martinfowler.com/eaaCatalog/unitOfWork.html)的有效示例，但 Razor Pages 支持这些开发模式。 有关详细信息，请参阅[设计基础结构持久性层](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)和[测试控制器逻辑](../mvc/controllers/testing.md)（该示例实现存储库模式）。

### <a name="test-app-organization"></a>测试应用组织

测试应用是 tests/RazorPagesProject.Tests 目录中的控制台应用。

| 测试应用目录 | 描述 |
| ------------------ | ----------- |
| AuthTests | 包含针对以下方面的测试方法：<ul><li>未经身份验证的用户访问安全页面。</li><li>经过身份验证的用户访问安全页面（通过模拟 <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>）。</li><li>获取 GitHub 用户配置文件，并检查配置文件的用户登录。</li></ul> |
| BasicTests | 包含用于路由和内容类型的测试方法。 |
| IntegrationTests | 包含使用自定义 `WebApplicationFactory` 类的索引页面的集成测试。 |
| Helpers/Utilities | <ul><li>Utilities.cs 包含用于通过测试数据设定数据库种子的 `InitializeDbForTests` 方法。</li><li>HtmlHelpers.cs 提供了一种方法，用于返回 AngleSharp `IHtmlDocument` 供测试方法使用。</li><li>HttpClientExtensions.cs 为 `SendAsync` 提供重载，以将请求提交到 SUT。</li></ul> |

测试框架为 [xUnit](https://xunit.github.io/)。 集成测试使用 [Microsoft.AspNetCore.TestHost](/dotnet/api/microsoft.aspnetcore.testhost)（包括 [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)）执行。 由于 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) 包用于配置测试主机和测试服务器，因此 `TestHost` 和 `TestServer` 包在测试应用的项目文件或测试应用的开发者配置中不需要直接包引用。

设定数据库种子以进行测试

集成测试在执行测试前通常需要数据库中的小型数据集。 例如，删除测试需要进行数据库记录删除，因此数据库必须至少有一个记录，删除请求才能成功。

示例应用使用 Utilities.cs 中的三个消息（测试在执行时可以使用它们）设定数据库种子：

[!code-csharp[](integration-tests/samples/3.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

SUT 的数据库上下文在其 `Startup.ConfigureServices` 方法中注册。 测试应用的 `builder.ConfigureServices` 回调在执行应用的 `Startup.ConfigureServices` 代码之后执行。 若要将不同的数据库用于测试，必须在 `builder.ConfigureServices` 中替换应用的数据库上下文。 有关详细信息，请参阅[自定义 WebApplicationFactory](#customize-webapplicationfactory) 部分。

对于仍使用 [Web 主机](xref:fundamentals/host/web-host)的 SUT，测试应用的 `builder.ConfigureServices` 回调先于 SUT 的 `Startup.ConfigureServices` 代码。 之后执行测试应用的 `builder.ConfigureTestServices` 回调。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

集成测试可在包含应用支持基础结构（如数据库、文件系统和网络）的级别上确保应用组件功能正常。 ASP.NET Core 通过将单元测试框架与测试 Web 主机和内存中测试服务器结合使用来支持集成测试。

本主题假设读者基本了解单元测试。 如果不熟悉测试概念，请参阅 [.NET Core 和 .NET Standard 中的单元测试](/dotnet/core/testing/)主题及其链接内容。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)（[如何下载](xref:index#how-to-download-a-sample)）

示例应用是 Razor Pages 应用，假设读者基本了解 Razor Pages。 如果不熟悉 Razor Pages，请参阅以下主题：

* [Razor Pages 简介](xref:razor-pages/index)
* [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)
* [Razor Pages 单元测试](xref:test/razor-pages-tests)

> [!NOTE]
> 对于测试 SPA，建议使用可以自动执行浏览器的工具，如 [Selenium](https://www.seleniumhq.org/)。

## <a name="introduction-to-integration-tests"></a>集成测试简介

与[单元测试](/dotnet/core/testing/)相比，集成测试可在更广泛的级别上评估应用的组件。 单元测试用于测试独立软件组件，如单独的类方法。 集成测试确认两个或更多应用组件一起工作以生成预期结果，可能包括完整处理请求所需的每个组件。

这些更广泛的测试用于测试应用的基础结构和整个框架，通常包括以下组件：

* 数据库
* 文件系统
* 网络设备
* 请求-响应管道

单元测试使用称为 fake 或 mock 对象的制造组件，而不是基础结构组件。

与单元测试相比，集成测试：

* 使用应用在生产环境中使用的实际组件。
* 需要进行更多代码和数据处理。
* 需要更长时间来运行。

因此，将集成测试的使用限制为最重要的基础结构方案。 如果可以使用单元测试或集成测试来测试行为，请选择单元测试。

> [!TIP]
> 请勿为通过数据库和文件系统进行的数据和文件访问的每个可能排列编写集成测试。 无论应用中有多少位置与数据库和文件系统交互，一组集中式读取、写入、更新和删除集成测试通常能够充分测试数据库和文件系统组件。 将单元测试用于与这些组件交互的方法逻辑的例程测试。 在单元测试中，使用基础结构 fake/mock 会导致更快地执行测试。

> [!NOTE]
> 在集成测试的讨论中，测试的项目经常称为“测试中的系统”，简称“SUT”。
>
> 本主题中使用“SUT”来指代测试的 ASP.NET Core 应用。

## <a name="aspnet-core-integration-tests"></a>ASP.NET Core 集成测试

ASP.NET Core 中的集成测试需要以下内容：

* 测试项目用于包含和执行测试。 测试项目具有对 SUT 的引用。
* 测试项目为 SUT 创建测试 Web 主机，并使用测试服务器客户端处理 SUT 的请求和响应。
* 测试运行程序用于执行测试并报告测试结果。

集成测试后跟一系列事件，包括常规“排列”、“操作”和“断言”测试步骤：

1. 已配置 SUT 的 Web 主机。
1. 创建测试服务器客户端以向应用提交请求。
1. 执行“排列”测试步骤：测试应用会准备请求。
1. 执行“操作”测试步骤：客户端提交请求并接收响应。
1. 执行“断言”测试步骤：实际响应基于预期响应验证为通过或失败。
1. 该过程会一直继续，直到执行了所有测试。
1. 报告测试结果。

通常，测试 Web 主机的配置与用于测试运行的应用常规 Web 主机不同。 例如，可以将不同的数据库或不同的应用设置用于测试。

基础结构组件（如测试 Web 主机和内存中测试服务器 ([TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver))）由 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) 包提供或管理。 使用此包可简化测试创建和执行。

`Microsoft.AspNetCore.Mvc.Testing` 包处理以下任务：

* 将依赖项文件 (.deps) 从 SUT 复制到测试项目的 bin 目录中。
* 将[内容根目录](xref:fundamentals/index#content-root)设置为 SUT 的项目根目录，以便可在执行测试时找到静态文件和页面/视图。
* 提供 [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) 类，以简化 SUT 在 `TestServer` 中的启动过程。

[单元测试](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)文档介绍如何设置测试项目和测试运行程序，以及有关如何运行测试的详细说明与有关如何命名测试和测试类的建议。

> [!NOTE]
> 为应用创建测试项目时，请将集成测试中的单元测试分隔到不同的项目中。 这可帮助确保不会意外地将基础结构测试组件包含在单元测试中。 通过分隔单元测试和集成测试还可以控制运行的测试集。

Razor Pages 应用与 MVC 应用的测试配置之间几乎没有任何区别。 唯一的区别在于测试的命名方式。 在 Razor Pages 应用中，页面终结点的测试通常以页面模型类命名（例如，`IndexPageTests` 用于为索引页面测试组件集成）。 在 MVC 应用中，测试通常按控制器类进行组织，并以它们所测试的控制器来命令（例如 `HomeControllerTests` 用于为主页控制器测试组件集成）。

## <a name="test-app-prerequisites"></a>测试应用先决条件

测试项目必须：

* 引用以下包：
  * [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)
  * [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing/)
* 在项目文件中指定 Web SDK (`<Project Sdk="Microsoft.NET.Sdk.Web">`)。 引用 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)时需要 Web SDK。

可以在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)中查看这些先决条件。 检查 tests/RazorPagesProject.Tests/RazorPagesProject.Tests.csproj 文件。 示例应用使用 [xUnit](https://xunit.github.io/) 测试框架和 [AngleSharp](https://anglesharp.github.io/) 分析程序库，因此示例应用还引用：

* [xunit](https://www.nuget.org/packages/xunit/)
* [xunit.runner.visualstudio](https://www.nuget.org/packages/xunit.runner.visualstudio/)
* [AngleSharp](https://www.nuget.org/packages/AngleSharp/)

## <a name="sut-environment"></a>SUT 环境

如果未设置 SUT 的[环境](xref:fundamentals/environments)，则环境会默认为“开发”。

## <a name="basic-tests-with-the-default-webapplicationfactory"></a>使用默认 WebApplicationFactory 的基本测试

[WebApplicationFactory\<TEntryPoint>](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) 用于为集成测试创建 [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)。 `TEntryPoint` 是 SUT 的入口点类，通常是 `Startup` 类。

测试类实现一个类固定例程接口 ([IClassFixture](https://xunit.github.io/docs/shared-context#class-fixture))，以指示类包含测试，并在类中的所有测试间提供共享对象实例。

以下测试类 `BasicTests` 使用 `WebApplicationFactory` 启动 SUT，并向测试方法 `Get_EndpointsReturnSuccessAndCorrectContentType` 提供 [HttpClient](/dotnet/api/system.net.http.httpclient)。 该方法检查响应状态代码是否成功（处于范围 200-299 中的状态代码），以及 `Content-Type` 标头是否为适用于多个应用页面的 `text/html; charset=utf-8`。

[CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) 创建自动追随重定向并处理 cookie 的 `HttpClient` 的实例。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/BasicTests.cs?name=snippet1)]

默认情况下，在启用了 [GDPR 同意策略](xref:security/gdpr)时，不会在请求间保留非必要 cookie。 若要保留非必要 cookie（如 TempData 提供程序所使用的），请在测试中将它们标记为必要。 有关将 cookie 标记为必要的说明，请参阅[必要 cookie](xref:security/gdpr#essential-cookies)。

## <a name="customize-webapplicationfactory"></a>自定义 WebApplicationFactory

通过从 `WebApplicationFactory` 来创建一个或多个自定义工厂，可以独立于测试类创建 Web 主机配置：

1. 从 `WebApplicationFactory` 继承并替代 [ConfigureWebHost](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.configurewebhost)。 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) 允许使用 [ConfigureServices](/dotnet/api/microsoft.aspnetcore.hosting.istartup.configureservices) 配置服务集合，该配置在应用的 `Startup.ConfigureServices` 之前执行。 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder) 允许使用 [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) 重写和修改服务集合中已注册的服务：

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/CustomWebApplicationFactory.cs?name=snippet1)]

   [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)中的数据库种子设定由 `InitializeDbForTests` 方法执行。 [集成测试示例：测试应用组织](#test-app-organization)部分中介绍了该方法。

2. 在测试类中使用自定义 `CustomWebApplicationFactory`。 下面的示例使用 `IndexPageTests` 类中的工厂：

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet1)]

   示例应用的客户端配置为阻止 `HttpClient` 追随重定向。 如稍后在[模拟身份验证](#mock-authentication)部分中所述，这允许测试检查应用第一个响应的结果。 第一个响应是在许多具有 `Location` 标头的测试中进行重定向。

3. 典型测试使用 `HttpClient` 和帮助程序方法处理请求和响应：

   [!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet2)]

对 SUT 发出的任何 POST 请求都必须满足防伪检查，该检查由应用的[数据保护防伪系统](xref:security/data-protection/introduction)自动执行。 若要安排测试的 POST 请求，测试应用必须：

1. 对页面发出请求。
1. 分析来自响应的防伪 cookie 和请求验证令牌。
1. 发出放置了防伪 cookie 和请求验证令牌的 POST 请求。

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples/)中的 `SendAsync` 帮助程序扩展方法 (Helpers/HttpClientExtensions.cs) 和 `GetDocumentAsync` 帮助程序方法 (Helpers/HtmlHelpers.cs) 使用 [AngleSharp](https://anglesharp.github.io/) 分析程序，通过以下方法处理防伪检查：

* `GetDocumentAsync`：接收 [HttpResponseMessage](/dotnet/api/system.net.http.httpresponsemessage)，并返回 `IHtmlDocument`。 `GetDocumentAsync` 使用一个基于原始 `HttpResponseMessage` 准备虚拟响应的工厂。 有关详细信息，请参阅 [AngleSharp 文档](https://github.com/AngleSharp/AngleSharp#documentation)。
* `HttpClient` 的 `SendAsync` 扩展方法撰写 [HttpRequestMessage](/dotnet/api/system.net.http.httprequestmessage) 并调用 [SendAsync(HttpRequestMessage)](/dotnet/api/system.net.http.httpclient.sendasync#System_Net_Http_HttpClient_SendAsync_System_Net_Http_HttpRequestMessage_)，以向 SUT 提交请求。 `SendAsync` 的重载接受 HTML 窗体 (`IHtmlFormElement`) 和以下内容：
  * 窗体的“提交”按钮 (`IHtmlElement`)
  * 窗体值集合 (`IEnumerable<KeyValuePair<string, string>>`)
  * 提交按钮 (`IHtmlElement`) 和窗体值 (`IEnumerable<KeyValuePair<string, string>>`)

> [!NOTE]
> [AngleSharp](https://anglesharp.github.io/) 是在本主题和示例应用中用于演示的第三方分析库。 ASP.NET Core 应用的集成测试不支持或不需要 AngleSharp。 可以使用其他分析程序，如 [Html Agility Pack (HAP)](https://html-agility-pack.net/)。 另一种方法是编写代码来直接处理防伪系统的请求验证令牌和防伪 cookie。

## <a name="customize-the-client-with-withwebhostbuilder"></a>使用 WithWebHostBuilder 自定义客户端

当测试方法中需要其他配置时，[WithWebHostBuilder](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.withwebhostbuilder) 可创建新 `WebApplicationFactory`，其中包含通过配置进一步自定义的 [IWebHostBuilder](/dotnet/api/microsoft.aspnetcore.hosting.iwebhostbuilder)。

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples) 的 `Post_DeleteMessageHandler_ReturnsRedirectToRoot` 测试方法演示了 `WithWebHostBuilder` 的使用。 此测试通过在 SUT 中触发窗体提交，在数据库中执行记录删除。

由于 `IndexPageTests` 类中的另一个测试会执行删除数据库中所有记录的操作，并且可能在 `Post_DeleteMessageHandler_ReturnsRedirectToRoot` 方法之前运行，因此数据库会在此测试方法中重新进行种子设定，以确保存在记录供 SUT 删除。 在 SUT 中选择 `messages` 窗体的第一个删除按钮可在向 SUT 发出的请求中进行模拟：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet3)]

## <a name="client-options"></a>客户端选项

下表显示在创建 `HttpClient` 实例时可用的默认 [WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions)。

| 选项 | 说明 | 默认 |
| ------ | ----------- | ------- |
| [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) | 获取或设置 `HttpClient` 实例是否应自动追随重定向响应。 | `true` |
| [BaseAddress](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.baseaddress) | 获取或设置 `HttpClient` 实例的基址。 | `http://localhost` |
| [处理 Cookie](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.handlecookies) | 获取或设置 `HttpClient` 实例是否应处理 cookie。 | `true` |
| [MaxAutomaticRedirections](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.maxautomaticredirections) | 获取或设置 `HttpClient` 实例应追随的重定向响应的最大数量。 | 7 |

创建 `WebApplicationFactoryClientOptions` 类并将它传递给 [CreateClient](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1.createclient) 方法（默认值显示在代码示例中）：

```csharp
// Default client option values are shown
var clientOptions = new WebApplicationFactoryClientOptions();
clientOptions.AllowAutoRedirect = true;
clientOptions.BaseAddress = new Uri("http://localhost");
clientOptions.HandleCookies = true;
clientOptions.MaxAutomaticRedirections = 7;

_client = _factory.CreateClient(clientOptions);
```

## <a name="inject-mock-services"></a>注入模拟服务

可以通过在主机生成器上调用 [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices)，在测试中替代服务。 若要注入模拟服务，SUT 必须具有包含 `Startup.ConfigureServices` 方法的 `Startup` 类。

示例 SUT 包含返回引用的作用域服务。 向索引页面进行请求时，引用嵌入在索引页面上的隐藏字段中。

Services/IQuoteService.cs：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Services/IQuoteService.cs?name=snippet1)]

Services/QuoteService.cs：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Services/QuoteService.cs?name=snippet1)]

*Startup.cs*：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet2)]

Pages/Index.cshtml.cs：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml.cs?name=snippet1&highlight=4,9,20,26)]

Pages/Index.cs：

[!code-cshtml[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Pages/Index.cshtml?name=snippet_Quote)]

运行 SUT 应用时，会生成以下标记：

```html
<input id="quote" type="hidden" value="Come on, Sarah. We&#x27;ve an appointment in 
    London, and we&#x27;re already 30,000 years late.">
```

若要在集成测试中测试服务和引用注入，测试会将模拟服务注入到 SUT 中。 模拟服务会将应用的 `QuoteService` 替换为测试应用提供的服务，称为 `TestQuoteService`：

IntegrationTests.IndexPageTests.cs：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet4)]

调用 `ConfigureTestServices`，并注册作用域服务：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/IndexPageTests.cs?name=snippet5&highlight=7-10,17,20-21)]

在测试执行过程中生成的标记反映由 `TestQuoteService` 提供的引用文本，因而断言通过：

```html
<input id="quote" type="hidden" value="Something&#x27;s interfering with time, 
    Mr. Scarman, and time is my business.">
```

## <a name="mock-authentication"></a>模拟身份验证

`AuthTests` 类中的测试检查安全终结点是否：

* 将未经身份验证的用户重定向到应用的登录页面。
* 为经过身份验证的用户返回内容。

在 SUT 中，`/SecurePage` 页面使用 [AuthorizePage](/dotnet/api/microsoft.extensions.dependencyinjection.pageconventioncollectionextensions.authorizepage) 约定将 [AuthorizeFilter](/dotnet/api/microsoft.aspnetcore.mvc.authorization.authorizefilter) 应用于页面。 有关详细信息，请参阅 [Razor Pages 授权约定](xref:security/authorization/razor-pages-authorization#require-authorization-to-access-a-page)。

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/src/RazorPagesProject/Startup.cs?name=snippet1)]

在 `Get_SecurePageRedirectsAnUnauthenticatedUser` 测试中，[WebApplicationFactoryClientOptions](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions) 设置为禁止重定向，具体方法是将 [AllowAutoRedirect](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactoryclientoptions.allowautoredirect) 设置为 `false`：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet2)]

通过禁止客户端追随重定向，可以执行以下检查：

* 可以根据预期 [HttpStatusCode.Redirect](/dotnet/api/system.net.httpstatuscode) 结果检查 SUT 返回的状态代码，而不是在重定向到登录页面之后的最终状态代码（这会是 [HttpStatusCode.OK](/dotnet/api/system.net.httpstatuscode)）。
* 检查响应标头中的 `Location` 标头值，以确认它以 `http://localhost/Identity/Account/Login` 开头，而不是最终登录页面响应（其中 `Location` 标头不存在）。

测试应用可以在 [ConfigureTestServices](/dotnet/api/microsoft.aspnetcore.testhost.webhostbuilderextensions.configuretestservices) 中模拟 <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>，以便测试身份验证和授权的各个方面。 最小方案返回一个 [AuthenticateResult.Success](xref:Microsoft.AspNetCore.Authentication.AuthenticateResult.Success*)：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet4&highlight=11-18)]

当身份验证方案设置为 `Test`（其中为 `ConfigureTestServices` 注册了 `AddAuthentication`）时，会调用 `TestAuthHandler` 以对用户进行身份验证：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/IntegrationTests/AuthTests.cs?name=snippet3&highlight=7-12)]

有关 `WebApplicationFactoryClientOptions` 的详细信息，请参阅[客户端选项](#client-options)部分。

## <a name="set-the-environment"></a>设置环境

默认情况下，SUT 的主机和应用环境配置为使用开发环境。 替代 SUT 的环境：

* 设置 `ASPNETCORE_ENVIRONMENT` 环境变量（例如，`Staging`、`Production` 或其他自定义值，例如 `Testing`）。
* 在测试应用中重写 `CreateWebHostBuilder`，以读取 `ASPNETCORE_ENVIRONMENT` 环境变量。

```csharp
public class CustomWebApplicationFactory<TStartup> 
    : WebApplicationFactory<TStartup> where TStartup: class
{
    protected override IWebHostBuilder CreateWebHostBuilder()
    {
        return base.CreateWebHostBuilder()
            .UseEnvironment(
                Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT"));
    }

    ...
}
```

还可以在自定义 <xref:Microsoft.AspNetCore.Mvc.Testing.WebApplicationFactory%601> 中直接在主机生成器上设置环境：

```csharp
public class CustomWebApplicationFactory<TStartup> 
    : WebApplicationFactory<TStartup> where TStartup: class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.UseEnvironment(
            Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT"));
    
        ...
    }

    ...
```

## <a name="how-the-test-infrastructure-infers-the-app-content-root-path"></a>测试基础结构如何推断应用内容根路径

`WebApplicationFactory` 构造函数通过在包含集成测试的程序集中搜索键等于 `TEntryPoint` 程序集 `System.Reflection.Assembly.FullName` 的 [WebApplicationFactoryContentRootAttribute](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactorycontentrootattribute)，来推断应用[内容根](xref:fundamentals/index#content-root)路径。 如果找不到具有正确键的属性，则 `WebApplicationFactory` 会回退到搜索解决方案文件 (.sln) 并将 `TEntryPoint` 程序集名称追加到解决方案目录。 应用根目录（内容根路径）用于发现视图和内容文件。

## <a name="disable-shadow-copying"></a>禁用卷影复制

卷影复制会导致在与输出目录不同的目录中执行测试。 若要使测试正常工作，必须禁用卷影复制。 [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)使用 xUnit 并通过包含具有正确配置设置的 xunit.runner.json 文件来对 xUnit 禁用卷影复制。 有关详细信息，请参阅[使用 JSON 配置 xUnit](https://xunit.github.io/docs/configuring-with-json.html)。

将包含以下内容的 xunit.runner.json 文件添加到测试项目的根：

```json
{
  "shadowCopy": false
}
```

如果使用 Visual Studio，将文件的“复制到输出目录”属性设置为“始终复制”。 如果不使用 Visual Studio，请将 `Content` 目标添加到测试应用的项目文件：

```xml
<ItemGroup>
  <Content Update="xunit.runner.json">
    <CopyToOutputDirectory>Always</CopyToOutputDirectory>
  </Content>
</ItemGroup>
```

## <a name="disposal-of-objects"></a>对象的处置

执行 `IClassFixture` 实现的测试之后，当 xUnit 处置 [WebApplicationFactory](/dotnet/api/microsoft.aspnetcore.mvc.testing.webapplicationfactory-1) 时，[TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver) 和 [HttpClient](/dotnet/api/system.net.http.httpclient) 会进行处置。 如果开发者实例化的对象需要处置，请在 `IClassFixture` 实现中处置它们。 有关详细信息，请参阅[实现 Dispose 方法](/dotnet/standard/garbage-collection/implementing-dispose)。

## <a name="integration-tests-sample"></a>集成测试示例

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/test/integration-tests/samples)包含两个应用：

| 应用 | 项目目录 | 描述 |
| --- | ----------------- | ----------- |
| 消息应用 (SUT) | src/RazorPagesProject | 允许用户添加消息、删除一个消息、删除所有消息和分析消息。 |
| 测试应用 | tests/RazorPagesProject.Tests | 用于集成测试 SUT。 |

可使用 IDE 的内置测试功能（例如 [Visual Studio](https://visualstudio.microsoft.com)）运行测试。 如果使用 [Visual Studio Code](https://code.visualstudio.com/) 或命令行，请在 tests/RazorPagesProject.Tests 目录中的命令提示符处执行以下命令：

```dotnetcli
dotnet test
```

### <a name="message-app-sut-organization"></a>消息应用 (SUT) 组织

SUT 是具有以下特征的 Razor Pages 消息系统：

* 应用的索引页面（Pages/Index.cshtml 和 Pages/Index.cshtml.cs）提供 UI 和页面模型方法，用于控制添加、删除和分析消息（每个消息的平均字词数）。
* 消息由 `Message` 类 (Data/Message.cs) 描述，并具有两个属性：`Id`（键）和 `Text`（消息）。 `Text` 属性是必需的，并限制为 200 个字符。
* 消息使用[实体框架的内存中数据库](/ef/core/providers/in-memory/)存储。&#8224;
* 应用在其数据库上下文类 `AppDbContext` (Data/AppDbContext.cs) 中包含数据访问层 (DAL)。
* 如果应用启动时数据库为空，则消息存储初始化为三条消息。
* 应用包含只能由经过身份验证的用户访问的 `/SecurePage`。

&#8224;EF 主题[使用 InMemory 进行测试](/ef/core/miscellaneous/testing/in-memory)说明如何将内存中数据库用于 MSTest 测试。 本主题使用 [xUnit](https://xunit.github.io/) 测试框架。 不同测试框架中的测试概念和测试实现相似，但不完全相同。

尽管应用未使用存储库模式且不是[工作单元 (UoW) 模式](https://martinfowler.com/eaaCatalog/unitOfWork.html)的有效示例，但 Razor Pages 支持这些开发模式。 有关详细信息，请参阅[设计基础结构持久性层](/dotnet/standard/microservices-architecture/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)和[测试控制器逻辑](../mvc/controllers/testing.md)（该示例实现存储库模式）。

### <a name="test-app-organization"></a>测试应用组织

测试应用是 tests/RazorPagesProject.Tests 目录中的控制台应用。

| 测试应用目录 | 描述 |
| ------------------ | ----------- |
| AuthTests | 包含针对以下方面的测试方法：<ul><li>未经身份验证的用户访问安全页面。</li><li>经过身份验证的用户访问安全页面（通过模拟 <xref:Microsoft.AspNetCore.Authentication.AuthenticationHandler`1>）。</li><li>获取 GitHub 用户配置文件，并检查配置文件的用户登录。</li></ul> |
| BasicTests | 包含用于路由和内容类型的测试方法。 |
| IntegrationTests | 包含使用自定义 `WebApplicationFactory` 类的索引页面的集成测试。 |
| Helpers/Utilities | <ul><li>Utilities.cs 包含用于通过测试数据设定数据库种子的 `InitializeDbForTests` 方法。</li><li>HtmlHelpers.cs 提供了一种方法，用于返回 AngleSharp `IHtmlDocument` 供测试方法使用。</li><li>HttpClientExtensions.cs 为 `SendAsync` 提供重载，以将请求提交到 SUT。</li></ul> |

测试框架为 [xUnit](https://xunit.github.io/)。 集成测试使用 [Microsoft.AspNetCore.TestHost](/dotnet/api/microsoft.aspnetcore.testhost)（包括 [TestServer](/dotnet/api/microsoft.aspnetcore.testhost.testserver)）执行。 由于 [Microsoft.AspNetCore.Mvc.Testing](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Testing) 包用于配置测试主机和测试服务器，因此 `TestHost` 和 `TestServer` 包在测试应用的项目文件或测试应用的开发者配置中不需要直接包引用。

设定数据库种子以进行测试

集成测试在执行测试前通常需要数据库中的小型数据集。 例如，删除测试需要进行数据库记录删除，因此数据库必须至少有一个记录，删除请求才能成功。

示例应用使用 Utilities.cs 中的三个消息（测试在执行时可以使用它们）设定数据库种子：

[!code-csharp[](integration-tests/samples/2.x/IntegrationTestsSample/tests/RazorPagesProject.Tests/Helpers/Utilities.cs?name=snippet1)]

::: moniker-end

## <a name="additional-resources"></a>其他资源

* [单元测试](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)
* <xref:test/razor-pages-tests>
* <xref:fundamentals/middleware/index>
* <xref:mvc/controllers/testing>