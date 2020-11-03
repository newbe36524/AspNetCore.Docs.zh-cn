---
title: 配置 ASP.NET Core 数据保护
author: rick-anderson
description: 了解如何在 ASP.NET Core 中配置数据保护。
ms.author: riande
ms.custom: mvc
ms.date: 11/02/2020
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
uid: security/data-protection/configuration/overview
ms.openlocfilehash: 048f6d6d57d3cc5d64004e18b18a3347ee92e450
ms.sourcegitcommit: d64bf0cbe763beda22a7728c7f10d07fc5e19262
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/03/2020
ms.locfileid: "93234564"
---
# <a name="configure-aspnet-core-data-protection"></a>配置 ASP.NET Core 数据保护

当数据保护系统已初始化时，它将基于操作环境应用 [默认设置](xref:security/data-protection/configuration/default-settings) 。 这些设置通常适用于在一台计算机上运行的应用程序。 在某些情况下，开发人员可能想要更改默认设置：

* 应用程序分布在多台计算机上。
* 出于合规性原因。

在这些情况下，数据保护系统提供了丰富的配置 API。

> [!WARNING]
> 与配置文件类似，应使用适当的权限保护数据保护密钥环。 你可以选择对静态密钥加密，但这不会阻止攻击者创建新密钥。 因此，应用的安全将受到影响。 使用数据保护配置的存储位置应该将其访问权限限制为应用本身，这与保护配置文件的方式类似。 例如，如果选择将密钥环存储在磁盘上，请使用文件系统权限。 确保你的 web 应用在其下运行的标识具有对该目录的读取、写入和创建访问权限。 如果使用 Azure Blob 存储，则只有 web 应用应能够在 Blob 存储中读取、写入或创建新条目等。
>
> 扩展方法 [AddDataProtection](/dotnet/api/microsoft.extensions.dependencyinjection.dataprotectionservicecollectionextensions.adddataprotection) 返回 [IDataProtectionBuilder](/dotnet/api/microsoft.aspnetcore.dataprotection.idataprotectionbuilder)。 `IDataProtectionBuilder` 公开扩展方法，你可以将这些方法链接在一起来配置数据保护选项。

::: moniker range=">= aspnetcore-3.0"

本文中使用的数据保护扩展插件需要以下 NuGet 包：

* [Azure.Extensions.AspNetCore.DataProtection.Blobs](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Blobs)
* [Azure.Extensions.AspNetCore.DataProtection.Keys](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Keys)

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

## <a name="protectkeyswithazurekeyvault"></a>ProtectKeysWithAzureKeyVault

使用 CLI 登录到 Azure，例如：

```azurecli
az login
``` 

若要在 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/)中存储密钥，请在类中配置 [ProtectKeysWithAzureKeyVault](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.protectkeyswithazurekeyvault) 系统 `Startup` 。 `blobUriWithSasToken` 应存储密钥文件的完整 URI。 此 URI 必须包含 SAS 令牌作为查询字符串参数：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToAzureBlobStorage(new Uri("<blobUriWithSasToken>"))
        .ProtectKeysWithAzureKeyVault(new Uri("<keyIdentifier>"), new DefaultAzureCredential());
}
```

将密钥环存储位置设置 (例如 [PersistKeysToAzureBlobStorage](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionbuilderextensions.persistkeystoazureblobstorage)) 。 必须设置位置，因为调用 `ProtectKeysWithAzureKeyVault` 实现了禁用自动数据保护设置的 [IXmlEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.xmlencryption.ixmlencryptor) ，包括密钥环存储位置。 前面的示例使用 Azure Blob 存储来持久保存密钥环。 有关详细信息，请参阅 [密钥存储提供程序： Azure 存储](xref:security/data-protection/implementation/key-storage-providers#azure-storage)。 还可以通过 [PersistKeysToFileSystem](xref:security/data-protection/implementation/key-storage-providers#file-system)将密钥环保存到本地。

`keyIdentifier`是用于密钥加密的密钥保管库密钥标识符。 例如，在中名为的密钥保管库中创建的密钥 `dataprotection` `contosokeyvault` 具有密钥标识符 `https://contosokeyvault.vault.azure.net/keys/dataprotection/` 。 向应用提供对密钥保管库的 **解包密钥** 和 **包装密钥** 权限。

`ProtectKeysWithAzureKeyVault` 重载

