---
title: 在 ASP.NET Core 中上传文件
author: rick-anderson
description: 如何在 ASP.NET Core MVC 中使用模型绑定和流式处理上传文件。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 08/21/2020
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
uid: mvc/models/file-uploads
ms.openlocfilehash: 14561bace565c104d0a9c926cad3105c4865e72a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061166"
---
# <a name="upload-files-in-aspnet-core"></a>在 ASP.NET Core 中上传文件

作者： [Steve Smith](https://ardalis.com/) 和 [Rutger 风暴](https://github.com/rutix)

::: moniker range=">= aspnetcore-5.0"

ASP.NET Core 支持使用缓冲的模型绑定（针对较小文件）和无缓冲的流式传输（针对较大文件）上传一个或多个文件。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="security-considerations"></a>安全注意事项

向用户提供向服务器上传文件的功能时，必须格外小心。 攻击者可能会尝试执行以下操作：

* 执行[拒绝服务](/windows-hardware/drivers/ifs/denial-of-service)攻击。
* 上传病毒或恶意软件。
* 以其他方式破坏网络和服务器。

降低成功攻击可能性的安全措施如下：

* 将文件上传到专用文件上传区域，最好是非系统驱动器。 使用专用位置便于对上传的文件实施安全限制。 禁用对文件上传位置的执行权限。&dagger;
* 请勿将上传的文件保存在与应用相同的目录树中  。&dagger;
* 使用应用确定的安全的文件名。 请勿使用用户提供的文件名或上载文件的不受信任的文件名。 &dagger; 显示时，HTML 对不受信任的文件名进行编码。 例如，记录文件名或在 UI 中显示 (Razor 会自动对输出) 进行 HTML 编码。
* 仅允许应用设计规范的已批准文件扩展名。&dagger; <!-- * Check the file format signature to prevent a user from uploading a masqueraded file.&dagger; For example, don't permit a user to upload an *.exe* file with a *.txt* extension. Add this back when we get instructions how to do this.  -->
* 验证是否在服务器上执行了客户端检查。 &dagger; 客户端检查很容易规避。
* 检查已上传文件的大小。 设置大小上限以防止上传大型文件。&dagger;
* 文件不应该被具有相同名称的上传文件覆盖时，先在数据库或物理存储上检查文件名，然后再上传文件。
* **先对上传的内容运行病毒/恶意软件扫描程序，然后再存储文件。**

&dagger;示例应用演示了符合条件的方法。

> [!WARNING]
> 将恶意代码上传到系统通常是执行代码的第一步，这些代码可以：
>
> * 完全获得对系统的控制权限。
> * 重载系统，导致系统崩溃。
> * 泄露用户或系统数据。
> * 将涂鸦应用于公共 UI。
>
> 有关在接受用户文件时减少攻击外围应用的信息，请参阅以下资源：
>
> * [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)（不受限制的文件上传）
> * [Azure 安全性：确保在接受用户文件时采取适当的控制措施](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

有关实现安全措施（包括示例应用中的示例）的详细信息，请参阅[验证](#validation)部分。

## <a name="storage-scenarios"></a>存储方案

常见的文件存储选项有：

* 数据库

  * 对于小型文件上传，数据库通常快于物理存储（文件系统或网络共享）选项。
  * 相对于物理存储选项，数据库通常更为便利，因为检索数据库记录来获取用户数据可同时提供文件内容（如头像图像）。
  * 相对于使用数据存储服务，数据库的成本可能更低。

* 物理存储（文件系统或网络共享）

  * 对于大型文件上传：
    * 数据库限制可能会限制上传的大小。
    * 相对于数据库存储，物理存储通常成本更高。
  * 相对于使用数据存储服务，物理存储的成本可能更低。
  * 应用的进程必须具有存储位置的读写权限。 切勿授予执行权限。 

* 数据存储服务（例如，[Azure Blob 存储](https://azure.microsoft.com/services/storage/blobs/)）

  * 服务通常通过本地解决方案提供提升的可伸缩性和复原能力，而它们往往受单一故障点的影响。
  * 在大型存储基础结构方案中，服务的成本可能更低。

  有关详细信息，请参阅 [快速入门：使用 .net 在对象存储中创建 blob](/azure/storage/blobs/storage-quickstart-blobs-dotnet)。

## <a name="file-upload-scenarios"></a>文件上传方案

缓冲和流式传输是上传文件的两种常见方法。

**缓冲**

整个文件读入 <xref:Microsoft.AspNetCore.Http.IFormFile>，它是文件的 C# 表示形式，用于处理或保存文件。

文件上传所用的资源（磁盘、内存）取决于并发文件上传的数量和大小。 如果应用尝试缓冲过多上传，站点就会在内存或磁盘空间不足时崩溃。 如果文件上传的大小或频率会消耗应用资源，请使用流式传输。

> [!NOTE]
> 会将大于 64 KB 的所有单个缓冲文件从内存移到磁盘的临时文件。

本主题的以下部分介绍了如何缓冲小型文件：

* [物理存储](#upload-small-files-with-buffered-model-binding-to-physical-storage)
* [Database](#upload-small-files-with-buffered-model-binding-to-a-database)

**流式处理**

从多部分请求收到文件，然后应用直接处理或保存它。 流式传输无法显著提高性能。 流式传输可降低上传文件时对内存或磁盘空间的需求。

[通过流式传输上传大型文件](#upload-large-files-with-streaming)部分介绍了如何流式传输大型文件。

### <a name="upload-small-files-with-buffered-model-binding-to-physical-storage"></a>通过缓冲的模型绑定将小型文件上传到物理存储

要上传小文件，请使用多部分窗体或使用 JavaScript 构造 POST 请求。

下面的示例演示 Razor 如何使用页面窗体上传示例应用) 中的单个文件 ( *Pages/BufferedSingleFileUploadPhysical* ：

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
            <span asp-validation-for="FileUpload.FormFile"></span>
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload" />
</form>
```

下面的示例与前面的示例类似，不同之处在于：

* 使用 JavaScript ([Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)) 提交窗体的数据。
* 无验证。

```cshtml
<form action="BufferedSingleFileUploadPhysical/?handler=Upload" 
      enctype="multipart/form-data" onsubmit="AJAXSubmit(this);return false;" 
      method="post">
    <dl>
        <dt>
            <label for="FileUpload_FormFile">File</label>
        </dt>
        <dd>
            <input id="FileUpload_FormFile" type="file" 
                name="FileUpload.FormFile" />
        </dd>
    </dl>

    <input class="btn" type="submit" value="Upload" />

    <div style="margin-top:15px">
        <output name="result"></output>
    </div>
</form>

<script>
  async function AJAXSubmit (oFormElement) {
    var resultElement = oFormElement.elements.namedItem("result");
    const formData = new FormData(oFormElement);

    try {
    const response = await fetch(oFormElement.action, {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      window.location.href = '/';
    }

    resultElement.value = 'Result: ' + response.status + ' ' + 
      response.statusText;
    } catch (error) {
      console.error('Error:', error);
    }
  }
</script>
```

若要使用 JavaScript 为[不支持 Fetch API](https://caniuse.com/#feat=fetch) 的客户端执行窗体发布，请使用以下方法之一：

* 使用 Fetch Polyfill（例如，[window.fetch polyfill (github/fetch)](https://github.com/github/fetch)）。
* 请使用 `XMLHttpRequest`。 例如： 。

  ```javascript
  <script>
    "use strict";

    function AJAXSubmit (oFormElement) {
      var oReq = new XMLHttpRequest();
      oReq.onload = function(e) { 
      oFormElement.elements.namedItem("result").value = 
        'Result: ' + this.status + ' ' + this.statusText;
      };
      oReq.open("post", oFormElement.action);
      oReq.send(new FormData(oFormElement));
    }
  </script>
  ```

为支持文件上传，HTML 窗体必须指定 `multipart/form-data` 的编码类型 (`enctype`)。

要使 `files` 输入元素支持上传多个文件，请在 `<input>` 元素上提供 `multiple` 属性：

```cshtml
<input asp-for="FileUpload.FormFiles" type="file" multiple>
```

上传到服务器的单个文件可使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 接口通过[模型绑定](xref:mvc/models/model-binding)进行访问。 示例应用演示了数据库和物理存储方案的多个缓冲文件上传。

<a name="filename"></a>

> [!WARNING]
> 除了显示和日志记录用途外，请勿使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 的 `FileName` 属性  。 显示或日志记录时，HTML 对文件名进行编码。 攻击者可以提供恶意文件名，包括完整路径或相对路径。 应用程序应：
>
> * 从用户提供的文件名中删除路径。
> * 为 UI 或日志记录保存经 HTML 编码、已删除路径的文件名。
> * 生成新的随机文件名进行存储。
>
> 以下代码可从文件名中删除路径：
>
> ```csharp
> string untrustedFileName = Path.GetFileName(pathName);
> ```
>
> 目前提供的示例未考虑安全注意事项。 以下各节及[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)提供了其他信息：
>
> * [安全注意事项](#security-considerations)
> * [验证](#validation)

使用模型绑定和 <xref:Microsoft.AspNetCore.Http.IFormFile> 上传文件时，操作方法可以接受以下内容：

* 单个 <xref:Microsoft.AspNetCore.Http.IFormFile>。
* 以下任何表示多个文件的集合：
  * <xref:Microsoft.AspNetCore.Http.IFormFileCollection>
  * <xref:System.Collections.IEnumerable>\<<xref:Microsoft.AspNetCore.Http.IFormFile>>
  * [成员列表](xref:System.Collections.Generic.List`1)\<<xref:Microsoft.AspNetCore.Http.IFormFile>>

> [!NOTE]
> 绑定根据名称匹配窗体文件。 例如，`<input type="file" name="formFile">` 中的 HTML `name` 值必须与 C# 参数/属性绑定 (`FormFile`) 匹配。 有关详细信息，请参阅[使名称属性值与 POST 方法的参数名匹配](#match-name-attribute-value-to-parameter-name-of-post-method)部分。

如下示例中：

* 循环访问一个或多个上传的文件。
* 使用 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 返回文件的完整路径，包括文件名称。 
* 使用应用生成的文件名将文件保存到本地文件系统。
* 返回上传的文件的总数量和总大小。

```csharp
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
```

使用 `Path.GetRandomFileName` 生成文件名（不含路径）。 在下面的示例中，从配置获取路径：

```csharp
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
```

传递到  的路径必须包含文件名<xref:System.IO.FileStream>  。 如果未提供文件名，则会在运行时引发 <xref:System.UnauthorizedAccessException>。

使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 技术上传的文件在处理之前会缓冲在内存中或服务器的磁盘中。 在操作方法中，<xref:Microsoft.AspNetCore.Http.IFormFile> 内容可作为 <xref:System.IO.Stream> 访问。 除本地文件系统之外，还可以将文件保存到网络共享或文件存储服务，如 [Azure Blob 存储](/azure/visual-studio/vs-storage-aspnet5-getting-started-blobs)。

若要查看循环访问要上传的多个文件并且使用安全文件名的其他示例，请参阅示例应用中的 Pages/BufferedMultipleFileUploadPhysical.cshtml.cs。 

> [!WARNING]
> 如果在未删除先前临时文件的情况下创建了 65,535 个以上的文件，则 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 将抛出一个 <xref:System.IO.IOException>。 65,535 个文件限制是每个服务器的限制。 有关 Windows 操作系统上的此限制的详细信息，请参阅以下主题中的说明：
>
> * [GetTempFileNameA 函数](/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea#remarks)
> * <xref:System.IO.Path.GetTempFileName*>

### <a name="upload-small-files-with-buffered-model-binding-to-a-database"></a>使用缓冲的模型绑定将小型文件上传到数据库

要使用[实体框架](/ef/core/index)将二进制文件数据存储在数据库中，请在实体上定义 <xref:System.Byte> 数组属性：

```csharp
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}
```

为包括 <xref:Microsoft.AspNetCore.Http.IFormFile> 的类指定页模型属性：

```csharp
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Http.IFormFile> 可以直接用作操作方法参数或绑定模型属性。 前面的示例使用绑定模型属性。

`FileUpload`在 Razor 页面窗体中使用：

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload">
</form>
```

将窗体发布到服务器后，将 <xref:Microsoft.AspNetCore.Http.IFormFile> 复制到流，并将它作为字节数组保存在数据库中。 在下面的示例中，`_dbContext` 存储应用的数据库上下文：

```csharp
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
```

上面的示例与示例应用中演示的方案相似：

* *Pages/BufferedSingleFileUploadDb.cshtml*
* *Pages/BufferedSingleFileUploadDb.cshtml.cs*

> [!WARNING]
> 在关系数据库中存储二进制数据时要格外小心，因为它可能对性能产生不利影响。
>
> 切勿依赖或信任未经验证的 <xref:Microsoft.AspNetCore.Http.IFormFile> 的 `FileName` 属性。 只应将 `FileName` 属性用于显示用途，并且只应在进行 HTML 编码后使用它。
>
> 提供的示例未考虑安全注意事项。 以下各节及[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)提供了其他信息：
>
> * [安全注意事项](#security-considerations)
> * [验证](#validation)

### <a name="upload-large-files-with-streaming"></a>通过流式传输上传大型文件

以下示例演示如何使用 JavaScript 将文件流式传输到控制器操作。 使用自定义筛选器属性生成文件的防伪令牌，并将其传递到客户端 HTTP 头中（而不是在请求正文中传递）。 由于操作方法直接处理上传的数据，所以其他自定义筛选器会禁用窗体模型绑定。 在该操作中，使用 `MultipartReader` 读取窗体的内容，它会读取每个单独的 `MultipartSection`，从而根据需要处理文件或存储内容。 读取多部分节后，该操作会执行自己的模型绑定。

初始页面响应会加载窗体，并 cookie 通过属性) 将防伪标记保存在 (中 `GenerateAntiforgeryTokenCookieAttribute` 。 属性使用 ASP.NET Core 的内置 [防伪支持](xref:security/anti-request-forgery) 来设置 cookie 具有请求令牌的：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/Antiforgery.cs?name=snippet_GenerateAntiforgeryTokenCookieAttribute)]

使用 `DisableFormValueModelBindingAttribute` 禁用模型绑定：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/ModelBinding.cs?name=snippet_DisableFormValueModelBindingAttribute)]

在示例应用中， `GenerateAntiforgeryTokenCookieAttribute` 和 `DisableFormValueModelBindingAttribute` `/StreamedSingleFileUploadDb` `/StreamedSingleFileUploadPhysical` `Startup.ConfigureServices` 使用[ Razor 页面约定](xref:razor-pages/razor-pages-conventions)作为筛选器应用于和的页面应用程序模型：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Startup.cs?name=snippet_AddRazorPages&highlight=7-10,16-19)]

由于模型绑定不读取窗体，因此不绑定从窗体绑定的参数（查询、路由和标头继续运行）。 操作方法直接使用 `Request` 属性。 `MultipartReader` 用于读取每个节。 在 `KeyValueAccumulator` 中存储键值数据。 读取多部分节后，系统会使用 `KeyValueAccumulator` 的内容将窗体数据绑定到模型类型。

使用 EF Core 流式传输到数据库的完整 `StreamingController.UploadDatabase` 方法：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadDatabase)]

`MultipartRequestHelper` (Utilities/MultipartRequestHelper.cs)： 

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Utilities/MultipartRequestHelper.cs)]

