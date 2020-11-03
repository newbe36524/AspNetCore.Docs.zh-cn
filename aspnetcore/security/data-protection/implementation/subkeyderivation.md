---
title: ASP.NET Core 中的子项派生和已验证的加密
author: rick-anderson
description: 了解 ASP.NET Core 数据保护子项派生和经过身份验证的加密的实现细节。
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
uid: security/data-protection/implementation/subkeyderivation
ms.openlocfilehash: efe8ad2f71feda9cbc1693d362e30eff29cbcd74
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060152"
---
# <a name="subkey-derivation-and-authenticated-encryption-in-aspnet-core"></a>ASP.NET Core 中的子项派生和已验证的加密

<a name="data-protection-implementation-subkey-derivation"></a>

密钥环中的大多数密钥将包含某种形式的熵，并将具有说明 "CBC-模式加密 + HMAC 验证" 或 "GCM 加密 + 验证" 的算法信息。 在这些情况下，我们将嵌入的平均信息量称为此密钥的主密钥材料 (或千米) ，并执行密钥派生函数来派生用于实际加密操作的密钥。

> [!NOTE]
> 键是抽象的，自定义实现的行为可能不如下。 如果密钥提供自己的实现 `IAuthenticatedEncryptor` 而不是使用我们的某个内置工厂，则本部分中所述的机制将不再适用。

<a name="data-protection-implementation-subkey-derivation-aad"></a>

## <a name="additional-authenticated-data-and-subkey-derivation"></a>附加经过身份验证的数据和子项派生

`IAuthenticatedEncryptor`接口用作所有经过身份验证的加密操作的核心接口。 其 `Encrypt` 方法使用两个缓冲区：纯文本和 additionalAuthenticatedData (AAD) 。 纯文本内容在对的调用中保持不变 `IDataProtector.Protect` ，但 AAD 由系统生成并由三个部分组成：

1. 32位幻标头 09 F0 C9 F0，用于标识此版本的数据保护系统。

2. 128位密钥 id。

3. 由创建了执行此操作的的、用于创建的可变长度字符串 `IDataProtector` 。

由于 AAD 对于所有三个组件的元组都是唯一的，因此我们可以使用它从公里派生新密钥，而不是在我们的所有加密操作中使用公里本身。 对于每个调用 `IAuthenticatedEncryptor.Encrypt` ，将发生以下密钥派生过程：

`( K_E, K_H ) = SP800_108_CTR_HMACSHA512(K_M, AAD, contextHeader || keyModifier)`

此处，我们将在计数器模式下调用 NIST SP800-108 KDF (参阅 [NIST SP800-108](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-108.pdf)，5.1) ，其参数如下：

* 密钥派生密钥 (KDK) = `K_M`

* PRF = HMACSHA512

* 标签 = additionalAuthenticatedData

* context = contextHeader | |keyModifier

上下文标头的长度可变，主要用作我们要派生和的算法的指纹 `K_E` `K_H` 。 密钥修饰符是为每次调用随机生成的128位字符串， `Encrypt` 可确保 KE 和 KH 对于此特定身份验证加密操作是唯一的，即使 KDF 的所有其他输入都是常量，也是如此。

对于 CBC 模式加密 + HMAC 验证操作， `| K_E |` 是对称块加密密钥的长度， `| K_H |` 是 HMAC 例程的摘要大小。 对于 GCM 加密 + 验证操作，为 `| K_H | = 0` 。

## <a name="cbc-mode-encryption--hmac-validation"></a>CBC 模式加密 + HMAC 验证

`K_E`通过上述机制生成一次后，我们将生成一个随机初始化向量，并运行对称块加密算法以加密纯文本。 然后，初始化向量和密码文本通过用密钥初始化的 HMAC 例程运行， `K_H` 以生成 MAC。 下面以图形方式表示此过程和返回值。

![CBC 模式进程和返回](subkeyderivation/_static/cbcprocess.png)

`output:= keyModifier || iv || E_cbc (K_E,iv,data) || HMAC(K_H, iv || E_cbc (K_E,iv,data))`

> [!NOTE]
> 在 `IDataProtector.Protect` 将 [幻标题和密钥 id](xref:security/data-protection/implementation/authenticated-encryption-details) 返回到调用方之前，实现会将其追加到输出之前。 由于幻标头和密钥 id 是 [AAD](xref:security/data-protection/implementation/subkeyderivation#data-protection-implementation-subkey-derivation-aad)的一部分，并且由于密钥修饰符作为输入送回 KDF，这意味着最终返回的有效负载的每个字节都由 MAC 进行身份验证。

## <a name="galoiscounter-mode-encryption--validation"></a>Galois/Counter 模式加密 + 验证

`K_E`通过上述机制生成一次后，我们将生成一个随机的96位 nonce，并运行对称块加密算法以加密纯文本并生成128位身份验证标记。

![GCM 模式进程和返回](subkeyderivation/_static/galoisprocess.png)

`output := keyModifier || nonce || E_gcm (K_E,nonce,data) || authTag`

> [!NOTE]
> 尽管 GCM 本身支持 AAD 的概念，但我们仍只向原始 KDF 提供 AAD，选择将空字符串传递给 GCM 以用于其 AAD 参数。 这样做的原因是两折。 首先， [为了支持灵活性](xref:security/data-protection/implementation/context-headers#data-protection-implementation-context-headers) ，我们永远不希望将其 `K_M` 作为加密密钥直接使用。 此外，GCM 对其输入施加非常严格的唯一性要求。 对于两个或多个具有相同 (键的不同输入数据集，GCM) 对的概率不能超过 2 ^ 32。 如果修复此问题，我们在 `K_E` 运行落入的 2 ^-32 限制之前，不能执行 2 ^ 32 个以上的加密操作。 这似乎是一种非常大的操作，但高流量 web 服务器可以在一天内只经过4000000000请求，在这些密钥的正常生存期内。 为了保持符合 2 ^-32 概率限制，我们继续使用128位的密钥修饰符和96位 nonce，这会大大扩展任何给定的可用操作计数 `K_M` 。 为简单起见，我们在 CBC 和 GCM 操作之间共享 KDF 代码路径，由于 AAD 已在 KDF 中考虑，因此无需将其转发到 GCM 例程。
