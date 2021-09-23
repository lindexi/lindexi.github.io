# dotnet C# 应用程序进程创建太多线程将会抛出 OutOfMemoryException 异常

本文记录一个 dotnet 的特性，在应用程序快速创建大量线程的时候，将会因为线程创建时没有足够的资源而创建失败，此时将会抛出 OutOfMemoryException 异常，但实际进程占用内存不多

<!--more-->
<!-- CreateTime:2021/9/17 19:37:42 -->

<!-- 发布 -->

如使用以下逗比代码进行线程的创建

```csharp
            while (true)
            {
                var thread = new Thread((_) =>
                {
                    Thread.Sleep(-1);
                });
                thread.Start();
            }
```

此时在运行时将会抛出 OutOfMemoryException 异常，在 x86 应用下，在我的设备上跑了大概 1000 个线程左右时将会炸掉，但是进程只占用了 60MB 左右的内存

[c# - Why Thread.Start can throw OutOfMemoryException - Stack Overflow](https://stackoverflow.com/questions/15789507/why-thread-start-can-throw-outofmemoryexception )



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