* [ProtectKeysWithAzureKeyVault (IDataProtectionBuilder、Uri、TokenCredential) ](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionkeyvaultkeybuilderextensions.protectkeyswithazurekeyvault#Microsoft_AspNetCore_DataProtection_AzureDataProtectionKeyVaultKeyBuilderExtensions_ProtectKeysWithAzureKeyVault_Microsoft_AspNetCore_DataProtection_IDataProtectionBuilder_System_Uri_Azure_Core_TokenCredential_) 允许使用 keyIdentifier Uri 和 TokenCredential 来使数据保护系统使用密钥保管库。
* [ProtectKeysWithAzureKeyVault (IDataProtectionBuilder，string，IKeyEncryptionKeyResolver) ](/dotnet/api/microsoft.aspnetcore.dataprotection.azuredataprotectionkeyvaultkeybuilderextensions.protectkeyswithazurekeyvault#Microsoft_AspNetCore_DataProtection_AzureDataProtectionKeyVaultKeyBuilderExtensions_ProtectKeysWithAzureKeyVault_Microsoft_AspNetCore_DataProtection_IDataProtectionBuilder_System_Uri_Azure_Core_TokenCredential_) 允许使用 keyIdentifier String 和 IKeyEncryptionKeyResolver 使数据保护系统能够使用密钥保管库。

如果应用使用以前的 Azure 包 ([`Microsoft.AspNetCore.DataProtection.AzureStorage`](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.AzureStorage) 和 [`Microsoft.AspNetCore.DataProtection.AzureKeyVault`](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.AzureKeyVault)) 并结合了 Azure Key Vault 和 Azure 存储来存储和保护密钥，则 <xref:System.UriFormatException?displayProperty=nameWithType> 会在密钥存储的 blob 不存在时引发。 可以在 Azure 门户中运行应用之前手动创建 blob，也可以使用以下过程：

1. 删除对的调用，以便 `ProtectKeysWithAzureKeyVault` 在首次运行时创建 blob。
1. 将对的调用添加到 `ProtectKeysWithAzureKeyVault` 后面的运行。

`ProtectKeysWithAzureKeyVault`建议删除首次运行，因为它可确保创建的文件具有正确的架构和值。 

建议升级到 [AspNetCore DataProtection](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Blobs) 和 [AspNetCore](https://www.nuget.org/packages/Azure.Extensions.AspNetCore.DataProtection.Keys) 包，因为如果 blob 不存在，则该 API 会自动创建该 blob 的包。

```csharp
services.AddDataProtection()
    //This blob must already exist before the application is run
    .PersistKeysToAzureBlobStorage("<storage account connection string", "<key store container name>", "<key store blob name>")
    //Removing this line below for an initial run will ensure the file is created correctly
    .ProtectKeysWithAzureKeyVault(new Uri("<keyIdentifier>"), new DefaultAzureCredential());
```

::: moniker-end

## <a name="persistkeystofilesystem"></a>PersistKeysToFileSystem

若要将密钥存储在 UNC 共享上，而不是存储在 *% LOCALAPPDATA%* 默认位置，请使用 [PersistKeysToFileSystem](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.persistkeystofilesystem)配置系统：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"));
}
```

> [!WARNING]
> 如果更改密钥持久性位置，系统将不再自动加密静态密钥，因为它不知道 DPAPI 是否为适当的加密机制。

## <a name="protectkeyswith"></a>ProtectKeysWith\*

可以通过调用任何[ProtectKeysWith \* ](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions)配置 api 将系统配置为保护静态密钥。 请考虑以下示例，该示例将密钥存储在 UNC 共享上，并使用特定的 x.509 证书对静态密钥进行加密：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"))
        .ProtectKeysWithCertificate("thumbprint");
}
```

::: moniker range=">= aspnetcore-2.1"

在 ASP.NET Core 2.1 或更高版本中，你可以提供[ProtectKeysWithCertificate](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.protectkeyswithcertificate)的[X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2) ，例如从文件中加载的证书：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"))
        .ProtectKeysWithCertificate(
            new X509Certificate2("certificate.pfx", "password"));
}
```

::: moniker-end

有关内置密钥加密机制的更多示例和讨论，请参阅 [静态密钥加密](xref:security/data-protection/implementation/key-encryption-at-rest) 。

::: moniker range=">= aspnetcore-2.1"

## <a name="unprotectkeyswithanycertificate"></a>UnprotectKeysWithAnyCertificate

在 ASP.NET Core 2.1 或更高版本中，你可以使用[UnprotectKeysWithAnyCertificate](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.unprotectkeyswithanycertificate)的[X509Certificate2](/dotnet/api/system.security.cryptography.x509certificates.x509certificate2)

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\directory\"))
        .ProtectKeysWithCertificate(
            new X509Certificate2("certificate.pfx", "password"));
        .UnprotectKeysWithAnyCertificate(
            new X509Certificate2("certificate_old_1.pfx", "password_1"),
            new X509Certificate2("certificate_old_2.pfx", "password_2"));
}
```

::: moniker-end

## <a name="setdefaultkeylifetime"></a>SetDefaultKeyLifetime

若要将系统配置为使用14天的密钥生存期而不是默认的90天，请使用 [SetDefaultKeyLifetime](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.setdefaultkeylifetime)：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .SetDefaultKeyLifetime(TimeSpan.FromDays(14));
}
```

## <a name="setapplicationname"></a>SetApplicationName

默认情况下，数据保护系统基于其 [内容根](xref:fundamentals/index#content-root) 路径将应用彼此隔离，即使它们共享相同的物理密钥存储库。 这可防止应用了解彼此的受保护负载。

在应用之间共享受保护的负载：

* <xref:Microsoft.AspNetCore.DataProtection.DataProtectionBuilderExtensions.SetApplicationName*>在每个应用中配置相同的值。
* 在应用程序中使用相同版本的数据保护 API 堆栈。 在应用的项目文件中执行以下 **任一** 操作：
  * 通过 [AspNetCore 元包](xref:fundamentals/metapackage-app)引用相同的共享框架版本。
  * 引用相同的 [数据保护包](xref:security/data-protection/introduction#package-layout) 版本。

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .SetApplicationName("shared app name");
}
```

