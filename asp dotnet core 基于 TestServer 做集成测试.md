# asp dotnet core 基于 TestServer 做集成测试

我有一个古老的 dotnet core 3.1 的 asp dotnet core 项目，现在我准备将他升级到 dotnet 5 了。但是我不想和博客园一样翻车，因此我需要做一点集成测试的辅助，尽管依然还是翻车了，但是我要学习博客园伟大的精神，将在这个项目里面所做的所有自动化测试项目的方法写下来

<!--more-->
<!-- CreateTime:2020/11/24 20:29:54 -->

<!-- 发布 -->

在开始从 dotnet core 3.1 升级到 dotnet 5 之前，我先开始准备集成测试。一开始准备的测试是开启主机，然后通过网络调用。然而这个方法一开启我就被拖出去了…… 因为开启主机会占用端口，而刚好我的几个项目都采用了相同的端口

而我开始尝试在配置文件里面指定随机的端口，而此时又有玄学的网络权限，但是我又不知道将谁拖出去

此时小伙伴给我安利了 TestServer 库，通过这个库可以不监听端口，全部都在内存中跑。当然了，访问外部服务就看注入了，没做注入也依然走网络的。只是自己的应用不会去监听端口而已

先新建一个项目，这是一个单元测试项目，用来做集成测试

在 dotnet 里面的套路就是先安装 NuGet 包，然后调用。安装的 NuGet 是 Microsoft.AspNetCore.TestHost 库。这个库一开始需要安装 3.1.10 的版本，在之后项目升级到 dotnet 5 才能使用最新的版本

```xml
        <PackageReference Include="Microsoft.AspNetCore.TestHost" Version="3.1.10" />
```

在我的单元测试项目里面全部安装的库如下

```xml
    <ItemGroup>
        <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.8.0" />
        <PackageReference Include="MSTest.TestAdapter" Version="2.1.1" />
        <PackageReference Include="MSTest.TestFramework" Version="2.1.1" />
        <PackageReference Include="coverlet.collector" Version="1.3.0" />
        <PackageReference Include="Moq" Version="4.15.1" />
        <PackageReference Include="MSTest.TestAdapter" Version="2.1.2" />
        <PackageReference Include="MSTest.TestFramework" Version="2.1.2" />
        <PackageReference Include="MSTestEnhancer" Version="2.0.1" />
        <PackageReference Include="System.Text.Encoding.CodePages" Version="5.0.0" />
        <PackageReference Include="Microsoft.AspNetCore.TestHost" Version="3.1.10" />
    </ItemGroup>
```

