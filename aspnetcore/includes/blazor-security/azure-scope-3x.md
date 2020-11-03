Azure 门户根据 AAD 租户是否有[已验证或未验证的发布者域](/azure/active-directory/develop/howto-configure-publisher-domain)提供以下应用 ID URI 格式之一：

* `api://{SERVER API APP CLIENT ID OR CUSTOM VALUE}`
* `https://{TENANT}.onmicrosoft.com/{API CLIENT ID OR CUSTOM VALUE}`

如果在客户端应用中使用前述两个应用 ID URI 中的后一个来形成作用域 URI，并且授权不成功，请尝试提供没有方案和主机的作用域 URI：

```csharp
options.ProviderOptions.DefaultAccessTokenScopes.Add(
    "{SERVER API APP CLIENT ID OR CUSTOM VALUE}/{SCOPE NAME}");
```

示例：

```csharp
options.ProviderOptions.DefaultAccessTokenScopes.Add(
    "41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
```
