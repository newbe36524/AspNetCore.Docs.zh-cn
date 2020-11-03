---
title: IdentityASP.NET Core 项目中的基架
author: rick-anderson
description: 了解如何 Identity 在 ASP.NET Core 项目中进行基架。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 5/1/2020
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
uid: security/authentication/scaffold-identity
ms.openlocfilehash: 813dd7837c265c78c584d66dd51bc23399d12fbe
ms.sourcegitcommit: 5156eab2118584405eb663e1fcd82f8bd7764504
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/31/2020
ms.locfileid: "93141490"
---
# <a name="scaffold-no-locidentity-in-aspnet-core-projects"></a>IdentityASP.NET Core 项目中的基架

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core 提供 [ASP.NET Core Identity](xref:security/authentication/identity) 为类库[ Razor ](xref:razor-pages/ui-class)。 包含的应用程序 Identity 可以应用 scaffolder 将类库中包含的源代码选择性地添加 Identity Razor (RCL) 。 建议生成源代码，以便修改代码和更改行为。 例如，可以指示基架生成在注册过程中使用的代码。 生成的代码优先于 Identity RCL 中的相同代码。 若要完全控制 UI，而不使用默认的 RCL，请参阅 [创建完整 Identity UI 源](#full)部分。

**不** 包含身份验证的应用程序可以应用 scaffolder 来添加 RCL Identity 包。 可以选择要生成的 Identity 代码。

尽管 scaffolder 生成了大部分必要的代码，但你需要更新项目以完成该过程。 本文档介绍完成基架更新所需的步骤 Identity 。

建议使用显示文件差异的源代码管理系统，并使您能够回退更改。 运行 scaffolder 后检查更改 Identity 。

使用 [双重身份验证](xref:security/authentication/identity-enable-qrcodes)、 [帐户确认和密码恢复](xref:security/authentication/accconfirm)和其他安全功能时，需要提供服务 Identity 。 基架时不生成服务或服务存根 Identity 。 要启用这些功能，必须手动添加服务。 例如，请参阅 [需要确认电子邮件](xref:security/authentication/accconfirm#require-email-confirmation)。

当 Identity 使用现有个人帐户将具有新数据上下文的基架插入到项目中时：

* 在中 `Startup.ConfigureServices` ，删除对的调用：
  * `AddDbContext`
  * `AddDefaultIdentity`

例如， `AddDbContext` `AddDefaultIdentity` 在以下代码中注释掉和：

[!code-csharp[](scaffold-identity/3.1sample/StartupRemove.cs?name=snippet)]

前面的代码注释了 *区域/ Identity / Identity HostingStartup.cs* 中的重复代码。

通常，使用个人帐户创建的应用应 * **not** _ 创建新的数据上下文。

## <a name="scaffold-no-locidentity-into-an-empty-project"></a>基架 Identity 到空项目

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

`Startup`用类似于下面的代码更新类：

[!code-csharp[](scaffold-identity/3.1sample/StartupMVC.cs?name=snippet)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

## <a name="scaffold-no-locidentity-into-a-no-locrazor-project-without-existing-authorization"></a>基架 Identity 到 Razor 无现有授权的项目

<!--  Updated for 3.0
set projNam=RPnoAuth
set projType=webapp

dotnet new %projType% -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet aspnet-codegenerator identity --useDefaultUI
dotnet ef database drop
dotnet ef migrations add CreateIdentitySchema0
dotnet ef database update
-->

<!-- ERROR
There is already an object named 'AspNetRoles' in the database.

Fixed via dotnet ef database drop
before dotnet ef database update
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

Identity在 _Areas/HostingStartup.cs * 中进行配置 Identity / Identity 。 有关详细信息，请参阅 [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration)。

<a name="efm"></a>

### <a name="migrations-useauthentication-and-layout"></a>迁移、UseAuthentication 和布局

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

<a name="useauthentication"></a>

### <a name="enable-authentication"></a>启用身份验证

`Startup`用类似于下面的代码更新类：

[!code-csharp[](scaffold-identity/3.1sample/StartupRP.cs?name=snippet)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

### <a name="layout-changes"></a>布局更改

可选：将登录部分 (`_LoginPartial`) 添加到布局文件中：

[!code-cshtml[](scaffold-identity/3.1sample/_Layout.cshtml?highlight=20)]

## <a name="scaffold-no-locidentity-into-a-no-locrazor-project-with-authorization"></a>Identity Razor 使用授权基架到项目中

<!--
Use >=2.1: dotnet new webapp -au Individual -o RPauth
Use = 2.0: dotnet new razor -au Individual -o RPauth
uld option: Use Local DB, not SQLite

dotnet new webapp -au Individual -uld -o RPauth
cd RPauth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files "Account.Register;Account.Register"
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]

某些 Identity 选项在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration)。

## <a name="scaffold-no-locidentity-into-an-mvc-project-without-existing-authorization"></a>基架 Identity 到没有现有授权的 MVC 项目

<!--
set projNam=MvcNoAuth
set projType=mvc
set version=2.1.0

dotnet new %projType% -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design -v %version%
dotnet restore
dotnet aspnet-codegenerator identity --useDefaultUI
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

可选：将登录名部分 (`_LoginPartial`) 添加到 *Views/Shared/_Layout* 文件中：

[!code-cshtml[](scaffold-identity/3.1sample/_Layout.cshtml?highlight=20)]

* 将 *Pages/shared/_LoginPartial cshtml* 文件移动到 *Views/shared/_LoginPartial。 cshtml*

Identity在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 IHostingStartup。

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

`Startup`用类似于下面的代码更新类：

[!code-csharp[](scaffold-identity/3.1sample/StartupMVC.cs?name=snippet)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

## <a name="scaffold-no-locidentity-into-an-mvc-project-with-authorization"></a>Identity使用授权基架到 MVC 项目

<!--
dotnet new mvc -au Individual -o MvcAuth
cd MvcAuth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
dotnet aspnet-codegenerator identity -dc MvcAuth.Data.ApplicationDbContext  --files "Account.Login;Account.Register"
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]

## <a name="scaffold-no-locidentity-into-a-no-locblazor-server-project-without-existing-authorization"></a>基架 Identity 到 Blazor Server 无现有授权的项目

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

Identity在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration)。

### <a name="migrations"></a>迁移

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

### <a name="pass-an-xsrf-token-to-the-app"></a>将 XSRF 令牌传递给应用

可以将令牌传递给组件：

* 设置身份验证令牌并将其保存到身份验证后 cookie ，可以将其传递给组件。
* Razor 组件不能 `HttpContext` 直接使用，因此无法获取 [ (XSRF 的) 令牌的反请求伪造](xref:security/anti-request-forgery) Identity `/Identity/Account/Logout` 。 可以将 XSRF 令牌传递给组件。

有关详细信息，请参阅 <xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app>。

在 *Pages/_Host cshtml* 文件中，在将其添加到和类后建立该令牌 `InitialApplicationState` `TokenProvider` ：

```csharp
@inject Microsoft.AspNetCore.Antiforgery.IAntiforgery Xsrf

...

var tokens = new InitialApplicationState
{
    ...

    XsrfToken = Xsrf.GetAndStoreTokens(HttpContext).RequestToken
};
```

`App`将组件更新 ( *App.razor* ) ，以分配 `InitialState.XsrfToken` ：

```csharp
@inject TokenProvider TokenProvider

...

TokenProvider.XsrfToken = InitialState.XsrfToken;
```

本 `TokenProvider` 主题中演示的服务在 `LoginDisplay` 以下 [布局和身份验证流更改](#layout-and-authentication-flow-changes) 部分的组件中使用。

### <a name="enable-authentication"></a>启用身份验证

在 `Startup` 类中：

* 确认 Razor 在中添加了页面服务 `Startup.ConfigureServices` 。
* 如果使用 [TokenProvider](xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app)，请注册服务。
* `UseDatabaseErrorPage`对于开发环境，请在中的应用程序生成器上调用 `Startup.Configure` 。
* 调用 `UseAuthentication` and `UseAuthorization` after `UseRouting` 。
* 添加页的终结点 Razor 。

[!code-csharp[](scaffold-identity/3.1sample/StartupBlazor.cs?highlight=3,6,14,27-28,32)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

### <a name="layout-and-authentication-flow-changes"></a>布局和身份验证流更改

将 `RedirectToLogin` *RedirectToLogin* ) 的 (组件添加到项目根目录中应用的 *共享* 文件夹中：

```razor
@inject NavigationManager Navigation
@code {
    protected override void OnInitialized()
    {
        Navigation.NavigateTo("Identity/Account/Login?returnUrl=" +
            Uri.EscapeDataString(Navigation.Uri), true);
    }
}
```

将 `LoginDisplay` () 的 *LoginDisplay.razor* 组件添加到应用的 *共享* 文件夹。 [TokenProvider 服务](xref:blazor/security/server/additional-scenarios#pass-tokens-to-a-blazor-server-app)提供 HTML 窗体的 XSRF 标记，该标记将发布到 Identity 注销终结点：

```razor
@using Microsoft.AspNetCore.Components.Authorization
@inject NavigationManager Navigation
@inject TokenProvider TokenProvider

<AuthorizeView>
    <Authorized>
        <a href="Identity/Account/Manage/Index">
            Hello, @context.User.Identity.Name!
        </a>
        <form action="/Identity/Account/Logout?returnUrl=%2F" method="post">
            <button class="nav-link btn btn-link" type="submit">Logout</button>
            <input name="__RequestVerificationToken" type="hidden" 
                value="@TokenProvider.XsrfToken">
        </form>
    </Authorized>
    <NotAuthorized>
        <a href="Identity/Account/Register">Register</a>
        <a href="Identity/Account/Login">Login</a>
    </NotAuthorized>
</AuthorizeView>
```

在 `MainLayout` ( *Shared/MainLayout* ) 的组件中，将组件添加 `LoginDisplay` 到顶层行 `<div>` 元素的内容：

```razor
<div class="top-row px-4 auth">
    <LoginDisplay />
    <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
</div>
```

### <a name="style-authentication-endpoints"></a>样式身份验证终结点

由于 Blazor Server 使用 Razor 页面 Identity 页，当访问者在页面和组件间导航时，UI 的样式会发生变化 Identity 。 可以使用两个选项来处理 incongruous 样式：

#### <a name="build-no-locidentity-components"></a>生成 Identity 组件

使用而不是页的组件的方法 Identity 是生成 Identity 组件。 由于 `SignInManager` `UserManager` 在组件中不支持和 Razor ，因此在应用程序中使用 API 终结点 Blazor Server 来处理用户帐户操作。

#### <a name="use-a-custom-layout-with-no-locblazor-app-styles"></a>在应用程序样式中使用自定义布局 Blazor

Identity可以修改页面布局和样式，以生成使用默认主题的页面 Blazor 。

> [!NOTE]
> 本节中的示例只是一个自定义的起点。 为了获得最佳用户体验，可能需要额外的工作。

`NavMenu_IdentityLayout` ( *Shared/NavMenu_ Identity Layout* ) 创建新的组件。 对于组件的标记和代码，请 `NavMenu` ( *Shared/NavMenu* ) 使用应用组件的相同内容。 去除 `NavLink` 无法匿名访问的组件，因为组件中的自动重定向 `RedirectToLogin` 对于需要身份验证或授权的组件失败。

在 *Pages/Shared/Layout* 文件中，进行以下更改：

* 将 Razor 指令添加到文件顶部，以使用 *共享* 文件夹中的标记帮助程序和应用组件：

  ```cshtml
  @addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
  @using {APPLICATION ASSEMBLY}.Shared
  ```

  替换 `{APPLICATION ASSEMBLY}` 为应用程序的程序集名称。

* 向 `<base>` 内容添加标记和 Blazor 样式 `<link>` 表 `<head>` ：

  ```cshtml
  <base href="~/" />
  <link rel="stylesheet" href="~/css/site.css" />
  ```

* 将标记的内容更改 `<body>` 为以下内容：

  ```cshtml
  <div class="sidebar" style="float:left">
      <component type="typeof(NavMenu_IdentityLayout)" 
          render-mode="ServerPrerendered" />
  </div>

  <div class="main" style="padding-left:250px">
      <div class="top-row px-4">
          @{
              var result = Engine.FindView(ViewContext, "_LoginPartial", 
                  isMainPage: false);
          }
          @if (result.Success)
          {
              await Html.RenderPartialAsync("_LoginPartial");
          }
          else
          {
              throw new InvalidOperationException("The default Identity UI " +
                  "layout requires a partial view '_LoginPartial'.");
          }
          <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
      </div>

      <div class="content px-4">
          @RenderBody()
      </div>
  </div>

  <script src="~/Identity/lib/jquery/dist/jquery.min.js"></script>
  <script src="~/Identity/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
  <script src="~/Identity/js/site.js" asp-append-version="true"></script>
  @RenderSection("Scripts", required: false)
  <script src="_framework/blazor.server.js"></script>
  ```

## <a name="scaffold-no-locidentity-into-a-no-locblazor-server-project-with-authorization"></a>Identity Blazor Server 使用授权基架到项目中

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]

某些 Identity 选项在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration)。

## <a name="standalone-or-hosted-no-locblazor-webassembly-apps"></a>独立应用或托管 Blazor WebAssembly 应用

客户端 Blazor WebAssembly 应用使用其自己的 Identity UI 方法，不能使用 ASP.NET Core Identity 基架。 托管解决方案的服务器端 ASP.NET Core 应用 Blazor 可以遵循 Razor 本文中的页面/MVC 指南，并配置为与支持的任何其他类型的 ASP.NET Core 应用一样 Identity 。

此 Blazor 框架不包括 Razor UI 页面的组件版本 Identity 。 Identity UI Razor 组件可以是自定义生成的，也可以从不受支持的第三方源获取。

有关详细信息，请参阅[ Blazor 安全性和 Identity 文章](xref:blazor/security/index)。

<a name="full"></a>

## <a name="create-full-no-locidentity-ui-source"></a>创建完整的 Identity UI 源

若要维护 UI 的完全控制 Identity ，请运行 Identity scaffolder 并选择 " **替代所有文件** "。

以下突出显示的代码显示了将默认 UI 替换为 Identity Identity ASP.NET Core 2.1 web 应用中的更改。 你可能希望执行此操作以对 UI 具有完全控制 Identity 。

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet1&highlight=13-14,17-999)]

