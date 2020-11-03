---
title: 迁移身份验证和 Identity ASP.NET Core
author: ardalis
description: 了解如何从 ASP.NET MVC 项目将身份验证和标识迁移到 ASP.NET Core MVC 项目。
ms.author: riande
ms.date: 3/22/2020
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
uid: migration/identity
ms.openlocfilehash: 8ceff0596c069d815c38b9bb526477a9d1430951
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060646"
---
# <a name="migrate-authentication-and-no-locidentity-to-aspnet-core"></a>迁移身份验证和 Identity ASP.NET Core

作者：[Steve Smith](https://ardalis.com/)

在前面的文章中，我们 [已将配置从 ASP.NET mvc 项目迁移到 ASP.NET CORE mvc](xref:migration/configuration)。 本文将迁移注册、登录和用户管理功能。

## <a name="configure-no-locidentity-and-membership"></a>配置 Identity 和成员资格

在 ASP.NET MVC 中，身份验证和标识功能是使用 Identity *Startup.Auth.cs* 和 *Identity Config.cs* 中的 ASP.NET 配置的，位于 *App_Start* 文件夹中。 在 ASP.NET Core MVC 中，这些功能在 *Startup.cs* 中进行配置。

安装以下 NuGet 包：

* `Microsoft.AspNetCore.Identity.EntityFrameworkCore`
* `Microsoft.AspNetCore.Authentication.Cookies`
* `Microsoft.EntityFrameworkCore.SqlServer`

在 *Startup.cs* 中，更新 `Startup.ConfigureServices` 方法以使用实体框架和 Identity 服务：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add EF services to the services container.
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));

    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

     services.AddMvc();
}
```

此时，在上面的代码中引用了两个类型，这些类型尚未从 ASP.NET MVC 项目迁移： `ApplicationDbContext` 和 `ApplicationUser` 。 在 ASP.NET Core 项目中创建新的 " *模型* " 文件夹，并将两个与这些类型对应的类添加到其中。 你将在 */Models/ Identity Models.cs* 中找到这些类的 ASP.NET MVC 版本，但我们会在迁移的项目中对每个类使用一个文件，因为这样做更为清晰。

*ApplicationUser.cs* ：

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;

namespace NewMvcProject.Models
{
  public class ApplicationUser : IdentityUser
  {
  }
}
```

*ApplicationDbContext.cs* ：

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.Data.Entity;

namespace NewMvcProject.Models
{
    public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }

        protected override void OnModelCreating(ModelBuilder builder)
        {
            base.OnModelCreating(builder);
            // Customize the ASP.NET Core Identity model and override the defaults if needed.
            // For example, you can rename the ASP.NET Core Identity table names and more.
            // Add your customizations after calling base.OnModelCreating(builder);
        }
    }
}
```

ASP.NET Core MVC 初学者 Web 项目不包括很多自定义的用户或 `ApplicationDbContext` 。 迁移实际应用时，还需要迁移应用的用户和类的所有自定义属性和方法，以及应用所使用的 `DbContext` 任何其他模型类。 例如，如果 `DbContext` 具有 `DbSet<Album>` ，则需要迁移 `Album` 该类。

将这些文件放入后，可通过更新其语句对 *Startup.cs* 文件进行编译 `using` ：

```csharp
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Hosting;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
```

现在，我们的应用程序已准备好支持身份验证和 Identity 服务。 只需将这些功能公开给用户。

## <a name="migrate-registration-and-login-logic"></a>迁移注册和登录逻辑

通过为 Identity 应用配置的服务和使用实体框架和 SQL Server 配置的数据访问，我们已准备好添加注册和登录到应用的支持。 请记住，在 [迁移过程](xref:migration/mvc#migrate-the-layout-file)中，我们已注释掉对 *_Layout* 中 *_LoginPartial* 的引用。 现在可以返回到该代码，取消注释，并添加必要的控制器和视图以支持登录功能。

取消注释 `@Html.Partial` *_Layout* 中的行：

```cshtml
      <li>@Html.ActionLink("Contact", "Contact", "Home")</li>
    </ul>
    @*@Html.Partial("_LoginPartial")*@
  </div>
</div>
```

现在，将 Razor 名为 *_LoginPartial* 的新视图添加到 *Views/Shared* 文件夹中：

用下面的代码更新 *_LoginPartial* ， () 替换其所有内容：

```cshtml
@inject SignInManager<ApplicationUser> SignInManager
@inject UserManager<ApplicationUser> UserManager

@if (SignInManager.IsSignedIn(User))
{
    <form asp-area="" asp-controller="Account" asp-action="Logout" method="post" id="logoutForm" class="navbar-right">
        <ul class="nav navbar-nav navbar-right">
            <li>
                <a asp-area="" asp-controller="Manage" asp-action="Index" title="Manage">Hello @UserManager.GetUserName(User)!</a>
            </li>
            <li>
                <button type="submit" class="btn btn-link navbar-btn navbar-link">Log out</button>
            </li>
        </ul>
    </form>
}
else
{
    <ul class="nav navbar-nav navbar-right">
        <li><a asp-area="" asp-controller="Account" asp-action="Register">Register</a></li>
        <li><a asp-area="" asp-controller="Account" asp-action="Login">Log in</a></li>
    </ul>
}
```

此时，您应该能够在浏览器中刷新站点。

## <a name="summary"></a>摘要

ASP.NET Core 引入了对 ASP.NET 功能的更改 Identity 。 本文介绍了如何将 ASP.NET 的身份验证和用户管理功能迁移 Identity 到 ASP.NET Core。
