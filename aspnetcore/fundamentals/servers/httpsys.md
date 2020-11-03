---
title: ASP.NET Core 中的 HTTP.sys Web 服务器实现
author: rick-anderson
description: 了解 Windows 上适用于 ASP.NET Core 的 Web 服务器 HTTP.sys。 HTTP.sys 构建于 HTTP.sys 内核模式驱动程序之上，是 Kestrel 的一种替代选择，可用来直接连接到 Internet，而无需使用 IIS。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 02/07/2020
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
uid: fundamentals/servers/httpsys
ms.openlocfilehash: 8ed9ec3447205107194ffa5c329c0e5ae0fc5553
ms.sourcegitcommit: e519d95d17443abafba8f712ac168347b15c8b57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/02/2020
ms.locfileid: "91653966"
---
# <a name="httpsys-web-server-implementation-in-aspnet-core"></a>ASP.NET Core 中的 HTTP.sys Web 服务器实现

作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Chris Ross](https://github.com/Tratcher)

::: moniker range=">= aspnetcore-3.1"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) 是仅在 Windows 上运行的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 HTTP.sys 是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器的替代选择，提供了一些 Kestrel 不提供的功能。

> [!IMPORTANT]
> HTTP.sys 与 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)不兼容，无法与 IIS 或 IIS Express 结合使用。

HTTP.sys 支持以下功能：

* [Windows 身份验证](xref:security/authentication/windowsauth)
* 端口共享
* 具有 SNI 的 HTTPS
* 基于 TLS 的 HTTP/2（Windows 10 或更高版本）
* 直接文件传输
* 响应缓存
* WebSocket（Windows 8 或更高版本）

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-httpsys"></a>何时使用 HTTP.sys

HTTP.sys 对于以下情形的部署来说很有用：

* 需要将服务器直接公开到 Internet 而不使用 IIS 的部署。

  ![HTTP.sys 直接与 Internet 进行通信](httpsys/_static/httpsys-to-internet.png)

* 内部部署需要 Kestrel 中没有的功能，如 [Windows 身份验证](xref:security/authentication/windowsauth)。

  ![HTTP.sys 直接与内部网络进行通信](httpsys/_static/httpsys-to-internal.png)

HTTP.sys 是一项成熟的技术，可以抵御多种攻击，并提供可靠、安全、可伸缩的全功能 Web 服务器。 IIS 本身作为 HTTP.sys 之上的 HTTP 侦听器运行。

## <a name="http2-support"></a>HTTP/2 支持

