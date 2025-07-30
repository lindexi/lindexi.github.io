---
title: dotnet C# 基础二进制处理 二进制数组与结构体的互转
description: 本文将告诉大家在 dotnet 里面的二进制基础处理知识，如何在 C# 里面将结构体数组和二进制数组进行相互转换的简单方法
tags: dotnet,C#
category: 
---

<!-- CreateTime:2023/8/21 8:31:57 -->
<!-- 标题： C# 二进制数组与结构体的互转 -->
<!-- 发布 -->
<!-- 博客 -->

尽管本文属于基础入门的知识，但是在阅读之前还请自行了解 C# 里面的结构体内存布局知识

本文将和大家介绍 MemoryMarshal 辅助类，通过这个辅助类用来实现结构体数组和二进制数组的相互转换

先演示如何从结构体数组和二进制数组的相互转换。准确来说是 Span 之间的相互转换，而不是真的转换为数组，只是 Span 的行为表现和数组十分相似

为了方便代码演示，我定义了一个 Foo1 的结构体，本文的全部代码都可以在本文末尾找到下载方法

```csharp
struct Foo1
{
    public int A { get; set; }
    public int B { get; set; }
    public int C { get; set; }
}
```

先创建出一个 Foo1 结构体数组，为了方便演示我还给 Foo1 的各个属性分别赋值，如以下代码

```csharp
        var foo1 = new Foo1()
        {
            A = 1,
            B = 2,
            C = 3,
        };
        var foo1Array = new Foo1[] { foo1 };
```

拿到 Foo1 的数组之后，可以非常方便转换为 Span 类型，只需要调用 `foo1Array.AsSpan()` 即可。接下来将 Foo1 数组转化在二进制数组，准确来说是 `Span<byte>` 类型，代码如下

```csharp
        Span<byte> foo1ByteSpan = MemoryMarshal.AsBytes(foo1Array.AsSpan());
```

此时编写一个辅助方法，将 `foo1ByteSpan` 的内容输出到控制台，方便让大家看到这个 `foo1ByteSpan` 对象就确实是 Foo1 结构体的内存空间的二进制内容

```csharp
        Log(foo1ByteSpan); // 01 00 00 00 02 00 00 00 03 00 00 00

    private static void Log(Span<byte> byteSpan)
    {
        var stringBuilder = new StringBuilder();
        foreach (var b in byteSpan)
        {
            stringBuilder.Append(b.ToString("X2"));
            stringBuilder.Append(' ');
        }

        Console.WriteLine(stringBuilder.ToString());
    }
```

可以看到以上输出的 01 02 03 就是对应 Foo1 结构体的 A 和 B 和 C 属性的值。本文这里没有对 Foo1 结构体进行固定布局等，这一点不够严谨，也就是说我只能和大家保证一定出现 Foo1 结构体的 A 和 B 和 C 属性的值，但是不能保证这些值出现的顺序。如果不了解这部分的知识，还请自行查阅 dotnet 里面的结构体的内存布局优化和内存对齐

接下来开始证明本文以上拿到的 `foo1ByteSpan` 和 `foo1Array` 指向相同的一片内存地址空间，也就是对 `foo1Array` 或 `foo1ByteSpan` 的内存修改，都会相互影响

先修改 `foo1Array` 里面的内容，比如修改一个属性的内容，如以下代码

```csharp
        foo1Array[0].C = 5;

        Log(foo1ByteSpan); // 01 00 00 00 02 00 00 00 05 00 00 00
```

可以看到修改了 C 属性之后，打印出的 `foo1ByteSpan` 也更改了

再尝试修改 `foo1ByteSpan` 的内容，看看是否也能反过来影响到 `foo1Array` 对象

```csharp
        foo1ByteSpan[0] = 6;

        Console.WriteLine(foo1Array[0].A); // 6

        var foo1Span = MemoryMarshal.Cast<byte, Foo1>(foo1ByteSpan);
        Console.WriteLine(foo1Span[0].A); // 6
```

通过以上代码即可证明了 `foo1ByteSpan` 和 `foo1Array` 指向相同的一片内存地址空间，也就是 MemoryMarshal.Cast 和 MemoryMarshal.AsBytes 不是重新申请一片内存空间存放数组内容，而是仅仅编写的代码上的魔法，内存都是相同的一片空间。如此减少了内存空间转换拷贝，可以极大的提升性能，同时也兼顾了安全性

通过 MemoryMarshal.Cast 方法，不仅可以支持结构体和 byte 之间的转换，也可以进行结构体之间的转换，比如再定义一个 Foo2 类型，这个 Foo2 类型和 Foo1 类型有相同的属性只是类型不相同而已，试试使用以下代码进行相互转换

```csharp
        var foo2Span = MemoryMarshal.Cast<Foo1, Foo2>(foo1Span);
        Console.WriteLine(foo2Span[0].A); // 6
        Console.WriteLine(foo2Span[0].B); // 2
        Console.WriteLine(foo2Span[0].C); // 5

struct Foo2
{
    public int A { get; set; }
    public int B { get; set; }
    public int C { get; set; }
}
```

