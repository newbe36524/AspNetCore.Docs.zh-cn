---
title: 第 2 部分，ASP.NET Core 中的 Razor 页面和 EF Core - CRUD
author: rick-anderson
description: Razor 页面和实体框架教程系列第 2 部分。
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
uid: data/ef-rp/crud
ms.openlocfilehash: 083214c01dbec6c6f44d6b82f5b514a029e57cbe
ms.sourcegitcommit: d1a897ebd89daa05170ac448e4831d327f6b21a8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2020
ms.locfileid: "91606733"
---
# <a name="part-2-no-locrazor-pages-with-ef-core-in-aspnet-core---crud"></a>第 2 部分，ASP.NET Core 中的 Razor 页面和 EF Core - CRUD

作者：[Tom Dykstra](https://github.com/tdykstra)、[Jon P Smith](https://twitter.com/thereformedprog) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE [about the series](~/includes/RP-EF/intro.md)]

::: moniker range=">= aspnetcore-5.0"

本教程将介绍和自定义已搭建基架的 CRUD （创建、读取、更新、删除）代码。

## <a name="no-repository"></a>无存储库

某些开发人员使用服务层或存储库模式在 UI (Razor Pages) 和数据访问层之间创建抽象层。 本教程不会这样做。 为最大程度降低复杂性并让本教程重点介绍 EF Core，将直接在页面模型类中添加 EF Core 代码。 

## <a name="update-the-details-page"></a>更新“详细信息”页

“学生”页的基架代码不包括注册数据。 本部分将向“详细信息”页添加注册。

### <a name="read-enrollments"></a>读取注册

为了在页面上显示学生的注册数据，你需要读取这些数据。 Pages/Students/Details.cshtml.cs 中的基架代码仅读取学生数据，但不读取注册数据：

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Pages/Students/Details1.cshtml.cs?name=snippet_OnGetAsync&highlight=8)]

使用以下代码替换 `OnGetAsync` 方法以读取所选学生的注册数据。 突出显示所作更改。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Details.cshtml.cs?name=snippet_OnGetAsync&highlight=8-12)]

[Include](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.include) 和 [ThenInclude](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.theninclude#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_ThenInclude__3_Microsoft_EntityFrameworkCore_Query_IIncludableQueryable___0_System_Collections_Generic_IEnumerable___1___System_Linq_Expressions_Expression_System_Func___1___2___) 方法使上下文加载 `Student.Enrollments` 导航属性，并在每个注册中加载 `Enrollment.Course` 导航属性。 这些方法将在[与数据读取相关](xref:data/ef-rp/read-related-data)的教程中进行详细介绍。

对于返回的实体未在当前上下文中更新的情况，[AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) 方法将会提升性能。 `AsNoTracking` 将在本教程的后续部分中讨论。

### <a name="display-enrollments"></a>显示注册

使用以下代码替换 *Pages/Students/Details.cshtml* 中的代码以显示注册列表。 突出显示所作更改。

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Details.cshtml?highlight=32-53)]

上面的代码循环通过 `Enrollments` 导航属性中的实体。 它将针对每个注册显示课程标题和成绩。 课程标题从 Course 实体中检索，该实体存储在 Enrollments 实体的 `Course` 导航属性中。

运行应用，选择“学生”选项卡，然后单击学生的“详细信息”链接 。 随即显示出所选学生的课程和成绩列表。

### <a name="ways-to-read-one-entity"></a>读取一个实体的方法

生成的代码使用 [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Threading_CancellationToken_) 读取一个实体。 如果未找到任何内容，则此方法返回 NULL；否则，它将返回满足查询筛选条件的第一行。 `FirstOrDefaultAsync` 通常是比以下备选方案更好的选择：

