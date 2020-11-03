---
title: ASP.NET Core 中的密钥管理
author: rick-anderson
description: 了解 ASP.NET Core 的数据保护密钥管理 Api 的实现细节。
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
uid: security/data-protection/implementation/key-management
ms.openlocfilehash: f7e0fa5e80208469188a11239a9453ecafdcf5f3
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060191"
---
# <a name="key-management-in-aspnet-core"></a>ASP.NET Core 中的密钥管理

<a name="data-protection-implementation-key-management"></a>

数据保护系统自动管理用于保护和取消保护负载的主密钥的生存期。 每个密钥可以有以下四个阶段之一：

* 已创建-密钥在密钥环中存在，但尚未激活。 在经过足够的时间之后，密钥有机会传播到使用此密钥环的所有计算机之前，不应将该密钥用于新的保护操作。

* 活动-密钥在密钥环中存在，并且应该用于所有新的保护操作。

* 已过期-密钥已运行其自然生存期，不应再用于新的保护操作。

* 已吊销-密钥已泄露，不能用于新的保护操作。

已创建、活动和过期的密钥均可用于取消保护传入有效负载。 默认情况下，吊销的密钥不能用于取消保护有效负载，但应用程序开发人员可以在必要时 [重写此行为](xref:security/data-protection/consumer-apis/dangerous-unprotect#data-protection-consumer-apis-dangerous-unprotect) 。

>[!WARNING]
> 开发人员可能会尝试从密钥环中删除密钥 (例如，通过从文件系统) 中删除相应的文件。 此时，由该密钥保护的所有数据都将永久 undecipherable，而且不会有任何紧急替代，就像吊销密钥一样。 删除密钥是真正的破坏性行为，因此数据保护系统不会公开用于执行此操作的第一类 API。

## <a name="default-key-selection"></a>默认键选择

当数据保护系统从后备存储库读取密钥环时，它将尝试从密钥环中查找 "默认" 密钥。 默认密钥用于新的保护操作。

一般试探法是数据保护系统选择最新激活日期为默认密钥的密钥。  (有一个较小的奶油因素，以允许服务器到服务器的时钟歪斜。 ) 如果密钥已过期或已被吊销，并且如果应用程序未禁用自动密钥生成，则会生成一个新密钥，并按以下 [密钥过期和滚动](xref:security/data-protection/implementation/key-management#data-protection-implementation-key-management-expiration) 策略立即激活。

数据保护系统立即生成新密钥，而不是回退到不同的密钥，这是因为新密钥生成应被视为在新密钥之前激活的所有密钥的隐式过期。 一般情况下，可能已使用不同于旧密钥的算法或静态加密机制配置了新密钥，并且系统应更倾向于回退当前配置。

出现异常。 如果应用程序开发人员 [禁用了自动密钥生成](xref:security/data-protection/configuration/overview#disableautomatickeygeneration)，则数据保护系统必须选择某些项作为默认密钥。 在此回退方案中，系统会选择具有最新激活日期的非吊销密钥，并为有时间传播到群集中的其他计算机的密钥提供首选项。 回退系统可能最终会选择过期的默认密钥。 回退系统永远不会选择吊销的密钥作为默认密钥，如果密钥环为空或每个密钥已被吊销，则系统会在初始化时产生错误。

<a name="data-protection-implementation-key-management-expiration"></a>

## <a name="key-expiration-and-rolling"></a>密钥过期和滚动

创建密钥后，会自动向其提供一个 {now + 2 days} 的激活日期和过期日期 {now + 90 天}。 激活前的2天延迟提供通过系统传播的关键时间。 也就是说，它允许指向后备存储的其他应用程序在其下一次自动刷新期间观察密钥，从而最大限度地提高密钥环变为活动状态的所有应用程序的可能性。

如果默认密钥将在2天内过期，且密钥环还没有在默认密钥过期后处于活动状态的密钥，则数据保护系统会自动将新密钥保存到密钥环。 此新密钥的激活日期为 {默认密钥的过期日期}，到期日期为 {now + 90 天}。 这使系统能够定期自动滚动更新密钥，而不会中断服务。

在某些情况下，将使用立即激活创建密钥。 例如，当应用程序未运行一段时间并且密钥环中的所有密钥都已过期时。 发生这种情况时，会为该密钥提供一个 {now} 的激活日期，而不会出现常规的2天激活延迟。

默认密钥生存期为90天，但可以配置这种生存期，如以下示例中所示。

```csharp
services.AddDataProtection()
       // use 14-day lifetime instead of 90-day lifetime
       .SetDefaultKeyLifetime(TimeSpan.FromDays(14));
```

管理员还可以更改系统范围内的默认值，尽管对的显式调用 `SetDefaultKeyLifetime` 将覆盖任何系统范围内的策略。 默认密钥生存期不能短于7天。

## <a name="automatic-key-ring-refresh"></a>自动密钥环刷新

当数据保护系统初始化时，它将从底层存储库读取密钥环，并将其缓存在内存中。 此缓存允许在不命中后备存储的情况下继续保护和取消保护操作。 系统将每隔24小时自动检查一次后备存储的更改或当前的默认密钥过期时，以先达到的时间为准。

>[!WARNING]
> 如果) 需要直接使用密钥管理 Api，开发人员非常少 (。 数据保护系统将执行上述自动密钥管理。

数据保护系统公开了一个接口 `IKeyManager` ，该接口可用于检查和更改密钥环。 提供实例的 DI 系统 `IDataProtectionProvider` 还可以 `IKeyManager` 为你的消耗提供的实例。 或者，你可以 `IKeyManager` 直接从中拉取， `IServiceProvider` 如以下示例中所示。

修改密钥环 (显式创建新密钥或执行吊销) 的任何操作都会使内存中缓存失效。 对或的下一个调用 `Protect` `Unprotect` 将导致数据保护系统重新读取键环并重新创建缓存。

下面的示例演示如何使用 `IKeyManager` 接口来检查和操作密钥环，包括吊销现有密钥并手动生成新密钥。

[!code-csharp[](key-management/samples/key-management.cs)]

[!INCLUDE[about the series](~/includes/code-comments-loc.md)]

## <a name="key-storage"></a>密钥存储

数据保护系统具有试探法，它尝试自动推导适当的密钥存储位置和静态加密机制。 应用程序开发人员也可以配置密钥持久性机制。 以下文档介绍了这些机制的内置实现：

* <xref:security/data-protection/implementation/key-storage-providers>
* <xref:security/data-protection/implementation/key-encryption-at-rest>
