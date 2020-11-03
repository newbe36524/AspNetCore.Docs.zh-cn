---
title: ASP.NET Core MVC 中的视图
author: ardalis
description: 了解 ASP.NET Core MVC 中的视图如何处理应用的数据表示和用户交互。
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
uid: mvc/views/overview
ms.openlocfilehash: 373b17377740441d3859e3b7d942017a22bc7a68
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060620"
---
# <a name="views-in-aspnet-core-mvc"></a>ASP.NET Core MVC 中的视图

作者：[Steve Smith](https://ardalis.com/)

本文档介绍在 ASP.NET Core MVC 应用程序中使用的视图。 有关页面的信息 Razor ，请参阅 [ Razor 页面简介](xref:razor-pages/index)。

在“模型-视图-控制器(MVC)”模式中，视图处理应用的数据表示和用户交互  。 视图是具有嵌入[ Razor 标记](xref:mvc/views/razor)的 HTML 模板。 Razor 标记是与 HTML 标记交互以生成发送到客户端的网页的代码。

在 ASP.NET Core MVC 中，视图是在标记中使用 [c # 编程语言](/dotnet/csharp/)的 *cshtml* 文件。 Razor 通常，视图文件会分组到以每个应用的[控制器](xref:mvc/controllers/actions)命名的文件夹中。 此文件夹存储在应用根目录的“Views”文件夹中  ：

![Visual Studio 解决方案资源管理器中的“Views”文件夹与“Home”文件夹一同打开，显示 About.cshtml、Contact.cshtml 和 Index.cshtml 文件](overview/_static/views_solution_explorer.png)

主页控制器由“Views”文件夹内的“Home”文件夹表示  。 “Home”文件夹包含“关于”、“联系人”和“索引”（主页）网页的视图  。 用户请求这三个网页中的一个时，主页控制器中的控制器操作决定使用三个视图中的哪一个来生成网页并将其返回给用户  。

使用[布局](xref:mvc/views/layout)提供一致的网页部分并减少代码重复。 布局通常包含页眉、导航和菜单元素以及页脚。 页眉和页脚通常包含许多元数据元素的样板标记以及脚本和样式资产的链接。 布局有助于在视图中避免这种样板标记。

[分部视图](xref:mvc/views/partial)通过管理视图的可重用部分来减少代码重复。 例如，分部视图可用于在多个视图中出现的博客网站上的作者简介。 作者简介是普通的视图内容，不需要执行代码就能生成网页的内容。 可以仅通过模型绑定查看作者简介内容，因此使用这种内容类型的分部视图是理想的选择。

[视图组件](xref:mvc/views/view-components)与分部视图的相似之处在于它们可以减少重复性代码，但视图组件还适用于需要在服务器上运行代码才能呈现网页的视图内容。 呈现的内容需要数据库交互时（例如网站购物车），视图组件非常有用。 为了生成网页输出，视图组件不局限于模型绑定。

## <a name="benefits-of-using-views"></a>使用视图的好处

视图可帮助在 MVC 应用内建立[关注点分离](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#separation-of-concerns)，方法是分隔用户界面标记与应用的其他部分。 采用 SoC 设计可使应用模块化，从而提供以下几个好处：

* 应用组织地更好，因此更易于维护。 视图一般按应用功能进行分组。 这使得在处理功能时更容易找到相关的视图。
* 应用的若干部分是松散耦合的。 可以生成和更新独立于业务逻辑和数据访问组件的应用视图。 可以修改应用的视图，而不必更新应用的其他部分。
* 因为视图是独立的单元，所以更容易测试应用的用户界面部分。
* 由于应用组织得更好，因此你不太可能会意外重复用户界面的各个部分。

## <a name="creating-a-view"></a>创建视图

在 Views / [ControllerName] 文件夹中创建特定于控制器的视图  。 控制器之间共享的视图都将置于 Views/Shared 文件夹  。 要创建一个视图，请添加新文件，并将其命名为与 .cshtml 文件扩展名相关联的控制器操作的相同名称  。 要创建与主页控制器中 About 操作相对应的视图，请在 Views/Home 文件夹中创建一个 About.cshtml 文件  ：

[!code-cshtml[](../../common/samples/WebApplication1/Views/Home/About.cshtml)]

*Razor* 标记以符号开头 `@` 。 运行 c # 语句，方法是将 c # 代码放在由大括号 () 设置的[ Razor 代码块](xref:mvc/views/razor#razor-code-blocks)中 `{ ... }` 。 有关示例，请参阅上面显示的“About”到 `ViewData["Title"]` 的分配。 只需用 `@` 符号来引用值，即可在 HTML 中显示这些值。 请参阅上面的 `<h2>` 和 `<h3>` 元素的内容。

以上所示的视图内容只是呈现给用户的整个网页中的一部分。 其他视图文件中指定了页面布局的其余部分和视图的其他常见方面。 要了解详细信息，请参阅[布局主题](xref:mvc/views/layout)。

## <a name="how-controllers-specify-views"></a>控制器如何指定视图

视图通常以 [ ViewResult](/dotnet/api/microsoft.aspnetcore.mvc.viewresult) 的形式从操作返回，这是一种 [ ActionResult ](/dotnet/api/microsoft.aspnetcore.mvc.actionresult)。 操作方法可以直接创建并返回 `ViewResult`，但通常不会这样做。 由于大多数控制器均继承自[控制器](/dotnet/api/microsoft.aspnetcore.mvc.controller)，因此只需使用 `View` 帮助程序方法即可返回 `ViewResult`：

HomeController.cs 

[!code-csharp[](../../common/samples/WebApplication1/Controllers/HomeController.cs?highlight=5&range=16-21)]

此操作返回时，最后一节显示的 About.cshtml 视图呈现为以下网页  ：

![Microsoft Edge 浏览器中呈现的“关于”页面](overview/_static/about-page.png)

`View` 帮助程序方法有多个重载。 可选择指定：

* 要返回的显式视图：

  ```csharp
  return View("Orders");
  ```

* 要传递给视图的[模型](xref:mvc/models/model-binding)：

  ```csharp
  return View(Orders);
  ```

* 视图和模型：

  ```csharp
  return View("Orders", Orders);
  ```

### <a name="view-discovery"></a>视图发现

操作返回一个视图时，会发生称为“视图发现”的过程  。 此过程基于视图名称确定使用哪个视图文件。 

`View` 方法 的默认行为 (`return View();`) 旨在返回与其从中调用的操作方法同名的视图。 例如，控制器的 " *关于* " `ActionResult` 方法名称用于搜索名为 " *关于* " 的视图文件。 运行时首先在 Views/[ControllerName] 文件夹中搜索该视图  。 如果在此处找不到匹配的视图，则会在“Shared”文件夹中搜索该视图  。

用 `return View();` 隐式返回 `ViewResult` 还是用 `return View("<ViewName>");` 将视图名称显式传递给 `View` 方法并不重要。 在这两种情况下，视图发现都会按以下顺序搜索匹配的视图文件：

   1. *Views/ \[ ControllerName]/ \[ ViewName] cshtml*
   1. *Views/Shared/ \[ ViewName] cshtml*

可以提供视图文件路径而不提供视图名称。 如果使用从应用根目录开始的绝对路径（可选择以“/”或“~/”开头），则须指定 .cshtml 扩展名  ：

```csharp
return View("Views/Home/About.cshtml");
```

也可使用相对路径在不同目录中指定视图，而无需指定 .cshtml 扩展名  。 在 `HomeController` 内，可以使用相对路径返回 Manage 视图的 Index 视图  ：

```csharp
return View("../Manage/Index");
```

同样，可以用“./”前缀来指示当前的控制器特定目录：

```csharp
return View("./About");
```

[分部视图](xref:mvc/views/partial)和[视图组件](xref:mvc/views/view-components)使用类似（但不完全相同）的发现机制。

可以使用自定义 [IViewLocationExpander](/dotnet/api/microsoft.aspnetcore.mvc.razor.iviewlocationexpander) 自定义如何在应用中定位视图的默认约定。

视图发现依赖于按文件名称查找视图文件。 如果基础文件系统区分大小写，则视图名称也可能区分大小写。 为了各操作系统的兼容性，请在控制器与操作名称之间，关联视图文件夹与文件名称之间匹配大小写。 如果在处理区分大小写的文件系统时遇到无法找到视图文件的错误，请确认请求的视图文件与实际视图文件名称之间的大小写是否匹配。

按照组织视图文件结构的最佳做法，反映控制器、操作和视图之间的关系，实现可维护性和清晰度。

## <a name="passing-data-to-views"></a>向视图传递数据

使用多种方法将数据传递给视图：

* 强类型数据：viewmodel
* 弱类型数据
  * `ViewData` (`ViewDataAttribute`)
  * `ViewBag`

### <a name="strongly-typed-data-viewmodel"></a>强类型数据 (viewmodel)

最可靠的方法是在视图中指定[模型](xref:mvc/models/model-binding)类型。 此模型通常称为 viewmodel  。 将 viewmodel 类型的实例传递给此操作的视图。

使用 viewmodel 将数据传递给视图可让视图充分利用强类型检查  。 强类型化（或强类型）意味着每个变量和常量都有明确定义的类型（例如 `string`、`int` 或 `DateTime`）  。 在编译时检查视图中使用的类型是否有效。

[Visual Studio](https://visualstudio.microsoft.com) 和 [Visual Studio Code](https://code.visualstudio.com/) 列出了使用 [IntelliSense](/visualstudio/ide/using-intellisense) 功能的强类型类成员。 如果要查看 viewmodel 的属性，请键入 viewmodel 的变量名称，后跟句点 (`.`)。 这有助于提高编写代码的速度并降低错误率。

使用 `@model` 指令指定模型。 使用带有 `@Model` 的模型：

```cshtml
@model WebApplication1.ViewModels.Address

<h2>Contact</h2>
<address>
    @Model.Street<br>
    @Model.City, @Model.State @Model.PostalCode<br>
    <abbr title="Phone">P:</abbr> 425.555.0100
</address>
```

为了将模型提供给视图，控制器将其作为参数进行传递：

```csharp
public IActionResult Contact()
{
    ViewData["Message"] = "Your contact page.";

    var viewModel = new Address()
    {
        Name = "Microsoft",
        Street = "One Microsoft Way",
        City = "Redmond",
        State = "WA",
        PostalCode = "98052-6399"
    };

    return View(viewModel);
}
```

没有针对可以提供给视图的模型类型的限制。 建议使用普通旧 CLR 对象 (POCO) viewmodel，它几乎没有已定义的行为（方法）。 通常，viewmodel 类要么存储在“Models”文件夹中，要么存储在应用根目录处的单独“ViewModels”文件夹中  。 上例中使用的 Address viewmodel 是存储在 Address.cs 文件中的 POCO viewmodel  ：

```csharp
namespace WebApplication1.ViewModels
{
    public class Address
    {
        public string Name { get; set; }
        public string Street { get; set; }
        public string City { get; set; }
        public string State { get; set; }
        public string PostalCode { get; set; }
    }
}
```

可随意对 viewmodel 类型和业务模型类型使用相同的类。 但是，使用单独的模型可使视图独立于应用的业务逻辑和数据访问部分。 模型为用户发送给应用的数据使用[模型绑定](xref:mvc/models/model-binding)和[验证](xref:mvc/models/validation)时，模型和 viewmodel 的分离也会提供安全优势。

<a name="VD_VB"></a>

### <a name="weakly-typed-data-viewdata-viewdata-attribute-and-viewbag"></a>弱类型数据（ViewData、ViewData 属性和 ViewBag）

`ViewBag`*在中 Razor 不可用页面。*

除了强类型视图，视图还可以访问弱类型（也称为松散类型）的数据集合  。 与强类型不同，弱类型（或松散类型）意味着不显式声明要使用的数据类型  。 可以使用弱类型数据集合将少量数据传入及传出控制器和视图。

| 传递数据于...                        | 示例                                                                        |
| ------------------------------------------------- | ------------------------------------------------------------------------------ |
| 控制器和视图                             | 用数据填充下拉列表。                                          |
| 视图和[布局视图](xref:mvc/views/layout)   | 在 **\<title>** 视图文件的布局视图中设置元素内容。  |
| [分部视图](xref:mvc/views/partial)和视图 | 基于用户请求的网页显示数据的小组件。      |

可以通过控制器和视图上的 `ViewData` 或 `ViewBag` 属性来引用此集合。 `ViewData` 属性是弱类型对象的字典。 `ViewBag` 属性是 `ViewData` 的包装器，为基础 `ViewData` 集合提供动态属性。 注意：对于和，键查找不区分大小 `ViewData` 写 `ViewBag` 。

`ViewData` 和 `ViewBag` 在运行时进行动态解析。 由于它们不提供编译时类型检查，因此使用这两者通常比使用 viewmodel 更容易出错。 出于上述原因，一些开发者希望尽量减少或根本不使用 `ViewData` 和 `ViewBag`。

<a name="VD"></a>

**ViewData**

`ViewData` 是通过 `string` 键访问的 [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) 对象。 字符串数据可以直接存储和使用，而不需要强制转换，但是在提取其他 `ViewData` 对象值时必须将其强制转换为特定类型。 可以使用 `ViewData` 将数据从控制器传递到视图，以及在视图（包括[分部视图](xref:mvc/views/partial)和[布局](xref:mvc/views/layout)）内传递数据。

以下是在操作中使用 `ViewData` 设置问候语和地址值的示例：

```csharp
public IActionResult SomeAction()
{
    ViewData["Greeting"] = "Hello";
    ViewData["Address"]  = new Address()
    {
        Name = "Steve",
        Street = "123 Main St",
        City = "Hudson",
        State = "OH",
        PostalCode = "44236"
    };

    return View();
}
```

在视图中处理数据：

```cshtml
@{
    // Since Address isn't a string, it requires a cast.
    var address = ViewData["Address"] as Address;
}

@ViewData["Greeting"] World!

<address>
    @address.Name<br>
    @address.Street<br>
    @address.City, @address.State @address.PostalCode
</address>
```

::: moniker range=">= aspnetcore-2.1"

**ViewData 特性**

另一种会使用 [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary) 的方法是 [ViewDataAttribute](/dotnet/api/microsoft.aspnetcore.mvc.viewdataattribute)。 使用属性标记的控制器或页模型上的属性 Razor `[ViewData]` 将存储其值并从字典中加载它们。

在下面的示例中，“主页”控制器包含使用 `[ViewData]` 标记的 `Title` 属性。 `About` 方法设置“关于”视图的标题：

```csharp
public class HomeController : Controller
{
    [ViewData]
    public string Title { get; set; }

    public IActionResult About()
    {
        Title = "About Us";
        ViewData["Message"] = "Your application description page.";

        return View();
    }
}
```

在布局中，从 ViewData 字典读取标题：

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"] - WebApplication</title>
    ...
```

::: moniker-end

**ViewBag**

`ViewBag`*在中 Razor 不可用页面。*

`ViewBag` 是一个 [DynamicViewData](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.internal.dynamicviewdata) 对象，它提供对存储在中的对象的动态访问 `ViewData` 。 `ViewBag` 不需要强制转换，因此使用起来更加方便。 下例演示如何使用与上述 `ViewData` 有相同结果的 `ViewBag`：

```csharp
public IActionResult SomeAction()
{
    ViewBag.Greeting = "Hello";
    ViewBag.Address  = new Address()
    {
        Name = "Steve",
        Street = "123 Main St",
        City = "Hudson",
        State = "OH",
        PostalCode = "44236"
    };

    return View();
}
```

```cshtml
@ViewBag.Greeting World!

<address>
    @ViewBag.Address.Name<br>
    @ViewBag.Address.Street<br>
    @ViewBag.Address.City, @ViewBag.Address.State @ViewBag.Address.PostalCode
</address>
```

**同时使用 ViewData 和 ViewBag**

`ViewBag`*在中 Razor 不可用页面。*

由于 `ViewData` 和 `ViewBag` 引用相同的基础 `ViewData` 集合，因此在读取和写入值时，可以同时使用 `ViewData` 和 `ViewBag`，并在两者之间进行混合和匹配。

在 About.cshtml 视图顶部，使用 `ViewBag` 设置标题并使用 `ViewData` 设置说明  ：

```cshtml
@{
    Layout = "/Views/Shared/_Layout.cshtml";
    ViewBag.Title = "About Contoso";
    ViewData["Description"] = "Let us tell you about Contoso's philosophy and mission.";
}
```

读取属性，但反向使用 `ViewData` 和 `ViewBag`。 在 _Layout.cshtml 文件中，使用 `ViewData` 获取标题并使用 `ViewBag` 获取说明  ：

```cshtml
<!DOCTYPE html>
<html lang="en">
<head>
    <title>@ViewData["Title"]</title>
    <meta name="description" content="@ViewBag.Description">
    ...
```

请记住，字符串不需要为 `ViewData` 进行强制转换。 可以使用 `@ViewData["Title"]` 而不需要强制转换。

可同时使用 `ViewData` 和 `ViewBag`也可混合和匹配读取及写入属性。 呈现以下标记：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>About Contoso</title>
    <meta name="description" content="Let us tell you about Contoso's philosophy and mission.">
    ...
```

**ViewData 和 ViewBag 之间差异的摘要**

 `ViewBag` 页中不可用 Razor 。

* `ViewData`
  * 派生自 [ViewDataDictionary](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.viewdatadictionary)，因此它具有可有用的字典属性，例如、、 `ContainsKey` `Add` `Remove` 和 `Clear` 。
  * 字典中的键是字符串，因此允许有空格。 示例： `ViewData["Some Key With Whitespace"]`
  * 任何非 `string` 类型均须在视图中进行强制转换才能使用 `ViewData`。
* `ViewBag`
  * 派生自 [DynamicViewData](/dotnet/api/microsoft.aspnetcore.mvc.viewfeatures.internal.dynamicviewdata)，因此它可使用点表示法 (`@ViewBag.SomeKey = <value or object>`) 创建动态属性，且无需强制转换。 `ViewBag` 的语法使添加到控制器和视图的速度更快。
  * 更易于检查 NULL 值。 示例： `@ViewBag.Person?.Name`

**何时使用 ViewData 或 ViewBag**

`ViewData` 和 `ViewBag` 都是在控制器和视图之间传递少量数据的有效方法。 根据偏好选择使用哪种方法。 可以混合和匹配 `ViewData` 和 `ViewBag` 对象，但是，使用一致的方法可以更轻松地读取和维护代码。 这两种方法都是在运行时进行动态解析的，因此容易造成运行时错误。 因而，一些开发团队会避免使用它们。

### <a name="dynamic-views"></a>动态视图

不使用 `@model` 声明模型类型，但有模型实例传递给它们的视图（如 `return View(Address);`）可动态引用实例的属性：

```cshtml
<address>
    @Model.Street<br>
    @Model.City, @Model.State @Model.PostalCode<br>
    <abbr title="Phone">P:</abbr> 425.555.0100
</address>
```

此功能提供了灵活性，但不提供编译保护或 IntelliSense。 如果属性不存在，则网页生成在运行时会失败。

## <a name="more-view-features"></a>更多视图功能

[标记帮助程序](xref:mvc/views/tag-helpers/intro)可以轻松地将服务器端行为添加到现有的 HTML 标记。 使用标记帮助程序可避免在视图内编写自定义代码或帮助程序。 标记帮助程序作为属性应用于 HTML 元素，并被无法处理它们的编辑器忽略。 这可让你在各种工具中编辑和呈现视图标记。

通过许多内置 HTML 帮助程序可生成自定义 HTML 标记。 通过[视图组件](xref:mvc/views/view-components)可以处理更复杂的用户界面逻辑。 视图组件提供的 SoC 与控制器和视图提供的相同。 它们无需使用处理数据（由常见用户界面元素使用）的操作和视图。

与 ASP.NET Core 的许多其他方面一样，视图支持[依赖关系注入](xref:fundamentals/dependency-injection)，可将服务[注入视图](xref:mvc/views/dependency-injection)。
