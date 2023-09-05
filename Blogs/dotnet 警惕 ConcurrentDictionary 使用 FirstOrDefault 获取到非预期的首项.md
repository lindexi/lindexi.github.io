在 dotnet 里面的 ConcurrentDictionary 是一个支持并发读写的线程安全字典，在这个字典里面有一些行为会出现随机性，即多次执行相同的代码返回的结果可能不相同。本文记录在 ConcurrentDictionary 使用 FirstOrDefault 获取到非预期的首项的问题

<!--more-->


<!-- CreateTime:2023/6/14 8:45:07 -->

<!-- 博客 -->
<!-- 发布 -->

在 dotnet 里面，无论是对 List 列表，还是 Dictionary 字典等获取首项，使用 FirstOrDefault 总是可以获取到第一个加入到集合或字典里面的元素。然而这个行为在 ConcurrentDictionary 里面是不成立的。在 ConcurrentDictionary 里面如果使用 FirstOrDefault 方法，则随机获取到字典里面的一项，但对相同的一个 ConcurrentDictionary 对象多次调用 FirstOrDefault 方法，在不更改 ConcurrentDictionary 内容的情况下，可以稳定获取到相同的首项元素对象

简单来说就是在 ConcurrentDictionary 里面，调用 FirstOrDefault 方法，不能保证获取到的对象就是第一个加入到 ConcurrentDictionary 字典里面的对象

如以下代码例子

```csharp
using System.Collections.Concurrent;

for (int i = 0; i < int.MaxValue; i++)
{
    var dictionary = new ConcurrentDictionary<Foo, int>();
    dictionary.TryAdd(new Foo(), i);
    dictionary.TryAdd(new Foo(), i + 1);

    var first = dictionary.FirstOrDefault();
    if (first.Value != i)
    {
        // 证明首个不是第一个加入的
        Console.WriteLine($"首个不是第一个加入的");
        return;
    }
}

class Foo
{
    public Foo()
    {
        Number = _count;
        _count++;
    }

    private static int _count;

    public int Number { get; }
}
```

以上代码在一个大的循环里面，每次循环都创建一个字典，在给字典加入两个元素，最后加入的元素设置为和循环次数不相同的值，通过此可以用来在后续调用 FirstOrDefault 时判断获取到的元素是否首个加入字典的元素

运行代码可以看到，使用 FirstOrDefault 获取到的元素，不是第一个加入字典的元素。同时，多次运行代码，可以看到进入 `if (first.Value != i)` 不等于条件时的循环次数也会不相同，这就可以证明使用 FirstOrDefault 的执行结果比较随机

具体原理是在 ConcurrentDictionary 里面需要维护一个 Table 字典，字典里面存放的顺序和传入的 Key 对象的 Hash 有关，调用 FirstOrDefault 方法时获取到的是里面的 Table 字典的按照内存空间顺序的首项

由此原理即可知道，使用 FirstOrDefault 获取 ConcurrentDictionary 的首现是无法确保获取到的是首个加入字典的元素对象。同时如果在 ConcurrentDictionary 字典发生变更，比如不断加入值时，将导致调用 FirstOrDefault 无法稳定返回相同的对象

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/623c123b1d5e5669c5321c846d72c09e042135a6/CerdearhachiRairwainalnearyal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/623c123b1d5e5669c5321c846d72c09e042135a6/CerdearhachiRairwainalnearyal) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 623c123b1d5e5669c5321c846d72c09e042135a6
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 623c123b1d5e5669c5321c846d72c09e042135a6
```

获取代码之后，进入 CerdearhachiRairwainalnearyal 文件夹
