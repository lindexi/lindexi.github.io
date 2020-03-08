# C# 高级面试题

很少会有人可以答对，如果你遇到一个来面试的人实在嚣张，就可以用本文的题去打击

本文内容就看着玩，请不要在严肃的面试中问题这样的题目

<!--more-->
<!-- CreateTime:2020/2/17 9:06:06 -->


如果面试到一个人可以回答出下面的题目也不能证明他的技术很强，只能说明他了解很多C#相关，或者他看过我的博客

<div id="toc"></div>

## 循环下面的代码

请在下面的代码的注释处填写代码，让函数 Foo 里面的代码输出

```csharp
        static void Main(string[] args)
        {
            // 请在此处写代码，调用 Foo 函数内的输出代码
        }

        private static void Foo()
        {
            try
            {
                while (true)
                {
                }
            }
            finally
            {
                Console.WriteLine("尝试调用 Foo 函数执行这一句代码");
            }
        }
```

参考答案

使用一个线程调用的方式，调用之后结束线程，此时就会输出

```csharp
        static void Main(string[] args)
        {
            // 请在此处写代码，调用 Foo 函数内的输出代码

            var thread = new Thread(Foo);
            thread.Start();
            Task.Delay(100).Wait();
            thread.Abort();// 这时就会结束循环

            Console.Read();
        }
```

注意，在 dotnet core 不支持 Abort 方法

## 从空转换

请写出 IFoo 和 Foo 的实现，让下面的代码不会抛出空异常

```csharp
        static void Main(string[] args)
        {
            Foo foo = (IFoo) null;
            foo.Name = "lindexi";

            Console.Read();
        }
```

参考答案

```csharp
    class IFoo
    {

    }

    class Foo
    {
        public string Name { get; set; }

        public static implicit operator Foo(IFoo foo)
        {
            return new Foo();
        }
    }
```

## 等待不存在的类

请添加新的类的代码让下面的代码编译通过

```csharp
    class Program
    {
        static async Task Main(string[] args)
        {
            Foo foo = await (object) null;
            foo.Name = "lindexi";

            Console.Read();
        }
    }

    public class Foo
    {
        public string Name { get; set; }
    }
```

参考答案

```csharp
   public class HeabdsdnbKevx : INotifyCompletion
    {
        public bool IsCompleted { get; }

        public Foo GetResult()
        {
            return new Foo();
        }

        /// <inheritdoc />
        public void OnCompleted(Action continuation)
        {
        }
    }

    public static class RelelnisSou 
    {
        public static HeabdsdnbKevx GetAwaiter(this object obj)
        {
            return new HeabdsdnbKevx();
        }
    }
```

再高级一点，写出下面的代码

```csharp
        static async Task Main(string[] args)
        {
            await await await await await await await await await await await await
                await await await await await await await "林德熙是逗比";
        }
```

其实很简单，也就是将 GetResult 修改一下，在上面的代码修改

```csharp
        public string GetResult()
        {
            return "林德熙是逗比";
        }
```

因为返回值是 string 所以又可以继续等待

## 如何不执行 finally 里面的代码

这里有一个代码，需要让 finally 里面的代码不执行，现在你只能写 Foo 方法，同时这个方法不能运行无限长时间

```csharp
            try
            {
                Foo();
            }
            finally
            {
                Console.WriteLine("不要让这个代码运行");
            } 
```

参考答案

因为不能让 Foo 运行无限长，就不能使用无限循环的方法，可以使用的方法有 Environment.FailFast 或 Environment.Exit 退出

```csharp
private static void Foo()
{
    Environment.Exit(0);
}
```

或者进行堆栈溢出，如下面代码

```csharp
        private static void Foo()
        {
            Foo();
        }
```

或者 少珺 小伙伴的不安全代码申请

```csharp
        private static void Foo()
        {
            unsafe
            {
                var n = stackalloc int[int.MaxValue];
            }
        }
```

或者干掉自己进程

