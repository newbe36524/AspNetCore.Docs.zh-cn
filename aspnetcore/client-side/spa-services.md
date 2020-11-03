---
title: 使用 JavaScript Services 在 ASP.NET Core 中创建单页应用程序
author: scottaddie
description: 了解使用 JavaScript Services 创建由 ASP.NET Core 提供支持的单页应用程序 (SPA) 的好处。
monikerRange: '>= aspnetcore-2.1'
ms.author: scaddie
ms.custom: H1Hack27Feb2017, devx-track-js
ms.date: 09/06/2019
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
uid: client-side/spa-services
ms.openlocfilehash: 33ae16c033142aa1c0f4cea53ca746e8a6aaf052
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690657"
---
# <a name="use-javascript-services-to-create-single-page-applications-in-aspnet-core"></a>使用 JavaScript Services 在 ASP.NET Core 中创建单页应用程序

作者：[Scott Addie](https://github.com/scottaddie) 和 [Fiyaz Hasan](https://fiyazhasan.me/)

单页应用程序 (SPA) 因其固有的丰富用户体验而成为一种常用的 Web 应用程序。 将客户端 SPA 框架或库（例如 [Angular](https://angular.io/) 或 [React](https://facebook.github.io/react/)）与服务器端框架（例如 ASP.NET Core）集成在一起可能会很困难。 开发 JavaScript Services 就是为了减少集成过程中的摩擦。 使用它可以在不同的客户端和服务器技术堆栈之间无缝操作。

::: moniker range=">= aspnetcore-3.0"

> [!WARNING]
> 本文所述的功能自 ASP.NET Core 3.0 起被弃用。 [Microsoft.AspNetCore.SpaServices.Extensions](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices.Extensions) NuGet 包提供了一种更简单的 SPA 框架集成机制。 有关详细信息，请参阅 [[Announcement] Obsoleting Microsoft.AspNetCore.SpaServices and Microsoft.AspNetCore.NodeServices](https://github.com/dotnet/AspNetCore/issues/12890)（[公告] 弃用 Microsoft.AspNetCore.SpaServices 和 Microsoft.AspNetCore.NodeServices）。

::: moniker-end

## <a name="what-is-javascript-services"></a>什么是 JavaScript Services

JavaScript Services 是用于 ASP.NET Core 的客户端技术集合。 其目标是将 ASP.NET Core 定位为开发人员生成 SPA 时的首选服务器端平台。

JavaScript Services 由两个不同的 NuGet 包组成：

* [Microsoft.AspNetCore.NodeServices](https://www.nuget.org/packages/Microsoft.AspNetCore.NodeServices/) (NodeServices)
* [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/) (SpaServices)

这些包在以下情况下很有用：

* 在服务器上运行 JavaScript
* 使用 SPA 框架或库
* 通过 Webpack 生成客户端资产

本文重点介绍了如何使用 SpaServices 包。

## <a name="what-is-spaservices"></a>什么是 SpaServices

创建 SpaServices 的目的是将 ASP.NET Core 定位为开发人员生成 SPA 时的首选服务器端平台。 使用 ASP.NET Core 开发 SPA 时不一定要使用 SpaServices，SpaServices 也不会将开发人员束缚在特定的客户端框架中。

SpaServices 可提供有用的基础结构，例如：

* [服务器端预呈现](#server-side-prerendering)
* [Webpack 开发中间件](#webpack-dev-middleware)
* [热模块更换](#hot-module-replacement)
* [路由帮助程序](#routing-helpers)

将这些基础结构组件结合使用时，可增强开发工作流和运行时体验。 这些组件也可单独使用。

## <a name="prerequisites-for-using-spaservices"></a>使用 SpaServices 的先决条件

若要使用 SpaServices，请安装以下各项：

* 带有 npm 的 [Node.js](https://nodejs.org/)（版本 6 或更高版本）

  * 若要确保已安装并且可找到这些组件，请从命令行运行以下命令：

    ```console
    node -v && npm -v
    ```

  * 如果部署到 Azure 网站，则无需执行任何操作 &mdash; 已经在服务器环境中安装 Node.js，并且 Node.js 可供使用。

* [!INCLUDE [](~/includes/net-core-sdk-download-link.md)]

  * 在使用 Visual Studio 2017 的 Windows 上，通过选择“.NET Core 跨平台开发”工作负载来安装该 SDK。

* [Microsoft.AspNetCore.SpaServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SpaServices/) NuGet 包

## <a name="server-side-prerendering"></a>服务器端预呈现

通用（也称为同构）应用程序是一种能够在服务器和客户端上运行的 JavaScript 应用程序。 Angular、React 和其他常用框架针对这种应用程序开发风格提供一个通用平台。 其思路是先通过 Node.js 在服务器上呈现框架组件，然后将进一步的执行任务委托给客户端。

SpaServices 提供的 ASP.NET Core [标记帮助程序](xref:mvc/views/tag-helpers/intro)通过调用服务器上的 JavaScript 函数来简化服务器端预呈现的实现。

### <a name="server-side-prerendering-prerequisites"></a>服务器端预呈现的先决条件

安装 [aspnet-prerendering](https://www.npmjs.com/package/aspnet-prerendering) npm 包：

```console
npm i -S aspnet-prerendering
```

### <a name="server-side-prerendering-configuration"></a>服务器端预呈现配置

可以通过在项目的 *_ViewImports.cshtml* 文件中注册命名空间来发现标记帮助程序：

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/_ViewImports.cshtml?highlight=3)]

这些标记帮助程序通过在 Razor 视图中利用类似 HTML 的语法来抽象化与低级 API 直接通信的复杂性：

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/Home/Index.cshtml?range=5)]

### <a name="asp-prerender-module-tag-helper"></a>asp-prerender-module 标记帮助程序

上面的代码示例中使用的 `asp-prerender-module` 标记帮助程序通过 Node.js 在服务器上执行 *ClientApp/dist/main-server.js* 。 为清楚起见， *main-server.js* 文件是 [Webpack](https://webpack.github.io/) 生成过程中 TypeScript 到 JavaScript 转译任务的产物。 Webpack 定义了入口点别名 `main-server`；此别名的依赖项关系图遍历始于 *ClientApp/boot-server.ts* 文件：

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=53)]

在以下 Angular 示例中， *ClientApp/boot-server.ts* 文件利用 `createServerRenderer` 函数和 `aspnet-prerendering` npm 包的 `RenderResult` 类型通过 Node.js 来配置服务器呈现。 用于服务器端呈现的 HTML 标记传递到解析函数调用，该调用包装在强类型的 JavaScript `Promise` 对象中。 `Promise` 对象的意义在于，它以异步方式将 HTML 标记提供给页面，以注入到 DOM 的占位符元素中。

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-34,79-)]

### <a name="asp-prerender-data-tag-helper"></a>asp-prerender-data 标记帮助程序

与 `asp-prerender-module` 标记帮助程序结合使用时，`asp-prerender-data` 标记帮助程序可用于将上下文信息从 Razor 视图传递到服务器端 JavaScript。 例如，以下标记将用户数据传递到 `main-server` 模块：

[!code-cshtml[](../client-side/spa-services/sample/SpaServicesSampleApp/Views/Home/Index.cshtml?range=9-12)]

收到的 `UserName` 参数使用内置的 JSON 序列化程序进行序列化，并存储在 `params.data` 对象中。 在以下 Angular 示例中，该数据用于在 `h1` 元素内构造个性化问候语：

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-21,38-52,79-)]

在标记帮助程序中传递的属性名称用 **PascalCase** 表示法表示。 与之相反，JavaScript 用 **camelCase** 表示相同的属性名称。 默认的 JSON 序列化配置是造成这种差异的原因所在。

若要扩展上面的代码示例，可以通过解冻提供给 `resolve` 函数的 `globals` 属性，将数据从服务器传递到视图：

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/boot-server.ts?range=6,10-21,57-77,79-)]

