---
title: ASP.NET Core 中的 Razor Pages 和 Entity Framework Core - 第 1 个教程（共 8 个）
author: rick-anderson
description: 介绍如何使用 Entity Framework Core 创建 Razor Pages 应用
ms.author: riande
ms.custom: mvc, seodec18
ms.date: 9/26/2020
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
uid: data/ef-rp/intro
ms.openlocfilehash: 35a5758500ae2bc691c8d08eccb22340f9998c39
ms.sourcegitcommit: 6c82d78662332cd40d614019b9ed17c46e25be28
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/29/2020
ms.locfileid: "91424268"
---
# <a name="no-locrazor-pages-with-entity-framework-core-in-aspnet-core---tutorial-1-of-8"></a>ASP.NET Core 中的 Razor Pages 和 Entity Framework Core - 第 1 个教程（共 8 个）

作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-5.0"

本文是系列教程的第一篇，这些教程展示如何在 [ASP.NET Core Razor Pages](xref:razor-pages/index) 应用中使用 Entity Framework (EF) Core。 这些教程为虚构的 Contoso University 生成一个网站。 网站包括学生录取、课程创建和讲师分配等功能。 本教程使用代码优先方法。 有关使用数据库优先方法学习本教程的信息，请参阅[此 Github 问题](https://github.com/dotnet/AspNetCore.Docs/issues/16897)。

[下载或查看已完成的应用。](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) [下载说明](xref:index#how-to-download-a-sample)。

## <a name="prerequisites"></a>先决条件

* 如果不熟悉 Razor Pages，则在开始前，请浏览 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)系列教程。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[VS prereqs](~/includes/net-core-prereqs-vs-5.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[VS Code prereqs](~/includes/net-core-prereqs-vsc-5.0.md)]

---

## <a name="database-engines"></a>数据库引擎

Visual Studio 指令使用 [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb)，它是只在 Windows 上运行的一种 SQL Server Express 版本。

Visual Studio Code 指令使用 [SQLite](https://www.sqlite.org/)，一种跨平台数据库引擎。

如果选择使用 SQLite，请下载并安装[适用于 SQLite 的数据库浏览器](https://sqlitebrowser.org/)等第三方工具，用于管理和查看 SQLite 数据库。

## <a name="troubleshooting"></a>疑难解答

如果遇到无法解决的问题，请将你的代码与[完成的项目](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples)进行比较。 获取帮助的一个好方法是使用 [ASP.NET Core 标记](https://stackoverflow.com/questions/tagged/asp.net-core)或 [EF Core 标记](https://stackoverflow.com/questions/tagged/entity-framework-core)将问题发布到 StackOverflow.com。

## <a name="the-sample-app"></a>示例应用

这些教程中所构建的应用是一个基本的大学网站。 用户可以查看和更新学生、课程和讲师信息。 以下是在本教程中创建的几个屏幕。

![“学生索引”页](intro/_static/students-index30.png)

![学生编辑页](intro/_static/student-edit30.png)

此网站的 UI 样式基于内置的项目模板。 本教程侧重于如何将 EF Core 和 ASP.NET Core 结合在一起使用，而不是如何自定义 UI。

<!-- 
Follow the link at the top of the page to get the source code for the completed project. The *cu50* folder has the code for the ASP.NET Core 5.0 version of the tutorial. Files that reflect the state of the code for tutorials 1-7 can be found in the *cu50snapshots* folder.

# [Visual Studio](#tab/visual-studio)

To run the app after downloading the completed project:

* Build the project.
* In Package Manager Console (PMC) run the following command:

  ```powershell
  Update-Database
  ```

* Run the project to seed the database.

# [Visual Studio Code](#tab/visual-studio-code)

To run the app after downloading the completed project:

* In *Program.cs*, remove the comments from `// webBuilder.UseStartup<StartupSQLite>();`  so `StartupSQLite` is used.
* Copy the contents of *appSettingsSQLite.json* into *appSettings.json*.
* Delete the *Migrations* folder, and rename *MigrationsSQL* to *Migrations*.
* Do a global search for `#if SQLiteVersion` and remove `#if SQLiteVersion` and the associated `#endif` statement.
* Build the project.
* At a command prompt in the project folder, run the following commands:

  ```dotnetcli
  dotnet tool install --global dotnet-ef -v 5.0.0-*
  dotnet ef database update
  ```

* In your SQLite tool, run the following SQL statement:

  ```sql
  UPDATE Department SET RowVersion = randomblob(8)
  ```

* Run the project to seed the database.

---

-->

## <a name="create-the-web-app-project"></a>创建 Web 应用项目

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 从 Visual Studio“文件”菜单中选择“新建”>“项目”。
* 选择“ASP.NET Core Web 应用程序”。
* 将该项目命名为 ContosoUniversity 。 请务必使用此名称（含大写），确保在复制和粘贴代码时与命名空间相匹配。
* 在下拉列表中选择“.NET Core”和“ASP.NET Core 5.0”，然后选择“Web 应用程序”。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 在终端中，导航到应在其中创建项目文件夹的文件夹。
* 运行以下命令，在新的项目文件夹中创建 Razor Pages 项目和 `cd`：

  ```dotnetcli
  dotnet new webapp -o ContosoUniversity
  cd ContosoUniversity  
  ```

---

## <a name="set-up-the-site-style"></a>设置网站样式

将下面的代码复制并粘贴到 Pages/Shared/_Layout.cshtml 文件中：[!code-cshtml[Main](intro/samples/cu50/Pages/Shared/_Layout.cshtml?highlight=6,14,21-35,49)]

布局文件会设置网站页眉、页脚和菜单。 上面的代码执行以下更改：

* 将文件中的“ContosoUniversity”更改为“Contoso University”。 需要更改三个地方。
* 删除“主页”和“隐私”菜单项。
* 添加“关于”、“学生”、“课程”、“讲师”和“部门”项。

在 Pages/Index.cshtml 中，将该文件的内容替换为以下代码：

[!code-cshtml[Main](intro/samples/cu50/Pages/Index.cshtml)]

前面的代码会将关于 ASP.NET Core 的文本替换为关于此应用的文本。

运行应用以验证主页是否显示。

## <a name="the-data-model"></a>数据模型

以下部分用于创建数据模型：

![Course-Enrollment-Student 数据模型关系图](intro/_static/data-model-diagram.png)

一名学生可以修读任意数量的课程，并且某一课程可以有任意数量的学生修读。

## <a name="the-student-entity"></a>Student 实体

![Student 实体关系图](intro/_static/student-entity.png)

* 在项目文件夹中创建“Models”文件夹。 

* 使用以下代码创建 Models/Student.cs：

  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Student.cs)]

`ID` 属性成为此类对应的数据库表的主键列。 默认情况下，EF Core 将名为 `ID` 或 `classnameID` 的属性视为主键。 因此，`Student` 类主键的另一种自动识别的名称是 `StudentID`。 有关详细信息，请参阅 [F Core - 密钥](/ef/core/modeling/keys?tabs=data-annotations)。

`Enrollments` 属性是[导航属性](/ef/core/modeling/relationships)。 导航属性中包含与此实体相关的其他实体。 在本例中，`Student` 实体的 `Enrollments` 属性包含与该 Student 相关的所有 `Enrollment` 实体。 例如，如果数据库中的 Student 行有两个相关的 Enrollment 行，则 `Enrollments` 导航属性包含这两个 Enrollment 实体。 

在数据库中，如果 StudentID 列包含学生的 ID 值，则 Enrollment 行与 Student 行相关。 例如，假设某个 Student 行的 ID=1。 则相关 Enrollment 行的 StudentID = 1。 StudentID 是 Enrollment 表中的外键。 

`Enrollments` 属性定义为 `ICollection<Enrollment>`，因为可能有多个相关的 Enrollment 实体。 可以使用 `List<Enrollment>` 或 `HashSet<Enrollment>` 等其他集合类型。 使用 `ICollection<Enrollment>` 时，EF Core 会默认创建 `HashSet<Enrollment>` 集合。

## <a name="the-enrollment-entity"></a>Enrollment 实体

![Enrollment 实体关系图](intro/_static/enrollment-entity.png)

使用以下代码创建 Models/Enrollment.cs：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Enrollment.cs)]

