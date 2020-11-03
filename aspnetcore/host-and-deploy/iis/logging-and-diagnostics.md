---
title: ASP.NET Core 模块的日志创建和重定向
author: rick-anderson
description: 配置 IIS 和 ASP.NET Core 模块以捕获日志和诊断信息。
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
uid: host-and-deploy/iis/logging-and-diagnostics
ms.openlocfilehash: 523eec53d7d21723dcf136c4e5ce299533a78cc6
ms.sourcegitcommit: daa9ccf580df531254da9dce8593441ac963c674
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91901020"
---
# <a name="log-creation-and-redirection"></a>日志创建和重定向

如果已设置 `aspNetCore` 元素的 `stdoutLogEnabled` 和 `stdoutLogFile` 属性，ASP.NET Core 模块会将 stdout 和 stderr 控制台输出重定向到磁盘。 创建日志文件时，模块会创建 `stdoutLogFile` 路径中提供的所有文件夹。 应用池必须对写入日志的位置具有写入权限（使用 `IIS AppPool\{APP POOL NAME}` 提供写入权限，其中占位符 `{APP POOL NAME}` 为应用池名称）。

日志不会旋转，除非回收/重新启动进程。 宿主负责限制日志占用的磁盘空间。

仅当在 IIS 上托管或使用 [Visual Studio 中针对 IIS 的开发时支持](xref:host-and-deploy/iis/development-time-iis-support)时，而不是在本地调试和使用 IIS Express 运行应用时，才建议使用 stdout 日志来排查应用程序启动问题。

不要将 stdout 日志用于常规应用日志记录。 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

创建日志文件时，将自动添加时间戳和文件扩展名。 日志文件名是通过将时间戳、进程 ID 和文件扩展名 (`.log`) 附加到由下划线分隔的 `stdoutLogFile` 路径的最后一段（通常为 `stdout`）组合而成的。 如果 `stdoutLogFile` 路径以 `stdout` 结尾，则 PID 为 1934 且于 2018 年 2 月 5 日 19:42:32 创建的应用的日志的文件名为 `stdout_20180205194132_1934.log`。

如果 `stdoutLogEnabled` 为 false，则会捕获应用启动时发生的错误，并将其发送到事件日志，最大为 30 KB。 启动后，将丢弃所有其他日志。

以下示例 `aspNetCore` 元素在相对路径 `.\log\` 上配置 stdout 日志记录。 确认应用池用户标识是否已提供写入路径的权限。

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout"
    hostingModel="inprocess">
</aspNetCore>
```

发布用于 Azure 应用服务部署的应用时，Web SDK 会将 `stdoutLogFile` 值设置为 `\\?\%home%\LogFiles\stdout`。 已为 Azure 应用服务托管的应用预定义了 `%home` 环境变量。

要创建日志记录筛选器规则，请参阅 ASP.NET Core 日志记录文档的[配置](xref:fundamentals/logging/index#log-filtering)和[日志筛选](xref:fundamentals/logging/index#log-filtering)部分。

有关路径格式的详细信息，请参阅 [Windows 系统上的文件路径格式](/dotnet/standard/io/file-path-formats)。

## <a name="enhanced-diagnostic-logs"></a>增强的诊断日志

可以配置 ASP.NET Core 模块，以提供增强的诊断日志。 将 `<handlerSettings>` 元素添加到 `web.config` 中的 `<aspNetCore>` 元素。 将 `debugLevel` 设置为 `TRACE` 将提供更准确的诊断信息：

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="false"
    stdoutLogFile="\\?\%home%\LogFiles\stdout"
    hostingModel="inprocess">
  <handlerSettings>
    <handlerSetting name="debugFile" value=".\logs\aspnetcore-debug.log" />
    <handlerSetting name="debugLevel" value="FILE,TRACE" />
  </handlerSettings>
</aspNetCore>
```

创建日志文件时，路径中的任何文件夹（上述示例中为 `logs`）由模块创建。 应用池必须对写入日志的位置具有写入权限（使用 `IIS AppPool\{APP POOL NAME}` 提供写入权限，其中占位符 `{APP POOL NAME}` 为应用池名称）。

调试级别 (`debugLevel`) 值可以同时包含级别和位置。

级别（详细程度由低到高）：

* 错误
* 警告
* INFO
* TRACE

位置（允许多个位置）：

* CONSOLE
* 事件日志
* 文件

还可以通过环境变量提供处理程序设置：

* `ASPNETCORE_MODULE_DEBUG_FILE`：调试日志文件的路径。 （默认值：`aspnetcore-debug.log`）
* `ASPNETCORE_MODULE_DEBUG`：调试级别设置。

> [!WARNING]
> 部署中启用的调试日志记录的时间不要超出排除故障所需的时间。 日志大小不限。 持续启用调试日志可能耗尽可用磁盘空间并导致服务器或应用服务崩溃。

有关 `web.config` 文件中的 `aspNetCore` 元素的示例，请参阅 [`web.config` 的 ASP.NET Core 模块配置](xref:host-and-deploy/iis/web-config#configuration-of-aspnet-core-module-with-webconfig)。