* [SingleOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_) - 如果有多个满足查询筛选器的实体，则引发异常。 若要确定查询是否可以返回多行，`SingleOrDefaultAsync` 会尝试提取多个行。 如果查询只能返回一个实体，就像它在搜索唯一键时一样，那么该额外工作是不必要的。
* [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.findasync#Microsoft_EntityFrameworkCore_DbContext_FindAsync_System_Type_System_Object___) - 查找具有主键 ( PK) 的实体。 如果具有 PK 的实体正在由上下文跟踪，会返回该实体且不向数据库发出请求。 此方法经过优化，可查找单个实体，但无法通过 `FindAsync` 调用 `Include`。  如果需要相关数据，`FirstOrDefaultAsync` 则是更好的选择。

### <a name="route-data-vs-query-string"></a>路由数据与查询字符串

“详细信息”页的 URL 是 `https://localhost:<port>/Students/Details?id=1`。 实体的主键值在查询字符串中。 某些开发人员偏向于在路由数据中传递键值：`https://localhost:<port>/Students/Details/1`。 有关详细信息，请参阅[更新生成的代码](xref:tutorials/razor-pages/da1#update-the-generated-code)。

## <a name="update-the-create-page"></a>更新“创建”页

“创建”页面的基架 `OnPostAsync` 代码容易受到[过多发布攻击](#overposting)。 使用以下代码替换 Pages/Students/Create.cshtml.cs 中的 `OnPostAsync` 方法。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Create.cshtml.cs?name=snippet_OnPostAsync)]

<a name="TryUpdateModelAsync"></a>

### <a name="tryupdatemodelasync"></a>TryUpdateModelAsync

前面的代码将创建一个 Student 对象，然后使用发布的表单域更新 Student 对象的属性。 [TryUpdateModelAsync](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.tryupdatemodelasync#Microsoft_AspNetCore_Mvc_ControllerBase_TryUpdateModelAsync_System_Object_System_Type_System_String_) 方法：

* 使用 [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel) 中 [PageContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.pagecontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_PageContext) 属性的已发布的表单值。
* 仅更新列出的属性 (`s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate`)。
* 查找带有“student”前缀的表单值。 例如 `Student.FirstMidName`。 该自变量不区分大小写。
* 使用[模型绑定](xref:mvc/models/model-binding)系统将字符串中的表单值转换为 `Student` 模型中的类型。 例如，`EnrollmentDate` 转换为 `DateTime`。

运行应用，并创建一个学生实体以测试“创建”页。

## <a name="overposting"></a>过多发布

使用 `TryUpdateModel` 更新具有已发布值的字段是一种最佳的安全做法，因为这能阻止过多发布。 例如，假设 Student 实体包含此网页不应更新或添加的 `Secret` 属性：

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Models/StudentZsecret.cs?name=snippet_Intro&highlight=7)]

即使应用的创建或更新 Razor 页面上没有 `Secret` 字段，黑客仍可利用过多发布设置 `Secret` 值。 黑客也可使用 Fiddler 等工具或通过编写某个 JavaScript 来发布 `Secret` 表单值。 原始代码不会限制模型绑定器在创建“学生”实例时使用的字段。

黑客为 `Secret` 表单域指定的任何值都会在数据库中更新。 下图显示 Fiddler 工具正在将 `Secret` 字段（值为“OverPost”）添加到已发布的表单值。

![Fiddler 添加 Secret 字段](../ef-mvc/crud/_static/fiddler.png)

值“OverPost”已成功添加到所插入行的 `Secret` 属性中。 即使应用设计器从未打算用“创建”页设置 `Secret` 属性，也会发生这种情况。

### <a name="view-model"></a>视图模型

视图模型还提供了一种防止过度发布的方法。

应用程序模型通常称为域模型。 域模型通常包含数据库中对应实体所需的全部属性。 视图模型仅包含 UI 页（例如“创建”页）所需的属性。

除视图模型外，某些应用使用绑定模型或输入模型在 Razor Pages 页面模型类和浏览器之间传递数据。 

请考虑以下 `StudentVM` 视图模型：

[!code-csharp[Main](intro/samples/cu50/ViewModels/StudentVM.cs?name=snippet)]

以下代码使用 `StudentVM` 视图模型创建新的学生：

[!code-csharp[Main](intro/samples/cu50/Pages/Students/CreateVM.cshtml.cs?name=snippet)]

[SetValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues.setvalues#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyValues_SetValues_System_Object_) 方法通过从另一个 [PropertyValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues) 对象读取值来设置此对象的值。 `SetValues` 使用属性名称匹配。 视图模型类型：

* 无需与模型类型相关。
* 需要具有匹配的属性。

使用 `StudentVM` 要求“创建”页使用 `StudentVM`，而不是 `Student`：

[!code-cshtml[Main](intro/samples/cu50/Pages/Students/CreateVM.cshtml)]

## <a name="update-the-edit-page"></a>更新“编辑”页

在 Pages/Students/Edit.cshtml.cs 中，使用以下代码替换 `OnGetAsync` 和 `OnPostAsync` 方法。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Edit.cshtml.cs?name=snippet_OnGetPost)]

代码更改与“创建”页类似，但有少数例外：

* 已将 `FirstOrDefaultAsync` 替换为 [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbset-1.findasync)。 不需要包含相关数据时，`FindAsync` 效率更高。
* `OnPostAsync` 具有 `id` 参数。
* 当前学生是从数据库中提取的，而非通过创建空学生获得。

运行应用，并通过创建和编辑学生进行测试。

## <a name="entity-states"></a>实体状态

数据库上下文会随时跟踪内存中的实体是否已与其在数据库中的对应行进行同步。 此跟踪信息可确定调用 [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_) 后的行为。 例如，将新实体传递到 [AddAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.addasync) 方法时，该实体的状态设置为 [Added](/dotnet/api/microsoft.entityframeworkcore.entitystate#Microsoft_EntityFrameworkCore_EntityState_Added)。 调用 `SaveChangesAsync` 时，数据库上下文会发出 SQL `INSERT` 命令。

实体可能处于[以下状态](/dotnet/api/microsoft.entityframeworkcore.entitystate)之一：

* `Added`：数据库中尚不存在实体。 `SaveChanges` 方法发出 `INSERT` 语句。

* `Unchanged`：无需保存对该实体所做的任何更改。 从数据库中读取实体时，该实体具有此状态。

* `Modified`：已修改实体的部分或全部属性值。 `SaveChanges` 方法发出 `UPDATE` 语句。

* `Deleted`：已标记该实体进行删除。 `SaveChanges` 方法发出 `DELETE` 语句。

* `Detached`：数据库上下文未跟踪该实体。

在桌面应用中，通常会自动设置状态更改。 读取实体并执行更改后，实体状态自动更改为 `Modified`。 调用 `SaveChanges` 会生成仅更新已更改属性的 SQL `UPDATE` 语句。

在 Web 应用中，读取实体并显示数据的 `DbContext` 将在页面呈现后进行处理。 调用页面 `OnPostAsync` 方法时，将发出具有 `DbContext` 的新实例的 Web 请求。 如果在这个新的上下文中重新读取实体，则会模拟桌面处理。

## <a name="update-the-delete-page"></a>更新“删除”页

在此部分中，当对 `SaveChanges` 的调用失败时，将实现自定义错误消息。

使用以下代码替换 *Pages/Students/Delete.cshtml.cs* 中的代码。 突出显示所作更改：

[!code-csharp[Main](intro/samples/cu50/Pages/Students/Delete.cshtml.cs?name=snippet_All&highlight=12-14,22,30-33,45-99)]

前面的代码将可选参数 `saveChangesError` 添加到 `OnGetAsync` 方法签名中。 `saveChangesError` 指示学生对象删除失败后是否调用该方法。 删除操作可能由于暂时性网络问题而失败。 数据库在云中时，更可能出现暂时性网络错误。 通过 UI 调用“删除”页 `OnGetAsync` 时，`saveChangesError` 参数为 `false`。 当 `OnPostAsync` 调用 `OnGetAsync`（由于删除操作失败）时，`saveChangesError` 参数为 `true`。

`OnPostAsync` 方法检索所选实体，然后调用 [Remove](/dotnet/api/microsoft.entityframeworkcore.dbcontext.remove#Microsoft_EntityFrameworkCore_DbContext_Remove_System_Object_) 方法将实体的状态设置为 `Deleted`。 调用 `SaveChanges` 时生成 SQL `DELETE` 命令。 如果 `Remove` 失败：

* 捕获数据库异常。
* 通过 `saveChangesError=true` 调用“删除”页 `OnGetAsync` 方法。

向 Pages/Students/Delete.cshtml 添加错误消息：

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Delete.cshtml?highlight=10)]

运行应用并删除学生以测试“删除”页。

## <a name="next-steps"></a>后续步骤

> [!div class="step-by-step"]
> [上一个教程](xref:data/ef-rp/intro)
> [下一个教程](xref:data/ef-rp/sort-filter-page)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

本教程将介绍和自定义已搭建基架的 CRUD （创建、读取、更新、删除）代码。

## <a name="no-repository"></a>无存储库

某些开发人员使用服务层或存储库模式在 UI (Razor Pages) 和数据访问层之间创建抽象层。 本教程不会这样做。 为最大程度降低复杂性并让本教程重点介绍 EF Core，将直接在页面模型类中添加 EF Core 代码。 

## <a name="update-the-details-page"></a>更新“详细信息”页

“学生”页的基架代码不包括注册数据。 本部分将向“详细信息”页添加注册。

### <a name="read-enrollments"></a>读取注册

若要在页面上显示学生的注册数据，需要读取注册数据。 Pages/Students/Details.cshtml.cs 中的基架代码仅读取学生数据，但不读取注册数据：

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Pages/Students/Details1.cshtml.cs?name=snippet_OnGetAsync&highlight=8)]

