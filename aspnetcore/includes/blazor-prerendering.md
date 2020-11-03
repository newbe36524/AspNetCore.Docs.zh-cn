在 Blazor Server 应用进行预呈现时，由于尚未建立与浏览器的连接，因此无法执行调用 JavaScript 等特定操作。 预呈现时，组件可能需要进行不同的呈现。

要将 JavaScript 互操作调用延迟到与浏览器建立连接之后，可使用 [OnAfterRenderAsync 组件生命周期事件](xref:blazor/components/lifecycle#after-component-render)。 仅在完成呈现应用并与客户端建立连接后，才会调用此事件。

```cshtml
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<div @ref="divElement">Text during render</div>

@code {
    private ElementReference divElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender)
        {
            await JSRuntime.InvokeVoidAsync(
                "setElementText", divElement, "Text after render");
        }
    }
}
```

对于上述示例代码，请在 `wwwroot/index.html` (Blazor WebAssembly) 或 `Pages/_Host.cshtml` (Blazor Server) 的 `<head>` 元素中，提供一个 `setElementText` JavaScript 函数。 该函数通过 <xref:Microsoft.JSInterop.JSRuntimeExtensions.InvokeVoidAsync%2A?displayProperty=nameWithType> 进行调用，不返回值：

```html
<script>
  window.setElementText = (element, text) => element.innerText = text;
</script>
```

> [!WARNING]
> 上述示例直接修改文档对象模型 (DOM)，以便仅供演示所用。 大多数情况下，不建议使用 JavaScript 直接修改 DOM，因为 JavaScript 可能会干扰 Blazor 的更改跟踪。

以下组件展示了如何以一种与预呈现兼容的方式将 JavaScript 互操作用作组件初始化逻辑的一部分。 该组件显示可从 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 内部触发呈现更新。 开发人员必须避免在此场景中创建无限循环。

如果调用 <xref:Microsoft.JSInterop.JSRuntime.InvokeAsync%2A?displayProperty=nameWithType>，则 `ElementRef` 仅在 <xref:Microsoft.AspNetCore.Components.ComponentBase.OnAfterRenderAsync%2A> 中使用，而不在任何更早的生命周期方法中使用，因为呈现组件后才会有 JavaScript 元素。

会调用 [StateHasChanged](xref:blazor/components/lifecycle#state-changes)，使用从 JavaScript 互操作调用中获取的新状态重新呈现该组件。 此代码不会创建无限循环，因为仅在 `infoFromJs` 为 `null` 时才调用 `StateHasChanged`。

```cshtml
@page "/prerendered-interop"
@using Microsoft.AspNetCore.Components
@using Microsoft.JSInterop
@inject IJSRuntime JSRuntime

<p>
    Get value via JS interop call:
    <strong id="val-get-by-interop">@(infoFromJs ?? "No value yet")</strong>
</p>

Set value via JS interop call:
<div id="val-set-by-interop" @ref="divElement"></div>

@code {
    private string infoFromJs;
    private ElementReference divElement;

    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        if (firstRender && infoFromJs == null)
        {
            infoFromJs = await JSRuntime.InvokeAsync<string>(
                "setElementText", divElement, "Hello from interop call!");

            StateHasChanged();
        }
    }
}
```

对于上述示例代码，请在 `wwwroot/index.html` (Blazor WebAssembly) 或 `Pages/_Host.cshtml` (Blazor Server) 的 `<head>` 元素中，提供一个 `setElementText` JavaScript 函数。 该函数通过 <xref:Microsoft.JSInterop.IJSRuntime.InvokeAsync%2A?displayProperty=nameWithType> 进行调用，会返回值：

```html
<script>
  window.setElementText = (element, text) => {
    element.innerText = text;
    return text;
  };
</script>
```

> [!WARNING]
> 上述示例直接修改文档对象模型 (DOM)，以便仅供演示所用。 大多数情况下，不建议使用 JavaScript 直接修改 DOM，因为 JavaScript 可能会干扰 Blazor 的更改跟踪。
