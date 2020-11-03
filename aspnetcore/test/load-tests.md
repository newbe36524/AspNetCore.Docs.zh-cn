---
title: ASP.NET Core 负载/压力测试
author: Jeremy-Meng
description: 了解针对 ASP.NET Core 应用进行负载测试和压力测试的几种实用工具和方法。
ms.author: riande
ms.custom: mvc
ms.date: 4/05/2019
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
uid: test/loadtests
ms.openlocfilehash: 8e3ca41312922cbf44361601c38e455b342e9fe1
ms.sourcegitcommit: b5ebaf42422205d212e3dade93fcefcf7f16db39
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/22/2020
ms.locfileid: "88632806"
---
# <a name="aspnet-core-loadstress-testing"></a>ASP.NET Core 负载/压力测试

负载测试和压力测试对于确保 web 应用的性能和可缩放性非常重要。 尽管它们的某些测试是相同的，但目标不同。

负载测试****：测试应用是否可以在特定情况下处理指定的用户负载，同时仍满足响应目标。 应用在正常状态下运行。

压力测试****：在极端条件下（通常为长时间）运行时测试应用的稳定性。 测试会对应用施加高用户负载（峰值或逐渐增加的负载）或限制应用的计算资源。

压力测试可确定压力下的应用是否能够从故障中恢复，并正常返回到预期的行为。 在压力下，应用不会在正常状态下运行。

Visual Studio 2019 宣布计划[取消负载测试](https://devblogs.microsoft.com/devops/cloud-based-load-testing-service-eol/)。 已关闭相应的 Azure DevOps 基于云的负载测试服务。

## <a name="third-party-tools"></a>第三方工具

以下列表包含具有各种功能集的第三方 web 性能工具：

* [Apache JMeter](https://jmeter.apache.org/)
* [ApacheBench (ab)](https://httpd.apache.org/docs/2.4/programs/ab.html)
* [Gatling](https://gatling.io/)
* [k6](https://k6.io)
* [Locust](https://locust.io/)
* [West Wind WebSurge](https://websurge.west-wind.com/)
* [Netling](https://github.com/hallatore/Netling)
* [Vegeta](https://github.com/tsenart/vegeta)