使用以下代码替换 `OnGetAsync` 方法以读取所选学生的注册数据。 突出显示所作更改。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Details.cshtml.cs?name=snippet_OnGetAsync&highlight=8-12)]

[Include](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.include) 和 [ThenInclude](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.theninclude#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_ThenInclude__3_Microsoft_EntityFrameworkCore_Query_IIncludableQueryable___0_System_Collections_Generic_IEnumerable___1___System_Linq_Expressions_Expression_System_Func___1___2___) 方法使上下文加载 `Student.Enrollments` 导航属性，并在每个注册中加载 `Enrollment.Course` 导航属性。 这些方法将在[与数据读取相关](xref:data/ef-rp/read-related-data)的教程中进行详细介绍。

对于返回的实体未在当前上下文中更新的情况，[AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) 方法将会提升性能。 `AsNoTracking` 将在本教程的后续部分中讨论。

### <a name="display-enrollments"></a>显示注册

使用以下代码替换 *Pages/Students/Details.cshtml* 中的代码以显示注册列表。 突出显示所作更改。

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Details.cshtml?highlight=32-53)]

上面的代码循环通过 `Enrollments` 导航属性中的实体。 它将针对每个注册显示课程标题和成绩。 课程标题从 Course 实体中检索，该实体存储在 Enrollments 实体的 `Course` 导航属性中。

运行应用，选择“学生”选项卡，然后单击学生的“详细信息”链接 。 随即显示出所选学生的课程和成绩列表。

### <a name="ways-to-read-one-entity"></a>读取一个实体的方法

生成的代码使用 [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Threading_CancellationToken_) 读取一个实体。 如果未找到任何内容，则此方法返回 NULL；否则，它将返回满足查询筛选条件的第一行。 `FirstOrDefaultAsync` 通常是比以下备选方案更好的选择：

