# WPF 获取进程启动到当前现在的时间

从应用程序线程创建现在的时间可以通过 Process 类的 StartTime 属性获取，也就是其实这个方法不局限 WPF 可用，任何 dotnet 应用都能此方法

<!--more-->
<!-- 发布 -->

通过 `Process.GetCurrentProcess().StartTime` 可以拿到进程启动时间，而通过 DateTime.Now 可以获取当前的时间

使用下面代码可以知道进程启动到现在的秒数

```csharp
(DateTime.Now - Process.GetCurrentProcess().StartTime).TotalSeconds
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
