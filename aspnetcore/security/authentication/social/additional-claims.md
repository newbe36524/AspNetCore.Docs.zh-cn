---
title: 在 ASP.NET Core 中保存外部提供程序的其他声明和令牌
author: rick-anderson
description: 了解如何建立来自外部提供程序的其他声明和标记。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/30/2020
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
uid: security/authentication/social/additional-claims
ms.openlocfilehash: 4503291ff887f79b1ad6eacd4e56943ce23335bc
ms.sourcegitcommit: 5156eab2118584405eb663e1fcd82f8bd7764504
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/31/2020
ms.locfileid: "93141503"
---
# <a name="persist-additional-claims-and-tokens-from-external-providers-in-aspnet-core"></a>在 ASP.NET Core 中保存外部提供程序的其他声明和令牌

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core 应用可以从外部身份验证提供程序（如 Facebook、Google、Microsoft 和 Twitter）建立其他声明和令牌。 每个提供程序都在其平台上显示有关用户的不同信息，但用于接收用户数据并将其转换为其他声明的模式是相同的。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="prerequisites"></a>先决条件

确定要在应用程序中支持的外部身份验证提供程序。 对于每个提供程序，注册应用程序，并获取客户端 ID 和客户端密码。 有关详细信息，请参阅 <xref:security/authentication/social/index>。 示例应用使用 [Google 身份验证提供程序](xref:security/authentication/google-logins)。

## <a name="set-the-client-id-and-client-secret"></a>设置客户端 ID 和客户端密码

OAuth 身份验证提供程序使用客户端 ID 和客户端密码与应用程序建立了信任关系。 当向提供程序注册应用程序时，外部身份验证提供程序会为应用程序创建客户端 ID 和客户端机密值。 应用使用的每个外部提供程序必须与提供程序的客户端 ID 和客户端机密一起单独配置。 有关详细信息，请参阅适用于你的方案的外部身份验证提供程序主题：

* [Facebook 身份验证](xref:security/authentication/facebook-logins)
* [Google 身份验证](xref:security/authentication/google-logins)
* [Microsoft 身份验证](xref:security/authentication/microsoft-logins)
* [Twitter 身份验证](xref:security/authentication/twitter-logins)
* [其他身份验证提供程序](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

示例应用使用 Google 提供的客户端 ID 和客户端机密配置 Google 身份验证提供程序：

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>建立身份验证范围

指定要从提供程序检索的权限的列表 <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> 。 常见外部提供程序的身份验证范围如下表中所示。

| 提供程序  | 范围                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `profile`, `email`, `openid`                                     |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

在示例应用中， `profile` `email` `openid` 当在上调用时，由框架自动添加 Google 的、和作用域 <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle%2A> <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> 。 如果应用需要其他作用域，请将它们添加到选项。 在以下示例中，添加了 Google `https://www.googleapis.com/auth/user.birthday.read` 范围以检索用户的生日：

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>映射用户数据密钥并创建声明

在提供程序的选项中，为 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> 外部提供程序的 JSON 用户数据中的每个键/子项指定或，以便在登录时读取应用程序标识。 有关声明类型的详细信息，请参阅 <xref:System.Security.Claims.ClaimTypes> 。

该示例应用程序 (`urn:google:locale`) 和图片 (`urn:google:picture` 从 `locale` `picture` Google 用户数据中的和密钥) 声明创建区域设置：

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

在中 `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` ， <xref:Microsoft.AspNetCore.Identity.IdentityUser> (`ApplicationUser`) 使用登录到应用 <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> 。 在登录过程中， <xref:Microsoft.AspNetCore.Identity.UserManager%601> 可以存储 `ApplicationUser` 提供的用户数据的声明 <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> 。

在示例应用中， `OnPostConfirmationAsync` ( *帐户/ExternalLogin* ) 将 (的 `urn:google:locale`) 和图片 (`urn:google:picture`) 声明中建立区域设置 `ApplicationUser` ，包括的声明 <xref:System.Security.Claims.ClaimTypes.GivenName> ：

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

默认情况下，会在身份验证中存储用户的声明 cookie 。 如果身份验证 cookie 太大，则可能会导致应用失败，原因是：

* 浏览器检测到 cookie 标头太长。
* 请求的整体大小太大。

如果需要大量用户数据来处理用户请求：

* 将请求处理的用户声明的数量和大小限制为仅应用需要的内容。
* 使用 <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> 用于 Cookie 身份验证中间件的自定义 <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> 来跨请求存储标识。 在服务器上保留大量标识信息，同时仅向客户端发送一个小型会话标识符密钥。

## <a name="save-the-access-token"></a>保存访问令牌

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> 定义在授权成功后是否应将访问和刷新令牌存储在中 <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> 。 `SaveTokens` 默认情况下，设置为以 `false` 减小最终身份验证的大小 cookie 。

示例应用将的值设置 `SaveTokens` 为 `true` 中的 <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> ：

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

`OnPostConfirmationAsync`执行时，将访问令牌存储在的外部提供程序中， ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) 。 `ApplicationUser` `AuthenticationProperties`

