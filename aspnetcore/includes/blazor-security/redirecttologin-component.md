`RedirectToLogin` 组件 (`Shared/RedirectToLogin.razor`)：

* 管理将未经授权的用户重定向到登录页。
* 保留用户尝试访问的当前 URL，以便在身份验证成功时可以将其返回到该页。

```razor
@inject NavigationManager Navigation
@using Microsoft.AspNetCore.Components.WebAssembly.Authentication
@code {
    protected override void OnInitialized()
    {
        Navigation.NavigateTo(
            $"authentication/login?returnUrl={Uri.EscapeDataString(Navigation.Uri)}");
    }
}
```
