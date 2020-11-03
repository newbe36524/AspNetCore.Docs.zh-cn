---
title: 将应用部署到应用服务 - 使用 ASP.NET Core 和 Azure 实施 DevOps
author: CamSoper
description: 要使用 ASP.NET Core 和 Azure 实施 DevOps，首先是将 ASP.NET Core 应用部署到 Azure 应用服务。
ms.author: casoper
ms.custom: devx-track-csharp, mvc, seodec18, devx-track-azurecli
ms.date: 10/24/2018
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
uid: azure/devops/deploy-to-app-service
ms.openlocfilehash: e6d8b4bcbbbe909fde971a8c706287654fcc98ba
ms.sourcegitcommit: 62cc131969b2379f7a45c286a751e22d961dfbdb
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/22/2020
ms.locfileid: "90847619"
---
# <a name="deploy-an-app-to-app-service"></a>将应用部署到应用服务

[Azure 应用服务](/azure/app-service/)是 Azure 的 Web 托管平台。 将 Web 应用部署到 Azure 应用服务的操作可手动完成，也可自动处理。 本指南的该部分介绍了一些部署方法，它们可使用命令行手动触发或按脚本触发，也可使用 Visual Studio 手动触发。

在本部分中，你将完成以下任务：

* 下载并构建示例应用。
* 使用 Azure Cloud Shell 创建 Azure 应用服务 Web 应用。
* 使用 Git 将示例应用部署到 Azure。
* 使用 Visual Studio 将更改部署到应用。
* 将过渡槽添加到 Web 应用。
* 将更新部署到过渡槽。
* 交换过渡槽和生产槽。

## <a name="download-and-test-the-app"></a>下载和测试应用