```csharp
        private static void Foo()
        {
           Process.GetCurrentProcess().Kill();
        }
```


但是申请大内存和退出当前线程方法都会让 finally 执行

```csharp
        private static void Foo()
        {
            var n = new int[int.MaxValue];
        }
        // 虽然提示内存不够，但是finally依然可以运行
```

退出当前线程抛出的是线程中断异常，和其他异常一样都能执行 finally 代码

```csharp
        private static void Foo()
        {
            Thread.CurrentThread.Abort();
        }
```

注意，在 dotnet core 不支持 Abort 方法

另外，如果进入 try 是不能使用 goto 跳出但不执行 finally 代码

如果是在 VisualStudio 调试，在 Foo 执行完之后，在 VS 里把调试箭头拖到 finally 的后面


## 请问下面代码输出多少

请问下面的代码的 n 的值是多少？

```csharp
        class Foo
        {
            public int N { get; } = 1;
        }

            Foo foo = null;
            var n = 2 + foo?.N ?? 1;

            Console.WriteLine(n);
```

参考答案

1

可能有小伙伴认为在 `2 + foo?.N` 这时如果 foo 为空就应该返回 `??` 后面的值，但是这是不对的上面的代码是和下面的代码差不多等同的

```csharp
            if (foo == null)
            {
                n = 1;
            }
            else
            {
                n = 2 + foo.N;
            }
```

而不是和下面的代码等价的

```csharp
           if (foo == null)
            {
                n = 2 + 1;
            }
            else
            {
                n = 2 + foo.N;
            }
```

在表达里面只有 `?` 的值为空，那么就不会执行

等等，为什么上面的代码说的是差不多等同而不是等价，因为尝试运行下面代码，会看到 Hi 输出，多谢 [九鼎](https://github.com/imba-tjd) 指出

```csharp
using System;
class Test
{
    class Foo
    {
        public int N
        {
            get
            {
                Console.WriteLine("Hi.");
                return 1;
            }
        }
    }

    static void Main()
    {
        Foo foo = null;
        Foo foo2 = new Foo();
        var n = 2 + foo?.N + foo2.N ?? 1;
        Console.WriteLine(n);
    }
}
```

上面代码中，第一个 `foo?.N` 会进行判断，因为 foo 不存在，所以整个表达式没有执行，但是表达式内的逻辑依然执行

或者试试下面代码就知道了

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            Foo foo = null;

            int n = Lindexi() + foo?.N ?? 1;

            Console.WriteLine(n);
        }

        private static int Lindexi()
        {
            Console.WriteLine("林德熙是逗比");
            return 2;
        }
    }

    class Foo
    {
        public int N { get; } = 1;
    }
```

## 模式匹配

请问下面代码输出什么？

```csharp

    class B
    {
        public static int operator &(B left, B right) => 1;
        public static int operator >(B left, B right) => 2;
        public static int operator <(B left, B right) => 3;

        public static int operator &(bool left, B right) => 5;
        public static int operator >(bool left, B right) => 6;
        public static int operator <(bool left, B right) => 7;
    }

        private static B B { get; }

        static void Main(string[] args)
        {
            object a = null;
            B c = null;
            Console.WriteLine(a is B b & c);
            Console.WriteLine(a is B b1 > c);
            Console.WriteLine(a is B b2 < c);

            a = new B();

            Console.WriteLine(a is B b5 & c);
            Console.WriteLine(a is B b6 > c);
            Console.WriteLine(a is B b7 < c);

        }
```

也许这是全部题目里面最简单的一道题

请看 [C# 匹配可空变量](https://blog.lindexi.com/post/c-%E5%8C%B9%E9%85%8D%E5%8F%AF%E7%A9%BA%E5%8F%98%E9%87%8F )

其实这里的 `a is B` 用的 `B` 是 `class` 不是定义的属性，对 `a is B b5` 返回的是 `bool` 所以将会是 `bool` 与 `B` 之间的运算

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
