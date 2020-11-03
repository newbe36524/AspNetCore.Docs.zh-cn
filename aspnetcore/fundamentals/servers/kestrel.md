---
title: ASP.NET Core 中的 Kestrel Web 服务器实现
author: rick-anderson
description: 了解跨平台 ASP.NET Core Web 服务器 Kestrel。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 05/04/2020
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
uid: fundamentals/servers/kestrel
ms.openlocfilehash: 44558a0f2fdc61eb860223658f5bef1d0117ba87
ms.sourcegitcommit: e519d95d17443abafba8f712ac168347b15c8b57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/02/2020
ms.locfileid: "91653931"
---
# <a name="kestrel-web-server-implementation-in-aspnet-core"></a>ASP.NET Core 中的 Kestrel Web 服务器实现

作者：[Tom Dykstra](https://github.com/tdykstra)、[Chris Ross](https://github.com/Tratcher) 和 [Stephen Halter](https://twitter.com/halter73)

::: moniker range=">= aspnetcore-3.0"

Kestrel 是一个跨平台的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 Kestrel 是 Web 服务器，默认包括在 ASP.NET Core 项目模板中。

Kestrel 支持以下方案：

* HTTPS
* 用于启用 [WebSocket](https://github.com/aspnet/websockets) 的不透明升级
* 用于获得 Nginx 高性能的 Unix 套接字
* HTTP/2（除 macOS&dagger; 以外）

macOS 的未来版本将支持 &dagger;HTTP/2。

.NET Core 支持的所有平台和版本均支持 Kestrel。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="http2-support"></a>HTTP/2 支持

如果满足以下基本要求，将为 ASP.NET Core 应用提供 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* 操作系统&dagger;
  * Windows Server 2016/Windows 10 或更高版本&Dagger;
  * 具有 OpenSSL 1.0.2 或更高版本的 Linux（例如，Ubuntu 16.04 或更高版本）
* 目标框架：.NET Core 2.2 或更高版本
* [应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接
* TLS 1.2 或更高版本的连接

macOS 的未来版本将支持 &dagger;HTTP/2。
&Dagger;Kestrel 在 Windows Server 2012 R2 和 Windows 8.1 上对 HTTP/2 的支持有限。 支持受限是因为可在这些操作系统上使用的受支持 TLS 密码套件列表有限。 可能需要使用椭圆曲线数字签名算法 (ECDSA) 生成的证书来保护 TLS 连接。

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/2`。

默认情况下，禁用 HTTP/2。 有关配置的详细信息，请参阅 [Kestrel 选项](#kestrel-options)和 [ListenOptions.Protocols](#listenoptionsprotocols) 部分。

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>何时结合使用 Kestrel 和反向代理

可以单独使用 Kestrel，也可以将其与反向代理服务器（如 [Internet Information Services (IIS)](https://www.iis.net/)、[Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)）结合使用。 反向代理服务器接收来自网络的 HTTP 请求，并将这些请求转发到 Kestrel。

Kestrel 用作边缘（面向 Internet）Web 服务器：

![Kestrel 直接与 Internet 通信，不使用反向代理服务器](kestrel/_static/kestrel-to-internet2.png)

Kestrel 用于反向代理配置：

![Kestrel 通过反向代理服务器（如 IIS、Nginx 或 Apache）间接与 Internet 进行通信](kestrel/_static/kestrel-to-internet.png)

无论配置是否使用反向代理服务器，都是受支持的托管配置。

在没有反向代理服务器的情况下用作边缘服务器的 Kestrel 不支持在多个进程间共享相同的 IP 和端口。 如果将 Kestrel 配置为侦听某个端口，Kestrel 会处理该端口的所有流量（无视请求的 `Host` 标头）。 可以共享端口的反向代理能在唯一的 IP 和端口上将请求转发至 Kestrel。

即使不需要反向代理服务器，使用反向代理服务器可能也是个不错的选择。

反向代理：

* 可以限制所承载的应用中的公开的公共外围应用。
* 提供额外的配置和防护层。
* 可以更好地与现有基础结构集成。
* 简化了负载均和和安全通信 (HTTPS) 配置。 仅反向代理服务器需要 X.509 证书，并且该服务器可使用普通 HTTP 在内部网络上与应用服务器通信。

> [!WARNING]
> 采用反向代理配置进行托管需要[主机筛选](#host-filtering)。

## <a name="kestrel-in-aspnet-core-apps"></a>ASP.NET Core 应用中的 Kestrel

默认情况下，ASP.NET Core 项目模板使用 Kestrel。 在“Program.cs”中，<xref:Microsoft.Extensions.Hosting.GenericHostBuilderExtensions.ConfigureWebHostDefaults*> 方法调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=8)]

有关生成主机的详细信息，请参阅 <xref:fundamentals/host/generic-host#set-up-a-host> 的“设置主机”和“默认生成器设置”部分 。

若要在调用 `ConfigureWebHostDefaults` 后提供其他配置，请使用 `ConfigureKestrel`：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(serverOptions =>
            {
                // Set properties and call methods on options
            })
            .UseStartup<Startup>();
        });
```

## <a name="kestrel-options"></a>Kestrel 选项

Kestrel Web 服务器具有约束配置选项，这些选项在面向 Internet 的部署中尤其有用。

对 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 类的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> 属性设置约束。 `Limits` 属性包含 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> 类的实例。

下面的示例使用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core> 命名空间。

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

在本文后面的示例中，Kestrel 选项是采用 C# 代码配置的。 还可以使用 [配置提供程序](xref:fundamentals/configuration/index)设置 Kestrel 选项。 例如，[文件配置提供程序](xref:fundamentals/configuration/index#file-configuration-provider)可以从 appsettings.json 或 appsettings.{Environment}.json 文件加载 Kestrel 配置 ：

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    },
    "DisableStringReuse": true
  }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 和 [终结点配置](#endpoint-configuration) 可以通过配置提供程序进行配置。 其余的 Kestrel 配置必须采用 C# 代码进行配置。

使用以下方法之一：

* 在 `Startup.ConfigureServices` 中配置 Kestrel：

  1. 将 `IConfiguration` 的实例注入到 `Startup` 类中。 下面的示例假定注入的配置已分配给 `Configuration` 属性。
  2. 在 `Startup.ConfigureServices` 中，将配置的 `Kestrel` 部分加载到 Kestrel 的配置中：

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* 构建主机时配置 Kestrel：

  在 Program.cs 中，将配置的 `Kestrel` 部分加载到 Kestrel 的配置中：

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IHostBuilder CreateHostBuilder(string[] args) =>
      Host.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .ConfigureWebHostDefaults(webBuilder =>
          {
              webBuilder.UseStartup<Startup>();
          });
  ```

上述两种方法适用于任何[配置提供程序](xref:fundamentals/configuration/index)。

### <a name="keep-alive-timeout"></a>保持活动状态超时

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

获取或设置[保持活动状态超时](https://tools.ietf.org/html/rfc7230#section-6.5)。 默认值为 2 分钟。

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=19-20)]

### <a name="maximum-client-connections"></a>客户端最大连接数

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

可使用以下代码为整个应用设置并发打开的最大 TCP 连接数：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

对于已从 HTTP 或 HTTPS 升级到另一个协议（例如，Websocket 请求）的连接，有一个单独的限制。 连接升级后，不会计入 `MaxConcurrentConnections` 限制。

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

默认情况下，最大连接数不受限制 (NULL)。

### <a name="maximum-request-body-size"></a>请求正文最大大小

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

默认的请求正文最大大小为 30,000,000 字节，大约 28.6 MB。

在 ASP.NET Core MVC 应用中替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

以下示例演示如何为每个请求上的应用配置约束：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

在中间件中替代特定请求的设置：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

如果应用在开始读取请求后配置请求限制，则会引发异常。 `IsReadOnly` 属性指示 `MaxRequestBodySize` 属性处于只读状态，意味已经无法再配置限制。

当应用在 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)后于[进程外](xref:host-and-deploy/iis/index#out-of-process-hosting-model)运行时，由于 IIS 已设置限制，因此禁用了 Kestrel 的请求正文大小限制。

### <a name="minimum-request-body-data-rate"></a>请求正文最小数据速率

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel 每秒检查一次数据是否以指定的速率（字节/秒）传入。 如果速率低于最小值，则连接超时。宽限期是 Kestrel 提供给客户端用于将其发送速率提升到最小值的时间量；在此期间不会检查速率。 宽限期有助于避免最初由于 TCP 慢启动而以较慢速率发送数据的连接中断。

默认的最小速率为 240 字节/秒，包含 5 秒的宽限期。

最小速率也适用于响应。 除了属性和接口名称中具有 `RequestBody` 或 `Response` 以外，用于设置请求限制和响应限制的代码相同。

以下示例演示如何在 Program.cs 中配置最小数据速率：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-11)]

在中间件中替代每个请求的最低速率限制：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

用于 HTTP/2 请求的 `HttpContext.Features` 中不存在前面示例中引用的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinResponseDataRateFeature>，因为鉴于协议支持请求多路复用，HTTP/2 通常不支持按请求修改速率限制。 不过，用于 HTTP/2 请求的 `HttpContext.Features` 中仍存在 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.Features.IHttpMinRequestBodyDataRateFeature>，因为仍可以通过将 `IHttpMinRequestBodyDataRateFeature.MinDataRate` 设置为 `null`（甚至对于 HTTP/2 请求），按请求完全禁用读取速率限制。 对于给定 HTTP/2 请求，尝试读取 `IHttpMinRequestBodyDataRateFeature.MinDataRate` 或尝试将它设置为除 `null` 以外的值会导致 `NotSupportedException` 抛出。

通过 `KestrelServerOptions.Limits` 配置的服务器范围的速率限制仍适用于 HTTP/1.x 和 HTTP/2 连接。

### <a name="request-headers-timeout"></a>请求标头超时

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

获取或设置服务器接收请求标头所花费的最大时间量。 默认值为 30 秒。

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=21-22)]

### <a name="maximum-streams-per-connection"></a>每个连接的最大流

`Http2.MaxStreamsPerConnection` 限制每个 HTTP/2 连接的并发请求流的数量。 拒绝过多的流。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
});
```

默认值为 100。

### <a name="header-table-size"></a>标题表大小

HPACK 解码器解压缩 HTTP/2 连接的 HTTP 标头。 `Http2.HeaderTableSize` 限制 HPACK 解码器使用的标头压缩表的大小。 该值以八位字节提供，且必须大于零 (0)。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.HeaderTableSize = 4096;
});
```

默认值为 4096。

### <a name="maximum-frame-size"></a>最大帧大小

`Http2.MaxFrameSize` 表示服务器接收或发送的 HTTP/2 连接帧有效负载的最大允许大小。 该值以八位字节提供，必须介于 2^14 (16,384) 和 2^24-1 (16,777,215) 之间。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxFrameSize = 16384;
});
```

默认值为 2^14 (16,384)。

### <a name="maximum-request-header-size"></a>最大请求标头大小

`Http2.MaxRequestHeaderFieldSize` 表示请求标头值的允许的最大大小（用八进制表示）。 此限制适用于名称和值的压缩和未压缩表示形式。 该值必须大于零 (0)。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
});
```

默认值为 8,192。

### <a name="initial-connection-window-size"></a>初始连接窗口大小

`Http2.InitialConnectionWindowSize` 表示服务器一次性缓存的最大请求主体数据大小（每次连接时在所有请求（流）中汇总，以字节为单位）。 请求也受 `Http2.InitialStreamWindowSize` 限制。 该值必须大于或等于 65,535，并小于 2^31 (2,147,483,648)。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
});
```

默认值为 128 KB (131,072)。

### <a name="initial-stream-window-size"></a>初始流窗口大小

