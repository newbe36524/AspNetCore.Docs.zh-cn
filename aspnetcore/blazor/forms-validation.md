---
title: ASP.NET Core Blazor 窗体和验证
author: guardrex
description: 了解如何在 Blazor 中使用窗体和字段验证方案。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 09/17/2020
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
uid: blazor/forms-validation
ms.openlocfilehash: ad244c29c8e8e904793745119366cd677389b12d
ms.sourcegitcommit: 2e3a967331b2c69f585dd61e9ad5c09763615b44
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/27/2020
ms.locfileid: "92690606"
---
# <a name="aspnet-core-no-locblazor-forms-and-validation"></a>ASP.NET Core Blazor 窗体和验证

作者：[Daniel Roth](https://github.com/danroth27)、[Rémi Bourgarel](https://remibou.github.io/) 和 [Luke Latham](https://github.com/guardrex)

在 Blazor 中，使用[数据注释](xref:mvc/models/validation)支持窗体和验证。

下面的 `ExampleModel` 类型使用数据注释定义验证逻辑：

```csharp
using System.ComponentModel.DataAnnotations;

public class ExampleModel
{
    [Required]
    [StringLength(10, ErrorMessage = "Name is too long.")]
    public string Name { get; set; }
}
```

窗体是使用 <xref:Microsoft.AspNetCore.Components.Forms.EditForm> 组件定义的。 以下窗体展示了典型的元素、组件和 Razor 代码：

```razor
<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <InputText id="name" @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        ...
    }
}
```

在上面的示例中：

* 该窗体使用 `ExampleModel` 类型中定义的验证来验证 `name` 字段中的用户输入。 该模型在组件的 `@code` 块中创建，并保存在私有字段 (`exampleModel`) 中。 该字段分配给 `<EditForm>` 元素的 `Model` 属性。
* <xref:Microsoft.AspNetCore.Components.Forms.InputText> 组件的 `@bind-Value` 进行以下绑定：
  * 将模型属性 (`exampleModel.Name`) 绑定到 <xref:Microsoft.AspNetCore.Components.Forms.InputText> 组件的 `Value` 属性。 有关属性绑定的详细信息，请参阅 <xref:blazor/components/data-binding#parent-to-child-binding-with-component-parameters>。
  * 将更改事件委托绑定到 <xref:Microsoft.AspNetCore.Components.Forms.InputText> 组件的 `ValueChanged` 属性。
* <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> [验证器组件](#validator-components)使用数据注释附加验证支持。
* <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 组件汇总验证消息。
* 窗体成功提交（通过验证）时触发 `HandleValidSubmit`。

## <a name="built-in-forms-components"></a>内置窗体组件

可使用一组内置的组件来接收和验证用户输入。 当更改输入和提交窗体时，将验证输入。 下表显示了可用的输入组件。

::: moniker range=">= aspnetcore-5.0"

| 输入组件 | 呈现为&hellip; |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| [`InputFile`](xref:blazor/file-uploads) | `<input type="file">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| [`InputRadio`](#radio-buttons) | `<input type="radio">` |
| [`InputRadioGroup`](#radio-buttons) | `<input type="radio">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

::: moniker-end

::: moniker range="< aspnetcore-5.0"

| 输入组件 | 呈现为&hellip; |
| --------------- | ------------------- |
| <xref:Microsoft.AspNetCore.Components.Forms.InputCheckbox> | `<input type="checkbox">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> | `<input type="date">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> | `<input type="number">` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601> | `<select>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputText> | `<input>` |
| <xref:Microsoft.AspNetCore.Components.Forms.InputTextArea> | `<textarea>` |

> [!NOTE]
> `InputRadio` 和 `InputRadioGroup` 组件在 ASP.NET Core 5.0 或更高版本中可用。 有关详细信息，请选择本文的 5.0 或更高版本。

::: moniker-end

所有输入组件（包括 <xref:Microsoft.AspNetCore.Components.Forms.EditForm>）都支持任意属性。 与某个组件参数不匹配的所有属性都将添加到呈现的 HTML 元素中。

输入组件为验证字段何时更改（包括更新字段 CSS 类以反映字段状态）提供默认行为。 某些组件包含有用的分析逻辑。 例如，<xref:Microsoft.AspNetCore.Components.Forms.InputDate%601> 和 <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601> 通过将无法分析的值注册为验证错误，以恰当的方式来处理无法分析的值。 可接受 Null 值的类型也支持目标字段的为 Null 性（例如，`int?`）。

下面的 `Starship` 类型使用比之前的 `ExampleModel` 更大的属性和数据注释集来定义验证逻辑：

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    [Required]
    [StringLength(16, ErrorMessage = "Identifier too long (16 character limit).")]
    public string Identifier { get; set; }

    public string Description { get; set; }

    [Required]
    public string Classification { get; set; }

    [Range(1, 100000, ErrorMessage = "Accommodation invalid (1-100000).")]
    public int MaximumAccommodation { get; set; }

    [Required]
    [Range(typeof(bool), "true", "true", 
        ErrorMessage = "This form disallows unapproved ships.")]
    public bool IsValidatedDesign { get; set; }

    [Required]
    public DateTime ProductionDate { get; set; }
}
```

在上面的示例中，`Description` 是可选的，因为不存在任何数据注释。

以下窗体使用 `Starship` 模型中定义的验证来验证用户输入：

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" />
        </label>
    </p>

    <button type="submit">Submit</button>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>, 
        &copy;1966-2019 CBS Studios, Inc. and 
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private void HandleValidSubmit()
    {
        ...
    }
}
```

<xref:Microsoft.AspNetCore.Components.Forms.EditForm> 创建一个 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 作为[级联值](xref:blazor/components/cascading-values-and-parameters)来跟踪有关编辑过程的元数据，其中包括已修改的字段和当前的验证消息。

将 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 或 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model?displayProperty=nameWithType> 分配给 <xref:Microsoft.AspNetCore.Components.Forms.EditForm> 。 不支持同时分配两者，会生成运行时错误。

<xref:Microsoft.AspNetCore.Components.Forms.EditForm> 为有效和无效的窗体提交提供便捷的事件：

* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit>
* <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnInvalidSubmit>

通过 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit>，使用自定义代码触发验证并检查字段值。

如下示例中：

* 选择 `Submit` 按钮时，执行 `HandleSubmit` 方法。
* 通过调用 <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Validate%2A?displayProperty=nameWithType> 验证窗体。
* 根据验证结果执行其他代码。 将业务逻辑放在分配给 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnSubmit> 的方法中。

```razor
<EditForm EditContext="@editContext" OnSubmit="@HandleSubmit">

    ...

    <button type="submit">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
    }

    private async Task HandleSubmit()
    {
        var isValid = editContext.Validate();

        if (isValid)
        {
            ...
        }
        else
        {
            ...
        }
    }
}
```

> [!NOTE]
> Framework API 不存在，无法直接从 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 清除验证消息。 因此，通常建议不要在窗体中将验证消息添加到新的 <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore>。 若要管理验证消息，请将[验证器组件](#validator-components)与[业务逻辑验证代码](#business-logic-validation)一起使用，如本文所述。

::: moniker range=">= aspnetcore-5.0"

## <a name="display-name-support"></a>显示名称支持

*本部分应用于 .NET 5 候选发布 1 (RC1) 或更高版本中的 ASP.NET Core。*

以下内置组件支持带有 `DisplayName` 参数的显示名称：

* <xref:Microsoft.AspNetCore.Components.Forms.InputDate%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputNumber%601>
* <xref:Microsoft.AspNetCore.Components.Forms.InputSelect%601>

在下面的 `InputDate` 组件示例中：

* 显示名称 (`DisplayName`) 设置为 `birthday`。
* 该组件作为 `DateTime` 类型绑定到 `BirthDate` 属性。

```razor
<InputDate @bind-Value="@BirthDate" DisplayName="birthday" />

