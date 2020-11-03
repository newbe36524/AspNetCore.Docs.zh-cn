---
title: 将身份验证和升级 Identity 到 ASP.NET Core 2。0
author: scottaddie
description: 本文概述迁移 ASP.NET Core 1.x 身份验证和 ASP.NET Core 2.0 的最常见步骤 Identity 。
ms.author: scaddie
ms.date: 06/21/2019
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
uid: migration/1x-to-2x/identity-2x
ms.openlocfilehash: cad7582670013661f5fcbfbebad923f0f092462e
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93057175"
---
# <a name="migrate-authentication-and-no-locidentity-to-aspnet-core-20"></a>将身份验证和升级 Identity 到 ASP.NET Core 2。0

作者： [Scott Addie](https://github.com/scottaddie) 和 [Hao Kung](https://github.com/HaoK)

ASP.NET Core 2.0 具有新的身份验证模型，并 [Identity](xref:security/authentication/identity) 使用服务简化了配置。 ASP.NET Core 1.x 使用身份验证的应用程序或 Identity 可以更新为使用如下所述的新模型。

## <a name="update-namespaces"></a>更新命名空间

在1.x 中，在 `IdentityRole` `IdentityUser` 命名空间中找到了类，如和 `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 。

在2.0 中， <xref:Microsoft.AspNetCore.Identity> 命名空间成为几个这样的类的新宿主。 对于默认 Identity 代码，受影响的类包括 `ApplicationUser` 和 `Startup` 。 调整 `using` 语句以解析受影响的引用。

<a name="auth-middleware"></a>

## <a name="authentication-middleware-and-services"></a>身份验证中间件和服务

在1.x 项目中，通过中间件配置身份验证。 为要支持的每个身份验证方案调用中间件方法。

下面的1.x 示例 Identity 在 *Startup.cs* 中配置 Facebook 身份验证：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext>();
}

public void Configure(IApplicationBuilder app, ILoggerFactory loggerfactory)
{
    app.UseIdentity();
    app.UseFacebookAuthentication(new FacebookOptions {
        AppId = Configuration["auth:facebook:appid"],
        AppSecret = Configuration["auth:facebook:appsecret"]
    });
}
```

在2.0 项目中，通过服务配置身份验证。 每个身份验证方案都是在 Startup.cs 的方法中注册的 `ConfigureServices` 。 *Startup.cs* 此 `UseIdentity` 方法将替换为 `UseAuthentication` 。

以下2.0 示例 Identity 在 *Startup.cs* 中配置 Facebook 身份验证：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
            .AddEntityFrameworkStores<ApplicationDbContext>();

    // If you want to tweak Identity cookies, they're no longer part of IdentityOptions.
    services.ConfigureApplicationCookie(options => options.LoginPath = "/Account/LogIn");
    services.AddAuthentication()
            .AddFacebook(options =>
            {
                options.AppId = Configuration["auth:facebook:appid"];
                options.AppSecret = Configuration["auth:facebook:appsecret"];
            });
}

public void Configure(IApplicationBuilder app, ILoggerFactory loggerfactory) {
    app.UseAuthentication();
}
```

`UseAuthentication`方法添加单个身份验证中间件组件，该组件负责自动身份验证和远程身份验证请求的处理。 它将所有单个中间件组件替换为一个公共中间件组件。

下面是每个主要身份验证方案的2.0 迁移说明。

### <a name="no-loccookie-based-authentication"></a>Cookie基于的身份验证

选择以下两个选项之一，并在 *Startup.cs* 中进行必要的更改：

1. 使用 cookieIdentity
    - `UseIdentity` `UseAuthentication` 在方法中将替换为 `Configure` ：

        ```csharp
        app.UseAuthentication();
        ```

    - `AddIdentity`在方法中调用方法 `ConfigureServices` 以添加 cookie 身份验证服务。
    - 还可以 `ConfigureApplicationCookie` `ConfigureExternalCookie` 在方法中调用或方法 `ConfigureServices` 来调整 Identity cookie 设置。

        ```csharp
        services.AddIdentity<ApplicationUser, IdentityRole>()
                .AddEntityFrameworkStores<ApplicationDbContext>()
                .AddDefaultTokenProviders();

        services.ConfigureApplicationCookie(options => options.LoginPath = "/Account/LogIn");
        ```

2. 使用 cookie 不带 Identity
    - `UseCookieAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

        ```csharp
        app.UseAuthentication();
        ```

    - `AddAuthentication` `AddCookie` 在方法中调用和方法 `ConfigureServices` ：

        ```csharp
        // If you don't want the cookie to be automatically authenticated and assigned to HttpContext.User,
        // remove the CookieAuthenticationDefaults.AuthenticationScheme parameter passed to AddAuthentication.
        services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
                .AddCookie(options =>
                {
                    options.LoginPath = "/Account/LogIn";
                    options.LogoutPath = "/Account/LogOff";
                });
        ```

