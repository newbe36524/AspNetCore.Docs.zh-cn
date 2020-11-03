---
title: 了解如何从 ASP.NET MVC 迁移到 ASP.NET Core MVC
author: wadepickett
description: 了解如何开始将 ASP.NET MVC 项目迁移到 ASP.NET Core MVC。
ms.author: wpickett
ms.date: 06/18/2020
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
uid: migration/mvc
ms.openlocfilehash: 226ac6da508378c7b3c81779d38dd2e0840f1fed
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051508"
---
# <a name="migrate-from-aspnet-mvc-to-aspnet-core-mvc"></a>从 ASP.NET MVC 迁移到 ASP.NET Core MVC

::: moniker range=">= aspnetcore-3.0"

本文介绍如何开始将 ASP.NET MVC 项目迁移到 [ASP.NET CORE mvc](xref:mvc/overview)。 在此过程中，它突出显示了 ASP.NET MVC 中的相关更改。

从 ASP.NET MVC 迁移是一个多步骤过程。 本文介绍：

* 初始设置。
* 基本控制器和视图。
* 静态内容。
* 客户端依赖关系。

若要迁移配置和 Identity 代码，请参阅将 [配置迁移到 ASP.NET Core](xref:migration/configuration) 并 [迁移身份验证和 Identity ASP.NET Core](xref:migration/identity)。

## <a name="prerequisites"></a>先决条件

[!INCLUDE [prerequisites](../includes/net-core-prereqs-vs-3.1.md)]

## <a name="create-the-starter-aspnet-mvc-project"></a>创建 starter ASP.NET MVC 项目

在 Visual Studio 中创建一个示例 ASP.NET MVC 项目以进行迁移：

1. 从“文件”菜单中选择“新建”>“项目”  。
1. 选择 " **ASP.NET Web 应用程序 ( .NET Framework")** ，然后选择 " **下一步** "。
1. 将项目命名为 *WebApp1* ，使命名空间与下一步中创建的 ASP.NET Core 项目相匹配。 选择“创建”。
1. 选择 " **MVC** "，然后选择 " **创建** "。

## <a name="create-the-aspnet-core-project"></a>创建 ASP.NET Core 项目

使用要迁移到的新 ASP.NET Core 项目创建新的解决方案：

1. 启动 Visual Studio 的第二个实例。
1. 从“文件”菜单中选择“新建”>“项目”  。
1. 选择“ASP.NET Core Web 应用程序”，然后选择“下一步” 。
1. 在 " **配置新项目** " 对话框中，将项目命名为 " *WebApp1* "。
1. 将位置设置为与上一个项目不同的目录，以使用相同的项目名称。 使用同一个命名空间可以更轻松地在两个项目之间复制代码。 选择“创建”。
1. 在“创建新的 ASP.NET Core Web 应用程序”对话框中，确认选择“.NET Core”和“ASP.NET Core 3.1”  。 选择 " **Web 应用程序 (模型-视图-控制器)** " 项目模板，然后选择 " **创建** "。

## <a name="configure-the-aspnet-core-site-to-use-mvc"></a>将 ASP.NET Core 网站配置为使用 MVC

