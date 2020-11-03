---
title: ASP.NET Core Blazor 全球化和本地化
author: guardrex
description: 了解如何使 Razor 组件能够供位于不同区域、使用不同语言的用户使用。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 06/04/2020
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
uid: blazor/globalization-localization
ms.openlocfilehash: 4345dd8525c2e72aaddc8e45a4fd4d9bfdd63040
ms.sourcegitcommit: b5ebaf42422205d212e3dade93fcefcf7f16db39
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/21/2020
ms.locfileid: "92326519"
---
# <a name="aspnet-core-no-locblazor-globalization-and-localization"></a>ASP.NET Core Blazor 全球化和本地化

作者：[Luke Latham](https://github.com/guardrex) 和 [Daniel Roth](https://github.com/danroth27)

Razor 组件可供位于不同区域、使用不同语言的用户使用。 以下 .NET 全球化和本地化方案可用：

* .NET 资源系统
* 特定于区域性的数字和日期格式

当前支持有限的 ASP.NET Core 本地化方案：

* Blazor 应用中支持 <xref:Microsoft.Extensions.Localization.IStringLocalizer> 和 <xref:Microsoft.Extensions.Localization.IStringLocalizer%601>。
* <xref:Microsoft.AspNetCore.Mvc.Localization.IHtmlLocalizer>、<xref:Microsoft.AspNetCore.Mvc.Localization.IViewLocalizer> 和数据注释本地化是 ASP.NET Core MVC 方案，在 Blazor 应用中不受支持。

有关详细信息，请参阅 <xref:fundamentals/localization>。

## <a name="globalization"></a>全球化

Blazor 的 [`@bind`](xref:mvc/views/razor#bind) 功能基于用户的当前区域性执行格式并分析值以进行显示。

可从 <xref:System.Globalization.CultureInfo.CurrentCulture?displayProperty=fullName> 属性访问当前区域性。

<xref:System.Globalization.CultureInfo.InvariantCulture?displayProperty=nameWithType> 用于以下字段类型（`<input type="{TYPE}" />`）：

* `date`
* `number`

上述字段类型：

* 使用其基于浏览器的相应格式规则进行显示。
* 不能包含自由格式的文本。
* 基于浏览器的实现提供用户交互特性。

以下字段类型具有特定的格式要求且当前不受 Blazor 支持，因为所有主流浏览器均不支持它们：

* `datetime-local`
* `month`
* `week`

[`@bind`](xref:mvc/views/razor#bind) 支持 `@bind:culture` 参数，以提供用于分析值并设置值格式的 <xref:System.Globalization.CultureInfo?displayProperty=fullName>。 使用 `date` 和 `number` 字段类型时，不建议指定区域性。 `date` 和 `number` 具有可提供所需区域性的内置 Blazor 支持。

## <a name="localization"></a>本地化

### Blazor WebAssembly

Blazor WebAssembly 应用使用用户的[语言首选项](https://developer.mozilla.org/docs/Web/API/NavigatorLanguage/languages)设置区域性。

若要显式配置区域性，请在 `Program.Main` 中设置 <xref:System.Globalization.CultureInfo.DefaultThreadCurrentCulture?displayProperty=nameWithType> 和 <xref:System.Globalization.CultureInfo.DefaultThreadCurrentUICulture?displayProperty=nameWithType>。

::: moniker range=">= aspnetcore-5.0"

默认情况下，Blazor WebAssembly 携带在用户区域性中显示值（如日期和货币）所需的最小全球化资源。 必须支持动态更改区域性的应用程序应在项目文件中配置 `BlazorWebAssemblyLoadAllGlobalizationData`：

```xml
<PropertyGroup>
  <BlazorWebAssemblyLoadAllGlobalizationData>true</BlazorWebAssemblyLoadAllGlobalizationData>
</PropertyGroup>
```

还可以通过传递给 `Blazor.start` 的选项将 Blazor WebAssembly 配置为使用特定应用程序区域性启动。 例如，下面的示例显示配置为使用 `en-GB` 区域性启动的应用：

```html
<script src="_framework/blazor.webassembly.js" autostart="false"></script>
<script>
  Blazor.start({
    applicationCulture: 'en-GB'
  });
</script>
```

`applicationCulture` 的值应符合 [BCP-47 语言标记格式](https://tools.ietf.org/html/bcp47)。

如果应用不需要本地化，你可以将应用配置为支持不变区域性，这基于 `en-US` 区域性：

```xml
<PropertyGroup>
  <InvariantGlobalization>true</InvariantGlobalization>
</PropertyGroup>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

默认情况下，Blazor WebAssembly 应用的中间语言 (IL) 链接器配置去除国际化信息（显式请求的区域设置除外）。 有关详细信息，请参阅 <xref:blazor/host-and-deploy/configure-linker#configure-the-linker-for-internationalization>。

::: moniker-end

虽然 Blazor 默认选择的区域性可能足以满足大多数用户的需求，但请考虑为用户提供一种指定其首选区域设置的方法。 如需获取具有区域性选取器的 Blazor WebAssembly 示例应用，请参阅 [`LocSample`](https://github.com/pranavkm/LocSample) 本地化示例应用。

### Blazor Server

Blazor Server 应用使用[本地化中间件](xref:fundamentals/localization#localization-middleware)进行本地化。 中间件为从应用请求资源的用户选择相应的区域性。

可使用以下方法之一设置区域性：

* [Cookie](#cookies)
* [提供用于选择区域性的 UI](#provide-ui-to-choose-the-culture)

有关更多信息和示例，请参见<xref:fundamentals/localization>。

#### <a name="no-loccookies"></a>Cookies

本地化区域性 cookie 可以保留用户的区域性。 本地化中间件会在后续请求上读取 cookie，以设置用户的区域性。 

使用 cookie 可确保 WebSocket 连接可以正确地传播区域性。 如果本地化方案基于 URL 路径或查询字符串，则该方案可能无法与 Websocket 协同使用，因而无法保留区域性。 因此，建议的方式是使用本地化区域性 cookie。

如果在本地化 cookie 中保留了区域性，则可以使用任意方法来分配区域性。 如果该应用已经为服务器端 ASP.NET Core 建立了本地化方案，请继续使用应用的现有本地化基础结构，并在应用方案中设置本地化区域性 cookie。

下面的示例演示如何在可由本地化中间件读取的 cookie 中设置当前区域性。 在 `Pages/_Host.cshtml` 文件中的开始 `<body>` 标记内立即创建一个 Razor 表达式：

```cshtml
@using System.Globalization
@using Microsoft.AspNetCore.Localization

...

<body>
    @{
        this.HttpContext.Response.Cookies.Append(
            CookieRequestCultureProvider.DefaultCookieName,
            CookieRequestCultureProvider.MakeCookieValue(
                new RequestCulture(
                    CultureInfo.CurrentCulture,
                    CultureInfo.CurrentUICulture)));
    }

    ...
</body>
```

本地化由应用按以下事件顺序进行处理：

1. 浏览器向应用发送初始 HTTP 请求。
1. 本地化中间件分配区域性。
1. `_Host` 页面 (`_Host.cshtml`) 中的 Razor 表达式将区域性作为响应的一部分保留在 cookie 中。
1. 浏览器打开 WebSocket 连接以创建交互式 Blazor Server 会话。
1. 本地化中间件读取 cookie 并分配区域性。
1. Blazor Server 会话以正确的区域性开始。

#### <a name="provide-ui-to-choose-the-culture"></a>提供用于选择区域性的 UI

若要提供支持用户选择区域性的 UI，建议使用基于重定向的方法。 此过程与用户尝试访问安全资源时 Web 应用中发生的情况类似。 用户会被重定向到登录页面，然后再重定向到原始资源。 

应用通过重定向到控制器来保留用户的所选区域性。 控制器将用户选择的区域性设置为 cookie，然后将用户重定向回原始 URI。

在服务器上创建一个 HTTP 终结点，以在 cookie 中设置用户的所选区域性，并执行重定向回原始 URI：

```csharp
[Route("[controller]/[action]")]
public class CultureController : Controller
{
    public IActionResult SetCulture(string culture, string redirectUri)
    {
        if (culture != null)
        {
            HttpContext.Response.Cookies.Append(
                CookieRequestCultureProvider.DefaultCookieName,
                CookieRequestCultureProvider.MakeCookieValue(
                    new RequestCulture(culture)));
        }

        return LocalRedirect(redirectUri);
    }
}
```

> [!WARNING]
> 使用 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.LocalRedirect%2A> 操作结果以阻止开放式重定向攻击。 有关详细信息，请参阅 <xref:security/preventing-open-redirects>。

如果应用未配置为处理控制器操作：

* 将 MVC 服务添加到 `Startup.ConfigureServices` 中的服务集合：

  ```csharp
  services.AddControllers();
  ```

* 在 `Startup.Configure` 中添加控制器终结点路由：

  ```csharp
  app.UseEndpoints(endpoints =>
  {
      endpoints.MapControllers();
      endpoints.MapBlazorHub();
      endpoints.MapFallbackToPage("/_Host");
  });
  ```

以下组件显示了一个示例，说明如何在用户选择区域性时执行初始重定向：

```razor
@inject NavigationManager NavigationManager

<h3>Select your language</h3>

<select @onchange="OnSelected">
    <option>Select...</option>
    <option value="en-US">English</option>
    <option value="fr-FR">Français</option>
</select>

@code {
    private void OnSelected(ChangeEventArgs e)
    {
        var culture = (string)e.Value;
        var uri = new Uri(NavigationManager.Uri)
            .GetComponents(UriComponents.PathAndQuery, UriFormat.Unescaped);
        var query = $"?culture={Uri.EscapeDataString(culture)}&" +
            $"redirectUri={Uri.EscapeDataString(uri)}";

        NavigationManager.NavigateTo("/Culture/SetCulture" + query, forceLoad: true);
    }
}
```

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/localization>
