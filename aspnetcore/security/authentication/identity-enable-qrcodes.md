---
title: 为 ASP.NET Core 中的 TOTP 验证器应用启用 QR 代码生成
author: rick-anderson
description: 了解如何为使用 ASP.NET Core 双因素身份验证的 TOTP 验证器应用启用 QR 码生成。
ms.author: riande
ms.date: 08/14/2018
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
uid: security/authentication/identity-enable-qrcodes
ms.openlocfilehash: b778e7238911ec9966edf7f0f7becd113b1e197a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060828"
---
# <a name="enable-qr-code-generation-for-totp-authenticator-apps-in-aspnet-core"></a>为 ASP.NET Core 中的 TOTP 验证器应用启用 QR 代码生成

::: moniker range="<= aspnetcore-2.0"

QR 码需要 ASP.NET Core 2.0 或更高版本。

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

ASP.NET Core 附带了对验证程序应用程序的支持，以便进行单个身份验证。 双因素身份验证 (2FA) 验证器应用，使用基于时间的一次性密码算法 (TOTP) ，是2FA 的行业建议方法。 2FA 使用 TOTP 是 SMS 2FA 的首选。 验证器应用提供6到8位的数字代码，用户在确认其用户名和密码后必须输入。 通常会在智能手机上安装验证器应用。

ASP.NET Core web 应用模板支持验证器，但不要为 QRCode 生成提供支持。 QRCode 生成器简化了2FA 的设置。 本文档将指导你完成将 [QR 代码](https://wikipedia.org/wiki/QR_code) 生成添加到 "2FA" 配置页的步骤。

使用外部身份验证提供程序（如 [Google](xref:security/authentication/google-logins) 或 [Facebook](xref:security/authentication/facebook-logins)）不会进行双重身份验证。 外部登录名受外部登录提供程序所提供的任何机制的保护。 例如，请考虑 [Microsoft](xref:security/authentication/microsoft-logins) 身份验证提供程序需要硬件密钥或其他2FA 方法。 如果默认模板强制实施了 "本地" 2FA，则用户需要满足两种2FA 方法，这并不是一种常用方案。

## <a name="adding-qr-codes-to-the-2fa-configuration-page"></a>将 QR 代码添加到 "2FA" 配置页

这些说明使用存储库中的 *qrcode.js* https://davidshimjs.github.io/qrcodejs/ 。

* 将 [qrcode.js javascript 库](https://davidshimjs.github.io/qrcodejs/) 下载到 `wwwroot\lib` 项目中的文件夹。

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

* 按照 [基架 Identity](xref:security/authentication/scaffold-identity)中的说明生成 */Areas/ Identity /Pages/Account/Manage/EnableAuthenticator.cshtml* 。
* 在 */Areas/ Identity /Pages/Account/Manage/EnableAuthenticator.cshtml* 中，找到 `Scripts` 文件末尾的部分：

::: moniker-end

::: moniker range="= aspnetcore-2.0"

* 在 " *页面/帐户/管理/EnableAuthenticator* " (Razor 页面) 或 *视图/管理/EnableAuthenticator* (MVC) 中，找到 `Scripts` 文件末尾的部分：

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

```cshtml
@section Scripts {
    @await Html.PartialAsync("_ValidationScriptsPartial")
}
```

* 更新 `Scripts` 节以添加对添加的库的引用 `qrcodejs` ，并调用以生成 QR 代码。 其外观应如下所示：

```cshtml
@section Scripts {
    @await Html.PartialAsync("_ValidationScriptsPartial")

    <script type="text/javascript" src="~/lib/qrcode.js"></script>
    <script type="text/javascript">
        new QRCode(document.getElementById("qrCode"),
            {
                text: "@Html.Raw(Model.AuthenticatorUri)",
                width: 150,
                height: 150
            });
    </script>
}
```

* 删除链接到这些说明的段落。

运行你的应用，并确保你可以扫描 QR 代码并验证验证器证明的代码。

## <a name="change-the-site-name-in-the-qr-code"></a>更改 QR 码中的站点名称

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

QR 代码中的站点名称取自最初创建项目时选择的项目名称。 可以通过在 `GenerateQrCodeUri(string email, string unformattedKey)` */Areas/ Identity /Pages/Account/Manage/EnableAuthenticator.cshtml.cs* 中查找方法来更改该方法。

::: moniker-end

::: moniker range="= aspnetcore-2.0"

QR 代码中的站点名称取自最初创建项目时选择的项目名称。 您可以通过在 `GenerateQrCodeUri(string email, string unformattedKey)` " *页面/帐户/管理/EnableAuthenticator* " (Razor 页面) 文件或 *控制器/ManageController* (MVC) 文件中查找方法来更改该方法。

::: moniker-end

::: moniker range=">= aspnetcore-2.0"

模板中的默认代码如下所示：

```csharp
private string GenerateQrCodeUri(string email, string unformattedKey)
{
    return string.Format(
        AuthenticatorUriFormat,
        _urlEncoder.Encode("Razor Pages"),
        _urlEncoder.Encode(email),
        unformattedKey);
}
```

对的调用中的第二个参数 `string.Format` 是您的站点名称，从您的解决方案名称获取。 可以将其更改为任何值，但必须始终以 URL 编码。

## <a name="using-a-different-qr-code-library"></a>使用其他 QR 码库

可以将 QR 代码库替换为首选库。 HTML 包含一个 `qrCode` 元素，你可以在其中放置 QR 码，方法是使用库提供的任何机制。

可以在中找到 QR 码的格式正确的 URL：

* `AuthenticatorUri` 模型的属性。
* `data-url` 元素中的属性 `qrCodeData` 。

## <a name="totp-client-and-server-time-skew"></a>TOTP 客户端和服务器时间偏差

TOTP (基于时间的 One-Time 密码) 身份验证取决于服务器和验证器设备的时间是否准确。 标记只持续30秒。 如果 TOTP 2FA 登录失败，请检查服务器时间是否准确，并最好是同步到准确的 NTP 服务。

::: moniker-end
