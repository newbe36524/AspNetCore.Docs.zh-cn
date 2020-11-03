---
title: ASP.NET Core 中基于资源的授权
author: scottaddie
description: 了解如何在 ASP.NET Core 应用中实现基于资源的授权。
ms.author: scaddie
ms.custom: mvc
ms.date: 11/15/2018
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
uid: security/authorization/resourcebased
ms.openlocfilehash: 202ca681a66ddf7f729d8835e2f77da846583df1
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060165"
---
# <a name="resource-based-authorization-in-aspnet-core"></a>ASP.NET Core 中基于资源的授权

授权策略取决于要访问的资源。 假设有一个具有 author 属性的文档。 仅允许作者更新文档。 因此，在进行授权评估之前，必须从数据存储中检索文档。

在数据绑定之前和在执行加载文档的页面处理程序或操作之前，会发生属性评估。 由于这些原因，具有属性的声明性授权 `[Authorize]` 无法满足要求。 相反，你可以调用自定义授权方法， &mdash; 这种方法称为 *命令式 authorization* 。

::: moniker range=">= aspnetcore-3.0"
[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples/3_0)（[如何下载](xref:index#how-to-download-a-sample)）。
::: moniker-end

 ::: moniker range=">= aspnetcore-2.0 < aspnetcore-3.0"
[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples/2_2)（[如何下载](xref:index#how-to-download-a-sample)）。
::: moniker-end

::: moniker range="<= aspnetcore-1.1"
[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/resourcebased/samples/1_1)（[如何下载](xref:index#how-to-download-a-sample)）。
::: moniker-end

使用[由授权保护的用户数据创建 ASP.NET Core 应用](xref:security/authorization/secure-data)包含使用基于资源的授权的示例应用。

## <a name="use-imperative-authorization"></a>使用命令性授权

授权作为 [IAuthorizationService](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizationservice) 服务实现，并在类中的服务集合中进行注册 `Startup` 。 通过 [依赖关系注入](xref:fundamentals/dependency-injection) 到页面处理程序或操作使该服务可用。

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Controllers/DocumentController.cs?name=snippet_IAuthServiceDI&highlight=6)]

`IAuthorizationService` 具有两个 `AuthorizeAsync` 方法重载：一个接受资源和策略名称，另一个接受资源并提供要评估的要求的列表。

::: moniker range=">= aspnetcore-2.0"

```csharp
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<AuthorizationResult> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

```csharp
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          IEnumerable<IAuthorizationRequirement> requirements);
Task<bool> AuthorizeAsync(ClaimsPrincipal user,
                          object resource,
                          string policyName);
```

::: moniker-end

<a name="security-authorization-resource-based-imperative"></a>

在下面的示例中，要保护的资源将加载到自定义 `Document` 对象中。 `AuthorizeAsync`调用重载来确定是否允许当前用户编辑提供的文档。 将自定义 "EditPolicy" 授权策略分解为决定。 有关创建授权策略的详细信息，请参阅 [基于策略的自定义授权](xref:security/authorization/policies) 。

> [!NOTE]
> 下面的代码示例假定已运行身份验证并设置了 `User` 属性。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Pages/Document/Edit.cshtml.cs?name=snippet_DocumentEditHandler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentEditAction)]

::: moniker-end

## <a name="write-a-resource-based-handler"></a>编写基于资源的处理程序

为基于资源的授权编写处理程序与 [编写简单的要求处理程序并无](xref:security/authorization/policies#security-authorization-policies-based-authorization-handler)差别。 创建自定义要求类，并实现需求处理程序类。 有关创建要求类的详细信息，请参阅 [要求](xref:security/authorization/policies#requirements)。

处理程序类同时指定要求和资源类型。 例如，使用和资源的处理程序 `SameAuthorRequirement` `Document` 如下所示：

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Services/DocumentAuthorizationHandler.cs?name=snippet_HandlerAndRequirement)]

::: moniker-end

在前面的示例中，假设 `SameAuthorRequirement` 是更为通用的类的特殊情况 `SpecificAuthorRequirement` 。 `SpecificAuthorRequirement`类 (未显示) 包含 `Name` 表示作者姓名的属性。 `Name`属性可以设置为当前用户。

在以下项中注册要求和处理程序 `Startup.ConfigureServices` ：

::: moniker range=">= aspnetcore-3.0"
[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=4-8,10)]
::: moniker-end

 ::: moniker range=">= aspnetcore-2.0 < aspnetcore-3.0"
[!code-csharp[](resourcebased/samples/2_2/ResourceBasedAuthApp2/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]
::: moniker-end

::: moniker range="<= aspnetcore-1.1"
[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Startup.cs?name=snippet_ConfigureServicesSample&highlight=3-7,9)]
::: moniker-end

### <a name="operational-requirements"></a>操作要求

如果要根据 CRUD (创建、读取、更新、删除) 操作的结果做出决策，请使用 [OperationAuthorizationRequirement](/dotnet/api/microsoft.aspnetcore.authorization.infrastructure.operationauthorizationrequirement) 帮助器类。 此类使你能够为每个操作类型编写单一处理程序而不是单个类。 若要使用它，请提供一些操作名称：

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_OperationsClass)]

处理程序的实现方式如下 `OperationAuthorizationRequirement` `Document` ：

 ::: moniker range=">= aspnetcore-2.0"
[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Services/DocumentAuthorizationCrudHandler.cs?name=snippet_Handler)]

::: moniker-end

前面的处理程序使用资源、用户的标识和要求的属性验证操作 `Name` 。

## <a name="challenge-and-forbid-with-an-operational-resource-handler"></a>操作资源处理程序的挑战和禁止

本部分说明如何处理质询和禁止操作结果，以及质询和禁止的不同之处。

若要调用操作资源处理程序，请 `AuthorizeAsync` 在页面处理程序或操作中调用时指定操作。 下面的示例确定是否允许经过身份验证的用户查看所提供的文档。

> [!NOTE]
> 下面的代码示例假定已运行身份验证并设置了 `User` 属性。

::: moniker range=">= aspnetcore-2.0"

[!code-csharp[](resourcebased/samples/3_0/ResourceBasedAuthApp2/Pages/Document/View.cshtml.cs?name=snippet_DocumentViewHandler&highlight=10-11)]

如果授权成功，则返回用于查看文档的页面。 如果授权失败，但用户已进行身份验证，则返回 `ForbidResult` 通知任何身份验证中间件验证失败。 `ChallengeResult`当必须执行身份验证时，将返回。 对于交互式浏览器客户端，可能需要将用户重定向到登录页。

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](resourcebased/samples/1_1/ResourceBasedAuthApp1/Controllers/DocumentController.cs?name=snippet_DocumentViewAction&highlight=11-12)]

如果授权成功，则返回文档的视图。 如果授权失败，则返回 `ChallengeResult` 通知任何身份验证中间件身份验证失败，中间件可以采取相应的响应。 适当的响应可能返回401或403状态代码。 对于交互式浏览器客户端，这可能意味着将用户重定向到登录页。

::: moniker-end