`EnrollmentID` 属性为主键；此实体使用 `classnameID` 模式而不是直接使用 `ID`。 对于生产数据模型，请选择一个模式并一直使用。 本教程两个都使用，只是为了说明这两个模式都能使用。 使用不具有 `classname` 的 `ID` 可以更轻松地实现某些类型的数据模型更改。

`Grade` 属性为 `enum`。 `Grade` 声明类型后的`?`表示 `Grade` 属性可以为 [null](/dotnet/csharp/programming-guide/nullable-types/)。 评级为 null 和评级为零是有区别的 &mdash; null 意味着评级未知或者尚未分配。

`StudentID` 属性是外键，其对应的导航属性为 `Student`。 `Enrollment` 实体与一个 `Student` 实体相关联，因此该属性只包含一个 `Student` 实体。

`CourseID` 属性是外键，其对应的导航属性为 `Course`。 `Enrollment` 实体与一个 `Course` 实体相关联。

如果属性命名为 `<navigation property name><primary key property name>`，EF Core 会将其视为外键。 例如，`StudentID` 是 `Student` 导航属性的外键，因为 `Student` 实体的主键为 `ID`。 还可以将外键属性命名为 `<primary key property name>`。 例如 `CourseID`，因为 `Course` 实体的主键为 `CourseID`。

## <a name="the-course-entity"></a>Course 实体

![Course 实体关系图](intro/_static/course-entity.png)

使用以下代码创建 Models/Course.cs：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Course.cs)]

`Enrollments` 属性是导航属性。 `Course` 实体可与任意数量的 `Enrollment` 实体相关。

应用可以通过 `DatabaseGenerated` 特性指定主键，而无需靠数据库生成。

生成项目以验证没有任何编译器错误。

## <a name="scaffold-student-pages"></a>搭建“学生”页的基架

本部分使用 ASP.NET Core 基架工具生成以下内容：

* EF Core `DbContext` 类。 上下文是为给定数据模型协调实体框架功能的主类。 它派生自 <xref:Microsoft.EntityFrameworkCore.DbContext?displayProperty=fullName> 类。
* Razor 页面，可处理 `Student` 实体的创建、读取、更新和删除 (CRUD) 操作。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 创建“Pages/Students”文件夹。
* 在“解决方案资源管理器”中，右键单击“Pages/Students”文件夹，然后选择“添加”>“新搭建基架的项目” 。
* 在“添加新基架项”对话框中：
  * 在左侧选项卡中，依次选择“已安装 > 通用 > Razor 页面”
  * 依次选择“使用实体框架的 Razor 页面(CRUD)”>“添加”。
* 在“添加使用实体框架的 Razor Pages (CRUD)”对话框中：
  * 在“模型类”下拉列表中，选择“Student (ContosoUniversity.Models)” 。
  * 在“数据上下文类”行中，选择 +（加号） 。
    * 将数据上下文名称更改为以 `SchoolContext` 结尾，而不以 `ContosoUniversityContext` 结尾。 更新后的上下文名称为：`ContosoUniversity.Data.SchoolContext`
   * 选择“添加”。

自动安装以下包：

* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.EntityFrameworkCore.Tools`
* `Microsoft.VisualStudio.Web.CodeGeneration.Design`

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 运行以下 .NET Core CLI 命令，安装所需的 NuGet 包：

  ```dotnetcli
  dotnet add package Microsoft.EntityFrameworkCore.SQLite -v 5.0.0-*
  dotnet add package Microsoft.EntityFrameworkCore.SqlServer -v 5.0.0-*
  dotnet add package Microsoft.EntityFrameworkCore.Design -v 5.0.0-*
  dotnet add package Microsoft.EntityFrameworkCore.Tools -v 5.0.0-*
  dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design -v 5.0.0-*
  dotnet add package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore -v 5.0.0-*  
  ```

   基架需要 Microsoft.VisualStudio.Web.CodeGeneration.Design 包。 虽然应用不使用 SQL Server，但基架工具需要 SQL Server 包。

* 创建“Pages/Students”文件夹。

* 运行以下命令安装 [aspnet-codegenerator 基架工具](xref:fundamentals/tools/dotnet-aspnet-codegenerator)。

  ```dotnetcli
  dotnet tool uninstall --global dotnet-aspnet-codegenerator
  dotnet tool install --global dotnet-aspnet-codegenerator --version 5.0.0-*  
  ```

* 运行以下命令，搭建“学生”页的基架。

  **在 Windows 上**

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages\Students --referenceScriptLibraries -sqlite  
  ```

  **在 macOS 或 Linux 上**

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages/Students --referenceScriptLibraries -sqlite  
  ```

---

如果上述步骤失败，请生成项目并重试基架搭建步骤。

基架流程：

* 在“Pages/Students”文件夹中创建 Razor 页面：
  * Create.cshtml 和 Create.cshtml.cs 
  * Delete.cshtml 和 Delete.cshtml.cs 
  * Details.cshtml 和 Details.cshtml.cs 
  * Edit.cshtml 和 Edit.cshtml.cs 
  * Index.cshtml 和 Index.cshtml.cs 
* 创建 Data/SchoolContext.cs。
* 将上下文添加到 Startup.cs 中的依赖项注入。
* 将数据库连接字符串添加到 appsettings.json。

## <a name="database-connection-string"></a>数据库连接字符串

基架工具会在 appsettings.json 文件中生成连接字符串。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

此连接字符串指定了 [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb)：

[!code-json[Main](intro/samples/cu50/appsettings.json?highlight=11)]

LocalDB 是轻型版本 SQL Server Express 数据库引擎，专门针对应用开发，而非生产使用。 默认情况下，LocalDB 会在 `C:/Users/<user>` 目录中创建 .mdf 文件。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

将 SQLite 连接字符串缩写为 CU.db：

[!code-json[Main](intro/samples/cu50/appsettingsSQLite.json?highlight=11)]

---

## <a name="update-the-database-context-class"></a>更新数据库上下文类

数据库上下文类是为给定数据模型协调 EF Core 功能的主类。 上下文派生自 [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)。 上下文指定数据模型中包含哪些实体。 在此项目中将数据库上下文类命名为 `SchoolContext`。

使用以下代码更新 Data/SchoolContext.cs：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/SchoolContext.cs?highlight=13-22)]

上述代码会将单数形式的 `DbSet<Student> Student` 更改为复数形式的 `DbSet<Student> Students`。 若要使 Razor 页面代码与新的 `DBSet` 名称匹配，请进行以下全局更改：`_context.Student.`
更改为 `_context.Students.`

更改发生 8 次。

由于一个实体集包含多个实体，因此许多开发人员更倾向于使用复数形式的 `DBSet` 属性名称。

突出显示的代码：

* 为每个实体集创建 [DbSet\<TEntity>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) 属性。 在 EF Core 术语中：
  * 实体集通常对应数据库表。
  * 实体对应表中的行。
* 调用 <xref:Microsoft.EntityFrameworkCore.DbContext.OnModelCreating%2A>。 `OnModelCreating`:
  * 在完成对 `SchoolContext` 的初始化后，并在模型已锁定并用于初始化上下文之前，进行调用。
  * 是必需的，因为在本教程的后续部分中，`Student` 实体将引用其他实体。
  <!-- Review, OnModelCreating needs review -->

生成项目以验证没有任何编译器错误。

## <a name="startupcs"></a>Startup.cs

ASP.NET Core 通过[依赖关系注入](xref:fundamentals/dependency-injection)进行生成。 服务（例如 `SchoolContext`）在应用程序启动期间通过依赖关系注入进行注册。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取数据库上下文实例的构造函数代码。

基架工具自动将上下文类注册到了依赖项注入容器。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

基架添加了下列突出显示的行：

[!code-csharp[Main](intro/samples/cu30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

验证基架所添加的代码是否调用 `UseSqlite`。

[!code-csharp[Main](intro/samples/cu30/StartupSQLite.cs?name=snippet_ConfigureServices&highlight=5-6)]

有关使用生产数据库的信息，请参阅[将 SQLite 用于开发，将 SQL Server 用于生产](xref:tutorials/razor-pages/model#use-sqlite-for-development-sql-server-for-production)。

---

通过调用 [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时， [ASP.NET Core 配置系统](xref:fundamentals/configuration/index) 在 *appsettings.json* 文件中读取数据库连接字符串。

### <a name="add-the-database-exception-filter"></a>添加数据库异常筛选器

将 `AddDatabaseDeveloperPageExceptionFilter` 添加到 `ConfigureServices`，如下面的代码所示：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!code-csharp[Main](intro/samples/cu50/Startup.cs?name=snippet_ConfigureServices&highlight=8)]

添加 [Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore) NuGet 包。

在 PMC 中，输入以下命令来添加此 NuGet 包：

```powershell
Install-Package Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore -Version 5.0.0-rc.1.20451.17
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!code-csharp[Main](intro/samples/cu50/StartupSQLite.cs?name=snippet_ConfigureServices&highlight=8)]