`Http2.InitialStreamWindowSize` 表示服务器针对每个请求（流）的一次性缓存的最大请求主体数据大小（以字节为单位）。 请求也受 `Http2.InitialConnectionWindowSize` 限制。 该值必须大于或等于 65,535，并小于 2^31 (2,147,483,648)。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
});
```

默认值为 96 KB (98,304)。

### <a name="trailers"></a>预告片

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a>重置

[!INCLUDE[](~/includes/reset.md)]

### <a name="synchronous-io"></a>同步 I/O

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> 控制是否允许对请求和响应使用同步 I/O。 默认值为 `false`。

> [!WARNING]
> 大量阻止同步 I/O 的操作可能会导致线程池资源不足，进而导致应用无响应。 仅在使用不支持异步 I/O 的库时，才启用 `AllowSynchronousIO`。

以下示例会启用同步 I/O：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

有关其他 Kestrel 选项和限制的信息，请参阅：

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a>终结点配置

默认情况下，ASP.NET Core 绑定到：

* `http://localhost:5000`
* `https://localhost:5001`（存在本地开发证书时）

使用以下内容指定 URL：

* `ASPNETCORE_URLS` 环境变量。
* `--urls` 命令行参数。
* `urls` 主机配置键。
* `UseUrls` 扩展方法。

采用这些方法提供的值可以是一个或多个 HTTP 和 HTTPS 终结点（如果默认证书可用，则为 HTTPS）。 将值配置为以分号分隔的列表（例如 `"Urls": "http://localhost:8000;http://localhost:8001"`）。

有关这些方法的详细信息，请参阅[服务器 URL](xref:fundamentals/host/web-host#server-urls) 和[重写配置](xref:fundamentals/host/web-host#override-configuration)。

关于开发证书的创建：

* 安装了 [.NET Core SDK](/dotnet/core/sdk) 时。
* [dev-certs tool](xref:aspnetcore-2.1#https) 用于创建证书。

某些浏览器需要授予显式权限才能信任本地开发证书。

项目模板将应用配置为默认情况下在 HTTPS 上运行，并包括 [HTTPS 重定向和 HSTS 支持](xref:security/enforcing-ssl)。

调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 上的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 或 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 方法以配置 URL 前缀和 Kestrel 的端口。

`UseUrls`、`--urls` 命令行参数、`urls` 主机配置键以及 `ASPNETCORE_URLS` 环境变量也有用，但具有本节后面注明的限制（必须要有可用于 HTTPS 终结点配置的默认证书）。

`KestrelServerOptions` 配置：

### <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults(Action\<ListenOptions>)

指定一个为每个指定的终结点运行的配置 `Action`。 多次调用 `ConfigureEndpointDefaults`，用最新指定的 `Action` 替换之前的 `Action`。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });
});
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)

指定一个为每个 HTTPS 终结点运行的配置 `Action`。 多次调用 `ConfigureHttpsDefaults`，用最新指定的 `Action` 替换之前的 `Action`。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        // certificate is an X509Certificate2
        listenOptions.ServerCertificate = certificate;
    });
});
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。

### <a name="configureiconfiguration"></a>Configure(IConfiguration)

创建配置加载程序，用于设置将 <xref:Microsoft.Extensions.Configuration.IConfiguration> 作为输入的 Kestrel。 配置必须针对 Kestrel 的配置节。

### <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

将 Kestrel 配置为使用 HTTPS。

`ListenOptions.UseHttps` 扩展：

* `UseHttps`：将 Kestrel 配置为使用 HTTPS，采用默认证书。 如果没有配置默认证书，则会引发异常。
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

`ListenOptions.UseHttps` 参数：

* `filename` 是证书文件的路径和文件名，关联包含应用内容文件的目录。
* `password` 是访问 X.509 证书数据所需的密码。
* `configureOptions` 是配置 `HttpsConnectionAdapterOptions` 的 `Action`。 返回 `ListenOptions`。
* `storeName` 是从中加载证书的证书存储。
* `subject` 是证书的主题名称。
* `allowInvalid` 指示是否存在需要留意的无效证书，例如自签名证书。
* `location` 是从中加载证书的存储位置。
* `serverCertificate` 是 X.509 证书。

在生产中，必须显式配置 HTTPS。 至少必须提供默认证书。

下面要描述的支持的配置：

* 无配置
* 从配置中替换默认证书
* 更改代码中的默认值

*无配置*

Kestrel 在 `http://localhost:5000` 和 `https://localhost:5001` 上进行侦听（如果默认证书可用）。

<a name="configuration"></a>

*从配置中替换默认证书*

`CreateDefaultBuilder` 在默认情况下调用 `Configure(context.Configuration.GetSection("Kestrel"))` 来加载 Kestrel 配置。 Kestrel 可以使用默认 HTTPS 应用设置配置架构。 从磁盘上的文件或从证书存储中配置多个终结点，包括要使用的 URL 和证书。

在以下 appsettings.json 示例中：

* 将 AllowInvalid 设置为 `true`，从而允许使用无效证书（例如自签名证书）。
* 任何未指定证书的 HTTPS 终结点（下例中的 HttpsDefaultCert）会回退至在 Certificates > Default 下定义的证书或开发证书  。

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },
      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },
      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },
      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },
      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

此外还可以使用任何证书节点的 Path 和 Password，采用证书存储字段指定证书 。 例如，可将 Certificates > Default 证书指定为 ：

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

架构的注意事项：

* 终结点的名称不区分大小写。 例如，`HTTPS` 和 `Https` 都是有效的。
* 每个终结点都要具备 `Url` 参数。 此参数的格式和顶层 `Urls` 配置参数一样，只不过它只能有单个值。
* 这些终结点不会添加进顶层 `Urls` 配置中定义的终结点，而是替换它们。 通过 `Listen` 在代码中定义的终结点与在配置节中定义的终结点相累积。
* `Certificate` 部分是可选的。 如果为指定 `Certificate` 部分，则使用在之前的方案中定义的默认值。 如果没有可用的默认值，服务器会引发异常且无法启动。
* `Certificate` 支持 Path&ndash;Password 和 Subject&ndash;Store 证书   。
* 只要不会导致端口冲突，就能以这种方式定义任何数量的终结点。
* `options.Configure(context.Configuration.GetSection("{SECTION}"))` 通过 `.Endpoint(string name, listenOptions => { })` 方法返回 `KestrelConfigurationLoader`，可以用于补充已配置的终结点设置：

```csharp
webBuilder.UseKestrel((context, serverOptions) =>
{
    serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
        .Endpoint("HTTPS", listenOptions =>
        {
            listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
        });
});
```

可以直接访问 `KestrelServerOptions.ConfigurationLoader` 以继续迭代现有加载程序，例如由 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 提供的加载程序。

* 每个终结点的配置节都可用于 `Endpoint` 方法中的选项，以便读取自定义设置。
* 通过另一节再次调用 `options.Configure(context.Configuration.GetSection("{SECTION}"))` 可能加载多个配置。 只使用最新配置，除非之前的实例上显式调用了 `Load`。 元包不会调用 `Load`，所以可能会替换它的默认配置节。
* `KestrelConfigurationLoader` 从 `KestrelServerOptions` 将 API 的 `Listen` 簇反射为 `Endpoint` 重载，因此可在同样的位置配置代码和配置终结点。 这些重载不使用名称，且只使用配置中的默认设置。

*更改代码中的默认值*

可以使用 `ConfigureEndpointDefaults` 和 `ConfigureHttpsDefaults` 更改 `ListenOptions` 和 `HttpsConnectionAdapterOptions` 的默认设置，包括重写之前的方案指定的默认证书。 需要在配置任何终结点之前调用 `ConfigureEndpointDefaults` 和 `ConfigureHttpsDefaults`。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureEndpointDefaults(listenOptions =>
    {
        // Configure endpoint defaults
    });

    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.SslProtocols = SslProtocols.Tls12;
    });
});
```

*SNI 的 Kestrel 支持*

[服务器名称指示 (SNI)](https://tools.ietf.org/html/rfc6066#section-3) 可用于承载相同 IP 地址和端口上的多个域。 为了运行 SNI，客户端在 TLS 握手过程中将进行安全会话的主机名发送至服务器，从而让服务器可以提供正确的证书。 在 TLS 握手后的安全会话期间，客户端将服务器提供的证书用于与服务器进行加密通信。

Kestrel 通过 `ServerCertificateSelector` 回调支持 SNI。 每次连接调用一次回调，从而允许应用检查主机名并选择合适的证书。

SNI 支持要求：

* 在目标框架 `netcoreapp2.1` 或更高版本上运行。 在 `net461` 或最高版本上，将调用回调，但是 `name` 始终为 `null`。 如果客户端未在 TLS 握手过程中提供主机名参数，则 `name` 也为 `null`。
* 所有网站在相同的 Kestrel 实例上运行。 Kestrel 在无反向代理时不支持跨多个实例共享一个 IP 地址和端口。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ListenAnyIP(5005, listenOptions =>
    {
        listenOptions.UseHttps(httpsOptions =>
        {
            var localhostCert = CertificateLoader.LoadFromStoreCert(
                "localhost", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var exampleCert = CertificateLoader.LoadFromStoreCert(
                "example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var subExampleCert = CertificateLoader.LoadFromStoreCert(
                "sub.example.com", "My", StoreLocation.CurrentUser,
                allowInvalid: true);
            var certs = new Dictionary<string, X509Certificate2>(
                StringComparer.OrdinalIgnoreCase);
            certs["localhost"] = localhostCert;
            certs["example.com"] = exampleCert;
            certs["sub.example.com"] = subExampleCert;

            httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
            {
                if (name != null && certs.TryGetValue(name, out var cert))
                {
                    return cert;
                }

                return exampleCert;
            };
        });
    });
});
```

### <a name="connection-logging"></a>连接日志记录

调用 <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> 以发出用于进行连接上的字节级别通信的调试级别日志。 连接日志记录有助于排查低级通信中的问题，例如在 TLS 加密期间和代理后。 如果 `UseConnectionLogging` 放置在 `UseHttps` 之前，则会记录加密的流量。 如果 `UseConnectionLogging` 放置于 `UseHttps` 之后，则会记录解密的流量。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a>绑定到 TCP 套接字

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 方法绑定至 TCP 套接字，且 options lambda 允许 X.509 证书配置：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=12-18)]

示例使用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions> 为终结点配置 HTTPS。 可使用相同 API 为特定终结点配置其他 Kestrel 设置。

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a>绑定到 Unix 套接字

可通过 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 侦听 Unix 套接字以提高 Nginx 的性能，如以下示例所示：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* 在 Nginx 配置文件中，将 `server` > `location` > `proxy_pass` 条目设置为 `http://unix:/tmp/{KESTREL SOCKET}:/;`。 `{KESTREL SOCKET}` 是提供给 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 的套接字的名称（例如，上述示例中的 `kestrel-test.sock`）。
* 确保套接字可由 Nginx （例如 `chmod go+w /tmp/kestrel-test.sock`）进行写入。

### <a name="port-0"></a>端口 0

如果指定端口号 `0`，Kestrel 将动态绑定到可用端口。 以下示例演示如何确定 Kestrel 在运行时实际绑定到的端口：

