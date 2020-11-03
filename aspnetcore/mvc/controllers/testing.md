---
title: ASP.NET Core 中的测试控制器逻辑
author: ardalis
description: 了解如何使用 Moq 和 xUnit 测试 ASP.NET Core 中的控制器逻辑。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 7/22/2020
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
uid: mvc/controllers/testing
ms.openlocfilehash: 962c1cf0be0f80ecd6c3adda7d22db7f16519a2a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060347"
---
# <a name="unit-test-controller-logic-in-aspnet-core"></a>ASP.NET Core 中的单元测试控制器逻辑

作者：[Steve Smith](https://ardalis.com/)

::: moniker range=">= aspnetcore-3.0"

[单元测试](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)涉及通过基础结构和依赖项单独测试应用的一部分。 单元测试控制器逻辑时，仅测试单个操作的内容，不测试其依赖项或框架自身的行为。

## <a name="unit-testing-controllers"></a>单元测试控制器

将控制器操作的单元测试设置为专注于控制器的行为。 控制器单元测试将避开[筛选器](xref:mvc/controllers/filters)、[路由](xref:fundamentals/routing)或[模型绑定](xref:mvc/models/model-binding)等方案。 涵盖共同响应请求的组件之间的交互的测试由 *集成测试* 处理。 有关集成测试的详细信息，请参阅<xref:test/integration-tests>。

如果编写自定义筛选器和路由，应对其单独进行单元测试，而不是在测试特定控制器操作时进行。

要演示控制器单元测试，请查看以下示例应用中的控制器。 

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/testing/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

主控制器显示集体讨论会话列表并允许使用 POST 请求创建新的集体讨论会话：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs?name=snippet_HomeController&highlight=1,5,10,31-32)]

前面的控制器：

* 遵循[显式依赖关系原则](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)。
* 期望[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 提供 `IBrainstormSessionRepository` 的实例。
* 可以通过使用 mock 对象框架（如 [Moq](https://www.nuget.org/packages/Moq/)）的模拟 `IBrainstormSessionRepository` 服务进行测试。 *模拟对象* 是由一组预先确定的用于测试的属性和方法行为的对象。 有关详细信息，请参阅[集成测试简介](xref:test/integration-tests#introduction-to-integration-tests)。

`HTTP GET Index` 方法没有循环或分支，且仅调用一个方法。 此操作的单元测试：

* 使用 `GetTestSessions` 方法模拟 `IBrainstormSessionRepository` 服务。 `GetTestSessions` 使用日期和会话名称创建两个 mock 集体讨论会话。
* 执行 `Index` 方法。
* 根据该方法返回的结果进行断言：
  * 将返回 <xref:Microsoft.AspNetCore.Mvc.ViewResult>。
  * [ViewDataDictionary.Model](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary.Model*) 是 `StormSessionViewModel`。
  * 有两个集体讨论会话存储在 `ViewDataDictionary.Model` 中。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_Index_ReturnsAViewResult_WithAListOfBrainstormSessions&highlight=14-17)]

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_GetTestSessions)]

主控制器的 `HTTP POST Index` 方法测试验证：

* 当 [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid*) 为 `false` 时，操作方法将返回有相应数据的 *400 错误请求* <xref:Microsoft.AspNetCore.Mvc.ViewResult>。
* 当 `ModelState.IsValid` 为 `true` 时：
  * 将调用存储库上的 `Add` 方法。
  * 将返回有正确参数的 <xref:Microsoft.AspNetCore.Mvc.RedirectToActionResult>。

通过使用 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> 添加错误来测试无效模型状态，如下第一个测试所示：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_ModelState_ValidOrInvalid&highlight=9,16-17,38-41)]

当 [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) 无效时，将返回与 GET 请求相同的 `ViewResult`。 测试不会尝试传入无效模型。 传入无效模型不是有效的方法，因为不会运行模型绑定（尽管[集成测试](xref:test/integration-tests)使用模型绑定）。 在本例中，不测试模型绑定。 这些单元测试仅测试操作方法中的代码。

第二个测试验证 `ModelState` 有效时的情况：

* 已通过存储库添加新的 `BrainstormSession`。
* 该方法将返回带有所需属性的 `RedirectToActionResult`。

通常会忽略未调用的模拟调用，但在设置调用末尾调用 `Verifiable` 就可以在测试中进行 mock 验证。 这通过对 `mockRepo.Verify` 的调用来完成，进行这种调用时，如果未调用所需方法，则测试将失败。

> [!NOTE]
> 通过此示例中使用的 Moq 库，可以混合可验证（或称“严格”）mock 和非可验证 mock（也称为“宽松”mock 或存根）。 详细了解[使用 Moq 自定义 Mock 行为](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior)。