---

`Microsoft.AspNetCore.Diagnostics.EntityFrameworkCore` NuGet 包提供 Entity Framework Core 错误页的 ASP.NET Core 中间件。 此中间件有助于检测和诊断 Entity Framework Core 迁移错误。

## <a name="create-the-database"></a>创建数据库

如果没有数据库，请更新 Program.cs 以创建数据库：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Program.cs?highlight=1-2,14-18,21-38)]

如果有上下文的数据库，则 [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated) 方法不执行任何操作。 如果没有数据库，则它将创建数据库和架构。 `EnsureCreated` 启用以下工作流来处理数据模型更改：

* 删除数据库。 任何现有数据丢失。
* 更改数据模型。 例如，添加 `EmailAddress` 字段。
* 运行应用。
* `EnsureCreated` 创建具有新架构的数据库。

在无需保存数据的情况下，当架构快速发展时，此工作流在早期开发过程中表现良好。 如果需要保存已输入数据库的数据，情况就有所不同了。 如果是这种情况，请使用迁移。

本系列教程的后续部分将删除 `EnsureCreated` 创建的数据库，转而使用迁移。 无法使用迁移更新 `EnsureCreated` 创建的数据库。

### <a name="test-the-app"></a>测试应用

* 运行应用。
* 依次选择“学生”链接、“新建” 。
* 测试“编辑”、“详细信息”和“删除”链接。

## <a name="seed-the-database"></a>设定数据库种子

`EnsureCreated` 方法将创建空数据库。 本节添加用测试数据填充数据库的代码。

使用以下代码创建 Data/DbInitializer.cs：
<!-- next update, keep this file in the project and surround with #if -->
  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/DbInitializer.cs)]

  该代码会检查数据库中是否存在任何学生。 如果不存在学生，它将向数据库添加测试数据。 该代码使用数组创建测试数据而不是使用 `List<T>` 集合是为了优化性能。

在 Program.cs 中，将 `EnsureCreated` 调用替换为 `DbInitializer.Initialize` 调用：

  ```csharp
  // context.Database.EnsureCreated();
  DbInitializer.Initialize(context);
  ```

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

如果应用正在运行，则停止应用，然后在包管理器控制台 (PMC) 中运行以下命令：

```powershell
Drop-Database -Confirm
```

使用 `Y` 进行响应，以删除数据库。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 如果应用正在运行，则停止应用，然后删除 CU.db 文件。

---

* 重新启动应用。
* 选择“学生”页查看已设定种子的数据。

## <a name="view-the-database"></a>查看数据库

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 从 Visual Studio 中的“视图”菜单打开 SQL Server 对象资源管理器 (SSOX) 。
* 在 SSOX 中，依次选择“(localdb)\MSSQLLocalDB”>“数据库”>“SchoolContext-{GUID}”。 数据库名称由之前提供的上下文名称以及短划线和 GUID 组成。
* 展开“表”节点。
* 右键单击 Student 表，然后单击“查看数据”，以查看创建的列和插入到表中的行 。
* 右键单击 Student 表，然后单击“查看代码”查看 `Student` 模型如何映射到 `Student` 表架构 。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

使用 SQLite 工具查看数据库架构和已设定种子的数据。 数据库文件名为 CU.db，位于项目文件夹。

---

## <a name="asynchronous-code"></a>异步代码

异步编程是 ASP.NET Core 和 EF Core 的默认模式。

Web 服务器的可用线程是有限的，而在高负载情况下的可能所有线程都被占用。 当发生这种情况的时候，服务器就无法处理新请求，直到线程被释放。 使用同步代码时，可能会出现多个线程被占用但不能执行操作的情况，因为它们正在等待 I/O 完成。 使用异步代码时，当进程正在等待 I/O 完成，服务器可以将其线程释放用于处理其他请求。 因此，使用异步代码可以更有效地利用服务器资源，并且服务器可以无延迟地处理更多流量。

异步代码会在运行时引入少量开销。 流量较低时，对性能的影响可以忽略不计，但流量较高时，潜在的性能改善非常显著。

在以下代码中，[async](/dotnet/csharp/language-reference/keywords/async) 关键字和 `Task<T>` 返回值，`await` 关键字和 `ToListAsync` 方法让代码异步执行。

```csharp
public async Task OnGetAsync()
{
    Students = await _context.Students.ToListAsync();
}
```

* `async` 关键字让编译器执行以下操作：
  * 为方法主体的各部分生成回调。
  * 创建返回的 [Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType) 对象。
* 返回类型 `Task<T>` 表示正在进行的工作。
* `await` 关键字让编译器将该方法拆分为两个部分。 第一部分是以异步方式结束已启动的操作。 第二部分是当操作完成时注入调用回调方法的地方。
* `ToListAsync` 是 `ToList` 扩展方法的异步版本。

编写使用 EF Core 的异步代码时需要注意的一些事项：

* 只有导致查询或发送数据库命令的语句才能以异步方式执行。 这包括 `ToListAsync`、`SingleOrDefaultAsync`、`FirstOrDefaultAsync` 和 `SaveChangesAsync`。 不包括只会更改 `IQueryable` 的语句，例如 `var students = context.Students.Where(s => s.LastName == "Davolio")`。
* EF Core 上下文并非线程安全：请勿尝试并行执行多个操作。
* 若要利用异步代码的性能优势，请验证在调用向数据库发送查询的 EF Core 方法时，库程序包（如用于分页）是否使用异步。

有关 .NET 中异步编程的详细信息，请参阅[异步概述](/dotnet/standard/async)和[使用 Async 和 Await 的异步编程](/dotnet/csharp/programming-guide/concepts/async/)。

<!-- Review: See https://github.com/dotnet/AspNetCore.Docs/issues/14528 -->
## <a name="performance-considerations"></a>性能注意事项

通常，网页不应加载任何数量的行。 查询应使用分页或限制方法。 例如，上述查询可以使用 `Take` 来限制返回的行数：

[!code-csharp[Main](intro/samples/cu50snapshots/Index.cshtml.cs?name=snippet)]

如果在枚举过程中出现异常数据库，则枚举视图中的大型表可能返回部分构造的 HTTP 200 响应。

<xref:Microsoft.AspNetCore.Mvc.MvcOptions.MaxModelBindingCollectionSize> 默认值为 1024。 以下代码将设置 `MaxModelBindingCollectionSize`：

[!code-csharp[Main](intro/samples/cu50/StartupMaxMBsize.cs?name=snippet_ConfigureServices)]

稍后将在教程中介绍分页。

## <a name="next-steps"></a>后续步骤

> [!div class="step-by-step"]
> [下一教程](xref:data/ef-rp/crud)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

