---
title: 高级配置
author: rick-anderson
description: ASP.NET Core 模块和 Internet Information Services (IIS) 的高级配置。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 5/7/2020
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
uid: host-and-deploy/iis/advanced
ms.openlocfilehash: ad2480faeea2f07e51585f5bc6a1c63b3a0b1668
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91755143"
---
# <a name="advanced-configuration-of-the-aspnet-core-module-and-iis"></a>ASP.NET Core 模块和 IIS 的高级配置

本文介绍 ASP.NET Core 模块和 IIS 的高级配置选项和方案。

## <a name="modify-the-stack-size"></a>修改堆栈大小

*仅适用于使用进程内托管模型的情况。*

使用 `web.config` 文件中的 `stackSize` 设置（以字节为单位）配置托管堆栈大小。 默认大小为 1,048,576 字节 (1 MB)。 下面的示例将堆栈大小更改为 2 MB（2,097,152 字节）：

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="stackSize" value="2097152" />
  </handlerSettings>
</aspNetCore>
```

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a>代理配置使用 HTTP 协议和配对令牌

*仅适用于进程外托管。*

在 ASP.NET Core 模块和 Kestrel 之间创建的代理使用 HTTP 协议。 因此，不存在从脱离服务器的位置窃取模块和 Kestrel 之间的流量的风险。

配对令牌用于保证 Kestrel 收到的请求已由 IIS 代理且不来自某些其他源。 模块已创建配对令牌并将其设置到环境变量 (`ASPNETCORE_TOKEN`)。 此外，配对令牌还设置到每个代理请求的标头 (`MS-ASPNETCORE-TOKEN`)。 IIS 中间件检查它所接收的每个请求，以确认配对令牌标头值与环境变量值相匹配。 如果令牌值不匹配，则将记录请求并拒绝该请求。 无法从脱离服务器的位置访问配对令牌环境变量及模块和 Kestrel 之间的流量。 如果不知道配对令牌值，攻击者就无法提交绕过 IIS 中间件中的检查的请求。

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a>具有 IIS 共享配置的 ASP.NET Core 模块

ASP.NET Core 模块安装程序使用 `TrustedInstaller` 帐户的权限运行。 由于本地系统帐户没有 IIS 共享配置所用的共享路径的修改权限，因此在尝试配置共享上的 `applicationHost.config` 文件中的模块设置时，安装程序将引发拒绝访问错误。

在与 IIS 安装相同的计算机上使用 IIS 共享配置时，请运行 ASP.NET Core Hosting Bundle 安装程序，并将 `OPT_NO_SHARED_CONFIG_CHECK` 参数设置为 `1`：

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

如果共享配置的路径与 IIS 安装的路径不在同一台计算机上，请按照下列步骤操作：

1. 禁用 IIS 共享配置。
1. 运行安装程序。
1. 将已更新的 `applicationHost.config` 文件导出到文件共享。
1. 重新启用 IIS 共享配置。

## <a name="data-protection"></a>数据保护

[ASP.NET Core 数据保护堆栈](xref:security/data-protection/introduction)由多个 ASP.NET Core [中间件](xref:fundamentals/middleware/index)使用，包括用于身份验证的中间件。 即使用户代码不调用数据保护 API，也应该使用部署脚本或在用户代码中配置数据保护，以创建持久的加密[密钥存储](xref:security/data-protection/implementation/key-management)。 如果不配置数据保护，则密钥存储在内存中。重启应用时，密钥会被丢弃。

如果应用重启时数据保护密钥环存储于内存中：

* 所有基于 cookie 的身份验证令牌都无效。 
* 用户需要在下一次请求时再次登录。 
* 无法再解密使用密钥环保护的任何数据。 这可能包括 [CSRF 令牌](xref:security/anti-request-forgery#aspnet-core-antiforgery-configuration)和 [ASP.NET Core MVC TempData cookie](xref:fundamentals/app-state#tempdata)。

若要在 IIS 下配置数据保护以持久化密钥环，请使用以下方法之一：

* **创建数据保护注册表项**

  ASP.NET Core 应用使用的数据保护密钥存储在应用外部的注册表中。 要持久保存给定应用的密钥，需为应用池创建注册表项。

  对于独立的非 Web 场 IIS 安装，可以对用于 ASP.NET Core 应用的每个应用池使用[数据保护 Provision-AutoGenKeys.ps1 PowerShell 脚本](https://github.com/dotnet/AspNetCore/blob/master/src/DataProtection/Provision-AutoGenKeys.ps1)。 此脚本在 HKLM 注册表中创建注册表项，仅应用的应用池工作进程帐户可对其进行访问。 通过计算机范围的密钥使用 DPAPI 对密钥静态加密。

  在 Web 场方案中，可以将应用配置为使用 UNC 路径存储其数据保护密钥环。 默认情况下，密钥未加密。 确保网络共享的文件权限仅限于应用在其下运行的 Windows 帐户。 可使用 X509 证书来保护静态密钥。 考虑允许用户上传证书的机制。 将证书置于用户信任的证书存储中，并确保这些证书对所有运行用户应用的计算机都可用。 有关详细信息，请参阅 <xref:security/data-protection/configuration/overview>。

* **配置 IIS 应用程序池以加载用户配置文件**

  此设置位于应用池“高级设置”下的“进程模型”部分 。 将“加载用户配置文件”设置为 `True`。 如果设置为 `True`，会将密钥存储在用户配置文件目录中，并使用 DPAPI 和特定于用户帐户的密钥进行保护。 密钥保留在 `%LOCALAPPDATA%/ASP.NET/DataProtection-Keys` 文件夹中。

  同时还必须启用应用池的 [`setProfileEnvironment` 属性](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)。 `setProfileEnvironment` 的默认值为 `true`。 在某些情况下（例如，Windows 操作系统），将 `setProfileEnvironment` 设置为 `false`。 如果密钥未按预期存储在用户配置文件目录中，请执行以下操作：

  1. 导航到 `%windir%/system32/inetsrv/config` 文件夹。
  1. 打开 `applicationHost.config` 文件。
  1. 查找 `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 元素。
  1. 确认 `setProfileEnvironment` 属性不存在，这会将值默认设置为 `true`，或者将属性的值显式设置为 `true`。

