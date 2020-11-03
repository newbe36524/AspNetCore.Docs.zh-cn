---
title: ASP.NET Core MVC 中的模型验证
author: rick-anderson
description: 了解 ASP.NET Core MVC 和页面中的模型验证 Razor 。
ms.author: riande
ms.custom: mvc
ms.date: 12/15/2019
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
uid: mvc/models/validation
ms.openlocfilehash: 77d49710b9d69f6fbbe92970f1c455de32489444
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93056954"
---
# <a name="model-validation-in-aspnet-core-mvc-and-no-locrazor-pages"></a>ASP.NET Core MVC 和页面中的模型验证 Razor

::: moniker range=">= aspnetcore-3.0"

作者：[Kirk Larkin](https://github.com/serpent5)

本文介绍如何在 ASP.NET Core MVC 或页面应用中验证用户输入 Razor 。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/validation/samples)（[如何下载](xref:index#how-to-download-a-sample)）。

## <a name="model-state"></a>模型状态

模型状态表示两个子系统的错误：模型绑定和模型验证。 源自[模型绑定](model-binding.md)的错误通常是数据转换错误。 例如，在一个整数字段中输入一个“x”。 模型验证在模型绑定后发生，并报告数据不符合业务规则的错误。 例如，在需要 1 到 5 之间评分的字段中输入 0。

模型绑定和模型验证在执行控制器操作或 Razor 页面处理程序方法之前发生。 Web 应用负责检查 `ModelState.IsValid` 并做出相应响应。 Web 应用通常会重新显示带有错误消息的页面：

[!code-csharp[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml.cs?name=snippet_OnPostAsync&highlight=3-6)]

如果 Web API 控制器具有 `[ApiController]` 特性，则它们不必检查 `ModelState.IsValid`。 在此情况下，如果模型状态无效，将返回包含错误详细信息的自动 HTTP 400 响应。 有关详细信息，请参阅[自动 HTTP 400 响应](xref:web-api/index#automatic-http-400-responses)。

## <a name="rerun-validation"></a>重新运行验证

验证自动进行，但是可能需要手动进行重复验证。 例如，你可能为属性计算一个值，并且希望将属性设置为所计算的值后，再重新运行验证。 若要重新运行验证，请调用 `TryValidateModel` 方法，如下所示：

[!code-csharp[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml.cs?name=snippet_TryValidate&highlight=3-6)]

## <a name="validation-attributes"></a>验证特性

通过验证特性可以为模型属性指定验证规则。 示例应用的以下示例显示使用验证特性进行注释的模型类。 `[ClassicMovie]` 特性为自定义的验证特性，其他特性为内置的验证特性。 `[ClassicMovieWithClientValidator]` 未显示。 `[ClassicMovieWithClientValidator]` 显示实现自定义特性的另一种方法。

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/Movie.cs?name=snippet_Class)]

## <a name="built-in-attributes"></a>内置特性

以下是一些内置验证特性：

* `[CreditCard]`：验证属性是否具有信用卡格式。 需要 [JQuery 验证其他方法](https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.19.1/additional-methods.min.js)。
* `[Compare]`：验证模型中的两个属性是否匹配。
* `[EmailAddress]`：验证属性是否具有电子邮件格式。
* `[Phone]`：验证属性是否具有电话号码格式。
* `[Range]`：验证属性值是否在指定的范围内。
* `[RegularExpression]`：验证属性值是否与指定的正则表达式匹配。
* `[Required]`：验证字段是否不为 null。 有关此属性的行为的详细信息，请参阅[ `[Required]` 特性](#required-attribute)。
* `[StringLength]`：验证字符串属性值是否不超过指定长度限制。
* `[Url]`：验证属性是否具有 URL 格式。
* `[Remote]`：通过在服务器上调用操作方法来验证客户端上的输入。 有关此属性的行为的详细信息，请参阅[ `[Remote]` 特性](#remote-attribute)。

在 [System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) 命名空间中可找到验证特性的完整列表。

### <a name="error-messages"></a>错误消息

通过验证特性可以指定要为无效输入显示的错误消息。 例如： 。

```csharp
[StringLength(8, ErrorMessage = "Name length can't be more than 8.")]
```

在内部，特性使用用于字段名的某个占位符调用 `String.Format`，有时还使用额外占位符。 例如： 。

```csharp
[StringLength(8, ErrorMessage = "{0} length must be between {2} and {1}.", MinimumLength = 6)]
```

应用于 `Name` 属性时，上述代码创建的错误消息将为“名称长度必须介于 6 到 8 之间”。

若要查找为特定特性的错误消息而传递给 `String.Format` 的参数，请参阅 [DataAnnotations 源代码](https://github.com/dotnet/runtime/tree/master/src/libraries/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations)。

## <a name="required-attribute"></a>[必需] 特性

.NET Core 3.0 和更高版本中的验证系统将不可为 null 的参数或绑定属性视为具有 `[Required]` 特性。 `decimal` 和 `int` 等[值类型](/dotnet/csharp/language-reference/keywords/value-types)是不可为 null 的类型。 可以通过在 `Startup.ConfigureServices` 中配置 <xref:Microsoft.AspNetCore.Mvc.MvcOptions.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes> 来禁用此行为：

```csharp
services.AddControllers(options => options.SuppressImplicitRequiredAttributeForNonNullableReferenceTypes = true);
```

### <a name="required-validation-on-the-server"></a>[必需] 服务器上的验证

在服务器上，如果属性为 null，则认为所需值缺失。 不可为 null 的字段始终有效，并且从不显示 `[Required]` 属性的错误消息。

但是，不可为 null 的属性的模型绑定可能会失败，从而导致 `The value '' is invalid` 等错误消息。 若要为不可为 null 的类型的服务器端验证指定自定义错误消息，可使用以下选项：

* 将字段设置为可以为 null（例如，`decimal?`而不是 `decimal`）。 [可以 \<T> 为 null](/dotnet/csharp/programming-guide/nullable-types/)值类型的处理方式类似于标准的可为 null 的类型。
* 指定模型绑定要使用的默认错误消息，如以下示例所示：

  [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=5-6)]

  有关模型绑定错误（可以为其设置默认消息）的更多信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods>。

### <a name="required-validation-on-the-client"></a>[必需] 客户端上的验证

在客户端上处理不可为 null 类型和字符串的方式与在服务器上不同。 在客户端上：

* 只有在为值输入一个输入时，才认为该值存在。 因此，客户端验证处理不可为 null 类型的方式与处理可以为 null 类型的方式相同。
* jQuery 验证[必需](https://jqueryvalidation.org/required-method/)方法将字符串字段中的空格视为有效输入。 如果只输入空格，服务器端验证会将必需的字符串字段视为无效。

如前所述，将不可为 null 类型视为具有 `[Required]` 特性。 这意味着即使不应用 `[Required]` 特性，也可进行客户端验证。 但如果不使用该特性，将收到默认错误消息。 若要指定自定义错误消息，使用该特性。

## <a name="remote-attribute"></a>[远程] 特性

`[Remote]` 特性实现客户端验证，该验证需要在服务器上调用方法，以确定字段输入是否有效。 例如，应用可能需要验证用户名是否已在使用。

若要实现远程验证：

1. 创建可供 JavaScript 调用的操作方法。  JQuery 验证 [远程](https://jqueryvalidation.org/remote-method/) 方法需要 JSON 响应：

   * `true` 表示输入数据有效。
   * `false`、`undefined` 或 `null` 表示输入无效。 显示默认错误消息。
   * 任何其他字符串都表示输入无效。 将字符串显示为自定义错误消息。

   以下是返回自定义错误消息的操作方法示例：

   [!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyEmail)]

1. 在模型类中，使用指向验证操作方法的 `[Remote]` 特性注释属性，如下例所示：

   [!code-csharp[](validation/samples/3.x/ValidationSample/Models/User.cs?name=snippet_Email)]
 
   `[Remote]` 特性位于 `Microsoft.AspNetCore.Mvc` 命名空间中。
   
### <a name="additional-fields"></a>其他字段

通过 `[Remote]` 特性的 `AdditionalFields` 属性可以根据服务器上的数据验证字段组合。 例如，如果 `User` 模型具有 `FirstName` 和 `LastName` 属性，可能需要验证该名称对尚未被现有用户占用。 下面的示例演示如何使用 `AdditionalFields`：

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/User.cs?name=snippet_Name&highlight=1,5)]

`AdditionalFields` 可以显式设置为字符串 "FirstName" 和 "LastName"，但使用 [nameof](/dotnet/csharp/language-reference/keywords/nameof) 运算符可简化稍后的重构过程。 此验证的操作方法必须接受 `firstName` 和 `lastName` 参数：

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyName)]

