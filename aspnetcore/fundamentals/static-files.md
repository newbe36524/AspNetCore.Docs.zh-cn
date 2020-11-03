---
title: ASP.NET Core 中的静态文件
author: rick-anderson
description: 了解如何在 ASP.NET Core Web 应用中提供和保护静态文件，以及如何配置静态文件托管中间件行为。
ms.author: riande
ms.custom: mvc
ms.date: 6/23/2020
no-loc:
- ASP.NET Core Identity
- cookie
- Cookie
- Blazor
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: fundamentals/static-files
ms.openlocfilehash: 36eeb3532ed5ea6f5f75c20cbe14331ed840a788
ms.sourcegitcommit: 74f4a4ddbe3c2f11e2e09d05d2a979784d89d3f5
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/27/2020
ms.locfileid: "91393933"
---
# <a name="static-files-in-aspnet-core"></a>ASP.NET Core 中的静态文件

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Kirk Larkin](https://twitter.com/serpent5)

默认情况下，静态文件（如 HTML、CSS、图像和 JavaScript）是 ASP.NET Core 应用直接提供给客户端的资产。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/static-files/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="serve-static-files"></a>提供静态文件

静态文件存储在项目的 [Web 根](xref:fundamentals/index#web-root)目录中。 默认目录为 `{content root}/wwwroot`，但可通过 <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot%2A> 方法更改目录。 有关详细信息，请参阅[内容根目录](xref:fundamentals/index#content-root)和 [Web 根目录](xref:fundamentals/index#web-root)。

采用 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder%2A> 方法可将内容根目录设置为当前目录：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/Program2.cs?name=snippet_Program)]

上述代码是使用 Web 应用程序模板创建的。

可通过 [Web 根目录](xref:fundamentals/index#web-root)的相关路径访问静态文件。 例如，Web 应用程序项目模板包含 `wwwroot` 文件夹中的多个文件夹：

* `wwwroot`
  * `css`
  * `js`
  * `lib`

请考虑创建 wwwroot/images 文件夹，并添加 wwwroot/images/MyImage 文件。  用于访问 `images` 文件夹中的文件的 URI 格式为 `https://<hostname>/images/<image_file_name>`。 例如，`https://localhost:5001/images/MyImage.jpg`

### <a name="serve-files-in-web-root"></a>在 Web 根目录中提供文件

默认 Web 应用模板在 `Startup.Configure` 中调用 <xref:Owin.StaticFileExtensions.UseStaticFiles%2A> 方法，这将允许提供静态文件：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/Startup.cs?name=snippet_Configure&highlight=15)]

