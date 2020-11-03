---
title: Windows 和 Azure 中的静态密钥加密使用 ASP.NET Core
author: rick-anderson
description: 了解 ASP.NET Core 静态数据保护密钥加密的实现细节。
ms.author: riande
ms.date: 07/16/2018
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
uid: security/data-protection/implementation/key-encryption-at-rest
ms.openlocfilehash: e2841278e8262e6dbbfcf172e0b6599bb19f1d6c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061270"
---
# <a name="key-encryption-at-rest-in-windows-and-azure-using-aspnet-core"></a>Windows 和 Azure 中的静态密钥加密使用 ASP.NET Core

默认情况下，数据保护系统 [使用发现机制](xref:security/data-protection/configuration/default-settings) 来确定应如何对加密密钥进行静态加密。 开发人员可以重写发现机制，并手动指定密钥的加密方式。

> [!WARNING]
> 如果指定显式 [密钥持久性位置](xref:security/data-protection/implementation/key-storage-providers)，数据保护系统将注销静态密钥加密机制。 因此，不再静态加密密钥。 建议为生产部署 [指定显式密钥加密机制](xref:security/data-protection/implementation/key-encryption-at-rest) 。 本主题介绍了静态加密机制选项。

::: moniker range=">= aspnetcore-2.1"

## <a name="azure-key-vault"></a>Azure Key Vault

若要在 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)中存储密钥，请在类中配置 [ProtectKeysWithAzureKeyVault](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.protectkeyswithazurekeyvault) 的系统 `Startup` ：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blobUriWithSasToken>"))
        .ProtectKeysWithAzureKeyVault("<keyIdentifier>", "<clientId>", "<clientSecret>");
}
```

有关详细信息，请参阅 [Configure ASP.NET Core Data Protection： ProtectKeysWithAzureKeyVault](xref:security/data-protection/configuration/overview#protectkeyswithazurekeyvault)。

::: moniker-end

## <a name="windows-dpapi"></a>Windows DPAPI

**仅适用于 Windows 部署。**

使用 Windows DPAPI 时，将使用 [CryptProtectData](/windows/desktop/api/dpapi/nf-dpapi-cryptprotectdata) 对密钥材料进行加密，然后将其保存到存储中。 DPAPI 是一种适用于当前计算机之外从不读取的数据的适当加密机制 (不过，可以将这些密钥上移到 Active Directory;请参阅 [DPAPI 和漫游配置文件](https://support.microsoft.com/kb/309408/#6)) 。 若要配置 DPAPI 静态密钥加密，请调用 [ProtectKeysWithDpapi](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithdpapi) 扩展方法之一：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Only the local user account can decrypt the keys
    services.AddDataProtection()
        .ProtectKeysWithDpapi();
}
```

如果 `ProtectKeysWithDpapi` 在没有参数的情况下调用，则只有当前的 Windows 用户帐户才能解密持久化密钥环。 您可以选择指定计算机上的任何用户帐户 (不只是当前用户帐户，) 能够破译密钥环：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // All user accounts on the machine can decrypt the keys
    services.AddDataProtection()
        .ProtectKeysWithDpapi(protectToLocalMachine: true);
}
```

::: moniker range=">= aspnetcore-2.0"

## <a name="x509-certificate"></a>X.509 证书

如果应用分布在多台计算机上，则在计算机上分发共享的 x.509 证书，并将托管应用配置为使用证书进行静态密钥加密可能会很方便。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .ProtectKeysWithCertificate("3BCE558E2AD3E0E34A7743EAB5AEA2A9BD2575A0");
}
```

由于 .NET Framework 限制，仅支持具有 CAPI 私钥的证书。 请参阅下面的内容，了解这些限制的可能解决方法。

::: moniker-end

## <a name="windows-dpapi-ng"></a>Windows DPAPI-NG

**此机制仅在 Windows 8/Windows Server 2012 或更高版本上可用。**

从 Windows 8 开始，Windows OS 支持 DPAPI-NG (也称为 CNG DPAPI) 。 有关详细信息，请参阅 [关于 CNG DPAPI](/windows/desktop/SecCNG/cng-dpapi)。

主体编码为保护描述符规则。 在以下调用 [ProtectKeysWithDpapiNG](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithdpaping)的示例中，只有具有指定 SID 的已加入域的用户才能解密密钥环：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Uses the descriptor rule "SID=S-1-5-21-..."
    services.AddDataProtection()
        .ProtectKeysWithDpapiNG("SID=S-1-5-21-...",
        flags: DpapiNGProtectionDescriptorFlags.None);
}
```

还有一个无参数的重载 `ProtectKeysWithDpapiNG` 。 使用此简便方法指定规则 "SID = {CURRENT_ACCOUNT_SID}"，其中 *CURRENT_ACCOUNT_SID* 是当前 Windows 用户帐户的 SID：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Use the descriptor rule "SID={current account SID}"
    services.AddDataProtection()
        .ProtectKeysWithDpapiNG();
}
```

在此方案中，AD 域控制器负责分发由 DPAPI-NG 操作使用的加密密钥。 如果进程在其标识) 下运行，则目标用户可以通过任何已加入域的计算机来解密已加密的有效负载 (。

## <a name="certificate-based-encryption-with-windows-dpapi-ng"></a>基于证书的加密和 Windows DPAPI-NG

如果应用在 Windows 8.1/Windows Server 2012 R2 或更高版本上运行，则可以使用 Windows DPAPI-NG 执行基于证书的加密。 使用规则描述符字符串 "CERTIFICATE = HashId： THUMBPRINT"，其中 *THUMBPRINT* 是证书的十六进制编码的 SHA1 指纹：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .ProtectKeysWithDpapiNG("CERTIFICATE=HashId:3BCE558E2...B5AEA2A9BD2575A0",
            flags: DpapiNGProtectionDescriptorFlags.None);
}
```

指向此存储库的任何应用都必须在 Windows 8.1/Windows Server 2012 R2 或更高版本上运行，才能解密密钥。

## <a name="custom-key-encryption"></a>自定义密钥加密

如果不适合使用机箱内机制，开发人员可以通过提供自定义 [IXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.ixmlencryptor)来指定其自己的密钥加密机制。
