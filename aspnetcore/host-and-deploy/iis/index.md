---
title: 使用 IIS 在 Windows 上托管 ASP.NET Core
author: rick-anderson
description: 了解如何在 Windows Server Internet Information Services (IIS) 上托管 ASP.NET Core 应用。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 5/7/2020
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
uid: host-and-deploy/iis/index
ms.openlocfilehash: e4a94ca9e3607868f3eb25d88338e8156f7f5206
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061517"
---
# <a name="host-aspnet-core-on-windows-with-iis"></a>使用 IIS 在 Windows 上托管 ASP.NET Core

::: moniker range=">= aspnetcore-5.0"

Internet Information Services (IIS) 是一种灵活、安全且可管理的 Web 服务器，用于托管 Web 应用（包括 ASP.NET Core）。

## <a name="supported-platforms"></a>受支持的平台

支持下列操作系统：

* Windows 7 或更高版本
* Windows Server 2012 R2 或更高版本

支持针对 32 位 (x86) 或 64 位 (x64) 部署发布的应用。 使用 32 位 (x86) .NET Core SDK 部署 32 位应用，除非应用符合以下情况：

* 需要适用于 64 位应用的更大虚拟内存地址空间。
* 需要更大 IIS 堆栈大小。
* 具有 64 位本机依赖项。

## <a name="install-the-aspnet-core-modulehosting-bundle"></a>安装 ASP.NET Core 模块/托管捆绑包

使用以下链接下载安装程序：

[当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

有关如何安装 ASP.NET Core Module 或不同版本的详细说明，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/hosting-bundle)。

## <a name="get-started"></a>入门

有关在 IIS 上托管网站的入门知识，请参阅[入门指南](xref:tutorials/publish-to-iis)。

有关在 Azure 应用服务上托管网站的入门知识，请参阅[“部署到 Azure 应用服务”指南](xref:host-and-deploy/azure-apps/index)。

## <a name="deployment-resources-for-iis-administrators"></a>面向 IIS 管理员的部署资源

