通过“Reset”，服务器可以使用指定的错误代码重置 HTTP/2 请求。 重置请求被视为中止。

```csharp
var resetFeature = httpContext.Features.Get<IHttpResetFeature>();
resetFeature.Reset(errorCode: 2);
```

前述代码示例中的 `Reset` 指定 `INTERNAL_ERROR` 错误代码。 有关 HTTP/2 错误代码的详细信息，请访问[“HTTP/2 规范错误代码”部分](https://tools.ietf.org/html/rfc7540#page-50)。