---
title: 在 ASP.NET Core 中通过 SMS 进行双因素身份验证
author: rick-anderson
description: 了解如何使用 ASP.NET Core 应用程序 (2FA) 设置双因素身份验证。
monikerRange: < aspnetcore-2.0
ms.author: riande
ms.date: 09/22/2018
ms.custom: mvc, seodec18
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
uid: security/authentication/2fa
ms.openlocfilehash: 1ee9e656c2e631c9b5588149e0a75e07108baff1
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051260"
---
# <a name="two-factor-authentication-with-sms-in-aspnet-core"></a>在 ASP.NET Core 中通过 SMS 进行双因素身份验证

作者： [Rick Anderson](https://twitter.com/RickAndMSFT) 和 [瑞士-开发人员](https://github.com/Swiss-Devs)

>[!WARNING]
> 双因素身份验证 (2FA) 验证器应用，使用基于时间的一次性密码算法 (TOTP) ，是2FA 的行业建议方法。 2FA 使用 TOTP 是 SMS 2FA 的首选。 有关详细信息，请参阅为 ASP.NET Core 2.0 及更高版本 [ASP.NET Core 中的 TOTP 验证器应用启用 QR 码生成](xref:security/authentication/identity-enable-qrcodes) 。

本教程演示如何使用短信 (2FA) 设置双因素身份验证。 说明适用于 [twilio](https://www.twilio.com/) 和 [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/)，但你可以使用任何其他 SMS 提供程序。 建议你在开始学习本教程之前完成 [帐户确认和密码恢复](xref:security/authentication/accconfirm) 。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/2fa/sample/Web2FA)。 [如何下载](xref:index#how-to-download-a-sample)。

## <a name="create-a-new-aspnet-core-project"></a>创建新的 ASP.NET Core 项目

使用单个用户帐户创建名为的新 ASP.NET Core web 应用 `Web2FA` 。 按照中的说明 <xref:security/enforcing-ssl> 进行操作，设置并要求 HTTPS。

### <a name="create-an-sms-account"></a>创建 SMS 帐户

创建一个 SMS 帐户，例如 [twilio](https://www.twilio.com/) 或 [ASPSMS](https://www.aspsms.com/asp.net/identity/core/testcredits/)。 为 ASPSMS：用户密钥和 Password) 记录 twilio： accountSid 和 authToken (的身份验证凭据。

#### <a name="figuring-out-sms-provider-credentials"></a>了解 SMS 提供程序凭据

**Twilio**

在 Twilio 帐户的 "仪表板" 选项卡中，复制 " **帐户 SID** " 和 " **身份验证令牌** "。

**ASPSMS:**

从帐户设置中，导航到 **用户密钥** ，并将其与 **密码** 一起复制。

稍后，我们会将这些值存储在中，并在密钥和中存储机密管理器工具 `SMSAccountIdentification` `SMSAccountPassword` 。

#### <a name="specifying-senderid--originator"></a>指定 SenderID/发起方

**Twilio：** 从 "数字" 选项卡中，复制 Twilio 的 **电话号码** 。

**ASPSMS：** 在 "解锁始发者" 菜单中，解锁一个或多个发信方，或者选择 "所有网络) 不支持 (的字母数字发信方"。

稍后我们将在密钥中将此值与机密管理器工具存储在一起 `SMSAccountFrom` 。

### <a name="provide-credentials-for-the-sms-service"></a>提供 SMS 服务的凭据

我们将使用 [Options 模式](xref:fundamentals/configuration/options) 来访问用户帐户和密钥设置。

* 创建一个类以提取安全短信密钥。 对于本示例， `SMSoptions` 类在 *服务/SMSoptions* 文件中创建。

[!code-csharp[](2fa/sample/Web2FA/Services/SMSoptions.cs)]

`SMSAccountIdentification` `SMSAccountPassword` `SMSAccountFrom` 用[机密管理器工具](xref:security/app-secrets)设置和。 例如： 。

```none
C:/Web2FA/src/WebApp1>dotnet user-secrets set SMSAccountIdentification 12345
info: Successfully saved SMSAccountIdentification = 12345 to the secret store.
```

* 为 SMS 提供程序添加 NuGet 包。 从包管理器控制台中 (PMC) 运行：

**Twilio**

`Install-Package Twilio`

**ASPSMS:**

`Install-Package ASPSMS`

* 添加 *服务/MessageServices* 文件中的代码以启用 SMS。 使用 Twilio 或 ASPSMS 部分：

**Twilio**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_twilio.cs)]

**ASPSMS:**  
[!code-csharp[](2fa/sample/Web2FA/Services/MessageServices_ASPSMS.cs)]

### <a name="configure-startup-to-use-smsoptions"></a>配置要使用的启动 `SMSoptions`

将添加到 Startup.cs 中的方法中的 `SMSoptions` 服务容器 `ConfigureServices` ： *Startup.cs*

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet1&highlight=4)]

