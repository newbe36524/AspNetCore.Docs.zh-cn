---
title: ASP.NET Core 中的多重身份验证
author: damienbod
description: 了解如何在 ASP.NET Core 应用中 (MFA) 设置多重身份验证。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/17/2020
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
uid: security/authentication/mfa
ms.openlocfilehash: 873f7d113df84c931ad7fbf2c72aa292e4e87c48
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060386"
---
# <a name="multi-factor-authentication-in-aspnet-core"></a>ASP.NET Core 中的多重身份验证

作者： [Damien Bowden](https://github.com/damienbod)

[ (damienbod/AspNetCoreHybridFlowWithApi GitHub 存储库查看或下载示例代码) ](https://github.com/damienbod/AspNetCoreHybridFlowWithApi)

多重身份验证 (MFA) 是在登录事件期间请求用户执行其他形式的标识的过程。 此提示可以是输入手机中的代码，使用 FIDO2 键，或提供指纹扫描。 当你需要另一种形式的身份验证时，安全性得到了增强。 攻击者无法轻松获取或复制额外的因素。

本文涵盖以下几个方面：

* 什么是 MFA 以及建议使用哪些 MFA 流
* 使用为管理页配置 MFA ASP.NET Core Identity
* 将 MFA 登录要求发送到 OpenID Connect 服务器
* 强制 ASP.NET Core OpenID Connect 客户端要求 MFA

## <a name="mfa-2fa"></a>MFA，2FA

MFA 至少需要两种或更多类型的身份验证，如你知道的东西、你拥有的内容或对用户进行身份验证的生物识别验证。

双因素身份验证 (2FA) 类似于 MFA 的子集，但不同之处在于，MFA 可能需要两个或多个因素来证明身份。

### <a name="mfa-totp-time-based-one-time-password-algorithm"></a>MFA TOTP (基于时间的一次性密码算法) 

使用 TOTP 的 MFA 是受支持的实现 ASP.NET Core Identity 。 这可以与任何兼容的验证器应用一起使用，包括：

* Microsoft Authenticator 应用
* Google 验证器应用

有关实现的详细信息，请参阅以下链接：

[为 ASP.NET Core 中的 TOTP 验证器应用启用 QR 代码生成](xref:security/authentication/identity-enable-qrcodes)

### <a name="mfa-fido2-or-passwordless"></a>MFA FIDO2 或无密码

FIDO2 目前：

* 实现 MFA 的最安全方法。
* 唯一防止仿冒攻击的 MFA 流。

目前，ASP.NET Core 不能直接支持 FIDO2。 FIDO2 可用于 MFA 或无密码流。

Azure Active Directory 提供对 FIDO2 和无密码流的支持。 有关详细信息，请参阅 [无密码 authentication options for Azure Active Directory](/azure/active-directory/authentication/concept-authentication-passwordless)。

### <a name="mfa-sms"></a>MFA 短信

与密码身份验证相比，具有短信的 MFA 增加了高度的安全性 (单个因素) 。 但是，不再建议使用短信作为第二个因素。 此类型的实现存在太多已知攻击媒介。

[NIST 指导原则](https://pages.nist.gov/800-63-3/sp800-63b.html)

## <a name="configure-mfa-for-administration-pages-using-no-locaspnet-core-identity"></a>使用为管理页配置 MFA ASP.NET Core Identity

可以强制用户在应用中访问敏感页面 ASP.NET Core Identity 。 对于不同标识存在不同级别访问权限的应用，这可能很有用。 例如，用户可以使用密码登录名查看配置文件数据，但管理员需要使用 MFA 来访问管理页面。

### <a name="extend-the-login-with-an-mfa-claim"></a>使用 MFA 声明扩展登录名

演示代码是使用和页 ASP.NET Core 设置 Identity 的 Razor 。 `AddIdentity`使用方法而不是 `AddDefaultIdentity` 一个方法，因此，在 `IUserClaimsPrincipalFactory` 成功登录后，可以使用实现将声明添加到标识。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(
            Configuration.GetConnectionString("DefaultConnection")));
    
    services.AddIdentity<IdentityUser, IdentityRole>(
            options => options.SignIn.RequireConfirmedAccount = false)
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddSingleton<IEmailSender, EmailSender>();
    services.AddScoped<IUserClaimsPrincipalFactory<IdentityUser>, 
        AdditionalUserClaimsPrincipalFactory>();

    services.AddAuthorization(options =>
        options.AddPolicy("TwoFactorEnabled",
            x => x.RequireClaim("amr", "mfa")));

    services.AddRazorPages();
}
```

`AdditionalUserClaimsPrincipalFactory`类 `amr` 仅在成功登录后将声明添加到用户声明。 将从数据库中读取声明的值。 此处添加了声明，因为如果该标识已使用 MFA 登录，则该用户只应访问受保护的视图。 如果直接从数据库中读取数据库视图而不是使用声明，则在激活 MFA 后，可以直接访问该视图，而无需进行 MFA。

```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.Extensions.Options;
using System.Collections.Generic;
using System.Security.Claims;
using System.Threading.Tasks;

namespace IdentityStandaloneMfa
{
    public class AdditionalUserClaimsPrincipalFactory : 
        UserClaimsPrincipalFactory<IdentityUser, IdentityRole>
    {
        public AdditionalUserClaimsPrincipalFactory( 
            UserManager<IdentityUser> userManager,
            RoleManager<IdentityRole> roleManager, 
            IOptions<IdentityOptions> optionsAccessor) 
            : base(userManager, roleManager, optionsAccessor)
        {
        }

        public async override Task<ClaimsPrincipal> CreateAsync(IdentityUser user)
        {
            var principal = await base.CreateAsync(user);
            var identity = (ClaimsIdentity)principal.Identity;

            var claims = new List<Claim>();

            if (user.TwoFactorEnabled)
            {
                claims.Add(new Claim("amr", "mfa"));
            }
            else
            {
                claims.Add(new Claim("amr", "pwd"));
            }

            identity.AddClaims(claims);
            return principal;
        }
    }
}
```

由于 Identity 服务设置在类中发生了更改 `Startup` ， Identity 需要更新的布局。 将 Identity 页面基架到应用。 在 *Identity /Account/Manage/_Layout cshtml* 文件中定义布局。

```cshtml
@{
    Layout = "/Pages/Shared/_Layout.cshtml";
}
```

同时为所有页面的 "管理" 页指定布局 Identity ：

```cshtml
@{
    Layout = "_Layout.cshtml";
}
```

### <a name="validate-the-mfa-requirement-in-the-administration-page"></a>在管理页中验证 MFA 要求

"管理" Razor 页将验证用户是否已使用 MFA 登录。 在 `OnGet` 方法中，标识用于访问用户声明。 `amr`检查此声明的值 `mfa` 。 如果标识缺少此声明或为 `false` ，则页面重定向到 "启用 MFA" 页。 这是可能的，因为用户已登录，但没有 MFA。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace IdentityStandaloneMfa
{
    public class AdminModel : PageModel
    {
        public IActionResult OnGet()
        {
            var claimTwoFactorEnabled = 
                User.Claims.FirstOrDefault(t => t.Type == "amr");

            if (claimTwoFactorEnabled != null && 
                "mfa".Equals(claimTwoFactorEnabled.Value))
            {
                // You logged in with MFA, do the administrative stuff
            }
            else
            {
                return Redirect(
                    "/Identity/Account/Manage/TwoFactorAuthentication");
            }

            return Page();
        }
    }
}
```

### <a name="ui-logic-to-toggle-user-login-information"></a>用于切换用户登录信息的 UI 逻辑

在启动时添加了授权策略。 策略要求声明的 `amr` 值为 `mfa` 。

```csharp
services.AddAuthorization(options =>
    options.AddPolicy("TwoFactorEnabled",
        x => x.RequireClaim("amr", "mfa")));
```

然后，可在视图中使用此策略 `_Layout` 来显示或隐藏带有警告的 **管理** 菜单：

```cshtml
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Identity
@inject SignInManager<IdentityUser> SignInManager
@inject UserManager<IdentityUser> UserManager
@inject IAuthorizationService AuthorizationService
```

如果标识已使用 MFA 登录，则会显示 " **管理** " 菜单而不显示工具提示警告。 如果用户登录时未启用 MFA，将显示 " **管理员 (未启用")** 菜单与工具提示一起显示，通知用户 (解释警告) 。

