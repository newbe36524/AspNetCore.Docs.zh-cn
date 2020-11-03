---
title: 在 ASP.NET Core 中限制受保护负载的生存期
author: rick-anderson
description: 了解如何使用 ASP.NET Core 数据保护 Api 限制受保护负载的生存期。
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
uid: security/data-protection/consumer-apis/limited-lifetime-payloads
ms.openlocfilehash: 74417d076399066a49271c27ff128d9de6c10f94
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060789"
---
# <a name="limit-the-lifetime-of-protected-payloads-in-aspnet-core"></a>在 ASP.NET Core 中限制受保护负载的生存期

在某些情况下，应用程序开发人员需要创建在设定的时间段之后过期的受保护负载。 例如，受保护的负载可能表示密码重置令牌，该令牌仅应在一小时内有效。 当然，开发人员也可以创建自己的负载格式，其中包含一个嵌入的过期日期，而高级开发人员可能希望这样做，但对于大多数管理这些过期的开发人员来说，这种情况都很繁琐。

为了使我们的开发人员受众更容易，包 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) 包含用于创建在设定的时间段后自动过期的有效负载的实用工具 api。 这些 Api 挂起 `ITimeLimitedDataProtector` 类型。

## <a name="api-usage"></a>API 使用情况

`ITimeLimitedDataProtector`接口是用于保护和取消保护时间限制/自过期负载的核心接口。 若要创建的实例 `ITimeLimitedDataProtector` ，首先需要使用特定用途构造的常规 [IDataProtector](xref:security/data-protection/consumer-apis/overview) 的实例。 `IDataProtector`实例可用后，调用 `IDataProtector.ToTimeLimitedDataProtector` 扩展方法以获取带有内置过期功能的保护程序。

`ITimeLimitedDataProtector` 公开以下 API surface 和 extension 方法：

* CreateProtector (string 目的) ： ITimeLimitedDataProtector-此 API 类似于现有的 `IDataProtectionProvider.CreateProtector` ，因为它可用于从根时间限制的保护程序创建 [用途链](xref:security/data-protection/consumer-apis/purpose-strings) 。

* 保护 (byte [] 纯文本、DateTimeOffset 过期) ： byte []

* 保护 (byte [] 纯文本、TimeSpan 生存期) ： byte []

* 保护 (byte [] 纯文本) ： byte []

* 保护 (字符串纯文本、DateTimeOffset 过期) ： string

* 保护 (字符串纯文本、TimeSpan 生存期) ： string

* 保护 (字符串纯文本) ： string

除了使用 `Protect` 纯文本的核心方法之外，还有一些新的重载，可用于指定有效负载的到期日期。 可以通过) 将到期日期指定为通过 `DateTimeOffset`) 或从当前系统时间 (的相对时间 (的绝对日期 `TimeSpan` 。 如果调用不带过期的重载，则假定负载永不过期。

* 取消保护 (byte [] protectedData，out DateTimeOffset 过期) ： byte []

* 取消保护 (byte [] protectedData) ： byte []

* 取消保护 (字符串 protectedData，out DateTimeOffset 过期) ： string

* 取消保护 (字符串 protectedData) ： string

`Unprotect`方法返回未受保护的原始数据。 如果负载尚未过期，则将以可选的 out 参数形式返回绝对过期，并将其作为不受保护的原始数据。 如果有效负载已过期，则解除保护方法的所有重载都将引发 System.security.cryptography.cryptographicexception。

>[!WARNING]
> 不建议使用这些 Api 来保护需要长期或无限持久性的负载。 "我可以承受受保护的负载在一个月后永久无法恢复吗？" 可以作为最佳经验法则;如果答案不是，开发人员应考虑其他 Api。

下面的示例使用 [非 DI 代码路径](xref:security/data-protection/configuration/non-di-scenarios) 来实例化数据保护系统。 若要运行此示例，请确保首先添加了对 AspNetCore 包的引用。

[!code-csharp[](limited-lifetime-payloads/samples/limitedlifetimepayloads.cs)]
