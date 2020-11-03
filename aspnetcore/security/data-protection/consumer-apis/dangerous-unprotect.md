---
title: 取消保护已在 ASP.NET Core 中吊销其密钥的负载
author: rick-anderson
description: 了解如何在 ASP.NET Core 应用中取消保护数据，并使用自吊销的密钥进行保护。
ms.author: riande
ms.custom: mvc
ms.date: 10/24/2018
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
uid: security/data-protection/consumer-apis/dangerous-unprotect
ms.openlocfilehash: 5f13b53182f720ac8b58d90701561d381981308a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051091"
---
# <a name="unprotect-payloads-whose-keys-have-been-revoked-in-aspnet-core"></a>取消保护已在 ASP.NET Core 中吊销其密钥的负载

<a name="data-protection-consumer-apis-dangerous-unprotect"></a>

ASP.NET Core 的数据保护 Api 主要用于机密负载的无限持久性。 其他技术（如 [WINDOWS CNG DPAPI](/windows/win32/seccng/cng-dpapi) 和 [Azure Rights Management](/rights-management/) ）更适用于无限存储的情况，并且具有相应的强大密钥管理功能。 也就是说，不会阻止开发人员使用 ASP.NET Core 的数据保护 Api 来长期保护机密数据。 密钥永远不会从密钥环中删除，因此， `IDataProtector.Unprotect` 只要密钥可用且有效，就可以始终恢复现有有效负载。

但是，当开发人员尝试取消保护已被吊销密钥保护的数据时，会出现问题， `IDataProtector.Unprotect` 这种情况下会引发异常。 这可能适用于短期或暂时性负载 (例如身份验证令牌) ，因为系统可以轻松地重新创建这种类型的负载，并且在最糟糕的情况下，站点访问者可能需要再次登录。 但对于持久化有效负载， `Unprotect` 引发 throw 可能导致数据丢失不可接受。

## <a name="ipersisteddataprotector"></a>IPersistedDataProtector

为支持允许负载不受保护的方案（即使在面对吊销密钥的情况下），数据保护系统包含 `IPersistedDataProtector` 类型。 若要获取实例 `IPersistedDataProtector` ，只需 `IDataProtector` 以正常方式获取实例，然后尝试将转换 `IDataProtector` 为 `IPersistedDataProtector` 。

> [!NOTE]
> 并非所有 `IDataProtector` 实例都可以转换为 `IPersistedDataProtector` 。 开发人员应将 c # 用作运算符或类似，以避免由无效强制转换导致的运行时异常，并应准备适当地处理故障情况。

`IPersistedDataProtector` 公开以下 API 图面：

```csharp
DangerousUnprotect(byte[] protectedData, bool ignoreRevocationErrors,
     out bool requiresMigration, out bool wasRevoked) : byte[]
```

此 API 使用受保护的负载 (作为字节数组) 并返回未受保护的负载。 没有基于字符串的重载。 这两个 out 参数如下所示。

* `requiresMigration`：如果用于保护此有效负载的密钥不再是活动的默认密钥，则将设置为 true，例如，用于保护此有效负载的密钥是旧的，并且已发生密钥滚动操作。 调用方可能需要考虑根据其业务需求重新保护负载。

* `wasRevoked`：如果用于保护此负载的密钥已被吊销，则设置为 true。

>[!WARNING]
> 传递给方法时， `ignoreRevocationErrors: true` 要格外小心 `DangerousUnprotect` 。 如果在调用此方法后 `wasRevoked` ，值为 true，则将吊销用于保护此负载的密钥，并且应将有效负载的真实性视为可疑。 在这种情况下，如果有一些单独的保证是可信的，例如它来自安全的数据库，而不是由不受信任的 web 客户端发送，则只能继续对未受保护的有效负载进行操作。

[!code-csharp[](dangerous-unprotect/samples/dangerous-unprotect.cs)]