本文是系列教程的第一篇，这些教程展示如何在 [ASP.NET Core Razor Pages](xref:razor-pages/index) 应用中使用 Entity Framework (EF) Core。 这些教程为虚构的 Contoso University 生成一个网站。 网站包括学生录取、课程创建和讲师分配等功能。 本教程使用代码优先方法。 有关使用数据库优先方法学习本教程的信息，请参阅[此 Github 问题](https://github.com/dotnet/AspNetCore.Docs/issues/16897)。

[下载或查看已完成的应用。](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) [下载说明](xref:index#how-to-download-a-sample)。

## <a name="prerequisites"></a>先决条件

* 如果不熟悉 Razor Pages，则在开始前，请浏览 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)系列教程。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[VS prereqs](~/includes/net-core-prereqs-vs-3.0.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[VS Code prereqs](~/includes/net-core-prereqs-vsc-3.0.md)]

---

## <a name="database-engines"></a>数据库引擎

Visual Studio 指令使用 [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb)，它是只在 Windows 上运行的一种 SQL Server Express 版本。

Visual Studio Code 指令使用 [SQLite](https://www.sqlite.org/)，一种跨平台数据库引擎。

如果选择使用 SQLite，请下载并安装[适用于 SQLite 的数据库浏览器](https://sqlitebrowser.org/)等第三方工具，用于管理和查看 SQLite 数据库。

## <a name="troubleshooting"></a>疑难解答

如果遇到无法解决的问题，请将你的代码与[完成的项目](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples)进行比较。 获取帮助的一个好方法是使用 [ASP.NET Core 标记](https://stackoverflow.com/questions/tagged/asp.net-core)或 [EF Core 标记](https://stackoverflow.com/questions/tagged/entity-framework-core)将问题发布到 StackOverflow.com。

## <a name="the-sample-app"></a>示例应用

这些教程中所构建的应用是一个基本的大学网站。 用户可以查看和更新学生、课程和讲师信息。 以下是在本教程中创建的几个屏幕。

![“学生索引”页](intro/_static/students-index30.png)

![学生编辑页](intro/_static/student-edit30.png)

此网站的 UI 样式基于内置的项目模板。 本教程侧重于如何使用 EF Core，而不是如何自定义 UI。

单击页面顶部的链接，获取已完成项目的源代码。 “cu30”文件夹中有本教程的 ASP.NET Core 3.0 版本的代码。 在“cu30snapshots”文件夹中可以找到反映教程 1-7 代码状态的文件。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

若要在下载完成的项目之后运行应用，请执行以下操作：

* 生成项目。
* 在包管理器控制台 (PMC) 中运行以下命令：

  ```powershell
  Update-Database
  ```

* 运行项目，设定数据库种子。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

若要在下载完成的项目之后运行应用，请执行以下操作：

* 删除 ContosoUniversity.csproj，然后将 ContosoUniversitySQLite.csproj 重命名为 ContosoUniversity.csproj******。
* 在“Program.cs”中注释掉 `#define Startup`，以便使用 `StartupSQLite`。
* 删除 appSettings.json，然后将 appSettingsSQLite.json 重命名为 appSettings.json******。
* 删除“Migrations”文件夹，然后将 MigrationsSQL 重命名为 Migrations  。
* 对 `#if SQLiteVersion` 执行全局搜索，并删除 `#if SQLiteVersion` 和相关 `#endif` 语句。
* 生成项目。
* 在项目文件夹中的命令提示符下运行以下命令：

  ```dotnetcli
  dotnet tool uninstall --global dotnet-ef
  dotnet tool install --global dotnet-ef --version 5.0.0-*
  dotnet ef database update
  ```

* 在 SQLite 工具中，运行以下 SQL 语句：

  ```sql
  UPDATE Department SET RowVersion = randomblob(8)
  ```
  
* 运行项目，设定数据库种子。

---

## <a name="create-the-web-app-project"></a>创建 Web 应用项目

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 从 Visual Studio“文件”菜单中选择“新建”>“项目”。
* 选择“ASP.NET Core Web 应用程序”。
* 将该项目命名为 ContosoUniversity 。 请务必使用此名称（含大写），确保在复制和粘贴代码时与命名空间相匹配。
* 在下拉列表中选择“.NET Core”和“ASP.NET Core 3.0”，然后选择“Web 应用程序”  。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 在终端中，导航到应在其中创建项目文件夹的文件夹。

* 运行以下命令，在新的项目文件夹中创建 Razor Pages 项目和 `cd`：

  ```dotnetcli
  dotnet new webapp -o ContosoUniversity
  cd ContosoUniversity
  ```

---

## <a name="set-up-the-site-style"></a>设置网站样式

更新 Pages/Shared/_Layout.cshtml 以设置网站的页眉、页脚和菜单：

* 将文件中的"ContosoUniversity"更改为"Contoso University"。 需要更改三个地方。

* 删除“主页”和“隐私”菜单项，然后添加“关于”、“学生”、“课程”、“讲师”和“院系”的菜单项      。

突出显示所作更改。

[!code-cshtml[Main](intro/samples/cu30/Pages/Shared/_Layout.cshtml?highlight=6,14,21-35,49)]

在 Pages/Index.cshtml 中，将文件内容替换为以下代码，以将有关 ASP.NET Core 的文本替换为有关本应用的文本：

[!code-cshtml[Main](intro/samples/cu30/Pages/Index.cshtml)]

运行应用以验证主页是否显示。

## <a name="the-data-model"></a>数据模型

以下部分用于创建数据模型：

![Course-Enrollment-Student 数据模型关系图](intro/_static/data-model-diagram.png)

一名学生可以修读任意数量的课程，并且某一课程可以有任意数量的学生修读。

## <a name="the-student-entity"></a>Student 实体

![Student 实体关系图](intro/_static/student-entity.png)

* 在项目文件夹中创建“Models”文件夹。
* 使用以下代码创建 Models/Student.cs：

  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Student.cs)]

`ID` 属性成为此类对应的数据库表的主键列。 默认情况下，EF Core 将名为 `ID` 或 `classnameID` 的属性视为主键。 因此，`Student` 类主键的另一种自动识别的名称是 `StudentID`。 有关详细信息，请参阅 [F Core - 密钥](/ef/core/modeling/keys?tabs=data-annotations)。

`Enrollments` 属性是[导航属性](/ef/core/modeling/relationships)。 导航属性中包含与此实体相关的其他实体。 在本例中，`Student` 实体的 `Enrollments` 属性包含与该 Student 相关的所有 `Enrollment` 实体。 例如，如果数据库中的 Student 行有两个相关的 Enrollment 行，则 `Enrollments` 导航属性包含这两个 Enrollment 实体。 

在数据库中，如果 StudentID 列包含学生的 ID 值，则 Enrollment 行与 Student 行相关。 例如，假设某个 Student 行的 ID=1。 则相关 Enrollment 行的 StudentID = 1。 StudentID 是 Enrollment 表中的外键。 

`Enrollments` 属性定义为 `ICollection<Enrollment>`，因为可能有多个相关的 Enrollment 实体。 可以使用 `List<Enrollment>` 或 `HashSet<Enrollment>` 等其他集合类型。 使用 `ICollection<Enrollment>` 时，EF Core 会默认创建 `HashSet<Enrollment>` 集合。

## <a name="the-enrollment-entity"></a>Enrollment 实体

![Enrollment 实体关系图](intro/_static/enrollment-entity.png)

使用以下代码创建 Models/Enrollment.cs：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Enrollment.cs)]

`EnrollmentID` 属性为主键；此实体使用 `classnameID` 模式而不是直接使用 `ID`。 对于生产数据模型，请选择一个模式并一直使用。 本教程两个都使用，只是为了说明这两个模式都能使用。 使用不具有 `classname` 的 `ID` 可以更轻松地实现某些类型的数据模型更改。

`Grade` 属性为 `enum`。 `Grade` 声明类型后的`?`表示 `Grade` 属性可以为 [null](/dotnet/csharp/programming-guide/nullable-types/)。 评级为 null 和评级为零是有区别的 &mdash; null 意味着评级未知或者尚未分配。

