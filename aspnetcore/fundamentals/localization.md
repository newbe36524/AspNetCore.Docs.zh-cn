---
title: ASP.NET Core 全球化和本地化
author: rick-anderson
description: 了解 ASP.NET Core 如何提供服务和中间件，将内容本地化为不同的语言和区域性。
ms.author: riande
ms.date: 11/30/2019
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
uid: fundamentals/localization
ms.openlocfilehash: fcf69bdaaed5cf0283ae27440c28061857d2cbcb
ms.sourcegitcommit: d1a897ebd89daa05170ac448e4831d327f6b21a8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2020
ms.locfileid: "91606780"
---
# <a name="globalization-and-localization-in-aspnet-core"></a>ASP.NET Core 全球化和本地化

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)、[Damien Bowden](https://twitter.com/damien_bod)、[Bart Calixto](https://twitter.com/bartmax)、[Nadeem Afana](https://afana.me/) 和 [Hisham Bin Ateya](https://twitter.com/hishambinateya)

多语言网站使网站可以覆盖更广泛的受众。 ASP.NET Core 提供的服务和中间件可将网站本地化为不同的语言和文化。

国际化涉及[全球化](/dotnet/api/system.globalization)和[本地化](/dotnet/standard/globalization-localization/localization)。 全球化是设计支持不同区域性的应用程序的过程。 全球化添加了对一组有关特定地理区域的已定义语言脚本的输入、显示和输出支持。

本地化是将已经针对可本地化性进行处理的全球化应用调整为特定的区域性/区域设置的过程。 有关详细信息，请参阅本文档邻近末尾的全球化和本地化术语。

应用本地化涉及以下内容：

1. 使应用内容可本地化
1. 为支持的语言和区域性提供本地化资源
1. 实施策略，为每个请求选择语言/区域性

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/3.x/Localization)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="make-the-apps-content-localizable"></a>使应用内容可本地化

已为 <xref:Microsoft.Extensions.Localization.IStringLocalizer> 和 <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> 设置架构，可以为开发本地化应用提高工作效率。 `IStringLocalizer` 使用 <xref:System.Resources.ResourceManager> 和 <xref:System.Resources.ResourceReader> 在运行时提供特定于区域性的资源。 接口具有一个索引器和一个用于返回本地化字符串的 `IEnumerable`。 `IStringLocalizer` 不要求在资源文件中存储默认语言字符串。 你可以开发针对本地化的应用，且无需在开发初期创建资源资源文件。 下面的代码演示如何针对本地化包装字符串“About Title”。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

在前面的代码中，`IStringLocalizer<T>` 实现来源于[依赖关系注入](dependency-injection.md)。 如果找不到“About Title”的本地化值，则返回索引器键，即字符串“About Title”。 可将默认语言文本字符串保留在应用中并将它们包装在本地化工具中，以便你可集中精力开发应用。 你使用默认语言开发应用，并针对本地化步骤进行准备，而无需首先创建默认资源文件。 或者，你可以使用传统方法，并提供键以检索默认语言字符串。 对于许多开发者而言，不具有默认语言 .resx 文件且简单包装字符串文本的新工作流可以减少本地化应用的开销。 其他开发者将首选传统工作流，因为它可以更轻松地使用较长字符串文本，更轻松地更新本地化字符串。

对包含 HTML 的资源使用 `IHtmlLocalizer<T>` 实现。 `IHtmlLocalizer` 对资源字符串中格式化的参数进行 HTML 编码，但不对资源字符串本身进行 HTML 编码。 在下面突出显示的示例中，仅 `name` 参数的值被 HTML 编码。

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 通常，仅本地化文本，而不是 HTML。

最低程度，你可以从[依赖关系注入](dependency-injection.md)获取 `IStringLocalizerFactory`：

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

上面的代码演示了这两种工厂创建方法。

可以按控制器、区域对本地化字符串分区，或只有一个容器。 在示例应用中，名为 `SharedResource` 的虚拟类用于共享资源。

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

某些开发者使用 `Startup` 类，以包含全局或共享字符串。 在下面的示例中，使用 `InfoController` 和 `SharedResource` 本地化工具：

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>视图本地化

`IViewLocalizer` 服务可为[视图](xref:mvc/views/overview)提供本地化字符串。 `ViewLocalizer` 类可实现此接口，并从视图文件路径找到资源位置。 下面的代码演示如何使用 `IViewLocalizer` 的默认实现：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

`IViewLocalizer` 的默认实现可根据视图的文件名查找资源文件。 没有使用全局共享资源文件的选项。 `ViewLocalizer` 使用 `IHtmlLocalizer` 实现本地化工具，因此 Razor 不会对本地化字符串进行 HTML 编码。 你可以参数化资源字符串，`IViewLocalizer` 将对参数进行 HTML 编码，但不会对资源字符串进行。 请考虑以下 Razor 标记：

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

法语资源文件可以包含以下信息：

| 键 | “值” |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

呈现的视图可能包含资源文件中的 HTML 标记。

> [!NOTE]
> 通常，仅本地化文本，而不是 HTML。

若要在视图中使用共享资源文件，请注入 `IHtmlLocalizer<T>`：

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations 本地化

DataAnnotations 错误消息已使用 `IStringLocalizer<T>` 本地化。 使用选项 `ResourcesPath = "Resources"`，`RegisterViewModel` 中的错误消息可以存储在以下路径之一：

* *Resources/ViewModels.Account.RegisterViewModel.fr.resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

在 ASP.NET Core MVC 1.1.0 和更高版本中，非验证属性已经进行了本地化。 ASP.NET Core MVC 1.0 不会为非验证属性查找本地化字符串。

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>对多个类使用一个资源字符串

下面的代码演示如何针对具有多个类的验证属性使用一个资源字符串：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

在上面的代码中，`SharedResource` 是对应于存储验证消息的 resx 的类。 使用此方法，DataAnnotations 将仅使用 `SharedResource`，而不是每个类的资源。

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>为支持的语言和区域性提供本地化资源

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures 和 SupportedUICultures

ASP.NET Core 允许指定两个区域性值，`SupportedCultures` 和 `SupportedUICultures`。 `SupportedCultures` 的 [CultureInfo](/dotnet/api/system.globalization.cultureinfo) 对象可决定区域性相关函数的结果，如日期、时间、数字和货币格式等。 `SupportedCultures` 确定文本的排序顺序、大小写约定和字符串比较。 请参阅 [CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture) 详细了解服务器如何获取区域性。 `SupportedUICultures` 可确定按 [ResourceManager](/dotnet/api/system.resources.resourcemanager) 来查找哪些转换字符串（位于 .resx 文件）。 `ResourceManager` 只需查找 `CurrentUICulture` 决定的区域性特定字符串。 .NET 中的每个线程都具有 `CurrentCulture` 和 `CurrentUICulture` 对象。 呈现区域性相关函数时，ASP.NET Core 可检查这些值。 例如，如果当前线程的区域性设置为“en-US”（英语，美国），`DateTime.Now.ToLongDateString()` 将显示“Thursday, February 18, 2016”，但如果 `CurrentCulture` 设置为“es-ES”（西班牙语，西班牙），则输出将为“jueves，18 de febrero de 2016”。

## <a name="resource-files"></a>资源文件

资源文件是将可本地化的字符串与代码分离的有用机制。 非默认语言的转换字符串在 .resx 资源文件中单独显示。 例如，你可能想要创建包含转换字符串、名为 Welcome.es.resx 的西班牙语资源文件。 “es”是西班牙语的语言代码。 要在 Visual Studio 中创建此资源文件，请支持以下操作：

1. 在“解决方案资源管理器”中，右键单击将包含资源文件的文件夹 >“添加”>“新项”  。

   ![嵌套的上下文菜单：在“解决方案资源管理器”中，“资源”可打开上下文菜单。 “添加”可打开第二个上下文菜单，突出显示“新项”命令。](localization/_static/newi.png)

1. 在“搜索已安装的模板”框中，输入“资源”并命名该文件。

   ![“添加新项”对话框](localization/_static/res.png)

1. 在“名称”列中输入键值（本机字符串），在“值”列中输入转换字符串 。

   ![Welcome.es.resx 文件（西班牙语版 Welcome 资源文件）的单词 Hello 位于“名称”列，Hola（西班牙语版 Hello）位于“值”列](localization/_static/hola.png)

   Visual Studio 将显示 Welcome.es.resx 文件。

   ![显示 Welcome Spanish (es) 资源文件的解决方案资源管理器](localization/_static/se.png)