```cshtml
@if (SignInManager.IsSignedIn(User))
{
    @if ((AuthorizationService.AuthorizeAsync(User, "TwoFactorEnabled")).Result.Succeeded)
    {
        <li class="nav-item">
            <a class="nav-link text-dark" asp-area="" asp-page="/Admin">Admin</a>
        </li>
    }
    else
    {
        <li class="nav-item">
            <a class="nav-link text-dark" asp-area="" asp-page="/Admin" 
               id="tooltip-demo"  
               data-toggle="tooltip" 
               data-placement="bottom" 
               title="MFA is NOT enabled. This is required for the Admin Page. If you have activated MFA, then logout, login again.">
                Admin (Not Enabled)
            </a>
        </li>
    }
}
```

如果用户在没有 MFA 的情况下登录，将显示警告：

![管理员 MFA 身份验证](mfa/_static/identitystandalonemfa_01.png)

单击 " **管理** " 链接时，用户将重定向到 "MFA 启用" 视图：

![管理员激活 MFA 身份验证](mfa/_static/identitystandalonemfa_02.png)

## <a name="send-mfa-sign-in-requirement-to-openid-connect-server"></a>将 MFA 登录要求发送到 OpenID Connect 服务器 

`acr_values`参数可用于 `mfa` 在身份验证请求中将所需的值从客户端传递到服务器。

