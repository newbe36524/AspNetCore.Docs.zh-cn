---
title: 使用 ASP.NET Core 中的应用程序模型
author: ardalis
description: 了解如何读取和控制应用程序模型，从而修改 MVC 元素在 ASP.NET Core 中的行为。
ms.author: riande
ms.date: 12/05/2019
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
uid: mvc/controllers/application-model
ms.openlocfilehash: a7a654eb43c0dbf375af911d8d5353ac4c04a825
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060932"
---
# <a name="work-with-the-application-model-in-aspnet-core"></a>使用 ASP.NET Core 中的应用程序模型

作者：[Steve Smith](https://ardalis.com/)

ASP.NET Core MVC 会定义一个 *应用程序模型* ，用于表示 MVC 应用的各个组件。 通过读取和处理此模型可修改 MVC 元素的行为方式。 默认情况下，MVC 遵循特定的约定，以确定将哪些类视作控制器，这些类上的哪些方法是操作，以及参数和路由的行为方式。 你可以自定义此行为以满足应用的需要，方法如下：创建自己的约定，并将它们应用于全局或作为属性应用。

## <a name="models-and-providers"></a>模型和提供程序

ASP.NET Core MVC 应用程序模型包括用于描述 MVC 应用程序的抽象接口和具体实现类。 此模型是 MVC 根据默认约定发现应用的控制器、操作、操作参数、路由和筛选器的结果。 通过使用应用程序模型，可以修改应用以遵循与默认 MVC 行为不同的约定。 参数、名称、路由和筛选器都用作操作和控制器的配置数据。

ASP.NET Core MVC 应用程序模型具有以下结构：

* ApplicationModel
  * 控制器 (ControllerModel)
    * 操作 (ActionModel)
      * 参数 (ParameterModel)

该模型的每个级别都有权访问公用 `Properties` 集合，层次结构中的较低级别可以访问和覆盖由较高级别设置的属性值。 创建操作时，属性保存到 `ActionDescriptor.Properties` 中。 之后，当处理请求时，可通过 `ActionContext.ActionDescriptor.Properties` 访问某个约定添加或修改的任何属性。 若要基于每项操作对筛选器、模型绑定器等进行配置，使用属性不失为一个好办法。

> [!NOTE]
> 一旦完成应用启动，`ActionDescriptor.Properties` 集合就不再是线程安全的（针对写入）。 约定是将数据安全添加到此集合的最佳方式。

### <a name="iapplicationmodelprovider"></a>IApplicationModelProvider

ASP.NET Core MVC 使用提供程序模式（由 [IApplicationModelProvider](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelprovider) 接口定义）加载应用程序模型。 此部分介绍此提供程序的工作原理的一些内部实现细节。 这是一项高级主题 — 利用应用程序模型的大多数应用应使用约定来执行此操作。

`IApplicationModelProvider` 接口的实现相互“包装”，每个实现都基于其 `Order` 属性以升序调用 `OnProvidersExecuting`。 然后，按相反的顺序调用 `OnProvidersExecuted` 方法。 该框架定义了多个提供程序：

首先 (`Order=-1000`)：

* [`DefaultApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.defaultapplicationmodelprovider)

然后 (`Order=-990`)：

* [`AuthorizationApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.internal.authorizationapplicationmodelprovider)
* [`CorsApplicationModelProvider`](/dotnet/api/microsoft.aspnetcore.mvc.cors.internal.corsapplicationmodelprovider)

> [!NOTE]
> 未定义具有相同 `Order` 值的两个提供程序的调用顺序，因此不应依赖此顺序。

> [!NOTE]
> `IApplicationModelProvider` 是一种高级概念，框架创建者可对其进行扩展。 一般情况下，应用应使用约定，而框架应使用提供程序。 主要不同之处在于提供程序始终先于约定运行。

`DefaultApplicationModelProvider` 建立了由 ASP.NET Core MVC 使用的许多默认行为。 其职责包括：

* 将全局筛选器添加到上下文
* 将控制器添加到上下文
* 将公共控制器方法作为操作添加
* 将操作方法参数添加到上下文
* 应用路由和其他属性

某些内置行为由 `DefaultApplicationModelProvider` 实现。 此提供程序负责构造 [`ControllerModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.controllermodel) ，后者又引用 [`ActionModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.actionmodel) 、 [`PropertyModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.propertymodel) 和 [`ParameterModel`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.parametermodel) 实例。 `DefaultApplicationModelProvider` 类是一个内部框架实现细节，未来将对其进行更改。 

`AuthorizationApplicationModelProvider` 负责应用与 `AuthorizeFilter` 和 `AllowAnonymousFilter` 属性关联的行为。 [详细了解这些属性](xref:security/authorization/simple)。

`CorsApplicationModelProvider` 可实现与 `IEnableCorsAttribute`、`IDisableCorsAttribute` 和 `DisableCorsAuthorizationFilter` 关联的行为。 [详细了解 CORS](xref:security/cors)。

## <a name="conventions"></a>约定

应用程序模型定义了约定抽象，通过约定抽象来自定义模型行为比重写整个模型或提供程序更简单。 建议使用这些抽象来修改应用的行为。 通过使用约定，可以编写能动态应用自定义项的代码。 使用[筛选器](xref:mvc/controllers/filters)可修改框架的行为，而利用自定义项可控制整个应用协同工作的方式。

可用约定如下：

* [`IApplicationModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iapplicationmodelconvention)
* [`IControllerModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.icontrollermodelconvention)
* [`IActionModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iactionmodelconvention)
* [`IParameterModelConvention`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.iparametermodelconvention)

约定的应用方法是将它们添加到 MVC 选项，或通过实现 `Attribute` 并将其应用于控制器、操作或操作参数 (类似于 [`Filters`](xref:mvc/controllers/filters)) 。 与筛选器不同的是，约定仅在应用启动时执行，而不作为每个请求的一部分执行。

### <a name="sample-modifying-the-applicationmodel"></a>示例：修改 ApplicationModel

以下约定用于向应用程序模型添加属性。 

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ApplicationDescription.cs)]

