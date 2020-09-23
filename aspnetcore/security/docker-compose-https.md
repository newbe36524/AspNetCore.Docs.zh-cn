---
title: 使用 docker 组合通过 HTTPS 在容器中托管 ASP.NET Core 映像
author: ravipal
description: 了解如何通过 HTTPS Docker Compose 宿主 ASP.NET Core 映像
monikerRange: '>= aspnetcore-2.1'
ms.author: ravipal
ms.custom: mvc
ms.date: 03/28/2020
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
uid: security/docker-compose-https
ms.openlocfilehash: 71ead7dcce32dab090a9b0b3573aa855d00fa7f1
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722756"
---
# <a name="hosting-aspnet-core-images-with-docker-compose-over-https"></a>通过 HTTPS Docker Compose 宿主 ASP.NET Core 映像


[默认情况下](./enforcing-ssl.md)，ASP.NET Core 使用 HTTPS。 [HTTPS](https://en.wikipedia.org/wiki/HTTPS) 依赖于信任、标识和加密的 [证书](https://en.wikipedia.org/wiki/Public_key_certificate) 。

本文档介绍如何通过 HTTPS 运行预生成的容器映像。

若要开发方案，请参阅 [通过 HTTPS 上的 Docker 开发 ASP.NET Core 应用程序](https://github.com/dotnet/dotnet-docker/blob/master/samples/run-aspnetcore-https-development.md) 。

此示例需要 docker [17.06](https://docs.docker.com/release-notes/docker-ce) 或更高版本的 [docker 客户端](https://www.docker.com/products/docker)。

## <a name="prerequisites"></a>先决条件

本文档中的某些说明需要 [.Net Core 2.2 SDK](https://dotnet.microsoft.com/download) 或更高版本。

## <a name="certificates"></a>证书

针对域的[生产主机](https://blogs.msdn.microsoft.com/webdev/2017/11/29/configuring-https-in-asp-net-core-across-different-platforms/)需要[证书颁发机构颁发](https://wikipedia.org/wiki/Certificate_authority)的证书。 [Let's Encrypt](https://letsencrypt.org/) 是提供免费证书的证书颁发机构。

本文档使用 [自签名开发证书](https://wikipedia.org/wiki/Self-signed_certificate) 来托管预生成的映像 `localhost` 。 说明类似于使用生产证书。

对于生产证书：

* 此 `dotnet dev-certs` 工具不是必需的。
* 无需将证书存储在说明中使用的位置。 将证书存储在站点目录之外的任何位置。

以下部分中包含的说明使用 `volumes` *docker-compose.yml. docker-compose.override.yml*中的属性将证书装载到容器中。 可以使用 Dockerfile 中的命令将证书添加到容器映像 `COPY` 中，但不建议这样做。 *Dockerfile* 由于以下原因，不建议将证书复制到映像：

* 这使得使用开发人员证书进行测试变得困难。
* 这使得难以使用同一个映像来托管生产证书。
* 证书泄露存在重大风险。

## <a name="starting-a-container-with-https-support-using-docker-compose"></a>使用 docker 合成启动支持 https 的容器

使用以下有关操作系统配置的说明。

### <a name="windows-using-linux-containers"></a>使用 Linux 容器的 Windows

生成证书并配置本地计算机：

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

在上述命令中，将替换 `{ password here }` 为密码。

创建包含以下内容的 _docker-compose.yml docker-compose.override.yml_ 文件：

```json
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
Docker 撰写文件中指定的密码必须与用于证书的密码匹配。

启动容器，其中包含为 HTTPS 配置的 ASP.NET Core：

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="macos-or-linux"></a>macOS 或 Linux

生成证书并配置本地计算机：

```dotnetcli
dotnet dev-certs https -ep ${HOME}/.aspnet/https/aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

`dotnet dev-certs https --trust` 仅在 macOS 和 Windows 上受支持。 你需要以分发所支持的方式在 Linux 上信任证书。 可能需要在浏览器中信任该证书。

在上述命令中，将替换 `{ password here }` 为密码。

创建包含以下内容的 _docker-compose.yml docker-compose.override.yml_ 文件：

```json
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=/https/aspnetapp.pfx
    volumes:
      - ~/.aspnet/https:/https:ro
```
Docker 撰写文件中指定的密码必须与用于证书的密码匹配。

启动容器，其中包含为 HTTPS 配置的 ASP.NET Core：

```console
docker-compose -f "docker-compose.debug.yml" up -d
```

### <a name="windows-using-windows-containers"></a>使用 Windows 容器的 windows

生成证书并配置本地计算机：

```dotnetcli
dotnet dev-certs https -ep %USERPROFILE%\.aspnet\https\aspnetapp.pfx -p { password here }
dotnet dev-certs https --trust
```

在上述命令中，将替换 `{ password here }` 为密码。

创建包含以下内容的 _docker-compose.yml docker-compose.override.yml_ 文件：

```json
version: '3.4'

services:
  webapp:
    image: mcr.microsoft.com/dotnet/core/samples:aspnetapp
    ports:
      - 80
      - 443
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ASPNETCORE_URLS=https://+:443;http://+:80
      - ASPNETCORE_Kestrel__Certificates__Default__Password=password
      - ASPNETCORE_Kestrel__Certificates__Default__Path=C:\https\aspnetapp.pfx
    volumes:
      - ${USERPROFILE}\.aspnet\https:C:\https:ro
```
Docker 撰写文件中指定的密码必须与用于证书的密码匹配。

启动容器，其中包含为 HTTPS 配置的 ASP.NET Core：

```console
docker-compose -f "docker-compose.debug.yml" up -d
```