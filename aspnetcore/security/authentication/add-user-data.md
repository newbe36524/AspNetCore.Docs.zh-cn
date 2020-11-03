---
title: 在 ASP.NET Core 项目中添加、下载和删除用户数据 Identity
author: rick-anderson
description: 了解如何将自定义用户数据添加到 Identity ASP.NET Core 项目中的。 删除每个 GDPR 的数据。
ms.author: riande
ms.date: 03/26/2020
ms.custom: mvc, seodec18
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
uid: security/authentication/add-user-data
ms.openlocfilehash: a4e1fd780947cfa5f09fb1e03964595fa09f0f18
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061413"
---
# <a name="add-download-and-delete-custom-user-data-to-no-locidentity-in-an-aspnet-core-project"></a>在 ASP.NET Core 项目中添加、下载和删除自定义用户数据 Identity

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

本文介绍以下操作：

* 向 ASP.NET Core web 应用添加自定义用户数据。
* 使用属性标记自定义用户数据模型 <xref:Microsoft.AspNetCore.Identity.PersonalDataAttribute> ，使其自动可供下载和删除。 使数据能够下载和删除有助于满足 [GDPR](xref:security/gdpr) 要求。

项目示例是从 Razor 页面 web 应用创建的，但 ASP.NET CORE MVC web 应用的说明类似。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/add-user-data)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="prerequisites"></a>先决条件

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [](~/includes/3.0-SDK.md)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE [](~/includes/2.2-SDK.md)]

::: moniker-end

## <a name="create-a-no-locrazor-web-app"></a>创建 Razor web 应用

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-3.0"

* 从 Visual Studio“文件”菜单中选择“新建” > “项目”  。 将项目命名为 " **WebApp1** " （如果你希望它与 [下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) 代码的命名空间相匹配）。
* 选择 **ASP.NET Core Web 应用程序** > **正常**
* 在下拉列表中选择 **ASP.NET Core 3.0**
* 选择 **Web 应用程序** > **正常**
* 生成并运行该项目。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

* 从 Visual Studio“文件”菜单中选择“新建” > “项目”  。 将项目命名为 " **WebApp1** " （如果你希望它与 [下载示例](https://github.com/dotnet/AspNetCore.Docs/tree/live/aspnetcore/security/authentication/add-user-data) 代码的命名空间相匹配）。
* 选择 **ASP.NET Core Web 应用程序** > **正常**
* 在下拉列表中选择 **ASP.NET Core 2.2**
* 选择 **Web 应用程序** > **正常**
* 生成并运行该项目。

::: moniker-end


# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp -o WebApp1
```

---

## <a name="run-the-no-locidentity-scaffolder"></a>运行 Identity scaffolder

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在 **解决方案资源管理器** 中，右键单击项目 > " **添加**  >  **新的基架项** "。
* 在 " **添加基架** " 对话框的左窗格中，选择 " **Identity**  >  **添加** "。
* 在 " **添加 Identity** " 对话框中，选择以下选项：
  * 选择现有的布局文件  *~/Pages/Shared/_Layout cshtml*
  * 选择以下要重写的文件：
    * **帐户/注册**
    * **帐户/管理/索引**
  * 选择 **+** 按钮以创建新的 **数据上下文类** 。 如果项目命名为 **WebApp1** ) ，则接受类型 ( **WebApp1. WebApp1Context** 。
  * 选择 **+** 按钮以创建新的 **用户类** 。 如果项目命名为 **WebApp1** ) > " **添加** "，则接受 " **WebApp1User** " (类型。
* 选择 **添加** 。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

如果你之前未安装 ASP.NET Core scaffolder，请立即安装：

```dotnetcli
dotnet tool install -g dotnet-aspnet-codegenerator
```

将对 [VisualStudio](https://www.nuget.org/packages/Microsoft.VisualStudio.Web.CodeGeneration.Design/) 的包引用添加到项目 ( .csproj) 文件中。 在项目目录中运行以下命令：

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet restore
```

运行以下命令以列出 Identity scaffolder 选项：

```dotnetcli
dotnet aspnet-codegenerator identity -h
```

在项目文件夹中，运行 Identity scaffolder：

```dotnetcli
dotnet aspnet-codegenerator identity -u WebApp1User -fi Account.Register;Account.Manage.Index
```

---

按照 " [迁移"、"UseAuthentication" 和 "布局](xref:security/authentication/scaffold-identity#efm) " 中的说明执行以下步骤：

* 创建迁移并更新数据库。
* 已将 `UseAuthentication` 添加到 `Startup.Configure`。
* 添加 `<partial name="_LoginPartial" />` 到布局文件中。
* 测试应用：
  * 注册用户
  * 选择 " **注销** " 链接旁 ("新用户名") 。 可能需要展开窗口或选择导航栏图标来显示用户名和其他链接。
  * 选择 " **个人数据** " 选项卡。
  * 选择 " **下载** " 按钮，并检查文件 *上的PersonalData.js* 。
  * 测试 **删除** 按钮，该按钮将删除已登录的用户。

## <a name="add-custom-user-data-to-the-no-locidentity-db"></a>向数据库添加自定义用户数据 Identity

`IdentityUser`用自定义属性更新派生类。 如果已将项目命名为 WebApp1，则该文件将命名为 *Areas/ Identity /Data/WebApp1User.cs* 。 用以下代码更新文件：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Data/WebApp1User.cs)]