Identity在以下代码中，将替换默认值：

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet2)]

下面的代码将设置 [LoginPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.loginpath)、 [LogoutPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.logoutpath)和 [AccessDeniedPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.accessdeniedpath)：

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet3)]

注册 `IEmailSender` 实现，例如：

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet4)]

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet)]

<!--
uld option: Use Local DB, not SQLite

dotnet new webapp -au Individual -uld -o RPauth
cd RPauth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.RegisterConfirmation"
-->

## <a name="password-configuration"></a>密码配置

如果 <xref:Microsoft.AspNetCore.Identity.PasswordOptions> 在中配置了 `Startup.ConfigureServices` ，则基架页中的属性可能需要[ `[StringLength]` 属性](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute)配置 `Password` Identity 。 `InputModel``Password`在以下文件中可以找到属性：

* `Areas/Identity/Pages/Account/Register.cshtml.cs`
* `Areas/Identity/Pages/Account/ResetPassword.cshtml.cs`

## <a name="disable-a-page"></a>禁用页面

本部分介绍如何禁用注册页面，但该方法可用于禁用任何页面。

禁用用户注册：

* 基架 Identity 。 包括帐户. Register、RegisterConfirmation。 例如：

  ```dotnetcli
   dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.RegisterConfirmation"
  ```