* **将文件系统用作密钥环存储**

  调整应用代码，[将文件系统用作密钥环存储](xref:security/data-protection/configuration/overview)。 使用 X509 证书保护密钥环，并确保该证书是受信任的证书。 如果它是自签名证书，则将该证书放置于受信任的根存储中。

  当在 Web 场中使用 IIS 时：

  * 使用所有计算机都可以访问的文件共享。
  * 将 X509 证书部署到每台计算机。 [通过代码配置数据保护](xref:security/data-protection/configuration/overview)。

* **设置用于数据保护的计算机范围的策略**

  数据保护系统对以下操作提供有限支持：为使用数据保护 API 的所有应用设置默认[计算机范围的策略](xref:security/data-protection/configuration/machine-wide-policy)。 有关详细信息，请参阅 <xref:security/data-protection/introduction>。

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

1. 导航到“控制面板” > “程序” > “程序和功能” > “启用或禁用 Windows 功能”（位于屏幕左侧）   。

1. 打开“Internet Information Services”节点。 打开“Web 管理工具”节点。

1. 选中“IIS 管理控制台”框。

1. 选中“万维网服务”框。

1. 接受“万维网服务”的默认功能，或自定义 IIS 功能。

   **Windows 身份验证（可选）**  
   若要启用 Windows 身份验证，请依次展开以下节点：“万维网服务” > “安全”。 选择“Windows 身份验证”功能。 有关详细信息，请参阅 [Windows 身份验证 `<windowsAuthentication>`](/iis/configuration/system.webServer/security/authentication/windowsAuthentication/) 和[配置 Windows 身份验证](xref:security/authentication/windowsauth)。

   **Websocket（可选）**  
   Websocket 支持 ASP.NET Core 1.1 或更高版本。 若要启用 WebSocket，请依次展开以下节点：“万维网服务” > “应用开发功能”。 选择“WebSocket 协议”功能。 有关详细信息，请参阅 [WebSockets](xref:fundamentals/websockets)。

1. 如果 IIS 安装需要重新启动，则重新启动系统。

![在“Windows 功能”中选择了“IIS 管理控制台”和“万维网服务”。](index/_static/windows-features-win10.png)

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

1. 在“添加应用程序”对话框中，使用“应用程序池”的“选择”按钮来分配为子应用创建的应用池。   选择“确定”  。

使用进程内托管模型时，需要向子应用分配单独的应用池。

有关进程内托管模型及 ASP.NET Core 模块配置的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

## <a name="application-pools"></a>应用程序池

由托管模型决定应用池隔离：

