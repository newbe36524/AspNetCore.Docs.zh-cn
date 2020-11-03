---
title: 从 ASP.NET 成员身份验证迁移到 ASP.NET Core 2。0 Identity
author: isaac2004
description: 了解如何使用成员身份验证将现有的 ASP.NET 应用迁移到 ASP.NET Core 2.0 Identity 。
ms.author: scaddie
ms.custom: mvc
ms.date: 01/10/2019
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
uid: migration/proper-to-2x/membership-to-core-identity
ms.openlocfilehash: d981c424fd2d6cad95b9164420f093672325c347
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051351"
---
# <a name="migrate-from-aspnet-membership-authentication-to-aspnet-core-20-no-locidentity"></a>从 ASP.NET 成员身份验证迁移到 ASP.NET Core 2。0 Identity

作者：[Isaac Levin](https://isaaclevin.com)

本文演示了如何使用成员身份验证将 ASP.NET 应用的数据库架构迁移到 ASP.NET Core 2.0 Identity 。

> [!NOTE]
> 本文档提供将基于 ASP.NET 成员身份的应用程序的数据库架构迁移到用于的数据库架构所需的步骤 ASP.NET Core Identity 。 有关从基于 ASP.NET 的基于成员身份的身份验证迁移到 ASP.NET 的详细信息 Identity ，请参阅[将现有应用从 Identity SQL 成员身份迁移到 ASP.NET ](/aspnet/identity/overview/migrations/migrating-an-existing-website-from-sql-membership-to-aspnet-identity)。 有关的详细信息 ASP.NET Core Identity ，请参阅 [ Identity ASP.NET Core 上的简介](xref:security/authentication/identity)。

## <a name="review-of-membership-schema"></a>成员资格架构评审

在 ASP.NET 2.0 之前，开发人员负责为其应用创建整个身份验证和授权过程。 使用 ASP.NET 2.0，引入了成员身份，提供了一个样本解决方案来处理 ASP.NET 应用中的安全性。 开发人员现在可以使用命令将架构启动到 SQL Server 数据库中 <https://docs.microsoft.com/previous-versions/ms229862(v=vs.140)> 。 运行此命令后，将在数据库中创建以下表。

  ![成员资格表](identity/_static/membership-tables.png)

若要将现有应用迁移到 ASP.NET Core 2.0 Identity ，需要将这些表中的数据迁移到新架构使用的表中 Identity 。

## <a name="no-locaspnet-core-identity-20-schema"></a>ASP.NET Core Identity 2.0 架构

ASP.NET Core 2.0 遵循 [Identity](/aspnet/identity/index) ASP.NET 4.5 中引入的原则。 尽管原则是共享的，但在不同版本的 ASP.NET Core 中，框架之间的实现是不同的 (请参阅 [迁移身份验证和 Identity ASP.NET Core 2.0](xref:migration/1x-to-2x/index)) 。

查看 ASP.NET Core 2.0 的架构的最快方法 Identity 是创建新的 ASP.NET Core 2.0 应用。 在 Visual Studio 2017 中执行以下步骤：

1. 选择“文件” > “新建” > “项目”  。
1. 创建名为 *Core Identity Sample* 的新 **ASP.NET Core Web 应用程序** 项目。
1. 在下拉列表中选择 **ASP.NET Core 2.0** ，然后选择 " **Web 应用程序** "。 此模板生成[ Razor 页面](xref:razor-pages/index)应用。 单击 **"确定"** 之前，请单击 " **更改身份验证** "。
1. 为模板选择 **单个用户帐户** Identity 。 最后，单击 **"确定** "，然后单击 **"确定"** 。 Visual Studio 将使用模板创建一个项目 ASP.NET Core Identity 。
1. 选择 " **工具** " "  >  **NuGet 包管理器** " "  >  **包管理器控制台** " 打开 **包管理器控制台** (PMC) "窗口。
1. 导航到 PMC 中的项目根，并运行 [实体框架 (EF) Core](/ef/core) `Update-Database` 命令。

    ASP.NET Core 2.0 Identity 使用 EF Core 与存储身份验证数据的数据库进行交互。 为了使新创建的应用程序正常工作，需要有数据库来存储此数据。 创建新应用后，在数据库环境中检查架构的最快方法是使用 [EF Core 迁移](/ef/core/managing-schemas/migrations/)来创建数据库。 此过程将创建一个在本地或其他位置模拟该架构的数据库。 有关详细信息，请查看前面的文档。

    EF Core 命令使用中指定的数据库的连接字符串 *appsettings.json* 。 以下连接字符串针对名为 *.asp 的* *localhost* 上的数据库。 在此设置中，EF Core 配置为使用 `DefaultConnection` 连接字符串。

    ```json
    {
      "ConnectionStrings": {
        "DefaultConnection": "Server=localhost;Database=aspnet-core-identity;Trusted_Connection=True;MultipleActiveResultSets=true"
      }
    }
    ```

1. 选择 " **查看**  >  **SQL Server 对象资源管理器** "。 展开与的属性中指定的数据库名称相对应的节点 `ConnectionStrings:DefaultConnection` *appsettings.json* 。

    该 `Update-Database` 命令创建了用架构指定的数据库以及应用初始化所需的任何数据。 下图描绘了在前面的步骤中创建的表结构。

    ![：： no (Identity) ：：： Tables](identity/_static/identity-tables.png)

## <a name="migrate-the-schema"></a>迁移架构

对于成员身份和的表结构和字段存在细微的差异 ASP.NET Core Identity 。 此模式已因 ASP.NET 和 ASP.NET Core 应用的身份验证/授权而发生了重大更改。 仍用于的关键对象 Identity 是 *用户* 和 *角色* 。 下面是 *用户* 、 *角色* 和 *UserRoles* 的映射表。

### <a name="users"></a>用户

|Identity<br> (`dbo.AspNetUsers`) 列  |类型     |Membership<br> (`dbo.aspnet_Users`  /  `dbo.aspnet_Membership`) 列|类型      |
|-------------------------------------------|-----------------------------------------------------------------------|
| `Id`                            | `string`| `aspnet_Users.UserId`                                      | `string` |
| `UserName`                      | `string`| `aspnet_Users.UserName`                                    | `string` |
| `Email`                         | `string`| `aspnet_Membership.Email`                                  | `string` |
| `NormalizedUserName`            | `string`| `aspnet_Users.LoweredUserName`                             | `string` |
| `NormalizedEmail`               | `string`| `aspnet_Membership.LoweredEmail`                           | `string` |
| `PhoneNumber`                   | `string`| `aspnet_Users.MobileAlias`                                 | `string` |
| `LockoutEnabled`                | `bit`   | `aspnet_Membership.IsLockedOut`                            | `bit`    |

> [!NOTE]
> 并非所有字段映射都类似于从成员身份到的一对一关系 ASP.NET Core Identity 。 上表采用默认的成员资格用户架构，并将其映射到 ASP.NET Core Identity 架构。 需要手动映射用于成员身份的任何其他自定义字段。 在此映射中，无密码映射，因为密码条件和密码 salts 不会在两者之间迁移。 **建议将密码保留为 null 并要求用户重置其密码。** 在中 ASP.NET Core Identity ， `LockoutEnd` 如果用户被锁定，则应设置为将来的某个日期。这会显示在迁移脚本中。

### <a name="roles"></a>角色

|Identity<br> (`dbo.AspNetRoles`) 列|类型|Membership<br> (`dbo.aspnet_Roles`) 列|类型|
|----------------------------------------|-----------------------------------|
|`Id`                           |`string`|`RoleId`         | `string`        |
|`Name`                         |`string`|`RoleName`       | `string`        |
|`NormalizedName`               |`string`|`LoweredRoleName`| `string`        |

### <a name="user-roles"></a>用户角色

|Identity<br> (`dbo.AspNetUserRoles`) 列|类型|Membership<br> (`dbo.aspnet_UsersInRoles`) 列|类型|
|-------------------------|----------|--------------|---------------------------|
|`RoleId`                 |`string`  |`RoleId`      |`string`                   |
|`UserId`                 |`string`  |`UserId`      |`string`                   |

创建 *用户* 和 *角色* 的迁移脚本时引用前面的映射表。 下面的示例假定数据库服务器上有两个数据库。 一个数据库包含现有的 ASP.NET 成员身份架构和数据。 其他 *核心 Identity 示例* 数据库是使用前面所述的步骤创建的。 有关详细信息，请以内联方式包含注释。

```sql
-- THIS SCRIPT NEEDS TO RUN FROM THE CONTEXT OF THE MEMBERSHIP DB
BEGIN TRANSACTION MigrateUsersAndRoles
USE aspnetdb

-- INSERT USERS
INSERT INTO CoreIdentitySample.dbo.AspNetUsers
            (Id,
             UserName,
             NormalizedUserName,
             PasswordHash,
             SecurityStamp,
             EmailConfirmed,
             PhoneNumber,
             PhoneNumberConfirmed,
             TwoFactorEnabled,
             LockoutEnd,
             LockoutEnabled,
             AccessFailedCount,
             Email,
             NormalizedEmail)
SELECT aspnet_Users.UserId,
       aspnet_Users.UserName,
       -- The NormalizedUserName value is upper case in ASP.NET Core Identity
       UPPER(aspnet_Users.UserName),
       -- Creates an empty password since passwords don't map between the 2 schemas
       '',
       /*
        The SecurityStamp token is used to verify the state of an account and
        is subject to change at any time. It should be initialized as a new ID.
       */
       NewID(),
       /*
        EmailConfirmed is set when a new user is created and confirmed via email.
        Users must have this set during migration to reset passwords.
       */
       1,
       aspnet_Users.MobileAlias,
       CASE
         WHEN aspnet_Users.MobileAlias IS NULL THEN 0
         ELSE 1
       END,
       -- 2FA likely wasn't setup in Membership for users, so setting as false.
       0,
       CASE
         -- Setting lockout date to time in the future (1,000 years)
         WHEN aspnet_Membership.IsLockedOut = 1 THEN Dateadd(year, 1000,
                                                     Sysutcdatetime())
         ELSE NULL
       END,
       aspnet_Membership.IsLockedOut,
       /*
        AccessFailedAccount is used to track failed logins. This is stored in
        Membership in multiple columns. Setting to 0 arbitrarily.
       */
       0,
       aspnet_Membership.Email,
       -- The NormalizedEmail value is upper case in ASP.NET Core Identity
       UPPER(aspnet_Membership.Email)
FROM   aspnet_Users
       LEFT OUTER JOIN aspnet_Membership
                    ON aspnet_Membership.ApplicationId =
                       aspnet_Users.ApplicationId
                       AND aspnet_Users.UserId = aspnet_Membership.UserId
       LEFT OUTER JOIN CoreIdentitySample.dbo.AspNetUsers
                    ON aspnet_Membership.UserId = AspNetUsers.Id
WHERE  AspNetUsers.Id IS NULL

-- INSERT ROLES
INSERT INTO CoreIdentitySample.dbo.AspNetRoles(Id, Name)
SELECT RoleId, RoleName
FROM aspnet_Roles;

-- INSERT USER ROLES
INSERT INTO CoreIdentitySample.dbo.AspNetUserRoles(UserId, RoleId)
SELECT UserId, RoleId
FROM aspnet_UsersInRoles;

IF @@ERROR <> 0
  BEGIN
    ROLLBACK TRANSACTION MigrateUsersAndRoles
    RETURN
  END

COMMIT TRANSACTION MigrateUsersAndRoles
```

完成上述脚本后，将 ASP.NET Core Identity 向成员资格用户填充先前创建的应用。 用户需要在登录之前更改密码。

> [!NOTE]
> 如果成员资格系统中的用户的用户名与其电子邮件地址不匹配，则需要对之前创建的应用进行更改，以满足此要求。 默认模板需要 `UserName` 和 `Email` 相同。 对于不同的情况，需要将登录过程修改为使用 `UserName` 而不是 `Email` 。

在登录页的 "Pages\Account\Login.cshtml.cs" 中， `PageModel` 从 " *Pages\Account\Login.cshtml.cs* `[EmailAddress]` *电子邮件* " 属性中删除该属性。 将其重命名为 *UserName* 。 这需要 `EmailAddress` 在 *视图* 和 *PageModel* 中提到的任何位置进行更改。 结果应类似如下所示：

 ![固定登录](identity/_static/fixed-login.png)

## <a name="next-steps"></a>后续步骤

在本教程中，已了解如何将用户从 SQL 成员身份移植到 ASP.NET Core 2.0 Identity 。 有关的详细信息 ASP.NET Core Identity ，请参阅[介绍 Identity ](xref:security/authentication/identity)。
