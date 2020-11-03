---
title: ASP.NET Core 的客户端 IP 安全安全
author: damienbod
description: 了解如何编写中间件或操作筛选器，以根据已批准的 IP 地址列表来验证远程 IP 地址。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 03/12/2020
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
uid: security/ip-safelist
ms.openlocfilehash: dfc134b97bb0976bc682a53d536cd27785550c7d
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059658"
---
# <a name="client-ip-safelist-for-aspnet-core"></a>ASP.NET Core 的客户端 IP 安全安全

作者： [Damien Bowden](https://twitter.com/damien_bod) 和 [Tom Dykstra](https://github.com/tdykstra)
 
本文介绍三种实现 IP 地址安全列表的方式 (在 ASP.NET Core 应用程序中也称为允许列表) 。 附带的示例应用程序演示了所有这三种方法。 可用工具如下：

* 用于检查每个请求的远程 IP 地址的中间件。
* MVC 操作筛选器，用于检查针对特定控制器或操作方法的请求的远程 IP 地址。
* Razor 页面筛选器，用于检查页面请求的远程 IP 地址 Razor 。

在每种情况下，包含批准的客户端 IP 地址的字符串存储在应用设置中。 中间件或筛选器：

* 将字符串分析为数组。 
* 检查阵列中是否存在远程 IP 地址。

如果数组包含 IP 地址，则允许访问。 否则，将返回 HTTP 403 禁止的状态代码。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/ip-safelist/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="ip-address-safelist"></a>IP 地址安全安全

在示例应用中，IP 地址安全项是：

* 由 `AdminSafeList` 文件中的属性定义 *appsettings.json* 。
* 分号分隔的字符串，可包含 [Internet 协议版本 4 (IPv4) ](https://wikipedia.org/wiki/IPv4) 和 [internet 协议版本 6 (IPv6) ](https://wikipedia.org/wiki/IPv6) 地址。

[!code-json[](ip-safelist/samples/3.x/ClientIpAspNetCore/appsettings.json?range=1-3&highlight=2)]

在前面的示例中，允许使用的 IPv4 地址和 `127.0.0.1` `192.168.1.5`) 的 `::1` (压缩格式的 IPv6 环回地址 `0:0:0:0:0:0:0:1` 。

## <a name="middleware"></a>中间件

`Startup.Configure`方法将自定义 `AdminSafeListMiddleware` 中间件类型添加到应用的请求管道。 使用 .NET Core 配置提供程序检索到该安全，并将其作为构造函数参数进行传递。

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureAddMiddleware)]

中间件将字符串分析为数组，并在数组中搜索远程 IP 地址。 如果找不到远程 IP 地址，中间件将返回 HTTP 403 禁止访问。 对于 HTTP GET 请求，将跳过此验证过程。

[!code-csharp[](ip-safelist/samples/Shared/ClientIpSafelistComponents/Middlewares/AdminSafeListMiddleware.cs?name=snippet_ClassOnly)]

## <a name="action-filter"></a>操作筛选器

如果需要针对特定 MVC 控制器或操作方法的安全安全访问控制，请使用操作筛选器。 例如： 。

[!code-csharp[](ip-safelist/samples/Shared/ClientIpSafelistComponents/Filters/ClientIpCheckActionFilter.cs?name=snippet_ClassOnly)]

在中 `Startup.ConfigureServices` ，将操作筛选器添加到 MVC 筛选器集合。 在下面的示例中， `ClientIpCheckActionFilter` 添加了一个操作筛选器。 安全日志和控制台记录器实例作为构造函数参数进行传递。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesActionFilter)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesActionFilter)]

::: moniker-end

然后，可以将操作筛选器应用到具有 [[ServiceFilter]](xref:Microsoft.AspNetCore.Mvc.ServiceFilterAttribute) 属性的控制器或操作方法：

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Controllers/ValuesController.cs?name=snippet_ActionFilter&highlight=1)]

在示例应用中，操作筛选器将应用于控制器的 `Get` 操作方法。 当你通过发送来测试应用程序时：

* HTTP GET 请求，该 `[ServiceFilter]` 属性验证客户端 IP 地址。 如果允许访问 `Get` 操作方法，则 "操作筛选器" 和 "操作" 方法将生成以下控制台输出的变体：

    ```
    dbug: ClientIpSafelistComponents.Filters.ClientIpCheckActionFilter[0]
          Remote IpAddress: ::1
    dbug: ClientIpAspNetCore.Controllers.ValuesController[0]
          successful HTTP GET    
    ```

* 除 GET 之外的 HTTP 请求谓词将 `AdminSafeListMiddleware` 验证客户端 IP 地址。

## <a name="no-locrazor-pages-filter"></a>Razor 页面筛选器

如果需要针对页面应用的安全筛选器驱动访问控制 Razor ，请使用 Razor 页面筛选器。 例如： 。

[!code-csharp[](ip-safelist/samples/Shared/ClientIpSafelistComponents/Filters/ClientIpCheckPageFilter.cs?name=snippet_ClassOnly)]

在中 `Startup.ConfigureServices` ， Razor 通过将页面筛选器添加到 MVC 筛选器集合来启用这些页面筛选器。 在下面的示例中，添加了一个 `ClientIpCheckPageFilter` Razor 页面筛选器。 安全日志和控制台记录器实例作为构造函数参数进行传递。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](ip-safelist/samples/3.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesPageFilter)]

::: moniker-end

::: moniker range="<= aspnetcore-2.2"

[!code-csharp[](ip-safelist/samples/2.x/ClientIpAspNetCore/Startup.cs?name=snippet_ConfigureServicesPageFilter)]

::: moniker-end

请求示例应用的 *索引* Razor 页时， Razor 页面筛选器将验证客户端 IP 地址。 筛选器生成以下控制台输出的变体：

```
dbug: ClientIpSafelistComponents.Filters.ClientIpCheckPageFilter[0]
      Remote IpAddress: ::1
```

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/middleware/index>
* [操作筛选器](xref:mvc/controllers/filters#action-filters)
* <xref:razor-pages/filter>
