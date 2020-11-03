---
title: ASP.NET Core 3.0 的新增功能
author: rick-anderson
description: 了解 ASP.NET Core 3.0 的新增功能。
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
uid: aspnetcore-3.0
ms.openlocfilehash: a1cbd8f434214ce488fc3febe63d8aaf54db71a3
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722782"
---
# <a name="whats-new-in-aspnet-core-30"></a>ASP.NET Core 3.0 的新增功能

本文重点介绍 ASP.NET Core 3.0 中最重要的更改，并提供相关文档的链接。

## Blazor

Blazor 是 ASP.NET Core 中的新框架，用于使用 .NET 生成交互式客户端 Web UI：

* 使用 C# 代替 JavaScript 来创建丰富的交互式 UI。
* 共享使用 .NET 编写的服务器端和客户端应用逻辑。
* 将 UI 呈现为 HTML 和 CSS，以支持众多浏览器，其中包括移动浏览器。

Blazor 框架支持的方案：

* 可重用的 UI 组件（Razor 组件）
* 客户端路由
* 组件布局
* 对依赖项注入的支持
* 窗体和验证
* 用 Razor 类库构建组件库
* JavaScript 互操作

有关详细信息，请参阅 <xref:blazor/index>。

### Blazor Server

Blazor 将组件呈现逻辑从 UI 更新的应用方式中分离出来。 Blazor Server在 ASP.NET Core 应用中支持在服务器上托管 Razor 组件。 可通过 SignalR 连接处理 UI 更新。 ASP.NET Core 3.0 支持 Blazor Server。

### <a name="no-locblazor-webassembly-preview"></a>Blazor WebAssembly（预览版）

还可以使用基于 WebAssembly 的 .NET 运行时直接在浏览器中运行 Blazor 应用。 Blazor WebAssembly 处于预览版阶段，ASP.NET Core 3.0 不提供支持。 ASP.NET Core 的未来版本将支持 Blazor WebAssembly。

### <a name="no-locrazor-components"></a>Razor 组件

Blazor 应用是基于组件构建的。 组件是自包含的用户界面 (UI) 块，例如页、对话框或窗体。 组件是定义 UI 呈现逻辑和客户端事件处理程序的普通 .NET 类。 无需 JavaScript 即可创建丰富的交互式 Web 应用。

通常使用 Razor 语法（HTML 和 C# 的自然混合）创建 Blazor 中的组件。 Razor 组件与 Razor Pages 和 MVC 视图类似，因为它们都使用 Razor。 与基于请求-响应模型的页和视图不同，组件专门用于处理 UI 构成。

## <a name="grpc"></a>gRPC

