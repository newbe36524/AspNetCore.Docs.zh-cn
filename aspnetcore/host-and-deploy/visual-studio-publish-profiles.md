---
title: 用于 ASP.NET Core 应用部署的 Visual Studio 发布配置文件 (.pubxml)
author: rick-anderson
description: 了解如何在 Visual Studio 中创建发布配置文件，并将它们用于管理针对各种目标的 ASP.NET Core 应用部署。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/28/2020
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
uid: host-and-deploy/visual-studio-publish-profiles
ms.openlocfilehash: e86a648e2707e808452e3bc92d991058c8fde480
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91754640"
---
# <a name="visual-studio-publish-profiles-pubxml-for-aspnet-core-app-deployment"></a>用于 ASP.NET Core 应用部署的 Visual Studio 发布配置文件 (.pubxml)

作者：[Sayed Ibrahim Hashimi](https://github.com/sayedihashimi) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

本文档重点介绍了如何通过 Visual Studio 2019 或更高版本创建和使用发布配置文件。 通过 Visual Studio 创建的发布配置文件可用于 MSBuild 和 Visual Studio。 要查看说明了解如何发布到 Azure，请参阅 <xref:tutorials/publish-to-azure-webapp-using-vs>。

`dotnet new mvc` 命令生成包含以下根级别 [\<Project> 元素](/visualstudio/msbuild/project-element-msbuild)的项目文件：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
    <!-- omitted for brevity -->
</Project>
```

前导 `<Project>` 元素的 `Sdk` 属性分别从 $(MSBuildSDKsPath)\Microsoft.NET.Sdk.Web\Sdk\Sdk.props 和 $(MSBuildSDKsPath)\Microsoft.NET.Sdk.Web\Sdk\Sdk.targets 导入 MSBuild [属性](/visualstudio/msbuild/msbuild-properties)和[目标](/visualstudio/msbuild/msbuild-targets)。 `$(MSBuildSDKsPath)`（装有 Visual Studio 2019 Enterprise）的默认位置是 %programfiles(x86)%\Microsoft Visual Studio\2019\Enterprise\MSBuild\Sdks 文件夹。

`Microsoft.NET.Sdk.Web` ([Web SDK](xref:razor-pages/web-sdk)) 依赖其他 SDK，包括 `Microsoft.NET.Sdk` ([.NET Core SDK](/dotnet/core/project-sdk/msbuild-props)) 和 `Microsoft.NET.Sdk.Razor` ([Razor Razor SDK](xref:razor-pages/sdk))。 将导入与每个从属 SDK 关联的 MSBuild 属性和目标。 发布目标将根据使用的发布方法，导入相应的目标集。

MSBuild 或 Visual Studio 加载项目时，执行下列高级别操作：

* 生成项目
* 计算要发布的文件
* 将文件发布到目标

## <a name="compute-project-items"></a>计算项目项

加载项目时，将计算 [MSBuild 项目项](/visualstudio/msbuild/common-msbuild-project-items)（文件）。 项类型确定如何处理该文件。 默认情况下，.cs 文件包含在 `Compile` 项列表内。 会对 `Compile` 项列表中的文件进行编译。

除了生成输出，`Content` 项列表还包含已发布的文件。 默认情况下，匹配模式 `wwwroot\**`、`**\*.config` 和 `**\*.json` 的文件包含在 `Content` 项列表内。 例如，`wwwroot\**`[glob 模式](https://gruntjs.com/configuring-tasks#globbing-patterns)与 wwwroot 文件夹及其子文件夹中的所有文件相匹配。

::: moniker range=">= aspnetcore-3.0"

[Web SDK](xref:razor-pages/web-sdk) 导入 [Razor SDK](xref:razor-pages/sdk)。 因此，匹配模式 `**\*.cshtml` 和 `**\*.razor` 的文件也同时包含在 `Content` 项列表内。

::: moniker-end

::: moniker range=">= aspnetcore-2.1 <= aspnetcore-2.2"

[Web SDK](xref:razor-pages/web-sdk) 导入 [Razor SDK](xref:razor-pages/sdk)。 因此，匹配 `**\*.cshtml` 模式的文件也同时包含在 `Content` 项列表内。

::: moniker-end

若要将文件显式添加到发布列表，请直接在 .csproj 文件中添加文件，如[包含文件](#include-files)一节中所示。

在 Visual Studio 中选择“发布”按钮时或从命令行发布时：

* 计算属性/项目（需要生成的文件）。
* **仅限 Visual Studio**：NuGet 包已还原。 （用户需要在 CLI 上执行显式还原。）
* 生成项目。
* 计算发布项（需要发布的文件）。
* 文件已发布（计算的文件将被复制到发布目标）。

当 ASP.NET Core 项目在项目文件中引用 `Microsoft.NET.Sdk.Web` 时，会将 app_offline.htm 文件放在 Web 应用目录的根目录下。 该文件存在时，ASP.NET Core 模块会在部署过程中正常关闭该应用并提供 app_offline.htm 文件。 有关详细信息，请参阅 [ASP.NET Core 模块配置参考](xref:host-and-deploy/aspnet-core-module#app_offlinehtm)。

## <a name="basic-command-line-publishing"></a>基本命令行发布

命令行发布适用于所有支持 .NET Core 的平台，而且不需要 Visual Studio。 在下面的示例中，从项目目录（其中包含 .csproj文件）运行 .NET Core CLI 的 [dotnet publish](/dotnet/core/tools/dotnet-publish) 命令。 如果当前工作目录中没有项目文件夹，则在项目文件路径中显式传递。 例如：

```dotnetcli
dotnet publish C:\Webs\Web1
```

运行以下命令以创建并发布 Web 应用：

```dotnetcli
dotnet new mvc
dotnet publish
```

`dotnet publish` 会生成以下输出的变体：

```console
C:\Webs\Web1>dotnet publish
Microsoft (R) Build Engine version {VERSION} for .NET Core
Copyright (C) Microsoft Corporation. All rights reserved.

  Restore completed in 36.81 ms for C:\Webs\Web1\Web1.csproj.
  Web1 -> C:\Webs\Web1\bin\Debug\{TARGET FRAMEWORK MONIKER}\Web1.dll
  Web1 -> C:\Webs\Web1\bin\Debug\{TARGET FRAMEWORK MONIKER}\Web1.Views.dll
  Web1 -> C:\Webs\Web1\bin\Debug\{TARGET FRAMEWORK MONIKER}\publish\
```

默认发布文件夹格式为 bin\Debug\\{TARGET FRAMEWORK MONIKER}\publish\\。 例如 bin\Debug\netcoreapp2.2\publish\\。

以下命令指定 `Release` 生成和发布目录：

```dotnetcli
dotnet publish -c Release -o C:\MyWebs\test
```

`dotnet publish` 命令调用 MSBuild，后者会调用 `Publish` 目标。 任何传递给 `dotnet publish` 的参数都将传递给 MSBuild。 `-c` 和 `-o` 参数分别映射到 MSBuild 的 `Configuration` 和 `OutputPath` 属性。

可以使用以下任一格式来传递 MSBuild 属性：

* `-p:<NAME>=<VALUE>`
* `/p:<NAME>=<VALUE>`

例如，以下命令将 `Release` 版本发布到网络共享。 网络共享通过正斜杠指定 (//r8/) 并适用于所有支持 .NET Core 的平台。

```dotnetcli
dotnet publish -c Release /p:PublishDir=//r8/release/AdminWeb
```

确认用于部署的发布应用未在运行。 如果应用正在运行，publish 文件夹中的文件会被锁定。 部署不会发生，因为无法复制锁定的文件。

## <a name="publish-profiles"></a>发布配置文件

本部分使用 Visual Studio 2019 或更高版本来创建发布配置文件。 创建配置文件后，可以从 Visual Studio 或命令行进行发布。 发布配置文件可以简化发布过程，并且可以存在任意数量的配置文件。

通过选择以下路径之一在 Visual Studio 中创建发布配置文件：

* 在“解决方案资源管理器”中，右键单击该项目并选择“发布”。
* 可以从“生成”菜单中选择“发布 {项目名称}” 。

随即显示应用程序容量页的“发布”选项卡。 如果项目缺少发布配置文件，将显示“选择发布目标”页面。 系统会要求选择下述发布目标之一：

* Azure 应用服务
* Linux 上的 Azure 应用服务
* Azure 虚拟机
* 文件夹
* IIS、FTP、Web 部署（面向任何 Web 服务器）
* 导入配置文件

要确定最适合的发布目标，请参阅[哪些发布选项适合我](/visualstudio/ide/not-in-toc/web-publish-options)。

当选择“文件夹”发布目标时，指定一个文件夹路径来存储发布的资产。 默认文件夹路径为 bin\\{PROJECT CONFIGURATION}\\{TARGET FRAMEWORK MONIKER}\publish\\。 例如 bin\Release\netcoreapp2.2\publish\\。 选择“创建配置文件”按钮完成操作。

创建发布配置文件后，“发布”选项卡的内容将更改。 新创建的配置文件显示在下拉列表中。 在下拉列表中，选择“创建新配置文件”，再新建一个配置文件。

Visual Studio 的生成工具会生成一个 Properties/PublishProfiles/{PROFILE NAME}.pubxml MSBuild 文件，它描述了发布配置文件。 .pubxml 文件：

* 包含发布配置设置并由发布过程使用。
* 可更改它来自定义生成和发布过程。

发布到 Azure 目标时，.pubxml 文件包含 Azure 订阅标识符。 不建议使用该目标类型将此文件添加到源代码管理。 发布到非 Azure 目标时，签入 .pubxml 文件是安全的。

敏感信息（如发布密码）在每个用户/机器级别均进行加密。 它存储在 Properties/PublishProfiles/{PROFILE NAME}.pubxml.user 文件中。 由于此文件可以存储敏感信息，因此不应将其签入源代码管理。

要简要了解如何发布 ASP.NET Core Web 应用，请参阅 <xref:host-and-deploy/index>。 发布 ASP.NET Core Web 应用所需的 MSBuild 任务和目标在 [dotnet/websdk repository](https://github.com/dotnet/websdk) 中是开放源代码的。

以下命令可使用文件夹、MSDeploy 和 [Kudu](https://github.com/projectkudu/kudu/wiki) 发布配置文件。 MSDeploy 缺少跨平台支持，因此仅支持在 Windows 上使用以下 MSDeploy 选项。

文件夹（跨平台工作）：

```dotnetcli
dotnet publish WebApplication.csproj /p:PublishProfile=<FolderProfileName>
```

```dotnetcli
dotnet build WebApplication.csproj /p:DeployOnBuild=true /p:PublishProfile=<FolderProfileName>
```

MSDeploy：

```dotnetcli
dotnet publish WebApplication.csproj /p:PublishProfile=<MsDeployProfileName> /p:Password=<DeploymentPassword>
```

```dotnetcli
dotnet build WebApplication.csproj /p:DeployOnBuild=true /p:PublishProfile=<MsDeployProfileName> /p:Password=<DeploymentPassword>
```

MSDeploy 包：

```dotnetcli
dotnet publish WebApplication.csproj /p:PublishProfile=<MsDeployPackageProfileName>
```

```dotnetcli
dotnet build WebApplication.csproj /p:DeployOnBuild=true /p:PublishProfile=<MsDeployPackageProfileName>
```

在上面的示例中：

* `dotnet publish` 和 `dotnet build` 支持 Kudu API 从任何平台发布到 Azure。 Visual Studio 发布支持 Kudu API，但 WebSDK 支持其跨平台发布到 Azure。
* 不要将 `DeployOnBuild` 传递到 `dotnet publish` 命令。

有关详细信息，请参阅 [Microsoft.NET.Sdk.Publish](https://github.com/dotnet/websdk#microsoftnetsdkpublish)。

向项目的 Properties/PublishProfiles 文件夹添加包含以下内容的发布配置文件：

```xml
<Project>
  <PropertyGroup>
    <PublishProtocol>Kudu</PublishProtocol>
    <PublishSiteName>nodewebapp</PublishSiteName>
    <UserName>username</UserName>
    <Password>password</Password>
  </PropertyGroup>
</Project>
```

## <a name="folder-publish-example"></a>文件夹发布示例

当使用名为 FolderProfile 的配置文件发布时，请使用以下命令之一：

```dotnetcli
dotnet publish /p:Configuration=Release /p:PublishProfile=FolderProfile`
```

```dotnetcli
dotnet build /p:DeployOnBuild=true /p:PublishProfile=FolderProfile
```

```bash
msbuild /p:DeployOnBuild=true /p:PublishProfile=FolderProfile
```

.NET Core CLI 的 [dotnet build](/dotnet/core/tools/dotnet-build) 命令会调用 `msbuild` 来运行生成和发布过程。 在文件夹配置文件中传递时，`dotnet build` 和 `msbuild` 命令作用相同。 在 Windows 上直接调用 `msbuild` 时，将使用 MSBuild 的 .NET Framework 版本。 在非文件夹配置文件上调用 `dotnet build`：

* 调用 `msbuild`后者使用 MSDeploy。
* 结果失败（即使在 Windows 上运行时）。 要使用非文件夹配置文件发布，请直接调用 `msbuild`。

以下文件夹发布配置文件通过 Visual Studio 创建，并被发布到网络共享：

```xml
<?xml version="1.0" encoding="utf-8"?>
<!--
This file is used by the publish/package process of your Web project.
You can customize the behavior of this process by editing this 
MSBuild file.
-->
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <PropertyGroup>
    <WebPublishMethod>FileSystem</WebPublishMethod>
    <PublishProvider>FileSystem</PublishProvider>
    <LastUsedBuildConfiguration>Release</LastUsedBuildConfiguration>
    <LastUsedPlatform>Any CPU</LastUsedPlatform>
    <SiteUrlToLaunchAfterPublish />
    <LaunchSiteAfterPublish>True</LaunchSiteAfterPublish>
    <ExcludeApp_Data>False</ExcludeApp_Data>
    <PublishFramework>netcoreapp1.1</PublishFramework>
    <ProjectGuid>c30c453c-312e-40c4-aec9-394a145dee0b</ProjectGuid>
    <publishUrl>\\r8\Release\AdminWeb</publishUrl>
    <DeleteExistingFiles>False</DeleteExistingFiles>
  </PropertyGroup>
</Project>
```

在上面的示例中：

* `<ExcludeApp_Data>` 属性仅为满足 XML 架构要求。 `<ExcludeApp_Data>` 属性不影响发布过程，即使项目根中存在 App_Data 文件夹也是如此。 App_Data 文件夹不会像在 ASP.NET 4.x 项目中那样得到特殊对待。
* `<LastUsedBuildConfiguration>` 属性设置为 `Release`。 从 Visual Studio 发布时，启动发布过程后将使用该值设置 `<LastUsedBuildConfiguration>` 的值。 `<LastUsedBuildConfiguration>` 比较特殊，不得在导入的 MSBuild 文件中覆盖它。 但是，可通过下述方法之一在命令行中覆盖此属性。
  * 使用 .NET Core CLI：

    ```dotnetcli
    dotnet publish /p:Configuration=Release /p:PublishProfile=FolderProfile
    ```

    ```dotnetcli
    dotnet build -c Release /p:DeployOnBuild=true /p:PublishProfile=FolderProfile
    ```

  * 使用 MSBuild：

    ```bash
    msbuild /p:Configuration=Release /p:DeployOnBuild=true /p:PublishProfile=FolderProfile
    ```

  有关详细信息，请参阅 [MSBuild：如何设置配置属性](http://sedodream.com/2012/10/27/MSBuildHowToSetTheConfigurationProperty.aspx)。

## <a name="publish-to-an-msdeploy-endpoint-from-the-command-line"></a>从命令行发布到 MSDeploy 终结点

下面的示例使用由 Visual Studio 创建的 ASP.NET Core Web 应用，名为 AzureWebApp。 通过 Visual Studio 添加 Azure 应用发布配置文件。 有关如何创建配置文件的详细信息，请参阅[发布配置文件](#publish-profiles)部分。

若要使用发布配置文件部署应用，请在 Visual Studio 开发人员命令提示中执行 `msbuild` 命令。 Windows 任务栏上的“开始”菜单的“Visual Studio”文件夹中提供命令提示符。 为了便于访问，可将命令提示符添加到 Visual Studio 中的“工具”菜单中。 有关详细信息，请参阅 [Visual Studio 开发人员命令提示符](/dotnet/framework/tools/developer-command-prompt-for-vs#run-the-command-prompt-from-inside-visual-studio)。

MSBuild 使用以下命令语法：

```bash
msbuild {PATH} 
    /p:DeployOnBuild=true 
    /p:PublishProfile={PROFILE} 
    /p:Username={USERNAME} 
    /p:Password={PASSWORD}
```

* `{PATH}`：应用的项目文件的路径。
* `{PROFILE}`：发布配置文件的名称。
* `{USERNAME}`：MSDeploy 用户名。 可以在发布配置文件中找到 `{USERNAME}`。
* `{PASSWORD}`：MSDeploy 密码。 从 {PROFILE}.PublishSettings 文件中获取 `{PASSWORD}`。 可以从以下位置下载 .PublishSettings 文件：
  * 解决方案资源管理器：选择“视图” > “Cloud Explorer” 。 连接你的 Azure 订阅。 打开“应用服务”。 右键单击应用。 选择“下载发布配置文件”。
  * Azure 门户：选择 Web 应用“概述”面板上的“获取发布配置文件” 。

下面的示例使用名为“AzureWebApp - Web 部署”的发布配置文件：

```bash
msbuild "AzureWebApp.csproj" 
    /p:DeployOnBuild=true 
    /p:PublishProfile="AzureWebApp - Web Deploy" 
    /p:Username="$AzureWebApp" 
    /p:Password=".........."
```

发布配置文件还可通过 Windows 命令行界面与 .NET Core CLI 的 [dotnet msbuild](/dotnet/core/tools/dotnet-msbuild) 一起使用：

```dotnetcli
dotnet msbuild "AzureWebApp.csproj"
    /p:DeployOnBuild=true 
    /p:PublishProfile="AzureWebApp - Web Deploy" 
    /p:Username="$AzureWebApp" 
    /p:Password=".........."
```

> [!IMPORTANT]
> `dotnet msbuild` 命令是一个跨平台命令，可在 macOS 和 Linux 上编译 ASP.NET Core 应用。 但是，macOS 和 Linux 上的 MSBuild 不能将应用部署到 Azure 或其他 MSDeploy 终结点。

## <a name="set-the-environment"></a>设置环境

将 `<EnvironmentName>` 属性包含在发布配置文件（.pubxml）或项目文件中，以设置应用的[环境](xref:fundamentals/environments)：

```xml
<PropertyGroup>
  <EnvironmentName>Development</EnvironmentName>
</PropertyGroup>
```

如果需要 web.config 转换（例如，基于配置、配置文件或环境设置环境变量），请参阅 <xref:host-and-deploy/iis/transform-webconfig>。

## <a name="exclude-files"></a>排除文件

在发布 ASP.NET Core Web 应用时，包括以下资产：

* 生成工件
* 与以下 glob 模式匹配的文件夹和文件：
  * `**\*.config`（例如，web.config）
  * `**\*.json`（例如，appsettings.json）
  * `wwwroot\**`

MSBuild 支持 [glob 模式](https://gruntjs.com/configuring-tasks#globbing-patterns)。 例如，以下 `<Content>` 元素禁止在 wwwroot\content 文件夹及其子文件夹中复制文本 (.txt) 文件：

```xml
<ItemGroup>
  <Content Update="wwwroot/content/**/*.txt" CopyToPublishDirectory="Never" />
</ItemGroup>
```

可以将上面的标记添加到发布配置文件或 .csproj 文件。 添加到 .csproj 文件时，会将该规则添加到项目中的所有发布配置文件中。

以下 `<MsDeploySkipRules>` 元素排除 wwwroot\content 文件夹中的所有文件：

```xml
<ItemGroup>
  <MsDeploySkipRules Include="CustomSkipFolder">
    <ObjectName>dirPath</ObjectName>
    <AbsolutePath>wwwroot\\content</AbsolutePath>
  </MsDeploySkipRules>
</ItemGroup>
```

`<MsDeploySkipRules>` 不会从部署站点删除跳过目标。 从部署站点中删除 `<Content>` 目标文件和文件夹。 例如，假设部署的 Web 应用具有以下文件：

* *Views/Home/About1.cshtml*
* *Views/Home/About2.cshtml*
* *Views/Home/About3.cshtml*

如果添加了以下 `<MsDeploySkipRules>` 元素，则不会在部署站点上删除这些文件。

```xml
<ItemGroup>
  <MsDeploySkipRules Include="CustomSkipFile">
    <ObjectName>filePath</ObjectName>
    <AbsolutePath>Views\\Home\\About1.cshtml</AbsolutePath>
  </MsDeploySkipRules>

  <MsDeploySkipRules Include="CustomSkipFile">
    <ObjectName>filePath</ObjectName>
    <AbsolutePath>Views\\Home\\About2.cshtml</AbsolutePath>
  </MsDeploySkipRules>

  <MsDeploySkipRules Include="CustomSkipFile">
    <ObjectName>filePath</ObjectName>
    <AbsolutePath>Views\\Home\\About3.cshtml</AbsolutePath>
  </MsDeploySkipRules>
</ItemGroup>
```

之前的 `<MsDeploySkipRules>` 元素阻止部署跳过的文件。 一旦部署这些文件就不会删除它们。

以下 `<Content>` 元素将在部署站点中删除目标文件：

```xml
<ItemGroup>
  <Content Update="Views/Home/About?.cshtml" CopyToPublishDirectory="Never" />
</ItemGroup>
```

如果将命令行部署和前面的 `<Content>` 元素一起使用，则将生成以下输出的变体：

```console
MSDeployPublish:
  Starting Web deployment task from source: manifest(C:\Webs\Web1\obj\Release\{TARGET FRAMEWORK MONIKER}\PubTmp\Web1.SourceManifest.
  xml) to Destination: auto().
  Deleting file (Web11112\Views\Home\About1.cshtml).
  Deleting file (Web11112\Views\Home\About2.cshtml).
  Deleting file (Web11112\Views\Home\About3.cshtml).
  Updating file (Web11112\web.config).
  Updating file (Web11112\Web1.deps.json).
  Updating file (Web11112\Web1.dll).
  Updating file (Web11112\Web1.pdb).
  Updating file (Web11112\Web1.runtimeconfig.json).
  Successfully executed Web deployment task.
  Publish Succeeded.
Done Building Project "C:\Webs\Web1\Web1.csproj" (default targets).
```

## <a name="include-files"></a>包含文件

下列各部分简要介绍了发布时用于包含文件的不同方法。 [常规文件包含](#general-file-inclusion)部分使用了 `DotNetPublishFiles` 项，后者由 [Web SDK](xref:razor-pages/web-sdk) 中的发布目标文件提供。 [选择性文件包含](#selective-file-inclusion)部分使用了 `ResolvedFileToPublish` 项，它由 [.NET Core SDK](/dotnet/core/project-sdk/msbuild-props) 中的发布目标文件提供。 Web SDK 依赖于 .NET Core SDK，因此上述任一项都可在 ASP.NET Core 项目中使用。

### <a name="general-file-inclusion"></a>常规文件包含

下例中的 `<ItemGroup>` 元素展示了将位于项目目录之外的文件夹复制到已发布的站点的文件夹中。 添加到下述标记的 `<ItemGroup>` 的所有文件都将默认包含在内。

```xml
<ItemGroup>
  <_CustomFiles Include="$(MSBuildProjectDirectory)/../images/**/*" />
  <DotNetPublishFiles Include="@(_CustomFiles)">
    <DestinationRelativePath>wwwroot/images/%(RecursiveDir)%(Filename)%(Extension)</DestinationRelativePath>
  </DotNetPublishFiles>
</ItemGroup>
```

前面的标记：

* 可以将标记添加到 .csproj 文件或发布配置文件。 如果将其添加到 .csproj 文件，它会包含在项目的每个发布配置文件中。
* 声明一个 `_CustomFiles` 项来存储与 `Include` 属性的 glob 模式匹配的文件。 模式中引用的“图像”文件夹未在项目目录中。 名为 `$(MSBuildProjectDirectory)` 的[保留属性](/visualstudio/msbuild/msbuild-reserved-and-well-known-properties)会解析为项目文件的绝对路径。
* 向 `DotNetPublishFiles` 项提供一个文件列表。 默认情况下，该项的 `<DestinationRelativePath>` 元素为空。 默认值在标记中重写，且使用 `%(RecursiveDir)` 等[常见的项元数据](/visualstudio/msbuild/msbuild-well-known-item-metadata)。 内部文本表示已发布的站点的 wwwroot/images 文件夹。

### <a name="selective-file-inclusion"></a>选择性文件包含

下例中突出显示的标记展示了：

* 将项目之外的文件复制到已发布的站点的 wwwroot 文件夹。 保留文件名 ReadMe2.md。
* 排除 wwwroot\Content 文件夹。
* 排除 Views\Home\About2.cshtml。

[!code-xml[](visual-studio-publish-profiles/samples/Web1.pubxml?highlight=18-23)]

前述示例使用 `ResolvedFileToPublish` 项，其默认行为是始终将 `Include` 中提供的文件复制到已发布的站点中。 通过 `Never` 或 `PreserveNewest` 的内部文本包含 `<CopyToPublishDirectory>`覆盖默认行为。 例如：

```xml
<ResolvedFileToPublish Include="..\ReadMe2.md">
  <RelativePath>wwwroot\ReadMe2.md</RelativePath>
  <CopyToPublishDirectory>PreserveNewest</CopyToPublishDirectory>
</ResolvedFileToPublish>
```

有关更多部署示例，请参阅 [Web SDK README 文件](https://github.com/dotnet/sdk/tree/master/src/WebSdk)。

## <a name="run-a-target-before-or-after-publishing"></a>在发布前或发布后运行目标

内置 `BeforePublish` 和 `AfterPublish` 目标在发布目标前/后执行目标。 将以下元素添加到发布配置文件，以在发布前/后均记录控制台消息：

```xml
<Target Name="CustomActionsBeforePublish" BeforeTargets="BeforePublish">
    <Message Text="Inside BeforePublish" Importance="high" />
  </Target>
  <Target Name="CustomActionsAfterPublish" AfterTargets="AfterPublish">
    <Message Text="Inside AfterPublish" Importance="high" />
</Target>
```

## <a name="publish-to-a-server-using-an-untrusted-certificate"></a>使用不受信任的证书发布到服务器

将值为 `True` 的 `<AllowUntrustedCertificate>` 属性添加到发布配置文件：

```xml
<PropertyGroup>
  <AllowUntrustedCertificate>True</AllowUntrustedCertificate>
</PropertyGroup>
```

## <a name="the-kudu-service"></a>Kudu 服务

要查看 Azure 应用服务 Web 应用部署中的文件，请使用 [Kudu 服务](https://github.com/projectkudu/kudu/wiki/Accessing-the-kudu-service)。 将 `scm` 令牌追加到 Web 应用名称。 例如：

| URL                                    | 结果       |
| -------------------------------------- | ------------ |
| `http://mysite.azurewebsites.net/`     | Web 应用      |
| `http://mysite.scm.azurewebsites.net/` | Kudu 服务 |

选择[调试控制台](https://github.com/projectkudu/kudu/wiki/Kudu-console)菜单项来查看、编辑、删除或添加文件。

## <a name="additional-resources"></a>其他资源

* [Web 部署](https://www.iis.net/downloads/microsoft/web-deploy) (MSDeploy) 简化了 Web 应用和网站到 IIS 服务器的部署。
* [Web SDK GitHub 存储库](https://github.com/dotnet/websdk/issues)：文件问题和部署的请求功能。
* [从 Visual Studio 将 ASP.NET Web 应用发布到 Azure VM](/azure/virtual-machines/windows/publish-web-app-from-visual-studio)
* <xref:host-and-deploy/iis/transform-webconfig>
