---
title: 比较 gRPC 服务和 HTTP API
author: jamesnk
description: 了解如何比较 gRPC 和 HTTP API 以及建议方案。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 12/05/2019
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
uid: grpc/comparison
ms.openlocfilehash: 3f0e44bb374214328f589c6ca3952c6d7aab88d8
ms.sourcegitcommit: 9c031530d2e652fe422e786bd43392bc500d622f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/18/2020
ms.locfileid: "90770124"
---
# <a name="compare-grpc-services-with-http-apis"></a>比较 gRPC 服务和 HTTP API

作者：[James Newton-King](https://twitter.com/jamesnk)

本文介绍如何将 [gRPC 服务](https://grpc.io/docs/guides/)与具有 JSON 的 HTTP API（包括 ASP.NET Core [Web API](xref:web-api/index)）进行比较。 用于为应用提供 API 的技术是一个重要选择，与 HTTP API 相比，gRPC 提供独特优势。 本文讨论 gRPC 的优点和缺点，并提供优先于其他技术选择使用 gRPC 的建议方案。

## <a name="high-level-comparison"></a>概括比较

下表对 gRPC 和具有 JSON 的 HTTP API 之间的功能进行了简单比较。

| 功能          | gRPC                                               | 具有 JSON 的 HTTP API           |
| ---------------- | -------------------------------------------------- | ----------------------------- |
| 协定         | 必需 ( *.proto*)                                | 可选 (OpenAPI)            |
| 协议         | HTTP/2                                             | HTTP                          |
| Payload          | [Protobuf（小型，二进制）](#performance)           | JSON（大型，人工可读取）  |
| 规定性 | [严格规范](#strict-specification)      | 宽松。 任何 HTTP 均有效。     |
| 流式处理        | [客户端、服务器，双向](#streaming)       | 客户端、服务器                |
| 浏览器支持  | [无（需要 grpc-web）](#limited-browser-support) | 是                           |
| 安全性         | 传输 (TLS)                                    | 传输 (TLS)               |
| 客户端代码生成 | [是](#code-generation)                      | OpenAPI + 第三方工具 |

## <a name="grpc-strengths"></a>gRPC 优点

### <a name="performance"></a>性能

gRPC 消息使用 [Protobuf](https://developers.google.com/protocol-buffers/docs/overview)（一种高效的二进制消息格式）进行序列化。 Protobuf 在服务器和客户端上可以非常快速地序列化。 Protobuf 序列化产生的有效负载较小，这在移动应用等带宽有限的方案中很重要。

gRPC 专为 HTTP/2（HTTP 的主要版本）而设计，与 HTTP 1.x 相比，HTTP/2 具有巨大性能优势：

* 二进制组帧和压缩。 HTTP/2 协议在发送和接收方面均紧凑且高效。
* 在单个 TCP 连接上多路复用多个 HTTP/2 调用。 多路复用可消除[队头阻塞](https://en.wikipedia.org/wiki/Head-of-line_blocking)。

HTTP/2 不是 gRPC 独占的。 许多请求类型（包括使用 JSON 的 HTTP API）都可以使用 HTTP/2，并受益于其性能改进。

### <a name="code-generation"></a>代码生成

所有 gRPC 框架都为代码生成提供一流支持。 [.proto 文件](https://developers.google.com/protocol-buffers/docs/proto3)是 gRPC 开发的核心文件，它定义 gRPC 服务和消息的协定。 通过此文件，gRPC 框架编码生成服务基类、消息和完整的客户端。

通过在服务器和客户端之间共享 *.proto* 文件，可以端到端生成消息和客户端代码。 客户端的代码生成消除了客户端和服务器上的消息重复，并为你创建强类型客户端。 无需编写客户端可在具有许多服务的应用程序中节省大量开发时间。

### <a name="strict-specification"></a>严格规范

具有 JSON 的 HTTP API 没有正式规范。 开发人员为 URL、HTTP 谓词和响应代码的最佳格式争论不休。

[gRPC 规范](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-HTTP2.md)对 gRPC 服务必须遵循的格式进行了规定。 gRPC 消除了争论并为开发人员节省了时间，因为 gRPC 在各个平台和实现中都是一致的。

### <a name="streaming"></a>流式处理

HTTP/2 为长期实时通信流提供基础。 gRPC 为通过 HTTP/2 进行流式传输提供一流支持。

gRPC 服务支持所有流式传输组合：

* 一元（无流式传输）
* 服务器到客户端流式传输
* 客户端到服务器流式传输
* 双向流式传输

### <a name="deadlinetimeouts-and-cancellation"></a>截止时间/超时和取消

gRPC 允许客户端指定其愿意等待 RPC 完成的时间期限。 [截止时间](https://grpc.io/blog/deadlines)会发送到服务器，如果超过截止时间，服务器可以决定要执行的操作。 例如，服务器可能会在超时后取消正在进行的 gRPC/HTTP/数据库请求。

通过 gRPC 子调用传播截止时间和取消有助于强制执行资源使用限制。

## <a name="grpc-recommended-scenarios"></a>gRPC 建议方案

gRPC 非常适合以下方案：

* 微服务：gRPC 设计用于低延迟和高吞吐量通信。 gRPC 对于效率至关重要的轻量级微服务非常有用。
* 点对点实时通信：gRPC 对双向流式传输提供出色的支持。 gRPC 服务可以实时推送消息而无需轮询。
* 多语言环境：gRPC 工具支持所有常用的开发语言，因此，gRPC 是多语言环境的理想选择。
* 网络受限环境：gRPC 消息使用 Protobuf（一种轻量级消息格式）进行序列化。 gRPC 消息始终小于等效的 JSON 消息。
* **进程间通信 (IPC)** ：IPC 传输（如 Unix 域套接字和命名管道）可与 gRPC 一起用于同一台计算机上的应用间通信。 有关详细信息，请参阅 <xref:grpc/interprocess>。

## <a name="grpc-weaknesses"></a>gRPC 弱点

### <a name="limited-browser-support"></a>浏览器支持受限

当前无法通过浏览器直接调用 gRPC 服务。 gRPC 大量使用 HTTP/2 功能，且没有浏览器在 Web 请求中提供支持 gRPC 客户端所需的控制级别。 例如，浏览器不允许调用方要求使用 HTTP/2，也不提供对 HTTP/2 基础框架的访问。

将 gRPC 引入浏览器应用有两种常见方法：

* [gRPC-Web](https://grpc.io/docs/tutorials/basic/web.html) 是 gRPC 团队的另一项技术，可在浏览器中提供 gRPC 支持。 gRPC-Web 允许浏览器应用从 gRPC 的高性能和低网络使用率获益。 gRPC-Web 并不支持所有 gRPC 功能。 不支持客户端和双向流式传输，并且对服务器流式传输的支持有限。

  .NET Core 对 gRPC-Web 提供支持。 有关详细信息，请参阅 <xref:grpc/browser>。

* 通过使用 [HTTP 元数据](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule)注释 .proto 文件，可以从 gRPC 服务自动创建 RESTful JSON Web API。 这使得应用可以同时支持 gRPC 和 JSON Web API，而无需重复为两者生成单独的服务。

  .NET Core 对从 gRPC 服务创建 JSON Web API 提供了实验性支持。 有关详细信息，请参阅 <xref:grpc/httpapi>。

### <a name="not-human-readable"></a>非人工可读取

HTTP API 请求以文本形式发送，并且可进行人工读取和创建。

默认情况下，gRPC 消息使用 Protobuf 进行编码。 尽管 Protobuf 可以高效地发送和接收，但其二进制格式非人工可读取。 Protobuf 要求在 *.proto* 文件中指定消息接口描述来正确地反序列化。 需要使用其他工具来分析网络上的 Protobuf 有效负载以及手动撰写请求。

[服务器反射](https://github.com/grpc/grpc/blob/master/doc/server-reflection.md)和 [gRPC 命令行工具](https://github.com/grpc/grpc/blob/master/doc/command_line_tool.md)等功能可帮助使用二进制 Protobuf 消息。 此外，Protobuf 消息支持[与 JSON 之间的转换](https://developers.google.com/protocol-buffers/docs/proto3#json)。 内置的 JSON 转换提供在调试时将 Protobuf 消息与人工可读取格式互相转换的高效方法。

## <a name="alternative-framework-scenarios"></a>备用框架方案

在以下方案中，建议使用其他框架取代 gRPC：

* 浏览器可访问的 API：gRPC 在浏览器中未受到完全支持。 gRPC-Web 可以提供浏览器支持，但它具有局限性并引入了服务器代理。
* 广播实时通信：gRPC 支持通过流式传输进行实时通信，但不存在将消息广播到注册连接的概念。 例如，在聊天室方案中，应将新的聊天消息发送到聊天室中的所有客户端，这要求每个 gRPC 调用将新的聊天消息单独流式传输到客户端。 [SignalR](xref:signalr/introduction) 是适用于此方案的框架。 SignalR 具有持久性连接的概念，并内置对广播消息的支持。

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/grpc/grpc-start>
* <xref:grpc/index>
* <xref:grpc/basics>
* <xref:grpc/migration>
