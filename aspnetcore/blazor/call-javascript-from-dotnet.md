---
title: 在 ASP.NET Core Blazor 中从 .NET 方法调用 JavaScript 函数
author: guardrex
description: 了解如何在 Blazor 应用中从 JavaScript 函数调用 .NET 方法。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/02/2020
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
uid: blazor/call-javascript-from-dotnet
ms.openlocfilehash: 3bd881b124e00b91ab0aa9d3eb7531f10ef895f2
ms.sourcegitcommit: b5ebaf42422205d212e3dade93fcefcf7f16db39
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/21/2020
ms.locfileid: "92326500"
---
# <a name="call-javascript-functions-from-net-methods-in-aspnet-core-no-locblazor"></a>在 ASP.NET Core Blazor 中从 .NET 方法调用 JavaScript 函数

作者：[Javier Calvarro Nelson](https://github.com/javiercn)、[Daniel Roth](https://github.com/danroth27) 和 [Luke Latham](https://github.com/guardrex)

Blazor 应用可从 .NET 方法调用 JavaScript 函数，也可从 JavaScript 函数调用 .NET 方法。 这被称为 JavaScript 互操作（JS 互操作） 。

本文介绍如何从 .NET 调用 JavaScript 函数。 有关如何从 JavaScript 调用 .NET 方法的信息，请参阅 <xref:blazor/call-dotnet-from-javascript>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

若要从 .NET 调入 JavaScript，请使用 <xref:Microsoft.JSInterop.IJSRuntime> 抽象。 若要发出 JS 互操作调用，请在组件中注入 <xref:Microsoft.JSInterop.IJSRuntime> 抽象。 <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> 需要使用你要调用的 JavaScript 函数的标识符，以及任意数量的 JSON 可序列化参数。 函数标识符相对于全局范围 (`window`)。 如果要调用 `window.someScope.someFunction`，则标识符是 `someScope.someFunction`。 无需在调用函数之前进行注册。 返回类型 `T` 也必须可进行 JSON 序列化。 `T` 应该与最能映射到所返回 JSON 类型的 .NET 类型匹配。

返回 [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) 的 JavaScript 函数使用 <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A> 调用。 `InvokeAsync` 会将 Promise 解包并返回 Promise 所等待的值。

对于启用了预呈现的 Blazor Server 应用，初始预呈现期间无法调入 JavaScript。 在建立与浏览器的连接之后，必须延迟 JavaScript 互操作调用。 有关详细信息，请参阅[检测 Blazor Server 应用进行预呈现的时间](#detect-when-a-blazor-server-app-is-prerendering)部分。

下面的示例基于 [`TextDecoder`](https://developer.mozilla.org/docs/Web/API/TextDecoder)（一种基于 JavaScript 的解码器）。 此示例展示了如何通过 C# 方法调用 JavaScript 函数，以将要求从开发人员代码卸载到现有 JavaScript API。 JavaScript 函数从 C# 方法接受字节数组，对数组进行解码，并将文本返回给组件进行显示。

在 `wwwroot/index.html` (Blazor WebAssembly) 或 `Pages/_Host.cshtml` (Blazor Server) 的 `<head>` 元素中，提供了一个 JavaScript 函数，该函数使用 `TextDecoder` 对传递的数组进行解码并返回解码后的值：

[!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-convertarray.html)]

JavaScript 代码（如前面示例中所示的代码）也可以通过对脚本文件的引用，从 JavaScript 文件 (`.js`) 加载：

```html
<script src="exampleJsInterop.js"></script>
```

以下组件：

* 在选择了组件按钮（“`Convert Array`”）后，使用 `JSRuntime` 调用 `convertArray` JavaScript 函数。
* 调用 JavaScript 函数之后，传递的数组会转换为字符串。 该字符串会返回给组件进行显示。

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/call-js-example.razor?highlight=2,34-35)]

## <a name="ijsruntime"></a>IJSRuntime

若要使用 <xref:Microsoft.JSInterop.IJSRuntime> 抽象，请采用以下任何方法：

* 将 <xref:Microsoft.JSInterop.IJSRuntime> 抽象注入 Razor 组件 (`.razor`) 中：

  [!code-razor[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction.razor?highlight=1)]

  在 `wwwroot/index.html` (Blazor WebAssembly) 或 `Pages/_Host.cshtml` (Blazor Server) 的 `<head>` 元素中，提供 `handleTickerChanged` JavaScript 函数。 该函数通过 <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> 进行调用，不返回值：

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged1.html)]

