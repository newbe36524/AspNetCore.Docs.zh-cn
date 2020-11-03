---
title: 使用授权保护的用户数据创建 ASP.NET Core 应用
author: rick-anderson
description: 了解如何使用授权保护的用户数据创建 ASP.NET Core web 应用。 包括 HTTPS、身份验证、安全性 ASP.NET Core Identity 。
ms.author: riande
ms.date: 7/18/2020
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
uid: security/authorization/secure-data
ms.openlocfilehash: accfd46fa72c33976f8af2a39267c993447e036e
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051936"
---
# <a name="create-an-aspnet-core-web-app-with-user-data-protected-by-authorization"></a>使用受授权保护的用户数据创建 ASP.NET Core web 应用

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Joe Audette](https://twitter.com/joeaudette)

::: moniker range="= aspnetcore-2.0"

查看 [此 pdf](https://webpifeed.blob.core.windows.net/webpifeed/Partners/asp.net_repo_pdf_July16_18.pdf)

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

本教程演示如何创建 ASP.NET Core 的 web 应用，其中包含由授权保护的用户数据。 它将显示已进行身份验证 (已创建的已注册) 用户的联系人列表。 有三个安全组：

* **已注册的用户** 可以查看所有已批准的数据，并可以编辑/删除他们自己的数据。
* **经理** 可以批准或拒绝联系人数据。 只有已批准的联系人对用户可见。
* **管理员** 可以批准/拒绝和编辑/删除任何数据。

此文档中的图像与最新模板并不完全匹配。

在下图中，用户 Rick (`rick@example.com`) 已登录。 Rick 只能查看已批准的联系人， **编辑** / **删除** / 为其联系人 **创建新** 链接。 只有 Rick 创建的最后一条记录才会显示 " **编辑** " 和 " **删除** " 链接。 在经理或管理员将状态更改为 "已批准" 之前，其他用户将看不到最后一条记录。

![显示已登录的 Rick 的屏幕截图](secure-data/_static/rick.png)

在下图中， `manager@contoso.com` 已登录，并在管理器的角色中：

![显示 manager@contoso.com 已登录的屏幕截图](secure-data/_static/manager1.png)

下图显示了联系人的经理详细信息视图：

![联系人的经理视图](secure-data/_static/manager.png)

" **批准** " 和 " **拒绝** " 按钮仅为经理和管理员显示。

在下图中，以 `admin@contoso.com` 管理员的角色登录和：

![显示 admin@contoso.com 已登录的屏幕截图](secure-data/_static/admin.png)

管理员具有所有权限。 她可以读取/编辑/删除任何联系人并更改联系人的状态。

此应用是通过 [基架](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) 创建的 `Contact` ：以下模型：

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

该示例包含以下授权处理程序：

* `ContactIsOwnerAuthorizationHandler`：确保用户只能编辑其数据。
* `ContactManagerAuthorizationHandler`：允许经理批准或拒绝联系人。
* `ContactAdministratorsAuthorizationHandler`：允许管理员批准或拒绝联系人以及编辑/删除联系人。

## <a name="prerequisites"></a>先决条件

本教程是高级教程。 你应该熟悉：

* [ASP.NET Core](xref:tutorials/first-mvc-app/start-mvc)
* [身份验证](xref:security/authentication/identity)
* [帐户确认和密码恢复](xref:security/authentication/accconfirm)
* [授权](xref:security/authorization/introduction)
* [Entity Framework Core](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a>入门和已完成的应用程序

[下载](xref:index#how-to-download-a-sample)[已完成](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples)的应用。 [测试](#test-the-completed-app) 已完成的应用程序，使其安全功能熟悉。

### <a name="the-starter-app"></a>入门应用

[下载](xref:index#how-to-download-a-sample)[初学者](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/)应用。

运行应用程序，点击 " **ContactManager** " 链接，并验证是否可以创建、编辑和删除联系人。

## <a name="secure-user-data"></a>保护用户数据

以下部分包含创建安全用户数据应用的所有主要步骤。 你可能会发现，引用已完成的项目非常有用。

### <a name="tie-the-contact-data-to-the-user"></a>将联系人数据与用户关联

使用 ASP.NET [Identity](xref:security/authentication/identity) 用户 ID 可确保用户能够编辑其数据，而不是其他用户数据。 将 `OwnerID` 和添加 `ContactStatus` 到 `Contact` 模型：

[!code-csharp[](secure-data/samples/final3/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

`OwnerID` 数据库中的表的用户 ID `AspNetUser` [Identity](xref:security/authentication/identity) 。 此 `Status` 字段确定常规用户是否可查看联系人。

创建新的迁移并更新数据库：

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-no-locidentity"></a>将角色服务添加到 Identity

追加 [AddRoles](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) 以添加角色服务：

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet2&highlight=9)]

<a name="rau"></a>

### <a name="require-authenticated-users"></a>需要经过身份验证的用户

设置后备身份验证策略，要求对用户进行身份验证：

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet&highlight=13-99)]

前面突出显示的代码设置了 [后备身份验证策略](xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.FallbackPolicy)。 回退身份验证策略要求 * *_所有_* _ 用户进行身份验证，但 Razor 页面、控制器或操作方法除外。 例如， Razor 使用或的页、控制器或操作方法 `[AllowAnonymous]` `[Authorize(PolicyName="MyPolicy")]` 使用应用的身份验证属性而不是后备身份验证策略。

回退身份验证策略：

_ 适用于所有未显式指定身份验证策略的请求。 对于终结点路由服务的请求，这将包括未指定授权属性的任何终结点。 对于在授权中间件之后由其他中间件提供服务的请求，例如 [静态文件](xref:fundamentals/static-files)，这会将该策略应用到所有请求。

将后备身份验证策略设置为 "要求用户进行身份验证" 可保护新添加的 Razor 页面和控制器。 默认情况下，需要进行身份验证比依赖新控制器和 Razor 页面以包括属性更安全 `[Authorize]` 。

<xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions>类还包含 <xref:Microsoft.AspNetCore.Authorization.AuthorizationOptions.DefaultPolicy?displayProperty=nameWithType> 。 `DefaultPolicy` `[Authorize]` 当未指定策略时，是与特性一起使用的策略。 `[Authorize]` 不包含命名策略，与不同 `[Authorize(PolicyName="MyPolicy")]` 。

有关策略的详细信息，请参阅 <xref:security/authorization/policies> 。

MVC 控制器和 Razor 页面要求对所有用户进行身份验证的另一种方法是添加授权筛选器：

[!code-csharp[](secure-data/samples/final3/Startup2.cs?name=snippet&highlight=14-99)]

前面的代码使用授权筛选器，设置回退策略使用终结点路由。 设置回退策略是要求对所有用户进行身份验证的首选方式。

将 [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) 添加到 `Index` 和 `Privacy` 页面，以便匿名用户在注册之前可以获取有关站点的信息：

[!code-csharp[](secure-data/samples/final3/Pages/Index.cshtml.cs?highlight=1,7)]

### <a name="configure-the-test-account"></a>配置测试帐户

`SeedData`类创建两个帐户：管理员和管理器。 使用 [机密管理器工具](xref:security/app-secrets) 来设置这些帐户的密码。 将项目目录中的密码设置 (包含 *Program.cs* ) 的目录：

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

如果未指定强密码，则在调用时会引发异常 `SeedData.Initialize` 。

更新 `Main` 以使用测试密码：

[!code-csharp[](secure-data/samples/final3/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a>创建测试帐户并更新联系人

更新 `Initialize` 类中的方法 `SeedData` ，以创建测试帐户：

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet_Initialize)]

向联系人添加管理员用户 ID 和 `ContactStatus` 。 使其中一个联系人 "已提交" 和一个 "已拒绝"。 将用户 ID 和状态添加到所有联系人。 只显示一个联系人：

[!code-csharp[](secure-data/samples/final3/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a>创建所有者、经理和管理员授权处理程序

`ContactIsOwnerAuthorizationHandler`在 *Authorization* 文件夹中创建一个类。 `ContactIsOwnerAuthorizationHandler`验证对资源的用户是否拥有该资源。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

`ContactIsOwnerAuthorizationHandler`调用[上下文。](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_)如果当前经过身份验证的用户是联系人所有者，则会成功。 授权处理程序通常：

* `context.Succeed`满足要求时返回。
* `Task.CompletedTask`当不满足要求时返回。 `Task.CompletedTask` 不是成功或失败， &mdash; 它允许其他授权处理程序运行。

如果需要显式失败，请返回 [context。失败](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail)。

该应用程序允许联系人所有者编辑/删除/创建他们自己的数据。 `ContactIsOwnerAuthorizationHandler` 不需要检查在要求参数中传递的操作。

### <a name="create-a-manager-authorization-handler"></a>创建管理器授权处理程序

`ContactManagerAuthorizationHandler`在 *Authorization* 文件夹中创建一个类。 `ContactManagerAuthorizationHandler`验证对资源的用户是否为管理员。 只有管理人员才能 (新的或更改的) 批准或拒绝内容更改。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a>创建管理员授权处理程序

`ContactAdministratorsAuthorizationHandler`在 *Authorization* 文件夹中创建一个类。 `ContactAdministratorsAuthorizationHandler`验证对资源的用户是否为管理员。 管理员可以执行所有操作。

[!code-csharp[](secure-data/samples/final3/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a>注册授权处理程序

Entity Framework Core 使用 AddScoped 的服务必须使用[AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions)注册以进行[依赖关系注入](xref:fundamentals/dependency-injection)。 `ContactIsOwnerAuthorizationHandler`使用 [Identity](xref:security/authentication/identity) 在 Entity Framework Core 上构建 ASP.NET Core。 向服务集合注册处理程序，使其可 `ContactsController` 通过 [依赖关系注入](xref:fundamentals/dependency-injection)获得。 将以下代码添加到的末尾 `ConfigureServices` ：

[!code-csharp[](secure-data/samples/final3/Startup.cs?name=snippet_defaultPolicy&highlight=23-99)]

`ContactAdministratorsAuthorizationHandler` 和 `ContactManagerAuthorizationHandler` 将添加为单一实例。 它们是单一实例的，因为它们不使用 EF，并且所需的所有信息都在 `Context` 方法的参数中 `HandleRequirementAsync` 。

## <a name="support-authorization"></a>支持授权

在本部分中，将更新 Razor 页面并添加操作要求类。

### <a name="review-the-contact-operations-requirements-class"></a>查看联系操作要求类

查看 `ContactOperations` 类。 此类包含应用支持的要求：

[!code-csharp[](secure-data/samples/final3/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-no-locrazor-pages"></a>为联系人页创建基类 Razor

创建一个包含联系人页中使用的服务的基类 Razor 。 基类将初始化代码放在一个位置：

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/DI_BasePageModel.cs)]

前面的代码：

* 添加 `IAuthorizationService` 服务以访问授权处理程序。
* 添加 Identity `UserManager` 服务。
* 添加 `ApplicationDbContext`。

### <a name="update-the-createmodel"></a>更新 CreateModel

更新 "创建页模型" 构造函数以使用 `DI_BasePageModel` 基类：

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

将 `CreateModel.OnPostAsync` 方法更新为：

* 将用户 ID 添加到 `Contact` 模型。
* 调用授权处理程序以验证用户是否有权创建联系人。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a>更新 IndexModel

更新 `OnGetAsync` 方法以便仅向一般用户显示已批准的联系人：

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a>更新 EditModel

添加一个授权处理程序来验证用户是否拥有该联系人。 由于正在验证资源授权，因此 `[Authorize]` 属性不够。 计算属性时，应用无法访问资源。 基于资源的授权必须是必需的。 如果应用有权访问该资源，则必须执行检查，方法是将其加载到页面模型中，或在处理程序本身中加载它。 通过传入资源键，可以频繁地访问资源。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a>更新 DeleteModel

更新 "删除" 页模型，以使用授权处理程序来验证用户是否具有对联系人的 "删除" 权限。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a>将授权服务注入视图

目前，UI 会显示用户不能修改的联系人的编辑和删除链接。

将授权服务注入 *Pages/_ViewImports cshtml* 文件，使其可供所有视图使用：

[!code-cshtml[](secure-data/samples/final3/Pages/_ViewImports.cshtml?highlight=6-99)]

前面的标记添加了多个 `using` 语句。

更新 *页面/联系人/索引* 中的 " **编辑** " 和 " **删除** " 链接，以便仅为具有适当权限的用户呈现它们：

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> 隐藏不具有更改数据权限的用户的链接不会保护应用的安全。 隐藏链接使应用程序更易于用户理解，只显示有效的链接。 用户可以通过攻击生成的 Url 来对其不拥有的数据调用编辑和删除操作。 Razor页或控制器必须强制进行访问检查以确保数据的安全。

### <a name="update-details"></a>更新详细信息

更新详细信息视图，以便经理可以批准或拒绝联系人：

[!code-cshtml[](secure-data/samples/final3/Pages/Contacts/Details.cshtml?name=snippet)]

更新详细信息页模型：

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a>在角色中添加或删除用户

有关信息，请参阅 [此问题](https://github.com/dotnet/AspNetCore.Docs/issues/8502) ：

* 正在删除用户的权限。 例如，在聊天应用中对用户进行静音。
* 向用户添加特权。

<a name="challenge"></a>

## <a name="differences-between-challenge-and-forbid"></a>质询和禁止之间的差异

此应用将默认策略设置为 " [需要经过身份验证的用户](#rau)"。 以下代码允许匿名用户。 允许匿名用户显示质询与禁止之间的差异。

[!code-csharp[](secure-data/samples/final3/Pages/Contacts/Details2.cshtml.cs?name=snippet)]

在上述代码中：

* 如果用户 **未通过身份** 验证， `ChallengeResult` 则返回。 返回后 `ChallengeResult` ，会将用户重定向到登录页。
* 如果用户已通过身份验证，但未获得授权， `ForbidResult` 则返回。 返回后 `ForbidResult` ，会将用户重定向到 "拒绝访问" 页。

## <a name="test-the-completed-app"></a>测试已完成的应用程序

如果尚未为种子设定用户帐户设置密码，请使用 [机密管理器工具](xref:security/app-secrets#secret-manager) 设置密码：

* 选择强密码：使用八个或更多字符，并且至少使用一个大写字符、数字和符号。 例如， `Passw0rd!` 满足强密码要求。
* 从项目的文件夹中执行以下命令，其中 `<PW>` 是密码：

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

如果应用有联系人：

* 删除表中的所有记录 `Contact` 。
* 重新启动应用以对数据库进行种子设定。

测试已完成应用程序的一种简单方法是启动三个不同的浏览器 (或 incognito/InPrivate 会话) 。 在一个浏览器中，注册新用户 (例如 `test@example.com`) 。 使用其他用户登录到每个浏览器。 验证下列操作：

* 已注册的用户可以查看所有已批准的联系人数据。
* 已注册的用户可以编辑/删除他们自己的数据。
* 经理可以批准/拒绝联系人数据。 此 `Details` 视图显示 " **批准** " 和 " **拒绝** " 按钮。
* 管理员可以批准/拒绝和编辑/删除所有数据。

| 用户                | 应用程序的种子 | 选项                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | 否                | 编辑/删除自己的数据。                |
| manager@contoso.com | 是               | 批准/拒绝和编辑/删除自己的数据。 |
| admin@contoso.com   | 是               | 批准/拒绝和编辑/删除所有数据。 |

在管理员的浏览器中创建联系人。 复制管理员联系人的 "删除" 和 "编辑" 的 URL。 将这些链接粘贴到测试用户的浏览器中，以验证测试用户是否无法执行这些操作。

## <a name="create-the-starter-app"></a>创建初学者应用

* 创建 Razor 名为 "ContactManager" 的页面应用
  * 创建具有 **单个用户帐户** 的应用。
  * 将其命名为 "ContactManager"，使命名空间与该示例中使用的命名空间匹配。
  * `-uld` 指定 LocalDB 而不是 SQLite

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* 添加 *模型/联系方式* ：

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* 基架 `Contact` 模型。
* 创建初始迁移并更新数据库：

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
dotnet ef database drop -f
dotnet ef migrations add initial
dotnet ef database update
```

如果使用命令遇到 bug `dotnet aspnet-codegenerator razorpage` ，请参阅 [此 GitHub 问题](https://github.com/aspnet/Scaffolding/issues/984)。

* 更新 *Pages/Shared/_Layout cshtml* 文件中的 **ContactManager** 定位点：

 ```cshtml
<a class="navbar-brand" asp-area="" asp-page="/Contacts/Index">ContactManager</a>
  ```

* 通过创建、编辑和删除联系人来测试应用

### <a name="seed-the-database"></a>设定数据库种子

将 [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/starter3/Data/SeedData.cs) 类添加到 *Data* 文件夹：

[!code-csharp[](secure-data/samples/starter3/Data/SeedData.cs)]

调用 `SeedData.Initialize` 自 `Main` ：

[!code-csharp[](secure-data/samples/starter3/Program.cs)]

测试该应用是否为该数据库的种子。 如果 contact DB 中存在任何行，则 seed 方法不会运行。

::: moniker-end

::: moniker range=">= aspnetcore-2.1 < aspnetcore-3.0"

本教程演示如何创建 ASP.NET Core 的 web 应用，其中包含由授权保护的用户数据。 它将显示已进行身份验证 (已创建的已注册) 用户的联系人列表。 有三个安全组：

* **已注册的用户** 可以查看所有已批准的数据，并可以编辑/删除他们自己的数据。
* **经理** 可以批准或拒绝联系人数据。 只有已批准的联系人对用户可见。
* **管理员** 可以批准/拒绝和编辑/删除任何数据。

在下图中，用户 Rick (`rick@example.com`) 已登录。 Rick 只能查看已批准的联系人， **编辑** / **删除** / 为其联系人 **创建新** 链接。 只有 Rick 创建的最后一条记录才会显示 " **编辑** " 和 " **删除** " 链接。 在经理或管理员将状态更改为 "已批准" 之前，其他用户将看不到最后一条记录。

![显示已登录的 Rick 的屏幕截图](secure-data/_static/rick.png)

在下图中， `manager@contoso.com` 已登录，并在管理器的角色中：

![显示 manager@contoso.com 已登录的屏幕截图](secure-data/_static/manager1.png)

下图显示了联系人的经理详细信息视图：

![联系人的经理视图](secure-data/_static/manager.png)

" **批准** " 和 " **拒绝** " 按钮仅为经理和管理员显示。

在下图中，以 `admin@contoso.com` 管理员的角色登录和：

![显示 admin@contoso.com 已登录的屏幕截图](secure-data/_static/admin.png)

管理员具有所有权限。 她可以读取/编辑/删除任何联系人并更改联系人的状态。

此应用是通过 [基架](xref:tutorials/first-mvc-app/adding-model#scaffold-the-movie-model) 创建的 `Contact` ：以下模型：

[!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

该示例包含以下授权处理程序：

* `ContactIsOwnerAuthorizationHandler`：确保用户只能编辑其数据。
* `ContactManagerAuthorizationHandler`：允许经理批准或拒绝联系人。
* `ContactAdministratorsAuthorizationHandler`：允许管理员批准或拒绝联系人以及编辑/删除联系人。

## <a name="prerequisites"></a>先决条件

本教程是高级教程。 你应该熟悉：

* [ASP.NET Core](xref:tutorials/first-mvc-app/start-mvc)
* [身份验证](xref:security/authentication/identity)
* [帐户确认和密码恢复](xref:security/authentication/accconfirm)
* [授权](xref:security/authorization/introduction)
* [Entity Framework Core](xref:data/ef-mvc/intro)

## <a name="the-starter-and-completed-app"></a>入门和已完成的应用程序

[下载](xref:index#how-to-download-a-sample)[已完成](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples)的应用。 [测试](#test-the-completed-app) 已完成的应用程序，使其安全功能熟悉。

### <a name="the-starter-app"></a>入门应用

[下载](xref:index#how-to-download-a-sample)[初学者](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/)应用。

运行应用程序，点击 " **ContactManager** " 链接，并验证是否可以创建、编辑和删除联系人。

## <a name="secure-user-data"></a>保护用户数据

以下部分包含创建安全用户数据应用的所有主要步骤。 你可能会发现，引用已完成的项目非常有用。

### <a name="tie-the-contact-data-to-the-user"></a>将联系人数据与用户关联

使用 ASP.NET [Identity](xref:security/authentication/identity) 用户 ID 可确保用户能够编辑其数据，而不是其他用户数据。 将 `OwnerID` 和添加 `ContactStatus` 到 `Contact` 模型：

[!code-csharp[](secure-data/samples/final2.1/Models/Contact.cs?name=snippet1&highlight=5-6,16-999)]

`OwnerID` 数据库中的表的用户 ID `AspNetUser` [Identity](xref:security/authentication/identity) 。 此 `Status` 字段确定常规用户是否可查看联系人。

创建新的迁移并更新数据库：

```dotnetcli
dotnet ef migrations add userID_Status
dotnet ef database update
```

### <a name="add-role-services-to-no-locidentity"></a>将角色服务添加到 Identity

追加 [AddRoles](/dotnet/api/microsoft.aspnetcore.identity.identitybuilder.addroles#Microsoft_AspNetCore_Identity_IdentityBuilder_AddRoles__1) 以添加角色服务：

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet2&highlight=11)]

### <a name="require-authenticated-users"></a>需要经过身份验证的用户

将默认的 "身份验证策略" 设置为 "要求用户进行身份验证"：

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet&highlight=17-99)] 

 您可以 Razor 通过属性在页、控制器或操作方法级别选择不进行身份验证 `[AllowAnonymous]` 。 将默认身份验证策略设置为 "要求用户进行身份验证" 可保护新添加的 Razor 页面和控制器。 默认情况下，需要进行身份验证比依赖新控制器和 Razor 页面以包括属性更安全 `[Authorize]` 。

将 [AllowAnonymous](/dotnet/api/microsoft.aspnetcore.authorization.allowanonymousattribute) 添加到 "索引"、"关于" 和 "联系人" 页，以便匿名用户在注册之前可以获取有关站点的信息。

[!code-csharp[](secure-data/samples/final2.1/Pages/Index.cshtml.cs?highlight=1,6)]

### <a name="configure-the-test-account"></a>配置测试帐户

`SeedData`类创建两个帐户：管理员和管理器。 使用 [机密管理器工具](xref:security/app-secrets) 来设置这些帐户的密码。 将项目目录中的密码设置 (包含 *Program.cs* ) 的目录：

```dotnetcli
dotnet user-secrets set SeedUserPW <PW>
```

如果未指定强密码，则在调用时会引发异常 `SeedData.Initialize` 。

更新 `Main` 以使用测试密码：

[!code-csharp[](secure-data/samples/final2.1/Program.cs?name=snippet)]

### <a name="create-the-test-accounts-and-update-the-contacts"></a>创建测试帐户并更新联系人

更新 `Initialize` 类中的方法 `SeedData` ，以创建测试帐户：

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet_Initialize)]

向联系人添加管理员用户 ID 和 `ContactStatus` 。 使其中一个联系人 "已提交" 和一个 "已拒绝"。 将用户 ID 和状态添加到所有联系人。 只显示一个联系人：

[!code-csharp[](secure-data/samples/final2.1/Data/SeedData.cs?name=snippet1&highlight=17,18)]

## <a name="create-owner-manager-and-administrator-authorization-handlers"></a>创建所有者、经理和管理员授权处理程序

创建一个 *授权* 文件夹并 `ContactIsOwnerAuthorizationHandler` 在其中创建一个类。 `ContactIsOwnerAuthorizationHandler`验证对资源的用户是否拥有该资源。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactIsOwnerAuthorizationHandler.cs)]

`ContactIsOwnerAuthorizationHandler`调用[上下文。](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.succeed#Microsoft_AspNetCore_Authorization_AuthorizationHandlerContext_Succeed_Microsoft_AspNetCore_Authorization_IAuthorizationRequirement_)如果当前经过身份验证的用户是联系人所有者，则会成功。 授权处理程序通常：

* `context.Succeed`满足要求时返回。
* `Task.CompletedTask`当不满足要求时返回。 `Task.CompletedTask` 不是成功或失败， &mdash; 它允许其他授权处理程序运行。

如果需要显式失败，请返回 [context。失败](/dotnet/api/microsoft.aspnetcore.authorization.authorizationhandlercontext.fail)。

该应用程序允许联系人所有者编辑/删除/创建他们自己的数据。 `ContactIsOwnerAuthorizationHandler` 不需要检查在要求参数中传递的操作。

### <a name="create-a-manager-authorization-handler"></a>创建管理器授权处理程序

`ContactManagerAuthorizationHandler`在 *Authorization* 文件夹中创建一个类。 `ContactManagerAuthorizationHandler`验证对资源的用户是否为管理员。 只有管理人员才能 (新的或更改的) 批准或拒绝内容更改。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactManagerAuthorizationHandler.cs)]

### <a name="create-an-administrator-authorization-handler"></a>创建管理员授权处理程序

`ContactAdministratorsAuthorizationHandler`在 *Authorization* 文件夹中创建一个类。 `ContactAdministratorsAuthorizationHandler`验证对资源的用户是否为管理员。 管理员可以执行所有操作。

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactAdministratorsAuthorizationHandler.cs)]

## <a name="register-the-authorization-handlers"></a>注册授权处理程序

Entity Framework Core 使用 AddScoped 的服务必须使用[AddScoped](/dotnet/api/microsoft.extensions.dependencyinjection.servicecollectionserviceextensions)注册以进行[依赖关系注入](xref:fundamentals/dependency-injection)。 `ContactIsOwnerAuthorizationHandler`使用 [Identity](xref:security/authentication/identity) 在 Entity Framework Core 上构建 ASP.NET Core。 向服务集合注册处理程序，使其可 `ContactsController` 通过 [依赖关系注入](xref:fundamentals/dependency-injection)获得。 将以下代码添加到的末尾 `ConfigureServices` ：

[!code-csharp[](secure-data/samples/final2.1/Startup.cs?name=snippet_defaultPolicy&highlight=27-99)]

`ContactAdministratorsAuthorizationHandler` 和 `ContactManagerAuthorizationHandler` 将添加为单一实例。 它们是单一实例的，因为它们不使用 EF，并且所需的所有信息都在 `Context` 方法的参数中 `HandleRequirementAsync` 。

## <a name="support-authorization"></a>支持授权

在本部分中，将更新 Razor 页面并添加操作要求类。

### <a name="review-the-contact-operations-requirements-class"></a>查看联系操作要求类

查看 `ContactOperations` 类。 此类包含应用支持的要求：

[!code-csharp[](secure-data/samples/final2.1/Authorization/ContactOperations.cs)]

### <a name="create-a-base-class-for-the-contacts-no-locrazor-pages"></a>为联系人页创建基类 Razor

创建一个包含联系人页中使用的服务的基类 Razor 。 基类将初始化代码放在一个位置：

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/DI_BasePageModel.cs)]

前面的代码：

* 添加 `IAuthorizationService` 服务以访问授权处理程序。
* 添加 Identity `UserManager` 服务。
* 添加 `ApplicationDbContext`。

### <a name="update-the-createmodel"></a>更新 CreateModel

更新 "创建页模型" 构造函数以使用 `DI_BasePageModel` 基类：

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippetCtor)]

将 `CreateModel.OnPostAsync` 方法更新为：

* 将用户 ID 添加到 `Contact` 模型。
* 调用授权处理程序以验证用户是否有权创建联系人。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Create.cshtml.cs?name=snippet_Create)]

### <a name="update-the-indexmodel"></a>更新 IndexModel

更新 `OnGetAsync` 方法以便仅向一般用户显示已批准的联系人：

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml.cs?name=snippet)]

### <a name="update-the-editmodel"></a>更新 EditModel

添加一个授权处理程序来验证用户是否拥有该联系人。 由于正在验证资源授权，因此 `[Authorize]` 属性不够。 计算属性时，应用无法访问资源。 基于资源的授权必须是必需的。 如果应用有权访问该资源，则必须执行检查，方法是将其加载到页面模型中，或在处理程序本身中加载它。 通过传入资源键，可以频繁地访问资源。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Edit.cshtml.cs?name=snippet)]

### <a name="update-the-deletemodel"></a>更新 DeleteModel

更新 "删除" 页模型，以使用授权处理程序来验证用户是否具有对联系人的 "删除" 权限。

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Delete.cshtml.cs?name=snippet)]

## <a name="inject-the-authorization-service-into-the-views"></a>将授权服务注入视图

目前，UI 会显示用户不能修改的联系人的编辑和删除链接。

将授权服务注入到 *views/_ViewImports cshtml* 文件中，使其可供所有视图使用：

[!code-cshtml[](secure-data/samples/final2.1/Pages/_ViewImports.cshtml?highlight=6-99)]

前面的标记添加了多个 `using` 语句。

更新 *页面/联系人/索引* 中的 " **编辑** " 和 " **删除** " 链接，以便仅为具有适当权限的用户呈现它们：

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Index.cshtml?highlight=34-36,62-999)]

> [!WARNING]
> 隐藏不具有更改数据权限的用户的链接不会保护应用的安全。 隐藏链接使应用程序更易于用户理解，只显示有效的链接。 用户可以通过攻击生成的 Url 来对其不拥有的数据调用编辑和删除操作。 Razor页或控制器必须强制进行访问检查以确保数据的安全。

### <a name="update-details"></a>更新详细信息

更新详细信息视图，以便经理可以批准或拒绝联系人：

[!code-cshtml[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml?name=snippet)]

更新详细信息页模型：

[!code-csharp[](secure-data/samples/final2.1/Pages/Contacts/Details.cshtml.cs?name=snippet)]

## <a name="add-or-remove-a-user-to-a-role"></a>在角色中添加或删除用户

有关信息，请参阅 [此问题](https://github.com/dotnet/AspNetCore.Docs/issues/8502) ：

* 正在删除用户的权限。 例如，在聊天应用中对用户进行静音。
* 向用户添加特权。

## <a name="test-the-completed-app"></a>测试已完成的应用程序

如果尚未为种子设定用户帐户设置密码，请使用 [机密管理器工具](xref:security/app-secrets#secret-manager) 设置密码：

* 选择强密码：使用八个或更多字符，并且至少使用一个大写字符、数字和符号。 例如， `Passw0rd!` 满足强密码要求。
* 从项目的文件夹中执行以下命令，其中 `<PW>` 是密码：

  ```dotnetcli
  dotnet user-secrets set SeedUserPW <PW>
  ```

* 删除和更新数据库

  ```dotnetcli
  dotnet ef database drop -f
  dotnet ef database update  
  ```

* 重新启动应用以对数据库进行种子设定。

测试已完成应用程序的一种简单方法是启动三个不同的浏览器 (或 incognito/InPrivate 会话) 。 在一个浏览器中，注册新用户 (例如 `test@example.com`) 。 使用其他用户登录到每个浏览器。 验证下列操作：

* 已注册的用户可以查看所有已批准的联系人数据。
* 已注册的用户可以编辑/删除他们自己的数据。
* 经理可以批准/拒绝联系人数据。 此 `Details` 视图显示 " **批准** " 和 " **拒绝** " 按钮。
* 管理员可以批准/拒绝和编辑/删除所有数据。

| 用户                | 应用程序的种子 | 选项                                  |
| ------------------- | :---------------: | ---------------------------------------- |
| test@example.com    | 否                | 编辑/删除自己的数据。                |
| manager@contoso.com | 是               | 批准/拒绝和编辑/删除自己的数据。 |
| admin@contoso.com   | 是               | 批准/拒绝和编辑/删除所有数据。 |

在管理员的浏览器中创建联系人。 复制管理员联系人的 "删除" 和 "编辑" 的 URL。 将这些链接粘贴到测试用户的浏览器中，以验证测试用户是否无法执行这些操作。

## <a name="create-the-starter-app"></a>创建初学者应用

* 创建 Razor 名为 "ContactManager" 的页面应用
  * 创建具有 **单个用户帐户** 的应用。
  * 将其命名为 "ContactManager"，使命名空间与该示例中使用的命名空间匹配。
  * `-uld` 指定 LocalDB 而不是 SQLite

  ```dotnetcli
  dotnet new webapp -o ContactManager -au Individual -uld
  ```

* 添加 *模型/联系方式* ：

  [!code-csharp[](secure-data/samples/starter2.1/Models/Contact.cs?name=snippet1)]

* 基架 `Contact` 模型。
* 创建初始迁移并更新数据库：

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Contact -udl -dc ApplicationDbContext -outDir Pages\Contacts --referenceScriptLibraries
  dotnet ef database drop -f
  dotnet ef migrations add initial
  dotnet ef database update
  ```

* 更新 *Pages/_Layout cshtml* 文件中的 **ContactManager** 定位点：

  ```cshtml
  <a asp-page="/Contacts/Index" class="navbar-brand">ContactManager</a>
  ```

* 通过创建、编辑和删除联系人来测试应用

### <a name="seed-the-database"></a>设定数据库种子

将 [SeedData](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authorization/secure-data/samples/starter2.1/Data/SeedData.cs) 类添加到 *Data* 文件夹中。

调用 `SeedData.Initialize` 自 `Main` ：

[!code-csharp[](secure-data/samples/starter2.1/Program.cs?name=snippet)]

测试该应用是否为该数据库的种子。 如果 contact DB 中存在任何行，则 seed 方法不会运行。

::: moniker-end

<a name="secure-data-add-resources-label"></a>

### <a name="additional-resources"></a>其他资源

* [在 Azure 应用服务中生成 .NET Core 和 SQL 数据库 Web 应用](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* [ASP.NET Core 授权实验室](https://github.com/blowdart/AspNetAuthorizationWorkshop)。 此实验室更详细地介绍了本教程中所介绍的安全功能。
* <xref:security/authorization/introduction>
* [自定义基于策略的授权](xref:security/authorization/policies)
