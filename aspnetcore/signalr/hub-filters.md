---
title: 在 ASP.NET Core 中使用集线器筛选器 SignalR
author: brecon
description: 了解如何在 ASP.NET Core 中使用集线器筛选器 SignalR 。
monikerRange: '>= aspnetcore-5.0'
ms.author: brecon
ms.custom: mvc
ms.date: 05/22/2020
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
uid: signalr/hub-filters
ms.openlocfilehash: 5a4cb5122080b72875ac11cf2e682162d017d7b9
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052716"
---
# <a name="use-hub-filters-in-aspnet-core-no-locsignalr"></a>在 ASP.NET Core 中使用集线器筛选器 SignalR

中心筛选器：

* 在 ASP.NET Core 5.0 或更高版本中可用。
* 允许在客户端调用中心方法之前和之后运行逻辑。

本文提供了有关编写和使用集线器筛选器的指南。

## <a name="configure-hub-filters"></a>配置中心筛选器

可以全局或按中心类型应用中心筛选器。 添加筛选器的顺序是筛选器的运行顺序。 全局集线器筛选器在本地中心筛选器之前运行。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSignalR(options =>
    {
        // Global filters will run first
        options.AddFilter<CustomFilter>();
    }).AddHubOptions<ChatHub>(options =>
    {
        // Local filters will run second
        options.AddFilter<CustomFilter2>();
    });
}
```

可以通过以下方式之一添加集线器筛选器：

* 按具体类型添加筛选器：

    ```csharp
    hubOptions.AddFilter<TFilter>();
    ```

    这将从依赖关系注入中解析 (DI) 或类型已激活。

* 按运行时类型添加筛选器：

    ```csharp
    hubOptions.AddFilter(typeof(TFilter));
    ```

    这将从 DI 或激活的类型进行解析。

* 按实例添加筛选器：

    ```csharp
    hubOptions.AddFilter(new MyFilter());
    ```

    此实例将像单一实例一样使用。 所有集线器方法调用都将使用相同的实例。

中心筛选器按中心调用创建和释放。 如果要将全局状态存储在筛选器中或不是状态，请将中心筛选器类型作为单一实例添加到 DI，以获得更好的性能。 或者，将筛选器添加为实例（如果可以）。

## <a name="create-hub-filters"></a>创建中心筛选器

通过声明从继承的类来创建筛选器 `IHubFilter` ，并添加 `InvokeMethodAsync` 方法。 此外，还 `OnConnectedAsync` `OnDisconnectedAsync` 可以选择实现来 `OnConnectedAsync` 分别包装和 `OnDisconnectedAsync` 中心方法。

```csharp
public class CustomFilter : IHubFilter
{
    public async ValueTask<object> InvokeMethodAsync(
        HubInvocationContext invocationContext, Func<HubInvocationContext, ValueTask<object>> next)
    {
        Console.WriteLine($"Calling hub method '{invocationContext.HubMethodName}'");
        try
        {
            return await next(invocationContext);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Exception calling '{invocationContext.HubMethodName}'");
            throw ex;
        }
    }

    // Optional method
    public Task OnConnectedAsync(HubLifetimeContext context, Func<HubLifetimeContext, Task> next)
    {
        return next(context);
    }

