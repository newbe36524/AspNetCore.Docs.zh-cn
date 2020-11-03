---
title: ASP.NET Core 依赖注入
author: rick-anderson
description: 了解 ASP.NET Core 如何实现依赖注入和如何使用它。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 7/21/2020
no-loc:
- ASP.NET Core Identity
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/dependency-injection
ms.openlocfilehash: 99e0109ea4c2526e9f91a8a4df23c4557e9be83a
ms.sourcegitcommit: d7991068bc6b04063f4bd836fc5b9591d614d448
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91762303"
---
# <a name="dependency-injection-in-aspnet-core"></a>ASP.NET Core 依赖注入

::: moniker range=">= aspnetcore-3.0"

作者：[Kirk Larkin](https://twitter.com/serpent5)、[Steve Smith](https://ardalis.com/)、[Scott Addie](https://scottaddie.com) 和 [Brandon Dahler](https://github.com/brandondahler)

ASP.NET Core 支持依赖关系注入 (DI) 软件设计模式，这是一种在类及其依赖关系之间实现[控制反转 (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) 的技术。

有关 MVC 控制器中依赖关系注入的详细信息，请参阅 <xref:mvc/controllers/dependency-injection>。

有关选项的依赖项注入的详细信息，请参阅 <xref:fundamentals/configuration/options>。

本主题介绍 ASP.NET Core 中的依赖关系注入。 若要了解如何在控制台应用中使用依赖关系注入，请参阅 [.NET 中的依赖关系注入](/dotnet/core/extensions/dependency-injection)。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/dependency-injection/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="overview-of-dependency-injection"></a>依赖关系注入概述

依赖项是指另一个对象所依赖的对象。 使用其他类所依赖的 `WriteMessage` 方法检查以下 `MyDependency` 类：

```csharp
public class MyDependency
{
    public void WriteMessage(string message)
    {
        Console.WriteLine($"MyDependency.WriteMessage called. Message: {message}");
    }
}
```

类可以创建 `MyDependency` 类的实例，以便利用其 `WriteMessage` 方法。 在以下示例中，`MyDependency` 类是 `IndexModel` 类的依赖项：

```csharp
public class IndexModel : PageModel
{
    private readonly MyDependency _dependency = new MyDependency();

    public void OnGet()
    {
        _dependency.WriteMessage("IndexModel.OnGet created this message.");
    }
}
```

该类创建并直接依赖于 `MyDependency` 类。 代码依赖项（如前面的示例）会产生问题，应避免使用，原因如下：

* 要用不同的实现替换 `MyDependency`，必须修改 `IndexModel` 类。
* 如果 `MyDependency` 具有依赖项，则必须由 `IndexModel` 类对其进行配置。 在具有多个依赖于 `MyDependency` 的类的大型项目中，配置代码将分散在整个应用中。
* 这种实现很难进行单元测试。 应用需使用模拟或存根 `MyDependency` 类，而该类不能使用此方法。

依赖关系注入通过以下方式解决了这些问题：

* 使用接口或基类将依赖关系实现抽象化。
* 在服务容器中注册依赖关系。 ASP.NET Core 提供了一个内置的服务容器 <xref:System.IServiceProvider>。 服务通常已在应用的 `Startup.ConfigureServices` 方法中注册。
* 将服务注入到使用它的类的构造函数中。 框架负责创建依赖关系的实例，并在不再需要时将其释放。

在[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/dependency-injection/samples)中，`IMyDependency` 接口定义 `WriteMessage` 方法：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

此接口由具体类型 `MyDependency` 实现：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

示例应用使用具体类型 `MyDependency` 注册 `IMyDependency` 服务。 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped%2A> 方法使用范围内生存期（单个请求的生存期）注册服务。 本主题后面将介绍[服务生存期](#service-lifetimes)。

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency.cs?name=snippet1)]

在示例应用中，请求 `IMyDependency` 服务并用于调用 `WriteMessage` 方法：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index2.cshtml.cs?name=snippet1)]

通过使用 DI 模式，表示控制器：

* 不使用具体类型 `MyDependency`，仅使用它实现的 `IMyDependency` 接口。 这样可以轻松地更改控制器使用的实现，而无需修改控制器。
* 不创建 `MyDependency` 的实例，这由 DI 容器创建。

可以通过使用内置日志记录 API 来改善 `IMyDependency` 接口的实现：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet2)]

更新的 `ConfigureServices` 方法注册新的 `IMyDependency` 实现：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupMyDependency2.cs?name=snippet1)]

`MyDependency2` 依赖于 <xref:Microsoft.Extensions.Logging.ILogger%601>，并在构造函数中对其进行请求。 `ILogger<TCategoryName>` 是[框架提供的服务](#framework-provided-services)。

以链式方式使用依赖关系注入并不罕见。 每个请求的依赖关系相应地请求其自己的依赖关系。 容器解析图中的依赖关系并返回完全解析的服务。 必须被解析的依赖关系的集合通常被称为“依赖关系树”、“依赖关系图”或“对象图”。

容器通过利用[（泛型）开放类型](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types)解析 `ILogger<TCategoryName>`，而无需注册每个[（泛型）构造类型](/dotnet/csharp/language-reference/language-specification/types#constructed-types)。

在依赖项注入术语中，服务：

* 通常是向其他对象提供服务的对象，如 `IMyDependency` 服务。
* 与 Web 服务无关，尽管服务可能使用 Web 服务。

框架提供可靠的[日志记录](xref:fundamentals/logging/index)系统。 编写上述示例中的 `IMyDependency` 实现来演示基本的 DI，而不是来实现日志记录。 大多数应用都不需要编写记录器。 下面的代码演示如何使用默认日志记录，这不要求在 `ConfigureServices` 中注册任何服务：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/About.cshtml.cs?name=snippet)]

使用前面的代码时，无需更新 `ConfigureServices`，因为框架提供[日志记录](xref:fundamentals/logging/index)。

## <a name="services-injected-into-startup"></a>注入 Startup 的服务

服务可以注入 `Startup` 构造函数和 `Startup.Configure` 方法。

使用泛型主机 (<xref:Microsoft.Extensions.Hosting.IHostBuilder>) 时，只能将以下服务注入 `Startup` 构造函数：

* <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

任何向 DI 容器注册的服务都可以注入 `Startup.Configure` 方法：

```csharp
public void Configure(IApplicationBuilder app, ILogger<Startup> logger)
{
    ...
}
```

有关详细信息，请参阅 <xref:fundamentals/startup> 和[访问 Startup 中的配置](xref:fundamentals/configuration/index#access-configuration-in-startup)。

## <a name="register-groups-of-services-with-extension-methods"></a>使用扩展方法注册服务组

ASP.NET Core 框架使用一种约定来注册一组相关服务。 约定使用单个 `Add{GROUP_NAME}` 扩展方法来注册该框架功能所需的所有服务。 例如，<Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers> 扩展方法会注册 MVC 控制器所需的服务。

下面的代码通过个人用户帐户由 Razor 页面模板生成，并演示如何使用扩展方法 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 和 <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity%2A> 将其他服务添加到容器中：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/StartupEF.cs?name=snippet)]

[!INCLUDE[](~/includes/combine-di.md)]

## <a name="service-lifetimes"></a>服务生存期

可以使用以下任一生存期注册服务：

* 暂时
* 作用域
* 单例

下列各部分描述了上述每个生存期。 为每个注册的服务选择适当的生存期。 

### <a name="transient"></a>暂时

暂时生存期服务是每次从服务容器进行请求时创建的。 这种生存期适合轻量级、 无状态的服务。 向 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddTransient%2A> 注册暂时性服务。

