---
title: 阻止 ASP.NET Core 中的开放重定向攻击
author: ardalis
description: 演示如何防止针对 ASP.NET Core 应用的开放重定向攻击
ms.author: riande
ms.date: 07/07/2017
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
uid: security/preventing-open-redirects
ms.openlocfilehash: e546cd852367921c7c694db3639f7a233f606e75
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93058384"
---
# <a name="prevent-open-redirect-attacks-in-aspnet-core"></a>阻止 ASP.NET Core 中的开放重定向攻击

重定向到通过请求（如 querystring 或窗体数据）指定的 URL 的 web 应用可能会被篡改，以将用户重定向到外部恶意 URL。 这种篡改称为 "开放重定向攻击"。

每当应用程序逻辑重定向到指定的 URL 时，必须验证重定向 URL 是否未被篡改。 ASP.NET Core 提供内置功能，可帮助保护应用程序的打开重定向 (也称为 "开放重定向) 攻击"。

## <a name="what-is-an-open-redirect-attack"></a>什么是开放重定向攻击？

Web 应用程序在访问需要身份验证的资源时经常将用户重定向到登录页。 重定向通常包括 `returnUrl` 查询字符串参数，以便用户在成功登录后可以返回到最初请求的 URL。 用户进行身份验证后，会重定向到最初请求的 URL。

由于目标 URL 是在请求的查询字符串中指定的，因此恶意用户可能会篡改 querystring。 篡改的 querystring 可能允许站点将用户重定向到外部的恶意站点。 此方法称为 " (重定向" 或 "重定向) 攻击"。

### <a name="an-example-attack"></a>示例攻击

恶意用户可能会受到攻击，目的是允许恶意用户访问用户的凭据或敏感信息。 若要开始攻击，恶意用户结论用户单击指向站点登录页的链接，并将 `returnUrl` 查询字符串值添加到 URL。 例如，请考虑一个在上 `contoso.com` 包含登录页的应用 `http://contoso.com/Account/LogOn?returnUrl=/Home/About` 。 攻击执行以下步骤：

1. 用户单击恶意链接 `http://contoso.com/Account/LogOn?returnUrl=http://contoso1.com/Account/LogOn` (第二个 URL 为 "contoso **1** .com"，而不是 "contoso.com" ) 。
2. 用户已成功登录。
3. 站点)  (重定向用户， `http://contoso1.com/Account/LogOn` (与真实站点) 完全相同的恶意网站。
4. 用户再次登录 (向恶意网站提供凭据) 并被重定向回真实站点。

用户可能认为他们第一次尝试登录失败，第二次尝试成功。 用户很可能仍不知道他们的凭据已泄露。

![打开重定向攻击过程](preventing-open-redirects/_static/open-redirection-attack-process.png)

除了登录页以外，某些站点还提供重定向页面或终结点。 假设你的应用程序有一个页面，其中包含一个打开的重定向 `/Home/Redirect` 。 例如，攻击者可以创建发送到的电子邮件中的链接 `[yoursite]/Home/Redirect?url=http://phishingsite.com/Home/Login` 。 典型用户将查看 URL，并将其以你的站点名称开头。 信任这一点，他们将单击该链接。 然后，打开的重定向会将用户发送到仿冒网站，该网站看上去与你的站点相同，并且用户可能会登录到他们认为是你的网站。

## <a name="protecting-against-open-redirect-attacks"></a>防范开放重定向攻击

开发 web 应用程序时，将所有用户提供的数据视为不可信。 如果你的应用程序具有基于 URL 内容重定向用户的功能，请确保此类重定向仅在你的应用程序中本地执行 (或到已知 URL，而不是在 querystring) 中提供的任何 URL。

### <a name="localredirect"></a>LocalRedirect

使用 `LocalRedirect` 基类中的 helper 方法 `Controller` ：

```csharp
public IActionResult SomeAction(string redirectUrl)
{
    return LocalRedirect(redirectUrl);
}
```

`LocalRedirect` 如果指定了一个非本地 URL，则将引发异常。 否则，它的行为与 `Redirect` 方法相同。

### <a name="islocalurl"></a>IsLocalUrl

重定向之前，请使用 [IsLocalUrl](/dotnet/api/Microsoft.AspNetCore.Mvc.IUrlHelper.islocalurl#Microsoft_AspNetCore_Mvc_IUrlHelper_IsLocalUrl_System_String_) 方法测试 url：

下面的示例演示如何在重定向前检查 URL 是否是本地的。

```csharp
private IActionResult RedirectToLocal(string returnUrl)
{
    if (Url.IsLocalUrl(returnUrl))
    {
        return Redirect(returnUrl);
    }
    else
    {
        return RedirectToAction(nameof(HomeController.Index), "Home");
    }
}
```

此 `IsLocalUrl` 方法可防止用户无意中重定向到恶意网站。 您可以记录在您需要本地 URL 的情况下提供非本地 URL 时提供的 URL 的详细信息。 记录重定向 Url 有助于诊断重定向攻击。
