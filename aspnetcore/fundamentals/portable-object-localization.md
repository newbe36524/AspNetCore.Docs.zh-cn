---
title: 在 ASP.NET Core 中配置可移植对象本地化
author: sebastienros
description: 本文介绍可移植对象文件，并概述通过 Orchard Core 框架在 ASP.NET Core 应用程序中使用这些文件的步骤。
ms.author: scaddie
ms.date: 09/26/2017
no-loc:
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
uid: fundamentals/portable-object-localization
ms.openlocfilehash: f471c5b7511434cf42717e52ef271663c2e36647
ms.sourcegitcommit: 6ecdc481d5b9a10d2c6e091217f017b36bdba957
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90456044"
---
# <a name="configure-portable-object-localization-in-aspnet-core"></a>在 ASP.NET Core 中配置可移植对象本地化

::: moniker range=">= aspnetcore-3.0"

作者：[Sébastien Ros](https://github.com/sebastienros)、[Scott Addie](https://twitter.com/Scott_Addie) 和 [Hisham Bin Ateya](https://github.com/hishamco)

本文演示通过 [Orchard Core](https://github.com/OrchardCMS/OrchardCore) 框架在 ASP.NET Core 应用程序中使用可移植对象 (PO) 文件的步骤。

**注意：** Orchard Core 不是 Microsoft 产品。 因此，Microsoft 不提供针对此功能的支持。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/3.x/POLocalization)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="what-is-a-po-file"></a>什么是 PO 文件？

PO 文件作为包含给定语言的已转换字符串的文本文件分发。 使用 PO 文件替代 .resx 文件的一些优势包括：
- PO 文件支持复数形式；而 .resx 文件不支持复数形式。
- PO 文件的编译方法与 .resx 文件不同。 同样，无需专用工具和生成步骤。
- PO 文件可很好地与协作联机编辑工具结合使用。

### <a name="example"></a>示例

下面是一个包含两个法语字符串（其中一个具有复数形式）转换的示例 PO 文件：

fr.po

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

此示例使用下列语法：

- `#:`：用于指示要转换的字符串的上下文的注释。 根据使用的位置，可对相同字符串进行不同转换。
- `msgid`：未转换的字符串。
- `msgstr`：已转换的字符串。

在支持复数形式的情况下，可定义多个条目。

- `msgid_plural`：未转换的复数形式字符串。
- `msgstr[0]`：针对事例 0 的已转换的字符串。
- `msgstr[N]`：针对事例 N 的已转换的字符串。

可在[此处](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html)找到 PO 文件规范。

## <a name="configuring-po-file-support-in-aspnet-core"></a>在 ASP.NET Core 中配置 PO 文件支持

此示例基于从 Visual Studio 2017 项目模板中生成的 ASP.NET Core MVC 应用程序。

### <a name="referencing-the-package"></a>引用包

添加对 `OrchardCore.Localization.Core` NuGet 包的引用。 它可在 [MyGet](https://www.myget.org/) 上的以下包源中获得： https://www.myget.org/F/orchardcore-preview/api/v3/index.json

.csproj 文件现在包含类似于以下内容的行（版本号可能不同）：

[!code-xml[](localization/sample/3.x/POLocalization/POLocalization.csproj?range=8)]

### <a name="registering-the-service"></a>注册服务

将所需服务添加到 Startup.cs 的 `ConfigureServices` 方法：

[!code-csharp[](localization/sample/3.x/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

将所需中间件添加到 Startup.cs 的 `Configure` 方法：

[!code-csharp[](localization/sample/3.x/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

将以下代码添加到所选的 Razor 视图中。 在此示例中，使用了 About.cshtml。

[!code-cshtml[](localization/sample/3.x/POLocalization/Views/Home/About.cshtml)]

注入了 `IViewLocalizer` 实例并将其用于转换文本“Hello world！”。

### <a name="creating-a-po-file"></a>创建 PO 文件

在应用程序根文件夹中创建名为 \<culture code>.po 的文件。 在此示例中，文件名为 fr.po，因为使用了法语：

[!code-text[](localization/sample/3.x/POLocalization/fr.po)]

此文件存储了要转换的字符串和已转换为法语的字符串。 如有必要，转换将还原为其父级区域性。 在此示例中，如果请求的区域性为 `fr-FR` 或 `fr-CA`，则使用 fr.po 文件。

### <a name="testing-the-application"></a>测试应用程序

运行应用程序并导航到 URL `/Home/About`。 此时将显示文本 Hello world! 。

导航到 URL `/Home/About?culture=fr-FR`。 此时将显示文本 Bonjour le monde! 。

## <a name="pluralization"></a>复数形式

PO 文件支持复数形式，在相同字符串需要基于基数以不同方式进行转换时，这非常有用。 此任务较为复杂，因为每种语言均定义了自定义规则，以基于基数选择要使用的字符串。

Orchard 本地化包提供了一个 API 以自动调用这些不同的复数形式。

### <a name="creating-pluralization-po-files"></a>创建复数形式 PO 文件

将以下内容添加到前面所述的 fr.po 文件：

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

有关此示例中每个条目所表示的内容的说明，请参阅[什么是 PO 文件？](#what-is-a-po-file)。

### <a name="adding-a-language-using-different-pluralization-forms"></a>使用不同的复数形式添加语言

前面的示例中使用了英语和法语字符串。 英语和法语只有两种复数形式且拥有相同形式规则，即一的基数映射到第一种复数形式。 任何其他基数映射到第二种复数形式。

并非所有语言都拥有相同的规则。 捷克语就是一个例子，它具有三种复数形式。

如下所示，创建 `cs.po` 文件，并记下复数形式如何需要三种不同转换：

[!code-text[](localization/sample/3.x/POLocalization/cs.po)]

若要接受捷克语本地化，请将 `"cs"` 添加到 `ConfigureServices` 方法中受支持的区域性列表：

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

编辑 Views/Home/About.cshtml 文件以呈现一些基数的已本地化复数形式字符串：

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

**注意：** 在实际方案中，将使用变量表示计数。 此处，我们通过三个不同的值重复相同代码，以公开非常特定的事例。

切换区域性时，将显示如下内容：

对于 `/Home/About`：

```html
There is one item.
There are 2 items.
There are 5 items.
```

对于 `/Home/About?culture=fr`：

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

对于 `/Home/About?culture=cs`：

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

请注意，对于捷克语区域性，这三种转换各不相同。 对于最后两个已转换字符串，法语和英语区域性具有相同构造。

## <a name="advanced-tasks"></a>高级任务

### <a name="contextualizing-strings"></a>将字符串置于上下文中理解

应用程序通常包含要在多个位置中进行转换的字符串。 在应用中的特定位置（Razor 视图或类文件），相同字符串可能具有不同转换。 PO 文件支持文件上下文概念，此概念可用于对所表示的字符串进行分类。 使用文件上下文，可将字符串进行不同转换，具体取决于文件上下文（或缺乏文件上下文）。

PO 本地化服务使用完整类的名称或转换字符串时使用的视图。 这通过在 `msgctxt` 条目上设置值来完成。

请考虑对以前的 fr.po 示例作一点小小的补充。 可通过设置保留的 `msgctxt` 条目的值将位于 Views/Home/About.cshtml 的 Razor 视图定义为文件上下文：

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

这样设置 `msgctxt` 后，导航到 `/Home/About?culture=fr-FR` 时将发生文本转换。 而导航到 `/Home/Contact?culture=fr-FR` 时，则不发生转换。

当没有特定条目与给定文件上下文相匹配时，Orchard Core 的回退机制将在没有上下文的情况下查找适当的 PO 文件。 假设不存在针对 Views/Home/Contact.cshtml 定义的特定文件上下文，导航到 `/Home/Contact?culture=fr-FR`，加载 PO 文件，如：

[!code-text[](localization/sample/3.x/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a>更改 PO 文件的位置

可以在 `ConfigureServices` 中更改 PO 文件的默认位置：

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

在此示例中，从本地化文件夹加载 PO 文件。

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a>实现用于查找本地化文件的自定义逻辑

当需要更复杂的逻辑以查找 PO 文件时，可实现 `OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` 接口并将其注册为服务。 在可将 PO 文件存储于不同位置或在文件夹层次结构中找到文件时，这非常有用。

### <a name="using-a-different-default-pluralized-language"></a>使用不同默认复数形式语言

此包包含特定于两种复数形式的 `Plural` 扩展方法。 对于需要更多复数形式的语言，请创建扩展方法。 通过扩展方法，无需提供默认语言的任何本地化文件 &mdash; 可在代码中直接使用原始字符串。

可使用更加广泛的、接受转换的字符串数组的 `Plural(int count, string[] pluralForms, params object[] arguments)` 重载。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Sébastien Ros](https://github.com/sebastienros) 和 [Scott Addie](https://twitter.com/Scott_Addie)

本文演示通过 [Orchard Core](https://github.com/OrchardCMS/OrchardCore) 框架在 ASP.NET Core 应用程序中使用可移植对象 (PO) 文件的步骤。

**注意：** Orchard Core 不是 Microsoft 产品。 因此，Microsoft 不提供针对此功能的支持。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/2.x/POLocalization)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="what-is-a-po-file"></a>什么是 PO 文件？

PO 文件作为包含给定语言的已转换字符串的文本文件分发。 使用 PO 文件替代 .resx 文件的一些优势包括：
- PO 文件支持复数形式；而 .resx 文件不支持复数形式。
- PO 文件的编译方法与 .resx 文件不同。 同样，无需专用工具和生成步骤。
- PO 文件可很好地与协作联机编辑工具结合使用。

### <a name="example"></a>示例

下面是一个包含两个法语字符串（其中一个具有复数形式）转换的示例 PO 文件：

fr.po

```text
#: Services/EmailService.cs:29
msgid "Enter a comma separated list of email addresses."
msgstr "Entrez une liste d'emails séparés par une virgule."

#: Views/Email.cshtml:112
msgid "The email address is \"{0}\"."
msgid_plural "The email addresses are \"{0}\"."
msgstr[0] "L'adresse email est \"{0}\"."
msgstr[1] "Les adresses email sont \"{0}\""
```

此示例使用下列语法：

- `#:`：用于指示要转换的字符串的上下文的注释。 根据使用的位置，可对相同字符串进行不同转换。
- `msgid`：未转换的字符串。
- `msgstr`：已转换的字符串。

在支持复数形式的情况下，可定义多个条目。

- `msgid_plural`：未转换的复数形式字符串。
- `msgstr[0]`：针对事例 0 的已转换的字符串。
- `msgstr[N]`：针对事例 N 的已转换的字符串。

可在[此处](https://www.gnu.org/savannah-checkouts/gnu/gettext/manual/html_node/PO-Files.html)找到 PO 文件规范。

## <a name="configuring-po-file-support-in-aspnet-core"></a>在 ASP.NET Core 中配置 PO 文件支持

此示例基于从 Visual Studio 2017 项目模板中生成的 ASP.NET Core MVC 应用程序。

### <a name="referencing-the-package"></a>引用包

添加对 `OrchardCore.Localization.Core` NuGet 包的引用。 它可在 [MyGet](https://www.myget.org/) 上的以下包源中获得： https://www.myget.org/F/orchardcore-preview/api/v3/index.json

.csproj 文件现在包含类似于以下内容的行（版本号可能不同）：

[!code-xml[](localization/sample/2.x/POLocalization/POLocalization.csproj?range=9)]

### <a name="registering-the-service"></a>注册服务

将所需服务添加到 Startup.cs 的 `ConfigureServices` 方法：

[!code-csharp[](localization/sample/2.x/POLocalization/Startup.cs?name=snippet_ConfigureServices&highlight=4-21)]

将所需中间件添加到 Startup.cs 的 `Configure` 方法：

[!code-csharp[](localization/sample/2.x/POLocalization/Startup.cs?name=snippet_Configure&highlight=15)]

将以下代码添加到所选的 Razor 视图中。 在此示例中，使用了 About.cshtml。

[!code-cshtml[](localization/sample/2.x/POLocalization/Views/Home/About.cshtml)]

注入了 `IViewLocalizer` 实例并将其用于转换文本“Hello world！”。

### <a name="creating-a-po-file"></a>创建 PO 文件

在应用程序根文件夹中创建名为 \<culture code>.po 的文件。 在此示例中，文件名为 fr.po，因为使用了法语：

[!code-text[](localization/sample/2.x/POLocalization/fr.po)]

此文件存储了要转换的字符串和已转换为法语的字符串。 如有必要，转换将还原为其父级区域性。 在此示例中，如果请求的区域性为 `fr-FR` 或 `fr-CA`，则使用 fr.po 文件。

### <a name="testing-the-application"></a>测试应用程序

运行应用程序并导航到 URL `/Home/About`。 此时将显示文本 Hello world! 。

导航到 URL `/Home/About?culture=fr-FR`。 此时将显示文本 Bonjour le monde! 。

## <a name="pluralization"></a>复数形式

PO 文件支持复数形式，在相同字符串需要基于基数以不同方式进行转换时，这非常有用。 此任务较为复杂，因为每种语言均定义了自定义规则，以基于基数选择要使用的字符串。

Orchard 本地化包提供了一个 API 以自动调用这些不同的复数形式。

### <a name="creating-pluralization-po-files"></a>创建复数形式 PO 文件

将以下内容添加到前面所述的 fr.po 文件：

```text
msgid "There is one item."
msgid_plural "There are {0} items."
msgstr[0] "Il y a un élément."
msgstr[1] "Il y a {0} éléments."
```

有关此示例中每个条目所表示的内容的说明，请参阅[什么是 PO 文件？](#what-is-a-po-file)。

### <a name="adding-a-language-using-different-pluralization-forms"></a>使用不同的复数形式添加语言

前面的示例中使用了英语和法语字符串。 英语和法语只有两种复数形式且拥有相同形式规则，即一的基数映射到第一种复数形式。 任何其他基数映射到第二种复数形式。

并非所有语言都拥有相同的规则。 捷克语就是一个例子，它具有三种复数形式。

如下所示，创建 `cs.po` 文件，并记下复数形式如何需要三种不同转换：

[!code-text[](localization/sample/2.x/POLocalization/cs.po)]

若要接受捷克语本地化，请将 `"cs"` 添加到 `ConfigureServices` 方法中受支持的区域性列表：

```csharp
var supportedCultures = new List<CultureInfo>
{
    new CultureInfo("en-US"),
    new CultureInfo("en"),
    new CultureInfo("fr-FR"),
    new CultureInfo("fr"),
    new CultureInfo("cs")
};
```

编辑 Views/Home/About.cshtml 文件以呈现一些基数的已本地化复数形式字符串：

```cshtml
<p>@Localizer.Plural(1, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(2, "There is one item.", "There are {0} items.")</p>
<p>@Localizer.Plural(5, "There is one item.", "There are {0} items.")</p>
```

**注意：** 在实际方案中，将使用变量表示计数。 此处，我们通过三个不同的值重复相同代码，以公开非常特定的事例。

切换区域性时，将显示如下内容：

对于 `/Home/About`：

```html
There is one item.
There are 2 items.
There are 5 items.
```

对于 `/Home/About?culture=fr`：

```html
Il y a un élément.
Il y a 2 éléments.
Il y a 5 éléments.
```

对于 `/Home/About?culture=cs`：

```html
Existuje jedna položka.
Existují 2 položky.
Existuje 5 položek.
```

请注意，对于捷克语区域性，这三种转换各不相同。 对于最后两个已转换字符串，法语和英语区域性具有相同构造。

## <a name="advanced-tasks"></a>高级任务

### <a name="contextualizing-strings"></a>将字符串置于上下文中理解

应用程序通常包含要在多个位置中进行转换的字符串。 在应用中的特定位置（Razor 视图或类文件），相同字符串可能具有不同转换。 PO 文件支持文件上下文概念，此概念可用于对所表示的字符串进行分类。 使用文件上下文，可将字符串进行不同转换，具体取决于文件上下文（或缺乏文件上下文）。

PO 本地化服务使用完整类的名称或转换字符串时使用的视图。 这通过在 `msgctxt` 条目上设置值来完成。

请考虑对以前的 fr.po 示例作一点小小的补充。 可通过设置保留的 `msgctxt` 条目的值将位于 Views/Home/About.cshtml 的 Razor 视图定义为文件上下文：

```text
msgctxt "Views.Home.About"
msgid "Hello world!"
msgstr "Bonjour le monde!"
```

这样设置 `msgctxt` 后，导航到 `/Home/About?culture=fr-FR` 时将发生文本转换。 而导航到 `/Home/Contact?culture=fr-FR` 时，则不发生转换。

当没有特定条目与给定文件上下文相匹配时，Orchard Core 的回退机制将在没有上下文的情况下查找适当的 PO 文件。 假设不存在针对 Views/Home/Contact.cshtml 定义的特定文件上下文，导航到 `/Home/Contact?culture=fr-FR`，加载 PO 文件，如：

[!code-text[](localization/sample/2.x/POLocalization/fr.po)]

### <a name="changing-the-location-of-po-files"></a>更改 PO 文件的位置

可以在 `ConfigureServices` 中更改 PO 文件的默认位置：

```csharp
services.AddPortableObjectLocalization(options => options.ResourcesPath = "Localization");
```

在此示例中，从本地化文件夹加载 PO 文件。

### <a name="implementing-a-custom-logic-for-finding-localization-files"></a>实现用于查找本地化文件的自定义逻辑

当需要更复杂的逻辑以查找 PO 文件时，可实现 `OrchardCore.Localization.PortableObject.ILocalizationFileLocationProvider` 接口并将其注册为服务。 在可将 PO 文件存储于不同位置或在文件夹层次结构中找到文件时，这非常有用。

### <a name="using-a-different-default-pluralized-language"></a>使用不同默认复数形式语言

此包包含特定于两种复数形式的 `Plural` 扩展方法。 对于需要更多复数形式的语言，请创建扩展方法。 通过扩展方法，无需提供默认语言的任何本地化文件 &mdash; 可在代码中直接使用原始字符串。

可使用更加广泛的、接受转换的字符串数组的 `Plural(int count, string[] pluralForms, params object[] arguments)` 重载。

::: moniker-end