用户输入名字或姓氏时，JavaScript 会进行远程调用，查看该名称对是否已占用。

若要验证两个或更多附加字段，可将其以逗号分隔列表形式提供。 例如，若要向模型中添加 `MiddleName` 属性，可按以下示例所示设置 `[Remote]` 特性：

```csharp
[Remote(action: "VerifyName", controller: "Users", AdditionalFields = nameof(FirstName) + "," + nameof(LastName))]
public string MiddleName { get; set; }
```

`AdditionalFields` 与所有属性参数一样，必须是常量表达式。 因此，请勿使用[内插字符串](/dotnet/csharp/language-reference/keywords/interpolated-strings)或调用 <xref:System.String.Join*> 来初始化 `AdditionalFields`。

## <a name="alternatives-to-built-in-attributes"></a>内置特性的替代特性

如果需要并非由内置属性提供的验证，可以：

* [创建自定义特性](#custom-attributes)。
* [实现 IValidatableObject](#ivalidatableobject)。

## <a name="custom-attributes"></a>自定义特性

对于内置验证特性无法处理的情况，可以创建自定义验证特性。 创建继承自 <xref:System.ComponentModel.DataAnnotations.ValidationAttribute> 的类，并替代 <xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*> 方法。

`IsValid` 方法接受名为 value 的对象，该对象是要进行验证的输入  。 重载还接受 `ValidationContext` 对象，该对象提供其他信息，例如模型绑定创建的模型实例。

以下示例验证“经典”流派电影的发行日期是否不晚于指定年份  。 `[ClassicMovie]` 属性：

* 仅在服务器上运行。
* 对于经典电影，验证发行日期：

[!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieAttribute.cs?name=snippet_Class)]

上述示例中的 `movie` 变量表示 `Movie` 对象，其中包含表单提交中的数据。 验证失败时，返回 `ValidationResult` 和错误消息。

## <a name="ivalidatableobject"></a>IValidatableObject

上述示例只适用于 `Movie` 类型。 类级别验证的另一方式是在模型类中实现 `IValidatableObject`，如下例所示：

[!code-csharp[](validation/samples/3.x/ValidationSample/Models/ValidatableMovie.cs?name=snippet_Class&highlight=1,26-34)]

## <a name="top-level-node-validation"></a>顶级节点验证

顶级节点包括：

* 操作参数
* 控制器属性
* 页处理程序参数
* 页模型属性

除了验证模型属性之外，还验证了模型绑定的顶级节点。 在示例应用的以下示例中，`VerifyPhone` 方法使用 <xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> 验证 `phone` 操作参数：

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyPhone)]

顶级节点可以将 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> 与验证属性结合使用。 在示例应用的以下示例中，`CheckAge` 方法指定在提交表单时必须从查询字符串绑定 `age` 参数：

[!code-csharp[](validation/samples/3.x/ValidationSample/Controllers/UsersController.cs?name=snippet_CheckAgeSignature)]

在“检查年限”页 (CheckAge.cshtml) 中，有两个表单  。 第一个表单将 `99` 的 `Age` 值作为查询字符串参数提交：`https://localhost:5001/Users/CheckAge?Age=99`。