* [IIS 文档](/iis)
* [IIS 中 IIS 管理器入门](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8)
* [.NET Core 应用程序部署](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:index>
* [Microsoft IIS 官方网站](https://www.iis.net/)
* [Windows Server 技术内容库](/windows-server/windows-server)
* [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

若要学习将 ASP.NET Core 应用发布到 IIS 服务器的教程，请参阅<xref:tutorials/publish-to-iis>。

[安装 .NET Core 托管捆绑包](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>支持的操作系统

支持下列操作系统：

* Windows 7 或更高版本
* Windows Server 2012 R2 或更高版本

[HTTP.sys 服务器](xref:fundamentals/servers/httpsys)（以前称为 WebListener）无法以反向代理配置形式与 IIS 结合使用。 请使用 [Kestrel 服务器](xref:fundamentals/servers/kestrel)。

有关在 Azure 上托管的信息，请参阅<xref:host-and-deploy/azure-apps/index>。

有关疑难解答指南，请参阅 <xref:test/troubleshoot>。

## <a name="supported-platforms"></a>受支持的平台

支持针对 32 位 (x86) 或 64 位 (x64) 部署发布的应用。 使用 32 位 (x86) .NET Core SDK 部署 32 位应用，除非应用符合以下情况：

* 需要适用于 64 位应用的更大虚拟内存地址空间。
* 需要更大 IIS 堆栈大小。
* 具有 64 位本机依赖项。

为 32 位 (x86) 发布的应用必须已为其 IIS 应用程序池启用 32 位。 有关详细信息，请参阅[创建 IIS 站点](#create-the-iis-site)部分。

使用 64 位 (x64) .NET Core SDK 发布 64 位应用。 主机系统必须具有 64 位运行时。

## <a name="hosting-models"></a>托管模型

### <a name="in-process-hosting-model"></a>进程内托管模型

使用进程内托管，ASP.NET Core 在与其 IIS 工作进程相同的进程中运行。 进程内承载相较进程外承载提供更优的性能，因为请求并不通过环回适配器进行代理，环回适配器是一个网络接口，用于将传出的网络流量返回给同一计算机。 IIS 使用 [Windows 进程激活服务 (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) 处理进程管理。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)：

* 执行应用初始化。
  * 加载 [CoreCLR](/dotnet/standard/glossary#coreclr)。
  * 调用 `Program.Main`。
* 处理 IIS 本机请求的生存期。

下图说明了 IIS、ASP.NET Core 模块和进程内托管的应用之间的关系：

![进程内托管方案中的 ASP.NET Core 模块](index/_static/ancm-inprocess.png)

1. 请求从 Web 到达内核模式 HTTP.sys 驱动程序。
1. 驱动程序将本机请求路由到网站的配置端口上的 IIS，通常为 80 (HTTP) 或 443 (HTTPS)。
1. ASP.NET Core 模块接收本机请求，并将其传递给 IIS HTTP 服务器 (`IISHttpServer`)。 IIS HTTP 服务器是将请求从本机转换为托管的 IIS 进程内服务器实现。

在 IIS HTTP 服务器处理请求后：

1. 请求被发送到 ASP.NET Core 中间件管道。
1. 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。
1. 应用的响应通过 IIS HTTP 服务器传递回 IIS。
1. IIS 将响应发送到发起请求的客户端。

对于现有应用，进程内托管是可选功能。 ASP.NET Core Web 模板使用进程内托管模型。

`CreateDefaultBuilder` 添加 <xref:Microsoft.AspNetCore.Hosting.Server.IServer> 实例的方式是：调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS%2A> 方法来启动 [CoreCLR](/dotnet/standard/glossary#coreclr) 和将应用托管在 IIS 工作进程（`w3wp.exe` 或 `iisexpress.exe`）内。 性能测试表明，与在进程外托管应用并将请求代理传入 [Kestrel](xref:fundamentals/servers/kestrel) 相比，在进程中托管 .NET Core 应用可提供明显更高的请求吞吐量。

作为单个文件可执行文件发布的应用无法由进程内托管模型加载。

### <a name="out-of-process-hosting-model"></a>进程外托管模型

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

### <a name="iis-options"></a>IIS 选项

**进程内承载模型**

要配置 IIS 服务器选项，请在 <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A> 中包括 <xref:Microsoft.AspNetCore.Builder.IISServerOptions> 的服务配置。 下面的示例禁用 AutomaticAuthentication：

```csharp
services.Configure<IISServerOptions>(options => 
{
    options.AutomaticAuthentication = false;
});
```

| 选项                         | 默认 | 设置 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | 若为 `true`，IIS 服务器将设置经过 [Windows 身份验证](xref:security/authentication/windowsauth)进行身份验证的 `HttpContext.User`。 若为 `false`，服务器仅提供 `HttpContext.User` 的标识并在 `AuthenticationScheme` 显式请求时响应质询。 必须在 IIS 中启用 Windows 身份验证使 `AutomaticAuthentication` 得以运行。 有关详细信息，请参阅 [Windows 身份验证](xref:security/authentication/windowsauth)。 |
| `AuthenticationDisplayName`    | `null`  | 设置在登录页上向用户显示的显示名。 |
| `AllowSynchronousIO`           | `false` | `HttpContext.Request` 和 `HttpContext.Response` 是否允许同步 I/O。 |
| `MaxRequestBodySize`           | `30000000`  | 获取或设置 `HttpRequest` 的最大请求正文大小。 请注意，IIS 本身有限制 `maxAllowedContentLength`，这一限制将在 `IISServerOptions` 中设置 `MaxRequestBodySize` 之前进行处理。 更改 `MaxRequestBodySize` 不会影响 `maxAllowedContentLength`。 若要增加 `maxAllowedContentLength`，请在 `web.config` 中添加一个将 `maxAllowedContentLength` 设置为更高值的项。 有关更多详细信息，请参阅[配置](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/#configuration)。 |

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

### <a name="webconfig-file"></a>`web.config` 文件

`web.config` 文件配置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 发布项目时，`web.config` 文件的创建、转换和发布是由 MSBuild 目标 (`_TransformWebConfig`) 处理的。 此目标位于 Web SDK 目标 (`Microsoft.NET.Sdk.Web`) 中。 SDK 设置在项目文件的顶部：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

如果项目中没有 `web.config` 文件，则该文件是使用正确的 `processPath` 和 `arguments`（用于配置 ASP.NET Core 模块）创建的，并且已被移到[发布的输出](xref:host-and-deploy/directory-structure)中。

如果项目中没有 `web.config` 文件，则该文件是通过正确的 `processPath` 和 `arguments`（用于配置 ASP.NET Core 模块）转换的，并且已被移到发布的输出中。 转换不会修改文件中的 IIS 配置设置。

`web.config` 文件可能会提供控制活动 IIS 模块的额外 IIS 配置设置。 有关能够处理 ASP.NET Core 应用请求的 IIS 模块的信息，请参阅 [IIS 模块](xref:host-and-deploy/iis/modules)主题。

为了防止 Web SDK 转换 `web.config` 文件，请在项目文件中使用 `<IsTransformWebConfigDisabled>` 属性：

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

禁用 Web SDK 对文件的转换时，`processPath` 和 `arguments` 应由开发人员手动设置。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

### <a name="webconfig-file-location"></a>`web.config` 文件位置

为了正确设置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)，`web.config` 文件必须存在于已部署应用的[内容根](xref:fundamentals/index#content-root)路径（通常为应用基路径）中。 该位置与向 IIS 提供的网站物理路径相同。 若要使用 Web 部署发布多个应用，应用的根路径中需要包含 `web.config` 文件。

敏感文件存在于应用的物理路径中，如 `{ASSEMBLY}.runtimeconfig.json`、`{ASSEMBLY}.xml`（XML 文档注释）和 `{ASSEMBLY}.deps.json`，其中 `{ASSEMBLY}` 占位符为程序集名称。 如果有 `web.config` 文件且站点正常启动时，IIS 在收到敏感文件请求时不会提供这些敏感文件。 如果 `web.config` 文件缺失、名字错误或者无法将站点配置为正常启动，IIS 可能会公开提供敏感文件。

`web.config` 文件必须始终存在于部署中、名称正确以及能够将站点配置为正常启动。切勿从生产部署中删除 `web.config` 文件。

### <a name="transform-webconfig"></a>转换 web.config

如果在发布时需要转换 `web.config`，请参阅 <xref:host-and-deploy/iis/transform-webconfig>。 你需要在发布时转换 `web.config`，以根据配置、配置文件或环境来设置环境变量。

## <a name="iis-configuration"></a>IIS 配置

**Windows Server 操作系统**

启用 Web 服务器 (IIS) 服务器角色并建立角色服务。

1. 通过“管理”菜单或“服务器管理器”中的链接使用“添加角色和功能”向导。 在“服务器角色”步骤中，选中“Web 服务器(IIS)”框 。

   ![在选择服务器角色步骤中选择了“Web 服务器 IIS”角色。](index/_static/server-roles-ws2016.png)

1. 在“功能”步骤后，为 Web 服务器 (IIS) 加载“角色服务”步骤。 选择所需 IIS 角色服务，或接受提供的默认角色服务。

   ![在选择角色服务步骤中选择了默认角色服务。](index/_static/role-services-ws2016.png)

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“Web 服务器” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 `<windowsAuthentication>`](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“Web 服务器” > “应用开发”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 继续执行“确认”步骤，安装 Web 服务器角色和服务。 安装 Web 服务器 (IIS) 角色后无需重启服务器/IIS。

**Windows 桌面操作系统**

启用“IIS 管理控制台”和“万维网服务”。

1. 导航到“控制面板”>“程序”>“程序和功能”>“打开或关闭 Windows 功能”（位于屏幕左侧）   。

1. 打开“Internet Information Services”节点。 打开“Web 管理工具”节点。

1. 选中“IIS 管理控制台”框。

1. 选中“万维网服务”框。

1. 接受“万维网服务”的默认功能，或自定义 IIS 功能。

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“万维网服务” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“万维网服务” > “应用开发功能”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 如果 IIS 安装需要重新启动，则重新启动系统。

![在“Windows 功能”中选择了“IIS 管理控制台”和“万维网服务”。](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>安装 .NET Core 托管捆绑包

在托管系统上安装 .NET Core 托管捆绑包。 捆绑包可安装 .NET Core 运行时、.NET Core 库和 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 该模块允许 ASP.NET Core 应用在 IIS 后面运行。

> [!IMPORTANT]
> 如果在 IIS 之前安装了托管捆绑包，则必须修复捆绑包安装。 在安装 IIS 后再次运行托管捆绑包安装程序。
>
> 如果在安装 64 位 (x64) 版本的 .NET Core 之后安装了 Hosting Bundle，则可能看上去缺少 SDK（[未检测到 .NET Core SDK](xref:test/troubleshoot#no-net-core-sdks-were-detected)）。 要解决此问题，请参阅 <xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle>。

### <a name="direct-download-current-version"></a>直接下载（当前版本）

使用以下链接下载安装程序：

[当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

### <a name="earlier-versions-of-the-installer"></a>先前版本的安装程序

若要获取先前版本的安装程序：

1. 导航到 [ .NET Core](https://dotnet.microsoft.com/download/dotnet-core) 页面。
1. 选择所需的 .NET Core 版本。
1. 在“运行应用 - 运行时”列中，查找所需的 .NET Core 运行时版本的那一行。
1. 使用“托管捆绑包”链接下载安装程序。

> [!WARNING]
> 某些安装程序包含已到达其生命周期结束 (EOL) 且不再受 Microsoft 支持的发行版本。 有关详细信息，请参阅[支持策略](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)。

### <a name="install-the-hosting-bundle"></a>安装托管捆绑包

1. 在服务器上运行安装程序。 通过管理员命令行界面运行安装程序时，以下参数可用：

   * `OPT_NO_ANCM=1`：跳过安装 ASP.NET Core 模块。
   * `OPT_NO_RUNTIME=1`：跳过安装 .NET Core 运行时。 当服务器仅承载[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) 时使用。
   * `OPT_NO_SHAREDFX=1`：跳过安装 ASP.NET 共享框架（ASP.NET 运行时）。 当服务器仅承载[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) 时使用。
   * `OPT_NO_X86=1`：跳过安装 x86 运行时。 确定不会托管 32 位应用时，请使用此参数。 如果有可能会同时托管 32 位和 64 位应用，请勿使用此参数，并安装两个运行时。
   * `OPT_NO_SHARED_CONFIG_CHECK=1`：当共享配置 (`applicationHost.config`) 与 IIS 安装位于同一台计算机上时，禁止对使用的是否是 IIS 共享配置进行检查。 仅适用于 ASP.NET Core 2.2 或更高版本托管捆绑程序安装程序。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>。
1. 重新启动系统，或在命令行界面中执行以下命令：

   ```console
   net stop was /y
   net start w3svc
   ```
   重启 IIS 会选取安装程序对系统 PATH（环境变量）所作的更改。

ASP.NET Core 不采用共享框架包的修补程序版本的前滚行为。 通过安装新的托管捆绑包升级共享框架后，请重新启动系统，或在命令行界面中执行以下命令：

```console
net stop was /y
net start w3svc
```

> [!NOTE]
> 有关 IIS 共享配置的信息，请参阅[使用 IIS 共享配置的 ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration)。

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>使用 Visual Studio 进行发布时安装 Web 部署

使用 [Web 部署](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)将应用部署到服务器时，请在服务器上安装最新版本的 Web 部署。 要安装 Web 部署，请使用 [Web 平台安装程序 (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) 或直接从 [Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=43717)获取安装程序。 建议使用 WebPI。 WebPI 为托管提供程序提供独立的安装程序和配置。

## <a name="create-the-iis-site"></a>创建 IIS 站点

1. 在托管系统上，创建一个文件夹以包含应用已发布的文件夹和文件。 在接下来的步骤中，文件夹路径作为应用程序的物理路径提供给 IIS。 有关应用程序部署文件夹和文件布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。

1. 在 IIS 管理器中，打开“连接”面板中的服务器节点。 右键单击“站点”文件夹。 选择上下文菜单中的“添加网站”。

1. 提供网站名称，并将物理路径设置为应用的部署文件夹 。 提供“绑定”配置，并通过选择“确定”创建网站：

   ![在“添加网站”步骤中提供网站名称、物理路径和主机名。](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定可能会为应用带来安全漏洞。 此行为同时适用于强通配符和弱通配符。 使用显式主机名而不是通配符。 如果可控制整个父域（区别于易受攻击的 `*.com`），则子域通配符绑定（例如，`*.mysub.com`）不具有此安全风险。 有关详细信息，请参阅 [rfc7230 第 5.4 条](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器节点下，选择“应用程序池”。

1. 右键单击站点的应用池，然后从上下文菜单中选择“基本设置”。

1. 在“编辑应用程序池”窗口中，将“.NET CLR 版本”设置为“无托管代码”：

   ![将“.NET CLR 版本”设置为“无托管代码”。](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core 在单独的进程中运行，并管理运行时。 ASP.NET Core 不依赖桌面 CLR (.NET CLR) 加载。 将启动 .NET Core 的 Core 公共语言运行时 (CoreCLR)，在工作进程中托管应用。 将“.NET CLR 版本”设置为“无托管代码”是可选步骤，但建议采用此设置。

1. *ASP.NET Core 2.2 或更高版本* ：

   * 对于使用 32 位 SDK 发布的 32 位 (x86) [独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)，且该 SDK 使用[进程内托管模型](#in-process-hosting-model)，请为 32 位启用应用程序池。 在 IIS 管理器中，导航到“连接”边栏中的“应用程序池” 。 选择应用的应用程序池。 在“操作”边栏中，选择，“高级设置” 。 将“启用 32 位应用程序”设置为 `True`。 

   * 对于使用[进程内托管模型](#in-process-hosting-model)的 64 位 (x64) [独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)，为 32 位 (x86) 进程禁用应用池。 在 IIS 管理器中，导航到“连接”边栏中的“应用程序池” 。 选择应用的应用程序池。 在“操作”边栏中，选择，“高级设置” 。 将“启用 32 位应用程序”设置为 `False`。 

1. 确认进程模型标识拥有适当的权限。

   如果将应用池的默认标识（“进程模型” > “Identity”）从 ApplicationPoolIdentity 更改为另一标识，请确保新标识拥有对应用文件夹、数据库和其他所需资源的必需访问权限  。 例如，应用池需要对文件夹的读取和写入权限，以便应用在其中读取和写入文件。

**Windows 身份验证配置（可选）**  
有关详细信息，请参阅[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

## <a name="deploy-the-app"></a>部署应用

将应用程序部署到 IIS 物理路径文件夹中，该文件夹是在[创建 IIS 站点](#create-the-iis-site)部分中创建的。 建议使用[Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)这种部署机制，不过将应用从项目的 `publish` 文件夹移到托管系统的部署文件夹还有几种选择。

### <a name="web-deploy-with-visual-studio"></a>在 Visual Studio 内使用 Web 部署

要了解如何创建用于 Web 部署的发布配置文件，请参阅[用于 ASP.NET Core 应用部署的 Visual Studio 发布配置文件](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)。 如果托管提供程序提供了发布配置文件或支持创建发布配置文件，请下载配置文件并使用 Visual Studio 的“发布”对话框将其导入：

![“发布”对话框页](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>在 Visual Studio 之外使用 Web 部署

也可以在 Visual Studio 之外从命令行使用 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)。 有关详细信息，请参阅 [Web Deployment Tool](/iis/publish/using-web-deploy/use-the-web-deployment-tool)（Web 部署工具）。

### <a name="alternatives-to-web-deploy"></a>Web 部署的替代方法

有多种方法可将应用移动到托管系统，例如手动复制、[Xcopy](/windows-server/administration/windows-commands/xcopy)、[Robocopy](/windows-server/administration/windows-commands/robocopy) 或 [PowerShell](/powershell/)，可使用其中任何一种方法。

有关将 ASP.NET Core 部署到 IIS 的详细信息，请参阅[面向 IIS 管理员的部署资源](#deployment-resources-for-iis-administrators)部分。

## <a name="browse-the-website"></a>浏览网站

将应用部署到托管系统后，向应用的一个公共终结点发出请求。

在以下示例中，站点被绑定到端口 `80` 上 `www.mysite.com` 的 IIS 主机名中。 向 `http://www.mysite.com` 发出请求：

![Microsoft Edge 浏览器已加载 IIS 启动页。](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>锁定的部署文件

如果应用正在运行，部署文件夹中的文件会被锁定。 在部署期间，无法覆盖已锁定的文件。 若要在部署中解除已锁定的文件，请使用以下方法之一停止应用池：

* 使用 Web 部署并在项目文件中引用 `Microsoft.NET.Sdk.Web`。 `app_offline.htm` 文件位于 Web 应用目录的根路径下。 如果该文件存在，ASP.NET Core 模块在部署期间正常关闭应用并提供 `app_offline.htm` 文件。 有关详细信息，请参阅 [ASP.NET Core 模块配置参考](xref:host-and-deploy/aspnet-core-module#app_offlinehtm)。
* 在服务器上的 IIS 管理器中手动停止应用池。
* 使用 PowerShell 删除 `app_offline.htm`（要求 PowerShell 5 或更高版本）：

  ```powershell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm
  ```

## <a name="data-protection"></a>数据保护

[ASP.NET Core 数据保护堆栈](xref:security/data-protection/introduction)由多个 ASP.NET Core [中间件](xref:fundamentals/middleware/index)使用，包括用于身份验证的中间件。 即使用户代码不调用数据保护 API，也应该使用部署脚本或在用户代码中配置数据保护，以创建持久的加密[密钥存储](xref:security/data-protection/implementation/key-management)。 如果不配置数据保护，则密钥存储在内存中。重启应用时，密钥会被丢弃。

如果密钥环存储于内存中，则在应用重启时：

* 所有基于 cookie 的身份验证令牌都无效。 
* 用户需要在下一次请求时再次登录。 
* 无法再解密使用密钥环保护的任何数据。 这可能包括 [CSRF 令牌](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)和 [ASP.NET Core MVC TempData cookie](xref:fundamentals/app-state#tempdata)。

若要在 IIS 下配置数据保护以持久化密钥环，请使用以下方法之一：

* **创建数据保护注册表项**

  ASP.NET Core 应用使用的数据保护密钥存储在应用外部的注册表中。 要持久保存给定应用的密钥，需为应用池创建注册表项。

  对于独立的非 Web 场 IIS 安装，可以对用于 ASP.NET Core 应用的每个应用池使用[数据保护 Provision-AutoGenKeys.ps1 PowerShell 脚本](https://github.com/dotnet/AspNetCore/blob/master/src/DataProtection/Provision-AutoGenKeys.ps1)。 此脚本在 HKLM 注册表中创建注册表项，仅应用程序的应用池工作进程帐户可对其进行访问。 通过计算机范围的密钥使用 DPAPI 对密钥静态加密。

  在 web 场方案中，可以将应用配置为使用 UNC 路径存储其数据保护密钥环。 默认情况下，数据保护密钥未加密。 确保网络共享的文件权限仅限于应用在其下运行的 Windows 帐户。 可使用 X509 证书来保护静态密钥。 考虑允许用户上传证书的机制：将证书置于用户信任的证书存储中，并确保这些证书对所有运行用户应用的计算机都可用。 有关详细信息，请参阅[配置 ASP.NET Core 数据保护](xref:security/data-protection/configuration/overview)。

* **配置 IIS 应用程序池以加载用户配置文件**

  此设置位于应用池“高级设置”下的“进程模型”部分 。 将“加载用户配置文件”设置为 `True`。 如果设置为 `True`，会将密钥存储在用户配置文件目录中，并使用 DPAPI 和特定于用户帐户的密钥进行保护。 密钥保存在 %LOCALAPPDATA%/ASP.NET/DataProtection-Keys 文件夹中。

  同时还必须启用应用池的 [setProfileEnvironment attribute](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)。 `setProfileEnvironment` 的默认值为 `true`。 在某些情况下（例如，Windows 操作系统），将 `setProfileEnvironment` 设置为 `false`。 如果密钥未按预期存储在用户配置文件目录中，请执行以下操作：

  1. 导航到 %windir%/system32/inetsrv/config 文件夹。
  1. 打开 applicationHost.config 文件。
  1. 查找 `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 元素。
  1. 确认 `setProfileEnvironment` 属性不存在，这会将值默认设置为 `true`，或者将属性的值显式设置为 `true`。

* **将文件系统用作密钥环存储**

  调整应用代码，[将文件系统用作密钥环存储](xref:security/data-protection/configuration/overview)。 使用 X509 证书保护密钥环，并确保该证书是受信任的证书。 如果它是自签名证书，则将该证书放置于受信任的根存储中。

  当在 Web 场中使用 IIS 时：

  * 使用所有计算机都可以访问的文件共享。
  * 将 X509 证书部署到每台计算机。 [通过代码配置数据保护](xref:security/data-protection/configuration/overview)。

* **设置用于数据保护的计算机范围的策略**

  数据保护系统对以下操作提供有限支持：为使用数据保护 API 的所有应用设置默认[计算机范围的策略](xref:security/data-protection/configuration/machine-wide-policy)。 有关详细信息，请参阅 <xref:security/data-protection/introduction>。

## <a name="virtual-directories"></a>虚拟目录

ASP.NET Core 应用不支持 [IIS 虚拟目录](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories)。 可将应用托管为[子应用程序](#sub-applications)。

## <a name="sub-applications"></a>子应用程序

可将 ASP.NET Core 应用托管为 [IIS 子应用程序（子应用）](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications)。 子应用的路径成为根应用 URL 的一部分。

子应用内的静态资产链接应使用波形符-斜杠 (`~/`) 符号。 波形符-斜杠符号触发[标记帮助器](xref:mvc/views/tag-helpers/intro)，来将子应用的基路径追加到呈现的相关链接前面。 对于 `/subapp_path` 处的子应用，使用 `src="~/image.png"` 链接的图像将呈现为 `src="/subapp_path/image.png"`。 根应用的静态文件中间件不处理静态文件请求。 此请求由子应用的静态文件中间件处理。

若将静态资产的 `src` 属性设置为绝对路径（如 `src="/image.png"`），则呈现的链接不包含子应用的基路径。 根应用的静态文件中间件试图从根应用的 [web 根目录](xref:fundamentals/index#web-root)提供资产，这会导致“404 - 找不到”响应生成，除非可以从根应用获得静态资产。

若要将 ASP.NET Core 应用作为子应用托管在其他 ASP.NET Core 应用下：

1. 为此子应用创建应用池。 将“.NET CLR 版本”设置为“无托管代码”，因为将启动 .NET Core 的核心公共语言运行时 (CoreCLR) ，将应用托管在工作进程中，而非桌面 CLR (.NET CLR) 中 。

1. 在 IIS 管理器中添加根网站，并且此子应用在根网站的某个文件夹中。

1. 在 IIS 管理器中右击此子应用文件夹，并选择“转换为应用程序”。

1. 在“添加应用程序”对话框中，使用“应用程序池”的“选择”按钮来分配为子应用创建的应用池。   选择“确定”。

使用进程内托管模型时，需要向子应用分配单独的应用池。

有关进程内托管模型及 ASP.NET Core 模块配置的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

## <a name="configuration-of-iis-with-webconfig"></a>使用 web.config 配置 IIS

IIS 配置受用于 IIS 方案（适用于包含 ASP.NET Core 模块的 ASP.NET Core 应用）的 web.config 的 `<system.webServer>` 部分影响。 例如，IIS 配置适用于动态压缩。 如果在服务器一级将 IIS 配置为使用动态压缩，可通过应用的 web.config 文件中的 `<urlCompression>` 元素，对 ASP.NET Core 应用禁用它。

有关详细信息，请参阅下列主题：

* [\<system.webServer> 的配置参考](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

要为在独立应用池中运行的各应用设置环境变量（IIS 10.0 或更高版本中支持此操作），请参阅 IIS 参考文档中[环境变量 \<environmentVariables>](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) 主题下的“AppCmd.exe 命令”部分。

## <a name="configuration-sections-of-webconfig"></a>web.config 的配置节

ASP.NET Core 应用不会使用 web.config 中的 ASP.NET 4.x 应用的配置部分进行配置：

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

会使用其他的配置提供程序配置 ASP.NET Core 应用。 有关详细信息，请参阅[配置](xref:fundamentals/configuration/index)。

## <a name="application-pools"></a>应用程序池

由托管模型决定应用池隔离：

* 托管在进程内：应用需要在单独的应用池中运行。
* 托管在进程外：建议在每个应用自己的应用池中运行各应用，以彼此隔离。

IIS“添加网站”对话框默认为每应用一个应用池。 提供了站点名称时，该文本会自动传输到“应用程序池”文本框 。 添加站点时，会使用该站点名称创建新的应用池。

## <a name="application-pool-no-locidentity"></a>应用程序池 Identity

通过应用池标识帐户，可以在唯一帐户下运行应用，而无需创建和管理域或本地帐户。 在 IIS 8.0 或更高版本上，IIS 管理员工作进程 (WAS) 将使用新应用池的名称创建一个虚拟帐户，并默认在此帐户下运行应用池的工作进程。 在 IIS 管理控制台中，确保应用池“高级设置”下的“Identity”设置为使用 ApplicationPoolIdentity  ：

![应用程序池“高级设置”对话框](index/_static/apppool-identity.png)

IIS 管理进程使用 Windows 安全系统中应用池的名称创建安全标识符。 可使用此标识保护资源。 但是，此标识不是真实的用户帐户，不会在 Windows 用户管理控制台中显示。

如果 IIS 工作进程需要对应用的高级访问权限，请为包含该应用的目录修改访问控制列表 (ACL)：

1. 打开 Windows 资源管理器并导航到目录。

1. 右键单击该目录，然后选择“属性”。

1. 在“安全”选项卡下，选择“编辑”按钮，然后单击“添加”按钮  。

1. 选择“位置”按钮，并确保该系统处于选中状态。

1. 在“输入要选择的对象名”区域中输入 `IIS AppPool\{APP POOL NAME}`，其中 `{APP POOL NAME}` 占位符为应用池名称。 选择“检查名称”按钮。 对于 DefaultAppPool，请使用 `IIS AppPool\DefaultAppPool` 检查名称。 如果选择了“检查名称”按钮，对象名称区域中将指示 `DefaultAppPool`。 无法直接在对象名称区域中输入应用池名称。 检查对象名称时使用 `IIS AppPool\{APP POOL NAME}` 格式，其中 `{APP POOL NAME}` 占位符是应用池名称。

   ![应用文件夹的“选择用户或组”对话框：在选择“检查名称”前，将“DefaultAppPool\"”的应用池名称追加到对象名称区域中的“IIS AppPool”。](index/_static/select-users-or-groups-1.png)

1. 选择“确定”。

   ![应用文件夹的“选择用户或组”对话框：在你选择“检查名称”后，对象名称“DefaultAppPool”显示在对象名称区域中。](index/_static/select-users-or-groups-2.png)

1. 默认情况下应授予读取 &amp; 执行权限。 根据需要请提供其他权限。

也可使用 ICACLS 工具在命令提示符处授予访问权限。 以 `DefaultAppPool` 为例，使用了以下命令：

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

有关详细信息，请参阅 [icacls](/windows-server/administration/windows-commands/icacls) 主题。

## <a name="http2-support"></a>HTTP/2 支持

以下 IIS 部署方案中的 ASP.NET Core 支持 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* 进程内
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * TLS 1.2 或更高版本的连接
* 进程外
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * 面向公众的边缘服务器连接使用 HTTP/2，但与 [Kestrel 服务器](xref:fundamentals/servers/kestrel)的反向代理连接使用 HTTP/1.1。
  * TLS 1.2 或更高版本的连接

对于建立了 HTTP/2 连接时的进程内部署，[`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 报告 `HTTP/2`。 对于建立了 HTTP/2 连接时的进程外部署，[`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 报告 `HTTP/1.1`。

若要详细了解进程内和进程外托管模型，请参阅 <xref:host-and-deploy/aspnet-core-module>。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 有关使用 IIS 部署的 HTTP/2 配置的详细信息，请参阅 [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)。

## <a name="cors-preflight-requests"></a>CORS 预检请求

本部分仅适用于面向 .NET Framework 的 ASP.NET Core 应用程序。

对于面向 .NET Framework 的 ASP.NET Core 应用程序，默认情况下，IIS 不会将 OPTIONS 请求传递给应用程序。 若要了解如何在 `web.config` 中配置应用的 IIS 处理程序以传递 OPTIONS 请求，请参阅[在 ASP.NET Web API 2 中启用跨域请求：CORS 的工作原理](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)。

## <a name="application-initialization-module-and-idle-timeout"></a>应用程序初始化模块和空闲超时

通过 ASP.NET Core 模板版本 2 托管在 IIS 中时：

* [应用程序初始化模块](#application-initialization-module)：可以将托管在[进程内](#in-process-hosting-model)或[进程外](#out-of-process-hosting-model)的应用配置为在工作进程重启或服务器重启后自动启动。
* [空闲超时](#idle-timeout)：可以将托管在[进程内](#in-process-hosting-model)的应用配置为在非活动时段不超时。

### <a name="application-initialization-module"></a>应用程序初始化模块

适用于托管在进程内和进程外的应用。

[IIS 应用程序初始化](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)是一种 IIS 功能，可在应用池启动或回收时向应用发送 HTTP 请求。 该请求会触发应用启动。 默认情况下，IIS 向应用的根 URL (`/`) 发出请求以初始化应用（有关配置的详细信息，请参阅[更多资源](#application-initialization-module-and-idle-timeout-additional-resources)）。

确认已启用 IIS 应用程序初始化角色功能：

Windows 7 或更高版本桌面系统，在本地使用 IIS 时：

1. 导航到“控制面板”>“程序”>“程序和功能”>“打开或关闭 Windows 功能”（位于屏幕左侧）   。
1. 打开“Internet Information Services”>“万维网服务”>“应用程序开发功能”。
1. 选中“应用程序初始化”的复选框。

Windows Server 2008 R2 或更高版本：

1. 打开“添加角色和功能向导”。
1. 在“选择角色服务”面板中，打开“应用程序开发”节点 。
1. 选中“应用程序初始化”的复选框。

使用下面的任一方法为站点启用“应用程序初始化模块”：

* 使用 IIS 管理器：

  1. 在“连接”面板中选择“应用程序池”。
  1. 在列表中右键单击应用的应用池，并选择“高级设置”。
  1. 默认的“启动模式”为“按需” 。 将“启动模式”设置为“始终运行”。 选择“确定”。
  1. 打开“连接”面板中的“网站”节点。
  1. 右键单击应用，并选择“管理网站”>“高级设置” 。
  1. 默认的“预加载已启用”设置为“False” 。 将“预加载已启用”设置为“True”。 选择“确定”  。

* 使用 `web.config` 向应用的 web.config` 文件中的 `<system.webServer>` 元素添加 `<applicationInitialization>` 元素（其中 `doAppInitAfterRestart` 设置为 `true`）：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <applicationInitialization doAppInitAfterRestart="true" />
      </system.webServer>
    </location>
  </configuration>
  ```

### <a name="idle-timeout"></a>空闲超时

仅适用于托管在进程内的应用。

若要防止应用进入空闲状态，请使用 IIS 管理器设置应用池的空闲超时：

1. 在“连接”面板中选择“应用程序池”。
1. 在列表中右键单击应用的应用池，并选择“高级设置”。
1. 默认的“空闲超时(分钟)”为“20”分钟 。 将“空闲超时(分钟)”设置为“0”（零） 。 选择“确定”。
1. 回收工作进程。

若要防止托管在[进程外](#out-of-process-hosting-model)的应用超时，请使用下列任一方法：

* 从外部服务 ping 应用，使其保持运行状态。
* 如果应用仅托管后台服务，请避免使用 IIS 托管，而是[使用 Windows 服务托管 ASP.NET Core 应用](xref:host-and-deploy/windows-service)。

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>关于应用程序初始化模块和空闲超时的更多资源

* [IIS 8.0 应用程序初始化](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)
* [应用程序初始化 \<applicationInitialization>](/iis/configuration/system.webserver/applicationinitialization/)。
* [应用程序池 \<processModel> 的进程模型设置](/iis/configuration/system.applicationhost/applicationpools/add/processmodel)。

## <a name="deployment-resources-for-iis-administrators"></a>面向 IIS 管理员的部署资源

* [IIS 文档](/iis)
* [IIS 中 IIS 管理器入门](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8)
* [.NET Core 应用程序部署](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:index>
* [Microsoft IIS 官方网站](https://www.iis.net/)
* [Windows Server 技术内容库](/windows-server/windows-server)
* [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

若要学习将 ASP.NET Core 应用发布到 IIS 服务器的教程，请参阅<xref:tutorials/publish-to-iis>。

[安装 .NET Core 托管捆绑包](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>支持的操作系统

支持下列操作系统：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

[HTTP.sys 服务器](xref:fundamentals/servers/httpsys)（以前称为 WebListener）无法以反向代理配置形式与 IIS 结合使用。 请使用 [Kestrel 服务器](xref:fundamentals/servers/kestrel)。

有关在 Azure 上托管的信息，请参阅<xref:host-and-deploy/azure-apps/index>。

有关疑难解答指南，请参阅 <xref:test/troubleshoot>。

## <a name="supported-platforms"></a>受支持的平台

支持针对 32 位 (x86) 或 64 位 (x64) 部署发布的应用。 使用 32 位 (x86) .NET Core SDK 部署 32 位应用，除非应用符合以下情况：

* 需要适用于 64 位应用的更大虚拟内存地址空间。
* 需要更大 IIS 堆栈大小。
* 具有 64 位本机依赖项。

使用 64 位 (x64) .NET Core SDK 发布 64 位应用。 主机系统必须具有 64 位运行时。

## <a name="hosting-models"></a>托管模型

### <a name="in-process-hosting-model"></a>进程内托管模型

使用进程内托管，ASP.NET Core 在与其 IIS 工作进程相同的进程中运行。 进程内托管的性能高于进程外托管的性能，因为：

* 请求并不通过环回适配器进行代理。 环回适配器是一个网络接口，用于将传出的网络流量返回给同一计算机。

IIS 使用 [Windows 进程激活服务 (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) 处理进程管理。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)：

* 执行应用初始化。
  * 加载 [CoreCLR](/dotnet/standard/glossary#coreclr)。
  * 调用 `Program.Main`。
* 处理 IIS 本机请求的生存期。

定目标到 .NET Framework 的 ASP.NET Core 应用不支持进程内托管模型。

下图说明了 IIS、ASP.NET Core 模块和进程内托管的应用之间的关系：

![进程内托管方案中的 ASP.NET Core 模块](index/_static/ancm-inprocess.png)

请求从 Web 到达内核模式 HTTP.sys 驱动程序。 驱动程序将本机请求路由到网站的配置端口上的 IIS，通常为 80 (HTTP) 或 443 (HTTPS)。 ASP.NET Core 模块接收本机请求，并将其传递给 IIS HTTP 服务器 (`IISHttpServer`)。 IIS HTTP 服务器是将请求从本机转换为托管的 IIS 进程内服务器实现。

IIS HTTP 服务器处理请求之后，请求会被推送到 ASP.NET Core 中间件管道中。 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。 应用的响应通过 IIS HTTP 服务器传递回 IIS。 IIS 将响应发送到发起请求的客户端。

进程内托管选择使用现有应用，但 [dotnet new](/dotnet/core/tools/dotnet-new) 模板默认使用所有 IIS 和 IIS Express 方案的进程内托管模型。

`CreateDefaultBuilder` 将通过调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS%2A> 方法添加 <xref:Microsoft.AspNetCore.Hosting.Server.IServer> 实例，来启动 [CoreCLR](/dotnet/standard/glossary#coreclr) 并在 IIS 工作进程（w3wp.exe 或 iisexpress.exe）内托管应用。  性能测试表明，与在进程外托管应用并将请求代理到 [Kestrel](xref:fundamentals/servers/kestrel) 服务器相比，在进程中托管 .NET Core 应用可以大大提升请求吞吐量。

### <a name="out-of-process-hosting-model"></a>进程外托管模型

由于运行 ASP.NET Core 的进程与 IIS 工作进程分开，因此 ASP.NET Core 模块会负责进程管理。 该模块在第一个请求到达时启动 ASP.NET Core 应用的进程，并在应用关闭或崩溃时重新启动该应用。 这基本上与在 [Windows 进程激活服务 (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) 托管的进程内运行的应用中出现的行为相同。

下图说明了 IIS、ASP.NET Core 模块和进程外托管的应用之间的关系：

![进程外托管方案中的 ASP.NET Core 模块](index/_static/ancm-outofprocess.png)

请求从 Web 到达内核模式 HTTP.sys 驱动程序。 驱动程序将请求路由到网站的配置端口上的 IIS，通常为 80 (HTTP) 或 443 (HTTPS)。 该模块将该请求转发到应用的随机端口（非端口 80/443）上的 Kestrel。

该模块在启动时通过环境变量指定端口，<xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> 扩展将服务器配置为侦听 `http://localhost:{PORT}`。 执行其他检查，拒绝不是来自该模块的请求。 该模块不支持 HTTPS 转发，因此即使请求由 IIS 通过 HTTPS 接收，它们还是通过 HTTP 转发。

Kestrel 从模块获取请求后，请求会被推送到 ASP.NET Core 中间件管道中。 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。 IIS 集成添加的中间件会将方案、远程 IP 和 pathbase 更新到帐户以将请求转发到 Kestrel。 应用的响应传递回 IIS，IIS 将响应推送回发起请求的 HTTP 客户端。

有关 ASP.NET Core 模块配置指南，请参阅 <xref:host-and-deploy/aspnet-core-module>。

有关托管的详细信息，请参阅[在 ASP.NET Core 中托管](xref:fundamentals/index#host)。

## <a name="application-configuration"></a>应用程序配置

### <a name="enable-the-iisintegration-components"></a>启用 IISIntegration 组件

在 `CreateWebHostBuilder` 中生成主机 ( *Program.cs* )，请调用 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> 以启用 IIS 集成：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        ...
```

有关 `CreateDefaultBuilder` 的详细信息，请参阅 <xref:fundamentals/host/web-host#set-up-a-host>。

### <a name="iis-options"></a>IIS 选项

**进程内承载模型**

要配置 IIS 服务器选项，请在 <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices%2A> 中包括 <xref:Microsoft.AspNetCore.Builder.IISServerOptions> 的服务配置。 下面的示例禁用 AutomaticAuthentication：

```csharp
services.Configure<IISServerOptions>(options => 
{
    options.AutomaticAuthentication = false;
});
```

| 选项                         | 默认 | 设置 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | 若为 `true`，IIS 服务器将设置经过 [Windows 身份验证](xref:security/authentication/windowsauth)进行身份验证的 `HttpContext.User`。 若为 `false`，服务器仅提供 `HttpContext.User` 的标识并在 `AuthenticationScheme` 显式请求时响应质询。 必须在 IIS 中启用 Windows 身份验证使 `AutomaticAuthentication` 得以运行。 有关详细信息，请参阅 [Windows 身份验证](xref:security/authentication/windowsauth)。 |
| `AuthenticationDisplayName`    | `null`  | 设置在登录页上向用户显示的显示名。 |

**进程外承载模型**

要配置 IIS 选项，请在 <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices*> 中包括 <xref:Microsoft.AspNetCore.Builder.IISOptions> 的服务配置。 下面的示例阻止应用填充 `HttpContext.Connection.ClientCertificate`：

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

配置转发头中间件的 [IIS 集成中间件](#enable-the-iisintegration-components)和 ASP.NET Core 模块将配置为转发方案 (HTTP/HTTPS) 和发出请求的远程 IP 地址。 对于托管在其他代理服务器和负载均衡器后方的应用，可能需要附加配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

### <a name="webconfig-file"></a>web.config 文件

web.config 文件配置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 发布项目时，MSBuild 目标 (`_TransformWebConfig`) 负责创建、转换和发布 web.config 文件。 此目标位于 Web SDK 目标 (`Microsoft.NET.Sdk.Web`) 中。 SDK 设置在项目文件的顶部：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

如果项目中不存在 web.config 文件，则会使用正确的 processPath 和参数创建该文件，以便配置 ASP.NET Core 模块，并将该文件移动到[已发布的输出](xref:host-and-deploy/directory-structure)  。

如果项目中存在 web.config 文件，则会使用正确的 processPath 和参数转换该文件，以便配置 ASP.NET Core 模块，并将该文件移动到已发布的输出。 转换不会修改文件中的 IIS 配置设置。

web.config 文件可能会提供其他 IIS 配置设置，以控制活动的 IIS 模块。 有关能够处理 ASP.NET Core 应用请求的 IIS 模块的信息，请参阅 [IIS 模块](xref:host-and-deploy/iis/modules)主题。

要防止 Web SDK 转换 web.config 文件，请使用项目文件中的 \<IsTransformWebConfigDisabled> 属性：

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

在禁用 Web SDK 转换文件时，开发人员应手动设置 processPath 和参数。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

### <a name="webconfig-file-location"></a>web.config 文件位置

为了正确设置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)，web.config 文件必须位于已部署应用的[内容根](xref:fundamentals/index#content-root)路径（通常为应用基路径）中。 该位置与向 IIS 提供的网站物理路径相同。 若要使用 Web 部署发布多个应用，应用的根路径中需要包含 web.config 文件。

敏感文件存在于应用的物理路径中，如 \<assembly>.runtimeconfig.json、\<assembly>.xml（XML 文档注释）和 \<assembly>.deps.json  。 如果存在 web.config 文件且站点正常启动，则请求获取这些敏感文件时，IIS 不会提供。 如果缺少 web.config 文件、命名不正确，或无法配置站点以正常启动，IIS 可能会公开提供敏感文件。

部署中必须始终存在 web.config 文件且正确命名，并可以配置站点以正常启动。 **请勿从生产部署中删除 web.config 文件。**

### <a name="transform-webconfig"></a>转换 web.config

如果需要在发布时转换 web.config（例如，基于配置、配置文件或环境设置环境变量），请参阅 <xref:host-and-deploy/iis/transform-webconfig>。

## <a name="iis-configuration"></a>IIS 配置

**Windows Server 操作系统**

启用 Web 服务器 (IIS) 服务器角色并建立角色服务。

1. 通过“管理”菜单或“服务器管理器”中的链接使用“添加角色和功能”向导。 在“服务器角色”步骤中，选中“Web 服务器(IIS)”框 。

   ![在选择服务器角色步骤中选择了“Web 服务器 IIS”角色。](index/_static/server-roles-ws2016.png)

1. 在“功能”步骤后，为 Web 服务器 (IIS) 加载“角色服务”步骤。 选择所需 IIS 角色服务，或接受提供的默认角色服务。

   ![在选择角色服务步骤中选择了默认角色服务。](index/_static/role-services-ws2016.png)

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“Web 服务器” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“Web 服务器” > “应用开发”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 继续执行“确认”步骤，安装 Web 服务器角色和服务。 安装 Web 服务器 (IIS) 角色后无需重启服务器/IIS。

**Windows 桌面操作系统**

启用“IIS 管理控制台”和“万维网服务”。

1. 导航到“控制面板”>“程序”>“程序和功能”>“打开或关闭 Windows 功能”（位于屏幕左侧）   。

1. 打开“Internet Information Services”节点。 打开“Web 管理工具”节点。

1. 选中“IIS 管理控制台”框。

1. 选中“万维网服务”框。

1. 接受“万维网服务”的默认功能，或自定义 IIS 功能。

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“万维网服务” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“万维网服务” > “应用开发功能”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 如果 IIS 安装需要重新启动，则重新启动系统。

![在“Windows 功能”中选择了“IIS 管理控制台”和“万维网服务”。](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>安装 .NET Core 托管捆绑包

在托管系统上安装 .NET Core 托管捆绑包。 捆绑包可安装 .NET Core 运行时、.NET Core 库和 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 该模块允许 ASP.NET Core 应用在 IIS 后面运行。

> [!IMPORTANT]
> 如果在 IIS 之前安装了托管捆绑包，则必须修复捆绑包安装。 在安装 IIS 后再次运行托管捆绑包安装程序。
>
> 如果在安装 64 位 (x64) 版本的 .NET Core 之后安装了 Hosting Bundle，则可能看上去缺少 SDK（[未检测到 .NET Core SDK](xref:test/troubleshoot#no-net-core-sdks-were-detected)）。 要解决此问题，请参阅 <xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle>。

### <a name="download"></a>下载

1. 导航到 [ .NET Core](https://dotnet.microsoft.com/download/dotnet-core) 页面。
1. 选择所需的 .NET Core 版本。
1. 在“运行应用 - 运行时”列中，查找所需的 .NET Core 运行时版本的那一行。
1. 使用“托管捆绑包”链接下载安装程序。

> [!WARNING]
> 某些安装程序包含已到达其生命周期结束 (EOL) 且不再受 Microsoft 支持的发行版本。 有关详细信息，请参阅[支持策略](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)。

### <a name="install-the-hosting-bundle"></a>安装托管捆绑包

1. 在服务器上运行安装程序。 通过管理员命令行界面运行安装程序时，以下参数可用：

   * `OPT_NO_ANCM=1`：跳过安装 ASP.NET Core 模块。
   * `OPT_NO_RUNTIME=1`：跳过安装 .NET Core 运行时。 当服务器仅承载[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) 时使用。
   * `OPT_NO_SHAREDFX=1`：跳过安装 ASP.NET 共享框架（ASP.NET 运行时）。 当服务器仅承载[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) 时使用。
   * `OPT_NO_X86=1`：跳过安装 x86 运行时。 确定不会托管 32 位应用时，请使用此参数。 如果有可能会同时托管 32 位和 64 位应用，请勿使用此参数，并安装两个运行时。
   * `OPT_NO_SHARED_CONFIG_CHECK=1`：当共享配置 (applicationHost.config) 与 IIS 安装位于同一台计算机上时，禁止对使用的是否是 IIS 共享配置进行检查。 仅适用于 ASP.NET Core 2.2 或更高版本托管捆绑程序安装程序。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>。
1. 重新启动系统，或在命令行界面中执行以下命令：

   ```console
   net stop was /y
   net start w3svc
   ```
   重启 IIS 会选取安装程序对系统 PATH（环境变量）所作的更改。

安装托管捆绑包时，无需在 IIS 中手动停止各个站点。 IIS 重新启动后，托管应用（IIS 站点）也将重新启动。 应用收到第一个请求（包括来自[应用程序初始化模块](#application-initialization-module-and-idle-timeout)）后，将再次启动。

ASP.NET Core 采用共享框架包的修补程序版本的前滚行为。 当 IIS 托管的应用重新启动 IIS 时，应用会在收到第一个请求后使用其引用的包的最新修补程序版本进行加载。 如果未重新启动 IIS，应用会在其工作进程被回收并收到第一个请求后重新启动并显示前滚行为。

> [!NOTE]
> 有关 IIS 共享配置的信息，请参阅[使用 IIS 共享配置的 ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration)。

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>使用 Visual Studio 进行发布时安装 Web 部署

使用 [Web 部署](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)将应用部署到服务器时，请在服务器上安装最新版本的 Web 部署。 要安装 Web 部署，请使用 [Web 平台安装程序 (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) 或直接从 [Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=43717)获取安装程序。 建议使用 WebPI。 WebPI 为托管提供程序提供独立的安装程序和配置。

## <a name="create-the-iis-site"></a>创建 IIS 站点

1. 在托管系统上，创建一个文件夹以包含应用已发布的文件夹和文件。 在接下来的步骤中，文件夹路径作为应用程序的物理路径提供给 IIS。 有关应用程序部署文件夹和文件布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。

1. 在 IIS 管理器中，打开“连接”面板中的服务器节点。 右键单击“站点”文件夹。 选择上下文菜单中的“添加网站”。

1. 提供网站名称，并将物理路径设置为应用的部署文件夹 。 提供“绑定”配置，并通过选择“确定”创建网站：

   ![在“添加网站”步骤中提供网站名称、物理路径和主机名。](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定可能会为应用带来安全漏洞。 此行为同时适用于强通配符和弱通配符。 使用显式主机名而不是通配符。 如果可控制整个父域（区别于易受攻击的 `*.com`），则子域通配符绑定（例如，`*.mysub.com`）不具有此安全风险。 有关详细信息，请参阅 [rfc7230 第 5.4 条](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器节点下，选择“应用程序池”。

1. 右键单击站点的应用池，然后从上下文菜单中选择“基本设置”。

1. 在“编辑应用程序池”窗口中，将“.NET CLR 版本”设置为“无托管代码”：

   ![将“.NET CLR 版本”设置为“无托管代码”。](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core 在单独的进程中运行，并管理运行时。 ASP.NET Core 不依赖桌面 CLR (.NET CLR) 加载：将启动 .NET Core 的 Core 公共语言运行时 (CoreCLR) ，在工作进程中托管应用。 将“.NET CLR 版本”设置为“无托管代码”是可选步骤，但建议采用此设置。

1. *ASP.NET Core 2.2 或更高版本* ：对于使用 [进程内托管模型](#in-process-hosting-model)的 64 位 (x64) [独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)，为 32 位 (x86) 进程禁用应用池。

   在 IIS 管理器 >“应用程序池”的“操作”侧栏中，选择“设置应用程序池默认设置”或“高级设置”。 找到“启用 32 位应用程序”并将值设置为 `False`。 此设置不会影响针对[进程外托管](xref:host-and-deploy/aspnet-core-module#out-of-process-hosting-model)部署的应用。

1. 确认进程模型标识拥有适当的权限。

   如果将应用池的默认标识（“进程模型” > “Identity”）从 ApplicationPoolIdentity 更改为另一标识，请确保新标识拥有对应用文件夹、数据库和其他所需资源的必需访问权限  。 例如，应用池需要对文件夹的读取和写入权限，以便应用在其中读取和写入文件。

**Windows 身份验证配置（可选）**  
有关详细信息，请参阅[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

## <a name="deploy-the-app"></a>部署应用

将应用程序部署到 IIS 物理路径文件夹中，该文件夹是在[创建 IIS 站点](#create-the-iis-site)部分中创建的。 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)是建议使用的部署机制，但有几个选项可以将应用程序从项目的发布文件夹移动到托管系统的部署文件夹。

### <a name="web-deploy-with-visual-studio"></a>在 Visual Studio 内使用 Web 部署

要了解如何创建用于 Web 部署的发布配置文件，请参阅[用于 ASP.NET Core 应用部署的 Visual Studio 发布配置文件](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)。 如果托管提供程序提供了发布配置文件或支持创建发布配置文件，请下载配置文件并使用 Visual Studio 的“发布”对话框将其导入：

![“发布”对话框页](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>在 Visual Studio 之外使用 Web 部署

也可以在 Visual Studio 之外从命令行使用 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)。 有关详细信息，请参阅 [Web Deployment Tool](/iis/publish/using-web-deploy/use-the-web-deployment-tool)（Web 部署工具）。

### <a name="alternatives-to-web-deploy"></a>Web 部署的替代方法

有多种方法可将应用移动到托管系统，例如手动复制、[Xcopy](/windows-server/administration/windows-commands/xcopy)、[Robocopy](/windows-server/administration/windows-commands/robocopy) 或 [PowerShell](/powershell/)，可使用其中任何一种方法。

有关将 ASP.NET Core 部署到 IIS 的详细信息，请参阅[面向 IIS 管理员的部署资源](#deployment-resources-for-iis-administrators)部分。

## <a name="browse-the-website"></a>浏览网站

将应用部署到托管系统后，向应用的一个公共终结点发出请求。

在以下示例中，站点被绑定到端口 `80` 上 `www.mysite.com` 的 IIS 主机名中。 向 `http://www.mysite.com` 发出请求：

![Microsoft Edge 浏览器已加载 IIS 启动页。](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>锁定的部署文件

如果应用正在运行，部署文件夹中的文件会被锁定。 在部署期间，无法覆盖已锁定的文件。 若要在部署中解除已锁定的文件，请使用以下方法之一停止应用池：

* 使用 Web 部署并在项目文件中引用 `Microsoft.NET.Sdk.Web`。 系统会在 Web 应用目录的根目录中放置一个 app_offline.htm 文件。 该文件存在时，ASP.NET Core 模块会在部署过程中正常关闭该应用并提供 app_offline.htm 文件。 有关详细信息，请参阅 [ASP.NET Core 模块配置参考](xref:host-and-deploy/aspnet-core-module#app_offlinehtm)。
* 在服务器上的 IIS 管理器中手动停止应用池。
* 使用 PowerShell 删除 app_offline.html（需要使用 PowerShell 5 或更高版本）：

  ```powershell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm

  ```

## <a name="data-protection"></a>数据保护

[ASP.NET Core 数据保护堆栈](xref:security/data-protection/introduction)由多个 ASP.NET Core [中间件](xref:fundamentals/middleware/index)使用，包括用于身份验证的中间件。 即使用户代码不调用数据保护 API，也应该使用部署脚本或在用户代码中配置数据保护，以创建持久的加密[密钥存储](xref:security/data-protection/implementation/key-management)。 如果不配置数据保护，则密钥存储在内存中。重启应用时，密钥会被丢弃。

如果密钥环存储于内存中，则在应用重启时：

* 所有基于 cookie 的身份验证令牌都无效。 
* 用户需要在下一次请求时再次登录。 
* 无法再解密使用密钥环保护的任何数据。 这可能包括 [CSRF 令牌](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)和 [ASP.NET Core MVC TempData cookie](xref:fundamentals/app-state#tempdata)。

若要在 IIS 下配置数据保护以持久化密钥环，请使用以下方法之一：

* **创建数据保护注册表项**

  ASP.NET Core 应用使用的数据保护密钥存储在应用外部的注册表中。 要持久保存给定应用的密钥，需为应用池创建注册表项。

  对于独立的非 Web 场 IIS 安装，可以对用于 ASP.NET Core 应用的每个应用池使用[数据保护 Provision-AutoGenKeys.ps1 PowerShell 脚本](https://github.com/dotnet/AspNetCore/blob/master/src/DataProtection/Provision-AutoGenKeys.ps1)。 此脚本在 HKLM 注册表中创建注册表项，仅应用程序的应用池工作进程帐户可对其进行访问。 通过计算机范围的密钥使用 DPAPI 对密钥静态加密。

  在 web 场方案中，可以将应用配置为使用 UNC 路径存储其数据保护密钥环。 默认情况下，数据保护密钥未加密。 确保网络共享的文件权限仅限于应用在其下运行的 Windows 帐户。 可使用 X509 证书来保护静态密钥。 考虑允许用户上传证书的机制：将证书置于用户信任的证书存储中，并确保这些证书对所有运行用户应用的计算机都可用。 有关详细信息，请参阅[配置 ASP.NET Core 数据保护](xref:security/data-protection/configuration/overview)。

* **配置 IIS 应用程序池以加载用户配置文件**

  此设置位于应用池“高级设置”下的“进程模型”部分 。 将“加载用户配置文件”设置为 `True`。 如果设置为 `True`，会将密钥存储在用户配置文件目录中，并使用 DPAPI 和特定于用户帐户的密钥进行保护。 密钥保存在 %LOCALAPPDATA%/ASP.NET/DataProtection-Keys 文件夹中。

  同时还必须启用应用池的 [setProfileEnvironment attribute](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)。 `setProfileEnvironment` 的默认值为 `true`。 在某些情况下（例如，Windows 操作系统），将 `setProfileEnvironment` 设置为 `false`。 如果密钥未按预期存储在用户配置文件目录中，请执行以下操作：

  1. 导航到 %windir%/system32/inetsrv/config 文件夹。
  1. 打开 applicationHost.config 文件。
  1. 查找 `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 元素。
  1. 确认 `setProfileEnvironment` 属性不存在，这会将值默认设置为 `true`，或者将属性的值显式设置为 `true`。

* **将文件系统用作密钥环存储**

  调整应用代码，[将文件系统用作密钥环存储](xref:security/data-protection/configuration/overview)。 使用 X509 证书保护密钥环，并确保该证书是受信任的证书。 如果它是自签名证书，则将该证书放置于受信任的根存储中。

  当在 Web 场中使用 IIS 时：

  * 使用所有计算机都可以访问的文件共享。
  * 将 X509 证书部署到每台计算机。 [通过代码配置数据保护](xref:security/data-protection/configuration/overview)。

* **设置用于数据保护的计算机范围的策略**

  数据保护系统对以下操作提供有限支持：为使用数据保护 API 的所有应用设置默认[计算机范围的策略](xref:security/data-protection/configuration/machine-wide-policy)。 有关详细信息，请参阅 <xref:security/data-protection/introduction>。

## <a name="virtual-directories"></a>虚拟目录

ASP.NET Core 应用不支持 [IIS 虚拟目录](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories)。 可将应用托管为[子应用程序](#sub-applications)。

## <a name="sub-applications"></a>子应用程序

可将 ASP.NET Core 应用托管为 [IIS 子应用程序（子应用）](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications)。 子应用的路径成为根应用 URL 的一部分。

子应用内的静态资产链接应使用波形符-斜杠 (`~/`) 符号。 波形符-斜杠符号触发[标记帮助器](xref:mvc/views/tag-helpers/intro)，来将子应用的基路径追加到呈现的相关链接前面。 对于 `/subapp_path` 处的子应用，使用 `src="~/image.png"` 链接的图像将呈现为 `src="/subapp_path/image.png"`。 根应用的静态文件中间件不处理静态文件请求。 此请求由子应用的静态文件中间件处理。

若将静态资产的 `src` 属性设置为绝对路径（如 `src="/image.png"`），则呈现的链接不包含子应用的基路径。 根应用的静态文件中间件试图从根应用的 [web 根目录](xref:fundamentals/index#web-root)提供资产，这会导致“404 - 找不到”响应生成，除非可以从根应用获得静态资产。

若要将 ASP.NET Core 应用作为子应用托管在其他 ASP.NET Core 应用下：

1. 为此子应用创建应用池。 将“.NET CLR 版本”设置为“无托管代码”，因为将启动 .NET Core 的核心公共语言运行时 (CoreCLR) ，将应用托管在工作进程中，而非桌面 CLR (.NET CLR) 中 。

1. 在 IIS 管理器中添加根网站，并且此子应用在根网站的某个文件夹中。

1. 在 IIS 管理器中右击此子应用文件夹，并选择“转换为应用程序”。

1. 在“添加应用程序”对话框中，使用“应用程序池”的“选择”按钮来分配为子应用创建的应用池。   选择“确定”。

使用进程内托管模型时，需要向子应用分配单独的应用池。

有关进程内托管模型及 ASP.NET Core 模块配置的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

## <a name="configuration-of-iis-with-webconfig"></a>使用 web.config 配置 IIS

IIS 配置受用于 IIS 方案（适用于包含 ASP.NET Core 模块的 ASP.NET Core 应用）的 web.config 的 `<system.webServer>` 部分影响。 例如，IIS 配置适用于动态压缩。 如果在服务器一级将 IIS 配置为使用动态压缩，可通过应用的 web.config 文件中的 `<urlCompression>` 元素，对 ASP.NET Core 应用禁用它。

有关详细信息，请参阅下列主题：

* [\<system.webServer> 的配置参考](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

要为在独立应用池中运行的各应用设置环境变量（IIS 10.0 或更高版本中支持此操作），请参阅 IIS 参考文档中[环境变量 \<environmentVariables>](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) 主题下的“AppCmd.exe 命令”部分。

## <a name="configuration-sections-of-webconfig"></a>web.config 的配置节

ASP.NET Core 应用不会使用 web.config 中的 ASP.NET 4.x 应用的配置部分进行配置：

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

会使用其他的配置提供程序配置 ASP.NET Core 应用。 有关详细信息，请参阅[配置](xref:fundamentals/configuration/index)。

## <a name="application-pools"></a>应用程序池

由托管模型决定应用池隔离：

* 托管在进程内：应用需要在单独的应用池中运行。
* 托管在进程外：建议在每个应用自己的应用池中运行各应用，以彼此隔离。

IIS“添加网站”对话框默认为每应用一个应用池。 提供了站点名称时，该文本会自动传输到“应用程序池”文本框 。 添加站点时，会使用该站点名称创建新的应用池。

## <a name="application-pool-no-locidentity"></a>应用程序池 Identity

通过应用池标识帐户，可以在唯一帐户下运行应用，而无需创建和管理域或本地帐户。 在 IIS 8.0 或更高版本上，IIS 管理员工作进程 (WAS) 将使用新应用池的名称创建一个虚拟帐户，并默认在此帐户下运行应用池的工作进程。 在 IIS 管理控制台中，确保应用池“高级设置”下的“Identity”设置为使用 ApplicationPoolIdentity  ：

![应用程序池“高级设置”对话框](index/_static/apppool-identity.png)

IIS 管理进程使用 Windows 安全系统中应用池的名称创建安全标识符。 可使用此标识保护资源。 但是，此标识不是真实的用户帐户，不会在 Windows 用户管理控制台中显示。

如果 IIS 工作进程需要对应用的高级访问权限，请为包含该应用的目录修改访问控制列表 (ACL)：

1. 打开 Windows 资源管理器并导航到目录。

1. 右键单击该目录，然后选择“属性”。

1. 在“安全”选项卡下，选择“编辑”按钮，然后单击“添加”按钮  。

1. 选择“位置”按钮，并确保该系统处于选中状态。

1. 在“输入要选择的对象名称”区域中输入 **IIS AppPool\\<app_pool_name>** 。 选择“检查名称”按钮。 有关 DefaultAppPool，请检查使用 IIS AppPool\DefaultAppPool 的名称。 当选择“检查名称”按钮时，对象名称区域中会显示 DefaultAppPool 的值。 无法直接在对象名称区域中输入应用池名称。 检查对象名称时，请使用 **IIS AppPool\\<app_pool_name>** 格式。

   ![应用文件夹的“选择用户或组”对话框：在选择“检查名称”前，将“DefaultAppPool\"”的应用池名称追加到对象名称区域中的“IIS AppPool”。](index/_static/select-users-or-groups-1.png)

1. 选择“确定”。

   ![应用文件夹的“选择用户或组”对话框：在你选择“检查名称”后，对象名称“DefaultAppPool”显示在对象名称区域中。](index/_static/select-users-or-groups-2.png)

1. 默认情况下应授予读取 &amp; 执行权限。 根据需要请提供其他权限。

也可使用 ICACLS 工具在命令提示符处授予访问权限。 以 DefaultAppPool 为例，使用以下命令：

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

有关详细信息，请参阅 [icacls](/windows-server/administration/windows-commands/icacls) 主题。

## <a name="http2-support"></a>HTTP/2 支持

以下 IIS 部署方案中的 ASP.NET Core 支持 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* 进程内
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * TLS 1.2 或更高版本的连接
* 进程外
  * Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
  * 面向公众的边缘服务器连接使用 HTTP/2，但与 [Kestrel 服务器](xref:fundamentals/servers/kestrel)的反向代理连接使用 HTTP/1.1。
  * TLS 1.2 或更高版本的连接

对于已建立 HTTP/2 连接时的进程内部署，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/2`。 对于已建立 HTTP/2 连接时的进程外部署，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/1.1`。

若要详细了解进程内和进程外托管模型，请参阅 <xref:host-and-deploy/aspnet-core-module>。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 有关使用 IIS 部署的 HTTP/2 配置的详细信息，请参阅 [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)。

## <a name="cors-preflight-requests"></a>CORS 预检请求

本部分仅适用于面向 .NET Framework 的 ASP.NET Core 应用程序。

对于面向 .NET Framework 的 ASP.NET Core 应用程序，默认情况下，IIS 不会将 OPTIONS 请求传递给应用程序。 若要了解如何在 web.config 中配置应用程序的 IIS 处理程序以传递 OPTIONS 请求，请参阅[在 ASP.NET Web API 2 中启用跨域请求：CORS 的工作原理](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)。

## <a name="application-initialization-module-and-idle-timeout"></a>应用程序初始化模块和空闲超时

通过 ASP.NET Core 模板版本 2 托管在 IIS 中时：

* [应用程序初始化模块](#application-initialization-module)：可以将托管在[进程内](#in-process-hosting-model)或[进程外](#out-of-process-hosting-model)的应用配置为在工作进程重启或服务器重启后自动启动。
* [空闲超时](#idle-timeout)：可以将托管在[进程内](#in-process-hosting-model)的应用配置为在非活动时段不超时。

### <a name="application-initialization-module"></a>应用程序初始化模块

适用于托管在进程内和进程外的应用。

[IIS 应用程序初始化](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)是一种 IIS 功能，可在应用池启动或回收时向应用发送 HTTP 请求。 该请求会触发应用启动。 默认情况下，IIS 向应用的根 URL (`/`) 发出请求以初始化应用（有关配置的详细信息，请参阅[更多资源](#application-initialization-module-and-idle-timeout-additional-resources)）。

确认已启用 IIS 应用程序初始化角色功能：

Windows 7 或更高版本桌面系统，在本地使用 IIS 时：

1. 导航到“控制面板”>“程序”>“程序和功能”>“打开或关闭 Windows 功能”（位于屏幕左侧）   。
1. 打开“Internet Information Services”>“万维网服务”>“应用程序开发功能”。
1. 选中“应用程序初始化”的复选框。

Windows Server 2008 R2 或更高版本：

1. 打开“添加角色和功能向导”。
1. 在“选择角色服务”面板中，打开“应用程序开发”节点 。
1. 选中“应用程序初始化”的复选框。

使用下面的任一方法为站点启用“应用程序初始化模块”：

* 使用 IIS 管理器：

  1. 在“连接”面板中选择“应用程序池”。
  1. 在列表中右键单击应用的应用池，并选择“高级设置”。
  1. 默认的“启动模式”为“按需” 。 将“启动模式”设置为“始终运行”。 选择“确定”。
  1. 打开“连接”面板中的“网站”节点。
  1. 右键单击应用，并选择“管理网站”>“高级设置” 。
  1. 默认的“预加载已启用”设置为“False” 。 将“预加载已启用”设置为“True”。 选择“确定”。

* 使用 web.config 将 `doAppInitAfterRestart` 已设置为 `true` 的 `<applicationInitialization>` 元素添加到应用的 web.config 文件中的 `<system.webServer>` 元素中 ：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <applicationInitialization doAppInitAfterRestart="true" />
      </system.webServer>
    </location>
  </configuration>
  ```

### <a name="idle-timeout"></a>空闲超时

仅适用于托管在进程内的应用。

若要防止应用进入空闲状态，请使用 IIS 管理器设置应用池的空闲超时：

1. 在“连接”面板中选择“应用程序池”。
1. 在列表中右键单击应用的应用池，并选择“高级设置”。
1. 默认的“空闲超时(分钟)”为“20”分钟 。 将“空闲超时(分钟)”设置为“0”（零） 。 选择“确定”。
1. 回收工作进程。

若要防止托管在[进程外](#out-of-process-hosting-model)的应用超时，请使用下列任一方法：

* 从外部服务 ping 应用，使其保持运行状态。
* 如果应用仅托管后台服务，请避免使用 IIS 托管，而是[使用 Windows 服务托管 ASP.NET Core 应用](xref:host-and-deploy/windows-service)。

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>关于应用程序初始化模块和空闲超时的更多资源

* [IIS 8.0 应用程序初始化](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)
* [应用程序初始化 \<applicationInitialization>](/iis/configuration/system.webserver/applicationinitialization/)。
* [应用程序池 \<processModel> 的进程模型设置](/iis/configuration/system.applicationhost/applicationpools/add/processmodel)。

## <a name="deployment-resources-for-iis-administrators"></a>面向 IIS 管理员的部署资源

* [IIS 文档](/iis)
* [IIS 中 IIS 管理器入门](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8)
* [.NET Core 应用程序部署](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:index>
* [Microsoft IIS 官方网站](https://www.iis.net/)
* [Windows Server 技术内容库](/windows-server/windows-server)
* [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

若要学习将 ASP.NET Core 应用发布到 IIS 服务器的教程，请参阅<xref:tutorials/publish-to-iis>。

[安装 .NET Core 托管捆绑包](#install-the-net-core-hosting-bundle)

## <a name="supported-operating-systems"></a>支持的操作系统

支持下列操作系统：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

[HTTP.sys 服务器](xref:fundamentals/servers/httpsys)（以前称为 WebListener）无法以反向代理配置形式与 IIS 结合使用。 请使用 [Kestrel 服务器](xref:fundamentals/servers/kestrel)。

有关在 Azure 上托管的信息，请参阅<xref:host-and-deploy/azure-apps/index>。

有关疑难解答指南，请参阅 <xref:test/troubleshoot>。

## <a name="supported-platforms"></a>受支持的平台

支持针对 32 位 (x86) 或 64 位 (x64) 部署发布的应用。 使用 32 位 (x86) .NET Core SDK 部署 32 位应用，除非应用符合以下情况：

* 需要适用于 64 位应用的更大虚拟内存地址空间。
* 需要更大 IIS 堆栈大小。
* 具有 64 位本机依赖项。

使用 64 位 (x64) .NET Core SDK 发布 64 位应用。 主机系统必须具有 64 位运行时。

ASP.NET Core 随附有 [Kestrel 服务器](xref:fundamentals/servers/kestrel)，后者是默认的跨平台 HTTP 服务器。

使用 [IIS](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture) 或 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 时，应用在独立于 IIS 工作进程（进程外）和 [Kestrel 服务器](xref:fundamentals/servers/index#kestrel)的进程中运行。

由于 ASP.NET Core 应用在独立于 IIS 工作进程的进程中运行，因此该模块会处理进程管理。 该模块在第一个请求到达时启动 ASP.NET Core 应用的进程，并在应用关闭或崩溃时重新启动该应用。 这基本上与在 [Windows 进程激活服务 (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) 托管的进程内运行的应用中出现的行为相同。

下图说明了 IIS、ASP.NET Core 模块和进程外托管的应用之间的关系：

![ASP.NET Core 模块](index/_static/ancm-outofprocess.png)

请求从 Web 到达内核模式 HTTP.sys 驱动程序。 驱动程序将请求路由到网站的配置端口上的 IIS，通常为 80 (HTTP) 或 443 (HTTPS)。 该模块将该请求转发到应用的随机端口（非端口 80/443）上的 Kestrel。

该模块在启动时通过环境变量指定端口，[IIS 集成中间件](xref:host-and-deploy/iis/index#enable-the-iisintegration-components)将服务器配置为侦听 `http://localhost:{port}`。 执行其他检查，拒绝不是来自该模块的请求。 该模块不支持 HTTPS 转发，因此即使请求由 IIS 通过 HTTPS 接收，它们还是通过 HTTP 转发。

Kestrel 从模块获取请求后，请求会被推送到 ASP.NET Core 中间件管道中。 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。 IIS 集成添加的中间件会将方案、远程 IP 和 pathbase 更新到帐户以将请求转发到 Kestrel。 应用的响应传递回 IIS，IIS 将响应推送回发起请求的 HTTP 客户端。

`CreateDefaultBuilder` 将 [Kestrel](xref:fundamentals/servers/kestrel) 服务器配置为 Web 服务器，并通过配置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)的基础路径和端口来启用 IIS 集成。

ASP.NET Core 模块生成分配给后端进程的动态端口。 `CreateDefaultBuilder` 调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration%2A> 方法。 `UseIISIntegration` 配置在 localhost IP 地址 (`127.0.0.1`) 中的动态端口上侦听的 Kestrel。 如果动态端口为 1234，则 Kestrel 在 `127.0.0.1:1234` 中侦听。 此配置将替换以下 API 提供的其他 URL 配置：

* `UseUrls`
* [Kestrel 的侦听 API](xref:fundamentals/servers/kestrel#endpoint-configuration)
* [配置](xref:fundamentals/configuration/index)（或 [命令行 -- URL 选项](xref:fundamentals/host/web-host#override-configuration)）

使用模块时，不需要调用 `UseUrls` 或 Kestrel 的 `Listen` API。 如果调用 `UseUrls` 或 `Listen`，则 Kestrel 仅会侦听在没有 IIS 的情况下运行应用时指定的端口。

有关 ASP.NET Core 模块配置指南，请参阅 <xref:host-and-deploy/aspnet-core-module>。

有关托管的详细信息，请参阅[在 ASP.NET Core 中托管](xref:fundamentals/index#host)。

## <a name="application-configuration"></a>应用程序配置

### <a name="enable-the-iisintegration-components"></a>启用 IISIntegration 组件

在 `CreateWebHostBuilder` 中生成主机 ( *Program.cs* )，请调用 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> 以启用 IIS 集成：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        ...
```

有关 `CreateDefaultBuilder` 的详细信息，请参阅 <xref:fundamentals/host/web-host#set-up-a-host>。

### <a name="iis-options"></a>IIS 选项

| 选项                         | 默认 | 设置 |
| ------------------------------ | :-----: | ------- |
| `AutomaticAuthentication`      | `true`  | 若为 `true`，IIS 服务器将设置经过 [Windows 身份验证](xref:security/authentication/windowsauth)进行身份验证的 `HttpContext.User`。 若为 `false`，服务器仅提供 `HttpContext.User` 的标识并在 `AuthenticationScheme` 显式请求时响应质询。 必须在 IIS 中启用 Windows 身份验证使 `AutomaticAuthentication` 得以运行。 有关详细信息，请参阅 [Windows 身份验证](xref:security/authentication/windowsauth)。 |
| `AuthenticationDisplayName`    | `null`  | 设置在登录页上向用户显示的显示名。 |

要配置 IIS 选项，请在 <xref:Microsoft.AspNetCore.Hosting.IStartup.ConfigureServices*> 中包括 <xref:Microsoft.AspNetCore.Builder.IISOptions> 的服务配置。 下面的示例阻止应用填充 `HttpContext.Connection.ClientCertificate`：

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

配置转发头中间件的 [IIS 集成中间件](#enable-the-iisintegration-components)和 ASP.NET Core 模块将配置为转发方案 (HTTP/HTTPS) 和发出请求的远程 IP 地址。 对于托管在其他代理服务器和负载均衡器后方的应用，可能需要附加配置。 有关详细信息，请参阅[配置 ASP.NET Core 以使用代理服务器和负载均衡器](xref:host-and-deploy/proxy-load-balancer)。

### <a name="webconfig-file"></a>web.config 文件

web.config 文件配置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 发布项目时，MSBuild 目标 (`_TransformWebConfig`) 负责创建、转换和发布 web.config 文件。 此目标位于 Web SDK 目标 (`Microsoft.NET.Sdk.Web`) 中。 SDK 设置在项目文件的顶部：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

如果项目中不存在 web.config 文件，则会使用正确的 processPath 和参数创建该文件，以便配置 ASP.NET Core 模块，并将该文件移动到[已发布的输出](xref:host-and-deploy/directory-structure)  。

如果项目中存在 web.config 文件，则会使用正确的 processPath 和参数转换该文件，以便配置 ASP.NET Core 模块，并将该文件移动到已发布的输出。 转换不会修改文件中的 IIS 配置设置。

web.config 文件可能会提供其他 IIS 配置设置，以控制活动的 IIS 模块。 有关能够处理 ASP.NET Core 应用请求的 IIS 模块的信息，请参阅 [IIS 模块](xref:host-and-deploy/iis/modules)主题。

要防止 Web SDK 转换 web.config 文件，请使用项目文件中的 \<IsTransformWebConfigDisabled> 属性：

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

在禁用 Web SDK 转换文件时，开发人员应手动设置 processPath 和参数。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

### <a name="webconfig-file-location"></a>web.config 文件位置

为了正确设置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)，web.config 文件必须位于已部署应用的[内容根](xref:fundamentals/index#content-root)路径（通常为应用基路径）中。 该位置与向 IIS 提供的网站物理路径相同。 若要使用 Web 部署发布多个应用，应用的根路径中需要包含 web.config 文件。

敏感文件存在于应用的物理路径中，如 \<assembly>.runtimeconfig.json、\<assembly>.xml（XML 文档注释）和 \<assembly>.deps.json  。 如果存在 web.config 文件且站点正常启动，则请求获取这些敏感文件时，IIS 不会提供。 如果缺少 web.config 文件、命名不正确，或无法配置站点以正常启动，IIS 可能会公开提供敏感文件。

部署中必须始终存在 web.config 文件且正确命名，并可以配置站点以正常启动。 **请勿从生产部署中删除 web.config 文件。**

### <a name="transform-webconfig"></a>转换 web.config

如果需要在发布时转换 web.config（例如，基于配置、配置文件或环境设置环境变量），请参阅 <xref:host-and-deploy/iis/transform-webconfig>。

## <a name="iis-configuration"></a>IIS 配置

**Windows Server 操作系统**

启用 Web 服务器 (IIS) 服务器角色并建立角色服务。

1. 通过“管理”菜单或“服务器管理器”中的链接使用“添加角色和功能”向导。 在“服务器角色”步骤中，选中“Web 服务器(IIS)”框 。

   ![在选择服务器角色步骤中选择了“Web 服务器 IIS”角色。](index/_static/server-roles-ws2016.png)

1. 在“功能”步骤后，为 Web 服务器 (IIS) 加载“角色服务”步骤。 选择所需 IIS 角色服务，或接受提供的默认角色服务。

   ![在选择角色服务步骤中选择了默认角色服务。](index/_static/role-services-ws2016.png)

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“Web 服务器” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“Web 服务器” > “应用开发”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 继续执行“确认”步骤，安装 Web 服务器角色和服务。 安装 Web 服务器 (IIS) 角色后无需重启服务器/IIS。

**Windows 桌面操作系统**

启用“IIS 管理控制台”和“万维网服务”。

1. 导航到“控制面板”>“程序”>“程序和功能”>“打开或关闭 Windows 功能”（位于屏幕左侧）   。

1. 打开“Internet Information Services”节点。 打开“Web 管理工具”节点。

1. 选中“IIS 管理控制台”框。

1. 选中“万维网服务”框。

1. 接受“万维网服务”的默认功能，或自定义 IIS 功能。

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“万维网服务” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 \<windowsAuthentication>](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“万维网服务” > “应用开发功能”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 如果 IIS 安装需要重新启动，则重新启动系统。

![在“Windows 功能”中选择了“IIS 管理控制台”和“万维网服务”。](index/_static/windows-features-win10.png)

## <a name="install-the-net-core-hosting-bundle"></a>安装 .NET Core 托管捆绑包

在托管系统上安装 .NET Core 托管捆绑包。 捆绑包可安装 .NET Core 运行时、.NET Core 库和 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)。 该模块允许 ASP.NET Core 应用在 IIS 后面运行。

> [!IMPORTANT]
> 如果在 IIS 之前安装了托管捆绑包，则必须修复捆绑包安装。 在安装 IIS 后再次运行托管捆绑包安装程序。
>
> 如果在安装 64 位 (x64) 版本的 .NET Core 之后安装了 Hosting Bundle，则可能看上去缺少 SDK（[未检测到 .NET Core SDK](xref:test/troubleshoot#no-net-core-sdks-were-detected)）。 要解决此问题，请参阅 <xref:test/troubleshoot#missing-sdk-after-installing-the-net-core-hosting-bundle>。

### <a name="download"></a>下载

1. 导航到 [ .NET Core](https://dotnet.microsoft.com/download/dotnet-core) 页面。
1. 选择所需的 .NET Core 版本。
1. 在“运行应用 - 运行时”列中，查找所需的 .NET Core 运行时版本的那一行。
1. 使用“托管捆绑包”链接下载安装程序。

> [!WARNING]
> 某些安装程序包含已到达其生命周期结束 (EOL) 且不再受 Microsoft 支持的发行版本。 有关详细信息，请参阅[支持策略](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)。

### <a name="install-the-hosting-bundle"></a>安装托管捆绑包

1. 在服务器上运行安装程序。 通过管理员命令行界面运行安装程序时，以下参数可用：

   * `OPT_NO_ANCM=1`：跳过安装 ASP.NET Core 模块。
   * `OPT_NO_RUNTIME=1`：跳过安装 .NET Core 运行时。 当服务器仅承载[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) 时使用。
   * `OPT_NO_SHAREDFX=1`：跳过安装 ASP.NET 共享框架（ASP.NET 运行时）。 当服务器仅承载[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd) 时使用。
   * `OPT_NO_X86=1`：跳过安装 x86 运行时。 确定不会托管 32 位应用时，请使用此参数。 如果有可能会同时托管 32 位和 64 位应用，请勿使用此参数，并安装两个运行时。
   * `OPT_NO_SHARED_CONFIG_CHECK=1`：当共享配置 (applicationHost.config) 与 IIS 安装位于同一台计算机上时，禁止对使用的是否是 IIS 共享配置进行检查。 仅适用于 ASP.NET Core 2.2 或更高版本托管捆绑程序安装程序。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration>。
1. 重新启动系统，或在命令行界面中执行以下命令：

   ```console
   net stop was /y
   net start w3svc
   ```
   重启 IIS 会选取安装程序对系统 PATH（环境变量）所作的更改。

安装托管捆绑包时，无需在 IIS 中手动停止各个站点。 IIS 重新启动后，托管应用（IIS 站点）也将重新启动。 应用收到第一个请求（包括来自[应用程序初始化模块](#application-initialization-module-and-idle-timeout)）后，将再次启动。

ASP.NET Core 采用共享框架包的修补程序版本的前滚行为。 当 IIS 托管的应用重新启动 IIS 时，应用会在收到第一个请求后使用其引用的包的最新修补程序版本进行加载。 如果未重新启动 IIS，应用会在其工作进程被回收并收到第一个请求后重新启动并显示前滚行为。

> [!NOTE]
> 有关 IIS 共享配置的信息，请参阅[使用 IIS 共享配置的 ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module#aspnet-core-module-with-an-iis-shared-configuration)。

## <a name="install-web-deploy-when-publishing-with-visual-studio"></a>使用 Visual Studio 进行发布时安装 Web 部署

使用 [Web 部署](/iis/install/installing-publishing-technologies/installing-and-configuring-web-deploy-on-iis-80-or-later)将应用部署到服务器时，请在服务器上安装最新版本的 Web 部署。 要安装 Web 部署，请使用 [Web 平台安装程序 (WebPI)](https://www.microsoft.com/web/downloads/platform.aspx) 或直接从 [Microsoft 下载中心](https://www.microsoft.com/download/details.aspx?id=43717)获取安装程序。 建议使用 WebPI。 WebPI 为托管提供程序提供独立的安装程序和配置。

## <a name="create-the-iis-site"></a>创建 IIS 站点

1. 在托管系统上，创建一个文件夹以包含应用已发布的文件夹和文件。 在接下来的步骤中，文件夹路径作为应用程序的物理路径提供给 IIS。 有关应用程序部署文件夹和文件布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。

1. 在 IIS 管理器中，打开“连接”面板中的服务器节点。 右键单击“站点”文件夹。 选择上下文菜单中的“添加网站”。

1. 提供网站名称，并将物理路径设置为应用的部署文件夹 。 提供“绑定”配置，并通过选择“确定”创建网站：

   ![在“添加网站”步骤中提供网站名称、物理路径和主机名。](index/_static/add-website-ws2016.png)

   > [!WARNING]
   > 不应使用顶级通配符绑定（`http://*:80/` 和 `http://+:80`）。 顶级通配符绑定可能会为应用带来安全漏洞。 此行为同时适用于强通配符和弱通配符。 使用显式主机名而不是通配符。 如果可控制整个父域（区别于易受攻击的 `*.com`），则子域通配符绑定（例如，`*.mysub.com`）不具有此安全风险。 有关详细信息，请参阅 [rfc7230 第 5.4 条](https://tools.ietf.org/html/rfc7230#section-5.4)。

1. 在服务器节点下，选择“应用程序池”。

1. 右键单击站点的应用池，然后从上下文菜单中选择“基本设置”。

1. 在“编辑应用程序池”窗口中，将“.NET CLR 版本”设置为“无托管代码”：

   ![将“.NET CLR 版本”设置为“无托管代码”。](index/_static/edit-apppool-ws2016.png)

    ASP.NET Core 在单独的进程中运行，并管理运行时。 ASP.NET Core 不依赖桌面 CLR (.NET CLR) 加载：将启动 .NET Core 的 Core 公共语言运行时 (CoreCLR) ，在工作进程中托管应用。 将“.NET CLR 版本”设置为“无托管代码”是可选步骤，但建议采用此设置。

1. *ASP.NET Core 2.2 或更高版本* ：对于使用 [进程内托管模型](#in-process-hosting-model)的 64 位 (x64) [独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)，为 32 位 (x86) 进程禁用应用池。

   在 IIS 管理器 >“应用程序池”的“操作”侧栏中，选择“设置应用程序池默认设置”或“高级设置”。 找到“启用 32 位应用程序”并将值设置为 `False`。 此设置不会影响针对[进程外托管](xref:host-and-deploy/aspnet-core-module#out-of-process-hosting-model)部署的应用。

1. 确认进程模型标识拥有适当的权限。

   如果将应用池的默认标识（“进程模型” > “Identity”）从 ApplicationPoolIdentity 更改为另一标识，请确保新标识拥有对应用文件夹、数据库和其他所需资源的必需访问权限  。 例如，应用池需要对文件夹的读取和写入权限，以便应用在其中读取和写入文件。

**Windows 身份验证配置（可选）**  
有关详细信息，请参阅[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

## <a name="deploy-the-app"></a>部署应用

将应用程序部署到 IIS 物理路径文件夹中，该文件夹是在[创建 IIS 站点](#create-the-iis-site)部分中创建的。 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)是建议使用的部署机制，但有几个选项可以将应用程序从项目的发布文件夹移动到托管系统的部署文件夹。

### <a name="web-deploy-with-visual-studio"></a>在 Visual Studio 内使用 Web 部署

要了解如何创建用于 Web 部署的发布配置文件，请参阅[用于 ASP.NET Core 应用部署的 Visual Studio 发布配置文件](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)。 如果托管提供程序提供了发布配置文件或支持创建发布配置文件，请下载配置文件并使用 Visual Studio 的“发布”对话框将其导入：

![“发布”对话框页](index/_static/pub-dialog.png)

### <a name="web-deploy-outside-of-visual-studio"></a>在 Visual Studio 之外使用 Web 部署

也可以在 Visual Studio 之外从命令行使用 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)。 有关详细信息，请参阅 [Web Deployment Tool](/iis/publish/using-web-deploy/use-the-web-deployment-tool)（Web 部署工具）。

### <a name="alternatives-to-web-deploy"></a>Web 部署的替代方法

有多种方法可将应用移动到托管系统，例如手动复制、[Xcopy](/windows-server/administration/windows-commands/xcopy)、[Robocopy](/windows-server/administration/windows-commands/robocopy) 或 [PowerShell](/powershell/)，可使用其中任何一种方法。

有关将 ASP.NET Core 部署到 IIS 的详细信息，请参阅[面向 IIS 管理员的部署资源](#deployment-resources-for-iis-administrators)部分。

## <a name="browse-the-website"></a>浏览网站

将应用部署到托管系统后，向应用的一个公共终结点发出请求。

在以下示例中，站点被绑定到端口 `80` 上 `www.mysite.com` 的 IIS 主机名中。 向 `http://www.mysite.com` 发出请求：

![Microsoft Edge 浏览器已加载 IIS 启动页。](index/_static/browsewebsite.png)

## <a name="locked-deployment-files"></a>锁定的部署文件

如果应用正在运行，部署文件夹中的文件会被锁定。 在部署期间，无法覆盖已锁定的文件。 若要在部署中解除已锁定的文件，请使用以下方法之一停止应用池：

* 使用 Web 部署并在项目文件中引用 `Microsoft.NET.Sdk.Web`。 系统会在 Web 应用目录的根目录中放置一个 app_offline.htm 文件。 该文件存在时，ASP.NET Core 模块会在部署过程中正常关闭该应用并提供 app_offline.htm 文件。 有关详细信息，请参阅 [ASP.NET Core 模块配置参考](xref:host-and-deploy/aspnet-core-module#app_offlinehtm)。
* 在服务器上的 IIS 管理器中手动停止应用池。
* 使用 PowerShell 删除 app_offline.html（需要使用 PowerShell 5 或更高版本）：

  ```powershell
  $pathToApp = 'PATH_TO_APP'

  # Stop the AppPool
  New-Item -Path $pathToApp app_offline.htm

  # Provide script commands here to deploy the app

  # Restart the AppPool
  Remove-Item -Path $pathToApp app_offline.htm

  ```

## <a name="data-protection"></a>数据保护

[ASP.NET Core 数据保护堆栈](xref:security/data-protection/introduction)由多个 ASP.NET Core [中间件](xref:fundamentals/middleware/index)使用，包括用于身份验证的中间件。 即使用户代码不调用数据保护 API，也应该使用部署脚本或在用户代码中配置数据保护，以创建持久的加密[密钥存储](xref:security/data-protection/implementation/key-management)。 如果不配置数据保护，则密钥存储在内存中。重启应用时，密钥会被丢弃。

如果密钥环存储于内存中，则在应用重启时：

* 所有基于 cookie 的身份验证令牌都无效。 
* 用户需要在下一次请求时再次登录。 
* 无法再解密使用密钥环保护的任何数据。 这可能包括 [CSRF 令牌](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)和 [ASP.NET Core MVC TempData cookie](xref:fundamentals/app-state#tempdata)。

若要在 IIS 下配置数据保护以持久化密钥环，请使用以下方法之一：

* **创建数据保护注册表项**

  ASP.NET Core 应用使用的数据保护密钥存储在应用外部的注册表中。 要持久保存给定应用的密钥，需为应用池创建注册表项。

  对于独立的非 Web 场 IIS 安装，可以对用于 ASP.NET Core 应用的每个应用池使用[数据保护 Provision-AutoGenKeys.ps1 PowerShell 脚本](https://github.com/dotnet/AspNetCore/blob/master/src/DataProtection/Provision-AutoGenKeys.ps1)。 此脚本在 HKLM 注册表中创建注册表项，仅应用程序的应用池工作进程帐户可对其进行访问。 通过计算机范围的密钥使用 DPAPI 对密钥静态加密。

  在 web 场方案中，可以将应用配置为使用 UNC 路径存储其数据保护密钥环。 默认情况下，数据保护密钥未加密。 确保网络共享的文件权限仅限于应用在其下运行的 Windows 帐户。 可使用 X509 证书来保护静态密钥。 考虑允许用户上传证书的机制：将证书置于用户信任的证书存储中，并确保这些证书对所有运行用户应用的计算机都可用。 有关详细信息，请参阅[配置 ASP.NET Core 数据保护](xref:security/data-protection/configuration/overview)。

* **配置 IIS 应用程序池以加载用户配置文件**

  此设置位于应用池“高级设置”下的“进程模型”部分 。 将“加载用户配置文件”设置为 `True`。 如果设置为 `True`，会将密钥存储在用户配置文件目录中，并使用 DPAPI 和特定于用户帐户的密钥进行保护。 密钥保存在 %LOCALAPPDATA%/ASP.NET/DataProtection-Keys 文件夹中。

  同时还必须启用应用池的 [setProfileEnvironment attribute](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)。 `setProfileEnvironment` 的默认值为 `true`。 在某些情况下（例如，Windows 操作系统），将 `setProfileEnvironment` 设置为 `false`。 如果密钥未按预期存储在用户配置文件目录中，请执行以下操作：

  1. 导航到 %windir%/system32/inetsrv/config 文件夹。
  1. 打开 applicationHost.config 文件。
  1. 查找 `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 元素。
  1. 确认 `setProfileEnvironment` 属性不存在，这会将值默认设置为 `true`，或者将属性的值显式设置为 `true`。

* **将文件系统用作密钥环存储**

  调整应用代码，[将文件系统用作密钥环存储](xref:security/data-protection/configuration/overview)。 使用 X509 证书保护密钥环，并确保该证书是受信任的证书。 如果它是自签名证书，则将该证书放置于受信任的根存储中。

  当在 Web 场中使用 IIS 时：

  * 使用所有计算机都可以访问的文件共享。
  * 将 X509 证书部署到每台计算机。 [通过代码配置数据保护](xref:security/data-protection/configuration/overview)。

* **设置用于数据保护的计算机范围的策略**

  数据保护系统对以下操作提供有限支持：为使用数据保护 API 的所有应用设置默认[计算机范围的策略](xref:security/data-protection/configuration/machine-wide-policy)。 有关详细信息，请参阅 <xref:security/data-protection/introduction>。

## <a name="virtual-directories"></a>虚拟目录

ASP.NET Core 应用不支持 [IIS 虚拟目录](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#virtual-directories)。 可将应用托管为[子应用程序](#sub-applications)。

## <a name="sub-applications"></a>子应用程序

可将 ASP.NET Core 应用托管为 [IIS 子应用程序（子应用）](/iis/get-started/planning-your-iis-architecture/understanding-sites-applications-and-virtual-directories-on-iis#applications)。 子应用的路径成为根应用 URL 的一部分。

子应用不应将 ASP.NET Core 模块作为处理程序包含在其中。 如果在子应用的 web.config 文件中将该模块添加为处理程序，则在尝试浏览子应用时会收到“500.19 内部服务器错误”，即引用错误的配置文件。

以下示例显示 ASP.NET Core 子应用的已发布 web.config 文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <aspNetCore processPath="dotnet" 
      arguments=".\MyApp.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

在 ASP.NET Core 应用下托管非 ASP.NET Core 子应用时，需在子应用的 web.config 文件中显式删除继承的处理程序：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <remove name="aspNetCore" />
    </handlers>
    <aspNetCore processPath="dotnet" 
      arguments=".\MyApp.dll" 
      stdoutLogEnabled="false" 
      stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

子应用内的静态资产链接应使用波形符-斜杠 (`~/`) 符号。 波形符-斜杠符号触发[标记帮助器](xref:mvc/views/tag-helpers/intro)，来将子应用的基路径追加到呈现的相关链接前面。 对于 `/subapp_path` 处的子应用，使用 `src="~/image.png"` 链接的图像将呈现为 `src="/subapp_path/image.png"`。 根应用的静态文件中间件不处理静态文件请求。 此请求由子应用的静态文件中间件处理。

若将静态资产的 `src` 属性设置为绝对路径（如 `src="/image.png"`），则呈现的链接不包含子应用的基路径。 根应用的静态文件中间件试图从根应用的 [web 根目录](xref:fundamentals/index#web-root)提供资产，这会导致“404 - 找不到”响应生成，除非可以从根应用获得静态资产。

若要将 ASP.NET Core 应用作为子应用托管在其他 ASP.NET Core 应用下：

1. 为此子应用创建应用池。 将“.NET CLR 版本”设置为“无托管代码”，因为将启动 .NET Core 的核心公共语言运行时 (CoreCLR) ，将应用托管在工作进程中，而非桌面 CLR (.NET CLR) 中 。

1. 在 IIS 管理器中添加根网站，并且此子应用在根网站的某个文件夹中。

1. 在 IIS 管理器中右击此子应用文件夹，并选择“转换为应用程序”。

1. 在“添加应用程序”对话框中，使用“应用程序池”的“选择”按钮来分配为子应用创建的应用池。   选择“确定”。

使用进程内托管模型时，需要向子应用分配单独的应用池。

有关进程内托管模型及 ASP.NET Core 模块配置的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

## <a name="configuration-of-iis-with-webconfig"></a>使用 web.config 配置 IIS

IIS 配置受用于 IIS 方案（适用于包含 ASP.NET Core 模块的 ASP.NET Core 应用）的 web.config 的 `<system.webServer>` 部分影响。 例如，IIS 配置适用于动态压缩。 如果在服务器一级将 IIS 配置为使用动态压缩，可通过应用的 web.config 文件中的 `<urlCompression>` 元素，对 ASP.NET Core 应用禁用它。

有关详细信息，请参阅下列主题：

* [\<system.webServer> 的配置参考](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

要为在独立应用池中运行的各应用设置环境变量（IIS 10.0 或更高版本中支持此操作），请参阅 IIS 参考文档中[环境变量 \<environmentVariables>](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) 主题下的“AppCmd.exe 命令”部分。

## <a name="configuration-sections-of-webconfig"></a>web.config 的配置节

ASP.NET Core 应用不会使用 web.config 中的 ASP.NET 4.x 应用的配置部分进行配置：

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

会使用其他的配置提供程序配置 ASP.NET Core 应用。 有关详细信息，请参阅[配置](xref:fundamentals/configuration/index)。

## <a name="application-pools"></a>应用程序池

在服务器上托管多个网站时，建议在每个应用自己的应用池中运行各应用，以彼此隔离。 IIS“添加网站”对话框默认执行此配置。 提供了站点名称时，该文本会自动传输到“应用程序池”文本框 。 添加站点时，会使用该站点名称创建新的应用池。

## <a name="application-pool-no-locidentity"></a>应用程序池 Identity

通过应用池标识帐户，可以在唯一帐户下运行应用，而无需创建和管理域或本地帐户。 在 IIS 8.0 或更高版本上，IIS 管理员工作进程 (WAS) 将使用新应用池的名称创建一个虚拟帐户，并默认在此帐户下运行应用池的工作进程。 在 IIS 管理控制台中，确保应用池“高级设置”下的“Identity”设置为使用 ApplicationPoolIdentity  ：

![应用程序池“高级设置”对话框](index/_static/apppool-identity.png)

IIS 管理进程使用 Windows 安全系统中应用池的名称创建安全标识符。 可使用此标识保护资源。 但是，此标识不是真实的用户帐户，不会在 Windows 用户管理控制台中显示。

如果 IIS 工作进程需要对应用的高级访问权限，请为包含该应用的目录修改访问控制列表 (ACL)：

1. 打开 Windows 资源管理器并导航到目录。

1. 右键单击该目录，然后选择“属性”。

1. 在“安全”选项卡下，选择“编辑”按钮，然后单击“添加”按钮  。

1. 选择“位置”按钮，并确保该系统处于选中状态。

1. 在“输入要选择的对象名称”区域中输入 **IIS AppPool\\<app_pool_name>** 。 选择“检查名称”按钮。 有关 DefaultAppPool，请检查使用 IIS AppPool\DefaultAppPool 的名称。 当选择“检查名称”按钮时，对象名称区域中会显示 DefaultAppPool 的值。 无法直接在对象名称区域中输入应用池名称。 检查对象名称时，请使用 **IIS AppPool\\<app_pool_name>** 格式。

   ![应用文件夹的“选择用户或组”对话框：在选择“检查名称”前，将“DefaultAppPool\"”的应用池名称追加到对象名称区域中的“IIS AppPool”。](index/_static/select-users-or-groups-1.png)

1. 选择“确定”。

   ![应用文件夹的“选择用户或组”对话框：在你选择“检查名称”后，对象名称“DefaultAppPool”显示在对象名称区域中。](index/_static/select-users-or-groups-2.png)

1. 默认情况下应授予读取 &amp; 执行权限。 根据需要请提供其他权限。

也可使用 ICACLS 工具在命令提示符处授予访问权限。 以 DefaultAppPool 为例，使用以下命令：

```console
ICACLS C:\sites\MyWebApp /grant "IIS AppPool\DefaultAppPool":F
```

有关详细信息，请参阅 [icacls](/windows-server/administration/windows-commands/icacls) 主题。

## <a name="http2-support"></a>HTTP/2 支持

满足以下基本要求的进程外部署支持 [HTTP/2](https://httpwg.org/specs/rfc7540.html)：

* Windows Server 2016/Windows 10 或更高版本；IIS 10 或更高版本
* 面向公众的边缘服务器连接使用 HTTP/2，但与 [Kestrel 服务器](xref:fundamentals/servers/kestrel)的反向代理连接使用 HTTP/1.1。
* 目标框架：不适用于进程外部署，因为 HTTP/2 连接完全由 IIS 处理。
* TLS 1.2 或更高版本的连接

如果已建立 HTTP/2 连接，[HttpRequest.Protocol](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 会报告 `HTTP/1.1`。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 有关使用 IIS 部署的 HTTP/2 配置的详细信息，请参阅 [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)。

## <a name="cors-preflight-requests"></a>CORS 预检请求

本部分仅适用于面向 .NET Framework 的 ASP.NET Core 应用程序。

对于面向 .NET Framework 的 ASP.NET Core 应用程序，默认情况下，IIS 不会将 OPTIONS 请求传递给应用程序。 若要了解如何在 web.config 中配置应用程序的 IIS 处理程序以传递 OPTIONS 请求，请参阅[在 ASP.NET Web API 2 中启用跨域请求：CORS 的工作原理](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)。

## <a name="deployment-resources-for-iis-administrators"></a>面向 IIS 管理员的部署资源

* [IIS 文档](/iis)
* [IIS 中 IIS 管理器入门](/iis/get-started/getting-started-with-iis/getting-started-with-the-iis-manager-in-iis-7-and-iis-8)
* [.NET Core 应用程序部署](/dotnet/core/deploying/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/directory-structure>
* <xref:host-and-deploy/iis/modules>
* <xref:test/troubleshoot-azure-iis>
* <xref:host-and-deploy/azure-iis-errors-reference>

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:index>
* [Microsoft IIS 官方网站](https://www.iis.net/)
* [Windows Server 技术内容库](/windows-server/windows-server)
* [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)
* <xref:host-and-deploy/iis/transform-webconfig>

::: moniker-end
