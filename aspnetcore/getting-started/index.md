---
title: ASP.NET Core 入门
author: rick-anderson
description: 介绍如何使用 ASP.NET Core 创建并运行简单的 Hello World 应用的快速教程。
ms.author: riande
ms.custom: mvc
ms.date: 01/07/2020
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
uid: getting-started
ms.openlocfilehash: fef5ae525a7c01d0ea7733e990233f413aac61a7
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93057786"
---
# <a name="tutorial-get-started-with-aspnet-core"></a>教程：ASP.NET Core 入门

本教程介绍如何使用 .NET Core CLI 创建并运行 ASP.NET Core Web 应用。

你将了解如何：

> [!div class="checklist"]
> * 创建 Web 应用项目。
> * 信任开发证书。
> * 运行应用。
> * 编辑 Razor 页面。

最后，在本地计算机上运行工作 Web 应用。

![Web 应用主页](_static/home-page.png)

## <a name="prerequisites"></a>先决条件

[!INCLUDE[](~/includes/3.1-SDK.md)]

## <a name="create-a-web-app-project"></a>创建 Web 应用项目

打开命令行界面，然后输入以下命令：

```dotnetcli
dotnet new webapp -o aspnetcoreapp
```

上面的命令：

* 创建新 Web 应用。  
* `-o aspnetcoreapp` 参数使用应用的源文件创建名为 aspnetcoreapp 的目录。

### <a name="trust-the-development-certificate"></a>信任开发证书

信任 HTTPS 开发证书：

# <a name="windows"></a>[Windows](#tab/windows)

```dotnetcli
dotnet dev-certs https --trust
```

以上命令会显示以下对话：

![安全警告对话](~/getting-started/_static/cert.png)

如果你同意信任开发证书，请选择“是”。

# <a name="macos"></a>[macOS](#tab/macos)

```dotnetcli
dotnet dev-certs https --trust
```

以上命令会显示以下消息：

*请求信任 HTTPS 开发证书。如果证书尚不受信任，我们将运行以下命令：* `'sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain <<certificate>>'`

此命令可能会提示你输入密码以在系统密钥链上安装证书。 如果你同意信任开发证书，请输入密码。

# <a name="linux"></a>[Linux](#tab/linux)

查看你的 Linux 分发对应的文档，了解如何信任 HTTPS 开发证书。

---

有关详细信息，请参阅[信任 ASP.NET Core HTTPS 开发证书](xref:security/enforcing-ssl#trust-the-aspnet-core-https-development-certificate-on-windows-and-macos)

## <a name="run-the-app"></a>运行应用

运行以下命令：

```dotnetcli
cd aspnetcoreapp
dotnet watch run
```

命令行界面指明应用已启动后，浏览到 `https://localhost:5001`。

## <a name="edit-a-no-locrazor-page"></a>编辑 Razor 页面

打开 Pages/Index.cshtml，并使用以下突出显示标记修改并保存页面：

[!code-cshtml[](sample/index.cshtml?highlight=9)]

浏览到 `https://localhost:5001`，刷新页面并验证是否显示更改。

## <a name="next-steps"></a>后续步骤

在本教程中，你将了解：

> [!div class="checklist"]
> * 创建 Web 应用项目。
> * 信任开发证书。
> * 运行该项目。
> * 进行更改。

要了解有关 ASP.NET Core 的详细信息，请参阅简介中推荐的学习路径：

> [!div class="nextstepaction"]
> <xref:index#recommended-learning-path>
