---
title: Azure 应用服务和 IIS 上 ASP.NET Core 的常见错误参考
author: rick-anderson
description: 获取在 Azure 应用服务和 IIS 上托管 ASP.NET Core 应用的常见错误的故障排除建议。
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
uid: host-and-deploy/azure-iis-errors-reference
ms.openlocfilehash: 46fa3138e3fbcc4ed5e4497ec017a5873a4cdb13
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91754653"
---
# <a name="common-errors-reference-for-azure-app-service-and-iis-with-aspnet-core"></a>Azure 应用服务和 IIS 上 ASP.NET Core 的常见错误参考

::: moniker range=">= aspnetcore-2.2"

本主题描述在 Azure 应用服务和 IIS 上托管 ASP.NET Core 应用时的常见错误，并提供特定错误的故障排除建议。

有关一般故障排除指南，请参阅 <xref:test/troubleshoot-azure-iis>。

收集以下信息：

* 浏览器行为（状态代码和错误消息）
* 应用程序事件日志条目
  * Azure 应用服务：请参阅 <xref:test/troubleshoot-azure-iis>。
  * IIS
    1. 选择 Windows 菜单中的“开始”，键入“事件查看器”，然后按 Enter 。
    1. 打开事件查看器后，展开侧栏中的“Windows 日志”>“应用程序”  。
* ASP.NET Core 模块 stdout 和 debug日志条目
  * Azure 应用服务：请参阅 <xref:test/troubleshoot-azure-iis>。
  * IIS：按照 ASP.NET Core 模块主题的[日志创建和重定向](xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection)和[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)部分中的说明进行操作。

将错误信息与以下常见错误进行比较。 如果找到匹配项，请按照故障排除建议进行操作。

本主题中的错误列表并未包含所有错误。 如果遇到此处未列出的错误，请使用本主题底部的“内容反馈”按钮打开一个新问题，并提供有关如何重现错误的详细说明。

[!INCLUDE[Azure App Service Preview Notice](../includes/azure-apps-preview-notice.md)]

## <a name="os-upgrade-removed-the-32-bit-aspnet-core-module"></a>OS 升级删除了 32 位 ASP.NET Core 模块

**应用程序日志：** 未能加载模块 DLL C:\WINDOWS\system32\inetsrv\aspnetcore.dll。 该数据是个错误。

疑难解答：

OS 升级期间不会保留 C:\Windows\SysWOW64\inetsrv 目录中的非 OS 文件。 如果在 OS 升级前已安装 ASP.NET Core 模块，且随后在 OS 升级后在 32 位模式下运行任何应用池，则会遇到此问题。 在 OS 升级后修复 ASP.NET Core 模块。 请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。 运行安装程序时，选择“修复”。

## <a name="missing-site-extension-32-bit-x86-and-64-bit-x64-site-extensions-installed-or-wrong-process-bitness-set"></a>缺少站点扩展、安装了 32 位 (x86) 和 64 位 (x64) 站点扩展，或设置了错误的进程位数

适用于 Azure 应用服务托管的应用。

* **浏览器：** HTTP 错误 500.0 - ANCM 进程内处理程序加载失败

* **应用程序日志：** 调用 hostfxr 以查找进程内请求处理程序失败，未找到任何本机依赖项。 找不到进程内请求处理程序。 调用 hostfxr 捕获的输出：无法找到任何兼容的框架版本。 找不到指定的框架“Microsoft.AspNetCore.App”、版本“{VERSION}-preview-\*”。 未能启动应用程序“/LM/W3SVC/1416782824/ROOT”，ErrorCode“0x8000ffff”。

* **ASP.NET Core 模块 stdout 日志：** 无法找到任何兼容的框架版本。 找不到指定的框架“Microsoft.AspNetCore.App”、版本“{VERSION}-preview-\*”。

* **ASP.NET Core 模块调试日志：** 调用 hostfxr 以查找进程内请求处理程序失败，未找到任何本机依赖项。 这很可能意味着应用配置错误，请检查 Microsoft.NetCore.App 和 Microsoft.AspNetCore.App 版本是否针对该应用程序并安装在计算机上。 返回失败的 HRESULT：0x8000ffff。 找不到进程内请求处理程序。 无法找到任何兼容的框架版本。 找不到指定的框架“Microsoft.AspNetCore.App”、版本“{VERSION}-preview-\*”。

疑难解答：

