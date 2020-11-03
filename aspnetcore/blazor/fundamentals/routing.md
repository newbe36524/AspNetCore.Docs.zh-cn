---
title: ASP.NET Core Blazor 路由
author: guardrex
description: 了解如何在应用中路由请求以及有关 NavLink 组件的信息。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/02/2020
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
uid: blazor/fundamentals/routing
ms.openlocfilehash: 09e7ca9c03103de116c566352496174e97fbc3ce
ms.sourcegitcommit: a07f83b00db11f32313045b3492e5d1ff83c4437
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90593003"
---
# <a name="aspnet-core-no-locblazor-routing"></a>ASP.NET Core Blazor 路由

作者：[Luke Latham](https://github.com/guardrex)

了解如何路由请求，以及如何使用 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> 组件在 Blazor 应用中创建导航链接。

## <a name="aspnet-core-endpoint-routing-integration"></a>ASP.NET Core 终结点路由集成

Blazor Server 已集成到 [ASP.NET Core 终结点路由](xref:fundamentals/routing)中。 ASP.NET Core 应用配置为接受 `Startup.Configure` 中带有 <xref:Microsoft.AspNetCore.Builder.ComponentEndpointRouteBuilderExtensions.MapBlazorHub%2A> 的交互式组件的传入连接：

[!code-csharp[](routing/samples_snapshot/3.x/Startup.cs?highlight=5)]

最典型的配置是将所有请求路由到 Razor 页面，该页面充当 Blazor Server 应用的服务器端部分的主机。 按照约定，“主机”页通常命名为 `_Host.cshtml`。 主机文件中指定的路由称为*回退路由*，因为它在路由匹配中以较低的优先级运行。 其他路由不匹配时，会考虑回退路由。 这让应用能够使用其他控制器和页面，而不会干扰 Blazor Server 应用。

若要了解如何为非根 URL 服务器托管配置 <xref:Microsoft.AspNetCore.Builder.RazorPagesEndpointRouteBuilderExtensions.MapFallbackToPage%2A>，请参阅 <xref:blazor/host-and-deploy/index#app-base-path>。

## <a name="route-templates"></a>路由模板

<xref:Microsoft.AspNetCore.Components.Routing.Router> 组件可实现到具有指定路由的每个组件的路由。 <xref:Microsoft.AspNetCore.Components.Routing.Router> 组件出现在 `App.razor` 文件中：

```razor
<Router AppAssembly="@typeof(Startup).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <p>Sorry, there's nothing at this address.</p>
    </NotFound>
</Router>
```

编译带有 `@page` 指令的 `.razor` 文件时，将为生成的类提供指定路由模板的 <xref:Microsoft.AspNetCore.Components.RouteAttribute>。

在运行时，<xref:Microsoft.AspNetCore.Components.RouteView> 组件：

* 从 <xref:Microsoft.AspNetCore.Components.Routing.Router> 接收 <xref:Microsoft.AspNetCore.Components.RouteData> 以及任何所需的参数。
* 通过指定参数使用指定组件的布局（或可选的默认布局）呈现该组件。

可选择使用布局类指定 <xref:Microsoft.AspNetCore.Components.RouteView.DefaultLayout> 参数，以用于未指定布局的组件。 默认的 Blazor 模板指定 `MainLayout` 组件。 `MainLayout.razor` 位于模板项目的 `Shared` 文件夹中。 有关布局的详细信息，请参阅 <xref:blazor/layouts>。

可将多个路由模板应用于一个组件。 以下组件响应对 `/BlazorRoute` 和 `/DifferentBlazorRoute` 的请求：

```razor
@page "/BlazorRoute"
@page "/DifferentBlazorRoute"

<h1>Blazor routing</h1>
```

> [!IMPORTANT]
> 若要正确解析 URL，应用必须在其 `wwwroot/index.html` 文件 (Blazor WebAssembly) 或 `Pages/_Host.cshtml` 文件 (Blazor Server) 中加入 `<base>` 标记，并在 `href` 属性 (`<base href="/">`) 中指定应用基路径。 有关详细信息，请参阅 <xref:blazor/host-and-deploy/index#app-base-path>。

## <a name="provide-custom-content-when-content-isnt-found"></a>在找不到内容时提供自定义内容

如果找不到所请求路由的内容，则 <xref:Microsoft.AspNetCore.Components.Routing.Router> 组件允许应用指定自定义内容。

在 `App.razor` 文件中，在 <xref:Microsoft.AspNetCore.Components.Routing.Router> 组件的 <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> 模板参数中设置自定义内容：

```razor
<Router AppAssembly="typeof(Startup).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <h1>Sorry</h1>
        <p>Sorry, there's nothing at this address.</p> b
    </NotFound>
</Router>
```

`<NotFound>` 标记的内容可以包括任意项，例如其他交互式组件。 若要将默认布局应用于 <xref:Microsoft.AspNetCore.Components.Routing.Router.NotFound> 内容，请参阅 <xref:blazor/layouts>。

## <a name="route-to-components-from-multiple-assemblies"></a>从多个程序集路由到组件

使用 <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> 参数为 <xref:Microsoft.AspNetCore.Components.Routing.Router> 组件指定搜索可路由组件时要考虑的其他程序集。 除 `AppAssembly` 指定的程序集外，还要考虑指定的程序集。 在以下示例中，`Component1` 是在引用的类库中定义的可路由组件。 以下 <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> 示例为 `Component1` 提供路由支持：

```razor
<Router
    AppAssembly="@typeof(Program).Assembly"
    AdditionalAssemblies="new[] { typeof(Component1).Assembly }">
    ...
</Router>
```

## <a name="route-parameters"></a>路由参数

路由器使用路由参数以相同的名称填充相应的组件参数（不区分大小写）：

```razor
@page "/RouteParameter"
@page "/RouteParameter/{text}"

<h1>Blazor is @Text!</h1>

@code {
    [Parameter]
    public string Text { get; set; }

    protected override void OnInitialized()
    {
        Text = Text ?? "fantastic";
    }
}
```

不支持可选参数。 上一个示例中应用了两个 `@page` 指令。 第一个指令允许导航到没有参数的组件。 第二个 `@page` 指令采用 `{text}` 路由参数，并将值赋予 `Text` 属性。

## <a name="route-constraints"></a>路由约束

路由约束强制在路由段和组件之间进行类型匹配。

在以下示例中，到 `Users` 组件的路由仅在以下情况下匹配：

* 请求 URL 上存在 `Id` 路由段。
* `Id` 段是整数 (`int`)。

[!code-razor[](routing/samples_snapshot/3.x/Constraint.razor?highlight=1)]

下表中显示的路由约束可用。 有关与固定区域性匹配的路由约束，请参阅表下方的警告了解详细信息。

| 约束 | 示例           | 匹配项示例                                                                  | 固定条件<br>区域性<br>匹配 |
| ---------- | ----------------- | -------------------------------------------------------------------------------- | :------------------------------: |
| `bool`     | `{active:bool}`   | `true`, `FALSE`                                                                  | 否                               |
| `datetime` | `{dob:datetime}`  | `2016-12-31`, `2016-12-31 7:32pm`                                                | 是                              |
| `decimal`  | `{price:decimal}` | `49.99`, `-1,000.01`                                                             | 是                              |
| `double`   | `{weight:double}` | `1.234`, `-1,001.01e8`                                                           | 是                              |
| `float`    | `{weight:float}`  | `1.234`, `-1,001.01e8`                                                           | 是                              |
| `guid`     | `{id:guid}`       | `CD2C1638-1638-72D5-1638-DEADBEEF1638`, `{CD2C1638-1638-72D5-1638-DEADBEEF1638}` | 否                               |
| `int`      | `{id:int}`        | `123456789`, `-123456789`                                                        | 是                              |
| `long`     | `{ticks:long}`    | `123456789`, `-123456789`                                                        | 是                              |

> [!WARNING]
> 验证 URL 的路由约束并将转换为始终使用固定区域性的 CLR 类型（例如 `int` 或 <xref:System.DateTime>）。 这些约束假定 URL 不可本地化。

### <a name="routing-with-urls-that-contain-dots"></a>使用包含点的 URL 进行路由

对于托管的 Blazor WebAssembly 和 Blazor Server 应用，服务器端默认路由模板假定，如果请求 URL 的最后一段包含一个点 (`.`)，则请求一个文件（例如 `https://localhost.com:5001/example/some.thing`）。 在没有额外配置的情况下，应用将返回“404 - 未找到”响应（如果这将路由到组件）。 若要使用具有包含点的一个或多个参数的路由，则应用必须使用自定义模板配置该路由。

请考虑下面的 `Example` 组件，它可以从 URL 的最后一段接收路由参数：

```razor
@page "/example"
@page "/example/{param}"

<p>
    Param: @Param
</p>

@code {
    [Parameter]
    public string Param { get; set; }
}
```

若要允许托管的 Blazor WebAssembly 解决方案的服务器应用路由 `param` 参数中包含一个点的请求，请添加一个回退文件路由模板，在该模板的 `Startup.Configure` (`Startup.cs`) 中包含该可选参数：

```csharp
endpoints.MapFallbackToFile("/example/{param?}", "index.html");
```

若要配置 Blazor Server 应用以在 `param` 参数中使用一个点来路由请求，请添加一个回退页面路由模板，该模板具有 `Startup.Configure` (`Startup.cs`) 中的可选参数：

```csharp
endpoints.MapFallbackToPage("/example/{param?}", "/_Host");
```

有关详细信息，请参阅 <xref:fundamentals/routing>。

## <a name="catch-all-route-parameters"></a>catch-all 路由参数

::: moniker range=">= aspnetcore-5.0"

*本部分应用于 .NET 5 候选发布 1 (RC1) 或更高版本中的 ASP.NET Core。*

组件支持可跨多个文件夹边界捕获路径的 catch-all 路由参数。 catch-all 路由参数必须满足以下条件：

* 以与路由段名称匹配的方式命名。 命名不区分大小写。
* `string` 类型。 框架不提供自动强制转换。
* 位于 URL 的末尾。

```razor
@page "/page/{*pageRoute}"

@code {
    [Parameter]
    public string PageRoute { get; set; }
}
```

对于具有 `/page/{*pageRoute}` 路由模板的 URL `/page/this/is/a/test`，`PageRoute` 的值设置为 `this/is/a/test`。

对捕获路径的斜杠和段进行解码。 对于 `/page/{*pageRoute}` 的路由模板，URL `/page/this/is/a%2Ftest%2A` 会生成 `this/is/a/test*`。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

.NET 5 候选发布 1 (RC1) 或更高版本的 ASP.NET Core 中支持 catch-all 路由参数。

::: moniker-end

## <a name="navlink-component"></a>NavLink 组件

创建导航链接时，请使用 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> 组件代替 HTML 超链接元素 (`<a>`)。 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> 组件的行为方式类似于 `<a>` 元素，但它根据其 `href` 是否与当前 URL 匹配来切换 `active` CSS 类。 `active` 类可帮助用户了解所显示导航链接中的哪个页面是活动页面。 也可以选择将 CSS 类名分配到 <xref:Microsoft.AspNetCore.Components.Routing.NavLink.ActiveClass?displayProperty=nameWithType>，以便在当前路由与 `href` 匹配时将自定义 CSS 类应用到呈现的链接。

以下 `NavMenu` 组件创建 [`Bootstrap`](https://getbootstrap.com/docs/) 导航栏，该导航栏演示如何使用 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> 组件：

[!code-razor[](routing/samples_snapshot/3.x/NavMenu.razor?highlight=4,9)]

有两个 <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch> 选项可分配给 `<NavLink>` 元素的 `Match` 属性：

* <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.All?displayProperty=nameWithType>：<xref:Microsoft.AspNetCore.Components.Routing.NavLink> 在与当前整个 URL 匹配的情况下处于活动状态。
* <xref:Microsoft.AspNetCore.Components.Routing.NavLinkMatch.Prefix?displayProperty=nameWithType>（默认）：<xref:Microsoft.AspNetCore.Components.Routing.NavLink> 在与当前 URL 的任何前缀匹配的情况下处于活动状态。

在前面的示例中，主页 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> `href=""` 与主页 URL 匹配，并且仅在应用的默认基路径 URL（例如，`https://localhost:5001/`）处接收 `active` CSS 类。 当用户访问带有 `MyComponent` 前缀的任何 URL（例如，`https://localhost:5001/MyComponent` 和 `https://localhost:5001/MyComponent/AnotherSegment`）时，第二个 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> 接收 `active` 类。

其他 <xref:Microsoft.AspNetCore.Components.Routing.NavLink> 组件属性会传递到呈现的定位标记。 在以下示例中，<xref:Microsoft.AspNetCore.Components.Routing.NavLink> 组件包括 `target` 属性：

```razor
<NavLink href="my-page" target="_blank">My page</NavLink>
```

呈现以下 HTML 标记：

```html
<a href="my-page" target="_blank">My page</a>
```

> [!WARNING]
> 由于 Blazor 呈现子内容的方式，如果在 `NavLink`（子）组件的内容中使用递增循环变量，则在 `for` 循环内呈现 `NavLink` 组件需要本地索引变量：
>
> ```razor
> @for (int c = 0; c < 10; c++)
> {
>     var current = c;
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @current
>         </NavLink>
>     </li>
> }
> ```
>
> 在[子内容](xref:blazor/components/index#child-content)中使用循环变量的任何子组件（而不仅仅是 `NavLink` 组件）都要求必须在此方案中使用索引变量。
>
> 或者，将 `foreach` 循环用于 <xref:System.Linq.Enumerable.Range%2A?displayProperty=nameWithType>：
>
> ```razor
> @foreach(var c in Enumerable.Range(0,10))
> {
>     <li ...>
>         <NavLink ... href="@c">
>             <span ...></span> @c
>         </NavLink>
>     </li>
> }
> ```

## <a name="uri-and-navigation-state-helpers"></a>URI 和导航状态帮助程序

在 C# 代码中将 <xref:Microsoft.AspNetCore.Components.NavigationManager> 与 URI 和导航配合使用。 <xref:Microsoft.AspNetCore.Components.NavigationManager> 提供下表所示的事件和方法。

| 成员 | 描述 |
| ------ | ----------- |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri> | 获取当前绝对 URI。 |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> | 获取可在相对 URI 路径之前添加用于生成绝对 URI 的基 URI（带有尾部反斜杠）。 通常，<xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> 对应于 `wwwroot/index.html` (Blazor WebAssembly) 或 `Pages/_Host.cshtml` (Blazor Server) 中文档的 `<base>` 元素上的 `href` 属性。 |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A> | 导航到指定 URI。 如果 `forceLoad` 为 `true`，则：<ul><li>客户端路由会被绕过。</li><li>无论 URI 是否通常由客户端路由器处理，浏览器都必须从服务器加载新页面。</li></ul> |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged> | 导航位置更改时触发的事件。 |
| <xref:Microsoft.AspNetCore.Components.NavigationManager.ToAbsoluteUri%2A> | 将相对 URI 转换为绝对 URI。 |
| <span style="word-break:normal;word-wrap:normal"><xref:Microsoft.AspNetCore.Components.NavigationManager.ToBaseRelativePath%2A></span> | 给定基 URI（例如，之前由 <xref:Microsoft.AspNetCore.Components.NavigationManager.BaseUri> 返回的 URI），将绝对 URI 转换为相对于基 URI 前缀的 URI。 |

选择该按钮后，以下组件导航到应用的 `Counter` 组件：

```razor
@page "/navigate"
@inject NavigationManager NavigationManager

<h1>Navigate in Code Example</h1>

<button class="btn btn-primary" @onclick="NavigateToCounterComponent">
    Navigate to the Counter component
</button>

@code {
    private void NavigateToCounterComponent()
    {
        NavigationManager.NavigateTo("counter");
    }
}
```

以下组件通过订阅 <xref:Microsoft.AspNetCore.Components.NavigationManager.LocationChanged?displayProperty=nameWithType> 来处理位置改变事件。 在框架调用 `Dispose` 时，解除挂接 `HandleLocationChanged` 方法。 解除挂接该方法可允许组件进行垃圾回收。

```razor
@implements IDisposable
@inject NavigationManager NavigationManager

...

protected override void OnInitialized()
{
    NavigationManager.LocationChanged += HandleLocationChanged;
}

private void HandleLocationChanged(object sender, LocationChangedEventArgs e)
{
    ...
}

public void Dispose()
{
    NavigationManager.LocationChanged -= HandleLocationChanged;
}
```

<xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs> 可提供以下有关该事件的信息：

* <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.Location>：新位置的 URL。
* <xref:Microsoft.AspNetCore.Components.Routing.LocationChangedEventArgs.IsNavigationIntercepted>：如果为 `true`，则 Blazor 拦截了浏览器中的导航。 如果为 `false`，则 <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> 导致了导航发生。

要详细了解组件处置，请参阅 <xref:blazor/components/lifecycle#component-disposal-with-idisposable>。

## <a name="query-string-and-parse-parameters"></a>查询字符串和分析参数

可以从 <xref:Microsoft.AspNetCore.Components.NavigationManager> 的 <xref:Microsoft.AspNetCore.Components.NavigationManager.Uri> 属性中获取请求的查询字符串：

```razor
@inject NavigationManager Navigation

...

var query = new Uri(Navigation.Uri).Query;
```

若要分析查询字符串的参数，请执行以下操作：

* 为 [Microsoft.AspNetCore.WebUtilities](https://www.nuget.org/packages/Microsoft.AspNetCore.WebUtilities) 添加包引用。
* 在使用 <xref:Microsoft.AspNetCore.WebUtilities.QueryHelpers.ParseQuery%2A?displayProperty=nameWithType> 分析查询字符串后获取值。

```razor
@page "/"
@using Microsoft.AspNetCore.WebUtilities
@inject NavigationManager NavigationManager

<h1>Query string parse example</h1>

<p>Value: @queryValue</p>

@code {
    private string queryValue = "Not set";

    protected override void OnInitialized()
    {
        var query = new Uri(NavigationManager.Uri).Query;

        if (QueryHelpers.ParseQuery(query).TryGetValue("{KEY}", out var value))
        {
            queryValue = value;
        }
    }
}
```

前面示例中的占位符 `{KEY}` 是查询字符串参数键。 例如，URL 键值对 `?ship=Tardis` 使用键 `ship`。
