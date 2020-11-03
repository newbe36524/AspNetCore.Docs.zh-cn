---
title: IIS 模块与 ASP.NET Core
author: rick-anderson
description: 了解适用于 ASP.NET Core 应用的活动和非活动 IIS 模块以及如何管理 IIS 模块。
monikerRange: '>= aspnetcore-2.1'
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
uid: host-and-deploy/iis/modules
ms.openlocfilehash: 6936071339786262fa8eeb669a59225a695d7488
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722801"
---
# <a name="iis-modules-with-aspnet-core"></a>IIS 模块与 ASP.NET Core

某些本机 IIS 模块和所有 IIS 托管模块无法处理 ASP.NET Core 应用的请求。 在许多情况下，ASP.NET Core 提供了 IIS 本机和托管模块解决的方案的替代方案。

## <a name="native-modules"></a>本机模块

该表指示在使用 ASP.NET Core 应用和 ASP.NET Core 模块时正常工作的本机 IIS 模块。

| 模块 | 在 ASP.NET Core 应用内可用 | ASP.NET Core 选项 |
| --- | :---: | --- |
| **匿名身份验证**<br>`AnonymousAuthenticationModule`                                  | 是 | |
| **基本身份验证**<br>`BasicAuthenticationModule`                                          | 是 | |
| **客户端证书映射身份验证**<br>`CertificateMappingAuthenticationModule`      | 是 | |
| **CGI**<br>`CgiModule`                                                                           | 否  | |
| **配置验证**<br>`ConfigurationValidationModule`                                  | 是 | |
| **HTTP 错误**<br>`CustomErrorModule`                                                           | 否  | [状态代码页中间件](xref:fundamentals/error-handling#usestatuscodepages) |
| **自定义日志记录**<br>`CustomLoggingModule`                                                      | 是 | |
| **默认文档**<br>`DefaultDocumentModule`                                                  | 否  | [默认文件中间件](xref:fundamentals/static-files#serve-a-default-document) |
| **摘要式身份验证**<br>`DigestAuthenticationModule`                                        | 是 | |
| **目录浏览**<br>`DirectoryListingModule`                                               | 否  | [目录浏览中间件](xref:fundamentals/static-files#enable-directory-browsing) |
| **动态压缩**<br>`DynamicCompressionModule`                                            | 是 | [响应压缩中间件](xref:performance/response-compression) |
| **请求跟踪失败**<br>`FailedRequestsTracingModule`                                     | 是 | [ASP.NET Core 日志记录](xref:fundamentals/logging/index#tracesource-provider) |
| **文件缓存**<br>`FileCacheModule`                                                            | 否  | [响应缓存中间件](xref:performance/caching/middleware) |
| **HTTP 缓存**<br>`HttpCacheModule`                                                            | 否  | [响应缓存中间件](xref:performance/caching/middleware) |
| **HTTP 日志记录**<br>`HttpLoggingModule`                                                          | 是 | [ASP.NET Core 日志记录](xref:fundamentals/logging/index) |
| **HTTP 重定向**<br>`HttpRedirectionModule`                                                  | 是 | [URL 重写中间件](xref:fundamentals/url-rewriting) |
| **HTTP 跟踪**<br>`TracingModule`                                                              | 是 | |
| **IIS 客户端证书映射身份验证**<br>`IISCertificateMappingAuthenticationModule` | 是 | |
| **IP 和域限制**<br>`IpRestrictionModule`                                          | 是 | |
| **ISAPI 筛选器**<br>`IsapiFilterModule`                                                         | 是 | [中间件](xref:fundamentals/middleware/index) |
| **ISAPI**<br>`IsapiModule`                                                                       | 是 | [中间件](xref:fundamentals/middleware/index) |
| **协议支持**<br>`ProtocolSupportModule`                                                  | 是 | |
| **请求筛选**<br>`RequestFilteringModule`                                                | 是 | [URL 重写中间件`IRule`](xref:fundamentals/url-rewriting#irule-based-rule) |
| **请求监视器**<br>`RequestMonitorModule`                                                    | 是 | |
| **URL 重写**&#8224；<br>`RewriteModule`                                                      | 是 | [URL 重写中间件](xref:fundamentals/url-rewriting) |
| **服务器端包括**<br>`ServerSideIncludeModule`                                            | 否  | |
| **静态压缩**<br>`StaticCompressionModule`                                              | 否  | [响应压缩中间件](xref:performance/response-compression) |
| **静态内容**<br>`StaticFileModule`                                                         | 否  | [静态文件中间件](xref:fundamentals/static-files) |
| **令牌缓存**<br>`TokenCacheModule`                                                          | 是 | |
| **URI 缓存**<br>`UriCacheModule`                                                              | 是 | |
| **URL 授权**<br>`UrlAuthorizationModule`                                                | 是 | [ASP.NET Core Identity](xref:security/authentication/identity) |
| **Windows 身份验证**<br>`WindowsAuthenticationModule`                                      | 是 | |

&#8224;由于[目录结构](xref:host-and-deploy/directory-structure)中的更改，URL 重写模块的 `isFile` 和 `isDirectory` 匹配类型不适用于 ASP.NET Core 应用。

## <a name="managed-modules"></a>托管模块

当应用池的 .NET CLR 版本设置为“无托管代码”时，托管 ASP.NET Core 应用的托管模块无法使用。 ASP.NET Core 在几种情况下提供了中间件替代方案。

| 模块                  | ASP.NET Core 选项 |
| ----------------------- | ------------------- |
| AnonymousIdentification | |
| DefaultAuthentication   | |
| FileAuthorization       | |
| FormsAuthentication     | [Cookie 身份验证中间件](xref:security/authentication/cookie) |
| OutputCache             | [响应缓存中间件](xref:performance/caching/middleware) |
| 配置文件                 | |
| RoleManager             | |
| ScriptModule-4.0        | |
| 会话                 | [会话中间件](xref:fundamentals/app-state) |
| UrlAuthorization        | |
| UrlMappingsModule       | [URL 重写中间件](xref:fundamentals/url-rewriting) |
| UrlRoutingModule-4.0    | [ASP.NET Core Identity](xref:security/authentication/identity) |
| WindowsAuthentication   | |

## <a name="iis-manager-application-changes"></a>IIS 管理器应用程序更改

使用 IIS 管理器配置设置时，应用的 web.config 文件已更改。 如果部署应用并包括 web.config，则使用 IIS 管理器所做的任何更改都会被部署的 web.config 文件覆盖。 如果更改服务器的 web.config 文件，请立即将服务器上已更新的 web.config 文件复制到本地项目。

## <a name="disabling-iis-modules"></a>禁用 IIS 模块

如果在服务器级别配置了必须为应用禁用的 IIS 模块，则应用 web.config 文件还可以禁用该模块。 将模块留在原位并使用配置设置（如果可用）将其停用或从应用中移除模块。

### <a name="module-deactivation"></a>模块停用

许多模块提供一个配置设置，允许在不从应用中删除模块的情况下禁用它们。 这是停用模块最简单、快捷的方式。 例如，可使用 web.config 中的 `<httpRedirect>` 元素禁用 HTTP 重定向模块：

```xml
<configuration>
  <system.webServer>
    <httpRedirect enabled="false" />
  </system.webServer>
</configuration>
```

有关通过配置设置禁用模块的详细信息，请按照 [IIS \<system.webServer>](/iis/configuration/system.webServer/) 的“子元素”部分中的链接进行操作。

### <a name="module-removal"></a>模块删除

如果选择使用 web.config 中的设置删除模块，请先解锁此模块和 web.config 的 `<modules>` 部分 ：

1. 解锁服务器级别的模块。 在 IIS 管理器“连接”边栏中选择 IIS 服务器。 打开 IIS 区域中的模块。 在列表中选择该模块。 在右侧的“操作”边栏中，选择“解锁”。 如果该模块的操作条目显示为“锁定”，则该模块已被解锁，且无需任何操作。 解锁尽可能多稍后计划从 web.config 中删除的模块。

2. 在不使用 web.config 的 `<modules>` 部分的情况下部署应用。如果使用包含 `<modules>` 部分的 web.config 部署应用，但未先在 IIS 管理器中解锁该部分，则配置管理器会在尝试解锁该部分时引发异常。 因此，请在不使用 `<modules>` 部分的情况下部署该应用。

3. 解锁 web.config 的 `<modules>` 部分。在“连接”边栏中，选择“站点”中的网站。 在“管理”区域中，打开“配置编辑器”。 使用导航控件选择 `system.webServer/modules` 部分。 在右侧的“操作”边栏中，选择“解锁”该部分。 如果该模块的操作条目显示为“锁定部分”，则该模块已被解锁，且无需任何操作。

4. 可使用 `<remove>` 元素将 `<modules>` 部分添加到应用的本地 web.config 文件，从应用中删除模块。 可添加多个 `<remove>` 元素来删除多个模块。 如果在服务器上更改 web.config，请立即在本地对项目的 web.config 文件执行相同的更改。 以这种方式删除模块不会影响模块的使用与服务器上的其他应用。

   ```xml
   <configuration>
    <system.webServer>
      <modules>
        <remove name="MODULE_NAME" />
      </modules>
    </system.webServer>
   </configuration>
   ```

要使用 web.config 为 IIS Express 添加或删除模块，请修改 applicationHost.config 以解锁 `<modules>` 部分 ：

1. 打开 {APPLICATION ROOT}\\.vs\config\applicationhost.config。

1. 找到 IIS 模块的 `<section>` 元素，并将 `overrideModeDefault` 从 `Deny` 更改为 `Allow`：

   ```xml
   <section name="modules"
            allowDefinition="MachineToApplication"
            overrideModeDefault="Allow" />
   ```

1. 找到 `<location path="" overrideMode="Allow"><system.webServer><modules>` 部分。 对于任何要删除的模块，请将 `lockItem` 从 `true` 设置为 `false`。 在以下示例中，已解锁 CGI 模块：

   ```xml
   <add name="CgiModule" lockItem="false" />
   ```

1. 解锁 `<modules>` 部分和单个模块后，可使用应用的 web.config 文件添加或删除 IIS 模块，以便在 IIS Express 上运行应用。

IIS 模块还可以使用 Appcmd.exe 删除。 该命令中提供 `MODULE_NAME` 和 `APPLICATION_NAME`：

```console
Appcmd.exe delete module MODULE_NAME /app.name:APPLICATION_NAME
```

例如，从默认网站中删除 `DynamicCompressionModule`：

```console
%windir%\system32\inetsrv\appcmd.exe delete module DynamicCompressionModule /app.name:"Default Web Site"
```

## <a name="minimum-module-configuration"></a>最小模块配置

要求运行 ASP.NET Core 应用的模块只有匿名身份验证模块和 ASP.NET Core 模块。

URI 缓存模块 (`UriCacheModule`) 允许 IIS 在 URL 级别缓存网站配置。 不使用此模块的话，即使重复请求相同的 URL，IIS 也必须读取并分析每个请求的配置。 分析每个请求的配置会导致严重的性能损失。 虽然托管的 ASP.NET Core 应用并非严格要求运行 URI 缓存模块，但我们建议为所有 ASP.NET Core 部署启用 URI 缓存模块。

HTTP 缓存模块 (`HttpCacheModule`) 实现 IIS 输出缓存以及用于在 HTTP.sys 缓存中缓存项目的逻辑。 如果不使用此模块，内容不再以内核模式缓存，并且缓存配置文件将被忽略。 删除 HTTP 缓存模块通常会对性能和资源使用情况产生不利影响。 虽然托管的 ASP.NET Core 应用程序并非严格要求运行 HTTP 缓存模块，但我们建议为所有 ASP.NET Core 部署启用 HTTP 缓存模块。

## <a name="additional-resources"></a>其他资源

* [IIS 体系结构简介：IIS 中的模块](/iis/get-started/introduction-to-iis/introduction-to-iis-architecture#modules-in-iis)
* [IIS 模块概述](/iis/get-started/introduction-to-iis/iis-modules-overview)
* [自定义 IIS 7.0 角色和模块](/previous-versions/tn-archive/cc627313(v=technet.10))
* [IIS \<system.webServer>](/iis/configuration/system.webServer/)