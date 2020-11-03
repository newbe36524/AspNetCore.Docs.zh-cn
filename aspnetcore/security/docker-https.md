---
title: 通过 HTTPS 在 Docker 上宿主 ASP.NET Core 映像
author: rick-anderson
description: 了解如何通过 HTTPS 在 Docker 上托管 ASP.NET Core 映像
monikerRange: '>= aspnetcore-2.1'
ms.author: riande
ms.custom: mvc
ms.date: 07/05/2019
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
uid: security/docker-https
ms.openlocfilehash: 63d6e220c0f28e552207039c1649041bfdf4a0d4
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93059671"
---
# <a name="hosting-aspnet-core-images-with-docker-over-https"></a>通过 HTTPS 在 Docker 上宿主 ASP.NET Core 映像

作者：[Rick Anderson](https://twitter.com/RickAndMSFT)

[默认情况下](./enforcing-ssl.md)，ASP.NET Core 使用 HTTPS。 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 依赖于信任、标识和加密的 [证书](https://en.wikipedia.org/wiki/Public_key_certificate) 。

本文档介绍如何通过 HTTPS 运行预生成的容器映像。

若要开发方案，请参阅 [通过 HTTPS 上的 Docker 开发 ASP.NET Core 应用程序](https://github.com/dotnet/dotnet-docker/blob/master/samples/run-aspnetcore-https-development.md) 。

此示例需要 docker [17.06](https://docs.docker.com/release-notes/docker-ce) 或更高版本的 [docker 客户端](https://www.docker.com/products/docker)。

## <a name="prerequisites"></a>先决条件

本文档中的某些说明需要 [.Net Core 2.2 SDK](https://dotnet.microsoft.com/download) 或更高版本。

## <a name="certificates"></a>证书

针对域的[生产主机](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/)需要[证书颁发机构颁发](https://wikipedia.org/wiki/Certificate_authority)的证书。 [Let's Encrypt](https://letsencrypt.org/) 是提供免费证书的证书颁发机构。

本文档使用 [自签名开发证书](https://en.wikipedia.org/wiki/Self-signed_certificate) 来托管预生成的映像 `localhost` 。 说明类似于使用生产证书。

对于生产证书：

* 此 `dotnet dev-certs` 工具不是必需的。
* 证书不需要存储在说明中使用的位置。 尽管不建议在网站目录中存储证书，但任何位置都应有效。

以下部分中包含的说明使用 Docker 的命令行选项将证书装载到容器中 `-v` 。 可以使用 Dockerfile 中的命令将证书添加到容器映像 `COPY` 中，但不建议这样做。 *Dockerfile* 由于以下原因，不建议将证书复制到映像：

* 使用同一个映像测试开发人员证书非常困难。
* 使用同一个映像通过生产证书进行托管很困难。
* 证书泄露存在重大风险。

## <a name="running-pre-built-container-images-with-https"></a>用 HTTPS 运行预生成的容器映像

使用以下有关操作系统配置的说明。

### <a name="windows-using-linux-containers"></a>使用 Linux 容器的 Windows

生成证书并配置本地计算机：

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

在上述命令中，将替换 `{ password here }` 为密码。

在命令行界面中使用为 HTTPS 配置的 ASP.NET Core 运行容器映像：

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

使用 [PowerShell](/powershell/scripting/overview)时，请将替换 `%USERPROFILE%` 为 `$env:USERPROFILE` 。

密码必须与用于证书的密码匹配。

### <a name="macos-or-linux"></a>macOS 或 Linux

生成证书并配置本地计算机：

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust` 仅在 macOS 和 Windows 上受支持。 你需要以分发的支持方式信任 Linux 上的证书。 可能需要在浏览器中信任该证书。

在上述命令中，将替换 `{ password here }` 为密码。

运行容器映像，并为 HTTPS 配置 ASP.NET Core：

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx -v ${HOME}/.aspnet/https:/https/ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

密码必须与用于证书的密码匹配。

### <a name="windows-using-windows-containers"></a>使用 Windows 容器的 windows

生成证书并配置本地计算机：

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

在上述命令中，将替换 `{ password here }` 为密码。 使用 [PowerShell](/powershell/scripting/overview)时，请将替换 `%USERPROFILE%` 为 `$env:USERPROFILE` 。

运行容器映像，并为 HTTPS 配置 ASP.NET Core：

```console
docker pull mcr.microsoft.com/dotnet/core/samples:aspnetapp
docker run --rm -it -p 8000:80 -p 8001:443 -e ASPNETCORE_URLS="https://+;http://+" -e ASPNETCORE_HTTPS_PORT=8001 -e ASPNETCORE_Kestrel__Certificates__Default__Password="password" -e ASPNETCORE_Kestrel__Certificates__Default__Path=\https\aspnetapp.pfx -v %USERPROFILE%\.aspnet\https:C:\https\ mcr.microsoft.com/dotnet/core/samples:aspnetapp
```

密码必须与用于证书的密码匹配。 使用 [PowerShell](/powershell/scripting/overview)时，请将替换 `%USERPROFILE%` 为 `$env:USERPROFILE` 。