[gRPC](https://grpc.io/)：

* 是一个流行的高性能 RPC（远程过程调用）框架。
* 提供固定的协定优先方法进行 API 开发。
* 使用的新式技术如下：

  * 用于传输的 HTTP/2。
  * 作为接口描述语言的协议缓冲区。
  * 二进制序列化格式。
* 提供如下功能：

  * 身份验证
  * 双向流式处理和流控制。
  * 取消和超时。

ASP.NET Core 3.0 中的 gRPC 功能包括：

* [Grpc.AspNetCore](https://www.nuget.org/packages/Grpc.AspNetCore)：用于托管 gRPC 服务的 ASP.NET Core 框架。 ASP.NET Core 上的 gRPC 与标准 ASP.NET Core 功能（例如日志记录、依赖关系注入 (DI)、身份验证和授权）集成。
* [Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client)：基于熟悉的 `HttpClient` 构建的 .NET Core 的 gRPC 客户端。
* [Grpc.Net.ClientFactory](https://www.nuget.org/packages/Grpc.Net.ClientFactory)：gRPC 客户端与 `HttpClientFactory` 的集成。

有关详细信息，请参阅 <xref:grpc/index>。

## SignalR

有关迁移说明，请参阅[更新 ](xref:migration/22-to-30#signalr) 代码SignalR。 SignalR 现在使用 `System.Text.Json` 来序列化/反序列化 JSON 消息。 有关还原基于 `Newtonsoft.Json` 的序列化程序的说明，请参阅[切换到 Newtonsoft.Json](xref:migration/22-to-30#switch-to-newtonsoftjson)。

在 SignalR 的 JavaScript 和 .NET 客户端中，添加了自动重新连接支持。 默认情况下，客户端会立即尝试重新连接，并根据需要分别在 2 秒、10 秒和 30 秒后重试。 如果客户端成功重新连接，则会收到新的连接 ID。 选择启用自动重新连接：

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withAutomaticReconnect()
    .build();
```

可以通过传递基于毫秒的持续时间数组来指定重新连接间隔：

```javascript
.withAutomaticReconnect([0, 3000, 5000, 10000, 15000, 30000])
//.withAutomaticReconnect([0, 2000, 10000, 30000]) The default intervals.
```

可以通过传入自定义实现来完全控制重新连接间隔。

如果在上一次重新连接间隔后重新连接失败：

* 客户端认为连接处于脱机状态。
* 客户端将停止尝试重新连接。

尝试重新连接时，更新应用 UI，通知用户正在尝试重新连接。

为了在连接中断时提供 UI 反馈，SignalR 客户端 API 已扩展为包含以下事件处理程序：

* `onreconnecting`：使开发人员有机会禁用 UI 或允许用户了解应用程序处于脱机状态。
* `onreconnected`：使开发人员有机会在重新建立连接后更新 UI。

以下代码在尝试连接时使用 `onreconnecting` 更新 UI：

```javascript
connection.onreconnecting((error) => {
    const status = `Connection lost due to error "${error}". Reconnecting.`;
    document.getElementById("messageInput").disabled = true;
    document.getElementById("sendButton").disabled = true;
    document.getElementById("connectionStatus").innerText = status;
});
```

以下代码在连接时使用 `onreconnected` 更新 UI：

```javascript
connection.onreconnected((connectionId) => {
    const status = `Connection reestablished. Connected.`;
    document.getElementById("messageInput").disabled = false;
    document.getElementById("sendButton").disabled = false;
    document.getElementById("connectionStatus").innerText = status;
});
```

当中心方法要求授权时，SignalR 3.0 和更高版本为授权处理程序提供了一个自定义资源。 资源是 `HubInvocationContext` 的一个实例。 `HubInvocationContext` 包括：

* `HubCallerContext`
* 正在调用的中心方法的名称。
* 中心方法的参数。

请考虑以下通过 Azure Active Directory 允许多个组织登录的聊天室应用示例。 拥有 Microsoft 帐户的任何人都可以登录到聊天，但只有所属组织的成员才能阻止用户或查看用户的聊天历史记录。 该应用可以限制特定用户的某些功能。

```csharp
public class DomainRestrictedRequirement :
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>,
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement,
        HubInvocationContext resource)
    {
        if (context.User?.Identity?.Name == null)
        {
            return Task.CompletedTask;
        }

        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name))
        {
            context.Succeed(requirement);
        }

        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName, string currentUsername)
    {
        if (hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase))
        {
            return currentUsername.Equals("bob42@jabbr.net", StringComparison.OrdinalIgnoreCase);
        }

        return currentUsername.EndsWith("@jabbr.net", StringComparison.OrdinalIgnoreCase));
    }
}
```

在前面的代码中，`DomainRestrictedRequirement` 作为自定义 `IAuthorizationRequirement`。 由于正在传入 `HubInvocationContext` 资源参数，因此内部逻辑可以：

* 检查在其中调用中心的上下文。
* 做出允许用户执行单个中心方法的决策。

可以通过代码在运行时检查的策略的名称来标记单个中心方法。 当客户端尝试调用单个中心方法时，`DomainRestrictedRequirement` 处理程序将运行并控制对方法的访问。 基于 `DomainRestrictedRequirement` 控制访问的方式：

* 所有已登录的用户都可以调用 `SendMessage` 方法。
* 只有使用 `@jabbr.net` 电子邮件地址登录的用户才能查看用户的历史记录。
* 只有 `bob42@jabbr.net` 才能阻止用户进入聊天室。

```csharp
[Authorize]
public class ChatHub : Hub
{
    public void SendMessage(string message)
    {
    }

    [Authorize("DomainRestricted")]
    public void BanUser(string username)
    {
    }

    [Authorize("DomainRestricted")]
    public void ViewUserHistory(string username)
    {
    }
}
```

创建 `DomainRestricted` 策略可能涉及：

* 在 Startup.cs 中，添加新策略。
* 提供自定义 `DomainRestrictedRequirement` 要求作为参数。
* 向授权中间件注册 `DomainRestricted`。

```csharp
services
    .AddAuthorization(options =>
    {
        options.AddPolicy("DomainRestricted", policy =>
        {
            policy.Requirements.Add(new DomainRestrictedRequirement());
        });
    });
