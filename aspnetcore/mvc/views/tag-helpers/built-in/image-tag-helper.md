---
title: ASP.NET Core 中的图像标记帮助程序
author: pkellner
description: 演示如何使用图像标记帮助程序。
ms.author: riande
ms.custom: mvc
ms.date: 04/06/2019
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
uid: mvc/views/tag-helpers/builtin-th/image-tag-helper
ms.openlocfilehash: edc6971ac42756504b5b371ba509d7a657a0c396
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061114"
---
# <a name="image-tag-helper-in-aspnet-core"></a>ASP.NET Core 中的图像标记帮助程序

作者：[Peter Kellner](https://peterkellner.net)

图像标记帮助程序增强了 `<img>` 标记，为静态图像文件提供缓存破坏行为。

缓存破坏字符串是一个唯一值，表示附加到资产 URL 的静态图像文件的哈希值。 唯一字符串会提示客户端（和某些代理）从主机 Web 服务器重新加载图像，而不是从客户端的缓存重新加载。

如果图像源 (`src`) 是主机 web 服务器上的静态文件：

* 将唯一缓存破坏字符串作为图像源的查询参数进行追加。
* 如果主机 Web 服务器上的文件发生更改，将生成包含已更新请求参数的唯一请求 URL。

有关标记帮助程序的概述，请参阅 <xref:mvc/views/tag-helpers/intro>。

## <a name="image-tag-helper-attributes"></a>图像标记帮助程序属性

### <a name="src"></a>src

若要激活图像标记帮助程序，`<img>` 元素需要有 `src` 属性。

图像源 (`src`) 必须指向服务器上的物理静态文件。 如果 `src` 是一个远程 URI，则不会生成缓存破坏查询字符串参数。

### <a name="asp-append-version"></a>asp-append-version

如果使用 `true` 值和 `src` 属性指定 `asp-append-version`，则会调用图像标记帮助程序。

下面的示例使用图像标记帮助程序：

```cshtml
<img src="~/images/asplogo.png" asp-append-version="true">
```

如果目录 /wwwroot/images/  中存在静态文件，则生成的 html 与下面类似（哈希有所不同）：

```html
<img src="/images/asplogo.png?v=Kl_dqr9NVtnMdsM2MUg4qthUnWZm5T1fCEimBPWDNgM">
```

分配给参数 `v` 的值是磁盘上的 asplogo.png 文件的哈希值。  如果 Web 服务器无法获取对静态文件的读取访问权限，则不会向呈现在标记中的 `src` 属性添加 `v` 参数。

## <a name="hash-caching-behavior"></a>哈希缓存行为

图像标记帮助程序使用本地 Web 服务器上的缓存提供程序来存储给定文件的已计算 `Sha512` 哈希。 如果多次请求文件，则不重新计算哈希值。 当计算该文件的 `Sha512` 哈希时，附加到该文件的文件观察程序会让 Cache 失效。 当磁盘上的的文件发生更改时，将会计算和缓存新的哈希。

## <a name="additional-resources"></a>其他资源

* <xref:performance/caching/memory>
