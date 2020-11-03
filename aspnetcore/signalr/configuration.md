---
title: ASP.NET Core SignalR 配置
author: bradygaster
description: 了解如何配置 ASP.NET Core SignalR 应用。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 04/12/2020
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
uid: signalr/configuration
ms.openlocfilehash: 7dac8c84683553a52e07ecc61c8bcf8616e77dc6
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061231"
---
# <a name="aspnet-core-no-locsignalr-configuration"></a>ASP.NET Core SignalR 配置

::: moniker range=">= aspnetcore-5.0"

## <a name="jsonmessagepack-serialization-options"></a>JSON/MessagePack 序列化选项

ASP.NET Core SignalR 支持两个用于编码消息的协议： [JSON](https://www.json.org/) 和 [MessagePack](https://msgpack.org/index.html)。 每个协议都具有序列化配置选项。

可以使用 [AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol) 扩展方法在服务器上配置 JSON 序列化。 `AddJsonProtocol`可在[添加 SignalR ](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr)后添加 `Startup.ConfigureServices` 。 `AddJsonProtocol`方法采用接收对象的委托 `options` 。 该对象上的 [PayloadSerializerOptions](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializeroptions) 属性是一个 `System.Text.Json` <xref:System.Text.Json.JsonSerializerOptions> 对象，该对象可用于配置自变量和返回值的序列化。 有关详细信息，请参阅 [ 文档](/dotnet/api/system.text.json)中的System.Text.Js。

例如，若要将序列化程序配置为不更改属性名称的大小写（而不是默认的 "camelCase" 名称），请使用中的以下代码 `Startup.ConfigureServices` ：

```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null
    });
```

在 .NET 客户端中， `AddJsonProtocol` [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder)上存在相同的扩展方法。 `Microsoft.Extensions.DependencyInjection`必须导入命名空间才能解析扩展方法：

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;

// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null;
    })
    .Build();
```

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 JSON 序列化。

### <a name="switch-to-newtonsoftjson"></a>切换到 Newtonsoft.Js

如果需要的功能 `Newtonsoft.Json` 在中不受支持 `System.Text.Json` ，请参阅 [切换到 Newtonsoft.Js](xref:migration/22-to-30#switch-to-newtonsoftjson)。

### <a name="messagepack-serialization-options"></a>MessagePack 序列化选项

可以通过向 [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) 调用提供委托来配置 MessagePack 序列化。 有关更多详细信息，请参阅[中 SignalR 的 MessagePack](xref:signalr/messagepackhubprotocol) 。

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 MessagePack 序列化。

## <a name="configure-server-options"></a>配置服务器选项

下表描述了用于配置中心的选项 SignalR ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 秒 | 如果客户端未收到消息 (在此时间间隔内包含 keep-alive) ，服务器将认为客户端已断开连接。 由于实现方式的原因，客户端实际标记为断开连接可能需要更长的时间。 建议值为值的两倍 `KeepAliveInterval` 。|
| `HandshakeTimeout` | 15 秒 | 如果客户端在此时间间隔内未发送初始握手消息，连接将关闭。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果服务器未在此时间间隔内发送消息，则会自动发送 ping 消息，使连接保持打开状态。 更改时 `KeepAliveInterval` ，请更改 `ServerTimeout` / `serverTimeoutInMilliseconds` 客户端上的设置。 建议 `ServerTimeout` / `serverTimeoutInMilliseconds` 值为值的两倍 `KeepAliveInterval` 。  |
| `SupportedProtocols` | 所有已安装的协议 | 此中心支持的协议。 默认情况下，将允许在服务器上注册的所有协议，但可以从此列表中删除协议，以禁用各个集线器的特定协议。 |
| `EnableDetailedErrors` | `false` | 如果为，则在 `true` 集线器方法中引发异常时，详细的异常消息将返回到客户端。 默认值为 `false` ，因为这些异常消息可能包含敏感信息。 |
| `StreamBufferCapacity` | `10` | 可为客户端上载流缓冲的最大项数。 如果达到此限制，则会阻止处理调用，直到服务器处理流项。|
| `MaximumReceiveMessageSize` | 32 KB | 单个传入集线器消息的最大大小。 |
| `MaximumParallelInvocationsPerClient` | 1 | 每个客户端可以在进行排队之前并行调用的最大集线器方法数。 |

可以通过在中提供对调用的选项委托，为所有中心配置选项 `AddSignalR` `Startup.ConfigureServices` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

单个集线器的选项用于替代和中提供的全局选项 `AddSignalR` ，可以使用进行配置 <xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*> ：

```csharp
services.AddSignalR().AddHubOptions<ChatHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>高级 HTTP 配置选项

用于 `HttpConnectionDispatcherOptions` 配置与传输和内存缓冲区管理相关的高级设置。 这些选项通过将委托传递给中的[MapHub \<T> ](/dotnet/api/microsoft.aspnetcore.builder.hubendpointroutebuilderextensions.maphub)来配置 `Startup.Configure` 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chathub", options =>
        {
            options.Transports =
                HttpTransportType.WebSockets |
                HttpTransportType.LongPolling;
        });
    });
}
```

下表描述了用于配置 ASP.NET Core SignalR 的高级 HTTP 选项的选项：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 在应用反压之前，服务器从客户端接收的最大字节数。 增大此值后，服务器可以更快地接收更大的消息，而无需应用反压，但会增加内存消耗。 |
| `AuthorizationData` | 从应用于 Hub 类的属性中自动收集的数据 `Authorize` 。 | 用于确定客户端是否有权连接到集线器的 [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) 对象的列表。 |
| `TransportMaxBufferSize` | 32 KB | 在观察反压之前，服务器要发送的最大字节数。 增大此值后，服务器可以更快地缓冲更大的消息，而无需等待反压，但会增加内存消耗。 |
| `Transports` | 所有传输均已启用。 | 值的位标志枚举 `HttpTransportType` ，可限制客户端可用于连接的传输。 |
| `LongPolling` | 请参阅下文。 | 特定于长轮询传输的其他选项。 |
| `WebSockets` | 请参阅下文。 | 特定于 Websocket 传输的其他选项。 |
| `MinimumProtocolVersion` | 0 | 指定 negotiate 协议的最低版本。 这用于将客户端限制到较新的版本。 |

长轮询传输具有可使用属性配置的其他选项 `LongPolling` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90秒 | 服务器在终止单个轮询请求之前等待发送到客户端的消息的最长时间。 减小此值将导致客户端更频繁地发出新的投票请求。 |

WebSocket 传输具有可使用属性配置的其他选项 `WebSockets` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 秒 | 服务器关闭后，如果客户端在此时间间隔内未能关闭，则连接将终止。 |
| `SubProtocolSelector` | `null` | 一个委托，可用于将 `Sec-WebSocket-Protocol` 标头设置为自定义值。 委托接收客户端请求的值作为输入，并且应返回所需的值。 |

## <a name="configure-client-options"></a>配置客户端选项

可以在 `HubConnectionBuilder` .net 和 JavaScript 客户端) 中可用的类型 (上配置客户端选项。 它在 Java 客户端中也可用，但 `HttpHubConnectionBuilder` 子类是包含生成器配置选项的内容，也是其 `HubConnection` 本身。

### <a name="configure-logging"></a>配置日志记录

使用方法在 .NET 客户端中配置日志记录 `ConfigureLogging` 。 日志提供程序和筛选器的注册方式与服务器上相同。 有关详细信息，请参阅 [ASP.NET Core 文档中的日志记录](xref:fundamentals/logging/index) 。

> [!NOTE]
> 若要注册日志记录提供程序，必须安装所需的包。 有关完整列表，请参阅文档的 [内置日志记录提供程序](xref:fundamentals/logging/index#built-in-logging-providers) 部分。

例如，若要启用控制台日志记录，请安装 `Microsoft.Extensions.Logging.Console` NuGet 包。 调用 `AddConsole` 扩展方法：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

在 JavaScript 客户端中，存在类似的 `configureLogging` 方法。 提供一个 `LogLevel` 值，该值指示要生成的日志消息的最小级别。 日志将写入浏览器控制台窗口中。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

`LogLevel`您还可以提供一个 `string` 表示日志级别名称的值，而不是一个值。 当 SignalR 你在无法访问常量的环境中配置日志记录时，这非常有用 `LogLevel` 。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging("warn")
    .build();
```

下表列出了可用的日志级别。 为 `configureLogging` 设置将记录的 **最小** 日志级别而提供的值。 将记录在此级别上记录的消息 **或在表中列出的级别** 。

| String                      | LogLevel               |
| --------------------------- | ---------------------- |
| `trace`                     | `LogLevel.Trace`       |
| `debug`                     | `LogLevel.Debug`       |
| `info` **或** `information` | `LogLevel.Information` |
| `warn` **或** `warning`     | `LogLevel.Warning`     |
| `error`                     | `LogLevel.Error`       |
| `critical`                  | `LogLevel.Critical`    |
| `none`                      | `LogLevel.None`        |

> [!NOTE]
> 若要完全禁用日志记录，请 `signalR.LogLevel.None` 在方法中指定 `configureLogging` 。

有关日志记录的详细信息，请参阅[ SignalR 诊断文档](xref:signalr/diagnostics)。

