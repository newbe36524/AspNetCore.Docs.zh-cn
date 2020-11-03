---
title: web.config 文件
author: rick-anderson
description: 了解 web.config 文件中的内容，以及如何配置不同的 ASP.NET Core 模块选项。
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
uid: host-and-deploy/iis/web-config
ms.openlocfilehash: 4d7305f7184745b66c5de6c86b907d419183cb3f
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91755150"
---
# <a name="webconfig-file"></a>`web.config` 文件

`web.config` 文件由 IIS 和 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)读取，用于使用 IIS 配置已托管的应用。

## <a name="webconfig-file-location"></a>`web.config` 文件位置

为了正确设置 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)，`web.config` 文件必须存在于已部署应用的[内容根](xref:fundamentals/index#content-root)路径（通常为应用基路径）中。 该位置与向 IIS 提供的网站物理路径相同。 若要使用 Web 部署发布多个应用，应用的根路径中需要包含 `web.config` 文件。

敏感文件存在于应用的物理路径中，如 `{ASSEMBLY}.runtimeconfig.json`、`{ASSEMBLY}.xml`（XML 文档注释）和 `{ASSEMBLY}.deps.json`，其中 `{ASSEMBLY}` 占位符为程序集名称。 如果有 `web.config` 文件且站点正常启动时，IIS 在收到敏感文件请求时不会提供这些敏感文件。 如果 `web.config` 文件缺失、名字错误或者无法将站点配置为正常启动，IIS 可能会公开提供敏感文件。

`web.config` 文件必须始终存在于部署中、名称正确以及能够将站点配置为正常启动。切勿从生产部署中删除 `web.config` 文件。

如果项目中没有 `web.config` 文件，则该文件是使用正确的 `processPath` 和 `arguments`（用于配置 ASP.NET Core 模块）创建的，并且已被移到[发布的输出](xref:host-and-deploy/directory-structure)中。

如果项目中没有 `web.config` 文件，则该文件是通过正确的 `processPath` 和 `arguments`（用于配置 ASP.NET Core 模块）转换的，并且已被移到发布的输出中。 转换不会修改文件中的 IIS 配置设置。

`web.config` 文件可能会提供控制活动 IIS 模块的额外 IIS 配置设置。 有关能够处理 ASP.NET Core 应用请求的 IIS 模块的信息，请参阅 [IIS 模块](xref:host-and-deploy/iis/modules)主题。

发布项目时，`web.config` 文件的创建、转换和发布是由 MSBuild 目标 (`_TransformWebConfig`) 处理的。 此目标位于 Web SDK 目标 (`Microsoft.NET.Sdk.Web`) 中。 SDK 设置在项目文件的顶部：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

为了防止 Web SDK 转换 `web.config` 文件，请在项目文件中使用 `<IsTransformWebConfigDisabled>` 属性：

```xml
<PropertyGroup>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

禁用 Web SDK 对文件的转换时，`processPath` 和 `arguments` 应由开发人员手动设置。 有关详细信息，请参阅 <xref:host-and-deploy/aspnet-core-module>。

## <a name="configuration-of-aspnet-core-module-with-webconfig"></a>使用 `web.config` 配置 ASP.NET Core 模块

在站点的 `web.config` 文件中使用 `system.webServer` 节点的 `aspNetCore` 部分配置 ASP.NET Core 模块。

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

以下 `web.config` 发布用于[独立部署](/dotnet/articles/core/deploying/#self-contained-deployments-scd)：

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

将 <xref:System.Configuration.SectionInformation.InheritInChildApplications%2A> 属性设置为 `false`，表示 [`<location>`](/iis/manage/managing-your-configuration-settings/understanding-iis-configuration-delegation#the-concept-of-location) 元素中指定的设置不会由驻留在应用子目录中的应用继承。

将应用部署为 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)时，`stdoutLogFile` 路径将设置为 `\\?\%home%\LogFiles\stdout`。 该路径会将 stdout 日志保存到 `LogFiles` 文件夹，该文件夹是由服务自动创建的位置。

有关 IIS 子应用程序配置的信息，请参阅<xref:host-and-deploy/iis/index#sub-applications>。

### <a name="attributes-of-the-aspnetcore-element"></a>`aspNetCore` 元素的属性

| 属性 | 描述 | 默认 |
| --------- | ----------- | :-----: |
| `arguments` | <p>可选的字符串属性。</p><p>`processPath` 中指定的可执行文件的参数。</p> | |
| `disableStartUpErrorPage` | <p>可选布尔属性。</p><p>如果为 true，将禁止显示“502.5 - 进程失败”页面，而会优先显示 `web.config` 中配置的 502 状态代码页面。</p> | `false` |
| `forwardWindowsAuthToken` | <p>可选布尔属性。</p><p>如果为 true，会将令牌作为每个请求的标头“MS-ASPNETCORE-WINAUTHTOKEN”转发到在 `%ASPNETCORE_PORT%` 上侦听的子进程。 该进程负责在每个请求的此令牌上调用 CloseHandle。</p> | `true` |
| `hostingModel` | <p>可选的字符串属性。</p><p>将托管模型指定为进程内 (`InProcess`/`inprocess`) 或进程外 (`OutOfProcess`/`outofprocess`)。</p> | `InProcess`<br>`inprocess` |
| `processesPerApplication` | <p>可选的整数属性。</p><p>指定每个应用均可启动的 `processPath` 设置中指定的进程的实例数。</p><p>&dagger;对于进程内托管，值限制为 `1`。</p><p>不建议设置 `processesPerApplication`。 将来的版本将删除此属性。</p> | 默认值：`1`<br>最小值：`1`<br>最大值：`100`&dagger; |
| `processPath` | <p>必需的字符串属性。</p><p>为 HTTP 请求启动进程侦听的可执行文件的路径。 支持相对路径。 如果路径以 `.` 开头，则该路径被视为与站点根目录相对。</p> | |
| `rapidFailsPerMinute` | <p>可选的整数属性。</p><p>指定允许 `processPath` 中指定的进程每分钟崩溃的次数。 如果超出了此限制，模块将在剩余分钟数内停止启动该进程。</p><p>不支持进程内托管。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`100` |
| `requestTimeout` | <p>可选的 timespan 属性。</p><p>指定 ASP.NET Core 模块等待来自 %ASPNETCORE_PORT% 上侦听的进程的响应的持续时间。</p><p>在 ASP.NET Core 2.1 或更高版本附带的 ASP.NET Core 模块版本中，使用小时数、分钟数和秒数指定 `requestTimeout`。</p><p>不适用于进程内托管。 对于进程内托管，该模块等待应用处理该请求。</p><p>此字符串的分钟段和秒钟段的有效值在 0-59 之间。 在分钟或秒钟值中使用“`60`”会导致“500 - 内部服务器错误”。</p> | 默认值：`00:02:00`<br>最小值：`00:00:00`<br>最大值：`360:00:00` |
| `shutdownTimeLimit` | <p>可选的整数属性。</p><p>检测到 `app_offline.htm` 文件时，模块等待可执行文件正常关闭的持续时间（以秒为单位）。</p> | 默认值：`10`<br>最小值：`0`<br>最大值：`600` |
| `startupTimeLimit` | <p>可选的整数属性。</p><p>模块等待可执行文件启动端口上侦听的进程的持续时间（以秒为单位）。 如果超出了此时间限制，模块将终止该进程。</p><p>进程内托管时：不会重新启动该进程，也不会使用 `rapidFailsPerMinute` 设置 。</p><p>进程外托管时：模块在收到新请求时尝试重新启动该进程，并在收到后续传入请求时继续尝试重新启动该进程，除非应用在上一回滚分钟内无法启动 `rapidFailsPerMinute` 次。</p><p>值 0（零）不被视为无限超时。</p> | 默认值：`120`<br>最小值：`0`<br>最大值：`3600` |
| `stdoutLogEnabled` | <p>可选布尔属性。</p><p>如果为 true，`processPath` 中指定的进程的 `stdout` 和 `stderr` 将重定向到 `stdoutLogFile` 中指定的文件。</p> | `false` |
| `stdoutLogFile` | <p>可选的字符串属性。</p><p>指定在其中记录 `processPath` 中指定进程的 `stdout` 和 `stderr` 的相对路径或绝对路径。 相对路径与站点根目录相对。 以 `.` 开头的任何路径均与站点根目录相对，所有其他路径被视为绝对路径。 创建日志文件时，模块会创建路径中提供的所有文件夹。 使用下划线分隔符，将时间戳、进程 ID 和文件扩展名 (`.log`) 添加到 `stdoutLogFile` 路径的最后一段。 如果 `.\logs\stdout` 作为值提供，则在示例 stdout 日志使用进程 ID 1934 于 2018 年 2 月 5 日 19:41:32 保存时，将在 `logs` 文件夹中保存为 `stdout_20180205194132_1934.log`。</p> | `aspnetcore-stdout` |

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

## <a name="configuration-of-iis-with-webconfig"></a>使用 `web.config` 配置 IIS

IIS 配置受用于 IIS 方案（适用于包含 ASP.NET Core 模块的 ASP.NET Core 应用）的 `web.config` 的 `<system.webServer>` 部分影响。 例如，IIS 配置适用于动态压缩。 如果在服务器一级将 IIS 配置为使用动态压缩，可通过应用的 `web.config` 文件中的 `<urlCompression>` 元素，对 ASP.NET Core 应用禁用它。

有关详细信息，请参阅下列主题：

* [`<system.webServer>` 的配置参考](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/aspnet-core-module>
* <xref:host-and-deploy/iis/modules>

要为在独立应用池中运行的各应用设置环境变量（IIS 10.0 或更高版本中支持此操作），请参阅 IIS 参考文档中[环境变量 `<environmentVariables>`](/iis/configuration/system.applicationHost/applicationPools/add/environmentVariables/#appcmdexe) 主题下的“`AppCmd.exe` 命令”部分。

## <a name="configuration-sections-of-webconfig"></a>`web.config` 的配置节

ASP.NET Core 应用不会使用 `web.config` 中的 ASP.NET 4.x 应用的配置部分进行配置：

* `<system.web>`
* `<appSettings>`
* `<connectionStrings>`
* `<location>`

会使用其他的配置提供程序配置 ASP.NET Core 应用。 有关详细信息，请参阅[配置](xref:fundamentals/configuration/index)。

## <a name="transform-webconfig"></a>转换 web.config

如果在发布时需要转换 `web.config`，请参阅 <xref:host-and-deploy/iis/transform-webconfig>。 你需要在发布时转换 `web.config`，以根据配置、配置文件或环境来设置环境变量。

## <a name="additional-resources"></a>其他资源

* [IIS \<system.webServer>](/iis/configuration/system.webServer/)
* <xref:host-and-deploy/iis/modules>
* <xref:host-and-deploy/iis/transform-webconfig>
