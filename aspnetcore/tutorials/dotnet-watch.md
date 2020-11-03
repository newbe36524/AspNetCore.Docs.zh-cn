---
title: 使用文件观察程序开发 ASP.NET Core 应用
author: rick-anderson
description: 本教程演示如何在 ASP.NET Core 应用中安装和使用 .NET Core CLI 的文件观察程序 (dotnet watch) 工具。
ms.author: riande
ms.date: 05/31/2018
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
uid: tutorials/dotnet-watch
ms.openlocfilehash: 3569e9440b8e431ec0e5357e548af2e3783481ac
ms.sourcegitcommit: 422e02bad384775bfe19a90910737340ad106c5b
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90083448"
---
# <a name="develop-aspnet-core-apps-using-a-file-watcher"></a>使用文件观察程序开发 ASP.NET Core 应用

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Victor Hurdugaci](https://twitter.com/victorhurdugaci)

`dotnet watch` 是一种在源文件更改时运行 [.NET Core CLI](/dotnet/core/tools) 命令的工具。 例如，文件更改可能触发编译、测试执行或部署。

本教程使用一个现有 Web API 和两个终结点：分别返回两个数的总和以及乘积。 乘积的方法有一个 bug，本教程将会对其进行修复。

下载[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/dotnet-watch/sample)。 它包含两个项目：WebApp (ASP.NET Core Web API) 和 WebAppTests（用于 Web API 的单元测试）。

在命令行界面中，导航到 WebApp 文件夹。 运行下面的命令：

```dotnetcli
dotnet run
```

> [!NOTE]
> 可使用 `dotnet run --project <PROJECT>` 来指定要运行的项目。 例如，从示例应用的根路径运行 `dotnet run --project WebApp` 还会运行 WebApp 项目。

控制台输出会显示如下类似的消息（表示应用正在运行且正在等待请求）：

```console
$ dotnet run
Hosting environment: Development
Content root path: C:/Docs/aspnetcore/tutorials/dotnet-watch/sample/WebApp
Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

在 Web 浏览器中，导航到 `http://localhost:<port number>/api/math/sum?a=4&b=5`。 应该会显示结果 `9`。

导航到产品 API (`http://localhost:<port number>/api/math/product?a=4&b=5`)。 它会返回 `9`，而不是所预期的 `20`。 本教程的后面部分将解决该问题。

::: moniker range="<= aspnetcore-2.0"

## <a name="add-dotnet-watch-to-a-project"></a>将 `dotnet watch` 添加到项目

`dotnet watch` 文件观察程序工具随 2.1.300 版本的 .NET Core SDK 一同提供。 使用早期版本的 .NET Core SDK 时，需要执行以下步骤。

1. 将 `Microsoft.DotNet.Watcher.Tools` 包引用添加到 .csproj 文件：

    ```xml
    <ItemGroup>
        <DotNetCliToolReference Include="Microsoft.DotNet.Watcher.Tools" Version="2.0.0" />
    </ItemGroup>
    ```

1. 运行以下命令，安装 `Microsoft.DotNet.Watcher.Tools` 包：

    ```dotnetcli
    dotnet restore
    ```

::: moniker-end

## <a name="run-net-core-cli-commands-using-dotnet-watch"></a>使用 `dotnet watch` 运行 .NET Core CLI 命令

`dotnet watch` 可用于运行任何 [.NET Core CLI 命令](/dotnet/core/tools#cli-commands) 例如：

| 命令 | 带 watch 的命令 |
| ---- | ----- |
| dotnet run | dotnet watch run |
| dotnet run -f netcoreapp3.1 | dotnet watch run -f netcoreapp3.1 |
| dotnet run -f netcoreapp3.1 -- --arg1 | dotnet watch run -f netcoreapp3.1 -- --arg1 |
| dotnet test | dotnet watch test |

运行“WebApp”文件夹中的 `dotnet watch run`。 控制台输出指示 `watch` 已启动。

::: moniker range=">= aspnetcore-5.0"
在 Web 应用上运行 `dotnet watch run` 将启动一个浏览器，一旦准备就绪，该浏览器将导航到应用的 URL。 `dotnet watch` 通过读取应用的控制台输出并等待 <xref:Microsoft.AspNetCore.WebHost> 显示的就绪消息来完成此操作。

当 `dotnet watch` 检测到监视文件的更改时刷新浏览器。 为此，watch 命令向应用注入一个中间件，用于修改应用创建的 HTML 响应。 中间件将 JavaScript 脚本块添加到页面，使 `dotnet watch` 可以指示浏览器进行刷新。 目前，对所有监视的文件（包括静态内容，如 .html 和 .css）的更改都会导致重建应用 。

`dotnet watch`:

* 默认情况下，仅监视影响生成的文件。
* 任何（通过配置）额外监视的文件仍然会导致生成发生。

有关配置的详细信息，请参阅本文档中的 [dotnet-watch 配置](#dotnet-watch-configuration)

::: moniker-end

> [!NOTE]
> 可使用 `dotnet watch --project <PROJECT>` 来指定要监视的项目。 例如，从示例应用的根路径运行 `dotnet watch --project WebApp run` 还会运行并监视 WebApp 项目。

## <a name="make-changes-with-dotnet-watch"></a>使用 `dotnet watch` 执行更改

确保 `dotnet watch` 正在运行。

修复 MathController 的 `Product` 方法中的 bug，使其返回乘积而非总和。

```csharp
public static int Product(int a, int b)
{
    return a * b;
}
```

保存该文件。 控制台输出指示 `dotnet watch` 已检测到文件更改并已重启应用。

验证 `http://localhost:<port number>/api/math/product?a=4&b=5` 是否返回正确结果。

## <a name="run-tests-using-dotnet-watch"></a>使用 `dotnet watch` 运行测试

1. 将 MathController.cs 的 `Product` 方法改回返回总和。 保存该文件。
1. 在命令外壳中，导航到“WebAppTests”文件夹。
1. 运行 [dotnet restore](/dotnet/core/tools/dotnet-restore)。
1. 运行 `dotnet watch test`。 其输出指示测试失败且观察程序正在等待文件更改：

     ```console
     Total tests: 2. Passed: 1. Failed: 1. Skipped: 0.
     Test Run Failed.
     ```

1. 修复 `Product` 方法代码，使其返回乘积。 保存该文件。

`dotnet watch` 检测到文件更改并重新运行测试。 控制台输出指示测试通过。

## <a name="customize-files-list-to-watch"></a>自定义要监视的文件列表

默认情况下，`dotnet-watch` 跟踪与以下 glob 模式匹配的所有文件：

* `**/*.cs`
* `*.csproj`
* `**/*.resx`

通过编辑 .csproj 文件，可向监视列表添加更多项。 可以单独指定项或使用 glob 模式指定。

```xml
<ItemGroup>
    <!-- extends watching group to include *.js files -->
    <Watch Include="**\*.js" Exclude="node_modules\**\*;**\*.js.map;obj\**\*;bin\**\*" />
</ItemGroup>
```

## <a name="opt-out-of-files-to-be-watched"></a>从要监视的文件中排除

`dotnet-watch` 可配置为忽略其默认设置。 要忽略特定文件，请在 .csproj 文件中向某项的定义中添加 `Watch="false"` 特性：

```xml
<ItemGroup>
    <!-- exclude Generated.cs from dotnet-watch -->
    <Compile Include="Generated.cs" Watch="false" />

    <!-- exclude Strings.resx from dotnet-watch -->
    <EmbeddedResource Include="Strings.resx" Watch="false" />

    <!-- exclude changes in this referenced project -->
    <ProjectReference Include="..\ClassLibrary1\ClassLibrary1.csproj" Watch="false" />
</ItemGroup>
```

## <a name="custom-watch-projects"></a>自定义监视项目

`dotnet-watch` 不仅限于 C# 项目。 可以创建自定义监视项目来处理不同的方案。 假设项目布局如下：

* **test/**
  * *UnitTests/UnitTests.csproj*
  * *IntegrationTests/IntegrationTests.csproj*

如果目标是监视这两个项目，请创建配置为监视这两个项目的自定义项目文件：

```xml
<Project>
    <ItemGroup>
        <TestProjects Include="**\*.csproj" />
        <Watch Include="**\*.cs" />
    </ItemGroup>

    <Target Name="Test">
        <MSBuild Targets="VSTest" Projects="@(TestProjects)" />
    </Target>

    <Import Project="$(MSBuildExtensionsPath)\Microsoft.Common.targets" />
</Project>
```

要对这两个项目启动文件监视，请更改为 test 文件夹。 请执行以下命令：

```dotnetcli
dotnet watch msbuild /t:Test
```

当任一测试项目中的任何文件发生更改时，将会执行 VSTest。

## <a name="dotnet-watch-configuration"></a>dotnet-watch 配置

一些配置选项可以通过环境变量传递给 `dotnet watch`。 可用变量有：

| 设置  | 说明 |
| ------------- | ------------- |
| `DOTNET_USE_POLLING_FILE_WATCHER`                | 如果设置为“1”或“true”，则 `dotnet watch` 使用轮询文件观察程序，而不是 CoreFx 的 `FileSystemWatcher`。 在监视网络共享或 Docker 装入卷上的文件时使用。                       |
| `DOTNET_WATCH_SUPPRESS_MSBUILD_INCREMENTALISM`   | 默认情况下，`dotnet watch` 通过避免某些操作（如在每次文件更改时运行还原或重新评估监视的文件集）来优化生成。 如果设置为“1”或“true”，则禁用这些优化。 |
| `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER`   | `dotnet watch run` 尝试为在 launchSettings.json 中配置了 `launchBrowser` 的 Web 应用启动浏览器。 如果设置为“1”或“true”，则抑制此行为。 |
| `DOTNET_WATCH_SUPPRESS_BROWSER_REFRESH`   | `dotnet watch run` 检测到文件更改时尝试刷新浏览器。 如果设置为“1”或“true”，则抑制此行为。 如果设置了 `DOTNET_WATCH_SUPPRESS_LAUNCH_BROWSER`，则也会抑制此行为。 |

## <a name="dotnet-watch-in-github"></a>GitHub 中的 `dotnet-watch`

`dotnet-watch` 是 GitHub [dotnet/AspNetCore 存储库](https://github.com/dotnet/AspNetCore/tree/master/src/Tools/dotnet-watch)的一部分。
