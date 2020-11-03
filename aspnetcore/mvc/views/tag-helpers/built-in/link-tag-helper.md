---
title: ASP.NET Core 中的链接标记帮助程序
author: rick-anderson
ms.author: riande
description: 了解 ASP.NET Core 链接标记帮助程序属性以及每个属性在扩展 HTML 链接标记的行为中所起的作用。
ms.custom: mvc
ms.date: 09/24/2019
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
uid: mvc/views/tag-helpers/builtin-th/link-tag-helper
ms.openlocfilehash: 7998cec94ebb56cbe6dbc321f7cb499260d6fe74
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059112"
---
# <a name="link-tag-helper-in-aspnet-core"></a>ASP.NET Core 中的链接标记帮助程序

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

[链接标记帮助程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper)用于生成指向主要或回退 CSS 文件的链接。 通常主 CSS 文件位于[内容分发网络](/office365/enterprise/content-delivery-networks#what-exactly-is-a-cdn) (CDN)。

[!INCLUDE[](~/includes/cdn.md)]

可以使用链接标记帮助程序指定 CSS 文件的 CDN 以及回退文件（CDN 不可用时）。 链接标记帮助程序借助本地宿主的可靠性提供 CDN 性能优势。

以下 Razor 标记显示 `head` 使用 ASP.NET Core web 应用程序模板创建的布局文件的元素：

[!code-cshtml[](link-tag-helper/sample/_Layout.cshtml?name=snippet)]

以下是上述代码呈现的 HTML（非开发环境）：

[!code-html[](link-tag-helper/sample/HtmlPage1.html)]

在上述代码中，链接标记帮助程序生成 `<meta name="x-stylesheet-fallback-test" content="" class="sr-only" />` 元素以及以下 JavaScript（用于验证是否可以从 CDN 获取请求的 bootstrap.min.css 文件）  。 在此示例中，可以获取 CSS 文件，因此标记帮助程序使用 CDN CSS 文件生成 `<link />` 元素。

## <a name="commonly-used-link-tag-helper-attributes"></a>常用的链接标记帮助程序属性

若要了解所有链接标记帮助程序属性和方法，请参阅[链接标记帮助程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper)。

### <a name="href"></a>href

链接的资源的首选地址。 在任何情况下，均会将此地址传递到生成的 HTML。

### <a name="asp-fallback-href"></a>asp-fallback-href

主 URL 失效后要回退到的 CSS 样式表的 URL。

### <a name="asp-fallback-test-class"></a>asp-fallback-test-class

样式表中定义的用于回退测试的类名称。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestClass>。

### <a name="asp-fallback-test-property"></a>asp-fallback-test-property

用于回退测试的 CSS 属性名称。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestProperty>。

### <a name="asp-fallback-test-value"></a>asp-fallback-test-value

用于回退测试的 CSS 属性值。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.LinkTagHelper.FallbackTestValue>。

## <a name="additional-resources"></a>其他资源

* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