* 更新 *区域/ Identity /Pages/Account/Register.cshtml.cs* ，使用户无法从此终结点注册：

  [!code-csharp[](scaffold-identity/sample/Register.cshtml.cs?name=snippet)]

* 更新 *区域/ Identity /Pages/Account/Register.cshtml* ，使其与前面的更改一致：

  [!code-cshtml[](scaffold-identity/sample/Register.cshtml)]

* 注释掉或删除 *区域/ Identity /Pages/Account/Login.cshtml* 中的注册链接

  ```cshtml
  @*
  <p>
      <a asp-page="./Register" asp-route-returnUrl="@Model.ReturnUrl">Register as a new user</a>
  </p>
  *@
  ```

* 更新 " *区域/ Identity /Pages/Account/RegisterConfirmation* " 页。

  * 删除来自 cshtml 文件的代码和链接。
  * 从中删除确认代码 `PageModel` ：

  ```csharp
   [AllowAnonymous]
    public class RegisterConfirmationModel : PageModel
    {
        public IActionResult OnGet()
        {  
            return Page();
        }
    }
  ```
  
### <a name="use-another-app-to-add-users"></a>使用其他应用添加用户

提供一种在 web 应用外部添加用户的机制。 用于添加用户的选项包括：

* 专用的管理 web 应用。
* 控制台应用。