当提交查询字符串中格式设置正确的 `age` 参数时，表单将进行验证。

“检查年限”页面上的第二个表单提交请求正文中的 `Age` 值，验证失败。 绑定失败，因为 `age` 参数必须来自查询字符串。

## <a name="maximum-errors"></a>最大错误数

达到最大错误数（默认为 200）时，验证停止。 可以使用 `Startup.ConfigureServices` 中的以下代码配置该数字：

[!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=4)]

## <a name="maximum-recursion"></a>最大递归次数

<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> 遍历所验证模型的对象图。 对于深度或无限递归的模型，验证可能会导致堆栈溢出。 [MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) 提供了在访问者递归超过配置深度时提前停止验证的方法。 `MvcOptions.MaxValidationDepth` 的默认值为 32。

## <a name="automatic-short-circuit"></a>自动短路

如果模型图不需要验证，验证将自动短路（跳过）。 运行时为其跳过验证的对象包括基元集合（如 `byte[]`、`string[]`、`Dictionary<string, string>`）和不具有任何验证器的复杂对象图。

## <a name="disable-validation"></a>禁用验证

若要禁用验证：

1. 创建不会将任何字段标记为无效的 `IObjectModelValidator` 实现。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/NullObjectModelValidator.cs?name=snippet_Class)]

1. 将以下代码添加到 `Startup.ConfigureServices`，以便替换依赖项注入容器中的默认 `IObjectModelValidator` 实现。

   [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_DisableValidation)]

你可能仍然会看到模型绑定的模型状态错误。

## <a name="client-side-validation"></a>客户端验证

客户端验证将阻止提交，直到表单变为有效为止。 “提交”按钮运行 JavaScript：要么提交表单要么显示错误消息。

表单上存在输入错误时，客户端验证会避免到服务器的不必要往返。 _Layout.cshtml 和 _ValidationScriptsPartial.cshtml 中的以下脚本引用支持客户端验证  ：

[!code-cshtml[](validation/samples/3.x/ValidationSample/Views/Shared/_Layout.cshtml?name=snippet_Scripts)]

[!code-cshtml[](validation/samples/3.x/ValidationSample/Views/Shared/_ValidationScriptsPartial.cshtml?name=snippet_Scripts)]