* 将 <xref:Microsoft.JSInterop.IJSRuntime> 抽象注入类 (`.cs`)：

  [!code-csharp[](call-javascript-from-dotnet/samples_snapshot/inject-abstraction-class.cs?highlight=5)]

  在 `wwwroot/index.html` (Blazor WebAssembly) 或 `Pages/_Host.cshtml` (Blazor Server) 的 `<head>` 元素中，提供 `handleTickerChanged` JavaScript 函数。 该函数通过 `JSRuntime.InvokeAsync` 进行调用，会返回值：

  [!code-html[](call-javascript-from-dotnet/samples_snapshot/index-script-handleTickerChanged2.html)]

* 对于使用 [BuildRenderTree](xref:blazor/advanced-scenarios#manual-rendertreebuilder-logic) 的动态内容生成，请使用 `[Inject]` 属性：

  ```razor
  [Inject]
  IJSRuntime JSRuntime { get; set; }
  ```

在本主题附带的客户端示例应用中，向应用提供了两个 JavaScript 函数，可与 DOM 交互以接收用户输入并显示欢迎消息：

* `showPrompt`：生成一个提示，用于接受用户输入（用户名），并将用户名返回给调用方。
* `displayWelcome`：将来自调用方的欢迎消息分配给 `id` 为 `welcome` 的 DOM 对象。

`wwwroot/exampleJsInterop.js`：

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=2-7)]

将引用 JavaScript 文件的 `<script>` 标记置于 `wwwroot/index.html` 文件 (Blazor WebAssembly) 或 `Pages/_Host.cshtml` 文件 (Blazor Server) 中。

`wwwroot/index.html` (Blazor WebAssembly):

[!code-html[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/index.html?highlight=22)]

`Pages/_Host.cshtml` (Blazor Server):

[!code-cshtml[](./common/samples/3.x/BlazorServerSample/Pages/_Host.cshtml?highlight=35)]

请勿将 `<script>` 标记置于组件文件中，因为 `<script>` 标记无法动态更新。

.NET 方法通过调用 <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> 与 `exampleJsInterop.js` 文件中的 JavaScript 函数进行互操作。

<xref:Microsoft.JSInterop.IJSRuntime> 抽象是异步的，以便可以实现 Blazor Server 方案。 如果应用是 Blazor WebAssembly 应用，并且要同步调用 JavaScript 函数，则向下转换为 <xref:Microsoft.JSInterop.IJSInProcessRuntime> 并改为调用 <xref:Microsoft.JSInterop.IJSInProcessRuntime.Invoke%2A>。 建议大多数 JS 互操作库使用异步 API，以确保库在所有方案中都可用。

::: moniker range=">= aspnetcore-5.0"

