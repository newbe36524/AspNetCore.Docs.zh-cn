---
title: ASP.NET Core 数据保护
author: rick-anderson
description: 了解数据保护的概念以及 ASP.NET Core 数据保护 Api 的设计原则。
ms.author: riande
ms.custom: mvc
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
uid: security/data-protection/introduction
ms.openlocfilehash: 5fd5676b286e758f0648d78bf8cb4171e7a98f60
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93051689"
---
# <a name="aspnet-core-data-protection"></a>ASP.NET Core 数据保护

Web 应用程序通常需要存储安全敏感数据。 Windows 提供了 DPAPI 用于桌面应用程序，但这不适用于 web 应用程序。 ASP.NET Core 数据保护堆栈提供简单易用的加密 API，开发人员可以使用它来保护数据，包括密钥管理和旋转。

ASP.NET Core 的数据保护堆栈旨在充当 &lt; ASP.NET 1.x-4.x 中 machineKey 元素的长期替换项 &gt; 。 它旨在解决旧加密堆栈的许多缺点，同时为大多数用例提供现成的解决方案，最新的应用程序可能会遇到这种情况。

## <a name="problem-statement"></a>问题陈述

整体问题声明可在一个句子中简单地表述：我需要保存可信信息供以后检索，但不信任持久性机制。 在 web 术语中，这可能被编写为 "我需要通过不受信任的客户端往返的受信任状态。"

此规范的典型示例是身份验证 cookie 令牌或持有者令牌。 服务器生成一个 "I Groot" 和 "xyz 权限" 令牌并将其交给客户端。 在将来的某个日期，客户端会将该令牌提供给服务器，但服务器需要某种形式的保证，那就是客户端未伪造令牌。 因此，第一个要求是 (也称为 完整性、防篡改) 。

由于持久状态受服务器信任，因此我们预计此状态可能包含特定于操作环境的信息。 这可能是文件路径、权限、句柄或其他间接引用的形式，也可能是其他特定于服务器的数据的形式。 此类信息通常不会泄露给不受信任的客户端。 因此，第二个要求：保密性。

最后，由于新式应用程序是组件化的，我们看到的是，单个组件需要利用此系统，而不考虑系统中的其他组件。 例如，如果持有者令牌组件正在使用此堆栈，则它应在不干扰可能使用同一堆栈的 CSRF 机制的情况下运行。 因此，最终要求：隔离。

我们可以提供进一步的约束来缩小需求的范围。 假设在 cryptosystem 内运行的所有服务都是同等信任的，并且无需在我们的直接控制下生成或使用服务外部的数据。 此外，我们还需要尽可能快地执行操作，因为每次向 web 服务发出的请求可能会经过 cryptosystem 一次或多次。 这使得对称加密非常适合我们的方案，并且我们可以为非对称加密提供折扣，直到需要这样的时间。

## <a name="design-philosophy"></a>设计理念

首先，我们要找出现有堆栈的问题。 完成这一工作后，我们调查了现有解决方案的前景，并最终找不到现有解决方案。 然后，基于多个指导原则对解决方案进行了设计。

* 系统应简化配置。 理想情况下，系统将是零配置，开发人员可以在运行。 当开发人员需要配置特定的方面 (例如密钥存储库) 时，应考虑将这些特定的配置简化。

* 提供简单的面向用户的 API。 Api 应易于使用，并且难以正确使用。

* 开发人员不应了解关键管理原则。 系统应代表开发人员处理算法选择和密钥生存期。 理想情况下，开发人员绝不会有权访问原始密钥材料。

* 应尽可能保护密钥。 系统应该找出适当的默认保护机制，并自动应用它。

考虑到这些原则，我们开发了简单 [易用](xref:security/data-protection/using-data-protection) 的数据保护堆栈。

ASP.NET Core 的数据保护 Api 主要用于机密负载的无限持久性。 其他技术（如 [WINDOWS CNG DPAPI](/windows/win32/seccng/cng-dpapi) 和 [Azure Rights Management](/rights-management/) ）更适用于无限存储的情况，并且具有相应的强大密钥管理功能。 也就是说，不会阻止开发人员使用 ASP.NET Core 的数据保护 Api 来长期保护机密数据。

## <a name="audience"></a>目标受众

数据保护系统分为五个主要包。 这些 Api 的各个方面面向三个主要受众;

1. [使用者 Api 概述](xref:security/data-protection/consumer-apis/overview)目标应用程序和框架开发人员。

   "我不想了解堆栈如何操作或如何进行配置。 我只是想要尽可能简单的方式执行一些操作，而使用 Api 成功的可能性很高。 "

2. [配置 api](xref:security/data-protection/configuration/overview)面向应用程序开发人员和系统管理员。

   "我需要告诉数据保护系统，我的环境需要非默认路径或设置。"

3. 扩展性 Api 面向开发人员负责实现自定义策略的目标。 这些 Api 的使用仅限于极少的情况和经验丰富的安全感知开发人员。

   "我需要替换系统中的整个组件，因为我有真正独特的行为要求。 我愿意了解 API 表面不常见使用的部分，以便构建满足我的需求的插件。 "

## <a name="package-layout"></a>程序包布局

数据保护堆栈包含5个包。

* [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Abstractions/) 包含 <xref:Microsoft.AspNetCore.DataProtection.IDataProtectionProvider> <xref:Microsoft.AspNetCore.DataProtection.IDataProtector> 用于创建数据保护服务的和接口。 它还包含用于处理这些类型的有用扩展方法 (例如 [IDataProtector](xref:Microsoft.AspNetCore.DataProtection.DataProtectionCommonExtensions.Protect*)) 。 如果数据保护系统在其他位置进行了实例化，而你使用的是 API，请参考 `Microsoft.AspNetCore.DataProtection.Abstractions` 。

* [AspNetCore. DataProtection](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection/) 包含数据保护系统的核心实现，包括核心加密操作、密钥管理、配置和扩展性。 若要实例化数据保护系统 (例如，将其添加到 <xref:Microsoft.Extensions.DependencyInjection.IServiceCollection>) 或修改或扩展其行为，引用 `Microsoft.AspNetCore.DataProtection` 。

* [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.Extensions/) 包含开发人员可能会发现但不属于核心包的其他 api。 例如，此包包含实例化数据保护系统的工厂方法，用于将密钥存储在文件系统上的某个位置，而无需进行依赖关系注入 (请参阅 <xref:Microsoft.AspNetCore.DataProtection.DataProtectionProvider>) 。 它还包含用于限制受保护负载的生存期的扩展方法 (参阅 <xref:Microsoft.AspNetCore.DataProtection.ITimeLimitedDataProtector>) 。

* [Microsoft.AspNetCore.DataProtection.Sys](https://www.nuget.org/packages/Microsoft.AspNetCore.DataProtection.SystemWeb/) 可以将 temWeb 安装到现有的 ASP.NET 4.x 应用程序中，以将其操作重定向 `<machineKey>` 为使用新的 ASP.NET Core 数据保护堆栈。 有关详细信息，请参阅 <xref:security/data-protection/compatibility/replacing-machinekey>。

* [AspNetCore 密钥派生](https://www.nuget.org/packages/Microsoft.AspNetCore.Cryptography.KeyDerivation/) 提供 PBKDF2 密码哈希例程的实现，并可由必须安全处理用户密码的系统使用。 有关详细信息，请参阅 <xref:security/data-protection/consumer-apis/password-hashing>。

## <a name="additional-resources"></a>其他资源

<xref:host-and-deploy/web-farm>