在使用 TestServer 进行集成测试的时候，其实就是将启动主机的逻辑替换掉，如 [ASP.NET Core搭建多层网站架构【12-xUnit单元测试之集成测试】 - kasnti - 博客园](https://www.cnblogs.com/kasnti/p/12246180.html ) 这篇博客所说的方法，咱来新建一个静态类，用来创建主机和运行

```csharp
    [TestClass]
    public static class TestHostBuild
    {
        public static HttpClient GetTestClient() => _host.GetTestClient();

        [AssemblyInitialize]
        public static async Task GlobalInitialize(TestContext testContext)
        {
            IHost host = await CreateAndRun();
            _host = host;
        }

        private static IHost _host;

        [AssemblyCleanup]
        public static void GlobalCleanup()
        {
            _host.Dispose();
        }

        private static Task<IHost> CreateAndRun() => CreateHostBuilder().StartAsync();

        public static IHostBuilder CreateHostBuilder() =>
            Host.CreateDefaultBuilder()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                    webBuilder.UseTestServer(); //关键是多了这一行建立TestServer
                })
                .ConfigureAppConfiguration((hostingContext, config) =>
                {
                	// 进行测试的配置
                    var appConfigurator = config.ToAppConfigurator();
                    // 这里使用了 https://github.com/dotnet-campus/dotnetCampus.Configurations 做配置
                    var apmConfiguration = appConfigurator.Of<ApmConfiguration>();
                    apmConfiguration.DisableApm = true;
                })
                // 使用 auto fac 代替默认的 IOC 容器 
                .UseServiceProviderFactory(new AutofacServiceProviderFactory());
               
    }
```

上面代码中的 CreateHostBuilder 和 asp dotnet core 项目的 Program.cs 的代码差不多，只是 ConfigureWebHostDefaults 方法更改了

而 ConfigureAppConfiguration 是进行配置，这里进行一些测试项目特意的配置，如禁用了我的 APM 服务。在做集成测试的时候，可以选择开启或关闭 APM 服务，如果你的运维小伙伴不会打你，那么还是开始 APM 比较好。这里的代码使用了 [https://github.com/dotnet-campus/dotnetCampus.Configurations](https://github.com/dotnet-campus/dotnetCampus.Configurations) 的方法进行配置

在 MSTest 单元测试项目里面，使用 AssemblyInitialize 特性，可以让某个静态方法在单元测试启动的时候运行一次。而使用 AssemblyCleanup 方法可以在单元测试完成之后，无论是否成功都会调用一次

因此在 GlobalInitialize 方法标记 AssemblyInitialize 特性，在这里面创建主机然后运行主机。此时运行的主机不会去监听端口，因此不能通过端口的形式去调用他，而是需要使用 TestServer 提供的扩展方法获取 HttpClient 去访问。也就是通过 TestHostBuild.GetTestClient 拿到的才能访问这个在内存的主机

我对每个控制器都创建一个测试文件，用来进行单元测试

如我的项目里面有一个 StatusOverviewController 控制器，这个控制器用来返回服务的内容，大概逻辑如下

```csharp
    [ApiController]
    [Route("[controller]")]
    [Route("/")]
    public class StatusOverviewController : ControllerBase
    {
    	[HttpGet]
        public string Get()
        {
        	// 也许使用 DateTimeOffset 更清真，但这又不是我写的
        	return DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss.fff");
        }
    }
```

新建一个单元测试来测试这个接口的访问

```csharp
    [TestClass]
    public class StatusOverviewControllerTest
    {
        [TestMethod]
        public async Task GetTest()
        {
            using var httpClient = TestHostBuild.GetTestClient();

            var result = await httpClient.GetStringAsync("/");
            Assert.IsNotNull(result);
        }
    }
```

大概这样就能完成对这个接口的测试了

当然了这是对简单的接口可以这样写，但是对复杂的接口来说，有很多特殊的需求，此时就需要用到 CUnit 库了，通过安装 MSTestEnhancer 这个 NuGet 库就可以添加单元测试辅助库，如下面代码

```xml
        <PackageReference Include="MSTestEnhancer" Version="2.0.1" />
```

安装之后，就可以写入如下面的逻辑

```csharp
[TestClass]
public class DemoTest
{
    [ContractTestCase]
    public void Foo()
    {
        "当满足 A 条件时，应该发生 A' 事。".Test(() =>
        {
            // Arrange
            // Action
            // Assert
        });
        
        "当满足 B 条件时，应该发生 B' 事。".Test(() =>
        {
            // Arrange
            // Action
            // Assert
        });
    }
}
```

这个 CUnit 在 GitHub 上完全开源，请看 [https://github.com/dotnet-campus/CUnit](https://github.com/dotnet-campus/CUnit)

在准备好了集成测试项目之后，我就开始准备升级到 dotnet 5 了，然而此时发现构建服务器翻车了，如 [刚刚我从服务器回滚了 dotnet 5 的环境](https://blog.lindexi.com/post/%E5%88%9A%E5%88%9A%E6%88%91%E4%BB%8E%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%9B%9E%E6%BB%9A%E4%BA%86-dotnet-5-%E7%9A%84%E7%8E%AF%E5%A2%83.html ) 博客的内容

终于我通过 [如何给 CI CD 服务器搭建上 .NET 5 构建和运行环境](https://blog.lindexi.com/post/%E5%A6%82%E4%BD%95%E7%BB%99-CI-CD-%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA%E4%B8%8A-.NET-5-%E6%9E%84%E5%BB%BA%E5%92%8C%E8%BF%90%E8%A1%8C%E7%8E%AF%E5%A2%83.html ) 的方法修好了

然而小伙伴告诉我从 dotnet core 3.1 到 dotnet 5 有如下的更改 [Breaking changes, version 3.1 to 5.0 - .NET Core](https://docs.microsoft.com/en-us/dotnet/core/compatibility/3.1-5.0#core-net-libraries )

在经过了两天的更新依然失败之后，我强行魔改了代码，上到了 dotet 5 之后，发现了 APM 挂了…… 因 APM 内部使用了原先 dotnet core 3.1 的在 dotnet 5 废弃的接口…… 然后就到了写博客时间了



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
