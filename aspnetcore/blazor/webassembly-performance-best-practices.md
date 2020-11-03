---
title: ASP.NET Core Blazor WebAssembly 性能最佳做法
author: pranavkm
description: 用于提高 ASP.NET Core Blazor WebAssembly 应用性能并避免常见性能问题的提示。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 10/09/2020
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
uid: blazor/webassembly-performance-best-practices
ms.openlocfilehash: 0e827680e7024eabed09b989466476a3a80eb225
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690276"
---
# <a name="aspnet-core-no-locblazor-webassembly-performance-best-practices"></a>ASP.NET Core Blazor WebAssembly 性能最佳做法

作者：[Pranav Krishnamoorthy](https://github.com/pranavkm) 和 [Steve Sanderson](https://github.com/SteveSandersonMS)

Blazor WebAssembly 经过精心设计和优化，可在最真实的应用程序 UI 方案中提高性能。 但是，仅当开发人员使用正确的模式和功能时才会产生最佳结果。 请考虑以下方面：

* 运行时吞吐量：.NET 代码在 WebAssembly 运行时内在解释器上运行，因此 CPU 吞吐量会受到限制。 在要求苛刻的方案中，应用受益于[优化呈现速度](#optimize-rendering-speed)。
* 启动时间：应用会将 .NET 运行时传输到浏览器，因此，使用[最小化应用程序下载大小](#minimize-app-download-size)的功能非常重要。

## <a name="optimize-rendering-speed"></a>优化呈现速度

以下部分提供最小化呈现工作负载和提升 UI 响应能力的建议。 遵循此建议可以轻松地以 UI 呈现速度提升 10 倍或更高。

### <a name="avoid-unnecessary-rendering-of-component-subtrees"></a>避免不必要地呈现组件子树

在运行时，组件作为层次结构存在。 根组件具有子组件。 反过来，根的子项也有其自己的子组件，依此类推。 发生事件（例如用户选择某个按钮）时，这就是 Blazor 如何决定要重新呈现哪些组件：

 1. 事件本身将被调度到呈现事件处理程序的任何组件。 执行事件处理程序后，将重新呈现该组件。
 1. 每当重新呈现任何组件时，都会向其每个子组件提供参数值的新副本。
 1. 接收一组新的参数值时，每个组件都会选择是否要重新呈现。 默认情况下，如果参数值可能已更改（例如，如果它们是可变对象），则组件重新呈现。

此序列的最后两个步骤以递归方式沿着组件层次结构继续向下。 在许多情况下，将重新呈现整个子树。 这意味着，针对高级组件的事件可能会导致成本高昂的重新呈现过程，因为必须重新呈现该点之下的所有内容。

如果要中断此过程并防止将递归呈现到特定子树中，则可以执行以下任一操作：

 * 确保某个组件的所有参数均属于基元不可变类型（例如，`string`、`int`、`bool`、`DateTime` 等其他类型）。 如果这些参数值均未更改，则用于检测更改的内置逻辑会自动跳过重新呈现。 如果使用 `<Customer CustomerId="@item.CustomerId" />` 呈现子组件（其中 `CustomerId` 为 `int` 值），则除非 `item.CustomerId` 更改，否则不会重新呈现。
 * 如果需要接受非基元参数值（如自定义模型类型、事件回调或 <xref:Microsoft.AspNetCore.Components.RenderFragment> 值），则可以重写 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 以控制有关是否呈现的决策，[使用 `ShouldRender`](#use-of-shouldrender) 部分中对此进行了介绍。

通过跳过重新呈现整个子树，当发生事件时，可以删除绝大多数的呈现开销。

你可能想要专门分解出子组件，以便可以跳过重新呈现 UI 的此部分。 这是降低父组件的呈现开销的有效方法。

#### <a name="use-of-shouldrender"></a>使用 `ShouldRender`

如果创作了一个仅限 UI 的组件，且该组件在最初呈现后从未更改（无论使用哪个参数值），则请将 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 配置为返回 `false`：

```razor
@code {
    protected override bool ShouldRender() => false;
}
```

如果组件只需在其参数值以特定方式改变时重新呈现，则可以使用私有字段跟踪必要的信息来检测更改。 在下面的示例中，`shouldRender` 基于检查结果，检查是否有任何类型的应提示重新呈现的更改或转变。 `prevOutboundFlightId` 和 `prevInboundFlightId` 跟踪下一个可能更新的信息：

```razor
@code {
    [Parameter]
    public FlightInfo OutboundFlight { get; set; }
    
    [Parameter]
    public FlightInfo InboundFlight { get; set; }

    private int prevOutboundFlightId;
    private int prevInboundFlightId;
    private bool shouldRender;

    protected override void OnParametersSet()
    {
        shouldRender = OutboundFlight.FlightId != prevOutboundFlightId
            || InboundFlight.FlightId != prevInboundFlightId;

        prevOutboundFlightId = OutboundFlight.FlightId;
        prevInboundFlightId = InboundFlight.FlightId;
    }

    protected override void ShouldRender() => shouldRender;

    // Note that 
}
```

在前面的代码中，事件处理程序还可以将 `shouldRender` 设置为 `true`，以便在事件之后重新呈现组件。

大多数组件不需要此级别的手动控制。 只应考虑跳过呈现子树（如果这些子树的呈现费用非常昂贵并导致 UI 延迟）。

有关详细信息，请参阅 <xref:blazor/components/lifecycle>。

::: moniker range=">= aspnetcore-5.0"

### <a name="virtualization"></a>虚拟化

在循环中呈现大量 UI（例如，具有数千个条目的列表或网格）时，呈现操作的极大数量可能会导致 UI 呈现延迟，从而导致用户体验不佳。 假设用户只能在不滚动的情况下同时查看少量元素，则花费太多时间呈现当前不可见的元素似乎太浪费了。

为了解决此问题，Blazor 提供内置 [`<Virtualize>` 组件](xref:blazor/components/virtualization)，该组件可创建任意规模的列表的外观和滚动行为，但实际上只会呈现当前滚动视区内的列表项。 例如，这意味着应用可以有一个包含 100,000 个条目的列表，但每次只需支付 20 个可见项的费用。 使用 `<Virtualize>` 组件可以按数量级提高 UI 性能。

`<Virtualize>` 可在以下情况下使用：

 * 在循环中呈现一组数据项。
 * 由于滚动，大多数项不可见。
 * 呈现的项的大小完全相同。 当用户滚动到任意点时，组件可计算要显示的可见项。

下面显示了非虚拟化列表的示例：

```razor
<div class="all-flights" style="height:500px;overflow-y:scroll">
    @foreach (var flight in allFlights)
    {
        <FlightSummary @key="flight.FlightId" Flight="@flight" />
    }
</div>
```

如果 `allFlights` 集合包含 10,000 个项，则会实例化并呈现 10,000 个 `<FlightSummary>` 组件实例。 相比之下，下面显示了非虚拟化列表的示例：

```razor
<div class="all-flights" style="height:500px;overflow-y:scroll">
    <Virtualize Items="@allFlights" Context="flight">
        <FlightSummary @key="flight.FlightId" Flight="@flight" />
    </Virtualize>
</div>
```

即使生成的 UI 看起来与用户相同，组件在后台也仅实例化并呈现填充可滚动区域所需数量的 `<FlightSummary>` 实例。 当用户滚动时，将重新计算并呈现显示的 `<FlightSummary>` 实例集。

`<Virtualize>` 也有其他优点。 例如，当组件请求外部 API 中的数据时，`<Virtualize>` 允许组件仅提取与当前可见区域相对应的记录切片，而不是下载集合中的所有数据。

有关详细信息，请参阅 <xref:blazor/components/virtualization>。

::: moniker-end

### <a name="create-lightweight-optimized-components"></a>创建轻型且优化的组件

大多数 Blazor 组件不需要强硬的优化工作。 这是因为大多数组件并不经常在 UI 中重复，也不会高频率重新呈现。 例如，`@page` 组件和表示高级 UI 块（如对话框或窗体）的组件在大多数情况下一次仅显示一个，并且仅为了响应用户手势而重新呈现。 这些组件不会创建较高的呈现工作负载，因此你可以自由地使用所需的任何框架功能组合，而无需担心呈现性能。

但是，还有生成需要大规模重复的组件的常见情况。 例如：

 * 大型嵌套窗体可能包含数百个单个输入、标签和其他元素。
 * 网格可能包含数千个单元格。
 * 散点图可能包含数百万个数据点。

如果将每个单位建模为单独的组件实例，则会有很多呈现性能变得至关重要的实例。 本部分提供了有关使此类组件变得轻量，以便 UI 保持快速且响应迅速的建议。

#### <a name="avoid-thousands-of-component-instances"></a>避免上千个组件实例

每个组件都是单独的，可以独立于其父组件和子组件进行呈现。 通过选择如何将 UI 拆分为组件的层次结构，你将控制 UI 呈现的粒度。 这可能导致性能良好或不佳。

 * 将 UI 拆分为多个组件后，当发生事件时，可以有较小部分的 UI 重新呈现。 例如，当用户单击表行中的某个按钮时，你可能能够仅重新呈现单行，而不是整页或整个表。
 * 但是，每个额外组件都包含一些额外内存和 CPU 开销，用于处理其独立状态和呈现生命周期。

优化 .NET 5 上的 Blazor WebAssembly 性能时，我们计算了每个组件实例大约 0.06 毫秒的呈现开销。 这基于接受在典型便携式计算机上运行的三个参数的简单组件。 在内部，开销很大程度上取决于从字典中检索每个组件的状态以及传递和接收参数。 通过成倍增加，你可以看到添加 2,000 个额外的组件实例会使呈现时间增加 0.12 秒，并且用户会觉得 UI 开始变得缓慢。

可以使组件变得更轻量，以便你可以拥有更多组件，但功能更强大的技术通常不会有如此多的组件。 以下部分将介绍两种方法。

##### <a name="inline-child-components-into-their-parents"></a>将子组件嵌入到其父组件中

请考虑呈现一系列子组件的以下组件：

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <ChatMessageDisplay Message="@message" />
    }
</div>
```

对于前面的示例代码，`<ChatMessageDisplay>` 组件在文件 `ChatMessageDisplay.razor` 中定义，其中包含：

```razor
<div class="chat-message">
    <span class="author">@Message.Author</span>
    <span class="text">@Message.Text</span>
</div>

@code {
    [Parameter]
    public ChatMessage Message { get; set; }
}
```

只要一次不显示数千条消息，前面的示例就可以正常工作且性能良好。 若要一次显示数千条消息，请考虑不分解出单独的 `ChatMessageDisplay` 组件。 而改为将呈现直接内联到父组件：

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    }
</div>
```

这样可以避免因呈现如此多的子组件，但无法单独重新呈现每个子组件而导致的每组件开销。

##### <a name="define-reusable-renderfragments-in-code"></a>在代码中定义可重用的 `RenderFragments`

你可能会分解出子组件，纯粹作为重复使用呈现逻辑的方法。 如果是这种情况，则仍可以重复使用呈现逻辑，而无需声明实际组件。 在任何组件的 `@code` 块中，你都可以定义发出 UI 并可从任何位置调用的 <xref:Microsoft.AspNetCore.Components.RenderFragment>：

```razor
<h1>Hello, world!</h1>

@RenderWelcomeInfo

@code {
    RenderFragment RenderWelcomeInfo = __builder =>
    {
        <div>
            <p>Welcome to your new app!</p>

            <SurveyPrompt Title="How is Blazor working for you?" />
        </div>
    };
}
```

如前面的示例中所述，组件可通过其 `@code` 块内外的代码发出标记。 此方法可在多处定义可在组件的常规呈现输出内呈现的 <xref:Microsoft.AspNetCore.Components.RenderFragment> 委托。 委托必须接受名为 `__builder`、类型为 <xref:Microsoft.AspNetCore.Components.Rendering.RenderTreeBuilder> 的参数，以便 Razor 编译器可以为其生成呈现说明。

如果要使其可在多个组件之间重复使用，请考虑将其声明为 `public static` 成员：

```razor
public static RenderFragment SayHello = __builder =>
{
    <h1>Hello!</h1>
};
```

现在可以从不相关的组件调用该委托。 此方法可用于生成无需任何每组件开销即可呈现的可重用标记段库。

<xref:Microsoft.AspNetCore.Components.RenderFragment> 委托也可以接受参数。 若要创建上述示例中的 `ChatMessageDisplay` 组件的等效项，请执行以下操作：

```razor
<div class="chat">
    @foreach (var message in messages)
    {
        @DisplayChatMessage(message)
    }
</div>

@code {
    RenderFragment<ChatMessage> DisplayChatMessage = message => __builder =>
    {
        <div class="chat-message">
            <span class="author">@message.Author</span>
            <span class="text">@message.Text</span>
        </div>
    };
}
```

此方法提供了无需每组件开销即可重复使用呈现逻辑的好处。 但是，既无法单独刷新 UI 的子树，也无法在呈现 UI 的父树时跳过呈现其子树，因为没有组件边界。

#### <a name="dont-receive-too-many-parameters"></a>不接收太多参数

如果某个组件极频繁地重复（例如，数百或数千次），请记住，会产生传递和接收每个参数的开销。

很少有过多参数严格地限制性能，但这可能是一个因素。 对于在网格内呈现 1,000 次的 `<TableCell>` 组件，传递给它的每个额外参数可能会使总呈现开销增加大约 15 毫秒。 如果每个单元格接受 10 个参数，每次组件呈现的参数传递大约需要 150 毫秒，则总计可能需要 150,000 毫秒（150 秒），因此可能会导致 UI 滞后。

若要减少此负载，可以通过自定义类将多个参数捆绑在一起。 例如，`<TableCell>` 组件可能接受：

```razor
@typeparam TItem

...

@code {
    [Parameter]
    public TItem Data { get; set; }
    
    [Parameter]
    public GridOptions Options { get; set; }
}
```

在前面的示例中，每个单元格的 `Data` 都是不同的，但 `Options` 在所有单元格中都是通用的。 当然，可以改为不需要 `<TableCell>` 组件，并且将其逻辑内联到父组件中。

#### <a name="ensure-cascading-parameters-are-fixed"></a>确保级联参数是固定的

`<CascadingValue>` 组件具有名为 `IsFixed` 的可选参数。

 * 如果 `IsFixed` 值为 `false`（默认值），则级联值的每个接收方都会将订阅设置为接收更改通知。 在这种情况下，由于订阅跟踪，每个 `[CascadingParameter]` 的开销大体上都要比常规 `[Parameter]` 昂贵。
 * 如果 `IsFixed` 值为 `true`（例如，`<CascadingValue Value="@someValue" IsFixed="true">`），则接收方会接收初始值，但不会将任何订阅设置为接收更新。 在这种情况下，每个 `[CascadingParameter]` 都是轻型的，并不比常规 `[Parameter]` 昂贵。

因此，只要有可能，就应对级联值使用 `IsFixed="true"`。 只要提供的值不随时间变化，就可以执行此操作。 在组件将 `this` 作为级联值传递的常见模式下，应使用 `IsFixed="true"`：

```razor
<CascadingValue Value="this" IsFixed="true">
    <SomeOtherComponents>
</CascadingValue>
```

如果有大量其他组件接收级联值，则这会带来巨大的差异。 有关详细信息，请参阅 <xref:blazor/components/cascading-values-and-parameters>。

#### <a name="avoid-attribute-splatting-with-captureunmatchedvalues"></a>使用 `CaptureUnmatchedValues` 避免特性展开

组件可以选择使用 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues> 标志接收“不匹配”参数值：

```razor
<div @attributes="OtherAttributes">...</div>

@code {
    [Parameter(CaptureUnmatchedValues = true)]
    public IDictionary<string, object> OtherAttributes { get; set; }
}
```

此方法允许将任意附加特性传递到元素。 但是，这也相当昂贵，因为呈现器必须：

* 将所有提供的参数与用于生成字典的已知参数集匹配。
* 跟踪同一特性的多个副本彼此覆盖的方式。

可随意在非性能关键组件（如不经常重复的组件）上使用 <xref:Microsoft.AspNetCore.Components.ParameterAttribute.CaptureUnmatchedValues>。 但对于大规模呈现的组件（如大型列表中的每个项或网格中的单元格），请尝试避免特性展开。

有关详细信息，请参阅 <xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters>。

#### <a name="implement-setparametersasync-manually"></a>手动实现 `SetParametersAsync`

每组件呈现开销的主要方面之一是将传入参数值写入 `[Parameter]` 属性。 呈现器必须使用反射来实现此目的。 即使此情况已经过优化，WebAssembly 运行时上缺少 JIT 支持也会施加限制。

在某些极端情况下，你可能希望避免反射并手动实现自己的参数设置逻辑。 这可能适用于以下情况：

 * 你有极频繁地呈现的组件（例如，UI 中有数百个或数千个副本）。
 * 它接受多个参数。
 * 你会发现接收参数的开销对 UI 响应能力有明显的影响。

在这些情况下，可以重写组件的虚拟 <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 方法，并实现自己的特定于组件的逻辑。 下面的示例特意避免了任何字典查找：

```razor
@code {
    [Parameter]
    public int MessageId { get; set; }

    [Parameter]
    public string Text { get; set; }

    [Parameter]
    public EventCallback<string> TextChanged { get; set; }

    [Parameter]
    public Theme CurrentTheme { get; set; }

    public override Task SetParametersAsync(ParameterView parameters)
    {
        foreach (var parameter in parameters)
        {
            switch (parameter.Name)
            {
                case nameof(MessageId):
                    MessageId = (int)parameter.Value;
                    break;
                case nameof(Text):
                    Text = (string)parameter.Value;
                    break;
                case nameof(TextChanged):
                    TextChanged = (EventCallback<string>)parameter.Value;
                    break;
                case nameof(CurrentTheme):
                    CurrentTheme = (Theme)parameter.Value;
                    break;
                default:
                    throw new ArgumentException($"Unknown parameter: {parameter.Name}");
            }
        }

        return base.SetParametersAsync(ParameterView.Empty);
    }
}
```

在上面的代码中，返回基类 <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 会运行正常的生命周期方法，而不会再次分配参数。

正如你在前面的代码中所看到的，重写 <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 和提供自定义逻辑比较复杂且繁琐，因此，我们通常不建议使用此方法。 在极端情况下，它可以提高 20-25% 的呈现性能，但只应在前面列出的方案中考虑使用此方法。

### <a name="dont-trigger-events-too-rapidly"></a>不要过快触发事件

某些浏览器事件极频繁地触发 `onmousemove` 和 `onscroll`，每秒可以触发数十或数百次。 在大多数情况下，不需要经常执行 UI 更新。 如果尝试这样做，可能会损害 UI 响应能力或消耗过多的 CPU 时间。

你可能更想使用 JS 互操作来注册不太频繁触发的回调，而不是使用本机 `@onmousemove` 或 `@onscroll` 事件。 例如，以下组件 (`MyComponent.razor`) 显示鼠标的位置，但每 500 毫秒最多只能更新一次：

```razor
@inject IJSRuntime JS
@implements IDisposable

<h1>@message</h1>

<div @ref="myMouseMoveElement" style="border:1px dashed red;height:200px;">
    Move mouse here
</div>

@code {
    ElementReference myMouseMoveElement;
    DotNetObjectReference<MyComponent> selfReference;
    private string message = "Move the mouse in the box";

    [JSInvokable]
    public void HandleMouseMove(int x, int y)
    {
        message = $"Mouse move at {x}, {y}";
        StateHasChanged();
    }

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            selfReference = DotNetObjectReference.Create(this);
            var minInterval = 500; // Only notify every 500 ms
            await JS.InvokeVoidAsync("onThrottledMouseMove", 
                myMouseMoveElement, selfReference, minInterval);
        }
    }

    public void Dispose() => selfReference?.Dispose();
}
```

相应的 JavaScript 代码（可放置在 `index.html` 页中或作为 ES6 模块加载）注册实际 DOM 事件侦听器。 在此示例中，事件侦听器使用 [Lodash 的 `throttle` 函数](https://lodash.com/docs/4.17.15#throttle)来限制调用速率：

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.20/lodash.min.js"></script>
<script>
  function onThrottledMouseMove(elem, component, interval) {
    elem.addEventListener('mousemove', _.throttle(e => {
      component.invokeMethodAsync('HandleMouseMove', e.offsetX, e.offsetY);
    }, interval));
  }
</script>
```

此方法对于 Blazor Server 更为重要，因为每个事件调用都涉及通过网络传递消息。 它对于 Blazor WebAssembly 非常有用，因为它可以极大地减少呈现工作量。

## <a name="optimize-javascript-interop-speed"></a>优化 JavaScript 互操作速度

.NET 和 JavaScript 之间的调用涉及一些额外的开销，因为：

 * 默认情况下，调用是异步的。
 * 默认情况下，参数和返回值已进行 JSON 序列化。 这是为了在 .NET 和 JavaScript 类型之间提供一种易于理解的转换机制。

此外，在 Blazor Server 上，这些调用通过网络传递。

### <a name="avoid-excessively-fine-grained-calls"></a>避免过度细化的调用

由于每个调用都涉及一些开销，因此减少调用次数可能会有用。 请考虑以下代码，该代码在浏览器的 `localStorage` 存储中存储项的集合：

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    foreach (var item in items)
    {
        await JS.InvokeVoidAsync("localStorage.setItem", item.Id, 
            JsonSerializer.Serialize(item));
    }
}
```

前面的示例对每个项进行单独的 JS 互操作调用。 而以下方法则会将 JS 互操作减少为单个调用：

```csharp
private async Task StoreAllInLocalStorage(IEnumerable<TodoItem> items)
{
    await JS.InvokeVoidAsync("storeAllInLocalStorage", items);
}
```

相应的 JavaScript 函数定义如下：

```javascript
function storeAllInLocalStorage(items) {
  items.forEach(item => {
    localStorage.setItem(item.id, JSON.stringify(item));
  });
}
```

对于 Blazor WebAssembly，这仅在进行大量 JS 互操作调用时才有影响。

### <a name="consider-making-synchronous-calls"></a>考虑进行同步调用

默认情况下，JavaScript 互操作调用是异步的，无论调用的代码是同步还是异步。 这是为了确保组件与 Blazor WebAssembly 和 Blazor Server 兼容。 在 Blazor Server 上，所有 JavaScript 互操作调用都必须是异步的，因为它们是通过网络连接发送的。

如果你确定你的应用只在 Blazor WebAssembly 上运行，则可以选择执行同步 JavaScript 互操作调用。 与进行异步调用相比，它的开销略少，并且可能会导致呈现周期更少，因为在等待结果时没有中间状态。

若要进行从 .NET 到 JavaScript 的同步调用，请将 <xref:Microsoft.JSInterop.IJSRuntime> 转换为 <xref:Microsoft.JSInterop.IJSInProcessRuntime>：

```razor
@inject IJSRuntime JS

...

@code {
    protected override void HandleSomeEvent()
    {
        var jsInProcess = (IJSInProcessRuntime)JS;
        var value = jsInProcess.Invoke<string>("javascriptFunctionIdentifier");
    }
}
```

::: moniker range=">= aspnetcore-5.0"

使用 `IJSObjectReference` 时，可以通过转换为 `IJSInProcessObjectReference` 进行同步调用。

::: moniker-end

若要进行从 JavaScript 到 .NET 的同步调用，请使用 `DotNet.invokeMethod` 而不是 `DotNet.invokeMethodAsync`。

同步调用在以下情况下起作用：

* 应用在 Blazor WebAssembly（而不是 Blazor Server）上运行。
* 调用的函数以同步方式返回值（它不是 `async` 方法，不会返回 .NET <xref:System.Threading.Tasks.Task> 或 JavaScript `Promise`）。

有关详细信息，请参阅 <xref:blazor/call-javascript-from-dotnet>。

::: moniker range=">= aspnetcore-5.0"
 
### <a name="consider-making-unmarshalled-calls"></a>考虑进行已打乱的调用

在 Blazor WebAssembly 上运行时，可以进行从 .NET 到 JavaScript 的已打乱调用。 这些是不执行参数或返回值的 JSON 序列化的同步调用。 内存管理以及 .NET 和 JavaScript 表示形式之间的转换的所有方面均留给开发人员处理。

> [!WARNING]
> 虽然使用 `IJSUnmarshalledRuntime` 这种 JS 互操作方法的开销最小，但与这些 API 交互所需的 JavaScript API 目前没有文档记录，而且可能会在将来的版本中出现中断性变更。

```javascript
function jsInteropCall() {
    return BINDING.js_to_mono_obj("Hello world");
}
```

```razor
@inject IJSRuntime JS

@code {
    protected override void OnInitialized()
    {
        var unmarshalledJs = (IJSUnmarshalledRuntime)JS;
        var value = unmarshalledJs.InvokeUnmarshalled<string>("jsInteropCall");
    }
}
```

::: moniker-end

## <a name="minimize-app-download-size"></a>最小化应用下载大小

::: moniker range=">= aspnetcore-5.0"

### <a name="intermediate-language-il-trimming"></a>中间语言 (IL) 剪裁

[从 Blazor WebAssembly 应用修剪未使用的程序集](xref:blazor/host-and-deploy/configure-trimmer)会通过删除应用的二进制文件中的未使用代码来减小应用的大小。 默认情况下，裁边器在发布应用程序时执行。 要从剪裁中受益，请使用 [`dotnet publish`](/dotnet/core/tools/dotnet-publish) 命令发布应用用于部署，并将 [-c|--configuration](/dotnet/core/tools/dotnet-publish#options) 选项设置为 `Release`：

::: moniker-end

::: moniker range="< aspnetcore-5.0"

### <a name="intermediate-language-il-linking"></a>中间语言 (IL) 链接

通过[链接 Blazor WebAssembly 应用](xref:blazor/host-and-deploy/configure-linker)，可剪裁应用二进制文件中未使用的代码来减小应用的大小。 默认情况下，仅在 `Release` 配置中生成时才启用中间语言 (IL) 链接器。 要从此中受益，请使用 [`dotnet publish`](/dotnet/core/tools/dotnet-publish) 命令发布应用用于部署，并将 [-c|--configuration](/dotnet/core/tools/dotnet-publish#options) 选项设置为 `Release`：

::: moniker-end

```dotnetcli
dotnet publish -c Release
```

### <a name="use-systemtextjson"></a>使用 System.Text.Json

Blazor 的 JS 互操作实现依赖于 <xref:System.Text.Json> - 这是一个性能高但内存分配较低的 JSON 序列化库。 与添加一个或多个备用 JSON 库相比，使用 <xref:System.Text.Json> 不会增加应用有效负载的大小。

有关迁移指南，请参阅[如何从 `Newtonsoft.Json` 迁移到 `System.Text.Json`](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to)。

### <a name="lazy-load-assemblies"></a>延迟加载程序集

当路由需要程序集时，在运行时加载程序集。 有关详细信息，请参阅 <xref:blazor/webassembly-lazy-load-assemblies>。

### <a name="compression"></a>压缩

发布 Blazor WebAssembly 应用时，将在发布过程中对输出内容进行静态压缩，从而减小应用的大小，并免去运行时压缩的开销。 Blazor 依赖服务器来执行内容协商和提供静态压缩的文件。

部署应用后，请验证该应用是否提供压缩的文件。 检查浏览器开发人员工具中的“网络”选项卡，并验证文件是否具有 `Content-Encoding: br` 或 `Content-Encoding: gz`。 如果主机未提供压缩的文件，请按照 <xref:blazor/host-and-deploy/webassembly#compression> 中的说明操作。

### <a name="disable-unused-features"></a>禁用未使用的功能

Blazor WebAssembly 的运行时包含以下 .NET 功能；如果应用不需要这些功能就能减少有效负载的大小，可将它们禁用：

* 包含数据文件来确保时区信息正确。 如果应用不需要此功能，请考虑通过将应用项目文件中的 `BlazorEnableTimeZoneSupport` MSBuild 属性设置为 `false` 来禁用它：

  ```xml
  <PropertyGroup>
    <BlazorEnableTimeZoneSupport>false</BlazorEnableTimeZoneSupport>
  </PropertyGroup>
  ```

::: moniker range=">= aspnetcore-5.0"

* 默认情况下，Blazor WebAssembly 携带在用户区域性中显示值（如日期和货币）所需的全球化资源。 如果应用不需要本地化，你可以[将应用配置为支持不变区域性](xref:blazor/globalization-localization)，这基于 `en-US` 区域性：

  ```xml
  <PropertyGroup>
    <InvariantGlobalization>true</InvariantGlobalization>
  </PropertyGroup>
  ```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

* 包括排序规则信息来确保 <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> 之类的 API 正常工作。 如果确定应用不需要排序规则数据，请考虑通过将应用项目文件中的 `BlazorWebAssemblyPreserveCollationData` MSBuild 属性设置为 `false` 来禁用它：

  ```xml
  <PropertyGroup>
    <BlazorWebAssemblyPreserveCollationData>false</BlazorWebAssemblyPreserveCollationData>
  </PropertyGroup>
  ```

::: moniker-end