SignalRJava 客户端使用[SLF4J](https://www.slf4j.org/)库进行日志记录。 这是一个高级日志记录 API，它允许库的用户通过引入特定的日志记录依赖项来选择自己的特定日志记录实现。 下面的代码段演示如何将用于 `java.util.logging` SignalR Java 客户端。

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

如果未在依赖项中配置日志记录，SLF4J 将加载默认的非操作记录器，并提供以下警告消息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

可以安全地忽略此情况。

### <a name="configure-allowed-transports"></a>配置允许的传输

使用的传输 SignalR 可在 `WithUrl` JavaScript) 的调用 (中进行配置 `withUrl` 。 的值的按位 "或" `HttpTransportType` 可用于将客户端限制为仅使用指定的传输。 默认情况下，将启用所有传输。

例如，禁用 Server-Sent 事件传输，但允许 Websocket 和长轮询连接：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

在 JavaScript 客户端中，通过 `transport` 在提供的选项对象上设置字段来配置传输 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

此版本的 Java 客户端 websocket 是唯一可用的传输。

在 Java 客户端中，通过中的方法选择了传输 `withTransport` `HttpHubConnectionBuilder` 。 Java 客户端默认使用 Websocket 传输。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withTransport(TransportEnum.WEBSOCKETS)
    .build();
```

> [!NOTE]
> SignalRJava 客户端尚不支持传输回退。

### <a name="configure-bearer-authentication"></a>配置持有者身份验证

若要与请求一起提供身份验证数据 SignalR ，请使用 `AccessTokenProvider` `accessTokenFactory` JavaScript) 中 (选项来指定返回所需访问令牌的函数。 在 .NET 客户端中，此访问令牌作为 HTTP "持有者身份验证" 令牌传入 (使用 `Authorization`) 类型的标头 `Bearer` 。 在 JavaScript 客户端中，访问令牌用作持有者令牌， **但** 在某些情况下，浏览器 api 会限制在 Server-Sent 事件和 websocket 请求)  (具体应用标头的能力。 在这些情况下，访问令牌作为查询字符串值提供 `access_token` 。

在 .NET 客户端中， `AccessTokenProvider` 可使用中的选项委托指定选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

在 JavaScript 客户端中，通过 `accessTokenFactory` 在中设置 "选项" 对象上的字段来配置访问令牌 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

在 SignalR Java 客户端中，可以通过向 [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java)提供访问令牌工厂来配置用于身份验证的持有者令牌。 使用[withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__)提供[RxJava](https://github.com/ReactiveX/RxJava) [Single \<String> ](https://reactivex.io/documentation/single.html)。 如果调用了 [单延迟](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)，你可以编写逻辑来为客户端生成访问令牌。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>配置超时和 keep-alive 选项

用于配置超时和保持活动状态的其他选项可用于 `HubConnection` 对象本身：

# <a name="net"></a>[.NET](#tab/dotnet)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `HandshakeTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

在 .NET 客户端中，超时值指定为 `TimeSpan` 值。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onclose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `keepAliveIntervalInMilliseconds` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

# <a name="java"></a>[Java](#tab/java)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onClose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并触发 `onClose` 事件。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `getKeepAliveInterval` / `setKeepAliveInterval` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

---

### <a name="configure-additional-options"></a>配置其他选项

可以在中的 `WithUrl` `withUrl` JavaScript) 方法 (上 `HubConnectionBuilder` 或在 `HttpHubConnectionBuilder` Java 客户端中的各种配置 api 上配置其他选项：

# <a name="net"></a>[.NET](#tab/dotnet)

| .NET 选项 |  默认值 | 说明 |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `SkipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `ClientCertificates` | 空 | 要发送以对请求进行身份验证的 TLS 证书的集合。 |
| `Cookies` | 空 | cookie要随每个 http 请求一起发送的 http 的集合。 |
| `Credentials` | 空 | 要随每个 HTTP 请求一起发送的凭据。 |
| `CloseTimeout` | 5 秒 | 仅 Websocket。 客户端在关闭之后等待服务器确认关闭请求的最长时间。 如果服务器在这段时间内没有确认关闭，客户端将断开连接。 |
| `Headers` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |
| `HttpMessageHandlerFactory` | `null` | 一个委托，可用于配置或替换 `HttpMessageHandler` 用于发送 HTTP 请求的。 不用于 WebSocket 连接。 此委托必须返回非 null 值，并接收默认值作为参数。 修改该默认值的设置并将其返回，或返回一个新的 `HttpMessageHandler` 实例。 **当替换处理程序时，请确保从提供的处理程序复制您要保留的设置，否则，所配置的选项 (如 Cookie s 和标头) 将不会应用于新的处理程序。** |
| `Proxy` | `null` | 发送 HTTP 请求时要使用的 HTTP 代理。 |
| `UseDefaultCredentials` | `false` | 设置此布尔值可发送 HTTP 和 Websocket 请求的默认凭据。 这样就可以使用 Windows 身份验证。 |
| `WebSocketConfiguration` | `null` | 可用于配置其他 WebSocket 选项的委托。 接收可用于配置选项的 [ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) 实例。 |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| JavaScript 选项 | 默认值 | 说明 |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `transport` | `null` | 一个 <xref:Microsoft.AspNetCore.Http.Connections.HttpTransportType> 值，该值指定用于连接的传输。 |
| `headers` | `null` | 每个 HTTP 请求发送的标头的字典。 在浏览器中发送标头对于 Websocket 或流不起作用 <xref:Microsoft.AspNetCore.Http.Connections.HttpTransportType.ServerSentEvents> 。 |
| `logMessageContent` | `null` | 设置为 `true` 可记录客户端发送和接收的消息的字节数/字符数。 |
| `skipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `withCredentials` | `true` | 指定是否将凭据与 CORS 请求一起发送。 Azure App Service cookie 将用于粘滞会话，并且需要启用此选项才能正常工作。 有关 CORS 的详细信息 SignalR ，请参阅 <xref:signalr/security#cross-origin-resource-sharing> 。 |

# <a name="java"></a>[Java](#tab/java)

| Java 选项 | 默认值 | 说明 |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `shouldSkipNegotiate` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `withHeader` `withHeaders` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |

---

在 .NET 客户端中，可以通过提供给的 options 委托来修改这些选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.Headers["Foo"] = "Bar";
        options.SkipNegotiation = true;
        options.Transports = HttpTransportType.WebSockets;
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

在 JavaScript 客户端中，可以在提供给的 JavaScript 对象中提供这些选项 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        // "Foo: Bar" will not be sent with WebSockets or Server-Sent Events requests
        headers: { "Foo": "Bar" },
        transport: signalR.HttpTransportType.LongPolling 
    })
    .build();
```

在 Java 客户端中，这些选项可以在 `HttpHubConnectionBuilder` 从 `HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>

::: moniker-end
::: moniker range="= aspnetcore-3.1"

## <a name="jsonmessagepack-serialization-options"></a>JSON/MessagePack 序列化选项

ASP.NET Core SignalR 支持两个用于编码消息的协议： [JSON](https://www.json.org/) 和 [MessagePack](https://msgpack.org/index.html)。 每个协议都具有序列化配置选项。

可以使用 [AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol) 扩展方法在服务器上配置 JSON 序列化。 `AddJsonProtocol`可在[添加 SignalR ](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr)后添加 `Startup.ConfigureServices` 。 `AddJsonProtocol`方法采用接收对象的委托 `options` 。 该对象上的 [PayloadSerializerOptions](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializeroptions) 属性是一个 `System.Text.Json` <xref:System.Text.Json.JsonSerializerOptions> 对象，该对象可用于配置自变量和返回值的序列化。 有关详细信息，请参阅 [ 文档](/dotnet/api/system.text.json)中的System.Text.Js。

例如，若要将序列化程序配置为不更改属性名称的大小写（而不是默认的 "camelCase" 名称），请使用中的以下代码 `Startup.ConfigureServices` ：

```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null
    });
```

在 .NET 客户端中， `AddJsonProtocol` [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder)上存在相同的扩展方法。 `Microsoft.Extensions.DependencyInjection`必须导入命名空间才能解析扩展方法：

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;

// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null;
    })
    .Build();
```

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 JSON 序列化。

### <a name="switch-to-newtonsoftjson"></a>切换到 Newtonsoft.Js

如果需要的功能 `Newtonsoft.Json` 在中不受支持 `System.Text.Json` ，请参阅 [切换到 Newtonsoft.Js](xref:migration/22-to-30#switch-to-newtonsoftjson)。

### <a name="messagepack-serialization-options"></a>MessagePack 序列化选项

可以通过向 [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) 调用提供委托来配置 MessagePack 序列化。 有关更多详细信息，请参阅[中 SignalR 的 MessagePack](xref:signalr/messagepackhubprotocol) 。

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 MessagePack 序列化。

## <a name="configure-server-options"></a>配置服务器选项

下表描述了用于配置中心的选项 SignalR ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 秒 | 如果客户端未收到消息 (在此时间间隔内包含 keep-alive) ，服务器将认为客户端已断开连接。 由于实现方式的原因，客户端实际标记为断开连接可能需要更长的时间。 建议值为值的两倍 `KeepAliveInterval` 。|
| `HandshakeTimeout` | 15 秒 | 如果客户端在此时间间隔内未发送初始握手消息，连接将关闭。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果服务器未在此时间间隔内发送消息，则会自动发送 ping 消息，使连接保持打开状态。 更改时 `KeepAliveInterval` ，请更改 `ServerTimeout` / `serverTimeoutInMilliseconds` 客户端上的设置。 建议 `ServerTimeout` / `serverTimeoutInMilliseconds` 值为值的两倍 `KeepAliveInterval` 。  |
| `SupportedProtocols` | 所有已安装的协议 | 此中心支持的协议。 默认情况下，将允许在服务器上注册的所有协议，但可以从此列表中删除协议，以禁用各个集线器的特定协议。 |
| `EnableDetailedErrors` | `false` | 如果为，则在 `true` 集线器方法中引发异常时，详细的异常消息将返回到客户端。 默认值为 `false` ，因为这些异常消息可能包含敏感信息。 |
| `StreamBufferCapacity` | `10` | 可为客户端上载流缓冲的最大项数。 如果达到此限制，则会阻止处理调用，直到服务器处理流项。|
| `MaximumReceiveMessageSize` | 32 KB | 单个传入集线器消息的最大大小。 |

可以通过在中提供对调用的选项委托，为所有中心配置选项 `AddSignalR` `Startup.ConfigureServices` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

单个集线器的选项用于替代和中提供的全局选项 `AddSignalR` ，可以使用进行配置 <xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*> ：

```csharp
services.AddSignalR().AddHubOptions<ChatHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>高级 HTTP 配置选项

