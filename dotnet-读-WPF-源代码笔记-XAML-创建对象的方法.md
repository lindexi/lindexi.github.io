
# dotnet 读 WPF 源代码笔记 XAML 创建对象的方法

在 WPF 中，在 XAML 里面定义的对象的创建，实际上不是完全通过反射来进行创建的，在WPF框架里面，有进行了一系列的优化

<!--more-->


<!-- CreateTime:2021/1/18 19:40:02 -->


<!-- 标签：WPF，WPF源代码 -->
<!-- 发布 -->

在 WPF 中，将会通过 XamlTypeInvoker 的 CreateInstance 方法来进行对象的创建，而默认的 XamlTypeInvoker 的 CreateInstance 定义如下

```csharp
        public virtual object CreateInstance(object[] arguments)
        {
            ThrowIfUnknown();
            if (!_xamlType.UnderlyingType.IsValueType && (arguments == null || arguments.Length == 0))
            {
                object result = DefaultCtorXamlActivator.CreateInstance(this);
                if (result != null)
                {
                    return result;
                }
            }
            return CreateInstanceWithActivator(_xamlType.UnderlyingType, arguments);
        }

        private object CreateInstanceWithActivator(Type type, object[] arguments)
        {
            return SafeReflectionInvoker.CreateInstance(type, arguments);
        }
```

也就是说将调用 SafeReflectionInvoker.CreateInstance 进行对象的创建，这里的创建方式就是通过反射，如下面代码

```csharp
    static class SafeReflectionInvoker
    {
        internal static object CreateInstance(Type type, object[] arguments)
        {
            return Activator.CreateInstance(type, arguments);
        }
    }
```