无参数 `UseStaticFiles` 方法重载将 [Web 根目录](xref:fundamentals/index#web-root)中的文件标记为可用。 以下标记引用 wwwroot/images/MyImage.jpg：

```html
<img src="~/images/MyImage.jpg" class="img" alt="My image" />
```

在上面的代码中，波形符 `~/` 指向 [Web 根目录](xref:fundamentals/index#web-root)。

### <a name="serve-files-outside-of-web-root"></a>提供 Web 根目录外的文件

考虑一个目录层次结构，其中要提供的静态文件位于 [Web 根目录](xref:fundamentals/index#web-root)之外：

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `red-rose.jpg`

按如下方式配置静态文件中间件后，请求可访问 `red-rose.jpg` 文件：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupRose.cs?name=snippet_Configure&highlight=15-22)]

在前面的代码中，MyStaticFiles 目录层次结构通过 StaticFiles URI 段公开 。 对 `https://<hostname>/StaticFiles/images/red-rose.jpg` 的请求将提供 red-rose.jpg 文件。

以下标记引用 MyStaticFiles/images/red-rose.jpg：

```html
<img src="~/StaticFiles/images/red-rose.jpg" class="img" alt="A red rose" />
```

### <a name="set-http-response-headers"></a>设置 HTTP 响应标头

<xref:Microsoft.AspNetCore.Builder.StaticFileOptions> 对象可用于设置 HTTP 响应标头。 除配置从 [Web 根目录](xref:fundamentals/index#web-root)提供静态文件外，以下代码还设置 `Cache-Control` 标头：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupAddHeader.cs?name=snippet_Configure&highlight=15-24)]

<!-- Q: The preceding code sets max-age to 604800 seconds (7 days), so what does the following mean? -->

静态文件可在 600 秒内公开缓存：

![已添加显示 Cache-Control 标头的响应头](static-files/_static/add-header.png)

## <a name="static-file-authorization"></a>静态文件授权

静态文件中间件不提供授权检查。 可公开访问由静态文件中间件提供的任何文件，包括 `wwwroot` 下的文件。 根据授权提供文件：

* 将文件存储在 `wwwroot` 和默认静态文件中间件可访问的任何目录之外。
* 在 `UseAuthorization` 后面调用 `UseStaticFiles` 并指定路径：

  [!code-csharp[](static-files/samples/3.x/StaticFileAuth/Startup.cs?name=snippet2)]
  
  上述方法要求用户已进行身份验证：

  [!code-csharp[](static-files/samples/3.x/StaticFileAuth/Startup.cs?name=snippet1&highlight=20-99)]

   [!INCLUDE[](~/includes/requireAuth.md)]

还有一种根据授权提供文件的方法是：

* 将文件存储在 `wwwroot` 和静态文件中间件可访问的任何目录之外。
* 通过应用授权的操作方法为其提供服务，并返回 <xref:Microsoft.AspNetCore.Mvc.FileResult> 对象：

  [!code-csharp[](static-files/samples/3.x/StaticFilesSample/Controllers/HomeController.cs?name=snippet_BannerImage)]

## <a name="directory-browsing"></a>目录浏览

目录浏览允许在指定目录中列出目录。

出于安全考虑，目录浏览默认处于禁用状态。 有关详细信息，请参阅[注意事项](#considerations)。

通过以下方式启用目录浏览：

* `Startup.ConfigureServices` 中的 <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A>。
* `Startup.Configure` 中的 <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser%2A>。

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ClassMembers&highlight=4,21-35)]

上述代码允许使用 URL `https://<hostname>/MyImages` 浏览 wwwroot/images 文件夹的目录，并链接到每个文件和文件夹：

![目录浏览](static-files/_static/dir-browse.png)

## <a name="serve-default-documents"></a>提供默认文档

设置默认页面为访问者提供网站的起点。 若要在没有完全限定 URI 的情况下提供 `wwwroot` 的默认页面，请调用 <xref:Owin.DefaultFilesExtensions.UseDefaultFiles%2A> 方法：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty.cs?name=snippet_Configure&highlight=15)]

要提供默认文件，必须在 `UseStaticFiles` 前调用 `UseDefaultFiles`。 `UseDefaultFiles` 是不提供文件的 URL 重写工具。

使用 `UseDefaultFiles` 请求对 `wwwroot` 中的文件夹搜索：

* default.htm
* default.html
* index.htm
* index.html

将请求视为完全限定 URI，提供在列表中找到的第一个文件。 浏览器 URL 继续反映请求的 URI。

以下代码将默认文件名更改为 mydefault.html：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupDefault.cs?name=snippet_DefaultFiles)]

以下代码通过上述代码演示了 `Startup.Configure`：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupDefault.cs?name=snippet_Configure&highlight=15-19)]

### <a name="usefileserver-for-default-documents"></a>默认文档的 UseFileServer

<xref:Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer*> 结合了 `UseStaticFiles`、`UseDefaultFiles` 和 `UseDirectoryBrowser`（可选）的功能。

调用 `app.UseFileServer` 以提供静态文件和默认文件。 未启用目录浏览。 以下代码通过 `UseFileServer` 演示了 `Startup.Configure`：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty2.cs?name=snippet_Configure&highlight=15)]

以下代码提供静态文件、默认文件和目录浏览：

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

以下代码通过上述代码演示了 `Startup.Configure`：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupEmpty3.cs?name=snippet_Configure&highlight=15)]

考虑以下目录层次结构：

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `MyImage.jpg`
  * `default.html`

以下代码提供静态文件、默认文件和 `MyStaticFiles` 的目录浏览：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileServer.cs?name=snippet_ClassMembers&highlight=4,21-31)]

必须在 `EnableDirectoryBrowsing` 属性值为 `true` 时调用 <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A>。

使用文件层次结构和前面的代码，URL 解析如下：

| URI            |      响应  |
| ------- | ------|
| `https://<hostname>/StaticFiles/images/MyImage.jpg` | MyStaticFiles/images/MyImage.jpg |
| `https://<hostname>/StaticFiles` | MyStaticFiles/default.html |

如果 MyStaticFiles 目录中不存在默认命名文件，则 `https://<hostname>/StaticFiles` 返回包含可单击链接的目录列表：

