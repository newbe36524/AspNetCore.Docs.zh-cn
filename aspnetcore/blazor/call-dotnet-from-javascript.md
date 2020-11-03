---
title: 从 ASP.NET Core Blazor 中的 JavaScript 函数调用 .NET 方法
author: guardrex
description: 了解如何在 Blazor 应用中通过 JavaScript 函数调用 .NET 方法。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 08/12/2020
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
uid: blazor/call-dotnet-from-javascript
ms.openlocfilehash: 4a7c06ed985c290eb4b3ffca6d5ed74c6bc4e031
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690295"
---
# <a name="call-net-methods-from-javascript-functions-in-aspnet-core-no-locblazor"></a>从 ASP.NET Core Blazor 中的 JavaScript 函数调用 .NET 方法

作者：[Javier Calvarro Nelson](https://github.com/javiercn)、[Daniel Roth](https://github.com/danroth27)、[Shashikant Rudrawadi](http://wisne.co) 和 [Luke Latham](https://github.com/guardrex)

Blazor 应用可从 .NET 方法调用 JavaScript 函数，也可从 JavaScript 函数调用 .NET 方法。 这被称为 JavaScript 互操作（JS 互操作） 。

本文介绍如何从 JavaScript 调用 .NET 方法。 要详细了解如何通过 .NET 调用 JavaScript 函数，请参阅 <xref:blazor/call-javascript-from-dotnet>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/common/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="static-net-method-call"></a>静态 .NET 方法调用

要从 JavaScript 调用静态 .NET 方法，请使用 `DotNet.invokeMethod` 或 `DotNet.invokeMethodAsync` 函数。 传入要调用的静态方法的标识符、包含该函数的程序集的名称以及任意自变量。 异步版本是支持 Blazor Server 方案的首选。 .NET 方法必须是公共的静态方法，并且包含 [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute) 特性。 当前不支持调用开放式泛型方法。

该示例应用包含一个 C# 方法，用于返回 `int` 数组。 [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute) 特性应用于方法。

`Pages/JsInterop.razor`：

```razor
<button type="button" class="btn btn-primary"
        onclick="exampleJsFunctions.returnArrayAsyncJs()">
    Trigger .NET static method ReturnArrayAsync
</button>

@code {
    [JSInvokable]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

为客户端提供的 JavaScript 会调用 C# .net 方法。

`wwwroot/exampleJsInterop.js`：

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=8-14)]

如果选择了“`Trigger .NET static method ReturnArrayAsync`”按钮，请在浏览器的 Web 开发人员工具中检查控制台输出。

控制台输出为：

```console
Array(4) [ 1, 2, 3, 4 ]
```

第四个数组值推送到 `ReturnArrayAsync` 返回的数组 (`data.push(4);`)。

默认情况下，方法标识符是方法名称，但你也可以使用 [`[JSInvokable]`](xref:Microsoft.JSInterop.JSInvokableAttribute) 特性构造函数来指定其他标识符：

```csharp
@code {
    [JSInvokable("DifferentMethodName")]
    public static Task<int[]> ReturnArrayAsync()
    {
        return Task.FromResult(new int[] { 1, 2, 3 });
    }
}
```

在客户端 JavaScript 文件中：

```javascript
returnArrayAsyncJs: function () {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'DifferentMethodName')
    .then(data => {
      data.push(4);
      console.log(data);
    });
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

## <a name="instance-method-call"></a>实例方法调用

还可以从 JavaScript 调用 .NET 实例方法。 从 JavaScript 调用 .NET 实例方法：

* 按引用向 JavaScript 传递 .NET 实例：
  * 对 <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A?displayProperty=nameWithType> 进行静态调用。
  * 在 <xref:Microsoft.JSInterop.DotNetObjectReference> 实例中包装实例，并在 <xref:Microsoft.JSInterop.DotNetObjectReference> 实例上调用 <xref:Microsoft.JSInterop.DotNetObjectReference.Create%2A>。 处置 <xref:Microsoft.JSInterop.DotNetObjectReference> 对象（本部分稍后会展示一个示例）。
* 使用 `invokeMethod` 或 `invokeMethodAsync` 函数在实例上调用 .NET 实例方法。 在从 JavaScript 调用其他 .NET 方法时，也可以将 .NET 实例作为自变量传递。

> [!NOTE]
> 示例应用会将消息记录到客户端控制台。 对于示例应用展示的以下示例，请在浏览器的开发人员工具中检查浏览器的控制台输出。

如果选择了“`Trigger .NET instance method HelloHelper.SayHello`”按钮，则 `ExampleJsInterop.CallHelloHelperSayHello` 会被调用并将名称 `Blazor` 传递给方法。

`Pages/JsInterop.razor`：

```razor
<button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
    Trigger .NET instance method HelloHelper.SayHello
</button>

@code {
    public async Task TriggerNetInstanceMethod()
    {
        var exampleJsInterop = new ExampleJsInterop(JSRuntime);
        await exampleJsInterop.CallHelloHelperSayHello("Blazor");
    }
}
```

`CallHelloHelperSayHello` 使用 `HelloHelper` 的新实例调用 JavaScript 函数 `sayHello`。

`JsInteropClasses/ExampleJsInterop.cs`：

[!code-csharp[](./common/samples/3.x/BlazorWebAssemblySample/JsInteropClasses/ExampleJsInterop.cs?name=snippet1&highlight=11-18)]

`wwwroot/exampleJsInterop.js`：

[!code-javascript[](./common/samples/3.x/BlazorWebAssemblySample/wwwroot/exampleJsInterop.js?highlight=15-18)]

该名称将传递给 `HelloHelper` 的构造函数，该构造函数设置 `HelloHelper.Name` 属性。 执行 JavaScript 函数 `sayHello` 时，`HelloHelper.SayHello` 返回 `Hello, {Name}!` 消息，JavaScript 函数将该消息写入控制台。

`JsInteropClasses/HelloHelper.cs`：

[!code-csharp[](./common/samples/3.x/BlazorWebAssemblySample/JsInteropClasses/HelloHelper.cs?name=snippet1&highlight=5,10-11)]

浏览器 Web 开发人员工具中的控制台输出：

```console
Hello, Blazor!
```

为避免内存泄露并允许对创建 <xref:Microsoft.JSInterop.DotNetObjectReference> 的组件进行垃圾回收，请采用以下任一方法：

* 处置类中创建了 <xref:Microsoft.JSInterop.DotNetObjectReference> 实例的对象：

  ```csharp
  public class ExampleJsInterop : IDisposable
  {
      private readonly IJSRuntime jsRuntime;
      private DotNetObjectReference<HelloHelper> objRef;

      public ExampleJsInterop(IJSRuntime jsRuntime)
      {
          this.jsRuntime = jsRuntime;
      }

      public ValueTask<string> CallHelloHelperSayHello(string name)
      {
          objRef = DotNetObjectReference.Create(new HelloHelper(name));

          return jsRuntime.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```

  还可以在组件中实现上述 `ExampleJsInterop` 类中所示的模式：

  ```razor
  @page "/JSInteropComponent"
  @using {APP ASSEMBLY}.JsInteropClasses
  @implements IDisposable
  @inject IJSRuntime JSRuntime

  <h1>JavaScript Interop</h1>

  <button type="button" class="btn btn-primary" @onclick="TriggerNetInstanceMethod">
      Trigger .NET instance method HelloHelper.SayHello
  </button>

  @code {
      private DotNetObjectReference<HelloHelper> objRef;

      public async Task TriggerNetInstanceMethod()
      {
          objRef = DotNetObjectReference.Create(new HelloHelper("Blazor"));

          await JSRuntime.InvokeAsync<string>(
              "exampleJsFunctions.sayHello",
              objRef);
      }

      public void Dispose()
      {
          objRef?.Dispose();
      }
  }
  ```
  
  占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

* 如果组件或类不处置 <xref:Microsoft.JSInterop.DotNetObjectReference>，请通过调用 `.dispose()` 在客户端上处置该对象：

  ```javascript
  window.myFunction = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'MyMethod');
    dotnetHelper.dispose();
  }
  ```

## <a name="component-instance-method-call"></a>组件实例方法调用

要调用组件的 .NET 方法，请执行以下操作：

* 使用 `invokeMethod` 或 `invokeMethodAsync` 函数对组件执行静态方法调用。
* 组件的静态方法将其实例方法调用包装为已调用的 <xref:System.Action>。

> [!NOTE]
> 对于多名用户可能同时使用同一组件的 Blazor Server 应用，请使用帮助程序类来调用实例方法。
>
> 有关详细信息，请参阅[组件实例方法帮助程序类](#component-instance-method-helper-class)部分。

在客户端 JavaScript 中：

```javascript
function updateMessageCallerJS() {
  DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

`Pages/JSInteropComponent.razor`:

```razor
@page "/JSInteropComponent"

<p>
    Message: @message
</p>

<p>
    <button onclick="updateMessageCallerJS()">Call JS Method</button>
</p>

@code {
    private static Action action;
    private string message = "Select the button.";

    protected override void OnInitialized()
    {
        action = UpdateMessage;
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        StateHasChanged();
    }

    [JSInvokable]
    public static void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

若要向实例方法传递参数：

* 向 JS 方法调用添加参数。 在下面的示例中，一个名称被传递给方法。 可根据需要将其他参数添加到列表。

  ```javascript
  function updateMessageCallerJS(name) {
    DotNet.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller', name);
  }
  ```
  
  占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

* 向参数的 <xref:System.Action> 提供正确的类型。 向 C# 方法提供参数列表。 使用参数 (`action.Invoke(name)`) 调用 <xref:System.Action> (`UpdateMessage`)。

  `Pages/JSInteropComponent.razor`:

  ```razor
  @page "/JSInteropComponent"

  <p>
      Message: @message
  </p>

  <p>
      <button onclick="updateMessageCallerJS('Sarah Jane')">
          Call JS Method
      </button>
  </p>

  @code {
      private static Action<string> action;
      private string message = "Select the button.";

      protected override void OnInitialized()
      {
          action = UpdateMessage;
      }

      private void UpdateMessage(string name)
      {
          message = $"{name}, UpdateMessage Called!";
          StateHasChanged();
      }

      [JSInvokable]
      public static void UpdateMessageCaller(string name)
      {
          action.Invoke(name);
      }
  }
  ```

  选择“调用 JS 方法”按钮时输出 `message`：

  ```
  Sarah Jane, UpdateMessage Called!
  ```

## <a name="component-instance-method-helper-class"></a>组件实例方法帮助程序类

帮助程序类用于将实例方法作为 <xref:System.Action> 进行调用。 帮助程序类在以下情况中非常有用：

* 同一类型的多个组件呈现在同一页上。
* 使用 Blazor Server 应用，其中多名用户可能同时使用某个组件。

如下示例中：

* `JSInteropExample` 组件包含若干 `ListItem` 组件。
* 每个 `ListItem` 组件都由一个消息和一个按钮组成。
* 选择 `ListItem` 组件按钮后，`ListItem` 的 `UpdateMessage` 方法会更改列表项文本并隐藏该按钮。

`MessageUpdateInvokeHelper.cs`:

```csharp
using System;
using Microsoft.JSInterop;

public class MessageUpdateInvokeHelper
{
    private Action action;

    public MessageUpdateInvokeHelper(Action action)
    {
        this.action = action;
    }

    [JSInvokable("{APP ASSEMBLY}")]
    public void UpdateMessageCaller()
    {
        action.Invoke();
    }
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

在客户端 JavaScript 中：

```javascript
window.updateMessageCallerJS = (dotnetHelper) => {
    dotnetHelper.invokeMethodAsync('{APP ASSEMBLY}', 'UpdateMessageCaller');
    dotnetHelper.dispose();
}
```

占位符 `{APP ASSEMBLY}` 是应用的应用程序集名称（例如 `BlazorSample`）。

`Shared/ListItem.razor`:

```razor
@inject IJSRuntime JsRuntime

<li>
    @message
    <button @onclick="InteropCall" style="display:@display">InteropCall</button>
</li>

@code {
    private string message = "Select one of these list item buttons.";
    private string display = "inline-block";
    private MessageUpdateInvokeHelper messageUpdateInvokeHelper;

    protected override void OnInitialized()
    {
        messageUpdateInvokeHelper = new MessageUpdateInvokeHelper(UpdateMessage);
    }

    protected async Task InteropCall()
    {
        await JsRuntime.InvokeVoidAsync("updateMessageCallerJS",
            DotNetObjectReference.Create(messageUpdateInvokeHelper));
    }

    private void UpdateMessage()
    {
        message = "UpdateMessage Called!";
        display = "none";
        StateHasChanged();
    }
}
```

`Pages/JSInteropExample.razor`:

```razor
@page "/JSInteropExample"

<h1>List of components</h1>

<ul>
    <ListItem />
    <ListItem />
    <ListItem />
    <ListItem />
</ul>
```

[!INCLUDE[Share interop code in a class library](~/includes/blazor-share-interop-code.md)]

## <a name="avoid-circular-object-references"></a>避免循环引用对象

不能在客户端上针对以下调用就包含循环引用的对象进行序列化：

* .NET 方法调用。
* 返回类型具有循环引用时，从 C# 发出的 JavaScript 方法调用。

有关详细信息，请查看以下问题：

* [循环引用不受支持，使用两个按钮 (dotnet/aspnetcore #20525)](https://github.com/dotnet/aspnetcore/issues/20525)
* [建议：在序列化时添加机制来处理循环引用 (dotnet/runtime #30820)](https://github.com/dotnet/runtime/issues/30820)

## <a name="additional-resources"></a>其他资源

* <xref:blazor/call-javascript-from-dotnet>
* [`InteropComponent.razor` 示例（dotnet/AspNetCore GitHub 存储库，3.1 版本分支）](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Components/test/testassets/BasicTestApp/InteropComponent.razor)
* [在 Blazor Server 应用中执行大型数据传输](xref:blazor/advanced-scenarios#perform-large-data-transfers-in-blazor-server-apps)
