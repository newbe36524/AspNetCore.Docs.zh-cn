---
title: 从 ASP.NET 迁移到 ASP.NET Core 2.0
author: isaac2004
description: 接收有关将现有 ASP.NET MVC 或 Web API 应用程序迁移到 ASP.NET Core 2.0 的指导。
ms.author: scaddie
ms.custom: mvc
ms.date: 10/24/2018
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
uid: migration/mvc2
ms.openlocfilehash: cf7d2e3a94c14fb752180d9349536d17b4557e0a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051325"
---
# <a name="migrate-from-aspnet-to-aspnet-core-20"></a>从 ASP.NET 迁移到 ASP.NET Core 2.0

作者：[Isaac Levin](https://isaaclevin.com)

本文可作为从 ASP.NET 应用程序迁移到 ASP.NET Core 2.0 的参考指南。

## <a name="prerequisites"></a>先决条件

从 .Net 下载安装以下内容 **之一** [： Windows](https://dotnet.microsoft.com/download)：

* .NET Core SDK
* 适用于 Windows 的 Visual Studio
  * ASP.NET 和 Web 开发工作负荷
  * .NET Core 跨平台开发工作负荷

## <a name="target-frameworks"></a>目标框架

ASP.NET Core 2.0 项目为开发人员提供了面向 .NET Core、.NET Framework 或同时面向这两者的灵活性。 若要确定最合适的目标框架，请参阅[为服务器应用选择 .NET Core 或 .NET Framework](/dotnet/standard/choosing-core-framework-server)。

面向 .NET Framework 时，项目需要引用单个 NuGet 包。

得益于有 ASP.NET Core 2.0 [元包](xref:fundamentals/metapackage)，面向 .NET Core 时可以避免进行大量的显式包引用。 在项目中安装 `Microsoft.AspNetCore.All` 元包：

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.9" />
</ItemGroup>
```

使用此元包时，应用不会部署元包中引用的任何包。 .NET Core 运行时存储中包含这些资产，并已预编译，旨在提升性能。 <xref:fundamentals/metapackage>有关更多详细信息，请参阅。

## <a name="project-structure-differences"></a>项目结构差异

ASP.NET Core 中简化了 .csproj 文件格式。 下面是一些显著的更改：

* 无需显式添加，即可将文件视作项目的一部分。 服务于大型团队时，这可减少出现 XML 合并冲突的风险。
* 没有对其他项目的基于 GUID 的引用，这可以提高文件的可读性。
* 无需在 Visual Studio 中卸载文件即可对它进行编辑：

  ![Visual Studio 2017 中的“编辑 CSPROJ”上下文菜单选项](_static/EditProjectVs2017.png)

## <a name="globalasax-file-replacement"></a>Global.asax 文件替换

ASP.NET Core 引入了启动应用的新机制。 ASP.NET 应用程序的入口点是 Global.asax 文件。 路由配置及筛选器和区域注册等任务在 Global.asax 文件中进行处理。

[!code-csharp[](samples/globalasax-sample.cs)]

此方法会将应用程序和应用程序要部署到的服务器耦合在一起，并且它们的耦合方式会干扰实现。 为了将它们分离，引入了 [OWIN](https://owin.org/) 来提供一种更为简便的同时使用多个框架的方法。 OWIN 提供了一个管道，可以只添加所需的模块。 托管环境使用 [Startup](xref:fundamentals/startup) 函数配置服务和应用的请求管道。 `Startup` 在应用程序中注册一组中间件。 对于每个请求，应用程序都使用现有处理程序集的链接列表的头指针调用各个中间件组件。 每个中间件组件可以向请求处理管道添加一个或多个处理程序。 为此，需要返回对成为列表新头的处理程序的引用。 每个处理程序负责记住并调用列表中的下一个处理程序。 使用 ASP.NET Core 时，应用程序的入口点是 `Startup`，不再具有 Global.asax 的依赖关系。 结合使用 OWIN 和 .NET Framework 时，使用的管道应如下所示：

[!code-csharp[](samples/webapi-owin.cs)]

这会配置默认路由，默认为 XmlSerialization 而不是 Json。 根据需要向此管道添加其他中间件（加载服务、配置设置、静态文件等）。

ASP.NET Core 使用相似的方法，但是不依赖 OWIN 处理条目。 而是通过 Program.cs `Main` 方法（类似于控制台应用程序）来完成，并且 `Startup` 会通过该处进行加载。

[!code-csharp[](samples/program.cs)]

`Startup` 必须包含 `Configure` 方法。 在 `Configure` 中，向管道添加必要的中间件。 在下列示例（位于默认的网站模板）中，使用了多个扩展方法为管道配置对以下内容的支持：

* [BrowserLink](https://vswebessentials.com/features/browserlink)
* 错误页
* 静态文件
* ASP.NET Core MVC
* Identity

[!code-csharp[](../../common/samples/WebApplication1/Startup.cs?highlight=8,9,10,14,17,19,21&start=58&end=84)]

现在主机和应用程序已分离，这样将来就可以灵活地迁移到其他平台。

有关 ASP.NET Core 启动和中间件的更深入的参考信息，请参阅 <xref:fundamentals/startup> 。

## <a name="storing-configurations"></a>存储配置

ASP.NET 支持存储设置。 这些设置可用于支持应用程序已部署到的环境（以此用途为例）。 常见做法是将所有的自定义键值对存储在 Web.config 文件的 `<appSettings>` 部分中：

[!code-xml[](samples/webconfig-sample.xml)]

应用程序使用 `System.Configuration` 命名空间中的 `ConfigurationManager.AppSettings` 集合读取这些设置：

[!code-csharp[](samples/read-webconfig.cs)]

ASP.NET Core 可以将应用程序的配置数据存储在任何文件中，并可在启动中间件的过程中加载它们。 项目模板中使用的默认文件为 *appsettings.json* ：

[!code-json[](samples/appsettings-sample.json)]

将此文件加载到应用程序内的 `IConfiguration` 的实例的过程在 Startup.cs 中完成：

[!code-csharp[](samples/startup-builder.cs)]

应用读取 `Configuration` 来获得这些设置：

[!code-csharp[](samples/read-appsettings.cs)]

此方法有扩展项，它们可使此过程更加可靠，例如使用[依存关系注入](xref:fundamentals/dependency-injection) (DI) 来加载使用这些值的服务。 DI 方法提供了一组强类型的配置对象。

```csharp
// Assume AppConfiguration is a class representing a strongly-typed version of AppConfiguration section
services.Configure<AppConfiguration>(Configuration.GetSection("AppConfiguration"));
```

**注意：** 有关 ASP.NET Core 配置的更深入的参考信息，请参阅 <xref:fundamentals/configuration/index> 。

## <a name="native-dependency-injection"></a>本机依存关系注入

生成大型可缩放应用程序时，一个重要的目标是将组件和服务松散耦合。 [依赖关系注入](xref:fundamentals/dependency-injection) 是实现此项的一种常用技术，它是 ASP.NET Core 的本机组件。

在 ASP.NET 应用程序中，开发人员依赖第三方库来实现依赖关系注入。 其中的一个库是 Microsoft 模式和做法提供的 [Unity](https://github.com/unitycontainer/unity)。

使用 Unity 设置依赖关系注入的示例是实现 `IDependencyResolver` 包装 `UnityContainer` ：

[!code-csharp[](samples/sample8.cs)]

创建 `UnityContainer` 的实例，注册服务，然后将 `HttpConfiguration` 的依赖关系解析程序设置为容器的 `UnityResolver` 新实例：

[!code-csharp[](samples/sample9.cs)]

在必要时注入 `IProductRepository`：

[!code-csharp[](samples/sample5.cs)]

由于依赖关系注入是 ASP.NET Core 的一部分，因此你可以将服务添加到 `Startup.ConfigureServices` ：

[!code-csharp[](samples/configure-services.cs)]

可在任意位置注入存储库，Unity 亦是如此。

有关 ASP.NET Core 中的依赖关系注入的详细信息，请参阅 <xref:fundamentals/dependency-injection> 。

## <a name="serving-static-files"></a>提供静态文件

Web 开发的一个重要环节是提供客户端静态资产的功能。 HTML、CSS、Javascript 和图像是最常见的静态文件示例。 这些文件需要保存在应用（或 CDN）的发布位置中，并且需要引用它们，以便请求可以加载这些文件。 在 ASP.NET Core 中，此过程发生了变化。

在 ASP.NET 中，静态文件存储在各种目录中，并在视图中进行引用。

在 ASP.NET Core 中，静态文件存储在“Web 根”（&lt;内容根&gt;/wwwroot）中，除非另有配置。 通过从 `Startup.Configure` 调用 `UseStaticFiles` 扩展方法将这些文件加载到请求管道中：

[!code-csharp[](../../fundamentals/static-files/samples/1.x/StaticFilesSample/StartupStaticFiles.cs?highlight=3&name=snippet_ConfigureMethod)]

注意：  如果面向 .NET Framework，则安装 NuGet 包 `Microsoft.AspNetCore.StaticFiles`。

例如，可以通过浏览器从类似 `http://<app>/images/<imageFileName>` 的位置访问 wwwroot/images 文件夹中的图像资产。

**注意：** 有关在 ASP.NET Core 中提供静态文件的更深入的参考信息，请参阅 <xref:fundamentals/static-files> 。

## <a name="additional-resources"></a>其他资源

* [将库移植到 .NET Core](/dotnet/core/porting/libraries)
