---
title: 在 ASP.NET Core 中创作标记帮助程序
author: rick-anderson
description: 了解如何在 ASP.NET Core 中创作标记帮助程序。
ms.author: riande
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
uid: mvc/views/tag-helpers/authoring
ms.openlocfilehash: 306416db3d9ae0219f859c3cf459eb08a5b778cf
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060919"
---
# <a name="author-tag-helpers-in-aspnet-core"></a>在 ASP.NET Core 中创作标记帮助程序

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/views/tag-helpers/authoring/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="get-started-with-tag-helpers"></a>标记帮助程序入门

本教程介绍标记帮助程序编程。 [标记帮助程序简介](intro.md)描述了标记帮助程序提供的优势。

标记帮助程序是实现 `ITagHelper` 接口的任何类。 但是，在创作标记帮助程序时，通常从 `TagHelper` 派生，这样可以访问 `Process` 方法。

1. 创建一个名为 AuthoringTagHelpers 的新 ASP.NET Core 项目  。 此项目不需要身份验证。

1. 创建一个名为“TagHelpers”的文件夹来保存标记帮助程序  。 “TagHelpers”文件夹不是必需的，但它是合理的约定  。 现在让我们开始编写一些简单的标记帮助程序。

## <a name="a-minimal-tag-helper"></a>最小的标记帮助程序

在本部分中，你将编写一个更新电子邮件标记的标记帮助程序。 例如：

```html
<email>Support</email>
```

服务器将使用电子邮件标记帮助程序将该标记转换为以下内容：

```html
<a href="mailto:Support@contoso.com">Support@contoso.com</a>
```

即，使其成为电子邮件链接的定位标记。 如果你正在编写博客引擎，并且需要它将营销、支持和其他联系人的电子邮件全部发送到同一个域，则可能需要执行此操作。

1. 将以下 `EmailTagHelper` 类添加到“TagHelpers”文件夹  。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1EmailTagHelperCopy.cs)]

   * 标记帮助程序使用面向根类名称的元素的命名约定（减去类名称的 TagHelper 部分）  。 在此示例中， **EmailTagHelper** 的根名称为 *email* ，因此 `<email>` 标记将针对。 此命名约定应适用于大多数标记帮助程序，稍后将介绍如何重写它。

   * `EmailTagHelper` 类从 `TagHelper` 派生。 `TagHelper` 类提供编写标记帮助程序的方法和属性。

   * 重写的 `Process` 方法控制标记帮助程序在执行时的操作。 `TagHelper` 类还提供具有相同参数的异步版本 (`ProcessAsync`)。

   * `Process`（和 `ProcessAsync`）的上下文参数包含与执行当前 HTML 标记相关的信息。

   * `Process`（和 `ProcessAsync`）的输出参数包含监控状态的 HTML 元素，它代表用于生成 HTML 标记和内容的原始源。

   * 类名称的后缀是 TagHelper，这不是必需的，但被认为是最佳做法约定  。 可将类声明为：

   ```csharp
   public class Email : TagHelper
   ```

1. 若要使 `EmailTagHelper` 类可用于所有 Razor 视图，请将 `addTagHelper` 指令添加到 *views/_ViewImports cshtml* 文件中：

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImportsCopyEmail.cshtml?highlight=2,3)]

   上面的代码使用通配符语法来指定程序集中的所有标记帮助程序都将可用。 `@addTagHelper` 之后的第一个字符串指定要加载的标记帮助程序（对所有标记帮助程序使用“*”），第二个字符串“AuthoringTagHelpers”指定标记帮助程序所在的程序集。 另请注意，第二行将使用通配符语法的 ASP.NET Core MVC 标记帮助程序 (在 [标记帮助程序简介](intro.md)中讨论了这些帮助程序。 ) 这是 `@addTagHelper` 使标记帮助程序可用于该视图的指令 Razor 。 或者，也可以提供标记帮助程序的完全限定的名称 (FQN)，如下所示：

```csharp
@using AuthoringTagHelpers
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper AuthoringTagHelpers.TagHelpers.EmailTagHelper, AuthoringTagHelpers
```

<!--
the following snippet uses TagHelpers3 and should use TagHelpers (not the 3)
    [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/_ViewImports.cshtml?highlight=3&range=1-3)]
-->

若要使用 FQN 将标记帮助程序添加到视图中，请依次添加 FQN (`AuthoringTagHelpers.TagHelpers.EmailTagHelper`) 和程序集名称  （AuthoringTagHelpers  ，不一定是 `namespace`）。 大多数开发者更喜欢使用通配符语法。 [标记帮助程序简介](intro.md)详细介绍了标记帮助程序的添加和删除方法，以及层次结构和通配符语法。

