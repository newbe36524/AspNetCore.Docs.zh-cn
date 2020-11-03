---
title: 具有 ASP.NET Core 的 Twitter 外部登录设置
author: rick-anderson
description: 本教程演示如何将 Twitter 帐户用户身份验证集成到现有 ASP.NET Core 应用。
ms.author: riande
ms.custom: mvc
ms.date: 03/19/2020
monikerRange: '>= aspnetcore-3.0'
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
uid: security/authentication/twitter-logins
ms.openlocfilehash: 47926d12ac5f922f2937df164d38ff6eb63cacf1
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053275"
---
# <a name="twitter-external-sign-in-setup-with-aspnet-core"></a>具有 ASP.NET Core 的 Twitter 外部登录设置

作者：[Valeriy Novytskyy](https://github.com/01binary) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

此示例演示如何使用户能够使用在[前一页](xref:security/authentication/social/index)上创建的示例 ASP.NET Core 3.0 项目[登录其 Twitter 帐户](https://dev.twitter.com/web/sign-in/desktop-browser)。

## <a name="create-the-app-in-twitter"></a>在 Twitter 中创建应用

* 将 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Twitter/3.0.0) NuGet 包添加到项目。

* 导航到 [https://apps.twitter.com/](https://apps.twitter.com/) 并登录。 如果还没有 Twitter 帐户，请使用 " **[立即注册](https://twitter.com/signup)** " 链接创建一个。

* 选择 " **创建应用** "。 填写 "应用 **名称** "、" **应用程序说明** " 和 "公共 **网站** URI" (这在您注册域名) 之前是暂时的：

* 选中 " **启用使用 Twitter 登录** " 旁边的框

* AspNetCore。Identity 要求用户在默认情况下具有电子邮件地址。 中转到 " **权限** " 选项卡，单击 " **编辑** " 按钮，然后选中 " **请求用户的电子邮件地址** " 旁边的框。

* 输入 `/signin-twitter` 附加到 " **回调 url** " 字段中的开发 URI (例如： `https://webapp128.azurewebsites.net/signin-twitter`) 。 稍后在本示例中配置的 Twitter 身份验证方案将自动处理路由中的请求 `/signin-twitter` 以实现 OAuth 流。

  > [!NOTE]
  > URI 段 `/signin-twitter` 设置为 Twitter 身份验证提供程序的默认回调。 通过[TwitterOptions](/dotnet/api/microsoft.aspnetcore.authentication.twitter.twitteroptions)类的继承的[RemoteAuthenticationOptions. CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath)属性配置 Twitter 身份验证中间件时，可以更改默认的回叫 URI。

* 填写表单的其余部分，然后选择 " **创建** "。 显示新应用程序详细信息：

## <a name="store-the-twitter-consumer-api-key-and-secret"></a>存储 Twitter 使用者 API 密钥和机密

使用 [机密管理器](xref:security/app-secrets)存储敏感设置，如 Twitter 使用者 API 密钥和机密。 对于本示例，请使用以下步骤：

1. 按照 [启用密钥存储](xref:security/app-secrets#enable-secret-storage)中的说明初始化密钥存储的项目。
1. 将敏感设置存储在本地密钥存储中，并提供机密密钥 `Authentication:Twitter:ConsumerKey` 和 `Authentication:Twitter:ConsumerSecret` ：

    ```dotnetcli
    dotnet user-secrets set "Authentication:Twitter:ConsumerAPIKey" "<consumer-api-key>"
    dotnet user-secrets set "Authentication:Twitter:ConsumerSecret" "<consumer-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

创建新的 Twitter 应用程序后，可以在 " **密钥和访问令牌** " 选项卡上找到这些令牌：

## <a name="configure-twitter-authentication"></a>配置 Twitter 身份验证

将 Twitter 服务添加到 `ConfigureServices` *Startup.cs* 文件的方法中：

[!code-csharp[](~/security/authentication/social/social-code/3.x/StartupTwitter3x.cs?name=snippet&highlight=10-15)]

[!INCLUDE [default settings configuration](includes/default-settings.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

有关 Twitter 身份验证支持的配置选项的详细信息，请参阅 [TwitterOptions](/dotnet/api/microsoft.aspnetcore.builder.twitteroptions) API 参考。 这可用于请求有关用户的其他信息。

## <a name="sign-in-with-twitter"></a>通过 Twitter 登录

运行应用并选择 **"登录"** 。 将显示使用 Twitter 登录的选项：

单击 **twitter** 重定向到 twitter 进行身份验证：

输入 Twitter 凭据后，会重定向回到网站，你可以在其中设置电子邮件。

你现在已使用 Twitter 凭据登录：

[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

<!-- 
### React to cancel Authorize External sign-in
Twitter doesn't support AccessDeniedPath
Rather in the twitter setup, you can provide an External sign-in homepage. The external sign-in homepage doesn't support localhost. Tested with https://cors3.azurewebsites.net/ and that works.
-->

## <a name="troubleshooting"></a>疑难解答

* **仅 ASP.NET Core 2.x：** 如果 Identity 未通过调用进行 `services.AddIdentity` 配置 `ConfigureServices` ，尝试进行身份验证会导致 *ArgumentException：必须提供 "SignInScheme" 选项* 。 本示例中使用的项目模板可确保完成此操作。
* 如果尚未通过应用初始迁移来创建站点数据库，则在处理请求错误时将会获得 *数据库操作失败* 。 点击 " **应用迁移** " 以创建数据库，然后单击 "刷新" 以继续出现错误。

## <a name="next-steps"></a>后续步骤

* 本文介绍了如何通过 Twitter 进行身份验证。 您可以遵循类似的方法向 [前一页](xref:security/authentication/social/index)上列出的其他提供程序进行身份验证。

* 将网站发布到 Azure web 应用后，应 `ConsumerSecret` 在 Twitter 开发人员门户中重置。

* `Authentication:Twitter:ConsumerKey` `Authentication:Twitter:ConsumerSecret` 在 Azure 门户中将和设置为应用程序设置。 配置系统设置为从环境变量读取密钥。
