---
title: ASP.NET Core 中的帐户确认和密码恢复
author: rick-anderson
description: 了解如何使用电子邮件确认和密码重置构建 ASP.NET Core 应用。
ms.author: riande
ms.date: 03/11/2019
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
uid: security/authentication/accconfirm
ms.openlocfilehash: 91148c67d5dc0bf97e2f926f50dcff5dd0708f4b
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052313"
---
# <a name="account-confirmation-and-password-recovery-in-aspnet-core"></a>ASP.NET Core 中的帐户确认和密码恢复

作者： [Rick Anderson](https://twitter.com/RickAndMSFT)、 [Ponant](https://github.com/Ponant)和 [Joe Audette](https://twitter.com/joeaudette)

本教程介绍如何使用电子邮件确认和密码重置构建 ASP.NET Core 应用。 本教程 **不** 是开始主题。 你应该熟悉：

* [ASP.NET Core](xref:tutorials/razor-pages/razor-pages-start)
* [身份验证](xref:security/authentication/identity)
* [Entity Framework Core](xref:data/ef-mvc/intro)

<!-- see C:/Dropbox/wrk/Code/SendGridConsole/Program.cs -->

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a>先决条件

[.NET Core 3.0 SDK 或更高版本](https://dotnet.microsoft.com/download/dotnet-core/3.0)

## <a name="create-and-test-a-web-app-with-authentication"></a>创建和测试使用身份验证的 web 应用

运行以下命令，创建具有身份验证的 web 应用。

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet run
```

运行应用，选择 " **注册** " 链接，然后注册用户。 注册后，你会被重定向到 "目标" `/Identity/Account/RegisterConfirmation` 页，其中包含用于模拟电子邮件确认的链接：

* 选择 `Click here to confirm your account` 链接。
* 选择 " **登录** " 链接，并以相同的凭据登录。
* 选择将 `Hello YourEmail@provider.com!` 您重定向到页面的链接 `/Identity/Account/Manage/PersonalData` 。
* 选择左侧的 " **个人数据** " 选项卡，然后选择 " **删除** "。

### <a name="configure-an-email-provider"></a>配置电子邮件提供程序

在本教程中，使用 [SendGrid](https://sendgrid.com) 发送电子邮件。 需要使用 SendGrid 帐户和密钥来发送电子邮件。 您可以使用其他电子邮件提供程序。 建议使用 SendGrid 或其他电子邮件服务发送电子邮件。 SMTP 难于保护和正确设置。

SendGrid 帐户可能需要 [添加发送方](https://sendgrid.com/docs/ui/sending-email/senders/)。

创建一个类以获取安全电子邮件密钥。 对于本示例，请创建 *服务/AuthMessageSenderOptions* ：

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>配置 SendGrid 用户机密

`SendGridUser` `SendGridKey` 用[机密管理器工具](xref:security/app-secrets)设置和。 例如： 。

```dotnetcli
dotnet user-secrets set SendGridUser RickAndMSFT
dotnet user-secrets set SendGridKey <key>

Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

在 Windows 上，机密管理器将密钥/值对存储在目录中的文件 *secrets.js上* `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` 。

不会对文件 *中secrets.js* 的内容进行加密。 以下标记显示了文件 *上的secrets.js* 。 已 `SendGridKey` 删除该值。

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

有关详细信息，请参阅 [Options 模式](xref:fundamentals/configuration/options) 和 [配置](xref:fundamentals/configuration/index)。

### <a name="install-sendgrid"></a>安装 SendGrid

本教程介绍如何通过 [SendGrid](https://sendgrid.com/)添加电子邮件通知，但你可以使用 SMTP 和其他机制发送电子邮件。

安装 `SendGrid` NuGet 包：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 "包管理器控制台" 中，输入以下命令：

```powershell
Install-Package SendGrid
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

在控制台中，输入以下命令：

```dotnetcli
dotnet add package SendGrid
```

---

请参阅 [SendGrid 的入门免费](https://sendgrid.com/free/) 版，注册免费 SendGrid 帐户。

### <a name="implement-iemailsender"></a>实现 IEmailSender

若要实现 `IEmailSender` ，请创建具有类似于下面的代码的 *服务/EmailSender* ：

[!code-csharp[](accconfirm/sample/WebPWrecover30/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>配置启动以支持电子邮件

将以下代码添加到 `ConfigureServices` *Startup.cs* 文件中的方法：

* 添加 `EmailSender` 为暂时性服务。
* 注册 `AuthMessageSenderOptions` 配置实例。

[!code-csharp[](accconfirm/sample/WebPWrecover30/Startup.cs?name=snippet1&highlight=11-15)]

## <a name="scaffold-registerconfirmation"></a>基架 RegisterConfirmation

按照[基架 Identity ](xref:security/authentication/scaffold-identity)和基架的说明进行操作 `RegisterConfirmation` 。

<!-- .NET 5 fixes this, see
https://github.com/dotnet/aspnetcore/blob/master/src/Identity/UI/src/Areas/Identity/Pages/V4/Account/RegisterConfirmation.cshtml.cs#L74-L77
-->

[!INCLUDE[](~/includes/disableVer.md)]

## <a name="register-confirm-email-and-reset-password"></a>注册、确认电子邮件并重置密码

运行 web 应用，并测试帐户确认和密码恢复流。

* 运行应用并注册新用户
* 检查电子邮件中的 "帐户确认" 链接。 如果没有收到电子邮件，请参阅 [调试电子邮件](#debug) 。
* 单击链接以确认你的电子邮件。
* 用电子邮件和密码登录。
* 注销。

### <a name="test-password-reset"></a>测试密码重置

* 如果已登录，请选择 " **注销** "。
* 选择 " **登录** " 链接，然后选择 " **忘记了密码？"** 链接。
* 输入用于注册该帐户的电子邮件。
* 发送了一封电子邮件，其中包含用于重置密码的链接。 检查你的电子邮件，然后单击链接以重置你的密码。 密码重置成功后，可以用电子邮件和新密码登录。

<a name="resend"></a>

## <a name="resend-email-confirmation"></a>重新发送电子邮件确认

在 ASP.NET Core 5.0 及更高版本中，选择 " **登录** " 页上的 " **重新发送电子邮件** " 链接。

### <a name="change-email-and-activity-timeout"></a>更改电子邮件和活动超时

默认的非活动超时为14天。 下面的代码将非活动超时设置为5天：

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>更改所有数据保护令牌 lifespans

以下代码将所有数据保护令牌超时期限更改为3小时：

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupAllTokens.cs?name=snippet1&highlight=11-12)]

内置 Identity 用户令牌 (参阅 [AspNetCore/src/ Identity /Extensions.Core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) 具有一天的 [超时时间](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)。

### <a name="change-the-email-token-lifespan"></a>更改电子邮件令牌的生命周期

[ Identity 用户令牌](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs)的默认令牌生存期为[1 天](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)。 本部分介绍如何更改电子邮件令牌的生命周期。

添加自定义[DataProtectorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1)和 <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions> ：

[!code-csharp[](accconfirm/sample/WebPWrecover30/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

将自定义提供程序添加到服务容器：

[!code-csharp[](accconfirm/sample/WebPWrecover30/StartupEmail.cs?name=snippet1&highlight=10-16)]

<a name="debug"></a>

### <a name="debug-email"></a>调试电子邮件

如果无法使用电子邮件：

* 在中设置一个断点 `EmailSender.Execute` ，以验证 `SendGridClient.SendEmailAsync` 调用。
* 创建一个 [控制台应用，用于](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) 使用类似的代码向发送电子邮件 `EmailSender.Execute` 。
* 查看 [电子邮件活动](https://sendgrid.com/docs/User_Guide/email_activity.html) 页。
* 检查垃圾邮件文件夹。
* 尝试在 Microsoft、Yahoo、Gmail 等不同的电子邮件 (提供程序上使用另一个电子邮件别名 ) 
* 尝试发送到不同的电子邮件帐户。

**最佳安全做法** 是 **不** 在测试和开发中使用生产机密。 如果将应用发布到 Azure，请在 Azure Web 应用门户中将 "SendGrid 机密" 设置为 "应用程序设置"。 配置系统设置为从环境变量读取密钥。

## <a name="combine-social-and-local-login-accounts"></a>合并社会和本地登录帐户

若要完成本部分，必须首先启用外部身份验证提供程序。 请参阅 [Facebook、Google 和外部提供程序身份验证](xref:security/authentication/social/index)。

可以通过单击电子邮件链接来合并本地帐户和社交帐户。 按照以下顺序，" RickAndMSFT@gmail.com " 首先创建为本地登录名; 但是，你可以先将该帐户创建为社交登录名，然后添加本地登录名。

![Web 应用程序： RickAndMSFT@gmail.com 用户已进行身份验证](accconfirm/_static/rick.png)

单击 " **管理** " 链接。 请注意，0外部 (社交登录) 与此帐户关联。

![管理视图](accconfirm/_static/manage.png)

单击指向另一登录服务的链接，并接受应用请求。 在下图中，Facebook 是外部身份验证提供程序：

![管理你的外部登录名视图列表 Facebook](accconfirm/_static/fb.png)

这两个帐户已组合在一起。 你可以用任一帐户登录。 你可能希望用户在社交登录身份验证服务关闭时添加本地帐户，或者更可能的情况是他们失去了社交帐户的访问权限。

## <a name="enable-account-confirmation-after-a-site-has-users"></a>在站点包含用户后启用帐户确认

在具有用户的站点上启用帐户确认会锁定所有现有用户。 现有用户被锁定，因为其帐户未得到确认。 若要解决现有用户锁定，请使用以下方法之一：

* 更新数据库，将所有现有用户标记为已确认。
* 确认现有用户。 例如，批处理-发送包含确认链接的电子邮件。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

## <a name="prerequisites"></a>先决条件

[.NET Core 2.2 SDK 或更高版本](https://dotnet.microsoft.com/download/dotnet-core)

## <a name="create-a-web--app-and-scaffold-no-locidentity"></a>创建 web 应用和基架 Identity

运行以下命令，创建具有身份验证的 web 应用。

```dotnetcli
dotnet new webapp -au Individual -uld -o WebPWrecover
cd WebPWrecover
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator identity -dc WebPWrecover.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.ConfirmEmail"
dotnet ef database drop -f
dotnet ef database update
dotnet run
```

> [!NOTE]
> 如果 <xref:Microsoft.AspNetCore.Identity.PasswordOptions> 在中配置了 `Startup.ConfigureServices` ，则基架页中的属性可能需要[ `[StringLength]` 属性](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute)配置 `Password` Identity 。 在 `InputModel` `Password` `Areas/Identity/Pages/Account/Register.cshtml.cs` 基架后面的文件中找到属性 Identity 。

## <a name="test-new-user-registration"></a>测试新用户注册

运行应用，选择 " **注册** " 链接，然后注册用户。 此时，电子邮件的唯一验证是具有 [`[EmailAddress]`](/dotnet/api/system.componentmodel.dataannotations.emailaddressattribute) 属性。 提交注册后，将登录到应用。 在本教程的后面部分，将更新代码，以便新用户在验证其电子邮件之前无法登录。

[!INCLUDE[](~/includes/view-identity-db.md)]

请注意，表的 `EmailConfirmed` 字段为 `False` 。

当应用发送确认电子邮件时，可能需要在下一步中再次使用此电子邮件。 右键单击该行，然后选择 " **删除** "。 删除电子邮件别名可以简化以下步骤。

<a name="prevent-login-at-registration"></a>

## <a name="require-email-confirmation"></a>需要确认电子邮件

最佳做法是确认新用户注册的电子邮件。 电子邮件确认有助于验证他们是否未模拟其他人 (也就是说，他们尚未注册其他人的电子邮件) 。 假设你有讨论论坛，并且想要阻止 " yli@example.com " 注册为 " nolivetto@contoso.com "。 如果未确认电子邮件，" nolivetto@contoso.com " 可能会从你的应用收到不需要的电子邮件。 假设用户意外注册为 " ylo@example.com "，但未注意到 "yli" 的拼写错误。 它们不能使用密码恢复，因为该应用没有正确的电子邮件。 电子邮件确认为 bot 提供有限的保护。 电子邮件确认不会为具有多个电子邮件帐户的恶意用户提供保护。

通常，在用户确认电子邮件之前，会阻止新用户将任何数据发布到您的网站。

更新 `Startup.ConfigureServices`  以要求确认电子邮件：

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=8-11)]

`config.SignIn.RequireConfirmedEmail = true;` 阻止注册的用户登录，直到其电子邮件得到确认。

### <a name="configure-email-provider"></a>配置电子邮件提供程序

在本教程中，使用 [SendGrid](https://sendgrid.com) 发送电子邮件。 需要使用 SendGrid 帐户和密钥来发送电子邮件。 您可以使用其他电子邮件提供程序。 ASP.NET Core 1.x 包括 `System.Net.Mail` ，这允许你从应用发送电子邮件。 建议使用 SendGrid 或其他电子邮件服务发送电子邮件。 SMTP 难于保护和正确设置。

创建一个类以获取安全电子邮件密钥。 对于本示例，请创建 *服务/AuthMessageSenderOptions* ：

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/AuthMessageSenderOptions.cs?name=snippet1)]

#### <a name="configure-sendgrid-user-secrets"></a>配置 SendGrid 用户机密

`SendGridUser` `SendGridKey` 用[机密管理器工具](xref:security/app-secrets)设置和。 例如： 。

```console
C:/WebAppl>dotnet user-secrets set SendGridUser RickAndMSFT
info: Successfully saved SendGridUser = RickAndMSFT to the secret store.
```

在 Windows 上，机密管理器将密钥/值对存储在目录中的文件 *secrets.js上* `%APPDATA%/Microsoft/UserSecrets/<WebAppName-userSecretsId>` 。

不会对文件 *中secrets.js* 的内容进行加密。 以下标记显示了文件 *上的secrets.js* 。 已 `SendGridKey` 删除该值。

```json
{
  "SendGridUser": "RickAndMSFT",
  "SendGridKey": "<key removed>"
}
```

有关详细信息，请参阅 [Options 模式](xref:fundamentals/configuration/options) 和 [配置](xref:fundamentals/configuration/index)。

### <a name="install-sendgrid"></a>安装 SendGrid

本教程介绍如何通过 [SendGrid](https://sendgrid.com/)添加电子邮件通知，但你可以使用 SMTP 和其他机制发送电子邮件。

安装 `SendGrid` NuGet 包：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 "包管理器控制台" 中，输入以下命令：

```powershell
Install-Package SendGrid
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

在控制台中，输入以下命令：

```dotnetcli
dotnet add package SendGrid
```

---

请参阅 [SendGrid 的入门免费](https://sendgrid.com/free/) 版，注册免费 SendGrid 帐户。

### <a name="implement-iemailsender"></a>实现 IEmailSender

若要实现 `IEmailSender` ，请创建具有类似于下面的代码的 *服务/EmailSender* ：

[!code-csharp[](accconfirm/sample/WebPWrecover22/Services/EmailSender.cs)]

### <a name="configure-startup-to-support-email"></a>配置启动以支持电子邮件

将以下代码添加到 `ConfigureServices` *Startup.cs* 文件中的方法：

* 添加 `EmailSender` 为暂时性服务。
* 注册 `AuthMessageSenderOptions` 配置实例。

[!code-csharp[](accconfirm/sample/WebPWrecover22/Startup.cs?name=snippet1&highlight=15-99)]

## <a name="enable-account-confirmation-and-password-recovery"></a>启用帐户确认和密码恢复

该模板包含用于帐户确认和密码恢复的代码。 `OnPostAsync`在 *区域/ Identity /Pages/Account/Register.cshtml.cs* 中查找方法。

通过注释掉以下行，阻止新注册的用户自动登录：

```csharp
await _signInManager.SignInAsync(user, isPersistent: false);
```

将显示完整的方法，其中突出显示了已更改的行：

[!code-csharp[](accconfirm/sample/WebPWrecover22/Areas/Identity/Pages/Account/Register.cshtml.cs?highlight=22&name=snippet_Register)]

## <a name="register-confirm-email-and-reset-password"></a>注册、确认电子邮件并重置密码

运行 web 应用，并测试帐户确认和密码恢复流。

* 运行应用并注册新用户
* 检查电子邮件中的 "帐户确认" 链接。 如果没有收到电子邮件，请参阅 [调试电子邮件](#debug) 。
* 单击链接以确认你的电子邮件。
* 用电子邮件和密码登录。
* 注销。

### <a name="view-the-manage-page"></a>查看 "管理" 页

在浏览器中选择你的用户名，并在浏览器窗口中选择用户名 ![](accconfirm/_static/un.png)

将显示 "管理" 页，并选中 " **配置文件** " 选项卡。 该 **电子邮件** 将显示一个复选框，指示已确认电子邮件。

### <a name="test-password-reset"></a>测试密码重置

* 如果已登录，请选择 " **注销** "。
* 选择 " **登录** " 链接，然后选择 " **忘记了密码？"** 链接。
* 输入用于注册该帐户的电子邮件。
* 发送了一封电子邮件，其中包含用于重置密码的链接。 检查你的电子邮件，然后单击链接以重置你的密码。 密码重置成功后，可以用电子邮件和新密码登录。

## <a name="change-email-and-activity-timeout"></a>更改电子邮件和活动超时

默认的非活动超时为14天。 下面的代码将非活动超时设置为5天：

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAppCookie.cs?name=snippet1)]

### <a name="change-all-data-protection-token-lifespans"></a>更改所有数据保护令牌 lifespans

以下代码将所有数据保护令牌超时期限更改为3小时：

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupAllTokens.cs?name=snippet1&highlight=15-16)]

内置 Identity 用户令牌 (参阅 [AspNetCore/src/ Identity /Extensions.Core/src/TokenOptions.cs](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs) ) 具有一天的 [超时时间](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)。

### <a name="change-the-email-token-lifespan"></a>更改电子邮件令牌的生命周期

[ Identity 用户令牌](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Extensions.Core/src/TokenOptions.cs)的默认令牌生存期为[1 天](https://github.com/dotnet/AspNetCore/blob/v2.2.2/src/Identity/Core/src/DataProtectionTokenProviderOptions.cs)。 本部分介绍如何更改电子邮件令牌的生命周期。

添加自定义[DataProtectorTokenProvider \<TUser> ](/dotnet/api/microsoft.aspnetcore.identity.dataprotectortokenprovider-1)和 <xref:Microsoft.AspNetCore.Identity.DataProtectionTokenProviderOptions> ：

[!code-csharp[](accconfirm/sample/WebPWrecover22/TokenProviders/CustomTokenProvider.cs?name=snippet1)]

将自定义提供程序添加到服务容器：

[!code-csharp[](accconfirm/sample/WebPWrecover22/StartupEmail.cs?name=snippet1&highlight=10-13,18)]

### <a name="resend-email-confirmation"></a>重新发送电子邮件确认

请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore/issues/5410)。

<a name="debug"></a>

### <a name="debug-email"></a>调试电子邮件

如果无法使用电子邮件：

* 在中设置一个断点 `EmailSender.Execute` ，以验证 `SendGridClient.SendEmailAsync` 调用。
* 创建一个 [控制台应用，用于](https://sendgrid.com/docs/Integrate/Code_Examples/v2_Mail/csharp.html) 使用类似的代码向发送电子邮件 `EmailSender.Execute` 。
* 查看 [电子邮件活动](https://sendgrid.com/docs/User_Guide/email_activity.html) 页。
* 检查垃圾邮件文件夹。
* 尝试在 Microsoft、Yahoo、Gmail 等不同的电子邮件 (提供程序上使用另一个电子邮件别名 ) 
* 尝试发送到不同的电子邮件帐户。

**最佳安全做法** 是 **不** 在测试和开发中使用生产机密。 如果将应用发布到 Azure，则可以在 Azure Web 应用门户中将 SendGrid 机密设置为应用程序设置。 配置系统设置为从环境变量读取密钥。

## <a name="combine-social-and-local-login-accounts"></a>合并社会和本地登录帐户

若要完成本部分，必须首先启用外部身份验证提供程序。 请参阅 [Facebook、Google 和外部提供程序身份验证](xref:security/authentication/social/index)。

可以通过单击电子邮件链接来合并本地帐户和社交帐户。 按照以下顺序，" RickAndMSFT@gmail.com " 首先创建为本地登录名; 但是，你可以先将该帐户创建为社交登录名，然后添加本地登录名。

![Web 应用程序： RickAndMSFT@gmail.com 用户已进行身份验证](accconfirm/_static/rick.png)

单击 " **管理** " 链接。 请注意，0外部 (社交登录) 与此帐户关联。

![管理视图](accconfirm/_static/manage.png)

单击指向另一登录服务的链接，并接受应用请求。 在下图中，Facebook 是外部身份验证提供程序：

![管理你的外部登录名视图列表 Facebook](accconfirm/_static/fb.png)

这两个帐户已组合在一起。 你可以用任一帐户登录。 你可能希望用户在社交登录身份验证服务关闭时添加本地帐户，或者更可能的情况是他们失去了社交帐户的访问权限。

## <a name="enable-account-confirmation-after-a-site-has-users"></a>在站点包含用户后启用帐户确认

在具有用户的站点上启用帐户确认会锁定所有现有用户。 现有用户被锁定，因为其帐户未得到确认。 若要解决现有用户锁定，请使用以下方法之一：

* 更新数据库，将所有现有用户标记为已确认。
* 确认现有用户。 例如，批处理-发送包含确认链接的电子邮件。

::: moniker-end