1. 使用以下更改更新 Views/Home/Contact.cshtml 文件中的标记： 

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. 运行应用并使用你喜爱的浏览器来查看 HTML 源，以便验证电子邮件标记是否替换为定位标记（例如，`<a>Support</a>`）。 Support 和 Marketing 呈现为链接，但它们不具备使其正常工作的 `href` 属性  。 此问题将在下一部分得以解决。

## <a name="setattribute-and-setcontent"></a>SetAttribute 和 SetContent

在本部分中，我们将更新 `EmailTagHelper`，使其能够为电子邮件创建有效的定位标记。 我们将对其进行更新，以 Razor 使用属性) 形式从视图中获取信息 `mail-to` ，并在生成定位点时使用该 (。

使用以下内容更新 `EmailTagHelper` 类：

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?range=6-22)]

* 标记帮助程序采用 Pascal 大小写格式的类和属性名将转换为各自相应的[短横线格式](https://stackoverflow.com/questions/11273282/whats-the-name-for-dash-separated-case/12273101)。 因此，要使用 `MailTo` 属性，请使用 `<email mail-to="value"/>` 等效项。

* 最后一行为最小功能标记帮助程序设置已完成的内容。

* 突出显示的行显示了添加属性的语法：

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailTo.cs?highlight=6&range=14-21)]

只要属性集合中当前不存在“href”属性，该方法就适用于此属性。 也可使用 `output.Attributes.Add` 方法将标记帮助程序属性添加到标记属性集合的末尾。

1. 使用以下更改更新 Views/Home/Contact.cshtml 文件中的标记： 

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/ContactCopy.cshtml?highlight=15,16)]

1. 运行应用并验证它是否生成正确的链接。

<a name="self-closing"></a>

   > [!NOTE]
   > 如果打算编写电子邮件标记自结束 (`<email mail-to="Rick" />`)，最终输出也将为自结束。 要启用只使用开始标记 (`<email mail-to="Rick">`) 来编写标记的功能，必须用以下内容来标记类：
   >
   > [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelperMailVoid.cs?highlight=1&range=6-10)]

   鉴于自结束电子邮件标记帮助程序，输出将为 `<a href="mailto:Rick@contoso.com" />`。 自结束定位标记不是有效的 HTML，因此你不想创建这样的标记，但你可能想要创建一个自结束的标记帮助程序。 标记帮助程序在读取标记后设置 `TagMode` 属性的类型。

### <a name="processasync"></a>ProcessAsync

在本部分中，我们将编写异步电子邮件帮助程序。

1. 将 `EmailTagHelper` 类替换为以下代码：

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/EmailTagHelper.cs?range=6-17)]

   **注意：**

   * 此版本使用异步 `ProcessAsync` 方法。 异步 `GetChildContentAsync` 返回包含 `TagHelperContent` 的 `Task`。

   * 使用 `output` 参数获取 HTML 元素的内容。

1. 对 Views/Home/Contact.cshtml 文件进行以下更改，以便标记帮助程序能够获取目标电子邮件  。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=15,16&range=1-17)]

1. 运行应用并验证它是否生成有效的电子邮件链接。

### <a name="removeall-precontentsethtmlcontent-and-postcontentsethtmlcontent"></a>RemoveAll、PreContent.SetHtmlContent 和 PostContent.SetHtmlContent

1. 将以下 `BoldTagHelper` 类添加到“TagHelpers”文件夹  。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/BoldTagHelper.cs)]

   * `[HtmlTargetElement]` 属性传递一个属性参数，该参数指定包含名为“bold”的 HTML 属性的任何 HTML 元素都将匹配，并且该类中的 `Process` 重写方法将会运行。 在此示例中，`Process` 方法删除了“bold”属性，并用 `<strong></strong>` 围住了包含的标记。

   * 因为你不想替换现有的标记内容，所以必须用 `PreContent.SetHtmlContent` 方法编写开头的 `<strong>` 标记，并用 `PostContent.SetHtmlContent` 方法编写结尾的 `</strong>` 标记。

1. 修改 About.cshtml 视图，以包含 `bold` 属性值  。 完成的代码如下所示。

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutBoldOnly.cshtml?highlight=7)]

1. 运行应用。 可以使用你喜爱的浏览器来检查源并验证标记。

   上面的 `[HtmlTargetElement]` 属性仅针对提供属性名称“bold”的 HTML 标记。 标记帮助程序未修改 `<bold>` 元素。

1. 标注出 `[HtmlTargetElement]` 属性行，它将默认为目标 `<bold>` 标记，也就是 `<bold>` 格式的 HTML 标记。 请记住，默认的命名约定会将类名称 BoldTagHelper 与 `<bold>` 标记相匹配  。

1. 运行应用并验证 `<bold>` 标记是否由标记帮助程序进行处理。

