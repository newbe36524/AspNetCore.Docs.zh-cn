---
title: 第 5 部分，ASP.NET Core 中的 Razor 页面和 EF Core - 数据模型
author: rick-anderson
description: Razor 页面和实体框架教程系列的第 5 部分。
ms.author: riande
ms.custom: mvc
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
uid: data/ef-rp/complex-data-model
ms.openlocfilehash: 55269c6985534b49cc2567b2d197e46d9b7b1fd7
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722522"
---
# <a name="part-5-no-locrazor-pages-with-ef-core-in-aspnet-core---data-model"></a>第 5 部分，ASP.NET Core 中的 Razor 页面和 EF Core - 数据模型

作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE [about the series](~/includes/RP-EF/intro.md)]

::: moniker range=">= aspnetcore-3.0"

前面的教程介绍了由三个实体组成的基本数据模型。 本教程将演示如何：

* 添加更多实体和关系。
* 通过指定格式设置、验证和数据库映射规则来自定义数据模型。

完成的数据模型如下图所示：

![实体关系图](complex-data-model/_static/diagram.png)

## <a name="the-student-entity"></a>Student 实体

![Student 实体](complex-data-model/_static/student-entity.png)

使用以下代码替换 Models/Student.cs 中的代码：

[!code-csharp[](intro/samples/cu30/Models/Student.cs)]

前面的代码将添加一个 `FullName` 属性，并将以下特性添加到现有属性：

* `[DataType]`
* `[DisplayFormat]`
* `[StringLength]`
* `[Column]`
* `[Required]`
* `[Display]`

### <a name="the-fullname-calculated-property"></a>FullName 计算属性

`FullName` 是计算属性，可返回通过串联两个其他属性创建的值。 无法设置 `FullName`，因此它只有一个 get 访问器。 数据库中不会创建任何 `FullName` 列。

### <a name="the-datatype-attribute"></a>DataType 特性

```csharp
[DataType(DataType.Date)]
```

对于学生注册日期，目前所有页面都显示时间和日期，但只有日期是相关的。 使用数据注释特性，可更改一次代码，修复每个页面中数据的显示格式。 

[DataType](/dotnet/api/system.componentmodel.dataannotations.datatypeattribute) 特性指定比数据库内部类型更具体的数据类型。 在此情况下，应仅显示日期，而不是日期加时间。 [DataType 枚举](/dotnet/api/system.componentmodel.dataannotations.datatype)提供多种数据类型，例如日期、时间、电话号码、货币、电子邮件地址等。应用还可通过 `DataType` 特性自动提供类型特定的功能。 例如：

* `mailto:` 链接将依据 `DataType.EmailAddress` 自动创建。
* 大多数浏览器中都提供面向 `DataType.Date` 的日期选择器。

`DataType` 特性发出 HTML 5 `data-`（读作 data dash）特性。 `DataType` 特性不提供验证。

### <a name="the-displayformat-attribute"></a>DisplayFormat 特性

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