![静态文件列表](static-files/_static/db2.png)

<xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles*> 和 <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser*> 执行从不带尾部 `/` 的目标 URI 到带尾部 `/` 的目标 URI 的客户端重定向。 例如，从 `https://<hostname>/StaticFiles` 到 `https://<hostname>/StaticFiles/`。 如果没有尾部斜杠 (`/`)，StaticFiles 目录中的相对 URL 无效。

## <a name="fileextensioncontenttypeprovider"></a>FileExtensionContentTypeProvider

<xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> 类包含 `Mappings` 属性，用作文件扩展名到 MIME 内容类型的映射。 在以下示例中，多个文件扩展名映射到了已知的 MIME 类型。 替换了 .rtf 扩展名，删除了 .mp4 ：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_Provider)]

以下代码通过上述代码演示了 `Startup.Configure`：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_Configure&highlight=15-43)]

请参阅 [MIME 内容类型](https://www.iana.org/assignments/media-types/media-types.xhtml)。

## <a name="non-standard-content-types"></a>非标准内容类型

静态文件中间件可理解近 400 种已知文件内容类型。 如果用户请求文件类型未知的文件，则静态文件中间件将请求传递给管道中的下一个中间件。 如果没有中间件处理请求，则返回“404 未找到”响应。 如果启用了目录浏览，则在目录列表中会显示该文件的链接。

以下代码提供未知类型，并以图像形式呈现未知文件：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_UseStaticFiles)]

以下代码通过上述代码演示了 `Startup.Configure`：

[!code-csharp[](~/fundamentals/static-files/samples/3.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_Configure&highlight=15-19)]

使用前面的代码，请求的文件含未知内容类型时，以图像形式返回请求。

> [!WARNING]
> 启用 <xref:Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes> 会形成安全隐患。 它默认处于禁用状态，不建议使用。 [FileExtensionContentTypeProvider](#fileextensioncontenttypeprovider) 提供了更安全的替代方法来提供含非标准扩展名的文件。

## <a name="serve-files-from-multiple-locations"></a>从多个位置提供文件

`UseStaticFiles` 和 `UseFileServer` 默认为指向 `wwwroot` 的文件提供程序。 可使用其他文件提供程序提供 `UseStaticFiles` 和 `UseFileServer` 的其他实例，从多个位置提供文件。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/15578)。

<a name="sc"></a>

### <a name="security-considerations-for-static-files"></a>静态文件的安全注意事项

> [!WARNING]
> `UseDirectoryBrowser` 和 `UseStaticFiles` 可能会泄漏机密。 强烈建议在生产中禁用目录浏览。 请仔细查看 `UseStaticFiles` 或 `UseDirectoryBrowser` 启用了哪些目录。 整个目录及其子目录均可公开访问。 将适合公开的文件存储在专用目录中，如 `<content_root>/wwwroot`。 将这些文件与 MVC 视图、Razor Pages 和配置文件等分开。

* 使用 `UseDirectoryBrowser` 和 `UseStaticFiles` 公开的内容的 URL 受大小写和基础文件系统字符限制的影响。 例如，Windows 不区分大小写，但 macOS 和 Linux 区分大小写。

* 托管于 IIS 中的 ASP.NET Core 应用使用 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)将所有请求转发到应用，包括静态文件请求。 不使用 IIS 静态文件处理程序，并且没有机会处理请求。

* 在 IIS Manager 中完成以下步骤，删除服务器或网站级别的 IIS 静态文件处理程序：
    1. 转到“模块”功能。
    1. 在列表中选择 StaticFileModule。
    1. 单击“操作”侧栏中的“删除” 。

> [!WARNING]
> 如果启用了 IIS 静态文件处理程序且 ASP.NET Core 模块配置不正确，则会提供静态文件。 例如，如果未部署 web.config 文件，则会发生这种情况。

