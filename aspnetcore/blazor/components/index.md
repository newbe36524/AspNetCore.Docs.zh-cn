---
title: 创建和使用 ASP.NET Core Razor 组件
author: guardrex
description: 了解如何创建和使用 Razor 组件，包括如何绑定到数据、处理事件和管理组件生命周期。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/19/2020
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
uid: blazor/components/index
ms.openlocfilehash: d30f40945a3b2799dfc2d9391bba37eee1bfdc18
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93056265"
---
# <a name="create-and-use-aspnet-core-no-locrazor-components"></a>创建和使用 ASP.NET Core Razor 组件

作者：[Luke Latham](https://github.com/guardrex)、[Daniel Roth](https://github.com/danroth27) 和 [Tobias Bartsch](https://www.aveo-solutions.com/)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

Blazor 应用是使用组件构建的。 组件是自包含的用户界面 (UI) 块，例如页、对话框或窗体。 组件包含插入数据或响应 UI 事件所需的 HTML 标记和处理逻辑。 组件非常灵活且轻量。 可在项目之间嵌套、重复使用和共享。

## <a name="component-classes"></a>组件类

组件是使用 C# 和 HTML 标记的组合在 [Razor](xref:mvc/views/razor) 组件文件 (`.razor`) 中实现的。 Blazor 中的组件正式称为 Razor 组件。

### <a name="no-locrazor-syntax"></a>Razor 语法

Blazor 应用中的 Razor 组件广泛使用 Razor 语法。 如果你不熟悉 Razor 标记语言，建议先阅读 <xref:mvc/views/razor>，然后再继续。

访问 Razor 语法上的内容时，请特别注意以下各节：

* [指令](xref:mvc/views/razor#directives)：前缀为 `@` 的保留关键字，通常会更改组件标记的分析或运行方式。
* [指令特性](xref:mvc/views/razor#directive-attributes)：前缀为 `@` 的保留关键字，通常会更改组件元素的分析或运行方式。

### <a name="names"></a>名称

组件的名称必须以大写字符开头。 例如，`MyCoolComponent.razor` 有效，而 `myCoolComponent.razor` 无效。

### <a name="routing"></a>路由

可以通过为应用中的每个可访问组件提供路由模板来实现 Blazor 中的路由。 编译具有 [`@page`][9] 指令的 Razor 文件时，将为生成的类提供指定路由模板的 <xref:Microsoft.AspNetCore.Mvc.RouteAttribute>。 在运行时，路由器将使用 <xref:Microsoft.AspNetCore.Mvc.RouteAttribute> 查找组件类，并呈现具有与请求的 URL 匹配的路由模板的任何组件。 有关详细信息，请参阅 <xref:blazor/fundamentals/routing>。

```razor
@page "/ParentComponent"

...
```

### <a name="markup"></a>标记

使用 HTML 定义组件的 UI。 动态呈现逻辑（例如，循环、条件、表达式）是使用名为 *Razor* 的嵌入式 C# 语法添加的。 在编译应用时，HTML 标记和 C# 呈现逻辑转换为组件类。 生成的类的名称与文件名匹配。

组件类的成员在 [`@code`][1] 块中定义。 在 [`@code`][1] 块中，通过用于处理事件或定义其他组件逻辑的方法指定组件状态（属性、字段）。 允许多个 [`@code`][1] 块。

可以使用以 `@` 开头的 C# 表达式将组件成员作为组件的呈现逻辑的一部分。 例如，通过为字段名称添加 `@` 前缀来呈现 C# 字段。 下面的示例计算并呈现：

* `font-style` 的 CSS 属性值的 `headingFontStyle`。
* `<h1>` 元素内容的 `headingText`。

```razor
<h1 style="font-style:@headingFontStyle">@headingText</h1>

@code {
    private string headingFontStyle = "italic";
    private string headingText = "Put on your new Blazor!";
}
```

最初呈现组件后，组件会为响应事件而重新生成其呈现树。 然后 Blazor 将新呈现树与前一个呈现树进行比较，并对浏览器文档对象模型 (DOM) 应用任何修改。

组件是普通 C# 类，可以放置在项目中的任何位置。 生成网页的组件通常位于 `Pages` 文件夹中。 非页面组件通常放置在 `Shared` 文件夹或添加到项目的自定义文件夹中。

### <a name="namespaces"></a>命名空间

通常，组件的命名空间是从应用的根命名空间和该组件在应用内的位置（文件夹）派生而来的。 如果应用的根命名空间是 `BlazorSample`，并且 `Counter` 组件位于 `Pages` 文件夹中：

* `Counter` 组件的命名空间为 `BlazorSample.Pages`。
* 组件的完全限定类型名称为 `BlazorSample.Pages.Counter`。

对于保存组件的自定义文件夹，将 [`@using`][2] 指令添加到父组件或应用的 `_Imports.razor` 文件。 下面的示例提供 `Components` 文件夹中的组件：

```razor
@using BlazorSample.Components
```

还可以使用其完全限定的名称来引用组件，而不需要 [`@using`][2] 指令：

```razor
<BlazorSample.Components.MyComponent />
```

使用 Razor 创建的组件的命名空间基于（按优先级顺序）：

* Razor 文件 (`.razor`) 标记 (`@namespace BlazorSample.MyNamespace`) 中的 [`@namespace`][8] 指定内容。
* 项目文件 (`<RootNamespace>BlazorSample</RootNamespace>`) 中项目的 `RootNamespace`。
* 项目名称，取自项目文件的文件名 (`.csproj`)，以及从项目根到组件的路径。 例如，框架将 `{PROJECT ROOT}/Pages/Index.razor` (`BlazorSample.csproj`) 解析为命名空间 `BlazorSample.Pages`。 组件遵循 C# 名称绑定规则。 对于本示例中的 `Index` 组件，范围内的组件是所有组件：
  * 在同一文件夹 `Pages` 中。
  * 未显式指定其他命名空间的项目根中的组件。

> [!NOTE]
> 不支持 `global::` 限定。
>
> 不支持导入具有别名 [`using`](/dotnet/csharp/language-reference/keywords/using-statement) 语句的组件（例如，`@using Foo = Bar`）。
>
> 不支持部分限定名称。 例如，不支持使用 `<Shared.NavMenu></Shared.NavMenu>` 添加 `@using BlazorSample` 和引用 `NavMenu` 组件 (`NavMenu.razor`)。

### <a name="partial-class-support"></a>分部类支持

Razor 组件作为分部类生成。 使用以下方法之一创建 Razor 组件：

* 在 [`@code`][1] 块中使用单个文件中的 HTML 标记和 Razor 代码定义 C# 代码。 Blazor 模板使用此方法来定义其 Razor 组件。
* C# 代码位于定义为分部类的代码隐藏文件中。

下面的示例显示了从 Blazor 模板生成的应用中具有 [`@code`][1] 块的默认 `Counter` 组件。 HTML 标记、Razor 代码和 C# 代码位于同一个文件中：

`Pages/Counter.razor`:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;

    void IncrementCount()
    {
        currentCount++;
    }
}
```

还可以使用具有分部类的代码隐藏文件创建 `Counter` 组件：

`Pages/Counter.razor`:

```razor
@page "/counter"

<h1>Counter</h1>

<p>Current count: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Click me</button>
```

`Counter.razor.cs`:

```csharp
namespace BlazorSample.Pages
{
    public partial class Counter
    {
        private int currentCount = 0;

        void IncrementCount()
        {
            currentCount++;
        }
    }
}
```

根据需要将任何所需的命名空间添加到分部类文件中。 Razor 组件使用的典型命名空间包括：

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Authorization;
using Microsoft.AspNetCore.Components.Forms;
using Microsoft.AspNetCore.Components.Routing;
using Microsoft.AspNetCore.Components.Web;
```

> [!IMPORTANT]
> `_Imports.razor` 文件中的 [`@using`][2] 指令仅适用于 Razor 文件 (`.razor`)，而不适用于 C# 文件 (`.cs`)。

### <a name="specify-a-base-class"></a>指定基类

[`@inherits`][6] 指令可用于指定组件的基类。 下面的示例演示组件如何继承基类 `BlazorRocksBase` 以提供组件的属性和方法。 基类应派生自 <xref:Microsoft.AspNetCore.Components.ComponentBase>。

`Pages/BlazorRocks.razor`:

```razor
@page "/BlazorRocks"
@inherits BlazorRocksBase

<h1>@BlazorRocksText</h1>
```

`BlazorRocksBase.cs`:

```csharp
using Microsoft.AspNetCore.Components;

namespace BlazorSample
{
    public class BlazorRocksBase : ComponentBase
    {
        public string BlazorRocksText { get; set; } = 
            "Blazor rocks the browser!";
    }
}
```

### <a name="use-components"></a>使用组件

通过使用 HTML 元素语法声明组件，组件可以包含其他组件。 使用组件的标记类似于 HTML 标记，其中标记的名称是组件类型。

`Pages/Index.razor` 中的以下标记呈现了 `HeadingComponent` 实例：

```razor
<HeadingComponent />
```

`Components/HeadingComponent.razor`:

[!code-razor[](index/samples_snapshot/HeadingComponent.razor)]

如果某个组件包含一个 HTML 元素，该元素的大写首字母与组件名称不匹配，则会发出警告，指示该元素名称异常。 为组件的命名空间添加 [`@using`][2] 指令使组件可用，从而解决此警告。

## <a name="parameters"></a>参数

### <a name="route-parameters"></a>路由参数

组件可以接收来自 [`@page`][9] 指令所提供的路由模板的路由参数。 路由器使用路由参数来填充相应的组件参数。

`Pages/RouteParameter.razor`:

[!code-razor[](index/samples_snapshot/RouteParameter.razor?highlight=2,7-8)]

不支持可选参数，因此在前面的示例中应用了两个 [`@page`][9] 指令。 第一个指令允许导航到没有参数的组件。 第二个 [`@page`][9] 指令会接收 `{text}` 路由参数，并将值赋予 `Text` 属性。

要了解 catch-all 路由参数 `{*pageRoute}`（它可跨多个文件夹边界捕获路径），请参阅 <xref:blazor/fundamentals/routing#catch-all-route-parameters>。

### <a name="component-parameters"></a>组件参数

组件可以有组件参数，这些参数是使用组件类中包含 [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) 特性的公共属性定义的。 使用这些属性在标记中为组件指定参数。

`Components/ChildComponent.razor`:

[!code-razor[](../common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=2,11-12)]

在示例应用的以下示例中，`ParentComponent` 设置 `ChildComponent` 的 `Title` 属性的值。

`Pages/ParentComponent.razor`:

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=5-6)]

