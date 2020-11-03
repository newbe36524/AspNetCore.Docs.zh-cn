---
title: ASP.NET Core 中的模型绑定
author: rick-anderson
description: 了解 ASP.NET Core 中模型绑定的工作原理以及如何自定义模型绑定的行为。
ms.assetid: 0be164aa-1d72-4192-bd6b-192c9c301164
ms.author: riande
ms.date: 12/18/2019
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
uid: mvc/models/model-binding
ms.openlocfilehash: a3be22134246c76b0a809ddb97b33ff97ace9a5b
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93057500"
---
# <a name="model-binding-in-aspnet-core"></a>ASP.NET Core 中的模型绑定

::: moniker range=">= aspnetcore-3.0"

本文解释了模型绑定的定义、模型绑定的工作原理，以及如何自定义模型绑定的行为。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/model-binding/samples)（[如何下载](xref:index#how-to-download-a-sample)）。

## <a name="what-is-model-binding"></a>什么是模型绑定

控制器和 Razor 页面处理来自 HTTP 请求的数据。 例如，路由数据可以提供一个记录键，而发布的表单域可以为模型的属性提供一个值。 编写代码以检索这些值，并将其从字符串转换为 .NET 类型不仅繁琐，而且还容易出错。 模型绑定会自动化该过程。 模型绑定系统：

* 从各种源（如路由数据、表单域和查询字符串）中检索数据。
* Razor在方法参数和公共属性中向控制器和页面提供数据。
* 将字符串数据转换为 .NET 类型。
* 更新复杂类型的属性。

## <a name="example"></a>示例

假设有以下操作方法：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Controllers/PetsController.cs?name=snippet_DogsOnly)]

并且应用收到一个带有以下 URL 的请求：

```
http://contoso.com/api/pets/2?DogsOnly=true
```

在路由系统选择该操作方法之后，模型绑定执行以下步骤：

* 查找 `GetByID` 的第一个参数，该参数是一个名为 `id` 的整数。
* 查找 HTTP 请求中的可用源，并在路由数据中查找 `id` =“2”。
* 将字符串“2”转换为整数 2。
* 查找 `GetByID` 的下一个参数，该参数是一个名为 `dogsOnly` 的布尔值。
* 查找源，并在查询字符串中查找“DogsOnly=true”。 名称匹配不区分大小写。
* 将字符串“true”转换为布尔值 `true`。

然后，该框架会调用 `GetById` 方法，为 `id` 参数传入 2，并为 `dogsOnly` 参数传入 `true`。

在前面的示例中，模型绑定目标是简单类型的方法参数。 目标也可以是复杂类型的属性。 成功绑定每个属性后，将对属性进行[模型验证](xref:mvc/models/validation)。 有关绑定到模型的数据以及任意绑定或验证错误的记录都存储在 [ControllerBase.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) 或 [PageModel.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) 中。 为查明该过程是否已成功，应用会检查 [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) 标志。

## <a name="targets"></a>目标

模型绑定尝试查找以下类型目标的值：

* 将请求路由到的控制器操作方法的参数。
* Razor请求路由到的页处理程序方法的参数。 
* 控制器或 `PageModel` 类的公共属性（若由特性指定）。

### <a name="bindproperty-attribute"></a>[BindProperty] 属性

可应用于控制器或 `PageModel` 类的公共属性，从而使模型绑定以该属性为目标：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Edit.cshtml.cs?name=snippet_BindProperty&highlight=3-4)]

### <a name="bindproperties-attribute"></a>[BindProperties] 属性

可在 ASP.NET Core 2.1 及更高版本中获得。  可应用于控制器或 `PageModel` 类，以使模型绑定以该类的所有公共属性为目标：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_BindProperties&highlight=1-2)]

### <a name="model-binding-for-http-get-requests"></a>HTTP GET 请求的模型绑定

默认情况下，不绑定 HTTP GET 请求的属性。 通常，GET 请求只需一个记录 ID 参数。 记录 ID 用于查找数据库中的项。 因此，无需绑定包含模型实例的属性。 在需要将属性绑定到 GET 请求中的数据的情况下，请将 `SupportsGet` 属性设置为 `true`：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_SupportsGet)]

## <a name="sources"></a>源

默认情况下，模型绑定以键值对的形式从 HTTP 请求中的以下源中获取数据：

1. 表单域
1. 请求正文（对于[具有 [ApiController] 属性的控制器](xref:web-api/index#binding-source-parameter-inference)。）
1. 路由数据
1. 查询字符串参数
1. 上传的文件

对于每个目标参数或属性，按照之前列表中指示的顺序扫描源。 有几个例外情况：

* 路由数据和查询字符串值仅用于简单类型。
* 上传的文件仅绑定到实现 `IFormFile` 或 `IEnumerable<IFormFile>` 的目标类型。

如果默认源不正确，请使用下列属性之一来指定源：

* [`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute) -从查询字符串获取值。 
* [`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute) -从路由数据中获取值。
* [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) -从已发布的表单字段中获取值。
* [`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute) -从请求正文中获取值。
* [`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) -从 HTTP 标头中获取值。

这些属性：

* 分别添加到模型属性（而不是模型类），如以下示例所示：

  [!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/Instructor.cs?name=snippet_FromQuery&highlight=5-6)]

* 选择性地在构造函数中接受模型名称值。 提供此选项的目的是应对属性名称与请求中的值不匹配的情况。 例如，请求中的值可能是名称中带有连字符的标头，如以下示例所示：

  [!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_FromHeader)]

### <a name="frombody-attribute"></a>[FromBody] 属性

