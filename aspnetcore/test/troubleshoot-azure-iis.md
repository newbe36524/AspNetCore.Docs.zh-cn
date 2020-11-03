---
title: 对 Azure 应用服务和 IIS 上的 ASP.NET Core 进行故障排除
author: rick-anderson
description: 了解如何诊断Azure 应用服务和 Internet Information Services (IIS) 的 ASP.NET Core 应用部署问题。
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
uid: test/troubleshoot-azure-iis
ms.openlocfilehash: bc968489a35405c1be0a4ac2e52e6dff4c3af050
ms.sourcegitcommit: 8fcb08312a59c37e3542e7a67dad25faf5bb8e76
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2020
ms.locfileid: "90009734"
---
# <a name="troubleshoot-aspnet-core-on-azure-app-service-and-iis"></a>对 Azure 应用服务和 IIS 上的 ASP.NET Core 进行故障排除

作者：[Justin Kotalik](https://github.com/jkotalik)

::: moniker range=">= aspnetcore-3.0"

本文提供了有关常见应用启动错误的信息，以及在将应用部署到 Azure 应用服务或 IIS 时如何诊断错误的说明：

[应用启动错误](#app-startup-errors)  
介绍常见的启动 HTTP 状态代码方案。

[排查 Azure 应用服务中的问题](#troubleshoot-on-azure-app-service)  
为部署到 Azure 应用服务的应用提供疑难解答建议。

[IIS 疑难解答](#troubleshoot-on-iis)  
为部署到 IIS 或在本地 IIS Express 上运行的应用提供疑难解答建议。 本指南适用于 Windows Server 和 Windows 桌面部署。

[清除包缓存](#clear-package-caches)  
说明当执行重大升级或更改包版本时，不一致的包中断应用时应如何处理。

[其他资源](#additional-resources)  
列出其他疑难解答主题。

## <a name="app-startup-errors"></a>应用启动错误

在 Visual Studio 中，ASP.NET Core 项目默认为在调试期间进行 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 托管。 本地调试时出现的“502.5 - 进程失败”或“500.30 - 启动失败”可以使用本主题中的建议进行诊断 。

### <a name="40314-forbidden"></a>403.14 禁止

应用无法启动。 记录以下错误：

```
The Web server is configured to not list the contents of this directory.
```

此错误通常是由于托管系统上的部署中断引起的，包括以下任何情况：

* 该应用部署到托管系统上的错误文件夹。
* 部署过程未能将所有应用的文件和文件夹移到托管系统上的部署文件夹中。
* 部署中缺少 web.config 文件，或者 web.config 文件内容格式不正确 。

执行以下步骤：

1. 从托管系统上的部署文件夹中删除所有文件和文件夹。
1. 使用常规部署方法（如 Visual Studio、PowerShell 或手动部署）将应用“发布”文件夹的内容重新部署到托管系统：
   * 确认部署中存在 web.config 文件，并且其内容正确。
   * 在 Azure 应用服务上托管时，请确认该应用已部署到 `D:\home\site\wwwroot` 文件夹。
   * 当应用由 IIS 托管时，请确认应用已部署到“IIS 管理器”的“基本设置”中显示的 IIS 物理路径  。
1. 通过将托管系统上的部署与项目“发布”文件夹的内容进行比较，确认已部署应用的所有文件和文件夹。

有关已发布 ASP.NET Core 应用布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。 有关 web.config 文件的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>。

### <a name="500-internal-server-error"></a>500 内部服务器错误

应用启动，但某个错误阻止了服务器完成请求。

在启动期间或在创建响应时，应用的代码内出现此错误。 响应可能不包含任何内容，或响应可能会在浏览器中显示为“500 内部服务器错误”。 应用程序事件日志通常表明应用正常启动。 从服务器的角度来看，这是正确的。 应用已启动，但无法生成有效的响应。 在服务器上的命令提示符下运行应用，或启用 ASP.NET Core 模块 stdout 日志来解决问题。

### <a name="5000-in-process-handler-load-failure"></a>500.0 进程内处理程序加载失败

工作进程失败。 应用不启动。

加载 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)组件时出现未知错误。 请执行以下一项操作：

* 联系 [Microsoft 支持部门](https://support.microsoft.com/oas/default.aspx?prid=15832)（依次选择“开发人员工具”和“ASP.NET Core”） 。
* 在 Stack Overflow 上提出问题。
* 在 [GitHub 存储库](https://github.com/dotnet/AspNetCore)中提出问题。

### <a name="50030-in-process-startup-failure"></a>500.30 进程内启动失败

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)尝试进程内启动 .NET Core CLR，但启动失败。 进程启动失败的原因通常可通过“应用程序事件日志”和“ASP.NET Core 模块 stdout 日志”中的条目进行确定。

常见失败情况：

* 由于目标 ASP.NET Core 共享框架版本不存在，因此应用配置错误。 检查目标计算机上安装的 ASP.NET Core 共享框架版本。
* 使用 Azure Key Vault 时，缺少对 Key Vault 的权限。 检查目标 Key Vault 中的访问策略，确保授予了正确的权限。

### <a name="50031-ancm-failed-to-find-native-dependencies"></a>500.31 ANCM 找不到本机依赖项

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)尝试进程内启动 .NET Core 运行时，但启动失败。 此类启动失败的最常见原因是未安装 `Microsoft.NETCore.App` 或 `Microsoft.AspNetCore.App`运行时。 如果将应用部署为面向 ASP.NET Core 3.0，并且计算机上不存在该版本，则会发生此错误。 示例错误消息如下所示：

```
The specified framework 'Microsoft.NETCore.App', version '3.0.0' was not found.
  - The following frameworks were found:
      2.2.1 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview5-27626-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27713-13 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27714-15 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
      3.0.0-preview6-27723-08 at [C:\Program Files\dotnet\x64\shared\Microsoft.NETCore.App]
```

错误消息列出了所有已安装的 .NET Core 版本以及应用请求的版本。 请通过以下一种方法修复此错误：

* 在计算机上安装适当版本的 .NET Core。
* 更改应用，使其面向计算机上已存在的 .NET Core 版本。
* 将应用作为[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)进行发布。

在开发过程（`ASPNETCORE_ENVIRONMENT` 环境变量设置为 `Development`）中运行时，HTTP 响应中会写入特定的错误。 进程启动失败的原因也可在“应用程序事件日志”中找到。

### <a name="50032-ancm-failed-to-load-dll"></a>500.32 ANCM 无法加载 dll

工作进程失败。 应用不启动。

此错误的最常见原因是针对不兼容的处理器体系结构发布了应用。 如果工作进程作为 32 位应用运行，而将应用发布为面向 64 位，则会发生此错误。

请通过以下一种方法修复此错误：

* 针对同一处理器体系结构将应用作为工作进程进行重新发布。
* 将应用作为[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-executables-fde)进行发布。

### <a name="50033-ancm-request-handler-load-failure"></a>500.33 ANCM 请求处理程序加载失败

工作进程失败。 应用不启动。

应用未引用 `Microsoft.AspNetCore.App` 框架。 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)只能托管面向 `Microsoft.AspNetCore.App` 框架的应用。

要修复此错误，请确保应用面向 `Microsoft.AspNetCore.App` 框架。 检查 `.runtimeconfig.json` 以验证该应用所面向的框架。

### <a name="50034-ancm-mixed-hosting-models-not-supported"></a>500.34 ANCM 混合托管模型不受支持

工作进程不能在同一进程中同时运行进程内应用和进程外应用。

要修复此错误，请在单独的 IIS 应用程序池中运行应用。

### <a name="50035-ancm-multiple-in-process-applications-in-same-process"></a>500.35 ANCM 同一进程内有多个进程内应用程序

工作进程不能在同一进程中运行多个进程内应用。

要修复此错误，请在单独的 IIS 应用程序池中运行应用。

### <a name="50036-ancm-out-of-process-handler-load-failure"></a>500.36 ANCM 进程外处理程序加载失败

进程外请求处理程序 aspnetcorev2_outofprocess.dll 未与 aspnetcorev2.dll 文件相邻 。 这表示 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)的安装已损坏。

要修复此错误，请修复 [.NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)（对于 IIS）或 Visual Studio（对于 IIS Express）的安装。

### <a name="50037-ancm-failed-to-start-within-startup-time-limit"></a>500.37 ANCM 无法在启动时间限制内启动

ANCM 无法在提供的启动时间限制内启动。 默认情况下，超时时间为 120 秒。

在同一台计算机上启动大量应用时，则可能发生此错误。 在启动期间检查服务器上的 CPU/内存使用峰值。 可能需要交错执行多个应用程序的启动进程。

### <a name="50038-ancm-application-dll-not-found"></a>500.38 ANCM 找不到应用程序 DLL

ANCM 找不到应用程序 ANCM，该内容应显示在可执行文件的旁边。

在使用进程内托管模型托管打包为[单文件可执行程序](/dotnet/core/whats-new/dotnet-core-3-0#single-file-executables)的应用。 该进程内模型要求 ANCM 将 .NET Core 应用加载到现有 IIS 进程中。 单文件部署模型不支持此方案。 请在应用的项目文件中使用下述方法之一来修复此错误：

1. 通过将 `PublishSingleFile` MSBuild 属性设置为 `false` 来禁用单文件发布。
1. 通过将 `AspNetCoreHostingModel` MSBuild 属性设置为 `OutOfProcess` 来切换到进程外托管模型。

### <a name="5025-process-failure"></a>502.5 进程失败

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)尝试启动工作进程，但启动失败。 进程启动失败的原因通常可通过“应用程序事件日志”和“ASP.NET Core 模块 stdout 日志”中的条目进行确定。

常见的失败情况是，由于目标 ASP.NET Core 共享框架版本不存在，因此应用配置错误。 检查目标计算机上安装的 ASP.NET Core 共享框架版本。 共享框架是安装在计算机上并由 `Microsoft.AspNetCore.App` 等元包引用的一组程序集（.dll 文件）。 元包引用可以指定所需的最低版本。 有关详细信息，请参阅[共享框架](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/)。

托管或应用配置错误导致工作进程失败时，将返回“502.5 进程失败”错误页面：

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>未能启动应用程序（错误代码“0x800700c1”）

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

应用未能启动，因为应用的程序集 (.dll) 无法加载。

当已发布的应用与 w3wp/iisexpress 进程之间的位数不匹配时，会出现此错误。

确认应用池的 32 位设置正确：

1. 在 IIS 管理器的“应用程序池”中选择应用池。
1. 在“操作”面板中的“编辑应用程序池”下选择“高级设置”。
1. 设置“启用 32 位应用程序”：
   * 如果部署 32 位 (x86) 应用，则将值设置为 `True`。
   * 如果部署 64 位 (x64) 应用，则将值设置为 `False`。

确认项目文件中的 `<Platform>` MSBuild 属性与应用的已发布位数之间没有冲突。

### <a name="connection-reset"></a>连接重置

如果在发送标头后出现错误，则服务器在出现错误时发送“500 内部服务器错误”已经太晚了。 通常在序列化响应的复杂对象期间出现错误时发生这种情况。 此类型的错误在客户端上显示为“连接重置”错误。 [应用程序日志记录](xref:fundamentals/logging/index)可以帮助解决这些类型的错误。

### <a name="default-startup-limits"></a>默认启动限制

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)的默认“startupTimeLimit”配置为 120 秒。 保留默认值时，在模块记录进程故障之前，可能最多需要两分钟来启动应用。 有关配置模块的信息，请参阅 [aspNetCore 元素的属性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)。

## <a name="troubleshoot-on-azure-app-service"></a>排查 Azure 应用服务中的问题

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>应用程序事件日志（Azure 应用服务）

若要访问应用程序事件日志，请在 Azure 门户中使用“诊断并解决问题”边栏选项卡：

1. 在 Azure 门户中打开“应用服务”中的应用。
1. 选择“诊断并解决问题”。
1. 选择“诊断工具”标题。
1. 在“支持工具”下，选择“应用程序事件”按钮 。
1. 检查“源”列中由 IIS AspNetCoreModule 或 IIS AspNetCoreModule V2条目提供的最新错误 。

使用“诊断并解决问题”边栏选项卡的替代方法是直接使用 [Kudu](https://github.com/projectkudu/kudu/wiki) 检查应用程序事件日志文件：

1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开 LogFiles 文件夹。
1. 选择 eventlog.xml 文件旁边的铅笔图标。
1. 检查日志。 滚动到日志底部以查看最新事件。

### <a name="run-the-app-in-the-kudu-console"></a>在 Kudu 控制台中运行应用

许多启动错误未在应用程序事件日志中生成有用信息。 可以在 [Kudu](https://github.com/projectkudu/kudu/wiki) 远程执行控制台中运行应用以发现错误：

1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。

#### <a name="test-a-32-bit-x86-app"></a>测试 32 位 (x86) 应用

**当前版本**

1. `cd d:\home\site\wwwroot`
1. 运行应用：
   * 如果应用是[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * 如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：

     ```console
     {ASSEMBLY NAME}.exe
     ```

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

**在预览版上运行的依赖框架的部署**

必须安装 ASP.NET Core {VERSION} (x86) 运行时站点扩展。

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32`（`{X.Y}` 是运行时版本）
1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

#### <a name="test-a-64-bit-x64-app"></a>测试 64 位 (x64) 应用

**当前版本**

* 如果应用是 64 位 (x64) [依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：
  1. `cd D:\Program Files\dotnet`
  1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* 如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：
  1. `cd D:\home\site\wwwroot`
  1. 运行应用：`{ASSEMBLY NAME}.exe`

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

**在预览版上运行的依赖框架的部署**

必须安装 ASP.NET Core {VERSION} (x64) 运行时站点扩展。

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64`（`{X.Y}` 是运行时版本）
1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.NET Core 模块 stdout 日志（Azure 应用服务）

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。 仅使用 stdout 日志记录来解决应用启动问题。
>
> 对于在 ASP.NET Core 应用启动后生成的常规日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

ASP.NET Core 模块 stdout 日志通常记录应用程序事件日志中找不到的有用错误消息。 若要启用和查看 stdout 日志，请执行以下操作：

1. 在 Azure 门户中，导航到 Web 应用。
1. 在“应用服务”边栏选项卡中，在搜索框中输入 kudu 。
1. 依次选择“高级工具”>“Go” 。
1. 选择“调试控制台”>“CMD”。
1. 导航到 site/wwwroot
1. 选择铅笔图标以编辑 web.config 文件。
1. 在 `<aspNetCore />` 元素中，设置 `stdoutLogEnabled="true"` 并选择“保存”。

故障排除完成后，通过设置 `stdoutLogEnabled="false"` 来禁用 stdout 日志记录。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>。

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>ASP.NET Core 模块调试日志（Azure 应用服务）

ASP.NET Core 模块调试日志从 ASP.NET Core 模块提供了更多、更详细的日志记录。 若要启用和查看 stdout 日志，请执行以下操作：

1. 要启用增强的诊断日志，请执行以下任一操作：
   * 按照[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)中的说明配置应用以获取增强的诊断日志记录。 重新部署应用。
   * 使用 Kudu 控制台将[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)中显示的 `<handlerSettings>` 添加到动态应用的 web.config 文件中：
     1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
     1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
     1. 打开路径“site > wwwroot”下的文件夹。 通过选择铅笔按钮编辑 web.config 文件。 添加 `<handlerSettings>` 部分（如[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)中所示）。 选择“保存”按钮。
1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开路径“site > wwwroot”下的文件夹。 如果没有为 aspnetcore-debug.log 文件提供路径，则该文件将显示在列表中。 如果提供了路径，请导航到日志文件的位置。
1. 使用文件名旁边的铅笔按钮打开日志文件。

故障排除完成后，禁用调试日志记录：

要禁用增强的调试日志，请执行以下任一操作：

* 从本地删除 web.config 文件中的 `<handlerSettings>` 并重新部署该应用。
* 使用 Kudu 控制台编辑 web.config 文件并删除 `<handlerSettings>` 部分。 保存该文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>。

> [!WARNING]
> 无法禁用调试日志可能会导致应用或服务器出现故障。 日志文件大小没有任何限制。 仅使用调试日志记录来解决应用启动问题。
>
> 对于在 ASP.NET Core 应用启动后生成的常规日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="slow-or-hanging-app-azure-app-service"></a>应用缓慢或挂起（Azure 应用服务）

如果应用程序响应缓慢或在有请求时挂起，请参阅以下文章：

* [解决 Azure 应用服务中 Web 应用性能缓慢的问题](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [使用故障诊断程序站点扩展来捕获 Azure Web 应用程序上间歇性异常问题或性能问题的转储](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>监视边栏选项卡

监视边栏选项卡提供了本主题前面所述的方法的替代故障排除体验。 这些边栏选项卡可用于诊断 500 系列错误。

确认是否已安装 ASP.NET Core 扩展。 如果未安装扩展，请手动进行安装：

1. 在“开发工具”边栏选项卡部分中，选择“扩展”边栏选项卡。
1. “ASP.NET Core 扩展”应显示在列表中。
1. 如果未安装扩展，请选择“添加”按钮。
1. 从列表中选择“ASP.NET Core 扩展”。
1. 选择“确定”以接受法律条款。
1. 选择“添加扩展”边栏选项卡上的“确定”。
1. 信息性弹出消息指示成功安装扩展的时间。

如果未启用 stdout 日志记录，请执行以下步骤：

1. 在 Azure 门户中，选择“开发工具”区域中的“高级工具”边栏选项卡。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开路径“site>wwwroot”下的文件夹，然后向下滚动以显示列表底部的 web.config 文件 。
1. 单击“web.config”文件旁边的铅笔图标。
1. 将“stdoutLogEnabled”设置为 `true`，并将“stdoutLogFile”路径更改为 `\\?\%home%\LogFiles\stdout`。
1. 选择“保存”以保存已更新的 web.config 文件。

继续激活诊断日志记录：

1. 在 Azure 门户中，选择“诊断日志”边栏选项卡。
1. 选择“应用程序日志记录(文件系统)”和“详细错误消息”的“开”开关。 选择边栏选项卡顶部的“保存”按钮。
1. 若要包含失败请求跟踪（也称为失败请求事件缓冲 (FREB) 日志记录），请选择“失败请求跟踪”的“开”开关。
1. 选择“日志流”边栏选项卡，将在门户中的“诊断日志”边栏选项卡下立即列出。
1. 向应用发出请求。
1. 在日志流数据中，指示了错误的原因。

故障排除完成后，请务必禁用 stdout 日志记录。

若要查看失败请求跟踪日志（FREB 日志），请执行以下操作：

1. 在 Azure 门户中导航到“诊断并解决问题”边栏选项卡。
1. 从侧栏的“支持工具”区域中选择“失败请求跟踪日志”。

有关详细信息，请参阅[“在 Azure 应用服务中启用 Web 应用的诊断日志记录”主题的“失败请求跟踪”部分](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces)和 [Azure 中的 Web 应用的应用程序性能常见问题：如何打开失败请求跟踪？](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)。

有关详细信息，请参阅[在 Azure 应用服务中启用 Web 应用的诊断日志记录](/azure/app-service/web-sites-enable-diagnostic-log)。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。
>
> 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

## <a name="troubleshoot-on-iis"></a>IIS 疑难解答

### <a name="application-event-log-iis"></a>应用程序事件日志 (IIS)

访问应用程序事件日志：

1. 打开“开始”菜单，搜索“事件查看器”，然后选择“事件查看器”应用。
1. 在“事件查看器”中，打开“Windows 日志”节点。
1. 选择“应用程序”以打开应用程序事件日志。
1. 搜索与失败应用相关联的错误。 错误具有“源”列中“IIS AspNetCore 模块”或“IIS Express AspNetCore 模块”的值。

### <a name="run-the-app-at-a-command-prompt"></a>在命令提示符处运行应用

许多启动错误未在应用程序事件日志中生成有用信息。 可以通过在托管系统上在命令提示符处运行应用来找到某些错误的原因。

#### <a name="framework-dependent-deployment"></a>依赖框架的部署

如果应用是[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：

1. 在命令提示符处，导航到部署文件夹并通过使用 dotnet.exe 执行应用的程序集来运行应用。 在以下命令中，替换 \<assembly_name> 的应用程序集的名称：`dotnet .\<assembly_name>.dll`。
1. 来自应用且显示任何错误的控制台输出将写入控制台窗口。
1. 如果向应用发出请求时出现错误，请向 Kestrel 侦听所在的主机和端口发出请求。 如果使用默认主机和端口，请向 `http://localhost:5000/` 发出请求。 如果应用在 Kestrel 终结点地址处正常响应，则问题更可能与承载配置相关，而不太可能在于应用。

#### <a name="self-contained-deployment"></a>独立部署

如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：

1. 在命令提示符处，导航到部署文件夹并运行应用的可执行文件。 在以下命令中，替换 \<assembly_name> 的应用程序集的名称：`<assembly_name>.exe`。
1. 来自应用且显示任何错误的控制台输出将写入控制台窗口。
1. 如果向应用发出请求时出现错误，请向 Kestrel 侦听所在的主机和端口发出请求。 如果使用默认主机和端口，请向 `http://localhost:5000/` 发出请求。 如果应用在 Kestrel 终结点地址处正常响应，则问题更可能与承载配置相关，而不太可能在于应用。

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.NET Core 模块 stdout 日志 (IIS)

若要启用和查看 stdout 日志，请执行以下操作：

1. 在托管系统上导航到站点的部署文件夹。
1. 如果 logs 文件夹不存在，请创建该文件夹。 有关如何启用 MSBuild 以在部署中自动创建 logs 文件夹的说明，请参阅[目录结构](xref:host-and-deploy/directory-structure)主题。
1. 编辑 web.config 文件。 将“stdoutLogEnabled”设置为 `true` 并更改“stdoutLogFile”路径以指向 logs 文件夹（例如，`.\logs\stdout`）。 路径中的 `stdout` 是日志文件名的前缀。 创建日志时，将自动添加时间戳、进程 ID 和文件扩展名。 如果将 `stdout` 用作文件名的前缀，典型的日志文件将命名为“stdout_20180205184032_5412.log”。
1. 请确保应用程序池的标识具有对日志文件夹的写入权限。
1. 保存已更新的 web.config 文件。
1. 向应用发出请求。
1. 导航到 logs 文件夹。 查找并打开最新的 stdout 日志。
1. 研究日志以查找错误。

故障排除完成后，禁用 stdout 日志记录：

1. 编辑 web.config 文件。
1. 将“stdoutLogEnabled”设置为 `false`。
1. 保存该文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。
>
> 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="aspnet-core-module-debug-log-iis"></a>ASP.NET Core 模块调试日志 (IIS)

将以下处理程序设置添加到应用的 web.config 文件以启用 ASP.NET Core 模块调试日志：

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

确认为日志指定的路径存在，并且应用池的标识具有该位置的写入权限。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>。

### <a name="enable-the-developer-exception-page"></a>启用开发人员异常页面

`ASPNETCORE_ENVIRONMENT` [环境变量可以添加到 web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) 以在开发环境中运行应用。 只要在应用启动时环境不被主机生成器上的 `UseEnvironment` 重写，设置环境变量就会在运行应用时显示“[开发人员异常页面](xref:fundamentals/error-handling)”。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

仅建议在未向 Internet 公开的暂存服务器和测试服务器上设置 `ASPNETCORE_ENVIRONMENT` 的环境变量。 在故障排除后从 web.config 文件中删除环境变量。 有关设置 web.config 中的环境变量的信息，请参阅 [aspNetCore 的 environmentVariables 子元素](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)。

### <a name="obtain-data-from-an-app"></a>从应用中获取数据

如果应用能够响应请求，则使用终端内联中间件从应用中获取请求、连接和其他数据。 有关详细信息和示例代码，请参阅<xref:test/troubleshoot#obtain-data-from-an-app>。

### <a name="slow-or-hanging-app-iis"></a>应用缓慢或挂起 (IIS)

故障转储是系统内存的一个快照，可帮助确定应用崩溃、启动故障或应用速度缓慢等状况的原因。

#### <a name="app-crashes-or-encounters-an-exception"></a>应用崩溃或引发异常

从 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) 中获取转储并进行分析：

1. 创建文件夹，将崩溃转储文件保存在 `c:\dumps`。 应用池必须对该文件夹具有写权限。
1. 运行 [EnableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1)：
   * 如果应用使用[进程内托管模型](xref:host-and-deploy/iis/index#in-process-hosting-model)，则请为 w3wp.exe 运行脚本：

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * 如果应用使用[进程外托管模型](xref:host-and-deploy/iis/index#out-of-process-hosting-model)，则请为 dotnet.exe 运行脚本：

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. 在造成崩溃的条件下运行应用。
1. 出现崩溃后，运行 [DisableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1)：
   * 如果应用使用[进程内托管模型](xref:host-and-deploy/iis/index#in-process-hosting-model)，则请为 w3wp.exe 运行脚本：

     ```console
     .\DisableDumps w3wp.exe
     ```

   * 如果应用使用[进程外托管模型](xref:host-and-deploy/iis/index#out-of-process-hosting-model)，则请为 dotnet.exe 运行脚本：

     ```console
     .\DisableDumps dotnet.exe
     ```

在应用崩溃并完成转储收集后，即可正常终止应用。 PowerShell 脚本会 WER 来按应用收集转储（最多收集 5 个）。

> [!WARNING]
> 崩溃转储可能会占用大量磁盘空间（每个最多占用数 GB）。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>应用挂起、在启动期间失败或正常运行

如果应用挂起（停止响应但不崩溃）、在启动期间失败或者正常运行*hangs*，请参阅[用户模式转储文件：选择最佳工具](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)，以选择适合用于生成转储的工具。

#### <a name="analyze-the-dump"></a>分析转储

可采用几种方法来分析转储。 有关详细信息，请参阅[分析用户模式转储文件](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)。

## <a name="clear-package-caches"></a>清除包缓存

正常运行的应用在开发计算机上升级 .NET Core SDK 或在应用内更改包版本后可能会立即出现故障。 在某些情况下，不同的包可能在执行主要升级时中断应用。 可以按照以下说明来修复其中大部分问题：

1. 删除 bin 和 obj 文件夹。
1. 通过从命令行界面执行 [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) 清除包缓存。

   清除包缓存还可通过使用 [nuget.exe](https://www.nuget.org/downloads) 工具并执行命令 `nuget locals all -clear` 来完成。 *nuget.exe* 不是与 Windows 桌面操作系统的捆绑安装，必须从 [NuGet 网站](https://www.nuget.org/downloads)中单独获取。

1. 还原并重新生成项目。
1. 在重新部署应用前，在服务器上删除部署文件夹中的所有文件。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure 文档

* [用于 ASP.NET Core 的 Application Insights](/azure/application-insights/app-insights-asp-net-core)
* [“使用 Visual Studio 对 Azure 应用服务中的 Web 应用进行故障排除”的“远程调试 Web 应用”一节](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Azure 应用服务诊断概述](/azure/app-service/app-service-diagnostics)
* [如何：在 Azure 应用服务中监视应用](/azure/app-service/web-sites-monitor)
* [使用 Visual Studio 对 Azure 应用服务中的 Web 应用进行故障排除](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [解决 Azure Web 应用中的“502 错误的网关”和“503 服务不可用”HTTP 错误](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [解决 Azure 应用服务中 Web 应用性能缓慢的问题](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Azure 中的 Web 应用的应用程序性能常见问题](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web 应用沙盒（应用服务运行时执行限制）](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday：Azure 应用服务诊断和疑难解答体验（12 分钟视频）](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio 文档

* [在 Azure 中的 IIS 上的 Visual Studio 2017 中远程调试 ASP.NET Core](/visualstudio/debugger/remote-debugging-azure)
* [在远程 IIS 计算机的 Visual Studio 2017 中远程调试 ASP.NET Core](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [了解如何使用 Visual Studio 进行调试](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Visual Studio Code 文档

* [使用 Visual Studio Code 进行调试](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end

::: moniker range="= aspnetcore-2.2"

本文提供了有关常见应用启动错误的信息，以及在将应用部署到 Azure 应用服务或 IIS 时如何诊断错误的说明：

[应用启动错误](#app-startup-errors)  
介绍常见的启动 HTTP 状态代码方案。

[排查 Azure 应用服务中的问题](#troubleshoot-on-azure-app-service)  
为部署到 Azure 应用服务的应用提供疑难解答建议。

[IIS 疑难解答](#troubleshoot-on-iis)  
为部署到 IIS 或在本地 IIS Express 上运行的应用提供疑难解答建议。 本指南适用于 Windows Server 和 Windows 桌面部署。

[清除包缓存](#clear-package-caches)  
说明当执行重大升级或更改包版本时，不一致的包中断应用时应如何处理。

[其他资源](#additional-resources)  
列出其他疑难解答主题。

## <a name="app-startup-errors"></a>应用启动错误

在 Visual Studio 中，ASP.NET Core 项目默认为在调试期间进行 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 托管。 本地调试时出现的“502.5 - 进程失败”或“500.30 - 启动失败”可以使用本主题中的建议进行诊断 。

### <a name="40314-forbidden"></a>403.14 禁止

应用无法启动。 记录以下错误：

```
The Web server is configured to not list the contents of this directory.
```

此错误通常是由于托管系统上的部署中断引起的，包括以下任何情况：

* 该应用部署到托管系统上的错误文件夹。
* 部署过程未能将所有应用的文件和文件夹移到托管系统上的部署文件夹中。
* 部署中缺少 web.config 文件，或者 web.config 文件内容格式不正确 。

执行以下步骤：

1. 从托管系统上的部署文件夹中删除所有文件和文件夹。
1. 使用常规部署方法（如 Visual Studio、PowerShell 或手动部署）将应用“发布”文件夹的内容重新部署到托管系统：
   * 确认部署中存在 web.config 文件，并且其内容正确。
   * 在 Azure 应用服务上托管时，请确认该应用已部署到 `D:\home\site\wwwroot` 文件夹。
   * 当应用由 IIS 托管时，请确认应用已部署到“IIS 管理器”的“基本设置”中显示的 IIS 物理路径  。
1. 通过将托管系统上的部署与项目“发布”文件夹的内容进行比较，确认已部署应用的所有文件和文件夹。

有关已发布 ASP.NET Core 应用布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。 有关 web.config 文件的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>。

### <a name="500-internal-server-error"></a>500 内部服务器错误

应用启动，但某个错误阻止了服务器完成请求。

在启动期间或在创建响应时，应用的代码内出现此错误。 响应可能不包含任何内容，或响应可能会在浏览器中显示为“500 内部服务器错误”。 应用程序事件日志通常表明应用正常启动。 从服务器的角度来看，这是正确的。 应用已启动，但无法生成有效的响应。 在服务器上的命令提示符下运行应用，或启用 ASP.NET Core 模块 stdout 日志来解决问题。

### <a name="5000-in-process-handler-load-failure"></a>500.0 进程内处理程序加载失败

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)无法找到 .NET Core CLR 和进程内请求处理程序 (aspnetcorev2_inprocess.dll)。 检查：

* 该应用针对 [Microsoft.AspNetCore.Server.IIS NuGet](https://www.nuget.org/packages/Microsoft.AspNetCore.Server.IIS) 包或 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)。
* 目标计算机上安装了该应用所针对的 ASP.NET Core 共享框架版本。

### <a name="5000-out-of-process-handler-load-failure"></a>500.0 进程外处理程序加载失败

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)无法找到进程外托管请求处理程序。 请确保 aspnetcorev2.dll 旁边的子文件夹中存在 aspnetcorev2_outofprocess.dll 。

### <a name="5025-process-failure"></a>502.5 进程失败

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)尝试启动工作进程，但启动失败。 进程启动失败的原因通常可通过“应用程序事件日志”和“ASP.NET Core 模块 stdout 日志”中的条目进行确定。

常见的失败情况是，由于目标 ASP.NET Core 共享框架版本不存在，因此应用配置错误。 检查目标计算机上安装的 ASP.NET Core 共享框架版本。 共享框架是安装在计算机上并由 `Microsoft.AspNetCore.App` 等元包引用的一组程序集（.dll 文件）。 元包引用可以指定所需的最低版本。 有关详细信息，请参阅[共享框架](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/)。

托管或应用配置错误导致工作进程失败时，将返回“502.5 进程失败”错误页面：

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>未能启动应用程序（错误代码“0x800700c1”）

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

应用未能启动，因为应用的程序集 (.dll) 无法加载。

当已发布的应用与 w3wp/iisexpress 进程之间的位数不匹配时，会出现此错误。

确认应用池的 32 位设置正确：

1. 在 IIS 管理器的“应用程序池”中选择应用池。
1. 在“操作”面板中的“编辑应用程序池”下选择“高级设置”。
1. 设置“启用 32 位应用程序”：
   * 如果部署 32 位 (x86) 应用，则将值设置为 `True`。
   * 如果部署 64 位 (x64) 应用，则将值设置为 `False`。

确认项目文件中的 `<Platform>` MSBuild 属性与应用的已发布位数之间没有冲突。

### <a name="connection-reset"></a>连接重置

如果在发送标头后出现错误，则服务器在出现错误时发送“500 内部服务器错误”已经太晚了。 通常在序列化响应的复杂对象期间出现错误时发生这种情况。 此类型的错误在客户端上显示为“连接重置”错误。 [应用程序日志记录](xref:fundamentals/logging/index)可以帮助解决这些类型的错误。

### <a name="default-startup-limits"></a>默认启动限制

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)的默认“startupTimeLimit”配置为 120 秒。 保留默认值时，在模块记录进程故障之前，可能最多需要两分钟来启动应用。 有关配置模块的信息，请参阅 [aspNetCore 元素的属性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)。

## <a name="troubleshoot-on-azure-app-service"></a>排查 Azure 应用服务中的问题

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>应用程序事件日志（Azure 应用服务）

若要访问应用程序事件日志，请在 Azure 门户中使用“诊断并解决问题”边栏选项卡：

1. 在 Azure 门户中打开“应用服务”中的应用。
1. 选择“诊断并解决问题”。
1. 选择“诊断工具”标题。
1. 在“支持工具”下，选择“应用程序事件”按钮 。
1. 检查“源”列中由 IIS AspNetCoreModule 或 IIS AspNetCoreModule V2条目提供的最新错误 。

使用“诊断并解决问题”边栏选项卡的替代方法是直接使用 [Kudu](https://github.com/projectkudu/kudu/wiki) 检查应用程序事件日志文件：

1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开 LogFiles 文件夹。
1. 选择 eventlog.xml 文件旁边的铅笔图标。
1. 检查日志。 滚动到日志底部以查看最新事件。

### <a name="run-the-app-in-the-kudu-console"></a>在 Kudu 控制台中运行应用

许多启动错误未在应用程序事件日志中生成有用信息。 可以在 [Kudu](https://github.com/projectkudu/kudu/wiki) 远程执行控制台中运行应用以发现错误：

1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。

#### <a name="test-a-32-bit-x86-app"></a>测试 32 位 (x86) 应用

**当前版本**

1. `cd d:\home\site\wwwroot`
1. 运行应用：
   * 如果应用是[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * 如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：

     ```console
     {ASSEMBLY NAME}.exe
     ```

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

**在预览版上运行的依赖框架的部署**

必须安装 ASP.NET Core {VERSION} (x86) 运行时站点扩展。

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32`（`{X.Y}` 是运行时版本）
1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

应用的控制台输出（显示任何错误）会传送到 Kudu 控制台。

#### <a name="test-a-64-bit-x64-app"></a>测试 64 位 (x64) 应用

**当前版本**

* 如果应用是 64 位 (x64) [依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：
  1. `cd D:\Program Files\dotnet`
  1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* 如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：
  1. `cd D:\home\site\wwwroot`
  1. 运行应用：`{ASSEMBLY NAME}.exe`

应用的控制台输出（显示任何错误）会传送到 Kudu 控制台。

**在预览版上运行的依赖框架的部署**

必须安装 ASP.NET Core {VERSION} (x64) 运行时站点扩展。

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64`（`{X.Y}` 是运行时版本）
1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.NET Core 模块 stdout 日志（Azure 应用服务）

ASP.NET Core 模块 stdout 日志通常记录应用程序事件日志中找不到的有用错误消息。 若要启用和查看 stdout 日志，请执行以下操作：

1. 在 Azure 门户中导航到“诊断并解决问题”边栏选项卡。
1. 在“选择问题类别”下，选择“Web 应用关闭”按钮。
1. 在“建议的解决方案”>“启用 Stdout 日志重定向”下，选择“打开 Kudu 控制台以编辑 Web.Config”对应的按钮  。
1. 在 Kudu 诊断控制台中，打开路径“站点 > wwwroot”下的文件夹。 向下滚动以在列表底部显示“web.config”文件。
1. 单击“web.config”文件旁边的铅笔图标。
1. 将“stdoutLogEnabled”设置为 `true`，并将“stdoutLogFile”路径更改为 `\\?\%home%\LogFiles\stdout`。
1. 选择“保存”以保存已更新的 web.config 文件。
1. 向应用发出请求。
1. 返回到 Azure 门户。 选择“开发工具”区域中的“高级工具”边栏选项卡。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 选择“LogFiles”文件夹。
1. 检查“已修改”列并选择铅笔图标以编辑具有最新修改日期的 stdout 日志。
1. 打开日志文件后，将显示错误。

故障排除完成后，禁用 stdout 日志记录：

1. 在 Kudu 诊断控制台中，返回到路径“site > wwwroot”以显示 web.config 文件。 通过选择铅笔图标再次打开 web.config 文件。
1. 将“stdoutLogEnabled”设置为 `false`。
1. 选择“保存”以保存文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。 仅使用 stdout 日志记录来解决应用启动问题。
>
> 对于在 ASP.NET Core 应用启动后生成的常规日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="aspnet-core-module-debug-log-azure-app-service"></a>ASP.NET Core 模块调试日志（Azure 应用服务）

ASP.NET Core 模块调试日志从 ASP.NET Core 模块提供了更多、更详细的日志记录。 若要启用和查看 stdout 日志，请执行以下操作：

1. 要启用增强的诊断日志，请执行以下任一操作：
   * 按照[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)中的说明配置应用以获取增强的诊断日志记录。 重新部署应用。
   * 使用 Kudu 控制台将[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)中显示的 `<handlerSettings>` 添加到动态应用的 web.config 文件中：
     1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
     1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
     1. 打开路径“site > wwwroot”下的文件夹。 通过选择铅笔按钮编辑 web.config 文件。 添加 `<handlerSettings>` 部分（如[增强的诊断日志](xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs)中所示）。 选择“保存”按钮。
1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开路径“site > wwwroot”下的文件夹。 如果没有为 aspnetcore-debug.log 文件提供路径，则该文件将显示在列表中。 如果提供了路径，请导航到日志文件的位置。
1. 使用文件名旁边的铅笔按钮打开日志文件。

故障排除完成后，禁用调试日志记录：

要禁用增强的调试日志，请执行以下任一操作：

* 从本地删除 web.config 文件中的 `<handlerSettings>` 并重新部署该应用。
* 使用 Kudu 控制台编辑 web.config 文件并删除 `<handlerSettings>` 部分。 保存该文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>。

> [!WARNING]
> 无法禁用调试日志可能会导致应用或服务器出现故障。 日志文件大小没有任何限制。 仅使用调试日志记录来解决应用启动问题。
>
> 对于在 ASP.NET Core 应用启动后生成的常规日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="slow-or-hanging-app-azure-app-service"></a>应用缓慢或挂起（Azure 应用服务）

如果应用程序响应缓慢或在有请求时挂起，请参阅以下文章：

* [解决 Azure 应用服务中 Web 应用性能缓慢的问题](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [使用故障诊断程序站点扩展来捕获 Azure Web 应用程序上间歇性异常问题或性能问题的转储](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>监视边栏选项卡

监视边栏选项卡提供了本主题前面所述的方法的替代故障排除体验。 这些边栏选项卡可用于诊断 500 系列错误。

确认是否已安装 ASP.NET Core 扩展。 如果未安装扩展，请手动进行安装：

1. 在“开发工具”边栏选项卡部分中，选择“扩展”边栏选项卡。
1. “ASP.NET Core 扩展”应显示在列表中。
1. 如果未安装扩展，请选择“添加”按钮。
1. 从列表中选择“ASP.NET Core 扩展”。
1. 选择“确定”以接受法律条款。
1. 选择“添加扩展”边栏选项卡上的“确定”。
1. 信息性弹出消息指示成功安装扩展的时间。

如果未启用 stdout 日志记录，请执行以下步骤：

1. 在 Azure 门户中，选择“开发工具”区域中的“高级工具”边栏选项卡。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开路径“site>wwwroot”下的文件夹，然后向下滚动以显示列表底部的 web.config 文件 。
1. 单击“web.config”文件旁边的铅笔图标。
1. 将“stdoutLogEnabled”设置为 `true`，并将“stdoutLogFile”路径更改为 `\\?\%home%\LogFiles\stdout`。
1. 选择“保存”以保存已更新的 web.config 文件。

继续激活诊断日志记录：

1. 在 Azure 门户中，选择“诊断日志”边栏选项卡。
1. 选择“应用程序日志记录(文件系统)”和“详细错误消息”的“开”开关。 选择边栏选项卡顶部的“保存”按钮。
1. 若要包含失败请求跟踪（也称为失败请求事件缓冲 (FREB) 日志记录），请选择“失败请求跟踪”的“开”开关。
1. 选择“日志流”边栏选项卡，将在门户中的“诊断日志”边栏选项卡下立即列出。
1. 向应用发出请求。
1. 在日志流数据中，指示了错误的原因。

故障排除完成后，请务必禁用 stdout 日志记录。

若要查看失败请求跟踪日志（FREB 日志），请执行以下操作：

1. 在 Azure 门户中导航到“诊断并解决问题”边栏选项卡。
1. 从侧栏的“支持工具”区域中选择“失败请求跟踪日志”。

有关详细信息，请参阅[“在 Azure 应用服务中启用 Web 应用的诊断日志记录”主题的“失败请求跟踪”部分](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces)和 [Azure 中的 Web 应用的应用程序性能常见问题：如何打开失败请求跟踪？](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)。

有关详细信息，请参阅[在 Azure 应用服务中启用 Web 应用的诊断日志记录](/azure/app-service/web-sites-enable-diagnostic-log)。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。
>
> 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

## <a name="troubleshoot-on-iis"></a>IIS 疑难解答

### <a name="application-event-log-iis"></a>应用程序事件日志 (IIS)

访问应用程序事件日志：

1. 打开“开始”菜单，搜索“事件查看器”，然后选择“事件查看器”应用。
1. 在“事件查看器”中，打开“Windows 日志”节点。
1. 选择“应用程序”以打开应用程序事件日志。
1. 搜索与失败应用相关联的错误。 错误具有“源”列中“IIS AspNetCore 模块”或“IIS Express AspNetCore 模块”的值。

### <a name="run-the-app-at-a-command-prompt"></a>在命令提示符处运行应用

许多启动错误未在应用程序事件日志中生成有用信息。 可以通过在托管系统上在命令提示符处运行应用来找到某些错误的原因。

#### <a name="framework-dependent-deployment"></a>依赖框架的部署

如果应用是[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：

1. 在命令提示符处，导航到部署文件夹并通过使用 dotnet.exe 执行应用的程序集来运行应用。 在以下命令中，替换 \<assembly_name> 的应用程序集的名称：`dotnet .\<assembly_name>.dll`。
1. 来自应用且显示任何错误的控制台输出将写入控制台窗口。
1. 如果向应用发出请求时出现错误，请向 Kestrel 侦听所在的主机和端口发出请求。 如果使用默认主机和端口，请向 `http://localhost:5000/` 发出请求。 如果应用在 Kestrel 终结点地址处正常响应，则问题更可能与承载配置相关，而不太可能在于应用。

#### <a name="self-contained-deployment"></a>独立部署

如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：

1. 在命令提示符处，导航到部署文件夹并运行应用的可执行文件。 在以下命令中，替换 \<assembly_name> 的应用程序集的名称：`<assembly_name>.exe`。
1. 来自应用且显示任何错误的控制台输出将写入控制台窗口。
1. 如果向应用发出请求时出现错误，请向 Kestrel 侦听所在的主机和端口发出请求。 如果使用默认主机和端口，请向 `http://localhost:5000/` 发出请求。 如果应用在 Kestrel 终结点地址处正常响应，则问题更可能与承载配置相关，而不太可能在于应用。

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.NET Core 模块 stdout 日志 (IIS)

若要启用和查看 stdout 日志，请执行以下操作：

1. 在托管系统上导航到站点的部署文件夹。
1. 如果 logs 文件夹不存在，请创建该文件夹。 有关如何启用 MSBuild 以在部署中自动创建 logs 文件夹的说明，请参阅[目录结构](xref:host-and-deploy/directory-structure)主题。
1. 编辑 web.config 文件。 将“stdoutLogEnabled”设置为 `true` 并更改“stdoutLogFile”路径以指向 logs 文件夹（例如，`.\logs\stdout`）。 路径中的 `stdout` 是日志文件名的前缀。 创建日志时，将自动添加时间戳、进程 ID 和文件扩展名。 如果将 `stdout` 用作文件名的前缀，典型的日志文件将命名为“stdout_20180205184032_5412.log”。
1. 请确保应用程序池的标识具有对日志文件夹的写入权限。
1. 保存已更新的 web.config 文件。
1. 向应用发出请求。
1. 导航到 logs 文件夹。 查找并打开最新的 stdout 日志。
1. 研究日志以查找错误。

故障排除完成后，禁用 stdout 日志记录：

1. 编辑 web.config 文件。
1. 将“stdoutLogEnabled”设置为 `false`。
1. 保存该文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。
>
> 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="aspnet-core-module-debug-log-iis"></a>ASP.NET Core 模块调试日志 (IIS)

将以下处理程序设置添加到应用的 web.config 文件以启用 ASP.NET Core 模块调试日志：

```xml
<aspNetCore ...>
  <handlerSettings>
    <handlerSetting name="debugLevel" value="file" />
    <handlerSetting name="debugFile" value="c:\temp\ancm.log" />
  </handlerSettings>
</aspNetCore>
```

确认为日志指定的路径存在，并且应用池的标识具有该位置的写入权限。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#enhanced-diagnostic-logs>。

### <a name="enable-the-developer-exception-page"></a>启用开发人员异常页面

`ASPNETCORE_ENVIRONMENT` [环境变量可以添加到 web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) 以在开发环境中运行应用。 只要在应用启动时环境不被主机生成器上的 `UseEnvironment` 重写，设置环境变量就会在运行应用时显示“[开发人员异常页面](xref:fundamentals/error-handling)”。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="InProcess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

仅建议在未向 Internet 公开的暂存服务器和测试服务器上设置 `ASPNETCORE_ENVIRONMENT` 的环境变量。 在故障排除后从 web.config 文件中删除环境变量。 有关设置 web.config 中的环境变量的信息，请参阅 [aspNetCore 的 environmentVariables 子元素](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)。

### <a name="obtain-data-from-an-app"></a>从应用中获取数据

如果应用能够响应请求，则使用终端内联中间件从应用中获取请求、连接和其他数据。 有关详细信息和示例代码，请参阅<xref:test/troubleshoot#obtain-data-from-an-app>。

### <a name="slow-or-hanging-app-iis"></a>应用缓慢或挂起 (IIS)

故障转储是系统内存的一个快照，可帮助确定应用崩溃、启动故障或应用速度缓慢等状况的原因。

#### <a name="app-crashes-or-encounters-an-exception"></a>应用崩溃或引发异常

从 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) 中获取转储并进行分析：

1. 创建文件夹，将崩溃转储文件保存在 `c:\dumps`。 应用池必须对该文件夹具有写权限。
1. 运行 [EnableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1)：
   * 如果应用使用[进程内托管模型](xref:host-and-deploy/iis/index#in-process-hosting-model)，则请为 w3wp.exe 运行脚本：

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * 如果应用使用[进程外托管模型](xref:host-and-deploy/iis/index#out-of-process-hosting-model)，则请为 dotnet.exe 运行脚本：

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. 在造成崩溃的条件下运行应用。
1. 出现崩溃后，运行 [DisableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1)：
   * 如果应用使用[进程内托管模型](xref:host-and-deploy/iis/index#in-process-hosting-model)，则请为 w3wp.exe 运行脚本：

     ```console
     .\DisableDumps w3wp.exe
     ```

   * 如果应用使用[进程外托管模型](xref:host-and-deploy/iis/index#out-of-process-hosting-model)，则请为 dotnet.exe 运行脚本：

     ```console
     .\DisableDumps dotnet.exe
     ```

在应用崩溃并完成转储收集后，即可正常终止应用。 PowerShell 脚本会 WER 来按应用收集转储（最多收集 5 个）。

> [!WARNING]
> 崩溃转储可能会占用大量磁盘空间（每个最多占用数 GB）。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>应用挂起、在启动期间失败或正常运行

如果应用挂起（停止响应但不崩溃）、在启动期间失败或者正常运行*hangs*，请参阅[用户模式转储文件：选择最佳工具](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)，以选择适合用于生成转储的工具。

#### <a name="analyze-the-dump"></a>分析转储

可采用几种方法来分析转储。 有关详细信息，请参阅[分析用户模式转储文件](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)。

## <a name="clear-package-caches"></a>清除包缓存

正常运行的应用在开发计算机上升级 .NET Core SDK 或在应用内更改包版本后可能会立即出现故障。 在某些情况下，不同的包可能在执行主要升级时中断应用。 可以按照以下说明来修复其中大部分问题：

1. 删除 bin 和 obj 文件夹。
1. 通过从命令行界面执行 [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) 清除包缓存。

   清除包缓存还可通过使用 [nuget.exe](https://www.nuget.org/downloads) 工具并执行命令 `nuget locals all -clear` 来完成。 *nuget.exe* 不是与 Windows 桌面操作系统的捆绑安装，必须从 [NuGet 网站](https://www.nuget.org/downloads)中单独获取。

1. 还原并重新生成项目。
1. 在重新部署应用前，在服务器上删除部署文件夹中的所有文件。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure 文档

* [用于 ASP.NET Core 的 Application Insights](/azure/application-insights/app-insights-asp-net-core)
* [“使用 Visual Studio 对 Azure 应用服务中的 Web 应用进行故障排除”的“远程调试 Web 应用”一节](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Azure 应用服务诊断概述](/azure/app-service/app-service-diagnostics)
* [如何：在 Azure 应用服务中监视应用](/azure/app-service/web-sites-monitor)
* [使用 Visual Studio 对 Azure 应用服务中的 Web 应用进行故障排除](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [解决 Azure Web 应用中的“502 错误的网关”和“503 服务不可用”HTTP 错误](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [解决 Azure 应用服务中 Web 应用性能缓慢的问题](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Azure 中的 Web 应用的应用程序性能常见问题](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web 应用沙盒（应用服务运行时执行限制）](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday：Azure 应用服务诊断和疑难解答体验（12 分钟视频）](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio 文档

* [在 Azure 中的 IIS 上的 Visual Studio 2017 中远程调试 ASP.NET Core](/visualstudio/debugger/remote-debugging-azure)
* [在远程 IIS 计算机的 Visual Studio 2017 中远程调试 ASP.NET Core](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [了解如何使用 Visual Studio 进行调试](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Visual Studio Code 文档

* [使用 Visual Studio Code 进行调试](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

本文提供了有关常见应用启动错误的信息，以及在将应用部署到 Azure 应用服务或 IIS 时如何诊断错误的说明：

[应用启动错误](#app-startup-errors)  
介绍常见的启动 HTTP 状态代码方案。

[排查 Azure 应用服务中的问题](#troubleshoot-on-azure-app-service)  
为部署到 Azure 应用服务的应用提供疑难解答建议。

[IIS 疑难解答](#troubleshoot-on-iis)  
为部署到 IIS 或在本地 IIS Express 上运行的应用提供疑难解答建议。 本指南适用于 Windows Server 和 Windows 桌面部署。

[清除包缓存](#clear-package-caches)  
说明当执行重大升级或更改包版本时，不一致的包中断应用时应如何处理。

[其他资源](#additional-resources)  
列出其他疑难解答主题。

## <a name="app-startup-errors"></a>应用启动错误

在 Visual Studio 中，ASP.NET Core 项目默认为在调试期间进行 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 托管。 本地调试时出现的“502.5 - 进程失败”可以使用本主题中的建议进行诊断。

### <a name="40314-forbidden"></a>403.14 禁止

应用无法启动。 记录以下错误：

```
The Web server is configured to not list the contents of this directory.
```

此错误通常是由于托管系统上的部署中断引起的，包括以下任何情况：

* 该应用部署到托管系统上的错误文件夹。
* 部署过程未能将所有应用的文件和文件夹移到托管系统上的部署文件夹中。
* 部署中缺少 web.config 文件，或者 web.config 文件内容格式不正确 。

执行以下步骤：

1. 从托管系统上的部署文件夹中删除所有文件和文件夹。
1. 使用常规部署方法（如 Visual Studio、PowerShell 或手动部署）将应用“发布”文件夹的内容重新部署到托管系统：
   * 确认部署中存在 web.config 文件，并且其内容正确。
   * 在 Azure 应用服务上托管时，请确认该应用已部署到 `D:\home\site\wwwroot` 文件夹。
   * 当应用由 IIS 托管时，请确认应用已部署到“IIS 管理器”的“基本设置”中显示的 IIS 物理路径  。
1. 通过将托管系统上的部署与项目“发布”文件夹的内容进行比较，确认已部署应用的所有文件和文件夹。

有关已发布 ASP.NET Core 应用布局的详细信息，请参阅 <xref:host-and-deploy/directory-structure>。 有关 web.config 文件的详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#configuration-with-webconfig>。

### <a name="500-internal-server-error"></a>500 内部服务器错误

应用启动，但某个错误阻止了服务器完成请求。

在启动期间或在创建响应时，应用的代码内出现此错误。 响应可能不包含任何内容，或响应可能会在浏览器中显示为“500 内部服务器错误”。 应用程序事件日志通常表明应用正常启动。 从服务器的角度来看，这是正确的。 应用已启动，但无法生成有效的响应。 在服务器上的命令提示符下运行应用，或启用 ASP.NET Core 模块 stdout 日志来解决问题。

### <a name="5025-process-failure"></a>502.5 进程失败

工作进程失败。 应用不启动。

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)尝试启动工作进程，但启动失败。 进程启动失败的原因通常可通过“应用程序事件日志”和“ASP.NET Core 模块 stdout 日志”中的条目进行确定。

常见的失败情况是，由于目标 ASP.NET Core 共享框架版本不存在，因此应用配置错误。 检查目标计算机上安装的 ASP.NET Core 共享框架版本。 共享框架是安装在计算机上并由 `Microsoft.AspNetCore.App` 等元包引用的一组程序集（.dll 文件）。 元包引用可以指定所需的最低版本。 有关详细信息，请参阅[共享框架](https://natemcmaster.com/blog/2018/08/29/netcore-primitives-2/)。

托管或应用配置错误导致工作进程失败时，将返回“502.5 进程失败”错误页面：

### <a name="failed-to-start-application-errorcode-0x800700c1"></a>未能启动应用程序（错误代码“0x800700c1”）

```
EventID: 1010
Source: IIS AspNetCore Module V2
Failed to start application '/LM/W3SVC/6/ROOT/', ErrorCode '0x800700c1'.
```

应用未能启动，因为应用的程序集 (.dll) 无法加载。

当已发布的应用与 w3wp/iisexpress 进程之间的位数不匹配时，会出现此错误。

确认应用池的 32 位设置正确：

1. 在 IIS 管理器的“应用程序池”中选择应用池。
1. 在“操作”面板中的“编辑应用程序池”下选择“高级设置”。
1. 设置“启用 32 位应用程序”：
   * 如果部署 32 位 (x86) 应用，则将值设置为 `True`。
   * 如果部署 64 位 (x64) 应用，则将值设置为 `False`。

确认项目文件中的 `<Platform>` MSBuild 属性与应用的已发布位数之间没有冲突。

### <a name="connection-reset"></a>连接重置

如果在发送标头后出现错误，则服务器在出现错误时发送“500 内部服务器错误”已经太晚了。 通常在序列化响应的复杂对象期间出现错误时发生这种情况。 此类型的错误在客户端上显示为“连接重置”错误。 [应用程序日志记录](xref:fundamentals/logging/index)可以帮助解决这些类型的错误。

### <a name="default-startup-limits"></a>默认启动限制

[ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)的默认“startupTimeLimit”配置为 120 秒。 保留默认值时，在模块记录进程故障之前，可能最多需要两分钟来启动应用。 有关配置模块的信息，请参阅 [aspNetCore 元素的属性](xref:host-and-deploy/aspnet-core-module#attributes-of-the-aspnetcore-element)。

## <a name="troubleshoot-on-azure-app-service"></a>排查 Azure 应用服务中的问题

[!INCLUDE [Azure App Service Preview Notice](~/includes/azure-apps-preview-notice.md)]

### <a name="application-event-log-azure-app-service"></a>应用程序事件日志（Azure 应用服务）

若要访问应用程序事件日志，请在 Azure 门户中使用“诊断并解决问题”边栏选项卡：

1. 在 Azure 门户中打开“应用服务”中的应用。
1. 选择“诊断并解决问题”。
1. 选择“诊断工具”标题。
1. 在“支持工具”下，选择“应用程序事件”按钮 。
1. 检查“源”列中由 IIS AspNetCoreModule 或 IIS AspNetCoreModule V2条目提供的最新错误 。

使用“诊断并解决问题”边栏选项卡的替代方法是直接使用 [Kudu](https://github.com/projectkudu/kudu/wiki) 检查应用程序事件日志文件：

1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开 LogFiles 文件夹。
1. 选择 eventlog.xml 文件旁边的铅笔图标。
1. 检查日志。 滚动到日志底部以查看最新事件。

### <a name="run-the-app-in-the-kudu-console"></a>在 Kudu 控制台中运行应用

许多启动错误未在应用程序事件日志中生成有用信息。 可以在 [Kudu](https://github.com/projectkudu/kudu/wiki) 远程执行控制台中运行应用以发现错误：

1. 打开“开发工具”区域中的“高级工具” 。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。

#### <a name="test-a-32-bit-x86-app"></a>测试 32 位 (x86) 应用

**当前版本**

1. `cd d:\home\site\wwwroot`
1. 运行应用：
   * 如果应用是[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：

     ```dotnetcli
     dotnet .\{ASSEMBLY NAME}.dll
     ```

   * 如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：

     ```console
     {ASSEMBLY NAME}.exe
     ```

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

**在预览版上运行的依赖框架的部署**

必须安装 ASP.NET Core {VERSION} (x86) 运行时站点扩展。

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x32`（`{X.Y}` 是运行时版本）
1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

应用的控制台输出（显示任何错误）会传送到 Kudu 控制台。

#### <a name="test-a-64-bit-x64-app"></a>测试 64 位 (x64) 应用

**当前版本**

* 如果应用是 64 位 (x64) [依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：
  1. `cd D:\Program Files\dotnet`
  1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`
* 如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：
  1. `cd D:\home\site\wwwroot`
  1. 运行应用：`{ASSEMBLY NAME}.exe`

应用的控制台输出（显示任何错误）会传送到 Kudu 控制台。

**在预览版上运行的依赖框架的部署**

必须安装 ASP.NET Core {VERSION} (x64) 运行时站点扩展。

1. `cd D:\home\SiteExtensions\AspNetCoreRuntime.{X.Y}.x64`（`{X.Y}` 是运行时版本）
1. 运行应用：`dotnet \home\site\wwwroot\{ASSEMBLY NAME}.dll`

来自应用且显示任何错误的控制台输出将传送到 Kudu 控制台。

### <a name="aspnet-core-module-stdout-log-azure-app-service"></a>ASP.NET Core 模块 stdout 日志（Azure 应用服务）

ASP.NET Core 模块 stdout 日志通常记录应用程序事件日志中找不到的有用错误消息。 若要启用和查看 stdout 日志，请执行以下操作：

1. 在 Azure 门户中导航到“诊断并解决问题”边栏选项卡。
1. 在“选择问题类别”下，选择“Web 应用关闭”按钮。
1. 在“建议的解决方案”>“启用 Stdout 日志重定向”下，选择“打开 Kudu 控制台以编辑 Web.Config”对应的按钮  。
1. 在 Kudu 诊断控制台中，打开路径“站点 > wwwroot”下的文件夹。 向下滚动以在列表底部显示“web.config”文件。
1. 单击“web.config”文件旁边的铅笔图标。
1. 将“stdoutLogEnabled”设置为 `true`，并将“stdoutLogFile”路径更改为 `\\?\%home%\LogFiles\stdout`。
1. 选择“保存”以保存已更新的 web.config 文件。
1. 向应用发出请求。
1. 返回到 Azure 门户。 选择“开发工具”区域中的“高级工具”边栏选项卡。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 选择“LogFiles”文件夹。
1. 检查“已修改”列并选择铅笔图标以编辑具有最新修改日期的 stdout 日志。
1. 打开日志文件后，将显示错误。

故障排除完成后，禁用 stdout 日志记录：

1. 在 Kudu 诊断控制台中，返回到路径“site > wwwroot”以显示 web.config 文件。 通过选择铅笔图标再次打开 web.config 文件。
1. 将“stdoutLogEnabled”设置为 `false`。
1. 选择“保存”以保存文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。 仅使用 stdout 日志记录来解决应用启动问题。
>
> 对于在 ASP.NET Core 应用启动后生成的常规日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="slow-or-hanging-app-azure-app-service"></a>应用缓慢或挂起（Azure 应用服务）

如果应用程序响应缓慢或在有请求时挂起，请参阅以下文章：

* [解决 Azure 应用服务中 Web 应用性能缓慢的问题](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [使用故障诊断程序站点扩展来捕获 Azure Web 应用程序上间歇性异常问题或性能问题的转储](https://blogs.msdn.microsoft.com/asiatech/2015/12/28/use-crash-diagnoser-site-extension-to-capture-dump-for-intermittent-exception-issues-or-performance-issues-on-azure-web-app/)

### <a name="monitoring-blades"></a>监视边栏选项卡

监视边栏选项卡提供了本主题前面所述的方法的替代故障排除体验。 这些边栏选项卡可用于诊断 500 系列错误。

确认是否已安装 ASP.NET Core 扩展。 如果未安装扩展，请手动进行安装：

1. 在“开发工具”边栏选项卡部分中，选择“扩展”边栏选项卡。
1. “ASP.NET Core 扩展”应显示在列表中。
1. 如果未安装扩展，请选择“添加”按钮。
1. 从列表中选择“ASP.NET Core 扩展”。
1. 选择“确定”以接受法律条款。
1. 选择“添加扩展”边栏选项卡上的“确定”。
1. 信息性弹出消息指示成功安装扩展的时间。

如果未启用 stdout 日志记录，请执行以下步骤：

1. 在 Azure 门户中，选择“开发工具”区域中的“高级工具”边栏选项卡。 选择“转到&rarr;”按钮。 此时将在新的浏览器选项卡或窗口中打开 Kudu 控制台。
1. 使用页面顶部的导航栏，打开“调试控制台”并选择“CMD”。
1. 打开路径“site>wwwroot”下的文件夹，然后向下滚动以显示列表底部的 web.config 文件 。
1. 单击“web.config”文件旁边的铅笔图标。
1. 将“stdoutLogEnabled”设置为 `true`，并将“stdoutLogFile”路径更改为 `\\?\%home%\LogFiles\stdout`。
1. 选择“保存”以保存已更新的 web.config 文件。

继续激活诊断日志记录：

1. 在 Azure 门户中，选择“诊断日志”边栏选项卡。
1. 选择“应用程序日志记录(文件系统)”和“详细错误消息”的“开”开关。 选择边栏选项卡顶部的“保存”按钮。
1. 若要包含失败请求跟踪（也称为失败请求事件缓冲 (FREB) 日志记录），请选择“失败请求跟踪”的“开”开关。
1. 选择“日志流”边栏选项卡，将在门户中的“诊断日志”边栏选项卡下立即列出。
1. 向应用发出请求。
1. 在日志流数据中，指示了错误的原因。

故障排除完成后，请务必禁用 stdout 日志记录。

若要查看失败请求跟踪日志（FREB 日志），请执行以下操作：

1. 在 Azure 门户中导航到“诊断并解决问题”边栏选项卡。
1. 从侧栏的“支持工具”区域中选择“失败请求跟踪日志”。

有关详细信息，请参阅[“在 Azure 应用服务中启用 Web 应用的诊断日志记录”主题的“失败请求跟踪”部分](/azure/app-service/web-sites-enable-diagnostic-log#failed-request-traces)和 [Azure 中的 Web 应用的应用程序性能常见问题：如何打开失败请求跟踪？](/azure/app-service/app-service-web-availability-performance-application-issues-faq#how-do-i-turn-on-failed-request-tracing)。

有关详细信息，请参阅[在 Azure 应用服务中启用 Web 应用的诊断日志记录](/azure/app-service/web-sites-enable-diagnostic-log)。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。
>
> 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

## <a name="troubleshoot-on-iis"></a>IIS 疑难解答

### <a name="application-event-log-iis"></a>应用程序事件日志 (IIS)

访问应用程序事件日志：

1. 打开“开始”菜单，搜索“事件查看器”，然后选择“事件查看器”应用。
1. 在“事件查看器”中，打开“Windows 日志”节点。
1. 选择“应用程序”以打开应用程序事件日志。
1. 搜索与失败应用相关联的错误。 错误具有“源”列中“IIS AspNetCore 模块”或“IIS Express AspNetCore 模块”的值。

### <a name="run-the-app-at-a-command-prompt"></a>在命令提示符处运行应用

许多启动错误未在应用程序事件日志中生成有用信息。 可以通过在托管系统上在命令提示符处运行应用来找到某些错误的原因。

#### <a name="framework-dependent-deployment"></a>依赖框架的部署

如果应用是[依赖框架的部署](/dotnet/core/deploying/#framework-dependent-deployments-fdd)：

1. 在命令提示符处，导航到部署文件夹并通过使用 dotnet.exe 执行应用的程序集来运行应用。 在以下命令中，替换 \<assembly_name> 的应用程序集的名称：`dotnet .\<assembly_name>.dll`。
1. 来自应用且显示任何错误的控制台输出将写入控制台窗口。
1. 如果向应用发出请求时出现错误，请向 Kestrel 侦听所在的主机和端口发出请求。 如果使用默认主机和端口，请向 `http://localhost:5000/` 发出请求。 如果应用在 Kestrel 终结点地址处正常响应，则问题更可能与承载配置相关，而不太可能在于应用。

#### <a name="self-contained-deployment"></a>独立部署

如果应用是[独立部署](/dotnet/core/deploying/#self-contained-deployments-scd)：

1. 在命令提示符处，导航到部署文件夹并运行应用的可执行文件。 在以下命令中，替换 \<assembly_name> 的应用程序集的名称：`<assembly_name>.exe`。
1. 来自应用且显示任何错误的控制台输出将写入控制台窗口。
1. 如果向应用发出请求时出现错误，请向 Kestrel 侦听所在的主机和端口发出请求。 如果使用默认主机和端口，请向 `http://localhost:5000/` 发出请求。 如果应用在 Kestrel 终结点地址处正常响应，则问题更可能与承载配置相关，而不太可能在于应用。

### <a name="aspnet-core-module-stdout-log-iis"></a>ASP.NET Core 模块 stdout 日志 (IIS)

若要启用和查看 stdout 日志，请执行以下操作：

1. 在托管系统上导航到站点的部署文件夹。
1. 如果 logs 文件夹不存在，请创建该文件夹。 有关如何启用 MSBuild 以在部署中自动创建 logs 文件夹的说明，请参阅[目录结构](xref:host-and-deploy/directory-structure)主题。
1. 编辑 web.config 文件。 将“stdoutLogEnabled”设置为 `true` 并更改“stdoutLogFile”路径以指向 logs 文件夹（例如，`.\logs\stdout`）。 路径中的 `stdout` 是日志文件名的前缀。 创建日志时，将自动添加时间戳、进程 ID 和文件扩展名。 如果将 `stdout` 用作文件名的前缀，典型的日志文件将命名为“stdout_20180205184032_5412.log”。
1. 请确保应用程序池的标识具有对日志文件夹的写入权限。
1. 保存已更新的 web.config 文件。
1. 向应用发出请求。
1. 导航到 logs 文件夹。 查找并打开最新的 stdout 日志。
1. 研究日志以查找错误。

故障排除完成后，禁用 stdout 日志记录：

1. 编辑 web.config 文件。
1. 将“stdoutLogEnabled”设置为 `false`。
1. 保存该文件。

有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module#log-creation-and-redirection>。

> [!WARNING]
> 无法禁用 stdout 日志可能会导致应用或服务器出现故障。 日志文件大小或创建的日志文件数没有限制。
>
> 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

### <a name="enable-the-developer-exception-page"></a>启用开发人员异常页面

`ASPNETCORE_ENVIRONMENT` [环境变量可以添加到 web.config](xref:host-and-deploy/aspnet-core-module#setting-environment-variables) 以在开发环境中运行应用。 只要在应用启动时环境不被主机生成器上的 `UseEnvironment` 重写，设置环境变量就会在运行应用时显示“[开发人员异常页面](xref:fundamentals/error-handling)”。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
  </environmentVariables>
</aspNetCore>
```

仅建议在未向 Internet 公开的暂存服务器和测试服务器上设置 `ASPNETCORE_ENVIRONMENT` 的环境变量。 在故障排除后从 web.config 文件中删除环境变量。 有关设置 web.config 中的环境变量的信息，请参阅 [aspNetCore 的 environmentVariables 子元素](xref:host-and-deploy/aspnet-core-module#setting-environment-variables)。

### <a name="obtain-data-from-an-app"></a>从应用中获取数据

如果应用能够响应请求，则使用终端内联中间件从应用中获取请求、连接和其他数据。 有关详细信息和示例代码，请参阅<xref:test/troubleshoot#obtain-data-from-an-app>。

### <a name="slow-or-hanging-app-iis"></a>应用缓慢或挂起 (IIS)

故障转储是系统内存的一个快照，可帮助确定应用崩溃、启动故障或应用速度缓慢等状况的原因。

#### <a name="app-crashes-or-encounters-an-exception"></a>应用崩溃或引发异常

从 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) 中获取转储并进行分析：

1. 创建文件夹，将崩溃转储文件保存在 `c:\dumps`。 应用池必须对该文件夹具有写权限。
1. 运行 [EnableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/EnableDumps.ps1)：
   * 如果应用使用[进程内托管模型](xref:host-and-deploy/iis/index#in-process-hosting-model)，则请为 w3wp.exe 运行脚本：

     ```console
     .\EnableDumps w3wp.exe c:\dumps
     ```

   * 如果应用使用[进程外托管模型](xref:host-and-deploy/iis/index#out-of-process-hosting-model)，则请为 dotnet.exe 运行脚本：

     ```console
     .\EnableDumps dotnet.exe c:\dumps
     ```

1. 在造成崩溃的条件下运行应用。
1. 出现崩溃后，运行 [DisableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/test/troubleshoot-azure-iis/scripts/DisableDumps.ps1)：
   * 如果应用使用[进程内托管模型](xref:host-and-deploy/iis/index#in-process-hosting-model)，则请为 w3wp.exe 运行脚本：

     ```console
     .\DisableDumps w3wp.exe
     ```

   * 如果应用使用[进程外托管模型](xref:host-and-deploy/iis/index#out-of-process-hosting-model)，则请为 dotnet.exe 运行脚本：

     ```console
     .\DisableDumps dotnet.exe
     ```

在应用崩溃并完成转储收集后，即可正常终止应用。 PowerShell 脚本会 WER 来按应用收集转储（最多收集 5 个）。

> [!WARNING]
> 崩溃转储可能会占用大量磁盘空间（每个最多占用数 GB）。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>应用挂起、在启动期间失败或正常运行

如果应用挂起（停止响应但不崩溃）、在启动期间失败或者正常运行*hangs*，请参阅[用户模式转储文件：选择最佳工具](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)，以选择适合用于生成转储的工具。

#### <a name="analyze-the-dump"></a>分析转储

可采用几种方法来分析转储。 有关详细信息，请参阅[分析用户模式转储文件](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)。

## <a name="clear-package-caches"></a>清除包缓存

正常运行的应用在开发计算机上升级 .NET Core SDK 或在应用内更改包版本后可能会立即出现故障。 在某些情况下，不同的包可能在执行主要升级时中断应用。 可以按照以下说明来修复其中大部分问题：

1. 删除 bin 和 obj 文件夹。
1. 通过从命令行界面执行 [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) 清除包缓存。

   清除包缓存还可通过使用 [nuget.exe](https://www.nuget.org/downloads) 工具并执行命令 `nuget locals all -clear` 来完成。 *nuget.exe* 不是与 Windows 桌面操作系统的捆绑安装，必须从 [NuGet 网站](https://www.nuget.org/downloads)中单独获取。

1. 还原并重新生成项目。
1. 在重新部署应用前，在服务器上删除部署文件夹中的所有文件。

## <a name="additional-resources"></a>其他资源

* <xref:test/troubleshoot>
* <xref:host-and-deploy/azure-iis-errors-reference>
* <xref:fundamentals/error-handling>
* <xref:host-and-deploy/aspnet-core-module>

### <a name="azure-documentation"></a>Azure 文档

* [用于 ASP.NET Core 的 Application Insights](/azure/application-insights/app-insights-asp-net-core)
* [“使用 Visual Studio 对 Azure 应用服务中的 Web 应用进行故障排除”的“远程调试 Web 应用”一节](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio#remotedebug)
* [Azure 应用服务诊断概述](/azure/app-service/app-service-diagnostics)
* [如何：在 Azure 应用服务中监视应用](/azure/app-service/web-sites-monitor)
* [使用 Visual Studio 对 Azure 应用服务中的 Web 应用进行故障排除](/azure/app-service/web-sites-dotnet-troubleshoot-visual-studio)
* [解决 Azure Web 应用中的“502 错误的网关”和“503 服务不可用”HTTP 错误](/azure/app-service/app-service-web-troubleshoot-http-502-http-503)
* [解决 Azure 应用服务中 Web 应用性能缓慢的问题](/azure/app-service/app-service-web-troubleshoot-performance-degradation)
* [Azure 中的 Web 应用的应用程序性能常见问题](/azure/app-service/app-service-web-availability-performance-application-issues-faq)
* [Azure Web 应用沙盒（应用服务运行时执行限制）](https://github.com/projectkudu/kudu/wiki/Azure-Web-App-sandbox)
* [Azure Friday：Azure 应用服务诊断和疑难解答体验（12 分钟视频）](https://channel9.msdn.com/Shows/Azure-Friday/Azure-App-Service-Diagnostic-and-Troubleshooting-Experience)

### <a name="visual-studio-documentation"></a>Visual Studio 文档

* [在 Azure 中的 IIS 上的 Visual Studio 2017 中远程调试 ASP.NET Core](/visualstudio/debugger/remote-debugging-azure)
* [在远程 IIS 计算机的 Visual Studio 2017 中远程调试 ASP.NET Core](/visualstudio/debugger/remote-debugging-aspnet-on-a-remote-iis-computer)
* [了解如何使用 Visual Studio 进行调试](/visualstudio/debugger/getting-started-with-the-debugger)

### <a name="visual-studio-code-documentation"></a>Visual Studio Code 文档

* [使用 Visual Studio Code 进行调试](https://code.visualstudio.com/docs/editor/debugging)

::: moniker-end
