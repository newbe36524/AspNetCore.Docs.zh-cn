---
title: 使用 IIS 和 ASP.NET Core 进行进程外托管
author: rick-anderson
description: 了解如何使用 IIS 和 ASP.NET Core 模块进行进程外托管。
monikerRange: '>= aspnetcore-5.0'
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
uid: host-and-deploy/iis/out-of-process-hosting
ms.openlocfilehash: 048a18349de4d784ae4abb33bd86a2d9c08c609d
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91755149"
---
# <a name="out-of-process-hosting-with-iis-and-aspnet-core"></a>使用 IIS 和 ASP.NET Core 进行进程外托管 

由于运行 ASP.NET Core 的进程与 IIS 工作进程分开，因此 ASP.NET Core 模块会负责进程管理。 该模块在第一个请求到达时启动 ASP.NET Core 应用的进程，并在应用关闭或崩溃时重新启动该应用。 这基本上与在 [Windows 进程激活服务 (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) 托管的进程内运行的应用中出现的行为相同。

下图说明了 IIS、ASP.NET Core 模块和进程外托管的应用之间的关系：

![进程外托管方案中的 ASP.NET Core 模块](index/_static/ancm-outofprocess.png)

1. 请求从 Web 到达内核模式 HTTP.sys 驱动程序。
1. 驱动程序将请求路由到网站的配置端口上的 IIS。 配置的端口通常是 80 (HTTP) 或 443 (HTTPS)。
1. 此模块将该请求转发到应用的随机端口上的 Kestrel。 随机端口不是 80 或 443。

<!-- make this a bullet list -->
ASP.NET Core 模块在启动时通过环境变量指定端口。 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration%2A> 扩展将服务器配置为侦听 `http://localhost:{PORT}`。 执行其他检查，拒绝不是来自该模块的请求。 此模块不支持 HTTPS 转发。 即使请求由 IIS 通过 HTTPS 接收，它们还是通过 HTTP 转发。

Kestrel 从模块获取请求后，请求会被转发到 ASP.NET Core 中间件管道中。 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。 IIS 集成添加的中间件会将方案、远程 IP 和 pathbase 更新到帐户以将请求转发到 Kestrel。 应用的响应传递回 IIS，IIS 将响应转发回发起请求的 HTTP 客户端。

有关 ASP.NET Core 模块配置指南，请参阅 <xref:host-and-deploy/aspnet-core-module>。

有关托管的详细信息，请参阅[在 ASP.NET Core 中托管](xref:fundamentals/index#host)。

## <a name="application-configuration"></a>应用程序配置

### <a name="enable-the-iisintegration-components"></a>启用 IISIntegration 组件

在 `CreateHostBuilder` 中生成主机 (`Program.cs`)，请调用 <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A> 以启用 IIS 集成：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        ...
```

有关 `CreateDefaultBuilder` 的详细信息，请参阅 <xref:fundamentals/host/generic-host#default-builder-settings>。


**进程外承载模型**

要配置 IIS 选项，请在 <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A> 中包括 <xref:Microsoft.AspNetCore.Builder.IISOptions> 的服务配置。 下面的示例阻止应用填充 `HttpContext.Connection.ClientCertificate`：

```csharp
services.Configure<IISOptions>(options => 
{
    options.ForwardClientCertificate = false;
});
```

| 选项                         | 默认 | 设置 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | 若为 `true`，[IIS 集成中间件](#enable-the-iisintegration-components)将设置经过 [Windows 身份验证](xref:security/authentication/windowsauth)进行身份验证的 `HttpContext.User`。 若为 `false`，中间件仅提供 `HttpContext.User` 的标识并在 `AuthenticationScheme` 显式请求时响应质询。 必须在 IIS 中启用 Windows 身份验证使 `AutomaticAuthentication` 得以运行。 有关详细信息，请参阅 [Windows 身份验证](xref:security/authentication/windowsauth)主题。 |
| `AuthenticationDisplayName`    | `null`  | 设置在登录页上向用户显示的显示名。 |
| `ForwardClientCertificate`     | `true`  | 若为 `true`，且存在 `MS-ASPNETCORE-CLIENTCERT` 请求头，则填充 `HttpContext.Connection.ClientCertificate`。 |


### <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

将 [IIS 集成中间件](#enable-the-iisintegration-components)和 ASP.NET Core 模块配置为转发：

* 方案 (HTTP/HTTPS)。
* 发起请求的远程 IP 地址。

[IIS 集成中间件](#enable-the-iisintegration-components)配置转发的标头中间件。

对于托管在其他代理服务器和负载均衡器后方的应用，可能需要附加配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。


### <a name="out-of-process-hosting-model"></a>进程外托管模型

若要配置进程外托管应用，请在项目文件 ( `.csproj`) 中将 `<AspNetCoreHostingModel>` 属性的值设置为 `OutOfProcess`：

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

进程内托管设为 `InProcess`，这是默认值。

`<AspNetCoreHostingModel>` 的值不区分大小写，因此 `inprocess` 和 `outofprocess` 均为有效值。

使用 [Kestrel](xref:fundamentals/servers/kestrel) 服务器，而不是 IIS HTTP 服务器 (`IISHttpServer`)。

对于进程外托管，[`CreateDefaultBuilder`](xref:fundamentals/host/generic-host#default-builder-settings) 会调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration%2A> 来进行以下操作：

* 在 ASP.NET Core 模块后运行时，配置服务器应侦听的端口和基本路径。
* 配置主机以捕获启动错误。

### <a name="process-name"></a>进程名

`Process.GetCurrentProcess().ProcessName` 报告 `w3wp`/`iisexpress`（进程内）或 `dotnet`（进程外）。

许多本机模块（如 Windows 身份验证）仍处于活动状态。 要详细了解随 ASP.NET Core 模块活动的 IIS 模块，请参阅 <xref:host-and-deploy/iis/modules>。

ASP.NET Core 模块还可以：

* 为工作进程设置环境变量。
* 将 stdout 输出记录到文件存储器，以解决启动问题。
* 转发 Windows 身份验证令牌。