* 托管在进程内：应用需要在单独的应用池中运行。
* 托管在进程外：建议在每个应用自己的应用池中运行各应用，以彼此隔离。

IIS“添加网站”对话框默认为每应用一个应用池。 提供了站点名称时，该文本会自动传输到“应用程序池”文本框 。 添加站点时，会使用该站点名称创建新的应用池。

## <a name="application-pool-no-locidentity"></a>应用程序池 Identity

通过应用池标识帐户，可以在唯一帐户下运行应用，而无需创建和管理域或本地帐户。 在 IIS 8.0 或更高版本上，IIS 管理员工作进程 (WAS) 将使用新应用池的名称创建一个虚拟帐户，并默认在此帐户下运行应用池的工作进程。 在 IIS 管理控制台中，确保应用池“高级设置”下的“Identity”设置为使用 `ApplicationPoolIdentity` ：

![应用程序池“高级设置”对话框](index/_static/apppool-identity.png)

IIS 管理进程使用 Windows 安全系统中应用池的名称创建安全标识符。 可使用此标识保护资源。 但是，此标识不是真实的用户帐户，不会在 Windows 用户管理控制台中显示。

如果 IIS 工作进程需要对应用的高级访问权限，请为包含该应用的目录修改访问控制列表 (ACL)：

1. 打开 Windows 资源管理器并导航到目录。

1. 右键单击该目录，然后选择“属性”。

1. 在“安全”选项卡下，选择“编辑”按钮，然后单击“添加”按钮  。

1. 选择“位置”按钮，并确保该系统处于选中状态。

