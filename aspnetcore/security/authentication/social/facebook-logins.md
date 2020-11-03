---
title: ASP.NET Core 中的 Facebook 外部登录设置
author: rick-anderson
description: 包含代码示例的教程演示如何将 Facebook 帐户用户身份验证集成到现有 ASP.NET Core 应用。
ms.author: riande
ms.custom: seoapril2019, mvc, seodec18
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
uid: security/authentication/facebook-logins
ms.openlocfilehash: be0b655645fd2bd0eab9f9c30a65485f386cead3
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053353"
---
# <a name="facebook-external-login-setup-in-aspnet-core"></a>ASP.NET Core 中的 Facebook 外部登录设置

作者：[Valeriy Novytskyy](https://github.com/01binary) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

<!-- per @rick-anderson and scott addie, don't update images. Remove images and point the customer to the FB set up page. FB needs to maintain  instructions to get key and secret.
-->

本教程中的代码示例演示如何使用户能够使用在 [前一页](xref:security/authentication/social/index)上创建的示例 ASP.NET Core 3.0 项目登录其 Facebook 帐户。 首先，我们要按照 [官方步骤](https://developers.facebook.com)创建 FACEBOOK 应用 ID。

## <a name="create-the-app-in-facebook"></a>在 Facebook 中创建应用

* 将 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Facebook) NuGet 包添加到项目。

* 导航到 [Facebook 开发人员应用](https://developers.facebook.com/apps/) 页面并登录。 如果还没有 Facebook 帐户，请使用登录页上的 " **注册 Facebook** " 链接创建一个。  获得 Facebook 帐户后，请按照说明注册为 Facebook 开发人员。

* 从 " **我的应用** " 菜单中，选择 " **创建应用** " 以创建新的应用 ID。

   ![Facebook for 开发人员门户在 Microsoft Edge 中打开](index/_static/FBMyApps.png)

* 填写表单，然后点击 " **创建应用 ID** " 按钮。

  ![创建新的应用 ID 窗体](index/_static/FBNewAppId.png)

* 在 "新建应用" 卡中，选择 " **添加产品** "。  在 **Facebook 登录** 卡上，单击 " **设置** " 

  ![产品设置页](index/_static/FBProductSetup.png)

* **快速入门** 向导会启动，并 **选择一个平台** 作为第一页。 现在，通过单击左下方菜单中的 " **FaceBook 登录****设置** " 链接，绕过向导：

  ![跳过快速入门](index/_static/FBSkipQuickStart.png)

* 将显示 " **客户端 OAuth 设置** " 页：

  !["客户端 OAuth 设置" 页](index/_static/FBOAuthSetup.png)

* 输入附加到 " **有效的 OAuth 重定向 uri** " 字段中的 */SIGNIN-FACEBOOK* 的开发 URI (例如： `https://localhost:44320/signin-facebook`) 。 稍后在本教程中配置的 Facebook 身份验证将自动处理 */signin-facebook* 路由中的请求以实现 OAuth 流。

> [!NOTE]
> URI */signin-facebook* 设置为 facebook 身份验证提供程序的默认回调。 通过[FacebookOptions](/dotnet/api/microsoft.aspnetcore.authentication.facebook.facebookoptions)类的继承的[RemoteAuthenticationOptions. CallbackPath](/dotnet/api/microsoft.aspnetcore.authentication.remoteauthenticationoptions.callbackpath)属性配置 Facebook 身份验证中间件时，可以更改默认的回叫 URI。

* 单击 **“保存更改”** 。

* 单击 **Settings**  >  左侧导航栏中的 "设置" " **基本** " 链接。

  在此页上，请记下 `App ID` 和 `App Secret` 。 在下一部分中，你将同时添加到 ASP.NET Core 应用程序：

* 部署站点时，需要重新访问 **Facebook 登录** 设置页面并注册新的公共 URI。

## <a name="store-the-facebook-app-id-and-secret"></a>存储 Facebook 应用 ID 和机密

用 [机密管理器](xref:security/app-secrets)存储敏感设置，如 FACEBOOK 应用 ID 和机密值。 对于本示例，请使用以下步骤：

1. 按照 [启用密钥存储](xref:security/app-secrets#enable-secret-storage)中的说明初始化密钥存储的项目。
1. 将敏感设置存储在本地密钥存储中，并提供机密密钥 `Authentication:Facebook:AppId` 和 `Authentication:Facebook:AppSecret` ：

    ```dotnetcli
    dotnet user-secrets set "Authentication:Facebook:AppId" "<app-id>"
    dotnet user-secrets set "Authentication:Facebook:AppSecret" "<app-secret>"
    ```

[!INCLUDE[](~/includes/environmentVarableColon.md)]

## <a name="configure-facebook-authentication"></a>配置 Facebook 身份验证

将 Facebook 服务添加到 `ConfigureServices` *Startup.cs* 文件的方法中：

```csharp
services.AddAuthentication().AddFacebook(facebookOptions =>
{
    facebookOptions.AppId = Configuration["Authentication:Facebook:AppId"];
    facebookOptions.AppSecret = Configuration["Authentication:Facebook:AppSecret"];
});
```

[!INCLUDE [default settings configuration](includes/default-settings.md)]

## <a name="sign-in-with-facebook"></a>用 Facebook 登录

* 运行应用并选择 **"登录"** 。 
* 在 " **使用其他服务进行登录** " 下，选择 Facebook。
* 你将重定向到 **Facebook** 进行身份验证。
* 输入 Facebook 凭据。
* 你将重定向回到你的网站，你可以在其中设置电子邮件。

你现在已使用 Facebook 凭据登录：

<a name="react"></a>

## <a name="react-to-cancel-authorize-external-sign-in"></a>响应取消授权外部登录

<xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.AccessDeniedPath> 当用户未批准请求的授权请求时，可以提供用户代理的重定向路径。

下面的代码将设置 `AccessDeniedPath` 为 `"/AccessDeniedPathInfo"` ：

[!code-csharp[](~/security/authentication/social/social-code/StartupAccessDeniedPath.cs?name=snippetFB)]

建议 `AccessDeniedPath` 页面包含以下信息：

*  远程身份验证已取消。
* 此应用需要身份验证。
* 若要再次尝试登录，请选择 "登录" 链接。

### <a name="test-accessdeniedpath"></a>测试 AccessDeniedPath

* 导航到 [facebook.com](https://www.facebook.com/)
* 如果已登录，则必须注销。
* 运行应用并选择 "Facebook 登录"。
* 选择 " **暂时 Not** "。 你将重定向到指定的 `AccessDeniedPath` 页面。

<!-- End of React  -->
[!INCLUDE[Forward request information when behind a proxy or load balancer section](includes/forwarded-headers-middleware.md)]

[!INCLUDE[](includes/chain-auth-providers.md)]

有关 Facebook 身份验证支持的配置选项的详细信息，请参阅 [FacebookOptions](/dotnet/api/microsoft.aspnetcore.builder.facebookoptions) API 参考。 配置选项可用于：

* 请求有关用户的其他信息。
* 添加查询字符串参数以自定义登录体验。

## <a name="troubleshooting"></a>疑难解答

* **仅 ASP.NET Core 2.x：** 如果 Identity 未通过调用进行 `services.AddIdentity` 配置 `ConfigureServices` ，尝试进行身份验证会导致 *ArgumentException：必须提供 "SignInScheme" 选项* 。 本教程中使用的项目模板可确保完成此操作。
* 如果尚未通过应用初始迁移来创建站点数据库，则在处理请求错误时，将会出现 *数据库操作失败* 的情况。 点击 " **应用迁移** " 以创建数据库，然后单击 "刷新" 以继续出现错误。

## <a name="next-steps"></a>后续步骤

* 本文演示了如何通过 Facebook 进行身份验证。 您可以遵循类似的方法向 [前一页](xref:security/authentication/social/index)上列出的其他提供程序进行身份验证。

* 将网站发布到 Azure web 应用后，应 `AppSecret` 在 Facebook 开发人员门户中重置。

* `Authentication:Facebook:AppId` `Authentication:Facebook:AppSecret` 在 Azure 门户中将和设置为应用程序设置。 配置系统设置为从环境变量读取密钥。