[!code-csharp[](kestrel/samples/3.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

在应用运行时，控制台窗口输出指示可用于访问应用的动态端口：

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a>限制

使用以下方法配置终结点：

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* `--urls` 命令行参数
* `urls` 主机配置键
* `ASPNETCORE_URLS` 环境变量

若要将代码用于 Kestrel 以外的服务器，这些方法非常有用。 不过，请注意以下限制：

* HTTPS 无法与这些方法结合使用，除非在 HTTPS 终结点配置中提供了默认证书（例如，使用 `KestrelServerOptions` 配置或配置文件，如本主题前面的部分所示）。
* 如果同时使用 `Listen` 和 `UseUrls` 方法，`Listen` 终结点将覆盖 `UseUrls` 终结点。

### <a name="iis-endpoint-configuration"></a>IIS 终结点配置

使用 IIS 时，由 `Listen` 或 `UseUrls` 设置用于 IIS 覆盖绑定的 URL 绑定。 有关详细信息，请参阅 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)主题。

### <a name="listenoptionsprotocols"></a>ListenOptions.Protocols

`Protocols` 属性建立在连接终结点上或为服务器启用的 HTTP 协议（`HttpProtocols`）。 从 `HttpProtocols` 枚举向 `Protocols` 属性赋值。

| `HttpProtocols` 枚举值 | 允许的连接协议 |
| -------------------------- | ----------------------------- |
| `Http1`                    | 仅 HTTP/1.1。 可以在具有 TLS 或没有 TLS 的情况下使用。 |
| `Http2`                    | 仅 HTTP/2。 仅当客户端支持[先验知识模式](https://tools.ietf.org/html/rfc7540#section-3.4)时，才可以在没有 TLS 的情况下使用。 |
| `Http1AndHttp2`            | HTTP/1.1 和 HTTP/2。 HTTP/2 要求客户端在 TLS [应用层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 握手过程中选择 HTTP/2；否则，连接默认为 HTTP/1.1。 |

任何终结点的默认 `ListenOptions.Protocols` 值都为 `HttpProtocols.Http1AndHttp2`。

HTTP/2 的 TLS 限制：

* TLS 版本 1.2 或更高版本
* 重新协商已禁用
* 压缩已禁用
* 最小的临时密钥交换大小：
  * 椭圆曲线 Diffie-Hellman (ECDHE) &lbrack;[RFC4492](https://www.ietf.org/rfc/rfc4492.txt)&rbrack;：最小 224 位
  * 有限字段 Diffie-Hellman (DHE) &lbrack;`TLS12`&rbrack;：最小 2048 位
* 不禁止密码套件。 

默认情况下，支持具有 P-256 椭圆曲线 &lbrack;`FIPS186`&rbrack; 的 `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` &lbrack;`TLS-ECDHE`&rbrack;。

以下示例允许端口 8000 上的 HTTP/1.1 和 HTTP/2 连接。 TLS 使用提供的证书来保护连接：

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

使用连接中间件，针对特定密码的每个连接筛选 TLS 握手（如有必要）。

下面的示例针对应用不支持的任何密码算法引发 <xref:System.NotSupportedException>。 或者，定义 [ITlsHandshakeFeature.CipherAlgorithm](xref:Microsoft.AspNetCore.Connections.Features.ITlsHandshakeFeature.CipherAlgorithm) 并将其与可接受的密码套件列表进行比较。

没有哪种加密是使用 [CipherAlgorithmType.Null](xref:System.Security.Authentication.CipherAlgorithmType) 密码算法。

```csharp
// using System.Net;
// using Microsoft.AspNetCore.Connections;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.UseTlsFilter();
    });
});
```

```csharp
using System;
using System.Security.Authentication;
using Microsoft.AspNetCore.Connections.Features;

namespace Microsoft.AspNetCore.Connections
{
    public static class TlsFilterConnectionMiddlewareExtensions
    {
        public static IConnectionBuilder UseTlsFilter(
            this IConnectionBuilder builder)
        {
            return builder.Use((connection, next) =>
            {
                var tlsFeature = connection.Features.Get<ITlsHandshakeFeature>();

                if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
                {
                    throw new NotSupportedException("Prohibited cipher: " +
                        tlsFeature.CipherAlgorithm);
                }

                return next();
            });
        }
    }
}
```

连接筛选也可以通过 <xref:Microsoft.AspNetCore.Connections.IConnectionBuilder> lambda 进行配置：

```csharp
// using System;
// using System.Net;
// using System.Security.Authentication;
// using Microsoft.AspNetCore.Connections;
// using Microsoft.AspNetCore.Connections.Features;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.Use((context, next) =>
        {
            var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

            if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
            {
                throw new NotSupportedException(
                    $"Prohibited cipher: {tlsFeature.CipherAlgorithm}");
            }

            return next();
        });
    });
});
```

在 Linux 上，<xref:System.Net.Security.CipherSuitesPolicy> 可用于针对每个连接筛选 TLS 握手：

```csharp
// using System.Net.Security;
// using Microsoft.AspNetCore.Hosting;
// using Microsoft.AspNetCore.Server.Kestrel.Core;
// using Microsoft.Extensions.DependencyInjection;
// using Microsoft.Extensions.Hosting;

webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.ConfigureHttpsDefaults(listenOptions =>
    {
        listenOptions.OnAuthenticate = (context, sslOptions) =>
        {
            sslOptions.CipherSuitesPolicy = new CipherSuitesPolicy(
                new[]
                {
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256,
                    TlsCipherSuite.TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,
                    // ...
                });
        };
    });
});
```

*从配置中设置协议*

`CreateDefaultBuilder` 在默认情况下调用 `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` 来加载 Kestrel 配置。

以下 appsettings.json 示例将 HTTP/1.1 建立为所有终结点的默认连接协议：

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1"
    }
  }
}
```

以下 appsettings.json 示例将 HTTP/1.1 建立为所有指定终结点的连接协议：

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1"
      }
    }
  }
}
```

代码中指定的协议覆盖了由配置设置的值。

## <a name="transport-configuration"></a>传输配置

对于需要使用 Libuv (<xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>) 的项目：

* 将用于 [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) 包的依赖项添加到应用的项目文件：

   ```xml
   <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                     Version="{VERSION}" />
   ```

* 调用 `IWebHostBuilder` 上的 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>：

   ```csharp
   public class Program
   {
       public static void Main(string[] args)
       {
           CreateHostBuilder(args).Build().Run();
       }

       public static IHostBuilder CreateHostBuilder(string[] args) =>
           Host.CreateDefaultBuilder(args)
               .ConfigureWebHostDefaults(webBuilder =>
               {
                   webBuilder.UseLibuv();
                   webBuilder.UseStartup<Startup>();
               });
   }
   ```

### <a name="url-prefixes"></a>URL 前缀

如果使用 `UseUrls`、`--urls` 命令行参数、`urls` 主机配置键或 `ASPNETCORE_URLS` 环境变量，URL 前缀可采用以下任意格式。

仅 HTTP URL 前缀是有效的。 使用 `UseUrls` 配置 URL 绑定时，Kestrel 不支持 HTTPS。

* 包含端口号的 IPv4 地址

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` 是一种绑定到所有 IPv4 地址的特殊情况。

* 包含端口号的 IPv6 地址

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` 是 IPv4 `0.0.0.0` 的 IPv6 等效项。

* 包含端口号的主机名

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  主机名、`*`和 `+` 并不特殊。 没有识别为有效 IP 地址或 `localhost` 的任何内容都将绑定到所有 IPv4 和 IPv6 IP。 若要将不同主机名绑定到相同端口上的不同 ASP.NET Core 应用，请使用 [HTTP.sys](xref:fundamentals/servers/httpsys) 或 IIS、Nginx 或 Apache 等反向代理服务器。

  > [!WARNING]
  > 采用反向代理配置进行托管需要[主机筛选](#host-filtering)。

* 包含端口号的主机 `localhost` 名称或包含端口号的环回 IP

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  指定 `localhost` 后，Kestrel 将尝试绑定到 IPv4 和 IPv6 环回接口。 如果其他服务正在任一环回接口上使用请求的端口，则 Kestrel 将无法启动。 如果任一环回接口出于任何其他原因（通常是因为 IPv6 不受支持）而不可用，则 Kestrel 将记录一个警告。

## <a name="host-filtering"></a>主机筛选

尽管 Kestrel 支持基于前缀的配置（例如 `http://example.com:5000`），但 Kestrel 在很大程度上会忽略主机名。 主机 `localhost` 是一个特殊情况，用于绑定至环回地址。 除了显式 IP 地址以外的所有主机都绑定至所有公共 IP 地址。 不验证 `Host` 标头。

解决方法是，使用主机筛选中间件。 主机筛选中间件由 [Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) 包提供，此包为 ASP.NET Core 应用隐式提供。 由调用 <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*> 的 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 添加中间件：

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

默认情况下，主机筛选中间件处于禁用状态。 要启用该中间件，请在 appsettings.json/appsettings.\<EnvironmentName>.json 中定义一个 `AllowedHosts` 键。 此值是以分号分隔的不带端口号的主机名列表：

appsettings.json：

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> [转接头中间件](xref:host-and-deploy/proxy-load-balancer) 同样包含 <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> 选项。 转接头中间件和主机筛选中间件具有适合不同方案的相似功能。 如果未保留 `Host` 标头，并且使用反向代理服务器或负载均衡器转接请求，则使用转接头中间件设置 `AllowedHosts` 比较合适。 将 Kestrel 用作面向公众的边缘服务器或直接转接 `Host` 标头时，使用主机筛选中间件设置 `AllowedHosts` 比较合适。
>
> 有关转接头中间件的详细信息，请参阅 <xref:host-and-deploy/proxy-load-balancer>。

::: moniker-end

::: moniker range="= aspnetcore-2.2"

Kestrel 是一个跨平台的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 Kestrel 是 Web 服务器，默认包括在 ASP.NET Core 项目模板中。

Kestrel 支持以下方案：

* HTTPS
* 用于启用 [WebSocket](https://github.com/aspnet/websockets) 的不透明升级
* 用于获得 Nginx 高性能的 Unix 套接字
* HTTP/2（除 macOS&dagger; 以外）

macOS 的未来版本将支持 &dagger;HTTP/2。

.NET Core 支持的所有平台和版本均支持 Kestrel。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="http2-support"></a>HTTP/2 支持

如果满足以下基本要求，将为 ASP.NET Core 应用提供 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* 操作系统&dagger;
  * Windows Server 2016/Windows 10 或更高版本&Dagger;
  * 具有 OpenSSL 1.0.2 或更高版本的 Linux（例如，Ubuntu 16.04 或更高版本）
* 目标框架：.NET Core 2.2 或更高版本
* [应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接
* TLS 1.2 或更高版本的连接

macOS 的未来版本将支持 &dagger;HTTP/2。
&Dagger;Kestrel 在 Windows Server 2012 R2 和 Windows 8.1 上对 HTTP/2 的支持有限。 支持受限是因为可在这些操作系统上使用的受支持 TLS 密码套件列表有限。 可能需要使用椭圆曲线数字签名算法 (ECDSA) 生成的证书来保护 TLS 连接。

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/2`。

默认情况下，禁用 HTTP/2。 有关配置的详细信息，请参阅 [Kestrel 选项](#kestrel-options)和 [ListenOptions.Protocols](#listenoptionsprotocols) 部分。

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>何时结合使用 Kestrel 和反向代理

可以单独使用 Kestrel，也可以将其与反向代理服务器（如 [Internet Information Services (IIS)](https://www.iis.net/)、[Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)）结合使用。 反向代理服务器接收来自网络的 HTTP 请求，并将这些请求转发到 Kestrel。

Kestrel 用作边缘（面向 Internet）Web 服务器：

![Kestrel 直接与 Internet 通信，不使用反向代理服务器](kestrel/_static/kestrel-to-internet2.png)

Kestrel 用于反向代理配置：

![Kestrel 通过反向代理服务器（如 IIS、Nginx 或 Apache）间接与 Internet 进行通信](kestrel/_static/kestrel-to-internet.png)

无论配置是否使用反向代理服务器，都是受支持的托管配置。

在没有反向代理服务器的情况下用作边缘服务器的 Kestrel 不支持在多个进程间共享相同的 IP 和端口。 如果将 Kestrel 配置为侦听某个端口，Kestrel 会处理该端口的所有流量（无视请求的 `Host` 标头）。 可以共享端口的反向代理能在唯一的 IP 和端口上将请求转发至 Kestrel。

即使不需要反向代理服务器，使用反向代理服务器可能也是个不错的选择。

反向代理：

* 可以限制所承载的应用中的公开的公共外围应用。
* 提供额外的配置和防护层。
* 可以更好地与现有基础结构集成。
* 简化了负载均和和安全通信 (HTTPS) 配置。 仅反向代理服务器需要 X.509 证书，并且该服务器可使用普通 HTTP 在内部网络上与应用服务器通信。

> [!WARNING]
> 采用反向代理配置进行托管需要[主机筛选](#host-filtering)。

## <a name="how-to-use-kestrel-in-aspnet-core-apps"></a>如何在 ASP.NET Core 应用中使用 Kestrel

[Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)中包括 [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) 包。

默认情况下，ASP.NET Core 项目模板使用 Kestrel。 在 Program.cs 中，模板代码调用 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>，后者在后台调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>。

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_DefaultBuilder&highlight=7)]

有关 `CreateDefaultBuilder` 和生成主机的详细信息，请参阅 <xref:fundamentals/host/web-host#set-up-a-host> 的“设置主机”部分。

若要在调用 `CreateDefaultBuilder` 后提供其他配置，请使用 `ConfigureKestrel`：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            // Set properties and call methods on serverOptions
        });
```

如果应用未调用 `CreateDefaultBuilder` 来设置主机，请在调用 `ConfigureKestrel` 之前先调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>：

```csharp
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseKestrel()
        .UseIISIntegration()
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            // Set properties and call methods on serverOptions
        })
        .Build();

    host.Run();
}
```

## <a name="kestrel-options"></a>Kestrel 选项

Kestrel Web 服务器具有约束配置选项，这些选项在面向 Internet 的部署中尤其有用。

对 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 类的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> 属性设置约束。 `Limits` 属性包含 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> 类的实例。

下面的示例使用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core> 命名空间。

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

Kestrel 选项（已在以下示例的 C# 代码中配置）也可以使用[配置提供程序](xref:fundamentals/configuration/index)进行设置。 例如，文件配置提供程序可以从 appsettings.json 或 appsettings.{Environment}.json 文件加载 Kestrel 配置 ：

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    }
  }
}
```

使用以下方法之一：

* 在 `Startup.ConfigureServices` 中配置 Kestrel：

  1. 将 `IConfiguration` 的实例注入到 `Startup` 类中。 下面的示例假定注入的配置已分配给 `Configuration` 属性。
  2. 在 `Startup.ConfigureServices` 中，将配置的 `Kestrel` 部分加载到 Kestrel 的配置中：

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* 构建主机时配置 Kestrel：

  在 Program.cs 中，将配置的 `Kestrel` 部分加载到 Kestrel 的配置中：

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .UseStartup<Startup>();
  ```

上述两种方法适用于任何[配置提供程序](xref:fundamentals/configuration/index)。

### <a name="keep-alive-timeout"></a>保持活动状态超时

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

获取或设置[保持活动状态超时](https://tools.ietf.org/html/rfc7230#section-6.5)。 默认值为 2 分钟。

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=15)]