在处理请求的应用中，在请求结束时会释放暂时服务。

### <a name="scoped"></a>作用域

作用域生存期服务针对每个客户端请求（连接）创建一次。 向 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped%2A> 注册范围内服务。

在处理请求的应用中，在请求结束时会释放有作用域的服务。

使用 Entity Framework Core 时，默认情况下 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 扩展方法使用范围内生存期来注册 `DbContext` 类型。

不要从单一实例解析范围内服务。 当处理后续请求时，它可能会导致服务处于不正确的状态。 可以：

* 从范围内或暂时性服务解析单一实例服务。
* 从其他范围内或暂时性服务解析范围内服务。

默认情况下在开发环境中，从具有较长生存期的其他服务解析服务将引发异常。 有关详细信息，请参阅[作用域验证](#sv)。

要在中间件中使用范围内服务，请使用以下方法之一：

* 将服务注入中间件的 `Invoke` 或 `InvokeAsync` 方法。 使用[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)会引发运行时异常，因为它强制使范围内服务的行为与单一实例类似。 [生存期和注册选项](#lifetime-and-registration-options)部分中的示例演示了 `InvokeAsync` 方法。
* 使用[基于工厂的中间件](xref:fundamentals/middleware/extensibility)。 使用此方法注册的中间件按客户端请求（连接）激活，这也使范围内服务可注入中间件的 `InvokeAsync` 方法。

有关详细信息，请参阅 <xref:fundamentals/middleware/write#per-request-middleware-dependencies>。

### <a name="singleton"></a>单例

创建单例生命周期服务的情况如下：

* 在首次请求它们时进行创建；或者
* 在向容器直接提供实现实例时由开发人员进行创建。 很少用到此方法。

每个后续请求都使用相同的实例。 如果应用需要单一实例行为，则允许服务容器管理服务的生存期。 不要实现单一实例设计模式，或提供代码来释放单一实例。 服务永远不应由解析容器服务的代码释放。 如果类型或工厂注册为单一实例，则容器自动释放单一实例。

向 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton%2A> 注册单一实例服务。 单一实例服务必须是线程安全的，并且通常在无状态服务中使用。

在处理请求的应用中，当应用关闭并释放 <xref:Microsoft.Extensions.DependencyInjection.ServiceProvider> 时，会释放单一实例服务。 由于应用关闭之前不释放内存，因此请考虑单一实例服务的内存使用。

> [!WARNING]
> 不要从单一实例解析范围内服务。 当处理后续请求时，它可能会导致服务处于不正确的状态。 可以从范围内或暂时性服务解析单一实例服务。

## <a name="service-registration-methods"></a>服务注册方法

框架提供了适用于特定场景的服务注册扩展方法：

<!-- Review: Auto disposal at end of app lifetime is not what you think of auto disposal  -->

| 方法                                                                                                                                                                              | 自动<br>对象 (object)<br>释放 | 多种<br>实现 | 传递参数 |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------:|:---------------------------:|:---------:|
| `Add{LIFETIME}<{SERVICE}, {IMPLEMENTATION}>()`<br>示例：<br>`services.AddSingleton<IMyDep, MyDep>();`                                                                             | 是                             | 是                         | 否        |
| `Add{LIFETIME}<{SERVICE}>(sp => new {IMPLEMENTATION})`<br>示例：<br>`services.AddSingleton<IMyDep>(sp => new MyDep());`<br>`services.AddSingleton<IMyDep>(sp => new MyDep(99));` | 是                             | 是                         | 是       |
| `Add{LIFETIME}<{IMPLEMENTATION}>()`<br>示例：<br>`services.AddSingleton<MyDep>();`                                                                                                | 是                             | 否                          | 否        |
| `AddSingleton<{SERVICE}>(new {IMPLEMENTATION})`<br>示例：<br>`services.AddSingleton<IMyDep>(new MyDep());`<br>`services.AddSingleton<IMyDep>(new MyDep(99));`                    | 否                              | 是                         | 是       |
| `AddSingleton(new {IMPLEMENTATION})`<br>示例：<br>`services.AddSingleton(new MyDep());`<br>`services.AddSingleton(new MyDep(99));`                                               | 否                              | 否                          | 是       |

要详细了解释放类型，请参阅[服务释放](#disposal-of-services)部分。 在[为测试模拟类型](xref:test/integration-tests#inject-mock-services)时，使用多个实现很常见。

框架还提供 `TryAdd{LIFETIME}` 扩展方法，只有当尚未注册某个实现时，才注册该服务。

在下面的示例中，对 `AddSingleton` 的调用会将 `MyDependency` 注册为 `IMyDependency`的实现。 对 `TryAddSingleton` 的调用没有任何作用，因为 `IMyDependency` 已有一个已注册的实现：

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
// The following line has no effect:
services.TryAddSingleton<IMyDependency, DifferentDependency>();
```

有关详细信息，请参阅：

* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAdd%2A>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddTransient%2A>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddScoped%2A>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddSingleton%2A>

[TryAddEnumerable(ServiceDescriptor)](xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddEnumerable%2A) 方法仅会在没有同一类型实现的情况下才注册该服务。 多个服务通过 `IEnumerable<{SERVICE}>` 解析。 注册服务时，开发人员应在尚未添加一个相同类型时添加实例。 通常库作者使用 `TryAddEnumerable` 来避免在容器中注册实现的多个副本。

在下面的示例中，对 `TryAddEnumerable` 的第一次调用会将 `MyDependency` 注册为 `IMyDependency1`的实现。 第二次调用向 `IMyDependency2` 注册 `MyDependency`。 第三次调用没有任何作用，因为 `IMyDependency1` 已有一个 `MyDependency` 的已注册的实现：

```csharp
public interface IMyDependency1 { }
public interface IMyDependency2 { }

public class MyDependency : IMyDependency1, IMyDependency2 { }

services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDependency1, MyDependency>());
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDependency2, MyDependency>());
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDependency1, MyDependency>());
```

服务注册通常与顺序无关，除了注册同一类型的多个实现时。

`IServiceCollection` 是 <xref:Microsoft.Extensions.DependencyInjection.ServiceDescriptor> 对象的集合。 以下示例演示如何通过创建和添加 `ServiceDescriptor` 来注册服务：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Startup5.cs?name=snippet)]

内置 `Add{LIFETIME}` 方法使用同一种方式。 相关示例请参阅 [AddScoped 源代码](https://github.com/dotnet/extensions/blob/v3.1.6/src/DependencyInjection/DI.Abstractions/src/ServiceCollectionServiceExtensions.cs#L216-L237)。

### <a name="constructor-injection-behavior"></a>构造函数注入行为

服务可使用以下方式来解析：

* <xref:System.IServiceProvider>
* <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities>:
  * 创建未在容器中注册的对象。
  * 与框架功能一起使用，例如[标记帮助程序](xref:mvc/views/tag-helpers/intro)、[MVC 控制器](xref:mvc/models/model-binding)和模型绑定器。

构造函数可以接受非依赖关系注入提供的参数，但参数必须分配默认值。

当服务由 `IServiceProvider` 或 `ActivatorUtilities` 解析时，[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)需要 public 构造函数。

当服务由 `ActivatorUtilities` 解析时，[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)要求只存在一个适用的构造函数。 支持构造函数重载，但其参数可以全部通过依赖注入来实现的重载只能存在一个。

## <a name="entity-framework-contexts"></a>实体框架上下文

默认情况下，使用[设置了范围的生存期](#service-lifetimes)将实体框架上下文添加到服务容器中，因为 Web 应用数据库操作通常将范围设置为客户端请求。 要使用其他生存期，请使用 <xref:Microsoft.Extensions.DependencyInjection.EntityFrameworkServiceCollectionExtensions.AddDbContext%2A> 重载来指定生存期。 给定生存期的服务不应使用生存期比服务生存期短的数据库上下文。

## <a name="lifetime-and-registration-options"></a>生存期和注册选项

为了演示服务生存期和注册选项之间的差异，请考虑以下接口，将任务表示为具有标识符 `OperationId` 的操作。 根据为以下接口配置操作服务的生存期的方式，容器在类请求时提供相同或不同的服务实例：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

以下 `Operation` 类实现了前面的所有接口。 `Operation` 构造函数生成 GUID，并将最后 4 个字符存储在 `OperationId` 属性中：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

<!--
An `OperationService` is registered that depends on each of the other `Operation` types. When `OperationService` is requested via dependency injection, it receives either a new instance of each service or an existing instance based on the lifetime of the dependent service.

* When transient services are created when requested from the container, the `OperationId` of the `IOperationTransient` service is different than the `OperationId` of the `OperationService`. `OperationService` receives a new instance of the `IOperationTransient` class. The new instance yields a different `OperationId`.
* When scoped services are created per client request, the `OperationId` of the `IOperationScoped` service is the same as that of `OperationService` within a client request. Across client requests, both services share a different `OperationId` value.
* When singleton and singleton-instance services are created once and used across all client requests and all services, the `OperationId` is constant across all service requests.

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Services/OperationService.cs?name=snippet1)]
-->

`Startup.ConfigureServices` 方法根据命名生存期创建 `Operation` 类的多个注册：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Startup2.cs?name=snippet1)]

示例应用一并演示了请求中和请求之间的对象生存期。 `IndexModel` 和中间件请求每种 `IOperation` 类型，并记录各自的 `OperationId`：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1)]

与 `IndexModel` 类似，中间件会解析相同的服务：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet)]

范围内服务必须在 `InvokeAsync` 方法中进行解析：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Middleware/MyMiddleware.cs?name=snippet2&highlight=2)]

记录器输出显示：

* 暂时性对象始终不同。 `IndexModel` 和中间件中的临时 `OperationId` 值不同。
* 范围内对象对每个请求而言是相同的，但在请求之间不同。
* 单一实例对象对于每个请求是相同的。

若要减少日志记录输出，请在 appsettings.Development.json 文件中设置“Logging:LogLevel:Microsoft:Error”：

[!code-json[](dependency-injection/samples/3.x/DependencyInjectionSample/appsettings.Development.json?highlight=7)]

## <a name="call-services-from-main"></a>从 main 调用服务

使用 [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A) 创建 <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> 以解析应用范围内的作用域服务。  此方法可以用于在启动时访问有作用域的服务以便运行初始化任务。

以下示例演示如何访问范围内 `IMyDependency` 服务并在 `Program.Main` 中调用其 `WriteMessage` 方法：

[!code-csharp[](dependency-injection/samples/3.x/DependencyInjectionSample/Program.cs?name=snippet)]

<a name="sv"></a>

## <a name="scope-validation"></a>作用域验证

如果应用在[开发环境](xref:fundamentals/environments)中运行，并调用 [CreateDefaultBuilder](xref:fundamentals/host/generic-host#default-builder-settings) 以生成主机，默认服务提供程序会执行检查，以确认以下内容：

* 没有从根服务提供程序解析到范围内服务。
* 未将范围内服务注入单一实例。

调用 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider%2A> 时创建根服务提供程序。 在启动提供程序和应用时，根服务提供程序的生存期对应于应用的生存期，并在关闭应用时释放。

有作用域的服务由创建它们的容器释放。 如果范围内服务创建于根容器，则该服务的生存期实际上提升至单一实例，因为根容器只会在应用关闭时将其释放。 验证服务作用域，将在调用 `BuildServiceProvider` 时收集这类情况。

有关详细信息，请参阅[作用域验证](xref:fundamentals/host/web-host#scope-validation)。

## <a name="request-services"></a>请求服务

ASP.NET Core 请求中可用的服务通过 [HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) 集合公开。 从请求内部请求服务时，将从 `RequestServices` 集合解析服务及其依赖项。

为每个请求创建一个作用域，`RequestServices` 公开作用域服务提供程序。 只要请求处于活动状态，所有作用域服务都有效。

> [!NOTE]
> 与解析 `RequestServices` 集合中的服务相比，以构造函数参数的形式请求依赖项是更优先的选择。 这样生成的类更易于测试。

## <a name="design-services-for-dependency-injection"></a>设计能够进行依赖关系注入的服务

在设计能够进行依赖注入的服务时：

* 避免有状态的、静态类和成员。 通过将应用设计为改用单一实例服务，避免创建全局状态。
* 避免在服务中直接实例化依赖类。 直接实例化会将代码耦合到特定实现。
* 不在服务中包含过多内容，确保设计规范，并易于测试。

如果一个类有过多注入依赖项，这可能表明该类拥有过多的责任并且违反了[单一责任原则 (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility)。 尝试通过将某些职责移动到一个新类来重构类。 请记住，Razor Pages 页面模型类和 MVC 控制器类应关注用户界面问题。

### <a name="disposal-of-services"></a>服务释放

容器为其创建的 <xref:System.IDisposable> 类型调用 <xref:System.IDisposable.Dispose%2A>。 从容器中解析的服务绝对不应由开发人员释放。 如果类型或工厂注册为单一实例，则容器自动释放单一实例。

在下面的示例中，服务由服务容器创建，并自动释放：

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Services/Service1.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup.cs?name=snippet)]

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Pages/Index.cshtml.cs?name=snippet)]

每次刷新索引页后，调试控制台显示以下输出：

```console
Service1: IndexModel.OnGet
Service2: IndexModel.OnGet
Service3: IndexModel.OnGet
Service1.Dispose
```

### <a name="services-not-created-by-the-service-container"></a>不由服务容器创建的服务

考虑下列代码：

[!code-csharp[](dependency-injection/samples/3.x/DIsample2/DIsample2/Startup2.cs?name=snippet)]

在上述代码中：

* 服务实例不是由服务容器创建的。
* 框架不会自动释放服务。
* 开发人员负责释放服务。

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a>暂时和共享实例的 IDisposable 指南

#### <a name="transient-limited-lifetime"></a>暂时、有限的生存期

**方案**

应用需要一个 <xref:System.IDisposable> 实例，该实例在以下任一情况下具有暂时性生存期：

* 在根范围（根容器）内解析实例。
* 应在作用域结束之前释放实例。

**解决方案**

使用工厂模式在父作用域外创建实例。  在这种情况下，应用通常会使用一个 `Create` 方法，该方法直接调用最终类型的构造函数。 如果最终类型具有其他依赖项，则工厂可以：

* 在其构造函数中接收 <xref:System.IServiceProvider>。
* 使用 <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> 在容器外部实例化实例，同时将容器用于其依赖项。

#### <a name="shared-instance-limited-lifetime"></a>共享实例，有限的生存期

**方案**

应用需要跨多个服务的共享 <xref:System.IDisposable> 实例，但 <xref:System.IDisposable> 实例应具有有限的生存期。

**解决方案**

为实例注册作用域生存期。 使用 <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A?displayProperty=nameWithType> 创建新 <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>。 使用作用域的 <xref:System.IServiceProvider> 获取所需的服务。 如果不再需要范围，请将其释放。

#### <a name="general-idisposable-guidelines"></a>一般 IDisposable 准则

* 不要为 <xref:System.IDisposable> 实例注册暂时性生存期。 请改用工厂模式。
* 不要在根范围内解析具有暂时性或范围内生存期的 <xref:System.IDisposable> 实例。 唯一的例外是应用创建/重新创建并释放 <xref:System.IServiceProvider> 的情况，但这不是理想模式。
* 通过 DI 接收 <xref:System.IDisposable> 依赖项不要求接收方自行实现 <xref:System.IDisposable>。 <xref:System.IDisposable> 依赖项的接收方不能对该依赖项调用 <xref:System.IDisposable.Dispose%2A>。
* 使用范围控制服务的生存期。  作用域不区分层次，并且在各作用域之间没有特定联系。

## <a name="default-service-container-replacement"></a>默认服务容器替换

内置的服务容器旨在满足框架和大多数消费者应用的需求。 我们建议使用内置容器，除非你需要的特定功能不受它支持，例如：

* 属性注入
* 基于名称的注入
* 子容器
* 自定义生存期管理
* 对迟缓初始化的 `Func<T>` 支持
* 基于约定的注册

以下第三方容器可用于 ASP.NET Core 应用：

* [Autofac](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html)
* [DryIoc](https://www.nuget.org/packages/DryIoc.Microsoft.DependencyInjection)
* [Grace](https://www.nuget.org/packages/Grace.DependencyInjection.Extensions)
* [LightInject](https://github.com/seesharper/LightInject.Microsoft.DependencyInjection)
* [Lamar](https://jasperfx.github.io/lamar/)
* [Stashbox](https://github.com/z4kn4fein/stashbox-extensions-dependencyinjection)
* [Unity](https://www.nuget.org/packages/Unity.Microsoft.DependencyInjection)

## <a name="thread-safety"></a>线程安全

创建线程安全的单一实例服务。 如果单一实例服务依赖于一个暂时服务，那么暂时服务可能也需要线程安全，具体取决于单一实例使用它的方式。

单个服务的工厂方法（例如 [AddSingleton\<TService>(IServiceCollection, Func\<IServiceProvider,TService>)](xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton%2A) 的第二个参数）不必是线程安全的。 像类型 (`static`) 构造函数一样，它保证仅由单个线程调用一次。

## <a name="recommendations"></a>建议

* 不支持基于`async/await` 和 `Task` 的服务解析。 由于 C# 不支持异步构造函数，因此请在同步解析服务后使用异步方法。
* 避免在服务容器中直接存储数据和配置。 例如，用户的购物车通常不应添加到服务容器中。 配置应使用 [选项模型](xref:fundamentals/configuration/options)。 同样，避免“数据持有者”对象，也就是仅仅为实现对另一个对象的访问而存在的对象。 最好通过 DI 请求实际项。
* 避免静态访问服务。 例如，避免将 [IApplicationBuilder.ApplicationServices](xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices) 捕获为静态字段或属性以便在其他地方使用。
* 使 DI 工厂保持快速且同步。
* 避免使用服务定位器模式。 例如，可以使用 DI 代替时，不要调用 <xref:System.IServiceProvider.GetService%2A>  来获取服务实例：

  **不正确：**

    ![错误代码](dependency-injection/_static/bad.png)

  **正确**：

  ```csharp
  public class MyClass
  {
      private readonly IOptionsMonitor<MyOptions> _optionsMonitor;

      public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
      {
          _optionsMonitor = optionsMonitor;
      }

      public void MyMethod()
      {
          var option = _optionsMonitor.CurrentValue.Option;

          ...
      }
  }
  ```
* 要避免的另一个服务定位器变体是注入需在运行时解析依赖项的工厂。 这两种做法混合了[控制反转](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion)策略。
* 避免静态访问 `HttpContext`（例如，[IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)）。

<a name="ASP0000"></a>
* 避免在 `ConfigureServices` 中调用 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider%2A>。 当开发人员想要在 `ConfigureServices` 中解析服务时，通常会调用 `BuildServiceProvider`。 例如，假设 `LoginPath` 从配置中加载。 避免采用以下方法：

  ![调用 BuildServiceProvider 的错误代码](~/fundamentals/dependency-injection/_static/badcodeX.png)

  在上图中，选择 `services.BuildServiceProvider` 下的绿色波浪线将显示以下 ASP0000 警告：

  > ASP0000 从应用程序代码调用“BuildServiceProvider”会导致创建单一实例服务的其他副本。 考虑依赖项注入服务等替代项作为“Configure”的参数。

  调用 `BuildServiceProvider` 会创建第二个容器，该容器可创建残缺的单一实例并导致跨多个容器引用对象图。

  获取 `LoginPath` 的正确方法是使用选项模式对 DI 的内置支持：

  [!code-csharp[](dependency-injection/samples/3.x/AntiPattern3/Startup.cs?name=snippet)]

* 可释放的暂时性服务由容器捕获以进行释放。 如果从顶级容器解析，这会变为内存泄漏。
* 启用范围验证，确保应用没有捕获范围内服务的单一实例。 有关详细信息，请参阅[作用域验证](#scope-validation)。

像任何一组建议一样，你可能会遇到需要忽略某建议的情况。 例外情况很少见，主要是框架本身内部的特殊情况。

DI 是静态/全局对象访问模式的替代方法。 如果将其与静态对象访问混合使用，则可能无法意识到 DI 的优点。

## <a name="recommended-patterns-for-multi-tenancy-in-di"></a>DI 中适用于多租户的推荐模式

[Orchard Core](https://github.com/OrchardCMS/OrchardCore) 是用于在 ASP.NET Core 上构建模块化多租户应用程序的应用程序框架。 有关详细信息，请参阅 [Orchard Core 文档](https://docs.orchardcore.net/en/dev/)。

请参阅 [Orchard Core 示例](https://github.com/OrchardCMS/OrchardCore.Samples)，获取有关如何仅使用 Orchard Core Framework 而无需任何 CMS 特定功能来构建模块化和多租户应用的示例。

## <a name="framework-provided-services"></a>框架提供的服务

`Startup.ConfigureServices` 方法注册应用使用的服务，包括 Entity Framework Core 和 ASP.NET Core MVC 等平台功能。 最初，提供给 `ConfigureServices` 的 `IServiceCollection` 具有框架定义的服务（具体取决于[主机配置方式](xref:fundamentals/index#host)）。 对于基于 ASP.NET Core 模板的应用，该框架会注册 250 个以上的服务。 

下表列出了框架注册的这些服务的一小部分：

| 服务类型                                                                                    | 生存期  |
|-------------------------------------------------------------------------------------------------|-----------|
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | 暂时 |
| <xref:Microsoft.Extensions.Hosting.IHostApplicationLifetime>                                    | 单例 |
| <xref:Microsoft.AspNetCore.Hosting.IWebHostEnvironment>                                         | 单例 |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName>                           | 单例 |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName>                     | 暂时 |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName>                     | 单例 |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName>                   | 暂时 |
| <xref:Microsoft.Extensions.Logging.ILogger%601?displayProperty=fullName>                        | 单例 |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName>                     | 单例 |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName>              | 单例 |
| <xref:Microsoft.Extensions.Options.IConfigureOptions%601?displayProperty=fullName>              | 暂时 |
| <xref:Microsoft.Extensions.Options.IOptions%601?displayProperty=fullName>                       | 单例 |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName>                             | 单例 |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName>                           | 单例 |

## <a name="additional-resources"></a>其他资源

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* [用于 DI 应用开发的 NDC 会议模式](https://www.youtube.com/watch?v=x-C-CNBVTaY)
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [在 ASP.NET Core 中释放 IDisposable 的四种方式](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [在 ASP.NET Core 中使用依赖关系注入编写干净代码 (MSDN) ](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [显式依赖关系原则](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [控制反转容器和依赖关系注入模式 (Martin Fowler)](https://www.martinfowler.com/articles/injection.html)
* [如何在 ASP.NET Core DI 中注册具有多个接口的服务](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Steve Smith](https://ardalis.com/)、[Scott Addie](https://scottaddie.com) 和 [Brandon Dahler](https://github.com/brandondahler)

ASP.NET Core 支持依赖关系注入 (DI) 软件设计模式，这是一种在类及其依赖关系之间实现[控制反转 (IoC)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion) 的技术。

有关 MVC 控制器中依赖关系注入的详细信息，请参阅 <xref:mvc/controllers/dependency-injection>。

[查看或下载示例代码](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/dependency-injection/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="overview-of-dependency-injection"></a>依赖关系注入概述

依赖项是指另一个对象所需的任何对象。 查看以下具有 `WriteMessage` 方法的 `MyDependency` 类，此类为应用中其他类所依赖：

```csharp
public class MyDependency
{
    public MyDependency()
    {
    }

