---
title: 调试 ASP.NET Core Blazor WebAssembly
author: guardrex
description: 了解如何调试 Blazor 应用。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/26/2020
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
uid: blazor/debug
ms.openlocfilehash: b8dd272d673e84b45a39272531385ebfd1d06175
ms.sourcegitcommit: daa9ccf580df531254da9dce8593441ac963c674
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/09/2020
ms.locfileid: "91900981"
---
# <a name="debug-aspnet-core-no-locblazor-webassembly"></a>调试 ASP.NET Core Blazor WebAssembly

[Daniel Roth](https://github.com/danroth27)

可以使用基于 Chromium 的浏览器 (Edge/Chrome) 中的浏览器开发工具调试 Blazor WebAssembly 应用。 还可以使用以下集成开发环境 (IDE) 调试应用：

* Visual Studio
* Visual Studio for Mac
* Visual Studio Code

可用方案包括：

* 设置和删除断点。
* 在 IDE 中运行具有调试支持的应用。
* 单步执行代码。
* 在 IDE 中使用键盘快捷方式恢复代码执行。
* 在“局部变量”窗口中，观察局部变量的值。
* 请参阅调用堆栈，包括 JavaScript 和 .NET 之间的调用链。

目前，无法执行以下操作：

* 出现未经处理的异常时中断。
* 于应用启动期间在调试代理运行之前命中断点。 这包括 `Program.Main` (`Program.cs`) 中的断点和组件的 [`OnInitialized{Async}` 方法](xref:blazor/components/lifecycle#component-initialization-methods) 中的断点，其中这些组件由请求自应用的第一页加载。

## <a name="prerequisites"></a>先决条件

调试需要使用以下浏览器之一：

* Google Chrome（版本 70 或更高版本）（默认）
* Microsoft Edge（版本 80 或更高版本）

Visual Studio for Mac 需要版本 8.8（内部版本 1532）或更高版本：

1. 选择 [Microsoft：Visual Studio for Mac](https://visualstudio.microsoft.com/vs/mac/) 页面上的“下载 Visual Studio for Mac”按钮，安装最新版本的 Visual Studio for Mac。
1. 从 Visual Studio 中选择预览通道。 有关详细信息，请参阅[安装 Visual Studio for Mac 的预览版本](/visualstudio/mac/install-preview)。

> [!NOTE]
> 当前不支持 macOS 上的 Apple Safari。

## <a name="enable-debugging"></a>启用调试

若要为现有 Blazor WebAssembly 应用启用调试，请更新启动项目中的 `launchSettings.json` 文件，使每个启动配置文件包含以下 `inspectUri` 属性：

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/_framework/debug/ws-proxy?browser={browserInspectUri}"
```

更新后，`launchSettings.json` 文件应类似于以下示例：

[!code-json[](debug/launchSettings.json?highlight=14,22)]

`inspectUri` 属性具有以下作用：

* 使 IDE 能够检测到该应用为 Blazor WebAssembly 应用。
* 指示脚本调试基础结构通过 Blazor 的调试代理连接到浏览器。

已启动的浏览器 (`browserInspectUri`) 上 WebSocket 协议 (`wsProtocol`)、主机 (`url.hostname`)、端口 (`url.port`) 和检查器 URI 的占位符值由框架提供。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

要在 Visual Studio 中调试 Blazor WebAssembly 应用，请按以下步骤执行：

1. 创建新的 ASP.NET Core 托管 Blazor WebAssembly 应用。
1. 按 <kbd>F5</kbd> 在调试器中运行应用。

   > [!NOTE]
   > 不支持“启动时不调试”(<kbd>Ctrl</kbd>+<kbd>F5</kbd>)。 当应用以调试配置运行时，调试开销始终会导致性能的小幅下降。

1. 在 `*Client*` 应用中，在 `Pages/Counter.razor` 中的 `currentCount++;` 行上设置断点。
1. 在浏览器中，导航到 `Counter` 页，然后选择“单击此处”按钮以命中断点。
1. 在 Visual Studio 中，检查“局部变量”窗口中 `currentCount` 字段的值。
1. 按 <kbd>F5</kbd> 继续执行。

调试 Blazor WebAssembly 应用时，还可以调试服务器代码：

1. 在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 的“`Pages/FetchData.razor`”页中设置断点。
1. 在 `Get` 操作方法的 `WeatherForecastController` 中设置一个断点。
1. 浏览到 `Fetch Data` 页，在 `FetchData` 组件中的首个断点向服务器发出 HTTP 请求前命中该断点。
1. 按 <kbd>F5</kbd> 以继续执行，然后在服务器上命中 `WeatherForecastController` 中的断点。
1. 再次按 <kbd>F5</kbd> 以继续执行，并查看浏览器中呈现的天气预报表。

> [!NOTE]
> 在运行调试代理之前，在应用启动期间不会命中断点。 这包括 `Program.Main` (`Program.cs`) 中的断点和组件的 [`OnInitialized{Async}` 方法](xref:blazor/components/lifecycle#component-initialization-methods) 中的断点，其中这些组件由请求自应用的第一页加载。

如果应用托管在不同于 `/` 的[应用程序基路径](xref:blazor/host-and-deploy/index#app-base-path)上，请更新 `Properties/launchSettings.json` 中的以下属性，以反映应用程序的基路径：

* `applicationUrl`:

  ```json
  "iisSettings": {
    ...
    "iisExpress": {
      "applicationUrl": "http://localhost:{INSECURE PORT}/{APP BASE PATH}/",
      "sslPort": {SECURE PORT}
    }
  },
  ```

* 每个配置文件的 `inspectUri`：

  ```json
  "profiles": {
    ...
    "{PROFILE 1, 2, ... N}": {
      ...
      "inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/{APP BASE PATH}/_framework/debug/ws-proxy?browser={browserInspectUri}",
      ...
    }
  }
  ```

前面设置中的占位符：

* `{INSECURE PORT}`：不安全的端口。 默认情况下提供随机值，但允许使用自定义端口。
* `{APP BASE PATH}`：应用程序的基路径。
* `{SECURE PORT}`：安全的端口。 默认情况下提供随机值，但允许使用自定义端口。
* `{PROFILE 1, 2, ... N}`：启动设置配置文件。 通常，应用会默认指定多个配置文件（例如，IIS Express 的配置文件和 Kestrel 服务器使用的项目配置文件）。

在下面的示例中，应用托管在 `/OAT` 上，并在 `wwwroot/index.html` 中将应用程序基路径配置为 `<base href="/OAT/">`：

```json
"applicationUrl": "http://localhost:{INSECURE PORT}/OAT/",
```

```json
"inspectUri": "{wsProtocol}://{url.hostname}:{url.port}/OAT/_framework/debug/ws-proxy?browser={browserInspectUri}",
```

有关将自定义应用基路径用于 Blazor WebAssembly 应用的信息，请参阅 <xref:blazor/host-and-deploy/index#app-base-path>。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

<h2 id="vscode">调试独立 Blazor WebAssembly</h2>

1. 在 VS Code 中打开独立 Blazor WebAssembly 应用。

   可能会收到通知，告诉你需要进行其他设置才能启用调试：

   > 需要进行其他设置才能调试 Blazor WebAssembly 应用程序。

   如果收到通知，请执行以下操作：

   * 确认是否已安装最新的[适用于 Visual Studio Code 的 C# 扩展](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp)。 若要检查是否已安装此扩展，请在菜单栏中依次打开“视图” > “扩展”，或选择“活动”边栏中的“扩展”图标。
   * 确认已启用 JavaScript 预览调试。 在菜单栏中打开设置（依次选择“文件” > “首选项” > “设置”）。 使用关键字 `debug preview` 进行搜索。 在搜索结果中，确认是否已选中“调试”>“JavaScript:使用预览”复选框。 如果启用预览调试的选项不存在，请升级到最新版本的 VS Code 或安装 [JavaScript 调试器扩展](https://marketplace.visualstudio.com/items?itemName=ms-vscode.js-debug-nightly)（VS Code 1.46 或更早版本）。
   * 重载窗口。

1. 使用 <kbd>F5</kbd> 键盘快捷方式或菜单项启动调试。

   > [!NOTE]
   > 不支持“启动时不调试”(<kbd>Ctrl</kbd>+<kbd>F5</kbd>)。 当应用以调试配置运行时，调试开销始终会导致性能的小幅下降。

1. 出现提示时，选择“Blazor WebAssembly 调试”选项以启动调试。

1. 此时会启动独立应用，并打开调试浏览器。

1. 在 `*Client*` 应用中，在 `Pages/Counter.razor` 中的 `currentCount++;` 行上设置断点。

1. 在浏览器中，导航到 `Counter` 页，然后选择“单击此处”按钮以命中断点。

> [!NOTE]
> 在运行调试代理之前，在应用启动期间不会命中断点。 这包括 `Program.Main` (`Program.cs`) 中的断点和组件的 [`OnInitialized{Async}` 方法](xref:blazor/components/lifecycle#component-initialization-methods) 中的断点，其中这些组件由请求自应用的第一页加载。

## <a name="debug-hosted-no-locblazor-webassembly"></a>调试托管 Blazor WebAssembly

1. 在 VS Code 中打开托管 Blazor WebAssembly 应用的“解决方案”文件夹。

1. 如果没有为项目设置启动配置，将显示以下通知。 选择 **“是”** 。

   > “{APPLICATION NAME}”中缺少进行生产和调试所需的资产。 警告的 VS Code

1. 在窗口顶部的命令面板中，选择托管解决方案内的“服务器”项目。

这会生成一个 `launch.json` 文件，其中包含用于启动调试器的启动配置。

## <a name="attach-to-an-existing-debugging-session"></a>附加到现有的调试会话

若要附加到正在运行的 Blazor 应用，请创建一个具有以下配置的 `launch.json` 文件：

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach to Existing Blazor WebAssembly Application"
}
```

> [!NOTE]
> 只有独立应用才支持附加到调试会话。 若要使用完整堆栈调试，必须从 VS Code 启动应用。

## <a name="launch-configuration-options"></a>启动配置选项

`blazorwasm` 调试类型 (`.vscode/launch.json`) 支持以下启动配置选项。

| 选项    | 描述 |
| --------- | ----------- |
| `request` | 使用 `launch` 启动调试会话并将其附加到 Blazor WebAssembly 应用，或使用 `attach` 将调试会话附加到已运行的应用。 |
| `url`     | 调试时要在浏览器中打开的 URL。 默认为 `https://localhost:5001`。 |
| `browser` | 要为调试会话启动的浏览器。 设置为 `edge` 或 `chrome`。 默认为 `chrome`。 |
| `trace`   | 用于从 JS 调试程序生成日志。 设置为 `true` 即可生成日志。 |
| `hosted`  | 如果要启动和调试托管的 Blazor WebAssembly 应用，则必须设置为 `true`。 |
| `webRoot` | 指定 Web 服务器的绝对路径。 如果应用是从子路由中提供的，则应设置它。 |
| `timeout` | 等待调试会话附加完成的毫秒数。 默认值为 30,000 毫秒（30 秒）。 |
| `program` | 为运行托管应用的服务器而对可执行文件进行的引用。 如果 `hosted` 是 `true`，则必须设置它。 |
| `cwd`     | 要在其中启动应用的工作目录。 如果 `hosted` 是 `true`，则必须设置它。 |
| `env`     | 要提供给已启动进程的环境变量。 仅当 `hosted` 设置为 `true` 时才适用。 |

## <a name="example-launch-configurations"></a>启动配置示例

### <a name="launch-and-debug-a-standalone-no-locblazor-webassembly-app"></a>启动并调试独立 Blazor WebAssembly 应用

```json
{
  "type": "blazorwasm",
  "request": "launch",
  "name": "Launch and Debug"
}
```

### <a name="attach-to-a-running-app-at-a-specified-url"></a>在指定的 URL 附加到正在运行的应用

```json
{
  "type": "blazorwasm",
  "request": "attach",
  "name": "Attach and Debug",
  "url": "http://localhost:5000"
}
```

### <a name="launch-and-debug-a-hosted-no-locblazor-webassembly-app-with-microsoft-edge"></a>使用 Microsoft Edge 启动并调试托管 Blazor WebAssembly 应用

浏览器默认配置为 Google Chrome。 使用 Microsoft Edge 进行调试时，将 `browser` 设置为 `edge`。 若要使用 Google Chrome，要么不要设置 `browser` 选项，要么将选项值设置为 `chrome`。

```json
{
  "name": "Launch and Debug Hosted Blazor WebAssembly App",
  "type": "blazorwasm",
  "request": "launch",
  "hosted": true,
  "program": "${workspaceFolder}/Server/bin/Debug/netcoreapp3.1/MyHostedApp.Server.dll",
  "cwd": "${workspaceFolder}/Server",
  "browser": "edge"
}
```

在前面的示例中，`MyHostedApp.Server.dll` 是“服务器”应用的程序集。 在“解决方案”文件夹中，“`.vscode`”文件夹位于“`Client`”、“`Server`”和“`Shared`”文件夹旁边。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

要在 Visual Studio for Mac 中调试 Blazor WebAssembly 应用，请按以下步骤执行：

1. 创建新的 ASP.NET Core 托管 Blazor WebAssembly 应用。
1. 按 <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> 在调试器中运行应用。

   > [!NOTE]
   > 不支持“启动时不调试”(<kbd>&#8997;</kbd>+<kbd>&#8984;</kbd>+<kbd>&#8617;</kbd>)。 当应用以调试配置运行时，调试开销始终会导致性能的小幅下降。

   > [!IMPORTANT]
   > Google Chrome 或 Microsoft Edge 必须是调试会话的选定浏览器。

1. 在 `*Client*` 应用中，在 `Pages/Counter.razor` 中的 `currentCount++;` 行上设置断点。
1. 在浏览器中，导航到 `Counter` 页，然后选择“单击此处”按钮以命中断点：
1. 在 Visual Studio 中，检查“局部变量”窗口中 `currentCount` 字段的值。
1. 按 <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> 继续执行。

调试 Blazor WebAssembly 应用时，还可以调试服务器代码：

1. 在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 的“`Pages/FetchData.razor`”页中设置断点。
1. 在 `Get` 操作方法的 `WeatherForecastController` 中设置一个断点。
1. 浏览到 `Fetch Data` 页，在 `FetchData` 组件中的首个断点向服务器发出 HTTP 请求前命中该断点。
1. 按 <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> 继续执行，然后在服务器上命中 `WeatherForecastController` 中的断点。
1. 再次按 <kbd>&#8984;</kbd>+<kbd>&#8617;</kbd> 以继续执行，并查看浏览器中呈现的天气预报表。

> [!NOTE]
> 在运行调试代理之前，在应用启动期间不会命中断点。 这包括 `Program.Main` (`Program.cs`) 中的断点和组件的 [`OnInitialized{Async}` 方法](xref:blazor/components/lifecycle#component-initialization-methods) 中的断点，其中这些组件由请求自应用的第一页加载。

有关详细信息，请参阅[使用 Visual Studio for Mac 进行调试](/visualstudio/mac/debugging)。

---

## <a name="debug-in-the-browser"></a>在浏览器中调试

本部分中的指南适用于在 Windows 上运行的 Google Chrome 和 Microsoft Edge。

1. 在开发环境中运行该应用的调试版本。

1. 启动浏览器并导航到应用程序的 URL（例如 `https://localhost:5001`）。

1. 在浏览器中，尝试通过按 <kbd>Shift</kbd>+<kbd>Alt</kbd>+<kbd>d</kbd> 启动远程调试。

   浏览器必须在已启用远程调试的情况下运行，而这并不是默认设置。 如果远程调试处于禁用状态，将呈现“无法找到可调试的浏览器选项卡”错误页，并且其中包含关于在调试端口打开的情况下启动浏览器的说明。 按照适用于你的浏览器的说明操作，接下来将打开一个新的浏览器窗口。 关闭上一个浏览器窗口。

<!-- HOLD 
1. In the browser, attempt to commence remote debugging by pressing:

   * <kbd>Shift</kbd>+<kbd>Alt</kbd>+<kbd>d</kbd> on Windows.
   * <kbd>Shift</kbd>+<kbd>&#8984;</kbd>+<kbd>d</kbd> on macOS.

   The browser must be running with remote debugging enabled, which isn't the default. If remote debugging is disabled, an **Unable to find debuggable browser tab** error page is rendered with instructions for launching the browser with the debugging port open. Follow the instructions for your browser, which opens a new browser window. Close the previous browser window.
-->

1. 在启用远程调试的情况下运行浏览器后，按上一步中的调试键盘快捷方式会打开新的调试程序选项卡。

1. 片刻后，“源”选项卡显示 `file://` 节点中应用的 .NET 程序集的列表。

1. 在组件代码（`.razor` 文件）和 C# 代码文件 (`.cs`) 中，当代码执行时将命中你设置的断点。 命中断点后，正常单步执行代码 (<kbd>F10</kbd>) 或恢复代码执行 (<kbd>F8</kbd>)。

Blazor 提供调试代理，该代理实现 [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/)，并使用特定于 .NET 的信息扩展该协议。 按下调试键盘快捷方式后，Blazor 会将 Chrome 开发者工具指向代理。 代理连接到要调试的浏览器窗口（因此需要启用远程调试）。

## <a name="browser-source-maps"></a>浏览器源映射

浏览器源映射允许浏览器将编译后的文件映射回其原始源文件，并且通常用于客户端调试。 但是，Blazor 当前不直接将 C# 映射到 JavaScript/WASM。 相反，Blazor 在浏览器中进行 IL 解释，因此源映射不相关。

## <a name="troubleshoot"></a>疑难解答

如果遇到错误，以下提示可能有所帮助：

* 在“调试程序”选项卡中，在浏览器中打开开发人员工具。 在控制台中，执行 `localStorage.clear()` 以删除所有断点。
* 确认你已安装并信任 ASP.NET Core HTTPS 开发证书。 有关详细信息，请参阅 <xref:security/enforcing-ssl#troubleshoot-certificate-problems>。
* Visual Studio 要求在“工具” > “选项” > “调试” > “常规”中选择“对 ASP.NET 启用 JavaScript 调试(Chrome、Edge 和 IE)”选项。 这是 Visual Studio 的默认设置。 如果调试不起作用，请确认已选中该选项。
* 如果你的环境使用 HTTP 代理，请确保在代理绕过设置中包含 `localhost`。 这可以通过在以下二者之一中设置 `NO_PROXY` 环境变量来实现：
  * 项目的 `launchSettings.json` 文件。
  * 在将其应用于所有应用时所在的用户或系统环境变量级别。 使用环境变量时，请重新启动 Visual Studio 以使更改生效。

### <a name="breakpoints-in-oninitializedasync-not-hit"></a>`OnInitialized{Async}` 中的未命中断点

Blazor 框架的调试代理需要一小段时间才能启动，因此 [`OnInitialized{Async}` 生命周期方法](xref:blazor/components/lifecycle#component-initialization-methods)中的断点可能不会命中。 建议在方法主体开头添加延迟，以便在命中断点之前，为调试代理指定一段时间来启动。 你可以根据 [`if` 编译器指令](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-if)包括延迟，以确保应用的发布版本不存在延迟。

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>:

```csharp
protected override void OnInitialized()
{
#if DEBUG
    Thread.Sleep(10000)
#endif

    ...
}
```

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>:

```csharp
protected override async Task OnInitializedAsync()
{
#if DEBUG
    await Task.Delay(10000)
#endif

    ...
}
```

### <a name="visual-studio-windows-timeout"></a>Visual Studio (Windows) 超时

如果 Visual Studio 引发了调试适配器启动因已达到超时而失败的异常，可使用注册表设置调整超时：

```console
VsRegEdit.exe set "<VSInstallFolder>" HKCU JSDebugger\Options\Debugging "BlazorTimeoutInMilliseconds" dword {TIMEOUT}
```

前述命令中的 `{TIMEOUT}` 占位符以毫秒为单位。 例如， 1 分钟指定为 `60000`。