### <a name="maximum-client-connections"></a>客户端最大连接数

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

可使用以下代码为整个应用设置并发打开的最大 TCP 连接数：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=3)]

对于已从 HTTP 或 HTTPS 升级到另一个协议（例如，Websocket 请求）的连接，有一个单独的限制。 连接升级后，不会计入 `MaxConcurrentConnections` 限制。

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=4)]

默认情况下，最大连接数不受限制 (NULL)。

### <a name="maximum-request-body-size"></a>请求正文最大大小

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

默认的请求正文最大大小为 30,000,000 字节，大约 28.6 MB。

在 ASP.NET Core MVC 应用中替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

以下示例演示如何为每个请求上的应用配置约束：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=5)]

在中间件中替代特定请求的设置：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

如果应用在开始读取请求后配置请求限制，则会引发异常。 `IsReadOnly` 属性指示 `MaxRequestBodySize` 属性处于只读状态，意味已经无法再配置限制。

当应用在 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)后于[进程外](xref:host-and-deploy/iis/index#out-of-process-hosting-model)运行时，由于 IIS 已设置限制，因此禁用了 Kestrel 的请求正文大小限制。

### <a name="minimum-request-body-data-rate"></a>请求正文最小数据速率

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel 每秒检查一次数据是否以指定的速率（字节/秒）传入。 如果速率低于最小值，则连接超时。宽限期是 Kestrel 提供给客户端用于将其发送速率提升到最小值的时间量；在此期间不会检查速率。 宽限期有助于避免最初由于 TCP 慢启动而以较慢速率发送数据的连接中断。

默认的最小速率为 240 字节/秒，包含 5 秒的宽限期。

最小速率也适用于响应。 除了属性和接口名称中具有 `RequestBody` 或 `Response` 以外，用于设置请求限制和响应限制的代码相同。

以下示例演示如何在 Program.cs 中配置最小数据速率：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=6-9)]

在中间件中替代每个请求的最低速率限制：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=6-21)]

由于协议支持请求多路复用，HTTP/2 不支持基于每个请求修改速率限制，因此 HTTP/2 请求的 `HttpContext.Features` 中不存在前面示例中引用的速率特性。 通过 `KestrelServerOptions.Limits` 配置的服务器范围的速率限制仍适用于 HTTP/1.x 和 HTTP/2 连接。

### <a name="request-headers-timeout"></a>请求标头超时

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

获取或设置服务器接收请求标头所花费的最大时间量。 默认值为 30 秒。

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_Limits&highlight=16)]

### <a name="maximum-streams-per-connection"></a>每个连接的最大流

`Http2.MaxStreamsPerConnection` 限制每个 HTTP/2 连接的并发请求流的数量。 拒绝过多的流。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxStreamsPerConnection = 100;
        });
```

默认值为 100。

### <a name="header-table-size"></a>标题表大小

HPACK 解码器解压缩 HTTP/2 连接的 HTTP 标头。 `Http2.HeaderTableSize` 限制 HPACK 解码器使用的标头压缩表的大小。 该值以八位字节提供，且必须大于零 (0)。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.HeaderTableSize = 4096;
        });
```

默认值为 4096。

### <a name="maximum-frame-size"></a>最大帧大小

`Http2.MaxFrameSize` 指示要接收的 HTTP/2 连接帧有效负载的最大大小。 该值以八位字节提供，必须介于 2^14 (16,384) 和 2^24-1 (16,777,215) 之间。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxFrameSize = 16384;
        });
```

默认值为 2^14 (16,384)。

### <a name="maximum-request-header-size"></a>最大请求标头大小

`Http2.MaxRequestHeaderFieldSize` 表示请求标头值的允许的最大大小（用八进制表示）。 此限制同时适用于压缩和未压缩表示形式中的名称和值。 该值必须大于零 (0)。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.MaxRequestHeaderFieldSize = 8192;
        });
```

默认值为 8,192。

### <a name="initial-connection-window-size"></a>初始连接窗口大小

`Http2.InitialConnectionWindowSize` 表示服务器一次性缓存的最大请求主体数据大小（每次连接时在所有请求（流）中汇总，以字节为单位）。 请求也受 `Http2.InitialStreamWindowSize` 限制。 该值必须大于或等于 65,535，并小于 2^31 (2,147,483,648)。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.InitialConnectionWindowSize = 131072;
        });
```

默认值为 128 KB (131,072)。

### <a name="initial-stream-window-size"></a>初始流窗口大小

`Http2.InitialStreamWindowSize` 表示服务器针对每个请求（流）的一次性缓存的最大请求主体数据大小（以字节为单位）。 请求也受 `Http2.InitialStreamWindowSize` 限制。 该值必须大于或等于 65,535，并小于 2^31 (2,147,483,648)。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.Limits.Http2.InitialStreamWindowSize = 98304;
        });
```

默认值为 96 KB (98,304)。

### <a name="synchronous-io"></a>同步 I/O

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> 控制是否允许对请求和响应使用同步 I/O。 默认值为 `true`。

> [!WARNING]
> 大量阻止同步 I/O 的操作可能会导致线程池资源不足，进而导致应用无响应。 仅在使用不支持异步 I/O 的库时，才启用 `AllowSynchronousIO`。

以下示例会启用同步 I/O：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_SyncIO)]

有关其他 Kestrel 选项和限制的信息，请参阅：

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a>终结点配置

默认情况下，ASP.NET Core 绑定到：

* `http://localhost:5000`
* `https://localhost:5001`（存在本地开发证书时）

使用以下内容指定 URL：

* `ASPNETCORE_URLS` 环境变量。
* `--urls` 命令行参数。
* `urls` 主机配置键。
* `UseUrls` 扩展方法。

采用这些方法提供的值可以是一个或多个 HTTP 和 HTTPS 终结点（如果默认证书可用，则为 HTTPS）。 将值配置为以分号分隔的列表（例如 `"Urls": "http://localhost:8000;http://localhost:8001"`）。

有关这些方法的详细信息，请参阅[服务器 URL](xref:fundamentals/host/web-host#server-urls) 和[重写配置](xref:fundamentals/host/web-host#override-configuration)。

关于开发证书的创建：

* 安装了 [.NET Core SDK](/dotnet/core/sdk) 时。
* [dev-certs tool](xref:aspnetcore-2.1#https) 用于创建证书。

某些浏览器需要授予显式权限才能信任本地开发证书。

项目模板将应用配置为默认情况下在 HTTPS 上运行，并包括 [HTTPS 重定向和 HSTS 支持](xref:security/enforcing-ssl)。

调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 上的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 或 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 方法以配置 URL 前缀和 Kestrel 的端口。

`UseUrls`、`--urls` 命令行参数、`urls` 主机配置键以及 `ASPNETCORE_URLS` 环境变量也有用，但具有本节后面注明的限制（必须要有可用于 HTTPS 终结点配置的默认证书）。

`KestrelServerOptions` 配置：

### <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults(Action\<ListenOptions>)

指定一个为每个指定的终结点运行的配置 `Action`。 多次调用 `ConfigureEndpointDefaults`，用最新指定的 `Action` 替换之前的 `Action`。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
        });
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)

指定一个为每个 HTTPS 终结点运行的配置 `Action`。 多次调用 `ConfigureHttpsDefaults`，用最新指定的 `Action` 替换之前的 `Action`。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                // certificate is an X509Certificate2
                listenOptions.ServerCertificate = certificate;
            });
        });
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。


### <a name="configureiconfiguration"></a>Configure(IConfiguration)

创建配置加载程序，用于设置将 <xref:Microsoft.Extensions.Configuration.IConfiguration> 作为输入的 Kestrel。 配置必须针对 Kestrel 的配置节。

### <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

将 Kestrel 配置为使用 HTTPS。

`ListenOptions.UseHttps` 扩展：

* `UseHttps`：将 Kestrel 配置为使用 HTTPS，采用默认证书。 如果没有配置默认证书，则会引发异常。
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

`ListenOptions.UseHttps` 参数：

* `filename` 是证书文件的路径和文件名，关联包含应用内容文件的目录。
* `password` 是访问 X.509 证书数据所需的密码。
* `configureOptions` 是配置 `HttpsConnectionAdapterOptions` 的 `Action`。 返回 `ListenOptions`。
* `storeName` 是从中加载证书的证书存储。
* `subject` 是证书的主题名称。
* `allowInvalid` 指示是否存在需要留意的无效证书，例如自签名证书。
* `location` 是从中加载证书的存储位置。
* `serverCertificate` 是 X.509 证书。

在生产中，必须显式配置 HTTPS。 至少必须提供默认证书。

下面要描述的支持的配置：

* 无配置
* 从配置中替换默认证书
* 更改代码中的默认值

*无配置*

Kestrel 在 `http://localhost:5000` 和 `https://localhost:5001` 上进行侦听（如果默认证书可用）。

<a name="configuration"></a>

*从配置中替换默认证书*