### <a name="enable-two-factor-authentication"></a>启用双因素身份验证

打开 *Views/管理/索引* Razor 视图文件并删除注释字符 (因此不) 注释掉标记。

## <a name="log-in-with-two-factor-authentication"></a>用双因素身份验证登录

* 运行应用并注册新用户

![在 Microsoft Edge 中打开的 Web 应用程序注册视图](2fa/_static/login2fa1.png)

* 点击你的用户名，该名称将激活 `Index` 管理控制器中的操作方法。 然后点击 "电话号码" " **添加** " 链接。

![管理视图-点击 "添加" 链接](2fa/_static/login2fa2.png)

* 添加将接收验证码的电话号码，然后点击 " **发送验证码** "。

![添加电话号码页面](2fa/_static/login2fa3.png)

* 你将收到一条包含验证码的短信。 输入它，然后点击 " **提交** "

![验证电话号码页面](2fa/_static/login2fa4.png)

如果未收到短信，请参阅 twilio log page。

* "管理" 视图显示已成功添加的电话号码。

![管理视图-电话号码已成功添加](2fa/_static/login2fa5.png)

* 点击 " **启用** " 以启用双因素身份验证。

![管理视图-启用双因素身份验证](2fa/_static/login2fa6.png)

### <a name="test-two-factor-authentication"></a>测试双因素身份验证

* 注销。

* 登录。

* 用户帐户已启用双因素身份验证，因此你必须提供第二个身份验证因素。 在本教程中，你已启用电话验证。 内置模板还允许您将电子邮件设置为第二个因素。 你可以设置其他第二种身份验证因素，例如 QR 码。 点击 " **提交** "。

![发送验证代码视图](2fa/_static/login2fa7.png)

* 输入在 SMS 消息中获取的代码。

* 单击 " **记住此浏览器** " 复选框会使你不需要使用2FA 在使用同一设备和浏览器时进行登录。 如果启用2FA 并单击 "记住"， **此浏览器** 将为你提供强大的2FA 防护，防止恶意用户尝试访问你的帐户，只要他们无权访问你的设备。 你可以在你经常使用的任何专用设备上执行此操作。 通过设置  **记住此浏览器** ，你可以从不经常使用的设备中获得2FA 的增强的安全性，并且你无需在自己的设备上通过2FA。

![验证视图](2fa/_static/login2fa8.png)

## <a name="account-lockout-for-protecting-against-brute-force-attacks"></a>帐户锁定，防止暴力破解攻击

建议使用2FA 帐户锁定。 用户通过本地帐户或社交帐户登录后，将存储2FA 的每次失败尝试。 如果达到了失败的最大访问尝试次数，则用户将被锁定 (默认值：5分钟在尝试访问尝试失败5次失败后锁定) 。 身份验证成功后，将重置失败的访问尝试计数并重置时钟。 可通过 [MaxFailedAccessAttempts](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.maxfailedaccessattempts) 和 [DefaultLockoutTimeSpan](/dotnet/api/microsoft.aspnetcore.identity.lockoutoptions.defaultlockouttimespan)设置最大失败访问尝试和锁定时间。 以下内容将帐户锁定配置为10分钟后失败的访问尝试次数：

[!code-csharp[](2fa/sample/Web2FA/Startup.cs?name=snippet2&highlight=13-17)]

确认 [PasswordSignInAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.passwordsigninasync) 设置 `lockoutOnFailure` 为 `true` ：

```csharp
var result = await _signInManager.PasswordSignInAsync(
                 Input.Email, Input.Password, Input.RememberMe, lockoutOnFailure: true);
```
