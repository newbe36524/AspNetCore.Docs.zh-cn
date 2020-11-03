---
title: 使用 Identity 服务器保护托管 ASP.NET Core Blazor WebAssembly 应用
author: guardrex
description: 了解如何使用 Identity 服务器保护托管 ASP.NET Core Blazor WebAssembly 应用。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/27/2020
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
uid: blazor/security/webassembly/hosted-with-identity-server
ms.openlocfilehash: a6fd005e19f532089ac1a1914756fb03eabb24c4
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690473"
---
# <a name="secure-an-aspnet-core-no-locblazor-webassembly-hosted-app-with-no-locidentity-server"></a>使用 Identity 服务器保护 ASP.NET Core Blazor WebAssembly 托管应用

作者：[Javier Calvarro Nelson](https://github.com/javiercn) 和 [Luke Latham](https://github.com/guardrex)

本文介绍如何创建[托管的 Blazor WebAssembly 应用](xref:blazor/hosting-models#blazor-webassembly)，该应用使用 [IdentityServer](https://identityserver.io/) 以对用户和 API 调用进行身份验证。

> [!NOTE]
> 若要将独立或托管 Blazor WebAssembly 应用配置为使用现有的外部 Identity 服务器实例，请按照 <xref:blazor/security/webassembly/standalone-with-authentication-library> 中的指南操作。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

新建具有身份验证机制的 Blazor WebAssembly 项目：

1. 在“新建 ASP.NET Core Web 应用”对话框中选择“Blazor WebAssembly应用”模板后，选择“身份验证”下的“更改”。

1. 通过“存储应用内的用户帐户”选项选择“单个用户帐户”，以使用 ASP.NET Core 的 [Identity](xref:security/authentication/identity) 系统存储应用内的用户。 

1. 在“高级”部分中选中“托管的 ASP.NET Core”复选框 。

# <a name="visual-studio-code--net-core-cli"></a>[Visual Studio Code/.NET Core CLI](#tab/visual-studio-code+netcore-cli)

若要在空文件夹中新建具有身份验证机制的 Blazor WebAssembly 项目，则通过 `-au|--auth` 选项指定 `Individual` 身份验证机制，以使用 ASP.NET Core 的 [Identity](xref:security/authentication/identity) 系统存储应用内的用户：

```dotnetcli
dotnet new blazorwasm -au Individual -ho -o {APP NAME}
```

| 占位符  | 示例        |
| ------------ | -------------- |
| `{APP NAME}` | `BlazorSample` |

使用 `-o|--output` 选项指定的输出位置将创建一个项目文件夹（如果该文件夹不存在）并成为应用程序名称的一部分。

有关详细信息，请参阅 .NET Core 指南中的 [`dotnet new`](/dotnet/core/tools/dotnet-new) 命令。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

新建具有身份验证机制的 Blazor WebAssembly 项目：

1. 在“配置新的 Blazor WebAssembly应用”步骤中，从“身份验证”下拉列表中选择“个人身份验证(应用内)”。

1. 此应用是使用 ASP.NET Core [Identity](xref:security/authentication/identity) 为应用中存储的个人用户创建的。

1. 选中“托管的 ASP.NET Core”复选框。

---

## <a name="server-app-configuration"></a>`Server` 应用配置

以下部分介绍了在包括身份验证支持的情况下对项目添加的内容。

### <a name="startup-class"></a>Startup 类

`Startup` 类包含以下添加项。

* 在 `Startup.ConfigureServices`中：

  * ASP.NET Core Identity:

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));

    services.AddDefaultIdentity<ApplicationUser>(options => 
            options.SignIn.RequireConfirmedAccount = true)
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

  * 包含附加 <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> 帮助器方法的 IdentityServer，该方法在 IdentityServer 之上设置默认 ASP.NET Core 约定：

    ```csharp
    services.AddIdentityServer()
        .AddApiAuthorization<ApplicationUser, ApplicationDbContext>();
    ```

  * 包含附加 <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> 帮助器方法的身份验证，该方法将应用配置为验证由 IdentityServer 生成的 JWT 令牌：

    ```csharp
    services.AddAuthentication()
        .AddIdentityServerJwt();
    ```