下面的代码概述了一种添加用户的方法：

* 用户列表将读入内存中。
* 为每个用户生成一个强唯一密码。
* 用户已添加到 Identity 数据库。
* 系统会通知用户并通知用户更改密码。

[!code-csharp[](scaffold-identity/consoleAddUser/Program.cs?name=snippet)]

下面的代码概述了如何添加用户：

[!code-csharp[](scaffold-identity/consoleAddUser/Data/SeedData.cs?name=snippet)]

对于生产方案，可以遵循类似的方法。

## <a name="prevent-publish-of-static-no-locidentity-assets"></a>禁止发布静态 Identity 资产

若要防止 Identity 将静态资产发布到 web 根目录，请参阅 <xref:security/authentication/identity#prevent-publish-of-static-identity-assets> 。

## <a name="additional-resources"></a>其他资源

* [更改为 ASP.NET Core 2.1 及更高版本的身份验证代码](xref:migration/20_21#changes-to-authentication-code)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core 2.1 和更高版本提供 [ASP.NET Core Identity](xref:security/authentication/identity) 为[ Razor 类库](xref:razor-pages/ui-class)。 包含的应用程序 Identity 可以应用 scaffolder 将类库中包含的源代码选择性地添加 Identity Razor (RCL) 。 建议生成源代码，以便修改代码和更改行为。 例如，可以指示基架生成在注册过程中使用的代码。 生成的代码优先于 Identity RCL 中的相同代码。 若要完全控制 UI，而不使用默认的 RCL，请参阅 [创建完全标识 UI 源](#full)部分。

**不** 包含身份验证的应用程序可以应用 scaffolder 来添加 RCL Identity 包。 可以选择要生成的 Identity 代码。

尽管 scaffolder 生成了大部分必要的代码，但你必须更新项目才能完成此过程。 本文档介绍完成基架更新所需的步骤 Identity 。

当 Identity scaffolder 运行时，将在项目目录中创建一个 *ScaffoldingReadme.txt* 文件。 *ScaffoldingReadme.txt* 文件包含有关完成基架更新所需内容的一般说明 Identity 。 本文档包含与 *ScaffoldingReadme.txt* 文件更完整的说明。

建议使用显示文件差异的源代码管理系统，并使您能够回退更改。 运行 scaffolder 后检查更改 Identity 。

> [!NOTE]
> 使用 [双重身份验证](xref:security/authentication/identity-enable-qrcodes)、 [帐户确认和密码恢复](xref:security/authentication/accconfirm)和其他安全功能时，需要提供服务 Identity 。 基架时不生成服务或服务存根 Identity 。 要启用这些功能，必须手动添加服务。 例如，请参阅 [需要确认电子邮件](xref:security/authentication/accconfirm#require-email-confirmation)。

## <a name="scaffold-no-locidentity-into-an-empty-project"></a>基架 Identity 到空项目

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

将以下突出显示的调用添加到 `Startup` 类：

[!code-csharp[](scaffold-identity/sample/StartupEmpty.cs?name=snippet1&highlight=5,20-23)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

## <a name="scaffold-no-locidentity-into-a-no-locrazor-project-without-existing-authorization"></a>基架 Identity 到 Razor 无现有授权的项目

<!--  Updated for 3.0
set projNam=RPnoAuth
set projType=webapp

dotnet new %projType% -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.AspNetCore.Identity.UI
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet restore
dotnet aspnet-codegenerator identity --useDefaultUI
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

Identity在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration)。

<a name="efm"></a>

### <a name="migrations-useauthentication-and-layout"></a>迁移、UseAuthentication 和布局

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

<a name="useauthentication"></a>

### <a name="enable-authentication"></a>启用身份验证

在 `Configure` 类的方法中 `Startup` ，在以下情况下调用 <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A> `UseStaticFiles` ：

[!code-csharp[](scaffold-identity/sample/StartupRPnoAuth.cs?name=snippet1&highlight=29)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

### <a name="layout-changes"></a>布局更改

可选：将登录部分 (`_LoginPartial`) 添加到布局文件中：

[!code-cshtml[](scaffold-identity/sample/_Layout.cshtml?highlight=37)]

## <a name="scaffold-no-locidentity-into-a-no-locrazor-project-with-authorization"></a>Identity Razor 使用授权基架到项目中

<!--
Use >=2.1: dotnet new webapp -au Individual -o RPauth
Use = 2.0: dotnet new razor -au Individual -o RPauth
uld option: Use Local DB, not SQLite

dotnet new webapp -au Individual -uld -o RPauth
cd RPauth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files "Account.Register;Account.Register"
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]

某些 Identity 选项在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 [IHostingStartup](xref:fundamentals/configuration/platform-specific-configuration)。

## <a name="scaffold-no-locidentity-into-an-mvc-project-without-existing-authorization"></a>基架 Identity 到没有现有授权的 MVC 项目

<!--
set projNam=MvcNoAuth
set projType=mvc
set version=2.1.0

dotnet new %projType% -o %projNam%
cd %projNam%
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design -v %version%
dotnet restore
dotnet aspnet-codegenerator identity --useDefaultUI
dotnet ef migrations add CreateIdentitySchema
dotnet ef database update
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg.md)]