```

SignalR 中心使用[终结点路由](xref:fundamentals/routing)。 以前 SignalR 中心连接是显式完成的：

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("hubs/chat");
});
```

在以前的版本中，开发人员需要将控制器、Razor 页面和中心连接到各种位置。 显式连接会生成一系列几乎相同的路由段：

```csharp
app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("hubs/chat");
});

app.UseRouting(routes =>
{
    routes.MapRazorPages();
});
```

可以通过终结点路由来路由 SignalR 3.0 中心。 通过终结点路由，通常可以在 `UseRouting` 中配置所有路由：

```csharp
app.UseRouting(routes =>
{
    routes.MapRazorPages();
    routes.MapHub<ChatHub>("hubs/chat");
});
```

ASP.NET Core 3.0 SignalR 添加了：

客户端到服务器的流式处理。 使用客户端到服务器的流式处理时，服务器端方法可以获取 `IAsyncEnumerable<T>` 或 `ChannelReader<T>` 的实例。 在下面的 C# 示例中，中心上的 `UploadStream` 方法将从客户端接收字符串流：

```csharp
public async Task UploadStream(IAsyncEnumerable<string> stream)
{
    await foreach (var item in stream)
    {
        // process content
    }
}
```

.NET 客户端应用程序可以将 `IAsyncEnumerable<T>` 或 `ChannelReader<T>` 实例作为上述 `UploadStream` 中心方法的 `stream` 参数传递。

`for` 循环完成并且本地函数退出后，将发送流完成：

```csharp
async IAsyncEnumerable<string> clientStreamData()
{
    for (var i = 0; i < 5; i++)
    {
        var data = await FetchSomeData();
        yield return data;
    }
}

await connection.SendAsync("UploadStream", clientStreamData());
```

