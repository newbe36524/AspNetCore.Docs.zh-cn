---
title: ASP.NET Core MVC 中的缓存标记帮助程序
author: pkellner
description: 了解如何使用缓存标记帮助程序。
ms.author: riande
ms.custom: mvc
ms.date: 10/10/2018
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
uid: mvc/views/tag-helpers/builtin-th/cache-tag-helper
ms.openlocfilehash: a87f91255bd1f280b1567f522423a6f4e88a6dd8
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060880"
---
# <a name="cache-tag-helper-in-aspnet-core-mvc"></a>ASP.NET Core MVC 中的缓存标记帮助程序

作者：[Peter Kellner](https://peterkellner.net)

缓存标记帮助程序通过将其内容缓存到内部 ASP.NET Core 缓存提供程序中，极大地提高了 ASP.NET Core 应用的性能。

有关标记帮助程序的概述，请参阅 <xref:mvc/views/tag-helpers/intro>。

以下 Razor 标记将缓存当前日期：

```cshtml
<cache>@DateTime.Now</cache>
```

对包含标记帮助程序的页面的第一个请求显示当前日期。 其他请求将显示已缓存的值，直到缓存过期（默认 20 分钟）或因内存压力而逐出。

## <a name="cache-tag-helper-attributes"></a>缓存标记帮助程序属性

### <a name="enabled"></a>enabled

| 属性类型  | 示例        | 默认 |
| --------------- | --------------- | ------- |
| 布尔         | `true`, `false` | `true`  |

`enabled` 确定是否缓存了缓存标记帮助程序所包含的内容。 默认值为 `true`。 如果设置为 `false`，则不会缓存呈现的输出  。

示例：

```cshtml
<cache enabled="true">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="expires-on"></a>expires-on

| 属性类型   | 示例                            |
| ---------------- | ---------------------------------- |
| `DateTimeOffset` | `@new DateTime(2025,1,29,17,02,0)` |

`expires-on` 为缓存项设置一个绝对到期日期。

以下示例将在 2025 年 1 月 29 日下午 5:02 之前缓存缓存标记帮助程序的内容：

```cshtml
<cache expires-on="@new DateTime(2025,1,29,17,02,0)">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="expires-after"></a>expires-after

| 属性类型 | 示例                      | 默认    |
| -------------- | ---------------------------- | ---------- |
| `TimeSpan`     | `@TimeSpan.FromSeconds(120)` | 20 分钟 |

`expires-after` 设置从第一个请求时间到缓存内容的时间长度。

示例：

```cshtml
<cache expires-after="@TimeSpan.FromSeconds(120)">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

Razor视图引擎将默认值设置 `expires-after` 为20分钟。

### <a name="expires-sliding"></a>expires-sliding

| 属性类型 | 示例                     |
| -------------- | --------------------------- |
| `TimeSpan`     | `@TimeSpan.FromSeconds(60)` |

设置某个缓存项的值未被访问时，该缓存项应被逐出的时间。

示例：

```cshtml
<cache expires-sliding="@TimeSpan.FromSeconds(60)">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-header"></a>vary-by-header

| 属性类型 | 示例                                    |
| -------------- | ------------------------------------------- |
| String         | `User-Agent`, `User-Agent,content-encoding` |

`vary-by-header` 接受逗号分隔的标头值列表，在标头值发生更改时触发缓存刷新。

以下示例监视标头值 `User-Agent`。 该示例将缓存提供给 Web 服务器的每个不同 `User-Agent` 的内容：

```cshtml
<cache vary-by-header="User-Agent">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-query"></a>vary-by-query

| 属性类型 | 示例             |
| -------------- | -------------------- |
| String         | `Make`, `Make,Model` |

`vary-by-query` 接受查询字符串(<xref:Microsoft.AspNetCore.Http.HttpRequest.Query*>) 中逗号分隔的 <xref:Microsoft.AspNetCore.Http.IQueryCollection.Keys*> 列表，它们在任何列出的键值发生更改时触发缓存刷新。

以下示例监视 `Make` 和 `Model` 值。 该示例将缓存提供给 Web 服务器的每个不同 `Make` 和 `Model` 的内容：

```cshtml
<cache vary-by-query="Make,Model">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-route"></a>vary-by-route

| 属性类型 | 示例             |
| -------------- | -------------------- |
| String         | `Make`, `Make,Model` |

`vary-by-route` 接受路由参数名称的逗号分隔列表，用于在路由数据参数值发生更改时触发缓存刷新。

示例：

*Startup.cs* ：

```csharp
routes.MapRoute(
    name: "default",
    template: "{controller=Home}/{action=Index}/{Make?}/{Model?}");
```

*索引 cshtml* ：

```cshtml
<cache vary-by-route="Make,Model">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-no-loccookie"></a>依cookie

| 属性类型 | 示例                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| String         | `.AspNetCore.Identity.Application`, `.AspNetCore.Identity.Application,HairColor` |

`vary-by-cookie` 接受以逗号分隔的名称列表 cookie ，这些名称会在值更改时触发缓存刷新 cookie 。

下面的示例将监视 cookie 与关联的 ASP.NET Core Identity 。 对用户进行身份验证时，中的更改会 Identity cookie 触发缓存刷新：

```cshtml
<cache vary-by-cookie=".AspNetCore.Identity.Application">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="vary-by-user"></a>vary-by-user

| 属性类型  | 示例        | 默认 |
| --------------- | --------------- | ------- |
| 布尔         | `true`, `false` | `true`  |

`vary-by-user` 指定当已登录用户（或上下文主体）发生更改时是否应重置缓存。 当前用户也称为请求上下文主体，可以 Razor 通过引用在视图中查看 `@User.Identity.Name` 。

下面的示例监视当前登录的用户触发缓存刷新：

```cshtml
<cache vary-by-user="true">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

通过登录和注销周期，使用此属性将内容维护在缓存中。 当值设置为 `true` 时，身份验证周期会使已经过身份验证的用户的缓存失效。 缓存会失效，因为 cookie 在对用户进行身份验证时，会生成一个新的唯一值。 如果不 cookie 存在或已过期，则会为匿名状态维护缓存 cookie 。 如果用户未经过  身份验证，则会维持缓存。

### <a name="vary-by"></a>vary-by

| 属性类型 | 示例  |
| -------------- | -------- |
| String         | `@Model` |

`vary-by` 允许自定义缓存的数据。 当属性的字符串值引用的对象发生更改时，会更新缓存标记帮助程序的内容。 通常将模型值的字符串串联分配给此属性。 从效果上看，这导致了更新任何已连接的值都会使缓存无效。

以下示例假定视图的控制器方法将两个路由参数 `myParam1` 和 `myParam2` 的整数值相加，并将其作为单个模型属性返回。 当此总和更改时，会再次呈现并缓存缓存标记帮助程序的内容。  

操作：

```csharp
public IActionResult Index(string myParam1, string myParam2, string myParam3)
{
    int num1;
    int num2;
    int.TryParse(myParam1, out num1);
    int.TryParse(myParam2, out num2);
    return View(viewName, num1 + num2);
}
```

*索引 cshtml* ：

```cshtml
<cache vary-by="@Model">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

### <a name="priority"></a>priority

| 属性类型      | 示例                               | 默认  |
| ------------------- | -------------------------------------- | -------- |
| `CacheItemPriority` | `High`, `Low`, `NeverRemove`, `Normal` | `Normal` |

`priority` 为内置缓存提供程序提供缓存逐出指导。 在内存压力下，Web 服务器将首先逐出 `Low` 缓存项。

示例：

```cshtml
<cache priority="High">
    Current Time Inside Cache Tag Helper: @DateTime.Now
</cache>
```

`priority` 属性并不能保证特定级别的缓存保留。 `CacheItemPriority` 仅供参考。 将此属性设置为 `NeverRemove` 并不能保证缓存项将始终保留。 请参阅[其他资源](#additional-resources)部分中的相关主题，以获取详细信息。

缓存标记帮助程序依赖于[内存缓存服务](xref:performance/caching/memory)。 如果尚未添加该服务，缓存标记帮助程序将为你添加。

## <a name="additional-resources"></a>其他资源

* <xref:performance/caching/memory>
* <xref:security/authentication/identity>
