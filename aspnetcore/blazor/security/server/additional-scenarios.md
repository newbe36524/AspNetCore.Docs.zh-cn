---
title: ASP.NET Core Blazor Server 其他安全方案
author: guardrex
description: 了解如何为其他安全方案配置 Blazor Server。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/06/2020
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
uid: blazor/security/server/additional-scenarios
ms.openlocfilehash: 89288f3fce2dbb6f2647693ba8aaf29500b5bb2b
ms.sourcegitcommit: 139c998d37e9f3e3d0e3d72e10dbce8b75957d89
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/07/2020
ms.locfileid: "91805487"
---
# <a name="aspnet-core-no-locblazor-server-additional-security-scenarios"></a>ASP.NET Core Blazor Server 其他安全方案

作者：[Javier Calvarro Nelson](https://github.com/javiercn)

::: moniker range=">= aspnetcore-5.0"

<h2 id="pass-tokens-to-a-blazor-server-app">将令牌传递到 Blazor Server 应用</h2>

可以使用本节中介绍的方法将 Blazor Server 应用中 Razor 组件外部可用的令牌传递给组件。

与对常规 Razor Pages 或 MVC 应用进行身份验证一样，对 Blazor Server 应用进行身份验证。 预配令牌并将其保存到身份验证 cookie。 例如：

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
});
```

可选择使用 `options.Scope.Add("{SCOPE}");` 添加其他作用域，其中占位符 `{SCOPE}` 是要添加的其他作用域。

定义可在 Blazor 应用中使用的作用域令牌提供程序服务，以解析[依赖项注入 (DI)](xref:blazor/fundamentals/dependency-injection) 中的令牌：

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

在 `Startup.ConfigureServices` 中，为以下对象添加服务：

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

定义一个类，以在初始应用状态下使用访问令牌和刷新令牌传递它：

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

在 `_Host.cshtml` 文件中，创建 `InitialApplicationState` 实例，并将其作为参数传递给应用：

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<component type="typeof(App)" param-InitialState="tokens" 
    render-mode="ServerPrerendered" />
```

在 `App` 组件 (`App.razor`) 中，解析服务并使用参数中的数据对其进行初始化：

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

