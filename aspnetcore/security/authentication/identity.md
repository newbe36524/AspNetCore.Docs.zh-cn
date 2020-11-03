---
title: ASP.NET Core 简介 Identity
author: rick-anderson
description: Identity与 ASP.NET Core 应用一起使用。 了解如何 (RequireDigit、RequiredLength、RequiredUniqueChars 等) 设置密码要求。
ms.author: riande
ms.date: 7/15/2020
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
uid: security/authentication/identity
ms.openlocfilehash: bfcef860beb07ab81dda1a10a1648491ae187bef
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052014"
---
# <a name="introduction-to-no-locidentity-on-aspnet-core"></a>ASP.NET Core 简介 Identity

::: moniker range=">= aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core Identity:

* 是一个 API，它支持用户界面) 登录功能 (UI。
* 管理用户、密码、配置文件数据、角色、声明、令牌、电子邮件确认等。

用户可以使用存储在中的登录信息创建一个帐户， Identity 也可以使用外部登录提供程序。 支持的外部登录提供程序包括 [Facebook、Google、Microsoft 帐户和 Twitter](xref:security/authentication/social/index)。

[!INCLUDE[](~/includes/requireAuth.md)]

GitHub 上提供了[ Identity 源代码](https://github.com/dotnet/AspNetCore/tree/master/src/Identity)。 [基架 Identity ](xref:security/authentication/scaffold-identity)并查看生成的文件以查看与的模板交互 Identity 。

Identity 通常使用 SQL Server 数据库配置以存储用户名、密码和配置文件数据。 另外，还可以使用另一个永久性存储，例如 Azure 表存储。

在本主题中，你将了解如何使用 Identity 注册、登录和注销用户。 注意：这些模板将用户的用户名和电子邮件视为相同。 有关创建使用的应用程序的更多详细说明 Identity ，请参阅 [后续步骤](#next)。

[Microsoft 标识平台](/azure/active-directory/develop/) 是：

* ) 开发人员平台 Azure AD Azure Active Directory (的演变。
* 与无关 ASP.NET Core Identity 。

[!INCLUDE[](~/includes/IdentityServer4.md)]

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/identity/sample) ([如何下载](xref:index#how-to-download-a-sample)) 。

<a name="adi"></a>

## <a name="create-a-web-app-with-authentication"></a>创建具有身份验证的 Web 应用

使用单独的用户帐户创建 ASP.NET Core Web 应用程序项目。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 选择 " **文件** " " > **新建** > **项目** "。
* 选择“ASP.NET Core Web 应用程序”。 将项目命名为 **WebApp1** ，使其命名空间与项目下载相同。 单击“确定”。
* 选择 ASP.NET Core **Web 应用程序** ，然后选择 " **更改身份验证** "。
* 选择 **单个用户帐户** ，然后单击 **"确定"** 。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp --auth Individual -o WebApp1
```

上述命令 Razor 使用 SQLite 创建 web 应用。 若要创建具有 LocalDB 的 web 应用，请运行以下命令：

```dotnetcli
dotnet new webapp --auth Individual -uld -o WebApp1
```

---

生成的项目 [ASP.NET Core Identity](xref:security/authentication/identity) 以类库形式[ Razor ](xref:razor-pages/ui-class)提供。 类库 Identity Razor 公开的终结点 `Identity` 。 例如： 。

* /Identity/Account/Login
* /Identity/Account/Logout
* /Identity/Account/Manage

### <a name="apply-migrations"></a>应用迁移

应用迁移以初始化数据库。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在包管理器控制台中运行以下命令 (PMC) ：

`PM> Update-Database`

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

使用 SQLite 时，此步骤不需要迁移。

[!INCLUDE [more information on the CLI for EF Core](~/includes/ef-cli.md)]

对于 LocalDB，请运行以下命令：

```dotnetcli
dotnet ef database update
```

---

### <a name="test-register-and-login"></a>测试注册和登录

运行应用并注册用户。 根据屏幕大小，你可能需要选择 "导航" 切换按钮以查看 " **寄存器** " 和 " **登录** " 链接。

[!INCLUDE[](~/includes/view-identity-db.md)]

<a name="pw"></a>

### <a name="configure-no-locidentity-services"></a>配置 Identity 服务

中添加了服务 `ConfigureServices` 。 典型模式是调用所有 `Add{Service}` 方法，然后调用所有 `services.Configure{Service}` 方法。

[!code-csharp[](identity/sample/WebApp3/Startup.cs?name=snippet_configureservices&highlight=11-99)]

前面突出显示的代码 Identity 用默认选项值进行配置。 服务通过 [依赖关系注入](xref:fundamentals/dependency-injection)提供给应用程序。

Identity 通过调用启用 <xref:Microsoft.AspNetCore.Builder.AuthAppBuilderExtensions.UseAuthentication*> 。 `UseAuthentication` 将身份验证 [中间件](xref:fundamentals/middleware/index) 添加到请求管道。

[!code-csharp[](identity/sample/WebApp3/Startup.cs?name=snippet_configure&highlight=19)]

模板生成的应用不使用 [授权](xref:security/authorization/secure-data)。 `app.UseAuthorization` 包括，以确保在应用添加授权时，按正确的顺序添加。 `UseRouting`、 `UseAuthentication` 、 `UseAuthorization` 和 `UseEndpoints` 必须按前面的代码中所示的顺序调用。

有关和的详细 `IdentityOptions` 信息 `Startup` ，请参阅 <xref:Microsoft.AspNetCore.Identity.IdentityOptions> 和 [应用程序启动](xref:fundamentals/startup)。

## <a name="scaffold-register-login-logout-and-registerconfirmation"></a>基架注册、登录、注销和 RegisterConfirmation

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

添加 `Register` 、 `Login` 、 `LogOut` 和 `RegisterConfirmation` 文件。 将 [基架标识置于 Razor 具有授权](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization) 说明的项目中，以生成本部分中所示的代码。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

如果创建的项目的名称为 **WebApp1** ，请运行以下命令。 否则，请使用正确的命名空间 `ApplicationDbContext` ：

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout;Account.RegisterConfirmation"
```

PowerShell 使用分号作为命令分隔符。 使用 PowerShell 时，请对文件列表中的分号进行转义，或将文件列表置于双引号中，如前面的示例所示。

有关基架的详细信息 Identity ，请参阅 [ Razor 使用授权将基架标识转换为项目](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization)。

---

### <a name="examine-register"></a>检查注册

当用户单击页面上的 " **注册** " 按钮时 `Register` ，将 `RegisterModel.OnPostAsync` 调用该操作。 用户由 [CreateAsync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_) 在对象上创建 `_userManager` ：

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=9)]

<!-- .NET 5 fixes this, see
https://github.com/dotnet/aspnetcore/blob/master/src/Identity/UI/src/Areas/Identity/Pages/V4/Account/RegisterConfirmation.cshtml.cs#L74-L77
-->
[!INCLUDE[](~/includes/disableVer.md)]

### <a name="log-in"></a>登录

当出现以下情况时，将显示登录窗体：

* 选择 " **登录** " 链接。
* 用户尝试访问他们无权访问的受限制的页面， **或** 未经系统的身份验证。

提交登录页上的窗体时，将 `OnPostAsync` 调用该操作。 `PasswordSignInAsync` 对 `_signInManager` 对象调用。

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=10-11)]