> [!WARNING]
> 请勿创建会写入其自己的组件参数的组件，而是使用私有字段。 有关详细信息，请参阅[重写参数](#overwritten-parameters)部分。

## <a name="child-content"></a>子内容

组件可以设置另一个组件的内容。 分配组件提供指定接收组件的标记之间的内容。

在下面的示例中，`ChildComponent` 具有一个表示 <xref:Microsoft.AspNetCore.Components.RenderFragment>（表示要呈现的 UI 段）的 `ChildContent` 属性。 `ChildContent` 的值放置在应呈现内容的组件标记中。 `ChildContent` 的值是从父组件接收的，并呈现在启动面板的 `panel-body` 中。

`Components/ChildComponent.razor`:

[!code-razor[](../common/samples/3.x/BlazorWebAssemblySample/Components/ChildComponent.razor?highlight=3,14-15)]

> [!NOTE]
> 必须按约定将接收 <xref:Microsoft.AspNetCore.Components.RenderFragment> 内容的属性命名为 `ChildContent`。

示例应用中的 `ParentComponent` 可以通过将内容置于 `<ChildComponent>` 标记中，提供用于呈现 `ChildComponent` 的内容。

`Pages/ParentComponent.razor`:

[!code-razor[](index/samples_snapshot/ParentComponent.razor?highlight=7-8)]

由于 Blazor 呈现子内容的方式，如果在子组件的内容中使用递增循环变量，则在 `for` 循环内呈现组件需要本地索引变量：
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <ChildComponent Param1="@c">
>         Child Content: Count: @current
>     </ChildComponent>
> }
> ```
>
> 或者，将 `foreach` 循环用于 <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType>：
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <ChildComponent Param1="@c">
>         Child Content: Count: @c
>     </ChildComponent>
> }
> ```