> [!NOTE]
> 若要在标准 [JavaScript 模块](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)中启用 JavaScript 隔离，请参阅 [BlazorJavaScript 隔离和对象引用](#blazor-javascript-isolation-and-object-references)部分。

::: moniker-end

该示例应用包含一个用于演示 JS 互操作的组件。 该组件：

* 通过 JavaScript 提示接收用户输入。
* 将文本返回给组件进行处理。
* 调用第二个 JavaScript 函数，该函数与 DOM 交互以显示欢迎消息。

`Pages/JsInterop.razor`:

```razor
@page "/JSInterop"
@using {APP ASSEMBLY}.JsInteropClasses
@inject IJSRuntime JSRuntime

<h1>JavaScript Interop</h1>

<h2>Invoke JavaScript functions from .NET methods</h2>

<button type="button" class="btn btn-primary" @onclick="TriggerJsPrompt">
    Trigger JavaScript Prompt
</button>

<h3 id="welcome" style="color:green;font-style:italic"></h3>

@code {
    public async Task TriggerJsPrompt()
    {
        var name = await JSRuntime.InvokeAsync<string>(
                "exampleJsFunctions.showPrompt",
                "What's your name?");

        await JSRuntime.InvokeVoidAsync(
                "exampleJsFunctions.displayWelcome",
                $"Hello {name}! Welcome to Blazor!");
    }
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

1. 通过选择组件的“`Trigger JavaScript Prompt`”按钮来执行 `TriggerJsPrompt` 时，则会调用在 `wwwroot/exampleJsInterop.js` 文件中提供的 JavaScript `showPrompt` 函数。
1. `showPrompt` 函数接受进行 HTML 编码并返回给组件的用户输入（用户的名称）。 组件将用户的名称存储在本地变量 `name` 中。
1. 存储在 `name` 中的字符串会合并为欢迎消息，而该消息会传递给 JavaScript 函数 `displayWelcome`（它将欢迎消息呈现到标题标记中）。

## <a name="call-a-void-javascript-function"></a>调用 void JavaScript 函数

在以下场景中使用 <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>：

* 返回 [void(0)/void 0](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/void) 或 [undefined](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/undefined) 的 JavaScript 函数。
* 如果 .NET 不需要读取 JavaScript 调用的结果。

## <a name="detect-when-a-no-locblazor-server-app-is-prerendering"></a>检测 Blazor Server 应用进行预呈现的时间
 
[!INCLUDE[](~/includes/blazor-prerendering.md)]

## <a name="capture-references-to-elements"></a>捕获对元素的引用

某些 JS 互操作方案需要引用 HTML 元素。 例如，一个 UI 库可能需要用于初始化的元素引用，或者你可能需要对元素调用类似于命令的 API（如 `focus` 或 `play`）。

使用以下方法在组件中捕获对 HTML 元素的引用：

* 向 HTML 元素添加 `@ref` 属性。
* 定义一个类型为 <xref:Microsoft.AspNetCore.Components.ElementReference> 字段，其名称与 `@ref` 属性的值匹配。

以下示例演示如何捕获对 `username` `<input>` 元素的引用：

```razor
<input @ref="username" ... />

@code {
    ElementReference username;
}
```

> [!WARNING]
> 只使用元素引用改变不与 Blazor 交互的空元素的内容。 当第三方 API 向元素提供内容时，此方案十分有用。 由于 Blazor 不与元素交互，因此在 Blazor 的元素表示形式与 DOM 之间不可能存在冲突。
>
> 在下面的示例中，改变无序列表 (`ul`) 的内容具有危险性，因为 Blazor 会与 DOM 交互以填充此元素的列表项 (`<li>`)：
>
> ```razor
> <ul ref="MyList">
>     @foreach (var item in Todos)
>     {
>         <li>@item.Text</li>
>     }
> </ul>
> ```
>
> 如果 JS 互操作改变元素 `MyList` 的内容，并且 Blazor 尝试将差异应用于元素，则差异与 DOM 不匹配。

就 .NET 代码而言，<xref:Microsoft.AspNetCore.Components.ElementReference> 是不透明的句柄。 可以对 <xref:Microsoft.AspNetCore.Components.ElementReference> 执行的唯一操作是通过 JS 互操作将它传递给 JavaScript 代码。 执行此操作时，JavaScript 端代码会收到一个 `HTMLElement` 实例，该实例可以与常规 DOM API 一起使用。

例如，以下代码定义一个 .NET 扩展方法，通过该方法可在元素上设置焦点：

`exampleJsInterop.js`:

```javascript
window.exampleJsFunctions = {
  focusElement : function (element) {
    element.focus();
  }
}
```

若要调用不返回值的 JavaScript 函数，请使用 <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType>。 下面的代码通过使用捕获的 <xref:Microsoft.AspNetCore.Components.ElementReference> 调用前面的 JavaScript 函数，在用户名输入上设置焦点：

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component1.razor?highlight=1,3,11-12)]

若要使用扩展方法，请创建接收 <xref:Microsoft.JSInterop.IJSRuntime> 实例的静态扩展方法：

```csharp
public static async Task Focus(this ElementReference elementRef, IJSRuntime jsRuntime)
{
    await jsRuntime.InvokeVoidAsync(
        "exampleJsFunctions.focusElement", elementRef);
}
```

`Focus` 方法在对象上直接调用。 下面的示例假设可从 `JsInteropClasses` 命名空间使用 `Focus` 方法：

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component2.razor?highlight=1-4,12)]

> [!IMPORTANT]
> 仅在呈现组件后填充 `username` 变量。 如果将未填充的 <xref:Microsoft.AspNetCore.Components.ElementReference> 传递给 JavaScript 代码，则 JavaScript 代码会收到 `null` 值。 若要在组件完成呈现之后操作元素引用（用于对元素设置初始焦点），请使用 [`OnAfterRenderAsync` 或 `OnAfterRender` 组件生命周期方法](xref:blazor/components/lifecycle#after-component-render)。

使用泛型类型并返回值时，请使用 <xref:System.Threading.Tasks.ValueTask%601>：

```csharp
public static ValueTask<T> GenericMethod<T>(this ElementReference elementRef, 
    IJSRuntime jsRuntime)
{
    return jsRuntime.InvokeAsync<T>(
        "exampleJsFunctions.doSomethingGeneric", elementRef);
}
```

`GenericMethod` 在具有类型的对象上直接调用。 下面的示例假设可从 `JsInteropClasses` 命名空间使用 `GenericMethod`：

[!code-razor[](call-javascript-from-dotnet/samples_snapshot/component3.razor?highlight=17)]

## <a name="reference-elements-across-components"></a>跨组件引用元素

<xref:Microsoft.AspNetCore.Components.ElementReference> 实例仅保证在组件的 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 方法中有效（并且元素引用为 `struct`），因此无法在组件之间传递元素引用。 若要使父组件可以向其他组件提供元素引用，父组件可以：

* 允许子组件注册回调。
* 在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 事件期间，通过传递的元素引用调用注册的回调。 此方法间接地允许子组件与父级的元素引用交互。

下面的 Blazor WebAssembly 示例阐释了这种方法。

在 `wwwroot/index.html` 的 `<head>` 中：

```html
<style>
    .red { color: red }
</style>
```

在 `wwwroot/index.html` 的 `<body>` 中：

```html
<script>
    function setElementClass(element, className) {
        /** @type {HTMLElement} **/
        var myElement = element;
        myElement.classList.add(className);
    }
</script>
```

`Pages/Index.razor`（父组件）：

```razor
@page "/"

<h1 @ref="title">Hello, world!</h1>

Welcome to your new app.

<SurveyPrompt Parent="this" Title="How is Blazor working for you?" />
```

`Pages/Index.razor.cs`:

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Components;

namespace {APP ASSEMBLY}.Pages
{
    public partial class Index : 
        ComponentBase, IObservable<ElementReference>, IDisposable
    {
        private bool disposing;
        private IList<IObserver<ElementReference>> subscriptions = 
            new List<IObserver<ElementReference>>();
        private ElementReference title;

        protected override void OnAfterRender(bool firstRender)
        {
            base.OnAfterRender(firstRender);

            foreach (var subscription in subscriptions)
            {
                try
                {
                    subscription.OnNext(title);
                }
                catch (Exception)
                {
                    throw;
                }
            }
        }

        public void Dispose()
        {
            disposing = true;

            foreach (var subscription in subscriptions)
            {
                try
                {
                    subscription.OnCompleted();
                }
                catch (Exception)
                {
                }
            }

            subscriptions.Clear();
        }

        public IDisposable Subscribe(IObserver<ElementReference> observer)
        {
            if (disposing)
            {
                throw new InvalidOperationException("Parent being disposed");
            }

            subscriptions.Add(observer);

            return new Subscription(observer, this);
        }

        private class Subscription : IDisposable
        {
            public Subscription(IObserver<ElementReference> observer, Index self)
            {
                Observer = observer;
                Self = self;
            }

            public IObserver<ElementReference> Observer { get; }
            public Index Self { get; }

            public void Dispose()
            {
                Self.subscriptions.Remove(Observer);
            }
        }
    }
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

`Shared/SurveyPrompt.razor`（子组件）：

```razor
@inject IJSRuntime JS

<div class="alert alert-secondary mt-4" role="alert">
    <span class="oi oi-pencil mr-2" aria-hidden="true"></span>
    <strong>@Title</strong>

    <span class="text-nowrap">
        Please take our
        <a target="_blank" class="font-weight-bold" 
            href="https://go.microsoft.com/fwlink/?linkid=2109206">brief survey</a>
    </span>
    and tell us what you think.
</div>

@code {
    [Parameter]
    public string Title { get; set; }
}
```

`Shared/SurveyPrompt.razor.cs`:

```csharp
using System;
using Microsoft.AspNetCore.Components;

namespace {APP ASSEMBLY}.Shared
{
    public partial class SurveyPrompt : 
        ComponentBase, IObserver<ElementReference>, IDisposable
    {
        private IDisposable subscription = null;

        [Parameter]
        public IObservable<ElementReference> Parent { get; set; }

        protected override void OnParametersSet()
        {
            base.OnParametersSet();

            if (subscription != null)
            {
                subscription.Dispose();
            }

            subscription = Parent.Subscribe(this);
        }

        public void OnCompleted()
        {
            subscription = null;
        }

        public void OnError(Exception error)
        {
            subscription = null;
        }

        public void OnNext(ElementReference value)
        {
            JS.InvokeAsync<object>(
                "setElementClass", new object[] { value, "red" });
        }

        public void Dispose()
        {
            subscription?.Dispose();
        }
    }
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

## <a name="harden-js-interop-calls"></a>强化 JS 互操作调用

JS 互操作可能会由于网络错误而失败，因此应视为不可靠。 默认情况下，Blazor Server 应用会在一分钟后，使服务器上的 JS 互操作调用超时。 如果应用可以容忍更激进的超时，请使用以下方法之一设置超时：

* 在 `Startup.ConfigureServices` 中全局指定超时：

  ```csharp
  services.AddServerSideBlazor(
      options => options.JSInteropDefaultCallTimeout = TimeSpan.FromSeconds({SECONDS}));
  ```

* 对于组件代码中的每个调用，单个调用可以指定超时：

  ```csharp
  var result = await JSRuntime.InvokeAsync<string>("MyJSOperation", 
      TimeSpan.FromSeconds({SECONDS}), new[] { "Arg1" });
  ```

有关资源耗尽的详细信息，请参阅 <xref:blazor/security/server/threat-mitigation>。

[!INCLUDE[](~/includes/blazor-share-interop-code.md)]

## <a name="avoid-circular-object-references"></a>避免循环引用对象

不能在客户端上针对以下调用就包含循环引用的对象进行序列化：

* .NET 方法调用。
* 返回类型具有循环引用时，从 C# 发出的 JavaScript 方法调用。

有关详细信息，请查看以下问题：

* [循环引用不受支持，使用两个按钮 (dotnet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525)
* [建议：在序列化时添加机制来处理循环引用 (dotnet/runtime #30820)](https://github.com/dotnet/runtime/issues/30820)

::: moniker range=">= aspnetcore-5.0"

## <a name="no-locblazor-javascript-isolation-and-object-references"></a>Blazor JavaScript 隔离和对象引用

Blazor 在标准 [JavaScript 模块](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Modules)中启用 JavaScript 隔离。 JavaScript 隔离具有以下优势：

* 导入的 JavaScript 不再污染全局命名空间。
* 库和组件的使用者不需要导入相关的 JavaScript。

例如，以下 JavaScript 模块导出用于显示浏览器提示的 JavaScript 函数：

```javascript
export function showPrompt(message) {
  return prompt(message, 'Type anything here');
}
```

将前面的 JavaScript 模块作为静态 Web 资产 (`wwwroot/exampleJsInterop.js`) 添加到 .NET 库，然后使用 <xref:Microsoft.JSInterop.IJSRuntime> 服务将该模块导入 .NET 代码。 以下示例将服务作为 `jsRuntime`（未显示）注入：

```csharp
var module = await jsRuntime.InvokeAsync<IJSObjectReference>(
    "import", "./_content/MyComponents/exampleJsInterop.js");
```

上例中的 `import` 标识符是专门用于导入 JavaScript 模块的特殊标识符。 使用模块的稳定静态 Web 资产路径 `_content/{LIBRARY NAME}/{PATH UNDER WWWROOT}` 指定模块。 占位符 `{LIBRARY NAME}` 是库的名称。 占位符 `{PATH UNDER WWWROOT}` 是 `wwwroot` 下脚本的路径。

<xref:Microsoft.JSInterop.IJSRuntime> 将模块作为 `IJSObjectReference` 导入，它表示对 .NET 代码中 JavaScript 对象的引用。 使用 `IJSObjectReference` 调用从模块导出的 JavaScript 函数：

```csharp
public async ValueTask<string> Prompt(string message)
{
    return await module.InvokeAsync<string>("showPrompt", message);
}
```

`IJSInProcessObjectReference` 表示对某个 JavaScript 对象的引用，该对象的函数可以同步进行调用。

`IJSUnmarshalledObjectReference` 表示对某个 JavaScript 对象的引用，该对象的函数无需 .NET 数据序列化开销即可调用。 当性能非常重要时，可以在 Blazor WebAssembly 中使用此项：

```javascript
window.unmarshalledInstance = {
  helloWorld: function (personNamePointer) {
    const personName = Blazor.platform.readStringField(value, 0);
    return `Hello ${personName}`;
  }
};
```

```csharp
var unmarshalledRuntime = (IJSUnmarshalledRuntime)jsRuntime;
var jsUnmarshalledReference = unmarshalledRuntime
    .InvokeUnmarshalled<IJSUnmarshalledObjectReference>("unmarshalledInstance");

string helloWorldString = jsUnmarshalledReference.InvokeUnmarshalled<string, string>(
    "helloWorld");
```

## <a name="use-of-javascript-libraries-that-render-ui-dom-elements"></a>使用呈现 UI 的 JavaScript 库（DOM 元素）

有时你可能需要使用在浏览器 DOM 内生成可见用户界面元素的 JavaScript 库。 乍一想，这似乎很难，因为 Blazor 的 diffing 系统依赖于对 DOM 元素树的控制，并且如果某个外部代码使 DOM 树发生变化并为了应用 diff 而使其机制失效，就会产生错误。 这并不是一个特定于 Blazor 的限制。 任何基于 diff 的 UI 框架都会面临同样的问题。

幸运的是，将外部生成的 UI 可靠地嵌入到 Blazor 组件 UI 非常简单。 推荐的方法是让组件的代码（`.razor` 文件）生成一个空元素。 就 Blazor 的 diffing 系统而言，该元素始终为空，这样呈现器就不会递归到元素中，而是保留元素内容不变。 这样就可以安全地用外部托管的任意内容来填充元素。

以下示例演示了这一概念。 在 `if` 语句中，当 `firstRender` 为 `true` 时，使用 `myElement` 来执行某些操作。 例如，调用某个外部 JavaScript 库来填充它。 Blazor 保留元素内容不变，直到此组件本身被删除。 删除组件时，会同时删除组件的整个 DOM 子树。

```razor
<h1>Hello! This is a Blazor component rendered at @DateTime.Now</h1>

<div @ref="myElement"></div>

@code {
    HtmlElement myElement;
    
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            ...
        }
    }
}
```

为了更详细地说明，请思考以下组件，该组件使用[开源 Mapbox API](https://www.mapbox.com/) 呈现交互式地图：

```razor
@inject IJSRuntime JS
@implements IAsyncDisposable

<div @ref="mapElement" style='width: 400px; height: 300px;'></div>

<button @onclick="() => ShowAsync(51.454514, -2.587910)">Show Bristol, UK</button>
<button @onclick="() => ShowAsync(35.6762, 139.6503)">Show Tokyo, Japan</button>

@code
{
    ElementReference mapElement;
    IJSObjectReference mapModule;
    IJSObjectReference mapInstance;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            mapModule = await JS.InvokeAsync<IJSObjectReference>(
                "import", "./mapComponent.js");
            mapInstance = await mapModule.InvokeAsync<IJSObjectReference>(
                "addMapToElement", mapElement);
        }
    }

    Task ShowAsync(double latitude, double longitude)
        => mapModule.InvokeVoidAsync("setMapCenter", mapInstance, latitude, 
            longitude).AsTask();

    private async ValueTask IAsyncDisposable.DisposeAsync()
    {
        await mapInstance.DisposeAsync();
        await mapModule.DisposeAsync();
    }
}
```

相应的 JavaScript 模块（应置于 `wwwroot/mapComponent.js` 处）如下所示：

```javascript
import 'https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.js';

// TO MAKE THE MAP APPEAR YOU MUST ADD YOUR ACCESS TOKEN FROM 
// https://account.mapbox.com
mapboxgl.accessToken = '{ACCESS TOKEN}';

export function addMapToElement(element) {
  return new mapboxgl.Map({
    container: element,
    style: 'mapbox://styles/mapbox/streets-v11',
    center: [-74.5, 40],
    zoom: 9
  });
}

export function setMapCenter(map, latitude, longitude) {
  map.setCenter([longitude, latitude]);
}
```

在前面的示例中，将字符串 `{ACCESS TOKEN}` 替换为可从 https://account.mapbox.com 中获取的有效访问令牌。

为生成正确的样式，将以下样式表标记添加到主机 HTML 页面中（`index.html` 或 `_Host.cshtml`）：

```html
<link rel="stylesheet" href="https://api.mapbox.com/mapbox-gl-js/v1.12.0/mapbox-gl.css" />
```

前面的示例会生成一个交互式地图 UI，其中用户：

* 拖动鼠标可以滚动或缩放。
* 单击相关按钮可跳转到预定义的位置。

![Mapbox 的日本东京街道地图，其中设有可用于选择英国布里斯托尔和日本东京的按钮](https://user-images.githubusercontent.com/1101362/94939821-92ef6700-04ca-11eb-858e-fff6df0053ae.png)

要了解的要点如下：

 * 就 Blazor 而言，具有 `@ref="mapElement"` 的 `<div>` 保留为空。 这样随着时间推移，`mapbox-gl.js` 填充和修改其内容是安全的。 可以将此方法与呈现 UI 的任何 JavaScript 库配合使用。 甚至可以在 Blazor 组件中嵌入第三方 JavaScript SPA 框架中的组件，只要它们不会尝试访问和改变页面的其他部分。 对于外部 JavaScript 代码，修改 Blazor 不视为空元素的元素是*不*安全的。
 * 使用此方法时，请记得有关 Blazor 保留或销毁 DOM 元素的方式的规则。 在前面的示例中，组件之所以能够安全处理按钮单击事件并更新现有的地图实例，是因为默认在可能的情况下保留 DOM 元素。 如果之前要从 `@foreach` 循环内呈现地图元素的列表，则需要使用 `@key` 来确保保留组件实例。 否则，列表数据的更改可能导致组件实例以不合适的方式保留以前实例的状态。 有关详细信息，请参阅[使用 @key 保留元素和组件](xref:blazor/components/index#use-key-to-control-the-preservation-of-elements-and-components)。

此外，上面的示例演示了如何在 ES6 模块中封装 JavaScript 逻辑和依赖关系，并使用 `import` 标识符动态加载该模块。 有关详细信息，请参阅 [JavaScript 隔离和对象引用](#blazor-javascript-isolation-and-object-references)。

::: moniker-end

## <a name="size-limits-on-js-interop-calls"></a>对 JS 互操作调用的大小限制

在 Blazor WebAssembly 中，框架对 JS 互操作调用的输入和输出大小不施加限制。

在 Blazor Server 中，JS 互操作调用的结果受 SignalR (<xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize>) 执行的最大有效负载大小的限制，默认值为 32 KB。 尝试响应有效负载大于 <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> 的 JS 互操作调用的应用程序将引发错误。 可以通过修改 <xref:Microsoft.AspNetCore.SignalR.HubOptions.MaximumReceiveMessageSize> 来配置更大的限制。 下面的示例将最大接收消息大小设置为 64 KB (64 * 1024 * 1024)：

```csharp
services.AddServerSideBlazor()
   .AddHubOptions(options => options.MaximumReceiveMessageSize = 64 * 1024 * 1024);
```

提高 SignalR 上限的代价是需要使用更多的服务器资源，这将使服务器面临来自恶意用户的更大风险。 此外，如果将大量内容作为字符串或字节数组读入内存中，还会导致垃圾回收器的分配工作状况不佳，从而导致额外的性能损失。 读取大型有效负载的一种方法是考虑以较小区块发送内容，并将有效负载作为 <xref:System.IO.Stream> 处理。 可以在读取大型 JSON 有效负载或者数据在 JavaScript 中以原始字节形式提供时使用此方法。 有关演示如何使用类似于 `InputFile` 组件的方法在 Blazor Server 中发送大型二进制有效负载的示例，请参阅[二进制文件提交示例应用](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/BinarySubmit)。

开发在 JavaScript 和 Blazor 之间传输大量数据的代码时，请考虑以下指南：

* 将数据切成小块，然后按顺序发送数据段，直到服务器收到所有数据。
* 不要在 JavaScript 和 C# 代码中分配大型对象。
* 发送或接收数据时，请勿长时间阻止主 UI 线程。
* 在进程完成或取消时释放消耗的所有内存。
* 为了安全起见，请强制执行以下附加要求：
  * 声明可以传递的最大文件或数据大小。
  * 声明从客户端到服务器的最低上传速率。
* 在服务器收到数据后，数据可以：
  * 暂时存储在内存缓冲区中，直到收集完所有数据段。
  * 立即使用。 例如，在收到每个数据段时，数据可以立即存储到数据库中或写入磁盘。

## <a name="additional-resources"></a>其他资源

* <xref:blazor/call-dotnet-from-javascript>
* [InteropComponent.razor 示例（dotnet/AspNetCore GitHub 存储库，3.1 版本分支）](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