如果满足以下基本要求，将为 ASP.NET Core 应用启用 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* Windows Server 2016/Windows 10 或更高版本
* [应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接
* TLS 1.2 或更高版本的连接

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/2`。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 在 Windows 的未来版本中，将提供 HTTP/2 配置标志，包括使用 HTTP.sys 禁用 HTTP/2 的功能。

## <a name="kernel-mode-authentication-with-kerberos"></a>对 Kerberos 进行内核模式身份验证

HTTP.sys 通过 Kerberos 身份验证协议委托给内核模式身份验证。 Kerberos 和 HTTP.sys 不支持用户模式身份验证。 必须使用计算机帐户来解密从 Active Directory 获取的并由客户端转发到服务器的 Kerberos 令牌/票证，以便对用户进行身份验证。 注册主机的服务主体名称 (SPN)，而不是应用的用户。

## <a name="how-to-use-httpsys"></a>如何使用 HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>配置 ASP.NET Core 应用以使用 HTTP.sys

生成主机时调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 扩展方法，指定所有必需的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions>。 以下示例将选项设置为其默认值：

[!code-csharp[](httpsys/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=5-13)]

通过[注册表设置](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)处理其他 HTTP.sys 配置。

**HTTP.sys 选项**

| Property | 描述 | 默认 |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | 控制是否允许 `HttpContext.Request.Body` 和 `HttpContext.Response.Body` 的同步输入/输出。 | `false` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 允许匿名请求。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 指定允许的身份验证方案。 可能在处理侦听器之前随时修改。 通过 [AuthenticationSchemes 枚举](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes)`Basic`、`Kerberos`、`Negotiate`、`None` 和 `NTLM` 提供值。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 尝试[内核模式](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)缓存，响应合格的标头。 该响应可能不包括 `Set-Cookie`、`Vary` 或 `Pragma` 标头。 它必须包括属性为 `public` 的 `Cache-Control` 标头和 `shared-max-age` 或 `max-age` 值，或 `Expires` 标头。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 最大并发接受数量。 | 5 &times; [环境。<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 要接受的最大并发连接数。 使用 `-1` 实现无限。 通过 `null` 使用注册表的计算机范围内的设置。 | `null`<br>（计算机范围内的<br>设置） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 请参阅 <a href="#maxrequestbodysize">MaxRequestBodySize</a> 部分。 | 30000000 个字节<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | 队列中允许的最大请求数。 | 1000 |
| `RequestQueueMode` | 这指示服务器是否负责创建和配置请求队列，或是否应附加到现有队列。<br>附加到现有队列时，大多数现有配置选项不适用。 | `RequestQueueMode.Create` |
| `RequestQueueName` | HTTP.sys 请求队列的名称。 | `null`（匿名队列） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 指示由于客户端断开连接而失败的响应主体写入应引发异常还是正常完成。 | `false`<br>（正常完成） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | 公开 HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 配置，也可以在注册表中进行配置。 请访问 API 链接详细了解每个设置，包括默认值：<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody)：允许 HTTP 服务器 API 在保持活动的连接上排出实体正文的时间。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody)：允许请求实体正文到达的时间。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait)：允许 HTTP 服务器 API 分析请求头的时间。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection)：允许空闲连接的时间。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond)：响应的最小发送速率。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue)：在应用选取请求前，允许请求在请求队列中停留的时间。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | 指定要向 HTTP.sys 注册的 <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection>。 最有用的是 [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*)，它用于将前缀添加到集合中。 可能在处理侦听器之前随时对这些设置进行修改。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

允许的请求正文的最大大小（以字节计）。 当设置为 `null` 时，最大请求正文大小不受限制。 此限制不会影响升级后的连接，这始终不受限制。

在 ASP.NET Core MVC 应用中为单个 `IActionResult` 替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

如果在应用开始读取请求后尝试配置请求限制，则会引发异常。 `IsReadOnly` 属性可用于指示 `MaxRequestBodySize` 属性是否处于只读状态。只读状态意味着已经太迟了，无法配置限制。

如果应用应替代每个请求的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize>，请使用 <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>：

[!code-csharp[](httpsys/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

如果使用的是 Visual Studio，请确保应用未经配置以运行 IIS 或 IIS Express。

在 Visual Studio 中，默认启动配置文件是针对 IIS Express 的。 若要作为控制台应用运行该项目，请手动更改所选配置文件，如以下屏幕截图中所示：

![选择控制台应用配置文件](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>配置 Windows Server

1. 确定要为应用打开的端口，并使用 [Windows 防火墙](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)或 [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell cmdlet 打开防火墙端口，以允许流量到达 HTTP.sys。 在以下命令和应用配置中，使用的是端口 443。

1. 在部署到 Azure VM 时，在[网络安全组](/azure/virtual-machines/windows/nsg-quickstart-portal)中打开端口。 在以下命令和应用配置中，使用的是端口 443。

1. 如果需要，获取并安装 X.509 证书。

   在 Windows 上，可使用 [New-SelfSignedCertificate PowerShell cmdlet](/powershell/module/pkiclient/new-selfsignedcertificate) 创建自签名证书。 有关不支持的示例，请参阅 [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1)。

   在服务器的“本地计算机”>“个人”存储中，安装自签名证书或 CA 签名证书 。

1. 如果应用为[框架相关部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)，则安装 .NET Core、.NET Framework 或两者（如果应用是面向 .NET Framework 的 .NET Core 应用）。

   * **.NET Core**：如果应用需要 .NET Core，请从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取并运行 .NET Core 运行时安装程序。 请勿在服务器上安装完整 SDK。
   * **.NET Framework**：如果应用需要 .NET Framework，请参阅 [.NET Framework 安装指南](/dotnet/framework/install/)。 安装所需的 .NET Framework。 可以从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取最新 .NET Framework 的安装程序。

   如果应用是[独立式部署](/dotnet/core/deploying/#self-contained-deployments-scd)，应用在部署中包含运行时。 无需在服务器上安装任何框架。

1. 在应用中配置 URL 和端口。

   默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 若要配置 URL 前缀和端口，可采用以下方法：

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` 命令行参数
   * `ASPNETCORE_URLS` 环境变量
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   下面的代码示例展示了如何对端口 443 结合使用 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> 和服务器的本地 IP 地址 `10.0.0.4`：

   [!code-csharp[](httpsys/samples_snapshot/3.x/Program.cs?highlight=7)]

   `UrlPrefixes` 的一个优点是会为格式不正确的前缀立即生成一条错误消息。

   `UrlPrefixes` 中的设置替代 `UseUrls`/`urls`/`ASPNETCORE_URLS` 设置。 因此，`UseUrls`、`urls` 和 `ASPNETCORE_URLS` 环境变量的一个优点是在 Kestrel 和 HTTP.sys 之间切换变得更加容易。

   HTTP.sys 使用 [HTTP 服务器 API UrlPrefix 字符串格式](/windows/win32/http/urlprefix-strings)。

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定会带来应用安全漏洞。 此行为同时适用于强通配符和弱通配符。 请使用显式主机名或 IP 地址，而不是通配符。 如果可控制整个父域（相对于易受攻击的 `*.com`），子域通配符绑定（例如，`*.mysub.com`）不会构成安全风险。 有关详细信息，请参阅 [RFC 7230：第 5.4 节：主机](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器上预注册 URL 前缀。

   用于配置 HTTP.sys 的内置工具为 *netsh.exe*。 *netsh.exe* 用于保留 URL 前缀并分配 X.509 证书。 此工具需要管理员特权。

   使用 netsh.exe 工具为应用注册 URL：

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`：完全限定的统一资源定位器 (URL)。 不要使用通配符绑定。 请使用有效主机名或本地 IP 地址。 URL 必须包含尾部反斜杠。
   * `<USER>`：指定用户名或用户组名称。

   在以下示例中，服务器的本地 IP 地址是 `10.0.0.4`：

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   在 URL 注册后，工具响应返回 `URL reservation successfully added`。

   若要删除已注册的 URL，请使用 `delete urlacl` 命令：

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. 在服务器上注册 X.509 证书。

   使用 netsh.exe 工具为应用注册证书：

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`：指定绑定的本地 IP 地址。 不要使用通配符绑定。 请使用有效 IP 地址。
   * `<PORT>`：指定绑定的端口。
   * `<THUMBPRINT>`：X.509 证书指纹。
   * `<GUID>`：开发人员生成的表示应用的 GUID，以供参考。

   为了便于参考，将 GUID 作为包标记存储在应用中：

   * 在 Visual Studio 中：
     * 在“解决方案资源管理器”中，右键单击应用，并选择“属性”，以打开应用的项目属性。
     * 选择“包”选项卡。
     * 在“标记”字段中输入已创建的 GUID。
   * 如果使用的不是 Visual Studio：
     * 打开应用的项目文件。
     * 使用已创建的 GUID，将 `<PackageTags>` 属性添加到新的或现有的 `<PropertyGroup>`：

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   如下示例中：

   * 服务器的本地 IP 地址是 `10.0.0.4`。
   * 联机随机 GUID 生成器提供 `appid` 值。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   在证书注册后，工具响应返回 `SSL Certificate successfully added`。

   若要删除证书注册，请使用 `delete sslcert` 命令：

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   *netsh.exe* 的参考文档：

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))（超文本传输协议 (HTTP) 的 Netsh 命令）
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings)（UrlPrefix 字符串）