## <a name="attribute-splatting-and-arbitrary-parameters"></a>属性展开和任意参数

除了组件的声明参数外，组件还可以捕获和呈现其他属性。 其他属性可以在字典中捕获，然后在使用 [`@attributes`][3] Razor 指令呈现组件时，将其展开到元素上。 此方案在定义生成支持各种自定义项的标记元素的组件时非常有用。 例如，为支持多个参数的 `<input>` 单独定义属性可能比较繁琐。

在下面的示例中，第一个 `<input>` 元素 (`id="useIndividualParams"`) 使用单个组件参数，第二个 `<input>` 元素 (`id="useAttributesDict"`) 使用属性展开：

```razor
<input id="useIndividualParams"
       maxlength="@maxlength"
       placeholder="@placeholder"
       required="@required"
       size="@size" />

<input id="useAttributesDict"
       @attributes="InputAttributes" />

@code {
    public string maxlength = "10";
    public string placeholder = "Input placeholder text";
    public string required = "required";
    public string size = "50";

    public Dictionary<string, object> InputAttributes { get; set; } =
        new Dictionary<string, object>()
        {
            { "maxlength", "10" },
            { "placeholder", "Input placeholder text" },
            { "required", "required" },
            { "size", "50" }
        };
}
```

参数的类型必须使用字符串键实现 `IEnumerable<KeyValuePair<string, object>>` 或 `IReadOnlyDictionary<string, object>`。

