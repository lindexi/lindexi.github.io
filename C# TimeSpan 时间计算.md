# C# TimeSpan 时间计算

本文告诉大家简单的方法进行时间计算。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<!-- 标签：C#，C#入门 -->

实际上使用 TimeSpan 可以做到让代码比较好懂，而代码很简单。

例如我使用下面的代码表示 5 秒

```csharp
const int needCount = 5 * 1000;
```

因为后面使用的是延迟，延迟的代码很简单

```csharp
Task.Delay(needCount)
```

这时传入的是一个毫秒，但是很多小伙伴问，为什么是 `5*1000` 表示 5秒，他不知道我使用的是毫秒。

所以建议使用 TimeSpan 来写时间，下面的需求是在判断在开机 20 秒内的延迟，如果在开机 20 秒内启动应用，那么就需要延迟时间

```csharp
            var needTime = TimeSpan.FromSeconds(20); //开机20秒左右 USB 已经加载完成
```

计算时间的减法或加法可以使用重载`+`和`-`，请看下面代码，就是把两个 TimeSpan 相减，返回的值也是一个 TimeSpan ，下面的代码是编译不通过的。

```csharp
            var chikesereHearpawwirboo = needTime - maxDelay;
            Console.WriteLine(chikesereHearpawwirboo);
```

如果需要从毫秒转 TimeSpan ，请看下面代码

```csharp
            // 毫秒转 TimeSpan
            var milliseconds = 5 * 1000;
            var time = TimeSpan.FromMilliseconds(milliseconds);

            // TimeSpan 转 毫秒
            milliseconds =(int) time.TotalMilliseconds;
```

因为从秒转毫秒的值是 double 需要进行转换，如果使用 int 转换有时会越界，建议使用下面代码

```csharp

            // 毫秒转 TimeSpan
            long milliseconds = 5 * 1000;
            var time = TimeSpan.FromMilliseconds(milliseconds);

            // TimeSpan 转 毫秒
            milliseconds = (long) Math.Ceiling(time.TotalMilliseconds);
```

这个计算适合在有天数和小时等的计算，如计算 1天 减去 3h10m 有多少毫秒，如果不使用 TimeSpan 自己重写，还是需要写很多代码

```csharp
            var time = TimeSpan.FromDays(1);
            var cut = new TimeSpan(0, 3, 10, 0);
            var milliseconds = (long)(time - cut).TotalMilliseconds;
```

尝试不使用 TimeSpan 想想需要怎么写

![](http://image.acmx.xyz/lindexi%2F2018612935409133.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