流式传输到物理位置的完整 `StreamingController.UploadPhysical` 方法：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadPhysical)]

在示例应用中，由 `FileHelpers.ProcessStreamedFile` 处理验证检查。

## <a name="validation"></a>验证

示例应用的 `FileHelpers` 类演示对缓冲 <xref:Microsoft.AspNetCore.Http.IFormFile> 和流式传输文件上传的多项检查。 有关示例应用如何处理 <xref:Microsoft.AspNetCore.Http.IFormFile> 缓冲文件上传的信息，请参阅 Utilities/FileHelpers.cs 文件中的 `ProcessFormFile` 方法。  有关如何处理流式传输的文件的信息，请参阅同一个文件中的 `ProcessStreamedFile` 方法。

> [!WARNING]
> 示例应用演示的验证处理方法不扫描上传的文件的内容。 在多数生产方案中，会先将病毒/恶意软件扫描程序 API 用于文件，然后再向用户或其他系统提供文件。
>
> 尽管主题示例提供了验证技巧工作示例，但是如果不满足以下情况，请勿在生产应用中实现 `FileHelpers` 类：
>
> * 完全理解此实现。
> * 根据应用的环境和规范修改实现。
>
> **切勿未处理这些要求即随意在应用中实现安全代码。**

### <a name="content-validation"></a>内容验证

**将第三方病毒/恶意软件扫描 API 用于上传的内容** 。

在大容量方案中，在服务器资源上扫描文件较为困难。 若文件扫描导致请求处理性能降低，请考虑将扫描工作卸载到[后台服务](xref:fundamentals/host/hosted-services)，该服务可以是在应用服务器之外的服务器上运行的服务。 通常会将卸载的文件保留在隔离区，直至后台病毒扫描程序检查它们。 文件通过检查时，会将相应的文件移到常规的文件存储位置。 通常在执行这些步骤的同时，会提供指示文件扫描状态的数据库记录。 通过此方法，应用和应用服务器可以持续以响应请求为重点。

### <a name="file-extension-validation"></a>文件扩展名验证

应在允许的扩展名列表中查找上传的文件的扩展名。 例如： 。

```csharp
private string[] permittedExtensions = { ".txt", ".pdf" };

var ext = Path.GetExtension(uploadedFileName).ToLowerInvariant();

if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains(ext))
{
    // The extension is invalid ... discontinue processing the file
}
```

### <a name="file-signature-validation"></a>文件签名验证

文件的签名由文件开头部分中的前几个字节确定。 可以使用这些字节指示扩展名是否与文件内容匹配。 示例应用检查一些常见文件类型的文件签名。 在下面的示例中，在文件上检查 JPEG 图像的文件签名：

```csharp
private static readonly Dictionary<string, List<byte[]>> _fileSignature = 
    new Dictionary<string, List<byte[]>>
{
    { ".jpeg", new List<byte[]>
        {
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 },
        }
    },
};

using (var reader = new BinaryReader(uploadedFileData))
{
    var signatures = _fileSignature[ext];
    var headerBytes = reader.ReadBytes(signatures.Max(m => m.Length));
    
    return signatures.Any(signature => 
        headerBytes.Take(signature.Length).SequenceEqual(signature));
}
```

若要获取其他文件签名，请参阅[文件签名数据库](https://www.filesignatures.net/)和官方文件规范。

### <a name="file-name-security"></a>文件名安全

切勿使用客户端提供的文件名来将文件保存到物理存储。 使用 [Path.GetRandomFileName](xref:System.IO.Path.GetRandomFileName*) 或 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 为文件创建安全的文件名，以创建完整路径（包括文件名）来执行临时存储。

Razor 自动对属性值进行 HTML 编码以便显示。 以下代码安全可用：

```cshtml
@foreach (var file in Model.DatabaseFiles) {
    <tr>
        <td>
            @file.UntrustedName
        </td>
    </tr>
}
```

在之外 Razor ，始终 <xref:System.Net.WebUtility.HtmlEncode*> 根据用户的请求来命名内容。

许多实现都必须包含关于文件是否存在的检查；否则文件会被使用相同名称的文件覆盖。 提供其他逻辑以符合应用的规范。

### <a name="size-validation"></a>大小验证

限制上传的文件的大小。

在示例应用中，文件大小限制为 2 MB（以字节为单位）。 此限制是通过以下文件的 [配置](xref:fundamentals/configuration/index) 提供的 *appsettings.json* ：

```json
{
  "FileSizeLimit": 2097152
}
```

将 `FileSizeLimit` 注入到 `PageModel` 类：

```csharp
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    private readonly long _fileSizeLimit;

    public BufferedSingleFileUploadPhysicalModel(IConfiguration config)
    {
        _fileSizeLimit = config.GetValue<long>("FileSizeLimit");
    }

    ...
}
```

文件大小超出限制时，将拒绝文件：

```csharp
if (formFile.Length > _fileSizeLimit)
{
    // The file is too large ... discontinue processing the file
}
```

### <a name="match-name-attribute-value-to-parameter-name-of-post-method"></a>使名称属性值与 POST 方法的参数名称匹配

在 Razor 发布窗体数据或直接使用 JavaScript 的非窗体中 `FormData` ，在窗体的元素中指定的名称或 `FormData` 必须与控制器的操作中参数的名称匹配。

如下示例中：

* 使用 `<input>` 元素时，将 `name` 属性设置为值 `battlePlans`：

  ```html
  <input type="file" name="battlePlans" multiple>
  ```

* 使用 JavaScript `FormData` 时，将名称设置为值 `battlePlans`：

  ```javascript
  var formData = new FormData();

  for (var file in files) {
    formData.append("battlePlans", file, file.name);
  }
  ```

将匹配的名称用于 C# 方法的参数 (`battlePlans`)：

* 对于名为的页 Razor 页面处理程序方法 `Upload` ：

  ```csharp
  public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> battlePlans)
  ```

* 对于 MVC POST 控制器操作方法：

  ```csharp
  public async Task<IActionResult> Post(List<IFormFile> battlePlans)
  ```

## <a name="server-and-app-configuration"></a>服务器和应用程序配置

### <a name="multipart-body-length-limit"></a>多部分正文长度限制

<xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> 设置每个多部分正文的长度限制。 分析超出此限制的窗体部分时，会引发 <xref:System.IO.InvalidDataException>。 默认值为 134,217,728 (128 MB)。 使用 `Startup.ConfigureServices` 中的 <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> 设置自定义此限制：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        // Set the limit to 256 MB
        options.MultipartBodyLengthLimit = 268435456;
    });
}
```

使用 <xref:Microsoft.AspNetCore.Mvc.RequestFormLimitsAttribute> 设置单个页面或操作的 <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit>。

在 Razor 页面应用中，将筛选器应用于中的 [约定](xref:razor-pages/razor-pages-conventions) `Startup.ConfigureServices` ：

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model.Filters.Add(
                new RequestFormLimitsAttribute()
                {
                    // Set the limit to 256 MB
                    MultipartBodyLengthLimit = 268435456
                });
});
```

