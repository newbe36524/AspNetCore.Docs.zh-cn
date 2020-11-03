---
title: 第 4 部分，ASP.NET Core 中的 Razor 页面和 EF Core - 迁移
author: rick-anderson
description: Razor 页面和实体框架教程系列第 4 部分。
ms.author: riande
ms.date: 07/22/2019
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
uid: data/ef-rp/migrations
ms.openlocfilehash: 78eb466fcfeb130e411df490f033114b3fdebeef
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722626"
---
# <a name="part-4-no-locrazor-pages-with-ef-core-migrations-in-aspnet-core"></a>第 4 部分，ASP.NET Core 中的 Razor 页面和 EF Core 迁移

作者：[Tom Dykstra](https://github.com/tdykstra)、[Jon P Smith](https://twitter.com/thereformedprog) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE [about the series](~/includes/RP-EF/intro.md)]

::: moniker range=">= aspnetcore-3.0"

本教程介绍管理数据模型更改的 EF Core 迁移功能。

开发新应用时，数据模型会频繁更改。 每当模型发生更改时，都无法与数据库进行同步。 本教程从配置实体框架以创建数据库（如果不存在）开始。 数据模型每次发生更改时，必须删除该数据库。 下次应用运行时，对 `EnsureCreated` 的调用将重新创建数据库以匹配新的数据模型。 然后 `DbInitializer` 类将运行以设定新数据库的种子。

这种使 DB 与数据模型保持同步的方法适用于多种情况，但将应用部署到生产环境的情况除外。 当应用在生产环境中运行时，应用通常会存储需要保留的数据。 每当发生更改（例如添加新列）时，应用都无法在具有测试数据库的环境下启动。 EF Core 迁移功能通过启用 EF Core 更新数据库架构而不是创建新数据库来解决此问题。

数据模型更改时，迁移不会删除并重新创建数据库，而是更新架构并保留现有数据。

[!INCLUDE[](~/includes/sqlite-warn.md)]

## <a name="drop-the-database"></a>删除数据库

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

使用 SQL Server 对象资源管理器 (SSOX) 删除数据库或在包管理器控制台 (PMC) 中运行以下命令 ：

```powershell
Drop-Database
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 在命令提示符下运行以下命令以安装 EF CLI：

  ```dotnetcli
  dotnet tool install --global dotnet-ef
  ```

* 在命令提示符下，导航到项目文件夹。 项目文件夹包含 ContosoUniversity.csproj 文件。

* 删除 CU.db 文件，或运行以下命令：

  ```dotnetcli
  dotnet ef database drop --force
  ```

---

## <a name="create-an-initial-migration"></a>创建初始迁移

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 PMC 中运行以下命令：

```powershell
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

确保命令提示符位于项目文件夹中，并运行以下命令：

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

## <a name="up-and-down-methods"></a>Up 和 Down 方法

EF Core `migrations add` 命令已生成用于创建数据库的代码。 此迁移代码位于 Migrations\<timestamp>_InitialCreate.cs 文件中。 `InitialCreate` 类的 `Up` 方法创建与数据模型实体集对应的数据库表。 `Down` 方法删除这些表，如下例所示：

[!code-csharp[](intro/samples/cu30/Migrations/20190731193522_InitialCreate.cs)]

前面的代码适用于初始迁移。 代码：

* 由 `migrations add InitialCreate` 命令生成。 
* 由 `database update` 命令执行。
* 为数据库上下文类指定的数据模型创建数据库。

迁移名称参数（本示例中为“InitialCreate”）用于指定文件名。 迁移名称可以是任何有效的文件名。 最好选择能概括迁移中所执行操作的字词或短语。 例如，添加了系表的迁移可称为“AddDepartmentTable”。

## <a name="the-migrations-history-table"></a>迁移历史记录表

* 使用 SSOX 或 SQLite 工具检查数据库。
* 请注意，增加了 `__EFMigrationsHistory` 表。 `__EFMigrationsHistory` 表跟踪已应用到数据库的迁移。
* 查看 `__EFMigrationsHistory` 表中的数据。 它显示第一次迁移的行。

## <a name="the-data-model-snapshot"></a>数据模型快照

迁移会在 Migrations/SchoolContextModelSnapshot.cs 中创建当前数据模型的快照 。 添加迁移时，EF 会通过将当前数据模型与快照文件进行对比来确定已更改的内容。

由于快照文件跟踪数据模型的状态，因此不能通过删除 `<timestamp>_<migrationname>.cs` 文件来删除迁移。 要返回最近的迁移，必须使用 `migrations remove` 命令。 该命令删除迁移并确保正确重置快照。 有关详细信息，请参阅 [dotnet ef migrations remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove)。

## <a name="remove-ensurecreated"></a>删除 EnsureCreated

本系列教程从使用 `EnsureCreated` 开始。 `EnsureCreated` 不创建迁移历史记录表，因此不能与迁移一起使用。 它专门用于在频繁删除并重新创建 DB 的情况下进行测试或快速制作原型。

从这个角度来看，教程将使用迁移。

在 Data/DBInitializer.cs 中，注释掉以下行：

```csharp
context.Database.EnsureCreated();
```
运行应用并验证数据库是否已设定种子。

## <a name="applying-migrations-in-production"></a>在生产环境中应用迁移

不建议生产应用在应用程序启动时调用 [Database.Migrate](/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.migrate#Microsoft_EntityFrameworkCore_RelationalDatabaseFacadeExtensions_Migrate_Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_)。 `Migrate` 不应从部署到服务器场的应用中调用。 如果应用横向扩展到多个服务器实例，则很难确保多个服务器不会发生数据库架构更新，或者这些更新不会与读/写访问冲突。

应在部署过程中以受控的方式执行数据库迁移。 生产数据库迁移方法包括：

* 使用迁移创建 SQL 脚本，并在部署过程中使用 SQL 脚本。
* 在受控的环境中运行 `dotnet ef database update`。

## <a name="troubleshooting"></a>疑难解答

如果应用使用 SQL Server LocalDB 并显示以下异常：

```text
SqlException: Cannot open database "ContosoUniversity" requested by the login.
The login failed.
Login failed for user 'user name'.
```

解决方案可能是在命令提示符下运行 `dotnet ef database update`。

### <a name="additional-resources"></a>其他资源

* [EF Core CLI](/ef/core/miscellaneous/cli/dotnet)。
* [包管理器控制台 (Visual Studio)](/ef/core/miscellaneous/cli/powershell)

## <a name="next-steps"></a>后续步骤

下一个教程将生成数据模型，并添加实体属性和新实体。

> [!div class="step-by-step"]
> [上一个教程](xref:data/ef-rp/sort-filter-page)
> [下一个教程](xref:data/ef-rp/complex-data-model)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本教程使用 EF Core 迁移功能管理数据模型更改。

如果遇到无法解决的问题，请下载[已完成应用](
https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples)。

开发新应用时，数据模型会频繁更改。 每当模型发生更改时，都无法与数据库进行同步。 本教程首先配置 Entity Framework 以创建数据库（如果不存在）。 每当数据模型发生更改时：

* DB 都会被删除。
* EF 都会创建一个新数据库来匹配该模型。
* 应用使用测试数据为 DB 设定种子。

这种使 DB 与数据模型保持同步的方法适用于多种情况，但将应用部署到生产环境的情况除外。 当应用在生产环境中运行时，应用通常会存储需要保留的数据。 每当发生更改（例如添加新列）时，应用都无法在具有测试 DB 的环境下启动。 EF Core 迁移功能可通过使 EF Core 更新 DB 架构而不是创建新 DB 来解决此问题。

数据模型发生更改时，迁移将更新架构并保留现有数据，而无需删除或重新创建 DB。

## <a name="drop-the-database"></a>删除数据库

使用 SQL Server 对象资源管理器 (SSOX) 或 `database drop` 命令：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在“包管理器控制台”(PMC) 中运行以下命令：

```powershell
Drop-Database
```

从 PMC 运行 `Get-Help about_EntityFrameworkCore`，获取帮助信息。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

打开命令窗口并导航到项目文件夹。 项目文件夹包含 Startup.cs 文件。

在命令窗口中输入以下内容：

 ```dotnetcli
 dotnet ef database drop
 ```

---

## <a name="create-an-initial-migration-and-update-the-db"></a>创建初始迁移并更新 DB

生成项目并创建第一个迁移。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
Add-Migration InitialCreate
Update-Database
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet ef migrations add InitialCreate
dotnet ef database update
```

---

### <a name="examine-the-up-and-down-methods"></a>了解 Up 和 Down 方法

EF Core `migrations add` 命令已生成用于创建 DB 的代码。 此迁移代码位于 Migrations\<timestamp>_InitialCreate.cs 文件中。 `InitialCreate` 类的 `Up` 的方法创建与数据模型实体集相对应的 DB 表。 `Down` 方法删除这些表，如下例所示：

[!code-csharp[](intro/samples/cu21/Migrations/20180626224812_InitialCreate.cs?range=7-24,77-88)]

迁移调用 `Up` 方法为迁移实现数据模型更改。 输入用于回退更新的命令时，迁移调用 `Down` 方法。

前面的代码适用于初始迁移。 该代码是运行 `migrations add InitialCreate` 命令时创建的。 迁移名称参数（本示例中为“InitialCreate”）用于指定文件名。 迁移名称可以是任何有效的文件名。 最好选择能概括迁移中所执行操作的字词或短语。 例如，添加了系表的迁移可称为“AddDepartmentTable”。

如果创建了初始迁移并且存在 DB：

* 会生成 DB 创建代码。
* DB 创建代码不需要运行，因为 DB 已与数据模型相匹配。 即使 DB 创建代码运行也不会做出任何更改，因为 DB 已与数据模型相匹配。

如果将应用部署到新环境，则必须运行 DB 创建代码才能创建 DB。

先前删除了 DB，因此已不存在，所以迁移会创建新的 DB。

### <a name="the-data-model-snapshot"></a>数据模型快照

迁移在 Migrations/SchoolContextModelSnapshot.cs 中创建当前数据库架构的快照 。 添加迁移时，EF 会通过将数据模型与快照文件进行对比来确定已更改的内容。

若要删除迁移，请使用以下命令：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

Remove-Migration

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet ef migrations remove
```

有关详细信息，请参阅 [dotnet ef migrations remove](/ef/core/miscellaneous/cli/dotnet#dotnet-ef-migrations-remove)。

---

删除迁移命令会删除迁移并确保正确重置快照。

### <a name="remove-ensurecreated-and-test-the-app"></a>删除 EnsureCreated 并测试应用

早期开发使用了 `EnsureCreated`。 本教程将使用迁移。 `EnsureCreated` 具有以下限制：

* 绕过迁移并创建 DB 和架构。
* 不会创建迁移表。
* 不能与迁移一起使用。
* 专门用于在频繁删除并重新创建 DB 的情况下进行测试或快速制作原型。

删除 `EnsureCreated`：

```csharp
context.Database.EnsureCreated();
```

运行应用并验证 DB 设定为种子。

### <a name="inspect-the-database"></a>检查数据库

使用 SQL Server 对象资源管理器检查 DB。 请注意，增加了 `__EFMigrationsHistory` 表。 `__EFMigrationsHistory` 表跟踪已应用到 DB 的迁移。 查看 `__EFMigrationsHistory` 表中的数据，其中显示对应初始迁移的一行数据。 上面的 CLI 输出示例中最后部分的日志显示了创建此行的 INSERT 语句。

运行应用并验证一切正常运行。

## <a name="applying-migrations-in-production"></a>在生产环境中应用迁移

不建议生产应用在应用程序启动时调用 [Database.Migrate](/dotnet/api/microsoft.entityframeworkcore.relationaldatabasefacadeextensions.migrate#Microsoft_EntityFrameworkCore_RelationalDatabaseFacadeExtensions_Migrate_Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_)。 不应从服务器场中的应用调用 `Migrate`。 例如，已将应用在云中部署为横向扩展（运行应用的多个示例）的情况。

应在部署过程中以受控的方式执行数据库迁移。 生产数据库迁移方法包括：

* 使用迁移创建 SQL 脚本，并在部署过程中使用 SQL 脚本。
* 在受控的环境中运行 `dotnet ef database update`。

EF Core 使用 `__MigrationsHistory` 表查看是否需要运行任何迁移。 如果 DB 已是最新，则无需运行迁移。

## <a name="troubleshooting"></a>疑难解答

下载[已完成应用](
https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu21snapshots/cu-part4-migrations)。

应用会生成以下异常：

```text
SqlException: Cannot open database "ContosoUniversity" requested by the login.
The login failed.
Login failed for user 'user name'.
```

解决方案：运行 `dotnet ef database update`

### <a name="additional-resources"></a>其他资源

* [本教程的 YouTube 版本](https://www.youtube.com/watch?v=OWSUuMLKTJo)
* [.NET Core CLI](/ef/core/miscellaneous/cli/dotnet).
* [包管理器控制台 (Visual Studio)](/ef/core/miscellaneous/cli/powershell)



> [!div class="step-by-step"]
> [上一页](xref:data/ef-rp/sort-filter-page)
> [下一页](xref:data/ef-rp/complex-data-model)

::: moniker-end