示例应用程序将访问令牌保存在 `OnPostConfirmationAsync` (新用户注册) 并 `OnGetCallbackAsync` (之前注册的用户) *帐户/ExternalLogin 中。*

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a>如何添加其他自定义令牌

为了演示如何添加作为的一部分存储的自定义令牌， `SaveTokens` 示例应用程序会 <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> <xref:System.DateTime> 为的 [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) 添加一个，其中包含 `TicketCreated` ：

[!code-csharp[](additional-claims/samples/3.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>创建和添加声明

框架提供用于创建声明并将其添加到集合的常见操作和扩展方法。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> 和 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>。

用户可以通过从派生 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> 并实现抽象方法来定义自定义操作 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> 。

有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>。

## <a name="removal-of-claim-actions-and-claims"></a>删除声明操作和声明

[ClaimActionCollection (字符串) ](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) 将从集合中移除给定的所有声明操作 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> 。 [ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection，String) ](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) 从标识中删除给定的声明 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> 。 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> 主要与 [OpenID connect (OIDC) ](/azure/active-directory/develop/v2-protocols-oidc) 一起使用，以删除协议生成的声明。

## <a name="sample-app-output"></a>示例应用程序输出

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core 应用可以从外部身份验证提供程序（如 Facebook、Google、Microsoft 和 Twitter）建立其他声明和令牌。 每个提供程序都在其平台上显示有关用户的不同信息，但用于接收用户数据并将其转换为其他声明的模式是相同的。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/social/additional-claims/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="prerequisites"></a>先决条件

确定要在应用程序中支持的外部身份验证提供程序。 对于每个提供程序，注册应用程序，并获取客户端 ID 和客户端密码。 有关详细信息，请参阅 <xref:security/authentication/social/index>。 示例应用使用 [Google 身份验证提供程序](xref:security/authentication/google-logins)。

## <a name="set-the-client-id-and-client-secret"></a>设置客户端 ID 和客户端密码

OAuth 身份验证提供程序使用客户端 ID 和客户端密码与应用程序建立了信任关系。 当向提供程序注册应用程序时，外部身份验证提供程序会为应用程序创建客户端 ID 和客户端机密值。 应用使用的每个外部提供程序必须与提供程序的客户端 ID 和客户端机密一起单独配置。 有关详细信息，请参阅适用于你的方案的外部身份验证提供程序主题：

* [Facebook 身份验证](xref:security/authentication/facebook-logins)
* [Google 身份验证](xref:security/authentication/google-logins)
* [Microsoft 身份验证](xref:security/authentication/microsoft-logins)
* [Twitter 身份验证](xref:security/authentication/twitter-logins)
* [其他身份验证提供程序](xref:security/authentication/otherlogins)
* [OpenIdConnect](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2)

示例应用使用 Google 提供的客户端 ID 和客户端机密配置 Google 身份验证提供程序：

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=4,9)]

## <a name="establish-the-authentication-scope"></a>建立身份验证范围

