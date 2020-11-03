---
title: 教程：使用 JavaScript 调用 ASP.NET Core Web API
author: rick-anderson
description: 了解如何使用 JavaScript 调用 ASP.NET Core Web API。
ms.author: riande
ms.custom: mvc, devx-track-js
ms.date: 11/26/2019
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
uid: tutorials/web-api-javascript
ms.openlocfilehash: b41288bd63267a9aa7035e25ebc8d838eed5d93b
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690688"
---
# <a name="tutorial-call-an-aspnet-core-web-api-with-javascript"></a>教程：使用 JavaScript 调用 ASP.NET Core Web API

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

此教程介绍如何通过 [Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API) 使用 JavaScript 调用 ASP.NET Core Web API。

::: moniker range="< aspnetcore-3.0"

有关 ASP.NET Core 2.2，请参阅 2.2 版本的[使用 JavaScript 调用 Web API](xref:tutorials/first-web-api#call-the-web-api-with-javascript)。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

## <a name="prerequisites"></a>先决条件

* 完成[教程：创建 web API](xref:tutorials/first-web-api)
* 熟悉 CSS、HTML 和 JavaScript

## <a name="call-the-web-api-with-javascript"></a>使用 JavaScript 调用 Web API

在本部分中，将添加一个 HTML 页面，其中包含用于创建和管理待办事项的窗体。 事件处理程序会附加到页面上的元素。 事件处理程序导致对 Web API 的操作方法发出 HTTP 请求。 Fetch API 的 `fetch` 函数可启动每个 HTTP 请求。

`fetch` 函数可返回 [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise) 对象，该对象包含表示为 `Response` 对象的 HTTP 响应。 常见模式是通过调用 `Response` 对象上的 `json` 函数来提取 JSON 响应正文。 JavaScript 会使用 Web API 响应的详细信息来更新页面。

最简单的 `fetch` 调用接受一个表示路由的参数。 第二个参数（称为 `init` 对象）是可选的。 `init` 用于配置 HTTP 请求。

1. 配置应用[提供静态文件](/dotnet/api/microsoft.aspnetcore.builder.staticfileextensions.usestaticfiles#Microsoft_AspNetCore_Builder_StaticFileExtensions_UseStaticFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_)并[启用默认文件映射](/dotnet/api/microsoft.aspnetcore.builder.defaultfilesextensions.usedefaultfiles#Microsoft_AspNetCore_Builder_DefaultFilesExtensions_UseDefaultFiles_Microsoft_AspNetCore_Builder_IApplicationBuilder_)。 在 Startup.cs 的 `Configure` 方法中需要以下突出显示的代码  ：

    [!code-csharp[](first-web-api/samples/3.0/TodoApi/StartupJavaScript.cs?highlight=8-9&name=snippet_configure)]

1. 在项目根中创建 wwwroot 文件夹  。

1. 在 wwwroot 文件夹中创建一个 js 文件夹   。

1. 将一个名为 index.html 的 HTML 文件添加到 wwwroot 文件夹   。 将 index.html 的内容替换为以下标记  ：

    [!code-html[](first-web-api/samples/3.0/TodoApi/wwwroot/index.html)]

1. 将一个名为 site.js 的 JavaScript 文件添加到 wwwroot/js 文件夹   。 将 site.js 的内容替换为以下代码  ：

    [!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_SiteJs)]

可能需要更改 ASP.NET Core 项目的启动设置，以便对 HTML 页面进行本地测试：

1. 打开 Properties\launchSettings.json  。
1. 删除 `launchUrl` 以便在项目的默认文件 index.html 强制打开应用  。

此示例调用 Web API 的所有 CRUD 方法。 下面说明 Web API 请求。

### <a name="get-a-list-of-to-do-items"></a>获取待办事项的列表

在以下代码中，会将 HTTP GET 请求发送到 api/TodoItems 路由  ：

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_GetItems)]

当 Web API 返回成功状态的代码时，将调用 `_displayItems` 函数。 将 `_displayItems` 接受的数组参数中的每个待办事项添加到具有“编辑”和“删除”按钮的表中   。 如果 Web API 请求失败，则会将错误记录到浏览器的控制台中。

### <a name="add-a-to-do-item"></a>添加待办事项

在以下代码中：

* 声明 `item` 变量来构造待办事项的对象文字表示形式。
* 使用以下选项来配置提取请求：
  * `method`&mdash;指定 POST HTTP 操作谓词。
  * `body`&mdash;指定请求正文的 JSON 表示形式。 通过将存储在 `item` 中的对象文字传递到 [JSON.stringify](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify) 函数来生成 JSON。
  * `headers`&mdash;指定 `Accept` 和 `Content-Type` HTTP 请求标头。 将两个标头都设置为 `application/json`，以便分别指定接收和发送的媒体类型。
* 将 HTTP POST 请求发送到 api/TodoItems 路由  。

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_AddItem)]

当 Web API 返回成功状态的代码时，将调用 `getItems` 函数来更新 HTML 表。 如果 Web API 请求失败，则会将错误记录到浏览器的控制台中。

### <a name="update-a-to-do-item"></a>更新待办事项

更新待办事项类似于添加待办事项；但是，二者存在两个重大差异：

* 路由的后缀为待更新项的唯一标识符。 例如，api/TodoItems/1  。
* 正如 `method` 选项所示，HTTP 操作谓词是 PUT。

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_UpdateItem)]

### <a name="delete-a-to-do-item"></a>删除待办事项

若要删除待办事项，请将请求的 `method` 选项设置为 `DELETE` 并指定该项在 URL 中的唯一标识符。

[!code-javascript[](first-web-api/samples/3.0/TodoApi/wwwroot/js/site.js?name=snippet_DeleteItem)]

前进到下一个教程，了解如何生成 Web API 帮助页：

> [!div class="nextstepaction"]
> <xref:tutorials/get-started-with-swashbuckle>

::: moniker-end