    public Task WriteMessage(string message)
    {
        Console.WriteLine(
            $"MyDependency.WriteMessage called. Message: {message}");

        return Task.FromResult(0);
    }
}
```

可以创建 `MyDependency` 类的实例以使 `WriteMessage` 方法可用于其他类。 `MyDependency` 类是 `IndexModel` 类的依赖项：

```csharp
public class IndexModel : PageModel
{
    MyDependency _dependency = new MyDependency();

    public async Task OnGetAsync()
    {
        await _dependency.WriteMessage(
            "IndexModel.OnGetAsync created this message.");
    }
}
```

该类创建并直接依赖于 `MyDependency` 实例。 代码依赖关系（如前面的示例）会产生问题，应避免使用，原因如下：

* 若要用另一个实现替换 `MyDependency`，则必须修改类。
* 如果 `MyDependency` 具有依赖关系，则必须由类对其进行配置。 在具有多个依赖于 `MyDependency` 的类的大型项目中，配置代码将分散在整个应用中。
* 这种实现很难进行单元测试。 应用需使用模拟或存根 `MyDependency` 类，而该类不能使用此方法。

依赖关系注入通过以下方式解决了这些问题：

* 使用接口或基类将依赖关系实现抽象化。
* 在服务容器中注册依赖关系。 ASP.NET Core 提供了一个内置的服务容器 <xref:System.IServiceProvider>。 服务已在应用的 `Startup.ConfigureServices` 方法中注册。
* 将服务注入到使用它的类的构造函数中。 框架负责创建依赖关系的实例，并在不再需要时将其释放。

在[示例应用](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/dependency-injection/samples)中，`IMyDependency` 接口定义了服务为应用提供的方法：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IMyDependency.cs?name=snippet1)]

此接口由具体类型 `MyDependency` 实现：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/MyDependency.cs?name=snippet1)]

`MyDependency` 在其构造函数中请求一个 <xref:Microsoft.Extensions.Logging.ILogger`1>。 以链式方式使用依赖关系注入并不罕见。 每个请求的依赖关系相应地请求其自己的依赖关系。 容器解析图中的依赖关系并返回完全解析的服务。 必须被解析的依赖关系的集合通常被称为“依赖关系树”、“依赖关系图”或“对象图”。

