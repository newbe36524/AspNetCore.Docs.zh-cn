---
title: ASP.NET Core 中基于策略的授权
author: rick-anderson
description: 了解如何创建和使用授权策略处理程序，以在 ASP.NET Core 的应用程序中强制实施授权要求。
ms.author: riande
ms.custom: mvc
ms.date: 04/15/2020
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
uid: security/authorization/policies
ms.openlocfilehash: 286dc3bcc66b86a2a6b7d3cb7b6052bf7b474aff
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060204"
---
# <a name="policy-based-authorization-in-aspnet-core"></a>ASP.NET Core 中基于策略的授权

::: moniker range=">= aspnetcore-3.0"

[基于角色的授权](xref:security/authorization/roles)和[基于声明的授权](xref:security/authorization/claims)，都使用了要求、要求处理程序和预配置的策略。 这些构建基块支持代码中授权评估的表达式。 结果是一种更丰富的可重复使用的授权结构。

授权策略包括一个或多个要求。 它在授权服务配置的一部分中注册， `Startup.ConfigureServices` 方法如下：

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=31-32,39-40,42-45, 53, 58)]

在前面的示例中，创建了 "AtLeast21" 策略。 它有一个 &mdash; 最小期限的要求，它作为要求的参数提供。

## <a name="iauthorizationservice"></a>IAuthorizationService

确定授权是否成功的主要服务是 <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> ：

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

前面的代码突出显示了 [IAuthorizationService](https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs)的两种方法。

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement> 是无方法的标记服务，以及用于跟踪授权是否成功的机制。

每个 <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> 负责检查是否满足要求：
<!--The following code is a copy/paste from 
https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

此 <xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> 类是处理程序用来标记是否已满足要求的类：

```csharp
 context.Succeed(requirement)
```

以下代码显示了简化的 (，并注释了授权服务的默认实现) 注释：

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

