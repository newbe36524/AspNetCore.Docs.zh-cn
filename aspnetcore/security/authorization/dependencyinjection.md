---
title: ASP.NET Core 中的要求处理程序中的依赖项注入
author: rick-anderson
description: 了解如何使用依赖关系注入将授权要求处理程序注入到 ASP.NET Core 应用。
ms.author: riande
ms.date: 10/14/2016
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
uid: security/authorization/dependencyinjection
ms.openlocfilehash: 6598a9c9cfd1e6597fffcc1aa0c53fa493532458
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060256"
---
# <a name="dependency-injection-in-requirement-handlers-in-aspnet-core"></a>ASP.NET Core 中的要求处理程序中的依赖项注入

<a name="security-authorization-di"></a>

在使用[依赖关系注入](xref:fundamentals/dependency-injection)的配置过程中，必须在服务集合中[注册授权处理程序](xref:security/authorization/policies#handler-registration)。

假设你有一个规则存储库，你希望在授权处理程序中评估该存储库，并在服务集合中注册该存储库。 授权将解析并注入到构造函数中。

例如，使用 ASP。网络日志记录基础结构，注入 `ILoggerFactory` 处理程序。 此类处理程序可能类似于以下代码：

```csharp
public class LoggingAuthorizationHandler : AuthorizationHandler<MyRequirement>
   {
       ILogger _logger;

       public LoggingAuthorizationHandler(ILoggerFactory loggerFactory)
       {
           _logger = loggerFactory.CreateLogger(this.GetType().FullName);
       }

       protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, MyRequirement requirement)
       {
           _logger.LogInformation("Inside my handler");
           // Check if the requirement is fulfilled.
           return Task.CompletedTask;
       }
   }
   ```

前面的处理程序可注册到任何 [服务生存期](/dotnet/core/extensions/dependency-injection#service-lifetimes)。 下面的代码使用 `AddSingleton` 注册前面的处理程序：

```csharp
services.AddSingleton<IAuthorizationHandler, LoggingAuthorizationHandler>();
```

处理程序的实例是在应用程序启动时创建的，DI 会将注册的注入 `ILoggerFactory` 构造函数中。

> [!NOTE]
> 使用实体框架的处理程序不应注册为单一实例。
