---
title: 使用 Visual Studio 将 ASP.NET Core 应用发布到 Azure
author: rick-anderson
description: 了解如何使用 Visual Studio 将 ASP.NET Core 应用发布到 Azure 应用服务。
ms.author: riande
ms.custom: devx-track-csharp, mvc
ms.date: 07/10/2019
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
uid: tutorials/publish-to-azure-webapp-using-vs
ms.openlocfilehash: ebfe89d68725d597d6975015bbf84414ff818829
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722905"
---
# <a name="publish-an-aspnet-core-app-to-azure-with-visual-studio"></a>使用 Visual Studio 将 ASP.NET Core 应用发布到 Azure

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)
::: moniker range=">= aspnetcore-3.0"

[!INCLUDE [Azure App Service Preview Notice](../includes/azure-apps-preview-notice.md)]

::: moniker-end


若使用的是 macOS，请参阅[使用 Visual Studio for Mac 将 Web 应用发布到 Azure 应用服务](/visualstudio/mac/publish-app-svc?view=vsmac-2019)。

若要对应用服务部署问题进行故障排除，请参阅 <xref:test/troubleshoot-azure-iis>。

## <a name="set-up"></a>设置

* 如果没有[免费 Azure 帐户](https://azure.microsoft.com/free/dotnet/)，请开设一个。 

## <a name="create-a-web-app"></a>创建 Web 应用

在 Visual Studio 起始页中，选择“文件”>“新建”>“项目...”

![“文件”菜单](publish-to-azure-webapp-using-vs/_static/file_new_project.png)

填写“新建项目”对话框：

* 选择“ASP.NET Core Web 应用程序”。
* 选择“下一步”。

![“新建项目”对话框](publish-to-azure-webapp-using-vs/_static/new_prj.png)

在“新建 ASP.NET Core Web 应用程序”对话框中：

* 选择“Web 应用程序”。
* 选择“身份验证”下的“更改”。

![“新建 ASP.NET Core Web”对话框](publish-to-azure-webapp-using-vs/_static/new_prj_2.png)

“更改身份验证”对话框随即出现。 

* 选择“个人用户帐户”。
* 选择“确定”返回到“新建 ASP.NET Core Web 应用程序”，然后选择“创建”。

![“新建 ASP.NET Core Web 身份验证”对话框](publish-to-azure-webapp-using-vs/_static/new_prj_auth.png) 

Visual Studio 随即创建解决方案。

## <a name="run-the-app"></a>运行应用

* 按 Ctrl+F5 运行项目。
* 测试“隐私”链接。

![localhost 上的 Microsoft Edge 中打开的 Web 应用程序](publish-to-azure-webapp-using-vs/_static/show.png)

### <a name="register-a-user"></a>注册用户

* 选择“注册”并注册新用户。 可使用虚构电子邮件地址。 提交时，页面上会显示以下错误：

    “处理请求时，数据库操作失败。*可通过向应用程序数据库上下文应用现有迁移解决此问题。”*
* 选择“应用迁移”，并在页面更新后刷新页面。

![处理请求时，数据库操作失败。 可能可通过向应用程序数据库上下文应用现有迁移解决此问题。](publish-to-azure-webapp-using-vs/_static/mig.png)

应用将显示用于注册新用户的电子邮件和一个“注销”链接。

![Microsoft Edge 中打开的 Web 应用程序。 注册链接会替换为文本“Hello user1@example.com!”](publish-to-azure-webapp-using-vs/_static/hello.png)

## <a name="deploy-the-app-to-azure"></a>将应用部署到 Azure

在解决方案资源管理器中右键单击该项目，然后选择“发布...”。

![打开且突出显示“发布”链接的上下文菜单](publish-to-azure-webapp-using-vs/_static/pub.png)

在“发布”对话框中：

* 选择“Azure”。
* 选择“下一步”。

![“发布”对话框](publish-to-azure-webapp-using-vs/_static/maas1.png)

在“发布”对话框中：

* 选择“Azure 应用服务 (Linux)”。
* 选择“下一步”。

![“发布”对话框：选择 Azure 服务](publish-to-azure-webapp-using-vs/_static/maas2.png)

在“发布”对话框中，选择“创建新的 Azure 应用服务...”

![“发布”对话框：选择 Azure 服务实例](publish-to-azure-webapp-using-vs/_static/maas3.png)

“创建应用服务”对话框随即显示：

* “应用名称”、“资源组”和“应用服务计划”输入字段已填充  。 可以保留这些名称，也可以进行更改。
* 选择“创建”。

![“创建应用服务”对话框](publish-to-azure-webapp-using-vs/_static/newrg1.png)

创建完成后，对话框将自动关闭，“发布”对话框将再次成为焦点：

* 将自动选择刚创建的新实例。
* 选择“完成”。

![“发布”对话框：选择应用服务实例](publish-to-azure-webapp-using-vs/_static/select_as.png)

接下来，你将看到“发布配置文件摘要”页。 Visual Studio 检测到此应用程序需要 SQL Server 数据库，并要求你进行配置。 选择“配置”。

![“发布配置文件摘要”页：配置 SQL Server 依赖项](publish-to-azure-webapp-using-vs/_static/sql.png)

“配置依赖项”对话框随即出现：

* 选择“Azure SQL 数据库”。
* 选择“下一步”。

![“配置 SQL Server 依赖项”对话框](publish-to-azure-webapp-using-vs/_static/sql1.png)

在“配置 Azure SQL 数据库”对话框中选择“创建 SQL 数据库”

![“配置 Azure SQL 数据库”对话框](publish-to-azure-webapp-using-vs/_static/sql2.png)

随即出现“创建 Azure SQL 数据库”：

* “数据库名称”、“资源组”、“数据库服务器”、“应用服务计划”输入字段已填充。 可以保留这些值，也可以进行更改。
* 为所选“数据库服务器”输入“数据库管理员用户名”和“数据库管理员密码”（请注意，你使用的帐户必须具有创建新 Azure SQL 数据库所需的权限）
* 选择“创建”。

![“新建 Azure SQL 数据库”对话框](publish-to-azure-webapp-using-vs/_static/sql_create.png)

创建完成后，对话框将自动关闭，“配置 Azure SQL 数据库”对话框将再次成为焦点：

* 将自动选择刚创建的新实例。
* 选择“下一步”。

![“配置 Azure SQL 数据库”对话框](publish-to-azure-webapp-using-vs/_static/sql_select.png)

在“配置 Azure SQL 数据库”对话框的下一步中：

* 输入“数据库连接用户名”和“数据库连接密码”字段。 应用程序将使用这些详细信息在运行时连接到数据库。 最佳做法是，避免使用与上一步中的管理员用户名和密码相同的详细信息。
* 选择“完成”。

![“配置 Azure SQL 数据库”对话框，连接字符串详细信息](publish-to-azure-webapp-using-vs/_static/sql_connection.png)

在“发布配置文件摘要”页中选择“设置”：

![“发布配置文件摘要”页：编辑设置](publish-to-azure-webapp-using-vs/_static/pp_configured.png)

在“发布”对话框的“设置”页面上 ：

* 展开“数据库”并选中“在运行时使用此连接字符串” 。
* 展开“Entity Framework 迁移”并选中“在发布时应用此迁移” 。

* 选择“保存”。 Visual Studio 将返回到“发布”对话框。 

![“发布”对话框：设置面板](publish-to-azure-webapp-using-vs/_static/pp_settings.png)

单击“发布” 。 Visual Studio 将应用发布到 Azure。 部署完成时，应用在浏览器中打开。

![“发布”对话框：设置面板](publish-to-azure-webapp-using-vs/_static/pp_publish.png)

### <a name="update-the-app"></a>更新应用

* 编辑 Pages/Index.cshtml Razor 页并更改其内容。 例如，可以将段落修改为显示“Hello ASP.NET Core!”：

    [!code-html[Index](publish-to-azure-webapp-using-vs/sample/index.cshtml?highlight=10&range=1-12)]

* 再次选择“发布配置文件摘要”页中的“发布”。

![“发布配置文件摘要”页](publish-to-azure-webapp-using-vs/_static/pp_publish.png)

* 应用发布后，验证所做的更改在 Azure 上是否可用。

![验证任务已完成](publish-to-azure-webapp-using-vs/_static/final.png)

### <a name="clean-up"></a>清理

完成应用测试后，转到 [Azure 门户](https://portal.azure.com/)并删除该应用。

* 选择“资源组”，然后选择所创建的资源组。

![Azure 门户：侧边栏菜单中的资源组](publish-to-azure-webapp-using-vs/_static/portalrg.png)

* 在“资源组”页面中，选择“删除” 。

![Azure 门户：“资源组”页](publish-to-azure-webapp-using-vs/_static/rgd.png)

* 输入资源组的名称并选择“删除”。 现已从 Azure 中删除了本教程中创建的应用和其他所有资源。

### <a name="next-steps"></a>后续步骤

* <xref:host-and-deploy/azure-apps/azure-continuous-deployment>

## <a name="additional-resources"></a>其他资源

* 对于 Visual Studio Code，请参阅[发布配置文件](xref:host-and-deploy/visual-studio-publish-profiles#publish-profiles)。
* [Azure 应用服务](/azure/app-service/app-service-web-overview)
* [Azure 资源组](/azure/azure-resource-manager/resource-group-overview#resource-groups)
* [Azure SQL 数据库](/azure/sql-database/)
* <xref:host-and-deploy/visual-studio-publish-profiles>
* <xref:test/troubleshoot-azure-iis>