可选：将登录名部分 (`_LoginPartial`) 添加到 *Views/Shared/_Layout* 文件中：

[!code-cshtml[](scaffold-identity/sample/_LayoutMvc.cshtml?highlight=37)]

* 将 *Pages/shared/_LoginPartial cshtml* 文件移动到 *Views/shared/_LoginPartial。 cshtml*

Identity在 *区域/ Identity / Identity HostingStartup.cs* 中配置。 有关详细信息，请参阅 IHostingStartup。

[!INCLUDE[](~/includes/scaffold-identity/migrations.md)]

调用 <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication%2A> 后 `UseStaticFiles` ：

[!code-csharp[](scaffold-identity/sample/StartupMvcNoAuth.cs?name=snippet1&highlight=23)]

[!INCLUDE[](~/includes/scaffold-identity/hsts.md)]

## <a name="scaffold-no-locidentity-into-an-mvc-project-with-authorization"></a>Identity使用授权基架到 MVC 项目

<!--
dotnet new mvc -au Individual -o MvcAuth
cd MvcAuth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
dotnet aspnet-codegenerator identity -dc MvcAuth.Data.ApplicationDbContext  --files "Account.Login;Account.Register"
-->

[!INCLUDE[](~/includes/scaffold-identity/id-scaffold-dlg-auth.md)]

