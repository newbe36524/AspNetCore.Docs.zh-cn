---
title: SignalR API 设计注意事项
author: anurse
description: 了解如何 SignalR 在应用版本间设计 api 以实现兼容性。
monikerRange: '>= aspnetcore-2.1'
ms.author: anurse
ms.custom: mvc
ms.date: 11/12/2019
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
uid: signalr/api-design
ms.openlocfilehash: 87665a7950edbc70b664230d2f078598e9dbc0aa
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059645"
---
# <a name="no-locsignalr-api-design-considerations"></a>SignalR API 设计注意事项

作者： [Andrew Stanton](https://twitter.com/anurse)

本文提供了基于生成的 SignalR api 的指南。

## <a name="use-custom-object-parameters-to-ensure-backwards-compatibility"></a>使用自定义对象参数以确保向后兼容性

将参数添加到 SignalR 客户端或服务器上的 (集线器方法) 是一项 *重大更改* 。 这意味着，较旧的客户端/服务器在尝试调用方法时，如果没有适当数量的参数，将会出现错误。 但是，将属性添加到自定义对象参数 **不** 是一项重大更改。 这可用于设计可对客户端或服务器上的更改复原的兼容 Api。

例如，请考虑如下所示的服务器端 API：

[!code-csharp[ParameterBasedOldVersion](api-design/sample/Samples.cs?name=ParameterBasedOldVersion)]

JavaScript 客户端使用调用此方法 `invoke` ，如下所示：

[!code-typescript[CallWithOneParameter](api-design/sample/Samples.ts?name=CallWithOneParameter)]

如果以后将第二个参数添加到服务器方法，较旧的客户端将不提供此参数值。 例如：

[!code-csharp[ParameterBasedNewVersion](api-design/sample/Samples.cs?name=ParameterBasedNewVersion)]

旧客户端尝试调用此方法时，会收到类似于下面的错误：

```
Microsoft.AspNetCore.SignalR.HubException: Failed to invoke 'GetTotalLength' due to an error on the server.
```

在服务器上，你会看到如下所示的日志消息：

```
System.IO.InvalidDataException: Invocation provides 1 argument(s) but target expects 2.
```

旧客户端只发送一个参数，但更新的服务器 API 需要两个参数。 使用自定义对象作为参数可提供更大的灵活性。 让我们重新设计原始 API，以使用自定义对象：

[!code-csharp[ObjectBasedOldVersion](api-design/sample/Samples.cs?name=ObjectBasedOldVersion)]

现在，客户端使用对象调用方法：

[!code-typescript[CallWithObject](api-design/sample/Samples.ts?name=CallWithObject)]

将属性添加到对象，而不是添加参数 `TotalLengthRequest` ：

[!code-csharp[ObjectBasedNewVersion](api-design/sample/Samples.cs?name=ObjectBasedNewVersion&highlight=4,9-13)]

旧客户端发送单个参数时， `Param2` 会保留额外的属性 `null` 。 通过选中 `Param2` `null` 并应用默认值，可以检测由较旧客户端发送的消息。 新的客户端可以发送这两个参数。

[!code-typescript[CallWithObjectNew](api-design/sample/Samples.ts?name=CallWithObjectNew)]

相同的技术适用于在客户端上定义的方法。 你可以从服务器端发送自定义对象：

[!code-csharp[ClientSideObjectBasedOld](api-design/sample/Samples.cs?name=ClientSideObjectBasedOld)]

在客户端，访问 `Message` 属性，而不是使用参数：

[!code-typescript[OnWithObjectOld](api-design/sample/Samples.ts?name=OnWithObjectOld)]

如果以后决定将消息的发件人添加到有效负载，请将属性添加到对象：

[!code-csharp[ClientSideObjectBasedNew](api-design/sample/Samples.cs?name=ClientSideObjectBasedNew&highlight=5)]

较旧的客户端不需要 `Sender` 该值，因此它们将忽略该值。 新的客户端可以通过更新读取新属性来接受它：

[!code-typescript[OnWithObjectNew](api-design/sample/Samples.ts?name=OnWithObjectNew&highlight=2-5)]

在这种情况下，新客户端也可以容忍不提供值的旧服务器 `Sender` 。 由于旧服务器不会提供 `Sender` 值，因此客户端会检查其是否存在，然后再访问它。