* [SingleOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_) - 如果有多个满足查询筛选器的实体，则引发异常。 若要确定查询是否可以返回多行，`SingleOrDefaultAsync` 会尝试提取多个行。 如果查询只能返回一个实体，就像它在搜索唯一键时一样，那么该额外工作是不必要的。
* [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.findasync#Microsoft_EntityFrameworkCore_DbContext_FindAsync_System_Type_System_Object___) - 查找具有主键 ( PK) 的实体。 如果具有 PK 的实体正在由上下文跟踪，会返回该实体且不向数据库发出请求。 此方法经过优化，可查找单个实体，但无法通过 `FindAsync` 调用 `Include`。  如果需要相关数据，`FirstOrDefaultAsync` 则是更好的选择。

### <a name="route-data-vs-query-string"></a>路由数据与查询字符串

“详细信息”页的 URL 是 `https://localhost:<port>/Students/Details?id=1`。 实体的主键值在查询字符串中。 某些开发人员偏向于在路由数据中传递键值：`https://localhost:<port>/Students/Details/1`。 有关详细信息，请参阅[更新生成的代码](xref:tutorials/razor-pages/da1#update-the-generated-code)。

## <a name="update-the-create-page"></a>更新“创建”页

“创建”页面的基架 `OnPostAsync` 代码容易受到[过多发布攻击](#overposting)。 使用以下代码替换 Pages/Students/Create.cshtml.cs 中的 `OnPostAsync` 方法。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Create.cshtml.cs?name=snippet_OnPostAsync)]

<a name="TryUpdateModelAsync"></a>

### <a name="tryupdatemodelasync"></a>TryUpdateModelAsync

前面的代码将创建一个 Student 对象，然后使用发布的表单域更新 Student 对象的属性。 [TryUpdateModelAsync](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.tryupdatemodelasync#Microsoft_AspNetCore_Mvc_ControllerBase_TryUpdateModelAsync_System_Object_System_Type_System_String_) 方法：

* 使用 [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel) 中 [PageContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.pagecontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_PageContext) 属性的已发布的表单值。
* 仅更新列出的属性 (`s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate`)。
* 查找带有“student”前缀的表单值。 例如 `Student.FirstMidName`。 该自变量不区分大小写。
* 使用[模型绑定](xref:mvc/models/model-binding)系统将字符串中的表单值转换为 `Student` 模型中的类型。 例如，`EnrollmentDate` 必须转换为 DateTime。

运行应用，并创建一个学生实体以测试“创建”页。

## <a name="overposting"></a>过多发布

使用 `TryUpdateModel` 更新具有已发布值的字段是一种最佳的安全做法，因为这能阻止过多发布。 例如，假设 Student 实体包含此网页不应更新或添加的 `Secret` 属性：

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Models/StudentZsecret.cs?name=snippet_Intro&highlight=7)]

即使应用的创建或更新 Razor 页面上没有 `Secret` 字段，黑客仍可利用过多发布设置 `Secret` 值。 黑客也可使用 Fiddler 等工具或通过编写某个 JavaScript 来发布 `Secret` 表单值。 原始代码不会限制模型绑定器在创建“学生”实例时使用的字段。

黑客为 `Secret` 表单域指定的任何值都会在数据库中更新。 下图显示 Fiddler 工具正在将 `Secret` 字段（值为“OverPost”）添加到已发布的表单值。

![Fiddler 添加 Secret 字段](../ef-mvc/crud/_static/fiddler.png)

值“OverPost”已成功添加到所插入行的 `Secret` 属性中。 即使应用设计器从未打算用“创建”页设置 `Secret` 属性，也会发生这种情况。

### <a name="view-model"></a>视图模型

视图模型还提供了一种防止过度发布的方法。

应用程序模型通常称为域模型。 域模型通常包含数据库中对应实体所需的全部属性。 视图模型只包含它所用于的 UI（例如，“创建”页）所需的属性。

除视图模型外，某些应用使用绑定模型或输入模型在 Razor Pages 页面模型类和浏览器之间传递数据。 

请考虑以下 `Student` 视图模型：

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Models/StudentVM.cs)]

以下代码使用 `StudentVM` 视图模型创建新的学生：

[!code-csharp[Main](intro/samples/cu30snapshots/2-crud/Pages/Students/CreateVM.cshtml.cs?name=snippet_OnPostAsync)]

[SetValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues.setvalues#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyValues_SetValues_System_Object_) 方法通过从另一个 [PropertyValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues) 对象读取值来设置此对象的值。 `SetValues` 使用属性名称匹配。 视图模型类型不需要与模型类型相关，它只需要具有匹配的属性。

使用 `StudentVM` 时需要更新 [Create.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu30snapshots/2-crud/Pages/Students/CreateVM.cshtml) 才能使用 `StudentVM` 而非 `Student`。

## <a name="update-the-edit-page"></a>更新“编辑”页

在 Pages/Students/Edit.cshtml.cs 中，使用以下代码替换 `OnGetAsync` 和 `OnPostAsync` 方法。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Edit.cshtml.cs?name=snippet_OnGetPost)]