用于 `HttpConnectionDispatcherOptions` 配置与传输和内存缓冲区管理相关的高级设置。 这些选项通过将委托传递给中的[MapHub \<T> ](/dotnet/api/microsoft.aspnetcore.builder.hubendpointroutebuilderextensions.maphub)来配置 `Startup.Configure` 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chathub", options =>
        {
            options.Transports =
                HttpTransportType.WebSockets |
                HttpTransportType.LongPolling;
        });
    });
}
```

下表描述了用于配置 ASP.NET Core SignalR 的高级 HTTP 选项的选项：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 在应用反压之前，服务器从客户端接收的最大字节数。 增大此值后，服务器可以更快地接收更大的消息，而无需应用反压，但会增加内存消耗。 |
| `AuthorizationData` | 从应用于 Hub 类的属性中自动收集的数据 `Authorize` 。 | 用于确定客户端是否有权连接到集线器的 [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) 对象的列表。 |
| `TransportMaxBufferSize` | 32 KB | 在观察反压之前，服务器要发送的最大字节数。 增大此值后，服务器可以更快地缓冲更大的消息，而无需等待反压，但会增加内存消耗。 |
| `Transports` | 所有传输均已启用。 | 值的位标志枚举 `HttpTransportType` ，可限制客户端可用于连接的传输。 |
| `LongPolling` | 请参阅下文。 | 特定于长轮询传输的其他选项。 |
| `WebSockets` | 请参阅下文。 | 特定于 Websocket 传输的其他选项。 |
| `MinimumProtocolVersion` | 0 | 指定 negotiate 协议的最低版本。 这用于将客户端限制到较新的版本。 |

长轮询传输具有可使用属性配置的其他选项 `LongPolling` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90秒 | 服务器在终止单个轮询请求之前等待发送到客户端的消息的最长时间。 减小此值将导致客户端更频繁地发出新的投票请求。 |

WebSocket 传输具有可使用属性配置的其他选项 `WebSockets` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 秒 | 服务器关闭后，如果客户端在此时间间隔内未能关闭，则连接将终止。 |
| `SubProtocolSelector` | `null` | 一个委托，可用于将 `Sec-WebSocket-Protocol` 标头设置为自定义值。 委托接收客户端请求的值作为输入，并且应返回所需的值。 |

## <a name="configure-client-options"></a>配置客户端选项

可以在 `HubConnectionBuilder` .net 和 JavaScript 客户端) 中可用的类型 (上配置客户端选项。 它在 Java 客户端中也可用，但 `HttpHubConnectionBuilder` 子类是包含生成器配置选项的内容，也是其 `HubConnection` 本身。

### <a name="configure-logging"></a>配置日志记录

使用方法在 .NET 客户端中配置日志记录 `ConfigureLogging` 。 日志提供程序和筛选器的注册方式与服务器上相同。 有关详细信息，请参阅 [ASP.NET Core 文档中的日志记录](xref:fundamentals/logging/index) 。

> [!NOTE]
> 若要注册日志记录提供程序，必须安装所需的包。 有关完整列表，请参阅文档的 [内置日志记录提供程序](xref:fundamentals/logging/index#built-in-logging-providers) 部分。

例如，若要启用控制台日志记录，请安装 `Microsoft.Extensions.Logging.Console` NuGet 包。 调用 `AddConsole` 扩展方法：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

在 JavaScript 客户端中，存在类似的 `configureLogging` 方法。 提供一个 `LogLevel` 值，该值指示要生成的日志消息的最小级别。 日志将写入浏览器控制台窗口中。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

`LogLevel`您还可以提供一个 `string` 表示日志级别名称的值，而不是一个值。 当 SignalR 你在无法访问常量的环境中配置日志记录时，这非常有用 `LogLevel` 。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging("warn")
    .build();
```

下表列出了可用的日志级别。 为 `configureLogging` 设置将记录的 **最小** 日志级别而提供的值。 将记录在此级别上记录的消息 **或在表中列出的级别** 。

| String                      | LogLevel               |
| --------------------------- | ---------------------- |
| `trace`                     | `LogLevel.Trace`       |
| `debug`                     | `LogLevel.Debug`       |
| `info` **或** `information` | `LogLevel.Information` |
| `warn` **或** `warning`     | `LogLevel.Warning`     |
| `error`                     | `LogLevel.Error`       |
| `critical`                  | `LogLevel.Critical`    |
| `none`                      | `LogLevel.None`        |

> [!NOTE]
> 若要完全禁用日志记录，请 `signalR.LogLevel.None` 在方法中指定 `configureLogging` 。

有关日志记录的详细信息，请参阅[ SignalR 诊断文档](xref:signalr/diagnostics)。

SignalRJava 客户端使用[SLF4J](https://www.slf4j.org/)库进行日志记录。 这是一个高级日志记录 API，它允许库的用户通过引入特定的日志记录依赖项来选择自己的特定日志记录实现。 下面的代码段演示如何将用于 `java.util.logging` SignalR Java 客户端。

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

如果未在依赖项中配置日志记录，SLF4J 将加载默认的非操作记录器，并提供以下警告消息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

可以安全地忽略此情况。

### <a name="configure-allowed-transports"></a>配置允许的传输

使用的传输 SignalR 可在 `WithUrl` JavaScript) 的调用 (中进行配置 `withUrl` 。 的值的按位 "或" `HttpTransportType` 可用于将客户端限制为仅使用指定的传输。 默认情况下，将启用所有传输。

例如，禁用 Server-Sent 事件传输，但允许 Websocket 和长轮询连接：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

在 JavaScript 客户端中，通过 `transport` 在提供的选项对象上设置字段来配置传输 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

此版本的 Java 客户端 websocket 是唯一可用的传输。

在 Java 客户端中，通过中的方法选择了传输 `withTransport` `HttpHubConnectionBuilder` 。 Java 客户端默认使用 Websocket 传输。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withTransport(TransportEnum.WEBSOCKETS)
    .build();
```

> [!NOTE]
> SignalRJava 客户端尚不支持传输回退。

### <a name="configure-bearer-authentication"></a>配置持有者身份验证

若要与请求一起提供身份验证数据 SignalR ，请使用 `AccessTokenProvider` `accessTokenFactory` JavaScript) 中 (选项来指定返回所需访问令牌的函数。 在 .NET 客户端中，此访问令牌作为 HTTP "持有者身份验证" 令牌传入 (使用 `Authorization`) 类型的标头 `Bearer` 。 在 JavaScript 客户端中，访问令牌用作持有者令牌， **但** 在某些情况下，浏览器 api 会限制在 Server-Sent 事件和 websocket 请求)  (具体应用标头的能力。 在这些情况下，访问令牌作为查询字符串值提供 `access_token` 。

在 .NET 客户端中， `AccessTokenProvider` 可使用中的选项委托指定选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

在 JavaScript 客户端中，通过 `accessTokenFactory` 在中设置 "选项" 对象上的字段来配置访问令牌 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

在 SignalR Java 客户端中，可以通过向 [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java)提供访问令牌工厂来配置用于身份验证的持有者令牌。 使用[withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__)提供[RxJava](https://github.com/ReactiveX/RxJava) [Single \<String> ](https://reactivex.io/documentation/single.html)。 如果调用了 [单延迟](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)，你可以编写逻辑来为客户端生成访问令牌。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>配置超时和 keep-alive 选项

用于配置超时和保持活动状态的其他选项可用于 `HubConnection` 对象本身：

# <a name="net"></a>[.NET](#tab/dotnet)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `HandshakeTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

在 .NET 客户端中，超时值指定为 `TimeSpan` 值。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onclose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `keepAliveIntervalInMilliseconds` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

# <a name="java"></a>[Java](#tab/java)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onClose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并触发 `onClose` 事件。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `getKeepAliveInterval` / `setKeepAliveInterval` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

---

### <a name="configure-additional-options"></a>配置其他选项

可以在中的 `WithUrl` `withUrl` JavaScript) 方法 (上 `HubConnectionBuilder` 或在 `HttpHubConnectionBuilder` Java 客户端中的各种配置 api 上配置其他选项：

# <a name="net"></a>[.NET](#tab/dotnet)

| .NET 选项 |  默认值 | 说明 |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `SkipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `ClientCertificates` | 空 | 要发送以对请求进行身份验证的 TLS 证书的集合。 |
| `Cookies` | 空 | cookie要随每个 http 请求一起发送的 http 的集合。 |
| `Credentials` | 空 | 要随每个 HTTP 请求一起发送的凭据。 |
| `CloseTimeout` | 5 秒 | 仅 Websocket。 客户端在关闭之后等待服务器确认关闭请求的最长时间。 如果服务器在这段时间内没有确认关闭，客户端将断开连接。 |
| `Headers` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |
| `HttpMessageHandlerFactory` | `null` | 一个委托，可用于配置或替换 `HttpMessageHandler` 用于发送 HTTP 请求的。 不用于 WebSocket 连接。 此委托必须返回非 null 值，并接收默认值作为参数。 修改该默认值的设置并将其返回，或返回一个新的 `HttpMessageHandler` 实例。 **当替换处理程序时，请确保从提供的处理程序复制您要保留的设置，否则，所配置的选项 (如 Cookie s 和标头) 将不会应用于新的处理程序。** |
| `Proxy` | `null` | 发送 HTTP 请求时要使用的 HTTP 代理。 |
| `UseDefaultCredentials` | `false` | 设置此布尔值可发送 HTTP 和 Websocket 请求的默认凭据。 这样就可以使用 Windows 身份验证。 |
| `WebSocketConfiguration` | `null` | 可用于配置其他 WebSocket 选项的委托。 接收可用于配置选项的 [ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) 实例。 |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| JavaScript 选项 | 默认值 | 说明 |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `transport` | `null` | 一个 <xref:Microsoft.AspNetCore.Http.Connections.HttpTransportType> 值，该值指定用于连接的传输。 |
| `logMessageContent` | `null` | 设置为 `true` 可记录客户端发送和接收的消息的字节数/字符数。 |
| `skipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |

# <a name="java"></a>[Java](#tab/java)

| Java 选项 | 默认值 | 说明 |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `shouldSkipNegotiate` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `withHeader` `withHeaders` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |

---

在 .NET 客户端中，可以通过提供给的 options 委托来修改这些选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

在 JavaScript 客户端中，可以在提供给的 JavaScript 对象中提供这些选项 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

在 Java 客户端中，这些选项可以在 `HttpHubConnectionBuilder` 从 `HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>

::: moniker-end
::: moniker range="= aspnetcore-3.0"

## <a name="jsonmessagepack-serialization-options"></a>JSON/MessagePack 序列化选项

ASP.NET Core SignalR 支持两个用于编码消息的协议： [JSON](https://www.json.org/) 和 [MessagePack](https://msgpack.org/index.html)。 每个协议都具有序列化配置选项。

可以使用 [AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol) 扩展方法在服务器上配置 JSON 序列化。 `AddJsonProtocol`可在[添加 SignalR ](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr)后添加 `Startup.ConfigureServices` 。 `AddJsonProtocol`方法采用接收对象的委托 `options` 。 该对象上的 [PayloadSerializerOptions](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializeroptions) 属性是一个 `System.Text.Json` <xref:System.Text.Json.JsonSerializerOptions> 对象，该对象可用于配置自变量和返回值的序列化。 有关详细信息，请参阅 [ 文档](/dotnet/api/system.text.json)中的System.Text.Js。

例如，若要将序列化程序配置为不更改属性名称的大小写（而不是默认的 "camelCase" 名称），请使用中的以下代码 `Startup.ConfigureServices` ：

```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null;
    });
