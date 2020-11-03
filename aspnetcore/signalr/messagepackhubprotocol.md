---
title: 使用中的 MessagePack Hub 协议 SignalR 进行 ASP.NET Core
author: bradygaster
description: 将 MessagePack Hub 协议添加到 ASP.NET Core SignalR 。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 09/24/2020
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
uid: signalr/messagepackhubprotocol
ms.openlocfilehash: e7d19a42e48048d2be4b87d6b0ac1ba6b2596ff1
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93058163"
---
# <a name="use-messagepack-hub-protocol-in-no-locsignalr-for-aspnet-core"></a>使用中的 MessagePack Hub 协议 SignalR 进行 ASP.NET Core

::: moniker range=">= aspnetcore-5.0"

本文假定读者 [熟悉入门中](xref:tutorials/signalr)介绍的主题。

## <a name="what-is-messagepack"></a>什么是 MessagePack？

[MessagePack](https://msgpack.org/index.html) 是一种快速、精简的二进制序列化格式。 当性能和带宽需要考虑时，它很有用，因为它会创建比 [JSON](https://www.json.org/)更小的消息。 在查看网络跟踪和日志时，不能读取二进制消息，除非这些字节是通过 MessagePack 分析器传递的。 SignalR 提供对 MessagePack 格式的内置支持，并为客户端和服务器提供要使用的 Api。

## <a name="configure-messagepack-on-the-server"></a>在服务器上配置 MessagePack

若要在服务器上启用 MessagePack 集线器协议，请 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 在应用中安装包。 在 `Startup.ConfigureServices` 方法中，将添加 `AddMessagePackProtocol` 到在 `AddSignalR` 服务器上启用 MessagePack 支持的调用。

> [!NOTE]
> 默认情况下启用 JSON。 添加 MessagePack 可支持 JSON 和 MessagePack 客户端。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol();
```

若要自定义 MessagePack 如何设置数据的格式，请 `AddMessagePackProtocol` 使用委托来配置选项。 在该委托中， `SerializerOptions` 属性可用于配置 MessagePack 序列化选项。 有关解析程序工作方式的详细信息，请访问 [MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp)上的 MessagePack 库。 属性可用于要序列化的对象，以定义应如何处理它们。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.SerializerOptions = MessagePackSerializerOptions.Standard
            .WithResolver(new CustomResolver())
            .WithSecurity(MessagePackSecurity.UntrustedData);
    });
```

> [!WARNING]
> 强烈建议查看 [CVE-2020-5234](https://github.com/neuecc/MessagePack-CSharp/security/advisories/GHSA-7q36-4xx7-xcxf) 和应用建议的修补程序。 例如， `.WithSecurity(MessagePackSecurity.UntrustedData)` 在替换时调用 `SerializerOptions` 。

## <a name="configure-messagepack-on-the-client"></a>在客户端上配置 MessagePack

> [!NOTE]
> 默认情况下，为支持的客户端启用 JSON。 客户端只能支持一个协议。 添加 MessagePack 支持将替换任何以前配置的协议。

### <a name="net-client"></a>.NET 客户端

若要在 .NET 客户端中启用 MessagePack，请 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 在上安装包并调用 `AddMessagePackProtocol` `HubConnectionBuilder` 。

```csharp
using Microsoft.AspNetCore.SignalR.Client;
using Microsoft.Extensions.DependencyInjection;

var hubConnection = new HubConnectionBuilder()
                        .WithUrl("/chathub")
                        .AddMessagePackProtocol()
                        .Build();
```

> [!NOTE]
> 此 `AddMessagePackProtocol` 调用采用一个委托来配置与服务器类似的选项。

### <a name="javascript-client"></a>JavaScript 客户端

MessagePack 支持 JavaScript 客户端由 [@microsoft/signalr-protocol-msgpack](https://www.npmjs.com/package/@microsoft/signalr-protocol-msgpack) npm 包提供。 通过在命令行界面中执行以下命令来安装包：

```bash
npm install @microsoft/signalr-protocol-msgpack
```

安装 npm 包后，可以通过 JavaScript 模块加载程序直接使用该模块，或通过引用以下文件将该模块导入到浏览器中：

*node_modules\\@microsoft\signalr-protocol-msgpack\dist\browser\signalr-protocol-msgpack.js* 

在浏览器中， `msgpack5` 还必须引用库。 使用 `<script>` 标记创建引用。 可在 *node_modules\msgpack5\dist\msgpack5.js* 中找到库。

> [!NOTE]
> 使用元素时 `<script>` ，顺序很重要。 如果在 *msgpack5.js* 之前引用 *signalr-protocol-msgpack.js* ，则在尝试使用 MessagePack 进行连接时将出现错误。 *signalr-protocol-msgpack.js* 之前也需要 *signalr.js* 。

```html
<script src="~/lib/signalr/signalr.js"></script>
<script src="~/lib/msgpack5/msgpack5.js"></script>
<script src="~/lib/signalr/signalr-protocol-msgpack.js"></script>
```

`.withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())`如果添加到，则 `HubConnectionBuilder` 会将客户端配置为在连接到服务器时使用 MessagePack 协议。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
    .build();
```

> [!NOTE]
> 目前，JavaScript 客户端上没有用于 MessagePack 协议的配置选项。

## <a name="messagepack-quirks"></a>MessagePack 兼容

使用 MessagePack 集线器协议时，需要注意几个问题。

### <a name="messagepack-is-case-sensitive"></a>MessagePack 区分大小写

MessagePack 协议区分大小写。 例如，请考虑以下 c # 类：

```csharp
public class ChatMessage
{
    public string Sender { get; }
    public string Message { get; }
}
```

从 JavaScript 客户端发送时，必须使用 `PascalCased` 属性名称，因为大小写必须与 c # 类完全匹配。 例如： 。

```javascript
connection.invoke("SomeMethod", { Sender: "Sally", Message: "Hello!" });
```

使用 `camelCased` 名称无法正确绑定到 c # 类。 可以通过使用 `Key` 属性为 MessagePack 属性指定不同的名称来解决此情况。 有关详细信息，请参阅 [MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp#object-serialization)。

### <a name="datetimekind-is-not-preserved-when-serializingdeserializing"></a>序列化/反序列化时不保留 DateTime. Kind

MessagePack 协议不提供对的值进行编码的方法 `Kind` `DateTime` 。 因此，在对日期进行反序列化时，如果为，则 MessagePack 集线器协议会转换为 UTC 格式， `DateTime.Kind` `DateTimeKind.Local` 否则它将不会触及时间并按原样传递它。 如果你使用的是 `DateTime` 值，则建议在发送这些值前将其转换为 UTC。 接收到本地时间时将它们从 UTC 转换为本地时间。

### <a name="datetimeminvalue-is-not-supported-by-messagepack-in-javascript"></a>JavaScript 中的 MessagePack 不支持 MinValue

JavaScript 客户端使用的 [msgpack5](https://github.com/mcollina/msgpack5) 库 SignalR 不支持 `timestamp96` MessagePack 中的类型。 此类型用于对非常大的日期值进行编码， (在将来或在未来) 中非常早的时间。 的值 `DateTime.MinValue` 为 `January 1, 0001` ，必须在值中对其进行编码 `timestamp96` 。 因此， `DateTime.MinValue` 不支持向 JavaScript 客户端发送发送。 当 `DateTime.MinValue` JavaScript 客户端收到时，将引发以下错误：

```
Uncaught Error: unable to find ext type 255 at decoder.js:427
```

通常， `DateTime.MinValue` 用于对 "缺失" 或值进行编码 `null` 。 如果需要在 MessagePack 中对该值进行编码，请使用 () 的可以为 null 的 `DateTime` 值， `DateTime?` 或对 `bool` 指示日期是否存在的单独值进行编码。

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2228](https://github.com/aspnet/SignalR/issues/2228)。

### <a name="messagepack-support-in-ahead-of-time-compilation-environment"></a>"提前" 编译环境中的 MessagePack 支持

.NET 客户端和服务器使用的 [MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp/tree/v2.1.90) 库使用代码生成来优化序列化。 因此，默认情况下，在使用 "预先" 编译 (（如 Xamarin iOS 或 Unity) ）的环境中不支持默认值。 可以通过 "预生成" 序列化程序/反序列化程序代码，在这些环境中使用 MessagePack。 有关详细信息，请参阅 [MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp/tree/v2.1.90#aot-code-generation-to-support-unityxamarin)。 预生成序列化程序后，可以使用传递给的配置委托注册它们 `AddMessagePackProtocol` ：

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        StaticCompositeResolver.Instance.Register(
            MessagePack.Resolvers.GeneratedResolver.Instance,
            MessagePack.Resolvers.StandardResolver.Instance
        );
        options.SerializerOptions = MessagePackSerializerOptions.Standard
            .WithResolver(StaticCompositeResolver.Instance)
            .WithSecurity(MessagePackSecurity.UntrustedData);
    });
```

### <a name="type-checks-are-more-strict-in-messagepack"></a>类型检查在 MessagePack 中更加严格

JSON 集线器协议将在反序列化过程中执行类型转换。 例如，如果传入的对象的属性值是 (的数字 `{ foo: 42 }`) 但 .net 类的属性的类型为 `string` ，则将转换该值。 但是，MessagePack 不会执行此转换，并将引发异常，该异常可在服务器端日志 (和控制台) 中出现：

```
InvalidDataException: Error binding arguments. Make sure that the types of the provided values match the types of the hub method being invoked.
```

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2937](https://github.com/aspnet/SignalR/issues/2937)。

## <a name="related-resources"></a>相关资源

* [入门](xref:tutorials/signalr)
* [.NET 客户端](xref:signalr/dotnet-client)
* [JavaScript 客户端](xref:signalr/javascript-client)

::: moniker-end

::: moniker range=">= aspnetcore-3.0 < aspnetcore-5.0"

本文假定读者 [熟悉入门中](xref:tutorials/signalr)介绍的主题。

## <a name="what-is-messagepack"></a>什么是 MessagePack？

[MessagePack](https://msgpack.org/index.html) 是一种快速、精简的二进制序列化格式。 当性能和带宽需要考虑时，它很有用，因为它会创建比 [JSON](https://www.json.org/)更小的消息。 在查看网络跟踪和日志时，不能读取二进制消息，除非这些字节是通过 MessagePack 分析器传递的。 SignalR 提供对 MessagePack 格式的内置支持，并为客户端和服务器提供要使用的 Api。

## <a name="configure-messagepack-on-the-server"></a>在服务器上配置 MessagePack

若要在服务器上启用 MessagePack 集线器协议，请 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 在应用中安装包。 在 `Startup.ConfigureServices` 方法中，将添加 `AddMessagePackProtocol` 到在 `AddSignalR` 服务器上启用 MessagePack 支持的调用。

> [!NOTE]
> 默认情况下启用 JSON。 添加 MessagePack 可支持 JSON 和 MessagePack 客户端。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol();
```

若要自定义 MessagePack 如何设置数据的格式，请 `AddMessagePackProtocol` 使用委托来配置选项。 在该委托中， `FormatterResolvers` 属性可用于配置 MessagePack 序列化选项。 有关解析程序工作方式的详细信息，请访问 [MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp)上的 MessagePack 库。 属性可用于要序列化的对象，以定义应如何处理它们。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.FormatterResolvers = new List<MessagePack.IFormatterResolver>()
        {
            MessagePack.Resolvers.StandardResolver.Instance
        };
    });
```

> [!WARNING]
> 强烈建议查看 [CVE-2020-5234](https://github.com/neuecc/MessagePack-CSharp/security/advisories/GHSA-7q36-4xx7-xcxf) 和应用建议的修补程序。 例如，将 `MessagePackSecurity.Active` 静态属性设置为 `MessagePackSecurity.UntrustedData` 。 设置 `MessagePackSecurity.Active` 需要手动安装[MessagePack 的1.9 版。](https://www.nuget.org/packages/MessagePack/1.9.3) 正在安装 `MessagePack` 版本为的 1.9. x 的升级 SignalR 。 `MessagePack` 版本2.x 引入了重大更改，与 SignalR 版本3.1 及更早版本不兼容。 如果 `MessagePackSecurity.Active` 未设置为 `MessagePackSecurity.UntrustedData` ，则恶意客户端可能会导致拒绝服务。 `MessagePackSecurity.Active`在中设置 `Program.Main` ，如下面的代码所示：

```csharp
using MessagePack;

public static void Main(string[] args)
{
  MessagePackSecurity.Active = MessagePackSecurity.UntrustedData;

  CreateHostBuilder(args).Build().Run();
}
```

## <a name="configure-messagepack-on-the-client"></a>在客户端上配置 MessagePack

> [!NOTE]
> 默认情况下，为支持的客户端启用 JSON。 客户端只能支持一个协议。 添加 MessagePack 支持将替换任何以前配置的协议。

### <a name="net-client"></a>.NET 客户端

若要在 .NET 客户端中启用 MessagePack，请 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 在上安装包并调用 `AddMessagePackProtocol` `HubConnectionBuilder` 。

```csharp
using Microsoft.AspNetCore.SignalR.Client;
using Microsoft.Extensions.DependencyInjection;

var hubConnection = new HubConnectionBuilder()
                        .WithUrl("/chathub")
                        .AddMessagePackProtocol()
                        .Build();
```

> [!NOTE]
> 此 `AddMessagePackProtocol` 调用采用一个委托来配置与服务器类似的选项。

### <a name="javascript-client"></a>JavaScript 客户端

MessagePack 支持 JavaScript 客户端由 [@microsoft/signalr-protocol-msgpack](https://www.npmjs.com/package/@microsoft/signalr-protocol-msgpack) npm 包提供。 通过在命令行界面中执行以下命令来安装包：

```bash
npm install @microsoft/signalr-protocol-msgpack
```

安装 npm 包后，可以通过 JavaScript 模块加载程序直接使用该模块，或通过引用以下文件将该模块导入到浏览器中：

*node_modules\\@microsoft\signalr-protocol-msgpack\dist\browser\signalr-protocol-msgpack.js* 

在浏览器中， `msgpack5` 还必须引用库。 使用 `<script>` 标记创建引用。 可在 *node_modules\msgpack5\dist\msgpack5.js* 中找到库。

> [!NOTE]
> 使用元素时 `<script>` ，顺序很重要。 如果在 *msgpack5.js* 之前引用 *signalr-protocol-msgpack.js* ，则在尝试使用 MessagePack 进行连接时将出现错误。 *signalr-protocol-msgpack.js* 之前也需要 *signalr.js* 。

```html
<script src="~/lib/signalr/signalr.js"></script>
<script src="~/lib/msgpack5/msgpack5.js"></script>
<script src="~/lib/signalr/signalr-protocol-msgpack.js"></script>
```

`.withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())`如果添加到，则 `HubConnectionBuilder` 会将客户端配置为在连接到服务器时使用 MessagePack 协议。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
    .build();
```

> [!NOTE]
> 目前，JavaScript 客户端上没有用于 MessagePack 协议的配置选项。

## <a name="messagepack-quirks"></a>MessagePack 兼容

使用 MessagePack 集线器协议时，需要注意几个问题。

### <a name="messagepack-is-case-sensitive"></a>MessagePack 区分大小写

MessagePack 协议区分大小写。 例如，请考虑以下 c # 类：

```csharp
public class ChatMessage
{
    public string Sender { get; }
    public string Message { get; }
}
```

从 JavaScript 客户端发送时，必须使用 `PascalCased` 属性名称，因为大小写必须与 c # 类完全匹配。 例如： 。

```javascript
connection.invoke("SomeMethod", { Sender: "Sally", Message: "Hello!" });
```

使用 `camelCased` 名称无法正确绑定到 c # 类。 可以通过使用 `Key` 属性为 MessagePack 属性指定不同的名称来解决此情况。 有关详细信息，请参阅 [MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp#object-serialization)。

### <a name="datetimekind-is-not-preserved-when-serializingdeserializing"></a>序列化/反序列化时不保留 DateTime. Kind

MessagePack 协议不提供对的值进行编码的方法 `Kind` `DateTime` 。 因此，在对日期进行反序列化时，MessagePack Hub 协议假设传入日期为 UTC 格式。 如果使用的是 `DateTime` 本地时间的值，建议在发送之前将值转换为 UTC。 接收到本地时间时将它们从 UTC 转换为本地时间。

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2632](https://github.com/aspnet/SignalR/issues/2632)。

### <a name="datetimeminvalue-is-not-supported-by-messagepack-in-javascript"></a>JavaScript 中的 MessagePack 不支持 MinValue

JavaScript 客户端使用的 [msgpack5](https://github.com/mcollina/msgpack5) 库 SignalR 不支持 `timestamp96` MessagePack 中的类型。 此类型用于对非常大的日期值进行编码， (在将来或在未来) 中非常早的时间。 的值 `DateTime.MinValue` 为 `January 1, 0001` ，必须在值中对其进行编码 `timestamp96` 。 因此， `DateTime.MinValue` 不支持向 JavaScript 客户端发送发送。 当 `DateTime.MinValue` JavaScript 客户端收到时，将引发以下错误：

```
Uncaught Error: unable to find ext type 255 at decoder.js:427
```

通常， `DateTime.MinValue` 用于对 "缺失" 或值进行编码 `null` 。 如果需要在 MessagePack 中对该值进行编码，请使用 () 的可以为 null 的 `DateTime` 值， `DateTime?` 或对 `bool` 指示日期是否存在的单独值进行编码。

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2228](https://github.com/aspnet/SignalR/issues/2228)。

### <a name="messagepack-support-in-ahead-of-time-compilation-environment"></a>"提前" 编译环境中的 MessagePack 支持

.NET 客户端和服务器使用的 [MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp/tree/v1.8.80) 库使用代码生成来优化序列化。 因此，默认情况下，在使用 "预先" 编译 (（如 Xamarin iOS 或 Unity) ）的环境中不支持默认值。 可以通过 "预生成" 序列化程序/反序列化程序代码，在这些环境中使用 MessagePack。 有关详细信息，请参阅 [MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp/tree/v1.8.80#pre-code-generationunityxamarin-supports)。 预生成序列化程序后，可以使用传递给的配置委托注册它们 `AddMessagePackProtocol` ：

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.FormatterResolvers = new List<MessagePack.IFormatterResolver>()
        {
            MessagePack.Resolvers.GeneratedResolver.Instance,
            MessagePack.Resolvers.StandardResolver.Instance
        };
    });
```

### <a name="type-checks-are-more-strict-in-messagepack"></a>类型检查在 MessagePack 中更加严格

JSON 集线器协议将在反序列化过程中执行类型转换。 例如，如果传入的对象的属性值是 (的数字 `{ foo: 42 }`) 但 .net 类的属性的类型为 `string` ，则将转换该值。 但是，MessagePack 不会执行此转换，并将引发异常，该异常可在服务器端日志 (和控制台) 中出现：

```
InvalidDataException: Error binding arguments. Make sure that the types of the provided values match the types of the hub method being invoked.
```

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2937](https://github.com/aspnet/SignalR/issues/2937)。

## <a name="related-resources"></a>相关资源

* [入门](xref:tutorials/signalr)
* [.NET 客户端](xref:signalr/dotnet-client)
* [JavaScript 客户端](xref:signalr/javascript-client)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本文假定读者 [熟悉入门中](xref:tutorials/signalr)介绍的主题。

## <a name="what-is-messagepack"></a>什么是 MessagePack？

[MessagePack](https://msgpack.org/index.html) 是一种快速、精简的二进制序列化格式。 当性能和带宽需要考虑时，它很有用，因为它会创建比 [JSON](https://www.json.org/)更小的消息。 在查看网络跟踪和日志时，不能读取二进制消息，除非这些字节是通过 MessagePack 分析器传递的。 SignalR 提供对 MessagePack 格式的内置支持，并为客户端和服务器提供要使用的 Api。

## <a name="configure-messagepack-on-the-server"></a>在服务器上配置 MessagePack

若要在服务器上启用 MessagePack 集线器协议，请 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 在应用中安装包。 在 `Startup.ConfigureServices` 方法中，将添加 `AddMessagePackProtocol` 到在 `AddSignalR` 服务器上启用 MessagePack 支持的调用。

> [!NOTE]
> 默认情况下启用 JSON。 添加 MessagePack 可支持 JSON 和 MessagePack 客户端。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol();
```

若要自定义 MessagePack 如何设置数据的格式，请 `AddMessagePackProtocol` 使用委托来配置选项。 在该委托中， `FormatterResolvers` 属性可用于配置 MessagePack 序列化选项。 有关解析程序工作方式的详细信息，请访问 [MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp)上的 MessagePack 库。 属性可用于要序列化的对象，以定义应如何处理它们。

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.FormatterResolvers = new List<MessagePack.IFormatterResolver>()
        {
            MessagePack.Resolvers.StandardResolver.Instance
        };
    });
```

> [!WARNING]
> 强烈建议查看 [CVE-2020-5234](https://github.com/neuecc/MessagePack-CSharp/security/advisories/GHSA-7q36-4xx7-xcxf) 和应用建议的修补程序。 例如，将 `MessagePackSecurity.Active` 静态属性设置为 `MessagePackSecurity.UntrustedData` 。 设置 `MessagePackSecurity.Active` 需要手动安装[MessagePack 的1.9 版。](https://www.nuget.org/packages/MessagePack/1.9.3) 正在安装 `MessagePack` 版本为的 1.9. x 的升级 SignalR 。 当未 `MessagePackSecurity.Active` 设置为时 `MessagePackSecurity.UntrustedData` ，恶意客户端可能会导致拒绝服务。 `MessagePackSecurity.Active`在中设置 `Program.Main` ，如下面的代码所示：

```csharp
using MessagePack;

public static void Main(string[] args)
{
  MessagePackSecurity.Active = MessagePackSecurity.UntrustedData;

  CreateHostBuilder(args).Build().Run();
}
```

## <a name="configure-messagepack-on-the-client"></a>在客户端上配置 MessagePack

> [!NOTE]
> 默认情况下，为支持的客户端启用 JSON。 客户端只能支持一个协议。 添加 MessagePack 支持将替换任何以前配置的协议。

### <a name="net-client"></a>.NET 客户端

若要在 .NET 客户端中启用 MessagePack，请 `Microsoft.AspNetCore.SignalR.Protocols.MessagePack` 在上安装包并调用 `AddMessagePackProtocol` `HubConnectionBuilder` 。

```csharp
using Microsoft.AspNetCore.SignalR.Client;
using Microsoft.Extensions.DependencyInjection;

var hubConnection = new HubConnectionBuilder()
                        .WithUrl("/chathub")
                        .AddMessagePackProtocol()
                        .Build();
```

> [!NOTE]
> 此 `AddMessagePackProtocol` 调用采用一个委托来配置与服务器类似的选项。

### <a name="javascript-client"></a>JavaScript 客户端

MessagePack 支持 JavaScript 客户端由 [@aspnet/signalr-protocol-msgpack](https://www.npmjs.com/package/@aspnet/signalr-protocol-msgpack) npm 包提供。 通过在命令行界面中执行以下命令来安装包：

```bash
npm install @aspnet/signalr-protocol-msgpack
```

安装 npm 包后，可以通过 JavaScript 模块加载程序直接使用该模块，或通过引用以下文件将该模块导入到浏览器中：

*node_modules\\@aspnet\signalr-protocol-msgpack\dist\browser\signalr-protocol-msgpack.js*

在浏览器中， `msgpack5` 还必须引用库。 使用 `<script>` 标记创建引用。 可在 *node_modules\msgpack5\dist\msgpack5.js* 中找到库。

> [!NOTE]
> 使用元素时 `<script>` ，顺序很重要。 如果在 *msgpack5.js* 之前引用 *signalr-protocol-msgpack.js* ，则在尝试使用 MessagePack 进行连接时将出现错误。 *signalr-protocol-msgpack.js* 之前也需要 *signalr.js* 。

```html
<script src="~/lib/signalr/signalr.js"></script>
<script src="~/lib/msgpack5/msgpack5.js"></script>
<script src="~/lib/signalr/signalr-protocol-msgpack.js"></script>
```

`.withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())`如果添加到，则 `HubConnectionBuilder` 会将客户端配置为在连接到服务器时使用 MessagePack 协议。

```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chathub")
    .withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
    .build();
```

> [!NOTE]
> 目前，JavaScript 客户端上没有用于 MessagePack 协议的配置选项。

## <a name="messagepack-quirks"></a>MessagePack 兼容

使用 MessagePack 集线器协议时，需要注意几个问题。

### <a name="messagepack-is-case-sensitive"></a>MessagePack 区分大小写

MessagePack 协议区分大小写。 例如，请考虑以下 c # 类：

```csharp
public class ChatMessage
{
    public string Sender { get; }
    public string Message { get; }
}
```

从 JavaScript 客户端发送时，必须使用 `PascalCased` 属性名称，因为大小写必须与 c # 类完全匹配。 例如： 。

```javascript
connection.invoke("SomeMethod", { Sender: "Sally", Message: "Hello!" });
```

使用 `camelCased` 名称无法正确绑定到 c # 类。 可以通过使用 `Key` 属性为 MessagePack 属性指定不同的名称来解决此情况。 有关详细信息，请参阅 [MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp#object-serialization)。

### <a name="datetimekind-is-not-preserved-when-serializingdeserializing"></a>序列化/反序列化时不保留 DateTime. Kind

MessagePack 协议不提供对的值进行编码的方法 `Kind` `DateTime` 。 因此，在对日期进行反序列化时，MessagePack Hub 协议假设传入日期为 UTC 格式。 如果使用的是 `DateTime` 本地时间的值，建议在发送之前将值转换为 UTC。 接收到本地时间时将它们从 UTC 转换为本地时间。

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2632](https://github.com/aspnet/SignalR/issues/2632)。

### <a name="datetimeminvalue-is-not-supported-by-messagepack-in-javascript"></a>JavaScript 中的 MessagePack 不支持 MinValue

JavaScript 客户端使用的 [msgpack5](https://github.com/mcollina/msgpack5) 库 SignalR 不支持 `timestamp96` MessagePack 中的类型。 此类型用于对非常大的日期值进行编码， (在将来或在未来) 中非常早的时间。 的值 `DateTime.MinValue` `January 1, 0001` 必须在值中进行编码 `timestamp96` 。 因此， `DateTime.MinValue` 不支持向 JavaScript 客户端发送发送。 当 `DateTime.MinValue` JavaScript 客户端收到时，将引发以下错误：

```
Uncaught Error: unable to find ext type 255 at decoder.js:427
```

通常， `DateTime.MinValue` 用于对 "缺失" 或值进行编码 `null` 。 如果需要在 MessagePack 中对该值进行编码，请使用 () 的可以为 null 的 `DateTime` 值， `DateTime?` 或对 `bool` 指示日期是否存在的单独值进行编码。

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2228](https://github.com/aspnet/SignalR/issues/2228)。

### <a name="messagepack-support-in-ahead-of-time-compilation-environment"></a>"提前" 编译环境中的 MessagePack 支持

.NET 客户端和服务器使用的 [MessagePack-CSharp](https://github.com/neuecc/MessagePack-CSharp/tree/v1.8.80) 库使用代码生成来优化序列化。 因此，默认情况下，在使用 "预先" 编译 (（如 Xamarin iOS 或 Unity) ）的环境中不支持默认值。 可以通过 "预生成" 序列化程序/反序列化程序代码，在这些环境中使用 MessagePack。 有关详细信息，请参阅 [MessagePack-CSharp 文档](https://github.com/neuecc/MessagePack-CSharp/tree/v1.8.80#pre-code-generationunityxamarin-supports)。 预生成序列化程序后，可以使用传递给的配置委托注册它们 `AddMessagePackProtocol` ：

```csharp
services.AddSignalR()
    .AddMessagePackProtocol(options =>
    {
        options.FormatterResolvers = new List<MessagePack.IFormatterResolver>()
        {
            MessagePack.Resolvers.GeneratedResolver.Instance,
            MessagePack.Resolvers.StandardResolver.Instance
        };
    });
```

### <a name="type-checks-are-more-strict-in-messagepack"></a>类型检查在 MessagePack 中更加严格

JSON 集线器协议将在反序列化过程中执行类型转换。 例如，如果传入的对象的属性值是 (的数字 `{ foo: 42 }`) 但 .net 类的属性的类型为 `string` ，则将转换该值。 但是，MessagePack 不会执行此转换，并将引发异常，该异常可在服务器端日志 (和控制台) 中出现：

```
InvalidDataException: Error binding arguments. Make sure that the types of the provided values match the types of the hub method being invoked.
```

有关此限制的详细信息，请参阅 GitHub 颁发 [aspnet/ SignalR #2937](https://github.com/aspnet/SignalR/issues/2937)。

## <a name="related-resources"></a>相关资源

* [入门](xref:tutorials/signalr)
* [.NET 客户端](xref:signalr/dotnet-client)
* [JavaScript 客户端](xref:signalr/javascript-client)

::: moniker-end
