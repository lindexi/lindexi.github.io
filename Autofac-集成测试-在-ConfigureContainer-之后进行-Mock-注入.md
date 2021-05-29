
# Autofac 集成测试 在 ConfigureContainer 之后进行 Mock 注入

在使用 Autofac 框架进行开发后，编写集成测试时，需要用 Mock 的用于测试的模拟的类型去代替容器里面已注入的实际类型，也就需要在 Autofac 完全收集完成之后，再次注入模拟的对象进行覆盖原有业务代码注册的正式对象。但 Autofac 默认没有提供此机制，我阅读了 Autofac 的源代码之后，创建了一些辅助代码，实现了此功能。本文将告诉大家如何在集成测试里面，在使用了 Autofac 的项目里面，在所有收集完成之后，注入用于测试的 Mock 类型，和 Autofac 接入的原理

<!--more-->


<!-- 发布 -->

## 背景

为什么选择使用 Autofac 框架？原因是在此前的 WPF 项目里面，有使用过的是 MEF 和 Autofac 两个框架，而 MEF 的性能比较糟心。解决 MEF 性能问题的是 VS-MEF 框架。在后续开发的一个 ASP.NET Core 项目里面，也就自然选用了 Autofac 框架

对比原生的 ASP.NET Core 自带的 DI 框架，使用 Autofac 的优势在于支持模块化的初始化，支持属性注入

默认的 Autofac 可以通过 `Autofac.Extensions.DependencyInjection` 将 Autofac 和 dotnet 通用依赖注入框架合入在一起，但在 Autofac 里面的定制要求是在 Startup 的 ConfigureContainer 函数里面进行依赖注入，也就是在默认的 ASP.NET Core 里面没有提供更靠后的依赖注入方法，可以在完成收集之后，再次注入测试所需要的类型，覆盖业务代码里面的实际对象

## 需求

假定在一个应用，如 ASP.NET Core 应用里面，进行集成测试，想要在集成测试里面，使用项目里面的依赖注入关系，只是将部分类型替换为测试项目里面的模拟的类型

而在应用里面，实际的业务类型是在 Autofac 的 Module 进行注入的。在应用里面的大概逻辑如下，在 Program 的 CreateHostBuilder 函数里面通过 UseServiceProviderFactory 方法使用 Autofac 替换 原生 的框架

```csharp
        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                })
                // 使用 auto fac 代替默认的 IOC 容器 
                .UseServiceProviderFactory(new AutofacServiceProviderFactory());
```

在 Startup 里面添加 ConfigureContainer 方法，代码如下

```csharp
        public void ConfigureContainer(ContainerBuilder builder)
        {
            
        }
```

在 ConfigureContainer 里面注入具体的需要初始化的业务模块，例如 FooModule 模块。在具体模块里面注入实际业务的类型，如 Foo 类型，代码如下

```csharp
    public class Startup
    {
    	// 忽略代码

        public void ConfigureContainer(ContainerBuilder builder)
        {
            builder.RegisterModule(new FooModule());
        }
    }

    class FooModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            Console.WriteLine($"06 FooModule");

            builder.RegisterType<Foo>().As<IFoo>();
        }
    }

    public class Foo : IFoo
    {
    }

    public interface IFoo
    {
    }
```

现在的需求是在集成测试里面，将 IFoo 的实际类型从 Foo 替换为 TestFoo 类型

在集成测试项目里面，可以使用如下代码获取实际的项目的依赖注入收集

```csharp
                var hostBuilder = Host.CreateDefaultBuilder()
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>();
                        webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                    })
                    // 使用 auto fac 代替默认的 IOC 容器 
                    .UseServiceProviderFactory(new AutofacServiceProviderFactory());

                var host = hostBuilder.Build();

                var foo = host.Services.GetService<IFoo>();
```

以上的 foo 就是从收集的容器里面获取的 IFoo 对象，以上代码获取到的是业务代码的 Foo 类型对象。假定需要让容器里面的 IFoo 的实际类型作为测试的 TestFoo 类型，就需要在实际项目的依赖注入收集完成之前，进行测试的注入

但实际上没有在 Autofac 里面找到任何的辅助方法可以用来实现此功能。如果是默认的应用框架，可以在 ConfigureWebHostDefaults 函数之后，通过 ConfigureServices 函数覆盖在 Startup 的 ConfigureServices 函数注入的类型

## 实现方法