::: moniker-end

具有 [PersonalData](/dotnet/api/microsoft.aspnetcore.identity.personaldataattribute) 属性的属性为：

* 当 *Areas/ Identity /Pages/Account/Manage/DeletePersonalData.cshtml* Razor 页调用时删除 `UserManager.Delete` 。
* 由 *Areas/ Identity /Pages/Account/Manage/DownloadPersonalData.cshtml* 页包含在下载的数据中 Razor 。

### <a name="update-the-accountmanageindexcshtml-page"></a>更新 "帐户/管理/索引" 页

`InputModel`用以下突出显示的代码更新 *区域/ Identity /Pages/Account/Manage/Index.cshtml.cs* 中的：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=24-32,48-49,96-104,106)]

用以下突出显示的标记更新 *区域/ Identity /Pages/Account/Manage/Index.cshtml* ：

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=18-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml.cs?name=snippet&highlight=28-36,63-64,98-106,119)]

用以下突出显示的标记更新 *区域/ Identity /Pages/Account/Manage/Index.cshtml* ：

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Manage/Index.cshtml?highlight=35-42)]

::: moniker-end

### <a name="update-the-accountregistercshtml-page"></a>更新帐户/注册. cshtml 页

`InputModel`用以下突出显示的代码更新 *区域/ Identity /Pages/Account/Register.cshtml.cs* 中的：

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=30-38,70-71)]

用以下突出显示的标记更新 *区域/ Identity /Pages/Account/Register.cshtml* ：

[!code-cshtml[](add-user-data/samples/3.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=28-36,67,66)]

用以下突出显示的标记更新 *区域/ Identity /Pages/Account/Register.cshtml* ：

[!code-cshtml[](add-user-data/samples/2.x/SampleApp/Areas/Identity/Pages/Account/Register.cshtml?highlight=16-25)]

::: moniker-end


生成项目。

### <a name="add-a-migration-for-the-custom-user-data"></a>添加自定义用户数据的迁移

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 Visual Studio **包管理器控制台** 中：

```powershell
Add-Migration CustomUserData
Update-Database
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet ef migrations add CustomUserData
dotnet ef database update
```

---

## <a name="test-create-view-download-delete-custom-user-data"></a>测试创建、查看、下载、删除自定义用户数据

测试应用：

* 注册新用户。
* 在页面上查看自定义用户数据 `/Identity/Account/Manage` 。
* 从页面下载并查看用户的个人数据 `/Identity/Account/Manage/PersonalData` 。

## <a name="add-claims-to-no-locidentity-using-iuserclaimsprincipalfactoryapplicationuser"></a>Identity使用 IUserClaimsPrincipalFactory 添加声明<ApplicationUser>

> [!NOTE]
> 本部分不是上一教程的扩展。 若要将以下步骤应用到使用本教程构建的应用，请参阅 [此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/18797)。

可以通过使用接口向添加其他声明 ASP.NET Core Identity `IUserClaimsPrincipalFactory<T>` 。 可在方法中将此类添加到应用 `Startup.ConfigureServices` 。 添加类的自定义实现，如下所示：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddIdentity<ApplicationUser, IdentityRole>()
        .AddEntityFrameworkStores<ApplicationDbContext>()
        .AddDefaultTokenProviders();

    services.AddScoped<IUserClaimsPrincipalFactory<ApplicationUser>, 
        AdditionalUserClaimsPrincipalFactory>();
```

演示代码使用 `ApplicationUser` 类。 此类添加一个 `IsAdmin` 属性，该属性用于添加其他声明。

```csharp
public class ApplicationUser : IdentityUser
{
    public bool IsAdmin { get; set; }
}
```

`AdditionalUserClaimsPrincipalFactory` 实现 `UserClaimsPrincipalFactory` 接口。 新的角色声明将添加到中 `ClaimsPrincipal` 。

```csharp
public class AdditionalUserClaimsPrincipalFactory 
        : UserClaimsPrincipalFactory<ApplicationUser, IdentityRole>
{
    public AdditionalUserClaimsPrincipalFactory( 
        UserManager<ApplicationUser> userManager,
        RoleManager<IdentityRole> roleManager, 
        IOptions<IdentityOptions> optionsAccessor) 
        : base(userManager, roleManager, optionsAccessor)
    {}

    public async override Task<ClaimsPrincipal> CreateAsync(ApplicationUser user)
    {
        var principal = await base.CreateAsync(user);
        var identity = (ClaimsIdentity)principal.Identity;

        var claims = new List<Claim>();
        if (user.IsAdmin)
        {
            claims.Add(new Claim(JwtClaimTypes.Role, "admin"));
        }
        else
        {
            claims.Add(new Claim(JwtClaimTypes.Role, "user"));
        }

        identity.AddClaims(claims);
        return principal;
    }
}
```

然后，可以在应用中使用其他声明。 在 Razor 页中， `IAuthorizationService` 可以使用实例访问声明值。

```cshtml
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

@if ((await AuthorizationService.AuthorizeAsync(User, "IsAdmin")).Succeeded)
{
    <ul class="mr-auto navbar-nav">
        <li class="nav-item">
            <a class="nav-link" asp-controller="Admin" asp-action="Index">ADMIN</a>
        </li>
    </ul>
}
```