指定要从提供程序检索的权限的列表 <xref:Microsoft.AspNetCore.Authentication.OAuth.OAuthOptions.Scope*> 。 常见外部提供程序的身份验证范围如下表中所示。

| 提供程序  | 范围                                                            |
| --------- | ---------------------------------------------------------------- |
| Facebook  | `https://www.facebook.com/dialog/oauth`                          |
| Google    | `https://www.googleapis.com/auth/userinfo.profile`               |
| Microsoft | `https://login.microsoftonline.com/common/oauth2/v2.0/authorize` |
| Twitter   | `https://api.twitter.com/oauth/authenticate`                     |

在示例应用中， `userinfo.profile` 当在上调用时，由框架自动添加 Google 的作用域 <xref:Microsoft.Extensions.DependencyInjection.GoogleExtensions.AddGoogle*> <xref:Microsoft.AspNetCore.Authentication.AuthenticationBuilder> 。 如果应用需要其他作用域，请将它们添加到选项。 在以下示例中，添加了 Google `https://www.googleapis.com/auth/user.birthday.read` 范围以便检索用户的生日：

```csharp
options.Scope.Add("https://www.googleapis.com/auth/user.birthday.read");
```

## <a name="map-user-data-keys-and-create-claims"></a>映射用户数据密钥并创建声明

在提供程序的选项中，为 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonKey*> <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.MapJsonSubKey*> 外部提供程序的 JSON 用户数据中的每个键/子项指定或，以便在登录时读取应用程序标识。 有关声明类型的详细信息，请参阅 <xref:System.Security.Claims.ClaimTypes> 。

该示例应用程序 (`urn:google:locale`) 和图片 (`urn:google:picture` 从 `locale` `picture` Google 用户数据中的和密钥) 声明创建区域设置：

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=13-14)]

在中 `Microsoft.AspNetCore.Identity.UI.Pages.Account.Internal.ExternalLoginModel.OnPostConfirmationAsync` ， <xref:Microsoft.AspNetCore.Identity.IdentityUser> (`ApplicationUser`) 使用登录到应用 <xref:Microsoft.AspNetCore.Identity.SignInManager%601.SignInAsync*> 。 在登录过程中， <xref:Microsoft.AspNetCore.Identity.UserManager%601> 可以存储 `ApplicationUser` 提供的用户数据的声明 <xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.Principal*> 。

在示例应用中， `OnPostConfirmationAsync` ( *帐户/ExternalLogin* ) 将 (的 `urn:google:locale`) 和图片 (`urn:google:picture`) 声明中建立区域设置 `ApplicationUser` ，包括的声明 <xref:System.Security.Claims.ClaimTypes.GivenName> ：

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=35-51)]

默认情况下，会在身份验证中存储用户的声明 cookie 。 如果身份验证 cookie 太大，则可能会导致应用失败，原因是：

* 浏览器检测到 cookie 标头太长。
* 请求的整体大小太大。

如果需要大量用户数据来处理用户请求：

* 将请求处理的用户声明的数量和大小限制为仅应用需要的内容。
* 使用 <xref:Microsoft.AspNetCore.Authentication.Cookies.ITicketStore> 用于 Cookie 身份验证中间件的自定义 <xref:Microsoft.AspNetCore.Authentication.Cookies.CookieAuthenticationOptions.SessionStore> 来跨请求存储标识。 在服务器上保留大量标识信息，同时仅向客户端发送一个小型会话标识符密钥。

## <a name="save-the-access-token"></a>保存访问令牌

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.SaveTokens*> 定义在授权成功后是否应将访问和刷新令牌存储在中 <xref:Microsoft.AspNetCore.Http.Authentication.AuthenticationProperties> 。 `SaveTokens` 默认情况下，设置为以 `false` 减小最终身份验证的大小 cookie 。

示例应用将的值设置 `SaveTokens` 为 `true` 中的 <xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions> ：

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=15)]

