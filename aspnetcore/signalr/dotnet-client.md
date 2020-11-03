---
title: ASP.NET Core SignalR .Net 客户端
author: bradygaster
description: 有关 ASP.NET Core SignalR .Net 客户端的信息
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 01/14/2020
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
uid: signalr/dotnet-client
ms.openlocfilehash: 7849322e50aeceda0850d94f9c9bcda79a6051c8
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061218"
---
# <a name="aspnet-core-no-locsignalr-net-client"></a>ASP.NET Core SignalR .Net 客户端

通过 ASP.NET Core SignalR .net 客户端库，你可以 SignalR 从 .net 应用程序与中心进行通信。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/signalr/dotnet-client/sample)（[如何下载](xref:index#how-to-download-a-sample)）

本文中的代码示例是使用 .Net 客户端 ASP.NET Core 的 WPF 应用程序 SignalR 。

## <a name="install-the-no-locsignalr-net-client-package"></a>安装 SignalR .net 客户端包

[AspNetCore. SignalR](https://www.nuget.org/packages/Microsoft.AspNetCore.SignalR.Client)若要连接到集线器，.net 客户端需要客户端包 SignalR 。

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

若要安装客户端库，请在 " **包管理器控制台** " 窗口中运行以下命令：

```powershell
Install-Package Microsoft.AspNetCore.SignalR.Client
```

# <a name="net-core-cli"></a>[.NET Core CLI](#tab/netcore-cli)

若要安装客户端库，请在命令 shell 中运行以下命令：

```dotnetcli
dotnet add package Microsoft.AspNetCore.SignalR.Client
```

---

## <a name="connect-to-a-hub"></a>连接到集线器

若要建立连接，请创建 `HubConnectionBuilder` 并调用 `Build` 。 在建立连接时，可以配置中心 URL、协议、传输类型、日志级别、标头和其他选项。 通过在中插入任意方法来配置任何所需的选项 `HubConnectionBuilder` `Build` 。 启动与的连接 `StartAsync` 。

[!code-csharp[Build hub connection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_MainWindowClass&highlight=15-17,39)]

## <a name="handle-lost-connection"></a>处理丢失的连接

::: moniker range=">= aspnetcore-3.0"

### <a name="automatically-reconnect"></a>自动重新连接

<xref:Microsoft.AspNetCore.SignalR.Client.HubConnection>可以将配置为使用上的方法自动重新连接 `WithAutomaticReconnect` <xref:Microsoft.AspNetCore.SignalR.Client.HubConnectionBuilder> 。 默认情况下，它不会自动重新连接。

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chathub"))
    .WithAutomaticReconnect()
    .Build();
```

在没有任何参数的情况下，会 `WithAutomaticReconnect()` 将客户端配置为分别等待0、2、10和30秒，然后再尝试重新连接尝试。

在开始任何重新连接尝试之前， `HubConnection` 将转换为 `HubConnectionState.Reconnecting` 状态，并触发 `Reconnecting` 事件。  这为用户提供警告连接已丢失并禁用 UI 元素的机会。 非交互式应用可以开始排队或删除消息。

```csharp
connection.Reconnecting += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Reconnecting);

    // Notify users the connection was lost and the client is reconnecting.
    // Start queuing or dropping messages.

    return Task.CompletedTask;
};
```

如果客户端在其前四次尝试内成功重新连接，则 `HubConnection` 将转换回 `Connected` 状态并激发 `Reconnected` 事件。 这为用户提供了通知用户已重新建立连接并取消排队消息的排队的机会。

由于连接在服务器上看起来是全新的，因此 `ConnectionId` 将向事件处理程序提供一个新的 `Reconnected` 。

> [!WARNING]
> `Reconnected` `connectionId` 如果 `HubConnection` 配置为[跳过协商](xref:signalr/configuration#configure-client-options)，事件处理程序的参数将为 null。

```csharp
connection.Reconnected += connectionId =>
{
    Debug.Assert(connection.State == HubConnectionState.Connected);

    // Notify users the connection was reestablished.
    // Start dequeuing messages queued while reconnecting if any.

    return Task.CompletedTask;
};
```

`WithAutomaticReconnect()` 不会将配置 `HubConnection` 为重试初始启动失败，因此，需要手动处理启动失败：

```csharp
public static async Task<bool> ConnectWithRetryAsync(HubConnection connection, CancellationToken token)
{
    // Keep trying to until we can start or the token is canceled.
    while (true)
    {
        try
        {
            await connection.StartAsync(token);
            Debug.Assert(connection.State == HubConnectionState.Connected);
            return true;
        }
        catch when (token.IsCancellationRequested)
        {
            return false;
        }
        catch
        {
            // Failed to connect, trying again in 5000 ms.
            Debug.Assert(connection.State == HubConnectionState.Disconnected);
            await Task.Delay(5000);
        }
    }
}
```

如果客户端在其前四次尝试中未成功重新连接，则 `HubConnection` 将转换为 `Disconnected` 状态并触发 <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed> 事件。 这为尝试手动重新启动连接或通知用户连接永久丢失有机会。

```csharp
connection.Closed += error =>
{
    Debug.Assert(connection.State == HubConnectionState.Disconnected);

    // Notify users the connection has been closed or manually try to restart the connection.

    return Task.CompletedTask;
};
```

若要在断开连接或更改重新连接时间安排之前配置自定义的重新连接尝试次数，请 `WithAutomaticReconnect` 接受一个数字数组，表示在开始每次重新连接尝试之前等待的延迟（以毫秒为单位）。

```csharp
HubConnection connection= new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chathub"))
    .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.Zero, TimeSpan.FromSeconds(10) })
    .Build();

    // .WithAutomaticReconnect(new[] { TimeSpan.Zero, TimeSpan.FromSeconds(2), TimeSpan.FromSeconds(10), TimeSpan.FromSeconds(30) }) yields the default behavior.