实现的方法是很简单的，关于此实现为什么能解决问题还请参阅下文的原理部分

集成测试项目不需要改动原有的业务项目即可完成测试，实现方法是在集成测试项目里面添加 FakeAutofacServiceProviderFactory 用来替换 Autofac 的 AutofacServiceProviderFactory 类型，代码如下

```csharp
    class FakeAutofacServiceProviderFactory : IServiceProviderFactory<ContainerBuilder>
    {
        public FakeAutofacServiceProviderFactory(
            ContainerBuildOptions containerBuildOptions = ContainerBuildOptions.None,
            Action<ContainerBuilder>? configurationActionOnBefore = null,
            Action<ContainerBuilder>? configurationActionOnAfter = null)
        {
            _configurationActionOnAfter = configurationActionOnAfter;
            AutofacServiceProviderFactory =
                new AutofacServiceProviderFactory(containerBuildOptions, configurationActionOnBefore);
        }

        private AutofacServiceProviderFactory AutofacServiceProviderFactory { get; }
        private readonly Action<ContainerBuilder>? _configurationActionOnAfter;

        public ContainerBuilder CreateBuilder(IServiceCollection services)
        {
            return AutofacServiceProviderFactory.CreateBuilder(services);
        }

        public IServiceProvider CreateServiceProvider(ContainerBuilder containerBuilder)
        {
            _configurationActionOnAfter?.Invoke(containerBuilder);
            return AutofacServiceProviderFactory.CreateServiceProvider(containerBuilder);
        }
    }
```

可以看到本质的 FakeAutofacServiceProviderFactory 实现就是通过 AutofacServiceProviderFactory 的属性实现，只是在 CreateServiceProvider 方法里面加入了委托，可以方便在单元测试里面进行注入自己的方法

在集成测试项目里面的使用方法如下

```csharp
                var hostBuilder = Host.CreateDefaultBuilder()
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>();
                        webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                    })
                    // 使用 auto fac 代替默认的 IOC 容器 
                    .UseServiceProviderFactory(new FakeAutofacServiceProviderFactory(configurationActionOnAfter:
                        builder =>
                        {
                            builder.RegisterModule<TestModule>();
                        }));
```

传入的委托需要注入测试的初始化模块，也就是 TestModule 需要加入注入，通过上面代码，可以让 TestModule 在依赖注入的最后进行初始化。在 TestModule 里面加入实际的测试类型注入的代码

```csharp
    class TestModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            builder.RegisterType<TestFoo>().As<IFoo>();
        }
    }

    class TestFoo : IFoo
    {
    }
```

通过上面方法就可以让集成测试项目从容器里面获取 IFoo 的对象，拿到的是 TestFoo 类型，集成测试项目的代码如下

```csharp
    [TestClass]
    public class FooTest
    {
        [ContractTestCase]
        public void Test()
        {
            "依赖注入的时机，可以在完成收集之后，覆盖原有的类型".Test(() =>
            {
                var hostBuilder = Host.CreateDefaultBuilder()
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>();
                        webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                    })
                    // 使用 auto fac 代替默认的 IOC 容器 
                    .UseServiceProviderFactory(new FakeAutofacServiceProviderFactory(configurationActionOnAfter:
                        builder =>
                        {
                            builder.RegisterModule<TestModule>();
                        }));

                var host = hostBuilder.Build();

                var foo = host.Services.GetService<IFoo>();
                Assert.IsInstanceOfType(foo, typeof(TestFoo));
            });
        }
    }

    class TestModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            builder.RegisterType<TestFoo>().As<IFoo>();
        }
    }

    class TestFoo : IFoo
    {
    }

    class FakeAutofacServiceProviderFactory : IServiceProviderFactory<ContainerBuilder>
    {
        public FakeAutofacServiceProviderFactory(
            ContainerBuildOptions containerBuildOptions = ContainerBuildOptions.None,
            Action<ContainerBuilder>? configurationActionOnBefore = null,
            Action<ContainerBuilder>? configurationActionOnAfter = null)
        {
            _configurationActionOnAfter = configurationActionOnAfter;
            AutofacServiceProviderFactory =
                new AutofacServiceProviderFactory(containerBuildOptions, configurationActionOnBefore);
        }

        private AutofacServiceProviderFactory AutofacServiceProviderFactory { get; }
        private readonly Action<ContainerBuilder>? _configurationActionOnAfter;

        public ContainerBuilder CreateBuilder(IServiceCollection services)
        {
            return AutofacServiceProviderFactory.CreateBuilder(services);
        }

        public IServiceProvider CreateServiceProvider(ContainerBuilder containerBuilder)
        {
            _configurationActionOnAfter?.Invoke(containerBuilder);
            return AutofacServiceProviderFactory.CreateServiceProvider(containerBuilder);
        }
    }
```

