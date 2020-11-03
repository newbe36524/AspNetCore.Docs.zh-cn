---
title: ASP.NET Core 中的 .NET 通用主机
author: rick-anderson
description: 使用 ASP.NET Core 应用中的 .NET Core 通用主机。  通用主机负责应用启动和生存期管理。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 4/17/2020
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
uid: fundamentals/host/generic-host
ms.openlocfilehash: d3de81ce7248372279b423da865513ee5db73c79
ms.sourcegitcommit: d7991068bc6b04063f4bd836fc5b9591d614d448
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91762316"
---
# <a name="net-generic-host-in-aspnet-core"></a>ASP.NET Core 中的 .NET 通用主机

::: moniker range=">= aspnetcore-5.0"

ASP.NET Core 模板会创建一个 .NET Core 泛型主机 (<xref:Microsoft.Extensions.Hosting.HostBuilder>)。

本主题介绍如何使用 ASP.NET Core 中的 .NET 通用主机。 若要了解如何使用控制台应用中的 .NET 通用主机，请参阅 [.NET 通用主机](/dotnet/core/extensions/generic-host)。

## <a name="host-definition"></a>主机定义

主机是封装应用资源的对象，例如：

* 依赖关系注入 (DI)
* Logging
* Configuration
* `IHostedService` 实现

当主机启动时，它将对在托管服务的服务容器集合中注册的 <xref:Microsoft.Extensions.Hosting.IHostedService> 的每个实现调用 <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType>。 在 web 应用中，其中一个 `IHostedService` 实现是启动 [HTTP 服务器实现](xref:fundamentals/index#servers)的 web 服务。

一个对象中包含所有应用的相互依赖资源的主要原因是生存期管理：控制应用启动和正常关闭。

## <a name="set-up-a-host"></a>设置主机

主机通常由 `Program` 类中的代码配置、生成和运行。 `Main` 方法：

* 调用 `CreateHostBuilder` 方法以创建和配置生成器对象。
* 对生成器对象调用 `Build` 和 `Run` 方法。

ASP.NET Core Web 模板会生成以下代码来创建一个主机：

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

以下代码会使用添加到 DI 容器中的 `IHostedService` 实现创建一个非 HTTP 工作负载。

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

对于 HTTP 工作负荷，`Main` 方法相同，但 `CreateHostBuilder` 调用 `ConfigureWebHostDefaults`：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

如果应用使用 Entity Framework Core，不要更改 `CreateHostBuilder` 方法的名称或签名。 [Entity Framework Core 工具](/ef/core/miscellaneous/cli/)应查找一个无需运行应用即可配置主机的 `CreateHostBuilder` 方法。 有关详细信息，请参阅[设计时 DbContext 创建](/ef/core/miscellaneous/cli/dbcontext-creation)。

## <a name="default-builder-settings"></a>默认生成器设置

<xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> 方法：

* 将[内容根目录](xref:fundamentals/index#content-root)设置为由 <xref:System.IO.Directory.GetCurrentDirectory*> 返回的路径。
* 通过以下项加载主机配置：
  * 前缀为 `DOTNET_` 的环境变量。
  * 命令行参数。
* 通过以下对象加载应用配置：
  * appsettings.json。
  * appsettings.{Environment}.json。
  * [密钥管理器](xref:security/app-secrets) 当应用在 `Development` 环境中运行时。
  * 环境变量。
  * 命令行参数。
* 添加以下[日志记录](xref:fundamentals/logging/index)提供程序：
  * 控制台
  * 调试
  * EventSource
  * EventLog（仅当在 Windows 上运行时）
* 当环境为“开发”时，启用[范围验证](xref:fundamentals/dependency-injection#scope-validation)和[依赖关系验证](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild)。

`ConfigureWebHostDefaults` 方法：

* 从前缀为 `ASPNETCORE_` 的环境变量加载主机配置。
* 使用应用的托管配置提供程序将 [Kestrel](xref:fundamentals/servers/kestrel) 服务器设置为 web 服务器并对其进行配置。 有关 Kestrel 服务器默认选项，请参阅 <xref:fundamentals/servers/kestrel#kestrel-options>。
* 添加[主机筛选中间件](xref:fundamentals/servers/kestrel#host-filtering)。
* 如果 `ASPNETCORE_FORWARDEDHEADERS_ENABLED` 等于 `true`，则添加[转接头中间件](xref:host-and-deploy/proxy-load-balancer#forwarded-headers)。
* 支持 IIS 集成。 有关 IIS 默认选项，请参阅 <xref:host-and-deploy/iis/index#iis-options>。

本文中后面的[所有应用类型的设置](#settings-for-all-app-types)和[ web 应用的设置](#settings-for-web-apps)部分介绍如何替代默认生成器设置。

## <a name="framework-provided-services"></a>框架提供的服务

自动注册以下服务：

* [IHostApplicationLifetime](#ihostapplicationlifetime)
* [IHostLifetime](#ihostlifetime)
* [IHostEnvironment / IWebHostEnvironment](#ihostenvironment)

有关框架提供的服务的详细信息，请参阅 <xref:fundamentals/dependency-injection#framework-provided-services>。

## <a name="ihostapplicationlifetime"></a>IHostApplicationLifetime

将 <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>（以前称为 `IApplicationLifetime`）服务注入任何类以处理启动后和正常关闭任务。 接口上的三个属性是用于注册应用启动和应用停止事件处理程序方法的取消令牌。 该接口还包括 `StopApplication` 方法。

以下示例是注册 `IHostApplicationLifetime` 事件的 `IHostedService` 实现：

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a>IHostLifetime

<xref:Microsoft.Extensions.Hosting.IHostLifetime> 实现控制主机何时启动和何时停止。 使用了已注册的最后一个实现。

`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` 是默认的 `IHostLifetime` 实现。 `ConsoleLifetime`:

* 侦听 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM 并调用 <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication*> 来启动关闭进程。
* 解除阻止 [RunAsync](#runasync) 和 [WaitForShutdownAsync](#waitforshutdownasync) 等扩展。

## <a name="ihostenvironment"></a>IHostEnvironment

将 <xref:Microsoft.Extensions.Hosting.IHostEnvironment> 服务注册到一个类，获取关于以下设置的信息：

* [ApplicationName](#applicationname)
* [EnvironmentName](#environmentname)
* [ContentRootPath](#contentroot)

Web 应用实现 `IWebHostEnvironment` 接口，该接口继承 `IHostEnvironment` 并添加 [WebRootPath](#webroot)。

## <a name="host-configuration"></a>主机配置

主机配置用于 <xref:Microsoft.Extensions.Hosting.IHostEnvironment> 实现的属性。

主机配置可以从 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 内的 [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) 获取。 在 `ConfigureAppConfiguration` 后，`HostBuilderContext.Configuration` 被替换为应用配置。

若要添加主机配置，请对 `IHostBuilder` 调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>。 可多次调用 `ConfigureHostConfiguration`，并得到累计结果。 主机使用上一次在一个给定键上设置值的选项。

`CreateDefaultBuilder` 包含前缀为 `DOTNET_` 的环境变量提供程序和命令行参数。 对于 web 应用程序，添加前缀为 `ASPNETCORE_` 的环境变量提供程序。 当系统读取环境变量时，便会删除前缀。 例如，`ASPNETCORE_ENVIRONMENT` 的环境变量值就变成 `environment` 密钥的主机配置值。

以下示例创建主机配置：

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a>应用配置

通过对 `IHostBuilder` 调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 创建应用配置。 可多次调用 `ConfigureAppConfiguration`，并得到累计结果。 应用使用上一次在一个给定键上设置值的选项。 

由 `ConfigureAppConfiguration` 创建的配置可以通过 [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) 获取以用于后续操作，也可以通过 DI 作为服务获取。 主机配置也会添加到应用配置。

有关详细信息，请参阅 [ ASP.NET Core 中的配置](xref:fundamentals/configuration/index#configureappconfiguration)。

## <a name="settings-for-all-app-types"></a>适用于所有应用类型的设置

本部分列出了适用于 HTTP 和非 HTTP 工作负荷的主机设置。 默认情况下，用来配置这些设置的环境变量可以具有 `DOTNET_` 或 `ASPNETCORE_` 前缀。

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a>ApplicationName

[IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) 属性是在主机构造期间通过主机配置设定的。

键：`applicationName`  
类型：`string`  
**默认**：包含应用入口点的程序集的名称。  
**环境变量**：`<PREFIX_>APPLICATIONNAME`

要设置此值，请使用环境变量。 

### <a name="contentroot"></a>ContentRoot

[IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) 属性决定主机从什么位置开始搜索内容文件。 如果路径不存在，主机将无法启动。

键：`contentRoot`  
类型：`string`  
**默认**：应用程序集所在的文件夹。  
**环境变量**：`<PREFIX_>CONTENTROOT`

若要设置此值，请使用环境变量或对 `IHostBuilder` 调用 `UseContentRoot`：

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

有关详情，请参阅：

* [基础知识：内容根目录](xref:fundamentals/index#content-root)
* [WebRoot](#webroot)

### <a name="environmentname"></a>EnvironmentName

[IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) 属性可以设置为任何值。 框架定义的值包括 `Development``Staging` 和 `Production`。 值不区分大小写。

键：`environment`  
类型：`string`  
**默认**：`Production`  
**环境变量**：`<PREFIX_>ENVIRONMENT`

若要设置此值，请使用环境变量或对 `IHostBuilder` 调用 `UseEnvironment`：

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a>ShutdownTimeout

[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) 设置 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 的超时。 默认值为 5 秒。  在超时时间段中，主机：

* 触发 [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping)。
* 尝试停止托管服务，对服务停止失败的错误进行日志记录。

如果在所有托管服务停止之前就达到了超时时间，则会在应用关闭时会终止剩余的所有活动的服务。 即使没有完成处理工作，服务也会停止。 如果停止服务需要额外的时间，请增加超时时间。

键：`shutdownTimeoutSeconds`  
类型：`int`  
**默认**：5 秒  
**环境变量**：`<PREFIX_>SHUTDOWNTIMEOUTSECONDS`

若要设置此值，请使用环境变量或配置 `HostOptions`。 以下示例将超时设置为 20 秒：

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

### <a name="disable-app-configuration-reload-on-change"></a>禁用“在更改时重载应用配置”

[默认情况下](xref:fundamentals/configuration/index#default)，appsettings.json 和 appsettings.{Environment}.json 会在文件更改时重载 。 若要在 ASP.NET Core 5.0 或更高版本中禁用此重载行为，请将 `hostBuilder:reloadConfigOnChange` 键设置为 `false`。

键：`hostBuilder:reloadConfigOnChange`  
类型：`bool`（`true` 或 `1`）  
**默认**：`true`  
命令行参数：`hostBuilder:reloadConfigOnChange`  
**环境变量**：`<PREFIX_>hostBuilder:reloadConfigOnChange`

> [!WARNING]
> 所有平台上的环境变量分层键都不支持冒号 (`:`) 分隔符。 有关详细信息，请参阅[环境变量](xref:fundamentals/configuration/index#environment-variables)。

## <a name="settings-for-web-apps"></a>适用于 Web 应用的设置

一些主机设置仅适用于 HTTP 工作负荷。 默认情况下，用来配置这些设置的环境变量可以具有 `DOTNET_` 或 `ASPNETCORE_` 前缀。

`IWebHostBuilder` 上的扩展方法适用于这些设置。 显示如何调用扩展方法的示例代码假定 `webBuilder` 是 `IWebHostBuilder` 的实例，如以下示例所示：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a>CaptureStartupErrors

当 `false` 时，启动期间出错导致主机退出。 当 `true` 时，主机在启动期间捕获异常并尝试启动服务器。

键：`captureStartupErrors`  
类型：`bool`（`true` 或 `1`）  
**默认**：默认为 `false`，除非应用使用 Kestrel 在 IIS 后方运行，其中默认值是 `true`。  
**环境变量**：`<PREFIX_>CAPTURESTARTUPERRORS`

若要设置此值，使用配置或调用 `CaptureStartupErrors`：

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a>DetailedErrors

如果启用，或环境为 `Development`，应用会捕获详细错误。

键：`detailedErrors`  
类型：`bool`（`true` 或 `1`）  
**默认**：`false`  
**环境变量**：`<PREFIX_>_DETAILEDERRORS`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a>HostingStartupAssemblies

承载启动程序集的以分号分隔的字符串在启动时加载。 虽然配置值默认为空字符串，但是承载启动程序集会始终包含应用的程序集。 提供承载启动程序集时，当应用在启动过程中生成其公用服务时将它们添加到应用的程序集加载。

键：`hostingStartupAssemblies`  
类型：`string`  
**默认**：空字符串  
**环境变量**：`<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a>HostingStartupExcludeAssemblies

承载启动程序集的以分号分隔的字符串在启动时排除。

键：`hostingStartupExcludeAssemblies`  
类型：`string`  
**默认**：空字符串  
**环境变量**：`<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a>HTTPS_Port

HTTPS 重定向端口。 用于[强制实施 HTTPS](xref:security/enforcing-ssl)。

键：`https_port`  
类型：`string`  
**默认**：未设置默认值。  
**环境变量**：`<PREFIX_>HTTPS_PORT`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a>PreferHostingUrls

指示主机是否应该侦听使用 `IWebHostBuilder` 配置的 URL，而不是使用 `IServer` 实现配置的 URL。

键：`preferHostingUrls`  
类型：`bool`（`true` 或 `1`）  
**默认**：`true`  
**环境变量**：`<PREFIX_>_PREFERHOSTINGURLS`

若要设置此值，请使用环境变量或调用 `PreferHostingUrls`：

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a>PreventHostingStartup

阻止承载启动程序集自动加载，包括应用的程序集所配置的承载启动程序集。 有关详细信息，请参阅 <xref:fundamentals/configuration/platform-specific-configuration>。

键：`preventHostingStartup`  
类型：`bool`（`true` 或 `1`）  
**默认**：`false`  
**环境变量**：`<PREFIX_>_PREVENTHOSTINGSTARTUP`

若要设置此值，请使用环境变量或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a>StartupAssembly

要搜索 `Startup` 类的程序集。

键：`startupAssembly`  
类型：`string`  
**默认**：应用的程序集  
**环境变量**：`<PREFIX_>STARTUPASSEMBLY`

若要设置此值，请使用环境变量或调用 `UseStartup`。 `UseStartup` 可以采用程序集名称 (`string`) 或类型 (`TStartup`)。 如果调用多个 `UseStartup` 方法，优先选择最后一个方法。

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a>URL

IP 地址或主机地址的分号分隔列表，其中包含服务器应针对请求侦听的端口和协议。 例如 `http://localhost:123`。 使用“\*”指示服务器应针对请求侦听的使用特定端口和协议（例如 `http://*:5000`）的 IP 地址或主机名。 协议（`http://` 或 `https://`）必须包含每个 URL。 不同的服务器支持的格式有所不同。

键：`urls`  
类型：`string`  
**默认值**：`http://localhost:5000` 和 `https://localhost:5001`  
**环境变量**：`<PREFIX_>URLS`

若要设置此值，请使用环境变量或调用 `UseUrls`：

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

Kestrel 具有自己的终结点配置 API。 有关详细信息，请参阅 <xref:fundamentals/servers/kestrel#endpoint-configuration>。

### <a name="webroot"></a>WebRoot

[IWebHostEnvironment.WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) 属性可确定应用静态资产的相对路径。 如果该路径不存在，则使用无操作文件提供程序。  

键：`webroot`  
类型：`string`  
**默认**：默认值为 `wwwroot`。 {content root}/wwwroot 的路径必须存在。  
**环境变量**：`<PREFIX_>WEBROOT`

若要设置此值，请使用环境变量或对 `IWebHostBuilder` 调用 `UseWebRoot`：

```csharp
webBuilder.UseWebRoot("public");
```

有关详情，请参阅：

* [基础知识：Web 根目录](xref:fundamentals/index#web-root)
* [ContentRoot](#contentroot)

## <a name="manage-the-host-lifetime"></a>管理主机生存期

对生成的 <xref:Microsoft.Extensions.Hosting.IHost> 实现调用方法，以启动和停止应用。 这些方法会影响所有在服务容器中注册的 <xref:Microsoft.Extensions.Hosting.IHostedService> 实现。

### <a name="run"></a>运行

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> 运行应用并阻止调用线程，直到关闭主机。

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> 运行应用并返回在触发取消令牌或关闭时完成的 <xref:System.Threading.Tasks.Task>。

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> 启用控制台支持、生成和启动主机，以及等待 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM 关闭。

### <a name="start"></a>Start

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> 同步启动主机。

### <a name="startasync"></a>StartAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> 启动主机并返回在触发取消令牌或关闭时完成的 <xref:System.Threading.Tasks.Task>。 

在 `StartAsync` 开始时调用 <xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*>，在继续之前，会一直等待该操作完成。 它可用于延迟启动，直到外部事件发出信号。

### <a name="stopasync"></a>StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> 尝试在提供的超时时间内停止主机。

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> 阻止调用线程，直到 IHostLifetime 触发关闭，例如通过 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM。

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> 返回在通过给定的令牌和调用 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 来触发关闭时完成的 <xref:System.Threading.Tasks.Task>。

### <a name="external-control"></a>外部控件

使用可从外部调用的方法，能够实现对主机生存期的直接控制：

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

ASP.NET Core 模板会创建一个 .NET Core 泛型主机 (<xref:Microsoft.Extensions.Hosting.HostBuilder>)。

本主题介绍如何使用 ASP.NET Core 中的 .NET 通用主机。 若要了解如何使用控制台应用中的 .NET 通用主机，请参阅 [.NET 通用主机](/dotnet/core/extensions/generic-host)。

## <a name="host-definition"></a>主机定义

主机是封装应用资源的对象，例如：

* 依赖关系注入 (DI)
* Logging
* Configuration
* `IHostedService` 实现

当主机启动时，它将对在托管服务的服务容器集合中注册的 <xref:Microsoft.Extensions.Hosting.IHostedService> 的每个实现调用 <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync%2A?displayProperty=nameWithType>。 在 web 应用中，其中一个 `IHostedService` 实现是启动 [HTTP 服务器实现](xref:fundamentals/index#servers)的 web 服务。

一个对象中包含所有应用的相互依赖资源的主要原因是生存期管理：控制应用启动和正常关闭。

## <a name="set-up-a-host"></a>设置主机

主机通常由 `Program` 类中的代码配置、生成和运行。 `Main` 方法：

* 调用 `CreateHostBuilder` 方法以创建和配置生成器对象。
* 对生成器对象调用 `Build` 和 `Run` 方法。

ASP.NET Core Web 模板会生成以下代码来创建泛型主机：

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

以下代码会使用非 HTTP 工作负载创建一个泛型主机。 `IHostedService` 实现会添加到 DI 容器中：

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureServices((hostContext, services) =>
            {
               services.AddHostedService<Worker>();
            });
}
```

对于 HTTP 工作负荷，`Main` 方法相同，但 `CreateHostBuilder` 调用 `ConfigureWebHostDefaults`：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

上述代码是由 ASP.NET Core 模板生成的。

如果应用使用 Entity Framework Core，不要更改 `CreateHostBuilder` 方法的名称或签名。 [Entity Framework Core 工具](/ef/core/miscellaneous/cli/)应查找一个无需运行应用即可配置主机的 `CreateHostBuilder` 方法。 有关详细信息，请参阅[设计时 DbContext 创建](/ef/core/miscellaneous/cli/dbcontext-creation)。

## <a name="default-builder-settings"></a>默认生成器设置

<xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder%2A> 方法：

* 将[内容根目录](xref:fundamentals/index#content-root)设置为由 <xref:System.IO.Directory.GetCurrentDirectory%2A> 返回的路径。
* 通过以下项加载主机配置：
  * 前缀为 `DOTNET_` 的环境变量。
  * 命令行参数。
* 通过以下对象加载应用配置：
  * appsettings.json。
  * appsettings.{Environment}.json。
  * [密钥管理器](xref:security/app-secrets) 当应用在 `Development` 环境中运行时。
  * 环境变量。
  * 命令行参数。
* 添加以下[日志记录](xref:fundamentals/logging/index)提供程序：
  * 控制台
  * 调试
  * EventSource
  * EventLog（仅当在 Windows 上运行时）
* 当环境为“开发”时，启用[范围验证](xref:fundamentals/dependency-injection#scope-validation)和[依赖关系验证](xref:Microsoft.Extensions.DependencyInjection.ServiceProviderOptions.ValidateOnBuild)。

`ConfigureWebHostDefaults` 方法：

* 从前缀为 `ASPNETCORE_` 的环境变量加载主机配置。
* 使用应用的托管配置提供程序将 [Kestrel](xref:fundamentals/servers/kestrel) 服务器设置为 web 服务器并对其进行配置。 有关 Kestrel 服务器默认选项，请参阅 <xref:fundamentals/servers/kestrel#kestrel-options>。
* 添加[主机筛选中间件](xref:fundamentals/servers/kestrel#host-filtering)。
* 如果 `ASPNETCORE_FORWARDEDHEADERS_ENABLED` 等于 `true`，则添加[转接头中间件](xref:host-and-deploy/proxy-load-balancer#forwarded-headers)。
* 支持 IIS 集成。 有关 IIS 默认选项，请参阅 <xref:host-and-deploy/iis/index#iis-options>。

本文中后面的[所有应用类型的设置](#settings-for-all-app-types)和[ web 应用的设置](#settings-for-web-apps)部分介绍如何替代默认生成器设置。

## <a name="framework-provided-services"></a>框架提供的服务

自动注册以下服务：

* [IHostApplicationLifetime](#ihostapplicationlifetime)
* [IHostLifetime](#ihostlifetime)
* [IHostEnvironment / IWebHostEnvironment](#ihostenvironment)

有关框架提供的服务的详细信息，请参阅 <xref:fundamentals/dependency-injection#framework-provided-services>。

## <a name="ihostapplicationlifetime"></a>IHostApplicationLifetime

将 <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>（以前称为 `IApplicationLifetime`）服务注入任何类以处理启动后和正常关闭任务。 接口上的三个属性是用于注册应用启动和应用停止事件处理程序方法的取消令牌。 该接口还包括 `StopApplication` 方法。

以下示例是注册 `IHostApplicationLifetime` 事件的 `IHostedService` 实现：

[!code-csharp[](generic-host/samples-snapshot/3.x/LifetimeEventsHostedService.cs?name=snippet_LifetimeEvents)]

## <a name="ihostlifetime"></a>IHostLifetime

<xref:Microsoft.Extensions.Hosting.IHostLifetime> 实现控制主机何时启动和何时停止。 使用了已注册的最后一个实现。

`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` 是默认的 `IHostLifetime` 实现。 `ConsoleLifetime`:

* 侦听 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM 并调用 <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime.StopApplication%2A> 来启动关闭进程。
* 解除阻止 [RunAsync](#runasync) 和 [WaitForShutdownAsync](#waitforshutdownasync) 等扩展。

## <a name="ihostenvironment"></a>IHostEnvironment

将 <xref:Microsoft.Extensions.Hosting.IHostEnvironment> 服务注册到一个类，获取关于以下设置的信息：

* [ApplicationName](#applicationname)
* [EnvironmentName](#environmentname)
* [ContentRootPath](#contentroot)

Web 应用实现 `IWebHostEnvironment` 接口，该接口继承 `IHostEnvironment` 并添加 [WebRootPath](#webroot)。

## <a name="host-configuration"></a>主机配置

主机配置用于 <xref:Microsoft.Extensions.Hosting.IHostEnvironment> 实现的属性。

主机配置可以从 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration%2A> 内的 [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration) 获取。 在 `ConfigureAppConfiguration` 后，`HostBuilderContext.Configuration` 被替换为应用配置。

若要添加主机配置，请对 `IHostBuilder` 调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration%2A>。 可多次调用 `ConfigureHostConfiguration`，并得到累计结果。 主机使用上一次在一个给定键上设置值的选项。

`CreateDefaultBuilder` 包含前缀为 `DOTNET_` 的环境变量提供程序和命令行参数。 对于 web 应用程序，添加前缀为 `ASPNETCORE_` 的环境变量提供程序。 当系统读取环境变量时，便会删除前缀。 例如，`ASPNETCORE_ENVIRONMENT` 的环境变量值就变成 `environment` 密钥的主机配置值。

以下示例创建主机配置：

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostConfig)]

## <a name="app-configuration"></a>应用配置

通过对 `IHostBuilder` 调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 创建应用配置。 可多次调用 `ConfigureAppConfiguration`，并得到累计结果。 应用使用上一次在一个给定键上设置值的选项。 

由 `ConfigureAppConfiguration` 创建的配置可以通过 [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) 获取以用于后续操作，也可以通过 DI 作为服务获取。 主机配置也会添加到应用配置。

有关详细信息，请参阅 [ ASP.NET Core 中的配置](xref:fundamentals/configuration/index#configureappconfiguration)。

## <a name="settings-for-all-app-types"></a>适用于所有应用类型的设置

本部分列出了适用于 HTTP 和非 HTTP 工作负荷的主机设置。 默认情况下，用来配置这些设置的环境变量可以具有 `DOTNET_` 或 `ASPNETCORE_` 前缀。

<!-- In the following sections, two spaces at end of line are used to force line breaks in the rendered page. -->

### <a name="applicationname"></a>ApplicationName

[IHostEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ApplicationName*) 属性是在主机构造期间通过主机配置设定的。

键：`applicationName`  
类型：`string`  
**默认**：包含应用入口点的程序集的名称。  
**环境变量**：`<PREFIX_>APPLICATIONNAME`

要设置此值，请使用环境变量。 

### <a name="contentroot"></a>ContentRoot

[IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath*) 属性决定主机从什么位置开始搜索内容文件。 如果路径不存在，主机将无法启动。

键：`contentRoot`  
类型：`string`  
**默认**：应用程序集所在的文件夹。  
**环境变量**：`<PREFIX_>CONTENTROOT`

若要设置此值，请使用环境变量或对 `IHostBuilder` 调用 `UseContentRoot`：

```csharp
Host.CreateDefaultBuilder(args)
    .UseContentRoot("c:\\content-root")
    //...
```

有关详情，请参阅：

* [基础知识：内容根目录](xref:fundamentals/index#content-root)
* [WebRoot](#webroot)

### <a name="environmentname"></a>EnvironmentName

[IHostEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostEnvironment.EnvironmentName*) 属性可以设置为任何值。 框架定义的值包括 `Development``Staging` 和 `Production`。 值不区分大小写。

键：`environment`  
类型：`string`  
**默认**：`Production`  
**环境变量**：`<PREFIX_>ENVIRONMENT`

若要设置此值，请使用环境变量或对 `IHostBuilder` 调用 `UseEnvironment`：

```csharp
Host.CreateDefaultBuilder(args)
    .UseEnvironment("Development")
    //...
```

### <a name="shutdowntimeout"></a>ShutdownTimeout

[HostOptions.ShutdownTimeout](xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*) 设置 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 的超时。 默认值为 5 秒。  在超时时间段中，主机：

* 触发 [IHostApplicationLifetime.ApplicationStopping](/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime.applicationstopping)。
* 尝试停止托管服务，对服务停止失败的错误进行日志记录。

如果在所有托管服务停止之前就达到了超时时间，则会在应用关闭时会终止剩余的所有活动的服务。 即使没有完成处理工作，服务也会停止。 如果停止服务需要额外的时间，请增加超时时间。

键：`shutdownTimeoutSeconds`  
类型：`int`  
**默认**：5 秒  
**环境变量**：`<PREFIX_>SHUTDOWNTIMEOUTSECONDS`

若要设置此值，请使用环境变量或配置 `HostOptions`。 以下示例将超时设置为 20 秒：

[!code-csharp[](generic-host/samples-snapshot/3.x/Program.cs?name=snippet_HostOptions)]

## <a name="settings-for-web-apps"></a>适用于 Web 应用的设置

一些主机设置仅适用于 HTTP 工作负荷。 默认情况下，用来配置这些设置的环境变量可以具有 `DOTNET_` 或 `ASPNETCORE_` 前缀。

`IWebHostBuilder` 上的扩展方法适用于这些设置。 显示如何调用扩展方法的示例代码假定 `webBuilder` 是 `IWebHostBuilder` 的实例，如以下示例所示：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.CaptureStartupErrors(true);
            webBuilder.UseStartup<Startup>();
        });
```

### <a name="capturestartuperrors"></a>CaptureStartupErrors

当 `false` 时，启动期间出错导致主机退出。 当 `true` 时，主机在启动期间捕获异常并尝试启动服务器。

键：`captureStartupErrors`  
类型：`bool`（`true` 或 `1`）  
**默认**：默认为 `false`，除非应用使用 Kestrel 在 IIS 后方运行，其中默认值是 `true`。  
**环境变量**：`<PREFIX_>CAPTURESTARTUPERRORS`

若要设置此值，使用配置或调用 `CaptureStartupErrors`：

```csharp
webBuilder.CaptureStartupErrors(true);
```

### <a name="detailederrors"></a>DetailedErrors

如果启用，或环境为 `Development`，应用会捕获详细错误。

键：`detailedErrors`  
类型：`bool`（`true` 或 `1`）  
**默认**：`false`  
**环境变量**：`<PREFIX_>_DETAILEDERRORS`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.DetailedErrorsKey, "true");
```

### <a name="hostingstartupassemblies"></a>HostingStartupAssemblies

承载启动程序集的以分号分隔的字符串在启动时加载。 虽然配置值默认为空字符串，但是承载启动程序集会始终包含应用的程序集。 提供承载启动程序集时，当应用在启动过程中生成其公用服务时将它们添加到应用的程序集加载。

键：`hostingStartupAssemblies`  
类型：`string`  
**默认**：空字符串  
**环境变量**：`<PREFIX_>_HOSTINGSTARTUPASSEMBLIES`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupAssembliesKey, "assembly1;assembly2");
```

### <a name="hostingstartupexcludeassemblies"></a>HostingStartupExcludeAssemblies

承载启动程序集的以分号分隔的字符串在启动时排除。

键：`hostingStartupExcludeAssemblies`  
类型：`string`  
**默认**：空字符串  
**环境变量**：`<PREFIX_>_HOSTINGSTARTUPEXCLUDEASSEMBLIES`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.HostingStartupExcludeAssembliesKey, "assembly1;assembly2");
```

### <a name="https_port"></a>HTTPS_Port

HTTPS 重定向端口。 用于[强制实施 HTTPS](xref:security/enforcing-ssl)。

键：`https_port`  
类型：`string`  
**默认**：未设置默认值。  
**环境变量**：`<PREFIX_>HTTPS_PORT`

要设置此值，使用配置或调用 `UseSetting`：

```csharp
webBuilder.UseSetting("https_port", "8080");
```

### <a name="preferhostingurls"></a>PreferHostingUrls

指示主机是否应该侦听使用 `IWebHostBuilder` 配置的 URL，而不是使用 `IServer` 实现配置的 URL。

键：`preferHostingUrls`  
类型：`bool`（`true` 或 `1`）  
**默认**：`true`  
**环境变量**：`<PREFIX_>_PREFERHOSTINGURLS`

若要设置此值，请使用环境变量或调用 `PreferHostingUrls`：

```csharp
webBuilder.PreferHostingUrls(false);
```

### <a name="preventhostingstartup"></a>PreventHostingStartup

阻止承载启动程序集自动加载，包括应用的程序集所配置的承载启动程序集。 有关详细信息，请参阅 <xref:fundamentals/configuration/platform-specific-configuration>。

键：`preventHostingStartup`  
类型：`bool`（`true` 或 `1`）  
**默认**：`false`  
**环境变量**：`<PREFIX_>_PREVENTHOSTINGSTARTUP`

若要设置此值，请使用环境变量或调用 `UseSetting`：

```csharp
webBuilder.UseSetting(WebHostDefaults.PreventHostingStartupKey, "true");
```

### <a name="startupassembly"></a>StartupAssembly

要搜索 `Startup` 类的程序集。

键：`startupAssembly`  
类型：`string`  
**默认**：应用的程序集  
**环境变量**：`<PREFIX_>STARTUPASSEMBLY`

若要设置此值，请使用环境变量或调用 `UseStartup`。 `UseStartup` 可以采用程序集名称 (`string`) 或类型 (`TStartup`)。 如果调用多个 `UseStartup` 方法，优先选择最后一个方法。

```csharp
webBuilder.UseStartup("StartupAssemblyName");
```

```csharp
webBuilder.UseStartup<Startup>();
```

### <a name="urls"></a>URL

IP 地址或主机地址的分号分隔列表，其中包含服务器应针对请求侦听的端口和协议。 例如 `http://localhost:123`。 使用“\*”指示服务器应针对请求侦听的使用特定端口和协议（例如 `http://*:5000`）的 IP 地址或主机名。 协议（`http://` 或 `https://`）必须包含每个 URL。 不同的服务器支持的格式有所不同。

键：`urls`  
类型：`string`  
**默认值**：`http://localhost:5000` 和 `https://localhost:5001`  
**环境变量**：`<PREFIX_>URLS`

若要设置此值，请使用环境变量或调用 `UseUrls`：

```csharp
webBuilder.UseUrls("http://*:5000;http://localhost:5001;https://hostname:5002");
```

Kestrel 具有自己的终结点配置 API。 有关详细信息，请参阅 <xref:fundamentals/servers/kestrel#endpoint-configuration>。

### <a name="webroot"></a>WebRoot

[IWebHostEnvironment.WebRootPath](xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment.WebRootPath) 属性可确定应用静态资产的相对路径。 如果该路径不存在，则使用无操作文件提供程序。  

键：`webroot`  
类型：`string`  
**默认**：默认值为 `wwwroot`。 {content root}/wwwroot 的路径必须存在。  
**环境变量**：`<PREFIX_>WEBROOT`

若要设置此值，请使用环境变量或对 `IWebHostBuilder` 调用 `UseWebRoot`：

```csharp
webBuilder.UseWebRoot("public");
```

有关详情，请参阅：

* [基础知识：Web 根目录](xref:fundamentals/index#web-root)
* [ContentRoot](#contentroot)

## <a name="manage-the-host-lifetime"></a>管理主机生存期

对生成的 <xref:Microsoft.Extensions.Hosting.IHost> 实现调用方法，以启动和停止应用。 这些方法会影响所有在服务容器中注册的 <xref:Microsoft.Extensions.Hosting.IHostedService> 实现。

### <a name="run"></a>运行

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> 运行应用并阻止调用线程，直到关闭主机。

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> 运行应用并返回在触发取消令牌或关闭时完成的 <xref:System.Threading.Tasks.Task>。

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> 启用控制台支持、生成和启动主机，以及等待 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM 关闭。

### <a name="start"></a>Start

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> 同步启动主机。

### <a name="startasync"></a>StartAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> 启动主机并返回在触发取消令牌或关闭时完成的 <xref:System.Threading.Tasks.Task>。 

在 `StartAsync` 开始时调用 <xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*>，在继续之前，会一直等待该操作完成。 它可用于延迟启动，直到外部事件发出信号。

### <a name="stopasync"></a>StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> 尝试在提供的超时时间内停止主机。

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> 阻止调用线程，直到 IHostLifetime 触发关闭，例如通过 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM。

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> 返回在通过给定的令牌和调用 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 来触发关闭时完成的 <xref:System.Threading.Tasks.Task>。

### <a name="external-control"></a>外部控件

使用可从外部调用的方法，能够实现对主机生存期的直接控制：

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core 应用配置和启动主机。 主机负责应用程序启动和生存期管理。

本文介绍 ASP.NET Core 泛型主机 (<xref:Microsoft.Extensions.Hosting.HostBuilder>)，该主机用于无法处理 HTTP 请求的应用。

泛型主机的用途是将 HTTP 管道从 Web 主机 API 中分离出来，从而启用更多的主机方案。 基于泛型主机的消息、后台任务和其他非 HTTP 工作负载可从横切功能（如配置、依赖关系注入 [DI] 和日志记录）中受益。

泛型主机是 ASP.NET Core 2.1 中的新增功能，不适用于 Web 承载方案。 对于 Web 承载方案，请使用 [Web 主机](xref:fundamentals/host/web-host)。 泛型主机将在未来版本中替换 Web 主机，并在 HTTP 和非 HTTP 方案中充当主要的主机 API。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

在 [Visual Studio Code](https://code.visualstudio.com/) 中运行示例应用时，请使用外部或集成终端。 请勿在 `internalConsole` 中运行示例。

在 Visual Studio Code 中设置控制台：

1. 打开 .vscode/launch.json 文件。
1. 在 .NET Core 启动（控制台）配置中，找到控制台条目 。 将值设置为 `externalTerminal` 或 `integratedTerminal`。

## <a name="introduction"></a>介绍

通用主机库位于 <xref:Microsoft.Extensions.Hosting> 命名空间中，由 [Microsoft.Extensions.Hosting](https://www.nuget.org/packages/Microsoft.Extensions.Hosting/) 包提供。 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)（ASP.NET Core 2.1 或更高版本）中包括 `Microsoft.Extensions.Hosting` 包。

<xref:Microsoft.Extensions.Hosting.IHostedService> 是执行代码的入口点。 每个 <xref:Microsoft.Extensions.Hosting.IHostedService> 实现都按照 [ConfigureServices 中服务注册](#configureservices)的顺序执行。 主机启动时，每个 <xref:Microsoft.Extensions.Hosting.IHostedService> 上都会调用 <xref:Microsoft.Extensions.Hosting.IHostedService.StartAsync*>，主机正常关闭时，以反向注册顺序调用 <xref:Microsoft.Extensions.Hosting.IHostedService.StopAsync*>。

## <a name="set-up-a-host"></a>设置主机

<xref:Microsoft.Extensions.Hosting.IHostBuilder> 是供库和应用初始化、生成和运行主机的主要组件：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_HostBuilder)]

## <a name="options"></a>选项

<xref:Microsoft.Extensions.Hosting.HostOptions> 配置 <xref:Microsoft.Extensions.Hosting.IHost> 的选项。

### <a name="shutdown-timeout"></a>关闭超时值

<xref:Microsoft.Extensions.Hosting.HostOptions.ShutdownTimeout*> 设置 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 的超时值。 默认值为 5 秒。

`Program.Main` 中的以下选项配置将默认值为 5 秒的关闭超时值增加至 20 秒：

```csharp
var host = new HostBuilder()
    .ConfigureServices((hostContext, services) =>
    {
        services.Configure<HostOptions>(option =>
        {
            option.ShutdownTimeout = System.TimeSpan.FromSeconds(20);
        });
    })
    .Build();
```

## <a name="default-services"></a>默认服务

在主机初始化期间注册以下服务：

* [环境](xref:fundamentals/environments) (<xref:Microsoft.Extensions.Hosting.IHostingEnvironment>)
* <xref:Microsoft.Extensions.Hosting.HostBuilderContext>
* [配置](xref:fundamentals/configuration/index) (<xref:Microsoft.Extensions.Configuration.IConfiguration>)
* <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> (`Microsoft.Extensions.Hosting.Internal.ApplicationLifetime`)
* <xref:Microsoft.Extensions.Hosting.IHostLifetime> (`Microsoft.Extensions.Hosting.Internal.ConsoleLifetime`)
* <xref:Microsoft.Extensions.Hosting.IHost>
* [选项](xref:fundamentals/configuration/options) (<xref:Microsoft.Extensions.DependencyInjection.OptionsServiceCollectionExtensions.AddOptions*>)
* [日志记录](xref:fundamentals/logging/index) (<xref:Microsoft.Extensions.DependencyInjection.LoggingServiceCollectionExtensions.AddLogging*>)

## <a name="host-configuration"></a>主机配置

主机配置的创建方式如下：

* 调用 <xref:Microsoft.Extensions.Hosting.IHostBuilder> 上的扩展方法以设置[“内容根”](#content-root)和[“环境”](#environment)。
* 从 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> 中的配置提供程序读取配置。

### <a name="extension-methods"></a>扩展方法

### <a name="application-key-name"></a>应用程序键（名称）

[IHostingEnvironment.ApplicationName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ApplicationName*) 属性是在主机构造期间通过主机配置设定的。 要显式设置值，请使用 [HostDefaults.ApplicationKey](xref:Microsoft.Extensions.Hosting.HostDefaults.ApplicationKey)：

键：`applicationName`  
类型：`string`  
**默认**：包含应用入口点的程序集的名称。  
**设置使用**：`HostBuilderContext.HostingEnvironment.ApplicationName`  
**环境变量**：`<PREFIX_>APPLICATIONNAME`（`<PREFIX_>` 是[用户定义的可选前缀](#configurehostconfiguration)）

### <a name="content-root"></a>内容根

此设置确定主机从哪里开始搜索内容文件。

键：`contentRoot`  
类型：`string`  
**默认**：默认为应用程序集所在的文件夹。  
**设置使用**：`UseContentRoot`  
**环境变量**：`<PREFIX_>CONTENTROOT`（`<PREFIX_>` 是[用户定义的可选前缀](#configurehostconfiguration)）

如果路径不存在，主机将无法启动。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseContentRoot)]

有关详细信息，请参阅[基础知识：内容根目录](xref:fundamentals/index#content-root)。

### <a name="environment"></a>环境

设置应用的[环境](xref:fundamentals/environments)。

键：`environment`  
类型：`string`  
**默认**：`Production`  
**设置使用**：`UseEnvironment`  
**环境变量**：`<PREFIX_>ENVIRONMENT`（`<PREFIX_>` 是[用户定义的可选前缀](#configurehostconfiguration)）

可将环境设置为任何值。 框架定义的值包括 `Development``Staging` 和 `Production`。 值不区分大小写。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseEnvironment)]

### <a name="configurehostconfiguration"></a>ConfigureHostConfiguration

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> 使用 <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> 来为主机创建 <xref:Microsoft.Extensions.Configuration.IConfiguration>。 主机配置用于初始化 <xref:Microsoft.Extensions.Hosting.IHostingEnvironment>，以供在应用的构建过程中使用。

可多次调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>，并得到累计结果。 主机使用上一次在一个给定键上设置值的选项。

默认情况下不包括提供程序。 必须在 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*> 中显式指定应用所需的任何配置提供程序，包括：

* 文件配置（例如，来自 hostsettings.json 文件）。
* 环境变量配置。
* 命令行参数配置。
* 任何其他所需的配置提供程序。

通过使用 `SetBasePath` 指定应用的基本路径，然后调用其中一个[文件配置提供程序](xref:fundamentals/configuration/index#file-configuration-provider)，可以启用主机的文件配置。 示例应用使用 JSON 文件 hostsettings.json，并调用 <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> 来使用文件的主机配置设置。

要添加主机的[环境变量配置](xref:fundamentals/configuration/index#environment-variables-configuration-provider)，请在主机生成器上调用 <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*>。 `AddEnvironmentVariables` 接受用户定义的前缀（可选）。 示例应用使用前缀 `PREFIX_`。 当系统读取环境变量时，便会删除前缀。 配置示例应用的主机后，`PREFIX_ENVIRONMENT` 的环境变量值就变成 `environment` 密钥的主机配置值。

在开发过程中，如果使用 [Visual Studio](https://visualstudio.microsoft.com) 或通过 `dotnet run` 运行应用，可能会在 Properties/launchSettings.json 文件中设置环境变量。 若在开发过程中使用 [Visual Studio Code](https://code.visualstudio.com/)，可能会在 .vscode/launch.json 文件中设置环境变量。 有关详细信息，请参阅 <xref:fundamentals/environments>。

通过调用 <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> 可添加[命令行配置](xref:fundamentals/configuration/index#command-line-configuration-provider)。 最后添加命令行配置以允许命令行参数替代之前配置提供程序提供的配置。

hostsettings.json：

[!code-json[](generic-host/samples/2.x/GenericHostSample/hostsettings.json)]

可以通过 [applicationName](#application-key-name) 和 [contentRoot](#content-root) 键提供其他配置。

示例 `HostBuilder` 配置使用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureHostConfiguration*>：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureHostConfiguration)]

## <a name="configureappconfiguration"></a>ConfigureAppConfiguration

通过在 <xref:Microsoft.Extensions.Hosting.IHostBuilder> 实现上调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 创建应用配置。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 使用 <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> 来为应用创建 <xref:Microsoft.Extensions.Configuration.IConfiguration>。 可多次调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>，并得到累计结果。 应用使用上一次在一个给定键上设置值的选项。 [HostBuilderContext.Configuration](xref:Microsoft.Extensions.Hosting.HostBuilderContext.Configuration*) 中提供 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 创建的配置，以供进行后续操作和在 <xref:Microsoft.Extensions.Hosting.IHost.Services*> 中使用。

应用配置会自动接收 [ConfigureHostConfiguration](#configurehostconfiguration) 提供的主机配置。

示例应用配置使用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*>：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureAppConfiguration)]

appsettings.json：

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.json)]

appsettings.Development.json：

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Development.json)]

appsettings.Production.json：

[!code-json[](generic-host/samples/2.x/GenericHostSample/appsettings.Production.json)]

要将设置文件移动到输出目录，请在项目文件中将设置文件指定为 [MSBuild 项目项](/visualstudio/msbuild/common-msbuild-project-items)。 示例应用移动具有以下 `<Content>` 项的 JSON 应用设置文件和 hostsettings.json：

```xml
<ItemGroup>
  <Content Include="**\*.json" Exclude="bin\**\*;obj\**\*" 
      CopyToOutputDirectory="PreserveNewest" />
</ItemGroup>
```

> [!NOTE]
> 配置扩展方法（如 <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> 和 <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*>）需要其他 NuGet 包（如 [Microsoft.Extensions.Configuration.Json](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.Json) 和 [Microsoft.Extensions.Configuration.EnvironmentVariables](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.EnvironmentVariables)）。 除非应用使用 [Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)，否则除了核心 [Microsoft.Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration) 包之外，还必须将这些包添加到项目。 有关详细信息，请参阅 <xref:fundamentals/configuration/index>。

## <a name="configureservices"></a>ConfigureServices

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*> 将服务添加到应用的[依赖关系](xref:fundamentals/dependency-injection)注入容器。 可多次调用 <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureServices*>，并得到累计结果。

托管服务是一个类，具有实现 <xref:Microsoft.Extensions.Hosting.IHostedService> 接口的后台任务逻辑。 有关详细信息，请参阅 <xref:fundamentals/host/hosted-services>。

[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/)使用 `AddHostedService` 扩展方法向应用添加生存期事件 `LifetimeEventsHostedService` 和定时后台任务 `TimedHostedService` 服务：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureServices)]

## <a name="configurelogging"></a>ConfigureLogging

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*> 添加了一个委托来配置提供的 <xref:Microsoft.Extensions.Logging.ILoggingBuilder>。 可以利用相加结果多次调用 <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.ConfigureLogging*>。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ConfigureLogging)]

### <a name="useconsolelifetime"></a>UseConsoleLifetime

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> 侦听 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM 并调用 <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> 来启动关闭进程。 <xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.UseConsoleLifetime*> 解除阻止 [ RunAsync](#runasync) 和 [WaitForShutdownAsync](#waitforshutdownasync) 等扩展。 `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime` 预注册为默认生存期实现。 使用注册的最后一个生存期。

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_UseConsoleLifetime)]

## <a name="container-configuration"></a>容器配置

为支持插入其他容器中，主机可以接受 <xref:Microsoft.Extensions.DependencyInjection.IServiceProviderFactory%601>。 提供工厂不属于 DI 容器注册，而是用于创建具体 DI 容器的主机内部函数。 [UseServiceProviderFactory(IServiceProviderFactory&lt;TContainerBuilder&gt;)](xref:Microsoft.Extensions.Hosting.HostBuilder.UseServiceProviderFactory*) 重写用于创建应用的服务提供程序的默认工厂。

<xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> 方法托管自定义容器配置。 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*> 提供在基础主机 API 的基础之上配置容器的强类型体验。 可以利用相加结果多次调用 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureContainer*>。

为应用创建服务容器：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainer.cs)]

提供服务容器工厂：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/ServiceContainerFactory.cs)]

使用该工厂并为应用配置自定义服务容器：

[!code-csharp[](generic-host/samples-snapshot/2.x/GenericHostSample/Program.cs?name=snippet_ContainerConfiguration)]

## <a name="extensibility"></a>扩展性

在 <xref:Microsoft.Extensions.Hosting.IHostBuilder> 上使用扩展方法实现主机扩展性。 以下示例介绍扩展方法如何使用 <xref:fundamentals/host/hosted-services> 中所示的 [TimedHostedService](xref:fundamentals/host/hosted-services#timed-background-tasks) 示例来扩展 <xref:Microsoft.Extensions.Hosting.IHostBuilder> 实现。

```csharp
var host = new HostBuilder()
    .UseHostedService<TimedHostedService>()
    .Build();

await host.StartAsync();
```

应用建立 `UseHostedService` 扩展方法，以注册在 `T` 中传递的托管服务：

```csharp
using System;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

public static class Extensions
{
    public static IHostBuilder UseHostedService<T>(this IHostBuilder hostBuilder)
        where T : class, IHostedService, IDisposable
    {
        return hostBuilder.ConfigureServices(services =>
            services.AddHostedService<T>());
    }
}
```

## <a name="manage-the-host"></a>管理主机

<xref:Microsoft.Extensions.Hosting.IHost> 实现负责启动和停止服务容器中注册的 <xref:Microsoft.Extensions.Hosting.IHostedService> 实现。

### <a name="run"></a>运行

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Run*> 运行应用并阻止调用线程，直到关闭主机：

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        host.Run();
    }
}
```

### <a name="runasync"></a>RunAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.RunAsync*> 运行应用并返回在触发取消令牌或关闭时完成的 <xref:System.Threading.Tasks.Task>：

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        await host.RunAsync();
    }
}
```

### <a name="runconsoleasync"></a>RunConsoleAsync

<xref:Microsoft.Extensions.Hosting.HostingHostBuilderExtensions.RunConsoleAsync*> 启用控制台支持、生成和启动主机，以及等待 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM 关闭。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var hostBuilder = new HostBuilder();

        await hostBuilder.RunConsoleAsync();
    }
}
```

### <a name="start-and-stopasync"></a>Start 和 StopAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.Start*> 同步启动主机。

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.StopAsync*> 尝试在提供的超时时间内停止主机。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            await host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

### <a name="startasync-and-stopasync"></a>StartAsync 和 StopAsync

<xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> 启动应用。

<xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 停止应用。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.StopAsync();
        }
    }
}
```

### <a name="waitforshutdown"></a>WaitForShutdown

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> 通过 <xref:Microsoft.Extensions.Hosting.IHostLifetime> 触发，例如 `Microsoft.Extensions.Hosting.Internal.ConsoleLifetime`（侦听 <kbd>Ctrl</kbd>+<kbd>C</kbd>/SIGINT 或 SIGTERM）。 <xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdown*> 调用 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*>。

```csharp
public class Program
{
    public void Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            host.Start();

            host.WaitForShutdown();
        }
    }
}
```

### <a name="waitforshutdownasync"></a>WaitForShutdownAsync

<xref:Microsoft.Extensions.Hosting.HostingAbstractionsHostExtensions.WaitForShutdownAsync*> 返回在通过给定的令牌和调用 <xref:Microsoft.Extensions.Hosting.IHost.StopAsync*> 来触发关闭时完成的 <xref:System.Threading.Tasks.Task>。

```csharp
public class Program
{
    public static async Task Main(string[] args)
    {
        var host = new HostBuilder()
            .Build();

        using (host)
        {
            await host.StartAsync();

            await host.WaitForShutdownAsync();
        }

    }
}
```

### <a name="external-control"></a>外部控件

使用可从外部调用的方法，能够实现主机的外部控件：

```csharp
public class Program
{
    private IHost _host;

    public Program()
    {
        _host = new HostBuilder()
            .Build();
    }

    public async Task StartAsync()
    {
        _host.StartAsync();
    }

    public async Task StopAsync()
    {
        using (_host)
        {
            await _host.StopAsync(TimeSpan.FromSeconds(5));
        }
    }
}
```

在 <xref:Microsoft.Extensions.Hosting.IHost.StartAsync*> 开始时调用 <xref:Microsoft.Extensions.Hosting.IHostLifetime.WaitForStartAsync*>，在继续之前，会一直等待该操作完成。 它可用于延迟启动，直到外部事件发出信号。

## <a name="ihostingenvironment-interface"></a>IHostingEnvironment 接口

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment> 提供有关应用托管环境的信息。 使用[构造函数注入](xref:fundamentals/dependency-injection)获取 <xref:Microsoft.Extensions.Hosting.IHostingEnvironment> 以使用其属性和扩展方法：

```csharp
public class MyClass
{
    private readonly IHostingEnvironment _env;

    public MyClass(IHostingEnvironment env)
    {
        _env = env;
    }

    public void DoSomething()
    {
        var environmentName = _env.EnvironmentName;
    }
}
```

有关详细信息，请参阅 <xref:fundamentals/environments>。

## <a name="iapplicationlifetime-interface"></a>IApplicationLifetime 接口

<xref:Microsoft.Extensions.Hosting.IApplicationLifetime> 允许启动后和关闭活动，包括正常关闭请求。 接口上的三个属性是用于注册 <xref:System.Action> 方法（用于定义启动和关闭事件）的取消标记。

| 取消标记 | 触发条件 |
| ------------------ | --------------------- |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStarted*> | 主机已完全启动。 |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopped*> | 主机正在完成正常关闭。 应处理所有请求。 关闭受到阻止，直到完成此事件。 |
| <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.ApplicationStopping*> | 主机正在执行正常关闭。 仍在处理请求。 关闭受到阻止，直到完成此事件。 |

构造函数将 <xref:Microsoft.Extensions.Hosting.IApplicationLifetime> 服务注入到任何类中。 [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/host/generic-host/samples/)将构造函数注入到 `LifetimeEventsHostedService` 类（一个 <xref:Microsoft.Extensions.Hosting.IHostedService> 实现）中，用于注册事件。

LifetimeEventsHostedService.cs：

[!code-csharp[](generic-host/samples/2.x/GenericHostSample/LifetimeEventsHostedService.cs?name=snippet1)]

<xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> 请求终止应用。 以下类在调用类的 `Shutdown` 方法时使用 <xref:Microsoft.Extensions.Hosting.IApplicationLifetime.StopApplication*> 正常关闭应用：

```csharp
public class MyClass
{
    private readonly IApplicationLifetime _appLifetime;

    public MyClass(IApplicationLifetime appLifetime)
    {
        _appLifetime = appLifetime;
    }

    public void Shutdown()
    {
        _appLifetime.StopApplication();
    }
}
```

::: moniker-end

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/host/hosted-services>