必须在服务容器中注册 `IMyDependency` 和 `ILogger<TCategoryName>`。 `IMyDependency` 在 `Startup.ConfigureServices` 中注册。 `ILogger<TCategoryName>` 由日志记录抽象基础结构注册，因此它是框架默认注册的[框架提供的服务](#framework-provided-services)。

容器通过利用[（泛型）开放类型](/dotnet/csharp/language-reference/language-specification/types#open-and-closed-types)解析 `ILogger<TCategoryName>`，而无需注册每个[（泛型）构造类型](/dotnet/csharp/language-reference/language-specification/types#constructed-types)：

```csharp
services.AddSingleton(typeof(ILogger<>), typeof(Logger<>));
```

在示例应用中，使用具体类型 `MyDependency` 注册 `IMyDependency` 服务。 注册将服务生存期的范围限定为单个请求的生存期。 本主题后面将介绍[服务生存期](#service-lifetimes)。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=5)]

> [!NOTE]
> 每个 `services.Add{SERVICE_NAME}` 扩展方法添加（并可能配置）服务。 例如，`services.AddMvc()` 添加 Razor Pages 和 MVC 需要的服务。 我们建议应用遵循此约定。 将扩展方法置于 [Microsoft.Extensions.DependencyInjection](/dotnet/api/microsoft.extensions.dependencyinjection) 命名空间中以封装服务注册组。

如果服务的构造函数需要[内置类型](/dotnet/csharp/language-reference/keywords/built-in-types-table)（如 `string`），则可以使用[配置](xref:fundamentals/configuration/index)或[选项模式](xref:fundamentals/configuration/options)注入该类型：

```csharp
public class MyDependency : IMyDependency
{
    public MyDependency(IConfiguration config)
    {
        var myStringValue = config["MyStringKey"];

        // Use myStringValue
    }

    ...
}
```

可通过类的构造函数请求服务实例，在该函数中可使用服务并将其分配给私有字段。 该字段用于在整个类中根据需要访问服务。

在示例应用中，请求 `IMyDependency` 实例并将其用于调用服务的 `WriteMessage` 方法：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=3,6,13,29-30)]

## <a name="services-injected-into-startup"></a>注入 Startup 的服务

使用泛型主机 (<xref:Microsoft.Extensions.Hosting.IHostBuilder>) 时，只能将以下服务类型注入 `Startup` 构造函数：

* `IWebHostEnvironment`
* <xref:Microsoft.Extensions.Hosting.IHostEnvironment>
* <xref:Microsoft.Extensions.Configuration.IConfiguration>

服务可以注入 `Startup.Configure`：

```csharp
public void Configure(IApplicationBuilder app, IOptions<MyOptions> options)
{
    ...
}
```

有关详细信息，请参阅 <xref:fundamentals/startup>。

## <a name="framework-provided-services"></a>框架提供的服务

`Startup.ConfigureServices` 方法负责定义应用使用的服务，包括 Entity Framework Core 和 ASP.NET Core MVC 等平台功能。 最初，提供给 `ConfigureServices` 的 `IServiceCollection` 具有框架定义的服务（具体取决于[主机配置方式](xref:fundamentals/index#host)）。 基于 ASP.NET Core 模板的应用程序具有框架注册的数百个服务的情况并不少见。 下表列出了框架注册的服务的一小部分。

| 服务类型 | 生存期 |
| ------------ | -------- |
| <xref:Microsoft.AspNetCore.Hosting.Builder.IApplicationBuilderFactory?displayProperty=fullName> | 暂时 |
| <xref:Microsoft.AspNetCore.Hosting.IApplicationLifetime?displayProperty=fullName> | 单例 |
| <xref:Microsoft.AspNetCore.Hosting.IHostingEnvironment?displayProperty=fullName> | 单例 |
| <xref:Microsoft.AspNetCore.Hosting.IStartup?displayProperty=fullName> | 单例 |
| <xref:Microsoft.AspNetCore.Hosting.IStartupFilter?displayProperty=fullName> | 暂时 |
| <xref:Microsoft.AspNetCore.Hosting.Server.IServer?displayProperty=fullName> | 单例 |
| <xref:Microsoft.AspNetCore.Http.IHttpContextFactory?displayProperty=fullName> | 暂时 |
| <xref:Microsoft.Extensions.Logging.ILogger`1?displayProperty=fullName> | 单例 |
| <xref:Microsoft.Extensions.Logging.ILoggerFactory?displayProperty=fullName> | 单例 |
| <xref:Microsoft.Extensions.ObjectPool.ObjectPoolProvider?displayProperty=fullName> | 单例 |
| <xref:Microsoft.Extensions.Options.IConfigureOptions`1?displayProperty=fullName> | 暂时 |
| <xref:Microsoft.Extensions.Options.IOptions`1?displayProperty=fullName> | 单例 |
| <xref:System.Diagnostics.DiagnosticSource?displayProperty=fullName> | 单例 |
| <xref:System.Diagnostics.DiagnosticListener?displayProperty=fullName> | 单例 |

## <a name="register-additional-services-with-extension-methods"></a>使用扩展方法注册附加服务

当服务集合扩展方法可用于注册服务（及其依赖服务，如果需要）时，约定使用单个 `Add{SERVICE_NAME}` 扩展方法来注册该服务所需的所有服务。 以下代码是如何使用扩展方法 [AddDbContext\<TContext>](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) 和 <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentityCore*> 向容器添加附加服务的示例：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    ...

    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    ...
}
```

有关详细信息，请参阅 API 文档中的 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollection> 类。

## <a name="service-lifetimes"></a>服务生存期

为每个注册的服务选择适当的生存期。 可以使用以下生存期配置 ASP.NET Core 服务：

### <a name="transient"></a>暂时

暂时生存期服务 (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddTransient*>) 是每次从服务容器进行请求时创建的。 这种生存期适合轻量级、 无状态的服务。

在处理请求的应用中，在请求结束时会释放暂时服务。

### <a name="scoped"></a>作用域

作用域生存期服务 (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddScoped*>) 针对每个客户端请求（连接）创建一次。

在处理请求的应用中，在请求结束时会释放有作用域的服务。

> [!WARNING]
> 在中间件内使用有作用域的服务时，请将该服务注入至 `Invoke` 或 `InvokeAsync` 方法。 请不要通过[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)进行注入，因为它会强制服务的行为与单一实例类似。 有关详细信息，请参阅 <xref:fundamentals/middleware/write#per-request-middleware-dependencies>。

### <a name="singleton"></a>单例

单一实例生存期服务 (<xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*>) 是在第一次请求时（或者在运行 `Startup.ConfigureServices` 并且使用服务注册指定实例时）创建的。 每个后续请求都使用相同的实例。 如果应用需要单一实例行为，建议允许服务容器管理服务的生存期。 不要在类中实现单一实例设计模式并提供用户代码来管理对象的生存期。

在处理请求的应用中，当应用关闭并释放 <xref:Microsoft.Extensions.DependencyInjection.ServiceProvider> 时，会释放单一实例服务。

> [!WARNING]
> 从单一实例解析有作用域的服务很危险。 当处理后续请求时，它可能会导致服务处于不正确的状态。

## <a name="service-registration-methods"></a>服务注册方法

服务注册扩展方法提供适用于特定场景的重载。

| 方法 | 自动<br>对象 (object)<br>释放 | 多种<br>实现 | 传递参数 |
| ------ | :-----------------------------: | :-------------------------: | :-------: |
| `Add{LIFETIME}<{SERVICE}, {IMPLEMENTATION}>()`<br>示例：<br>`services.AddSingleton<IMyDep, MyDep>();` | 是 | 是 | 否 |
| `Add{LIFETIME}<{SERVICE}>(sp => new {IMPLEMENTATION})`<br>示例：<br>`services.AddSingleton<IMyDep>(sp => new MyDep());`<br>`services.AddSingleton<IMyDep>(sp => new MyDep("A string!"));` | 是 | 是 | 是 |
| `Add{LIFETIME}<{IMPLEMENTATION}>()`<br>示例：<br>`services.AddSingleton<MyDep>();` | 是 | 否 | 否 |
| `AddSingleton<{SERVICE}>(new {IMPLEMENTATION})`<br>示例：<br>`services.AddSingleton<IMyDep>(new MyDep());`<br>`services.AddSingleton<IMyDep>(new MyDep("A string!"));` | 否 | 是 | 是 |
| `AddSingleton(new {IMPLEMENTATION})`<br>示例：<br>`services.AddSingleton(new MyDep());`<br>`services.AddSingleton(new MyDep("A string!"));` | 否 | 否 | 是 |

要详细了解释放类型，请参阅[服务释放](#disposal-of-services)部分。 多个实现的常见场景是[为了测试而模拟各个类型](xref:test/integration-tests#inject-mock-services)。

`TryAdd{LIFETIME}` 方法仅当尚未注册实现时，注册该服务。

在以下示例中，第一行向 `IMyDependency` 注册 `MyDependency`。 第二行没有任何作用，因为 `IMyDependency` 已有一个已注册的实现：

```csharp
services.AddSingleton<IMyDependency, MyDependency>();
// The following line has no effect:
services.TryAddSingleton<IMyDependency, DifferentDependency>();
```

有关详情，请参阅：

* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAdd*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddTransient*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddScoped*>
* <xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddSingleton*>

[TryAddEnumerable(ServiceDescriptor)](xref:Microsoft.Extensions.DependencyInjection.Extensions.ServiceCollectionDescriptorExtensions.TryAddEnumerable*) 方法仅当没有同一类型的实现时，注册该服务。 多个服务通过 `IEnumerable<{SERVICE}>` 解析。 注册服务时，开发人员只希望在尚未添加一个相同类型时添加实例。 通常情况下，库创建者使用此方法来避免在容器中注册一个实例的两个副本。

在以下示例中，第一行向 `IMyDep1` 注册 `MyDep`。 第二行向 `IMyDep2` 注册 `MyDep`。 第三行没有任何作用，因为 `IMyDep1` 已有一个 `MyDep` 的已注册的实现：

```csharp
public interface IMyDep1 {}
public interface IMyDep2 {}

public class MyDep : IMyDep1, IMyDep2 {}

services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep2, MyDep>());
// Two registrations of MyDep for IMyDep1 is avoided by the following line:
services.TryAddEnumerable(ServiceDescriptor.Singleton<IMyDep1, MyDep>());
```

### <a name="constructor-injection-behavior"></a>构造函数注入行为

服务可以通过两种机制来解析：

* <xref:System.IServiceProvider>
* <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities>：允许在依赖关系注入容器中创建没有服务注册的对象。 `ActivatorUtilities` 用于面向用户的抽象，例如标记帮助器、MVC 控制器和模型绑定器。

构造函数可以接受非依赖关系注入提供的参数，但参数必须分配默认值。

当服务由 `IServiceProvider` 或 `ActivatorUtilities` 解析时，[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)需要 public 构造函数。

当服务由 `ActivatorUtilities` 解析时，[构造函数注入](xref:mvc/controllers/dependency-injection#constructor-injection)要求只存在一个适用的构造函数。 支持构造函数重载，但其参数可以全部通过依赖注入来实现的重载只能存在一个。

## <a name="entity-framework-contexts"></a>实体框架上下文

通常将实体框架上下文添加到使用[作用域生存期](#service-lifetimes)的服务容器中，因为 Web 应用数据库操作通常将范围设置为客户端请求。 如果在注册数据库上下文时，[AddDbContext\<TContext>](/dotnet/api/microsoft.extensions.dependencyinjection.entityframeworkservicecollectionextensions.adddbcontext) 重载未指定生存期，则默认生存期为作用域。 给定生存期的服务不应使用生存期比服务短的数据库上下文。

## <a name="lifetime-and-registration-options"></a>生存期和注册选项

为了演示生存期和注册选项之间的差异，请考虑以下接口，将任务表示为具有唯一标识符 `OperationId` 的操作。 根据为以下接口配置操作服务的生存期的方式，容器在类请求时提供相同或不同的服务实例：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Interfaces/IOperation.cs?name=snippet1)]

接口在 `Operation` 类中实现。 `Operation` 构造函数将生成一个 GUID（如果未提供）：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Models/Operation.cs?name=snippet1)]

注册 `OperationService` 取决于，每个其他 `Operation` 类型。 当通过依赖关系注入请求 `OperationService` 时，它将基于依赖服务的生存期接收每个服务的新实例或现有实例。

* 如果从容器请求时创建了临时服务，则 `IOperationTransient` 服务的 `OperationId` 与 `OperationService` 的 `OperationId` 不同。 `OperationService` 将接收 `IOperationTransient` 类的新实例。 新实例将生成一个不同的 `OperationId`。
* 如果按客户端请求创建有作用域的服务，则 `IOperationScoped` 服务的 `OperationId` 与客户端请求中 `OperationService` 的该 ID 相同。 在客户端请求中，两个服务共享不同的 `OperationId` 值。
* 如果创建了一次单例服务和单例实例服务并在所有客户端请求和所有服务中使用，则 `OperationId` 在所有服务请求中保持不变。

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Services/OperationService.cs?name=snippet1)]

