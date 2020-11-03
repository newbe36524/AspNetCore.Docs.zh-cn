---
title: 配置适用于 ASP.NET Core Blazor 的裁边器
author: guardrex
description: 了解在构建 Blazor 应用时如何控制中间语言 (IL) 链接器（裁边器）。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 09/14/2020
no-loc:
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
uid: blazor/host-and-deploy/configure-trimmer
ms.openlocfilehash: 57d8f069c79b558020253968d736f350bc8a6f03
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90721731"
---
# <a name="configure-the-trimmer-for-aspnet-core-no-locblazor"></a>配置适用于 ASP.NET Core Blazor 的裁边器

作者：[Pranav Krishnamoorthy](https://github.com/pranavkm)

Blazor WebAssembly 执行[中间语言 (IL)](/dotnet/standard/managed-code#intermediate-language--execution) 剪裁以减小发布输出的大小。

剪裁应用可以优化大小，但可能会造成不利影响。 使用反射或相关动态功能的应用可能会在剪裁时中断，因为链接器不知道此动态行为，而且通常无法确定在运行时反射所需的类型。 若要剪裁此类应用，必须通知链接器应用所依赖的代码和包或框架中的反射所需的任何类型。

若要确保剪裁后的应用在部署后正常工作，请务必在开发时经常对已发布的输出进行测试。

可以通过在应用的项目文件中将 `PublishTrimmed` MSBuild 属性设置为 `false` 来禁用对 .NET 应用的剪裁：

```xml
<PropertyGroup>
  <PublishTrimmed>false</PublishTrimmed>
</PropertyGroup>
```
配置剪裁的其他选项可以在[剪裁选项](/dotnet/core/deploying/trimming-options)中找到。

## <a name="additional-resources"></a>其他资源

* [裁剪自包含部署和可执行文件](/dotnet/core/deploying/trim-self-contained)
* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-trimming>