`globals` 对象中定义的 `postList` 数组附加到浏览器的全局 `window` 对象。 将此变量提升到全局范围可消除重复的工作，特别是在服务器上加载了一次数据，之后又在客户端上加载相同的数据时。

![附加到 window 对象的全局 postList 变量](spa-services/_static/global_variable.png)

## <a name="webpack-dev-middleware"></a>Webpack 开发中间件

[Webpack 开发中间件](https://webpack.js.org/guides/development/#using-webpack-dev-middleware)引入了简化的开发工作流，Webpack 可根据该工作流按需生成资源。 在浏览器中重新加载页面时，该中间件会自动编译并提供客户端资源。 另一种方法是在第三方依赖项或自定义代码发生更改时，通过项目的 npm 生成脚本手动调用 Webpack。 以下示例显示了 *package.json* 文件中的 npm 生成脚本：

```json
"build": "npm run build:vendor && npm run build:custom",
```

### <a name="webpack-dev-middleware-prerequisites"></a>Webpack 开发中间件的先决条件

安装 [aspnet-webpack](https://www.npmjs.com/package/aspnet-webpack) npm 包：

```console
npm i -D aspnet-webpack
```

### <a name="webpack-dev-middleware-configuration"></a>Webpack 开发中间件配置

Webpack 开发中间件通过 *Startup.cs* 文件的 `Configure` 方法中的以下代码注册到 HTTP 请求管道中：

[!code-csharp[](../client-side/spa-services/sample/SpaServicesSampleApp/Startup.cs?name=snippet_WebpackMiddlewareRegistration&highlight=4)]

通过 `UseStaticFiles` 扩展方法[注册静态文件托管](xref:fundamentals/static-files)之前，必须先调用 `UseWebpackDevMiddleware` 扩展方法。 出于安全原因，仅在应用以开发模式运行时才注册该中间件。

*webpack.config.js* 文件的 `output.publicPath` 属性指示中间件监视 `dist` 文件夹中的更改：

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=6,13-16)]

