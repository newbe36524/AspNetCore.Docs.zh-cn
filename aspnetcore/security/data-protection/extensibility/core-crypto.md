---
title: ASP.NET Core 中的核心加密扩展性
author: rick-anderson
description: 了解 IAuthenticatedEncryptor、IAuthenticatedEncryptorDescriptor、IAuthenticatedEncryptorDescriptorDeserializer 和顶层工厂。
ms.author: riande
ms.date: 08/11/2017
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
uid: security/data-protection/extensibility/core-crypto
ms.openlocfilehash: 99ce283d56a6010ddd846f21e0ca9a9a324d55fc
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060776"
---
# <a name="core-cryptography-extensibility-in-aspnet-core"></a>ASP.NET Core 中的核心加密扩展性

<a name="data-protection-extensibility-core-crypto"></a>

>[!WARNING]
> 实现以下任何接口的类型对于多个调用方应是线程安全的。

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptor"></a>

## <a name="iauthenticatedencryptor"></a>IAuthenticatedEncryptor

**IAuthenticatedEncryptor** 接口是加密子系统的基本构建基块。 通常，每个密钥有一个 IAuthenticatedEncryptor，IAuthenticatedEncryptor 实例包装执行加密操作所需的所有加密密钥材料和算法信息。

顾名思义，类型负责提供经过身份验证的加密和解密服务。 它公开了以下两个 Api。

* `Decrypt(ArraySegment<byte> ciphertext, ArraySegment<byte> additionalAuthenticatedData) : byte[]`

* `Encrypt(ArraySegment<byte> plaintext, ArraySegment<byte> additionalAuthenticatedData) : byte[]`