将 `[FromBody]` 特性应用于一个参数，以便从一个 HTTP 请求的正文填充其属性。 ASP.NET Core 运行时将读取正文的责任委托给输入格式化程序。 输入格式化程序的解释位于[本文后面部分](#input-formatters)。

将 `[FromBody]` 应用于复杂类型参数时，应用于其属性的任何绑定源属性都将被忽略。 例如，以下 `Create` 操作指定从正文填充其 `pet` 参数：

```csharp
public ActionResult<Pet> Create([FromBody] Pet pet)
```

`Pet` 类指定从查询字符串参数填充其 `Breed` 属性：

```csharp
public class Pet
{
    public string Name { get; set; }

    [FromQuery] // Attribute is ignored.
    public string Breed { get; set; }
}
```

在上面的示例中：

* `[FromQuery]` 特性被忽略。
* `Breed` 属性未从查询字符串参数进行填充。 

输入格式化程序只读取正文，不了解绑定源特性。 如果在正文中找到合适的值，则使用该值填充 `Breed` 属性。

不要将 `[FromBody]` 应用于每个操作方法的多个参数。 输入格式化程序读取请求流后，无法再次读取该流以绑定其他 `[FromBody]` 参数。

### <a name="additional-sources"></a>其他源

源数据由“值提供程序”提供给模型绑定系统  。 你可以编写并注册自定义值提供程序，这些提供程序从其他源中获取用于模型绑定的数据。 例如，你可能需要来自 cookie 或会话状态的数据。 要从新的源中获取数据，请执行以下操作：

* 创建用于实现 `IValueProvider` 的类。
* 创建用于实现 `IValueProviderFactory` 的类。
* 在 `Startup.ConfigureServices` 中注册工厂类。

该示例应用包含一个 [值提供程序](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/models/model-binding/samples/3.x/ModelBindingSample/CookieValueProvider.cs) 和一个 [工厂](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/models/model-binding/samples/3.x/ModelBindingSample/CookieValueProviderFactory.cs) 示例，可从 s 中获取值 cookie 。 以下是 `Startup.ConfigureServices` 中的注册代码：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=4)]

所示代码将自定义值提供程序置于所有内置值提供程序之后。  要将其置于列表中的首位，请调用 `Insert(0, new CookieValueProviderFactory())` 而不是 `Add`。

## <a name="no-source-for-a-model-property"></a>不存在模型属性的源

默认情况下，如果找不到模型属性的值，则不会创建模型状态错误。 该属性设置为 NULL 或默认值：

* 可以为 Null 的简单类型设置为 `null`。
* 不可以为 Null 的值类型设置为 `default(T)`。 例如，参数 `int id` 设置为 0。
* 对于复杂类型，模型绑定使用默认构造函数来创建实例，而不设置属性。
* 数组设置为 `Array.Empty<T>()`，但 `byte[]` 数组设置为 `null`。

如果在模型属性的窗体字段中未找到任何内容时模型状态应失效，请使用 [`[BindRequired]`](#bindrequired-attribute) 特性。

请注意，此 `[BindRequired]` 行为适用于发布的表单数据中的模型绑定，而不适用于请求正文中的 JSON 或 XML 数据。 请求正文数据由[输入格式化程序](#input-formatters)进行处理。

## <a name="type-conversion-errors"></a>类型转换错误

如果找到源，但无法将其转换为目标类型，则模型状态将被标记为无效。 目标参数或属性设置为 NULL 或默认值，如上一部分所述。

在具有 `[ApiController]` 属性的 API 控制器中，无效的模型状态会导致自动 HTTP 400 响应。

在 Razor 页面中，重新显示页面并显示一条错误消息：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_HandleMBError&highlight=3-6)]

客户端验证将捕获大多数提交到 Razor 页面窗体的错误数据。 此验证使得先前突出显示的代码难以被触发。 示例应用包含一个“提交无效日期”按钮，该按钮将错误数据置于“雇用日期”字段中并提交表单  。 此按钮显示在发生数据转换错误时用于重新显示页的代码将如何工作。

在使用先前的代码重新显示页时，表单域中不会显示无效的输入。 这是因为模型属性已设置为 NULL 或默认值。 无效输入会出现在错误消息中。 但是，如果要在表单域中重新显示错误数据，可以考虑将模型属性设置为字符串并手动执行数据转换。

如果不希望发生类型转换错误导致模型状态错误的情况，建议使用相同的策略。 在这种情况下，将模型属性设置为字符串。

## <a name="simple-types"></a>简单类型

模型绑定器可以将源字符串转换为以下简单类型：

* [布尔值](xref:System.ComponentModel.BooleanConverter)
* [字节](xref:System.ComponentModel.ByteConverter)、[SByte](xref:System.ComponentModel.SByteConverter)
* [Char](xref:System.ComponentModel.CharConverter)
* [DateTime](xref:System.ComponentModel.DateTimeConverter)
* [DateTimeOffset](xref:System.ComponentModel.DateTimeOffsetConverter)
* [小数](xref:System.ComponentModel.DecimalConverter)
* [双精度](xref:System.ComponentModel.DoubleConverter)
* [枚举](xref:System.ComponentModel.EnumConverter)
* [Guid.empty](xref:System.ComponentModel.GuidConverter)
* [Int16](xref:System.ComponentModel.Int16Converter)、[Int32](xref:System.ComponentModel.Int32Converter)、[Int64](xref:System.ComponentModel.Int64Converter)
* [单精度](xref:System.ComponentModel.SingleConverter)
* [TimeSpan](xref:System.ComponentModel.TimeSpanConverter)
* [UInt16](xref:System.ComponentModel.UInt16Converter)、[UInt32](xref:System.ComponentModel.UInt32Converter)、[UInt64](xref:System.ComponentModel.UInt64Converter)
* [Uri](xref:System.UriTypeConverter)
* [版本](xref:System.ComponentModel.VersionConverter)

## <a name="complex-types"></a>复杂类型

复杂类型必须具有要绑定的公共默认构造函数和公共可写属性。 进行模型绑定时，将使用公共默认构造函数来实例化类。 

对于复杂类型的每个属性，模型绑定会查找名称模式 prefix.property_name 的源  。 如果未找到，它将仅查找不含前缀的 properties_name  。

对于绑定到参数，前缀是参数名称。 对于绑定到 `PageModel` 公共属性，前缀是公共属性名称。 某些属性具有 `Prefix` 属性，让你可以替代参数或属性名称的默认用法。

例如，假设复杂类型是以下 `Instructor` 类：

  ```csharp
  public class Instructor
  {
      public int ID { get; set; }
      public string LastName { get; set; }
      public string FirstName { get; set; }
  }
  ```

### <a name="prefix--parameter-name"></a>前缀 = 参数名称

如果要绑定的模型是一个名为 `instructorToUpdate` 的参数：

```csharp
public IActionResult OnPost(int? id, Instructor instructorToUpdate)
```