在 ASP.NET Core 3.0 及更高版本的项目中，.NET Framework 不再是受支持的目标框架。 你的项目必须面向 .NET Core。 包含 MVC 的 ASP.NET Core 共享框架是 .NET Core 运行时安装的一部分。 使用项目文件中的 `Microsoft.NET.Sdk.Web` SDK 时，会自动引用共享框架：

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
```

有关详细信息，请参阅 [框架引用](xref:migration/22-to-30#framework-reference)。

在 ASP.NET Core 中， `Startup` 类：

* 替换 *global.asax* 。
* 处理所有应用启动任务。

有关详细信息，请参阅 <xref:fundamentals/startup>。

在 ASP.NET Core 项目中，打开 *Startup.cs* 文件：

[!code-csharp[](mvc/samples/3.x/Startup.cs?highlight=13,30,32&name=snippet)]

ASP.NET Core 应用必须选择包含中间件的框架功能。 上一个模板生成的代码添加以下服务和中间件：

* <xref:Microsoft.Extensions.DependencyInjection.MvcServiceCollectionExtensions.AddControllersWithViews%2A>扩展方法为控制器、API 相关的功能和视图注册 MVC 服务支持。 有关 MVC 服务注册选项的详细信息，请参阅 [mvc 服务注册](xref:migration/22-to-30#mvc-service-registration)
* <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A>扩展方法添加静态文件处理程序 `Microsoft.AspNetCore.StaticFiles` 。 `UseStaticFiles`必须先调用扩展方法 `UseRouting` 。 有关详细信息，请参阅 <xref:fundamentals/static-files>。
* <xref:Microsoft.AspNetCore.Builder.EndpointRoutingApplicationBuilderExtensions.UseRouting%2A>扩展方法将添加路由。 有关详细信息，请参阅 <xref:fundamentals/routing>。

此现有配置包括迁移示例 ASP.NET MVC 项目所需的内容。 有关 ASP.NET Core 中间件选项的详细信息，请参阅 <xref:fundamentals/startup> 。

## <a name="migrate-controllers-and-views"></a>迁移控制器和视图

在 ASP.NET Core 项目中，将添加新的空控制器类和视图类作为占位符，使用与要从中进行迁移的任何 ASP.NET MVC 项目中的控制器和视图类相同的名称。

ASP.NET Core *WebApp1* 项目已包含与 ASP.NET MVC 项目相同的名称的最小示例控制器和视图。 因此，这些占位符将用作 ASP.NET MVC 控制器的占位符，以及要从 ASP.NET MVC *WebApp1* 项目迁移的视图。

1. 复制 ASP.NET MVC 中的方法 `HomeController` 以替换新的 ASP.NET Core `HomeController` 方法。 无需更改操作方法的返回类型。 ASP.NET MVC 内置模板的控制器操作方法返回类型为 <https://docs.microsoft.com/dotnet/api/system.web.mvc.actionresult?view=aspnet-mvc-5.2> ; 在 ASP.NET CORE mvc 中，操作方法将改为返回 `IActionResult` 。 `ActionResult` 可实现 `IActionResult`。
1. 在 ASP.NET Core 项目中，右键单击 " *视图"/"主* " 目录，选择 " **添加** > **现有项** "。
1. 在 " **添加现有项** " 对话框中，导航到 ASP.NET MVC *WebApp1* 项目的 " *视图"/"主* 目录"。
1. 选择 " *关于* *"，然后依次选择 "* *Index.cshtml* Razor **添加** "、"替换现有文件"。

有关详细信息，请参阅 <xref:mvc/controllers/actions> 和 <xref:mvc/views/overview>。

## <a name="test-each-method"></a>测试每个方法

可以测试每个控制器终结点，但在本文档的后面部分介绍了布局和样式。

1. 运行 ASP.NET Core 应用。
1. 通过将当前端口号替换为 ASP.NET Core 项目中使用的端口号，在运行 ASP.NET Core 应用程序的浏览器中调用呈现的视图。 例如 `https://localhost:44375/home/about`。

## <a name="migrate-static-content"></a>迁移静态内容

