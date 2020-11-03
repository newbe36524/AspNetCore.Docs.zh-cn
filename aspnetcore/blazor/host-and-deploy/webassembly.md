---
title: 托管和部署 ASP.NET Core Blazor WebAssembly
author: guardrex
description: 了解如何使用 ASP.NET Core、内容分发网络 (CDN)、文件服务器和 GitHub 页来托管和部署 Blazor 应用。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/09/2020
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
uid: blazor/host-and-deploy/webassembly
ms.openlocfilehash: c3f537ff3b55f295db478cb097bc99023cc71a87
ms.sourcegitcommit: b5ebaf42422205d212e3dade93fcefcf7f16db39
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/21/2020
ms.locfileid: "92326516"
---
# <a name="host-and-deploy-aspnet-core-no-locblazor-webassembly"></a>托管和部署 ASP.NET Core Blazor WebAssembly

作者：[Luke Latham](https://github.com/guardrex)、[Rainer Stropek](https://www.timecockpit.com)、[Daniel Roth](https://github.com/danroth27)、[Ben Adams](https://twitter.com/ben_a_adams) 和 [Safia Abdalla](https://safia.rocks)

利用 [Blazor WebAssembly 托管模型](xref:blazor/hosting-models#blazor-webassembly)：

* 将 Blazor 应用、其依赖项及 .NET 运行时并行下载到浏览器。
* 应用将在浏览器线程中直接执行。

支持以下部署策略：

* Blazor 应用由 ASP.NET Core 应用提供服务。 [使用 ASP.NET Core 进行托管部署](#hosted-deployment-with-aspnet-core)部分中介绍了此策略。
* Blazor 应用位于静态托管 Web 服务器或服务中，其中未使用 .NET 对 Blazor 应用提供服务。 [独立部署](#standalone-deployment)部分介绍了此策略，包括有关将 Blazor WebAssembly 应用作为 IIS 子应用托管的信息。

## <a name="compression"></a>压缩

发布 Blazor WebAssembly 应用时，将在发布过程中对输出内容进行静态压缩，从而减小应用的大小，并免去运行时压缩的开销。 使用以下压缩算法：

* [Brotli](https://tools.ietf.org/html/rfc7932)（级别最高）
* [Gzip](https://tools.ietf.org/html/rfc1952)

Blazor 依赖于主机提供适当的压缩文件。 使用 ASP.NET Core 托管项目时，主机项目能够执行内容协商并提供静态压缩文件。 托管 Blazor WebAssembly 独立应用时，可能需要额外的工作来确保提供静态压缩文件：

* 有关 IIS `web.config` 压缩配置，请参阅 [IIS：Brotli 和 Gzip 压缩](#brotli-and-gzip-compression) 部分。 
* 如果在不支持静态压缩文件内容协商的静态托管解决方案（例如 GitHub 页面）上进行托管，请考虑配置应用以提取和解码 Brotli 压缩文件：

  * 从 [google/brotli GitHub repository](https://github.com/google/brotli) 中获取 JavaScript Brotli 解码器。 自 2020 年 9 月起，解码器文件命名为 `decode.js`，并且位于存储库的 [`js` 文件夹](https://github.com/google/brotli/tree/master/js)中。
  
    > [!NOTE]
    > [google/brotli GitHub 存储库](https://github.com/google/brotli)中的缩小版 `decode.js` 脚本 (`decode.min.js`) 中存在回归。 在问题[“decode.min.js 中未设置 Window.BrotliDecode”(google/brotli #844)](https://github.com/google/brotli/issues/844) 解决之前，请自行缩小脚本，或者使用 [npm 包](https://www.npmjs.com/package/brotli)。 本部分中的示例代码使用脚本的未缩小版。

  * 更新应用以使用解码器。 将 `wwwroot/index.html` 中结束 `<body>` 标记内的标记更改为以下内容：
  
    ```html
    <script src="decode.js"></script>
    <script src="_framework/blazor.webassembly.js" autostart="false"></script>
    <script>
      Blazor.start({
        loadBootResource: function (type, name, defaultUri, integrity) {
          if (type !== 'dotnetjs' && location.hostname !== 'localhost') {
            return (async function () {
              const response = await fetch(defaultUri + '.br', { cache: 'no-cache' });
              if (!response.ok) {
                throw new Error(response.statusText);
              }
              const originalResponseBuffer = await response.arrayBuffer();
              const originalResponseArray = new Int8Array(originalResponseBuffer);
              const decompressedResponseArray = BrotliDecode(originalResponseArray);
              const contentType = type === 
                'dotnetwasm' ? 'application/wasm' : 'application/octet-stream';
              return new Response(decompressedResponseArray, 
                { headers: { 'content-type': contentType } });
            })();
          }
        }
      });
    </script>
    ```
 
若要禁用压缩，请将 `BlazorEnableCompression` MSBuild 属性添加到应用的项目文件，并将值设置为 `false`：

```xml
<PropertyGroup>
  <BlazorEnableCompression>false</BlazorEnableCompression>
</PropertyGroup>
```

可在命令行界面中使用以下语法将 `BlazorEnableCompression` 属性传递给 [`dotnet publish`](/dotnet/core/tools/dotnet-publish) 命令：

```dotnetcli
dotnet publish -p:BlazorEnableCompression=false
```

## <a name="rewrite-urls-for-correct-routing"></a>重写 URL，以实现正确路由

在 Blazor WebAssembly 应用中路由对页组件的请求不如在 Blazor Server 托管应用中路由请求直接。 假设有一个具有两个组件的 Blazor WebAssembly 应用：

* `Main.razor`：在应用的根目录处加载，并包含指向 `About` 组件 (`href="About"`) 的链接。
* `About.razor`：`About` 组件。

使用浏览器的地址栏（例如，`https://www.contoso.com/`）请求应用的默认文档：

1. 浏览器发出请求。
1. 返回默认页，通常为 `index.html`。
1. `index.html` 启动应用。
1. Blazor 的路由器进行加载，然后呈现 Razor `Main` 组件。

在 Main 页中，选择指向 `About` 组件的链接适用于客户端，因为 Blazor 路由器阻止浏览器在 Internet 上发出请求，针对 `About` 转到 `www.contoso.com`，并为呈现的 `About` 组件本身提供服务。 针对 Blazor WebAssembly 应用中的内部终结点的所有请求，工作原理都相同：这些请求不会触发对 Internet 上的服务器托管资源的基于浏览器的请求。 路由器将在内部处理请求。

如果针对 `www.contoso.com/About` 使用浏览器的地址栏发出请求，则请求会失败。 应用的 Internet 主机上不存在此类资源，所以返回的是“404 - 找不到”响应。

由于浏览器针对客户端页面请求基于 Internet 的主机，因此 Web 服务器和托管服务必须将对服务器上非物理方式资源的所有请求重写为 `index.html` 页。 如果返回 `index.html`，应用的 Blazor 路由器将接管工作并使用正确的资源响应。

部署到 IIS 服务器时，可以将 URL 重写模块与应用的已发布 `web.config` 文件一起使用。 有关详细信息，请参阅 [IIS](#iis) 部分。

## <a name="hosted-deployment-with-aspnet-core"></a>使用 ASP.NET Core 进行托管部署

托管部署通过在 Web 服务器上运行的 [ASP.NET Core](xref:index) 应用为浏览器提供 Blazor WebAssembly 应用。

客户端 Blazor WebAssembly 应用与服务器应用的其他任何静态 Web 资产一起发布到服务器应用的 `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` 文件夹。 这两个应用一起部署。 需要能够托管 ASP.NET Core 应用的 Web 服务器。 对于托管部署，Visual Studio 会在选择 `Hosted` 选项（使用 `dotnet new` 命令时为 `-ho|--hosted`）的情况下，包含 Blazor WebAssembly 应用项目模板（使用 [`dotnet new`](/dotnet/core/tools/dotnet-new) 命令时为 `blazorwasm` 模板） 。

有关托管和部署 ASP.NET Core 应用的详细信息，请参阅 <xref:host-and-deploy/index>。

若要了解如何部署到 Azure 应用服务，请参阅 <xref:tutorials/publish-to-azure-webapp-using-vs>。

## <a name="hosted-deployment-with-multiple-no-locblazor-webassembly-apps"></a>具有多个 Blazor WebAssembly 应用的托管部署

### <a name="app-configuration"></a>应用配置

若要配置托管的 Blazor 解决方案以处理多个 Blazor WebAssembly 应用，请执行以下操作：

* 使用现有的托管 Blazor 解决方案，或者从 Blazor 托管的项目模板创建一个新解决方案。

* 在客户端应用的项目文件中，在 `<PropertyGroup>` 中添加一个值为 `FirstApp` 的 `<StaticWebAssetBasePath>` 属性，以设置项目静态资产的基路径：

  ```xml
  <PropertyGroup>
    ...
    <StaticWebAssetBasePath>FirstApp</StaticWebAssetBasePath>
  </PropertyGroup>
  ```

* 向解决方案添加第二个客户端应用：

  * 向解决方案的文件夹添加一个名为 `SecondClient` 的文件夹。
  * 在 Blazor WebAssembly 项目模板的 `SecondClient` 文件夹中创建一个名为 `SecondBlazorApp.Client` 的 Blazor WebAssembly 应用。
  * 在应用的项目文件中，执行以下操作：

    * 向 `<PropertyGroup>` 添加一个值为 `SecondApp` 的 `<StaticWebAssetBasePath>` 属性：

      ```xml
      <PropertyGroup>
        ...
        <StaticWebAssetBasePath>SecondApp</StaticWebAssetBasePath>
      </PropertyGroup>
      ```

    * 向 `Shared` 项目添加一个项目引用：

      ```xml
      <ItemGroup>
        <ProjectReference Include="..\Shared\{SOLUTION NAME}.Shared.csproj" />
      </ItemGroup>
      ```

      占位符 `{SOLUTION NAME}` 是解决方案的名称。

* 在服务器应用的项目文件中，为新增的客户端应用创建一个项目引用：

  ```xml
  <ItemGroup>
    ...
    <ProjectReference Include="..\SecondClient\SecondBlazorApp.Client.csproj" />
  </ItemGroup>
  ```

* 在服务器应用的 `Properties/launchSettings.json` 文件中，配置 Kestrel 配置文件 (`{SOLUTION NAME}.Server`) 的 `applicationUrl`，以访问位于端口 5001 和 5002 的客户端应用：

  ```json
  "applicationUrl": "https://localhost:5001;https://localhost:5002",
  ```

* 在服务器应用的 `Startup.Configure` 方法 (`Startup.cs`) 中，删除在调用 <xref:Microsoft.AspNetCore.Builder.HttpsPolicyBuilderExtensions.UseHttpsRedirection%2A> 后显示的以下行：

  ```csharp
  app.UseBlazorFrameworkFiles();
  app.UseStaticFiles();

  app.UseRouting();

  app.UseEndpoints(endpoints =>
  {
      endpoints.MapRazorPages();
      endpoints.MapControllers();
      endpoints.MapFallbackToFile("index.html");
  });
  ```

  添加将请求映射到客户端应用的中间件。 下面的示例将中间件配置为在以下情况下运行：

  * 原始客户端应用的请求端口为 5001，或新增的客户端应用的请求端口为 5002。
  * 原始客户端应用的请求主机为 `firstapp.com`，或新增的客户端应用的请求主机为 `secondapp.com`。

    > [!NOTE]
    > 本部分中显示的示例需要其他配置以实现以下目的：
    >
    > * 访问示例主机域 `firstapp.com` 和 `secondapp.com` 上的应用。
    > * 获取客户端应用的证书以启用 TLS 安全性 (HTTPS)。
    >
    > 所需的配置不在本文内容范围内，并且取决于解决方案的托管方式。 有关详细信息，请参阅[托管和部署文章](xref:host-and-deploy/index)。

  将以下代码放在先前已删除行的位置：

  ```csharp
  app.MapWhen(ctx => ctx.Request.Host.Port == 5001 || 
      ctx.Request.Host.Equals("firstapp.com"), first =>
  {
      first.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/FirstApp" + ctx.Request.Path;
          return nxt();
      });

      first.UseBlazorFrameworkFiles("/FirstApp");
      first.UseStaticFiles();
      first.UseStaticFiles("/FirstApp");
      first.UseRouting();

      first.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/FirstApp/{*path:nonfile}", 
              "FirstApp/index.html");
      });
  });
  
  app.MapWhen(ctx => ctx.Request.Host.Port == 5002 || 
      ctx.Request.Host.Equals("secondapp.com"), second =>
  {
      second.Use((ctx, nxt) =>
      {
          ctx.Request.Path = "/SecondApp" + ctx.Request.Path;
          return nxt();
      });

      second.UseBlazorFrameworkFiles("/SecondApp");
      second.UseStaticFiles();
      second.UseStaticFiles("/SecondApp");
      second.UseRouting();

      second.UseEndpoints(endpoints =>
      {
          endpoints.MapControllers();
          endpoints.MapFallbackToFile("/SecondApp/{*path:nonfile}", 
              "SecondApp/index.html");
      });
  });
  ```

* 在服务器应用的天气预报控制器 (`Controllers/WeatherForecastController.cs`) 中，将到 `WeatherForecastController` 的现有路由 (`[Route("[controller]")]`) 替换为以下路由：

  ```csharp
  [Route("FirstApp/[controller]")]
  [Route("SecondApp/[controller]")]
  ```

  先前添加到服务器应用的 `Startup.Configure` 方法的中间件会将向 `/WeatherForecast` 发出的传入请求修改为 `/FirstApp/WeatherForecast` 或 `/SecondApp/WeatherForecast`，具体取决于端口 (5001/5002) 或域 (`firstapp.com`/`secondapp.com`)。 为了将天气数据从服务器应用返回到客户端应用，需要前面的控制器路由。

### <a name="static-assets-and-class-libraries"></a>静态资产和类库

对于静态资产，请使用以下方法：

* 如果资产位于客户端应用的 `wwwroot` 文件夹中，请正常提供其路径：

  ```razor
  <img alt="..." src="/{ASSET FILE NAME}" />
  ```

* 如果资产位于 [Razor 类库 (RCL)](xref:blazor/components/class-libraries) 的 `wwwroot` 文件夹中，请按照 [RCL 文章](xref:razor-pages/ui-class#consume-content-from-a-referenced-rcl)中的指导，在客户端应用程序中引用静态资产：

  ```razor
  <img alt="..." src="_content/{LIBRARY NAME}/{ASSET FILE NAME}" />
  ```

::: moniker range=">= aspnetcore-5.0"

通常引用按类库提供给客户端应用的组件。 如果任何组件需要样式表或 JavaScript 文件，请使用以下方法之一来获取静态资产：

* 客户端应用的 `wwwroot/index.html` 文件可以链接 (`<link>`) 到静态资产。
* 组件可以使用框架的 [`Link` 组件](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements)获取静态资产。

下面的示例中演示了上述方法。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

通常引用按类库提供给客户端应用的组件。 如果任何组件需要样式表或 JavaScript 文件，客户端应用的 `wwwroot/index.html` 文件必须包含正确的静态资产链接。 下面的示例中演示了这些方法。

::: moniker-end

将以下 `Jeep` 组件添加到其中一个客户端应用中。 `Jeep` 组件使用：

* 来自客户端应用的 `wwwroot` 文件夹中的图像 (`jeep-cj.png`)。
* 来自[已添加的 Razor 组件库](xref:blazor/components/class-libraries) (`JeepImage`) `wwwroot` 文件夹中的图像 (`jeep-yj.png`)。
* 将 `JeepImage` 库添加到解决方案中后，RCL 项目模板会自动创建示例组件 (`Component1`)。

```razor
@page "/Jeep"

<h1>1979 Jeep CJ-5&trade;</h1>

<p>
    <img alt="1979 Jeep CJ-5&trade;" src="/jeep-cj.png" />
</p>

<h1>1991 Jeep YJ&trade;</h1>

<p>
    <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
</p>

<p>
    <em>Jeep CJ-5</em> and <em>Jeep YJ</em> are a trademarks of 
    <a href="https://www.fcagroup.com">Fiat Chrysler Automobiles</a>.
</p>

<JeepImage.Component1 />
```

> [!WARNING]
> 除非图像归你所有，否则不要公开发布车辆的图像。 否则，可能会侵犯版权。

::: moniker range=">= aspnetcore-5.0"

另外，还可以向库的 `Component1` 组件(`Component1.razor`) 添加库的 `jeep-yj.png` 图像。 若要向客户端应用的页面提供 `my-component` CSS 类，请使用框架的 [`Link` 组件](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements)链接到库的样式表：

```razor
<div class="my-component">
    <Link href="_content/JeepImage/styles.css" rel="stylesheet" />

    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

或者，使用 [`Link` 组件](xref:blazor/fundamentals/additional-scenarios#influence-html-head-tag-elements)的另一种方法是从客户端应用的 `wwwroot/index.html` 文件加载样式表。 这种方法使样式表可用于客户端应用中的所有组件：

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

还可以向库的 `Component1` 组件 (`Component1.razor`) 添加库的 `jeep-yj.png` 图像：

```razor
<div class="my-component">
    <h1>JeepImage.Component1</h1>

    <p>
        This Blazor component is defined in the <strong>JeepImage</strong> package.
    </p>

    <p>
        <img alt="1991 Jeep YJ&trade;" src="_content/JeepImage/jeep-yj.png" />
    </p>
</div>
```

客户端应用的 `wwwroot/index.html` 文件要求库的样式表添加以下 `<link>` 标记：

```html
<head>
    ...
    <link href="_content/JeepImage/styles.css" rel="stylesheet" />
</head>
```

::: moniker-end

向客户端应用的 `NavMenu` 组件中的 `Jeep` 组件 (`Shared/NavMenu.razor`) 添加导航：

```razor
<li class="nav-item px-3">
    <NavLink class="nav-link" href="Jeep">
        <span class="oi oi-list-rich" aria-hidden="true"></span> Jeep
    </NavLink>
</li>
```

有关 RCL 的详细信息，请参阅：

* <xref:blazor/components/class-libraries>
* <xref:razor-pages/ui-class>

## <a name="standalone-deployment"></a>独立部署

独立部署将 Blazor WebAssembly 应用作为客户端直接请求的一组静态文件提供。 任何静态文件服务器均可提供 Blazor 应用。

独立部署资产将发布到 `/bin/Release/{TARGET FRAMEWORK}/publish/wwwroot` 文件夹。

### <a name="azure-app-service"></a>Azure 应用服务

可以将 Blazor WebAssembly 应用部署到 Windows 上的 Azure 应用服务，该服务在 [IIS](#iis) 上托管应用。

目前不支持将独立的 Blazor WebAssembly 应用部署到适用于 Linux 的 Azure 应用服务。 目前无法提供用于托管应用的 Linux 服务器映像。 正在进行此工作以支持此场景。

### <a name="iis"></a>IIS

IIS 是适用于 Blazor 应用的强大静态文件服务器。 要配置 IIS 以托管 Blazor，请参阅[在 IIS 上生成静态网站](/iis/manage/creating-websites/scenario-build-a-static-website-on-iis)。

`/bin/Release/{TARGET FRAMEWORK}/publish` 文件夹中已创建发布的资产。 在 Web 服务器或托管服务上托管 `publish` 文件夹的内容。

#### <a name="webconfig"></a>web.config

发布 Blazor 项目时，将使用以下 IIS 配置创建 `web.config` 文件：

* 对以下文件扩展名设置 MIME 类型：
  * `.dll`: `application/octet-stream`
  * `.json`: `application/json`
  * `.wasm`: `application/wasm`
  * `.woff`: `application/font-woff`
  * `.woff2`: `application/font-woff`
* 对以下 MIME 类型启用 HTTP 压缩：
  * `application/octet-stream`
  * `application/wasm`
* 建立 URL 重写模块规则：
  * 提供应用的静态资产所驻留的子目录 (`wwwroot/{PATH REQUESTED}`)。
  * 创建 SPA 回退路由，以便非文件资产请求能够重定向到应用的静态资产文件夹中的默认文档 (`wwwroot/index.html`)。
  
#### <a name="use-a-custom-webconfig"></a>使用自定义 web.config

要使用自定义 `web.config` 文件，请将自定义 `web.config` 文件置于项目文件夹的根目录下。 将项目配置为使用应用项目文件中的 `PublishIISAssets` 发布特定于 IIS 的资源，并发布项目：

```xml
<PropertyGroup>
  <PublishIISAssets>true</PublishIISAssets>
</PropertyGroup>
```

#### <a name="install-the-url-rewrite-module"></a>安装 URL 重写模块

重写 URL 必须使用 [URL 重写模块](https://www.iis.net/downloads/microsoft/url-rewrite)。 此模块默认不安装，且不适用于安装为 Web 服务器 (IIS) 角色服务功能。 必须从 IIS 网站下载该模块。 使用 Web 平台安装程序安装模块：

1. 以本地方式导航到 [URL 重写模块下载页](https://www.iis.net/downloads/microsoft/url-rewrite#additionalDownloads)。 对于英语版本，请选择“WebPI”以下载 WebPI 安装程序。 对于其他语言，请选择适当的服务器体系结构 (x86/x64) 下载安装程序。
1. 将安装程序复制到服务器。 运行安装程序。 选择“安装”按钮，并接受许可条款。 安装完成后无需重启服务器。

#### <a name="configure-the-website"></a>配置网站

将网站的物理路径设置为应用的文件夹。 该文件夹包含：

* `web.config` 文件，IIS 使用该文件配置网站，包括所需的重定向规则和文件内容类型。
* 应用的静态资产文件夹。

#### <a name="host-as-an-iis-sub-app"></a>作为 IIS 子应用托管

如果独立应用作为 IIS 子应用托管，请执行下列任一操作：

* 禁用继承的 ASP.NET Core 模块处理程序。

  通过向文件添加 `<handlers>` 部分，删除 Blazor 应用的已发布 `web.config` 文件中的处理程序：

  ```xml
  <handlers>
    <remove name="aspNetCore" />
  </handlers>
  ```

* 通过使用 `<location>` 元素并且将 `inheritInChildApplications` 设置为 `false`，禁止继承根（父级）应用的 `<system.webServer>` 部分：

  ```xml
  <?xml version="1.0" encoding="utf-8"?>
  <configuration>
    <location path="." inheritInChildApplications="false">
      <system.webServer>
        <handlers>
          <add name="aspNetCore" ... />
        </handlers>
        <aspNetCore ... />
      </system.webServer>
    </location>
  </configuration>
  ```

除[配置应用的基路径](xref:blazor/host-and-deploy/index#app-base-path)外，还需删除处理程序或禁用继承。 在 IIS 中配置子应用时，在应用的 `index.html` 文件中将应用基路径设置为 IIS 别名。

#### <a name="brotli-and-gzip-compression"></a>Brotli 和 Gzip 压缩

通过 `web.config` 可将 IIS 配置为提供 Brotli 或 Gzip 压缩的 Blazor 资产。 有关示例配置，请参阅 [`web.config`](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/blazor/host-and-deploy/webassembly/_samples/web.config?raw=true)。

#### <a name="troubleshooting"></a>疑难解答

如果你看到“500 - 内部服务器错误”，且 IIS 管理器在尝试访问网站配置时抛出错误，请确认是否已安装 URL 重写模块。 如果未安装该模块，则 IIS 无法分析 `web.config` 文件。 这可以防止 IIS 管理器加载网站配置，并防止网站对 Blazor 的静态文件提供服务。

有关排查部署到 IIS 的问题的详细信息，请参阅 <xref:test/troubleshoot-azure-iis>。

### <a name="azure-storage"></a>Azure 存储

[Azure 存储](/azure/storage/)静态文件承载允许无服务器的 Blazor 应用承载。 支持自定义域名、Azure 内容分发网络 (CDN) 以及 HTTPS。

为存储帐户上的静态网站承载启用 blob 服务时：

* 设置“索引文档名称”到 `index.html`。
* 设置“错误文档路径”到 `index.html`。 Razor 组件和其他非文件终结点不会驻留在由 blob 服务存储的静态内容中的物理路径中。 当收到 Blazor 路由器应处理的对这些资源之一的请求时，由 blob 服务生成的“404 - 未找到”错误会将此请求路由到“错误文档路径”。 返回 `index.html` blob，Blazor 路由器会加载并处理此路径。

如果由于文件的 `Content-Type` 标头中的 MIME 类型不正确，导致在运行时未加载文件，请执行以下任一操作：

* 配置工具，用于在部署文件时设置正确的 MIME 类型（`Content-Type` 标头）。
* 在部署应用后更改文件的 MIME 类型（`Content-Type` 标头）。

  在每个文件的存储资源管理器（Azure 门户）中，执行以下操作：
  
  1. 右键单击该文件并选择“属性”。
  1. 设置“ContentType”并选择“保存”按钮 。

有关更多信息，请参阅 [Azure 存储中的静态网站承载](/azure/storage/blobs/storage-blob-static-website)。

### <a name="nginx"></a>Nginx

以下 `nginx.conf` 文件已简化，以展示如何配置 Nginx，以便每当在磁盘上找不到相应文件时，就发送 `index.html` 文件。

```
events { }
http {
    server {
        listen 80;

        location / {
            root      /usr/share/nginx/html;
            try_files $uri $uri/ /index.html =404;
        }
    }
}
```

使用 [`limit_req`](https://nginx.org/docs/http/ngx_http_limit_req_module.html#limit_req) 设置 [NGINX 突发速率限制](https://www.nginx.com/blog/rate-limiting-nginx/#bursts)时，Blazor WebAssembly 应用可能需要一个较大的 `burst` 参数值来容纳应用发出的请求数。 首先，将值设置为不低于 60：

```
http {
    server {
        ...

        location / {
            ...

            limit_req zone=one burst=60 nodelay;
        }
    }
}
```

如果浏览器开发人员工具或网络流量工具指示请求收到“503 - 服务不可用”状态代码，则将该值调高。

有关生产 Nginx Web 服务器配置的详细信息，请参阅 [Creating NGINX Plus and NGINX Configuration Files](https://docs.nginx.com/nginx/admin-guide/basic-functionality/managing-configuration-files/)（创建 NGINX 增强版和 NGINX 配置文件）。

### <a name="nginx-in-docker"></a>Docker 中的 Nginx

要使用 Nginx 在 Docker 中托管 Blazor，请设置 Dockerfile，以使用基于 Alpine 的 Nginx 映像。 更新 Dockerfile，以将 `nginx.config` 文件复制到容器。

向 Dockerfile 添加一行，如下面的示例中所示：

```dockerfile
FROM nginx:alpine
COPY ./bin/Release/netstandard2.0/publish /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/nginx.conf
```

### <a name="apache"></a>Apache

若要将 Blazor WebAssembly 应用部署到 CentOS 7 或更高版本，请执行以下操作：

1. 创建 Apache 配置文件。 下面的示例展示了一个简化的配置文件 (`blazorapp.config`)：

   ```
   <VirtualHost *:80>
       ServerName www.example.com
       ServerAlias *.example.com

       DocumentRoot "/var/www/blazorapp"
       ErrorDocument 404 /index.html

       AddType application/wasm .wasm
       AddType application/octet-stream .dll
   
       <Directory "/var/www/blazorapp">
           Options -Indexes
           AllowOverride None
       </Directory>

       <IfModule mod_deflate.c>
           AddOutputFilterByType DEFLATE text/css
           AddOutputFilterByType DEFLATE application/javascript
           AddOutputFilterByType DEFLATE text/html
           AddOutputFilterByType DEFLATE application/octet-stream
           AddOutputFilterByType DEFLATE application/wasm
           <IfModule mod_setenvif.c>
           BrowserMatch ^Mozilla/4 gzip-only-text/html
           BrowserMatch ^Mozilla/4.0[678] no-gzip
           BrowserMatch bMSIE !no-gzip !gzip-only-text/html
       </IfModule>
       </IfModule>

       ErrorLog /var/log/httpd/blazorapp-error.log
       CustomLog /var/log/httpd/blazorapp-access.log common
   </VirtualHost>
   ```

1. 将 Apache 配置文件放入 `/etc/httpd/conf.d/` 目录（这是 CentOS 7 中的默认 Apache 配置目录）。

1. 将应用的文件放入 `/var/www/blazorapp` 目录（配置文件中特定于 `DocumentRoot` 的位置）。

1. 重启 Apache 服务。

有关详细信息，请参阅 [`mod_mime`](https://httpd.apache.org/docs/2.4/mod/mod_mime.html) 和 [`mod_deflate`](https://httpd.apache.org/docs/current/mod/mod_deflate.html)。

### <a name="github-pages"></a>GitHub 页

要处理 URL 重写，请使用脚本添加 `wwwroot/404.html` 文件，该脚本可处理到 `index.html` 页的重定向请求。 有关示例，请参阅 [SteveSandersonMS/BlazorOnGitHubPages GitHub 存储库](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)：

* [`wwwroot/404.html`](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/wwwroot/404.html)
* [实时站点](https://stevesandersonms.github.io/BlazorOnGitHubPages/)

如果使用项目站点而非组织站点，请在 `wwwroot/index.html` 中更新 `<base>` 标记。 将 `href` 属性值设置为，包含尾部斜杠的 GitHub 存储库名称（例如，`/my-repository/`）。 在 [SteveSandersonMS/BlazorOnGitHubPages GitHub 存储库](https://github.com/SteveSandersonMS/BlazorOnGitHubPages)中，将在发布时通过 [`.github/workflows/main.yml` 配置文件](https://github.com/SteveSandersonMS/BlazorOnGitHubPages/blob/master/.github/workflows/main.yml)更新基本 `href`。

> [!NOTE]
> [SteveSandersonMS/BlazorOnGitHubPages GitHub 存储库](https://github.com/SteveSandersonMS/BlazorOnGitHubPages) 不归 .NET Foundation 或 Microsoft 所有，也不由它们提供维护和支持。

## <a name="host-configuration-values"></a>主机配置值

在开发环境中，[Blazor WebAssembly 应用](xref:blazor/hosting-models#blazor-webassembly)可以在运行时接受以下主机配置值作为命令行参数。

### <a name="content-root"></a>内容根

`--contentroot` 参数设置包含应用内容文件的目录的绝对路径（[内容根目录](xref:fundamentals/index#content-root)）。 在下面的示例中，`/content-root-path` 是应用的内容根路径。

* 以本地方式在命令提示符下运行应用时传递该参数。 在应用的目录中，执行以下操作：

  ```dotnetcli
  dotnet run --contentroot=/content-root-path
  ```

* 在 IIS Express 配置文件中，向应用的 `launchSettings.json` 文件添加条目。 如果使用 Visual Studio 调试器并在命令提示符中运行 `dotnet run` 来运行应用，使用的是此设置。

  ```json
  "commandLineArgs": "--contentroot=/content-root-path"
  ```

* 在 Visual Studio 中，在“属性” > “调试” > “应用程序参数”中指定参数  。 在 Visual Studio 属性页中设置参数可将参数添加到 `launchSettings.json` 文件。

  ```console
  --contentroot=/content-root-path
  ```

### <a name="path-base"></a>基路径

`--pathbase` 参数可设置使用非根相对 URL 路径本地运行的应用的应用基路径（将 `<base>` 标记 `href` 针对暂存和生产设置为 `/` 之外的路径）。 在下面的示例中，`/relative-URL-path` 是应用的基路径。 有关详细信息，请参阅[应用基路径](xref:blazor/host-and-deploy/index#app-base-path)。

> [!IMPORTANT]
> 不同于向 `href` 标记的 `<base>` 提供的路径，传递 `--pathbase` 参数值时不包括尾部反斜杠 (`/`)。 如果在 `<base>` 标记中以 `<base href="/CoolApp/">` 形式（包括尾部反斜杠）提供应用基路径，则以 `--pathbase=/CoolApp` 形式（无尾部反斜杠）传递命令行参数值。

* 以本地方式在命令提示符下运行应用时传递该参数。 在应用的目录中，执行以下操作：

  ```dotnetcli
  dotnet run --pathbase=/relative-URL-path
  ```

* 在 IIS Express 配置文件中，向应用的 `launchSettings.json` 文件添加条目。 如果使用 Visual Studio 调试器并在命令提示符中运行 `dotnet run` 来运行应用，使用的是此设置。

  ```json
  "commandLineArgs": "--pathbase=/relative-URL-path"
  ```

* 在 Visual Studio 中，在“属性” > “调试” > “应用程序参数”中指定参数  。 在 Visual Studio 属性页中设置参数可将参数添加到 `launchSettings.json` 文件。

  ```console
  --pathbase=/relative-URL-path
  ```

### <a name="urls"></a>URL

`--urls` 参数设置 IP 地址或主机地址，其中包含侦听请求的端口和协议。

* 以本地方式在命令提示符下运行应用时传递该参数。 在应用的目录中，执行以下操作：

  ```dotnetcli
  dotnet run --urls=http://127.0.0.1:0
  ```

* 在 IIS Express 配置文件中，向应用的 `launchSettings.json` 文件添加条目。 如果使用 Visual Studio 调试器并在命令提示符中运行 `dotnet run` 来运行应用，使用的是此设置。

  ```json
  "commandLineArgs": "--urls=http://127.0.0.1:0"
  ```

* 在 Visual Studio 中，在“属性” > “调试” > “应用程序参数”中指定参数  。 在 Visual Studio 属性页中设置参数可将参数添加到 `launchSettings.json` 文件。

  ```console
  --urls=http://127.0.0.1:0
  ```

::: moniker range=">= aspnetcore-5.0"

## <a name="configure-the-trimmer"></a>配置裁边器

Blazor 对每个发布版本执行中间语言 (IL) 剪裁，以从输出程序集中删除不必要的 IL。 有关详细信息，请参阅 <xref:blazor/host-and-deploy/configure-trimmer>。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## <a name="configure-the-linker"></a>配置链接器

Blazor 对每个发布版本执行中间语言 (IL) 链接，以从输出程序集中删除不必要的 IL。 有关详细信息，请参阅 <xref:blazor/host-and-deploy/configure-linker>。

::: moniker-end

## <a name="custom-boot-resource-loading"></a>自定义启动资源加载

Blazor WebAssembly 应用可使用 `loadBootResource` 函数进行初始化，以覆盖内置的启动资源加载机制。 在以下场景中使用 `loadBootResource`：

* 允许用户从 CDN 加载静态资源，例如时区数据或 `dotnet.wasm`。
* 使用 HTTP 请求加载压缩的程序集，并将其解压缩到不支持从服务器提取压缩内容的主机的客户端上。
* 将每个 `fetch` 请求重定向到新名称，从而为资源提供别名。

`loadBootResource` 参数如下表所示。

| 参数    | 描述 |
| ------------ | ----------- |
| `type`       | 资源类型。 允许使用的类型：`assembly`、`pdb`、`dotnetjs`、`dotnetwasm`、`timezonedata` |
| `name`       | 资源的名称。 |
| `defaultUri` | 资源的相对或绝对 URI。 |
| `integrity`  | 表示响应中的预期内容的完整性字符串。 |

`loadBootResource` 返回以下任何内容以替代加载过程：

* URI 字符串。 在以下示例 (`wwwroot/index.html`) 中，以下文件来自 CDN (`https://my-awesome-cdn.com/`)：

  * `dotnet.*.js`
  * `dotnet.wasm`
  * 时区数据

  ```html
  ...

  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        console.log(`Loading: '${type}', '${name}', '${defaultUri}', '${integrity}'`);
        switch (type) {
          case 'dotnetjs':
          case 'dotnetwasm':
          case 'timezonedata':
            return `https://my-awesome-cdn.com/blazorwebassembly/3.2.0/${name}`;
        }
      }
    });
  </script>
  ```

* `Promise<Response>`. 在标头中传递 `integrity` 参数以保持默认的完整性检查行为。

  以下示例 (`wwwroot/index.html`) 将一个自定义 HTTP 标头添加到出站请求，并将 `integrity` 参数传递到 `fetch` 调用：
  
  ```html
  <script src="_framework/blazor.webassembly.js" autostart="false"></script>
  <script>
    Blazor.start({
      loadBootResource: function (type, name, defaultUri, integrity) {
        return fetch(defaultUri, { 
          cache: 'no-cache',
          integrity: integrity,
          headers: { 'MyCustomHeader': 'My custom value' }
        });
      }
    });
  </script>
  ```

* `null`/`undefined`，这将导致默认的加载行为。

外部源必须返回浏览器所需的 CORS 标头以允许跨域资源加载。 CDN 通常会默认提供所需的标头。

你只需指定自定义行为的类型。 框架会根据默认的加载行为加载未指定到 `loadBootResource` 的类型。

## <a name="change-the-filename-extension-of-dll-files"></a>更改 DLL 文件的文件扩展名

如果需要更改应用的已发布 `.dll` 文件的文件扩展名，请按照本部分中的指导进行操作。

发布应用后，使用 shell 脚本或 DevOps 生成管道将 `.dll` 文件重命名，以使用其他文件扩展名。 将 `.dll` 文件的目标位置设为应用的已发布输出的 `wwwroot` 目录中（例如 `{CONTENT ROOT}/bin/Release/netstandard2.1/publish/wwwroot`）。

在下面的示例中，重命名 `.dll` 文件，以使用 `.bin` 文件扩展名。

在 Windows 上：

```powershell
dir .\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content .\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content .\_framework\blazor.boot.json
```

如果服务工作进程资产也在使用中，请添加以下命令：

```powershell
((Get-Content .\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content .\service-worker-assets.js
```

在 Linux 或 macOS 上：

```console
for f in _framework/_bin/*; do mv "$f" "`echo $f | sed -e 's/\.dll\b/.bin/g'`"; done
sed -i 's/\.dll"/.bin"/g' _framework/blazor.boot.json
```

如果服务工作进程资产也在使用中，请添加以下命令：

```console
sed -i 's/\.dll"/.bin"/g' service-worker-assets.js
```
   
要使用不同于 `.bin` 的其他文件扩展名，请在前面的命令中替换 `.bin`。

若要处理压缩的 `blazor.boot.json.gz` 和 `blazor.boot.json.br` 文件，请采用以下方法之一：

* 删除压缩的 `blazor.boot.json.gz` 和 `blazor.boot.json.br` 文件。 此方法禁用压缩。
* 重新压缩更新后的 `blazor.boot.json` 文件。

上述指导也适用于正在使用服务工作进程资产的情况。 删除或重新压缩 `wwwroot/service-worker-assets.js.br` 和 `wwwroot/service-worker-assets.js.gz`。 否则，浏览器中的文件完整性检查将失败。

以下 Windows 示例使用项目根目录中的 PowerShell 脚本。

`ChangeDLLExtensions.ps1:`:

```powershell
param([string]$filepath,[string]$tfm)
dir $filepath\bin\Release\$tfm\wwwroot\_framework\_bin | rename-item -NewName { $_.name -replace ".dll\b",".bin" }
((Get-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json
Remove-Item $filepath\bin\Release\$tfm\wwwroot\_framework\blazor.boot.json.gz
```

如果服务工作进程资产也在使用中，请添加以下命令：

```powershell
((Get-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js -Raw) -replace '.dll"','.bin"') | Set-Content $filepath\bin\Release\$tfm\wwwroot\service-worker-assets.js
```

在项目文件中，在发布应用后运行脚本：

```xml
<Target Name="ChangeDLLFileExtensions" AfterTargets="Publish" Condition="'$(Configuration)'=='Release'">
  <Exec Command="powershell.exe -command &quot;&amp; { .\ChangeDLLExtensions.ps1 '$(SolutionDir)' '$(TargetFramework)'}&quot;" />
</Target>
```

> [!NOTE]
> 重命名和延迟加载相同的程序集时，请参阅 <xref:blazor/webassembly-lazy-load-assemblies#onnavigateasync-events-and-renamed-assembly-files> 中的指南。

## <a name="resolve-integrity-check-failures"></a>解决完整性检查失败

当 Blazor WebAssembly 下载应用的启动文件时，它会指示浏览器对响应执行完整性检查。 它使用 `blazor.boot.json` 文件中的信息为 `.dll`、`.wasm` 和其他文件指定预期的 SHA-256 哈希值。 这对以下原因有所帮助：

* 它可确保不会出现加载不一致文件集的风险，例如，在用户正在下载应用程序文件时将新部署应用到 Web 服务器的情况。 不一致的文件可能导致未定义的行为。
* 它可确保用户的浏览器从不缓存不一致或无效的响应，这些响应可能会阻止他们启动应用（即使他们手动刷新了页面也是如此）。
* 它可以安全地缓存响应，甚至无需检查服务器端更改，直到预期的 SHA-256 哈希本身发生更改，因此，后续页面加载需要较少的请求即可快速完成。

如果你的 Web 服务器返回的响应与预期的 SHA-256 哈希不匹配，你将看到类似于以下内容的错误显示在浏览器的开发人员控制台中：

```
Failed to find a valid digest in the 'integrity' attribute for resource 'https://myapp.example.com/_framework/MyBlazorApp.dll' with computed SHA-256 integrity 'IIa70iwvmEg5WiDV17OpQ5eCztNYqL186J56852RpJY='. The resource has been blocked.
```

在大多数情况下，这不是完整性检查本身的问题。 相反，它表示存在其他问题，并且完整性检查会警告你其他问题。

### <a name="diagnosing-integrity-problems"></a>诊断完整性问题

生成应用时，生成的 `blazor.boot.json` 清单将描述生成输出生成时启动资源（例如，`.dll`、`.wasm` 和其他文件）的 SHA-256 哈希。 只要 `blazor.boot.json` 中的 SHA-256 哈希与传递到浏览器的文件相匹配，完整性检查就会通过。

此失败的常见原因包括：

 * Web 服务器的响应是一个错误（例如，“404 - 找不到”或“500 - 内部服务器错误”），而不是浏览器所请求的文件。 浏览器会将其报告为完整性检查失败，而不是响应失败。
 * 在文件生成和传递到浏览器之间已更改文件的内容。 下面可能会发生这种情况：
   * 你或生成工具手动修改生成输出的情况。
   * 部署过程的某个方面修改了文件的情况。 例如，在使用基于 Git 的部署机制时，请记住，如果你在 Windows 上提交文件并在 Linux 上检查它们，则 Git 会以透明方式将 Windows 样式的行尾转换为 Unix 样式的行尾。 更改文件行尾将更改 SHA-256 哈希。 若要避免此问题，请考虑[使用 `.gitattributes` 将生成项目视为 `binary` 文件](https://git-scm.com/book/en/v2/Customizing-Git-Git-Attributes)。
   * Web 服务器在提供文件内容的过程中对其进行修改。 例如，某些内容分发网络 (CDN) 会自动尝试[缩小](xref:client-side/bundling-and-minification#minification) HTML，从而对其进行修改。 可能需要禁用此类功能。

若要诊断哪些功能适用于你的情况，请执行执行操作：

 1. 通过读取错误消息来记下哪个文件触发错误。
 1. 打开浏览器的开发人员工具，然后在“网络”选项卡中查找。如有必要，请重新加载页面以查看请求和响应的列表。 在该列表中查找触发错误的文件。
 1. 检查响应中的 HTTP 状态代码。 如果服务器返回除“200 - 正常”（或其他 2xx 状态代码）以外的任何内容，则需要诊断服务器端问题。 例如，状态代码 403 表示存在授权问题，而状态代码 500 表示服务器以未指定的方式失败。 请参阅服务器端日志以诊断和修复应用。
 1. 如果资源的状态代码为“200 - 正常”，请在浏览器的开发人员工具中查看响应内容，并检查内容是否与预期的数据匹配。 例如，常见问题是错误配置了路由，因此请求甚至返回其他文件的 `index.html` 数据。 请确保对 `.wasm` 请求的响应是 WebAssembly 二进制文件，对 `.dll` 请求的响应是 .NET 程序集二进制文件。 如果不是，则需要诊断服务器端路由问题。

如果确认服务器返回看似正确的数据，则必须在生成文件和传递文件之间修改内容。 若要对此进行调查，请执行以下操作：

 * 如果在生成文件后修改文件，请检查生成工具链和部署机制。 例如，在 Git 转换文件行尾时，如前所述。
 * 如设置为动态修改响应（例如，尝试缩小 HTML），请检查 Web 服务器或 CDN 配置。 Web 服务器可以实现 HTTP 压缩（例如，返回 `content-encoding: br` 或 `content-encoding: gzip`），因为这不会影响解压缩后的结果。 但是，Web 服务器不可以修改未压缩的数据。

### <a name="disable-integrity-checking-for-non-pwa-apps"></a>禁用非 PWA 应用的完整性检查

在大多数情况下，不要禁用完整性检查。 禁用完整性检查并不能解决导致意外响应的根本问题，并且会导致丢失前面列出的权益。

在某些情况下，Web 服务器无法用于返回一致的响应，但别无选择，只能禁用完整性检查。 若要禁用完整性检查，请将以下内容添加到 Blazor WebAssembly 项目的 `.csproj` 文件中的属性组：

```xml
<BlazorCacheBootResources>false</BlazorCacheBootResources>
```

`BlazorCacheBootResources` 还会根据 SHA-256 哈希禁用 Blazor 缓存 `.dll`、`.wasm` 和其他文件的默认行为，因为属性指示无法依靠 SHA-256 哈希来确保正确性。 即使有此设置，浏览器的普通 HTTP 缓存仍可能会缓存这些文件，但是否发生这种情况取决于你的 Web 服务器配置和它所提供的 `cache-control` 标头。

> [!NOTE]
> `BlazorCacheBootResources` 属性不会禁用[渐进式 Web 应用程序 (PWA)](xref:blazor/progressive-web-app) 的完整性检查。 有关 PWA 的相关指南，请参阅[禁用 PWA 的完整性检查](#disable-integrity-checking-for-pwas)部分。

### <a name="disable-integrity-checking-for-pwas"></a>禁用 PWA 的完整性检查

Blazor 的渐进式 Web 应用程序 (PWA) 模板包含建议的 `service-worker.published.js` 文件，该文件负责获取和存储应用程序文件以供脱机使用。 这是普通应用启动机制的独立进程，具有其自己单独的完整性检查逻辑。

在 `service-worker.published.js` 文件中，出现以下行：

```javascript
.map(asset => new Request(asset.url, { integrity: asset.hash }));
```

若要禁用完整性检查，请通过将该行更改为以下行来删除 `integrity` 参数：

```javascript
.map(asset => new Request(asset.url));
```

同样，禁用完整性检查意味着会丢失完整性检查提供的安全保证。 例如，如果用户的浏览器在你部署新版本的那一刻缓存应用，则会存在风险，它可能会缓存旧部署中的某些文件和新部署中的某些文件。 如果发生这种情况，则在部署进一步更新之前，应用程序会在中断状态下停滞。
