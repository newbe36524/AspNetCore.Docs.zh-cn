---
title: ASP.NET Core 中的临时数据保护提供程序
author: rick-anderson
description: 了解 ASP.NET Core 临时数据保护提供程序的实现细节。
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
uid: security/data-protection/implementation/key-storage-ephemeral
ms.openlocfilehash: ed0fca88ecf2b002a4421fb120d90adff1b5b12e
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93052693"
---
# <a name="ephemeral-data-protection-providers-in-aspnet-core"></a>ASP.NET Core 中的临时数据保护提供程序

<a name="data-protection-implementation-key-storage-ephemeral"></a>

在某些情况下，应用程序需要一次性 `IDataProtectionProvider` 。 例如，开发人员可能只是在一次性的控制台应用程序中试验，或者应用程序本身 (脚本编写或) 单元测试项目时是暂时性的。 为支持这些方案， [AspNetCore. DataProtection](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection/) 包包含类型 `EphemeralDataProtectionProvider` 。 此类型提供 `IDataProtectionProvider` 其密钥存储库仅保存在内存中且不会写出到任何后备存储的基本实现。

每个实例都 `EphemeralDataProtectionProvider` 使用其自己的唯一主密钥。 因此，如果中的根为的 `IDataProtector` `EphemeralDataProtectionProvider` 生成受保护的负载，则该负载仅在 `IDataProtector` 给定相同 [目的](xref:security/data-protection/consumer-apis/purpose-strings#data-protection-consumer-apis-purposes) 链) 在同一实例上的情况下，不能通过等效 (来保护 `EphemeralDataProtectionProvider` 。

下面的示例演示如何实例化 `EphemeralDataProtectionProvider` 并使用它来保护数据并对其取消保护。

```csharp
using System;
using Microsoft.AspNetCore.DataProtection;

public class Program
{
    public static void Main(string[] args)
    {
        const string purpose = "Ephemeral.App.v1";

        // create an ephemeral provider and demonstrate that it can round-trip a payload
        var provider = new EphemeralDataProtectionProvider();
        var protector = provider.CreateProtector(purpose);
        Console.Write("Enter input: ");
        string input = Console.ReadLine();

        // protect the payload
        string protectedPayload = protector.Protect(input);
        Console.WriteLine($"Protect returned: {protectedPayload}");

        // unprotect the payload
        string unprotectedPayload = protector.Unprotect(protectedPayload);
        Console.WriteLine($"Unprotect returned: {unprotectedPayload}");

        // if I create a new ephemeral provider, it won't be able to unprotect existing
        // payloads, even if I specify the same purpose
        provider = new EphemeralDataProtectionProvider();
        protector = provider.CreateProtector(purpose);
        unprotectedPayload = protector.Unprotect(protectedPayload); // THROWS
    }
}

/*
* SAMPLE OUTPUT
*
* Enter input: Hello!
* Protect returned: CfDJ8AAAAAAAAAAAAAAAAAAAAA...uGoxWLjGKtm1SkNACQ
* Unprotect returned: Hello!
* << throws CryptographicException >>
*/
```
