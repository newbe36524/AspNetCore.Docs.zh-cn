---
title: 使用 cookie ASP.NET Core 中的 SameSite
author: rick-anderson
description: 了解如何使用 cookie 在 ASP.NET Core 中 SameSite s
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
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
- Electron
uid: security/samesite
ms.openlocfilehash: 6f826416e3045df32abf41e94e667120e71ae717
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051611"
---
# <a name="work-with-samesite-no-loccookies-in-aspnet-core"></a>使用 cookie ASP.NET Core 中的 SameSite

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

SameSite 是一种 [IETF](https://ietf.org/about/) 草案标准，旨在针对跨站点请求伪造 (CSRF) 攻击提供一些保护。 最初在 [2016](https://tools.ietf.org/html/draft-west-first-party-cookies-07)中起草草案标准版已在 [2019](https://tools.ietf.org/html/draft-west-cookie-incrementalism-00)中更新。 更新的标准与以前的标准不是向后兼容，以下是最明显的区别：

* Cookie默认情况下，不使用 SameSite 标头 `SameSite=Lax` 。
* `SameSite=None` 必须使用以允许跨站点 cookie 使用。
* Cookie断言 `SameSite=None` 还必须标记为 `Secure` 。
* 使用的应用程序 [`<iframe>`](https://developer.mozilla.org/docs/Web/HTML/Element/iframe) 可能会遇到或的问题， `sameSite=Lax` `sameSite=Strict` cookie 因为 `<iframe>` 被视为跨站点方案。
* `SameSite=None` [2016 标准](https://tools.ietf.org/html/draft-west-first-party-cookies-07)不允许该值，并导致某些实现将此类视为 cookie `SameSite=Strict` 。 请参阅本文档中的 [支持旧版浏览器](#sob) 。

此 `SameSite=Lax` 设置适用于大多数应用程序 cookie 。 某些形式的身份验证，例如 [OpenID connect](https://openid.net/connect/) (OIDC) ，并且 [WS 联合](https://auth0.com/docs/protocols/ws-fed) 身份验证默认为基于 POST 的重定向。 基于后期的重定向会触发 SameSite 浏览器保护，因此，对这些组件禁用了 SameSite。 由于请求的流动方式不同，大多数 [OAuth](https://oauth.net/) 登录名不受影响。

发出的每个 ASP.NET Core 组件都 cookie 需要确定 SameSite 是否合适。

## <a name="samesite-and-no-locidentity"></a>SameSite 和 Identity

[!INCLUDE[](~/includes/SameSiteIdentity.md)]

## <a name="samesite-test-sample-code"></a>SameSite 测试示例代码

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

可下载和测试以下示例：

| 示例               | 文档 |
| ----------------- | ------------ |
| [.NET Core MVC](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [.NET Core Razor 页](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

可下载并测试以下示例：


| 示例               | 文档 |
| ----------------- | ------------ |
| [.NET Core Razor 页](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

## <a name="net-core-support-for-the-samesite-attribute"></a>.NET Core 对 sameSite 属性的支持

.NET Core 2.2 和更高版本支持 SameSite 的2019草稿标准版，因为年12月2019更新版本。 开发人员能够使用属性以编程方式控制 sameSite 属性的值 `HttpCookie.SameSite` 。 如果将 `SameSite` 属性设置为 Strict、宽松或 None，则会在网络上用来写入这些值 cookie 。 如果将其设置为 `(SameSiteMode)(-1)` "等于"，则表示网络中不应包含 sameSite 属性， cookie

[!code-csharp[](samesite/snippets/Privacy.cshtml.cs?name=snippet)]

.NET Core 3.0 和更高版本支持更新后的 SameSite 值，并向枚举添加额外的枚举值 `SameSiteMode.Unspecified` `SameSiteMode` 。
此新值指示不应向发送 sameSite cookie 。

::: moniker-end

::: moniker range="= aspnetcore-2.1"

## <a name="december-patch-behavior-changes"></a>12月修补程序行为更改

.NET Framework 和 .NET Core 2.1 的特定行为更改是 `SameSite` 属性解释值的方式 `None` 。 在修补程序的值为 `None` "根本不发出属性" 之前，修补后表示 "发出值为的属性" `None` 。 修补后，的 `SameSite` 值将 `(SameSiteMode)(-1)` 导致不发出属性。

Forms 身份验证和会话状态的默认 SameSite 值 cookie 从更改 `None` 为 `Lax` 。

::: moniker-end

## <a name="api-usage-with-samesite"></a>使用 SameSite 的 API 用法

[HttpContext 响应。 Cookies。](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) 将默认值追加到 `Unspecified` ，这意味着，不会将 SameSite 属性添加到中， cookie 并且客户端将使用其默认行为 (对新浏览器的宽松，而不能为旧浏览器) 。 下面的代码演示如何将 cookie SameSite 值更改为 `SameSiteMode.Lax` ：

[!code-csharp[](samesite/sample/Pages/Index.cshtml.cs?name=snippet)]

发出的所有 ASP.NET Core 组件都 cookie 用适用于方案的设置替代前面的默认值。 先前重写的默认值尚未更改。

| 组件 | cookie | 默认 |
| ------------- | ------------- |
| <xref:Microsoft.AspNetCore.Http.CookieBuilder> | <xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite> | `Unspecified` |
| <xref:Microsoft.AspNetCore.Http.HttpContext.Session>  | [SessionOptions.Cookie](xref:Microsoft.AspNetCore.Builder.SessionOptions.Cookie) |`Lax` |
| <xref:Microsoft.AspNetCore.Mvc.ViewFeatures.CookieTempDataProvider>  | [CookieTempDataProviderOptions.Cookie](xref:Microsoft.AspNetCore.Mvc.CookieTempDataProviderOptions.Cookie) | `Lax` |
| <xref:Microsoft.AspNetCore.Antiforgery.IAntiforgery> | [AntiforgeryOptions.Cookie](xref:Microsoft.AspNetCore.Antiforgery.AntiforgeryOptions.Cookie)| `Strict` |
| [Cookie 验证](xref:Microsoft.Extensions.DependencyInjection.CookieExtensions.AddCookie*) | [CookieAuthenticationOptions.Cookie](xref:Microsoft.AspNetCore.Builder.CookieAuthenticationOptions.CookieName) | `Lax` |
| <xref:Microsoft.Extensions.DependencyInjection.TwitterExtensions.AddTwitter*> | [CookieTwitterOptions](xref:Microsoft.AspNetCore.Authentication.Twitter.TwitterOptions.StateCookie) | `Lax`  |
| <xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationHandler`1> | [RemoteAuthenticationOptionsCookie](xref:Microsoft.AspNetCore.Authentication.RemoteAuthenticationOptions.CorrelationCookie)  | `None` |
| <xref:Microsoft.Extensions.DependencyInjection.OpenIdConnectExtensions.AddOpenIdConnect*> | [OpenIdConnectOptionsCookie](xref:Microsoft.AspNetCore.Authentication.OpenIdConnect.OpenIdConnectOptions.NonceCookie)| `None` |
| [HttpContext 响应。 Cookies. 追加](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*) | <xref:Microsoft.AspNetCore.Http.CookieOptions> | `Unspecified` |

::: moniker range=">= aspnetcore-3.1"

ASP.NET Core 3.1 和更高版本提供了以下 SameSite 支持：

* 重新定义要发出的的行为 `SameSiteMode.None``SameSite=None`
* 添加新值 `SameSiteMode.Unspecified` 以省略 SameSite 属性。
* 所有 cookie s api 默认为 `Unspecified` 。 某些使用的组件 cookie 将值设置为对其方案更为具体的值。 有关示例，请参阅上表。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

在 ASP.NET Core 3.0 及更高版本中，SameSite 默认值已更改，以避免与客户端默认值不一致发生冲突。 以下 Api 已将默认值从更改为， `SameSiteMode.Lax ` `-1` 以避免发出以下的 SameSite 属性 cookie ：

* <xref:Microsoft.AspNetCore.Http.CookieOptions> 与 HttpContext 一起使用 [。 Cookies. 追加](xref:Microsoft.AspNetCore.Http.IResponseCookies.Append*)
* <xref:Microsoft.AspNetCore.Http.CookieBuilder>  用作的工厂 `CookieOptions`
* [CookiePolicyOptions.MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)

::: moniker-end

## <a name="history-and-changes"></a>历史记录和更改

SameSite 支持在2.0 中第一次 ASP.NET Core 实现，使用 [2016 草案标准](https://tools.ietf.org/html/draft-west-first-party-cookies-07#section-4.1)。 2016标准已选择加入。 默认情况下，ASP.NET Core 选择加入 cookie `Lax` 。 在遇到身份验证的几个 [问题](https://github.com/aspnet/Announcements/issues/318) 后，大多数 SameSite 使用已 [禁用](https://github.com/aspnet/Announcements/issues/348)。

2019年11月发布了[修补程序](https://devblogs.microsoft.com/dotnet/net-core-November-2019/)，从2016标准版更新为2019标准。 [SameSite 规范的2019草案](https://github.com/aspnet/Announcements/issues/390)：

* **不** 向后兼容2016草案。 有关详细信息，请参阅本文档中的 [支持旧版浏览器](#sob) 。
* 指定 cookie `SameSite=Lax` 按默认值处理。
* 指定 cookie 显式断言以便 `SameSite=None` 启用跨站点传递的是应该标记为的 `Secure` 。 `None` 要选择退出的新项。
* 为 ASP.NET Core 2.1、2.2 和3.0 颁发的修补程序支持。 ASP.NET Core 3.1 具有附加的 SameSite 支持。
* 默认 [情况下，计划](https://chromestatus.com/feature/5088147346030592) 在 [2020 年2月](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)启用。 浏览器已开始在2019中移动到此标准。

## <a name="apis-impacted-by-the-change-from-the-2016-samesite-draft-standard-to-the-2019-draft-standard"></a>受从 2016 SameSite 草案标准到2019草案标准的更改影响的 Api

* [SameSiteMode](xref:Microsoft.AspNetCore.Http.SameSiteMode)
* [Cookie选项。 SameSite](xref:Microsoft.AspNetCore.Http.CookieOptions.SameSite)
* [CookieSameSite](xref:Microsoft.AspNetCore.Http.CookieBuilder.SameSite)
* [CookiePolicyOptions.MinimumSameSitePolicy](xref:Microsoft.AspNetCore.Builder.CookiePolicyOptions.MinimumSameSitePolicy)
* <xref:Microsoft.Net.Http.Headers.SameSiteMode?displayProperty=fullName>
* <xref:Microsoft.Net.Http.Headers.SetCookieHeaderValue.SameSite?displayProperty=fullName>

<a name="sob"></a>

## <a name="supporting-older-browsers"></a>支持旧版浏览器

2016 SameSite 标准规定，未知值必须被视为 `SameSite=Strict` 值。 从支持 2016 SameSite 标准的旧版浏览器访问的应用在收到值为的 SameSite 属性时可能会中断 `None` 。 如果 Web 应用要支持较旧的浏览器，则必须实现浏览器检测。 ASP.NET Core 不会实现浏览器检测，因为 User-Agents 值是高度易失性的，经常更改。 中的扩展点 <xref:Microsoft.AspNetCore.CookiePolicy> 允许插入 User-Agent 的特定逻辑。

在中 `Startup.Configure` ，添加在调用之前调用的代码 <xref:Microsoft.AspNetCore.Builder.CookiePolicyAppBuilderExtensions.UseCookiePolicy*> <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> 或 *任何* 写入的方法 cookie ：

[!code-csharp[](samesite/sample/Startup.cs?name=snippet5&highlight=18-19)]

在中 `Startup.ConfigureServices` ，添加类似于下面的代码：

::: moniker range="= aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet)]

::: moniker-end

::: moniker range="< aspnetcore-3.1"

[!code-csharp[](samesite/sample/Startup.cs?name=snippet)]

::: moniker-end

在前面的示例中， `MyUserAgentDetectionLib.DisallowsSameSiteNone` 是一个用户提供的库，用于检测用户代理是否不支持 SameSite `None` ：

[!code-csharp[](samesite/sample/Startup31.cs?name=snippet2)]

下面的代码演示了一个示例 `DisallowsSameSiteNone` 方法：

> [!WARNING]
> 以下代码仅用于演示：
> * 不应将其视为已完成。
> * 它不维护或不受支持。

[!code-csharp[](samesite/sample/Startup31.cs?name=snippetX)]

## <a name="test-apps-for-samesite-problems"></a>测试应用的 SameSite 问题

与远程站点（如通过第三方登录）交互的应用需要：

* 测试多个浏览器上的交互。
* 应用本文档所述的[ Cookie 策略浏览器检测和缓解措施](#sob)。

使用可选择新的 SameSite 行为的客户端版本测试 web 应用。 Chrome、Firefox 和 Chromium Edge 都具有可用于测试的新的可选功能标志。 应用应用 SameSite 修补程序后，请对其进行测试，使其与早期版本的客户端版本（尤其是 Safari） 有关详细信息，请参阅本文档中的 [支持旧版浏览器](#sob) 。

### <a name="test-with-chrome"></a>用 Chrome 测试

Chrome 78 + 提供了令人误解的结果，因为它具有临时的缓解措施。 Chrome 78 + 临时缓解允许 cookie 两分钟以内。 已启用适当测试标志的 Chrome 76 或77提供更准确的结果。 若要测试新的 SameSite 行为切换 `chrome://flags/#same-site-by-default-cookies` 为 **启用状态** 。 旧版本的 Chrome (75 及更) 低版本将报告为失败，并出现新 `None` 设置。 请参阅本文档中的 [支持旧版浏览器](#sob) 。

Google 不会使旧版 chrome 版本可用。 遵循 [下载 Chromium](https://www.chromium.org/getting-involved/download-chromium) 中的说明来测试旧版 Chrome。 不要从通过搜索旧版 chrome 提供的 **链接下载 chrome** 。

* [Chromium 76 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/664998/)
* [Chromium 74 Win64](https://commondatastorage.googleapis.com/chromium-browser-snapshots/index.html?prefix=Win_x64/638880/)

从 "未完成" 的版本开始 `80.0.3975.0` ，可以使用新标志来禁用宽松的 + 后续暂时缓解，以 `--enable-features=SameSiteDefaultChecksMethodRigorously` 允许在删除缓解功能的最终状态下测试站点和服务。 有关详细信息，请参阅 Chromium 项目 [SameSite Updates](https://www.chromium.org/updates/same-site)

### <a name="test-with-safari"></a>用 Safari 测试

Safari 12 严格实现了之前的草稿，如果新值在中，则会失败 `None` cookie 。 `None` 通过本文档中 [支持旧版浏览](#sob) 器的浏览器检测代码，避免了这种情况。 使用 MSAL、ADAL 或所使用的任何库，测试 Safari 12、Safari 13 和基于 WebKit 的 OS 样式登录。 问题取决于基础 OS 版本。 已知 OSX Mojave (10.14) 和 iOS 12 与新的 SameSite 行为存在兼容性问题。 将 OS 升级到 OSX Catalina (10.15) 或 iOS 13 修复了此问题。 Safari 当前没有用于测试新规范行为的选择标记。

### <a name="test-with-firefox"></a>用 Firefox 测试

可以在版本 68 + 上测试对新标准的 Firefox 支持，方法是在页面上选择 `about:config` 功能标志 `network.cookie.sameSite.laxByDefault` 。 以前版本的 Firefox 没有出现兼容性问题的报告。

### <a name="test-with-edge-browser"></a>通过 Edge 浏览器进行测试

Edge 支持旧的 SameSite 标准。 边缘版本44与新的标准没有任何已知的兼容性问题。

### <a name="test-with-edge-chromium"></a> (Chromium 的边缘测试) 

页面上设置了 SameSite 标志 `edge://flags/#same-site-by-default-cookies` 。 未发现边缘 Chromium 的兼容性问题。

### <a name="test-with-no-locelectron"></a>测试 Electron

版本的 Electron 包含 Chromium 的旧版本。 例如，团队使用的版本 Electron 是 Chromium 66，它展示了较旧的行为。 你必须使用产品的版本来执行你自己的兼容性测试 Electron 。 请参阅下一节中的 [支持旧版浏览器](#sob) 。

## <a name="additional-resources"></a>其他资源

* [Chromium 博客：开发人员：准备好新 SameSite = 无;安全 Cookie 设置](https://blog.chromium.org/2019/10/developers-get-ready-for-new.html)
* [cookie说明 SameSite](https://web.dev/samesite-cookies-explained/)
* [2019年11月修补程序](https://devblogs.microsoft.com/dotnet/net-core-November-2019/)

 ::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

| 示例               | 文档 |
| ----------------- | ------------ |
| [.NET Core MVC](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)  | <xref:security/samesite/mvc21> |
| [.NET Core Razor 页](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21RazorPages)  | <xref:security/samesite/rp21> |

::: moniker-end

 ::: moniker range=">= aspnetcore-3.0"

| 示例               | 文档 |
| ----------------- | ------------ |
| [.NET Core Razor 页](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore31RazorPages)  | <xref:security/samesite/rp31> |

::: moniker-end
