当使用已通过 AAD 注册的服务器 API，并且应用的 AAD 注册在依赖于[未验证的发布者域](/azure/active-directory/develop/howto-configure-publisher-domain)的租户中时，服务器 API 应用的应用 ID URI 不是 `api://{SERVER API APP CLIENT ID OR CUSTOM VALUE}`，而是采用 `https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}` 格式。 如果是这种情况，则 `Client` 应用的 `Program.Main` (`Program.cs`) 中的默认访问令牌作用域如下所示：

```csharp
options.ProviderOptions.DefaultAccessTokenScopes
    .Add("https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}/{DEFAULT SCOPE}");
```

若要为匹配的受众配置服务器 API 应用，请将 `Server` API 应用设置文件 (`appsettings.json`) 中的 `Audience` 设置为匹配 Azure 门户提供的应用受众：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/{TENANT ID}",
    "ClientId": "{SERVER API APP CLIENT ID}",
    "ValidateAuthority": true,
    "Audience": "https://{TENANT}.onmicrosoft.com/{SERVER API APP CLIENT ID OR CUSTOM VALUE}"
  }
}
```

在上面的配置中，`Audience` 值的末尾不包括默认作用域 `/{DEFAULT SCOPE}`。

示例：

`Client` 应用的 `Program.Main` (`Program.cs`)：

```csharp
options.ProviderOptions.DefaultAccessTokenScopes
    .Add("https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd/API.Access");
```

通过匹配的受众 (`Audience`) 配置 `Server` API 应用设置文件 (`appsettings.json`)：

```json
{
  "AzureAd": {
    "Authority": "https://login.microsoftonline.com/e86c78e2-...-918e0565a45e",
    "ClientId": "41451fa7-82d9-4673-8fa5-69eff5a761fd",
    "ValidateAuthority": true,
    "Audience": "https://contoso.onmicrosoft.com/41451fa7-82d9-4673-8fa5-69eff5a761fd"
  }
}
```

在前面的示例中，`Audience` 值的末尾不包括默认作用域 `/API.Access`。
