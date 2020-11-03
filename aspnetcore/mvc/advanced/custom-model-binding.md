---
title: ASP.NET Core 中的自定义模型绑定
author: ardalis
description: 了解如何通过模型绑定，使控制器操作能够直接使用 ASP.NET Core 中的模型类型。
ms.author: riande
ms.date: 01/06/2020
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
uid: mvc/advanced/custom-model-binding
ms.openlocfilehash: 7675e95c43b9ee428ee5fda86ea3ead9815ed645
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93058462"
---
# <a name="custom-model-binding-in-aspnet-core"></a>ASP.NET Core 中的自定义模型绑定

::: moniker range=">= aspnetcore-3.0"

作者：[Steve Smith](https://ardalis.com/) 和 [Kirk Larkin](https://twitter.com/serpent5)

通过模型绑定，控制器操作可直接使用模型类型（作为方法参数传入）而不是 HTTP 请求。 由模型绑定器处理传入的请求数据和应用程序模型之间的映射。 开发人员可以通过实现自定义模型绑定器来扩展内置的模型绑定功能（尽管通常不需要编写自己的提供程序）。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/custom-model-binding/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="default-model-binder-limitations"></a>默认模型绑定器限制

默认模型绑定器支持大多数常见的 .NET Core 数据类型，能够满足大部分开发人员的需求。 他们希望将基于文本的输入从请求直接绑定到模型类型。 绑定输入之前，可能需要对其进行转换。 例如，当拥有某个可以用来查找模型数据的键时。 基于该键，用户可以使用自定义模型绑定器来获取数据。

## <a name="model-binding-review"></a>模型绑定查看

模型绑定为其操作对象的类型使用特定定义。 简单类型  转换自输入中的单个字符串。 复杂类型  转换自多个输入值。 框架基于是否存在 `TypeConverter` 来确定差异。 如果简单 `string` -> `SomeType` 映射不需要外部资源，建议创建类型转换器。

创建自己的自定义模型绑定器之前，有必要查看现有模型绑定器的实现方式。 考虑使用 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder>，它可将 base64 编码的字符串转换为字节数组。 字节数组通常存储为文件或数据库 BLOB 字段。

### <a name="working-with-the-bytearraymodelbinder"></a>使用 ByteArrayModelBinder

Base64 编码的字符串可用来表示二进制数据。 例如，可将图像编码为一个字符串。 示例包括作为使用 [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/advanced/custom-model-binding/samples/3.x/CustomModelBindingSample/Base64String.txt) 的 base64 编码字符串的图像。

ASP.NET Core MVC 可以采用 base64 编码的字符串，并使用 `ByteArrayModelBinder` 将其转换为字节数组。 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider> 将 `byte[]` 参数映射到 `ByteArrayModelBinder`：

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        var loggerFactory = context.Services.GetRequiredService<ILoggerFactory>();
        return new ByteArrayModelBinder(loggerFactory);
    }

    return null;
}
```

创建自己的自定义模型绑定器时，可实现自己的 `IModelBinderProvider` 类型，或使用 <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute>。

以下示例显示如何使用 `ByteArrayModelBinder` 将 base64 编码的字符串转换为 `byte[]`，并将结果保存到文件中：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_Post)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

可以使用 [Postman](https://www.getpostman.com/) 等工具将 base64 编码的字符串发布到此 api 方法：

![postman](custom-model-binding/images/postman.png "Postman")

只要绑定器可以将请求数据绑定到相应命名的属性或参数，模型绑定就会成功。 以下示例演示如何将 `ByteArrayModelBinder` 与 视图模型结合使用：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/ImageController.cs?name=snippet_SaveProfile&highlight=2)]

## <a name="custom-model-binder-sample"></a>自定义模型绑定器示例

在本部分中，我们将实现具有以下功能的自定义模型绑定器：

- 将传入的请求数据转换为强类型键参数。
- 使用 Entity Framework Core 来提取关联的实体。
- 将关联的实体作为自变量传递给操作方法。

以下示例在 `Author` 模型上使用 `ModelBinder` 属性：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

在前面的代码中，`ModelBinder` 属性指定应当用于绑定 `Author` 操作参数的 `IModelBinder` 的类型。

以下 `AuthorEntityBinder` 类通过 Entity Framework Core 和 `authorId` 提取数据源中的实体来绑定 `Author` 参数：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=snippet_Class)]

> [!NOTE]
> 前面的 `AuthorEntityBinder` 类旨在说明自定义模型绑定器。 该类不是意在说明查找方案的最佳做法。 对于查找，请绑定 `authorId` 并在操作方法中查询数据库。 此方法将模型绑定故障与 `NotFound` 案例分开。

以下代码显示如何在操作方法中使用 `AuthorEntityBinder`：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_Get&highlight=2)]

可使用 `ModelBinder` 属性将 `AuthorEntityBinder` 应用于不使用默认约定的参数：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=snippet_GetById&highlight=2)]

在此示例中，由于参数的名称不是默认的 `authorId`，因此，使用 `ModelBinder` 属性在参数上指定该名称。 比起在操作方法中查找实体，控制器和操作方法都得到了简化。 使用 Entity Framework Core 获取创建者的逻辑会移动到模型绑定器。 如果有多种方法绑定到 `Author` 模型，就能得到很大程度的简化。

可以将 `ModelBinder` 属性应用到各个模型属性（例如视图模型上）或操作方法参数，以便为该类型或操作指定某一模型绑定器或模型名称。

### <a name="implementing-a-modelbinderprovider"></a>实现 ModelBinderProvider

可以实现 `IModelBinderProvider`，而不是应用属性。 这就是内置框架绑定器的实现方式。 指定绑定器所操作的类型时，指定它生成的参数的类型，而不是  绑定器接受的输入。 以下绑定器提供程序适用于 `AuthorEntityBinder`。 将其添加到 MVC 提供程序的集合中时，无需在 `Author` 或 `Author` 类型参数上使用 `ModelBinder` 属性。

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> 注意：上述代码返回 `BinderTypeModelBinder`。 `BinderTypeModelBinder` 充当模型绑定器中心，并提供依赖关系注入 (DI)。 `AuthorEntityBinder` 需要 DI 来访问 EF Core。 如果模型绑定器需要 DI 中的服务，请使用 `BinderTypeModelBinder`。

若要使用自定义模型绑定器提供程序，请将其添加到 `ConfigureServices` 中：

[!code-csharp[](custom-model-binding/samples/3.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-8)]

评估模型绑定器时，按顺序检查提供程序的集合。 使用第一个返回与输入模型匹配的联编程序的提供程序。 因此，将提供程序添加到集合的末尾可能会导致在自定义联编程序有可能之前调用内置模型联编程序。 在此示例中，将自定义提供程序添加到集合的开头，以确保它始终用于 `Author` 操作参数。

### <a name="polymorphic-model-binding"></a>多态模型绑定

绑定到不同的派生类型模型称为多态模型绑定。 如果请求值必须绑定到特定的派生模型类型，则需要多态自定义模型绑定。 多态模型绑定：

* 对于旨在与所有语言进行互操作的 REST API 并不常见。
* 使绑定模型难以推理。

但是，如果应用需要多态模型绑定，则实现可能类似于以下代码：

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a>建议和最佳做法

自定义模型绑定：

- 不应尝试设置状态代码或返回结果（例如 404 Not Found）。 如果模型绑定失败，那么该操作方法本身的[操作筛选器](xref:mvc/controllers/filters)或逻辑会处理失败。
- 对于消除操作方法中的重复代码和跨领域问题最为有用。
- 通常不应用其将字符串转换为自定义类型，而应选择用 <xref:System.ComponentModel.TypeConverter> 来完成此操作。

::: moniker-end
::: moniker range="< aspnetcore-3.0"

作者：[Steve Smith](https://ardalis.com/)

通过模型绑定，控制器操作可直接使用模型类型（作为方法参数传入）而不是 HTTP 请求。 由模型绑定器处理传入的请求数据和应用程序模型之间的映射。 开发人员可以通过实现自定义模型绑定器来扩展内置的模型绑定功能（尽管通常不需要编写自己的提供程序）。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/advanced/custom-model-binding/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="default-model-binder-limitations"></a>默认模型绑定器限制

默认模型绑定器支持大多数常见的 .NET Core 数据类型，能够满足大部分开发人员的需求。 他们希望将基于文本的输入从请求直接绑定到模型类型。 绑定输入之前，可能需要对其进行转换。 例如，当拥有某个可以用来查找模型数据的键时。 基于该键，用户可以使用自定义模型绑定器来获取数据。

## <a name="model-binding-review"></a>模型绑定查看

模型绑定为其操作对象的类型使用特定定义。 简单类型  转换自输入中的单个字符串。 复杂类型  转换自多个输入值。 框架基于是否存在 `TypeConverter` 来确定差异。 如果简单 `string` -> `SomeType` 映射不需要外部资源，建议创建类型转换器。

创建自己的自定义模型绑定器之前，有必要查看现有模型绑定器的实现方式。 考虑使用 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinder>，它可将 base64 编码的字符串转换为字节数组。 字节数组通常存储为文件或数据库 BLOB 字段。

### <a name="working-with-the-bytearraymodelbinder"></a>使用 ByteArrayModelBinder

Base64 编码的字符串可用来表示二进制数据。 例如，可将图像编码为一个字符串。 示例包括作为使用 [Base64String.txt](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/advanced/custom-model-binding/samples/2.x/CustomModelBindingSample/Base64String.txt) 的 base64 编码字符串的图像。

ASP.NET Core MVC 可以采用 base64 编码的字符串，并使用 `ByteArrayModelBinder` 将其转换为字节数组。 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Binders.ByteArrayModelBinderProvider> 将 `byte[]` 参数映射到 `ByteArrayModelBinder`：

```csharp
public IModelBinder GetBinder(ModelBinderProviderContext context)
{
    if (context == null)
    {
        throw new ArgumentNullException(nameof(context));
    }

    if (context.Metadata.ModelType == typeof(byte[]))
    {
        return new ByteArrayModelBinder();
    }

    return null;
}
```

创建自己的自定义模型绑定器时，可实现自己的 `IModelBinderProvider` 类型，或使用 <xref:Microsoft.AspNetCore.Mvc.ModelBinderAttribute>。

以下示例显示如何使用 `ByteArrayModelBinder` 将 base64 编码的字符串转换为 `byte[]`，并将结果保存到文件中：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post1)]

可以使用 [Postman](https://www.getpostman.com/) 等工具将 base64 编码的字符串发布到此 api 方法：

![postman](custom-model-binding/images/postman.png "Postman")

只要绑定器可以将请求数据绑定到相应命名的属性或参数，模型绑定就会成功。 以下示例演示如何将 `ByteArrayModelBinder` 与 视图模型结合使用：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/ImageController.cs?name=post2&highlight=2)]

## <a name="custom-model-binder-sample"></a>自定义模型绑定器示例

在本部分中，我们将实现具有以下功能的自定义模型绑定器：

- 将传入的请求数据转换为强类型键参数。
- 使用 Entity Framework Core 来提取关联的实体。
- 将关联的实体作为自变量传递给操作方法。

以下示例在 `Author` 模型上使用 `ModelBinder` 属性：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Data/Author.cs?highlight=6)]

在前面的代码中，`ModelBinder` 属性指定应当用于绑定 `Author` 操作参数的 `IModelBinder` 的类型。

以下 `AuthorEntityBinder` 类通过 Entity Framework Core 和 `authorId` 提取数据源中的实体来绑定 `Author` 参数：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinder.cs?name=demo)]

> [!NOTE]
> 前面的 `AuthorEntityBinder` 类旨在说明自定义模型绑定器。 该类不是意在说明查找方案的最佳做法。 对于查找，请绑定 `authorId` 并在操作方法中查询数据库。 此方法将模型绑定故障与 `NotFound` 案例分开。

以下代码显示如何在操作方法中使用 `AuthorEntityBinder`：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo2&highlight=2)]

可使用 `ModelBinder` 属性将 `AuthorEntityBinder` 应用于不使用默认约定的参数：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Controllers/BoundAuthorsController.cs?name=demo1&highlight=2)]

在此示例中，由于参数的名称不是默认的 `authorId`，因此，使用 `ModelBinder` 属性在参数上指定该名称。 比起在操作方法中查找实体，控制器和操作方法都得到了简化。 使用 Entity Framework Core 获取创建者的逻辑会移动到模型绑定器。 如果有多种方法绑定到 `Author` 模型，就能得到很大程度的简化。

可以将 `ModelBinder` 属性应用到各个模型属性（例如视图模型上）或操作方法参数，以便为该类型或操作指定某一模型绑定器或模型名称。

### <a name="implementing-a-modelbinderprovider"></a>实现 ModelBinderProvider

可以实现 `IModelBinderProvider`，而不是应用属性。 这就是内置框架绑定器的实现方式。 指定绑定器所操作的类型时，指定它生成的参数的类型，而不是  绑定器接受的输入。 以下绑定器提供程序适用于 `AuthorEntityBinder`。 将其添加到 MVC 提供程序的集合中时，无需在 `Author` 或 `Author` 类型参数上使用 `ModelBinder` 属性。

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Binders/AuthorEntityBinderProvider.cs?highlight=17-20)]

> 注意：上述代码返回 `BinderTypeModelBinder`。 `BinderTypeModelBinder` 充当模型绑定器中心，并提供依赖关系注入 (DI)。 `AuthorEntityBinder` 需要 DI 来访问 EF Core。 如果模型绑定器需要 DI 中的服务，请使用 `BinderTypeModelBinder`。

若要使用自定义模型绑定器提供程序，请将其添加到 `ConfigureServices` 中：

[!code-csharp[](custom-model-binding/samples/2.x/CustomModelBindingSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-10)]

评估模型绑定器时，按顺序检查提供程序的集合。 使用返回绑定器的第一个提供程序。 向集合的末尾添加提供程序，可能会导致在调用自定义绑定器之前调用内置模型绑定器。 在此示例中，向集合的开头添加自定义提供程序，确保它用于 `Author` 操作参数。

### <a name="polymorphic-model-binding"></a>多态模型绑定

绑定到不同的派生类型模型称为多态模型绑定。 如果请求值必须绑定到特定的派生模型类型，则需要多态自定义模型绑定。 多态模型绑定：

* 对于旨在与所有语言进行互操作的 REST API 并不常见。
* 使绑定模型难以推理。

但是，如果应用需要多态模型绑定，则实现可能类似于以下代码：

[!code-csharp[](custom-model-binding/samples/3.x/PolymorphicModelBindingSample/ModelBinders/PolymorphicModelBinder.cs?name=snippet)]

## <a name="recommendations-and-best-practices"></a>建议和最佳做法

自定义模型绑定：

- 不应尝试设置状态代码或返回结果（例如 404 Not Found）。 如果模型绑定失败，那么该操作方法本身的[操作筛选器](xref:mvc/controllers/filters)或逻辑会处理失败。
- 对于消除操作方法中的重复代码和跨领域问题最为有用。
- 通常不应用其将字符串转换为自定义类型，而应选择用 <xref:System.ComponentModel.TypeConverter> 来完成此操作。

::: moniker-end