@code {
    public DateTime BirthDate { get; set; }
}
```

如果用户不提供日期值，则验证错误将显示为：

```
The birthday must be a date.
```

::: moniker-end

## <a name="validator-components"></a>验证器组件

验证器组件通过管理窗体的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 的 <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> 来支持窗体验证。

Blazor 框架提供了 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件，以将验证支持附加到基于[验证属性（数据批注）](xref:mvc/models/validation#validation-attributes)的窗体。 创建自定义验证器组件，以处理同一页上不同窗体或不同窗体处理步骤上相同窗体的验证消息，例如客户端验证，然后是服务器端验证。 本文的以下部分将使用本部分 `CustomValidator` 中所示的验证器组件示例：

* [业务逻辑验证](#business-logic-validation)
* [服务器验证](#server-validation)

> [!NOTE]
> 在许多情况下，可使用自定义数据注释验证属性来代替自定义验证器组件。 应用于窗体模型的自定义属性使用 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件激活。 当与服务器端验证一起使用时，应用于模型的所有自定义属性都必须可在服务器上执行。 有关详细信息，请参阅 <xref:mvc/models/validation#alternatives-to-built-in-attributes>。

从 <xref:Microsoft.AspNetCore.Components.ComponentBase> 创建验证器组件：

* 窗体的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 是组件的[级联参数](xref:blazor/components/cascading-values-and-parameters)。
* 初始化验证器组件时，将创建一个新的 <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore> 来维护当前的窗体错误列表。
* 当窗体组件中的开发人员代码调用 `DisplayErrors` 方法时，消息存储接收错误。 这些错误会传递到 [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) 中的 `DisplayErrors` 方法。 在字典中，键是具有一个或多个错误的窗体字段的名称。 值为错误列表。
* 发生以下任一情况时，将清除消息：
  * 引发 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 事件时，会在 <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnValidationRequested> 上请求验证。 所有错误都将被清除。
  * 引发 <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> 事件时，窗体中的字段会更改。 仅清除字段的错误。
  * `ClearErrors` 方法由开发人员代码调用。 所有错误都将被清除。

```csharp
using System;
using System.Collections.Generic;
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Forms;

namespace BlazorSample.Client
{
    public class CustomValidator : ComponentBase
    {
        private ValidationMessageStore messageStore;

        [CascadingParameter]
        private EditContext CurrentEditContext { get; set; }

