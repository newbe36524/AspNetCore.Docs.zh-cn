---
title: ASP.NET Core 中一般数据保护条例 (GDPR) 支持
author: rick-anderson
description: 了解如何访问 ASP.NET Core web 应用中的 GDPR 扩展点。
ms.author: riande
ms.custom: mvc
ms.date: 07/11/2019
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
uid: security/gdpr
ms.openlocfilehash: ec65a2c8362c15716bebd6b22f5639785ba74c98
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051000"
---
# <a name="eu-general-data-protection-regulation-gdpr-support-in-aspnet-core"></a>欧盟一般数据保护条例 (GDPR) 支持 ASP.NET Core

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core 提供 Api 和模板来帮助满足某些 [欧盟一般数据保护条例 (GDPR) ](https://ec.europa.eu/info/law/law-topic/data-protection/reform/what-does-general-data-protection-regulation-gdpr-govern_en) 要求：

::: moniker range=">= aspnetcore-3.0"

* 项目模板包括扩展点和用作存根标记，你可以将其替换为你的隐私和 cookie 使用策略。
* *Pages/私密. cshtml* 页面或 *Views/Home/私密* 视图提供了一个页面，用于详细描述您的站点的隐私策略。

若要启用默认 cookie 许可功能，如 ASP.NET Core 3.0 模板生成的应用中的 ASP.NET Core 2.2 模板中所示：

* 将添加 `using Microsoft.AspNetCore.Http` 到 using 指令列表。
* 将[ Cookie PolicyOptions](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyoptions)添加到 `Startup.ConfigureServices` ，并[使用 Cookie 策略](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyappbuilderextensions.usecookiepolicy) `Startup.Configure` 执行以下操作：

  [!code-csharp[Main](gdpr/sample/RP3.0/Startup.cs?name=snippet1&highlight=12-19,38)]

* 将 cookie 同意部分添加到 *_Layout 的 cshtml* 文件中：

  [!code-cshtml[Main](gdpr/sample/RP3.0/Pages/Shared/_Layout.cshtml?name=snippet&highlight=4)]

* 将 *\_ Cookie ConsentPartial* 文件添加到项目：

  [!code-cshtml[Main](gdpr/sample/RP3.0/Pages/Shared/_CookieConsentPartial.cshtml)]

* 若要了解同意功能，请选择本文的 ASP.NET Core 2.2 版本 cookie 。

::: moniker-end

::: moniker range="= aspnetcore-2.2"

* 项目模板包括扩展点和用作存根标记，你可以将其替换为你的隐私和 cookie 使用策略。
* cookie许可功能允许您 (和跟踪用户的) 同意，以存储个人信息。 如果用户未同意数据收集，并且应用已将 [CheckConsentNeeded](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyoptions.checkconsentneeded) 设置为，则不会将不 `true` 必要 cookie 的发送到浏览器。
* Cookie可以将 s 标记为必要。 cookie即使用户尚未同意并禁用跟踪，也会将重要的发送到浏览器。
* 禁用跟踪后[，TempData 和会话 cookie s](#tempdata)不起作用。
* " [ Identity 管理](#pd)" 页提供了一个链接，用于下载和删除用户数据。

该 [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/gdpr/sample) 允许你测试添加到 ASP.NET Core 2.1 模板的大多数 GDPR 扩展点和 api。 有关测试说明，请参阅 [自述](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/gdpr/sample) 文件。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/gdpr/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="aspnet-core-gdpr-support-in-template-generated-code"></a>模板生成的代码中的 ASP.NET Core GDPR 支持

Razor 用项目模板创建的页和 MVC 项目包含以下 GDPR 支持：

* [ Cookie PolicyOptions](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyoptions)和[Use Cookie 策略](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyappbuilderextensions.usecookiepolicy)在类中设置 `Startup` 。
* *\_ Cookie ConsentPartial* [分部视图](xref:mvc/views/tag-helpers/builtin-th/partial-tag-helper)。 此文件中包括 " **接受** " 按钮。 用户单击 " **接受** " 按钮时， cookie 会提供许可。
* *Pages/私密. cshtml* 页面或 *Views/Home/私密* 视图提供了一个页面，用于详细描述您的站点的隐私策略。 *\_ Cookie ConsentPartial* 文件生成指向隐私页面的链接。
* 对于通过单独用户帐户创建的应用，"管理" 页提供了下载和删除 [个人用户数据](#pd)的链接。

### <a name="no-loccookiepolicyoptions-and-useno-loccookiepolicy"></a>CookiePolicyOptions 和使用 Cookie 策略

[ Cookie PolicyOptions](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyoptions)在中进行初始化 `Startup.ConfigureServices` ：

[!code-csharp[Main](gdpr/sample/Startup.cs?name=snippet1&highlight=14-20)]

[使用 Cookie策略](/dotnet/api/microsoft.aspnetcore.builder.cookiepolicyappbuilderextensions.usecookiepolicy) 在中调用 `Startup.Configure` ：

[!code-csharp[](gdpr/sample/Startup.cs?name=snippet1&highlight=51)]

### <a name="_no-loccookieconsentpartialcshtml-partial-view"></a>\_CookieConsentPartial 分部视图

*\_ Cookie ConsentPartial* 分部视图：

[!code-cshtml[](gdpr/sample/RP2.2/Pages/Shared/_CookieConsentPartial.cshtml)]

此部分内容：

* 获取用户的跟踪状态。 如果应用配置为要求同意，则用户必须同意才能 cookie 跟踪。 如果需要同意，则 cookie 在由 *\_ 布局 cshtml* 文件创建的导航栏的顶部固定同意面板。
* 提供一个 HTML `<p>` 元素，用于汇总隐私和 cookie 使用策略。
* 提供指向隐私页面或视图的链接，您可以在其中详细说明网站的隐私策略。

## <a name="essential-no-loccookies"></a>基本 cookie s

如果 cookie 尚未提供许可，则仅将 cookie 标记为 "必需" 的标记发送到浏览器。 以下代码 cookie 非常重要：

[!code-csharp[Main](gdpr/sample/RP2.2/Pages/Cookie.cshtml.cs?name=snippet1&highlight=5)]

<a name="tempdata"></a>

### <a name="tempdata-provider-and-session-state-no-loccookies-arent-essential"></a>TempData 提供程序和会话状态 cookie 不必要

[TempData 提供程序](xref:fundamentals/app-state#tempdata) cookie 不是必需的。 如果禁用跟踪，则 TempData 提供程序不起作用。 若要在禁用跟踪时启用 TempData 提供程序，请在中将 TempData 标记 cookie 为 "重要" `Startup.ConfigureServices` 。

[!code-csharp[Main](gdpr/sample/RP2.2/Startup.cs?name=snippet1)]

[会话状态](xref:fundamentals/app-state) cookie不是必需的。 禁用跟踪后，会话状态不起作用。 以下代码使会话成为 cookie 必要的：

[!code-csharp[](gdpr/sample/RP2.2/Startup.cs?name=snippet2)]

<a name="pd"></a>

## <a name="personal-data"></a>个人数据

ASP.NET Core 通过单独用户帐户创建的应用包括下载和删除个人数据的代码。

选择用户名，然后选择 " **个人数据** "：

![管理个人数据页](gdpr/_static/pd.png)

说明：

* 若要生成 `Account/Manage` 代码，请[参阅 Identity 基架](xref:security/authentication/scaffold-identity)。
* " **删除** " 和 " **下载** " 链接仅作用于默认标识数据。 必须扩展用于创建自定义用户数据的应用，以删除/下载自定义用户数据。 有关详细信息，请参阅向[添加、下载和删除自定义用户 Identity 数据](xref:security/authentication/add-user-data)。
* Identity `AspNetUserTokens` 由于[外键](https://github.com/aspnet/Identity/blob/release/2.1/src/EF/IdentityUserContext.cs#L152)，通过级联删除行为删除用户时，将删除存储在数据库表中的用户的已保存令牌。
* [外部提供程序身份验证](xref:security/authentication/social/index)（如 Facebook 和 Google）在接受策略之前不可用 cookie 。

::: moniker-end

## <a name="encryption-at-rest"></a>静态加密

某些数据库和存储机制允许静态加密。 静态加密：

* 自动加密存储的数据。
* 对于访问数据的软件，不对其进行配置、编程或其他操作。
* 是最简单且最安全的选项。
* 允许数据库管理密钥和加密。

例如： 。

* Microsoft SQL 和 Azure SQL 提供 [透明数据加密](/sql/relational-databases/security/encryption/transparent-data-encryption) (TDE) 。
* [默认情况下，SQL Azure 加密数据库](https://azure.microsoft.com/updates/newly-created-azure-sql-databases-encrypted-by-default/)
* [默认情况下，加密 Azure blob、文件、表和队列存储](https://azure.microsoft.com/blog/announcing-default-encryption-for-azure-blobs-files-table-and-queue-storage/)。

对于不提供静态内置加密的数据库，您可以使用磁盘加密来提供相同的保护。 例如： 。

* [适用于 Windows Server 的 BitLocker](/windows/security/information-protection/bitlocker/bitlocker-how-to-deploy-on-windows-server)
* Linux：
  * [eCryptfs](https://launchpad.net/ecryptfs)
  * [EncFS](https://github.com/vgough/encfs)。

## <a name="additional-resources"></a>其他资源

* [Microsoft.com/GDPR](https://www.microsoft.com/trustcenter/Privacy/GDPR)
* [GDPR-在 ASP.NET Core 中添加 "撤消许可" 按钮](https://www.joeaudette.com/blog/2018/08/28/gdpr---adding-a-revoke-consent-button-in-aspnet-core)
