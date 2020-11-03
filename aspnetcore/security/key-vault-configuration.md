---
title: ASP.NET Core 中的 Azure Key Vault 配置提供程序
author: rick-anderson
description: 了解如何使用 Azure Key Vault 配置提供程序通过在运行时加载的名称/值对来配置应用。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc, devx-track-azurecli
ms.date: 02/07/2020
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
uid: security/key-vault-configuration
ms.openlocfilehash: 10a949831c180f51bc6bb9b8294150a558f9343c
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93060126"
---
# <a name="azure-key-vault-configuration-provider-in-aspnet-core"></a>ASP.NET Core 中的 Azure Key Vault 配置提供程序

作者： [Andrew Stanton](https://github.com/anurse)

::: moniker range=">= aspnetcore-3.0"

本文档说明如何使用 [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 配置提供程序从 Azure Key Vault 机密加载应用配置值。 Azure Key Vault 是一项基于云的服务，可帮助保护应用程序和服务使用的加密密钥和机密。 将 Azure Key Vault 用于 ASP.NET Core 应用的常见方案包括：

* 控制对敏感配置数据的访问。
* 满足 FIPS 140-2 第2级验证的硬件安全模块在存储配置数据时 (HSM) 的要求。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/key-vault-configuration/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="packages"></a>包

向Microsoft.Extensions.Configu 添加包引用 [ 。AzureKeyVault](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) 包。

## <a name="sample-app"></a>示例应用

该示例应用在由 `#define` *Program.cs* 文件顶部的语句确定的两种模式中运行：

* `Certificate`：演示如何使用 Azure Key Vault 的客户端 ID 和 x.509 证书来访问存储在 Azure Key Vault 中的机密。 可以从部署到 Azure App Service 的任何位置运行此版本的示例，也可以从部署到 ASP.NET Core 应用程序的任何主机上运行。
* `Managed`：演示如何使用 [Azure 资源的托管标识](/azure/active-directory/managed-identities-azure-resources/overview) 对应用进行身份验证，以通过 Azure AD 身份验证 Azure Key Vault，而不会在应用的代码或配置中存储凭据。 使用托管标识进行身份验证时，不需要 Azure AD 的应用程序 ID 和密码 (客户端密钥) 。 `Managed`必须将示例的版本部署到 Azure。 按照 [使用 Azure 资源的托管标识](#use-managed-identities-for-azure-resources) 部分中的指导进行操作。

有关如何使用预处理器指令 () 配置示例应用的详细信息 `#define` ，请参阅 <xref:index#preprocessor-directives-in-sample-code> 。

## <a name="secret-storage-in-the-development-environment"></a>开发环境中的机密存储

使用 [机密管理器工具](xref:security/app-secrets)在本地设置机密。 当在开发环境中的本地计算机上运行示例应用时，会从本地密钥管理器存储中加载密码。

机密管理器工具需要 `<UserSecretsId>` 应用的项目文件中的属性。 将属性值 (`{GUID}`) 设置为任何唯一 GUID：

```xml
<PropertyGroup>
  <UserSecretsId>{GUID}</UserSecretsId>
</PropertyGroup>
```

机密以名称-值对的形式创建。 配置节 (层次结构值) `:` 在 [ASP.NET Core 配置](xref:fundamentals/configuration/index) 项名称中将 (冒号) 作为分隔符。

机密管理器用于从打开的命令 shell 到项目的 [内容根目录](xref:fundamentals/index#content-root)，其中 `{SECRET NAME}` 是名称， `{SECRET VALUE}` 是值：

```dotnetcli
dotnet user-secrets set "{SECRET NAME}" "{SECRET VALUE}"
```

从项目的 [内容根](xref:fundamentals/index#content-root) 在命令外壳中执行以下命令，为示例应用设置机密：

```dotnetcli
dotnet user-secrets set "SecretName" "secret_value_1_dev"
dotnet user-secrets set "Section:SecretName" "secret_value_2_dev"
```

如果这些机密存储在 [生产环境中的机密存储](#secret-storage-in-the-production-environment-with-azure-key-vault) Azure Key Vault 中，并 Azure Key Vault 部分，则 `_dev` 后缀将更改为 `_prod` 。 后缀在应用的输出中提供视觉提示，指示配置值的源。

## <a name="secret-storage-in-the-production-environment-with-azure-key-vault"></a>Azure Key Vault 中的生产环境中的机密存储

本 [快速入门：使用 Azure CLI 主题的 Azure Key Vault 设置和检索机密](/azure/key-vault/quick-create-cli) 的说明，请参阅此处，以创建 Azure Key Vault 和存储示例应用使用的机密。 有关更多详细信息，请参阅主题。

1. 使用 [Azure 门户](https://portal.azure.com/)中的以下方法之一打开 Azure Cloud shell：

   * 选择代码块右上角的“试用”。 在文本框中使用搜索字符串 "Azure CLI"。
   * 在浏览器中打开 Cloud Shell，并提供 " **启动 Cloud Shell** " 按钮。
   * 选择 Azure 门户右上角菜单中的 " **Cloud Shell** " 按钮。

   有关详细信息，请参阅 Azure Cloud Shell [Azure CLI](/cli/azure/) 和 [概述](/azure/cloud-shell/overview)。

1. 如果尚未通过身份验证，请在命令中登录 `az login` 。

1. 使用以下命令创建资源组，其中 `{RESOURCE GROUP NAME}` 是新资源组的资源组名称， `{LOCATION}` 是 Azure 区域 (datacenter) ：

   ```azurecli
   az group create --name "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. 使用以下命令在资源组中创建密钥保管库，其中 `{KEY VAULT NAME}` 是新密钥保管库的名称， `{LOCATION}` 是 Azure 区域 (datacenter) ：

   ```azurecli
   az keyvault create --name {KEY VAULT NAME} --resource-group "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. 在密钥保管库中，以名称-值对的形式创建机密。

   Azure Key Vault 机密名称仅限字母数字字符和短划线。 配置节 (层次结构值) 使用 `--` (两个短划线) 作为分隔符。 冒号（通常用于在 [ASP.NET Core 配置](xref:fundamentals/configuration/index)中的子项中分隔部分）在 key vault 机密名称中不允许使用。 因此，当机密加载到应用的配置中时，将使用两个短划线并为冒号交换。

   以下机密用于示例应用。 这些值包含 `_prod` 后缀，以将其与 `_dev` 从用户机密的开发环境中加载的后缀值区分开来。 将替换 `{KEY VAULT NAME}` 为在上一步中创建的密钥保管库的名称：

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "SecretName" --value "secret_value_1_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "Section--SecretName" --value "secret_value_2_prod"
   ```

## <a name="use-application-id-and-x509-certificate-for-non-azure-hosted-apps"></a>为非 Azure 托管的应用使用应用程序 ID 和 x.509 证书

配置 Azure AD、Azure Key Vault 和应用，以便在 **Azure 外托管应用时** 使用 Azure Active Directory 应用程序 ID 和 x.509 证书对密钥保管库进行身份验证。 有关详细信息，请参阅[关于密钥、机密和证书](/azure/key-vault/about-keys-secrets-and-certificates)。

> [!NOTE]
> 尽管在 Azure 中托管的应用程序支持使用应用程序 ID 和 x.509 证书，但在 Azure 中托管应用程序时，我们建议使用 [Azure 资源的托管标识](#use-managed-identities-for-azure-resources) 。 托管标识不需要在应用或开发环境中存储证书。

当 `#define` *Program.cs* 文件顶部的语句设置为时，示例应用使用应用程序 ID 和 x.509 证书 `Certificate` 。

1. 创建 PKCS # 12 存档 ( *.pfx* ) 证书。 用于创建证书的选项包括 Windows 和[OpenSSL](https://www.openssl.org/)[上的 MakeCert](/windows/desktop/seccrypto/makecert) 。
1. 将证书安装到当前用户的个人证书存储中。 将密钥标记为可导出是可选的。 记下证书的指纹，此过程将在此过程中使用。
1. 将 PKCS # 12 存档 ( *.pfx* ) 证书导 () *.cer* 编码的证书。
1. 将应用注册 Azure AD ( **应用注册** ) 。
1. 将 DER 编码的证书 ( *.cer* ) 上传到 Azure AD：
   1. 在 Azure AD 中选择应用。
   1. 导航到 " **证书" & "机密** "。
   1. 选择 " **上传证书** "，上传包含公钥的证书。 *.Cer* 、 *pem* 或 *.crt* 证书是可接受的。
1. 在应用的文件中存储密钥保管库名称、应用程序 ID 和证书指纹 *appsettings.json* 。
1. 导航到 Azure 门户中的 **密钥保管库** 。
1. 选择在 [生产环境中的机密存储中](#secret-storage-in-the-production-environment-with-azure-key-vault) 创建的密钥保管库，其中 Azure Key Vault "部分。
1. 选择“访问策略”。
1. 选择“添加访问策略”。
1. 打开 **机密权限** ，并为应用提供 **Get** 和 **List** 权限。
1. 选择 " **选择主体** "，并按名称选择注册的应用。 选择“选择”按钮  。
1. 选择“确定”  。
1. 选择“保存”  。
1. 部署应用。

`Certificate`示例应用从中获取其配置值，其 `IConfigurationRoot` 名称与机密名称相同：

* 非分层值：通过获取的值 `SecretName` `config["SecretName"]` 。
* ) 部分 (层次结构值：使用 `:` (冒号) 表示法或 `GetSection` 扩展方法。 使用以下任一方法来获取配置值：
  * `config["Section:SecretName"]`
  * `config.GetSection("Section")["SecretName"]`

X.509 证书由操作系统管理。 应用 <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 使用文件提供的值调用 *appsettings.json* ：

[!code-csharp[](key-vault-configuration/samples/3.x/SampleApp/Program.cs?name=snippet1&highlight=20-23)]

示例值：

* 密钥保管库名称： `contosovault`
* 应用程序 ID： `627e911e-43cc-61d4-992e-12db9c81b413`
* 证书指纹： `fe14593dd66b2406c5269d742d04b6e1ab03adb1`

*appsettings.json* :

[!code-json[](key-vault-configuration/samples/3.x/SampleApp/appsettings.json?highlight=10-12)]

运行应用时，网页将显示加载的机密值。 在开发环境中，机密值用后缀进行加载 `_dev` 。 在生产环境中，值以后缀进行加载 `_prod` 。

## <a name="use-managed-identities-for-azure-resources"></a>使用 Azure 资源的托管标识

**部署到 azure 的应用** 可以利用 [Azure 资源的托管标识](/azure/active-directory/managed-identities-azure-resources/overview)，这允许应用使用 Azure AD 身份验证（不含凭据 (应用程序 ID 和密码/客户端密钥) 存储在应用中）对 Azure Key Vault 进行身份验证。

当 `#define` *Program.cs* 文件顶部的语句设置为时，示例应用使用 Azure 资源的托管标识 `Managed` 。

在应用的文件中输入保管库名称 *appsettings.json* 。 当设置为版本时，示例应用不需要应用程序 ID 和密码 (的客户端机密) `Managed` ，因此你可以忽略这些配置条目。 应用将部署到 Azure，Azure 将对应用进行身份验证，以便仅使用存储在文件中的保管库名称访问 Azure Key Vault *appsettings.json* 。

将示例应用部署到 Azure App Service。

在创建服务时，会自动向 Azure AD 注册部署到 Azure App Service 的应用。 从部署中获取对象 ID 以在下面的命令中使用。 对象 ID 显示在应用服务面板上的 "Azure 门户中 **Identity** 。

使用 Azure CLI 和应用程序的对象 ID，为应用程序提供 `list` `get` 访问密钥保管库的和权限：

```azurecli
az keyvault set-policy --name {KEY VAULT NAME} --object-id {OBJECT ID} --secret-permissions get list
```

使用 Azure CLI、PowerShell 或 Azure 门户 **重启应用** 。

示例应用：

* 创建 `AzureServiceTokenProvider` 不带连接字符串的类的实例。 如果未提供连接字符串，提供程序将尝试从 Azure 资源的托管标识获取访问令牌。
* <xref:Microsoft.Azure.KeyVault.KeyVaultClient>使用 `AzureServiceTokenProvider` 实例标记回调创建新的。
* <xref:Microsoft.Azure.KeyVault.KeyVaultClient>实例与加载所有机密值的的默认实现一起使用， <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> 并将 `--` 冒号 () 替换为 `:` 键名称中 () 。

[!code-csharp[](key-vault-configuration/samples/3.x/SampleApp/Program.cs?name=snippet2&highlight=13-21)]

Key vault 名称示例值： `contosovault`
    
*appsettings.json* :

```json
{
  "KeyVaultName": "Key Vault Name"
}
```

运行应用时，网页将显示加载的机密值。 在开发环境中，机密值具有 `_dev` 后缀，因为它们是由用户机密提供的。 在生产环境中，值加载时使用 `_prod` 后缀，因为它们由 Azure Key Vault 提供。

如果收到 `Access denied` 错误，请确认该应用已注册到 Azure AD，并提供对密钥保管库的访问权限。 确认已在 Azure 中重新启动该服务。

有关将提供程序与托管标识和 Azure DevOps 管道一起使用的信息，请参阅使用 [托管服务标识创建与 VM 的 Azure 资源管理器服务连接](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity)。

## <a name="configuration-options"></a>配置选项

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 可以接受 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.AzureKeyVaultConfigurationOptions> ：

```csharp
config.AddAzureKeyVault(
    new AzureKeyVaultConfigurationOptions()
    {
        ...
    });
```

| 属性         | 说明 |
| ---------------- | ----------- |
| `Client`         | <xref:Microsoft.Azure.KeyVault.KeyVaultClient> 用于检索值。 |
| `Manager`        | <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> 用于控制机密加载的实例。 |
| `ReloadInterval` | `Timespan` 如果为，则在轮询密钥保管库以进行更改之间等待。 默认值为 `null` (不) 重新加载配置。 |
| `Vault`          | 密钥保管库 URI。 |

## <a name="use-a-key-name-prefix"></a>使用密钥名称前缀

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 提供一个重载，该重载接受的实现 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> ，该实现允许您控制密钥保管库机密如何转换为配置密钥。 例如，你可以实现接口，以便基于在应用启动时提供的前缀值加载机密值。 例如，你可以根据应用程序的版本加载机密。

> [!WARNING]
> 不要对 key vault 机密使用前缀，将多个应用的机密放入同一密钥保管库，或放置环境机密 (例如， *开发* 与 *生产* 机密) 到同一保管库中。 建议不同的应用和开发/生产环境使用不同的密钥保管库，以便隔离应用环境以获得最高级别的安全性。

在下面的示例中，密钥保管库中建立了机密 (和使用开发环境的机密管理器工具) 对于 `5000-AppSecret` 密钥保管库机密名称) 中不允许使用 (期间。 此机密代表应用程序版本5.0.0.0 的应用程序机密。 对于其他版本的应用，5.1.0.0 会将机密添加到密钥保管库 (并使用机密管理器工具) `5100-AppSecret` 。 每个应用版本会将其版本控制的机密值加载到其配置中，并在 `AppSecret` 加载机密时去除版本。

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 使用自定义调用 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> ：

[!code-csharp[](key-vault-configuration/samples_snapshot/Program.cs)]

该 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> 实现将对机密的版本前缀做出反应，以将适当的机密加载到配置中：

* `Load` 当机密名称以前缀开头时，加载密码。 未加载其他机密。
* `GetKey`:
  * 从机密名称中删除前缀。
  * 将任何名称中的两个短划线替换为 `KeyDelimiter` ，它是在配置 (中使用的分隔符，通常为冒号) 。 Azure Key Vault 不允许在机密名称中使用冒号。

[!code-csharp[](key-vault-configuration/samples_snapshot/Startup.cs)]

`Load`方法由提供程序算法调用，该算法循环访问保管库机密以查找具有版本前缀的文件。 如果找到了版本前缀 `Load` ，则该算法将使用 `GetKey` 方法来返回密钥名称的配置名称。 它从机密名称中去除版本前缀，并返回密钥名称的其余部分，以便加载到应用的配置名称-值对中。

实现此方法时：

1. 应用的项目文件中指定的应用版本。 在以下示例中，应用的版本设置为 `5.0.0.0` ：

   ```xml
   <PropertyGroup>
     <Version>5.0.0.0</Version>
   </PropertyGroup>
   ```

1. 确认 `<UserSecretsId>` 应用的项目文件中存在属性，其中 `{GUID}` 是用户提供的 GUID：

   ```xml
   <PropertyGroup>
     <UserSecretsId>{GUID}</UserSecretsId>
   </PropertyGroup>
   ```

   用 [机密管理器工具](xref:security/app-secrets)本地保存以下机密：

   ```dotnetcli
   dotnet user-secrets set "5000-AppSecret" "5.0.0.0_secret_value_dev"
   dotnet user-secrets set "5100-AppSecret" "5.1.0.0_secret_value_dev"
   ```

1. 使用以下 Azure CLI 命令在 Azure Key Vault 中保存机密：

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5000-AppSecret" --value "5.0.0.0_secret_value_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5100-AppSecret" --value "5.1.0.0_secret_value_prod"
   ```

1. 当应用运行时，将加载密钥保管库机密。 的字符串机密与 `5000-AppSecret` 应用程序的项目文件中指定的应用版本匹配 (`5.0.0.0`) 。

1. `5000`用破折号)  (的版本将从键名称中去除。 在整个应用程序中，通过密钥读取配置会 `AppSecret` 加载机密值。

1. 如果在项目文件中将应用程序的版本更改为 `5.1.0.0` ，并再次运行应用程序，则返回的机密值位于 `5.1.0.0_secret_value_dev` 开发环境和生产环境中 `5.1.0.0_secret_value_prod` 。

> [!NOTE]
> 你还可以向提供自己的 <xref:Microsoft.Azure.KeyVault.KeyVaultClient> 实现 <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 。 自定义客户端允许跨应用共享客户端的单个实例。

## <a name="bind-an-array-to-a-class"></a>将数组绑定至类

提供程序能够将配置值读入数组，以便绑定到 POCO 数组。

当从允许键包含冒号 () 分隔符的配置源中进行读取时 `:` ，将使用数字键段来区分组成数组 (`:0:` 、) 的键 `:1:` &hellip; `:{n}:` 。 有关详细信息，请参阅 [配置：将数组绑定到类](xref:fundamentals/configuration/index#bind-an-array-to-a-class)。

Azure Key Vault 密钥不能使用冒号作为分隔符。 本主题中所述的方法使用双短划线 (`--`) 作为层次结构值 (节) 的分隔符。 数组键存储在 Azure Key Vault 中 `--0--` ， (，，) 为双短划线和数值段 `--1--` &hellip; `--{n}--` 。

检查 JSON 文件提供的以下 [Serilog](https://serilog.net/) 日志记录提供程序配置。 在数组中定义了两个对象文本 `WriteTo` ，它们反映了两个 Serilog *接收器* ，它们描述了日志记录输出的目标：

```json
"Serilog": {
  "WriteTo": [
    {
      "Name": "AzureTableStorage",
      "Args": {
        "storageTableName": "logs",
        "connectionString": "DefaultEnd...ountKey=Eby8...GMGw=="
      }
    },
    {
      "Name": "AzureDocumentDB",
      "Args": {
        "endpointUrl": "https://contoso.documents.azure.com:443",
        "authorizationKey": "Eby8...GMGw=="
      }
    }
  ]
}
```

前面的 JSON 文件中所示的配置将存储在 Azure Key Vault 中，使用双划线 (`--`) 表示法和数值段：

| 密钥 | 值 |
| --- | ----- |
| `Serilog--WriteTo--0--Name` | `AzureTableStorage` |
| `Serilog--WriteTo--0--Args--storageTableName` | `logs` |
| `Serilog--WriteTo--0--Args--connectionString` | `DefaultEnd...ountKey=Eby8...GMGw==` |
| `Serilog--WriteTo--1--Name` | `AzureDocumentDB` |
| `Serilog--WriteTo--1--Args--endpointUrl` | `https://contoso.documents.azure.com:443` |
| `Serilog--WriteTo--1--Args--authorizationKey` | `Eby8...GMGw==` |

## <a name="reload-secrets"></a>重载机密

在调用之前会缓存机密 `IConfigurationRoot.Reload()` 。 在执行之前，应用程序不会考虑到密钥保管库中已过期、已禁用和更新的机密 `Reload` 。

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a>禁用和过期的机密

禁用和过期的机密会引发 <xref:Microsoft.Azure.KeyVault.Models.KeyVaultErrorException> 。 若要防止应用程序引发，请使用不同的配置提供程序提供配置，或更新禁用或过期的机密。

## <a name="troubleshoot"></a>疑难解答

当应用无法使用访问接口加载配置时，会将错误消息写入 [ASP.NET Core 日志记录基础结构](xref:fundamentals/logging/index)。 以下条件将阻止加载配置：

* Azure Active Directory 中未正确配置应用或证书。
* Azure Key Vault 中不存在密钥保管库。
* 应用无权访问密钥保管库。
* 访问策略不包括 `Get` 和 `List` 权限。
* 在密钥保管库中，配置数据 (名称-值对) 错误地命名、缺失、禁用或过期。
* 应用具有错误的密钥保管库名称 (`KeyVaultName`) 、Azure AD 应用程序 Id (`AzureADApplicationId`) 或 Azure AD 证书指纹 (`AzureADCertThumbprint`) 。
* 要尝试加载的值在应用中 (名称) 的配置密钥不正确。
* 向密钥保管库添加应用的访问策略时，策略已创建，但未在 **访问策略** UI 中选择 " **保存** " 按钮。

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/configuration/index>
* [Microsoft Azure： Key Vault](https://azure.microsoft.com/services/key-vault/)
* [Microsoft Azure： Key Vault 文档](/azure/key-vault/)
* [如何为 Azure Key Vault 生成和传输受 HSM 保护的密钥](/azure/key-vault/key-vault-hsm-protected-keys)
* [KeyVaultClient 类](/dotnet/api/microsoft.azure.keyvault.keyvaultclient)
* [快速入门：使用 .NET Web 应用在 Azure Key Vault 中设置和检索机密](/azure/key-vault/quick-create-net)
* [教程：如何将 Azure Key Vault 与通过 .NET 编写的 Azure Windows 虚拟机配合使用](/azure/key-vault/tutorial-net-windows-virtual-machine)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

本文档说明如何使用 [Microsoft Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 配置提供程序从 Azure Key Vault 机密加载应用配置值。 Azure Key Vault 是一项基于云的服务，可帮助保护应用程序和服务使用的加密密钥和机密。 将 Azure Key Vault 用于 ASP.NET Core 应用的常见方案包括：

* 控制对敏感配置数据的访问。
* 满足 FIPS 140-2 第2级验证的硬件安全模块在存储配置数据时 (HSM) 的要求。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/key-vault-configuration/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="packages"></a>包

向Microsoft.Extensions.Configu 添加包引用 [ 。AzureKeyVault](https://www.nuget.org/packages/Microsoft.Extensions.Configuration.AzureKeyVault/) 包。

## <a name="sample-app"></a>示例应用

该示例应用在由 `#define` *Program.cs* 文件顶部的语句确定的两种模式中运行：

* `Certificate`：演示如何使用 Azure Key Vault 的客户端 ID 和 x.509 证书来访问存储在 Azure Key Vault 中的机密。 可以从部署到 Azure App Service 的任何位置运行此版本的示例，也可以从部署到 ASP.NET Core 应用程序的任何主机上运行。
* `Managed`：演示如何使用 [Azure 资源的托管标识](/azure/active-directory/managed-identities-azure-resources/overview) 对应用进行身份验证，以通过 Azure AD 身份验证 Azure Key Vault，而不会在应用的代码或配置中存储凭据。 使用托管标识进行身份验证时，不需要 Azure AD 的应用程序 ID 和密码 (客户端密钥) 。 `Managed`必须将示例的版本部署到 Azure。 按照 [使用 Azure 资源的托管标识](#use-managed-identities-for-azure-resources) 部分中的指导进行操作。

有关如何使用预处理器指令 () 配置示例应用的详细信息 `#define` ，请参阅 <xref:index#preprocessor-directives-in-sample-code> 。

## <a name="secret-storage-in-the-development-environment"></a>开发环境中的机密存储

使用 [机密管理器工具](xref:security/app-secrets)在本地设置机密。 当在开发环境中的本地计算机上运行示例应用时，会从本地密钥管理器存储中加载密码。

机密管理器工具需要 `<UserSecretsId>` 应用的项目文件中的属性。 将属性值 (`{GUID}`) 设置为任何唯一 GUID：

```xml
<PropertyGroup>
  <UserSecretsId>{GUID}</UserSecretsId>
</PropertyGroup>
```

机密以名称-值对的形式创建。 配置节 (层次结构值) `:` 在 [ASP.NET Core 配置](xref:fundamentals/configuration/index) 项名称中将 (冒号) 作为分隔符。

机密管理器用于从打开的命令 shell 到项目的 [内容根目录](xref:fundamentals/index#content-root)，其中 `{SECRET NAME}` 是名称， `{SECRET VALUE}` 是值：

```dotnetcli
dotnet user-secrets set "{SECRET NAME}" "{SECRET VALUE}"
```

从项目的 [内容根](xref:fundamentals/index#content-root) 在命令外壳中执行以下命令，为示例应用设置机密：

```dotnetcli
dotnet user-secrets set "SecretName" "secret_value_1_dev"
dotnet user-secrets set "Section:SecretName" "secret_value_2_dev"
```

如果这些机密存储在 [生产环境中的机密存储](#secret-storage-in-the-production-environment-with-azure-key-vault) Azure Key Vault 中，并 Azure Key Vault 部分，则 `_dev` 后缀将更改为 `_prod` 。 后缀在应用的输出中提供视觉提示，指示配置值的源。

## <a name="secret-storage-in-the-production-environment-with-azure-key-vault"></a>Azure Key Vault 中的生产环境中的机密存储

本 [快速入门：使用 Azure CLI 主题的 Azure Key Vault 设置和检索机密](/azure/key-vault/quick-create-cli) 的说明，请参阅此处，以创建 Azure Key Vault 和存储示例应用使用的机密。 有关更多详细信息，请参阅主题。

1. 使用 [Azure 门户](https://portal.azure.com/)中的以下方法之一打开 Azure Cloud shell：

   * 选择代码块右上角的“试用”。 在文本框中使用搜索字符串 "Azure CLI"。
   * 在浏览器中打开 Cloud Shell，并提供 " **启动 Cloud Shell** " 按钮。
   * 选择 Azure 门户右上角菜单中的 " **Cloud Shell** " 按钮。

   有关详细信息，请参阅 Azure Cloud Shell [Azure CLI](/cli/azure/) 和 [概述](/azure/cloud-shell/overview)。

1. 如果尚未通过身份验证，请在命令中登录 `az login` 。

1. 使用以下命令创建资源组，其中 `{RESOURCE GROUP NAME}` 是新资源组的资源组名称， `{LOCATION}` 是 Azure 区域 (datacenter) ：

   ```azurecli
   az group create --name "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. 使用以下命令在资源组中创建密钥保管库，其中 `{KEY VAULT NAME}` 是新密钥保管库的名称， `{LOCATION}` 是 Azure 区域 (datacenter) ：

   ```azurecli
   az keyvault create --name {KEY VAULT NAME} --resource-group "{RESOURCE GROUP NAME}" --location {LOCATION}
   ```

1. 在密钥保管库中，以名称-值对的形式创建机密。

   Azure Key Vault 机密名称仅限字母数字字符和短划线。 配置节 (层次结构值) 使用 `--` (两个短划线) 作为分隔符。 冒号（通常用于在 [ASP.NET Core 配置](xref:fundamentals/configuration/index)中的子项中分隔部分）在 key vault 机密名称中不允许使用。 因此，当机密加载到应用的配置中时，将使用两个短划线并为冒号交换。

   以下机密用于示例应用。 这些值包含 `_prod` 后缀，以将其与 `_dev` 从用户机密的开发环境中加载的后缀值区分开来。 将替换 `{KEY VAULT NAME}` 为在上一步中创建的密钥保管库的名称：

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "SecretName" --value "secret_value_1_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "Section--SecretName" --value "secret_value_2_prod"
   ```

## <a name="use-application-id-and-x509-certificate-for-non-azure-hosted-apps"></a>为非 Azure 托管的应用使用应用程序 ID 和 x.509 证书

配置 Azure AD、Azure Key Vault 和应用，以便在 **Azure 外托管应用时** 使用 Azure Active Directory 应用程序 ID 和 x.509 证书对密钥保管库进行身份验证。 有关详细信息，请参阅[关于密钥、机密和证书](/azure/key-vault/about-keys-secrets-and-certificates)。

> [!NOTE]
> 尽管在 Azure 中托管的应用程序支持使用应用程序 ID 和 x.509 证书，但在 Azure 中托管应用程序时，我们建议使用 [Azure 资源的托管标识](#use-managed-identities-for-azure-resources) 。 托管标识不需要在应用或开发环境中存储证书。

当 `#define` *Program.cs* 文件顶部的语句设置为时，示例应用使用应用程序 ID 和 x.509 证书 `Certificate` 。

1. 创建 PKCS # 12 存档 ( *.pfx* ) 证书。 用于创建证书的选项包括 Windows 和[OpenSSL](https://www.openssl.org/)[上的 MakeCert](/windows/desktop/seccrypto/makecert) 。
1. 将证书安装到当前用户的个人证书存储中。 将密钥标记为可导出是可选的。 记下证书的指纹，此过程将在此过程中使用。
1. 将 PKCS # 12 存档 ( *.pfx* ) 证书导 () *.cer* 编码的证书。
1. 将应用注册 Azure AD ( **应用注册** ) 。
1. 将 DER 编码的证书 ( *.cer* ) 上传到 Azure AD：
   1. 在 Azure AD 中选择应用。
   1. 导航到 " **证书" & "机密** "。
   1. 选择 " **上传证书** "，上传包含公钥的证书。 *.Cer* 、 *pem* 或 *.crt* 证书是可接受的。
1. 在应用的文件中存储密钥保管库名称、应用程序 ID 和证书指纹 *appsettings.json* 。
1. 导航到 Azure 门户中的 **密钥保管库** 。
1. 选择在 [生产环境中的机密存储中](#secret-storage-in-the-production-environment-with-azure-key-vault) 创建的密钥保管库，其中 Azure Key Vault "部分。
1. 选择“访问策略”。
1. 选择“添加访问策略”。
1. 打开 **机密权限** ，并为应用提供 **Get** 和 **List** 权限。
1. 选择 " **选择主体** "，并按名称选择注册的应用。 选择“选择”按钮  。
1. 选择“确定”  。
1. 选择“保存”  。
1. 部署应用。

`Certificate`示例应用从中获取其配置值，其 `IConfigurationRoot` 名称与机密名称相同：

* 非分层值：通过获取的值 `SecretName` `config["SecretName"]` 。
* ) 部分 (层次结构值：使用 `:` (冒号) 表示法或 `GetSection` 扩展方法。 使用以下任一方法来获取配置值：
  * `config["Section:SecretName"]`
  * `config.GetSection("Section")["SecretName"]`

X.509 证书由操作系统管理。 应用 <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 使用文件提供的值调用 *appsettings.json* ：

[!code-csharp[](key-vault-configuration/samples/2.x/SampleApp/Program.cs?name=snippet1&highlight=20-23)]

示例值：

* 密钥保管库名称： `contosovault`
* 应用程序 ID： `627e911e-43cc-61d4-992e-12db9c81b413`
* 证书指纹： `fe14593dd66b2406c5269d742d04b6e1ab03adb1`

*appsettings.json* :

[!code-json[](key-vault-configuration/samples/2.x/SampleApp/appsettings.json?highlight=10-12)]

运行应用时，网页将显示加载的机密值。 在开发环境中，机密值用后缀进行加载 `_dev` 。 在生产环境中，值以后缀进行加载 `_prod` 。

## <a name="use-managed-identities-for-azure-resources"></a>使用 Azure 资源的托管标识

**部署到 azure 的应用** 可以利用 [Azure 资源的托管标识](/azure/active-directory/managed-identities-azure-resources/overview)，这允许应用使用 Azure AD 身份验证（不含凭据 (应用程序 ID 和密码/客户端密钥) 存储在应用中）对 Azure Key Vault 进行身份验证。

当 `#define` *Program.cs* 文件顶部的语句设置为时，示例应用使用 Azure 资源的托管标识 `Managed` 。

在应用的文件中输入保管库名称 *appsettings.json* 。 当设置为版本时，示例应用不需要应用程序 ID 和密码 (的客户端机密) `Managed` ，因此你可以忽略这些配置条目。 应用将部署到 Azure，Azure 将对应用进行身份验证，以便仅使用存储在文件中的保管库名称访问 Azure Key Vault *appsettings.json* 。

将示例应用部署到 Azure App Service。

在创建服务时，会自动向 Azure AD 注册部署到 Azure App Service 的应用。 从部署中获取对象 ID 以在下面的命令中使用。 对象 ID 显示在应用服务面板上的 "Azure 门户中 **Identity** 。

使用 Azure CLI 和应用程序的对象 ID，为应用程序提供 `list` `get` 访问密钥保管库的和权限：

```azurecli
az keyvault set-policy --name {KEY VAULT NAME} --object-id {OBJECT ID} --secret-permissions get list
```

使用 Azure CLI、PowerShell 或 Azure 门户 **重启应用** 。

示例应用：

* 创建 `AzureServiceTokenProvider` 不带连接字符串的类的实例。 如果未提供连接字符串，提供程序将尝试从 Azure 资源的托管标识获取访问令牌。
* <xref:Microsoft.Azure.KeyVault.KeyVaultClient>使用 `AzureServiceTokenProvider` 实例标记回调创建新的。
* <xref:Microsoft.Azure.KeyVault.KeyVaultClient>实例与加载所有机密值的的默认实现一起使用， <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> 并将 `--` 冒号 () 替换为 `:` 键名称中 () 。

[!code-csharp[](key-vault-configuration/samples/2.x/SampleApp/Program.cs?name=snippet2&highlight=13-21)]

Key vault 名称示例值： `contosovault`
    
*appsettings.json* :

```json
{
  "KeyVaultName": "Key Vault Name"
}
```

运行应用时，网页将显示加载的机密值。 在开发环境中，机密值具有 `_dev` 后缀，因为它们是由用户机密提供的。 在生产环境中，值加载时使用 `_prod` 后缀，因为它们由 Azure Key Vault 提供。

如果收到 `Access denied` 错误，请确认该应用已注册到 Azure AD，并提供对密钥保管库的访问权限。 确认已在 Azure 中重新启动该服务。

有关将提供程序与托管标识和 Azure DevOps 管道一起使用的信息，请参阅使用 [托管服务标识创建与 VM 的 Azure 资源管理器服务连接](/azure/devops/pipelines/library/connect-to-azure#create-an-azure-resource-manager-service-connection-to-a-vm-with-a-managed-service-identity)。

## <a name="use-a-key-name-prefix"></a>使用密钥名称前缀

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 提供一个重载，该重载接受的实现 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> ，该实现允许您控制密钥保管库机密如何转换为配置密钥。 例如，你可以实现接口，以便基于在应用启动时提供的前缀值加载机密值。 例如，你可以根据应用程序的版本加载机密。

> [!WARNING]
> 不要对 key vault 机密使用前缀，将多个应用的机密放入同一密钥保管库，或放置环境机密 (例如， *开发* 与 *生产* 机密) 到同一保管库中。 建议不同的应用和开发/生产环境使用不同的密钥保管库，以便隔离应用环境以获得最高级别的安全性。

在下面的示例中，密钥保管库中建立了机密 (和使用开发环境的机密管理器工具) 对于 `5000-AppSecret` 密钥保管库机密名称) 中不允许使用 (期间。 此机密代表应用程序版本5.0.0.0 的应用程序机密。 对于其他版本的应用，5.1.0.0 会将机密添加到密钥保管库 (并使用机密管理器工具) `5100-AppSecret` 。 每个应用版本会将其版本控制的机密值加载到其配置中，并在 `AppSecret` 加载机密时去除版本。

<xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 使用自定义调用 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> ：

[!code-csharp[](key-vault-configuration/samples_snapshot/Program.cs)]

该 <xref:Microsoft.Extensions.Configuration.AzureKeyVault.IKeyVaultSecretManager> 实现将对机密的版本前缀做出反应，以将适当的机密加载到配置中：

* `Load` 当机密名称以前缀开头时，加载密码。 未加载其他机密。
* `GetKey`:
  * 从机密名称中删除前缀。
  * 将任何名称中的两个短划线替换为 `KeyDelimiter` ，它是在配置 (中使用的分隔符，通常为冒号) 。 Azure Key Vault 不允许在机密名称中使用冒号。

[!code-csharp[](key-vault-configuration/samples_snapshot/Startup.cs)]

`Load`方法由提供程序算法调用，该算法循环访问保管库机密以查找具有版本前缀的文件。 如果找到了版本前缀 `Load` ，则该算法将使用 `GetKey` 方法来返回密钥名称的配置名称。 它从机密名称中去除版本前缀，并返回密钥名称的其余部分，以便加载到应用的配置名称-值对中。

实现此方法时：

1. 应用的项目文件中指定的应用版本。 在以下示例中，应用的版本设置为 `5.0.0.0` ：

   ```xml
   <PropertyGroup>
     <Version>5.0.0.0</Version>
   </PropertyGroup>
   ```

1. 确认 `<UserSecretsId>` 应用的项目文件中存在属性，其中 `{GUID}` 是用户提供的 GUID：

   ```xml
   <PropertyGroup>
     <UserSecretsId>{GUID}</UserSecretsId>
   </PropertyGroup>
   ```

   用 [机密管理器工具](xref:security/app-secrets)本地保存以下机密：

   ```dotnetcli
   dotnet user-secrets set "5000-AppSecret" "5.0.0.0_secret_value_dev"
   dotnet user-secrets set "5100-AppSecret" "5.1.0.0_secret_value_dev"
   ```

1. 使用以下 Azure CLI 命令在 Azure Key Vault 中保存机密：

   ```azurecli
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5000-AppSecret" --value "5.0.0.0_secret_value_prod"
   az keyvault secret set --vault-name {KEY VAULT NAME} --name "5100-AppSecret" --value "5.1.0.0_secret_value_prod"
   ```

1. 当应用运行时，将加载密钥保管库机密。 的字符串机密与 `5000-AppSecret` 应用程序的项目文件中指定的应用版本匹配 (`5.0.0.0`) 。

1. `5000`用破折号)  (的版本将从键名称中去除。 在整个应用程序中，通过密钥读取配置会 `AppSecret` 加载机密值。

1. 如果在项目文件中将应用程序的版本更改为 `5.1.0.0` ，并再次运行应用程序，则返回的机密值位于 `5.1.0.0_secret_value_dev` 开发环境和生产环境中 `5.1.0.0_secret_value_prod` 。

> [!NOTE]
> 你还可以向提供自己的 <xref:Microsoft.Azure.KeyVault.KeyVaultClient> 实现 <xref:Microsoft.Extensions.Configuration.AzureKeyVaultConfigurationExtensions.AddAzureKeyVault*> 。 自定义客户端允许跨应用共享客户端的单个实例。

## <a name="bind-an-array-to-a-class"></a>将数组绑定至类

提供程序能够将配置值读入数组，以便绑定到 POCO 数组。

当从允许键包含冒号 () 分隔符的配置源中进行读取时 `:` ，将使用数字键段来区分组成数组 (`:0:` 、) 的键 `:1:` &hellip; `:{n}:` 。 有关详细信息，请参阅 [配置：将数组绑定到类](xref:fundamentals/configuration/index#bind-an-array-to-a-class)。

Azure Key Vault 密钥不能使用冒号作为分隔符。 本主题中所述的方法使用双短划线 (`--`) 作为层次结构值 (节) 的分隔符。 数组键存储在 Azure Key Vault 中 `--0--` ， (，，) 为双短划线和数值段 `--1--` &hellip; `--{n}--` 。

检查 JSON 文件提供的以下 [Serilog](https://serilog.net/) 日志记录提供程序配置。 在数组中定义了两个对象文本 `WriteTo` ，它们反映了两个 Serilog *接收器* ，它们描述了日志记录输出的目标：

```json
"Serilog": {
  "WriteTo": [
    {
      "Name": "AzureTableStorage",
      "Args": {
        "storageTableName": "logs",
        "connectionString": "DefaultEnd...ountKey=Eby8...GMGw=="
      }
    },
    {
      "Name": "AzureDocumentDB",
      "Args": {
        "endpointUrl": "https://contoso.documents.azure.com:443",
        "authorizationKey": "Eby8...GMGw=="
      }
    }
  ]
}
```

前面的 JSON 文件中所示的配置将存储在 Azure Key Vault 中，使用双划线 (`--`) 表示法和数值段：

| 密钥 | 值 |
| --- | ----- |
| `Serilog--WriteTo--0--Name` | `AzureTableStorage` |
| `Serilog--WriteTo--0--Args--storageTableName` | `logs` |
| `Serilog--WriteTo--0--Args--connectionString` | `DefaultEnd...ountKey=Eby8...GMGw==` |
| `Serilog--WriteTo--1--Name` | `AzureDocumentDB` |
| `Serilog--WriteTo--1--Args--endpointUrl` | `https://contoso.documents.azure.com:443` |
| `Serilog--WriteTo--1--Args--authorizationKey` | `Eby8...GMGw==` |

## <a name="reload-secrets"></a>重载机密

在调用之前会缓存机密 `IConfigurationRoot.Reload()` 。 在执行之前，应用程序不会考虑到密钥保管库中已过期、已禁用和更新的机密 `Reload` 。

```csharp
Configuration.Reload();
```

## <a name="disabled-and-expired-secrets"></a>禁用和过期的机密

禁用和过期的机密会引发 <xref:Microsoft.Azure.KeyVault.Models.KeyVaultErrorException> 。 若要防止应用程序引发，请使用不同的配置提供程序提供配置，或更新禁用或过期的机密。

## <a name="troubleshoot"></a>疑难解答

当应用无法使用访问接口加载配置时，会将错误消息写入 [ASP.NET Core 日志记录基础结构](xref:fundamentals/logging/index)。 以下条件将阻止加载配置：

* Azure Active Directory 中未正确配置应用或证书。
* Azure Key Vault 中不存在密钥保管库。
* 应用无权访问密钥保管库。
* 访问策略不包括 `Get` 和 `List` 权限。
* 在密钥保管库中，配置数据 (名称-值对) 错误地命名、缺失、禁用或过期。
* 应用具有错误的密钥保管库名称 (`KeyVaultName`) 、Azure AD 应用程序 Id (`AzureADApplicationId`) 或 Azure AD 证书指纹 (`AzureADCertThumbprint`) 。
* 要尝试加载的值在应用中 (名称) 的配置密钥不正确。
* 向密钥保管库添加应用的访问策略时，策略已创建，但未在 **访问策略** UI 中选择 " **保存** " 按钮。

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/configuration/index>
* [Microsoft Azure： Key Vault](https://azure.microsoft.com/services/key-vault/)
* [Microsoft Azure： Key Vault 文档](/azure/key-vault/)
* [如何为 Azure Key Vault 生成和传输受 HSM 保护的密钥](/azure/key-vault/key-vault-hsm-protected-keys)
* [KeyVaultClient 类](/dotnet/api/microsoft.azure.keyvault.keyvaultclient)
* [快速入门：使用 .NET Web 应用在 Azure Key Vault 中设置和检索机密](/azure/key-vault/quick-create-net)
* [教程：如何将 Azure Key Vault 与通过 .NET 编写的 Azure Windows 虚拟机配合使用](/azure/key-vault/tutorial-net-windows-virtual-machine)

::: moniker-end