下面的代码演示了一个典型的 `ConfigureServices` ：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddControllersWithViews();
    services.AddRazorPages();
}
```

使用 <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> 或 `[Authorize(Policy = "Something")]` 进行授权。

## <a name="apply-policies-to-mvc-controllers"></a>将策略应用到 MVC 控制器

如果使用的是 Razor 页面，请参阅本文档中的 [将策略应用于 Razor 页面](#apply-policies-to-razor-pages) 。

策略通过使用 `[Authorize]` 具有策略名称的属性应用到控制器。 例如： 。

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="apply-policies-to-no-locrazor-pages"></a>将策略应用到 Razor 页面

策略是 Razor 通过使用具有策略名称的属性应用于页面的 `[Authorize]` 。 例如： 。

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

策略可以在 **not** Razor 页面处理程序级别应用 * not *，它们必须应用于页面。

可以 Razor 通过使用 [授权约定](xref:security/authorization/razor-pages-authorization)将策略应用于页面。

## <a name="requirements"></a>要求

授权要求是一个数据参数集合，策略可以使用这些参数来评估当前用户主体。 在我们的 "AtLeast21" 策略中，要求是 &mdash; 最小年龄的单个参数。 要求实现 [IAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement)，它是一个空的标记接口。 可以按如下所示实现参数化最低期限要求：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

如果授权策略包含多个授权要求，则所有要求必须通过，才能成功进行策略评估。 换句话说，添加到单个授权策略中的多个授权要求将按 _ *和* * 来处理。

> [!NOTE]
> 要求不需要具有数据或属性。

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>授权处理程序

授权处理程序负责计算要求的属性。 授权处理程序根据提供的 [AuthorizationHandlerContext](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) 评估要求，以确定是否允许访问。

要求可以有 [多个处理程序](#security-authorization-policies-based-multiple-handlers)。 处理程序可能会[继承 \<TRequirement> AuthorizationHandler](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1)，其中 `TRequirement` 是要处理的要求。 或者，处理程序可以实现 [IAuthorizationHandler](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) 来处理多种类型的要求。

### <a name="use-a-handler-for-one-requirement"></a>为一个要求使用处理程序

<a name="security-authorization-handler-example"></a>

下面是一种一对一关系的示例，其中最小 age 处理程序利用了单一要求：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

上述代码确定当前用户主体是否具有由已知的可信颁发者颁发的出生日期。 当缺少声明时无法进行授权，在这种情况下，将返回已完成的任务。 当声明存在时，将计算用户的年龄。 如果用户达到了要求所定义的最小时间，则会认为授权成功。 授权成功后， `context.Succeed` 将通过满足要求作为其唯一参数调用。

### <a name="use-a-handler-for-multiple-requirements"></a>为多个要求使用处理程序

下面是一个一对多关系的示例，其中权限处理程序可以处理三种不同类型的要求：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

前面的代码遍历[PendingRequirements](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements) &mdash; 属性，该属性包含未标记为成功的要求。 对于 `ReadPermission` 要求，用户必须是所有者或主办方才能访问请求的资源。 对于 `EditPermission` 或 `DeletePermission` 要求，该用户必须是访问所请求资源的所有者。

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>处理程序注册

在配置过程中，将在服务集合中注册处理程序。 例如： 。

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=31-32,39-40,42-45, 53-55, 58)]

前面的代码 `MinimumAgeHandler` 通过调用注册为单一实例 `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();` 。 可以使用任何内置 [服务生存期](xref:fundamentals/dependency-injection#service-lifetimes)来注册处理程序。

## <a name="what-should-a-handler-return"></a>处理程序应返回什么？

请注意， `Handle` [处理程序示例](#security-authorization-handler-example) 中的方法不返回值。 表示成功或失败的状态如何？

* 处理程序通过调用来指示成功 `context.Succeed(IAuthorizationRequirement requirement)` ，同时传递已成功验证的要求。

* 处理程序通常不需要处理故障，因为同一要求的其他处理程序可能会成功。

* 若要保证故障，即使其他要求处理程序成功，请调用 `context.Fail` 。

如果处理程序调用 `context.Succeed` 或 `context.Fail` ，则仍将调用所有其他处理程序。 这允许要求产生副作用，如日志记录，即使另一个处理程序已成功验证或失败，也会发生这种情况。 当设置为时 `false` ， [InvokeHandlersAfterFailure](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) 属性将在调用时将处理程序的执行短路 `context.Fail` 。 `InvokeHandlersAfterFailure` 默认为 `true` ，在这种情况下，将调用所有处理程序。

> [!NOTE]
> 即使身份验证失败，也会调用授权处理程序。

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>为什么需要多个处理程序才能实现要求？

如果希望计算基于 **或** ，请为单个要求实现多个处理程序。 例如，Microsoft 的门只有用密钥卡打开。 如果在家中保留了密钥卡，则接待员会打印一个临时贴纸，并为您打开门。 在这种情况下，你将有一个要求 *BuildingEntry* ，但有多个处理程序，每个处理程序都检查单个需求。

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

确保两个处理程序都 [已注册](xref:security/authorization/policies#security-authorization-policies-based-handler-registration)。 如果某个处理程序在某一策略评估后成功 `BuildingEntryRequirement` ，则策略评估将成功。

## <a name="use-a-func-to-fulfill-a-policy"></a>使用 func 来实现策略

在某些情况下，实现策略的情况非常简单，只是在代码中表达。 `Func<AuthorizationHandlerContext, bool>`使用策略生成器配置策略时，可以提供 `RequireAssertion` 。

例如， `BadgeEntryHandler` 可以按如下所示重写前面的：

[!code-csharp[](policies/samples/3.0PoliciesAuthApp1/Startup.cs?range=42-43,47-53)]

## <a name="access-mvc-request-context-in-handlers"></a>访问处理程序中的 MVC 请求上下文

`HandleRequirementAsync`在授权处理程序中实现的方法具有两个参数： `AuthorizationHandlerContext` 和 `TRequirement` 正在处理的。 诸如 MVC 或的框架可 SignalR 自由地将任何对象添加到 `Resource` 上的属性 `AuthorizationHandlerContext` ，以传递附加信息。

使用终结点路由时，授权通常由授权中间件进行处理。 在这种情况下， `Resource` 属性为的实例 <xref:Microsoft.AspNetCore.Http.Endpoint> 。 终结点可用于探测要路由到的基础资源。 例如： 。

```csharp
if (context.Resource is Endpoint endpoint)
{
   var actionDescriptor = endpoint.Metadata.GetMetadata<ControllerActionDescriptor>();
   ...
}
```

终结点不提供对当前的访问 `HttpContext` 。 使用终结点路由时，使用在 `IHttpContextAcessor` `HttpContext` 授权处理程序内进行访问。 有关详细信息，请参阅 [使用来自自定义组件的 HttpContext](xref:fundamentals/httpcontext#use-httpcontext-from-custom-components)。

对于传统路由，或在 MVC 的授权筛选器中进行授权时，的值 `Resource` 为 <xref:Microsoft.AspNetCore.Mvc.Filters.AuthorizationFilterContext> 实例。 此属性提供对 `HttpContext` 、以及 `RouteData` MVC 和页面提供的其他内容的访问 Razor 。

使用 `Resource` 属性是特定于框架的。 使用属性中的信息会将 `Resource` 授权策略限制为特定框架。 应 `Resource` 使用关键字强制转换属性 `is` ，然后确认强制转换已成功，以确保在 `InvalidCastException` 其他框架上运行时代码不会崩溃：

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```

