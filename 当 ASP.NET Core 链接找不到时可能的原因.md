# 当 ASP.NET Core 链接找不到时可能的原因

我逗比用了最新的 dotnet core 3 的预览版本创建了新的项目，但是我发现我的呆魔项目和 Postman 都找不到链接，此时原因是默认的模板创建的路径和之前版本创建的不相同

<!--more-->
<!-- CreateTime:2019/8/26 18:52:28 -->

<!-- csdn -->

在之前版本创建 Controller 时，使用的 Route 是加上了 `api` 路径的，也就是如下面代码

```csharp
    [Route("api/[controller]")]
    public class ResourceController : ControllerBase
```

所以我就直接使用了 `127.0.0.1:5000/api/Resource` 去访问我的链接，但是在 dotnet core 3.0.100-preview7-012821 创建的项目里面，默认将 `api` 去掉，请看下面代码

```csharp
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
```

也就是此时需要使用 `127.0.0.1:5000/WeatherForecast` 才能访问到

所以在发现找不到链接的时候，请先从各个 Route 开始找

[What's new in .NET Core 3.0](https://docs.microsoft.com/en-us/dotnet/core/whats-new/dotnet-core-3-0 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