以上集成测试使用了 CUnit 中文单元测试框架辅助，在上面代码里面，可以看到集成测试里面的容器拿到的 IFoo 对象就是 TestFoo 类型。通过这个方法就可以在业务代码执行过程，注入测试需要的类型

为什么通过以上的代码即可实现此功能，为什么需要自己实现一个 FakeAutofacServiceProviderFactory 类型，为什么不能在 AutofacServiceProviderFactory.CreateServiceProvider 方法之前注入类型，而是需要再定义一个 TestModule 模块，在测试初始化模块进行初始化。更多细节请看下文

## 原理

回答以上问题，需要了解各个注入方法调用的顺序，我在代码里面通过控制台输出各个方法的顺序。标记了顺序的代码放在本文最后

以下是按照调用顺序的方法代码

### Startup 的 ConfigureServices 方法

```csharp
    public class Startup
    {
        // 忽略代码

        // This method gets called by the runtime. Use this method to add services to the container.
        // For more information on how to configure your application, visit https://go.microsoft.com/fwlink/?LinkID=398940
        public void ConfigureServices(IServiceCollection services)
        {
            Console.WriteLine($"01 ConfigureServices");
        }
    }
```

在 Startup 的 ConfigureServices 是依赖注入中最开始调用的方法，这也是原生的框架自带的方法

### IHostBuilder 的 ConfigureServices 扩展方法

```csharp
                var hostBuilder = Host.CreateDefaultBuilder()
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>();
                        webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                    })
                    .ConfigureServices(collection => Console.WriteLine($"02 ConfigureServices Delegate"))
```

在 IHostBuilder 的 ConfigureServices 扩展方法将会在 Startup 的 ConfigureServices 方法执行完成之后调用，因此如果只使用原生的依赖注入，可以在此方法进行覆盖，也就是如果没有使用 Autofac 框架，只使用原生的框架，可以在集成测试，在此方法注入测试的类型

### Startup 的 ConfigureContainer 方法

```csharp
    public class Startup
    {
        // 忽略代码

        public void ConfigureContainer(ContainerBuilder builder)
        {
            Console.WriteLine($"03 ConfigureContainer");
            builder.RegisterModule(new FooModule());
        }
    }
```

可以看到 `public void ConfigureContainer(ContainerBuilder builder)` 方法的调用在 ConfigureServices 方法之后，在 Autofac 也通过此机制实现代替原生的依赖注入功能，也通过此方法从原生的注入获取依赖

关于 Autofac 的实际逻辑，请参阅下文

### FakeAutofacServiceProviderFactory 的 CreateServiceProvider 方法

在如上代码，咱编写了 FakeAutofacServiceProviderFactory 用于替换 AutofacServiceProviderFactory 类型。在 FakeAutofacServiceProviderFactory 的 CreateServiceProvider 方法将会在调用 ConfigureContainer 之后执行

```csharp
    class FakeAutofacServiceProviderFactory : IServiceProviderFactory<ContainerBuilder>
    {
        public FakeAutofacServiceProviderFactory(
            ContainerBuildOptions containerBuildOptions = ContainerBuildOptions.None,
            Action<ContainerBuilder>? configurationActionOnBefore = null,
            Action<ContainerBuilder>? configurationActionOnAfter = null)
        {
            _configurationActionOnAfter = configurationActionOnAfter;
            AutofacServiceProviderFactory =
                new AutofacServiceProviderFactory(containerBuildOptions, configurationActionOnBefore);
        }

        private AutofacServiceProviderFactory AutofacServiceProviderFactory { get; }
        private readonly Action<ContainerBuilder>? _configurationActionOnAfter;

        public ContainerBuilder CreateBuilder(IServiceCollection services)
        {
            return AutofacServiceProviderFactory.CreateBuilder(services);
        }

        public IServiceProvider CreateServiceProvider(ContainerBuilder containerBuilder)
        {
            Console.WriteLine($"04 FakeAutofacServiceProviderFactory");
            _configurationActionOnAfter?.Invoke(containerBuilder);
            return AutofacServiceProviderFactory.CreateServiceProvider(containerBuilder);
        }
    }
```