示例应用中的 [SessionController](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs) 显示与特定集体讨论会话相关的信息。 该控制器包含用于处理无效 `id` 值的逻辑（以下示例中有两个 `return` 方案可用来应对这些情况）。 最后的 `return` 语句向视图 (Controllers/SessionController.cs) 返回一个新的 `StormSessionViewModel` ：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs?name=snippet_SessionController&highlight=12-16,18-22,31)]

单元测试包括对会话控制器 `Index` 操作中的每个 `return` 方案执行一个测试：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs?name=snippet_SessionControllerTests&highlight=2,11-14,18,31-32,36,50-55)]

移动到想法控制器，应用会将功能公开为 `api/ideas` 路由上的 Web API：

* `ForSession` 方法将返回与集体讨论会话关联的想法列表 (`IdeaDTO`)。
* `Create` 方法会向会话中添加新想法。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionAndCreate&highlight=1-2,21-22)]

避免直接通过 API 调用返回企业域实体。 域实体：

* 包含的数据通常比客户端所需的数据更多。
* 无需将应用的内部域模型与公开的 API 结合。

可以执行域实体与返回到客户端的类型之间的映射：

* 手动执行 LINQ `Select`，如同示例应用所用的那样。 有关详细信息，请参阅 [LINQ（语言集成查询）](/dotnet/standard/using-linq)。
* 自动生成库，如 [AutoMapper](https://github.com/AutoMapper/AutoMapper)。

接着，示例应用会演示想法控制器的 `Create` 和 `ForSession` API 方法的单元测试。

示例应用包含两个 `ForSession` 测试。 第一个测试可确定 `ForSession` 是否返回无效会话的 <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>（找不到 HTTP）：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests4&highlight=5,7-8,15-16)]

第二个 `ForSession` 测试可确定 `ForSession` 是否返回有效会话的会话想法列表 (`<List<IdeaDTO>>`)。 这些测试还会检查第一个想法，以确认其 `Name` 属性正确：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests5&highlight=5,7-8,15-18)]

若要测试 `Create` 方法在 `ModelState` 无效时的行为，示例应用会在测试中将模型错误添加到控制器。 请勿在单元测试中尝试测试模型有效性或模型绑定&mdash;仅测试操作方法在遇到无效 `ModelState` 时的行为：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests1&highlight=7,13)]

`Create` 的第二个测试依赖存储库返回 `null`，所以 mock 存储库配置为返回 `null`。 无需创建测试数据库（在内存中或其他位置）并构建将返回此结果的查询。 该测试可以在单个语句中完成，如示例代码所示：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests2&highlight=7-8,15)]

第三个 `Create` 测试 `Create_ReturnsNewlyCreatedIdeaForSession` 验证调用了存储库的 `UpdateAsync` 方法。 使用 `Verifiable` 调用 mock，然后调用模拟存储库的 `Verify` 方法，以确认执行了可验证的方法。 确保 `UpdateAsync` 方法保存了数据不是单元测试的职责&mdash;这可以通过集成测试完成。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests3&highlight=20-22,28-33)]

## <a name="test-actionresultt"></a>测试 ActionResult\<T>

在 ASP.NET Core 2.1 或更高版本中， [ActionResult \<T> ](xref:web-api/action-return-types#actionresultt-type) (<xref:Microsoft.AspNetCore.Mvc.ActionResult%601>) 使你可以返回派生自 `ActionResult` 或返回特定类型的类型。

示例应用包含将返回给定会话 `id` 的 `List<IdeaDTO>` 的方法。 如果会话 `id` 不存在，控制器将返回 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>：

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionActionResult&highlight=10,21)]

`ApiIdeasControllerTests` 中包含 `ForSessionActionResult` 控制器的两个测试。

第一个测试可确认控制器将返回 `ActionResult`，而不是不存在会话 `id` 的不存在想法列表：

