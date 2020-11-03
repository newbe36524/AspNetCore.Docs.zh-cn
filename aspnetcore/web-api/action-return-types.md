---
title: ASP.NET Core Web API 中控制器操作的返回类型
author: scottaddie
description: 了解在 ASP.NET Core Web API 中使用各种控制器操作方法返回类型的相关信息。
ms.author: scaddie
ms.custom: mvc
ms.date: 02/03/2020
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
uid: web-api/action-return-types
ms.openlocfilehash: a2866970a20785ae8fa306d484972697817b7f92
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93058943"
---
# <a name="controller-action-return-types-in-aspnet-core-web-api"></a>ASP.NET Core Web API 中控制器操作的返回类型

作者：[Scott Addie](https://github.com/scottaddie)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/action-return-types/samples)（[如何下载](xref:index#how-to-download-a-sample)）

ASP.NET Core 提供以下 Web API 控制器操作返回类型选项：

::: moniker range=">= aspnetcore-2.1"

* [特定类型](#specific-type)
* [IActionResult](#iactionresult-type)
* [ActionResult\<T>](#actionresultt-type)

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

* [特定类型](#specific-type)
* [IActionResult](#iactionresult-type)

::: moniker-end

本文档说明每种返回类型的最佳适用情况。

## <a name="specific-type"></a>特定类型

最简单的操作返回基元或复杂数据类型（如 `string` 或自定义对象类型）。 请参考以下操作，该操作返回自定义 `Product` 对象的集合：

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_Get)]

在执行操作期间无需防范已知条件，返回特定类型即可满足要求。 上述操作不接受任何参数，因此不需要参数约束验证。

当有多个返回类型时，通常会将 <xref:Microsoft.AspNetCore.Mvc.ActionResult> 返回类型与基元或复杂返回类型混合使用。 需要[IActionResult](#iactionresult-type)或[ActionResult \<T> ](#actionresultt-type)才能容纳此类操作。 本文档中提供了多个返回类型的多个示例。

### <a name="return-ienumerablet-or-iasyncenumerablet"></a>返回 IEnumerable \<T> 或 IAsyncEnumerable\<T>

在 ASP.NET Core 2.2 及更低版本中，从操作返回 <xref:System.Collections.Generic.IEnumerable%601> 会导致序列化程序同步集合迭代。 因此会阻止调用，并且可能会导致线程池资源不足。 为了说明这一点，假设 Entity Framework (EF) Core 用于满足 Web API 的数据访问需求。 序列化期间，将同步枚举以下操作的返回类型：

```csharp
public IEnumerable<Product> GetOnSaleProducts() =>
    _context.Products.Where(p => p.IsOnSale);
```

若要避免在 ASP.NET Core 2.2 及更低版本的数据库上同步枚举和阻止等待，请调用 `ToListAsync`：

```csharp
public async Task<IEnumerable<Product>> GetOnSaleProducts() =>
    await _context.Products.Where(p => p.IsOnSale).ToListAsync();
```

在 ASP.NET Core 3.0 及更高版本中，从操作返回 `IAsyncEnumerable<T>`：

* 不再会导致同步迭代。
* 变成和返回 <xref:System.Collections.Generic.IEnumerable%601> 一样高效。

ASP.NET Core 3.0 和更高版本将缓冲以下操作的结果，然后将其提供给序列化程序：

```csharp
public IEnumerable<Product> GetOnSaleProducts() =>
    _context.Products.Where(p => p.IsOnSale);
```

考虑将操作签名的返回类型声明为 `IAsyncEnumerable<T>` 以保证异步迭代。 最终，迭代模式基于要返回的基础具体类型。 MVC 自动对实现 `IAsyncEnumerable<T>` 的任何具体类型进行缓冲。

请考虑以下操作，该操作将销售价格的产品记录返回为 `IEnumerable<Product>`：

[!code-csharp[](../web-api/action-return-types/samples/3x/WebApiSample.Api.30/Controllers/ProductsController.cs?name=snippet_GetOnSaleProducts)]

上述操作的 `IAsyncEnumerable<Product>` 等效项为：

[!code-csharp[](../web-api/action-return-types/samples/3x/WebApiSample.Api.30/Controllers/ProductsController.cs?name=snippet_GetOnSaleProductsAsync)]

自 ASP.NET Core 3.0 起，前面两个操作均为非阻止性。

## <a name="iactionresult-type"></a>IActionResult 类型

当操作中可能有多个 `ActionResult` 返回类型时，适合使用 <xref:Microsoft.AspNetCore.Mvc.IActionResult> 返回类型。 `ActionResult` 类型表示多种 HTTP 状态代码。 派生自 `ActionResult` 的任何非抽象类都限定为有效的返回类型。 此类别中的某些常见返回类型为 <xref:Microsoft.AspNetCore.Mvc.BadRequestResult> (400)、<xref:Microsoft.AspNetCore.Mvc.NotFoundResult> (404) 和 <xref:Microsoft.AspNetCore.Mvc.OkObjectResult> (200)。 或者，可以使用 <xref:Microsoft.AspNetCore.Mvc.ControllerBase> 类中的便利方法从操作返回 `ActionResult` 类型。 例如，`return BadRequest();` 是 `return new BadRequestResult();` 的简写形式。

由于这种类型的操作具有多个返回类型和路径，因此需要大量使用 [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) 特性。 此特性可针对 [Swagger](xref:tutorials/web-api-help-pages-using-swagger) 等工具生成的 Web API 帮助页生成更多描述性响应详细信息。 `[ProducesResponseType]` 指示操作将返回的已知类型和 HTTP 状态代码。

### <a name="synchronous-action"></a>同步操作

请参考以下同步操作，其中有两种可能的返回类型：

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetByIdIActionResult&highlight=8,11)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_GetById&highlight=8,11)]

::: moniker-end

在上述操作中：

* 当 `id` 代表的产品不在基础数据存储中时，则返回 404 状态代码。 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*> 便利方法作为 `return new NotFoundResult();` 的简写调用。
* 如果产品确实存在，则返回状态代码 200 及 `Product` 对象。 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.Ok*> 便利方法作为 `return new OkObjectResult(product);` 的简写调用。

### <a name="asynchronous-action"></a>异步操作

请参考以下异步操作，其中有两种可能的返回类型：

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsyncIActionResult&highlight=9,14)]

