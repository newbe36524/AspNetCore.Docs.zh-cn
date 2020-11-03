---
title: 替换 ASP.NET Core 中的 ASP.NET machineKey
author: rick-anderson
description: 了解如何在 ASP.NET 中替换 machineKey，以允许使用新的、更安全的数据保护系统。
ms.author: riande
ms.date: 04/06/2019
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
uid: security/data-protection/compatibility/replacing-machinekey
ms.openlocfilehash: 7c6766fa20b0df021013da77b5d88fecefd40c62
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93053054"
---
# <a name="replace-the-aspnet-machinekey-in-aspnet-core"></a>替换 ASP.NET Core 中的 ASP.NET machineKey

<a name="compatibility-replacing-machinekey"></a>

`<machineKey>`ASP.NET 中元素的实现[是可替换](https://blogs.msdn.microsoft.com/webdev/2012/10/23/cryptographic-improvements-in-asp-net-4-5-pt-2/)的。 这样，就可以通过一种替代数据保护机制（包括新的数据保护系统）来路由对 ASP.NET 加密例程的大多数调用。

## <a name="package-installation"></a>包安装

> [!NOTE]
> 新的数据保护系统只能安装到面向 .NET 4.5.1 或更高版本的现有 ASP.NET 应用程序中。 如果应用程序面向 .NET 4.5 或更低版本，则安装将失败。

若要在现有的 ASP.NET 4.5.1 + 项目中安装新的数据保护系统，请 Microsoft.AspNetCore.DataProtection.SystemWeb 安装包。 这将使用 [默认配置](xref:security/data-protection/configuration/default-settings) 设置来实例化数据保护系统。

安装包时，它会在 *Web.config* 中插入一行，告诉 ASP.NET 将其用于 [大多数加密操作](https://blogs.msdn.microsoft.com/webdev/2012/10/23/cryptographic-improvements-in-asp-net-4-5-pt-2/)，包括 forms 身份验证、视图状态和对 MachineKey 的调用。 插入的行如下所示。

```xml
<machineKey compatibilityMode="Framework45" dataProtectorType="..." />
```

>[!TIP]
> 可以通过检查 `__VIEWSTATE` （如下面的示例中应以 "CfDJ8" 开头）等字段来判断新的数据保护系统是否处于活动状态。 "CfDJ8" 是用于标识受数据保护系统保护的有效负载的幻 "09 F0 C9 F0" 标头的 base64 表示形式。

```html
<input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="CfDJ8AWPr2EQPTBGs3L2GCZOpk...">
```

## <a name="package-configuration"></a>包配置

数据保护系统使用默认的零安装配置进行实例化。 但是，由于默认情况下会将密钥保存到本地文件系统，因此，这不能用于在场中部署的应用程序。 若要解决此问题，可以通过创建子类 DataProtectionStartup 并重写其 ConfigureServices 方法的类型来提供配置。

下面是一个自定义数据保护启动类型的示例，该类型配置了两个密钥的保存位置和静态加密方式。 它还通过提供自己的应用程序名称来替代默认应用隔离策略。

```csharp
using System;
using System.IO;
using Microsoft.AspNetCore.DataProtection;
using Microsoft.AspNetCore.DataProtection.SystemWeb;
using Microsoft.Extensions.DependencyInjection;

namespace DataProtectionDemo
{
    public class MyDataProtectionStartup : DataProtectionStartup
    {
        public override void ConfigureServices(IServiceCollection services)
        {
            services.AddDataProtection()
                .SetApplicationName("my-app")
                .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\myapp-keys\"))
                .ProtectKeysWithCertificate("thumbprint");
        }
    }
}
```

>[!TIP]
> 你还可以使用 `<machineKey applicationName="my-app" ... />` 来代替对 SetApplicationName 的显式调用。 这是一种简便的机制，可避免开发人员在要配置的所有应用程序名称设置的情况下，强制创建 DataProtectionStartup 派生的类型。

若要启用此自定义配置，请返回 Web.config，并查找 `<appSettings>` 包安装添加到配置文件中的元素。 它将类似于以下标记：

```xml
<appSettings>
  <!--
  If you want to customize the behavior of the ASP.NET Core Data Protection stack, set the
  "aspnet:dataProtectionStartupType" switch below to be the fully-qualified name of a
  type which subclasses Microsoft.AspNetCore.DataProtection.SystemWeb.DataProtectionStartup.
  -->
  <add key="aspnet:dataProtectionStartupType" value="" />
</appSettings>
```

用刚刚创建的 DataProtectionStartup 派生类型的程序集限定名称填充空白值。 如果应用程序的名称为 DataProtectionDemo，则如下所示。

```xml
<add key="aspnet:dataProtectionStartupType"
     value="DataProtectionDemo.MyDataProtectionStartup, DataProtectionDemo" />
```

新配置的数据保护系统现在可以在应用程序内部使用。
