---
title: ASP.NET Core 中的缓存内存
author: rick-anderson
description: 了解如何在 ASP.NET Core 内存中缓存数据。
ms.author: riande
ms.custom: mvc
ms.date: 02/02/2020
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
uid: performance/caching/memory
ms.openlocfilehash: 4d5f459d54a3c74a2eb23a50db6537eeaf8596b3
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061439"
---
# <a name="cache-in-memory-in-aspnet-core"></a>ASP.NET Core 中的缓存内存

::: moniker range=">= aspnetcore-3.0"

作者： [Rick Anderson](https://twitter.com/RickAndMSFT)、 [John Luo](https://github.com/JunTaoLuo)和 [Steve Smith](https://ardalis.com/)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/memory/3.0sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="caching-basics"></a>缓存基础知识

缓存可以减少生成内容所需的工作，从而显著提高应用程序的性能和可伸缩性。 缓存最适用于不经常更改的 **数据，生成** 成本很高。 通过缓存，可以比从数据源返回的数据的副本速度快得多。 应该对应用进行编写和测试，使其 **永不** 依赖于缓存的数据。

ASP.NET Core 支持多个不同的缓存。 最简单的缓存基于 [IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache)。 `IMemoryCache` 表示存储在 web 服务器的内存中的缓存。 在服务器场上运行的应用 (多台服务器) 应确保会话在使用内存中缓存时处于粘滞状态。 粘滞会话确保来自客户端的后续请求都将发送到相同的服务器。 例如，Azure Web 应用使用 [应用程序请求路由](https://www.iis.net/learn/extensions/planning-for-arr) (ARR) 将所有后续请求路由到同一服务器。

Web 场中的非粘滞会话需要 [分布式缓存](distributed.md) ，以避免缓存一致性问题。 对于某些应用，分布式缓存可支持比内存中缓存更高的向外扩展。 使用分布式缓存会将缓存内存卸载到外部进程。

内存中缓存可以存储任何对象。 分布式缓存接口仅限 `byte[]` 。 内存中和分布式缓存将缓存项作为键值对。

## <a name="systemruntimecachingmemorycache"></a>MemoryCache/缓存

<xref:System.Runtime.Caching>/<xref:System.Runtime.Caching.MemoryCache> ([NuGet 包](https://www.nuget.org/packages/System.Runtime.Caching/)) 可用于：

* .NET Standard 2.0 或更高版本。
* 面向 .NET Standard 2.0 或更高版本的任何 [.net 实现](/dotnet/standard/net-standard#net-implementation-support) 。 例如，ASP.NET Core 2.0 或更高版本。
* .NET Framework 4.5 或更高版本。

[Microsoft.Extensions.Caching.Memory](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/) / `IMemoryCache` 建议使用 (本文中所述) ， `System.Runtime.Caching` / `MemoryCache` 因为它更好地集成到 ASP.NET Core 中。 例如， `IMemoryCache` 使用 ASP.NET Core [依赖关系注入](xref:fundamentals/dependency-injection)本身工作。

将 `System.Runtime.Caching` / `MemoryCache` ASP.NET 4.x 中的代码移植到 ASP.NET Core 时，请使用作为兼容性桥。

## <a name="cache-guidelines"></a>缓存指南

* 代码应始终具有回退选项，以获取数据，而 **不** 是依赖于可用的缓存值。
* 缓存使用稀有资源内存。 限制缓存增长：
  * 不要 **使用外部** 输入作为缓存键。
  * 使用过期限制缓存增长。
  * [使用 SetSize、Size 和 SizeLimit 限制缓存大小](#use-setsize-size-and-sizelimit-to-limit-cache-size)。 ASP.NET Core 运行时不会根据内存 **压力限制缓存** 大小。 开发人员需要限制缓存大小。

## <a name="use-imemorycache"></a>使用 IMemoryCache

> [!WARNING]
> 使用 [依赖关系注入](xref:fundamentals/dependency-injection)中的 *共享* 内存缓存，并调用 `SetSize` 、 `Size` 或 `SizeLimit` 以限制缓存大小可能会导致应用程序失败。 在缓存上设置大小限制时，在添加时，所有项都必须指定大小。 这可能会导致问题，因为开发人员可能无法完全控制使用共享缓存的内容。 例如，Entity Framework Core 使用共享缓存并且未指定大小。 如果应用设置了缓存大小限制并使用 EF Core，则应用将引发 `InvalidOperationException` 。
> 当使用 `SetSize` 、 `Size` 或 `SizeLimit` 来限制缓存时，为缓存创建一个缓存单独。 有关详细信息和示例，请参阅 [使用 SetSize、Size 和 SizeLimit 限制缓存大小](#use-setsize-size-and-sizelimit-to-limit-cache-size)。
> 共享缓存由其他框架或库共享。 例如，EF Core 使用共享缓存并且未指定大小。 

内存中缓存是从应用程序中使用 [依赖关系注入](xref:fundamentals/dependency-injection)引用的一种 *服务* 。 `IMemoryCache`在构造函数中请求实例：

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ctor)]

以下代码使用 [TryGetValue](/dotnet/api/microsoft.extensions.caching.memory.imemorycache.trygetvalue?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) 来检查缓存中是否有时间。 如果未缓存时间，则将创建一个新条目，并 [将其设置](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_)为已添加到缓存中。 `CacheKeys`类是下载示例的一部分。

[!code-csharp[](memory/3.0sample/WebCacheSample/CacheKeys.cs)]

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet1)]

显示当前时间和缓存时间：

[!code-cshtml[](memory/3.0sample/WebCacheSample/Views/Home/Cache.cshtml)]

如果在 `DateTime` 超时期限内存在请求，则缓存值将保留在缓存中。

以下代码使用 [GetOrCreate](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreate#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) 和 [GetOrCreateAsync](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreateasync#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) 来缓存数据。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

以下代码调用 [Get 获取](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) 缓存时间：

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_gct)]

下面的代码获取或创建具有绝对过期的缓存项：

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet99)]

