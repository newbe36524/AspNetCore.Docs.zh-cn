---
title: 自定义 AuthorizationMiddleware 的行为
author: pranavkm
ms.author: prkrishn
description: 本文介绍如何自定义 AuthorizationMiddleware 的结果处理。
monikerRange: '>= aspnetcore-5.0'
uid: security/authorization/authorizationmiddlewareresulthandler
ms.openlocfilehash: 55f4433c080d6ce6676ca1072dacacc137f15638
ms.sourcegitcommit: b3ec60f7682e43211c2b40c60eab3d4e45a48ab1
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/17/2020
ms.locfileid: "92156765"
---
# <a name="customize-the-behavior-of-authorizationmiddleware"></a>自定义 AuthorizationMiddleware 的行为

应用程序可以注册 `Microsoft.AspNetCore.Authorization.IAuthorizationMiddlewareResultHandler` ，以自定义中间件处理授权结果的方式。 应用程序可以使用自定义中间件来执行以下操作：

* 返回自定义响应。
* 提高默认质询或禁止响应。

下面的代码显示了授权处理程序的示例，该示例为某些类型的授权失败返回自定义响应：

[!code-csharp[](customizingauthorizationmiddlewareresponse/sample/AuthorizationMiddlewareResultHandlerSample/MyAuthorizationMiddlewareResultHandler.cs)]

注册 `MyAuthorizationMiddlewareResultHandler` 到 `Startup.ConfigureServices` ：

[!code-csharp[](customizingauthorizationmiddlewareresponse/sample/AuthorizationMiddlewareResultHandlerSample/Startup.cs?name=snippet)]

<!-- <xref:Microsoft.AspNetCore.Authorization.IAuthorizationMiddlewareResultHandler /> -->