当在 `Startup` 的 `ConfigureServices` 中添加 MVC 时，应用程序模型约定作为选项应用。

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=5)]

可从控制器操作内的 `ActionDescriptor` 属性集合中访问属性：

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/AppModelController.cs?name=AppModelController)]

### <a name="sample-modifying-the-controllermodel-description"></a>示例：修改 ControllerModel 说明

与上一个示例一样，也可以修改控制器模型，以包含自定义属性。 这些属性将覆盖应用程序模型中指定的具有相同名称的现有属性。 以下约定属性可在控制器级别添加说明：

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ControllerDescriptionAttribute.cs)]

此约定在控制器上作为属性应用。

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=ControllerDescription&highlight=1)]

访问“description”属性的方式与前面示例中一样。

### <a name="sample-modifying-the-actionmodel-description"></a>示例：修改 ActionModel 说明

可向各项操作应用不同的属性约定，并覆盖已在应用程序或控制器级别应用的行为。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/ActionDescriptionAttribute.cs)]

通过将此约定应用于上一示例的控制器中的某项操作，演示了它如何覆盖控制器级别的约定：

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/DescriptionAttributesController.cs?name=DescriptionAttributesController&highlight=9)]

### <a name="sample-modifying-the-parametermodel"></a>示例：修改 ParameterModel

可将以下约定应用于操作参数，以修改其 `BindingInfo`。 以下约定要求参数为路由参数；忽略其他可能的绑定源（比如查询字符串值）。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/MustBeInRouteParameterModelConvention.cs)]

该属性可应用于任何操作参数：

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/ParameterModelController.cs?name=ParameterModelController&highlight=5)]

### <a name="sample-modifying-the-actionmodel-name"></a>示例：修改 ActionModel 名称

以下约定可修改 `ActionModel`，以更新其应用到的操作的 *名称* 。 新名称以参数形式提供给该属性。 此新名称供路由使用，因此它将影响用于访问此操作方法的路由。

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/CustomActionNameAttribute.cs)]

此属性应用于 `HomeController` 中的操作方法：

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/HomeController.cs?name=ActionModelConvention&highlight=2)]

即使方法名称为 `SomeName`，该属性也会覆盖 MVC 使用该方法名称的约定，并将操作名称替换为 `MyCoolAction`。 因此，用于访问此操作的路由为 `/Home/MyCoolAction`。

> [!NOTE]
> 此示例本质上与使用内置 [ActionName](/dotnet/api/microsoft.aspnetcore.mvc.actionnameattribute) 属性相同。

### <a name="sample-custom-routing-convention"></a>示例：自定义路由约定

可以使用 `IApplicationModelConvention` 来自定义路由的工作方式。 例如，以下约定会将控制器的命名空间合并到其路由中，并将命名空间中的 `.` 替换为路由中的 `/`：

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/NamespaceRoutingConvention.cs)]