模型绑定从查找键 `instructorToUpdate.ID` 的源开始操作。 如果未找到，它将查找不含前缀的 `ID`。

### <a name="prefix--property-name"></a>前缀 = 属性名称

如果要绑定的模型是控制器或 `PageModel` 类的一个名为 `Instructor` 的属性：

```csharp
[BindProperty]
public Instructor Instructor { get; set; }
```

模型绑定从查找键 `Instructor.ID` 的源开始操作。 如果未找到，它将查找不含前缀的 `ID`。

### <a name="custom-prefix"></a>自定义前缀

如果要绑定的模型是名为 `instructorToUpdate` 的参数，并且 `Bind` 属性指定 `Instructor` 作为前缀：

```csharp
public IActionResult OnPost(
    int? id, [Bind(Prefix = "Instructor")] Instructor instructorToUpdate)
```

模型绑定从查找键 `Instructor.ID` 的源开始操作。 如果未找到，它将查找不含前缀的 `ID`。

### <a name="attributes-for-complex-type-targets"></a>复杂类型目标的属性

多个内置属性可用于控制复杂类型的模型绑定：

* `[Bind]`
* `[BindRequired]`
* `[BindNever]`

> [!WARNING]
> 如果发布的表单数据是值的源，则这些属性会影响模型绑定。 它们 **不会** 影响输入格式化程序，后者处理已发布的 JSON 和 XML 请求正文。 输入格式化程序的解释位于[本文后面部分](#input-formatters)。

### <a name="bind-attribute"></a>[Bind] 属性

可应用于类或方法参数。 指定模型绑定中应包含的模型属性。 `[Bind]` 不 _*_影响输入_*_ 格式化程序。

在下面的示例中，当调用任意处理程序或操作方法时，只绑定 `Instructor` 模型的指定属性：

```csharp
[Bind("LastName,FirstMidName,HireDate")]
public class Instructor
```

在下面的示例中，当调用 `OnPost` 方法时，只绑定 `Instructor` 模型的指定属性：

```csharp
[HttpPost]
public IActionResult OnPost([Bind("LastName,FirstMidName,HireDate")] Instructor instructor)
```

`[Bind]`特性可用于防范 _create * 方案中的过多发布。 由于排除的属性设置为 NULL 或默认值，而不是保持不变，因此它在编辑方案中无法很好地工作。 为防止过多发布，建议使用视图模型，而不是使用 `[Bind]` 属性。 有关详细信息，请参阅[有关过多发布的安全性说明](xref:data/ef-mvc/crud#security-note-about-overposting)。

### <a name="bindrequired-attribute"></a>[BindRequired] 属性

只能应用于模型属性，不能应用于方法参数。 如果无法对模型属性进行绑定，则会导致模型绑定添加模型状态错误。 以下是一个示例：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/InstructorWithCollection.cs?name=snippet_BindRequired&highlight=8-9)]

另请参阅[模型验证](xref:mvc/models/validation#required-attribute)中针对 `[Required]` 属性的讨论。

### <a name="bindnever-attribute"></a>[BindNever] 属性

只能应用于模型属性，不能应用于方法参数。 防止模型绑定设置模型的属性。 以下是一个示例：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/InstructorWithDictionary.cs?name=snippet_BindNever&highlight=3-4)]

## <a name="collections"></a>集合

对于是简单类型集合的目标，模型绑定将查找 parameter_name 或 property_name 的匹配项  。 如果找不到匹配项，它将查找某种不含前缀的受支持的格式。 例如： 。

* 假设要绑定的参数是名为 `selectedCourses` 的数组：

  ```csharp
  public IActionResult OnPost(int? id, int[] selectedCourses)
  ```

* 表单或查询字符串数据可以采用以下某种格式：
   
  ```
  selectedCourses=1050&selectedCourses=2000 
  ```

  ```
  selectedCourses[0]=1050&selectedCourses[1]=2000
  ```

  ```
  [0]=1050&[1]=2000
  ```

  ```
  selectedCourses[a]=1050&selectedCourses[b]=2000&selectedCourses.index=a&selectedCourses.index=b
  ```

  ```
  [a]=1050&[b]=2000&index=a&index=b
  ```

* 只有表单数据支持以下格式：

  ```
  selectedCourses[]=1050&selectedCourses[]=2000
  ```

* 对于前面所有的示例格式，模型绑定将两个项的数组传递给 `selectedCourses` 参数：

  * selectedCourses[0]=1050
  * selectedCourses[1]=2000

  使用下标数字的数据格式 (... [0] ... [1] ...) 必须确保从零开始按顺序进行编号。 如果下标编号中存在任何间隔，则间隔后的所有项都将被忽略。 例如，如果下标是 0 和 2，而不是 0 和 1，则第二个项会被忽略。

## <a name="dictionaries"></a>字典

对于 `Dictionary` 目标，模型绑定会查找 parameter_name 或 property_name 的匹配项  。 如果找不到匹配项，它将查找某种不含前缀的受支持的格式。 例如： 。

* 假设目标参数是名为 `selectedCourses` 的 `Dictionary<int, string>`：

  ```csharp
  public IActionResult OnPost(int? id, Dictionary<int, string> selectedCourses)
  ```

* 发布的表单或查询字符串数据可以类似于以下某一示例：

  ```
  selectedCourses[1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  [1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  selectedCourses[0].Key=1050&selectedCourses[0].Value=Chemistry&
  selectedCourses[1].Key=2000&selectedCourses[1].Value=Economics
  ```

  ```
  [0].Key=1050&[0].Value=Chemistry&[1].Key=2000&[1].Value=Economics
  ```

* 对于前面所有的示例格式，模型绑定将两个项的字典传递给 `selectedCourses` 参数：

  * selectedCourses["1050"]="Chemistry"
  * selectedCourses["2000"]="Economics"
  
::: moniker-end

::: moniker range=">= aspnetcore-5.0"

## <a name="constructor-binding-and-record-types"></a>构造函数绑定和记录类型

模型绑定要求复杂类型具有无参数的构造函数。 `System.Text.Json`和 `Newtonsoft.Json` 基于的输入格式化程序都支持对不具有无参数构造函数的类进行反序列化。 