### <a name="jwt-bearer-authentication"></a>JWT 持有者身份验证

在 *Startup.cs* 中进行以下更改：
- `UseJwtBearerAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

    ```csharp
    app.UseAuthentication();
    ```

- `AddJwtBearer`在方法中调用方法 `ConfigureServices` ：

    ```csharp
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.Audience = "http://localhost:5001/";
                options.Authority = "http://localhost:5000/";
            });
    ```

    此代码段不使用 Identity ，因此应通过传递给方法来设置默认方案 `JwtBearerDefaults.AuthenticationScheme` `AddAuthentication` 。

### <a name="openid-connect-oidc-authentication"></a>OpenID Connect (OIDC) 身份验证

在 *Startup.cs* 中进行以下更改：

- `UseOpenIdConnectAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

    ```csharp
    app.UseAuthentication();
    ```

- `AddOpenIdConnect`在方法中调用方法 `ConfigureServices` ：

    ```csharp
    services.AddAuthentication(options =>
    {
        options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddOpenIdConnect(options =>
    {
        options.Authority = Configuration["auth:oidc:authority"];
        options.ClientId = Configuration["auth:oidc:clientid"];
    });
    ```

- `PostLogoutRedirectUri`将操作中的属性替换为 `OpenIdConnectOptions` `SignedOutRedirectUri` ：

    ```csharp
    .AddOpenIdConnect(options =>
    {
        options.SignedOutRedirectUri = "https://contoso.com";
    });
    ```
    
### <a name="facebook-authentication"></a>Facebook 身份验证

在 *Startup.cs* 中进行以下更改：
- `UseFacebookAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

    ```csharp
    app.UseAuthentication();
    ```

- `AddFacebook`在方法中调用方法 `ConfigureServices` ：

    ```csharp
    services.AddAuthentication()
            .AddFacebook(options =>
            {
                options.AppId = Configuration["auth:facebook:appid"];
                options.AppSecret = Configuration["auth:facebook:appsecret"];
            });
    ```

### <a name="google-authentication"></a>Google 身份验证

在 *Startup.cs* 中进行以下更改：
- `UseGoogleAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

    ```csharp
    app.UseAuthentication();
    ```

- `AddGoogle`在方法中调用方法 `ConfigureServices` ：

    ```csharp
    services.AddAuthentication()
            .AddGoogle(options =>
            {
                options.ClientId = Configuration["auth:google:clientid"];
                options.ClientSecret = Configuration["auth:google:clientsecret"];
            });
    ```

### <a name="microsoft-account-authentication"></a>Microsoft 帐户身份验证

有关 Microsoft 帐户身份验证的详细信息，请参阅 [此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/14455)。

