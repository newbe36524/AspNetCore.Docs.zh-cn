---
title: ASP.NET Core 中的密钥管理扩展性
author: rick-anderson
description: 了解 ASP.NET Core 的数据保护密钥管理扩展性。
ms.author: riande
ms.custom: mvc, seodec18
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
uid: security/data-protection/extensibility/key-management
ms.openlocfilehash: 08ec018238d18cd253e793267c5b146df8319485
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060763"
---
# <a name="key-management-extensibility-in-aspnet-core"></a>ASP.NET Core 中的密钥管理扩展性

阅读本部分之前，请阅读 [密钥管理](xref:security/data-protection/implementation/key-management#data-protection-implementation-key-management) 部分，因为它说明了这些 api 背后的一些基本概念。

**警告** ：对于多个调用方，实现以下任何接口的类型应该是线程安全的。

## <a name="key"></a>密钥

`IKey`接口是 cryptosystem 中密钥的基本表示形式。 此处的术语关键字在抽象意义上使用，而不是在 "加密密钥材料" 的文字意义上使用。 密钥具有以下属性：

* 激活、创建和到期日期

* 吊销状态

* GUID)  (的密钥标识符

::: moniker range=">= aspnetcore-2.0"

此外，还 `IKey` 公开了一个 `CreateEncryptor` 方法，该方法可用于创建绑定到此密钥的 [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) 实例。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

此外，还 `IKey` 公开了一个 `CreateEncryptorInstance` 方法，该方法可用于创建绑定到此密钥的 [IAuthenticatedEncryptor](xref:security/data-protection/extensibility/core-crypto#data-protection-extensibility-core-crypto-iauthenticatedencryptor) 实例。

::: moniker-end

> [!NOTE]
> 没有用于从实例中检索原始加密材料的 API `IKey` 。

## <a name="ikeymanager"></a>IKeyManager

`IKeyManager`接口表示负责常规密钥存储、检索和操作的对象。 它公开了三个高级操作：

* 创建新密钥并将其保存到存储中。

* 从存储获取所有密钥。

* 撤消一个或多个密钥并将吊销信息保存到存储中。

>[!WARNING]
> 编写 `IKeyManager` 是一种非常高级的任务，大多数开发人员都不应尝试。 相反，大多数开发人员应充分利用 [XmlKeyManager](#xmlkeymanager) 类提供的功能。

## <a name="xmlkeymanager"></a>XmlKeyManager

`XmlKeyManager`类型是的机箱内的具体实现 `IKeyManager` 。 它提供若干有用的功能，包括静态密钥的密钥委托和加密。 此系统中的键表示为 XML 元素 (具体说来就是 [system.xml.linq.xelement>](/dotnet/csharp/programming-guide/concepts/linq/xelement-class-overview)) 。

`XmlKeyManager` 依赖于完成其任务的过程中的多个其他组件：

::: moniker range=">= aspnetcore-2.0"

* `AlgorithmConfiguration`，它指示新密钥使用的算法。

* `IXmlRepository`，控制在存储中保留密钥的位置。

* `IXmlEncryptor` [可选]，这允许静态加密密钥。

* `IKeyEscrowSink` [可选]，它提供密钥委托服务。

::: moniker-end

::: moniker range="< aspnetcore-2.0"

* `IXmlRepository`，控制在存储中保留密钥的位置。

* `IXmlEncryptor` [可选]，这允许静态加密密钥。

* `IKeyEscrowSink` [可选]，它提供密钥委托服务。

::: moniker-end

下面是高级关系图，这些关系图指示如何在中将这些组件连接在一起 `XmlKeyManager` 。

::: moniker range=">= aspnetcore-2.0"

![密钥创建](key-management/_static/keycreation2.png)

*密钥创建/CreateNewKey*

在的实现中 `CreateNewKey` ， `AlgorithmConfiguration` 组件用于创建一个唯一的 `IAuthenticatedEncryptorDescriptor` ，然后将其序列化为 XML。 如果存在密钥托管接收器，则会提供原始 (未加密的) XML，用于长期存储的接收器。 `IXmlEncryptor`如果需要) 生成加密的 xml 文档，则会通过 (运行未加密的 xml。 此加密文档通过将持久保存到长期存储 `IXmlRepository` 。  (如果未 `IXmlEncryptor` 配置，则会将未加密的文档保留在中 `IXmlRepository` 。 ) 

![密钥检索](key-management/_static/keyretrieval2.png)

::: moniker-end

::: moniker range="< aspnetcore-2.0"

![密钥创建](key-management/_static/keycreation1.png)

*密钥创建/CreateNewKey*

在的实现中 `CreateNewKey` ， `IAuthenticatedEncryptorConfiguration` 组件用于创建一个唯一的 `IAuthenticatedEncryptorDescriptor` ，然后将其序列化为 XML。 如果存在密钥托管接收器，则会提供原始 (未加密的) XML，用于长期存储的接收器。 `IXmlEncryptor`如果需要) 生成加密的 xml 文档，则会通过 (运行未加密的 xml。 此加密文档通过将持久保存到长期存储 `IXmlRepository` 。  (如果未 `IXmlEncryptor` 配置，则会将未加密的文档保留在中 `IXmlRepository` 。 ) 

![密钥检索](key-management/_static/keyretrieval1.png)

::: moniker-end

*密钥检索/GetAllKeys*

在的实现中 `GetAllKeys` ，将从基础读取表示键和吊销的 XML 文档 `IXmlRepository` 。 如果这些文档已加密，系统会自动将其解密。 `XmlKeyManager` 创建相应的 `IAuthenticatedEncryptorDescriptorDeserializer` 实例，以将文档反序列化回 `IAuthenticatedEncryptorDescriptor` 实例，然后将其包装在单个 `IKey` 实例中。 此实例集合 `IKey` 将返回到调用方。

有关特定 XML 元素的详细信息，请参阅 [密钥存储格式文档](xref:security/data-protection/implementation/key-storage-format#data-protection-implementation-key-storage-format)。

## <a name="ixmlrepository"></a>IXmlRepository

`IXmlRepository`接口表示可将 xml 持久保存到后备存储并从中检索 xml 的类型。 它公开两个 Api：

* `GetAllElements` :`IReadOnlyCollection<XElement>`

* `StoreElement(XElement element, string friendlyName)`

的实现 `IXmlRepository` 不需要分析通过它们传递的 XML。 它们应将 XML 文档视为不透明，并使更高的层担心生成和分析文档。

有四种内置的具体类型可实现 `IXmlRepository` ：

::: moniker range=">= aspnetcore-2.2"

* [FileSystemXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.filesystemxmlrepository)
* [RegistryXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.registryxmlrepository)
* [AzureStorage. AzureBlobXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.azurestorage.azureblobxmlrepository)
* [RedisXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.stackexchangeredis.redisxmlrepository)

::: moniker-end

::: moniker range="< aspnetcore-2.2"

* [FileSystemXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.filesystemxmlrepository)
* [RegistryXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.repositories.registryxmlrepository)
* [AzureStorage. AzureBlobXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.azurestorage.azureblobxmlrepository)
* [RedisXmlRepository](/dotnet/api/microsoft.aspnetcore.dataprotection.redisxmlrepository)

::: moniker-end

有关详细信息，请参阅 [密钥存储提供程序文档](xref:security/data-protection/implementation/key-storage-providers) 。

`IXmlRepository`当使用不同的后备存储 (例如，Azure 表存储) 时，注册自定义适用。

若要更改应用程序范围内的默认存储库，请注册自定义 `IXmlRepository` 实例：

::: moniker range=">= aspnetcore-2.0"

```csharp
services.Configure<KeyManagementOptions>(options => options.XmlRepository = new MyCustomXmlRepository());
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
services.AddSingleton<IXmlRepository>(new MyCustomXmlRepository());
```

::: moniker-end

## <a name="ixmlencryptor"></a>IXmlEncryptor

`IXmlEncryptor`接口表示可加密纯文本 XML 元素的类型。 它公开一个 API：

* 加密 (System.xml.linq.xelement> plaintextElement) ： EncryptedXmlInfo

如果序列化的 `IAuthenticatedEncryptorDescriptor` 包含任何标记为 "需要加密" 的元素，则 `XmlKeyManager` 将通过配置 `IXmlEncryptor` 的的方法运行这些元素 `Encrypt` ，并将到加密元素而不是纯文本元素保存到 `IXmlRepository` 。 此方法的输出 `Encrypt` 是一个 `EncryptedXmlInfo` 对象。 此对象是一个包装，其中包含结果到加密 `XElement` 和类型，该类型表示 `IXmlDecryptor` 可用于对相应元素进行解密的。

有四种内置的具体类型可实现 `IXmlEncryptor` ：

* [CertificateXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.certificatexmlencryptor)
* [DpapiNGXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.dpapingxmlencryptor)
* [DpapiXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.dpapixmlencryptor)
* [NullXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.nullxmlencryptor)

有关详细信息，请参阅 [静态密钥加密文档](xref:security/data-protection/implementation/key-encryption-at-rest) 。

若要在应用程序范围内更改默认的密钥加密机制，请注册自定义 `IXmlEncryptor` 实例：

::: moniker range=">= aspnetcore-2.0"

```csharp
services.Configure<KeyManagementOptions>(options => options.XmlEncryptor = new MyCustomXmlEncryptor());
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
services.AddSingleton<IXmlEncryptor>(new MyCustomXmlEncryptor());
```

::: moniker-end

## <a name="ixmldecryptor"></a>IXmlDecryptor

`IXmlDecryptor`接口表示一种类型，该类型知道如何对 `XElement` 通过进行到加密的进行解密 `IXmlEncryptor` 。 它公开一个 API：

* 解密 (System.xml.linq.xelement> encryptedElement) ： System.xml.linq.xelement>

`Decrypt`方法撤消执行的加密 `IXmlEncryptor.Encrypt` 。 通常，每个具体 `IXmlEncryptor` 的实现都具有相应的具体 `IXmlDecryptor` 实现。

实现的类型 `IXmlDecryptor` 应具有以下两个公共构造函数之一：

* .ctor (IServiceProvider) 
* .ctor ( # A1

> [!NOTE]
> `IServiceProvider`传递给构造函数的不能为 null。

## <a name="ikeyescrowsink"></a>IKeyEscrowSink

`IKeyEscrowSink`接口表示可以执行敏感信息的委托的类型。 请记住，序列化描述符可能包含敏感信息 (如加密材料) ，这就是第一处引入 [IXmlEncryptor](#ixmlencryptor) 类型的结果。 但是，会发生事故，关键环可能会被删除或损坏。

托管接口提供了紧急转义影线，允许在任何已配置的 [IXmlEncryptor](#ixmlencryptor)转换之前访问原始序列化的 XML。 接口公开单个 API：

* 存储 (Guid keyId，System.xml.linq.xelement> 元素) 

这是由 `IKeyEscrowSink` 实现来处理所提供的元素，其安全方式与业务策略一致。 一个可能的实现可能是，托管接收器使用已知的公司 x.509 证书（其中证书的私钥已托管）对 XML 元素进行加密; `CertificateXmlEncryptor` 类型可帮助此。 `IKeyEscrowSink`实现还负责正确保存提供的元素。

默认情况下，不启用任何托管机制，尽管服务器管理员可对 [此进行全局配置](xref:security/data-protection/configuration/machine-wide-policy)。 它还可通过方法以编程方式进行配置 `IDataProtectionBuilder.AddKeyEscrowSink` ，如下面的示例中所示。 `AddKeyEscrowSink`方法重载镜像 `IServiceCollection.AddSingleton` 和 `IServiceCollection.AddInstance` 重载，因为实例要 `IKeyEscrowSink` 单一实例。 如果 `IKeyEscrowSink` 注册了多个实例，则会在密钥生成过程中调用每个实例，以便可以将密钥同时托管到多个机制。

没有用于从实例读取材料的 API `IKeyEscrowSink` 。 这与托管机制的设计理论一致：它旨在使密钥材料可供受信任的颁发机构访问，因为该应用程序本身不是受信任的颁发机构，所以它不应访问其自己的托管材料。

下面的示例代码演示如何创建和注册 `IKeyEscrowSink` where 密钥的托管，以便只有 "CONTOSODomain Admins" 的成员才能恢复它们。

> [!NOTE]
> 若要运行此示例，你必须在已加入域的 Windows 8/Windows Server 2012 计算机上，并且域控制器必须为 Windows Server 2012 或更高版本。

[!code-csharp[](key-management/samples/key-management-extensibility.cs)]
