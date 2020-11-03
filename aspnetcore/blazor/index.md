---
title: ASP.NET Core Blazor 简介
author: guardrex
description: 了解 ASP.NET Core Blazor，用户可用它在 ASP.NET Core 应用中使用 .NET 生成交互式客户端 Web UI。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, seoapril2019, devx-track-js
ms.date: 09/25/2020
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
uid: blazor/index
ms.openlocfilehash: ecdf4f59aca0fe71bbfcfe61a99109127c8b92df
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93055707"
---
# <a name="introduction-to-aspnet-core-no-locblazor"></a>ASP.NET Core Blazor 简介

作者：[Daniel Roth](https://github.com/danroth27) 和 [Luke Latham](https://github.com/guardrex)

*欢迎使用 Blazor！*

Blazor 是一个使用 [.NET](/dotnet/standard/tour) 生成交互式客户端 Web UI 的框架：

* 使用 [C#](/dotnet/csharp/) 代替 [JavaScript](https://www.javascript.com) 来创建信息丰富的交互式 UI。
* 共享使用 .NET 编写的服务器端和客户端应用逻辑。
* 将 UI 呈现为 HTML 和 CSS，以支持众多浏览器，其中包括移动浏览器。
* 与新式托管平台（如 [Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/index)）集成。

使用 .NET 进行客户端 Web 开发可提供以下优势：

* 使用 C# 代替 JavaScript 来编写代码。
* 利用现有的 [.NET 库](/dotnet/standard/class-libraries)生态系统。
* 在服务器和客户端之间共享应用逻辑。
* 受益于 .NET 的性能、可靠性和安全性。
* 在 Windows、Linux 和 macOS 上使用 [Visual Studio](https://visualstudio.microsoft.com) 保持高效工作。
* 以一组稳定、功能丰富且易用的通用语言、框架和工具为基础来进行生成。

## <a name="components"></a>组件

Blazor应用基于组件。 Blazor 中的组件是指 UI 元素，例如页面、对话框或数据输入窗体。

组件是内置到 [.NET 程序集](/dotnet/standard/assembly/)的 .NET C# 类，它们用于：

* 定义灵活的 UI 呈现逻辑。
* 处理用户事件。
* 可以嵌套和重用。
* 可作为 [Razor 类库](xref:razor-pages/ui-class)或 [NuGet 包](/nuget/what-is-nuget)共享和分发。

组件类通常以 [Razor](xref:mvc/views/razor) 标记页（文件扩展名为 `.razor`）的形式编写。 Blazor 中的组件有时被称为 Razor 组件。 Razor 是一种语法，用于将 HTML 标记与专为提高开发人员工作效率而设计的 C# 代码结合在一起。 借助 Razor，可使用 Visual Studio 中的 [IntelliSense](/visualstudio/ide/using-intellisense) 编程支持在同一文件中的 HTML 标记与 C# 之间切换。 Razor Pages 和 MVC 也使用 Razor。 与基于请求/响应模型生成的 Razor Pages 和 MVC 不同，组件专门用于处理客户端 UI 逻辑和构成。

Blazor 使用 UI 构成的自然 HTML 标记。 下面的 Razor 标记演示了一个组件 (`Dialog.razor`)，它显示一个对话框，并处理在用户选择按钮时发生的事件：

```razor
<div class="card" style="width:22rem">
    <div class="card-body">
        <h3 class="card-title">@Title</h3>
        <p class="card-text">@ChildContent</p>
        <button @onclick="OnYes">Yes!</button>
    </div>
</div>

@code {
    [Parameter]
    public RenderFragment ChildContent { get; set; }

    [Parameter]
    public string Title { get; set; }

    private void OnYes()
    {
        Console.WriteLine("Write to the console in C#! 'Yes' button selected.");
    }
}
```

在上述示例中，`OnYes` 是由按钮的 `onclick` 事件触发的 C# 方法。 对话框的文本 (`ChildContent`) 和标题 (`Title`) 由在其 UI 中使用此组件的下述组件提供。

使用 HTML 标记将 `Dialog` 组件嵌入到另一个组件中。 在以下示例中，`Index` 组件 (`Pages/Index.razor`) 使用前面的 `Dialog` 组件。 标记的 `Title` 属性向 `Dialog` 组件的 `Title` 属性传递标题的值。  `Dialog` 组件的文本 (`ChildContent`) 由 `<Dialog>` 元素的内容设置。 向 `Index` 组件添加 `Dialog` 组件后，[Visual Studio 中的 IntelliSense](/visualstudio/ide/using-intellisense) 可加快使用语法和参数补全进行开发的速度。

```razor
@page "/"

<h1>Hello, world!</h1>

<p>
    Welcome to your new app.
</p>

<Dialog Title="Learn More">
    Do you want to <i>learn more</i> about Blazor?
</Dialog>
```

在浏览器中访问父级 `Index` 组件时，将呈现该对话框。 当用户选择此按钮时，浏览器的开发人员工具控制台会显示由 `OnYes` 方法编写的消息：

![在 Index 组件中嵌入的浏览器内呈现 Dialog 组件。 当用户在 UI 中选择 Yes! 按钮时，浏览器开发人员工具控制台会显示由 C# 代码 编写的消息。](index/_static/dialog.png)

组件呈现为浏览器[文档对象模型 (DOM)](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction) 的内存中表现形式，它被称为“呈现树”，用于以灵活高效的方式更新 UI。

## Blazor WebAssembly

Blazor WebAssembly 是[单页应用 (SPA) 框架](/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps)，用于使用 .NET 生成交互式客户端 Web 应用。 Blazor WebAssembly 使用无插件或将代码重新编译为其他语言的开放式 Web 标准。 Blazor WebAssembly 适用于所有新式 Web 浏览器，包括移动浏览器。

通过 [WebAssembly](https://webassembly.org)（缩写为 `wasm`），可在 Web 浏览器内运行 .NET 代码。 WebAssembly 是针对快速下载和最大执行速度优化的压缩字节码格式。 WebAssembly 是开放的 Web 标准，支持用于无插件的 Web 浏览器。

WebAssembly 代码可通过 JavaScript（称为 JavaScript 互操作性，通常简称为 JavaScript 互操作或 JS 互操作）访问浏览器的完整功能  。 通过浏览器中的 WebAssembly 执行的 .NET 代码在浏览器的 JavaScript 沙盒中运行，沙盒提供的保护可防御客户端计算机上的恶意操作。

![Blazor WebAssembly 使用 WebAssembly 在浏览器中运行 .NET 代码。](index/_static/blazor-webassembly.png)

当 Blazor WebAssembly 应用生成并在浏览器中运行时：

* C# 代码文件和 Razor 文件将被编译为 .NET 程序集。
* 该程序集和 [.NET 运行时](/dotnet/framework/get-started/overview)将被下载到浏览器。
* Blazor WebAssembly 启动 .NET 运行时，并配置运行时，以为应用加载程序集。 Blazor WebAssembly 运行时使用 JavaScript 互操作来处理 DOM 操作和浏览器 API 调用。

已发布应用的大小（其有效负载大小）是应用可用性的关键性能因素。 大型应用需要相对较长的时间才能下载到浏览器，这会损害用户体验。 Blazor WebAssembly 优化有效负载大小，以缩短下载时间：

::: moniker range=">= aspnetcore-5.0"

* 在[中间语言 (IL) 裁边器](xref:blazor/host-and-deploy/configure-trimmer)发布应用时，会从应用删除未使用的代码。
* 压缩 HTTP 响应。
* .NET 运行时和程序集缓存在浏览器中。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* 在[中间语言 (IL) 链接器](xref:blazor/host-and-deploy/configure-linker)发布应用时，会从应用删除未使用的代码。
* 压缩 HTTP 响应。
* .NET 运行时和程序集缓存在浏览器中。

::: moniker-end

## Blazor Server

Blazor 将组件呈现逻辑从 UI 更新的应用方式中分离出来。 Blazor Server在 ASP.NET Core 应用中支持在服务器上托管 Razor 组件。 可通过 [SignalR](xref:signalr/introduction) 连接处理 UI 更新。

运行时会处理以下事件：

* 将 UI 事件从浏览器发送到服务器。
* 将 UI 更新应用于服务器发送回的已呈现的组件。

Blazor Server用于与浏览器通信的连接还用于处理 JavaScript 互操作调用。

![Blazor 服务器在服务器上运行 .NET 代码，并通过 SignalR 连接与客户端上的文档对象模型进行交互](index/_static/blazor-server.png)

## <a name="javascript-interop"></a>JavaScript 互操作

对于需要第三方 JavaScript 库和访问浏览器 API 的应用，组件与 JavaScript 进行互操作。 组件能够使用 JavaScript 能够使用的任何库或 API。 C# 代码可[调用到 JavaScript 代码](xref:blazor/call-javascript-from-dotnet)，而 JavaScript 代码可[调用到 C# 代码](xref:blazor/call-dotnet-from-javascript)。

## <a name="code-sharing-and-net-standard"></a>代码共享和 .NET Standard

Blazor 实现 [.NET Standard](/dotnet/standard/net-standard)，它使 Blazor 项目引用符合 .NET Standard 规范的库。 .NET Standard 是正式的 .NET API 规范，常见于 .NET 实现中。 .NET Standard 类库可以在不同 .NET 平台之间共享，例如 Blazor、.NET Framework、.NET Core、Xamarin、Mono 和 Unity。

不适用于 Web 浏览器内部的 API（例如，访问文件系统、打开套接字和线程处理）将引发 <xref:System.PlatformNotSupportedException>。

## <a name="additional-resources"></a>其他资源

* [WebAssembly](https://webassembly.org)
* <xref:blazor/hosting-models>
* <xref:tutorials/signalr-blazor-webassembly>
* <xref:blazor/call-javascript-from-dotnet>
* <xref:blazor/call-dotnet-from-javascript>
* [C# 指南](/dotnet/csharp/)
* <xref:mvc/views/razor>
* [HTML](https://www.w3.org/html/)
* [令人惊叹的 Blazor](https://github.com/AdrienTorris/awesome-blazor) 社区链接
