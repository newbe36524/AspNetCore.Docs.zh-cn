---
title: 配置 ASP.NET Core Identity
author: AdrienTorris
description: 了解 ASP.NET Core Identity 默认值并了解如何配置 Identity 属性以使用自定义值。
ms.author: riande
ms.custom: mvc
ms.date: 02/11/2019
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
uid: security/authentication/identity-configuration
ms.openlocfilehash: b11a2d584b7275a9065c9915021ac945823531f8
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051988"
---
# <a name="configure-no-locaspnet-core-identity"></a>配置 ASP.NET Core Identity

ASP.NET Core Identity 使用密码策略、锁定和配置等设置的默认值 cookie 。 可以在类中重写这些设置 `Startup` 。

## <a name="no-locidentity-options"></a>Identity 选项

[ Identity Options](/dotnet/api/microsoft.aspnetcore.identity.identityoptions)类表示可用于配置系统的选项 Identity 。 `IdentityOptions` 必须 **在** 调用或之后 `AddIdentity` 设置 `AddDefaultIdentity` 。

### <a name="claims-no-locidentity"></a>支付 Identity

[ Identity Options。声明 Identity ](/dotnet/api/microsoft.aspnetcore.identity.identityoptions.claimsidentity)指定[声明 Identity 选项](/dotnet/api/microsoft.aspnetcore.identity.claimsidentityoptions)，其中包含下表中所示的属性。

| 属性 | 说明 | 默认 |
| -------- | ----------- | :-----: |
| [RoleClaimType](/dotnet/api/microsoft.aspnetcore.identity.claimsidentityoptions.roleclaimtype) | 获取或设置用于角色声明的声明类型。 | [ClaimTypes](/dotnet/api/system.security.claims.claimtypes.role) |
| [SecurityStampClaimType](/dotnet/api/microsoft.aspnetcore.identity.claimsidentityoptions.securitystampclaimtype) | 获取或设置用于安全戳声明的声明类型。 | `AspNet.Identity.SecurityStamp` |
| [UserIdClaimType](/dotnet/api/microsoft.aspnetcore.identity.claimsidentityoptions.useridclaimtype) | 获取或设置用于用户标识符声明的声明类型。 | [ClaimTypes. NameIdentifier](/dotnet/api/system.security.claims.claimtypes.nameidentifier) |
| [UserNameClaimType](/dotnet/api/microsoft.aspnetcore.identity.claimsidentityoptions.usernameclaimtype) | 获取或设置用于用户名声明的声明类型。 | [ClaimTypes.Name](/dotnet/api/system.security.claims.claimtypes.name) |

### <a name="lockout"></a>锁定

