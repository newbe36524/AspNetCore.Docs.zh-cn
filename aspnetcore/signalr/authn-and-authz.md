---
title: ASP.NET Core 中的身份验证和授权 SignalR
author: bradygaster
description: 了解如何在 ASP.NET Core 中使用身份验证和授权 SignalR 。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 12/05/2019
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
uid: signalr/authn-and-authz
ms.openlocfilehash: 0e220d72fe9ef4ada402b449ef20e31324f7bcd2
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060113"
---
# <a name="authentication-and-authorization-in-aspnet-core-no-locsignalr"></a>ASP.NET Core 中的身份验证和授权 SignalR

作者： [Andrew Stanton](https://twitter.com/anurse)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/authn-and-authz/sample/)[（如何下载）](xref:index#how-to-download-a-sample)

## <a name="authenticate-users-connecting-to-a-no-locsignalr-hub"></a>对连接到集线器的用户进行身份验证 SignalR

SignalR 可以与 [ASP.NET Core authentication](xref:security/authentication/identity) 一起使用，以将用户与每个连接相关联。 在中心中，可以从 [HubConnectionContext](/dotnet/api/microsoft.aspnetcore.signalr.hubconnectioncontext.user) 属性访问身份验证数据。 身份验证允许中心对与用户关联的所有连接调用方法。 有关详细信息，请参阅中的 "[管理 SignalR 用户和组" ](xref:signalr/groups)。 单个用户可以关联多个连接。

下面是 `Startup.Configure` 使用 SignalR 和 ASP.NET Core 身份验证的示例：

::: moniker range=">= aspnetcore-3.0"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chat");
        endpoints.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

```csharp
public void Configure(IApplicationBuilder app)
{
    ...

    app.UseStaticFiles();

    app.UseAuthentication();

    app.UseSignalR(hubs =>
    {
        hubs.MapHub<ChatHub>("/chat");
    });

    app.UseMvc(routes =>
    {
        routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
```

> [!NOTE]
> 注册 SignalR 和 ASP.NET Core 身份验证中间件的顺序。 始终调用 `UseAuthentication` ， `UseSignalR` 以便在 SignalR 上有用户 `HttpContext` 。

::: moniker-end

### <a name="no-loccookie-authentication"></a>Cookie身份验证

在基于浏览器的应用程序中， cookie 身份验证允许现有用户凭据自动流动到 SignalR 连接。 使用浏览器客户端时，无需进行其他配置。 如果用户已登录到你的应用，则 SignalR 连接将自动继承此身份验证。

Cookies 是一种特定于浏览器的发送访问令牌的方法，但非浏览器客户端可以发送这些令牌。 使用 [.Net 客户端](xref:signalr/dotnet-client)时， `Cookies` 可以在调用中配置属性 `.WithUrl` 以提供 cookie 。 但是，在 cookie .net 客户端中使用身份验证要求应用提供一个 API 来交换的身份验证数据 cookie 。

### <a name="bearer-token-authentication"></a>持有者令牌身份验证

客户端可以提供访问令牌，而不是使用 cookie 。 服务器验证令牌并使用它来标识用户。 仅在建立连接时才执行此验证。 在连接的生命周期内，服务器不会自动重新验证以检查令牌是否已吊销。

在 JavaScript 客户端中，可使用 [accessTokenFactory](xref:signalr/configuration#configure-bearer-authentication) 选项提供令牌。

[!code-typescript[Configure Access Token](authn-and-authz/sample/wwwroot/js/chat.ts?range=52-55)]

在 .NET 客户端中，有一个类似的 [AccessTokenProvider](xref:signalr/configuration#configure-bearer-authentication) 属性，可用于配置令牌：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    { 
        options.AccessTokenProvider = () => Task.FromResult(_myAccessToken);
    })
    .Build();