* 如果在预览运行时运行该应用，请安装 32 位 (x86) 或  64位 (x64) 站点扩展，以匹配应用的位数和应用的运行时版本。 请勿同时安装扩展或扩展的多个运行时版本。

  * ASP.NET Core {RUNTIME VERSION} (x86) 运行时
  * ASP.NET Core {RUNTIME VERSION} (x64) 运行时

  重新启动应用。 等待几秒钟，以便应用重新启动。

* 如果在预览运行时运行应用，且同时安装了 32 位 (x86) 和 64 位 (x64) [站点扩展](xref:host-and-deploy/azure-apps/index#install-the-preview-site-extension)，请卸载与应用的位数不匹配的站点扩展。 删除站点扩展之后，重新启动应用。 等待几秒钟，以便应用重新启动。

* 如果在预览运行时运行该应用，并且站点扩展的位数匹配应用的位数，请确认预览站点扩展的运行时版本匹配应用的运行时版本。

* 确认“应用程序设置”中的应用“平台”与应用的位数匹配。

有关详细信息，请参阅 <xref:host-and-deploy/azure-apps/index#install-the-preview-site-extension>。

## <a name="an-x86-app-is-deployed-but-the-app-pool-isnt-enabled-for-32-bit-apps"></a>已部署 x86 应用，但 32 位应用未启用应用池

* **浏览器：** HTTP 错误 500.30 - ANCM 进程内启动失败

* **应用程序日志：** 物理根路径为 '{PATH}' 的应用程序 '/LM/W3SVC/5/ROOT' 遇到意外的托管异常，异常代码= '0xe0434352'。 请检查 stderr 日志以获取详细信息。 物理根路径为 '{PATH}' 的应用程序 '/LM/W3SVC/5/ROOT' 未能加载 clr 和托管应用程序。 CLR 工作线程提前退出

* **ASP.NET Core 模块 stdout 日志：** 日志文件已创建，但为空。

* **ASP.NET Core 模块调试日志：** 返回失败的 HRESULT：0x8007023e

发布自包含应用时，SDK 会捕获此方案。 如果 RID 与平台目标（例如，`win10-x64` RID 与项目文件中的 `<PlatformTarget>x86</PlatformTarget>`）不匹配，则 SDK 会产生错误。

疑难解答：

对于依赖 x86 框架的部署 (`<PlatformTarget>x86</PlatformTarget>`)，请为 32 位应用启用 IIS 应用池。 在 IIS 管理器中，打开应用池的“高级设置”并将“启用 32 位应用程序”设置为 True  。

## <a name="platform-conflicts-with-rid"></a>平台与 RID 冲突

* **浏览器：** HTTP 错误 502.5 - 进程失败

* **应用程序日志：** 物理根路径为 'C:\{PATH}\' 的应用程序 'MACHINE/WEBROOT/APPHOST/{ASSEMBLY}' 未能通过 '"C:\{PATH}{ASSEMBLY}.{exe|dll}" ' 命令行启动进程，ErrorCode = '0x80004005 : ff。

* **ASP.NET Core 模块 stdout 日志：** 未经处理的异常：System.BadImageFormatException：无法加载文件或程序集 '{ASSEMBLY}.dll'。 试图加载的程序的格式不正确。

疑难解答：

* 确认应用在 Kestrel 上本地运行。 进程失败可能是由应用的内部问题导致的。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

* 如果在升级应用和部署更新的程序集时，Azure 应用部署出现此异常，请从先前的部署中手动删除所有文件。 部署升级的应用时，延迟的不兼容程序集可能造成 `System.BadImageFormatException` 异常。

## <a name="uri-endpoint-wrong-or-stopped-website"></a>URI 终结点错误或网站已停止

* **浏览器：** ERR_CONNECTION_REFUSED --或者-- 无法连接

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

* **ASP.NET Core 模块调试日志：** 未创建日志文件。

疑难解答：

* 确认正在使用的应用的 URI 端点是否正确。 检查绑定。

* 确认 IIS 网站未处于“已停止”状态。

## <a name="corewebengine-or-w3svc-server-features-disabled"></a>已禁用 CoreWebEngine 或 W3SVC 服务器功能

**OS 异常：** 必须安装 IIS 7.0 CoreWebEngine 和 W3SVC 功能才能使用 ASP.NET Core 模块。

疑难解答：

确认启用了适当的角色和功能。 请参阅 [IIS 配置](xref:host-and-deploy/iis/index#iis-configuration)。

## <a name="incorrect-website-physical-path-or-app-missing"></a>网站物理路径不正确或缺少应用

* **浏览器：** 403 禁止访问 - 访问被拒绝 --或者-- 403.14 禁止访问 - Web 服务器被配置为不列出此目录的内容。

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

* **ASP.NET Core 模块调试日志：** 未创建日志文件。

疑难解答：

检查 IIS 网站的“基本设置”和物理应用文件夹。 确认应用所在的文件夹位于 IIS 网站的物理路径中。

## <a name="incorrect-role-aspnet-core-module-not-installed-or-incorrect-permissions"></a>角色不正确、未安装 ASP.NET Core 模块或权限不正确

* **浏览器：** 500.19 内部服务器错误 - 无法访问请求的页面，因为该页面的相关配置数据无效。 --或者-- 无法显示此页面

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

* **ASP.NET Core 模块调试日志：** 未创建日志文件。

疑难解答：

* 确认启用了适当的角色。 请参阅 [IIS 配置](xref:host-and-deploy/iis/index#iis-configuration)。

* 打开“程序和功能”或“应用和功能”，然后确认是否安装了 Windows Server Hosting  。 如果已安装的程序列表中没有 Windows Server Hosting，请下载并安装 .NET Core 托管捆绑包。

  [当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

  有关详细信息，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

* 请确保将“应用程序池”>“进程模型”>“Identity”设置为“ApplicationPoolIdentity”，或确保自定义标识具有访问应用部署文件夹的相应权限   。

* 如果卸载了 ASP.NET Core 托管捆绑包并已安装了一个早期版本的托管捆绑包，则 applicationHost.config 文件将不包含 ASP.NET Core 模块分区。 可打开 %windir%/System32/inetsrv/config 处的 applicationHost.config 文件并找到 `<configuration><configSections><sectionGroup name="system.webServer">` 分区组 。 如果分区组中缺少 ASP.NET Core 模块分区，请添加以下分区元素：

  ```xml
  <section name="aspNetCore" overrideModeDefault="Allow" />
  ```

  或者，安装最新版本的 ASP.NET Core 托管捆绑包。 最新版本与受支持的 ASP.NET Core 应用向后兼容。

## <a name="incorrect-processpath-missing-path-variable-hosting-bundle-not-installed-systemiis-not-restarted-vc-redistributable-not-installed-or-dotnetexe-access-violation"></a>processPath 不正确、缺少 PATH 变量、未安装托管捆绑包、未重启系统/IIS、未安装 VC++ Redistributable 或 dotnet.exe 访问冲突

* **浏览器：** HTTP 错误 500.0 - ANCM 进程内处理程序加载失败

* **应用程序日志：** 物理根路径为 'C:\{PATH}\' 的应用程序 'MACHINE/WEBROOT/APPHOST/{ASSEMBLY}' 未能通过 '"{...}" ' 命令行启动进程 ，ErrorCode = '0x80070002 :0。 应用程序 '{PATH}' 无法启动。 '{PATH}' 中未找到可执行文件。 未能启动应用程序 '/LM/W3SVC/2/ROOT'，ErrorCode '0x8007023e'。

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

* **ASP.NET Core 模块调试日志：** 事件日志：应用程序 '{PATH}' 无法启动。 '{PATH}' 中未找到可执行文件。 返回失败的 HRESULT：0x8007023e

疑难解答：

* 确认应用在 Kestrel 上本地运行。 进程失败可能是由应用的内部问题导致的。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

* 检查 web.config 中 `<aspNetCore>` 元素的 processPath 属性，对于依赖框架的部署 (FDD)，确保它为 `dotnet`，对于[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd)，确保它为 `.\{ASSEMBLY}.exe` 。

* 对于 FDD，可能无法通过路径设置访问 dotnet.exe。 确认“系统路径”设置中存在“C:\Program Files\dotnet”\\。

* 对于 FDD，应用池的用户标识可能无法访问 dotnet.exe。 确认应用池用户标识具有访问 C:\Program Files\dotnet 目录的权限。 确认没有为应用池用户标识配置针对 C:\Program Files\dotnet 和应用目录的拒绝规则。

* 可能已部署 FDD 并安装了 .NET Core，但未重启 IIS。 重启服务器，或通过从命令提示符依次执行 net stop was /y 和 net start w3svc 来重启 IIS 。

* 可能已部署 FDD，但未在托管系统上安装 .NET Core 运行时。 如果尚未安装 .NET Core 运行时，则运行系统上的 **.NET Core 托管捆绑包安装程序**。

  [当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

  有关详细信息，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

  如果需要特定的运行时，请从 [.NET 下载存档](https://dotnet.microsoft.com/download/archives)下载运行时并将其安装在系统上。 重启系统，或通过从命令提示符依次执行 net stop was /y 和 net start w3svc 来重启 IIS，完成安装 。

## <a name="incorrect-arguments-of-aspnetcore-element"></a>\<aspNetCore> 元素的参数不正确

* **浏览器：** HTTP 错误 500.0 - ANCM 进程内处理程序加载失败

* **应用程序日志：** 调用 hostfxr 以查找进程内请求处理程序失败，未找到任何本机依赖项。 这很可能意味着应用配置错误，请检查 Microsoft.NetCore.App 和 Microsoft.AspNetCore.App 版本是否针对该应用程序并安装在计算机上。 找不到进程内请求处理程序。 调用 hostfxr 捕获的输出：是否希望运行 dotnet SDK 命令？ 请从以下位置安装 dotnet SDK： https://go.microsoft.com/fwlink/?LinkID=798306&clcid=0x409 未能启动应用程序 '/LM/W3SVC/3/ROOT'，ErrorCode '0x8000ffff'。

* **ASP.NET Core 模块 stdout 日志：** 是否希望运行 dotnet SDK 命令？ 请从以下位置安装 dotnet SDK： https://go.microsoft.com/fwlink/?LinkID=798306&clcid=0x409

* **ASP.NET Core 模块调试日志：** 调用 hostfxr 以查找进程内请求处理程序失败，未找到任何本机依赖项。 这很可能意味着应用配置错误，请检查 Microsoft.NetCore.App 和 Microsoft.AspNetCore.App 版本是否针对该应用程序并安装在计算机上。 返回失败的 HRESULT：0x8000ffff 找不到进程内请求处理程序。 调用 hostfxr 捕获的输出：是否希望运行 dotnet SDK 命令？ 请从以下位置安装 dotnet SDK： https://go.microsoft.com/fwlink/?LinkID=798306&clcid=0x409 返回失败的 HRESULT：0x8000ffff

疑难解答：

* 确认应用在 Kestrel 上本地运行。 进程失败可能是由应用的内部问题导致的。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

* 检查 web.config 中 `<aspNetCore>` 元素的 arguments 属性，确认该属性：(a) 对于依赖框架的部署 (FDD) 为 `.\{ASSEMBLY}.dll`；或 (b) 对于独立部署 (SCD)，则为不存在、为空字符串 (`arguments=""`)，或为应用参数列表 (`arguments="{ARGUMENT_1}, {ARGUMENT_2}, ... {ARGUMENT_X}"`) 。

## <a name="missing-net-core-shared-framework"></a>缺少 .NET Core 共享框架

* **浏览器：** HTTP 错误 500.0 - ANCM 进程内处理程序加载失败

* **应用程序日志：** 调用 hostfxr 以查找进程内请求处理程序失败，未找到任何本机依赖项。 这很可能意味着应用配置错误，请检查 Microsoft.NetCore.App 和 Microsoft.AspNetCore.App 版本是否针对该应用程序并安装在计算机上。 找不到进程内请求处理程序。 调用 hostfxr 捕获的输出：无法找到任何兼容的框架版本。 找不到指定的框架 'Microsoft.AspNetCore.App'、版本 '{VERSION}'。

未能启动应用程序 '/LM/W3SVC/5/ROOT'，ErrorCode '0x8000ffff'。

* **ASP.NET Core 模块 stdout 日志：** 无法找到任何兼容的框架版本。 找不到指定的框架 'Microsoft.AspNetCore.App'、版本 '{VERSION}'。

* **ASP.NET Core 模块调试日志：** 返回失败的 HRESULT：0x8000ffff

疑难解答：

对于依赖框架的部署 (FDD)，确认已在系统上安装正确的运行时。

## <a name="stopped-application-pool"></a>应用程序池已停止

* **浏览器：** 503 服务不可用

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

* **ASP.NET Core 模块调试日志：** 未创建日志文件。

疑难解答：

确认应用程序池未处于“已停止”状态。

## <a name="sub-application-includes-a-handlers-section"></a>子应用程序包括 \<handlers> 部分

* **浏览器：** HTTP 错误 500.19 - 内部服务器错误

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 创建根应用的日志文件并显示正常操作。 未创建子应用的日志文件。

* **ASP.NET Core 模块调试日志：** 创建根应用的日志文件并显示正常操作。 未创建子应用的日志文件。

疑难解答：

确认子应用的 web.config 文件不包含 `<handlers>` 部分，或者子应用未继承父级应用的处理程序。

父级应用的 web.config 的`<system.webServer>` 放在 `<location>` 元素内。 将 <xref:System.Configuration.SectionInformation.InheritInChildApplications*> 属性设置为 `false`，表示 [\<location>](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) 元素中指定的设置不会由驻留在父级应用子目录中的应用继承。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

## <a name="stdout-log-path-incorrect"></a>stdout 日志路径不正确

* **浏览器：** 应用正常响应。

* **应用程序日志：** 无法在 C:\Program Files\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll 中启动 stdout 重定向。 异常消息：在 {PATH}\aspnetcoremodulev2\commonlib\fileoutputmanager.cpp:84 返回了 HRESULT 0x80070005。 无法在 C:\Program Files\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll 中停止 stdout 重定向。 异常消息：在 {PATH} 返回了 HRESULT 0x80070002。 无法在 {PATH}\aspnetcorev2_inprocess.dll 中启动 stdout 重定向。

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

* **ASP.NET Core 模块调试日志：** 无法在 C:\Program Files\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll 中启动 stdout 重定向。 异常消息：在 {PATH}\aspnetcoremodulev2\commonlib\fileoutputmanager.cpp:84 返回了 HRESULT 0x80070005。 无法在 C:\Program Files\IIS\Asp.Net Core Module\V2\aspnetcorev2.dll 中停止 stdout 重定向。 异常消息：在 {PATH} 返回了 HRESULT 0x80070002。 无法在 {PATH}\aspnetcorev2_inprocess.dll 中启动 stdout 重定向。

疑难解答：

* web.config 的 `<aspNetCore>` 元素中指定的 `stdoutLogFile` 路径不存在。 有关详细信息，请参阅 [ASP.NET Core 模块：日志创建和重定向](xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection)。

* 应用池用户没有 stdout 日志路径的写入权限。

## <a name="application-configuration-general-issue"></a>应用程序配置常见问题

* **浏览器：** HTTP 错误 500.0 - ANCM 进程内处理程序加载失败 --或者-- HTTP 错误 500.30 - ANCM 进程内启动失败

* **应用程序日志：** 变量

* **ASP.NET Core 模块 stdout 日志：** 日志文件已创建，但为空或使用常规条目创建，直到应用失败。

* **ASP.NET Core 模块调试日志：** 变量

疑难解答：

此进程无法启动，这很可能是由应用配置或编程问题引起的。

有关详细信息，请参阅下列主题：

* <xref:test/troubleshoot-azure-iis>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

本主题描述在 Azure 应用服务和 IIS 上托管 ASP.NET Core 应用时的常见错误，并提供特定错误的故障排除建议。

有关一般故障排除指南，请参阅 <xref:test/troubleshoot-azure-iis>。

收集以下信息：

* 浏览器行为（状态代码和错误消息）
* 应用程序事件日志条目
  * Azure 应用服务：请参阅 <xref:test/troubleshoot-azure-iis>。
  * IIS
    1. 选择 Windows 菜单中的“开始”，键入“事件查看器”，然后按 Enter 。
    1. 打开事件查看器后，展开侧栏中的“Windows 日志”>“应用程序”  。
* ASP.NET Core 模块 stdout 和 debug日志条目
  * Azure 应用服务：请参阅 <xref:test/troubleshoot-azure-iis>。
  * IIS：按照 ASP.NET Core 模块主题的[日志创建和重定向](xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection)和[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)部分中的说明进行操作。

将错误信息与以下常见错误进行比较。 如果找到匹配项，请按照故障排除建议进行操作。

本主题中的错误列表并未包含所有错误。 如果遇到此处未列出的错误，请使用本主题底部的“内容反馈”按钮打开一个新问题，并提供有关如何重现错误的详细说明。

[!INCLUDE[Azure App Service Preview Notice](../includes/azure-apps-preview-notice.md)]

## <a name="os-upgrade-removed-the-32-bit-aspnet-core-module"></a>OS 升级删除了 32 位 ASP.NET Core 模块

**应用程序日志：** 未能加载模块 DLL C:\WINDOWS\system32\inetsrv\aspnetcore.dll。 该数据是个错误。

疑难解答：

OS 升级期间不会保留 C:\Windows\SysWOW64\inetsrv 目录中的非 OS 文件。 如果在 OS 升级前已安装 ASP.NET Core 模块，且随后在 OS 升级后在 32 位模式下运行任何应用池，则会遇到此问题。 在 OS 升级后修复 ASP.NET Core 模块。 请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。 运行安装程序时，选择“修复”。

## <a name="missing-site-extension-32-bit-x86-and-64-bit-x64-site-extensions-installed-or-wrong-process-bitness-set"></a>缺少站点扩展、安装了 32 位 (x86) 和 64 位 (x64) 站点扩展，或设置了错误的进程位数

适用于 Azure 应用服务托管的应用。

* **浏览器：** HTTP 错误 500.0 - ANCM 进程内处理程序加载失败

* **应用程序日志：** 调用 hostfxr 以查找进程内请求处理程序失败，未找到任何本机依赖项。 找不到进程内请求处理程序。 调用 hostfxr 捕获的输出：无法找到任何兼容的框架版本。 找不到指定的框架“Microsoft.AspNetCore.App”、版本“{VERSION}-preview-\*”。 未能启动应用程序“/LM/W3SVC/1416782824/ROOT”，ErrorCode“0x8000ffff”。

* **ASP.NET Core 模块 stdout 日志：** 无法找到任何兼容的框架版本。 找不到指定的框架“Microsoft.AspNetCore.App”、版本“{VERSION}-preview-\*”。

疑难解答：

* 如果在预览运行时运行该应用，请安装 32 位 (x86) 或  64位 (x64) 站点扩展，以匹配应用的位数和应用的运行时版本。 请勿同时安装扩展或扩展的多个运行时版本。

  * ASP.NET Core {RUNTIME VERSION} (x86) 运行时
  * ASP.NET Core {RUNTIME VERSION} (x64) 运行时

  重新启动应用。 等待几秒钟，以便应用重新启动。

* 如果在预览运行时运行应用，且同时安装了 32 位 (x86) 和 64 位 (x64) [站点扩展](xref:host-and-deploy/azure-apps/index#install-the-preview-site-extension)，请卸载与应用的位数不匹配的站点扩展。 删除站点扩展之后，重新启动应用。 等待几秒钟，以便应用重新启动。

* 如果在预览运行时运行该应用，并且站点扩展的位数匹配应用的位数，请确认预览站点扩展的运行时版本匹配应用的运行时版本。

* 确认“应用程序设置”中的应用“平台”与应用的位数匹配。

有关详细信息，请参阅 <xref:host-and-deploy/azure-apps/index#install-the-preview-site-extension>。

## <a name="an-x86-app-is-deployed-but-the-app-pool-isnt-enabled-for-32-bit-apps"></a>已部署 x86 应用，但 32 位应用未启用应用池

* **浏览器：** HTTP 错误 500.30 - ANCM 进程内启动失败

* **应用程序日志：** 物理根路径为 '{PATH}' 的应用程序 '/LM/W3SVC/5/ROOT' 遇到意外的托管异常，异常代码= '0xe0434352'。 请检查 stderr 日志以获取详细信息。 物理根路径为 '{PATH}' 的应用程序 '/LM/W3SVC/5/ROOT' 未能加载 clr 和托管应用程序。 CLR 工作线程提前退出

* **ASP.NET Core 模块 stdout 日志：** 日志文件已创建，但为空。

发布自包含应用时，SDK 会捕获此方案。 如果 RID 与平台目标（例如，`win10-x64` RID 与项目文件中的 `<PlatformTarget>x86</PlatformTarget>`）不匹配，则 SDK 会产生错误。

疑难解答：

对于依赖 x86 框架的部署 (`<PlatformTarget>x86</PlatformTarget>`)，请为 32 位应用启用 IIS 应用池。 在 IIS 管理器中，打开应用池的“高级设置”并将“启用 32 位应用程序”设置为 True  。

## <a name="platform-conflicts-with-rid"></a>平台与 RID 冲突

* **浏览器：** HTTP 错误 502.5 - 进程失败

* **应用程序日志：** 物理根路径为 'C:\{PATH}\' 的应用程序 'MACHINE/WEBROOT/APPHOST/{ASSEMBLY}' 未能通过 '"C:\{PATH}{ASSEMBLY}.{exe|dll}" ' 命令行启动进程，ErrorCode = '0x80004005 : ff。

* **ASP.NET Core 模块 stdout 日志：** 未经处理的异常：System.BadImageFormatException：无法加载文件或程序集 '{ASSEMBLY}.dll'。 试图加载的程序的格式不正确。

疑难解答：

* 确认应用在 Kestrel 上本地运行。 进程失败可能是由应用的内部问题导致的。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

* 如果在升级应用和部署更新的程序集时，Azure 应用部署出现此异常，请从先前的部署中手动删除所有文件。 部署升级的应用时，延迟的不兼容程序集可能造成 `System.BadImageFormatException` 异常。

## <a name="uri-endpoint-wrong-or-stopped-website"></a>URI 终结点错误或网站已停止

* **浏览器：** ERR_CONNECTION_REFUSED --或者-- 无法连接

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

疑难解答：

* 确认正在使用的应用的 URI 端点是否正确。 检查绑定。

* 确认 IIS 网站未处于“已停止”状态。

## <a name="corewebengine-or-w3svc-server-features-disabled"></a>已禁用 CoreWebEngine 或 W3SVC 服务器功能

**OS 异常：** 必须安装 IIS 7.0 CoreWebEngine 和 W3SVC 功能才能使用 ASP.NET Core 模块。

疑难解答：

确认启用了适当的角色和功能。 请参阅 [IIS 配置](xref:host-and-deploy/iis/index#iis-configuration)。

## <a name="incorrect-website-physical-path-or-app-missing"></a>网站物理路径不正确或缺少应用

* **浏览器：** 403 禁止访问 - 访问被拒绝 --或者-- 403.14 禁止访问 - Web 服务器被配置为不列出此目录的内容。

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

疑难解答：

检查 IIS 网站的“基本设置”和物理应用文件夹。 确认应用所在的文件夹位于 IIS 网站的物理路径中。

## <a name="incorrect-role-aspnet-core-module-not-installed-or-incorrect-permissions"></a>角色不正确、未安装 ASP.NET Core 模块或权限不正确

* **浏览器：** 500.19 内部服务器错误 - 无法访问请求的页面，因为该页面的相关配置数据无效。 --或者-- 无法显示此页面

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

疑难解答：

* 确认启用了适当的角色。 请参阅 [IIS 配置](xref:host-and-deploy/iis/index#iis-configuration)。

* 打开“程序和功能”或“应用和功能”，然后确认是否安装了 Windows Server Hosting  。 如果已安装的程序列表中没有 Windows Server Hosting，请下载并安装 .NET Core 托管捆绑包。

  [当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

  有关详细信息，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

* 请确保将“应用程序池”>“进程模型”>“Identity”设置为“ApplicationPoolIdentity”，或确保自定义标识具有访问应用部署文件夹的相应权限   。

* 如果卸载了 ASP.NET Core 托管捆绑包并已安装了一个早期版本的托管捆绑包，则 applicationHost.config 文件将不包含 ASP.NET Core 模块分区。 可打开 %windir%/System32/inetsrv/config 处的 applicationHost.config 文件并找到 `<configuration><configSections><sectionGroup name="system.webServer">` 分区组 。 如果分区组中缺少 ASP.NET Core 模块分区，请添加以下分区元素：

  ```xml
  <section name="aspNetCore" overrideModeDefault="Allow" />
  ```

  或者，安装最新版本的 ASP.NET Core 托管捆绑包。 最新版本与受支持的 ASP.NET Core 应用向后兼容。

## <a name="incorrect-processpath-missing-path-variable-hosting-bundle-not-installed-systemiis-not-restarted-vc-redistributable-not-installed-or-dotnetexe-access-violation"></a>processPath 不正确、缺少 PATH 变量、未安装托管捆绑包、未重启系统/IIS、未安装 VC++ Redistributable 或 dotnet.exe 访问冲突

* **浏览器：** HTTP 错误 502.5 - 进程失败

* **应用程序日志：** 物理根路径为 'C:\{PATH}\' 的应用程序 'MACHINE/WEBROOT/APPHOST/{ASSEMBLY}' 未能通过 '"{...}" ' 命令行启动进程 ，ErrorCode = '0x80070002 :0.

* **ASP.NET Core 模块 stdout 日志：** 日志文件已创建，但为空。

疑难解答：

* 确认应用在 Kestrel 上本地运行。 进程失败可能是由应用的内部问题导致的。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

* 检查 web.config 中 `<aspNetCore>` 元素的 processPath 属性，对于依赖框架的部署 (FDD)，确保它为 `dotnet`，对于[独立部署 (SCD)](/dotnet/core/deploying/#self-contained-deployments-scd)，确保它为 `.\{ASSEMBLY}.exe` 。

* 对于 FDD，可能无法通过路径设置访问 dotnet.exe。 确认“系统路径”设置中存在“C:\Program Files\dotnet”\\。

* 对于 FDD，应用池的用户标识可能无法访问 dotnet.exe。 确认应用池用户标识具有访问 C:\Program Files\dotnet 目录的权限。 确认没有为应用池用户标识配置针对 C:\Program Files\dotnet 和应用目录的拒绝规则。

* 可能已部署 FDD 并安装了 .NET Core，但未重启 IIS。 重启服务器，或通过从命令提示符依次执行 net stop was /y 和 net start w3svc 来重启 IIS 。

* 可能已部署 FDD，但未在托管系统上安装 .NET Core 运行时。 如果尚未安装 .NET Core 运行时，则运行系统上的 **.NET Core 托管捆绑包安装程序**。

  [当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

  有关详细信息，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

  如果需要特定的运行时，请从 [.NET 下载存档](https://dotnet.microsoft.com/download/archives)下载运行时并将其安装在系统上。 重启系统，或通过从命令提示符依次执行 net stop was /y 和 net start w3svc 来重启 IIS，完成安装 。

## <a name="incorrect-arguments-of-aspnetcore-element"></a>\<aspNetCore> 元素的参数不正确

* **浏览器：** HTTP 错误 502.5 - 进程失败

* **应用程序日志：** 物理根路径为 'C:\{PATH}\' 的应用程序 'MACHINE/WEBROOT/APPHOST/{ASSEMBLY}' 未能通过 '"dotnet" .\{ASSEMBLY}.dll' 命令行启动进程，ErrorCode = '0x80004005 :80008081。

* **ASP.NET Core 模块 stdout 日志：** 要执行的应用程序不存在：'PATH\{ASSEMBLY}.dll'

疑难解答：

* 确认应用在 Kestrel 上本地运行。 进程失败可能是由应用的内部问题导致的。 有关详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

* 检查 web.config 中 `<aspNetCore>` 元素的 arguments 属性，确认该属性：(a) 对于依赖框架的部署 (FDD) 为 `.\{ASSEMBLY}.dll`；或 (b) 对于独立部署 (SCD)，则为不存在、为空字符串 (`arguments=""`)，或为应用参数列表 (`arguments="{ARGUMENT_1}, {ARGUMENT_2}, ... {ARGUMENT_X}"`) 。

疑难解答：

对于依赖框架的部署 (FDD)，确认已在系统上安装正确的运行时。

## <a name="stopped-application-pool"></a>应用程序池已停止

* **浏览器：** 503 服务不可用

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

疑难解答：

确认应用程序池未处于“已停止”状态。

## <a name="sub-application-includes-a-handlers-section"></a>子应用程序包括 \<handlers> 部分

* **浏览器：** HTTP 错误 500.19 - 内部服务器错误

* **应用程序日志：** 没有条目

* **ASP.NET Core 模块 stdout 日志：** 创建根应用的日志文件并显示正常操作。 未创建子应用的日志文件。

疑难解答：

确认子应用的 web.config 文件不包括 `<handlers>` 部分。

## <a name="stdout-log-path-incorrect"></a>stdout 日志路径不正确

* **浏览器：** 应用正常响应。

* **应用程序日志：** 警告：无法创建 stdoutLogFile \\?\{PATH}\path_doesnt_exist\stdout_{PROCESS ID}_{TIMESTAMP}.log，ErrorCode = -2147024893。

* **ASP.NET Core 模块 stdout 日志：** 未创建日志文件。

疑难解答：

* web.config 的 `<aspNetCore>` 元素中指定的 `stdoutLogFile` 路径不存在。 有关详细信息，请参阅 [ASP.NET Core 模块：日志创建和重定向](xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection)。

* 应用池用户没有 stdout 日志路径的写入权限。

## <a name="application-configuration-general-issue"></a>应用程序配置常见问题

* **浏览器：** HTTP 错误 502.5 - 进程失败

* **应用程序日志：** 物理根路径为 'C:\{PATH}\' 的应用程序 'MACHINE/WEBROOT/APPHOST/{ASSEMBLY}' 通过 '"C:\{PATH}\{ASSEMBLY}.{exe|dll}" ' 命令行创建了进程，但该进程出现故障、未响应或未在给定的端口 '{PORT}' 上侦听，ErrorCode = '{ERROR CODE}'

* **ASP.NET Core 模块 stdout 日志：** 日志文件已创建，但为空。

疑难解答：

此进程无法启动，这很可能是由应用配置或编程问题引起的。

有关详细信息，请参阅下列主题：

* <xref:test/troubleshoot-azure-iis>
* <xref:test/troubleshoot>

::: moniker-end