JQuery 非介入式 [验证](https://github.com/aspnet/jquery-validation-unobtrusive) 脚本是一种基于常用 [jQuery 验证](https://jqueryvalidation.org/) 插件构建的自定义 Microsoft 前端库。 如果没有 jQuery 非介入式验证，则必须在两个位置编码相同的验证逻辑：一次是在模型属性上的服务器端验证特性中，一次是在客户端脚本中。 [标记帮助程序](xref:mvc/views/tag-helpers/intro)和 [HTML 帮助程序](xref:mvc/views/overview)则使用模型属性中的验证特性和类型元数据，呈现需要验证的表单元素的 HTML 5 `data-` 特性。 jQuery 非介入式验证分析 `data-` 属性，并将逻辑传递到 JQuery 验证，从而有效地将服务器端验证逻辑 "复制" 到客户端。 可以使用标记帮助程序在客户端上显示验证错误，如下所示：

[!code-cshtml[](validation/samples/3.x/ValidationSample/Pages/Movies/Create.cshtml?name=snippet_ReleaseDate&highlight=3-4)]

上述标记帮助程序呈现以下 HTML：

```html
<div class="form-group">
    <label class="control-label" for="Movie_ReleaseDate">Release Date</label>
    <input class="form-control" type="date" data-val="true"
        data-val-required="The Release Date field is required."
        id="Movie_ReleaseDate" name="Movie.ReleaseDate" value="">
    <span class="text-danger field-validation-valid"
        data-valmsg-for="Movie.ReleaseDate" data-valmsg-replace="true"></span>
</div>
```

请注意，HTML 输出中的 `data-` 特性与 `Movie.ReleaseDate` 属性的验证特性相对应。 `data-val-required` 特性包含在用户未填写上映日期字段时将显示的错误消息。 jQuery 非介入式验证将此值传递给 [所需的 JQuery 验证 ( # B1](https://jqueryvalidation.org/required-method/) 方法，然后在随附的元素中显示该消息 **\<span>** 。

如果 `[DataType]` 特性未替代属性的 .NET 类型，则数据类型验证基于该类型。 浏览器具有自己的默认错误消息，但是 jQuery 验证非介入式验证包可以替代这些消息。 通过 `[DataType]` 特性和 `[EmailAddress]` 等子类可以指定错误消息。

## <a name="unobtrusive-validation"></a>非介入式验证

有关非介入式验证的信息，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/1111)。

### <a name="add-validation-to-dynamic-forms"></a>向动态表单添加验证

当首次加载页面时，jQuery 非引人注目验证会将验证逻辑和参数传递给 jQuery 验证。 因此，不会对动态生成的表单自动执行验证。 若要启用验证，指示 jQuery 非介入式验证在创建动态表单后立即对其进行分析。 例如，以下代码在通过 AJAX 添加的表单上设置客户端验证。

```javascript
$.get({
    url: "https://url/that/returns/a/form",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add form. " + errorThrown);
    },
    success: function(newFormHTML) {
        var container = document.getElementById("form-container");
        container.insertAdjacentHTML("beforeend", newFormHTML);
        var forms = container.getElementsByTagName("form");
        var newForm = forms[forms.length - 1];
        $.validator.unobtrusive.parse(newForm);
    }
})
```

`$.validator.unobtrusive.parse()` 方法采用 jQuery 选择器作为它的一个参数。 此方法指示 jQuery 非介入式验证分析该选择器内表单的 `data-` 属性。 然后，将这些属性的值传递给 jQuery 验证插件。

### <a name="add-validation-to-dynamic-controls"></a>向动态控件添加验证

`$.validator.unobtrusive.parse()` 方法适用于整个表单，而不是 `<input>` 和 `<select/>` 等单个动态生成的控件。 若要重新分析表单，删除之前分析表单时添加的验证数据，如下例所示：

```javascript
$.get({
    url: "https://url/that/returns/a/control",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add control. " + errorThrown);
    },
    success: function(newInputHTML) {
        var form = document.getElementById("my-form");
        form.insertAdjacentHTML("beforeend", newInputHTML);
        $(form).removeData("validator")    // Added by jQuery Validation
               .removeData("unobtrusiveValidation");   // Added by jQuery Unobtrusive Validation
        $.validator.unobtrusive.parse(form);
    }
})
```

## <a name="custom-client-side-validation"></a>自定义客户端验证

自定义客户端验证是通过生成 `data-` 适用于自定义 JQuery 验证适配器的 HTML 特性来完成的。 以下示例适配器代码是为本文前面部分介绍的 `[ClassicMovie]` 和 `[ClassicMovieWithClientValidator]` 特性编写的：

[!code-javascript[](validation/samples/3.x/ValidationSample/wwwroot/js/classicMovieValidator.js)]

有关如何编写适配器的信息，请参阅 [JQuery 验证文档](https://jqueryvalidation.org/documentation/)。

给定字段的适配器的使用由 `data-` 特性触发，这些特性：

* 将字段标记为正在验证 (`data-val="true"`)。
* 确定验证规则名称和错误消息文本（例如，`data-val-rulename="Error message."`）。
* 提供验证器所需的任何其他参数（例如，`data-val-rulename-param1="value"`）。

以下示例显示示例应用的 `ClassicMovie` 特性的 `data-` 特性：

```html
<input class="form-control" type="date"
    data-val="true"
    data-val-classicmovie="Classic movies must have a release year no later than 1960."
    data-val-classicmovie-year="1960"
    data-val-required="The Release Date field is required."
    id="Movie_ReleaseDate" name="Movie.ReleaseDate" value="">
```

如前所述，[标记帮助程序](xref:mvc/views/tag-helpers/intro)和 [HTML 帮助程序](xref:mvc/views/overview)使用验证特性的信息呈现 `data-` 特性。 编写用于创建自定义 `data-` HTML 特性的代码有以下两种方式：

* 创建派生自 `AttributeAdapterBase<TAttribute>` 的类和实现 `IValidationAttributeAdapterProvider` 的类，并在 DI 中注册特性及其适配器。 此方法遵循[单一责任主体](https://wikipedia.org/wiki/Single_responsibility_principle)，这体现在与服务器相关的验证代码和与客户端相关的验证代码位于不同的类中。 适配器还有以下优势：由于适配器是在 DI 中注册的，所以如果需要，适配器可以使用 DI 中的其他服务。
* 在 `ValidationAttribute` 类中实现 `IClientModelValidator`。 如果特性既不进行任何服务器端验证，也不需要 DI 的任何服务，则此方法可能适用。

### <a name="attributeadapter-for-client-side-validation"></a>用于客户端验证的 AttributeAdapter

在 HTML 中呈现 `data-` 特性的方法在示例应用中由 `ClassicMovie` 特性使用。 若要使用此方法添加客户端验证：

1. 为自定义验证特性创建特性适配器类。 从[AttributeAdapterBase \<T> ](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.attributeadapterbase-1?view=aspnetcore-2.2)派生类。 创建将 `data-` 特性添加到所呈现输出中的 `AddValidation` 方法，如下例所示：

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieAttributeAdapter.cs?name=snippet_Class)]

1. 创建实现 <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider> 的适配器提供程序类。 使用 `GetAttributeAdapter` 方法，将自定义属性传递给适配器的构造函数，如下例所示：

   [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/CustomValidationAttributeAdapterProvider.cs?name=snippet_Class)]

1. 在 `Startup.ConfigureServices` 中为 DI 注册适配器提供程序：

   [!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_Configuration&highlight=9-10)]

### <a name="iclientmodelvalidator-for-client-side-validation"></a>用于客户端验证的 IClientModelValidator

在 HTML 中呈现 `data-` 特性的方法在示例应用中由 `ClassicMovieWithClientValidator` 特性使用。 若要使用此方法添加客户端验证：

* 在自定义验证特性中，实现 `IClientModelValidator` 接口并创建 `AddValidation` 方法。 使用 `AddValidation` 方法，添加 `data-` 特性进行验证，如下例所示：

  [!code-csharp[](validation/samples/3.x/ValidationSample/Validation/ClassicMovieWithClientValidatorAttribute.cs?name=snippet_Class)]

## <a name="disable-client-side-validation"></a>禁用客户端验证

下面的代码在页中禁用客户端验证 Razor ：

[!code-csharp[](validation/samples/3.x/ValidationSample/Startup.cs?name=snippet_DisableClientValidation&highlight=2-5)]

可禁用客户端验证的其他选项：

* 在所有 *.cshtml* 文件中注释掉对 `_ValidationScriptsPartial` 的引用。
* 删除 *Pages\Shared\_ValidationScriptsPartial.cshtml* 文件的内容。

上述方法不会阻止对类库的客户端验证 ASP.NET Core Identity Razor 。 有关详细信息，请参阅 <xref:security/authentication/scaffold-identity>。

## <a name="additional-resources"></a>其他资源