在 Razor 页面应用或 MVC 应用中，将筛选器应用于页面模型或操作方法：

```csharp
// Set the limit to 256 MB
[RequestFormLimits(MultipartBodyLengthLimit = 268435456)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="kestrel-maximum-request-body-size"></a>Kestrel 最大请求正文大小

对于 Kestrel 托管的应用，默认的最大请求正文大小为 30,000,000 个字节，约为 28.6 MB。 使用 [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) Kestrel 服务器选项自定义限制：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel((context, options) =>
            {
                // Handle requests up to 50 MB
                options.Limits.MaxRequestBodySize = 52428800;
            })
            .UseStartup<Startup>();
        });
```

使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 设置单个页面或操作的 [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size)。

在 Razor 页面应用中，将筛选器应用于中的 [约定](xref:razor-pages/razor-pages-conventions) `Startup.ConfigureServices` ：

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model =>
            {
                // Handle requests up to 50 MB
                model.Filters.Add(
                    new RequestSizeLimitAttribute(52428800));
            });
});
```

在 Razor 页面应用或 MVC 应用中，将筛选器应用于页面处理程序类或操作方法：

```csharp
// Handle requests up to 50 MB
[RequestSizeLimit(52428800)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

`RequestSizeLimitAttribute`还可以使用 [`@attribute`](xref:mvc/views/razor#attribute) Razor 指令应用：

```cshtml
@attribute [RequestSizeLimitAttribute(52428800)]
```

### <a name="other-kestrel-limits"></a>其他 Kestrel 限制

其他 Kestrel 限制可能适用于 Kestrel 托管的应用：

* [客户端最大连接数](xref:fundamentals/servers/kestrel#maximum-client-connections)
* [请求和响应数据速率](xref:fundamentals/servers/kestrel#minimum-request-body-data-rate)

### <a name="iis"></a>IIS

默认请求限制 (`maxAllowedContentLength`) 为30000000字节，即约 28.6 MB。 自定义文件中的限制 `web.config` 。 在下面的示例中，将限制设置为 50 MB (52428800 字节) ：

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

此 `maxAllowedContentLength` 设置仅适用于 IIS。 有关详细信息，请参阅[请求 `<requestLimits>` 限制](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/)。

## <a name="troubleshoot"></a>疑难解答

以下是上传文件时遇到的一些常见问题及其可能的解决方案。

### <a name="not-found-error-when-deployed-to-an-iis-server"></a>部署到 IIS 服务器时出现“找不到”错误

以下错误表示上传的文件超过服务器配置的内容长度：

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

有关详细信息，请参阅 [IIS](#iis) 部分。

### <a name="connection-failure"></a>连接失败

连接错误和重置服务器连接可能表示上传的文件超出 Kestrel 的最大请求正文大小。 有关详细信息，请参阅 [Kestrel 最大请求正文大小](#kestrel-maximum-request-body-size)部分。 可能还需要调整 Kestrel 客户端连接限制。

### <a name="null-reference-exception-with-iformfile"></a>IFormFile 的空引用异常

如果控制器正在接受使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 上传的文件，但该值为 `null`，请确认 HTML 窗体指定的 `multipart/form-data` 值是否为 `enctype`。 如果未在 `<form>` 元素上设置此属性，则不会发生文件上传，并且任何绑定的 <xref:Microsoft.AspNetCore.Http.IFormFile> 参数都为 `null`。 此外，请确认[窗体数据中的上传命名是否与应用的命名相匹配](#match-name-attribute-value-to-parameter-name-of-post-method)。

### <a name="stream-was-too-long"></a>数据流太长

本主题中的示例依赖于 <xref:System.IO.MemoryStream> 来保存已上传的文件的内容。 `MemoryStream` 的大小限制为 `int.MaxValue`。 如果应用的文件上传方案要求保存大于 50 MB 的文件内容，请使用另一种方法，该方法不依赖单个 `MemoryStream` 来保存已上传文件的内容。

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

ASP.NET Core 支持使用缓冲的模型绑定（针对较小文件）和无缓冲的流式传输（针对较大文件）上传一个或多个文件。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="security-considerations"></a>安全注意事项

向用户提供向服务器上传文件的功能时，必须格外小心。 攻击者可能会尝试执行以下操作：

* 执行[拒绝服务](/windows-hardware/drivers/ifs/denial-of-service)攻击。
* 上传病毒或恶意软件。
* 以其他方式破坏网络和服务器。

降低成功攻击可能性的安全措施如下：

* 将文件上传到专用文件上传区域，最好是非系统驱动器。 使用专用位置便于对上传的文件实施安全限制。 禁用对文件上传位置的执行权限。&dagger;
* 请勿将上传的文件保存在与应用相同的目录树中  。&dagger;
* 使用应用确定的安全的文件名。 请勿使用用户提供的文件名或上载文件的不受信任的文件名。 &dagger; 显示时，HTML 对不受信任的文件名进行编码。 例如，记录文件名或在 UI 中显示 (Razor 会自动对输出) 进行 HTML 编码。
* 仅允许应用设计规范的已批准文件扩展名。&dagger; <!-- * Check the file format signature to prevent a user from uploading a masqueraded file.&dagger; For example, don't permit a user to upload an *.exe* file with a *.txt* extension. Add this back when we get instructions how to do this.  -->
* 验证是否在服务器上执行了客户端检查。 &dagger; 客户端检查很容易规避。
* 检查已上传文件的大小。 设置大小上限以防止上传大型文件。&dagger;
* 文件不应该被具有相同名称的上传文件覆盖时，先在数据库或物理存储上检查文件名，然后再上传文件。
* **先对上传的内容运行病毒/恶意软件扫描程序，然后再存储文件。**

&dagger;示例应用演示了符合条件的方法。

> [!WARNING]
> 将恶意代码上传到系统通常是执行代码的第一步，这些代码可以：
>
> * 完全获得对系统的控制权限。
> * 重载系统，导致系统崩溃。
> * 泄露用户或系统数据。
> * 将涂鸦应用于公共 UI。
>
> 有关在接受用户文件时减少攻击外围应用的信息，请参阅以下资源：
>
> * [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)（不受限制的文件上传）
> * [Azure 安全性：确保在接受用户文件时采取适当的控制措施](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

有关实现安全措施（包括示例应用中的示例）的详细信息，请参阅[验证](#validation)部分。

## <a name="storage-scenarios"></a>存储方案

常见的文件存储选项有：

* 数据库

  * 对于小型文件上传，数据库通常快于物理存储（文件系统或网络共享）选项。
  * 相对于物理存储选项，数据库通常更为便利，因为检索数据库记录来获取用户数据可同时提供文件内容（如头像图像）。
  * 相对于使用数据存储服务，数据库的成本可能更低。

* 物理存储（文件系统或网络共享）

  * 对于大型文件上传：
    * 数据库限制可能会限制上传的大小。
    * 相对于数据库存储，物理存储通常成本更高。
  * 相对于使用数据存储服务，物理存储的成本可能更低。
  * 应用的进程必须具有存储位置的读写权限。 切勿授予执行权限。 

* 数据存储服务（例如，[Azure Blob 存储](https://azure.microsoft.com/services/storage/blobs/)）

  * 服务通常通过本地解决方案提供提升的可伸缩性和复原能力，而它们往往受单一故障点的影响。
  * 在大型存储基础结构方案中，服务的成本可能更低。

  有关详细信息，请参阅 [快速入门：使用 .net 在对象存储中创建 blob](/azure/storage/blobs/storage-quickstart-blobs-dotnet)。

## <a name="file-upload-scenarios"></a>文件上传方案

缓冲和流式传输是上传文件的两种常见方法。

**缓冲**

整个文件读入 <xref:Microsoft.AspNetCore.Http.IFormFile>，它是文件的 C# 表示形式，用于处理或保存文件。

文件上传所用的资源（磁盘、内存）取决于并发文件上传的数量和大小。 如果应用尝试缓冲过多上传，站点就会在内存或磁盘空间不足时崩溃。 如果文件上传的大小或频率会消耗应用资源，请使用流式传输。

> [!NOTE]
> 会将大于 64 KB 的所有单个缓冲文件从内存移到磁盘的临时文件。

本主题的以下部分介绍了如何缓冲小型文件：

* [物理存储](#upload-small-files-with-buffered-model-binding-to-physical-storage)
* [Database](#upload-small-files-with-buffered-model-binding-to-a-database)

**流式处理**

从多部分请求收到文件，然后应用直接处理或保存它。 流式传输无法显著提高性能。 流式传输可降低上传文件时对内存或磁盘空间的需求。

[通过流式传输上传大型文件](#upload-large-files-with-streaming)部分介绍了如何流式传输大型文件。

### <a name="upload-small-files-with-buffered-model-binding-to-physical-storage"></a>通过缓冲的模型绑定将小型文件上传到物理存储

要上传小文件，请使用多部分窗体或使用 JavaScript 构造 POST 请求。

下面的示例演示 Razor 如何使用页面窗体上传示例应用) 中的单个文件 ( *Pages/BufferedSingleFileUploadPhysical* ：

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
            <span asp-validation-for="FileUpload.FormFile"></span>
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload" />
</form>
```

下面的示例与前面的示例类似，不同之处在于：

* 使用 JavaScript ([Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)) 提交窗体的数据。
* 无验证。

```cshtml
<form action="BufferedSingleFileUploadPhysical/?handler=Upload" 
      enctype="multipart/form-data" onsubmit="AJAXSubmit(this);return false;" 
      method="post">
    <dl>
        <dt>
            <label for="FileUpload_FormFile">File</label>
        </dt>
        <dd>
            <input id="FileUpload_FormFile" type="file" 
                name="FileUpload.FormFile" />
        </dd>
    </dl>

    <input class="btn" type="submit" value="Upload" />

    <div style="margin-top:15px">
        <output name="result"></output>
    </div>
</form>

<script>
  async function AJAXSubmit (oFormElement) {
    var resultElement = oFormElement.elements.namedItem("result");
    const formData = new FormData(oFormElement);

    try {
    const response = await fetch(oFormElement.action, {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      window.location.href = '/';
    }

    resultElement.value = 'Result: ' + response.status + ' ' + 
      response.statusText;
    } catch (error) {
      console.error('Error:', error);
    }
  }
</script>
```

若要使用 JavaScript 为[不支持 Fetch API](https://caniuse.com/#feat=fetch) 的客户端执行窗体发布，请使用以下方法之一：

* 使用 Fetch Polyfill（例如，[window.fetch polyfill (github/fetch)](https://github.com/github/fetch)）。
* 请使用 `XMLHttpRequest`。 例如： 。

  ```javascript
  <script>
    "use strict";

    function AJAXSubmit (oFormElement) {
      var oReq = new XMLHttpRequest();
      oReq.onload = function(e) { 
      oFormElement.elements.namedItem("result").value = 
        'Result: ' + this.status + ' ' + this.statusText;
      };
      oReq.open("post", oFormElement.action);
      oReq.send(new FormData(oFormElement));
    }
  </script>
  ```

为支持文件上传，HTML 窗体必须指定 `multipart/form-data` 的编码类型 (`enctype`)。

要使 `files` 输入元素支持上传多个文件，请在 `<input>` 元素上提供 `multiple` 属性：

```cshtml
<input asp-for="FileUpload.FormFiles" type="file" multiple>
```

上传到服务器的单个文件可使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 接口通过[模型绑定](xref:mvc/models/model-binding)进行访问。 示例应用演示了数据库和物理存储方案的多个缓冲文件上传。

<a name="filename"></a>

> [!WARNING]
> 除了显示和日志记录用途外，请勿使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 的 `FileName` 属性  。 显示或日志记录时，HTML 对文件名进行编码。 攻击者可以提供恶意文件名，包括完整路径或相对路径。 应用程序应：
>
> * 从用户提供的文件名中删除路径。
> * 为 UI 或日志记录保存经 HTML 编码、已删除路径的文件名。
> * 生成新的随机文件名进行存储。
>
> 以下代码可从文件名中删除路径：
>
> ```csharp
> string untrustedFileName = Path.GetFileName(pathName);
> ```
>
> 目前提供的示例未考虑安全注意事项。 以下各节及[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)提供了其他信息：
>
> * [安全注意事项](#security-considerations)
> * [验证](#validation)

使用模型绑定和 <xref:Microsoft.AspNetCore.Http.IFormFile> 上传文件时，操作方法可以接受以下内容：

* 单个 <xref:Microsoft.AspNetCore.Http.IFormFile>。
* 以下任何表示多个文件的集合：
  * <xref:Microsoft.AspNetCore.Http.IFormFileCollection>
  * <xref:System.Collections.IEnumerable>\<<xref:Microsoft.AspNetCore.Http.IFormFile>>
  * [成员列表](xref:System.Collections.Generic.List`1)\<<xref:Microsoft.AspNetCore.Http.IFormFile>>

> [!NOTE]
> 绑定根据名称匹配窗体文件。 例如，`<input type="file" name="formFile">` 中的 HTML `name` 值必须与 C# 参数/属性绑定 (`FormFile`) 匹配。 有关详细信息，请参阅[使名称属性值与 POST 方法的参数名匹配](#match-name-attribute-value-to-parameter-name-of-post-method)部分。

如下示例中：

* 循环访问一个或多个上传的文件。
* 使用 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 返回文件的完整路径，包括文件名称。 
* 使用应用生成的文件名将文件保存到本地文件系统。
* 返回上传的文件的总数量和总大小。

```csharp
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
```

使用 `Path.GetRandomFileName` 生成文件名（不含路径）。 在下面的示例中，从配置获取路径：

```csharp
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
```

传递到  的路径必须包含文件名<xref:System.IO.FileStream>  。 如果未提供文件名，则会在运行时引发 <xref:System.UnauthorizedAccessException>。

使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 技术上传的文件在处理之前会缓冲在内存中或服务器的磁盘中。 在操作方法中，<xref:Microsoft.AspNetCore.Http.IFormFile> 内容可作为 <xref:System.IO.Stream> 访问。 除本地文件系统之外，还可以将文件保存到网络共享或文件存储服务，如 [Azure Blob 存储](/azure/visual-studio/vs-storage-aspnet5-getting-started-blobs)。

若要查看循环访问要上传的多个文件并且使用安全文件名的其他示例，请参阅示例应用中的 Pages/BufferedMultipleFileUploadPhysical.cshtml.cs。 

> [!WARNING]
> 如果在未删除先前临时文件的情况下创建了 65,535 个以上的文件，则 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 将抛出一个 <xref:System.IO.IOException>。 65,535 个文件限制是每个服务器的限制。 有关 Windows 操作系统上的此限制的详细信息，请参阅以下主题中的说明：
>
> * [GetTempFileNameA 函数](/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea#remarks)
> * <xref:System.IO.Path.GetTempFileName*>

### <a name="upload-small-files-with-buffered-model-binding-to-a-database"></a>使用缓冲的模型绑定将小型文件上传到数据库

要使用[实体框架](/ef/core/index)将二进制文件数据存储在数据库中，请在实体上定义 <xref:System.Byte> 数组属性：

```csharp
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}
```

为包括 <xref:Microsoft.AspNetCore.Http.IFormFile> 的类指定页模型属性：

```csharp
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Http.IFormFile> 可以直接用作操作方法参数或绑定模型属性。 前面的示例使用绑定模型属性。

`FileUpload`在 Razor 页面窗体中使用：

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload">
</form>
```

将窗体发布到服务器后，将 <xref:Microsoft.AspNetCore.Http.IFormFile> 复制到流，并将它作为字节数组保存在数据库中。 在下面的示例中，`_dbContext` 存储应用的数据库上下文：

```csharp
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
```

上面的示例与示例应用中演示的方案相似：

* *Pages/BufferedSingleFileUploadDb.cshtml*
* *Pages/BufferedSingleFileUploadDb.cshtml.cs*

> [!WARNING]
> 在关系数据库中存储二进制数据时要格外小心，因为它可能对性能产生不利影响。
>
> 切勿依赖或信任未经验证的 <xref:Microsoft.AspNetCore.Http.IFormFile> 的 `FileName` 属性。 只应将 `FileName` 属性用于显示用途，并且只应在进行 HTML 编码后使用它。
>
> 提供的示例未考虑安全注意事项。 以下各节及[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)提供了其他信息：
>
> * [安全注意事项](#security-considerations)
> * [验证](#validation)

### <a name="upload-large-files-with-streaming"></a>通过流式传输上传大型文件

以下示例演示如何使用 JavaScript 将文件流式传输到控制器操作。 使用自定义筛选器属性生成文件的防伪令牌，并将其传递到客户端 HTTP 头中（而不是在请求正文中传递）。 由于操作方法直接处理上传的数据，所以其他自定义筛选器会禁用窗体模型绑定。 在该操作中，使用 `MultipartReader` 读取窗体的内容，它会读取每个单独的 `MultipartSection`，从而根据需要处理文件或存储内容。 读取多部分节后，该操作会执行自己的模型绑定。

初始页面响应会加载窗体，并 cookie 通过属性) 将防伪标记保存在 (中 `GenerateAntiforgeryTokenCookieAttribute` 。 属性使用 ASP.NET Core 的内置 [防伪支持](xref:security/anti-request-forgery) 来设置 cookie 具有请求令牌的：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/Antiforgery.cs?name=snippet_GenerateAntiforgeryTokenCookieAttribute)]

使用 `DisableFormValueModelBindingAttribute` 禁用模型绑定：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Filters/ModelBinding.cs?name=snippet_DisableFormValueModelBindingAttribute)]

在示例应用中， `GenerateAntiforgeryTokenCookieAttribute` 和 `DisableFormValueModelBindingAttribute` `/StreamedSingleFileUploadDb` `/StreamedSingleFileUploadPhysical` `Startup.ConfigureServices` 使用[ Razor 页面约定](xref:razor-pages/razor-pages-conventions)作为筛选器应用于和的页面应用程序模型：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Startup.cs?name=snippet_AddRazorPages&highlight=7-10,16-19)]

由于模型绑定不读取窗体，因此不绑定从窗体绑定的参数（查询、路由和标头继续运行）。 操作方法直接使用 `Request` 属性。 `MultipartReader` 用于读取每个节。 在 `KeyValueAccumulator` 中存储键值数据。 读取多部分节后，系统会使用 `KeyValueAccumulator` 的内容将窗体数据绑定到模型类型。

使用 EF Core 流式传输到数据库的完整 `StreamingController.UploadDatabase` 方法：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadDatabase)]

`MultipartRequestHelper` (Utilities/MultipartRequestHelper.cs)： 

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Utilities/MultipartRequestHelper.cs)]

流式传输到物理位置的完整 `StreamingController.UploadPhysical` 方法：

[!code-csharp[](file-uploads/samples/3.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadPhysical)]

在示例应用中，由 `FileHelpers.ProcessStreamedFile` 处理验证检查。

## <a name="validation"></a>验证

示例应用的 `FileHelpers` 类演示对缓冲 <xref:Microsoft.AspNetCore.Http.IFormFile> 和流式传输文件上传的多项检查。 有关示例应用如何处理 <xref:Microsoft.AspNetCore.Http.IFormFile> 缓冲文件上传的信息，请参阅 Utilities/FileHelpers.cs 文件中的 `ProcessFormFile` 方法。  有关如何处理流式传输的文件的信息，请参阅同一个文件中的 `ProcessStreamedFile` 方法。

> [!WARNING]
> 示例应用演示的验证处理方法不扫描上传的文件的内容。 在多数生产方案中，会先将病毒/恶意软件扫描程序 API 用于文件，然后再向用户或其他系统提供文件。
>
> 尽管主题示例提供了验证技巧工作示例，但是如果不满足以下情况，请勿在生产应用中实现 `FileHelpers` 类：
>
> * 完全理解此实现。
> * 根据应用的环境和规范修改实现。
>
> **切勿未处理这些要求即随意在应用中实现安全代码。**

### <a name="content-validation"></a>内容验证

**将第三方病毒/恶意软件扫描 API 用于上传的内容** 。

在大容量方案中，在服务器资源上扫描文件较为困难。 若文件扫描导致请求处理性能降低，请考虑将扫描工作卸载到[后台服务](xref:fundamentals/host/hosted-services)，该服务可以是在应用服务器之外的服务器上运行的服务。 通常会将卸载的文件保留在隔离区，直至后台病毒扫描程序检查它们。 文件通过检查时，会将相应的文件移到常规的文件存储位置。 通常在执行这些步骤的同时，会提供指示文件扫描状态的数据库记录。 通过此方法，应用和应用服务器可以持续以响应请求为重点。

### <a name="file-extension-validation"></a>文件扩展名验证

应在允许的扩展名列表中查找上传的文件的扩展名。 例如： 。

```csharp
private string[] permittedExtensions = { ".txt", ".pdf" };

var ext = Path.GetExtension(uploadedFileName).ToLowerInvariant();

if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains(ext))
{
    // The extension is invalid ... discontinue processing the file
}
```

### <a name="file-signature-validation"></a>文件签名验证

文件的签名由文件开头部分中的前几个字节确定。 可以使用这些字节指示扩展名是否与文件内容匹配。 示例应用检查一些常见文件类型的文件签名。 在下面的示例中，在文件上检查 JPEG 图像的文件签名：

```csharp
private static readonly Dictionary<string, List<byte[]>> _fileSignature = 
    new Dictionary<string, List<byte[]>>
{
    { ".jpeg", new List<byte[]>
        {
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 },
        }
    },
};

using (var reader = new BinaryReader(uploadedFileData))
{
    var signatures = _fileSignature[ext];
    var headerBytes = reader.ReadBytes(signatures.Max(m => m.Length));
    
    return signatures.Any(signature => 
        headerBytes.Take(signature.Length).SequenceEqual(signature));
}
```

若要获取其他文件签名，请参阅[文件签名数据库](https://www.filesignatures.net/)和官方文件规范。

### <a name="file-name-security"></a>文件名安全

切勿使用客户端提供的文件名来将文件保存到物理存储。 使用 [Path.GetRandomFileName](xref:System.IO.Path.GetRandomFileName*) 或 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 为文件创建安全的文件名，以创建完整路径（包括文件名）来执行临时存储。

Razor 自动对属性值进行 HTML 编码以便显示。 以下代码安全可用：

```cshtml
@foreach (var file in Model.DatabaseFiles) {
    <tr>
        <td>
            @file.UntrustedName
        </td>
    </tr>
}
```

在之外 Razor ，始终 <xref:System.Net.WebUtility.HtmlEncode*> 根据用户的请求来命名内容。

许多实现都必须包含关于文件是否存在的检查；否则文件会被使用相同名称的文件覆盖。 提供其他逻辑以符合应用的规范。

### <a name="size-validation"></a>大小验证

限制上传的文件的大小。

在示例应用中，文件大小限制为 2 MB（以字节为单位）。 此限制是通过以下文件的 [配置](xref:fundamentals/configuration/index) 提供的 *appsettings.json* ：

```json
{
  "FileSizeLimit": 2097152
}
```

将 `FileSizeLimit` 注入到 `PageModel` 类：

```csharp
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    private readonly long _fileSizeLimit;

    public BufferedSingleFileUploadPhysicalModel(IConfiguration config)
    {
        _fileSizeLimit = config.GetValue<long>("FileSizeLimit");
    }

    ...
}
```

文件大小超出限制时，将拒绝文件：

```csharp
if (formFile.Length > _fileSizeLimit)
{
    // The file is too large ... discontinue processing the file
}
```

### <a name="match-name-attribute-value-to-parameter-name-of-post-method"></a>使名称属性值与 POST 方法的参数名称匹配

在 Razor 发布窗体数据或直接使用 JavaScript 的非窗体中 `FormData` ，在窗体的元素中指定的名称或 `FormData` 必须与控制器的操作中参数的名称匹配。

如下示例中：

* 使用 `<input>` 元素时，将 `name` 属性设置为值 `battlePlans`：

  ```html
  <input type="file" name="battlePlans" multiple>
  ```

* 使用 JavaScript `FormData` 时，将名称设置为值 `battlePlans`：

  ```javascript
  var formData = new FormData();

  for (var file in files) {
    formData.append("battlePlans", file, file.name);
  }
  ```

将匹配的名称用于 C# 方法的参数 (`battlePlans`)：

* 对于名为的页 Razor 页面处理程序方法 `Upload` ：

  ```csharp
  public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> battlePlans)
  ```

* 对于 MVC POST 控制器操作方法：

  ```csharp
  public async Task<IActionResult> Post(List<IFormFile> battlePlans)
  ```

## <a name="server-and-app-configuration"></a>服务器和应用程序配置

### <a name="multipart-body-length-limit"></a>多部分正文长度限制

<xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> 设置每个多部分正文的长度限制。 分析超出此限制的窗体部分时，会引发 <xref:System.IO.InvalidDataException>。 默认值为 134,217,728 (128 MB)。 使用 `Startup.ConfigureServices` 中的 <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> 设置自定义此限制：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        // Set the limit to 256 MB
        options.MultipartBodyLengthLimit = 268435456;
    });
}
```

使用 <xref:Microsoft.AspNetCore.Mvc.RequestFormLimitsAttribute> 设置单个页面或操作的 <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit>。

在 Razor 页面应用中，将筛选器应用于中的 [约定](xref:razor-pages/razor-pages-conventions) `Startup.ConfigureServices` ：

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model.Filters.Add(
                new RequestFormLimitsAttribute()
                {
                    // Set the limit to 256 MB
                    MultipartBodyLengthLimit = 268435456
                });
});
```

在 Razor 页面应用或 MVC 应用中，将筛选器应用于页面模型或操作方法：

```csharp
// Set the limit to 256 MB
[RequestFormLimits(MultipartBodyLengthLimit = 268435456)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="kestrel-maximum-request-body-size"></a>Kestrel 最大请求正文大小

对于 Kestrel 托管的应用，默认的最大请求正文大小为 30,000,000 个字节，约为 28.6 MB。 使用 [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) Kestrel 服务器选项自定义限制：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel((context, options) =>
            {
                // Handle requests up to 50 MB
                options.Limits.MaxRequestBodySize = 52428800;
            })
            .UseStartup<Startup>();
        });
```

使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 设置单个页面或操作的 [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size)。

在 Razor 页面应用中，将筛选器应用于中的 [约定](xref:razor-pages/razor-pages-conventions) `Startup.ConfigureServices` ：

```csharp
services.AddRazorPages(options =>
{
    options.Conventions
        .AddPageApplicationModelConvention("/FileUploadPage",
            model =>
            {
                // Handle requests up to 50 MB
                model.Filters.Add(
                    new RequestSizeLimitAttribute(52428800));
            });
});
```

在 Razor 页面应用或 MVC 应用中，将筛选器应用于页面处理程序类或操作方法：

```csharp
// Handle requests up to 50 MB
[RequestSizeLimit(52428800)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

`RequestSizeLimitAttribute`还可以使用 [`@attribute`](xref:mvc/views/razor#attribute) Razor 指令应用：

```cshtml
@attribute [RequestSizeLimitAttribute(52428800)]
```

### <a name="other-kestrel-limits"></a>其他 Kestrel 限制

其他 Kestrel 限制可能适用于 Kestrel 托管的应用：

* [客户端最大连接数](xref:fundamentals/servers/kestrel#maximum-client-connections)
* [请求和响应数据速率](xref:fundamentals/servers/kestrel#minimum-request-body-data-rate)

### <a name="iis"></a>IIS

默认请求限制 (`maxAllowedContentLength`) 为30000000字节，即约 28.6 MB。 自定义文件中的限制 `web.config` 。 在下面的示例中，将限制设置为 50 MB (52428800 字节) ：

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

此 `maxAllowedContentLength` 设置仅适用于 IIS。 有关详细信息，请参阅[请求 `<requestLimits>` 限制](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/)。

通过在中设置来增加 HTTP 请求的最大请求正文大小 <xref:Microsoft.AspNetCore.Builder.IISServerOptions.MaxRequestBodySize%2A?displayProperty=nameWithType> `Startup.ConfigureServices` 。 在下面的示例中，将限制设置为 50 MB (52428800 字节) ：

```csharp
services.Configure<IISServerOptions>(options =>
{
    options.MaxRequestBodySize = 52428800;
});
```

有关详细信息，请参阅 <xref:host-and-deploy/iis/index#iis-options>。

## <a name="troubleshoot"></a>疑难解答

以下是上传文件时遇到的一些常见问题及其可能的解决方案。

### <a name="not-found-error-when-deployed-to-an-iis-server"></a>部署到 IIS 服务器时出现“找不到”错误

以下错误表示上传的文件超过服务器配置的内容长度：

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

有关详细信息，请参阅 [IIS](#iis) 部分。

### <a name="connection-failure"></a>连接失败

连接错误和重置服务器连接可能表示上传的文件超出 Kestrel 的最大请求正文大小。 有关详细信息，请参阅 [Kestrel 最大请求正文大小](#kestrel-maximum-request-body-size)部分。 可能还需要调整 Kestrel 客户端连接限制。

### <a name="null-reference-exception-with-iformfile"></a>IFormFile 的空引用异常

如果控制器正在接受使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 上传的文件，但该值为 `null`，请确认 HTML 窗体指定的 `multipart/form-data` 值是否为 `enctype`。 如果未在 `<form>` 元素上设置此属性，则不会发生文件上传，并且任何绑定的 <xref:Microsoft.AspNetCore.Http.IFormFile> 参数都为 `null`。 此外，请确认[窗体数据中的上传命名是否与应用的命名相匹配](#match-name-attribute-value-to-parameter-name-of-post-method)。

### <a name="stream-was-too-long"></a>数据流太长

本主题中的示例依赖于 <xref:System.IO.MemoryStream> 来保存已上传的文件的内容。 `MemoryStream` 的大小限制为 `int.MaxValue`。 如果应用的文件上传方案要求保存大于 50 MB 的文件内容，请使用另一种方法，该方法不依赖单个 `MemoryStream` 来保存已上传文件的内容。

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core 支持使用缓冲的模型绑定（针对较小文件）和无缓冲的流式传输（针对较大文件）上传一个或多个文件。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="security-considerations"></a>安全注意事项

向用户提供向服务器上传文件的功能时，必须格外小心。 攻击者可能会尝试执行以下操作：

* 执行[拒绝服务](/windows-hardware/drivers/ifs/denial-of-service)攻击。
* 上传病毒或恶意软件。
* 以其他方式破坏网络和服务器。

降低成功攻击可能性的安全措施如下：

* 将文件上传到专用文件上传区域，最好是非系统驱动器。 使用专用位置便于对上传的文件实施安全限制。 禁用对文件上传位置的执行权限。&dagger;
* 请勿将上传的文件保存在与应用相同的目录树中  。&dagger;
* 使用应用确定的安全的文件名。 请勿使用用户提供的文件名或上载文件的不受信任的文件名。 &dagger; 显示时，HTML 对不受信任的文件名进行编码。 例如，记录文件名或在 UI 中显示 (Razor 会自动对输出) 进行 HTML 编码。
* 仅允许应用设计规范的已批准文件扩展名。&dagger; <!-- * Check the file format signature to prevent a user from uploading a masqueraded file.&dagger; For example, don't permit a user to upload an *.exe* file with a *.txt* extension. Add this back when we get instructions how to do this.  -->
* 验证是否在服务器上执行了客户端检查。 &dagger; 客户端检查很容易规避。
* 检查已上传文件的大小。 设置大小上限以防止上传大型文件。&dagger;
* 文件不应该被具有相同名称的上传文件覆盖时，先在数据库或物理存储上检查文件名，然后再上传文件。
* **先对上传的内容运行病毒/恶意软件扫描程序，然后再存储文件。**

&dagger;示例应用演示了符合条件的方法。

> [!WARNING]
> 将恶意代码上传到系统通常是执行代码的第一步，这些代码可以：
>
> * 完全获得对系统的控制权限。
> * 重载系统，导致系统崩溃。
> * 泄露用户或系统数据。
> * 将涂鸦应用于公共 UI。
>
> 有关在接受用户文件时减少攻击外围应用的信息，请参阅以下资源：
>
> * [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)（不受限制的文件上传）
> * [Azure 安全性：确保在接受用户文件时采取适当的控制措施](/azure/security/azure-security-threat-modeling-tool-input-validation#controls-users)

有关实现安全措施（包括示例应用中的示例）的详细信息，请参阅[验证](#validation)部分。

## <a name="storage-scenarios"></a>存储方案

常见的文件存储选项有：

* 数据库

  * 对于小型文件上传，数据库通常快于物理存储（文件系统或网络共享）选项。
  * 相对于物理存储选项，数据库通常更为便利，因为检索数据库记录来获取用户数据可同时提供文件内容（如头像图像）。
  * 相对于使用数据存储服务，数据库的成本可能更低。

* 物理存储（文件系统或网络共享）

  * 对于大型文件上传：
    * 数据库限制可能会限制上传的大小。
    * 相对于数据库存储，物理存储通常成本更高。
  * 相对于使用数据存储服务，物理存储的成本可能更低。
  * 应用的进程必须具有存储位置的读写权限。 切勿授予执行权限。 

* 数据存储服务（例如，[Azure Blob 存储](https://azure.microsoft.com/services/storage/blobs/)）

  * 服务通常通过本地解决方案提供提升的可伸缩性和复原能力，而它们往往受单一故障点的影响。
  * 在大型存储基础结构方案中，服务的成本可能更低。

  有关详细信息，请参阅 [快速入门：使用 .net 在对象存储中创建 blob](/azure/storage/blobs/storage-quickstart-blobs-dotnet)。 此主题说明了 <xref:Microsoft.Azure.Storage.File.CloudFile.UploadFromFileAsync*>，但在处理 <xref:System.IO.Stream> 时，可以使用 <xref:Microsoft.Azure.Storage.File.CloudFile.UploadFromStreamAsync*> 将 <xref:System.IO.FileStream> 保存到 blob 存储。

## <a name="file-upload-scenarios"></a>文件上传方案

缓冲和流式传输是上传文件的两种常见方法。

**缓冲**

整个文件读入 <xref:Microsoft.AspNetCore.Http.IFormFile>，它是文件的 C# 表示形式，用于处理或保存文件。

文件上传所用的资源（磁盘、内存）取决于并发文件上传的数量和大小。 如果应用尝试缓冲过多上传，站点就会在内存或磁盘空间不足时崩溃。 如果文件上传的大小或频率会消耗应用资源，请使用流式传输。

> [!NOTE]
> 会将大于 64 KB 的所有单个缓冲文件从内存移到磁盘的临时文件。

本主题的以下部分介绍了如何缓冲小型文件：

* [物理存储](#upload-small-files-with-buffered-model-binding-to-physical-storage)
* [Database](#upload-small-files-with-buffered-model-binding-to-a-database)

**流式处理**

从多部分请求收到文件，然后应用直接处理或保存它。 流式传输无法显著提高性能。 流式传输可降低上传文件时对内存或磁盘空间的需求。

[通过流式传输上传大型文件](#upload-large-files-with-streaming)部分介绍了如何流式传输大型文件。

### <a name="upload-small-files-with-buffered-model-binding-to-physical-storage"></a>通过缓冲的模型绑定将小型文件上传到物理存储

要上传小文件，请使用多部分窗体或使用 JavaScript 构造 POST 请求。

下面的示例演示 Razor 如何使用页面窗体上传示例应用) 中的单个文件 ( *Pages/BufferedSingleFileUploadPhysical* ：

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
            <span asp-validation-for="FileUpload.FormFile"></span>
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload" />
</form>
```

下面的示例与前面的示例类似，不同之处在于：

* 使用 JavaScript ([Fetch API](https://developer.mozilla.org/docs/Web/API/Fetch_API)) 提交窗体的数据。
* 无验证。

```cshtml
<form action="BufferedSingleFileUploadPhysical/?handler=Upload" 
      enctype="multipart/form-data" onsubmit="AJAXSubmit(this);return false;" 
      method="post">
    <dl>
        <dt>
            <label for="FileUpload_FormFile">File</label>
        </dt>
        <dd>
            <input id="FileUpload_FormFile" type="file" 
                name="FileUpload.FormFile" />
        </dd>
    </dl>

    <input class="btn" type="submit" value="Upload" />

    <div style="margin-top:15px">
        <output name="result"></output>
    </div>
</form>

<script>
  async function AJAXSubmit (oFormElement) {
    var resultElement = oFormElement.elements.namedItem("result");
    const formData = new FormData(oFormElement);

    try {
    const response = await fetch(oFormElement.action, {
      method: 'POST',
      body: formData
    });

    if (response.ok) {
      window.location.href = '/';
    }

    resultElement.value = 'Result: ' + response.status + ' ' + 
      response.statusText;
    } catch (error) {
      console.error('Error:', error);
    }
  }
</script>
```

若要使用 JavaScript 为[不支持 Fetch API](https://caniuse.com/#feat=fetch) 的客户端执行窗体发布，请使用以下方法之一：

* 使用 Fetch Polyfill（例如，[window.fetch polyfill (github/fetch)](https://github.com/github/fetch)）。
* 请使用 `XMLHttpRequest`。 例如： 。

  ```javascript
  <script>
    "use strict";

    function AJAXSubmit (oFormElement) {
      var oReq = new XMLHttpRequest();
      oReq.onload = function(e) { 
      oFormElement.elements.namedItem("result").value = 
        'Result: ' + this.status + ' ' + this.statusText;
      };
      oReq.open("post", oFormElement.action);
      oReq.send(new FormData(oFormElement));
    }
  </script>
  ```

为支持文件上传，HTML 窗体必须指定 `multipart/form-data` 的编码类型 (`enctype`)。

要使 `files` 输入元素支持上传多个文件，请在 `<input>` 元素上提供 `multiple` 属性：

```cshtml
<input asp-for="FileUpload.FormFiles" type="file" multiple>
```

上传到服务器的单个文件可使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 接口通过[模型绑定](xref:mvc/models/model-binding)进行访问。 示例应用演示了数据库和物理存储方案的多个缓冲文件上传。

<a name="filename2"></a>

> [!WARNING]
> 除了显示和日志记录用途外，请勿使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 的 `FileName` 属性  。 显示或日志记录时，HTML 对文件名进行编码。 攻击者可以提供恶意文件名，包括完整路径或相对路径。 应用程序应：
>
> * 从用户提供的文件名中删除路径。
> * 为 UI 或日志记录保存经 HTML 编码、已删除路径的文件名。
> * 生成新的随机文件名进行存储。
>
> 以下代码可从文件名中删除路径：
>
> ```csharp
> string untrustedFileName = Path.GetFileName(pathName);
> ```
>
> 目前提供的示例未考虑安全注意事项。 以下各节及[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)提供了其他信息：
>
> * [安全注意事项](#security-considerations)
> * [验证](#validation)

使用模型绑定和 <xref:Microsoft.AspNetCore.Http.IFormFile> 上传文件时，操作方法可以接受以下内容：

* 单个 <xref:Microsoft.AspNetCore.Http.IFormFile>。
* 以下任何表示多个文件的集合：
  * <xref:Microsoft.AspNetCore.Http.IFormFileCollection>
  * <xref:System.Collections.IEnumerable>\<<xref:Microsoft.AspNetCore.Http.IFormFile>>
  * [成员列表](xref:System.Collections.Generic.List`1)\<<xref:Microsoft.AspNetCore.Http.IFormFile>>

> [!NOTE]
> 绑定根据名称匹配窗体文件。 例如，`<input type="file" name="formFile">` 中的 HTML `name` 值必须与 C# 参数/属性绑定 (`FormFile`) 匹配。 有关详细信息，请参阅[使名称属性值与 POST 方法的参数名匹配](#match-name-attribute-value-to-parameter-name-of-post-method)部分。

如下示例中：

* 循环访问一个或多个上传的文件。
* 使用 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 返回文件的完整路径，包括文件名称。 
* 使用应用生成的文件名将文件保存到本地文件系统。
* 返回上传的文件的总数量和总大小。

```csharp
public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> files)
{
    long size = files.Sum(f => f.Length);

    foreach (var formFile in files)
    {
        if (formFile.Length > 0)
        {
            var filePath = Path.GetTempFileName();

            using (var stream = System.IO.File.Create(filePath))
            {
                await formFile.CopyToAsync(stream);
            }
        }
    }

    // Process uploaded files
    // Don't rely on or trust the FileName property without validation.

    return Ok(new { count = files.Count, size });
}
```

使用 `Path.GetRandomFileName` 生成文件名（不含路径）。 在下面的示例中，从配置获取路径：

```csharp
foreach (var formFile in files)
{
    if (formFile.Length > 0)
    {
        var filePath = Path.Combine(_config["StoredFilesPath"], 
            Path.GetRandomFileName());

        using (var stream = System.IO.File.Create(filePath))
        {
            await formFile.CopyToAsync(stream);
        }
    }
}
```

传递到  的路径必须包含文件名<xref:System.IO.FileStream>  。 如果未提供文件名，则会在运行时引发 <xref:System.UnauthorizedAccessException>。

使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 技术上传的文件在处理之前会缓冲在内存中或服务器的磁盘中。 在操作方法中，<xref:Microsoft.AspNetCore.Http.IFormFile> 内容可作为 <xref:System.IO.Stream> 访问。 除本地文件系统之外，还可以将文件保存到网络共享或文件存储服务，如 [Azure Blob 存储](/azure/visual-studio/vs-storage-aspnet5-getting-started-blobs)。

若要查看循环访问要上传的多个文件并且使用安全文件名的其他示例，请参阅示例应用中的 Pages/BufferedMultipleFileUploadPhysical.cshtml.cs。 

> [!WARNING]
> 如果在未删除先前临时文件的情况下创建了 65,535 个以上的文件，则 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 将抛出一个 <xref:System.IO.IOException>。 65,535 个文件限制是每个服务器的限制。 有关 Windows 操作系统上的此限制的详细信息，请参阅以下主题中的说明：
>
> * [GetTempFileNameA 函数](/windows/desktop/api/fileapi/nf-fileapi-gettempfilenamea#remarks)
> * <xref:System.IO.Path.GetTempFileName*>

### <a name="upload-small-files-with-buffered-model-binding-to-a-database"></a>使用缓冲的模型绑定将小型文件上传到数据库

要使用[实体框架](/ef/core/index)将二进制文件数据存储在数据库中，请在实体上定义 <xref:System.Byte> 数组属性：

```csharp
public class AppFile
{
    public int Id { get; set; }
    public byte[] Content { get; set; }
}
```

为包括 <xref:Microsoft.AspNetCore.Http.IFormFile> 的类指定页模型属性：

```csharp
public class BufferedSingleFileUploadDbModel : PageModel
{
    ...

    [BindProperty]
    public BufferedSingleFileUploadDb FileUpload { get; set; }

    ...
}

public class BufferedSingleFileUploadDb
{
    [Required]
    [Display(Name="File")]
    public IFormFile FormFile { get; set; }
}
```

> [!NOTE]
> <xref:Microsoft.AspNetCore.Http.IFormFile> 可以直接用作操作方法参数或绑定模型属性。 前面的示例使用绑定模型属性。

`FileUpload`在 Razor 页面窗体中使用：

```cshtml
<form enctype="multipart/form-data" method="post">
    <dl>
        <dt>
            <label asp-for="FileUpload.FormFile"></label>
        </dt>
        <dd>
            <input asp-for="FileUpload.FormFile" type="file">
        </dd>
    </dl>
    <input asp-page-handler="Upload" class="btn" type="submit" value="Upload">
</form>
```

将窗体发布到服务器后，将 <xref:Microsoft.AspNetCore.Http.IFormFile> 复制到流，并将它作为字节数组保存在数据库中。 在下面的示例中，`_dbContext` 存储应用的数据库上下文：

```csharp
public async Task<IActionResult> OnPostUploadAsync()
{
    using (var memoryStream = new MemoryStream())
    {
        await FileUpload.FormFile.CopyToAsync(memoryStream);

        // Upload the file if less than 2 MB
        if (memoryStream.Length < 2097152)
        {
            var file = new AppFile()
            {
                Content = memoryStream.ToArray()
            };

            _dbContext.File.Add(file);

            await _dbContext.SaveChangesAsync();
        }
        else
        {
            ModelState.AddModelError("File", "The file is too large.");
        }
    }

    return Page();
}
```

上面的示例与示例应用中演示的方案相似：

* *Pages/BufferedSingleFileUploadDb.cshtml*
* *Pages/BufferedSingleFileUploadDb.cshtml.cs*

> [!WARNING]
> 在关系数据库中存储二进制数据时要格外小心，因为它可能对性能产生不利影响。
>
> 切勿依赖或信任未经验证的 <xref:Microsoft.AspNetCore.Http.IFormFile> 的 `FileName` 属性。 只应将 `FileName` 属性用于显示用途，并且只应在进行 HTML 编码后使用它。
>
> 提供的示例未考虑安全注意事项。 以下各节及[示例应用](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/file-uploads/samples/)提供了其他信息：
>
> * [安全注意事项](#security-considerations)
> * [验证](#validation)

### <a name="upload-large-files-with-streaming"></a>通过流式传输上传大型文件

以下示例演示如何使用 JavaScript 将文件流式传输到控制器操作。 使用自定义筛选器属性生成文件的防伪令牌，并将其传递到客户端 HTTP 头中（而不是在请求正文中传递）。 由于操作方法直接处理上传的数据，所以其他自定义筛选器会禁用窗体模型绑定。 在该操作中，使用 `MultipartReader` 读取窗体的内容，它会读取每个单独的 `MultipartSection`，从而根据需要处理文件或存储内容。 读取多部分节后，该操作会执行自己的模型绑定。

初始页面响应会加载窗体，并 cookie 通过属性) 将防伪标记保存在 (中 `GenerateAntiforgeryTokenCookieAttribute` 。 属性使用 ASP.NET Core 的内置 [防伪支持](xref:security/anti-request-forgery) 来设置 cookie 具有请求令牌的：

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Filters/Antiforgery.cs?name=snippet_GenerateAntiforgeryTokenCookieAttribute)]

使用 `DisableFormValueModelBindingAttribute` 禁用模型绑定：

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Filters/ModelBinding.cs?name=snippet_DisableFormValueModelBindingAttribute)]

在示例应用中， `GenerateAntiforgeryTokenCookieAttribute` 和 `DisableFormValueModelBindingAttribute` `/StreamedSingleFileUploadDb` `/StreamedSingleFileUploadPhysical` `Startup.ConfigureServices` 使用[ Razor 页面约定](xref:razor-pages/razor-pages-conventions)作为筛选器应用于和的页面应用程序模型：

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Startup.cs?name=snippet_AddMvc&highlight=8-11,17-20)]

由于模型绑定不读取窗体，因此不绑定从窗体绑定的参数（查询、路由和标头继续运行）。 操作方法直接使用 `Request` 属性。 `MultipartReader` 用于读取每个节。 在 `KeyValueAccumulator` 中存储键值数据。 读取多部分节后，系统会使用 `KeyValueAccumulator` 的内容将窗体数据绑定到模型类型。

使用 EF Core 流式传输到数据库的完整 `StreamingController.UploadDatabase` 方法：

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadDatabase)]

`MultipartRequestHelper` (Utilities/MultipartRequestHelper.cs)： 

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Utilities/MultipartRequestHelper.cs)]

流式传输到物理位置的完整 `StreamingController.UploadPhysical` 方法：

[!code-csharp[](file-uploads/samples/2.x/SampleApp/Controllers/StreamingController.cs?name=snippet_UploadPhysical)]

在示例应用中，由 `FileHelpers.ProcessStreamedFile` 处理验证检查。

## <a name="validation"></a>验证

示例应用的 `FileHelpers` 类演示对缓冲 <xref:Microsoft.AspNetCore.Http.IFormFile> 和流式传输文件上传的多项检查。 有关示例应用如何处理 <xref:Microsoft.AspNetCore.Http.IFormFile> 缓冲文件上传的信息，请参阅 Utilities/FileHelpers.cs 文件中的 `ProcessFormFile` 方法。  有关如何处理流式传输的文件的信息，请参阅同一个文件中的 `ProcessStreamedFile` 方法。

> [!WARNING]
> 示例应用演示的验证处理方法不扫描上传的文件的内容。 在多数生产方案中，会先将病毒/恶意软件扫描程序 API 用于文件，然后再向用户或其他系统提供文件。
>
> 尽管主题示例提供了验证技巧工作示例，但是如果不满足以下情况，请勿在生产应用中实现 `FileHelpers` 类：
>
> * 完全理解此实现。
> * 根据应用的环境和规范修改实现。
>
> **切勿未处理这些要求即随意在应用中实现安全代码。**

### <a name="content-validation"></a>内容验证

**将第三方病毒/恶意软件扫描 API 用于上传的内容** 。

在大容量方案中，在服务器资源上扫描文件较为困难。 若文件扫描导致请求处理性能降低，请考虑将扫描工作卸载到[后台服务](xref:fundamentals/host/hosted-services)，该服务可以是在应用服务器之外的服务器上运行的服务。 通常会将卸载的文件保留在隔离区，直至后台病毒扫描程序检查它们。 文件通过检查时，会将相应的文件移到常规的文件存储位置。 通常在执行这些步骤的同时，会提供指示文件扫描状态的数据库记录。 通过此方法，应用和应用服务器可以持续以响应请求为重点。

### <a name="file-extension-validation"></a>文件扩展名验证

应在允许的扩展名列表中查找上传的文件的扩展名。 例如： 。

```csharp
private string[] permittedExtensions = { ".txt", ".pdf" };

var ext = Path.GetExtension(uploadedFileName).ToLowerInvariant();

if (string.IsNullOrEmpty(ext) || !permittedExtensions.Contains(ext))
{
    // The extension is invalid ... discontinue processing the file
}
```

### <a name="file-signature-validation"></a>文件签名验证

文件的签名由文件开头部分中的前几个字节确定。 可以使用这些字节指示扩展名是否与文件内容匹配。 示例应用检查一些常见文件类型的文件签名。 在下面的示例中，在文件上检查 JPEG 图像的文件签名：

```csharp
private static readonly Dictionary<string, List<byte[]>> _fileSignature = 
    new Dictionary<string, List<byte[]>>
{
    { ".jpeg", new List<byte[]>
        {
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE0 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE2 },
            new byte[] { 0xFF, 0xD8, 0xFF, 0xE3 },
        }
    },
};

using (var reader = new BinaryReader(uploadedFileData))
{
    var signatures = _fileSignature[ext];
    var headerBytes = reader.ReadBytes(signatures.Max(m => m.Length));
    
    return signatures.Any(signature => 
        headerBytes.Take(signature.Length).SequenceEqual(signature));
}
```

若要获取其他文件签名，请参阅[文件签名数据库](https://www.filesignatures.net/)和官方文件规范。

### <a name="file-name-security"></a>文件名安全

切勿使用客户端提供的文件名来将文件保存到物理存储。 使用 [Path.GetRandomFileName](xref:System.IO.Path.GetRandomFileName*) 或 [Path.GetTempFileName](xref:System.IO.Path.GetTempFileName*) 为文件创建安全的文件名，以创建完整路径（包括文件名）来执行临时存储。

Razor 自动对属性值进行 HTML 编码以便显示。 以下代码安全可用：

```cshtml
@foreach (var file in Model.DatabaseFiles) {
    <tr>
        <td>
            @file.UntrustedName
        </td>
    </tr>
}
```

在之外 Razor ，始终 <xref:System.Net.WebUtility.HtmlEncode*> 根据用户的请求来命名内容。

许多实现都必须包含关于文件是否存在的检查；否则文件会被使用相同名称的文件覆盖。 提供其他逻辑以符合应用的规范。

### <a name="size-validation"></a>大小验证

限制上传的文件的大小。

在示例应用中，文件大小限制为 2 MB（以字节为单位）。 此限制是通过以下文件的 [配置](xref:fundamentals/configuration/index) 提供的 *appsettings.json* ：

```json
{
  "FileSizeLimit": 2097152
}
```

将 `FileSizeLimit` 注入到 `PageModel` 类：

```csharp
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    private readonly long _fileSizeLimit;

    public BufferedSingleFileUploadPhysicalModel(IConfiguration config)
    {
        _fileSizeLimit = config.GetValue<long>("FileSizeLimit");
    }

    ...
}
```

文件大小超出限制时，将拒绝文件：

```csharp
if (formFile.Length > _fileSizeLimit)
{
    // The file is too large ... discontinue processing the file
}
```

### <a name="match-name-attribute-value-to-parameter-name-of-post-method"></a>使名称属性值与 POST 方法的参数名称匹配

在 Razor 发布窗体数据或直接使用 JavaScript 的非窗体中 `FormData` ，在窗体的元素中指定的名称或 `FormData` 必须与控制器的操作中参数的名称匹配。

如下示例中：

* 使用 `<input>` 元素时，将 `name` 属性设置为值 `battlePlans`：

  ```html
  <input type="file" name="battlePlans" multiple>
  ```

* 使用 JavaScript `FormData` 时，将名称设置为值 `battlePlans`：

  ```javascript
  var formData = new FormData();

  for (var file in files) {
    formData.append("battlePlans", file, file.name);
  }
  ```

将匹配的名称用于 C# 方法的参数 (`battlePlans`)：

* 对于名为的页 Razor 页面处理程序方法 `Upload` ：

  ```csharp
  public async Task<IActionResult> OnPostUploadAsync(List<IFormFile> battlePlans)
  ```

* 对于 MVC POST 控制器操作方法：

  ```csharp
  public async Task<IActionResult> Post(List<IFormFile> battlePlans)
  ```

## <a name="server-and-app-configuration"></a>服务器和应用程序配置

### <a name="multipart-body-length-limit"></a>多部分正文长度限制

<xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> 设置每个多部分正文的长度限制。 分析超出此限制的窗体部分时，会引发 <xref:System.IO.InvalidDataException>。 默认值为 134,217,728 (128 MB)。 使用 `Startup.ConfigureServices` 中的 <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit> 设置自定义此限制：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<FormOptions>(options =>
    {
        // Set the limit to 256 MB
        options.MultipartBodyLengthLimit = 268435456;
    });
}
```

使用 <xref:Microsoft.AspNetCore.Mvc.RequestFormLimitsAttribute> 设置单个页面或操作的 <xref:Microsoft.AspNetCore.Http.Features.FormOptions.MultipartBodyLengthLimit>。

在 Razor 页面应用中，将筛选器应用于中的 [约定](xref:razor-pages/razor-pages-conventions) `Startup.ConfigureServices` ：

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.Conventions
            .AddPageApplicationModelConvention("/FileUploadPage",
                model.Filters.Add(
                    new RequestFormLimitsAttribute()
                    {
                        // Set the limit to 256 MB
                        MultipartBodyLengthLimit = 268435456
                    });
    })
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

在 Razor 页面应用或 MVC 应用中，将筛选器应用于页面模型或操作方法：

```csharp
// Set the limit to 256 MB
[RequestFormLimits(MultipartBodyLengthLimit = 268435456)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="kestrel-maximum-request-body-size"></a>Kestrel 最大请求正文大小

对于 Kestrel 托管的应用，默认的最大请求正文大小为 30,000,000 个字节，约为 28.6 MB。 使用 [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size) Kestrel 服务器选项自定义限制：

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureKestrel((context, options) =>
        {
            // Handle requests up to 50 MB
            options.Limits.MaxRequestBodySize = 52428800;
        });
```

使用 <xref:Microsoft.AspNetCore.Mvc.RequestSizeLimitAttribute> 设置单个页面或操作的 [MaxRequestBodySize](xref:fundamentals/servers/kestrel#maximum-request-body-size)。

在 Razor 页面应用中，将筛选器应用于中的 [约定](xref:razor-pages/razor-pages-conventions) `Startup.ConfigureServices` ：

```csharp
services.AddMvc()
    .AddRazorPagesOptions(options =>
    {
        options.Conventions
            .AddPageApplicationModelConvention("/FileUploadPage",
                model =>
                {
                    // Handle requests up to 50 MB
                    model.Filters.Add(
                        new RequestSizeLimitAttribute(52428800));
                });
    })
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
```

在 Razor 页面应用或 MVC 应用中，将筛选器应用于页面处理程序类或操作方法：

```csharp
// Handle requests up to 50 MB
[RequestSizeLimit(52428800)]
public class BufferedSingleFileUploadPhysicalModel : PageModel
{
    ...
}
```

### <a name="other-kestrel-limits"></a>其他 Kestrel 限制

其他 Kestrel 限制可能适用于 Kestrel 托管的应用：

* [客户端最大连接数](xref:fundamentals/servers/kestrel#maximum-client-connections)
* [请求和响应数据速率](xref:fundamentals/servers/kestrel#minimum-request-body-data-rate)

### <a name="iis"></a>IIS

默认请求限制 (`maxAllowedContentLength`) 为30000000字节，即约 28.6 MB。 自定义文件中的限制 `web.config` 。 在下面的示例中，将限制设置为 50 MB (52428800 字节) ：

```xml
<system.webServer>
  <security>
    <requestFiltering>
      <requestLimits maxAllowedContentLength="52428800" />
    </requestFiltering>
  </security>
</system.webServer>
```

此 `maxAllowedContentLength` 设置仅适用于 IIS。 有关详细信息，请参阅[请求 `<requestLimits>` 限制](/iis/configuration/system.webServer/security/requestFiltering/requestLimits/)。

通过在中设置来增加 HTTP 请求的最大请求正文大小 <xref:Microsoft.AspNetCore.Builder.IISServerOptions.MaxRequestBodySize%2A?displayProperty=nameWithType> `Startup.ConfigureServices` 。 在下面的示例中，将限制设置为 50 MB (52428800 字节) ：

```csharp
services.Configure<IISServerOptions>(options =>
{
    options.MaxRequestBodySize = 52428800;
});
```

有关详细信息，请参阅 <xref:host-and-deploy/iis/index#iis-options>。

## <a name="troubleshoot"></a>疑难解答

以下是上传文件时遇到的一些常见问题及其可能的解决方案。

### <a name="not-found-error-when-deployed-to-an-iis-server"></a>部署到 IIS 服务器时出现“找不到”错误

以下错误表示上传的文件超过服务器配置的内容长度：

```
HTTP 404.13 - Not Found
The request filtering module is configured to deny a request that exceeds the request content length.
```

有关详细信息，请参阅 [IIS](#iis) 部分。

### <a name="connection-failure"></a>连接失败

连接错误和重置服务器连接可能表示上传的文件超出 Kestrel 的最大请求正文大小。 有关详细信息，请参阅 [Kestrel 最大请求正文大小](#kestrel-maximum-request-body-size)部分。 可能还需要调整 Kestrel 客户端连接限制。

### <a name="null-reference-exception-with-iformfile"></a>IFormFile 的空引用异常

如果控制器正在接受使用 <xref:Microsoft.AspNetCore.Http.IFormFile> 上传的文件，但该值为 `null`，请确认 HTML 窗体指定的 `multipart/form-data` 值是否为 `enctype`。 如果未在 `<form>` 元素上设置此属性，则不会发生文件上传，并且任何绑定的 <xref:Microsoft.AspNetCore.Http.IFormFile> 参数都为 `null`。 此外，请确认[窗体数据中的上传命名是否与应用的命名相匹配](#match-name-attribute-value-to-parameter-name-of-post-method)。

### <a name="stream-was-too-long"></a>数据流太长

本主题中的示例依赖于 <xref:System.IO.MemoryStream> 来保存已上传的文件的内容。 `MemoryStream` 的大小限制为 `int.MaxValue`。 如果应用的文件上传方案要求保存大于 50 MB 的文件内容，请使用另一种方法，该方法不依赖单个 `MemoryStream` 来保存已上传文件的内容。

::: moniker-end


## <a name="additional-resources"></a>其他资源

* [HTTP 连接请求排出](xref:fundamentals/servers/kestrel#http11-request-draining)
* [Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)（不受限制的文件上传）
* [Azure 安全：安全框架：输入验证 |措施](/azure/security/azure-security-threat-modeling-tool-input-validation)
* [Azure 云设计模式：附属密钥模式](/azure/architecture/patterns/valet-key)