用多个 `[HtmlTargetElement]` 属性修饰类会导致目标出现逻辑 OR。 例如，使用下面的代码时，系统将匹配出 bold 标记或 bold 属性。

[!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zBoldTagHelperCopy.cs?highlight=1,2&range=5-15)]

将多个属性添加到同一语句时，运行时会将其视为逻辑 AND。 例如，在下面的代码中，HTML 元素必须命名为“bold”并具有名为“bold”的属性 (`<bold bold />`) 才能匹配。

```csharp
[HtmlTargetElement("bold", Attributes = "bold")]
   ```

也可使用 `[HtmlTargetElement]` 更改目标元素的名称。 例如，如果你希望 `BoldTagHelper` 以 `<MyBold>` 标记为目标，则可使用以下属性：

```csharp
[HtmlTargetElement("MyBold")]
```

## <a name="pass-a-model-to-a-tag-helper"></a>将模型传递到标记帮助程序

1. 添加“Models”文件夹  。

1. 将以下 `WebsiteContext` 类添加到“模型”文件夹：

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Models/WebsiteContext.cs)]

1. 将以下 `WebsiteInformationTagHelper` 类添加到“TagHelpers”文件夹  。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/WebsiteInformationTagHelper.cs)]

   * 如前所述，标记帮助程序会将标记帮助程序采用 Pascal 大小写格式的 C# 类名和属性转换为[短横线格式](https://wiki.c2.com/?KebabCase)。 因此，若要 `WebsiteInformationTagHelper` 在中使用 Razor ，你将编写 `<website-information />` 。

   * 未显式标识具有 `[HtmlTargetElement]` 属性的目标元素，因此 `website-information` 的默认值将成为目标元素。 如果应用了以下属性（请注意，它虽不是短横线格式，但却与类名相匹配）：

   ```csharp
   [HtmlTargetElement("WebsiteInformation")]
   ```

   短横线格式标记 `<website-information />` 不匹配。 若要使用 `[HtmlTargetElement]` 属性，请使用短横线格式，如下所示：

   ```csharp
   [HtmlTargetElement("Website-Information")]
   ```

   * 自结束的元素没有任何内容。 在此示例中， Razor 标记将使用一个自结束标记，但标记帮助程序将创建一个 [节](https://www.w3.org/TR/html5/sections.html#the-section-element) 元素，该 (元素不会自行关闭，并且你在) 元素内编写内容 `section` 。 因此，需要将 `TagMode` 设置为 `StartTagAndEndTag` 以写入输出。 或者，可以标注出行设置 `TagMode` 并用结束标记编写标记。 （本教程后面将提供示例标记。）

   * 下一行中的 `$`（美元符号）使用[内插字符串](/dotnet/csharp/language-reference/keywords/interpolated-strings)：

   ```cshtml
   $@"<ul><li><strong>Version:</strong> {Info.Version}</li>
   ```

1. 将以下标记添加到 About.cshtml 视图  。 突出显示的标记显示 Web 站点信息。

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?highlight=1,4-8, 18-999)]

   > [!NOTE]
   > 在 Razor 下面显示的标记中：
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/About.cshtml?range=18-18)]
   >
   > Razor 知道 `info` 属性是一个类，而不是字符串，并且您想要编写 c # 代码。 编写任何非字符串标记帮助程序属性时，都不应使用 `@` 字符。

1. 运行应用，并导航到“关于”视图查看 Web 站点信息。

   > [!NOTE]
   > 可使用带有结束标记的以下标记，并在标记帮助程序中删除带有 `TagMode.StartTagAndEndTag` 的行：
   >
   > [!code-html[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/AboutNotSelfClosing.cshtml?range=20-21)]

## <a name="condition-tag-helper"></a>条件标记帮助程序

条件标记帮助程序在传递 true 值时呈现输出。

1. 将以下 `ConditionTagHelper` 类添加到“TagHelpers”文件夹  。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/ConditionTagHelper.cs)]

1. 将 Views/Home/Index.cshtml 文件的内容  替换为以下标记：

   [!code-cshtml[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Index.cshtml)]

1. 将 `Home` 控制器中的 `Index` 方法替换为以下代码：

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Controllers/HomeController.cs?range=9-18)]

1. 运行应用并浏览到主页。 条件 `div` 中的标记不会呈现。 将查询字符串 `?approved=true` 追加到 URL（例如，`http://localhost:1235/Home/Index?approved=true`）。 `approved` 设置为 true，并将显示条件标记。

> [!NOTE]
> 使用 [nameof](/dotnet/csharp/language-reference/keywords/nameof) 运算符将属性指定为目标，而不是像使用 bold 标记帮助程序那样指定字符串：
>
> [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/zConditionTagHelperCopy.cs?highlight=1,2,5&range=5-18)]
>
> 如果代码被重构，[nameof](/dotnet/csharp/language-reference/keywords/nameof) 运算符将保护它（可能需要将名称更改为 `RedCondition`）。