* [System.ComponentModel.DataAnnotations 命名空间](xref:System.ComponentModel.DataAnnotations)
* [模型绑定](model-binding.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本文介绍如何在 ASP.NET Core MVC 或页面应用中验证用户输入 Razor 。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/validation/sample)（[如何下载](xref:index#how-to-download-a-sample)）。

## <a name="model-state"></a>模型状态

模型状态表示两个子系统的错误：模型绑定和模型验证。 [模型绑定](model-binding.md)的错误通常是数据转换错误（例如，在要求为整数的字段中输入“x”）。 模型验证在模型绑定之后进行，并在数据不符合业务规则时报告错误（例如，在要求评级为 1 至 5 之间的字段中输入 0）。

模型绑定和验证都在执行控制器操作或 Razor 页面处理程序方法之前发生。 Web 应用负责检查 `ModelState.IsValid` 并做出相应响应。 Web 应用通常会重新显示带有错误消息的页面：

[!code-csharp[](validation/samples_snapshot/2.x/Create.cshtml.cs?name=snippet&highlight=3-6)]

如果 Web API 控制器具有 `[ApiController]` 特性，则它们不必检查 `ModelState.IsValid`。 在此情况下，如果模型状态无效，将返回包含错误详细信息的自动 HTTP 400 响应。 有关详细信息，请参阅[自动 HTTP 400 响应](xref:web-api/index#automatic-http-400-responses)。

## <a name="rerun-validation"></a>重新运行验证

验证自动进行，但是可能需要手动进行重复验证。 例如，你可能为属性计算一个值，并且希望将属性设置为所计算的值后，再重新运行验证。 若要重新运行验证，请调用 `TryValidateModel` 方法，如下所示：

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/MoviesController.cs?name=snippet_TryValidateModel&highlight=11)]

## <a name="validation-attributes"></a>验证特性

通过验证特性可以为模型属性指定验证规则。 [示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/validation/sample)的以下示例显示使用验证特性进行注释的模型类。 `[ClassicMovie]` 特性为自定义的验证特性，其他特性为内置的验证特性。 `[ClassicMovie2]` 未显示，它表示实现自定义特性的另一种方法。

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/Movie.cs?name=snippet_ModelClass)]

## <a name="built-in-attributes"></a>内置特性

内置验证特性包括：

