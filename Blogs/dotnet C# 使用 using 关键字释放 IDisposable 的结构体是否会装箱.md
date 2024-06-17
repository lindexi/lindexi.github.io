在 C# 里面的 using 关键字可以非常方便调用 IDisposable 接口的 Dispose 方法，进行一些资源的释放或实现有趣的逻辑的执行

<!--more-->


<!-- 发布 -->
<!-- 博客 -->

配合 using 关键字使用的类型需要继承 IDisposable 接口，根据基础的 C# 知识，大家都知道 using 关键字其实会自动在 IL 层拆开为在 finally 里面调用 Dispose 方法。如以下的简单代码

```csharp
IDisposable disposable = xxx;
using (disposable)
{
   ... // 执行一些代码
}
```

将会被转换为大概如下的代码

```csharp
IDisposable disposable = xxx;

try
{
   ... // 执行一些代码
}
finally
{
    disposable.Dispose();
}
```

再根据另一个 C# 基础知识，如果一个结构体被当成接口使用，即使用接口承接结构体，那这个过程将会进行装箱。结构体装箱将意味着需要更高的开销，将会导致这个过程创建一个对象，频繁使用可能存在一点 GC 压力

一般情况下会在这里使用结构体的业务，都是期望 GC 没有压力的。如果 using 会导致结构体转换为接口，从而导致装箱，无疑这个过程是有伤的

额外提一下为什么结构体转换为接口将需要装箱的过程，这是因为结构体将会在接口里丢失结构体信息，由于结构体在局部变量作用范围时是存放在栈上的，如作为方法参数传递时，也都是在栈范围的。再使用方法调用参数传递作为例子，结构体在栈上这就意味着需要运行时知道压栈空间的大小。结构体是明确知道其占用空间的，但是接口则不然，这部分将导致无法进行编译时处理，如果依然让接口使用结构体形式在内存中存放，将会由其占用空间不可知导致方法调用无法正常工作。那运行时能够知道一个接口是由结构体组成的，那为什么运行时不做呢？其实运行时也只有在将结构体传递给接口变量那一刻之后，后续就不可知了，因为运行时也没有为此分配更多的内存空间来进行记录，一旦分配更多的内存空间来记录一个接口是否实际为结构体，那这个分配成本就和装箱差不多了。除了方法调用装箱之外，还有数组集合等一系列问题。数组问题可以稍微提一下就是如果一个接口的数组里面既然存放有几层此接口的结构体和类型，那这个接口数组要怎么办？数组本身需要明确的分配空间大小，如果开发者期望这么玩，那就不好玩了，究竟一个数组里面的元素应该占用多大的空间才合适，这是在数组创建的时候不知道的，只有对象放入到数组里面时，数组才能知道。如此大家也能看到结构体给接口时，进行装箱能完全将结构体放入到对象里面，解决了非常多的问题，这也就是为什么如此设计的原因

那本文提出的问题呢？答案是不会装箱的。毕竟 using 只是一个语法而已，聪明的构建器自然不会做出先将结构体装箱给到接口再调用接口方法的事情

如以下代码定义了一个结构体继承 IDisposable 接口

```csharp
internal struct DisposableStruct : IDisposable
{
    public void Dispose()
    {

    }
}
```

使用如下代码时，不会出现装箱问题

```csharp
using var disposableStruct = new DisposableStruct();
Console.WriteLine("Hello, World!");
```

从 IL 层面看，以上代码的逻辑如下

```IL
    IL_0000: ldloca.s     V_0
    IL_0002: initobj      KiheekawyalawGechurwagocal.DisposableStruct
    .try
    {
      IL_0008: ldstr        "Hello, World!"
      IL_000d: call         void [System.Console]System.Console::WriteLine(string)
      IL_0012: leave.s      IL_0022
    } // end of .try
    finally
    {
      IL_0014: ldloca.s     V_0
      IL_0016: constrained. KiheekawyalawGechurwagocal.DisposableStruct
      IL_001c: callvirt     instance void [System.Runtime]System.IDisposable::Dispose()
      IL_0021: endfinally
    } // end of finally
    IL_0022: ret
```

以上的 IL 重新转换为 C# 代码如下

```csharp
    DisposableStruct disposableStruct = new DisposableStruct();

    try
    {
        Console.WriteLine("Hello, World!");
    }
    finally
    {
        disposableStruct.Dispose();
    }
```

从 IL 上没有看到任何装箱代码，从转换回的 C# 代码也可以看到没有任何的将结构体给到接口的代码

通过以上的说明，大家可以放心给继承 IDisposable 的结构体使用 using 语法，这是一个非常高性能的做法

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/61400df7abb7994de43efaeae1187abf34e16524/Workbench/KiheekawyalawGechurwagocal) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/61400df7abb7994de43efaeae1187abf34e16524/Workbench/KiheekawyalawGechurwagocal) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 61400df7abb7994de43efaeae1187abf34e16524
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 61400df7abb7994de43efaeae1187abf34e16524
```

获取代码之后，进入 Workbench/KiheekawyalawGechurwagocal 文件夹，即可获取到源代码