使用这两种方法呈现的 `<input>` 元素相同：

```html
<input id="useIndividualParams"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">

<input id="useAttributesDict"
       maxlength="10"
       placeholder="Input placeholder text"
       required="required"
       size="50">
```

若要接受任意特性，请使用 [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) 特性定义组件参数，并将 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> 属性设置为 `true`：

```razor
@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public Dictionary<string, object> InputAttributes { get; set; }
}
```

借助 [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) 上的 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> 属性，参数可以匹配所有不匹配其他任何参数的特性。 组件只能使用 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> 定义单个参数。 与 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> 一起使用的属性类型必须可以使用字符串键从 `Dictionary<string, object>` 中分配。 `IEnumerable<KeyValuePair<string, object>>` 或 `IReadOnlyDictionary<string, object>` 也是此方案中的选项。

相对于元素特性位置的 [`@attributes`][3] 位置很重要。 在元素上展开 [`@attributes`][3] 时，将从右到左（从最后一个到第一个）处理特性。 请考虑以下使用 `Child` 组件的组件示例：

`ParentComponent.razor`:

```razor
<ChildComponent extra="10" />
```

`ChildComponent.razor`:

```razor
<div @attributes="AdditionalAttributes" extra="5" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

`Child` 组件的 `extra` 属性设置为 [`@attributes`][3] 右侧。 通过附加特性传递时，`Parent` 组件的呈现的 `<div>` 包含 `extra="5"`，因为特性是从右到左（从最后一个到第一个）处理的：

```html
<div extra="5" />
```

在下面的示例中，`extra` 和 [`@attributes`][3] 的顺序在 `Child` 组件的 `<div>` 中反转：

`ParentComponent.razor`:

```razor
<ChildComponent extra="10" />
```

`ChildComponent.razor`:

```razor
<div extra="5" @attributes="AdditionalAttributes" />

[Parameter(CaptureUnmatchedValues = true)]
public IDictionary<string, object> AdditionalAttributes { get; set; }
```

通过附加属性传递时，`Parent` 组件中呈现的 `<div>` 包含 `extra="10"`：

```html
<div extra="10" />
```

## <a name="capture-references-to-components"></a>捕获对组件的引用

组件引用提供了一种引用组件实例的方法，以便可以向该实例发出命令，例如 `Show` 或 `Reset`。 若要捕获组件引用，请执行以下操作：

* 向子组件添加 [`@ref`][4] 特性。
* 定义与子组件类型相同的字段。

```razor
<CustomLoginDialog @ref="loginDialog" ... />