删除 *页面/共享* 文件夹和该文件夹中的文件。

<a name="full"></a>

## <a name="create-full-no-locidentity-ui-source"></a>创建完整的 Identity UI 源

若要维护 UI 的完全控制 Identity ，请运行 Identity scaffolder 并选择 " **替代所有文件** "。

以下突出显示的代码显示了将默认 UI 替换为 Identity Identity ASP.NET Core 2.1 web 应用中的更改。 你可能希望执行此操作以对 UI 具有完全控制 Identity 。

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet1&highlight=13-14,17-999)]

Identity在以下代码中，将替换默认值：

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet2)]

下面的代码将设置 [LoginPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.loginpath)、 [LogoutPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.logoutpath)和 [AccessDeniedPath](/dotnet/api/microsoft.aspnetcore.authentication.cookies.cookieauthenticationoptions.accessdeniedpath)：

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet3)]

注册 `IEmailSender` 实现，例如：

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet4)]

[!code-csharp[](scaffold-identity/sample/StartupFull.cs?name=snippet)]

<!--
uld option: Use Local DB, not SQLite

dotnet new webapp -au Individual -uld -o RPauth
cd RPauth
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.RegisterConfirmation"
-->

## <a name="password-configuration"></a>密码配置

如果 <xref:Microsoft.AspNetCore.Identity.PasswordOptions> 在中配置了 `Startup.ConfigureServices` ，则基架页中的属性可能需要[ `[StringLength]` 属性](xref:System.ComponentModel.DataAnnotations.StringLengthAttribute)配置 `Password` Identity 。 `InputModel``Password`在以下文件中可以找到属性：