```

> [!NOTE]
> 提供的访问令牌函数在发出的 **每个** HTTP 请求之前调用 SignalR 。 如果你需要续订标记以便保持连接处于活动状态， (因为它可能会在连接) 期间过期，请在此函数中执行此操作，并返回已更新的令牌。

在标准 web Api 中，持有者令牌是在 HTTP 标头中发送的。 但是，在 SignalR 使用某些传输时，无法在浏览器中设置这些标头。 使用 Websocket 和 Server-Sent 事件时，会将令牌作为查询字符串参数进行传输。 

#### <a name="built-in-jwt-authentication"></a>内置的 JWT 身份验证

在服务器上，使用 [JWT 持有者中间件](xref:Microsoft.Extensions.DependencyInjection.JwtBearerExtensions.AddJwtBearer%2A)配置持有者令牌身份验证：

[!code-csharp[Configure Server to accept access token from Query String](authn-and-authz/sample/Startup.cs?name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

> [!NOTE]
> 由于浏览器 API 限制，连接到 Websocket 和 Server-Sent 事件时，将在浏览器上使用查询字符串。 使用 HTTPS 时，查询字符串值受 TLS 连接保护。 但是，许多服务器都记录查询字符串值。 有关详细信息，请参阅[ASP.NET Core SignalR 中的安全注意事项](xref:signalr/security)。 SignalR 使用标头在支持 (如 .NET 和 Java 客户端) 的环境中传输标记。

#### <a name="no-locidentity-server-jwt-authentication"></a>Identity 服务器 JWT 身份验证

使用 Identity 服务器时，将 <xref:Microsoft.Extensions.Options.PostConfigureOptions%601> 服务添加到项目：

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.Extensions.Options;
public class ConfigureJwtBearerOptions : IPostConfigureOptions<JwtBearerOptions>
{
    public void PostConfigure(string name, JwtBearerOptions options)
    {
        var originalOnMessageReceived = options.Events.OnMessageReceived;
        options.Events.OnMessageReceived = async context =>
        {
            await originalOnMessageReceived(context);
                
            if (string.IsNullOrEmpty(context.Token))
            {
                var accessToken = context.Request.Query["access_token"];
                var path = context.HttpContext.Request.Path;
                
                if (!string.IsNullOrEmpty(accessToken) && 
                    path.StartsWithSegments("/hubs"))
                {
                    context.Token = accessToken;
                }
            }
        };
    }
}
```

将服务添加到 `Startup.ConfigureServices` 身份验证 (<xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication%2A>) 和 Identity 服务器 () 的身份验证处理程序之后，请在中注册该服务 <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> ：

```csharp
services.AddAuthentication()
    .AddIdentityServerJwt();
services.TryAddEnumerable(
    ServiceDescriptor.Singleton<IPostConfigureOptions<JwtBearerOptions>, 
        ConfigureJwtBearerOptions>());
```

### <a name="no-loccookies-vs-bearer-tokens"></a>Cookie和持有者令牌 

Cookie是特定于浏览器的。 与发送持有者令牌相比，从其他类型的客户端发送这些客户端增加了复杂性。 因此， cookie 除非应用只需从浏览器客户端对用户进行身份验证，否则不建议进行身份验证。 当使用浏览器客户端之外的客户端时，建议使用持有者令牌身份验证。

### <a name="windows-authentication"></a>Windows 身份验证

如果在你的应用中配置了 [Windows 身份验证](xref:security/authentication/windowsauth) ，则 SignalR 可以使用该标识来保护中心。 但是，若要将消息发送给单个用户，则需要添加自定义用户 ID 提供程序。 Windows 身份验证系统不提供 "名称标识符" 声明。 SignalR 使用声明来确定用户名。

添加一个新类，该类实现 `IUserIdProvider` 并检索用户要用作标识符的声明之一。 例如，若要使用 "名称" 声明 (是) 格式的 Windows 用户名 `[Domain]\[Username]` ，请创建以下类：

[!code-csharp[Name based provider](authn-and-authz/sample/nameuseridprovider.cs?name=NameUserIdProvider)]

`ClaimTypes.Name`您可以使用 (中的任何值 `User` ，例如 Windows SID 标识符，等等) 。

> [!NOTE]
> 您选择的值在系统中的所有用户中必须是唯一的。 否则，用于一个用户的消息可能最终会转到其他用户。

在方法中注册此组件 `Startup.ConfigureServices` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services.AddSignalR();
    services.AddSingleton<IUserIdProvider, NameUserIdProvider>();
}
```

在 .NET 客户端中，必须通过设置 [UseDefaultCredentials](/dotnet/api/microsoft.aspnetcore.http.connections.client.httpconnectionoptions.usedefaultcredentials) 属性来启用 Windows 身份验证：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options =>
    {
        options.UseDefaultCredentials = true;
    })
    .Build();
```

Internet Explorer 和 Microsoft Edge 支持 Windows 身份验证，但在所有浏览器中都不支持。 例如，在 Chrome 和 Safari 中，尝试使用 Windows 身份验证和 Websocket 会失败。 当 Windows 身份验证失败时，客户端将尝试回退到可能工作的其他传输。

### <a name="use-claims-to-customize-identity-handling"></a>使用声明自定义标识处理

对用户进行身份验证的应用可以 SignalR 从用户声明派生用户 id。 若要指定 SignalR 创建用户 id 的方式，请实现 `IUserIdProvider` 并注册实现。

示例代码演示了如何使用声明选择用户的电子邮件地址作为识别属性。 

> [!NOTE]
> 您选择的值在系统中的所有用户中必须是唯一的。 否则，用于一个用户的消息可能最终会转到其他用户。

