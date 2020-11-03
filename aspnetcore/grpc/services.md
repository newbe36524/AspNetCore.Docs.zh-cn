---
title: 创建 gRPC 服务和方法
author: jamesnk
description: 了解如何创建 gRPC 服务和方法。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 08/25/2020
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
uid: grpc/services
ms.openlocfilehash: c4d37556a3345d275f45c537a40908c5966fe015
ms.sourcegitcommit: ecae2aa432628b9181d1fa11037c231c7dd56c9e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92113616"
---
# <a name="create-grpc-services-and-methods"></a>创建 gRPC 服务和方法

作者：[James Newton-King](https://twitter.com/jamesnk)

本文档介绍如何以 C# 创建 gRPC 服务和方法。 主题包括：

* 如何在 .proto 文件中定义服务和方法。
* 使用 gRPC C# 工具生成的代码。
* 实现 gRPC 服务和方法。

## <a name="create-new-grpc-services"></a>创建新的 gRPC 服务

[使用 C# 的 gRPC 服务](xref:grpc/basics)介绍了 gRPC 的 API 开发协定优先方法。 在 .proto 文件中定义服务和消息。 然后，C# 工具从 .proto 文件生成代码。 对于服务器端资产，将为每个服务生成一个抽象基类型，同时为所有消息生成类。

以下 .proto 文件：

* 定义 `Greeter` 服务。
* `Greeter` 服务定义 `SayHello` 调用。
* `SayHello` 发送 `HelloRequest` 消息并接收 `HelloReply` 消息

```protobuf
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```

C# 工具生成 C# `GreeterBase` 基类型：

```csharp
public abstract partial class GreeterBase
{
    public virtual Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        throw new RpcException(new Status(StatusCode.Unimplemented, ""));
    }
}

public class HelloRequest
{
    public string Name { get; set; }
}

public class HelloReply
{
    public string Message { get; set; }
}
```

默认情况下，生成的 `GreeterBase` 不执行任何操作。 它的虚拟 `SayHello` 方法会将 `UNIMPLEMENTED` 错误返回到调用它的任何客户端。 为了使服务有用，应用必须创建 `GreeterBase` 的具体实现：

```csharp
public class GreeterService : GreeterBase
{
    public override Task<HelloReply> SayHello(HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloRequest { Message = $"Hello {request.Name}" });
    }
}
```

服务实现已注册到应用。 如果服务由 ASP.NET Core gRPC 托管，则应使用 `MapGrpcService` 方法将其添加到路由管道。

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
```

有关更多信息，请参见 <xref:grpc/aspnetcore> 。

## <a name="implement-grpc-methods"></a>实现 gRPC 方法

gRPC 服务可以有不同类型的方法。 服务发送和接收消息的方式取决于所定义的方法的类型。 gRPC 方法类型如下：

* 一元
* 服务器流式处理
* 客户端流式处理
* 双向流式处理

流式处理调用是使用 `stream` 关键字在 .proto 文件中指定的。 `stream` 可以放置在调用的请求消息和/或响应消息中。

```protobuf
syntax = "proto3";

service ExampleService {
  // Unary
  rpc UnaryCall (ExampleRequest) returns (ExampleResponse);

  // Server streaming
  rpc StreamingFromServer (ExampleRequest) returns (stream ExampleResponse);

  // Client streaming
  rpc StreamingFromClient (stream ExampleRequest) returns (ExampleResponse);

  // Bi-directional streaming
  rpc StreamingBothWays (stream ExampleRequest) returns (stream ExampleResponse);
}
```

每个调用类型都有不同的方法签名。 在具体实现中替代从抽象基本服务类型生成的方法，可确保使用正确的参数和返回类型。

### <a name="unary-method"></a>一元方法

一元方法以参数的形式获取请求消息，并返回响应。 返回响应时，一元调用完成。

```csharp
public override Task<ExampleResponse> UnaryCall(ExampleRequest request,
    ServerCallContext context)
{
    var response = new ExampleResponse();
    return Task.FromResult(response);
}
```

一元调用与 [Web API 控制器上的操作](xref:web-api/index)最为相似。 gRPC 方法与操作的一个重要区别是，gRPC 方法无法将请求的某些部分绑定到不同的方法参数。 对于传入请求数据，gRPC 方法始终有一个消息参数。 通过在请求消息中设置多个值字段，仍可以将多个值发送到 gRPC 服务：

```protobuf
message ExampleRequest {
    int pageIndex = 1;
    int pageSize = 2;
    bool isDescending = 3;
}
```

### <a name="server-streaming-method"></a>服务器流式处理方法

服务器流式处理方法以参数的形式获取请求消息。 由于可以将多个消息流式传输回调用方，因此可使用 `responseStream.WriteAsync` 发送响应消息。 当方法返回时，服务器流式处理调用完成。

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    for (var i = 0; i < 5; i++)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1));
    }
}
```

服务器流式处理方法启动后，客户端无法发送其他消息或数据。 某些流式处理方法设计为永久运行。 对于连续流式处理方法，客户端可以在不再需要调用时将其取消。 当发生取消时，客户端会将信号发送到服务器，并引发 [ServerCallContext.CancellationToken](xref:System.Threading.CancellationToken)。 应在服务器上通过异步方法使用 `CancellationToken` 标记，以实现以下目的：

* 所有异步工作都与流式处理调用一起取消。
* 该方法快速退出。

```csharp
public override async Task StreamingFromServer(ExampleRequest request,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    while (!context.CancellationToken.IsCancellationRequested)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

### <a name="client-streaming-method"></a>客户端流式处理方法

客户端流式处理方法在该方法没有接收消息的情况下启动。 `requestStream` 参数用于从客户端读取消息。 返回响应消息时，客户端流式处理调用完成：

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    while (await requestStream.MoveNext())
    {
        var message = requestStream.Current;
        // ...
    }
    return new ExampleResponse();
}
```

如果使用 C# 8 或更高版本，则可使用 `await foreach` 语法来读取消息。 `IAsyncStreamReader<T>.ReadAllAsync()` 扩展方法读取请求数据流中的所有消息：

```csharp
public override async Task<ExampleResponse> StreamingFromClient(
    IAsyncStreamReader<ExampleRequest> requestStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        // ...
    }
    return new ExampleResponse();
}
```

### <a name="bi-directional-streaming-method"></a>双向流式处理方法

双向流式处理方法在该方法没有接收到消息的情况下启动。 `requestStream` 参数用于从客户端读取消息。 该方法可选择使用 `responseStream.WriteAsync` 发送消息。 当方法返回时，双向流式处理调用完成：

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    await foreach (var message in requestStream.ReadAllAsync())
    {
        await responseStream.WriteAsync(new ExampleResponse());
    }
}
```

前面的代码：

* 发送每个请求的响应。
* 是双向流式处理的基本用法。

可以支持更复杂的方案，例如同时读取请求和发送响应：

```csharp
public override async Task StreamingBothWays(IAsyncStreamReader<ExampleRequest> requestStream,
    IServerStreamWriter<ExampleResponse> responseStream, ServerCallContext context)
{
    // Read requests in a background task.
    var readTask = Task.Run(async () =>
    {
        await foreach (var message in requestStream.ReadAllAsync())
        {
            // Process request.
        }
    });
    
    // Send responses until the client signals that it is complete.
    while (!readTask.IsCompleted)
    {
        await responseStream.WriteAsync(new ExampleResponse());
        await Task.Delay(TimeSpan.FromSeconds(1), context.CancellationToken);
    }
}
```

在双向流式处理方法中，客户端和服务可在任何时间互相发送消息。 双向方法的最佳实现根据需求而有所不同。

## <a name="access-grpc-request-headers"></a>访问 gRPC 请求标头

请求消息并不是客户端将数据发送到 gRPC 服务的唯一方法。 标头值在使用 `ServerCallContext.RequestHeaders` 的服务中可用。

```csharp
public override Task<ExampleResponse> UnaryCall(ExampleRequest request, ServerCallContext context)
{
    var userAgent = context.RequestHeaders.GetValue("user-agent");
    // ...

    return Task.FromResult(new ExampleResponse());
}
```

## <a name="additional-resources"></a>其他资源

* <xref:grpc/basics>
* <xref:grpc/client>