在以上的 CreateServiceProvider 方法将会在 Startup 的 ConfigureContainer 方法之后执行，如上面代码。按照上面代码，将会执行 `_configurationActionOnAfter` 委托

因此下一个执行的就是传入的委托

```csharp
                IHostBuilder? hostBuilder = Host.CreateDefaultBuilder()
                    .ConfigureWebHostDefaults(webBuilder =>
                    {
                        webBuilder.UseStartup<Startup>();
                        webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                    })
                    .ConfigureServices(collection => Console.WriteLine($"02 ConfigureServices Delegate"))
                    // 使用 auto fac 代替默认的 IOC 容器 
                    .UseServiceProviderFactory(new FakeAutofacServiceProviderFactory(configurationActionOnAfter:
                        builder =>
                        {
                            Console.WriteLine($"05 ConfigurationActionOnAfter");
                            builder.RegisterModule<TestModule>();
                        }));
```

也就是如上代码的 ConfigurationActionOnAfter 委托

但尽管 FakeAutofacServiceProviderFactory 的 CreateServiceProvider 在 Startup 的 ConfigureContainer 方法之后执行，实际上很多开发者不会在 Startup 的 ConfigureContainer 方法完成注册，而是在 ConfigureContainer 里面初始化模块。如上面代码，在业务逻辑注册的模块的初始化还没被调用。只有在实际的 ContainerBuilder 调用 Build 方法，才会执行模块的 Load 方法

因此下一个调用就是业务逻辑注册的模块

### FooModule 的 Load 方法

按照 Autofac 的定义，在 ConfigureContainer 的 Build 方法里面，才会执行模块的初始化，调用 Load 方法

```csharp
    class FooModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            Console.WriteLine($"06 FooModule");

            builder.RegisterType<Foo>().As<IFoo>();
        }
    }
```

在 Autofac 里面，将会按照模块注册的顺序，调用模块的 Load 方法，如上面代码，可以看到 TestModule 在最后被注册，因此将会最后执行

### TestModule 的 Load 方法

在上面代码 TestModule 是最后注册到 Autofac 的模块，也就是 TestModule 将会最后被执行

```csharp
    class TestModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            Console.WriteLine($"07 TestModule");

            builder.RegisterType<TestFoo>().As<IFoo>();
        }
    }
```

如上面代码，在 TestModule 注入的测试类型，将会替换业务代码的实际类型

### Autofac 接入的方法

通过上面的方法调用顺序，大家也可以了解为什么集成测试的代码这样写就有效果。更深入的逻辑是 Autofac 的设计，为什么可以让 Autofac 框架可以接入到 ASP.NET Core 应用里面，我在此前可一直都是在 WPF 框架使用的。这个问题其实很简单，所有的 dotnet 项目，无论是 ASP.NET Core 还是 WPF 等，都是相同的 dotnet 逻辑，装配方式都相同，只是顶层业务逻辑实现方法有所不同，因此只需要加一点适配逻辑就能通用

从上面项目安装的 NuGet 包可以看到，安装了 `Autofac.Extensions.DependencyInjection` 库就是提供 Autofac 与 dotnet 通用依赖注入框架链接的功能，而 ASP.NET Core 原生的框架就是基于 dotnet 通用依赖注入框架，因此就能将 Autofac 接入到 ASP.NET Core 应用

在 UseServiceProviderFactory 方法里面，将会执行 ASP.NET Core 框架的依赖注入容器相关方法，此方法注入的 IServiceProviderFactory 带泛形的类型，将可以支持在 Startup 方法里面添加 ConfigureContainer 方法，参数就是 IServiceProviderFactory 的泛形

如加入了 FakeAutofacServiceProviderFactory 类型，此类型继承了 `IServiceProviderFactory<ContainerBuilder>` 接口，也就是 IServiceProviderFactory 的 泛形 是 ContainerBuilder 类型，因此可以在 Startup 的 ConfigureContainer 方法参数就是 ContainerBuilder 类型

```csharp
    public class Startup
    {
        // 忽略代码

        public void ConfigureContainer(ContainerBuilder builder)
        {
          
        }
    }
```

而 ConfigureContainer 将会被 Microsoft.AspNetCore.Hosting.GenericWebHostBuilder 进行调用，在 GenericWebHostBuilder 的调用顺序是先调用 ConfigureServices 再调用 对应的 ConfigureContainer 方法