只有具有可调过期的缓存项集存在过时的风险。 如果访问的时间比滑动过期时间间隔更频繁，则该项将永不过期。 将弹性过期与绝对过期组合在一起，以保证项目在其绝对过期时间通过后过期。 绝对过期会将项的上限设置为可缓存项的时间，同时仍允许项在可调整过期时间间隔内未请求时提前过期。 如果同时指定了绝对过期和可调过期时间，则过期时间以逻辑方式运算。 如果滑动过期时间间隔 *或* 绝对过期时间通过，则从缓存中逐出该项。

下面的代码获取或创建具有可调 *和* 绝对过期的缓存项：

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet9)]

前面的代码保证数据的缓存时间不超过绝对时间。

<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreate*>、 <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreateAsync*> 和 <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.Get*> 是类中的扩展方法 <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions> 。 这些方法扩展了的功能 <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> 。

## <a name="memorycacheentryoptions"></a>MemoryCacheEntryOptions

下面的示例：

* 设置可调过期时间。 访问此缓存项的请求将重置可调过期时间。
* 将缓存优先级设置为 [CacheItemPriority. NeverRemove](xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove)。
* 设置在从缓存中逐出项后将调用的 [PostEvictionDelegate](/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate) 。 回调在从缓存中删除项的代码的不同线程上运行。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_et&highlight=14-21)]

## <a name="use-setsize-size-and-sizelimit-to-limit-cache-size"></a>使用 SetSize、Size 和 SizeLimit 限制缓存大小

`MemoryCache`实例可以选择指定并强制实施大小限制。 缓存大小限制没有定义的度量单位，因为缓存没有度量条目大小的机制。 如果设置了缓存大小限制，则所有条目都必须指定 size。 ASP.NET Core 运行时不会根据内存压力限制缓存大小。 开发人员需要限制缓存大小。 指定的大小以开发人员选择的单位为单位。

例如： 。

* 如果 web 应用主要是缓存字符串，则每个缓存条目大小都可以是字符串长度。
* 应用可以将所有条目的大小指定为1，而大小限制则为条目的计数。

如果 <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheOptions.SizeLimit> 未设置，则缓存的大小不受限制。 当系统内存不足时，ASP.NET Core 运行时不会剪裁缓存。 应用必须构建为：

* 限制缓存增长。
* <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Compact*> <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Remove*> 当可用内存有限时调用或：

以下代码创建了 <xref:Microsoft.Extensions.Caching.Memory.MemoryCache> [依赖关系注入](xref:fundamentals/dependency-injection)可访问的无大小固定大小：

[!code-csharp[](memory/sample/RPcache/Services/MyMemoryCache.cs?name=snippet)]

`SizeLimit` 没有单位。 如果已设置缓存大小限制，则缓存条目必须以其认为最适合的任何单位指定大小。 缓存实例的所有用户都应使用同一单元系统。 如果缓存条目大小之和超过指定的值，则不会缓存条目 `SizeLimit` 。 如果未设置任何缓存大小限制，则将忽略在该项上设置的缓存大小。