## <a name="globally-require-all-users-to-be-authenticated"></a>全局要求对所有用户进行身份验证

[!INCLUDE[](~/includes/requireAuth.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[基于角色的授权](xref:security/authorization/roles)和[基于声明的授权](xref:security/authorization/claims)，都使用了要求、要求处理程序和预配置的策略。 这些构建基块支持代码中授权评估的表达式。 结果是一种更丰富的可重复使用的授权结构。

授权策略包括一个或多个要求。 它在授权服务配置的一部分中注册， `Startup.ConfigureServices` 方法如下：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,66)]

在前面的示例中，创建了 "AtLeast21" 策略。 它有一个 &mdash; 最小期限的要求，它作为要求的参数提供。

## <a name="iauthorizationservice"></a>IAuthorizationService 

确定授权是否成功的主要服务是 <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> ：

[!code-csharp[](policies/samples/stubs/copy_of_IAuthorizationService.cs?highlight=24-25,48-49&name=snippet)]

[!INCLUDE[request localized comments](~/includes/code-comments-loc.md)]

前面的代码突出显示了 [IAuthorizationService](https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationService.cs)的两种方法。

<xref:Microsoft.AspNetCore.Authorization.IAuthorizationRequirement> 是无方法的标记服务，以及用于跟踪授权是否成功的机制。

每个 <xref:Microsoft.AspNetCore.Authorization.IAuthorizationHandler> 负责检查是否满足要求：
<!--The following code is a copy/paste from 
https://github.com/dotnet/AspNetCore/blob/v2.2.4/src/Security/Authorization/Core/src/IAuthorizationHandler.cs -->

```csharp
/// <summary>
/// Classes implementing this interface are able to make a decision if authorization
/// is allowed.
/// </summary>
public interface IAuthorizationHandler
{
    /// <summary>
    /// Makes a decision if authorization is allowed.
    /// </summary>
    /// <param name="context">The authorization information.</param>
    Task HandleAsync(AuthorizationHandlerContext context);
}
```

此 <xref:Microsoft.AspNetCore.Authorization.AuthorizationHandlerContext> 类是处理程序用来标记是否已满足要求的类：

```csharp
 context.Succeed(requirement)
```

以下代码显示了简化的 (，并注释了授权服务的默认实现) 注释：

```csharp
public async Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user, 
             object resource, IEnumerable<IAuthorizationRequirement> requirements)
{
    // Create a tracking context from the authorization inputs.
    var authContext = _contextFactory.CreateContext(requirements, user, resource);

    // By default this returns an IEnumerable<IAuthorizationHandlers> from DI.
    var handlers = await _handlers.GetHandlersAsync(authContext);

    // Invoke all handlers.
    foreach (var handler in handlers)
    {
        await handler.HandleAsync(authContext);
    }

    // Check the context, by default success is when all requirements have been met.
    return _evaluator.Evaluate(authContext);
}
```

下面的代码演示了一个典型的 `ConfigureServices` ：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add all of your handlers to DI.
    services.AddSingleton<IAuthorizationHandler, MyHandler1>();
    // MyHandler2, ...

    services.AddSingleton<IAuthorizationHandler, MyHandlerN>();

    // Configure your policies
    services.AddAuthorization(options =>
          options.AddPolicy("Something",
          policy => policy.RequireClaim("Permission", "CanViewPage", "CanViewAnything")));


    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