C # 9 引入了记录类型，这是一个很好的方法，可以在网络上简洁地表示数据。 ASP.NET Core 增加了对模型绑定和使用单个构造函数验证记录类型的支持：

```csharp
public record Person([Required] string Name, [Range(0, 150)] int Age);

public class PersonController
{
   public IActionResult Index() => View();

   [HttpPost]
   public IActionResult Index(Person person)
   {
       ...
   }
}
```

`Person/Index.cshtml`:

```cshtml
@model Person

Name: <input asp-for="Name" />
...
Age: <input asp-for="Age" />
```

在验证记录类型时，运行时将搜索专用于参数而不是属性的验证元数据。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

<a name="glob"></a>

## <a name="globalization-behavior-of-model-binding-route-data-and-query-strings"></a>模型绑定路由数据和查询字符串的全球化行为

ASP.NET Core 路由值提供程序和查询字符串值提供程序：

* 将值视为固定区域性。
* URL 的区域性应固定。

相反，来自窗体数据的值要进行区分区域性的转换。 这是设计使然，目的是让 URL 可在各个区域设置中共享。

使 ASP.NET Core 路由值提供程序和查询字符串值提供程序进行区分区域性的转换：

* 继承自 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.IValueProviderFactory>
* 从 [QueryStringValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs) 或 [RouteValueValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/ModelBinding/RouteValueProviderFactory.cs) 复制代码
* 使用 [CultureInfo.CurrentCulture](xref:System.Globalization.CultureInfo.CurrentCulture) 替换传递给值提供程序构造函数的[区域性值](https://github.com/dotnet/AspNetCore/blob/e625fe29b049c60242e8048b4ea743cca65aa7b5/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs#L30)
* 将 MVC 选项中的默认值提供程序工厂替换为新的工厂：

[!code-csharp[](model-binding/samples_snapshot/3.x/Startup.cs?name=snippet)]
[!code-csharp[](model-binding/samples_snapshot/3.x/Startup.cs?name=snippet1)]

## <a name="special-data-types"></a>特殊数据类型

模型绑定可以处理某些特殊的数据类型。

### <a name="iformfile-and-iformfilecollection"></a>IFormFile 和 IFormFileCollection

HTTP 请求中包含的上传文件。  还支持多个文件的 `IEnumerable<IFormFile>`。

### <a name="cancellationtoken"></a>CancellationToken

用于取消异步控制器中的活动。

### <a name="formcollection"></a>FormCollection

用于从发布的表单数据中检索所有的值。

## <a name="input-formatters"></a>输入格式化程序

请求正文中的数据可以是 JSON、XML 或其他某种格式。 要分析此数据，模型绑定会使用配置为处理特定内容类型的输入格式化程序  。 默认情况下，ASP.NET Core 包括用于处理 JSON 数据的基于 JSON 的输入格式化程序。 可以为其他内容类型添加其他格式化程序。

ASP.NET Core 基于 [Consumes](xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute) 属性来选择输入格式化程序。 如果没有属性，它将使用 [Content-Type 标头](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html)。

要使用内置 XML 输入格式化程序，请执行以下操作：

* 安装 `Microsoft.AspNetCore.Mvc.Formatters.Xml` NuGet 包。

* 在 `Startup.ConfigureServices` 中，调用 <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlSerializerFormatters*> 或 <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlDataContractSerializerFormatters*>。

  [!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=10)]

* 将 `Consumes` 属性应用于应在请求正文中使用 XML 的控制器类或操作方法。

  ```csharp
  [HttpPost]
  [Consumes("application/xml")]
  public ActionResult<Pet> Create(Pet pet)
  ```

  有关更多信息，请参阅 [XML 序列化简介](/dotnet/standard/serialization/introducing-xml-serialization)。

### <a name="customize-model-binding-with-input-formatters"></a>使用输入格式化程序自定义模型绑定

由输入格式化程序完全负责从请求正文读取数据。 若要自定义此过程，请配置输入格式化程序使用的 API。 此部分介绍如何自定义基于 `System.Text.Json` 的输入格式化程序，以了解自定义类型 `ObjectId`。 

以包含自定义 `ObjectId` 属性 `Id` 的模型为例：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/ModelWithObjectId.cs?name=snippet_Class&highlight=3)]

使用 `System.Text.Json` 时，若要自定义模型绑定过程，请创建派生自 <xref:System.Text.Json.Serialization.JsonConverter%601> 的类：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/JsonConverters/ObjectIdConverter.cs?name=snippet_Class)]

将 <xref:System.Text.Json.Serialization.JsonConverterAttribute> 属性应用到此类型，以使用自定义转换器。 在下面的示例中，为 `ObjectId` 类型配置了 `ObjectIdConverter` 来作为其自定义转换器：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Models/ObjectId.cs?name=snippet_Class&highlight=1)]

有关详细信息，请参阅[如何编写自定义转换器](/dotnet/standard/serialization/system-text-json-converters-how-to)。

## <a name="exclude-specified-types-from-model-binding"></a>从模型绑定中排除指定类型

模型绑定和验证系统的行为由 [ModelMetadata](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata) 驱动。 可通过向 [MvcOptions.ModelMetadataDetailsProviders](xref:Microsoft.AspNetCore.Mvc.MvcOptions.ModelMetadataDetailsProviders) 添加详细信息提供程序来自定义 `ModelMetadata`。 内置详细信息提供程序可用于禁用指定类型的模型绑定或验证。

要禁用指定类型的所有模型的模型绑定，请在 `Startup.ConfigureServices` 中添加 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.ExcludeBindingMetadataProvider>。 例如，禁用对 `System.Version` 类型的所有模型的模型绑定：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=5-6)]

要禁用指定类型的属性的验证，请在 `Startup.ConfigureServices` 中添加 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.SuppressChildValidationMetadataProvider>。 例如，禁用对 `System.Guid` 类型的属性的验证：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=7-8)]

## <a name="custom-model-binders"></a>自定义模型绑定器

通过编写自定义模型绑定器，并使用 `[ModelBinder]` 属性为给定目标选择该模型绑定器，可扩展模型绑定。 详细了解[自定义模型绑定](xref:mvc/advanced/custom-model-binding)。

## <a name="manual-model-binding"></a>手动模型绑定 

