---
title: 自定义存储提供程序 ASP.NET Core Identity
author: ardalis
description: 了解如何为配置自定义存储提供程序 ASP.NET Core Identity 。
ms.author: riande
ms.custom: mvc
ms.date: 07/23/2019
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
uid: security/authentication/identity-custom-storage-providers
ms.openlocfilehash: c89098bf0b2c4396f9856aca2be9967af5df0cb7
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051897"
---
# <a name="custom-storage-providers-for-no-locaspnet-core-identity"></a>自定义存储提供程序 ASP.NET Core Identity

作者：[Steve Smith](https://ardalis.com/)

ASP.NET Core Identity 是一个可扩展系统，可让你创建自定义存储提供程序并将其连接到你的应用程序。 本主题介绍如何为创建自定义的存储提供程序 ASP.NET Core Identity 。 它介绍了用于创建自己的存储提供程序的重要概念，但并不是分步演练。

[查看或下载 GitHub 中的示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/authentication/identity-custom-storage-providers/sample/CustomIdentityProviderSample)。

## <a name="introduction"></a>简介

默认情况下， ASP.NET Core Identity 系统使用 Entity Framework Core 将用户信息存储在 SQL Server 数据库中。 对于许多应用程序而言，这种方法的效果很好。 但是，你可能希望使用不同的持久性机制或数据架构。 例如： 。

* 使用 [Azure 表存储](/azure/storage/) 或其他数据存储。
* 数据库表具有不同的结构。 
* 你可能想要使用不同的数据访问方法，例如 [Dapper](https://github.com/StackExchange/Dapper)。 

在上述每种情况下，都可以为存储机制编写自定义的提供程序，并将该提供程序插入到应用程序中。

ASP.NET Core Identity 包含在 Visual Studio 中具有 "单独用户帐户" 选项的项目模板中。

使用 .NET Core CLI 时，请添加 `-au Individual` ：

```dotnetcli
dotnet new mvc -au Individual
```

## <a name="the-no-locaspnet-core-identity-architecture"></a>ASP.NET Core Identity体系结构

ASP.NET Core Identity 由称为管理器和存储的类组成。 *管理* 层是应用程序开发人员用来执行操作（如创建用户）的高级类 Identity 。 *存储* 是用于指定如何保存实体（如用户和角色）的较低级别类。 存储遵循存储库模式，并与持久性机制紧密耦合。 管理器与存储分离，这意味着，你可以在不更改应用程序代码的情况下替换持久性机制， (配置) 除外。

下图显示了 web 应用如何与管理器进行交互，同时存储与数据访问层交互。

![ASP.NET Core 应用使用管理器 (例如 "UserManager"、"RoleManager") 。 管理器使用商店 (例如，"UserStore") 使用库（如 Entity Framework Core）与数据源进行通信。](identity-custom-storage-providers/_static/identity-architecture-diagram.png)

若要创建自定义存储提供程序，请创建数据源、数据访问层以及与此数据访问层交互的存储类， (上图中的绿色和灰色框) 。 不需要自定义与它们交互的管理器或应用代码 () 上方的蓝色框。

在创建的新实例时， `UserManager` 或 `RoleManager` 提供 user 类的类型并将 store 类的实例作为参数传递。 此方法使你能够将自定义类插入 ASP.NET Core。 

[重新配置应用程序以使用新的存储提供程序](#reconfigure-app-to-use-a-new-storage-provider) 演示如何实例化 `UserManager` 和 `RoleManager` 使用自定义存储。

## <a name="no-locaspnet-core-identity-stores-data-types"></a>ASP.NET Core Identity 存储数据类型

[ASP.NET Core Identity](https://github.com/aspnet/identity) 以下部分详细介绍了数据类型：

### <a name="users"></a>用户

网站的已注册用户。 可以扩展[ Identity 用户](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuser)类型，也可以将其用作你自己的自定义类型的示例。 无需从特定类型继承即可实现您自己的自定义标识存储解决方案。

### <a name="user-claims"></a>用户声明

一组语句 (或 [声明](/dotnet/api/system.security.claims.claim) 表示用户身份的用户) 。 可以启用用户标识的更大表达式，而不能通过角色来实现。

### <a name="user-logins"></a>用户登录

有关外部身份验证提供程序的信息 (如 Facebook 或 Microsoft 帐户) 在用户登录时使用。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuserlogin)

### <a name="roles"></a>角色

站点的授权组。 包括角色 Id 和角色名称 (如 "管理员" 或 "Employee" ) 。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.identityrole)

## <a name="the-data-access-layer"></a>数据访问层

本主题假定您熟悉要使用的持久性机制以及如何创建该机制的实体。 本主题不提供有关如何创建存储库或数据访问类的详细信息;在使用时，它提供了有关设计决策的一些建议 ASP.NET Core Identity 。

设计自定义存储提供程序的数据访问层时，有很多自由。 你只需为你打算在应用程序中使用的功能创建持久性机制。 例如，如果你未在应用中使用角色，则无需为角色或用户角色关联创建存储。 你的技术和现有基础结构可能需要与的默认实现非常不同的结构 ASP.NET Core Identity 。 在数据访问层中，提供用于处理存储实现结构的逻辑。

数据访问层提供了将数据保存 ASP.NET Core Identity 到数据源的逻辑。 自定义存储提供程序的数据访问层可能包含以下类来存储用户和角色信息。

### <a name="context-class"></a>Context 类

封装信息以连接到永久性机制并执行查询。 几个数据类需要此类的实例，通常通过依赖关系注入来提供此类实例。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.identitydbcontext-1)。

### <a name="user-storage"></a>用户存储

存储和检索用户信息 (例如用户名和密码哈希) 。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

### <a name="role-storage"></a>角色存储

存储和检索角色信息 (如角色名称) 。 [示例](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.rolestore-1)

### <a name="userclaims-storage"></a>UserClaims 存储

存储和检索用户声明信息 (如声明类型和值) 。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

### <a name="userlogins-storage"></a>UserLogins 存储

存储和检索用户登录信息， (例如外部身份验证提供程序) 。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

### <a name="userrole-storage"></a>UserRole 存储

存储和检索为哪些用户分配了哪些角色。 [示例](/dotnet/api/microsoft.aspnet.identity.corecompat.userstore-1)

**提示：** 仅实现你打算在应用程序中使用的类。

在数据访问类中，提供代码来执行持久性机制的数据操作。 例如，在自定义提供程序中，你可能具有以下代码，以便在 *store* 类中创建新用户：

[!code-csharp[](identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/CustomUserStore.cs?name=createuser&highlight=7)]

用于创建用户的实现逻辑位于 `_usersTable.CreateAsync` 方法中，如下所示。

## <a name="customize-the-user-class"></a>自定义用户类

实现存储提供程序时，创建一个与[ Identity 用户类](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuser)等效的用户类。

用户类至少必须包括 `Id` 和 `UserName` 属性。

`IdentityUser`类定义 `UserManager` 执行请求的操作时调用的属性。 属性的默认类型 `Id` 是字符串，但您可以从继承 `IdentityUser<TKey, TUserClaim, TUserRole, TUserLogin, TUserToken>` 并指定其他类型。 框架需要存储实现来处理数据类型转换。

## <a name="customize-the-user-store"></a>自定义用户存储

创建一个 `UserStore` 类，该类提供针对用户的所有数据操作的方法。 此类等效于[UserStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.userstore-1)类。 在 `UserStore` 类中，实现 `IUserStore<TUser>` 和所需的可选接口。 根据应用中提供的功能，选择要实现的可选接口。

### <a name="optional-interfaces"></a>可选接口

* [IUserRoleStore](/dotnet/api/microsoft.aspnetcore.identity.iuserrolestore-1)
* [IUserClaimStore](/dotnet/api/microsoft.aspnetcore.identity.iuserclaimstore-1)
* [IUserPasswordStore](/dotnet/api/microsoft.aspnetcore.identity.iuserpasswordstore-1)
* [IUserSecurityStampStore](/dotnet/api/microsoft.aspnetcore.identity.iusersecuritystampstore-1)
* [IUserEmailStore](/dotnet/api/microsoft.aspnetcore.identity.iuseremailstore-1)
* [IUserPhoneNumberStore](/dotnet/api/microsoft.aspnetcore.identity.iuserphonenumberstore-1)
* [IQueryableUserStore](/dotnet/api/microsoft.aspnetcore.identity.iqueryableuserstore-1)
* [IUserLoginStore](/dotnet/api/microsoft.aspnetcore.identity.iuserloginstore-1)
* [IUserTwoFactorStore](/dotnet/api/microsoft.aspnetcore.identity.iusertwofactorstore-1)
* [IUserLockoutStore](/dotnet/api/microsoft.aspnetcore.identity.iuserlockoutstore-1)

可选接口继承自 `IUserStore<TUser>` 。 可以在 [示例应用](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/security/authentication/identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/CustomUserStore.cs)中查看部分实现的示例用户存储。

在 `UserStore` 类中，可以使用您创建的数据访问类来执行操作。 使用依赖关系注入传入它们。 例如，在使用 Dapper 实现的 SQL Server 中， `UserStore` 类具有方法，该 `CreateAsync` 方法使用的实例 `DapperUsersTable` 插入新记录：

[!code-csharp[](identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/DapperUsersTable.cs?name=createuser&highlight=7)]

### <a name="interfaces-to-implement-when-customizing-user-store"></a>自定义用户存储时要实现的接口

* **IUserStore**  
 [IUserStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserstore-1)接口是必须在用户存储中实现的唯一接口。 它定义用于创建、更新、删除和检索用户的方法。
* **IUserClaimStore**  
 [IUserClaimStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserclaimstore-1)接口定义为启用用户声明而实现的方法。 它包含添加、删除和检索用户声明的方法。
* **IUserLoginStore**  
 [IUserLoginStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserloginstore-1)定义用于启用外部身份验证提供程序的方法。 它包含添加、删除和检索用户登录名的方法，以及用于根据登录信息检索用户的方法。
* **IUserRoleStore**  
 [IUserRoleStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserrolestore-1)接口定义你实现的方法，用于将用户映射到角色。 它包含添加、删除和检索用户角色的方法，以及用于检查是否向用户分配了角色的方法。
* **IUserPasswordStore**  
 [IUserPasswordStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserpasswordstore-1)接口定义你实现的用于保存哈希密码的方法。 它包含用于获取和设置哈希密码的方法，以及一个指示用户是否已设置密码的方法。
* **IUserSecurityStampStore**  
 [IUserSecurityStampStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iusersecuritystampstore-1)接口定义你实现的方法，以使用安全戳记来指示用户的帐户信息是否已更改。 当用户更改密码或添加或删除登录名时，此戳记会更新。 它包含用于获取和设置安全标记的方法。
* **IUserTwoFactorStore**  
 [IUserTwoFactorStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iusertwofactorstore-1)接口定义你实现的方法，以支持双重身份验证。 它包含的方法可用于获取和设置是否为用户启用了双因素身份验证。
* **IUserPhoneNumberStore**  
 [IUserPhoneNumberStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserphonenumberstore-1)接口定义你为存储用户电话号码而实现的方法。 它包含的方法可用于获取和设置电话号码，以及电话号码是否已确认。
* **IUserEmailStore**  
 [IUserEmailStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuseremailstore-1)接口定义你实现的用于存储用户电子邮件地址的方法。 它包含用于获取和设置电子邮件地址以及是否已确认电子邮件的方法。
* **IUserLockoutStore**  
 [IUserLockoutStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iuserlockoutstore-1)接口定义用于存储有关锁定帐户的信息的方法。 它包含用于跟踪失败的访问尝试和锁定的方法。
* **IQueryableUserStore**  
 [IQueryableUserStore &lt; TUser &gt; ](/dotnet/api/microsoft.aspnetcore.identity.iqueryableuserstore-1)接口定义为提供可查询用户存储而实现的成员。

只需实现应用程序中所需的接口。 例如： 。

```csharp
public class UserStore : IUserStore<IdentityUser>,
                         IUserClaimStore<IdentityUser>,
                         IUserLoginStore<IdentityUser>,
                         IUserRoleStore<IdentityUser>,
                         IUserPasswordStore<IdentityUser>,
                         IUserSecurityStampStore<IdentityUser>
{
    // interface implementations not shown
}
```

### <a name="no-locidentityuserclaim-no-locidentityuserlogin-and-no-locidentityuserrole"></a>IdentityUserClaim、 Identity UserLogin 和 Identity UserRole

`Microsoft.AspNet.Identity.EntityFramework`命名空间包含[ Identity UserClaim](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.identityuserclaim-1)、 [ Identity UserLogin](/dotnet/api/microsoft.aspnet.identity.corecompat.identityuserlogin)和[ Identity UserRole](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.identityuserrole-1)类的实现。 如果使用这些功能，可能需要创建自己版本的这些类并定义应用的属性。 但是，在执行基本操作时，如果不将这些实体加载到内存中， (例如添加或删除用户的声明) ，则效率更高。 而后端存储类可以直接对数据源执行这些操作。 例如， `UserStore.GetClaimsAsync` 方法可以调用 `userClaimTable.FindByUserId(user.Id)` 方法来对该表直接执行查询，并返回声明列表。

## <a name="customize-the-role-class"></a>自定义 role 类

实现角色存储提供程序时，可以创建自定义角色类型。 它不需要实现特定接口，但它必须具有 `Id` ，并且通常具有一个 `Name` 属性。

下面是一个示例 role 类：

[!code-csharp[](identity-custom-storage-providers/sample/CustomIdentityProviderSample/CustomProvider/ApplicationRole.cs)]

## <a name="customize-the-role-store"></a>自定义角色存储

你可以创建一个 `RoleStore` 类，该类提供针对角色的所有数据操作的方法。 此类等效于[RoleStore &lt; TRole &gt; ](/dotnet/api/microsoft.aspnetcore.identity.entityframeworkcore.rolestore-1)类。 在 `RoleStore` 类中，你可以实现 `IRoleStore<TRole>` 和（可选） `IQueryableRoleStore<TRole>` 接口。

* **IRoleStore &lt; TRole&gt;**  
 [IRoleStore &lt; TRole &gt; ](/dotnet/api/microsoft.aspnetcore.identity.irolestore-1)接口定义要在角色存储区类中实现的方法。 它包含用于创建、更新、删除和检索角色的方法。
* **RoleStore &lt; TRole&gt;**  
 若要自定义 `RoleStore` ，请创建实现接口的类 `IRoleStore<TRole>` 。 

## <a name="reconfigure-app-to-use-a-new-storage-provider"></a>重新配置应用程序以使用新的存储提供程序

实现存储提供程序后，可将应用程序配置为使用该访问接口。 如果你的应用程序使用了默认提供程序，请将其替换为你的自定义提供程序。

1. 删除 `Microsoft.AspNetCore.EntityFramework.Identity` NuGet 包。
1. 如果存储提供程序驻留在单独的项目或包中，请添加对它的引用。
1. 将所有对的引用替换为 `Microsoft.AspNetCore.EntityFramework.Identity` 存储提供程序的命名空间的 using 语句。
1. 在 `ConfigureServices` 方法中，将 `AddIdentity` 方法更改为使用您的自定义类型。 出于此目的，你可以创建自己的扩展方法。 有关示例，请参阅[ Identity ServiceCollectionExtensions](https://github.com/aspnet/Identity/blob/rel/1.1.0/src/Microsoft.AspNetCore.Identity/IdentityServiceCollectionExtensions.cs) 。
1. 如果使用的是角色，请更新 `RoleManager` 以使用 `RoleStore` 类。
1. 更新应用程序配置的连接字符串和凭据。

示例：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Add identity types
    services.AddIdentity<ApplicationUser, ApplicationRole>()
        .AddDefaultTokenProviders();

    // Identity Services
    services.AddTransient<IUserStore<ApplicationUser>, CustomUserStore>();
    services.AddTransient<IRoleStore<ApplicationRole>, CustomRoleStore>();
    string connectionString = Configuration.GetConnectionString("DefaultConnection");
    services.AddTransient<SqlConnection>(e => new SqlConnection(connectionString));
    services.AddTransient<DapperUsersTable>();

    // additional configuration
}
```

## <a name="references"></a>参考

* [ASP.NET 4.x 的自定义存储提供程序 Identity](/aspnet/identity/overview/extensibility/overview-of-custom-storage-providers-for-aspnet-identity)
* [ASP.NET Core Identity](https://github.com/dotnet/AspNetCore/tree/master/src/Identity)：此存储库包含社区维护的存储提供程序的链接。
