---
title: ASP.NET Core Razor 组件类库
author: guardrex
description: 了解如何从外部组件库将组件包含在 Blazor 应用中。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/27/2020
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
uid: blazor/components/class-libraries
ms.openlocfilehash: afd1bfffae11520a5d9abccc1d2ee4cf3a46a4bf
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722457"
---
# <a name="aspnet-core-no-locrazor-components-class-libraries"></a>ASP.NET Core Razor 组件类库

作者：[Simon Timms](https://github.com/stimms)

组件可在 [Razor 类库 (RCL)](xref:razor-pages/ui-class) 中跨项目共享。 Razor 组件类库可以包含在以下各项中：

* 解决方案中的另一个项目。
* NuGet 程序包。
* 引用的 .NET 库。

正如组件是常规的 .NET 类型一样，RCL 提供的组件也是普通的 .NET 程序集。

## <a name="create-an-rcl"></a>创建 RCL

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

1. 创建新项目。
1. 选择“Razor 类库”。 选择“下一步”。
1. 在“创建新的 Razor 类库”对话框中，选择“创建” 。
1. 在“项目名称”字段提供项目名称，或接受默认项目名称。 本主题中的示例使用项目名称 `ComponentLibrary`。 选择“创建”。
1. 将 RCL 添加到一个解决方案：
   1. 右键单击该解决方案。 选择“添加” > “现有项目” 。
   1. 导航到 RCL 的项目文件。
   1. 选择 RCL 的项目文件 (`.csproj`)。
1. 从应用中添加 RCL 的引用：
   1. 右键单击该应用项目。 选择“添加” > “引用” 。
   1. 选择 RCL 项目。 选择“确定”。

> [!NOTE]
> 如果在从模板生成 RCL 时选中了“支持页面和视图”复选框，还请通过以下内容，将 `_Imports.razor` 文件添加到所生成项目的根目录中，以启用 Razor 组件创作：
>
> ```razor
> @using Microsoft.AspNetCore.Components.Web
> ```
>
> 手动将该文件添加到所生成项目的根目录中。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

1. 在命令行界面中通过 [`dotnet new`](/dotnet/core/tools/dotnet-new) 命令使用 Razor 类库模板 (`razorclasslib`)。 下面的示例创建了名为 `ComponentLibrary` 的 RCL。 执行命令时，自动创建包含 `ComponentLibrary` 的文件夹：

   ```dotnetcli
   dotnet new razorclasslib -o ComponentLibrary
   ```

   > [!NOTE]
   > 如果在从模板生成 RCL 时使用 `-s|--support-pages-and-views` 开关，还请通过以下内容将 `_Imports.razor` 文件添加到所生成项目的根目录中，以启用 Razor 组件创作：
   >
   > ```razor
   > @using Microsoft.AspNetCore.Components.Web
   > ```
   >
   > 手动将该文件添加到所生成项目的根目录中。

1. 若要将库添加到现有项目中，请在命令行界面中使用 [`dotnet add reference`](/dotnet/core/tools/dotnet-add-reference) 命令。 在下面的示例中，将 RCL 添加到应用中。 使用指向库的路径从应用的项目文件夹执行以下命令：

   ```dotnetcli
   dotnet add reference {PATH TO LIBRARY}
   ```

---

## <a name="consume-a-library-component"></a>使用库组件

若要使用另一项目的库中定义的组件，请使用以下任一方法：

* 使用命名空间的完整类型名称。
* 使用 Razor 的 [`@using`](xref:mvc/views/razor#using) 指令。 单个组件可以按名称添加。

在下面的示例中，`ComponentLibrary` 是一个包含 `Component1` 组件 (`Component1.razor`) 的组件库。 `Component1` 是库创建成功后，RCL 项目模板会自动添加的示例组件。

使用 `Component1` 组件的命名空间引用它：

```razor
<h1>Hello, world!</h1>

Welcome to your new app.

<ComponentLibrary.Component1 />
```

或者，使用 [`@using`](xref:mvc/views/razor#using) 指令将库纳入范围，并在没有命名空间的情况下使用该组件：

```razor
@using ComponentLibrary

<h1>Hello, world!</h1>

Welcome to your new app.

<Component1 />
```

可以选择在顶级 `_Import.razor` 文件中包含 `@using ComponentLibrary` 指令，使库的组件可用于整个项目。 将指令添加到任何级别的 `_Import.razor` 文件，将命名空间应用于文件夹中的单个组件或一组组件。

::: moniker range=">= aspnetcore-5.0"

若要向组件提供 `Component1` 的 `my-component` CSS 类，请在 `Component1.razor` 中使用框架的 [`Link` 组件](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements)链接到库的样式表：

```razor
<div class="my-component">
    <Link href="_content/ComponentLibrary/styles.css" rel="stylesheet" />

    <p>
        This Blazor component is defined in the <strong>ComponentLibrary</strong> package.
    </p>
</div>
```

若要在应用中提供样式表，可以在应用的 `wwwroot/index.html` 文件 (Blazor WebAssembly) 或 `Pages/_Host.cshtml` 文件 (Blazor Server) 中链接到库的样式表：

```html
<head>
    ...
    <link href="_content/ComponentLibrary/styles.css" rel="stylesheet" />
</head>
```

在子组件中使用 `Link` 组件时，只要呈现 `Link` 组件的子组件，链接的资源就可用于父组件的任何其他子组件。 在子组件中使用 `Link` 组件，与在 `wwwroot/index.html` 或 `Pages/_Host.cshtml` 中放置一个 `<link>` HTML 标记之间的区别是，框架组件已呈现的 HTML 标记：

* 可以根据应用程序状态进行修改。 不能根据应用程序状态修改硬编码 `<link>` HTML 标记。
* 将在不再呈现父组件的情况下从 HTML `<head>` 中被删除。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

若要提供 `Component1` 的 `my-component` CSS 类，请在应用的 `wwwroot/index.html` 文件 (Blazor WebAssembly) 或 `Pages/_Host.cshtml` 文件 (Blazor Server) 中链接到库的样式表：

```html
<head>
    ...
    <link href="_content/ComponentLibrary/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

## <a name="create-a-no-locrazor-components-class-library-with-static-assets"></a>创建包括静态资源的 Razor 组件类库

RCL 可以包括静态资产。 静态资产可用于任何使用该库的应用。 有关详细信息，请参阅 <xref:razor-pages/ui-class#create-an-rcl-with-static-assets>。

## <a name="supply-components-and-static-assets-to-multiple-hosted-no-locblazor-apps"></a>向多个托管的 Blazor 应用提供组件和静态资产

有关详细信息，请参阅 <xref:blazor/host-and-deploy/webassembly#static-assets-and-class-libraries>。

::: moniker range=">= aspnetcore-5.0"

## <a name="browser-compatibility-analyzer-for-no-locblazor-webassembly"></a>Blazor WebAssembly 的浏览器兼容性分析器

Blazor WebAssembly 应用面向整个 .NET API 外围应用，但由于浏览器沙盒约束，并非所有 .NET API 在 WebAssembly 上都受支持。 在 WebAssembly 上运行时，不支持的 API 将引发 <xref:System.PlatformNotSupportedException>。 当应用使用应用目标平台不支持的 API 时，平台兼容性分析器会向开发人员发出警告。 对于 Blazor WebAssembly 应用，这意味着需要检查浏览器是否支持这些 API。 为兼容性分析器注释 .NET Framework API 是一个持续的过程，因此并不是所有的 .NET Framework API 当前都已进行注释。

Blazor WebAssembly 和 Razor 类库项目自动启用浏览器兼容性检查，方法是使用 `SupportedPlatform` MSBuild 项将 `browser` 添加为支持的平台。 库开发人员可以手动将 `SupportedPlatform` 项添加到库的项目文件以启用该功能：

```xml
<ItemGroup>
  <SupportedPlatform Include="browser" />
</ItemGroup>
```

编写库时，通过将 `browser` 指定为 <xref:System.Runtime.Versioning.UnsupportedOSPlatformAttribute> 来指示浏览器不支持特定的 API：

```csharp
[UnsupportedOSPlatform("browser")]
private static string GetLoggingDirectory()
{
    ...
}
```

有关详细信息，请参阅[在特定平台（dotnet/designs GitHub 存储库）上将 API 注释为不受支持](https://github.com/dotnet/designs/blob/main/accepted/2020/platform-exclusion/platform-exclusion.md#build-configuration-for-platforms)。

## <a name="no-locblazor-javascript-isolation-and-object-references"></a>Blazor JavaScript 隔离和对象引用

Blazor 在标准 [JavaScript 模块](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)中启用 JavaScript 隔离。 JavaScript 隔离具有以下优势：

* 导入的 JavaScript 不再污染全局命名空间。
* 库和组件的使用者不需要手动导入相关的 JavaScript。

有关详细信息，请参阅 <xref:blazor/call-javascript-from-dotnet#blazor-javascript-isolation-and-object-references>。

::: moniker-end

## <a name="build-pack-and-ship-to-nuget"></a>生成并打包库，再将其传送到 NuGet

由于组件库是标准的 .NET 库，因此将它们打包并传送到 NuGet 与将任何库打包并传送到 NuGet 没有什么区别。 在命令行界面中使用 [`dotnet pack`](/dotnet/core/tools/dotnet-pack) 命令，执行打包操作：

```dotnetcli
dotnet pack
```

在命令行界面中使用 [`dotnet nuget push`](/dotnet/core/tools/dotnet-nuget-push) 命令，将包上传到 NuGet。

## <a name="additional-resources"></a>其他资源

::: moniker range=">= aspnetcore-5.0"

* <xref:razor-pages/ui-class>
* [向库添加 XML 中间语言 (IL) 裁边器配置文件](xref:blazor/host-and-deploy/configure-trimmer)

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* <xref:razor-pages/ui-class>
* [向库添加 XML 中间语言 (IL) 链接器配置文件](xref:blazor/host-and-deploy/configure-linker#add-an-xml-linker-configuration-file-to-a-library)

::: moniker-end
