---
title: Facebook、Google 和外部提供程序身份验证 ASP.NET Core Identity
author: rick-anderson
description: 有关使用 Facebook、Google、Twitter 等帐户用户身份验证的 ASP.NET Core Identity 说明。
ms.author: riande
ms.date: 12/10/2019
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
uid: security/authentication/social/social-without-identity
ms.openlocfilehash: cd7545a3ddaccedfa64ef5e9d5458c21c651257a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060282"
---
# <a name="use-social-sign-in-provider-authentication-without-no-locaspnet-core-identity"></a>在不使用社交登录提供程序身份验证的情况下使用 ASP.NET Core Identity

作者： [Kirk Larkin](https://twitter.com/serpent5) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

<xref:security/authentication/social/index> 描述如何使用户能够使用 OAuth 2.0 通过外部身份验证提供程序中的凭据进行登录。 该主题中所述的方法包括 ASP.NET Core Identity 作为身份验证提供程序。

此示例演示如何在不使用的 **情况下** 使用外部身份验证提供程序 ASP.NET Core Identity 。 这对于不需要的所有功能 ASP.NET Core Identity 但仍需要与受信任的外部身份验证提供程序集成的应用很有用。

此示例使用 [Google 身份验证](xref:security/authentication/google-logins) 对用户进行身份验证。 使用 Google 身份验证将管理登录过程的许多复杂性转移到 Google。 若要与其他外部身份验证提供程序集成，请参阅以下主题：

* [Facebook 身份验证](xref:security/authentication/facebook-logins)
* [Microsoft 身份验证](xref:security/authentication/microsoft-logins)
* [Twitter 身份验证](xref:security/authentication/twitter-logins)
* [其他提供程序](xref:security/authentication/otherlogins)

## <a name="configuration"></a>Configuration

在 `ConfigureServices` 方法中，使用 <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> 、 <xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*> 和方法配置应用的身份验证方案 <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> ：

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Startup.cs?name=snippet1)]

用于 <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> 设置应用的的调用 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme> 。 `DefaultScheme`是以下 `HttpContext` 身份验证扩展方法使用的默认方案：

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

将应用的设置 `DefaultScheme` 为[ Cookie AuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) ( " Cookie s" ) 将应用配置为使用 Cookie 作为这些扩展方法的默认方案。 将应用的设置 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> 为 [GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) ( "google" ) 将应用配置为使用 Google 作为调用的默认方案 `ChallengeAsync` 。 `DefaultChallengeScheme` 重写 `DefaultScheme` 。 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions>有关设置时重写的其他属性，请参阅 `DefaultScheme` 。

在中 `Startup.Configure` ， `UseAuthentication` 在 `UseAuthorization` 调用和之间调用和 `UseRouting` `UseEndpoints` 。 这会设置 `HttpContext.User` 属性并为请求运行授权中间件：

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Startup.cs?name=snippet2&highlight=3-4)]

若要详细了解身份验证方案，请参阅 [身份验证概念](xref:security/authentication/index#authentication-concepts)。 若要了解有关 cookie 身份验证的详细信息，请参阅 <xref:security/authentication/cookie> 。

## <a name="apply-authorization"></a>应用授权

通过将 `AuthorizeAttribute` 特性应用于控制器、操作或页来测试应用的身份验证配置。 以下代码将访问 *隐私* 页面的权限限制为已经过身份验证的用户：

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>注销

若要注销当前用户并将其删除 cookie ，请调用 [SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*)。 下面的代码将 `Logout` 页面处理程序添加到 *索引* 页：

[!code-csharp[](social-without-identity/samples_snapshot/3.x/Pages/Index.cshtml.cs?name=snippet&highlight=3-7)]

请注意，对的调用 `SignOutAsync` 未指定身份验证方案。 的应用程序 `DefaultScheme` 用作 `CookieAuthenticationDefaults.AuthenticationScheme` 回退。

## <a name="additional-resources"></a>其他资源

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
::: moniker range="< aspnetcore-3.0"

<xref:security/authentication/social/index> 描述如何使用户能够使用 OAuth 2.0 通过外部身份验证提供程序中的凭据进行登录。 该主题中所述的方法包括 ASP.NET Core Identity 作为身份验证提供程序。

此示例演示如何在不使用的 **情况下** 使用外部身份验证提供程序 ASP.NET Core Identity 。 这对于不需要的所有功能 ASP.NET Core Identity 但仍需要与受信任的外部身份验证提供程序集成的应用很有用。

此示例使用 [Google 身份验证](xref:security/authentication/google-logins) 对用户进行身份验证。 使用 Google 身份验证将管理登录过程的许多复杂性转移到 Google。 若要与其他外部身份验证提供程序集成，请参阅以下主题：

* [Facebook 身份验证](xref:security/authentication/facebook-logins)
* [Microsoft 身份验证](xref:security/authentication/microsoft-logins)
* [Twitter 身份验证](xref:security/authentication/twitter-logins)
* [其他提供程序](xref:security/authentication/otherlogins)

## <a name="configuration"></a>Configuration

在 `ConfigureServices` 方法中，使用 `AddAuthentication` 、 `AddCookie` 和方法配置应用的身份验证方案 `AddGoogle` ：

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Startup.cs?name=snippet1)]