```

前面的示例将配置 `HubConnection` 为在连接丢失后立即开始尝试重新连接。 这也适用于默认配置。

如果第一次重新连接尝试失败，则第二次重新连接尝试还会立即启动，而不是等待2秒，就像在默认配置中一样。

如果第二次重新连接尝试失败，则第三次重新连接尝试将在10秒内启动，这与默认配置相同。

然后，在第三次重新连接尝试失败后，自定义行为将再次从默认行为与其分离。 在默认配置中，将在另一个30秒内再次尝试重新连接。

如果需要更好地控制计时和自动重新连接尝试的次数，则 `WithAutomaticReconnect` 接受一个实现接口的 `IRetryPolicy` 对象，该对象具有一个名为的方法 `NextRetryDelay` 。

`NextRetryDelay` 采用类型为的单个自变量 `RetryContext` 。 `RetryContext`有三个属性： `PreviousRetryCount` 、 `ElapsedTime` 和 `RetryReason` ， `long` 分别为、和 `TimeSpan` `Exception` 。 第一次重新连接尝试之前， `PreviousRetryCount` 和均 `ElapsedTime` 为零， `RetryReason` 将是导致连接丢失的异常。 每次失败的重试次数递增一次后，将 `PreviousRetryCount` `ElapsedTime` 更新以反映到目前为止所用的时间， `RetryReason` 这是导致上次重新连接尝试失败的异常。

`NextRetryDelay` 必须返回一个 TimeSpan，表示在下一次重新连接尝试之前要等待的时间，或者 `null` ，如果应停止重新连接，则为 `HubConnection` 。

```csharp
public class RandomRetryPolicy : IRetryPolicy
{
    private readonly Random _random = new Random();

    public TimeSpan? NextRetryDelay(RetryContext retryContext)
    {
        // If we've been reconnecting for less than 60 seconds so far,
        // wait between 0 and 10 seconds before the next reconnect attempt.
        if (retryContext.ElapsedTime < TimeSpan.FromSeconds(60))
        {
            return TimeSpan.FromSeconds(_random.NextDouble() * 10);
        }
        else
        {
            // If we've been reconnecting for more than 60 seconds so far, stop reconnecting.
            return null;
        }
    }
}
```

```csharp
HubConnection connection = new HubConnectionBuilder()
    .WithUrl(new Uri("http://127.0.0.1:5000/chathub"))
    .WithAutomaticReconnect(new RandomRetryPolicy())
    .Build();
