---
title: ASP.NET Core MVC 入门
author: rick-anderson
description: 了解如何开始使用 ASP.NET Core MVC。
ms.author: riande
ms.date: 10/16/2019
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
uid: tutorials/first-mvc-app/start-mvc
ms.openlocfilehash: 177112106d143a6826c1f927aac807da0aa9f2b4
ms.sourcegitcommit: ecae2aa432628b9181d1fa11037c231c7dd56c9e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/16/2020
ms.locfileid: "92113824"
---
# <a name="get-started-with-aspnet-core-mvc"></a>ASP.NET Core MVC 入门

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

本教程介绍构建 ASP.NET Core MVC Web 应用的基础知识。

该应用管理电影标题的数据库。 您将学习如何：

> [!div class="checklist"]
> * 创建 Web 应用。
> * 添加和构架模型。
> * 使用数据库。
> * 添加搜索和验证。

在结束时，你会获得可以管理和显示电影数据的应用。

[!INCLUDE[](~/includes/mvc-intro/download.md)]

## <a name="prerequisites"></a>先决条件

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs-3.1.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-3.1.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-3.1.md)]

---

## <a name="create-a-web-app"></a>创建 Web 应用

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在 Visual Studio 中，选择“创建新项目”。

* 选择“ASP.NET Core Web 应用程序”>“下一步”。

![新建 ASP.NET Core Web 应用程序](start-mvc/_static/np_2.1.png)

* 将项目命名为“MvcMovie”，然后选择“创建” 。 将项目命名为“MvcMovie”非常重要，这样在复制代码时，命名空间才会匹配。

  ![新建 ASP.NET Core Web 应用程序](start-mvc/_static/config.png)

* 选择“Web 应用(模型-视图-控制器)”。 在下拉框中，选择“.NET Core”和“ASP.NET Core 3.1”，然后选择“创建”。

![“新建项目”对话框，左窗格中的“.NET Core”，ASP.NET Core Web ](start-mvc/_static/new_project30.png)

Visual Studio 为刚刚创建的 MVC 项目使用默认模板。 输入项目名称并选择几个选项后，就拥有了一个可正常运行的应用。 这是一个基本的入门项目。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