## <a name="resource-file-naming"></a>资源文件命名

资源名称是类的完整类型名称减去程序集名称。 例如，类 `LocalizationWebsite.Web.Startup` 的主要程序集为 `LocalizationWebsite.Web.dll` 的项目中的法语资源将命名为 Startup.fr.resx。 类 `LocalizationWebsite.Web.Controllers.HomeController` 的资源将命名为 Controllers.HomeController.fr.resx。 如果目标类的命名空间与将需要完整类型名称的程序集名称不同。 例如，在示例项目中，类型 `ExtraNamespace.Tools` 的资源将命名为 ExtraNamespace.Tools.fr.resx。

在示例项目中，`ConfigureServices` 方法将 `ResourcesPath` 设置为“资源”，因此主控制器的法语资源文件的项目相对路径是 Resources/Controllers.HomeController.fr.resx。 或者，你可以使用文件夹组织资源文件。 对于主控制器，该路径将为 Resources/Controllers/HomeController.fr.resx。 如果不使用 `ResourcesPath` 选项，.resx 文件将转到项目的基目录中。 `HomeController` 的资源文件将命名为 Controllers.HomeController.fr.resx。 是选择使用圆点还是路径命名约定，具体取决于你想如何组织资源文件。

| 资源名称 | 圆点或路径命名 |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | 圆点  |
| Resources/Controllers/HomeController.fr.resx  | 路径 |

Razor 视图中使用 `@inject IViewLocalizer` 的资源文件遵循类似的模式。 可以使用圆点命名或路径命名约定对视图的资源文件进行命名。 Razor 视图资源文件可模拟其关联视图文件的路径。 假设我们将 `ResourcesPath` 设置为“Resources”，与 *Views/Home/About.cshtml* 视图关联的法语资源文件可能是下面其中之一 ：

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

如果不使用 `ResourcesPath` 选项，视图的 .resx 文件将位于视图所在的文件夹。

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

[RootNamespace](/dotnet/api/microsoft.extensions.localization.rootnamespaceattribute?view=aspnetcore-2.1) 属性在程序集的根命名空间不同于程序集名称时，提供程序集的根命名空间。 

> [!WARNING]
> 当项目名称不是有效的 .NET 标识符时，可能会发生这种情况。 例如，`my-project-name.csproj` 将使用根命名空间 `my_project_name` 和导致此错误的程序集名称 `my-project-name`。 

如果程序集的根命名空间不同于程序集名称：

* 默认情况下无法进行本地化。
* 因程序集内搜索资源的方式导致本地化失败。 `RootNamespace` 是生成时间值，不可用于正在执行的进程。 

如果 `RootNamespace` 不同于 `AssemblyName`，请在 AssemblyInfo.cs 中包括以下内容（参数值替换为实际值）：

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

上述代码可成功解析 resx 文件。

## <a name="culture-fallback-behavior"></a>区域性回退行为

在搜索资源时，本地化会进行“区域性回退”。 从所请求的区域性开始，如果未能找到，则还原至该区域性的父区域性。 另外，[CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) 属性代表父区域性。 这通常（但并不是总是）意味着从 ISO 中移除区域签名。 例如，墨西哥的西班牙语方言为“es-MX”。 它具备一个父级“es”西班牙，没有特别指定国家/地区。

假设你的站点接收到了一个区域性为“fr-CA”的“Welcome”资源的请求。 本地化系统按顺序查找以下资源，并选择第一个匹配项：

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx*（如果 `NeutralResourcesLanguage` 为“fr-CA”）

例如，如果删除了“.fr”区域性指示符，而且已将区域性设置为“法语”，将读取默认资源文件，并本地化字符串。 对于不满足所请求区域性的情况，资源管理器可指定默认资源或回退资源。 缺少适用于请求区域性的资源时，如果只想返回键，不得具有默认资源文件。

### <a name="generate-resource-files-with-visual-studio"></a>使用 Visual Studio 生成资源文件

如果在 Visual Studio 中创建文件名没有区域性的资源文件（例如 Welcome.resx），Visual Studio 将创建一个 C# 类，并且具有每个字符串的属性。 这通常不是你想在 ASP.NET Core 中使用的。 你通常没有默认的 .resx 资源文件（没有区域性名称的 .resx 文件） 。 建议创建具有区域性名称（例如 Welcome.fr.resx）的 .resx 文件 。 创建具有区域性名称的 .resx 文件时，Visual Studio 不会生成类文件。

### <a name="add-other-cultures"></a>添加其他区域性

每个语言和区域性组合（除默认语言外）都需要唯一资源文件。 通过新建 ISO 语言代码属于名称一部分的资源文件，为不同的区域性和区域设置创建资源文件（例如，en-us、fr-ca 和 en-gb）  。 这些 ISO 编码位于文件名和 .resx 文件扩展之间，如 Welcome.es-MX.resx（西班牙语/墨西哥） 。

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>实施策略，为每个请求选择语言/区域性

### <a name="configure-localization"></a>配置本地化

通过 `Startup.ConfigureServices` 方法配置本地化：

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` 将本地化服务添加到服务容器。 上面的代码还将资源路径设置为“Resources”。

* `AddViewLocalization` 添加对本地化视图文件的支持。 在此示例视图中，本地化基于视图文件后缀。 例如，Index.fr.cshtml 文件中的“fr”。

* `AddDataAnnotationsLocalization` 添加通过 `IStringLocalizer` 抽象对本地化 `DataAnnotations` 验证消息的支持。

### <a name="localization-middleware"></a>本地化中间件

在本地化[中间件](xref:fundamentals/middleware/index)中设置有关请求的当前区域性。 在 `Startup.Configure` 方法中启用本地化中间件。 必须在中间件前面配置本地化中间件，它可能检查请求区域性（例如，`app.UseMvcWithDefaultRoute()`）。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` 初始化 `RequestLocalizationOptions` 对象。 在每个请求上，枚举了 `RequestLocalizationOptions` 的 `RequestCultureProvider` 列表，使用了可成功决定请求区域性的第一个提供程序。 默认提供程序来自 `RequestLocalizationOptions` 类：

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

默认列表从最具体到最不具体排序。 在本文的后面部分，我们将了解如何更改顺序，甚至添加一个自定义区域性提供程序。 如果没有一个提供程序可以确定请求区域性，则使用 `DefaultRequestCulture`。

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

某些应用将使用查询字符串来设置<https://docs.microsoft.com/dotnet/api/system.globalization.cultureinfo?view=netcore-3.1>。 对于使用 cookie 或接受语言标题方法的应用，向 URL 添加查询字符串有助于调试和测试代码。 默认情况下，`QueryStringRequestCultureProvider` 注册为 `RequestCultureProvider` 列表中的第一个本地化提供程序。 传递查询字符串参数 `culture` 和 `ui-culture`。 下面的示例将特定区域性（语言和区域）设置为“西班牙语/墨西哥”：

   `http://localhost:5000/?culture=es-MX&ui-culture=es-MX`

如果仅传入两种区域性之一（`culture` 或 `ui-culture`，查询字符串提供程序将使用你传入的区域性设置这两个值。 例如，仅设置区域性将同时设置 `Culture` 和 `UICulture`：

```
http://localhost:5000/?culture=es-MX
```

### <a name="no-loccookierequestcultureprovider"></a>CookieRequestCultureProvider

通常，生产应用将提供一种机制来使用 ASP.NET Core 区域性 cookie 设置区域性。 若要创建 cookie，请使用 `MakeCookieValue` 方法。

`CookieRequestCultureProvider` `DefaultCookieName` 将返回用来跟踪用户首选区域性信息的默认 cookie 名称。 默认的 cookie 名称为 `.AspNetCore.Culture`。

cookie 格式为 `c=%LANGCODE%|uic=%LANGCODE%`，其中 `c` 是 `Culture`，`uic` 是 `UICulture`，例如：

```
c=en-UK|uic=en-US
```

如果仅指定其中一个区域性信息和 UI 区域性，则指定的区域性将同时用于区域性信息和 UI 区域性。

### <a name="the-accept-language-http-header"></a>接受语言 HTTP 标题

