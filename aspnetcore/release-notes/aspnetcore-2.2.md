---
title: ASP.NET Core 2.2 的新增功能
author: rick-anderson
description: 了解 ASP.NET Core 2.2 的新增功能。
ms.author: riande
ms.custom: mvc
ms.date: 12/05/2019
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
uid: aspnetcore-2.2
ms.openlocfilehash: 085a26d508af5bdce506ab923a3256046b3894ab
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722483"
---
# <a name="whats-new-in-aspnet-core-22"></a>ASP.NET Core 2.2 的新增功能

本文重点介绍 ASP.NET Core 2.2 中最重要的更改，并提供相关文档的链接。

## <a name="openapi-analyzers--conventions"></a>开放 API 分析器和约定

开放 API（以前称为 Swagger）是一个与语言无关的规范，用于描述 REST API。 开放 API 生态系统具有一些工具，可用于发现、测试和生成使用该规范的客户端代码。 对在 ASP.NET Core MVC 中生成和直观呈现 OpenAPI 文档的支持是通过社区驱动项目（如 [NSwag](https://github.com/RicoSuter/NSwag) 和 [Swashbuckle.AspNetCore](https://github.com/domaindrivendev/Swashbuckle.AspNetCore)）提供。 对于创建开放 API 文档，ASP.NET Core 2.2 提供了改进的工具和运行时体验。

有关更多信息，请参见以下资源：

* <xref:web-api/advanced/analyzers>
* <xref:web-api/advanced/conventions>
* [ASP.NET Core 2.2.0-preview1：开放 API 分析器和约定](https://blogs.msdn.microsoft.com/webdev/2018/08/23/asp-net-core-2-20-preview1-open-api-analyzers-conventions/)

## <a name="problem-details-support"></a>问题详细信息支持

ASP.NET Core 2.1 引入了 `ProblemDetails`（基于用于通过 HTTP 响应传送错误详细信息的 [RFC 7807](https://tools.ietf.org/html/rfc7807) 规范）。 在 2.2 中，`ProblemDetails` 是针对具有 `ApiControllerAttribute` 特性的控制器中客户端错误代码的标准响应。 返回客户端错误状态代码 (4xx) 的 `IActionResult` 现在返回 `ProblemDetails` 正文。 结果还包括可以用于将使用请求日志的错误相关联的关联 ID。 对于客户端错误，`ProducesResponseType` 默认为使用 `ProblemDetails` 作为响应类型。 这会记录在使用 NSwag 或 Swashbuckle.AspNetCore 生成的开放 API/Swagger 输出中。

## <a name="endpoint-routing"></a>终结点路由

ASP.NET Core 2.2 使用新的终结点路由  系统来改进请求的调度。 更改包括新链接生成 API 成员和路由参数转换器。

有关更多信息，请参见以下资源：

* [2.2 中的终结点路由](https://blogs.msdn.microsoft.com/webdev/2018/08/27/asp-net-core-2-2-0-preview1-endpoint-routing/)
* [路由参数转换器](https://www.hanselman.com/blog/ASPNETCore22ParameterTransformersForCleanURLGenerationAndSlugsInRazorPagesOrMVC.aspx)（请参阅“路由”  部分）
* [基于 IRouter 与基于终结点的路由之间的差异](xref:fundamentals/routing?view=aspnetcore-2.2#differences-from-earlier-versions-of-routing)

## <a name="health-checks"></a>运行状况检查

通过新的运行状况检查服务可以更轻松地在需要运行状况检查的环境（如 Kubernetes）中使用 ASP.NET Core。 运行状况检查包括中间件和一组定义 `IHealthCheck` 抽象和服务的库。

运行状况检查由容器业务流程协调程序或负载均衡器用于快速确定系统是否正常响应请求。 容器业务流程协调程序可以通过停止滚动部署或重新启动容器来响应失败的运行状况检查。 负载均衡器可以通过路由流量以避开失败的服务实例，来响应运行状况检查。

运行状况检查由应用程序公开为监视系统所使用的 HTTP 终结点。 可以为各种实时监视方案和监视系统配置运行状况检查。 运行状况检查服务与 [BeatPulse 项目](https://github.com/Xabaril/BeatPulse)集成。 从而可以更轻松地添加为众多常用系统和依赖项添加检查。

有关详细信息，请参阅 [ASP.NET Core 中的运行状况检查](xref:host-and-deploy/health-checks)。

## <a name="http2-in-kestrel"></a>Kestrel 中的 HTTP/2

ASP.NET Core 2.2 添加了对 HTTP/2 的支持。

HTTP/2 是 HTTP 协议的主要修订版本。 HTTP/2 的重要功能包括：

* 标头压缩支持。
* 单个连接上的完全多路复用流。

尽管 HTTP/2 保留了 HTTP 的语义（如 HTTP 标头和方法），不过与 HTTP/1.x 相比，在客户端和服务器之间对此数据进行组帧和发送的方式发生了重大更改。

由于组帧中的这一更改，服务器和客户端需要协商所使用的协议版本。 应用层协议协商 (ALPN) 是一种 TLS 扩展，允许服务器和客户端协商用作其 TLS 握手一部分的协议版本。 虽然服务器与客户端之间可以事先了解协议，不过所有主要浏览器都支持 ALPN 作为建立 HTTP/2 连接的唯一方法。

有关详细信息，请参阅 [HTTP/2 支持](xref:fundamentals/servers/index?view=aspnetcore-2.2#http2-support)。

## <a name="kestrel-configuration"></a>Kestrel 配置

在早期版本的 ASP.NET Core 中，Kestrel 选项通过调用 `UseKestrel` 来配置。 在 2.2 中，Kestrel 选项通过在主机生成器上调用 `ConfigureKestrel` 来配置。 此更改为进程内承载解决了 `IServer` 注册顺序方面的问题。 有关更多信息，请参见以下资源：

* [缓解 UseIIS 冲突](https://github.com/aspnet/KestrelHttpServer/issues/2760)
* [使用 ConfigureKestrel 配置 Kestrel 服务器选项](xref:fundamentals/servers/kestrel?view=aspnetcore-2.2#how-to-use-kestrel-in-aspnet-core-apps)

## <a name="iis-in-process-hosting"></a>IIS 进程内承载

在早期版本的 ASP.NET Core 中，IIS 用作反向代理。 在 2.2 中，ASP.NET Core 模块可以启动 CoreCLR 并在 IIS 工作进程 (w3wp.exe  ) 内承载应用。 在使用 IIS 运行时，进程内承载可提供性能和诊断提升。

有关详细信息，请参阅 [IIS 进程内承载](xref:host-and-deploy/aspnet-core-module?view=aspnetcore-2.2#in-process-hosting-model)。

## <a name="no-locsignalr-java-client"></a>SignalR Java 客户端

ASP.NET Core 2.2 引入了适用于 SignalR 的 Java 客户端。 此客户端支持通过 Java 代码连接到 ASP.NET Core SignalR Server（包括 Android 应用）。

有关详细信息，请参阅 [ASP.NET Core SignalR Java 客户端](../signalr/java-client.md?view=aspnetcore-2.2)。

## <a name="cors-improvements"></a>CORS 改进

在早期版本的 ASP.NET Core 中，CORS 中间件允许发送 `Accept`、`Accept-Language`、`Content-Language` 和 `Origin` 标头（不考虑在 `CorsPolicy.Headers` 中配置的值）。 在 2.2 中，仅当在 `Access-Control-Request-Headers` 中发送的标头与 `WithHeaders` 中声明的标头完全匹配时，才能进行 CORS 中间件策略匹配。

有关详细信息，请参阅 [CORS 中间件](xref:security/cors?view=aspnetcore-2.2#set-the-allowed-request-headers)。

## <a name="response-compression"></a>响应压缩

ASP.NET Core 2.2 可以使用 [Brotli 压缩格式](https://tools.ietf.org/html/rfc7932)来压缩响应。

有关详细信息，请参阅[响应压缩中间件支持 Brotli 压缩](xref:performance/response-compression?view=aspnetcore-2.2#brotli-compression-provider)。

## <a name="project-templates"></a>项目模板

ASP.NET Core Web 项目模板已更新为 [Bootstrap 4](https://getbootstrap.com/docs/4.1/migration/) 和 [Angular 6](https://blog.angular.io/version-6-of-angular-now-available-cc56b0efa7a4)。 新的外观在视觉上更简单，可以更轻松地查看应用的重要结构。

![主页或索引页](~/tutorials/razor-pages/razor-pages-start/_static/home2.2.png)

## <a name="validation-performance"></a>验证性能

根据设计，MVC 的验证系统具有可扩展性和灵活性，让你能够基于每个请求确定应用于给定模型的验证程序。 这非常适用于创作复杂的验证提供程序。 但最常见的情况是，应用程序仅使用内置验证程序，不需要这种额外的灵活性。 内置验证程序包括 DataAnnotations（如 [Required] 和 [StringLength]）和 `IValidatableObject`。

在 ASP.NET Core 2.2 中，如果 MVC 确定给定模型关系图不需要验证，则可以绕过验证。 在验证无法具有或不具有任何验证程序的模型时，跳过验证可获得显著改进。 这包括诸如基元集合（如 `byte[]`、`string[]`、`Dictionary<string, string>` ）之类的对象，或没有许多验证程序的复杂对象关系图。

## <a name="http-client-performance"></a>HTTP 客户端性能

在 ASP.NET Core 2.2 中，通过减少连接池锁定争用提高了 `SocketsHttpHandler` 的性能。 对于进行大量传出 HTTP 请求的应用（如某些微服务体系结构），吞吐量得到了提高。 在负载下，`HttpClient` 吞吐量在 Linux 上可以提高多达 60%，在 Windows 上提高多达 20%。

有关详细信息，请参阅[实现此改进的拉取请求](https://github.com/dotnet/corefx/pull/32568)。

## <a name="additional-information"></a>其他信息

要获取完整的更改列表，请参阅 [ASP.NET Core 2.2 发行说明](https://github.com/dotnet/aspnetcore/releases/tag/2.2.0)。