* `[CreditCard]`：验证属性是否具有信用卡格式。
* `[Compare]`：验证模型中的两个属性是否匹配。 例如， *Register.cshtml.cs* 文件使用 `[Compare]` 来验证输入的两个密码是否匹配。 [基架 Identity ](xref:security/authentication/scaffold-identity)查看注册代码。
* `[EmailAddress]`：验证属性是否具有电子邮件格式。
* `[Phone]`：验证属性是否具有电话号码格式。
* `[Range]`：验证属性值是否在指定的范围内。
* `[RegularExpression]`：验证属性值是否与指定的正则表达式匹配。
* `[Required]`：验证字段是否不为 null。 有关此属性的行为的详细信息，请参阅[ `[Required]` 特性](#required-attribute)。
* `[StringLength]`：验证字符串属性值是否不超过指定长度限制。
* `[Url]`：验证属性是否具有 URL 格式。
* `[Remote]`：通过在服务器上调用操作方法来验证客户端上的输入。 有关此属性的行为的详细信息，请参阅[ `[Remote]` 特性](#remote-attribute)。

将 `[RegularExpression]` 属性用于客户端验证时，在客户端上使用 JavaScript 执行正则表达式。 这意味着将使用 [ECMAScript](/dotnet/standard/base-types/regular-expression-options#ecmascript-matching-behavior) 匹配行为。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/corefx/issues/42487)。

在 [System.ComponentModel.DataAnnotations](xref:System.ComponentModel.DataAnnotations) 命名空间中可找到验证特性的完整列表。

### <a name="error-messages"></a>错误消息

通过验证特性可以指定要为无效输入显示的错误消息。 例如： 。

```csharp
[StringLength(8, ErrorMessage = "Name length can't be more than 8.")]
```

在内部，特性使用用于字段名的某个占位符调用 `String.Format`，有时还使用额外占位符。 例如： 。

```csharp
[StringLength(8, ErrorMessage = "{0} length must be between {2} and {1}.", MinimumLength = 6)]
```

应用于 `Name` 属性时，上述代码创建的错误消息将为“名称长度必须介于 6 到 8 之间”。

若要查找为特定特性的错误消息而传递给 `String.Format` 的参数，请参阅 [DataAnnotations 源代码](https://github.com/dotnet/corefx/tree/master/src/System.ComponentModel.Annotations/src/System/ComponentModel/DataAnnotations)。

## <a name="required-attribute"></a>[必需] 特性

默认情况下，验证系统将不可为 null 的参数或属性视为具有 `[Required]` 特性。 `decimal` 和 `int` 等[值类型](/dotnet/csharp/language-reference/keywords/value-types)是不可为 null 的类型。

### <a name="required-validation-on-the-server"></a>[必需] 服务器上的验证

在服务器上，如果属性为 null，则认为所需值缺失。 不可为 null 的字段始终有效，并且从不显示 [必需] 特性的错误消息。

但是，不可为 null 的属性的模型绑定可能会失败，从而导致 `The value '' is invalid` 等错误消息。 若要为不可为 null 的类型的服务器端验证指定自定义错误消息，可使用以下选项：

* 将字段设置为可以为 null（例如，`decimal?`而不是 `decimal`）。 [可以 \<T> 为 null](/dotnet/csharp/programming-guide/nullable-types/)值类型的处理方式类似于标准的可为 null 的类型。
* 指定模型绑定要使用的默认错误消息，如以下示例所示：

  [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=4-5)]

  有关模型绑定错误（可以为其设置默认消息）的更多信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.DefaultModelBindingMessageProvider#methods>。

### <a name="required-validation-on-the-client"></a>[必需] 客户端上的验证

在客户端上处理不可为 null 类型和字符串的方式与在服务器上不同。 在客户端上：

* 只有在为值输入一个输入时，才认为该值存在。 因此，客户端验证处理不可为 null 类型的方式与处理可以为 null 类型的方式相同。
* jQuery 验证[必需](https://jqueryvalidation.org/required-method/)方法将字符串字段中的空格视为有效输入。 如果只输入空格，服务器端验证会将必需的字符串字段视为无效。

如前所述，将不可为 null 类型视为具有 `[Required]` 特性。 这意味着即使不应用 `[Required]` 特性，也可进行客户端验证。 但如果不使用该特性，将收到默认错误消息。 若要指定自定义错误消息，使用该特性。

## <a name="remote-attribute"></a>[远程] 特性

`[Remote]` 特性实现客户端验证，该验证需要在服务器上调用方法，以确定字段输入是否有效。 例如，应用可能需要验证用户名是否已在使用。

若要实现远程验证：

1. 创建可供 JavaScript 调用的操作方法。  jQuery Validate [远程](https://jqueryvalidation.org/remote-method/)方法要求 JSON 响应：

   * `"true"` 表示输入数据有效。
   * `"false"`、`undefined` 或 `null` 表示输入无效。  显示默认错误消息。
   * 任何其他字符串都表示输入无效。 将字符串显示为自定义错误消息。

   以下是返回自定义错误消息的操作方法示例：

   [!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyEmail)]

1. 在模型类中，使用指向验证操作方法的 `[Remote]` 特性注释属性，如下例所示：

   [!code-csharp[](validation/samples/2.x/ValidationSample/Models/User.cs?name=snippet_UserEmailProperty)]
 
   `[Remote]` 特性位于 `Microsoft.AspNetCore.Mvc` 命名空间中。 如果未使用 `Microsoft.AspNetCore.App` 或 `Microsoft.AspNetCore.All` 元包，则请安装 [Microsoft.AspNetCore.Mvc.ViewFeatures](https://www.nuget.org/packages/Microsoft.AspNetCore.Mvc.ViewFeatures) NuGet 包。
   
### <a name="additional-fields"></a>其他字段

通过 `[Remote]` 特性的 `AdditionalFields` 属性可以根据服务器上的数据验证字段组合。 例如，如果 `User` 模型具有 `FirstName` 和 `LastName` 属性，可能需要验证该名称对尚未被现有用户占用。 下面的示例演示如何使用 `AdditionalFields`：

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/User.cs?name=snippet_UserNameProperties)]

`AdditionalFields`可以显式设置为字符串 `"FirstName"` 和 `"LastName"`，但使用 [nameof](/dotnet/csharp/language-reference/keywords/nameof) 运算符可简化稍后的重构过程。 此验证的操作方法必须接受 first name 和 last name 参数：

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyName)]

用户输入名字或姓氏时，JavaScript 会进行远程调用，查看该名称对是否已占用。

若要验证两个或更多附加字段，可将其以逗号分隔列表形式提供。 例如，若要向模型中添加 `MiddleName` 属性，可按以下示例所示设置 `[Remote]` 特性：

```csharp
[Remote(action: "VerifyName", controller: "Users", AdditionalFields = nameof(FirstName) + "," + nameof(LastName))]
public string MiddleName { get; set; }
```

`AdditionalFields` 与所有属性参数一样，必须是常量表达式。 因此，请勿使用[内插字符串](/dotnet/csharp/language-reference/keywords/interpolated-strings)或调用 <xref:System.String.Join*> 来初始化 `AdditionalFields`。

## <a name="alternatives-to-built-in-attributes"></a>内置特性的替代特性

如果需要并非由内置属性提供的验证，可以：

* [创建自定义特性](#custom-attributes)。
* [实现 IValidatableObject](#ivalidatableobject)。

## <a name="custom-attributes"></a>自定义特性

对于内置验证特性无法处理的情况，可以创建自定义验证特性。 创建继承自 <xref:System.ComponentModel.DataAnnotations.ValidationAttribute> 的类，并替代 <xref:System.ComponentModel.DataAnnotations.ValidationAttribute.IsValid*> 方法。

`IsValid` 方法接受名为 value 的对象，该对象是要进行验证的输入  。 重载还接受 `ValidationContext` 对象，该对象提供其他信息，例如模型绑定创建的模型实例。

以下示例验证“经典”流派电影的发行日期是否不晚于指定年份  。 `[ClassicMovie2]` 特性首先检查流派，只有流派为“经典”时才会继续检查  。 如果电影流派为经典，它会检查发行日期，确保该日期不晚于传递给特性构造函数的限值。）

[!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovieAttribute.cs?name=snippet_ClassicMovieAttribute)]

上述示例中的 `movie` 变量表示 `Movie` 对象，其中包含表单提交中的数据。 `IsValid` 方法检查日期和流派。 验证成功时，`IsValid` 返回 `ValidationResult.Success` 代码。 验证失败时，返回 `ValidationResult` 和错误消息。

## <a name="ivalidatableobject"></a>IValidatableObject

上述示例只适用于 `Movie` 类型。 类级别验证的另一方式是在模型类中实现 `IValidatableObject`，如下例所示：

[!code-csharp[](validation/samples/2.x/ValidationSample/Models/MovieIValidatable.cs?name=snippet&highlight=1,26-34)]

## <a name="top-level-node-validation"></a>顶级节点验证

顶级节点包括：

* 操作参数
* 控制器属性
* 页处理程序参数
* 页模型属性

除了验证模型属性之外，还验证了模型绑定的顶级节点。 在示例应用的以下示例中，`VerifyPhone` 方法使用 <xref:System.ComponentModel.DataAnnotations.RegularExpressionAttribute> 验证 `phone` 操作参数：

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_VerifyPhone)]

顶级节点可以将 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.BindRequiredAttribute> 与验证属性结合使用。 在示例应用的以下示例中，`CheckAge` 方法指定在提交表单时必须从查询字符串绑定 `age` 参数：

[!code-csharp[](validation/samples/2.x/ValidationSample/Controllers/UsersController.cs?name=snippet_CheckAge)]

在“检查年限”页 (CheckAge.cshtml) 中，有两个表单  。 第一个表单提交 `Age` 的值 `99` 作为查询字符串：`https://localhost:5001/Users/CheckAge?Age=99`。

当提交查询字符串中格式设置正确的 `age` 参数时，表单将进行验证。

“检查年限”页面上的第二个表单提交请求正文中的 `Age` 值，验证失败。 绑定失败，因为 `age` 参数必须来自查询字符串。

在 `CompatibilityVersion.Version_2_1` 或更高版本上运行时，默认启用顶级节点验证。 否则，禁用顶级节点验证。 通过在 (`Startup.ConfigureServices`) 中设置 <xref:Microsoft.AspNetCore.Mvc.MvcOptions.AllowValidatingTopLevelNodes*> 属性，可以替代默认选项，如下所示：

