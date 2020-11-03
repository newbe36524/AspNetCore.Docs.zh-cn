---
title: 第 7 部分，ASP.NET Core 中的 Razor 页面和 EF Core - 更新相关数据
author: rick-anderson
description: Razor 页面和实体框架教程系列的第 7 部分。
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
uid: data/ef-rp/update-related-data
ms.openlocfilehash: 17b200f0ba90035c417c96689798263af16551de
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722814"
---
# <a name="part-7-no-locrazor-pages-with-ef-core-in-aspnet-core---update-related-data"></a>第 7 部分，ASP.NET Core 中的 Razor 页面和 EF Core - 更新相关数据

作者：[Tom Dykstra](https://github.com/tdykstra) 和 [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE [about the series](../../includes/RP-EF/intro.md)]

::: moniker range=">= aspnetcore-3.0"

本教程将介绍如何更新相关数据。 下图显示了部分已完成页面。

![课程“编辑”页](update-related-data/_static/course-edit30.png)
![讲师”编辑”页](update-related-data/_static/instructor-edit-courses30.png)

## <a name="update-the-course-create-and-edit-pages"></a>更新课程“创建”和“编辑”页

课程“创建”和“编辑”页的基架搭建代码具有显示院系 ID（整数）的“院系”下拉列表。 下拉列表应显示院系名称，因此这两个页面都需要院系名称列表。 若要提供该列表，请使用“创建”和“编辑”页的基类。

### <a name="create-a-base-class-for-course-create-and-edit"></a>创建课程“创建”和“编辑”的基类

使用以下代码创建 Pages/Courses/DepartmentNamePageModel.cs 文件：

[!code-csharp[](intro/samples/cu30/Pages/Courses/DepartmentNamePageModel.cs)]

上面的代码创建 [SelectList](/dotnet/api/microsoft.aspnetcore.mvc.rendering.selectlist) 以包含系名称列表。 如果指定了 `selectedDepartment`，可在 `SelectList` 中选择该系。

“创建”和“编辑”页模型类将派生自 `DepartmentNamePageModel`。

### <a name="update-the-course-create-page-model"></a>更新课程“创建”页模型

课程分配给院系。 “创建”和“编辑”页的基类提供 `SelectList`，用于选择院系。 采用 `SelectList` 的下拉列表设置 `Course.DepartmentID` 外键 (FK) 属性。 EF Core 使用 `Course.DepartmentID` FK 加载 `Department` 导航属性。

![创建课程](update-related-data/_static/ddl30.png)

使用以下代码更新 Pages/Courses/Create.cshtml.cs：

[!code-csharp[](intro/samples/cu30/Pages/Courses/Create.cshtml.cs?highlight=7,18,27-41)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

前面的代码：

* 派生自 `DepartmentNamePageModel`。
* 使用 `TryUpdateModelAsync` 防止[过多发布](xref:data/ef-rp/crud#overposting)。
* 删除 `ViewData["DepartmentID"]`。 基类中的 `DepartmentNameSL` 是强类型模型，将用于 Razor 页面。 建议使用强类型而非弱类型。 有关详细信息，请参阅[弱类型数据（ViewData 和 ViewBag）](xref:mvc/views/overview#VD_VB)。

### <a name="update-the-course-create-no-locrazor-page"></a>更新“课程创建”Razor 页面

使用以下代码更新 Pages/Courses/Create.cshtml：

[!code-cshtml[](intro/samples/cu30/Pages/Courses/Create.cshtml?highlight=29-34)]

上面的代码执行以下更改：

* 将标题从“DepartmentID”更改为“Department” 。
* 将 `"ViewBag.DepartmentID"` 替换为 `DepartmentNameSL`（来自基类）。
* 添加“选择系”选项。 如果尚未选择院系（而不是已选中首个院系），此更改将在下拉列表中显示“选择院系”。
* 在未选择系时添加验证消息。

Razor 页面使用[选择标记帮助程序](xref:mvc/views/working-with-forms#the-select-tag-helper)：

[!code-cshtml[](intro/samples/cu/Pages/Courses/Create.cshtml?range=28-35&highlight=3-6)]

测试“创建”页。 “创建”页显示系名称，而不是系 ID。

### <a name="update-the-course-edit-page-model"></a>更新课程“编辑”页模型

使用以下代码更新 Pages/Courses/Edit.cshtml.cs：

[!code-csharp[](intro/samples/cu30/Pages/Courses/Edit.cshtml.cs?highlight=8,28,35,36,40-66)]

这些更改与在“创建”页模型中所做的更改相似。 在上面的代码中，`PopulateDepartmentsDropDownList` 在院系 ID 中传递并将在下拉列表中选择该院系。

### <a name="update-the-course-edit-no-locrazor-page"></a>更新“课程编辑”Razor 页面

使用以下代码更新 Pages/Courses/Edit.cshtml：

[!code-cshtml[](intro/samples/cu30/Pages/Courses/Edit.cshtml?highlight=17-20,32-35)]

上面的代码执行以下更改：

* 显示课程 ID。 通常不显示实体的主键 (PK)。 PK 对用户不具有任何意义。 在这种情况下，PK 就是课程编号。
* 将“院系”下拉列表的标题从“DepartmentID”更改为“Department” 。
* 将 `"ViewBag.DepartmentID"` 替换为 `DepartmentNameSL`（来自基类）。

该页面包含课程编号的隐藏域 (`<input type="hidden">`)。 添加具有 `asp-for="Course.CourseID"` 的 `<label>` 标记帮助器也同样需要隐藏域。 用户单击“保存”时，需要 `<input type="hidden">`，以便在已发布的数据中包括课程编号。

## <a name="update-the-course-details-and-delete-pages"></a>更新课程“详细信息”和“删除”页

[AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) 可以在不需要跟踪时提高性能。

### <a name="update-the-course-page-models"></a>更新“课程”页模型

使用以下代码更新 Pages/Courses/Delete.cshtml.cs 以添加 `AsNoTracking`：

[!code-csharp[](intro/samples/cu30/Pages/Courses/Delete.cshtml.cs?highlight=29)]

在 Pages/Courses/Details.cshtml.cs 文件中进行相同的更改：

[!code-csharp[](intro/samples/cu30/Pages/Courses/Details.cshtml.cs?highlight=28)]

### <a name="update-the-course-no-locrazor-pages"></a>更新“课程”Razor 页面

使用以下代码更新 Pages/Courses/Delete.cshtml：

[!code-cshtml[](intro/samples/cu30/Pages/Courses/Delete.cshtml?highlight=15-20,37)]

对“详细信息”页执行相同更改。

[!code-cshtml[](intro/samples/cu30/Pages/Courses/Details.cshtml?highlight=14-19,36)]

## <a name="test-the-course-pages"></a>测试“课程”页

测试“创建”、“编辑”、“详细信息”和“删除”页面。

## <a name="update-the-instructor-create-and-edit-pages"></a>更新讲师“创建”和“编辑”页

讲师可能教授任意数量的课程。 下图显示包含一系列课程复选框的讲师“编辑”页。

![带课程信息的讲师“编辑”页](update-related-data/_static/instructor-edit-courses30.png)

通过复选框可对分配给讲师的课程进行更改。 数据库中的每一门课程均有对应显示的复选框。 已分配给讲师的课程将会被选中。 用户可以通过选择或清除复选框来更改课程分配。 如果课程数过多，另一个 UI 的使用效果可能更好。 但此处所示的用于管理多对多关系的方法不会发生变化。 若要创建或删除关系，则需要使用联接实体。

### <a name="create-a-class-for-assigned-courses-data"></a>为已分配的课程数据创建类

使用以下代码创建 SchoolViewModels/AssignedCourseData.cs：

[!code-csharp[](intro/samples/cu30/Models/SchoolViewModels/AssignedCourseData.cs)]

`AssignedCourseData` 类包含的数据可用于为已分配给讲师的课程创建复选框。

### <a name="create-an-instructor-page-model-base-class"></a>创建“讲师”页模型基类

创建 Pages/Instructors/InstructorCoursesPageModel.cs 基类：

[!code-csharp[](intro/samples/cu30/Pages/Instructors/InstructorCoursesPageModel.cs?name=snippet_All)]

`InstructorCoursesPageModel` 是将用于“编辑”和“创建”页模型的基类。 `PopulateAssignedCourseData` 读取所有 `Course` 实体以填充 `AssignedCourseDataList`。 该代码将设置每门课程的 `CourseID` 和标题，并决定是否为讲师分配该课程。 [HashSet](/dotnet/api/system.collections.generic.hashset-1) 用于高效查找。

Razor 页面没有 Course 实体的集合，因此模型绑定器无法自动更新 `CourseAssignments` 导航属性。 可在新的 `UpdateInstructorCourses` 方法中更新 `CourseAssignments` 导航属性，而不必使用模型绑定器。 为此，需要从模型绑定中排除 `CourseAssignments` 属性。 此操作无需对调用 `TryUpdateModel` 的代码进行任何更改，因为使用的重载包含已声明的属性，并且 `CourseAssignments` 不包括在该列表中。

如果未选中任何复选框，则 `UpdateInstructorCourses` 中的代码将使用空集合初始化 `CourseAssignments` 导航属性，并返回以下内容：

[!code-csharp[](intro/samples/cu30/Pages/Instructors/InstructorCoursesPageModel.cs?name=snippet_IfNull)]

之后，代码会循环访问数据库中的所有课程，并逐一检查当前分配给讲师的课程和页面中处于选中状态的课程。 为便于高效查找，后两个集合存储在 `HashSet` 对象中。

如果某课程的复选框处于选中状态，但该课程不在 `Instructor.CourseAssignments` 导航属性中，则会将该课程添加到导航属性中的集合中。

[!code-csharp[](intro/samples/cu30/Pages/Instructors/InstructorCoursesPageModel.cs?name=snippet_UpdateCourses)]

如果某课程的复选框未处于选中状态，但该课程存在 `Instructor.CourseAssignments` 导航属性中，则会从导航属性中删除该课程。

[!code-csharp[](intro/samples/cu30/Pages/Instructors/InstructorCoursesPageModel.cs?name=snippet_UpdateCoursesElse)]

### <a name="handle-office-location"></a>处理办公室位置

“编辑”页必须处理的另一个关系是 Instructor 实体与 `OfficeAssignment` 实体之间的一对零或一对一关系。 讲师编辑代码必须处理以下场景： 

* 如果用户清除办公室分配，则需删除 `OfficeAssignment` 实体。
* 如果用户输入办公室分配，且办公室分配原本为空，则需创建一个新 `OfficeAssignment` 实体。
* 如果用户更改办公室分配，则需更新 `OfficeAssignment` 实体。

### <a name="update-the-instructor-edit-page-model"></a>更新讲师“编辑”页模型

使用以下代码更新 Pages/Instructors/Edit.cshtml.cs：

[!code-csharp[](intro/samples/cu30/Pages/Instructors/Edit.cshtml.cs?name=snippet_All&highlight=9,28-32,38,42-77)]

前面的代码：

* 使用 `OfficeAssignment`、`CourseAssignment` 和 `CourseAssignment.Course` 导航属性的预先加载从数据库获取当前的 `Instructor` 实体。
* 用模型绑定器中的值更新检索到的 `Instructor` 实体。 `TryUpdateModel` 可防止[过多发布](xref:data/ef-rp/crud#overposting)。
* 如果办公室位置为空，则将 `Instructor.OfficeAssignment` 设置为 null。 当 `Instructor.OfficeAssignment` 为 null 时，`OfficeAssignment` 表中的相关行将会删除。
* 调用 `OnGetAsync` 中的 `PopulateAssignedCourseData`，使用 `AssignedCourseData` 视图模型类为复选框提供信息。
* 调用 `OnPostAsync` 中的 `UpdateInstructorCourses`，将复选框中的信息应用于将要编辑的 Instructor 实体。
* 如果 `TryUpdateModel` 失败，则调用 `OnPostAsync` 中的 `PopulateAssignedCourseData` 和 `UpdateInstructorCourses`。 这些方法调用将在页面重新显示错误消息时还原页面上所输入的已分配课程数据。

### <a name="update-the-instructor-edit-no-locrazor-page"></a>更新“讲师编辑”Razor 页面

使用以下代码更新 Pages/Instructors/Edit.cshtml：

[!code-cshtml[](intro/samples/cu30/Pages/Instructors/Edit.cshtml?highlight=29-59)]

上面的代码将创建一个具有三列的 HTML 表。 每列均具有一个复选框和包含课程编号及标题的标题。 所有复选框均具有相同的名称（“selectedCourses”）。 使用相同名称可指示模型绑定器将它们视为一个组。 每个复选框的值特性都设置为 `CourseID`。 发布页面时，模型绑定器会传递一个数组，该数组只包括所选复选框的 `CourseID` 值。

初次呈现复选框时，分配给讲师的课程均已选中。

注意：此处所使用的编辑讲师课程数据的方法适用于数量有限的课程。 对于规模远大于此的集合，则使用不同的 UI 和不同的更新方法会更实用和更高效。

运行应用并测试更新的讲师“编辑”页。 更改某些课程分配。 这些更改将反映在“索引”页上。

### <a name="update-the-instructor-create-page"></a>更新讲师“创建”页

使用类似于“编辑”页面的代码更新“讲师创建”页面模型和 Razor 页面：

[!code-csharp[](intro/samples/cu30/Pages/Instructors/Create.cshtml.cs)]

[!code-cshtml[](intro/samples/cu30/Pages/Instructors/Create.cshtml)]

测试讲师“创建”页。

## <a name="update-the-instructor-delete-page"></a>更新讲师“删除”页

使用以下代码更新 Pages/Instructors/Delete.cshtml.cs：

[!code-csharp[](intro/samples/cu30/Pages/Instructors/Delete.cshtml.cs?highlight=45-61)]

上面的代码执行以下更改：

* 对 `CourseAssignments` 导航属性使用预先加载。 必须包含 `CourseAssignments`，否则删除讲师时将不会删除课程。 为避免阅读它们，可以在数据库中配置级联删除。

* 如果要删除的讲师被指派为任何系的管理员，则需从这些系中删除该讲师分配。

运行应用并测试“删除”页。

## <a name="next-steps"></a>后续步骤

> [!div class="step-by-step"]
> [上一个教程](xref:data/ef-rp/read-related-data)
> [下一个教程](xref:data/ef-rp/concurrency)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本教程演示如何更新相关数据。 如果遇到无法解决的问题，请[下载或查看已完成的应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/data/ef-rp/intro/samples)。 [下载说明](xref:index#how-to-download-a-sample)。

下图显示了部分已完成页面。

![课程“编辑”页](update-related-data/_static/course-edit.png)
![讲师”编辑”页](update-related-data/_static/instructor-edit-courses.png)

查看并测试“创建”和“编辑”课程页。 创建新课程。 系按其主键（一个整数）进行选择，而不是按名称。 编辑新课程。 完成测试后，请删除新课程。

## <a name="create-a-base-class-to-share-common-code"></a>创建基类以共享通用代码

“课程/创建”和“课程/编辑”页分别需要一个系名称列表。 针对“创建”和“编辑”页创建 Pages/Courses/DepartmentNamePageModel.cshtml.cs 基类：

[!code-csharp[](intro/samples/cu/Pages/Courses/DepartmentNamePageModel.cshtml.cs?highlight=9,11,20-21)]

上面的代码创建 [SelectList](/dotnet/api/microsoft.aspnetcore.mvc.rendering.selectlist) 以包含系名称列表。 如果指定了 `selectedDepartment`，可在 `SelectList` 中选择该系。

“创建”和“编辑”页模型类将派生自 `DepartmentNamePageModel`。

## <a name="customize-the-courses-pages"></a>自定义课程页

创建新的课程实体时，新实体必须与现有院系有关系。 若要在创建课程的同时添加系，则“创建”和“编辑”的基类必须包含用于选择系的下拉列表。 该下拉列表设置 `Course.DepartmentID` 外键 (FK) 属性。 EF Core 使用 `Course.DepartmentID` FK 加载 `Department` 导航属性。

![创建课程](update-related-data/_static/ddl.png)

使用以下代码更新“创建”页模型：

[!code-csharp[](intro/samples/cu/Pages/Courses/Create.cshtml.cs?highlight=7,18,32-999)]

前面的代码：

* 派生自 `DepartmentNamePageModel`。
* 使用 `TryUpdateModelAsync` 防止[过多发布](xref:data/ef-rp/crud#overposting)。
* 将 `ViewData["DepartmentID"]` 替换为 `DepartmentNameSL`（来自基类）。

`ViewData["DepartmentID"]` 将替换为强类型 `DepartmentNameSL`。 建议使用强类型而非弱类型。 有关详细信息，请参阅[弱类型数据（ViewData 和 ViewBag）](xref:mvc/views/overview#VD_VB)。

### <a name="update-the-courses-create-page"></a>更新“课程创建”页

使用以下代码更新 Pages/Courses/Create.cshtml：

[!code-cshtml[](intro/samples/cu/Pages/Courses/Create.cshtml?highlight=29-34)]

上述标记进行以下更改：

* 将标题从“DepartmentID”更改为“Department” 。
* 将 `"ViewBag.DepartmentID"` 替换为 `DepartmentNameSL`（来自基类）。
* 添加“选择系”选项。 此更改将导致呈现“选择系”而不是第一个系。
* 在未选择系时添加验证消息。

Razor 页面使用[选择标记帮助程序](xref:mvc/views/working-with-forms#the-select-tag-helper)：

[!code-cshtml[](intro/samples/cu/Pages/Courses/Create.cshtml?range=28-35&highlight=3-6)]

测试“创建”页。 “创建”页显示系名称，而不是系 ID。

### <a name="update-the-courses-edit-page"></a>更新“课程编辑”页。

使用以下代码替换 Pages/Courses/Edit.cshtml.cs 中的代码：

[!code-csharp[](intro/samples/cu/Pages/Courses/Edit.cshtml.cs?highlight=8,28,35,36,40,47-999)]

这些更改与在“创建”页模型中所做的更改相似。 在上面的代码中，`PopulateDepartmentsDropDownList` 在系 ID 中传递并将选择下拉列表中指定的系。

使用以下标记更新 Pages/Courses/Edit.cshtml：

[!code-cshtml[](intro/samples/cu/Pages/Courses/Edit.cshtml?highlight=17-20,32-35)]

上述标记进行以下更改：

* 显示课程 ID。 通常不显示实体的主键 (PK)。 PK 对用户不具有任何意义。 在这种情况下，PK 就是课程编号。
* 将标题从“DepartmentID”更改为“Department” 。
* 将 `"ViewBag.DepartmentID"` 替换为 `DepartmentNameSL`（来自基类）。

该页面包含课程编号的隐藏域 (`<input type="hidden">`)。 添加具有 `asp-for="Course.CourseID"` 的 `<label>` 标记帮助器也同样需要隐藏域。 用户单击“保存”时，需要 `<input type="hidden">`，以便在已发布的数据中包括课程编号。

测试更新的代码。 创建、编辑和删除课程。

## <a name="add-asnotracking-to-the-details-and-delete-page-models"></a>将 AsNoTracking 添加到“详细信息”和“删除”页模型

[AsNoTracking](/dotnet/api/microsoft.entityframeworkcore.entityframeworkqueryableextensions.asnotracking#Microsoft_EntityFrameworkCore_EntityFrameworkQueryableExtensions_AsNoTracking__1_System_Linq_IQueryable___0__) 可以在不需要跟踪时提高性能。 将 `AsNoTracking` 添加到“删除”和“详细信息”页模型。 下面的代码显示更新的“删除”页模型：

[!code-csharp[](intro/samples/cu/Pages/Courses/Delete.cshtml.cs?name=snippet&highlight=21,23,40,41)]

在 Pages/Courses/Details.cshtml.cs 文件中更新 `OnGetAsync` 方法：

[!code-csharp[](intro/samples/cu/Pages/Courses/Details.cshtml.cs?name=snippet)]

### <a name="modify-the-delete-and-details-pages"></a>修改“删除”和“详细信息”页

使用以下标记更新“删除”Razor 页面：

[!code-cshtml[](intro/samples/cu/Pages/Courses/Delete.cshtml?highlight=15-20)]

对“详细信息”页执行相同更改。

### <a name="test-the-course-pages"></a>测试“课程”页

测试“创建”、“编辑”、“详细信息”和“删除”。

## <a name="update-the-instructor-pages"></a>更新“讲师”页

以下各部分介绍更新“讲师”页。

### <a name="add-office-location"></a>添加办公室位置

编辑讲师记录时，可能希望更新讲师的办公室分配。 `Instructor` 实体与 `OfficeAssignment` 实体是一对零或一关系。 讲师代码必须处理：

* 如果用户清除办公室分配，则需删除 `OfficeAssignment` 实体。
* 如果用户输入办公室分配，且办公室分配原本为空，则需创建一个新 `OfficeAssignment` 实体。
* 如果用户更改办公室分配，则需更新 `OfficeAssignment` 实体。

使用以下代码更新讲师“编辑”页模型：

[!code-csharp[](intro/samples/cu/Pages/Instructors/Edit1.cshtml.cs?name=snippet&highlight=20-23,32,39-999)]

前面的代码：

* 使用 `OfficeAssignment` 导航属性的预先加载从数据库获取当前的 `Instructor` 实体。
* 用模型绑定器中的值更新检索到的 `Instructor` 实体。 `TryUpdateModel` 可防止[过多发布](xref:data/ef-rp/crud#overposting)。
* 如果办公室位置为空，则将 `Instructor.OfficeAssignment` 设置为 null。 当 `Instructor.OfficeAssignment` 为 null 时，`OfficeAssignment` 表中的相关行将会删除。

### <a name="update-the-instructor-edit-page"></a>更新讲师“编辑”页

使用办公室位置更新 Pages/Instructors/Edit.cshtml：

[!code-cshtml[](intro/samples/cu/Pages/Instructors/Edit1.cshtml?highlight=29-33)]

验证是否可以更改讲师办公室位置。

## <a name="add-course-assignments-to-the-instructor-edit-page"></a>向“讲师编辑”页添加课程分配

讲师可能教授任意数量的课程。 本部分将添加更改课程分配的功能。 下图显示更新的讲师“编辑”页：

![带课程信息的讲师“编辑”页](update-related-data/_static/instructor-edit-courses.png)

`Course` 和 `Instructor` 具有多对多关系。 若要添加和删除关系，可以从 `CourseAssignments` 联接实体集中添加和删除实体。

通过复选框可对分配给讲师的课程进行更改。 数据库中的每一门课程均有对应显示的复选框。 已分配给讲师的课程将会被勾选。 用户可以通过选择或清除复选框来更改课程分配。 如果课程数量过多：

* 可以使用其他用户界面显示课程。
* 使用联接实体创建或删除关系的方法不会发生更改。

### <a name="add-classes-to-support-create-and-edit-instructor-pages"></a>添加类以支持“创建”和“编辑”讲师页

使用以下代码创建 SchoolViewModels/AssignedCourseData.cs：

[!code-csharp[](intro/samples/cu/Models/SchoolViewModels/AssignedCourseData.cs)]

`AssignedCourseData` 类包含的数据可用于为讲师已分配的课程创建复选框。

创建 Pages/Instructors/InstructorCoursesPageModel.cshtml.cs 基类：

[!code-csharp[](intro/samples/cu/Pages/Instructors/InstructorCoursesPageModel.cshtml.cs)]

`InstructorCoursesPageModel` 是将用于“编辑”和“创建”页模型的基类。 `PopulateAssignedCourseData` 读取所有 `Course` 实体以填充 `AssignedCourseDataList`。 该代码将设置每门课程的 `CourseID` 和标题，并决定是否为讲师分配该课程。 [HashSet](/dotnet/api/system.collections.generic.hashset-1) 用于创建高效查找。

### <a name="instructors-edit-page-model"></a>讲师“编辑”页模型

使用以下代码更新讲师“编辑”页模型：

[!code-csharp[](intro/samples/cu/Pages/Instructors/Edit.cshtml.cs?name=snippet&highlight=1,20-24,30,34,41-999)]

上面的代码处理办公室分配更改。

更新讲师 Razor 视图：

[!code-cshtml[](intro/samples/cu/Pages/Instructors/Edit.cshtml?highlight=34-59)]

<a id="notepad"></a>
> [!NOTE]
> 将代码粘贴到 Visual Studio 中时，换行符会发生更改，其更改方式会导致代码中断。 按 Ctrl+Z 一次可撤消自动格式设置。 按 Ctrl+Z 可以修复换行符，使其看起来如此处所示。 缩进不一定要呈现完美形式，但 `@:</tr><tr>`、`@:<td>`、`@:</td>` 和 `@:</tr>` 行必须各成一行（如下所示）。 选中新的代码块后，按 Tab 三次，使新代码与现有代码对齐。 [通过此链接](https://developercommunity.visualstudio.com/content/problem/147795/razor-editor-malforms-pasted-markup-and-creates-in.html)投票或查看此 bug 的状态。

上面的代码将创建一个具有三列的 HTML 表。 每列均具有一个复选框和包含课程编号及标题的标题。 所有复选框均具有相同的名称（“selectedCourses”）。 使用相同名称可指示模型绑定器将它们视为一个组。 每个复选框的值特性都设置为 `CourseID`。 发布页面时，模型绑定器会传递一个数组，该数组只包括所选复选框的 `CourseID` 值。

初次呈现复选框时，分配给讲师的课程具有已勾选的特性。

运行应用并测试更新的讲师“编辑”页。 更改某些课程分配。 这些更改将反映在“索引”页上。

注意：此处所使用的编辑讲师课程数据的方法适用于数量有限的课程。 对于规模远大于此的集合，则使用不同的 UI 和不同的更新方法会更实用和更高效。

### <a name="update-the-instructors-create-page"></a>更新讲师“创建”页

使用以下代码更新讲师“创建”页模型：

[!code-csharp[](intro/samples/cu/Pages/Instructors/Create.cshtml.cs)]

前面的代码与 Pages/Instructors/Edit.cshtml.cs 代码类似。

使用以下标记更新“讲师创建”Razor 页面：

[!code-cshtml[](intro/samples/cu/Pages/Instructors/Create.cshtml?highlight=32-62)]

测试讲师“创建”页。

## <a name="update-the-delete-page"></a>更新“删除”页

使用以下代码更新“删除”页模型：

[!code-csharp[](intro/samples/cu/Pages/Instructors/Delete.cshtml.cs?highlight=5,40-999)]

上面的代码执行以下更改：

* 对 `CourseAssignments` 导航属性使用预先加载。 必须包含 `CourseAssignments`，否则删除讲师时将不会删除课程。 为避免阅读它们，可以在数据库中配置级联删除。

* 如果要删除的讲师被指派为任何系的管理员，则需从这些系中删除该讲师分配。

## <a name="additional-resources"></a>其他资源

* [本教程的 YouTube 版本（第 1 部分）](https://www.youtube.com/watch?v=Csh6gkmwc9E)
* [本教程的 YouTube 版本（第 2 部分）](https://www.youtube.com/watch?v=mOAankB_Zgc)

> [!div class="step-by-step"]
> [上一页](xref:data/ef-rp/read-related-data)
> [下一页](xref:data/ef-rp/concurrency)

::: moniker-end
