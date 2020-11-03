---
title: ASP.NET Core Blazor WebAssembly 与 Azure Active Directory 组和角色
author: guardrex
description: 了解如何配置 Blazor WebAssembly 以使用 Azure Active Directory 组和角色。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 07/28/2020
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
uid: blazor/security/webassembly/aad-groups-roles
ms.openlocfilehash: ac666a4c7493140d4ae93047e18202c3d8314c7b
ms.sourcegitcommit: daa9ccf580df531254da9dce8593441ac963c674
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91900695"
---
# <a name="azure-active-directory-aad-groups-administrator-roles-and-user-defined-roles"></a>Azure Active Directory (AAD) 组、管理员角色和用户定义的角色

作者：[Luke Latham](https://github.com/guardrex) 和 [Javier Calvarro Nelson](https://github.com/javiercn)

Azure Active Directory (AAD) 提供了多种授权方法，它们可与 ASP.NET Core Identity 结合使用：

* 用户定义的组
  * 安全性
  * Microsoft 365
  * 分布
* 角色
  * AAD 管理员角色
  * 用户定义的角色

本文中的指南适用于以下主题中所述的 Blazor WebAssembly AAD 部署方案：

* [包含 Microsoft 帐户的独立产品](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [包含 AAD 的独立产品](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [由 AAD 托管](xref:blazor/security/webassembly/hosted-with-azure-active-directory)

## <a name="microsoft-graph-api-permission"></a>Microsoft 图形 API 权限

具有五个以上 AAD 管理员角色和安全组成员身份的任何应用用户都需要调用 [Microsoft 图形 API](/graph/use-the-api)。

若要允许图形 API 调用，请在 Azure 门户中向托管的 Blazor 解决方案的独立应用或 `Client` 应用授予以下任意[图形 API 权限](/graph/permissions-reference)：

* `Directory.Read.All`
* `Directory.ReadWrite.All`
* `Directory.AccessAsUser.All`

`Directory.Read.All` 是具有最低特权的权限，本文中的示例使用此权限。

## <a name="user-defined-groups-and-administrator-roles"></a>用户定义的组和管理员角色

要在 Azure 门户中配置应用以提供 `groups` 成员资格声明，请参阅以下 Azure 文章。 将用户分配到用户定义的 AAD 组和 AAD 管理员角色。

* [使用 Azure AD 安全组的角色](/azure/architecture/multitenant-identity/app-roles#roles-using-azure-ad-security-groups)
* [`groupMembershipClaims` 属性](/azure/active-directory/develop/reference-app-manifest#groupmembershipclaims-attribute)

以下示例假定用户被分配到 AAD 计费管理员角色。

AAD 发送的单个 `groups` 声明在 JSON 数组中将用户的组和角色作为对象 ID (GUID) 显示。 应用必须将组和角色的 JSON 数组转换为单个 `group` 声明，应用可以针对这些声明生成[策略](xref:security/authorization/policies)。

当分配的 AAD 管理员角色和用户定义的组的数量超过五个时，AAD 发送具有 `true` 值的 `hasgroups` 声明，而不是发送 `groups` 声明。 任何可能为其用户分配了五个以上角色和组的应用都必须单独调用图形 API 才能获取用户的角色和组。 本文中提供的示例实现就是针对这种情况的。 有关详细信息，请参阅 [Microsoft 标识平台访问令牌：负载声明](/azure/active-directory/develop/access-tokens#payload-claims)一文中的 `groups` 和 `hasgroups` 声明信息。

扩展 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteUserAccount> 以包括组和角色的数组属性。 为每个属性分配一个空数组，这样以后在 `foreach` 循环中使用这些属性时，就不需要检查 `null` 了。

`CustomUserAccount.cs`:

```csharp
using System.Text.Json.Serialization;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class CustomUserAccount : RemoteUserAccount
{
    [JsonPropertyName("groups")]
    public string[] Groups { get; set; } = new string[] { };

    [JsonPropertyName("roles")]
    public string[] Roles { get; set; } = new string[] { };
}
```

在托管的 Blazor 解决方案的独立应用或 `Client` 应用中，创建自定义 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> 类。 对获取角色和组信息的图形 API 调用使用正确的作用域（权限）。

`GraphAPIAuthorizationMessageHandler.cs`:

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;

public class GraphAPIAuthorizationMessageHandler : AuthorizationMessageHandler
{
    public GraphAPIAuthorizationMessageHandler(IAccessTokenProvider provider,
        NavigationManager navigationManager)
        : base(provider, navigationManager)
    {
        ConfigureHandler(
            authorizedUrls: new[] { "https://graph.microsoft.com" },
            scopes: new[] { "https://graph.microsoft.com/Directory.Read.All" });
    }
}
```

在 `Program.Main` (`Program.cs`)中，添加 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> 实现服务，并添加一个已命名的 <xref:System.Net.Http.HttpClient> 以发出图形 API 请求。 下面的示例将命名客户端 `GraphAPI`：

```csharp
builder.Services.AddScoped<GraphAPIAuthorizationMessageHandler>();

builder.Services.AddHttpClient("GraphAPI",
        client => client.BaseAddress = new Uri("https://graph.microsoft.com"))
    .AddHttpMessageHandler<GraphAPIAuthorizationMessageHandler>();
```

创建 AAD 目录对象类以接收来自图形 API 调用的 Open Data Protocol (OData) 角色和组。 OData 以 JSON 格式送达，并且对 <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> 的调用将填充 `DirectoryObjects` 类的实例。

`DirectoryObjects.cs`:

```csharp
using System.Collections.Generic;
using System.Text.Json.Serialization;

public class DirectoryObjects
{
    [JsonPropertyName("@odata.context")]
    public string Context { get; set; }

    [JsonPropertyName("value")]
    public List<Value> Values { get; set; }
}

public class Value
{
    [JsonPropertyName("@odata.type")]
    public string Type { get; set; }

    [JsonPropertyName("id")]
    public string Id { get; set; }
}
```

创建一个自定义用户工厂来处理角色和组声明。 下面的示例实现还处理 `roles` 声明数组，[用户定义角色](#user-defined-roles)一节中对此进行了说明。 如果存在 `hasgroups` 声明，则使用已命名的 <xref:System.Net.Http.HttpClient> 向提供图形 API 发出授权请求，以获取用户的角色和组。 此实现使用 Microsoft Identity Platform v1.0 终结点 `https://graph.microsoft.com/v1.0/me/memberOf`（[API 文档](/graph/api/user-list-memberof)）。 当 MSAL 包升级到 v2.0 时，本主题的指南将更新到 Identity v2.0。

`CustomAccountFactory.cs`:

```csharp
using System.Linq;
using System.Net.Http;
using System.Net.Http.Json;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;
using Microsoft.Extensions.Logging;

public class CustomUserFactory
    : AccountClaimsPrincipalFactory<CustomUserAccount>
{
    private readonly ILogger<CustomUserFactory> logger;
    private readonly IHttpClientFactory clientFactory;

    public CustomUserFactory(IAccessTokenProviderAccessor accessor, 
        IHttpClientFactory clientFactory, 
        ILogger<CustomUserFactory> logger)
        : base(accessor)
    {
        this.clientFactory = clientFactory;
        this.logger = logger;
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        CustomUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var initialUser = await base.CreateUserAsync(account, options);

        if (initialUser.Identity.IsAuthenticated)
        {
            var userIdentity = (ClaimsIdentity)initialUser.Identity;

            foreach (var role in account.Roles)
            {
                userIdentity.AddClaim(new Claim("role", role));
            }

            if (userIdentity.HasClaim(c => c.Type == "hasgroups"))
            {
                try
                {
                    var client = clientFactory.CreateClient("GraphAPI");

                    var response = await client.GetAsync("v1.0/me/memberOf");

                    if (response.IsSuccessStatusCode)
                    {
                        var userObjects = await response.Content
                            .ReadFromJsonAsync<DirectoryObjects>();

                        foreach (var obj in userObjects?.Values)
                        {
                            userIdentity.AddClaim(new Claim("group", obj.Id));
                        }

                        var claim = userIdentity.Claims.FirstOrDefault(
                            c => c.Type == "hasgroups");

                        userIdentity.RemoveClaim(claim);
                    }
                    else
                    {
                        logger.LogError("Graph API request failure: {REASON}", 
                            response.ReasonPhrase);
                    }
                }
                catch (AccessTokenNotAvailableException exception)
                {
                    logger.LogError("Graph API access token failure: {MESSAGE}", 
                        exception.Message);
                }
            }
            else
            {
                foreach (var group in account.Groups)
                {
                    userIdentity.AddClaim(new Claim("group", group));
                }
            }
        }

        return initialUser;
    }
}
```

无需提供代码来删除原始 `groups` 声明（如果存在），因为框架会自动删除它。

> [!NOTE]
> 本示例中的方法执行以下操作：
>
> * 添加一个自定义 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.AuthorizationMessageHandler> 类，将访问令牌附加到传出请求。
> * 添加一个已命名的 <xref:System.Net.Http.HttpClient>，将 Web API 请求发送到安全的外部 Web API 终结点。
> * 使用已命名的 <xref:System.Net.Http.HttpClient> 发出授权请求。
>
> 有关此方法的常规覆盖范围，请参阅 <xref:blazor/security/webassembly/additional-scenarios#custom-authorizationmessagehandler-class>一文。

在托管的 Blazor 解决方案的独立应用或 `Client` 应用的 `Program.Main` (`Program.cs`) 中注册工厂。 同意将 `Directory.Read.All` 权限范围作为应用的附加范围：

```csharp
builder.Services.AddMsalAuthentication<RemoteAuthenticationState, 
    CustomUserAccount>(options =>
{
    builder.Configuration.Bind("AzureAd", 
        options.ProviderOptions.Authentication);
    options.ProviderOptions.DefaultAccessTokenScopes.Add("...");

    options.ProviderOptions.AdditionalScopesToConsent.Add(
        "https://graph.microsoft.com/Directory.Read.All");
})
.AddAccountClaimsPrincipalFactory<RemoteAuthenticationState, CustomUserAccount, 
    CustomUserFactory>();
```

为 `Program.Main` 中的每个组或角色创建[策略](xref:security/authorization/policies)。 以下示例为 AAD 计费管理员角色创建策略：

```csharp
builder.Services.AddAuthorizationCore(options =>
{
    options.AddPolicy("BillingAdministrator", policy => 
        policy.RequireClaim("group", "69ff516a-b57d-4697-a429-9de4af7b5609"));
});
```

有关 AAD 角色对象 ID 的完整列表，请参阅 [AAD 管理员角色对象 ID](#aad-administrator-role-object-ids) 部分。

在以下示例中，应用使用前面的策略来授权用户。

[`AuthorizeView` 组件](xref:blazor/security/index#authorizeview-component)适用于以下策略：

```razor
<AuthorizeView Policy="BillingAdministrator">
    <Authorized>
        <p>
            The user is in the 'Billing Administrator' AAD Administrator Role
            and can see this content.
        </p>
    </Authorized>
    <NotAuthorized>
        <p>
            The user is NOT in the 'Billing Administrator' role and sees this
            content.
        </p>
    </NotAuthorized>
</AuthorizeView>
```

对整个组件的访问可以基于使用 [`[Authorize]` 属性指令](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>) 的策略：

```razor
@page "/"
@using Microsoft.AspNetCore.Authorization
@attribute [Authorize(Policy = "BillingAdministrator")]
```

如果用户未登录，则会将他们重定向到 AAD 登录页，然后在他们登录后再返回到组件。

策略检查也可以[在具有过程逻辑的代码中执行](xref:blazor/security/index#procedural-logic)：

```razor
@page "/checkpolicy"
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<h1>Check Policy</h1>

<p>This component checks a policy in code.</p>

<button @onclick="CheckPolicy">Check 'BillingAdministrator' policy</button>

<p>Policy Message: @policyMessage</p>

@code {
    private string policyMessage = "Check hasn't been made yet.";

    [CascadingParameter]
    private Task<AuthenticationState> authenticationStateTask { get; set; }

    private async void CheckPolicy()
    {
        var user = (await authenticationStateTask).User;

        if ((await AuthorizationService.AuthorizeAsync(user, 
            "BillingAdministrator")).Succeeded)
        {
            policyMessage = "Yes! The 'BillingAdministrator' policy is met.";
        }
        else
        {
            policyMessage = "No! 'BillingAdministrator' policy is NOT met.";
        }
    }
}
```

## <a name="authorize-server-api-access-for-user-defined-groups-and-administrator-roles"></a>授权用户定义的组和管理员角色访问服务器 API

除了在客户端 WebAssembly 应用中授权用户访问页面和资源以外，服务器 API 还可以授权用户访问安全 API 终结点。 在服务器应用验证用户的访问令牌后：

* 该应用使用用户来自 JWT (`id_token`) 的不可变[对象标识符声明 (`oid`)](/azure/active-directory/develop/id-tokens#payload-claims) 来获取图形 API 的访问令牌。
* 图形 API 调用获取用户的 Azure 用户定义的安全组和管理员角色成员身份。
* 成员身份用于建立 `group` 声明。
* [授权策略](xref:security/authorization/policies)可用于限制用户对服务器 API 终结点的访问。

> [!NOTE]
> 本指南目前不涵盖根据用户的 [AAD 用户定义的角色](#user-defined-roles)对用户进行授权。

### <a name="packages"></a>包

将包引用添加到以下包的服务器应用中：

* [Microsoft.Graph](https://www.nuget.org/packages/Microsoft.Graph)
* [Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages?q=Microsoft.IdentityModel.Clients.ActiveDirectory)

### <a name="azure-configuration"></a>Azure 配置

* 确认为服务器应用注册授予了对图形 API `Directory.Read.All` 权限的 API 访问，这是安全组的最低特权访问级别。 确认在分配权限后，将管理员同意应用于权限。
* 向服务器应用分配新的客户端密码。 请留意[应用设置](#app-settings)部分中应用配置的密码。

### <a name="app-settings"></a>应用设置

在应用设置文件（`appsettings.json` 或 `appsettings.Production.json`）中，使用服务器应用的客户端密码从 Azure 门户创建 `ClientSecret` 条目：

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "XXXXXXXXXXXX.onmicrosoft.com",
  "TenantId": "{GUID}",
  "ClientId": "{GUID}",
  "ClientSecret": "{CLIENT SECRET}"
},
```

例如：

```json
"AzureAd": {
  "Instance": "https://login.microsoftonline.com/",
  "Domain": "contoso.onmicrosoft.com",
  "TenantId": "34bf0ec1-7aeb-4b5d-ba42-82b059b3abe8",
  "ClientId": "05d198e0-38c6-4efc-a67c-8ee87ed9bd3d",
  "ClientSecret": "54uE~9a.-wW91fe8cRR25ag~-I5gEq_92~"
},
```

### <a name="authorization-policies"></a>授权策略

基于组对象 ID 和 [AAD 管理员角色对象 ID](#aad-administrator-role-object-ids)，为服务器应用的 `Startup.ConfigureServices` (`Startup.cs`) 中的 AAD 安全组和 AAD 管理员角色创建[授权策略](xref:security/authorization/policies)。

例如，Azure 计费管理员角色策略具有以下配置：

```csharp
services.AddAuthorization(options =>
{
    options.AddPolicy("BillingAdmin", policy => 
        policy.RequireClaim("group", "69ff516a-b57d-4697-a429-9de4af7b5609"));
});
```

有关详细信息，请参阅 <xref:security/authorization/policies>。

### <a name="controller-access"></a>控制器访问

需要服务器应用控制器的策略。

下面的示例使用名为 `BillingAdmin` 的策略，将对 `BillingDataController` 中计费数据的访问限制为 Azure 计费管理员，如[授权策略](#authorization-policies)部分配置的那样：

```csharp
[Authorize(Policy = "BillingAdmin")]
[ApiController]
[Route("[controller]")]
public class BillingDataController : ControllerBase
{
    ...
}
```

### <a name="service-configuration"></a>服务配置

在服务器应用的 `Startup.ConfigureServices` 方法中，添加逻辑以进行图形 API 调用，并为用户的安全组和角色建立用户 `group` 声明。

> [!NOTE]
> 本节中的示例代码使用基于 Microsoft Identity Platform v1.0 的 Active Directory 身份验证库 (ADAL)。 本主题将针对适用于 .NET 5 的 Identity v2.0 进行更新。 通过监视 [[RC1] Microsoft Identity Platform 2.0 for Blazor (dotnet/AspNetCore.Docs #19503)](https://github.com/dotnet/AspNetCore.Docs/issues/19503) 可跟踪此项工作的进度。

服务器应用的 `Startup` 类中的代码需要更多命名空间。 下面的一组 `using` 语句包含本节下面的代码所需的命名空间：

```csharp
using System;
using System.IdentityModel.Tokens.Jwt;
using System.Linq;
using System.Net.Http.Headers;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.AzureAD.UI;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.Graph;
using Microsoft.IdentityModel.Clients.ActiveDirectory;
using Microsoft.IdentityModel.Logging;
```

配置 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents> 时：

* 可以选择包括对 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnAuthenticationFailed?displayProperty=nameWithType> 的处理。 例如，应用可以记录失败的身份验证。
* 在 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated?displayProperty=nameWithType> 中，进行图形 API 调用以获取用户的组和角色。

> [!WARNING]
> <xref:Microsoft.IdentityModel.Logging.IdentityModelEventSource.ShowPII?displayProperty=nameWithType> 在日志消息中提供个人身份信息 (PII)。 只能用测试用户帐号激活 PII 进行调试。

在 `Startup.ConfigureServices`中：

```csharp
#if DEBUG
IdentityModelEventSource.ShowPII = true;
#endif

services.AddAuthentication(AzureADDefaults.BearerAuthenticationScheme)
    .AddAzureADBearer(options => Configuration.Bind("AzureAd", options));

services.Configure<JwtBearerOptions>(AzureADDefaults.JwtBearerAuthenticationScheme, 
    options =>
{
    options.Events = new JwtBearerEvents()
    {
        OnAuthenticationFailed = context =>
        {
            // Optional: Log the exception

#if DEBUG
            Console.WriteLine($"OnAuthenticationFailed: {context.Exception}");
#endif

            return Task.FromResult(0);
        },
        OnTokenValidated = async context =>
        {
            var accessToken = context.SecurityToken as JwtSecurityToken;
            var oid = accessToken.Claims.FirstOrDefault(x => x.Type == "oid")?
                .Value;

            if (!string.IsNullOrEmpty(oid))
            {
                var authContext = new AuthenticationContext(
                    Configuration["AzureAd:Instance"] +
                    Configuration["AzureAd:TenantId"]);
                AuthenticationResult authResult = null;

                try
                {
                    authResult = await authContext.AcquireTokenSilentAsync(
                        "https://graph.microsoft.com", 
                        Configuration["AzureAd:ClientId"]);
                }
                catch (AdalException adalException)
                {
                    if (adalException.ErrorCode == 
                        AdalError.FailedToAcquireTokenSilently || 
                        adalException.ErrorCode == 
                        AdalError.UserInteractionRequired)
                    {
                        var userAssertion = new UserAssertion(accessToken.RawData,
                            "urn:ietf:params:oauth:grant-type:jwt-bearer", oid);
                        var clientCredential = new ClientCredential(
                            Configuration["AzureAd:ClientId"],
                            Configuration["AzureAd:ClientSecret"]);
                        authResult = await authContext.AcquireTokenAsync(
                            "https://graph.microsoft.com", clientCredential, 
                            userAssertion);
                    }
                }

                var graphClient = new GraphServiceClient(
                    new DelegateAuthenticationProvider(async requestMessage => {
                        requestMessage.Headers.Authorization =
                            new AuthenticationHeaderValue("Bearer", 
                                authResult.AccessToken);

                        await Task.CompletedTask;
                    }));

                var userIdentity = (ClaimsIdentity)context.Principal.Identity;

                IUserMemberOfCollectionWithReferencesPage groupsAndAzureRoles = 
                    null;

                try
                {
                    groupsAndAzureRoles = await graphClient.Users[oid].MemberOf
                        .Request().GetAsync();
                }
                catch (ServiceException serviceException)
                {
                    // Optional: Log the error

#if DEBUG
                    Console.WriteLine(
                        "OnTokenValidated: Service Exception: " +
                        $"{serviceException.Message}");
#endif
                }

                if (groupsAndAzureRoles != null)
                {
                    foreach (var entry in groupsAndAzureRoles)
                    {
                        userIdentity.AddClaim(new Claim("group", entry.Id));
                    }
                }
            }
            else
            {
                // Optional: Log missing OID claim

#if DEBUG
                Console.WriteLine($"OnTokenValidated: OID missing: " +
                    $"{accessToken.RawData}");
#endif
            }

            await Task.FromResult(0);
        }
    };
});
```

在上面的示例中：

* 首先尝试静默令牌获取 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.AcquireTokenSilentAsync%2A>)，因为访问令牌可能已经存储在 ADAL 令牌缓存中。 从缓存中获取令牌比请求新令牌更快。
* 如果访问令牌不是从缓存中获取的（引发 <xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AdalError.FailedToAcquireTokenSilently?displayProperty=nameWithType> 或 <xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AdalError.UserInteractionRequired?displayProperty=nameWithType>），则使用客户端凭据 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential>) 生成用户断言 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.UserAssertion>)，以代表用户 (<xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.AcquireTokenAsync%2A>) 获取令牌。 接下来，`Microsoft.Graph.GraphServiceClient` 可以继续使用令牌进行图形 API 调用。 令牌放置在 ADAL 令牌缓存中。 对于同一用户的未来图形 API 调用，将通过 <xref:Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext.AcquireTokenSilentAsync%2A> 以无提示方式从缓存中获取令牌。

<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated> 中的代码不会获得可传递的成员身份。 若要更改代码以获取直接和可传递的成员身份：

* 对于代码行：

  ```csharp
  IUserMemberOfCollectionWithReferencesPage groupsAndAzureRoles = null;
  ```

  将上面的行替换为：

  ```csharp
  IUserTransitiveMemberOfCollectionWithReferencesPage groupsAndAzureRoles = null;
  ```

* 对于代码行：

  ```csharp
  groupsAndAzureRoles = await graphClient.Users[oid].MemberOf.Request().GetAsync();
  ```

  将上面的行替换为：

  ```csharp
  groupsAndAzureRoles = await graphClient.Users[oid].TransitiveMemberOf.Request()
      .GetAsync();
  ```

创建声明时，<xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerEvents.OnTokenValidated> 中的代码不会区分 AAD 安全组和 AAD 管理员角色。 若要使应用区分组和角色，请在循环访问组和角色时检查 `entry.ODataType`。 若要创建单独的安全组和角色声明，请使用类似于下面的代码：

```csharp
foreach (var entry in groupsAndAzureRoles)
{
    if (entry.ODataType == "#microsoft.graph.group")
    {
        userIdentity.AddClaim(new Claim("group", entry.Id));
    }
    else
    {
        // entry.ODataType == "#microsoft.graph.directoryRole"
        userIdentity.AddClaim(new Claim("role", entry.Id));
    }
}
```

## <a name="user-defined-roles"></a>用户定义的角色

还可以将 AAD 注册应用配置为使用用户定义的角色。

要在 Azure 门户中配置应用以提供 `roles` 成员资格声明，请参阅 Azure 文档中的[如何：在应用程序中添加应用角色并在令牌中接收它们](/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps)。

以下示例假定应用配置了两个角色：

* `admin`
* `developer`

> [!NOTE]
> 尽管无法将角色分配给没有 Azure AD Premium 帐户的安全组，但你可以将用户分配给角色，并为具有标准 Azure 帐户的用户接收 `roles` 声明。 本部分中的指南不需要 Azure AD Premium 帐户。
>
> 通过为其他每个角色分配重新添加用户，在 Azure 门户中分配多个角色。

AAD 发送的单个 `roles` 声明在 JSON 数组中将用户定义的角色作为 `appRoles` 的 `value` 显示。 应用必须将 JSON 角色数组转换为单个 `role` 声明。

[用户定义的组和 AAD 管理员角色](#user-defined-groups-and-administrator-roles)部分中显示的 `CustomUserFactory` 设置为对具有 JSON 数组值的 `roles` 声明执行操作。 在托管的 Blazor 解决方案的独立应用或 `Client` 应用中添加和注册 `CustomUserFactory`，如[用户定义的组和 AAD 管理员角色](#user-defined-groups-and-administrator-roles)部分中所述。 无需提供代码来删除原始 `roles` 声明，因为框架会自动删除它。

在托管的 Blazor 解决方案的独立应用或 `Client` 应用的 `Program.Main` 中，将名为“`role`”的声明指定为角色声明：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...

    options.UserOptions.RoleClaim = "role";
});
```

组件授权方法此时有效。 组件中的任何授权机制都可以使用 `admin` 角色来授权用户：

* [`AuthorizeView` 组件](xref:blazor/security/index#authorizeview-component)（例如 `<AuthorizeView Roles="admin">`）
* [`[Authorize]` 属性指令](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)（例如 `@attribute [Authorize(Roles = "admin")]`）
* [过程逻辑](xref:blazor/security/index#procedural-logic)（例如 `if (user.IsInRole("admin")) { ... }`）

  支持多个角色测试：

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  {
      ...
  }
  ```

## <a name="aad-administrator-role-object-ids"></a>AAD 管理员角色对象 ID

下表中显示的对象 ID 指示用于为 `group` 声明创建[策略](xref:security/authorization/policies)。 策略允许应用授权用户执行应用中的各种活动。 有关详细信息，请参阅[用户定义的组和 AAD 管理员角色](#user-defined-groups-and-administrator-roles)部分。

AAD 管理员角色 | 对象 ID
--- | ---
应用程序管理员 | fa11557b-4f15-4ddd-85d5-313c7cd74047
应用程序开发人员 | 68adcbb8-9504-44f6-89f2-5cd48dc74a2c
身份验证管理员 | 02d110a1-96b1-419e-af87-746461b60ed7
Azure DevOps 管理员 | a5311ace-ca41-44cd-b833-8d22caa0b34f
Azure 信息保护管理员 | 18632dce-f9b5-4f01-abb5-37051f06860e
B2C IEF 密钥集管理员 | 0c2e87e5-94f9-4adb-ae8c-bcafe11bd368
B2C IEF 策略管理员 | bfcab36c-10c6-4b13-b63c-4d8b62c0c44e
B2C 用户流管理员 | baa531b7-8cf0-44ad-8f98-eded88dae827
B2C 用户流属性管理员 | dd0baca0-a535-48c1-b871-8431abe16452
计费管理员 | 69ff516a-b57d-4697-a429-9de4af7b5609
云应用程序管理员 | 250b5fe3-b553-458d-9a53-b782c13c34bf
云设备管理员 | 26cd4b44-2636-4ddb-bdfa-27feae66f86d
法规管理员 | 9d6e1dd0-c9f8-45f8-b558-b134f700116c
合规性数据管理员 | 4c0ca3a2-231e-416c-9411-4abe57d5cb9d
条件访问管理员 | 8f71a611-137d-49af-87ad-e97f1fd5da76
客户密码箱访问审批人 | c18d54a8-b13e-4954-a1a4-7deaf2e4f184
桌面分析管理员 | c62c4ac5-e4c6-4096-8a2f-1ee3cbaaae15
目录读者 | e1fc84a6-7762-4b9b-8e29-518b4adbc23b
Dynamics 365 管理员 | f20a9cfa-9fdf-49a8-a977-1afe446a1d6e
Exchange 管理员 | b2ec2cc0-d5c9-4864-ad9b-38dd9dba2652
外部 Identity 提供者管理员 | febfaeb4-e478-407a-b4b3-f4d9716618a2
全局管理员 | a45ba61b-44db-462c-924b-3b2719152588
全局读取者 | f6903b21-6aba-4124-b44c-76671796b9d5
组管理员 | 158b3e5a-d89d-460b-92b5-3b34985f0197
来宾邀请者 | 4c730a1d-cc22-44af-8f9f-4eec635c7502
支持管理员 | 108678c8-6628-44e1-8d01-caf598a6a5f5
Intune 管理员 | 79950741-23fa-4189-b2cb-46640601c497
Kaizala 管理员 | d6322af2-48e7-42e0-8c68-0bbe31af3412
许可证管理员 | 3355458a-e423-44bf-8b98-4ac5e572cea5
消息中心隐私读取者 | 6395db95-9fb8-42b9-b1ed-30a2405eee6f
消息中心读取者 | fd5d37b8-4e24-434b-9e63-70ed3b759a16
Office 应用管理员 | 5f3870cd-b042-4f93-86d7-c9d77c664dc7
密码管理员 | 466e48b7-5d66-4ae5-8911-1a118de74941
Power BI 管理员 | 984e83b8-8337-4255-91a1-acb663175ab4
Power Platform 管理员 | 76d6f95e-9a15-4d7d-8d21-00de00faf9fd
特权身份验证管理员 | 0829f731-b46d-419f-9742-aeb122367d11
特权角色管理员 | f20a725a-d1c8-4107-83ea-1171c97d00c7
报告读者 | 54635450-e8ed-4f2d-9632-07db2517b4de
搜索管理员 | c770a2f1-c9ba-4e60-9176-9f52b1eb1a31
搜索编辑员 | 6a6858c6-5f0d-44ac-87c7-0190fbedd271
安全管理员 | 20fa50e3-6531-44d8-bd39-b251420568ad
安全操作员 | 43aae017-8e51-4188-91ab-e6debd572800
安全读取者 | 45035cd3-fd97-4250-8197-3a53d3562d9b
服务支持管理员 | 2c92cf45-c914-48f8-9bf9-fc14b28818ab
SharePoint 管理员 | e1c32229-875e-461d-ae24-3cb99116e86c
Skype for Business 管理员 | 0a8cee12-e21d-43ef-abd9-f1ea85710e30
Teams 通信管理员 | 2393e455-6e13-4743-9f52-63fcec2b6a9c
Teams 通信支持工程师 | 802dd94e-d717-46f6-af98-b9167071e9fc
Teams 通信专家 | ef547281-cf46-4cc6-bcaa-f5eac3f030c9
Teams 服务管理员 | 8846a0be-197b-443a-b13c-11192691fa24
用户管理员 | 1f6eed58-7dd3-460b-a298-666f975427a1

## <a name="additional-resources"></a>其他资源

* <xref:security/authorization/claims>
* <xref:blazor/security/index>
