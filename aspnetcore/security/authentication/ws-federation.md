---
title: 通过 ASP.NET Core 中的 WS-Federation 对用户进行身份验证
author: chlowell
description: 本教程演示如何在 ASP.NET Core 应用程序中使用 WS-Federation。
ms.author: scaddie
ms.custom: mvc
ms.date: 01/16/2019
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
uid: security/authentication/ws-federation
ms.openlocfilehash: ed78923a2bdd1ed683a72c0a6f34337a38350035
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053366"
---
# <a name="authenticate-users-with-ws-federation-in-aspnet-core"></a>通过 ASP.NET Core 中的 WS-Federation 对用户进行身份验证

本教程演示如何使用户能够使用 WS-Federation 身份验证提供程序（例如 Active Directory 联合身份验证服务 (ADFS) 或 [Azure Active Directory](/azure/active-directory/) (AAD) ）登录。 它使用 [Facebook、Google 和 external 提供程序身份验证](xref:security/authentication/social/index)中介绍的 ASP.NET Core 示例应用。

对于 ASP.NET Core 应用，WS-Federation 支持由 [WsFederation](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.WsFederation)提供。 此组件从 [Owin](https://www.nuget.org/packages/Microsoft.Owin.Security.WsFederation) 移植，并共享该组件的许多机制。 不过，这些组件在一些重要的方面有所不同。

默认情况下，新的中间件：

* 不允许未经请求的登录。 WS-Federation 协议的此功能容易受到 XSRF 攻击。 但是，可以通过 `AllowUnsolicitedLogins` 选项启用。
* 不会检查每个窗体张贴内容中的登录消息。 仅检查对的请求 `CallbackPath` 。 `CallbackPath` 默认为， `/signin-wsfed` 但可以通过[WsFederationOptions](/dotnet/api/microsoft.aspnetcore.authentication.wsfederation.wsfederationoptions)类的继承的[RemoteAuthenticationOptions. CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath)属性更改为。 通过启用 [SkipUnrecognizedRequests](/dotnet/api/microsoft.aspnetcore.authentication.wsfederation.wsfederationoptions.skipunrecognizedrequests) 选项，可以将此路径与其他身份验证提供程序共享。

## <a name="register-the-app-with-active-directory"></a>将应用注册到 Active Directory

### <a name="active-directory-federation-services"></a>Active Directory 联合身份验证服务

* 从 ADFS 管理控制台打开服务器的 " **添加信赖方信任向导** "：

![添加信赖方信任向导：欢迎](ws-federation/_static/AdfsAddTrust.png)

* 选择手动输入数据：

![添加信赖方信任向导：选择数据源](ws-federation/_static/AdfsSelectDataSource.png)

* 为信赖方输入显示名称。 该名称对 ASP.NET Core 应用并不重要。

* [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.WsFederation) 不支持令牌加密，因此请不要配置令牌加密证书：

![添加信赖方信任向导：配置证书](ws-federation/_static/AdfsConfigureCert.png)

* 使用应用的 URL 启用对 WS-Federation 被动协议的支持。 验证该应用的端口是否正确：

![添加信赖方信任向导：配置 URL](ws-federation/_static/AdfsConfigureUrl.png)

> [!NOTE]
> 这必须是 HTTPS URL。 IIS Express 可以在开发过程中托管应用时提供自签名证书。 Kestrel 需要手动配置证书。 有关更多详细信息，请参阅 [Kestrel 文档](xref:fundamentals/servers/kestrel) 。

* 在向导的其余部分中单击 " **下一步** "，然后 **关闭** 。

* ASP.NET Core Identity 需要 **名称 ID** 声明。 从 " **编辑声明规则** " 对话框中添加一个：

![编辑声明规则](ws-federation/_static/EditClaimRules.png)

* 在 " **添加转换声明规则向导** " 中，保留 "默认 **发送 LDAP 属性作为声明** " 模板，然后单击 " **下一步** "。 添加一个规则，将 **SAM 帐户名称** LDAP 属性映射到 **名称 ID** 传出声明：

![添加转换声明规则向导：配置声明规则](ws-federation/_static/AddTransformClaimRule.png)

* 单击 **Finish**  >  " **编辑声明规则** " 窗口中的 **"完成"** 。

### <a name="azure-active-directory"></a>Azure Active Directory

* 导航到 AAD 租户的 "应用注册" 边栏选项卡。 单击 " **新应用程序注册** "：

![Azure Active Directory：应用注册](ws-federation/_static/AadNewAppRegistration.png)

* 输入应用注册的名称。 这对于 ASP.NET Core 应用并不重要。
* 输入应用作为 **登录 url** 侦听的 url：

![Azure Active Directory：创建应用注册](ws-federation/_static/AadCreateAppRegistration.png)

* 单击 " **终结点** " 并记下 " **联合元数据文档** " URL。 这是 WS-Federation 中间件的 `MetadataAddress` ：

![Azure Active Directory：终结点](ws-federation/_static/AadFederationMetadataDocument.png)

* 导航到新的应用注册。 单击 " **公开 API** "。 单击 "应用程序 ID URI **集**  >  **保存** "。 记下  **应用程序 ID URI** 。 这是 WS-Federation 中间件的 `Wtrealm` ：

![Azure Active Directory：应用注册属性](ws-federation/_static/AadAppIdUri.png)

## <a name="use-ws-federation-without-no-locaspnet-core-identity"></a>使用 WS-Federation 而不 ASP.NET Core Identity

WS-Federation 中间件可以在不使用的情况下使用 Identity 。 例如： 。
::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](ws-federation/samples/StartupNon31.cs?name=snippet)]
::: moniker-end

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"
[!code-csharp[](ws-federation/samples/StartupNon21.cs?name=snippet)]
::: moniker-end

## <a name="add-ws-federation-as-an-external-login-provider-for-no-locaspnet-core-identity"></a>添加 WS-Federation 作为的外部登录提供程序 ASP.NET Core Identity

* 将 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.WsFederation) 上的依赖项添加到项目。
* 将 WS-Federation 添加到 `Startup.ConfigureServices` ：

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](ws-federation/samples/Startup31.cs?name=snippet)]
::: moniker-end

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"
[!code-csharp[](ws-federation/samples/Startup21.cs?name=snippet)]
::: moniker-end

[!INCLUDE [default settings configuration](social/includes/default-settings.md)]

### <a name="log-in-with-ws-federation"></a>用 WS-Federation 登录

浏览到应用，并单击导航头中的 " **登录** " 链接。 有一个选项可以使用 WsFederation： ![ 登录页登录](ws-federation/_static/WsFederationButton.png)

使用 ADFS 作为提供程序时，按钮会重定向到 ADFS 登录页： ![ adfs 登录页](ws-federation/_static/AdfsLoginPage.png)

使用 Azure Active Directory 作为提供程序时，该按钮将重定向到 AAD 登录页： ![ aad 登录页](ws-federation/_static/AadSignIn.png)

成功登录新用户重定向到应用的用户注册页面： ![ 注册页面](ws-federation/_static/Register.png)