以下代码将注册 `MyMemoryCache` 到 [依赖关系注入](xref:fundamentals/dependency-injection) 容器。

[!code-csharp[](memory/3.0sample/RPcache/Startup.cs?name=snippet)]

`MyMemoryCache` 对于识别此大小限制缓存并知道如何适当地设置缓存条目大小的组件，将创建为独立的内存缓存。

以下代码使用 `MyMemoryCache` ：

[!code-csharp[](memory/3.0sample/RPcache/Pages/SetSize.cshtml.cs?name=snippet)]

缓存项的大小可以通过 <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryOptions.Size> 或 <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheEntryExtensions.SetSize*> 扩展方法来设置：

[!code-csharp[](memory/3.0sample/RPcache/Pages/SetSize.cshtml.cs?name=snippet2&highlight=9,10,14,15)]

### <a name="memorycachecompact"></a>MemoryCache

`MemoryCache.Compact` 尝试按以下顺序删除缓存的指定百分比：

* 所有过期项。
* 按优先级排序。 首先删除最低优先级项。
* 最近最少使用的对象。
* 绝对过期的项。
* 具有最早的可调过期项的项。

永远不会删除具有优先级的固定项 <xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove> 。 以下代码将删除缓存项并调用 `Compact` ：

[!code-csharp[](memory/3.0sample/RPcache/Pages/TestCache.cshtml.cs?name=snippet3)]

有关详细信息，请参阅 [GitHub 上的 Compact 源](https://github.com/dotnet/extensions/blob/v3.0.0-preview8.19405.4/src/Caching/Memory/src/MemoryCache.cs#L382-L393) 。

## <a name="cache-dependencies"></a>缓存依赖关系

下面的示例演示如何在依赖条目过期时使缓存条目过期。 <xref:Microsoft.Extensions.Primitives.CancellationChangeToken>添加到缓存的项。 当 `Cancel` 在上调用时 `CancellationTokenSource` ，将逐出两个缓存项。

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ed)]

使用 <xref:System.Threading.CancellationTokenSource> 允许将多个缓存条目作为一个组逐出。 `using`在上面的代码中，在块中创建的缓存条目 `using` 将继承触发器和过期设置。

## <a name="additional-notes"></a>附加说明

* 不会在后台进行过期。 没有计时器可主动扫描过期项目的缓存。 缓存上的任何活动 (`Get` ， `Set` `Remove`) 可以触发过期项的后台扫描。  (上的计时器 `CancellationTokenSource` <xref:System.Threading.CancellationTokenSource.CancelAfter*>) 还会删除项并触发扫描过期项。 下面的示例将 [CancellationTokenSource (TimeSpan) ](/dotnet/api/system.threading.cancellationtokensource.-ctor) 用于注册的令牌。 此令牌激发后，会立即删除该条目，并激发逐出回调：

[!code-csharp[](memory/3.0sample/WebCacheSample/Controllers/HomeController.cs?name=snippet_ae)]

* 使用回调来重新填充缓存项时：

  * 多个请求可以查找缓存的密钥值为空，因为回调未完成。
  * 这可能会导致多个线程重新填充缓存的项。

* 当使用一个缓存条目创建另一个缓存条目时，子对象会复制父条目的过期令牌和基于时间的过期设置。 手动删除或更新父项时，子级不会过期。

* 用于 <xref:Microsoft.Extensions.Caching.Memory.ICacheEntry.PostEvictionCallbacks> 设置在从缓存中逐出缓存项后将触发的回调。
* 对于大多数应用， `IMemoryCache` 启用。 例如， `AddMvc` 在中调用、、、 `AddControllersWithViews` `AddRazorPages` `AddMvcCore().AddRazorViewEngine` 和许多其他 `Add{Service}` 方法将 `ConfigureServices` 启用 `IMemoryCache` 。 对于未调用上述方法之一的应用程序 `Add{Service}` ，可能需要 <xref:Microsoft.Extensions.DependencyInjection.MemoryCacheServiceCollectionExtensions.AddMemoryCache*> 在中调用 `ConfigureServices` 。

## <a name="background-cache-update"></a>后台缓存更新

使用 [后台服务](xref:fundamentals/host/hosted-services) （如） <xref:Microsoft.Extensions.Hosting.IHostedService> 来更新缓存。 后台服务可重新计算条目，然后仅在准备就绪时将其分配给缓存。