    // Optional method
    public Task OnDisconnectedAsync(
        HubLifetimeContext context, Exception exception, Func<HubLifetimeContext, Exception, Task> next)
    {
        return next(context, exception);
    }
}
```

筛选器与中间件非常类似。 `next`方法调用下一个筛选器。 最终筛选器将调用中心方法。 筛选器还可以存储在 `next` 调用中心方法之前调用的结果，然后再从返回结果 `next` 。

若要跳过筛选器中的集线器方法调用，请引发类型为的异常， `HubException` 而不是调用 `next` 。 如果要求，客户端将收到错误消息。

## <a name="use-hub-filters"></a>使用中心筛选器

编写筛选器逻辑时，请尝试使用集线器方法上的属性而不是检查中心方法名称，使其成为泛型。

假设有一个筛选器，它将检查禁止的短语的集线器方法参数，并替换它所找到的任何短语 `***` 。
对于本示例，假定定义了一个 `LanguageFilterAttribute` 类。 类具有一个名为 `FilterArgument` 的属性，该属性可在使用特性时进行设置。

1. 将属性放置在具有要清理的字符串参数的 hub 方法上：

    ```csharp
    public class ChatHub
    {
        [LanguageFilter(filterArgument: 0)]
        public async Task SendMessage(string message, string username)
        {
            await Clients.All.SendAsync("SendMessage", $"{username} says: {message}");
        }
    }
    ```

1. 定义一个中心筛选器来检查属性，并将集线器方法参数中的禁止的短语替换为 `**_` ：

    ```csharp
    public class LanguageFilter : IHubFilter
    {
        // populated from a file or inline
        private List<string> bannedPhrases = new List<string> { "async void", ".Result" };

        public async ValueTask<object> InvokeMethodAsync(HubInvocationContext invocationContext, 
            Func<HubInvocationContext, ValueTask<object>> next)
        {
            var languageFilter = (LanguageFilterAttribute)Attribute.GetCustomAttribute(
                invocationContext.HubMethod, typeof(LanguageFilterAttribute));
            if (languageFilter != null &&
                invocationContext.HubMethodArguments.Count > languageFilter.FilterArgument &&
                invocationContext.HubMethodArguments[languageFilter.FilterArgument] is string str)
            {
                foreach (var bannedPhrase in bannedPhrases)
                {
                    str = str.Replace(bannedPhrase, "_**");
                }

                arguments = invocationContext.HubMethodArguments.ToArray();
                arguments[languageFilter.FilterArgument] = str;
                invocationContext = new HubInvocationContext(invocationContext.Context,
                    invocationContext.ServiceProvider,
                    invocationContext.Hub,
                    invocationContext.HubMethod,
                    arguments);
            }

            return await next(invocationContext);
        }
    }
    ```

1. 在方法中注册集线器筛选器 `Startup.ConfigureServices` 。 为了避免为每个调用重新初始化禁止使用的词组列表，将中心筛选器注册为单一实例：

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddSignalR(hubOptions =>
        {
            hubOptions.AddFilter<LanguageFilter>();
        });
    
        services.AddSingleton<LanguageFilter>();
    }
    ```

## <a name="the-hubinvocationcontext-object"></a>HubInvocationContext 对象

`HubInvocationContext`包含当前集线器方法调用的信息。

| 属性 | 说明 | 类型 |
| ------ | ------ | ----------- |
| `Context ` | `HubCallerContext`包含有关连接的信息。 | `HubCallerContext` |
| `Hub` | 正在用于此集线器方法调用的集线器的实例。 | `Hub` |
| `HubMethodName` | 正在调用的集线器方法的名称。 | `string` |
| `HubMethodArguments` | 要传递到中心方法的参数的列表。 | `IReadOnlyList<string>` |
| `ServiceProvider` | 此中心方法调用的作用域内服务提供程序。 | `IServiceProvider` |
| `HubMethod` | 集线器方法信息。 | `MethodInfo` |

## <a name="the-hublifetimecontext-object"></a>HubLifetimeContext 对象

`HubLifetimeContext`包含 `OnConnectedAsync` 和集线器方法的信息 `OnDisconnectedAsync` 。

| 属性 | 说明 | 类型 |
| ------ | ------ | ----------- |
| `Context ` | `HubCallerContext`包含有关连接的信息。 | `HubCallerContext` |
| `Hub` | 正在用于此集线器方法调用的集线器的实例。 | `Hub` |
| `ServiceProvider` | 此中心方法调用的作用域内服务提供程序。 | `IServiceProvider` |

## <a name="authorization-and-filters"></a>授权和筛选器

在中心筛选器之前，[对集线器方法上的属性授权](xref:signalr/authn-and-authz#use-authorization-handlers-to-customize-hub-method-authorization)。
