# dotnet 对 DateTime 排序

在写 DateTime 排序时，按照时间的先后，离现在过去越远的越小。按照从小到大排序，将会先排最过去的时间，最后的值的时间是最大的。将时间按照从1970开始计算秒数，可以算出数值，数值代表值大小

<!--more-->
<!-- CreateTime:2019/9/29 14:55:49 -->

<!-- csdn -->

通过 List 的 OrderBy 是从时间从小到大升序排列，也就是最之前的时间排在最前，如下面的测试代码

```csharp
            var dateTimeList = new List<DateTime>()
            {
                DateTime.Now,
                DateTime.Now.AddHours(1),
                DateTime.Now.AddHours(2),
            };
```

此时用下面代码进行排序

```csharp
            foreach (var dateTime in dateTimeList.OrderBy(temp => temp))
            {
                Console.WriteLine(dateTime);
            }
```

可以看到输出，最 15 点开始

```csharp
2019/9/29 15:00:00
2019/9/29 16:00:00
2019/9/29 17:00:00
```

反过来，按照从未来到过去的顺序，从大到小排列，可以使用 OrderByDescending 方法

```csharp
            foreach (var dateTime in dateTimeList.OrderByDescending(temp => temp))
            {
                Console.WriteLine(dateTime);
            }
```

如果列表里面某个类，那么按照时间排序可以传入对应属性

其实在开始写的过程，有点无法理解时间排序，问了小伙伴说按照从1970到现在秒数排列就可以，此时就知道从小到大排序是按照从过去到现在的时间，这篇文章有些水

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/blob/65637abed6e0d5268eb34e3df7457ffdad5c9172/LabairliwoKelluyewhidee) 欢迎小伙伴下载

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