::: moniker-end

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.Pre21/Controllers/ProductsController.cs?name=snippet_CreateAsync&highlight=9,14)]

::: moniker-end

在上述操作中：

* 当产品说明包含“XYZ 小组件”时，返回 400 状态代码。 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*> 便利方法作为 `return new BadRequestResult();` 的简写调用。
* 在创建产品后，<xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> 便利方法生成 201 状态代码。 调用 `CreatedAtAction` 的替代方法是 `return new CreatedAtActionResult(nameof(GetById), "Products", new { id = product.Id }, product);`。 在此代码路径中，将在响应正文中提供 `Product` 对象。 提供了包含新建产品 URL 的 `Location` 响应标头。

例如，以下模型指明请求必须包含 `Name` 和 `Description` 属性。 未在请求中提供 `Name` 和 `Description` 会导致模型验证失败。

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.DataAccess/Models/Product.cs?name=snippet_ProductClass&highlight=5-6,8-9)]

::: moniker range=">= aspnetcore-2.1"

如果 [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute) 应用 ASP.NET Core 2.1 或更高版本中的属性，则模型验证错误将导致400状态代码。 有关详细信息，请参阅[自动 HTTP 400 响应](xref:web-api/index#automatic-http-400-responses)。

## <a name="actionresultt-type"></a>ActionResult \<T> 类型

ASP.NET Core 2.1 引入了 web API 控制器操作的[ActionResult \<T> ](xref:Microsoft.AspNetCore.Mvc.ActionResult`1)返回类型。 它支持返回从 <xref:Microsoft.AspNetCore.Mvc.ActionResult> 派生的类型或返回[特定类型](#specific-type)。 `ActionResult<T>` 通过 [IActionResult 类型](#iactionresult-type)可提供以下优势：

* [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) `Type` 可以排除特性的属性。 例如，`[ProducesResponseType(200, Type = typeof(Product))]` 简化为 `[ProducesResponseType(200)]`。 此操作的预期返回类型改为根据 `ActionResult<T>` 中的 `T` 进行推断。
* [隐式强制转换运算符](/dotnet/csharp/language-reference/keywords/implicit)支持将 `T` 和 `ActionResult` 均转换为 `ActionResult<T>`。 将 `T` 转换为 <xref:Microsoft.AspNetCore.Mvc.ObjectResult>，也就是将 `return new ObjectResult(T);` 简化为 `return T;`。

C# 不支持对接口使用隐式强制转换运算符。 因此，必须使用 `ActionResult<T>`，才能将接口转换为具体类型。 例如，在下面的示例中，使用 `IEnumerable` 不起作用：

```csharp
[HttpGet]
public ActionResult<IEnumerable<Product>> Get() =>
    _repository.GetProducts();
```

上面代码的一种修复方法是返回 `_repository.GetProducts().ToList();`。

大多数操作具有特定返回类型。 执行操作期间可能出现意外情况，不返回特定类型就是其中之一。 例如，操作的输入参数可能无法通过模型验证。 在此情况下，通常会返回相应的 `ActionResult` 类型，而不是特定类型。

### <a name="synchronous-action"></a>同步操作

请参考以下同步操作，其中有两种可能的返回类型：

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_GetByIdActionResult&highlight=8,11)]

在上述操作中：

* 当产品不在数据库中时返回状态代码 404。
* 如果产品确实存在，则返回状态代码 200 及相应的 `Product` 对象。 ASP.NET Core 2.1 之前，`return product;` 行必须是 `return Ok(product);`。

### <a name="asynchronous-action"></a>异步操作

请参考以下异步操作，其中有两种可能的返回类型：

[!code-csharp[](../web-api/action-return-types/samples/2x/WebApiSample.Api.21/Controllers/ProductsController.cs?name=snippet_CreateAsyncActionResult&highlight=9,14)]

在上述操作中：

* 在以下情况下，ASP.NET Core 运行时返回 400 状态代码 (<xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>)：
  * [`[ApiController]`](xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute)已应用该属性，并且模型验证失败。
  * 产品说明包含“XYZ 小组件”。
* 在创建产品后，<xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*> 方法生成 201 状态代码。 在此代码路径中，将在响应正文中提供 `Product` 对象。 提供了包含新建产品 URL 的 `Location` 响应标头。

::: moniker-end

## <a name="additional-resources"></a>其他资源

* <xref:mvc/controllers/actions>
* <xref:mvc/models/validation>
* <xref:tutorials/web-api-help-pages-using-swagger>