`OnPostConfirmationAsync`执行时，将访问令牌存储在的外部提供程序中， ([ExternalLoginInfo. AuthenticationTokens](xref:Microsoft.AspNetCore.Identity.ExternalLoginInfo.AuthenticationTokens*)) 。 `ApplicationUser` `AuthenticationProperties`

示例应用程序将访问令牌保存在 `OnPostConfirmationAsync` (新用户注册) 并 `OnGetCallbackAsync` (之前注册的用户) *帐户/ExternalLogin 中。*

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Areas/Identity/Pages/Account/ExternalLogin.cshtml.cs?name=snippet_OnPostConfirmationAsync&highlight=54-56)]

## <a name="how-to-add-additional-custom-tokens"></a>如何添加其他自定义令牌

为了演示如何添加作为的一部分存储的自定义令牌， `SaveTokens` 示例应用程序会 <xref:Microsoft.AspNetCore.Authentication.AuthenticationToken> <xref:System.DateTime> 为的 [AuthenticationToken.Name](xref:Microsoft.AspNetCore.Authentication.AuthenticationToken.Name*) 添加一个，其中包含 `TicketCreated` ：

[!code-csharp[](additional-claims/samples/2.x/ClaimsSample/Startup.cs?name=snippet_AddGoogle&highlight=17-30)]

## <a name="creating-and-adding-claims"></a>创建和添加声明

框架提供用于创建声明并将其添加到集合的常见操作和扩展方法。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions> 和 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionUniqueExtensions>。

用户可以通过从派生 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction> 并实现抽象方法来定义自定义操作 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.Run*> 。

有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims>。

## <a name="removal-of-claim-actions-and-claims"></a>删除声明操作和声明

[ClaimActionCollection (字符串) ](xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimActionCollection.Remove*) 将从集合中移除给定的所有声明操作 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> 。 [ClaimActionCollectionMapExtensions. DeleteClaim (ClaimActionCollection，String) ](xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*) 从标识中删除给定的声明 <xref:Microsoft.AspNetCore.Authentication.OAuth.Claims.ClaimAction.ClaimType> 。 <xref:Microsoft.AspNetCore.Authentication.ClaimActionCollectionMapExtensions.DeleteClaim*> 主要与 [OpenID connect (OIDC) ](/azure/active-directory/develop/v2-protocols-oidc) 一起使用，以删除协议生成的声明。

## <a name="sample-app-output"></a>示例应用程序输出

```
User Claims

http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
    9b342344f-7aab-43c2-1ac1-ba75912ca999
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
    someone@gmail.com
AspNet.Identity.SecurityStamp
    7D4312MOWRYYBFI1KXRPHGOSTBVWSFDE
http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
    Judy
urn:google:locale
    en
urn:google:picture
    https://lh4.googleusercontent.com/-XXXXXX/XXXXXX/XXXXXX/XXXXXX/photo.jpg

Authentication Properties

.Token.access_token
    yc23.AlvoZqz56...1lxltXV7D-ZWP9
.Token.token_type
    Bearer
.Token.expires_at
    2019-04-11T22:14:51.0000000+00:00
.Token.TicketCreated
    4/11/2019 9:14:52 PM
.TokenNames
    access_token;token_type;expires_at;TicketCreated
.persistent
.issued
    Thu, 11 Apr 2019 20:51:06 GMT
.expires
    Thu, 25 Apr 2019 20:51:06 GMT

```

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

::: moniker-end

## <a name="additional-resources"></a>其他资源

* [dotnet/AspNetCore 工程 SocialSample 应用](https://github.com/dotnet/AspNetCore/tree/master/src/Security/Authentication/samples/SocialSample)：链接的示例应用位于 [Dotnet/AspNetCore GitHub 存储库的](https://github.com/dotnet/AspNetCore) `master` 工程分支中。 `master`分支包含处于活动开发下的下一版本 ASP.NET Core 的代码。 若要查看 ASP.NET Core 的已发布版本的示例应用的版本，请使用 **分支** 下拉列表选择发布分支 (例如 `release/{X.Y}`) "。
