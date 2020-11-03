---
title: ASP.NET Core Blazor 文件上传
author: guardrex
description: 了解如何使用 InputFile 组件在 Blazor 中上传文件。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/29/2020
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
uid: blazor/file-uploads
ms.openlocfilehash: 06d1464cb731a8008362fc911f463e4ff8a37b6b
ms.sourcegitcommit: d1a897ebd89daa05170ac448e4831d327f6b21a8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/01/2020
ms.locfileid: "91606654"
---
# <a name="aspnet-core-no-locblazor-file-uploads"></a>ASP.NET Core Blazor 文件上传

作者：[Daniel Roth](https://github.com/danroth27) 和 [Pranav Krishnamoorthy](https://github.com/pranavkm)

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/file-uploads/samples/)（[如何下载](xref:index#how-to-download-a-sample)）

使用 `InputFile` 组件将浏览器文件数据（包括文件上传）读入 .NET 代码。

> [!WARNING]
> 请始终遵循文件上传安全最佳做法。 有关详细信息，请参阅 <xref:mvc/models/file-uploads#security-considerations>。

## <a name="inputfile-component"></a>`InputFile` 组件

`InputFile` 组件呈现为 `file` 类型的 HTML 输入。

默认情况下，用户选择单个文件。 可添加 `multiple` 属性以允许用户一次上传多个文件。 当用户选择了一个或多个文件时，`InputFile` 组件将触发 `OnChange` 事件并传入 `InputFileChangeEventArgs`，以允许访问所选文件列表和各文件的详细信息。

从用户选择的文件中读取数据：

* 对文件调用 `OpenReadStream`，并从返回的流中进行读取。 有关详细信息，请参阅[文件流](#file-streams)部分。
* 请使用 `ReadAsync`。 默认情况下，`ReadAsync` 只允许读取 524,288 KB (512 KB) 以下的文件。 此限制旨在防止开发人员意外地将大型文件读入到内存中。 如果必须支持较大的文件，则为所需的最大文件大小指定合理的近似值。 避免将传入的文件流直接读入到内存中。 例如，不要将文件字节复制到 <xref:System.IO.MemoryStream>，也不要以字节数组的形式进行读取。 这些方法可能会导致性能和安全问题，尤其是在 Blazor Server 中。 请考虑将文件字节复制到外部存储（如 blob 或磁盘上的文件）。

接收图像文件的组件可以对文件调用 `RequestImageFileAsync` 便利方法，在图像流式传入应用之前，在浏览器的 JavaScript 运行时内调整图像数据的大小。

以下示例演示在组件中上传多个图像文件。 通过 `InputFileChangeEventArgs.GetMultipleFiles`，可以读取多个文件。 请指定希望读取的最大文件数，以防止恶意用户上传的文件数超过应用的预期值。 如果文件上传不支持多个文件，则可以通过 `InputFileChangeEventArgs.File` 读取第一个文件，并且只能读取此文件。

```razor
<h3>Upload PNG images</h3>

<p>
    <InputFile OnChange="@OnInputFileChange" multiple />
</p>

@if (imageDataUrls.Count > 0)
{
    <h4>Images</h4>

    <div class="card" style="width:30rem;">
        <div class="card-body">
            @foreach (var imageDataUrl in imageDataUrls)
            {
                <img class="rounded m-1" src="@imageDataUrl" />
            }
        </div>
    </div>
}

@code {
    IList<string> imageDataUrls = new List<string>();

    private async Task OnInputFileChange(InputFileChangeEventArgs e)
    {
        var maxAllowedFiles = 3;
        var format = "image/png";

        foreach (var imageFile in e.GetMultipleFiles(maxAllowedFiles))
        {
            var resizedImageFile = await imageFile.RequestImageFileAsync(format, 
                100, 100);
            var buffer = new byte[resizedImageFile.Size];
            await resizedImageFile.OpenReadStream().ReadAsync(buffer);
            var imageDataUrl = 
                $"data:{format};base64,{Convert.ToBase64String(buffer)}";
            imageDataUrls.Add(imageDataUrl);
        }
    }
}
```

`IBrowserFile` 会以属性形式返回[浏览器公开的元数据](https://developer.mozilla.org/docs/Web/API/File#Instance_properties)。 此元数据可用于进行初步验证。 有关示例，请参阅 [`FileUpload.razor` 和 `FilePreview.razor` 示例组件](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/blazor/file-uploads/samples/)。

## <a name="file-streams"></a>文件流

在 Blazor WebAssembly 应用中，数据直接流式传入浏览器中的 .NET 代码。

在 Blazor Server 应用中，从流中读取文件时，文件数据通过 SignalR 连接流式传入服务器上的 .NET 代码。 通过 [`Forms.RemoteBrowserFileStreamOptions`](https://github.com/dotnet/aspnetcore/blob/master/src/Components/Web/src/Forms/InputFile/RemoteBrowserFileStreamOptions.cs)，可以配置 Blazor Server 的文件上传特性。

## <a name="additional-resources"></a>其他资源

* <xref:mvc/models/file-uploads#security-considerations>
* <xref:blazor/forms-validation>