`CreateDefaultBuilder` 在默认情况下调用 `Configure(context.Configuration.GetSection("Kestrel"))` 来加载 Kestrel 配置。 Kestrel 可以使用默认 HTTPS 应用设置配置架构。 从磁盘上的文件或从证书存储中配置多个终结点，包括要使用的 URL 和证书。

在以下 appsettings.json 示例中：

* 将 AllowInvalid 设置为 `true`，从而允许使用无效证书（例如自签名证书）。
* 任何未指定证书的 HTTPS 终结点（下例中的 HttpsDefaultCert）会回退至在 Certificates > Default 下定义的证书或开发证书  。

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },

      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },

      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },

      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },

      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

此外还可以使用任何证书节点的 Path 和 Password，采用证书存储字段指定证书 。 例如，可将 Certificates > Default 证书指定为 ：

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

架构的注意事项：

* 终结点的名称不区分大小写。 例如，`HTTPS` 和 `Https` 都是有效的。
* 每个终结点都要具备 `Url` 参数。 此参数的格式和顶层 `Urls` 配置参数一样，只不过它只能有单个值。
* 这些终结点不会添加进顶层 `Urls` 配置中定义的终结点，而是替换它们。 通过 `Listen` 在代码中定义的终结点与在配置节中定义的终结点相累积。
* `Certificate` 部分是可选的。 如果为指定 `Certificate` 部分，则使用在之前的方案中定义的默认值。 如果没有可用的默认值，服务器会引发异常且无法启动。
* `Certificate` 支持 Path&ndash;Password 和 Subject&ndash;Store 证书   。
* 只要不会导致端口冲突，就能以这种方式定义任何数量的终结点。
* `options.Configure(context.Configuration.GetSection("{SECTION}"))` 通过 `.Endpoint(string name, listenOptions => { })` 方法返回 `KestrelConfigurationLoader`，可以用于补充已配置的终结点设置：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
                .Endpoint("HTTPS", listenOptions =>
                {
                    listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
                });
        });
```

可以直接访问 `KestrelServerOptions.ConfigurationLoader` 以继续迭代现有加载程序，例如由 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 提供的加载程序。

* 每个终结点的配置节都可用于 `Endpoint` 方法中的选项，以便读取自定义设置。
* 通过另一节再次调用 `options.Configure(context.Configuration.GetSection("{SECTION}"))` 可能加载多个配置。 只使用最新配置，除非之前的实例上显式调用了 `Load`。 元包不会调用 `Load`，所以可能会替换它的默认配置节。
* `KestrelConfigurationLoader` 从 `KestrelServerOptions` 将 API 的 `Listen` 簇反射为 `Endpoint` 重载，因此可在同样的位置配置代码和配置终结点。 这些重载不使用名称，且只使用配置中的默认设置。

*更改代码中的默认值*

可以使用 `ConfigureEndpointDefaults` 和 `ConfigureHttpsDefaults` 更改 `ListenOptions` 和 `HttpsConnectionAdapterOptions` 的默认设置，包括重写之前的方案指定的默认证书。 需要在配置任何终结点之前调用 `ConfigureEndpointDefaults` 和 `ConfigureHttpsDefaults`。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
            
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                listenOptions.SslProtocols = SslProtocols.Tls12;
            });
        });
```

*SNI 的 Kestrel 支持*

[服务器名称指示 (SNI)](https://tools.ietf.org/html/rfc6066#section-3) 可用于承载相同 IP 地址和端口上的多个域。 为了运行 SNI，客户端在 TLS 握手过程中将进行安全会话的主机名发送至服务器，从而让服务器可以提供正确的证书。 在 TLS 握手后的安全会话期间，客户端将服务器提供的证书用于与服务器进行加密通信。

Kestrel 通过 `ServerCertificateSelector` 回调支持 SNI。 每次连接调用一次回调，从而允许应用检查主机名并选择合适的证书。

SNI 支持要求：

* 在目标框架 `netcoreapp2.1` 或更高版本上运行。 在 `net461` 或最高版本上，将调用回调，但是 `name` 始终为 `null`。 如果客户端未在 TLS 握手过程中提供主机名参数，则 `name` 也为 `null`。
* 所有网站在相同的 Kestrel 实例上运行。 Kestrel 在无反向代理时不支持跨多个实例共享一个 IP 地址和端口。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ListenAnyIP(5005, listenOptions =>
            {
                listenOptions.UseHttps(httpsOptions =>
                {
                    var localhostCert = CertificateLoader.LoadFromStoreCert(
                        "localhost", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var exampleCert = CertificateLoader.LoadFromStoreCert(
                        "example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var subExampleCert = CertificateLoader.LoadFromStoreCert(
                        "sub.example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var certs = new Dictionary<string, X509Certificate2>(
                        StringComparer.OrdinalIgnoreCase);
                    certs["localhost"] = localhostCert;
                    certs["example.com"] = exampleCert;
                    certs["sub.example.com"] = subExampleCert;

                    httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
                    {
                        if (name != null && certs.TryGetValue(name, out var cert))
                        {
                            return cert;
                        }

                        return exampleCert;
                    };
                });
            });
        });
```

### <a name="connection-logging"></a>连接日志记录

调用 <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> 以发出用于进行连接上的字节级别通信的调试级别日志。 连接日志记录有助于排查低级通信中的问题，例如在 TLS 加密期间和代理后。 如果 `UseConnectionLogging` 放置在 `UseHttps` 之前，则会记录加密的流量。 如果 `UseConnectionLogging` 放置于 `UseHttps` 之后，则会记录解密的流量。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a>绑定到 TCP 套接字

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 方法绑定至 TCP 套接字，且 options lambda 允许 X.509 证书配置：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_TCPSocket&highlight=9-16)]

示例使用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions> 为终结点配置 HTTPS。 可使用相同 API 为特定终结点配置其他 Kestrel 设置。

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a>绑定到 Unix 套接字

可通过 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 侦听 Unix 套接字以提高 Nginx 的性能，如以下示例所示：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Program.cs?name=snippet_UnixSocket)]

* 在 Nginx confiuguration 文件中，将 `server` > `location` > `proxy_pass` 条目设置为 `http://unix:/tmp/{KESTREL SOCKET}:/;`。 `{KESTREL SOCKET}` 是提供给 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 的套接字的名称（例如，上述示例中的 `kestrel-test.sock`）。
* 确保套接字可由 Nginx （例如 `chmod go+w /tmp/kestrel-test.sock`）进行写入。 

### <a name="port-0"></a>端口 0

如果指定端口号 `0`，Kestrel 将动态绑定到可用端口。 以下示例演示如何确定 Kestrel 在运行时实际绑定到的端口：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

在应用运行时，控制台窗口输出指示可用于访问应用的动态端口：

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a>限制

使用以下方法配置终结点：

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* `--urls` 命令行参数
* `urls` 主机配置键
* `ASPNETCORE_URLS` 环境变量

若要将代码用于 Kestrel 以外的服务器，这些方法非常有用。 不过，请注意以下限制：

* HTTPS 无法与这些方法结合使用，除非在 HTTPS 终结点配置中提供了默认证书（例如，使用 `KestrelServerOptions` 配置或配置文件，如本主题前面的部分所示）。
* 如果同时使用 `Listen` 和 `UseUrls` 方法，`Listen` 终结点将覆盖 `UseUrls` 终结点。

### <a name="iis-endpoint-configuration"></a>IIS 终结点配置

使用 IIS 时，由 `Listen` 或 `UseUrls` 设置用于 IIS 覆盖绑定的 URL 绑定。 有关详细信息，请参阅 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)主题。

### <a name="listenoptionsprotocols"></a>ListenOptions.Protocols

`Protocols` 属性建立在连接终结点上或为服务器启用的 HTTP 协议（`HttpProtocols`）。 从 `HttpProtocols` 枚举向 `Protocols` 属性赋值。

| `HttpProtocols` 枚举值 | 允许的连接协议 |
| -------------------------- | ----------------------------- |
| `Http1`                    | 仅 HTTP/1.1。 可以在具有 TLS 或没有 TLS 的情况下使用。 |
| `Http2`                    | 仅 HTTP/2。 仅当客户端支持[先验知识模式](https://tools.ietf.org/html/rfc7540#section-3.4)时，才可以在没有 TLS 的情况下使用。 |
| `Http1AndHttp2`            | HTTP/1.1 和 HTTP/2。 HTTP/2 需要 TLS 和[应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接；否则，连接默认为 HTTP/1.1。 |

默认协议是 HTTP/1.1。

HTTP/2 的 TLS 限制：

* TLS 版本 1.2 或更高版本
* 重新协商已禁用
* 压缩已禁用
* 最小的临时密钥交换大小：
  * 椭圆曲线 Diffie-Hellman (ECDHE) &lbrack;[RFC4492](https://www.ietf.org/rfc/rfc4492.txt)&rbrack;：最小 224 位
  * 有限字段 Diffie-Hellman (DHE) &lbrack;`TLS12`&rbrack;：最小 2048 位
* 未阻止密码套件

默认情况下，支持具有 P-256 椭圆曲线 &lbrack;`FIPS186`&rbrack; 的 `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256` &lbrack;`TLS-ECDHE`&rbrack;。

以下示例允许端口 8000 上的 HTTP/1.1 和 HTTP/2 连接。 TLS 使用提供的证书来保护连接：

```csharp
.ConfigureKestrel((context, serverOptions) =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps("testCert.pfx", "testPassword");
    });
});
```

（可选）创建 `IConnectionAdapter` 实现，以针对特定密码的每个连接筛选 TLS 握手：

```csharp
.ConfigureKestrel((context, serverOptions) =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
        listenOptions.UseHttps("testCert.pfx", "testPassword");
        listenOptions.ConnectionAdapters.Add(new TlsFilterAdapter());
    });
});
```

```csharp
private class TlsFilterAdapter : IConnectionAdapter
{
    public bool IsHttps => false;

    public Task<IAdaptedConnection> OnConnectionAsync(ConnectionAdapterContext context)
    {
        var tlsFeature = context.Features.Get<ITlsHandshakeFeature>();

        // Throw NotSupportedException for any cipher algorithm that the app doesn't
        // wish to support. Alternatively, define and compare
        // ITlsHandshakeFeature.CipherAlgorithm to a list of acceptable cipher
        // suites.
        //
        // No encryption is used with a CipherAlgorithmType.Null cipher algorithm.
        if (tlsFeature.CipherAlgorithm == CipherAlgorithmType.Null)
        {
            throw new NotSupportedException("Prohibited cipher: " + tlsFeature.CipherAlgorithm);
        }

        return Task.FromResult<IAdaptedConnection>(new AdaptedConnection(context.ConnectionStream));
    }

    private class AdaptedConnection : IAdaptedConnection
    {
        public AdaptedConnection(Stream adaptedStream)
        {
            ConnectionStream = adaptedStream;
        }

        public Stream ConnectionStream { get; }

        public void Dispose()
        {
        }
    }
}
```

*从配置中设置协议*

<xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 在默认情况下调用 `serverOptions.Configure(context.Configuration.GetSection("Kestrel"))` 来加载 Kestrel 配置。

在以下 appsettings.json 示例中，为 Kestrel 的所有终结点建立默认的连接协议（HTTP/1.1 和 HTTP/2）：

```json
{
  "Kestrel": {
    "EndpointDefaults": {
      "Protocols": "Http1AndHttp2"
    }
  }
}
```

以下配置文件示例为特定终结点建立了连接协议：

```json
{
  "Kestrel": {
    "Endpoints": {
      "HttpsDefaultCert": {
        "Url": "https://localhost:5001",
        "Protocols": "Http1AndHttp2"
      }
    }
  }
}
```

代码中指定的协议覆盖了由配置设置的值。

## <a name="transport-configuration"></a>传输配置

对于 ASP.NET Core 2.1 版，Kestrel 默认传输不再基于 Libuv，而是基于托管的套接字。 这是 ASP.NET Core 2.0 应用升级到 2.1 时的一个重大更改，它调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> 并依赖于以下包中的一个：

* [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/)（直接包引用）
* [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)

对于需要使用 Libuv 的项目：

* 将用于 [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) 包的依赖项添加到应用的项目文件：

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* 调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>：

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateWebHostBuilder(args).Build().Run();
      }

      public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
          WebHost.CreateDefaultBuilder(args)
              .UseLibuv()
              .UseStartup<Startup>();
  }
  ```

### <a name="url-prefixes"></a>URL 前缀

如果使用 `UseUrls`、`--urls` 命令行参数、`urls` 主机配置键或 `ASPNETCORE_URLS` 环境变量，URL 前缀可采用以下任意格式。

仅 HTTP URL 前缀是有效的。 使用 `UseUrls` 配置 URL 绑定时，Kestrel 不支持 HTTPS。

* 包含端口号的 IPv4 地址

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` 是一种绑定到所有 IPv4 地址的特殊情况。

* 包含端口号的 IPv6 地址

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` 是 IPv4 `0.0.0.0` 的 IPv6 等效项。

* 包含端口号的主机名

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  主机名、`*`和 `+` 并不特殊。 没有识别为有效 IP 地址或 `localhost` 的任何内容都将绑定到所有 IPv4 和 IPv6 IP。 若要将不同主机名绑定到相同端口上的不同 ASP.NET Core 应用，请使用 [HTTP.sys](xref:fundamentals/servers/httpsys) 或 IIS、Nginx 或 Apache 等反向代理服务器。

  > [!WARNING]
  > 采用反向代理配置进行托管需要[主机筛选](#host-filtering)。

* 包含端口号的主机 `localhost` 名称或包含端口号的环回 IP

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  指定 `localhost` 后，Kestrel 将尝试绑定到 IPv4 和 IPv6 环回接口。 如果其他服务正在任一环回接口上使用请求的端口，则 Kestrel 将无法启动。 如果任一环回接口出于任何其他原因（通常是因为 IPv6 不受支持）而不可用，则 Kestrel 将记录一个警告。

## <a name="host-filtering"></a>主机筛选

尽管 Kestrel 支持基于前缀的配置（例如 `http://example.com:5000`），但 Kestrel 在很大程度上会忽略主机名。 主机 `localhost` 是一个特殊情况，用于绑定至环回地址。 除了显式 IP 地址以外的所有主机都绑定至所有公共 IP 地址。 不验证 `Host` 标头。

解决方法是，使用主机筛选中间件。 主机筛选中间件由 [Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) 包提供，此包包含在 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)中（ASP.NET Core 2.1 或 2.2）。 由调用 <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*> 的 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 添加中间件：

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