[!code-csharp[](validation/samples_snapshot/2.x/Startup.cs?name=snippet_AddMvc&highlight=4)]

## <a name="maximum-errors"></a>最大错误数

达到最大错误数（默认为 200）时，验证停止。 可以使用 `Startup.ConfigureServices` 中的以下代码配置该数字：

[!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=3)]

## <a name="maximum-recursion"></a>最大递归次数

<xref:Microsoft.AspNetCore.Mvc.ModelBinding.Validation.ValidationVisitor> 遍历所验证模型的对象图。 如果模型非常深或无限递归，验证可能导致堆栈溢出。 [MvcOptions.MaxValidationDepth](xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxValidationDepth) 提供了在访问者递归超过配置深度时提前停止验证的方法。 在 `CompatibilityVersion.Version_2_2` 或更高版本上运行时，`MvcOptions.MaxValidationDepth` 的默认值为 32。 对于更低版本，该值为 null，这表示没有深度约束。

## <a name="automatic-short-circuit"></a>自动短路

如果模型图不需要验证，验证将自动短路（跳过）。 运行时为其跳过验证的对象包括基元集合（如 `byte[]`、`string[]`、`Dictionary<string, string>`）和不具有任何验证器的复杂对象图。

## <a name="disable-validation"></a>禁用验证

若要禁用验证：

1. 创建不会将任何字段标记为无效的 `IObjectModelValidator` 实现。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/NullObjectModelValidator.cs?name=snippet_DisableValidation)]

1. 将以下代码添加到 `Startup.ConfigureServices`，以便替换依赖项注入容器中的默认 `IObjectModelValidator` 实现。

   [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_DisableValidation)]

你可能仍然会看到模型绑定的模型状态错误。

## <a name="client-side-validation"></a>客户端验证

客户端验证将阻止提交，直到表单变为有效为止。 “提交”按钮运行 JavaScript：要么提交表单要么显示错误消息。

表单上存在输入错误时，客户端验证会避免到服务器的不必要往返。 _Layout.cshtml 和 _ValidationScriptsPartial.cshtml 中的以下脚本引用支持客户端验证  ：

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Shared/_Layout.cshtml?name=snippet_ScriptTag)]

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Shared/_ValidationScriptsPartial.cshtml?name=snippet_ScriptTags)]