[!code-csharp[Email provider](authn-and-authz/sample/EmailBasedUserIdProvider.cs?name=EmailBasedUserIdProvider)]

帐户注册会将类型为的声明添加 `ClaimsTypes.Email` 到 ASP.NET 标识数据库。

[!code-csharp[Adding the email to the ASP.NET identity claims](authn-and-authz/sample/pages/account/Register.cshtml.cs?name=AddEmailClaim)]

在中注册此组件 `Startup.ConfigureServices` 。

```csharp
services.AddSingleton<IUserIdProvider, EmailBasedUserIdProvider>();
```

## <a name="authorize-users-to-access-hubs-and-hub-methods"></a>授权用户访问集线器和集线器方法

默认情况下，集线器中的所有方法都可由未经身份验证的用户调用。 若要要求身份验证，请将 [授权](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) 属性应用于中心：

[!code-csharp[Restrict a hub to only authorized users](authn-and-authz/sample/Hubs/ChatHub.cs?range=8-10,32)]

可使用 `[Authorize]` 特性的构造函数参数和属性将访问权限仅限于匹配特定[授权策略](xref:security/authorization/policies)的用户。 例如，如果你有一个名为的自定义授权策略， `MyAuthorizationPolicy` 则可以确保只有符合该策略的用户才能使用以下代码访问该中心：

```csharp
[Authorize("MyAuthorizationPolicy")]
public class ChatHub : Hub
{
}
```

单个集线器方法 `[Authorize]` 也可以应用该属性。 如果当前用户与应用于方法的策略不匹配，则会向调用方返回错误：

```csharp
[Authorize]
public class ChatHub : Hub
{
    public async Task Send(string message)
    {
        // ... send a message to all users ...
    }

    [Authorize("Administrators")]
    public void BanUser(string userName)
    {
        // ... ban a user from the chat room (something only Administrators can do) ...
    }
}
```

::: moniker range=">= aspnetcore-3.0"

### <a name="use-authorization-handlers-to-customize-hub-method-authorization"></a>使用授权处理程序自定义集线器方法授权

SignalR 当集线器方法要求授权时，为授权处理程序提供自定义资源。 资源是 `HubInvocationContext` 的一个实例。 `HubInvocationContext`包括 `HubCallerContext` 、正在调用的集线器方法的名称，以及中心方法的参数。

请考虑允许通过 Azure Active Directory 多个组织登录的聊天室的示例。 拥有 Microsoft 帐户的任何人都可以登录到聊天，但只有拥有组织的成员才能阻止用户或查看用户的聊天历史记录。 而且，我们可能希望限制某些用户的某些功能。 使用 ASP.NET Core 3.0 中的更新功能，这是完全可能的。 请注意如何 `DomainRestrictedRequirement` 充当自定义 `IAuthorizationRequirement` 。 既然 `HubInvocationContext` 正在传入资源参数，内部逻辑就可以检查正在调用中心的上下文，并决定是否允许用户执行单个集线器方法。

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

public class DomainRestrictedRequirement : 
    AuthorizationHandler<DomainRestrictedRequirement, HubInvocationContext>, 
    IAuthorizationRequirement
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context,
        DomainRestrictedRequirement requirement, 
        HubInvocationContext resource)
    {
        if (IsUserAllowedToDoThis(resource.HubMethodName, context.User.Identity.Name) && 
            context.User.Identity.Name.EndsWith("@microsoft.com"))
        {
            context.Succeed(requirement);
        }
        return Task.CompletedTask;
    }

    private bool IsUserAllowedToDoThis(string hubMethodName,
        string currentUsername)
    {
        return !(currentUsername.Equals("asdf42@microsoft.com") && 
            hubMethodName.Equals("banUser", StringComparison.OrdinalIgnoreCase));
    }
}
```

在中 `Startup.ConfigureServices` ，添加新策略，并将自定义 `DomainRestrictedRequirement` 要求用作创建策略的参数 `DomainRestricted` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // ... other services ...

    services
        .AddAuthorization(options =>
        {
            options.AddPolicy("DomainRestricted", policy =>
            {
                policy.Requirements.Add(new DomainRestrictedRequirement());
            });
        });
}
```

在前面的示例中， `DomainRestrictedRequirement` 类既是 `IAuthorizationRequirement` 又是 `AuthorizationHandler` 该要求的。 可以将这两个组件拆分为单独的类，以分隔问题。 该示例方法的优点是，无需在 `AuthorizationHandler` 启动过程中注入，因为要求和处理程序是相同的。

::: moniker-end

## <a name="additional-resources"></a>其他资源

* [ASP.NET Core 中的持有者令牌身份验证](https://blogs.msdn.microsoft.com/webdev/2016/10/27/bearer-token-authentication-in-asp-net-core/)
* [基于资源的授权](xref:security/authorization/resourcebased)
