# dotnet C# 结构体出方法弹栈之后的行为

本文记录我在 .NET 9 里测试的行为，在方法里面创建的在栈上的结构体，在方法执行结束之后，栈上的结构体将会被弹栈进入不受管理区域，此时的结构体内存内容不会立刻被清空或被改写

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

这是我在对 [dotnet X11 栈空间被回收导致调用 XPutShmImage 闪退](https://blog.lindexi.com/post/dotnet-X11-%E6%A0%88%E7%A9%BA%E9%97%B4%E8%A2%AB%E5%9B%9E%E6%94%B6%E5%AF%BC%E8%87%B4%E8%B0%83%E7%94%A8-XPutShmImage-%E9%97%AA%E9%80%80.html ) 博客的内容进行更多的测试，确保和 X11 没有关系，只是存 dotnet C# 的行为
<!-- [dotnet X11 栈空间被回收导致调用 XPutShmImage 闪退 - lindexi - 博客园](https://www.cnblogs.com/lindexi/p/18375092 ) -->

如以下代码，在 Foo 方法里面创建 F 结构体，此时 F 结构体将会在栈上分配。当 Foo 方法执行完成之后，将会弹栈。然而 Foo 的返回值就是指向 F 结构体的栈内存地址的指针，出了方法之后，尝试获取其字段输出

```csharp
    F* foo = Foo();

    var a1 = foo->A1;
    var a2 = foo->A2;
    var a3 = foo->A3;

    GC.KeepAlive(a1);
    GC.KeepAlive(a2);
    GC.KeepAlive(a3);

    Console.WriteLine($"{a1} {a2} {a3}");

    F* Foo()
    {
        F f = new F()
        {
            A1 = 100,
            A2 = 200,
            A3 = 300
        };

        return &f;
    }

struct F
{
    public int A1;
    public int A2;
    public int A3;
}
```

经过实验测试，发现无论在 DEBUG 下，还是 RELEASE 都可以输出符合预期的 100 200 300 的值。通过此实验可以证明 dotnet C# 里面没有使用如 [C++ - 面向基于堆栈的缓冲区保护的 Visual C++ 支持 - Microsoft Learn](https://learn.microsoft.com/zh-cn/archive/msdn-magazine/2017/december/c-visual-c-support-for-stack-based-buffer-protection ) 文档所述的各种机制，如使用 0xCC 填充不被使用的地址空间

如果我在此基础之上，继续调用其他方法，让其他方法压入栈，这将会污染或破坏 f 指针指向的结构体的内容。如下面所示

```csharp
    F* foo = Foo();
    F2(100,100);

    var a1 = foo->A1;

    var a2 = foo->A2;
    var a3 = foo->A3;

    GC.KeepAlive(a1);
    GC.KeepAlive(a2);
    GC.KeepAlive(a3);

    Console.WriteLine($"{a1} {a2} {a3}");

    int F2(int n, int count)
    {
        if (count == 0)
        {
            return n;
        }

        if (n == count)
        {
            return n;
        }

        count--;
        n = Random.Shared.Next();
        return F2(n, count);
    }
```

以上代码运行，所输出的 a1 和 a2 和 a3 的值一般不会是 100 200 300 的值，而是会被后续的 F2 方法污染了栈空间，导致值被改写

通过以上测试可以了解到不安全代码确实不安全，在日常编写过程中，如使用栈地址空间，则必须小心栈地址是否持续可用。这部分没有其他兜底逻辑，需要开发者自行处理安全性问题

感觉这也很符合 C# dotnet 的设计，不安全代码就是不安全，开发者使用不安全代码就需要自己处理好代码的安全和稳定

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0a4038a09597a2478aa6073a1382fc0ce60a766e/Workbench/HilahawcaWarcerbakuwhi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0a4038a09597a2478aa6073a1382fc0ce60a766e/Workbench/HilahawcaWarcerbakuwhi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0a4038a09597a2478aa6073a1382fc0ce60a766e
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0a4038a09597a2478aa6073a1382fc0ce60a766e
```

获取代码之后，进入 Workbench/HilahawcaWarcerbakuwhi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )