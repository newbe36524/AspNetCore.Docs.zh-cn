---
title: ASP.NET Core 中的用途字符串
author: rick-anderson
description: 了解如何在 ASP.NET Core 数据保护 Api 中使用目的字符串。
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
uid: security/data-protection/consumer-apis/purpose-strings
ms.openlocfilehash: 9a55131714b23909da5d00b73607078b295a1c4d
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060802"
---
# <a name="purpose-strings-in-aspnet-core"></a>ASP.NET Core 中的用途字符串

<a name="data-protection-consumer-apis-purposes"></a>

使用的组件 `IDataProtectionProvider` 必须将唯一的 *用途* 参数传递给 `CreateProtector` 方法。 目的 *参数* 对于数据保护系统的安全性是固有的，因为它在加密使用者之间提供隔离，即使根加密密钥相同也是如此。

当使用者指定目的时，会将目的字符串与根加密密钥一起使用，以派生该使用者独有的加密子项。 这会将使用者与应用程序中的所有其他加密使用者隔离：其他组件不能读取其有效负载，也不能读取任何其他组件的负载。 此隔离还会针对组件呈现不可行的整个攻击类别。

![用途关系图示例](purpose-strings/_static/purposes.png)

在上图中， `IDataProtector` 实例 A 和 B **无法** 读取彼此各自的负载。

用途字符串不必是机密。 它应该是唯一的，因为任何其他正常行为的组件都不能提供相同的目的字符串。

>[!TIP]
> 使用数据保护 Api 的组件命名空间和类型名称是一个很好的经验法则，因为在实践中，此信息永远不会发生冲突。
>
>负责 minting 持有者令牌的 Contoso 创作组件可能会使用 BearerToken 作为其目的字符串。 甚至更好的是，它可能使用 BearerToken 作为其目的字符串。 追加版本号后，将来的版本将使用 BearerToken 作为其用途，而不同的版本将在负载开始时彼此之间完全隔离。

由于的用途参数 `CreateProtector` 是一个字符串数组，因此可以改为将上面的指定为 `[ "Contoso.Security.BearerToken", "v1" ]` 。 这允许建立一个目的层次结构，并使用数据保护系统开启多租户方案。

<a name="data-protection-contoso-purpose"></a>

>[!WARNING]
> 组件不应允许不受信任的用户输入成为目的链的唯一输入源。
>
>例如，假设有一个 SecureMessage，该组件负责存储安全消息。 如果要调用安全消息传送组件 `CreateProtector([ username ])` ，则恶意用户可能会在尝试获取要调用的组件时创建用户名为 "BearerToken" 的帐户 `CreateProtector([ "Contoso.Security.BearerToken" ])` ，从而无意中导致安全消息传送系统 mint 可视为身份验证令牌的负载。
>
>消息传送组件的更好的目的链是 `CreateProtector([ "Contoso.Messaging.SecureMessage", "User: username" ])` ，它提供了适当的隔离。

由提供的隔离和 `IDataProtectionProvider` 、和目的的行为如下所示 `IDataProtector` ：

* 对于给定的 `IDataProtectionProvider` 对象， `CreateProtector` 方法会创建一个 `IDataProtector` 唯一绑定到 `IDataProtectionProvider` 创建它的对象和传递到该方法的用途参数的对象。

* 用途参数不得为 null。  (如果将目的指定为数组，这意味着数组的长度不能为零，且数组的所有元素都必须为非 null。 ) 在技术上允许空字符串目的，但不鼓励这样做。

* 当且仅当它们包含相同的字符串 (按相同顺序) 使用序号比较器时，两个用途参数相等。 单个用途参数等效于相应的单元素用途数组。

* `IDataProtector`如果和的对象是从 `IDataProtectionProvider` 具有等效目的参数的等效对象创建的，则这两个对象是等效的。

* 对于给定的 `IDataProtector` 对象， `Unprotect(protectedData)` `unprotectedData` 当且仅当 `protectedData := Protect(unprotectedData)` 对于等效对象时，对的调用将返回原始 `IDataProtector` 。

> [!NOTE]
> 我们不考虑这样的情况：某些组件有意选择的目的字符串与另一个组件冲突。 此类组件实质上被视为恶意组件，并且此系统不会在恶意代码已在工作进程内部运行的情况下提供安全保证。