* `ActionResult` 类型为 `ActionResult<List<IdeaDTO>>`。
* <xref:Microsoft.AspNetCore.Mvc.ActionResult`1.Result*> 为 <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=7,10,13-14)]

对于有效会话 `id`，第二个测试可确认该方法将返回：

* 类型为 `List<IdeaDTO>` 的 `ActionResult`。
* [ActionResult \<T> 。值](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)为 `List<IdeaDTO>` 类型。
* 列表中的第一项是与 mock 会话中存储的想法匹配的有效想法（通过调用 `GetTestSession` 获取）。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsIdeasForSession&highlight=7-8,15-18)]

示例应用还包含用于为给定会话创建新的 `Idea` 的方法。 控制器将返回：

* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>（对于无效模型）。
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>（如果会话不存在）。
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>（当使用新想法更新会话时）。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_CreateActionResult&highlight=9,16,29)]

`ApiIdeasControllerTests` 中包含 `CreateActionResult` 的三个测试。

第一个测试可确认将返回 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>（对于无效模型）。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsBadRequest_GivenInvalidModel&highlight=7,13-14)]

第二个测试可确认将返回 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>（如果会话不存在）。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=5,15,22-23)]

对于有效会话 `id`，最后一个测试可确认：

* 该方法将返回类型为 `BrainstormSession` 的 `ActionResult`。
* [ActionResult \<T> 。Result](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Result*)为 <xref:Microsoft.AspNetCore.Mvc.CreatedAtActionResult> 。 `CreatedAtActionResult` 类似于包含 `Location` 标头的 *201 Created* 响应。
* [ActionResult \<T> 。值](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)为 `BrainstormSession` 类型。
* 调用了用于更新会话 `UpdateAsync(testSession)` 的 mock 调用。 通过执行断言中的 `mockRepo.Verify()` 来检查 `Verifiable` 方法调用。
* 将返回该会话的两个 `Idea` 对象。
* 最后一项（通过对 `UpdateAsync` 的 mock 调用而添加的 `Idea`）与添加到测试中的会话的 `newIdea` 匹配。

[!code-csharp[](testing/samples/3.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNewlyCreatedIdeaForSession&highlight=20-22,28-34)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[控制器](xref:mvc/controllers/actions)在任何 ASP.NET Core MVC 应用中起着核心作用。 因此，应该对控制器表现达到预期怀有信心。 在将应用部署到生产环境之前，自动测试可以检测到错误。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/controllers/testing/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="unit-tests-of-controller-logic"></a>控制器逻辑的单元测试

[单元测试](/dotnet/articles/core/testing/unit-testing-with-dotnet-test)涉及通过基础结构和依赖项单独测试应用的一部分。 单元测试控制器逻辑时，仅测试单个操作的内容，不测试其依赖项或框架自身的行为。

将控制器操作的单元测试设置为专注于控制器的行为。 控制器单元测试将避开[筛选器](xref:mvc/controllers/filters)、[路由](xref:fundamentals/routing)或[模型绑定](xref:mvc/models/model-binding)等方案。 涵盖共同响应请求的组件之间的交互的测试由 *集成测试* 处理。 有关集成测试的详细信息，请参阅<xref:test/integration-tests>。

如果编写自定义筛选器和路由，应对其单独进行单元测试，而不是在测试特定控制器操作时进行。

要演示控制器单元测试，请查看以下示例应用中的控制器。 主控制器显示集体讨论会话列表并允许使用 POST 请求创建新的集体讨论会话：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/HomeController.cs?name=snippet_HomeController&highlight=1,5,10,31-32)]

前面的控制器：

* 遵循[显式依赖关系原则](/dotnet/standard/modern-web-apps-azure-architecture/architectural-principles#explicit-dependencies)。
* 期望[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 提供 `IBrainstormSessionRepository` 的实例。
* 可以通过使用 mock 对象框架（如 [Moq](https://www.nuget.org/packages/Moq/)）的模拟 `IBrainstormSessionRepository` 服务进行测试。 *模拟对象* 是由一组预先确定的用于测试的属性和方法行为的对象。 有关详细信息，请参阅[集成测试简介](xref:test/integration-tests#introduction-to-integration-tests)。

`HTTP GET Index` 方法没有循环或分支，且仅调用一个方法。 此操作的单元测试：

* 使用 `GetTestSessions` 方法模拟 `IBrainstormSessionRepository` 服务。 `GetTestSessions` 使用日期和会话名称创建两个 mock 集体讨论会话。
* 执行 `Index` 方法。
* 根据该方法返回的结果进行断言：
  * 将返回 <xref:Microsoft.AspNetCore.Mvc.ViewResult>。
  * [ViewDataDictionary.Model](xref:Microsoft.AspNetCore.Mvc.ViewFeatures.ViewDataDictionary.Model*) 是 `StormSessionViewModel`。
  * 有两个集体讨论会话存储在 `ViewDataDictionary.Model` 中。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_Index_ReturnsAViewResult_WithAListOfBrainstormSessions&highlight=14-17)]

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_GetTestSessions)]

主控制器的 `HTTP POST Index` 方法测试验证：

* 当 [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid*) 为 `false` 时，操作方法将返回有相应数据的 *400 错误请求* <xref:Microsoft.AspNetCore.Mvc.ViewResult>。
* 当 `ModelState.IsValid` 为 `true` 时：
  * 将调用存储库上的 `Add` 方法。
  * 将返回有正确参数的 <xref:Microsoft.AspNetCore.Mvc.RedirectToActionResult>。

通过使用 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.AddModelError*> 添加错误来测试无效模型状态，如下第一个测试所示：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/HomeControllerTests.cs?name=snippet_ModelState_ValidOrInvalid&highlight=9,16-17,38-41)]

当 [ModelState](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary) 无效时，将返回与 GET 请求相同的 `ViewResult`。 测试不会尝试传入无效模型。 传入无效模型不是有效的方法，因为不会运行模型绑定（尽管[集成测试](xref:test/integration-tests)使用模型绑定）。 在本例中，不测试模型绑定。 这些单元测试仅测试操作方法中的代码。

第二个测试验证 `ModelState` 有效时的情况：

* 已通过存储库添加新的 `BrainstormSession`。
* 该方法将返回带有所需属性的 `RedirectToActionResult`。

通常会忽略未调用的模拟调用，但在设置调用末尾调用 `Verifiable` 就可以在测试中进行 mock 验证。 这通过对 `mockRepo.Verify` 的调用来完成，进行这种调用时，如果未调用所需方法，则测试将失败。

> [!NOTE]
> 通过此示例中使用的 Moq 库，可以混合可验证（或称“严格”）mock 和非可验证 mock（也称为“宽松”mock 或存根）。 详细了解[使用 Moq 自定义 Mock 行为](https://github.com/Moq/moq4/wiki/Quickstart#customizing-mock-behavior)。

示例应用中的 [SessionController](https://github.com/dotnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/controllers/testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs) 显示与特定集体讨论会话相关的信息。 该控制器包含用于处理无效 `id` 值的逻辑（以下示例中有两个 `return` 方案可用来应对这些情况）。 最后的 `return` 语句向视图 (Controllers/SessionController.cs) 返回一个新的 `StormSessionViewModel` ：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Controllers/SessionController.cs?name=snippet_SessionController&highlight=12-16,18-22,31)]

单元测试包括对会话控制器 `Index` 操作中的每个 `return` 方案执行一个测试：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/SessionControllerTests.cs?name=snippet_SessionControllerTests&highlight=2,11-14,18,31-32,36,50-55)]

移动到想法控制器，应用会将功能公开为 `api/ideas` 路由上的 Web API：

* `ForSession` 方法将返回与集体讨论会话关联的想法列表 (`IdeaDTO`)。
* `Create` 方法会向会话中添加新想法。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionAndCreate&highlight=1-2,21-22)]

避免直接通过 API 调用返回企业域实体。 域实体：

* 包含的数据通常比客户端所需的数据更多。
* 无需将应用的内部域模型与公开的 API 结合。

可以执行域实体与返回到客户端的类型之间的映射：

* 手动执行 LINQ `Select`，如同示例应用所用的那样。 有关详细信息，请参阅 [LINQ（语言集成查询）](/dotnet/standard/using-linq)。
* 自动生成库，如 [AutoMapper](https://github.com/AutoMapper/AutoMapper)。

接着，示例应用会演示想法控制器的 `Create` 和 `ForSession` API 方法的单元测试。

示例应用包含两个 `ForSession` 测试。 第一个测试可确定 `ForSession` 是否返回无效会话的 <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>（找不到 HTTP）：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests4&highlight=5,7-8,15-16)]

第二个 `ForSession` 测试可确定 `ForSession` 是否返回有效会话的会话想法列表 (`<List<IdeaDTO>>`)。 这些测试还会检查第一个想法，以确认其 `Name` 属性正确：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests5&highlight=5,7-8,15-18)]

若要测试 `Create` 方法在 `ModelState` 无效时的行为，示例应用会在测试中将模型错误添加到控制器。 请勿在单元测试中尝试测试模型有效性或模型绑定&mdash;仅测试操作方法在遇到无效 `ModelState` 时的行为：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests1&highlight=7,13)]

`Create` 的第二个测试依赖存储库返回 `null`，所以 mock 存储库配置为返回 `null`。 无需创建测试数据库（在内存中或其他位置）并构建将返回此结果的查询。 该测试可以在单个语句中完成，如示例代码所示：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests2&highlight=7-8,15)]

第三个 `Create` 测试 `Create_ReturnsNewlyCreatedIdeaForSession` 验证调用了存储库的 `UpdateAsync` 方法。 使用 `Verifiable` 调用 mock，然后调用模拟存储库的 `Verify` 方法，以确认执行了可验证的方法。 确保 `UpdateAsync` 方法保存了数据不是单元测试的职责&mdash;这可以通过集成测试完成。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ApiIdeasControllerTests3&highlight=20-22,28-33)]

## <a name="test-actionresultt"></a>测试 ActionResult\<T>

在 ASP.NET Core 2.1 或更高版本中， [ActionResult \<T> ](xref:web-api/action-return-types#actionresultt-type) (<xref:Microsoft.AspNetCore.Mvc.ActionResult%601>) 使你可以返回派生自 `ActionResult` 或返回特定类型的类型。

示例应用包含将返回给定会话 `id` 的 `List<IdeaDTO>` 的方法。 如果会话 `id` 不存在，控制器将返回 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>：

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_ForSessionActionResult&highlight=10,21)]

`ApiIdeasControllerTests` 中包含 `ForSessionActionResult` 控制器的两个测试。

第一个测试可确认控制器将返回 `ActionResult`，而不是不存在会话 `id` 的不存在想法列表：

* `ActionResult` 类型为 `ActionResult<List<IdeaDTO>>`。
* <xref:Microsoft.AspNetCore.Mvc.ActionResult`1.Result*> 为 <xref:Microsoft.AspNetCore.Mvc.NotFoundObjectResult>。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=7,10,13-14)]