        protected override void OnInitialized()
        {
            if (CurrentEditContext == null)
            {
                throw new InvalidOperationException(
                    $"{nameof(CustomValidator)} requires a cascading " +
                    $"parameter of type {nameof(EditContext)}. " +
                    $"For example, you can use {nameof(CustomValidator)} " +
                    $"inside an {nameof(EditForm)}.");
            }

            messageStore = new ValidationMessageStore(CurrentEditContext);

            CurrentEditContext.OnValidationRequested += (s, e) => 
                messageStore.Clear();
            CurrentEditContext.OnFieldChanged += (s, e) => 
                messageStore.Clear(e.FieldIdentifier);
        }

        public void DisplayErrors(Dictionary<string, List<string>> errors)
        {
            foreach (var err in errors)
            {
                messageStore.Add(CurrentEditContext.Field(err.Key), err.Value);
            }

            CurrentEditContext.NotifyValidationStateChanged();
        }

        public void ClearErrors()
        {
            messageStore.Clear();
            CurrentEditContext.NotifyValidationStateChanged();
        }
    }
}
```

## <a name="business-logic-validation"></a>业务逻辑验证

可通过接收字典中的窗体错误的[验证器组件](#validator-components)完成业务逻辑验证。

如下示例中：

* 使用本文的[验证器组件](#validator-components)部分的 `CustomValidator` 组件。
* 如果用户选择 `Defense` 交付分类 (`Classification`)，则验证需要交付说明 (`Description`) 的值。

在组件中设置验证消息时，它们将被添加到验证器的 <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore>，并在 <xref:Microsoft.AspNetCore.Components.Forms.EditForm> 中显示：

```razor
@page "/FormsValidation"

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <CustomValidator @ref="customValidator" />
    <ValidationSummary />

    ...

</EditForm>

@code {
    private CustomValidator customValidator;
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private void HandleValidSubmit()
    {
        customValidator.ClearErrors();

        var errors = new Dictionary<string, List<string>>();

        if (starship.Classification == "Defense" &&
                string.IsNullOrEmpty(starship.Description))
        {
            errors.Add(nameof(starship.Description),
                new List<string>() { "For a 'Defense' ship classification, " +
                "'Description' is required." });
        }

        if (errors.Count() > 0)
        {
            customValidator.DisplayErrors(errors);
        }
        else
        {
            // Process the form
        }
    }
}
```

> [!NOTE]
> 除了使用[验证组件](#validator-components)，还可使用数据注释验证属性。 应用于窗体模型的自定义属性使用 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件激活。 当与服务器端验证一起使用时，该属性都必须可在服务器上执行。 有关详细信息，请参阅 <xref:mvc/models/validation#alternatives-to-built-in-attributes>。

## <a name="server-validation"></a>服务器验证

服务器验证可通过服务器[验证器组件](#validator-components)完成：

* 使用 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件处理窗体中的客户端验证。
* 当窗体传递客户端验证（调用 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.OnValidSubmit>）时，将 <xref:Microsoft.AspNetCore.Components.Forms.EditContext.Model?displayProperty=nameWithType> 发送到后端服务器 API 进行窗体处理。
* 处理服务器上的模型验证。
* 服务器 API 包括开发人员提供的内置框架数据注释验证和自定义验证逻辑。 如果验证在服务器上传递，则处理窗格并发送回成功状态代码（200 - 正常）。 如果验证失败，则返回失败状态代码（400 - 错误请求）和字段验证错误。
* 成功时禁用窗体，否则显示错误。

下面的示例基于：

* 托管的 Blazor 解决方案，由 [Blazor 托管项目模板](xref:blazor/hosting-models#blazor-webassembly)创建。 此示例可与[安全性和 Identity 文档](xref:blazor/security/webassembly/index#implementation-guidance)中介绍的任何安全托管 Blazor 方案一起使用。
* 前面的[内置窗体组件](#built-in-forms-components)部分中的 Starfleet Starship 数据库窗体示例。
* Blazor 框架的 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件。
* [验证器组件](#validator-components)部分中显示的 `CustomValidator` 组件。

在下面的示例中，如果用户选择 `Defense` 交付分类 (`Classification`)，则服务器 API 将验证是否为交付说明 (`Description`) 提供了值。

将 `Starship` 模型放入解决方案的 `Shared` 项目中，以便客户端和服务器应用都可使用该模型。 模型需要数据注释，因此请将 [`System.ComponentModel.Annotations`](https://www.nuget.org/packages/System.ComponentModel.Annotations) 的包引用添加到 `Shared` 项目的项目文件中：

```xml
<ItemGroup>
  <PackageReference Include="System.ComponentModel.Annotations" Version="{VERSION}" />
</ItemGroup>
```

若要确定包的最新非预览版本，请前往 [NuGet.org](https://www.nuget.org/packages/System.ComponentModel.Annotations) 查看包版本历史记录。

在服务器 API 项目中，添加控制器来处理 Starship 验证请求 (`Controllers/StarshipValidation.cs`) 并返回失败的验证消息：

```csharp
using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using BlazorSample.Shared;

namespace BlazorSample.Server.Controllers
{
    [Authorize]
    [ApiController]
    [Route("[controller]")]
    public class StarshipValidationController : ControllerBase
    {
        private readonly ILogger<StarshipValidationController> logger;

