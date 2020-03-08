# C# 很少人知道的科技

本文来告诉大家在C#很少有人会发现的科技。即使是工作了好多年的老司机也不一定会知道，如果觉得我在骗你，那么请看看下面

<!--more-->
<!-- CreateTime:2019/11/29 10:12:43 -->

<div id="toc"></div>

因为C#在微软的帮助，已经从原来很简单的，到现在的很好用。在10多年，很少人知道微软做了哪些，我在网上找了很多大神的博客，然后和很多大神聊天，知道了一些科技，所以就在这里说。如果大家看到这个博客里面没有的科技，请告诉我

## 无限级判断空

在 C# 6.0 可以使用`??`判断空，那么就可以使用下面代码

```csharp
            var v1 = "123";
            string v2 = null;
            string v3 = null;

            var v = v1 ?? v2 ?? v3;
```

实际上可以无限的使用`??`判断前面一个函数为空，那么问题来了，下面的代码输出的是多少？

```csharp
var n = 2 + foo?.N ?? 1;
```

上面代码的 foo 就是空的，那么 n 是多少？是 1 还是 2 还是 3 还是空？

想要了解这道题的推导过程请看[C# 高级面试题](https://blog.lindexi.com/post/C-%E9%AB%98%E7%BA%A7%E9%9D%A2%E8%AF%95%E9%A2%98.html ) 里面写了很多老司机都不一定能解出

## using 省略长的定义

例如有这个代码，使用了很多的 List ，里面有很多定义

```csharp
var foo =
                new System.Collections.Generic.Dictionary<
                    System.Collections.Generic.List<System.Collections.Generic.List<string>>, string>();
```

可以看到需要写很多代码，如果这个值作为参数，才是可怕。

一个简单的方法是使用 `using HvcnrclHnlfk= System.Collections.Generic.Dictionary<System.Collections.Generic.List<System.Collections.Generic.List<string>>,string>;` ，这个文件里的所有定义都可以使用 `using` 后面的值可以代替。

```csharp
var foo = new HvcnrclHnlfk();
```

## 辣么大

实际上我有些不好意思，好像刚刚说的都是大家都知道的，那么我就要开始写大家很少知道

```csharp
          Func<string,string, EventHandler> foo = (x, y) => (s, e) =>
            {
                var button = (Button) s;
                button.Left = x;
                button.Top = y;
            };

            Button1.Click += foo(0, -1);
```

看一下就知道这个定义可以做什么

当然使用委托可是会出现这个问题，请问下面的代码实际调用的是哪个委托，下面代码的 a 和 b 和 c 都是 `Action` 委托，同时都不是空的

```csharp
((a + b + c) - (a + c))();
```

## 冲突的类型

如果遇到两个命名空间相同的类型，很多时候都是把命名空间全写

```csharp
var webControl = new System.Web.UI.WebControls.Control();
var formControl = new System.Windows.Forms.Control();
```

如果经常使用这两个控件，那么就需要写空间，代码很多，但是微软给了一个坑，使用这个可以不用写空间

```csharp
using web = System.Web.UI.WebControls;
using win = System.Windows.Forms;

web::Control webControl = new web::Control();
win::Control formControl = new win::Control();
```

参见：https://stackoverflow.com/a/9099/6116637

## extern alias 

如果使用了两个 dll ，都有相同命名空间和类型，那么如何使用指定的库

```csharp
//a.dll

namespace F
{
	public class Foo
	{

	}
}

//b.dll

namespace F
{
	public class Foo
	{
		
	}
}

```

这时就可以使用 extern alias

参见：[C#用extern alias解决两个assembly中相同的类型全名 - fresky - 博客园](http://www.cnblogs.com/fresky/archive/2012/12/24/2831697.html )

## 字符串

大家看到了 C# 6.0 的`$`，是不是可以和`@`一起？

```csharp
            var str = "kktpqfThiq";
            string foo = $@"换行
{str}";
```

注意两个的顺序，反过来直接告诉你代码不能这样写

## 表达式树获取函数命名

定义一个类，下面通过表达式树从类获得函数命名

```csharp
    class Foo
    {
        public void KzcSevfio()
        {
        }
    }
```

```csharp
       static void Main(string[] args)
        {
            GetMethodName<Foo>(foo => foo.KzcSevfio());
        }

        private static void GetMethodName<T>(Expression<Action<T>> action) where T : class
        {
            if (action.Body is MethodCallExpression expression)
            {
                Console.WriteLine(expression.Method.Name);
            }
        }
```

这样就可以拿到函数的命名

## 特殊关键字

实际上有下面几个关键字是没有文档，可能只有垃圾微软的编译器才知道

```csharp
__makeref

__reftype

__refvalue

__arglist
```

不过在 C# 7.2 都可以使用其他的关键字做到一些，详细请看我的 C# 7.0 博客

## DebuggerDisplay

如果想要在调试的时候，鼠标移动到变量显示他的信息，可以重写类的 ToString 

```csharp
    public sealed class Foo
    {
        public int Count { get; set; }

        public override string ToString()
        {
            return Count.ToString();
        }
    }
```

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F201831693435.jpg)

但是如果 ToString 被其他地方用了，如何显示？

垃圾微软告诉大家，使用 DebuggerDisplay 特性

```csharp
    [DebuggerDisplay("{DebuggerDisplay}")]
    public sealed class Foo
    {
        public int Count { get; set; }

        private string DebuggerDisplay => $"(count {Count})";
    }
```

他可以使用私有的属性、字段，使用方法很简单

参见[Using the DebuggerDisplay Attribute](https://msdn.microsoft.com/en-us/library/x810d419.aspx )

## 使用 Unions （C++ 一样的）

如果看到 C++ 可以使用内联，不要说 C# 没有，实际上也可以使用 FieldOffset ，请看下面

```csharp
[StructLayout(LayoutKind.Explicit)]
public class A
{
    [FieldOffset(0)]
    public byte One;

    [FieldOffset(1)]
    public byte Two;

    [FieldOffset(2)]
    public byte Three;

    [FieldOffset(3)]
    public byte Four;

    [FieldOffset(0)]
    public int Int32;
}
```

这时就定义了`int`变量，修改他就是修改其他的三个

```csharp
     static void Main(string[] args)
    {
        A a = new A { Int32 = int.MaxValue };

        Console.WriteLine(a.Int32);
        Console.WriteLine("{0:X} {1:X} {2:X} {3:X}", a.One, a.Two, a.Three, a.Four);

        a.Four = 0;
        a.Three = 0;
        Console.WriteLine(a.Int32);
    }
``` 

这时会输出

```csharp
2147483647
FF FF FF 7F
65535
```

## 接口默认方法

实际上可以给接口使用默认方法，使用的方式

```csharp
public static void Foo(this IF1 foo)
{
     //实际上大家也看到是如何定义
}
```

## 数字格式

```csharp
string format = "000;-#;(0)";

string pos = 1.ToString(format);     // 001
string neg = (-1).ToString(format);  // -1
string zer = 0.ToString(format);     // (0)
```

参见：[自定义数字格式字符串 ](https://docs.microsoft.com/zh-cn/dotnet/standard/base-types/custom-numeric-format-strings#SectionSeparator )

## stackalloc 

实际上很多人都不知道这个，这是不安全代码，从栈申请空间

```csharp
int* block = stackalloc int[100]; 
```

参见：[stackalloc ](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/stackalloc )

## 调用堆栈

如果需要获得调用方法的堆栈，可以使用[这个文章的方法](https://lindexi.gitee.io/post/WPF-%E5%88%A4%E6%96%AD%E8%B0%83%E7%94%A8%E6%96%B9%E6%B3%95%E5%A0%86%E6%A0%88.html )

```csharp
  class Program
    {
        static void Main(string[] args)
        {
            var foo = new Foo();
            foo.F1();
        }
    }

    public sealed class Foo
    {
        public void F1()
        {
            F2();
        }

        void F2()
        {
            var stackTrace = new StackTrace();
            var n = stackTrace.FrameCount;
            for (int i = 0; i < n; i++)
            {
                Console.WriteLine(stackTrace.GetFrame(i).GetMethod().Name);
            }
        }
    }
```

输出

```csharp
F2
F1
```

参见：[WPF 判断调用方法堆栈](https://lindexi.gitee.io/post/WPF-%E5%88%A4%E6%96%AD%E8%B0%83%E7%94%A8%E6%96%B9%E6%B3%95%E5%A0%86%E6%A0%88.html )

## 指定编译

如果使用 Conditional 可以让代码在指定条件不使用，我写了这个代码，在 Release 下就不会使用 F2

```csharp
    public sealed clas Foo
    {
        public Foo F1()
        {
            Console.WriteLine("进入F1");
            return this;
        }

        [Conditional("DEBUG")]
        public void F2()
        {
            Console.WriteLine("F2");
        }
    }
```

简单让代码跑一下

```csharp
        static void Main(string[] args)
        {
            var foo = new Foo();
            foo.F1();
            foo.F2();
        }
```

结果是什么，大家也知道，在 Debug 和 Release 输出是不相同。但是这么简单的怎么会在这里说，请大家看看这个代码输出什么

```csharp
     static void Main(string[] args)
        {
            var foo = new Foo();
            foo.F1().F2();
        }
```

实际上在 Release 下什么都不会输出，F1 不会被执行

## true 判断

下面写个见鬼的代码

```csharp
            var foo = new Foo(10);

            if (foo)
            {
                Console.WriteLine("我的类没有继承 bool ，居然可以这样写");
            }
```

没错 Foo 没有继承 bool 居然可以这样写

实际上就是重写 true ，请看代码

```csharp
    public class Foo
    {
        public Foo(int value)
        {
            _count = value;
        }

        private readonly int _count;

        public static bool operator true(Foo mt)
        {
            return mt._count > 0;
        }

        public static bool operator false(Foo mt)
        {
            return mt._count < 0;
        }
    }

```

是不是觉得很多有人这样写，下面让大家看一个很少人会知道的科技，感谢[walterlv](https://walterlv.github.io/ )

## 重写运算返回

很少人知道实际上重写 `==` 可以返回任意的类型，而不是只有 bool ，请看下面代码

![](http://image.acmx.xyz/lindexi%2F2018422105951305.jpg)

是可以编译通过的，因为我重写运算

```csharp
   class Foo
    {
        public int Count { get; set; }
     
        public static string operator ==(Foo f1, Foo f2)
        {
            if (f1?.Count == f2?.Count)
            {
                return "lindexi";
            }

            return "";
        }

        public static string operator !=(Foo f1, Foo f2)
        {
            return "";
        }
    }
```

可以重写的运算很多，返回值可以自己随意定义。

## await 任何类型

```csharp
await "林德熙逗比";

await "不告诉你";
```

这个代码是可以编译通过的，但是只有在我的设备，然后看了这个[博客](https://lindexi.gitee.io/post/C-await-%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.html )，可能你也可以在你的设备编译

其实 await 是可以写很多次的，如下面代码

```csharp
await await await await await await await await await await await await await await await await await await await await await await await "林德熙逗比";
```

## 变量名使用中文

实际上在C#支持所有 Unicode ，所以变量名使用中文也是可以的，而且可以使用特殊的字符

```csharp
        public string H\u00e5rføner()
        {
            return "可以编译";
        }
```

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018316102739.jpg)

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F2018316102848.jpg)

## if this == null

一般看到下面的代码都觉得是不可能

```csharp
if (this == null) Console.WriteLine("this is null");
```

如果在 if 里面都能使用 this == null 成立，那么一定是vs炸了。实际上这个代码还是可以运行。

在一般的函数，如 Foo ，在调用就需要使用`f.Foo()`的方法，方法里 this 就是 f ，如果 `f == null` 那么在调用方法就直接不让运行，如何到方法里的判断

```csharp
f.Foo(); //如果 f 为空，那么这里就不执行

void Foo()
{
   // 如果 this 为空，怎么可以调用这个方法
   if (this == null) Console.WriteLine("this is null");
}
```

实际上是可以做的，请看[（C#）if (this == null)？你在逗我，this 怎么可能为 null！用 IL 编译和反编译看穿一切 - walterlv](https://walterlv.github.io/post/this-could-be-null.html )

如上面博客，关键在修改`callvirt `为 `call`，直接修改 IL 可以做出很多特殊的写法。

那么这个可以用在哪里？可以用在防止大神反编译，如需要使用下面逻辑

```csharp
//执行的代码

//不执行的代码
```

```csharp
if(true)
{
   //执行的代码
}
else
{
   //不执行的代码 
}
```

但是直接写 true 很容易让反编译看到不使用代码，而且在优化代码会被去掉，所以可以使用下面代码

```csharp
if(this == null)
{
   //执行的代码
}
else
{
   //不执行的代码 
}
```
    
实际在微软代码也是这样写，点击[string](https://referencesource.microsoft.com/#mscorlib/system/string.cs,507 )可以看到微软代码

## 重载的运算符

实际上我可以将 null 强转某个类，创建一个新的对象，请看代码

```csharp
Fantastic fantastic = (FantasticInfo) null;
fantastic.Foo();
```

这里的 FantasticInfo 和 Fantastic 没有任何继承关系，而且调用 Foo 不会出现空引用，也就是 fantastic 是从一个空的对象创建出来的。

是不是觉得上面的科技很黑，实际原理没有任何黑的科技，请看代码

```csharp
    public class Fantastic
    {
        private Fantastic()
        {
        }

        public static implicit operator Fantastic(FantasticInfo value) => new Fantastic();

        public void Foo()
        {
        }
    }

    public class FantasticInfo
    {
    }
```

通过这个方式可以让开发者无法直接创建 Fantastic 类，而且在不知道 FantasticInfo 的情况无法创建 Fantastic 也就是让大家需要了解 FantasticInfo 才可以通过上面的方法创建，具体请看[只有你能 new 出来！.NET 隐藏构造函数的 n 种方法（Builder Pattern / 构造器模式） - walterlv](https://walterlv.com/post/hide-your-constructor.html )

课件链接： https://r302.cc/J4gxOX

当然还有新的 [C# 7.0](https://blog.lindexi.com/post/C-7.0.html ) 和 [C# 8.0](https://blog.lindexi.com/post/VisualStudio-2019-%E5%B0%9D%E8%AF%95%E4%BD%BF%E7%94%A8-C-8.0-%E6%96%B0%E7%9A%84%E6%96%B9%E5%BC%8F.html ) 的新的语法

例如下面的内部方法返回自身

## 方法返回自身可以接近无限调用

有一天我看到了下面的代码，你猜小伙伴用个字符的代码定义了Foo这个代码？

```csharp
Foo()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()()();
```

其实只需要定义一个委托，用内部方法实现委托，因为内部方法是可以返回自身，于是就可以使用5行代码写出 Foo 的定义

```csharp
        delegate Foo Foo(); // 定义委托

static void Main(string[] args)
{
            Foo Foo() // 定义内部方法
            {
                return Foo;
            }
}
```

不过括号还不可以无限使用，因为编译器有一个表达式的长度限制

欢迎加入 dotnet 职业技术学院 https://t.me/dotnet_campus 使用 Telegram 方法请看 [如何使用 Telegram](https://blog.lindexi.com/post/%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-Telegram.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
