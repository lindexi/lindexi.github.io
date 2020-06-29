# C# dotnet Thread.GetCurrentProcessorId 和 CurrentProcess.Id 的区别

使用 Thread.GetCurrentProcessorId 可以获取当前线程处理器的 Id 是哪个，而通过 Process.GetCurrentProcess().Id 可以获取当前进程的 Id 号，这两个的差别从上面描述就能看出

<!--more-->
<!-- 发布 -->

更进一步的 Thread.GetCurrentProcessorId() 方法将会缓存处理器的 Id 因此前后两次线程如果处理器切换，此时拿到的值是不对的。在官方文档里面说了使用代码不得依赖于其正确性，因此这个值基本上只在 DUMP 使用

而 Process.GetCurrentProcess().Id 是一个不需要 win32 的支持的方法，获取速度特别快，详细请看 [.NET 中 GetProcess 相关方法的性能 - walterlv](https://blog.walterlv.com/post/performance-of-get-process.html )

获取速度如下

```csharp
// 100 次执行耗时

    Process.GetProcesses()
        00:00:00.7254688
    Process.GetProcessById(id)
        00:00:01.3660640（实际数值取决于当前进程数）
    Process.GetProcessesByName("Walterlv.Demo")
        00:00:00.5604279
    Process.GetCurrentProcess()
        00:00:00.0000546

```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
