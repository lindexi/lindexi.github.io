# asp dotnet core 不正经的提升效率的单元测试方法

在写 asp dotnet core 时，如果没有单元测试保证，需要每个方法都从 web api 的入口开始运行，此时的执行效率是很低的。而如果写单元测试，又有一个坑的问题是写单元测试也是需要时间的。本文告诉大家一些提高效率的方法，这些方法不是正经的用法，但是能提升效率。至于能不能用好不好用就请观众老爷自己决定

<!--more-->
<!-- CreateTime:2020/2/1 17:04:53 -->

<!-- 发布 -->

## CUnit 中文命名单元测试

在写单元测试时，小伙伴说需要让单元测试的方法名符合 `条件_执行_结果` 而要求这个方法命名为英文，我的英文就超级渣，这一点 [少珺](https://blog.sdlsj.net/) 小伙伴可以帮我证明。于是你会看到我写了以下的测试 `WhenABuDengYuThree_DokanarkelawNinirahajairi_SetSlj` 的命名，而如果要我优化这个单元测试的命名，大家都知道，有些小伙伴和我一样想一个好的命名可能占了开发的一半时间

写单元测试时，大量的单元测试方法命名将会占用大量的时间，让小伙伴不愿意写单元测试。或者写出来的单元测试的只有自己能读懂

在一个团队里面的，如果英文水平参差不齐，如我所在的团队有英文特别厉害的[walterlv](https://walterlv.com)和[天龙](https://getandplay.github.io/ )也有英文特别差国语也特别差的大壮哥，还有英文有毒的本渣。此时用英文命名的单元测试就是一个神坑，除非团队能成立一个改名部专门协助命名

一个解决方法是干脆用中文命名单元测试算了，请看下面单元测试

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

运行单元测试将看到这样的结果视图

![](http://image.acmx.xyz/lindexi%2F20202111018744.jpg)

只要有任何一个单元测试炸了，相信小伙伴看提示特别快就知道哪里炸了

使用这个库的前提是用 NuGet 安装 [MSTestEnhancer](https://www.nuget.org/packages/MSTestEnhancer) 库，如果是 SDK 格式的项目文件，可以添加下面代码

```
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.2.0" />
    <PackageReference Include="Moq" Version="4.13.1" />
    <PackageReference Include="MSTest.TestAdapter" Version="2.0.0" />
    <PackageReference Include="MSTest.TestFramework" Version="2.0.0" />
    <PackageReference Include="coverlet.collector" Version="1.0.1" />
    <PackageReference Include="MSTestEnhancer" Version="1.6.0" />
  </ItemGroup>
```

注意版本号需要你自己更新

在单元测试的方法里面，推荐写某个需要测试的方法，在方法上面添加特性 `ContractTestCase` 请看代码

```csharp
    [ContractTestCase]
    public void Foo()
```

接下来在方法里面用一段字符串和 `.Test` 写出对应的单元测试

```csharp
        "当满足 A 条件时，应该发生 A' 事。".Test(() =>
        {
            // Arrange
            // Action
            // Assert
        });
```

一个例子是我在[DotNetGitLabWebHook](https://github.com/dotnet-campus/DotNetGitLabWebHook )用到的方法，代码请看 [github](https://github.com/dotnet-campus/DotNetGitLabWebHook/blob/da88f6b108b10f87fdc78231628da603363db205/DotNetGitLabWebHookToMatterMost.Tests/Business/Check/RepoManagerTests.cs) 是不是觉得写起来特别快

用 CUnit(MSTestEnhancer) 能让团队内小伙伴写单元测试的效率提升，也能提升团队里面读单元测试以及单元测试炸了解决的效率

现在问题只有一个，你的团队内对中文的看法是如何？千万不要在我的博客下评论，我的博客的评论做的很渣，如果有很多人都在评论我的博客就用不了

## 利用原有依赖注入

在 asp dotnet core 的各个类可以在构造函数添加依赖注入的方法，如我的[DotNetGitLabWebHook](https://github.com/dotnet-campus/DotNetGitLabWebHook )就在各个类里面的构造函数添加了依赖注入

在 asp dotnet core 默认的构造函数依赖注入非常好用，例如我的 [GitLabMRCheckerFlow.cs](https://github.com/dotnet-campus/DotNetGitLabWebHook/blob/da88f6b108b10f87fdc78231628da603363db205/DotNetGitLabWebHook/Business/GitLabMRCheckerFlow.cs ) 用到两个类 Notify 和 FileChecker 类，而 Notify 用到了 IConfiguration 配置，于是我可以这样写

```csharp
    public class Notify
    {
        /// <inheritdoc />
        public Notify(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        // 忽略代码
    }

    // GitLabMRCheckerFlow.cs

            public GitLabMRCheckerFlow(Notify notify, FileChecker fileChecker)
```

在 Startup.cs 的 ConfigureServices 添加注入

```csharp
            services.AddScoped<GitLabMRCheckerFlow>();
            services.AddScoped<Notify>();
            services.AddScoped<FileChecker>();
```

代码请看 [Startup.cs](https://github.com/dotnet-campus/DotNetGitLabWebHook/blob/da88f6b108b10f87fdc78231628da603363db205/DotNetGitLabWebHook/Startup.cs )

此时获取对象的方法都是放在构造函数参数，此时各个参数对应的类的创建也会自动注入构造参数。如在 GitLabMRCheckerFlow 需要传入 Notify 参数，而创建 Notify 类需要传入 IConfiguration 参数，这些都会在自带的依赖注入完成

在写 Controller 的单元测试时，难道我是需要运行一个 ASP.NET Core 服务，然后用 postman 进行测试？这样的效率太低了，可以尝试直接创建类调用对应的方法。而如果需要每个类都自己创建，这个创建效率实在太低，因为创建一个类需要在他的构造函数传入其他类，而这个类的构造函数可能后续修改，这样的单元测试小伙伴都想砍人

简单的方法是在单元测试创建服务

```csharp
            var hostBuilder = Program.CreateHostBuilder(new string[0]);
            var build = hostBuilder.Build();
            var serviceProvider = build.Services;
```

上面的代码在单元测试里面调用，调用上面代码将会创建服务

然后拿到 serviceProvider 创建对象。如我需要测试 GitLabWebHookController 我可以给他的构造函数每个参数都在 serviceProvider 获取，此时就不需要手动创建

```csharp
        public void MergeRequestTest()
        {
            var hostBuilder = Program.CreateHostBuilder(new string[0]);
            var build = hostBuilder.Build();
            var serviceProvider = build.Services;
            
            using (var scope = serviceProvider.CreateScope())
            {
                var gitLabMrCheckerFlow = scope.ServiceProvider.GetService<GitLabMRCheckerFlow>();

                var gitLabWebHookController = new GitLabWebHookController(gitLabMrCheckerFlow);
                gitLabWebHookController.MergeRequest(TestMRJson.GetObject());
            }
        }
```

这里有细节是 Controller 的注入有很多参数都是在 Scope 需要创建

而如果我的 Controller 有某些参数需要使用 Fake 或 Mock 的，这些参数就自己用 Mock 啦

通过这个方法会降低单元测试运行速度，但是能提升写单元测试的效率

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