```

或者，你可以编写将手动重新连接客户端的代码，如 [手动重新连接](#manually-reconnect)中所示。

::: moniker-end

### <a name="manually-reconnect"></a>手动重新连接

::: moniker range="< aspnetcore-3.0"

> [!WARNING]
> 在3.0 之前，.NET 客户端 SignalR 不会自动重新连接。 必须编写代码来手动重新连接客户端。

::: moniker-end

使用 <xref:Microsoft.AspNetCore.SignalR.Client.HubConnection.Closed> 事件响应断开连接。 例如，你可能需要自动重新连接。

`Closed`事件需要一个返回的委托 `Task` ，该委托允许异步代码在不使用的情况下运行 `async void` 。 若要在 `Closed` 同步运行的事件处理程序中满足委托签名，请返回 `Task.CompletedTask` ：

```csharp
connection.Closed += (error) => {
    // Do your close logic.
    return Task.CompletedTask;
};
```

异步支持的主要原因是，可以重启连接。 启动连接是一种异步操作。

在 `Closed` 重启连接的处理程序中，考虑等待一些随机延迟以防止服务器过载，如以下示例中所示：

[!code-csharp[Use Closed event handler to automate reconnection](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ClosedRestart)]

## <a name="call-hub-methods-from-client"></a>从客户端调用集线器方法

`InvokeAsync` 在集线器上调用方法。 将 hub 方法名称和在 hub 方法中定义的任何自变量传递给 `InvokeAsync` 。 SignalR 是异步的，因此 `async` 在 `await` 进行调用时使用和。

[!code-csharp[InvokeAsync method](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_InvokeAsync)]

`InvokeAsync`方法返回一个在 `Task` 服务器方法返回时完成的。 返回值（如果有）作为的结果提供 `Task` 。 服务器上的方法所引发的任何异常都将导致出错 `Task` 。 使用 `await` 语法等待服务器方法完成，并使用 `try...catch` 语法来处理错误。

`SendAsync`方法返回一个，该方法在 `Task` 消息已发送到服务器时完成。 不会提供返回值，因为它 `Task` 不会等到服务器方法完成。 发送消息时，在客户端上引发的任何异常都会产生出错 `Task` 。 使用 `await` 和 `try...catch` 语法处理发送错误。

> [!NOTE]
> 仅 SignalR 在 *默认* 模式下使用 Azure 服务时，才支持从客户端调用中心方法。 有关详细信息，请参阅 [Signalr GitHub 存储库)  (常见问题解答 ](https://github.com/Azure/azure-signalr/blob/dev/docs/faq.md#what-is-the-meaning-of-service-mode-defaultserverlessclassic-how-can-i-choose)。

## <a name="call-client-methods-from-hub"></a>从中心调用客户端方法

定义集线器在 `connection.On` 生成之后、启动连接之前调用的方法。

[!code-csharp[Define client methods](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ConnectionOn)]

`connection.On`当服务器端代码使用方法调用时，上面的代码将运行 `SendAsync` 。

[!code-csharp[Call client method](dotnet-client/sample/signalrchat/hubs/chathub.cs?name=snippet_SendMessage)]

## <a name="error-handling-and-logging"></a>错误处理和日志记录

使用 try-catch 语句处理错误。 检查 `Exception` 对象，确定发生错误后要采取的适当操作。

[!code-csharp[Logging](dotnet-client/sample/signalrchatclient/MainWindow.xaml.cs?name=snippet_ErrorHandling)]

## <a name="additional-resources"></a>其他资源

* [中心](xref:signalr/hubs)
* [JavaScript 客户端](xref:signalr/javascript-client)
* [发布到 Azure](xref:signalr/publish-to-azure-web-app)
* [Azure SignalR Service 无服务器文档](/azure/azure-signalr/signalr-concept-serverless-development-config)