```

在 .NET 客户端中， `AddJsonProtocol` [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder)上存在相同的扩展方法。 `Microsoft.Extensions.DependencyInjection`必须导入命名空间才能解析扩展方法：

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;

// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerOptions.PropertyNamingPolicy = null;
    })
    .Build();
```

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 JSON 序列化。

### <a name="switch-to-newtonsoftjson"></a>切换到 Newtonsoft.Js

如果需要的功能 `Newtonsoft.Json` 在中不受支持 `System.Text.Json` ，请参阅 [切换到 Newtonsoft.Js](xref:migration/22-to-30#switch-to-newtonsoftjson)。

### <a name="messagepack-serialization-options"></a>MessagePack 序列化选项

可以通过向 [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) 调用提供委托来配置 MessagePack 序列化。 有关更多详细信息，请参阅[中 SignalR 的 MessagePack](xref:signalr/messagepackhubprotocol) 。

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 MessagePack 序列化。

## <a name="configure-server-options"></a>配置服务器选项

下表描述了用于配置中心的选项 SignalR ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 秒 | 如果客户端未收到消息 (在此时间间隔内包含 keep-alive) ，服务器将认为客户端已断开连接。 由于实现方式的原因，客户端实际标记为断开连接可能需要更长的时间。 建议值为值的两倍 `KeepAliveInterval` 。|
| `HandshakeTimeout` | 15 秒 | 如果客户端在此时间间隔内未发送初始握手消息，连接将关闭。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果服务器未在此时间间隔内发送消息，则会自动发送 ping 消息，使连接保持打开状态。 更改时 `KeepAliveInterval` ，请更改 `ServerTimeout` / `serverTimeoutInMilliseconds` 客户端上的设置。 建议 `ServerTimeout` / `serverTimeoutInMilliseconds` 值为值的两倍 `KeepAliveInterval` 。  |
| `SupportedProtocols` | 所有已安装的协议 | 此中心支持的协议。 默认情况下，将允许在服务器上注册的所有协议，但可以从此列表中删除协议，以禁用各个集线器的特定协议。 |
| `EnableDetailedErrors` | `false` | 如果为，则在 `true` 集线器方法中引发异常时，详细的异常消息将返回到客户端。 默认值为 `false` ，因为这些异常消息可能包含敏感信息。 |
| `StreamBufferCapacity` | `10` | 可为客户端上载流缓冲的最大项数。 如果达到此限制，则会阻止处理调用，直到服务器处理流项。|
| `MaximumReceiveMessageSize` | 32 KB | 单个传入集线器消息的最大大小。 |

可以通过在中提供对调用的选项委托，为所有中心配置选项 `AddSignalR` `Startup.ConfigureServices` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

单个集线器的选项用于替代和中提供的全局选项 `AddSignalR` ，可以使用进行配置 <xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*> ：

```csharp
services.AddSignalR().AddHubOptions<ChatHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>高级 HTTP 配置选项

用于 `HttpConnectionDispatcherOptions` 配置与传输和内存缓冲区管理相关的高级设置。 这些选项通过将委托传递给中的[MapHub \<T> ](/dotnet/api/microsoft.aspnetcore.builder.hubendpointroutebuilderextensions.maphub)来配置 `Startup.Configure` 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseRouting();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHub<ChatHub>("/chathub", options =>
        {
            options.Transports =
                HttpTransportType.WebSockets |
                HttpTransportType.LongPolling;
        });
    });
}
```

下表描述了用于配置 ASP.NET Core SignalR 的高级 HTTP 选项的选项：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 在应用反压之前，服务器从客户端接收的最大字节数。 增大此值后，服务器可以更快地接收更大的消息，而无需应用反压，但会增加内存消耗。 |
| `AuthorizationData` | 从应用于 Hub 类的属性中自动收集的数据 `Authorize` 。 | 用于确定客户端是否有权连接到集线器的 [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) 对象的列表。 |
| `TransportMaxBufferSize` | 32 KB | 在观察反压之前，服务器要发送的最大字节数。 增大此值后，服务器可以更快地缓冲更大的消息，而无需等待反压，但会增加内存消耗。 |
| `Transports` | 所有传输均已启用。 | 值的位标志枚举 `HttpTransportType` ，可限制客户端可用于连接的传输。 |
| `LongPolling` | 请参阅下文。 | 特定于长轮询传输的其他选项。 |
| `WebSockets` | 请参阅下文。 | 特定于 Websocket 传输的其他选项。 |

长轮询传输具有可使用属性配置的其他选项 `LongPolling` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90秒 | 服务器在终止单个轮询请求之前等待发送到客户端的消息的最长时间。 减小此值将导致客户端更频繁地发出新的投票请求。 |

WebSocket 传输具有可使用属性配置的其他选项 `WebSockets` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 秒 | 服务器关闭后，如果客户端在此时间间隔内未能关闭，则连接将终止。 |
| `SubProtocolSelector` | `null` | 一个委托，可用于将 `Sec-WebSocket-Protocol` 标头设置为自定义值。 委托接收客户端请求的值作为输入，并且应返回所需的值。 |

## <a name="configure-client-options"></a>配置客户端选项

可以在 `HubConnectionBuilder` .net 和 JavaScript 客户端) 中可用的类型 (上配置客户端选项。 它在 Java 客户端中也可用，但 `HttpHubConnectionBuilder` 子类是包含生成器配置选项的内容，也是其 `HubConnection` 本身。

### <a name="configure-logging"></a>配置日志记录

使用方法在 .NET 客户端中配置日志记录 `ConfigureLogging` 。 日志提供程序和筛选器的注册方式与服务器上相同。 有关详细信息，请参阅 [ASP.NET Core 文档中的日志记录](xref:fundamentals/logging/index) 。

> [!NOTE]
> 若要注册日志记录提供程序，必须安装所需的包。 有关完整列表，请参阅文档的 [内置日志记录提供程序](xref:fundamentals/logging/index#built-in-logging-providers) 部分。

例如，若要启用控制台日志记录，请安装 `Microsoft.Extensions.Logging.Console` NuGet 包。 调用 `AddConsole` 扩展方法：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

在 JavaScript 客户端中，存在类似的 `configureLogging` 方法。 提供一个 `LogLevel` 值，该值指示要生成的日志消息的最小级别。 日志将写入浏览器控制台窗口中。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

`LogLevel`您还可以提供一个 `string` 表示日志级别名称的值，而不是一个值。 当 SignalR 你在无法访问常量的环境中配置日志记录时，这非常有用 `LogLevel` 。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging("warn")
    .build();
```

下表列出了可用的日志级别。 为 `configureLogging` 设置将记录的 **最小** 日志级别而提供的值。 将记录在此级别上记录的消息 **或在表中列出的级别** 。

| String                      | LogLevel               |
| --------------------------- | ---------------------- |
| `trace`                     | `LogLevel.Trace`       |
| `debug`                     | `LogLevel.Debug`       |
| `info` **或** `information` | `LogLevel.Information` |
| `warn` **或** `warning`     | `LogLevel.Warning`     |
| `error`                     | `LogLevel.Error`       |
| `critical`                  | `LogLevel.Critical`    |
| `none`                      | `LogLevel.None`        |

> [!NOTE]
> 若要完全禁用日志记录，请 `signalR.LogLevel.None` 在方法中指定 `configureLogging` 。

有关日志记录的详细信息，请参阅[ SignalR 诊断文档](xref:signalr/diagnostics)。

SignalRJava 客户端使用[SLF4J](https://www.slf4j.org/)库进行日志记录。 这是一个高级日志记录 API，它允许库的用户通过引入特定的日志记录依赖项来选择自己的特定日志记录实现。 下面的代码段演示如何将用于 `java.util.logging` SignalR Java 客户端。

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

如果未在依赖项中配置日志记录，SLF4J 将加载默认的非操作记录器，并提供以下警告消息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

可以安全地忽略此情况。

### <a name="configure-allowed-transports"></a>配置允许的传输

使用的传输 SignalR 可在 `WithUrl` JavaScript) 的调用 (中进行配置 `withUrl` 。 的值的按位 "或" `HttpTransportType` 可用于将客户端限制为仅使用指定的传输。 默认情况下，将启用所有传输。

例如，禁用 Server-Sent 事件传输，但允许 Websocket 和长轮询连接：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

在 JavaScript 客户端中，通过 `transport` 在提供的选项对象上设置字段来配置传输 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

此版本的 Java 客户端 websocket 是唯一可用的传输。

在 Java 客户端中，通过中的方法选择了传输 `withTransport` `HttpHubConnectionBuilder` 。 Java 客户端默认使用 Websocket 传输。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withTransport(TransportEnum.WEBSOCKETS)
    .build();
```

> [!NOTE]
> SignalRJava 客户端尚不支持传输回退。

### <a name="configure-bearer-authentication"></a>配置持有者身份验证

若要与请求一起提供身份验证数据 SignalR ，请使用 `AccessTokenProvider` `accessTokenFactory` JavaScript) 中 (选项来指定返回所需访问令牌的函数。 在 .NET 客户端中，此访问令牌作为 HTTP "持有者身份验证" 令牌传入 (使用 `Authorization`) 类型的标头 `Bearer` 。 在 JavaScript 客户端中，访问令牌用作持有者令牌， **但** 在某些情况下，浏览器 api 会限制在 Server-Sent 事件和 websocket 请求)  (具体应用标头的能力。 在这些情况下，访问令牌作为查询字符串值提供 `access_token` 。

在 .NET 客户端中， `AccessTokenProvider` 可使用中的选项委托指定选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

在 JavaScript 客户端中，通过 `accessTokenFactory` 在中设置 "选项" 对象上的字段来配置访问令牌 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

在 SignalR Java 客户端中，可以通过向 [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java)提供访问令牌工厂来配置用于身份验证的持有者令牌。 使用[withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__)提供[RxJava](https://github.com/ReactiveX/RxJava) [Single \<String> ](https://reactivex.io/documentation/single.html)。 如果调用了 [单延迟](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)，你可以编写逻辑来为客户端生成访问令牌。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>配置超时和 keep-alive 选项

用于配置超时和保持活动状态的其他选项可用于 `HubConnection` 对象本身：

# <a name="net"></a>[.NET](#tab/dotnet)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `HandshakeTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

在 .NET 客户端中，超时值指定为 `TimeSpan` 值。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onclose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `keepAliveIntervalInMilliseconds` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

# <a name="java"></a>[Java](#tab/java)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onClose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并触发 `onClose` 事件。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `getKeepAliveInterval` / `setKeepAliveInterval` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

---

### <a name="configure-additional-options"></a>配置其他选项

可以在中的 `WithUrl` `withUrl` JavaScript) 方法 (上 `HubConnectionBuilder` 或在 `HttpHubConnectionBuilder` Java 客户端中的各种配置 api 上配置其他选项：

# <a name="net"></a>[.NET](#tab/dotnet)

| .NET 选项 |  默认值 | 说明 |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `SkipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `ClientCertificates` | 空 | 要发送以对请求进行身份验证的 TLS 证书的集合。 |
| `Cookies` | 空 | cookie要随每个 http 请求一起发送的 http 的集合。 |
| `Credentials` | 空 | 要随每个 HTTP 请求一起发送的凭据。 |
| `CloseTimeout` | 5 秒 | 仅 Websocket。 客户端在关闭之后等待服务器确认关闭请求的最长时间。 如果服务器在这段时间内没有确认关闭，客户端将断开连接。 |
| `Headers` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |
| `HttpMessageHandlerFactory` | `null` | 一个委托，可用于配置或替换 `HttpMessageHandler` 用于发送 HTTP 请求的。 不用于 WebSocket 连接。 此委托必须返回非 null 值，并接收默认值作为参数。 修改该默认值的设置并将其返回，或返回一个新的 `HttpMessageHandler` 实例。 **当替换处理程序时，请确保从提供的处理程序复制您要保留的设置，否则，所配置的选项 (如 Cookie s 和标头) 将不会应用于新的处理程序。** |
| `Proxy` | `null` | 发送 HTTP 请求时要使用的 HTTP 代理。 |
| `UseDefaultCredentials` | `false` | 设置此布尔值可发送 HTTP 和 Websocket 请求的默认凭据。 这样就可以使用 Windows 身份验证。 |
| `WebSocketConfiguration` | `null` | 可用于配置其他 WebSocket 选项的委托。 接收可用于配置选项的 [ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) 实例。 |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| JavaScript 选项 | 默认值 | 说明 |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `transport` | `null` | 一个 <xref:Microsoft.AspNetCore.Http.Connections.HttpTransportType> 值，该值指定用于连接的传输。 |
| `logMessageContent` | `null` | 设置为 `true` 可记录客户端发送和接收的消息的字节数/字符数。 |
| `skipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |

# <a name="java"></a>[Java](#tab/java)

| Java 选项 | 默认值 | 说明 |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `shouldSkipNegotiate` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `withHeader` `withHeaders` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |

---

在 .NET 客户端中，可以通过提供给的 options 委托来修改这些选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

在 JavaScript 客户端中，可以在提供给的 JavaScript 对象中提供这些选项 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

在 Java 客户端中，这些选项可以在 `HttpHubConnectionBuilder` 从 `HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>

::: moniker-end
::: moniker range="= aspnetcore-2.2"

## <a name="jsonmessagepack-serialization-options"></a>JSON/MessagePack 序列化选项

ASP.NET Core SignalR 支持两个用于编码消息的协议： [JSON](https://www.json.org/) 和 [MessagePack](https://msgpack.org/index.html)。 每个协议都具有序列化配置选项。

可以使用[AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol)扩展方法在服务器上配置 JSON 序列化，该扩展方法可在方法[添加 SignalR ](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr)后添加 `Startup.ConfigureServices` 。 `AddJsonProtocol`方法采用接收对象的委托 `options` 。 该对象的 [PayloadSerializerSettings](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializersettings) 属性是一个 JSON.NET `JsonSerializerSettings` 对象，该对象可用于配置自变量和返回值的序列化。 有关详细信息，请参阅 [JSON.NET 文档](https://www.newtonsoft.com/json/help/html/Introduction.htm)。
 
例如，若要将序列化程序配置为使用 "PascalCase" 属性名称，而不是默认的 "camelCase" 名称，请在中使用以下代码 `Startup.ConfigureServices` ：
 
```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    });
```

在 .NET 客户端中， `AddJsonProtocol` [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder)上存在相同的扩展方法。 `Microsoft.Extensions.DependencyInjection`必须导入命名空间才能解析扩展方法：

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;
 
// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    })
    .Build();
```

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 JSON 序列化。

### <a name="messagepack-serialization-options"></a>MessagePack 序列化选项

可以通过向 [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) 调用提供委托来配置 MessagePack 序列化。 有关更多详细信息，请参阅[中 SignalR 的 MessagePack](xref:signalr/messagepackhubprotocol) 。

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 MessagePack 序列化。

## <a name="configure-server-options"></a>配置服务器选项

下表描述了用于配置中心的选项 SignalR ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ClientTimeoutInterval` | 30 秒 | 如果客户端未收到消息 (在此时间间隔内包含 keep-alive) ，服务器将认为客户端已断开连接。 由于实现方式的原因，客户端实际标记为断开连接可能需要更长的时间。 建议值为值的两倍 `KeepAliveInterval` 。|
| `HandshakeTimeout` | 15 秒 | 如果客户端在此时间间隔内未发送初始握手消息，连接将关闭。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果服务器未在此时间间隔内发送消息，则会自动发送 ping 消息，使连接保持打开状态。 更改时 `KeepAliveInterval` ，请更改 `ServerTimeout` / `serverTimeoutInMilliseconds` 客户端上的设置。 建议 `ServerTimeout` / `serverTimeoutInMilliseconds` 值为值的两倍 `KeepAliveInterval` 。  |
| `SupportedProtocols` | 所有已安装的协议 | 此中心支持的协议。 默认情况下，将允许在服务器上注册的所有协议，但可以从此列表中删除协议，以禁用各个集线器的特定协议。 |
| `EnableDetailedErrors` | `false` | 如果为，则在 `true` 集线器方法中引发异常时，详细的异常消息将返回到客户端。 默认值为 `false` ，因为这些异常消息可能包含敏感信息。 |

可以通过在中提供对调用的选项委托，为所有中心配置选项 `AddSignalR` `Startup.ConfigureServices` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

单个集线器的选项用于替代和中提供的全局选项 `AddSignalR` ，可以使用进行配置 <xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*> ：

```csharp
services.AddSignalR().AddHubOptions<ChatHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>高级 HTTP 配置选项

用于 `HttpConnectionDispatcherOptions` 配置与传输和内存缓冲区管理相关的高级设置。 这些选项通过将委托传递给中的[MapHub \<T> ](/dotnet/api/microsoft.aspnetcore.signalr.hubroutebuilder.maphub)来配置 `Startup.Configure` 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseSignalR((configure) =>
    {
        var desiredTransports =
            HttpTransportType.WebSockets |
            HttpTransportType.LongPolling;

        configure.MapHub<ChatHub>("/chathub", (options) =>
        {
            options.Transports = desiredTransports;
        });
    });
}
```

下表描述了用于配置 ASP.NET Core SignalR 的高级 HTTP 选项的选项：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 服务器缓冲的客户端接收到的最大字节数。 增大此值可使服务器接收更大的消息，但可能会对内存消耗产生负面影响。 |
| `AuthorizationData` | 从应用于 Hub 类的属性中自动收集的数据 `Authorize` 。 | 用于确定客户端是否有权连接到集线器的 [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) 对象的列表。 |
| `TransportMaxBufferSize` | 32 KB | 由服务器缓冲的应用发送的最大字节数。 增大此值后，服务器将发送更大的消息，但会对内存消耗产生负面影响。 |
| `Transports` | 所有传输均已启用。 | 值的位标志枚举 `HttpTransportType` ，可限制客户端可用于连接的传输。 |
| `LongPolling` | 请参阅下文。 | 特定于长轮询传输的其他选项。 |
| `WebSockets` | 请参阅下文。 | 特定于 Websocket 传输的其他选项。 |

长轮询传输具有可使用属性配置的其他选项 `LongPolling` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90秒 | 服务器在终止单个轮询请求之前等待发送到客户端的消息的最长时间。 减小此值将导致客户端更频繁地发出新的投票请求。 |

WebSocket 传输具有可使用属性配置的其他选项 `WebSockets` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 秒 | 服务器关闭后，如果客户端在此时间间隔内未能关闭，则连接将终止。 |
| `SubProtocolSelector` | `null` | 一个委托，可用于将 `Sec-WebSocket-Protocol` 标头设置为自定义值。 委托接收客户端请求的值作为输入，并且应返回所需的值。 |

## <a name="configure-client-options"></a>配置客户端选项

可以在 `HubConnectionBuilder` .net 和 JavaScript 客户端) 中可用的类型 (上配置客户端选项。 它在 Java 客户端中也可用，但 `HttpHubConnectionBuilder` 子类是包含生成器配置选项的内容，也是其 `HubConnection` 本身。

### <a name="configure-logging"></a>配置日志记录

使用方法在 .NET 客户端中配置日志记录 `ConfigureLogging` 。 日志提供程序和筛选器的注册方式与服务器上相同。 有关详细信息，请参阅 [ASP.NET Core 文档中的日志记录](xref:fundamentals/logging/index) 。

> [!NOTE]
> 若要注册日志记录提供程序，必须安装所需的包。 有关完整列表，请参阅文档的 [内置日志记录提供程序](xref:fundamentals/logging/index#built-in-logging-providers) 部分。

例如，若要启用控制台日志记录，请安装 `Microsoft.Extensions.Logging.Console` NuGet 包。 调用 `AddConsole` 扩展方法：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

在 JavaScript 客户端中，存在类似的 `configureLogging` 方法。 提供一个 `LogLevel` 值，该值指示要生成的日志消息的最小级别。 日志将写入浏览器控制台窗口中。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

> [!NOTE]
> 若要完全禁用日志记录，请 `signalR.LogLevel.None` 在方法中指定 `configureLogging` 。

有关日志记录的详细信息，请参阅[ SignalR 诊断文档](xref:signalr/diagnostics)。

SignalRJava 客户端使用[SLF4J](https://www.slf4j.org/)库进行日志记录。 这是一个高级日志记录 API，它允许库的用户通过引入特定的日志记录依赖项来选择自己的特定日志记录实现。 下面的代码段演示如何将用于 `java.util.logging` SignalR Java 客户端。

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

如果未在依赖项中配置日志记录，SLF4J 将加载默认的非操作记录器，并提供以下警告消息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

可以安全地忽略此情况。

### <a name="configure-allowed-transports"></a>配置允许的传输

使用的传输 SignalR 可在 `WithUrl` JavaScript) 的调用 (中进行配置 `withUrl` 。 的值的按位 "或" `HttpTransportType` 可用于将客户端限制为仅使用指定的传输。 默认情况下，将启用所有传输。

例如，禁用 Server-Sent 事件传输，但允许 Websocket 和长轮询连接：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

在 JavaScript 客户端中，通过 `transport` 在提供的选项对象上设置字段来配置传输 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

此版本的 Java 客户端 websocket 是唯一可用的传输。

### <a name="configure-bearer-authentication"></a>配置持有者身份验证

若要与请求一起提供身份验证数据 SignalR ，请使用 `AccessTokenProvider` `accessTokenFactory` JavaScript) 中 (选项来指定返回所需访问令牌的函数。 在 .NET 客户端中，此访问令牌作为 HTTP "持有者身份验证" 令牌传入 (使用 `Authorization`) 类型的标头 `Bearer` 。 在 JavaScript 客户端中，访问令牌用作持有者令牌， **但** 在某些情况下，浏览器 api 会限制在 Server-Sent 事件和 websocket 请求)  (具体应用标头的能力。 在这些情况下，访问令牌作为查询字符串值提供 `access_token` 。

在 .NET 客户端中， `AccessTokenProvider` 可使用中的选项委托指定选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

在 JavaScript 客户端中，通过 `accessTokenFactory` 在中设置 "选项" 对象上的字段来配置访问令牌 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

在 SignalR Java 客户端中，可以通过向 [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java)提供访问令牌工厂来配置用于身份验证的持有者令牌。 使用[withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__)提供[RxJava](https://github.com/ReactiveX/RxJava) [Single \<String> ](https://reactivex.io/documentation/single.html)。 如果调用了 [单延迟](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)，你可以编写逻辑来为客户端生成访问令牌。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>配置超时和 keep-alive 选项

用于配置超时和保持活动状态的其他选项可用于 `HubConnection` 对象本身：

# <a name="net"></a>[.NET](#tab/dotnet)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `HandshakeTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

