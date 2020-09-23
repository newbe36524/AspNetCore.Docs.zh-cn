---
title: ASP.NET Core 中的配置
author: rick-anderson
description: 理解如何使用配置 API 配置 ASP.NET Core 应用。
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 3/29/2020
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
uid: fundamentals/configuration/index
ms.openlocfilehash: c6b77f20792c32a697fff8d1d78ef3c2536c5f8f
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722892"
---
# <a name="configuration-in-aspnet-core"></a>ASP.NET Core 中的配置

作者：[Rick Anderson](https://twitter.com/RickAndMSFT) 和 [Kirk Larkin](https://twitter.com/serpent5)

::: moniker range=">= aspnetcore-3.0"

ASP.NET Core 中的配置是使用一个或多个[配置提供程序](#cp)执行的。 配置提供程序使用各种配置源从键值对读取配置数据：

* 设置文件，例如 appsettings.json
* 环境变量
* Azure Key Vault
* Azure 应用程序配置
* 命令行参数
* 已安装或已创建的自定义提供程序
* 目录文件
* 内存中的 .NET 对象

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples)（[如何下载](xref:index#how-to-download-a-sample)）

<a name="default"></a>

## <a name="default-configuration"></a>默认配置

通过 [dotnet new](/dotnet/core/tools/dotnet-new) 或 Visual Studio 创建的 ASP.NET Core Web 应用会生成以下代码：

[!code-csharp[](index/samples/3.x/ConfigSample/Program.cs?name=snippet&highlight=9)]

 <xref:Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder*> 按照以下顺序为应用提供默认配置：

1. [ChainedConfigurationProvider](xref:Microsoft.Extensions.Configuration.ChainedConfigurationSource)：添加现有 `IConfiguration` 作为源。 在默认配置示例中，添加[主机](#hvac)配置，并将它设置为应用配置的第一个源。
1. 使用 [JSON 配置提供程序](#file-configuration-provider)通过 [appsettings.json](#appsettingsjson) 提供。
1. 使用 [JSON 配置提供程序](#file-configuration-provider)通过 appsettings.`Environment`.json 提供 。 例如，appsettings.Production.json 和 appsettings.Development.json 。
1. 应用在 `Development` 环境中运行时的[应用机密](xref:security/app-secrets)。
1. 使用[环境变量配置提供程序](#evcp)通过环境变量提供。
1. 使用[命令行配置提供程序](#command-line)通过命令行参数提供。

后来添加的配置提供程序会替代之前的密钥设置。 例如，如果在 appsettings.json 和环境中设置了 `MyKey`，则会使用环境值。 使用默认配置提供程序，[命令行配置提供程序](#clcp)将替代所有其他的提供程序。

若要详细了解 `CreateDefaultBuilder`，请参阅[默认生成器设置](xref:fundamentals/host/generic-host#default-builder-settings)。

以下代码按添加顺序显示了已启用的配置提供程序：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Index2.cshtml.cs?name=snippet)]

### <a name="appsettingsjson"></a>appsettings.json

请考虑使用以下 appsettings.json 文件：

[!code-json[](index/samples/3.x/ConfigSample/appsettings.json)]

以下来自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)的代码显示了上述的一些配置设置：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

默认的 <xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> 会按以下顺序加载配置：

1. *appsettings.json*
1. appsettings.`Environment`.json ：例如，appsettings.Production.json 和 appsettings.Development.json  文件。 文件的环境版本是根据 [IHostingEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.EnvironmentName*) 加载的。 有关详细信息，请参阅 <xref:fundamentals/environments>。

appsettings.`Environment`.json 值将替代 appsettings.json 中的密钥  。 例如，默认情况下：

* 在开发环境中，appsettings.Development.json 配置将覆盖在 appsettings.json 中找到的值 。
* 在生产环境中，appsettings.Production.json 配置将覆盖在 appsettings.json 中找到的值 。 例如，在将应用部署到 Azure 时。

<a name="optpat"></a>

### <a name="bind-hierarchical-configuration-data-using-the-options-pattern"></a>使用选项模式绑定分层配置数据

[!INCLUDE[](~/includes/bind.md)]

使用[默认](#default)配置，会通过 [reloadOnChange: true](https://github.com/dotnet/extensions/blob/release/3.1/src/Hosting/Hosting/src/Host.cs#L74-L75) 启用 appsettings.json 和 appsettings.`Environment`.json 文件  。 应用启动后，对 appsettings.json 和 appsettings.`Environment`.json 文件做出的更改将由 [JSON 配置提供程序](#jcp)读取  。

有关添加其他 JSON 配置文件的信息，请参阅本文档中的 [JSON 配置提供程序](#jcp)。

## <a name="combining-service-collection"></a>合并服务集合

[!INCLUDE[](~/includes/combine-di.md)]

<a name="security"></a>

## <a name="security-and-secret-manager"></a>安全和机密管理器

配置数据指南：

* 请勿在配置提供程序代码或纯文本配置文件中存储密码或其他敏感数据。 [机密管理器](xref:security/app-secrets)可用于存储开发环境中的机密。
* 不要在开发或测试环境中使用生产机密。
* 请在项目外部指定机密，避免将其意外提交到源代码存储库。

[默认情况下](#default)，[机密管理器](xref:security/app-secrets)会在 appsettings.json 和 appsettings.`Environment`.json 之后读取配置设置  。

有关存储密码或其他敏感数据的详细信息：

* <xref:fundamentals/environments>
* <xref:security/app-secrets>：包含有关如何使用环境变量来存储敏感数据的建议。 机密管理器使用[文件配置提供程序](#fcp)将用户机密存储在本地系统上的 JSON 文件中。

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 安全存储 ASP.NET Core 应用的应用机密。 有关详细信息，请参阅 <xref:security/key-vault-configuration>。

<a name="evcp"></a>

## <a name="environment-variables"></a>环境变量

使用[默认](#default)配置，<xref:Microsoft.Extensions.Configuration.EnvironmentVariables.EnvironmentVariablesConfigurationProvider> 会在读取 appsettings.json、appsettings.`Environment`.json 和[机密管理器](xref:security/app-secrets)后从环境变量键值对加载配置  。 因此，从环境中读取的键值会替代从 appsettings.json、appsettings.`Environment`.json 和机密管理器中读取的值  。

[!INCLUDE[](~/includes/environmentVarableColon.md)]

以下 `set` 命令：

* 在 Windows 上设置[上述示例](#appsettingsjson)的环境键和值。
* 在使用[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)时测试设置。 `dotnet run` 命令必须在项目目录中运行。

```dotnetcli
set MyKey="My key from Environment"
set Position__Title=Environment_Editor
set Position__Name=Environment_Rick
dotnet run
```

前面的环境设置：

* 仅在进程中设置，这些进程是从设置进程的命令窗口启动的。
* 不会由通过 Visual Studio 启动的浏览器读取。

以下 [setx](/windows-server/administration/windows-commands/setx) 命令可用于在 Windows 上设置环境键和值。 与 `set` 不同，`setx` 设置是持久的。 `/M` 在系统环境中设置变量。 如果未使用 `/M` 开关，则会设置用户环境变量。

```cmd
setx MyKey "My key from setx Environment" /M
setx Position__Title Setx_Environment_Editor /M
setx Position__Name Environment_Rick /M
```

测试前面的命令是否会替代 appsettings.json 和 appsettings.`Environment`.json：

* 使用 Visual Studio：退出并重启 Visual Studio。
* 使用 CLI：启动新的命令窗口并输入 `dotnet run`。

使用字符串调用 <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> 以指定环境变量的前缀：

[!code-csharp[](~/fundamentals/configuration/index/samples/3.x/ConfigSample/Program.cs?name=snippet4&highlight=12)]

在上述代码中：

* `config.AddEnvironmentVariables(prefix: "MyCustomPrefix_")` 被添加到[默认配置提供程序](#default)之后。 有关对配置提供程序进行排序的示例，请参阅 [JSON 配置提供程序](#jcp)。
* 使用 `MyCustomPrefix_` 前缀设置的环境变量将替代[默认配置提供程序](#default)。 这包括没有前缀的环境变量。

前缀会在读取配置键值对时被去除。

以下命令对自定义前缀进行测试：

```dotnetcli
set MyCustomPrefix_MyKey="My key with MyCustomPrefix_ Environment"
set MyCustomPrefix_Position__Title=Editor_with_customPrefix
set MyCustomPrefix_Position__Name=Environment_Rick_cp
dotnet run
```

[默认配置](#default)会加载前缀为 `DOTNET_` 和 `ASPNETCORE_` 的环境变量和命令行参数。 `DOTNET_` 和 `ASPNETCORE_` 前缀会由 ASP.NET Core 用于[主机和应用配置](xref:fundamentals/host/generic-host#host-configuration)，但不用于用户配置。 有关主机和应用配置的详细信息，请参阅 [.NET 通用主机](xref:fundamentals/host/generic-host)。

在 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)上，选择“设置”>“配置”页面上的“新应用程序设置” 。 Azure 应用服务应用程序设置：

* 已静态加密且通过加密的通道进行传输。
* 已作为环境变量公开。

有关详细信息，请参阅 [Azure 应用：使用 Azure 门户替代应用配置](xref:host-and-deploy/azure-apps/index#override-app-configuration-using-the-azure-portal)。

有关 Azure 数据库连接字符串的信息，请参阅[连接字符串前缀](#constr)。

### <a name="environment-variables-set-in-launchsettingsjson"></a>在 launchSettings.json 中设置的环境变量

在 launchSettings.json 中设置的环境变量将替代在系统环境中设置的变量。

<a name="clcp"></a>

## <a name="command-line"></a>命令行

使用[默认](#default)配置，<xref:Microsoft.Extensions.Configuration.CommandLine.CommandLineConfigurationProvider> 会从以下配置源后的命令行参数键值对中加载配置：

* appsettings.json 和 appsettings.`Environment`.json 文件  。
* 开发环境中的[应用机密（机密管理器）](xref:security/app-secrets)。
* 环境变量。

[默认情况下](#default)，在命令行上设置的配置值会替代通过所有其他配置提供程序设置的配置值。

### <a name="command-line-arguments"></a>命令行参数

以下命令使用 `=` 设置键和值：

```dotnetcli
dotnet run MyKey="My key from command line" Position:Title=Cmd Position:Name=Cmd_Rick
```

以下命令使用 `/` 设置键和值：

```dotnetcli
dotnet run /MyKey "Using /" /Position:Title=Cmd_ /Position:Name=Cmd_Rick
```

以下命令使用 `--` 设置键和值：

```dotnetcli
dotnet run --MyKey "Using --" --Position:Title=Cmd-- --Position:Name=Cmd--Rick
```

键值：

* 必须后跟 `=`，或者当值后跟一个空格时，键必须具有一个 `--` 或 `/` 的前缀。
* 如果使用 `=`，则不是必需的。 例如 `MySetting=`。

在同一命令中，请勿将使用 `=` 的命令行参数键值对与使用空格的键值对混合使用。

### <a name="switch-mappings"></a>交换映射

交换映射支持键名替换逻辑。 提供针对 <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> 方法的交换替换字典。

当使用交换映射字典时，会检查字典中是否有与命令行参数提供的键匹配的键。 如果在字典中找到了命令行键，则会传回字典值将键值对设置为应用的配置。 对任何具有单划线 (`-`) 前缀的命令行键而言，交换映射都是必需的。

交换映射字典键规则：

* 交换必须以 `-` 或 `--` 开头。
* 交换映射字典不得包含重复键。

若要使用交换映射字典，请将它传递到对 `AddCommandLine` 的调用中：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramSwitch.cs?name=snippet&highlight=10-18,23)]

下面的代码显示了替换后的键的键值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test3.cshtml.cs?name=snippet)]

以下命令可用于测试键替换：

```dotnetcli
dotnet run -k1 value1 -k2 value2 --alt3=value2 /alt4=value3 --alt5 value5 /alt6 value6
```

对于使用交换映射的应用，调用 `CreateDefaultBuilder` 不应传递参数。 `CreateDefaultBuilder` 方法的 `AddCommandLine` 调用不包括映射的交换，并且无法将交换映射字典传递给 `CreateDefaultBuilder`。 解决方案不是将参数传递给 `CreateDefaultBuilder`，而是允许 `ConfigurationBuilder` 方法的 `AddCommandLine` 方法处理参数和交换映射字典。

## <a name="hierarchical-configuration-data"></a>分层配置数据

配置 API 在配置键中使用分隔符来展平分层数据，以此来读取分层配置数据。

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)包含以下 appsettings.json 文件：

[!code-json[](index/samples/3.x/ConfigSample/appsettings.json)]

以下来自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)的代码显示了一些配置设置：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

读取分层配置数据的首选方法是使用选项模式。 有关详细信息，请参阅本文档中的[绑定分层配置数据](#optpat)。

<xref:Microsoft.Extensions.Configuration.ConfigurationSection.GetSection*> 和 <xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*> 方法可用于隔离各个节和配置数据中某节的子节。 稍后将在 [GetSection、GetChildren 和 Exists](#getsection) 中介绍这些方法。

<!--
[Azure Key Vault configuration provider](xref:security/key-vault-configuration) implement change detection.
-->

## <a name="configuration-keys-and-values"></a>配置键和值

配置键：

* 不区分大小写。 例如，`ConnectionString` 和 `connectionstring` 被视为等效键。
* 如果在多个配置提供程序中设置了某一键和值，则会使用最后添加的提供程序中的值。 有关详细信息，请参阅[默认配置](#default)。
* 分层键
  * 在配置 API 中，冒号分隔符 (`:`) 适用于所有平台。
  * 在环境变量中，冒号分隔符可能无法适用于所有平台。 所有平台均支持采用双下划线 `__`，并且它会自动转换为冒号 `:`。
  * 在 Azure Key Vault 中，分层键使用 `--` 作为分隔符。 当机密加载到应用的配置中时，[Azure Key Vault 配置提供程序](xref:security/key-vault-configuration) 会自动将 `--` 替换为 `:`。
* <xref:Microsoft.Extensions.Configuration.ConfigurationBinder> 支持使用配置键中的数组索引将数组绑定到对象。 数组绑定将在[将数组绑定到类](#boa)部分中进行介绍。

配置值：

* 为字符串。
* NULL 值不能存储在配置中或绑定到对象。

<a name="cp"></a>

## <a name="configuration-providers"></a>配置提供程序

下表显示了 ASP.NET Core 应用可用的配置提供程序。

| 提供程序 | 通过以下对象提供配置 |
| -------- | ----------------------------------- |
| [Azure Key Vault 配置提供程序](xref:security/key-vault-configuration) | Azure Key Vault |
| [Azure 应用配置提供程序](/azure/azure-app-configuration/quickstart-aspnet-core-app) | Azure 应用程序配置 |
| [命令行配置提供程序](#clcp) | 命令行参数 |
| [自定义配置提供程序](#custom-configuration-provider) | 自定义源 |
| [环境变量配置提供程序](#evcp) | 环境变量 |
| [文件配置提供程序](#file-configuration-provider) | INI、JSON 和 XML 文件 |
| [Key-per-file 配置提供程序](#key-per-file-configuration-provider) | 目录文件 |
| [内存配置提供程序](#memory-configuration-provider) | 内存中集合 |
| [机密管理器](xref:security/app-secrets)  | 用户配置文件目录中的文件 |

按照指定的配置提供程序的顺序读取配置源。 代码中的配置提供程序应以特定顺序排列，从而满足应用所需的基础配置源的优先级。

配置提供程序的典型顺序为：

1. *appsettings.json*
1. appsettings.`Environment`.json
1. [机密管理器](xref:security/app-secrets)
1. 使用[环境变量配置提供程序](#evcp)通过环境变量提供。
1. 使用[命令行配置提供程序](#command-line-configuration-provider)通过命令行参数提供。

通常的做法是将命令行配置提供程序添加到一系列提供程序的末尾，使命令行参数能够替代由其他提供程序设置的配置。

[默认配置](#default)中使用了上述提供程序顺序。

<a name="constr"></a>

### <a name="connection-string-prefixes"></a>连接字符串前缀

对于四个连接字符串环境变量，配置 API 具有特殊的处理规则。 这些连接字符串涉及了为应用环境配置 Azure 连接字符串。 使用[默认配置](#default)或没有向 `AddEnvironmentVariables` 应用前缀时，具有表中所示前缀的环境变量将加载到应用中。

| 连接字符串前缀 | 提供程序 |
| ------------------------ | -------- |
| `CUSTOMCONNSTR_` | 自定义提供程序 |
| `MYSQLCONNSTR_` | [MySQL](https://www.mysql.com/) |
| `SQLAZURECONNSTR_` | [Azure SQL 数据库](https://azure.microsoft.com/services/sql-database/) |
| `SQLCONNSTR_` | [SQL Server](https://www.microsoft.com/sql-server/) |

当发现环境变量并使用表中所示的四个前缀中的任何一个加载到配置中时：

* 通过删除环境变量前缀并添加配置键节 (`ConnectionStrings`) 来创建配置键。
* 创建一个新的配置键值对，表示数据库连接提供程序（`CUSTOMCONNSTR_` 除外，它没有声明的提供程序）。

| 环境变量键 | 转换的配置键 | 提供程序配置条目                                                    |
| ------------------------ | --------------------------- | ------------------------------------------------------------------------------- |
| `CUSTOMCONNSTR_{KEY} `   | `ConnectionStrings:{KEY}`   | 配置条目未创建。                                                |
| `MYSQLCONNSTR_{KEY}`     | `ConnectionStrings:{KEY}`   | 键：`ConnectionStrings:{KEY}_ProviderName`：<br>值：`MySql.Data.MySqlClient` |
| `SQLAZURECONNSTR_{KEY}`  | `ConnectionStrings:{KEY}`   | 键：`ConnectionStrings:{KEY}_ProviderName`：<br>值：`System.Data.SqlClient`  |
| `SQLCONNSTR_{KEY}`       | `ConnectionStrings:{KEY}`   | 键：`ConnectionStrings:{KEY}_ProviderName`：<br>值：`System.Data.SqlClient`  |

<a name="fcp"></a>

## <a name="file-configuration-provider"></a>文件配置提供程序

<xref:Microsoft.Extensions.Configuration.FileConfigurationProvider> 是从文件系统加载配置的基类。 以下配置提供程序派生自 `FileConfigurationProvider`：

* [INI 配置提供程序](#ini-configuration-provider)
* [JSON 配置提供程序](#jcp)
* [XML 配置提供程序](#xml-configuration-provider)

### <a name="ini-configuration-provider"></a>INI 配置提供程序

<xref:Microsoft.Extensions.Configuration.Ini.IniConfigurationProvider> 在运行时从 INI 文件键值对加载配置。

以下代码会清除所有配置提供程序并添加多个配置提供程序：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramINI.cs?name=snippet&highlight=10-30)]

在前面的代码中，MyIniConfig.ini 和 MyIniConfig.`Environment`.ini 文件中的设置会被以下提供程序中的设置替代  ：

* [环境变量配置提供程序](#evcp)
* [命令行配置提供程序](#clcp)。

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)包含以下 MyIniConfig.ini 文件：

[!code-ini[](index/samples/3.x/ConfigSample/MyIniConfig.ini)]

以下来自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)的代码显示了上述的一些配置设置：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

<a name="jcp"></a>

### <a name="json-configuration-provider"></a>JSON 配置提供程序

<xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> 从 JSON 文件键值对加载配置。

重载可以指定：

* 文件是否可选。
* 如果文件更改，是否重载配置。

考虑下列代码：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSON.cs?name=snippet&highlight=12-14)]

前面的代码：

* 通过以下选项将 JSON 配置提供程序配置为加载 MyConfig.json 文件：
  * `optional: true`：文件是可选的。
  * `reloadOnChange: true`：保存更改后会重载文件。
* 读取 MyConfig.json 文件之前的[默认配置提供程序](#default)。 MyConfig.json 文件中的设置会替代默认配置提供程序中的设置，包括[环境变量配置提供程序](#evcp)和[命令行配置提供程序](#clcp)。

通常情况下，你不会希望自定义 JSON 文件替代在[环境变量配置提供程序](#evcp)和[命令行配置提供程序](#clcp)中设置的值。

以下代码会清除所有配置提供程序并添加多个配置提供程序：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSON2.cs?name=snippet)]

在前面的代码中，MyConfig.json 和 MyConfig.`Environment`.json 文件中的设置  ：

* 会替代 appsettings.json 和 appsettings.`Environment`.json 文件中的设置  。
* 会被[环境变量配置提供程序](#evcp)和[命令行配置提供程序](#clcp)中的设置所替代。

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)包含以下 MyConfig.json 文件：

[!code-json[](index/samples/3.x/ConfigSample/MyConfig.json)]

以下来自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)的代码显示了上述的一些配置设置：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

### <a name="xml-configuration-provider"></a>XML 配置提供程序

<xref:Microsoft.Extensions.Configuration.Xml.XmlConfigurationProvider> 在运行时从 XML 文件键值对加载配置。

以下代码会清除所有配置提供程序并添加多个配置提供程序：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramXML.cs?name=snippet)]

在前面的代码中，MyXMLFile.xml 和 MyXMLFile.`Environment`.xml 文件中的设置会被以下提供程序中的设置替代  ：

* [环境变量配置提供程序](#evcp)
* [命令行配置提供程序](#clcp)。

[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)包含以下 MyXMLFile.xml 文件：

[!code-xml[](index/samples/3.x/ConfigSample/MyXMLFile.xml)]

以下来自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)的代码显示了上述的一些配置设置：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

如果使用 `name` 属性来区分元素，则使用相同元素名称的重复元素可以正常工作：

[!code-xml[](index/samples/3.x/ConfigSample/MyXMLFile3.xml)]

以下代码会读取前面的配置文件并显示键和值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/XML/Index.cshtml.cs?name=snippet)]

属性可用于提供值：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <key attribute="value" />
  <section>
    <key attribute="value" />
  </section>
</configuration>
```

以前的配置文件使用 `value` 加载以下键：

* key:attribute
* section:key:attribute

## <a name="key-per-file-configuration-provider"></a>Key-per-file 配置提供程序

<xref:Microsoft.Extensions.Configuration.KeyPerFile.KeyPerFileConfigurationProvider> 使用目录的文件作为配置键值对。 该键是文件名。 该值包含文件的内容。 Key-per-file 配置提供程序用于 Docker 托管方案。

若要激活 Key-per-file 配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.KeyPerFileConfigurationBuilderExtensions.AddKeyPerFile*> 扩展方法。 文件的 `directoryPath` 必须是绝对路径。

重载允许指定：

* 配置源的 `Action<KeyPerFileConfigurationSource>` 委托。
* 目录是否可选以及目录的路径。

双下划线字符 (`__`) 用作文件名中的配置键分隔符。 例如，文件名 `Logging__LogLevel__System` 生成配置键 `Logging:LogLevel:System`。

构建主机时调用 `ConfigureAppConfiguration` 以指定应用的配置：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    var path = Path.Combine(
        Directory.GetCurrentDirectory(), "path/to/files");
    config.AddKeyPerFile(directoryPath: path, optional: true);
})
```

<a name="mcp"></a>

## <a name="memory-configuration-provider"></a>内存配置提供程序

<xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationProvider> 使用内存中集合作为配置键值对。

以下代码将内存集合添加到配置系统中：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet6)]

以下来自[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)的代码显示了上述配置设置：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Test.cshtml.cs?name=snippet)]

在前面的代码中，`config.AddInMemoryCollection(Dict)` 会被添加到[默认配置提供程序](#default)之后。 有关对配置提供程序进行排序的示例，请参阅 [JSON 配置提供程序](#jcp)。

有关使用 `MemoryConfigurationProvider` 的其他示例，请参阅[绑定数组](#boa)。

## <a name="getvalue"></a>GetValue

[`ConfigurationBinder.GetValue<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue*) 从配置中提取一个具有指定键的值，并将它转换为指定的类型：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestNum.cshtml.cs?name=snippet)]

在前面的代码中，如果在配置中找不到 `NumberKey`，则使用默认值 `99`。

## <a name="getsection-getchildren-and-exists"></a>GetSection、GetChildren 和 Exists

对于下面的示例，请考虑以下 MySubsection.json 文件：

[!code-json[](index/samples/3.x/ConfigSample/MySubsection.json)]

以下代码将 MySubsection.json 添加到配置提供程序：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSONsection.cs?name=snippet)]

### <a name="getsection"></a>GetSection

[IConfiguration.GetSection](xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection*) 会返回具有指定子节键的配置子节。

以下代码将返回 `section1` 的值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection.cshtml.cs?name=snippet)]

以下代码将返回 `section2:subsection0` 的值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection2.cshtml.cs?name=snippet)]

`GetSection` 永远不会返回 `null`。 如果找不到匹配的节，则返回空 `IConfigurationSection`。

当 `GetSection` 返回匹配的部分时，<xref:Microsoft.Extensions.Configuration.IConfigurationSection.Value> 未填充。 存在该部分时，返回一个 <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Key> 和 <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Path> 部分。

### <a name="getchildren-and-exists"></a>GetChildren 和 Exists

以下代码将调用 [IConfiguration.GetChildren](xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*) 并返回 `section2:subsection0` 的值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/TestSection4.cshtml.cs?name=snippet)]

前面的代码将调用 [ConfigurationExtensions.Exists](xref:Microsoft.Extensions.Configuration.ConfigurationExtensions.Exists*) 以验证该节是否存在：

 <a name="boa"></a>

## <a name="bind-an-array"></a>绑定数组

[ConfigurationBinder.Bind](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*) 支持使用配置键中的数组索引将数组绑定到对象。 公开数值键段的任何数组格式都能够与 [POCO](https://wikipedia.org/wiki/Plain_Old_CLR_Object) 类数组进行数组绑定。

请考虑[示例下载](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples/3.x/ConfigSample)中的 MyArray.json：

[!code-json[](index/samples/3.x/ConfigSample/MyArray.json)]

以下代码将 MyArray.json 添加到配置提供程序：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramJSONarray.cs?name=snippet)]

以下代码将读取配置并显示值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

前面的代码会返回以下输出：

```text
Index: 0  Value: value00
Index: 1  Value: value10
Index: 2  Value: value20
Index: 3  Value: value40
Index: 4  Value: value50
```

在前面的输出中，索引 3 具有值 `value40`，与 MyArray.json 中的 `"4": "value40",` 相对应。 绑定的数组索引是连续的，并且未绑定到配置键索引。 配置绑定器不能绑定 NULL 值，也不能在绑定的对象中创建 NULL 条目

以下代码将通过 <xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*> 扩展方法加载 `array:entries` 配置：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet)]

以下代码将读取 `arrayDict` `Dictionary` 中的配置并显示值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

前面的代码会返回以下输出：

```text
Index: 0  Value: value0
Index: 1  Value: value1
Index: 2  Value: value2
Index: 3  Value: value4
Index: 4  Value: value5
```

绑定对象中的索引 &num;3 保留 `array:4` 配置键的配置数据及其值 `value4`。 当绑定包含数组的配置数据时，配置键中的数组索引用于在创建对象时迭代配置数据。 无法在配置数据中保留 null 值，并且当配置键中的数组跳过一个或多个索引时，不会在绑定对象中创建 null 值条目。

可以在由任何读取索引 &num;3 键/值对的配置提供程序绑定到 `ArrayExample` 实例之前提供索引 &num;3 的缺失配置项。 请考虑示例下载中的以下 Value3.json 文件：

[!code-json[](index/samples/3.x/ConfigSample/Value3.json)]

以下代码包含 Value3.json 和 `arrayDict` `Dictionary` 的配置：

[!code-csharp[](index/samples/3.x/ConfigSample/ProgramArray.cs?name=snippet2)]

以下代码将读取上述配置并显示值：

[!code-csharp[](index/samples/3.x/ConfigSample/Pages/Array.cshtml.cs?name=snippet)]

前面的代码会返回以下输出：

```text
Index: 0  Value: value0
Index: 1  Value: value1
Index: 2  Value: value2
Index: 3  Value: value3
Index: 4  Value: value4
Index: 5  Value: value5
```

不需要自定义配置提供程序实现数组绑定。

## <a name="custom-configuration-provider"></a>自定义配置提供程序

该示例应用演示了如何使用[实体框架 (EF)](/ef/core/) 创建从数据库读取配置键值对的基本配置提供程序。

提供程序具有以下特征：

* EF 内存中数据库用于演示目的。 若要使用需要连接字符串的数据库，请实现辅助 `ConfigurationBuilder` 以从另一个配置提供程序提供连接字符串。
* 提供程序在启动时将数据库表读入配置。 提供程序不会基于每个键查询数据库。
* 未实现更改时重载，因此在应用启动后更新数据库对应用的配置没有任何影响。

定义用于在数据库中存储配置值的 `EFConfigurationValue` 实体。

*Models/EFConfigurationValue.cs*：

[!code-csharp[](index/samples/3.x/ConfigurationSample/Models/EFConfigurationValue.cs?name=snippet1)]

添加 `EFConfigurationContext` 以存储和访问配置的值。

*EFConfigurationProvider/EFConfigurationContext.cs*：

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationContext.cs?name=snippet1)]

创建用于实现 <xref:Microsoft.Extensions.Configuration.IConfigurationSource> 的类。

*EFConfigurationProvider/EFConfigurationSource.cs*：

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationSource.cs?name=snippet1)]

通过从 <xref:Microsoft.Extensions.Configuration.ConfigurationProvider> 继承来创建自定义配置提供程序。 当数据库为空时，配置提供程序将对其进行初始化。 由于[配置密钥不区分大小写](#keys)，因此用来初始化数据库的字典是用不区分大小写的比较程序 ([StringComparer.OrdinalIgnoreCase](xref:System.StringComparer.OrdinalIgnoreCase)) 创建的。

*EFConfigurationProvider/EFConfigurationProvider.cs*：

[!code-csharp[](index/samples/3.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationProvider.cs?name=snippet1)]

可以使用 `AddEFConfiguration` 扩展方法将配置源添加到 `ConfigurationBuilder`。

Extensions/EntityFrameworkExtensions.cs：

[!code-csharp[](index/samples/3.x/ConfigurationSample/Extensions/EntityFrameworkExtensions.cs?name=snippet1)]

下面的代码演示如何在 Program.cs 中使用自定义的 `EFConfigurationProvider`：

[!code-csharp[](index/samples_snippets/3.x/ConfigurationSample/Program.cs?highlight=7-8)]

<a name="acs"></a>

## <a name="access-configuration-in-startup"></a>访问 Startup 中的配置

以下代码显示 `Startup` 方法中的配置数据：

[!code-csharp[](index/samples/3.x/ConfigSample/StartupKey.cs?name=snippet&highlight=13,18)]

有关使用启动便捷方法访问配置的示例，请参阅[应用启动：便捷方法](xref:fundamentals/startup#convenience-methods)。

## <a name="access-configuration-in-no-locrazor-pages"></a>访问 Razor Pages 中的配置

以下代码显示 Razor Pages 中的配置数据：

[!code-cshtml[](index/samples/3.x/ConfigSample/Pages/Test5.cshtml)]

在以下代码中，`MyOptions` 已通过 <xref:Microsoft.Extensions.DependencyInjection.OptionsConfigurationServiceCollectionExtensions.Configure*> 被添加到了服务容器并已绑定到了配置：

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup3.cs?name=snippet_Example2)]

以下标记使用 [`@inject`](xref:mvc/views/razor#inject) Razor 指令来解析和显示选项值：

[!code-cshtml[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Pages/Test3.cshtml)]

## <a name="access-configuration-in-a-mvc-view-file"></a>访问 MVC 视图文件中的配置

以下代码显示 MVC 视图中的配置数据：

[!code-cshtml[](index/samples/3.x/ConfigSample/Views/Home2/Index.cshtml)]

## <a name="configure-options-with-a-delegate"></a>使用委托来配置选项

在委托中配置的选项替代在配置提供程序中设置的值。

示例应用中的示例 2 展示了如何使用委托来配置选项。

在以下代码中，向服务容器添加了 <xref:Microsoft.Extensions.Options.IConfigureOptions%601> 服务。 它使用委托来配置 `MyOptions` 的值：

[!code-csharp[](~/fundamentals/configuration/options/samples/3.x/OptionsSample/Startup2.cs?name=snippet_Example2)]

以下代码显示选项值：

[!code-csharp[](options/samples/3.x/OptionsSample/Pages/Test2.cshtml.cs?name=snippet)]

在前面的示例中，`Option1` 和 `Option2` 的值在 appsettings.json 中指定，然后被配置的委托替代。

<a name="hvac"></a>

## <a name="host-versus-app-configuration"></a>主机与应用配置

在配置并启动应用之前，配置并启动主机。 主机负责应用程序启动和生存期管理。 应用和主机均使用本主题中所述的配置提供程序进行配置。 应用的配置中也包含主机配置键值对。 有关在构建主机时如何使用配置提供程序以及配置源如何影响主机配置的详细信息，请参阅 <xref:fundamentals/index#host>。

<a name="dhc"></a>

## <a name="default-host-configuration"></a>默认主机配置

有关使用 [Web 主机](xref:fundamentals/host/web-host)时默认配置的详细信息，请参阅[本主题的 ASP.NET Core 2.2 版本](?view=aspnetcore-2.2)。

* 主机配置通过以下方式提供：
  * 使用[环境变量配置提供程序](#environment-variables)通过前缀为 `DOTNET_`的环境变量（例如，`DOTNET_ENVIRONMENT`）提供。 在配置键值对加载后，前缀 (`DOTNET_`) 会遭去除。
  * 使用[命令行配置提供程序](#command-line-configuration-provider)通过命令行参数提供。
* 已建立 Web 主机默认配置 (`ConfigureWebHostDefaults`)：
  * Kestrel 用作 Web 服务器，并使用应用的配置提供程序对其进行配置。
  * 添加主机筛选中间件。
  * 如果 `ASPNETCORE_FORWARDEDHEADERS_ENABLED` 环境变量设置为 `true`，则添加转发的标头中间件。
  * 启用 IIS 集成。

## <a name="other-configuration"></a>其他配置

本主题仅与应用配置相关。 运行和托管 ASP.NET Core 应用的其他方面是使用本主题中未包含的配置文件进行配置：

* launch.json/launchSettings.json 是用于开发环境的工具配置文件，如
  * <xref:fundamentals/environments#development> 中所述。
  * 整个文档集中的文件用于为开发方案配置 ASP.NET Core 应用。
* web.config 是服务器配置文件，如以下主题中所述：
  * <xref:host-and-deploy/iis/index>
  * <xref:host-and-deploy/aspnet-core-module>

在 launchSettings.json 中设置的环境变量将替代在系统环境中设置的变量。

若要详细了解如何从旧版 ASP.NET 迁移应用配置，请参阅 <xref:migration/proper-to-2x/index#store-configurations>。

## <a name="add-configuration-from-an-external-assembly"></a>从外部程序集添加配置

通过 <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> 实现，可在启动时从应用 `Startup` 类之外的外部程序集向应用添加增强功能。 有关详细信息，请参阅 <xref:fundamentals/configuration/platform-specific-configuration>。

## <a name="additional-resources"></a>其他资源

* [配置源代码](https://github.com/dotnet/extensions/tree/master/src/Configuration)
* <xref:fundamentals/configuration/options>
* <xref:blazor/fundamentals/configuration>

::: moniker-end

::: moniker range="< aspnetcore-3.0"

ASP.NET Core 中的应用配置基于配置提供程序建立的键值对。 配置提供程序将配置数据从各种配置源读取到键值对：

* Azure Key Vault
* Azure 应用程序配置
* 命令行参数
* （已安装或已创建的）自定义提供程序
* 目录文件
* 环境变量
* 内存中的 .NET 对象
* 设置文件

[Microsoft.AspNetCore.App metapackage](xref:fundamentals/metapackage-app) 中包含通用配置提供程序方案的配置包 ([Microsoft Extensions.Configuration](https://www.nuget.org/packages/Microsoft.Extensions.Configuration/))。

后面的代码示例和示例应用中的代码示例使用 <xref:Microsoft.Extensions.Configuration> 命名空间：

```csharp
using Microsoft.Extensions.Configuration;
```

选项模式是本主题中描述的配置概念的扩展。 选项使用类来表示相关设置的组。 有关详细信息，请参阅 <xref:fundamentals/configuration/options>。

[查看或下载示例代码](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/fundamentals/configuration/index/samples)（[如何下载](xref:index#how-to-download-a-sample)）

## <a name="host-versus-app-configuration"></a>主机与应用配置

在配置并启动应用之前，配置并启动主机。 主机负责应用程序启动和生存期管理。 应用和主机均使用本主题中所述的配置提供程序进行配置。 应用的配置中也包含主机配置键值对。 有关在构建主机时如何使用配置提供程序以及配置源如何影响主机配置的详细信息，请参阅 <xref:fundamentals/index#host>。

## <a name="other-configuration"></a>其他配置

本主题仅与应用配置相关。 运行和托管 ASP.NET Core 应用的其他方面是使用本主题中未包含的配置文件进行配置：

* launch.json/launchSettings.json 是用于开发环境的工具配置文件，如
  * <xref:fundamentals/environments#development> 中所述。
  * 整个文档集中的文件用于为开发方案配置 ASP.NET Core 应用。
* web.config 是服务器配置文件，如以下主题中所述：
  * <xref:host-and-deploy/iis/index>
  * <xref:host-and-deploy/aspnet-core-module>

若要详细了解如何从旧版 ASP.NET 迁移应用配置，请参阅 <xref:migration/proper-to-2x/index#store-configurations>。

## <a name="default-configuration"></a>默认配置

基于 ASP.NET Core [dotnet new](/dotnet/core/tools/dotnet-new)模板的 Web 应用在生成主机时会调用 <xref:Microsoft.AspNetCore.WebHost.CreateDefaultBuilder*>。 `CreateDefaultBuilder` 按照以下顺序为应用提供默认配置：

以下内容适用于使用 [Web 主机](xref:fundamentals/host/web-host)的应用。 有关使用[通用主机](xref:fundamentals/host/generic-host)时默认配置的详细信息，请参阅[本主题的最新版本](xref:fundamentals/configuration/index)。

* 主机配置通过以下方式提供：
  * 使用[环境变量配置提供程序](#environment-variables-configuration-provider)，通过前缀为 `ASPNETCORE_`（例如，`ASPNETCORE_ENVIRONMENT`）的环境变量提供。 在配置键值对加载后，前缀 (`ASPNETCORE_`) 会遭去除。
  * 使用 [ 命令行配置提供程序](#command-line-configuration-provider)，通过命令行参数提供。
* 应用配置通过以下方式提供：
  * 使用[文件配置提供程序](#file-configuration-provider)，通过 appsettings.json 提供。
  * 使用[文件配置提供程序](#file-configuration-provider)，通过 appsettings.{Environment}.json 提供。
  * 应用在使用入口程序集的 `Development` 环境中运行时的[机密管理器](xref:security/app-secrets)。
  * 使用 [ 环境变量配置提供程序](#environment-variables-configuration-provider)，通过环境变量提供。
  * 使用 [ 命令行配置提供程序](#command-line-configuration-provider)，通过命令行参数提供。

## <a name="security"></a>安全性

采用以下做法来保护敏感配置数据：

* 请勿在配置提供程序代码或纯文本配置文件中存储密码或其他敏感数据。
* 不要在开发或测试环境中使用生产机密。
* 请在项目外部指定机密，避免将其意外提交到源代码存储库。

有关详细信息，请参阅下列主题：

* <xref:fundamentals/environments>
* <xref:security/app-secrets>：包含有关如何使用环境变量来存储敏感数据的建议。 Secret Manager 使用文件配置提供程序将用户机密存储在本地系统上的 JSON 文件中。 本主题后面将介绍文件配置提供程序。

[Azure Key Vault](https://azure.microsoft.com/services/key-vault/) 安全存储 ASP.NET Core 应用的应用机密。 有关详细信息，请参阅 <xref:security/key-vault-configuration>。

## <a name="hierarchical-configuration-data"></a>分层配置数据

配置 API 能够通过在配置键中使用分隔符来展平分层数据以保持分层配置数据。

在以下 JSON 文件中，两个节的结构化层次结构中存在四个键：

```json
{
  "section0": {
    "key0": "value",
    "key1": "value"
  },
  "section1": {
    "key0": "value",
    "key1": "value"
  }
}
```

将文件读入配置时，将创建唯一键以保持配置源的原始分层数据结构。 使用冒号 (`:`) 展平节和键以保持原始结构：

* section0:key0
* section0:key1
* section1:key0
* section1:key1

<xref:Microsoft.Extensions.Configuration.ConfigurationSection.GetSection*> 和 <xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*> 方法可用于隔离各个节和配置数据中某节的子节。 稍后将在 [GetSection、GetChildren 和 Exists](#getsection-getchildren-and-exists) 中介绍这些方法。

## <a name="conventions"></a>约定

### <a name="sources-and-providers"></a>源和提供程序

在应用启动时，将按照指定的配置提供程序的顺序读取配置源。

实现更改检测的配置提供程序能够在基础设置更改时重新加载配置。 例如，文件配置提供程序（本主题后面将对此进行介绍）和 [Azure Key Vault 配置提供程序](xref:security/key-vault-configuration)实现更改检测。

应用的[依赖关系注入 (DI)](xref:fundamentals/dependency-injection) 容器中提供了 <xref:Microsoft.Extensions.Configuration.IConfiguration>。 <xref:Microsoft.Extensions.Configuration.IConfiguration> 可注入到 Razor Pages <xref:Microsoft.AspNetCore.Mvc.RazorPages.PageModel> 或 MVC <xref:Microsoft.AspNetCore.Mvc.Controller> 中，以获取类的配置。

在下面的示例中，使用 `_config` 字段来访问配置值：

```csharp
public class IndexModel : PageModel
{
    private readonly IConfiguration _config;

    public IndexModel(IConfiguration config)
    {
        _config = config;
    }
}
```

```csharp
public class HomeController : Controller
{
    private readonly IConfiguration _config;

    public HomeController(IConfiguration config)
    {
        _config = config;
    }
}
```

配置提供程序不能使用 DI，因为主机在设置这些提供程序时 DI 不可用。

### <a name="keys"></a>键

配置键采用以下约定：

* 键不区分大小写。 例如，`ConnectionString` 和 `connectionstring` 被视为等效键。
* 如果由相同或不同的配置提供程序设置相同键的值，则键上设置的最后一个值就是所使用的值。 要详细了解重复的 JSON 密钥，请参阅[此 GitHub 问题](https://github.com/dotnet/extensions/issues/2381)。
* 分层键
  * 在配置 API 中，冒号分隔符 (`:`) 适用于所有平台。
  * 在环境变量中，冒号分隔符可能无法适用于所有平台。 所有平台均支持采用双下划线 (`__`)，并可以将其自动转换为冒号。
  * 在 Azure Key Vault 中，分层键使用 `--`（两个破折号）作为分隔符。 将机密加载到应用的配置中时，用冒号替换破折号。
* <xref:Microsoft.Extensions.Configuration.ConfigurationBinder> 支持使用配置键中的数组索引将数组绑定到对象。 数组绑定将在[将数组绑定到类](#bind-an-array-to-a-class)部分中进行介绍。

### <a name="values"></a>值

配置值采用以下约定：

* 值是字符串。
* NULL 值不能存储在配置中或绑定到对象。

## <a name="providers"></a>提供程序

下表显示了 ASP.NET Core 应用可用的配置提供程序。

| 提供程序 | 通过以下对象提供配置&hellip; |
| -------- | ----------------------------------- |
| [Azure Key Vault 配置提供程序](xref:security/key-vault-configuration)（安全主题） | Azure Key Vault |
| [Azure 应用程序配置提供程序](/azure/azure-app-configuration/quickstart-aspnet-core-app)（Azure 文档） | Azure 应用程序配置 |
| [命令行配置提供程序](#command-line-configuration-provider) | 命令行参数 |
| [自定义配置提供程序](#custom-configuration-provider) | 自定义源 |
| [环境变量配置提供程序](#environment-variables-configuration-provider) | 环境变量 |
| [文件配置提供程序](#file-configuration-provider) | 文件（INI、JSON、XML） |
| [Key-per-file 配置提供程序](#key-per-file-configuration-provider) | 目录文件 |
| [内存配置提供程序](#memory-configuration-provider) | 内存中集合 |
| [用户机密 (Secret Manager)](xref:security/app-secrets)（安全主题） | 用户配置文件目录中的文件 |

按照启动时指定的配置提供程序的顺序读取配置源。 本主题中所述的配置提供程序按字母顺序进行介绍，而不是按代码排列顺序进行介绍。 代码中的配置提供程序应以特定顺序排列，从而满足应用所需的基础配置源的优先级。

配置提供程序的典型顺序为：

1. 文件（appsettings.json、appsettings.{Environment}.json，其中 `{Environment}` 是应用的当前托管环境）
1. [Azure 密钥保管库](xref:security/key-vault-configuration)
1. [用户机密 (Secret Manager)](xref:security/app-secrets)（仅限开发环境中）
1. 环境变量
1. 命令行参数

通常的做法是将命令行配置提供程序置于一系列提供程序的末尾，以允许命令行参数替代由其他提供程序设置的配置。

使用 `CreateDefaultBuilder` 初始化新的主机生成器时，将使用上述提供程序序列。 有关详细信息，请参阅[默认配置](#default-configuration)部分。

## <a name="configure-the-host-builder-with-useconfiguration"></a>用 UseConfiguration 配置主机生成器

若要配置主机生成器，请使用配置在主机生成器上调用 <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseConfiguration*>。

```csharp
public static IWebHostBuilder CreateWebHostBuilder(string[] args)
{
    var dict = new Dictionary<string, string>
    {
        {"MemoryCollectionKey1", "value1"},
        {"MemoryCollectionKey2", "value2"}
    };

    var config = new ConfigurationBuilder()
        .AddInMemoryCollection(dict)
        .Build();

    return WebHost.CreateDefaultBuilder(args)
        .UseConfiguration(config)
        .UseStartup<Startup>();
}
```

## <a name="configureappconfiguration"></a>ConfigureAppConfiguration

构建主机时调用 `ConfigureAppConfiguration` 以指定应用的配置提供程序以及 `CreateDefaultBuilder` 自动添加的配置提供程序：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=20)]

### <a name="override-previous-configuration-with-command-line-arguments"></a>用命令行参数替代以前的配置

若要提供命令行参数可替代的应用配置，最后请调用 `AddCommandLine`：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Call other providers here
    config.AddCommandLine(args);
})
```

### <a name="remove-providers-added-by-createdefaultbuilder"></a>删除 CreateDefaultBuilder 添加的提供程序

要删除 `CreateDefaultBuilder` 添加的提供程序，请先对 [IConfigurationBuilder.Sources](xref:Microsoft.Extensions.Configuration.IConfigurationBuilder.Sources) 调用 [Clear](/dotnet/api/system.collections.generic.icollection-1.clear)：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.Sources.Clear();
    // Add providers here
})
```

### <a name="consume-configuration-during-app-startup"></a>在应用启动期间使用配置

在应用启动期间，可以使用 `ConfigureAppConfiguration` 中提供给应用的配置，包括 `Startup.ConfigureServices`。 有关详细信息，请参阅[在启动期间访问配置](#access-configuration-during-startup)部分。

## <a name="command-line-configuration-provider"></a>命令行配置提供程序

<xref:Microsoft.Extensions.Configuration.CommandLine.CommandLineConfigurationProvider> 在运行时从命令行参数键值对加载配置。

要激活命令行配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> 扩展方法。

调用 `CreateDefaultBuilder(string [])` 时会自动调用 `AddCommandLine`。 有关详细信息，请参阅[默认配置](#default-configuration)部分。

此外，`CreateDefaultBuilder` 也会加载：

* appsettings.json 和 appsettings.{Environment}.json 文件中的可选配置 。
* [用户机密 (Secret Manager)](xref:security/app-secrets)（在开发环境中）。
* 环境变量。

`CreateDefaultBuilder` 最后添加命令行配置提供程序。 在运行时传递的命令行参数会替代由其他提供程序设置的配置。

`CreateDefaultBuilder` 在构造主机时起作用。 因此，`CreateDefaultBuilder` 激活的命令行配置可能会影响主机的配置方式。

对于基于 ASP.NET Core 模板的应用，`CreateDefaultBuilder` 已调用 `AddCommandLine`。 若要添加其他配置提供程序并保持能够用命令行参数替代这些提供程序的配置，请在 `ConfigureAppConfiguration` 中调用应用的其他提供程序，并最后调用 `AddCommandLine`。

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    // Call other providers here
    config.AddCommandLine(args);
})
```

**示例**

示例应用利用静态便捷方法 `CreateDefaultBuilder` 来生成主机，其中包括一个对 <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> 的调用。

1. 在项目的目录中打开命令提示符。
1. 为 `dotnet run` 命令提供命令行参数 `dotnet run CommandLineKey=CommandLineValue`。
1. 应用运行后，在 `http://localhost:5000` 打开应用的浏览器。
1. 观察输出是否包含提供给 `dotnet run` 的配置命令行参数的键值对。

### <a name="arguments"></a>自变量

该值必须后跟一个等号 (`=`)，否则当值后跟一个空格时，键必须具有前缀（`--` 或 `/`）。 如果使用等号（例如 `CommandLineKey=`），则不需要该值。

| 键前缀               | 示例                                                |
| ------------------------ | ------------------------------------------------------ |
| 无前缀                | `CommandLineKey1=value1`                               |
| 双划线 (`--`)        | `--CommandLineKey2=value2`, `--CommandLineKey2 value2` |
| 正斜杠 (`/`)      | `/CommandLineKey3=value3`, `/CommandLineKey3 value3`   |

在同一命令中，不要将使用等号的命令行参数键值对与使用空格的键值对混合使用。

示例命令：

```dotnetcli
dotnet run CommandLineKey1=value1 --CommandLineKey2=value2 /CommandLineKey3=value3
dotnet run --CommandLineKey1 value1 /CommandLineKey2 value2
dotnet run CommandLineKey1= CommandLineKey2=value2
```

### <a name="switch-mappings"></a>交换映射

交换映射支持键名替换逻辑。 使用 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 手动构建配置时，需要为 <xref:Microsoft.Extensions.Configuration.CommandLineConfigurationExtensions.AddCommandLine*> 方法提供交换替换字典。

当使用交换映射字典时，会检查字典中是否有与命令行参数提供的键匹配的键。 如果在字典中找到命令行键，则传回字典值（键替换）以将键值对设置为应用的配置。 对任何具有单划线 (`-`) 前缀的命令行键而言，交换映射都是必需的。

交换映射字典键规则：

* 交换必须以单划线 (`-`) 或双划线 (`--`) 开头。
* 交换映射字典不得包含重复键。

创建交换映射字典。 在以下示例中，创建了两个交换映射：

```csharp
public static readonly Dictionary<string, string> _switchMappings = 
    new Dictionary<string, string>
    {
        { "-CLKey1", "CommandLineKey1" },
        { "-CLKey2", "CommandLineKey2" }
    };
```

生成主机后，使用交换映射字典来调用 `AddCommandLine`：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddCommandLine(args, _switchMappings);
})
```

对于使用交换映射的应用，调用 `CreateDefaultBuilder` 不应传递参数。 `CreateDefaultBuilder` 方法的 `AddCommandLine` 调用不包括映射的交换，并且无法将交换映射字典传递给 `CreateDefaultBuilder`。 解决方案不是将参数传递给 `CreateDefaultBuilder`，而是允许 `ConfigurationBuilder` 方法的 `AddCommandLine` 方法处理参数和交换映射字典。

创建交换映射字典后，它将包含下表所示的数据。

| 密钥       | 值             |
| --------- | ----------------- |
| `-CLKey1` | `CommandLineKey1` |
| `-CLKey2` | `CommandLineKey2` |

如果在启动应用时使用了交换映射的键，则配置将接收字典提供的密钥上的配置值：

```dotnetcli
dotnet run -CLKey1=value1 -CLKey2=value2
```

运行上述命令后，配置包含下表中显示的值。

| 键               | “值”    |
| ----------------- | -------- |
| `CommandLineKey1` | `value1` |
| `CommandLineKey2` | `value2` |

## <a name="environment-variables-configuration-provider"></a>环境变量配置提供程序

<xref:Microsoft.Extensions.Configuration.EnvironmentVariables.EnvironmentVariablesConfigurationProvider> 在运行时从环境变量键值对加载配置。

要激活环境变量配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.EnvironmentVariablesExtensions.AddEnvironmentVariables*> 扩展方法。

[!INCLUDE[](~/includes/environmentVarableColon.md)]

借助 [Azure 应用服务](https://azure.microsoft.com/services/app-service/)，可在 Azure 门户中设置使用环境变量配置提供程序替代应用配置的环境变量。 有关详细信息，请参阅 [Azure 应用：使用 Azure 门户替代应用配置](xref:host-and-deploy/azure-apps/index#override-app-configuration-using-the-azure-portal)。

如果使用 [Web 主机](xref:fundamentals/host/web-host)初始化新的主机生成器，且调用 `CreateDefaultBuilder`，则使用 `AddEnvironmentVariables` 为[主机配置](#host-versus-app-configuration)加载前缀为 `ASPNETCORE_` 的环境变量。 有关详细信息，请参阅[默认配置](#default-configuration)部分。

此外，`CreateDefaultBuilder` 也会加载：

* 来自没有前缀的环境变量的应用配置，方法是通过调用不带前缀的 `AddEnvironmentVariables`。
* appsettings.json 和 appsettings.{Environment}.json 文件中的可选配置 。
* [用户机密 (Secret Manager)](xref:security/app-secrets)（在开发环境中）。
* 命令行参数。

环境变量配置提供程序是在配置已根据用户机密和 appsettings 文件建立后调用。 在此位置调用提供程序允许在运行时读取的环境变量替代由用户机密和 appsettings 文件设置的配置。

要从其他环境变量提供应用配置，请在 `ConfigureAppConfiguration` 中调用应用的其他提供程序，并使用前缀调用 `AddEnvironmentVariables`：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddEnvironmentVariables(prefix: "PREFIX_");
})
```

最后调用 `AddEnvironmentVariables`让带给定前缀的环境变量可替代其他提供程序中的值。

**示例**

示例应用利用静态便捷方法 `CreateDefaultBuilder` 来生成主机，其中包括一个对 `AddEnvironmentVariables` 的调用。

1. 运行示例应用。 在 `http://localhost:5000` 打开应用的浏览器。
1. 观察输出是否包含环境变量 `ENVIRONMENT` 的键值对。 该值反映了应用运行的环境，在本地运行时通常为 `Development`。

为了缩短应用呈现的环境变量列表，应用会筛选环境变量。 请参阅示例应用的“Pages/Index.cshtml.cs”文件。

要公开应用可用的所有环境变量，请将 Pages/Index.cshtml.cs 中的 `FilteredConfiguration` 更改为以下内容：

```csharp
FilteredConfiguration = _config.AsEnumerable();
```

### <a name="prefixes"></a>前缀

为 `AddEnvironmentVariables` 方法提供前缀时，将筛选加载到应用的配置中的环境变量。 例如，要筛选前缀 `CUSTOM_` 上的环境变量，请将前缀提供给配置提供程序：

```csharp
var config = new ConfigurationBuilder()
    .AddEnvironmentVariables("CUSTOM_")
    .Build();
```

创建配置键值对时，将去除前缀。

若已创建主机生成器，则主机配置由环境变量提供。 有关用于这些环境变量的前缀的详细信息，请参阅[默认配置](#default-configuration)部分。

**连接字符串前缀**

针对为应用环境配置 Azure 连接字符串所涉及的四个连接字符串环境变量，配置 API 具有特殊的处理规则。 如果没有向 `AddEnvironmentVariables` 提供前缀，则具有表中所示前缀的环境变量将加载到应用中。

| 连接字符串前缀 | 提供程序 |
| ------------------------ | -------- |
| `CUSTOMCONNSTR_` | 自定义提供程序 |
| `MYSQLCONNSTR_` | [MySQL](https://www.mysql.com/) |
| `SQLAZURECONNSTR_` | [Azure SQL 数据库](https://azure.microsoft.com/services/sql-database/) |
| `SQLCONNSTR_` | [SQL Server](https://www.microsoft.com/sql-server/) |

当发现环境变量并使用表中所示的四个前缀中的任何一个加载到配置中时：

* 通过删除环境变量前缀并添加配置键节 (`ConnectionStrings`) 来创建配置键。
* 创建一个新的配置键值对，表示数据库连接提供程序（`CUSTOMCONNSTR_` 除外，它没有声明的提供程序）。

| 环境变量键 | 转换的配置键 | 提供程序配置条目                                                    |
| ------------------------ | --------------------------- | ------------------------------------------------------------------------------- |
| `CUSTOMCONNSTR_{KEY} `   | `ConnectionStrings:{KEY}`   | 配置条目未创建。                                                |
| `MYSQLCONNSTR_{KEY}`     | `ConnectionStrings:{KEY}`   | 键：`ConnectionStrings:{KEY}_ProviderName`：<br>值：`MySql.Data.MySqlClient` |
| `SQLAZURECONNSTR_{KEY}`  | `ConnectionStrings:{KEY}`   | 键：`ConnectionStrings:{KEY}_ProviderName`：<br>值：`System.Data.SqlClient`  |
| `SQLCONNSTR_{KEY}`       | `ConnectionStrings:{KEY}`   | 键：`ConnectionStrings:{KEY}_ProviderName`：<br>值：`System.Data.SqlClient`  |

**示例**

在服务器上创建了一个自定义连接字符串环境变量：

* 名称：`CUSTOMCONNSTR_ReleaseDB`
* 值：`Data Source=ReleaseSQLServer;Initial Catalog=MyReleaseDB;Integrated Security=True`

如果 `IConfiguration` 已引入并分配给名为 `_config` 的字段，请读取值：

```csharp
_config["ConnectionStrings:ReleaseDB"]
```

## <a name="file-configuration-provider"></a>文件配置提供程序

<xref:Microsoft.Extensions.Configuration.FileConfigurationProvider> 是从文件系统加载配置的基类。 以下配置提供程序专用于特定文件类型：

* [INI 配置提供程序](#ini-configuration-provider)
* [JSON 配置提供程序](#json-configuration-provider)
* [XML 配置提供程序](#xml-configuration-provider)

### <a name="ini-configuration-provider"></a>INI 配置提供程序

<xref:Microsoft.Extensions.Configuration.Ini.IniConfigurationProvider> 在运行时从 INI 文件键值对加载配置。

若要激活 INI 文件配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.IniConfigurationExtensions.AddIniFile*> 扩展方法。

冒号可用作 INI 文件配置中的节分隔符。

重载允许指定：

* 文件是否可选。
* 如果文件更改，是否重载配置。
* <xref:Microsoft.Extensions.FileProviders.IFileProvider> 用于访问该文件。

构建主机时调用 `ConfigureAppConfiguration` 以指定应用的配置：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddIniFile(
        "config.ini", optional: true, reloadOnChange: true);
})
```

INI 配置文件的通用示例：

```ini
[section0]
key0=value
key1=value

[section1]
subsection:key=value

[section2:subsection0]
key=value

[section2:subsection1]
key=value
```

以前的配置文件使用 `value` 加载以下键：

* section0:key0
* section0:key1
* section1:subsection:key
* section2:subsection0:key
* section2:subsection1:key

### <a name="json-configuration-provider"></a>JSON 配置提供程序

<xref:Microsoft.Extensions.Configuration.Json.JsonConfigurationProvider> 在运行时期间从 JSON 文件键值对加载配置。

若要激活 JSON 文件配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.JsonConfigurationExtensions.AddJsonFile*> 扩展方法。

重载允许指定：

* 文件是否可选。
* 如果文件更改，是否重载配置。
* <xref:Microsoft.Extensions.FileProviders.IFileProvider> 用于访问该文件。

使用 `CreateDefaultBuilder` 初始化新的主机生成器时，会自动调用两次 `AddJsonFile`。 调用该方法来从以下文件加载配置：

* appsettings.json：先读取此文件。 该文件的环境版本可以替代 appsettings.json 文件提供的值。
* appsettings.{Environment}.json：文件的环境版本是根据 [IHostingEnvironment.EnvironmentName](xref:Microsoft.Extensions.Hosting.IHostingEnvironment.EnvironmentName*) 加载的。

有关详细信息，请参阅[默认配置](#default-configuration)部分。

此外，`CreateDefaultBuilder` 也会加载：

* 环境变量。
* [用户机密 (Secret Manager)](xref:security/app-secrets)（在开发环境中）。
* 命令行参数。

首先建立 JSON 配置提供程序。 因此，用户机密、环境变量和命令行参数会替代由 appsettings 文件设置的配置。

构建主机时调用 `ConfigureAppConfiguration` 以指定除 appsettings.json 和 appsettings.{Environment}.json 以外的文件的应用配置：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddJsonFile(
        "config.json", optional: true, reloadOnChange: true);
})
```

**示例**

示例应用利用静态便捷方法 `CreateDefaultBuilder` 来生成主机，其中包括两个对 `AddJsonFile` 的调用：

* 第一次调用 `AddJsonFile` 会从 appsettings 加载配置：

  [!code-json[](index/samples/2.x/ConfigurationSample/appsettings.json)]

* 第二次调用 `AddJsonFile` 会从 appsettings.{Environment}.json 加载配置。 对于示例应用中的 appsettings.Development.json，将加载以下文件：

  [!code-json[](index/samples/2.x/ConfigurationSample/appsettings.Development.json)]

1. 运行示例应用。 在 `http://localhost:5000` 打开应用的浏览器。
1. 输出包含配置的键值对（由应用的环境而定）。 在开发环境中运行应用时，键 `Logging:LogLevel:Default` 的日志级别为 `Debug`。
1. 再次在生产环境中运行示例应用：
   1. 打开 Properties/launchSettings.json 文件。
   1. 在 `ConfigurationSample` 配置文件中，将 `ASPNETCORE_ENVIRONMENT` 环境变量的值更改为 `Production`。
   1. 保存文件，然后在命令外壳中使用 `dotnet run` 运行应用。
1. appsettings.Development.json 中的设置不再替代 appsettings.json 中的设置 。 键 `Logging:LogLevel:Default` 的日志级别为 `Warning`。

### <a name="xml-configuration-provider"></a>XML 配置提供程序

<xref:Microsoft.Extensions.Configuration.Xml.XmlConfigurationProvider> 在运行时从 XML 文件键值对加载配置。

若要激活 XML 文件配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.XmlConfigurationExtensions.AddXmlFile*> 扩展方法。

重载允许指定：

* 文件是否可选。
* 如果文件更改，是否重载配置。
* <xref:Microsoft.Extensions.FileProviders.IFileProvider> 用于访问该文件。

创建配置键值对时，将忽略配置文件的根节点。 不要在文件中指定文档类型定义 (DTD) 或命名空间。

构建主机时调用 `ConfigureAppConfiguration` 以指定应用的配置：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddXmlFile(
        "config.xml", optional: true, reloadOnChange: true);
})
```

XML 配置文件可以为重复节使用不同的元素名称：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <section0>
    <key0>value</key0>
    <key1>value</key1>
  </section0>
  <section1>
    <key0>value</key0>
    <key1>value</key1>
  </section1>
</configuration>
```

以前的配置文件使用 `value` 加载以下键：

* section0:key0
* section0:key1
* section1:key0
* section1:key1

如果使用 `name` 属性来区分元素，则使用相同元素名称的重复元素可以正常工作：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <section name="section0">
    <key name="key0">value</key>
    <key name="key1">value</key>
  </section>
  <section name="section1">
    <key name="key0">value</key>
    <key name="key1">value</key>
  </section>
</configuration>
```

以前的配置文件使用 `value` 加载以下键：

* section:section0:key:key0
* section:section0:key:key1
* section:section1:key:key0
* section:section1:key:key1

属性可用于提供值：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
  <key attribute="value" />
  <section>
    <key attribute="value" />
  </section>
</configuration>
```

以前的配置文件使用 `value` 加载以下键：

* key:attribute
* section:key:attribute

## <a name="key-per-file-configuration-provider"></a>Key-per-file 配置提供程序

<xref:Microsoft.Extensions.Configuration.KeyPerFile.KeyPerFileConfigurationProvider> 使用目录的文件作为配置键值对。 该键是文件名。 该值包含文件的内容。 Key-per-file 配置提供程序用于 Docker 托管方案。

若要激活 Key-per-file 配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.KeyPerFileConfigurationBuilderExtensions.AddKeyPerFile*> 扩展方法。 文件的 `directoryPath` 必须是绝对路径。

重载允许指定：

* 配置源的 `Action<KeyPerFileConfigurationSource>` 委托。
* 目录是否可选以及目录的路径。

双下划线字符 (`__`) 用作文件名中的配置键分隔符。 例如，文件名 `Logging__LogLevel__System` 生成配置键 `Logging:LogLevel:System`。

构建主机时调用 `ConfigureAppConfiguration` 以指定应用的配置：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    var path = Path.Combine(
        Directory.GetCurrentDirectory(), "path/to/files");
    config.AddKeyPerFile(directoryPath: path, optional: true);
})
```

## <a name="memory-configuration-provider"></a>内存配置提供程序

<xref:Microsoft.Extensions.Configuration.Memory.MemoryConfigurationProvider> 使用内存中集合作为配置键值对。

若要激活内存中集合配置，请在 <xref:Microsoft.Extensions.Configuration.ConfigurationBuilder> 的实例上调用 <xref:Microsoft.Extensions.Configuration.MemoryConfigurationBuilderExtensions.AddInMemoryCollection*> 扩展方法。

可以使用 `IEnumerable<KeyValuePair<String,String>>` 初始化配置提供程序。

构建主机时调用 `ConfigureAppConfiguration` 以指定应用的配置。

在下面的示例中，创建了配置字典：

```csharp
public static readonly Dictionary<string, string> _dict = 
    new Dictionary<string, string>
    {
        {"MemoryCollectionKey1", "value1"},
        {"MemoryCollectionKey2", "value2"}
    };
```

通过 `AddInMemoryCollection` 的调用使用字典，以提供配置：

```csharp
.ConfigureAppConfiguration((hostingContext, config) =>
{
    config.AddInMemoryCollection(_dict);
})
```

## <a name="getvalue"></a>GetValue

[`ConfigurationBinder.GetValue<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.GetValue*) 从配置中提取一个具有指定键的值，并将它转换为指定的非集合类型。 重载接受默认值。

如下示例中：

* 使用键 `NumberKey` 从配置中提取字符串值。 如果在配置键中找不到 `NumberKey`，则使用默认值 `99`。
* 键入值作为 `int`。
* 存储 `NumberConfig` 属性中的值，以供页面使用。

```csharp
public class IndexModel : PageModel
{
    public IndexModel(IConfiguration config)
    {
        _config = config;
    }

    public int NumberConfig { get; private set; }

    public void OnGet()
    {
        NumberConfig = _config.GetValue<int>("NumberKey", 99);
    }
}
```

## <a name="getsection-getchildren-and-exists"></a>GetSection、GetChildren 和 Exists

对于下面的示例，请考虑以下 JSON 文件。 在两个节中找到四个键，其中一个包含一对子节：

```json
{
  "section0": {
    "key0": "value",
    "key1": "value"
  },
  "section1": {
    "key0": "value",
    "key1": "value"
  },
  "section2": {
    "subsection0" : {
      "key0": "value",
      "key1": "value"
    },
    "subsection1" : {
      "key0": "value",
      "key1": "value"
    }
  }
}
```

将文件读入配置时，会创建以下唯一的分层键来保存配置值：

* section0:key0
* section0:key1
* section1:key0
* section1:key1
* section2:subsection0:key0
* section2:subsection0:key1
* section2:subsection1:key0
* section2:subsection1:key1

### <a name="getsection"></a>GetSection

[IConfiguration.GetSection](xref:Microsoft.Extensions.Configuration.IConfiguration.GetSection*) 使用指定的子节键提取配置子节。

若要返回仅包含 `section1` 中键值对的 <xref:Microsoft.Extensions.Configuration.IConfigurationSection>，请调用 `GetSection` 并提供节名称：

```csharp
var configSection = _config.GetSection("section1");
```

`configSection` 不具有值，只有密钥和路径。

同样，若要获取 `section2:subsection0` 中键的值，请调用 `GetSection` 并提供节路径：

```csharp
var configSection = _config.GetSection("section2:subsection0");
```

`GetSection` 永远不会返回 `null`。 如果找不到匹配的节，则返回空 `IConfigurationSection`。

当 `GetSection` 返回匹配的部分时，<xref:Microsoft.Extensions.Configuration.IConfigurationSection.Value> 未填充。 存在该部分时，返回一个 <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Key> 和 <xref:Microsoft.Extensions.Configuration.IConfigurationSection.Path> 部分。

### <a name="getchildren"></a>GetChildren

在 `section2` 上调用 [IConfiguration.GetChildren](xref:Microsoft.Extensions.Configuration.IConfiguration.GetChildren*) 会获得 `IEnumerable<IConfigurationSection>`，其中包括：

* `subsection0`
* `subsection1`

```csharp
var configSection = _config.GetSection("section2");

var children = configSection.GetChildren();
```

### <a name="exists"></a>存在

使用 [ConfigurationExtensions.Exists](xref:Microsoft.Extensions.Configuration.ConfigurationExtensions.Exists*) 确定配置节是否存在：

```csharp
var sectionExists = _config.GetSection("section2:subsection2").Exists();
```

给定示例数据，`sectionExists` 为 `false`，因为配置数据中没有 `section2:subsection2` 节。

## <a name="bind-to-an-object-graph"></a>绑定至对象图

<xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*> 能够绑定整个 POCO 对象图。 与绑定简单对象一样，只绑定公共读取/写入属性。

该示例包含 `TvShow` 模型，其对象图包含 `Metadata` 和 `Actors` 类 (Models/TvShow.cs)：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/TvShow.cs?name=snippet1)]

示例应用有一个包含配置数据的 tvshow.xml 文件：

[!code-xml[](index/samples/2.x/ConfigurationSample/tvshow.xml)]

使用 `Bind` 方法将配置绑定到整个 `TvShow` 对象图。 将绑定实例分配给用于呈现的属性：

```csharp
var tvShow = new TvShow();
_config.GetSection("tvshow").Bind(tvShow);
TvShow = tvShow;
```

[`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) 绑定并返回指定的类型。 `Get<T>` 比使用 `Bind` 更方便。 以下代码演示了如何通过前述示例使用 `Get<T>`：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Pages/Index.cshtml.cs?name=snippet_tvshow)]

## <a name="bind-an-array-to-a-class"></a>将数组绑定至类

示例应用演示了本部分中介绍的概念。

<xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Bind*> 支持使用配置键中的数组索引将数组绑定到对象。 公开数字键段（`:0:`、`:1:`、&hellip; `:{n}:`）的任何数组格式都能够与 POCO 类数组进行数组绑定。

> [!NOTE]
> 绑定是按约定提供的。 不需要自定义配置提供程序实现数组绑定。

**内存中数组处理**

请考虑下表中所示的配置键和值。

| 键             | “值”  |
| :-------------: | :----: |
| array:entries:0 | value0 |
| array:entries:1 | value1 |
| array:entries:2 | value2 |
| array:entries:4 | value4 |
| array:entries:5 | value5 |

使用内存配置提供程序在示例应用中加载这些键和值：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=5-12,22)]

该数组跳过索引 &num;3 的值。 配置绑定程序无法绑定 null 值，也无法在绑定对象中创建 null 条目，这在演示将此数组绑定到对象的结果时变得清晰。

在示例应用中，POCO 类可用于保存绑定的配置数据：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/ArrayExample.cs?name=snippet1)]

将配置数据绑定至对象：

```csharp
var arrayExample = new ArrayExample();
_config.GetSection("array").Bind(arrayExample);
```

还可以使用 [`ConfigurationBinder.Get<T>`](xref:Microsoft.Extensions.Configuration.ConfigurationBinder.Get*) 语法，这样会得到更精简的代码：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Pages/Index.cshtml.cs?name=snippet_array)]

绑定对象（`ArrayExample` 的实例）从配置接收数组数据。

| `ArrayExample.Entries` 索引 | `ArrayExample.Entries` 值 |
| :--------------------------: | :--------------------------: |
| 0                            | value0                       |
| 1                            | value1                       |
| 2                            | value2                       |
| 3                            | value4                       |
| 4                            | value5                       |

绑定对象中的索引 &num;3 保留 `array:4` 配置键的配置数据及其值 `value4`。 当绑定包含数组的配置数据时，配置键中的数组索引仅用于在创建对象时迭代配置数据。 无法在配置数据中保留 null 值，并且当配置键中的数组跳过一个或多个索引时，不会在绑定对象中创建 null 值条目。

可以在由任何在配置中生成正确键值对的配置提供程序绑定到 `ArrayExample` 实例之前提供索引 &num;3 的缺失配置项。 如果示例包含具有缺失键值对的其他 JSON 配置提供程序，则 `ArrayExample.Entries` 与完整配置数组相匹配：

*missing_value.json*:

```json
{
  "array:entries:3": "value3"
}
```

在 `ConfigureAppConfiguration`中：

```csharp
config.AddJsonFile(
    "missing_value.json", optional: false, reloadOnChange: false);
```

将表中所示的键值对加载到配置中。

| 密钥             | 值  |
| :-------------: | :----: |
| array:entries:3 | value3 |

如果在 JSON 配置提供程序包含索引 &num;3 的条目之后绑定 `ArrayExample` 类实例，则 `ArrayExample.Entries` 数组包含该值。

| `ArrayExample.Entries` 索引 | `ArrayExample.Entries` 值 |
| :--------------------------: | :--------------------------: |
| 0                            | value0                       |
| 1                            | value1                       |
| 2                            | value2                       |
| 3                            | value3                       |
| 4                            | value4                       |
| 5                            | value5                       |

**JSON 数组处理**

如果 JSON 文件包含数组，则会为具有从零开始的节索引的数组元素创建配置键。 在以下配置文件中，`subsection` 是一个数组：

[!code-json[](index/samples/2.x/ConfigurationSample/json_array.json)]

JSON 配置提供程序将配置数据读入以下键值对：

| 键                     | 值  |
| ----------------------- | :----: |
| json_array:key          | valueA |
| json_array:subsection:0 | valueB |
| json_array:subsection:1 | valueC |
| json_array:subsection:2 | valueD |

在示例应用中，以下 POCO 类可用于绑定配置键值对：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/JsonArrayExample.cs?name=snippet1)]

绑定后，`JsonArrayExample.Key` 保存值 `valueA`。 子节值存储在 POCO 数组属性 `Subsection` 中。

| `JsonArrayExample.Subsection` 索引 | `JsonArrayExample.Subsection` 值 |
| :---------------------------------: | :---------------------------------: |
| 0                                   | valueB                              |
| 1                                   | valueC                              |
| 2                                   | valueD                              |

## <a name="custom-configuration-provider"></a>自定义配置提供程序

该示例应用演示了如何使用[实体框架 (EF)](/ef/core/) 创建从数据库读取配置键值对的基本配置提供程序。

提供程序具有以下特征：

* EF 内存中数据库用于演示目的。 若要使用需要连接字符串的数据库，请实现辅助 `ConfigurationBuilder` 以从另一个配置提供程序提供连接字符串。
* 提供程序在启动时将数据库表读入配置。 提供程序不会基于每个键查询数据库。
* 未实现更改时重载，因此在应用启动后更新数据库对应用的配置没有任何影响。

定义用于在数据库中存储配置值的 `EFConfigurationValue` 实体。

*Models/EFConfigurationValue.cs*：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Models/EFConfigurationValue.cs?name=snippet1)]

添加 `EFConfigurationContext` 以存储和访问配置的值。

*EFConfigurationProvider/EFConfigurationContext.cs*：

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationContext.cs?name=snippet1)]

创建用于实现 <xref:Microsoft.Extensions.Configuration.IConfigurationSource> 的类。

*EFConfigurationProvider/EFConfigurationSource.cs*：

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationSource.cs?name=snippet1)]

通过从 <xref:Microsoft.Extensions.Configuration.ConfigurationProvider> 继承来创建自定义配置提供程序。 当数据库为空时，配置提供程序将对其进行初始化。

*EFConfigurationProvider/EFConfigurationProvider.cs*：

[!code-csharp[](index/samples/2.x/ConfigurationSample/EFConfigurationProvider/EFConfigurationProvider.cs?name=snippet1)]

可以使用 `AddEFConfiguration` 扩展方法将配置源添加到 `ConfigurationBuilder`。

Extensions/EntityFrameworkExtensions.cs：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Extensions/EntityFrameworkExtensions.cs?name=snippet1)]

下面的代码演示如何在 Program.cs 中使用自定义的 `EFConfigurationProvider`：

[!code-csharp[](index/samples/2.x/ConfigurationSample/Program.cs?name=snippet_Program&highlight=29-30)]

## <a name="access-configuration-during-startup"></a>在启动期间访问配置

将 `IConfiguration` 注入 `Startup` 构造函数以访问 `Startup.ConfigureServices` 中的配置值。 若要访问 `Startup.Configure` 中的配置，请将 `IConfiguration` 直接注入方法或使用构造函数中的实例：

```csharp
public class Startup
{
    private readonly IConfiguration _config;

    public Startup(IConfiguration config)
    {
        _config = config;
    }

    public void ConfigureServices(IServiceCollection services)
    {
        var value = _config["key"];
    }

    public void Configure(IApplicationBuilder app, IConfiguration config)
    {
        var value = config["key"];
    }
}
```

有关使用启动便捷方法访问配置的示例，请参阅[应用启动：便捷方法](xref:fundamentals/startup#convenience-methods)。

## <a name="access-configuration-in-a-no-locrazor-pages-page-or-mvc-view"></a>在 Razor Pages 页面或 MVC 视图中访问配置

若要访问 Razor Pages 页面或 MVC 视图中的配置设置，请为 [Microsoft.Extensions.Configuration namespace](xref:Microsoft.Extensions.Configuration) 命名空间添加 [using 指令](xref:mvc/views/razor#using)（[C# 参考：using 指令](/dotnet/csharp/language-reference/keywords/using-directive)）并将 <xref:Microsoft.Extensions.Configuration.IConfiguration> 注入该页面或视图。

在 Razor Pages 页面中：

```cshtml
@page
@model IndexModel
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Index Page</title>
</head>
<body>
    <h1>Access configuration in a Razor Pages page</h1>
    <p>Configuration value for 'key': @Configuration["key"]</p>
</body>
</html>
```

在 MVC 视图中：

```cshtml
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration

<!DOCTYPE html>
<html lang="en">
<head>
    <title>Index View</title>
</head>
<body>
    <h1>Access configuration in an MVC view</h1>
    <p>Configuration value for 'key': @Configuration["key"]</p>
</body>
</html>
```

## <a name="add-configuration-from-an-external-assembly"></a>从外部程序集添加配置

通过 <xref:Microsoft.AspNetCore.Hosting.IHostingStartup> 实现，可在启动时从应用 `Startup` 类之外的外部程序集向应用添加增强功能。 有关详细信息，请参阅 <xref:fundamentals/configuration/platform-specific-configuration>。

## <a name="additional-resources"></a>其他资源

* <xref:fundamentals/configuration/options>

::: moniker-end