# ASP.NET Core 开启后台任务

本文告诉大家如何通过 Microsoft.Extensions.Hosting.BackgroundService  开启后台任务

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->

<!-- 标签：asp,aspdotnetcore,dotnetcore -->

实现 BackManagerService 类继承 BackgroundService 抽象类，请看代码

```csharp
    public class BackManagerService : BackgroundService
    {
        /// <inheritdoc />
        protected override Task ExecuteAsync(CancellationToken stoppingToken)
        {
            return Task.CompletedTask;
        }
    }
```

然后打开 Startup.cs 在 ConfigureServices 方法注入

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSingleton<IHostedService, BackManagerService>();

            services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
        }
```

现在运行 ASP.NET Core 程序就可以看到调用进 ExecuteAsync 方法了

那么如何实现轮询？大概在30秒左右做某个任务？在没有用任何设计的情况，假如这个任务就放在了 BackManagerService 的 Foo 方法，可以通过下面代码调用

```csharp
        protected override async Task ExecuteAsync(CancellationToken stoppingToken)
        {
            while (!stoppingToken.IsCancellationRequested)
            {
                Foo();
                await Task.Delay(TimeSpan.FromSeconds(30), stoppingToken);
            }
        }
```

通过 Task.Delay 的方法延迟指定的时间就可以了，那么更复杂的封装就在大佬们的封装变得更加好用，更多封装请看 Ron 大佬博客

所有代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/8260ae90d65d616e284b54841e2f95ca6a34f8a3/KorburxetiCheewharorwale )

[Asp.Net Core 轻松学-基于微服务的后台任务调度管理器 - Ron.Liang - 博客园](https://www.cnblogs.com/viter/p/10078488.html )

[在 ASP.NET Core 中使用托管服务实现后台任务](https://docs.microsoft.com/zh-cn/aspnet/core/fundamentals/host/hosted-services?view=aspnetcore-2.2 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
