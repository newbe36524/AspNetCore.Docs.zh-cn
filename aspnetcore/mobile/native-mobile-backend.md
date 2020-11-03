---
title: 使用 ASP.NET Core 为本机移动应用创建后端服务
author: ardalis
description: 了解如何使用 ASP.NET Core MVC 创建后端服务，以支持本机移动应用。
ms.author: riande
ms.date: 12/05/2019
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
uid: mobile/native-mobile-backend
ms.openlocfilehash: 0bbf740cb49b77b476e7e015afee311110bbe5ea
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060984"
---
# <a name="create-backend-services-for-native-mobile-apps-with-aspnet-core"></a>使用 ASP.NET Core 为本机移动应用创建后端服务

作者：[Steve Smith](https://ardalis.com/)

移动应用可与 ASP.NET Core 后端服务通信。 有关从 iOS 模拟器和 Android 仿真程序连接本地 Web 服务的说明，请参阅[从 iOS 模拟器和 Android 仿真程序连接到本地 Web 服务](/xamarin/cross-platform/deploy-test/connect-to-local-web-services)。

[查看或下载后端服务代码示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mobile/native-mobile-backend/sample)

## <a name="the-sample-native-mobile-app"></a>本机移动应用示例

本教程演示如何创建使用 ASP.NET Core MVC 支持本机移动应用的后端服务。 它使用 [Xamarin Forms ToDoRest 应用](/xamarin/xamarin-forms/data-cloud/consuming/rest) 作为其本机客户端，其中包括 Android、 iOS、 Windows Universal 和 Window Phone 设备的单独本机客户端。 你可以遵循链接中的教程来创建本机应用程序（并安装需要的免费 Xamarin 工具），以及下载 Xamarin 示例解决方案。 Xamarin 示例包含一个 ASP.NET Web API 2 服务项目，使用本文中的 ASP.NET Core 应用替换（客户端无需进行任何更改）。

![在 Android 智能手机上运行的 ToDoRest 应用程序](native-mobile-backend/_static/todo-android.png)

### <a name="features"></a>功能

ToDoRest 应用支持列出、 添加、删除和更新待办事项。 每个项都有一个 ID、 Name（名称）、Notes（说明）以及一个指示该项是否已完成的属性 Done。

待办事项的主视图如上所示，列出每个项的名称，并使用复选标记指示它是否已完成。

点击 `+` 图标打开“添加项”对话框：

![“添加项”对话框](native-mobile-backend/_static/todo-android-new-item.png)

点击主列表屏幕上的项将打开一个编辑对话框，在其中可以修改项的名称、 说明以及是否完成，或删除项目：

![“编辑项”对话框](native-mobile-backend/_static/todo-android-edit-item.png)

此示例默认配置为使用托管在 developer.xamarin.com上的后端服务，允许只读操作。 若要使用在你计算机上运行的下一节创建的 ASP.NET Core 应用对其进行测试，你需要更新应用程序的 `RestUrl` 常量。 导航到 `ToDoREST` 项目，然后打开 *Constants.cs* 文件。 使用包含计算机 IP 的 URL 地址替换 `RestUrl`（不是 localhost 或 127.0.0.1，因为此地址用于从设备模拟器中，而不是从你的计算机中访问）。 请包括端口号 (5000)。 为了测试你的服务能否在设备上正常运行，请确保没有活动的防火墙阻止访问此端口。

```csharp
// URL of REST service (Xamarin ReadOnly Service)
//public static string RestUrl = "http://developer.xamarin.com:8081/api/todoitems{0}";

// use your machine's IP address
public static string RestUrl = "http://192.168.1.207:5000/api/todoitems/{0}";
```

## <a name="creating-the-aspnet-core-project"></a>创建 ASP.NET Core 项目

在 Visual Studio 中创建一个新的 ASP.NET Core Web 应用程序。 选择 Web API 模板和 No Authentication（无身份验证）。 将项目命名为 *ToDoApi* 。

![“新建 ASP.NET Web 应用程序”对话框，其中已选中 Web API 项目模板](native-mobile-backend/_static/web-api-template.png)

对于向端口 5000 进行的请求，应用程序均需作出响应。 更新 Program.cs，使其包含 `.UseUrls("http://*:5000")`，以便实现以下操作  ：

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Program.cs?range=10-16&highlight=3)]

> [!NOTE]
> 请确保直接运行应用程序，而不是在 IIS Express 后运行，因为在默认情况下，后者会忽略非本地请求。 从命令提示符处运行 [dotnet run](/dotnet/core/tools/dotnet-run)，或从 Visual Studio 工具栏中的“调试目标”下拉列表中选择应用程序名称配置文件。

添加一个模型类来表示待办事项。 使用 `[Required]` 属性标记必需字段：

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Models/ToDoItem.cs)]

API 方法需要通过某种方式处理数据。 使用原始 Xamarin 示例所用的 `IToDoRepository` 接口：

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Interfaces/IToDoRepository.cs)]

在此示例中，该实现仅使用一个专用项集合：

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Services/ToDoRepository.cs)]

在 *Startup.cs* 中配置该实现:

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Startup.cs?highlight=6&range=29-35)]

现可创建 ToDoItemsController  。