@code {
    private CustomLoginDialog loginDialog;

    private void OnSomething()
    {
        loginDialog.Show();
    }
}
```

呈现组件时，将用 `MyLoginDialog` 子组件实例填充 `loginDialog` 字段。 然后，可以在组件实例上调用 .NET 方法。

> [!IMPORTANT]
> 仅在呈现组件后填充 `loginDialog` 变量，其输出包含 `MyLoginDialog` 元素。 在呈现组件之前，没有任何可引用的内容。
>
> 若要在组件完成呈现后操作组件引用，请使用 [`OnAfterRenderAsync` 或 `OnAfterRender` 方法](xref:blazor/components/lifecycle#after-component-render)。
>
> 若要结合使用事件处理程序和引用变量，请使用 Lambda 表达式，或在 [`OnAfterRenderAsync` 或 `OnAfterRender` 方法](xref:blazor/components/lifecycle#after-component-render)中分配事件处理程序委托。 这可确保在分配事件处理程序之前先分配引用变量。
>
> ```razor
> <button type="button" 
>     @onclick="@(() => loginDialog.DoSomething())">Do Something</button>
>
> <MyLoginDialog @ref="loginDialog" ... />
>
> @code {
>     private MyLoginDialog loginDialog;
> }
> ```

要引用循环中的组件，请参阅[捕获对多个相似子组件的引用 (dotnet/aspnetcore #13358)](https://github.com/dotnet/aspnetcore/issues/13358)。

尽管捕获组件引用使用与[捕获元素引用](xref:blazor/call-javascript-from-dotnet#capture-references-to-elements)类似的语法，但它不是 JavaScript 互操作功能。 组件引用不会传递给 JavaScript 代码。 组件引用只在 .NET 代码中使用。

> [!NOTE]
> 不要使用组件引用来改变子组件的状态。 请改用常规声明性参数将数据传递给子组件。 使用常规声明性参数使子组件在正确的时间自动重新呈现。

## <a name="synchronization-context"></a>同步上下文

Blazor 使用同步上下文 (<xref:System.Threading.SynchronizationContext>) 来强制执行单个逻辑线程。 组件的[生命周期方法](xref:blazor/components/lifecycle)和 Blazor 引发的任何事件回调都在此同步上下文上执行。

Blazor Server的同步上下文尝试模拟单线程环境，使其与浏览器中的单线程 WebAssembly 模型紧密匹配。 在任意给定的时间点，工作只在一个线程上执行，从而造成单个逻辑线程的印象。 不会同时执行两个操作。

### <a name="avoid-thread-blocking-calls"></a>避免阻止线程的调用

通常，不要调用以下方法。 以下方法阻止线程，进而阻止应用继续工作，直到基础 <xref:System.Threading.Tasks.Task> 完成：

* <xref:System.Threading.Tasks.Task%601.Result%2A>
* <xref:System.Threading.Tasks.Task.Wait%2A>
* <xref:System.Threading.Tasks.Task.WaitAny%2A>
* <xref:System.Threading.Tasks.Task.WaitAll%2A>
* <xref:System.Threading.Thread.Sleep%2A>
* <xref:System.Runtime.CompilerServices.TaskAwaiter.GetResult%2A>

### <a name="invoke-component-methods-externally-to-update-state"></a>在外部调用组件方法以更新状态

如果组件必须根据外部事件（如计时器或其他通知）进行更新，请使用 `InvokeAsync` 方法来调度到 Blazor 的同步上下文。 例如，假设有一个可通知任何侦听组件更新状态的通告程序服务：

```csharp
public class NotifierService
{
    // Can be called from anywhere
    public async Task Update(string key, int value)
    {
        if (Notify != null)
        {
            await Notify.Invoke(key, value);
        }
    }

    public event Func<string, int, Task> Notify;
}
```

注册 `NotifierService`：

* 在 Blazor WebAssembly 中，在 `Program.Main` 中将服务注册为单一实例：

  ```csharp
  builder.Services.AddSingleton<NotifierService>();
  ```

* 在 Blazor Server 中，在 `Startup.ConfigureServices` 中将服务注册为有作用域：

  ```csharp
  services.AddScoped<NotifierService>();
  ```

使用 `NotifierService` 更新组件：

```razor
@page "/"
@inject NotifierService Notifier
@implements IDisposable

<p>Last update: @lastNotification.key = @lastNotification.value</p>

