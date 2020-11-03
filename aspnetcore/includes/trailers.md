HTTP 尾部类似于 HTTP 标头，只不过它是在发送响应正文后发送的。 IIS 和 HTTP.SYS 仅支持 HTTP/2 响应尾部。

```csharp
if (httpContext.Response.SupportsTrailers())
{
    httpContext.Response.DeclareTrailer("trailername"); 

    // Write body
    httpContext.Response.WriteAsync("Hello world");

    httpContext.Response.AppendTrailer("trailername", "TrailerValue");
}
```

在前面的示例代码中：

* `SupportsTrailers` 确保响应支持尾部。
* `DeclareTrailer` 将给定的尾部名称添加到 `Trailer` 响应头。 虽然并不是必须要声明响应尾部，但是建议这样做。 如果要调用 `DeclareTrailer`，则必须在发送响应标头之前进行此操作。
* `AppendTrailer` 追加尾部。