可以使用 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*> 方法手动调用模型绑定。 `ControllerBase` 和 `PageModel` 类上均定义了此方法。 方法重载允许指定要使用的前缀和值提供程序。 如果模型绑定失败，该方法返回 `false`。 以下是一个示例：

[!code-csharp[](model-binding/samples/3.x/ModelBindingSample/Pages/InstructorsWithCollection/Create.cshtml.cs?name=snippet_TryUpdate&highlight=1-4)]

<xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*> 使用值提供程序从窗体正文、查询字符串和路由数据获取数据。 `TryUpdateModelAsync` 通常有以下特点： 

* 与 Razor 使用控制器和视图的页面和 MVC 应用一起使用，以防止过度发布。
* 不用于 Web API（除非窗体数据、查询字符串和路由数据使用它）。 使用 JSON 的 Web API 终结点使用[输入格式化程序](#input-formatters)将请求正文反序列化为对象。

有关详细信息，请参阅 [TryUpdateModelAsync](xref:data/ef-rp/crud#TryUpdateModelAsync)。

## <a name="fromservices-attribute"></a>[FromServices] 属性

此属性的名称遵循指定数据源的模型绑定属性的模式。 但这与绑定来自值提供程序的数据无关。 它从[依赖关系注入](xref:fundamentals/dependency-injection)容器中获取类型的实例。 其目的在于，在仅当调用特定方法时需要服务的情况下，提供构造函数注入的替代方法。

## <a name="additional-resources"></a>其他资源

* <xref:mvc/models/validation>
* <xref:mvc/advanced/custom-model-binding>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本文解释了模型绑定的定义、模型绑定的工作原理，以及如何自定义模型绑定的行为。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/model-binding/samples)（[如何下载](xref:index#how-to-download-a-sample)）。

## <a name="what-is-model-binding"></a>什么是模型绑定

控制器和 Razor 页面处理来自 HTTP 请求的数据。 例如，路由数据可以提供一个记录键，而发布的表单域可以为模型的属性提供一个值。 编写代码以检索这些值，并将其从字符串转换为 .NET 类型不仅繁琐，而且还容易出错。 模型绑定会自动化该过程。 模型绑定系统：

* 从各种源（如路由数据、表单域和查询字符串）中检索数据。
* Razor在方法参数和公共属性中向控制器和页面提供数据。
* 将字符串数据转换为 .NET 类型。
* 更新复杂类型的属性。

## <a name="example"></a>示例

假设有以下操作方法：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Controllers/PetsController.cs?name=snippet_DogsOnly)]

并且应用收到一个带有以下 URL 的请求：

```
http://contoso.com/api/pets/2?DogsOnly=true
```

在路由系统选择该操作方法之后，模型绑定执行以下步骤：

* 查找 `GetByID` 的第一个参数，该参数是一个名为 `id` 的整数。
* 查找 HTTP 请求中的可用源，并在路由数据中查找 `id` =“2”。
* 将字符串“2”转换为整数 2。
* 查找 `GetByID` 的下一个参数，该参数是一个名为 `dogsOnly` 的布尔值。
* 查找源，并在查询字符串中查找“DogsOnly=true”。 名称匹配不区分大小写。
* 将字符串“true”转换为布尔值 `true`。

然后，该框架会调用 `GetById` 方法，为 `id` 参数传入 2，并为 `dogsOnly` 参数传入 `true`。

在前面的示例中，模型绑定目标是简单类型的方法参数。 目标也可以是复杂类型的属性。 成功绑定每个属性后，将对属性进行[模型验证](xref:mvc/models/validation)。 有关绑定到模型的数据以及任意绑定或验证错误的记录都存储在 [ControllerBase.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) 或 [PageModel.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) 中。 为查明该过程是否已成功，应用会检查 [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid) 标志。

## <a name="targets"></a>目标

模型绑定尝试查找以下类型目标的值：

* 将请求路由到的控制器操作方法的参数。
* Razor请求路由到的页处理程序方法的参数。 
* 控制器或 `PageModel` 类的公共属性（若由特性指定）。

### <a name="bindproperty-attribute"></a>[BindProperty] 属性

可应用于控制器或 `PageModel` 类的公共属性，从而使模型绑定以该属性为目标：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Edit.cshtml.cs?name=snippet_BindProperty&highlight=3-4)]

### <a name="bindproperties-attribute"></a>[BindProperties] 属性

可在 ASP.NET Core 2.1 及更高版本中获得。  可应用于控制器或 `PageModel` 类，以使模型绑定以该类的所有公共属性为目标：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_BindProperties&highlight=1-2)]

### <a name="model-binding-for-http-get-requests"></a>HTTP GET 请求的模型绑定

默认情况下，不绑定 HTTP GET 请求的属性。 通常，GET 请求只需一个记录 ID 参数。 记录 ID 用于查找数据库中的项。 因此，无需绑定包含模型实例的属性。 在需要将属性绑定到 GET 请求中的数据的情况下，请将 `SupportsGet` 属性设置为 `true`：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_SupportsGet)]

## <a name="sources"></a>源

默认情况下，模型绑定以键值对的形式从 HTTP 请求中的以下源中获取数据：

1. 表单域
1. 请求正文（对于[具有 [ApiController] 属性的控制器](xref:web-api/index#binding-source-parameter-inference)。）
1. 路由数据
1. 查询字符串参数
1. 上传的文件

对于每个目标参数或属性，按照之前列表中指示的顺序扫描源。 有几个例外情况：

* 路由数据和查询字符串值仅用于简单类型。
* 上传的文件仅绑定到实现 `IFormFile` 或 `IEnumerable<IFormFile>` 的目标类型。

如果默认源不正确，请使用下列属性之一来指定源：

* [`[FromQuery]`](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute) -从查询字符串获取值。 
* [`[FromRoute]`](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute) -从路由数据中获取值。
* [`[FromForm]`](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute) -从已发布的表单字段中获取值。
* [`[FromBody]`](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute) -从请求正文中获取值。
* [`[FromHeader]`](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute) -从 HTTP 标头中获取值。

这些属性：

* 分别添加到模型属性（而不是模型类），如以下示例所示：

  [!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Models/Instructor.cs?name=snippet_FromQuery&highlight=5-6)]