## <a name="hot-module-replacement"></a>热模块更换

可将 Webpack 的[热模块更换](https://webpack.js.org/concepts/hot-module-replacement/) (HMR) 功能视作 [Webpack 开发中间件](#webpack-dev-middleware)的进化版。 HMR 引入了所有相同的优点，但是通过在编译更改后自动更新页面内容，进一步简化了开发工作流。 不要将其与浏览器的刷新功能混淆，后者会干扰 SPA 的当前内存中状态和调试会话。 Webpack 开发中间件服务与浏览器之间有一个实时链接，这意味着系统会将更改推送到浏览器。

### <a name="hot-module-replacement-prerequisites"></a>热模块更换的先决条件

安装 [webpack-hot-middleware](https://www.npmjs.com/package/webpack-hot-middleware) npm 包：

```console
npm i -D webpack-hot-middleware
```

### <a name="hot-module-replacement-configuration"></a>热模块更换配置

必须在 `Configure` 方法中将 HMR 组件注册到 MVC 的 HTTP 请求管道：

```csharp
app.UseWebpackDevMiddleware(new WebpackDevMiddlewareOptions {
    HotModuleReplacement = true
});
```

与 [Webpack 开发中间件](#webpack-dev-middleware)一样，调用 `UseStaticFiles` 扩展方法之前，必须先调用 `UseWebpackDevMiddleware` 扩展方法。 出于安全原因，仅在应用以开发模式运行时才注册该中间件。

*webpack.config.js* 文件必须定义一个 `plugins` 数组，即便将其留空亦可：

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/webpack.config.js?range=6,25)]

在浏览器中加载应用后，开发人员工具的“控制台”选项卡会提供 HMR 激活确认：

![已连接热模块更换消息](spa-services/_static/hmr_connected.png)

## <a name="routing-helpers"></a>路由帮助程序

在大多数基于 ASP.NET Core 的 SPA 中，除服务器端路由外，通常还需要进行客户端路由。 SPA 和 MVC 路由系统可以独立工作而互不干扰。 但是，有一种极端情况带来了挑战：标识 404 HTTP 响应。

以使用 `/some/page` 的无扩展路由的情况为例。 假设请求的模式与服务器端路由不匹配，但与客户端路由匹配。 现在以针对 `/images/user-512.png` 的传入请求为例，该请求通常需要在服务器上查找映像文件。 如果请求的资源路径与任何服务器端路由或静态文件都不匹配，则客户端应用程序不太可能处理它 &mdash; 通常需要返回 404 HTTP 状态代码。

### <a name="routing-helpers-prerequisites"></a>路由帮助程序的先决条件

安装客户端路由 npm 包。 以 Angular 为例：

```console
npm i -S @angular/router
```

### <a name="routing-helpers-configuration"></a>路由帮助程序配置

在 `Configure` 方法中使用名为 `MapSpaFallbackRoute` 的扩展方法：

[!code-csharp[](../client-side/spa-services/sample/SpaServicesSampleApp/Startup.cs?name=snippet_MvcRoutingTable&highlight=7-9)]

系统按路由配置顺序评估路由。 因此，上面的代码示例中的 `default` 路由先用于模式匹配。

## <a name="create-a-new-project"></a>创建新项目

JavaScript Services 提供预配置的应用程序模板。 在这些模板中，SpaServices 与各种框架和库（例如 Angular、React 和 Redux）结合使用。

可以通过使用 .NET Core CLI 运行以下命令来安装这些模板：

```dotnetcli
dotnet new --install Microsoft.AspNetCore.SpaTemplates::*
```

系统会显示可用 SPA 模板的列表：

| 模板                                 | 短名称 | 语言 | Tags        |
| ------------------------------------------| :--------: | :------: | :---------: |
| 含 Angular 的 MVC ASP.NET Core             | angular    | [C#]     | Web/MVC/SPA |
| 含 React.js 的 MVC ASP.NET Core            | react      | [C#]     | Web/MVC/SPA |
| 含 React.js 和 Redux 的 MVC ASP.NET Core  | reactredux | [C#]     | Web/MVC/SPA |

若要使用其中一个 SPA 模板创建新项目，请在 [dotnet new](/dotnet/core/tools/dotnet-new) 命令中包含该模板的 **短名称** 。 以下命令将使用为服务器端配置的 ASP.NET Core MVC 创建 Angular 应用程序：

```dotnetcli
dotnet new angular
```

### <a name="set-the-runtime-configuration-mode"></a>设置运行时配置模式

存在两种主要运行时配置模式：

* **开发** ：
  * 包含源映射以简化调试。
  * 不优化客户端代码的性能。
* **生产** ：
  * 不包含源映射。
  * 通过捆绑和缩小来优化客户端代码。

ASP.NET Core 使用名为 `ASPNETCORE_ENVIRONMENT` 的环境变量来存储配置模式。 有关详细信息，请参阅[设置环境](xref:fundamentals/environments#set-the-environment)。

### <a name="run-with-net-core-cli"></a>使用 .NET Core CLI 运行

通过在项目根目录下运行以下命令来还原所需的 NuGet 和 npm 包：

```dotnetcli
dotnet restore && npm i
```

生成并运行应用程序：

```dotnetcli
dotnet run
```

应用程序根据[运行时配置模式](#set-the-runtime-configuration-mode)在 localhost 上启动。 在浏览器中导航到 `http://localhost:5000` 会显示登陆页面。

### <a name="run-with-visual-studio-2017"></a>使用 Visual Studio 2017 运行

打开由 [dotnet new](/dotnet/core/tools/dotnet-new) 命令生成的 *.csproj* 文件。 所需的 NuGet 和 npm 包在项目打开时会自动还原。 此还原过程可能需要几分钟的时间，应用程序在此过程完成后即可运行。 单击绿色的运行按钮或按 `Ctrl + F5`，浏览器将打开到应用程序的登陆页面。 应用程序根据[运行时配置模式](#set-the-runtime-configuration-mode)在 localhost 上运行。

## <a name="test-the-app"></a>测试应用

SpaServices 模板已预先配置为使用 [Karma](https://karma-runner.github.io/1.0/index.html) 和 [Jasmine](https://jasmine.github.io/) 运行客户端测试。 Jasmine 是适用于 JavaScript 的常用单元测试框架，而 Karma 是这些测试的测试运行程序。 Karma 配置为使用 [Webpack 开发中间件](#webpack-dev-middleware)，使开发人员无需在每次进行更改时都停止并运行测试。 无论是针对测试用例运行的代码还是测试用例本身，测试都会自动运行。

以 Angular 应用程序为例，系统已经为 *counter.component.spec.ts* 文件中的 `CounterComponent` 提供了两个 Jasmine 测试用例：

[!code-typescript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/app/components/counter/counter.component.spec.ts?range=15-28)]

在 *ClientApp* 目录中打开命令提示符。 运行下面的命令：

```console
npm test
```

该脚本将启动 Karma 测试运行程序，而后者将读取 *karma.conf.js* 文件中定义的设置。 除其他设置外， *karma.conf.js* 还通过其 `files` 数组标识要执行的测试文件：

[!code-javascript[](../client-side/spa-services/sample/SpaServicesSampleApp/ClientApp/test/karma.conf.js?range=4-5,8-11)]

## <a name="publish-the-app"></a>发布应用

有关发布到 Azure 的详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/12474)。

将生成的客户端资产和已发布的 ASP.NET Core 项目组合成一个可即时部署的包的过程可能会很繁琐。 值得庆幸的是，SpaServices 可使用名为 `RunWebpack` 的自定义 MSBuild 目标来协调整个发布过程：

[!code-xml[](../client-side/spa-services/sample/SpaServicesSampleApp/SpaServicesSampleApp.csproj?range=31-45)]

该 MSBuild 目标具有以下职责：

1. 还原 npm 包。
1. 创建第三方客户端资产的生产级生成。
1. 创建自定义客户端资产的生产级生成。
1. 将 Webpack 生成的资产复制到发布文件夹。

运行以下命令时将调用该 MSBuild 目标：

```dotnetcli
dotnet publish -c Release
```

## <a name="additional-resources"></a>其他资源

* [Angular 文档](https://angular.io/docs)