## <a name="additional-resources"></a>其他资源

* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

<!-- This is the 2.1 version -->
作者： [Rick Anderson](https://twitter.com/RickAndMSFT)、 [John Luo](https://github.com/JunTaoLuo)和 [Steve Smith](https://ardalis.com/)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/performance/caching/memory/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="caching-basics"></a>缓存基础知识

缓存可以减少生成内容所需的工作，从而显著提高应用程序的性能和可伸缩性。 缓存最适用于不经常更改的数据。 通过缓存，可以比从原始数据源返回的数据的副本速度快得多。 应编写并测试代码，使其 **绝不会** 依赖于缓存的数据。

ASP.NET Core 支持多个不同的缓存。 最简单的缓存基于 [IMemoryCache](/dotnet/api/microsoft.extensions.caching.memory.imemorycache)，这表示存储在 web 服务器内存中的缓存。 在服务器场上运行的应用 (多台服务器) 应确保会话在使用内存中缓存时处于粘滞。 粘滞会话确保以后来自客户端的请求都发送到相同的服务器。 例如，Azure Web 应用使用 [应用程序请求路由](https://www.iis.net/learn/extensions/planning-for-arr) (ARR) 将来自用户代理的所有请求路由到同一服务器。

Web 场中的非粘滞会话需要 [分布式缓存](distributed.md) ，以避免缓存一致性问题。 对于某些应用，分布式缓存可支持比内存中缓存更高的向外扩展。 使用分布式缓存会将缓存内存卸载到外部进程。

内存中缓存可以存储任何对象。 分布式缓存接口仅限 `byte[]` 。 内存中和分布式缓存将缓存项作为键值对。

## <a name="systemruntimecachingmemorycache"></a>MemoryCache/缓存

<xref:System.Runtime.Caching>/<xref:System.Runtime.Caching.MemoryCache> ([NuGet 包](https://www.nuget.org/packages/System.Runtime.Caching/)) 可用于：

* .NET Standard 2.0 或更高版本。
* 面向 .NET Standard 2.0 或更高版本的任何 [.net 实现](/dotnet/standard/net-standard#net-implementation-support) 。 例如，ASP.NET Core 2.0 或更高版本。
* .NET Framework 4.5 或更高版本。

[Microsoft.Extensions.Caching.Memory](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/) / `IMemoryCache` 建议使用 (本文中所述) ， `System.Runtime.Caching` / `MemoryCache` 因为它更好地集成到 ASP.NET Core 中。 例如， `IMemoryCache` 使用 ASP.NET Core [依赖关系注入](xref:fundamentals/dependency-injection)本身工作。

将 `System.Runtime.Caching` / `MemoryCache` ASP.NET 4.x 中的代码移植到 ASP.NET Core 时，请使用作为兼容性桥。

## <a name="cache-guidelines"></a>缓存指南

* 代码应始终具有回退选项，以获取数据，而 **不** 是依赖于可用的缓存值。
* 缓存使用稀有资源内存。 限制缓存增长：
  * 不要 **使用外部** 输入作为缓存键。
  * 使用过期限制缓存增长。
  * [使用 SetSize、Size 和 SizeLimit 限制缓存大小](#use-setsize-size-and-sizelimit-to-limit-cache-size)。 ASP.NET Core 运行时不会根据内存压力限制缓存大小。 开发人员需要限制缓存大小。

## <a name="using-imemorycache"></a>使用 IMemoryCache

> [!WARNING]
> 使用 [依赖关系注入](xref:fundamentals/dependency-injection)中的 *共享* 内存缓存，并调用 `SetSize` 、 `Size` 或 `SizeLimit` 以限制缓存大小可能会导致应用程序失败。 在缓存上设置大小限制时，在添加时，所有项都必须指定大小。 这可能会导致问题，因为开发人员可能无法完全控制使用共享缓存的内容。 例如，Entity Framework Core 使用共享缓存并且未指定大小。 如果应用设置了缓存大小限制并使用 EF Core，则应用将引发 `InvalidOperationException` 。
> 当使用 `SetSize` 、 `Size` 或 `SizeLimit` 来限制缓存时，为缓存创建一个缓存单独。 有关详细信息和示例，请参阅 [使用 SetSize、Size 和 SizeLimit 限制缓存大小](#use-setsize-size-and-sizelimit-to-limit-cache-size)。

内存中缓存是从应用程序中使用 [依赖关系注入](../../fundamentals/dependency-injection.md)引用的一种 *服务* 。 调用 `AddMemoryCache` 于 `ConfigureServices` ：

[!code-csharp[](memory/sample/WebCache/Startup.cs?highlight=9)]

`IMemoryCache`在构造函数中请求实例：

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ctor)]

`IMemoryCache`需要 NuGet 包 AspNetCore，此包可在[元包](xref:fundamentals/metapackage-app)中找到[的。](https://www.nuget.org/packages/Microsoft.Extensions.Caching.Memory/)

以下代码使用 [TryGetValue](/dotnet/api/microsoft.extensions.caching.memory.imemorycache.trygetvalue?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_IMemoryCache_TryGetValue_System_Object_System_Object__) 来检查缓存中是否有时间。 如果未缓存时间，则将创建一个新条目，并 [将其设置](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.set?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_CacheExtensions_Set__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object___0_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_)为已添加到缓存中。

[!code-csharp[](memory/sample/WebCache/CacheKeys.cs)]

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet1)]

显示当前时间和缓存时间：

[!code-cshtml[](memory/sample/WebCache/Views/Home/Cache.cshtml)]

如果在 `DateTime` 超时期限内存在请求，则缓存值将保留在缓存中。 下图显示了从缓存中检索到的当前时间和旧时间：

![显示两个不同时间的索引视图](memory/_static/time.png)

以下代码使用 [GetOrCreate](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreate#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreate__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry___0__) 和 [GetOrCreateAsync](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.getorcreateasync#Microsoft_Extensions_Caching_Memory_CacheExtensions_GetOrCreateAsync__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_System_Func_Microsoft_Extensions_Caching_Memory_ICacheEntry_System_Threading_Tasks_Task___0___) 来缓存数据。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet2&highlight=3-7,14-19)]

以下代码调用 [Get 获取](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) 缓存时间：

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_gct)]

<xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreate*> 、 <xref:Microsoft.Extensions.Caching.Memory.CacheExtensions.GetOrCreateAsync*> 和 [Get](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions.get#Microsoft_Extensions_Caching_Memory_CacheExtensions_Get__1_Microsoft_Extensions_Caching_Memory_IMemoryCache_System_Object_) 是 [CacheExtensions](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions) 类的扩展方法的一部分，它扩展了的功能 <xref:Microsoft.Extensions.Caching.Memory.IMemoryCache> 。 有关其他缓存方法的说明，请参阅 [IMemoryCache 方法](/dotnet/api/microsoft.extensions.caching.memory.imemorycache) 和 [CacheExtensions 方法](/dotnet/api/microsoft.extensions.caching.memory.cacheextensions) 。

## <a name="memorycacheentryoptions"></a>MemoryCacheEntryOptions

下面的示例：

* 设置可调过期时间。 访问此缓存项的请求将重置可调过期时间。
* 将缓存优先级设置为 `CacheItemPriority.NeverRemove` 。
* 设置在从缓存中逐出项后将调用的 [PostEvictionDelegate](/dotnet/api/microsoft.extensions.caching.memory.postevictiondelegate) 。 回调在从缓存中删除项的代码的不同线程上运行。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_et&highlight=14-21)]

## <a name="use-setsize-size-and-sizelimit-to-limit-cache-size"></a>使用 SetSize、Size 和 SizeLimit 限制缓存大小

`MemoryCache`实例可以选择指定并强制实施大小限制。 缓存大小限制没有定义的度量单位，因为缓存没有度量条目大小的机制。 如果设置了缓存大小限制，则所有条目都必须指定 size。 ASP.NET Core 运行时不会根据内存压力限制缓存大小。 开发人员需要限制缓存大小。 指定的大小以开发人员选择的单位为单位。

例如： 。

* 如果 web 应用主要是缓存字符串，则每个缓存条目大小都可以是字符串长度。
* 应用可以将所有条目的大小指定为1，而大小限制则为条目的计数。

如果 <xref:Microsoft.Extensions.Caching.Memory.MemoryCacheOptions.SizeLimit> 未设置，则缓存的大小不受限制。 当系统内存不足时，ASP.NET Core 运行时不会剪裁缓存。 应用程序的体系结构非常多：

* 限制缓存增长。
* <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Compact*> <xref:Microsoft.Extensions.Caching.Memory.MemoryCache.Remove*> 当可用内存有限时调用或：

以下代码创建了 <xref:Microsoft.Extensions.Caching.Memory.MemoryCache> [依赖关系注入](xref:fundamentals/dependency-injection)可访问的无大小固定大小：

[!code-csharp[](memory/sample/RPcache/Services/MyMemoryCache.cs?name=snippet)]

`SizeLimit` 没有单位。 如果已设置缓存大小限制，则缓存条目必须以其认为最适合的任何单位指定大小。 缓存实例的所有用户都应使用同一单元系统。 如果缓存条目大小之和超过指定的值，则不会缓存条目 `SizeLimit` 。 如果未设置任何缓存大小限制，则将忽略在该项上设置的缓存大小。

以下代码将注册 `MyMemoryCache` 到 [依赖关系注入](xref:fundamentals/dependency-injection) 容器。

[!code-csharp[](memory/sample/RPcache/Startup.cs?name=snippet&highlight=5)]

`MyMemoryCache` 对于识别此大小限制缓存并知道如何适当地设置缓存条目大小的组件，将创建为独立的内存缓存。

以下代码使用 `MyMemoryCache` ：

[!code-csharp[](memory/sample/RPcache/Pages/About.cshtml.cs?name=snippet)]

缓存项的大小可以通过 [大小](/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryoptions.size?view=aspnetcore-2.1#Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_Size) 或 [SetSize](/dotnet/api/microsoft.extensions.caching.memory.memorycacheentryextensions.setsize?view=aspnetcore-2.0#Microsoft_Extensions_Caching_Memory_MemoryCacheEntryExtensions_SetSize_Microsoft_Extensions_Caching_Memory_MemoryCacheEntryOptions_System_Int64_) 扩展方法来设置：

[!code-csharp[](memory/sample/RPcache/Pages/About.cshtml.cs?name=snippet2&highlight=9,10,14,15)]

### <a name="memorycachecompact"></a>MemoryCache

`MemoryCache.Compact` 尝试按以下顺序删除缓存的指定百分比：

* 所有过期项。
* 按优先级排序。 首先删除最低优先级项。
* 最近最少使用的对象。
* 绝对过期的项。
* 具有最早的可调过期项的项。

永远不会删除具有优先级的固定项 <xref:Microsoft.Extensions.Caching.Memory.CacheItemPriority.NeverRemove> 。

[!code-csharp[](memory/3.0sample/RPcache/Pages/TestCache.cshtml.cs?name=snippet3)]

有关详细信息，请参阅 [GitHub 上的 Compact 源](https://github.com/dotnet/extensions/blob/v3.0.0-preview8.19405.4/src/Caching/Memory/src/MemoryCache.cs#L382-L393) 。

## <a name="cache-dependencies"></a>缓存依赖关系

下面的示例演示如何在依赖条目过期时使缓存条目过期。 <xref:Microsoft.Extensions.Primitives.CancellationChangeToken>添加到缓存的项。 当 `Cancel` 在上调用时 `CancellationTokenSource` ，将逐出两个缓存项。

[!code-csharp[](memory/sample/WebCache/Controllers/HomeController.cs?name=snippet_ed)]

使用 `CancellationTokenSource` 允许将多个缓存条目作为一个组逐出。 `using`在上面的代码中，在块中创建的缓存条目 `using` 将继承触发器和过期设置。

## <a name="additional-notes"></a>附加说明

* 使用回调来重新填充缓存项时：

  * 多个请求可以查找缓存的密钥值为空，因为回调未完成。
  * 这可能会导致多个线程重新填充缓存的项。

* 当使用一个缓存条目创建另一个缓存条目时，子对象会复制父条目的过期令牌和基于时间的过期设置。 手动删除或更新父项时，子级不会过期。

* 使用 [PostEvictionCallbacks](/dotnet/api/microsoft.extensions.caching.memory.icacheentry.postevictioncallbacks#Microsoft_Extensions_Caching_Memory_ICacheEntry_PostEvictionCallbacks) 设置从缓存中逐出缓存项后将触发的回调。

## <a name="background-cache-update"></a>后台缓存更新

使用 [后台服务](xref:fundamentals/host/hosted-services) （如） <xref:Microsoft.Extensions.Hosting.IHostedService> 来更新缓存。 后台服务可重新计算条目，然后仅在准备就绪时将其分配给缓存。

## <a name="additional-resources"></a>其他资源

* <xref:performance/caching/distributed>
* <xref:fundamentals/change-tokens>
* <xref:performance/caching/response>
* <xref:performance/caching/middleware>
* <xref:mvc/views/tag-helpers/builtin-th/cache-tag-helper>
* <xref:mvc/views/tag-helpers/builtin-th/distributed-cache-tag-helper>

::: moniker-end