在 *Startup.cs* 中进行以下更改：
- `UseMicrosoftAccountAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

    ```csharp
    app.UseAuthentication();
    ```

- `AddMicrosoftAccount`在方法中调用方法 `ConfigureServices` ：

    ```csharp
    services.AddAuthentication()
            .AddMicrosoftAccount(options =>
            {
                options.ClientId = Configuration["auth:microsoft:clientid"];
                options.ClientSecret = Configuration["auth:microsoft:clientsecret"];
            });
    ```

### <a name="twitter-authentication"></a>Twitter 身份验证

在 *Startup.cs* 中进行以下更改：
- `UseTwitterAuthentication`将方法中的方法调用替换 `Configure` 为 `UseAuthentication` ：

    ```csharp
    app.UseAuthentication();
    ```

- `AddTwitter`在方法中调用方法 `ConfigureServices` ：

    ```csharp
    services.AddAuthentication()
            .AddTwitter(options =>
            {
                options.ConsumerKey = Configuration["auth:twitter:consumerkey"];
                options.ConsumerSecret = Configuration["auth:twitter:consumersecret"];
            });
    ```

### <a name="setting-default-authentication-schemes"></a>设置默认身份验证方案

在1.x 中，将 `AutomaticAuthenticate` `AutomaticChallenge` 在单个身份验证方案中设置 [AuthenticationOptions](/dotnet/api/Microsoft.AspNetCore.Builder.AuthenticationOptions?view=aspnetcore-1.1) 基类的和属性。 没有正确的方法来强制执行此操作。

在2.0 中，这两个属性已作为各个实例的属性删除 `AuthenticationOptions` 。 可以在 Startup.cs 的方法中的方法调用中配置它们 `AddAuthentication` `ConfigureServices` ： *Startup.cs*

```csharp
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme);
```

在上面的代码段中，默认方案设置为 `CookieAuthenticationDefaults.AuthenticationScheme` ( " Cookie s" ) 。

或者，使用方法的重载版本 `AddAuthentication` 设置多个属性。 在下面的重载方法示例中，将默认方案设置为 `CookieAuthenticationDefaults.AuthenticationScheme` 。 也可以在单个 `[Authorize]` 属性或授权策略中指定身份验证方案。

```csharp
services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
});
```

如果满足以下条件之一，则定义2.0 中的默认方案：
- 希望用户自动登录
- 在 `[Authorize]` 不指定方案的情况下使用属性或授权策略

此规则的例外情况是 `AddIdentity` 方法。 此方法 cookie 为你添加了，并将默认身份验证和质询方案设置为应用程序 cookie `IdentityConstants.ApplicationScheme` 。 此外，它还将默认登录方案设置为外部 cookie `IdentityConstants.ExternalScheme` 。

<a name="obsolete-interface"></a>

## <a name="use-httpcontext-authentication-extensions"></a>使用 HttpContext 身份验证扩展

`IAuthenticationManager`接口是到1.x 身份验证系统的主要入口点。 它已替换为命名空间中的一组新 `HttpContext` 扩展方法 `Microsoft.AspNetCore.Authentication` 。

例如，1.x 项目引用 `Authentication` 属性：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

在2.0 项目中，导入 `Microsoft.AspNetCore.Authentication` 命名空间，并删除 `Authentication` 属性引用：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

<a name="windows-auth-changes"></a>

## <a name="windows-authentication-httpsys--iisintegration"></a>Windows 身份验证 ( # A0/IISIntegration) 

Windows 身份验证有两种变体：

* 该主机仅允许经过身份验证的用户。 此变体不受2.0 更改的影响。
* 宿主允许匿名用户和经过身份验证的用户。 此变体受2.0 更改的影响。 例如，应用程序应该允许 [IIS](xref:host-and-deploy/iis/index) 或 [HTTP.sys](xref:fundamentals/servers/httpsys) 层上的匿名用户，但在控制器级别对用户进行授权。 在此方案中，在方法中设置默认方案 `Startup.ConfigureServices` 。

  对于 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IISIntegration/)，将默认方案设置为 `IISDefaults.AuthenticationScheme` ：

  ```csharp
  using Microsoft.AspNetCore.Server.IISIntegration;

  services.AddAuthentication(IISDefaults.AuthenticationScheme);
  ```

  对于 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/)，将默认方案设置为 `HttpSysDefaults.AuthenticationScheme` ：

  ```csharp
  using Microsoft.AspNetCore.Server.HttpSys;

  services.AddAuthentication(HttpSysDefaults.AuthenticationScheme);
  ```

  未能设置默认方案会阻止授权 (质询) 请求使用以下例外：

  > `System.InvalidOperationException`：未指定 authenticationScheme，且未找到 DefaultChallengeScheme。

有关详细信息，请参阅 <xref:security/authentication/windowsauth>。

<a name="identity-cookie-options"></a>

## <a name="no-locidentityno-loccookieoptions-instances"></a>IdentityCookie选项实例

2.0 更改的副作用是切换到使用命名选项而不是 cookie 选项实例。 将删除自定义 Identity cookie 方案名称的功能。

例如，1.x 项目使用 [构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection) 将 `IdentityCookieOptions` 参数传递到 *AccountController.cs* 和 *ManageController.cs* 中。 cookie可从提供的实例中访问外部身份验证方案：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/AccountController.cs?name=snippet_AccountControllerConstructor&highlight=4,11)]

上述构造函数注入在2.0 项目中变得不必要， `_externalCookieScheme` 可以删除该字段：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AccountControllerConstructor)]

1.x 项目使用此字段， `_externalCookieScheme` 如下所示：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

在2.0 项目中，将前面的代码替换为以下代码。 `IdentityConstants.ExternalScheme`可以直接使用该常量。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AuthenticationProperty)]

`SignOutAsync`通过导入以下命名空间来解析新添加的调用：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/AccountController.cs?name=snippet_AuthenticationImport)]

<a name="navigation-properties"></a>

## <a name="add-no-locidentityuser-poco-navigation-properties"></a>添加 Identity 用户 POCO 导航属性

已 `IdentityUser` 删除基本 POCO (普通旧 CLR 对象) 的实体框架 (EF) Core 导航属性。 如果你的1.x 项目使用这些属性，请将其手动添加回2.0 项目：

```csharp
/// <summary>
/// Navigation property for the roles this user belongs to.
/// </summary>
public virtual ICollection<IdentityUserRole<int>> Roles { get; } = new List<IdentityUserRole<int>>();

