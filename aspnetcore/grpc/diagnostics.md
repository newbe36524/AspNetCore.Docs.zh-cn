---
title: .NET 上 gRPC 中的日志记录和诊断
author: jamesnk
description: 了解如何从 .NET 上的 gRPC 应用收集诊断。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 09/23/2019
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
uid: grpc/diagnostics
ms.openlocfilehash: 7d2da20d04b93ebcd16fb58a4b74b5b67d37bd72
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722918"
---
# <a name="logging-and-diagnostics-in-grpc-on-net"></a>.NET 上 gRPC 中的日志记录和诊断

作者：[James Newton-King](https://twitter.com/jamesnk)

本文提供从 gRPC 应用收集诊断以帮助解决问题的指南。 涵盖的主题包括：

* **日志记录** - 写入 [.NET Core 日志记录](xref:fundamentals/logging/index)的结​​构化日志。 应用框架使用 <xref:Microsoft.Extensions.Logging.ILogger> 来编写日志，用户使用它在应用中进行用户自己的日志记录。
* **跟踪** - 与使用 `DiaganosticSource` 和 `Activity` 进行编写的操作相关的事件。 来自诊断源的跟踪通常用于通过库（如 [Application Insights](/azure/azure-monitor/app/asp-net-core) 和 [OpenTelemetry](https://github.com/open-telemetry/opentelemetry-dotnet)）收集应用遥测数据。
* **指标** - 一段时间间隔内数据度量值的表示形式，例如每秒请求数。 指标是使用 `EventCounter` 发出的，可以使用 [dotnet-counters](/dotnet/core/diagnostics/dotnet-counters) 命令行工具或 [Application Insights](/azure/azure-monitor/app/eventcounters) 进行观察。

## <a name="logging"></a>日志记录

gRPC 服务和 gRPC 客户端使用 [.NET Core 日志记录](xref:fundamentals/logging/index)编写日志。 当你需要调试应用中的意外行为时，日志是一个不错的起点。

### <a name="grpc-services-logging"></a>gRPC 服务日志记录

> [!WARNING]
> 服务器端日志可能包含来自应用的敏感信息。 切勿将来自生产应用的原始日志发布到 GitHub 等公共论坛。

由于 gRPC 服务托管在 ASP.NET Core 上，因此它使用 ASP.NET Core 日志记录系统。 在默认配置中，gRPC 只记录很少的信息，但这可以进行配置。 有关配置 ASP.NET Core 日志记录的详细信息，请参阅 [ASP.NET Core 日志记录](xref:fundamentals/logging/index#configuration)上的文档。

gRPC 在 `Grpc` 类别下添加日志。 若要启用来自 gRPC 的详细日志，请通过在 `Logging` 中的 `LogLevel` 子节中添加以下项目，将 `Grpc` 前缀配置为 appsettings.json 文件中的 `Debug` 级别：

[!code-json[](diagnostics/sample/logging-config.json?highlight=7)]

你也可以在具有 `ConfigureLogging` 的 Startup.cs 中配置此项：

[!code-csharp[](diagnostics/sample/logging-config-code.cs?highlight=5)]

如果不使用基于 JSON 的配置，请在配置系统中设置以下配置值：

* `Logging:LogLevel:Grpc` = `Debug`

查看配置系统的文档以确定如何指定嵌套的配置值。 例如，使用环境变量时，使用两个 `_` 而不是 `:` 字符（例如 `Logging__LogLevel__Grpc`）。

建议在为应用收集更详细的诊断时使用 `Debug` 级别。 `Trace` 级别产生的诊断级别很低，且很少需要它来诊断应用中的问题。

#### <a name="sample-logging-output"></a>日志记录输出示例

下面是 `Debug` 级别的 gRPC 服务控制台输出示例：

```console
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 POST https://localhost:5001/Greet.Greeter/SayHello application/grpc
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /Greet.Greeter/SayHello'
dbug: Grpc.AspNetCore.Server.ServerCallHandler[1]
      Reading message.
info: GrpcService.GreeterService[0]
      Hello World
dbug: Grpc.AspNetCore.Server.ServerCallHandler[6]
      Sending message.
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /Greet.Greeter/SayHello'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 1.4113ms 200 application/grpc
```

### <a name="access-server-side-logs"></a>访问服务器端日志

访问服务器端日志的方式取决于在其中运行的环境。

#### <a name="as-a-console-app"></a>作为控制台应用

如果在控制台应用中运行，则默认情况下应启用[控制台记录器](xref:fundamentals/logging/index#console)。 gRPC 日志将在控制台中显示。

#### <a name="other-environments"></a>其他环境

如果将应用部署到另一个环境（例如 Docker、Kubernetes 或 Windows 服务），请参阅 <xref:fundamentals/logging/index>，了解有关如何配置适用于环境的日志记录提供程序的详细信息。

### <a name="grpc-client-logging"></a>gRPC 客户端日志记录

> [!WARNING]
> 客户端日志可能包含来自应用的敏感信息。 切勿将来自生产应用的原始日志发布到 GitHub 等公共论坛。

若要从 .NET 客户端获取日志，可以在创建客户端通道时设置 `GrpcChannelOptions.LoggerFactory` 属性。 如果要从 ASP.NET Core 的应用调用 gRPC 服务，则可以通过依赖关系注入 (DI) 来解析记录器工厂：

[!code-csharp[](diagnostics/sample/net-client-dependency-injection.cs?highlight=7,16)]

启用客户端日志记录的另一种方法是使用 [gRPC 客户端工厂](xref:grpc/clientfactory)创建客户端。 已向客户端工厂注册且解析自 DI 的 gRPC 客户端将自动使用应用的已配置日志记录。

如果应用未使用 DI，则可以使用 [LoggerFactory.Create](xref:Microsoft.Extensions.Logging.LoggerFactory.Create*) 创建新的 `ILoggerFactory` 实例。 若要访问此方法，请将 [Microsoft.Extensions.Logging](https://www.nuget.org/packages/microsoft.extensions.logging/) 包添加到应用。

[!code-csharp[](diagnostics/sample/net-client-loggerfactory-create.cs?highlight=1,8)]

#### <a name="grpc-client-log-scopes"></a>gRPC 客户端日志作用域

gRPC 客户端可将[日志记录作用域](../fundamentals/logging/index.md#log-scopes)添加到在 gRPC 调用期间创建的日志。 作用域具有与 gRPC 调用相关的元数据：

* **GrpcMethodType** - gRPC 方法类型。 可能的值为来自 `Grpc.Core.MethodType` 枚举的名称，如一元
* **GrpcUri** - gRPC 方法的相对 URI，例如 /greet.Greeter/SayHellos

#### <a name="sample-logging-output"></a>日志记录输出示例

下面是 `Debug` 级别的 gRPC 客户端控制台输出示例：

```console
dbug: Grpc.Net.Client.Internal.GrpcCall[1]
      Starting gRPC call. Method type: 'Unary', URI: 'https://localhost:5001/Greet.Greeter/SayHello'.
dbug: Grpc.Net.Client.Internal.GrpcCall[6]
      Sending message.
dbug: Grpc.Net.Client.Internal.GrpcCall[1]
      Reading message.
dbug: Grpc.Net.Client.Internal.GrpcCall[4]
      Finished gRPC call.
```

## <a name="tracing"></a>跟踪

gRPC 服务和 gRPC 客户端使用 [DiagnosticSource](/dotnet/api/system.diagnostics.diagnosticsource) 和 [Activity](/dotnet/api/system.diagnostics.activity) 提供有关 gRPC 调用的信息。

* .NET gRPC 使用活动来表示 gRPC 调用。
* 跟踪事件将在 gRPC 调用活动开始和结束时写入到诊断源。
* 跟踪不会捕获有关在 gRPC 流调用的生存期内消息何时发送的信息。

### <a name="grpc-service-tracing"></a>gRPC 服务跟踪

gRPC 服务托管在会报告有关传入 HTTP 请求事件的 ASP.NET Core 上。 特定于 gRPC 的元数据将添加到 ASP.NET Core 提供的现有 HTTP 请求诊断。

* 诊断源名称为 `Microsoft.AspNetCore`。
* 活动名称为 `Microsoft.AspNetCore.Hosting.HttpRequestIn`。
  * gRPC 调用所调用的 gRPC 方法的名称将添加为标记，标记名称为 `grpc.method`。
  * gRPC 调用完成后，其状态代码将添加为标记，标记名称为 `grpc.status_code`。

### <a name="grpc-client-tracing"></a>gRPC 客户端跟踪

.NET gRPC 客户端使用 `HttpClient` 进行 gRPC 调用。 尽管 `HttpClient` 可编写诊断事件，但 .NET gRPC 客户端提供自定义诊断源、活动和事件，以便可以收集有关 gRPC 调用的完整信息。

* 诊断源名称为 `Grpc.Net.Client`。
* 活动名称为 `Grpc.Net.Client.GrpcOut`。
  * gRPC 调用所调用的 gRPC 方法的名称将添加为标记，标记名称为 `grpc.method`。
  * gRPC 调用完成后，其状态代码将添加为标记，标记名称为 `grpc.status_code`。

### <a name="collecting-tracing"></a>收集跟踪

使用 `DiagnosticSource` 的最简单方法是在应用中配置遥测库，如 [Application Insights](/azure/azure-monitor/app/asp-net-core) 或 [OpenTelemetry](https://github.com/open-telemetry/opentelemetry-dotnet)。 该库将与其他应用遥测一起处理有关 gRPC 调用的信息。

可以在托管服务（如 Application Insights）中查看跟踪，也可以选择运行自己的分布式跟踪系统。 OpenTelemetry 支持将跟踪数据导出到 [Jaeger](https://www.jaegertracing.io/) 和 [Zipkin](https://zipkin.io/)。

`DiagnosticSource` 可以使用 `DiagnosticListener` 在代码中使用跟踪事件。 有关使用代码侦听诊断源的信息，请参阅 [DiagnosticSource 用户指南](https://github.com/dotnet/corefx/blob/d3942d4671919edb0cca6ddc1840190f524a809d/src/System.Diagnostics.DiagnosticSource/src/DiagnosticSourceUsersGuide.md#consuming-data-with-diagnosticlistener)。

> [!NOTE]
> 遥测库当前不捕获特定于 gRPC 的 `Grpc.Net.Client.GrpcOut` 遥测。 改进捕获此跟踪的遥测库的工作正在进行。

## <a name="metrics"></a>指标

指标是一段时间间隔内数据度量值的表示形式，例如每秒请求数。 使用指标数据可以在高级别观察应用的状态。 .NET gRPC 指标是使用 `EventCounter` 发出的。

### <a name="grpc-service-metrics"></a>gRPC 服务指标

gRPC 服务器指标在 `Grpc.AspNetCore.Server` 事件源上报告。

| “属性”                      | 描述                   |
| --------------------------|-------------------------------|
| `total-calls`             | 总调用数                   |
| `current-calls`           | 当前调用                 |
| `calls-failed`            | 失败调用总数            |
| `calls-deadline-exceeded` | 超出截止时间的调用总数 |
| `messages-sent`           | 发送的邮件总数           |
| `messages-received`       | 收到的消息总数       |
| `calls-unimplemented`     | 总未实现调用数     |

ASP.NET Core 还在 `Microsoft.AspNetCore.Hosting` 事件源上提供其自己的指标。

### <a name="grpc-client-metrics"></a>gRPC 客户端指标

gRPC 客户端指标在 `Grpc.Net.Client` 事件源上报告。

| “属性”                      | 描述                   |
| --------------------------|-------------------------------|
| `total-calls`             | 总调用数                   |
| `current-calls`           | 当前调用                 |
| `calls-failed`            | 失败调用总数            |
| `calls-deadline-exceeded` | 超出截止时间的调用总数 |
| `messages-sent`           | 发送的邮件总数           |
| `messages-received`       | 收到的消息总数       |

### <a name="observe-metrics"></a>观察指标

[dotnet-counters](/dotnet/core/diagnostics/dotnet-counters) 是一个性能监视工具，用于临时运行状况监视和初级性能调查。 使用 `Grpc.AspNetCore.Server` 或 `Grpc.Net.Client` 作为提供程序名称监视 .NET 应用。

```console
> dotnet-counters monitor --process-id 1902 Grpc.AspNetCore.Server

Press p to pause, r to resume, q to quit.
    Status: Running
[Grpc.AspNetCore.Server]
    Total Calls                                 300
    Current Calls                               5
    Total Calls Failed                          0
    Total Calls Deadline Exceeded               0
    Total Messages Sent                         295
    Total Messages Received                     300
    Total Calls Unimplemented                   0
```

观察 gRPC 指标的另一种方法是使用 Application Insights 的 [ 包](/azure/azure-monitor/app/eventcounters)捕获计数器数据。 设置完成后，Application Insights 可在运行时收集常见的 .NET 计数器。 默认情况下，不收集 gRPC 的计数器，但可以[自定义 App Insights 以包括其他计数器](/azure/azure-monitor/app/eventcounters#customizing-counters-to-be-collected)。

为 Application Insight 指定 gRPC 计数器以在 Startup.cs 中收集：

```csharp
    using Microsoft.ApplicationInsights.Extensibility.EventCounterCollector;

    public void ConfigureServices(IServiceCollection services)
    {
        //... other code...

        services.ConfigureTelemetryModule<EventCounterCollectionModule>(
            (module, o) =>
            {
                // Configure App Insights to collect gRPC counters gRPC services hosted in an ASP.NET Core app
                module.Counters.Add(new EventCounterCollectionRequest("Grpc.AspNetCore.Server", "current-calls"));
                module.Counters.Add(new EventCounterCollectionRequest("Grpc.AspNetCore.Server", "total-calls"));
                module.Counters.Add(new EventCounterCollectionRequest("Grpc.AspNetCore.Server", "calls-failed"));
            }
        );
    }
```

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/logging/index>
* <xref:grpc/configuration>
* <xref:grpc/clientfactory>