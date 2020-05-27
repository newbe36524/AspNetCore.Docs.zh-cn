---
<span data-ttu-id="47bc8-101">标题： "ASP.NET Core Blazor WebAssembly 性能最佳做法作者：说明：在 ASP.NET Core WebAssembly 应用中提高性能 Blazor 并避免常见性能问题的提示。"</span><span class="sxs-lookup"><span data-stu-id="47bc8-101">title: 'ASP.NET Core Blazor WebAssembly performance best practices' author: description: 'Tips for increasing performance in ASP.NET Core Blazor WebAssembly apps and avoiding common performance problems.'</span></span>
<span data-ttu-id="47bc8-102">monikerRange: ms.author: ms.custom: ms.date: no-loc:</span><span class="sxs-lookup"><span data-stu-id="47bc8-102">monikerRange: ms.author: ms.custom: ms.date: no-loc:</span></span>
- <span data-ttu-id="47bc8-103">'Blazor'</span><span class="sxs-lookup"><span data-stu-id="47bc8-103">'Blazor'</span></span>
- <span data-ttu-id="47bc8-104">'Identity'</span><span class="sxs-lookup"><span data-stu-id="47bc8-104">'Identity'</span></span>
- <span data-ttu-id="47bc8-105">'Let's Encrypt'</span><span class="sxs-lookup"><span data-stu-id="47bc8-105">'Let's Encrypt'</span></span>
- <span data-ttu-id="47bc8-106">'Razor'</span><span class="sxs-lookup"><span data-stu-id="47bc8-106">'Razor'</span></span>
- <span data-ttu-id="47bc8-107">'SignalR' uid:</span><span class="sxs-lookup"><span data-stu-id="47bc8-107">'SignalR' uid:</span></span> 

---
# <a name="aspnet-core-blazor-webassembly-performance-best-practices"></a><span data-ttu-id="47bc8-108">ASP.NET Core Blazor WebAssembly 性能最佳做法</span><span class="sxs-lookup"><span data-stu-id="47bc8-108">ASP.NET Core Blazor WebAssembly performance best practices</span></span>

