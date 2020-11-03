---
title: ASP.NET Core 中的组件标记帮助程序
author: guardrex
ms.author: riande
description: 了解如何使用 ASP.NET Core 组件标记帮助程序 Razor 在页面和视图中呈现组件。
ms.custom: mvc
ms.date: 04/15/2020
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
uid: mvc/views/tag-helpers/builtin-th/component-tag-helper
ms.openlocfilehash: cddbca7f95e4d2143d4632aaa83133bc6210e251
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059151"
---
# <a name="component-tag-helper-in-aspnet-core"></a>ASP.NET Core 中的组件标记帮助程序

作者：[Daniel Roth](https://github.com/danroth27) 和 [Luke Latham](https://github.com/guardrex)

若要从页面或视图呈现组件，请使用[组件标记帮助程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper)。

## <a name="prerequisites"></a>先决条件

按照本文中 " *准备应用程序以使用组件" 页和 "视图* " 部分中的指导进行操作 <xref:blazor/components/integrate-components-into-razor-pages-and-mvc-apps#prepare-the-app> 。

## <a name="component-tag-helper"></a>组件标记帮助程序

以下组件标记帮助程序 `Counter` 在页面或视图中呈现组件：

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using {APP ASSEMBLY}.Pages

...

<component type="typeof(Counter)" render-mode="ServerPrerendered" />
```

前面的示例假定 `Counter` 组件位于应用的 *Pages* 文件夹中。 占位符 `{APP ASSEMBLY}` 是应用的程序集名称（例如 `@using BlazorSample.Pages`）。

组件标记帮助器还可以将参数传递给组件。 请考虑以下 `ColorfulCheckbox` 用于设置复选框标签颜色和大小的组件：

```razor
<label style="font-size:@(Size)px;color:@Color">
    <input @bind="Value"
           id="survey" 
           name="blazor" 
           type="checkbox" />
    Enjoying Blazor?
</label>

@code {
    [Parameter]
    public bool Value { get; set; }

    [Parameter]
    public int Size { get; set; } = 8;

    [Parameter]
    public string Color { get; set; }

    protected override void OnInitialized()
    {
        Size += 10;
    }
}
```

`Size` `int` `Color` `string` 组件标记帮助器可以设置 () 和 () [组件参数](xref:blazor/components/index#component-parameters)：

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using {APP ASSEMBLY}.Shared

...

<component type="typeof(ColorfulCheckbox)" render-mode="ServerPrerendered" 
    param-Size="14" param-Color="@("blue")" />
```

前面的示例假定 `ColorfulCheckbox` 组件位于应用的 *共享* 文件夹中。 占位符 `{APP ASSEMBLY}` 是应用的程序集名称（例如 `@using BlazorSample.Shared`）。

在页面或视图中呈现以下 HTML：

```html
<label style="font-size:24px;color:blue">
    <input id="survey" name="blazor" type="checkbox">
    Enjoying Blazor?
</label>
```

传递带引号的字符串需要 [显式 Razor 表达式](xref:mvc/views/razor#explicit-razor-expressions)，如 `param-Color` 前面的示例中所示。 Razor `string` 由于属性是类型，因此类型值的分析行为不适用于 `param-*` 特性 `object` 。

参数类型必须是 JSON 可序列化的，这通常意味着该类型必须具有默认的构造函数和可设置的属性。 例如，你可以 `Size` `Color` 在前面的示例中指定和的值，因为和的类型 `Size` `Color` 是 (`int` 和 `string`) （JSON 序列化程序支持的）的基元类型。

在下面的示例中，将类对象传递到组件：

*MyClass.cs* ：

```csharp
public class MyClass
{
    public MyClass()
    {
    }

    public int MyInt { get; set; } = 999;
    public string MyString { get; set; } = "Initial value";
}
```

**该类必须具有公共的无参数构造函数。**

*Shared/MyComponent* ：

```razor
<h2>MyComponent</h2>

<p>Int: @MyObject.MyInt</p>
<p>String: @MyObject.MyString</p>

@code
{
    [Parameter]
    public MyClass MyObject { get; set; }
}
```

*Pages/m y* ：

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@using {APP ASSEMBLY}
@using {APP ASSEMBLY}.Shared

...

@{
    var myObject = new MyClass();
    myObject.MyInt = 7;
    myObject.MyString = "Set by MyPage";
}

<component type="typeof(MyComponent)" render-mode="ServerPrerendered" 
    param-MyObject="@myObject" />
```

前面的示例假定 `MyComponent` 组件位于应用的 *共享* 文件夹中。 占位符 `{APP ASSEMBLY}` 是应用程序的程序集名称 (例如， `@using BlazorSample` `@using BlazorSample.Shared`) 。 `MyClass` 在应用的命名空间中。

<xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode> 配置组件是否：

* 在页面中预呈现。
* 在页面上呈现为静态 HTML，或者包含从用户代理启动 Blazor 应用所需的信息。

| 呈现模式 | 描述 |
| ----------- | ----------- |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> | 在静态 HTML 中呈现组件，并包含 Blazor Server 应用的标记。 用户代理启动时，此标记用于启动 Blazor 应用。 |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> | 呈现 Blazor Server 应用的标记。 不包括组件的输出。 用户代理启动时，此标记用于启动 Blazor 应用。 |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Static> | 将组件呈现为静态 HTML。 |

尽管页面和视图可以使用组件，但不是这样。 组件不能使用视图和页特定的功能，如分部视图和节。 若要在组件中通过分部视图使用逻辑，请将分部视图逻辑分解为一个组件。

不支持从静态 HTML 页面呈现服务器组件。

## <a name="additional-resources"></a>其他资源

* <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper>
* <xref:mvc/views/tag-helpers/intro>
* <xref:blazor/components/index>