## <a name="disableautomatickeygeneration"></a>DisableAutomaticKeyGeneration

你可能会遇到这样的情况：你不希望应用程序自动滚动更新密钥 (创建新密钥) ，因为它们接近过期。 这种情况的一个示例可能是在主/辅助关系中设置的应用，其中只有主应用负责密钥管理问题，辅助应用只是具有密钥环的只读视图。 可以将辅助应用配置为使用以下配置系统将密钥环视为只读 <xref:Microsoft.AspNetCore.DataProtection.DataProtectionBuilderExtensions.DisableAutomaticKeyGeneration*> ：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDataProtection()
        .DisableAutomaticKeyGeneration();
}
```

## <a name="per-application-isolation"></a>每应用程序隔离

当数据保护系统由 ASP.NET Core 主机提供时，它会自动将应用彼此隔离，即使这些应用在相同的工作进程帐户下运行并且使用相同的主密钥材料也是如此。 这有点类似于 System.web 元素中的 IsolateApps 修饰符 `<machineKey>` 。

隔离机制的工作原理是将本地计算机上的每个应用视为唯一的租户，因此， <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> 任何给定应用的根都将自动包括应用 ID 作为鉴别器。 应用的唯一 ID 是应用的物理路径：

* 对于在 IIS 中托管的应用，唯一 ID 是应用的 IIS 物理路径。 如果在 web 场环境中部署了应用，则此值是稳定的，假定在 web 场中的所有计算机上配置了类似的 IIS 环境。
* 对于在 [Kestrel 服务器](xref:fundamentals/servers/index#kestrel)上运行的自承载应用程序，唯一 ID 是指向磁盘上的应用程序的物理路径。

唯一标识符旨在重置 &mdash; 单独的应用程序和计算机本身。

此隔离机制假定应用不是恶意的。 恶意应用始终会影响在同一工作进程帐户下运行的任何其他应用。 在应用不受信任的共享主机环境中，托管提供商应采取措施来确保应用之间的操作系统级隔离，包括分离应用程序的底层密钥存储库。

如果 ASP.NET Core 主机未提供数据保护系统 (例如，如果通过具体类型对其进行实例化 `DataProtectionProvider`) 则默认情况下禁用应用程序隔离。 禁用应用隔离后，只要提供相应的 [用途](xref:security/data-protection/consumer-apis/purpose-strings)，同一密钥材料支持的所有应用就可以共享有效负载。 若要在此环境中提供应用隔离，请对配置对象调用 [SetApplicationName](#setapplicationname) 方法，并为每个应用提供唯一的名称。

## <a name="changing-algorithms-with-usecryptographicalgorithms"></a>用 UseCryptographicAlgorithms 更改算法

数据保护堆栈允许您更改新生成的密钥使用的默认算法。 执行此操作的最简单方法是从配置回调调用 [UseCryptographicAlgorithms](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.usecryptographicalgorithms) ：

::: moniker range=">= aspnetcore-2.0"

```csharp
services.AddDataProtection()
    .UseCryptographicAlgorithms(
        new AuthenticatedEncryptorConfiguration()
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_CBC,
        ValidationAlgorithm = ValidationAlgorithm.HMACSHA256
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

```csharp
services.AddDataProtection()
    .UseCryptographicAlgorithms(
        new AuthenticatedEncryptionSettings()
    {
        EncryptionAlgorithm = EncryptionAlgorithm.AES_256_CBC,
        ValidationAlgorithm = ValidationAlgorithm.HMACSHA256
    });
```

::: moniker-end

默认 EncryptionAlgorithm 为 AES-256-CBC，默认 ValidationAlgorithm 为 HMACSHA256。 系统管理员可以通过 [计算机范围内的策略](xref:security/data-protection/configuration/machine-wide-policy)来设置默认策略，但会对 `UseCryptographicAlgorithms` 替代默认策略进行显式调用。

通过调用， `UseCryptographicAlgorithms` 可以从预定义的内置列表中指定所需的算法。 您无需担心算法的实现。 在上述方案中，如果在 Windows 上运行，数据保护系统将尝试使用 AES 的 CNG 实现。 否则，它会回退到托管的[系统。](/dotnet/api/system.security.cryptography.aes)

可以通过调用 [UseCustomCryptographicAlgorithms](/dotnet/api/microsoft.aspnetcore.dataprotection.dataprotectionbuilderextensions.usecustomcryptographicalgorithms)手动指定实现。

> [!TIP]
> 更改算法不会影响密钥环中的现有密钥。 它仅影响新生成的键。

### <a name="specifying-custom-managed-algorithms"></a>指定自定义托管算法

::: moniker range=">= aspnetcore-2.0"

若要指定自定义托管算法，请创建一个指向实现类型的 [ManagedAuthenticatedEncryptorConfiguration](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.managedauthenticatedencryptorconfiguration) 实例：

```csharp
serviceCollection.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new ManagedAuthenticatedEncryptorConfiguration()
    {
        // A type that subclasses SymmetricAlgorithm
        EncryptionAlgorithmType = typeof(Aes),

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // A type that subclasses KeyedHashAlgorithm
        ValidationAlgorithmType = typeof(HMACSHA256)
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

若要指定自定义托管算法，请创建一个指向实现类型的 [ManagedAuthenticatedEncryptionSettings](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.managedauthenticatedencryptionsettings) 实例：

```csharp
serviceCollection.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new ManagedAuthenticatedEncryptionSettings()
    {
        // A type that subclasses SymmetricAlgorithm
        EncryptionAlgorithmType = typeof(Aes),

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // A type that subclasses KeyedHashAlgorithm
        ValidationAlgorithmType = typeof(HMACSHA256)
    });