从 [.NET/C# 反射的的性能数据，以及高性能开发建议（反射获取 Attribute 和反射调用方法） - walterlv](https://blog.walterlv.com/post/dotnet-high-performance-reflection-suggestions.html ) 和 [C# 直接创建多个类和使用反射创建类的性能](https://blog.lindexi.com/post/C-%E7%9B%B4%E6%8E%A5%E5%88%9B%E5%BB%BA%E5%A4%9A%E4%B8%AA%E7%B1%BB%E5%92%8C%E4%BD%BF%E7%94%A8%E5%8F%8D%E5%B0%84%E5%88%9B%E5%BB%BA%E7%B1%BB%E7%9A%84%E6%80%A7%E8%83%BD.html ) 可以了解，使用反射创建和对象创建性能相差大概有 30 倍

如果 WPF 真的全部使用反射进行创建，那么整体性能将会很低

从 XamlTypeInvoker 的 CreateInstance 方法的定义可以看到，这是一个可以被重写的方法，也就是说上面的代码只是默认的实现而已。在 WPF 中的一个重写方法是 WpfKnownTypeInvoker 类，这里面的定义如下

```csharp
    class WpfKnownTypeInvoker : XamlTypeInvoker
    {
        WpfKnownType _type;

        public override object CreateInstance(object[] arguments)
        {
            if ((arguments == null || arguments.Length == 0) && _type.DefaultConstructor != null)
            {
                return _type.DefaultConstructor.Invoke();
            }
            else
            {
                return base.CreateInstance(arguments);
            }
        }
    }
```

也就是将会尝试调用 WpfKnownType 的 DefaultConstructor 方法，这里的定义如下

```csharp
    class WpfKnownType : WpfXamlType, ICustomAttributeProvider
    {
        public Func<object> DefaultConstructor
        {
            get { return _defaultConstructor; }
            set
            {
                CheckFrozen();
                _defaultConstructor = value;
            }
        }
    }
```

也就是说其实这里面是委托创建，性能将会比反射的运行效率大概高几十倍的速度

这里的委托是在 WpfSharedBamlSchemaContext 类里面定义的，这里面的内容大概如下

```csharp
        [System.Runtime.CompilerServices.MethodImpl(System.Runtime.CompilerServices.MethodImplOptions.NoInlining)]
        private WpfKnownType Create_BamlType_TextBlock(bool isBamlType, bool useV3Rules)
        {
            var bamlType = new WpfKnownType(this, // SchemaContext
                                              638, "TextBlock",
                                              typeof(System.Windows.Controls.TextBlock),
                                              isBamlType, useV3Rules);
            bamlType.DefaultConstructor = delegate() { return new System.Windows.Controls.TextBlock(); };
            bamlType.ContentPropertyName = "Inlines";
            bamlType.RuntimeNamePropertyName = "Name";
            bamlType.XmlLangPropertyName = "Language";
            bamlType.UidPropertyName = "Uid";
            bamlType.IsUsableDuringInit = true;
            bamlType.Freeze();
            return bamlType;
        }
```

可以看到对于 WPF 框架里面了解的对象，都将会创建委托的方式提升性能

这个类超过了一万行，可以看到这里用了很大的逻辑来提升 XAML 对象创建的性能

那如果是 WPF 不认识的类呢？如我自己定义的类型，那么将会进入 XamlTypeInvoker 的 CreateInstance 方法的 DefaultCtorXamlActivator 类，在这个类里面的逻辑如下

```csharp
        private static class DefaultCtorXamlActivator
        {
            public static object CreateInstance(XamlTypeInvoker type)
            {
                if (!EnsureConstructorDelegate(type))
                {
                    return null;
                }
                object inst = CallCtorDelegate(type);
                return inst;
            }

            // 忽略代码
        }
```

在 EnsureConstructorDelegate 方法里面将会判断如果对象是公开的，那么尝试获取默认构造函数，将默认构造函数做成委托。此时的性能将会是类型第一次进入的时候的速度比较慢，但是后续进入的时候就能使用委托创建，此时性能将会比较好。通过反射创建委托提升性能的方法，详细请看 [.NET Core/Framework 创建委托以大幅度提高反射调用的性能 - walterlv](https://blog.walterlv.com/post/create-delegate-to-improve-reflection-performance.html )

这里的 EnsureConstructorDelegate 方法相对复杂，我删减了一些代码，让逻辑相对清晰。详细的代码还请到 WPF 官方仓库获取

```csharp
            private static bool EnsureConstructorDelegate(XamlTypeInvoker type)
            {
            	// 如果类型初始化过构造函数创建，那么返回，这是缓存的方法
                if (type._constructorDelegate != null)
                {
                    return true;
                }

                // 如果不是公开的方法，那么将无法使用反射创建委托的科技
                if (!type.IsPublic)
                {
                    return false;
                }

                // 反射获取对象的构造函数
                Type underlyingType = type._xamlType.UnderlyingType.UnderlyingSystemType;
                // Look up public ctors only, for equivalence with Activator.CreateInstance
                ConstructorInfo tConstInfo = underlyingType.GetConstructor(Type.EmptyTypes);
                IntPtr constPtr = tConstInfo.MethodHandle.GetFunctionPointer();
               
                // 反射创建委托，这样下次访问就不需要使用反射，可以提升性能
                // This requires Reflection Permission
                Action<object> ctorDelegate = ctorDelegate =
                    (Action<object>)s_actionCtor.Invoke(new object[] { null, constPtr });
                type._constructorDelegate = ctorDelegate;
                return true;
            }
```

也就是说只有第一次的类型进入才会调用反射创建委托用来提升性能，之后的进入将会使用第一次创建出来的委托来创建对象，这样能提升性能

从上面代码可以看到，如果对象不是公开的，那么将因为 .NET 的限制，不能使用反射创建委托的方法来提升性能。因此一个性能提升的建议是在 XAML 里面使用的类尽量都是公开的，这样能提升一些性能

在获取到了构造函数的对应的委托之后，就能调用 CallCtorDelegate 方法来创建对象，此时的创建对象速度会比反射快很多

但是如果对象的类不是公开的，那么将需要用到 CreateInstanceWithActivator 使用反射创建对象，此时的性能相对来说比较差

因此在 WPF 的 XAML 创建对象，只有在尝试了判断这是 WPF 已知的对象失败之后，同时对象对应的类不是公开的不能使用反射创建委托的科技，才会使用反射创建对象。大多数的时候，使用 XAML 都不会有很多性能损失

而对于自己定义的非公开的类，我给 WPF 官方提一个建议，就是提供让开发端自己注入创建器的方式，用来提升性能，请看 [API Request: Allow developers to inject a XAML factory for creating objects · Issue #4022 · dotnet/wpf](https://github.com/dotnet/wpf/issues/4022 )

当前的 WPF 在 [https://github.com/dotnet/wpf](https://github.com/dotnet/wpf) 完全开源，使用友好的 MIT 协议，意味着允许任何人任何组织和企业任意处置，包括使用，复制，修改，合并，发表，分发，再授权，或者销售。在仓库里面包含了完全的构建逻辑，只需要本地的网络足够好（因为需要下载一堆构建工具），即可进行本地构建





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。