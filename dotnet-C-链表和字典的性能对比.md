
# dotnet C# 链表和字典的性能对比

本文来告诉大家我实际使用基准测试的在 .NET Core 3.1 的链表 LinkedList 和 Dictionary 字典的在元素增删的性能对比

<!--more-->


<!-- CreateTime:2020/12/23 12:39:56 -->

<!-- 发布 -->

从算法分析上，其实字典和链表在时间上的性能是差不多的，甚至可以认为字典的速度会比链表更高。但是从空间上，如果一边增加元素一边删除元素那还是链表省空间。同时在字典空间满了之后，修改字典容量会比链表使用更多的时间

以下是我用不够严谨的基准性能测试的数据

下面是对比一边加入元素一边删除元素的性能

|             Method |       Mean |    Error |   StdDev | Ratio | RatioSD |
|------------------- |-----------:|---------:|---------:|------:|--------:|
| 'LinkedList' | 2,424.9 us | 28.53 us | 23.83 us |     ? |       ? |
| 'Dictionary' |   686.0 us |  5.61 us |  4.97 us |  1.00 |    0.00 |


下面是将元素完全加入之后再删除全部删除的性能对比

|                Method |     Mean |     Error |    StdDev | Ratio | RatioSD |
|---------------------- |---------:|----------:|----------:|------:|--------:|
| 'LinkedList' | 2.582 ms | 0.0303 ms | 0.0253 ms |     ? |       ? |
| 'Dictionary' | 1.366 ms | 0.0192 ms | 0.0170 ms |  1.00 |    0.00 |

仅加入元素的性能对比

|        Method |     Mean |     Error |    StdDev | Ratio | RatioSD |
|-------------- |---------:|----------:|----------:|------:|--------:|
| 'LinkedList' | 2.371 ms | 0.0466 ms | 0.0697 ms |  0.83 |    0.02 |
| 'Dictionary' | 2.855 ms | 0.0380 ms | 0.0317 ms |  1.00 |    0.00 |

通过测试可以看到，只有在加入元素的性能，链表的性能才会比字典快一点点。而如果包含了删除，那么性能还是字典强

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/aa1ba2b209cd484e9b17d2fe43102f5e7e1fc3e5/HurnabahearwhawJehearhefena)欢迎小伙伴访问

本文的测试本来是为了给 WPF 框架做性能优化使用的，请看 [dotnet 读 WPF 源代码笔记 AppDomainShutdownMonitor 的设计](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-AppDomainShutdownMonitor-%E7%9A%84%E8%AE%BE%E8%AE%A1.html ) 但实际发现使用字典性能更好

本文的测试仅仅只是适用于 WPF 的 AppDomainShutdownMonitor 类的情况，不代表其他业务下依然是字典更优





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。