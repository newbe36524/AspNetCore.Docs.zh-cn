---
title: ASP.NET Core Blazor 数据绑定
author: guardrex
description: 了解 Blazor 应用中组件和 DOM 元素的数据绑定功能。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/19/2020
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
uid: blazor/components/data-binding
ms.openlocfilehash: 0884b0bedd9ed31b8c85790c6950c7c5d63bdf44
ms.sourcegitcommit: e519d95d17443abafba8f712ac168347b15c8b57
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/02/2020
ms.locfileid: "91653901"
---
# <a name="aspnet-core-no-locblazor-data-binding"></a>ASP.NET Core Blazor 数据绑定

作者：[Luke Latham](https://github.com/guardrex)、[Daniel Roth](https://github.com/danroth27) 和 [Steve Sanderson](https://github.com/SteveSandersonMS)

Razor 组件通过名为 [`@bind`](xref:mvc/views/razor#bind) 的 HTML 元素特性提供了数据绑定功能，该特性具有字段、属性或 Razor 表达式值。

下面的示例将 `<input>` 元素绑定到 `currentValue` 字段，将`<input>` 元素绑定到 `CurrentValue` 属性：

```razor
<p>
    <input @bind="currentValue" /> Current value: @currentValue
</p>

<p>
    <input @bind="CurrentValue" /> Current value: @CurrentValue
</p>

@code {
    private string currentValue;

    private string CurrentValue { get; set; }
}
```

当其中一个元素失去焦点时，将更新其绑定字段或属性。

仅当呈现组件时（而不是响应字段或属性值更改），才会在 UI 中更新文本框。 由于组件会在事件处理程序代码执行之后呈现自己，因此在触发事件处理程序之后，通常会在 UI 中反映字段和属性更新。

将 [`@bind`](xref:mvc/views/razor#bind) 与 `CurrentValue` 属性结合使用 (`<input @bind="CurrentValue" />`) 在本质上等效于以下内容：

```razor
<input value="@CurrentValue"
    @onchange="@((ChangeEventArgs __e) => CurrentValue = 
        __e.Value.ToString())" />

@code {
    private string CurrentValue { get; set; }
}
```

呈现组件时，输入元素的 `value` 来自 `CurrentValue` 属性。 用户在文本框中键入并更改元素焦点时，会激发 `onchange` 事件并将 `CurrentValue` 属性设置为更改的值。 实际上，代码生成会比这更加复杂，因为 [`@bind`](xref:mvc/views/razor#bind) 会处理执行类型转换的情况。 原则上，[`@bind`](xref:mvc/views/razor#bind) 将表达式的当前值与 `value` 属性关联，并使用注册的处理程序处理更改。

通过同时包含带有 `event` 参数的 `@bind:event` 属性，在其他事件上绑定属性或字段。 以下示例在 `oninput` 事件上绑定 `CurrentValue` 属性：

```razor
<input @bind="CurrentValue" @bind:event="oninput" />

@code {
    private string CurrentValue { get; set; }
}
```

与在元素失去焦点时激发的 `onchange` 不同，`oninput` 在文本框的值更改时激发。

<!-- Hold location for resolution of https://github.com/dotnet/AspNetCore.Docs/issues/19721 -->

属性绑定区分大小写：

* `@bind` 有效。
* `@Bind` 和 `@BIND` 无效。

## <a name="unparsable-values"></a>无法分析的值

如果用户向数据绑定元素提供无法分析的值，则在触发绑定事件时，无法分析的值会自动还原为以前的值。

请参考以下方案：

* `<input>` 元素绑定到 `int` 类型，其初始值为 `123`：

  ```razor
  <input @bind="inputValue" />

  @code {
      private int inputValue = 123;
  }
  ```

* 用户在页面中将该元素的值更新为 `123.45`，并更改元素焦点。

在上面的方案中，元素的值会还原为 `123`。 如果拒绝值 `123.45` 以采用原始值 `123`，则用户会了解其值不被接受。

默认情况下，绑定适用于元素的 `onchange` 事件 (`@bind="{PROPERTY OR FIELD}"`)。 使用 `@bind="{PROPERTY OR FIELD}" @bind:event={EVENT}` 对其他事件触发绑定。 对于 `oninput` 事件 (`@bind:event="oninput"`)，在任何引入无法分析的值的击键之后，会进行还原。 当使用 `int` 绑定类型以 `oninput` 事件为目标时，会阻止用户键入 `.` 字符。 `.` 字符会立即删除，因此用户会收到仅允许整数的即时反馈。 在某些情况下，在 `oninput` 事件中还原值并不理想，例如在应该允许用户清除无法解析的 `<input>` 值时。 替代方案包括：

* 不使用 `oninput` 事件。 使用默认 `onchange` 事件（仅指定 `@bind="{PROPERTY OR FIELD}"`），其中无效值在元素失去焦点之前不会还原。
* 绑定到可以为 null 的类型（如 `int?` 或 `string`），并提供自定义逻辑来处理无效条目。
* 使用[窗体验证组件](xref:blazor/forms-validation)，如 <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> 或 <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601>。 窗体验证组件具有用于管理无效输入的内置支持。 有关详细信息，请参阅 <xref:blazor/forms-validation>。 窗体验证组件：
  * 允许用户提供无效输入并在关联的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 上接收验证错误。
  * 在 UI 中显示验证错误，而不干扰用户输入其他 webform 数据。

## <a name="format-strings"></a>格式字符串

数据绑定使用 `@bind:format` 处理 <xref:System.DateTime> 格式字符串。 目前无法使用其他格式表达式，如货币或数字格式。

```razor
<input @bind="startDate" @bind:format="yyyy-MM-dd" />

@code {
    private DateTime startDate = new DateTime(2020, 1, 1);
}
```

在前面的代码中，`<input>` 元素的字段类型 (`type`) 默认为 `text`。 对于绑定以下 .NET 类型，支持 `@bind:format`：

* <xref:System.DateTime?displayProperty=fullName>
* <xref:System.DateTime?displayProperty=fullName>?
* <xref:System.DateTimeOffset?displayProperty=fullName>
* <xref:System.DateTimeOffset?displayProperty=fullName>?

`@bind:format` 属性指定要应用于 `<input>` 元素的 `value` 的日期格式。 该格式还用于在 `onchange` 事件发生时分析值。

不建议为 `date` 字段类型指定格式，因为 Blazor 具有用于设置日期格式的内置支持。 尽管提出了建议，但如果使用 `date` 字段类型提供格式，则只有使用 `yyyy-MM-dd` 日期格式才能使绑定正常工作：

```razor
<input type="date" @bind="startDate" @bind:format="yyyy-MM-dd">
```

## <a name="parent-to-child-binding-with-component-parameters"></a>使用组件参数的父级到子级绑定

通过组件参数，可使用 `@bind-{PROPERTY OR FIELD}` 语法绑定父组件的属性和字段。

以下 `Child` 组件 (`Shared/Child.razor`) 具有 `Year` 组件参数和 `YearChanged` 回调：

```razor
<div class="card bg-light mt-3" style="width:18rem ">
    <div class="card-body">
        <h3 class="card-title">Child Component</h3>
        <p class="card-text">Child <code>Year</code>: @Year</p>
    </div>
</div>

@code {
    [Parameter]
    public int Year { get; set; }

    [Parameter]
    public EventCallback<int> YearChanged { get; set; }
}
```

回调 (<xref:Microsoft.AspNetCore.Components.EventCallback%601>) 必须命名为组件参数名后跟“`Changed`”后缀 (`{PARAMETER NAME}Changed`)。 在上一示例中，回调名为 `YearChanged`。 有关 <xref:Microsoft.AspNetCore.Components.EventCallback%601> 的详细信息，请参阅 <xref:blazor/components/event-handling#eventcallback>。

在下面的 `Parent` 组件 (`Parent.razor`) 中，`year` 字段绑定到子组件的 `Year` 参数：

```razor
@page "/Parent"

<h1>Parent Component</h1>

<p>Parent <code>year</code>: @year</p>

<button @onclick="UpdateYear">Update Parent <code>year</code></button>

<Child @bind-Year="year" />

@code {
    private Random r = new Random();
    private int year = 1979;

    private void UpdateYear()
    {
        year = r.Next(1950, 2021);
    }
}
```

`Year` 参数是可绑定的，因为它具有与 `Year` 参数类型相匹配的伴随 `YearChanged` 事件。

按照约定，可通过包含分配到处理程序的 `@bind-{PROPERTY}:event` 特性，将属性绑定到对应的事件处理程序。 `<Child @bind-Year="year" />` 等效于此写入：

```razor
<Child @bind-Year="year" @bind-Year:event="YearChanged" />
```

## <a name="child-to-parent-binding-with-chained-bind"></a>使用链接绑定的子级到父级绑定

一种常见方案是将数据绑定参数链接到组件输出中的页面元素。 此方案称为链接绑定，因为多个级别的绑定会同时进行。

无法在子组件中使用 [`@bind`](xref:mvc/views/razor#bind) 语法实现链接绑定。 必须单独指定事件处理程序和值。 但是，父组件可将 [`@bind`](xref:mvc/views/razor#bind) 语法用于子组件的参数。

以下 `PasswordField` 组件 (`PasswordField.razor`)：

* 将 `<input>` 元素的值设置为 `password` 字段。
* 将 `Password` 属性的更改公开给父组件，其中 [`EventCallback`](xref:blazor/components/event-handling#eventcallback) 以子级 `password` 字段的当前值作为参数传递。
* 使用 `onclick` 事件触发 `ToggleShowPassword` 方法。 有关详细信息，请参阅 <xref:blazor/components/event-handling>。

```razor
<h1>Provide your password</h1>

Password:

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

@code {
    private bool showPassword;
    private string password;

    [Parameter]
    public string Password { get; set; }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        password = e.Value.ToString();

        return PasswordChanged.InvokeAsync(password);
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

`PasswordField` 组件在另一个组件中使用：

```razor
@page "/Parent"

<h1>Parent Component</h1>

<PasswordField @bind-Password="password" />

@code {
    private string password;
}
```

在调用绑定的委托的方法中执行检查或捕获错误。 如果密码的值中使用了空格，则以下示例会向用户提供即时反馈：

```razor
<h1>Child Component</h1>

Password: 

<input @oninput="OnPasswordChanged" 
       required 
       type="@(showPassword ? "text" : "password")" 
       value="@password" />

<button class="btn btn-primary" @onclick="ToggleShowPassword">
    Show password
</button>

<span class="text-danger">@validationMessage</span>

@code {
    private bool showPassword;
    private string password;
    private string validationMessage;

    [Parameter]
    public string Password { get; set; }

    [Parameter]
    public EventCallback<string> PasswordChanged { get; set; }

    private Task OnPasswordChanged(ChangeEventArgs e)
    {
        password = e.Value.ToString();
        
        if (password.Contains(' '))
        {
            validationMessage = "Spaces not allowed!";

            return Task.CompletedTask;
        }
        else
        {
            validationMessage = string.Empty;

            return PasswordChanged.InvokeAsync(password);
        }
    }

    private void ToggleShowPassword()
    {
        showPassword = !showPassword;
    }
}
```

## <a name="bind-across-more-than-two-components"></a>绑定到两个以上的组件

可以绑定到任意数量的嵌套组件，但必须采用单向数据流：

* 更改通知沿层次结构向上传递。
* 新参数值按层次结构向下传递。

推荐的常见方法是仅将基础数据存储在父组件中，以避免对必须更新的状态产生任何混淆。

下面的组件演示了上述概念：

`ParentComponent.razor`:

```razor
<h1>Parent Component</h1>

<p>Parent Property: <b>@parentValue</b></p>

<p>
    <button @onclick="ChangeValue">Change from Parent</button>
</p>

<ChildComponent @bind-Property="parentValue" />

@code {
    private string parentValue = "Initial value set in Parent";

    private void ChangeValue()
    {
        parentValue = $"Set in Parent {DateTime.Now}";
    }
}
```

`ChildComponent.razor`:

```razor
<div class="border rounded m-1 p-1">
    <h2>Child Component</h2>

    <p>Child Property: <b>@Property</b></p>

    <p>
        <button @onclick="ChangeValue">Change from Child</button>
    </p>

    <GrandchildComponent @bind-Property="BoundValue" />
</div>

@code {
    [Parameter]
    public string Property { get; set; }

    [Parameter]
    public EventCallback<string> PropertyChanged { get; set; }

    private string BoundValue
    {
        get => Property;
        set => PropertyChanged.InvokeAsync(value);
    }

    private Task ChangeValue()
    {
        return PropertyChanged.InvokeAsync($"Set in Child {DateTime.Now}");
    }
}
```

`GrandchildComponent.razor`:

```razor
<div class="border rounded m-1 p-1">
    <h3>Grandchild Component</h3>

    <p>Grandchild Property: <b>@Property</b></p>

    <p>
        <button @onclick="ChangeValue">Change from Grandchild</button>
    </p>
</div>

@code {
    [Parameter]
    public string Property { get; set; }

    [Parameter]
    public EventCallback<string> PropertyChanged { get; set; }

    private Task ChangeValue()
    {
        return PropertyChanged.InvokeAsync($"Set in Grandchild {DateTime.Now}");
    }
}
```

有关适用于不必嵌套的组件在内存中共享数据的可选方法，请参阅 <xref:blazor/state-management#in-memory-state-container-service>。

## <a name="additional-resources"></a>其他资源

* [绑定到窗体中的单选按钮](xref:blazor/forms-validation#radio-buttons)
* [将 `<select>` 元素选项绑定到窗体中的 C# 对象 `null` 值](xref:blazor/forms-validation#binding-select-element-options-to-c-object-null-values)
