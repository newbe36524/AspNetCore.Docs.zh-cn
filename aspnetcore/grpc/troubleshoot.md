---
title: 对 .NET Core 上的 gRPC 进行故障排除
author: jamesnk
description: 排查使用 .NET Core 上的 gRPC 时遇到的错误。
monikerRange: '>= aspnetcore-3.0'
ms.author: jamesnk
ms.custom: mvc
ms.date: 07/09/2020
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
uid: grpc/troubleshoot
ms.openlocfilehash: 0c897c8c640f8713fc7d3b6cad0e6c571131d7a5
ms.sourcegitcommit: ecae2aa432628b9181d1fa11037c231c7dd56c9e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92113837"
---
# <a name="troubleshoot-grpc-on-net-core"></a>对 .NET Core 上的 gRPC 进行故障排除

作者：[James Newton-King](https://twitter.com/jamesnk)

本文档讨论了在 .NET 上开发 gRPC 应用时经常遇到的问题。

## <a name="mismatch-between-client-and-service-ssltls-configuration"></a>客户端和服务 SSL/TLS 配置不匹配

默认情况下，gRPC 模板和示例使用[传输层安全性 (TLS)](https://tools.ietf.org/html/rfc5246) 来保护 gRPC 服务。 gRPC 客户端需要使用安全连接才能成功调用受保护的 gRPC 服务。

可在应用启动时验证 ASP.NET Core gRPC 服务是否正在使用 TLS。 该服务将在 HTTPS 终结点上侦听：

```
info: Microsoft.Hosting.Lifetime[0]
      Now listening on: https://localhost:5001
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
```

.NET Core 客户端必须在服务器地址中使用 `https` 才能使用安全连接进行调用：

```csharp
static async Task Main(string[] args)
{
    // The port number(5001) must match the port of the gRPC server.
    var channel = GrpcChannel.ForAddress("https://localhost:5001");
    var client = new Greet.GreeterClient(channel);
}
```

所有 gRPC 客户端实现都支持 TLS。 其他语言的 gRPC 客户端通常需要配置有 `SslCredentials` 的通道。 `SslCredentials` 指定客户端将使用的证书，必须使用该证书，而不是使用不安全凭据。 有关配置不同 gRPC 客户端实现以使用 TLS 的示例，请参阅 [gRPC 身份验证](https://www.grpc.io/docs/guides/auth/)。

## <a name="call-a-grpc-service-with-an-untrustedinvalid-certificate"></a>使用不受信任/无效证书调用 gRPC 服务

.NET gRPC 客户端要求服务具有受信任的证书。 在没有受信任证书的情况下调用 gRPC 服务时，将返回以下错误消息：

> 未经处理的异常。 System.Net.Http.HttpRequestException：无法建立 SSL 连接，请查看内部异常。
> ---> System.Security.Authentication.AuthenticationException：根据验证过程，远程证书无效。

如果在本地测试应用且 ASP.NET Core HTTPS 开发证书不受信任，则可能会显示此错误。 有关解决此问题的说明，请参阅[在 Windows 和 macOS 上信任 ASP.NET Core HTTPS 开发证书](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)。

如果要在另一台计算机上调用 gRPC 服务且无法信任该证书，则可以将 gRPC 客户端配置为忽略无效的证书。 下面的代码使用 [HttpClientHandler.ServerCertificateCustomValidationCallback](/dotnet/api/system.net.http.httpclienthandler.servercertificatecustomvalidationcallback) 来允许在没有受信任证书的情况下进行调用：

```csharp
var httpHandler = new HttpClientHandler();
// Return `true` to allow certificates that are untrusted/invalid
httpHandler.ServerCertificateCustomValidationCallback = 
    HttpClientHandler.DangerousAcceptAnyServerCertificateValidator;

var channel = GrpcChannel.ForAddress("https://localhost:5001",
    new GrpcChannelOptions { HttpHandler = httpHandler });
var client = new Greet.GreeterClient(channel);
```

> [!WARNING]
> 不受信任的证书只应在应用开发过程中使用。 生产应用应始终使用有效的证书。

## <a name="call-insecure-grpc-services-with-net-core-client"></a>使用 .NET Core 客户端调用不安全的 gRPC 服务

如果应用使用 .NET Core 3.x，必须进行其他配置，才能使用 .NET Core 客户端调用不安全的 gRPC 服务。 gRPC 客户端必须将 `System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport` 开关设置为 `true` 并在服务器地址中使用 `http`：

```csharp
// This switch must be set before creating the GrpcChannel/HttpClient.
AppContext.SetSwitch(
    "System.Net.Http.SocketsHttpHandler.Http2UnencryptedSupport", true);

// The port number(5000) must match the port of the gRPC server.
var channel = GrpcChannel.ForAddress("http://localhost:5000");
var client = new Greet.GreeterClient(channel);
```

.NET 5 应用不需要其他配置，但若要调用不安全的 gRPC 服务，它们必须使用 `Grpc.Net.Client` 版本 2.32.0 或更高版本。

## <a name="unable-to-start-aspnet-core-grpc-app-on-macos"></a>无法在 macOS 上启动 ASP.NET Core gRPC 应用

Kestrel 不支持 macOS 和更早的 Windows 版本（如 Windows 7）上的带有 TLS 的 HTTP/2。 默认情况下，ASP.NET Core gRPC 模板和示例使用 TLS。 尝试启动 gRPC 服务器时，你将看到以下错误消息：

> 无法绑定到 IPv4 环回接口上的 https://localhost:5001 ：“由于缺少 ALPN 支持，macOS 不支持使用 TLS 的 HTTP/2。”。

若要解决此问题，请将 Kestrel 和 gRPC 客户端配置为使用不带有 TLS 的 HTTP/2。 应仅在开发过程中执行此操作。 如果不使用 TLS，将会在不加密的情况下发送 gRPC 消息。

Kestrel 必须在 Program.cs 中配置不带有 TLS 的 HTTP/2 终结点：

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.ConfigureKestrel(options =>
            {
                // Setup a HTTP/2 endpoint without TLS.
                options.ListenLocalhost(5000, o => o.Protocols = 
                    HttpProtocols.Http2);
            });
            webBuilder.UseStartup<Startup>();
        });
```

如果在未带有 TLS 的情况下配置了 HTTP/2 终结点，则终结点的 [ListenOptions.Protocols](xref:fundamentals/servers/kestrel#listenoptionsprotocols) 必须设置为 `HttpProtocols.Http2`。 无法使用 `HttpProtocols.Http1AndHttp2`，因为需要使用 TLS 来协商 HTTP/2。 如果未带有 TLS，则与终结点的所有连接均默认为 HTTP/1.1，且 gRPC 调用会失败。

还必须将 gRPC 客户端配置为不使用 TLS。 有关详细信息，请参阅[使用 .NET Core 客户端调用不安全的 gRPC 服务](#call-insecure-grpc-services-with-net-core-client)。

> [!WARNING]
> 应仅在应用开发过程中使用不带有 TLS 的 HTTP/2。 生产应用应始终使用传输安全性。 有关详细信息，请参阅[适用于 ASP.NET Core 的 gRPC 的安全注意事项](xref:grpc/security#transport-security)。

## <a name="grpc-c-assets-are-not-code-generated-from-proto-files"></a>gRPC C# 资产不是从 .proto 文件生成的代码

具体客户端和服务基类的 gRPC 代码生成需要从项目引用 protobuf 文件和工具。 必须包括：

* 要在 `<Protobuf>` 项目组中使用的 .proto 文件。 [导入的 .proto 文件](https://developers.google.com/protocol-buffers/docs/proto3#importing-definitions)必须由项目引用。
* 对 gRPC 工具包 [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) 的包引用。

有关生成 gRPC C# 资产的详细信息，请参阅 <xref:grpc/basics>。

托管 gRPC 服务的 ASP.NET Core web 应用仅需要已生成的服务基类：

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Server" />
</ItemGroup>
```

发出 gRPC 调用的 gRPC 客户端应用仅需要已生成的具体客户端：

```xml
<ItemGroup>
  <Protobuf Include="Protos\greet.proto" GrpcServices="Client" />
</ItemGroup>
```

## <a name="wpf-projects-unable-to-generate-grpc-c-assets-from-proto-files"></a>WPF 项目无法从 .proto 文件生成 gRPC C# 资产

WPF 项目存在一个[已知问题](https://github.com/dotnet/wpf/issues/810)，其阻止 gRPC 代码生成正常运行。 在 WPF 项目中通过引用 `Grpc.Tools` 和 .proto 文件生成的任何 gRPC 类型在使用时都将创建编译错误：

> 错误 CS0246：找不到类型名称或命名空间名称 ’MyGrpcServices’ (是否缺少 using 指令或程序集引用?)

可以通过以下方式解决此问题：

1. 创建新的 .NET Core 类库项目。
2. 在新项目中，添加引用以[从 \*.proto 文件启用 C# 代码生成](xref:grpc/basics#generated-c-assets)：
    * 将包引用添加到 [Grpc.Tools](https://www.nuget.org/packages/Grpc.Tools/) 包。
    * 将 \*.proto 文件添加到 `<Protobuf>` 项目组。
3. 在 WPF 应用程序中，添加对新项目的引用。

WPF 应用程序可以使用来自新类库项目的 gRPC 生成的类型。

[!INCLUDE[](~/includes/gRPCazure.md)]
