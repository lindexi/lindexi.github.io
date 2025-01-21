
# dotnet 6 InterpolatedStringHandler 构造函数的 out 参数有什么意义

根据微软的官方文档可以了解到，编写一个自定义的 InterpolatedStringHandler 时，可以在构造方法的参数里面带上一个 out 的布尔参数。本文将来和大家介绍这个布尔参数的作用

<!--more-->


<!-- CreateTime:2025/01/21 07:05:14 -->

<!-- 发布 -->
<!-- 博客 -->

根据 [探索 C# 字符串内插处理程序 - C# - Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/advanced-topics/performance/interpolated-string-handler ) 文档可以知道，在文档的最后一步介绍了在构造方法里面带上一个 `out bool isEnable` 参数。但是文档没有说明带上这个参数会给应用程序带来什么优化

本文就来接着官方文档继续探索一下

如以下代码示例，编写一个 InterpolatedStringHandler 类型，且将其作为另一个类的内部结构体

```csharp
class Fx
{
    public void Foo([InterpolatedStringHandlerArgument("")] ref FooInterpolatedStringHandler fooInterpolatedStringHandler)
    {
    }

    public bool Enable { set; get; }

    [InterpolatedStringHandler]
    public ref struct FooInterpolatedStringHandler
    {
        public FooInterpolatedStringHandler(int literalLength, int formattedCount, Fx fx, out bool isEnable)
        {
            isEnable = fx.Enable;
        }

        public void AppendLiteral(string s)
        {
        }

        public void AppendFormatted<T>(T value)
        {
        }
    }
}
```

期望的就是在 Fx 的 Enable 为 true 时才干活，这是通常一些运行时的调试代码减少资源损耗的方法

如以上代码可以看到，在调用 Foo 方法的时候，将会替换内插字符串为 FooInterpolatedStringHandler 的作用

如以上代码可以看到 FooInterpolatedStringHandler 的构造方法里面最后一个参数是一个 `out bool isEnable` 布尔参数

这里的语法是限定的，绑定 C# 10 或更高版本语义限定。要求 `out bool isEnable` 参数作为构造函数最后一个参数，且只能使用 bool 类型，禁止其他类型。使用其他类型将会在调用方报错提示找不到最合适的重载或未提供所需参数等错误。但对参数名没有限制，可以随意命名，如 `out bool lindexiIsDoubi` 也是合法的

那么这个 out 参数的作用是什么？咱编写一点测试代码来测试一下

```csharp
var fx = new Fx();
fx.Foo($"asd{GetTestInfo()}");

Console.WriteLine("Hello, World!");

int GetTestInfo()
{
    Console.WriteLine($"GetTestInfo");
    return 2;
}
```

如果没有黑科技的话，预期就是 GetTestInfo 方法会被调用，且输出 GetTestInfo 内容到控制台。但如果大家尝试到本文末尾找到本文所有代码的下载方法，下载到本文的测试代码，将会发现 GetTestInfo 方法没有被调用，控制台也没有输出任何内容

这就意味着如果构造函数最后一个布尔参数返回 false 时，将不会对内插字符串的参数进行求值。这是一个非常好的优化

这就意味着可以放心在编写一些调试输出字符串，这些调试输出字符串也可以放心带上一些需要求值的参数值，这个过程是无伤的，不会有资源损耗

那么这个黑科技是如何玩出来的？以上的 `fx.Foo($"asd{GetTestInfo()}");` 会实际被解析为以下等价 C# 代码

```csharp
    Fx fx = new Fx();
    Fx.FooInterpolatedStringHandler fooInterpolatedStringHandler = new Fx.FooInterpolatedStringHandler(3, 1, fx, out bool isEnable);

    if (isEnable)
    {
      fooInterpolatedStringHandler.AppendLiteral("asd");
      fooInterpolatedStringHandler.AppendFormatted<int>(GetTestInfo());
    }

    fx.Foo(ref fooInterpolatedStringHandler);
```

如以上等价代码可见，在构造函数返回的布尔参数，将会被用来作为判断逻辑。如以上代码所示，返回的 `isEnable` 会被作为判断条件，决定后续的参数是否传入

对于很多运行时调试逻辑来说，这个科技能有非常好的优化，可以极大减少调试逻辑的资源占用

即在运行时调试逻辑里面，如果判定当前是非调试模式，则可将 `out bool` 参数返回为 false 从而不再对后续参数进行求值和方法调用。再加上本身 InterpolatedStringHandler 可以如本文的 FooInterpolatedStringHandler 一样设计为 ref struct 结构体，在此调用过程中，完全没有堆对象分配，完全在栈上完，几乎没有成本开销。详细请看 [Improved interpolated strings - C# feature specifications Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/proposals/csharp-10.0/improved-interpolated-strings )

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/baa4bf01fabe21d92cbc0c1a5430e4fbfea90523/Workbench/WhichekehealemkaWelcelahee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/baa4bf01fabe21d92cbc0c1a5430e4fbfea90523/Workbench/WhichekehealemkaWelcelahee) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin baa4bf01fabe21d92cbc0c1a5430e4fbfea90523
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin baa4bf01fabe21d92cbc0c1a5430e4fbfea90523
```

获取代码之后，进入 Workbench/WhichekehealemkaWelcelahee 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。