在 .NET 客户端中，超时值指定为 `TimeSpan` 值。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onclose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `keepAliveIntervalInMilliseconds` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

# <a name="java"></a>[Java](#tab/java)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onClose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并触发 `onClose` 事件。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `getKeepAliveInterval` / `setKeepAliveInterval` | 15秒 (15000 毫秒)  | 确定客户端发送 ping 消息的间隔。 如果从客户端发送任何消息，则会将计时器重置为间隔的开始时间。 如果客户端没有在服务器上的设置中发送消息 `ClientTimeoutInterval` ，则服务器会将客户端视为已断开连接。 |

---

### <a name="configure-additional-options"></a>配置其他选项

可以在中的 `WithUrl` `withUrl` JavaScript) 方法 (上 `HubConnectionBuilder` 或在 `HttpHubConnectionBuilder` Java 客户端中的各种配置 api 上配置其他选项：

# <a name="net"></a>[.NET](#tab/dotnet)

| .NET 选项 |  默认值 | 说明 |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `SkipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `ClientCertificates` | 空 | 要发送以对请求进行身份验证的 TLS 证书的集合。 |
| `Cookies` | 空 | cookie要随每个 http 请求一起发送的 http 的集合。 |
| `Credentials` | 空 | 要随每个 HTTP 请求一起发送的凭据。 |
| `CloseTimeout` | 5 秒 | 仅 Websocket。 客户端在关闭之后等待服务器确认关闭请求的最长时间。 如果服务器在这段时间内没有确认关闭，客户端将断开连接。 |
| `Headers` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |
| `HttpMessageHandlerFactory` | `null` | 一个委托，可用于配置或替换 `HttpMessageHandler` 用于发送 HTTP 请求的。 不用于 WebSocket 连接。 此委托必须返回非 null 值，并接收默认值作为参数。 修改该默认值的设置并将其返回，或返回一个新的 `HttpMessageHandler` 实例。 **当替换处理程序时，请确保从提供的处理程序复制您要保留的设置，否则，所配置的选项 (如 Cookie s 和标头) 将不会应用于新的处理程序。** |
| `Proxy` | `null` | 发送 HTTP 请求时要使用的 HTTP 代理。 |
| `UseDefaultCredentials` | `false` | 设置此布尔值可发送 HTTP 和 Websocket 请求的默认凭据。 这样就可以使用 Windows 身份验证。 |
| `WebSocketConfiguration` | `null` | 可用于配置其他 WebSocket 选项的委托。 接收可用于配置选项的 [ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) 实例。 |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| JavaScript 选项 | 默认值 | 说明 |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `transport` | `null` | 一个 <xref:Microsoft.AspNetCore.Http.Connections.HttpTransportType> 值，该值指定用于连接的传输。 |
| `logMessageContent` | `null` | 设置为 `true` 可记录客户端发送和接收的消息的字节数/字符数。 |
| `skipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |

# <a name="java"></a>[Java](#tab/java)

| Java 选项 | 默认值 | 说明 |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `shouldSkipNegotiate` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `withHeader` `withHeaders` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |

---

在 .NET 客户端中，可以通过提供给的 options 委托来修改这些选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

在 JavaScript 客户端中，可以在提供给的 JavaScript 对象中提供这些选项 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

在 Java 客户端中，这些选项可以在 `HttpHubConnectionBuilder` 从 `HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>

::: moniker-end
::: moniker range="< aspnetcore-2.2"

## <a name="jsonmessagepack-serialization-options"></a>JSON/MessagePack 序列化选项

ASP.NET Core SignalR 支持两个用于编码消息的协议： [JSON](https://www.json.org/) 和 [MessagePack](https://msgpack.org/index.html)。 每个协议都具有序列化配置选项。

可以使用[AddJsonProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.jsonprotocoldependencyinjectionextensions.addjsonprotocol)扩展方法在服务器上配置 JSON 序列化，该扩展方法可在方法[添加 SignalR ](/dotnet/api/microsoft.extensions.dependencyinjection.signalrdependencyinjectionextensions.addsignalr)后添加 `Startup.ConfigureServices` 。 `AddJsonProtocol`方法采用接收对象的委托 `options` 。 该对象的 [PayloadSerializerSettings](/dotnet/api/microsoft.aspnetcore.signalr.jsonhubprotocoloptions.payloadserializersettings) 属性是一个 JSON.NET `JsonSerializerSettings` 对象，该对象可用于配置自变量和返回值的序列化。 有关详细信息，请参阅 [JSON.NET 文档](https://www.newtonsoft.com/json/help/html/Introduction.htm)。
 
例如，若要将序列化程序配置为使用 "PascalCase" 属性名称，而不是默认的 "camelCase" 名称，请在中使用以下代码 `Startup.ConfigureServices` ：
 
```csharp
services.AddSignalR()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    });
```

在 .NET 客户端中， `AddJsonProtocol` [HubConnectionBuilder](/dotnet/api/microsoft.aspnetcore.signalr.client.hubconnectionbuilder)上存在相同的扩展方法。 `Microsoft.Extensions.DependencyInjection`必须导入命名空间才能解析扩展方法：

```csharp
// At the top of the file:
using Microsoft.Extensions.DependencyInjection;
 
