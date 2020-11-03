---
title: 使用类库中的 ASP.NET Core API
author: scottaddie
description: 了解如何使用类库中的 ASP.NET Core API。
ms.author: scaddie
ms.custom: mvc
ms.date: 12/16/2019
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
uid: fundamentals/target-aspnetcore
ms.openlocfilehash: 571e6c66f60bbc09b902ff9064d2fb1c18c433dc
ms.sourcegitcommit: d5ecad1103306fac8d5468128d3e24e529f1472c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/26/2020
ms.locfileid: "88630050"
---
# <a name="use-aspnet-core-apis-in-a-class-library"></a>使用类库中的 ASP.NET Core API

作者：[Scott Addie](https://github.com/scottaddie)

此文档提供了关于如何使用类库中的 ASP.NET Core API 的指南。 若要查看所有其他的库指南，请参阅[开放源代码库指南](/dotnet/standard/library-guidance/)。

## <a name="determine-which-aspnet-core-versions-to-support"></a>确定要支持哪些 ASP.NET Core 版本

ASP.NET Core 遵从 [.NET Core 支持策略](https://dotnet.microsoft.com/platform/support/policy/dotnet-core)。 确定库要支持哪些 ASP.NET Core 版本时，请参阅支持策略。 库应符合以下条件：

* 努力支持列为“长期支持”(LTS) 类别的所有 ASP.NET Core 版本。
* 无需支持列为“生命周期结束”(EOL) 类别的 ASP.NET Core 版本。

由于 ASP.NET Core 预览版已推出，因此已在 [aspnet/Announcements](https://github.com/aspnet/Announcements/issues) GitHub 存储库中发布中断性变更。 开发框架功能时，可执行库兼容性测试。

## <a name="use-the-aspnet-core-shared-framework"></a>使用 ASP.NET Core 共享框架

随着 .NET Core 3.0 发布，许多 ASP.NET Core 程序集不再作为包发布到 NuGet。 而是改为将这些程序集包含在通过 .NET Core SDK 和运行时安装程序安装的 `Microsoft.AspNetCore.App` 共享框架中。 若要查看不再发布的包列表，请参阅[删除过时的包引用](xref:migration/22-to-30#remove-obsolete-package-references)。

自 .NET Core 3.0 起，使用 `Microsoft.NET.Sdk.Web` MSBuild SDK 的项目隐式引用此共享框架。 使用 `Microsoft.NET.Sdk` 或 `Microsoft.NET.Sdk.Razor` SDK 的项目必须引用 ASP.NET Core，才能使用共享框架中的 ASP.NET Core API。

若要引用 ASP.NET Core，请将以下 `<FrameworkReference>` 元素添加到项目文件：

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj?highlight=8)]

仅面向 .NET Core 3.x 的项目支持使用此方式引用 ASP.NET Core。

## <a name="include-no-locblazor-extensibility"></a>包括 Blazor 扩展性

Blazor 支持 WebAssembly (WASM) 和服务器[托管模型](xref:blazor/hosting-models)。 除非出于特定原因无法实现支持，否则 [Razor 组件](xref:blazor/components/index)库应同时支持这两种托管模型。 Razor 组件库必须使用 [Microsoft.NET.Sdk.RazorSDK](xref:razor-pages/sdk)。

### <a name="support-both-hosting-models"></a>同时支持两种托管模型

请针对自己的编辑器使用以下说明，以同时支持 [Blazor Server](xref:blazor/hosting-models#blazor-server) 和 [Blazor WASM](xref:blazor/hosting-models#blazor-webassembly) 项目的 Razor 组件消耗。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

使用 Razor 类库项目模板。 应取消选中此模板的“支持页和视图”复选框。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

在“集成终端”中运行以下命令：

```dotnetcli
dotnet new razorclasslib
```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

使用 Razor 类库项目模板。

---

模板生成的项目执行以下操作：

* 面向 .NET Standard 2.0。
* 将 `RazorLangVersion` 属性设置为 `3.0`。 .NET Core 3.x 的默认值是 `3.0`。
* 添加以下包引用：
  * [Microsoft.AspNetCore.Components](https://www.nuget.org/packages/Microsoft.AspNetCore.Components)
  * [Microsoft.AspNetCore.Components.Web](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.Web)

例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-components-library.csproj)]

### <a name="support-a-specific-hosting-model"></a>支持特定托管模型

支持单个 Blazor 托管模型并不常见。 例如，通过完成以下操作来仅支持 [Blazor Server](xref:blazor/hosting-models#blazor-server) 项目的 Razor 组件消耗：

* 面向 .NET Core 3.x。
* 添加针对共享框架的 `<FrameworkReference>` 元素。

例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-components-library.csproj)]

有关包含 Razor 组件的库的详细信息，请参阅 [ASP.NET Core Razor 组件类库](xref:blazor/components/class-libraries)。

## <a name="include-mvc-extensibility"></a>包括 MVC 扩展性

此部分概述了针对包括以下内容的库的建议：

* [Razor 视图或 Razor 页面](#razor-views-or-razor-pages)
* [标记帮助程序](#tag-helpers)
* [视图组件](#view-components)

此部分未探讨用于支持多个 MVC 版本的多目标。 若要查看关于支持多个 ASP.NET Core 版本的指南，请参阅[支持多个 ASP.NET Core 版本](#support-multiple-aspnet-core-versions)。

### <a name="no-locrazor-views-or-no-locrazor-pages"></a>Razor 视图或 Razor 页面

包括 [Razor 视图](xref:mvc/views/overview)或 [Razor 页面](xref:razor-pages/index)的项目必须使用 [Microsoft.NET.Sdk.RazorSDK](xref:razor-pages/sdk)。

若项目面向 .NET Core 3.x，它必须满足以下要求：

* `AddRazorSupportForMvc` MSBuild 属性设置为 `true`。
* 具有针对共享框架的 `<FrameworkReference>` 元素。

Razor 类库项目模板符合针对面向 .NET Core 3.x 的项目的上述要求。 请针对自己的编辑器使用以下说明。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

使用 Razor 类库项目模板。 应选中此模板的“支持页和视图”复选框。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

在“集成终端”中运行以下命令：

```dotnetcli
dotnet new razorclasslib -s
```

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

此时无任何项目模板支持。

---

例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-razor-views-pages-library.csproj)]

若项目改为面向 .NET Standard，则需要 [Microsoft.AspNetCore.Mvc](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc) 包引用。 `Microsoft.AspNetCore.Mvc` 包移到了 ASP.NET Core 3.0 的共享框架中，因此不再发布。 例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-razor-views-pages-library.csproj?highlight=8)]

### <a name="tag-helpers"></a>标记帮助程序

包含[标记帮助器](xref:mvc/views/tag-helpers/intro)的项目应使用 `Microsoft.NET.Sdk` SDK。 若面向 .NET Core 3.x，则添加针对共享框架的 `<FrameworkReference>` 元素。 例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

若面向 .NET Standard（以支持 ASP.NET Core 3.x 之前的版本），则添加对 [Microsoft.AspNetCore.Mvc.Razor](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor) 的包引用。 `Microsoft.AspNetCore.Mvc.Razor` 包移到了共享框架，因此不再发布。 例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-tag-helpers-library.csproj)]

