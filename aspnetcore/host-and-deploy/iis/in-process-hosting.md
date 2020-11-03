---
title: 使用 IIS 和 ASP.NET Core 进行进程内托管
author: rick-anderson
description: 了解如何使用 IIS 和 ASP.NET Core 模块进行进程内托管。
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
uid: host-and-deploy/iis/in-process-hosting
ms.openlocfilehash: ecf873e6ad2044aae87a5e7dc07316eae0e10912
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91755142"
---
# <a name="in-process-hosting-with-iis-and-aspnet-core"></a>使用 IIS 和 ASP.NET Core 进行进程内托管 

进程内托管在与其 IIS 工作进程相同的进程中运行 ASP.NET Core 应用。 进程内承载相较进程外承载提供更优的性能，因为请求并不通过环回适配器进行代理，环回适配器是一个网络接口，用于将传出的网络流量返回给同一计算机。

下图说明了 IIS、ASP.NET Core 模块和进程内托管的应用之间的关系：

![进程内托管方案中的 ASP.NET Core 模块](index/_static/ancm-inprocess.png)

## <a name="enable-in-process-hosting"></a>启用进程内托管

自 ASP.NET Core 3.0 起，默认情况下已为部署到 IIS 的所有应用启用进程内托管。

若要显式配置进程内托管的应用，请在项目文件 (`.csproj`) 中将 `<AspNetCoreHostingModel>` 属性的值设置为 `InProcess`：

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

## <a name="general-architecture"></a>一般体系结构

请求的常规流程如下：

1. 请求从 Web 到达内核模式 HTTP.sys 驱动程序。
1. 驱动程序将本机请求路由到网站的配置端口上的 IIS，通常为 80 (HTTP) 或 443 (HTTPS)。
1. ASP.NET Core 模块接收本机请求，并将其传递给 IIS HTTP 服务器 (`IISHttpServer`)。 IIS HTTP 服务器是将请求从本机转换为托管的 IIS 进程内服务器实现。

在 IIS HTTP 服务器处理请求后：

1. 请求被发送到 ASP.NET Core 中间件管道。
1. 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。
1. 应用的响应通过 IIS HTTP 服务器传递回 IIS。
1. IIS 将响应发送到发起请求的客户端。

`CreateDefaultBuilder` 添加 <xref:Microsoft.AspNetCore.Hosting.Server.IServer> 实例的方式是：调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS%2A> 方法来启动 [CoreCLR](/dotnet/standard/glossary#coreclr) 和将应用托管在 IIS 工作进程（`w3wp.exe` 或 `iisexpress.exe`）内。 性能测试表明，与在进程外托管应用并将请求代理传入 [Kestrel](xref:fundamentals/servers/kestrel) 相比，在进程中托管 .NET Core 应用可提供明显更高的请求吞吐量。

作为单个文件可执行文件发布的应用无法由进程内托管模型加载。

## <a name="application-configuration"></a>应用程序配置

要配置 IIS 选项，请在 <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A> 中包括 <xref:Microsoft.AspNetCore.Builder.IISServerOptions> 的服务配置。 下面的示例禁用 AutomaticAuthentication：

```csharp
services.Configure<IISServerOptions>(options => 
{
    options.AutomaticAuthentication = false;
});
```

| 选项 | 默认 | 设置 |
| ------ | :-----: | ------- |
| `AutomaticAuthentication` | `true` | 若为 `true`，IIS 服务器将设置经过 [Windows 身份验证](xref:security/authentication/windowsauth)进行身份验证的 `HttpContext.User`。 若为 `false`，服务器仅提供 `HttpContext.User` 的标识并在 `AuthenticationScheme` 显式请求时响应质询。 必须在 IIS 中启用 Windows 身份验证使 `AutomaticAuthentication` 得以运行。 有关详细信息，请参阅 [Windows 身份验证](xref:security/authentication/windowsauth)。 |
| `AuthenticationDisplayName` | `null` | 设置在登录页上向用户显示的显示名。 |
| `AllowSynchronousIO` | `false` | `HttpContext.Request` 和 `HttpContext.Response` 是否允许同步 I/O。 |
| `MaxRequestBodySize` | `30000000` | 获取或设置 `HttpRequest` 的最大请求正文大小。 请注意，IIS 本身有限制 `maxAllowedContentLength`，这一限制将在 `IISServerOptions` 中设置 `MaxRequestBodySize` 之前进行处理。 更改 `MaxRequestBodySize` 不会影响 `maxAllowedContentLength`。 若要增加 `maxAllowedContentLength`，请在 `web.config` 中添加一个将 `maxAllowedContentLength` 设置为更高值的项。 有关更多详细信息，请参阅[配置](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/#configuration)。 |

## <a name="differences-between-in-process-and-out-of-process-hosting"></a>进程内和进程外托管之间的差异

在进程内托管时，将应用以下特征：

* 使用 IIS HTTP 服务器 (`IISHttpServer`) 而不是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器。 对于进程内托管，[`CreateDefaultBuilder`](xref:fundamentals/host/generic-host#default-builder-settings) 会调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS%2A> 来进行以下操作：

  * 注册 `IISHttpServer`。
  * 在 ASP.NET Core 模块后运行时，配置服务器应侦听的端口和基本路径。
  * 配置主机以捕获启动错误。

* [`requestTimeout` 属性](xref:host-and-deploy/iis/web-config#attributes-of-the-aspnetcore-element)不适用于进程内托管。

* 不支持在应用之间共享应用池。 每个应用使用一个应用池。

* 应用和已安装的运行时（x64 或 x86）的体系结构（位数）必须与应用池的体系结构匹配。 例如，为 32 位 (x86) 发布的应用必须已为其 IIS 应用程序池启用 32 位。 有关详细信息，请参阅[创建 IIS 站点](xref:tutorials/publish-to-iis#create-the-iis-site)部分。

* 检测到客户端连接断开。 客户端断开连接时，将取消 [`HttpContext.RequestAborted`](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted%2A) 取消标记。

* 在进程内托管时，不会在内部调用 <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync%2A> 来初始化用户。 因此，默认情况下不激活每次身份验证后用于转换声明的 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 实现。 使用 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 实现转换声明时，请调用 <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication%2A> 以添加身份验证服务：

  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddTransient<IClaimsTransformation, ClaimsTransformer>();
      services.AddAuthentication(IISServerDefaults.AuthenticationScheme);
  }

  public void Configure(IApplicationBuilder app)
  {
      app.UseAuthentication();
  }
  ```
  
* 不支持 [Web 包（单文件）部署](/aspnet/web-forms/overview/deployment/web-deployment-in-the-enterprise/deploying-web-packages)。