> [!NOTE]
> `acr_values`需要在 OpenID connect 服务器上处理参数，此参数才有效。

### <a name="openid-connect-aspnet-core-client"></a>OpenID Connect ASP.NET Core 客户端

ASP.NET Core Razor 页面 OpenID connect 客户端应用使用 `AddOpenIdConnect` 方法登录到 OpenID connect 服务器。 `acr_values`参数设置为 `mfa` 值，并随身份验证请求一起发送。 `OpenIdConnectEvents`用于添加此。

有关推荐的 `acr_values` 参数值，请参阅 [身份验证方法引用值](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08)。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(options =>
    {
        options.DefaultScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme =
            OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddOpenIdConnect(options =>
    {
        options.SignInScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.Authority = "<OpenID Connect server URL>";
        options.RequireHttpsMetadata = true;
        options.ClientId = "<OpenID Connect client ID>";
        options.ClientSecret = "<>";
        // Code with PKCE can also be used here
        options.ResponseType = "code id_token";
        options.Scope.Add("profile");
        options.Scope.Add("offline_access");
        options.SaveTokens = true;
        options.Events = new OpenIdConnectEvents
        {
            OnRedirectToIdentityProvider = context =>
            {
                context.ProtocolMessage.SetParameter("acr_values", "mfa");
                return Task.FromResult(0);
            }
        };
    });
```

### <a name="example-openid-connect-no-locidentityserver-4-server-with-no-locaspnet-core-identity"></a>示例 OpenID Connect Identity server 4 服务器与 ASP.NET Core Identity

在使用 MVC 视图实现的 OpenID Connect 服务器上 ASP.NET Core Identity ，将创建一个名为 *ErrorEnable2FA* 的新视图。 视图：

* 显示来自 Identity 需要 MFA 但用户未在中激活此应用程序的应用程序 Identity 。
* 通知用户并添加一个用于激活此的链接。

```cshtml
@{
    ViewData["Title"] = "ErrorEnable2FA";
}

<h1>The client application requires you to have MFA enabled. Enable this, try login again.</h1>

<br />

You can enable MFA to login here:

<br />

<a asp-controller="Manage" asp-action="TwoFactorAuthentication">Enable MFA</a>
```

在 `Login` 方法中， `IIdentityServerInteractionService` 接口实现 `_interaction` 用于访问 OpenID connect 请求参数。 `acr_values`使用属性访问参数 `AcrValues` 。 当客户端通过集发送此时 `mfa` ，可以检查此情况。

如果需要 MFA，并且中的用户 ASP.NET Core Identity 启用了 mfa，则登录将继续。 如果用户未启用 MFA，则会将用户重定向到自定义视图 *ErrorEnable2FA* 。 然后 ASP.NET Core Identity 在中对用户进行签名。

```csharp
//
// POST: /Account/Login
[HttpPost]
[AllowAnonymous]
[ValidateAntiForgeryToken]
public async Task<IActionResult> Login(LoginInputModel model)
{
    var returnUrl = model.ReturnUrl;
    var context = 
        await _interaction.GetAuthorizationContextAsync(returnUrl);
    var requires2Fa = 
        context?.AcrValues.Count(t => t.Contains("mfa")) >= 1;

    var user = await _userManager.FindByNameAsync(model.Email);
    if (user != null && !user.TwoFactorEnabled && requires2Fa)
    {
        return RedirectToAction(nameof(ErrorEnable2FA));
    }

    // code omitted for brevity
```

`ExternalLoginCallback`方法的工作方式类似于 Identity 本地登录。 将 `AcrValues` 检查该属性的 `mfa` 值。 如果 `mfa` 值存在，则在登录完成之前强制执行 MFA (例如，重定向到) 的 `ErrorEnable2FA` 视图。

```csharp
//
// GET: /Account/ExternalLoginCallback
[HttpGet]
[AllowAnonymous]
public async Task<IActionResult> ExternalLoginCallback(
    string returnUrl = null,
    string remoteError = null)
{
    var context =
        await _interaction.GetAuthorizationContextAsync(returnUrl);
    var requires2Fa =
        context?.AcrValues.Count(t => t.Contains("mfa")) >= 1;

    if (remoteError != null)
    {
        ModelState.AddModelError(
            string.Empty,
            _sharedLocalizer["EXTERNAL_PROVIDER_ERROR", 
            remoteError]);
        return View(nameof(Login));
    }
    var info = await _signInManager.GetExternalLoginInfoAsync();

    if (info == null)
    {
        return RedirectToAction(nameof(Login));
    }

    var email = info.Principal.FindFirstValue(ClaimTypes.Email);

    if (!string.IsNullOrEmpty(email))
    {
        var user = await _userManager.FindByNameAsync(email);
        if (user != null && !user.TwoFactorEnabled && requires2Fa)
        {
            return RedirectToAction(nameof(ErrorEnable2FA));
        }
    }

    // Sign in the user with this external login provider if the user already has a login.
    var result = await _signInManager
        .ExternalLoginSignInAsync(
            info.LoginProvider, 
            info.ProviderKey, 
            isPersistent: 
            false);

    // code omitted for brevity
```

如果用户已登录，则客户端应用：

* 仍验证 `amr` 声明。
* 可以使用指向视图的链接来设置 MFA ASP.NET Core Identity 。

![acr_values-1](mfa/_static/acr_values-1.png)

## <a name="force-aspnet-core-openid-connect-client-to-require-mfa"></a>强制 ASP.NET Core OpenID Connect 客户端要求 MFA

此示例演示如何 Razor 使用 OpenID connect 登录的 ASP.NET Core 页面应用程序可能要求用户使用 MFA 进行身份验证。

若要验证 MFA 要求，请 `IAuthorizationRequirement` 创建一个要求。 这将使用需要 MFA 的策略添加到页面中。

```csharp
using Microsoft.AspNetCore.Authorization;
 
namespace AspNetCoreRequireMfaOidc
{
    public class RequireMfa : IAuthorizationRequirement{}
}
```

`AuthorizationHandler`实现，它将使用 `amr` 声明并检查值 `mfa` 。 `amr`在 `id_token` 身份验证成功的中返回，并且可以有多个不同的值，如[身份验证方法引用值](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08)规范中所定义。

返回的值取决于标识身份验证的方式，以及 OpenID Connect 服务器实现上的身份验证方式。

`AuthorizationHandler`使用 `RequireMfa` 要求并验证 `amr` 声明。 OpenID Connect 服务器可以通过使用服务器4来实现 Identity ASP.NET Core Identity 。 当用户使用 TOTP 登录时，将 `amr` 使用 MFA 值返回声明。 如果使用不同的 OpenID Connect 服务器实现或不同的 MFA 类型，则 `amr` 声明或可以具有不同的值。 要接受此代码，还必须对代码进行扩展。

```csharp
using Microsoft.AspNetCore.Authorization;
using System;
using System.Linq;
using System.Threading.Tasks;

namespace AspNetCoreRequireMfaOidc
{
    public class RequireMfaHandler : AuthorizationHandler<RequireMfa>
    {
        protected override Task HandleRequirementAsync(
            AuthorizationHandlerContext context, 
            RequireMfa requirement)
        {
            if (context == null)
                throw new ArgumentNullException(nameof(context));
            if (requirement == null)
                throw new ArgumentNullException(nameof(requirement));

            var amrClaim =
                context.User.Claims.FirstOrDefault(t => t.Type == "amr");

            if (amrClaim != null && amrClaim.Value == Amr.Mfa)
            {
                context.Succeed(requirement);
            }

            return Task.CompletedTask;
        }
    }
}
```

在 `Startup.ConfigureServices` 方法中，将 `AddOpenIdConnect` 方法用作默认质询方案。 用于检查声明的授权处理程序 `amr` 将添加到控制容器的反转。 然后，将创建一个策略来添加 `RequireMfa` 要求。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.ConfigureApplicationCookie(options =>
        options.Cookie.SecurePolicy =
            CookieSecurePolicy.Always);

    services.AddSingleton<IAuthorizationHandler, RequireMfaHandler>();

    services.AddAuthentication(options =>
    {
        options.DefaultScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme =
            OpenIdConnectDefaults.AuthenticationScheme;
    })
    .AddCookie()
    .AddOpenIdConnect(options =>
    {
        options.SignInScheme =
            CookieAuthenticationDefaults.AuthenticationScheme;
        options.Authority = "https://localhost:44352";
        options.RequireHttpsMetadata = true;
        options.ClientId = "AspNetCoreRequireMfaOidc";
        options.ClientSecret = "AspNetCoreRequireMfaOidcSecret";
        options.ResponseType = "code id_token";
        options.Scope.Add("profile");
        options.Scope.Add("offline_access");
        options.SaveTokens = true;
    });

    services.AddAuthorization(options =>
    {
        options.AddPolicy("RequireMfa", policyIsAdminRequirement =>
        {
            policyIsAdminRequirement.Requirements.Add(new RequireMfa());
        });
    });

    services.AddRazorPages();
}
```

然后，将在页面中根据需要使用此策略 Razor 。 也可以全局为整个应用程序添加策略。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Extensions.Logging;

namespace AspNetCoreRequireMfaOidc.Pages
{
    [Authorize(Policy= "RequireMfa")]
    public class IndexModel : PageModel
    {
        private readonly ILogger<IndexModel> _logger;

        public IndexModel(ILogger<IndexModel> logger)
        {
            _logger = logger;
        }

        public void OnGet()
        {
        }
    }
}
```

如果用户不使用 MFA 进行身份验证，则 `amr` 声明可能具有 `pwd` 值。 请求不会被授权访问此页。 如果使用默认值，则用户将被重定向到 *Account/AccessDenied* 页。 此行为可以更改，也可以在此处实现自己的自定义逻辑。 在此示例中，添加了一个链接，以便有效的用户可以为其帐户设置 MFA。

```cshtml
@page
@model AspNetCoreRequireMfaOidc.AccessDeniedModel
@{
    ViewData["Title"] = "AccessDenied";
    Layout = "~/Pages/Shared/_Layout.cshtml";
}

<h1>AccessDenied</h1>

You require MFA to login here

<a href="https://localhost:44352/Manage/TwoFactorAuthentication">Enable MFA</a>
```

现在只有通过 MFA 进行身份验证的用户才能访问该页面或网站。 如果使用不同的 MFA 类型，或2FA 为正常，则 `amr` 声明将具有不同的值，并且需要正确处理。 不同的 OpenID Connect 服务器还为此声明返回不同的值，并且可能不遵循 [身份验证方法引用值](https://tools.ietf.org/html/draft-ietf-oauth-amr-values-08) 规范。

如果不使用 MFA 登录 (例如，只使用密码) ：

* `amr`具有 `pwd` 值：

    ![require_mfa_oidc_02.png](mfa/_static/require_mfa_oidc_02.png)

* 拒绝访问：

    ![require_mfa_oidc_03.png](mfa/_static/require_mfa_oidc_03.png)

或者，使用 OTP 登录 Identity ：

![require_mfa_oidc_01.png](mfa/_static/require_mfa_oidc_01.png)

## <a name="additional-resources"></a>其他资源

* [为 ASP.NET Core 中的 TOTP 验证器应用启用 QR 代码生成](xref:security/authentication/identity-enable-qrcodes)
* [Azure Active Directory 的无密码 authentication 选项](/azure/active-directory/authentication/concept-authentication-passwordless)
* [FIDO2 .NET library for FIDO2/WebAuthn 证明和断言（使用 .NET）](https://github.com/abergs/fido2-net-lib)
* [WebAuthn 出色](https://github.com/herrjemand/awesome-webauthn)