对 [AddAuthentication](/dotnet/api/microsoft.extensions.dependencyinjection.authenticationservicecollectionextensions.addauthentication#Microsoft_Extensions_DependencyInjection_AuthenticationServiceCollectionExtensions_AddAuthentication_Microsoft_Extensions_DependencyInjection_IServiceCollection_System_Action_Microsoft_AspNetCore_Authentication_AuthenticationOptions__) 的调用将设置应用的 [DefaultScheme](xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultScheme)。 `DefaultScheme`是以下 `HttpContext` 身份验证扩展方法使用的默认方案：

* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.AuthenticateAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ChallengeAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.ForbidAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignInAsync*>
* <xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*>

将应用的设置 `DefaultScheme` 为[ Cookie AuthenticationDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationDefaults.AuthenticationScheme) ( " Cookie s" ) 将应用配置为使用 Cookie 作为这些扩展方法的默认方案。 将应用的设置 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions.DefaultChallengeScheme> 为 [GoogleDefaults. AuthenticationScheme](xref:Microsoft.AspNetCore.Authentication.Google.GoogleDefaults.AuthenticationScheme) ( "google" ) 将应用配置为使用 Google 作为调用的默认方案 `ChallengeAsync` 。 `DefaultChallengeScheme` 重写 `DefaultScheme` 。 <xref:Microsoft.AspNetCore.Authentication.AuthenticationOptions>有关设置时重写的其他属性，请参阅 `DefaultScheme` 。

在 `Configure` 方法中，调用 `UseAuthentication` 方法以调用设置属性的身份验证中间件 `HttpContext.User` 。 `UseAuthentication`在调用或之前调用 `UseMvcWithDefaultRoute` 方法 `UseMvc` ：

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Startup.cs?name=snippet2)]

若要详细了解身份验证方案，请参阅 [身份验证概念](xref:security/authentication/index#authentication-concepts)。 若要了解有关 cookie 身份验证的详细信息，请参阅 <xref:security/authentication/cookie> 。

## <a name="apply-authorization"></a>应用授权

通过将 `AuthorizeAttribute` 特性应用于控制器、操作或页来测试应用的身份验证配置。 以下代码将访问 *隐私* 页面的权限限制为已经过身份验证的用户：

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Pages/Privacy.cshtml.cs?name=snippet&highlight=1)]

## <a name="sign-out"></a>注销

若要注销当前用户并将其删除 cookie ，请调用 [SignOutAsync](xref:Microsoft.AspNetCore.Authentication.AuthenticationHttpContextExtensions.SignOutAsync*)。 下面的代码将 `Logout` 页面处理程序添加到 *索引* 页：

[!code-csharp[](social-without-identity/samples_snapshot/2.x/Pages/Index.cshtml.cs?name=snippet&highlight=3-7)]

请注意，对的调用 `SignOutAsync` 未指定身份验证方案。 的应用程序 `DefaultScheme` 用作 `CookieAuthenticationDefaults.AuthenticationScheme` 回退。

## <a name="additional-resources"></a>其他资源

* <xref:security/authorization/simple>
* <xref:security/authentication/social/additional-claims>

::: moniker-end