// When constructing your connection:
var connection = new HubConnectionBuilder()
    .AddJsonProtocol(options => {
        options.PayloadSerializerSettings.ContractResolver =
            new DefaultContractResolver();
    })
    .Build();
```

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 JSON 序列化。

### <a name="messagepack-serialization-options"></a>MessagePack 序列化选项

可以通过向 [AddMessagePackProtocol](/dotnet/api/microsoft.extensions.dependencyinjection.msgpackprotocoldependencyinjectionextensions.addmessagepackprotocol) 调用提供委托来配置 MessagePack 序列化。 有关更多详细信息，请参阅[中 SignalR 的 MessagePack](xref:signalr/messagepackhubprotocol) 。

> [!NOTE]
> 目前不能在 JavaScript 客户端中配置 MessagePack 序列化。

## <a name="configure-server-options"></a>配置服务器选项

下表描述了用于配置中心的选项 SignalR ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `HandshakeTimeout` | 15 秒 | 如果客户端在此时间间隔内未发送初始握手消息，连接将关闭。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |
| `KeepAliveInterval` | 15 秒 | 如果服务器未在此时间间隔内发送消息，则会自动发送 ping 消息，使连接保持打开状态。 更改时 `KeepAliveInterval` ，请更改 `ServerTimeout` / `serverTimeoutInMilliseconds` 客户端上的设置。 建议 `ServerTimeout` / `serverTimeoutInMilliseconds` 值为值的两倍 `KeepAliveInterval` 。  |
| `SupportedProtocols` | 所有已安装的协议 | 此中心支持的协议。 默认情况下，将允许在服务器上注册的所有协议，但可以从此列表中删除协议，以禁用各个集线器的特定协议。 |
| `EnableDetailedErrors` | `false` | 如果为，则在 `true` 集线器方法中引发异常时，详细的异常消息将返回到客户端。 默认值为 `false` ，因为这些异常消息可能包含敏感信息。 |

可以通过在中提供对调用的选项委托，为所有中心配置选项 `AddSignalR` `Startup.ConfigureServices` 。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(hubOptions =>
    {
        hubOptions.EnableDetailedErrors = true;
        hubOptions.KeepAliveInterval = TimeSpan.FromMinutes(1);
    });
}
```

