---
title: 在 ASP.NET Core Blazor WebAssembly 中延迟加载程序集
author: guardrex
description: 了解如何在 ASP.NET Core Blazor WebAssembly 应用中延迟加载程序集。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/09/2020
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
uid: blazor/webassembly-lazy-load-assemblies
ms.openlocfilehash: e874ec6f24d8b03fc6c7133013147498cbbc293a
ms.sourcegitcommit: 4febe4efaf6e1a7be65d772b500c00fca0af216a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91451161"
---
# <a name="lazy-load-assemblies-in-aspnet-core-no-locblazor-webassembly"></a>在 ASP.NET Core Blazor WebAssembly 中延迟加载程序集

作者：[Safia Abdalla](https://safia.rocks) 和 [Luke Latham](https://github.com/guardrex)

通过推迟某些应用程序程序集的加载，直到需要时才加载，来提高 Blazor WebAssembly 应用启动性能，这种方式称为“延迟加载”。 例如，只有在用户导航到某个组件时，才将用于呈现该组件的程序集设置为加载。 加载后，程序集被缓存在客户端，供以后的所有导航使用。

使用 Blazor 的延迟加载功能，可以将应用程序集标记为延迟加载，当用户导航到特定路由时，将在运行时加载程序集。 此功能包含对项目文件的更改和对应用程序的路由器的更改。

> [!NOTE]
> 由于程序集不会被下载到 Blazor Server 应用中的客户端，因此程序集的延迟加载对 Blazor Server 应用并没有好处。

## <a name="project-file"></a>项目文件

使用 `BlazorWebAssemblyLazyLoad` 项标记应用的项目文件 (`.csproj`) 中用于延迟加载的程序集。 使用带 `.dll` 扩展名的程序集名称。 Blazor 框架可防止在应用启动时加载由此项组指定的程序集。 下面的示例将一个大型自定义程序集 (`GrantImaharaRobotControls.dll`) 标记为进行延迟加载。 如果标记为进行延迟加载的程序集具有依赖项，还必须在项目文件中同时将这些依赖项标记为延迟加载。

```xml
<ItemGroup>
  <BlazorWebAssemblyLazyLoad Include="GrantImaharaRobotControls.dll" />
</ItemGroup>
```

## <a name="router-component"></a>`Router` 组件

Blazor 的 `Router` 组件指定哪个程序集 Blazor 搜索可路由组件。 `Router` 组件还负责为用户导航的路由呈现组件。 `Router` 组件支持 `OnNavigateAsync` 功能，该功能可与延迟加载一起使用。

在应用的 `Router` 组件 (`App.razor`) 中：

* 添加一个 `OnNavigateAsync` 回调。 当用户执行以下操作时，将调用 `OnNavigateAsync` 处理程序：
  * 通过直接从自己的浏览器导航到某个路由，首次访问该路由。
  * 使用链接或 <xref:Microsoft.AspNetCore.Components.NavigationManager.NavigateTo%2A?displayProperty=nameWithType> 调用导航到一个新路由。
* 如果延迟加载的程序集包含可路由的组件，则向组件添加一个 [List](xref:System.Collections.Generic.List%601)\<<xref:System.Reflection.Assembly>>（例如，命名为 `lazyLoadedAssemblies`）。 如果程序集包含可路由的组件，程序集将被传递回 <xref:Microsoft.AspNetCore.Components.Routing.Router.AdditionalAssemblies> 集合。 框架会搜索程序集中的路由，并在发现任何新路由时更新路由集合。

```razor
@using System.Reflection

<Router AppAssembly="@typeof(Program).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>

@code {
    private List<Assembly> lazyLoadedAssemblies = new List<Assembly>();

    private async Task OnNavigateAsync(NavigationContext args)
    {
    }
}
```

如果 `OnNavigateAsync` 回调引发一个未处理的异常，则会调用 [Blazor 错误 UI](xref:blazor/fundamentals/handle-errors#detailed-errors-during-development)。

### <a name="assembly-load-logic-in-onnavigateasync"></a>`OnNavigateAsync` 中的程序集加载逻辑

`OnNavigateAsync` 有一个 `NavigationContext` 参数，它提供当前异步导航事件的相关信息，包括目标路径 (`Path`) 和取消标记 (`CancellationToken`)：

* `Path` 属性是相对于应用基路径（如 `/robot`）的用户目标路径。
* `CancellationToken` 可以用来观察异步任务的取消。 当用户导航到其他页时，`OnNavigateAsync` 会自动取消当前正在运行的导航任务。

在 `OnNavigateAsync` 内，实现逻辑以确定要加载的程序集。 选项包括：

* `OnNavigateAsync` 方法内部的条件检查。
* 一个将路由映射到程序集名称的查找表，可以注入到组件中，也可以在 [`@code`](xref:mvc/views/razor#code) 块内实现。

`LazyAssemblyLoader` 是一个框架提供的用于加载程序集的单一实例服务。 将 `LazyAssemblyLoader` 注入 `Router` 组件：

```razor
...
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

...
```

`LazyAssemblyLoader` 提供的 `LoadAssembliesAsync` 方法可以：

* 使用 JS 互操作通过网络调用来提取程序集。
* 在浏览器中将程序集加载到正在 WebAssembly 上执行的运行时。

框架的延迟加载实现支持在托管的 Blazor 解决方案中使用预呈现进行延迟加载。 在预呈现期间，所有的程序集（包括那些被标记为延迟加载的程序集）都被假定为已加载。 在服务器项目的 `Startup.ConfigureServices` 方法 (`Startup.cs`) 中手动注册 `LazyAssemblyLoader`：

```csharp
services.AddScoped<LazyAssemblyLoader>();
```

### <a name="user-interaction-with-navigating-content"></a>用户与 `<Navigating>` 内容的交互

加载程序集可能需要几秒钟的时间，`Router` 组件可以向用户指示正在发生页面转换：

* 为 <xref:Microsoft.AspNetCore.Components.Routing?displayProperty=fullName> 命名空间添加一条 [`@using`](xref:mvc/views/razor#using) 指令。
* 向组件添加一个 `<Navigating>` 标记，其中包含在页面转换事件期间显示的标记。

```razor
...
@using Microsoft.AspNetCore.Components.Routing
...

<Router ...>
    <Navigating>
        <div style="...">
            <p>Loading the requested page&hellip;</p>
        </div>
    </Navigating>
</Router>

...
```

### <a name="handle-cancellations-in-onnavigateasync"></a>处理 `OnNavigateAsync` 中的取消

传递到 `OnNavigateAsync` 回调的 `NavigationContext` 对象包含的 `CancellationToken` 在发生新导航事件时进行设置。 设置此取消标记时，`OnNavigateAsync` 回调必须引发，以避免在过时的导航中继续运行 `OnNavigateAsync` 回调。

如果用户导航到路由 A，然后立即路由到 B，则应用不应继续运行路由 A 的 `OnNavigateAsync` 回调：

```razor
@inject HttpClient Http
@inject ProductCatalog Products

<Router AppAssembly="@typeof(Program).Assembly" 
    OnNavigateAsync="@OnNavigateAsync">
    ...
</Router>

@code {
    private async Task OnNavigateAsync(NavigationContext context)
    {
        if (context.Path == "/about") 
        {
            var stats = new Stats = { Page = "/about" };
            await Http.PostAsJsonAsync("api/visited", stats, context.CancellationToken);
        }
        else if (context.Path == "/store")
        {
            var productIds = [345, 789, 135, 689];

            foreach (var productId in productIds) 
            {
                context.CancellationToken.ThrowIfCancellationRequested();
                Products.Prefetch(productId);
            }
        }
    }
}
```

> [!NOTE]
> 如果取消 `NavigationContext` 中的取消标记会导致意外的行为（例如，呈现上一次导航中的组件），则不会引发。

### <a name="onnavigateasync-events-and-renamed-assembly-files"></a>`OnNavigateAsync` 事件和重命名的程序集文件

资源加载程序依赖于在 `blazor.boot.json` 文件中定义的程序集名称。 如果[程序集已重命名](xref:blazor/host-and-deploy/webassembly#change-the-filename-extension-of-dll-files)，则 `OnNavigateAsync` 方法中使用的程序集名称和 `blazor.boot.json` 文件中的程序集名称将不同步。

若要更正此问题，请执行以下操作：

* 确定要使用的程序集名称时，检查应用是否在生产环境中运行。
* 将重命名的程序集名称存储在单独的文件中，并从该文件中读取，以确定要在 `LazyLoadAssemblyService` 和 `OnNavigateAsync` 方法中使用的程序集名称。

### <a name="complete-example"></a>完整示例

下面是完整的 `Router` 组件，它演示在用户导航到 `/robot` 时如何加载 `GrantImaharaRobotControls.dll` 程序集。 在页面转换期间，系统会向用户显示一条带样式的消息。

```razor
@using System.Reflection
@using Microsoft.AspNetCore.Components.Routing
@using Microsoft.AspNetCore.Components.WebAssembly.Services
@inject LazyAssemblyLoader assemblyLoader

<Router AppAssembly="@typeof(Program).Assembly" 
    AdditionalAssemblies="@lazyLoadedAssemblies" OnNavigateAsync="@OnNavigateAsync">
    <Navigating>
        <div style="padding:20px;background-color:blue;color:white">
            <p>Loading the requested page&hellip;</p>
        </div>
    </Navigating>
    <Found Context="routeData">
        <RouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)" />
    </Found>
    <NotFound>
        <LayoutView Layout="@typeof(MainLayout)">
            <p>Sorry, there's nothing at this address.</p>
        </LayoutView>
    </NotFound>
</Router>

@code {
    private List<Assembly> lazyLoadedAssemblies = new List<Assembly>();

    private async Task OnNavigateAsync(NavigationContext args)
    {
        try
        {
            if (args.Path.EndsWith("/robot"))
            {
                var assemblies = await assemblyLoader.LoadAssembliesAsync(
                    new List<string>() { "GrantImaharaRobotControls.dll" });
                lazyLoadedAssemblies.AddRange(assemblies);
            }
        }
        catch (Exception ex)
        {
            ...
        }
    }
}
```

## <a name="troubleshoot"></a>疑难解答

* 如果出现意外的呈现（例如，呈现上一次导航中的组件），请确认在设置取消标记时代码是否引发。
* 如果在应用程序启动时仍会加载程序集，请检查程序集是否在项目文件中被标记为延迟加载。

## <a name="additional-resources"></a>其他资源

* <xref:blazor/webassembly-performance-best-practices>
