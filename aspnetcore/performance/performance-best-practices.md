---
title: ASP.NET Core 性能优化最佳实践
author: mjrousos
description: 在 ASP.NET Core 应用程序中提高性能和避免常见性能问题的小知识。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.date: 04/06/2020
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
uid: performance/performance-best-practices
ms.openlocfilehash: a3fc398569fafefc0b4634e80433a5d4e0e1b4ff
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060997"
---

# <a name="aspnet-core-performance-best-practices"></a>ASP.NET Core 性能优化最佳实践

由 [Mike Rousos](https://github.com/mjrousos)

本文提供了 ASP.NET Core 的性能最佳实践指南。

## <a name="cache-aggressively"></a>积极利用缓存

这里有一篇文档在多个部分中讨论了如何积极利用缓存。 有关详细信息，请参阅︰ <xref:performance/caching/response>.

## <a name="understand-hot-code-paths"></a>了解代码中的热点路径

在本文档中， *代码热点路径* 定义为频繁调用的代码路径以及执行时间的大部分时间。 代码热点路径通常限制应用程序的扩展和性能，并在本文档的多个部分中进行讨论。

## <a name="avoid-blocking-calls"></a>避免阻塞式调用

ASP.NET Core 应用程序应设计为同时处理许多请求。 异步 API 可以使用一个小池线程通过非阻塞式调用来处理数以千计的并发请求。 线程可以处理另一个请求，而不是等待长时间运行的同步任务完成。

ASP.NET Core 应用程序中的常见性能问题通常是由于那些本可以异步调用但却采用阻塞时调用而导致的。 同步阻塞会调用导致 [线程池饥饿](https://blogs.msdn.microsoft.com/vancem/2018/10/16/diagnosing-net-core-threadpool-starvation-with-perfview-why-my-service-is-not-saturating-all-cores-or-seems-to-stall/) 和响应时间降级。

**请勿** ：

* 通过调用 [Task.Wait](/dotnet/api/system.threading.tasks.task.wait) 或 [Task.Result](/dotnet/api/system.threading.tasks.task-1.result) 来阻止异步执行。
* 在公共代码路径中加锁。 ASP.NET Core 应用程序应设计为并行运行代码，如此才能使得性能最佳。
* 调用 [Task.Run](/dotnet/api/system.threading.tasks.task.run) 并立即 await 。 ASP.NET Core 本身已经是在线程池线程上运行应用程序代码了，因此这样调用 Task.Run 只会导致额外的不必要的线程池调度。 而且即使被调度的代码会阻止线程， Task.Run 也并不能避免这种情况，这样做没有意义。

**要**:

* 使 [热代码路径](#understand-hot-code-paths) 处于异步状态。
* 如果异步 API 可用，则异步调用数据访问、i/o 和长时间运行的操作 Api。 不要 **使用**[任务。运行](/dotnet/api/system.threading.tasks.task.run)以使同步 API 成为异步同步。
* 使控制器/ Razor 页面操作异步。 为了受益于 [async/await](/dotnet/csharp/programming-guide/concepts/async/) 模式，整个调用堆栈是异步的。

## <a name="return-ienumerablet-or-iasyncenumerablet"></a>使用 IEumerable&lt;T&gt;<T> 或 IAsyncEnumerable&lt;T&gt; 作为返回值<T>

在 Action 中返回`IEumerable<T>`将会被序列化器中进行同步迭代 。 结果是可能导致阻塞或者线程池饥饿。 想要要避免同步迭代集合，可以在返回迭代集合之前使用 `ToListAsync`使其异步化。

从 ASP.NET Core 3.0 开始， `IAsyncEnumerable<T>` 可以用作为 ` IEumerable<T>` 的替代方法，以异步方式进行迭代。 有关更多信息，请参阅 [Controller Action 的返回值类型](xref:web-api/action-return-types#return-ienumerablet-or-iasyncenumerablet)。

## <a name="minimize-large-object-allocations"></a>尽可能少的使用大对象

[.NET Core 垃圾收集器](/dotnet/standard/garbage-collection/) 在 ASP.NET Core 应用程序中起到自动管理内存的分配和释放的作用。 自动垃圾回收通常意味着开发者不需要担心如何或何时释放内存。 但是，清除未引用的对象将会占用 CPU 时间，因此开发者应最小化 [代码热点路径](#understand-hot-code-paths) 中的分配的对象。 垃圾回收在大对象上代价特大 (> 85 K 字节 ) 。 大对象存储在 [large object heap](/dotnet/standard/garbage-collection/large-object-heap) 上，需要 full (generation 2) garbage collection 来清理。 与 generation 0 和 generation 1 不同，generation 2 需要临时暂挂应用程序。 故而频繁分配和取消分配大型对象会导致性能耗损。

建议 :

* **请考虑缓存** 经常使用的大型对象。 缓存大型对象会阻止开销较高的分配。
* 使用 [ArrayPool \<T>](/dotnet/api/system.buffers.arraypool-1)存储大型数组 **来池缓冲区** 。
* **不要** 在 [热代码路径](#understand-hot-code-paths)上分配很多生存期较短的大型对象。

可以通过查看 [PerfView](https://github.com/Microsoft/perfview) 中的垃圾回收 (GC) 统计信息来诊断并检查内存问题，其中包括:

* 垃圾回收挂起时间。
* 垃圾回收中耗用的处理器时间百分比。
* 有多少垃圾回收发生在 generation 0, 1, 和 2.

有关更多信息，请参阅 [垃圾回收和性能](/dotnet/standard/garbage-collection/performance)。

## <a name="optimize-data-access-and-io"></a>优化数据操作和 I/O

与数据存储器和其他远程服务的交互通常是 ASP.NET Core 应用程序最慢的部分。 高效读取和写入数据对于良好的性能至关重要。

建议 :


* **请** 以异步方式调用所有数据访问 api。
* 检索的数据 **不** 是必需的。 编写查询以仅返回当前 HTTP 请求所必需的数据。
* 如果数据可以接受， **请考虑缓存** 经常访问的从数据库或远程服务检索的数据。 使用 [MemoryCache](xref:performance/caching/memory) 或 [microsoft.web.distributedcache](xref:performance/caching/distributed)，具体取决于方案。 有关详细信息，请参阅 <xref:performance/caching/response>。
* **尽量减少** 网络往返次数。 目标是使用单个调用而不是多个调用来检索所需数据。
* 在访问数据时， **请不要** 在 Entity Framework Core 中使用 [无跟踪查询](/ef/core/querying/tracking#no-tracking-queries)。 EF Core 可以更有效地返回无跟踪查询的结果。
* 使用、或语句 **(筛选和** 聚合 LINQ 查询 `.Where` `.Select` `.Sum` ，例如) ，以便数据库执行筛选。
* **请考虑 EF Core** 在客户端上解析一些查询运算符，这可能导致查询执行效率低下。 有关详细信息，请参阅 [客户端评估性能问题](/ef/core/querying/client-eval#client-evaluation-performance-issues)。
* **不要** 对集合使用投影查询，这可能会导致执行 "N + 1" 个 SQL 查询。 有关详细信息，请参阅 [相关子查询的优化](/ef/core/what-is-new/ef-core-2.1#optimization-of-correlated-subqueries)。

请参阅 [EF 高性能专题](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries) 以了解可能提高应用性能的方法:

* [DbContext 池](/ef/core/what-is-new/ef-core-2.0#dbcontext-pooling)
* [显式编译的查询](/ef/core/what-is-new/ef-core-2.0#explicitly-compiled-queries)

在代码提交之前，我们建议评估上述高性能方法的影响。 编译查询的额外复杂性可能无法一定确保性能提高。

可以通过使用 [Application Insights](/azure/application-insights/app-insights-overview) 或使用分析工具查看访问数据所花费的时间来检测查询问题。 大多数数据库还提供有关频繁执行的查询的统计信息，这也可以作为重要参考。

## <a name="pool-http-connections-with-httpclientfactory"></a>通过 HttpClientFactory 建立 HTTP 连接池

虽然 [HttpClient](/dotnet/api/system.net.http.httpclient) 实现了 `IDisposable` 接口，但它其实被设计为可以重复使用单个实例。 关闭 `HttpClient` 实例会使套接字在短时间内以 `TIME_WAIT` 状态打开。 如果经常创建和释放 `HttpClient` 对象，那么应用程序可能会耗尽可用套接字。 在 ASP.NET Core 2.1中，引入了[HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) 作为解决这个问题的办法。 它以池化 HTTP 连接的方式从而优化性能和可靠性。

建议 :

* **不要** 直接创建和释放 `HttpClient` 实例。
* **要** 使用 [HttpClientFactory](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests) 来获取 `HttpClient` 实例。 有关更多信息，请参阅 [使用 HttpClientFactory 以实现弹性 HTTP 请求](/dotnet/standard/microservices-architecture/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)。

## <a name="keep-common-code-paths-fast"></a>确保公共代码路径快若鹰隼

如果你想要所有的代码都保持高速， 高频调用的代码路径就是优化的最关键路径。 优化措施包括:

* 考虑优化应用程序请求处理管道中的 Middleware ，尤其是在管道中排在更前面运行的 Middleware 。 这些组件对性能有很大影响。
* 考虑优化那些每个请求都要执行或每个请求多次执行的代码。 例如，自定义日志，身份认证与授权或 transient 服务的创建等等。

建议 :

* **不要** 使用自定义 middleware 运行长时任务 。
* **要** 使用性能分析工具( 如 [Visual Studio Diagnostic Tools](/visualstudio/profiling/profiling-feature-tour) 或 [PerfView](https://github.com/Microsoft/perfview)) 来定位 [代码热点路径](#understand-hot-code-paths)。

## <a name="complete-long-running-tasks-outside-of-http-requests"></a>在 HTTP 请求之外运行长时任务

对 ASP.NET Core 应用程序的大多数请求可以由调用服务的 controller 或页面模型处理，并返回 HTTP 响应。 对于涉及长时间运行的任务的某些请求，最好使整个请求-响应进程异步。

建议 :

* **请** 不要等待长时间运行的任务在普通的 HTTP 请求处理过程中完成。
* **请考虑使用**[后台服务](xref:fundamentals/host/hosted-services)处理长时间运行的请求，或使用 [Azure 函数](/azure/azure-functions/)处理进程外的请求。 在进程外完成工作对于 CPU 密集型任务特别有用。
* **请使用实时** 通信选项（如 [SignalR](xref:signalr/introduction) ）以异步方式与客户端进行通信。

## <a name="minify-client-assets"></a>缩小客户端资源

复杂的 ASP.NET Core 应用程序经常包含很有前端文件例如 JavaScript， CSS 或图片文件。 可以通过以下方法优化初始请求的性能:

* 打包，将多个文件合并为一个文件。
* 压缩，通过除去空格和注释来缩小文件大小。

建议 :

* **要** 使用 ASP.NET Core 的 [内置支持](xref:client-side/bundling-and-minification) 用于打包和压缩客户端资源文件的组件。
* **要** 考虑其他第三方工具，如 [Webpack](https://webpack.js.org/)，用于复杂客户资产管理。

## <a name="compress-responses"></a>压缩 Http 响应

 减少响应的大小通常会显着提高应用程序的响应性。 而减小内容大小的一种方法是压缩应用程序的响应。 有关更多信息，请参阅 [响应压缩](xref:performance/response-compression)。

## <a name="use-the-latest-aspnet-core-release"></a>使用最新的 ASP.NET Core 发行版

ASP.NET Core 的每个新发行版都包含性能改进。 .NET Core 和 ASP.NET Core 中的优化意味着较新的版本通常优于较旧版本。 例如， .NET Core 2.1 添加了对预编译的正则表达式的支持，并从使用 [Span&lt;T&gt;](/archive/msdn-magazine/2018/january/csharp-all-about-span-exploring-a-new-net-mainstay) 改进性能。 ASP.NET Core 2.2 添加了对 HTTP/2的支持。 [ASP.NET Core 3.0 增加了许多改进](xref:aspnetcore-3.0) ，以减少内存使用量并提高吞吐量。 如果性能是优先考虑的事情，那么请升级到 ASP.NET Core 的当前版本。

## <a name="minimize-exceptions"></a>最小化异常

异常应该竟可能少。 相对于正常代码流程来说，抛出和捕获异常是缓慢的。 因此，不应使用异常来控制正常程序流。

建议 :

* **不要** 使用引发或捕获异常作为正常程序流的方法，尤其是在 [热代码路径](#understand-hot-code-paths)中。
* 在应用程序 **中包括逻辑** ，以检测和处理会导致异常的情况。
* **引发或** 捕获异常或意外情况的异常。

应用程序诊断工具( 如 Application Insights ) 可以帮助识别应用程序中可能影响性能的常见异常。

## <a name="performance-and-reliability"></a>性能和可靠性

下文将提供常见性能提示和已知可靠性问题的解决方案。

## <a name="avoid-synchronous-read-or-write-on-httprequesthttpresponse-body"></a>避免在 HttpRequest/HttpResponse body 上同步读取或写入

ASP.NET Core 中的所有 I/O 都是异步的。 服务器实现了 `Stream` 接口，它同时具有同步和异步的方法重载。 应该首选异步方式以避免阻塞线程池线程。 阻塞线程会导致线程池饥饿。

请勿 **执行此操作：** 下面的示例使用 <xref:System.IO.StreamReader.ReadToEnd*> 。 此方法阻止当前线程等待结果。 这是一个 [通过异步同步](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md#warning-sync-over-async
)的示例。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/MyFirstController.cs?name=snippet1)]

在上述代码中， `Get` 采用同步的方式将整个 HTTP 请求主体读取到内存中。 如果客户端上载数据很慢，那么应用程序就会出现看似异步实际同步的操作。 应用程序看似异步实际同步，因为 Kestrel **不** 支持同步读取。

**应该采用如下操作:** <xref:System.IO.StreamReader.ReadToEndAsync*> ，在读取时不阻塞线程。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/MyFirstController.cs?name=snippet2)]

上述代码异步将整个 HTTP request body 读取到内存中。

> [!WARNING] 如果请求很大，那么将整个 HTTP request body 读取到内存中可能会导致内存不足 (OOM) 。 OOM 可导致应用奔溃。  有关更多信息，请参阅 [避免将大型请求主体或响应主体读取到内存中](#arlb)。

**应该采用如下操作:** 使用不缓冲的方式完成 request body 操作:

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/MyFirstController.cs?name=snippet3)]

上述代码采用异步方式将 request body 序列化为 C# 对象。

## <a name="prefer-readformasync-over-requestform"></a>优先选用 Request.Form 的 ReadFormAsync

应该使用 `HttpContext.Request.ReadFormAsync` 而不是 `HttpContext.Request.Form`。 `HttpContext.Request.Form` 只能在以下场景用安全使用。

* 该表单已被 `ReadFormAsync`调用，并且
* 数据已经被从 `HttpContext.Request.Form` 读取并缓存

请勿 **执行此操作：** 下面的示例使用 `HttpContext.Request.Form` 。  `HttpContext.Request.Form`[通过异步使用同步](https://github.com/davidfowl/AspNetCoreDiagnosticScenarios/blob/master/AsyncGuidance.md#warning-sync-over-async
)，并可能导致线程池不足。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/MySecondController.cs?name=snippet1)]

**应该使用如下操作:** 使用`HttpContext.Request.ReadFormAsync` 异步读取表单正文。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/MySecondController.cs?name=snippet2)]

<a name="arlb"></a>

## <a name="avoid-reading-large-request-bodies-or-response-bodies-into-memory"></a>避免将大型 request body 或 response body 读取到内存中

在 .NET 中，大于 85 KB 的对象会被分配在大对象堆 ([LOH](https://blogs.msdn.microsoft.com/maoni/2006/04/19/large-object-heap/) )。 大型对象的开销较大，包含两方面:

* 分配大对象内存时需要对被分配的内存进行清空，这个操作成本较高。 CLR 会保证清空所有新分配的对象的内存。（将内存全部设置为0）
* LOH 只会在内存剩余不足时回收。 LOH 需要在 [full garbage collection](/dotnet/standard/garbage-collection/fundamentals) 或者 [Gen2 collection](/dotnet/standard/garbage-collection/fundamentals#generations)进行回收。

此 [博文](https://adamsitnik.com/Array-Pool/#the-problem) 很好描述了该问题:

> 当分配大对象时，它会被标记为 Gen 2 对象。 而不像是 Gen 0 那样的小对象。 这样的后果是，如果你在使用 LOH 时耗尽内存， GC 会清除整个托管堆，而不仅仅是 LOH 部分。 因此，它将清理 Gen 0, Gen 1 and Gen 2 ( 包括 LOH ) 。 这称为 full garbage collection，是最耗时的垃圾回收。 对于很多应用，这是可以接受的。 但绝对不适用于高性能 Web 服务器，因为高性能 Web 服务器需要更多的内存用于处理常规 Web 请求 ( 从套接字读取，解压缩，解码 JSON 等等 )。

天真地将一个大型 request 或者 response body 存储到单个 `byte[]` 或 `string`中:

* 这可能导致 LOH 的剩余空间快速耗尽。
* 因此产生的 full GC 可能会导致应用程序的性能问题。

## <a name="working-with-a-synchronous-data-processing-api"></a>使用同步 API 处理数据

例如使用仅支持同步读取和写入的序列化器/反序列化器时 ( 例如，  [JSON.NET](https://www.newtonsoft.com/json/help/html/Introduction.htm)):

* 将数据异步缓冲到内存中，然后将其传递到序列化器/反序列化器。

> [!WARNING] 如果请求较大，那么可能导致内存不足 (OOM) 。 OOM 可导致应用奔溃。  有关更多信息，请参阅 [避免将大型请求主体或响应主体读取到内存](#arlb)。

ASP.NET Core 3.0 默认情况下使用 <xref:System.Text.Json>进行 JSON 序列化，这将带来如下好处。 <xref:System.Text.Json>:

* 异步读取和写入 JSON 。
* 针对 UTF-8 文本进行了优化。
* 通常比 `Newtonsoft.Json` 更高的性能。

## <a name="do-not-store-ihttpcontextaccessorhttpcontext-in-a-field"></a>不要将 IHttpContextAccessor.HttpContext 存储在字段中

[IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext) `HttpContext` 从请求线程访问时，IHttpContextAccessor 将返回活动请求的。 `IHttpContextAccessor.HttpContext`**不** 应存储在字段或变量中。

请勿 **执行此操作：** 下面的示例将存储 `HttpContext` 在字段中，并稍后尝试使用它。

[!code-csharp[](performance-best-practices/samples/3.0/MyType.cs?name=snippet1)]

以上代码在构造函数中经常得到 Null 或不正确的 `HttpContext`。

**应该采用如下操作:**

* 在字段中保存 <xref:Microsoft.AspNetCore.Http.IHttpContextAccessor>。
* 在恰当的时机获取并使用 `HttpContext` ，并检查是否为 `null`。

[!code-csharp[](performance-best-practices/samples/3.0/MyType.cs?name=snippet2)]

## <a name="do-not-access-httpcontext-from-multiple-threads"></a>不要尝试在多线程下使用 HttpContext

`HttpContext`*不* 是线程安全的。 `HttpContext`并行从多个线程进行访问可能会导致未定义的行为，如挂起、崩溃和数据损坏。

请勿 **执行此操作：** 下面的示例执行三个并行请求，并在传出 HTTP 请求之前和之后记录传入的请求路径。 可以从多个线程访问请求路径，可能会并行进行。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/AsyncFirstController.cs?name=snippet1&highlight=25,28)]

**应该这样操作:** 以下示例在发出三个并行请求之前，从传入请求复制下文需要使用的数据。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/AsyncFirstController.cs?name=snippet2&highlight=6,8,22,28)]

## <a name="do-not-use-the-httpcontext-after-the-request-is-complete"></a>请求处理完成后不要使用 HttpContext

`HttpContext` 只有在 ASP.NET Core 管道处理活跃的 HTTP 请求时才可用。 整个 ASP.NET Core 管道是由异步代理组成的调用链，用于处理每个请求。 当 `Task` 从调用链完成并返回时,`HttpContext` 就会被回收。

请勿 **执行此操作：** 下面的示例使用 `async void` ，当达到第一个时，它将使 HTTP 请求完成 `await` ：

* 在 ASP.NET Core 应用程序中， 这是一个**完全错误** 的做法
* 在 HTTP 请求完成后访问 `HttpResponse`。
* 进程崩溃。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/AsyncBadVoidController.cs?name=snippet1)]

**应该进行如下操作:** 以下示例将 `Task` 返回给框架，因此，在操作完成之前， HTTP 请求不会完成。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/AsyncSecondController.cs?name=snippet1)]

## <a name="do-not-capture-the-httpcontext-in-background-threads"></a>不要在后台线程中使用 HttpContext

请勿 **执行此操作：** 下面的示例演示关闭 `HttpContext` 从 `Controller` 属性捕获。 这是一种不好的做法，因为工作项可以：

* 代码运行在 Http 请求作用域之外。
* 尝试读取错误的 `HttpContext`。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/FireAndForgetFirstController.cs?name=snippet1)]

**应该采用如下操作:**

* 在请求处理阶段将后台线程需要的数据全部进行复制。
* 不要使用 controller 的所有引用

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/FireAndForgetFirstController.cs?name=snippet2)]

后台任务最好采用托管服务进行操作。 有关更多信息，请参阅 [采用托管服务运行后台任务](xref:fundamentals/host/hosted-services) 。

## <a name="do-not-capture-services-injected-into-the-controllers-on-background-threads"></a>不要在后台线程获取注入到 controller 中的服务

请勿 **执行此操作：** 下面的示例演示关闭 `DbContext` `Controller` 操作从操作参数捕获。 这是一种不好的做法。  工作项可以在请求范围之外运行。 的 `ContosoDbContext` 作用域限定为请求，导致 `ObjectDisposedException` 。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/FireAndForgetSecondController.cs?name=snippet1)]

