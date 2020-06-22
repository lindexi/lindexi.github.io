# dotnet core 2 使用 DispatchProxy 做动态代理 AOP 入门

本文简单告诉大家如何在 .NET Core 里面使用 AOP 切面编程，使用 DispatchProxy 做任意接口的动态代理实现

<!--more-->
<!-- 发布 -->

使用 DispatchProxy 有一些限制，如只能创建接口的实例。使用 DispatchProxy 可以让咱做到从无中创建出某个实例，这个实例继承咱想要的接口

首先需要创建某个动态代理类继承 DispatchProxy 类，例如创建 Foo 类。此时需要实现 Invoke 接口，这个接口的含义是当代码调用接口里面的函数，包括属性的设置和获取函数的时候将会进入 Invoke 方法

```csharp
        protected override object Invoke(MethodInfo targetMethod, object[] args)
        {
            // 忽略代码
        }
```

因此通过 Invoke 方法可以假装自己是对应的接口的实现，可以进行随意更改执行逻辑以及修改返回值

而 DispatchProxy 另一个核心方法是静态的 Create 方法，这个方法要求传入两个泛形，第一个类型是想要创建出来的实例继承的接口，第二个类型是某个继承 DispatchProxy 的类

例如咱定义了一个 IF1 的接口，此时可以在 Foo 添加如下代码创建出一个不存在的类型实例，这个实例继承了 IF1 接口

```csharp
    public class Foo : DispatchProxy
    {
        public static T GetObject<T>()
        {
            return DispatchProxy.Create<T, Foo>();
        }

        protected override object Invoke(MethodInfo targetMethod, object[] args)
        {
            return "lindexi";
        }
    }
```

假设 IF1 的定义如下

```csharp
    interface IF1
    {
        string F2();
    }
```

此时的使用方法如下

```csharp
            Console.WriteLine(Foo.GetObject<IF1>().F2());
```

执行代码可以看到输出了 `lindexi` 也就是实际调用了 Foo 方法

可以看到 DispatchProxy 的作用就是提供静态方法用于创建继承指定接口的对象，同时让继承 DispatchProxy 的类可以拿到接口所调用的方法，用来修改执行逻辑和返回值

一个实际的使用的例子，提供了调用某个对象的每个方法之前给这个对象调用方法记日志，这是最简便的记日志的方法了，尽管性能很渣

```csharp
    public class LoggingAdvice<T> : DispatchProxy
    {
        private T Object { set; get; }


        public static T CreateLogging(Func<T> creator)
        {
            object proxy = DispatchProxy.Create<T, LoggingAdvice<T>>();
            ((LoggingAdvice<T>)proxy).Object = creator();
            return (T)proxy;
        }

        protected override object Invoke(MethodInfo targetMethod, object[] args)
        {
            Console.WriteLine($"开始执行 {targetMethod.Name}");

            var result = targetMethod.Invoke(Object, args);

            Console.WriteLine($"执行完成 {targetMethod.Name}");

            return result;
        }
    }
```

此时就能做到在方法执行前后添加日志，如这里有一个接口和一个类，使用方法请参考以下示例

```csharp
    class Program
    {
        static void Main(string[] args)
        {
            var foo = LoggingAdvice<IF1>.CreateLogging(() => new Foo());
            foo.F2();
        }
    }

    interface IF1
    {
        string F2();
    }

    class Foo : IF1
    {
        /// <inheritdoc />
        public string F2()
        {
            return "lindexi";
        }
    }
```


参考

[在.NET Core中使用DispatchProxy“实现”非公开的接口 - LamondLu - 博客园](https://www.cnblogs.com/lwqlun/p/11575686.html )

[使用.net core中的类DispatchProxy实现AOP - 欧阳.NET - 博客园](https://www.cnblogs.com/oyang168/p/11853851.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