默认情况下，主机筛选中间件处于禁用状态。 要启用该中间件，请在 appsettings.json/appsettings.\<EnvironmentName>.json 中定义一个 `AllowedHosts` 键。 此值是以分号分隔的不带端口号的主机名列表：

appsettings.json：

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> [转接头中间件](xref:host-and-deploy/proxy-load-balancer) 同样包含 <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> 选项。 转接头中间件和主机筛选中间件具有适合不同方案的相似功能。 如果未保留 `Host` 标头，并且使用反向代理服务器或负载均衡器转接请求，则使用转接头中间件设置 `AllowedHosts` 比较合适。 将 Kestrel 用作面向公众的边缘服务器或直接转接 `Host` 标头时，使用主机筛选中间件设置 `AllowedHosts` 比较合适。
>
> 有关转接头中间件的详细信息，请参阅 <xref:host-and-deploy/proxy-load-balancer>。

::: moniker-end

::: moniker range="= aspnetcore-2.1"

Kestrel 是一个跨平台的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 Kestrel 是 Web 服务器，默认包括在 ASP.NET Core 项目模板中。

Kestrel 支持以下方案：

* HTTPS
* 用于启用 [WebSocket](https://github.com/aspnet/websockets) 的不透明升级
* 用于获得 Nginx 高性能的 Unix 套接字

.NET Core 支持的所有平台和版本均支持 Kestrel。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/kestrel/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-kestrel-with-a-reverse-proxy"></a>何时结合使用 Kestrel 和反向代理

可以单独使用 Kestrel，也可以将其与反向代理服务器（如 [Internet Information Services (IIS)](https://www.iis.net/)、[Nginx](https://nginx.org) 或 [Apache](https://httpd.apache.org/)）结合使用。 反向代理服务器接收来自网络的 HTTP 请求，并将这些请求转发到 Kestrel。

Kestrel 用作边缘（面向 Internet）Web 服务器：

![Kestrel 直接与 Internet 通信，不使用反向代理服务器](kestrel/_static/kestrel-to-internet2.png)

Kestrel 用于反向代理配置：

![Kestrel 通过反向代理服务器（如 IIS、Nginx 或 Apache）间接与 Internet 进行通信](kestrel/_static/kestrel-to-internet.png)

无论配置是否使用反向代理服务器，都是受支持的托管配置。

在没有反向代理服务器的情况下用作边缘服务器的 Kestrel 不支持在多个进程间共享相同的 IP 和端口。 如果将 Kestrel 配置为侦听某个端口，Kestrel 会处理该端口的所有流量（无视请求的 `Host` 标头）。 可以共享端口的反向代理能在唯一的 IP 和端口上将请求转发至 Kestrel。

即使不需要反向代理服务器，使用反向代理服务器可能也是个不错的选择。

反向代理：

* 可以限制所承载的应用中的公开的公共外围应用。
* 提供额外的配置和防护层。
* 可以更好地与现有基础结构集成。
* 简化了负载均和和安全通信 (HTTPS) 配置。 仅反向代理服务器需要 X.509 证书，并且该服务器可使用普通 HTTP 在内部网络上与应用服务器通信。

> [!WARNING]
> 采用反向代理配置进行托管需要[主机筛选](#host-filtering)。

## <a name="how-to-use-kestrel-in-aspnet-core-apps"></a>如何在 ASP.NET Core 应用中使用 Kestrel

[Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)中包括 [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/) 包。

默认情况下，ASP.NET Core 项目模板使用 Kestrel。 在 Program.cs 中，模板代码调用 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>，后者在后台调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>。

若要在调用 `CreateDefaultBuilder` 后提供其他配置，请调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderKestrelExtensions.UseKestrel*>：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            // Set properties and call methods on serverOptions
        });
```

有关 `CreateDefaultBuilder` 和生成主机的详细信息，请参阅 <xref:fundamentals/host/web-host#set-up-a-host> 的“设置主机”部分。

## <a name="kestrel-options"></a>Kestrel 选项

Kestrel Web 服务器具有约束配置选项，这些选项在面向 Internet 的部署中尤其有用。

对 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 类的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Limits> 属性设置约束。 `Limits` 属性包含 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits> 类的实例。

下面的示例使用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core> 命名空间。

```csharp
using Microsoft.AspNetCore.Server.Kestrel.Core;
```

Kestrel 选项（已在以下示例的 C# 代码中配置）也可以使用[配置提供程序](xref:fundamentals/configuration/index)进行设置。 例如，文件配置提供程序可以从 appsettings.json 或 appsettings.{Environment}.json 文件加载 Kestrel 配置 ：

```json
{
  "Kestrel": {
    "Limits": {
      "MaxConcurrentConnections": 100,
      "MaxConcurrentUpgradedConnections": 100
    }
  }
}
```

使用以下方法之一：

* 在 `Startup.ConfigureServices` 中配置 Kestrel：

  1. 将 `IConfiguration` 的实例注入到 `Startup` 类中。 下面的示例假定注入的配置已分配给 `Configuration` 属性。
  2. 在 `Startup.ConfigureServices` 中，将配置的 `Kestrel` 部分加载到 Kestrel 的配置中：

     ```csharp
     using Microsoft.Extensions.Configuration
     
     public class Startup
     {
         public Startup(IConfiguration configuration)
         {
             Configuration = configuration;
         }

         public IConfiguration Configuration { get; }

         public void ConfigureServices(IServiceCollection services)
         {
             services.Configure<KestrelServerOptions>(
                 Configuration.GetSection("Kestrel"));
         }

         public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
         {
             ...
         }
     }
     ```

* 构建主机时配置 Kestrel：

  在 Program.cs 中，将配置的 `Kestrel` 部分加载到 Kestrel 的配置中：

  ```csharp
  // using Microsoft.Extensions.DependencyInjection;

  public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
      WebHost.CreateDefaultBuilder(args)
          .ConfigureServices((context, services) =>
          {
              services.Configure<KestrelServerOptions>(
                  context.Configuration.GetSection("Kestrel"));
          })
          .UseStartup<Startup>();
  ```

上述两种方法适用于任何[配置提供程序](xref:fundamentals/configuration/index)。

### <a name="keep-alive-timeout"></a>保持活动状态超时

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.KeepAliveTimeout>

获取或设置[保持活动状态超时](https://tools.ietf.org/html/rfc7230#section-6.5)。 默认值为 2 分钟。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.KeepAliveTimeout = TimeSpan.FromMinutes(2);
        });
```

### <a name="maximum-client-connections"></a>客户端最大连接数

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentConnections>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxConcurrentUpgradedConnections>

可使用以下代码为整个应用设置并发打开的最大 TCP 连接数：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxConcurrentConnections = 100;
        });
```

对于已从 HTTP 或 HTTPS 升级到另一个协议（例如，Websocket 请求）的连接，有一个单独的限制。 连接升级后，不会计入 `MaxConcurrentConnections` 限制。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxConcurrentUpgradedConnections = 100;
        });
```

默认情况下，最大连接数不受限制 (NULL)。

### <a name="maximum-request-body-size"></a>请求正文最大大小

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MaxRequestBodySize>

默认的请求正文最大大小为 30,000,000 字节，大约 28.6 MB。

在 ASP.NET Core MVC 应用中替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

以下示例演示如何为每个请求上的应用配置约束：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MaxRequestBodySize = 10 * 1024;
        });
```

在中间件中替代特定请求的设置：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Limits&highlight=3-4)]

如果应用在开始读取请求后配置请求限制，则会引发异常。 `IsReadOnly` 属性指示 `MaxRequestBodySize` 属性处于只读状态，意味已经无法再配置限制。

当应用在 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)后于[进程外](xref:host-and-deploy/iis/index#out-of-process-hosting-model)运行时，由于 IIS 已设置限制，因此禁用了 Kestrel 的请求正文大小限制。

### <a name="minimum-request-body-data-rate"></a>请求正文最小数据速率

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinRequestBodyDataRate>
<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.MinResponseDataRate>

Kestrel 每秒检查一次数据是否以指定的速率（字节/秒）传入。 如果速率低于最小值，则连接超时。宽限期是 Kestrel 提供给客户端用于将其发送速率提升到最小值的时间量；在此期间不会检查速率。 宽限期有助于避免最初由于 TCP 慢启动而以较慢速率发送数据的连接中断。

默认的最小速率为 240 字节/秒，包含 5 秒的宽限期。

最小速率也适用于响应。 除了属性和接口名称中具有 `RequestBody` 或 `Response` 以外，用于设置请求限制和响应限制的代码相同。

以下示例演示如何在 Program.cs 中配置最小数据速率：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.MinRequestBodyDataRate =
                new MinDataRate(bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
            serverOptions.Limits.MinResponseDataRate =
                new MinDataRate(bytesPerSecond: 100, gracePeriod: TimeSpan.FromSeconds(10));
        });
```

### <a name="request-headers-timeout"></a>请求标头超时

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits.RequestHeadersTimeout>

获取或设置服务器接收请求标头所花费的最大时间量。 默认值为 30 秒。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Limits.RequestHeadersTimeout = TimeSpan.FromMinutes(1);
        });