@code {
    private (string key, int value) lastNotification;

    protected override void OnInitialized()
    {
        Notifier.Notify += OnNotify;
    }

    public async Task OnNotify(string key, int value)
    {
        await InvokeAsync(() =>
        {
            lastNotification = (key, value);
            StateHasChanged();
        });
    }

    public void Dispose()
    {
        Notifier.Notify -= OnNotify;
    }
}
```

在前面的示例中，`NotifierService` 在 Blazor 的同步上下文之外调用组件的 `OnNotify` 方法。 `InvokeAsync` 用于切换到正确的上下文，并将呈现排入队列。

## <a name="use-key-to-control-the-preservation-of-elements-and-components"></a>使用 \@ 键控制是否保留元素和组件

在呈现元素或组件的列表并且元素或组件随后发生更改时，Blazor 的比较算法必须决定之前的元素或组件中有哪些可以保留，以及模型对象应如何映射到这些元素或组件。 通常，此过程是自动的，可以忽略，但在某些情况下，可能需要控制该过程。

请看下面的示例：

```csharp
@foreach (var person in People)
{
    <DetailsEditor Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

`People` 集合的内容可能会随插入、删除或重新排序的条目而更改。 当组件重新呈现时，`<DetailsEditor>` 组件可能会更改以接收不同的 `Details` 参数值。 这可能导致重新呈现比预期更复杂。 在某些情况下，重新呈现可能会导致可见行为差异，如失去元素焦点。

可通过 [`@key`][5] 指令属性来控制映射过程。 [`@key`][5] 使比较算法保证基于键的值保留元素或组件：

```csharp
@foreach (var person in People)
{
    <DetailsEditor @key="person" Details="@person.Details" />
}

@code {
    [Parameter]
    public IEnumerable<Person> People { get; set; }
}
```

当 `People` 集合发生更改时，比较算法会保留 `<DetailsEditor>` 实例与 `person` 实例之间的关联：

* 如果从 `People` 列表中删除 `Person`，则仅从 UI 中删除相应的 `<DetailsEditor>` 实例。 其他实例保持不变。
* 如果在列表中的某个位置插入 `Person`，则会在相应位置插入一个新的 `<DetailsEditor>` 实例。 其他实例保持不变。
* 如果对 `Person` 项进行重新排序，则保留相应的 `<DetailsEditor>` 实例，并在 UI 中重新排序。

在某些场景中，使用 [`@key`][5] 可最大程度降低重新呈现的复杂性，并避免在 DOM 的有状态部分发生变化时出现潜在问题，如焦点位置。

> [!IMPORTANT]
> 对于每个容器元素或组件而言，键是本地的。 不会在整个文档中全局地比较键。

### <a name="when-to-use-key"></a>何时使用 \@key

通常，每当列表呈现（例如，在 [foreach](/dotnet/csharp/language-reference/keywords/foreach-in) 块中）且存在合适的值来定义 [`@key`][5] 时，最好使用 [`@key`][5]。

还可以使用 [`@key`][5] 来防止 Blazor 在对象发生更改时保留元素或组件子树：

```razor
<div @key="currentPerson">
    ... content that depends on currentPerson ...
</div>
```

如果 `@currentPerson` 更改，则 [`@key`][5] 属性指令强制 Blazor 放弃整个 `<div>` 及其子代，并利用新的元素和组件在 UI 中重新生成子树。 如果需要确保在 `@currentPerson` 更改时不保留 UI 状态，这会很有用。

### <a name="when-not-to-use-key"></a>何时不使用 \@key

与 [`@key`][5] 进行比较时，会产生性能成本。 性能成本不是很大，但仅在控制元素或组件保留规则对应用有利的情况下才指定 [`@key`][5]。

即使未使用 [`@key`][5]，Blazor 也会尽可能地保留子元素和组件实例。 使用 [`@key`][5] 的唯一优点是控制如何将模型实例映射到保留的组件实例，而不是选择映射的比较算法。

### <a name="what-values-to-use-for-key"></a>\@key 使用哪些值

通常，为 [`@key`][5] 提供以下类型之一的值：

* 模型对象实例（例如，先前示例中的 `Person` 实例）。 这可确保基于对象引用相等性的保留。
* 唯一标识符（例如，`int`、`string` 或 `Guid` 类型的主键值）。

确保用于 [`@key`][5] 的值不冲突。 如果在同一父元素内检测到冲突值，则 Blazor 引发异常，因为它无法明确地将旧元素或组件映射到新元素或组件。 仅使用非重复值，例如对象实例或主键值。

## <a name="overwritten-parameters"></a>重写参数

在父组件重新呈现时提供了新的参数值，这通常会重写现有参数。

请考虑使用以下 `Expander` 组件，它们会：

* 呈现子内容。
* 切换以使用组件参数显示子内容。

```razor
<div @onclick="@Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>Expanded</code> = @Expanded)</h2>

        @if (Expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    private void Toggle()
    {
        Expanded = !Expanded;
    }
}
```

`Expander` 组件会添加到可调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>的父组件中：

```razor
@page "/expander"

<Expander Expanded="true">
    Expander 1 content
</Expander>

<Expander Expanded="true" />

<button @onclick="StateHasChanged">
    Call StateHasChanged
</button>
```

最初，在切换 `Expanded` 属性时，`Expander` 组件独立地作出行为。 子组件会按预期方式维护其状态。 在父组件中调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> 时，第一个子组件的 `Expanded` 会重新重置为它的初始值 (`true`)。 第二个 `Expander` 组件的 `Expanded` 值不会重置，因为第二个组件中没有呈现任何子内容。

要维持在前述情况中的状态，请在 `Expander` 组件中使用私有字段来保留它的切换状态。

以下经修定的 `Expander` 组件：

* 接受父项中的 `Expanded` 组件参数值。
* 将组件参数值分配给 [OnInitialized 事件](xref:blazor/components/lifecycle#component-initialization-methods)中的私有字段 (`expanded`)。
* 使用私有字段来维持它的内部切换状态。

```razor
<div @onclick="@Toggle" class="card bg-light mb-3" style="width:30rem">
    <div class="card-body">
        <h2 class="card-title">Toggle (<code>expanded</code> = @expanded)</h2>

        @if (expanded)
        {
            <p class="card-text">@ChildContent</p>
        }
    </div>
</div>

@code {
    private bool expanded;

    [Parameter]
    public bool Expanded { get; set; }

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    protected override void OnInitialized()
    {
        expanded = Expanded;
    }

    private void Toggle()
    {
        expanded = !expanded;
    }
}
```

## <a name="apply-an-attribute"></a>应用属性

可以通过 [`@attribute`][7] 指令在 Razor 组件中应用属性。 下面的示例将 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) 特性应用于组件类：

```razor
@page "/"
@attribute [Authorize]
```

## <a name="conditional-html-element-attributes"></a>条件 HTML 元素属性

HTML 元素属性基于 .NET 值有条件地呈现。 如果值为 `false` 或 `null`，则不会呈现属性。 如果值为 `true`，则以最小化呈现属性。

在下面的示例中，`IsCompleted` 确定是否在元素的标记中呈现 `checked`：

```razor
<input type="checkbox" checked="@IsCompleted" />

@code {
    [Parameter]
    public bool IsCompleted { get; set; }
}
```

如果 `IsCompleted` 为 `true`，则复选框呈现为：

```html
<input type="checkbox" checked />
```

如果 `IsCompleted` 为 `false`，则复选框呈现为：

```html
<input type="checkbox" />
```

有关详细信息，请参阅 <xref:mvc/views/razor>。

> [!WARNING]
> .NET 类型为 `bool` 时，某些 HTML 属性（如 [`aria-pressed`](https://developer.mozilla.org/docs/Web/Accessibility/ARIA/Roles/button_role#Toggle_buttons)）无法正常运行。 在这些情况下，请使用 `string` 类型，而不是 `bool`。

## <a name="raw-html"></a>原始 HTML

通常使用 DOM 文本节点呈现字符串，这意味着将忽略它们可能包含的任何标记，并将其视为文字文本。 若要呈现原始 HTML，请将 HTML 内容包装在 `MarkupString` 值中。 将该值分析为 HTML 或 SVG，并插入到 DOM 中。

> [!WARNING]
> 呈现从任何不受信任的源构造的原始 HTML 存在安全风险，应避免！

下面的示例演示如何使用 `MarkupString` 类型向组件的呈现输出添加静态 HTML 内容块：

```html
@((MarkupString)myMarkup)

@code {
    private string myMarkup = 
        "<p class='markup'>This is a <em>markup string</em>.</p>";
}
```

## <a name="no-locrazor-templates"></a>Razor 模板

可以使用 Razor 模板语法来定义呈现片段。 Razor 模板是一种定义 UI 代码片段的方法，请使用以下格式：

```razor
@<{HTML tag}>...</{HTML tag}>
```

下面的示例演示如何在组件中指定 <xref:Microsoft.AspNetCore.Components.RenderFragment> 和 <xref:Microsoft.AspNetCore.Components.RenderFragment%601> 值并直接呈现模板。 还可以将呈现片段作为参数传递给[模板化组件](xref:blazor/components/templated-components)。

```razor
@timeTemplate

@petTemplate(new Pet { Name = "Rex" })

@code {
    private RenderFragment timeTemplate = @<p>The time is @DateTime.Now.</p>;
    private RenderFragment<Pet> petTemplate = (pet) => @<p>Pet: @pet.Name</p>;

    private class Pet
    {
        public string Name { get; set; }
    }
}
```

以上代码的呈现输出：

```html
<p>The time is 10/04/2018 01:26:52.</p>

<p>Pet: Rex</p>
```

## <a name="static-assets"></a>静态资产

Blazor 遵循 ASP.NET Core 应用的约定，将静态资产放在项目的 [`web root (wwwroot)` 文件夹](xref:fundamentals/index#web-root)下。

使用基相对路径 (`/`) 来引用静态资产的 Web 根。 在下面的示例中，`logo.png` 实际位于 `{PROJECT ROOT}/wwwroot/images` 文件夹中：

```razor
<img alt="Company logo" src="/images/logo.png" />
```

Razor 组件不支持波浪符斜杠表示法 (`~/`)。

有关设置应用基路径的信息，请参阅 <xref:blazor/host-and-deploy/index#app-base-path>。

## <a name="tag-helpers-arent-supported-in-components"></a>组件中不支持标记帮助程序

Razor 组件（`.razor` 文件）不支持 [`Tag Helpers`](xref:mvc/views/tag-helpers/intro)。 若要在 Blazor 中提供类似标记帮助程序的功能，请创建一个具有与标记帮助程序相同功能的组件，并改为使用该组件。

## <a name="scalable-vector-graphics-svg-images"></a>可缩放的向量图形 (SVG) 图像

由于 Blazor 呈现 HTML，因此通过 `<img>` 标记支持浏览器支持的图像，包括可缩放的矢量图形 (SVG) 图像(`.svg`)：

```html
<img alt="Example image" src="some-image.svg" />
```

同样，样式表文件 (`.css`) 的 CSS 规则支持 SVG 图像：

```css
.my-element {
    background-image: url("some-image.svg");
}
```

但是，并非在所有情况下都支持内联的 SVG 标记。 如果将 `<svg>` 标记直接放入组件文件 (`.razor`)，则支持基本图像呈现，但很多高级场景尚不受支持。 例如，当前未遵循 `<use>` 标记，并且 [`@bind`][10] 不能与某些 SVG 标记一起使用。 有关详细信息，请参阅 [Blazor (dotnet/aspnetcore #18271) 中的 SVG 支持](https://github.com/dotnet/aspnetcore/issues/18271)。

## <a name="additional-resources"></a>其他资源

* <xref:blazor/security/server/threat-mitigation>：包括有关如何生成必须应对资源耗尽的 Blazor Server应用的指南。

<!--Reference links in article-->
[1]: <xref:mvc/views/razor#code>
[2]: <xref:mvc/views/razor#using>
[3]: <xref:mvc/views/razor#attributes>
[4]: <xref:mvc/views/razor#ref>
[5]: <xref:mvc/views/razor#key>
[6]: <xref:mvc/views/razor#inherits>
[7]: <xref:mvc/views/razor#attribute>
[8]: <xref:mvc/views/razor#namespace>
[9]: <xref:mvc/views/razor#page>
[10]: <xref:mvc/views/razor#bind>
