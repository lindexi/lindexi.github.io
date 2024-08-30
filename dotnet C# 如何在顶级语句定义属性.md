# dotnet C# 如何在顶级语句定义属性

随着 dotnet 6 开始，现在的 C# dotnet 可以使用顶级语句非常方便创建一个小型项目，包含的代码也特别少。本文将和大家介绍如何在顶级语句里面定义属性

<!--more-->
<!-- CreateTime:2024/08/30 07:07:26 -->

<!-- 发布 -->
<!-- 博客 -->

如以下代码是传统的控制台应用程序的代码

```csharp
using System;

namespace Application
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

可以看到即使是一个简单的应用，也需要不少的代码。在 dotnet 6 开始，咱可以使用非常方便的顶级语句代替以上代码的功能，只需一句代码即可

```csharp
Console.WriteLine("Hello World!");
```

详细请看 [顶级语句 - C# 教程 - C# - Microsoft Learn](https://learn.microsoft.com/zh-cn/dotnet/csharp/tutorials/top-level-statements )

然而有些时候，咱需要定义一些属性用来辅助某些特定的业务，却会发现在顶级语句里面定义方法很简单，但是定义属性却报错

如以下代码将不能通过构建

```csharp
Foo = "Hello, World!";

Console.WriteLine(Foo);

static string Foo { set; get; }
```

错误提示内容如下

```
error CS0116: 命名空间不能直接包含字段、方法或语句之类的成员
```

解决方法是写一个名为 Program 的 partial 类，代码如下

```csharp
partial class Program
{

}
```

在 Program 里面定义属性是非常正确的，修改之后的代码如下

```csharp
Foo = "Hello, World!";

Console.WriteLine(Foo);

partial class Program
{
    public static string Foo { set; get; }
}
```

以上代码是可以通过构建的。其根本原因是顶级语句只是一个语法层面的功能，构建之后的代码全部都会被放入到名为 Program 的类型的 Main 方法里面

回顾一开始最简短的如下一句代码的顶级语句

```csharp
Console.WriteLine("Hello World!");
```

其构建出来的代码对应的低级 C# 大概如下

```csharp
using System;
using System.Runtime.CompilerServices;

[CompilerGenerated]
internal class Program
{
  private static void <Main>$(string[] args)
  {
    Console.WriteLine("Hello, World!");
  }

  public Program()
  {
    base..ctor();
  }
}
```

如此可以看到实际上是生成了一个名为 Program 的类型。这时候再写一个 partial 的 Program 就可以与生成的 Program 类合并

换句话说，如下代码也是完全可以构建通过的

```csharp
var program = new Program();

Console.WriteLine("Hello, World!");
```

即不需要手动定义 Program 类，即可使用自动生成的 Program 类型

利用自动生成的 Program 类添加属性，从而被顶级语句方便的访问到，这就是在顶级语句里面添加属性的核心实现原理

```csharp
Foo = "Hello, World!";

Console.WriteLine(Foo);

partial class Program
{
    public static string Foo { set; get; }
}
```

以上代码的对应的低级 C# 代码大概如下

```csharp
using System;
using System.Diagnostics;
using System.Runtime.CompilerServices;

internal class Program
{
  [DebuggerBrowsable(DebuggerBrowsableState.Never)]
  private static string <Foo>k__BackingField;

  private static void <Main>$(string[] args)
  {
    Program.Foo = "Hello, World!";
    Console.WriteLine(Program.Foo);
  }

  public static string Foo
  {
    [CompilerGenerated]
    set
    {
      Program.<Foo>k__BackingField = value;
    }
    [CompilerGenerated]
    get
    {
      return Program.<Foo>k__BackingField;
    }
  }

  public Program()
  {
    base..ctor();
  }
}
```

由于顶级语句是将代码放入到名为 Program 类里面，此时即可通过再写一个 partial 的 Program 用来定义属性，就可以在构建时自动被合入到生成的类里面

可以看到写到一个文件里面的 Foo 属性在构建的时候被和顶级语句合并到一起，如此的代码可以做到看起来十分简单，且不会丢失属性的功能

额外说明一下的是这里定义的属性只能的静态的，在顶级语句里面，除非构建 Program 的实例，否则将无法直接访问到属性

如此即可实现在顶级语句里面定义属性

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/082e98d35ef7f959057b76e1f70010651fa18713/Workbench/DawkuhelwhehairallweeFewewoyelfalar) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/082e98d35ef7f959057b76e1f70010651fa18713/Workbench/DawkuhelwhehairallweeFewewoyelfalar) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 082e98d35ef7f959057b76e1f70010651fa18713
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 082e98d35ef7f959057b76e1f70010651fa18713
```

获取代码之后，进入 Workbench/DawkuhelwhehairallweeFewewoyelfalar 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )