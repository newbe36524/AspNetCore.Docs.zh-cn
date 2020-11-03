---
title: ASP.NET Core 模块
author: rick-anderson
description: 了解用于通过 IIS 托管 ASP.NET Core 应用的 ASP.NET Core 模块。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2020
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
uid: host-and-deploy/aspnet-core-module
ms.openlocfilehash: 8ee9ab2b598bc8ff62faa45a5666615ee7ab239b
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91754666"
---
# <a name="aspnet-core-module"></a>ASP.NET Core 模块

作者：[Tom Dykstra](https://github.com/tdykstra)、[Rick Strahl](https://github.com/RickStrahl)、[Chris Ross](https://github.com/Tratcher)、[Rick Anderson](https://twitter.com/RickAndMSFT)、[Sourabh Shirhatti](https://twitter.com/sshirhatti) 和 [Justin Kotalik](https://github.com/jkotalik)

::: moniker range=">= aspnetcore-5.0"

ASP.NET Core 模块是插入 IIS 管道的本机 IIS 模块，能让 ASP.NET Core 应用程序通过 IIS 运行。 使用以下任一方式通过 IIS 运行 ASP.NET Core 应用： 

* 在 IIS 工作进程 (`w3wp.exe`) 内托管 ASP.NET Core 应用，称为[进程内托管模型](xref:host-and-deploy/iis/in-process-hosting)。
* 将 Web 请求转发到运行 Kestrel 服务器的后端 ASP.NET Core 应用，称为[进程外托管模型](xref:host-and-deploy/iis/out-of-process-hosting)。

我们需要对这两类托管模型进行权衡。 默认情况下使用的是进程内托管模型，因为这样可以得到更好的性能和诊断。

## <a name="install-aspnet-core-module"></a>安装 ASP.NET Core 模块

使用以下链接下载安装程序：

[当前 .NET Core 托管捆绑包安装程序（直接下载）](https://dotnet.microsoft.com/permalink/dotnetcore-current-windows-runtime-bundle-installer)

若要详细了解如何安装 ASP.NET Core 模块或不同版本的模块，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/hosting-bundle)。

若要学习将 ASP.NET Core 应用发布到 IIS 服务器的教程，请参阅<xref:tutorials/publish-to-iis>。

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

ASP.NET Core 模块是插入 IIS 管道的本机 IIS 模块，用于：

* 在 IIS 工作进程 (`w3wp.exe`) 内托管 ASP.NET Core 应用，称为[进程内托管模型](#in-process-hosting-model)。
* 将 Web 请求转发到运行 [Kestrel 服务器](xref:fundamentals/servers/kestrel)的后端 ASP.NET Core 应用，称为[进程外托管模型](#out-of-process-hosting-model)。

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2012 R2 或更高版本

在进程内托管时，该模块会使用 IIS 进程内服务器实现，即 IIS HTTP 服务器 (`IISHttpServer`)。

在进程外托管时，该模块仅适用于 Kestrel。 该模块无法与 [HTTP.sys](xref:fundamentals/servers/httpsys) 一起工作。

## <a name="hosting-models"></a>托管模型

### <a name="in-process-hosting-model"></a>进程内托管模型

ASP.NET Core 应用默认为进程内托管模型。

在进程内托管时，将应用以下特征：

* 使用 IIS HTTP 服务器 (`IISHttpServer`) 而不是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器。 在进程内，[CreateDefaultBuilder](xref:fundamentals/host/generic-host#default-builder-settings) 调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> 以执行以下操作：

  * 注册 `IISHttpServer`。
  * 在 ASP.NET Core 模块后运行时，配置服务器应侦听的端口和基本路径。
  * 配置主机以捕获启动错误。

* [requestTimeout 属性](#attributes-of-the-aspnetcore-element)不适用于进程内托管。

* 不支持在应用之间共享应用池。 每个应用使用一个应用池。

* 使用 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)或手动将 [`app_offline.htm` 文件置于部署中](xref:host-and-deploy/iis/index#locked-deployment-files)时，如果有已打开的连接，则应用可能不会立即关闭。 例如，WebSocket 连接可能会延迟应用关闭。

* 应用和已安装的运行时（x64 或 x86）的体系结构（位数）必须与应用池的体系结构匹配。

* 检测到客户端连接断开。 客户端断开连接时，将取消 [`HttpContext.RequestAborted`](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) 取消标记。

* 在 ASP.NET Core 2.2.1 或更早版本中，<xref:System.IO.Directory.GetCurrentDirectory*> 会返回 IIS 所启动进程的工作目录而非应用目录（例如，对于 `w3wp.exe`，是 `C:\Windows\System32\inetsrv`）。

  对于设置应用的当前目录的示例代码，请参阅 [`CurrentDirectoryHelpers` 类](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/3.x/CurrentDirectoryHelpers.cs)。 调用 `SetCurrentDirectory` 方法。 后续 <xref:System.IO.Directory.GetCurrentDirectory*> 调用提供应用的目录。

* 在进程内托管时，不会在内部调用 <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> 来初始化用户。 因此，默认情况下不激活每次身份验证后用于转换声明的 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 实现。 使用 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 实现转换声明时，请调用 <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> 以添加身份验证服务：

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

### <a name="out-of-process-hosting-model"></a>进程外托管模型

若要配置进程外托管应用，请在项目文件 ( `.csproj`) 中将 `<AspNetCoreHostingModel>` 属性的值设置为 `OutOfProcess`：

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

进程内托管设为 `InProcess`，这是默认值。

`<AspNetCoreHostingModel>` 的值不区分大小写，因此 `inprocess` 和 `outofprocess` 均为有效值。

使用 [Kestrel](xref:fundamentals/servers/kestrel) 服务器，而不是 IIS HTTP 服务器 (`IISHttpServer`)。

对于进程外托管，[`CreateDefaultBuilder`](xref:fundamentals/host/generic-host#default-builder-settings) 会调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> 来进行以下操作：

* 在 ASP.NET Core 模块后运行时，配置服务器应侦听的端口和基本路径。
* 配置主机以捕获启动错误。

### <a name="hosting-model-changes"></a>托管模型更改

如果在 `web.config` 文件中更改了 `hostingModel` 设置（如 [`web.config`web.config 的配置](#configuration-with-webconfig)部分中所述），则该模块会再循环 IIS 工作进程。

对于 IIS Express，该模块不会再循环工作进程，但改为触发当前 IIS Express 进程的正常关闭。 应用的下一个请求会生成新的 IIS Express 进程。

### <a name="process-name"></a>进程名

`Process.GetCurrentProcess().ProcessName` 报告 `w3wp`/`iisexpress`（进程内）或 `dotnet`（进程外）。

许多本机模块（如 Windows 身份验证）仍处于活动状态。 要详细了解随 ASP.NET Core 模块活动的 IIS 模块，请参阅 <xref:host-and-deploy/iis/modules>。

ASP.NET Core 模块还可以：

* 为工作进程设置环境变量。
* 将 stdout 输出记录到文件存储器，以解决启动问题。
* 转发 Windows 身份验证令牌。

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>如何安装和使用 ASP.NET Core 模块

有关如何安装 ASP.NET Core 模块的说明，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

## <a name="configuration-with-webconfig"></a>web.config 的配置

在站点的 *web.config* 文件中使用 `system.webServer` 节点的 `aspNetCore` 部分配置 ASP.NET Core 模块。

以下 `web.config` 文件发布用于[依赖框架的部署](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd)，并配置 ASP.NET Core 模块以处理站点请求：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\MyApp.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

以下 web.config 发布用于[独立部署](/dotnet/articles/core/deploying/#self-contained-deployments-scd)：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath=".\MyApp.exe"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

将 <xref:System.Configuration.SectionInformation.InheritInChildApplications*> 属性设置为 `false`，表示 [`<location>`](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) 元素中指定的设置不会由驻留在应用子目录中的应用继承。

将应用部署为 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)时，`stdoutLogFile` 路径将设置为 `\\?\%home%\LogFiles\stdout`。 该路径会将 stdout 日志保存到 `LogFiles` 文件夹，该文件夹是由服务自动创建的位置。

有关 IIS 子应用程序配置的信息，请参阅<xref:host-and-deploy/iis/index#sub-applications>。

### <a name="attributes-of-the-aspnetcore-element"></a>aspNetCore 元素的属性

| 特性 | 描述 | 默认 |
| --------- | ----------- | :-----: |
| `arguments` | <p>可选的字符串属性。</p><p>processPath 中指定的可执行文件的参数。</p> | |
| `disableStartUpErrorPage` | <p>可选布尔属性。</p><p>如果为 true，将禁止显示“502.5 - 进程失败”页面，而会优先显示 web.config 中配置的 502 状态代码页面。</p> | `false` |
| `forwardWindowsAuthToken` | <p>可选布尔属性。</p><p>如果为 true，会将令牌作为每个请求的标头 `'MS-ASPNETCORE-WINAUTHTOKEN'` 转发到在 `%ASPNETCORE_PORT%` 上侦听的子进程。 该进程负责在每个请求的此令牌上调用 CloseHandle。</p> | `true` |
| `hostingModel` | <p>可选的字符串属性。</p><p>将托管模型指定为进程内 (`InProcess`/`inprocess`) 或进程外 (`OutOfProcess`/`outofprocess`)。</p> | `InProcess`<br>`inprocess` |
| `processesPerApplication` | <p>可选的整数属性。</p><p>指定每个应用均可启动的 **processPath** 设置中指定的进程的实例数。</p><p>&dagger;对于进程内托管，值限制为 `1`。</p><p>不建议设置 `processesPerApplication`。 将来的版本将删除此属性。</p> | 默认值：`1`<br>最小值：`1`<br>最大值：`100`&dagger; |
| `processPath` | <p>必需的字符串属性。</p><p>为 HTTP 请求启动进程侦听的可执行文件的路径。 支持相对路径。 如果路径以 `.` 开头，则该路径被视为与站点根目录相对。</p> | |
| `rapidFailsPerMinute` | <p>可选的整数属性。</p><p>指定允许 processPath 中指定的进程每分钟崩溃的次数。 如果超出了此限制，模块将在剩余分钟数内停止启动该进程。</p><p>不支持进程内托管。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`100` |
| `requestTimeout` | <p>可选的 timespan 属性。</p><p>指定 ASP.NET Core 模块等待来自 %ASPNETCORE_PORT% 上侦听的进程的响应的持续时间。</p><p>在 ASP.NET Core 2.1 或更高版本附带的 ASP.NET Core 模块版本中，使用小时数、分钟数和秒数指定 `requestTimeout`。</p><p>不适用于进程内托管。 对于进程内托管，该模块等待应用处理该请求。</p><p>此字符串的分钟段和秒钟段的有效值在 0-59 之间。 在分钟或秒钟值中使用“60”会导致“500 - 内部服务器错误”。</p> | 默认值：`00:02:00`<br>最小值：`00:00:00`<br>最大值：`360:00:00` |
| `shutdownTimeLimit` | <p>可选的整数属性。</p><p>检测到 app_offline.htm 文件时，模块等待可执行文件正常关闭的持续时间（以秒为单位）。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`600` |
| `startupTimeLimit` | <p>可选的整数属性。</p><p>模块等待可执行文件启动端口上侦听的进程的持续时间（以秒为单位）。 如果超出了此时间限制，模块将终止该进程。</p><p>进程内托管时：不会重新启动该进程，也不会使用 rapidFailsPerMinute 设置  。</p><p>进程外托管时：模块在收到新请求时尝试重新启动该进程，并在收到后续传入请求时继续尝试重新启动该进程，除非应用在上一回滚分钟内无法启动 rapidFailsPerMinute 次。</p><p>值 0（零）不被视为无限超时。</p> | 默认值：`120`<br>最小值：`0`<br>最大值：`3600` |
| `stdoutLogEnabled` | <p>可选布尔属性。</p><p>如果为 true，processPath 中指定的 进程的 stdout 和 stderr 将重定向到 stdoutLogFile 中指定的文件。</p> | `false` |
| `stdoutLogFile` | <p>可选的字符串属性。</p><p>指定在其中记录 processPath 中指定进程的 stdout 和 stderr 的相对路径或绝对路径。 相对路径与站点根目录相对。 以 `.` 开头的任何路径均与站点根目录相对，所有其他路径被视为绝对路径。 创建日志文件时，模块会创建路径中提供的所有文件夹。 使用下划线分隔符，将时间戳、进程 ID 和文件扩展名 (`.log`) 添加到 stdoutLogFile 路径的最后一段。 如果 `.\logs\stdout` 作为值提供，则在示例 stdout 日志使用进程 ID 1934 于 2018 年 2 月 5 日 19:41:32 保存时，将在 `logs` 文件夹中保存为 `stdout_20180205194132_1934.log`。</p> | `aspnetcore-stdout` |

### <a name="set-environment-variables"></a>设置环境变量

可以为 `processPath` 属性中的进程指定环境变量。 使用 `<environmentVariables>` 集合元素的 `<environmentVariable>` 子元素指定环境变量。 本部分中设置的环境变量优先于系统环境变量。

以下示例在 `web.config` 中设置了两个环境变量。 `ASPNETCORE_ENVIRONMENT` 将应用的环境配置为 `Development`。 开发人员可能会暂时在 `web.config` 文件中设置此值，以便在调试应用异常时强制加载[开发人员异常页面](xref:fundamentals/error-handling)。 `CONFIG_DIR` 是用户定义的环境变量的一个示例，其中开发人员已写入可在启动时读取值的代码以便形成用于加载应用配置文件的路径。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!NOTE]
> 直接在 `web.config` 中设置环境的替代方法是将 `<EnvironmentName>` 属性包含在[发布配置文件 (`.pubxml`)](xref:host-and-deploy/visual-studio-publish-profiles) 或项目文件中。 此方法在发布项目时设置 `web.config` 中的环境：
>
> ```xml
> <PropertyGroup>
>   <EnvironmentName>Development</EnvironmentName>
> </PropertyGroup>
> ```

> [!WARNING]
> 仅当不受信任的网络（如 Internet）无法访问暂存和测试服务器时，才能将 `ASPNETCORE_ENVIRONMENT` 环境变量设置为 `Development`。

## `app_offline.htm`

如果在应用的根目录中检测到名为“`app_offline.htm`”的文件，ASP.NET Core 模块将尝试正常关闭应用并停止处理传入请求。 如果应用在 `shutdownTimeLimit` 中定义的秒数之后仍在运行，ASP.NET Core 模块将终止正在运行的进程。

存在 `app_offline.htm` 文件时，ASP.NET Core 模块会通过发送回 `app_offline.htm` 文件的内容来响应请求。 删除 `app_offline.htm` 文件后，下一个请求将启动应用。

使用进程外托管模型时，如果有已打开的连接，则应用可能不会立即关闭。 例如，WebSocket 连接可能会延迟应用关闭。

## <a name="start-up-error-page"></a>启动错误页面

进程内和进程外托管在无法启动应用时均会生成自定义错误页面。

如果 ASP.NET Core 模块无法找到进程内或进程外请求处理程序，则会显示“500.0 - 进程内/进程外处理程序加载失败”状态代码页。

对于进程内托管，如果 ASP.NET Core 模块无法启动应用，则会显示“500.30 - 启动失败”状态代码页。

对于进程外托管，如果 ASP.NET Core 模块无法启动后端进程或后端进程启动但无法在配置的端口上侦听，则将显示“502.5 - 进程失败”状态代码页。

若要禁止显示此页面并还原为默认 IIS 5xx 状态代码页，请使用 `disableStartUpErrorPage` 属性。 有关配置自定义错误消息的详细信息，请参阅 [HTTP 错误 `<httpErrors>`](/iis/configuration/system.webServer/httpErrors/)。

## <a name="log-creation-and-redirection"></a>日志创建和重定向

如果已设置 `aspNetCore` 元素的 `stdoutLogEnabled` 和 `stdoutLogFile` 属性，ASP.NET Core 模块会将 stdout 和 stderr 控制台输出重定向到磁盘。 创建日志文件时，模块会创建 `stdoutLogFile` 路径中提供的所有文件夹。 应用池必须对写入日志的位置具有写入权限（使用 `IIS AppPool\<app_pool_name>` 提供写入权限）。

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

有关 `web.config` 文件中的 `aspNetCore` 元素的示例，请参阅 [web.config 的配置](#configuration-with-webconfig)。

## <a name="modify-the-stack-size"></a>修改堆栈大小

*仅适用于使用进程内托管模型的情况。*

在 `web.config` 中使用 `stackSize` 设置（以字节为单位）配置托管堆栈大小。 默认大小为 1,048,576 字节 (1 MB)。

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

ASP.NET Core 模块安装程序使用 TrustedInstaller 帐户的权限运行。 由于本地系统帐户没有 IIS 共享配置所用的共享路径的修改权限，因此在尝试配置共享上的 `applicationHost.config` 文件中的模块设置时，安装程序将引发拒绝访问错误。

在与 IIS 安装相同的计算机上使用 IIS 共享配置时，请运行 ASP.NET Core Hosting Bundle 安装程序，并将 `OPT_NO_SHARED_CONFIG_CHECK` 参数设置为 `1`：

```console
dotnet-hosting-{VERSION}.exe OPT_NO_SHARED_CONFIG_CHECK=1
```

如果共享配置的路径与 IIS 安装的路径不在同一台计算机上，请按照下列步骤操作：

1. 禁用 IIS 共享配置。
1. 运行安装程序。
1. 将已更新的 `applicationHost.config` 文件导出到共享。
1. 重新启用 IIS 共享配置。

## <a name="module-version-and-hosting-bundle-installer-logs"></a>模块版本和托管捆绑安装程序日志

若要确定已安装 ASP.NET Core 模块的版本，请执行以下操作：

1. 在托管系统上，导航到 `%windir%\System32\inetsrv`。
1. 找到 `aspnetcore.dll` 文件。
1. 右键单击该文件，然后从上下文菜单中选择“属性”。
1. 选择“详细信息”选项卡。“文件版本”和“产品版本”表示模块的已安装版本。

可在 `C:\Users\%UserName%\AppData\Local\Temp` 中找到模块的托管捆绑安装程序日志。 该文件命名为 `dd_DotNetCoreWinSvrHosting__{TIMESTAMP}_000_AspNetCoreModule_x64.log`。

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

::: moniker-end

::: moniker range="= aspnetcore-2.2"

ASP.NET Core 模块是插入 IIS 管道的本机 IIS 模块，用于：

* 在 IIS 工作进程 (`w3wp.exe`) 内托管 ASP.NET Core 应用，称为[进程内托管模型](#in-process-hosting-model)。
* 将 Web 请求转发到运行 [Kestrel 服务器](xref:fundamentals/servers/kestrel)的后端 ASP.NET Core 应用，称为[进程外托管模型](#out-of-process-hosting-model)。

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

在进程内托管时，该模块会使用 IIS 进程内服务器实现，即 IIS HTTP 服务器 (`IISHttpServer`)。

在进程外托管时，该模块仅适用于 Kestrel。 该模块无法与 [HTTP.sys](xref:fundamentals/servers/httpsys) 一起工作。

## <a name="hosting-models"></a>托管模型

### <a name="in-process-hosting-model"></a>进程内托管模型

若要配置用于进程内托管的应用，请将 `<AspNetCoreHostingModel>` 属性添加到值为 `InProcess`（进程外托管使用 `OutOfProcess` 进行设置）的应用项目文件：

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>InProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

定目标到 .NET Framework 的 ASP.NET Core 应用不支持进程内托管模型。

`<AspNetCoreHostingModel>` 的值不区分大小写，因此 `inprocess` 和 `outofprocess` 均为有效值。

如果文件中不存在 `<AspNetCoreHostingModel>` 属性，则默认值为 `OutOfProcess`。

在进程内托管时，将应用以下特征：

* 使用 IIS HTTP 服务器 (`IISHttpServer`) 而不是 [Kestrel](xref:fundamentals/servers/kestrel) 服务器。 在进程内，[CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) 调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIIS*> 以执行以下操作：

  * 注册 `IISHttpServer`。
  * 在 ASP.NET Core 模块后运行时，配置服务器应侦听的端口和基本路径。
  * 配置主机以捕获启动错误。

* [requestTimeout 属性](#attributes-of-the-aspnetcore-element)不适用于进程内托管。

* 不支持在应用之间共享应用池。 每个应用使用一个应用池。

* 使用 [Web 部署](/iis/publish/using-web-deploy/introduction-to-web-deploy)或手动将 [app_offline.htm 文件置于部署中](xref:host-and-deploy/iis/index#locked-deployment-files)时，如果有已打开的连接，则应用可能不会立即关闭。 例如，WebSocket 连接可能会延迟应用关闭。

* 应用和已安装的运行时（x64 或 x86）的体系结构（位数）必须与应用池的体系结构匹配。

* 检测到客户端连接断开。 客户端连接断开时，[HttpContext.RequestAborted](xref:Microsoft.AspNetCore.Http.HttpContext.RequestAborted*) 取消标记将会取消。

* 在 ASP.NET Core 2.2.1 或早期版本中，<xref:System.IO.Directory.GetCurrentDirectory*> 会返回 IIS 启动的进程的工作目录而非应用目录（例如，对于 w3wp.exe，是 C:\Windows\System32\inetsrv ）。

  对于设置应用的当前目录的示例代码，请参阅 [CurrentDirectoryHelpers 类](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/aspnet-core-module/samples_snapshot/2.x/CurrentDirectoryHelpers.cs)。 调用 `SetCurrentDirectory` 方法。 后续 <xref:System.IO.Directory.GetCurrentDirectory*> 调用提供应用的目录。

* 在进程内托管时，不会在内部调用 <xref:Microsoft.AspNetCore.Authentication.AuthenticationService.AuthenticateAsync*> 来初始化用户。 因此，默认情况下不激活每次身份验证后用于转换声明的 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 实现。 使用 <xref:Microsoft.AspNetCore.Authentication.IClaimsTransformation> 实现转换声明时，请调用 <xref:Microsoft.Extensions.DependencyInjection.AuthenticationServiceCollectionExtensions.AddAuthentication*> 以添加身份验证服务：

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

### <a name="out-of-process-hosting-model"></a>进程外托管模型

若要配置进程外托管应用，请在项目文件中使用以下方法之一：

* 不指定 `<AspNetCoreHostingModel>` 属性。 如果文件中不存在 `<AspNetCoreHostingModel>` 属性，则默认值为 `OutOfProcess`。
* 将 `<AspNetCoreHostingModel>` 属性的值设为 `OutOfProcess`（进程内托管设为 `InProcess`）：

```xml
<PropertyGroup>
  <AspNetCoreHostingModel>OutOfProcess</AspNetCoreHostingModel>
</PropertyGroup>
```

值不区分大小写，因此 `inprocess` 和 `outofprocess` 均为有效值。

使用 [Kestrel](xref:fundamentals/servers/kestrel) 服务器，而不是 IIS HTTP 服务器 (`IISHttpServer`)。

在进程外，[CreateDefaultBuilder](xref:fundamentals/host/web-host#set-up-a-host) 调用 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilderIISExtensions.UseIISIntegration*> 以执行以下操作：

* 在 ASP.NET Core 模块后运行时，配置服务器应侦听的端口和基本路径。
* 配置主机以捕获启动错误。

### <a name="hosting-model-changes"></a>托管模型更改

如果 `hostingModel` 设置在 web.config 文件中被更改（如 [web.config 的配置](#configuration-with-webconfig)部分中所述），则该模块会再循环 IIS 工作进程。

对于 IIS Express，该模块不会再循环工作进程，但改为触发当前 IIS Express 进程的正常关闭。 应用的下一个请求会生成新的 IIS Express 进程。

### <a name="process-name"></a>进程名

`Process.GetCurrentProcess().ProcessName` 报告 `w3wp`/`iisexpress`（进程内）或 `dotnet`（进程外）。

许多本机模块（如 Windows 身份验证）仍处于活动状态。 要详细了解随 ASP.NET Core 模块活动的 IIS 模块，请参阅 <xref:host-and-deploy/iis/modules>。

ASP.NET Core 模块还可以：

* 为工作进程设置环境变量。
* 将 stdout 输出记录到文件存储器，以解决启动问题。
* 转发 Windows 身份验证令牌。

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>如何安装和使用 ASP.NET Core 模块

有关如何安装 ASP.NET Core 模块的说明，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

## <a name="configuration-with-webconfig"></a>web.config 的配置

在站点的 *web.config* 文件中使用 `system.webServer` 节点的 `aspNetCore` 部分配置 ASP.NET Core 模块。

以下 *web.config* 文件发布用于[依赖框架的部署](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd)，并配置 ASP.NET Core 模块以处理站点请求：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet"
                  arguments=".\MyApp.dll"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

以下 web.config 发布用于[独立部署](/dotnet/articles/core/deploying/#self-contained-deployments-scd)：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath=".\MyApp.exe"
                  stdoutLogEnabled="false"
                  stdoutLogFile=".\logs\stdout"
                  hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
```

将 <xref:System.Configuration.SectionInformation.InheritInChildApplications*> 属性设置为 `false`，表示 [\<location>](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) 元素中指定的设置不会由驻留在应用子目录中的应用继承。

将应用部署为 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)时，`stdoutLogFile` 路径将设置为 `\\?\%home%\LogFiles\stdout`。 该路径会将 stdout 日志保存到 *LogFiles* 文件夹，该文件夹是由服务自动创建的位置。

有关 IIS 子应用程序配置的信息，请参阅<xref:host-and-deploy/iis/index#sub-applications>。

### <a name="attributes-of-the-aspnetcore-element"></a>aspNetCore 元素的属性

| 特性 | 描述 | 默认 |
| --------- | ----------- | :-----: |
| `arguments` | <p>可选的字符串属性。</p><p>`processPath` 中指定的可执行文件的参数。</p> | |
| `disableStartUpErrorPage` | <p>可选布尔属性。</p><p>如果为 true，将禁止显示“502.5 - 进程失败”页面，而会优先显示 web.config 中配置的 502 状态代码页面。</p> | `false` |
| `forwardWindowsAuthToken` | <p>可选布尔属性。</p><p>如果为 true，会将令牌作为每个请求的标头“MS-ASPNETCORE-WINAUTHTOKEN”，转发到在 %ASPNETCORE_PORT% 上侦听的子进程。 该进程负责在每个请求的此令牌上调用 CloseHandle。</p> | `true` |
| `hostingModel` | <p>可选的字符串属性。</p><p>将托管模型指定为进程内 (`InProcess`/`inprocess`) 或进程外 (`OutOfProcess`/`outofprocess`)。</p> | `OutOfProcess`<br>`outofprocess` |
| `processesPerApplication` | <p>可选的整数属性。</p><p>指定每个应用均可启动的 `processPath` 设置中指定的进程的实例数。</p><p>&dagger;对于进程内托管，值限制为 `1`。</p><p>不建议设置 `processesPerApplication`。 将来的版本将删除此属性。</p> | 默认值：`1`<br>最小值：`1`<br>最大值：`100`&dagger; |
| `processPath` | <p>必需的字符串属性。</p><p>为 HTTP 请求启动进程侦听的可执行文件的路径。 支持相对路径。 如果路径以 `.` 开头，则该路径被视为与站点根目录相对。</p> | |
| `rapidFailsPerMinute` | <p>可选的整数属性。</p><p>指定允许 `processPath` 中指定的进程每分钟崩溃的次数。 如果超出了此限制，模块将在剩余分钟数内停止启动该进程。</p><p>不支持进程内托管。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`100` |
| `requestTimeout` | <p>可选的 timespan 属性。</p><p>指定 ASP.NET Core 模块等待来自 %ASPNETCORE_PORT% 上侦听的进程的响应的持续时间。</p><p>在 ASP.NET Core 2.1 或更高版本附带的 ASP.NET Core 模块版本中，使用小时数、分钟数和秒数指定 `requestTimeout`。</p><p>不适用于进程内托管。 对于进程内托管，该模块等待应用处理该请求。</p><p>此字符串的分钟段和秒钟段的有效值在 0-59 之间。 在分钟或秒钟值中使用“60”会导致“500 - 内部服务器错误”。</p> | 默认值：`00:02:00`<br>最小值：`00:00:00`<br>最大值：`360:00:00` |
| `shutdownTimeLimit` | <p>可选的整数属性。</p><p>检测到 `app_offline.htm` 文件时，模块等待可执行文件正常关闭的持续时间（以秒为单位）。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`600` |
| `startupTimeLimit` | <p>可选的整数属性。</p><p>模块等待可执行文件启动端口上侦听的进程的持续时间（以秒为单位）。 如果超出了此时间限制，模块将终止该进程。</p><p>进程内托管时：不会重新启动该进程，也不会使用 `rapidFailsPerMinute` 设置 。</p><p>进程外托管时：模块在收到新请求时尝试重新启动该进程，并在收到后续传入请求时继续尝试重新启动该进程，除非应用在上一回滚分钟内无法启动 `rapidFailsPerMinute` 次。</p><p>值 0（零）不被视为无限超时。</p> | 默认值：`120`<br>最小值：`0`<br>最大值：`3600` |
| `stdoutLogEnabled` | <p>可选布尔属性。</p><p>如果为 true，`processPath` 中指定的进程的 stdout 和 stderr 将重定向到 stdoutLogFile 中指定的文件  。</p> | `false` |
| `stdoutLogFile` | <p>可选的字符串属性。</p><p>指定在其中记录 `processPath` 中指定进程的 `stdout` 和 `stderr` 的相对路径或绝对路径。 相对路径与站点根目录相对。 以 `.` 开头的任何路径均与站点根目录相对，所有其他路径被视为绝对路径。 创建日志文件时，模块会创建路径中提供的所有文件夹。 使用下划线分隔符，将时间戳、进程 ID 和文件扩展名 (`.log`) 添加到 `stdoutLogFile` 路径的最后一段。 如果 `.\logs\stdout` 作为值提供，则在示例 stdout 日志使用进程 ID 1934 于 2018 年 2 月 5 日 19:41:32 保存时，将在 `logs` 文件夹中保存为 `stdout_20180205194132_1934.log`。</p> | `aspnetcore-stdout` |

### <a name="setting-environment-variables"></a>设置环境变量

可以为 `processPath` 属性中的进程指定环境变量。 使用 `<environmentVariables>` 集合元素的 `<environmentVariable>` 子元素指定环境变量。 本部分中设置的环境变量优先于系统环境变量。

以下示例设置了两个环境变量。 `ASPNETCORE_ENVIRONMENT` 将应用的环境配置为 `Development`。 开发人员可能会暂时在 `web.config` 文件中设置此值，以便在调试应用异常时强制加载[开发人员异常页面](xref:fundamentals/error-handling)。 `CONFIG_DIR` 是用户定义的环境变量的一个示例，其中开发人员已写入可在启动时读取值的代码以便形成用于加载应用配置文件的路径。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile=".\logs\stdout"
      hostingModel="inprocess">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!NOTE]
> 直接在 `web.config` 中设置环境的替代方法是将 `<EnvironmentName>` 属性包含在[发布配置文件 (.pubxml)](xref:host-and-deploy/visual-studio-publish-profiles) 或项目文件中。 此方法在发布项目时设置 `web.config` 中的环境：
>
> ```xml
> <PropertyGroup>
>   <EnvironmentName>Development</EnvironmentName>
> </PropertyGroup>
> ```

> [!WARNING]
> 仅当不受信任的网络（如 Internet）无法访问暂存和测试服务器时，才能将 `ASPNETCORE_ENVIRONMENT` 环境变量设置为 `Development`。

## <a name="app_offlinehtm"></a>app_offline.htm

如果在应用的根目录中检测到名为“`app_offline.htm`”的文件，ASP.NET Core 模块将尝试正常关闭应用并停止处理传入请求。 如果应用在 `shutdownTimeLimit` 中定义的秒数之后仍在运行，ASP.NET Core 模块将终止正在运行的进程。

存在 `app_offline.htm` 文件时，ASP.NET Core 模块会通过发送回 `app_offline.htm` 文件的内容来响应请求。 删除 `app_offline.htm` 文件后，下一个请求将启动应用。

使用进程外托管模型时，如果有已打开的连接，则应用可能不会立即关闭。 例如，WebSocket 连接可能会延迟应用关闭。

## <a name="start-up-error-page"></a>启动错误页面

进程内和进程外托管在无法启动应用时均会生成自定义错误页面。

如果 ASP.NET Core 模块无法找到进程内或进程外请求处理程序，则会显示“500.0 - 进程内/进程外处理程序加载失败”状态代码页。

对于进程内托管，如果 ASP.NET Core 模块无法启动应用，则会显示“500.30 - 启动失败”状态代码页。

对于进程外托管，如果 ASP.NET Core 模块无法启动后端进程或后端进程启动但无法在配置的端口上侦听，则将显示“502.5 - 进程失败”状态代码页。

若要禁止显示此页面并还原为默认 IIS 5xx 状态代码页，请使用 `disableStartUpErrorPage` 属性。 有关配置自定义错误消息的详细信息，请参阅 [HTTP 错误 \<httpErrors>](/iis/configuration/system.webServer/httpErrors/)。

## <a name="log-creation-and-redirection"></a>日志创建和重定向

如果已设置 `aspNetCore` 元素的 `stdoutLogEnabled` 和 `stdoutLogFile` 属性，ASP.NET Core 模块会将 stdout 和 stderr 控制台输出重定向到磁盘。 创建日志文件时，模块会创建 `stdoutLogFile` 路径中提供的所有文件夹。 应用池必须对写入日志的位置具有写入权限（使用 `IIS AppPool\{APP POOL NAME}` 提供写入权限，其中占位符 `{APP POOL NAME}` 为应用池名称）。

日志不会旋转，除非回收/重新启动进程。 宿主负责限制日志占用的磁盘空间。

仅当在 IIS 上托管或使用 [Visual Studio 中针对 IIS 的开发时支持](xref:host-and-deploy/iis/development-time-iis-support)时，而不是在本地调试和使用 IIS Express 运行应用时，才建议使用 stdout 日志来排查应用程序启动问题。

不要将 stdout 日志用于常规应用日志记录。 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

创建日志文件时，将自动添加时间戳和文件扩展名。 日志文件名是通过将时间戳、进程 ID 和文件扩展名 (`.log`) 附加到由下划线分隔的 `stdoutLogFile` 路径的最后一段（通常为 `stdout`）组合而成的。 如果 `stdoutLogFile` 路径以 `stdout` 结尾，则 PID 为 1934 且于 2018 年 2 月 5 日 19:42:32 创建的应用的日志的文件名为 `stdout_20180205194132_1934.log`。

如果 `stdoutLogEnabled` 为 false，则会捕获应用启动时发生的错误，并将其发送到事件日志，最大为 30 KB。 启动后，将丢弃所有其他日志。

以下示例 `aspNetCore` 元素在相对路径 `.\log\` 上配置 stdout 日志记录。 确认应用池用户标识是否已获得写入路径的权限。

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout"
    hostingModel="inprocess">
</aspNetCore>
```

发布用于 Azure 应用服务部署的应用时，Web SDK 会将 `stdoutLogFile` 值设置为 `\\?\%home%\LogFiles\stdout`。 已为 Azure 应用服务托管的应用预定义了 `%home` 环境变量。

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

提供给 `<handlerSetting>` 值（上述示例中为 `logs`）的路径中的文件夹不是由模块自动创建的，并且应该预先存在于部署中。 应用池必须对写入日志的位置具有写入权限（使用 `IIS AppPool\{APP POOL NAME}` 提供写入权限，其中占位符 `{APP POOL NAME}` 为应用池名称）。

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

有关 `web.config` 文件中的 `aspNetCore` 元素的示例，请参阅 [web.config 的配置](#configuration-with-webconfig)。

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
1. 将已更新的 `applicationHost.config` 文件导出到共享。
1. 重新启用 IIS 共享配置。

## <a name="module-version-and-hosting-bundle-installer-logs"></a>模块版本和托管捆绑安装程序日志

若要确定已安装 ASP.NET Core 模块的版本，请执行以下操作：

1. 在托管系统上，导航到 `%windir%\System32\inetsrv`。
1. 找到 `aspnetcore.dll` 文件。
1. 右键单击该文件，然后从上下文菜单中选择“属性”。
1. 选择“详细信息”选项卡。“文件版本”和“产品版本”表示模块的已安装版本。

可在 `C:\\Users\\%UserName%\\AppData\\Local\\Temp` 中找到模块的托管捆绑安装程序日志。 该文件的名称为 `dd_DotNetCoreWinSvrHosting__\{TIMESTAMP}_000_AspNetCoreModule_x64.log`，其中占位符 `{TIMESTAMP}` 为时间戳。

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

::: moniker-end

::: moniker range="< aspnetcore-2.2"

ASP.NET Core 模块是插入 IIS 管道的本机 IIS 模块，用于将 Web 请求转发到后端 ASP.NET Core 应用。

受支持的 Windows 版本：

* Windows 7 或更高版本
* Windows Server 2008 R2 或更高版本

该模块仅适用于 Kestrel。 该模块与 [HTTP.sys](xref:fundamentals/servers/httpsys) 不兼容。

由于 ASP.NET Core 应用在独立于 IIS 辅助进程的进程中运行，因此该模块还处理进程管理。 该模块在第一个请求到达时启动 ASP.NET Core 应用的进程，并在应用崩溃时重新启动该应用。 这基本上与在 [Windows 进程激活服务 (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was) 托管的 IIS 中在进程内运行的 ASP.NET 4.x 应用中出现的行为相同。

下图说明了 IIS、ASP.NET Core 模块和应用之间的关系：

![ASP.NET Core 模块](iis/index/_static/ancm-outofprocess.png)

请求从 Web 到达内核模式 HTTP.sys 驱动程序。 驱动程序将请求路由到网站的配置端口上的 IIS，通常为 80 (HTTP) 或 443 (HTTPS)。 该模块将该请求转发到应用的随机端口（非端口 80/443）上的 Kestrel。

该模块在启动时通过环境变量指定端口，[IIS 集成中间件](xref:host-and-deploy/iis/index#enable-the-iisintegration-components)将服务器配置为侦听 `http://localhost:{port}`。 执行其他检查，拒绝不是来自该模块的请求。 该模块不支持 HTTPS 转发，因此即使请求由 IIS 通过 HTTPS 接收，它们还是通过 HTTP 转发。

Kestrel 从模块获取请求后，请求会被推送到 ASP.NET Core 中间件管道中。 中间件管道处理该请求并将其作为 `HttpContext` 实例传递给应用的逻辑。 IIS 集成添加的中间件会将方案、远程 IP 和 pathbase 更新到帐户以将请求转发到 Kestrel。 应用的响应传递回 IIS，IIS 将响应推送回发起请求的 HTTP 客户端。

许多本机模块（如 Windows 身份验证）仍处于活动状态。 要详细了解随 ASP.NET Core 模块活动的 IIS 模块，请参阅 <xref:host-and-deploy/iis/modules>。

ASP.NET Core 模块还可以：

* 为工作进程设置环境变量。
* 将 stdout 输出记录到文件存储器，以解决启动问题。
* 转发 Windows 身份验证令牌。

## <a name="how-to-install-and-use-the-aspnet-core-module"></a>如何安装和使用 ASP.NET Core 模块

有关如何安装 ASP.NET Core 模块的说明，请参阅[安装 .NET Core 托管捆绑包](xref:host-and-deploy/iis/index#install-the-net-core-hosting-bundle)。

## <a name="configuration-with-webconfig"></a>web.config 的配置

在站点的 *web.config* 文件中使用 `system.webServer` 节点的 `aspNetCore` 部分配置 ASP.NET Core 模块。

以下 *web.config* 文件发布用于[依赖框架的部署](/dotnet/articles/core/deploying/#framework-dependent-deployments-fdd)，并配置 ASP.NET Core 模块以处理站点请求：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath="dotnet"
                arguments=".\MyApp.dll"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

以下 web.config 发布用于[独立部署](/dotnet/articles/core/deploying/#self-contained-deployments-scd)：

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <system.webServer>
    <handlers>
      <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModule" resourceType="Unspecified" />
    </handlers>
    <aspNetCore processPath=".\MyApp.exe"
                stdoutLogEnabled="false"
                stdoutLogFile=".\logs\stdout" />
  </system.webServer>
</configuration>
```

将应用部署为 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)时，`stdoutLogFile` 路径将设置为 `\\?\%home%\LogFiles\stdout`。 该路径会将 stdout 日志保存到 *LogFiles* 文件夹，该文件夹是由服务自动创建的位置。

有关 IIS 子应用程序配置的信息，请参阅<xref:host-and-deploy/iis/index#sub-applications>。

### <a name="attributes-of-the-aspnetcore-element"></a>aspNetCore 元素的属性

| 特性 | 描述 | 默认 |
| --------- | ----------- | :-----: |
| `arguments` | <p>可选的字符串属性。</p><p>processPath 中指定的可执行文件的参数。</p>| |
| `disableStartUpErrorPage` | <p>可选布尔属性。</p><p>如果为 true，将禁止显示“502.5 - 进程失败”页面，而会优先显示 web.config 中配置的 502 状态代码页面。</p> | `false` |
| `forwardWindowsAuthToken` | <p>可选布尔属性。</p><p>如果为 true，会将令牌作为每个请求的标头“MS-ASPNETCORE-WINAUTHTOKEN”，转发到在 %ASPNETCORE_PORT% 上侦听的子进程。 该进程负责在每个请求的此令牌上调用 CloseHandle。</p> | `true` |
| `processesPerApplication` | <p>可选的整数属性。</p><p>指定每个应用均可启动的 **processPath** 设置中指定的进程的实例数。</p><p>不建议设置 `processesPerApplication`。 将来的版本将删除此属性。</p> | 默认值：`1`<br>最小值：`1`<br>最大值：`100` |
| `processPath` | <p>必需的字符串属性。</p><p>为 HTTP 请求启动进程侦听的可执行文件的路径。 支持相对路径。 如果路径以 `.` 开头，则该路径被视为与站点根目录相对。</p> | |
| `rapidFailsPerMinute` | <p>可选的整数属性。</p><p>指定允许 processPath 中指定的进程每分钟崩溃的次数。 如果超出了此限制，模块将在剩余分钟数内停止启动该进程。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`100` |
| `requestTimeout` | <p>可选的 timespan 属性。</p><p>指定 ASP.NET Core 模块等待来自 %ASPNETCORE_PORT% 上侦听的进程的响应的持续时间。</p><p>在 ASP.NET Core 2.1 或更高版本附带的 ASP.NET Core 模块版本中，使用小时数、分钟数和秒数指定 `requestTimeout`。</p> | 默认值：`00:02:00`<br>最小值：`00:00:00`<br>最大值：`360:00:00` |
| `shutdownTimeLimit` | <p>可选的整数属性。</p><p>检测到 app_offline.htm 文件时，模块等待可执行文件正常关闭的持续时间（以秒为单位）。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`600` |
| `startupTimeLimit` | <p>可选的整数属性。</p><p>模块等待可执行文件启动端口上侦听的进程的持续时间（以秒为单位）。 如果超出了此时间限制，模块将终止该进程。 模块在收到新请求时尝试重新启动该进程，并在收到后续传入请求时继续尝试重新启动该进程，除非应用在上一回滚分钟内无法启动 rapidFailsPerMinute 次。</p><p>值 0（零）不被视为无限超时。</p> | 默认值：`120`<br>最小值：`0`<br>最大值：`3600` |
| `stdoutLogEnabled` | <p>可选布尔属性。</p><p>如果为 true，processPath 中指定的 进程的 stdout 和 stderr 将重定向到 stdoutLogFile 中指定的文件。</p> | `false` |
| `stdoutLogFile` | <p>可选的字符串属性。</p><p>指定在其中记录 processPath 中指定进程的 stdout 和 stderr 的相对路径或绝对路径。 相对路径与站点根目录相对。 以 `.` 开头的任何路径均与站点根目录相对，所有其他路径被视为绝对路径。 路径中提供的任何文件夹都必须存在，以便模块创建日志文件。 使用下划线分隔符，将时间戳、进程 ID 和文件扩展名 (.log) 添加到 stdoutLogFile 路径的最后一段。 如果 `.\logs\stdout` 作为值提供，则在示例 stdout 日志使用进程 ID 1934 于 2018 年 2 月 5 日 19:41:32 保存时，将在 logs 文件夹中保存为 stdout_20180205194132_1934.log。</p> | `aspnetcore-stdout` |

### <a name="setting-environment-variables"></a>设置环境变量

可以为 `processPath` 属性中的进程指定环境变量。 使用 `<environmentVariables>` 集合元素的 `<environmentVariable>` 子元素指定环境变量。

> [!WARNING]
> 本部分中设置的环境变量与使用同一名称设置的系统环境变量冲突。 如果在 web.config 文件以及 Windows 中的系统级别中同时设置了环境变量，则 web.config 文件中的值将被追加到系统环境变量值（例如，`ASPNETCORE_ENVIRONMENT: Development;Development`），这将阻止应用启动。

以下示例设置了两个环境变量。 `ASPNETCORE_ENVIRONMENT` 将应用的环境配置为 `Development`。 开发人员可能会暂时在 web.config 文件中设置此值，以便在调试应用异常时强制加载[开发人员异常页面](xref:fundamentals/error-handling)。 `CONFIG_DIR` 是用户定义的环境变量的一个示例，其中开发人员已写入可在启动时读取值的代码以便形成用于加载应用配置文件的路径。

```xml
<aspNetCore processPath="dotnet"
      arguments=".\MyApp.dll"
      stdoutLogEnabled="false"
      stdoutLogFile="\\?\%home%\LogFiles\stdout">
  <environmentVariables>
    <environmentVariable name="ASPNETCORE_ENVIRONMENT" value="Development" />
    <environmentVariable name="CONFIG_DIR" value="f:\application_config" />
  </environmentVariables>
</aspNetCore>
```

> [!WARNING]
> 仅当不受信任的网络（如 Internet）无法访问暂存和测试服务器时，才能将 `ASPNETCORE_ENVIRONMENT` 环境变量设置为 `Development`。

## <a name="app_offlinehtm"></a>app_offline.htm

如果在应用的根目录中检测到名为 “app_offline.htm”的文件，ASP.NET Core 模块将尝试正常关闭应用并停止处理传入请求。 如果应用在 `shutdownTimeLimit` 中定义的秒数之后仍在运行，ASP.NET Core 模块将终止正在运行的进程。

存在 app_offline.htm 文件时，ASP.NET Core 模块会通过发送回 app_offline.htm 文件的内容来响应请求。 删除 app_offline.htm 文件后，下一个请求将启动应用。

## <a name="start-up-error-page"></a>启动错误页面

如果 ASP.NET Core 模块无法启动后端进程或后端进程启动但无法在配置的端口上侦听，则将显示“502.5 - 进程失败”状态代码页。 若要禁止显示此页面并还原为默认 IIS 502 状态代码页面，请使用 `disableStartUpErrorPage` 属性。 有关配置自定义错误消息的详细信息，请参阅 [HTTP 错误 \<httpErrors>](/iis/configuration/system.webServer/httpErrors/)。

## <a name="log-creation-and-redirection"></a>日志创建和重定向

如果已设置 `aspNetCore` 元素的 `stdoutLogEnabled` 和 `stdoutLogFile` 属性，ASP.NET Core 模块会将 stdout 和 stderr 控制台输出重定向到磁盘。 创建日志文件时，模块会创建 `stdoutLogFile` 路径中提供的所有文件夹。 应用池必须对写入日志的位置具有写入权限（使用 `IIS AppPool\<app_pool_name>` 提供写入权限）。

日志不会旋转，除非回收/重新启动进程。 宿主负责限制日志占用的磁盘空间。

仅当在 IIS 上托管或使用 [Visual Studio 中针对 IIS 的开发时支持](xref:host-and-deploy/iis/development-time-iis-support)时，而不是在本地调试和使用 IIS Express 运行应用时，才建议使用 stdout 日志来排查应用程序启动问题。

不要将 stdout 日志用于常规应用日志记录。 对于 ASP.NET Core 应用中的例程日志记录，使用限制日志文件大小和旋转日志的日志记录库。 有关详细信息，请参阅[第三方日志记录提供程序](xref:fundamentals/logging/index#third-party-logging-providers)。

创建日志文件时，将自动添加时间戳和文件扩展名。 日志文件名是通过将时间戳、进程 ID 和文件扩展名 (.log) 附加到由下划线分隔的 `stdoutLogFile` 路径的最后一段（通常为 stdout）组合而成的。 如果 `stdoutLogFile` 路径以 stdout 结尾，则 PID 为 1934 且于 2018 年 2 月 5 日 19:42:32 创建的应用的日志的文件名为 stdout_20180205194132_1934.log。

以下示例 `aspNetCore` 元素在相对路径 `.\log\` 上配置 stdout 日志记录。 确认应用池用户标识是否已提供写入路径的权限。

```xml
<aspNetCore processPath="dotnet"
    arguments=".\MyApp.dll"
    stdoutLogEnabled="true"
    stdoutLogFile=".\logs\stdout">
</aspNetCore>
```

发布用于 Azure 应用服务部署的应用时，Web SDK 会将 `stdoutLogFile` 值设置为 `\\?\%home%\LogFiles\stdout`。 已为 Azure 应用服务托管的应用预定义了 `%home` 环境变量。

要创建日志记录筛选器规则，请参阅 ASP.NET Core 日志记录文档的[配置](xref:fundamentals/logging/index#log-filtering)和[日志筛选](xref:fundamentals/logging/index#log-filtering)部分。

有关路径格式的详细信息，请参阅 [Windows 系统上的文件路径格式](/dotnet/standard/io/file-path-formats)。

## <a name="proxy-configuration-uses-http-protocol-and-a-pairing-token"></a>代理配置使用 HTTP 协议和配对令牌

在 ASP.NET Core 模块和 Kestrel 之间创建的代理使用 HTTP 协议。 因此，不存在从脱离服务器的位置窃取模块和 Kestrel 之间的流量的风险。

配对令牌用于保证 Kestrel 收到的请求已由 IIS 代理且不来自某些其他源。 模块已创建配对令牌并将其设置到环境变量 (`ASPNETCORE_TOKEN`)。 此外，配对令牌还设置到每个代理请求的标头 (`MS-ASPNETCORE-TOKEN`)。 IIS 中间件检查它所接收的每个请求，以确认配对令牌标头值与环境变量值相匹配。 如果令牌值不匹配，则将记录请求并拒绝该请求。 无法从脱离服务器的位置访问配对令牌环境变量及模块和 Kestrel 之间的流量。 如果不知道配对令牌值，攻击者就无法提交绕过 IIS 中间件中的检查的请求。

## <a name="aspnet-core-module-with-an-iis-shared-configuration"></a>具有 IIS 共享配置的 ASP.NET Core 模块

ASP.NET Core 模块安装程序使用 TrustedInstaller 帐户的权限运行。 由于本地系统帐户没有 IIS 共享配置所用的共享路径的修改权限，因此在尝试配置共享上的 applicationHost.config 文件中的模块设置时，安装程序将引发拒绝访问错误。

使用 IIS 共享配置时，请执行以下步骤：

1. 禁用 IIS 共享配置。
1. 运行安装程序。
1. 将已更新的 applicationHost.config 文件导出到共享。
1. 重新启用 IIS 共享配置。

## <a name="module-version-and-hosting-bundle-installer-logs"></a>模块版本和托管捆绑安装程序日志

若要确定已安装 ASP.NET Core 模块的版本，请执行以下操作：

1. 在托管系统上，导航到 %windir%\System32\inetsrv。
1. 找到 aspnetcore.dll 文件。
1. 右键单击该文件，然后从上下文菜单中选择“属性”。
1. 选择“详细信息”选项卡。“文件版本”和“产品版本”表示模块的已安装版本。

在 C:\\Users\\%UserName%\\AppData\\Local\\Temp 中找到了模块的托管捆绑安装程序日志。该文件名为 dd_DotNetCoreWinSvrHosting__\<timestamp>_000_AspNetCoreModule_x64.log。

## <a name="module-schema-and-configuration-file-locations"></a>模块、架构和配置文件位置

### <a name="module"></a>模块

**IIS (x86/amd64)** ：

* %windir%\System32\inetsrv\aspnetcore.dll

* %windir%\SysWOW64\inetsrv\aspnetcore.dll

**IIS Express (x86/amd64)** ：

* %ProgramFiles%\IIS Express\aspnetcore.dll

* %ProgramFiles(x86)%\IIS Express\aspnetcore.dll

### <a name="schema"></a>架构

**IIS**

* %windir%\System32\inetsrv\config\schema\aspnetcore_schema.xml

**IIS Express**

* %ProgramFiles%\IIS Express\config\schema\aspnetcore_schema.xml

### <a name="configuration"></a>Configuration

**IIS**

* %windir%\System32\inetsrv\config\applicationHost.config

**IIS Express**

* Visual Studio：{APPLICATION ROOT}\\.vs\config\applicationHost.config

* iisexpress.exe CLI：%USERPROFILE%\Documents\IISExpress\config\applicationhost.config

可以通过在 applicationHost.config 文件中搜索 aspnetcore 找到这些文件。

::: moniker-end

## <a name="additional-resources"></a>其他资源

* <xref:host-and-deploy/iis/index>
* <xref:host-and-deploy/azure-apps/index>
* [ASP.NET Core 模块引用源（主分支）](https://github.com/dotnet/aspnetcore/tree/master/src/Servers/IIS/AspNetCoreModuleV2)：使用“分支”下拉列表选择特定版本（例如，`release/3.1`）。
* <xref:host-and-deploy/iis/modules>
