# ASP.NET Core 解决控制台输出日志内容前面[40m等乱码字符

在默认我写了一个 WPF 程序去做管理 ASP.NET Core 进程的日志的时候，重定向输出的内容里面每一行前面都添加了很多乱码字符串。其实这是 ASP.NET Core 控制台的颜色字符，解决方法是禁用控制台颜色

<!--more-->
<!-- CreateTime:6/24/2020 3:19:38 PM -->



在看到重定向的输出里面包含以下乱码字符

```
[40m[32minfo[39m[22m[49m:
```

或者如下乱码字符

```
[41m[30mfail[39m[22m[49m
```

这里 asp dotnet core 的日志内容的 `[40m[32m` 和 `[41m[30m` 字符是控制颜色的字符，可以在 Startup.cs 的 ConfigureServices 方法使用下面代码禁用控制台输出颜色

```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddLogging(builder => builder
        .ClearProviders()
        // 这里的颜色会被显示为 [41m[30mfail[39m[22m[49m 的内容
        // [40m[32minfo[39m[22m[49m:
        .AddConsole(options => { options.DisableColors = true; }));
}
```

注意上面代码使用 ClearProviders 清空了日志输出，上面代码禁用颜色用的是 `options.DisableColors = true;` 禁用

禁用之后输出控制台没有颜色，但重定向的日志里面也没有乱码

另外，在 WPF 之外，在 Kubernetes 上的重定向输出也会是差不多的乱码

```
�[40m�[32minfo�[39m�[22m�[49m: Microsoft.Hosting.Lifetime[0]
      Now listening on: http://[::]:12312
�[40m�[32minfo�[39m�[22m�[49m: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
�[40m�[32minfo�[39m�[22m�[49m: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Production
�[40m�[32minfo�[39m�[22m�[49m: Microsoft.Hosting.Lifetime[0]
      Content root path: /lindexi/doubi
```

解决方法和上面相同

---

更新： 对于 .NET 5 及更加新的版本，需要使用以下代码禁用颜色

```csharp
loggingBuilder.AddSimpleConsole(options =>
{
    options.ColorBehavior = LoggerColorBehavior.Disabled;
});
```

详细请看 <https://learn.microsoft.com/en-us/dotnet/core/compatibility/core-libraries/5.0/obsolete-consoleloggeroptions-properties>