在  	Microsoft.Extensions.Hosting.HostBuilder.CreateServiceProvider 方法就是实际创建容器的方法，这个方法里面，将会先调用完成 ConfigureServices 的配置，然后再调用 ConfigureContainer 的配置，代码如下

```csharp
    public class HostBuilder : IHostBuilder
    {
        private void CreateServiceProvider()
        {
           // 忽略代码
            var services = new ServiceCollection();

            foreach (Action<HostBuilderContext, IServiceCollection> configureServicesAction in _configureServicesActions)
            {
                configureServicesAction(_hostBuilderContext, services);
            }

            object containerBuilder = _serviceProviderFactory.CreateBuilder(services);

            foreach (IConfigureContainerAdapter containerAction in _configureContainerActions)
            {
                containerAction.ConfigureContainer(_hostBuilderContext, containerBuilder);
            }

            _appServices = _serviceProviderFactory.CreateServiceProvider(containerBuilder);
        }
    }
```

此时的 `_serviceProviderFactory` 将会是注入的 FakeAutofacServiceProviderFactory 类型，将会调用对应的 CreateBuilder 方法，也就是如下代码将会调用


```csharp
    class FakeAutofacServiceProviderFactory : IServiceProviderFactory<ContainerBuilder>
    {
        // 忽略代码
        public ContainerBuilder CreateBuilder(IServiceCollection services)
        {
            return AutofacServiceProviderFactory.CreateBuilder(services);
        }
    }
```

在 HostBuilder 的 `_configureContainerActions` 委托调用 ConfigureContainer 的逻辑，实际就是 Startup 类型里面定义的 ConfigureContainer 方法

因此就是先调用 Startup 类型和 IHostBuilder 的 ConfigureServices 方法，然后再调用 ConfigureContainer 方法

在 Autofac 的 AutofacServiceProviderFactory 在 CreateBuilder 方法就可以拿到了原生注册的所有类型，因为在调用 CreateBuilder 之前已经完成了所有的原生逻辑

在 AutofacServiceProviderFactory 的 CreateBuilder 方法将会先创建 ContainerBuilder 对象，然后调用 Populate 方法，从原生的 IServiceCollection 获取注册的类型，重新放到 ContainerBuilder 容器

```csharp
        public ContainerBuilder CreateBuilder(IServiceCollection services)
        {
            var builder = new ContainerBuilder();

            builder.Populate(services);

            _configurationAction(builder);

            return builder;
        }
```

上面代码的 ContainerBuilder 是 Autofac 框架的，而 Populate 是扩展方法，和 AutofacServiceProviderFactory 都是在 `Autofac.Extensions.DependencyInjection` 库提供的，通过此扩展方法和 AutofacServiceProviderFactory 即可实现 Autofac 和 dotnet 原生接入。在 Populate 方法从 dotnet 原生拿到注册的类型，放入到 Autofac 的 ContainerBuilder 里，这样所有之前使用 dotnet 原生注入的类型就可以在 Autofac 拿到

```csharp
        public static void Populate(
            this ContainerBuilder builder,
            IEnumerable<ServiceDescriptor> descriptors,
            object lifetimeScopeTagForSingletons = null)
        {
            if (descriptors == null)
            {
                throw new ArgumentNullException(nameof(descriptors));
            }

            builder.RegisterType<AutofacServiceProvider>().As<IServiceProvider>().ExternallyOwned();
            builder.RegisterType<AutofacServiceScopeFactory>().As<IServiceScopeFactory>();

            Register(builder, descriptors, lifetimeScopeTagForSingletons);
        }
```

以上的 `IEnumerable<ServiceDescriptor>` 就是 IServiceCollection 类型的对象，实际代码是 Register 里面拿到注册的类型，重新放入到 Autofac 里