* 将代码文件（包括 .cs 和 .cshtml）放在应用项目的 [Web 根目录](xref:fundamentals/index#web-root)之外 。 这样就在应用的客户端内容和基于服务器的代码间创建了逻辑分隔。 可以防止服务器端代码泄漏。

## <a name="additional-resources"></a>其他资源

* [中间件](xref:fundamentals/middleware/index)
* [ASP.NET Core 简介](xref:index)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Scott Addie](https://twitter.com/Scott_Addie)

静态文件（如 HTML、CSS、图像和 JavaScript）是 ASP.NET Core 应用直接提供给客户端的资产。 需要进行一些配置才能提供这些文件。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/static-files/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="serve-static-files"></a>提供静态文件

静态文件存储在项目的 [Web 根](xref:fundamentals/index#web-root)目录中。 默认目录是 {content root}/wwwroot，但可通过 <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseWebRoot%2A> 方法更改目录。 有关详细信息，请参阅[内容根目录](xref:fundamentals/index#content-root)和 [Web 根目录](xref:fundamentals/index#web-root)。

应用的 Web 主机必须识别内容根目录。

采用 `WebHost.CreateDefaultBuilder` 方法可将内容根目录设置为当前目录：

[!code-csharp[](../common/samples/WebApplication1DotNetCore2.0App/Program.cs?name=snippet_Main&highlight=9)]

可通过 [Web 根目录](xref:fundamentals/index#web-root)的相关路径访问静态文件。 例如，Web 应用程序项目模板包含 `wwwroot` 文件夹中的多个文件夹：

* `wwwroot`
  * `css`
  * `images`
  * `js`

用于访问 images 子文件夹中的文件的 URI 格式为 http://\<server_address>/images/\<image_file_name> 。 例如， *http://localhost:9189/images/banner3.svg* 。

如果以 .NET Framework 为目标，请将 [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles/) 包添加到项目。 如果以 .NET Core 为目标，[Microsoft.AspNetCore.App 元包](xref:fundamentals/metapackage-app)将包括此包。

配置提供静态文件的[中间件](xref:fundamentals/middleware/index)。

### <a name="serve-files-inside-of-web-root"></a>提供 Web 根目录内的文件

调用 `Startup.Configure` 中的 <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> 方法：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupStaticFiles.cs?name=snippet_ConfigureMethod&highlight=3)]

无参数 `UseStaticFiles` 方法重载将 [Web 根目录](xref:fundamentals/index#web-root)中的文件标记为可用。 以下标记引用 wwwroot/images/banner1.svg：

[!code-cshtml[](static-files/samples/1.x/StaticFilesSample/Views/Home/Index.cshtml?name=snippet_static_file_wwwroot)]

在上面的代码中，波形符 `~/` 指向 [Web 根目录](xref:fundamentals/index#web-root)。

### <a name="serve-files-outside-of-web-root"></a>提供 Web 根目录外的文件

考虑一个目录层次结构，其中要提供的静态文件位于 [Web 根目录](xref:fundamentals/index#web-root)之外：

* `wwwroot`
  * `css`
  * `images`
  * `js`
* `MyStaticFiles`
  * `images`
    * `banner1.svg`

按如下方式配置静态文件中间件后，请求可访问 banner1.svg 文件：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupTwoStaticFiles.cs?name=snippet_ConfigureMethod&highlight=5-10)]

在前面的代码中，MyStaticFiles 目录层次结构通过 StaticFiles URI 段公开 。 请求 http://\<server_address>/StaticFiles/images/banner1.svg 提供 banner1.svg 文件 。

以下标记引用 MyStaticFiles/images/banner1.svg：

[!code-cshtml[](static-files/samples/1.x/StaticFilesSample/Views/Home/Index.cshtml?name=snippet_static_file_outside)]

### <a name="set-http-response-headers"></a>设置 HTTP 响应标头

<xref:Microsoft.AspNetCore.Builder.StaticFileOptions> 对象可用于设置 HTTP 响应标头。 除配置从 [Web 根目录](xref:fundamentals/index#web-root)提供静态文件外，以下代码还设置 `Cache-Control` 标头：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupAddHeader.cs?name=snippet_ConfigureMethod)]
[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

<xref:Microsoft.AspNetCore.Http.HeaderDictionaryExtensions.Append%2A?displayProperty=nameWithType> 方法位于 [Microsoft.AspNetCore.Http](https://www.nuget.org/packages/Microsoft.AspNetCore.Http/) 包中。

在开发环境中可公开缓存这些文件 10 分钟（600 秒）：

![已添加显示 Cache-Control 标头的响应头](static-files/_static/add-header.png)

## <a name="static-file-authorization"></a>静态文件授权

静态文件中间件不提供授权检查。 可公开访问由静态文件中间件提供的任何文件，包括 wwwroot 下的文件。 根据授权提供文件：

* 将文件存储在 wwwroot 和静态文件中间件可访问的任何目录之外。
* 通过有授权的操作方法提供文件。 返回一个 <xref:Microsoft.AspNetCore.Mvc.FileResult> 对象：

  [!code-csharp[](static-files/samples/1.x/StaticFilesSample/Controllers/HomeController.cs?name=snippet_BannerImageAction)]

## <a name="enable-directory-browsing"></a>启用目录浏览

通过目录浏览，Web 应用的用户可查看目录列表和指定目录中的文件。 出于安全考虑，目录浏览默认处于禁用状态（请参阅[注意事项](#considerations)）。 调用 `Startup.Configure` 中的 <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser%2A> 方法来启用目录浏览：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureMethod&highlight=12-17)]

通过从 `Startup.ConfigureServices` 调用 <xref:Microsoft.Extensions.DependencyInjection.DirectoryBrowserServiceExtensions.AddDirectoryBrowser%2A> 方法添加所需服务：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureServicesMethod&highlight=3)]

上述代码允许使用 URL http://\<server_address>/MyImages 浏览 wwwroot/images 文件夹的目录，并链接到每个文件和文件夹 ：

![目录浏览](static-files/_static/dir-browse.png)

有关启用浏览时的安全风险，请参阅[注意事项](#considerations)。

请注意以下示例中的两个 `UseStaticFiles` 调用。 第一个调用提供 wwwroot 文件夹中的静态文件。 第二个调用使用 URL http://\<server_address>/MyImages 浏览 wwwroot/images 文件夹的目录 ：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupBrowse.cs?name=snippet_ConfigureMethod&highlight=3,5)]

## <a name="serve-a-default-document"></a>提供默认文档

设置默认主页为访问者访问网站时提供了逻辑起点。 若要在用户不完全限定 URI 的情况下提供默认页面，请调用 `Startup.Configure` 中的 <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles%2A> 方法：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupEmpty.cs?name=snippet_ConfigureMethod&highlight=3)]

> [!IMPORTANT]
> 要提供默认文件，必须在 `UseStaticFiles` 前调用 `UseDefaultFiles`。 `UseDefaultFiles` 实际上用于重写 URL，不提供文件。 通过 `UseStaticFiles` 启用静态文件中间件来提供文件。

使用 `UseDefaultFiles` 请求文件夹搜索：

* default.htm
* default.html
* index.htm
* index.html

将请求视为完全限定 URI，提供在列表中找到的第一个文件。 浏览器 URL 继续反映请求的 URI。

以下代码将默认文件名更改为 mydefault.html：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupDefault.cs?name=snippet_ConfigureMethod)]

## <a name="usefileserver"></a>UseFileServer

<xref:Microsoft.AspNetCore.Builder.FileServerExtensions.UseFileServer*> 结合了 `UseStaticFiles`、`UseDefaultFiles` 和 `UseDirectoryBrowser`（可选）的功能。

以下代码提供静态文件和默认文件。 未启用目录浏览。

```csharp
app.UseFileServer();
```

以下代码通过启用目录浏览基于无参数重载进行构建：

```csharp
app.UseFileServer(enableDirectoryBrowsing: true);
```

考虑以下目录层次结构：

* **wwwroot**
  * **css**
  * **images**
  * **js**
* **MyStaticFiles**
  * **images**
    * banner1.svg
  * default.html

以下代码启用静态文件、默认文件和及 `MyStaticFiles` 的目录浏览：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupUseFileServer.cs?name=snippet_ConfigureMethod&highlight=5-11)]