在 [PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync#Microsoft_AspNetCore_Identity_SignInManager_1_PasswordSignInAsync_System_String_System_String_System_Boolean_System_Boolean_) 方法中设置锁定：

[!code-csharp[](identity-configuration/sample/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=9)]

前面的代码基于 `Login` Identity 模板。 

锁定选项设置为 `StartUp.ConfigureServices` ：

[!code-csharp[](identity-configuration/sample/Startup.cs?name=snippet_lock)]

前面的代码将[ Identity 选项](/dotnet/api/microsoft.aspnetcore.identity.identityoptions) [LockoutOptions](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions)设置为默认值。

身份验证成功后，将重置失败的访问尝试计数并重置时钟。

[ Identity 选项。锁定](/dotnet/api/microsoft.aspnetcore.identity.identityoptions.lockout)将指定[LockoutOptions](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions) ，其中包含表中所示的属性。

| 属性 | 说明 | 默认 |
| -------- | ----------- | :-----: |
| [AllowedForNewUsers](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.allowedfornewusers) | 确定新用户是否可以锁定。 | `true` |
| [DefaultLockoutTimeSpan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan) | 锁定发生时用户被锁定的时间长度。 | 5 分钟 |
| [MaxFailedAccessAttempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts) | 如果启用了锁定，则在用户被锁定之前失败的访问尝试次数。 | 5 |

### <a name="password"></a>密码

默认情况下， Identity 要求密码包含大写字符、小写字符、数字以及非字母数字字符。 密码长度必须至少为6个字符。

密码配置为：

* `Startup.ConfigureServices` 中的 <xref:Microsoft.AspNetCore.Identity.PasswordOptions>。
* [ `[StringLength]` ](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute) `Password` 如果 Identity 将[基架到应用程序中](xref:security/authentication/scaffold-identity)，则为属性的属性。 `InputModel``Password`在以下文件中可以找到属性：
  * `Areas/Identity/Pages/Account/Register.cshtml.cs`
  * `Areas/Identity/Pages/Account/ResetPassword.cshtml.cs`

[!code-csharp[](identity-configuration/sample/Startup.cs?name=snippet_pw)]

[ Identity Options. Password](/dotnet/api/microsoft.aspnetcore.identity.identityoptions.password)指定[PasswordOptions](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions) ，其中包含表中所示的属性。

| 属性 | 说明 | 默认 |
| -------- | ----------- | :-----: |
| [RequireDigit](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions.requiredigit) | 要求密码中的数字介于0-9 之间。 | `true` |
| [RequiredLength](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions.requiredlength) | 密码的最小长度。 | 6 |
| [RequireLowercase](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions.requirelowercase) | 密码中需要小写字符。 | `true` |
| [RequireNonAlphanumeric](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions.requirenonalphanumeric) | 密码中需要一个非字母数字字符。 | `true` |
| [RequiredUniqueChars](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions.requireduniquechars) | 仅适用于 ASP.NET Core 2.0 或更高版本。<br><br> 需要密码中的非重复字符数。 | 1 |
| [RequireUppercase](/dotnet/api/microsoft.aspnetcore.identity.passwordoptions.requireuppercase) | 密码中需要大写字符。 | `true` |

### <a name="sign-in"></a>登录

下面的代码将 `SignIn` 设置 (设置为默认值) ：

[!code-csharp[](identity-configuration/sample/Startup.cs?name=snippet_si)]

" [ Identity 登录](/dotnet/api/microsoft.aspnetcore.identity.identityoptions.signin)" 指定[SignInOptions](/dotnet/api/microsoft.aspnetcore.identity.signinoptions) ，其中包含表中所示的属性。

| 属性 | 说明 | 默认 |
| -------- | ----------- | :-----: |
| [RequireConfirmedEmail](/dotnet/api/microsoft.aspnetcore.identity.signinoptions.requireconfirmedemail) | 需要确认电子邮件登录。 | `false` |
| [RequireConfirmedPhoneNumber](/dotnet/api/microsoft.aspnetcore.identity.signinoptions.requireconfirmedphonenumber) | 需要确认电话号码才能登录。 | `false` |

### <a name="tokens"></a>令牌

[ Identity 选项。标记](/dotnet/api/microsoft.aspnetcore.identity.identityoptions.tokens)指定[TokenOptions](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions) ，其中包含表中所示的属性。

| 属性 | 说明 |
| -------- | ----------- |
| [AuthenticatorTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions.authenticatortokenprovider) | 获取或设置 `AuthenticatorTokenProvider` 用于使用验证器验证双重登录的。 |
| [ChangeEmailTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions.changeemailtokenprovider) | 获取或设置 `ChangeEmailTokenProvider` 用于生成电子邮件更改确认电子邮件中使用的令牌的。 |
| [ChangePhoneNumberTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions.changephonenumbertokenprovider) | 获取或设置 `ChangePhoneNumberTokenProvider` 用于生成更改电话号码时使用的令牌的。 |
| [EmailConfirmationTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions.emailconfirmationtokenprovider) | 获取或设置用于生成帐户确认电子邮件中使用的令牌的令牌提供程序。 |
| [PasswordResetTokenProvider](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions.passwordresettokenprovider) | 获取或设置用于生成密码重置电子邮件中使用的令牌的[IUserTwoFactorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.iusertwofactortokenprovider-1) 。 |
| [ProviderMap](/dotnet/api/microsoft.aspnetcore.identity.tokenoptions.providermap) | 用于使用用作提供程序名称的密钥构造 [用户令牌提供程序](/dotnet/api/microsoft.aspnetcore.identity.tokenproviderdescriptor) 。 |

### <a name="user"></a>用户

[!code-csharp[](identity-configuration/sample/Startup.cs?name=snippet_user)]

[ Identity Options。 User](/dotnet/api/microsoft.aspnetcore.identity.identityoptions.user)指定[UserOptions](/dotnet/api/microsoft.aspnetcore.identity.useroptions) ，其中包含表中所示的属性。

| 属性 | 说明 | 默认 |
| -------- | ----------- | :-----: |
| [AllowedUserNameCharacters](/dotnet/api/microsoft.aspnetcore.identity.useroptions.allowedusernamecharacters) | 用户名中允许使用的字符。 | abcdefghijklmnopqrstuvwxyz<br>ABCDEFGHIJKLMNOPQRSTUVWXYZ<br>0123456789<br>-.\_@+ |
| [RequireUniqueEmail](/dotnet/api/microsoft.aspnetcore.identity.useroptions.requireuniqueemail) | 要求每个用户都有唯一的电子邮件。 | `false` |

### <a name="no-loccookie-settings"></a>Cookie 设置

在中配置应用 cookie 程序 `Startup.ConfigureServices` 。 [ConfigureApplication Cookie](/dotnet/api/microsoft.extensions.dependencyinjection.identityservicecollectionextensions.configureapplicationcookie#Microsoft_Extensions_DependencyInjection_IdentityServiceCollectionExtensions_ConfigureApplicationCookie_Microsoft_Extensions_DependencyInjection_IServiceCollection_System_Action_Microsoft_AspNetCore_Authentication_Cookies_CookieAuthenticationOptions__)调用或 **后** 必须调用 `AddIdentity` `AddDefaultIdentity` 。

[!code-csharp[](identity-configuration/sample/Startup.cs?name=snippet_cookie)]

有关详细信息，请参阅[ Cookie AuthenticationOptions](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions)。

## <a name="password-hasher-options"></a>Password Hasher 选项

<xref:Microsoft.AspNetCore.Identity.PasswordHasherOptions> 获取和设置用于密码哈希的选项。

| 选项 | 说明 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Identity.PasswordHasherOptions.CompatibilityMode> | 对新密码进行哈希处理时使用的兼容性模式。 默认为 <xref:Microsoft.AspNetCore.Identity.PasswordHasherCompatibilityMode.IdentityV3>。 哈希密码的第一个字节称为 *格式标记* ，它指定用于对密码进行哈希处理的哈希算法的版本。 针对哈希验证密码时，该方法会根据 <xref:Microsoft.AspNetCore.Identity.PasswordHasher`1.VerifyHashedPassword*> 第一个字节选择正确的算法。 无论使用哪个版本的算法对密码进行哈希处理，客户端都可以进行身份验证。 设置兼容性模式会影响 *新密码* 的哈希。 |
| <xref:Microsoft.AspNetCore.Identity.PasswordHasherOptions.IterationCount> | 使用 PBKDF2 对密码进行哈希处理时使用的迭代次数。 仅当设置为时，才使用此值 <xref:Microsoft.AspNetCore.Identity.PasswordHasherOptions.CompatibilityMode> <xref:Microsoft.AspNetCore.Identity.PasswordHasherCompatibilityMode.IdentityV3> 。 该值必须是正整数并且默认值为 `10000` 。 |

在下面的示例中，将 <xref:Microsoft.AspNetCore.Identity.PasswordHasherOptions.IterationCount> 设置为 `12000` in `Startup.ConfigureServices` ：

```csharp
// using Microsoft.AspNetCore.Identity;

services.Configure<PasswordHasherOptions>(option =>
{
    option.IterationCount = 12000;
});
```

## <a name="globally-require-all-users-to-be-authenticated"></a>全局要求对所有用户进行身份验证

[!INCLUDE[](~/includes/requireAuth.md)]