加密方法返回包含到加密纯文本和身份验证标记的 blob。 身份验证标记必须包含附加的经过身份验证的数据 (AAD) ，但 AAD 本身无需从最终有效负载恢复。 解密方法验证身份验证标记并返回已解码的有效负载。 除 System.argumentnullexception 和类似) 之外的所有故障 (都应该 homogenized 到 System.security.cryptography.cryptographicexception。

> [!NOTE]
> IAuthenticatedEncryptor 实例本身实际上不需要包含密钥材料。 例如，实现可能会委托给所有操作的 HSM。

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptorfactory"></a>
<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor"></a>

## <a name="how-to-create-an-iauthenticatedencryptor"></a>如何创建 IAuthenticatedEncryptor

# <a name="aspnet-core-2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

**IAuthenticatedEncryptorFactory** 接口表示知道如何创建 [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor)实例的类型。 其 API 如下所示。

* CreateEncryptorInstance (IKey key) ： IAuthenticatedEncryptor

对于任何给定的 IKey 实例，由其 CreateEncryptorInstance 方法创建的任何经过身份验证的加密程序都应视为等效，如以下代码示例所示。

```csharp
// we have an IAuthenticatedEncryptorFactory instance and an IKey instance
IAuthenticatedEncryptorFactory factory = ...;
IKey key = ...;

// get an encryptor instance and perform an authenticated encryption operation
ArraySegment<byte> plaintext = new ArraySegment<byte>(Encoding.UTF8.GetBytes("plaintext"));
ArraySegment<byte> aad = new ArraySegment<byte>(Encoding.UTF8.GetBytes("AAD"));
var encryptor1 = factory.CreateEncryptorInstance(key);
byte[] ciphertext = encryptor1.Encrypt(plaintext, aad);

// get another encryptor instance and perform an authenticated decryption operation
var encryptor2 = factory.CreateEncryptorInstance(key);
byte[] roundTripped = encryptor2.Decrypt(new ArraySegment<byte>(ciphertext), aad);


// the 'roundTripped' and 'plaintext' buffers should be equivalent
```

# <a name="aspnet-core-1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

**IAuthenticatedEncryptorDescriptor** 接口表示知道如何创建 [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor)实例的类型。 其 API 如下所示。

* CreateEncryptorInstance ( # A1： IAuthenticatedEncryptor

* ExportToXml ( # A1： XmlSerializedDescriptorInfo

与 IAuthenticatedEncryptor 一样，假定 IAuthenticatedEncryptorDescriptor 的实例包装一个特定键。 这意味着，对于任何给定的 IAuthenticatedEncryptorDescriptor 实例，由其 CreateEncryptorInstance 方法创建的任何经过身份验证的加密程序都应视为等效，如以下代码示例中所示。

```csharp
// we have an IAuthenticatedEncryptorDescriptor instance
IAuthenticatedEncryptorDescriptor descriptor = ...;

// get an encryptor instance and perform an authenticated encryption operation
ArraySegment<byte> plaintext = new ArraySegment<byte>(Encoding.UTF8.GetBytes("plaintext"));
ArraySegment<byte> aad = new ArraySegment<byte>(Encoding.UTF8.GetBytes("AAD"));
var encryptor1 = descriptor.CreateEncryptorInstance();
byte[] ciphertext = encryptor1.Encrypt(plaintext, aad);

// get another encryptor instance and perform an authenticated decryption operation
var encryptor2 = descriptor.CreateEncryptorInstance();
byte[] roundTripped = encryptor2.Decrypt(new ArraySegment<byte>(ciphertext), aad);


// the 'roundTripped' and 'plaintext' buffers should be equivalent
```

---

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor"></a>

## <a name="iauthenticatedencryptordescriptor-aspnet-core-2x-only"></a>IAuthenticatedEncryptorDescriptor (仅 ASP.NET Core 2.x) 

# <a name="aspnet-core-2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

**IAuthenticatedEncryptorDescriptor** 接口表示一种类型，该类型知道如何将自身导出到 XML。 其 API 如下所示。

* ExportToXml ( # A1： XmlSerializedDescriptorInfo

# <a name="aspnet-core-1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

---

## <a name="xml-serialization"></a>XML 序列化

IAuthenticatedEncryptor 和 IAuthenticatedEncryptorDescriptor 之间的主要区别在于，描述符知道如何创建加密程序并为其提供有效的参数。 假设其实现依赖于 System.security.cryptography.symmetricalgorithm 和 KeyedHashAlgorithm 的 IAuthenticatedEncryptor。 加密程序的工作就是使用这些类型，但不一定知道这些类型来自何处，因此，如果应用程序重启，则无法真正地写出如何重新创建其自身的正确说明。 描述符在此基础上充当更高的级别。 由于描述符知道如何创建加密程序实例 (例如，它知道如何创建所需的算法) ，它可以将 XML 格式的知识序列化，以便可以在应用程序重置后重新创建加密程序实例。

<a name="data-protection-extensibility-core-crypto-exporttoxml"></a>

描述符可以通过其 ExportToXml 例程进行序列化。 此例程返回一个 XmlSerializedDescriptorInfo，其中包含两个属性：描述符的 System.xml.linq.xelement> 表示形式和表示 [IAuthenticatedEncryptorDescriptorDeserializer](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer) 的类型，该类型可用于在给定相应 system.xml.linq.xelement> 的情况下恢复此描述符。

序列化描述符可能包含敏感信息，如加密密钥材料。 数据保护系统在将信息保存到存储之前，对其进行内置支持。 若要利用这一点，描述符应标记包含特性名称为 "requiresEncryption" (xmlns " <http://schemas.asp.net/2015/03/dataProtection> " ) ，值为 "true" 的敏感信息的元素。

>[!TIP]
> 有一个用于设置此属性的帮助器 API。 调用位于命名空间 Microsoft.AspNetCore.DataProtection.AuthenticatedEncryption.ConfigurationModel ( # A1 的扩展方法 System.xml.linq.xelement>。

也可能存在序列化描述符不包含敏感信息的情况。 再次考虑存储在 HSM 中的加密密钥的情况。 此描述符在序列化时无法写出密钥材料，因为 HSM 不会以纯文本形式公开材料。 相反，如果 HSM 允许以这种方式导出) 或 HSM 自己的密钥唯一标识符，则该描述符可能会写出密钥的密钥包装版本 (。

<a name="data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptordeserializer"></a>

## <a name="iauthenticatedencryptordescriptordeserializer"></a>IAuthenticatedEncryptorDescriptorDeserializer

**IAuthenticatedEncryptorDescriptorDeserializer** 接口表示知道如何从 system.xml.linq.xelement> 反序列化 IAuthenticatedEncryptorDescriptor 实例的类型。 它公开了单一方法：

* ImportFromXml (System.xml.linq.xelement> 元素) ： IAuthenticatedEncryptorDescriptor

ImportFromXml 方法采用 [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-exporttoxml) 返回的 system.xml.linq.xelement>，并创建原始 IAuthenticatedEncryptorDescriptor 的等效项。

实现 IAuthenticatedEncryptorDescriptorDeserializer 的类型应具有以下两个公共构造函数之一：

* .ctor (IServiceProvider) 

* .ctor ( # A1

> [!NOTE]
> 传递给构造函数的 IServiceProvider 可能为 null。

## <a name="the-top-level-factory"></a>顶级工厂

# <a name="aspnet-core-2x"></a>[ASP.NET Core 2.x](#tab/aspnetcore2x)

**AlgorithmConfiguration** 类表示一种知道如何创建 [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor)实例的类型。 它公开一个 API。

* CreateNewDescriptor ( # A1： IAuthenticatedEncryptorDescriptor

将 AlgorithmConfiguration 看作顶级工厂。 配置用作模板。 它包装算法信息 (例如，此配置会生成包含 128-GCM 主密钥) 的描述符，但尚未与特定密钥相关联。

调用 CreateNewDescriptor 时，将仅为此调用创建新的密钥材料，并生成新的 IAuthenticatedEncryptorDescriptor 来包装此密钥材料以及使用该材料所需的算法信息。 密钥材料可以在软件 (中创建，并保存在内存) 中，可以在 HSM 中创建和保存。 关键是，对 CreateNewDescriptor 的任何两次调用都不应创建等效的 IAuthenticatedEncryptorDescriptor 实例。

AlgorithmConfiguration 类型用作密钥创建例程（如 [自动密钥滚动](xref:security/data-protection/implementation/key-management#key-expiration-and-rolling)）的入口点。 若要更改所有将来键的实现，请在 KeyManagementOptions 中设置 AuthenticatedEncryptorConfiguration 属性。

# <a name="aspnet-core-1x"></a>[ASP.NET Core 1.x](#tab/aspnetcore1x)

**IAuthenticatedEncryptorConfiguration** 接口表示知道如何创建 [IAuthenticatedEncryptorDescriptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptordescriptor)实例的类型。 它公开一个 API。

* CreateNewDescriptor ( # A1： IAuthenticatedEncryptorDescriptor

将 IAuthenticatedEncryptorConfiguration 看作顶级工厂。 配置用作模板。 它包装算法信息 (例如，此配置会生成包含 128-GCM 主密钥) 的描述符，但尚未与特定密钥相关联。

调用 CreateNewDescriptor 时，将仅为此调用创建新的密钥材料，并生成新的 IAuthenticatedEncryptorDescriptor 来包装此密钥材料以及使用该材料所需的算法信息。 密钥材料可以在软件 (中创建，并保存在内存) 中，可以在 HSM 中创建和保存。 关键是，对 CreateNewDescriptor 的任何两次调用都不应创建等效的 IAuthenticatedEncryptorDescriptor 实例。

IAuthenticatedEncryptorConfiguration 类型用作密钥创建例程（如 [自动密钥滚动](xref:security/data-protection/implementation/key-management#key-expiration-and-rolling)）的入口点。 若要更改所有将来键的实现，请在服务容器中注册单一实例 IAuthenticatedEncryptorConfiguration。

---