该约定作为一个选项添加到 Startup 中。

[!code-csharp[](./application-model/sample/src/AppModelSample/Startup.cs?name=ConfigureServices&highlight=6)]

> [!TIP]
> 可以使用 `services.Configure<MvcOptions>(c => c.Conventions.Add(YOURCONVENTION));` 来访问 `MvcOptions`，以将约定添加到[中间件](xref:fundamentals/middleware/index)

此示例将此约定应用于未使用属性路由的路由，其中，控制器名称包含“Namespace”。 以下控制器演示了此约定：

[!code-csharp[](./application-model/sample/src/AppModelSample/Controllers/NamespaceRoutingController.cs?highlight=7-8)]

## <a name="application-model-usage-in-webapicompatshim"></a>应用程序模型在 WebApiCompatShim 中的使用

ASP.NET Core MVC 使用一组不同于 ASP.NET Web API 2 的约定。 使用自定义约定，可以修改 ASP.NET Core MVC 应用的行为，使其与 Web API 应用保持一致。 Microsoft 附带了专用于此的 [WebApiCompatShim](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.WebApiCompatShim/)。

> [!NOTE]
> 详细了解[从 ASP.NET Web API 迁移](xref:migration/webapi)。

若要使用 Web API Compatibility Shim，需将该包添加到项目中，然后通过调用 `Startup` 中的 `AddWebApiConventions`，将约定添加到 MVC：

```csharp
services.AddMvc().AddWebApiConventions();
```

该填充程序提供的约定仅适用于应用中已应用特定属性的部分。 以下四个属性用于控制哪些控制器应使用该填充程序的约定来修改自己的约定：

* [UseWebApiActionConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiactionconventionsattribute)
* [UseWebApiOverloadingAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapioverloadingattribute)
* [UseWebApiParameterConventionsAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiparameterconventionsattribute)
* [UseWebApiRoutesAttribute](/dotnet/api/microsoft.aspnetcore.mvc.webapicompatshim.usewebapiroutesattribute)

### <a name="action-conventions"></a>操作约定

`UseWebApiActionConventionsAttribute` 用于根据名称将 HTTP 方法映射到操作（例如，`Get` 将映射到 `HttpGet`）。 它仅适用于不使用属性路由的操作。

### <a name="overloading"></a>重载

`UseWebApiOverloadingAttribute` 用于应用 `WebApiOverloadingApplicationModelConvention` 约定。 此约定可向操作选择过程添加 `OverloadActionConstraint`，以将候选操作限制为其请求满足所有非可选参数的操作。

### <a name="parameter-conventions"></a>参数约定

`UseWebApiParameterConventionsAttribute` 用于应用 `WebApiParameterConventionsApplicationModelConvention` 操作约定。 此约定指定用作操作参数的简单类型默认来自 URI，而复杂类型来自请求正文。

### <a name="routes"></a>路由

`UseWebApiRoutesAttribute` 控制是否应用 `WebApiApplicationModelConvention` 控制器约定。 启用后，此约定用于向路由添加对[区域](xref:mvc/controllers/areas)的支持。

除了一组约定外，该兼容性包还包含一个 `System.Web.Http.ApiController` 基类，用于替换 Web API 提供的等效项。 这允许针对 Web API 编写并且继承自 `ApiController` 的控制器在 ASP.NET Core MVC 上运行时，能够按照设计的方式运行。 前面列出的所有 `UseWebApi*` 属性将应用于基本控制器类。 `ApiController` 公开了与在 Web API 中找到的属性、方法和结果类型兼容的属性、方法和结果类型。

## <a name="using-apiexplorer-to-document-your-app"></a>使用 ApiExplorer 记录应用

应用程序模型 [`ApiExplorer`](/dotnet/api/microsoft.aspnetcore.mvc.applicationmodels.apiexplorermodel) 在每个级别公开一个属性，该属性可用于遍历应用的结构。 这可用于[使用 Swagger 等工具为 Web API 生成帮助页](xref:tutorials/web-api-help-pages-using-swagger)。 `ApiExplorer` 属性公开了 `IsVisible` 属性，后者可设置为指定应公开的应用模型部分。 可以使用约定配置此设置：

[!code-csharp[](./application-model/sample/src/AppModelSample/Conventions/EnableApiExplorerApplicationConvention.cs)]

使用此方法（和附加约定，如有需要），可以在应用中的任何级别启用或禁用 API 可见性。 
