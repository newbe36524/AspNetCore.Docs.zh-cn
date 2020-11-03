---
title: ASP.NET Core Web API 中的 JSON 修补程序
author: rick-anderson
description: 了解如何处理 ASP.NET Core Web API 中的 JSON 修补程序请求。
ms.author: riande
ms.custom: mvc
ms.date: 04/02/2020
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
uid: web-api/jsonpatch
ms.openlocfilehash: da507974b88c21de22e2c7a56950943207565138
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060542"
---
# <a name="jsonpatch-in-aspnet-core-web-api"></a>ASP.NET Core Web API 中的 JSON 修补程序

作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Kirk Larkin](https://github.com/serpent5)

::: moniker range=">= aspnetcore-3.0"

本文介绍如何处理 ASP.NET Core Web API 中的 JSON 修补程序请求。

## <a name="package-installation"></a>包安装

若要在应用中启用 JSON 修补程序支持，请完成以下步骤：

1. 安装 [`Microsoft.AspNetCore.Mvc.NewtonsoftJson`](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.NewtonsoftJson/) NuGet 包。
1. 更新项目的 `Startup.ConfigureServices` 要调用的方法 <xref:Microsoft.Extensions.DependencyInjection.NewtonsoftJsonMvcBuilderExtensions.AddNewtonsoftJson*> 。 例如： 。

    ```csharp
    services
        .AddControllersWithViews()
        .AddNewtonsoftJson();
    ```

`AddNewtonsoftJson` 与 MVC 服务注册方法兼容：

* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddRazorPages*>
* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews*>
* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllers*>

## <a name="json-patch-addnewtonsoftjson-and-systemtextjson"></a>JSON Patch、AddNewtonsoftJson 和 System.Text.Js

`AddNewtonsoftJson` 替换 `System.Text.Json` 用于格式化 **所有** JSON 内容的基于的输入和输出格式化程序。 若要使用添加对 JSON 修补程序的支持 `Newtonsoft.Json` ，而使其他格式化程序保持不变，请按如下所示更新项目的 `Startup.ConfigureServices` 方法：

[!code-csharp[](jsonpatch/samples/3.0/WebApp1/Startup.cs?name=snippet)]

前面的代码需要 `Microsoft.AspNetCore.Mvc.NewtonsoftJson` 包和以下 `using` 语句：

[!code-csharp[](jsonpatch/samples/3.0/WebApp1/Startup.cs?name=snippet1)]

## <a name="patch-http-request-method"></a>PATCH HTTP 请求方法

PUT 和 [PATCH](https://tools.ietf.org/html/rfc5789) 方法用于更新现有资源。 它们之间的区别是，PUT 会替换整个资源，而PATCH 仅指定更改。

## <a name="json-patch"></a>JSON 修补程序

[JSON 修补程序](https://tools.ietf.org/html/rfc6902)是一种格式，用于指定要应用于资源的更新。 JSON 修补程序文档有一个  操作数组。 每个操作标识一种特定类型的更改。 此类更改的示例包括添加数组元素或替换属性值。

例如，以下 JSON 文档表示资源、资源的 JSON 修补文档和应用修补程序操作的结果。

### <a name="resource-example"></a>资源示例

[!code-json[](jsonpatch/samples/2.2/JSON/customer.json)]

### <a name="json-patch-example"></a>JSON 修补程序示例

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

在上述 JSON 中：

* `op` 属性指示操作的类型。
* `path` 属性指示要更新的元素。
* `value` 属性提供新值。

### <a name="resource-after-patch"></a>修补程序之后的资源

下面是应用上述 JSON 修补程序文档后的资源：

```json
{
  "customerName": "Barry",
  "orders": [
    {
      "orderName": "Order0",
      "orderType": null
    },
    {
      "orderName": "Order1",
      "orderType": null
    },
    {
      "orderName": "Order2",
      "orderType": null
    }
  ]
}
```

通过向资源应用 JSON 修补程序文档所做的更改是原子的。 如果列表中的任何操作失败，则不会应用列表中的任何操作。

## <a name="path-syntax"></a>路径语法

操作对象的[路径](https://tools.ietf.org/html/rfc6901)属性的级别之间有斜杠。 例如 `"/address/zipCode"`。

使用从零开始的索引来指定数组元素。 `addresses` 数组的第一个元素将位于 `/addresses/0`。 若要到 `add` 数组的末尾，请使用连字符 (`-`) 而不是索引号： `/addresses/-` 。

### <a name="operations"></a>Operations

下表显示了 [JSON 修补程序规范](https://tools.ietf.org/html/rfc6902)中定义的支持操作：

|Operation  | 备注 |
|-----------|--------------------------------|
| `add`     | 添加属性或数组元素。 对于现有属性：设置值。|
| `remove`  | 删除属性或数组元素。 |
| `replace` | 与在相同位置后跟 `add` 的 `remove` 相同。 |
| `move`    | 与从后跟 `add` 的源到使用源中的值的目标的 `remove` 相同。 |
| `copy`    | 与到使用源中的值的目标的 `add` 相同。 |
| `test`    | 如果 `path` 处的值 = 提供的 `value`，则返回成功状态代码。|

## <a name="json-patch-in-aspnet-core"></a>ASP.NET Core 中的 JSON 修补程序

[Microsoft.AspNetCore.JsonPatch](https://www.nuget.org/packages/microsoft.aspnetcore.jsonpatch/) NuGet 包中提供了 JSON 修补程序的 ASP.NET Core 实现。

## <a name="action-method-code"></a>操作方法代码

在 API 控制器中，JSON 修补程序的操作方法：

* 使用 `HttpPatch` 属性进行批注。
* 接受 `JsonPatchDocument<T>`，通常带有 `[FromBody]`。
* 在修补程序文档上调用 `ApplyTo` 以应用更改。

以下是一个示例：

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_PatchAction&highlight=1,3,9)]

此示例应用程序中的代码可用于以下 `Customer` 模型：

[!code-csharp[](jsonpatch/samples/2.2/Models/Customer.cs?name=snippet_Customer)]

[!code-csharp[](jsonpatch/samples/2.2/Models/Order.cs?name=snippet_Order)]

示例操作方法：

* 构造一个 `Customer`。
* 应用修补程序。
* 在响应的正文中返回结果。

在实际应用中，该代码将从存储（如数据库）中检索数据并在应用修补程序后更新数据库。

### <a name="model-state"></a>模型状态

上述操作方法示例调用将模型状态用作其参数之一的 `ApplyTo` 的重载。 使用此选项，可以在响应中收到错误消息。 以下示例显示了 `test` 操作的“400 错误请求”响应的正文：

```json
{
    "Customer": [
        "The current value 'John' at path 'customerName' is not equal to the test value 'Nancy'."
    ]
}
```

### <a name="dynamic-objects"></a>动态对象

以下操作方法示例演示如何将修补程序应用于动态对象：

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_Dynamic)]

## <a name="the-add-operation"></a>添加操作

* 如果 `path` 指向数组元素：将新元素插入到 `path` 指定的元素之前。
* 如果 `path` 指向属性：设置属性值。
* 如果 `path` 指向不存在的位置：
  * 如果要修补的资源是一个动态对象：添加属性。
  * 如果要修补的资源是一个静态对象：请求失败。

以下示例修补程序文档设置 `CustomerName` 的值，并将 `Order` 对象添加到 `Orders` 数组末尾。

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

## <a name="the-remove-operation"></a>删除操作

* 如果 `path` 指向数组元素：删除元素。
* 如果 `path` 指向属性：
  * 如果要修补的资源是一个动态对象：删除属性。
  * 如果要修补的资源是一个静态对象：
    * 如果属性可以为 Null：将其设置为 null。
    * 如果属性不可以为 Null，将其设置为 `default<T>`。

下面的示例将修补文档集设置 `CustomerName` 为 null 并删除 `Orders[0]` ：

[!code-json[](jsonpatch/samples/2.2/JSON/remove.json)]

## <a name="the-replace-operation"></a>替换操作

此操作在功能上与后跟 `add` 的 `remove` 相同。

以下示例修补文档设置的值 `CustomerName` ，并将替换 `Orders[0]` 为新的 `Order` 对象：

[!code-json[](jsonpatch/samples/2.2/JSON/replace.json)]

## <a name="the-move-operation"></a>移动操作

* 如果 `path` 指向数组元素：将 `from` 元素复制到 `path` 元素的位置，然后对 `from` 元素运行 `remove` 操作。
* 如果 `path` 指向属性：将 `from` 属性的值复制到 `path` 属性，然后对 `from` 属性运行 `remove` 操作。
* 如果 `path` 指向不存在的属性：
  * 如果要修补的资源是一个静态对象：请求失败。
  * 如果要修补的资源是一个动态对象：将 `from` 属性复制到 `path` 指示的位置，然后对 `from` 属性运行 `remove` 操作。

以下示例修补程序文档：

* 将 `Orders[0].OrderName` 的值复制到 `CustomerName`。
* 将 `Orders[0].OrderName` 设置为 null。
* 将 `Orders[1]` 移动到 `Orders[0]` 之前。

[!code-json[](jsonpatch/samples/2.2/JSON/move.json)]

## <a name="the-copy-operation"></a>复制操作

此操作在功能上与不包含最后 `remove` 步骤的 `move` 操作相同。

以下示例修补程序文档：

* 将 `Orders[0].OrderName` 的值复制到 `CustomerName`。
* 将 `Orders[1]` 的副本插入到 `Orders[0]` 之前。

[!code-json[](jsonpatch/samples/2.2/JSON/copy.json)]

## <a name="the-test-operation"></a>测试操作

如果 `path` 指示的位置处的值与 `value` 中提供的值不同，则请求会失败。 在这种情况下，整个 PATCH 请求会失败，即使修补程序文档中的所有其他操作均成功也是如此。

`test` 操作通常用于在发生并发冲突时阻止更新。

如果 `CustomerName` 的初始值是“John”，则以下示例修补程序文档不起任何作用，因为测试失败：

[!code-json[](jsonpatch/samples/2.2/JSON/test-fail.json)]

## <a name="get-the-code"></a>获取代码

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/jsonpatch/samples)。 （[下载方法](xref:index#how-to-download-a-sample)）。

若要测试此示例，请使用以下设置运行应用并发送 HTTP 请求：

* URL： `http://localhost:{port}/jsonpatch/jsonpatchwithmodelstate`
* HTTP 方法：`PATCH`
* 标头：`Content-Type: application/json-patch+json`
* Body：从 *json* 项目文件夹复制并粘贴一个 json 修补文档示例。

## <a name="additional-resources"></a>其他资源

* [IETF RFC 5789 PATCH 方法规范](https://tools.ietf.org/html/rfc5789)
* [IETF RFC 6902 JSON 修补程序规范](https://tools.ietf.org/html/rfc6902)
* [IETF RFC 6901 JSON 修补程序路径格式规范](https://tools.ietf.org/html/rfc6901)
* [JSON 路径文档](https://jsonpatch.com/)。 包括指向用于创建 JSON 修补程序文档的资源的链接。
* [ASP.NET Core JSON 修补程序源代码](https://github.com/dotnet/AspNetCore/tree/master/src/Features/JsonPatch/src)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本文介绍如何处理 ASP.NET Core Web API 中的 JSON 修补程序请求。

## <a name="patch-http-request-method"></a>PATCH HTTP 请求方法

PUT 和 [PATCH](https://tools.ietf.org/html/rfc5789) 方法用于更新现有资源。 它们之间的区别是，PUT 会替换整个资源，而PATCH 仅指定更改。

## <a name="json-patch"></a>JSON 修补程序

[JSON 修补程序](https://tools.ietf.org/html/rfc6902)是一种格式，用于指定要应用于资源的更新。 JSON 修补程序文档有一个  操作数组。 每个操作标识特定类型的更改，例如添加数组元素或替换属性值。

例如，以下 JSON 文档表示资源、资源的 JSON 修补程序文档和应用修补程序操作的结果。

### <a name="resource-example"></a>资源示例

[!code-json[](jsonpatch/samples/2.2/JSON/customer.json)]

### <a name="json-patch-example"></a>JSON 修补程序示例

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

在上述 JSON 中：

* `op` 属性指示操作的类型。
* `path` 属性指示要更新的元素。
* `value` 属性提供新值。

### <a name="resource-after-patch"></a>修补程序之后的资源

下面是应用上述 JSON 修补程序文档后的资源：

```json
{
  "customerName": "Barry",
  "orders": [
    {
      "orderName": "Order0",
      "orderType": null
    },
    {
      "orderName": "Order1",
      "orderType": null
    },
    {
      "orderName": "Order2",
      "orderType": null
    }
  ]
}
```

通过将 JSON 修补程序文档应用于资源所做的更改是原子操作：如果列表中的任何操作失败，则不会应用列表中的任何操作。

## <a name="path-syntax"></a>路径语法

操作对象的[路径](https://tools.ietf.org/html/rfc6901)属性的级别之间有斜杠。 例如 `"/address/zipCode"`。

使用从零开始的索引来指定数组元素。 `addresses` 数组的第一个元素将位于 `/addresses/0`。 若要将 `add` 置于数组末尾，请使用连字符 (-)，而不是索引号：`/addresses/-`。

### <a name="operations"></a>Operations

下表显示了 [JSON 修补程序规范](https://tools.ietf.org/html/rfc6902)中定义的支持操作：

|Operation  | 备注 |
|-----------|--------------------------------|
| `add`     | 添加属性或数组元素。 对于现有属性：设置值。|
| `remove`  | 删除属性或数组元素。 |
| `replace` | 与在相同位置后跟 `add` 的 `remove` 相同。 |
| `move`    | 与从后跟 `add` 的源到使用源中的值的目标的 `remove` 相同。 |
| `copy`    | 与到使用源中的值的目标的 `add` 相同。 |
| `test`    | 如果 `path` 处的值 = 提供的 `value`，则返回成功状态代码。|

## <a name="jsonpatch-in-aspnet-core"></a>ASP.NET Core 中的 JSON 修补程序

[Microsoft.AspNetCore.JsonPatch](https://www.nuget.org/packages/microsoft.aspnetcore.jsonpatch/) NuGet 包中提供了 JSON 修补程序的 ASP.NET Core 实现。 此包包含在 [AspnetCore](xref:fundamentals/metapackage-app) 元包中。

## <a name="action-method-code"></a>操作方法代码

在 API 控制器中，JSON 修补程序的操作方法：

* 使用 `HttpPatch` 属性进行批注。
* 接受 `JsonPatchDocument<T>`，通常带有 `[FromBody]`。
* 在修补程序文档上调用 `ApplyTo` 以应用更改。

以下是一个示例：

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_PatchAction&highlight=1,3,9)]

示例应用中的此代码适用于以下 `Customer` 模型。

[!code-csharp[](jsonpatch/samples/2.2/Models/Customer.cs?name=snippet_Customer)]

[!code-csharp[](jsonpatch/samples/2.2/Models/Order.cs?name=snippet_Order)]

示例操作方法：

* 构造一个 `Customer`。
* 应用修补程序。
* 在响应的正文中返回结果。

 在实际应用中，该代码将从存储（如数据库）中检索数据并在应用修补程序后更新数据库。

### <a name="model-state"></a>模型状态

上述操作方法示例调用将模型状态用作其参数之一的 `ApplyTo` 的重载。 使用此选项，可以在响应中收到错误消息。 以下示例显示了 `test` 操作的“400 错误请求”响应的正文：

```json
{
    "Customer": [
        "The current value 'John' at path 'customerName' is not equal to the test value 'Nancy'."
    ]
}
```

### <a name="dynamic-objects"></a>动态对象

以下操作方法示例演示如何将修补程序应用于动态对象。

[!code-csharp[](jsonpatch/samples/2.2/Controllers/HomeController.cs?name=snippet_Dynamic)]

## <a name="the-add-operation"></a>添加操作

* 如果 `path` 指向数组元素：将新元素插入到 `path` 指定的元素之前。
* 如果 `path` 指向属性：设置属性值。
* 如果 `path` 指向不存在的位置：
  * 如果要修补的资源是一个动态对象：添加属性。
  * 如果要修补的资源是一个静态对象：请求失败。

以下示例修补程序文档设置 `CustomerName` 的值，并将 `Order` 对象添加到 `Orders` 数组末尾。

[!code-json[](jsonpatch/samples/2.2/JSON/add.json)]

## <a name="the-remove-operation"></a>删除操作

* 如果 `path` 指向数组元素：删除元素。
* 如果 `path` 指向属性：
  * 如果要修补的资源是一个动态对象：删除属性。
  * 如果要修补的资源是一个静态对象：
    * 如果属性可以为 Null：将其设置为 null。
    * 如果属性不可以为 Null，将其设置为 `default<T>`。

以下示例修补程序文档将 `CustomerName` 设置为 null 并删除 `Orders[0]`。

[!code-json[](jsonpatch/samples/2.2/JSON/remove.json)]

## <a name="the-replace-operation"></a>替换操作

此操作在功能上与后跟 `add` 的 `remove` 相同。

以下示例修补程序文档设置 `CustomerName` 的值，并将 `Orders[0]` 替换为新的 `Order` 对象。

[!code-json[](jsonpatch/samples/2.2/JSON/replace.json)]

## <a name="the-move-operation"></a>移动操作

* 如果 `path` 指向数组元素：将 `from` 元素复制到 `path` 元素的位置，然后对 `from` 元素运行 `remove` 操作。
* 如果 `path` 指向属性：将 `from` 属性的值复制到 `path` 属性，然后对 `from` 属性运行 `remove` 操作。
* 如果 `path` 指向不存在的属性：
  * 如果要修补的资源是一个静态对象：请求失败。
  * 如果要修补的资源是一个动态对象：将 `from` 属性复制到 `path` 指示的位置，然后对 `from` 属性运行 `remove` 操作。

以下示例修补程序文档：

* 将 `Orders[0].OrderName` 的值复制到 `CustomerName`。
* 将 `Orders[0].OrderName` 设置为 null。
* 将 `Orders[1]` 移动到 `Orders[0]` 之前。

[!code-json[](jsonpatch/samples/2.2/JSON/move.json)]

## <a name="the-copy-operation"></a>复制操作

此操作在功能上与不包含最后 `remove` 步骤的 `move` 操作相同。

以下示例修补程序文档：

* 将 `Orders[0].OrderName` 的值复制到 `CustomerName`。
* 将 `Orders[1]` 的副本插入到 `Orders[0]` 之前。

[!code-json[](jsonpatch/samples/2.2/JSON/copy.json)]

## <a name="the-test-operation"></a>测试操作

如果 `path` 指示的位置处的值与 `value` 中提供的值不同，则请求会失败。 在这种情况下，整个 PATCH 请求会失败，即使修补程序文档中的所有其他操作均成功也是如此。

`test` 操作通常用于在发生并发冲突时阻止更新。

如果 `CustomerName` 的初始值是“John”，则以下示例修补程序文档不起任何作用，因为测试失败：

[!code-json[](jsonpatch/samples/2.2/JSON/test-fail.json)]

## <a name="get-the-code"></a>获取代码

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/jsonpatch/samples/2.2)。 （[下载方法](xref:index#how-to-download-a-sample)）。

若要测试此示例，请使用以下设置运行应用并发送 HTTP 请求：

* URL： `http://localhost:{port}/jsonpatch/jsonpatchwithmodelstate`
* HTTP 方法：`PATCH`
* 标头：`Content-Type: application/json-patch+json`
* Body：从 *json* 项目文件夹复制并粘贴一个 json 修补文档示例。

## <a name="additional-resources"></a>其他资源

* [IETF RFC 5789 PATCH 方法规范](https://tools.ietf.org/html/rfc5789)
* [IETF RFC 6902 JSON 修补程序规范](https://tools.ietf.org/html/rfc6902)
* [IETF RFC 6901 JSON 修补程序路径格式规范](https://tools.ietf.org/html/rfc6901)
* [JSON 路径文档](https://jsonpatch.com/)。 包括指向用于创建 JSON 修补程序文档的资源的链接。
* [ASP.NET Core JSON 修补程序源代码](https://github.com/dotnet/AspNetCore/tree/master/src/Features/JsonPatch/src)

::: moniker-end