有关如何做出授权决策的信息，请参阅 <xref:security/authorization/introduction> 。

### <a name="log-out"></a>注销

" **注销** " 链接将调用该 `LogoutModel.OnPost` 操作。 

[!code-csharp[](identity/sample/WebApp3/Areas/Identity/Pages/Account/Logout.cshtml.cs?highlight=36)]

在前面的代码中，代码 `return RedirectToPage();` 需要是重定向，以便浏览器执行新请求并更新用户的标识。

[SignOutAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync) 清除存储在中的用户声明 cookie 。

Post 在 *Pages/Shared/_LoginPartial 中指定。 cshtml* ：

[!code-cshtml[](identity/sample/WebApp3/Pages/Shared/_LoginPartial.cshtml?highlight=15)]

## <a name="test-no-locidentity"></a>考试 Identity

默认 web 项目模板允许匿名访问主页。 若要测试 Identity ，请添加 [`[Authorize]`](xref:Microsoft.AspNetCore.Authorization.AuthorizeAttribute) ：

[!code-csharp[](identity/sample/WebApp3/Pages/Privacy.cshtml.cs?highlight=7)]

如果已登录，请注销。运行应用并选择 " **隐私** " 链接。 将被重定向到登录页。

### <a name="explore-no-locidentity"></a>浏览 Identity

