---
title: 保护 ASP.NET Core Blazor WebAssembly
author: guardrex
description: 了解如何将 Blazor WebAssemlby 应用作为单页应用程序 (SPA) 进行保护。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/16/2020
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
uid: blazor/security/webassembly/index
ms.openlocfilehash: e1af8f1de61edd934505a44e75ea07e0f09a67b5
ms.sourcegitcommit: a07f83b00db11f32313045b3492e5d1ff83c4437
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90592951"
---
# <a name="secure-aspnet-core-no-locblazor-webassembly"></a>保护 ASP.NET Core Blazor WebAssembly

作者：[Javier Calvarro Nelson](https://github.com/javiercn)

Blazor WebAssembly 应用的保护方式与单页应用 (SPA) 相同。 可通过多种方式向 SPA 进行用户身份验证，但最常用、最全面的方式是使用基于 [OAuth 2.0 协议](https://oauth.net/)的实现，例如 [OpenID Connect (OIDC)](https://openid.net/connect/)。

## <a name="authentication-library"></a>身份验证库

Blazor WebAssembly 支持通过 [`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) 库使用 OIDC 对应用进行身份验证和授权。 该库提供一组基元，可用于对 ASP.NET Core 后端进行无缝身份验证。 这个库将 ASP.NET Core Identity 与以 [Identity 服务器](https://identityserver.io/)为基础的 API 身份验证支持集成。 它可以针对支持 OIDC 的任何第三方 Identity 提供者 (IP)，即 OpenID 提供者 (OP)，进行身份验证。

Blazor WebAssembly 中的身份验证支持建立在 `oidc-client.js` 库的基础之上，此库用于处理基础身份验证协议详细信息。

还有对 SPA 进行身份验证的其他选项，例如使用 SameSite cookie。 但是，Blazor WebAssembly 的工程设计决定，OAuth 和 OIDC 是在 Blazor WebAssembly 应用中进行身份验证的最佳选择。 出于以下功能和安全原因，选择了以 [JSON Web 令牌 (JWT)](https://self-issued.info/docs/draft-ietf-oauth-json-web-token.html) 为基础的[基于令牌的身份验证](xref:security/anti-request-forgery#token-based-authentication)而不是[基于 cookie 的身份验证](xref:security/anti-request-forgery#cookie-based-authentication)：

* 使用基于令牌的协议可以减小攻击面，因为并非所有请求中都会发送令牌。
* 服务器终结点不要求针对[跨站点请求伪造 (CSRF)](xref:security/anti-request-forgery) 进行保护，因为会显式发送令牌。 因此，可以将 Blazor WebAssembly 应用与 MVC 或 Razor Pages 应用一起托管。
* 令牌的权限比 cookie 窄。 例如，令牌不能用于管理用户帐户或更改用户密码，除非显式实现了此类功能。
* 令牌的生命周期更短（默认为一小时），这限制了攻击时间窗口。 还可随时撤销令牌。
* 自包含 JWT 向客户端和服务器提供身份验证进程保证。 例如，客户端可以检测和验证它收到的令牌是否合法，以及是否是在给定身份验证过程中发出的。 如果有第三方尝试在身份验证进程中偷换令牌，客户端可以检测被偷换的令牌并避免使用它。
* OAuth 和 OIDC 的令牌不依赖于用户代理行为正确以确保应用安全。
* 基于令牌的协议（例如 OAuth 和 OIDC）允许用同一组安全特征对托管和独立应用进行验证和授权。

## <a name="authentication-process-with-oidc"></a>使用 OIDC 的身份验证进程

[`Microsoft.AspNetCore.Components.WebAssembly.Authentication`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.WebAssembly.Authentication) 库提供几个基元，用于使用 OIDC 实现身份验证和授权。 从广义上说来，身份验证的原理如下：

* 当匿名用户选择登录按钮或请求应用了 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 特性的页面时，会将其重定向到应用的登录页 (`/authentication/login`)。
* 在登录页上，身份验证库准备重定向到授权终结点。 授权终结点在 Blazor WebAssembly 应用之外，可以托管在单独的原点上。 该终结点负责确定用户是否通过身份验证，并发送一个或更多令牌作为响应。 身份验证库提供登录回叫以接收身份验证响应。
  * 如果用户未通过身份验证，则会被重定向到底层身份验证系统，通常是 ASP.NET Core Identity。
  * 如果用户已通过身份验证，则授权终结点会生成相应的令牌，并将浏览器重定向回登录回叫终结点 (`/authentication/login-callback`)。
* 当 Blazor WebAssembly 应用加载登录回叫终结点 (`/authentication/login-callback`) 时，就处理了身份验证响应。
  * 如果身份验证进程成功完成，则用户通过身份验证，可以选择返回该用户请求的原受保护 URL。
  * 如果身份验证进程由于任何原因而失败，会将用户导向登录失败页 (`/authentication/login-failed`)，并显示错误。

## <a name="authentication-component"></a>`Authentication` 组件

`Authentication` 组件 (`Pages/Authentication.razor`) 会处理远程身份验证操作并允许应用：

* 为身份验证状态配置应用路由。
* 为身份验证状态设置 UI 内容。
* 管理身份验证状态。

身份验证操作（例如注册用户或使用户登录）传递到 Blazor 框架的 <xref:Microsoft.AspNetCore.Components.WebAssembly.Authentication.RemoteAuthenticatorViewCore%601> 组件，该组件会保留和控制整个身份验证操作中的状态。

有关更多信息和示例，请参见<xref:blazor/security/webassembly/additional-scenarios>。

## <a name="authorization"></a>授权

在 Blazor WebAssembly 应用中，可绕过授权检查，因为用户可修改所有客户端代码。 所有客户端应用程序技术都是如此，其中包括 JavaScript SPA 框架或任何操作系统的本机应用程序。

**始终对客户端应用程序访问的任何 API 终结点内的服务器执行授权检查。**

## <a name="require-authorization-for-the-entire-app"></a>需要对整个应用授权

使用以下方法之一将 [`[Authorize]` 属性](xref:blazor/security/index#authorize-attribute)（[API 文档](xref:System.Web.Mvc.AuthorizeAttribute)）应用到应用的每个 Razor 组件：

* 在 `_Imports.razor` 文件中使用 [`@attribute`](xref:mvc/views/razor#attribute) 指令：

  ```razor
  @using Microsoft.AspNetCore.Authorization
  @attribute [Authorize]
  ```

* 向 `Pages` 文件夹中的每个 Razor 组件添加属性。

> [!NOTE]
> 不支持使用 <xref:Microsoft.AspNetCore.Authorization.AuthorizationPolicyBuilder.RequireAuthenticatedUser%2A> 将 <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy?displayProperty=nameWithType> 设置为策略。

## <a name="refresh-tokens"></a>刷新令牌

在 Blazor WebAssembly 应用中，无法在客户端保护刷新令牌。 因此，不得将刷新令牌发送到应用以供直接使用。

在托管的 Blazor WebAssembly 解决方案中，服务器端应用可以维护和使用刷新令牌来访问第三方 API。 有关详细信息，请参阅 <xref:blazor/security/webassembly/additional-scenarios#authenticate-users-with-a-third-party-provider-and-call-protected-apis-on-the-host-server-and-the-third-party>。

## <a name="establish-claims-for-users"></a>为用户建立声明

应用通常要求用户声明基于对服务器的 web API 调用。 例如，声明常用于在应用中[建立授权](xref:blazor/security/index#authorization)。 在这些情况下，应用会请求访问令牌来访问服务，并使用该令牌获取声明的用户数据。 有关示例，请参阅以下资源：

* [其他场景：自定义用户](xref:blazor/security/webassembly/additional-scenarios#customize-the-user)
* <xref:blazor/security/webassembly/aad-groups-roles>

## <a name="implementation-guidance"></a>实施指南

此“概述”下的文章介绍了如何针对特定提供商对 Blazor WebAssembly 应用中的用户进行身份验证。

独立 Blazor WebAssembly 应用：

* [OIDC 提供程序和 WebAssembly 身份验证库的通用指南](xref:blazor/security/webassembly/standalone-with-authentication-library)
* [Microsoft 帐户](xref:blazor/security/webassembly/standalone-with-microsoft-accounts)
* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/standalone-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/standalone-with-azure-active-directory-b2c)

托管 Blazor WebAssembly 应用：

* [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
* [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
* [Identity 服务器](xref:blazor/security/webassembly/hosted-with-identity-server)

有关配置方面的更多指南，请参阅 <xref:blazor/security/webassembly/additional-scenarios>。