/// <summary>
/// Navigation property for the claims this user possesses.
/// </summary>
public virtual ICollection<IdentityUserClaim<int>> Claims { get; } = new List<IdentityUserClaim<int>>();

/// <summary>
/// Navigation property for this users login accounts.
/// </summary>
public virtual ICollection<IdentityUserLogin<int>> Logins { get; } = new List<IdentityUserLogin<int>>();
```

若要防止在运行 EF Core 迁移时出现重复的外键，请在 `IdentityDbContext` 调用) 后将以下内容添加到类的 `OnModelCreating` 方法 (`base.OnModelCreating();` ：

```csharp
protected override void OnModelCreating(ModelBuilder builder)
{
    base.OnModelCreating(builder);
    // Customize the ASP.NET Core Identity model and override the defaults if needed.
    // For example, you can rename the ASP.NET Core Identity table names and more.
    // Add your customizations after calling base.OnModelCreating(builder);

    builder.Entity<ApplicationUser>()
        .HasMany(e => e.Claims)
        .WithOne()
        .HasForeignKey(e => e.UserId)
        .IsRequired()
        .OnDelete(DeleteBehavior.Cascade);

    builder.Entity<ApplicationUser>()
        .HasMany(e => e.Logins)
        .WithOne()
        .HasForeignKey(e => e.UserId)
        .IsRequired()
        .OnDelete(DeleteBehavior.Cascade);

    builder.Entity<ApplicationUser>()
        .HasMany(e => e.Roles)
        .WithOne()
        .HasForeignKey(e => e.UserId)
        .IsRequired()
        .OnDelete(DeleteBehavior.Cascade);
}
```

<a name="synchronous-method-removal"></a>

## <a name="replace-getexternalauthenticationschemes"></a>替换 GetExternalAuthenticationSchemes

删除同步方法 `GetExternalAuthenticationSchemes` 是为了支持异步版本。 1.x 项目的 *控制器/ManageController* 中包含以下代码：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Controllers/ManageController.cs?name=snippet_GetExternalAuthenticationSchemes)]

此方法也出现在 *Views/Account/Login 中。 cshtml* ：

[!code-cshtml[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Views/Account/Login.cshtml?name=snippet_GetExtAuthNSchemes&highlight=2)]

在2.0 项目中，请使用 <xref:Microsoft.AspNetCore.Identity.SignInManager`1.GetExternalAuthenticationSchemesAsync*> 方法。 *ManageController.cs* 中的更改类似于以下代码：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Controllers/ManageController.cs?name=snippet_GetExternalAuthenticationSchemesAsync)]

在 *Login* 中， `AuthenticationScheme` 在循环中访问的属性 `foreach` 将更改为 `Name` ：

[!code-cshtml[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Views/Account/Login.cshtml?name=snippet_GetExtAuthNSchemesAsync&highlight=2,19)]

<a name="property-change"></a>

## <a name="manageloginsviewmodel-property-change"></a>ManageLoginsViewModel 属性更改

`ManageLoginsViewModel`对象用于 `ManageLogins` *ManageController.cs* 的操作。 在1.x 项目中，对象的 `OtherLogins` 属性返回类型为 `IList<AuthenticationDescription>` 。 此返回类型需要导入 `Microsoft.AspNetCore.Http.Authentication` ：

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore1App/AspNetCoreDotNetCore1App/Models/ManageViewModels/ManageLoginsViewModel.cs?name=snippet_ManageLoginsViewModel&highlight=2,11)]

在2.0 项目中，返回类型更改为 `IList<AuthenticationScheme>` 。 这一新的返回类型需要 `Microsoft.AspNetCore.Http.Authentication` 使用导入替换导入 `Microsoft.AspNetCore.Authentication` 。

[!code-csharp[](../1x-to-2x/samples/AspNetCoreDotNetCore2App/AspNetCoreDotNetCore2App/Models/ManageViewModels/ManageLoginsViewModel.cs?name=snippet_ManageLoginsViewModel&highlight=2,11)]

<a name="additional-resources"></a>

## <a name="additional-resources"></a>其他资源

有关详细信息，请参阅 GitHub 上 [的有关 Auth 2.0](https://github.com/aspnet/Security/issues/1338) 问题的讨论。
