---
title: ASP.NET Core 中的哈希密码
author: rick-anderson
description: 了解如何使用 ASP.NET Core 数据保护 Api 为密码进行哈希处理。
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
uid: security/data-protection/consumer-apis/password-hashing
ms.openlocfilehash: a970d44a1ca6b9f3534bddb34b037e7c2fdc5389
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051858"
---
# <a name="hash-passwords-in-aspnet-core"></a>ASP.NET Core 中的哈希密码

数据保护基本代码包括一个包含加密密钥派生函数的 *AspNetCore 密钥派生* 。 此包是一个独立的组件，与数据保护系统的其余部分没有任何依赖关系。 它可以完全独立地使用。 为了方便起见，源与数据保护代码库并存。

包当前提供了一个方法 `KeyDerivation.Pbkdf2` ，该方法允许使用 [PBKDF2 算法](https://tools.ietf.org/html/rfc2898#section-5.2)对密码进行哈希处理。 此 API 与 .NET Framework 现有的 [Rfc2898DeriveBytes 类型](/dotnet/api/system.security.cryptography.rfc2898derivebytes)非常相似，但有三个重要的区别：

1. `KeyDerivation.Pbkdf2`方法支持在当前、和)  (使用多个 PRFs， `HMACSHA1` `HMACSHA256` `HMACSHA512` 而 `Rfc2898DeriveBytes` 类型仅支持 `HMACSHA1` 。

2. `KeyDerivation.Pbkdf2`方法将检测当前操作系统，并尝试选择最适合的例程实现，在某些情况下提供更好的性能。  (在 Windows 8 上，它提供的吞吐量大约为10倍 `Rfc2898DeriveBytes` 。 ) 

3. `KeyDerivation.Pbkdf2`方法要求调用方指定 (盐、PRF 和迭代计数) 的所有参数。 `Rfc2898DeriveBytes`类型为这些提供默认值。

[!code-csharp[](password-hashing/samples/passwordhasher.cs)]

查看[source code](https://github.com/dotnet/AspNetCore/blob/master/src/Identity/Extensions.Core/src/PasswordHasher.cs) ASP.NET Core Identity `PasswordHasher` 真实用例的类型的源代码。
