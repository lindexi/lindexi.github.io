
# dotnet C# 只创建对象不调用构造函数方法

有时我期望只是创建出对象，但是不要调用对象的构造方法，可以通过使用 FormatterServices 的 GetUninitializedObject 函数来实现只创建对象不调用构造函数方法

<!--more-->


<!-- CreateTime:2021/6/17 20:25:45 -->

<!-- 发布 -->

这个 [FormatterServices.GetUninitializedObject](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatterservices.getuninitializedobject?WT.mc_id=WD-MVP-5003260) 方法大部分是用在做序列化使用的，然而在很多 IOC 容器，也都使用此方法来创建对象，而通过其他方法拿到构造函数

在 WPF 的 XAML 创建对象，也有用到此方法，详细请看 [dotnet 读 WPF 源代码笔记 XAML 创建对象的方法](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-XAML-%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%96%B9%E6%B3%95.html )

以下是一个实现的例子

```csharp
            Foo foo = null;
            try
            {
                foo = (Foo) FormatterServices.GetUninitializedObject(typeof(Foo));
                var constructorInfo = typeof(Foo).GetConstructor(new Type[0]);
                constructorInfo!.Invoke(foo, null);
            }
            catch
            {
            }

class Foo
{

}
```

此方法可以用来处理在构造函数时，如果抛出了异常，但是此对象的 Dispose 需要被显式调用的问题。因为如果在构造函数抛出异常，那么在 C# 代码层面将拿不到此对象，也就无法调用对应的 Dispose 释放

如以下代码，可以看到 Foo 对象依然是空

```csharp
        private void F1()
        {
            Foo foo = null;
            try
            {
                foo = new Foo();
            }
            catch
            {
                // 忽略
            }
        }

    class Foo : IDisposable
    {
        public Foo()
        {
            throw new Exception("lindexi is doubi");
        }

        ~Foo()
        {
        }

        public void Dispose()
        {
            GC.SuppressFinalize(this);
        }
    }
```

此时如果期望调用 Foo 对象的 Dispose 方法，将会因为拿不到对象而无法调用

解决此方法的做法就是通过只创建对象而不调用构造的方法，先拿到对象然后再调用构造，如果构造出错，依然还可以调用对象的 Dispose 方法

```csharp
        private void F2()
        {
            Foo foo = null;
            try
            {
                foo = (Foo) FormatterServices.GetUninitializedObject(typeof(Foo));
                var constructorInfo = typeof(Foo).GetConstructor(new Type[0]);
                constructorInfo!.Invoke(foo, null);
            }
            catch
            {
                // 忽略
            }
            finally
            {
                try
                {
                    foo?.Dispose();
                }
                catch
                {
                    // 可以调用到 Dispose 方法
                }
            }
        }

    class Foo : IDisposable
    {
        public Foo()
        {
            throw new Exception("lindexi is doubi");
        }

        ~Foo()
        {
            Dispose();
        }

        public void Dispose()
        {
            GC.SuppressFinalize(this);

            throw new Exception($"lsj is doubi");
        }
    }
```

这个设计可以用来解决，如果对象的构造函数还没完全完成，调用释放函数将会抛出异常。如果没有使用如上方法，那么在释放函数的异常将会在 GC 回收线程抛出，而让应用程序退出

这就是为什么有很多容器和底层库喜欢使用此方法创建对象的原因

本文代码还请到 [github](https://github.com/lindexi/lindexi_gd/tree/11077dd21a4ee5314757536ca379ecca6956b040/HojeneceabuHallwhallhebo) 或 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/11077dd21a4ee5314757536ca379ecca6956b040/HojeneceabuHallwhallhebo) 上阅读代码

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 11077dd21a4ee5314757536ca379ecca6956b040
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 HojeneceabuHallwhallhebo 文件夹

[FormatterServices.GetUninitializedObject(Type) Method (System.Runtime.Serialization)](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatterservices.getuninitializedobject?WT.mc_id=WD-MVP-5003260)

补充：

在 dotnet 运行时里面，高版本的 dotnet 将可以使用 RuntimeHelpers 的 [GetUninitializedObject](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.compilerservices.runtimehelpers.getuninitializedobject?WT.mc_id=WD-MVP-5003260) 方法代替，因为在高版本的 dotnet 里面，对 [FormatterServices.GetUninitializedObject](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatterservices.getuninitializedobject?WT.mc_id=WD-MVP-5003260) 的实现如下

```csharp
        public static object GetUninitializedObject(
            // This API doesn't call any constructors, but the type needs to be seen as constructed.
            // A type is seen as constructed if a constructor is kept.
            // This obviously won't cover a type with no constructor. Reference types with no
            // constructor are an academic problem. Valuetypes with no constructors are a problem,
            // but IL Linker currently treats them as always implicitly boxed.
            [DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicConstructors | DynamicallyAccessedMemberTypes.NonPublicConstructors)]
            Type type) => RuntimeHelpers.GetUninitializedObject(type);
```

也就是说 调用 RuntimeHelpers 的 [GetUninitializedObject](https://docs.microsoft.com/zh-cn/dotnet/api/system.runtime.compilerservices.runtimehelpers.getuninitializedobject?WT.mc_id=WD-MVP-5003260) 方法和调用 [FormatterServices.GetUninitializedObject](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.serialization.formatterservices.getuninitializedobject?WT.mc_id=WD-MVP-5003260) 在逻辑上是等价的





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。