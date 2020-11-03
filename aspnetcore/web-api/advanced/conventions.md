---
title: 使用 Web API 约定
author: pranavkm
description: 了解 ASP.NET Core 中的 Web API 约定。
monikerRange: '>= aspnetcore-2.2'
ms.author: scaddie
ms.custom: mvc
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
uid: web-api/advanced/conventions
ms.openlocfilehash: 0c5ea8ba69e4c6287afce1771ac9cee65bb188a8
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052534"
---
# <a name="use-web-api-conventions"></a>使用 Web API 约定

作者：[Pranav Krishnamoorthy](https://github.com/pranavkm) 和 [Scott Addie](https://github.com/scottaddie)

ASP.NET Core 2.2 及更高版本附带一种方法，可提取常见的 [API 文档](xref:tutorials/web-api-help-pages-using-swagger)并将其应用于多个操作、控制器或某程序集内的所有控制器。 Web API 约定是使用修饰单独操作的替代方案 [`[ProducesResponseType]`](xref:Microsoft.AspNetCore.Mvc.ProducesResponseTypeAttribute) 。

使用此约定，可以：

* 定义通过特定操作类型返回的、最常见的返回类型和状态代码。
* 识别偏离所定义的标准的操作。

ASP.NET Core MVC 2.2 及更高版本在 <xref:Microsoft.AspNetCore.Mvc.DefaultApiConventions?displayProperty=fullName> 中包含一组默认的约定。 约定基于 ASP.NET Core API 项目模板中提供的控件 (ValuesController.cs)  。 若操作遵循模板中的模式，则应成功使用默认约定。 如果默认约定不能满足需要，请参阅[创建 Web API 约定](#create-web-api-conventions)。

在运行时，<xref:Microsoft.AspNetCore.Mvc.ApiExplorer> 会了解约定。 `ApiExplorer` 是 MVC 与 [OpenAPI](https://www.openapis.org/)（也称为 Swagger）文档生成器进行通信的抽象内容。 已应用的约定中的属性与某个操作相关联，并包含在操作的 OpenAPI 文档中。 [API 分析器](xref:web-api/advanced/analyzers) 还了解约定。 若操作为非常规操作（例如，它返回已应用的约定未记录的状态代码），则会生成警告，建议记录该状态代码。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/conventions/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="apply-web-api-conventions"></a>应用 Web API 约定

约定不是组合而成的，每个操作可能只与一个约定相关联。 更明确的约定优先于不太明确的约定。 当具有相同优先级的两个或更多约定应用于某个操作时，选择是不确定的。 存在以下可将约定应用于操作的选项，明确性依次降低：

1. `Microsoft.AspNetCore.Mvc.ApiConventionMethodAttribute`&mdash;适用于各个操作，并指定应用的约定类型和约定方法。

    在以下示例中，默认约定类型的 `Microsoft.AspNetCore.Mvc.DefaultApiConventions.Put` 约定方法将应用于 `Update` 操作：

    [!code-csharp[](conventions/sample/Controllers/ContactsConventionController.cs?name=snippet_ApiConventionMethod&highlight=3)]

    `Microsoft.AspNetCore.Mvc.DefaultApiConventions.Put` 约定方法可将以下属性应用于操作：

    ```csharp
    [ProducesDefaultResponseType]
    [ProducesResponseType(StatusCodes.Status204NoContent)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    ```

    若要详细了解 `[ProducesDefaultResponseType]`，请参阅[默认响应](https://swagger.io/docs/specification/describing-responses/#default)。

1. `Microsoft.AspNetCore.Mvc.ApiConventionTypeAttribute` 应用于控制器 &mdash; 将指定约定类型应用于控制器上的所有操作。 约定方法都标记有提示，可确定要向其应用约定方法的操作。 有关提示的详细信息，请参阅[创建 Web API 约定](#create-web-api-conventions)）。

    在以下示例中，默认的约定集将应用于 ContactsConventionController 中的所有操作  ：

    [!code-csharp[](conventions/sample/Controllers/ContactsConventionController.cs?name=snippet_ApiConventionTypeAttribute&highlight=2)]

1. `Microsoft.AspNetCore.Mvc.ApiConventionTypeAttribute` 应用于程序集 &mdash; 将指定约定类型应用于当前程序集中的所有控制器。 建议将程序集级别的属性应用于 Startup.cs  文件。

    在以下示例中，默认的约定集将应用于程序集中的所有操作：

    [!code-csharp[](conventions/sample/Startup.cs?name=snippet_ApiConventionTypeAttribute&highlight=1)]

## <a name="create-web-api-conventions"></a>创建 Web API 约定

如果默认 API 约定不能满足需要，请创建自己的约定。 约定是：

* 带有方法的静态类型。
* 能够对操作定义[响应类型](#response-types)和[命名要求](#naming-requirements)。

### <a name="response-types"></a>响应类型

这些方法使用 `[ProducesResponseType]` 或 `[ProducesDefaultResponseType]` 属性进行批注。 例如： 。

```csharp
public static class MyAppConventions
{
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public static void Find(int id)
    {
    }
}
```

如果没有更具体的元数据属性，则将此约定应用于程序集可强制实现以下内容：

* 该约定方法应用于所有名为 `Find` 的操作。
* `Find` 操作上存在名为 `id` 的参数。

### <a name="naming-requirements"></a>命名要求

`[ApiConventionNameMatch]` 和 `[ApiConventionTypeMatch]` 属性可应用于约定方法，确定它们所要应用的操作。 例如： 。

```csharp
[ProducesResponseType(StatusCodes.Status200OK)]
[ProducesResponseType(StatusCodes.Status404NotFound)]
[ApiConventionNameMatch(ApiConventionNameMatchBehavior.Prefix)]
public static void Find(
    [ApiConventionNameMatch(ApiConventionNameMatchBehavior.Suffix)]
    int id)
{ }
```

在上面的示例中：

* 应用于该方法的 `Microsoft.AspNetCore.Mvc.ApiExplorer.ApiConventionNameMatchBehavior.Prefix` 选项表示该约定可匹配前缀是“Find”的任何操作。 匹配的操作可以是 `Find`、`FindPet` 和 `FindById`。
* 应用于该参数的 `Microsoft.AspNetCore.Mvc.ApiExplorer.ApiConventionNameMatchBehavior.Suffix` 表示该约定可匹配带有唯一以标识符作为后缀结尾的参数的方法。 示例包括 `id` 或 `petId` 等参数。 与此类似，可将 `ApiConventionTypeMatch` 应用于类型，以约束参数类型。 `params[]` 参数指示无需显式匹配的剩余参数。

## <a name="additional-resources"></a>其他资源

* <xref:web-api/advanced/analyzers>
* <xref:tutorials/web-api-help-pages-using-swagger>