* `Areas/Identity/Pages/Account/Register.cshtml.cs`
* `Areas/Identity/Pages/Account/ResetPassword.cshtml.cs`

## <a name="disable-register-page"></a>禁用注册页

禁用用户注册：

* 基架 Identity 。 包括帐户. Register、RegisterConfirmation。 例如：

  ```dotnetcli
   dotnet aspnet-codegenerator identity -dc RPauth.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.RegisterConfirmation"
  ```

* 更新 *区域/ Identity /Pages/Account/Register.cshtml.cs* ，使用户无法从此终结点注册：

  [!code-csharp[](scaffold-identity/sample/Register.cshtml.cs?name=snippet)]

* 更新 *区域/ Identity /Pages/Account/Register.cshtml* ，使其与前面的更改一致：

  [!code-cshtml[](scaffold-identity/sample/Register.cshtml)]

* 注释掉或删除 *区域/ Identity /Pages/Account/Login.cshtml* 中的注册链接

```cshtml
@*
<p>
    <a asp-page="./Register" asp-route-returnUrl="@Model.ReturnUrl">Register as a new user</a>
</p>
*@
```

* 更新 " *区域/ Identity /Pages/Account/RegisterConfirmation* " 页。

  * 删除来自 cshtml 文件的代码和链接。
  * 从中删除确认代码 `PageModel` ：

  ```csharp
   [AllowAnonymous]
    public class RegisterConfirmationModel : PageModel
    {
        public IActionResult OnGet()
        {  
            return Page();
        }
    }
  ```
  
### <a name="use-another-app-to-add-users"></a>使用其他应用添加用户

提供一种在 web 应用外部添加用户的机制。 用于添加用户的选项包括：

* 专用的管理 web 应用。
* 控制台应用。

下面的代码概述了一种添加用户的方法：

* 用户列表将读入内存中。
* 为每个用户生成一个强唯一密码。
* 用户已添加到 Identity 数据库。
* 系统会通知用户并通知用户更改密码。

[!code-csharp[](scaffold-identity/consoleAddUser/Program.cs?name=snippet)]

下面的代码概述了如何添加用户：

[!code-csharp[](scaffold-identity/consoleAddUser/Data/SeedData.cs?name=snippet)]

对于生产方案，可以遵循类似的方法。

## <a name="additional-resources"></a>其他资源

* [更改为 ASP.NET Core 2.1 及更高版本的身份验证代码](xref:migration/20_21#changes-to-authentication-code)

::: moniker-end
