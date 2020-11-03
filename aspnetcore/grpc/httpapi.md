---
title: 从 gRPC 创建 JSON Web API
author: jamesnk
description: 了解如何为 gRPC 服务创建 JSON HTTP API。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/28/2020
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
uid: grpc/httpapi
ms.openlocfilehash: fa4e7489920338344b78874690e64d4080b5a719
ms.sourcegitcommit: 139c998d37e9f3e3d0e3d72e10dbce8b75957d89
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/07/2020
ms.locfileid: "91805578"
---
# <a name="create-json-web-apis-from-grpc"></a>从 gRPC 创建 JSON Web API

作者：[James Newton-King](https://twitter.com/jamesnk)

> [!IMPORTANT]
> gRPC HTTP API 是一个试验性项目，而不是已提交的产品。 我们希望：
>
> * 测试我们为 gRPC 服务创建 JSON Web API 的方法是否有效。
> * 获取有关此方法是否对 .NET 开发者有用的反馈。
>
> 请[提供反馈](https://github.com/grpc/grpc-dotnet/issues/167)，确保我们生成的内容受到开发者欢迎并能帮助他们提高工作效率。

gRPC 是一种在应用之间进行通信的新方式。 gRPC 使用 HTTP/2、流式传输、Protobuf 和消息协定来创建高性能的实时服务。

gRPC 有一个限制，即不是所有平台都可以使用它。 浏览器并不完全支持 HTTP/2，这使得 REST 和 JSON 成为将数据引入浏览器应用的主要方式。 即使有 gRPC 带来的好处，REST 和 JSON 在新式应用中也发挥着重要作用。 构建 gRPC 和 JSON Web API 给应用开发增加了不必要的开销。

本文讨论如何使用 gRPC 服务创建 JSON Web API。

## <a name="grpc-http-api"></a>gRPC HTTP API

gRPC HTTP API 是为 gRPC 服务创建 RESTful JSON API 的 ASP.NET Core 的实验性扩展。 配置 gRPC HTTP API 后，应用可以使用熟悉的 HTTP 概念调用 gRPC 服务：

* HTTP 谓词
* URL 参数绑定
* JSON 请求/响应

gRPC 仍然可以用来调用服务。

### <a name="usage"></a>使用情况

1. 将包引用添加到 [Microsoft.AspNetCore.Grpc.HttpApi](https://www.nuget.org/packages/Microsoft.AspNetCore.Grpc.HttpApi)。
1. 使用 `AddGrpcHttpApi` 在 *Startup.cs* 中注册服务。
1. 将 [google/api/http.proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/http.proto) 和 [google/api/annotations.proto](https://github.com/aspnet/AspLabs/blob/c1e59cacf7b9606650d6ec38e54fa3a82377f360/src/GrpcHttpApi/sample/Proto/google/api/annotations.proto) 文件添加到你的项目。
1. 用 HTTP 绑定和路由在 .proto 文件中注释 gRPC 方法：

```protobuf
syntax = "proto3";

import "google/api/annotations.proto";

package greet;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {
    option (google.api.http) = {
      get: "/v1/greeter/{name}"
    };
  }
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

`SayHello` gRPC 方法现在可以作为 gRPC+Protobuf 和 HTTP API 调用：

* 请求： `HTTP/1.1 GET /v1/greeter/world`
* 响应： `{ "message": "Hello world" }`

服务器日志显示 HTTP 调用是由 gRPC 服务执行的。 gRPC HTTP API 将传入的 HTTP 请求映射到 gRPC 消息，然后将响应消息转换为 JSON。

```
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET https://localhost:5001/v1/greeter/world
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint 'gRPC - /v1/greeter/{name}'
info: Server.GreeterService[0]
      Sending hello to world
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint 'gRPC - /v1/greeter/{name}'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 1.996ms 200 application/json
```

下面是一个基本示例。 如需了解更多自定义选项，请参见 [HttpRule](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule)。

### <a name="grpc-http-api-vs-grpc-web"></a>gRPC HTTP API 与 gRPC-Web

gRPC HTTP API 和 gRPC-Web 都支持从浏览器调用 gRPC 服务。 但是，它们的操作方式是不同的：

* gRPC-Web 允许浏览器应用通过 gRPC-Web 客户端和 Protobuf 从浏览器调用 gRPC 服务。 gRPC-Web 需要浏览器应用生成 gRPC 客户端，并且具有快速发送小型 Protobuf 消息的优点。
* gRPC HTTP API 允许浏览器应用调用 gRPC 服务，就像它们是使用 JSON 的 RESTful API 一样。 浏览器应用不需要生成 gRPC 客户端或了解 gRPC 的任何信息。

没有为 gRPC HTTP API 创建生成的客户端。 可以使用浏览器 JavaScript API 调用以前的 `Greeter` 服务：

```javascript
var name = nameInput.value;

fetch("/v1/greeter/" + name).then(function (response) {
  response.json().then(function (data) {
    console.log("Result: " + data.message);
  });
});
```

### <a name="experimental-status"></a>试验性状态

gRPC HTTP API 是一个试验。 它不完整，且不受支持。 我们对该技术及其支持应用开发者同时快速创建 gRPC 和 JSON 服务的功能感兴趣。 没有完成 gRPC HTTP API 的承诺。

我们想评估开发者对 gRPC HTTP API 的兴趣。 如果你对 gRPC HTTP API 感兴趣，请[提供反馈](https://github.com/grpc/grpc-dotnet/issues/167)。

## <a name="grpc-gateway"></a>grpc-gateway

[Grpc-gateway](https://grpc-ecosystem.github.io/grpc-gateway/) 是从 grpc 服务创建 RESTful JSON API 的另一种技术。 它使用相同的 .proto 注释将 HTTP 概念映射到 gRPC 服务。

Grpc-gateway 和 gRPC HTTP API 最大的区别是 grpc-gateway 使用代码生成来创建反向代理服务器。 反向代理将 RESTful 调用转换为 gRPC，然后将它们发送到 gRPC 服务。

如需了解 grpc-gateway 的安装和使用，请参阅 [grpc-gateway 文档](https://grpc-ecosystem.github.io/grpc-gateway/docs/usage.html)。

## <a name="additional-resources"></a>其他资源

* [google.api.HttpRule 文档](https://cloud.google.com/service-infrastructure/docs/service-management/reference/rpc/google.api#google.api.HttpRule)
* <xref:grpc/browser>
