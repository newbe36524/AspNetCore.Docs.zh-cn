---
title: ASP.NET Core Blazor 状态管理
author: guardrex
description: 了解如何在 Blazor Server 应用中保留状态。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/22/2020
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
uid: blazor/state-management
zone_pivot_groups: blazor-hosting-models
ms.openlocfilehash: a74f056447839c4cf057948f26a9ece9b5799656
ms.sourcegitcommit: d1a897ebd89daa05170ac448e4831d327f6b21a8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2020
ms.locfileid: "91606728"
---
# <a name="aspnet-core-no-locblazor-state-management"></a>ASP.NET Core Blazor 状态管理

作者：[Steve Sanderson](https://github.com/SteveSandersonMS) 及 [Luke Latham](https://github.com/guardrex)

::: zone pivot="webassembly"

在 Blazor WebAssembly 应用中创建的用户状态会保存在浏览器的内存中。

浏览器内存中保留的用户状态的示例：

* 呈现的 UI 中组件实例的层次结构及其最新的呈现输出。
* 组件实例中的字段和属性的值。
* [依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 服务实例中保留的数据。
* 通过 [JavaScript 互操作](xref:blazor/call-javascript-from-dotnet)调用设置的值。

当用户关闭并重新打开其浏览器或重新加载页面时，浏览器的内存中保存的用户状态丢失。

## <a name="persist-state-across-browser-sessions"></a>跨浏览器会话保留状态

通常情况下，在用户主动创建数据，而不是简单地读取已存在的数据时，会跨浏览器会话保持状态。

若要跨浏览器会话保留状态，应用必须将数据保存到浏览器的内存以外的其他存储位置。 状态暂留并非是自动进行的。 必须在开发应用时采取措施来实现有状态的数据暂留。

通常，只有用户投入了大量精力所创建的高价值状态才需要数据暂留。 在下面的示例中，保留状态可以节省时间或有助于商业活动：

* 多步骤 Web 窗体：如果多步骤 Web 窗体的多个已完成步骤的状态丢失，用户重新输入这些步骤的数据会非常耗时。 如果用户离开窗体并在稍后返回，在这种应用场景下，用户将丢失状态。
* 购物车：应用中任何代表潜在收入且具有重要商业价值的组件都可以保留。 如果用户丢失了其状态，进而丢失了其购物车，则在他们稍后返回站点时可购买较少的产品或服务。

应用只能保留应用状态。 不能保留 UI，如组件实例及其呈现树。 组件和呈现树通常不能序列化。 若要保留 UI 状态（如树视图控件的展开节点），应用必须使用自定义代码将 UI 状态行为建模为可序列化应用状态。

## <a name="where-to-persist-state"></a>保留状态的位置

用于保留状态的常见位置有：

* [服务器端存储](#server-side-storage)
* [URL](#url)
* [浏览器存储](#browser-storage)
* [内存中状态容器服务](#in-memory-state-container-service)

### <a name="server-side-storage"></a>服务器端存储

对于跨多个用户和设备的永久数据持久性，应用可以使用通过 Web API 访问的独立服务器端存储。 选项包括：

* Blob 存储
* 键值存储
* 关系数据库
* 表存储

保存数据后，将保留用户的状态，并在任何新的浏览器会话中可用。

由于 Blazor WebAssembly 应用完全在用户的浏览器中运行，因此它们需要额外的措施来访问安全的外部系统，如存储服务和数据库。 Blazor WebAssembly 应用的保护方式与单页应用 (SPA) 相同。 通常，应用通过 [OAuth](https://oauth.net)/[OpenID Connect (OIDC)](https://openid.net/connect/) 对用户进行身份验证，然后通过对服务器端应用的 Web API 调用与存储服务和数据库进行交互。 服务器端应用可协调 Blazor WebAssembly 应用与存储服务或数据库之间的数据传输。 Blazor WebAssembly 应用保持与服务器端应用的临时连接，而服务器端应用具有到存储的持久连接。

有关详细信息，请参阅以下资源：

* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>
* Blazor 安全和 Identity 文章

有关 Azure 数据存储选项的详细信息，请参阅以下内容：

* [Azure 数据库](https://azure.microsoft.com/product-categories/databases/)
* [Azure 存储文档](/azure/storage/)

### <a name="url"></a>URL

对于表示导航状态的暂时性数据，请将数据作为 URL 的一部分进行建模。 URL 中建模的用户状态示例：

* 已查看实体的 ID。
* 分页网格中的当前页码。

在用户手动重新加载页面时保留的浏览器地址栏的内容。

有关使用 [`@page`](xref:mvc/views/razor#page) 指令定义 URL 模式的信息，请参阅 <xref:blazor/fundamentals/routing>。

### <a name="browser-storage"></a>浏览器存储

对于用户正在主动创建的暂时性数据，通用存储位置是浏览器的 [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) 和 [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) 集合：

* `localStorage` 的应用范围限定为浏览器的窗口。 如果用户重载页面或关闭并重新打开浏览器，则状态保持不变。 如果用户打开多个浏览器选项卡，则状态跨选项卡共享。 数据保留在 `localStorage` 中，直到被显式清除为止。
* `sessionStorage` 的应用范围限定为浏览器的选项卡。如果用户重载该选项卡，则状态保持不变。 如果用户关闭该选项卡或该浏览器，则状态丢失。 如果用户打开多个浏览器选项卡，则每个选项卡都有自己独立的数据版本。

> [!NOTE]
> `localStorage` 和 `sessionStorage` 可用于 Blazor WebAssembly 应用，但只能通过编写自定义代码或使用第三方包的方式使用。

通常，`sessionStorage` 使用起来更安全。 `sessionStorage` 避免了用户打开多个选项卡并遇到以下问题的风险：

* 跨选项卡的状态存储中出现 bug。
* 一个选项卡覆盖其他选项卡的状态时出现混乱行为。

如果应用必须在关闭和重新打开浏览器期间保持状态，则 `localStorage` 是更好的选择。

> [!WARNING]
> 用户可以查看或篡改 `localStorage` 和 `sessionStorage` 中存储的数据。

## <a name="in-memory-state-container-service"></a>内存中状态容器服务

[!INCLUDE[](~/includes/blazor-state-management/state-container.md)]

## <a name="additional-resources"></a>其他资源

* [在身份验证操作之前保存应用状态](xref:blazor/security/webassembly/additional-scenarios#save-app-state-before-an-authentication-operation)
* <xref:blazor/call-web-api>
* <xref:blazor/security/webassembly/index>

::: zone-end

::: zone pivot="server"

Blazor Server 是有状态的应用框架。 大多数情况下，应用保持与服务器的连接。 用户的状态保留在线路中的服务器内存中。 

线路中保留的用户状态示例：

* 呈现的 UI 中组件实例的层次结构及其最新的呈现输出。
* 组件实例中的字段和属性的值。
* 在线路范围内的[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 服务实例中保留的数据。

还可以通过 [JavaScript 互操作](xref:blazor/call-javascript-from-dotnet) 调用在浏览器的内存集的 JavaScript 变量中找到用户状态。

如果用户遇到暂时的网络连接丢失问题，Blazor 会尝试将用户重新连接到具有其原始状态的原始线路。 但是，将用户重新连接到服务器内存中的原始电路并非总是能够实现的：

* 服务器不能永久保留断开连接的线路。 超时后或在服务器面临内存压力时，服务器必须释放断开连接的线路。
* 在负载均衡的多服务器部署环境中，不再需要单个服务器处理整个请求量时，它可能会失败或被自动删除。 在用户尝试重新连接时，用户的原始服务器处理请求可能会变得不可用。
* 用户可能会关闭并重新打开其浏览器或重载页面，这会删除浏览器内存中保留的所有状态。 例如，通过 JavaScript 互操作调用设置的 JavaScript 变量值会丢失。

当无法将用户重新连接到其原始线路时，用户将收到一个具有空状态的新线路。 这等效于关闭并重新打开桌面应用。

## <a name="persist-state-across-circuits"></a>跨线路保留状态

通常情况下，在用户主动创建数据，而不是简单地读取已存在的数据时，会跨线路保持状态。

若要跨线路保留状态，应用必须将数据保存到服务器的内存以外的其他存储位置。 状态暂留并非是自动进行的。 必须在开发应用时采取措施来实现有状态的数据暂留。

通常，只有用户投入了大量精力所创建的高价值状态才需要数据暂留。 在下面的示例中，保留状态可以节省时间或有助于商业活动：

* 多步骤 Web 窗体：如果多步骤 Web 窗体的多个已完成步骤的状态丢失，用户重新输入这些步骤的数据会非常耗时。 如果用户离开窗体并在稍后返回，在这种应用场景下，用户将丢失状态。
* 购物车：应用中任何代表潜在收入且具有重要商业价值的组件都可以保留。 如果用户丢失了其状态，进而丢失了其购物车，则在他们稍后返回站点时可购买较少的产品或服务。

应用只能保留应用状态。 不能保留 UI，如组件实例及其呈现树。 组件和呈现树通常不能序列化。 若要保留 UI 状态（如树视图控件的展开节点），应用必须使用自定义代码将 UI 状态行为建模为可序列化应用状态。

## <a name="where-to-persist-state"></a>保留状态的位置

用于保留状态的常见位置有：

* [服务器端存储](#server-side-storage)
* [URL](#url)
* [浏览器存储](#browser-storage)
* [内存中状态容器服务](#in-memory-state-container-service)

### <a name="server-side-storage"></a>服务器端存储

对于跨多个用户和设备的永久数据持久性，应用可以使用服务器端存储。 选项包括：

* Blob 存储
* 键值存储
* 关系数据库
* 表存储

保存数据后，将保留用户的状态，并在任何新的线路中可用。

有关 Azure 数据存储选项的详细信息，请参阅以下内容：

* [Azure 数据库](https://azure.microsoft.com/product-categories/databases/)
* [Azure 存储文档](/azure/storage/)

### <a name="url"></a>URL

对于表示导航状态的暂时性数据，请将数据作为 URL 的一部分进行建模。 URL 中建模的用户状态示例：

* 已查看实体的 ID。
* 分页网格中的当前页码。

保留浏览器地址栏的内容：

* 如果用户手动重载页面。
* 如果 Web 服务器不可用，且用户被强制重载页面，以便连接到其他服务器。

有关使用 [`@page`](xref:mvc/views/razor#page) 指令定义 URL 模式的信息，请参阅 <xref:blazor/fundamentals/routing>。

### <a name="browser-storage"></a>浏览器存储

对于用户正在主动创建的暂时性数据，通用存储位置是浏览器的 [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) 和 [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage) 集合：

* `localStorage` 的应用范围限定为浏览器的窗口。 如果用户重载页面或关闭并重新打开浏览器，则状态保持不变。 如果用户打开多个浏览器选项卡，则状态跨选项卡共享。 数据保留在 `localStorage` 中，直到被显式清除为止。
* `sessionStorage` 的应用范围限定为浏览器的选项卡。如果用户重载该选项卡，则状态保持不变。 如果用户关闭该选项卡或该浏览器，则状态丢失。 如果用户打开多个浏览器选项卡，则每个选项卡都有自己独立的数据版本。

通常，`sessionStorage` 使用起来更安全。 `sessionStorage` 避免了用户打开多个选项卡并遇到以下问题的风险：

* 跨选项卡的状态存储中出现 bug。
* 一个选项卡覆盖其他选项卡的状态时出现混乱行为。

如果应用必须在关闭和重新打开浏览器期间保持状态，则 `localStorage` 是更好的选择。

使用浏览器存储时的注意事项：

* 与使用服务器端数据库类似，加载和保存数据都是异步的。
* 与服务器端数据库不同，在预呈现期间，存储不可用，因为在预呈现阶段，请求的页面在浏览器中不存在。
* 保留状态的位置对于 Blazor Server 应用，持久存储几千字节的数据是合理的。 超出几千字节后，你就须考虑性能影响，因为数据是跨网络加载和保存的。
* 用户可以查看或篡改数据。 [ASP.NET Core 数据保护](xref:security/data-protection/introduction)可以降低风险。 例如，[ASP.NET Core 受保护的浏览器存储](#aspnet-core-protected-browser-storage)使用 ASP.NET Core 数据保护。

第三方 NuGet 包提供使用 `localStorage` 和 `sessionStorage` 时采用的 API。 值得考虑的是，选择一个透明地使用 [ASP.NET Core 数据保护](xref:security/data-protection/introduction)的包。 数据保护可对存储的数据进行加密，并降低篡改存储数据的潜在风险。 如果 JSON 序列化的数据以纯文本形式存储，则用户可以使用浏览器开发人员工具查看数据，还可以修改存储的数据。 保护数据并非总是一个问题，因为有些数据本质上可能是无足轻重的。 例如，读取或修改 UI 元素的存储颜色不会对用户或组织造成严重的安全风险。 避免允许用户检查或篡改敏感数据。

::: moniker range=">= aspnetcore-5.0"

## <a name="aspnet-core-protected-browser-storage"></a>ASP.NET Core 受保护的浏览器存储

ASP.NET Core 受保护的浏览器存储将 [ASP.NET Core 数据保护](xref:security/data-protection/introduction)用于 [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) 和 [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage)。

> [!NOTE]
> 受保护的浏览器存储依赖于 ASP.NET Core 数据保护，仅支持用于 Blazor Server 应用。

### <a name="configuration"></a>配置

1. 将包引用添加到 [`Microsoft.AspNetCore.Components.Web.Extensions`](https://www.nuget.org/packages/Microsoft.AspNetCore.Http.Extensions)。
1. 在 `Startup.ConfigureServices` 中，调用 `AddProtectedBrowserStorage` 以将 `localStorage` 和 `sessionStorage` 服务添加到服务集合：

   ```csharp
   services.AddProtectedBrowserStorage();
   ```

### <a name="save-and-load-data-within-a-component"></a>保存和加载组件中的数据

在需要将数据加载或保存到浏览器存储的任何组件中，使用 [`@inject`](xref:mvc/views/razor#inject) 指令注入以下任意一项的实例：

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

具体选择取决于要使用的浏览器存储位置。 在以下示例中，使用 `sessionStorage`：

```razor
@using Microsoft.AspNetCore.Components.Web.Extensions
@inject ProtectedSessionStorage ProtectedSessionStore
```

可将 `@using` 指令放在应用的 `_Imports.razor` 文件而不是组件中。 使用 `_Imports.razor` 文件可使命名空间可用于应用的较大部分或整个应用。

若要在基于 Blazor Server 项目模板的应用的 `Counter` 组件中保留 `currentCount` 值，请修改 `IncrementCount` 方法以使用 `ProtectedSessionStore.SetAsync`：

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

在更大、更真实的应用中，存储单个字段是不太可能出现的情况。 应用更有可能存储包含复杂状态的整个模型对象。 `ProtectedSessionStore` 自动串行化和反序列化 JSON 数据以存储复杂的状态对象。

在前面的代码示例中，`currentCount` 数据存储为用户浏览器中的 `sessionStorage['count']`。 数据不会以纯文本形式存储，而是使用 ASP.NET Core 的数据保护进行保护。 如果在浏览器的开发人员控制台中评估了 `sessionStorage['count']`，则可以检查已加密的数据。

若要在用户稍后返回到 `Counter` 组件时（包括用于位于新线路上时）恢复 `currentCount` 数据，请使用 `ProtectedSessionStore.GetAsync`：

```csharp
protected override async Task OnInitializedAsync()
{
    var result = await ProtectedSessionStore.GetAsync<int>("count");
    currentCount = result.Success ? result.Value : 0;
}
```

如果组件的参数包括导航状态，请调用 `ProtectedSessionStore.GetAsync` 并将非 `null` 结果分配给 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>，而不是 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>。 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 仅在首次实例化组件时调用一次。 如果用户导航到不同的 URL，而仍然停留在相同的页面上，则 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 之后不会再次调用。 有关详细信息，请参阅 <xref:blazor/components/lifecycle>。

> [!WARNING]
> 本节中的示例仅在服务器未启用预呈现的情况下有效。 启用预呈现后，会生成错误，说明由于正在预呈现组件，无法发起 JavaScript 互操作调用。
>
> 禁用预呈现或添加其他代码以处理预呈现。 若要了解有关编写可处理预呈现的代码的详细信息，请参阅[处理预呈现](#handle-prerendering)一节。

### <a name="handle-the-loading-state"></a>处理加载状态

由于浏览器存储是异步访问（通过网络连接进行访问）的，因此往往需要一段时间才能加载完数据并可供组件使用。 为获得最佳结果，请在加载进行过程中呈现加载状态消息，而不要显示空数据或默认数据。

一种方法是跟踪数据是否为 `null`（表示数据仍在加载）。 在默认 `Counter` 组件中，计数保留在 `int` 中。 通过将问号 (`?`) 添加到类型 (`int`)，[使 `currentCount` 可以为 null](/dotnet/csharp/language-reference/builtin-types/nullable-value-types)：

```csharp
private int? currentCount;
```

请勿无条件地显示计数和“`Increment`”按钮，而是禁止通过检查 <xref:System.Nullable%601.HasValue%2A> 以加载完数据后才显示这些元素：

```razor
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

### <a name="handle-prerendering"></a>处理预呈现

预呈现期间：

* 与用户浏览器的交互式连接不存在。
* 浏览器尚无可在其中运行 JavaScript 代码的页面。

在预呈现期间，`localStorage` 或 `sessionStorage` 不可用。 如果组件尝试与存储进行交互，则会生成错误，说明由于正在预呈现组件，无法发起 JavaScript 互操作调用。

解决此错误的一种方法是禁用预呈现。 如果应用大量使用基于浏览器的存储，则这通常是最佳选择。 预呈现会增加复杂性，且不会给应用带来好处，因为在 `localStorage` 或 `sessionStorage` 可用之前，应用无法预呈现任何有用的内容。

若要禁用预呈现，请打开 `Pages/_Host.cshtml` 文件，并将[组件标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)的 `render-mode` 属性更改为 <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server>：

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

对于不使用 `localStorage` 或 `sessionStorage` 的其他页面，预呈现可能很有用。 若要保持预呈现状态，可延迟加载操作，直到浏览器连接到线路。 以下是存储计数器值的示例：

```razor
@using Microsoft.AspNetCore.Components.Web.Extensions
@inject ProtectedLocalStorage ProtectedLocalStore

@if (isConnected)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}

@code {
    private int currentCount;
    private bool isConnected;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            isConnected = true;
            await LoadStateAsync();
            StateHasChanged();
        }
    }

    private async Task LoadStateAsync()
    {
        var result = await ProtectedLocalStore.GetAsync<int>("count");
        currentCount = result.Success ? result.Value : 0;
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await ProtectedLocalStore.SetAsync("count", currentCount);
    }
}
```

### <a name="factor-out-the-state-preservation-to-a-common-location"></a>将状态保留提取到常见位置

如果多个组件依赖于基于浏览器的存储，则多次重新实现状态提供程序代码会造成代码重复。 若要避免代码重复，一种方法是创建一个封装了状态提供程序逻辑的状态提供程序父组件。 子组件可处理保留的数据，而无需考虑状态暂留机制。

在 `CounterStateProvider` 组件的以下示例中，将计数器数据保留到 `sessionStorage`：

```razor
@using Microsoft.AspNetCore.Components.Web.Extensions
@inject ProtectedSessionStorage ProtectedSessionStore

@if (isLoaded)
{
    <CascadingValue Value="@this">
        @ChildContent
    </CascadingValue>
}
else
{
    <p>Loading...</p>
}

@code {
    private bool isLoaded;

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public int CurrentCount { get; set; }

    protected override async Task OnInitializedAsync()
    {
        var result = await ProtectedSessionStore.GetAsync<int>("count");
        currentCount = result.Success ? result.Value : 0;
        isLoaded = true;
    }

    public async Task SaveChangesAsync()
    {
        await ProtectedSessionStore.SetAsync("count", CurrentCount);
    }
}
```

`CounterStateProvider` 组件处理加载阶段的方式是在加载完成后才呈现其子内容。

若要使用 `CounterStateProvider` 组件，请围绕需要访问计数器状态的任何其他组件包装该组件的实例。 若要使某个应用中的所有组件都可以访问该状态，请围绕 `App` 组件 (`App.razor`) 中的 <xref:Microsoft.AspNetCore.Components.Routing.Router> 来包装 `CounterStateProvider` 组件：

```razor
<CounterStateProvider>
    <Router AppAssembly="typeof(Startup).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

已包装的组件接收并可修改保留的计数器状态。 以下 `Counter` 组件实现了该模式：

```razor
@page "/counter"

<p>Current count: <strong>@CounterStateProvider.CurrentCount</strong></p>
<button @onclick="IncrementCount">Increment</button>

@code {
    [CascadingParameter]
    private CounterStateProvider CounterStateProvider { get; set; }

    private async Task IncrementCount()
    {
        CounterStateProvider.CurrentCount++;
        await CounterStateProvider.SaveChangesAsync();
    }
}
```

与 `ProtectedBrowserStorage` 进行交互无需前面的组件，该组件也不会处理“正在加载”阶段。

如前所述，若要处理预呈现，可对 `CounterStateProvider` 进行修改，以便所有使用计数器数据的组件均可自动处理预呈现。 有关详细信息，请参阅[处理预呈现](#handle-prerendering)部分。

通常，建议在以下情况下使用状态提供程序父组件模式：

* 跨多个组件使用状态。
* 只有一个顶级状态对象要保留时。

若要保留多个不同的状态对象并在不同位置使用不同的对象子集，最好避免全局保留状态。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="protected-browser-storage-experimental-nuget-package"></a>受保护的浏览器存储实验性 NuGet 包

ASP.NET Core 受保护的浏览器存储将 [ASP.NET Core 数据保护](xref:security/data-protection/introduction)用于 [`localStorage`](https://developer.mozilla.org/docs/Web/API/Window/localStorage) 和 [`sessionStorage`](https://developer.mozilla.org/docs/Web/API/Window/sessionStorage)。

> [!WARNING]
> `Microsoft.AspNetCore.ProtectedBrowserStorage` 是一个不受支持的实验性包，不适合用于生产。
>
> 此包仅适用于 ASP.NET Core 3.1 Blazor Server 应用。

### <a name="configuration"></a>配置

1. 将包引用添加到 [`Microsoft.AspNetCore.ProtectedBrowserStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.ProtectedBrowserStorage)。
1. 在 `Pages/_Host.cshtml` 文件中，将以下脚本添加到结束标记 `</body>` 之前：

   ```cshtml
   <script src="_content/Microsoft.AspNetCore.ProtectedBrowserStorage/protectedBrowserStorage.js"></script>
   ```

1. 在 `Startup.ConfigureServices` 中，调用 `AddProtectedBrowserStorage` 以将 `localStorage` 和 `sessionStorage` 服务添加到服务集合：

   ```csharp
   services.AddProtectedBrowserStorage();
   ```

### <a name="save-and-load-data-within-a-component"></a>保存和加载组件中的数据

在需要将数据加载或保存到浏览器存储的任何组件中，使用 [`@inject`](xref:mvc/views/razor#inject) 指令注入以下任意一项的实例：

* `ProtectedLocalStorage`
* `ProtectedSessionStorage`

具体选择取决于要使用的浏览器存储位置。 在以下示例中，使用 `sessionStorage`：

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore
```

可将 `@using` 语句放置在 `_Imports.razor` 文件而不是组件中。 使用 `_Imports.razor` 文件可使命名空间可用于应用的较大部分或整个应用。

若要在基于 Blazor Server 项目模板的应用的 `Counter` 组件中保留 `currentCount` 值，请修改 `IncrementCount` 方法以使用 `ProtectedSessionStore.SetAsync`：

```csharp
private async Task IncrementCount()
{
    currentCount++;
    await ProtectedSessionStore.SetAsync("count", currentCount);
}
```

在更大、更真实的应用中，存储单个字段是不太可能出现的情况。 应用更有可能存储包含复杂状态的整个模型对象。 `ProtectedSessionStore` 自动串行化和反序列化 JSON 数据。

在前面的代码示例中，`currentCount` 数据存储为用户浏览器中的 `sessionStorage['count']`。 数据不会以纯文本形式存储，而是使用 ASP.NET Core 的数据保护进行保护。 如果在浏览器的开发人员控制台中评估了 `sessionStorage['count']`，则可以检查已加密的数据。

若要在用户稍后返回到 `Counter` 组件时（包括他们位于全新线路上时）恢复 `currentCount` 数据，请使用 `ProtectedSessionStore.GetAsync`：

```csharp
protected override async Task OnInitializedAsync()
{
    currentCount = await ProtectedSessionStore.GetAsync<int>("count");
}
```

如果组件的参数包括导航状态，请调用 `ProtectedSessionStore.GetAsync` 并将结果分配给 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnParametersSetAsync%2A>，而不是 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A>。 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 仅在首次实例化组件时调用一次。 如果用户导航到不同的 URL，而仍然停留在相同的页面上，则 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnInitializedAsync%2A> 之后不会再次调用。 有关详细信息，请参阅 <xref:blazor/components/lifecycle>。

> [!WARNING]
> 本节中的示例仅在服务器未启用预呈现的情况下有效。 启用预呈现后，会生成错误，说明由于正在预呈现组件，无法发起 JavaScript 互操作调用。
>
> 禁用预呈现或添加其他代码以处理预呈现。 若要了解有关编写可处理预呈现的代码的详细信息，请参阅[处理预呈现](#handle-prerendering)一节。

### <a name="handle-the-loading-state"></a>处理加载状态

由于浏览器存储是异步访问（通过网络连接进行访问）的，因此往往需要一段时间才能加载完数据并可供组件使用。 为获得最佳结果，请在加载进行过程中呈现加载状态消息，而不要显示空数据或默认数据。

一种方法是跟踪数据是否为 `null`（表示数据仍在加载）。 在默认 `Counter` 组件中，计数保留在 `int` 中。 通过将问号 (`?`) 添加到类型 (`int`)，[使 `currentCount` 可以为 null](/dotnet/csharp/language-reference/builtin-types/nullable-value-types)：

```csharp
private int? currentCount;
```

请勿无条件地显示计数和“`Increment`”按钮，而选择仅在数据已加载后才显示这些元素：

```razor
@if (currentCount.HasValue)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}
```

### <a name="handle-prerendering"></a>处理预呈现

预呈现期间：

* 与用户浏览器的交互式连接不存在。
* 浏览器尚无可在其中运行 JavaScript 代码的页面。

在预呈现期间，`localStorage` 或 `sessionStorage` 不可用。 如果组件尝试与存储进行交互，则会生成错误，说明由于正在预呈现组件，无法发起 JavaScript 互操作调用。

解决此错误的一种方法是禁用预呈现。 如果应用大量使用基于浏览器的存储，则这通常是最佳选择。 预呈现会增加复杂性，且不会给应用带来好处，因为在 `localStorage` 或 `sessionStorage` 可用之前，应用无法预呈现任何有用的内容。

若要禁用预呈现，请打开 `Pages/_Host.cshtml` 文件，并将[组件标记帮助程序](xref:mvc/views/tag-helpers/builtin-th/component-tag-helper)的 `render-mode` 属性更改为 <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server>：

```cshtml
<component type="typeof(App)" render-mode="Server" />
```

对于不使用 `localStorage` 或 `sessionStorage` 的其他页面，预呈现可能很有用。 若要保持预呈现状态，可延迟加载操作，直到浏览器连接到线路。 以下是存储计数器值的示例：

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedLocalStorage ProtectedLocalStore

@if (isConnected)
{
    <p>Current count: <strong>@currentCount</strong></p>
    <button @onclick="IncrementCount">Increment</button>
}
else
{
    <p>Loading...</p>
}

@code {
    private int? currentCount;
    private bool isConnected = false;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            isConnected = true;
            await LoadStateAsync();
            StateHasChanged();
        }
    }

    private async Task LoadStateAsync()
    {
        currentCount = await ProtectedLocalStore.GetAsync<int>("count");
    }

    private async Task IncrementCount()
    {
        currentCount++;
        await ProtectedLocalStore.SetAsync("count", currentCount);
    }
}
```

### <a name="factor-out-the-state-preservation-to-a-common-location"></a>将状态保留提取到常见位置

如果多个组件依赖于基于浏览器的存储，则多次重新实现状态提供程序代码会造成代码重复。 若要避免代码重复，一种方法是创建一个封装了状态提供程序逻辑的状态提供程序父组件。 子组件可处理保留的数据，而无需考虑状态暂留机制。

在 `CounterStateProvider` 组件的以下示例中，将计数器数据保留到 `sessionStorage`：

```razor
@using Microsoft.AspNetCore.ProtectedBrowserStorage
@inject ProtectedSessionStorage ProtectedSessionStore

@if (isLoaded)
{
    <CascadingValue Value="@this">
        @ChildContent
    </CascadingValue>
}
else
{
    <p>Loading...</p>
}

@code {
    private bool isLoaded;

    [Parameter]
    public RenderFragment ChildContent { get; set; }

    public int CurrentCount { get; set; }

    protected override async Task OnInitializedAsync()
    {
        CurrentCount = await ProtectedSessionStore.GetAsync<int>("count");
        isLoaded = true;
    }

    public async Task SaveChangesAsync()
    {
        await ProtectedSessionStore.SetAsync("count", CurrentCount);
    }
}
```

`CounterStateProvider` 组件处理加载阶段的方式是在加载完成后才呈现其子内容。

若要使用 `CounterStateProvider` 组件，请围绕需要访问计数器状态的任何其他组件包装该组件的实例。 若要使某个应用中的所有组件都可以访问该状态，请围绕 `App` 组件 (`App.razor`) 中的 <xref:Microsoft.AspNetCore.Components.Routing.Router> 来包装 `CounterStateProvider` 组件：

```razor
<CounterStateProvider>
    <Router AppAssembly="typeof(Startup).Assembly">
        ...
    </Router>
</CounterStateProvider>
```

已包装的组件接收并可修改保留的计数器状态。 以下 `Counter` 组件实现了该模式：

```razor
@page "/counter"

<p>Current count: <strong>@CounterStateProvider.CurrentCount</strong></p>
<button @onclick="IncrementCount">Increment</button>

@code {
    [CascadingParameter]
    private CounterStateProvider CounterStateProvider { get; set; }

    private async Task IncrementCount()
    {
        CounterStateProvider.CurrentCount++;
        await CounterStateProvider.SaveChangesAsync();
    }
}
```

与 `ProtectedBrowserStorage` 进行交互无需前面的组件，该组件也不会处理“正在加载”阶段。

如前所述，若要处理预呈现，可对 `CounterStateProvider` 进行修改，以便所有使用计数器数据的组件均可自动处理预呈现。 有关详细信息，请参阅[处理预呈现](#handle-prerendering)部分。

通常，建议在以下情况下使用状态提供程序父组件模式：

* 跨多个组件使用状态。
* 只有一个顶级状态对象要保留时。

若要保留多个不同的状态对象并在不同位置使用不同的对象子集，最好避免全局保留状态。

::: moniker-end

## <a name="in-memory-state-container-service"></a>内存中状态容器服务

[!INCLUDE[](~/includes/blazor-state-management/state-container.md)]

::: zone-end
