对于 HttpClient 的请求响应值 HttpResponseMessage 来说，既然继承了 IDisposable 接口，自然就是想让大家可以通过 using 或者手动调用 Dispose 进行释放的。本文将来聊聊对 HttpResponseMessage 调用 Dispose 进行释放的意义有多大，有没有必要性的问题

<!--more-->


<!-- CreateTime:2023/7/18 19:54:27 -->

<!-- csdn -->
<!-- 博客 -->
<!-- 发布 -->

先说结论，建议通过 using 或者手动调用 Dispose 释放 HttpResponseMessage 对象。 但不直接或间接调用 Dispose 释放也没有出现什么大问题，也不会出现重大的内存泄露问题

在 HttpResponseMessage 的终结器（析构）里面也会自动调用 Dispose 释放资源，也就是在 HttpResponseMessage 对象被 GC 时候，也会能够调用到 Dispose 的逻辑

调用 HttpResponseMessage 的 Dispose 的意义在于释放 HttpResponseMessage 的 Content 资源。更细分的需要聊到请求对应的 HTTP 版本。对于 HTTP 1.1 版本来说，调用 Dispose 方法约等于啥都没做，约等于标记状态而已，调用或不调用约等于没有差别

对于 HTTP 2 来说，直接或间接调用 HttpResponseMessage 的 Dispose 方法可以提升整体的性能。原因是在 HttpResponseMessage 的 Dispose 方法里面，将会释放 HttpResponseMessage 的 Content 所使用的数组池的资源，让 Content 里的缓存数组返回给到数组池，方便其他业务逻辑复用。其次在 Content 还没被读取完成的时候，调用 Dispose 方法能够让 HTTP 连接归还连接池，方便后续其他逻辑复用连接。而在 Content 被读取完成时，自然连接就还给了连接池，此时调用 Dispose 方法将没有连接池的优化。但无论如何，直接或间接调用 HttpResponseMessage 的 Dispose 方法，还是有一定的提升的，至少还能归还 Content 里的缓存数组到数组池

总的来说，推荐使用 using 关键字释放 HttpResponseMessage 对象。至少这样写起来不亏

对 HttpResponseMessage 直接或间接调用 Dispose 方法，将会自动调用到 HttpResponseMessage 的 Content 的释放，也就是从 Content 里所获取的 Stream 可以不用再释放。当然，同时对 Content 里所获取的 Stream 和 HttpResponseMessage 都调用释放也没有什么问题，如以下代码是没有问题的，框架内部处理了

```csharp
            using HttpResponseMessage response = Xxx();
            using var stream = response.Content.ReadAsStream();
```

只对 HttpResponseMessage 或者是 Content 里所获取的 Stream 进行释放，其中之一调用 Dispose 释放效果约等于相同。这是因为在 HttpResponseMessage 的 Dispose 里面最重要的就是调用 Content 的释放，因此只对 Content 调用释放也是完全合理的

感谢 [lsj](https://blog.sdlsj.net) 阅读 dotnet 源代码然后告诉我，让我的代码有了基础的支持。同时也需要吐槽一下官方文档对此行为没有写清楚，让我以为调用 Dispose 会断开连接，导致连接无法还连接池。实际上是在 HTTP 2 调用 Dispose 会更好的还给连接池

至于对 Content 里所获取的 Stream 调用 `await using` 进行异步释放，这是没有必要的，因为这里的释放逻辑没有什么需要异步的。虽然加了也不亏
