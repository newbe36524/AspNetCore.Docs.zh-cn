---
title: Razor与 ASP.NET Core 中的应用程序部件共享控制器、视图和页面等
author: rick-anderson
description: Razor与 ASP.NET Core 中的应用程序部件共享控制器、视图、页面及更多内容
ms.author: riande
ms.date: 11/11/2019
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
uid: mvc/extensibility/app-parts
ms.openlocfilehash: 33deb5ff794982e0c074186bb2abb88344e8a116
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061179"
---
# <a name="share-controllers-views-no-locrazor-pages-and-more-with-application-parts"></a>与应用程序部件共享控制器、视图、 Razor 页和更多内容

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts)（[如何下载](xref:index#how-to-download-a-sample)）

应用程序部件是对应用资源的抽象化。  应用程序部件允许 ASP.NET Core 发现控制器、查看组件、标记帮助程序、 Razor 页面、razor 编译源等。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> 是应用程序部件。 `AssemblyPart` 用于封装程序集引用，并公开类型和编译引用。

[功能提供程序](#fp)使用应用程序部件填充 ASP.NET Core 应用的功能。 应用程序部件的主要用例是将应用配置为从程序集中发现（或避免加载）ASP.NET Core 功能。 例如，可能需要在多个应用之间共享通用功能。 使用应用程序部件，你可以使用多个应用共享包含控制器、视图、 Razor 页面、razor 编译源、标记帮助程序以及更多应用程序 (DLL) 的程序集。 相对于在多个项目中复制代码，首选共享程序集。

ASP.NET Core 应用从 <xref:System.Web.WebPages.ApplicationPart> 加载功能。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> 类表示受程序集支持的应用程序部件。

## <a name="load-aspnet-core-features"></a>加载 ASP.NET Core 功能

使用 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> 和 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> 类发现并加载 ASP.NET Core 功能（控制器、视图组件等）。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> 跟踪可用的应用程序部件和功能提供程序。 在 `Startup.ConfigureServices` 中配置 `ApplicationPartManager`：

[!code-csharp[](./app-parts/3.0sample1/WebAppParts/Startup.cs?name=snippet)]

以下代码提供使用 `AssemblyPart` 配置 `ApplicationPartManager` 的可选方法：

[!code-csharp[](./app-parts/3.0sample1/WebAppParts/Startup2.cs?name=snippet)]

前面的两个代码示例从程序集加载 `SharedController`。 `SharedController` 未在该应用的项目中。 请参阅 [WebAppParts 解决方案](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/3.0sample1/WebAppParts)示例下载。

### <a name="include-views"></a>包含视图

使用类库[ Razor 将视图](xref:razor-pages/ui-class)包含在程序集中。

### <a name="prevent-loading-resources"></a>阻止加载资源

可以使用应用程序部件来避免加载特定程序集或位置中的资源。  添加或删除 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> 集合的成员，将隐藏或提供资源。 `ApplicationParts` 集合中条目的顺序并不重要。 在使用 `ApplicationPartManager` 配置容器中的服务之前，对该类进行配置。 例如，在调用 `AddControllersAsServices` 之前配置 `ApplicationPartManager`。 在 `ApplicationParts` 集合上调用 `Remove`，将删除资源。

`ApplicationPartManager` 包括以下内容的部件：

* 应用的程序集和依赖程序集。
* `Microsoft.AspNetCore.Mvc.ApplicationParts.CompiledRazorAssemblyPart`
* `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation`
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

<a name="fp"></a>

## <a name="feature-providers"></a>功能提供程序

应用程序功能提供程序用于检查应用程序部件，并为这些部件提供功能。 以下 ASP.NET Core 功能有内置功能提供程序：

* <xref:Microsoft.AspNetCore.Mvc.Controllers.ControllerFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.TagHelpers.TagHelperFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.Compilation.MetadataReferenceFeatureProvider>
* <xref:Microsoft.AspNetCore.Mvc.Razor.Compilation.ViewsFeatureProvider>
* `internal class`[ Razor CompiledItemFeatureProvider](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Razor/src/ApplicationParts/RazorCompiledItemFeatureProvider.cs#L14)

功能提供程序从 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1> 继承，其中 `T` 是功能的类型。 可以为上面列出的任意功能类型实现功能提供程序。 `ApplicationPartManager.FeatureProviders` 中的功能提供程序的顺序可能影响运行时行为。 较晚添加的提供程序可能会影响较早添加的提供程序执行的操作。

### <a name="display-available-features"></a>显示可用功能

通过[依存关系注入](../../fundamentals/dependency-injection.md)请求 `ApplicationPartManager` 即可以枚举应用的可用功能：

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

[下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample2)使用前面的代码显示应用功能：

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```

## <a name="discovery-in-application-parts"></a>应用程序部件中的发现

使用应用程序部件进行开发时，会遇到 HTTP 404 错误且并不鲜见。 发生这些错误的原因通常是由于未满足某项针对应用程序部件发现方式的基本要求。 如果应用返回 HTTP 404 错误，请验证是否满足以下要求：

* 需要将 `applicationName` 设置设置为用于发现的根程序集。 用于发现的根程序集通常是入口点程序集。
* 根程序集需要引用用于发现的部件。 引用可以是直接的，也可以是可传递的。
* 根程序集需要引用 Web SDK。 该框架的逻辑会将属性标记到用于发现的根程序集中。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts)（[如何下载](xref:index#how-to-download-a-sample)）

应用程序部件是对应用资源的抽象化。  应用程序部件允许 ASP.NET Core 发现控制器、查看组件、标记帮助程序、 Razor 页面、razor 编译源等。 [AssemblyPart](/dotnet/api/microsoft.aspnetcore.mvc.applicationparts.assemblypart#Microsoft_AspNetCore_Mvc_ApplicationParts_AssemblyPart) 是一种应用程序部件。 `AssemblyPart` 用于封装程序集引用，并公开类型和编译引用。

*功能提供程序* 使用应用程序部件填充 ASP.NET Core 应用的功能。 应用程序部件的主要用例是将应用配置为从程序集中发现（或避免加载）ASP.NET Core 功能。 例如，可能需要在多个应用之间共享通用功能。 使用应用程序部件，你可以使用多个应用共享包含控制器、视图、 Razor 页面、razor 编译源、标记帮助程序以及更多应用程序 (DLL) 的程序集。 相对于在多个项目中复制代码，首选共享程序集。

ASP.NET Core 应用从 <xref:System.Web.WebPages.ApplicationPart> 加载功能。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.AssemblyPart> 类表示受程序集支持的应用程序部件。

## <a name="load-aspnet-core-features"></a>加载 ASP.NET Core 功能

使用 `ApplicationPart` 和 `AssemblyPart` 类发现并加载 ASP.NET Core 功能（控制器、视图组件等）。 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.ApplicationPartManager> 跟踪可用的应用程序部件和功能提供程序。 在 `Startup.ConfigureServices` 中配置 `ApplicationPartManager`：

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup.cs?name=snippet)]

以下代码提供使用 `AssemblyPart` 配置 `ApplicationPartManager` 的可选方法：

[!code-csharp[](./app-parts/sample1/WebAppParts/Startup2.cs?name=snippet)]

前面的两个代码示例从程序集加载 `SharedController`。 `SharedController` 未在应用程序的项目中。 请参阅 [WebAppParts 解决方案](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample1/WebAppParts)示例下载。

### <a name="include-views"></a>包含视图

使用类库[ Razor 将视图](xref:razor-pages/ui-class)包含在程序集中。

### <a name="prevent-loading-resources"></a>阻止加载资源

可以使用应用程序部件来避免加载特定程序集或位置中的资源。  添加或删除 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> 集合的成员，将隐藏或提供资源。 `ApplicationParts` 集合中条目的顺序并不重要。 在使用 `ApplicationPartManager` 配置容器中的服务之前，对该类进行配置。 例如，在调用 `AddControllersAsServices` 之前配置 `ApplicationPartManager`。 在 `ApplicationParts` 集合上调用 `Remove`，将删除资源。

以下代码使用 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts> 删除应用中的 `MyDependentLibrary`：[!code-csharp[](./app-parts/sample1/WebAppParts/StartupRm.cs?name=snippet)]

`ApplicationPartManager` 包括以下内容的部件：

* 应用的程序集和依赖程序集。
* `Microsoft.AspNetCore.Mvc.TagHelpers`.
* `Microsoft.AspNetCore.Mvc.Razor`.

## <a name="application-feature-providers"></a>应用程序功能提供程序

应用程序功能提供程序用于检查应用程序部件，并为这些部件提供功能。 以下 ASP.NET Core 功能有内置功能提供程序：

* [Controllers](/dotnet/api/microsoft.aspnetcore.mvc.controllers.controllerfeatureprovider)
* [标记帮助程序](/dotnet/api/microsoft.aspnetcore.mvc.razor.taghelpers.taghelperfeatureprovider)
* [查看组件](/dotnet/api/microsoft.aspnetcore.mvc.viewcomponents.viewcomponentfeatureprovider)

功能提供程序从 <xref:Microsoft.AspNetCore.Mvc.ApplicationParts.IApplicationFeatureProvider`1> 继承，其中 `T` 是功能的类型。 可以为上面列出的任意功能类型实现功能提供程序。 `ApplicationPartManager.FeatureProviders` 中的功能提供程序的顺序可能影响运行时行为。 较晚添加的提供程序可能会影响较早添加的提供程序执行的操作。

### <a name="display-available-features"></a>显示可用功能

通过[依存关系注入](../../fundamentals/dependency-injection.md)请求 `ApplicationPartManager` 即可以枚举应用的可用功能：

[!code-csharp[](./app-parts/sample2/AppPartsSample/Controllers/FeaturesController.cs?highlight=16,25-27)]

[下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/app-parts/sample2)使用前面的代码显示应用功能：

```text
Controllers:
    - FeaturesController
    - HomeController
    - HelloController
    - GenericController`1
    - GenericController`1
Tag Helpers:
    - PrerenderTagHelper
    - AnchorTagHelper
    - CacheTagHelper
    - DistributedCacheTagHelper
    - EnvironmentTagHelper
    - Additional Tag Helpers omitted for brevity.
View Components:
    - SampleViewComponent
```

## <a name="discovery-in-application-parts"></a>应用程序部件中的发现

使用应用程序部件进行开发时，会遇到 HTTP 404 错误且并不鲜见。 发生这些错误的原因通常是由于未满足某项针对应用程序部件发现方式的基本要求。 如果应用返回 HTTP 404 错误，请验证是否满足以下要求：

* 需要将 `applicationName` 设置设置为用于发现的根程序集。 用于发现的根程序集通常是入口点程序集。
* 根程序集需要引用用于发现的部件。 引用可以是直接的，也可以是可传递的。
* 根程序集需要引用 Web SDK。
  * ASP.NET Core 框架具有自定义生成逻辑，会将属性标记到用于发现的根程序集中。

::: moniker-end