* 在 `Startup.Configure`中：

  * IdentityServer 中间件公开 OpenID Connect (OIDC) 终结点：

    ```csharp
    app.UseIdentityServer();
    ```

  * 身份验证中间件负责验证请求凭据并在请求上下文中设置用户：

    ```csharp
    app.UseAuthentication();
    ```

  * 授权中间件支持授权功能：

    ```csharp
    app.UseAuthentication();
    app.UseAuthorization();
    ```

### <a name="addapiauthorization"></a>AddApiAuthorization

<xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> 帮助器方法配置 [IdentityServer](https://identityserver.io/) 用于 ASP.NET Core 方案。 IdentityServer 是一个功能强大且可扩展的框架，用于处理应用安全问题。 IdentityServer 公开大多数情况下不必要的复杂性。 因此，提供了一组约定和配置选项作为良好的起点。 一旦身份验证需要更改，就可使用 IdentityServer 的完整功能自定义身份验证以满足应用的要求。

### <a name="addno-locidentityserverjwt"></a>AddIdentityServerJwt

<xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> 帮助器方法将应用的策略方案配置为默认身份验证处理程序。 该策略配置为允许 Identity 处理路由到 Identity URL 空间 `/Identity` 中任何子路径的所有请求。 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> 处理所有其他请求。 此外，此方法还可以：

* 向具有默认 `{APPLICATION NAME}API` 范围的 IdentityServer 注册 `{APPLICATION NAME}API` API 资源。
* 配置 JWT 持有者令牌中间件以验证 IdentityServer 为应用颁发的令牌。

### <a name="weatherforecastcontroller"></a>WeatherForecastController

在 `WeatherForecastController` (`Controllers/WeatherForecastController.cs`) 中，[`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 特性应用于该类。 该属性指示用户必须根据默认策略获得授权才能访问资源。 默认授权策略配置为使用默认身份验证方案，由 <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilderExtensions.AddIdentityServerJwt%2A> 设置。 帮助器方法将 <xref:Microsoft.AspNetCore.Authentication.JwtBearer.JwtBearerHandler> 配置为对应用的请求的默认处理程序。

### <a name="applicationdbcontext"></a>ApplicationDbContext

在 `ApplicationDbContext` (`Data/ApplicationDbContext.cs`) 中，<xref:Microsoft.EntityFrameworkCore.DbContext> 扩展 <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> 以包括 IdentityServer 的架构。 <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiAuthorizationDbContext%601> 派生自 <xref:Microsoft.AspNetCore.Identity.EntityFrameworkCore.IdentityDbContext>。

要获取对数据库架构的完全控制，请从其中一个可用的 Identity <xref:Microsoft.EntityFrameworkCore.DbContext> 类继承，并通过在 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A> 方法中调用 `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` 来配置上下文以包括 Identity 架构。

### <a name="oidcconfigurationcontroller"></a>OidcConfigurationController

在 `OidcConfigurationController` (`Controllers/OidcConfigurationController.cs`) 中，客户端终结点预配为提供 OIDC 参数。

### <a name="app-settings"></a>应用设置

在项目根目录的应用设置文件 (`appsettings.json`) 中，`IdentityServer` 部分描述已配置的客户端列表。 下例中存在一个客户端。 客户端名称对应于应用名称，并通过约定映射到 OAuth `ClientId` 参数。 配置文件指示正在配置的应用类型。 配置文件在内部用于促进简化服务器配置过程的约定。 <!-- There are several profiles available, as explained in the [Application profiles](#application-profiles) section. -->

```json
"IdentityServer": {
  "Clients": {
    "{APP ASSEMBLY}.Client": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

占位符 `{APP ASSEMBLY}` 是应用的程序集名称（例如 `BlazorSample.Client`）。

## <a name="client-app-configuration"></a>`Client` 应用配置

### <a name="authentication-package"></a>身份验证包

创建应用以使用个人用户帐户 (`Individual`) 时，该应用会在其项目文件中自动接收 [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) 包的包引用。 此包提供了一组基元，可帮助应用验证用户身份并获取令牌以调用受保护的 API。

如果向应用添加身份验证，请手动将包添加到应用的项目文件中：

```xml
<PackageReference 
  Include="Microsoft.AspNetCore.Components.WebAssembly.Authentication" 
  Version="{VERSION}" />
```

对于占位符 `{VERSION}`，可在包的版本历史记录（位于 [NuGet.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication)）中找到与应用的共享框架版本匹配的最新稳定版本的包。

### <a name="httpclient-configuration"></a>`HttpClient` 配置

在 `Program.Main` (`Program.cs`) 中，命名的 <xref:System.Net.Http.HttpClient> (`HostIS.ServerAPI`) 配置为提供 <xref:System.Net.Http.HttpClient> 实例，以在向服务器 API 发出请求时包含访问令牌：

```csharp
builder.Services.AddHttpClient("HostIS.ServerAPI", 
        client => client.BaseAddress = new Uri(builder.HostEnvironment.BaseAddress))
    .AddHttpMessageHandler<BaseAddressAuthorizationMessageHandler>();

builder.Services.AddScoped(sp => sp.GetRequiredService<IHttpClientFactory>()
    .CreateClient("HostIS.ServerAPI"));
```

> [!NOTE]
> 若要将 Blazor WebAssembly 应用配置为使用不属于托管 Blazor 解决方案的现有 Identity 服务器实例，请将 <xref:System.Net.Http.HttpClient> 基址注册从 <xref:Microsoft.AspNetCore.Components.WebAssembly.Hosting.IWebAssemblyHostEnvironment.BaseAddress?displayProperty=nameWithType> (`builder.HostEnvironment.BaseAddress`) 更改为服务器应用的 API 授权终结点 URL。

### <a name="api-authorization-support"></a>API 身份验证支持

使用 [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) 包中提供的扩展方法在服务容器中加入用户身份验证支持。 此方法设置应用所需的服务以与现有授权系统交互。

```csharp
builder.Services.AddApiAuthorization();
```

默认情况下，应用的配置按约定从 `_configuration/{client-id}` 加载。 按照约定，客户端 ID 设置为应用的程序集名称。 可以通过使用选项调用重载来更改此 URL 以指向单独的终结点。

### <a name="imports-file"></a>导入文件

[!INCLUDE[](~/includes/blazor-security/imports-file-hosted.md)]

### <a name="index-page"></a>索引页

[!INCLUDE[](~/includes/blazor-security/index-page-authentication.md)]

### <a name="app-component"></a>应用组件

[!INCLUDE[](~/includes/blazor-security/app-component.md)]

### <a name="redirecttologin-component"></a>RedirectToLogin 组件

[!INCLUDE[](~/includes/blazor-security/redirecttologin-component.md)]

### <a name="logindisplay-component"></a>LoginDisplay 组件

`LoginDisplay` 组件 (`Shared/LoginDisplay.razor`) 在 `MainLayout` 组件 (`Shared/MainLayout.razor`) 中呈现并管理以下行为：

* 对于经过身份验证的用户：
  * 显示当前用户名。
  * 提供指向 ASP.NET Core Identity 中的用户配置文件页面的链接。
  * 提供用于注销应用的按钮。
* 对于匿名用户：
  * 提供用于注册的选项。
  * 提供用于登录的选项。

```razor
@using Microsoft.AspNetCore.Components.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@inject NavigationManager Navigation
@inject SignOutSessionStateManager SignOutManager

<AuthorizeView>
    <Authorized>
        <a href="authentication/profile">Hello, @context.User.Identity.Name!</a>
        <button class="nav-link btn btn-link" @onclick="BeginSignOut">
            Log out
        </button>
    </Authorized>
    <NotAuthorized>
        <a href="authentication/register">Register</a>
        <a href="authentication/login">Log in</a>
    </NotAuthorized>
</AuthorizeView>

@code {
    private async Task BeginSignOut(MouseEventArgs args)
    {
        await SignOutManager.SetSignOutState();
        Navigation.NavigateTo("authentication/logout");
    }
}
```

### <a name="authentication-component"></a>身份验证组件

[!INCLUDE[](~/includes/blazor-security/authentication-component.md)]

### <a name="fetchdata-component"></a>FetchData 组件

[!INCLUDE[](~/includes/blazor-security/fetchdata-component.md)]

## <a name="run-the-app"></a>运行应用

从服务器项目运行应用。 使用 Visual Studio 时，请执行以下任一操作：

* 在工具栏中将“启动项目”下拉列表设置为“服务器 API 应用”，然后选择“运行”按钮。
* 在“解决方案资源管理器”中选择服务器项目，然后选择工具栏中的“运行”按钮，或从“调试”菜单启动应用  。

## <a name="name-and-role-claim-with-api-authorization"></a>具有 API 授权的名称和角色声明

### <a name="custom-user-factory"></a>自定义用户工厂

在 `Client` 应用中，创建自定义用户工厂。 Identity 服务器在一个 `role` 声明中发送多个角色作为 JSON 数组。 单个角色在该声明中作为单个字符串值进行发送。 工厂为每个用户的角色创建单个 `role` 声明。

`CustomUserFactory.cs`:

```csharp
using System.Linq;
using System.Security.Claims;
using System.Text.Json;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication;
using Microsoft.AspNetCore.Components.WebAssembly.Authentication.Internal;

public class CustomUserFactory
    : AccountClaimsPrincipalFactory<RemoteUserAccount>
{
    public CustomUserFactory(IAccessTokenProviderAccessor accessor)
        : base(accessor)
    {
    }

    public async override ValueTask<ClaimsPrincipal> CreateUserAsync(
        RemoteUserAccount account,
        RemoteAuthenticationUserOptions options)
    {
        var user = await base.CreateUserAsync(account, options);

        if (user.Identity.IsAuthenticated)
        {
            var identity = (ClaimsIdentity)user.Identity;
            var roleClaims = identity.FindAll(identity.RoleClaimType).ToArray();

            if (roleClaims != null && roleClaims.Any())
            {
                foreach (var existingClaim in roleClaims)
                {
                    identity.RemoveClaim(existingClaim);
                }

                var rolesElem = account.AdditionalProperties[identity.RoleClaimType];

                if (rolesElem is JsonElement roles)
                {
                    if (roles.ValueKind == JsonValueKind.Array)
                    {
                        foreach (var role in roles.EnumerateArray())
                        {
                            identity.AddClaim(new Claim(options.RoleClaim, role.GetString()));
                        }
                    }
                    else
                    {
                        identity.AddClaim(new Claim(options.RoleClaim, roles.GetString()));
                    }
                }
            }
        }

        return user;
    }
}
```

在 `Client` 应用中，在 `Program.Main` (`Program.cs`) 中注册工厂：

```csharp
builder.Services.AddApiAuthorization()
    .AddAccountClaimsPrincipalFactory<CustomUserFactory>();
```

在 `Server` 应用中，调用 Identity 生成器上的 <xref:Microsoft.AspNetCore.Identity.IdentityBuilder.AddRoles*>，添加与角色相关的服务：

```csharp
using Microsoft.AspNetCore.Identity;

...

services.AddDefaultIdentity<ApplicationUser>(options => 
    options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

### <a name="configure-no-locidentity-server"></a>配置 Identity 服务器

使用以下方法之一：

* [API 身份验证选项](#api-authorization-options)
* [配置文件服务](#profile-service)

#### <a name="api-authorization-options"></a>API 身份验证选项

在 `Server` 应用中：

* 配置 Identity 服务器，将 `name` 和 `role` 声明放入 ID 令牌和访问令牌中。
* 阻止 JWT 令牌处理程序中角色的默认映射。

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Linq;

...

services.AddIdentityServer()
    .AddApiAuthorization<ApplicationUser, ApplicationDbContext>(options => {
        options.IdentityResources["openid"].UserClaims.Add("name");
        options.ApiResources.Single().UserClaims.Add("name");
        options.IdentityResources["openid"].UserClaims.Add("role");
        options.ApiResources.Single().UserClaims.Add("role");
    });

JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Remove("role");
```

#### <a name="profile-service"></a>配置文件服务

在 `Server` 应用中，创建 `ProfileService` 实现。

`ProfileService.cs`:

```csharp
using IdentityModel;
using IdentityServer4.Models;
using IdentityServer4.Services;
using System.Threading.Tasks;

public class ProfileService : IProfileService
{
    public ProfileService()
    {
    }

    public async Task GetProfileDataAsync(ProfileDataRequestContext context)
    {
        var nameClaim = context.Subject.FindAll(JwtClaimTypes.Name);
        context.IssuedClaims.AddRange(nameClaim);

        var roleClaims = context.Subject.FindAll(JwtClaimTypes.Role);
        context.IssuedClaims.AddRange(roleClaims);

        await Task.CompletedTask;
    }

    public async Task IsActiveAsync(IsActiveContext context)
    {
        await Task.CompletedTask;
    }
}
```

在 `Server` 应用中，在 `Startup.ConfigureServices` 中注册配置文件服务：

```csharp
using IdentityServer4.Services;

...

services.AddTransient<IProfileService, ProfileService>();
```

### <a name="use-authorization-mechanisms"></a>使用授权机制

在 `Client` 应用中，组件授权方法此时可以正常工作。 组件中的任何授权机制都可以使用角色来授权用户：

* [`AuthorizeView` 组件](xref:blazor/security/index#authorizeview-component)（例如：`<AuthorizeView Roles="admin">`）
* [`[Authorize]` 属性指令](xref:blazor/security/index#authorize-attribute) (<xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute>)（例如 `@attribute [Authorize(Roles = "admin")]`）
* [过程逻辑](xref:blazor/security/index#procedural-logic)（例如 `if (user.IsInRole("admin")) { ... }`）

  支持多个角色测试：

  ```csharp
  if (user.IsInRole("admin") && user.IsInRole("developer"))
  {
      ...
  }
  ```

`User.Identity.Name` 在 `Client` 应用中进行填充，并带有用户的用户名，这通常是他们的登录电子邮件地址。

[!INCLUDE[](~/includes/blazor-security/usermanager-signinmanager.md)]

## <a name="host-in-azure-app-service-with-a-custom-domain"></a>使用自定义域托管在 Azure 应用服务中

以下指南介绍如何通过 Identity 服务器，使用自定义域将托管的 Blazor WebAssembly 应用部署到 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)。

对于这种托管方案，请勿将相同的证书用于 [Identity 服务器的令牌签名密钥](https://docs.identityserver.io/en/latest/topics/crypto.html#token-signing-and-validation)以及站点与浏览器的 HTTPS 安全通信：

* 针对这两个要求使用不同的证书是一种很好的安全做法，因为这样能隔离不同用途的私钥。
* 用于与浏览器进行通信的 TLS 证书是独立管理的，不会影响 Identity 服务器的令牌签名。
* 当 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 向应用服务应用提供用于绑定自定义域的证书时，Identity 服务器无法从 Azure Key Vault 获取相同的证书来进行令牌签名。 尽管可以将 Identity 服务器配置为使用来自物理路径的相同 TLS 证书，但将安全证书置于源代码管理中是一种不太好的做法，在大多数情况下都应尽量避免此类行为。

接下来的指南将在 Azure Key Vault 中创建一个仅用于 Identity 服务器令牌签名的自签名证书。 Identity 服务器配置通过应用的 `My` > `CurrentUser` 证书存储使用密钥保管库证书。 其他用于自定义域 HTTPS 流量的证书是与 Identity 服务器签名证书分开创建和配置的。

若要将应用、Azure 应用服务和 Azure Key Vault 配置为使用自定义域和 HTTPS 进行托管，请执行以下操作：

1. 创建计划级别不低于 `Basic B1` 的[应用服务计划](/azure/app-service/overview-hosting-plans)。 应用服务需要 `Basic B1` 或更高的服务层级才能使用自定义域。
1. 使用组织控制的站点的完全限定的域名 (FQDN) 的公用名（例如 `www.contoso.com`）为站点的安全浏览器通信（HTTPS 协议）创建 PFX 证书。 通过以下几项内容创建证书：
   * 密钥使用
     * 数字签名验证 (`digitalSignature`)
     * 密钥加密 (`keyEncipherment`)
   * 增强/扩展的密钥使用
     * 客户端身份验证 (1.3.6.1.5.5.7.3.2)
     * 服务器身份验证 (1.3.6.1.5.5.7.3.1)

   若要创建证书，请使用以下方法之一或任何其他合适的工具或在线服务：

   * [Azure Key Vault](/azure/key-vault/certificates/quick-create-portal#add-a-certificate-to-key-vault)
   * [Windows 上的 MakeCert](/windows/desktop/seccrypto/makecert)
   * [OpenSSL](https://www.openssl.org)

   记下密码，稍后将证书导入 Azure Key Vault 时会用到该密码。

   有关 Azure Key Vault 证书的详细信息，请参阅 [Azure Key Vault：](/azure/key-vault/certificates/)证书”。
1. 创建新的 Azure Key Vault 或使用 Azure 订阅中现有的密钥保管库。
1. 在密钥保管库的“证书”区域中，导入 PFX 站点证书。 记录证书的指纹，稍后在应用的配置过程中会用到它。
1. 在 Azure Key Vault 中，生成新的自签名证书以用于 Identity 服务器令牌签名。 给出证书的“证书名称”以及“使用者” 。 “使用者”指定为 `CN={COMMON NAME}`，其中 `{COMMON NAME}` 占位符是证书的公用名。 公用名可以是任意字母数字字符串。 例如 `CN=IdentityServerSigning` 就是一个有效的证书“使用者”。 使用默认的高级策略配置设置。 记录证书的指纹，稍后在应用的配置过程中会用到它。
1. 导航到 Azure 门户中的 Azure 应用服务，并使用以下配置创建新的应用服务：
   * 将“发布”设置为 `Code`。
   * 将“运行时堆栈”设置为应用的运行时。
   * 对于“Sku 和大小”，请确认应用服务层级不低于 `Basic B1`。  应用服务需要 `Basic B1` 或更高的服务层级才能使用自定义域。
1. 在 Azure 创建应用服务后，请打开应用的“配置”并添加一个新的应用程序设置，该设置指定之前记录的证书指纹。 应用设置密钥为 `WEBSITE_LOAD_CERTIFICATES`。 使用逗号分隔应用设置值中的证书指纹，如下面的示例所示：
   * 键：`WEBSITE_LOAD_CERTIFICATES`
   * 值：`57443A552A46DB...D55E28D412B943565,29F43A772CB6AF...1D04F0C67F85FB0B1`

   在 Azure 门户中，可通过两个步骤保存应用设置：保存 `WEBSITE_LOAD_CERTIFICATES` 键-值设置，然后选择边栏选项卡顶部的“保存”按钮。
1. 选择应用的“TLS/SSL 设置”。 选择“私钥证书(.pfx)”。 完成两次“导入 Key Vault 证书”流程，导入用于 HTTPS 通信的站点证书以及站点的自签名 Identity 服务器令牌签名证书。
1. 导航到“自定义域”边栏选项卡。 在域注册机构的网站上，使用 IP 地址和自定义域验证 ID 来配置域 。 域配置通常包括：
   * 一个 A 记录，主机为 `@`，以及来自 Azure 门户的 IP 地址值 。
   * 一个 TXT 记录，主机为 `asuid`，以及由 Azure 生成且由 Azure 门户提供的验证 ID 的值 。

   请确保将更改正确地保存到域注册机构的网站上。 部分注册机构网站需要执行两个步骤来保存域记录：先单独保存一条或多条记录，然后使用单独的按钮更新域的注册。
1. 返回到 Azure 门户中的“自定义域”边栏选项卡。 选择“添加自定义域”。 选择“A 记录”选项。 提供域并选择“验证”。 如果域记录正确并跨 Internet 传播，则门户允许选择“添加自定义域”按钮。

   域注册机构处理域注册更改后，可能需要几天的时间才能让更改在 Internet 域名服务器 (DNS) 之间传播。 如果域记录在三个工作日内未更新，请确认是否已通过域注册机构正确设置记录，并与其客户支持部门联系。
1. 在“自定义域”边栏选项卡中，域的“SSL STATE”被标记为 `Not Secure` 。 选择“添加绑定”链接。 从密钥保管库中选择站点 HTTPS 证书以绑定自定义域。
1. 在 Visual Studio 中，打开 Server 项目的应用设置文件（`appsettings.json` 或 `appsettings.Production.json`）。 在 Identity 服务器配置中，添加以下 `Key` 部分。 为 `Name` 密钥指定自签名证书使用者。 在以下示例中，密钥保管库中分配的证书公用名为 `IdentityServerSigning`，它生成的使用者为 `CN=IdentityServerSigning`：

   ```json
   "IdentityServer": {

     ...

     "Key": {
       "Type": "Store",
       "StoreName": "My",
       "StoreLocation": "CurrentUser",
       "Name": "CN=IdentityServerSigning"
     }
   },
   ```

1. 在 Visual Studio 中，创建用于“Server”项目的 Azure 应用服务[发布配置文件](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)。 在菜单栏中，依次选择：“生成” > “发布” > “新建” > “Azure” > “Azure 应用服务”（Windows 或 Linux）    。 Visual Studio 连接到 Azure 订阅后，可以按资源类型来设置 Azure 资源的视图 。 在“Web 应用”列表中导航，查找应用的应用服务并将其选中。 选择“完成”  。
1. 当 Visual Studio 返回到“发布”窗口时，会自动检测密钥保管库和 SQL Server 数据库服务依赖关系。

   对于密钥保管库服务，不需要对默认设置进行配置更改。

   为了进行测试，应用的本地 [SQLite](https://www.sqlite.org/index.html) 数据库（默认情况下由 Blazor 模板配置）可以与应用一起部署，而无需进行其他配置。 本文不讨论在生产环境中为 Identity 服务器配置其他数据库的情况。 有关详细信息，请参阅以下文档集中的数据库资源：
   * [应用服务](/azure/app-service/)
   * [Identity 服务器](https://identityserver4.readthedocs.io/en/latest/)

1. 在窗口顶部的部署配置文件名称下，选择“编辑”链接。 将“目标 URL”更改为站点的自定义域 URL（例如 `https://www.contoso.com`）。 保存设置。
1. 发布应用。 Visual Studio 将打开一个浏览器窗口，并请求其自定义域所对应的站点。

Azure 文档包含有关在应用服务中通过 TLS 绑定使用 Azure 服务和自定义域的其他详细信息，包括有关使用 CNAME 记录而不是 A 记录的信息。 有关更多信息，请参见以下资源：

* [应用服务文档](/azure/app-service/)
* [教程：将现有的自定义 DNS 名称映射到 Azure 应用服务](/azure/app-service/app-service-web-tutorial-custom-domain)
* [在 Azure 应用服务中使用 TLS/SSL 绑定保护自定义 DNS 名称](/azure/app-service/configure-ssl-bindings)
* [Azure Key Vault](/azure/key-vault/)

在 Azure 门户中的应用、应用配置或 Azure 服务发生更改后，建议使用新的私密或隐匿浏览器窗口运行每个应用测试。 在测试站点时，即使站点的配置是正确的，前一个测试运行中的延迟 cookie 仍然可能导致身份验证失败或授权失败。 若要详细了解如何将 Visual Studio 配置为针对每个测试运行打开新的私密或隐匿浏览器窗口，请参阅 [Cookie 和站点数据](#cookies-and-site-data)部分。

如果在 Azure 门户中更改了应用服务配置，则更新通常会快速生效，但不会立即生效。 有时，必须等待一小段时间来让应用服务重新启动，配置更改才能生效。

若要对证书加载问题进行故障排除，请在 Azure 门户 [Kudu](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service) PowerShell 命令外壳中执行以下命令。 该命令提供应用可在 `My` > `CurrentUser` 证书存储中访问的证书列表。 调试应用时，输出包括非常有用的证书使用者和指纹：

```powershell
Get-ChildItem -path Cert:\CurrentUser\My -Recurse | Format-List DnsNameList, Subject, Thumbprint, EnhancedKeyUsageList
```

[!INCLUDE[](~/includes/blazor-security/troubleshoot.md)]

## <a name="additional-resources"></a>其他资源

* [部署到 Azure 应用服务](xref:security/authentication/identity/spa#deploy-to-production)
* [导入来自 Key Vault 的证书（Azure 文档）](/azure/app-service/configure-ssl-certificate#import-a-certificate-from-key-vault)
* <xref:blazor/security/webassembly/additional-scenarios>
* [使用安全默认客户端的应用中未经身份验证或未经授权的 Web API 请求](xref:blazor/security/webassembly/additional-scenarios#unauthenticated-or-unauthorized-web-api-requests-in-an-app-with-a-secure-default-client)
