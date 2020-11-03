---
title: 具有截止时间和取消功能的可靠的 gRPC 服务
author: jamesnk
description: 了解如何在 .NET 中创建具有截止时间和取消功能的可靠的 gRPC 服务。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 09/07/2020
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
uid: grpc/deadlines-cancellation
ms.openlocfilehash: 59b737a032ea37a554ad5ddd0f4d44e4e1602d88
ms.sourcegitcommit: a07f83b00db11f32313045b3492e5d1ff83c4437
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/15/2020
ms.locfileid: "90594363"
---
# <a name="reliable-grpc-services-with-deadlines-and-cancellation"></a>具有截止时间和取消功能的可靠的 gRPC 服务

作者：[James Newton-King](https://twitter.com/jamesnk)

截止时间和取消功能是 gRPC 客户端用来中止进行中调用的功能。 本文介绍截止时间和取消功能非常重要的原因，以及如何在 .NET gRPC 应用中使用它们。

## <a name="deadlines"></a>截止时间

截止时间功能让 gRPC 客户端可以指定等待调用完成的时间。 超过截止时间时，将取消调用。 设定一个截止时间非常重要，因为它将提供调用可运行的最长时间。 它能阻止异常运行的服务持续运行并耗尽服务器资源。 截止时间对于构建可靠应用非常有效，应该进行配置。

截止时间配置：

* 在进行调用时，使用 `CallOptions.Deadline` 配置截止时间。
* 没有截止时间默认值。 gRPC 调用没有时间限制，除非指定了截止时间。
* 截止时间指的是超过截止时间的 UTC 时间。 例如，`DateTime.UtcNow.AddSeconds(5)` 是从现在起 5 秒的截止时间。
* 如果使用的是过去或当前的时间，则调用将立即超过截止时间。
* 截止时间随 gRPC 调用发送到服务，并由客户端和服务独立跟踪。 gRPC 调用可能在一台计算机上完成，但当响应返回给客户端时，已超过了截止时间。

如果超过了截止时间，客户端和服务将有不同的行为：

* 客户端将立即中止基础的 HTTP 请求并引发 `DeadlineExceeded` 错误。 客户端应用可以选择捕获错误并向用户显示超时消息。
* 在服务器上，将中止正在执行的 HTTP 请求，并引发 [ServerCallContext.CancellationToken](xref:System.Threading.CancellationToken)。 尽管中止了 HTTP 请求，gRPC 调用仍将继续在服务器上运行，直到方法完成。 将取消令牌传递给异步方法，使其随调用一同被取消，这非常重要。 例如，向异步数据库查询和 HTTP 请求传递取消令牌。 传递取消令牌让取消的调用可以在服务器上快速完成，并为其他调用释放资源。

配置 `CallOptions.Deadline` 以设置 gRPC 调用的截止时间：

[!code-csharp[](~/grpc/deadlines-cancellation/deadline-client.cs?highlight=7,12)]

在 gRPC 服务中使用 `ServerCallContext.CancellationToken`：

[!code-csharp[](~/grpc/deadlines-cancellation/deadline-server.cs?highlight=5)]

### <a name="propagating-deadlines"></a>传播截止时间

从正在执行的 gRPC 服务进行 gRPC 调用时，应传播截止时间。 例如：

1. 客户端应用调用带有截止时间的 `FrontendService.GetUser`。
2. `FrontendService` 调用 `UserService.GetUser`。 客户端指定的截止时间应随新的 gRPC 调用进行指定。
3. `UserService.GetUser` 接收截止时间。 如果超过了客户端应用的截止时间，将正确超时。

调用上下文将使用 `ServerCallContext.Deadline` 提供截止时间：

[!code-csharp[](~/grpc/deadlines-cancellation/deadline-propagate.cs?highlight=7)]

手动传播截止时间可能会很繁琐。 截止时间需要传递给每个调用，很容易不小心错过。 gRPC 客户端工厂提供自动解决方案。 指定 `EnableCallContextPropagation`：

* 自动将截止时间和取消令牌传播到子调用。
* 这是确保复杂的嵌套 gRPC 场景始终传播截止时间和取消的一种极佳方式。

[!code-csharp[](~/grpc/deadlines-cancellation/clientfactory-propagate.cs?highlight=6)]

有关详细信息，请参阅 <xref:grpc/clientfactory#deadline-and-cancellation-propagation>。

## <a name="cancellation"></a>取消

取消功能让 gRPC 客户端可以取消不再需要的长期运行的调用。 例如，当用户访问网站上的页面时，将启动流实时更新的 gRPC 调用。 当用户离开页面时，应取消流。

通过传递带有 [CallOptions.CancellationToken](xref:System.Threading.CancellationToken) 的取消令牌，或通过调用 `Dispose`，可以在客户端中取消 gRPC 调用。

[!code-csharp[](~/grpc/deadlines-cancellation/cancellation-client.cs?highlight=19)]

可以取消的 gRPC 服务应：
* 将 `ServerCallContext.CancellationToken` 传递给异步方法。 取消异步方法可以使服务器上的调用快速完成。
* 将取消令牌传播给子调用。 传播取消令牌可确保子调用与其父级一起取消。 [gRPC 客户端工厂](xref:grpc/clientfactory)和 `EnableCallContextPropagation()` 自动传播取消令牌。

## <a name="additional-resources"></a>其他资源

* <xref:grpc/client>
* <xref:grpc/clientfactory>