* 选择性地在构造函数中接受模型名称值。 提供此选项的目的是应对属性名称与请求中的值不匹配的情况。 例如，请求中的值可能是名称中带有连字符的标头，如以下示例所示：

  [!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Index.cshtml.cs?name=snippet_FromHeader)]

### <a name="frombody-attribute"></a>[FromBody] 属性

将 `[FromBody]` 特性应用于一个参数，以便从一个 HTTP 请求的正文填充其属性。 ASP.NET Core 运行时将读取正文的责任委托给输入格式化程序。 输入格式化程序的解释位于[本文后面部分](#input-formatters)。

将 `[FromBody]` 应用于复杂类型参数时，应用于其属性的任何绑定源属性都将被忽略。 例如，以下 `Create` 操作指定从正文填充其 `pet` 参数：

```csharp
public ActionResult<Pet> Create([FromBody] Pet pet)
```

`Pet` 类指定从查询字符串参数填充其 `Breed` 属性：

```csharp
public class Pet
{
    public string Name { get; set; }

    [FromQuery] // Attribute is ignored.
    public string Breed { get; set; }
}
```

在上面的示例中：

* `[FromQuery]` 特性被忽略。
* `Breed` 属性未从查询字符串参数进行填充。 

输入格式化程序只读取正文，不了解绑定源特性。 如果在正文中找到合适的值，则使用该值填充 `Breed` 属性。

不要将 `[FromBody]` 应用于每个操作方法的多个参数。 输入格式化程序读取请求流后，无法再次读取该流以绑定其他 `[FromBody]` 参数。

### <a name="additional-sources"></a>其他源

源数据由“值提供程序”提供给模型绑定系统  。 你可以编写并注册自定义值提供程序，这些提供程序从其他源中获取用于模型绑定的数据。 例如，你可能需要来自 cookie 或会话状态的数据。 要从新的源中获取数据，请执行以下操作：

* 创建用于实现 `IValueProvider` 的类。
* 创建用于实现 `IValueProviderFactory` 的类。
* 在 `Startup.ConfigureServices` 中注册工厂类。

该示例应用包含一个 [值提供程序](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/models/model-binding/samples/2.x/ModelBindingSample/CookieValueProvider.cs) 和一个 [工厂](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/models/model-binding/samples/2.x/ModelBindingSample/CookieValueProviderFactory.cs) 示例，可从 s 中获取值 cookie 。 以下是 `Startup.ConfigureServices` 中的注册代码：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=3)]

所示代码将自定义值提供程序置于所有内置值提供程序之后。  要将其置于列表中的首位，请调用 `Insert(0, new CookieValueProviderFactory())` 而不是 `Add`。

## <a name="no-source-for-a-model-property"></a>不存在模型属性的源

默认情况下，如果找不到模型属性的值，则不会创建模型状态错误。 该属性设置为 NULL 或默认值：

* 可以为 Null 的简单类型设置为 `null`。
* 不可以为 Null 的值类型设置为 `default(T)`。 例如，参数 `int id` 设置为 0。
* 对于复杂类型，模型绑定使用默认构造函数来创建实例，而不设置属性。
* 数组设置为 `Array.Empty<T>()`，但 `byte[]` 数组设置为 `null`。

如果在模型属性的窗体字段中未找到任何内容时模型状态应失效，请使用 [`[BindRequired]`](#bindrequired-attribute) 特性。

请注意，此 `[BindRequired]` 行为适用于发布的表单数据中的模型绑定，而不适用于请求正文中的 JSON 或 XML 数据。 请求正文数据由[输入格式化程序](#input-formatters)进行处理。

## <a name="type-conversion-errors"></a>类型转换错误

如果找到源，但无法将其转换为目标类型，则模型状态将被标记为无效。 目标参数或属性设置为 NULL 或默认值，如上一部分所述。

在具有 `[ApiController]` 属性的 API 控制器中，无效的模型状态会导致自动 HTTP 400 响应。

在 Razor 页面中，重新显示页面并显示一条错误消息：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/Instructors/Create.cshtml.cs?name=snippet_HandleMBError&highlight=3-6)]

客户端验证将捕获大多数提交到 Razor 页面窗体的错误数据。 此验证使得先前突出显示的代码难以被触发。 示例应用包含一个“提交无效日期”按钮，该按钮将错误数据置于“雇用日期”字段中并提交表单  。 此按钮显示在发生数据转换错误时用于重新显示页的代码将如何工作。

在使用先前的代码重新显示页时，表单域中不会显示无效的输入。 这是因为模型属性已设置为 NULL 或默认值。 无效输入会出现在错误消息中。 但是，如果要在表单域中重新显示错误数据，可以考虑将模型属性设置为字符串并手动执行数据转换。

如果不希望发生类型转换错误导致模型状态错误的情况，建议使用相同的策略。 在这种情况下，将模型属性设置为字符串。

## <a name="simple-types"></a>简单类型

模型绑定器可以将源字符串转换为以下简单类型：

* [布尔值](xref:System.ComponentModel.BooleanConverter)
* [字节](xref:System.ComponentModel.ByteConverter)、[SByte](xref:System.ComponentModel.SByteConverter)
* [Char](xref:System.ComponentModel.CharConverter)
* [DateTime](xref:System.ComponentModel.DateTimeConverter)
* [DateTimeOffset](xref:System.ComponentModel.DateTimeOffsetConverter)
* [小数](xref:System.ComponentModel.DecimalConverter)
* [双精度](xref:System.ComponentModel.DoubleConverter)
* [枚举](xref:System.ComponentModel.EnumConverter)
* [Guid.empty](xref:System.ComponentModel.GuidConverter)
* [Int16](xref:System.ComponentModel.Int16Converter)、[Int32](xref:System.ComponentModel.Int32Converter)、[Int64](xref:System.ComponentModel.Int64Converter)
* [单精度](xref:System.ComponentModel.SingleConverter)
* [TimeSpan](xref:System.ComponentModel.TimeSpanConverter)
* [UInt16](xref:System.ComponentModel.UInt16Converter)、[UInt32](xref:System.ComponentModel.UInt32Converter)、[UInt64](xref:System.ComponentModel.UInt64Converter)
* [Uri](xref:System.UriTypeConverter)
* [版本](xref:System.ComponentModel.VersionConverter)

