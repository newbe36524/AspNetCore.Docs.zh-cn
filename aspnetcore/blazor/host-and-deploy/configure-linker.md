---
title: 配置 ASP.NET Core Blazor 链接器
author: guardrex
description: 了解在构建 Blazor 应用时如何控制中间语言 (IL) 链接器。
monikerRange: '>= aspnetcore-3.1 < aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 05/19/2020
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
uid: blazor/host-and-deploy/configure-linker
ms.openlocfilehash: 34582fdeb4951a110b03880887b978add07687f4
ms.sourcegitcommit: 0cfada7cbcd8e76aba0ae70eb6bbbf4437f287cc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/14/2020
ms.locfileid: "90081632"
---
# <a name="configure-the-linker-for-aspnet-core-no-locblazor"></a>配置 ASP.NET Core Blazor 链接器

作者：[Luke Latham](https://github.com/guardrex)

Blazor WebAssembly 在生成期间执行[中间语言 (IL)](/dotnet/standard/managed-code#intermediate-language--execution) 链接，以从应用的输出程序集中剪裁不必要的 IL。 在调试配置中生成时，将禁用链接器。 应用必须在发布配置中生成才能启用链接器。 部署 Blazor WebAssembly 应用时，建议在发布中生成。 

链接应用可以优化大小，但可能会造成不利影响。 使用反射或相关动态功能的应用可能会在剪裁时中断，因为链接器不知道此动态行为，而且通常无法确定在运行时反射所需的类型。 若要剪裁此类应用，必须通知链接器应用所依赖的代码和包或框架中的反射所需的任何类型。

若要确保剪裁后的应用在部署后正常工作，请务必在开发时经常对应用的发行版本进行测试。

可以使用以下 MSBuild 功能配置 Blazor 应用的链接：

* 使用 [MSBuild 属性](#control-linking-with-an-msbuild-property)全局配置链接。
* 使用[配置文件](#control-linking-with-a-configuration-file)按程序集控制链接。

## <a name="control-linking-with-an-msbuild-property"></a>使用 MSBuild 属性控制链接

在 `Release` 配置中生成应用时，将启用链接。 若要对此进行更改，请在项目文件中配置 `BlazorWebAssemblyEnableLinking` MSBuild 属性：

```xml
<PropertyGroup>
  <BlazorWebAssemblyEnableLinking>false</BlazorWebAssemblyEnableLinking>
</PropertyGroup>
```

## <a name="control-linking-with-a-configuration-file"></a>使用配置文件控制链接

通过提供 XML 配置文件并在项目文件中将该文件指定为 MSBuild 项，按程序集控制链接：

```xml
<ItemGroup>
  <BlazorLinkerDescriptor Include="LinkerConfig.xml" />
</ItemGroup>
```

`LinkerConfig.xml`：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
  This file specifies which parts of the BCL or Blazor packages must not be
  stripped by the IL Linker even if they aren't referenced by user code.
-->
<linker>
  <assembly fullname="mscorlib">
    <!--
      Preserve the methods in WasmRuntime because its methods are called by 
      JavaScript client-side code to implement timers.
      Fixes: https://github.com/dotnet/blazor/issues/239
    -->
    <type fullname="System.Threading.WasmRuntime" />
  </assembly>
  <assembly fullname="System.Core">
    <!--
      System.Linq.Expressions* is required by Json.NET and any 
      expression.Compile caller. The assembly isn't stripped.
    -->
    <type fullname="System.Linq.Expressions*" />
  </assembly>
  <!--
    In this example, the app's entry point assembly is listed. The assembly
    isn't stripped by the IL Linker.
  -->
  <assembly fullname="MyCoolBlazorApp" />
</linker>
```

有关详细信息和示例，请参阅[数据格式（mono/链接器 GitHub 存储库）](https://github.com/mono/linker/blob/master/docs/data-formats.md)。

## <a name="add-an-xml-linker-configuration-file-to-a-library"></a>将 XML 链接器配置文件添加到库

要针对特定库配置链接器，请将 XML 链接器配置文件作为嵌入的资源添加到库中。 嵌入的资源必须与程序集同名。

在以下示例中，`LinkerConfig.xml` 文件被指定为与库的程序集同名的嵌入资源：

```xml
<ItemGroup>
  <EmbeddedResource Include="LinkerConfig.xml">
    <LogicalName>$(MSBuildProjectName).xml</LogicalName>
  </EmbeddedResource>
</ItemGroup>
```

### <a name="configure-the-linker-for-internationalization"></a>配置链接器以实现国际化

默认情况下，Blazor 对于 Blazor WebAssembly 应用的链接器配置会去除国际化信息（显式请求的区域设置除外）。 删除这些程序集可最大程度地缩减应用的大小。

要控制保留哪些国际化程序集，请在项目文件中设置 `<BlazorWebAssemblyI18NAssemblies>` MSBuild 属性：

```xml
<PropertyGroup>
  <BlazorWebAssemblyI18NAssemblies>{all|none|REGION1,REGION2,...}</BlazorWebAssemblyI18NAssemblies>
</PropertyGroup>
```

| 区域值     | Mono 区域程序集    |
| ---------------- | ----------------------- |
| `all`            | 包含的所有程序集 |
| `cjk`            | `I18N.CJK.dll`          |
| `mideast`        | `I18N.MidEast.dll`      |
| `none`（默认值） | None                    |
| `other`          | `I18N.Other.dll`        |
| `rare`           | `I18N.Rare.dll`         |
| `west`           | `I18N.West.dll`         |

各个值之间用逗号分隔（例如：`mideast,west`）。

有关详细信息，请参阅[国际化：Pnetlib 国际化框架库（mono/mono GitHub 存储库）](https://github.com/mono/mono/tree/master/mcs/class/I18N)。

## <a name="additional-resources"></a>其他资源

* <xref:blazor/webassembly-performance-best-practices#intermediate-language-il-linking>
