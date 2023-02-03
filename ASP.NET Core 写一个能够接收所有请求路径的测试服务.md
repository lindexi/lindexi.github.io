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