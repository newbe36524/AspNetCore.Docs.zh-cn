---
title: ASP.NET Core 安全性概述
author: rick-anderson
description: 了解 ASP.NET Core 中的身份验证、授权和安全基础知识。
ms.author: riande
ms.custom: mvc
ms.date: 10/24/2018
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
uid: security/index
ms.openlocfilehash: 3c86c66bebe8a5ce1c195ebf931193e7e2a73fef
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051624"
---
# <a name="overview-of-aspnet-core-security"></a>ASP.NET Core 安全性概述

通过 ASP.NET Core，开发者可轻松配置和管理其应用的安全性。 ASP.NET Core 的功能包括管理身份验证、授权、数据保护、HTTPS 强制、应用机密、XSRF/CSRF 防护及 CORS 管理。 通过这些安全功能，可以生成安全可靠的 ASP.NET Core 应用。

## <a name="aspnet-core-security-features"></a>ASP.NET Core 安全性功能

ASP.NET Core 提供许多用于保护应用安全的工具和库（包括内置的标识提供者），但你也可以使用第三方标识服务（如 Facebook、Twitter 和 LinkedIn）。 利用 ASP.NET Core 可以轻松管理应用机密，无需将机密信息暴露在代码中就可存储和使用它们。

## <a name="authentication-vs-authorization"></a>身份验证 vs授权

身份验证是这样一个过程：由用户提供凭据，然后将其与存储在操作系统、数据库、应用或资源中的凭据进行比较。 在授权过程中，如果凭据匹配，则用户身份验证成功，可执行已向其授权的操作。 授权指判断允许用户执行的操作的过程。

也可以将身份验证理解为进入空间（例如服务器、数据库、应用或资源）的一种方式，而授权是用户可以对该空间（服务器、数据库或应用）内的哪些对象执行哪些操作。

## <a name="common-vulnerabilities-in-software"></a>软件中的常见漏洞

ASP.NET Core 和 EF 提供维护应用安全、预防安全漏洞的功能。 下表中链接的文档详细介绍了在 Web 应用中避免最常见安全漏洞的技术：

* [跨站点脚本 (XSS) 攻击](xref:security/cross-site-scripting)
* [SQL 注入式攻击](/ef/core/querying/raw-sql)
* [跨站点请求伪造 (XSRF/CSRF) 攻击](xref:security/anti-request-forgery)
* [打开重定向攻击](xref:security/preventing-open-redirects)

还应注意其他漏洞。 有关详细信息，请参阅目录的“安全性和 Identity”部分中的其他文章  。
