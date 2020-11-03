---
title: ASP.NET Core Blazor 组件虚拟化
author: guardrex
description: 了解如何在 ASP.NET Core Blazor 应用中使用组件虚拟化。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 10/02/2020
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
uid: blazor/components/virtualization
ms.openlocfilehash: eafad420d72a974cc64ebfd6abb3eff2d73a115d
ms.sourcegitcommit: 139c998d37e9f3e3d0e3d72e10dbce8b75957d89
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/07/2020
ms.locfileid: "91805552"
---
# <a name="aspnet-core-no-locblazor-component-virtualization"></a>ASP.NET Core Blazor 组件虚拟化

作者：[Daniel Roth](https://github.com/danroth27)

使用 Blazor 框架的内置虚拟化支持提高组件呈现的感知性能。 虚拟化是一种技术，用于将 UI 呈现限制为仅当前可见的部分。 例如，当应用必须呈现项的长列表，并且在任何给定的时间只需要一小部分项可见时，虚拟化很有帮助。 Blazor 提供 `Virtualize` 组件，可用于向应用的组件添加虚拟化。

如果不使用虚拟化，典型列表可能会使用 C# [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) 循环来呈现列表中的每一项：

```razor
@foreach (var employee in employees)
{
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
}
```

如果列表包含数千项，则呈现该列表可能会花费较长时间。 用户可能会遇到明显的 UI 延迟。

与其一次性呈现列表中的所有项，不如将 [`foreach`](/dotnet/csharp/language-reference/keywords/foreach-in) 循环替换为 `Virtualize` 组件，并使用 `Items` 指定固定的项源。 这样，将仅呈现当前可见的项：

```razor
<Virtualize Context="employee" Items="@employees">
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
</Virtualize>
```

如果未使用 `Context` 指定组件的上下文，请在项目内容模板中使用 `context` 值 (`@context.{PROPERTY}`)：

```razor
<Virtualize Items="@employees">
    <p>
        @context.FirstName @context.LastName has the 
        job title of @context.JobTitle.
    </p>
</Virtualize>
```

`Virtualize` 组件根据容器的高度和呈现的项的大小来计算要呈现的项数。

`Virtualize` 组件的项内容可以包括：

* 纯 HTML 和 Razor 代码，如前面的示例所示。
* 一个或多个 Razor 组件。
* HTML/Razor 和 Razor 组件的组合。

## <a name="item-provider-delegate"></a>项提供程序委托

如果不想将所有项加载到内存中，可向组件的 `ItemsProvider` 参数指定项提供程序委托方法，以按需异步检索请求的项：

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <p>
        @employee.FirstName @employee.LastName has the 
        job title of @employee.JobTitle.
    </p>
</Virtualize>
```

项提供程序接收 `ItemsProviderRequest`，它指定从特定起始索引开始的请求项数。 然后，项提供程序在数据库或其他服务中检索请求的项，并以 `ItemsProviderResult<TItem>` 形式将这些项与项总数一起返回。 项提供程序可以选择按每个请求检索项，也可以将项缓存以便后续使用。 请勿尝试使用项提供程序来为同一个 `Virtualize` 组件分配 `Items` 的集合。

以下示例从 `EmployeeService` 加载员工：

```csharp
private async ValueTask<ItemsProviderResult<Employee>> LoadEmployees(
    ItemsProviderRequest request)
{
    var numEmployees = Math.Min(request.Count, totalEmployees - request.StartIndex);
    var employees = await EmployeesService.GetEmployeesAsync(request.StartIndex, 
        numEmployees, request.CancellationToken);

    return new ItemsProviderResult<Employee>(employees, totalEmployees);
}
```

## <a name="placeholder"></a>占位符

由于从远程数据源请求项可能需要一些时间，你可以选择呈现占位符 (`<Placeholder>...</Placeholder>`)，直到项数据可用：

```razor
<Virtualize Context="employee" ItemsProvider="@LoadEmployees">
    <ItemContent>
        <p>
            @employee.FirstName @employee.LastName has the 
            job title of @employee.JobTitle.
        </p>
    </ItemContent>
    <Placeholder>
        <p>
            Loading&hellip;
        </p>
    </Placeholder>
</Virtualize>
```

## <a name="item-size"></a>项大小

你可以使用 `ItemSize` 设置每个项的像素大小（默认值：50px）：

```razor
<Virtualize Context="employee" Items="@employees" ItemSize="25">
    ...
</Virtualize>
```

## <a name="overscan-count"></a>溢出扫描计数

`OverscanCount` 确定在可见区域之前和之后呈现的额外项数。 此设置有助于降低滚动期间的呈现频率。 但是，值越大，页面中呈现的元素越多（默认值：3）：

```razor
<Virtualize Context="employee" Items="@employees" OverscanCount="4">
    ...
</Virtualize>
```

## <a name="state-changes"></a>状态更改

当对 `Virtualize` 组件呈现的项进行更改时，将调用 <xref:Microsoft.AspNetCore.Components.ComponentBase.StateHasChanged%2A> 以强制重新计算和重新呈现组件。