[接受语言标题](https://www.w3.org/International/questions/qa-accept-lang-locales)在大多数浏览器中可设置，最初用于指定用户的语言。 此设置指示浏览器已设置为发送或已从基础操作系统继承的内容。 浏览器请求的接受语言 HTTP 标题不是检测用户首选语言的可靠方法（请参阅 [Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php)（在浏览器中设置首选项）。 生产应用应包括一种用户可以自定义区域性选择的方法。

### <a name="set-the-accept-language-http-header-in-ie"></a>在 IE 中设置接受语言 HTTP 标题

1. 在齿轮图标中，点击“Internet 选项”。

1. 点击“语言”。

   ![Internet 选项](localization/_static/lang.png)

1. 点击“设置语言首选项”。

1. 点击“添加语言”。

1. 添加语言。

1. 点击语言，然后点击“向上移动”。

### <a name="use-a-custom-provider"></a>使用自定义提供程序

假设你想要让客户在数据库中存储其语言和区域性。 你可以编写一个提供程序来查找用户的这些值。 下面的代码演示如何添加自定义提供程序：

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

使用 `RequestLocalizationOptions` 添加或删除本地化提供程序。

### <a name="set-the-culture-programmatically"></a>以编程方式设置区域性

[GitHub](https://github.com/aspnet/entropy) 上的示例项目 Localization.StarterWeb 包含设置 `Culture` 的 UI。 Views/Shared/_SelectLanguagePartial.cshtml 文件允许你从支持的区域性列表中选择区域性：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

Views/Shared/_SelectLanguagePartial.cshtml 文件添加到了布局文件的 `footer` 部分，使它将可供所有视图使用：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

`SetLanguage` 方法可设置区域性 cookie。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

不能将 _SelectLanguagePartial.cshtml 插入此项目的示例代码。 [GitHub](https://github.com/aspnet/entropy) 上的 Localization.StarterWeb 项目包含的代码可通过[依赖项注入](dependency-injection.md)容器将 `RequestLocalizationOptions` 流到 Razor 部分。

## <a name="model-binding-route-data-and-query-strings"></a>模型绑定路由数据和查询字符串

请参阅[模型绑定路由数据和查询字符串的全球化行为](xref:mvc/models/model-binding#glob)。

## <a name="globalization-and-localization-terms"></a>全球化和本地化术语

本地化应用的过程还要求基本了解现代软件开发中常用的相关字符集，以及与之相关的问题。 尽管所有计算机将文本都存储为数字（代码），但不同的系统使用不同的数字存储相同的文本。 本地化过程是指针对特定区域性/区域设置转换应用的用户界面 (UI)。

[本地化性](/dotnet/standard/globalization-localization/localizability-review)是一个中间过程，用于验证全球化应用是否准备好进行本地化。

区域性名称的 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 格式为 `<languagecode2>-<country/regioncode2>`，其中 `<languagecode2>` 是语言代码，`<country/regioncode2>` 是子区域性代码。 例如，`es-CL` 表示西班牙语（智利），`en-US` 表示英语（美国），而 `en-AU` 表示英语（澳大利亚）。 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 是一个与语言相关的 ISO 639 双小写字母的区域性代码和一个与国家/地区相关的 ISO 3166 双大写字母子区域性代码的组合。 请参阅 <https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>。

国际化常缩写为“I18N”。 缩写采用第一个和最后一个字母以及它们之间的字母数，因此 18 代表第一个字母“I”和最后一个“N”之间的字母数。 这同样适用于全球化 (G11N) 和本地化 (L10N)。

术语：

* 全球化 (G11N)：使应用支持不同语言和区域的过程。
* 本地化 (L10N)：针对给定语言和区域自定义应用的过程。
* 国际化 (I18N)：介绍了全球化和本地化。
* 区域性：它是一种语言和区域（可选）。
* 非特定区域性：具有指定语言但不具有区域的区域性。 （例如，“en”，“es”）
* 特定区域性：具有指定语言和区域的区域性。 （例如，“en-US”，“en-GB”，“es-CL”）
* 父区域性：包含特定区域性的非特定区域性。 （例如，“en”是“en-US”和“en-GB”的父区域性）
* 区域设置：区域设置与区域性相同。

[!INCLUDE[](~/includes/localization/currency.md)]

[!INCLUDE[](~/includes/localization/unsupported-culture-log-level.md)]

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* 本文所用的 [Localization.StarterWeb 项目](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb)。
* [对 .NET 应用程序进行全球化和本地化](/dotnet/standard/globalization-localization/index)
* [.resx 文件中的资源](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft 多语言应用工具包](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [本地化与泛型](http://hishambinateya.com/localization-and-generics)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)、[Damien Bowden](https://twitter.com/damien_bod)、[Bart Calixto](https://twitter.com/bartmax)、[Nadeem Afana](https://afana.me/) 和 [Hisham Bin Ateya](https://twitter.com/hishambinateya)

多语言网站使网站可以覆盖更广泛的受众。 ASP.NET Core 提供的服务和中间件可将网站本地化为不同的语言和文化。

国际化涉及[全球化](/dotnet/api/system.globalization)和[本地化](/dotnet/standard/globalization-localization/localization)。 全球化是设计支持不同区域性的应用程序的过程。 全球化添加了对一组有关特定地理区域的已定义语言脚本的输入、显示和输出支持。

本地化是将已经针对可本地化性进行处理的全球化应用调整为特定的区域性/区域设置的过程。 有关详细信息，请参阅本文档邻近末尾的全球化和本地化术语。

应用本地化涉及以下内容：

1. 使应用内容可本地化
1. 为支持的语言和区域性提供本地化资源
1. 实施策略，为每个请求选择语言/区域性

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/Localization)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="make-the-apps-content-localizable"></a>使应用内容可本地化

已为 <xref:Microsoft.Extensions.Localization.IStringLocalizer> 和 <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> 设置架构，可以为开发本地化应用提高工作效率。 `IStringLocalizer` 使用 <xref:System.Resources.ResourceManager> 和 <xref:System.Resources.ResourceReader> 在运行时提供特定于区域性的资源。 接口具有一个索引器和一个用于返回本地化字符串的 `IEnumerable`。 `IStringLocalizer` 不要求在资源文件中存储默认语言字符串。 你可以开发针对本地化的应用，且无需在开发初期创建资源资源文件。 下面的代码演示如何针对本地化包装字符串“About Title”。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

在前面的代码中，`IStringLocalizer<T>` 实现来源于[依赖关系注入](dependency-injection.md)。 如果找不到“About Title”的本地化值，则返回索引器键，即字符串“About Title”。 可将默认语言文本字符串保留在应用中并将它们包装在本地化工具中，以便你可集中精力开发应用。 你使用默认语言开发应用，并针对本地化步骤进行准备，而无需首先创建默认资源文件。 或者，你可以使用传统方法，并提供键以检索默认语言字符串。 对于许多开发者而言，不具有默认语言 .resx 文件且简单包装字符串文本的新工作流可以减少本地化应用的开销。 其他开发者将首选传统工作流，因为它可以更轻松地使用较长字符串文本，更轻松地更新本地化字符串。

对包含 HTML 的资源使用 `IHtmlLocalizer<T>` 实现。 `IHtmlLocalizer` 对资源字符串中格式化的参数进行 HTML 编码，但不对资源字符串本身进行 HTML 编码。 在下面突出显示的示例中，仅 `name` 参数的值被 HTML 编码。

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 通常，仅本地化文本，而不是 HTML。

最低程度，你可以从[依赖关系注入](dependency-injection.md)获取 `IStringLocalizerFactory`：

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

上面的代码演示了这两种工厂创建方法。

可以按控制器、区域对本地化字符串分区，或只有一个容器。 在示例应用中，名为 `SharedResource` 的虚拟类用于共享资源。

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

某些开发者使用 `Startup` 类，以包含全局或共享字符串。 在下面的示例中，使用 `InfoController` 和 `SharedResource` 本地化工具：

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>视图本地化

`IViewLocalizer` 服务可为[视图](xref:mvc/views/overview)提供本地化字符串。 `ViewLocalizer` 类可实现此接口，并从视图文件路径找到资源位置。 下面的代码演示如何使用 `IViewLocalizer` 的默认实现：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

`IViewLocalizer` 的默认实现可根据视图的文件名查找资源文件。 没有使用全局共享资源文件的选项。 `ViewLocalizer` 使用 `IHtmlLocalizer` 实现本地化工具，因此 Razor 不会对本地化字符串进行 HTML 编码。 你可以参数化资源字符串，`IViewLocalizer` 将对参数进行 HTML 编码，但不会对资源字符串进行。 请考虑以下 Razor 标记：

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

法语资源文件可以包含以下信息：

| 键 | “值” |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

呈现的视图可能包含资源文件中的 HTML 标记。

> [!NOTE]
> 通常，仅本地化文本，而不是 HTML。

若要在视图中使用共享资源文件，请注入 `IHtmlLocalizer<T>`：

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations 本地化

DataAnnotations 错误消息已使用 `IStringLocalizer<T>` 本地化。 使用选项 `ResourcesPath = "Resources"`，`RegisterViewModel` 中的错误消息可以存储在以下路径之一：

* *Resources/ViewModels.Account.RegisterViewModel.fr.resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

在 ASP.NET Core MVC 1.1.0 和更高版本中，非验证属性已经进行了本地化。 ASP.NET Core MVC 1.0 不会为非验证属性查找本地化字符串。

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>对多个类使用一个资源字符串

下面的代码演示如何针对具有多个类的验证属性使用一个资源字符串：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

在上面的代码中，`SharedResource` 是对应于存储验证消息的 resx 的类。 使用此方法，DataAnnotations 将仅使用 `SharedResource`，而不是每个类的资源。

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>为支持的语言和区域性提供本地化资源

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures 和 SupportedUICultures

ASP.NET Core 允许指定两个区域性值，`SupportedCultures` 和 `SupportedUICultures`。 `SupportedCultures` 的 [CultureInfo](/dotnet/api/system.globalization.cultureinfo) 对象可决定区域性相关函数的结果，如日期、时间、数字和货币格式等。 `SupportedCultures` 确定文本的排序顺序、大小写约定和字符串比较。 请参阅 [CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture) 详细了解服务器如何获取区域性。 `SupportedUICultures` 可确定按 [ResourceManager](/dotnet/api/system.resources.resourcemanager) 来查找哪些转换字符串（位于 .resx 文件）。 `ResourceManager` 只需查找 `CurrentUICulture` 决定的区域性特定字符串。 .NET 中的每个线程都具有 `CurrentCulture` 和 `CurrentUICulture` 对象。 呈现区域性相关函数时，ASP.NET Core 可检查这些值。 例如，如果当前线程的区域性设置为“en-US”（英语，美国），`DateTime.Now.ToLongDateString()` 将显示“Thursday, February 18, 2016”，但如果 `CurrentCulture` 设置为“es-ES”（西班牙语，西班牙），则输出将为“jueves，18 de febrero de 2016”。

## <a name="resource-files"></a>资源文件

资源文件是将可本地化的字符串与代码分离的有用机制。 非默认语言的转换字符串在 .resx 资源文件中单独显示。 例如，你可能想要创建包含转换字符串、名为 Welcome.es.resx 的西班牙语资源文件。 “es”是西班牙语的语言代码。 要在 Visual Studio 中创建此资源文件，请支持以下操作：

1. 在“解决方案资源管理器”中，右键单击将包含资源文件的文件夹 >“添加”>“新项”  。

   ![嵌套的上下文菜单：在“解决方案资源管理器”中，“资源”可打开上下文菜单。 “添加”可打开第二个上下文菜单，突出显示“新项”命令。](localization/_static/newi.png)

1. 在“搜索已安装的模板”框中，输入“资源”并命名该文件。

   ![“添加新项”对话框](localization/_static/res.png)

1. 在“名称”列中输入键值（本机字符串），在“值”列中输入转换字符串 。

   ![Welcome.es.resx 文件（西班牙语版 Welcome 资源文件）的单词 Hello 位于“名称”列，Hola（西班牙语版 Hello）位于“值”列](localization/_static/hola.png)

   Visual Studio 将显示 Welcome.es.resx 文件。

   ![显示 Welcome Spanish (es) 资源文件的解决方案资源管理器](localization/_static/se.png)

## <a name="resource-file-naming"></a>资源文件命名

资源名称是类的完整类型名称减去程序集名称。 例如，类 `LocalizationWebsite.Web.Startup` 的主要程序集为 `LocalizationWebsite.Web.dll` 的项目中的法语资源将命名为 Startup.fr.resx。 类 `LocalizationWebsite.Web.Controllers.HomeController` 的资源将命名为 Controllers.HomeController.fr.resx。 如果目标类的命名空间与将需要完整类型名称的程序集名称不同。 例如，在示例项目中，类型 `ExtraNamespace.Tools` 的资源将命名为 ExtraNamespace.Tools.fr.resx。

在示例项目中，`ConfigureServices` 方法将 `ResourcesPath` 设置为“资源”，因此主控制器的法语资源文件的项目相对路径是 Resources/Controllers.HomeController.fr.resx。 或者，你可以使用文件夹组织资源文件。 对于主控制器，该路径将为 Resources/Controllers/HomeController.fr.resx。 如果不使用 `ResourcesPath` 选项，.resx 文件将转到项目的基目录中。 `HomeController` 的资源文件将命名为 Controllers.HomeController.fr.resx。 是选择使用圆点还是路径命名约定，具体取决于你想如何组织资源文件。

| 资源名称 | 圆点或路径命名 |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | 圆点  |
| Resources/Controllers/HomeController.fr.resx  | 路径 |

Razor 视图中使用 `@inject IViewLocalizer` 的资源文件遵循类似的模式。 可以使用圆点命名或路径命名约定对视图的资源文件进行命名。 Razor 视图资源文件可模拟其关联视图文件的路径。 假设我们将 `ResourcesPath` 设置为“Resources”，与 *Views/Home/About.cshtml* 视图关联的法语资源文件可能是下面其中之一 ：

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

如果不使用 `ResourcesPath` 选项，视图的 .resx 文件将位于视图所在的文件夹。

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

[RootNamespace](/dotnet/api/microsoft.extensions.localization.rootnamespaceattribute?view=aspnetcore-2.1) 属性在程序集的根命名空间不同于程序集名称时，提供程序集的根命名空间。 

> [!WARNING]
> 当项目名称不是有效的 .NET 标识符时，可能会发生这种情况。 例如，`my-project-name.csproj` 将使用根命名空间 `my_project_name` 和导致此错误的程序集名称 `my-project-name`。 

如果程序集的根命名空间不同于程序集名称：

* 默认情况下无法进行本地化。
* 因程序集内搜索资源的方式导致本地化失败。 `RootNamespace` 是生成时间值，不可用于正在执行的进程。 

如果 `RootNamespace` 不同于 `AssemblyName`，请在 AssemblyInfo.cs 中包括以下内容（参数值替换为实际值）：

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

上述代码可成功解析 resx 文件。

## <a name="culture-fallback-behavior"></a>区域性回退行为

在搜索资源时，本地化会进行“区域性回退”。 从所请求的区域性开始，如果未能找到，则还原至该区域性的父区域性。 另外，[CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) 属性代表父区域性。 这通常（但并不是总是）意味着从 ISO 中移除区域签名。 例如，墨西哥的西班牙语方言为“es-MX”。 它具备一个父级“es”西班牙，没有特别指定国家/地区。

假设你的站点接收到了一个区域性为“fr-CA”的“Welcome”资源的请求。 本地化系统按顺序查找以下资源，并选择第一个匹配项：

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx*（如果 `NeutralResourcesLanguage` 为“fr-CA”）

例如，如果删除了“.fr”区域性指示符，而且已将区域性设置为“法语”，将读取默认资源文件，并本地化字符串。 对于不满足所请求区域性的情况，资源管理器可指定默认资源或回退资源。 缺少适用于请求区域性的资源时，如果只想返回键，不得具有默认资源文件。

### <a name="generate-resource-files-with-visual-studio"></a>使用 Visual Studio 生成资源文件

如果在 Visual Studio 中创建文件名没有区域性的资源文件（例如 Welcome.resx），Visual Studio 将创建一个 C# 类，并且具有每个字符串的属性。 这通常不是你想在 ASP.NET Core 中使用的。 你通常没有默认的 .resx 资源文件（没有区域性名称的 .resx 文件） 。 建议创建具有区域性名称（例如 Welcome.fr.resx）的 .resx 文件 。 创建具有区域性名称的 .resx 文件时，Visual Studio 不会生成类文件。

### <a name="add-other-cultures"></a>添加其他区域性

每个语言和区域性组合（除默认语言外）都需要唯一资源文件。 通过新建 ISO 语言代码属于名称一部分的资源文件，为不同的区域性和区域设置创建资源文件（例如，en-us、fr-ca 和 en-gb）  。 这些 ISO 编码位于文件名和 .resx 文件扩展之间，如 Welcome.es-MX.resx（西班牙语/墨西哥） 。

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>实施策略，为每个请求选择语言/区域性

### <a name="configure-localization"></a>配置本地化

通过 `Startup.ConfigureServices` 方法配置本地化：

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` 将本地化服务添加到服务容器。 上面的代码还将资源路径设置为“Resources”。

* `AddViewLocalization` 添加对本地化视图文件的支持。 在此示例视图中，本地化基于视图文件后缀。 例如，Index.fr.cshtml 文件中的“fr”。

* `AddDataAnnotationsLocalization` 添加通过 `IStringLocalizer` 抽象对本地化 `DataAnnotations` 验证消息的支持。

### <a name="localization-middleware"></a>本地化中间件

在本地化[中间件](xref:fundamentals/middleware/index)中设置有关请求的当前区域性。 在 `Startup.Configure` 方法中启用本地化中间件。 必须在中间件前面配置本地化中间件，它可能检查请求区域性（例如，`app.UseMvcWithDefaultRoute()`）。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` 初始化 `RequestLocalizationOptions` 对象。 在每个请求上，枚举了 `RequestLocalizationOptions` 的 `RequestCultureProvider` 列表，使用了可成功决定请求区域性的第一个提供程序。 默认提供程序来自 `RequestLocalizationOptions` 类：

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

默认列表从最具体到最不具体排序。 在本文的后面部分，我们将了解如何更改顺序，甚至添加一个自定义区域性提供程序。 如果没有一个提供程序可以确定请求区域性，则使用 `DefaultRequestCulture`。

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

某些应用将使用查询字符串来设置<https://docs.microsoft.com/dotnet/api/system.globalization.cultureinfo?view=netcore-3.1>。 对于使用 cookie 或接受语言标题方法的应用，向 URL 添加查询字符串有助于调试和测试代码。 默认情况下，`QueryStringRequestCultureProvider` 注册为 `RequestCultureProvider` 列表中的第一个本地化提供程序。 传递查询字符串参数 `culture` 和 `ui-culture`。 下面的示例将特定区域性（语言和区域）设置为“西班牙语/墨西哥”：

```
http://localhost:5000/?culture=es-MX&ui-culture=es-MX
```

如果仅传入两种区域性之一（`culture` 或 `ui-culture`，查询字符串提供程序将使用你传入的区域性设置这两个值。 例如，仅设置区域性将同时设置 `Culture` 和 `UICulture`：

```
http://localhost:5000/?culture=es-MX
```

### <a name="no-loccookierequestcultureprovider"></a>CookieRequestCultureProvider

通常，生产应用将提供一种机制来使用 ASP.NET Core 区域性 cookie 设置区域性。 若要创建 cookie，请使用 `MakeCookieValue` 方法。

`CookieRequestCultureProvider` `DefaultCookieName` 将返回用来跟踪用户首选区域性信息的默认 cookie 名称。 默认的 cookie 名称为 `.AspNetCore.Culture`。

cookie 格式为 `c=%LANGCODE%|uic=%LANGCODE%`，其中 `c` 是 `Culture`，`uic` 是 `UICulture`，例如：

```
c=en-UK|uic=en-US
```

如果仅指定其中一个区域性信息和 UI 区域性，则指定的区域性将同时用于区域性信息和 UI 区域性。

### <a name="the-accept-language-http-header"></a>接受语言 HTTP 标题

[接受语言标题](https://www.w3.org/International/questions/qa-accept-lang-locales)在大多数浏览器中可设置，最初用于指定用户的语言。 此设置指示浏览器已设置为发送或已从基础操作系统继承的内容。 浏览器请求的接受语言 HTTP 标题不是检测用户首选语言的可靠方法（请参阅 [Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php)（在浏览器中设置首选项）。 生产应用应包括一种用户可以自定义区域性选择的方法。

### <a name="set-the-accept-language-http-header-in-ie"></a>在 IE 中设置接受语言 HTTP 标题

1. 在齿轮图标中，点击“Internet 选项”。

1. 点击“语言”。

   ![Internet 选项](localization/_static/lang.png)

1. 点击“设置语言首选项”。

1. 点击“添加语言”。

1. 添加语言。

1. 点击语言，然后点击“向上移动”。

### <a name="use-a-custom-provider"></a>使用自定义提供程序

假设你想要让客户在数据库中存储其语言和区域性。 你可以编写一个提供程序来查找用户的这些值。 下面的代码演示如何添加自定义提供程序：

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.RequestCultureProviders.Insert(0, new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

使用 `RequestLocalizationOptions` 添加或删除本地化提供程序。

### <a name="set-the-culture-programmatically"></a>以编程方式设置区域性

[GitHub](https://github.com/aspnet/entropy) 上的示例项目 Localization.StarterWeb 包含设置 `Culture` 的 UI。 Views/Shared/_SelectLanguagePartial.cshtml 文件允许你从支持的区域性列表中选择区域性：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

Views/Shared/_SelectLanguagePartial.cshtml 文件添加到了布局文件的 `footer` 部分，使它将可供所有视图使用：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

`SetLanguage` 方法可设置区域性 cookie。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

不能将 _SelectLanguagePartial.cshtml 插入此项目的示例代码。 [GitHub](https://github.com/aspnet/entropy) 上的 Localization.StarterWeb 项目包含的代码可通过[依赖项注入](dependency-injection.md)容器将 `RequestLocalizationOptions` 流到 Razor 部分。

## <a name="model-binding-route-data-and-query-strings"></a>模型绑定路由数据和查询字符串

请参阅[模型绑定路由数据和查询字符串的全球化行为](xref:mvc/models/model-binding#glob)。

## <a name="globalization-and-localization-terms"></a>全球化和本地化术语

本地化应用的过程还要求基本了解现代软件开发中常用的相关字符集，以及与之相关的问题。 尽管所有计算机将文本都存储为数字（代码），但不同的系统使用不同的数字存储相同的文本。 本地化过程是指针对特定区域性/区域设置转换应用的用户界面 (UI)。

[本地化性](/dotnet/standard/globalization-localization/localizability-review)是一个中间过程，用于验证全球化应用是否准备好进行本地化。

区域性名称的 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 格式为 `<languagecode2>-<country/regioncode2>`，其中 `<languagecode2>` 是语言代码，`<country/regioncode2>` 是子区域性代码。 例如，`es-CL` 表示西班牙语（智利），`en-US` 表示英语（美国），而 `en-AU` 表示英语（澳大利亚）。 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 是一个与语言相关的 ISO 639 双小写字母的区域性代码和一个与国家/地区相关的 ISO 3166 双大写字母子区域性代码的组合。 请参阅 <https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>。

国际化常缩写为“I18N”。 缩写采用第一个和最后一个字母以及它们之间的字母数，因此 18 代表第一个字母“I”和最后一个“N”之间的字母数。 这同样适用于全球化 (G11N) 和本地化 (L10N)。

术语：

* 全球化 (G11N)：使应用支持不同语言和区域的过程。
* 本地化 (L10N)：针对给定语言和区域自定义应用的过程。
* 国际化 (I18N)：介绍了全球化和本地化。
* 区域性：它是一种语言和区域（可选）。
* 非特定区域性：具有指定语言但不具有区域的区域性。 （例如，“en”，“es”）
* 特定区域性：具有指定语言和区域的区域性。 （例如，“en-US”，“en-GB”，“es-CL”）
* 父区域性：包含特定区域性的非特定区域性。 （例如，“en”是“en-US”和“en-GB”的父区域性）
* 区域设置：区域设置与区域性相同。

[!INCLUDE[](~/includes/localization/currency.md)]

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* 本文所用的 [Localization.StarterWeb 项目](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb)。
* [对 .NET 应用程序进行全球化和本地化](/dotnet/standard/globalization-localization/index)
* [.resx 文件中的资源](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft 多语言应用工具包](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [本地化与泛型](http://hishambinateya.com/localization-and-generics)

::: moniker-end

<!-- ASP.NET Core 5.x starts here -->
::: moniker range="> aspnetcore-3.1"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)、[Damien Bowden](https://twitter.com/damien_bod)、[Bart Calixto](https://twitter.com/bartmax)、[Nadeem Afana](https://afana.me/) 和 [Hisham Bin Ateya](https://twitter.com/hishambinateya)

多语言网站使网站可以覆盖更广泛的受众。 ASP.NET Core 提供的服务和中间件可将网站本地化为不同的语言和文化。

国际化涉及[全球化](/dotnet/api/system.globalization)和[本地化](/dotnet/standard/globalization-localization/localization)。 全球化是设计支持不同区域性的应用程序的过程。 全球化添加了对一组有关特定地理区域的已定义语言脚本的输入、显示和输出支持。

本地化是将已经针对可本地化性进行处理的全球化应用调整为特定的区域性/区域设置的过程。 有关详细信息，请参阅本文档邻近末尾的全球化和本地化术语。

应用本地化涉及以下内容：

1. 使应用内容可本地化
1. 为支持的语言和区域性提供本地化资源
1. 实施策略，为每个请求选择语言/区域性

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/localization/sample/2.x/)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="make-the-apps-content-localizable"></a>使应用内容可本地化

已为 <xref:Microsoft.Extensions.Localization.IStringLocalizer> 和 <xref:Microsoft.Extensions.Localization.IStringLocalizer%601> 设置架构，可以为开发本地化应用提高工作效率。 `IStringLocalizer` 使用 <xref:System.Resources.ResourceManager> 和 <xref:System.Resources.ResourceReader> 在运行时提供特定于区域性的资源。 接口具有一个索引器和一个用于返回本地化字符串的 `IEnumerable`。 `IStringLocalizer` 不要求在资源文件中存储默认语言字符串。 你可以开发针对本地化的应用，且无需在开发初期创建资源资源文件。 下面的代码演示如何针对本地化包装字符串“About Title”。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/AboutController.cs)]

在前面的代码中，`IStringLocalizer<T>` 实现来源于[依赖关系注入](dependency-injection.md)。 如果找不到“About Title”的本地化值，则返回索引器键，即字符串“About Title”。 可将默认语言文本字符串保留在应用中并将它们包装在本地化工具中，以便你可集中精力开发应用。 你使用默认语言开发应用，并针对本地化步骤进行准备，而无需首先创建默认资源文件。 或者，你可以使用传统方法，并提供键以检索默认语言字符串。 对于许多开发者而言，不具有默认语言 .resx 文件且简单包装字符串文本的新工作流可以减少本地化应用的开销。 其他开发者将首选传统工作流，因为它可以更轻松地使用较长字符串文本，更轻松地更新本地化字符串。

对包含 HTML 的资源使用 `IHtmlLocalizer<T>` 实现。 `IHtmlLocalizer` 对资源字符串中格式化的参数进行 HTML 编码，但不对资源字符串本身进行 HTML 编码。 在下面突出显示的示例中，仅 `name` 参数的值被 HTML 编码。

[!code-csharp[](~/fundamentals/localization/sample/3.x/Localization/Controllers/BookController.cs?highlight=3,5,20&start=1&end=24)]

> [!NOTE]
> 通常，仅本地化文本，而不是 HTML。

最低程度，你可以从[依赖关系注入](dependency-injection.md)获取 `IStringLocalizerFactory`：

[!code-csharp[](localization/sample/3.x/Localization/Controllers/TestController.cs?start=9&end=26&highlight=7-13)]

上面的代码演示了这两种工厂创建方法。

可以按控制器、区域对本地化字符串分区，或只有一个容器。 在示例应用中，名为 `SharedResource` 的虚拟类用于共享资源。

[!code-csharp[](localization/sample/3.x/Localization/SharedResource.cs)]

某些开发者使用 `Startup` 类，以包含全局或共享字符串。 在下面的示例中，使用 `InfoController` 和 `SharedResource` 本地化工具：

[!code-csharp[](localization/sample/3.x/Localization/Controllers/InfoController.cs?range=9-26)]

## <a name="view-localization"></a>视图本地化

`IViewLocalizer` 服务可为[视图](xref:mvc/views/overview)提供本地化字符串。 `ViewLocalizer` 类可实现此接口，并从视图文件路径找到资源位置。 下面的代码演示如何使用 `IViewLocalizer` 的默认实现：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Home/About.cshtml)]

`IViewLocalizer` 的默认实现可根据视图的文件名查找资源文件。 没有使用全局共享资源文件的选项。 `ViewLocalizer` 使用 `IHtmlLocalizer` 实现本地化工具，因此 Razor 不会对本地化字符串进行 HTML 编码。 你可以参数化资源字符串，`IViewLocalizer` 将对参数进行 HTML 编码，但不会对资源字符串进行。 请考虑以下 Razor 标记：

```cshtml
@Localizer["<i>Hello</i> <b>{0}!</b>", UserManager.GetUserName(User)]
```

法语资源文件可以包含以下信息：

| 键 | “值” |
| --- | ----- |
| `<i>Hello</i> <b>{0}!</b>` | `<i>Bonjour</i> <b>{0} !</b>` |

呈现的视图可能包含资源文件中的 HTML 标记。

> [!NOTE]
> 通常，仅本地化文本，而不是 HTML。

若要在视图中使用共享资源文件，请注入 `IHtmlLocalizer<T>`：

[!code-cshtml[](~/fundamentals/localization/sample/3.x/Localization/Views/Test/About.cshtml?highlight=5,12)]

## <a name="dataannotations-localization"></a>DataAnnotations 本地化

DataAnnotations 错误消息已使用 `IStringLocalizer<T>` 本地化。 使用选项 `ResourcesPath = "Resources"`，`RegisterViewModel` 中的错误消息可以存储在以下路径之一：

* *Resources/ViewModels.Account.RegisterViewModel.fr.resx*
* *Resources/ViewModels/Account/RegisterViewModel.fr.resx*

[!code-csharp[](localization/sample/3.x/Localization/ViewModels/Account/RegisterViewModel.cs?start=9&end=26)]

在 ASP.NET Core MVC 1.1.0 和更高版本中，非验证属性已经进行了本地化。 ASP.NET Core MVC 1.0 不会为非验证属性查找本地化字符串。

<a name="one-resource-string-multiple-classes"></a>

### <a name="using-one-resource-string-for-multiple-classes"></a>对多个类使用一个资源字符串

下面的代码演示如何针对具有多个类的验证属性使用一个资源字符串：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddDataAnnotationsLocalization(options => {
            options.DataAnnotationLocalizerProvider = (type, factory) =>
                factory.Create(typeof(SharedResource));
        });
}
```

在上面的代码中，`SharedResource` 是对应于存储验证消息的 resx 的类。 使用此方法，DataAnnotations 将仅使用 `SharedResource`，而不是每个类的资源。

## <a name="provide-localized-resources-for-the-languages-and-cultures-you-support"></a>为支持的语言和区域性提供本地化资源

### <a name="supportedcultures-and-supporteduicultures"></a>SupportedCultures 和 SupportedUICultures

ASP.NET Core 允许指定两个区域性值，`SupportedCultures` 和 `SupportedUICultures`。 `SupportedCultures` 的 [CultureInfo](/dotnet/api/system.globalization.cultureinfo) 对象可决定区域性相关函数的结果，如日期、时间、数字和货币格式等。 `SupportedCultures` 确定文本的排序顺序、大小写约定和字符串比较。 请参阅 [CultureInfo.CurrentCulture](/dotnet/api/system.stringcomparer.currentculture#System_StringComparer_CurrentCulture) 详细了解服务器如何获取区域性。 `SupportedUICultures` 可确定按 [ResourceManager](/dotnet/api/system.resources.resourcemanager) 来查找哪些转换字符串（位于 .resx 文件）。 `ResourceManager` 只需查找 `CurrentUICulture` 决定的区域性特定字符串。 .NET 中的每个线程都具有 `CurrentCulture` 和 `CurrentUICulture` 对象。 呈现区域性相关函数时，ASP.NET Core 可检查这些值。 例如，如果当前线程的区域性设置为“en-US”（英语，美国），`DateTime.Now.ToLongDateString()` 将显示“Thursday, February 18, 2016”，但如果 `CurrentCulture` 设置为“es-ES”（西班牙语，西班牙），则输出将为“jueves，18 de febrero de 2016”。

## <a name="resource-files"></a>资源文件

资源文件是将可本地化的字符串与代码分离的有用机制。 非默认语言的转换字符串在 .resx 资源文件中单独显示。 例如，你可能想要创建包含转换字符串、名为 Welcome.es.resx 的西班牙语资源文件。 “es”是西班牙语的语言代码。 要在 Visual Studio 中创建此资源文件，请支持以下操作：

1. 在“解决方案资源管理器”中，右键单击将包含资源文件的文件夹 >“添加”>“新项”  。

   ![嵌套的上下文菜单：在“解决方案资源管理器”中，“资源”可打开上下文菜单。 “添加”可打开第二个上下文菜单，突出显示“新项”命令。](localization/_static/newi.png)

1. 在“搜索已安装的模板”框中，输入“资源”并命名该文件。

   ![“添加新项”对话框](localization/_static/res.png)

1. 在“名称”列中输入键值（本机字符串），在“值”列中输入转换字符串 。

   ![Welcome.es.resx 文件（西班牙语版 Welcome 资源文件）的单词 Hello 位于“名称”列，Hola（西班牙语版 Hello）位于“值”列](localization/_static/hola.png)

   Visual Studio 将显示 Welcome.es.resx 文件。

   ![显示 Welcome Spanish (es) 资源文件的解决方案资源管理器](localization/_static/se.png)

## <a name="resource-file-naming"></a>资源文件命名

资源名称是类的完整类型名称减去程序集名称。 例如，类 `LocalizationWebsite.Web.Startup` 的主要程序集为 `LocalizationWebsite.Web.dll` 的项目中的法语资源将命名为 Startup.fr.resx。 类 `LocalizationWebsite.Web.Controllers.HomeController` 的资源将命名为 Controllers.HomeController.fr.resx。 如果目标类的命名空间与将需要完整类型名称的程序集名称不同。 例如，在示例项目中，类型 `ExtraNamespace.Tools` 的资源将命名为 ExtraNamespace.Tools.fr.resx。

在示例项目中，`ConfigureServices` 方法将 `ResourcesPath` 设置为“资源”，因此主控制器的法语资源文件的项目相对路径是 Resources/Controllers.HomeController.fr.resx。 或者，你可以使用文件夹组织资源文件。 对于主控制器，该路径将为 Resources/Controllers/HomeController.fr.resx。 如果不使用 `ResourcesPath` 选项，.resx 文件将转到项目的基目录中。 `HomeController` 的资源文件将命名为 Controllers.HomeController.fr.resx。 是选择使用圆点还是路径命名约定，具体取决于你想如何组织资源文件。

| 资源名称 | 圆点或路径命名 |
| ------------   | ------------- |
| Resources/Controllers.HomeController.fr.resx | 圆点  |
| Resources/Controllers/HomeController.fr.resx  | 路径 |

Razor 视图中使用 `@inject IViewLocalizer` 的资源文件遵循类似的模式。 可以使用圆点命名或路径命名约定对视图的资源文件进行命名。 Razor 视图资源文件可模拟其关联视图文件的路径。 假设我们将 `ResourcesPath` 设置为“Resources”，与 *Views/Home/About.cshtml* 视图关联的法语资源文件可能是下面其中之一 ：

* Resources/Views/Home/About.fr.resx

* Resources/Views.Home.About.fr.resx

如果不使用 `ResourcesPath` 选项，视图的 .resx 文件将位于视图所在的文件夹。

### <a name="rootnamespaceattribute"></a>RootNamespaceAttribute 

[RootNamespace](/dotnet/api/microsoft.extensions.localization.rootnamespaceattribute?view=aspnetcore-2.1) 属性在程序集的根命名空间不同于程序集名称时，提供程序集的根命名空间。 

> [!WARNING]
> 当项目名称不是有效的 .NET 标识符时，可能会发生这种情况。 例如，`my-project-name.csproj` 将使用根命名空间 `my_project_name` 和导致此错误的程序集名称 `my-project-name`。 

如果程序集的根命名空间不同于程序集名称：

* 默认情况下无法进行本地化。
* 因程序集内搜索资源的方式导致本地化失败。 `RootNamespace` 是生成时间值，不可用于正在执行的进程。 

如果 `RootNamespace` 不同于 `AssemblyName`，请在 AssemblyInfo.cs 中包括以下内容（参数值替换为实际值）：

```csharp
using System.Reflection;
using Microsoft.Extensions.Localization;

[assembly: ResourceLocation("Resource Folder Name")]
[assembly: RootNamespace("App Root Namespace")]
```

上述代码可成功解析 resx 文件。

## <a name="culture-fallback-behavior"></a>区域性回退行为

在搜索资源时，本地化会进行“区域性回退”。 从所请求的区域性开始，如果未能找到，则还原至该区域性的父区域性。 另外，[CultureInfo.Parent](/dotnet/api/system.globalization.cultureinfo.parent) 属性代表父区域性。 这通常（但并不是总是）意味着从 ISO 中移除区域签名。 例如，墨西哥的西班牙语方言为“es-MX”。 它具备一个父级“es”西班牙，没有特别指定国家/地区。

假设你的站点接收到了一个区域性为“fr-CA”的“Welcome”资源的请求。 本地化系统按顺序查找以下资源，并选择第一个匹配项：

* *Welcome.fr-CA.resx*
* *Welcome.fr.resx*
* *Welcome.resx*（如果 `NeutralResourcesLanguage` 为“fr-CA”）

例如，如果删除了“.fr”区域性指示符，而且已将区域性设置为“法语”，将读取默认资源文件，并本地化字符串。 对于不满足所请求区域性的情况，资源管理器可指定默认资源或回退资源。 缺少适用于请求区域性的资源时，如果只想返回键，不得具有默认资源文件。

### <a name="generate-resource-files-with-visual-studio"></a>使用 Visual Studio 生成资源文件

如果在 Visual Studio 中创建文件名没有区域性的资源文件（例如 Welcome.resx），Visual Studio 将创建一个 C# 类，并且具有每个字符串的属性。 这通常不是你想在 ASP.NET Core 中使用的。 你通常没有默认的 .resx 资源文件（没有区域性名称的 .resx 文件） 。 建议创建具有区域性名称（例如 Welcome.fr.resx）的 .resx 文件 。 创建具有区域性名称的 .resx 文件时，Visual Studio 不会生成类文件。

### <a name="add-other-cultures"></a>添加其他区域性

每个语言和区域性组合（除默认语言外）都需要唯一资源文件。 通过新建 ISO 语言代码属于名称一部分的资源文件，为不同的区域性和区域设置创建资源文件（例如，en-us、fr-ca 和 en-gb）  。 这些 ISO 编码位于文件名和 .resx 文件扩展之间，如 Welcome.es-MX.resx（西班牙语/墨西哥） 。

## <a name="implement-a-strategy-to-select-the-languageculture-for-each-request"></a>实施策略，为每个请求选择语言/区域性

### <a name="configure-localization"></a>配置本地化

通过 `Startup.ConfigureServices` 方法配置本地化：

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet1)]

* `AddLocalization` 将本地化服务添加到服务容器。 上面的代码还将资源路径设置为“Resources”。

* `AddViewLocalization` 添加对本地化视图文件的支持。 在此示例视图中，本地化基于视图文件后缀。 例如，Index.fr.cshtml 文件中的“fr”。

* `AddDataAnnotationsLocalization` 添加通过 `IStringLocalizer` 抽象对本地化 `DataAnnotations` 验证消息的支持。

### <a name="localization-middleware"></a>本地化中间件

在本地化[中间件](xref:fundamentals/middleware/index)中设置有关请求的当前区域性。 在 `Startup.Configure` 方法中启用本地化中间件。 必须在中间件前面配置本地化中间件，它可能检查请求区域性（例如，`app.UseMvcWithDefaultRoute()`）。

[!code-csharp[](localization/sample/3.x/Localization/Startup.cs?name=snippet2)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

`UseRequestLocalization` 初始化 `RequestLocalizationOptions` 对象。 在每个请求上，枚举了 `RequestLocalizationOptions` 的 `RequestCultureProvider` 列表，使用了可成功决定请求区域性的第一个提供程序。 默认提供程序来自 `RequestLocalizationOptions` 类：

1. `QueryStringRequestCultureProvider`
1. `CookieRequestCultureProvider`
1. `AcceptLanguageHeaderRequestCultureProvider`

默认列表从最具体到最不具体排序。 在本文的后面部分，我们将了解如何更改顺序，甚至添加一个自定义区域性提供程序。 如果没有一个提供程序可以确定请求区域性，则使用 `DefaultRequestCulture`。

### <a name="querystringrequestcultureprovider"></a>QueryStringRequestCultureProvider

某些应用将使用查询字符串来设置<xref:System.Globalization.CultureInfo>。 对于使用 cookie 或接受语言标题方法的应用，向 URL 添加查询字符串有助于调试和测试代码。 默认情况下，`QueryStringRequestCultureProvider` 注册为 `RequestCultureProvider` 列表中的第一个本地化提供程序。 传递查询字符串参数 `culture` 和 `ui-culture`。 下面的示例将特定区域性（语言和区域）设置为“西班牙语/墨西哥”：

```
http://localhost:5000/?culture=es-MX&ui-culture=es-MX
```

如果仅传入两种区域性之一（`culture` 或 `ui-culture`，查询字符串提供程序将使用你传入的区域性设置这两个值。 例如，仅设置区域性将同时设置 `Culture` 和 `UICulture`：

```
http://localhost:5000/?culture=es-MX
```

### <a name="no-loccookierequestcultureprovider"></a>CookieRequestCultureProvider

通常，生产应用将提供一种机制来使用 ASP.NET Core 区域性 cookie 设置区域性。 若要创建 cookie，请使用 `MakeCookieValue` 方法。

`CookieRequestCultureProvider` `DefaultCookieName` 将返回用来跟踪用户首选区域性信息的默认 cookie 名称。 默认的 cookie 名称为 `.AspNetCore.Culture`。

cookie 格式为 `c=%LANGCODE%|uic=%LANGCODE%`，其中 `c` 是 `Culture`，`uic` 是 `UICulture`，例如：

```
c=en-UK|uic=en-US
```

如果仅指定其中一个区域性信息和 UI 区域性，则指定的区域性将同时用于区域性信息和 UI 区域性。

### <a name="the-accept-language-http-header"></a>接受语言 HTTP 标题

[接受语言标题](https://www.w3.org/International/questions/qa-accept-lang-locales)在大多数浏览器中可设置，最初用于指定用户的语言。 此设置指示浏览器已设置为发送或已从基础操作系统继承的内容。 浏览器请求的接受语言 HTTP 标题不是检测用户首选语言的可靠方法（请参阅 [Setting language preferences in a browser](https://www.w3.org/International/questions/qa-lang-priorities.en.php)（在浏览器中设置首选项）。 生产应用应包括一种用户可以自定义区域性选择的方法。

### <a name="set-the-accept-language-http-header-in-ie"></a>在 IE 中设置接受语言 HTTP 标题

1. 在齿轮图标中，点击“Internet 选项”。

1. 点击“语言”。

   ![Internet 选项](localization/_static/lang.png)

1. 点击“设置语言首选项”。

1. 点击“添加语言”。

1. 添加语言。

1. 点击语言，然后点击“向上移动”。

### <a name="the-content-language-http-header"></a>Content-Language HTTP 标头

[Content-Language](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Language) 实体标头：

* 用于描述面向受众的语言。
* 允许用户根据用户的首选语言来区分。

实体标头用于 HTTP 请求和响应。

可通过设置属性 `ApplyCurrentCultureToResponseHeaders` 来添加 `Content-Language` 标头。

添加 `Content-Language` 标头：

* 允许 RequestLocalizationMiddleware 使用 `CurrentUICulture` 设置 `Content-Language` 标头。
* 无需显式设置响应标头 `Content-Language`。

```csharp
app.UseRequestLocalization(new RequestLocalizationOptions
{
    ApplyCurrentCultureToResponseHeaders = true
});
```

### <a name="use-a-custom-provider"></a>使用自定义提供程序

假设你想要让客户在数据库中存储其语言和区域性。 你可以编写一个提供程序来查找用户的这些值。 下面的代码演示如何添加自定义提供程序：

```csharp
private const string enUSCulture = "en-US";

services.Configure<RequestLocalizationOptions>(options =>
{
    var supportedCultures = new[]
    {
        new CultureInfo(enUSCulture),
        new CultureInfo("fr")
    };

    options.DefaultRequestCulture = new RequestCulture(culture: enUSCulture, uiCulture: enUSCulture);
    options.SupportedCultures = supportedCultures;
    options.SupportedUICultures = supportedCultures;

    options.AddInitialRequestCultureProvider(new CustomRequestCultureProvider(async context =>
    {
        // My custom request culture logic
        return new ProviderCultureResult("en");
    }));
});
```

使用 `RequestLocalizationOptions` 添加或删除本地化提供程序。

### <a name="set-the-culture-programmatically"></a>以编程方式设置区域性

[GitHub](https://github.com/aspnet/entropy) 上的示例项目 Localization.StarterWeb 包含设置 `Culture` 的 UI。 Views/Shared/_SelectLanguagePartial.cshtml 文件允许你从支持的区域性列表中选择区域性：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_SelectLanguagePartial.cshtml)]

Views/Shared/_SelectLanguagePartial.cshtml 文件添加到了布局文件的 `footer` 部分，使它将可供所有视图使用：

[!code-cshtml[](localization/sample/3.x/Localization/Views/Shared/_Layout.cshtml?range=43-56&highlight=10)]

`SetLanguage` 方法可设置区域性 cookie。

[!code-csharp[](localization/sample/3.x/Localization/Controllers/HomeController.cs?range=57-67)]

不能将 _SelectLanguagePartial.cshtml 插入此项目的示例代码。 [GitHub](https://github.com/aspnet/entropy) 上的 Localization.StarterWeb 项目包含的代码可通过[依赖项注入](dependency-injection.md)容器将 `RequestLocalizationOptions` 流到 Razor 部分。

## <a name="model-binding-route-data-and-query-strings"></a>模型绑定路由数据和查询字符串

请参阅[模型绑定路由数据和查询字符串的全球化行为](xref:mvc/models/model-binding#glob)。

## <a name="globalization-and-localization-terms"></a>全球化和本地化术语

本地化应用的过程还要求基本了解现代软件开发中常用的相关字符集，以及与之相关的问题。 尽管所有计算机将文本都存储为数字（代码），但不同的系统使用不同的数字存储相同的文本。 本地化过程是指针对特定区域性/区域设置转换应用的用户界面 (UI)。

[本地化性](/dotnet/standard/globalization-localization/localizability-review)是一个中间过程，用于验证全球化应用是否准备好进行本地化。

区域性名称的 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 格式为 `<languagecode2>-<country/regioncode2>`，其中 `<languagecode2>` 是语言代码，`<country/regioncode2>` 是子区域性代码。 例如，`es-CL` 表示西班牙语（智利），`en-US` 表示英语（美国），而 `en-AU` 表示英语（澳大利亚）。 [RFC 4646](https://www.ietf.org/rfc/rfc4646.txt) 是一个与语言相关的 ISO 639 双小写字母的区域性代码和一个与国家/地区相关的 ISO 3166 双大写字母子区域性代码的组合。 请参阅 <https://docs.microsoft.com/previous-versions/commerce-server/ee825488(v=cs.20)>。

国际化常缩写为“I18N”。 缩写采用第一个和最后一个字母以及它们之间的字母数，因此 18 代表第一个字母“I”和最后一个“N”之间的字母数。 这同样适用于全球化 (G11N) 和本地化 (L10N)。

术语：

* 全球化 (G11N)：使应用支持不同语言和区域的过程。
* 本地化 (L10N)：针对给定语言和区域自定义应用的过程。
* 国际化 (I18N)：介绍了全球化和本地化。
* 区域性：它是一种语言和区域（可选）。
* 非特定区域性：具有指定语言但不具有区域的区域性。 （例如，“en”，“es”）
* 特定区域性：具有指定语言和区域的区域性。 （例如，“en-US”，“en-GB”，“es-CL”）
* 父区域性：包含特定区域性的非特定区域性。 （例如，“en”是“en-US”和“en-GB”的父区域性）
* 区域设置：区域设置与区域性相同。

[!INCLUDE[](~/includes/localization/currency.md)]

[!INCLUDE[](~/includes/localization/unsupported-culture-log-level.md)]

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/troubleshoot-aspnet-core-localization>
* 本文所用的 [Localization.StarterWeb 项目](https://github.com/aspnet/Entropy/tree/master/samples/Localization.StarterWeb)。
* [对 .NET 应用程序进行全球化和本地化](/dotnet/standard/globalization-localization/index)
* [.resx 文件中的资源](/dotnet/framework/resources/working-with-resx-files-programmatically)
* [Microsoft 多语言应用工具包](https://marketplace.visualstudio.com/items?itemName=MultilingualAppToolkit.MultilingualAppToolkit-18308)
* [本地化与泛型](http://hishambinateya.com/localization-and-generics)

::: moniker-end