**应该采用如下操作:**

* 注入 <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory> ，并且在后台线程中创建新的作用域。 `IServiceScopeFactory` 是一个单例对象，所以这样没有问题。
* 在后台线程中创建新作用域注入依赖的服务。
* 不要引用 controller 的所有内容
* 不要从请求中读取 `ContocoDbContext`。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/FireAndForgetSecondController.cs?name=snippet2)]

以下高亮的的代码说明:

* 为后台操作创建新的作用域，并且从中获取需要的服务。
* 在正确的作用域中使用 `ContocoDbContext`，即只能在请求作用域中使用该对象。

[!code-csharp[](performance-best-practices/samples/3.0/Controllers/FireAndForgetSecondController.cs?name=snippet2&highlight=9-16)]

## <a name="do-not-modify-the-status-code-or-headers-after-the-response-body-has-started"></a>不要在响应正文已经开始发送时尝试修改 status code 或者 header

ASP.NET Core 不会缓冲 HTTP 响应正文。 当正文一旦开始发送:

* Header 就会与正文的数据包一起发送到客户端。
* 此时就无法修改 header 了。

请勿 **执行此操作：** 以下代码在响应已启动之后尝试添加响应标头：

[!code-csharp[](performance-best-practices/samples/3.0/Startup22.cs?name=snippet1)]