        public StarshipValidationController(
            ILogger<StarshipValidationController> logger)
        {
            this.logger = logger;
        }

        [HttpPost]
        public async Task<IActionResult> Post(Starship starship)
        {
            try
            {
                if (starship.Classification == "Defense" && 
                    string.IsNullOrEmpty(starship.Description))
                {
                    ModelState.AddModelError(nameof(starship.Description),
                        "For a 'Defense' ship " +
                        "classification, 'Description' is required.");
                }
                else
                {
                    // Process the form asynchronously
                    // async ...

                    return Ok(ModelState);
                }
            }
            catch (Exception ex)
            {
                logger.LogError("Validation Error: {Message}", ex.Message);
            }

            return BadRequest(ModelState);
        }
    }
}
```

当服务器上发生模型绑定验证错误时，[`ApiController`](xref:web-api/index) (<xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute>) 通常通过 <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> 返回[默认错误请求响应](xref:web-api/index#default-badrequest-response)。 如下例所示，当“Starfleet Starship 数据库”窗格的部分字段未提交且窗格未通过验证时，响应包含的数据不仅仅是验证错误：

```json
{
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": {
    "Identifier": ["The Identifier field is required."],
    "Classification": ["The Classification field is required."],
    "IsValidatedDesign": ["This form disallows unapproved ships."],
    "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
  }
}
```

如果服务器 API 返回前面的默认 JSON 响应，则客户端可分析响应以获取 `errors` 节点的子节点。 但是，分析文件不方便。 分析 JSON 需要调用 <xref:System.Net.Http.Json.HttpContentJsonExtensions.ReadFromJsonAsync%2A> 后的其他代码，以生成窗体验证错误处理的 [`Dictionary<string, List<string>>`](xref:System.Collections.Generic.Dictionary`2) 错误。 理想情况下，服务器 API 应只返回验证错误：

```json
{
  "Identifier": ["The Identifier field is required."],
  "Classification": ["The Classification field is required."],
  "IsValidatedDesign": ["This form disallows unapproved ships."],
  "MaximumAccommodation": ["Accommodation invalid (1-100000)."]
}
```

若要修改服务器 API 的响应，使其仅返回验证错误，请更改在 `Startup.ConfigureServices` 中注释了 <xref:Microsoft.AspNetCore.Mvc.ApiControllerAttribute> 的操作上调用的委托。 对于 API 终结点 (`/StarshipValidation`)，返回具有 <xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary> 的 <xref:Microsoft.AspNetCore.Mvc.BadRequestObjectResult>。 对于任何其他 API 终结点，通过使用新的 <xref:Microsoft.AspNetCore.Mvc.ValidationProblemDetails> 返回对象结果来保留默认行为：

```csharp
using Microsoft.AspNetCore.Mvc;

...

services.AddControllersWithViews()
    .ConfigureApiBehaviorOptions(options =>
    {
        options.InvalidModelStateResponseFactory = context =>
        {
            if (context.HttpContext.Request.Path == "/StarshipValidation")
            {
                return new BadRequestObjectResult(context.ModelState);
            }
            else
            {
                return new BadRequestObjectResult(
                    new ValidationProblemDetails(context.ModelState));
            }
        };
    });
```

有关详细信息，请参阅 <xref:web-api/handle-errors#validation-failure-error-response>。