## <a name="complex-types"></a>复杂类型

复杂类型必须具有要绑定的公共默认构造函数和公共可写属性。 进行模型绑定时，将使用公共默认构造函数来实例化类。 

对于复杂类型的每个属性，模型绑定会查找名称模式 prefix.property_name 的源  。 如果未找到，它将仅查找不含前缀的 properties_name  。

对于绑定到参数，前缀是参数名称。 对于绑定到 `PageModel` 公共属性，前缀是公共属性名称。 某些属性具有 `Prefix` 属性，让你可以替代参数或属性名称的默认用法。

例如，假设复杂类型是以下 `Instructor` 类：

  ```csharp
  public class Instructor
  {
      public int ID { get; set; }
      public string LastName { get; set; }
      public string FirstName { get; set; }
  }
  ```

### <a name="prefix--parameter-name"></a>前缀 = 参数名称

如果要绑定的模型是一个名为 `instructorToUpdate` 的参数：

```csharp
public IActionResult OnPost(int? id, Instructor instructorToUpdate)
```

模型绑定从查找键 `instructorToUpdate.ID` 的源开始操作。 如果未找到，它将查找不含前缀的 `ID`。

### <a name="prefix--property-name"></a>前缀 = 属性名称

如果要绑定的模型是控制器或 `PageModel` 类的一个名为 `Instructor` 的属性：

```csharp
[BindProperty]
public Instructor Instructor { get; set; }
```

模型绑定从查找键 `Instructor.ID` 的源开始操作。 如果未找到，它将查找不含前缀的 `ID`。

### <a name="custom-prefix"></a>自定义前缀

如果要绑定的模型是名为 `instructorToUpdate` 的参数，并且 `Bind` 属性指定 `Instructor` 作为前缀：

```csharp
public IActionResult OnPost(
    int? id, [Bind(Prefix = "Instructor")] Instructor instructorToUpdate)
```

模型绑定从查找键 `Instructor.ID` 的源开始操作。 如果未找到，它将查找不含前缀的 `ID`。

### <a name="attributes-for-complex-type-targets"></a>复杂类型目标的属性

多个内置属性可用于控制复杂类型的模型绑定：

* `[BindRequired]`
* `[BindNever]`
* `[Bind]`

> [!NOTE]
> 如果发布的表单数据是值的源，则这些属性会影响模型绑定。 它们不会影响处理发布的 JSON 和 XML 请求正文的输入格式化程序。 输入格式化程序的解释位于[本文后面部分](#input-formatters)。
>
> 另请参阅[模型验证](xref:mvc/models/validation#required-attribute)中针对 `[Required]` 属性的讨论。

### <a name="bindrequired-attribute"></a>[BindRequired] 属性

只能应用于模型属性，不能应用于方法参数。 如果无法对模型属性进行绑定，则会导致模型绑定添加模型状态错误。 以下是一个示例：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Models/InstructorWithCollection.cs?name=snippet_BindRequired&highlight=8-9)]

### <a name="bindnever-attribute"></a>[BindNever] 属性

只能应用于模型属性，不能应用于方法参数。 防止模型绑定设置模型的属性。 以下是一个示例：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Models/InstructorWithDictionary.cs?name=snippet_BindNever&highlight=3-4)]

### <a name="bind-attribute"></a>[Bind] 属性

可应用于类或方法参数。 指定模型绑定中应包含的模型属性。

在下面的示例中，当调用任意处理程序或操作方法时，只绑定 `Instructor` 模型的指定属性：

```csharp
[Bind("LastName,FirstMidName,HireDate")]
public class Instructor
```

在下面的示例中，当调用 `OnPost` 方法时，只绑定 `Instructor` 模型的指定属性：

```csharp
[HttpPost]
public IActionResult OnPost([Bind("LastName,FirstMidName,HireDate")] Instructor instructor)
```

