
# ASP.NET Core 写一个能够接收所有请求路径的测试服务

我在测试一些奇怪的网络请求客户端，需要有一个服务端来配合接受，于是写了一个能够匹配所有请求路径的测试服务

<!--more-->


<!-- CreateTime:2023/2/1 8:55:50 -->

<!-- 发布 -->


代码如下，核心是通过 `{*x}` 进行匹配

以下代码是在 .NET 7 下可用


```csharp
    public static void Main(string[] args)
    {
        var builder = WebApplication.CreateBuilder(args);
        builder.Services.AddSingleton<IHttpContextAccessor>(new HttpContextAccessor());
        var app = builder.Build();

        app.MapGet("{*x}", async () =>
        {
            var httpContext = app.Services.GetRequiredService<IHttpContextAccessor>().HttpContext;
            await Task.Delay(TimeSpan.FromSeconds(1000));
            return "Hello World!";
        });

        app.Run();
    }
```




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。