在客户端项目中，添加[验证器组件](#validator-components)部分中显示的验证器组件。

在客户端项目中，更新“Starfleet Starship 数据库”窗体，以显示服务器验证错误和 `CustomValidator` 组件的帮助。 当服务器 API 返回验证消息时，这些消息将添加到 `CustomValidator` 组件的 <xref:Microsoft.AspNetCore.Components.Forms.ValidationMessageStore>。 此错误在窗体的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 中提供，以供窗体的 <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 显示：

```razor
@page "/FormValidation"
@using System.Net
@using System.Net.Http.Json
@using Microsoft.AspNetCore.Authorization
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@using Microsoft.Extensions.Logging
@using BlazorSample.Shared
@attribute [Authorize]
@inject HttpClient Http
@inject ILogger<FormValidation> Logger

<h1>Starfleet Starship Database</h1>

<h2>New Ship Entry Form</h2>

<EditForm Model="@starship" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <CustomValidator @ref="customValidator" />
    <ValidationSummary />

    <p>
        <label>
            Identifier:
            <InputText @bind-Value="starship.Identifier" disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Description (optional):
            <InputTextArea @bind-Value="starship.Description" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Primary Classification:
            <InputSelect @bind-Value="starship.Classification" disabled="@disabled">
                <option value="">Select classification ...</option>
                <option value="Exploration">Exploration</option>
                <option value="Diplomacy">Diplomacy</option>
                <option value="Defense">Defense</option>
            </InputSelect>
        </label>
    </p>
    <p>
        <label>
            Maximum Accommodation:
            <InputNumber @bind-Value="starship.MaximumAccommodation" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Engineering Approval:
            <InputCheckbox @bind-Value="starship.IsValidatedDesign" 
                disabled="@disabled" />
        </label>
    </p>
    <p>
        <label>
            Production Date:
            <InputDate @bind-Value="starship.ProductionDate" disabled="@disabled" />
        </label>
    </p>

    <button type="submit" disabled="@disabled">Submit</button>

    <p style="@messageStyles">
        @message
    </p>

    <p>
        <a href="http://www.startrek.com/">Star Trek</a>,
        &copy;1966-2019 CBS Studios, Inc. and
        <a href="https://www.paramount.com">Paramount Pictures</a>
    </p>
</EditForm>

@code {
    private bool disabled;
    private string message;
    private string messageStyles = "visibility:hidden";
    private CustomValidator customValidator;
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };

    private async Task HandleValidSubmit(EditContext editContext)
    {
        customValidator.ClearErrors();

        try
        {
            var response = await Http.PostAsJsonAsync<Starship>(
                "StarshipValidation", (Starship)editContext.Model);

            var errors = await response.Content
                .ReadFromJsonAsync<Dictionary<string, List<string>>>();

            if (response.StatusCode == HttpStatusCode.BadRequest && 
                errors.Count() > 0)
            {
                customValidator.DisplayErrors(errors);
            }
            else if (!response.IsSuccessStatusCode)
            {
                throw new HttpRequestException(
                    $"Validation failed. Status Code: {response.StatusCode}");
            }
            else
            {
                disabled = true;
                messageStyles = "color:green";
                message = "The form has been processed.";
            }
        }
        catch (AccessTokenNotAvailableException ex)
        {
            ex.Redirect();
        }
        catch (Exception ex)
        {
            Logger.LogError("Form processing error: {Message}", ex.Message);
            disabled = true;
            messageStyles = "color:red";
            message = "There was an error processing the form.";
        }
    }
}
```

> [!NOTE]
> 除了[验证组件](#validator-components)，还可使用数据注释验证属性。 应用于窗体模型的自定义属性使用 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件激活。 当与服务器端验证一起使用时，该属性都必须可在服务器上执行。 有关详细信息，请参阅 <xref:mvc/models/validation#alternatives-to-built-in-attributes>。

> [!NOTE]
> 本部分中的服务器端验证方法适用于本文档集中的所有 Blazor WebAssembly 托管解决方案示例：
>
> * [Azure Active Directory (AAD)](xref:blazor/security/webassembly/hosted-with-azure-active-directory)
> * [Azure Active Directory (AAD) B2C](xref:blazor/security/webassembly/hosted-with-azure-active-directory-b2c)
> * [Identity 服务器](xref:blazor/security/webassembly/hosted-with-identity-server)

## <a name="inputtext-based-on-the-input-event"></a>基于输入事件的 InputText

使用 <xref:Microsoft.AspNetCore.Components.Forms.InputText> 组件创建一个使用 `input` 事件而不是 `change` 事件的自定义组件。

在下面的示例中，`CustomInputText` 组件继承框架的 `InputText` 组件，并将事件绑定 (<xref:Microsoft.AspNetCore.Components.EventCallbackFactoryBinderExtensions.CreateBinder%2A>) 设置为 `oninput` 事件。

`Shared/CustomInputText.razor`:

```razor
@inherits InputText

<input 
    @attributes="AdditionalAttributes" 
    class="@CssClass" 
    value="@CurrentValue"
    @oninput="EventCallback.Factory.CreateBinder<string>(
         this, __value => CurrentValueAsString = __value, 
         CurrentValueAsString)" />
```

`CustomInputText` 组件可在任何使用 <xref:Microsoft.AspNetCore.Components.Forms.InputText> 的位置使用：

`Pages/TestForm.razor`:

```razor
@page "/testform"
@using System.ComponentModel.DataAnnotations;

<EditForm Model="@exampleModel" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    <CustomInputText @bind-Value="exampleModel.Name" />

    <button type="submit">Submit</button>
</EditForm>

<p>
    CurrentValue: @exampleModel.Name
</p>

@code {
    private ExampleModel exampleModel = new ExampleModel();

    private void HandleValidSubmit()
    {
        ...
    }

    public class ExampleModel
    {
        [Required]
        [StringLength(10, ErrorMessage = "Name is too long.")]
        public string Name { get; set; }
    }
}
```

## <a name="radio-buttons"></a>单选按钮

::: moniker range=">= aspnetcore-5.0"

结合使用 `InputRadio` 组件和 `InputRadioGroup` 组件以创建单选按钮组。 在下面的示例中，将属性添加到[内置窗体组件](#built-in-forms-components)部分所述的 `Starship` 模型中：

```csharp
[Required]
[Range(typeof(Manufacturer), nameof(Manufacturer.SpaceX), 
    nameof(Manufacturer.VirginGalactic), ErrorMessage = "Pick a manufacturer.")]
public Manufacturer Manufacturer { get; set; } = Manufacturer.Unknown;

[Required, EnumDataType(typeof(Color))]
public Color? Color { get; set; } = null;

[Required, EnumDataType(typeof(Engine))]
public Engine? Engine { get; set; } = null;
```

将以下 `enums` 添加到应用。 创建一个新文件来保存 `enums`，或将 `enums` 添加到 `Starship.cs` 文件中。 使 `Starship` 模型和 Starfleet Starship 数据库窗体都可访问 `enums`：

```csharp
public enum Manufacturer { SpaceX, NASA, ULA, Virgin, Unknown }
public enum Color { ImperialRed, SpacecruiserGreen, StarshipBlue, VoyagerOrange }
public enum Engine { Ion, Plasma, Fusion, Warp }
```

更新[内置窗体组件](#built-in-forms-components)部分所述的 Starfleet Starship 数据库窗体。 添加组件以生成：

* 用于选择飞船制造商的单选按钮组。
* 用于选择飞船颜色和引擎的嵌套式单选按钮组。

```razor
<p>
    <InputRadioGroup @bind-Value="starship.Manufacturer">
        Manufacturer:
        <br>
        @foreach (var manufacturer in (Manufacturer[])Enum
            .GetValues(typeof(Manufacturer)))
        {
            <InputRadio Value="manufacturer" />
            @manufacturer
            <br>
        }
    </InputRadioGroup>
</p>

<p>
    Pick one color and one engine:
    <InputRadioGroup Name="engine" @bind-Value="starship.Engine">
        <InputRadioGroup Name="color" @bind-Value="starship.Color">
            <InputRadio Name="color" Value="Color.ImperialRed" />Imperial Red<br>
            <InputRadio Name="engine" Value="Engine.Ion" />Ion<br>
            <InputRadio Name="color" Value="Color.SpacecruiserGreen" />
                Spacecruiser Green<br>
            <InputRadio Name="engine" Value="Engine.Plasma" />Plasma<br>
            <InputRadio Name="color" Value="Color.StarshipBlue" />Starship Blue<br>
            <InputRadio Name="engine" Value="Engine.Fusion" />Fusion<br>
            <InputRadio Name="color" Value="Color.VoyagerOrange" />
                Voyager Orange<br>
            <InputRadio Name="engine" Value="Engine.Warp" />Warp<br>
        </InputRadioGroup>
    </InputRadioGroup>
</p>
```

> [!NOTE]
> 如果省略 `Name`，则 `InputRadio` 组件按其最新上级进行分组。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

使用窗体中的单选按钮时，数据绑定的处理方式与其他元素不同，因为单选按钮是作为一个组进行计算的。 每个单选按钮的值是固定的，但单选按钮组的值是所选单选按钮的值。 以下示例介绍如何：

* 处理单选按钮组的数据绑定。
* 使用自定义 `InputRadio` 组件支持验证。

```razor
@using System.Globalization
@typeparam TValue
@inherits InputBase<TValue>

<input @attributes="AdditionalAttributes" type="radio" value="@SelectedValue" 
       checked="@(SelectedValue.Equals(Value))" @onchange="OnChange" />

@code {
    [Parameter]
    public TValue SelectedValue { get; set; }

    private void OnChange(ChangeEventArgs args)
    {
        CurrentValueAsString = args.Value.ToString();
    }

    protected override bool TryParseValueFromString(string value, 
        out TValue result, out string errorMessage)
    {
        var success = BindConverter.TryConvertTo<TValue>(
            value, CultureInfo.CurrentCulture, out var parsedValue);
        if (success)
        {
            result = parsedValue;
            errorMessage = null;

            return true;
        }
        else
        {
            result = default;
            errorMessage = $"{FieldIdentifier.FieldName} field isn't valid.";

            return false;
        }
    }
}
```

以下 <xref:Microsoft.AspNetCore.Components.Forms.EditForm> 使用前面的 `InputRadio` 组件来获取和验证用户的评级：

```razor
@page "/RadioButtonExample"
@using System.ComponentModel.DataAnnotations

<h1>Radio Button Group Test</h1>

<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary />

    @for (int i = 1; i <= 5; i++)
    {
        <label>
            <InputRadio name="rate" SelectedValue="i" @bind-Value="model.Rating" />
            @i
        </label>
    }

    <button type="submit">Submit</button>
</EditForm>

<p>You chose: @model.Rating</p>

@code {
    private Model model = new Model();

    private void HandleValidSubmit()
    {
        ...
    }

    public class Model
    {
        [Range(1, 5)]
        public int Rating { get; set; }
    }
}
```

::: moniker-end

## <a name="binding-select-element-options-to-c-object-null-values"></a>将 `<select>` 元素选项绑定到 C# 对象 `null` 值

由于以下原因，没有将 `<select>` 元素选项值表示为 C# 对象 `null` 值的合理方法：

* HTML 属性不能具有 `null` 值。 HTML 中最接近的 `null` 等效项是 `<option>` 元素中缺少 HTML `value` 属性。
* 选择没有 `value` 属性的 `<option>` 时，浏览器会将值视为该 `<option>` 的元素的 文本内容。

Blazor 框架不会尝试取消默认行为，因为这会涉及以下操作：

* 在框架中创建一系列特殊的解决办法。
* 对当前框架行为进行重大更改。

::: moniker range=">= aspnetcore-5.0"

HTML 中最合理的 `null` 等效项是空字符串 `value`。 Blazor 框架处理 `null` 到空字符串之间的转换，以便双向绑定到 `<select>` 的值。

::: moniker-end

::: moniker range="< aspnetcore-5.0"

尝试双向绑定到 `<select>` 的值时，Blazor 框架不会自动处理 `null` 到空字符串之间的转换。 有关详细信息，请参阅[修复 `<select>` 到 null 值的绑定 (dotnet/aspnetcore #23221)](https://github.com/dotnet/aspnetcore/pull/23221)。

::: moniker-end

## <a name="validation-support"></a>验证支持

<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件使用数据注释将验证支持附加到级联的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext>。 使用数据注释启用对验证的支持需要此显式手势。 若要使用不同于数据注释的验证系统，请用自定义实现替换 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator>。 可在以下参考源中检查 ASP.NET Core 的实现[`DataAnnotationsValidator`](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/DataAnnotationsValidator.cs)/[`AddDataAnnotationsValidation`](https://github.com/dotnet/AspNetCore/blob/master/src/Components/Forms/src/EditContextDataAnnotationsExtensions.cs)： 前面的参考源链接提供了来自存储库 `master` 分支的代码，该分支表示产品单元当前对 ASP.NET Core 下一版本的开发。 若要为其他版本选择分支，请使用 GitHub 分支选择器（例如 `release/3.1`）。

Blazor 执行两种类型的验证：

* 当用户从某个字段中跳出时，将执行 *字段验证* 。 在字段验证期间，<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件将报告的所有验证结果与该字段相关联。
* 当用户提交窗体时，将执行 *模型验证* 。 在模型验证期间，<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件尝试根据验证结果报告的成员名称来确定字段。 与单个成员无关的验证结果将与模型而不是字段相关联。

### <a name="validation-summary-and-validation-message-components"></a>验证摘要和验证消息组件

<xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 组件用于汇总所有验证消息，这与[验证摘要标记帮助程序](xref:mvc/views/working-with-forms#the-validation-summary-tag-helper)类似：

```razor
<ValidationSummary />
```

使用 `Model` 参数输出特定模型的验证消息：
  
```razor
<ValidationSummary Model="@starship" />
```

<xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> 组件用于显示特定字段的验证消息，这与[验证消息标记帮助程序](xref:mvc/views/working-with-forms#the-validation-message-tag-helper)类似。 使用 `For` 属性和一个为模型属性命名的 Lambda 表达式来指定要验证的字段：

```razor
<ValidationMessage For="@(() => starship.MaximumAccommodation)" />
```

<xref:Microsoft.AspNetCore.Components.Forms.ValidationMessage%601> 和 <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 组件支持任意属性。 与某个组件参数不匹配的所有属性都将添加到生成的 `<div>` 或 `<ul>` 元素中。

在应用的样式表（`wwwroot/css/app.css` 或 `wwwroot/css/site.css`）中控制验证消息的样式。 默认 `validation-message` 类将验证消息的文本颜色设置为红色：

```css
.validation-message {
    color: red;
}
```

### <a name="custom-validation-attributes"></a>自定义验证属性

当使用[自定义验证属性](xref:mvc/models/validation#custom-attributes)时，为确保验证结果与字段正确关联，请在创建 <xref:System.ComponentModel.DataAnnotations.ValidationResult> 时传递验证上下文的 <xref:System.ComponentModel.DataAnnotations.ValidationContext.MemberName>：

```csharp
using System;
using System.ComponentModel.DataAnnotations;

private class CustomValidator : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, 
        ValidationContext validationContext)
    {
        ...

        return new ValidationResult("Validation message to user.",
            new[] { validationContext.MemberName });
    }
}
```

> [!NOTE]
> <xref:System.ComponentModel.DataAnnotations.ValidationContext.GetService%2A?displayProperty=nameWithType> 为 `null`。 不支持在 `IsValid` 方法中注入用于验证的服务。

::: moniker range=">= aspnetcore-5.0"

## <a name="custom-validation-class-attributes"></a>自定义验证类属性

与 CSS 框架集成时，自定义验证类名称非常有用，例如[启动](https://getbootstrap.com/)。 若要指定自定义验证类名称，请创建从 `FieldCssClassProvider` 派生的类，并在 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 实例上设置该类：

```csharp
var editContext = new EditContext(model);
editContext.SetFieldCssClassProvider(new MyFieldClassProvider());

...

private class MyFieldClassProvider : FieldCssClassProvider
{
    public override string GetFieldCssClass(EditContext editContext, 
        in FieldIdentifier fieldIdentifier)
    {
        var isValid = !editContext.GetValidationMessages(fieldIdentifier).Any();

        return isValid ? "good field" : "bad field";
    }
}
```

::: moniker-end

### <a name="no-locblazor-data-annotations-validation-package"></a>Blazor 数据注释验证包

[`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) 是使用 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件填补验证经验空白的包。 该包目前处于 *试验阶段* 。

> [!NOTE]
> [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) 包具有最新版本的候选发布 ([Nuget.org](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation))。此时继续使用实验性候选发布包。 在将来的版本中，包的程序集可能会移动到框架或运行时。 请观看[公告 GitHub 存储库](https://github.com/aspnet/Announcements)、[dotnet/aspnetcore GitHub 存储库](https://github.com/dotnet/aspnetcore)或本主题部分，获取进一步更新。

### <a name="compareproperty-attribute"></a>[CompareProperty] 属性

<xref:System.ComponentModel.DataAnnotations.CompareAttribute> 不适用于 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 组件，因为它不会将验证结果与特定成员关联。 这可能会导致字段级验证的行为与提交时整个模型的验证行为不一致。 [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) 试验性包引入了一个附加的验证属性 `ComparePropertyAttribute`，它可以克服这些限制。 在 Blazor 应用中，`[CompareProperty]` 可直接替代 [`[Compare]`](xref:System.ComponentModel.DataAnnotations.CompareAttribute) 特性。

### <a name="nested-models-collection-types-and-complex-types"></a>嵌套模型、集合类型和复杂类型

Blazor 支持结合使用数据注释和内置的 <xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 来验证窗体输入。 但是，<xref:Microsoft.AspNetCore.Components.Forms.DataAnnotationsValidator> 仅验证绑定到窗体的模型的顶级属性（不包括集合类型或复杂类型的属性）。

若要验证绑定模型的整个对象图（包括集合类型和复杂类型的属性），请使用试验性 [`Microsoft.AspNetCore.Components.DataAnnotations.Validation`](https://www.nuget.org/packages/Microsoft.AspNetCore.Components.DataAnnotations.Validation) 包提供的 `ObjectGraphDataAnnotationsValidator`：

```razor
<EditForm Model="@model" OnValidSubmit="@HandleValidSubmit">
    <ObjectGraphDataAnnotationsValidator />
    ...
</EditForm>
```

用 `[ValidateComplexType]` 注释模型属性。 在以下模型类中，`ShipDescription` 类包含附加数据注释，用于在将模型绑定到窗体时进行验证：

`Starship.cs`:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class Starship
{
    ...

    [ValidateComplexType]
    public ShipDescription ShipDescription { get; set; } = 
        new ShipDescription();

    ...
}
```

`ShipDescription.cs`:

```csharp
using System;
using System.ComponentModel.DataAnnotations;

public class ShipDescription
{
    [Required]
    [StringLength(40, ErrorMessage = "Description too long (40 char).")]
    public string ShortDescription { get; set; }

    [Required]
    [StringLength(240, ErrorMessage = "Description too long (240 char).")]
    public string LongDescription { get; set; }
}
```

### <a name="enable-the-submit-button-based-on-form-validation"></a>基于窗体验证启用提交按钮

若要基于窗体验证启用和禁用提交按钮，请执行以下操作：

* 使用窗体的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 在初始化组件时分配模型。
* 在上下文的 <xref:Microsoft.AspNetCore.Components.Forms.EditContext.OnFieldChanged> 回调中验证窗体，以启用和禁用提交按钮。
* 解除挂接 `Dispose` 方法中的事件处理程序。 有关详细信息，请参阅 <xref:blazor/components/lifecycle#component-disposal-with-idisposable>。

> [!NOTE]
> 使用 <xref:Microsoft.AspNetCore.Components.Forms.EditContext> 时，也不要将 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> 分配给 <xref:Microsoft.AspNetCore.Components.Forms.EditForm>。

```razor
@implements IDisposable

<EditForm EditContext="@editContext">
    <DataAnnotationsValidator />
    <ValidationSummary />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private Starship starship = new Starship() { ProductionDate = DateTime.UtcNow };
    private bool formInvalid = true;
    private EditContext editContext;

    protected override void OnInitialized()
    {
        editContext = new EditContext(starship);
        editContext.OnFieldChanged += HandleFieldChanged;
    }

    private void HandleFieldChanged(object sender, FieldChangedEventArgs e)
    {
        formInvalid = !editContext.Validate();
        StateHasChanged();
    }

    public void Dispose()
    {
        editContext.OnFieldChanged -= HandleFieldChanged;
    }
}
```

在上面的示例中，如果满足以下条件，则将 `formInvalid` 设置为 `false`：

* 窗体已预加载有效的默认值。
* 你希望在加载窗体时启用提交按钮。

上述方法的副作用是在用户与任何一个字段进行交互后，<xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 组件都会填充无效的字段。 可通过以下方式之一解决此情况：

* 不在窗体上使用 <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 组件。
* 选择提交按钮时，使 <xref:Microsoft.AspNetCore.Components.Forms.ValidationSummary> 组件可见（例如，在 `HandleValidSubmit` 方法中）。

```razor
<EditForm EditContext="@editContext" OnValidSubmit="@HandleValidSubmit">
    <DataAnnotationsValidator />
    <ValidationSummary style="@displaySummary" />

    ...

    <button type="submit" disabled="@formInvalid">Submit</button>
</EditForm>

@code {
    private string displaySummary = "display:none";

    ...

    private void HandleValidSubmit()
    {
        displaySummary = "display:block";
    }
}
```

## <a name="troubleshoot"></a>疑难解答

> InvalidOperationException：EditForm 需要 Model 参数或 EditContext 参数，但不能同时需要这两个参数。

确认 <xref:Microsoft.AspNetCore.Components.Forms.EditForm> 是否有 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> 或 <xref:Microsoft.AspNetCore.Components.Forms.EditContext>。 不要对同一窗体使用这两者。

在将 <xref:Microsoft.AspNetCore.Components.Forms.EditForm.Model> 分配到窗体时，请确认模型类型是否已实例化，如下面的示例所示：

```csharp
private ExampleModel exampleModel = new ExampleModel();
```

::: moniker range=">= aspnetcore-5.0"

## <a name="additional-resources"></a>其他资源

* <xref:blazor/file-uploads>

::: moniker-end
