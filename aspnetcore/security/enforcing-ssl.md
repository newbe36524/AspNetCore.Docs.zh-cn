---
title: 在 ASP.NET Core 强制实施 HTTPS
author: rick-anderson
description: 了解如何在 ASP.NET Core web 应用中要求使用 HTTPS/TLS。
ms.author: riande
ms.custom: mvc
ms.date: 12/06/2019
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
uid: security/enforcing-ssl
ms.openlocfilehash: e473da9a7cbd91a601ad4af0c7c02c7f576f348c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051117"
---
# <a name="enforce-https-in-aspnet-core"></a>在 ASP.NET Core 强制实施 HTTPS

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

本文档介绍如何执行以下操作：

* 所有请求都需要 HTTPS。
* 将所有 HTTP 请求重定向到 HTTPS。

任何 API 都不能阻止客户端发送第一个请求上的敏感数据。

::: moniker range=">= aspnetcore-3.0"

> [!WARNING]
> ## <a name="api-projects"></a>API 项目
>
> 不要 **对** 接收敏感信息的 Web Api 使用 [RequireHttpsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.requirehttpsattribute) 。 `RequireHttpsAttribute` 使用 HTTP 状态代码将浏览器从 HTTP 重定向到 HTTPS。 API 客户端可能不理解或遵循从 HTTP 到 HTTPS 的重定向。 此类客户端可以通过 HTTP 发送信息。 Web Api 应：
>
> * 不侦听 HTTP。
> * 关闭状态代码为400的连接 (错误请求) 并且不处理请求。
>
> ## <a name="hsts-and-api-projects"></a>HSTS 和 API 项目
>
> 默认 API 项目不包括 [HSTS](#hsts) ，因为 HSTS 通常是仅限浏览器的指令。 其他调用方（如电话或桌面应用程序） **不** 遵守说明。 即使是在浏览器中，通过 HTTP 对 API 进行单个身份验证调用也会对不安全网络产生风险。 安全方法是将 API 项目配置为仅侦听并通过 HTTPS 进行响应。

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

> [!WARNING]
> ## <a name="api-projects"></a>API 项目
>
> 不要 **对** 接收敏感信息的 Web Api 使用 [RequireHttpsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.requirehttpsattribute) 。 `RequireHttpsAttribute` 使用 HTTP 状态代码将浏览器从 HTTP 重定向到 HTTPS。 API 客户端可能不理解或遵循从 HTTP 到 HTTPS 的重定向。 此类客户端可以通过 HTTP 发送信息。 Web Api 应：
>
> * 不侦听 HTTP。
> * 关闭状态代码为400的连接 (错误请求) 并且不处理请求。

::: moniker-end

## <a name="require-https"></a>需要 HTTPS

建议将生产 ASP.NET Core web 应用使用：

* HTTPS 重定向中间件 (<xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection*>) 将 HTTP 请求重定向到 HTTPS。
* HSTS 中间件 ([UseHsts](#http-strict-transport-security-protocol-hsts)) 将 HTTP 严格传输安全协议 (HSTS) 标头发送到客户端。

> [!NOTE]
> 使用反向代理配置部署的应用允许代理处理 (HTTPS) 的连接安全。 如果代理还处理 HTTPS 重定向，则无需使用 HTTPS 重定向中间件。 如果代理服务器还处理写入 HSTS 标头 (例如， [IIS 10.0 (1709) 或更高版本) 中的本机 HSTS 支持](/iis/get-started/whats-new-in-iis-10-version-1709/iis-10-version-1709-hsts#iis-100-version-1709-native-hsts-support) ，则该应用程序不要求 HSTS 中间件。 有关详细信息，请参阅 [在创建项目时选择退出 HTTPS/HSTS](#opt-out-of-httpshsts-on-project-creation)。

### <a name="usehttpsredirection"></a>UseHttpsRedirection

下面的代码 `UseHttpsRedirection` 在类中调用 `Startup` ：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](enforcing-ssl/sample-snapshot/3.x/Startup.cs?name=snippet1&highlight=14)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](enforcing-ssl/sample-snapshot/2.x/Startup.cs?name=snippet1&highlight=13)]

::: moniker-end

前面突出显示的代码：

* 使用默认的 [HttpsRedirectionOptions. RedirectStatusCode](/dotnet/api/microsoft.aspnetcore.httpspolicy.httpsredirectionoptions.redirectstatuscode) ([Status307TemporaryRedirect](/dotnet/api/microsoft.aspnetcore.http.statuscodes.status307temporaryredirect)) 。
* 使用默认的 [HttpsRedirectionOptions. HttpsPort](/dotnet/api/microsoft.aspnetcore.httpspolicy.httpsredirectionoptions.httpsport) (null) ，除非由 `ASPNETCORE_HTTPS_PORT` 环境变量或 [IServerAddressesFeature](/dotnet/api/microsoft.aspnetcore.hosting.server.features.iserveraddressesfeature)重写。

建议使用临时重定向，而不是永久重定向。 链接缓存会导致开发环境中的行为不稳定。 如果希望在应用处于非开发环境中时发送永久重定向状态代码，请参阅在 [生产中配置永久重定向](#configure-permanent-redirects-in-production) 部分。 我们建议使用 [HSTS](#http-strict-transport-security-protocol-hsts) 来向仅应将安全资源请求发送到应用的客户端发出信号， (仅在生产) 中发送。

### <a name="port-configuration"></a>端口配置

端口必须可用于中间件，以将不安全的请求重定向到 HTTPS。 如果没有可用的端口：

* 不会重定向到 HTTPS。
* 中间件记录警告 "无法确定用于重定向的 https 端口"。

使用以下任一方法指定 HTTPS 端口：

* 设置 [HttpsRedirectionOptions. HttpsPort](#options)。

::: moniker range=">= aspnetcore-3.0"

* 设置 `https_port` [主机设置](../fundamentals/host/generic-host.md?view=aspnetcore-3.0#https_port)：

  * 在 "主机配置" 中。
  * 通过设置 `ASPNETCORE_HTTPS_PORT` 环境变量。
  * 通过在中添加顶级条目 *appsettings.json* ：

    [!code-json[](enforcing-ssl/sample-snapshot/3.x/appsettings.json?highlight=2)]

* 使用 [ASPNETCORE_URLS 环境变量](../fundamentals/host/generic-host.md?view=aspnetcore-3.0#urls)指示包含安全方案的端口。 环境变量配置服务器。 中间件通过间接发现 HTTPS 端口 <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature> 。 此方法在反向代理部署中不起作用。

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

* 设置 `https_port` [主机设置](xref:fundamentals/host/web-host#https-port)：

  * 在 "主机配置" 中。
  * 通过设置 `ASPNETCORE_HTTPS_PORT` 环境变量。
  * 通过在中添加顶级条目 *appsettings.json* ：

    [!code-json[](enforcing-ssl/sample-snapshot/2.x/appsettings.json?highlight=2)]

* 使用 [ASPNETCORE_URLS 环境变量](xref:fundamentals/host/web-host#server-urls)指示包含安全方案的端口。 环境变量配置服务器。 中间件通过间接发现 HTTPS 端口 <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature> 。 此方法在反向代理部署中不起作用。

::: moniker-end

* 在开发中，在 *launchsettings.js上* 设置 HTTPS URL。 当使用 IIS Express 时，启用 HTTPS。

* 为 [Kestrel](xref:fundamentals/servers/kestrel) 服务器或 [HTTP.sys](xref:fundamentals/servers/httpsys) 服务器的面向公众的边缘部署配置 HTTPS URL 终结点。 此应用只使用 **一个 HTTPS 端口** 。 中间件通过发现端口 <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature> 。

> [!NOTE]
> 当应用在反向代理配置中运行时， <xref:Microsoft.AspNetCore.Hosting.Server.Features.IServerAddressesFeature> 不可用。 使用本部分中所述的其他方法之一设置端口。

### <a name="edge-deployments"></a>Edge 部署 

当 Kestrel 或 HTTP.sys 用作面向公众的边缘服务器时，必须将 Kestrel 或 HTTP.sys 配置为侦听两者：

* 重定向客户端的安全端口 (通常是443在生产和5001开发) 。
* 在生产环境中，80 (通常为，开发5000环境中) 。

客户端必须能够访问不安全的端口，以便应用接收不安全的请求，并将客户端重定向到安全端口。

有关详细信息，请参阅 [Kestrel 终结点配置](xref:fundamentals/servers/kestrel#endpoint-configuration) 或 <xref:fundamentals/servers/httpsys> 。

### <a name="deployment-scenarios"></a>部署方案

客户端和服务器之间的任何防火墙都必须为流量打开通信端口。

如果在反向代理配置中转发请求，请在调用 HTTPS 重定向中间件前使用 [转发的标头中间件](xref:host-and-deploy/proxy-load-balancer) 。 转发的标头中间件 `Request.Scheme` 使用 `X-Forwarded-Proto` 标头更新。 中间件允许重定向 Uri 和其他安全策略正常工作。 当未使用转发的标头中间件时，后端应用程序可能无法接收正确的方案并最终出现在重定向循环中。 常见的最终用户错误消息是发生了太多的重定向。

部署到 Azure App Service 时，请按照 [教程：将现有的自定义 SSL 证书绑定到 Azure Web 应用](/azure/app-service/app-service-web-tutorial-custom-ssl)中的指导进行操作。

### <a name="options"></a>选项

以下突出显示的代码调用 [AddHttpsRedirection](/dotnet/api/microsoft.aspnetcore.builder.httpsredirectionservicesextensions.addhttpsredirection) 来配置中间件选项：


::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](enforcing-ssl/sample-snapshot/3.x/Startup.cs?name=snippet2&highlight=14-18)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](enforcing-ssl/sample-snapshot/2.x/Startup.cs?name=snippet2&highlight=14-18)]

::: moniker-end


`AddHttpsRedirection`只有在更改或的值时，才需要调用 `HttpsPort` `RedirectStatusCode` 。

前面突出显示的代码：

* 将 [HttpsRedirectionOptions](xref:Microsoft.AspNetCore.HttpsPolicy.HttpsRedirectionOptions.RedirectStatusCode*) 设置为 <xref:Microsoft.AspNetCore.Http.StatusCodes.Status307TemporaryRedirect> ，这是默认值。 使用类的字段将 <xref:Microsoft.AspNetCore.Http.StatusCodes> 分配给 `RedirectStatusCode` 。
* 将 HTTPS 端口设置为5001。

#### <a name="configure-permanent-redirects-in-production"></a>在生产环境中配置永久重定向

中间件默认为通过所有重定向发送 [Status307TemporaryRedirect](/dotnet/api/microsoft.aspnetcore.http.statuscodes.status307temporaryredirect) 。 如果希望在应用处于非开发环境中时发送永久重定向状态代码，请在非开发环境的条件检查中包装中间件选项配置。

::: moniker range=">= aspnetcore-3.0"

在 *Startup.cs* 中配置服务时：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // IWebHostEnvironment (stored in _env) is injected into the Startup class.
    if (!_env.IsDevelopment())
    {
        services.AddHttpsRedirection(options =>
        {
            options.RedirectStatusCode = StatusCodes.Status308PermanentRedirect;
            options.HttpsPort = 443;
        });
    }
}
```

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

在 *Startup.cs* 中配置服务时：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // IHostingEnvironment (stored in _env) is injected into the Startup class.
    if (!_env.IsDevelopment())
    {
        services.AddHttpsRedirection(options =>
        {
            options.RedirectStatusCode = StatusCodes.Status308PermanentRedirect;
            options.HttpsPort = 443;
        });
    }
}
```

::: moniker-end


## <a name="https-redirection-middleware-alternative-approach"></a>HTTPS 重定向中间件备用方法

 () 使用 HTTPS 重定向中间件的替代方法 `UseHttpsRedirection` 是使用 URL 重写中间件 (`AddRedirectToHttps`) 。 `AddRedirectToHttps` 执行重定向时，还可以设置状态代码和端口。 有关详细信息，请参阅 [URL 重写中间件](xref:fundamentals/url-rewriting)。

在不需要其他重定向规则的情况下重定向到 HTTPS 时，我们建议使用 HTTPS 重定向中间件 (`UseHttpsRedirection` 本主题中所述) 。

<a name="hsts"></a>

## <a name="http-strict-transport-security-protocol-hsts"></a>HTTP 严格传输安全协议 (HSTS) 

根据 [OWASP](https://www.owasp.org/index.php/About_The_Open_Web_Application_Security_Project)， [HTTP 严格传输安全 (HSTS) ](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Strict_Transport_Security_Cheat_Sheet.html) 是通过使用响应标头由 web 应用指定的选择加入安全增强功能。 当 [支持 HSTS 的浏览器](https://cheatsheetseries.owasp.org/cheatsheets/Transport_Layer_Protection_Cheat_Sheet.html#browser-support) 收到此标头时：

* 浏览器存储域的配置，阻止通过 HTTP 发送任何通信。 浏览器强制通过 HTTPS 进行的所有通信。
* 浏览器阻止用户使用不受信任或无效的证书。 浏览器将禁用允许用户暂时信任此类证书的提示。

由于 HSTS 是由客户端强制执行的，因此存在一些限制：

* 客户端必须支持 HSTS。
* HSTS 需要至少一个成功的 HTTPS 请求才能建立 HSTS 策略。
* 应用程序必须检查每个 HTTP 请求并重定向或拒绝 HTTP 请求。

ASP.NET Core 2.1 和更高版本通过 `UseHsts` 扩展方法实现 HSTS。 `UseHsts`当应用未处于[开发模式](xref:fundamentals/environments)时，以下代码将调用：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](enforcing-ssl/sample-snapshot/3.x/Startup.cs?name=snippet1&highlight=11)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](enforcing-ssl/sample-snapshot/2.x/Startup.cs?name=snippet1&highlight=10)]

::: moniker-end

`UseHsts` 不建议在开发中使用，因为 HSTS 设置通过浏览器高度可缓存。 默认情况下，不 `UseHsts` 包括本地环回地址。

对于第一次实现 HTTPS 的生产环境，请使用其中一种方法将初始[HstsOptions](xref:Microsoft.AspNetCore.HttpsPolicy.HstsOptions.MaxAge*)设置为较小的值。 <xref:System.TimeSpan> 将值从小时设置为不超过一天，以防需要将 HTTPS 基础结构还原到 HTTP。 在你确信 HTTPS 配置的可持续性后，请增加 HSTS `max-age` 值; 一个常用值为一年。

下面的代码：


::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](enforcing-ssl/sample-snapshot/3.x/Startup.cs?name=snippet2&highlight=5-12)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](enforcing-ssl/sample-snapshot/2.x/Startup.cs?name=snippet2&highlight=5-12)]