```

### <a name="synchronous-io"></a>同步 I/O

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.AllowSynchronousIO> 控制是否允许对请求和响应使用同步 I/O。 默认值为 `true`。

> [!WARNING]
> 大量阻止同步 I/O 的操作可能会导致线程池资源不足，进而导致应用无响应。 仅在使用不支持异步 I/O 的库时，才启用 `AllowSynchronousIO`。

以下示例会禁用同步 I/O：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.AllowSynchronousIO = false;
        });
```

有关其他 Kestrel 选项和限制的信息，请参阅：

* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerLimits>
* <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions>

## <a name="endpoint-configuration"></a>终结点配置

默认情况下，ASP.NET Core 绑定到：

* `http://localhost:5000`
* `https://localhost:5001`（存在本地开发证书时）

使用以下内容指定 URL：

* `ASPNETCORE_URLS` 环境变量。
* `--urls` 命令行参数。
* `urls` 主机配置键。
* `UseUrls` 扩展方法。

采用这些方法提供的值可以是一个或多个 HTTP 和 HTTPS 终结点（如果默认证书可用，则为 HTTPS）。 将值配置为以分号分隔的列表（例如 `"Urls": "http://localhost:8000;http://localhost:8001"`）。

有关这些方法的详细信息，请参阅[服务器 URL](xref:fundamentals/host/web-host#server-urls) 和[重写配置](xref:fundamentals/host/web-host#override-configuration)。

关于开发证书的创建：

* 安装了 [.NET Core SDK](/dotnet/core/sdk) 时。
* [dev-certs tool](xref:aspnetcore-2.1#https) 用于创建证书。

某些浏览器需要授予显式权限才能信任本地开发证书。

项目模板将应用配置为默认情况下在 HTTPS 上运行，并包括 [HTTPS 重定向和 HSTS 支持](xref:security/enforcing-ssl)。

调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions> 上的 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 或 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 方法以配置 URL 前缀和 Kestrel 的端口。

`UseUrls`、`--urls` 命令行参数、`urls` 主机配置键以及 `ASPNETCORE_URLS` 环境变量也有用，但具有本节后面注明的限制（必须要有可用于 HTTPS 终结点配置的默认证书）。

`KestrelServerOptions` 配置：

### <a name="configureendpointdefaultsactionlistenoptions"></a>ConfigureEndpointDefaults(Action\<ListenOptions>)

指定一个为每个指定的终结点运行的配置 `Action`。 多次调用 `ConfigureEndpointDefaults`，用最新指定的 `Action` 替换之前的 `Action`。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
        });
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureEndpointDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。

### <a name="configurehttpsdefaultsactionhttpsconnectionadapteroptions"></a>ConfigureHttpsDefaults(Action\<HttpsConnectionAdapterOptions>)

指定一个为每个 HTTPS 终结点运行的配置 `Action`。 多次调用 `ConfigureHttpsDefaults`，用最新指定的 `Action` 替换之前的 `Action`。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                // certificate is an X509Certificate2
                listenOptions.ServerCertificate = certificate;
            });
        });
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。

### <a name="configureiconfiguration"></a>Configure(IConfiguration)

创建配置加载程序，用于设置将 <xref:Microsoft.Extensions.Configuration.IConfiguration> 作为输入的 Kestrel。 配置必须针对 Kestrel 的配置节。

### <a name="listenoptionsusehttps"></a>ListenOptions.UseHttps

将 Kestrel 配置为使用 HTTPS。

`ListenOptions.UseHttps` 扩展：

* `UseHttps`：将 Kestrel 配置为使用 HTTPS，采用默认证书。 如果没有配置默认证书，则会引发异常。
* `UseHttps(string fileName)`
* `UseHttps(string fileName, string password)`
* `UseHttps(string fileName, string password, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(StoreName storeName, string subject)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location)`
* `UseHttps(StoreName storeName, string subject, bool allowInvalid, StoreLocation location, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(X509Certificate2 serverCertificate)`
* `UseHttps(X509Certificate2 serverCertificate, Action<HttpsConnectionAdapterOptions> configureOptions)`
* `UseHttps(Action<HttpsConnectionAdapterOptions> configureOptions)`

`ListenOptions.UseHttps` 参数：

* `filename` 是证书文件的路径和文件名，关联包含应用内容文件的目录。
* `password` 是访问 X.509 证书数据所需的密码。
* `configureOptions` 是配置 `HttpsConnectionAdapterOptions` 的 `Action`。 返回 `ListenOptions`。
* `storeName` 是从中加载证书的证书存储。
* `subject` 是证书的主题名称。
* `allowInvalid` 指示是否存在需要留意的无效证书，例如自签名证书。
* `location` 是从中加载证书的存储位置。
* `serverCertificate` 是 X.509 证书。

在生产中，必须显式配置 HTTPS。 至少必须提供默认证书。

下面要描述的支持的配置：

* 无配置
* 从配置中替换默认证书
* 更改代码中的默认值

*无配置*

Kestrel 在 `http://localhost:5000` 和 `https://localhost:5001` 上进行侦听（如果默认证书可用）。

<a name="configuration"></a>

*从配置中替换默认证书*

`CreateDefaultBuilder` 在默认情况下调用 `Configure(context.Configuration.GetSection("Kestrel"))` 来加载 Kestrel 配置。 Kestrel 可以使用默认 HTTPS 应用设置配置架构。 从磁盘上的文件或从证书存储中配置多个终结点，包括要使用的 URL 和证书。

在以下 appsettings.json 示例中：

* 将 AllowInvalid 设置为 `true`，从而允许使用无效证书（例如自签名证书）。
* 任何未指定证书的 HTTPS 终结点（下例中的 HttpsDefaultCert）会回退至在 Certificates > Default 下定义的证书或开发证书  。

```json
{
  "Kestrel": {
    "Endpoints": {
      "Http": {
        "Url": "http://localhost:5000"
      },

      "HttpsInlineCertFile": {
        "Url": "https://localhost:5001",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      },

      "HttpsInlineCertStore": {
        "Url": "https://localhost:5002",
        "Certificate": {
          "Subject": "<subject; required>",
          "Store": "<certificate store; required>",
          "Location": "<location; defaults to CurrentUser>",
          "AllowInvalid": "<true or false; defaults to false>"
        }
      },

      "HttpsDefaultCert": {
        "Url": "https://localhost:5003"
      },

      "Https": {
        "Url": "https://*:5004",
        "Certificate": {
          "Path": "<path to .pfx file>",
          "Password": "<certificate password>"
        }
      }
    },
    "Certificates": {
      "Default": {
        "Path": "<path to .pfx file>",
        "Password": "<certificate password>"
      }
    }
  }
}
```

此外还可以使用任何证书节点的 Path 和 Password，采用证书存储字段指定证书 。 例如，可将 Certificates > Default 证书指定为 ：

```json
"Default": {
  "Subject": "<subject; required>",
  "Store": "<cert store; required>",
  "Location": "<location; defaults to CurrentUser>",
  "AllowInvalid": "<true or false; defaults to false>"
}
```

架构的注意事项：

* 终结点的名称不区分大小写。 例如，`HTTPS` 和 `Https` 都是有效的。
* 每个终结点都要具备 `Url` 参数。 此参数的格式和顶层 `Urls` 配置参数一样，只不过它只能有单个值。
* 这些终结点不会添加进顶层 `Urls` 配置中定义的终结点，而是替换它们。 通过 `Listen` 在代码中定义的终结点与在配置节中定义的终结点相累积。
* `Certificate` 部分是可选的。 如果为指定 `Certificate` 部分，则使用在之前的方案中定义的默认值。 如果没有可用的默认值，服务器会引发异常且无法启动。
* `Certificate` 支持 Path&ndash;Password 和 Subject&ndash;Store 证书   。
* 只要不会导致端口冲突，就能以这种方式定义任何数量的终结点。
* `options.Configure(context.Configuration.GetSection("{SECTION}"))` 通过 `.Endpoint(string name, listenOptions => { })` 方法返回 `KestrelConfigurationLoader`，可以用于补充已配置的终结点设置：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.Configure(context.Configuration.GetSection("Kestrel"))
                .Endpoint("HTTPS", listenOptions =>
                {
                    listenOptions.HttpsOptions.SslProtocols = SslProtocols.Tls12;
                });
        });
```

可以直接访问 `KestrelServerOptions.ConfigurationLoader` 以继续迭代现有加载程序，例如由 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 提供的加载程序。

* 每个终结点的配置节都可用于 `Endpoint` 方法中的选项，以便读取自定义设置。
* 通过另一节再次调用 `options.Configure(context.Configuration.GetSection("{SECTION}"))` 可能加载多个配置。 只使用最新配置，除非之前的实例上显式调用了 `Load`。 元包不会调用 `Load`，所以可能会替换它的默认配置节。
* `KestrelConfigurationLoader` 从 `KestrelServerOptions` 将 API 的 `Listen` 簇反射为 `Endpoint` 重载，因此可在同样的位置配置代码和配置终结点。 这些重载不使用名称，且只使用配置中的默认设置。

*更改代码中的默认值*

可以使用 `ConfigureEndpointDefaults` 和 `ConfigureHttpsDefaults` 更改 `ListenOptions` 和 `HttpsConnectionAdapterOptions` 的默认设置，包括重写之前的方案指定的默认证书。 需要在配置任何终结点之前调用 `ConfigureEndpointDefaults` 和 `ConfigureHttpsDefaults`。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ConfigureEndpointDefaults(listenOptions =>
            {
                // Configure endpoint defaults
            });
            
            serverOptions.ConfigureHttpsDefaults(listenOptions =>
            {
                listenOptions.SslProtocols = SslProtocols.Tls12;
            });
        });
```

*SNI 的 Kestrel 支持*

[服务器名称指示 (SNI)](https://tools.ietf.org/html/rfc6066#section-3) 可用于承载相同 IP 地址和端口上的多个域。 为了运行 SNI，客户端在 TLS 握手过程中将进行安全会话的主机名发送至服务器，从而让服务器可以提供正确的证书。 在 TLS 握手后的安全会话期间，客户端将服务器提供的证书用于与服务器进行加密通信。

Kestrel 通过 `ServerCertificateSelector` 回调支持 SNI。 每次连接调用一次回调，从而允许应用检查主机名并选择合适的证书。

SNI 支持要求：

* 在目标框架 `netcoreapp2.1` 或更高版本上运行。 在 `net461` 或最高版本上，将调用回调，但是 `name` 始终为 `null`。 如果客户端未在 TLS 握手过程中提供主机名参数，则 `name` 也为 `null`。
* 所有网站在相同的 Kestrel 实例上运行。 Kestrel 在无反向代理时不支持跨多个实例共享一个 IP 地址和端口。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel((context, serverOptions) =>
        {
            serverOptions.ListenAnyIP(5005, listenOptions =>
            {
                listenOptions.UseHttps(httpsOptions =>
                {
                    var localhostCert = CertificateLoader.LoadFromStoreCert(
                        "localhost", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var exampleCert = CertificateLoader.LoadFromStoreCert(
                        "example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var subExampleCert = CertificateLoader.LoadFromStoreCert(
                        "sub.example.com", "My", StoreLocation.CurrentUser,
                        allowInvalid: true);
                    var certs = new Dictionary<string, X509Certificate2>(
                        StringComparer.OrdinalIgnoreCase);
                    certs["localhost"] = localhostCert;
                    certs["example.com"] = exampleCert;
                    certs["sub.example.com"] = subExampleCert;

                    httpsOptions.ServerCertificateSelector = (connectionContext, name) =>
                    {
                        if (name != null && certs.TryGetValue(name, out var cert))
                        {
                            return cert;
                        }

                        return exampleCert;
                    };
                });
            });
        })
        .Build();
```

### <a name="connection-logging"></a>连接日志记录

调用 <xref:Microsoft.AspNetCore.Hosting.ListenOptionsConnectionLoggingExtensions.UseConnectionLogging*> 以发出用于进行连接上的字节级别通信的调试级别日志。 连接日志记录有助于排查低级通信中的问题，例如在 TLS 加密期间和代理后。 如果 `UseConnectionLogging` 放置在 `UseHttps` 之前，则会记录加密的流量。 如果 `UseConnectionLogging` 放置于 `UseHttps` 之后，则会记录解密的流量。

```csharp
webBuilder.ConfigureKestrel(serverOptions =>
{
    serverOptions.Listen(IPAddress.Any, 8000, listenOptions =>
    {
        listenOptions.UseConnectionLogging();
    });
});
```

### <a name="bind-to-a-tcp-socket"></a>绑定到 TCP 套接字

<xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 方法绑定至 TCP 套接字，且 options lambda 允许 X.509 证书配置：

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Listen(IPAddress.Loopback, 5000);
            serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testPassword");
            });
        });
```

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.Listen(IPAddress.Loopback, 5000);
            serverOptions.Listen(IPAddress.Loopback, 5001, listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testPassword");
            });
        });