在 `Startup.ConfigureServices` 中，根据其指定的生存期，将每个类型添加到容器中：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Startup.cs?name=snippet1&highlight=6-9,12)]

`IOperationSingletonInstance` 服务正在使用已知 ID 为 `Guid.Empty` 的特定实例。 此类型在使用时很明显（其 GUID 全部为零）。

示例应用演示了各个请求中和请求之间的对象生存期。 示例应用的 `IndexModel` 请求每种 `IOperation` 类型和 `OperationService`。 然后，页面通过属性分配显示所有页面模型类和服务的 `OperationId` 值：

[!code-csharp[](dependency-injection/samples/2.x/DependencyInjectionSample/Pages/Index.cshtml.cs?name=snippet1&highlight=7-11,14-18,21-25)]

以下两个输出显示了两个请求的结果：

**第一个请求：**

控制器操作：

暂时性：d233e165-f417-469b-a866-1cf1935d2518  
作用域：5d997e2d-55f5-4a64-8388-51c4e3a1ad19  
单一实例：01271bc1-9e31-48e7-8f7c-7261b040ded9  
实例：00000000-0000-0000-0000-000000000000

`OperationService` 操作：

暂时性：c6b049eb-1318-4e31-90f1-eb2dd849ff64  
作用域：5d997e2d-55f5-4a64-8388-51c4e3a1ad19  
单一实例：01271bc1-9e31-48e7-8f7c-7261b040ded9  
实例：00000000-0000-0000-0000-000000000000

