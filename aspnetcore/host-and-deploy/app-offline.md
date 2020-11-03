---
title: 应用脱机文件 (app_offline.htm)
author: rick-anderson
description: 了解如何将应用脱机文件 (`app_offline.htm`) 与 ASP.NET Core 模块配合使用。
monikerRange: '>= aspnetcore-5.0'
ms.author: riande
ms.custom: mvc
ms.date: 01/13/2020
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
uid: host-and-deploy/iis/app-offline
ms.openlocfilehash: 95dfadd084af5909fee754308ad5d65f54d4875d
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91755148"
---
# <a name="app-offline-file-app_offlinehtm"></a>应用脱机文件 (`app_offline.htm`)

ASP.NET Core 模块使用应用脱机文件 (`app_offline.htm`) 来关闭应用。

如果在应用的根目录中检测到名为“`app_offline.htm`”的文件，ASP.NET Core 模块将尝试正常关闭应用并停止处理传入请求。 如果应用在 `shutdownTimeLimit` 中定义的秒数之后仍在运行，ASP.NET Core 模块将停止正在运行的进程。

存在 `app_offline.htm` 文件时，ASP.NET Core 模块会通过发送回 `app_offline.htm` 文件的内容来响应请求。 删除 `app_offline.htm` 文件后，下一个请求将启动应用。

使用进程外托管模型时，如果有已打开的连接，则应用可能不会立即关闭。 例如，WebSocket 连接可能会延迟应用关闭。

## <a name="locked-deployment-files"></a>锁定的部署文件

如果应用正在运行，部署文件夹中的文件会被锁定。 在部署期间，无法覆盖已锁定的文件。

`app_offline.htm` 是释放锁定文件的主要机制。 `app_offline.htm` 供 Web 部署用于正确地停止和启动应用。

可以手动将 `app_offline.htm` 用于启动和停止应用（需要 PowerShell 5 或更高版本）：

```powershell
$pathToApp = '{PATH TO APP}'

New-Item -Path $pathToApp app_offline.htm

# Provide script commands here to deploy the app

Remove-Item -Path $pathToApp app_offline.htm
```

在上述 PowerShell 脚本中：

* 占位符 `{PATH TO APP}` 是指向应用的路径。
* `New-Item` 命令停止应用池。
* `Remove-Item` 命令启动应用池。
* 开发人员提供 `New-Item` 命令和 `Remove-Item` 命令之间的命令来部署应用。

还可以通过在服务器上的 IIS 管理器中手动停止应用池来解锁文件。 使用 IIS 管理器停止和重新启动应用池时，请勿使用 `app_offine.htm` 文件。
