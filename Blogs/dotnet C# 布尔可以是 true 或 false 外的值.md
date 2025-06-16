---
title: dotnet C# 布尔可以是 true 或 false 外的值
description: 在我的编程习惯意识里，布尔 bool 只能是 true 或 false 的值。如果要算上可空布尔，那就最多加上一个 null 空值。然而从 dotnet 运行时的角度上说，布尔完全完全可以是 true 或 false 之外的值
tags: dotnet,C#
category: 
---

<!-- 发布 -->
<!-- 博客 -->

试试看以下的代码片段，猜猜将会输出什么内容

```csharp
using System.Runtime.CompilerServices;

byte t = 2;
var foo = Unsafe.As<byte, bool>(ref t);
Console.WriteLine(foo);

if (foo)
{
    Console.WriteLine($"if (foo.F1)");
}

if (foo == true)
{
    Console.WriteLine($"if (foo.F1 == true)");
}
else
{
    Console.WriteLine($"if (foo.F1 != true)");
}

var t1 = true;
if (foo == t1)
{
    Console.WriteLine($"if (foo.F1 == t1)");
}
else
{
    Console.WriteLine($"if (foo.F1 != t1)");
}
```

有些伙伴也许和我一样，在看到前面几句话时，就开始感觉到不妙。正如 `var foo = Unsafe.As<byte, bool>(ref t);` 代码所示，将一个 `byte t = 2;` 塞到 `bool foo` 里面。这似乎看起来很有违和感，但在 dotnet 运行时里面却又是合法的行为。因为 bool 的长度也和 byte 相同

这就有趣起来了，众所周知，在 dotnet C# 里面的布尔 true 和 false 分别是 1 和 0 的值。如果我强行将一个 2 塞到布尔里面，那会发生什么事情呢？此时的 foo 还是一个合法的布尔值么

答案是 foo 仍然是一个布尔值，如 `if (foo)` 判断绝对能够通过。即 `Console.WriteLine(foo);` 能够输出 `true` 值，且 `if (foo)` 能判断通过，输出 `Console.WriteLine($"if (foo.F1)");` 信息

但 `if (foo == true)` 呢？要知道，现在 foo 是 2 的值，而不是为 `1` 的 `true` 值。大家猜猜这个判断能否通过呢

开答案，那是能够通过的啦。这是为什么呢？其实这个判断条件通过更大的关系是和编译器相关。聪明的编译器认为 `if (foo == true)` 中的 `== true` 可以省略掉，于是 IL 代码里面其实是这样的

```
    IL_008b: ldloc.3      // foo
    IL_008c: stloc.s      V_7

    IL_008e: ldloc.s      V_7
    IL_0090: brfalse.s    IL_00a1

    // [21 1 - 21 2]
    IL_0092: nop

    // [22 5 - 22 47]
    IL_0093: ldstr        "if (foo.F1 == true)"
    IL_0098: call         void [System.Console]System.Console::WriteLine(string)
    IL_009d: nop

    // [23 1 - 23 2]
    IL_009e: nop

    IL_009f: br.s         IL_00ae

    // [25 1 - 25 2]
    IL_00a1: nop

    // [26 5 - 26 47]
    IL_00a2: ldstr        "if (foo.F1 != true)"
    IL_00a7: call         void [System.Console]System.Console::WriteLine(string)
    IL_00ac: nop
```

重新将 IL 翻译为低级 C# 代码如下

```csharp
    if (foo)
    {
        Console.WriteLine("if (foo.F1 == true)");
    }
    else
    {
        Console.WriteLine("if (foo.F1 != true)");
    }
```

可以看到完全将 `== true` 忽略掉了

按照计算机习惯，在 if 判断里面，非 0 就是 true 值。于是为 `2` 的 `foo` 也就能通过 if 的判断啦

但如本文一开始的代码所示，直接和一个布尔变量判断呢？如以下代码所示，此时还能输出什么呢

```csharp
var t1 = true;
if (foo == t1)
{
    Console.WriteLine($"if (foo.F1 == t1)");
}
else
{
    Console.WriteLine($"if (foo.F1 != t1)");
}
```

本次的答案就是输出了 `if (foo.F1 != t1)` 内容，证明和 t1 判断不相等。这是因为此时聪明的编译器不敢进行省略 t1 的值，毕竟这个变量没有标记常量，也许开发者有奇怪的意图呢。于是就真真的执行了一次判断逻辑，其 IL 代码如下

```
    // [29 1 - 29 15]
    IL_00ae: ldc.i4.1
    IL_00af: stloc.s      t1

    // [30 1 - 30 15]
    IL_00b1: ldloc.3      // foo
    IL_00b2: ldloc.s      t1
    IL_00b4: ceq
    IL_00b6: stloc.s      V_8

    IL_00b8: ldloc.s      V_8
    IL_00ba: brfalse.s    IL_00cb

    // [31 1 - 31 2]
    IL_00bc: nop

    // [32 5 - 32 45]
    IL_00bd: ldstr        "if (foo.F1 == t1)"
    IL_00c2: call         void [System.Console]System.Console::WriteLine(string)
    IL_00c7: nop

    // [33 1 - 33 2]
    IL_00c8: nop

    IL_00c9: br.s         IL_00d8

    // [35 1 - 35 2]
    IL_00cb: nop

    // [36 5 - 36 45]
    IL_00cc: ldstr        "if (foo.F1 != t1)"
    IL_00d1: call         void [System.Console]System.Console::WriteLine(string)
    IL_00d6: nop
```

以上 IL 翻译回 C# 代码和原本的代码一样，就真真的用了 `ceq` 去判断了 foo 和 t1 是否相等。由于 foo 是 2 的值，而 t1 是 1 的值，自然从内存的角度上就返回了不相等

本文一开始的代码的最终输出内容如下

```csharp
True
if (foo.F1)
if (foo.F1 == true)
if (foo.F1 != t1)
```

这是一个非常有趣的事情，可以用来编写一些让静态代码审核完全阵亡的代码。比如说从一个数据接收源里面，将一个布尔放入到某个结构体里面，再利用 MemoryMarshal 或 `Unsafe.As` 等方式将数据直接拍到结构体上面。于是此时就可以在结构体里面制造出一个不为 true 或 false 的布尔字段

再在代码逻辑里面，尝试使用此与一个可为空的布尔变量进行判断。对于可为空的布尔判断，额外写一个 else 也不过分吧，毕竟人家还有 null 值呢。于是代码审查将会看到一段必不为空的分支里面，走进了 else 分支里面去了。估计没有对应的数据配合调试，依靠静态代码审查是完全看不出来的

恭喜你，又学了一样可能没有用的技能，和一项绕过静态代码审查的坏知识

好开发者们可不用将此技术用在自己的实际产品项目中哦，不然开发时要带好头盔，免得被其他开发伙伴用板砖拍了

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/e3c800f41d3f0ed46997cb69baa1bd5e87c576a6/Workbench/DeharkallairheaLawgijewho) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/e3c800f41d3f0ed46997cb69baa1bd5e87c576a6/Workbench/DeharkallairheaLawgijewho) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e3c800f41d3f0ed46997cb69baa1bd5e87c576a6
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin e3c800f41d3f0ed46997cb69baa1bd5e87c576a6
```

获取代码之后，进入 Workbench/DeharkallairheaLawgijewho 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