在上述的代码中，如果 `next()`已经开始写入响应，则`context.Response.Headers["test"] = "test value"; `将会抛出异常。

**应该采用如下操作:** 以下示例检查 HTTP 响应在修改 Header 之前是否已启动。

[!code-csharp[](performance-best-practices/samples/3.0/Startup22.cs?name=snippet2)]

**应该采用如下操作:** 以下示例使用 `HttpResponse.OnStarting` 来设置 Header，这样便可以在响应启动时将Header一次性写入到客户端。

通过这种方式，响应头将在响应开始时调用已注册的回调进行一次性写入。 如此这般便可以:

* 在恰当的时候进行响应头的修改或者覆盖。
* 不需要了解管道中的下一个 middleware 的行为。

[!code-csharp[](performance-best-practices/samples/3.0/Startup22.cs?name=snippet3)]

## <a name="do-not-call-next-if-you-have-already-started-writing-to-the-response-body"></a>如果已开始写入响应主体，则请不要调用 `next()`

仅当后续组件能够处理响应或时才调用它们，因此如果当前已经开始写入响应主体，后续操作就已经不再需要，并有可能引发异常情况。

## <a name="use-in-process-hosting-with-iis"></a>托管于 IIS 应该使用 In-process 模式

使用 in-process 模式托管， ASP.NET Core 应用程序将与 IIS 工作进程在同一进程中运行。 In-process 模式拥有比 out-of-process 更加优秀的性能表现，因为这样不需要将请求通过回环网络适配器进行代理中转。 回环网络适配器是将本机发送的网络流量重新转回本机的的网络适配器。 IIS 进程管理由 [Windows Process Activation Service (WAS)](/iis/manage/provisioning-and-managing-iis/features-of-the-windows-process-activation-service-was)来完成。

在 ASP.NET Core 3.0 和更高版本中的默认将采用 in-process 模式进行托管。

有关更多信息，请参阅 [在 Windows 上使用 IIS 托管 ASP.NET Core](xref:host-and-deploy/iis/index)