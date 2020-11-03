---
title: 在 ASP.NET Core 中将依赖项注入到控制器
author: ardalis
description: 了解 ASP.NET Core MVC 控制器如何使用 ASP.NET Core 中的依赖项注入通过构造函数显式请求其依赖项。
ms.author: riande
ms.date: 02/24/2019
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
uid: mvc/controllers/dependency-injection
ms.openlocfilehash: 1282cd984584be423fba755e64e5d2f1afd2af89
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060607"
---
# <a name="dependency-injection-into-controllers-in-aspnet-core"></a>在 ASP.NET Core 中将依赖项注入到控制器

::: moniker range=">= aspnetcore-3.0"

作者：[Shadi Namrouti](https://github.com/shadinamrouti)、[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Steve Smith](https://github.com/ardalis)

ASP.NET Core MVC 控制器通过构造函数显式请求依赖关系。 ASP.NET Core 内置有对[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 的支持。 DI 使应用更易于测试和维护。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/dependency-injection/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="constructor-injection"></a>构造函数注入

服务作为构造函数参数添加，并且运行时从服务容器中解析服务。 通常使用接口来定义服务。 例如，考虑需要当前时间的应用。 以下接口公开 `IDateTime` 服务：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Interfaces/IDateTime.cs?name=snippet)]

以下代码实现 `IDateTime` 接口：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Services/SystemDateTime.cs?name=snippet)]

将服务添加到服务容器中：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Startup1.cs?name=snippet&highlight=3)]

有关 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*> 的详细信息，请参阅 [DI 服务生存期](xref:fundamentals/dependency-injection#service-lifetimes)。

以下代码根据一天中的时间向用户显示问候语：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Controllers/HomeController.cs?name=snippet)]

运行应用并且系统将根据时间显示一条消息。

## <a name="action-injection-with-fromservices"></a>FromServices 的操作注入

<xref:Microsoft.AspNetCore.Mvc.FromServicesAttribute> 允许将服务直接注入到操作方法，而无需使用构造函数注入：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Controllers/HomeController.cs?name=snippet2)]

## <a name="access-settings-from-a-controller"></a>从控制器访问设置

从控制器中访问应用或配置设置是一种常见模式。 <xref:fundamentals/configuration/options> 中所述的选项模式是管理设置的首选方法  。 通常情况下，不直接将 <xref:Microsoft.Extensions.Configuration.IConfiguration> 注入到控制器。

创建表示选项的类。 例如： 。

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Models/SampleWebSettings.cs?name=snippet)]

将配置类添加到服务集合中：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Startup.cs?highlight=4&name=snippet1)]

将应用配置为从 JSON 格式文件中读取设置：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Program.cs?name=snippet&range=10-15)]

以下代码从服务容器请求 `IOptions<SampleWebSettings>` 设置，并通过 `Index` 方法使用它们：

[!code-csharp[](dependency-injection/3.1sample/ControllerDI/Controllers/SettingsController.cs?name=snippet)]

## <a name="additional-resources"></a>其他资源

* 请参阅 <xref:mvc/controllers/testing>，了解如何显式请求控制器中的依赖关系，以更轻松地测试代码。

* [将默认依赖关系注入容器替换为第三方实现](xref:fundamentals/dependency-injection#default-service-container-replacement)。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Shadi Namrouti](https://github.com/shadinamrouti)、[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Steve Smith](https://github.com/ardalis)

ASP.NET Core MVC 控制器通过构造函数显式请求依赖关系。 ASP.NET Core 内置有对[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 的支持。 DI 使应用更易于测试和维护。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/dependency-injection/sample)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="constructor-injection"></a>构造函数注入

服务作为构造函数参数添加，并且运行时从服务容器中解析服务。 通常使用接口来定义服务。 例如，考虑需要当前时间的应用。 以下接口公开 `IDateTime` 服务：

[!code-csharp[](dependency-injection/sample/ControllerDI/Interfaces/IDateTime.cs?name=snippet)]

以下代码实现 `IDateTime` 接口：

[!code-csharp[](dependency-injection/sample/ControllerDI/Services/SystemDateTime.cs?name=snippet)]

将服务添加到服务容器中：

[!code-csharp[](dependency-injection/sample/ControllerDI/Startup1.cs?name=snippet&highlight=3)]

有关 <xref:Microsoft.Extensions.DependencyInjection.ServiceCollectionServiceExtensions.AddSingleton*> 的详细信息，请参阅 [DI 服务生存期](xref:fundamentals/dependency-injection#service-lifetimes)。

以下代码根据一天中的时间向用户显示问候语：

[!code-csharp[](dependency-injection/sample/ControllerDI/Controllers/HomeController.cs?name=snippet)]

运行应用并且系统将根据时间显示一条消息。

## <a name="action-injection-with-fromservices"></a>FromServices 的操作注入

<xref:Microsoft.AspNetCore.Mvc.FromServicesAttribute> 允许将服务直接注入到操作方法，而无需使用构造函数注入：

[!code-csharp[](dependency-injection/sample/ControllerDI/Controllers/HomeController.cs?name=snippet2)]

## <a name="access-settings-from-a-controller"></a>从控制器访问设置

从控制器中访问应用或配置设置是一种常见模式。 <xref:fundamentals/configuration/options> 中所述的选项模式是管理设置的首选方法  。 通常情况下，不直接将 <xref:Microsoft.Extensions.Configuration.IConfiguration> 注入到控制器。

创建表示选项的类。 例如： 。

[!code-csharp[](dependency-injection/sample/ControllerDI/Models/SampleWebSettings.cs?name=snippet)]

将配置类添加到服务集合中：

[!code-csharp[](dependency-injection/sample/ControllerDI/Startup.cs?highlight=4&name=snippet1)]

将应用配置为从 JSON 格式文件中读取设置：

[!code-csharp[](dependency-injection/sample/ControllerDI/Program.cs?name=snippet&range=10-15)]

以下代码从服务容器请求 `IOptions<SampleWebSettings>` 设置，并通过 `Index` 方法使用它们：

[!code-csharp[](dependency-injection/sample/ControllerDI/Controllers/SettingsController.cs?name=snippet)]

## <a name="additional-resources"></a>其他资源

* 请参阅 <xref:mvc/controllers/testing>，了解如何显式请求控制器中的依赖关系，以更轻松地测试代码。

* [将默认依赖关系注入容器替换为第三方实现](xref:fundamentals/dependency-injection#default-service-container-replacement)。

::: moniker-end