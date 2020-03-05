# dotnet 使用 GC.GetAllocatedBytesForCurrentThread 获取当前线程分配过的内存大小

在 dotnet framework 4.8 的时候支持调用 GC.GetAllocatedBytesForCurrentThread 获取当前线程分配过的内存大小

<!--more-->
<!-- CreateTime:2019/5/21 11:33:18 -->

<!-- 标签：C#,dotnet -->

创建一个简单的控制台程序，在调用 GC.GetAllocatedBytesForCurrentThread 方法返回调用方法所在的线程的内存大小，代码请看[GetAllocatedBytesForCurrentThread](https://github.com/lindexi/lindexi_gd/tree/4470807a21a7ae2396d5bf9719ddcecc22f72e99/GetAllocatedBytesForCurrentThread )

调用 GC.GetAllocatedBytesForCurrentThread 返回的是当前线程从启动到调用这个方法分配的内存大小，这个内存大小不是指在 GC 回收之后占用的大小

如我使用[下面代码](https://github.com/lindexi/lindexi_gd/blob/4470807a21a7ae2396d5bf9719ddcecc22f72e99/GetAllocatedBytesForCurrentThread/GetAllocatedBytesForCurrentThread/Program.cs#L31-L34) 每次调用 Foo 的时候，从 GetAllocatedBytesForCurrentThread 返回的值会不断添加，而不会随着 foo 被回收减少

```csharp
        private void Foo()
        {
            var foo = new byte[100];
        }
```

这个方法在调用某些事件申请的内存的时候非常有用，可以看到在调用某些方法申请的内存有多少

可以在进入某个事件或方法的时候先调用获取当前线程的内存分配过的大小，在调用方法完成之后再调用一次，对比两个值就知道在这个方法里面申请的内存需要多少

这个方法只有在 .NET Core 2.0 以上和 .NET Framework 4.8 以上才可以用到

[GC.GetAllocatedBytesForCurrentThread Method (System)](https://docs.microsoft.com/en-us/dotnet/api/system.gc.getallocatedbytesforcurrentthread?wt.mc_id=MVP)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
