---
title: 在 ASP.NET Core 中配置证书身份验证
author: blowdart
description: 了解如何在 ASP.NET Core for IIS 和 HTTP.sys 中配置证书身份验证。
monikerRange: '>= aspnetcore-3.0'
ms.author: bdorrans
ms.date: 07/16/2020
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
uid: security/authentication/certauth
ms.openlocfilehash: 83525a4c1e87a60b57130c1bba14360c7d03f552
ms.sourcegitcommit: ca34c1ac578e7d3daa0febf1810ba5fc74f60bbf
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/30/2020
ms.locfileid: "93061374"
---
# <a name="configure-certificate-authentication-in-aspnet-core"></a>在 ASP.NET Core 中配置证书身份验证

`Microsoft.AspNetCore.Authentication.Certificate` 包含类似于 ASP.NET Core 的 [证书身份验证](https://tools.ietf.org/html/rfc5246#section-7.4.4) 的实现。 证书身份验证在 TLS 级别发生，远在到达 ASP.NET Core 之前。 更准确地说，这是验证证书的身份验证处理程序，然后向你提供可将该证书解析到的事件 `ClaimsPrincipal` 。 

将[服务器配置](#configure-your-server-to-require-certificates)为使用证书进行身份验证，如 IIS、Kestrel、Azure Web 应用，或者使用任何其他方法。

## <a name="proxy-and-load-balancer-scenarios"></a>代理和负载均衡器方案

证书身份验证是一种有状态方案，主要用于代理或负载均衡器不处理客户端和服务器之间的流量。 如果使用代理或负载平衡器，则仅当代理或负载均衡器：

* 处理身份验证。
* 将用户身份验证信息传递给应用 (例如，在请求标头) 中，它对身份验证信息起作用。

在使用代理和负载平衡器的环境中证书身份验证的替代方法是 Active Directory 联合服务 (ADFS) 与 OpenID Connect (OIDC) 。

## <a name="get-started"></a>入门

获取并应用 HTTPS 证书，并将 [服务器配置](#configure-your-server-to-require-certificates) 为需要证书。

在 web 应用中，添加对 [AspNetCore](https://www.nuget.org/packages/Microsoft.AspNetCore.Authentication.Certificate) 包的引用。 然后在 `Startup.ConfigureServices` 方法中， `services.AddAuthentication(CertificateAuthenticationDefaults.AuthenticationScheme).AddCertificate(...);` 使用你的选项调用，同时提供一个委托，用于对 `OnCertificateValidated` 随请求发送的客户端证书进行任何补充验证。 将该信息转换为 `ClaimsPrincipal` 并在属性上设置 `context.Principal` 。

如果身份验证失败，此处理程序将 `403 (Forbidden)` `401 (Unauthorized)` 像你所料，返回响应，而不是。 原因是，在初次 TLS 连接期间应进行身份验证。 当它到达处理程序时，它的时间太晚。 无法将连接从匿名连接升级到证书。

还会 `app.UseAuthentication();` 在方法中添加 `Startup.Configure` 。 否则， `HttpContext.User` 将不会设置为 `ClaimsPrincipal` 从证书创建。 例如： 。

::: moniker range=">= aspnetcore-5.0"

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate()
        // Adding an ICertificateValidationCache results in certificate auth caching the results.
        // The default implementation uses a memory cache.
        .AddCertificateCache();

    // All other service configuration
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();

    // All other app configuration
}
```

::: moniker-end

::: moniker range="< aspnetcore-5.0"

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
        .AddCertificate();

    // All other service configuration
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();

    // All other app configuration
}
```

::: moniker-end

前面的示例演示了添加证书身份验证的默认方法。 处理程序使用通用证书属性构造用户主体。

## <a name="configure-certificate-validation"></a>配置证书验证

`CertificateAuthenticationOptions`处理程序具有一些内置的验证，这些验证是你应在证书上执行的最小验证。 默认情况下，将启用这些设置中的每一个。

### <a name="allowedcertificatetypes--chained-selfsigned-or-all-chained--selfsigned"></a>AllowedCertificateTypes = 链式、SelfSigned 或 All (链式 |SelfSigned) 

默认值：30`CertificateTypes.Chained`

此检查将验证是否只允许使用适当的证书类型。 如果应用使用自签名证书，则需要将此选项设置为 `CertificateTypes.All` 或 `CertificateTypes.SelfSigned` 。

### <a name="validatecertificateuse"></a>ValidateCertificateUse

默认值：30`true`

此检查将验证客户端提供的证书是否 (EKU) 使用了客户端身份验证扩展密钥，或者根本没有 Eku。 如规范所示，如果未指定 EKU，则所有 Eku 均视为有效。

### <a name="validatevalidityperiod"></a>ValidateValidityPeriod

默认值：30`true`

此检查将验证证书是否在其有效期内。 对于每个请求，处理程序将确保在其在其当前会话期间提供的证书过期时，该证书是有效的。

### <a name="revocationflag"></a>RevocationFlag

默认值：30`X509RevocationFlag.ExcludeRoot`

一个标志，该标志指定将检查链中的哪些证书进行吊销。

仅当证书链接到根证书时才执行吊销检查。

### <a name="revocationmode"></a>RevocationMode

默认值：30`X509RevocationMode.Online`

指定如何执行吊销检查的标志。

指定联机检查可能会导致长时间延迟，同时与证书颁发机构联系。

仅当证书链接到根证书时才执行吊销检查。

### <a name="can-i-configure-my-app-to-require-a-certificate-only-on-certain-paths"></a>我是否可以将我的应用程序配置为只需要特定路径上的证书？

这是不可能的。 请记住，证书交换已完成 HTTPS 会话的启动，在该连接上收到第一个请求之前，服务器会完成此操作，因此无法基于任何请求字段进行作用域。

## <a name="handler-events"></a>处理程序事件

处理程序有两个事件：

* `OnAuthenticationFailed`：如果在身份验证过程中发生异常，则调用，并允许您做出反应。
* `OnCertificateValidated`：在验证证书后调用，并已创建默认主体。 此事件允许你执行自己的验证并增加或替换主体。 例如：
  * 确定你的服务是否知道该证书。
  * 构造自己的主体。 请看下面 `Startup.ConfigureServices` 中的示例：

```csharp
services.AddAuthentication(
    CertificateAuthenticationDefaults.AuthenticationScheme)
    .AddCertificate(options =>
    {
        options.Events = new CertificateAuthenticationEvents
        {
            OnCertificateValidated = context =>
            {
                var claims = new[]
                {
                    new Claim(
                        ClaimTypes.NameIdentifier, 
                        context.ClientCertificate.Subject,
                        ClaimValueTypes.String, 
                        context.Options.ClaimsIssuer),
                    new Claim(ClaimTypes.Name,
                        context.ClientCertificate.Subject,
                        ClaimValueTypes.String, 
                        context.Options.ClaimsIssuer)
                };

                context.Principal = new ClaimsPrincipal(
                    new ClaimsIdentity(claims, context.Scheme.Name));
                context.Success();

                return Task.CompletedTask;
            }
        };
    });
```

如果发现入站证书不符合额外的验证，请调用 `context.Fail("failure reason")` 失败原因。

对于实际功能，你可能需要调用在依赖关系注入中注册的服务，该服务连接到数据库或其他类型的用户存储。 使用传递到委托中的上下文访问你的服务。 请看下面 `Startup.ConfigureServices` 中的示例：

```csharp
services.AddAuthentication(
    CertificateAuthenticationDefaults.AuthenticationScheme)
    .AddCertificate(options =>
    {
        options.Events = new CertificateAuthenticationEvents
        {
            OnCertificateValidated = context =>
            {
                var validationService =
                    context.HttpContext.RequestServices
                        .GetRequiredService<ICertificateValidationService>();
                
                if (validationService.ValidateCertificate(
                    context.ClientCertificate))
                {
                    var claims = new[]
                    {
                        new Claim(
                            ClaimTypes.NameIdentifier, 
                            context.ClientCertificate.Subject, 
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer),
                        new Claim(
                            ClaimTypes.Name, 
                            context.ClientCertificate.Subject, 
                            ClaimValueTypes.String, 
                            context.Options.ClaimsIssuer)
                    };

                    context.Principal = new ClaimsPrincipal(
                        new ClaimsIdentity(claims, context.Scheme.Name));
                    context.Success();
                }                     

                return Task.CompletedTask;
            }
        };
    });
```

从概念上讲，验证证书是一种授权问题。 例如，在授权策略中添加一个颁发者或指纹，而不是在中， `OnCertificateValidated` 这是完全可以接受的。

## <a name="configure-your-server-to-require-certificates"></a>将服务器配置为需要证书

### <a name="kestrel"></a>Kestrel

在 *Program.cs* 中，按如下所示配置 Kestrel：

```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args)
{
    return Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
            webBuilder.ConfigureKestrel(o =>
            {
                o.ConfigureHttpsDefaults(o => 
            o.ClientCertificateMode = 
                ClientCertificateMode.RequireCertificate);
            });
        });
}
```

> [!NOTE]
> 通过在调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.ConfigureHttpsDefaults*> 之前调用 <xref:Microsoft.AspNetCore.Server.Kestrel.Core.KestrelServerOptions.Listen*> 创建的终结点将不会应用默认值。

### <a name="iis"></a>IIS

在 IIS 管理器中完成以下步骤：

1. 从 " **连接** " 选项卡中选择你的站点。
1. 双击 " **功能视图** " 窗口中的 " **SSL 设置** " 选项。
1. 选中 " **需要 SSL** " 复选框，并选择 " **客户端证书** " 部分中的 " **要求** " 单选按钮。

![IIS 中的客户端证书设置](README-IISConfig.png)

### <a name="azure-and-custom-web-proxies"></a>Azure 和自定义 web 代理

有关如何配置证书转发中间件的详细 [说明，请参阅托管和部署文档](xref:host-and-deploy/proxy-load-balancer#certificate-forwarding) 。

### <a name="use-certificate-authentication-in-azure-web-apps"></a>在 Azure Web 应用中使用证书身份验证

Azure 不需要转发配置。 此设置已在证书转发中间件中进行设置。

> [!NOTE]
> 这要求存在 CertificateForwardingMiddleware。

### <a name="use-certificate-authentication-in-custom-web-proxies"></a>在自定义 web 代理中使用证书身份验证

`AddCertificateForwarding`方法用于指定：

* 客户端标头名称。
* 如何使用属性)  (加载证书 `HeaderConverter` 。

例如，在自定义 web 代理中，证书作为自定义请求标头传递 `X-SSL-CERT` 。 若要使用它，请在中配置证书转发 `Startup.ConfigureServices` ：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddCertificateForwarding(options =>
    {
        options.CertificateHeader = "X-SSL-CERT";
        options.HeaderConverter = (headerValue) =>
        {
            X509Certificate2 clientCertificate = null;
        
            if(!string.IsNullOrWhiteSpace(headerValue))
            {
                byte[] bytes = StringToByteArray(headerValue);
                clientCertificate = new X509Certificate2(bytes);
            }

            return clientCertificate;
        };
    });
}

private static byte[] StringToByteArray(string hex)
{
    int NumberChars = hex.Length;
    byte[] bytes = new byte[NumberChars / 2];

    for (int i = 0; i < NumberChars; i += 2)
    {
        bytes[i / 2] = Convert.ToByte(hex.Substring(i, 2), 16);
    }

    return bytes;
}
```

然后，该 `Startup.Configure` 方法将添加中间件。 `UseCertificateForwarding` 调用和之前调用 `UseAuthentication` `UseAuthorization` ：

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    ...

    app.UseRouting();

    app.UseCertificateForwarding();
    app.UseAuthentication();
    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

单独的类可用于实现验证逻辑。 由于本示例中使用了相同的自签名证书，因此请确保只能使用证书。 验证客户端证书和服务器证书的指纹是否匹配，否则，任何证书都可以使用，并且足以进行身份验证。 这将在方法中使用 `AddCertificate` 。 如果使用的是中间证书或子证书，也可以在此处验证使用者或颁发者。

```csharp
using System.IO;
using System.Security.Cryptography.X509Certificates;

namespace AspNetCoreCertificateAuthApi
{
    public class MyCertificateValidationService
    {
        public bool ValidateCertificate(X509Certificate2 clientCertificate)
        {
            // Do not hardcode passwords in production code
            // Use thumbprint or key vault
            var cert = new X509Certificate2(
                Path.Combine("sts_dev_cert.pfx"), "1234");

            if (clientCertificate.Thumbprint == cert.Thumbprint)
            {
                return true;
            }

            return false;
        }
    }
}
```

#### <a name="implement-an-httpclient-using-a-certificate-and-the-httpclienthandler"></a>使用证书和 HttpClientHandler 实现 HttpClient

`HttpClientHandler`可以直接在类的构造函数中添加 `HttpClient` 。 创建实例时应小心谨慎 `HttpClient` 。 `HttpClient`然后，将随每个请求发送证书。

```csharp
private async Task<JsonDocument> GetApiDataUsingHttpClientHandler()
{
    var cert = new X509Certificate2(Path.Combine(_environment.ContentRootPath, "sts_dev_cert.pfx"), "1234");
    var handler = new HttpClientHandler();
    handler.ClientCertificates.Add(cert);
    var client = new HttpClient(handler);
     
    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:44379/api/values"),
        Method = HttpMethod.Get,
    };
    var response = await client.SendAsync(request);
    if (response.IsSuccessStatusCode)
    {
        var responseContent = await response.Content.ReadAsStringAsync();
        var data = JsonDocument.Parse(responseContent);
        return data;
    }
 
    throw new ApplicationException($"Status code: {response.StatusCode}, Error: {response.ReasonPhrase}");
}
```

#### <a name="implement-an-httpclient-using-a-certificate-and-a-named-httpclient-from-ihttpclientfactory"></a>使用证书和 IHttpClientFactory 中的命名 HttpClient 实现 HttpClient 

在下面的示例中， `HttpClientHandler` 使用来自处理程序的属性将客户端证书添加到 `ClientCertificates` 。 然后，可以使用方法在的命名实例中使用此处理程序 `HttpClient` `ConfigurePrimaryHttpMessageHandler` 。 这是在中设置的 `Startup.ConfigureServices` ：

```csharp
var clientCertificate = 
    new X509Certificate2(
      Path.Combine(_environment.ContentRootPath, "sts_dev_cert.pfx"), "1234");
 
var handler = new HttpClientHandler();
handler.ClientCertificates.Add(clientCertificate);
 
services.AddHttpClient("namedClient", c =>
{
}).ConfigurePrimaryHttpMessageHandler(() => handler);
```

`IHttpClientFactory`然后，可以使用处理程序和证书获取命名实例。 使用 `CreateClient` 在类中定义的客户端名称的方法 `Startup` 来获取实例。 可根据需要使用客户端发送 HTTP 请求。

```csharp
private readonly IHttpClientFactory _clientFactory;
 
public ApiService(IHttpClientFactory clientFactory)
{
    _clientFactory = clientFactory;
}
 
private async Task<JsonDocument> GetApiDataWithNamedClient()
{
    var client = _clientFactory.CreateClient("namedClient");
 
    var request = new HttpRequestMessage()
    {
        RequestUri = new Uri("https://localhost:44379/api/values"),
        Method = HttpMethod.Get,
    };
    var response = await client.SendAsync(request);
    if (response.IsSuccessStatusCode)
    {
        var responseContent = await response.Content.ReadAsStringAsync();
        var data = JsonDocument.Parse(responseContent);
        return data;
    }
 
    throw new ApplicationException($"Status code: {response.StatusCode}, Error: {response.ReasonPhrase}");
}
```

如果将正确的证书发送到服务器，则返回数据。 如果未发送证书或证书不正确，则返回 HTTP 403 状态代码。

### <a name="create-certificates-in-powershell"></a>在 PowerShell 中创建证书

创建证书是最难设置此流的部分。 可以使用 PowerShell cmdlet 创建根证书 `New-SelfSignedCertificate` 。 创建证书时，请使用强密码。 `KeyUsageProperty`如图所示，添加参数和参数非常重要 `KeyUsage` 。

#### <a name="create-root-ca"></a>创建根 CA

```powershell
New-SelfSignedCertificate -DnsName "root_ca_dev_damienbod.com", "root_ca_dev_damienbod.com" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(20) -FriendlyName "root_ca_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\root_ca_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath root_ca_dev_damienbod.crt
```

> [!NOTE]
> `-DnsName`参数值必须与应用的部署目标匹配。 例如，用于开发的 "localhost"。

#### <a name="install-in-the-trusted-root"></a>在受信任的根中安装

根证书需要在主机系统上受信任。 默认情况下，不信任证书颁发机构创建的根证书。 下面的链接说明了如何在 Windows 上完成此操作：

https://social.msdn.microsoft.com/Forums/SqlServer/5ed119ef-1704-4be4-8a4f-ef11de7c8f34/a-certificate-chain-processed-but-terminated-in-a-root-certificate-which-is-not-trusted-by-the

#### <a name="intermediate-certificate"></a>中间证书

现在可以从根证书创建中间证书。 这并不是所有用例所必需的，但你可能需要创建多个证书，或者需要激活或禁用证书组。 `TextExtension`参数是设置证书的基本约束中的路径长度所必需的。

然后，可以将中间证书添加到 Windows 主机系统中的受信任中间证书。

```powershell
$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint of the root..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "intermediate_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "intermediate_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature -TextExtension @("2.5.29.19={text}CA=1&pathlength=1")

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\intermediate_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath intermediate_dev_damienbod.crt
```

#### <a name="create-child-certificate-from-intermediate-certificate"></a>从中间证书创建子证书

可以从中间证书创建子证书。 这是最终实体，不需要创建更多的子证书。

```powershell
$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint from the Intermediate certificate..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com"

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath child_a_dev_damienbod.crt
```

#### <a name="create-child-certificate-from-root-certificate"></a>从根证书创建子证书

还可以直接从根证书创建子证书。 

```powershell
$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\"The thumbprint from the root cert..." )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $rootcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com"

$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

Get-ChildItem -Path cert:\localMachine\my\"The thumbprint..." | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\"The thumbprint..." -FilePath child_a_dev_damienbod.crt
```

#### <a name="example-root---intermediate-certificate---certificate"></a>示例根中间证书证书

```powershell
$mypwdroot = ConvertTo-SecureString -String "1234" -Force -AsPlainText
$mypwd = ConvertTo-SecureString -String "1234" -Force -AsPlainText

New-SelfSignedCertificate -DnsName "root_ca_dev_damienbod.com", "root_ca_dev_damienbod.com" -CertStoreLocation "cert:\LocalMachine\My" -NotAfter (Get-Date).AddYears(20) -FriendlyName "root_ca_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature

Get-ChildItem -Path cert:\localMachine\my\0C89639E4E2998A93E423F919B36D4009A0F9991 | Export-PfxCertificate -FilePath C:\git\root_ca_dev_damienbod.pfx -Password $mypwdroot

Export-Certificate -Cert cert:\localMachine\my\0C89639E4E2998A93E423F919B36D4009A0F9991 -FilePath root_ca_dev_damienbod.crt

$rootcert = ( Get-ChildItem -Path cert:\LocalMachine\My\0C89639E4E2998A93E423F919B36D4009A0F9991 )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_a_dev_damienbod.com" -Signer $rootcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_a_dev_damienbod.com" -KeyUsageProperty All -KeyUsage CertSign, CRLSign, DigitalSignature -TextExtension @("2.5.29.19={text}CA=1&pathlength=1")

Get-ChildItem -Path cert:\localMachine\my\BA9BF91ED35538A01375EFC212A2F46104B33A44 | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\BA9BF91ED35538A01375EFC212A2F46104B33A44 -FilePath child_a_dev_damienbod.crt

$parentcert = ( Get-ChildItem -Path cert:\LocalMachine\My\BA9BF91ED35538A01375EFC212A2F46104B33A44 )

New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "child_b_from_a_dev_damienbod.com" -Signer $parentcert -NotAfter (Get-Date).AddYears(20) -FriendlyName "child_b_from_a_dev_damienbod.com" 

Get-ChildItem -Path cert:\localMachine\my\141594A0AE38CBBECED7AF680F7945CD51D8F28A | Export-PfxCertificate -FilePath C:\git\AspNetCoreCertificateAuth\Certs\child_b_from_a_dev_damienbod.pfx -Password $mypwd

Export-Certificate -Cert cert:\localMachine\my\141594A0AE38CBBECED7AF680F7945CD51D8F28A -FilePath child_b_from_a_dev_damienbod.crt
```

使用根证书、中间证书或子证书时，可以根据需要使用指纹或 PublicKey 来验证证书。

```csharp
using System.Collections.Generic;
using System.IO;
using System.Security.Cryptography.X509Certificates;

namespace AspNetCoreCertificateAuthApi
{
    public class MyCertificateValidationService 
    {
        public bool ValidateCertificate(X509Certificate2 clientCertificate)
        {
            return CheckIfThumbprintIsValid(clientCertificate);
        }

        private bool CheckIfThumbprintIsValid(X509Certificate2 clientCertificate)
        {
            var listOfValidThumbprints = new List<string>
            {
                "141594A0AE38CBBECED7AF680F7945CD51D8F28A",
                "0C89639E4E2998A93E423F919B36D4009A0F9991",
                "BA9BF91ED35538A01375EFC212A2F46104B33A44"
            };

            if (listOfValidThumbprints.Contains(clientCertificate.Thumbprint))
            {
                return true;
            }

            return false;
        }
    }
}
```

<a name="occ"></a>

::: moniker range=">= aspnetcore-5.0"

## <a name="certificate-validation-caching"></a>证书验证缓存

ASP.NET Core 5.0 及更高版本支持启用验证结果缓存的功能。 缓存极大地提高了证书身份验证的性能，因为验证是一种代价高昂的操作。

默认情况下，证书身份验证禁用缓存。 若要启用缓存，请调用 `AddCertificateCache` 中的 `Startup.ConfigureServices` ：

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(
        CertificateAuthenticationDefaults.AuthenticationScheme)
            .AddCertificate()
            .AddCertificateCache(options =>
            {
                options.CacheSize = 1024;
                options.CacheEntryExpiration = TimeSpan.FromMinutes(2);
            });
}
```

默认的缓存实现将结果存储在内存中。 可以通过实现 `ICertificateValidationCache` 并将其注册到依赖关系注入来提供自己的缓存。 例如 `services.AddSingleton<ICertificateValidationCache, YourCache>()`。

::: moniker-end

## <a name="optional-client-certificates"></a>可选客户端证书

本部分为必须使用证书保护应用程序子集的应用提供信息。 例如， Razor 应用中的页面或控制器可能需要客户端证书。 这是客户端证书带来的挑战：
  
* 是 TLS 功能，而不是 HTTP 功能。
* 按连接进行协商，在连接开始时必须在任何 HTTP 数据可用之前进行协商。 在连接开始时，仅知道服务器名称指示 (SNI) &dagger; 是已知的。 客户端和服务器证书在第一次请求连接之前进行协商，请求通常无法重新协商。

TLS 重新协商是实现可选客户端证书的一种方法。 不建议这样做，因为：
- 在 HTTP/1.1 中，在 POST 请求期间 renegotiating 可能会导致死锁，其中，请求正文填充 TCP 窗口，而重新协商数据包无法接收。
- HTTP/2 [显式禁止](https://tools.ietf.org/html/rfc7540#section-9.2.1) 重新协商。
- TLS 1.3 已 [删除](https://tools.ietf.org/html/rfc8740#section-1) 对重新协商的支持。

ASP.NET Core 5 preview 7 及更高版本为可选的客户端证书添加了更方便的支持。 有关详细信息，请参阅 [可选证书示例](https://github.com/dotnet/aspnetcore/tree/9ce4a970a21bace3fb262da9591ed52359309592/src/Security/Authentication/Certificate/samples/Certificate.Optional.Sample)。

以下方法支持可选的客户端证书：

* 设置域和子域的绑定：
  * 例如，在和上设置绑定 `contoso.com` `myClient.contoso.com` 。 `contoso.com`主机不需要客户端证书，而是 `myClient.contoso.com` 。
  * 有关详细信息，请参阅：
    * [Kestrel](/fundamentals/servers/kestrel)：
      * [ListenOptions.UseHttps](xref:fundamentals/servers/kestrel#listenoptionsusehttps)
      * <xref:Microsoft.AspNetCore.Server.Kestrel.Https.HttpsConnectionAdapterOptions.ClientCertificateMode>
      * 请注意，Kestrel 目前不支持在一个绑定上支持多个 TLS 配置，需要两个具有唯一 Ip 或端口的绑定。 请参见https://github.com/dotnet/runtime/issues/31097
    * IIS
      * [承载 IIS](xref:host-and-deploy/iis/index#create-the-iis-site)
      * [配置 IIS 上的安全性](/iis/manage/configuring-security/how-to-set-up-ssl-on-iis#configure-ssl-settings-2)
    * Http.Sys： [配置 Windows Server](xref:fundamentals/servers/httpsys#configure-windows-server)
* 对于需要客户端证书且没有客户端证书的 web 应用的请求：
  * 使用客户端证书保护的子域重定向到同一页面。
  * 例如，重定向到 `myClient.contoso.com/requestedPage` 。 由于与的请求 `myClient.contoso.com/requestedPage` 不同于的主机名 `contoso.com/requestedPage` ，因此客户端会建立一个不同的连接，并提供客户端证书。
  * 有关详细信息，请参阅 <xref:security/authorization/introduction>。

在 [此 GitHub 讨论](https://github.com/dotnet/AspNetCore.Docs/issues/18720) 问题中，对可选客户端证书留下疑问、评论和其他反馈。

&dagger; 服务器名称指示 (SNI) 是一个 TLS 扩展，可将虚拟域作为 SSL 协商的一部分包括在内。 这实际上意味着，可以使用虚拟域名或主机名来识别网络终结点。
