---
title: ASP.NET Core 中的密钥存储格式
author: rick-anderson
description: 了解 ASP.NET Core 数据保护密钥存储格式的实现细节。
ms.author: riande
ms.date: 04/08/2020
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
uid: security/data-protection/implementation/key-storage-format
ms.openlocfilehash: 4a8503964c98d1828dc9d02640a7621b370e679c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060139"
---
# <a name="key-storage-format-in-aspnet-core"></a>ASP.NET Core 中的密钥存储格式

<a name="data-protection-implementation-key-storage-format"></a>

对象静态存储在 XML 表示形式中。 密钥存储的默认目录为：

* Windows： *%LOCALAPPDATA%\ASP.NET\DataProtection-Keys\*
* macOS/Linux： *$HOME/.aspnet/dataprotection-keys*

## <a name="the-key-element"></a>\<key> 元素

键作为顶级对象存在于密钥存储库中。 按约定键具有文件名 **key {guid} .xml** ，其中 {guid} 是密钥的 id。 每个这样的文件都包含一个键。 文件的格式如下所示。

```xml
<?xml version="1.0" encoding="utf-8"?>
<key id="80732141-ec8f-4b80-af9c-c4d2d1ff8901" version="1">
  <creationDate>2015-03-19T23:32:02.3949887Z</creationDate>
  <activationDate>2015-03-19T23:32:02.3839429Z</activationDate>
  <expirationDate>2015-06-17T23:32:02.3839429Z</expirationDate>
  <descriptor deserializerType="{deserializerType}">
    <descriptor>
      <encryption algorithm="AES_256_CBC" />
      <validation algorithm="HMACSHA256" />
      <enc:encryptedSecret decryptorType="{decryptorType}" xmlns:enc="...">
        <encryptedKey>
          <!-- This key is encrypted with Windows DPAPI. -->
          <value>AQAAANCM...8/zeP8lcwAg==</value>
        </encryptedKey>
      </enc:encryptedSecret>
    </descriptor>
  </descriptor>
</key>
```

\<key>元素包含以下属性和子元素：

* 密钥 id。此值被视为权威值;文件名只是一种 nicety 的可读性。

* 元素的版本 \<key> ，当前已固定为1。

* 密钥的创建、激活和到期日期。

* 一个 \<descriptor> 元素，其中包含有关此密钥中包含的经过身份验证的加密实现的信息。

在上面的示例中，密钥 id 是 {80732141-ec8f-4b80-af9c-c4d2d1ff8901}，它是在年3月 2015 19 日创建和激活的，它的生存期为90天。 偶尔 (，激活日期可能会略微早于创建日期，如本示例中所示。 这是因为 Api 的工作方式吹毛求疵，而在实践中是无害的。 ) 

## <a name="the-descriptor-element"></a>\<descriptor> 元素

外部 \<descriptor> 元素包含属性 deserializerType，它是实现 IAuthenticatedEncryptorDescriptorDeserializer 的类型的程序集限定名称。 此类型负责读取内部 \<descriptor> 元素以及分析中包含的信息。

元素的特定格式 \<descriptor> 取决于由密钥封装的经过身份验证的加密器实现，并且每个反序列化程序类型都需要为此设置略有不同的格式。 但一般情况下，此元素包含 (名称、类型、Oid 或类似) 和密钥材料的算法信息。 在上面的示例中，描述符指定此密钥包装 AES-256-CBC encryption + HMACSHA256 验证。

## <a name="the-encryptedsecret-element"></a>\<encryptedSecret> 元素

如果 [启用静态加密机密，](xref:security/data-protection/implementation/key-encryption-at-rest)则包含加密形式的密钥材料的 **&lt; encryptedSecret &gt;** 元素可能存在。 特性 `decryptorType` 是实现 [IXmlDecryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.ixmldecryptor)的类型的程序集限定名称。 此类型负责读取内部 **&lt; encryptedKey &gt;** 元素，并对其进行解密以恢复原始纯文本。

与一样 `<descriptor>` ，元素的特定格式 `<encryptedSecret>` 依赖于正在使用的静态加密机制。 在上面的示例中，使用 Windows DPAPI 按注释对主密钥进行加密。

## <a name="the-revocation-element"></a>\<revocation> 元素

吊销作为顶级对象存在于密钥存储库中。 按照约定吊销，使用 filename **吊销-{timestamp} .xml** (来撤消在特定日期) 或 **吊销-{guid} .xml** (吊销特定密钥) 之前的所有密钥。 每个文件都包含单个 \<revocation> 元素。

对于单个密钥的吊销，文件内容将如下所示。

```xml
<?xml version="1.0" encoding="utf-8"?>
<revocation version="1">
  <revocationDate>2015-03-20T22:45:30.2616742Z</revocationDate>
  <key id="eb4fc299-8808-409d-8a34-23fc83d026c9" />
  <reason>human-readable reason</reason>
</revocation>
```

在这种情况下，仅吊销指定的密钥。 不过，如果密钥 id 为 "*"，如以下示例中所示，创建日期早于指定吊销日期的所有密钥都将被吊销。

```xml
<?xml version="1.0" encoding="utf-8"?>
<revocation version="1">
  <revocationDate>2015-03-20T15:45:45.7366491-07:00</revocationDate>
  <!-- All keys created before the revocation date are revoked. -->
  <key id="*" />
  <reason>human-readable reason</reason>
</revocation>
```

\<reason>系统永远不会读取元素。 这只是一个方便的位置，用于存储可供用户阅读的吊销原因。
