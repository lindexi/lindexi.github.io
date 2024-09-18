---
title: dotnet C# 警惕可空结构体的方法内部赋值无效
description: 本文将记录一个 C# dotnet 里的一个稍微隐藏的行为，那就是如果有一个结构体存在某个的方法，此方法的作用是修改结构里面的字段或属性的值，那此时将会在可空的结构体调用此方法时，发现没有真正修改到可空结构体局部变量本身

<!--more-->

tags: dotnet C#
category: 
---

<!-- CreateTime:2024/3/16 16:09:32 -->

<!-- 发布 -->
<!-- 博客 -->

其实这个问题非常好理解，只不过可能在编写代码的时候，由于语法原因，可能不小心才会踩到这样的坑。先来讲讲我踩到这个坑的故事，这是我在编写一个 WPF 应用程序时，我有一段逻辑代码，我需要将一个 WPF 的 Rect 类型进行 Union 一个点，从而求出加入包含某个点的矩形范围

简单的编写代码如下

```csharp
        Rect? rect1 = new Rect(10, 10, 10, 10);
        rect1.Value.Union(new Point(100, 100));
```

以上代码的 `rect1.Value.Union` 则是将传入的点参数加入到 Rect 包含范围里面，将会在 Union 方法里面修改 Rect 的宽度高度和 X 和 Y 坐标

预期以上代码的能够将 Rect 的范围，也就是右下角坐标放大到 100x100 的坐标，然而通过以下代码输出到控制台时，却发现结果不符合预期

```csharp
        Console.WriteLine($"{rect1.Value.X} {rect1.Value.Y} {rect1.Value.Width} {rect1.Value.Height}");
```

以上控制台输出的内容如下

```
10 10 10 10
```

可以看到 rect1 局部变量依然保持初始的值

此时我以为是代码哪里没有写对，我就写了一个非可空的 rect2 变量

```csharp
        Rect rect2 = new Rect(10, 10, 10, 10);
```

依然和 rect1 一样调用 Union 方法

```csharp
        rect2.Union(new Point(100, 100));
```

此时的输出就符合预期了

```csharp
        Console.WriteLine($"{rect2.X} {rect2.Y} {rect2.Width} {rect2.Height}");
```

以上代码输出的是

```
10 10 90 90
```

意味着右下角坐标放大到 100x100 的坐标

这里需要提一下的是 WPF 的坐标系是左上角是坐标 0 点，从左往右 X 越来越大，从上到下 Y 越来越大

那这究竟是为什么呢？为什么可空会有此影响呢？为了了解这个问题，防止是 WPF 的 Rect 投毒，咱自己编写一个名为 Foo 的结构体，在这个结构体里面添加一个方法，用于修改结构体里面的属性

```csharp
struct Foo
{
    public int Number {  set; get; }

    public void SetNumber(int value) => Number = value;
}
```

尝试调用 SetNumber 方法给可空结构体赋值，如以下代码

```csharp
        Foo? foo = new Foo();
        foo.Value.SetNumber(100);
        Console.WriteLine(foo.Value.Number);
```

运行以上代码，可以看到控制台输出的是 0 的值，也就是说 SetNumber 方法没有能够给 foo 局部变量的 Number 属性赋值

其实如果大家尝试不通过 SetNumber 赋值，而是直接对 Number 属性赋值，就能看到其实在 dotnet 里面已经尝试给出拦截了，如以下代码将会提示构建失败

```csharp
        foo.Value.Number = 100;
```

以上代码会构建失败，提示如下

```
error CS1612: 无法修改“Foo?.Value”的返回值，因为它不是变量
```

这是因为 `foo.Value.Number = 100;` 这句话里面隐式包含了从 foo 可空类型里面取出 Value 的代码。根据 C# 基础知识可以知道，局部变量获取结构体就是获取结构体的一份在栈上的拷贝

换句话说就是如果想要获取一个结构体的拷贝可以如何做？获取一个结构体或准确来说一个值类型的拷贝可以直接通过局部变量赋值，赋值就是拷贝的过程，如 `int a = b;` 一样，就让 a 获取了 b 的拷贝值

于是 `foo.Value` 其实就是隐藏了一个获取 foo 可空类型的 Value 内容的隐藏的变量，如果此时写 `foo.Value.SetNumber(100)` 则是对隐藏的变量调用 SetNumber 方法，自然修改的是这个隐藏的变量，而不是 foo 可空类型本身的结构体的值

从 IL 层面可以更好看出来 `foo.Value.SetNumber(100)` 这句话的实际逻辑

```IL
    IL_0011: ldloca.s     foo
    IL_0013: call         instance !0/*valuetype DurkalbaliNerkalcemya.Foo*/ valuetype [System.Runtime]System.Nullable`1<valuetype DurkalbaliNerkalcemya.Foo>::get_Value()
    IL_0018: stloc.1      // V_1
    IL_0019: ldloca.s     V_1
    IL_001b: ldc.i4.s     100 // 0x64
    IL_001d: call         instance void DurkalbaliNerkalcemya.Foo::SetNumber(int32)
    IL_0022: nop
```

可以看到这里其实有一个隐藏的名为 V_1 的局部变量，大概实际的运行的代码如下

```csharp
        var temp = foo.Value;
        temp.SetNumber(100);
```

从以上的代码相信大家也就知道为什么可空结构体的方法对内部的属性赋值无效的原因了，从 `var temp = foo.Value;` 这一句其实就获取了结构体的拷贝了，之后 SetNumber 的对内部属性的赋值自然就无法影响到可空类型里面的结构体了

这是一个很简单的基础的 C# 结构体值类型的知识，只是可能有时写成一句话了，就没看出来

以上的 `foo.Value.SetNumber(100)` 的符合预期的行为的改法如下

```csharp
        Foo temp = foo.Value;
        temp.SetNumber(100);
        foo = temp;
```

相对来说需要多写几句话

现在有了 record 和 readonly struct 的出现，很多时候结构体从设计上都不会让方法去修改自身，大部分推荐的都是返回新的结构体回来让原本的结构体保持不变

按照以上方式的优化如下

```csharp
readonly record struct Foo(int Number)
{
    public Foo SetNumber(int value) => this with { Number = value };
}
```

优化之后的代码依然十分简单，此时的赋值代码就可以修改为如下逻辑

```csharp
        Foo? foo = new Foo();
        foo = foo.Value.SetNumber(100);
```

由于是直接修改 foo 可空类型局部变量本身，自然就可以完成进行赋值

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/066cae4e4f6aa4f31d3e43eca9c278aa7b546b60/DurkalbaliNerkalcemya) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/066cae4e4f6aa4f31d3e43eca9c278aa7b546b60/DurkalbaliNerkalcemya) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 066cae4e4f6aa4f31d3e43eca9c278aa7b546b60
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 066cae4e4f6aa4f31d3e43eca9c278aa7b546b60
```

获取代码之后，进入 DurkalbaliNerkalcemya 文件夹，即可获取到源代码