对于有效会话 `id`，第二个测试可确认该方法将返回：

* 类型为 `List<IdeaDTO>` 的 `ActionResult`。
* [ActionResult \<T> 。值](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)为 `List<IdeaDTO>` 类型。
* 列表中的第一项是与 mock 会话中存储的想法匹配的有效想法（通过调用 `GetTestSession` 获取）。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_ForSessionActionResult_ReturnsIdeasForSession&highlight=7-8,15-18)]

示例应用还包含用于为给定会话创建新的 `Idea` 的方法。 控制器将返回：

* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>（对于无效模型）。
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>（如果会话不存在）。
* <xref:Microsoft.AspNetCore.Mvc.ControllerBase.CreatedAtAction*>（当使用新想法更新会话时）。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/src/TestingControllersSample/Api/IdeasController.cs?name=snippet_CreateActionResult&highlight=9,16,29)]

`ApiIdeasControllerTests` 中包含 `CreateActionResult` 的三个测试。

第一个测试可确认将返回 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.BadRequest*>（对于无效模型）。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsBadRequest_GivenInvalidModel&highlight=7,13-14)]

第二个测试可确认将返回 <xref:Microsoft.AspNetCore.Mvc.ControllerBase.NotFound*>（如果会话不存在）。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNotFoundObjectResultForNonexistentSession&highlight=5,15,22-23)]