本指南使用的是预生成的 ASP.NET Core 应用，即[简单源读取器](https://github.com/Azure-Samples/simple-feed-reader/)。 它是一种 Razor Pages 应用，使用 `Microsoft.SyndicationFeed.ReaderWriter` API 来检索 RSS/Atom 源并在列表中显示消息项。

可随意查看代码，但请务必了解此应用并无特别之处。 它只是一个简单的、用于说明的 ASP.NET Core 应用。

在命令行界面中，按如下方式下载代码、生成项目并运行该项目。

> *注意：* Linux/macOS 用户应对路径进行适当的更改，例如使用正斜杠 (`/`) 而不是反斜杠 (`\`)。

1. 将代码复制到本地计算机上的文件夹。

    ```console
    git clone https://github.com/Azure-Samples/simple-feed-reader/
    ```

2. 将工作文件夹更改为刚创建的“简单源读取器”文件夹  。

    ```console
    cd .\simple-feed-reader\SimpleFeedReader
    ```

3. 还原包并生成解决方案。

    ```dotnetcli
    dotnet build
    ```

4. 运行应用。

    ```dotnetcli
    dotnet run
    ```

    ![dotnet run 命令运行成功](./media/deploying-to-app-service/dotnet-run.png)

5. 打开浏览器并导航到 `http://localhost:5000` 。 通过应用，可键入或粘贴联合源 URL，还可查看消息项列表。

     ![显示 RSS 源内容的应用](./media/deploying-to-app-service/app-in-browser.png)

6. 如果确信应用运行正常，请在命令行界面中按 Ctrl+C 将其关闭   。

## <a name="create-the-azure-app-service-web-app"></a>创建 Azure 应用服务 Web 应用

要部署应用，需要创建一个应用服务 [Web 应用](/azure/app-service/app-service-web-overview)。 创建 Web 应用后，你将使用 Git 从本地计算机部署到该应用。

1. 登录到 [Azure Cloud Shell](https://shell.azure.com/bash)。 注意：首次登录时，Cloud Shell 会提示为配置文件创建存储帐户。 请接受默认名称或提供唯一名称。

2. 在 Cloud Shell 中执行以下步骤。

    a. 声明一个变量，用于存储 Web 应用的名称。 该名称必须是唯一的，才能在默认 URL 中使用。 使用 `$RANDOM` Bash 函数构造名称可确保唯一性且生成 `webappname99999` 格式的结果。

    ```console
    webappname=mywebapp$RANDOM
    ```

    b. 创建资源组。 通过资源组可将 Azure 资源聚合起来，以组的形式进行管理。

    ```azurecli
    az group create --location centralus --name AzureTutorial
    ```

    `az` 命令会调用 [Azure CLI](/cli/azure/)。 尽管 CLI 可本地运行，但在 Cloud Shell 中使用可省时省配置。

    c. 在 S1 层级创建应用服务计划。 应用服务计划是一组共享同一定价层的 Web 应用。 S1 层级并不免费，但却是过渡槽功能所必需的。

    ```azurecli
    az appservice plan create --name $webappname --resource-group AzureTutorial --sku S1
    ```

    d. 使用同一资源组中的应用服务计划创建 Web 应用资源。

    ```azurecli
    az webapp create --name $webappname --resource-group AzureTutorial --plan $webappname
    ```

    e. 设置部署凭据。 这些部署凭据适用于订阅中的所有 Web 应用。 用户名中不要使用特殊字符。

    ```azurecli
    az webapp deployment user set --user-name REPLACE_WITH_USER_NAME --password REPLACE_WITH_PASSWORD
    ```

    f. 将 Web 应用配置为接受本地 Git 部署并显示 Git 部署 URL  。 请记下此 URL 供稍后参考  。

    ```azurecli
    echo Git deployment URL: $(az webapp deployment source config-local-git --name $webappname --resource-group AzureTutorial --query url --output tsv)
    ```

    g. 显示 Web 应用 URL  。 访问此 URL，查看空白 Web 应用。 请记下此 URL 供稍后参考  。

    ```console
    echo Web app URL: http://$webappname.azurewebsites.net
    ```

3. 使用本地计算机上的命令行界面导航到 Web 应用的项目文件夹（例如 `.\simple-feed-reader\SimpleFeedReader`）。 执行以下命令，将 Git 设置为推送到部署 URL：

    a. 将远程 URL 添加到本地存储库。

    ```console
    git remote add azure-prod GIT_DEPLOYMENT_URL
    ```

    b. 将本地 master 分支推送到 azure-prod 远程的 master 分支    。

    ```console
    git push azure-prod master
    ```

    系统将提示使用前面创建的部署凭据。 查看命令行界面中的输出。 Azure 会远程构建 ASP.NET Core 应用。

4. 在浏览器中，导航到 Web 应用 URL 并注意应用是否已构建且已经过部署  。 使用 `git commit` 可将其他更改提交到本地 Git 存储库。 使用前面的 `git push` 命令可将这些更改推送到 Azure。

## <a name="deployment-with-visual-studio"></a>使用 Visual Studio 进行部署

> *注意：本部分仅适用于 Windows。Linux 和 macOS 用户应进行下面步骤 2 中所述的更改。保存文件，然后使用 `git commit` 将更改提交到本地存储库。最后如第一部分所述，使用 `git push` 推送更改*。

已从命令行界面部署了应用。 接下来使用 Visual Studio 的集成工具将更新部署到应用。 在后台，Visual Studio 会实现与命令行工具相同的操作，但是在 Visual Studio 的常见 UI 中完成的。

1. 在 Visual Studio 中打开 SimpleFeedReader.sln  。
2. 在解决方案资源管理器中，打开 Pages\Index.cshtml  。 将 `<h2>Simple Feed Reader</h2>` 更改为 `<h2>Simple Feed Reader - V2</h2>`。
3. 按 Ctrl+Shift+B 构建应用    。
4. 在解决方案资源管理器中，右键单击该项目并单击“发布”  。

    ![显示右键单击和“发布”的屏幕截图](./media/deploying-to-app-service/publish.png)
5. Visual Studio 可创建新的应用服务资源，但此更新将通过现有部署发布。 在“选取发布目标”对话框中，选择左侧列表中的“应用服务”，再选择“选择现有项”    。 单击“发布”  。
6. 在“应用服务”对话框中，确认右上角显示了用于创建 Azure 订阅的 Microsoft 帐户或组织帐户  。 如果未显示，请单击下拉箭头并进行添加。
7. 确认已选择正确的 Azure 订阅  。 在“视图”部分，选择“资源组”   。 展开 AzureTutorial 资源组，然后选择现有的 Web 应用  。 单击 **“确定”** 。

    ![显示“发布应用服务”对话框的屏幕截图](./media/deploying-to-app-service/publish-dialog.png)

Visual Studio 会构建应用并将其部署到 Azure。 访问 Web 应用 URL。 验证 `<h2>` 元素修改是否有效。

![标题已更改的应用](./media/deploying-to-app-service/app-v2.png)

## <a name="deployment-slots"></a>部署槽位

部署槽位支持更改过渡，且不会影响生产环境中运行的应用。 在质量保证团队验证过渡版应用后，就可交换生产槽和过渡槽。 过渡期应用可以这种方式推广到生产槽。 以下步骤将创建一个过渡槽，对过渡槽部署一些更改，然后在验证后交换过渡槽和生产槽。

1. 如果尚未登录 [Azure Cloud Shell](https://shell.azure.com/bash)，请先登录。
2. 创建过渡槽。

    a. 创建名为“过渡”的部署槽位  。

    ```azurecli
    az webapp deployment slot create --name $webappname --resource-group AzureTutorial --slot staging
    ```

    b. 将过渡槽配置为使用本地 Git 部署并获取过渡部署 URL  。 请记下此 URL 供稍后参考  。

    ```azurecli
    echo Git deployment URL for staging: $(az webapp deployment source config-local-git --name $webappname --resource-group AzureTutorial --slot staging --query url --output tsv)
    ```

    c. 显示过渡槽的 URL。 访问 URL，查看空过渡槽。 请记下此 URL 供稍后参考  。

    ```console
    echo Staging web app URL: http://$webappname-staging.azurewebsites.net
    ```

3. 在文本编辑器或 Visual Studio 中，再次修改 Pages/Index.cshtml，以便 `<h2>` 元素读取 `<h2>Simple Feed Reader - V3</h2>` 并保存文件  。

4. 将文件提交到本地 Git 存储库，方式是使用 Visual Studio“团队资源管理器”选项卡中的“更改”页面，或者使用本地计算机的命令行界面输入以下内容   ：

    ```console
    git commit -a -m "upgraded to V3"
    ```

5. 使用本地计算机的命令行界面将过渡部署 URL 添加为 Git 远程，并推送已提交的更改：

    a. 将过渡的远程 URL 添加到本地 Git 存储库。

    ```console
    git remote add azure-staging <Git_staging_deployment_URL>
    ```

    b. 将本地 master 分支推送到 azure-staging 远程的 master 分支    。

    ```console
    git push azure-staging master
    ```

    请在 Azure 构建和部署应用时稍事等待。

6. 若要验证 V3 是否已部署到过渡槽，请打开两个浏览器窗口。 在其中一个窗口中，访问初始 Web 应用 URL。 在另一个窗口中，访问过渡 Web 应用 URL。 生产 URL 适用于应用的 V2。 过渡 URL 适用于应用的 V3。

    ![对比浏览器窗口的屏幕截图](./media/deploying-to-app-service/ready-to-swap.png)

7. 在 Cloud Shell 中，将已验证/已准备好的过渡槽交换到生产环境。

    ```azurecli
    az webapp deployment slot swap --name $webappname --resource-group AzureTutorial --slot staging
    ```

8. 刷新两个浏览器窗口，验证是否进行了交换。

    ![对比交换后的浏览器窗口](./media/deploying-to-app-service/swapped.png)

## <a name="summary"></a>总结

在本部分中完成了以下任务：

* 下载并构建了示例应用。
* 使用 Azure Cloud Shell 创建了 Azure 应用服务 Web 应用。
* 使用 Git 将示例应用部署到了 Azure。
* 使用 Visual Studio 将更改部署到了应用。
* 将过渡槽添加到了 Web 应用。
* 将更新部署到了过渡槽。
* 交换了过渡槽和生产槽。

在下一部分，你将了解如何使用 Azure Pipelines 构建 DevOps 管道。

## <a name="additional-reading"></a>其他阅读材料

* [Web 应用概述](/azure/app-service/app-service-web-overview)
* [在 Azure 应用服务中生成 .NET Core 和 SQL 数据库 Web 应用](/azure/app-service/app-service-web-tutorial-dotnetcore-sqldb)
* [为 Azure 应用服务配置部署凭据](/azure/app-service/app-service-deployment-credentials)
* [设置 Azure 应用服务中的过渡环境](/azure/app-service/web-sites-staged-publishing)