代码更改与“创建”页类似，但有少数例外：

* 已将 `FirstOrDefaultAsync` 替换为 [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbset-1.findasync)。 如果不需要包含的相关数据，`FindAsync` 更高效。
* `OnPostAsync` 具有 `id` 参数。
* 当前学生是从数据库中提取的，而非通过创建空学生获得。

运行应用，并通过创建和编辑学生进行测试。

## <a name="entity-states"></a>实体状态

数据库上下文会随时跟踪内存中的实体是否已与其在数据库中的对应行进行同步。 此跟踪信息可确定调用 [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_) 后的行为。 例如，将新实体传递到 [AddAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.addasync) 方法时，该实体的状态设置为 [Added](/dotnet/api/microsoft.entityframeworkcore.entitystate#Microsoft_EntityFrameworkCore_EntityState_Added)。 调用 `SaveChangesAsync` 时，数据库上下文会发出 SQL INSERT 命令。

实体可能处于[以下状态](/dotnet/api/microsoft.entityframeworkcore.entitystate)之一：

* `Added`：数据库中尚不存在实体。 `SaveChanges` 方法发出 INSERT 语句。

* `Unchanged`：无需保存对该实体所做的任何更改。 从数据库中读取实体时，该实体具有此状态。

* `Modified`：已修改实体的部分或全部属性值。 `SaveChanges` 方法发出 UPDATE 语句。

* `Deleted`：已标记该实体进行删除。 `SaveChanges` 方法发出 DELETE 语句。

* `Detached`：数据库上下文未跟踪该实体。

在桌面应用中，通常会自动设置状态更改。 读取实体并执行更改后，实体状态自动更改为 `Modified`。 调用 `SaveChanges` 会生成仅更新已更改属性的 SQL UPDATE 语句。

在 Web 应用中，读取实体并显示数据的 `DbContext` 将在页面呈现后进行处理。 调用页面 `OnPostAsync` 方法时，将发出具有 `DbContext` 的新实例的 Web 请求。 如果在这个新的上下文中重新读取实体，则会模拟桌面处理。

## <a name="update-the-delete-page"></a>更新“删除”页

在此部分中，当对 `SaveChanges` 的调用失败时，将实现自定义错误消息。

使用以下代码替换 *Pages/Students/Delete.cshtml.cs* 中的代码。 更改将突出显示（而不是清除 `using` 语句）。

[!code-csharp[Main](intro/samples/cu30/Pages/Students/Delete.cshtml.cs?name=snippet_All&highlight=20,22,30,38-41,53-71)]

前面的代码将可选参数 `saveChangesError` 添加到 `OnGetAsync` 方法签名中。 `saveChangesError` 指示学生对象删除失败后是否调用该方法。 删除操作可能由于暂时性网络问题而失败。 数据库在云中时，更可能出现暂时性网络错误。 通过 UI 调用“删除”页 `OnGetAsync` 时，`saveChangesError` 参数为 false。 当 `OnPostAsync` 调用 `OnGetAsync`（由于删除操作失败）时，`saveChangesError` 参数为 true。

`OnPostAsync` 方法检索所选实体，然后调用 [Remove](/dotnet/api/microsoft.entityframeworkcore.dbcontext.remove#Microsoft_EntityFrameworkCore_DbContext_Remove_System_Object_) 方法将实体的状态设置为 `Deleted`。 调用 `SaveChanges` 时生成 SQL DELETE 命令。 如果 `Remove` 失败：

* 捕获数据库异常。
* 通过 `saveChangesError=true` 调用“删除”页 `OnGetAsync` 方法。

向“删除”Razor 页面添加错误消息 (Pages/Students/Delete.cshtml)：

[!code-cshtml[Main](intro/samples/cu30/Pages/Students/Delete.cshtml?highlight=10)]

运行应用并删除学生以测试“删除”页。

## <a name="next-steps"></a>后续步骤

> [!div class="step-by-step"]
> [上一个教程](xref:data/ef-rp/intro)
> [下一个教程](xref:data/ef-rp/sort-filter-page)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本教程将介绍和自定义已搭建基架的 CRUD （创建、读取、更新、删除）代码。

为最大程度降低复杂性并让这些教程集中介绍 EF Core，将在页面模型中使用 EF Core 代码。 某些开发人员使用服务层或存储库模式在 UI (Razor Pages) 和数据访问层之间创建抽象层。

本教程将检查 Students 文件夹中的“创建”、“编辑”、“删除”和“详细信息”Razor 页面。

基架代码将以下模式用于“创建”、“编辑”和“删除”页面：

* 使用 HTTP GET 方法 `OnGetAsync` 获取和显示请求数据。
* 使用 HTTP POST 方法 `OnPostAsync` 将更改保存到数据。

“索引”和“详细信息”页面使用 HTTP GET 方法 `OnGetAsync` 获取和显示请求数据

## <a name="singleordefaultasync-vs-firstordefaultasync"></a>SingleOrDefaultAsync 与FirstOrDefaultAsync

生成的代码使用 [FirstOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstOrDefaultAsync__1_System_Linq_IQueryable___0__System_Threading_CancellationToken_)其推荐度通常高于 [SingleOrDefaultAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleordefaultasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleOrDefaultAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_)。

 提取一个实体时，使用 `FirstOrDefaultAsync` 比使用 `SingleOrDefaultAsync` 更高效：

* 代码需要验证查询仅返回一个实体时除外。
* `SingleOrDefaultAsync` 会提取更多数据并执行不必要的工作。
* 如果有多个实体符合筛选部分，`SingleOrDefaultAsync` 将引发异常。
* 如果有多个实体符合筛选部分，`FirstOrDefaultAsync` 不引发异常。

<a name="FindAsync"></a>

### <a name="findasync"></a>FindAsync

在大部分基架代码中，[FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.findasync#Microsoft_EntityFrameworkCore_DbContext_FindAsync_System_Type_System_Object___) 可用于替代 `FirstOrDefaultAsync`。

`FindAsync`:

* 查找具有主键 (PK) 的实体。 如果具有 PK 的实体正在由上下文跟踪，会返回该实体且不向 DB 发出请求。
* 既简单又简洁。
* 经过优化后可查找单个实体。
* 在某些情况下可以提供性能优势，但很少发生在典型的 Web 应用中。
* 以隐式方式使用 [FirstAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.firstasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_FirstAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_) 而不是 [SingleAsync](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.singleasync#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_SingleAsync__1_System_Linq_IQueryable___0__System_Linq_Expressions_Expression_System_Func___0_System_Boolean___System_Threading_CancellationToken_)。

如果想要 `Include` 其他实体，则 `FindAsync` 将不再适用。 这意味着可能需要放弃 `FindAsync` 并随着应用运行移动到查询。

## <a name="customize-the-details-page"></a>自定义“详细信息”页

浏览到 `Pages/Students` 页面。 “编辑”、“详细信息”和“删除”链接是在 Pages/Students/Index.cshtml 文件中由[定位点标记帮助器](xref:mvc/views/tag-helpers/builtin-th/anchor-tag-helper)生成的  。

[!code-cshtml[](intro/samples/cu21/Pages/Students/Index1.cshtml?name=snippet)]

运行应用并选择“详细信息”链接。 URL 的格式为 `http://localhost:5000/Students/Details?id=2`。 “学生 ID”通过查询字符串 (`?id=2`) 进行传递。

更新“编辑”、“详细信息”和“删除”Razor 页面以使用 `"{id:int}"` 路由模板。 将上述每个页面的页面指令从 `@page` 更改为 `@page "{id:int}"`。

如果对具有不包含整数路由值的“{id:int}”路由模板的页面发起请求，则该请求将返回 HTTP 404（找不到）错误。 例如，`http://localhost:5000/Students/Details` 返回 404 错误。 若要使 ID 可选，请将 `?` 追加到路由约束：

 ```cshtml
@page "{id:int?}"
```

运行应用，单击“详细信息”链接，并验证确认 URL 正在将 ID 作为路由数据 (`http://localhost:5000/Students/Details/2`) 进行传递。

不要将 `@page` 全局更改为 `@page "{id:int}"`，这样做会破坏指向“主页”和“创建”页面的链接。

<!-- See https://github.com/aspnet/Scaffolding/issues/590 -->

### <a name="add-related-data"></a>添加相关数据

“学生索引”页的基架代码不包括 `Enrollments` 属性。 在本部分，`Enrollments` 集合的内容显示在“详细信息”页中。

Pages/Students/Details.cshtml.cs 的 `OnGetAsync` 方法使用 `FirstOrDefaultAsync` 方法检索单个 `Student` 实体。 添加以下突出显示的代码：

[!code-csharp[](intro/samples/cu21/Pages/Students/Details.cshtml.cs?name=snippet_Details&highlight=8-12)]

[Include](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.include) 和 [ThenInclude](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.theninclude#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_ThenInclude__3_Microsoft_EntityFrameworkCore_Query_IIncludableQueryable___0_System_Collections_Generic_IEnumerable___1___System_Linq_Expressions_Expression_System_Func___1___2___) 方法使上下文加载 `Student.Enrollments` 导航属性，并在每个注册中加载 `Enrollment.Course` 导航属性。 这些方法将在与数据读取相关的教程中进行详细介绍。

对于返回的实体未在当前上下文中更新的情况，[AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) 方法将会提升性能。 `AsNoTracking` 将在本教程的后续部分中讨论。

### <a name="display-related-enrollments-on-the-details-page"></a>在“详细信息”页中显示相关注册

打开 Pages/Students/Details.cshtml。 添加以下突出显示的代码以显示注册列表：

[!code-cshtml[](intro/samples/cu21/Pages/Students/Details.cshtml?highlight=32-53)]

如果代码缩进在粘贴代码后出现错误，请按 CTRL-K-D 进行更正。

上面的代码循环通过 `Enrollments` 导航属性中的实体。 它将针对每个注册显示课程标题和成绩。 课程标题从 Course 实体中检索，该实体存储在 Enrollments 实体的 `Course` 导航属性中。

运行应用，选择“学生”选项卡，然后单击学生的“详细信息”链接 。 随即显示出所选学生的课程和成绩列表。

## <a name="update-the-create-page"></a>更新“创建”页

将 Pages/Students/Create.cshtml.cs 中的 `OnPostAsync` 方法更新为以下代码：

[!code-csharp[](intro/samples/cu21/Pages/Students/Create.cshtml.cs?name=snippet_OnPostAsync)]

<a name="TryUpdateModelAsync"></a>

### <a name="tryupdatemodelasync"></a>TryUpdateModelAsync

检查 [TryUpdateModelAsync](/dotnet/api/microsoft.aspnetcore.mvc.controllerbase.tryupdatemodelasync#Microsoft_AspNetCore_Mvc_ControllerBase_TryUpdateModelAsync_System_Object_System_Type_System_String_) 代码：

[!code-csharp[](intro/samples/cu21/Pages/Students/Create.cshtml.cs?name=snippet_TryUpdateModelAsync)]

在前面的代码中，`TryUpdateModelAsync<Student>` 尝试使用 [PageModel](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel) 的 [PageContext](/dotnet/api/microsoft.aspnetcore.mvc.razorpages.pagemodel.pagecontext#Microsoft_AspNetCore_Mvc_RazorPages_PageModel_PageContext) 属性中已发布的表单值更新 `emptyStudent` 对象。 `TryUpdateModelAsync` 仅更新列出的属性 (`s => s.FirstMidName, s => s.LastName, s => s.EnrollmentDate`)。

在上述示例中：

* 第二个自变量 (`"student", // Prefix`) 是用于查找值的前缀。 该自变量不区分大小写。
* 已发布的表单值通过[模型绑定](xref:mvc/models/model-binding)转换为 `Student` 模型中的类型。

<a id="overpost"></a>

### <a name="overposting"></a>过多发布

使用 `TryUpdateModel` 更新具有已发布值的字段是一种最佳的安全做法，因为这能阻止过多发布。 例如，假设 Student 实体包含此网页不应更新或添加的 `Secret` 属性：

[!code-csharp[](intro/samples/cu21/Models/StudentZsecret.cs?name=snippet_Intro&highlight=7)]

即使应用的创建或更新 Razor 页面上没有 `Secret` 字段，黑客仍可利用过多发布设置 `Secret` 值。 黑客也可使用 Fiddler 等工具或通过编写某个 JavaScript 来发布 `Secret` 表单值。 原始代码不会限制模型绑定器在创建“学生”实例时使用的字段。

黑客为 `Secret` 表单字段指定的任何值都会在 DB 中更新。 下图显示 Fiddler 工具正在将 `Secret` 字段（值为“OverPost”）添加到已发布的表单值。

![Fiddler 添加 Secret 字段](../ef-mvc/crud/_static/fiddler.png)

值“OverPost”已成功添加到所插入行的 `Secret` 属性中。 应用程序设计器绝不会在“创建”页设置 `Secret` 属性。

<a name="vm"></a>

### <a name="view-model"></a>视图模型

视图模型通常包含应用程序所用的模型中包括的属性的子集。 应用程序模型通常称为域模型。 域模型通常包含 DB 中对应实体所需的全部属性。 视图模型仅包含 UI 层（例如“创建”页）所需的属性。 除视图模型外，某些应用使用绑定模型或输入模型在 Razor Pages 页面模型类和浏览器之间传递数据。 请考虑以下 `Student` 视图模型：

[!code-csharp[](intro/samples/cu21/Models/StudentVM.cs)]

视图模型还提供了一种防止过度发布的方法。 视图模型仅包含要查看（显示）或更新的属性。

以下代码使用 `StudentVM` 视图模型创建新的学生：

[!code-csharp[](intro/samples/cu21/Pages/Students/CreateVM.cshtml.cs?name=snippet_OnPostAsync)]

[SetValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues.setvalues#Microsoft_EntityFrameworkCore_ChangeTracking_PropertyValues_SetValues_System_Object_) 方法通过从另一个 [PropertyValues](/dotnet/api/microsoft.entityframeworkcore.changetracking.propertyvalues) 对象读取值来设置此对象的值。 `SetValues` 使用属性名称匹配。 视图模型类型不需要与模型类型相关，它只需要具有匹配的属性。

使用 `StudentVM` 时需要更新 [CreateVM.cshtml](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples/cu21/Pages/Students/CreateVM.cshtml) 才能使用 `StudentVM` 而非 `Student`。

在 Razor Pages 中，`PageModel` 派生类就是视图模型。

## <a name="update-the-edit-page"></a>更新“编辑”页

更新“编辑”页的页面模型。 突出显示所作的主要更改：

[!code-csharp[](intro/samples/cu21/Pages/Students/Edit.cshtml.cs?name=snippet_OnPostAsync&highlight=20,36)]

代码更改与“创建”页类似，但有少数例外：

* `OnPostAsync` 具有可选的 `id` 参数。
* 当前学生是从 DB 提取的，而非通过创建空学生获得。
* 已将 `FirstOrDefaultAsync` 替换为 [FindAsync](/dotnet/api/microsoft.entityframeworkcore.dbset-1.findasync)。 从主键中选择实体时，使用 `FindAsync` 是一个不错的选择。 请参阅 [FindAsync](#FindAsync) 了解详细信息。

### <a name="test-the-edit-and-create-pages"></a>测试“编辑”和“创建”页

创建和编辑几个学生实体。

## <a name="entity-states"></a>实体状态

DB 上下文会随时跟踪内存中的实体是否已与其在 DB 中的对应行进行同步。 DB 上下文同步信息可决定调用 [SaveChangesAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.savechangesasync#Microsoft_EntityFrameworkCore_DbContext_SaveChangesAsync_System_Threading_CancellationToken_) 后的行为。 例如，将新实体传递到 [AddAsync](/dotnet/api/microsoft.entityframeworkcore.dbcontext.addasync) 方法时，该实体的状态设置为 [Added](/dotnet/api/microsoft.entityframeworkcore.entitystate#Microsoft_EntityFrameworkCore_EntityState_Added)。 调用 `SaveChangesAsync` 时，DB 上下文会发出 SQL INSERT 命令。

实体可能处于[以下状态](/dotnet/api/microsoft.entityframeworkcore.entitystate)之一：

* `Added`：DB 中尚不存在实体。 `SaveChanges` 方法发出 INSERT 语句。

* `Unchanged`：无需保存对该实体所做的任何更改。 从 DB 中读取实体时，该实体将具有此状态。

* `Modified`：已修改实体的部分或全部属性值。 `SaveChanges` 方法发出 UPDATE 语句。

* `Deleted`：已标记该实体进行删除。 `SaveChanges` 方法发出 DELETE 语句。

* `Detached`：DB 上下文未跟踪该实体。

在桌面应用中，通常会自动设置状态更改。 读取实体并执行更改后，实体状态自动更改为 `Modified`。 调用 `SaveChanges` 会生成仅更新已更改属性的 SQL UPDATE 语句。

在 Web 应用中，读取实体并显示数据的 `DbContext` 将在页面呈现后进行处理。 调用页面 `OnPostAsync` 方法时，将发出具有 `DbContext` 的新实例的 Web 请求。 如果在这个新的上下文中重新读取实体，则会模拟桌面处理。

## <a name="update-the-delete-page"></a>更新“删除”页

在此部分中，当对 `SaveChanges` 的调用失败时，将添加用于实现自定义错误消息的代码。 添加字符串，使其包含可能的错误消息：

[!code-csharp[](intro/samples/cu21/Pages/Students/Delete.cshtml.cs?name=snippet1&highlight=12)]

将 `OnGetAsync` 方法替换为以下代码：

[!code-csharp[](intro/samples/cu21/Pages/Students/Delete.cshtml.cs?name=snippet_OnGetAsync&highlight=1,9,17-20)]

上述代码包含可选参数 `saveChangesError`。 `saveChangesError` 指示学生对象删除失败后是否调用该方法。 删除操作可能由于暂时性网络问题而失败。 云端更可能出现暂时性网络错误。 通过 UI 调用“删除”页 `OnGetAsync` 时，`saveChangesError` 为 false。 当 `OnPostAsync` 调用 `OnGetAsync`（由于删除操作失败）时，`saveChangesError` 参数为 true。

### <a name="the-delete-pages-onpostasync-method"></a>“删除”页 OnPostAsync 方法

将 `OnPostAsync` 替换为以下代码：

[!code-csharp[](intro/samples/cu21/Pages/Students/Delete.cshtml.cs?name=snippet_OnPostAsync)]

上述代码检索所选的实体，然后调用 [Remove](/dotnet/api/microsoft.entityframeworkcore.dbcontext.remove#Microsoft_EntityFrameworkCore_DbContext_Remove_System_Object_) 方法，将实体的状态设置为 `Deleted`。 调用 `SaveChanges` 时生成 SQL DELETE 命令。 如果 `Remove` 失败：

* 会捕获 DB 异常。
* 通过 `saveChangesError=true` 调用“删除”页 `OnGetAsync` 方法。

### <a name="update-the-delete-no-locrazor-page"></a>更新“删除”Razor 页面

将以下突出显示的错误消息添加到“删除”Razor 页面。
<!--
[!code-cshtml[](intro/samples/cu21/Pages/Students/Delete.cshtml?name=snippet&highlight=11)]
-->
[!code-cshtml[](intro/samples/cu21/Pages/Students/Delete.cshtml?range=1-13&highlight=10)]

测试“删除”。

## <a name="common-errors"></a>常见错误

“学生/索引”或其他链接不起作用：

验证确认 Razor 页面包含正确的 `@page` 指令。 例如，“学生”/“索引”Razor 页面不得包含路由模板：

```cshtml
@page "{id:int}"
```

每个 Razor 页面均必须包含 `@page` 指令。



## <a name="additional-resources"></a>其他资源

* [本教程的 YouTube 版本](https://www.youtube.com/watch?v=K4X1MT2jt6o)

> [!div class="step-by-step"]
> [上一页](xref:data/ef-rp/intro)
> [下一页](xref:data/ef-rp/sort-filter-page)

::: moniker-end