[jQuery 非介入式验证](https://github.com/aspnet/jquery-validation-unobtrusive)脚本是一个基于热门 [jQuery Validate](https://jqueryvalidation.org/) 插件的自定义 Microsoft 前端库。 如果没有 jQuery 非介入式验证，则必须在两个位置编码相同的验证逻辑：一次是在模型属性上的服务器端验证特性中，一次是在客户端脚本中。 [标记帮助程序](xref:mvc/views/tag-helpers/intro)和 [HTML 帮助程序](xref:mvc/views/overview)则使用模型属性中的验证特性和类型元数据，呈现需要验证的表单元素的 HTML 5 `data-` 特性。 jQuery 非介入式验证分析 `data-` 特性并将逻辑传递给 jQuery Validate，从而将服务器端验证逻辑有效地“复制”到客户端。 可以使用标记帮助程序在客户端上显示验证错误，如下所示：

[!code-cshtml[](validation/samples/2.x/ValidationSample/Views/Movies/Create.cshtml?name=snippet_ReleaseDate&highlight=4-5)]

上述标记帮助程序呈现以下 HTML。

```html
<form action="/Movies/Create" method="post">
    <div class="form-horizontal">
        <h4>Movie</h4>
        <div class="text-danger"></div>
        <div class="form-group">
            <label class="col-md-2 control-label" for="ReleaseDate">ReleaseDate</label>
            <div class="col-md-10">
                <input class="form-control" type="datetime"
                data-val="true" data-val-required="The ReleaseDate field is required."
                id="ReleaseDate" name="ReleaseDate" value="">
                <span class="text-danger field-validation-valid"
                data-valmsg-for="ReleaseDate" data-valmsg-replace="true"></span>
            </div>
        </div>
    </div>
</form>
```

请注意，HTML 输出中的 `data-` 特性与 `ReleaseDate` 属性的验证特性相对应。 `data-val-required` 特性包含在用户未填写上映日期字段时将显示的错误消息。 jQuery 非介入式验证将此值传递给 [所需的 JQuery 验证 ( # B1](https://jqueryvalidation.org/required-method/) 方法，然后在随附的元素中显示该消息 **\<span>** 。

如果 `[DataType]` 特性未替代属性的 .NET 类型，则数据类型验证基于该类型。 浏览器具有自己的默认错误消息，但是 jQuery 验证非介入式验证包可以替代这些消息。 通过 `[DataType]` 特性和 `[EmailAddress]` 等子类可以指定错误消息。

### <a name="add-validation-to-dynamic-forms"></a>向动态表单添加验证

jQuery 非介入式验证会在第一次加载页面时将验证逻辑和参数传递到 jQuery Validate。 因此，不会对动态生成的表单自动执行验证。 若要启用验证，指示 jQuery 非介入式验证在创建动态表单后立即对其进行分析。 例如，以下代码在通过 AJAX 添加的表单上设置客户端验证。

```javascript
$.get({
    url: "https://url/that/returns/a/form",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add form. " + errorThrown);
    },
    success: function(newFormHTML) {
        var container = document.getElementById("form-container");
        container.insertAdjacentHTML("beforeend", newFormHTML);
        var forms = container.getElementsByTagName("form");
        var newForm = forms[forms.length - 1];
        $.validator.unobtrusive.parse(newForm);
    }
})
```

`$.validator.unobtrusive.parse()` 方法采用 jQuery 选择器作为它的一个参数。 此方法指示 jQuery 非介入式验证分析该选择器内表单的 `data-` 属性。 然后将这些特性的值传递给 jQuery Validate 插件。

### <a name="add-validation-to-dynamic-controls"></a>向动态控件添加验证

`$.validator.unobtrusive.parse()` 方法适用于整个表单，而不是 `<input>` 和 `<select/>` 等单个动态生成的控件。 若要重新分析表单，删除之前分析表单时添加的验证数据，如下例所示：

```javascript
$.get({
    url: "https://url/that/returns/a/control",
    dataType: "html",
    error: function(jqXHR, textStatus, errorThrown) {
        alert(textStatus + ": Couldn't add control. " + errorThrown);
    },
    success: function(newInputHTML) {
        var form = document.getElementById("my-form");
        form.insertAdjacentHTML("beforeend", newInputHTML);
        $(form).removeData("validator")    // Added by jQuery Validate
               .removeData("unobtrusiveValidation");   // Added by jQuery Unobtrusive Validation
        $.validator.unobtrusive.parse(form);
    }
})
```

## <a name="custom-client-side-validation"></a>自定义客户端验证

生成适用于自定义 jQuery Validate 适配器的 `data-` HTML 特性，完成自定义客户端验证。 以下示例适配器代码是为本文前面部分介绍的 `ClassicMovie` 和 `ClassicMovie2` 特性编写的：

[!code-javascript[](validation/samples/2.x/ValidationSample/wwwroot/js/classicMovieValidator.js?name=snippet_UnobtrusiveValidation)]

有关如何编写适配器的信息，请参阅 [jQuery Validate 文档](https://jqueryvalidation.org/documentation/)。

给定字段的适配器的使用由 `data-` 特性触发，这些特性：

* 将字段标记为正在验证 (`data-val="true"`)。
* 确定验证规则名称和错误消息文本（例如，`data-val-rulename="Error message."`）。
* 提供验证器所需的任何其他参数（例如，`data-val-rulename-parm1="value"`）。

以下示例显示示例应用的 `ClassicMovie` 特性的 `data-` 特性：

```html
<input class="form-control" type="datetime"
    data-val="true"
    data-val-classicmovie1="Classic movies must have a release year earlier than 1960."
    data-val-classicmovie1-year="1960"
    data-val-required="The ReleaseDate field is required."
    id="ReleaseDate" name="ReleaseDate" value="">
```

如前所述，[标记帮助程序](xref:mvc/views/tag-helpers/intro)和 [HTML 帮助程序](xref:mvc/views/overview)使用验证特性的信息呈现 `data-` 特性。 编写用于创建自定义 `data-` HTML 特性的代码有以下两种方式：

* 创建派生自 `AttributeAdapterBase<TAttribute>` 的类和实现 `IValidationAttributeAdapterProvider` 的类，并在 DI 中注册特性及其适配器。 此方法遵循[单一责任主体](https://wikipedia.org/wiki/Single_responsibility_principle)，这体现在与服务器相关的验证代码和与客户端相关的验证代码位于不同的类中。 适配器还有以下优势：由于适配器是在 DI 中注册的，所以如果需要，适配器可以使用 DI 中的其他服务。
* 在 `ValidationAttribute` 类中实现 `IClientModelValidator`。 如果特性既不进行任何服务器端验证，也不需要 DI 的任何服务，则此方法可能适用。

### <a name="attributeadapter-for-client-side-validation"></a>用于客户端验证的 AttributeAdapter

在 HTML 中呈现 `data-` 特性的方法在示例应用中由 `ClassicMovie` 特性使用。 若要使用此方法添加客户端验证：

1. 为自定义验证特性创建特性适配器类。 从[AttributeAdapterBase \<T> ](/dotnet/api/microsoft.aspnetcore.mvc.dataannotations.attributeadapterbase-1?view=aspnetcore-2.2)派生类。 创建将 `data-` 特性添加到所呈现输出中的 `AddValidation` 方法，如下例所示：

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovieAttributeAdapter.cs?name=snippet_ClassicMovieAttributeAdapter)]

1. 创建实现 <xref:Microsoft.AspNetCore.Mvc.DataAnnotations.IValidationAttributeAdapterProvider> 的适配器提供程序类。 使用 `GetAttributeAdapter` 方法，将自定义属性传递给适配器的构造函数，如下例所示：

   [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/CustomValidationAttributeAdapterProvider.cs?name=snippet_CustomValidationAttributeAdapterProvider)]

1. 在 `Startup.ConfigureServices` 中为 DI 注册适配器提供程序：

   [!code-csharp[](validation/samples/2.x/ValidationSample/Startup.cs?name=snippet_MaxModelValidationErrors&highlight=8-10)]

### <a name="iclientmodelvalidator-for-client-side-validation"></a>用于客户端验证的 IClientModelValidator

在 HTML 中呈现 `data-` 特性的方法在示例应用中由 `ClassicMovie2` 特性使用。 若要使用此方法添加客户端验证：

* 在自定义验证特性中，实现 `IClientModelValidator` 接口并创建 `AddValidation` 方法。 使用 `AddValidation` 方法，添加 `data-` 特性进行验证，如下例所示：

  [!code-csharp[](validation/samples/2.x/ValidationSample/Attributes/ClassicMovie2Attribute.cs?name=snippet_ClassicMovie2Attribute)]

## <a name="disable-client-side-validation"></a>禁用客户端验证

以下代码禁用 MVC 视图中的客户端验证：

[!code-csharp[](validation/samples_snapshot/2.x/Startup2.cs?name=snippet_DisableClientValidation)]

Razor页面：

[!code-csharp[](validation/samples_snapshot/2.x/Startup3.cs?name=snippet_DisableClientValidation)]

禁用客户端验证的另一方式是在 .cshtml 文件中为 `_ValidationScriptsPartial` 的引用添加注释  。

## <a name="additional-resources"></a>其他资源

* [System.ComponentModel.DataAnnotations 命名空间](xref:System.ComponentModel.DataAnnotations)
* [模型绑定](model-binding.md)

::: moniker-end
