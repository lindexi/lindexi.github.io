# dotnet 警惕 C# 的 is var 写法

本文将和大家介绍 C# 语言设计里面，我认为比较坑的一个语法。通过 is var 的写法，会让开发者误以为 null 是不被包含的，然而事实是在这里的 var 是被赋予含义的，将被允许 null 通过判断逻辑，于是就会让开发者收到了奇怪的空异常

<!--more-->
<!-- CreateTime:2023/11/18 10:15:51 -->

<!-- 发布 -->
<!-- 博客 -->

比如看看以下的代码，大家猜猜控制台是否会输出

```csharp
IFoo? foo = null;

if (foo is var f2)
{
    Console.WriteLine($"居然进来了。 F2={f2}");
}
```

答案是控制台居然输出居然进来了，也就是说 null 在 is 判断里面是通过，而 var 的含义似乎不只是一个语法上的可有可无的关键词而已，而是赋予了运行时含义的关键词

换句话说就是在以上代码里面的 var 关键词已经违背了 C# 初始设计 var 里面的含义了。最初的 C# 里面的 var 只是一个在构建过程中可以被平替为具体类型的关键词，是一个不会影响到语义、运行时逻辑的语法而已。然而在 is 这里面，将 var 当成了一个可以处理空值的特殊语法结构

这和咱长久的使用 is 来过滤空值的编程思想是冲突的，我感觉绝大部分开发者在写到 is var 的过程，将会想着应该是自动过滤掉 null 值。然而事实是按照 C# 的新设计（C# 7.0-8.0）来说，这里的 var 是一个模式匹配的语法而已，且 var 不再只是一个可有可无的关键词，而是将会影响运行逻辑的关键词

相信许多开发者会和我一样，第一次编写 is var 的时候，会认为一定会过滤掉空值，导致出现了预期之外的空异常

通过以上的代码测试，可以看到以上代码里面的 var 和 IFoo 是不等价的。咱更进一步编写更多的代码，用来测试一下具体的语法行为，如以下代码的两个 var 的含义是完全不同的

```csharp
IFoo? foo = null;

var f1 = foo;

if (foo is var f2)
{
    Console.WriteLine($"居然进来了。 F2={f2}");
}
```

第一个 var 是传统的用法，只是让开发者省略编写重复的代码，没有影响到任何的语义和运行逻辑。第一个 var 和 IFoo 是等价的

然而第二个 var 在上面代码里面，却不能够平替为 IFoo 类型，试试看替换为 IFoo 类型试试，如以下代码，大家可以看到运行逻辑是完全不相同的

```csharp
var f1 = foo;

if (foo is var f2)
{
    Console.WriteLine($"居然进来了。 F2={f2}");
}

if (foo is IFoo f3)
{
    Console.WriteLine($"不进来");
}
```

如果将 is var 替换为 is IFoo 则非常符合预期的过滤掉 null 值

这个如此奇怪的行为是如何被设计出来的，设计这样的行为为什么能够通过大家的语法评审？难道有这么多的开发者大佬脑袋都被大门夹了？

整个 C# 语言的设计是在不断迭代的，现在已经是 C# 12 了。在当年 C# 7.0 时候引入了 pattern 写法时，大家都为此开森，因为这个语法写起来特别漂亮。然而潜藏的 is var 就在 8.0 的对 pattern 模式匹配里面的更进一步改进里面，不得不被引入了这个奇怪的行为，看看以下咱平时写的很爽的语法

```csharp
static Point Transform(Point point) => point switch
{
    var (x, y) when x < y => new Point(-x, y),
    var (x, y) when x > y => new Point(x, -y),
    var (x, y) => new Point(x, y),
};
```

以上的模式匹配里面其实就隐含了 is var 的定义设计，准确来说 is 和 switch 都属于 C# 语法里面的模式匹配的语法，两者应该都有相同的设计

更何况在过滤空对象时，还可以使用 `is {}` 语法，这就导致了如果将 is var 设计为过滤 null 对象，将会和 `is {}` 语法是重叠的，浪费关键词。为了能够更好的实现比较长的链路短写法，于是就如[官方文档](https://learn.microsoft.com/zh-cn/dotnet/csharp/language-reference/operators/patterns#var-pattern)所述将 var 匹配当成为对一切的匹配，包含 null 对象的匹配

换句话说使用 var 匹配就相当于只是拿出来一个变量而已，而不会做其他任何的处理逻辑。用途之处在于大概如下的代码里面

```csharp
    static bool IsFoo() =>
        GetFxx() is var fxx
        && CheckXx(fxx) is var result
        && DoXxx(result);
```

以上代码可以非常方便的利用短路逻辑和 is var 逻辑取出变量执行后续过程。如此写法的完全展开形式也是非常长的

```csharp
    static bool IsFoo()
    {
        if (GetFxx() is var fxx)
        {
            if (CheckXx(fxx) is var result)
            {
                return DoXxx(result);
            }
        }

        return false;
    }
```

如此可以看来 is var 的设计还是在一些逻辑上可以很好的减少代码量的

这个 is var 的决议最早的有记录的会议可以追溯到 2015 那会，详细请看 https://github.com/dotnet/csharplang/blob/20dde78e36028ac0492035f51e28437a92d1b4f2/meetings/2015/LDM-2015-01-21.md 和 https://github.com/dotnet/csharplang/blob/20dde78e36028ac0492035f51e28437a92d1b4f2/meetings/2015/LDM-2015-03-10-17.md 等会议记录内容

从 IL 层面上看 is var 的语法，可以发现 is var 只是就是一个局部变量赋值，从 IL 上看的 is 判断只是空气而已，什么都没有

如以下的 C# 代码和 IL 的对应，可以看到 `if (foo is var f2)` 和 `var f2 = foo;` 是等价的

```
C#:
    if (foo is var f2)

IL:
    IL_0005: ldloc.0      // foo
    IL_0006: stloc.2      // f2

-------------------------------------
C#:
   var f2 = foo;

IL:
    IL_0007: ldloc.0      // foo
    IL_0008: stloc.1      // f2
```

这和 `if (foo is IFoo f3)` 的逻辑是完全不一样的，如以下的 C# 和 IL 对应代码

```
C#:
    if (foo is IFoo f3)

IL:
    IL_0007: ldloc.0      // foo
    IL_0038: isinst       IFoo
    IL_003d: stloc.1      // f3
    IL_003e: ldloc.1      // f3
    IL_003f: brfalse.s    IL_006a
```

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/2ec91207fff919837fff1c3121d57d0172b4f2bb/FaydeenereqelnairderlaHuwicagall) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/2ec91207fff919837fff1c3121d57d0172b4f2bb/FaydeenereqelnairderlaHuwicagall) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 2ec91207fff919837fff1c3121d57d0172b4f2bb
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 2ec91207fff919837fff1c3121d57d0172b4f2bb
```

获取代码之后，进入 FaydeenereqelnairderlaHuwicagall 文件夹