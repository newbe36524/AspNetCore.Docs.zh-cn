---
title: ASP.NET Core 中的 Azure Active Directory B2C 的云身份验证
author: camsoper
description: 了解如何设置 ASP.NET Core Azure Active Directory B2C 身份验证。
ms.author: casoper
ms.custom: devx-track-csharp, mvc
ms.date: 01/21/2019
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
uid: security/authentication/azure-ad-b2c
ms.openlocfilehash: f917bec8f2d929e62bf43494159a63458f135c5f
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061387"
---
# <a name="cloud-authentication-with-azure-active-directory-b2c-in-aspnet-core"></a>ASP.NET Core 中的 Azure Active Directory B2C 的云身份验证

作者：[Cam Soper](https://twitter.com/camsoper)

[Azure Active Directory B2C](/azure/active-directory-b2c/active-directory-b2c-overview) (Azure AD B2C) 是用于 web 和移动应用的云标识管理解决方案。 此服务为云中和本地托管的应用提供身份验证。 身份验证类型包括个人帐户、社交网络帐户和联合企业帐户。 此外，Azure AD B2C 可以通过最少的配置来提供多重身份验证。

> [!TIP]
> Azure Active Directory (Azure AD) 和 Azure AD B2C 是单独的产品产品。 Azure AD 租户表示组织，而 Azure AD B2C 租户表示与信赖方应用程序一起使用的标识集合。 若要了解详细信息，请参阅 [Azure AD B2C：常见问题解答)  (](/azure/active-directory-b2c/active-directory-b2c-faqs)常见问题。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 创建 Azure Active Directory B2C 租户
> * 在 Azure AD B2C 中注册应用
> * 使用 Visual Studio 创建一个 ASP.NET Core web 应用，该应用配置为使用 Azure AD B2C 租户进行身份验证
> * 配置控制 Azure AD B2C 租户的行为的策略

## <a name="prerequisites"></a>先决条件

本演练需要以下各项：

* [Microsoft Azure 订阅](https://azure.microsoft.com/free/dotnet/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio)
* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/?utm_medium=microsoft&utm_source=docs.microsoft.com&utm_campaign=inline+link&utm_content=download+vs2019)

## <a name="create-the-azure-active-directory-b2c-tenant"></a>创建 Azure Active Directory B2C 租户

[按照文档中的说明](/azure/active-directory-b2c/active-directory-b2c-get-started)创建 Azure Active Directory B2C 租户。 出现提示时，在本教程中，将租户与 Azure 订阅相关联是可选的。

## <a name="register-the-app-in-azure-ad-b2c"></a>在 Azure AD B2C 中注册应用程序

在新创建的 Azure AD B2C 租户中，使用 " **注册 web 应用** " 部分下的 [文档中的步骤](/azure/active-directory-b2c/tutorial-register-applications#register-a-web-application)注册应用。 请在 **创建 web 应用客户端机密** 部分停止。 本教程不需要客户端密码。 

使用以下值：

| 设置                       | 值                     | 注释                                                                                                                                                                                              |
|-------------------------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 名称                      | *&lt;应用名称&gt;*        | 输入向使用者描述你的应用程序的应用程序的 **名称** 。                                                                                                                                 |
|  包括 Web 应用/Web API | 是                       |                                                                                                                                                                                                    |
|  允许隐式流       | 是                       |                                                                                                                                                                                                    |
| 回复 URL                 | `https://localhost:44300/signin-oidc` | 回复 URL 属于终结点，允许 Azure AD B2C 在其中返回应用请求的任何令牌。 Visual Studio 提供要使用的回复 URL。 现在，请按 enter `https://localhost:44300/signin-oidc` 完成表单。 |
| **应用 ID URI**                | 留空               | 本教程不需要。                                                                                                                                                                    |
| **包含本机客户端**     | 否                        |                                                                                                                                                                                                    |

> [!WARNING]
> 如果设置非 localhost 回复 URL，请注意 ["回复 url" 列表中允许的内容的约束](/azure/active-directory-b2c/tutorial-register-applications#register-a-web-application)。 

注册应用后，会显示租户中的应用列表。 选择刚注册的应用程序。 选择 " **应用程序 ID** " 字段右侧的 " **复制** " 图标，将其复制到剪贴板。

目前不能在 Azure AD B2C 租户中配置其他内容，但会将浏览器窗口保持打开状态。 创建 ASP.NET Core 应用后，会进行更多配置。

## <a name="create-an-aspnet-core-app-in-visual-studio"></a>在 Visual Studio 中创建 ASP.NET Core 应用

可以将 Visual Studio Web 应用程序模板配置为使用 Azure AD B2C 租户进行身份验证。

在 Visual Studio 中：

1. 创建新的 ASP.NET Core Web 应用程序。 
2. 从模板列表中选择 " **Web 应用程序** "。
3. 选择 " **更改身份验证** " 按钮。
    
    !["更改身份验证" 按钮](./azure-ad-b2c/_static/changeauth.png)

4. 在 " **更改身份验证** " 对话框中，选择 " **个人用户帐户** "，然后在下拉列表中选择 " **连接到云中的现有用户存储** "。 
    
    !["更改身份验证" 对话框](./azure-ad-b2c/_static/changeauthdialog.png)

5. 完成具有以下值的窗体：
    
    | 设置                       | 值                                                 |
    |-------------------------------|-------------------------------------------------------|
    | **域名**               | *&lt;B2C 租户的域名&gt;*          |
    | **应用程序 ID**            | *&lt;从剪贴板粘贴应用程序 ID&gt;* |
    | **回调路径**             | *&lt;使用默认值&gt;*                       |
    | **注册或登录策略** | `B2C_1_SiUpIn`                                        |
    | **重置密码策略**     | `B2C_1_SSPR`                                          |
    | **编辑配置文件策略**       | *&lt;留空&gt;*                                 |
    
    选择 " **回复 uri** " 旁边的 " **复制** " 链接，以将答复 uri 复制到剪贴板。 选择 **"确定"** 以关闭 " **更改身份验证** " 对话框。 选择 **"确定"** 以创建 web 应用。

## <a name="finish-the-b2c-app-registration"></a>完成 B2C 应用注册

返回到包含 B2C 应用程序属性的浏览器窗口。 将前面指定的临时 **回复 URL** 更改为从 Visual Studio 复制的值。 选择窗口顶部的 " **保存** "。

> [!TIP]
> 如果未复制回复 URL，请使用 web 项目属性中的 "调试" 选项卡上的 HTTPS 地址，并从中追加 **CallbackPath** 值 *appsettings.json* 。

## <a name="configure-policies"></a>配置策略

使用 Azure AD B2C 文档中的步骤 [创建注册或登录策略](/azure/active-directory-b2c/active-directory-b2c-reference-policies#user-flow-versions)，然后 [创建密码重置策略](/azure/active-directory-b2c/active-directory-b2c-reference-policies#user-flow-versions)。 使用提供 **Identity 者** 文档、 **注册属性** 和 **应用程序声明** 中提供的示例值。 使用 " **立即运行** " 按钮测试文档中所述的策略是可选的。

> [!WARNING]
> 确保策略名称与文档中所述的完全相同，因为在 Visual Studio 的 " **更改身份验证** " 对话框中使用了这些策略。 可以在中验证策略名称 *appsettings.json* 。

## <a name="configure-the-underlying-openidconnectoptionsjwtbearerno-loccookie-options"></a>配置基础 OpenIdConnectOptions/JwtBearer/ Cookie options

若要直接配置基础选项，请在中使用适当的方案常数 `Startup.ConfigureServices` ：

```csharp
services.Configure<OpenIdConnectOptions>(
    AzureAD[B2C]Defaults.OpenIdScheme, options => 
    {
        // Omitted for brevity
    });

services.Configure<CookieAuthenticationOptions>(
    AzureAD[B2C]Defaults.CookieScheme, options => 
    {
        // Omitted for brevity
    });

services.Configure<JwtBearerOptions>(
    AzureAD[B2C]Defaults.JwtBearerAuthenticationScheme, options => 
    {
        // Omitted for brevity
    });
```

## <a name="run-the-app"></a>运行应用程序

在 Visual Studio 中，按 **F5** 生成并运行应用。 Web 应用启动后，选择 " **接受** " 以接受 cookie (如果系统提示) ，然后选择 " **登录** "。

![登录应用](./azure-ad-b2c/_static/signin.png)

浏览器重定向到 Azure AD B2C 租户。 使用现有帐户登录 (如果已创建测试策略) 或选择 " **立即注册** " 创建新帐户。 " **忘记了密码？** " 链接用于重置忘记的密码。

![Azure AD B2C 登录](./azure-ad-b2c/_static/b2csts.png)

成功登录后，浏览器将重定向到 web 应用。

![Success](./azure-ad-b2c/_static/success.png)

## <a name="next-steps"></a>后续步骤

在本教程中，你了解了如何执行以下操作：

> [!div class="checklist"]
> * 创建 Azure Active Directory B2C 租户
> * 在 Azure AD B2C 中注册应用
> * 使用 Visual Studio 创建一个 ASP.NET Core Web 应用程序，该应用程序配置为使用 Azure AD B2C 租户进行身份验证
> * 配置控制 Azure AD B2C 租户的行为的策略

现在，ASP.NET Core 应用配置为使用 Azure AD B2C 进行身份验证， [授权属性](xref:security/authorization/simple) 可用于保护你的应用。 通过学习以下内容继续开发应用：

* [自定义 Azure AD B2C 用户界面](/azure/active-directory-b2c/active-directory-b2c-reference-ui-customization)。
* [配置密码复杂性要求](/azure/active-directory-b2c/active-directory-b2c-reference-password-complexity)。
* [启用多重身份验证](/azure/active-directory-b2c/active-directory-b2c-reference-mfa)。
* 配置其他标识提供程序，例如 [Microsoft](/azure/active-directory-b2c/active-directory-b2c-setup-msa-app)、 [Facebook](/azure/active-directory-b2c/active-directory-b2c-setup-fb-app)、 [Google](/azure/active-directory-b2c/active-directory-b2c-setup-goog-app)、 [Amazon](/azure/active-directory-b2c/active-directory-b2c-setup-amzn-app)、 [Twitter](/azure/active-directory-b2c/active-directory-b2c-setup-twitter-app)等。
* [使用 Azure AD 图形 API](/azure/active-directory-b2c/active-directory-b2c-devquickstarts-graph-dotnet) 从 Azure AD B2C 租户检索其他用户信息，例如组成员身份。
* [如何使用 Azure AD B2C 保护使用 ASP.NET Core 生成的 WEB API](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/tree/master/4-WebApp-your-API/4-2-B2C)。
* [教程：使用 Azure Active Directory B2C 授予对 ASP.NET WEB API 的访问权限](/azure/active-directory-b2c/tutorial-web-api-dotnet)。