`[Bind]` 属性可用于防止“创建”方案中的过多发布情况  。 由于排除的属性设置为 NULL 或默认值，而不是保持不变，因此它在编辑方案中无法很好地工作。 为防止过多发布，建议使用视图模型，而不是使用 `[Bind]` 属性。 有关详细信息，请参阅[有关过多发布的安全性说明](xref:data/ef-mvc/crud#security-note-about-overposting)。

## <a name="collections"></a>集合

对于是简单类型集合的目标，模型绑定将查找 parameter_name 或 property_name 的匹配项  。 如果找不到匹配项，它将查找某种不含前缀的受支持的格式。 例如： 。

* 假设要绑定的参数是名为 `selectedCourses` 的数组：

  ```csharp
  public IActionResult OnPost(int? id, int[] selectedCourses)
  ```

* 表单或查询字符串数据可以采用以下某种格式：
   
  ```
  selectedCourses=1050&selectedCourses=2000 
  ```

  ```
  selectedCourses[0]=1050&selectedCourses[1]=2000
  ```

  ```
  [0]=1050&[1]=2000
  ```

  ```
  selectedCourses[a]=1050&selectedCourses[b]=2000&selectedCourses.index=a&selectedCourses.index=b
  ```

  ```
  [a]=1050&[b]=2000&index=a&index=b
  ```

* 只有表单数据支持以下格式：

  ```
  selectedCourses[]=1050&selectedCourses[]=2000
  ```

* 对于前面所有的示例格式，模型绑定将两个项的数组传递给 `selectedCourses` 参数：

  * selectedCourses[0]=1050
  * selectedCourses[1]=2000

  使用下标数字的数据格式 (... [0] ... [1] ...) 必须确保从零开始按顺序进行编号。 如果下标编号中存在任何间隔，则间隔后的所有项都将被忽略。 例如，如果下标是 0 和 2，而不是 0 和 1，则第二个项会被忽略。

## <a name="dictionaries"></a>字典

对于 `Dictionary` 目标，模型绑定会查找 parameter_name 或 property_name 的匹配项  。 如果找不到匹配项，它将查找某种不含前缀的受支持的格式。 例如： 。

* 假设目标参数是名为 `selectedCourses` 的 `Dictionary<int, string>`：

  ```csharp
  public IActionResult OnPost(int? id, Dictionary<int, string> selectedCourses)
  ```

* 发布的表单或查询字符串数据可以类似于以下某一示例：

  ```
  selectedCourses[1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  [1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  selectedCourses[0].Key=1050&selectedCourses[0].Value=Chemistry&
  selectedCourses[1].Key=2000&selectedCourses[1].Value=Economics
  ```

  ```
  [0].Key=1050&[0].Value=Chemistry&[1].Key=2000&[1].Value=Economics
  ```

* 对于前面所有的示例格式，模型绑定将两个项的字典传递给 `selectedCourses` 参数：

  * selectedCourses["1050"]="Chemistry"
  * selectedCourses["2000"]="Economics"

<a name="glob"></a>

## <a name="globalization-behavior-of-model-binding-route-data-and-query-strings"></a>模型绑定路由数据和查询字符串的全球化行为

ASP.NET Core 路由值提供程序和查询字符串值提供程序：

* 将值视为固定区域性。
* URL 的区域性应固定。

相反，来自窗体数据的值要进行区分区域性的转换。 这是设计使然，目的是让 URL 可在各个区域设置中共享。

使 ASP.NET Core 路由值提供程序和查询字符串值提供程序进行区分区域性的转换：

* 继承自 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.IValueProviderFactory>
* 从 [QueryStringValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs) 或 [RouteValueValueProviderFactory](https://github.com/dotnet/AspNetCore/blob/master/src/Mvc/Mvc.Core/src/ModelBinding/RouteValueProviderFactory.cs) 复制代码
* 使用 [CultureInfo.CurrentCulture](xref:System.Globalization.CultureInfo.CurrentCulture) 替换传递给值提供程序构造函数的[区域性值](https://github.com/dotnet/AspNetCore/blob/e625fe29b049c60242e8048b4ea743cca65aa7b5/src/Mvc/Mvc.Core/src/ModelBinding/QueryStringValueProviderFactory.cs#L30)
* 将 MVC 选项中的默认值提供程序工厂替换为新的工厂：

[!code-csharp[](model-binding/samples_snapshot/2.x/Startup.cs?name=snippet)]
[!code-csharp[](model-binding/samples_snapshot/2.x/Startup.cs?name=snippet1)]

## <a name="special-data-types"></a>特殊数据类型

模型绑定可以处理某些特殊的数据类型。

### <a name="iformfile-and-iformfilecollection"></a>IFormFile 和 IFormFileCollection

HTTP 请求中包含的上传文件。  还支持多个文件的 `IEnumerable<IFormFile>`。

### <a name="cancellationtoken"></a>CancellationToken

用于取消异步控制器中的活动。

### <a name="formcollection"></a>FormCollection

用于从发布的表单数据中检索所有的值。

## <a name="input-formatters"></a>输入格式化程序

请求正文中的数据可以是 JSON、XML 或其他某种格式。 要分析此数据，模型绑定会使用配置为处理特定内容类型的输入格式化程序  。 默认情况下，ASP.NET Core 包括用于处理 JSON 数据的基于 JSON 的输入格式化程序。 可以为其他内容类型添加其他格式化程序。

ASP.NET Core 基于 [Consumes](xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute) 属性来选择输入格式化程序。 如果没有属性，它将使用 [Content-Type 标头](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html)。

要使用内置 XML 输入格式化程序，请执行以下操作：

* 安装 `Microsoft.AspNetCore.Mvc.Formatters.Xml` NuGet 包。

* 在 `Startup.ConfigureServices` 中，调用 <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlSerializerFormatters*> 或 <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlDataContractSerializerFormatters*>。

  [!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=9)]

* 将 `Consumes` 属性应用于应在请求正文中使用 XML 的控制器类或操作方法。

  ```csharp
  [HttpPost]
  [Consumes("application/xml")]
  public ActionResult<Pet> Create(Pet pet)
  ```

  有关更多信息，请参阅 [XML 序列化简介](/dotnet/standard/serialization/introducing-xml-serialization)。

## <a name="exclude-specified-types-from-model-binding"></a>从模型绑定中排除指定类型

模型绑定和验证系统的行为由 [ModelMetadata](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata) 驱动。 可通过向 [MvcOptions.ModelMetadataDetailsProviders](xref:Microsoft.AspNetCore.Mvc.MvcOptions.ModelMetadataDetailsProviders) 添加详细信息提供程序来自定义 `ModelMetadata`。 内置详细信息提供程序可用于禁用指定类型的模型绑定或验证。

要禁用指定类型的所有模型的模型绑定，请在 `Startup.ConfigureServices` 中添加 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.ExcludeBindingMetadataProvider>。 例如，禁用对 `System.Version` 类型的所有模型的模型绑定：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=4-5)]

要禁用指定类型的属性的验证，请在 `Startup.ConfigureServices` 中添加 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.SuppressChildValidationMetadataProvider>。 例如，禁用对 `System.Guid` 类型的属性的验证：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Startup.cs?name=snippet_ValueProvider&highlight=6-7)]

## <a name="custom-model-binders"></a>自定义模型绑定器

通过编写自定义模型绑定器，并使用 `[ModelBinder]` 属性为给定目标选择该模型绑定器，可扩展模型绑定。 详细了解[自定义模型绑定](xref:mvc/advanced/custom-model-binding)。

## <a name="manual-model-binding"></a>手动模型绑定

可以使用 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*> 方法手动调用模型绑定。 `ControllerBase` 和 `PageModel` 类上均定义了此方法。 方法重载允许指定要使用的前缀和值提供程序。 如果模型绑定失败，该方法返回 `false`。 以下是一个示例：

[!code-csharp[](model-binding/samples/2.x/ModelBindingSample/Pages/InstructorsWithCollection/Create.cshtml.cs?name=snippet_TryUpdate&highlight=1-4)]

## <a name="fromservices-attribute"></a>[FromServices] 属性

此属性的名称遵循指定数据源的模型绑定属性的模式。 但这与绑定来自值提供程序的数据无关。 它从[依赖关系注入](xref:fundamentals/dependency-injection)容器中获取类型的实例。 其目的在于，在仅当调用特定方法时需要服务的情况下，提供构造函数注入的替代方法。

## <a name="additional-resources"></a>其他资源

* <xref:mvc/models/validation>
* <xref:mvc/advanced/custom-model-binding>

::: moniker-end