**第二个请求：**

控制器操作：

暂时性：b63bd538-0a37-4ff1-90ba-081c5138dda0  
作用域：31e820c5-4834-4d22-83fc-a60118acb9f4  
单一实例：01271bc1-9e31-48e7-8f7c-7261b040ded9  
实例：00000000-0000-0000-0000-000000000000

`OperationService` 操作：

暂时性：c4cbacb8-36a2-436d-81c8-8c1b78808aaf  
作用域：31e820c5-4834-4d22-83fc-a60118acb9f4  
单一实例：01271bc1-9e31-48e7-8f7c-7261b040ded9  
实例：00000000-0000-0000-0000-000000000000

观察哪个 `OperationId` 值会在一个请求之内和不同请求之间变化：

* 暂时性对象始终不同。 第一个和第二个客户端请求的暂时性 `OperationId` 值对于 `OperationService` 操作和在客户端请求内都是不同的。 为每个服务请求和客户端请求提供了一个新实例。
* 作用域对象在一个客户端请求中是相同的，但在多个客户端请求中是不同的。
* 单一实例对象对每个对象和每个请求都是相同的（不管 `Startup.ConfigureServices` 中是否提供 `Operation` 实例）。

## <a name="call-services-from-main"></a>从 main 调用服务

使用 [IServiceScopeFactory.CreateScope](xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope*) 创建 <xref:Microsoft.Extensions.DependencyInjection.IServiceScope> 以解析应用范围内的作用域服务。  此方法可以用于在启动时访问有作用域的服务以便运行初始化任务。 以下示例演示如何在 `Program.Main` 中获取 `MyScopedService` 的上下文：

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;

