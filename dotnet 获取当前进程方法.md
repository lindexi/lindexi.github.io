# dotnet 获取当前进程方法

本文告诉大家如何在 dotnet 程序获取当前进程

<!--more-->
<!-- CreateTime:2019/9/2 11:03:03 -->

<!-- csdn -->

使用下面代码可以获取当前进程

```csharp
var process = Process.GetCurrentProcess();
```

那么这个方法的性能如何?

这个获取的方法内部有缓存，第一次获取的速度会比较慢，稍后会比较快，我使用下面代码测量

```csharp
            var stopwatch = new Stopwatch();

            for (int i = 0; i < 100; i++)
            {
                stopwatch.Restart();

                var process = Process.GetCurrentProcess();
                stopwatch.Stop();

                Console.WriteLine(stopwatch.ElapsedTicks);
            }
```

第一次运行的时候比较长，稍后运行速度会快很多

```csharp
908
161
25
15
14
17
15
15
18
20
// 忽略
```

如果只是想要拿到进程号可以使用下面代码

```csharp
        [DllImport("kernel32.dll", CharSet = CharSet.Auto)]
        public static extern int GetCurrentProcessId();
```

使用上面代码获取的性能能更快，但第一次获取的速度也不快

如果是在循环或其他代码里面，建议将进程号缓存起来

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
