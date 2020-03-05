# C# 判断方法是否被子类重写

本文告诉大家如何通过反射判断某个方法是否被基类重写

<!--more-->
<!-- CreateTime:2019/11/26 8:49:55 -->

<!-- csdn -->

在 C# 如果在类定义 virtual 的方法，那么可以在子类重写，如何判断一个方法被子类可以通过反射

例如创建一个 Foo 定义 Test 虚方法

```csharp
    class Foo
    {
        public virtual void Test()
        {

        }
    }
```

在 Foo 代码可以通过 `GetType()` 方法获取当前的类，如果是可以拿到 Foo 类对象，通过调用 GetType() 方法可以获取对象的类

在 Foo 写 IsOverride 用来判断 Test 方法是否被重写

```csharp
        public bool IsOverride()
        {
            return !(GetType().GetMethod("Test").DeclaringType == typeof(Foo));
        }
```

如果是判断其他方法，请替换 `"Test"` 为对应方法，上面判断方法对属性也可以

如下面代码写 F1 继承重写方法

```csharp
    class F1 : Foo
    {
        /// <inheritdoc />
        public override void Test()
        {
        }
    }
```

运行下面代码可以看到 F1 类输出的是重写方法

```csharp
        static void Main(string[] args)
        {
            Foo f1 = new F1();

            Console.WriteLine(f1.IsOverride()); // true

            f1 = new Foo();
            Console.WriteLine(f1.IsOverride()); // false
        }
```

除了上面方法判断重写，可以使用下面方法判断方法是否重写

```csharp
            var methodInfo = GetType().GetMethod("Test");
            if (methodInfo != methodInfo.GetBaseDefinition())
            {
            	// 重写
            }
```

判断是否重写方法需要使用反射，性能会比较低，如果多次使用，建议缓存。因为类是不能运行时修改的，所以只需要获取类就可以知道是否重写

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bc4e53eb523363a0fe11bdfe89b80169bc33fedf/KababijawrallCairfeeqairwaw) 欢迎小伙伴访问

[Detect if a method was overridden using Reflection (C#) - Stack Overflow](https://stackoverflow.com/questions/2932421/detect-if-a-method-was-overridden-using-reflection-c )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