public class Program
{
    public static async Task Main(string[] args)
    {
        var host = CreateWebHostBuilder(args).Build();

        using (var serviceScope = host.Services.CreateScope())
        {
            var services = serviceScope.ServiceProvider;

            try
            {
                var serviceContext = services.GetRequiredService<MyScopedService>();
                // Use the context here
            }
            catch (Exception ex)
            {
                var logger = services.GetRequiredService<ILogger<Program>>();
                logger.LogError(ex, "An error occurred.");
            }
        }
    
        await host.RunAsync();
    }

    public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>();
}
```

## <a name="scope-validation"></a>作用域验证

如果在开发环境中运行应用，默认的服务提供程序会执行检查，从而确认以下内容：

* 没有从根服务提供程序直接或间接解析到有作用域的服务。
* 未将有作用域的服务直接或间接注入到单一实例。

调用 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionContainerBuilderExtensions.BuildServiceProvider*> 时创建根服务提供程序。 在启动提供程序和应用时，根服务提供程序的生存期对应于应用/服务的生存期，并在关闭应用时释放。

有作用域的服务由创建它们的容器释放。 如果作用域服务创建于根容器，则该服务的生存期会实际上提升至单一实例，因为根容器只会在应用/服务关闭时将其释放。 验证服务作用域，将在调用 `BuildServiceProvider` 时收集这类情况。

有关详细信息，请参阅 <xref:fundamentals/host/web-host#scope-validation>。   

## <a name="request-services"></a>请求服务

来自 `HttpContext` 的 ASP.NET Core 请求中可用的服务通过 [HttpContext.RequestServices](xref:Microsoft.AspNetCore.Http.HttpContext.RequestServices) 集合公开。

请求服务表示作为应用的一部分配置和请求的服务。 当对象指定依赖关系时，`RequestServices`（而不是 `ApplicationServices`）中的类型将满足这些要求。

通常，应用不应直接使用这些属性。 相反，通过类构造函数请求类所需的类型，并允许框架注入依赖关系。 这样生成的类更易于测试。

> [!NOTE]
> 与访问 `RequestServices` 集合相比，以构造函数参数的形式请求依赖项是更优先的选择。

## <a name="design-services-for-dependency-injection"></a>设计能够进行依赖关系注入的服务

最佳做法是：

* 设计服务以使用依赖关系注入来获取其依赖关系。
* 避免有状态的、静态类和成员。 将应用设计为改用单一实例服务，可避免创建全局状态。
* 避免在服务中直接实例化依赖类。 直接实例化会将代码耦合到特定实现。
* 不在应用类中包含过多内容，确保设计规范，并易于测试。

如果一个类似乎有过多的依赖关系注入，这通常表明该类拥有过多的责任并且违反了[单一责任原则 (SRP)](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#single-responsibility)。 尝试通过将某些职责移动到一个新类来重构类。 请记住，Razor Pages 页面模型类和 MVC 控制器类应关注用户界面问题。 业务规则和数据访问实现细节应保留在适用于这些[分离的关注点](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns)的类中。

### <a name="disposal-of-services"></a>服务释放

容器为其创建的 <xref:System.IDisposable> 类型调用 <xref:System.IDisposable.Dispose*>。 如果实例是通过用户代码添加到容器中，则不会自动释放该实例。

在下面的示例中，服务由服务容器创建，并自动释放：

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public interface IService3 {}
public class Service3 : IService3, IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<Service1>();
    services.AddSingleton<Service2>();
    services.AddSingleton<IService3>(sp => new Service3());
}
```

