---
title: 将配置迁移到 ASP.NET Core
author: ardalis
description: 了解如何将配置从 ASP.NET MVC 项目迁移到 ASP.NET Core MVC 项目。
ms.author: riande
ms.date: 10/14/2016
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
uid: migration/configuration
ms.openlocfilehash: d84204c8c791bfaf36432462cde3a42c294c7966
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059788"
---
# <a name="migrate-configuration-to-aspnet-core"></a>将配置迁移到 ASP.NET Core

作者：[Steve Smith](https://ardalis.com/) 和 [Scott Addie](https://scottaddie.com)

在前面的文章中，我们开始将 [ASP.NET mvc 项目迁移到 ASP.NET CORE mvc](xref:migration/mvc)。 本文将迁移配置。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/migration/configuration/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="setup-configuration"></a>设置配置

ASP.NET Core 不再使用以前版本的 ASP.NET 使用的 *global.asax* 和 *web.config* 文件。 在早期版本的 ASP.NET 中，应用程序启动逻辑放置在 `Application_StartUp` *global.asax* 内的方法中。 稍后，在 ASP.NET MVC 中， *Startup.cs* 文件包含在项目的根目录中;并在应用程序启动时调用。 ASP.NET Core 通过将所有启动逻辑放在 *Startup.cs* 文件中来完全采用这种方法。

*web.config* 文件也已替换为 ASP.NET Core。 配置本身现在可以配置为 *Startup.cs* 中所述的应用程序启动过程的一部分。 配置仍可利用 XML 文件，但通常 ASP.NET Core 项目会将配置值放入 JSON 格式的文件中，例如 *appsettings.json* 。 ASP.NET Core 的配置系统还可以轻松地访问环境变量，从而为特定于环境的值提供 [更安全、更可靠的位置](xref:security/app-secrets) 。 对于不应签入源控件的机密（如连接字符串和 API 密钥），尤其如此。 若要详细了解 ASP.NET Core 中的配置，请参阅 [配置](xref:fundamentals/configuration/index) 。

对于本文，我们将从 [上一篇文章](xref:migration/mvc)中的部分迁移 ASP.NET Core 项目开始。 若要设置配置，请将以下构造函数和属性添加到位于项目根目录中的 *Startup.cs* 文件：

[!code-csharp[](configuration/samples/WebApp1/src/WebApp1/Startup.cs?range=11-16)]

请注意，此时， *Startup.cs* 文件不会进行编译，因为我们仍需要添加以下 `using` 语句：

```csharp
using Microsoft.Extensions.Configuration;
```

*appsettings.json* 使用适当的项模板，将文件添加到项目的根目录：

![添加 AppSettings JSON](configuration/_static/add-appsettings-json.png)

## <a name="migrate-configuration-settings-from-webconfig"></a>从 web.config 迁移配置设置

在元素的 *web.config* 中，我们的 ASP.NET MVC 项目包含所需的数据库连接字符串 `<connectionStrings>` 。 在 ASP.NET Core 项目中，我们要将此信息存储在文件中 *appsettings.json* 。 打开 *appsettings.json* ，请注意，它已包含以下内容：

[!code-json[](../migration/configuration/samples/WebApp1/src/WebApp1/appsettings.json?highlight=4)]

在上面所示的突出显示的行中，将数据库的名称从 **_CHANGE_ME** 更改为数据库的名称。

## <a name="summary"></a>摘要

ASP.NET Core 将应用程序的所有启动逻辑放在一个文件中，可以在其中定义和配置所需的服务和依赖项。 它将 *web.config* 文件替换为灵活的配置功能，该功能可利用各种文件格式（如 JSON）以及环境变量。