单个集线器的选项用于替代和中提供的全局选项 `AddSignalR` ，可以使用进行配置 <xref:Microsoft.Extensions.DependencyInjection.SignalRDependencyInjectionExtensions.AddHubOptions*> ：

```csharp
services.AddSignalR().AddHubOptions<ChatHub>(options =>
{
    options.EnableDetailedErrors = true;
});
```

### <a name="advanced-http-configuration-options"></a>高级 HTTP 配置选项

用于 `HttpConnectionDispatcherOptions` 配置与传输和内存缓冲区管理相关的高级设置。 这些选项通过将委托传递给中的[MapHub \<T> ](/dotnet/api/microsoft.aspnetcore.signalr.hubroutebuilder.maphub)来配置 `Startup.Configure` 。

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseSignalR((configure) =>
    {
        var desiredTransports =
            HttpTransportType.WebSockets |
            HttpTransportType.LongPolling;

        configure.MapHub<ChatHub>("/chathub", (options) =>
        {
            options.Transports = desiredTransports;
        });
    });
}
```

下表描述了用于配置 ASP.NET Core SignalR 的高级 HTTP 选项的选项：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ApplicationMaxBufferSize` | 32 KB | 服务器缓冲的客户端接收到的最大字节数。 增大此值可使服务器接收更大的消息，但可能会对内存消耗产生负面影响。 |
| `AuthorizationData` | 从应用于 Hub 类的属性中自动收集的数据 `Authorize` 。 | 用于确定客户端是否有权连接到集线器的 [IAuthorizeData](/dotnet/api/microsoft.aspnetcore.authorization.iauthorizedata) 对象的列表。 |
| `TransportMaxBufferSize` | 32 KB | 由服务器缓冲的应用发送的最大字节数。 增大此值后，服务器将发送更大的消息，但会对内存消耗产生负面影响。 |
| `Transports` | 所有传输均已启用。 | 值的位标志枚举 `HttpTransportType` ，可限制客户端可用于连接的传输。 |
| `LongPolling` | 请参阅下文。 | 特定于长轮询传输的其他选项。 |
| `WebSockets` | 请参阅下文。 | 特定于 Websocket 传输的其他选项。 |

长轮询传输具有可使用属性配置的其他选项 `LongPolling` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `PollTimeout` | 90秒 | 服务器在终止单个轮询请求之前等待发送到客户端的消息的最长时间。 减小此值将导致客户端更频繁地发出新的投票请求。 |

WebSocket 传输具有可使用属性配置的其他选项 `WebSockets` ：

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `CloseTimeout` | 5 秒 | 服务器关闭后，如果客户端在此时间间隔内未能关闭，则连接将终止。 |
| `SubProtocolSelector` | `null` | 一个委托，可用于将 `Sec-WebSocket-Protocol` 标头设置为自定义值。 委托接收客户端请求的值作为输入，并且应返回所需的值。 |

## <a name="configure-client-options"></a>配置客户端选项