```

::: moniker-end

通常， \* 类型属性必须指向具体的可实例化 (通过 [System.security.cryptography.symmetricalgorithm](/dotnet/api/system.security.cryptography.symmetricalgorithm) 和 [KeyedHashAlgorithm](/dotnet/api/system.security.cryptography.keyedhashalgorithm)的公共无参数的) ctor 实现，尽管系统特别适用于一些值， `typeof(Aes)` 以便于方便。

> [!NOTE]
> System.security.cryptography.symmetricalgorithm 必须具有≥128位的密钥长度和≥64位的块大小，并且必须支持 PKCS #7 填充的 CBC 模式加密。 KeyedHashAlgorithm 的摘要大小必须为 >= 128 位，并且它必须支持长度等于哈希算法摘要长度的键。 KeyedHashAlgorithm 不一定是 HMAC。

### <a name="specifying-custom-windows-cng-algorithms"></a>指定自定义 Windows CNG 算法

::: moniker range=">= aspnetcore-2.0"

若要通过 HMAC 验证使用 CBC 模式加密指定自定义 Windows CNG 算法，请创建包含算法信息的 [CngCbcAuthenticatedEncryptorConfiguration](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.cngcbcauthenticatedencryptorconfiguration) 实例：

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngCbcAuthenticatedEncryptorConfiguration()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // Passed to BCryptOpenAlgorithmProvider
        HashAlgorithm = "SHA256",
        HashAlgorithmProvider = null
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

若要通过 HMAC 验证使用 CBC 模式加密指定自定义 Windows CNG 算法，请创建包含算法信息的 [CngCbcAuthenticatedEncryptionSettings](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.cngcbcauthenticatedencryptionsettings) 实例：

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngCbcAuthenticatedEncryptionSettings()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256,

        // Passed to BCryptOpenAlgorithmProvider
        HashAlgorithm = "SHA256",
        HashAlgorithmProvider = null
    });
```