### <a name="view-components"></a>视图组件

包含[视图组件](xref:mvc/views/view-components)的项目应使用 `Microsoft.NET.Sdk` SDK。 若面向 .NET Core 3.x，则添加针对共享框架的 `<FrameworkReference>` 元素。 例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netcoreapp3.0-basic-library.csproj)]

若面向 .NET Standard（以支持 ASP.NET Core 3.x 之前的版本），则添加 [Microsoft.AspNetCore.Mvc.ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures) 的包引用。 `Microsoft.AspNetCore.Mvc.ViewFeatures` 包移到了共享框架，因此不再发布。 例如：

[!code-xml[](target-aspnetcore/samples/single-tfm/netstandard2.0-view-components-library.csproj)]

## <a name="support-multiple-aspnet-core-versions"></a>支持多个 ASP.NET Core 版本

创作支持多个 ASP.NET Core 变体的库需要多目标。 以下列情况为例：标记帮助器库必须支持以下 ASP.NET Core 变体：

* 面向 .NET Framework 4.6.1 的 ASP.NET Core 2.1
* 面向 .NET Core 2.x 的 ASP.NET Core 2.x
* 面向 .NET Core 3.x 的 ASP.NET Core 3.x

以下项目文件通过 `TargetFrameworks` 属性支持这些变体：

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

上面的项目文件将完成以下操作：

* 为所有使用者添加 `Markdig` 包。
* [Microsoft.AspNetCore.Mvc.Razor](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.Razor) 的引用 已针对面向 .NET Framework 4.6.1 或更高版本或者 .NET Core 2.x 的使用者添加。 由于向后兼容性，此包的版本 2.1.0 适用于 ASP.NET Core 2.2。
* 为面向 .NET Core 3.x 的使用者引用共享框架。 共享框架中包括 `Microsoft.AspNetCore.Mvc.Razor` 包。

或者，可以面向 .NET Standard 2.0，而不面向 .NET Core 2.1 和 .NET Framework 4.6.1：

[!code-xml[](target-aspnetcore/samples/multi-tfm/alternative-tag-helpers-library.csproj?highlight=4)]

对于上面的项目文件，有以下注意事项：

* 由于库只包含标记帮助器，面向 ASP.NET Core 运行的特定平台（.NET Core 和 .NET Framework）更为简单。 其他兼容 .NET Standard 2.0 的目标框架（如 Unity、UWP 和 Xamarin）无法使用标记帮助器。
* 在 .NET Framework 中使用 .NET Standard 2.0 存在一些问题，这些在 .NET Framework 4.7.2 中已得到解决。 通过面向 .NET Framework 4.6.1，可以改进使用 .NET Framework 4.6.1 到 4.7.1 的使用者的体验。

