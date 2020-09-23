---
title: 用于 ASP.NET Core 横向扩展的 Redis 底板 SignalR
author: bradygaster
description: 了解如何设置 Redis 底板以便为 ASP.NET Core 应用启用横向扩展 SignalR 。
monikerRange: '>= aspnetcore-2.1'
ms.author: bradyg
ms.custom: mvc
ms.date: 11/12/2019
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
uid: signalr/redis-backplane
ms.openlocfilehash: a57176409c5f9bcc620db7e070f6616951eb9a54
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722769"
---
# <a name="set-up-a-redis-backplane-for-aspnet-core-no-locsignalr-scale-out"></a>设置用于 ASP.NET Core 横向扩展的 Redis 底板 SignalR

作者： [Andrew Stanton](https://twitter.com/anurse)、 [Brady Gaster](https://twitter.com/bradygaster)和[Tom Dykstra](https://github.com/tdykstra)

本文介绍了 SignalR 设置 [Redis](https://redis.io/) 服务器以用于横向扩展 ASP.NET Core 应用程序的特定方面 SignalR 。

## <a name="set-up-a-redis-backplane"></a>设置 Redis 底板

* 部署 Redis 服务器。

  > [!IMPORTANT] 
  > 对于生产用途，建议仅当 Redis 底板与应用在同一数据中心内运行时才使用它 SignalR 。 否则，网络延迟会降低性能。 如果你 SignalR 的应用在 azure 云中运行，我们建议使用 azure SignalR 服务，而不是 Redis 底板。 可以使用 Azure Redis 缓存服务进行开发和测试环境。

  有关详细信息，请参阅以下资源：

  * <xref:signalr/scale>
  * [Redis 文档](https://redis.io/)
  * [Azure Redis 缓存文档](/azure/redis-cache/)

::: moniker range="= aspnetcore-2.1"

* 在 SignalR 应用中，安装 `Microsoft.AspNetCore.SignalR.Redis` NuGet 包。
* 在 `Startup.ConfigureServices` 方法中，调用 `AddRedis` after `AddSignalR` ：

  ```csharp
  services.AddSignalR().AddRedis("<your_Redis_connection_string>");
  ```

* 根据需要配置选项：
 
  可以在连接字符串中或在 [ConfigurationOptions](https://stackexchange.github.io/StackExchange.Redis/Configuration#configuration-options) 对象中设置大多数选项。 中指定的选项 `ConfigurationOptions` 替代在连接字符串中设置的选项。

  下面的示例演示如何在对象中设置选项 `ConfigurationOptions` 。 此示例将添加一个通道前缀，以便多个应用可以共享同一 Redis 实例，如以下步骤中所述。

  ```csharp
  services.AddSignalR()
    .AddRedis(connectionString, options => {
        options.Configuration.ChannelPrefix = "MyApp";
    });
  ```

  在前面的代码中， `options.Configuration` 用连接字符串中指定的内容进行初始化。

::: moniker-end

::: moniker range="= aspnetcore-2.2"

* 在 SignalR 应用中，安装以下 NuGet 包之一：

  * `Microsoft.AspNetCore.SignalR.StackExchangeRedis` -依赖于 Stackexchange.redis. Redis 2. 采用2.X.X。 建议将此包用于 ASP.NET Core 2.2 及更高版本。
  * `Microsoft.AspNetCore.SignalR.Redis` -依赖于 Stackexchange.redis. Redis 采用2.X.X。 此包不包含在 ASP.NET Core 3.0 及更高版本中。

* 在 `Startup.ConfigureServices` 方法中，调用 <xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisDependencyInjectionExtensions.AddStackExchangeRedis*> ：

  ```csharp
  services.AddSignalR().AddStackExchangeRedis("<your_Redis_connection_string>");
  ```

 使用时 `Microsoft.AspNetCore.SignalR.Redis` ，请调用 <xref:Microsoft.Extensions.DependencyInjection.RedisDependencyInjectionExtensions.AddRedis*> 。

* 根据需要配置选项：
 
  可以在连接字符串中或在 [ConfigurationOptions](https://stackexchange.github.io/StackExchange.Redis/Configuration#configuration-options) 对象中设置大多数选项。 中指定的选项 `ConfigurationOptions` 替代在连接字符串中设置的选项。

  下面的示例演示如何在对象中设置选项 `ConfigurationOptions` 。 此示例将添加一个通道前缀，以便多个应用可以共享同一 Redis 实例，如以下步骤中所述。

  ```csharp
  services.AddSignalR()
    .AddStackExchangeRedis(connectionString, options => {
        options.Configuration.ChannelPrefix = "MyApp";
    });
  ```

 使用时 `Microsoft.AspNetCore.SignalR.Redis` ，请调用 <xref:Microsoft.Extensions.DependencyInjection.RedisDependencyInjectionExtensions.AddRedis*> 。

  在前面的代码中， `options.Configuration` 用连接字符串中指定的内容进行初始化。

  有关 Redis 选项的信息，请参阅 [Stackexchange.redis Redis 文档](https://stackexchange.github.io/StackExchange.Redis/Configuration.html)。

::: moniker-end

::: moniker range=">= aspnetcore-3.0"

* 在 SignalR 应用中，安装以下 NuGet 包：

  * `Microsoft.AspNetCore.SignalR.StackExchangeRedis`
  
* 在 `Startup.ConfigureServices` 方法中，调用 <xref:Microsoft.Extensions.DependencyInjection.StackExchangeRedisDependencyInjectionExtensions.AddStackExchangeRedis*> ：

  ```csharp
  services.AddSignalR().AddStackExchangeRedis("<your_Redis_connection_string>");
  ```
  
* 根据需要配置选项：
 
  可以在连接字符串中或在 [ConfigurationOptions](https://stackexchange.github.io/StackExchange.Redis/Configuration#configuration-options) 对象中设置大多数选项。 中指定的选项 `ConfigurationOptions` 替代在连接字符串中设置的选项。

  下面的示例演示如何在对象中设置选项 `ConfigurationOptions` 。 此示例将添加一个通道前缀，以便多个应用可以共享同一 Redis 实例，如以下步骤中所述。

  ```csharp
  services.AddSignalR()
    .AddStackExchangeRedis(connectionString, options => {
        options.Configuration.ChannelPrefix = "MyApp";
    });
  ```

  在前面的代码中， `options.Configuration` 用连接字符串中指定的内容进行初始化。

  有关 Redis 选项的信息，请参阅 [Stackexchange.redis Redis 文档](https://stackexchange.github.io/StackExchange.Redis/Configuration.html)。

::: moniker-end

* 如果将一个 Redis 服务器用于多个 SignalR 应用程序，请为每个应用程序使用不同的通道前缀 SignalR 。

  设置通道前缀会将一个 SignalR 应用与其他使用不同通道前缀的应用隔离开来。 如果未分配不同的前缀，则从一个应用发送到其所有客户端的消息将发送到使用 Redis 服务器作为底板的所有应用的所有客户端。

* 为粘滞会话配置服务器场负载平衡软件。 下面是有关如何执行此操作的一些文档示例：

  * [IIS](/iis/extensions/configuring-application-request-routing-arr/http-load-balancing-using-application-request-routing)
  * [HAProxy](https://www.haproxy.com/blog/load-balancing-affinity-persistence-sticky-sessions-what-you-need-to-know/)
  * [Nginx](https://docs.nginx.com/nginx/admin-guide/load-balancer/http-load-balancer/#sticky)
  * [pfSense](https://www.netgate.com/docs/pfsense/loadbalancing/inbound-load-balancing.html#sticky-connections)

## <a name="redis-server-errors"></a>Redis 服务器错误

当 Redis 服务器发生故障时，将 SignalR 引发指示消息无法传递的异常。 一些典型的异常消息：

* *写入消息失败*
* *未能调用中心方法 "方法名称"*
* *未能连接到 Redis*

SignalR 不会在服务器重新启动时缓冲消息以发送这些消息。 Redis 服务器关闭时发送的任何消息都将丢失。

SignalR 当 Redis 服务器再次可用时，自动重新连接。

### <a name="custom-behavior-for-connection-failures"></a>连接失败的自定义行为

下面是演示如何处理 Redis 连接失败事件的示例。

::: moniker range="= aspnetcore-2.1"

```csharp
services.AddSignalR()
        .AddRedis(o =>
        {
            o.ConnectionFactory = async writer =>
            {
                var config = new ConfigurationOptions
                {
                    AbortOnConnectFail = false
                };
                config.EndPoints.Add(IPAddress.Loopback, 0);
                config.SetDefaultPorts();
                var connection = await ConnectionMultiplexer.ConnectAsync(config, writer);
                connection.ConnectionFailed += (_, e) =>
                {
                    Console.WriteLine("Connection to Redis failed.");
                };

                if (!connection.IsConnected)
                {
                    Console.WriteLine("Did not connect to Redis.");
                }

                return connection;
            };
        });
```

::: moniker-end

::: moniker range="> aspnetcore-2.1"

```csharp
services.AddSignalR()
        .AddMessagePackProtocol()
        .AddStackExchangeRedis(o =>
        {
            o.ConnectionFactory = async writer =>
            {
                var config = new ConfigurationOptions
                {
                    AbortOnConnectFail = false
                };
                config.EndPoints.Add(IPAddress.Loopback, 0);
                config.SetDefaultPorts();
                var connection = await ConnectionMultiplexer.ConnectAsync(config, writer);
                connection.ConnectionFailed += (_, e) =>
                {
                    Console.WriteLine("Connection to Redis failed.");
                };

                if (!connection.IsConnected)
                {
                    Console.WriteLine("Did not connect to Redis.");
                }

                return connection;
            };
        });
```

::: moniker-end

## <a name="redis-clustering"></a>Redis 群集

[Redis 聚类分析](https://redis.io/topics/cluster-spec) 是一种通过使用多个 Redis 服务器实现高可用性的方法。 群集不是正式支持的，但它可能会起作用。

## <a name="next-steps"></a>后续步骤

有关详细信息，请参阅以下资源：

* <xref:signalr/scale>
* [Redis 文档](https://redis.io/documentation)
* [Stackexchange.redis Redis 文档](https://stackexchange.github.io/StackExchange.Redis/)
* [Azure Redis 缓存文档](/azure/redis-cache/)