`EnableDirectoryBrowsing` 属性值为 `true` 时必须调用 `AddDirectoryBrowser`：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupUseFileServer.cs?name=snippet_ConfigureServicesMethod)]

使用文件层次结构和前面的代码，URL 解析如下：

| URI            |                             响应  |
| ------- | ------|
| http://\<server_address>/StaticFiles/images/banner1.svg    |      MyStaticFiles/images/banner1.svg |
| http://\<server_address>/StaticFiles             |     MyStaticFiles/default.html |

如果 MyStaticFiles 目录中不存在默认命名文件，则 http://\<server_address>/StaticFiles 返回包含可单击链接的目录列表 ：

![静态文件列表](static-files/_static/db2.png)

> [!NOTE]
> <xref:Microsoft.AspNetCore.Builder.DefaultFilesExtensions.UseDefaultFiles*> 和 <xref:Microsoft.AspNetCore.Builder.DirectoryBrowserExtensions.UseDirectoryBrowser*> 执行从 `http://{SERVER ADDRESS}/StaticFiles`（不带尾部斜杠）到 `http://{SERVER ADDRESS}/StaticFiles/`（带尾部斜杠）的客户端重定向。 如果没有尾部斜杠，StaticFiles 目录中的相对 URL 无效。

## <a name="fileextensioncontenttypeprovider"></a>FileExtensionContentTypeProvider

<xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> 类包含 `Mappings` 属性，用作文件扩展名到 MIME 内容类型的映射。 在以下示例中，多个文件扩展名注册到了已知的 MIME 类型。 替换了 .rtf 扩展名，删除了 .mp4 。

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupFileExtensionContentTypeProvider.cs?name=snippet_ConfigureMethod&highlight=3-12,19)]

请参阅 [MIME 内容类型](https://www.iana.org/assignments/media-types/media-types.xhtml)。

若要了解如何使用自定义 <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider>，或者要配置 Blazor Server 应用中的其他 <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>，请参阅 <xref:blazor/fundamentals/additional-scenarios#static-files>。

## <a name="non-standard-content-types"></a>非标准内容类型

静态文件中间件可理解近 400 种已知文件内容类型。 如果用户请求文件类型未知的文件，则静态文件中间件将请求传递给管道中的下一个中间件。 如果没有中间件处理请求，则返回“404 未找到”响应。 如果启用了目录浏览，则在目录列表中会显示该文件的链接。

以下代码提供未知类型，并以图像形式呈现未知文件：

[!code-csharp[](static-files/samples/1.x/StaticFilesSample/StartupServeUnknownFileTypes.cs?name=snippet_ConfigureMethod)]

使用前面的代码，请求的文件含未知内容类型时，以图像形式返回请求。

> [!WARNING]
> 启用 <xref:Microsoft.AspNetCore.Builder.StaticFileOptions.ServeUnknownFileTypes> 会形成安全隐患。 它默认处于禁用状态，不建议使用。 [FileExtensionContentTypeProvider](#fileextensioncontenttypeprovider) 提供了更安全的替代方法来提供含非标准扩展名的文件。

## <a name="serve-files-from-multiple-locations"></a>从多个位置提供文件

`UseStaticFiles` 和 `UseFileServer` 默认为指向 wwwroot 的文件提供程序。 可使用其他文件提供程序提供 `UseStaticFiles` 和 `UseFileServer` 的其他实例，从多个位置提供文件。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/15578)。

### <a name="considerations"></a>注意事项

> [!WARNING]
> `UseDirectoryBrowser` 和 `UseStaticFiles` 可能会泄漏机密。 强烈建议在生产中禁用目录浏览。 请仔细查看 `UseStaticFiles` 或 `UseDirectoryBrowser` 启用了哪些目录。 整个目录及其子目录均可公开访问。 将适合公开的文件存储在专用目录中，如 \<content_root>/wwwroot。 将这些文件与 MVC 视图、Razor 页面（仅限 2.x）和配置文件等分开。

* 使用 `UseDirectoryBrowser` 和 `UseStaticFiles` 公开的内容的 URL 受大小写和基础文件系统字符限制的影响。 例如，Windows 不区分大小写 &mdash; macOS 和 Linux 却要区分。

* 托管于 IIS 中的 ASP.NET Core 应用使用 [ASP.NET Core 模块](xref:host-and-deploy/aspnet-core-module)将所有请求转发到应用，包括静态文件请求。 未使用 IIS 静态文件处理程序。 在模块处理请求前，处理程序没有机会处理请求。

* 在 IIS Manager 中完成以下步骤，删除服务器或网站级别的 IIS 静态文件处理程序：
    1. 转到“模块”功能。
    1. 在列表中选择 StaticFileModule。
    1. 单击“操作”侧栏中的“删除” 。

> [!WARNING]
> 如果启用了 IIS 静态文件处理程序且 ASP.NET Core 模块配置不正确，则会提供静态文件。 例如，如果未部署 web.config 文件，则会发生这种情况。

* 将代码文件（包括 .cs 和 .cshtml ）放在应用项目的 [Web 根目录](xref:fundamentals/index#web-root)之外 。 这样就在应用的客户端内容和基于服务器的代码间创建了逻辑分隔。 可以防止服务器端代码泄漏。

## <a name="additional-resources"></a>其他资源

* [中间件](xref:fundamentals/middleware/index)
* [ASP.NET Core 简介](xref:index)

::: moniker-end
