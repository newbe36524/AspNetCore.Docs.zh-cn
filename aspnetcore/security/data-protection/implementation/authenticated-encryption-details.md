---
title: ASP.NET Core 中经过身份验证的加密详细信息
author: rick-anderson
description: 了解 ASP.NET Core 数据保护身份验证加密的实现细节。
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
uid: security/data-protection/implementation/authenticated-encryption-details
ms.openlocfilehash: 7978725534cdd3a5b425851f61b1c7ae3ada88df
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051832"
---
# <a name="authenticated-encryption-details-in-aspnet-core"></a>ASP.NET Core 中经过身份验证的加密详细信息

<a name="data-protection-implementation-authenticated-encryption-details"></a>

对 IDataProtector 的调用是经过身份验证的加密操作。 保护方法同时提供机密性和真实性，并与用于从其根 IDataProtectionProvider 派生此特定 IDataProtector 实例的用途链相关联。

IDataProtector 使用 byte [] 纯文本参数，并生成一个 byte [] 受保护的负载，其格式如下所述。  (还有一个扩展方法重载，它采用字符串纯文本参数并返回一个字符串受保护的负载。 如果使用此 API，受保护的负载格式仍将具有以下结构，但它将被 [base64url 编码](https://tools.ietf.org/html/rfc4648#section-5)。 ) 

## <a name="protected-payload-format"></a>受保护的负载格式

受保护的负载格式由三个主要组件组成：

* 用于标识数据保护系统版本的32位幻标头。

* 一个128位密钥 id，用于标识用于保护此特定有效负载的密钥。

* 受保护负载的其余部分 [特定于此密钥封装的加密程序](xref:security/data-protection/implementation/subkeyderivation#data-protection-implementation-subkey-derivation)。 在下面的示例中，键表示 AES-256-CBC + HMACSHA256 加密器，负载进一步细分，如下所示：
  * 128位密钥修饰符。
  * 128位初始化向量。
  * 48字节的 AES-256-CBC 输出。
  * HMACSHA256 authentication 标记。

下面说明了一个受保护的有效负载示例。

```
09 F0 C9 F0 80 9C 81 0C 19 66 19 40 95 36 53 F8
AA FF EE 57 57 2F 40 4C 3F 7F CC 9D CC D9 32 3E
84 17 99 16 EC BA 1F 4A A1 18 45 1F 2D 13 7A 28
79 6B 86 9C F8 B7 84 F9 26 31 FC B1 86 0A F1 56
61 CF 14 58 D3 51 6F CF 36 50 85 82 08 2D 3F 73
5F B0 AD 9E 1A B2 AE 13 57 90 C8 F5 7C 95 4E 6A
8A AA 06 EF 43 CA 19 62 84 7C 11 B2 C8 71 9D AA
52 19 2E 5B 4C 1E 54 F0 55 BE 88 92 12 C1 4B 5E
52 C9 74 A0
```

从第一个32位以上的负载格式，或4个字节为标识版本 (09 F0 C9 F0 的幻标头) 

接下来的128位或16字节是密钥标识符 (80 9C 81 0C 19 66 19 40 95 36 53 F8 AA FF EE 57) 

余数包含负载并且特定于所使用的格式。

> [!WARNING]
> 所有受指定密钥保护的有效负载都将以相同的20字节 (幻数值，键 id) 标头开头。 管理员可以将此事实用于诊断目的，以大致了解生成负载的时间。 例如，上面的负载对应于键 {0c819c80-6619-4019-9536-53f8aaffee57}。 如果在检查密钥存储库之后发现此特定密钥的激活日期为2015-01-01，并且其到期日期为2015-03-01，则必须假定有效负载 (如果未篡改在该窗口中生成的) ，请在任一端提供或奶油一小部分。
