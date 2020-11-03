---
title: ASP.NET Core 中的密钥永久性和密钥设置
author: rick-anderson
description: 了解 ASP.NET Core 数据保护密钥永久性 Api 的实现细节。
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
uid: security/data-protection/implementation/key-immutability
ms.openlocfilehash: 8efca1d96945cb7af0132b27801b23a45714e08a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061244"
---
# <a name="key-immutability-and-key-settings-in-aspnet-core"></a>ASP.NET Core 中的密钥永久性和密钥设置

一旦将对象保存到后备存储，它的表示形式就会永远固定。 可以将新数据添加到后备存储，但不能转变现有数据。 此行为的主要目的是防止数据损坏。

此行为的一个结果是，将密钥写入后备存储后，它是不可变的。 它的创建、激活和到期日期永远不会更改，但可以通过使用进行吊销 `IKeyManager` 。 此外，其基础算法信息、主密钥材料和静态加密属性也是不可变的。

如果开发人员更改了任何影响密钥持久性的设置，则在下一次生成密钥之前，这些更改不会生效，无论是通过显式调用 `IKeyManager.CreateNewKey` 或通过数据保护系统自己的 [自动密钥生成](xref:security/data-protection/implementation/key-management#data-protection-implementation-key-management) 行为来实现的。 影响密钥持久性的设置如下所示：

* [默认密钥生存期](xref:security/data-protection/implementation/key-management#data-protection-implementation-key-management)

* [静态密钥加密机制](xref:security/data-protection/implementation/key-encryption-at-rest)

* [密钥内包含的算法信息](xref:security/data-protection/configuration/overview#changing-algorithms-with-usecryptographicalgorithms)

如果需要在下一次自动密钥滚动时间之前开始使用这些设置，请考虑进行显式调用 `IKeyManager.CreateNewKey` 来强制创建新密钥。 请记住提供一个明确的激活日期， ( {now + 2 days} 是一种很好的经验规则，以允许更改传播) 和到期日期。

>[!TIP]
> 接触存储库的所有应用程序都应指定相同的设置和 `IDataProtectionBuilder` 扩展方法。 否则，持久化密钥的属性将依赖于调用密钥生成例程的特定应用程序。
