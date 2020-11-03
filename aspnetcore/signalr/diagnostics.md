---
title: ASP.NET Core 中的日志记录和诊断 SignalR
author: anurse
description: 了解如何从 ASP.NET Core 应用收集诊断信息 SignalR 。
monikerRange: '>= aspnetcore-2.1'
ms.author: anurse
ms.custom: devx-track-csharp, signalr, devx-track-js
ms.date: 06/12/2020
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
uid: signalr/diagnostics
ms.openlocfilehash: 6e5e9d866a1e03e69856cc63dcfe30284048dd6d
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061309"
---
# <a name="logging-and-diagnostics-in-aspnet-core-no-locsignalr"></a>ASP.NET Core 中的日志记录和诊断 SignalR

作者： [Andrew Stanton](https://twitter.com/anurse)

本文提供了从 ASP.NET Core 应用收集诊断 SignalR 信息以帮助解决问题的指南。

## <a name="server-side-logging"></a>服务器端日志记录

> [!WARNING]
> 服务器端日志可能包含来自应用的敏感信息。 切勿将来自生产应用的原始日志发布到 GitHub 等公共论坛。

由于 SignalR 是 ASP.NET Core 的一部分，因此它使用 ASP.NET Core 日志记录系统。 在默认配置中，只 SignalR 记录很少信息，但可以配置这种信息。 有关配置 ASP.NET Core 日志记录的详细信息，请参阅 [ASP.NET Core 日志记录](xref:fundamentals/logging/index#configuration)上的文档。

SignalR 使用两个记录器类别：

* `Microsoft.AspNetCore.SignalR`：用于与集线器协议相关的日志、激活集线器、调用方法以及其他与中心相关的活动。
* `Microsoft.AspNetCore.Http.Connections`：用于与传输相关的日志，例如 Websocket、长轮询、Server-Sent 事件和低级别 SignalR 基础结构。

若要从中启用详细日志 SignalR ，请 `Debug` *appsettings.json* 通过将以下项添加到 `LogLevel` 中的子部分，将上述两个前缀配置为文件中的级别 `Logging` ：

[!code-json[](diagnostics/logging-config.json?highlight=7-8)]

你还可以在方法的代码中对此进行配置 `CreateWebHostBuilder` ：

[!code-csharp[](diagnostics/logging-config-code.cs?highlight=5-6)]

如果不使用基于 JSON 的配置，请在配置系统中设置以下配置值：

* `Logging:LogLevel:Microsoft.AspNetCore.SignalR` = `Debug`
* `Logging:LogLevel:Microsoft.AspNetCore.Http.Connections` = `Debug`

查看配置系统的文档以确定如何指定嵌套的配置值。 例如，使用环境变量时，使用两个 `_` 而不是 `:` 字符（例如 `Logging__LogLevel__Microsoft.AspNetCore.SignalR`）。

建议在为应用收集更详细的诊断时使用 `Debug` 级别。 `Trace` 级别产生的诊断级别很低，且很少需要它来诊断应用中的问题。

## <a name="access-server-side-logs"></a>访问服务器端日志

访问服务器端日志的方式取决于在其中运行的环境。

### <a name="as-a-console-app-outside-iis"></a>作为 IIS 外部的控制台应用

如果在控制台应用中运行，则默认情况下应启用[控制台记录器](xref:fundamentals/logging/index#console)。 SignalR 日志将显示在控制台中。

### <a name="within-iis-express-from-visual-studio"></a>在 Visual Studio IIS Express 中

Visual Studio 会在 " **输出** " 窗口中显示日志输出。 选择 **ASP.NET Core Web 服务器** "下拉选项。

### <a name="azure-app-service"></a>Azure 应用服务

在 Azure App Service 门户的 " **诊断日志** " 部分中，启用 **应用程序日志记录 (Filesystem)** 选项，并将 **级别** 配置为 `Verbose` 。 日志 **流** 服务和应用服务文件系统的日志中应提供日志。 有关详细信息，请参阅 [Azure 日志流式处理](xref:fundamentals/logging/index#azure-log-streaming)。

### <a name="other-environments"></a>其他环境

如果将应用部署到另一个环境（例如 Docker、Kubernetes 或 Windows 服务），请参阅 <xref:fundamentals/logging/index>，了解有关如何配置适用于环境的日志记录提供程序的详细信息。

## <a name="javascript-client-logging"></a>JavaScript 客户端日志记录

> [!WARNING]
> 客户端日志可能包含来自应用的敏感信息。 切勿将来自生产应用的原始日志发布到 GitHub 等公共论坛。

使用 JavaScript 客户端时，可以使用中的方法配置日志记录选项 `configureLogging` `HubConnectionBuilder` ：

[!code-javascript[](diagnostics/logging-config-js.js?highlight=3)]

若要完全禁用日志记录，请 `signalR.LogLevel.None` 在方法中指定 `configureLogging` 。

下表显示了可用于 JavaScript 客户端的日志级别。 将日志级别设置为这些值之一，可以在表中对该级别和其之上的所有级别进行日志记录。

| Level | 说明 |
| ----- | ----------- |
| `None` | 不记录任何消息。 |
| `Critical` | 指示整个应用程序中的失败的消息。 |
| `Error` | 指示当前操作失败的消息。 |
| `Warning` | 指示非严重问题的消息。 |
| `Information` | 信息性消息。 |
| `Debug` | 诊断消息对于调试很有用。 |
| `Trace` | 旨在诊断特定问题的详细诊断消息。 |

配置详细级别后，日志将写入浏览器控制台 (或) 中的 NodeJS 应用程序中的标准输出。

如果要将日志发送到自定义日志记录系统，可以提供实现接口的 JavaScript 对象 `ILogger` 。 唯一需要实现的方法是 `log` ，它将使用事件的级别和与事件关联的消息。 例如：

[!code-typescript[](diagnostics/custom-logger.ts?highlight=3-7,13)]

## <a name="net-client-logging"></a> .NET 客户端日志记录

> [!WARNING]
> 客户端日志可能包含来自应用的敏感信息。 切勿将来自生产应用的原始日志发布到 GitHub 等公共论坛。

若要从 .NET 客户端获取日志，可以对使用 `ConfigureLogging` 方法 `HubConnectionBuilder` 。 这与和上的方法的工作方式相同 `ConfigureLogging` `WebHostBuilder` `HostBuilder` 。 你可以配置 ASP.NET Core 中使用的相同日志记录提供程序。 但是，您必须为单独的日志提供程序手动安装和启用 NuGet 包。

若要将 .NET 客户端日志记录添加到 Blazor WebAssembly 应用程序，请参阅 <xref:blazor/fundamentals/logging#blazor-webassembly-signalr-net-client-logging> 。

### <a name="console-logging"></a>控制台日志记录

若要启用控制台日志记录，请添加[""。](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Console) 然后，使用 `AddConsole` 方法来配置控制台记录器：

[!code-csharp[](diagnostics/net-client-console-log.cs?highlight=6)]

### <a name="debug-output-window-logging"></a>调试输出窗口日志记录

还可以配置日志，以便在 Visual Studio 中切换到 " **输出** " 窗口。 安装 " ["，](https://www.nuget.org/packages/Microsoft.Extensions.Logging.Debug) 然后使用 `AddDebug` 方法：

[!code-csharp[](diagnostics/net-client-debug-log.cs?highlight=6)]

### <a name="other-logging-providers"></a>其他日志记录提供程序

SignalR 支持其他日志记录提供程序，如 Serilog、Seq、NLog 或与集成的任何其他日志记录系统 `Microsoft.Extensions.Logging` 。 如果日志记录系统提供 `ILoggerProvider` ，则可以将其注册到 `AddProvider` ：

[!code-csharp[](diagnostics/net-client-custom-log.cs?highlight=6)]

### <a name="control-verbosity"></a>控件详细级别

如果要从应用中的其他位置进行日志记录，则将默认级别更改为 `Debug` 可能太详细。 您可以使用筛选器来配置日志的日志记录级别 SignalR 。 可以在代码中完成此操作，其方式与在服务器上的操作大致相同：

[!code-csharp[Controlling verbosity in .NET client](diagnostics/logging-config-client-code.cs?highlight=9-10)]

## <a name="network-traces"></a>网络跟踪

> [!WARNING]
> 网络跟踪包含应用发送的每个消息的全部内容。 **切勿** 将原始网络跟踪从生产应用发布到 GitHub 等公共论坛。

如果遇到问题，网络跟踪有时可以提供很多有用的信息。 如果要在我们的问题跟踪程序上发布问题，此方法特别有用。

## <a name="collect-a-network-trace-with-fiddler-preferred-option"></a>使用 Fiddler (首选选项收集网络跟踪) 

此方法适用于所有应用。

Fiddler 是一个非常强大的工具，用于收集 HTTP 跟踪。 从 [telerik.com/fiddler](https://www.telerik.com/fiddler)安装它，启动它，然后运行你的应用程序并重现此问题。 Fiddler 适用于 Windows，并且有适用于 macOS 和 Linux 的 beta 版本。

如果使用 HTTPS 进行连接，则需要执行一些额外的步骤来确保 Fiddler 可以解密 HTTPS 流量。 有关更多详细信息，请参阅 [Fiddler 文档](https://docs.telerik.com/fiddler/Configure-Fiddler/Tasks/DecryptHTTPS)。

收集跟踪后，可以通过从菜单栏中选择 "文件" **File**  >  " **保存**  >  **所有会话** " 来导出跟踪。

![正在从 Fiddler 导出所有会话](diagnostics/fiddler-export.png)

## <a name="collect-a-network-trace-with-tcpdump-macos-and-linux-only"></a>仅使用 tcpdump (macOS 和 Linux 收集网络跟踪) 

此方法适用于所有应用。

可以通过在命令行界面中运行以下命令，使用 tcpdump 收集原始 TCP 跟踪。 如果出现权限错误，你可能需要将 `root` 命令作为或的前缀 `sudo` ：

```console
tcpdump -i [interface] -w trace.pcap
```

将替换 `[interface]` 为要捕获的网络接口。 通常，这种情况类似于 `/dev/eth0` 标准以太网接口的 () 或 `/dev/lo0` (localhost 流量) 。 有关详细信息，请参阅 `tcpdump` 主机系统上的手册页。

## <a name="collect-a-network-trace-in-the-browser"></a>在浏览器中收集网络跟踪

此方法仅适用于基于浏览器的应用。

大多数浏览器开发人员工具都有一个 "网络" 选项卡，该选项卡允许您捕获浏览器和服务器之间的网络活动。 但是，这些跟踪不包括 WebSocket 和 Server-Sent 事件消息。 如果使用的是这些传输，请使用) 下面所述的工具（如 Fiddler 或 TcpDump） (更好的方法。

### <a name="microsoft-edge-and-internet-explorer"></a>Microsoft Edge 和 Internet Explorer

 (边缘和 Internet Explorer 的说明相同) 

1. 按 F12 打开开发工具
2. 单击 "网络" 选项卡
3. 如果需要，请刷新页面 () 并重现此问题
4. 单击工具栏中的 "保存" 图标，将跟踪作为 "HAR" 文件导出：

![Microsoft Edge 开发工具网络选项卡上的 "保存" 图标](diagnostics/ie-edge-har-export.png)

### <a name="google-chrome"></a>Google Chrome

1. 按 F12 打开开发工具
2. 单击 "网络" 选项卡
3. 如果需要，请刷新页面 () 并重现此问题
4. 右键单击请求列表中的任意位置，然后选择 "另存为包含内容的 HAR"：

![Google Chrome 开发工具网络选项卡中的 "另存为 HAR 与内容" 选项](diagnostics/chrome-har-export.png)

### <a name="mozilla-firefox"></a>Mozilla Firefox

1. 按 F12 打开开发工具
2. 单击 "网络" 选项卡
3. 如果需要，请刷新页面 () 并重现此问题
4. 右键单击请求列表中的任意位置，然后选择 "全部保存为 HAR"

![Mozilla Firefox 开发工具网络选项卡中的 "全部保存为 HAR" 选项](diagnostics/firefox-har-export.png)

## <a name="attach-diagnostics-files-to-github-issues"></a>将诊断文件附加到 GitHub 问题

可以通过对其进行重命名，将诊断文件附加到 GitHub 问题，以便它们具有一个 `.txt` 扩展，然后将其拖放到该问题上。

> [!NOTE]
> 请不要将日志文件或网络跟踪的内容粘贴到 GitHub 问题中。 这些日志和跟踪可能会很大，GitHub 通常会将其截断。

![将日志文件拖到 GitHub 问题上](diagnostics/attaching-diagnostics-files.png)

## <a name="metrics"></a>指标

度量值是基于时间间隔的数据度量值的表示形式。 例如，每秒请求数。 指标数据允许以较高的级别观察应用的状态。 .NET gRPC 指标是使用 <xref:System.Diagnostics.Tracing.EventCounter> 发出的。

### <a name="no-locsignalr-server-metrics"></a>SignalR 服务器指标

SignalR 在事件源上报告服务器指标 <xref:Microsoft.AspNetCore.Http.Connections> 。

| “属性”                    | 说明                 |
|-------------------------|-----------------------------|
| `connections-started`   | 已启动的连接总数   |
| `connections-stopped`   | 已停止的连接总数   |
| `connections-timed-out` | 总连接超时 |
| `current-connections`   | 当前连接数         |
| `connections-duration`  | 平均连接持续时间 |

### <a name="observe-metrics"></a>观察指标

[dotnet-counters](/dotnet/core/diagnostics/dotnet-counters) 是一个性能监视工具，用于临时运行状况监视和初级性能调查。 使用 `Microsoft.AspNetCore.Http.Connections` 作为提供程序名称监视 .net 应用程序。 例如：

```console
> dotnet-counters monitor --process-id 37016 Microsoft.AspNetCore.Http.Connections

Press p to pause, r to resume, q to quit.
    Status: Running
[Microsoft.AspNetCore.Http.Connections]
    Average Connection Duration (ms)       16,040.56
    Current Connections                         1
    Total Connections Started                   8
    Total Connections Stopped                   7
    Total Connections Timed Out                 0
```

## <a name="additional-resources"></a>其他资源

* <xref:signalr/configuration>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