JavaScript 客户端应用将 SignalR `Subject`（或 [RxJS 主题](https://rxjs.dev/api/index/class/Subject)）用于上面的 `UploadStream` 中心方法的 `stream` 参数。

```javascript
let subject = new signalR.Subject();
await connection.send("StartStream", "MyAsciiArtStream", subject);
```

当字符串被捕获并准备发送给服务器时，JavaScript 代码可以使用 `subject.next` 方法处理字符串。

```javascript
subject.next("example");
subject.complete();
```

使用类似于上述两个代码片段的代码，可创建实时流式处理体验。

## <a name="new-json-serialization"></a>新的 JSON 序列化

ASP.NET Core 3.0 现在默认使用 <xref:System.Text.Json> 进行 JSON 序列化：

* 以异步方式读取和写入 JSON。
* 针对 UTF-8 文本进行了优化。
* 通常比 `Newtonsoft.Json` 性能更高。

若要将 Json.NET 添加到 ASP.NET Core 3.0，请参阅[添加基于 Newtonsoft.Json 的 JSON 格式支持](xref:web-api/advanced/formatting#add-newtonsoftjson-based-json-format-support)。

## <a name="new-no-locrazor-directives"></a>新的 Razor 指令

下面的列表包含新的 Razor 指令：

* [`@attribute`](xref:mvc/views/razor#attribute)：`@attribute` 指令将给定的属性应用于生成的页或视图的类。 例如 `@attribute [Authorize]`。
* [`@implements`](xref:mvc/views/razor#implements)：`@implements` 指令为生成的类实现接口。 例如 `@implements IDisposable`。

## <a name="no-locidentityserver4-supports-authentication-and-authorization-for-web-apis-and-spas"></a>IdentityServer4 支持 Web API 和 SPA 的身份验证和授权

ASP.NET Core 3.0 使用 Web API 授权的支持在单页应用 (SPA) 中提供身份验证。 ASP.NET Core Identity 与 [IdentityServer4](https://identityserver.io/) 结合使用，前者用于验证和存储用户身份信息，后者用于实现 OpenID Connect。

IdentityServer4 是适用于 ASP.NET Core 3.0 的 OpenID Connect 和 OAuth 2.0 框架。 它提供了以下安全功能：

* 身份验证即服务 (AaaS)
* 跨多个应用程序类型的单一登录/注销 (SSO)
* API 的访问控制
* Federation Gateway

有关详细信息，请参阅 [IdentityServer4 文档](http://docs.identityserver.io/en/latest/index.html)或 [SPA 的身份验证和授权](xref:security/authentication/identity/spa)。

## <a name="certificate-and-kerberos-authentication"></a>证书和 Kerberos 身份验证

证书身份验证需要：

* 将服务器配置为接受证书。
* 在 `Startup.Configure` 中添加身份验证中间件。
* 将证书身份验证服务添加到 `Startup.ConfigureServices`。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate();
    // Other service configuration removed.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // Other app configuration removed.
}
```

用于证书身份验证的选项包括执行以下操作的能力：

* 接受自签名证书。
* 检查证书吊销。
* 检查提供证书中是否有正确的使用标志。

默认的用户主体是从证书属性构造的。 用户主体包含允许补充或替换主体的事件。 有关详细信息，请参阅 <xref:security/authentication/certauth>。

[Windows 身份验证](/windows-server/security/windows-authentication/windows-authentication-overview)已扩展到 Linux 和 macOS。 在以前的版本中，Windows 身份验证限制为 [IIS](xref:host-and-deploy/iis/index) 和 [HttpSys](xref:fundamentals/servers/httpsys)。 在 ASP.NET Core 3.0 中，[Kestrel](xref:fundamentals/servers/kestrel) 能够在 Windows、Linux 和 macOS 上对已加入 Windows 域的主机使用 Negotiate、[Kerberos](/windows-server/security/kerberos/kerberos-authentication-overview) 和 [NTLM](/windows-server/security/kerberos/ntlm-overview)。 Kestrel 对这些身份验证方案的支持由 [Microsoft.AspNetCore.Authentication.Negotiate NuGet](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Negotiate) 包提供。 对于其他身份验证服务，请配置身份验证应用范围，然后配置服务：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(NegotiateDefaults.AuthenticationScheme)
        .AddNegotiate();
    // Other service configuration removed.
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    // Other app configuration removed.
}
```

主机要求：

* Windows 主机必须将[主体名称](/windows/win32/ad/service-principal-names) (SPN) 添加到托管应用的用户帐户。
* Linux 和 macOS 计算机必须加入域。
  * 必须为 Web 进程创建 SPN。
  * 必须在主机上生成并配置 [Keytab 文件](/archive/blogs/pie/all-you-need-to-know-about-keytab-files)。

有关详细信息，请参阅 <xref:security/authentication/windowsauth>。

## <a name="template-changes"></a>模板更改

Web UI 模板（Razor Pages、具有控制器和视图的 MVC）已删除以下内容：

* cookie 同意 UI 不再包括在内。 若要在 ASP.NET Core 3.0 模板生成的应用中启用 cookie 同意功能，请参阅 <xref:security/gdpr>。
* 脚本和相关静态资产现在作为本地文件（而不是使用 CDN）进行引用。 有关详细信息，[脚本和相关静态资产现在基于当前环境作为本地文件（而不是使用 CDN）进行引用 (aspnet/AspNetCore.Docs #14350)](https://github.com/dotnet/AspNetCore.Docs/issues/14350)。

Angular 模板已更新，以便使用 Angular 8。

默认情况下，Razor 类库 (RCL) 模板默认为 Razor 组件开发。 Visual Studio 中的新模板选项提供了对页面和视图的模板支持。 在命令行界面中通过模板创建 RCL 时，请传递 `--support-pages-and-views` 选项 (`dotnet new razorclasslib --support-pages-and-views`)。

## <a name="generic-host"></a>泛型主机

ASP.NET Core 3.0 模板使用 <xref:fundamentals/host/generic-host>。 以前版本使用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder>。 使用 .NET Core 泛型主机 (<xref:Microsoft.Extensions.Hosting.HostBuilder>) 可以更好地将 ASP.NET Core 应用与非 Web 特定的其他服务器方案集成。 有关详细信息，请参阅 [HostBuilder 替换 WebHostBuilder](xref:migration/22-to-30?view=aspnetcore-2.2#hostbuilder-replaces-webhostbuilder)。

### <a name="host-configuration"></a>主机配置

在 ASP.NET Core 3.0 版本之前，为 Web 主机的主机配置加载了前缀为 `ASPNETCORE_` 的环境变量。 在 3.0 中，对于带有 `CreateDefaultBuilder` 的主机配置，使用 `AddEnvironmentVariables` 加载以 `DOTNET_` 为前缀的环境变量。

### <a name="changes-to-startup-constructor-injection"></a>对启动构造函数注入的更改

泛型主机仅支持以下类型的 `Startup` 构造函数注入：

* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

所有服务仍可以直接作为 `Startup.Configure` 方法的参数注入。 有关详细信息，请参阅[泛型主机限制启动构造函数注入 (aspnet/Announcements #353)](https://github.com/aspnet/Announcements/issues/353)。

## <a name="kestrel"></a>Kestrel

* 已更新 Kestrel 配置以迁移到泛型主机。 在 3.0 中，Kestrel 是在 `ConfigureWebHostDefaults` 提供的 Web 主机生成器上配置的。
* 已从 Kestrel 中删除连接适配器，并将其替换为连接中间件，这与 ASP.NET Core 管道中的 HTTP 中间件类似，但适用于较低级别的连接。
* Kestrel 传输层已作为 `Connections.Abstractions` 中的公共接口公开。
* 通过将尾随标题移到新集合中，已解决了标头和尾部之间的歧义。
* 线程不足会导致应用崩溃，而同步 I/O API（例如 `HttpRequest.Body.Read`）是导致线程不足的常见原因。 在 3.0 中，默认情况下禁用 `AllowSynchronousIO`。

有关详细信息，请参阅 <xref:migration/22-to-30#kestrel>。

## <a name="http2-enabled-by-default"></a>默认情况下启用 HTTP/2

默认情况下，在 HTTPS 终结点的 Kestrel 中启用 HTTP/2。 当操作系统支持 HTTP/2 时，对 IIS 或 HTTP.sys 启用 HTTP/2 支持。

## <a name="eventcounters-on-request"></a>根据请求提供事件计数器

托管事件源 `Microsoft.AspNetCore.Hosting` 发出以下与传入请求有关的新 <xref:System.Diagnostics.Tracing.EventCounter> 类型：

* `requests-per-second`
* `total-requests`
* `current-requests`
* `failed-requests`

## <a name="endpoint-routing"></a>终结点路由

增强了终结点路由，终结点路由可以让框架（例如 MVC）与中间件配合使用：

* 中间件和终结点的顺序可在 `Startup.Configure` 的请求处理管道中进行配置。
* 终结点和中间件与其他基于 ASP.NET Core 的技术（如运行状况检查）很好地结合。
* 终结点可以在中间件和 MVC 中实现策略，例如 CORS 或授权。
* 可以将筛选器和属性置于控制器的方法中。

有关详细信息，请参阅 <xref:fundamentals/routing#routing-basics>。

## <a name="health-checks"></a>运行状况检查

运行状况检查将终结点路由与泛型主机一起使用。 在 `Startup.Configure` 内，使用终结点 URL 或相对路径在终结点生成器上调用 `MapHealthChecks`：

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapHealthChecks("/health");
});
```

运行状况检查终结点可以：

* 指定一个或多个允许的主机/端口。
* 需要授权。
* 需要 CORS。

有关详细信息，请参阅以下文章：

* <xref:migration/22-to-30#health-checks>
* <xref:host-and-deploy/health-checks>

## <a name="pipes-on-httpcontext"></a>HttpContext 上的管道

现在可以使用 <xref:System.IO.Pipelines> API 读取请求正文和写入响应正文。 必须向 <!-- <xref:Microsoft.AspNetCore.Http.HttpRequest.BodyReader> --> `HttpRequest.BodyReader` 属性提供可用于读取请求正文的 <xref:System.IO.Pipelines.PipeReader>。 必须向 <!-- <xref:Microsoft.AspNetCore.Http.> --> `HttpResponse.BodyWriter` 属性提供可用于写入响应正文的 <xref:System.IO.Pipelines.PipeWriter>。 `HttpRequest.BodyReader` 是 `HttpRequest.Body` 流的模拟。 `HttpResponse.BodyWriter` 是 `HttpResponse.Body` 流的模拟。

<!-- indirectly related, https://github.com/dotnet/docs/pull/14414 won't be published by 9/23  -->

## <a name="improved-error-reporting-in-iis"></a>改进了 IIS 中的错误报告

在 IIS 中托管 ASP.NET Core 应用时出现的启动错误现在会生成更丰富的诊断数据。 如果适用，这些错误将报告给具有堆栈跟踪的 Windows 事件日志。 此外，所有警告、错误和未经处理的异常都将记录到 Windows 事件日志中。

## <a name="worker-service-and-worker-sdk"></a>辅助角色服务和辅助角色 SDK

.NET Core 3.0 引入了新的辅助角色服务应用模板。 可根据此模板开始在 .NET Core 中编写长期运行的服务。

有关详情，请参阅：

* [作为 Windows 服务的 .NET Core 辅助角色](https://devblogs.microsoft.com/aspnet/net-core-workers-as-windows-services/)
* <xref:fundamentals/host/hosted-services>
* <xref:host-and-deploy/windows-service>

## <a name="forwarded-headers-middleware-improvements"></a>转接标头中间件改进

在以前版本的 ASP.NET Core 中，在部署到 Azure Linux 或除 IIS 以外的任何反向代理后，调用 <xref:Microsoft.AspNetCore.Builder.HstsBuilderExtensions.UseHsts*> 和 <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*> 会出现问题。 [转发 Linux 和非 IIS 反向代理的方案](xref:host-and-deploy/proxy-load-balancer#forward-the-scheme-for-linux-and-non-iis-reverse-proxies)记录了以前版本的修复。

此方案已在 ASP.NET Core 3.0 中修复。 如果 `ASPNETCORE_FORWARDEDHEADERS_ENABLED` 环境变量设置为 `true`，则主机启用[转接的标头中间件](xref:host-and-deploy/proxy-load-balancer#forwarded-headers-middleware-options)。 在容器映像中，`ASPNETCORE_FORWARDEDHEADERS_ENABLED` 设置为 `true`。

## <a name="performance-improvements"></a>性能改进

ASP.NET Core 3.0 包含了许多改进，可减少内存使用量并提高吞吐量：

* 降低了使用内置的依赖项注入容器来实现作用域服务时的内存使用量。
* 减少跨框架的分配，包括中间件方案和路由。
* 降低了 WebSocket 连接的内存使用量。
* 减少 HTTPS 连接的内存使用量并提高了其吞吐量。
* 新的优化和完全异步 JSON 序列化程序。
* 减少了窗体分析的内存使用量并提高了其吞吐量。

## <a name="aspnet-core-30-only-runs-on-net-core-30"></a>ASP.NET Core 3.0 仅在 .NET Core 3.0 上运行

从 ASP.NET Core 3.0 开始，.NET Framework 不再是受支持的目标框架。 面向 .NET Framework 的项目可以使用 [.NET Core 2.1 LTS 版本](https://dotnet.microsoft.com/download/dotnet-core/2.1)以完全受支持的方式继续。 超过 .NET Core 2.1 的 3 年 LTS 期之后，将无限期地向大多数 ASP.NET Core 2.1.x 相关包提供支持。

有关迁移信息，请参阅[将代码从 .NET Framework 移植到 .NET Core](/dotnet/core/porting/)。

## <a name="use-the-aspnet-core-shared-framework"></a>使用 ASP.NET Core 共享框架

[Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) 中包含的 ASP.NET Core 3.0 共享框架不再需要项目文件中有一个显式 `<PackageReference />` 元素。 使用项目文件中的 `Microsoft.NET.Sdk.Web` SDK 时，会自动引用共享框架：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

## <a name="assemblies-removed-from-the-aspnet-core-shared-framework"></a>从 ASP.NET Core 共享框架中删除的程序集

从 ASP.NET Core 3.0 共享框架中删除的最值得注意的程序集有：

* [Newtonsoft.Json](https://www.nuget.org/packages/Newtonsoft.Json/) (Json.NET)。 若要将 Json.NET 添加到 ASP.NET Core 3.0，请参阅[添加基于 Newtonsoft.Json 的 JSON 格式支持](xref:web-api/advanced/formatting#add-newtonsoftjson-based-json-format-support)。 ASP.NET Core 3.0 引入了 `System.Text.Json` 以读取和写入 JSON。 有关详细信息，请参阅本文档中的[新 JSON 序列化](#new-json-serialization)。
* [Entity Framework Core](/ef/core/)

有关从共享框架中删除的程序集的完整列表，请参阅[从 Microsoft.AspNetCore.App 3.0 中删除的程序集](https://github.com/dotnet/AspNetCore/issues/3755)。 有关此更改的动机的详细信息，请参阅 [3.0 中对 Microsoft.AspNetCore.App 所做的重大变更](https://github.com/aspnet/Announcements/issues/325)和[首先查看 ASP.NET Core 3.0 中的变更](https://devblogs.microsoft.com/aspnet/a-first-look-at-changes-coming-in-asp-net-core-3-0/)。

<!-- 
## Additional information
For the complete list of changes, see the [ASP.NET Core 3.0 Release Notes](WHERE IS THIS????).
-->