::: moniker-end


* 设置标头的预载参数 `Strict-Transport-Security` 。 预加载不属于 [RFC HSTS 规范](https://tools.ietf.org/html/rfc6797)，但 web 浏览器支持在全新安装时预加载 HSTS 站点。 有关详细信息，请参阅 [https://hstspreload.org/](https://hstspreload.org/)。
* 启用 [includeSubDomain](https://tools.ietf.org/html/rfc6797#section-6.1.2)，这会将 HSTS 策略应用到托管子域。
* 将 `max-age` 标头的参数显式设置 `Strict-Transport-Security` 为60天。 如果未设置，则默认值为30天。 有关详细信息，请参阅 [最大期限指令](https://tools.ietf.org/html/rfc6797#section-6.1.1)。
* 添加 `example.com` 到要排除的主机列表。

`UseHsts` 排除以下环回主机：

* `localhost` ： IPv4 环回地址。
* `127.0.0.1` ： IPv4 环回地址。
* `[::1]` ： IPv6 环回地址。

## <a name="opt-out-of-httpshsts-on-project-creation"></a>在项目创建时选择退出 HTTPS/HSTS

在某些后端服务方案中，如果在网络面向公众的边缘处理连接安全，则不需要在每个节点上配置连接安全性。 从 Visual Studio 中的模板或从 [dotnet new](/dotnet/core/tools/dotnet-new) 命令生成的 Web 应用启用 [HTTPS 重定向](#require-https) 和 [HSTS](#http-strict-transport-security-protocol-hsts)。 对于不需要这些方案的部署，可以从模板创建应用时选择退出 HTTPS/HSTS。

选择退出 HTTPS/HSTS：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio) 

取消选中 " **为 HTTPS 配置** " 复选框。

::: moniker range=">= aspnetcore-3.0"

!["新建 ASP.NET Core Web 应用程序" 对话框，其中显示未选择 "配置为 HTTPS" 复选框。](enforcing-ssl/_static/out-vs2019.png)

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

!["新建 ASP.NET Core Web 应用程序" 对话框，其中显示未选择 "配置为 HTTPS" 复选框。](enforcing-ssl/_static/out.png)

::: moniker-end


# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli) 

使用 `--no-https` 选项。 例如：

```dotnetcli
dotnet new webapp --no-https
```

---

<a name="trust"></a>

## <a name="trust-the-aspnet-core-https-development-certificate-on-windows-and-macos"></a>信任 Windows 和 macOS 上的 ASP.NET Core HTTPS 开发证书

.NET Core SDK 包含 HTTPS 开发证书。 此证书作为首次运行体验的一部分进行安装。 例如，会 `dotnet --info` 生成以下输出的变体：

```
ASP.NET Core
------------
Successfully installed the ASP.NET Core HTTPS Development Certificate.
To trust the certificate run 'dotnet dev-certs https --trust' (Windows and macOS only).
For establishing trust on other platforms refer to the platform specific documentation.
For more information on configuring HTTPS see https://go.microsoft.com/fwlink/?linkid=848054.
```

安装 .NET Core SDK 会将 ASP.NET Core HTTPS 开发证书安装到本地用户证书存储。 已安装证书，但该证书不受信任。 若要信任该证书，请执行一次性步骤来运行 dotnet `dev-certs` 工具：

```dotnetcli
dotnet dev-certs https --trust
```

下面的命令提供有关 `dev-certs` 工具的帮助：

```dotnetcli
dotnet dev-certs https --help
```

## <a name="how-to-set-up-a-developer-certificate-for-docker"></a>如何为 Docker 设置开发人员证书

请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/6199)。

<a name="ssl-linux"></a>

## <a name="trust-https-certificate-on-linux"></a>在 Linux 上信任 HTTPS 证书

<!-- Instructions to be updated by engineering team after 5.0 RTM. -->

有关 Linux 的说明，请参阅分发文档。

<a name="wsl"></a>

## <a name="trust-https-certificate-from-windows-subsystem-for-linux"></a>从适用于 Linux 的 Windows 子系统信任 HTTPS 证书

适用于 Linux 的 Windows 子系统 (WSL) 生成一个 HTTPS 自签名证书。若要将 Windows 证书存储配置为信任 WSL 证书，请执行以下操作：

* 运行以下命令以导出 WSL 生成的证书：

  ```
  dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p <cryptic-password>
  ```
* 在 WSL 窗口中运行以下命令：

  ```
    ASPNETCORE_Kestrel__Certificates__Default__Password="<cryptic-password>" 
    ASPNETCORE_Kestrel__Certificates__Default__Path=/mnt/c/Users/user-name/.aspnet/https/aspnetapp.pfx
    dotnet watch run
  ```

  上述命令将设置环境变量，以便 Linux 使用 Windows 受信任的证书。

## <a name="troubleshoot-certificate-problems"></a>排查证书问题

本部分提供了在 [安装和信任](#trust)ASP.NET Core HTTPS 开发证书时，但仍会出现浏览器警告，指出该证书不受信任。 [Kestrel](xref:fundamentals/servers/kestrel)使用 ASP.NET Core HTTPS 开发证书。

### <a name="all-platforms---certificate-not-trusted"></a>所有平台-证书不受信任

运行以下命令：

```dotnetcli
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```

关闭所有打开的浏览器实例。 在应用程序中打开新的浏览器窗口。 证书信任由浏览器进行缓存。

前面的命令解决了大多数浏览器信任问题。 如果浏览器仍不信任证书，请遵循以下特定于平台的建议。

### <a name="docker---certificate-not-trusted"></a>Docker-证书不受信任

* 删除 *C:\Users \{ USER} \AppData\Roaming\ASP.NET\Https* 文件夹。
* 清理解决方案。 删除 bin 和 obj 文件夹。
* 重新启动开发工具。 例如，Visual Studio、Visual Studio Code 或 Visual Studio for Mac。

### <a name="windows---certificate-not-trusted"></a>Windows-证书不受信任

* 检查证书存储区中的证书。 `localhost`在和中应有一个具有 `ASP.NET Core HTTPS development certificate` 友好名称的证书 `Current User > Personal > Certificates``Current User > Trusted root certification authorities > Certificates`
* 从 "个人" 和 "受信任的根证书颁发机构" 中删除所有找到的证书。 请勿 **删除 IIS Express** localhost 证书。
* 运行以下命令：

```dotnetcli
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```

关闭所有打开的浏览器实例。 在应用程序中打开新的浏览器窗口。

### <a name="os-x---certificate-not-trusted"></a>OS X-证书不受信任

* 打开密钥链访问。
* 选择系统密钥链。
* 检查是否存在 localhost 证书。
* 检查它是否 `+` 在图标上包含符号，以指示其对所有用户都是受信任的。
* 从系统密钥链中删除证书。
* 运行以下命令：

```dotnetcli
dotnet dev-certs https --clean
dotnet dev-certs https --trust
```

关闭所有打开的浏览器实例。 在应用程序中打开新的浏览器窗口。

请参阅 [HTTPS 错误，使用 IIS Express (dotnet/AspNetCore #16892) ](https://github.com/dotnet/AspNetCore/issues/16892) 用于排查 Visual Studio 的证书问题。

### <a name="iis-express-ssl-certificate-used-with-visual-studio"></a>用于 Visual Studio 的 IIS Express SSL 证书

若要解决 IIS Express 证书的问题，请从 Visual Studio 安装程序中选择 " **修复** "。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/aspnetcore/issues/16892)。

## <a name="additional-information"></a>其他信息

* <xref:host-and-deploy/proxy-load-balancer>
* [在 Linux 上利用 Apache： HTTPS 配置托管 ASP.NET Core](xref:host-and-deploy/linux-apache#https-configuration)
* [Nginx 上的主机 ASP.NET Core： HTTPS 配置](xref:host-and-deploy/linux-nginx#https-configuration)
* [如何在 IIS 上设置 SSL](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis)
* [OWASP HSTS 浏览器支持](https://www.owasp.org/index.php/HTTP_Strict_Transport_Security_Cheat_Sheet#Browser_Support)