本教程假定用户熟悉 VS Code。 有关详细信息，请参阅 [VS Code 入门](https://code.visualstudio.com/docs) 和 [Visual Studio Code 帮助](#visual-studio-code-help)。

* 打开[集成终端](https://code.visualstudio.com/docs/editor/integrated-terminal)。
* 将目录更改为 (`cd`) 包含项目的文件夹。
* 运行下面的命令：

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * 一个对话框随即出现，其中包含：“‘MvcMovie’中缺少进行生成和调试所需的资产。是否添加它们?”  选择“是”

  * `dotnet new mvc -o MvcMovie`：在 MvcMovie 文件夹中创建一个新的 ASP.NET Core MVC 项目。
  * `code -r MvcMovie`：在 Visual Studio Code 中加载 MvcMovie.csproj 项目文件。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 选择“文件”>“新建解决方案” 。

  ![macOS 新建解决方案](start-mvc/_static/new_project_vsmac.png)

* 在版本 8.6 之前的 Visual Studio for Mac 中，依次选择“.NET Core” > “应用” > “Web 应用程序(模型-视图-控制器)” > “下一步”。    在版本 8.6 或更高版本中，依次选择“Web 和控制台” > “应用” > “Web 应用程序(模型-视图-控制器)” > “下一步”。   

  ![macOS Web 应用模板选择](start-mvc/_static/web_app_template_vsmac.png)

* 在“配置新的 Web 应用”对话框中：

  * 确认已将“身份验证”设置为“无身份验证”。
  * 如果看到用于选择“目标框架”的选项，请选择最新的 3.x 版本。

  选择“下一步”。

* 将项目命名为“MvcMovie”，然后选择“创建”。 

  ![macOS 命名项目](start-mvc/_static/MvcMovie.png)

---

### <a name="run-the-app"></a>运行应用

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

选择 Ctrl+F5 以在非调试模式下运行应用。

[!INCLUDE[](~/includes/trustCertVS.md)]

* Visual Studio 启动 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 并运行应用。 请注意，地址栏显示 `localhost:port#`，而不显示 `example.com` 之类的内容。 这是因为 `localhost` 是本地计算机的标准主机名。 Visual Studio 创建 Web 项目时，Web 服务器使用的是随机端口。
* 使用 Ctrl+F5 启动应用（非调试模式）后，可执行代码更改、保存文件、刷新浏览器和查看代码更改等操作。 许多开发人员更喜欢使用非调试模式快速启动应用并查看更改。
* 可以从“调试”菜单项中以调试或非调试模式启动应用：

  ![调试菜单](start-mvc/_static/debug_menu.png)

* 可以通过选择“IIS Express”按钮来调试应用

  ![IIS Express](start-mvc/_static/iis_express.png)

  下图显示该应用：

  ![主页或索引页](start-mvc/_static/home2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

按 Ctrl+F5 以在不使用调试程序的情况下运行。

[!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code 启动 [Kestrel](xref:fundamentals/servers/kestrel)，启动浏览器并导航到 `https://localhost:5001`。 地址栏显示 `localhost:port:5001`，而不是显示 `example.com`。 这是因为 `localhost` 是本地计算机的标准主机名。 Localhost 仅为来自本地计算机的 Web 请求提供服务。

  使用 Ctrl+F5 启动应用（非调试模式）后，可执行代码更改、保存文件、刷新浏览器和查看代码更改等操作。 许多开发人员更喜欢使用非调试模式刷新页面并查看更改。

  ![主页或索引页](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

选择“运行” > “开始执行(不调试)”以启动应用 。 Visual Studio for Mac 启动 [Kestrel](xref:fundamentals/servers/index#kestrel) 服务器，启动浏览器并导航到 `http://localhost:port`，其中的 port 是随机选择的端口号。

[!INCLUDE[](~/includes/trustCertMac.md)]

* 地址栏显示 `localhost:port#`，而不是显示 `example.com`。 这是因为 `localhost` 是本地计算机的标准主机名。 Visual Studio 创建 Web 项目时，Web 服务器使用的是随机端口。 运行应用时，将看到不同的端口号。
* 可以从“运行”菜单中以调试或非调试模式启动应用。

  下图显示该应用：

  ![主页或索引页](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

在本教程的下一部分中，你将了解 MVC 并开始撰写一些代码。

> [!div class="step-by-step"]
> [下一页](adding-controller.md)

::: moniker-end

::: moniker range="< aspnetcore-3.0"

[!INCLUDE [consider RP](~/includes/razor.md)]

本教程介绍构建 ASP.NET Core MVC Web 应用的基础知识。

该应用管理电影标题的数据库。 您将学习如何：

> [!div class="checklist"]
> * 创建 Web 应用。
> * 添加和构架模型。
> * 使用数据库。
> * 添加搜索和验证。

在结束时，你会获得可以管理和显示电影数据的应用。

[!INCLUDE[](~/includes/mvc-intro/download.md)]

## <a name="prerequisites"></a>先决条件

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

[!INCLUDE[](~/includes/net-core-prereqs-vs2019-2.2.md)]

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

[!INCLUDE[](~/includes/net-core-prereqs-vsc-2.2.md)]

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

[!INCLUDE[](~/includes/net-core-prereqs-mac-2.2.md)]

---
## <a name="create-a-web-app"></a>创建 Web 应用

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

* 在 Visual Studio 中，选择“创建新项目”。

* 选择“ASP.NET Core Web 应用程序”，然后选择“下一步” 。

![新建 ASP.NET Core Web 应用程序](start-mvc/_static/np_2.1.png)

* 将项目命名为“MvcMovie”，然后选择“创建” 。 将项目命名为“MvcMovie”非常重要，这样在复制代码时，命名空间才会匹配。

  ![新建 ASP.NET Core Web 应用程序](start-mvc/_static/config.png)


* 选择“Web 应用程序(模型-视图-控制器)”，然后选择“创建” 。

![“新建项目”对话框，左窗格中的“.NET Core”，ASP.NET Core Web ](start-mvc/_static/new_project22-21.png)

Visual Studio 为刚刚创建的 MVC 项目使用默认模板。 输入项目名称并选择几个选项后，就拥有了一个可正常运行的应用。 这是一个基本的初学者项目，适合入门使用。

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

本教程假定用户熟悉 VS Code。 有关详细信息，请参阅 [VS Code 入门](https://code.visualstudio.com/docs) 和 [Visual Studio Code 帮助](#visual-studio-code-help)。

* 打开[集成终端](https://code.visualstudio.com/docs/editor/integrated-terminal)。
* 将目录更改为 (`cd`) 包含项目的文件夹。
* 运行下面的命令：

   ```dotnetcli
   dotnet new mvc -o MvcMovie
   code -r MvcMovie
   ```

  * 一个对话框随即出现，其中包含：“‘MvcMovie’中缺少进行生成和调试所需的资产。是否添加它们?”  选择“是”

  * `dotnet new mvc -o MvcMovie`：在 MvcMovie 文件夹中创建一个新的 ASP.NET Core MVC 项目。
  * `code -r MvcMovie`：在 Visual Studio Code 中加载 MvcMovie.csproj 项目文件。

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

* 选择“文件”>“新建解决方案” 。

  ![macOS 新建解决方案](./start-mvc/_static/new_project_vsmac.png)

* 在版本 8.6 之前的 Visual Studio for Mac 中，依次选择“.NET Core” > “应用” > “Web 应用程序(模型-视图-控制器)” > “下一步”。    在版本 8.6 或更高版本中，依次选择“Web 和控制台” > “应用” > “Web 应用程序(模型-视图-控制器)” > “下一步”。   

* 在“配置新的 Web 应用”对话框中：

  * 确认已将“身份验证”设置为“无身份验证”。
  * 如果看到用于选择“目标框架”的选项，请选择最新的 2.x 版本。

  选择“下一步”。

* 将项目命名为“MvcMovie”，然后选择“创建”。 

---

### <a name="run-the-app"></a>运行应用

# <a name="visual-studio"></a>[Visual Studio](#tab/visual-studio)

选择 Ctrl+F5 以在非调试模式下运行应用。

[!INCLUDE[](~/includes/trustCertVS.md)]

* Visual Studio 启动 [IIS Express](/iis/extensions/introduction-to-iis-express/iis-express-overview) 并运行应用。 请注意，地址栏显示 `localhost:port#`，而不显示 `example.com` 之类的内容。 这是因为 `localhost` 是本地计算机的标准主机名。 Visual Studio 创建 Web 项目时，Web 服务器使用的是随机端口。
* 使用 Ctrl+F5 启动应用（非调试模式）后，可执行代码更改、保存文件、刷新浏览器和查看代码更改等操作。 许多开发人员更喜欢使用非调试模式快速启动应用并查看更改。
* 可以从“调试”菜单项中以调试或非调试模式启动应用：

  ![调试菜单](start-mvc/_static/debug_menu.png)

* 可以通过选择“IIS Express”按钮来调试应用

  ![IIS Express](start-mvc/_static/iis_express.png)

* 选择“接受”以同意跟踪。 此应用不会跟踪个人信息。 模板生成的代码包含有助于符合[一般数据保护条例 (GDPR)](xref:security/gdpr) 的资产。

  ![主页或索引页](start-mvc/_static/privacy.png)

  下图展示了接受跟踪后的应用：

  ![主页或索引页](start-mvc/_static/home2.2.png)

# <a name="visual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

按 Ctrl+F5 以在不使用调试程序的情况下运行。

[!INCLUDE[](~/includes/trustCertVSC.md)]

  Visual Studio Code 启动 [Kestrel](xref:fundamentals/servers/kestrel)，启动浏览器并导航到 `https://localhost:5001`。 地址栏显示 `localhost:port:5001`，而不是显示 `example.com`。 这是因为 `localhost` 是本地计算机的标准主机名。 Localhost 仅为来自本地计算机的 Web 请求提供服务。

  使用 Ctrl+F5 启动应用（非调试模式）后，可执行代码更改、保存文件、刷新浏览器和查看代码更改等操作。 许多开发人员更喜欢使用非调试模式刷新页面并查看更改。

* 选择“接受”以同意跟踪。 此应用不会跟踪个人信息。 模板生成的代码包含有助于符合[一般数据保护条例 (GDPR)](xref:security/gdpr) 的资产。

  ![主页或索引页](start-mvc/_static/privacy.png)

  下图展示了接受跟踪后的应用：

  ![主页或索引页](start-mvc/_static/home2.2.png)

# <a name="visual-studio-for-mac"></a>[Visual Studio for Mac](#tab/visual-studio-mac)

选择“运行” > “开始执行(不调试)”以启动应用 。 Visual Studio for Mac 启动 [Kestrel](xref:fundamentals/servers/index#kestrel) 服务器，启动浏览器并导航到 `http://localhost:port`，其中的 port 是随机选择的端口号。

[!INCLUDE[](~/includes/trustCertMac.md)]

* 地址栏显示 `localhost:port#`，而不是显示 `example.com`。 这是因为 `localhost` 是本地计算机的标准主机名。 Visual Studio 创建 Web 项目时，Web 服务器使用的是随机端口。 运行应用时，将看到不同的端口号。
* 可以从“运行”菜单中以调试或非调试模式启动应用。

* 选择“接受”以同意跟踪。 此应用不会跟踪个人信息。 模板生成的代码包含有助于符合[一般数据保护条例 (GDPR)](xref:security/gdpr) 的资产。

  ![主页或索引页](./start-mvc/_static/output_privacy_macos.png)

  下图展示了接受跟踪后的应用：

  ![主页或索引页](./start-mvc/_static/output_macos.png)

---

[!INCLUDE[](~/includes/vs-vsc-vsmac-help.md)]

在本教程的下一部分中，你将了解 MVC 并开始撰写一些代码。

> [!div class="step-by-step"]
> [下一页](adding-controller.md)

::: moniker-end