`StudentID` 属性是外键，其对应的导航属性为 `Student`。 `Enrollment` 实体与一个 `Student` 实体相关联，因此该属性只包含一个 `Student` 实体。

`CourseID` 属性是外键，其对应的导航属性为 `Course`。 `Enrollment` 实体与一个 `Course` 实体相关联。

如果属性命名为 `<navigation property name><primary key property name>`，EF Core 会将其视为外键。 例如，`StudentID` 是 `Student` 导航属性的外键，因为 `Student` 实体的主键为 `ID`。 还可以将外键属性命名为 `<primary key property name>`。 例如 `CourseID`，因为 `Course` 实体的主键为 `CourseID`。

## <a name="the-course-entity"></a>Course 实体

![Course 实体关系图](intro/_static/course-entity.png)

使用以下代码创建 Models/Course.cs：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Models/Course.cs)]

`Enrollments` 属性是导航属性。 `Course` 实体可与任意数量的 `Enrollment` 实体相关。

应用可以通过 `DatabaseGenerated` 特性指定主键，而无需靠数据库生成。

生成项目以验证没有任何编译器错误。

## <a name="scaffold-student-pages"></a>搭建“学生”页的基架

本部分使用 ASP.NET Core 基架工具生成以下内容：

* EF Core 上下文类。 上下文是为给定数据模型协调实体框架功能的主类。 它派生自 `Microsoft.EntityFrameworkCore.DbContext` 类。
* Razor 页面，可处理 `Student` 实体的创建、读取、更新和删除 (CRUD) 操作。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在“Pages”文件夹中创建“Students”文件夹 。
* 在“解决方案资源管理器”中，右键单击“Pages/Students”文件夹，然后选择“添加”>“新搭建基架的项目” 。
* 在“添加基架”对话框中，依次选择“使用实体框架的 Razor Pages (CRUD)”>“添加”。
* 在“添加使用实体框架的 Razor Pages (CRUD)”对话框中：
  * 在“模型类”下拉列表中，选择“Student (ContosoUniversity.Models)” 。
  * 在“数据上下文类”行中，选择 +（加号） 。
  * 将数据上下文名称从 ContosoUniversity.Models.ContosoUniversityContext 更改为 ContosoUniversity.Data.SchoolContext 。
  * 选择“添加”。

自动安装以下包：

* `Microsoft.VisualStudio.Web.CodeGeneration.Design`
* `Microsoft.EntityFrameworkCore.SqlServer`
* `Microsoft.Extensions.Logging.Debug`
* `Microsoft.EntityFrameworkCore.Tools`

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 运行以下 .NET Core CLI 命令，安装所需的 NuGet 包：
<!-- TO DO  After testing, Replace with
[!INCLUDE[](~/includes/includes/add-EF-NuGet-SQLite-CLI.md)]
remove dotnet tool install --global  below
 -->
  ```dotnetcli
  dotnet add package Microsoft.EntityFrameworkCore.SQLite
  dotnet add package Microsoft.EntityFrameworkCore.SqlServer
  dotnet add package Microsoft.EntityFrameworkCore.Design
  dotnet add package Microsoft.EntityFrameworkCore.Tools
  dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
  dotnet add package Microsoft.Extensions.Logging.Debug
  ```

  基架需要 Microsoft.VisualStudio.Web.CodeGeneration.Design 包。 虽然应用不使用 SQL Server，但基架工具需要 SQL Server 包。

* 创建“Pages/Students”文件夹。

* 运行以下命令安装 [aspnet-codegenerator 基架工具](xref:fundamentals/tools/dotnet-aspnet-codegenerator)。

  ```dotnetcli
  dotnet tool install --global dotnet-aspnet-codegenerator
  ```

* 运行以下命令，搭建“学生”页的基架。

  **在 Windows 上**

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages\Students --referenceScriptLibraries
  ```

  **在 macOS 或 Linux 上**

  ```dotnetcli
  dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Data.SchoolContext -udl -outDir Pages/Students --referenceScriptLibraries
  ```

---

如果对上述步骤有疑问，请生成项目并重试基架搭建步骤。

基架流程：

* 在“Pages/Students”文件夹中创建 Razor 页面：
  * Create.cshtml 和 Create.cshtml.cs 
  * Delete.cshtml 和 Delete.cshtml.cs 
  * Details.cshtml 和 Details.cshtml.cs 
  * Edit.cshtml 和 Edit.cshtml.cs 
  * Index.cshtml 和 Index.cshtml.cs 
* 创建 Data/SchoolContext.cs。
* 将上下文添加到 Startup.cs 中的依赖项注入。
* 将数据库连接字符串添加到 appsettings.json。

## <a name="database-connection-string"></a>数据库连接字符串

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

appsettings.json 文件指定了连接字符串 [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb)。

[!code-json[Main](intro/samples/cu30/appsettings.json?highlight=11)]

LocalDB 是轻型版本 SQL Server Express 数据库引擎，专门针对应用开发，而非生产使用。 默认情况下，LocalDB 会在 `C:/Users/<user>` 目录中创建 .mdf 文件。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

更改连接字符串以指向名为 CU.db 的 SQLite 数据库文件：

[!code-json[Main](intro/samples/cu30/appsettingsSQLite.json?highlight=11)]

---

## <a name="update-the-database-context-class"></a>更新数据库上下文类

数据库上下文类是为给定数据模型协调 EF Core 功能的主类。 上下文派生自 [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)。 上下文指定数据模型中包含哪些实体。 在此项目中将数据库上下文类命名为 `SchoolContext`。

使用以下代码更新 Data/SchoolContext.cs：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/SchoolContext.cs?highlight=13-22)]

突出显示的代码为每个实体集创建 [DbSet\<TEntity>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) 属性。 在 EF Core 术语中：

* 实体集通常对应数据库表。
* 实体对应表中的行。

由于实体集包含多个实体，因此 DBSet 属性应为复数名称。 由于基架工具创建了 `Student` DBSet，因此此步骤将其更改为复数 `Students`。 

为了使 Razor Pages 代码与新的 DBSet 名称相匹配，请在整个项目中进行全局更改，将 `_context.Student` 更改为 `_context.Students`。  更改发生 8 次。

生成项目以验证没有任何编译器错误。

## <a name="startupcs"></a>Startup.cs

ASP.NET Core 通过[依赖关系注入](xref:fundamentals/dependency-injection)进行生成。 在应用程序启动过程通过依赖注入注册相关服务（例如 EF Core 数据库上下文）。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取数据库上下文实例的构造函数代码。

基架工具自动将上下文类注册到了依赖项注入容器。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在 `ConfigureServices` 中，基架添加了突出显示的行：

  [!code-csharp[Main](intro/samples/cu30/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 在 `ConfigureServices` 中，确保基架所添加的代码调用 `UseSqlite`。

  [!code-csharp[Main](intro/samples/cu30/StartupSQLite.cs?name=snippet_ConfigureServices&highlight=5-6)]

---

通过调用 [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时， [ASP.NET Core 配置系统](xref:fundamentals/configuration/index) 在 *appsettings.json* 文件中读取数据库连接字符串。

## <a name="create-the-database"></a>创建数据库

如果没有数据库，请更新 Program.cs 以创建数据库：

[!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Program.cs?highlight=1-2,14-18,21-38)]

如果有上下文的数据库，则 [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated) 方法不执行任何操作。 如果没有数据库，则它将创建数据库和架构。 `EnsureCreated` 启用以下工作流来处理数据模型更改：

* 删除数据库。 任何现有数据丢失。
* 更改数据模型。 例如，添加 `EmailAddress` 字段。
* 运行应用。
* `EnsureCreated` 创建具有新架构的数据库。

在无需保存数据的情况下，当架构快速发展时，此工作流在早期开发过程中表现良好。 如果需要保存已输入数据库的数据，情况就有所不同了。 如果是这种情况，请使用迁移。

本系列教程的后续部分将删除 `EnsureCreated` 创建的数据库，转而使用迁移。 无法使用迁移更新 `EnsureCreated` 创建的数据库。

### <a name="test-the-app"></a>测试应用

* 运行应用。
* 依次选择“学生”链接、“新建” 。
* 测试“编辑”、“详细信息”和“删除”链接。

## <a name="seed-the-database"></a>设定数据库种子

`EnsureCreated` 方法将创建空数据库。 本节添加用测试数据填充数据库的代码。

使用以下代码创建 Data/DbInitializer.cs：
<!-- next update, keep this file in the project and surround with #if -->
  [!code-csharp[Main](intro/samples/cu30snapshots/1-intro/Data/DbInitializer.cs)]

  该代码会检查数据库中是否存在任何学生。 如果不存在学生，它将向数据库添加测试数据。 该代码使用数组创建测试数据而不是使用 `List<T>` 集合是为了优化性能。

* 在 Program.cs 中，将 `EnsureCreated` 调用替换为 `DbInitializer.Initialize` 调用：

  ```csharp
  // context.Database.EnsureCreated();
  DbInitializer.Initialize(context);
  ```

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

如果应用正在运行，则停止应用，然后在包管理器控制台 (PMC) 中运行以下命令：

```powershell
Drop-Database
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 如果应用正在运行，则停止应用，然后删除 CU.db 文件。

