---
title: ASP.NET Core Blazor 生命周期
author: guardrex
description: 了解如何使用 ASP.NET Core Blazor 应用中的 Razor 组件生命周期方法。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/14/2020
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
uid: blazor/components/lifecycle
ms.openlocfilehash: bf528bb68af25a8c469a0c7710abc7c0b730fce2
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690617"
---
# <a name="aspnet-core-no-locblazor-lifecycle"></a>ASP.NET Core Blazor 生命周期

作者：[Luke Latham](https://github.com/guardrex) 和 [Daniel Roth](https://github.com/danroth27)

Blazor 框架包括同步和异步生命周期方法。 替代生命周期方法，以在组件初始化和呈现期间对组件执行其他操作。

下图展示的是 Blazor 生命周期。 本文以下部分中的示例定义了生命周期方法。

组件生命周期事件：

1. 如果组件是第一次呈现在请求上：
   * 创建组件的实例。
   * 执行属性注入。 运行 `SetParametersAsync`。
   * 调用 [`OnInitialized{Async}`](#component-initialization-methods)。 如果返回 <xref:System.Threading.Tasks.Task>，则将等待 <xref:System.Threading.Tasks.Task>，然后呈现组件。 如果未返回 <xref:System.Threading.Tasks.Task>，则呈现组件。
1. 调用 [`OnParametersSet{Async}`](#after-parameters-are-set)。 如果返回 <xref:System.Threading.Tasks.Task>，则将等待 <xref:System.Threading.Tasks.Task>，然后呈现组件。 如果未返回 <xref:System.Threading.Tasks.Task>，则呈现组件。

![Blazor 中 Razor 组件的组件生命周期事件](lifecycle/_static/lifecycle1.png)

文档对象模型 (DOM) 事件处理：

1. 运行事件处理程序。
1. 如果返回 <xref:System.Threading.Tasks.Task>，则将等待 <xref:System.Threading.Tasks.Task>，然后呈现组件。 如果未返回 <xref:System.Threading.Tasks.Task>，则呈现组件。

![文档对象模型 (DOM) 事件处理](lifecycle/_static/lifecycle2.png)

`Render` 生命周期：

1. 如果这不是组件的第一个呈现或 [`ShouldRender`](#suppress-ui-refreshing) 计算为 `false`，那么请不要对该组件执行进一步的操作。
1. 生成呈现树差异并呈现组件。
1. 等待 DOM 更新。
1. 调用 [`OnAfterRender{Async}`](#after-component-render)。

![呈现生命周期](lifecycle/_static/lifecycle3.png)

开发人员调用 [`StateHasChanged`](#state-changes) 会产生呈现。

## <a name="lifecycle-methods"></a>生命周期方法

### <a name="before-parameters-are-set"></a>设置参数之前

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 在呈现树中设置组件的父组件提供的参数：

```csharp
public override async Task SetParametersAsync(ParameterView parameters)
{
    await ...

    await base.SetParametersAsync(parameters);
}
```

每次调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 时，<xref:Microsoft.AspNetCore.Components.ParameterView> 都包含该组件的参数值集。

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 的默认实现使用 [`[Parameter]`](xref:Microsoft.AspNetCore.Components.ParameterAttribute) 或 [`[CascadingParameter]`](xref:Microsoft.AspNetCore.Components.CascadingParameterAttribute) 特性（在 <xref:Microsoft.AspNetCore.Components.ParameterView> 中具有对应的值）设置每个属性的值。 在 <xref:Microsoft.AspNetCore.Components.ParameterView> 中没有对应值的参数保持不变。

如果未调用 [`base.SetParametersAync`](xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A)，则自定义代码可使用任何需要的方式解释传入的参数值。 例如，不要求将传入参数分配给类的属性。

如果设置有事件处理程序，处置时会将其解除挂接。 有关详细信息，请参阅[使用 `IDisposable` 处置组件](#component-disposal-with-idisposable)部分。

### <a name="component-initialization-methods"></a>组件初始化方法

<xref:Microsoft.AspNetCore.Components.ComponentBase.SetParametersAsync%2A> 中的组件在从其父组件接收初始参数后初始化，此时，将调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 和 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>。 

在组件执行异步操作时使用 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>，并应在操作完成后刷新。

对于同步操作，替代 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A>：

```csharp
protected override void OnInitialized()
{
    ...
}
```

若要执行异步操作，请替代 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 并对该操作使用 [`await`](/dotnet/csharp/language-reference/operators/await) 运算符：

```csharp
protected override async Task OnInitializedAsync()
{
    await ...
}
```

[预呈现其内容](xref:blazor/fundamentals/additional-scenarios#render-mode)的 Blazor Server 应用调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 两次：

* 在组件最初作为页面的一部分静态呈现时调用一次。
* 在浏览器重新建立与服务器的连接时调用第二次。

为了防止 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 中的开发人员代码运行两次，请参阅[预呈现后的有状态重新连接](#stateful-reconnection-after-prerendering)部分。

在 Blazor Server 应用进行预呈现时，由于尚未建立与浏览器的连接，无法执行调用 JavaScript 等特定操作。 预呈现时，组件可能需要进行不同的呈现。 有关详细信息，请参阅[检测应用何时预呈现](#detect-when-the-app-is-prerendering)部分。

如果设置有事件处理程序，处置时会将其解除挂接。 有关详细信息，请参阅[使用 `IDisposable` 处置组件](#component-disposal-with-idisposable)部分。

### <a name="after-parameters-are-set"></a>设置参数之后

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> 或 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSet%2A> 在以下情况下调用：

* 在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitialized%2A> 或 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 中初始化组件后。
* 当父组件重新呈现并提供以下内容时：
  * 至少一个参数已更改的唯一已知基元不可变类型。
  * 任何复杂类型的参数。 框架无法知道复杂类型参数的值是否在内部发生了改变，因此，它将参数集视为已更改。

```csharp
protected override async Task OnParametersSetAsync()
{
    await ...
}
```

> [!NOTE]
> 应用参数和属性值时，异步操作必须在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A> 生命周期事件期间发生。

```csharp
protected override void OnParametersSet()
{
    ...
}
```

如果设置有事件处理程序，处置时会将其解除挂接。 有关详细信息，请参阅[使用 `IDisposable` 处置组件](#component-disposal-with-idisposable)部分。

### <a name="after-component-render"></a>组件呈现之后

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 和 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 在组件完成呈现后调用。 此时会填充元素和组件引用。 在此阶段中，可使用呈现的内容执行其他初始化步骤，例如激活对呈现的 DOM 元素进行操作的第三方 JavaScript 库。

<xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 和 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 的 `firstRender` 参数：

* 在第一次呈现组件实例时设置为 `true`。
* 可用于确保初始化操作仅执行一次。

```csharp
protected override async Task OnAfterRenderAsync(bool firstRender)
{
    if (firstRender)
    {
        await ...
    }
}
```

> [!NOTE]
> 呈现后立即进行的异步操作必须在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 生命周期事件期间发生。
>
> 即使从 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 返回 <xref:System.Threading.Tasks.Task>，框架也不会在任务完成后为组件再安排一个呈现循环。 这是为了避免无限呈现循环。 它与其他生命周期方法不同，后者在返回的任务完成后会再安排呈现循环。

```csharp
protected override void OnAfterRender(bool firstRender)
{
    if (firstRender)
    {
        ...
    }
}
```

在服务器上的预呈现过程中，不会调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 和 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>。 在预呈现完成后以交互方式呈现组件时，将调用这些方法。 当应用预呈现时：

1. 组件将在服务器上执行，以在 HTTP 响应中生成一些静态 HTML 标记。 在此阶段，不会调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 和 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>。
1. 当 `blazor.server.js` 或 `blazor.webassembly.js` 在浏览器中启动时，组件将以交互呈现模式重新启动。 组件重新启动后，将调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 和 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A>，因为应用不再处于预呈现阶段。

如果设置有事件处理程序，处置时会将其解除挂接。 有关详细信息，请参阅[使用 `IDisposable` 处置组件](#component-disposal-with-idisposable)部分。

### <a name="suppress-ui-refreshing"></a>禁止 UI 刷新

替代 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 以禁止 UI 刷新。 如果实现返回 `true`，则刷新 UI：

```csharp
protected override bool ShouldRender()
{
    var renderUI = true;

    return renderUI;
}
```

每次呈现组件时都会调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>。

即使 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 被替代，组件也始终在最初呈现。

有关详细信息，请参阅 <xref:blazor/webassembly-performance-best-practices#avoid-unnecessary-rendering-of-component-subtrees>。

## <a name="state-changes"></a>状态更改

<xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> 通知组件其状态已更改。 如果适用，调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> 会导致组件重新呈现。

将自动为 <xref:Microsoft.AspNetCore.Components.EventCallback> 方法调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>。 有关详细信息，请参阅 <xref:blazor/components/event-handling#eventcallback>。

## <a name="handle-incomplete-async-actions-at-render"></a>处理呈现时的不完整异步操作

在呈现组件之前，在生命周期事件中执行的异步操作可能尚未完成。 执行生命周期方法时，对象可能为 `null` 或未完全填充数据。 提供呈现逻辑以确认对象已初始化。 对象为 `null` 时，呈现占位符 UI 元素（例如，加载消息）。

在 Blazor 模板的 `FetchData` 组件中，替代 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 以异步接收预测数据 (`forecasts`)。 当 `forecasts` 为 `null` 时，将向用户显示加载消息。 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 返回的 `Task` 完成后，该组件以更新后的状态重新呈现。

Blazor Server 模板中的 `Pages/FetchData.razor`：

[!code-razor[](lifecycle/samples_snapshot/FetchData.razor?highlight=9,21,25)]

## <a name="handle-errors"></a>处理错误

有关在生命周期方法执行期间处理错误的信息，请参阅 <xref:blazor/fundamentals/handle-errors#lifecycle-methods>。

## <a name="stateful-reconnection-after-prerendering"></a>预呈现后的有状态重新连接

在 Blazor Server 应用中，当 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> 为 <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> 时，组件最初作为页面的一部分静态呈现。 浏览器重新建立与服务器的连接后，将 *再次* 呈现组件，并且该组件现在为交互式。 如果存在用于初始化组件的 [`OnInitialized{Async}`](#component-initialization-methods) 生命周期方法，则该方法执行两次：

* 在静态预呈现组件时执行一次。
* 在建立服务器连接后执行一次。

在最终呈现组件时，这可能导致 UI 中显示的数据发生明显变化。

若要避免 Blazor Server 应用中出现双重呈现，请执行以下操作：

* 传递一个标识符，该标识符可用于在预呈现期间缓存状态并在应用重启后检索状态。
* 在预呈现期间使用该标识符保存组件状态。
* 预呈现后使用该标识符检索缓存的状态。

以下代码演示基于模板的 Blazor Server 应用中更新后的 `WeatherForecastService`，其避免了双重呈现：

```csharp
public class WeatherForecastService
{
    private static readonly string[] summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild",
        "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };
    
    public WeatherForecastService(IMemoryCache memoryCache)
    {
        MemoryCache = memoryCache;
    }
    
    public IMemoryCache MemoryCache { get; }

    public Task<WeatherForecast[]> GetForecastAsync(DateTime startDate)
    {
        return MemoryCache.GetOrCreateAsync(startDate, async e =>
        {
            e.SetOptions(new MemoryCacheEntryOptions
            {
                AbsoluteExpirationRelativeToNow = 
                    TimeSpan.FromSeconds(30)
            });

            var rng = new Random();

            await Task.Delay(TimeSpan.FromSeconds(10));

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = startDate.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = summaries[rng.Next(summaries.Length)]
            }).ToArray();
        });
    }
}
```

有关 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> 的详细信息，请参阅 <xref:blazor/fundamentals/additional-scenarios#render-mode>。

## <a name="detect-when-the-app-is-prerendering"></a>检测应用何时预呈现

[!INCLUDE[](~/includes/blazor-prerendering.md)]

## <a name="component-disposal-with-idisposable"></a>使用 IDisposable 处置组件

如果组件实现 <xref:System.IDisposable>，则在从 UI 中删除该组件时调用 [`Dispose` 方法](/dotnet/standard/garbage-collection/implementing-dispose)。 可随时进行处置，包括在[组件初始化](#component-initialization-methods)期间。 以下组件使用 `@implements IDisposable` 和 `Dispose` 方法：

```razor
@using System
@implements IDisposable

...

@code {
    public void Dispose()
    {
        ...
    }
}
```

> [!NOTE]
> 不支持在 `Dispose` 中调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A>。 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> 可能在拆除呈现器时调用，因此不支持在此时请求 UI 更新。

取消订阅 .NET 事件中的事件处理程序。 下面的 [Blazor 窗体](xref:blazor/forms-validation)示例演示如何解除挂接 `Dispose` 方法中的事件处理程序：

* 专用字段和 Lambda 方法

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-1.razor?highlight=23,28)]

* 专用方法

  [!code-razor[](lifecycle/samples_snapshot/event-handler-disposal-2.razor?highlight=16,26)]

## <a name="cancelable-background-work"></a>可取消的后台工作

组件通常会执行长时间运行的后台工作，如进行网络调用 (<xref:System.Net.Http.HttpClient>) 以及与数据库交互。 在几种情况下，最好停止后台工作以节省系统资源。 例如，当用户离开组件时，后台异步操作不会自动停止。

后台工作项可能需要取消的其他原因包括：

* 正在执行的后台任务由错误的输入数据或处理参数启动。
* 正在执行的一组后台工作项必须替换为一组新的工作项。
* 必须更改当前正在执行的任务的优先级。
* 必须关闭应用才能将其重新部署到服务器。
* 服务器资源受到限制，需要重新计划后台工作项。

要在组件中实现可取消的后台工作模式：

* 使用 <xref:System.Threading.CancellationTokenSource> 和 <xref:System.Threading.CancellationToken>。
* 在[释放组件](#component-disposal-with-idisposable)时，以及需要随时通过手动取消标记进行取消时，请调用 [`CancellationTokenSource.Cancel`](xref:System.Threading.CancellationTokenSource.Cancel%2A) 以指示应取消后台工作。
* 异步调用返回后，对该标记调用 <xref:System.Threading.CancellationToken.ThrowIfCancellationRequested%2A>。

如下示例中：

* `await Task.Delay(5000, cts.Token);` 表示长时间运行的异步后台工作。
* `BackgroundResourceMethod` 表示如果在调用方法之前释放 `Resource`，则不应启动的长时间运行的后台方法。

```razor
@implements IDisposable
@using System.Threading

<button @onclick="LongRunningWork">Trigger long running work</button>

@code {
    private Resource resource = new Resource();
    private CancellationTokenSource cts = new CancellationTokenSource();

    protected async Task LongRunningWork()
    {
        await Task.Delay(5000, cts.Token);

        cts.Token.ThrowIfCancellationRequested();
        resource.BackgroundResourceMethod();
    }

    public void Dispose()
    {
        cts.Cancel();
        cts.Dispose();
        resource.Dispose();
    }

    private class Resource : IDisposable
    {
        private bool disposed;

        public void BackgroundResourceMethod()
        {
            if (disposed)
            {
                throw new ObjectDisposedException(nameof(Resource));
            }
            
            ...
        }
        
        public void Dispose()
        {
            disposed = true;
        }
    }
}
```
