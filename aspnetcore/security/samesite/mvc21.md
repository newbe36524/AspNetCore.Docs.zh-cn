---
title: ASP.NET Core 2.1 MVC SameSite cookie 示例
author: rick-anderson
description: ASP.NET Core 2.1 MVC SameSite cookie 示例
monikerRange: '>= aspnetcore-2.1 < aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 12/03/2019
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
uid: security/samesite/mvc21
ms.openlocfilehash: 38e5f0d1a2ecf5b46f73bf8574f73934a070880f
ms.sourcegitcommit: 24106b7ffffc9fff410a679863e28aeb2bbe5b7e
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 09/17/2020
ms.locfileid: "90722600"
---
# <a name="aspnet-core-21-mvc-samesite-no-loccookie-sample"></a>ASP.NET Core 2.1 MVC SameSite cookie 示例

ASP.NET Core 2.1 内置了对 [SameSite](https://www.owasp.org/index.php/SameSite) 属性的支持，但它已写入原始标准。 修补后的 [行为](https://github.com/dotnet/aspnetcore/issues/8212) 更改了的含义 `SameSite.None` ，以发出值为的 sameSite 特性 `None` ，而不是根本不发出值。 如果您不希望发出该值，则可以将属性设置 `SameSite` cookie 为-1。

[!INCLUDE[](~/includes/SameSiteIdentity.md)]

## <a name="writing-the-samesite-attribute"></a><a name="sampleCode"></a>编写 SameSite 属性

下面是如何在上编写 SameSite 属性的示例 cookie ：

```c#
var cookieOptions = new CookieOptions
{
    // Set the secure flag, which Chrome's changes will require for SameSite none.
    // Note this will also require you to be running on HTTPS
    Secure = true,

    // Set the cookie to HTTP only which is good practice unless you really do need
    // to access it client side in scripts.
    HttpOnly = true,

    // Add the SameSite attribute, this will emit the attribute with a value of none.
    // To not emit the attribute at all set the SameSite property to (SameSiteMode)(-1).
    SameSite = SameSiteMode.None
};

// Add the cookie to the response cookie collection
Response.Cookies.Append(CookieName, "cookieValue", cookieOptions);
```

## <a name="setting-no-loccookie-authentication-and-session-state-no-loccookies"></a>设置 Cookie 身份验证和会话状态 cookie

Cookie 身份验证、会话状态和 [各种其他组件](../samesite.md?view=aspnetcore-2.1) 通过选项设置其 sameSite 选项 Cookie ，例如

```c#
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.SameSite = SameSiteMode.None;
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
        options.Cookie.IsEssential = true;
    });

services.AddSession(options =>
{
    options.Cookie.SameSite = SameSiteMode.None;
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.IsEssential = true;
});
```

在前面的代码中， cookie 身份验证和会话状态将其 sameSite 属性设置为 `None` ，发出具有值的属性， `None` 同时将 Secure 特性设置为 true。

### <a name="run-the-sample"></a>运行示例

如果运行 [示例项目](https://github.com/blowdart/AspNetSameSiteSamples/tree/master/AspNetCore21MVC)，请在初始页面上加载浏览器调试器，并使用它查看站点的 cookie 集合。 若要在 Edge 和 Chrome 中进行此操作，请按， `F12` 选择 `Application` 选项卡，然后单击节中的选项下的 "站点 URL" `Cookies` `Storage` 。

![Browser 调试器：：： no (Cookie) ：：： List](BrowserDebugger.png)

cookie当你单击 "创建 SameSite" 按钮的 SameSite 属性值为时，你可以从上图中看到该示例所创建的，这与 Cookie `Lax` [示例代码](#sampleCode)中设置的值匹配。

## <a name="intercepting-no-loccookies"></a><a name="interception"></a>截获 cookie

为了截获 cookie ，若要根据用户的浏览器代理中的支持来调整无值，必须使用 `CookiePolicy` 中间件。 在中写入和配置的任何组件 **之前** ，必须将其放入 http 请求管道 cookie `ConfigureServices()` 。

若要将其插入管道中，请在 Startup.cs 的方法中使用 `app.UseCookiePolicy()` `Configure(IApplicationBuilder, IHostingEnvironment)` 。 [Startup.cs](https://github.com/blowdart/AspNetSameSiteSamples/blob/master/AspNetCore21MVC/Startup.cs) 例如：

```c#
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
       app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Home/Error");
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseCookiePolicy();
    app.UseAuthentication();
    app.UseSession();

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
```

然后在中，将 `ConfigureServices(IServiceCollection services)` cookie 策略配置为在 cookie 附加或删除时调用帮助器类。 例如：

```c#
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
        options.OnAppendCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
        options.OnDeleteCookie = cookieContext =>
            CheckSameSite(cookieContext.Context, cookieContext.CookieOptions);
    });
}

private void CheckSameSite(HttpContext httpContext, CookieOptions options)
{
    if (options.SameSite == SameSiteMode.None)
    {
        var userAgent = httpContext.Request.Headers["User-Agent"].ToString();
        if (SameSite.BrowserDetection.DisallowsSameSiteNone(userAgent))
        {
            options.SameSite = (SameSiteMode)(-1);
        }
    }
}
```

Helper 函数 `CheckSameSite(HttpContext, CookieOptions)` ：

* 当 cookie 向请求追加或从请求中删除时，将调用。
* 检查 `SameSite` 属性是否设置为 `None` 。
* 如果将 `SameSite` 设置为 `None` ，并且已知当前用户代理不支持 none 特性值，则为。 使用 [SameSiteSupport](https://github.com/dotnet/AspNetCore.Docs/tree/master/aspnetcore/security/samesite/sample/snippets/SameSiteSupport.cs) 类完成检查：
  * 设置 `SameSite` 为不通过将属性设置为来发出该值 `(SameSiteMode)(-1)`

## <a name="targeting-net-framework"></a>目标 .NET Framework

ASP.NET Core 和 System.web (ASP.NET 经典) 具有独立的 SameSite 实现。 如果使用 ASP.NET Core 或 ( .NET 4.7.2) 适用于 ASP.NET Core，则不需要 .NET Framework 的 SameSite KB 修补程序。

.NET 上的 ASP.NET Core 要求更新 nuget 程序包依赖项以获取适当的修补程序。

若要获取的 ASP.NET Core 更改 .NET Framework 确保你对修补的包以及版本 (2.1.14 或更高版本2.1 版本) 直接引用。

```xml
<PackageReference Include="Microsoft.Net.Http.Headers" Version="2.1.14" />
<PackageReference Include="Microsoft.AspNetCore.CookiePolicy" Version="2.1.14" />
```

### <a name="more-information"></a>更多信息
 
[Chrome 更新](https://www.chromium.org/updates/same-site) 
[ASP.NET Core SameSite 文档](../samesite.md?view=aspnetcore-2.1) 
[ASP.NET Core 2.1 SameSite 更改公告](https://github.com/dotnet/aspnetcore/issues/8212)