对于有效会话 `id`，最后一个测试可确认：

* 该方法将返回类型为 `BrainstormSession` 的 `ActionResult`。
* [ActionResult \<T> 。Result](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Result*)为 <xref:Microsoft.AspNetCore.Mvc.CreatedAtActionResult> 。 `CreatedAtActionResult` 类似于包含 `Location` 标头的 *201 Created* 响应。
* [ActionResult \<T> 。值](xref:Microsoft.AspNetCore.Mvc.ActionResult%601.Value*)为 `BrainstormSession` 类型。
* 调用了用于更新会话 `UpdateAsync(testSession)` 的 mock 调用。 通过执行断言中的 `mockRepo.Verify()` 来检查 `Verifiable` 方法调用。
* 将返回该会话的两个 `Idea` 对象。
* 最后一项（通过对 `UpdateAsync` 的 mock 调用而添加的 `Idea`）与添加到测试中的会话的 `newIdea` 匹配。

[!code-csharp[](testing/samples/2.x/TestingControllersSample/tests/TestingControllersSample.Tests/UnitTests/ApiIdeasControllerTests.cs?name=snippet_CreateActionResult_ReturnsNewlyCreatedIdeaForSession&highlight=20-22,28-34)]

::: moniker-end

## <a name="additional-resources"></a>其他资源

* <xref:test/integration-tests>
* [用 Visual Studio 创建和运行单元测试](/visualstudio/test/unit-test-your-code)
* [MyTested AspNetCore-ASP.NET CORE mvc 的流畅测试库](https://github.com/ivaylokenov/MyTested.AspNetCore.Mvc)：强类型单元测试库，提供用于测试 Mvc 和 web API 应用的流畅界面。 （ *不由 Microsoft 进行支持或维护* 。）
* [JustMockLite](https://github.com/telerik/JustMockLite)：面向 .NET 开发人员的模拟框架。 （ *不由 Microsoft 进行支持或维护* 。）