```

使用 <xref:Microsoft.AspNetCore.Authorization.IAuthorizationService> 或 `[Authorize(Policy = "Something")]` 进行授权。

## <a name="apply-policies-to-mvc-controllers"></a>将策略应用到 MVC 控制器

如果使用的是 Razor 页面，请参阅本文档中的 [将策略应用于 Razor 页面](#apply-policies-to-razor-pages) 。

策略通过使用 `[Authorize]` 具有策略名称的属性应用到控制器。 例如： 。

[!code-csharp[](policies/samples/PoliciesAuthApp1/Controllers/AlcoholPurchaseController.cs?name=snippet_AlcoholPurchaseControllerClass&highlight=4)]

## <a name="apply-policies-to-no-locrazor-pages"></a>将策略应用到 Razor 页面

策略是 Razor 通过使用具有策略名称的属性应用于页面的 `[Authorize]` 。 例如： 。

[!code-csharp[](policies/samples/PoliciesAuthApp2/Pages/AlcoholPurchase.cshtml.cs?name=snippet_AlcoholPurchaseModelClass&highlight=4)]

还可以 Razor 通过使用 [授权约定](xref:security/authorization/razor-pages-authorization)，将策略应用到页面。

## <a name="requirements"></a>要求

授权要求是一个数据参数集合，策略可以使用这些参数来评估当前用户主体。 在我们的 "AtLeast21" 策略中，要求是 &mdash; 最小年龄的单个参数。 要求实现 [IAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationrequirement)，它是一个空的标记接口。 可以按如下所示实现参数化最低期限要求：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/MinimumAgeRequirement.cs?name=snippet_MinimumAgeRequirementClass)]

如果授权策略包含多个授权要求，则所有要求必须通过，才能成功进行策略评估。 换句话说，添加到单个授权策略中的多个授权要求 **将分别处理** 。

> [!NOTE]
> 要求不需要具有数据或属性。

<a name="security-authorization-policies-based-authorization-handler"></a>

## <a name="authorization-handlers"></a>授权处理程序

授权处理程序负责计算要求的属性。 授权处理程序根据提供的 [AuthorizationHandlerContext](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext) 评估要求，以确定是否允许访问。

要求可以有 [多个处理程序](#security-authorization-policies-based-multiple-handlers)。 处理程序可能会[继承 \<TRequirement> AuthorizationHandler](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandler-1)，其中 `TRequirement` 是要处理的要求。 或者，处理程序可以实现 [IAuthorizationHandler](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationhandler) 来处理多种类型的要求。

### <a name="use-a-handler-for-one-requirement"></a>为一个要求使用处理程序

<a name="security-authorization-handler-example"></a>

下面是一种一对一关系的示例，其中最小 age 处理程序利用了单一要求：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/MinimumAgeHandler.cs?name=snippet_MinimumAgeHandlerClass)]

上述代码确定当前用户主体是否具有由已知的可信颁发者颁发的出生日期。 当缺少声明时无法进行授权，在这种情况下，将返回已完成的任务。 当声明存在时，将计算用户的年龄。 如果用户达到了要求所定义的最小时间，则会认为授权成功。 授权成功后， `context.Succeed` 将通过满足要求作为其唯一参数调用。

### <a name="use-a-handler-for-multiple-requirements"></a>为多个要求使用处理程序

下面是一个一对多关系的示例，其中权限处理程序可以处理三种不同类型的要求：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/PermissionHandler.cs?name=snippet_PermissionHandlerClass)]

前面的代码遍历[PendingRequirements](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.pendingrequirements#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_PendingRequirements) &mdash; 属性，该属性包含未标记为成功的要求。 对于 `ReadPermission` 要求，用户必须是所有者或主办方才能访问请求的资源。 对于 `EditPermission` 或 `DeletePermission` 要求，该用户必须是访问所请求资源的所有者。

<a name="security-authorization-policies-based-handler-registration"></a>

### <a name="handler-registration"></a>处理程序注册

在配置过程中，将在服务集合中注册处理程序。 例如： 。

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=32-33,48-53,61,62-63,66)]

前面的代码 `MinimumAgeHandler` 通过调用注册为单一实例 `services.AddSingleton<IAuthorizationHandler, MinimumAgeHandler>();` 。 可以使用任何内置 [服务生存期](xref:fundamentals/dependency-injection#service-lifetimes)来注册处理程序。

## <a name="what-should-a-handler-return"></a>处理程序应返回什么？

请注意， `Handle` [处理程序示例](#security-authorization-handler-example) 中的方法不返回值。 表示成功或失败的状态如何？

* 处理程序通过调用来指示成功 `context.Succeed(IAuthorizationRequirement requirement)` ，同时传递已成功验证的要求。

* 处理程序通常不需要处理故障，因为同一要求的其他处理程序可能会成功。

* 若要保证故障，即使其他要求处理程序成功，请调用 `context.Fail` 。

如果处理程序调用 `context.Succeed` 或 `context.Fail` ，则仍将调用所有其他处理程序。 这允许要求产生副作用，如日志记录，即使另一个处理程序已成功验证或失败，也会发生这种情况。 当设置为时 `false` ， [InvokeHandlersAfterFailure](/dotnet/api/microsoft.aspnetcore.authorization.authorizationoptions.invokehandlersafterfailure#Microsoft_AspNetCore_Authorization_AuthorizationOptions_InvokeHandlersAfterFailure) 属性将在调用时将处理程序的执行短路 `context.Fail` 。 `InvokeHandlersAfterFailure` 默认为 `true` ，在这种情况下，将调用所有处理程序。

> [!NOTE]
> 即使身份验证失败，也会调用授权处理程序。

<a name="security-authorization-policies-based-multiple-handlers"></a>

## <a name="why-would-i-want-multiple-handlers-for-a-requirement"></a>为什么需要多个处理程序才能实现要求？

如果希望计算基于 **或** ，请为单个要求实现多个处理程序。 例如，Microsoft 的门只有用密钥卡打开。 如果在家中保留了密钥卡，则接待员会打印一个临时贴纸，并为您打开门。 在这种情况下，你将有一个要求 *BuildingEntry* ，但有多个处理程序，每个处理程序都检查单个需求。

*BuildingEntryRequirement.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Requirements/BuildingEntryRequirement.cs?name=snippet_BuildingEntryRequirementClass)]

*BadgeEntryHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/BadgeEntryHandler.cs?name=snippet_BadgeEntryHandlerClass)]

*TemporaryStickerHandler.cs*

[!code-csharp[](policies/samples/PoliciesAuthApp1/Services/Handlers/TemporaryStickerHandler.cs?name=snippet_TemporaryStickerHandlerClass)]

确保两个处理程序都 [已注册](xref:security/authorization/policies#security-authorization-policies-based-handler-registration)。 如果某个处理程序在某一策略评估后成功 `BuildingEntryRequirement` ，则策略评估将成功。

## <a name="use-a-func-to-fulfill-a-policy"></a>使用 func 来实现策略

在某些情况下，实现策略的情况非常简单，只是在代码中表达。 `Func<AuthorizationHandlerContext, bool>`使用策略生成器配置策略时，可以提供 `RequireAssertion` 。

例如， `BadgeEntryHandler` 可以按如下所示重写前面的：

[!code-csharp[](policies/samples/PoliciesAuthApp1/Startup.cs?range=50-51,55-61)]

## <a name="access-mvc-request-context-in-handlers"></a>访问处理程序中的 MVC 请求上下文

`HandleRequirementAsync`在授权处理程序中实现的方法具有两个参数： `AuthorizationHandlerContext` 和 `TRequirement` 正在处理的。 诸如 MVC 或的框架可 SignalR 自由地将任何对象添加到 `Resource` 上的属性 `AuthorizationHandlerContext` ，以传递附加信息。

例如，MVC 在属性中传递 [AuthorizationFilterContext](/dotnet/api/?term=AuthorizationFilterContext) 的实例 `Resource` 。 此属性提供对 `HttpContext` 、以及 `RouteData` MVC 和页面提供的其他内容的访问 Razor 。

使用 `Resource` 属性是特定于框架的。 使用属性中的信息会将 `Resource` 授权策略限制为特定框架。 应 `Resource` 使用关键字强制转换属性 `is` ，然后确认强制转换已成功，以确保在 `InvalidCastException` 其他框架上运行时代码不会崩溃：

```csharp
// Requires the following import:
//     using Microsoft.AspNetCore.Mvc.Filters;
if (context.Resource is AuthorizationFilterContext mvcContext)
{
    // Examine MVC-specific things like routing data.
}
```

::: moniker-end