为 [`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) NuGet 包添加对应用的包引用。

在发出安全 API 请求的服务中，注入令牌提供程序并检索 API 请求的令牌：

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class WeatherForecastService
{
    private readonly HttpClient client;
    private readonly TokenProvider tokenProvider;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        client = clientFactory.CreateClient();
        this.tokenProvider = tokenProvider;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var token = tokenProvider.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await client.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

<h2 id="set-the-authentication-scheme">设置身份验证方案</h2>

对于使用多个身份验证中间件并因此具有多个身份验证方案的应用，可以在 `Startup.Configure` 的终结点配置中显式设置 Blazor 使用的方案。 下面的示例设置 Azure Active Directory 方案：

```csharp
endpoints.MapBlazorHub().RequireAuthorization(
    new AuthorizeAttribute 
    {
        AuthenticationSchemes = AzureADDefaults.AuthenticationScheme
    });
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

<h2 id="pass-tokens-to-a-blazor-server-app">将令牌传递到 Blazor Server 应用</h2>

可以使用本节中介绍的方法将 Blazor Server 应用中 Razor 组件外部可用的令牌传递给组件。

与对常规 Razor Pages 或 MVC 应用进行身份验证一样，对 Blazor Server 应用进行身份验证。 预配令牌并将其保存到身份验证 cookie。 例如：

```csharp
using Microsoft.AspNetCore.Authentication.OpenIdConnect;
using Microsoft.IdentityModel.Protocols.OpenIdConnect;

...

services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options =>
{
    options.ResponseType = OpenIdConnectResponseType.Code;
    options.SaveTokens = true;

    options.Scope.Add("offline_access");
});
```

可选择使用 `options.Scope.Add("{SCOPE}");` 添加其他作用域，其中占位符 `{SCOPE}` 是要添加的其他作用域。

可选择使用 `options.Resource = "{RESOURCE}";` 指定资源，其中占位符 `{RESOURCE}` 即为资源。 例如：

```csharp
options.Resource = "https://graph.microsoft.com";
```

定义一个类，以在初始应用状态下使用访问令牌和刷新令牌传递它：

```csharp
public class InitialApplicationState
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

定义可在 Blazor 应用中使用的作用域令牌提供程序服务，以解析[依赖项注入 (DI)](xref:blazor/fundamentals/dependency-injection) 中的令牌：

```csharp
public class TokenProvider
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
```

在 `Startup.ConfigureServices` 中，为以下对象添加服务：

* `IHttpClientFactory`
* `TokenProvider`

```csharp
services.AddHttpClient();
services.AddScoped<TokenProvider>();
```

在 `_Host.cshtml` 文件中，创建 `InitialApplicationState` 实例，并将其作为参数传递给应用：

```cshtml
@using Microsoft.AspNetCore.Authentication

...

@{
    var tokens = new InitialApplicationState
    {
        AccessToken = await HttpContext.GetTokenAsync("access_token"),
        RefreshToken = await HttpContext.GetTokenAsync("refresh_token")
    };
}

<app>
    <component type="typeof(App)" param-InitialState="tokens" 
        render-mode="ServerPrerendered" />
</app>
```

在 `App` 组件 (`App.razor`) 中，解析服务并使用参数中的数据对其进行初始化：

```razor
@inject TokenProvider TokenProvider

...

@code {
    [Parameter]
    public InitialApplicationState InitialState { get; set; }

    protected override Task OnInitializedAsync()
    {
        TokenProvider.AccessToken = InitialState.AccessToken;
        TokenProvider.RefreshToken = InitialState.RefreshToken;

        return base.OnInitializedAsync();
    }
}
```

为 [`Microsoft.AspNet.WebApi.Client`](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Client) NuGet 包添加对应用的包引用。

在发出安全 API 请求的服务中，注入令牌提供程序并检索 API 请求的令牌：

```csharp
using System;
using System.Net.Http;
using System.Threading.Tasks;

public class WeatherForecastService
{
    private readonly HttpClient client;
    private readonly TokenProvider tokenProvider;

    public WeatherForecastService(IHttpClientFactory clientFactory, 
        TokenProvider tokenProvider)
    {
        client = clientFactory.CreateClient();
        this.tokenProvider = tokenProvider;
    }

    public async Task<WeatherForecast[]> GetForecastAsync()
    {
        var token = tokenProvider.AccessToken;
        var request = new HttpRequestMessage(HttpMethod.Get, 
            "https://localhost:5003/WeatherForecast");
        request.Headers.Add("Authorization", $"Bearer {token}");
        var response = await client.SendAsync(request);
        response.EnsureSuccessStatusCode();

        return await response.Content.ReadAsAsync<WeatherForecast[]>();
    }
}
```

<h2 id="set-the-authentication-scheme">设置身份验证方案</h2>

对于使用多个身份验证中间件并因此具有多个身份验证方案的应用，可以在 `Startup.Configure` 的终结点配置中显式设置 Blazor 使用的方案。 下面的示例设置 Azure Active Directory 方案：

```csharp
endpoints.MapBlazorHub().RequireAuthorization(
    new AuthorizeAttribute 
    {
        AuthenticationSchemes = AzureADDefaults.AuthenticationScheme
    });
```

## <a name="use-openid-connect-oidc-v20-endpoints"></a>使用 OpenID Connect (OIDC) v2.0 终结点

在 5.0 之前的 ASP.NET Core 版本中，身份验证库和 Blazor 模板使用 OpenID Connect (OIDC) v1.0 终结点。 若要在 5.0 以前的版本中使用 v2.0 终结点，请在 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> 中配置 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority?displayProperty=nameWithType> 选项：

```csharp
services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, 
    options =>
    {
        options.Authority += "/v2.0";
    }
```

也可以在应用设置 (`appsettings.json`) 文件中进行设置：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/common/oauth2/v2.0/",
    ...
  }
}
```

如果将段添加到授权不适合应用的 OIDC 提供程序（例如，使用非 AAD 提供程序），则直接设置 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> 属性。 使用 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Authority> 键在 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> 或应用设置文件中设置属性。

### <a name="code-changes"></a>代码更改

* ID 令牌中的声明列表针对 v2.0 终结点会发生更改。 有关详细信息，请参阅[为什么要更新到 Microsoft 标识平台 (v2.0)？](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison) （位于 Azure 文档中）。
* 由于在 v2.0 终结点的范围 URI 中指定了资源，请删除 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions> 中的 <xref:Microsoft.AspNetCore.Builder.OpenIdConnectOptions.Resource?displayProperty=nameWithType> 属性设置：

  ```csharp
  services.Configure<OpenIdConnectOptions>(AzureADDefaults.OpenIdScheme, options => 
      {
          ...
          options.Resource = "...";    // REMOVE THIS LINE
          ...
      }
  ```

  有关详细信息，请参阅 Azure 文档中的[作用域，非资源](/azure/active-directory/azuread-dev/azure-ad-endpoint-comparison#scopes-not-resources)部分。

### <a name="app-id-uri"></a>应用 ID URI

* 使用 v2.0 终结点时，API 会定义一个 `App ID URI`，来表示 API 的唯一标识符。
* 所有作用域都将应用 ID URI 用作前缀，v2.0 终结点以应用 ID URI 为受众发出访问令牌。
* 使用 V2.0 终结点时，服务器 API 中配置的客户端 ID 会从 API 应用程序 ID（客户端 ID）更改为应用 ID URI。

`appsettings.json`:

```json
{
  "AzureAd": {
    ...
    "ClientId": "https://{TENANT}.onmicrosoft.com/{APP NAME}"
    ...
  }
}
```

可以在 OIDC 提供程序应用注册说明中找到要使用的应用 ID URI。

::: moniker-end
