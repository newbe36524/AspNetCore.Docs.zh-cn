---
title: 使用 .NET 客户端调用 gRPC 服务
author: jamesnk
description: 了解如何使用 .NET gRPC 客户端调用 gRPC 服务。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.date: 07/27/2020
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
uid: grpc/client
ms.openlocfilehash: 6515e87845cc5aa101532c18711d175a73581bee
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722704"
---
# <a name="call-grpc-services-with-the-net-client"></a>使用 .NET 客户端调用 gRPC 服务

[Grpc.Net.Client](https://www.nuget.org/packages/Grpc.Net.Client) NuGet 包提供了 .NET gRPC 客户端库。 本文档介绍如何执行以下操作：

* 配置 gRPC 客户端以调用 gRPC 服务。
* 对一元、服务器流式处理、客户端流式处理和双向流式处理方法进行 gRPC 调用。

## <a name="configure-grpc-client"></a>配置 gRPC 客户端

gRPC 客户端是从 [\*.proto  文件生成的](xref:grpc/basics#generated-c-assets)具体客户端类型。 具体 gRPC 客户端具有转换为 \*.proto  文件中 gRPC 服务的方法。 例如，名为 `Greeter` 的服务生成 `GreeterClient` 类型（包含调用服务的方法）。

gRPC 客户端是通过通道创建的。 首先使用 `GrpcChannel.ForAddress` 创建一个通道，然后使用该通道创建 gRPC 客户端：

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greet.GreeterClient(channel);
```

通道表示与 gRPC 服务的长期连接。 创建通道后，进行配置，使其具有与调用服务相关的选项。 例如，可在 `GrpcChannelOptions` 上指定用于调用的 `HttpClient`、发收和接收消息的最大大小以及记录日志，并将其与 `GrpcChannel.ForAddress` 一起使用。 有关选项的完整列表，请参阅[客户端配置选项](xref:grpc/configuration#configure-client-options)。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");

var greeterClient = new Greet.GreeterClient(channel);
var counterClient = new Count.CounterClient(channel);

// Use clients to call gRPC services
```

### <a name="configure-tls"></a>配置 TLS

gRPC 客户端必须使用与被调用服务相同的连接级别安全性。 gRPC 客户端传输层安全性 (TLS) 是在创建 gRPC 通道时配置的。 如果在调用服务时通道和服务的连接级别安全性不一致，gRPC 客户端就会抛出错误。

若要将 gRPC 通道配置为使用 TLS，请确保服务器地址以 `https` 开头。 例如，`GrpcChannel.ForAddress("https://localhost:5001")` 使用 HTTPS 协议。 gRPC 通道自动协商由 TLS 保护的连接，并使用安全连接进行 gRPC 调用。

> [!TIP]
> gRPC 支持通过 TLS 进行客户端证书身份验证。 若要了解如何为 gRPC 通道配置客户端证书，请参阅 <xref:grpc/authn-and-authz#client-certificate-authentication>。

若要调用不安全的 gRPC 服务，请确保服务器地址以 `http` 开头。 例如，`GrpcChannel.ForAddress("http://localhost:5000")` 使用 HTTP 协议。 在 .NET Core 3.1 或更高版本中，必须进行其他配置，才能[使用 .NET 客户端调用不安全的 gRPC 服务](xref:grpc/troubleshoot#call-insecure-grpc-services-with-net-core-client)。

### <a name="client-performance"></a>客户端性能

通道及客户端性能和使用情况：

* 创建通道成本高昂。 重用 gRPC 调用的通道可提高性能。
* gRPC 客户端是使用通道创建的。 gRPC 客户端是轻型对象，无需缓存或重用。
* 可从一个通道创建多个 gRPC 客户端（包括不同类型的客户端）。
* 通道和从该通道创建的客户端可由多个线程安全使用。
* 从通道创建的客户端可同时进行多个调用。

`GrpcChannel.ForAddress` 不是创建 gRPC 客户端的唯一选项。 如果要从 ASP.NET Core 应用调用 gRPC 服务，请考虑 [gRPC 客户端工厂集成](xref:grpc/clientfactory)。 gRPC 与 `HttpClientFactory` 集成是创建 gRPC 客户端的集中式操作备选方案。

> [!NOTE]
> Xamarin 当前不支持通过 HTTP/2 使用 `Grpc.Net.Client` 调用 gRPC。 我们正在改进 Xamarin 未来版本中的 HTTP/2 支持。 [Grpc.Core](https://www.nuget.org/packages/Grpc.Core) 和 [gRPC-Web](xref:grpc/browser) 是立即生效的可行备选方案。

## <a name="make-grpc-calls"></a>进行 gRPC 调用

在客户端上调用方法会启动 gRPC 调用。 gRPC 客户端将处理消息序列化，并为 gRPC 调用寻址到正确服务。

gRPC 具有不同类型的方法。 如何使用客户端来进行 gRPC 调用取决于所调用的方法类型。 gRPC 方法类型如下：

* 一元
* 服务器流式处理
* 客户端流式处理
* 双向流式处理

### <a name="unary-call"></a>一元调用

一元调用从客户端发送请求消息开始。 服务结束后，返回响应消息。

```csharp
var client = new Greet.GreeterClient(channel);
var response = await client.SayHelloAsync(new HelloRequest { Name = "World" });

Console.WriteLine("Greeting: " + response.Message);
// Greeting: Hello World
```

\*.proto 文件中的每个一元服务方法将在用于调用方法的具体 gRPC 客户端类型上产生两个 .NET 方法：异步方法和阻塞方法  。 例如，`GreeterClient` 具有两种调用 `SayHello` 的方法：

* `GreeterClient.SayHelloAsync` - 以异步方式调用 `Greeter.SayHello` 服务。 敬请期待。
* `GreeterClient.SayHello` - 调用 `Greeter.SayHello` 服务并阻塞，直至结束。 不要在异步代码中使用。

### <a name="server-streaming-call"></a>服务器流式处理调用

服务器流式处理调用从客户端发送请求消息开始。 `ResponseStream.MoveNext()` 读取从服务流式处理的消息。 `ResponseStream.MoveNext()` 返回 `false` 时，服务器流式处理调用已完成。

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHellos(new HelloRequest { Name = "World" });

while (await call.ResponseStream.MoveNext())
{
    Console.WriteLine("Greeting: " + call.ResponseStream.Current.Message);
    // "Greeting: Hello World" is written multiple times
}
```

如果使用 C# 8 或更高版本，则可使用 `await foreach` 语法来读取消息。 `IAsyncStreamReader<T>.ReadAllAsync()` 扩展方法读取响应数据流中的所有消息：

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHellos(new HelloRequest { Name = "World" });

await foreach (var response in call.ResponseStream.ReadAllAsync())
{
    Console.WriteLine("Greeting: " + response.Message);
    // "Greeting: Hello World" is written multiple times
}
```

### <a name="client-streaming-call"></a>客户端流式处理调用

客户端无需发送消息即可开始客户端流式处理调用  。 客户端可选择使用 `RequestStream.WriteAsync` 发送消息。 客户端发送完消息后，应调用 `RequestStream.CompleteAsync()` 来通知服务。 服务返回响应消息时，调用完成。

```csharp
var client = new Counter.CounterClient(channel);
using var call = client.AccumulateCount();

for (var i = 0; i < 3; i++)
{
    await call.RequestStream.WriteAsync(new CounterRequest { Count = 1 });
}
await call.RequestStream.CompleteAsync();

var response = await call;
Console.WriteLine($"Count: {response.Count}");
// Count: 3
```

### <a name="bi-directional-streaming-call"></a>双向流式处理调用

客户端无需发送消息即可开始双向流式处理调用  。 客户端可选择使用 `RequestStream.WriteAsync` 发送消息。 使用 `ResponseStream.MoveNext()` 或 `ResponseStream.ReadAllAsync()` 可访问从服务流式处理的消息。 `ResponseStream` 没有更多消息时，双向流式处理调用完成。

```csharp
var client = new Echo.EchoClient(channel);
using var call = client.Echo();

Console.WriteLine("Starting background task to receive messages");
var readTask = Task.Run(async () =>
{
    await foreach (var response in call.ResponseStream.ReadAllAsync())
    {
        Console.WriteLine(response.Message);
        // Echo messages sent to the service
    }
});

Console.WriteLine("Starting to send messages");
Console.WriteLine("Type a message to echo then press enter.");
while (true)
{
    var result = Console.ReadLine();
    if (string.IsNullOrEmpty(result))
    {
        break;
    }

    await call.RequestStream.WriteAsync(new EchoMessage { Message = result });
}

Console.WriteLine("Disconnecting");
await call.RequestStream.CompleteAsync();
await readTask;
```

为获得最佳性能并避免客户端和服务中出现不必要的错误，请尝试正常完成双向流式调用。 当服务器已读取请求流且客户端已读取响应流时，双向调用正常完成。 前面的示例调用就是一个正常结束的双向调用。 在调用中，客户端：

1. 通过调用 `EchoClient.Echo` 启动新的双向流式调用。
2. 使用 `ResponseStream.ReadAllAsync()` 创建用于从服务中读取消息的后台任务。
3. 使用 `RequestStream.WriteAsync` 将消息发送到服务器。
4. 使用 `RequestStream.CompleteAsync()` 通知服务器它已发送消息。
5. 等待直到后台任务已读取所有传入消息。

双向流式处理调用期间，客户端和服务可在任何时间互相发送消息。 与双向调用交互的最佳客户端逻辑因服务逻辑而异。

## <a name="access-grpc-trailers"></a>访问 gRPC 尾部

gRPC 调用可能会返回 gRPC 尾部。 gRPC 尾部用于提供有关调用的名称/值元数据。 尾部提供与 HTTP 头相似的功能，但在调用结尾获得。

gRPC 尾部可通过 `GetTrailers()` 进行访问，它会返回元数据的集合。 尾部是在响应完成后返回的，因此你必须等待收到所有响应消息，然后才能访问尾部。

一元和客户端流式调用必须等待出现 `ResponseAsync` 后才能调用 `GetTrailers()`：

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHelloAsync(new HelloRequest { Name = "World" });
var response = await call.ResponseAsync;

Console.WriteLine("Greeting: " + response.Message);
// Greeting: Hello World

var trailers = call.GetTrailers();
var myValue = trailers.GetValue("my-trailer-name");
```

服务器和双向流式调用必须等到出现响应流，然后才能调用 `GetTrailers()`：

```csharp
var client = new Greet.GreeterClient(channel);
using var call = client.SayHellos(new HelloRequest { Name = "World" });

await foreach (var response in call.ResponseStream.ReadAllAsync())
{
    Console.WriteLine("Greeting: " + response.Message);
    // "Greeting: Hello World" is written multiple times
}

var trailers = call.GetTrailers();
var myValue = trailers.GetValue("my-trailer-name");
```

gRPC 尾部也可通过 `RpcException` 进行访问。 服务可能会同时返回尾部和“异常”gRPC 状态。 在这种情况下，尾部是从 gRPC 客户端引起的异常中检索得到的：

```csharp
var client = new Greet.GreeterClient(channel);
string myValue = null;

try
{
    using var call = client.SayHelloAsync(new HelloRequest { Name = "World" });
    var response = await call.ResponseAsync;

    Console.WriteLine("Greeting: " + response.Message);
    // Greeting: Hello World

    var trailers = call.GetTrailers();
    myValue = trailers.GetValue("my-trailer-name");
}
catch (RpcException ex)
{
    var trailers = ex.Trailers;
    myValue = trailers.GetValue("my-trailer-name");
}
```

## <a name="configure-deadline"></a>配置截止时间

建议配置 gRPC 调用截止时间，因为它提供调用时间的上限。 它能阻止异常运行的服务持续运行并耗尽服务器资源。 截止时间对于构建可靠应用非常有效。

配置 `CallOptions.Deadline` 以设置 gRPC 调用的截止时间：

[!code-csharp[](~/grpc/deadlines-cancellation/deadline-client.cs?highlight=7,12)]

有关详细信息，请参阅 <xref:grpc/deadlines-cancellation#deadlines>。

## <a name="additional-resources"></a>其他资源

* <xref:grpc/clientfactory>
* <xref:grpc/deadlines-cancellation>
* <xref:grpc/basics>