可以在 `HubConnectionBuilder` .net 和 JavaScript 客户端) 中可用的类型 (上配置客户端选项。 它在 Java 客户端中也可用，但 `HttpHubConnectionBuilder` 子类是包含生成器配置选项的内容，也是其 `HubConnection` 本身。

### <a name="configure-logging"></a>配置日志记录

使用方法在 .NET 客户端中配置日志记录 `ConfigureLogging` 。 日志提供程序和筛选器的注册方式与服务器上相同。 有关详细信息，请参阅 [ASP.NET Core 文档中的日志记录](xref:fundamentals/logging/index) 。

> [!NOTE]
> 若要注册日志记录提供程序，必须安装所需的包。 有关完整列表，请参阅文档的 [内置日志记录提供程序](xref:fundamentals/logging/index#built-in-logging-providers) 部分。

例如，若要启用控制台日志记录，请安装 `Microsoft.Extensions.Logging.Console` NuGet 包。 调用 `AddConsole` 扩展方法：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub")
    .ConfigureLogging(logging => {
        logging.SetMinimumLevel(LogLevel.Information);
        logging.AddConsole();
    })
    .Build();
```

在 JavaScript 客户端中，存在类似的 `configureLogging` 方法。 提供一个 `LogLevel` 值，该值指示要生成的日志消息的最小级别。 日志将写入浏览器控制台窗口中。

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .configureLogging(signalR.LogLevel.Information)
    .build();
```

> [!NOTE]
> 若要完全禁用日志记录，请 `signalR.LogLevel.None` 在方法中指定 `configureLogging` 。

有关日志记录的详细信息，请参阅[ SignalR 诊断文档](xref:signalr/diagnostics)。

SignalRJava 客户端使用[SLF4J](https://www.slf4j.org/)库进行日志记录。 这是一个高级日志记录 API，它允许库的用户通过引入特定的日志记录依赖项来选择自己的特定日志记录实现。 下面的代码段演示如何将用于 `java.util.logging` SignalR Java 客户端。

```gradle
implementation 'org.slf4j:slf4j-jdk14:1.7.25'
```

如果未在依赖项中配置日志记录，SLF4J 将加载默认的非操作记录器，并提供以下警告消息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

可以安全地忽略此情况。

### <a name="configure-allowed-transports"></a>配置允许的传输

使用的传输 SignalR 可在 `WithUrl` JavaScript) 的调用 (中进行配置 `withUrl` 。 的值的按位 "或" `HttpTransportType` 可用于将客户端限制为仅使用指定的传输。 默认情况下，将启用所有传输。

例如，禁用 Server-Sent 事件传输，但允许 Websocket 和长轮询连接：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", HttpTransportType.WebSockets | HttpTransportType.LongPolling)
    .Build();
```

在 JavaScript 客户端中，通过 `transport` 在提供的选项对象上设置字段来配置传输 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", { transport: signalR.HttpTransportType.WebSockets | signalR.HttpTransportType.LongPolling })
    .build();
```

### <a name="configure-bearer-authentication"></a>配置持有者身份验证

若要与请求一起提供身份验证数据 SignalR ，请使用 `AccessTokenProvider` `accessTokenFactory` JavaScript) 中 (选项来指定返回所需访问令牌的函数。 在 .NET 客户端中，此访问令牌作为 HTTP "持有者身份验证" 令牌传入 (使用 `Authorization`) 类型的标头 `Bearer` 。 在 JavaScript 客户端中，访问令牌用作持有者令牌， **但** 在某些情况下，浏览器 api 会限制在 Server-Sent 事件和 websocket 请求)  (具体应用标头的能力。 在这些情况下，访问令牌作为查询字符串值提供 `access_token` 。

在 .NET 客户端中， `AccessTokenProvider` 可使用中的选项委托指定选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.AccessTokenProvider = async () => {
            // Get and return the access token.
        };
    })
    .Build();
```

在 JavaScript 客户端中，通过 `accessTokenFactory` 在中设置 "选项" 对象上的字段来配置访问令牌 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        accessTokenFactory: () => {
            // Get and return the access token.
            // This function can return a JavaScript Promise if asynchronous
            // logic is required to retrieve the access token.
        }
    })
    .build();
```

在 SignalR Java 客户端中，可以通过向 [HttpHubConnectionBuilder](/java/api/com.microsoft.signalr._http_hub_connection_builder?view=aspnet-signalr-java)提供访问令牌工厂来配置用于身份验证的持有者令牌。 使用[withAccessTokenFactory](/java/api/com.microsoft.signalr._http_hub_connection_builder.withaccesstokenprovider?view=aspnet-signalr-java#com_microsoft_signalr__http_hub_connection_builder_withAccessTokenProvider_Single_String__)提供[RxJava](https://github.com/ReactiveX/RxJava) [Single \<String> ](https://reactivex.io/documentation/single.html)。 如果调用了 [单延迟](https://reactivex.io/RxJava/javadoc/io/reactivex/Single.html#defer-java.util.concurrent.Callable-)，你可以编写逻辑来为客户端生成访问令牌。

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
    .withAccessTokenProvider(Single.defer(() -> {
        // Your logic here.
        return Single.just("An Access Token");
    })).build();
```

### <a name="configure-timeout-and-keep-alive-options"></a>配置超时和 keep-alive 选项

用于配置超时和保持活动状态的其他选项可用于 `HubConnection` 对象本身：

# <a name="net"></a>[.NET](#tab/dotnet)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `ServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `HandshakeTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并 `Closed` `onclose` 在 JavaScript) 中触发事件 (。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |

在 .NET 客户端中，超时值指定为 `TimeSpan` 值。

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `serverTimeoutInMilliseconds` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onclose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |

# <a name="java"></a>[Java](#tab/java)

| 选项 | 默认值 | 说明 |
| ------ | ------------- | ----------- |
| `getServerTimeout` / `setServerTimeout` | 30秒 (30000 毫秒)  | 服务器活动超时。 如果服务器未在此时间间隔内发送消息，则客户端会将服务器视为断开连接，并触发 `onClose` 事件。 此值必须足够大，以便从服务器发送 ping 消息 **，并** 在超时间隔内由客户端接收该消息。 建议值至少为服务器值的两倍 `KeepAliveInterval` ，以允许 ping 到达的时间。 |
| `withHandshakeResponseTimeout` | 15 秒 | 初始服务器握手的超时时间。 如果服务器在此时间间隔内未发送握手响应，则客户端将取消握手，并触发 `onClose` 事件。 这是一种高级设置，只应在握手超时错误由于严重网络延迟而发生时进行修改。 有关握手过程的详细信息，请参阅[ SignalR 集线器协议规范](https://github.com/aspnet/SignalR/blob/master/specs/HubProtocol.md)。 |

---

### <a name="configure-additional-options"></a>配置其他选项

可以在中的 `WithUrl` `withUrl` JavaScript) 方法 (上 `HubConnectionBuilder` 或在 `HttpHubConnectionBuilder` Java 客户端中的各种配置 api 上配置其他选项：

# <a name="net"></a>[.NET](#tab/dotnet)

| .NET 选项 |  默认值 | 说明 |
| ----------- | -------------- | ----------- |
| `AccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `SkipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `ClientCertificates` | 空 | 要发送以对请求进行身份验证的 TLS 证书的集合。 |
| `Cookies` | 空 | cookie要随每个 http 请求一起发送的 http 的集合。 |
| `Credentials` | 空 | 要随每个 HTTP 请求一起发送的凭据。 |
| `CloseTimeout` | 5 秒 | 仅 Websocket。 客户端在关闭之后等待服务器确认关闭请求的最长时间。 如果服务器在这段时间内没有确认关闭，客户端将断开连接。 |
| `Headers` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |
| `HttpMessageHandlerFactory` | `null` | 一个委托，可用于配置或替换 `HttpMessageHandler` 用于发送 HTTP 请求的。 不用于 WebSocket 连接。 此委托必须返回非 null 值，并接收默认值作为参数。 修改该默认值的设置并将其返回，或返回一个新的 `HttpMessageHandler` 实例。 **当替换处理程序时，请确保从提供的处理程序复制您要保留的设置，否则，所配置的选项 (如 Cookie s 和标头) 将不会应用于新的处理程序。** |
| `Proxy` | `null` | 发送 HTTP 请求时要使用的 HTTP 代理。 |
| `UseDefaultCredentials` | `false` | 设置此布尔值可发送 HTTP 和 Websocket 请求的默认凭据。 这样就可以使用 Windows 身份验证。 |
| `WebSocketConfiguration` | `null` | 可用于配置其他 WebSocket 选项的委托。 接收可用于配置选项的 [ClientWebSocketOptions](/dotnet/api/system.net.websockets.clientwebsocketoptions) 实例。 |

# <a name="javascript"></a>[JavaScript](#tab/javascript)

| JavaScript 选项 | 默认值 | 说明 |
| ----------------- | ------------- | ----------- |
| `accessTokenFactory` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `transport` | `null` | 一个 <xref:Microsoft.AspNetCore.Http.Connections.HttpTransportType> 值，该值指定用于连接的传输。 |
| `logMessageContent` | `null` | 设置为 `true` 可记录客户端发送和接收的消息的字节数/字符数。 |
| `skipNegotiation` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |

# <a name="java"></a>[Java](#tab/java)

| Java 选项 | 默认值 | 说明 |
| ----------- | ------------- | ----------- |
| `withAccessTokenProvider` | `null` | 一个函数，它返回作为 HTTP 请求中的持有者身份验证令牌提供的字符串。 |
| `shouldSkipNegotiate` | `false` | 将此设置为 `true` 以跳过协商步骤。 **仅当 websocket 传输为唯一启用的传输时才受支持** 。 使用 Azure 服务时，无法启用此设置 SignalR 。 |
| `withHeader` `withHeaders` | 空 | 要随每个 HTTP 请求一起发送的附加 HTTP 标头的映射。 |

---

在 .NET 客户端中，可以通过提供给的 options 委托来修改这些选项 `WithUrl` ：

```csharp
var connection = new HubConnectionBuilder()
    .WithUrl("https://example.com/chathub", options => {
        options.Headers["Foo"] = "Bar";
        options.Cookies.Add(new Cookie(/* ... */);
        options.ClientCertificates.Add(/* ... */);
    })
    .Build();
```

在 JavaScript 客户端中，可以在提供给的 JavaScript 对象中提供这些选项 `withUrl` ：

```javascript
let connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub", {
        skipNegotiation: true,
        transport: signalR.HttpTransportType.WebSockets
    })
    .build();
```

在 Java 客户端中，这些选项可以在 `HttpHubConnectionBuilder` 从 `HubConnectionBuilder.create("HUB URL")`

```java
HubConnection hubConnection = HubConnectionBuilder.create("https://example.com/chathub")
        .withHeader("Foo", "Bar")
        .shouldSkipNegotiate(true)
        .withHandshakeResponseTimeout(30*1000)
        .build();
```

## <a name="additional-resources"></a>其他资源

* <xref:tutorials/signalr>
* <xref:signalr/hubs>
* <xref:signalr/javascript-client>
* <xref:signalr/dotnet-client>
* <xref:signalr/messagepackhubprotocol>
* <xref:signalr/supported-platforms>

::: moniker-end
