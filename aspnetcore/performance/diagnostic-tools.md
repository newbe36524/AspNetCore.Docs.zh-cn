---
title: 性能诊断工具
author: mjrousos
description: 用于诊断 ASP.NET Core 应用程序中的性能问题的有用工具。
monikerRange: '>= aspnetcore-1.1'
ms.author: riande
ms.date: 04/11/2019
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
uid: performance/diagnostic-tools
ms.openlocfilehash: 71386de232265840f9b825bd33d23bb1d218efc6
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060854"
---
# <a name="performance-diagnostic-tools"></a>性能诊断工具

作者：[Mike Rousos](https://github.com/mjrousos)

本文列出了用于诊断 ASP.NET Core 中性能问题的工具。

## <a name="visual-studio-diagnostic-tools"></a>Visual Studio 诊断工具

Visual Studio 中内置的 [分析和诊断工具](/visualstudio/profiling) 是开始调查性能问题的好地方。 这些工具的功能非常强大，可以方便地从 Visual Studio 开发环境使用。 该工具允许分析 ASP.NET Core 应用中的 CPU 使用率、内存使用率和性能事件。 内置使分析变得非常简单。

有关详细信息，请 [参阅 Visual Studio 文档](/visualstudio/profiling/profiling-overview)。

## <a name="application-insights"></a>Application Insights

[Application Insights](/azure/application-insights/app-insights-overview) 提供应用程序的详细性能数据。 Application Insights 会自动收集有关响应速率、失败率、依赖项响应时间等的数据。 Application Insights 支持记录特定于应用的自定义事件和指标。

Azure 应用程序 Insights 提供多种方法来提供对受监视应用的见解：

- [应用程序映射](/azure/application-insights/app-insights-app-map) -有助于在分布式应用程序的所有组件中发现性能瓶颈或故障热点。
- [Azure 指标资源管理器](/azure/azure-monitor/platform/metrics-getting-started) 是 Microsoft Azure 门户的一个组件，它允许绘制图表、直观关联趋势，并调查指标值中的峰值和 dip。
- [Application Insights 门户中的 "性能" 边栏选项卡](/azure/application-insights/app-insights-tutorial-performance)：

  - 显示监视的应用程序中不同操作的性能详细信息。
  - 允许钻取到单个操作，以检查影响长时间的所有部件/依赖关系。
  - 可以从源调用探查器，按需收集性能跟踪。

- [Azure 应用程序 Insights 探查器](/azure/azure-monitor/app/profiler) 允许 .net 应用的常规和按需分析。  Azure 门户显示通过调用堆栈和热路径捕获的性能跟踪。 还可以使用 PerfView 下载跟踪文件以进行更深入的分析。

Application Insights 可用于各种环境：

- 经过优化，可在 Azure 中工作。
- 在生产、开发和过渡环境中工作。
- 从 [Visual Studio](/azure/application-insights/app-insights-visual-studio) 或其他宿主环境中运行。

有关详细信息，请参阅[适用于 ASP.NET Core 的 Application Insights](/azure/application-insights/app-insights-asp-net-core)。

## <a name="perfview"></a>PerfView

[PerfView](https://github.com/Microsoft/perfview) 是 .net 团队为诊断 .net 性能问题而创建的一种性能分析工具。 PerfView 允许对 CPU 使用情况、内存和 GC 行为、性能事件和时钟时间进行分析。

你可以了解有关 PerfView 的详细信息以及如何开始使用 [PerfView 视频教程](https://channel9.msdn.com/Series/PerfView-Tutorial) ，或阅读该工具或 [GitHub 上](https://github.com/Microsoft/perfview)提供的用户指南。

## <a name="windows-performance-toolkit"></a>Windows 性能工具包

[Windows 性能工具包](/windows-hardware/test/wpt/) (WPT) 包含两个组件： Windows 性能记录器 (") 和 Windows 性能分析器 (WPA) 。 这些工具会生成 Windows 操作系统和应用的深入性能配置文件。 WPT 提供了更丰富的数据可视化方法，但其数据收集功能不如 PerfView。

## <a name="perfcollect"></a>PerfCollect

尽管 PerfView 是适用于 .NET 方案的有用性能分析工具，但它仅在 Windows 上运行，因此不能使用它从 Linux 环境中运行的 ASP.NET Core 应用程序收集跟踪。

[PerfCollect](https://github.com/dotnet/coreclr/blob/master/Documentation/project-docs/linux-performance-tracing.md) 是一种 bash 脚本，它使用本机 Linux 分析工具 ([Perf](https://perf.wiki.kernel.org/index.php/Main_Page) 和 [LTTng](https://lttng.org/)) 收集 Linux 上可通过 PerfView 分析的跟踪。 当在不能直接使用 PerfView 的 Linux 环境中出现性能问题时，PerfCollect 非常有用。 相反，PerfCollect 可以从 .NET Core 应用收集跟踪，然后使用 PerfView 在 Windows 计算机上进行分析。

[GitHub 上](https://github.com/dotnet/coreclr/blob/master/Documentation/project-docs/linux-performance-tracing.md)提供了有关如何安装和开始使用 PerfCollect 的详细信息。

## <a name="other-third-party-performance-tools"></a>其他第三方性能工具

下面列出了一些可在 .NET Core 应用程序的性能调查中发挥作用的第三方性能工具。

- [MiniProfiler](https://miniprofiler.com/)
- [JetBrains](https://www.jetbrains.com/)中的[dotTrace](https://www.jetbrains.com/profiler/)和[dotMemory](https://www.jetbrains.com/dotmemory/)
- 来自 Intel 的[VTune](https://software.intel.com/content/www/us/en/develop/tools/vtune-profiler.html)
