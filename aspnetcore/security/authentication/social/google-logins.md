---
title: ASP.NET Core 中的 Google 外部登录设置
author: rick-anderson
description: 本教程演示如何将 Google 帐户用户身份验证集成到现有 ASP.NET Core 应用。
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 03/19/2020
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
uid: security/authentication/google-logins
ms.openlocfilehash: 111ea7c972778dfd5296d0401c16563aeaa36a63
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060308"
---
# <a name="google-external-login-setup-in-aspnet-core"></a>ASP.NET Core 中的 Google 外部登录设置

作者：[Valeriy Novytskyy](https://github.com/01binary) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

本教程演示如何让用户能够使用在 [前一页](xref:security/authentication/social/index)上创建的 ASP.NET Core 3.0 项目使用其 Google 帐户登录。

## <a name="create-a-google-api-console-project-and-client-id"></a>创建 Google API 控制台项目和客户端 ID

* 请安装 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Google)。
* 导航到将 [Google Sign-In 集成到 web 应用中](https://developers.google.com/identity/sign-in/web/sign-in) ，然后选择 " **配置项目** "。
* 在 " **配置 OAuth 客户端** " 对话框中，选择 " **Web 服务器** "。
* 在 " **授权重定向 uri** " 文本输入框中，设置重定向 URI。 例如： `https://localhost:44312/signin-google`
* 保存 **客户端 ID** 和 **客户端密码** 。
* 部署站点时，从 **Google 控制台** 注册新的公共 url。

## <a name="store-the-google-client-id-and-secret"></a>存储 Google 客户端 ID 和机密

用 [机密管理器](xref:security/app-secrets)存储敏感设置，例如 Google 客户端 ID 和机密值。 对于本示例，请使用以下步骤：

1. 按照 [启用密钥存储](xref:security/app-secrets#enable-secret-storage)中的说明初始化密钥存储的项目。
1. 将敏感设置存储在本地密钥存储中，并提供机密密钥 `Authentication:Google:ClientId` 和 `Authentication:Google:ClientSecret` ：

    ```dotnetcli
    dotnet user-secrets set "Authentication:Google:ClientId" "<client-id>"
    dotnet user-secrets set "Authentication:Google:ClientSecret" "<client-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

你可以在 [Api 控制台](https://console.developers.google.com/apis/dashboard)中管理 api 凭据和使用情况。

## <a name="configure-google-authentication"></a>配置 Google 身份验证

将 Google 服务添加到 `Startup.ConfigureServices` ：

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupGoogle3x.cs?highlight=11-19)]

[!INCLUDE [default settings configuration](includes/default-settings2-2.md)]

## <a name="sign-in-with-google"></a>登录 Google

* 运行应用程序，并单击 " **登录"** 。 此时将显示使用 Google 登录的选项。
* 单击 " **google** " 按钮，该按钮将重定向到 google 进行身份验证。
* 输入 Google 凭据后，会重定向回网站。

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

<xref:Microsoft.AspNetCore.Authentication.Google.GoogleOptions>有关 Google 身份验证支持的配置选项的详细信息，请参阅 API 参考。 这可用于请求有关用户的其他信息。

## <a name="change-the-default-callback-uri"></a>更改默认回调 URI

URI 段 `/signin-google` 设置为 Google 身份验证提供程序的默认回调。 通过[GoogleOptions](/dotnet/api/microsoft.aspnetcore.authentication.google.googleoptions)类的继承的[RemoteAuthenticationOptions. CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath)属性配置 Google 身份验证中间件时，可以更改默认的回叫 URI。

## <a name="troubleshooting"></a>疑难解答

* 如果登录不起作用，并且没有出现任何错误，请切换到开发模式，以便更轻松地进行调试。
* 如果 Identity 未通过调用 `services.AddIdentity` 进行配置 `ConfigureServices` ，则尝试在 ArgumentException 中对结果进行身份验证 *：必须提供 "SignInScheme" 选项* 。 本教程中使用的项目模板可确保完成此操作。
* 如果尚未通过应用初始迁移来创建站点数据库，则在处理请求错误时，将会出现 *数据库操作失败* 的情况。 选择 " **应用迁移** " 以创建数据库，并刷新页面以继续出现错误。

## <a name="next-steps"></a>后续步骤

* 本文演示了如何通过 Google 进行身份验证。 您可以遵循类似的方法向 [前一页](xref:security/authentication/social/index)上列出的其他提供程序进行身份验证。
* 将应用发布到 Azure 后，请 `ClientSecret` 在 GOOGLE API 控制台中重置。
* `Authentication:Google:ClientId` `Authentication:Google:ClientSecret` 在 Azure 门户中将和设置为应用程序设置。 配置系统设置为从环境变量读取密钥。
