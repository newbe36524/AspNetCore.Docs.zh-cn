---
title: ASP.NET Core 中的用途层次结构和多租户
author: rick-anderson
description: 了解用途字符串层次结构和多租户，因为它与 ASP.NET Core 的数据保护 Api 相关。
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
uid: security/data-protection/consumer-apis/purpose-strings-multitenancy
ms.openlocfilehash: 84714cd852a3cbc7ff77d0fd0c88931536dead1a
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052781"
---
# <a name="purpose-hierarchy-and-multi-tenancy-in-aspnet-core"></a>ASP.NET Core 中的用途层次结构和多租户

由于 `IDataProtector` 也是隐式的 `IDataProtectionProvider` ，因此，目的可以链接在一起。 在这种意义上， `provider.CreateProtector([ "purpose1", "purpose2" ])` 等效于 `provider.CreateProtector("purpose1").CreateProtector("purpose2")` 。

这允许通过数据保护系统实现一些有趣的层次结构关系。 在前面的 [SecureMessage](xref:security/data-protection/consumer-apis/purpose-strings#data-protection-contoso-purpose)中，SecureMessage 组件可以提前调用， `provider.CreateProtector("Contoso.Messaging.SecureMessage")` 并将结果缓存到私有 `_myProvider` 字段。 以后可以通过对的调用创建未来的保护 `_myProvider.CreateProtector("User: username")` 程序，这些保护程序将用于保护单个消息。

这也可以是反向的。 假设有一个逻辑应用程序托管多个租户 (CMS 看似合理) ，每个租户都可以使用其自己的身份验证和状态管理系统进行配置。 该应用程序具有单个主提供程序，它调用 `provider.CreateProtector("Tenant 1")` 并为 `provider.CreateProtector("Tenant 2")` 每个租户提供自己的数据保护系统隔离切片。 然后，租户可以根据自己的需求来派生自己的单独保护程序，但不管它们尝试的情况如何，都不能创建与系统中任何其他租户发生冲突的保护程序。 以图形方式表示，如下所示。

![多租户用途](purpose-strings-multitenancy/_static/purposes-multi-tenancy.png)

>[!WARNING]
> 这会假定应用程序控制各个租户可用的 Api，并且租户无法在服务器上执行任意代码。 如果租户可以执行任意代码，则他们可能会执行私有反射来打破隔离保证，也可以直接读取主密钥材料，并派生出所需的任何子项。

数据保护系统实际上在其默认的现成配置中使用一种多租户。 默认情况下，主密钥材料存储在工作进程帐户的用户配置文件文件夹 (或注册表中，用于 IIS 应用程序池标识) 。 但实际使用单个帐户运行多个应用程序的情况并不是很常见，因此，所有这些应用程序都将结束共享主密钥材料。 若要解决此情况，数据保护系统会自动插入一个唯一的每个应用程序标识符作为总体用途链中的第一个元素。 这种隐式目的是通过有效地将每个应用程序视为系统内的唯一租户来将 [各个应用程序彼此隔离](xref:security/data-protection/configuration/overview#per-application-isolation) ，并且保护程序创建过程看起来与上图中的相同。