可以看到通过 MemoryMarshal.Cast 是可以实现多个结构体之间的直接转换的，且没有重新在堆上重新开辟数组空间

但是本文以上的代码是不严谨的，以上代码没有固定 Foo1 结构体和 Foo2 结构体的内存布局，以上的代码只是用来告诉大家 MemoryMarshal.Cast 的用法，而不是推荐大家在正式的项目跟随我这么写。如果在正式项目里面，需要确保多个结构体之间的内存布局相同或者是在各个情况下的直接内存转换是符合预期的才能这么做

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/6bd28ceca1e9b73bfda270f9a3a3bddd7b8ebcc4/HallehuwearjewhoQedelqarnalar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6bd28ceca1e9b73bfda270f9a3a3bddd7b8ebcc4/HallehuwearjewhoQedelqarnalar) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6bd28ceca1e9b73bfda270f9a3a3bddd7b8ebcc4
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6bd28ceca1e9b73bfda270f9a3a3bddd7b8ebcc4
```

获取代码之后，进入 HallehuwearjewhoQedelqarnalar 文件夹

从 byte 数组转换为结构体或结构体数组的示例：

如以下的一段 byte 列表，其创建方式如下

```csharp
List<byte> buffer = [];

for (int i = 1; i < 6; i++)
{
    buffer.AddRange([(byte) (1 * i), 0, 0, 0, (byte) (2 * i), 0, 0, 0, (byte) (3 * i), 0, 0, 0]);
}

for (var i = 0; i < buffer.Count; i++)
{
    Console.WriteLine($"[{i}] - {buffer[i]:X2}");
}
```

控制台输出内容如下

```
[0] - 01
[1] - 00
[2] - 00
[3] - 00
[4] - 02
[5] - 00
[6] - 00
[7] - 00
[8] - 03
[9] - 00
[10] - 00
[11] - 00
[12] - 02
[13] - 00
[14] - 00
[15] - 00
[16] - 04
[17] - 00
[18] - 00
[19] - 00
[20] - 06
[21] - 00
[22] - 00
[23] - 00
[24] - 03
[25] - 00
[26] - 00
[27] - 00
[28] - 06
[29] - 00
[30] - 00
[31] - 00
[32] - 09
[33] - 00
[34] - 00
[35] - 00
[36] - 04
[37] - 00
[38] - 00
[39] - 00
[40] - 08
[41] - 00
[42] - 00
[43] - 00
[44] - 0C
[45] - 00
[46] - 00
[47] - 00
[48] - 05
[49] - 00
[50] - 00
[51] - 00
[52] - 0A
[53] - 00
[54] - 00
[55] - 00
[56] - 0F
[57] - 00
[58] - 00
[59] - 00
```

使用 MemoryMarshal.Cast 转换为结构体的 Span 结构，约等于结构体集合，代码如下

```csharp
Span<Foo1> foo1Span = MemoryMarshal.Cast<byte, Foo1>(buffer.ToArray().AsSpan());
```

尝试输出 Foo1 结构体内容，如以下代码

```csharp
foreach (var foo1 in foo1Span)
{
    Console.WriteLine($"foo1.A={foo1.A}");
    Console.WriteLine($"foo1.B={foo1.B}");
    Console.WriteLine($"foo1.C={foo1.C}");
}
```

此时可见结构体内容已经被填充为数组的内容

```csharp
foo1.A=1
foo1.B=2
foo1.C=3
foo1.A=2
foo1.B=4
foo1.C=6
foo1.A=3
foo1.B=6
foo1.C=9
foo1.A=4
foo1.B=8
foo1.C=12
foo1.A=5
foo1.B=10
foo1.C=15
```

以上示例代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8bbf489bc158921db69b3dc1dae38affb696565f/Workbench/WherurgurhemBahegargaw) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/8bbf489bc158921db69b3dc1dae38affb696565f/Workbench/WherurgurhemBahegargaw) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 8bbf489bc158921db69b3dc1dae38affb696565f
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 8bbf489bc158921db69b3dc1dae38affb696565f
```

获取代码之后，进入 Workbench/WherurgurhemBahegargaw 文件夹，即可获取到源代码

更多 dotnet 基础知识相关博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

---

如果只是简单的基础类型，期望转换为二进制 byte 数组表示，可以尝试零分配的写法，完全在栈上分配，无 GC 压力。大概写法示例如下，假定有 `ushort foo` 变量，期望将其转换为 `Span<byte>` 对象

```csharp
Span<byte> byteBuffer = stackalloc byte[sizeof(ushort)];
BitConverter.TryWriteBytes(byteBuffer, (ushort) foo);
```

以上的代码是完全安全的代码，不需要接触指针，也不需要开启不安全开关
