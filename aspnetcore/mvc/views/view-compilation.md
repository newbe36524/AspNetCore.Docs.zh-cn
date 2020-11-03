---
title: Razor ASP.NET Core 中的文件编译
author: rick-anderson
description: 了解文件编译如何 Razor 出现在 ASP.NET Core 的应用程序中。
ms.author: riande
ms.custom: mvc
ms.date: 04/14/2020
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
uid: mvc/views/view-compilation
ms.openlocfilehash: 77ca96b329136ee044ab6fc5f6b5ebb5b67fe64c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059073"
---
# <a name="no-locrazor-file-compilation-in-aspnet-core"></a>Razor ASP.NET Core 中的文件编译

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.1"

Razor使用 [ Razor SDK](xref:razor-pages/sdk)的生成和发布时间都将编译带有 *..* 扩展名的文件。 可以选择通过配置项目来启用运行时编译。

## <a name="no-locrazor-compilation"></a>Razor 汇编

Razor默认情况下，SDK 会启用文件的生成时间和发布时编译 Razor 。 启用后，运行时编译会补充生成时编译，允许在 Razor 编辑文件时对其进行更新。

## <a name="enable-runtime-compilation-at-project-creation"></a>在创建项目时启用运行时编译

Razor页面和 MVC 项目模板包括一个选项，用于在创建项目时启用运行时编译。 ASP.NET Core 3.1 及更高版本中支持此选项。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 " **新建 ASP.NET Core web 应用程序** " 对话框中：

1. 选择 " **Web 应用程序** " 或 " **web 应用程序 (模型-视图-控制器)** " 项目模板。
1. 选中 " **启用 Razor 运行时编译** " 复选框。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

使用 `-rrc` 或 `--razor-runtime-compilation` 模板选项。 例如，以下命令将创建一个启用了 Razor 运行时编译的新页项目：

```dotnetcli
dotnet new webapp --razor-runtime-compilation
```

---

## <a name="enable-runtime-compilation-in-an-existing-project"></a>在现有项目中启用运行时编译

若要为现有项目中的所有环境启用运行时编译：

1. 安装 [AspNetCore Razor .。RuntimeCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation/) NuGet 包。
1. 更新项目的 `Startup.ConfigureServices` 方法以包含对 <xref:Microsoft.Extensions.DependencyInjection.RazorRuntimeCompilationMvcBuilderExtensions.AddRazorRuntimeCompilation*> 的调用。 例如： 。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddRazorPages()
            .AddRazorRuntimeCompilation();

        // code omitted for brevity
    }
    ```

## <a name="conditionally-enable-runtime-compilation-in-an-existing-project"></a>在现有项目中有条件地启用运行时编译

启用运行时编译时可使其仅用于本地开发。 以这种方式有条件地启用可确保已发布的输出：

* 使用编译视图。
* 不会在生产环境中启用文件观察程序。

仅在开发环境中启用运行时编译：

1. 安装 [AspNetCore Razor .。RuntimeCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation/) NuGet 包。
1. 修改 `environmentVariables` *launchSettings.js上* 的启动配置文件部分：
    * 验证 `ASPNETCORE_ENVIRONMENT` 是否设置为 `"Development"` 。
    * 将 `ASPNETCORE_HOSTINGSTARTUPASSEMBLIES` 设置为 `"Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation"`。

在下面的示例中，在 `IIS Express` 和启动配置文件的开发环境中启用了运行时编译 `RazorPagesApp` ：

[!code-json[](~/mvc/views/view-compilation/samples/3.1/launchSettings.json?highlight=15-16,24-25)]

项目的类中不需要更改代码 `Startup` 。 在运行时，ASP.NET Core 在中搜索 [程序集级别的 HostingStartup 特性](xref:fundamentals/configuration/platform-specific-configuration#hostingstartup-attribute) `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` 。 `HostingStartup`特性指定要执行的应用程序启动代码。 该启动代码启用运行时编译。

## <a name="enable-runtime-compilation-for-a-no-locrazor-class-library"></a>为类库启用运行时编译 Razor

请考虑以下方案 Razor ：页面项目引用类库 (名为 *MYCLASSLIB* 的 [ Razor RCL)](xref:razor-pages/ui-class) 。 RCL 包含一个 *_Layout 的 cshtml* 文件，其中的所有团队 MVC 和 Razor 页面项目都使用。 需要为 RCL 中的 *_Layout cshtml* 文件启用运行时编译。 在页面项目中进行以下更改 Razor ：

1. 使用在 [现有项目中有条件地启用运行时编译中](#conditionally-enable-runtime-compilation-in-an-existing-project)的说明启用运行时编译。
1. 配置中的运行时编译选项 `Startup.ConfigureServices` ：

    [!code-csharp[](~/mvc/views/view-compilation/samples/3.1/Startup.cs?name=snippet_ConfigureServices&highlight=5-10)]

    在前面的代码中，构造 *MyClassLib* RCL 的绝对路径。 [PHYSICALFILEPROVIDER API](xref:fundamentals/file-providers#physicalfileprovider)用于查找该绝对路径中的目录和文件。 最后，将 `PhysicalFileProvider` 实例添加到 file 提供程序集合，该集合允许访问 RCL 的文件 *.cshtml* 。

## <a name="additional-resources"></a>其他资源

* [ Razor CompileOnBuild 和 Razor CompileOnPublish](xref:razor-pages/sdk#properties)属性。
* <xref:razor-pages/index>
* <xref:mvc/views/overview>
* <xref:razor-pages/sdk>

::: moniker-end

::: moniker range="= aspnetcore-3.0"

Razor使用 [ Razor SDK](xref:razor-pages/sdk)的生成和发布时间都将编译带有 *..* 扩展名的文件。 通过配置应用程序，可以选择启用运行时编译。

## <a name="no-locrazor-compilation"></a>Razor 汇编

Razor默认情况下，SDK 会启用文件的生成时间和发布时编译 Razor 。 启用后，运行时编译会补充生成时编译，允许在 Razor 编辑文件时对其进行更新。

## <a name="runtime-compilation"></a>运行时编译

为所有环境和配置模式启用运行时编译：

1. 安装 [AspNetCore Razor .。RuntimeCompilation](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation/) NuGet 包。

1. 更新项目的 `Startup.ConfigureServices` 方法以包含对 <xref:Microsoft.Extensions.DependencyInjection.RazorRuntimeCompilationMvcBuilderExtensions.AddRazorRuntimeCompilation*> 的调用。 例如： 。

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddRazorPages()
            .AddRazorRuntimeCompilation();

        // code omitted for brevity
    }
    ```