Identity了解更多详细信息：

* [创建完全标识 UI 源](xref:security/authentication/scaffold-identity#create-full-identity-ui-source)
* 检查每个页面的源，并单步执行调试程序。

## <a name="no-locidentity-components"></a>Identity 组分

所有 Identity 相关的 NuGet 包都包含在 [ASP.NET Core 共享框架](xref:aspnetcore-3.0#use-the-aspnet-core-shared-framework)中。

的主包为 Identity [AspNetCore。 Identity ](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/) 此包包含的核心接口集 ASP.NET Core Identity ，由提供 `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 。

## <a name="migrating-to-no-locaspnet-core-identity"></a>迁移到 ASP.NET Core Identity

有关迁移现有存储的详细信息和指南 Identity ，请参阅[迁移身份验证 Identity 和](xref:migration/identity)。

## <a name="setting-password-strength"></a>设置密码强度

有关设置最小密码要求的示例，请参阅 [配置](#pw) 。

## <a name="adddefaultno-locidentity-and-addno-locidentity"></a>AddDefault Identity 并添加Identity

<xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> 是在 ASP.NET Core 2.1 中引入的。 调用 `AddDefaultIdentity` 类似于调用以下内容：

* <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentity*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderExtensions.AddDefaultTokenProviders*>

有关详细信息，请参阅 [AddDefault Identity 源](https://github.com/dotnet/AspNetCore/blob/release/3.1/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) 。

## <a name="prevent-publish-of-static-no-locidentity-assets"></a>禁止发布静态 Identity 资产

若要防止将静态 Identity 资产发布 (用于 UI) 的用户的样式和 JavaScript 文件 Identity ，请将以下 `ResolveStaticWebAssetsInputsDependsOn` 属性和目标添加 `RemoveIdentityAssets` 到应用的项目文件中：

```xml
<PropertyGroup>
  <ResolveStaticWebAssetsInputsDependsOn>RemoveIdentityAssets</ResolveStaticWebAssetsInputsDependsOn>
</PropertyGroup>

<Target Name="RemoveIdentityAssets">
  <ItemGroup>
    <StaticWebAsset Remove="@(StaticWebAsset)" Condition="%(SourceId) == 'Microsoft.AspNetCore.Identity.UI'" />
  </ItemGroup>
</Target>
```

<a name="next"></a>

## <a name="next-steps"></a>后续步骤

* [ASP.NET Core Identity 源代码](https://github.com/dotnet/aspnetcore/tree/master/src/Identity)
* 有关使用 SQLite 进行配置的信息，请参阅 [此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/5131) Identity 。
* [配置 Identity](xref:security/authentication/identity-configuration)
* <xref:security/authorization/secure-data>
* <xref:security/authentication/add-user-data>
* <xref:security/authentication/identity-enable-qrcodes>
* <xref:migration/identity>
* <xref:security/authentication/accconfirm>
* <xref:security/authentication/2fa>
* <xref:host-and-deploy/web-farm>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

ASP.NET Core Identity 是将登录功能添加到 ASP.NET Core 应用的成员资格系统。 用户可以使用存储在中的登录信息创建一个帐户， Identity 也可以使用外部登录提供程序。 支持的外部登录提供程序包括 [Facebook、Google、Microsoft 帐户和 Twitter](xref:security/authentication/social/index)。

Identity 可以使用 SQL Server 数据库配置以存储用户名、密码和配置文件数据。 另外，还可以使用另一个永久性存储，例如 Azure 表存储。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/identity/sample/src/ASPNETCore-IdentityDemoComplete/) ([如何下载](xref:index#how-to-download-a-sample)) 。

在本主题中，你将了解如何使用 Identity 注册、登录和注销用户。 有关创建使用的应用程序的更多详细说明 Identity ，请参阅本文末尾的后续步骤部分。

<a name="adi"></a>

## <a name="adddefaultno-locidentity-and-addno-locidentity"></a>AddDefault Identity 并添加Identity

<xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionUIExtensions.AddDefaultIdentity*> 是在 ASP.NET Core 2.1 中引入的。 调用 `AddDefaultIdentity` 类似于调用以下内容：

* <xref:Microsoft.Extensions.DependencyInjection.IdentityServiceCollectionExtensions.AddIdentity*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderUIExtensions.AddDefaultUI*>
* <xref:Microsoft.AspNetCore.Identity.IdentityBuilderExtensions.AddDefaultTokenProviders*>

有关详细信息，请参阅 [AddDefault Identity 源](https://github.com/dotnet/AspNetCore/blob/release/2.1/src/Identity/UI/src/IdentityServiceCollectionUIExtensions.cs#L47-L63) 。

## <a name="create-a-web-app-with-authentication"></a>创建具有身份验证的 Web 应用

使用单独的用户帐户创建 ASP.NET Core Web 应用程序项目。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 选择 " **文件** " " > **新建** > **项目** "。
* 选择“ASP.NET Core Web 应用程序”。 将项目命名为 **WebApp1** ，使其命名空间与项目下载相同。 单击“确定”。
* 选择 ASP.NET Core **Web 应用程序** ，然后选择 " **更改身份验证** "。
* 选择 **单个用户帐户** ，然后单击 **"确定"** 。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet new webapp --auth Individual -o WebApp1
```

---

生成的项目 [ASP.NET Core Identity](xref:security/authentication/identity) 以类库形式[ Razor ](xref:razor-pages/ui-class)提供。 类库 Identity Razor 公开的终结点 `Identity` 。 例如： 。

* /Identity/Account/Login
* /Identity/Account/Logout
* /Identity/Account/Manage

### <a name="apply-migrations"></a>应用迁移

应用迁移以初始化数据库。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在包管理器控制台中运行以下命令 (PMC) ：

```powershell
Update-Database
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

```dotnetcli
dotnet ef database update
```

---

### <a name="test-register-and-login"></a>测试注册和登录

运行应用并注册用户。 根据屏幕大小，你可能需要选择 "导航" 切换按钮以查看 " **寄存器** " 和 " **登录** " 链接。

[!INCLUDE[](~/includes/view-identity-db.md)]

<a name="pw"></a>

### <a name="configure-no-locidentity-services"></a>配置 Identity 服务

中添加了服务 `ConfigureServices` 。 典型模式是调用所有 `Add{Service}` 方法，然后调用所有 `services.Configure{Service}` 方法。

[!code-csharp[](identity/sample/WebApp1/Startup.cs?name=snippet_configureservices)]

前面的代码 Identity 用默认选项值进行配置。 服务通过 [依赖关系注入](xref:fundamentals/dependency-injection)提供给应用程序。

Identity 通过调用 [UseAuthentication](/dotnet/api/microsoft.aspnetcore.builder.authappbuilderextensions.useauthentication#Microsoft_AspNetCore_Builder_AuthAppBuilderExtensions_UseAuthentication_Microsoft_AspNetCore_Builder_IApplicationBuilder_)启用。 `UseAuthentication` 将身份验证 [中间件](xref:fundamentals/middleware/index) 添加到请求管道。

[!code-csharp[](identity/sample/WebApp1/Startup.cs?name=snippet_configure&highlight=18)]

有关详细信息，请参阅[ Identity Options 类](/dotnet/api/microsoft.aspnetcore.identity.identityoptions)和[应用程序启动](xref:fundamentals/startup)。

## <a name="scaffold-register-login-and-logout"></a>基架注册、登录和注销

将 [基架标识置于 Razor 具有授权](xref:security/authentication/scaffold-identity#scaffold-identity-into-a-razor-project-with-authorization) 说明的项目中，以生成本部分中所示的代码。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

添加注册、登录和注销文件。

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

如果创建的项目的名称为 **WebApp1** ，请运行以下命令。 否则，请使用正确的命名空间 `ApplicationDbContext` ：

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet aspnet-codegenerator identity -dc WebApp1.Data.ApplicationDbContext --files "Account.Register;Account.Login;Account.Logout"
```

PowerShell 使用分号作为命令分隔符。 使用 PowerShell 时，请对文件列表中的分号进行转义，或将文件列表置于双引号中，如前面的示例所示。

---

### <a name="examine-register"></a>检查注册

当用户单击 " **注册** " 链接时，将 `RegisterModel.OnPostAsync` 调用该操作。 用户由 [CreateAsync](/dotnet/api/microsoft.aspnetcore.identity.usermanager-1.createasync#Microsoft_AspNetCore_Identity_UserManager_1_CreateAsync__0_System_String_) 在对象上创建 `_userManager` ：

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Register.cshtml.cs?name=snippet&highlight=7)]

如果用户已成功创建，则对的调用会登录该用户 `_signInManager.SignInAsync` 。

**注意：** 请参阅 [帐户确认](xref:security/authentication/accconfirm#prevent-login-at-registration) 以了解在注册时要阻止立即登录的步骤。

### <a name="log-in"></a>登录

当出现以下情况时，将显示登录窗体：

* 选择 " **登录** " 链接。
* 用户尝试访问他们无权访问的受限制的页面， **或** 未经系统的身份验证。

提交登录页上的窗体时，将 `OnPostAsync` 调用该操作。 `PasswordSignInAsync` 对 `_signInManager` 对象调用。

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Login.cshtml.cs?name=snippet&highlight=10-11)]