1. 在“输入要选择的对象名”区域中，输入 `IIS AppPool\{APP POOL NAME}` 格式，其中 `{APP POOL NAME}` 占位符为应用池名称。 选择“检查名称”按钮。 对于 DefaultAppPool，请使用 `IIS AppPool\DefaultAppPool` 检查名称。 如果选择了“检查名称”按钮，对象名称区域中将指示 `DefaultAppPool`。 无法直接在对象名称区域中输入应用池名称。 检查对象名称时使用 `IIS AppPool\{APP POOL NAME}` 格式，其中 `{APP POOL NAME}` 占位符是应用池名称。

   ![应用文件夹的“选择用户或组”对话框：在选择“检查名称”前，将“DefaultAppPool\"”的应用池名称追加到对象名称区域中的“IIS AppPool”。](index/_static/select-users-or-groups-1.png)

1. 选择“确定”  。

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

对于建立了 HTTP/2 连接时的进程内部署，[`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 报告 `HTTP/2`。 对于建立了 HTTP/2 连接时的进程外部署，[`HttpRequest.Protocol`](xref:Microsoft.AspNetCore.Http.HttpRequest.Protocol*) 报告 `HTTP/1.1`。

若要详细了解进程内和进程外托管模型，请参阅 <xref:host-and-deploy/aspnet-core-module>。

默认情况下将启用 HTTP/2。 如果未建立 HTTP/2 连接，连接会回退到 HTTP/1.1。 有关使用 IIS 部署的 HTTP/2 配置的详细信息，请参阅 [IIS 上的 HTTP/2](/iis/get-started/whats-new-in-iis-10/http2-on-iis)。

## <a name="cors-preflight-requests"></a>CORS 预检请求

本部分仅适用于面向 .NET Framework 的 ASP.NET Core 应用程序。

对于面向 .NET Framework 的 ASP.NET Core 应用程序，默认情况下，IIS 不会将 OPTIONS 请求传递给应用程序。 若要了解如何在 `web.config` 中配置应用的 IIS 处理程序以传递 OPTIONS 请求，请参阅[在 ASP.NET Web API 2 中启用跨域请求：CORS 的工作原理](/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api#how-cors-works)。

## <a name="application-initialization-module-and-idle-timeout"></a>应用程序初始化模块和空闲超时

通过 ASP.NET Core 模板版本 2 托管在 IIS 中时：

* [应用程序初始化模块](#application-initialization-module)：可以将托管在[进程内](xref:host-and-deploy/iis/in-process-hosting)或[进程外](xref:host-and-deploy/iis/out-of-process-hosting)的应用配置为在工作进程重启或服务器重启后自动启动。
* [空闲超时](#idle-timeout)：可以将托管在[进程内](xref:host-and-deploy/iis/in-process-hosting)的应用配置为在非活动时段不超时。

### <a name="application-initialization-module"></a>应用程序初始化模块

适用于托管在进程内和进程外的应用。

[IIS 应用程序初始化](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)是一种 IIS 功能，可在应用池启动或回收时向应用发送 HTTP 请求。 该请求会触发应用启动。 默认情况下，IIS 向应用的根 URL (`/`) 发出请求以初始化应用（有关配置的详细信息，请参阅[更多资源](#application-initialization-module-and-idle-timeout-additional-resources)）。

确认已启用 IIS 应用程序初始化角色功能：

Windows 7 或更高版本桌面系统，在本地使用 IIS 时：

1. 导航到“控制面板” > “程序” > “程序和功能” > “启用或禁用 Windows 功能”（位于屏幕左侧）   。
1. 打开“Internet Information Services” > “万维网服务” > “应用程序开发功能”。
1. 选中“应用程序初始化”的复选框。

Windows Server 2008 R2 或更高版本：

1. 打开“添加角色和功能向导”。
1. 在“选择角色服务”面板中，打开“应用程序开发”节点 。
1. 选中“应用程序初始化”的复选框。

使用下面的任一方法为站点启用“应用程序初始化模块”：

* 使用 IIS 管理器：

  1. 在“连接”面板中选择“应用程序池”。
  1. 在列表中右键单击应用的应用池，并选择“高级设置”。
  1. 默认的“启动模式”为“`OnDemand`”。 将“启动模式”设置为“`AlwaysRunning`”。 选择“确定”  。
  1. 打开“连接”面板中的“网站”节点。
  1. 右键单击应用，并选择“管理网站” > “高级设置” 。
  1. 默认的“预加载已启用”设置为“`False`”。 将“预加载已启用”设置为“`True`”。 选择“确定”  。

* 使用 `web.config` 向应用的 `web.config` 文件中的 `<system.webServer>` 元素添加 `<applicationInitialization>` 元素（其中 `doAppInitAfterRestart` 设置为 `true`）：

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
1. 默认的“空闲超时(分钟)”为“`20`”分钟。 将“空闲超时(分钟)”设置为“`0`”（零）。 选择“确定”  。
1. 回收工作进程。

若要防止托管在[进程外](xref:host-and-deploy/iis/out-of-process-hosting)的应用超时，请使用下列任一方法：

* 从外部服务 ping 应用，使其保持运行状态。
* 如果应用仅托管后台服务，请避免使用 IIS 托管，而是[使用 Windows 服务托管 ASP.NET Core 应用](xref:host-and-deploy/windows-service)。

### <a name="application-initialization-module-and-idle-timeout-additional-resources"></a>关于应用程序初始化模块和空闲超时的更多资源

* [IIS 8.0 应用程序初始化](/iis/get-started/whats-new-in-iis-8/iis-80-application-initialization)
* [应用程序初始化 `<applicationInitialization>`](/iis/configuration/system.webserver/applicationinitialization/)。
* [应用程序池 `<processModel>` 的进程模型设置](/iis/configuration/system.applicationhost/applicationpools/add/processmodel)。

## <a name="module-schema-and-configuration-file-locations"></a>模块、架构和配置文件位置

### <a name="module"></a>模块

**IIS (x86/amd64)** ：

* `%windir%\System32\inetsrv\aspnetcore.dll`

* `%windir%\SysWOW64\inetsrv\aspnetcore.dll`

* `%ProgramFiles%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll`

**IIS Express (x86/amd64)** ：

* `%ProgramFiles%\IIS Express\aspnetcore.dll`

* `%ProgramFiles(x86)%\IIS Express\aspnetcore.dll`

* `%ProgramFiles%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

* `%ProgramFiles(x86)%\IIS Express\Asp.Net Core Module\V2\aspnetcorev2.dll`

### <a name="schema"></a>架构

**IIS**

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml`

* `%windir%\System32\inetsrv\config\schema\aspnetcore_schema_v2.xml`

**IIS Express**

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml`

* `%ProgramFiles%\IIS Express\config\schema\aspnetcore_schema_v2.xml`

### <a name="configuration"></a>Configuration

**IIS**

* `%windir%\System32\inetsrv\config\applicationHost.config`

**IIS Express**

* Visual Studio：`{APPLICATION ROOT}\.vs\config\applicationHost.config`

* iisexpress.exe CLI：`%USERPROFILE%\Documents\IISExpress\config\applicationhost.config`

通过在 `applicationHost.config` 文件中搜索 `aspnetcore` 可以找到这些文件。
