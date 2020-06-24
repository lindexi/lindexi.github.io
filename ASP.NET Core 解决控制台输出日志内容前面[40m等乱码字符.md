# ASP.NET Core 解决控制台输出日志内容前面[40m等乱码字符

在默认我写了一个 WPF 程序去做管理 ASP.NET Core 进程的日志的时候，重定向输出的内容里面每一行前面都添加了很多乱码字符串。其实这是 ASP.NET Core 控制台的颜色字符，解决方法是禁用控制台颜色

<!--more-->
<!-- 发布 -->

在看到重定向的输出里面包含以下乱码字符

```csharp
[40m[32minfo[39m[22m[49m:
```

或者如下乱码字符

```csharp
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

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
