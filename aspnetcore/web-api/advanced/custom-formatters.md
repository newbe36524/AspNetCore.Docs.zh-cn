---
title: ASP.NET Core Web API 中的自定义格式化程序
author: rick-anderson
description: 了解如何为 ASP.NET Core 中的 Web API 创建和使用自定义格式化程序。
ms.author: riande
ms.date: 06/25/2020
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
uid: web-api/advanced/custom-formatters
ms.openlocfilehash: e4d73fdc0db3faeace5d68b3d71718315e68cae3
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93058917"
---
# <a name="custom-formatters-in-aspnet-core-web-api"></a>ASP.NET Core Web API 中的自定义格式化程序

作者： [Kirk Larkin](https://twitter.com/serpent5) 和 [Tom Dykstra](https://github.com/tdykstra)。

ASP.NET Core MVC 使用输入和输出格式化程序支持 Web API 中的数据交换。 [模型绑定](xref:mvc/models/model-binding)使用输入格式化程序。 输出格式化程序用于 [设置响应的格式](xref:web-api/advanced/formatting)。

该框架为 JSON 和 XML 提供内置的输入和输出格式化程序。 它为纯文本提供内置的输出格式化程序，但不为纯文本提供输入格式化程序。

本文展示如何通过创建自定义格式化程序，添加对其他格式的支持。 有关自定义纯文本输入格式化程序的示例，请参阅 GitHub 上的 [TextPlainInputFormatter](https://github.com/aspnet/Entropy/blob/master/samples/Mvc.Formatters/TextPlainInputFormatter.cs) 。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/custom-formatters/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="when-to-use-custom-formatters"></a>何时使用自定义格式化程序

使用自定义格式化程序添加对不是由内置格式化程序处理的内容类型的支持。

## <a name="overview-of-how-to-use-a-custom-formatter"></a>有关如何使用自定义格式化程序的概述

创建自定义格式化程序：

* 若要对发送到客户端的数据进行序列化，请创建输出格式化程序类。
* 若要对从客户端收到的数据进行反序列化，请创建一个输入格式化程序类。
* 将格式化程序类的实例添加 `InputFormatters` 到 `OutputFormatters` 中的和集合 <xref:Microsoft.AspNetCore.Mvc.MvcOptions> 。

## <a name="how-to-create-a-custom-formatter-class"></a>如何创建自定义格式化程序类

若要创建格式化程序，请执行以下操作：

* 从相应的基类中派生类。 该示例应用派生自 <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> 和 <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> 。
* 在构造函数中指定有效的媒体类型和编码。
* 替代 <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.CanReadType%2A> 和 <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter.CanWriteType%2A> 方法。
* 替代 <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter.ReadRequestBodyAsync%2A> 和 `WriteResponseBodyAsync` 方法。

下面的代码演示了 `VcardOutputFormatter` [示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/custom-formatters/samples)中的类：

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_Class)]
  
### <a name="derive-from-the-appropriate-base-class"></a>从相应的基类中派生

对于文本媒体类型 (例如，vCard) ，派生自 <xref:Microsoft.AspNetCore.Mvc.Formatters.TextInputFormatter> 或 <xref:Microsoft.AspNetCore.Mvc.Formatters.TextOutputFormatter> 基类。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ClassDeclaration)]

对于二进制类型，派生自 <xref:Microsoft.AspNetCore.Mvc.Formatters.InputFormatter> 或 <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatter> 基类。

### <a name="specify-valid-media-types-and-encodings"></a>指定有效的媒体类型和编码

在构造函数中，通过添加到 `SupportedMediaTypes` 和 `SupportedEncodings` 集合来指定有效的媒体类型和编码。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_ctor)]

格式化程序类 **不** 能将构造函数注入用于其依赖项。 例如， `ILogger<VcardOutputFormatter>` 不能作为参数添加到构造函数。 若要访问服务，请使用传递给方法的上下文对象。 本文中的代码示例和 [示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/custom-formatters/samples) 演示了如何执行此操作。

### <a name="override-canreadtype-and-canwritetype"></a>重写 CanReadType 和 CanWriteType

通过重写或方法指定要反序列化或序列化的类型 `CanReadType` `CanWriteType` 。 例如，从某一类型创建 vCard 文本， `Contact` 反之亦然。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_CanWriteType)]

#### <a name="the-canwriteresult-method"></a>CanWriteResult 方法

在某些情况下， `CanWriteResult` 必须重写而不是 `CanWriteType` 。 如果满足以下条件，则使用 `CanWriteResult`：

* 操作方法返回模型类。
* 具有可能在运行时返回的派生类。
* 操作返回的派生类必须在运行时已知。

例如，假定操作方法：

* 签名返回一个 `Person` 类型。
* 可以返回 `Student` `Instructor` 派生自的或类型 `Person` 。 

要使格式化程序仅处理 `Student` 对象，请 <xref:Microsoft.AspNetCore.Mvc.Formatters.OutputFormatterCanWriteContext.Object> 在提供给方法的上下文对象中检查的类型 `CanWriteResult` 。 当操作方法返回时 `IActionResult` ：

* 不需要使用 `CanWriteResult` 。
* `CanWriteType`方法接收运行时类型。

<a id="read-write"></a>

### <a name="override-readrequestbodyasync-and-writeresponsebodyasync"></a>重写 ReadRequestBodyAsync 和 WriteResponseBodyAsync

反序列化或序列化在或中执行 `ReadRequestBodyAsync` `WriteResponseBodyAsync` 。 下面的示例演示如何从依赖关系注入容器中获取服务。 无法从构造函数参数获取服务。

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardOutputFormatter.cs?name=snippet_WriteResponseBodyAsync)]

## <a name="how-to-configure-mvc-to-use-a-custom-formatter"></a>如何将 MVC 配置为使用自定义格式化程序

若要使用自定义格式化程序，请将格式化程序类的实例添加到 `InputFormatters` 或 `OutputFormatters` 集合。

::: moniker range=">= aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Startup.cs?name=snippet_ConfigureServices&highlight=5-6)]

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!code-csharp[](custom-formatters/samples/2.x/CustomFormattersSample/Startup.cs?name=mvcoptions&highlight=3-4)]

::: moniker-end

按格式化程序的插入顺序对其进行计算。 第一个优先。

## <a name="the-complete-vcardinputformatter-class"></a>完整 `VcardInputFormatter` 类

下面的代码演示了 `VcardInputFormatter` [示例](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/custom-formatters/samples)中的类：

[!code-csharp[](custom-formatters/samples/3.x/CustomFormattersSample/Formatters/VcardInputFormatter.cs?name=snippet_Class)]

## <a name="test-the-app"></a>测试应用

[运行本文的示例应用程序](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/web-api/advanced/custom-formatters/samples)，它实现基本 vCard 输入和输出格式化程序。 此应用程序读取和写入电子名片，如下所示：

```
BEGIN:VCARD
VERSION:2.1
N:Davolio;Nancy
FN:Nancy Davolio
END:VCARD
```

若要查看 vCard 输出，请运行应用，并将 Get 请求发送到 Accept 标头 `text/vcard` `https://localhost:5001/api/contacts` 。

将 vCard 添加到内存中的联系人集合：

* `Post` `/api/contacts` 使用 Postman 之类的工具将请求发送到。
* 将 `Content-Type` 标头设置为 `text/vcard`。
* 设置 `vCard` 正文中的文本，格式与前面的示例类似。

## <a name="additional-resources"></a>其他资源

* <xref:web-api/advanced/formatting>
* <xref:grpc/dotnet-grpc>