### <a name="avoid-tag-helper-conflicts"></a>避免标记帮助程序冲突

在本部分中，你将编写一对自动链接标记帮助程序。 第一个标记帮助程序会将包含以 HTTP 开头的 URL 的标记替换为包含相同 URL 的 HTML 定位标记（从而产生指向 URL 的链接）。 第二个标记帮助程序也会对以 WWW 开头的 URL 执行相同的操作。

由于这两个帮助程序密切相关，并且你将来可能会重构它们，因此可将其保存在同一文件中。

1. 将以下 `AutoLinkerHttpTagHelper` 类添加到“TagHelpers”文件夹  。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=7-19)]

   >[!NOTE]
   >`AutoLinkerHttpTagHelper` 类以 `p` 元 素为目标，并使用[正则表达式](/dotnet/standard/base-types/regular-expression-language-quick-reference)来创建定位点。

1. 将以下标记添加到 Views/Home/Contact.cshtml 文件的末尾  ：

   [!code-cshtml[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/Views/Home/Contact.cshtml?highlight=19)]

1. 运行应用并验证标记帮助程序是否正确呈现定位点。

1. 更新 `AutoLinker` 类以包含 `AutoLinkerWwwTagHelper`，这会将 www 文本转换为还包含原始 www 文本的定位标记。 更新后的代码在下方突出显示：

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?highlight=15-34&range=7-34)]

1. 运行应用。 请注意 www 文本呈现为链接，但 HTTP 文本不是。 如果将中断点放在这两个类中，可以看到 HTTP 标记帮助程序类首先运行。 问题是，标记帮助程序输出已缓存，当运行 WWW 标记帮助程序时，它会覆盖 HTTP 标记帮助程序的缓存输出。 本教程稍后将介绍如何控制标记帮助程序中的运行顺序。 我们将用以下方法修复代码：

   [!code-csharp[](../../../mvc/views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10,21,22,26&range=8-37)]

   > [!NOTE]
   > 在自动链接标记帮助程序的第一版中，使用以下代码获取了目标的内容：
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinker.cs?range=12)]
   >
   > 也就是说，使用传递到 `ProcessAsync` 方法的 `TagHelperOutput` 调用 `GetChildContentAsync`。 如前所述，由于输出已缓存，因此将调用最后运行的标记帮助程序。 使用以下代码解决了这个问题：
   >
   > [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?range=34-35)]
   >
   > 上面的代码检查内容是否已修改，如果已修改，则从输出缓冲区获取内容。

1. 运行应用并验证这两个链接是否按预期工作。 虽然它可能显示自动链接器标记帮助程序是正确且完整的，但它有一个细微的问题。 如果首先运行 WWW 标记帮助程序，则 www 链接不正确。 通过添加 `Order` 重载更新代码来控制标记运行的顺序。 `Order` 属性确定相对于面向同一元素的其他标记帮助程序的执行顺序。 默认顺序值为零，并首先执行具有较低值的实例。

   [!code-csharp[](authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z2AutoLinkerCopy.cs?highlight=5,6,7,8&range=8-15)]

   上面的代码可以保证 HTTP 标记帮助程序在 WWW 标记帮助程序之前运行。 将 `Order` 更改为 `MaxValue` 并验证为 WWW 标记生成的标记是否不正确。

## <a name="inspect-and-retrieve-child-content"></a>检查和检索子内容

标记帮助程序提供多个属性来检索内容。

* 可将 `GetChildContentAsync` 的结果追加到 `output.Content`。
* 可使用 `GetContent` 检查 `GetChildContentAsync` 的结果。
* 如果修改 `output.Content`，则不会执行或呈现 TagHelper 主体，除非像自动链接器示例中那样调用 `GetChildContentAsync`：

[!code-csharp[](../../views/tag-helpers/authoring/sample/AuthoringTagHelpers/src/AuthoringTagHelpers/TagHelpers/z1AutoLinkerCopy.cs?highlight=5,6,10&range=8-21)]

* 对 `GetChildContentAsync` 的多次调用返回相同的值，且不重新执行 `TagHelper` 主体，除非传入一个指示不使用缓存结果的 false 参数。

## <a name="load-minified-partial-view-taghelper"></a>加载缩小的分部视图 TagHelper

在生产环境中，可以通过加载缩小的分部视图来提升性能。 若要在生产环境中利用缩小的分部视图，请执行以下操作：

* 创建/设置缩小分部视图的预生成过程。
* 使用以下代码在非开发环境中加载缩小的分部视图。

[!code-csharp[](authoring/sample/AuthoringTagHelpers/src/MinifiedVersionTagHelper.cs?name=snippet)]
