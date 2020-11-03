---
title: ASP.NET Core Blazor 托管模型配置
author: guardrex
description: 了解有关 ASP.NET Core Blazor 托管模型配置的其他方案。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/12/2020
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
uid: blazor/fundamentals/additional-scenarios
ms.openlocfilehash: 075bcc68fd2dff0ebf2cfceacec24fde8c818603
ms.sourcegitcommit: b5ebaf42422205d212e3dade93fcefcf7f16db39
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/21/2020
ms.locfileid: "92326542"
---
# <a name="aspnet-core-no-locblazor-hosting-model-configuration"></a>ASP.NET Core Blazor 托管模型配置

作者：[Daniel Roth](https://github.com/danroth27)、[Mackinnon Buck](https://github.com/MackinnonBuck) 和 [Luke Latham](https://github.com/guardrex)

本文介绍了托管模型配置。

### <a name="no-locsignalr-cross-origin-negotiation-for-authentication"></a>用于身份验证的 SignalR 跨源协商

本部分适用于 Blazor WebAssembly。

若要将 SignalR 的基础客户端配置为发送凭据（如 cookie 或 HTTP 身份验证标头），请执行以下操作：

* 使用 <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.WebAssemblyHttpRequestMessageExtensions.SetBrowserRequestCredentials%2A> 在跨源 [`fetch`](https://developer.mozilla.org/docs/Web/API/Fetch_API/Using_Fetch) 提取请求中设置 <xref:Microsoft.AspNetCore.Components.WebAssembly.Http.BrowserRequestCredentials.Include>：

  ```csharp
  public class IncludeRequestCredentialsMessageHandler : DelegatingHandler
  {
      protected override Task<HttpResponseMessage> SendAsync(
          HttpRequestMessage request, CancellationToken cancellationToken)
      {
          request.SetBrowserRequestCredentials(BrowserRequestCredentials.Include);
          return base.SendAsync(request, cancellationToken);
      }
  }
  ```

* 将 <xref:System.Net.Http.HttpMessageHandler> 分配给 <xref:Microsoft.AspNetCore.Http.Connections.Client.HttpConnectionOptions.HttpMessageHandlerFactory> 选项：

  ```csharp
  var connection = new HubConnectionBuilder()
      .WithUrl(new Uri("http://signalr.example.com"), options =>
      {
          options.HttpMessageHandlerFactory = innerHandler => 
              new IncludeRequestCredentialsMessageHandler { InnerHandler = innerHandler };
      }).Build();
  ```

有关详细信息，请参阅 <xref:signalr/configuration#configure-additional-options>。

## <a name="reflect-the-connection-state-in-the-ui"></a>反映 UI 中的连接状态

本部分适用于 Blazor Server。

如果客户端检测到连接已丢失，在客户端尝试重新连接时会向用户显示默认 UI。 如果重新连接失败，则会向用户提供重试选项。

若要自定义 UI，请在 `_Host.cshtml` Razor 页面的 `<body>` 中定义一个 `id` 为 `components-reconnect-modal` 的元素：

```cshtml
<div id="components-reconnect-modal">
    ...
</div>
```

向应用的样式表（`wwwroot/css/app.css` 或 `wwwroot/css/site.css`）添加以下内容：

```css
#components-reconnect-modal {
    display: none;
}

#components-reconnect-modal.components-reconnect-show {
    display: block;
}
```

下表介绍应用于 `components-reconnect-modal` 元素的 CSS 类。

| CSS 类                       | 指示&hellip; |
| ------------------------------- | ----------------- |
| `components-reconnect-show`     | 连接已丢失。 客户端正在尝试重新连接。 显示模式。 |
| `components-reconnect-hide`     | 将为服务器重新建立活动连接。 隐藏模式。 |
| `components-reconnect-failed`   | 重新连接失败，可能是由于网络故障引起的。 若要尝试重新连接，请调用 `window.Blazor.reconnect()`。 |
| `components-reconnect-rejected` | 已拒绝重新连接。 已达到服务器，但拒绝连接，服务器上的用户状态丢失。 若要重载应用，请调用 `location.reload()`。 当出现以下情况时，可能会导致此连接状态：<ul><li>服务器端线路发生故障。</li><li>客户端断开连接的时间足以使服务器删除用户的状态。 已释放用户正在与之交互的组件的实例。</li><li>服务器已重启，或者应用的工作进程被回收。</li></ul> |

## <a name="render-mode"></a>呈现模式

本部分适用于 Blazor Server。

默认情况下，Blazor Server 应用设置为：在客户端与服务器建立连接之前在服务器上预呈现 UI。 这是在 `_Host.cshtml` Razor 页中设置的：

```cshtml
<body>
    <app>
      <component type="typeof(App)" render-mode="ServerPrerendered" />
    </app>

    <script src="_framework/blazor.server.js"></script>
</body>
```

<xref:Microsoft.AspNetCore.Mvc.TagHelpers.ComponentTagHelper.RenderMode> 配置组件是否：

* 在页面中预呈现。
* 在页面上呈现为静态 HTML，或者包含从用户代理启动 Blazor 应用所需的信息。

| 呈现模式 | 描述 |
| --- | --- |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.ServerPrerendered> | 在静态 HTML 中呈现组件，并包含 Blazor Server 应用的标记。 用户代理启动时，此标记用于启动 Blazor 应用。 |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Server> | 呈现 Blazor Server 应用的标记。 不包括组件的输出。 用户代理启动时，此标记用于启动 Blazor 应用。 |
| <xref:Microsoft.AspNetCore.Mvc.Rendering.RenderMode.Static> | 将组件呈现为静态 HTML。 |

不支持从静态 HTML 页面呈现服务器组件。

## <a name="initialize-the-no-locblazor-circuit"></a>初始化 Blazor 回路

本部分适用于 Blazor Server。

在 `Pages/_Host.cshtml` 文件中配置 Blazor Server 应用 [SignalR 回路 ](xref:blazor/hosting-models#circuits) 的手动启动：

* 将 `autostart="false"` 属性添加到 `blazor.server.js` 脚本的 `<script>` 标记中。
* 将调用 `Blazor.start` 的脚本放置在 `blazor.server.js` 脚本标记之后并放在结束的 `</body>` 标记内。

禁用 `autostart` 时，应用中不依赖该回路的任何方面都能正常工作。 例如，客户端路由正常运行。 但是，在调用 `Blazor.start` 之前，依赖于该回路的任何方面不会正常运行。 如果没有已建立的回路，应用行为是不可预测的。 例如，在回路断开连接时，组件方法无法执行。

### <a name="initialize-no-locblazor-when-the-document-is-ready"></a>文档准备就绪时初始化 Blazor

文档准备就绪时初始化 Blazor 应用：

```cshtml
<body>

    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      document.addEventListener("DOMContentLoaded", function() {
        Blazor.start();
      });
    </script>
</body>
```

### <a name="chain-to-the-promise-that-results-from-a-manual-start"></a>链接到由手动启动生成的 `Promise`

若要执行其他任务（如 JS 互操作初始化），请使用 `then` 链接到 `Promise`（由手动 Blazor 应用启动生成）：

```cshtml
<body>

    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start().then(function () {
        ...
      });
    </script>
</body>
```

### <a name="configure-the-no-locsignalr-client"></a>配置 SignalR 客户端

#### <a name="logging"></a>日志记录

若要配置 SignalR 客户端日志，请传入调用 `configureLogging` 的配置对象 (`configureSignalR`)，此对象在客户端生成器上具有日志级别：

```cshtml
<body>

    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        configureSignalR: function (builder) {
          builder.configureLogging("information");
        }
      });
    </script>
</body>
```

在前面的示例中，`information` 相当于日志级别 <xref:Microsoft.Extensions.Logging.LogLevel.Information?displayProperty=nameWithType>。

### <a name="modify-the-reconnection-handler"></a>修改重新连接处理程序

可以针对自定义行为修改重新连接处理程序的线路连接事件，如：

* 在连接断开时通知用户。
* 在线路连接时（通过客户端）执行日志记录。

若要修改连接事件，请为以下连接更改注册回调：

* 使用 `onConnectionDown` 删除的连接。
* 已建立/重新建立的连接使用 `onConnectionUp`。

必须同时指定 `onConnectionDown` 和 `onConnectionUp`：

```cshtml
<body>

    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        reconnectionHandler: {
          onConnectionDown: (options, error) => console.error(error);
          onConnectionUp: () => console.log("Up, up, and away!");
        }
      });
    </script>
</body>
```

### <a name="adjust-the-reconnection-retry-count-and-interval"></a>调整重新连接重试计数和间隔

若要调整重新连接重试次数和间隔，请设置重试次数 (`maxRetries`) 和允许每次重试运行的毫秒数 (`retryIntervalMilliseconds`)：

```cshtml
<body>

    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      Blazor.start({
        reconnectionOptions: {
          maxRetries: 3,
          retryIntervalMilliseconds: 2000
        }
      });
    </script>
</body>
```

## <a name="hide-or-replace-the-reconnection-display"></a>隐藏或替换重新连接显示

若要隐藏重新连接显示，请将重新连接处理程序的 `_reconnectionDisplay` 设置为空对象（`{}` 或 `new Object()`）：

```cshtml
<body>

    ...

    <script autostart="false" src="_framework/blazor.server.js"></script>
    <script>
      window.addEventListener('beforeunload', function () {
        Blazor.defaultReconnectionHandler._reconnectionDisplay = {};
      });

      Blazor.start();
    </script>
</body>
```

若要替换重新连接显示，请将前面示例中的 `_reconnectionDisplay` 设置为要显示的元素：

```javascript
Blazor.defaultReconnectionHandler._reconnectionDisplay = 
  document.getElementById("{ELEMENT ID}");
```

占位符 `{ELEMENT ID}` 是要显示的 HTML 元素的 ID。

::: moniker range=">= aspnetcore-5.0"

通过在应用的 CSS (`wwwroot/css/site.css`) 中为模式元素设置 `transition-delay` 属性，自定义重新连接显示出现之前的延迟。 以下示例将转换延迟从 500 毫秒（默认值）设置为 1000 毫秒（1 秒）：

```css
#components-reconnect-modal {
    transition: visibility 0s linear 1000ms;
}
```

## <a name="disconnect-the-no-locblazor-circuit-from-the-client"></a>从客户端断开 Blazor 线路连接

默认情况下，触发 [`unload` 页面事件](https://developer.mozilla.org/docs/Web/API/Window/unload_event)时，Blazor 线路会断开连接。 若要断开客户端上其他方案的线路连接，请在相应的事件处理程序中调用 `Blazor.disconnect`。 在下面的示例中，当页面隐藏（[`pagehide` 事件](https://developer.mozilla.org/docs/Web/API/Window/pagehide_event)）时，线路会断开连接：

```javascript
window.addEventListener('pagehide', () => {
  Blazor.disconnect();
});
```

## <a name="influence-html-head-tag-elements"></a>影响 HTML `<head>` 标记元素

本部分适用于 Blazor WebAssembly 和 Blazor Server 即将发布的 ASP.NET Core 5.0 版本。

`Title`、`Link` 和 `Meta` 组件呈现时，会在 HTML `<head>` 标记元素中添加或更新数据：

```razor
@using Microsoft.AspNetCore.Components.Web.Extensions.Head

<Title Value="{TITLE}" />
<Link href="{URL}" rel="stylesheet" />
<Meta content="{DESCRIPTION}" name="description" />
```

在前面的示例中，`{TITLE}`、`{URL}` 和 `{DESCRIPTION}` 的占位符是字符串值、Razor 变量或 Razor 表达式。

具有以下特性：

* 支持服务器端预呈现。
* `Value` 参数是 `Title` 组件唯一有效的参数。
* 为 `Meta` 和 `Link` 组件提供的 HTML 属性是在[其他属性](xref:blazor/components/index#attribute-splatting-and-arbitrary-parameters)中捕获的，并传递到呈现的 HTML 标记。
* 对于多个 `Title` 组件，页的标题反映了呈现的最后一个 `Title` 组件的 `Value`。
* 如果多个 `Meta` 或 `Link` 组件包含在相同的属性中，则每个 `Meta` 或 `Link` 组件呈现且仅呈现一个 HTML 标记。 两个 `Meta` 或 `Link` 组件不能引用同一个呈现的 HTML 标记。
* 对现有 `Meta` 或 `Link` 组件的参数所做的更改将在其呈现的 HTML 标记中体现。
* 如果不再呈现 `Link` 或 `Meta` 组件，并因此由框架释放，则会删除其呈现的 HTML 标记。

在子组件中使用其中一个框架组件时，只要呈现包含框架组件的子组件，呈现的 HTML 标记就会影响父组件的任何其他子组件。 在子组件中使用其中一个框架组件，与在 `wwwroot/index.html` 或 `Pages/_Host.cshtml` 中放置一个 HTML 标记之间的区别是，框架组件已呈现的 HTML 标记：

* 可以根据应用程序状态进行修改。 不能根据应用程序状态修改硬编码 HTML 标记。
* 将在不再呈现父组件的情况下从 HTML `<head>` 中被删除。

::: moniker-end

## <a name="static-files"></a>静态文件

本部分适用于 Blazor Server。

若要使用 <xref:Microsoft.AspNetCore.StaticFiles.FileExtensionContentTypeProvider> 创建其他文件映射，或者要配置其他 <xref:Microsoft.AspNetCore.Builder.StaticFileOptions>，请使用以下方法之一。 在以下示例中，`{EXTENSION}` 占位符为文件扩展名，`{CONTENT TYPE}` 占位符为内容类型。

* 使用 <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> 通过 `Startup.ConfigureServices` (`Startup.cs`) 中的[依赖项注入 (DI)](xref:blazor/fundamentals/dependency-injection) 来配置选项：

  ```csharp
  using Microsoft.AspNetCore.StaticFiles;

  ...

  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  services.Configure<StaticFileOptions>(options =>
  {
      options.ContentTypeProvider = provider;
  });
  ```

  此方法会配置用于为 `blazor.server.js`提供服务的相同文件提供程序，因此请确保你的自定义配置不会干扰为 `blazor.server.js` 提供服务。 例如，不要通过使用 `provider.Mappings.Remove(".js")` 配置提供程序来删除 JavaScript 文件的映射。

* 在 `Startup.Configure` (`Startup.cs`) 中使用两次对 <xref:Microsoft.AspNetCore.Builder.StaticFileExtensions.UseStaticFiles%2A> 的调用：
  * 使用 <xref:Microsoft.AspNetCore.Builder.StaticFileOptions> 在第一次调用中配置自定义文件提供程序。
  * 第二个中间件提供 `blazor.server.js`，其使用 Blazor 框架提供的默认静态文件配置。

  ```csharp
  using Microsoft.AspNetCore.StaticFiles;

  ...

  var provider = new FileExtensionContentTypeProvider();
  provider.Mappings["{EXTENSION}"] = "{CONTENT TYPE}";

  app.UseStaticFiles(new StaticFileOptions { ContentTypeProvider = provider });
  app.UseStaticFiles();
  ```

* 可以使用 <xref:Microsoft.AspNetCore.Builder.MapWhenExtensions.MapWhen%2A> 执行自定义静态文件中间件来避免在提供 `_framework/blazor.server.js` 时受到干扰：

  ```csharp
  app.MapWhen(ctx => !ctx.Request.Path
      .StartsWithSegments("_framework/blazor.server.js", 
          subApp => subApp.UseStaticFiles(new StaticFileOptions(){ ... })));
  ```

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/logging/index>
