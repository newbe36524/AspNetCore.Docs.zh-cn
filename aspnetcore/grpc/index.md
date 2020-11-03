---
title: .NET Core 上的 gRPC 的简介
author: juntaoluo
description: 了解使用 Kestrel 服务器和 ASP.NET Core 堆栈的 gRPC 服务。
monikerRange: '>= aspnetcore-3.0'
ms.author: johluo
ms.date: 09/20/2019
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
uid: grpc/index
ms.openlocfilehash: 80f44e3845cc1e3c87d5d657807a318eb65e6c6f
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059892"
---
# <a name="introduction-to-grpc-on-net-core"></a>.NET Core 上的 gRPC 的简介

作者：[John Luo](https://github.com/juntaoluo) 和 [James Newton-King](https://twitter.com/jamesnk)

[gRPC](https://grpc.io/docs/guides/) 是一种与语言无关的高性能远程过程调用 (RPC) 框架。

gRPC 的主要优点是：
* 现代高性能轻量级 RPC 框架。
* 协定优先 API 开发，默认使用协议缓冲区，允许与语言无关的实现。
* 可用于多种语言的工具，以生成强类型服务器和客户端。
* 支持客户端、服务器和双向流式处理调用。
* 使用 Protobuf 二进制序列化减少对网络的使用。

这些优点使 gRPC 适用于：
* 效率至关重要的轻量级微服务。
* 需要多种语言用于开发的 Polyglot 系统。
* 需要处理流式处理请求或响应的点对点实时服务。

[!INCLUDE[](~/includes/gRPCazure.md)]

## <a name="c-tooling-support-for-proto-files"></a>.proto 文件的 C# 工具支持

gRPC 使用协定优先方法进行 API 开发。 在 \*.proto  文件中定义服务和消息：

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

通过在项目中包含 \*.proto  文件，可以自动生成用于服务、客户端和消息的 .NET 类型：

* 将包引用添加到 [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) 包。
* 将 \*.proto 文件添加到 `<Protobuf>` 项目组。

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" />
</ItemGroup>
```

有关 gRPC 工具支持的详细信息，请参阅 <xref:grpc/basics>。

## <a name="grpc-services-on-aspnet-core"></a>ASP.NET Core 上的 gRPC 服务

gRPC 服务可以托管在 ASP.NET Core 上。 这些服务与常用的 ASP.NET Core 功能（例如日志记录、依赖关系注入(DI)、身份验证和授权）完全集成。

gRPC 服务项目模板提供了一个入门版服务：

```csharp
public class GreeterService : Greeter.GreeterBase
{
    private readonly ILogger<GreeterService> _logger;

    public GreeterService(ILogger<GreeterService> logger)
    {
        _logger = logger;
    }

    public override Task<HelloReply> SayHello(HelloRequest request,
        ServerCallContext context)
    {
        _logger.LogInformation("Saying hello to {Name}", request.Name);
        return Task.FromResult(new HelloReply 
        {
            Message = "Hello " + request.Name
        });
    }
}
```

`GreeterService` 继承自 `GreeterBase` 类型，后者是从 \*.proto  文件的 `Greeter` 服务生成的。 Startup.cs  中的客户端可以访问该服务：

```csharp
app.UseEndpoints(endpoints =>
{
    endpoints.MapGrpcService<GreeterService>();
});
```

若要了解有关 ASP.NET Core 上的 gRPC 服务的详细信息，请参阅 <xref:grpc/aspnetcore>。

## <a name="call-grpc-services-with-a-net-client"></a>使用 .NET 客户端调用 gRPC 服务

gRPC 客户端是从 [\*.proto  文件生成的](xref:grpc/basics#generated-c-assets)具体客户端类型。 具体 gRPC 客户端具有转换为 \*.proto  文件中 gRPC 服务的方法。

```csharp
var channel = GrpcChannel.ForAddress("https://localhost:5001");
var client = new Greeter.GreeterClient(channel);

var response = await client.SayHelloAsync(
    new HelloRequest { Name = "World" });

Console.WriteLine(response.Message);
```

gRPC 客户端是使用通道创建的，该通道表示与 gRPC 服务的长期连接。 可以使用 `GrpcChannel.ForAddress` 创建通道。

有关创建客户端、调用不同服务方法的详细信息，请参阅 <xref:grpc/client>。

## <a name="additional-resources"></a>其他资源

* <xref:grpc/basics>
* <xref:grpc/aspnetcore>
* <xref:grpc/client>
* <xref:grpc/clientfactory>
* <xref:tutorials/grpc/grpc-start>