`DataType.Date` 不指定显示日期的格式。 默认情况下，日期字段根据基于服务器的 [CultureInfo](xref:fundamentals/localization#provide-localized-resources-for-the-languages-and-cultures-you-support) 的默认格式进行显示。

`DisplayFormat` 特性用于显式指定日期格式。 `ApplyFormatInEditMode` 设置指定还应对编辑 UI 应用该格式设置。 某些字段不应使用 `ApplyFormatInEditMode`。 例如，编辑文本框中通常不应显示货币符号。

`DisplayFormat` 特性可由其本身使用。 搭配使用 `DataType` 特性和 `DisplayFormat` 特性通常是很好的做法。 `DataType` 特性按照数据在屏幕上的呈现方式传达数据的语义。 `DataType` 特性可提供 `DisplayFormat` 中所不具有的以下优点：

* 浏览器可启用 HTML5 功能。 例如，显示日历控件、区域设置适用的货币符号、电子邮件链接和客户端输入验证。
* 默认情况下，浏览器将根据区域设置采用正确的格式呈现数据。

有关详细信息，请参阅 [\<input> 标记帮助程序文档](xref:mvc/views/working-with-forms#the-input-tag-helper)。

### <a name="the-stringlength-attribute"></a>StringLength 特性

```csharp
[StringLength(50, ErrorMessage = "First name cannot be longer than 50 characters.")]
```

可使用特性指定数据验证规则和验证错误消息。 [StringLength](/dotnet/api/system.componentmodel.dataannotations.stringlengthattribute) 特性指定数据字段中允许的字符的最小长度和最大长度。 显示的代码将名称限制为不超过 50 个字符。 [稍后](#the-required-attribute)显示一个设置最小字符串长度的示例。

`StringLength` 特性还提供客户端和服务器端验证。 最小值对数据库架构没有任何影响。

`StringLength` 特性不会阻止用户在名称中输入空格。 [RegularExpression](/dotnet/api/system.componentmodel.dataannotations.regularexpressionattribute) 特性可用于向输入应用限制。 例如，以下代码要求第一个字符为大写，其余字符按字母顺序排列：

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z]*$")]
```

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在“SQL Server 对象资源管理器”(SSOX) 中，双击 Student 表，打开 Student 表设计器 。

![迁移前 SSOX 中的 Student 表](complex-data-model/_static/ssox-before-migration.png)

上图显示 `Student` 表的架构。 名称字段具有类型 `nvarchar(MAX)`。 在本教程的后续部分创建和应用迁移时，因为字符串的长度特性，名称字段将变为 `nvarchar(50)`。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

在 SQLite 工具中检查 `Student` 表的列定义。 名称字段具有类型 `Text`。 请注意，首个名称字段名为 `FirstMidName`。 下一节会将该列的名称更改为 `FirstName`。

---

### <a name="the-column-attribute"></a>Column 特性

```csharp
[Column("FirstName")]
public string FirstMidName { get; set; }
```

特性可以控制类和属性映射到数据库的方式。 在 `Student` 模型中，`Column` 特性用于将 `FirstMidName` 属性的名称映射到数据库中的“FirstName”。

创建数据库后，模型上的属性名将用作列名（使用 `Column` 特性时除外）。 `Student` 模型使用 `FirstMidName` 作为名字字段，因为该字段也可能包含中间名。

使用 `[Column]` 特性，数据模型中的 `Student.FirstMidName` 可映射到 `Student` 表的 `FirstName` 列。 添加 `Column` 特性后，`SchoolContext` 的支持模型会发生改变。 `SchoolContext` 的支持模型将不再与数据库匹配。 这种差异将通过在本教程后面添加迁移来解决。

### <a name="the-required-attribute"></a>Required 特性

```csharp
[Required]
```

`Required` 特性使名称属性成为必填字段。 值类型（`DateTime`、`int` 和 `double`）等不可为 null 的类型不需要 `Required` 特性。 系统会将不可为 NULL 的类型自动视为必填字段。

`Required` 特性必须与 `MinimumLength` 结合使用才能强制执行 `MinimumLength`。

```csharp
[Display(Name = "Last Name")]
[Required]
[StringLength(50, MinimumLength=2)]
public string LastName { get; set; }
```

`MinimumLength` 和 `Required` 允许通过空格来满足验证。 使用 `RegularExpression` 特性来完全控制字符串。

### <a name="the-display-attribute"></a>Display 特性

```csharp
[Display(Name = "Last Name")]
```

`Display` 特性指定文本框的标题栏应为“FirstName”、“LastName”、“FullName”和“EnrollmentDate”。 标题栏默认不使用空格分隔词语，如“Lastname”。

### <a name="create-a-migration"></a>创建迁移

运行应用并转到“学生”页。 此时引发异常。 `[Column]` 特性导致 EF 希望查找名为 `FirstName` 的列，但数据库中的列名称仍为 `FirstMidName`。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

错误消息类似于以下示例：

```
SqlException: Invalid column name 'FirstName'.
```

* 在 PMC 中，输入以下命令以创建新迁移并更新数据库：

  ```powershell
  Add-Migration ColumnFirstName
  Update-Database
  ```

  其中的第一个命令将生成以下警告消息：

  ```text
  An operation was scaffolded that may result in the loss of data.
  Please review the migration for accuracy.
  ```

  生成警告的原因是名称字段现已限制为 50 个字符。 如果数据库中的名称超过 50 个字符，则第 51 个字符及后面的所有字符都将丢失。

* 在 SSOX 中打开 Student 表：

  ![迁移后 SSOX 中的 Students 表](complex-data-model/_static/ssox-after-migration.png)

  执行迁移前，名称列的类型为 [nvarchar (MAX)](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql)。 名称列现在的类型为 `nvarchar(50)`。 列名已从 `FirstMidName` 更改为 `FirstName`。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

错误消息类似于以下示例：

```
SqliteException: SQLite Error 1: 'no such column: s.FirstName'.
```

* 在项目文件夹中打开命令窗口。 输入以下命令以创建新迁移并更新数据库：

  ```dotnetcli
  dotnet ef migrations add ColumnFirstName
  dotnet ef database update
  ```

  数据库更新命令显示类似于以下示例的错误：

  ```text
  SQLite does not support this migration operation ('AlterColumnOperation'). For more information, see http://go.microsoft.com/fwlink/?LinkId=723262.
  ```

在本教程中，解决此错误的方法是删除并重新创建初始迁移。 有关详细信息，请参阅[迁移教程](xref:data/ef-rp/migrations)顶部的 SQLite 警告说明。

* 删除“Migrations”文件夹。
* 运行以下命令以删除数据库，创建新的初始迁移，并应用该迁移：

  ```dotnetcli
  dotnet ef database drop --force
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

* 在 SQLite 工具中检查 Student 表。 以前为 FirstMidName 的列现在为 FirstName。

---

* 运行应用并转到“学生”页。
* 请注意，时间和日期既未输入，也未显示。
* 选择“新建”，尝试输入不超过 50 个字符的名称。

> [!Note]
> 在以下部分中，在某些阶段生成应用会生成编译器错误。 说明用于指定生成应用的时间。

## <a name="the-instructor-entity"></a>Instructor 实体

![Instructor 实体](complex-data-model/_static/instructor-entity.png)

用以下代码创建 Models/Instructor.cs：

[!code-csharp[](intro/samples/cu30/Models/Instructor.cs)]

一行可包含多个特性。 可按如下方式编写 `HireDate` 特性：

```csharp
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

### <a name="navigation-properties"></a>导航属性

`CourseAssignments` 和 `OfficeAssignment` 是导航属性。

一名讲师可以教授任意数量的课程，因此 `CourseAssignments` 定义为集合。

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

讲师最多可以有一个办公室，因此 `OfficeAssignment` 属性包含一个 `OfficeAssignment` 实体。 如果未分配办公室，则 `OfficeAssignment` 为 NULL。

```csharp
public OfficeAssignment OfficeAssignment { get; set; }
```

## <a name="the-officeassignment-entity"></a>OfficeAssignment 实体

![OfficeAssignment 实体](complex-data-model/_static/officeassignment-entity.png)

用以下代码创建 Models/OfficeAssignment.cs：

[!code-csharp[](intro/samples/cu30/Models/OfficeAssignment.cs)]

### <a name="the-key-attribute"></a>Key 特性

`[Key]` 特性用于在属性名不是 classnameID 或 ID 时将属性标识为主键 (PK)。

`Instructor` 和 `OfficeAssignment` 实体之间存在一对零或一关系。 仅当与分配到办公室的讲师之间建立关系时才存在办公室分配。 `OfficeAssignment` PK 也是其到 `Instructor` 实体的外键 (FK)。

EF Core 无法自动将 `InstructorID` 识别为 `OfficeAssignment` 的 PK，因为 `InstructorID` 不遵循 ID 或 classnameID 命名约定。 因此，`Key` 特性用于将 `InstructorID` 识别为 PK：

```csharp
[Key]
public int InstructorID { get; set; }
```

默认情况下，EF Core 将键视为非数据库生成，因为该列面向的是识别关系。

### <a name="the-instructor-navigation-property"></a>Instructor 导航属性

`Instructor.OfficeAssignment` 导航属性可以为 null，因为给定的讲师可能没有 `OfficeAssignment` 行。 一名讲师可能没有办公室分配。

`OfficeAssignment.Instructor` 导航属性将始终具有一个 Instructor 实体，因为外键 `InstructorID` 类型为 `int`，不可为 null 的值类型。 没有讲师则不可能存在办公室分配。

当 `Instructor` 实体具有相关 `OfficeAssignment` 实体时，每个实体都具有对其导航属性中的另一个实体的引用。

## <a name="the-course-entity"></a>Course 实体

![Course 实体](complex-data-model/_static/course-entity.png)

用以下代码更新 *Models/Course.cs*：

[!code-csharp[](intro/samples/cu30/Models/Course.cs?highlight=2,10,13,16,19,21,23)]

`Course` 实体具有外键 (FK) 属性 `DepartmentID`。 `DepartmentID` 指向相关的 `Department` 实体。 `Course` 实体具有 `Department` 导航属性。

当数据模型具有相关实体的导航属性时，EF Core 不要求此模型具有外键属性。 EF Core 可在数据库中的任何所需位置自动创建 FK。 EF Core 为自动创建的 FK 创建[阴影属性](/ef/core/modeling/shadow-properties)。 然而，在数据模型中显式包含 FK 可使更新更简单和更高效。 例如，假设某个模型中不包含 FK 属性 `DepartmentID`。 当提取 Course 实体进行编辑时：

* 如果未显式加载，则 `Department` 属性为 NULL。
* 若要更新 Course 实体，则必须先提取 `Department` 实体。

如果数据模型中包含 FK 属性 `DepartmentID`，则无需在更新前提取 `Department` 实体。

### <a name="the-databasegenerated-attribute"></a>DatabaseGenerated 特性

`[DatabaseGenerated(DatabaseGeneratedOption.None)]` 特性指定 PK 由应用程序提供而不是由数据库生成。

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

默认情况下，EF Core 假定 PK 值由数据库生成。 数据库生成通常是最佳方法。 `Course` 实体的 PK 由用户指定。 例如，对于课程编号，数学系可以使用 1000 系列的编号，英语系可以使用 2000 系列的编号。

`DatabaseGenerated` 特性还可用于生成默认值。 例如，数据库可以自动生成日期字段以记录数据行的创建或更新日期。 有关详细信息，请参阅[生成的属性](/ef/core/modeling/generated-properties)。

### <a name="foreign-key-and-navigation-properties"></a>外键和导航属性

`Course` 实体中的外键 (FK) 属性和导航属性可反映以下关系：

课程将分配到一个系，因此将存在 `DepartmentID` FK 和 `Department` 导航属性。

```csharp
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

参与一门课程的学生数量不定，因此 `Enrollments` 导航属性是一个集合：

```csharp
public ICollection<Enrollment> Enrollments { get; set; }
```

一门课程可能由多位讲师讲授，因此 `CourseAssignments` 导航属性是一个集合：

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

`CourseAssignment` 在[后文](#many-to-many-relationships)介绍。

## <a name="the-department-entity"></a>Department 实体

![Department 实体](complex-data-model/_static/department-entity.png)

用以下代码创建 Models/Department.cs：

[!code-csharp[](intro/samples/cu30snapshots/5-complex/Models/Department1.cs)]

### <a name="the-column-attribute"></a>Column 特性

`Column` 特性以前用于更改列名映射。 在 `Department` 实体的代码中，`Column` 特性用于更改 SQL 数据类型映射。 `Budget` 列通过数据库中的 SQL Server 货币类型进行定义：

```csharp
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

通常不需要列映射。 EF Core 基于属性的 CLR 类型选择相应的 SQL Server 数据类型。 CLR `decimal` 类型会映射到 SQL Server `decimal` 类型。 `Budget` 用于货币，但货币数据类型更适合货币。

### <a name="foreign-key-and-navigation-properties"></a>外键和导航属性

FK 和导航属性可反映以下关系：

* 一个系可能有也可能没有管理员。
* 管理员始终由讲师担任。 因此，`InstructorID` 属性作为到 `Instructor` 实体的 FK 包含在其中。

导航属性名为 `Administrator`，但其中包含 `Instructor` 实体：

```csharp
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

上面代码中的问号 (?) 指定属性可以为 NULL。

一个系可以有多门课程，因此存在 Course 导航属性：

```csharp
public ICollection<Course> Courses { get; set; }
```

按照约定，EF Core 能针对不可为 NULL 的 FK 和多对多关系启用级联删除。 此默认行为可能导致形成循环级联删除规则。 循环级联删除规则会在添加迁移时引发异常。

例如，如果将 `Department.InstructorID` 属性定义为不可为 null，EF Core 将配置级联删除规则。 在这种情况下，指定为管理员的讲师被删除时，该学院将被删除。 在这种情况下，限制规则将更有意义。 以下 [fluent API](#fluent-api-alternative-to-attributes) 将设置限制规则并禁用级联规则。

  ```csharp
  modelBuilder.Entity<Department>()
     .HasOne(d => d.Administrator)
     .WithMany()
     .OnDelete(DeleteBehavior.Restrict)
  ```

## <a name="the-enrollment-entity"></a>Enrollment 实体

一份注册记录面向一名学生所注册的一门课程。

![Enrollment 实体](complex-data-model/_static/enrollment-entity.png)

用以下代码更新 *Models/Enrollment.cs*：

[!code-csharp[](intro/samples/cu30/Models/Enrollment.cs?highlight=1-2,16)]

### <a name="foreign-key-and-navigation-properties"></a>外键和导航属性

FK 属性和导航属性可反映以下关系：

注册记录面向一门课程，因此存在 `CourseID` FK 属性和 `Course` 导航属性：

```csharp
public int CourseID { get; set; }
public Course Course { get; set; }
```

一份注册记录针对一名学生，因此存在 `StudentID` FK 属性和 `Student` 导航属性：

```csharp
public int StudentID { get; set; }
public Student Student { get; set; }
```

## <a name="many-to-many-relationships"></a>多对多关系

`Student` 和 `Course` 实体之间存在多对多关系。 `Enrollment` 实体充当数据库中“具有有效负载”的多对多联接表。 “具有有效负载”表示 `Enrollment` 表除了联接表的 FK 外还包含其他数据（本教程中为 PK 和 `Grade`）。

下图显示这些关系在实体关系图中的外观。 （此关系图是使用适用于 EF 6.X 的 [EF Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition) 生成的。 本教程不介绍如何创建此关系图。）

![学生-课程之间的多对多关系](complex-data-model/_static/student-course.png)

每条关系线的一端显示 1，另一端显示星号 (*)，这表示一对多关系。

如果 `Enrollment` 表不包含年级信息，则它只需包含两个 FK（`CourseID` 和 `StudentID`）。 无有效负载的多对多联接表有时称为纯联接表 (PJT)。

`Instructor` 和 `Course` 实体具有使用纯联接表的多对多关系。

注意：EF 6.x 支持多对多关系的隐式联接表，但 EF Core 不支持。 有关详细信息，请参阅 [EF Core 2.0 中的多对多关系](https://blog.oneunicorn.com/2017/09/25/many-to-many-relationships-in-ef-core-2-0-part-1-the-basics/)。

## <a name="the-courseassignment-entity"></a>CourseAssignment 实体

![CourseAssignment 实体](complex-data-model/_static/courseassignment-entity.png)

用以下代码创建 Models/CourseAssignment.cs：

[!code-csharp[](intro/samples/cu30/Models/CourseAssignment.cs)]

“讲师-课程”的多对多关系要求使用联接表，而该联接表的实体则为 CourseAssignment。

![讲师-课程 m:M](complex-data-model/_static/courseassignment.png)

常规做法是将联接实体命名为 `EntityName1EntityName2`。 例如，使用此模式的“讲师-课程”联接表将是 `CourseInstructor`。 但是，我们建议使用可描述关系的名称。

数据模型开始时很简单，其内容会逐渐增加。 无有效负载的联接表 (PJT) 通常会发展为包含有效负载。 该名称以描述性实体名称开始，因此不需要随联接表更改而更改。 理想情况下，联接实体在业务领域中可能具有专业名称（可能是一个词）。 例如，可以使用名为“阅读率”的联接实体链接“书籍”和“读客”。 对于“讲师-课程”的多对多关系，使用 `CourseAssignment` 比使用`CourseInstructor`更合适。

### <a name="composite-key"></a>组合键

`CourseAssignment` 中的两个 FK（`InstructorID` 和 `CourseID`）共同唯一标识 `CourseAssignment` 表的每一行。 `CourseAssignment` 不需要专用的 PK。 `InstructorID` 和 `CourseID` 属性充当组合 PK。 使用 Fluent API 是向 EF Core 指定组合 PK 的唯一方法。 下一部分演示如何配置组合 PK。

组合键可确保：

* 允许一门课程对应多行。
* 允许一名讲师对应多行。
* 不允许同一讲师和课程对应多行。

`Enrollment` 联接实体定义其自己的 PK，因此可能会出现此类重复。 若要防止此类重复：

* 请在 FK 字段上添加唯一索引，或
* 配置具有主要组合键（与 `CourseAssignment` 类似）的 `Enrollment`。 有关详细信息，请参阅[索引](/ef/core/modeling/indexes)。

## <a name="update-the-database-context"></a>更新数据库上下文

使用以下代码更新 Data/SchoolContext.cs：

[!code-csharp[](intro/samples/cu30/Data/SchoolContext.cs?highlight=15-18,25-31)]

上面的代码添加新实体并配置 `CourseAssignment` 实体的组合 PK。

## <a name="fluent-api-alternative-to-attributes"></a>用 Fluent API 替代特性

上面代码中的 `OnModelCreating` 方法使用 Fluent API 配置 EF Core 行为。 API 称为“Fluent”，因为它通常在将一系列方法调用连接成单个语句后才能使用。 [下面的代码](/ef/core/modeling/#use-fluent-api-to-configure-a-model)是 Fluent API 的示例：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

在本教程中，fluent API 仅用于不能通过特性完成的数据库映射。 但是，Fluent API 可以指定可通过特性完成的大多数格式设置、验证和映射规则。

`MinimumLength` 等特性不能通过 Fluent API 应用。 `MinimumLength` 不会更改架构，它仅应用最小长度验证规则。

某些开发者倾向于仅使用 Fluent API 以保持实体类的“纯净”。 特性和 Fluent API 可以相互混合。 某些配置只能通过 Fluent API 完成（指定组合 PK）。 有些配置只能通过特性完成 (`MinimumLength`)。 使用 Fluent API 或特性的建议做法：

* 选择以下两种方法之一。
* 尽可能以前后一致的方法使用所选的方法。

本教程中使用的某些特性可用于：

* 仅限验证（例如，`MinimumLength`）。
* 仅限 EF Core 配置（例如，`HasKey`）。
* 验证和 EF Core 配置（例如，`[StringLength(50)]`）。

有关特性和 Fluent API 的详细信息，请参阅[配置方法](/ef/core/modeling/)。

## <a name="entity-diagram"></a>实体关系图

下图显示 EF Power Tools 针对已完成的学校模型创建的关系图。

![实体关系图](complex-data-model/_static/diagram.png)

上面的关系图显示：

* 几条一对多关系线（1 到 \*）。
* `Instructor` 和 `OfficeAssignment` 实体之间的一对零或一关系线（1 到 0..1）。
* `Instructor` 和 `Department` 实体之间的零或一到多关系线（0..1 到 *）。

## <a name="seed-the-database"></a>设定数据库种子

更新 Data/DbInitializer.cs 中的代码：

[!code-csharp[](intro/samples/cu30/Data/DbInitializer.cs)]

前面的代码为新实体提供种子数据。 大多数此类代码会创建新实体对象并加载示例数据。 示例数据用于测试。 有关如何对多对多联接表进行种子设定的示例，请参阅 `Enrollments` 和 `CourseAssignments`。

## <a name="add-a-migration"></a>添加迁移

生成项目。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 PMC 中运行以下命令。

```powershell
Add-Migration ComplexDataModel
```

前面的命令显示可能存在数据丢失的相关警告。

```text
An operation was scaffolded that may result in the loss of data.
Please review the migration for accuracy.
To undo this action, use 'ef migrations remove'
```

如果运行 `database update` 命令，则会生成以下错误：

```text
The ALTER TABLE statement conflicted with the FOREIGN KEY constraint "FK_dbo.Course_dbo.Department_DepartmentID". The conflict occurred in
database "ContosoUniversity", table "dbo.Department", column 'DepartmentID'.
```

下一节将介绍如何处理此错误。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

如果添加迁移并运行 `database update` 命令，则将生成以下错误：

```text
SQLite does not support this migration operation ('DropForeignKeyOperation').
For more information, see http://go.microsoft.com/fwlink/?LinkId=723262.
```

下一节将介绍如何避免此错误。

---

## <a name="apply-the-migration-or-drop-and-re-create"></a>应用迁移或删除并重新创建

现已有一个数据库，需要考虑如何将更改应用到其中。 本教程演示两种替代方法：

* [删除并重新创建数据库](#drop)。 如果使用 SQLite，请选择此部分。
* [将迁移应用到现有数据库](#applyexisting)。 本部分的说明仅适用于 SQL Server，不适用于 SQLite。 

这两个选项都适用于 SQL Server。 虽然应用迁移方法更复杂且耗时，但在实际应用和生产环境中为首选方法。 

<a name="drop"></a>

## <a name="drop-and-re-create-the-database"></a>删除并重新创建数据库

如果使用 SQL Server 并且想要在以下部分执行应用迁移方法，[请跳过此部分](#apply-the-migration)。

若要强制 EF Core 创建新的数据库，请删除并更新该数据库：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在“包管理器控制台”(PMC) 中运行以下命令：

  ```powershell
  Drop-Database
  ```

* 删除“Migrations”文件夹，然后运行以下命令：

  ```powershell
  Add-Migration InitialCreate
  Update-Database
  ```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 打开命令窗口并导航到项目文件夹。 项目文件夹包含 ContosoUniversity.csproj 文件。

* 运行下面的命令：

  ```dotnetcli
  dotnet ef database drop --force
  ```

* 删除“Migrations”文件夹，然后运行以下命令：

  ```dotnetcli
  dotnet ef migrations add InitialCreate
  dotnet ef database update
  ```

---

运行应用。 运行应用后将运行 `DbInitializer.Initialize` 方法。 `DbInitializer.Initialize` 将填充新数据库。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在 SSOX 中打开数据库：

* 如果之前已打开过 SSOX，请单击“刷新”按钮。
* 展开“表”节点。 随后将显示出已创建的表。

  ![SSOX 中的表](complex-data-model/_static/ssox-tables.png)

* 查看 CourseAssignment 表：

  * 右键单击 CourseAssignment 表，然后选择“查看数据” 。
  * 验证 CourseAssignment 表包含数据。

  ![SSOX 中的 CourseAssignment 数据](complex-data-model/_static/ssox-ci-data.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

使用 SQLite 工具检查数据库：

* 新的表和列。
* 表中设定种子的数据，例如 CourseAssignment 表。

---

<a name="applyexisting"></a>

## <a name="apply-the-migration"></a>应用迁移

本部分是可选的。 这些步骤仅适用于 SQL Server LocalDB，并且仅当跳过前面的[删除并重新创建数据库](#drop)部分时才适用。

当现有数据与迁移一起运行时，可能存在不满足现有数据的 FK 约束。 使用生产数据时，必须采取步骤来迁移现有数据。 本部分提供修复 FK 约束冲突的示例。 务必在备份后执行这些代码更改。 如果已完成上述[删除并重新创建数据库](#drop)部分，请不要更改这些代码。

{timestamp}_ComplexDataModel.cs 文件包含以下代码：

[!code-csharp[](intro/samples/cu30snapshots/5-complex/Migrations/ComplexDataModel.cs?name=snippet_DepartmentID)]

上面的代码将向 `Course` 表添加不可为 NULL 的 `DepartmentID` FK。 前面教程中的数据库在 `Course` 中包含行，以便迁移时不会更新表。

若要使 `ComplexDataModel` 迁移可与现有数据搭配运行：

* 请更改代码以便为新列 (`DepartmentID`) 赋予默认值。
* 创建名为“临时”的虚拟系来充当默认的系。

#### <a name="fix-the-foreign-key-constraints"></a>修复外键约束

在 `ComplexDataModel` 迁移类中，更新 `Up` 方法：

* 打开 {timestamp}_ComplexDataModel.cs 文件。
* 对将 `DepartmentID` 列添加到 `Course` 表的代码行添加注释。

[!code-csharp[](intro/samples/cu30snapshots/5-complex/Migrations/ComplexDataModel.cs?name=snippet_CommentOut&highlight=9-13)]

添加以下突出显示的代码。 新代码在 `.CreateTable( name: "Department"` 块后：

[!code-csharp[](intro/samples/cu30snapshots/5-complex/Migrations/ComplexDataModel.cs?name=snippet_CreateDefaultValue&highlight=23-31)]

经过上面的更改，现有的 `Course` 行将在 `ComplexDataModel.Up` 方法运行后与“临时”院系建立联系。

本教程简化了此处所示的处理方式。 生产应用可能：

* 包含用于将 `Department` 行和相关 `Course` 行添加到新 `Department` 行的代码或脚本。
* 不会使用“临时”系或 `Course.DepartmentID` 的默认值。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在“包管理器控制台”(PMC) 中运行以下命令：

  ```powershell
  Update-Database
  ```

由于 `DbInitializer.Initialize` 方法仅适用于空数据库，因此请使用 SSOX 删除 Student 表和 Course 表中的所有行。 （级联删除将负责 Enrollment 表。）

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

* 如果在 Visual Studio Code 中使用 SQL Server LocalDB，请运行以下命令：

  ```dotnetcli
  dotnet ef database update
  ```

---

运行应用。 运行应用后将运行 `DbInitializer.Initialize` 方法。 `DbInitializer.Initialize` 将填充新数据库。

## <a name="next-steps"></a>后续步骤

接下来的两个教程演示如何读取和更新相关数据。

> [!div class="step-by-step"]
> [上一个教程](xref:data/ef-rp/migrations)
> [下一个教程](xref:data/ef-rp/read-related-data)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

前面的教程介绍了由三个实体组成的基本数据模型。 本教程将演示如何：

* 添加更多实体和关系。
* 通过指定格式设置、验证和数据库映射规则来自定义数据模型。

已完成数据模型的实体类如下图所示：

![实体关系图](complex-data-model/_static/diagram.png)

如果遇到无法解决的问题，请下载[已完成应用](
https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples)。

## <a name="customize-the-data-model-with-attributes"></a>使用特性自定义数据模型

此部分将使用特性自定义数据模型。

### <a name="the-datatype-attribute"></a>DataType 特性

学生页面当前显示注册日期。 通常情况下，日期字段仅显示日期，不显示时间。

用以下突出显示的代码更新 *Models/Student.cs*：

[!code-csharp[](intro/samples/cu21/Models/Student.cs?name=snippet_DataType&highlight=3,12-13)]

[DataType](/dotnet/api/system.componentmodel.dataannotations.datatypeattribute) 特性指定比数据库内部类型更具体的数据类型。 在此情况下，应仅显示日期，而不是日期加时间。 [DataType 枚举](/dotnet/api/system.componentmodel.dataannotations.datatype)提供多种数据类型，例如日期、时间、电话号码、货币、电子邮件地址等。应用还可通过 `DataType` 特性自动提供类型特定的功能。 例如：

* `mailto:` 链接将依据 `DataType.EmailAddress` 自动创建。
* 大多数浏览器中都提供面向 `DataType.Date` 的日期选择器。

`DataType` 特性发出 HTML 5 `data-`（读作 data dash）特性供 HTML 5 浏览器使用。 `DataType` 特性不提供验证。

`DataType.Date` 不指定显示日期的格式。 默认情况下，日期字段根据基于服务器的 [CultureInfo](xref:fundamentals/localization#provide-localized-resources-for-the-languages-and-cultures-you-support) 的默认格式进行显示。

`DisplayFormat` 特性用于显式指定日期格式：

```csharp
[DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

`ApplyFormatInEditMode` 设置指定还应对编辑 UI 应用该格式设置。 某些字段不应使用 `ApplyFormatInEditMode`。 例如，编辑文本框中通常不应显示货币符号。

`DisplayFormat` 特性可由其本身使用。 搭配使用 `DataType` 特性和 `DisplayFormat` 特性通常是很好的做法。 `DataType` 特性按照数据在屏幕上的呈现方式传达数据的语义。 `DataType` 特性可提供 `DisplayFormat` 中所不具有的以下优点：

* 浏览器可启用 HTML5 功能。 例如，显示日历控件、区域设置适用的货币符号、电子邮件链接、客户端输入验证等。
* 默认情况下，浏览器将根据区域设置采用正确的格式呈现数据。

有关详细信息，请参阅 [\<input> 标记帮助程序文档](xref:mvc/views/working-with-forms#the-input-tag-helper)。

运行应用。 导航到学生索引页。 将不再显示时间。 使用 `Student` 模型的每个视图将显示日期，不显示时间。

![“学生”索引页显示不带时间的日期](complex-data-model/_static/dates-no-times.png)

### <a name="the-stringlength-attribute"></a>StringLength 特性

可使用特性指定数据验证规则和验证错误消息。 [StringLength](/dotnet/api/system.componentmodel.dataannotations.stringlengthattribute) 特性指定数据字段中允许的字符的最小长度和最大长度。 `StringLength` 特性还提供客户端和服务器端验证。 最小值对数据库架构没有任何影响。

使用以下代码更新 `Student` 模型：

[!code-csharp[](intro/samples/cu21/Models/Student.cs?name=snippet_StringLength&highlight=10,12)]

上面的代码将名称限制为不超过 50 个字符。 `StringLength` 特性不会阻止用户在名称中输入空格。 [RegularExpression](/dotnet/api/system.componentmodel.dataannotations.regularexpressionattribute) 特性用于向输入应用限制。 例如，以下代码要求第一个字符为大写，其余字符按字母顺序排列：

```csharp
[RegularExpression(@"^[A-Z]+[a-zA-Z]*$")]
```

运行应用：

* 导航到学生页。
* 选择“新建”并输入不超过 50 个字符的名称。
* 选择“创建”时，客户端验证会显示一条错误消息。

![显示字符串长度错误的“学生索引”页](complex-data-model/_static/string-length-errors.png)

在“SQL Server 对象资源管理器”(SSOX) 中，双击 Student 表，打开 Student 表设计器 。

![迁移前 SSOX 中的 Student 表](complex-data-model/_static/ssox-before-migration.png)

上图显示 `Student` 表的架构。 名称字段的类型为 `nvarchar(MAX)`，因为数据库上尚未运行迁移。 稍后在本教程中运行迁移时，名称字段将变成 `nvarchar(50)`。

### <a name="the-column-attribute"></a>Column 特性

特性可以控制类和属性映射到数据库的方式。 在本部分，`Column` 特性用于将 `FirstMidName` 属性的名称映射到数据库中的“FirstName”。

创建数据库后，模型上的属性名将用作列名（使用 `Column` 特性时除外）。

`Student` 模型使用 `FirstMidName` 作为名字字段，因为该字段也可能包含中间名。

用以下突出显示的代码更新 *Student.cs* 文件：

[!code-csharp[](intro/samples/cu21/Models/Student.cs?name=snippet_Column&highlight=4,14)]

进行上述更改后，应用中的 `Student.FirstMidName` 将映射到 `Student` 表的 `FirstName` 列。

添加 `Column` 特性后，`SchoolContext` 的支持模型会发生改变。 `SchoolContext` 的支持模型将不再与数据库匹配。 如果在执行迁移前运行应用，则会生成如下异常：

```
SqlException: Invalid column name 'FirstName'.
```

若要更新数据库：

* 生成项目。
* 在项目文件夹中打开命令窗口。 输入以下命令以创建新迁移并更新数据库：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
Add-Migration ColumnFirstName
Update-Database
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet ef migrations add ColumnFirstName
dotnet ef database update
```

---

`migrations add ColumnFirstName` 命令将生成如下警告消息：

```text
An operation was scaffolded that may result in the loss of data.
Please review the migration for accuracy.
```

生成警告的原因是名称字段现已限制为 50 个字符。 如果数据库中的名称超过 50 个字符，则第 51 个字符及后面的所有字符都将丢失。

* 测试应用。

在 SSOX 中打开 Student 表：

![迁移后 SSOX 中的 Students 表](complex-data-model/_static/ssox-after-migration.png)

执行迁移前，名称列的类型为 [nvarchar (MAX)](/sql/t-sql/data-types/nchar-and-nvarchar-transact-sql)。 名称列现在的类型为 `nvarchar(50)`。 列名已从 `FirstMidName` 更改为 `FirstName`。

> [!Note]
> 在下一部分中，在某些阶段生成应用会生成编译器错误。 说明用于指定生成应用的时间。

## <a name="student-entity-update"></a>Student 实体更新

![Student 实体](complex-data-model/_static/student-entity.png)

用以下代码更新 *Models/Student.cs*：

[!code-csharp[](intro/samples/cu21/Models/Student.cs?name=snippet_BeforeInheritance&highlight=11,13,15,18,22,24-31)]

### <a name="the-required-attribute"></a>Required 特性

`Required` 特性使名称属性成为必填字段。 值类型（`DateTime`、`int`、`double`）等不可为 NULL 的类型不需要 `Required` 特性。 系统会将不可为 NULL 的类型自动视为必填字段。

不能用 `StringLength` 特性中的最短长度参数替换 `Required` 特性：

```csharp
[Display(Name = "Last Name")]
[StringLength(50, MinimumLength=1)]
public string LastName { get; set; }
```

### <a name="the-display-attribute"></a>Display 特性

`Display` 特性指定文本框的标题栏应为“FirstName”、“LastName”、“FullName”和“EnrollmentDate”。 标题栏默认不使用空格分隔词语，如“Lastname”。

### <a name="the-fullname-calculated-property"></a>FullName 计算属性

`FullName` 是计算属性，可返回通过串联两个其他属性创建的值。 `FullName` 不能设置并且仅具有一个 get 访问器。 数据库中不会创建任何 `FullName` 列。

## <a name="create-the-instructor-entity"></a>创建 Instructor 实体

![Instructor 实体](complex-data-model/_static/instructor-entity.png)

用以下代码创建 Models/Instructor.cs：

[!code-csharp[](intro/samples/cu21/Models/Instructor.cs)]

一行可包含多个特性。 可按如下方式编写 `HireDate` 特性：

```csharp
[DataType(DataType.Date),Display(Name = "Hire Date"),DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}", ApplyFormatInEditMode = true)]
```

### <a name="the-courseassignments-and-officeassignment-navigation-properties"></a>CourseAssignments 和 OfficeAssignment 导航属性

`CourseAssignments` 和 `OfficeAssignment` 是导航属性。

一名讲师可以教授任意数量的课程，因此 `CourseAssignments` 定义为集合。

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

如果导航属性包含多个实体：

* 它必须是可在其中添加、删除和更新实体的列表类型。

导航属性类型包括：

* `ICollection<T>`
* `List<T>`
* `HashSet<T>`

如果指定了 `ICollection<T>`，EF Core 会默认创建 `HashSet<T>` 集合。

`CourseAssignment` 实体在“多对多关系”部分进行介绍。

Contoso University 业务规则规定一名讲师最多可获得一间办公室。 `OfficeAssignment` 属性包含一个 `OfficeAssignment` 实体。 如果未分配办公室，则 `OfficeAssignment` 为 NULL。

```csharp
public OfficeAssignment OfficeAssignment { get; set; }
```

## <a name="create-the-officeassignment-entity"></a>创建 OfficeAssignment 实体

![OfficeAssignment 实体](complex-data-model/_static/officeassignment-entity.png)

用以下代码创建 Models/OfficeAssignment.cs：

[!code-csharp[](intro/samples/cu21/Models/OfficeAssignment.cs)]

### <a name="the-key-attribute"></a>Key 特性

`[Key]` 特性用于在属性名不是 classnameID 或 ID 时将属性标识为主键 (PK)。

`Instructor` 和 `OfficeAssignment` 实体之间存在一对零或一关系。 仅当与分配到办公室的讲师之间建立关系时才存在办公室分配。 `OfficeAssignment` PK 也是其到 `Instructor` 实体的外键 (FK)。 EF Core 无法自动将 `InstructorID` 识别为 `OfficeAssignment` 的 PK，因为：

* `InstructorID` 不遵循 ID 或 classnameID 命名约定。

因此，`Key` 特性用于将 `InstructorID` 识别为 PK：

```csharp
[Key]
public int InstructorID { get; set; }
```

默认情况下，EF Core 将键视为非数据库生成，因为该列面向的是识别关系。

### <a name="the-instructor-navigation-property"></a>Instructor 导航属性

`Instructor` 实体的 `OfficeAssignment` 导航属性可以为 NULL，因为：

* 引用类型（例如，类可以为 NULL）。
* 一名讲师可能没有办公室分配。

`OfficeAssignment` 实体具有不可为 NULL 的 `Instructor` 导航属性，因为：

* `InstructorID` 不可为 NULL。
* 没有讲师则不可能存在办公室分配。

当 `Instructor` 实体具有相关 `OfficeAssignment` 实体时，每个实体都具有对其导航属性中的另一个实体的引用。

`[Required]` 特性可以应用于 `Instructor` 导航属性：

```csharp
[Required]
public Instructor Instructor { get; set; }
```

上面的代码指定必须存在相关的讲师。 上面的代码没有必要，因为 `InstructorID` 外键（也是 PK）不可为 NULL。

## <a name="modify-the-course-entity"></a>修改 Course 实体

![Course 实体](complex-data-model/_static/course-entity.png)

用以下代码更新 *Models/Course.cs*：

[!code-csharp[](intro/samples/cu21/Models/Course.cs?name=snippet_Final&highlight=2,10,13,16,19,21,23)]

`Course` 实体具有外键 (FK) 属性 `DepartmentID`。 `DepartmentID` 指向相关的 `Department` 实体。 `Course` 实体具有 `Department` 导航属性。

当数据模型具有相关实体的导航属性时，EF Core 不要求此模型具有 FK 属性。

EF Core 可在数据库中的任何所需位置自动创建 FK。 EF Core 为自动创建的 FK 创建[阴影属性](/ef/core/modeling/shadow-properties)。 数据模型中包含 FK 后可使更新更简单和更高效。 例如，假设某个模型中不包含 FK 属性 `DepartmentID`。 当提取 Course 实体进行编辑时：

* 如果未显式加载 `Department` 实体，则该实体将为 NULL。
* 若要更新 Course 实体，则必须先提取 `Department` 实体。

如果数据模型中包含 FK 属性 `DepartmentID`，则无需在更新前提取 `Department` 实体。

### <a name="the-databasegenerated-attribute"></a>DatabaseGenerated 特性

`[DatabaseGenerated(DatabaseGeneratedOption.None)]` 特性指定 PK 由应用程序提供而不是由数据库生成。

```csharp
[DatabaseGenerated(DatabaseGeneratedOption.None)]
[Display(Name = "Number")]
public int CourseID { get; set; }
```

默认情况下，EF Core 假定 PK 值由数据库生成。 由数据库生成 PK 值通常是最佳方法。 `Course` 实体的 PK 由用户指定。 例如，对于课程编号，数学系可以使用 1000 系列的编号，英语系可以使用 2000 系列的编号。

`DatabaseGenerated` 特性还可用于生成默认值。 例如，数据库可以自动生成日期字段以记录数据行的创建或更新日期。 有关详细信息，请参阅[生成的属性](/ef/core/modeling/generated-properties)。

### <a name="foreign-key-and-navigation-properties"></a>外键和导航属性

`Course` 实体中的外键 (FK) 属性和导航属性可反映以下关系：

课程将分配到一个系，因此将存在 `DepartmentID` FK 和 `Department` 导航属性。

```csharp
public int DepartmentID { get; set; }
public Department Department { get; set; }
```

参与一门课程的学生数量不定，因此 `Enrollments` 导航属性是一个集合：

```csharp
public ICollection<Enrollment> Enrollments { get; set; }
```

一门课程可能由多位讲师讲授，因此 `CourseAssignments` 导航属性是一个集合：

```csharp
public ICollection<CourseAssignment> CourseAssignments { get; set; }
```

`CourseAssignment` 在[后文](#many-to-many-relationships)介绍。

## <a name="create-the-department-entity"></a>创建 Department 实体

![Department 实体](complex-data-model/_static/department-entity.png)

用以下代码创建 Models/Department.cs：

[!code-csharp[](intro/samples/cu21/Models/Department.cs?name=snippet_Begin)]

### <a name="the-column-attribute"></a>Column 特性

`Column` 特性以前用于更改列名映射。 在 `Department` 实体的代码中，`Column` 特性用于更改 SQL 数据类型映射。 `Budget` 列通过数据库中的 SQL Server 货币类型进行定义：

```csharp
[Column(TypeName="money")]
public decimal Budget { get; set; }
```

通常不需要列映射。 EF Core 通常基于属性的 CLR 类型选择相应的 SQL Server 数据类型。 CLR `decimal` 类型会映射到 SQL Server `decimal` 类型。 `Budget` 用于货币，但货币数据类型更适合货币。

### <a name="foreign-key-and-navigation-properties"></a>外键和导航属性

FK 和导航属性可反映以下关系：

* 一个系可能有也可能没有管理员。
* 管理员始终由讲师担任。 因此，`InstructorID` 属性作为到 `Instructor` 实体的 FK 包含在其中。

导航属性名为 `Administrator`，但其中包含 `Instructor` 实体：

```csharp
public int? InstructorID { get; set; }
public Instructor Administrator { get; set; }
```

上面代码中的问号 (?) 指定属性可以为 NULL。

一个系可以有多门课程，因此存在 Course 导航属性：

```csharp
public ICollection<Course> Courses { get; set; }
```

注意：按照约定，EF Core 能针对不可为 NULL 的 FK 和多对多关系启用级联删除。 级联删除可能导致形成循环级联删除规则。 循环级联删除规则会在添加迁移时引发异常。

例如，如果 `Department.InstructorID` 属性定义为不可为 NULL：

* EF Core 会配置级联删除规则，以在删除讲师时删除院系。
* 在删除讲师时删除院系并不是预期行为。
* 以下 [fluent API](#fluent-api-alternative-to-attributes) 将设置限制规则而不是级联规则。

   ```csharp
   modelBuilder.Entity<Department>()
      .HasOne(d => d.Administrator)
      .WithMany()
      .OnDelete(DeleteBehavior.Restrict)
  ```

上面的代码会针对“系-讲师”关系禁用级联删除。

## <a name="update-the-enrollment-entity"></a>更新 Enrollment 实体

一份注册记录面向一名学生所注册的一门课程。

![Enrollment 实体](complex-data-model/_static/enrollment-entity.png)

用以下代码更新 *Models/Enrollment.cs*：

[!code-csharp[](intro/samples/cu21/Models/Enrollment.cs?name=snippet_Final&highlight=1-2,16)]

### <a name="foreign-key-and-navigation-properties"></a>外键和导航属性

FK 属性和导航属性可反映以下关系：

注册记录面向一门课程，因此存在 `CourseID` FK 属性和 `Course` 导航属性：

```csharp
public int CourseID { get; set; }
public Course Course { get; set; }
```

一份注册记录针对一名学生，因此存在 `StudentID` FK 属性和 `Student` 导航属性：

```csharp
public int StudentID { get; set; }
public Student Student { get; set; }
```

## <a name="many-to-many-relationships"></a>多对多关系

`Student` 和 `Course` 实体之间存在多对多关系。 `Enrollment` 实体充当数据库中“具有有效负载”的多对多联接表。 “具有有效负载”表示 `Enrollment` 表除了联接表的 FK 外还包含其他数据（本教程中为 PK 和 `Grade`）。

下图显示这些关系在实体关系图中的外观。 （此关系图是使用适用于 EF 6.X 的 [EF Power Tools](https://marketplace.visualstudio.com/items?itemName=ErikEJ.EntityFramework6PowerToolsCommunityEdition) 生成的。 本教程不介绍如何创建此关系图。）

![学生-课程之间的多对多关系](complex-data-model/_static/student-course.png)

每条关系线的一端显示 1，另一端显示星号 (*)，这表示一对多关系。

如果 `Enrollment` 表不包含年级信息，则它只需包含两个 FK（`CourseID` 和 `StudentID`）。 无有效负载的多对多联接表有时称为纯联接表 (PJT)。

`Instructor` 和 `Course` 实体具有使用纯联接表的多对多关系。

注意：EF 6.x 支持多对多关系的隐式联接表，但 EF Core 不支持。 有关详细信息，请参阅 [EF Core 2.0 中的多对多关系](https://blog.oneunicorn.com/2017/09/25/many-to-many-relationships-in-ef-core-2-0-part-1-the-basics/)。

## <a name="the-courseassignment-entity"></a>CourseAssignment 实体

![CourseAssignment 实体](complex-data-model/_static/courseassignment-entity.png)

用以下代码创建 Models/CourseAssignment.cs：

[!code-csharp[](intro/samples/cu21/Models/CourseAssignment.cs)]

### <a name="instructor-to-courses"></a>讲师-课程

![讲师-课程 m:M](complex-data-model/_static/courseassignment.png)

讲师-课程的多对多关系：

* 要求必须用实体集表示联接表。
* 为纯联接表（无有效负载的表）。

常规做法是将联接实体命名为 `EntityName1EntityName2`。 例如，使用此模式的“讲师-课程”联接表是 `CourseInstructor`。 但是，我们建议使用可描述关系的名称。

数据模型开始时很简单，其内容会逐渐增加。 无有效负载联接 (PJT) 通常会发展为包含有效负载。 该名称以描述性实体名称开始，因此不需要随联接表更改而更改。 理想情况下，联接实体在业务领域中可能具有专业名称（可能是一个词）。 例如，可以使用名为“阅读率”的联接实体链接“书籍”和“读客”。 对于“讲师-课程”的多对多关系，使用 `CourseAssignment` 比使用`CourseInstructor`更合适。

### <a name="composite-key"></a>组合键

FK 不能为 NULL。 `CourseAssignment` 中的两个 FK（`InstructorID` 和 `CourseID`）共同唯一标识 `CourseAssignment` 表的每一行。 `CourseAssignment` 不需要专用的 PK。 `InstructorID` 和 `CourseID` 属性充当组合 PK。 使用 Fluent API 是向 EF Core 指定组合 PK 的唯一方法。 下一部分演示如何配置组合 PK。

组合键可确保：

* 允许一门课程对应多行。
* 允许一名讲师对应多行。
* 不允许相同的讲师和课程对应多行。

`Enrollment` 联接实体定义其自己的 PK，因此可能会出现此类重复。 若要防止此类重复：

* 请在 FK 字段上添加唯一索引，或
* 配置具有主要组合键（与 `CourseAssignment` 类似）的 `Enrollment`。 有关详细信息，请参阅[索引](/ef/core/modeling/indexes)。

## <a name="update-the-db-context"></a>更新数据库上下文

将以下突出显示的代码添加到 Data/SchoolContext.cs：

[!code-csharp[](intro/samples/cu21/Data/SchoolContext.cs?name=snippet_BeforeInheritance&highlight=15-18,25-31)]

上面的代码添加新实体并配置 `CourseAssignment` 实体的组合 PK。

## <a name="fluent-api-alternative-to-attributes"></a>用 Fluent API 替代特性

上面代码中的 `OnModelCreating` 方法使用 Fluent API 配置 EF Core 行为。 API 称为“Fluent”，因为它通常在将一系列方法调用连接成单个语句后才能使用。 [下面的代码](/ef/core/modeling/#use-fluent-api-to-configure-a-model)是 Fluent API 的示例：

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Blog>()
        .Property(b => b.Url)
        .IsRequired();
}
```

在本教程中，Fluent API 仅用于不能通过特性完成的数据库映射。 但是，Fluent API 可以指定可通过特性完成的大多数格式设置、验证和映射规则。

`MinimumLength` 等特性不能通过 Fluent API 应用。 `MinimumLength` 不会更改架构，它仅应用最小长度验证规则。

某些开发者倾向于仅使用 Fluent API 以保持实体类的“纯净”。 特性和 Fluent API 可以相互混合。 某些配置只能通过 Fluent API 完成（指定组合 PK）。 有些配置只能通过特性完成 (`MinimumLength`)。 使用 Fluent API 或特性的建议做法：

* 选择以下两种方法之一。
* 尽可能以前后一致的方法使用所选的方法。

本教程中使用的某些特性可用于：

* 仅限验证（例如，`MinimumLength`）。
* 仅限 EF Core 配置（例如，`HasKey`）。
* 验证和 EF Core 配置（例如，`[StringLength(50)]`）。

有关特性和 Fluent API 的详细信息，请参阅[配置方法](/ef/core/modeling/)。

## <a name="entity-diagram-showing-relationships"></a>显示关系的实体关系图

下图显示 EF Power Tools 针对已完成的学校模型创建的关系图。

![实体关系图](complex-data-model/_static/diagram.png)

上面的关系图显示：

* 几条一对多关系线（1 到 \*）。
* `Instructor` 和 `OfficeAssignment` 实体之间的一对零或一关系线（1 到 0..1）。
* `Instructor` 和 `Department` 实体之间的零或一到多关系线（0..1 到 *）。

## <a name="seed-the-db-with-test-data"></a>使用测试数据为数据库设定种子

更新 Data/DbInitializer.cs 中的代码：

[!code-csharp[](intro/samples/cu21/Data/DbInitializer.cs?name=snippet_Final)]

前面的代码为新实体提供种子数据。 大多数此类代码会创建新实体对象并加载示例数据。 示例数据用于测试。 有关如何对多对多联接表进行种子设定的示例，请参阅 `Enrollments` 和 `CourseAssignments`。

## <a name="add-a-migration"></a>添加迁移

生成项目。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

```powershell
Add-Migration ComplexDataModel
```

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

```dotnetcli
dotnet ef migrations add ComplexDataModel
```

---

前面的命令显示可能存在数据丢失的相关警告。

```text
An operation was scaffolded that may result in the loss of data.
Please review the migration for accuracy.
Done. To undo this action, use 'ef migrations remove'
```

如果运行 `database update` 命令，则会生成以下错误：

```text
The ALTER TABLE statement conflicted with the FOREIGN KEY constraint "FK_dbo.Course_dbo.Department_DepartmentID". The conflict occurred in
database "ContosoUniversity", table "dbo.Department", column 'DepartmentID'.
```

## <a name="apply-the-migration"></a>应用迁移

现已有一个数据库，需要考虑如何将未来的更改应用到其中。 本教程演示两种方法：

* [删除并重新创建数据库](#drop)
* [将迁移应用到现有数据库](#applyexisting)。 虽然此方法更复杂且耗时，但在实际应用和生产环境中为首选方法。 **说明**：这是本教程的一个可选部分。 你可以执行删除和重新创建的相关步骤并跳过此部分。 如果希望执行本部分中的步骤，请勿执行删除和重新创建步骤。 

<a name="drop"></a>

### <a name="drop-and-re-create-the-database"></a>删除并重新创建数据库

已更新 `DbInitializer` 中的代码将为新实体添加种子数据。 若要强制 EF Core 创建新的 DB，请删除并更新 DB：

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

在“包管理器控制台”(PMC) 中运行以下命令：

```powershell
Drop-Database
Update-Database
```

从 PMC 运行 `Get-Help about_EntityFrameworkCore`，获取帮助信息。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

打开命令窗口并导航到项目文件夹。 项目文件夹包含 Startup.cs 文件。

在命令窗口中输入以下内容：

```dotnetcli
dotnet ef database drop
dotnet ef database update
```

---

运行应用。 运行应用后将运行 `DbInitializer.Initialize` 方法。 `DbInitializer.Initialize` 将填充新数据库。

在 SSOX 中打开数据库：

* 如果之前已打开过 SSOX，请单击“刷新”按钮。
* 展开“表”节点。 随后将显示出已创建的表。

![SSOX 中的表](complex-data-model/_static/ssox-tables.png)

查看 CourseAssignment 表：

* 右键单击 CourseAssignment 表，然后选择“查看数据” 。
* 验证 CourseAssignment 表包含数据。

![SSOX 中的 CourseAssignment 数据](complex-data-model/_static/ssox-ci-data.png)

<a name="applyexisting"></a>

### <a name="apply-the-migration-to-the-existing-database"></a>将迁移应用到现有数据库

本部分是可选的。 只有当跳过之前的[删除并重新创建数据库](#drop)部分时才可以执行上述步骤。

当现有数据与迁移一起运行时，可能存在不满足现有数据的 FK 约束。 使用生产数据时，必须采取步骤来迁移现有数据。 本部分提供修复 FK 约束冲突的示例。 务必在备份后执行这些代码更改。 如果已完成上述部分并更新数据库，则不要执行这些代码更改。

{timestamp}_ComplexDataModel.cs 文件包含以下代码：

[!code-csharp[](intro/samples/cu/Migrations/20171027005808_ComplexDataModel.cs?name=snippet_DepartmentID)]

上面的代码将向 `Course` 表添加不可为 NULL 的 `DepartmentID` FK。 前面教程中的数据库在 `Course` 中包含行，以便迁移时不会更新表。

若要使 `ComplexDataModel` 迁移可与现有数据搭配运行：

* 请更改代码以便为新列 (`DepartmentID`) 赋予默认值。
* 创建名为“临时”的虚拟系来充当默认的系。

#### <a name="fix-the-foreign-key-constraints"></a>修复外键约束

更新 `ComplexDataModel` 类 `Up` 方法：

* 打开 {timestamp}_ComplexDataModel.cs 文件。
* 对将 `DepartmentID` 列添加到 `Course` 表的代码行添加注释。

[!code-csharp[](intro/samples/cu/Migrations/20171027005808_ComplexDataModel.cs?name=snippet_CommentOut&highlight=9-13)]

添加以下突出显示的代码。 新代码在 `.CreateTable( name: "Department"` 块后：

[!code-csharp[](intro/samples/cu/Migrations/20171027005808_ComplexDataModel.cs?name=snippet_CreateDefaultValue&highlight=22-32)]

经过上面的更改，现有的 `Course` 行将在 `ComplexDataModel` `Up` 方法运行后与“临时”院系建立联系。

生产应用可能：

* 包含用于将 `Department` 行和相关 `Course` 行添加到新 `Department` 行的代码或脚本。
* 不会使用“临时”系或 `Course.DepartmentID` 的默认值。

下一教程将介绍相关数据。

## <a name="additional-resources"></a>其他资源

* [本教程的 YouTube 版本（第 1 部分）](https://www.youtube.com/watch?v=0n2f0ObgCoA)
* [本教程的 YouTube 版本（第 2 部分）](https://www.youtube.com/watch?v=Je0Z5K1TNmY)

> [!div class="step-by-step"]
> [上一页](xref:data/ef-rp/migrations)
> [下一页](xref:data/ef-rp/read-related-data)

::: moniker-end
