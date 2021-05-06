# Autofac 通过 PreserveExistingDefaults 解决单元测试 Fake 对象被覆盖

在使用 Autofac 作为 IoC 容器，因为 Autofac 默认的创建时机是在主机运行时。而在此 Module 被 Load 时注入的对象的注入的时机，将会在单元测试 Fake 注入之后，这就意味着 Load 时注入的对象将会覆盖 Fake 的对象。可以通过调用 Autofac 的 PreserveExistingDefaults 方法解决覆盖的问题

<!--more-->

<!-- 发布 -->

在进行集成测试，需要注入一些 Fake 的或者 Mock 的等用来测试的对象，这些对象期望替换掉原有的业务逻辑的对象。而在使用 Autofac 框架，将因为对象创建时机的问题，而让单元测试不好玩

单元测试注入的顺序，是在业务对象注入之前，因此业务对象将会替换掉单元测试注入的对象

通过 PreserveExistingDefaults 方法，可以在框架判断，如果在此之前已有注册，那么将不再进行注册，代码如下

```csharp
            builder.RegisterType<Foo>().As<IFoo>()
                // 通过 PreserveExistingDefaults 可以在已经注册过了的应用，不会被覆盖为 Foo 类型
                // 在单元测试使用，单元测试注入了测试用的消费者，可以不被覆盖
                .PreserveExistingDefaults();
```

此时就可以在单元测试中，通过如下代码注入 FakeFoo 对象

```csharp
            Microsoft.Extensions.Hosting.Host.CreateDefaultBuilder()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                })
                // 使用 auto fac 代替默认的 IOC 容器 
                .UseServiceProviderFactory(new AutofacServiceProviderFactory(builder =>
                {
                    builder.RegisterModule(new FakeFooModule());
                }))

    class FakeFooModule : Autofac.Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            builder.RegisterType<FakeFoo>().As<FakeFoo>().As<IFoo>().SingleInstance();
        }
    }
```

上面代码就是尝试注入 FakeFoo 作为 IFoo 服务，在业务逻辑里面，将判断 IFoo 服务是否已注册，如果没有被注册，才注册为 Foo 对象

更多集成测试请看 [asp dotnet core 基于 TestServer 做集成测试](https://blog.lindexi.com/post/asp-dotnet-core-%E5%9F%BA%E4%BA%8E-TestServer-%E5%81%9A%E9%9B%86%E6%88%90%E6%B5%8B%E8%AF%95.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、 使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