<span data-ttu-id="47bc8-109">作者： [Pranav Krishnamoorthy](https://github.com/pranavkm)</span><span class="sxs-lookup"><span data-stu-id="47bc8-109">By [Pranav Krishnamoorthy](https://github.com/pranavkm)</span></span>

<span data-ttu-id="47bc8-110">本文提供 ASP.NET Core Blazor WebAssembly 性能最佳做法的准则。</span><span class="sxs-lookup"><span data-stu-id="47bc8-110">This article provides guidelines for ASP.NET Core Blazor WebAssembly performance best practices.</span></span>

## <a name="avoid-unnecessary-component-renders"></a><span data-ttu-id="47bc8-111">避免不必要的组件呈现</span><span class="sxs-lookup"><span data-stu-id="47bc8-111">Avoid unnecessary component renders</span></span>

Blazor<span data-ttu-id="47bc8-112">如果算法感觉组件未发生更改，则其比较算法可避免 rerendering 组件。</span><span class="sxs-lookup"><span data-stu-id="47bc8-112">'s diffing algorithm avoids rerendering a component when the algorithm perceives that the component hasn't changed.</span></span> <span data-ttu-id="47bc8-113">重写 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A?displayProperty=nameWithType> 以对组件呈现进行精细控制。</span><span class="sxs-lookup"><span data-stu-id="47bc8-113">Override <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A?displayProperty=nameWithType> for fine-grained control over component rendering.</span></span>

<span data-ttu-id="47bc8-114">如果创作一个仅限 UI 的组件，而该组件在初始呈现后永不更改，则将配置 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 为返回 `false` ：</span><span class="sxs-lookup"><span data-stu-id="47bc8-114">If authoring a UI-only component that never changes after the initial render, configure <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> to return `false`:</span></span>

```razor
@code {
    protected override bool ShouldRender() => false;
}
```

<span data-ttu-id="47bc8-115">大多数应用不需要细化控制，但 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 也可用于有选择地呈现响应 UI 事件的组件。</span><span class="sxs-lookup"><span data-stu-id="47bc8-115">Most apps don't require fine-grained control, but <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> can also be used to selectively render a component responding to a UI event.</span></span>

<span data-ttu-id="47bc8-116">如下示例中：</span><span class="sxs-lookup"><span data-stu-id="47bc8-116">In the following example:</span></span>

* <span data-ttu-id="47bc8-117"><xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A>被重写并设置为该字段的值 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> ，这是最初 `false` 加载组件时的值。</span><span class="sxs-lookup"><span data-stu-id="47bc8-117"><xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> is overridden and set to the value of the <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> field, which is initially `false` when the component loads.</span></span>
* <span data-ttu-id="47bc8-118">选择该按钮时，将 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 设置为 `true` ，这会强制组件诸如此类已更新 `currentCount` 。</span><span class="sxs-lookup"><span data-stu-id="47bc8-118">When the button is selected, <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> is set to `true`, which forces the component to rerender with the updated `currentCount`.</span></span>
* <span data-ttu-id="47bc8-119">在 rerendering 后， <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> 将返回的值设置 <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> 为， `false` 以防止在下一次选中按钮之前 rerendering 进一步。</span><span class="sxs-lookup"><span data-stu-id="47bc8-119">Immediately after rerendering, <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRender%2A> sets the value of <xref:Microsoft.AspNetCore.Components.ComponentBase.ShouldRender%2A> back to `false` to prevent further rerendering until the next time the button is selected.</span></span>

```razor
<p>Current count: @currentCount</p>

<button @onclick="IncrementCount">Click me</button>

@code {
    private int currentCount = 0;
    private bool shouldRender;

    protected override bool ShouldRender() => shouldRender;

    protected override void OnAfterRender(bool first)
    {
        shouldRender = false;
    }

    private void IncrementCount()
    {
        currentCount++;
        shouldRender = true;
    }
}
```

<span data-ttu-id="47bc8-120">有关详细信息，请参阅 <xref:blazor/lifecycle#after-component-render>。</span><span class="sxs-lookup"><span data-stu-id="47bc8-120">For more information, see <xref:blazor/lifecycle#after-component-render>.</span></span>

## <a name="virtualize-re-usable-fragments"></a><span data-ttu-id="47bc8-121">虚拟化可重复使用的片段</span><span class="sxs-lookup"><span data-stu-id="47bc8-121">Virtualize re-usable fragments</span></span>

<span data-ttu-id="47bc8-122">组件提供一种方便的方法来生成可重复使用的代码片段和标记。</span><span class="sxs-lookup"><span data-stu-id="47bc8-122">Components offer a convenient approach to produce re-usable fragments of code and markup.</span></span> <span data-ttu-id="47bc8-123">通常，我们建议你创作最符合应用要求的单个组件。</span><span class="sxs-lookup"><span data-stu-id="47bc8-123">In general, we recommend authoring individual components that best align with the app's requirements.</span></span> <span data-ttu-id="47bc8-124">需要注意的一点是，每个附加的子组件都有一个用于呈现父组件的总时间。</span><span class="sxs-lookup"><span data-stu-id="47bc8-124">One caveat is that each additional child component contributes to the total time it takes to render a parent component.</span></span> <span data-ttu-id="47bc8-125">对于大多数应用程序，额外的开销可忽略不计。</span><span class="sxs-lookup"><span data-stu-id="47bc8-125">For most apps, the additional overhead is negligible.</span></span> <span data-ttu-id="47bc8-126">生成大量组件的应用应考虑使用策略来减少处理开销，例如限制呈现的组件数。</span><span class="sxs-lookup"><span data-stu-id="47bc8-126">Apps that produce a large number of components should consider using strategies to reduce processing overhead, such as limiting the number of rendered components.</span></span>

<span data-ttu-id="47bc8-127">例如，呈现数百个包含组件的行的网格或列表是要呈现的处理器密集型。</span><span class="sxs-lookup"><span data-stu-id="47bc8-127">For example, a grid or list that renders hundreds of rows containing components is processor intensive to render.</span></span> <span data-ttu-id="47bc8-128">考虑虚拟化网格或列表布局，以便在任意给定时间只呈现部分组件。</span><span class="sxs-lookup"><span data-stu-id="47bc8-128">Consider virtualizing a grid or list layout so that only a subset of the components is rendered at any given time.</span></span> <span data-ttu-id="47bc8-129">有关组件子集呈现的示例，请参阅[虚拟化示例应用（aspnet/示例 GitHub 存储库）](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/Virtualization)中的以下组件：</span><span class="sxs-lookup"><span data-stu-id="47bc8-129">For an example of component subset rendering, see the following components in the [Virtualization sample app (aspnet/samples GitHub repository)](https://github.com/aspnet/samples/tree/master/samples/aspnetcore/blazor/Virtualization):</span></span>

* <span data-ttu-id="47bc8-130">`Virtualize`组件（[共享/虚拟化 razor](https://github.com/aspnet/samples/blob/master/samples/aspnetcore/blazor/Virtualization/Shared/Virtualize.cs)）：用 c # 编写的组件，该组件实现 <xref:Microsoft.AspNetCore.Components.ComponentBase> 以基于用户滚动呈现一组天气数据行。</span><span class="sxs-lookup"><span data-stu-id="47bc8-130">`Virtualize` component ([Shared/Virtualize.razor](https://github.com/aspnet/samples/blob/master/samples/aspnetcore/blazor/Virtualization/Shared/Virtualize.cs)): A component written in C# that implements <xref:Microsoft.AspNetCore.Components.ComponentBase> to render a set of weather data rows based on user scrolling.</span></span>
* <span data-ttu-id="47bc8-131">`FetchData`组件（[Pages/FetchData](https://github.com/aspnet/samples/blob/master/samples/aspnetcore/blazor/Virtualization/Pages/FetchData.razor)）：使用 `Virtualize` 组件一次显示25行天气数据。</span><span class="sxs-lookup"><span data-stu-id="47bc8-131">`FetchData` component ([Pages/FetchData.razor](https://github.com/aspnet/samples/blob/master/samples/aspnetcore/blazor/Virtualization/Pages/FetchData.razor)): Uses the `Virtualize` component to display 25 rows of weather data at a time.</span></span>

## <a name="avoid-javascript-interop-to-marshal-data"></a><span data-ttu-id="47bc8-132">避免 JavaScript 互操作对数据进行封送处理</span><span class="sxs-lookup"><span data-stu-id="47bc8-132">Avoid JavaScript interop to marshal data</span></span>

<span data-ttu-id="47bc8-133">在 Blazor WebAssembly 中，JavaScript （JS）互操作调用必须遍历 WebAssembly 边界。</span><span class="sxs-lookup"><span data-stu-id="47bc8-133">In Blazor WebAssembly, a JavaScript (JS) interop call must traverse the WebAssembly-JS boundary.</span></span> <span data-ttu-id="47bc8-134">在两个上下文之间序列化和反序列化内容会为应用创建处理开销。</span><span class="sxs-lookup"><span data-stu-id="47bc8-134">Serializing and deserializing content across the two contexts creates processing overhead for the app.</span></span> <span data-ttu-id="47bc8-135">经常会对性能产生不利影响。</span><span class="sxs-lookup"><span data-stu-id="47bc8-135">Frequent JS interop calls often adversely affects performance.</span></span> <span data-ttu-id="47bc8-136">为了减少跨边界数据的封送，确定应用是否可以将许多小型有效负载整合到单个大型负载中，以避免 WebAssembly 与 JS 之间发生的大量上下文切换。</span><span class="sxs-lookup"><span data-stu-id="47bc8-136">To reduce the marshalling of data across the boundary, determine if the app can consolidate many small payloads into a single large payload to avoid the high volume of context switching between WebAssembly and JS.</span></span>

## <a name="use-systemtextjson"></a><span data-ttu-id="47bc8-137">使用 system.exception</span><span class="sxs-lookup"><span data-stu-id="47bc8-137">Use System.Text.Json</span></span>

Blazor<span data-ttu-id="47bc8-138">的 JS 互操作实现依赖于 <xref:System.Text.Json> ，后者是内存分配较少的高性能 JSON 序列化库。</span><span class="sxs-lookup"><span data-stu-id="47bc8-138">'s JS interop implementation relies on <xref:System.Text.Json>, which is a high-performance JSON serialization library with low memory allocation.</span></span> <span data-ttu-id="47bc8-139">通过 <xref:System.Text.Json> 添加一个或多个备用 JSON 库，使用不会导致其他应用负载大小。</span><span class="sxs-lookup"><span data-stu-id="47bc8-139">Using <xref:System.Text.Json> doesn't result in additional app payload size over adding one or more alternate JSON libraries.</span></span>

<span data-ttu-id="47bc8-140">有关迁移指南，请参阅[如何从 newtonsoft.json 迁移到 system.object](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to)。</span><span class="sxs-lookup"><span data-stu-id="47bc8-140">For migration guidance, see [How to migrate from Newtonsoft.Json to System.Text.Json](/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to).</span></span>

## <a name="use-synchronous-and-unmarshalled-js-interop-apis-where-appropriate"></a><span data-ttu-id="47bc8-141">在适当的位置使用同步和打乱 JS 互操作 Api</span><span class="sxs-lookup"><span data-stu-id="47bc8-141">Use synchronous and unmarshalled JS interop APIs where appropriate</span></span>

Blazor<span data-ttu-id="47bc8-142">WebAssembly 提供了 <xref:Microsoft.JSInterop.IJSRuntime> 可用于服务器应用的单一版本的其他两个版本 Blazor ：</span><span class="sxs-lookup"><span data-stu-id="47bc8-142"> WebAssembly offers two additional versions of <xref:Microsoft.JSInterop.IJSRuntime> over the single version available to Blazor Server apps:</span></span>

* <span data-ttu-id="47bc8-143"><xref:Microsoft.JSInterop.IJSInProcessRuntime>允许以同步方式调用 JS 互操作调用，其开销低于异步版本：</span><span class="sxs-lookup"><span data-stu-id="47bc8-143"><xref:Microsoft.JSInterop.IJSInProcessRuntime> allows invoking JS interop calls synchronously, which has less overhead than the asynchronous versions:</span></span>

  ```razor
  @inject IJSRuntime JS

  @code {
      protected override void OnInitialized()
      {
          var jsInProcess = (IJSInProcessRuntime)JS;

          var value = jsInProcess.Invoke<string>("jsInteropCall");
      }
  }
  ```

* <span data-ttu-id="47bc8-144"><xref:Microsoft.JSInterop.WebAssembly.WebAssemblyJSRuntime>允许打乱 JS 互操作调用：</span><span class="sxs-lookup"><span data-stu-id="47bc8-144"><xref:Microsoft.JSInterop.WebAssembly.WebAssemblyJSRuntime> permits unmarshalled JS interop calls:</span></span>

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
          var jsInProcess = (WebAssemblyJSRuntime)JS;

          var value = jsInProcess.InvokeUnmarshalled<string>("jsInteropCall");
      }
  }
  ```

  > [!WARNING]
  > <span data-ttu-id="47bc8-145">虽然使用的 <xref:Microsoft.JSInterop.WebAssembly.WebAssemblyJSRuntime> 最小开销是 JS 互操作方法，但当前未记录与这些 api 交互所需的 JavaScript api，但在将来的版本中可能会发生重大更改。</span><span class="sxs-lookup"><span data-stu-id="47bc8-145">While using <xref:Microsoft.JSInterop.WebAssembly.WebAssemblyJSRuntime> has the least overhead of the JS interop approaches, the JavaScript APIs required to interact with these APIs are currently undocumented and subject to breaking changes in future releases.</span></span>

## <a name="reduce-app-size"></a><span data-ttu-id="47bc8-146">降低应用程序大小</span><span class="sxs-lookup"><span data-stu-id="47bc8-146">Reduce app size</span></span>

### <a name="intermediate-language-il-linking"></a><span data-ttu-id="47bc8-147">中间语言（IL）链接</span><span class="sxs-lookup"><span data-stu-id="47bc8-147">Intermediate Language (IL) linking</span></span>

<span data-ttu-id="47bc8-148">[链接 BlazorWebAssembly 应用](xref:host-and-deploy/blazor/configure-linker)通过修整应用二进制文件中未使用的代码来减少应用的大小。</span><span class="sxs-lookup"><span data-stu-id="47bc8-148">[Linking a Blazor WebAssembly app](xref:host-and-deploy/blazor/configure-linker) reduces the app's size by trimming unused code in the app's binaries.</span></span> <span data-ttu-id="47bc8-149">默认情况下，仅当在配置中生成时才启用链接器 `Release` 。</span><span class="sxs-lookup"><span data-stu-id="47bc8-149">By default, the linker is only enabled when building in `Release` configuration.</span></span> <span data-ttu-id="47bc8-150">为此，请使用[dotnet publish](/dotnet/core/tools/dotnet-publish)命令发布应用，并将[-c |--configuration](/dotnet/core/tools/dotnet-publish#options)选项设置为 `Release` ：</span><span class="sxs-lookup"><span data-stu-id="47bc8-150">To benefit from this, publish the app for deployment using the [dotnet publish](/dotnet/core/tools/dotnet-publish) command with the [-c|--configuration](/dotnet/core/tools/dotnet-publish#options) option set to `Release`:</span></span>

```dotnetcli
dotnet publish -c Release
```

### <a name="disable-unused-features"></a><span data-ttu-id="47bc8-151">禁用未使用的功能</span><span class="sxs-lookup"><span data-stu-id="47bc8-151">Disable unused features</span></span>

Blazor<span data-ttu-id="47bc8-152">WebAssembly 的运行时包括以下 .NET 功能，如果应用程序不要求这些功能的负载大小较小，则可以禁用这些功能：</span><span class="sxs-lookup"><span data-stu-id="47bc8-152"> WebAssembly's runtime includes the following .NET features that can be disabled if the app doesn't require them for a smaller payload size:</span></span>

* <span data-ttu-id="47bc8-153">包含数据文件以使时区信息正确。</span><span class="sxs-lookup"><span data-stu-id="47bc8-153">A data file is included to make timezone information correct.</span></span> <span data-ttu-id="47bc8-154">如果应用不需要此功能，请考虑通过将 `BlazorEnableTimeZoneSupport` 应用的项目文件中的 MSBuild 属性设置为来禁用它 `false` ：</span><span class="sxs-lookup"><span data-stu-id="47bc8-154">If the app doesn't require this feature, consider disabling it by setting the `BlazorEnableTimeZoneSupport` MSBuild property in the app's project file to `false`:</span></span>

  ```xml
  <PropertyGroup>
    <BlazorEnableTimeZoneSupport>false</BlazorEnableTimeZoneSupport>
  </PropertyGroup>
  ```

* <span data-ttu-id="47bc8-155">包括排序规则信息以使 Api <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> 正常工作。</span><span class="sxs-lookup"><span data-stu-id="47bc8-155">Collation information is included to make APIs such as <xref:System.StringComparison.InvariantCultureIgnoreCase?displayProperty=nameWithType> work correctly.</span></span> <span data-ttu-id="47bc8-156">如果确定应用不需要排序规则数据，请考虑通过将 `BlazorWebAssemblyPreserveCollationData` 应用的项目文件中的 MSBuild 属性设置为来禁用它 `false` ：</span><span class="sxs-lookup"><span data-stu-id="47bc8-156">If you're certain that the app doesn't require the collation data, consider disabling it by setting the `BlazorWebAssemblyPreserveCollationData` MSBuild property in the app's project file to `false`:</span></span>

  ```xml
  <PropertyGroup>
    <BlazorWebAssemblyPreserveCollationData>false</BlazorWebAssemblyPreserveCollationData>
  </PropertyGroup>
  ```