如果库需要调用平台特定的 API，则面向特定 .NET 实现，而不面向 .NET Standard。 有关详细信息，请参阅[多目标](/dotnet/standard/library-guidance/cross-platform-targeting#multi-targeting)。

## <a name="use-an-api-that-hasnt-changed"></a>使用未变更的 API

以下面的情况为例：你要将中间件库从 .NET Core 2.2 升级到 3.0。 库正在使用的 ASP.NET Core 中间件 API 尚未从 ASP.NET Core 2.2 变更到 3.0。 若要继续让 .NET Core 3.0 支持此中间件库，请执行以下步骤：

* 按照[标准库指南](/dotnet/standard/library-guidance/)操作。
* 若共享框架中不存在相应的程序集，则添加针对每个 API 的 NuGet 包的包引用。

## <a name="use-an-api-that-changed"></a>使用已变更的 API

以下面的情况为例：你要将库从 .NET Core 2.2 升级到 .NET Core 3.0。 库正在使用的 ASP.NET Core API 包含 ASP.NET Core 3.0 的[中断性变更](/dotnet/core/compatibility/breaking-changes)。 请考虑此库是否可以重写为不使用所有版本中已损坏的 API。

若可以重写此库，则进行重写，并通过包引用继续面向早期版本的目标框架（例如 .NET Standard 2.0 或 .NET Framework 4.6.1）。

若无法重写此库，则执行以下步骤：

* 添加针对 .NET Core 3.0 的目标。
* 添加针对共享框架的 `<FrameworkReference>` 元素。
* 结合使用 [#if 预处理器指令](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if)和对应的目标框架符号，以进行有条件的代码编译。

例如，自 ASP.NET Core 3.0 起默认不可对 HTTP 请求和响应流进行同步读写。 默认情况下，ASP.NET Core 2.2 支持同步行为。 以一个中间件库为例，在该库中，发生 I/O 时应可进行同步读写。 此库应将用于启用同步功能的代码括在相应的预处理器指令中。 例如：

[!code-csharp[](target-aspnetcore/samples/middleware.cs?highlight=9-24)]

## <a name="use-an-api-introduced-in-30"></a>使用 3.0 引入的 API

假设你想要使用 ASP.NET Core 3.0 引入的 ASP.NET Core API。 请考虑下列问题：

1. 此库在功能上是否需要此新 API？
1. 库是否可以通过其他方式实现此功能？

若此库在功能上需要此 API，并且没有实现它的下层方法：

* 仅面向 .NET Core 3.x。
* 添加针对共享框架的 `<FrameworkReference>` 元素。

若此库可以通过其他方式实现此功能：

* 将 .NET Core 3.x 添加为目标框架。
* 添加针对共享框架的 `<FrameworkReference>` 元素。
* 结合使用 [#if 预处理器指令](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if)和对应的目标框架符号，以进行有条件的代码编译。

例如，以下标记帮助器使用 ASP.NET Core 3.0 引入的 <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment> 接口。 面向 .NET Core 3.0 的使用者执行 `NETCOREAPP3_0` 目标框架符号定义的代码路径。 对于 .NET Core 2.1 和 .NET Framework 4.6.1 使用者，标记帮助器的构造函数参数类型更改为 <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment>。 此更改很有必要，因为 ASP.NET Core 3.0 将 `IHostingEnvironment` 标记为过时，并推荐将 `IWebHostEnvironment` 作为替代项。

```csharp
[HtmlTargetElement("script", Attributes = "asp-inline")]
public class ScriptInliningTagHelper : TagHelper
{
    private readonly IFileProvider _wwwroot;

#if NETCOREAPP3_0
    public ScriptInliningTagHelper(IWebHostEnvironment env)
#else
    public ScriptInliningTagHelper(IHostingEnvironment env)
#endif
    {
        _wwwroot = env.WebRootFileProvider;
    }

    // code omitted for brevity
}
```

以下多目标项目文件支持此标记帮助器方案：

[!code-xml[](target-aspnetcore/samples/multi-tfm/recommended-tag-helpers-library.csproj)]

## <a name="use-an-api-removed-from-the-shared-framework"></a>使用共享框架已删除的 API

若要使用共享框架已删除的 ASP.NET Core 程序集，请添加相应的包引用。 若要查看 ASP.NET Core 3.0 的共享框架已删除的包列表，请参阅[删除过时的包引用](xref:migration/22-to-30#remove-obsolete-package-references)。

例如，添加 Web API 客户端：

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNet.WebApi.Client" Version="5.2.7" />
  </ItemGroup>

</Project>
```

## <a name="additional-resources"></a>其他资源

* <xref:razor-pages/ui-class>
* <xref:blazor/components/class-libraries>
* [.NET 实现支持](/dotnet/standard/net-standard#net-implementation-support)
* [.NET 支持策略](https://dotnet.microsoft.com/platform/support/policy)
