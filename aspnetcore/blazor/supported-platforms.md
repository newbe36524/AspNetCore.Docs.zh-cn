---
title: ASP.NET Core Blazor 支持的平台
author: guardrex
description: 了解 ASP.NET Core Blazor 支持的平台。
monikerRange: '>= aspnetcore-3.1'
ms.author: riande
ms.custom: mvc
ms.date: 10/01/2020
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
uid: blazor/supported-platforms
ms.openlocfilehash: 1ffe98636ed200adbf00e89c2c3499eb69792d3f
ms.sourcegitcommit: d60bfd52bfb559e805abd654b87a2a0c7eb69cf8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/06/2020
ms.locfileid: "91754536"
---
# <a name="aspnet-core-no-locblazor-supported-platforms"></a>ASP.NET Core Blazor 支持的平台

作者：[Luke Latham](https://github.com/guardrex)

::: moniker range=">= aspnetcore-5.0"

下表中所示的浏览器支持 Blazor WebAssembly 和 Blazor Server。

| 浏览者                          | Version         |
| -------------------------------- | --------------- |
| Apple Safari，包括 iOS      | 当前&dagger; |
| Google Chrome，包括 Android | 当前&dagger; |
| Microsoft Edge                   | 当前&dagger; |
| Mozilla Firefox                  | 当前&dagger; |  

&dagger;最新指的是浏览器的最新版本。  

::: moniker-end

::: moniker range="< aspnetcore-5.0"

## Blazor WebAssembly

| 浏览者                          | Version               |
| -------------------------------- | --------------------- |
| Apple Safari，包括 iOS      | 当前&dagger;       |
| Google Chrome，包括 Android | 当前&dagger;       |
| Microsoft Edge                   | 当前&dagger;       |
| Microsoft Internet Explorer      | 不支持&Dagger; |
| Mozilla Firefox                  | 当前&dagger;       |  

&dagger;最新指的是浏览器的最新版本。  
&Dagger;Microsoft Internet Explorer 不支持 [WebAssembly](https://webassembly.org)。

## Blazor Server

| 浏览者                          | Version         |
| -------------------------------- | --------------- |
| Apple Safari，包括 iOS      | 当前&dagger; |
| Google Chrome，包括 Android | 当前&dagger; |
| Microsoft Edge                   | 当前&dagger; |
| Microsoft Internet Explorer      | 11&Dagger;      |
| Mozilla Firefox                  | 当前&dagger; |

&dagger;最新指的是浏览器的最新版本。  
&Dagger;需要其他填充代码。 例如，可通过 [`Polyfill.io`](https://polyfill.io/v3/) 捆绑包添加承诺。

::: moniker-end

## <a name="additional-resources"></a>其他资源

* <xref:blazor/hosting-models>
* <xref:signalr/supported-platforms>
