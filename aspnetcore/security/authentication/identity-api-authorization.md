---
title: ASP.NET Core 上的单页面应用的身份验证简介
author: javiercn
description: 用于 Identity ASP.NET Core 应用内托管的单页面应用。
monikerRange: '>= aspnetcore-3.0'
ms.author: scaddie
ms.custom: mvc
ms.date: 10/27/2020
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
uid: security/authentication/identity/spa
ms.openlocfilehash: 8acc34c88bf62b3da1b920acc7318c94435c100e
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051975"
---
# <a name="authentication-and-authorization-for-spas"></a>Spa 的身份验证和授权

ASP.NET Core 3.1 和更高版本模板在单页面应用中提供身份验证， (Spa) 使用对 API 授权的支持。 ASP.NET Core Identity对于身份验证和存储用户，将与用于实现 OpenID Connect 的[ Identity 服务器](https://identityserver.io/)结合。

已将身份验证参数添加到 " **角度** " 和 " **响应** " 项目模板，该模板类似于 Web 应用程序中的身份验证参数 **(模型-视图-控制器)** (MVC) 和 **web 应用** (Razor 页) 项目模板。 允许的参数值为 **None** 和 **个体** 。 **React.js 和 Redux** 项目模板此时不支持身份验证参数。

## <a name="create-an-app-with-api-authorization-support"></a>使用 API 授权支持创建应用

用户身份验证和授权可用于角度和响应 Spa。 打开命令 shell，并运行以下命令：

**角** ：

```dotnetcli
dotnet new angular -o <output_directory_name> -au Individual
```

**响应** ：

```dotnetcli
dotnet new react -o <output_directory_name> -au Individual
```

上述命令创建一个 ASP.NET Core 应用，其中包含包含 SPA 的 *ClientApp* 目录。

## <a name="general-description-of-the-aspnet-core-components-of-the-app"></a>应用的 ASP.NET Core 组件的一般描述

以下各节介绍了在包括身份验证支持的情况下对项目添加的内容：

### <a name="startup-class"></a>Startup 类

下面的代码示例依赖于 [AspNetCore. ApiAuthorization。 IdentityServer](https://www.nuget.org/packages/Microsoft.AspNetCore.ApiAuthorization.IdentityServer) NuGet 包。 示例使用 <xref:Microsoft.Extensions.DependencyInjection.IdentityServerBuilderConfigurationExtensions.AddApiAuthorization%2A> 和扩展方法配置 API 身份验证和授权 <xref:Microsoft.AspNetCore.ApiAuthorization.IdentityServer.ApiResourceCollection.AddIdentityServerJwt%2A> 。 使用带有身份验证的响应或角 SPA 项目模板的项目包括对此包的引用。

`Startup`类添加了以下内容：

* 在 `Startup.ConfigureServices` 方法中：
  * Identity 对于默认的 UI：

    ```csharp
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));

    services.AddDefaultIdentity<ApplicationUser>()
        .AddEntityFrameworkStores<ApplicationDbContext>();
    ```

  * Identity具有其他 `AddApiAuthorization` 帮助器方法的服务器，用于在服务器上设置某些默认 ASP.NET Core 约定 Identity ：

    ```csharp
    services.AddIdentityServer()
        .AddApiAuthorization<ApplicationUser, ApplicationDbContext>();
    ```

  * 包含附加 `AddIdentityServerJwt` 帮助器方法的身份验证，该方法将应用配置为验证由 IdentityServer 生成的 JWT 令牌：

    ```csharp
    services.AddAuthentication()
        .AddIdentityServerJwt();
    ```

* 在 `Startup.Configure` 方法中：
  * 负责验证请求凭据并在请求上下文上设置用户的身份验证中间件：

    ```csharp
    app.UseAuthentication();
    ```

  * Identity公开 OpenID connect 终结点的服务器中间件：

    ```csharp
    app.UseIdentityServer();
    ```

### <a name="addapiauthorization"></a>AddApiAuthorization

此帮助程序方法将 Identity 服务器配置为使用受支持的配置。 IdentityServer 是一个功能强大且可扩展的框架，用于处理应用安全问题。 同时，对于最常见的方案，这会造成不必要的复杂性。 因此，系统会为您提供一组约定和配置选项，这是一个很好的起点。 一旦你的身份验证需要更改，服务器的全部功能 Identity 仍可用于自定义身份验证以满足你的需求。

### <a name="addno-locidentityserverjwt"></a>AddIdentityServerJwt

此帮助器方法将应用程序的策略方案配置为默认的身份验证处理程序。 此策略配置为允许 Identity 处理路由到 Identity URL 空间 "/" 中任何子路径的所有请求 Identity 。 `JwtBearerHandler` 处理所有其他请求。 此外，此方法向服务器注册一个 `<<ApplicationName>>API` Identity 具有默认作用域的 API 资源 `<<ApplicationName>>API` ，并将 JWT 持有者令牌中间件配置为验证 Identity 服务器为应用程序颁发的令牌。

### <a name="weatherforecastcontroller"></a>WeatherForecastController

在 *Controllers\WeatherForecastController.cs* 文件中，请注意 `[Authorize]` 应用于类的属性，该属性指示用户需要根据默认策略进行授权才能访问资源。 默认授权策略将配置为使用默认的身份验证方案，该方案由设置 `AddIdentityServerJwt` 为上面提到的策略方案，并使此 `JwtBearerHandler` 类 helper 方法配置的应用程序请求的默认处理程序。

### <a name="applicationdbcontext"></a>ApplicationDbContext

在 *Data\ApplicationDbContext.cs* 文件中，请注意， `DbContext` 在中使用了相同的， Identity 因为它将扩展 `ApiAuthorizationDbContext` (一个派生程度更高的类 `IdentityDbContext`) 以包括服务器的架构 Identity 。

若要完全控制数据库架构，请从某个可用的类继承， Identity `DbContext` 并将上下文配置为 Identity 通过调用方法来包含架构 `builder.ConfigurePersistedGrantContext(_operationalStoreOptions.Value)` `OnModelCreating` 。

### <a name="oidcconfigurationcontroller"></a>OidcConfigurationController

在 *Controllers\OidcConfigurationController.cs* 文件中，请注意为提供客户端需要使用的 OIDC 参数而设置的终结点。

### appsettings.json

在 *appsettings.json* 项目根目录的文件中，有一个新 `IdentityServer` 部分描述了已配置的客户端的列表。 下例中存在一个客户端。 客户端名称对应于应用名称，并通过约定映射到 OAuth `ClientId` 参数。 配置文件指示正在配置的应用类型。 它在内部用于驱动约定，以简化服务器的配置过程。 有几个配置文件可用，如 " [应用程序配置文件](#application-profiles) " 部分中所述。

```json
"IdentityServer": {
  "Clients": {
    "angularindividualpreview3final": {
      "Profile": "IdentityServerSPA"
    }
  }
}
```

### <a name="appsettingsdevelopmentjson"></a>appsettings.Development.js

在项目根的 *appsettings.Development.js* 文件中，有一个 `IdentityServer` 描述用于对令牌进行签名的密钥的部分。 部署到生产环境时，需要在应用中预配和部署密钥，如 " [部署到生产](#deploy-to-production) " 一节中所述。

```json
"IdentityServer": {
  "Key": {
    "Type": "Development"
  }
}
```

## <a name="general-description-of-the-angular-app"></a>角应用的一般说明

角度模板中的身份验证和 API 授权支持位于其自身的 *ClientApp\src\api-authorization* 目录中。 模块由以下元素组成：

* 3个组件：
  * *login. ts* ：处理应用的登录流。
  * node.js：处理应用程序的注销 *流。*
  * *login-menu. ts* ：一个小组件，显示以下一组链接：
    * 用户进行身份验证时，用户配置文件管理和注销链接。
    * 用户未通过身份验证时的注册和登录链接。
* `AuthorizeGuard`可以添加到路由中的路由防护，要求先对用户进行身份验证，然后才能访问路由。
* `AuthorizeInterceptor`向用户进行身份验证时，将访问令牌附加到针对 API 的传出 HTTP 请求的 http 侦听器。
* 一 `AuthorizeService` 项服务，用于处理身份验证过程的较低级别详细信息，并向应用程序的其余部分提供有关使用情况的已通过身份验证的用户的信息。
* 用于定义与应用的身份验证部分关联的路由的角模块。 它公开登录菜单组件、侦听器、防护和服务，以便从应用程序的其余部分使用。

## <a name="general-description-of-the-react-app"></a>响应应用的一般说明

响应模板中的身份验证和 API 授权支持位于 *ClientApp\src\components\api-authorization* 目录中。 它由以下元素组成：

* 4个组件：
  * *Login.js* ：处理应用的登录流。
  * *Logout.js* ：处理应用程序的注销流。
  * *LoginMenu.js* ：显示以下链接集之一的小组件：
    * 用户进行身份验证时，用户配置文件管理和注销链接。
    * 用户未通过身份验证时的注册和登录链接。
  * *AuthorizeRoute.js* ：路由组件，需要先对用户进行身份验证，然后才能呈现参数中指示的组件 `Component` 。
* `authService`类的导出实例 `AuthorizeService` ，用于处理身份验证过程的较低级别细节，并向应用程序的其余部分提供有关使用情况的已通过身份验证的用户的信息。

现在，你已了解解决方案的主要组件，可以更深入地了解应用程序的各个方案。

## <a name="require-authorization-on-a-new-api"></a>要求对新 API 进行授权

默认情况下，系统配置为轻松地要求对新 Api 进行授权。 为此，请创建新的控制器，将 `[Authorize]` 属性添加到控制器类或控制器中的任何操作。

## <a name="customize-the-api-authentication-handler"></a>自定义 API 身份验证处理程序

若要自定义 API 的 JWT 处理程序的配置，请配置其 <xref:Microsoft.AspNetCore.Builder.JwtBearerOptions> 实例：

```csharp
services.AddAuthentication()
    .AddIdentityServerJwt();

services.Configure<JwtBearerOptions>(
    IdentityServerJwtConstants.IdentityServerJwtBearerScheme,
    options =>
    {
        ...
    });
```

API 的 JWT 处理程序会引发事件，这些事件可以使用来控制身份验证过程 `JwtBearerEvents` 。 若要为 API 授权提供支持，请 `AddIdentityServerJwt` 注册其自己的事件处理程序。

若要自定义事件的处理，请根据需要使用其他逻辑来包装现有的事件处理程序。 例如： 。

```csharp
services.Configure<JwtBearerOptions>(
    IdentityServerJwtConstants.IdentityServerJwtBearerScheme,
    options =>
    {
        var onTokenValidated = options.Events.OnTokenValidated;       
        
        options.Events.OnTokenValidated = async context =>
        {
            await onTokenValidated(context);
            ...
        }
    });
```

在前面的代码中， `OnTokenValidated` 事件处理程序将替换为自定义实现。 此实现：

1. 调用 API 授权支持提供的原始实现。
1. 运行自己的自定义逻辑。

## <a name="protect-a-client-side-route-angular"></a>保护客户端路由 (角度) 

通过将授权防护添加到在配置路由时要运行的防护列表，来保护客户端路由。 例如，可以在 `fetch-data` 主应用角模块内查看路由的配置方式：

```typescript
RouterModule.forRoot([
  // ...
  { path: 'fetch-data', component: FetchDataComponent, canActivate: [AuthorizeGuard] },
])
```

需要说明的是，保护路由不会保护实际端点 (仍需要 `[Authorize]` 应用属性) 但它只会阻止用户在未通过身份验证时导航到给定的客户端路由。

## <a name="authenticate-api-requests-angular"></a> (角度) 对 API 请求进行身份验证

对与应用程序一起托管的 Api 的请求进行身份验证是通过使用应用定义的 HTTP 客户端侦听器来自动完成的。

## <a name="protect-a-client-side-route-react"></a>保护客户端路由 (响应) 

使用 `AuthorizeRoute` 组件而不是普通组件来保护客户端路由 `Route` 。 例如，请注意该 `fetch-data` 路由在组件中的配置方式 `App` ：

```jsx
<AuthorizeRoute path='/fetch-data' component={FetchData} />
```

保护路由：

* 不保护实际端点 (仍需要将 `[Authorize]` 属性应用于) 。
* 仅阻止用户在未通过身份验证时导航到给定的客户端路由。

## <a name="authenticate-api-requests-react"></a>对 API 请求进行身份验证 (响应) 

通过从中导入实例，对具有反应的请求进行身份验证 `authService` `AuthorizeService` 。 访问令牌从检索 `authService` ，并附加到请求，如下所示。 在做出反应的组件中，这种工作通常是在生命周期方法中完成的， `componentDidMount` 或者是来自某些用户交互的结果。

### <a name="import-the-authservice-into-your-component"></a>将 authService 导入组件

```javascript
import authService from './api-authorization/AuthorizeService'
```

### <a name="retrieve-and-attach-the-access-token-to-the-response"></a>检索访问令牌并将其附加到响应

```javascript
async populateWeatherData() {
  const token = await authService.getAccessToken();
  const response = await fetch('api/SampleData/WeatherForecasts', {
    headers: !token ? {} : { 'Authorization': `Bearer ${token}` }
  });
  const data = await response.json();
  this.setState({ forecasts: data, loading: false });
}
```

## <a name="deploy-to-production"></a>部署到生产

若要将应用部署到生产环境，需预配以下资源：

* 用于存储 Identity 用户帐户和服务器授予的数据库 Identity 。
* 用于对令牌进行签名的生产证书。
  * 此证书没有特定要求;此证书可以是自签名证书，也可以是通过 CA 颁发机构预配的证书。
  * 它可通过 PowerShell 或 OpenSSL 等标准工具生成。
  * 可以将其安装到目标计算机上的证书存储中，或部署为带有强密码的 *.pfx* 文件。

### <a name="example-deploy-to-a-non-azure-web-hosting-provider"></a>示例：部署到非 Azure web 托管提供程序

在 web 托管面板中，创建或加载证书。 然后，在应用的 *appsettings.json* 文件中修改 `IdentityServer` 部分以包含关键详细信息。 例如： 。

```json
"IdentityServer": {
  "Key": {
    "Type": "Store",
    "StoreName": "WebHosting",
    "StoreLocation": "CurrentUser",
    "Name": "CN=MyApplication"
  }
}
```

在上面的示例中：

* `StoreName` 表示存储证书的证书存储区的名称。 在这种情况下，它指向 web 宿主存储区。
* `StoreLocation` 表示 `CurrentUser` 在这种情况下从 (中加载证书的位置) 。
* `Name` 对应于证书的可分辨主题。

### <a name="example-deploy-to-azure-app-service"></a>示例：部署到 Azure App Service

本部分介绍如何使用存储在证书存储区中的证书，将应用部署到 Azure App Service。 若要将应用修改为从证书存储区中加载证书，请在稍后的步骤中配置 Azure 门户应用时，需要使用标准层服务计划或更高版本。

在应用的 *appsettings.json* 文件中，修改 `IdentityServer` 部分以包含关键详细信息：

```json
"IdentityServer": {
  "Key": {
    "Type": "Store",
    "StoreName": "My",
    "StoreLocation": "CurrentUser",
    "Name": "CN=MyApplication"
  }
}
```

* 存储名称表示存储证书的证书存储区的名称。 在这种情况下，它指向个人用户存储区。
* 应用商店位置表示从 (或) 中加载证书的位置 `CurrentUser` `LocalMachine` 。
* 证书上的名称属性对应于证书的可分辨主题。

若要部署到 Azure App Service，请按照将 [应用程序部署到 Azure](xref:tutorials/publish-to-azure-webapp-using-vs#deploy-the-app-to-azure)中的步骤操作，该步骤介绍了如何创建必要的 Azure 资源，以及如何将应用程序部署到生产环境。

按照前面的说明操作后，应用程序将部署到 Azure，但尚不起作用。 应用使用的证书必须在 Azure 门户中进行配置。 找到证书的指纹，并按照 [加载证书](/azure/app-service/app-service-web-ssl-cert-load#load-the-certificate-in-code)中所述的步骤进行操作。

虽然这些步骤提及 SSL，但在 Azure 门户中有一个 " **私有证书** " 部分，你可以在其中上传预配的证书以与应用一起使用。

在 Azure 门户中配置应用和应用设置后，请在门户中重新启动应用。

## <a name="other-configuration-options"></a>其他配置选项

支持 API 授权构建在 Identity 服务器之上，具有一组约定、默认值和增强功能，可简化 spa 的体验。 不用说， Identity 如果 ASP.NET Core 集成不涵盖方案，则可以在幕后使用服务器的完整功能。 ASP.NET Core 支持重点介绍 "第一方" 应用，其中的所有应用都是由我们的组织创建和部署的。 因此，不支持许可或联合等。 对于这些方案，请使用 Identity 服务器并按照其文档进行操作。

### <a name="application-profiles"></a>应用程序配置文件

应用程序配置文件是用于进一步定义其参数的应用的预定义配置。 目前支持以下配置文件：

* `IdentityServerSPA`：表示 Identity 作为一个单元与服务器一起托管的 SPA。
  * `redirect_uri`默认值为 `/authentication/login-callback` 。
  * `post_logout_redirect_uri`默认值为 `/authentication/logout-callback` 。
  * 作用域集包括 `openid` `profile` 为应用中的 api 定义的、和每个作用域。
  * 一组允许的 OIDC 响应类型为 `id_token token` ，每个响应类型分别 `id_token` (`token`) 。
  * 允许的响应模式为 `fragment` 。
* `SPA`：表示不随服务器承载的 SPA Identity 。
  * 作用域集包括 `openid` `profile` 为应用中的 api 定义的、和每个作用域。
  * 一组允许的 OIDC 响应类型为 `id_token token` ，每个响应类型分别 `id_token` (`token`) 。
  * 允许的响应模式为 `fragment` 。
* `IdentityServerJwt`：表示与服务器一起托管的 API Identity 。
  * 应用配置为具有一个默认为应用名称的作用域。
* `API`：表示不与服务器托管的 API Identity 。
  * 应用配置为具有一个默认为应用名称的作用域。

### <a name="configuration-through-appsettings"></a>通过 AppSettings 配置

通过将应用添加到或列表，通过配置系统配置应用 `Clients` `Resources` 。

配置每个客户端的 `redirect_uri` 和 `post_logout_redirect_uri` 属性，如以下示例中所示：

```json
"IdentityServer": {
  "Clients": {
    "MySPA": {
      "Profile": "SPA",
      "RedirectUri": "https://www.example.com/authentication/login-callback",
      "LogoutUri": "https://www.example.com/authentication/logout-callback"
    }
  }
}
```

配置资源时，可以按如下所示配置资源的作用域：

```json
"IdentityServer": {
  "Resources": {
    "MyExternalApi": {
      "Profile": "API",
      "Scopes": "a b c"
    }
  }
}
```

### <a name="configuration-through-code"></a>通过代码进行配置

你还可以通过代码使用的重载来配置客户端和资源 `AddApiAuthorization` ，该重载采用操作来配置选项。

```csharp
AddApiAuthorization<ApplicationUser, ApplicationDbContext>(options =>
{
    options.Clients.AddSPA(
        "My SPA", spa =>
        spa.WithRedirectUri("http://www.example.com/authentication/login-callback")
           .WithLogoutRedirectUri(
               "http://www.example.com/authentication/logout-callback"));

    options.ApiResources.AddApiResource("MyExternalApi", resource =>
        resource.WithScopes("a", "b", "c"));
});
```

## <a name="additional-resources"></a>其他资源

* <xref:spa/angular>
* <xref:spa/react>
* <xref:security/authentication/scaffold-identity>