> [!TIP]
> 有关创建 Web API 的详细信息，请参阅[使用 ASP.NET Core MVC 和 Visual Studio 生成首个 Web API](../tutorials/first-web-api.md)。

## <a name="creating-the-controller"></a>创建控制器

在项目中添加新控制器 *ToDoItemsController* 。 它应继承 Microsoft.AspNetCore.Mvc.Controller。 添加 `Route` 属性以指示控制器将处理路径以 `api/todoitems` 开始的请求。 路由中的 `[controller]` 标记会被控制器的名称代替（省略 `Controller` 后缀），这对全局路由特别有用。 详细了解 [路由](../fundamentals/routing.md)。

控制器需要 `IToDoRepository` 才能正常运行；通过控制器的构造函数请求该类型的实例。 在运行时，此实例将使用框架对 [依赖关系注入](../fundamentals/dependency-injection.md) 的支持来提供。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=1-17&highlight=9,14)]

此 API 支持四个不同的 HTTP 谓词来执行对数据源的 CRUD（创建、读取、更新、删除）操作。 最简单的是读取操作，它对应于 HTTP GET 请求。

### <a name="reading-items"></a>读取项目

要请求项列表，可对 `List` 方法使用 GET 请求。 `[HttpGet]` 方法的 `List` 属性指示此操作应仅处理 GET 请求。 此操作的路由是在控制器上指定的路由。 你不一定必须将操作名称用作路由的一部分。 你只需确保每个操作都有唯一的和明确的路由。 路由属性可以分别应用在控制器和方法级别，以此生成特定的路由。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=19-23)]

`List` 方法返回 200 OK 响应代码和所有 ToDo 项，并序列化为 JSON 。

你可以使用多种工具测试新的 API 方法，如 [Postman](https://www.getpostman.com/docs/)，如此处所示：

![Postman 控制台，其中显示一个 todoitem 的 GET 请求，以及显示所返回的三个项目的 JSON 的响应正文](native-mobile-backend/_static/postman-get.png)

### <a name="creating-items"></a>创建项目

按照约定，创建新数据项映射到 HTTP POST 谓词。 `Create` 方法具有应用于它的 `[HttpPost]` 属性，并接受 `ToDoItem` 实例。 由于 `item` 参数在 POST 的正文中传递，因此该参数会指定 `[FromBody]` 属性。

在该方法中，会检查项的有效性和之前是否存在于数据存储，并且如果没有任何问题，则使用存储库添加。 检查 `ModelState.IsValid` 将执行 [模型验证](../mvc/models/validation.md)，应该在每个接受用户输入的 API 方法中执行此步骤。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=25-46)]

示例中使用一个枚举，后者包含传递到移动客户端的错误代码：

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=91-99)]

使用 Postman 测试添加新项，选择 POST 谓词并在请求正文中以 JSON 格式提供新对象。 你还应添加一个请求标头指定 `Content-Type` 为 `application/json`。

![显示 POST 和响应的 Postman 控制台](native-mobile-backend/_static/postman-post.png)

该方法返回在响应中新建的项。

### <a name="updating-items"></a>更新项目

通过使用 HTTP PUT 请求来修改记录。 除了此更改之外，`Edit` 方法几乎与 `Create` 完全相同。 请注意，如果未找到记录，则 `Edit` 操作将返回 `NotFound`(404) 响应。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=48-69)]

若要使用 Postman 进行测试，将谓词更改为 PUT。 在请求正文中指定要更新的对象数据。

![显示 PUT 和响应的 Postman 控制台](native-mobile-backend/_static/postman-put.png)

为了与预先存在的 API 保持一致，此方法在成功时返回 `NoContent` (204) 响应。

### <a name="deleting-items"></a>删除项

删除记录可以通过向服务发出 DELETE 请求并传递要删除项的 ID 来完成。 与更新一样，请求的项不存在时会收到 `NotFound` 响应。 请求成功会得到 `NoContent` (204) 响应。

[!code-csharp[](native-mobile-backend/sample/ToDoApi/src/ToDoApi/Controllers/ToDoItemsController.cs?range=71-88)]

请注意，在测试删除功能时，请求正文中不需要任何内容。

![显示 DELETE 和响应的 Postman 控制台](native-mobile-backend/_static/postman-delete.png)

## <a name="common-web-api-conventions"></a>常见的 Web API 约定

开发应用程序的后端服务时，你将想要使用一组一致的约定或策略来处理横切关注点。 例如，在上面所示服务中，针对不存在的特定记录的请求会收到 `NotFound` 响应，而不是`BadRequest` 响应。 同样，对于此服务，传递模型绑定类型的命令始终检查 `ModelState.IsValid` 并为无效的模型类型返回 `BadRequest`。

一旦为 Api 指定通用策略，一般可以将其封装在 [Filter（筛选器）](../mvc/controllers/filters.md)。 详细了解 [如何封装 ASP.NET Core MVC 应用程序中的通用 API 策略](/archive/msdn-magazine/2016/august/asp-net-core-real-world-asp-net-core-mvc-filters)。

## <a name="additional-resources"></a>其他资源

* [身份验证和授权](/xamarin/xamarin-forms/enterprise-application-patterns/authentication-and-authorization)