有关如何做出授权决策的信息，请参阅 <xref:security/authorization/introduction> 。

### <a name="log-out"></a>注销

" **注销** " 链接将调用该 `LogoutModel.OnPost` 操作。 

[!code-csharp[](identity/sample/WebApp1/Areas/Identity/Pages/Account/Logout.cshtml.cs)]

[SignOutAsync](/dotnet/api/microsoft.aspnetcore.identity.signinmanager-1.signoutasync#Microsoft_AspNetCore_Identity_SignInManager_1_SignOutAsync) 清除存储在中的用户声明 cookie 。

Post 在 *Pages/Shared/_LoginPartial 中指定。 cshtml* ：

[!code-cshtml[](identity/sample/WebApp1/Pages/Shared/_LoginPartial.cshtml?highlight=16)]

## <a name="test-no-locidentity"></a>考试 Identity

默认 web 项目模板允许匿名访问主页。 若要进行测试 Identity ，请将添加 [`[Authorize]`](/dotnet/api/microsoft.aspnetcore.authorization.authorizeattribute) 到 "隐私" 页。

[!code-csharp[](identity/sample/WebApp1/Pages/Privacy.cshtml.cs?highlight=7)]

如果已登录，请注销。运行应用并选择 " **隐私** " 链接。 将被重定向到登录页。

### <a name="explore-no-locidentity"></a>浏览 Identity

Identity了解更多详细信息：

* [创建完全标识 UI 源](xref:security/authentication/scaffold-identity#create-full-identity-ui-source)
* 检查每个页面的源，并单步执行调试程序。

## <a name="no-locidentity-components"></a>Identity 组分

所有 Identity 相关 NuGet 包都包含在 [AspNetCore 元包](xref:fundamentals/metapackage-app)中。

的主包为 Identity [AspNetCore。 Identity ](https://www.nuget.org/packages/Microsoft.AspNetCore.Identity/) 此包包含的核心接口集 ASP.NET Core Identity ，由提供 `Microsoft.AspNetCore.Identity.EntityFrameworkCore` 。

## <a name="migrating-to-no-locaspnet-core-identity"></a>迁移到 ASP.NET Core Identity

有关迁移现有存储的详细信息和指南 Identity ，请参阅[迁移身份验证 Identity 和](xref:migration/identity)。

## <a name="setting-password-strength"></a>设置密码强度

有关设置最小密码要求的示例，请参阅 [配置](#pw) 。

## <a name="next-steps"></a>后续步骤

* 有关使用 SQLite 进行配置的信息，请参阅 [此 GitHub 问题](https://github.com/dotnet/AspNetCore.Docs/issues/5131) Identity 。
* [配置 Identity](xref:security/authentication/identity-configuration)
* <xref:security/authorization/secure-data>
* <xref:security/authentication/add-user-data>
* <xref:security/authentication/identity-enable-qrcodes>
* <xref:migration/identity>
* <xref:security/authentication/accconfirm>
* <xref:security/authentication/2fa>
* <xref:host-and-deploy/web-farm>

::: moniker-end