---

* 重新启动应用。

* 选择“学生”页查看已设定种子的数据。

## <a name="view-the-database"></a>查看数据库

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 从 Visual Studio 中的“视图”菜单打开 SQL Server 对象资源管理器 (SSOX) 。
* 在 SSOX 中，依次选择“(localdb)\MSSQLLocalDB”>“数据库”>“SchoolContext-{GUID}”。 数据库名称由之前提供的上下文名称以及短划线和 GUID 组成。
* 展开“表”节点。
* 右键单击 Student 表，然后单击“查看数据”，以查看创建的列和插入到表中的行 。
* 右键单击 Student 表，然后单击“查看代码”查看 `Student` 模型如何映射到 `Student` 表架构 。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

使用 SQLite 工具查看数据库架构和已设定种子的数据。 数据库文件名为 CU.db，位于项目文件夹。

---

## <a name="asynchronous-code"></a>异步代码

异步编程是 ASP.NET Core 和 EF Core 的默认模式。

Web 服务器的可用线程是有限的，而在高负载情况下的可能所有线程都被占用。 当发生这种情况的时候，服务器就无法处理新请求，直到线程被释放。 使用同步代码时，可能会出现多个线程被占用但不能执行任何操作的情况，因为它们正在等待 I/O 完成。 使用异步代码时，当进程正在等待 I/O 完成，服务器可以将其线程释放用于处理其他请求。 因此，使用异步代码可以更有效地利用服务器资源，并且服务器可以无延迟地处理更多流量。

异步代码会在运行时引入少量开销。 流量较低时，对性能的影响可以忽略不计，但流量较高时，潜在的性能改善非常显著。

在以下代码中，[async](/dotnet/csharp/language-reference/keywords/async) 关键字和 `Task<T>` 返回值，`await` 关键字和 `ToListAsync` 方法让代码异步执行。

```csharp
public async Task OnGetAsync()
{
    Students = await _context.Students.ToListAsync();
}
```

* `async` 关键字让编译器执行以下操作：
  * 为方法主体的各部分生成回调。
  * 创建返回的 [Task](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType) 对象。
* 返回类型 `Task<T>` 表示正在进行的工作。
* `await` 关键字让编译器将该方法拆分为两个部分。 第一部分是以异步方式结束已启动的操作。 第二部分是当操作完成时注入调用回调方法的地方。
* `ToListAsync` 是 `ToList` 扩展方法的异步版本。

编写使用 EF Core 的异步代码时需要注意的一些事项：

* 只有导致查询或发送数据库命令的语句才能以异步方式执行。 这包括 `ToListAsync`、`SingleOrDefaultAsync`、`FirstOrDefaultAsync` 和 `SaveChangesAsync`。 不包括只会更改 `IQueryable` 的语句，例如 `var students = context.Students.Where(s => s.LastName == "Davolio")`。
* EF Core 上下文并非线程安全：请勿尝试并行执行多个操作。
* 若要利用异步代码的性能优势，请验证在调用向数据库发送查询的 EF Core 方法时，库程序包（如用于分页）是否使用异步。

有关 .NET 中异步编程的详细信息，请参阅[异步概述](/dotnet/standard/async)和[使用 Async 和 Await 的异步编程](/dotnet/csharp/programming-guide/concepts/async/)。

## <a name="next-steps"></a>后续步骤

> [!div class="step-by-step"]
> [下一教程](xref:data/ef-rp/crud)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

Contoso University 示例 Web 应用演示了如何使用 Entity Framework (EF) Core 创建 ASP.NET Core Razor Pages 应用。

该示例应用是一个虚构的 Contoso University 的网站。 其中包括学生录取、课程创建和讲师分配等功能。 本页是介绍如何构建 Contoso University 示例应用系列教程中的第一部分。

