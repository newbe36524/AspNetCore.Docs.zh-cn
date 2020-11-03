---
title: ASP.NET Core 中的脚本标记帮助程序
author: rick-anderson
ms.author: riande
description: 了解 ASP.NET Core 脚本标记帮助程序属性以及每个属性在扩展 HTML 脚本标记的行为中所起的作用。
ms.custom: mvc
ms.date: 12/02/2019
no-loc:
- appsettings.json
- cookie
- Cookie
- Blazor
- Blazor Server
- Blazor WebAssembly
- Identity
- Let's Encrypt
- Razor
- SignalR
uid: mvc/views/tag-helpers/builtin-th/script-tag-helper
ms.openlocfilehash: f5856bf19681a42551f82bb15c769f192f338b4a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053496"
---
# <a name="script-tag-helper-in-aspnet-core"></a>ASP.NET Core 中的脚本标记帮助程序

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

[标记帮助程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper)用于生成指向主要或回退脚本文件的链接。 通常主脚本文件位于[内容分发网络](/office365/enterprise/content-delivery-networks#what-exactly-is-a-cdn) (CDN)。

[!INCLUDE[](~/includes/cdn.md)]

可以使用脚本标记帮助程序指定脚本文件的 CDN 以及回退文件（CDN 不可用时）。 脚本标记帮助程序借助本地宿主的可靠性提供 CDN 性能优势。

以下 Razor 标记显示了 `script` 具有回退的元素：

```html
<script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-3.3.1.min.js"
        asp-fallback-src="~/lib/jquery/dist/jquery.min.js"
        asp-fallback-test="window.jQuery"
        crossorigin="anonymous"
        integrity="sha384-tsQFqpEReu7ZLhBV2VZlAu7zcOV+rXbYlF2cqB8txI/8aZajjp4Bqd+V6D5IgvKT">
</script>
```

请勿使用 `<script>` 元素的 [](https://developer.mozilla.org/docs/Web/HTML/Element/script) 属性来延迟加载 CDN 脚本。 脚本标记帮助程序呈现能够立即执行 [asp-fallback-test](#asp-fallback-test) 表达式的 JavaScript。 如果延迟加载 CDN 脚本，则该表达式失败。

## <a name="commonly-used-script-tag-helper-attributes"></a>常用的脚本标记帮助程序属性

若要了解所有脚本标记帮助程序属性和方法，请参阅[标记帮助程序](xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper)。

### <a name="asp-fallback-test"></a>asp-fallback-test

主脚本中定义的用于回退测试的脚本方法。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper.FallbackTestExpression>。

### <a name="asp-fallback-src"></a>asp-fallback-src

主 URL 失效后要回退到的脚本标签的 URL。 有关详细信息，请参阅 <xref:Microsoft.AspNetCore.Mvc.TagHelpers.ScriptTagHelper.FallbackSrc>。

## <a name="additional-resources"></a>其他资源

* <xref:mvc/views/tag-helpers/intro>
* <xref:mvc/controllers/areas>
* <xref:razor-pages/index>
* <xref:mvc/compatibility-version>