### <a name="conditionally-enable-runtime-compilation"></a>有条件地启用运行时编译

启用运行时编译时可使其仅用于本地开发。 以这种方式有条件地启用可确保已发布的输出：

* 使用编译视图。
* 较小。
* 不会在生产环境中启用文件观察程序。

基于环境和配置模式启用运行时编译：

1. 按条件引用 [AspNetCore Razor 。](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation/) 基于活动值的 RuntimeCompilation 包 `Configuration` ：

    ```xml
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation" Version="3.1.0" Condition="'$(Configuration)' == 'Debug'" />
    ```

1. 更新项目的 `Startup.ConfigureServices` 方法以包含对 `AddRazorRuntimeCompilation` 的调用。 有条件地执行 `AddRazorRuntimeCompilation`，使其仅当 `ASPNETCORE_ENVIRONMENT` 变量设置为 `Development`时在调试模式下运行：

    [!code-csharp[](~/mvc/views/view-compilation/samples/3.0/Startup.cs?name=snippet)]

## <a name="additional-resources"></a>其他资源

* [ Razor CompileOnBuild 和 Razor CompileOnPublish](xref:razor-pages/sdk#properties)属性。
* <xref:razor-pages/index>
* <xref:mvc/views/overview>
* <xref:razor-pages/sdk>
* 有关演示如何在项目中执行运行时编译的示例，请参阅 [GitHub 上的运行时编译示例](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/mvc/runtimecompilation) 。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Razor调用关联的 Razor 页或 MVC 视图时，将在运行时编译文件。 Razor使用[ Razor SDK](xref:razor-pages/sdk)在生成和发布时编译文件。

## <a name="no-locrazor-compilation"></a>Razor 汇编

Razor默认情况下，SDK 会启用文件的生成和发布时编译 Razor 。 在 Razor 生成时，支持更新已更新的文件。 默认情况下，仅编译 *Views.dll* 编译的Views.dll *和不需要* 的文件或引用程序集。 Razor

> [!IMPORTANT]
> 已弃用预编译工具，并且将在 ASP.NET Core 3.0 中删除该工具。 建议迁移到[ Razor Sdk](xref:razor-pages/sdk)。
>
> RazorSDK 仅在项目文件中未设置预编译特定属性时有效。 例如，设置 *.csproj* 文件的 `MvcRazorCompileOnPublish` 属性以 `true` 禁用该 Razor SDK。

## <a name="runtime-compilation"></a>运行时编译

生成时编译由文件的运行时编译补充 Razor 。 ASP.NET Core MVC 文件的 Razor 内容发生更改时 *，将* 重新编译文件。

## <a name="additional-resources"></a>其他资源

* <xref:razor-pages/index>
* <xref:mvc/views/overview>
* <xref:razor-pages/sdk>

::: moniker-end