在 ASP.NET MVC 5 及更早版本中，静态内容是从 web 项目的根目录承载的，与服务器端文件混合。 在 ASP.NET Core 中，静态文件存储在项目的 [web 根目录](xref:fundamentals/index#web-root) 中。 默认目录为 *{content root}/wwwroot* ，但可以对其进行更改。 有关详细信息，请参阅 [ASP.NET Core 中的静态文件](xref:fundamentals/static-files#serve-static-files)。

将 ASP.NET MVC *WebApp1* 项目中的静态内容复制到 ASP.NET Core *WebApp1* 项目中的 *wwwroot* 目录：

1. 在 ASP.NET Core 项目中，右键单击 *wwwroot* 目录，选择 " **添加** > **现有项** "。
1. 在 " **添加现有项** " 对话框中，导航到 "ASP.NET MVC *WebApp1* " 项目。
1. 选择 *favicon* 文件，然后选择 " **添加** "，替换现有文件。

## <a name="migrate-the-layout-files"></a>迁移布局文件

将 ASP.NET MVC 项目布局文件复制到 ASP.NET Core 项目：

1. 在 ASP.NET Core 项目中，右键单击 " *视图* " 目录，选择 " **添加** > **现有项** "。
1. 在 " **添加现有项** " 对话框中，导航到 "ASP.NET MVC *WebApp1* " 项目的 " *视图* " 目录。
1. 选择 *_ViewStart* 的文件，然后选择 " **添加** "。

将 ASP.NET MVC 项目共享布局文件复制到 ASP.NET Core 项目：

1. 在 ASP.NET Core 项目中，右键单击 " *视图"/"共享* " 目录，选择 " **添加** > **现有项** "。
1. 在 " **添加现有项** " 对话框中，导航到 "ASP.NET MVC *WebApp1* " 项目的 " *视图"/"共享* 目录"。
1. 选择 *_Layout* 的文件，然后选择 " **添加** "，替换现有文件。

在 ASP.NET Core 项目中，打开 *_Layout。* 进行以下更改，使其与下面显示的已完成代码相匹配：

更新启动 CSS 包含项以匹配以下已完成的代码：

1. 替换为 `@Styles.Render("~/Content/css")` `<link>` 加载 *启动 .css* 的元素 (参阅下面的) 。
1. 删除 `@Scripts.Render("~/bundles/modernizr")`。

已完成的启动 CSS 包含的替换标记：

```html
<link rel="stylesheet"
    href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
    integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u"
    crossorigin="anonymous">
```

更新 jQuery 和启动 JavaScript 包含项以匹配以下已完成的代码：

1. 替换 `@Scripts.Render("~/bundles/jquery")` 为 `<script>` 元素 (参见下面的) 。
1. 替换 `@Scripts.Render("~/bundles/bootstrap")` 为 `<script>` 元素 (参见下面的) 。

JQuery 和启动 JavaScript 包含的已完成替换标记：

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"
    integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
```

更新后的 *_Layout cshtml* 文件如下所示：

[!code-cshtml[](mvc/samples/3.x/Views/Shared/_Layout.cshtml?highlight=7-10,40-42)]

在浏览器中查看站点。 它应采用所需的样式进行呈现。

## <a name="configure-bundling-and-minification"></a>配置捆绑和缩小

ASP.NET Core 与若干开源绑定和缩减解决方案（例如 [WebOptimizer](https://github.com/ligershark/WebOptimizer) 和其他类似库）兼容。 ASP.NET Core 不提供本机绑定和缩减解决方案。 有关配置绑定和缩减的信息，请参阅 [捆绑和缩减](xref:client-side/bundling-and-minification)。

## <a name="solve-http-500-errors"></a>解决 HTTP 500 错误

有许多问题可能会导致 HTTP 500 错误消息，其中不包含问题根源的相关信息。 例如，如果 *Views/_ViewImports cshtml* 文件包含项目中不存在的命名空间，则会生成 HTTP 500 错误。 默认情况下，在 ASP.NET Core 应用中， `UseDeveloperExceptionPage` 会将扩展添加到， `IApplicationBuilder` 并在 *开发* 环境时执行。 下面的代码对此进行了详细说明：

[!code-csharp[](mvc/samples/3.x/Startup.cs?highlight=17-21&name=snippet)]

ASP.NET Core 将未经处理的异常转换为 HTTP 500 错误响应。 通常，这些响应中不包含错误详细信息，以防止泄露有关服务器的可能敏感信息。 有关详细信息，请参阅 [开发者异常页](xref:fundamentals/error-handling#developer-exception-page)。

## <a name="next-steps"></a>后续步骤

* <xref:migration/identity>

## <a name="additional-resources"></a>其他资源

* <xref:blazor/index>
* <xref:mvc/views/tag-helpers/intro>

::: moniker-end

::: moniker range="= aspnetcore-2.2"

本文介绍如何开始将 ASP.NET MVC 项目迁移到 [ASP.NET CORE mvc](xref:mvc/overview) 2.2。 在此过程中，它突出显示了从 ASP.NET MVC 中更改的许多内容。 从 ASP.NET MVC 迁移是一个多步骤过程。 本文介绍：

* 初始设置
* 基本控制器和视图
* 静态内容
* 客户端依赖关系。

有关迁移配置和 Identity 代码的详细，请参阅 <xref:migration/configuration> 和 <xref:migration/identity> 。

> [!NOTE]
> 示例中的版本号可能不是最新的，请相应地更新项目。

## <a name="create-the-starter-aspnet-mvc-project"></a>创建 starter ASP.NET MVC 项目

为了演示升级，我们首先创建一个 ASP.NET MVC 应用程序。 创建名称为 *WebApp1* 的命名空间，使命名空间与下一步中创建的 ASP.NET Core 项目匹配。

![Visual Studio“新建项目”对话框](mvc/_static/new-project.png)

!["新建 Web 应用程序" 对话框：在 "ASP.NET 模板" 面板中选择的 MVC 项目模板](mvc/_static/new-project-select-mvc-template.png)

*可选：* 将解决方案的名称从 *WebApp1* 更改为 *Mvc5* 。 Visual Studio 将显示新的解决方案名称 ( *Mvc5* ) ，这使你可以更轻松地从下一个项目中通知此项目。

## <a name="create-the-aspnet-core-project"></a>创建 ASP.NET Core 项目

使用与上一个项目相同的名称创建新的 *空* ASP.NET Core web 应用 ( *WebApp1* ) ，使这两个项目中的命名空间匹配。 通过具有相同的命名空间，可以更轻松地在两个项目之间复制代码。 在与前一个项目相同的目录中创建此项目。

![“新建项目”对话框](mvc/_static/new_core.png)

![New ASP.NET Web 应用程序对话框：在 ASP.NET Core 模板 "面板中选择的空项目模板](mvc/_static/new-project-select-empty-aspnet5-template.png)

* *可选：* 使用 " *Web 应用程序* " 项目模板创建新的 ASP.NET Core 应用程序。 将项目命名为 " *WebApp1* "，并选择 **单个用户帐户** 的身份验证选项。 将此应用重命名为 *FullAspNetCore* 。 创建此项目将在转换时节省时间。 可在模板生成的代码中查看最终结果，可以将代码复制到转换项目，或者将代码与模板生成的项目进行比较。

## <a name="configure-the-site-to-use-mvc"></a>将站点配置为使用 MVC

* 面向 .NET Core 时，默认情况下将引用 [AspNetCore 元包](xref:fundamentals/metapackage-app) 。 此包包含 MVC 应用通常使用的包。 如果目标 .NET Framework，则包引用必须单独列出在项目文件中。

`Microsoft.AspNetCore.Mvc` 是 ASP.NET Core MVC 框架。 `Microsoft.AspNetCore.StaticFiles` 为静态文件处理程序。 ASP.NET Core 应用明确选择用于中间件，如用于提供静态文件。 有关详细信息，请参阅[静态文件](xref:fundamentals/static-files)。

* 打开 *Startup.cs* 文件并更改代码，使其与以下内容匹配：

[!code-csharp[](mvc/samples/2.x/Startup.cs?highlight=7,20-25&name=snippet)]

<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*>扩展方法添加静态文件处理程序。 有关详细信息，请参阅 [应用程序启动](xref:fundamentals/startup) 和 [路由](xref:fundamentals/routing)。

## <a name="add-a-controller-and-view"></a>添加控制器和视图

在本部分中，将添加最小控制器和视图，作为 ASP.NET MVC 控制器的占位符和下一部分中迁移的视图。

* 添加 *控制器* 目录。

* 将名为 *HomeController.cs* 的 **控制器类** 添加到 *控制器* 目录。

![“添加新项”对话框](mvc/_static/add_mvc_ctl.png)

* 添加 *Views* 目录。

* 添加 *视图/主* 目录。

* 将名为 *Index* 的 **Razor 视图** 添加到 *Views/Home* 目录。

![“添加新项”对话框](mvc/_static/view.png)

项目结构如下所示：

![显示 WebApp1 的文件和目录的解决方案资源管理器](mvc/_static/project-structure-controller-view.png)

将 Views/Home/Index.cshtml 文件的内容  替换为以下标记：

```html
<h1>Hello world!</h1>
```

运行应用。

![在 Microsoft Edge 中打开的 Web 应用](mvc/_static/hello-world.png)

有关详细信息，请参阅 [控制器](xref:mvc/controllers/actions) 和 [视图](xref:mvc/views/overview)。

以下功能需要从示例 ASP.NET MVC 项目迁移到 ASP.NET Core 项目：

*  (CSS、字体和脚本的客户端内容) 

* 控制器

* 视图

* 模型

* 销售

* 筛选器

* 登录/注销， Identity (在下一教程中完成此操作。 ) 

## <a name="controllers-and-views"></a>控制器和视图

* 将 ASP.NET MVC 中的每个方法复制 `HomeController` 到新的 `HomeController` 。 在 ASP.NET MVC 中，内置模板的控制器操作方法返回类型为 <https://docs.microsoft.com/dotnet/api/system.web.mvc.actionresult?view=aspnet-mvc-5.2> ; 在 ASP.NET CORE mvc 中，操作方法将改为返回 `IActionResult` 。 `ActionResult` 实现 `IActionResult` ，因此无需更改操作方法的返回类型。

* 将 ASP.NET MVC 项目中的 *About* 、 *Contact**和* Razor 视图文件复制到 ASP.NET Core 项目。

## <a name="test-each-method"></a>测试每个方法

尚未迁移布局文件和样式，因此呈现的视图仅包含视图文件中的内容。 和视图的布局文件生成的 `About` 链接 `Contact` 尚不可用。

通过将当前端口号替换为 ASP.NET core 项目中使用的端口号，从正在运行的 ASP.NET core 应用上的浏览器中调用呈现的视图。 例如：`https://localhost:44375/home/about`。

![联系人页](mvc/_static/contact-page.png)

请注意缺少样式和菜单项。 下一节将对样式进行固定。

## <a name="static-content"></a>静态内容

在 ASP.NET MVC 5 及更早版本中，静态内容是从 web 项目的根托管的，与服务器端文件混合。 在 ASP.NET Core 中，静态内容托管在 *wwwroot* 目录中。 将 ASP.NET MVC 应用的静态内容复制到 ASP.NET Core 项目中的 *wwwroot* 目录。 在此示例转换中：

* 将 *favicon* 文件从 ASP.NET MVC 项目复制到 ASP.NET Core 项目中的 *wwwroot* 目录。

ASP.NET MVC 项目使用 [启动](https://getbootstrap.com/) 来设置其样式，并将启动文件存储在 *Content* 和 *Scripts* 目录中。 生成 ASP.NET MVC 项目的模板引用布局文件中的启动 ( *Views/Shared/_Layout* ) 。 *bootstrap.js* 和 *启动 .css* 文件可从 ASP.NET MVC 项目复制到新项目中的 *wwwroot* 目录。 本文档改为使用 Cdn 在下一节中添加对) 使用的启动 (和其他客户端库的支持。

## <a name="migrate-the-layout-file"></a>迁移布局文件

* 将 *_ViewStart* 的 ASP.NET 文件从 MVC 项目的 " *视图* " 目录复制到 ASP.NET Core 项目的 " *视图* " 目录中。 未在 ASP.NET Core MVC 中更改 *_ViewStart cshtml* 文件。

* 创建 *视图/共享* 目录。

* *可选：* 将 *FullAspNetCore* MVC 项目的 *views* 目录中 *_ViewImports* 为 ASP.NET Core 项目的 *视图* 目录。 删除 *_ViewImports cshtml* 文件中的任何命名空间声明。 *_ViewImports 的 cshtml* 文件提供所有视图文件的命名空间，并提供 [标记帮助](xref:mvc/views/tag-helpers/intro)程序。 标记帮助程序在新的布局文件中使用。 *_ViewImports* ASP.NET Core 的新文件。

* 将 *_Layout* 的 ASP.NET 文件从 MVC 项目的 *视图/共享* 目录复制到 ASP.NET Core 项目的 " *视图"/"共享* 目录" 中。

打开 *_Layout.* # 文件，并进行以下更改 (完成的代码如下所示) ：

* 替换为 `@Styles.Render("~/Content/css")` `<link>` 加载 *启动 .css* 的元素 (参阅下面的) 。

* 删除 `@Scripts.Render("~/bundles/modernizr")`。

* 注释掉 `@Html.Partial("_LoginPartial")`) 行 (环绕行的行 `@*...*@` 。 有关详细信息，请参阅 [迁移身份验证和 Identity ASP.NET Core](xref:migration/identity)

* 替换 `@Scripts.Render("~/bundles/jquery")` 为 `<script>` 元素 (参见下面的) 。

* 替换 `@Scripts.Render("~/bundles/bootstrap")` 为 `<script>` 元素 (参见下面的) 。

用于启动 CSS 的替换标记包含：

```html
<link rel="stylesheet"
    href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
    integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u"
    crossorigin="anonymous">
```

JQuery 和启动 JavaScript 的替换标记包含：

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"
    integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
```

更新后的 *_Layout cshtml* 文件如下所示：

[!code-cshtml[](mvc/samples/2.x/Views/Shared/_Layout.cshtml?highlight=7-10,29,41-44)]

在浏览器中查看站点。 它现在应正确加载，并具有所需的样式。

* *可选：* 尝试使用新的布局文件。 复制 *FullAspNetCore* 项目中的布局文件。 新的布局文件使用 [标记帮助](xref:mvc/views/tag-helpers/intro) 程序，并具有其他改进功能。

## <a name="configure-bundling-and-minification"></a>配置捆绑和缩小

有关如何配置绑定和缩减的信息，请参阅 [捆绑和缩减](xref:client-side/bundling-and-minification)。

## <a name="solve-http-500-errors"></a>解决 HTTP 500 错误

有许多问题可能会导致 HTTP 500 错误消息，这些错误消息不包含问题根源的相关信息。 例如，如果 *Views/_ViewImports cshtml* 文件包含项目中不存在的命名空间，则会生成 HTTP 500 错误。 默认情况下，在 ASP.NET Core 应用中，会将 `UseDeveloperExceptionPage` 扩展添加到， `IApplicationBuilder` 并在配置为 *开发* 时执行。 请参阅以下代码中的示例：

[!code-csharp[](mvc/samples/2.x/Startup.cs?highlight=11-15&name=snippet)]

ASP.NET Core 将未经处理的异常转换为 HTTP 500 错误响应。 通常，这些响应中不包含错误详细信息，以防止泄露有关服务器的可能敏感信息。 有关详细信息，请参阅 [开发者异常页](xref:fundamentals/error-handling#developer-exception-page)。

## <a name="additional-resources"></a>其他资源

* <xref:blazor/index>
* <xref:mvc/views/tag-helpers/intro>

::: moniker-end

::: moniker range="<= aspnetcore-2.1"

本文介绍如何开始将 ASP.NET MVC 项目迁移到 [ASP.NET CORE mvc](xref:mvc/overview) 2.1。 在此过程中，它突出显示了从 ASP.NET MVC 中更改的许多内容。 从 ASP.NET MVC 迁移是一个多步骤过程。 本文介绍：

* 初始设置
* 基本控制器和视图
* 静态内容
* 客户端依赖关系。

若要迁移配置和 Identity 代码，请参阅将 [配置迁移到 ASP.NET Core](xref:migration/configuration) 并 [迁移身份验证和 Identity ASP.NET Core](xref:migration/identity)。

> [!NOTE]
> 示例中的版本号可能不是最新的，请相应地更新项目。

## <a name="create-the-starter-aspnet-mvc-project"></a>创建 starter ASP.NET MVC 项目

为了演示升级，我们首先创建一个 ASP.NET MVC 应用程序。 创建名称为 *WebApp1* 的命名空间，使命名空间与下一步中创建的 ASP.NET Core 项目匹配。

![Visual Studio“新建项目”对话框](mvc/_static/new-project.png)

!["新建 Web 应用程序" 对话框：在 "ASP.NET 模板" 面板中选择的 MVC 项目模板](mvc/_static/new-project-select-mvc-template.png)

*可选：* 将解决方案的名称从 *WebApp1* 更改为 *Mvc5* 。 Visual Studio 将显示新的解决方案名称 ( *Mvc5* ) ，这使你可以更轻松地从下一个项目中通知此项目。

## <a name="create-the-aspnet-core-project"></a>创建 ASP.NET Core 项目

使用与上一个项目相同的名称创建新的 *空* ASP.NET Core web 应用 ( *WebApp1* ) ，使这两个项目中的命名空间匹配。 通过具有相同的命名空间，可以更轻松地在两个项目之间复制代码。 在与前一个项目相同的目录中创建此项目。

![“新建项目”对话框](mvc/_static/new_core.png)

![New ASP.NET Web 应用程序对话框：在 ASP.NET Core 模板 "面板中选择的空项目模板](mvc/_static/new-project-select-empty-aspnet5-template.png)

* *可选：* 使用 " *Web 应用程序* " 项目模板创建新的 ASP.NET Core 应用程序。 将项目命名为 " *WebApp1* "，并选择 **单个用户帐户** 的身份验证选项。 将此应用重命名为 *FullAspNetCore* 。 创建此项目将在转换时节省时间。 可在模板生成的代码中查看最终结果，可以将代码复制到转换项目，或者将代码与模板生成的项目进行比较。

## <a name="configure-the-site-to-use-mvc"></a>将站点配置为使用 MVC

* 面向 .NET Core 时，默认情况下将引用 [AspNetCore 元包](xref:fundamentals/metapackage-app) 。 此包包含 MVC 应用通常使用的包。 如果目标 .NET Framework，则包引用必须单独列出在项目文件中。

`Microsoft.AspNetCore.Mvc` 是 ASP.NET Core MVC 框架。 `Microsoft.AspNetCore.StaticFiles` 为静态文件处理程序。 ASP.NET Core 应用明确选择用于中间件，如用于提供静态文件。 有关详细信息，请参阅[静态文件](xref:fundamentals/static-files)。

* 打开 *Startup.cs* 文件并更改代码，使其与以下内容匹配：

[!code-csharp[](mvc/samples/2.x/Startup.cs?highlight=7,20-25&name=snippet)]

<xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles*>扩展方法添加静态文件处理程序。 `UseMvc`扩展方法将添加路由。 有关详细信息，请参阅 [应用程序启动](xref:fundamentals/startup) 和 [路由](xref:fundamentals/routing)。

## <a name="add-a-controller-and-view"></a>添加控制器和视图

在本部分中，将添加最小控制器和视图，作为 ASP.NET MVC 控制器的占位符和下一部分中迁移的视图。

* 添加 *控制器* 目录。

* 将名为 *HomeController.cs* 的 **控制器类** 添加到 *控制器* 目录。

![“添加新项”对话框](mvc/_static/add_mvc_ctl.png)

* 添加 *Views* 目录。

* 添加 *视图/主* 目录。

* 将名为 *Index* 的 **Razor 视图** 添加到 *Views/Home* 目录。

![“添加新项”对话框](mvc/_static/view.png)

项目结构如下所示：

![显示 WebApp1 的文件和目录的解决方案资源管理器](mvc/_static/project-structure-controller-view.png)

将 Views/Home/Index.cshtml 文件的内容  替换为以下标记：

```html
<h1>Hello world!</h1>
```

运行应用。

![在 Microsoft Edge 中打开的 Web 应用](mvc/_static/hello-world.png)

有关详细信息，请参阅 [控制器](xref:mvc/controllers/actions) 和 [视图](xref:mvc/views/overview)。

以下功能需要从示例 ASP.NET MVC 项目迁移到 ASP.NET Core 项目：

*  (CSS、字体和脚本的客户端内容) 

* 控制器

* 视图

* 模型

* 销售

* 筛选器

* 登录/注销， Identity (在下一教程中完成此操作。 ) 

## <a name="controllers-and-views"></a>控制器和视图

* 将 ASP.NET MVC 中的每个方法复制 `HomeController` 到新的 `HomeController` 。 在 ASP.NET MVC 中，内置模板的控制器操作方法返回类型为 <https://docs.microsoft.com/dotnet/api/system.web.mvc.actionresult?view=aspnet-mvc-5.2> ; 在 ASP.NET CORE mvc 中，操作方法将改为返回 `IActionResult` 。 `ActionResult` 实现 `IActionResult` ，因此无需更改操作方法的返回类型。

* 将 ASP.NET MVC 项目中的 *About* 、 *Contact**和* Razor 视图文件复制到 ASP.NET Core 项目。

## <a name="test-each-method"></a>测试每个方法

尚未迁移布局文件和样式，因此呈现的视图仅包含视图文件中的内容。 和视图的布局文件生成的 `About` 链接 `Contact` 尚不可用。

* 通过将当前端口号替换为 ASP.NET core 项目中使用的端口号，从正在运行的 ASP.NET core 应用上的浏览器中调用呈现的视图。 例如：`https://localhost:44375/home/about`。

![联系人页](mvc/_static/contact-page.png)

请注意缺少样式和菜单项。 下一节将对样式进行固定。

## <a name="static-content"></a>静态内容

在 ASP.NET MVC 5 及更早版本中，静态内容是从 web 项目的根托管的，与服务器端文件混合。 在 ASP.NET Core 中，静态内容托管在 *wwwroot* 目录中。 将 ASP.NET MVC 应用的静态内容复制到 ASP.NET Core 项目中的 *wwwroot* 目录。 在此示例转换中：

* 将 *favicon* 文件从 ASP.NET MVC 项目复制到 ASP.NET Core 项目中的 *wwwroot* 目录。

ASP.NET MVC 项目使用 [启动](https://getbootstrap.com/) 来设置其样式，并将启动文件存储在 *Content* 和 *Scripts* 目录中。 生成 ASP.NET MVC 项目的模板引用布局文件中的启动 ( *Views/Shared/_Layout* ) 。 *bootstrap.js* 和 *启动 .css* 文件可从 ASP.NET MVC 项目复制到新项目中的 *wwwroot* 目录。 本文档改为使用 Cdn 在下一节中添加对) 使用的启动 (和其他客户端库的支持。

## <a name="migrate-the-layout-file"></a>迁移布局文件

* 将 *_ViewStart* 的 ASP.NET 文件从 MVC 项目的 " *视图* " 目录复制到 ASP.NET Core 项目的 " *视图* " 目录中。 未在 ASP.NET Core MVC 中更改 *_ViewStart cshtml* 文件。

* 创建 *视图/共享* 目录。

* *可选：* 将 *FullAspNetCore* MVC 项目的 *views* 目录中 *_ViewImports* 为 ASP.NET Core 项目的 *视图* 目录。 删除 *_ViewImports cshtml* 文件中的任何命名空间声明。 *_ViewImports 的 cshtml* 文件提供所有视图文件的命名空间，并提供 [标记帮助](xref:mvc/views/tag-helpers/intro)程序。 标记帮助程序在新的布局文件中使用。 *_ViewImports* ASP.NET Core 的新文件。

* 将 *_Layout* 的 ASP.NET 文件从 MVC 项目的 *视图/共享* 目录复制到 ASP.NET Core 项目的 " *视图"/"共享* 目录" 中。

打开 *_Layout.* # 文件，并进行以下更改 (完成的代码如下所示) ：

* 替换为 `@Styles.Render("~/Content/css")` `<link>` 加载 *启动 .css* 的元素 (参阅下面的) 。

* 删除 `@Scripts.Render("~/bundles/modernizr")`。

* 注释掉 `@Html.Partial("_LoginPartial")`) 行 (环绕行的行 `@*...*@` 。 有关详细信息，请参阅 [迁移身份验证和 Identity ASP.NET Core](xref:migration/identity)

* 替换 `@Scripts.Render("~/bundles/jquery")` 为 `<script>` 元素 (参见下面的) 。

* 替换 `@Scripts.Render("~/bundles/bootstrap")` 为 `<script>` 元素 (参见下面的) 。

用于启动 CSS 的替换标记包含：

```html
<link rel="stylesheet"
    href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css"
    integrity="sha384-BVYiiSIFeK1dGmJRAkycuHAHRg32OmUcww7on3RYdg4Va+PmSTsz/K68vbdEjh4u"
    crossorigin="anonymous">
```

JQuery 和启动 JavaScript 的替换标记包含：

```html
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"
    integrity="sha384-Tc5IQib027qvyjSMfHjOMaLkfuWVxZxUPnCJA7l2mCWNIpG9mGCD8wGNIcPD7Txa" crossorigin="anonymous"></script>
```

更新后的 *_Layout cshtml* 文件如下所示：

[!code-cshtml[](mvc/samples/2.x/Views/Shared/_Layout.cshtml?highlight=7-10,29,41-44)]

在浏览器中查看站点。 它现在应正确加载，并具有所需的样式。

* *可选：* 尝试使用新的布局文件。 复制 *FullAspNetCore* 项目中的布局文件。 新的布局文件使用 [标记帮助](xref:mvc/views/tag-helpers/intro) 程序，并具有其他改进功能。

## <a name="configure-bundling-and-minification"></a>配置捆绑和缩小

有关如何配置绑定和缩减的信息，请参阅 [捆绑和缩减](xref:client-side/bundling-and-minification)。

## <a name="solve-http-500-errors"></a>解决 HTTP 500 错误

有许多问题可能会导致 HTTP 500 错误消息，这些错误消息不包含问题根源的相关信息。 例如，如果 *Views/_ViewImports cshtml* 文件包含项目中不存在的命名空间，则会生成 HTTP 500 错误。 默认情况下，在 ASP.NET Core 应用中，会将 `UseDeveloperExceptionPage` 扩展添加到， `IApplicationBuilder` 并在配置为 *开发* 时执行。 请参阅以下代码中的示例：

[!code-csharp[](mvc/samples/2.x/Startup.cs?highlight=11-15&name=snippet)]

ASP.NET Core 将未经处理的异常转换为 HTTP 500 错误响应。 通常，这些响应中不包含错误详细信息，以防止泄露有关服务器的可能敏感信息。 有关详细信息，请参阅 [开发者异常页](xref:fundamentals/error-handling#developer-exception-page)。

## <a name="additional-resources"></a>其他资源

* <xref:blazor/index>
* <xref:mvc/views/tag-helpers/intro>

::: moniker-end