```csharp
        private static void Register(
            ContainerBuilder builder,
            IEnumerable<ServiceDescriptor> descriptors,
            object lifetimeScopeTagForSingletons)
        {
            foreach (var descriptor in descriptors)
            {
                if (descriptor.ImplementationType != null)
                {
                    // Test if the an open generic type is being registered
                    var serviceTypeInfo = descriptor.ServiceType.GetTypeInfo();
                    if (serviceTypeInfo.IsGenericTypeDefinition)
                    {
                        builder
                            .RegisterGeneric(descriptor.ImplementationType)
                            .As(descriptor.ServiceType)
                            .ConfigureLifecycle(descriptor.Lifetime, lifetimeScopeTagForSingletons);
                    }
                    else
                    {
                        builder
                            .RegisterType(descriptor.ImplementationType)
                            .As(descriptor.ServiceType)
                            .ConfigureLifecycle(descriptor.Lifetime, lifetimeScopeTagForSingletons);
                    }
                }
                else if (descriptor.ImplementationFactory != null)
                {
                    var registration = RegistrationBuilder.ForDelegate(descriptor.ServiceType, (context, parameters) =>
                        {
                            var serviceProvider = context.Resolve<IServiceProvider>();
                            return descriptor.ImplementationFactory(serviceProvider);
                        })
                        .ConfigureLifecycle(descriptor.Lifetime, lifetimeScopeTagForSingletons)
                        .CreateRegistration();

                    builder.RegisterComponent(registration);
                }
                else
                {
                    builder
                        .RegisterInstance(descriptor.ImplementationInstance)
                        .As(descriptor.ServiceType)
                        .ConfigureLifecycle(descriptor.Lifetime, null);
                }
            }
        }
```

上面代码拿到的 ServiceDescriptor 就是在原生框架里面的注入类型的定义，可以看到这些都重新放到 Autofac 的容器里面

这就是为什么 Autofac 能拿到在 ASP.NET Core 框架里面其他框架注入的类型的代码

在 HostBuilder 的 CreateServiceProvider 方法最后就是调用 IServiceProviderFactory 的 CreateServiceProvider 方法返回实际的容器

也就是调用了 Autofac 的 CreateServiceProvider 方法，代码如下

```csharp
        public IServiceProvider CreateServiceProvider(ContainerBuilder containerBuilder)
        {
            if (containerBuilder == null) throw new ArgumentNullException(nameof(containerBuilder));

            var container = containerBuilder.Build(_containerBuildOptions);

            return new AutofacServiceProvider(container);
        }
```

可以看到本质就是调用了 ContainerBuilder 的 Build 方法，而在 Build 方法里面，才会初始化 Autofac 的模块。因此在 FakeAutofacServiceProviderFactory 的 CreateServiceProvider 方法里面添加的代码，是不会在具体业务模块的初始化模块调用之前被调用。但在 Autofac 里面，模块的初始化顺序是模块加入 Autofac 的顺序，因此可以在 FakeAutofacServiceProviderFactory 里面再加入测试的模块，测试的模块将会是最后加入的模块，也就是将会最后被执行

因此想要在接入 Autofac 框架覆盖业务逻辑注册的类型，就需要在 Autofac 里面注册一个测试使用的模块，要求这个模块最后注册，然后在此模块里面进行注册类型，这样就可以让测试模块注册的类型是最后注册的，覆盖原有的类型。而想要让测试模块最后注册，就需要自己实现一个继承 `IServiceProviderFactory<ContainerBuilder>` 的类型，才能在 AutofacServiceProviderFactory 的 CreateServiceProvider 方法调用之前注册模块

虽然我很喜欢使用 Autofac 框架，但是我觉得在接入 ASP.NET Core 时，没有很好加入测试的机制，而让开发者需要自己理解底层的逻辑才能进行注册测试的类型

这里也需要给 dotnet 的设计点赞，在一开始的 ASP.NET Core 选择依赖注入框架时，选择的是 dotnet 通用依赖注入框架，而 dotnet 通用依赖注入框架最底层的是使用最初的装配器接口，在 C# 语言里面接口的定义是最通用的，接口只约束而不定义。通过这一套传承的定义，可以让 10 多年前的 Autofac 框架依然可以跑在现代的应用里面

这 10 多年也不是 Autofac 啥都不做，上面的说法只是为了说明 dotnet 的兼容性特别强和最初的 dotnet 设计大佬的强大

本文的实现方法，虽然代码很少，但要理解 dotnet 的依赖注入和 ASP.NET Core 的依赖注入使用，和 Autofac 的接入方法。看起来就是 Autofac 的接入机制其实没有考虑全，当然，也许是我的技术不够，也许有更好的实现方法，还请大佬们教教我

## 代码

本文所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/741538bdb469e3e877c2664f8003a528245ba3d5/HihukekelralCayagaynofo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/741538bdb469e3e877c2664f8003a528245ba3d5/HihukekelralCayagaynofo) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。