```

示例使用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.ListenOptions> 为终结点配置 HTTPS。 可使用相同 API 为特定终结点配置其他 Kestrel 设置。

[!INCLUDE [How to make an X.509 cert](~/includes/make-x509-cert.md)]

### <a name="bind-to-a-unix-socket"></a>绑定到 Unix 套接字

可通过 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 侦听 Unix 套接字以提高 Nginx 的性能，如以下示例所示：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .UseKestrel(serverOptions =>
        {
            serverOptions.ListenUnixSocket("/tmp/kestrel-test.sock");
            serverOptions.ListenUnixSocket("/tmp/kestrel-test.sock", listenOptions =>
            {
                listenOptions.UseHttps("testCert.pfx", "testpassword");
            });
        });
```

* 在 Nginx confiuguration 文件中，将 `server` > `location` > `proxy_pass` 条目设置为 `http://unix:/tmp/{KESTREL SOCKET}:/;`。 `{KESTREL SOCKET}` 是提供给 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ListenUnixSocket*> 的套接字的名称（例如，上述示例中的 `kestrel-test.sock`）。
* 确保套接字可由 Nginx （例如 `chmod go+w /tmp/kestrel-test.sock`）进行写入。 

### <a name="port-0"></a>端口 0

如果指定端口号 `0`，Kestrel 将动态绑定到可用端口。 以下示例演示如何确定 Kestrel 在运行时实际绑定到的端口：

[!code-csharp[](kestrel/samples/2.x/KestrelSample/Startup.cs?name=snippet_Configure&highlight=3-4,15-21)]

在应用运行时，控制台窗口输出指示可用于访问应用的动态端口：

```console
Listening on the following addresses: http://127.0.0.1:48508
```

### <a name="limitations"></a>限制

使用以下方法配置终结点：

* <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
* `--urls` 命令行参数
* `urls` 主机配置键
* `ASPNETCORE_URLS` 环境变量

若要将代码用于 Kestrel 以外的服务器，这些方法非常有用。 不过，请注意以下限制：

* HTTPS 无法与这些方法结合使用，除非在 HTTPS 终结点配置中提供了默认证书（例如，使用 `KestrelServerOptions` 配置或配置文件，如本主题前面的部分所示）。
* 如果同时使用 `Listen` 和 `UseUrls` 方法，`Listen` 终结点将覆盖 `UseUrls` 终结点。

### <a name="iis-endpoint-configuration"></a>IIS 终结点配置

使用 IIS 时，由 `Listen` 或 `UseUrls` 设置用于 IIS 覆盖绑定的 URL 绑定。 有关详细信息，请参阅 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)主题。

## <a name="transport-configuration"></a>传输配置

对于 ASP.NET Core 2.1 版，Kestrel 默认传输不再基于 Libuv，而是基于托管的套接字。 这是 ASP.NET Core 2.0 应用升级到 2.1 时的一个重大更改，它调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*> 并依赖于以下包中的一个：

* [Microsoft.AspNetCore.Server.Kestrel](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel/)（直接包引用）
* [Microsoft.AspNetCore.App](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)

对于需要使用 Libuv 的项目：

* 将用于 [Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv/) 包的依赖项添加到应用的项目文件：

  ```xml
  <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel.Transport.Libuv"
                    Version="{VERSION}" />
  ```

* 调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderLibuvExtensions.UseLibuv*>：

  ```csharp
  public class Program
  {
      public static void Main(string[] args)
      {
          CreateWebHostBuilder(args).Build().Run();
      }

      public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
          WebHost.CreateDefaultBuilder(args)
              .UseLibuv()
              .UseStartup<Startup>();
  }
  ```

### <a name="url-prefixes"></a>URL 前缀

如果使用 `UseUrls`、`--urls` 命令行参数、`urls` 主机配置键或 `ASPNETCORE_URLS` 环境变量，URL 前缀可采用以下任意格式。

仅 HTTP URL 前缀是有效的。 使用 `UseUrls` 配置 URL 绑定时，Kestrel 不支持 HTTPS。

* 包含端口号的 IPv4 地址

  ```
  http://65.55.39.10:80/
  ```

  `0.0.0.0` 是一种绑定到所有 IPv4 地址的特殊情况。

* 包含端口号的 IPv6 地址

  ```
  http://[0:0:0:0:0:ffff:4137:270a]:80/
  ```

  `[::]` 是 IPv4 `0.0.0.0` 的 IPv6 等效项。

* 包含端口号的主机名

  ```
  http://contoso.com:80/
  http://*:80/
  ```

  主机名、`*`和 `+` 并不特殊。 没有识别为有效 IP 地址或 `localhost` 的任何内容都将绑定到所有 IPv4 和 IPv6 IP。 若要将不同主机名绑定到相同端口上的不同 ASP.NET Core 应用，请使用 [HTTP.sys](xref:fundamentals/servers/httpsys) 或 IIS、Nginx 或 Apache 等反向代理服务器。

  > [!WARNING]
  > 采用反向代理配置进行托管需要[主机筛选](#host-filtering)。

* 包含端口号的主机 `localhost` 名称或包含端口号的环回 IP

  ```
  http://localhost:5000/
  http://127.0.0.1:5000/
  http://[::1]:5000/
  ```

  指定 `localhost` 后，Kestrel 将尝试绑定到 IPv4 和 IPv6 环回接口。 如果其他服务正在任一环回接口上使用请求的端口，则 Kestrel 将无法启动。 如果任一环回接口出于任何其他原因（通常是因为 IPv6 不受支持）而不可用，则 Kestrel 将记录一个警告。

## <a name="host-filtering"></a>主机筛选

尽管 Kestrel 支持基于前缀的配置（例如 `http://example.com:5000`），但 Kestrel 在很大程度上会忽略主机名。 主机 `localhost` 是一个特殊情况，用于绑定至环回地址。 除了显式 IP 地址以外的所有主机都绑定至所有公共 IP 地址。 不验证 `Host` 标头。

解决方法是，使用主机筛选中间件。 主机筛选中间件由 [Microsoft.AspNetCore.HostFiltering](https://www.nuget.org/packages/Microsoft.AspNetCore.HostFiltering) 包提供，此包包含在 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)中（ASP.NET Core 2.1 或 2.2）。 由调用 <xref:Microsoft.AspNetCore.Builder.HostFilteringServicesExtensions.AddHostFiltering*> 的 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 添加中间件：

[!code-csharp[](kestrel/samples-snapshot/2.x/KestrelSample/Program.cs?name=snippet_Program&highlight=9)]

默认情况下，主机筛选中间件处于禁用状态。 要启用该中间件，请在 appsettings.json/appsettings.\<EnvironmentName>.json 中定义一个 `AllowedHosts` 键。 此值是以分号分隔的不带端口号的主机名列表：

appsettings.json：

```json
{
  "AllowedHosts": "example.com;localhost"
}
```

> [!NOTE]
> [转接头中间件](xref:host-and-deploy/proxy-load-balancer) 同样包含 <xref:Microsoft.AspNetCore.Builder.ForwardedHeadersOptions.AllowedHosts> 选项。 转接头中间件和主机筛选中间件具有适合不同方案的相似功能。 如果未保留 `Host` 标头，并且使用反向代理服务器或负载均衡器转接请求，则使用转接头中间件设置 `AllowedHosts` 比较合适。 将 Kestrel 用作面向公众的边缘服务器或直接转接 `Host` 标头时，使用主机筛选中间件设置 `AllowedHosts` 比较合适。
>
> 有关转接头中间件的详细信息，请参阅 <xref:host-and-deploy/proxy-load-balancer>。

::: moniker-end

## <a name="http11-request-draining"></a>HTTP/1.1 请求排出

打开 HTTP 连接非常耗时。 对于 HTTPS 而言，这也是资源密集型。 因此，Kestrel 会尝试按 HTTP/1.1 协议重新使用连接。 请求正文必须完全使用才能允许重新使用连接。 应用不会始终使用请求正文，例如 `POST` 请求，其中服务器返回重定向或 404 响应。 在 `POST` 重定向的情况下：

* 客户端可能已发送部分 `POST` 数据。
* 服务器写入 301 响应。
* 在完全读取上一个请求正文中的 `POST` 数据之前，不能将连接用于新请求。
* Kestrel 尝试排出请求正文。 排出请求正文意味着读取和丢弃数据，而不处理数据。

排出过程会在允许重新使用连接与排出任何剩余数据所用的时间之间进行权衡：

* 排出超时为五秒，该值不可配置。
* 如果 `Content-Length` 或 `Transfer-Encoding` 标头指定的所有数据在超时之前都未被读取，则连接将关闭。

有时，你可能想要在写入响应之前或之后立即终止请求。 例如，客户端可能具有限制性的数据上限，因此可以优先考虑限制上传的数据。 在这种情况下，若要终止请求，请从控制器、Razor 页面或中间件调用 [HttpContext](xref:Microsoft.AspNetCore.Http.HttpContext.Abort%2A)。

在调用 `Abort` 时，存在一些注意事项：

* 创建新连接可能会很慢且成本高昂。
* 在连接关闭之前无法保证客户端已读取响应。
* 应极少调用 `Abort`，并且应将此操作留给严重错误情况（而不是常见错误情况）。
  * 只有在需要解决特定问题时才调用 `Abort`。 例如，如果恶意客户端正在尝试 `POST` 数据或在客户端代码中存在导致大量请求的 bug，则调用 `Abort`。
  * 不要为常见错误情况（如 HTTP 404（找不到））调用 `Abort`。

调用 `Abort` 之前调用 [HttpResponse.CompleteAsync](xref:Microsoft.AspNetCore.Http.HttpResponse.CompleteAsync%2A) 可确保服务器已完成写入响应。 但是，客户端行为是不可预测的，在连接中止前它们可能未读取响应。

对于 HTTP/2，此过程又有所不同，因为协议支持在不关闭连接的情况下中止单个请求流。 五秒排出超时不适用。 如果在完成响应后存在任何未读的请求正文数据，则服务器会发送 HTTP/2 RST 帧。 其他请求正文数据帧将被忽略。

如果可能，客户端最好使用 [Expect:100-continue](https://developer.mozilla.org/docs/Web/HTTP/Status/100) 请求标头，然后等到服务器响应后再开始发送请求正文。 这样，客户端便有机会在发送不需要的数据之前检查响应和中止。

## <a name="additional-resources"></a>其他资源

* 在 Linux 上使用 UNIX 套接字时，该套接字不会在应用关闭时自动删除。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/aspnetcore/issues/14134)。
* <xref:test/troubleshoot>
* <xref:security/enforcing-ssl>
* <xref:host-and-deploy/proxy-load-balancer>
* [RFC 7230：消息语法和路由（5.4 节：主机）](https://tools.ietf.org/html/rfc7230#section-5.4)