如下示例中：

* 服务实例不是由服务容器创建的。
* 框架不知道预期的服务生存期。
* 框架不会自动释放服务。
* 服务如果未通过开发者代码显式释放，则会一直保留，直到应用关闭。

```csharp
public class Service1 : IDisposable {}
public class Service2 : IDisposable {}

public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<Service1>(new Service1());
    services.AddSingleton(new Service2());
}
```

### <a name="idisposable-guidance-for-transient-and-shared-instances"></a>暂时和共享实例的 IDisposable 指南

#### <a name="transient-limited-lifetime"></a>暂时、有限的生存期

**方案**

应用需要一个 <xref:System.IDisposable> 实例，该实例在以下任一情况下具有暂时性生存期：

* 在根作用域内解析实例。
* 应在作用域结束之前释放实例。

**解决方案**

使用工厂模式在父作用域外创建实例。  在这种情况下，应用通常会使用一个 `Create` 方法，该方法直接调用最终类型的构造函数。 如果最终类型具有其他依赖项，则工厂可以：

* 在其构造函数中接收 <xref:System.IServiceProvider>。
* 使用 <xref:Microsoft.Extensions.DependencyInjection.ActivatorUtilities.CreateInstance%2A?displayProperty=nameWithType> 在容器外部实例化实例，同时将容器用于其依赖项。

#### <a name="shared-instance-limited-lifetime"></a>共享实例，有限的生存期

**方案**

应用需要跨多个服务的共享 <xref:System.IDisposable> 实例，但 <xref:System.IDisposable> 应具有有限的生存期。

**解决方案**

为实例注册作用域生存期。  使用 <xref:Microsoft.Extensions.DependencyInjection.IServiceScopeFactory.CreateScope%2A?displayProperty=nameWithType> 启动并创建新的 <xref:Microsoft.Extensions.DependencyInjection.IServiceScope>。 使用作用域的 <xref:System.IServiceProvider> 获取所需的服务。 在生存期应结束时释放作用域。

#### <a name="general-guidelines"></a>通用准则

* 不要为<xref:System.IDisposable> 实例注册暂时作用域。 请改用工厂模式。
* 不要在根作用域内解析暂时或作用域 <xref:System.IDisposable> 实例。 唯一的一般性例外是应用创建/重建和释放 <xref:System.IServiceProvider> 的情况，这不是理想的模式。
* 通过 DI 接收 <xref:System.IDisposable> 依赖项不要求接收方自行实现 <xref:System.IDisposable>。 <xref:System.IDisposable> 依赖项的接收方不能对该依赖项调用 <xref:System.IDisposable.Dispose%2A>。
* 作用域应该用于控制服务的生存期。  作用域不区分层次，并且在各作用域之间没有特定联系。

## <a name="default-service-container-replacement"></a>默认服务容器替换

内置的服务容器旨在满足框架和大多数消费者应用的需求。 我们建议使用内置容器，除非你需要的特定功能不受内置容器支持，例如：

* 属性注入
* 基于名称的注入
* 子容器
* 自定义生存期管理
* 对迟缓初始化的 `Func<T>` 支持
* 基于约定的注册

以下第三方容器可用于 ASP.NET Core 应用：

* [Autofac](https://autofac.readthedocs.io/en/latest/integration/aspnetcore.html)
* [DryIoc](https://www.nuget.org/packages/DryIoc.Microsoft.DependencyInjection)
* [Grace](https://www.nuget.org/packages/Grace.DependencyInjection.Extensions)
* [LightInject](https://github.com/seesharper/LightInject.Microsoft.DependencyInjection)
* [Lamar](https://jasperfx.github.io/lamar/)
* [Stashbox](https://github.com/z4kn4fein/stashbox-extensions-dependencyinjection)
* [Unity](https://www.nuget.org/packages/Unity.Microsoft.DependencyInjection)

### <a name="thread-safety"></a>线程安全

创建线程安全的单一实例服务。 如果单例服务依赖于一个暂时服务，那么暂时服务可能也需要线程安全，具体取决于单例使用它的方式。

单个服务的工厂方法（例如 [AddSingleton\<TService>(IServiceCollection, Func\<IServiceProvider,TService>)](xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*) 的第二个参数）不必是线程安全的。 像类型 (`static`) 构造函数一样，它保证单个线程只调用一次。

## <a name="recommendations"></a>建议

* 不支持基于 `async/await` 和 `Task` 的服务解析。 C# 不支持异步构造函数；因此建议模式是在同步解析服务后使用异步方法。

* 避免在服务容器中直接存储数据和配置。 例如，用户的购物车通常不应添加到服务容器中。 配置应使用 [选项模型](xref:fundamentals/configuration/options)。 同样，避免"数据持有者"对象，也就是仅仅为实现对某些其他对象的访问而存在的对象。 最好通过 DI 请求实际项。
* 避免静态访问服务。 例如，避免静态键入 [IApplicationBuilder.ApplicationServices](xref:Microsoft.AspNetCore.Builder.IApplicationBuilder.ApplicationServices) 以便在其他地方使用。

* 避免使用服务定位器模式，该模式混合了[控制反转](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#dependency-inversion)策略。
  * 可以使用 DI 代替时，不要调用 <xref:System.IServiceProvider.GetService*> 来获取服务实例：

    **不正确：**

    ```csharp
    public class MyClass()
   
      public void MyMethod()
      {
          var optionsMonitor = 
              _services.GetService<IOptionsMonitor<MyOptions>>();
          var option = optionsMonitor.CurrentValue.Option;
   
          ...
      }
      ```
   
    **正确**：

    ```csharp
    public class MyClass
    {
        private readonly IOptionsMonitor<MyOptions> _optionsMonitor;

        public MyClass(IOptionsMonitor<MyOptions> optionsMonitor)
        {
            _optionsMonitor = optionsMonitor;
        }

        public void MyMethod()
        {
            var option = _optionsMonitor.CurrentValue.Option;

            ...
        }
    }
    ```

* 避免注入一个使用 <xref:System.IServiceProvider.GetService*> 在运行时解析依赖项的中心。
* 避免静态访问 `HttpContext`（例如，[IHttpContextAccessor.HttpContext](xref:Microsoft.AspNetCore.Http.IHttpContextAccessor.HttpContext)）。

像任何一组建议一样，你可能会遇到需要忽略某建议的情况。 例外情况很少见，主要是框架本身内部的特殊情况。

DI 是静态/全局对象访问模式的替代方法。 如果将其与静态对象访问混合使用，则可能无法意识到 DI 的优点。

## <a name="additional-resources"></a>其他资源

* <xref:mvc/views/dependency-injection>
* <xref:mvc/controllers/dependency-injection>
* <xref:security/authorization/dependencyinjection>
* <xref:blazor/fundamentals/dependency-injection>
* <xref:fundamentals/startup>
* <xref:fundamentals/middleware/extensibility>
* [在 ASP.NET Core 中释放 IDisposable 的四种方式](https://andrewlock.net/four-ways-to-dispose-idisposables-in-asp-net-core/)
* [在 ASP.NET Core 中使用依赖关系注入编写干净代码 (MSDN) ](/archive/msdn-magazine/2016/may/asp-net-writing-clean-code-in-asp-net-core-with-dependency-injection)
* [显式依赖关系原则](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)
* [控制反转容器和依赖关系注入模式 (Martin Fowler)](https://www.martinfowler.com/articles/injection.html)
* [如何在 ASP.NET Core DI 中注册具有多个接口的服务](https://andrewlock.net/how-to-register-a-service-with-multiple-interfaces-for-in-asp-net-core-di/)

::: moniker-end