::: moniker-end

> [!NOTE]
> 对称块加密算法的密钥长度必须为 >= 128 位，块大小为 >= 64 位，并且它必须支持 PKCS #7 填充的 CBC 模式加密。 哈希算法的摘要大小必须为 >= 128 位，并且必须支持使用 BCRYPT \_ ALG \_ 句柄 \_ HMAC \_ 标志标志打开。 \*提供程序属性可以设置为 null，以将默认提供程序用于指定的算法。 有关详细信息，请参阅 [BCryptOpenAlgorithmProvider](/windows/win32/api/bcrypt/nf-bcrypt-bcryptopenalgorithmprovider) 文档。

::: moniker range=">= aspnetcore-2.0"

若要使用 Galois/Counter 模式加密和验证来指定自定义 Windows CNG 算法，请创建包含算法信息的 [CngGcmAuthenticatedEncryptorConfiguration](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.cnggcmauthenticatedencryptorconfiguration) 实例：

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngGcmAuthenticatedEncryptorConfiguration()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256
    });
```

::: moniker-end

::: moniker range="< aspnetcore-2.0"

若要使用 Galois/Counter 模式加密和验证来指定自定义 Windows CNG 算法，请创建包含算法信息的 [CngGcmAuthenticatedEncryptionSettings](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.cnggcmauthenticatedencryptionsettings) 实例：

```csharp
services.AddDataProtection()
    .UseCustomCryptographicAlgorithms(
        new CngGcmAuthenticatedEncryptionSettings()
    {
        // Passed to BCryptOpenAlgorithmProvider
        EncryptionAlgorithm = "AES",
        EncryptionAlgorithmProvider = null,

        // Specified in bits
        EncryptionAlgorithmKeySize = 256
    });
```

::: moniker-end

> [!NOTE]
> 对称块密码算法的密钥长度必须为 >= 128 位，块大小正好为128位，并且必须支持 GCM 加密。 可以将 [EncryptionAlgorithmProvider](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.configurationmodel.cngcbcauthenticatedencryptorconfiguration.encryptionalgorithmprovider) 属性设置为 null，以将默认提供程序用于指定的算法。 有关详细信息，请参阅 [BCryptOpenAlgorithmProvider](/windows/win32/api/bcrypt/nf-bcrypt-bcryptopenalgorithmprovider) 文档。

### <a name="specifying-other-custom-algorithms"></a>指定其他自定义算法

尽管不是作为第一类 API 公开的，但数据保护系统的扩展能力足以允许指定几乎任何类型的算法。 例如，可以将硬件安全模块中包含的所有密钥保留 (HSM) ，并提供核心加密和解密例程的自定义实现。 有关详细信息，请参阅[核心加密扩展性](xref:security/data-protection/extensibility/core-crypto)中的[IAuthenticatedEncryptor](/dotnet/api/microsoft.aspnetcore.dataprotection.authenticatedencryption.iauthenticatedencryptor) 。

## <a name="persisting-keys-when-hosting-in-a-docker-container"></a>在 Docker 容器中托管时保持密钥

在 [Docker](/dotnet/standard/microservices-architecture/container-docker-introduction/) 容器中托管时，应在以下任一项中维护密钥：

* 一个文件夹，它是在容器的生存期之外保留的 Docker 卷，如共享卷或主机装入的卷。
* 外部提供程序，如 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 或 [Redis](https://redis.io/)。

## <a name="persisting-keys-with-redis"></a>通过 Redis 保持密钥

只应使用支持 [Redis 数据暂留](/azure/azure-cache-for-redis/cache-how-to-premium-persistence) 的 Redis 版本来存储密钥。 [Azure Blob 存储](/azure/storage/blobs/storage-blobs-introduction) 是持久性的，可用于存储密钥。 有关详细信息，请参阅[此 GitHub 问题](https://github.com/dotnet/AspNetCore/issues/13476)。

## <a name="additional-resources"></a>其他资源

* <xref:security/data-protection/configuration/non-di-scenarios>
* <xref:security/data-protection/configuration/machine-wide-policy>
* <xref:host-and-deploy/web-farm>
* <xref:security/data-protection/implementation/key-storage-providers>
