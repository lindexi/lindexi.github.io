# FileStream 的 FlushAsync 方法在 .NET Framework 与 .NET Core 行为的不同

本文记录 FileStream 的 FlushAsync 方法在 .NET Framework 与 .NET Core 行为的不同

<!--more-->
<!-- CreateTime:2022/2/7 8:56:20 -->

<!-- 博客 -->
<!-- 发布 -->

在使用 HID 设备进行 IO 通讯时，可以采用 FileStream 包装，从而方便进行异步读写，然而在写入完成时，期望调用 FlushAsync 方法将缓存内容刷入设备。然而调用 FlushAsync 时，在 .NET Framework 下的默认行为是将 flushToDisk 参数设置为 true 的值，这将会导致抛出操作对象不支持异常。在 .NET Core 下，如 [Stephen Toub](https://github.com/stephentoub ) 大佬所说，当前没有一个系统提供了异步的 Flush 的支持，因此在 .NET Core 调用 FlushAsync 时，相当于设置为 flushToDisk 参数设置为 false 的刷入缓存到设备的行为

以上就是 FileStream 的 FlushAsync 方法在 .NET Framework 与 .NET Core 行为的不同。不同在于 .NET Framework 下默认将 flushToDisk 参数设置为 true 的值。而 .NET Core 下默认将 flushToDisk 参数设置为 false 的值

值得一说的是以上说的缓存其实有两层，一层是系统层提供的，调用 Flush 或 FlushAsync 配合 flushToDisk 参数设置为 true 的值，将让系统将内容刷入到设备。另一层是 dotnet 层的，调用 Flush 或 FlushAsync 将会让 dotnet 层的缓存刷入到系统里，由系统写入到 IO 设备

不过无论是 .NET Framework 还是 .NET Core 都没有给 FlushAsync 提供 flushToDisk 可选参数，也就是调用 FlushAsync 方法时，无法明确设置 flushToDisk 的值，需要依赖 CLR 的行为。这就是 FileStream 的 FlushAsync 方法在 .NET Framework 与 .NET Core 行为不同的原因

那是否会在后续版本，给 FlushAsync 加上 `bool flushToDisk` 重载？我在 GitHub 上提了此建议，参阅： [Adding overloading FileStream.FlushAsync(bool flushToDisk) · Issue #64485 · dotnet/runtime](https://github.com/dotnet/runtime/issues/64485 )

官方 [Stephen Toub](https://github.com/stephentoub ) 大佬的决策是基于当前没有任何系统支持异步的 Flush 的支持，因此很长时间内都不会考虑给 FlushAsync 提供此重载。我也认同此决策

> To my knowledge, there is no OS-level "flush to disk" functionality that's asynchronous, on either Linux, Windows, or macOS. The best we could do internally is the same you can do via public APIs: queue a work item to call Flush(true). We won't add such an API unless we can do meaningfully better than that.


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
