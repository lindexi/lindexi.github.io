---
title: dotnet C# 警惕结构体加等事件
description: 前几天我在对文本库进行性能优化，将其中一个枚举迭代器从类改成结构体，然而改造之后却遇到了本文记录的在结构体内加等事件的坑
tags: dotnet,C#
category: 
---

<!-- 发布 -->
<!-- 博客 -->

如以下代码所示，还请大家猜猜以下代码输出会是多少。如以下代码，在 F1 结构体的构造函数里面，加等了 F2 类型里面的 F3 事件。随后又在外面，确保 F2 触发了 F3 事件

```csharp
F2 f2 = new F2();
F1 f1 = new F1(f2);
f2.RaiseF3();

Console.WriteLine(f1.N1);

struct F1
{
    public F1(F2 f2)
    {
        N1 = 0;
        f2.F3 += F2_F3;
    }
    private void F2_F3(object? sender, EventArgs e)
    {
        N1++;
    }
    public int N1 { get; set; }
}
class F2
{
    public event EventHandler? F3;
    public void RaiseF3()
    {
        F3?.Invoke(this, EventArgs.Empty);
    }
}
```

看完了代码，大家能够大概了解到，如果此时的 F1 监听 F3 的函数能运行，那自然 `N1++` 就被执行，此时 N1 属性就是 1 的值。但如果尝试运行代码，可见此时输出是 0 的值

如果断点打在 `F2_F3` 方法里面，可见此时的 N1 是 1 的值了，此时为什么外面的 f1 的 N1 还是 0 的值

我一开始没有反应过来，稍微卡一会。随后才发现，这是由于 F1 是结构体的原因。尽管是在 F1 的构造函数加等的 F3 事件，但加等事件本身会构建一个委托，委托尝试捕获 this 对象，此时的 this 对象就是 F1 结构体。再由于结构体赋值过程就是拷贝过程，于是加等 F3 事件的委托对象，实际上就是复制了一次 F1 对象。即使监听方法能够执行，甚至重复用 Visual Studio 拖动执行语句重复触发 F3 事件，也能观察到 N1 值在不断变化。但这些变化的 N1 值都是在委托里面的 F1 拷贝对象里的，对外面的 F1 对象毫无关系

换句话就是 F1 对象被拷贝一份到委托里面了，导致事件变更无法影响到外面的 F1 对象

那为什么设计上 C# 是让委托捕获结构体过程是进行拷贝呢？想想，局部变量如果是结构体，则不严谨但至少在当前代码环境下正确来说，结构体分配在栈上。加等事件时，通过挂委托对象到事件里面，这是一个长生命周期的过程。在栈上的结构体，会在当前方法结束后，被清栈回收。可以理解为方法执行完了，结构体也就没了。但如果此时委托捕获的是栈上的结构体对象引用，那当事件触发时，则可能委托捕获的栈上结构体内存已经被清理了，现在这段内存已经用作其他对象了，将会执行错误访问。为了防止此问题的发生，聪明的软软就设计了委托加等事件的时候，在结构体内时，执行的都是结构体赋值，即结构体拷贝。确保结构体内存不会在超过结构体对象被回收之后被访问

如果大家对栈上空间在方法执行完成之后进行清栈回收感到疑惑，还请自行补充一下基础知识，这部分过于基础，我也难用简单的话来说清楚。同样，对结构体赋值过程就是结构体拷贝过程这个知识点疑惑，也请大家自行补充

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/f3ed2c55c1f7c22bde56f640fc343f384a283db9/Workbench/WhubaydeeluliwardayKachejekeajel) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/f3ed2c55c1f7c22bde56f640fc343f384a283db9/Workbench/WhubaydeeluliwardayKachejekeajel) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin f3ed2c55c1f7c22bde56f640fc343f384a283db9
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin f3ed2c55c1f7c22bde56f640fc343f384a283db9
```

获取代码之后，进入 Workbench/WhubaydeeluliwardayKachejekeajel 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
