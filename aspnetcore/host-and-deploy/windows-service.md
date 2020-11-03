---
title: 在 Windows 服务中托管 ASP.NET Core
author: rick-anderson
description: 了解如何在 Windows 服务中托管 ASP.NET Core 应用。
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
uid: host-and-deploy/windows-service
ms.openlocfilehash: d4df10f9450ca956d7b1a4297caa63cdd0caf23e
ms.sourcegitcommit: ecae2aa432628b9181d1fa11037c231c7dd56c9e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92113746"
---
# <a name="host-aspnet-core-in-a-windows-service"></a>在 Windows 服务中托管 ASP.NET Core

::: moniker range=">= aspnetcore-3.0"

不使用 IIS 时，可以在 Windows 上将 ASP.NET Core 应用作为 [Windows 服务](/dotnet/framework/windows-services/introduction-to-windows-service-applications)进行托管。 作为 Windows 服务进行托管时，应用将在服务器重新启动后自动启动。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/windows-service/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="prerequisites"></a>先决条件

* [ASP.NET .NET Core SDK 2.1 或更高版本](https://dotnet.microsoft.com/download)
* [PowerShell 6.2 或更高版本](https://github.com/PowerShell/PowerShell)

## <a name="worker-service-template"></a>辅助角色服务模板

ASP.NET Core 辅助角色服务模板可作为编写长期服务应用的起点。 要使用该模板作为编写 Windows 服务应用的基础：

1. 从 .NET Core 模板创建辅助角色服务应用。
1. 按照[应用配置](#app-configuration)部分中的指导来更新辅助角色服务应用，以便它可以作为 Windows 服务运行。

[!INCLUDE[](~/includes/worker-template-instructions.md)]

## <a name="app-configuration"></a>应用配置

应用需要 [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.Extensions.Hosting.WindowsServices) 的包引用。

生成主机时会调用 `IHostBuilder.UseWindowsService`。 若应用作为 Windows 服务运行，方法为：

* 将主机生存期设置为 `WindowsServiceLifetime`。
* 将[内容根](xref:fundamentals/index#content-root)设置为 [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory)。 有关详细信息，请参阅[当前目录和内容根](#current-directory-and-content-root)部分。
* 为事件日志启用日志记录：
  * 应用名称用作默认源名称。
  * 对于基于 ASP.NET Core 模板且调用 `CreateDefaultBuilder` 生成主机的应用，默认日志级别为“警告”或更高级别。
  * 在 appsettings.json/appsettings.{Environment}.json 或其他配置提供程序中，使用 `Logging:EventLog:LogLevel:Default` 键覆盖默认日志级别。
  * 只有管理员可以创建新的事件源。 无法使用应用程序名称创建事件源时，应用程序源将记录一条警告，并禁用事件源。

在 Program.cs 的 `CreateHostBuilder` 中：

```csharp
Host.CreateDefaultBuilder(args)
    .UseWindowsService()
    ...
```

本主题附带了以下示例应用：

* 后台辅助角色服务示例：一个基于[辅助角色服务模板](#worker-service-template)的非 Web 应用示例，该模板使用[托管服务](xref:fundamentals/host/hosted-services)执行后台任务。
* Web 应用服务示例：一个作为 Windows 服务运行的 Razor Pages Web 应用示例，通过[托管服务](xref:fundamentals/host/hosted-services)执行后台任务。

有关 MVC 指南，请参阅 <xref:mvc/overview> 和 <xref:migration/22-to-30> 下的文章。

## <a name="deployment-type"></a>部署类型

有关部署方案的信息和建议，请参阅 [.NET Core 应用程序部署](/dotnet/core/deploying/)。

### <a name="sdk"></a>SDK

对于使用 Razor Pages 或 MVC 框架的基于 Web 应用的服务，请在项目文件中指定 Web SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

如果服务仅执行后台任务（例如 [托管服务](xref:fundamentals/host/hosted-services)），请在项目文件中指定辅助角色 SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>依赖框架的部署 (FDD)

依赖框架的部署 (FDD) 依赖目标系统上存在共享系统级版本的 .NET Core。 按照本文中的指南采用 FDD 方案时，SDK 会生成一个称为“依赖框架的可执行文件”的可执行文件 (.exe)。 

如果使用 [Web SDK](#sdk)，则 web.config 文件（通常在发布 ASP.NET Core 应用时生成）不是 Windows 服务应用的必要文件。 若要禁止创建 web.config 文件，请将 `<IsTransformWebConfigDisabled>` 属性集添加到 `true`。

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp3.0</TargetFramework>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>独立部署 (SCD)

独立部署 (SCD) 不依赖主机系统上存在的共享框架。 运行时和应用的依赖项将与应用一起部署。

将 Windows [运行时标识符 (RID)](/dotnet/core/rid-catalog) 添加到包含目标框架的 `<PropertyGroup>` 中：

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

要发布多个 RID：

* 通过以分号分隔的列表提供 RID。
* 使用属性名称 [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers)（复数）。

有关详细信息，请参阅 [.NET Core RID 目录](/dotnet/core/rid-catalog)。

## <a name="service-user-account"></a>服务用户帐户

在管理 PowerShell 6 命令行界面中运行 [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet，为服务创建用户帐户。

对于 Windows 10 2018 年 10 月更新（版本 1809/内部版本 10.0.17763）或更高版本：

```powershell
New-LocalUser -Name {SERVICE NAME}
```

对于 Windows 10 2018 年 10 月更新（版本 1809/内部版本 10.0.17763）之前的 Windows 操作系统：

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

在系统提示时，提供[强密码](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)。

除非将 `-AccountExpires` 参数提供给到期时间为 <xref:System.DateTime> 的 [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet，否则用户帐户不会到期。

有关详细信息，请参阅 [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) 和[服务用户帐户](/windows/desktop/services/service-user-accounts)。

使用 Active Directory 时管理用户的另一种方法是使用托管服务帐户。 有关详细信息，请参阅[组托管服务帐户概述](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。

## <a name="log-on-as-a-service-rights"></a>以服务身份登录权限

为服务用户帐户创建“以服务身份登录”权限：

1. 通过运行 secpool.msc，打开本地安全策略编辑器。
1. 展开“本地策略”节点，选择“用户权限分配”。 
1. 打开“以服务身份登录”策略。
1. 选择“添加用户或组”。
1. 使用下列方法之一提供对象名称（用户帐户）：
   1. 在对象名称字段键入用户帐户 (`{DOMAIN OR COMPUTER NAME\USER}`)，然后选择“确定”，以将此用户添加到策略。
   1. 选择“高级”。 选择“开始查找”。 从列表中选择该用户帐户。 选择“确定”。 再次选择“确定”，以将该用户添加到策略。
1. 选择“确定”或“应用”，以接受更改。 

## <a name="create-and-manage-the-windows-service"></a>创建和管理 Windows 服务

### <a name="create-a-service"></a>创建服务

使用 PowerShell 注册服务。 从 PowerShell 6 命令行界面，执行以下命令：

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = "{DOMAIN OR COMPUTER NAME\USER}", "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName "{EXE FILE PATH}" -Credential "{DOMAIN OR COMPUTER NAME\USER}" -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`：应用在主机上的文件夹的路径（如 `d:\myservice`）。 请勿在此路径中包含应用的可执行文件。 尾部反斜杠是非必需项。
* `{DOMAIN OR COMPUTER NAME\USER}`：服务用户帐户（如 `Contoso\ServiceUser`）。
* `{SERVICE NAME}`：服务名称（如 `MyService`）。
* `{EXE FILE PATH}`：应用的可执行文件路径（如 `d:\myservice\myservice.exe`）。 请将可执行文件的文件名和扩展名包括在内。
* `{DESCRIPTION}`：服务说明（如 `My sample service`）。
* `{DISPLAY NAME}`：服务显示名称（如 `My Service`）。

### <a name="start-a-service"></a>启动服务

使用以下 PowerShell 6 命令启动服务：

```powershell
Start-Service -Name {SERVICE NAME}
```

此命令需要几秒钟才能启动服务。

### <a name="determine-a-services-status"></a>确信服务状态

要检查服务的状态，请使用以下 PowerShell 6 命令：

```powershell
Get-Service -Name {SERVICE NAME}
```

状态报告为以下值之一：

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>停止服务

使用以下 Powershell 6 命令停止服务：

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>删除服务

在停止服务一小段时间后，使用以下 Powershell 6 命令删除服务：

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

与来自 Internet 或公司网络的请求进行交互且在代理或负载均衡器后方的服务可能需要其他配置。 有关详细信息，请参阅 <xref:host-and-deploy/proxy-load-balancer>。

## <a name="configure-endpoints"></a>配置终结点

默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 通过设置 `ASPNETCORE_URLS` 环境变量来配置 URL 和端口。

有关其他 URL 和端口配置方法，请参阅相关服务器文章：

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

上述指南介绍了对 HTTPS 终结点的支持。 例如，在使用 Windows 服务进行身份验证时，为 HTTPS 配置应用。

> [!NOTE]
> 不支持使用 ASP.NET Core HTTPS 开发证书保护服务终结点。

## <a name="current-directory-and-content-root"></a>当前目录和内容根

通过为 Windows 服务调用 <xref:System.IO.Directory.GetCurrentDirectory*> 返回的当前工作目录是 C:\\WINDOWS\\system32 文件夹。 system32 文件夹不是存储服务文件（如设置文件）的合适位置。 使用以下方法之一来维护和访问服务的资产和设置文件。

### <a name="use-contentrootpath-or-contentrootfileprovider"></a>使用 ContentRootPath 或 ContentRootFileProvider

使用 [IHostEnvironment.ContentRootPath](xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath) 或 <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootFileProvider> 查找应用的资源。

应用作为服务运行时，<xref:Microsoft.Extensions.Hosting.WindowsServiceLifetimeHostBuilderExtensions.UseWindowsService*> 将 <xref:Microsoft.Extensions.Hosting.IHostEnvironment.ContentRootPath> 设置为 [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory)。

通过调用 [CreateDefaultBuilder during host construction](xref:fundamentals/host/generic-host#set-up-a-host)，从应用的内容根加载应用的默认设置文件 appsettings.json 和 appsettings.{Environment}.json。 

对于 <xref:Microsoft.Extensions.Hosting.HostBuilder.ConfigureAppConfiguration*> 中的开发人员代码加载的其他设置文件，无需调用 <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*>。 在下面的示例中，custom_settings.json 文件位于应用的内容根，加载它时未显式设置基本路径：

[!code-csharp[](windows-service/samples_snapshot/CustomSettingsExample.cs?highlight=13)]

请勿尝试使用 <xref:System.IO.Directory.GetCurrentDirectory*> 来获取资源路径，因为 Windows 服务应用会返回“C:\\WINDOWS\\system32”文件夹作为其当前目录。

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>将服务的文件存储在磁盘中的合适位置

使用 <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> 时，使用 <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> 指定到包含文件的文件夹的绝对路径。

## <a name="troubleshoot"></a>疑难解答

若要排除 Windows 服务应用的故障，请参阅 <xref:test/troubleshoot>。

### <a name="common-errors"></a>常见错误

* 正在使用 PowerShell 的早期版本或预发布版本。
* 注册的服务未使用来自 [dotnet publish](/dotnet/core/tools/dotnet-publish) 命令的应用的已发布输出。 应用部署不支持 [dotnet build](/dotnet/core/tools/dotnet-build) 命令的输出。 已发布的资产位于以下文件夹之一中，具体取决于部署类型：
  * *bin/Release/{TARGET FRAMEWORK}/publish* (FDD)
  * *bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)
* 服务未处于“正在运行”状态。
* 应用使用的资源（例如证书）的路径不正确。 Windows 服务的基本路径是“c:\\Windows\\System32”。
* 用户没有“以服务身份登录”权限。
* 在执行 `New-Service` PowerShell 命令时，用户密码已过期，或以不正确的方式传递。
* 应用需要进行 ASP.NET Core 身份验证，但是未配置安全连接 (HTTPS)。
* 请求 URL 端口不正确或未在应用中正确配置。

### <a name="system-and-application-event-logs"></a>系统和应用程序事件日志

访问系统和应用程序事件日志：

1. 打开“开始”菜单，搜索“事件查看器”，然后选择“事件查看器”应用。
1. 在“事件查看器”中，打开“Windows 日志”节点。
1. 选择“系统”，以打开系统事件日志。 选择“应用程序”以打开应用程序事件日志。
1. 搜索与失败应用相关联的错误。

### <a name="run-the-app-at-a-command-prompt"></a>在命令提示符处运行应用

许多启动错误未在事件日志中生成有用信息。 可以通过在托管系统上在命令提示符处运行应用来找到某些错误的原因。 若要记录应用中的其他详细信息，则降低[日志级别](xref:fundamentals/logging/index#log-level)，或在[开发环境](xref:fundamentals/environments)中运行此应用。

### <a name="clear-package-caches"></a>清除包缓存

正常运行的应用在开发计算机上升级 .NET Core SDK 或在应用内更改包版本后可能会立即出现故障。 在某些情况下，不同的包可能在执行主要升级时中断应用。 可以按照以下说明来修复其中大部分问题：

1. 删除 bin 和 obj 文件夹。
1. 通过从命令行界面执行 [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) 清除包缓存。

   清除包缓存还可通过使用 [nuget.exe](https://www.nuget.org/downloads) 工具并执行命令 `nuget locals all -clear` 来完成。 *nuget.exe* 不是与 Windows 桌面操作系统的捆绑安装，必须从 [NuGet 网站](https://www.nuget.org/downloads)中单独获取。

1. 还原并重新生成项目。
1. 在重新部署应用前，在服务器上删除部署文件夹中的所有文件。

### <a name="slow-or-hanging-app"></a>应用缓慢或挂起

故障转储是系统内存的一个快照，可帮助确定应用崩溃、启动故障或应用速度缓慢等状况的原因。

#### <a name="app-crashes-or-encounters-an-exception"></a>应用崩溃或引发异常

从 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) 中获取转储并进行分析：

1. 创建文件夹，将崩溃转储文件保存在 `c:\dumps`。
1. 使用应用程序可执行文件名称运行 [EnableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/samples/scripts/EnableDumps.ps1)：

   ```powershell
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. 在造成崩溃的条件下运行应用。
1. 出现崩溃后，运行 [DisableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/samples/scripts/DisableDumps.ps1)：

   ```powershell
   .\DisableDumps {APPLICATION EXE}
   ```

在应用崩溃并完成转储收集后，即可正常终止应用。 PowerShell 脚本会 WER 来按应用收集转储（最多收集 5 个）。

> [!WARNING]
> 崩溃转储可能会占用大量磁盘空间（每个最多占用数 GB）。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>应用挂起、在启动期间失败或正常运行

如果应用挂起（停止响应但不崩溃）、在启动期间失败或者正常运行*hangs*，请参阅[用户模式转储文件：选择最佳工具](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)，以选择适合用于生成转储的工具。

#### <a name="analyze-the-dump"></a>分析转储

可采用几种方法来分析转储。 有关详细信息，请参阅[分析用户模式转储文件](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)。

## <a name="additional-resources"></a>其他资源

* [Kestrel 终结点配置](xref:fundamentals/servers/kestrel#endpoint-configuration)（包括 HTTPS 配置和 SNI 支持）
* <xref:fundamentals/host/generic-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

不使用 IIS 时，可以在 Windows 上将 ASP.NET Core 应用作为 [Windows 服务](/dotnet/framework/windows-services/introduction-to-windows-service-applications)进行托管。 作为 Windows 服务进行托管时，应用将在服务器重新启动后自动启动。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/windows-service/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="prerequisites"></a>先决条件

* [ASP.NET .NET Core SDK 2.1 或更高版本](https://dotnet.microsoft.com/download)
* [PowerShell 6.2 或更高版本](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a>应用配置

应用需要引用 [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) 和 [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) 包。

在服务之外运行时，若要进行测试和调试，请添加代码以确定应用是否作为服务或控制台应用运行。 检查是否已连接调试器或是否存在 `--console` 开关。 如果其中一个条件为 true（应用不作为服务运行），请调用 <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>。 如果条件为 false（应用作为服务运行）：

* 调用 <xref:System.IO.Directory.SetCurrentDirectory*> 并使用应用的发布位置路径。 不要调用 <xref:System.IO.Directory.GetCurrentDirectory*> 来获取路径，因为在调用 <xref:System.IO.Directory.GetCurrentDirectory*> 时，Windows 服务应用将返回 C:\\WINDOWS\\system32 文件夹。 有关详细信息，请参阅[当前目录和内容根](#current-directory-and-content-root)部分。 请先执行此步骤，然后再在 `CreateWebHostBuilder` 中配置应用。
* 调用 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> 以将应用作为服务运行。

由于[命令行配置提供程序](xref:fundamentals/configuration/index#command-line-configuration-provider)需要命令行参数的名称/值对，因此将先从参数中删除 `--console` 开关，然后 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 会接收这些参数。

若要写入 Windows 事件日志，请将事件日志提供程序添加到 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>。 使用 appsettings.Production.json文件中的 `Logging:LogLevel:Default` 键设置日志记录级别。

示例应用中的以下示例调用了 `RunAsCustomService`，来代替 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>，以处理应用内的生存期事件。 有关详细信息，请参阅[处理启动和停止事件](#handle-starting-and-stopping-events)部分。

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a>部署类型

有关部署方案的信息和建议，请参阅 [.NET Core 应用程序部署](/dotnet/core/deploying/)。

### <a name="sdk"></a>SDK

对于使用 Razor Pages 或 MVC 框架的基于 Web 应用的服务，请在项目文件中指定 Web SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

如果服务仅执行后台任务（例如 [托管服务](xref:fundamentals/host/hosted-services)），请在项目文件中指定辅助角色 SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>依赖框架的部署 (FDD)

依赖框架的部署 (FDD) 依赖目标系统上存在共享系统级版本的 .NET Core。 按照本文中的指南采用 FDD 方案时，SDK 会生成一个称为“依赖框架的可执行文件”的可执行文件 (.exe)。 

Windows [运行时标识符 (RID)](/dotnet/core/rid-catalog) ([\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier)) 包含目标框架。 在以下示例中，将 RID 设置为 `win7-x64`。 `<SelfContained>` 属性设置为 `false`。 这些属性指示 SDK 生成用于 Windows 的可执行文件 (.exe) 以及一个依赖共享 NET Core 框架的应用。

web.config文件（通常在发布 ASP.NET Core 应用时生成）对于 Windows 服务应用来说是不必要的。 若要禁止创建 web.config 文件，请将 `<IsTransformWebConfigDisabled>` 属性集添加到 `true`。

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>独立部署 (SCD)

独立部署 (SCD) 不依赖主机系统上存在的共享框架。 运行时和应用的依赖项将与应用一起部署。

将 Windows [运行时标识符 (RID)](/dotnet/core/rid-catalog) 添加到包含目标框架的 `<PropertyGroup>` 中：

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

要发布多个 RID：

* 通过以分号分隔的列表提供 RID。
* 使用属性名称 [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers)（复数）。

有关详细信息，请参阅 [.NET Core RID 目录](/dotnet/core/rid-catalog)。

将 `<SelfContained>` 属性设置为 `true`：

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a>服务用户帐户

在管理 PowerShell 6 命令行界面中运行 [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet，为服务创建用户帐户。

对于 Windows 10 2018 年 10 月更新（版本 1809/内部版本 10.0.17763）或更高版本：

```powershell
New-LocalUser -Name {SERVICE NAME}
```

对于 Windows 10 2018 年 10 月更新（版本 1809/内部版本 10.0.17763）之前的 Windows 操作系统：

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

在系统提示时，提供[强密码](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)。

除非将 `-AccountExpires` 参数提供给到期时间为 <xref:System.DateTime> 的 [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet，否则用户帐户不会到期。

有关详细信息，请参阅 [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) 和[服务用户帐户](/windows/desktop/services/service-user-accounts)。

使用 Active Directory 时管理用户的另一种方法是使用托管服务帐户。 有关详细信息，请参阅[组托管服务帐户概述](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。

## <a name="log-on-as-a-service-rights"></a>以服务身份登录权限

为服务用户帐户创建“以服务身份登录”权限：

1. 通过运行 secpool.msc，打开本地安全策略编辑器。
1. 展开“本地策略”节点，选择“用户权限分配”。 
1. 打开“以服务身份登录”策略。
1. 选择“添加用户或组”。
1. 使用下列方法之一提供对象名称（用户帐户）：
   1. 在对象名称字段键入用户帐户 (`{DOMAIN OR COMPUTER NAME\USER}`)，然后选择“确定”，以将此用户添加到策略。
   1. 选择“高级”。 选择“开始查找”。 从列表中选择该用户帐户。 选择“确定”。 再次选择“确定”，以将该用户添加到策略。
1. 选择“确定”或“应用”，以接受更改。 

## <a name="create-and-manage-the-windows-service"></a>创建和管理 Windows 服务

### <a name="create-a-service"></a>创建服务

使用 PowerShell 注册服务。 从 PowerShell 6 命令行界面，执行以下命令：

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`：应用在主机上的文件夹的路径（如 `d:\myservice`）。 请勿在此路径中包含应用的可执行文件。 尾部反斜杠是非必需项。
* `{DOMAIN OR COMPUTER NAME\USER}`：服务用户帐户（如 `Contoso\ServiceUser`）。
* `{SERVICE NAME}`：服务名称（如 `MyService`）。
* `{EXE FILE PATH}`：应用的可执行文件路径（如 `d:\myservice\myservice.exe`）。 请将可执行文件的文件名和扩展名包括在内。
* `{DESCRIPTION}`：服务说明（如 `My sample service`）。
* `{DISPLAY NAME}`：服务显示名称（如 `My Service`）。

### <a name="start-a-service"></a>启动服务

使用以下 PowerShell 6 命令启动服务：

```powershell
Start-Service -Name {SERVICE NAME}
```

此命令需要几秒钟才能启动服务。

### <a name="determine-a-services-status"></a>确信服务状态

要检查服务的状态，请使用以下 PowerShell 6 命令：

```powershell
Get-Service -Name {SERVICE NAME}
```

状态报告为以下值之一：

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>停止服务

使用以下 Powershell 6 命令停止服务：

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>删除服务

在停止服务一小段时间后，使用以下 Powershell 6 命令删除服务：

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a>处理启动和停止事件

处理 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*> 和 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*> 事件：

1. 使用 `OnStarting`、`OnStarted` 和 `OnStopping` 方法创建从 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> 派生的类：

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. 创建可将 `CustomWebHostService` 传递给 <xref:System.ServiceProcess.ServiceBase.Run*> 的 <xref:Microsoft.AspNetCore.Hosting.IWebHost> 的扩展方法：

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. 在 `Program.Main` 中，调用 `RunAsCustomService` 扩展方法，而不是 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>：

   ```csharp
   host.RunAsCustomService();
   ```

   若要在 `Program.Main` 中查看 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> 的位置，请参阅[部署类型](#deployment-type)部分中所示的代码示例。

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

与来自 Internet 或公司网络的请求进行交互且在代理或负载均衡器后方的服务可能需要其他配置。 有关详细信息，请参阅 <xref:host-and-deploy/proxy-load-balancer>。

## <a name="configure-endpoints"></a>配置终结点

默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 通过设置 `ASPNETCORE_URLS` 环境变量来配置 URL 和端口。

有关其他 URL 和端口配置方法，请参阅相关服务器文章：

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

上述指南介绍了对 HTTPS 终结点的支持。 例如，在使用 Windows 服务进行身份验证时，为 HTTPS 配置应用。

> [!NOTE]
> 不支持使用 ASP.NET Core HTTPS 开发证书保护服务终结点。

## <a name="current-directory-and-content-root"></a>当前目录和内容根

通过为 Windows 服务调用 <xref:System.IO.Directory.GetCurrentDirectory*> 返回的当前工作目录是 C:\\WINDOWS\\system32 文件夹。 system32 文件夹不是存储服务文件（如设置文件）的合适位置。 使用以下方法之一来维护和访问服务的资产和设置文件。

### <a name="set-the-content-root-path-to-the-apps-folder"></a>设置应用文件夹的内容根路径

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> 是创建服务时提供给 `binPath` 参数的同一路径。 请调用包含应用[内容根目录](xref:fundamentals/index#content-root)的路径的 <xref:System.IO.Directory.SetCurrentDirectory*>，而不是调用 `GetCurrentDirectory` 来创建设置文件的路径。

在 `Program.Main` 中，确定服务可执行文件的文件夹路径，并使用该路径来建立应用的内容根：

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>将服务的文件存储在磁盘中的合适位置

使用 <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> 时，使用 <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> 指定到包含文件的文件夹的绝对路径。

## <a name="troubleshoot"></a>疑难解答

若要排除 Windows 服务应用的故障，请参阅 <xref:test/troubleshoot>。

### <a name="common-errors"></a>常见错误

* 正在使用 PowerShell 的早期版本或预发布版本。
* 注册的服务未使用来自 [dotnet publish](/dotnet/core/tools/dotnet-publish) 命令的应用的已发布输出。 应用部署不支持 [dotnet build](/dotnet/core/tools/dotnet-build) 命令的输出。 已发布的资产位于以下文件夹之一中，具体取决于部署类型：
  * *bin/Release/{TARGET FRAMEWORK}/publish* (FDD)
  * *bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)
* 服务未处于“正在运行”状态。
* 应用使用的资源（例如证书）的路径不正确。 Windows 服务的基本路径是“c:\\Windows\\System32”。
* 用户没有“以服务身份登录”权限。
* 在执行 `New-Service` PowerShell 命令时，用户密码已过期，或以不正确的方式传递。
* 应用需要进行 ASP.NET Core 身份验证，但是未配置安全连接 (HTTPS)。
* 请求 URL 端口不正确或未在应用中正确配置。

### <a name="system-and-application-event-logs"></a>系统和应用程序事件日志

访问系统和应用程序事件日志：

1. 打开“开始”菜单，搜索“事件查看器”，然后选择“事件查看器”应用。
1. 在“事件查看器”中，打开“Windows 日志”节点。
1. 选择“系统”，以打开系统事件日志。 选择“应用程序”以打开应用程序事件日志。
1. 搜索与失败应用相关联的错误。

### <a name="run-the-app-at-a-command-prompt"></a>在命令提示符处运行应用

许多启动错误未在事件日志中生成有用信息。 可以通过在托管系统上在命令提示符处运行应用来找到某些错误的原因。 若要记录应用中的其他详细信息，则降低[日志级别](xref:fundamentals/logging/index#log-level)，或在[开发环境](xref:fundamentals/environments)中运行此应用。

### <a name="clear-package-caches"></a>清除包缓存

正常运行的应用在开发计算机上升级 .NET Core SDK 或在应用内更改包版本后可能会立即出现故障。 在某些情况下，不同的包可能在执行主要升级时中断应用。 可以按照以下说明来修复其中大部分问题：

1. 删除 bin 和 obj 文件夹。
1. 通过从命令行界面执行 [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) 清除包缓存。

   清除包缓存还可通过使用 [nuget.exe](https://www.nuget.org/downloads) 工具并执行命令 `nuget locals all -clear` 来完成。 *nuget.exe* 不是与 Windows 桌面操作系统的捆绑安装，必须从 [NuGet 网站](https://www.nuget.org/downloads)中单独获取。

1. 还原并重新生成项目。
1. 在重新部署应用前，在服务器上删除部署文件夹中的所有文件。

### <a name="slow-or-hanging-app"></a>应用缓慢或挂起

故障转储是系统内存的一个快照，可帮助确定应用崩溃、启动故障或应用速度缓慢等状况的原因。

#### <a name="app-crashes-or-encounters-an-exception"></a>应用崩溃或引发异常

从 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) 中获取转储并进行分析：

1. 创建文件夹，将崩溃转储文件保存在 `c:\dumps`。
1. 使用应用程序可执行文件名称运行 [EnableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1)：

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. 在造成崩溃的条件下运行应用。
1. 出现崩溃后，运行 [DisableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1)：

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

在应用崩溃并完成转储收集后，即可正常终止应用。 PowerShell 脚本会 WER 来按应用收集转储（最多收集 5 个）。

> [!WARNING]
> 崩溃转储可能会占用大量磁盘空间（每个最多占用数 GB）。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>应用挂起、在启动期间失败或正常运行

如果应用挂起（停止响应但不崩溃）、在启动期间失败或者正常运行*hangs*，请参阅[用户模式转储文件：选择最佳工具](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)，以选择适合用于生成转储的工具。

#### <a name="analyze-the-dump"></a>分析转储

可采用几种方法来分析转储。 有关详细信息，请参阅[分析用户模式转储文件](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)。

## <a name="additional-resources"></a>其他资源

* [Kestrel 终结点配置](xref:fundamentals/servers/kestrel#endpoint-configuration)（包括 HTTPS 配置和 SNI 支持）
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end

::: moniker range="< aspnetcore-2.2"

不使用 IIS 时，可以在 Windows 上将 ASP.NET Core 应用作为 [Windows 服务](/dotnet/framework/windows-services/introduction-to-windows-service-applications)进行托管。 作为 Windows 服务进行托管时，应用将在服务器重新启动后自动启动。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/windows-service/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="prerequisites"></a>先决条件

* [ASP.NET .NET Core SDK 2.1 或更高版本](https://dotnet.microsoft.com/download)
* [PowerShell 6.2 或更高版本](https://github.com/PowerShell/PowerShell)

## <a name="app-configuration"></a>应用配置

应用需要引用 [Microsoft.AspNetCore.Hosting.WindowsServices](https://www.nuget.org/packages/Microsoft.AspNetCore.Hosting.WindowsServices) 和 [Microsoft.Extensions.Logging.EventLog](https://www.nuget.org/packages/Microsoft.Extensions.Logging.EventLog) 包。

在服务之外运行时，若要进行测试和调试，请添加代码以确定应用是否作为服务或控制台应用运行。 检查是否已连接调试器或是否存在 `--console` 开关。 如果其中一个条件为 true（应用不作为服务运行），请调用 <xref:Microsoft.AspNetCore.Hosting.WebHostExtensions.Run*>。 如果条件为 false（应用作为服务运行）：

* 调用 <xref:System.IO.Directory.SetCurrentDirectory*> 并使用应用的发布位置路径。 不要调用 <xref:System.IO.Directory.GetCurrentDirectory*> 来获取路径，因为在调用 <xref:System.IO.Directory.GetCurrentDirectory*> 时，Windows 服务应用将返回 C:\\WINDOWS\\system32 文件夹。 有关详细信息，请参阅[当前目录和内容根](#current-directory-and-content-root)部分。 请先执行此步骤，然后再在 `CreateWebHostBuilder` 中配置应用。
* 调用 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> 以将应用作为服务运行。

由于[命令行配置提供程序](xref:fundamentals/configuration/index#command-line-configuration-provider)需要命令行参数的名称/值对，因此将先从参数中删除 `--console` 开关，然后 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*> 会接收这些参数。

若要写入 Windows 事件日志，请将事件日志提供程序添加到 <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder.ConfigureLogging*>。 使用 appsettings.Production.json文件中的 `Logging:LogLevel:Default` 键设置日志记录级别。

示例应用中的以下示例调用了 `RunAsCustomService`，来代替 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>，以处理应用内的生存期事件。 有关详细信息，请参阅[处理启动和停止事件](#handle-starting-and-stopping-events)部分。

[!code-csharp[](windows-service/samples/2.x/AspNetCoreService/Program.cs?name=snippet_Program)]

## <a name="deployment-type"></a>部署类型

有关部署方案的信息和建议，请参阅 [.NET Core 应用程序部署](/dotnet/core/deploying/)。

### <a name="sdk"></a>SDK

对于使用 Razor Pages 或 MVC 框架的基于 Web 应用的服务，请在项目文件中指定 Web SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

如果服务仅执行后台任务（例如 [托管服务](xref:fundamentals/host/hosted-services)），请在项目文件中指定辅助角色 SDK：

```xml
<Project Sdk="Microsoft.NET.Sdk.Worker">
```

### <a name="framework-dependent-deployment-fdd"></a>依赖框架的部署 (FDD)

依赖框架的部署 (FDD) 依赖目标系统上存在共享系统级版本的 .NET Core。 按照本文中的指南采用 FDD 方案时，SDK 会生成一个称为“依赖框架的可执行文件”的可执行文件 (.exe)。 

Windows [运行时标识符 (RID)](/dotnet/core/rid-catalog) ([\<RuntimeIdentifier>](/dotnet/core/tools/csproj#runtimeidentifier)) 包含目标框架。 在以下示例中，将 RID 设置为 `win7-x64`。 `<SelfContained>` 属性设置为 `false`。 这些属性指示 SDK 生成用于 Windows 的可执行文件 (.exe) 以及一个依赖共享 NET Core 框架的应用。

`<UseAppHost>` 属性设置为 `true`。 此属性为服务提供 FDD 的激活路径（一个可执行文件，格式为 .exe）。

web.config文件（通常在发布 ASP.NET Core 应用时生成）对于 Windows 服务应用来说是不必要的。 若要禁止创建 web.config 文件，请将 `<IsTransformWebConfigDisabled>` 属性集添加到 `true`。

```xml
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <RuntimeIdentifier>win7-x64</RuntimeIdentifier>
  <UseAppHost>true</UseAppHost>
  <SelfContained>false</SelfContained>
  <IsTransformWebConfigDisabled>true</IsTransformWebConfigDisabled>
</PropertyGroup>
```

### <a name="self-contained-deployment-scd"></a>独立部署 (SCD)

独立部署 (SCD) 不依赖主机系统上存在的共享框架。 运行时和应用的依赖项将与应用一起部署。

将 Windows [运行时标识符 (RID)](/dotnet/core/rid-catalog) 添加到包含目标框架的 `<PropertyGroup>` 中：

```xml
<RuntimeIdentifier>win7-x64</RuntimeIdentifier>
```

要发布多个 RID：

* 通过以分号分隔的列表提供 RID。
* 使用属性名称 [\<RuntimeIdentifiers>](/dotnet/core/tools/csproj#runtimeidentifiers)（复数）。

有关详细信息，请参阅 [.NET Core RID 目录](/dotnet/core/rid-catalog)。

将 `<SelfContained>` 属性设置为 `true`：

```xml
<SelfContained>true</SelfContained>
```

## <a name="service-user-account"></a>服务用户帐户

在管理 PowerShell 6 命令行界面中运行 [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet，为服务创建用户帐户。

对于 Windows 10 2018 年 10 月更新（版本 1809/内部版本 10.0.17763）或更高版本：

```powershell
New-LocalUser -Name {SERVICE NAME}
```

对于 Windows 10 2018 年 10 月更新（版本 1809/内部版本 10.0.17763）之前的 Windows 操作系统：

```console
powershell -Command "New-LocalUser -Name {SERVICE NAME}"
```

在系统提示时，提供[强密码](/windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)。

除非将 `-AccountExpires` 参数提供给到期时间为 <xref:System.DateTime> 的 [New-LocalUser](/powershell/module/microsoft.powershell.localaccounts/new-localuser) cmdlet，否则用户帐户不会到期。

有关详细信息，请参阅 [Microsoft.PowerShell.LocalAccounts](/powershell/module/microsoft.powershell.localaccounts/) 和[服务用户帐户](/windows/desktop/services/service-user-accounts)。

使用 Active Directory 时管理用户的另一种方法是使用托管服务帐户。 有关详细信息，请参阅[组托管服务帐户概述](/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview)。

## <a name="log-on-as-a-service-rights"></a>以服务身份登录权限

为服务用户帐户创建“以服务身份登录”权限：

1. 通过运行 secpool.msc，打开本地安全策略编辑器。
1. 展开“本地策略”节点，选择“用户权限分配”。 
1. 打开“以服务身份登录”策略。
1. 选择“添加用户或组”。
1. 使用下列方法之一提供对象名称（用户帐户）：
   1. 在对象名称字段键入用户帐户 (`{DOMAIN OR COMPUTER NAME\USER}`)，然后选择“确定”，以将此用户添加到策略。
   1. 选择“高级”。 选择“开始查找”。 从列表中选择该用户帐户。 选择“确定”。 再次选择“确定”，以将该用户添加到策略。
1. 选择“确定”或“应用”，以接受更改。 

## <a name="create-and-manage-the-windows-service"></a>创建和管理 Windows 服务

### <a name="create-a-service"></a>创建服务

使用 PowerShell 注册服务。 从 PowerShell 6 命令行界面，执行以下命令：

```powershell
$acl = Get-Acl "{EXE PATH}"
$aclRuleArgs = {DOMAIN OR COMPUTER NAME\USER}, "Read,Write,ReadAndExecute", "ContainerInherit,ObjectInherit", "None", "Allow"
$accessRule = New-Object System.Security.AccessControl.FileSystemAccessRule($aclRuleArgs)
$acl.SetAccessRule($accessRule)
$acl | Set-Acl "{EXE PATH}"

New-Service -Name {SERVICE NAME} -BinaryPathName {EXE FILE PATH} -Credential {DOMAIN OR COMPUTER NAME\USER} -Description "{DESCRIPTION}" -DisplayName "{DISPLAY NAME}" -StartupType Automatic
```

* `{EXE PATH}`：应用在主机上的文件夹的路径（如 `d:\myservice`）。 请勿在此路径中包含应用的可执行文件。 尾部反斜杠是非必需项。
* `{DOMAIN OR COMPUTER NAME\USER}`：服务用户帐户（如 `Contoso\ServiceUser`）。
* `{SERVICE NAME}`：服务名称（如 `MyService`）。
* `{EXE FILE PATH}`：应用的可执行文件路径（如 `d:\myservice\myservice.exe`）。 请将可执行文件的文件名和扩展名包括在内。
* `{DESCRIPTION}`：服务说明（如 `My sample service`）。
* `{DISPLAY NAME}`：服务显示名称（如 `My Service`）。

### <a name="start-a-service"></a>启动服务

使用以下 PowerShell 6 命令启动服务：

```powershell
Start-Service -Name {SERVICE NAME}
```

此命令需要几秒钟才能启动服务。

### <a name="determine-a-services-status"></a>确信服务状态

要检查服务的状态，请使用以下 PowerShell 6 命令：

```powershell
Get-Service -Name {SERVICE NAME}
```

状态报告为以下值之一：

* `Starting`
* `Running`
* `Stopping`
* `Stopped`

### <a name="stop-a-service"></a>停止服务

使用以下 Powershell 6 命令停止服务：

```powershell
Stop-Service -Name {SERVICE NAME}
```

### <a name="remove-a-service"></a>删除服务

在停止服务一小段时间后，使用以下 Powershell 6 命令删除服务：

```powershell
Remove-Service -Name {SERVICE NAME}
```

## <a name="handle-starting-and-stopping-events"></a>处理启动和停止事件

处理 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarting*>、<xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStarted*> 和 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService.OnStopping*> 事件：

1. 使用 `OnStarting`、`OnStarted` 和 `OnStopping` 方法创建从 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostService> 派生的类：

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/CustomWebHostService.cs?name=snippet_CustomWebHostService)]

2. 创建可将 `CustomWebHostService` 传递给 <xref:System.ServiceProcess.ServiceBase.Run*> 的 <xref:Microsoft.AspNetCore.Hosting.IWebHost> 的扩展方法：

   [!code-csharp[](windows-service/samples/2.x/AspNetCoreService/WebHostServiceExtensions.cs?name=ExtensionsClass)]

3. 在 `Program.Main` 中，调用 `RunAsCustomService` 扩展方法，而不是 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*>：

   ```csharp
   host.RunAsCustomService();
   ```

   若要在 `Program.Main` 中查看 <xref:Microsoft.AspNetCore.Hosting.WindowsServices.WebHostWindowsServiceExtensions.RunAsService*> 的位置，请参阅[部署类型](#deployment-type)部分中所示的代码示例。

## <a name="proxy-server-and-load-balancer-scenarios"></a>代理服务器和负载均衡器方案

与来自 Internet 或公司网络的请求进行交互且在代理或负载均衡器后方的服务可能需要其他配置。 有关详细信息，请参阅 <xref:host-and-deploy/proxy-load-balancer>。

## <a name="configure-endpoints"></a>配置终结点

默认情况下，ASP.NET Core 绑定到 `http://localhost:5000`。 通过设置 `ASPNETCORE_URLS` 环境变量来配置 URL 和端口。

有关其他 URL 和端口配置方法，请参阅相关服务器文章：

* <xref:fundamentals/servers/kestrel#endpoint-configuration>
* <xref:fundamentals/servers/httpsys#configure-windows-server>

上述指南介绍了对 HTTPS 终结点的支持。 例如，在使用 Windows 服务进行身份验证时，为 HTTPS 配置应用。

> [!NOTE]
> 不支持使用 ASP.NET Core HTTPS 开发证书保护服务终结点。

## <a name="current-directory-and-content-root"></a>当前目录和内容根

通过为 Windows 服务调用 <xref:System.IO.Directory.GetCurrentDirectory*> 返回的当前工作目录是 C:\\WINDOWS\\system32 文件夹。 system32 文件夹不是存储服务文件（如设置文件）的合适位置。 使用以下方法之一来维护和访问服务的资产和设置文件。

### <a name="set-the-content-root-path-to-the-apps-folder"></a>设置应用文件夹的内容根路径

<xref:Microsoft.Extensions.Hosting.IHostingEnvironment.ContentRootPath*> 是创建服务时提供给 `binPath` 参数的同一路径。 请调用包含应用[内容根目录](xref:fundamentals/index#content-root)的路径的 <xref:System.IO.Directory.SetCurrentDirectory*>，而不是调用 `GetCurrentDirectory` 来创建设置文件的路径。

在 `Program.Main` 中，确定服务可执行文件的文件夹路径，并使用该路径来建立应用的内容根：

```csharp
var pathToExe = Process.GetCurrentProcess().MainModule.FileName;
var pathToContentRoot = Path.GetDirectoryName(pathToExe);
Directory.SetCurrentDirectory(pathToContentRoot);

CreateWebHostBuilder(args)
    .Build()
    .RunAsService();
```

### <a name="store-a-services-files-in-a-suitable-location-on-disk"></a>将服务的文件存储在磁盘中的合适位置

使用 <xref:Microsoft.Extensions.Configuration.IConfigurationBuilder> 时，使用 <xref:Microsoft.Extensions.Configuration.FileConfigurationExtensions.SetBasePath*> 指定到包含文件的文件夹的绝对路径。

## <a name="troubleshoot"></a>疑难解答

若要排除 Windows 服务应用的故障，请参阅 <xref:test/troubleshoot>。

### <a name="common-errors"></a>常见错误

* 正在使用 PowerShell 的早期版本或预发布版本。
* 注册的服务未使用来自 [dotnet publish](/dotnet/core/tools/dotnet-publish) 命令的应用的已发布输出。 应用部署不支持 [dotnet build](/dotnet/core/tools/dotnet-build) 命令的输出。 已发布的资产位于以下文件夹之一中，具体取决于部署类型：
  * *bin/Release/{TARGET FRAMEWORK}/publish* (FDD)
  * *bin/Release/{TARGET FRAMEWORK}/{RUNTIME IDENTIFIER}/publish* (SCD)
* 服务未处于“正在运行”状态。
* 应用使用的资源（例如证书）的路径不正确。 Windows 服务的基本路径是“c:\\Windows\\System32”。
* 用户没有“以服务身份登录”权限。
* 在执行 `New-Service` PowerShell 命令时，用户密码已过期，或以不正确的方式传递。
* 应用需要进行 ASP.NET Core 身份验证，但是未配置安全连接 (HTTPS)。
* 请求 URL 端口不正确或未在应用中正确配置。

### <a name="system-and-application-event-logs"></a>系统和应用程序事件日志

访问系统和应用程序事件日志：

1. 打开“开始”菜单，搜索“事件查看器”，然后选择“事件查看器”应用。
1. 在“事件查看器”中，打开“Windows 日志”节点。
1. 选择“系统”，以打开系统事件日志。 选择“应用程序”以打开应用程序事件日志。
1. 搜索与失败应用相关联的错误。

### <a name="run-the-app-at-a-command-prompt"></a>在命令提示符处运行应用

许多启动错误未在事件日志中生成有用信息。 可以通过在托管系统上在命令提示符处运行应用来找到某些错误的原因。 若要记录应用中的其他详细信息，则降低[日志级别](xref:fundamentals/logging/index#log-level)，或在[开发环境](xref:fundamentals/environments)中运行此应用。

### <a name="clear-package-caches"></a>清除包缓存

正常运行的应用在开发计算机上升级 .NET Core SDK 或在应用内更改包版本后可能会立即出现故障。 在某些情况下，不同的包可能在执行主要升级时中断应用。 可以按照以下说明来修复其中大部分问题：

1. 删除 bin 和 obj 文件夹。
1. 通过从命令行界面执行 [dotnet nuget locals all --clear](/dotnet/core/tools/dotnet-nuget-locals) 清除包缓存。

   清除包缓存还可通过使用 [nuget.exe](https://www.nuget.org/downloads) 工具并执行命令 `nuget locals all -clear` 来完成。 *nuget.exe* 不是与 Windows 桌面操作系统的捆绑安装，必须从 [NuGet 网站](https://www.nuget.org/downloads)中单独获取。

1. 还原并重新生成项目。
1. 在重新部署应用前，在服务器上删除部署文件夹中的所有文件。

### <a name="slow-or-hanging-app"></a>应用缓慢或挂起

故障转储是系统内存的一个快照，可帮助确定应用崩溃、启动故障或应用速度缓慢等状况的原因。

#### <a name="app-crashes-or-encounters-an-exception"></a>应用崩溃或引发异常

从 [Windows 错误报告 (WER)](/windows/desktop/wer/windows-error-reporting) 中获取转储并进行分析：

1. 创建文件夹，将崩溃转储文件保存在 `c:\dumps`。
1. 使用应用程序可执行文件名称运行 [EnableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/EnableDumps.ps1)：

   ```console
   .\EnableDumps {APPLICATION EXE} c:\dumps
   ```

1. 在造成崩溃的条件下运行应用。
1. 出现崩溃后，运行 [DisableDumps PowerShell 脚本](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/host-and-deploy/windows-service/scripts/DisableDumps.ps1)：

   ```console
   .\DisableDumps {APPLICATION EXE}
   ```

在应用崩溃并完成转储收集后，即可正常终止应用。 PowerShell 脚本会 WER 来按应用收集转储（最多收集 5 个）。

> [!WARNING]
> 崩溃转储可能会占用大量磁盘空间（每个最多占用数 GB）。

#### <a name="app-hangs-fails-during-startup-or-runs-normally"></a>应用挂起、在启动期间失败或正常运行

如果应用挂起（停止响应但不崩溃）、在启动期间失败或者正常运行*hangs*，请参阅[用户模式转储文件：选择最佳工具](/windows-hardware/drivers/debugger/user-mode-dump-files#choosing-the-best-tool)，以选择适合用于生成转储的工具。

#### <a name="analyze-the-dump"></a>分析转储

可采用几种方法来分析转储。 有关详细信息，请参阅[分析用户模式转储文件](/windows-hardware/drivers/debugger/analyzing-a-user-mode-dump-file)。

## <a name="additional-resources"></a>其他资源

* [Kestrel 终结点配置](xref:fundamentals/servers/kestrel#endpoint-configuration)（包括 HTTPS 配置和 SNI 支持）
* <xref:fundamentals/host/web-host>
* <xref:test/troubleshoot>

::: moniker-end