1. 运行应用。

   结合使用 HTTP（而不是 HTTPS）和大于 1024 的端口号绑定到 localhost，无需管理员权限，即可运行应用。 对于其他配置（例如，使用本地 IP 地址或绑定到端口 443），必须有管理员权限才能运行应用。

   应用在服务器的公共 IP 地址处响应。 此示例在 Internet 上的公共 IP 地址 `104.214.79.47` 处访问服务器。

   此示例使用的是开发证书。 在绕过浏览器的不受信任证书警告后，页面安全加载。

   ![显示应用索引页已加载的浏览器窗口](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

如果应用由 HTTP.sys 托管并且与来自 Internet 或公司网络的请求进行交互，当在代理服务器和负载均衡器后托管时，可能需要其他配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="advanced-http2-features-to-support-grpc"></a>用于支持 gRPC 的高级 HTTP/2 功能

HTTP.sys 中的其他 HTTP/2 功能支持 gRPC，包括对响应尾部和发送重置帧的支持。

使用 HTTP.SYS 运行 gRPC 的要求：

* Windows 10，OS 内部版本 19041.508 或更高版本
* TLS 1.2 或更高版本的连接

### <a name="trailers"></a>预告片

[!INCLUDE[](~/includes/trailers.md)]

### <a name="reset"></a>重置

[!INCLUDE[](~/includes/reset.md)]

## <a name="additional-resources"></a>其他资源

* [使用 HTTP.sys 启用 Windows 身份验证](xref:security/authentication/windowsauth#httpsys)
* [HTTP 服务器 API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub 存储库（源代码）](https://github.com/aspnet/HttpSysServer/)
* [主机](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-3.0"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) 是仅在 Windows 上运行的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 HTTP.sys 是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器的替代选择，提供了一些 Kestrel 不提供的功能。

> [!IMPORTANT]
> HTTP.sys 与 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)不兼容，无法与 IIS 或 IIS Express 结合使用。

HTTP.sys 支持以下功能：

* [Windows 身份验证](xref:security/authentication/windowsauth)
* 端口共享
* 具有 SNI 的 HTTPS
* 基于 TLS 的 HTTP/2（Windows 10 或更高版本）
* 直接文件传输
* 响应缓存
* WebSocket（Windows 8 或更高版本）

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-httpsys"></a>何时使用 HTTP.sys

HTTP.sys 对于以下情形的部署来说很有用：

* 需要将服务器直接公开到 Internet 而不使用 IIS 的部署。

  ![HTTP.sys 直接与 Internet 进行通信](httpsys/_static/httpsys-to-internet.png)

* 内部部署需要 Kestrel 中没有的功能，如 [Windows 身份验证](xref:security/authentication/windowsauth)。

  ![HTTP.sys 直接与内部网络进行通信](httpsys/_static/httpsys-to-internal.png)

HTTP.sys 是一项成熟的技术，可以抵御多种攻击，并提供可靠、安全、可伸缩的全功能 Web 服务器。 IIS 本身作为 HTTP.sys 之上的 HTTP 侦听器运行。

## <a name="http2-support"></a>HTTP/2 支持

如果满足以下基本要求，将为 ASP.NET Core 应用启用 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* Windows Server 2016/Windows 10 或更高版本
* [应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接
* TLS 1.2 或更高版本的连接

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/2`。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 在 Windows 的未来版本中，将提供 HTTP/2 配置标志，包括使用 HTTP.sys 禁用 HTTP/2 的功能。

## <a name="kernel-mode-authentication-with-kerberos"></a>对 Kerberos 进行内核模式身份验证

HTTP.sys 通过 Kerberos 身份验证协议委托给内核模式身份验证。 Kerberos 和 HTTP.sys 不支持用户模式身份验证。 必须使用计算机帐户来解密从 Active Directory 获取的并由客户端转发到服务器的 Kerberos 令牌/票证，以便对用户进行身份验证。 注册主机的服务主体名称 (SPN)，而不是应用的用户。

## <a name="how-to-use-httpsys"></a>如何使用 HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>配置 ASP.NET Core 应用以使用 HTTP.sys

生成主机时调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 扩展方法，指定所有必需的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions>。 以下示例将选项设置为其默认值：

[!code-csharp[](httpsys/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=5-13)]

通过[注册表设置](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)处理其他 HTTP.sys 配置。

**HTTP.sys 选项**

| Property | 描述 | 默认 |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | 控制是否允许 `HttpContext.Request.Body` 和 `HttpContext.Response.Body` 的同步输入/输出。 | `false` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 允许匿名请求。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 指定允许的身份验证方案。 可能在处理侦听器之前随时修改。 通过 [AuthenticationSchemes 枚举](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes)`Basic`、`Kerberos`、`Negotiate`、`None` 和 `NTLM` 提供值。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 尝试[内核模式](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)缓存，响应合格的标头。 该响应可能不包括 `Set-Cookie`、`Vary` 或 `Pragma` 标头。 它必须包括属性为 `public` 的 `Cache-Control` 标头和 `shared-max-age` 或 `max-age` 值，或 `Expires` 标头。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 最大并发接受数量。 | 5 &times; [环境。<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 要接受的最大并发连接数。 使用 `-1` 实现无限。 通过 `null` 使用注册表的计算机范围内的设置。 | `null`<br>（计算机范围内的<br>设置） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 请参阅 <a href="#maxrequestbodysize">MaxRequestBodySize</a> 部分。 | 30000000 个字节<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | 队列中允许的最大请求数。 | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 指示由于客户端断开连接而失败的响应主体写入应引发异常还是正常完成。 | `false`<br>（正常完成） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | 公开 HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 配置，也可以在注册表中进行配置。 请访问 API 链接详细了解每个设置，包括默认值：<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody)：允许 HTTP 服务器 API 在保持活动的连接上排出实体正文的时间。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody)：允许请求实体正文到达的时间。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait)：允许 HTTP 服务器 API 分析请求头的时间。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection)：允许空闲连接的时间。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond)：响应的最小发送速率。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue)：在应用选取请求前，允许请求在请求队列中停留的时间。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | 指定要向 HTTP.sys 注册的 <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection>。 最有用的是 [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*)，它用于将前缀添加到集合中。 可能在处理侦听器之前随时对这些设置进行修改。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

允许的请求正文的最大大小（以字节计）。 当设置为 `null` 时，最大请求正文大小不受限制。 此限制不会影响升级后的连接，这始终不受限制。

在 ASP.NET Core MVC 应用中为单个 `IActionResult` 替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

如果在应用开始读取请求后尝试配置请求限制，则会引发异常。 `IsReadOnly` 属性可用于指示 `MaxRequestBodySize` 属性是否处于只读状态。只读状态意味着已经太迟了，无法配置限制。

如果应用应替代每个请求的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize>，请使用 <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>：

[!code-csharp[](httpsys/samples/3.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

如果使用的是 Visual Studio，请确保应用未经配置以运行 IIS 或 IIS Express。

在 Visual Studio 中，默认启动配置文件是针对 IIS Express 的。 若要作为控制台应用运行该项目，请手动更改所选配置文件，如以下屏幕截图中所示：

![选择控制台应用配置文件](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>配置 Windows Server

1. 确定要为应用打开的端口，并使用 [Windows 防火墙](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)或 [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell cmdlet 打开防火墙端口，以允许流量到达 HTTP.sys。 在以下命令和应用配置中，使用的是端口 443。

1. 在部署到 Azure VM 时，在[网络安全组](/azure/virtual-machines/windows/nsg-quickstart-portal)中打开端口。 在以下命令和应用配置中，使用的是端口 443。

1. 如果需要，获取并安装 X.509 证书。

   在 Windows 上，可使用 [New-SelfSignedCertificate PowerShell cmdlet](/powershell/module/pkiclient/new-selfsignedcertificate) 创建自签名证书。 有关不支持的示例，请参阅 [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1)。

   在服务器的“本地计算机”>“个人”存储中，安装自签名证书或 CA 签名证书 。

1. 如果应用为[框架相关部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)，则安装 .NET Core、.NET Framework 或两者（如果应用是面向 .NET Framework 的 .NET Core 应用）。

   * **.NET Core**：如果应用需要 .NET Core，请从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取并运行 .NET Core 运行时安装程序。 请勿在服务器上安装完整 SDK。
   * **.NET Framework**：如果应用需要 .NET Framework，请参阅 [.NET Framework 安装指南](/dotnet/framework/install/)。 安装所需的 .NET Framework。 可以从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取最新 .NET Framework 的安装程序。

   如果应用是[独立式部署](/dotnet/core/deploying/#self-contained-deployments-scd)，应用在部署中包含运行时。 无需在服务器上安装任何框架。

1. 在应用中配置 URL 和端口。

   默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 若要配置 URL 前缀和端口，可采用以下方法：

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` 命令行参数
   * `ASPNETCORE_URLS` 环境变量
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   下面的代码示例展示了如何对端口 443 结合使用 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> 和服务器的本地 IP 地址 `10.0.0.4`：

   [!code-csharp[](httpsys/samples_snapshot/3.x/Program.cs?highlight=7)]

   `UrlPrefixes` 的一个优点是会为格式不正确的前缀立即生成一条错误消息。

   `UrlPrefixes` 中的设置替代 `UseUrls`/`urls`/`ASPNETCORE_URLS` 设置。 因此，`UseUrls`、`urls` 和 `ASPNETCORE_URLS` 环境变量的一个优点是在 Kestrel 和 HTTP.sys 之间切换变得更加容易。

   HTTP.sys 使用 [HTTP 服务器 API UrlPrefix 字符串格式](/windows/win32/http/urlprefix-strings)。

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定会带来应用安全漏洞。 此行为同时适用于强通配符和弱通配符。 请使用显式主机名或 IP 地址，而不是通配符。 如果可控制整个父域（相对于易受攻击的 `*.com`），子域通配符绑定（例如，`*.mysub.com`）不会构成安全风险。 有关详细信息，请参阅 [RFC 7230：第 5.4 节：主机](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器上预注册 URL 前缀。

   用于配置 HTTP.sys 的内置工具为 *netsh.exe*。 *netsh.exe* 用于保留 URL 前缀并分配 X.509 证书。 此工具需要管理员特权。

   使用 netsh.exe 工具为应用注册 URL：

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`：完全限定的统一资源定位器 (URL)。 不要使用通配符绑定。 请使用有效主机名或本地 IP 地址。 URL 必须包含尾部反斜杠。
   * `<USER>`：指定用户名或用户组名称。

   在以下示例中，服务器的本地 IP 地址是 `10.0.0.4`：

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   在 URL 注册后，工具响应返回 `URL reservation successfully added`。

   若要删除已注册的 URL，请使用 `delete urlacl` 命令：

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. 在服务器上注册 X.509 证书。

   使用 netsh.exe 工具为应用注册证书：

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`：指定绑定的本地 IP 地址。 不要使用通配符绑定。 请使用有效 IP 地址。
   * `<PORT>`：指定绑定的端口。
   * `<THUMBPRINT>`：X.509 证书指纹。
   * `<GUID>`：开发人员生成的表示应用的 GUID，以供参考。

   为了便于参考，将 GUID 作为包标记存储在应用中：

   * 在 Visual Studio 中：
     * 在“解决方案资源管理器”中，右键单击应用，并选择“属性”，以打开应用的项目属性。
     * 选择“包”选项卡。
     * 在“标记”字段中输入已创建的 GUID。
   * 如果使用的不是 Visual Studio：
     * 打开应用的项目文件。
     * 使用已创建的 GUID，将 `<PackageTags>` 属性添加到新的或现有的 `<PropertyGroup>`：

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   如下示例中：

   * 服务器的本地 IP 地址是 `10.0.0.4`。
   * 联机随机 GUID 生成器提供 `appid` 值。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   在证书注册后，工具响应返回 `SSL Certificate successfully added`。

   若要删除证书注册，请使用 `delete sslcert` 命令：

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   *netsh.exe* 的参考文档：

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))（超文本传输协议 (HTTP) 的 Netsh 命令）
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings)（UrlPrefix 字符串）

1. 运行应用。

   结合使用 HTTP（而不是 HTTPS）和大于 1024 的端口号绑定到 localhost，无需管理员权限，即可运行应用。 对于其他配置（例如，使用本地 IP 地址或绑定到端口 443），必须有管理员权限才能运行应用。

   应用在服务器的公共 IP 地址处响应。 此示例在 Internet 上的公共 IP 地址 `104.214.79.47` 处访问服务器。

   此示例使用的是开发证书。 在绕过浏览器的不受信任证书警告后，页面安全加载。

   ![显示应用索引页已加载的浏览器窗口](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

如果应用由 HTTP.sys 托管并且与来自 Internet 或公司网络的请求进行交互，当在代理服务器和负载均衡器后托管时，可能需要其他配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="additional-resources"></a>其他资源

* [使用 HTTP.sys 启用 Windows 身份验证](xref:security/authentication/windowsauth#httpsys)
* [HTTP 服务器 API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub 存储库（源代码）](https://github.com/aspnet/HttpSysServer/)
* [主机](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) 是仅在 Windows 上运行的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 HTTP.sys 是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器的替代选择，提供了一些 Kestrel 不提供的功能。

> [!IMPORTANT]
> HTTP.sys 与 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)不兼容，无法与 IIS 或 IIS Express 结合使用。

HTTP.sys 支持以下功能：

* [Windows 身份验证](xref:security/authentication/windowsauth)
* 端口共享
* 具有 SNI 的 HTTPS
* 基于 TLS 的 HTTP/2（Windows 10 或更高版本）
* 直接文件传输
* 响应缓存
* WebSocket（Windows 8 或更高版本）

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-httpsys"></a>何时使用 HTTP.sys

HTTP.sys 对于以下情形的部署来说很有用：

* 需要将服务器直接公开到 Internet 而不使用 IIS 的部署。

  ![HTTP.sys 直接与 Internet 进行通信](httpsys/_static/httpsys-to-internet.png)

* 内部部署需要 Kestrel 中没有的功能，如 [Windows 身份验证](xref:security/authentication/windowsauth)。

  ![HTTP.sys 直接与内部网络进行通信](httpsys/_static/httpsys-to-internal.png)

HTTP.sys 是一项成熟的技术，可以抵御多种攻击，并提供可靠、安全、可伸缩的全功能 Web 服务器。 IIS 本身作为 HTTP.sys 之上的 HTTP 侦听器运行。

## <a name="http2-support"></a>HTTP/2 支持

如果满足以下基本要求，将为 ASP.NET Core 应用启用 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* Windows Server 2016/Windows 10 或更高版本
* [应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接
* TLS 1.2 或更高版本的连接

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/2`。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 在 Windows 的未来版本中，将提供 HTTP/2 配置标志，包括使用 HTTP.sys 禁用 HTTP/2 的功能。

## <a name="kernel-mode-authentication-with-kerberos"></a>对 Kerberos 进行内核模式身份验证

HTTP.sys 通过 Kerberos 身份验证协议委托给内核模式身份验证。 Kerberos 和 HTTP.sys 不支持用户模式身份验证。 必须使用计算机帐户来解密从 Active Directory 获取的并由客户端转发到服务器的 Kerberos 令牌/票证，以便对用户进行身份验证。 注册主机的服务主体名称 (SPN)，而不是应用的用户。

## <a name="how-to-use-httpsys"></a>如何使用 HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>配置 ASP.NET Core 应用以使用 HTTP.sys

使用 [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) ([nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)) 时，不需要项目文件中的包引用。 未使用 `Microsoft.AspNetCore.App` 元包时，向 [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) 添加包引用。

生成主机时调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 扩展方法，指定所有必需的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions>。 以下示例将选项设置为其默认值：

[!code-csharp[](httpsys/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=4-12)]

通过[注册表设置](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)处理其他 HTTP.sys 配置。

**HTTP.sys 选项**

| Property | 描述 | 默认 |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | 控制是否允许 `HttpContext.Request.Body` 和 `HttpContext.Response.Body` 的同步输入/输出。 | `true` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 允许匿名请求。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 指定允许的身份验证方案。 可能在处理侦听器之前随时修改。 通过 [AuthenticationSchemes 枚举](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes)`Basic`、`Kerberos`、`Negotiate`、`None` 和 `NTLM` 提供值。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 尝试[内核模式](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)缓存，响应合格的标头。 该响应可能不包括 `Set-Cookie`、`Vary` 或 `Pragma` 标头。 它必须包括属性为 `public` 的 `Cache-Control` 标头和 `shared-max-age` 或 `max-age` 值，或 `Expires` 标头。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 最大并发接受数量。 | 5 &times; [环境。<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 要接受的最大并发连接数。 使用 `-1` 实现无限。 通过 `null` 使用注册表的计算机范围内的设置。 | `null`<br>（计算机范围内的<br>设置） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 请参阅 <a href="#maxrequestbodysize">MaxRequestBodySize</a> 部分。 | 30000000 个字节<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | 队列中允许的最大请求数。 | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 指示由于客户端断开连接而失败的响应主体写入应引发异常还是正常完成。 | `false`<br>（正常完成） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | 公开 HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 配置，也可以在注册表中进行配置。 请访问 API 链接详细了解每个设置，包括默认值：<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody)：允许 HTTP 服务器 API 在保持活动的连接上排出实体正文的时间。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody)：允许请求实体正文到达的时间。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait)：允许 HTTP 服务器 API 分析请求头的时间。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection)：允许空闲连接的时间。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond)：响应的最小发送速率。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue)：在应用选取请求前，允许请求在请求队列中停留的时间。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | 指定要向 HTTP.sys 注册的 <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection>。 最有用的是 [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*)，它用于将前缀添加到集合中。 可能在处理侦听器之前随时对这些设置进行修改。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

允许的请求正文的最大大小（以字节计）。 当设置为 `null` 时，最大请求正文大小不受限制。 此限制不会影响升级后的连接，这始终不受限制。

在 ASP.NET Core MVC 应用中为单个 `IActionResult` 替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

如果在应用开始读取请求后尝试配置请求限制，则会引发异常。 `IsReadOnly` 属性可用于指示 `MaxRequestBodySize` 属性是否处于只读状态。只读状态意味着已经太迟了，无法配置限制。

如果应用应替代每个请求的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize>，请使用 <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>：

[!code-csharp[](httpsys/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

如果使用的是 Visual Studio，请确保应用未经配置以运行 IIS 或 IIS Express。

在 Visual Studio 中，默认启动配置文件是针对 IIS Express 的。 若要作为控制台应用运行该项目，请手动更改所选配置文件，如以下屏幕截图中所示：

![选择控制台应用配置文件](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>配置 Windows Server

1. 确定要为应用打开的端口，并使用 [Windows 防火墙](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)或 [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell cmdlet 打开防火墙端口，以允许流量到达 HTTP.sys。 在以下命令和应用配置中，使用的是端口 443。

1. 在部署到 Azure VM 时，在[网络安全组](/azure/virtual-machines/windows/nsg-quickstart-portal)中打开端口。 在以下命令和应用配置中，使用的是端口 443。

1. 如果需要，获取并安装 X.509 证书。

   在 Windows 上，可使用 [New-SelfSignedCertificate PowerShell cmdlet](/powershell/module/pkiclient/new-selfsignedcertificate) 创建自签名证书。 有关不支持的示例，请参阅 [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1)。

   在服务器的“本地计算机”>“个人”存储中，安装自签名证书或 CA 签名证书 。

1. 如果应用为[框架相关部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)，则安装 .NET Core、.NET Framework 或两者（如果应用是面向 .NET Framework 的 .NET Core 应用）。

   * **.NET Core**：如果应用需要 .NET Core，请从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取并运行 .NET Core 运行时安装程序。 请勿在服务器上安装完整 SDK。
   * **.NET Framework**：如果应用需要 .NET Framework，请参阅 [.NET Framework 安装指南](/dotnet/framework/install/)。 安装所需的 .NET Framework。 可以从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取最新 .NET Framework 的安装程序。

   如果应用是[独立式部署](/dotnet/core/deploying/#self-contained-deployments-scd)，应用在部署中包含运行时。 无需在服务器上安装任何框架。

1. 在应用中配置 URL 和端口。

   默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 若要配置 URL 前缀和端口，可采用以下方法：

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` 命令行参数
   * `ASPNETCORE_URLS` 环境变量
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   下面的代码示例展示了如何对端口 443 结合使用 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> 和服务器的本地 IP 地址 `10.0.0.4`：

   [!code-csharp[](httpsys/samples_snapshot/2.x/Program.cs?highlight=6)]

   `UrlPrefixes` 的一个优点是会为格式不正确的前缀立即生成一条错误消息。

   `UrlPrefixes` 中的设置替代 `UseUrls`/`urls`/`ASPNETCORE_URLS` 设置。 因此，`UseUrls`、`urls` 和 `ASPNETCORE_URLS` 环境变量的一个优点是在 Kestrel 和 HTTP.sys 之间切换变得更加容易。

   HTTP.sys 使用 [HTTP 服务器 API UrlPrefix 字符串格式](/windows/win32/http/urlprefix-strings)。

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定会带来应用安全漏洞。 此行为同时适用于强通配符和弱通配符。 请使用显式主机名或 IP 地址，而不是通配符。 如果可控制整个父域（相对于易受攻击的 `*.com`），子域通配符绑定（例如，`*.mysub.com`）不会构成安全风险。 有关详细信息，请参阅 [RFC 7230：第 5.4 节：主机](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器上预注册 URL 前缀。

   用于配置 HTTP.sys 的内置工具为 *netsh.exe*。 *netsh.exe* 用于保留 URL 前缀并分配 X.509 证书。 此工具需要管理员特权。

   使用 netsh.exe 工具为应用注册 URL：

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`：完全限定的统一资源定位器 (URL)。 不要使用通配符绑定。 请使用有效主机名或本地 IP 地址。 URL 必须包含尾部反斜杠。
   * `<USER>`：指定用户名或用户组名称。

   在以下示例中，服务器的本地 IP 地址是 `10.0.0.4`：

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   在 URL 注册后，工具响应返回 `URL reservation successfully added`。

   若要删除已注册的 URL，请使用 `delete urlacl` 命令：

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. 在服务器上注册 X.509 证书。

   使用 netsh.exe 工具为应用注册证书：

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`：指定绑定的本地 IP 地址。 不要使用通配符绑定。 请使用有效 IP 地址。
   * `<PORT>`：指定绑定的端口。
   * `<THUMBPRINT>`：X.509 证书指纹。
   * `<GUID>`：开发人员生成的表示应用的 GUID，以供参考。

   为了便于参考，将 GUID 作为包标记存储在应用中：

   * 在 Visual Studio 中：
     * 在“解决方案资源管理器”中，右键单击应用，并选择“属性”，以打开应用的项目属性。
     * 选择“包”选项卡。
     * 在“标记”字段中输入已创建的 GUID。
   * 如果使用的不是 Visual Studio：
     * 打开应用的项目文件。
     * 使用已创建的 GUID，将 `<PackageTags>` 属性添加到新的或现有的 `<PropertyGroup>`：

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   如下示例中：

   * 服务器的本地 IP 地址是 `10.0.0.4`。
   * 联机随机 GUID 生成器提供 `appid` 值。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   在证书注册后，工具响应返回 `SSL Certificate successfully added`。

   若要删除证书注册，请使用 `delete sslcert` 命令：

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   *netsh.exe* 的参考文档：

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))（超文本传输协议 (HTTP) 的 Netsh 命令）
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings)（UrlPrefix 字符串）

1. 运行应用。

   结合使用 HTTP（而不是 HTTPS）和大于 1024 的端口号绑定到 localhost，无需管理员权限，即可运行应用。 对于其他配置（例如，使用本地 IP 地址或绑定到端口 443），必须有管理员权限才能运行应用。

   应用在服务器的公共 IP 地址处响应。 此示例在 Internet 上的公共 IP 地址 `104.214.79.47` 处访问服务器。

   此示例使用的是开发证书。 在绕过浏览器的不受信任证书警告后，页面安全加载。

   ![显示应用索引页已加载的浏览器窗口](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

如果应用由 HTTP.sys 托管并且与来自 Internet 或公司网络的请求进行交互，当在代理服务器和负载均衡器后托管时，可能需要其他配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="additional-resources"></a>其他资源

* [使用 HTTP.sys 启用 Windows 身份验证](xref:security/authentication/windowsauth#httpsys)
* [HTTP 服务器 API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub 存储库（源代码）](https://github.com/aspnet/HttpSysServer/)
* [主机](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

[HTTP.sys](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#hypertext-transfer-protocol-stack-httpsys) 是仅在 Windows 上运行的[适用于 ASP.NET Core 的 Web 服务器](xref:fundamentals/servers/index)。 HTTP.sys 是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器的替代选择，提供了一些 Kestrel 不提供的功能。

> [!IMPORTANT]
> HTTP.sys 与 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)不兼容，无法与 IIS 或 IIS Express 结合使用。

HTTP.sys 支持以下功能：

* [Windows 身份验证](xref:security/authentication/windowsauth)
* 端口共享
* 具有 SNI 的 HTTPS
* 基于 TLS 的 HTTP/2（Windows 10 或更高版本）
* 直接文件传输
* 响应缓存
* WebSocket（Windows 8 或更高版本）

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/servers/httpsys/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-httpsys"></a>何时使用 HTTP.sys

HTTP.sys 对于以下情形的部署来说很有用：

* 需要将服务器直接公开到 Internet 而不使用 IIS 的部署。

  ![HTTP.sys 直接与 Internet 进行通信](httpsys/_static/httpsys-to-internet.png)

* 内部部署需要 Kestrel 中没有的功能，如 [Windows 身份验证](xref:security/authentication/windowsauth)。

  ![HTTP.sys 直接与内部网络进行通信](httpsys/_static/httpsys-to-internal.png)

HTTP.sys 是一项成熟的技术，可以抵御多种攻击，并提供可靠、安全、可伸缩的全功能 Web 服务器。 IIS 本身作为 HTTP.sys 之上的 HTTP 侦听器运行。

## <a name="http2-support"></a>HTTP/2 支持

如果满足以下基本要求，将为 ASP.NET Core 应用启用 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* Windows Server 2016/Windows 10 或更高版本
* [应用程序层协议协商 (ALPN)](https://tools.ietf.org/html/rfc7301#section-3) 连接
* TLS 1.2 或更高版本的连接

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/1.1`。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 在 Windows 的未来版本中，将提供 HTTP/2 配置标志，包括使用 HTTP.sys 禁用 HTTP/2 的功能。

## <a name="kernel-mode-authentication-with-kerberos"></a>对 Kerberos 进行内核模式身份验证

HTTP.sys 通过 Kerberos 身份验证协议委托给内核模式身份验证。 Kerberos 和 HTTP.sys 不支持用户模式身份验证。 必须使用计算机帐户来解密从 Active Directory 获取的并由客户端转发到服务器的 Kerberos 令牌/票证，以便对用户进行身份验证。 注册主机的服务主体名称 (SPN)，而不是应用的用户。

## <a name="how-to-use-httpsys"></a>如何使用 HTTP.sys

### <a name="configure-the-aspnet-core-app-to-use-httpsys"></a>配置 ASP.NET Core 应用以使用 HTTP.sys

使用 [Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) ([nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.App/)) 时，不需要项目文件中的包引用。 未使用 `Microsoft.AspNetCore.App` 元包时，向 [Microsoft.AspNetCore.Server.HttpSys](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.HttpSys/) 添加包引用。

生成主机时调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderHttpSysExtensions.UseHttpSys*> 扩展方法，指定所有必需的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions>。 以下示例将选项设置为其默认值：

[!code-csharp[](httpsys/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=4-12)]

通过[注册表设置](https://support.microsoft.com/help/820129/http-sys-registry-settings-for-windows)处理其他 HTTP.sys 配置。

**HTTP.sys 选项**

| Property | 描述 | 默认 |
| -------- | ----------- | :-----: |
| [AllowSynchronousIO](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.AllowSynchronousIO) | 控制是否允许 `HttpContext.Request.Body` 和 `HttpContext.Response.Body` 的同步输入/输出。 | `true` |
| [Authentication.AllowAnonymous](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.AllowAnonymous) | 允许匿名请求。 | `true` |
| [Authentication.Schemes](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationManager.Schemes) | 指定允许的身份验证方案。 可能在处理侦听器之前随时修改。 通过 [AuthenticationSchemes 枚举](xref:Microsoft.AspNetCore.Server.HttpSys.AuthenticationSchemes)`Basic`、`Kerberos`、`Negotiate`、`None` 和 `NTLM` 提供值。 | `None` |
| [EnableResponseCaching](xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.EnableResponseCaching) | 尝试[内核模式](/windows-hardware/drivers/gettingstarted/user-mode-and-kernel-mode)缓存，响应合格的标头。 该响应可能不包括 `Set-Cookie`、`Vary` 或 `Pragma` 标头。 它必须包括属性为 `public` 的 `Cache-Control` 标头和 `shared-max-age` 或 `max-age` 值，或 `Expires` 标头。 | `true` |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxAccepts> | 最大并发接受数量。 | 5 &times; [环境。<br>ProcessorCount](xref:System.Environment.ProcessorCount) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxConnections> | 要接受的最大并发连接数。 使用 `-1` 实现无限。 通过 `null` 使用注册表的计算机范围内的设置。 | `null`<br>（计算机范围内的<br>设置） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize> | 请参阅 <a href="#maxrequestbodysize">MaxRequestBodySize</a> 部分。 | 30000000 个字节<br>(~28.6 MB) |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.RequestQueueLimit> | 队列中允许的最大请求数。 | 1000 |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.ThrowWriteExceptions> | 指示由于客户端断开连接而失败的响应主体写入应引发异常还是正常完成。 | `false`<br>（正常完成） |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.Timeouts> | 公开 HTTP.sys <xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager> 配置，也可以在注册表中进行配置。 请访问 API 链接详细了解每个设置，包括默认值：<ul><li>[TimeoutManager.DrainEntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.DrainEntityBody)：允许 HTTP 服务器 API 在保持活动的连接上排出实体正文的时间。</li><li>[TimeoutManager.EntityBody](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.EntityBody)：允许请求实体正文到达的时间。</li><li>[TimeoutManager.HeaderWait](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.HeaderWait)：允许 HTTP 服务器 API 分析请求头的时间。</li><li>[TimeoutManager.IdleConnection](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.IdleConnection)：允许空闲连接的时间。</li><li>[TimeoutManager.MinSendBytesPerSecond](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.MinSendBytesPerSecond)：响应的最小发送速率。</li><li>[TimeoutManager.RequestQueue](xref:Microsoft.AspNetCore.Server.HttpSys.TimeoutManager.RequestQueue)：在应用选取请求前，允许请求在请求队列中停留的时间。</li></ul> |  |
| <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> | 指定要向 HTTP.sys 注册的 <xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection>。 最有用的是 [UrlPrefixCollection.Add](xref:Microsoft.AspNetCore.Server.HttpSys.UrlPrefixCollection.Add*)，它用于将前缀添加到集合中。 可能在处理侦听器之前随时对这些设置进行修改。 |  |

<a name="maxrequestbodysize"></a>

**MaxRequestBodySize**

允许的请求正文的最大大小（以字节计）。 当设置为 `null` 时，最大请求正文大小不受限制。 此限制不会影响升级后的连接，这始终不受限制。

在 ASP.NET Core MVC 应用中为单个 `IActionResult` 替代限制的推荐方法是在操作方法上使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 属性：

```csharp
[RequestSizeLimit(100000000)]
public IActionResult MyActionMethod()
```

如果在应用开始读取请求后尝试配置请求限制，则会引发异常。 `IsReadOnly` 属性可用于指示 `MaxRequestBodySize` 属性是否处于只读状态。只读状态意味着已经太迟了，无法配置限制。

如果应用应替代每个请求的 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.MaxRequestBodySize>，请使用 <xref:Microsoft.AspNetCore.Http.Features.IHttpMaxRequestBodySizeFeature>：

[!code-csharp[](httpsys/samples/2.x/SampleApp/Startup.cs?name=snippet1&highlight=6-7)]

如果使用的是 Visual Studio，请确保应用未经配置以运行 IIS 或 IIS Express。

在 Visual Studio 中，默认启动配置文件是针对 IIS Express 的。 若要作为控制台应用运行该项目，请手动更改所选配置文件，如以下屏幕截图中所示：

![选择控制台应用配置文件](httpsys/_static/vs-choose-profile.png)

### <a name="configure-windows-server"></a>配置 Windows Server

1. 确定要为应用打开的端口，并使用 [Windows 防火墙](/windows/security/threat-protection/windows-firewall/create-an-inbound-port-rule)或 [New-NetFirewallRule](/powershell/module/netsecurity/new-netfirewallrule) PowerShell cmdlet 打开防火墙端口，以允许流量到达 HTTP.sys。 在以下命令和应用配置中，使用的是端口 443。

1. 在部署到 Azure VM 时，在[网络安全组](/azure/virtual-machines/windows/nsg-quickstart-portal)中打开端口。 在以下命令和应用配置中，使用的是端口 443。

1. 如果需要，获取并安装 X.509 证书。

   在 Windows 上，可使用 [New-SelfSignedCertificate PowerShell cmdlet](/powershell/module/pkiclient/new-selfsignedcertificate) 创建自签名证书。 有关不支持的示例，请参阅 [UpdateIISExpressSSLForChrome.ps1](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/includes/make-x509-cert/UpdateIISExpressSSLForChrome.ps1)。

   在服务器的“本地计算机”>“个人”存储中，安装自签名证书或 CA 签名证书 。

1. 如果应用为[框架相关部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)，则安装 .NET Core、.NET Framework 或两者（如果应用是面向 .NET Framework 的 .NET Core 应用）。

   * **.NET Core**：如果应用需要 .NET Core，请从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取并运行 .NET Core 运行时安装程序。 请勿在服务器上安装完整 SDK。
   * **.NET Framework**：如果应用需要 .NET Framework，请参阅 [.NET Framework 安装指南](/dotnet/framework/install/)。 安装所需的 .NET Framework。 可以从 [.NET Core 下载](https://dotnet.microsoft.com/download)页获取最新 .NET Framework 的安装程序。

   如果应用是[独立式部署](/dotnet/core/deploying/#self-contained-deployments-scd)，应用在部署中包含运行时。 无需在服务器上安装任何框架。

1. 在应用中配置 URL 和端口。

   默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 若要配置 URL 前缀和端口，可采用以下方法：

   * <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*>
   * `urls` 命令行参数
   * `ASPNETCORE_URLS` 环境变量
   * <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes>

   下面的代码示例展示了如何对端口 443 结合使用 <xref:Microsoft.AspNetCore.Server.HttpSys.HttpSysOptions.UrlPrefixes> 和服务器的本地 IP 地址 `10.0.0.4`：

   [!code-csharp[](httpsys/samples_snapshot/2.x/Program.cs?highlight=6)]

   `UrlPrefixes` 的一个优点是会为格式不正确的前缀立即生成一条错误消息。

   `UrlPrefixes` 中的设置替代 `UseUrls`/`urls`/`ASPNETCORE_URLS` 设置。 因此，`UseUrls`、`urls` 和 `ASPNETCORE_URLS` 环境变量的一个优点是在 Kestrel 和 HTTP.sys 之间切换变得更加容易。

   HTTP.sys 使用 [HTTP 服务器 API UrlPrefix 字符串格式](/windows/win32/http/urlprefix-strings)。

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定会带来应用安全漏洞。 此行为同时适用于强通配符和弱通配符。 请使用显式主机名或 IP 地址，而不是通配符。 如果可控制整个父域（相对于易受攻击的 `*.com`），子域通配符绑定（例如，`*.mysub.com`）不会构成安全风险。 有关详细信息，请参阅 [RFC 7230：第 5.4 节：主机](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器上预注册 URL 前缀。

   用于配置 HTTP.sys 的内置工具为 *netsh.exe*。 *netsh.exe* 用于保留 URL 前缀并分配 X.509 证书。 此工具需要管理员特权。

   使用 netsh.exe 工具为应用注册 URL：

   ```console
   netsh http add urlacl url=<URL> user=<USER>
   ```

   * `<URL>`：完全限定的统一资源定位器 (URL)。 不要使用通配符绑定。 请使用有效主机名或本地 IP 地址。 URL 必须包含尾部反斜杠。
   * `<USER>`：指定用户名或用户组名称。

   在以下示例中，服务器的本地 IP 地址是 `10.0.0.4`：

   ```console
   netsh http add urlacl url=https://10.0.0.4:443/ user=Users
   ```

   在 URL 注册后，工具响应返回 `URL reservation successfully added`。

   若要删除已注册的 URL，请使用 `delete urlacl` 命令：

   ```console
   netsh http delete urlacl url=<URL>
   ```

1. 在服务器上注册 X.509 证书。

   使用 netsh.exe 工具为应用注册证书：

   ```console
   netsh http add sslcert ipport=<IP>:<PORT> certhash=<THUMBPRINT> appid="{<GUID>}"
   ```

   * `<IP>`：指定绑定的本地 IP 地址。 不要使用通配符绑定。 请使用有效 IP 地址。
   * `<PORT>`：指定绑定的端口。
   * `<THUMBPRINT>`：X.509 证书指纹。
   * `<GUID>`：开发人员生成的表示应用的 GUID，以供参考。

   为了便于参考，将 GUID 作为包标记存储在应用中：

   * 在 Visual Studio 中：
     * 在“解决方案资源管理器”中，右键单击应用，并选择“属性”，以打开应用的项目属性。
     * 选择“包”选项卡。
     * 在“标记”字段中输入已创建的 GUID。
   * 如果使用的不是 Visual Studio：
     * 打开应用的项目文件。
     * 使用已创建的 GUID，将 `<PackageTags>` 属性添加到新的或现有的 `<PropertyGroup>`：

       ```xml
       <PropertyGroup>
         <PackageTags>9412ee86-c21b-4eb8-bd89-f650fbf44931</PackageTags>
       </PropertyGroup>
       ```

   如下示例中：

   * 服务器的本地 IP 地址是 `10.0.0.4`。
   * 联机随机 GUID 生成器提供 `appid` 值。

   ```console
   netsh http add sslcert 
       ipport=10.0.0.4:443 
       certhash=b66ee04419d4ee37464ab8785ff02449980eae10 
       appid="{9412ee86-c21b-4eb8-bd89-f650fbf44931}"
   ```

   在证书注册后，工具响应返回 `SSL Certificate successfully added`。

   若要删除证书注册，请使用 `delete sslcert` 命令：

   ```console
   netsh http delete sslcert ipport=<IP>:<PORT>
   ```

   *netsh.exe* 的参考文档：

   * [Netsh Commands for Hypertext Transfer Protocol (HTTP)](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc725882(v=ws.10))（超文本传输协议 (HTTP) 的 Netsh 命令）
   * [UrlPrefix Strings](/windows/win32/http/urlprefix-strings)（UrlPrefix 字符串）

1. 运行应用。

   结合使用 HTTP（而不是 HTTPS）和大于 1024 的端口号绑定到 localhost，无需管理员权限，即可运行应用。 对于其他配置（例如，使用本地 IP 地址或绑定到端口 443），必须有管理员权限才能运行应用。

   应用在服务器的公共 IP 地址处响应。 此示例在 Internet 上的公共 IP 地址 `104.214.79.47` 处访问服务器。

   此示例使用的是开发证书。 在绕过浏览器的不受信任证书警告后，页面安全加载。

   ![显示应用索引页已加载的浏览器窗口](httpsys/_static/browser.png)

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

如果应用由 HTTP.sys 托管并且与来自 Internet 或公司网络的请求进行交互，当在代理服务器和负载均衡器后托管时，可能需要其他配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

## <a name="additional-resources"></a>其他资源

* [使用 HTTP.sys 启用 Windows 身份验证](xref:security/authentication/windowsauth#httpsys)
* [HTTP 服务器 API](/windows/win32/http/http-api-start-page)
* [aspnet/HttpSysServer GitHub 存储库（源代码）](https://github.com/aspnet/HttpSysServer/)
* [主机](xref:fundamentals/index#host)
* <xref:test/troubleshoot>

::: moniker-end
