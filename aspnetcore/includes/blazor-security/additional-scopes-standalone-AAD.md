添加 <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions> 以通过 <xref:Microsoft.Authentication.WebAssembly.Msal.Models.MsalProviderOptions.DefaultAccessTokenScopes> 获取 `User.Read` 权限：

```csharp
builder.Services.AddMsalAuthentication(options =>
{
    ...
    options.ProviderOptions.DefaultAccessTokenScopes
        .Add("https://graph.microsoft.com/User.Read");
});
```