[下载或查看已完成的应用。](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples) [下载说明](xref:index#how-to-download-a-sample)。

## <a name="prerequisites"></a>先决条件

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE [](~/includes/net-core-prereqs-windows.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE [](~/includes/2.1-SDK.md)]

---

熟悉 [Razor Pages](xref:razor-pages/index)。 新程序员在开始学习本系列之前，应先完成 [Razor Pages 入门](xref:tutorials/razor-pages/razor-pages-start)。

## <a name="troubleshooting"></a>疑难解答

如果遇到无法解决的问题，可以通过与 [已完成的项目](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples)对比代码来查找解决方案。 获取帮助的一个好方法是将问题发布到适用于 [ASP.NET Core](https://stackoverflow.com/questions/tagged/asp.net-core) 或 [EF Core](https://stackoverflow.com/questions/tagged/entity-framework-core) 的 [StackOverflow.com](https://stackoverflow.com/questions/tagged/asp.net-core)。

## <a name="the-contoso-university-web-app"></a>Contoso University Web 应用

这些教程中所构建的应用是一个基本的大学网站。

用户可以查看和更新学生、课程和讲师信息。 以下是在本教程中创建的几个屏幕。

![“学生索引”页](intro/_static/students-index.png)

![学生编辑页](intro/_static/student-edit.png)

此网站的 UI 样式与内置模板生成的 UI 样式类似。 教程的重点是 EF Core 和 Razor Pages，而非 UI。

## <a name="create-the-contosouniversity-no-locrazor-pages-web-app"></a>创建 ContosoUniversity Razor Pages Web 应用

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 从 Visual Studio“文件”菜单中选择“新建”>“项目”。
* 创建新的 ASP.NET Core Web 应用程序。 将该项目命名为 ContosoUniversity 。 务必将该项目命名为 ContosoUniversity，以便复制/粘贴代码时命名空间相匹配。
* 在下拉列表中选择“ASP.NET Core 2.1”，然后选择“Web 应用程序” 。

有关上述步骤的图像，请参阅[创建 Razor Web 应用](xref:tutorials/razor-pages/razor-pages-start#create-a-razor-pages-web-app)。
运行应用。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet new webapp -o ContosoUniversity
cd ContosoUniversity
dotnet run
```

---

## <a name="set-up-the-site-style"></a>设置网站样式

设置网站菜单、布局和主页时需作少量更改。 进行以下更改以更新 Pages/Shared/_Layout.cshtml：

* 将文件中的"ContosoUniversity"更改为"Contoso University"。 需要更改三个地方。

* 添加菜单项 **Students**，**Courses**，**Instructors**，和 **Department**，并删除 **Contact**菜单项。

突出显示所作更改。 （没有显示全部标记。）

[!code-cshtml[](intro/samples/cu21/Pages/Shared/_Layout.cshtml?highlight=6,29,35-38,50&name=snippet)]

在 Pages/Index.cshtml 中，将文件内容替换为以下代码，以将有关 ASP.NET 和 MVC 的文本替换为有关本应用的文本：

[!code-cshtml[](intro/samples/cu21/Pages/Index.cshtml)]

## <a name="create-the-data-model"></a>创建数据模型

创建 Contoso University 应用的实体类。 从以下三个实体开始：

![Course-Enrollment-Student 数据模型关系图](intro/_static/data-model-diagram.png)

`Student` 和 `Enrollment` 实体之间存在一对多关系。 `Course` 和 `Enrollment` 实体之间存在一对多关系。 一名学生可以报名参加任意数量的课程。 一门课程中可以包含任意数量的学生。

以下部分将为这几个实体中的每一个实体创建一个类。

### <a name="the-student-entity"></a>Student 实体

![Student 实体关系图](intro/_static/student-entity.png)

创建 Models 文件夹。 在 Models 文件夹中，使用以下代码创建一个名为 Student.cs 的类文件 ：

[!code-csharp[](intro/samples/cu21/Models/Student.cs?name=snippet_Intro)]

`ID` 属性成为此类对应的数据库 (DB) 表的主键列。 默认情况下，EF Core 将名为 `ID` 或 `classnameID` 的属性视为主键。 在 `classnameID` 中，`classname` 为类名称。 另一种自动识别的主键是上例中的 `StudentID`。

`Enrollments` 属性是[导航属性](/ef/core/modeling/relationships)。 导航属性链接到与此实体相关的其他实体。 在这种情况下，`Student entity` 的 `Enrollments` 属性包含与该 `Student` 相关的所有 `Enrollment` 实体。 例如，如果数据库中的 Student 行有两个相关的 Enrollment 行，则 `Enrollments` 导航属性包含这两个 `Enrollment` 实体。 相关的 `Enrollment` 行是 `StudentID` 列中包含该学生的主键值的行。 例如，假设 ID=1 的学生在 `Enrollment` 表中有两行。 `Enrollment` 表中有两行的 `StudentID` = 1。 `StudentID` 是 `Enrollment` 表中的外键，用于指定 `Student` 表中的学生。

如果导航属性包含多个实体，则导航属性必须是列表类型，例如 `ICollection<T>`。 可以指定 `ICollection<T>` 或诸如 `List<T>` 或 `HashSet<T>` 的类型。 使用 `ICollection<T>` 时，EF Core 会默认创建 `HashSet<T>` 集合。 包含多个实体的导航属性来自于多对多和一对多关系。

### <a name="the-enrollment-entity"></a>Enrollment 实体

![Enrollment 实体关系图](intro/_static/enrollment-entity.png)

在 Models 文件夹中，使用以下代码创建 Enrollment.cs ：

[!code-csharp[](intro/samples/cu21/Models/Enrollment.cs?name=snippet_Intro)]

`EnrollmentID` 属性为主键。 `Student` 实体使用的是 `ID` 模式，而本实体使用的是 `classnameID` 模式。 通常情况下，开发者会选择一种模式并在整个数据模型中都使用该模式。 下一个教程将介绍如何使用不带类名的 ID，以便更轻松地在数据模型中实现集成。

`Grade` 属性为 `enum`。 `Grade` 声明类型后的`?`表示 `Grade` 属性可以为 null。 评级为 null 和评级为零是有区别的 --null 意味着评级未知或者尚未分配。

`StudentID` 属性是外键，其对应的导航属性为 `Student`。 `Enrollment` 实体与一个 `Student` 实体相关联，因此该属性只包含一个 `Student` 实体。 `Student` 实体与 `Student.Enrollments` 导航属性不同，后者包含多个 `Enrollment` 实体。

`CourseID` 属性是外键，其对应的导航属性为 `Course`。 `Enrollment` 实体与一个 `Course` 实体相关联。

如果属性命名为 `<navigation property name><primary key property name>`，EF Core 会将其视为外键。 例如 `Student` 导航属性的 `StudentID`，因为 `Student` 实体的主键为 `ID`。 还可以将外键属性命名为 `<primary key property name>`。 例如 `CourseID`，因为 `Course` 实体的主键为 `CourseID`。

### <a name="the-course-entity"></a>Course 实体

![Course 实体关系图](intro/_static/course-entity.png)

在 Models 文件夹中，使用以下代码创建 Course.cs ：

[!code-csharp[](intro/samples/cu21/Models/Course.cs?name=snippet_Intro)]

`Enrollments` 属性是导航属性。 `Course` 实体可与任意数量的 `Enrollment` 实体相关。

应用可以通过 `DatabaseGenerated` 特性指定主键，而无需靠数据库生成。

## <a name="scaffold-the-student-model"></a>为“学生”模型搭建基架

本部分将为“学生”模型搭建基架。 确切地说，基架工具将生成页面，用于对“学生”模型执行创建、读取、更新和删除 (CRUD) 操作。

* 生成项目。
* 创建 Pages/Students 文件夹。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在“解决方案资源管理器”中，右键单击“Pages/Students”文件夹 >“添加”>“新搭建基架的项目”。
* 在“添加基架”对话框中，依次选择“使用实体框架的 Razor Pages (CRUD)”>“添加”。

完成“添加使用实体框架的 Razor 页面 (CRUD)”对话框：

* 在“模型类”下拉列表中，选择“Student (ContosoUniversity.Models)” 。
* 在“数据上下文类”行中，选择加号 (+) 并将生成的名称更改为 ContosoUniversity.Models.SchoolContext  。
* 在“数据上下文类”下拉列表中，选择“ContosoUniversity.Models.SchoolContext” 
* 选择“添加”。

![CRUD 对话框](intro/_static/s1.png)

如果对前面的步骤有疑问，请参阅[搭建“电影”模型的基架](xref:tutorials/razor-pages/model#scaffold-the-movie-model)。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

运行以下命令，搭建“学生”模型的基架。

```dotnetcli
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design --version 2.1.0
dotnet tool install --global dotnet-aspnet-codegenerator
dotnet aspnet-codegenerator razorpage -m Student -dc ContosoUniversity.Models.SchoolContext -udl -outDir Pages/Students --referenceScriptLibraries
```

---

搭建基架的过程会创建并更改以下文件：

### <a name="files-created"></a>创建的文件

* Pages/Students：“创建”、“删除”、“详细信息”、“编辑”、“索引”。
* Data/SchoolContext.cs

### <a name="file-updates"></a>更新的文件

* *Startup.cs*：下一部分详细介绍对此文件所作的更改。
* *appsettings.json*：添加用于连接到本地数据库的连接字符串。

## <a name="examine-the-context-registered-with-dependency-injection"></a>检查通过依赖关系注入注册的上下文

ASP.NET Core 通过[依赖关系注入](xref:fundamentals/dependency-injection)进行生成。 服务（例如 EF Core 数据库上下文）在应用程序启动期间通过依赖关系注入进行注册。 需要这些服务（如 Razor 页面）的组件通过构造函数参数提供相应服务。 本教程的后续部分介绍了用于获取数据库上下文实例的构造函数代码。

基架工具自动创建 DB 上下文并将其注册到依赖关系注入容器。

在 Startup.cs 中检查 `ConfigureServices` 方法。 基架添加了突出显示的行：

[!code-csharp[](intro/samples/cu21/Startup.cs?name=snippet_SchoolContext&highlight=13-14)]

通过调用 [DbContextOptions](/dotnet/api/microsoft.entityframeworkcore.dbcontextoptions) 对象中的一个方法将连接字符串名称传递到上下文。 进行本地开发时， [ASP.NET Core 配置系统](xref:fundamentals/configuration/index) 在 *appsettings.json* 文件中读取数据库连接字符串。

## <a name="update-main"></a>更新 main

在 Program.cs 中，修改 `Main` 方法以执行以下操作：

* 从依赖关系注入容器获取数据库上下文实例。
* 调用 [EnsureCreated](/dotnet/api/microsoft.entityframeworkcore.infrastructure.databasefacade.ensurecreated#Microsoft_EntityFrameworkCore_Infrastructure_DatabaseFacade_EnsureCreated)。
* `EnsureCreated` 方法完成时释放上下文。

下面的代码显示更新后的 Program.cs 文件。

[!code-csharp[](intro/samples/cu21/Program.cs?name=snippet)]

`EnsureCreated` 确保存在上下文数据库。 如果存在，则不需要任何操作。 如果不存在，则会创建数据库及其所有架构。 `EnsureCreated` 不使用迁移创建数据库。 使用 `EnsureCreated` 创建的数据库稍后无法使用迁移更新。

启动应用时会调用 `EnsureCreated`，以进行以下工作流：

* 删除数据库。
* 更改数据库架构（例如添加一个 `EmailAddress` 字段）。
* 运行应用。
* `EnsureCreated` 创建一个带有 `EmailAddress` 列的数据库。

架构快速演变时，在开发初期使用 `EnsureCreated` 很方便。 本教程后面将删除 DB 并使用迁移。

### <a name="test-the-app"></a>测试应用

运行应用并接受 cookie 策略。 此应用不保留个人信息。 有关 cookie 策略的信息，请参阅[欧盟一般数据保护条例 (GDPR) 支持](xref:security/gdpr)。

* 依次选择“学生”链接、“新建” 。
* 测试“编辑”、“详细信息”和“删除”链接。

## <a name="examine-the-schoolcontext-db-context"></a>检查 SchoolContext DB 上下文

数据库上下文类是为给定数据模型协调 EF Core 功能的主类。 数据上下文派生自 [Microsoft.EntityFrameworkCore.DbContext](/dotnet/api/microsoft.entityframeworkcore.dbcontext)。 数据上下文指定数据模型中包含哪些实体。 在此项目中将数据库上下文类命名为 `SchoolContext`。

使用以下代码更新 SchoolContext.cs：

[!code-csharp[](intro/samples/cu21/Data/SchoolContext.cs?name=snippet_Intro&highlight=12-14)]

突出显示的代码为每个实体集创建 [DbSet\<TEntity>](/dotnet/api/microsoft.entityframeworkcore.dbset-1) 属性。 在 EF Core 术语中：

* 实体集通常对应一个数据库表。
* 实体对应表中的行。

可以省略 `DbSet<Enrollment>` 和 `DbSet<Course>`。 EF Core 隐式包含了它们，因为 `Student` 实体引用 `Enrollment` 实体，而 `Enrollment` 实体引用 `Course` 实体。 在本教程中，将 `DbSet<Enrollment>` 和 `DbSet<Course>` 保留在 `SchoolContext` 中。

### <a name="sql-server-express-localdb"></a>SQL Server Express LocalDB

连接字符串指定 [SQL Server LocalDB](/sql/database-engine/configure-windows/sql-server-2016-express-localdb)。 LocalDB 是轻型版本 SQL Server Express 数据库引擎，专门针对应用开发，而非生产使用。 LocalDB 作为按需启动并在用户模式下运行的轻量级数据库没有复杂的配置。 默认情况下，LocalDB 会在 `C:/Users/<user>` 目录中创建 .mdf 数据库文件。

## <a name="add-code-to-initialize-the-db-with-test-data"></a>添加代码，以使用测试数据初始化该数据库

EF Core 会创建一个空的数据库。 本部分中编写了 `Initialize` 方法来使用测试数据填充该数据库。

在 Data 文件夹中，新建一个名为 DbInitializer.cs 的类文件，并添加以下代码 ：

[!code-csharp[](intro/samples/cu21/Data/DbInitializer.cs?name=snippet_Intro)]

注意：上面的代码对命名空间使用 `Models` (`namespace ContosoUniversity.Models`)，而不是 `Data`。 `Models` 与基架生成的代码一致。 有关详细信息，请参阅[此 GitHub 基架问题](https://github.com/aspnet/Scaffolding/issues/822)。

该代码会检查数据库中是否存在任何学生。 如果 DB 中没有任何学生，则会使用测试数据初始化该 DB。 代码中使用数组存放测试数据而不是使用 `List<T>` 集合是为了优化性能。

`EnsureCreated` 方法自动为数据库上下文创建数据库。 如果数据库已存在，则返回 `EnsureCreated`，并且不修改数据库。

在 Program.cs 中，将 `Main` 方法修改为调用 `Initialize`：

[!code-csharp[](intro/samples/cu21/Program.cs?name=snippet2&highlight=14-15)]

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

如果应用正在运行，则停止应用，然后在包管理器控制台 (PMC) 中运行以下命令：

```powershell
Drop-Database
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 如果应用正在运行，则停止应用，然后删除 CU.db 文件。

---

## <a name="view-the-db"></a>查看数据库

数据库名称由之前提供的上下文名称以及短划线和 GUID 组成。 因此，数据库名称为“SchoolContext-{GUID}”。 GUID 因用户而异。
从 Visual Studio 中的“视图”菜单打开 SQL Server 对象资源管理器 (SSOX) 。
在 SSOX 中，依次单击“(localdb)\MSSQLLocalDB”>“数据库”>“SchoolContext-{GUID}”。

展开“表”节点。

右键单击 Student 表，然后单击“查看数据”，以查看创建的列和插入到表中的行 。

## <a name="asynchronous-code"></a>异步代码

异步编程是 ASP.NET Core 和 EF Core 的默认模式。

Web 服务器的可用线程是有限的，而在高负载情况下的可能所有线程都被占用。 当发生这种情况的时候，服务器就无法处理新请求，直到线程被释放。 使用同步代码时，可能会出现多个线程被占用但不能执行任何操作的情况，因为它们正在等待 I/O 完成。 使用异步代码时，当进程正在等待 I/O 完成，服务器可以将其线程释放用于处理其他请求。 因此，使用异步代码可以更有效地利用服务器资源，并且可以让服务器在没有延迟的情况下处理更多流量。

异步代码会在运行时引入少量开销。 流量较低时，对性能的影响可以忽略不计，但流量较高时，潜在的性能改善非常显著。

在以下代码中，[async](/dotnet/csharp/language-reference/keywords/async) 关键字和 `Task<T>` 返回值，`await` 关键字和 `ToListAsync` 方法让代码异步执行。

[!code-csharp[](intro/samples/cu21/Pages/Students/Index.cshtml.cs?name=snippet_ScaffoldedIndex)]

* `async` 关键字让编译器执行以下操作：
  * 为方法主体的各部分生成回调。
  * 自动创建返回的 [Task](/dotnet/api/system.threading.tasks.task) 对象。 有关详细信息，请参阅[任务返回类型](/dotnet/csharp/programming-guide/concepts/async/async-return-types#BKMK_TaskReturnType)。

* 隐式返回类型 `Task` 表示正在进行的工作。
* `await` 关键字让编译器将该方法拆分为两个部分。 第一部分是以异步方式结束已启动的操作。 第二部分是当操作完成时注入调用回调方法的地方。
* `ToListAsync` 是 `ToList` 扩展方法的异步版本。

编写使用 EF Core 的异步代码时需要注意的一些事项：

* 只会异步执行导致查询或命令被发送到数据库的语句。 这包括 `ToListAsync`、`SingleOrDefaultAsync`、`FirstOrDefaultAsync` 和 `SaveChangesAsync`。 不包括只会更改 `IQueryable` 的语句，例如 `var students = context.Students.Where(s => s.LastName == "Davolio")`。
* EF Core 上下文并非线程安全：请勿尝试并行执行多个操作。
* 若要利用异步代码的性能优势，请验证在调用向数据库发送查询的 EF Core 方法时，库程序包（如用于分页）是否使用异步。

有关 .NET 中异步编程的详细信息，请参阅[异步概述](/dotnet/standard/async)和[使用 Async 和 Await 的异步编程](/dotnet/csharp/programming-guide/concepts/async/)。

下一个教程将介绍基本的 CRUD（创建、读取、更新、删除）操作。



## <a name="additional-resources"></a>其他资源

* [本教程的 YouTube 版本](https://www.youtube.com/watch?v=P7iTtQnkrNs)

> [!div class="step-by-step"]
> [下一页](xref:data/ef-rp/crud)

::: moniker-end
