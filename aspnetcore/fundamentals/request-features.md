---
title: ASP.NET Core 中的请求功能
author: ardalis
description: 了解与 ASP.NET Core 的接口中定义的 HTTP 请求和响应相关的 Web 服务器实现详细信息。
ms.author: riande
ms.custom: mvc
ms.date: 10/20/2020
no-loc:
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
uid: fundamentals/request-features
ms.openlocfilehash: 879b775ba2998ee803708ebf231b5fcd363b811c
ms.sourcegitcommit: b5ebaf42422205d212e3dade93fcefcf7f16db39
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/21/2020
ms.locfileid: "92326428"
---
# <a name="request-features-in-aspnet-core"></a>ASP.NET Core 中的请求功能

作者：[Steve Smith](https://ardalis.com/)

应用程序和中间件用于处理请求的 `HttpContext` API 具有一个抽象层，它将其称为“功能接口”。 每个功能接口都提供 `HttpContext` 公开的功能的粒度子集。 在处理请求时，服务器或中间件可以添加、修改、包装、替换甚至删除这些接口，而不必重新实现整个 `HttpContext`。 它们还可以在测试时用于模拟功能。

## <a name="feature-collections"></a>功能集合

`HttpContext` 的 <xref:Microsoft.AspNetCore.Http.HttpContext.Features> 属性提供对当前请求的功能接口集合的访问。 由于功能集合即使在请求的上下文中也是可变的，所以可使用中间件来修改集合并添加对其他功能的支持。 某些高级功能只能通过功能集合访问关联接口提供。

## <a name="feature-interfaces"></a>功能接口

ASP.NET Core 在 <xref:Microsoft.AspNetCore.Http.Features?displayProperty=fullName> 中定义了许多常见的 HTTP 功能接口，各种服务器和中间件共享这些接口来标识其支持的功能。 服务器和中间件还可以提供自己的具有附加功能的接口。

大多数功能接口都提供可选的点亮功能，如果该功能不存在，则它们的相关 `HttpContext` API 提供默认值。 以下内容中会根据需要指出几个接口，因为它们提供核心请求和响应功能，必须实现它们才能处理请求。

以下功能接口来自 <xref:Microsoft.AspNetCore.Http.Features?displayProperty=fullName>：

<xref:Microsoft.AspNetCore.Http.Features.IHttpRequestFeature>：定义 HTTP 请求的结构，包括协议、路径、查询字符串、标头和正文。 此功能是处理请求所必需的。

<xref:Microsoft.AspNetCore.Http.Features.IHttpResponseFeature>：定义 HTTP 响应的结构，包括状态代码、标头和响应的正文。 此功能是处理请求所必需的。

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpResponseBodyFeature>：定义使用 `Stream`、`PipeWriter` 或文件写出响应正文的不同方式。 此功能是处理请求所必需的。 这会替换 `IHttpResponseFeature.Body` 和 `IHttpSendFileFeature`。

::: moniker-end

<xref:Microsoft.AspNetCore.Http.Features.Authentication.IHttpAuthenticationFeature>：保存当前与请求关联的 <xref:System.Security.Claims.ClaimsPrincipal>。

<xref:Microsoft.AspNetCore.Http.Features.IFormFeature>：用于分析和缓存传入的 HTTP 和多部分表单提交。

::: moniker range=">= aspnetcore-2.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpBodyControlFeature>：用于控制请求或响应正文是否允许同步 IO 操作。

::: moniker-end
   
::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpBufferingFeature>：定义禁用请求和/或响应缓冲的方法。

::: moniker-end

<xref:Microsoft.AspNetCore.Http.Features.IHttpConnectionFeature>：为连接 ID 以及本地和远程地址和端口定义属性。

::: moniker range=">= aspnetcore-2.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>：控制当前请求允许的最大请求正文大小。

::: moniker-end

::: moniker range=">= aspnetcore-5.0"

`IHttpRequestBodyDetectionFeature`：指示请求是否可以有正文。

::: moniker-end

<xref:Microsoft.AspNetCore.Http.Features.IHttpRequestIdentifierFeature>：添加一个可以实现的属性来唯一标识请求。

<xref:Microsoft.AspNetCore.Http.Features.IHttpRequestLifetimeFeature>：定义支持中止连接，或者检测是否已提前终止请求（如由于客户端断开连接）。

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpRequestTrailersFeature>：提供对请求尾部标头（如果有）的访问权限。

<xref:Microsoft.AspNetCore.Http.Features.IHttpResetFeature>：用于为支持它们的协议（如 HTTP/2 或 HTTP/3）发送重置消息。

::: moniker-end

::: moniker range=">= aspnetcore-2.2"

<xref:Microsoft.AspNetCore.Http.Features.IHttpResponseTrailersFeature>：允许应用程序提供响应尾部标头（如支持）。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpSendFileFeature>：定义异步发送文件的方法。

::: moniker-end

<xref:Microsoft.AspNetCore.Http.Features.IHttpUpgradeFeature>：定义对 [HTTP 升级](https://tools.ietf.org/html/rfc2616.html#section-14.42)的支持，允许客户端指定在服务器需要切换协议时要使用的其他协议。

<xref:Microsoft.AspNetCore.Http.Features.IHttpWebSocketFeature>：定义支持 Web 套接字的 API。

::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.Features.IHttpsCompressionFeature>：控制是否应通过 HTTPS 连接使用响应压缩。

::: moniker-end

<xref:Microsoft.AspNetCore.Http.Features.IItemsFeature>：存储每个请求应用程序状态的 <xref:Microsoft.AspNetCore.Http.Features.IItemsFeature.Items> 集合。

<xref:Microsoft.AspNetCore.Http.Features.IQueryFeature>：分析并缓存查询字符串。
   
::: moniker range=">= aspnetcore-3.0"

<xref:Microsoft.AspNetCore.Http.Features.IRequestBodyPipeFeature>：将请求正文表示为 <xref:System.IO.Pipelines.PipeReader>。
 
::: moniker-end

<xref:Microsoft.AspNetCore.Http.Features.IRequestCookiesFeature>：分析并缓存请求 `Cookie` 标头值。

<xref:Microsoft.AspNetCore.Http.Features.IResponseCookiesFeature>：控制如何将响应 cookie 应用到 `Set-Cookie` 标头。

::: moniker range=">= aspnetcore-2.2"

<xref:Microsoft.AspNetCore.Http.Features.IServerVariablesFeature>：此功能可用于访问请求服务器变量，如 IIS 提供的变量。

::: moniker-end
   
<xref:Microsoft.AspNetCore.Http.Features.IServiceProvidersFeature>：提供对具有限定范围的请求服务的 <xref:System.IServiceProvider> 的访问。

<xref:Microsoft.AspNetCore.Http.Features.ISessionFeature>：为支持用户会话定义 `ISessionFactory` 和 <xref:Microsoft.AspNetCore.Http.ISession> 抽象。 `ISessionFeature` 是由 <xref:Microsoft.AspNetCore.Session.SessionMiddleware> 实现的（请参见 <xref:fundamentals/app-state>）。

<xref:Microsoft.AspNetCore.Http.Features.ITlsConnectionFeature>：定义用于检索客户端证书的 API。

<xref:Microsoft.AspNetCore.Http.Features.ITlsTokenBindingFeature>：定义使用 TLS 令牌绑定参数的方法。
   
::: moniker range=">= aspnetcore-2.2"
   
<xref:Microsoft.AspNetCore.Http.Features.ITrackingConsentFeature>：用于查询、授予和撤消有关存储与站点活动和功能相关的用户信息的用户同意。
   
::: moniker-end

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/servers/index>
* <xref:fundamentals/middleware/index>
