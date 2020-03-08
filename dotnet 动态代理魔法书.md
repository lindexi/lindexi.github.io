# dotnet 动态代理魔法书

看到标题的小伙伴是不是想知道什么是魔法书，如果你需要写一段代码，这段代码是在做神奇的业务，只有你查询到了魔法书你才能找到这个对象，同时你还需要实现自己的接口，通过自己实现的接口调用才能用到有趣的方法

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


在 C# 里面是不能直接让两个有相同方法的但没有继承的接口直接转换，但是通过透明代理和反射可以做到在不同的程序集定义的两个接口，这两个接口有相同的方法，那么将可以用另一个程序集的接口调用到传入程序集里面的接口

在本文开始之前，期望的读者是了解透明代理的，通过透明代理可以让每个调用方法之前先调用透明代理的方法。在透明代理的方法可以知道用户调用的是哪个方法，同时传入的参数是什么

先封装一个类，在这个类里面放一些定义好的实例，在透明代理方法里面通过传入用户调用的方法，使用反射调用对应实例的方法，然后将方法返回值返回

每次用户可以通过一个 Guid 才能获取透明代理，因为用户获取的是透明代理，是无法直接拿到对应的实例里面的字段，同时需要用户传入自己定义的接口，不想让用户访问的内容，用户是很难猜到的

这样的写法其实就和 COM 一样，从文档找到 COM 的 GUID 然后自己定义一个接口获取

先看我定义的使用方法

```csharp
            // 在第一个程序集注入了 F1 代码

            var guid = new Guid("{97C70651-EE85-4AED-9E2F-AD73AF34CF5D}");

            DynamicProxy.Add(guid,new F1());

            // 自己实现两个接口
            var f2 = DynamicProxy.GetObject<IF2>(guid);
            Console.WriteLine(f2.GetName());

            var f3 = DynamicProxy.GetObject<IF3>(guid);
            Console.WriteLine(f3.GetName());
```

期望的写法是调用 DynamicProxy.Add 在一个程序集，获取变量在另一个程序集

这里的 F1 和接口 IF2 IF3 都没有继承关系

```csharp
    interface IF3
    {
        string GetName();
    }

    interface IF2
    {
        string GetName();
    }

    class F1
    {
        public string GetName()
        {
            return "林德熙是逗比";
        }
    }
```

只要两个接口之间有定义相同的，那么这两个接口之间就可以相互转换，也就是在用户自己写的代码，是可以做到从文档里面找到其中需要使用的几个方法，然后定义自己的接口，通过上面方法就可以拿到

这个方法每次返回的对象都不相同，用户也不知道是不相同的对象，例如我可以将相同的一个对象作为两个不同的 Guid 传入，然后告诉用户两个不同的接口

```csharp
            var guid1 = new Guid("{97C70651-EE85-4AED-9E2F-AD73AF34CF5D}");
            var guid2 = new Guid("{05D1936F-7121-43BA-B986-A42A56555AAE}");

            var f1 = new F1();
            DynamicProxy.Add(guid1, f1);
            DynamicProxy.Add(guid2, f1);

            // 自己实现两个接口
            var f2 = DynamicProxy.GetObject<IF2>(guid1);
            Console.WriteLine(f2.GetName());
            Console.WriteLine(f2.GetName(new F3()));

            var f3 = DynamicProxy.GetObject<IF3>(guid2);
            Console.WriteLine(f3.GetName());
```

如果是在不同的业务上，估计小伙伴很难知道其实使用的 f2 和 f3 是相同一个类

这是我从 VisualStudio 学到的黑科技，通过这个方式定义，可以隐藏很多用户不需要了解的，同时可以随意的变动逻辑，只需要保持存在对应的接口就可以

例如现在我通过 F1 传入，但是我发现其中的某几个 Guid 获取的对应方法我需要修改了，于是我可以再创建一个 F2 的类，修改原因的 Guid 的值。还是使用上面的代码，我发现了通过 guid2 获取到的 IF3 的 GetName 方法需要修改。但是我不能影响到 IF2 的逻辑，于是我可以在传入的时候修改代码

```diff
- DynamicProxy.Add(guid2, f1);
+ DynamicProxy.Add(guid2, new F2());
```

这样就可以做到原有的 IF2 的逻辑不变，同时修改了另一个接口的方法

这里 DynamicProxy 的代码其实很简单，只是重写 Invoke 方法，从方法里面拿到用户调用的方法，通过反射调用实例的方法

```csharp
        class Express
        {
            /// <inheritdoc />
            public Express(Lazy<object> instance)
            {
                _instance = instance;
            }

            public object Instance => _instance.Value;

            private readonly Lazy<object> _instance;
        }

        class Proxy : RealProxy
        {
            /// <inheritdoc />
            public Proxy(Type classToProxy, Express express) : base(classToProxy)
            {
                Express = express;
            }

            public Express Express { get; }

            /// <inheritdoc />
            public override IMessage Invoke(IMessage msg)
            {
                MethodCallMessageWrapper callMessageWrapper = new MethodCallMessageWrapper((IMethodCallMessage) msg);
                MethodInfo methodBase = callMessageWrapper.MethodBase as MethodInfo;
                if (methodBase == null)
                    return null;

                var instance = Express.Instance;
                var type = instance.GetType();

                Type[] argumentTypeList;
                if (callMessageWrapper.Args?.Any() is true)
                {
                    argumentTypeList = callMessageWrapper.Args.Select(temp => temp.GetType()).ToArray();
                }
                else
                {
                    argumentTypeList = Type.EmptyTypes;
                }

                var method = type.GetMethod(methodBase.Name, argumentTypeList);

                if (method == null)
                {
                    throw new ArgumentException("调用方法不匹配，找不到" + methodBase + "方法");
                }

                return new ReturnMessage(
                    method.Invoke(instance, callMessageWrapper.Args),
                    callMessageWrapper.Args, callMessageWrapper.ArgCount, callMessageWrapper.LogicalCallContext,
                    callMessageWrapper);
            }
        }
```

创建 DynamicProxy 类，这个类就是给用户用的，当然实际的命名将会修改

```csharp
    class DynamicProxy
    {
        public static void Add(Guid guid, object instance)
        {
            _dictionary[guid] = new Express(new Lazy<object>(() => instance));
        }

        public static T GetObject<T>(Guid guid)
        {
            if (!typeof(T).IsInterface)
            {
                throw new ArgumentException();
            }

            return (T) new Proxy(typeof(T), _dictionary[guid]).GetTransparentProxy();
        }

        // 实际代码请使用缓存池
        private static Dictionary<Guid, Express> _dictionary = new Dictionary<Guid, Express>();

        class Express
        {
        	// 忽略代码
        }

        class Proxy : RealProxy
        {
        	// 忽略代码
        }

    }
```

在实际使用上面代码还需要做很多更改，例如支持 Express 里面添加多个实例，也就是给用户一个 Guid 和接口调用的方法，不同的方法实际可以从多个类里面调用

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
