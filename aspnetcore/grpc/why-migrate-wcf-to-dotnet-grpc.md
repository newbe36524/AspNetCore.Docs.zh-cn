---
title: 为何要将 WCF 迁移到 ASP.NET Core gRPC
author: markrendle
description: 本文总结了 ASP.NET Core gRPC 非常适合希望迁移到现代架构和平台的 Windows Communication Foundation (WCF) 开发人员的原因。
monikerRange: '>= aspnetcore-3.0'
ms.author: wpickett
ms.date: 09/02/2020
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
uid: grpc/wcf
ms.openlocfilehash: 811e6037b058b26fcf91063123d04d448a9a28a8
ms.sourcegitcommit: 8fcb08312a59c37e3542e7a67dad25faf5bb8e76
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/11/2020
ms.locfileid: "90012741"
---
# <a name="grpc-for-windows-communication-foundation-wcf-developers"></a>适用于 Windows Communication Foundation (WCF) 开发人员的 gRPC

本文总结了 ASP.NET Core gRPC 非常适合希望迁移到现代架构和平台的 Windows Communication Foundation (WCF) 开发人员的原因。

## <a name="comparison-to-wcf"></a>与 WCF 的比较

尽管 gRPC 的实现和方法不同，但对于 WCF 开发人员来说，通过 gRPC 开发和使用服务的体验应该是直观的。 WCF 和 gRPC 是具有相同目标的 RPC（远程过程调用）框架：

* 使得可以向客户端和服务器在同一个平台上一样进行编码。
* 提供简化的便携网络 API。

尽管声明接口的过程不同，但两个平台都有声明和实现接口的需求。 gRPC 支持的许多 RPC 调用类型很好地映射到 WCF 服务可用的绑定。 有关详细信息和示例，请参阅[将 WCF 解决方案迁移到 gRPC](/dotnet/architecture/grpc-for-wcf-developers/migrate-wcf-to-grpc)。

## <a name="benefits-of-grpc"></a>gRPC 的优点

由于以下原因，gRPC 提供了比其他方法更好的框架。

### <a name="performance"></a>性能

gRPC 使用 HTTP/2。 与 HTTP/1.1、HTTP/2 相比：

* 是更小、更快的二进制协议。
* 对于要分析的计算机，更有效。
* 支持通过单个连接进行多路复用请求。 多路复用允许在一个连接上发送多个请求，而这些请求不会相互阻塞。 在 HTTP/1.1 中，阻塞被称为“队头 (HOL) 阻塞”。

gRPC 使用 Protobuf（一种高效的二进制格式）来序列化消息。 Protobuf 消息能够：
* 快速序列化和反序列化。
* 使用比基于文本的格式更少的带宽。 

gRPC 是一个很好的解决方案，适用于带宽限制的移动设备和网络。

### <a name="interoperability"></a>互操作性

对于所有主要的编程语言和平台，都有 gRPC 工具和库，包括 .NET、Java、Python、Go、C++、Node.js、Swift、Dart、Ruby 以及 PHP。 由于 Protobuf 二进制线格式和每个平台的高效代码生成，开发人员可以构建跨平台、高性能的应用。

### <a name="usability-and-productivity"></a>易用性和工作效率

gRPC 是一个全面的 RPC 解决方案。 它可以跨多种语言和平台一致地工作。 它还提供很好的工具，许多样板代码都是自动生成的。 与 WCF 一样，gRPC 自动生成消息和强类型客户端。 开发人员可以腾出时间来专注于业务逻辑。

### <a name="streaming"></a>流式处理

gRPC 具有完全双向流式传输，它提供与 WCF 的全双工服务类似的功能。 gRPC 流式传输可以通过常规的 Internet 连接、负载均衡器和服务网格运行。

### <a name="deadlines-timeouts-and-cancellation"></a>截止时间、超时和取消

gRPC 允许客户端指定 RPC 完成的最长时间。 如果超过了指定的截止时间，服务器可以独立于客户端取消操作。 截止时间和取消可以通过随后的 gRPC 调用进行传播，以帮助实施资源使用限制。 客户端可以在超过截止时间时停止操作，必要时可以提前停止操作。 例如，客户端可以因为用户交互而停止操作。

### <a name="security"></a>安全性

gRPC 可以使用 TLS 和 HTTP/2 在客户端和服务器之间提供端到端的加密连接。 对客户端证书身份验证的支持进一步提高了客户端和服务器之间的安全性和信任度。

## <a name="grpc-as-a-migration-path-for-wcf-to-net-core-and-net-5"></a>gRPC 作为 WCF 到 .NET Core 和 .NET 5 的迁移路径

.NET Core 和 .NET5 标志着 Microsoft 向希望跨一系列平台提供服务的开发人员提供远程通信解决方案的方式发生了转变。 .NET Core 和 .NET 5 支持[调用 WCF 服务](/dotnet/core/additional-tools/wcf-web-service-reference-guide)，但不会为承载 WCF 提供服务器端支持。

现代化 WCF 应用有两个推荐的路径：

* gRPC 基于现代技术建立，已经成为 RPC 应用开发人员社区中最受欢迎的选择。 从 .NET Core 3.0 开始，现代 .NET 平台就对 gRPC 有很好的支持。 迁移 WCF 服务以使用 gRPC 有助于提供 RPC 功能、性能和现代应用所需的互操作性。

* [CoreWCF](https://github.com/CoreWCF/CoreWCF) 是社区共同努力的成果，可为 .NET Core 和 .NET 5 提供托管 WCF 服务支持。 预览版已经发布，项目正朝着生产就绪的方向努力。 CoreWCF 仅支持 WCF 功能的一个子集，迁移到使用它的 .NET Framework 应用需要代码更改和测试才能成功。 如果应用必须保持与调用 WCF 服务的现有客户端的兼容性，那么 CoreWCF 是一个不错的选择。

## <a name="get-started"></a>入门

有关在适用于 WCF 开发人员的 ASP.NET Core 中构建 gRPC 服务的详细指南，请参阅[适用于 WCF 开发人员的 ASP.NET Core gRPC](/dotnet/architecture/grpc-for-wcf-developers)
