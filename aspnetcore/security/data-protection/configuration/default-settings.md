---
title: ASP.NET Core 中的数据保护密钥管理和生存期信息
author: rick-anderson
description: 了解 ASP.NET Core 中的数据保护密钥管理和生存期。
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
uid: security/data-protection/configuration/default-settings
ms.openlocfilehash: 1303c5c2c993f1d20383457666aebfa2a583e938
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053002"
---
# <a name="data-protection-key-management-and-lifetime-in-aspnet-core"></a>ASP.NET Core 中的数据保护密钥管理和生存期信息

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

## <a name="key-management"></a>密钥管理

应用会尝试检测其操作环境并自行处理密钥配置。

1. 如果应用托管在 [Azure 应用](https://azure.microsoft.com/services/app-service/)中，则密钥将保留在 *%HOME%\ASP.NET\DataProtection-Keys* 文件夹中。 此文件夹由网络存储提供支持，并跨托管应用的所有计算机同步。
   * 密钥不是静态保护的。
   * *DataProtection* 文件夹在单个部署槽位中向应用的所有实例提供密钥环。
   * 各部署槽位（例如过渡槽和生成槽）不共享密钥环。 当你在部署槽之间进行交换时（例如，将暂存交换到生产环境或使用 A/B 测试），任何使用数据保护的应用都将无法使用上一个槽内的密钥环来解密存储的数据。 这会导致用户注销使用标准 ASP.NET Core cookie 身份验证的应用，因为它使用数据保护来保护它 cookie 。 如果需要与槽无关的密钥环，请使用外部密钥环形提供程序，例如 Azure Blob 存储、Azure Key Vault、SQL 存储或 Redis 缓存。

1. 如果用户配置文件可用，则密钥将保留在 *%LOCALAPPDATA%\ASP.NET\DataProtection-Keys* 文件夹中。 如果操作系统为 Windows，则使用 DPAPI 对密钥进行静态加密。

   同时还必须启用应用池的 [setProfileEnvironment attribute](/iis/configuration/system.applicationhost/applicationpools/add/processmodel#configuration)。 `setProfileEnvironment` 的默认值为 `true`。 在某些情况下（例如，Windows 操作系统），将 `setProfileEnvironment` 设置为 `false`。 如果密钥未按预期存储在用户配置文件目录中，请执行以下操作：

   1. 导航到 %windir%/system32/inetsrv/config 文件夹。
   1. 打开 applicationHost.config 文件。
   1. 查找 `<system.applicationHost><applicationPools><applicationPoolDefaults><processModel>` 元素。
   1. 确认 `setProfileEnvironment` 属性不存在，这会将值默认设置为 `true`，或者将属性的值显式设置为 `true`。

1. 如果应用托管在 IIS 中，则密钥将保留在仅列入工作进程帐户的特殊注册表项中的 HKLM 注册表中。 使用 DPAPI 对密钥静态加密。

1. 如果这些条件都不匹配，则不会在当前进程的外部保留键。 当进程关闭时，所有生成的密钥都将丢失。

开发人员始终处于完全控制下，并且可以重写存储键的方式和位置。 上述前三个选项应为大多数应用提供良好的默认设置，这与 ASP.NET **\<machineKey>** 自动生成例程过去的工作方式类似。 最终的回退选项是要求开发人员在需要进行密钥持久性的情况下预先指定 [配置](xref:security/data-protection/configuration/overview) 的唯一方案，但只有在极少数情况下才会发生此回退。

在 Docker 容器中托管时，密钥应保留在一个文件夹中，该文件夹 (共享卷或主机装载的卷，该卷会超出容器生存期) 或外部提供程序（如 [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 或 [Redis](https://redis.io/)）。 如果应用无法访问共享的网络卷，则外部提供程序也可用于 web 场方案 (请参阅 [PersistKeysToFileSystem](xref:security/data-protection/configuration/overview#persistkeystofilesystem) 了解详细信息) 。

> [!WARNING]
> 如果开发人员覆盖上述规则，并在特定的密钥存储库中指向数据保护系统，将禁用静态密钥的自动加密。 可以通过 [配置](xref:security/data-protection/configuration/overview)重新启用静态保护。

## <a name="key-lifetime"></a>密钥生存期

默认情况下，密钥的生存期为90天。 密钥过期后，应用会自动生成新密钥，并将新密钥设置为活动密钥。 只要已停用的密钥仍保留在系统中，你的应用就可以对任何受其保护的数据进行解密。 有关详细信息，请参阅 [密钥管理](xref:security/data-protection/implementation/key-management#key-expiration-and-rolling) 。

## <a name="default-algorithms"></a>默认算法

使用的默认负载保护算法为 AES-256-CBC，以实现保密性，使用 HMACSHA256 进行身份验证。 512位主密钥（每90天更改一次）用于根据每个有效负载派生用于这些算法的两个子项。 有关详细信息，请参阅 [子项派生](xref:security/data-protection/implementation/subkeyderivation#additional-authenticated-data-and-subkey-derivation) 。

## <a name="additional-resources"></a>其他资源

* <xref:security/data-protection/extensibility/key-management>
* <xref:host-and-deploy/web-farm>
