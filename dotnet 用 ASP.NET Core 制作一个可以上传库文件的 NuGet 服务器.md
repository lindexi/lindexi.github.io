# dotnet 用 ASP.NET Core 制作一个可以上传库文件的 NuGet 服务器

我在写一个有趣的 WPF 应用，我想要测试这个 WPF 应用的一个功能，这个功能就是一键点击自动推送 NuGet 包到服务器。我想要做一点自动化的测试，我需要有某个假装是 NuGet 的服务器用来接收我这个应用推送的 NuGet 包。用 ASP.NET Core 写一个假装的 NuGet 服务器，支持被 NuGet 推送包是特别简单的，本文就来和大家说说这个后台如何写

<!--more-->
<!-- CreateTime:4/5/2020 3:39:34 PM -->



其实有现成的整个 NuGet 服务器，包含了包的列举和上传等功能，这就是 [BaGet](https://github.com/loic-sharma/BaGet ) 项目，但是这个项目存在的问题是太大了，我想要做到自动测试里面去，又有很多有趣的逻辑需要写

那么自己从零开始写一个 NuGet 服务器，这个服务器只有一个功能就是接收 NuGet 推送的包，这个工作量大不大？在使用 ASP.NET Core 时只能说工作量特别小

下面让我用 3 分钟告诉大家如何在 asp dotnet core 里面写一个支持被推送 nuget 包的服务器

首先是创建一个空白的工程，此时这个功能请去掉 https 的逻辑，注意去掉 Startup.cs 的重定向逻辑。先跑通过了 http 之后小伙伴自己再去配置 https 哦

根据 [官方文档](https://docs.microsoft.com/en-us/nuget/api/package-publish-resource ) 说的，默认的 NuGet 的上传文件就是通过发送一个 multipart form data 数据，发送到制定的源里面，例如我准备推送 [ant-design-blazor.nupkg](https://github.com/lindexi/ant-design-blazor ) 库的时候，通过下面代码

```
nuget push -Source http://localhost:49614/api/v2/package AntBlazor.0.0.1.nupkg -ApiKey 123
```

将会向服务器 `http://localhost:49614/api/v2/package` 发送一个 multipart form data 数据，这个数据里面只包含了一个文件信息

在 asp dotnet core 可以通过下面代码接收 form 表单信息

```csharp
        [HttpPut]
        public async Task<IActionResult> Push([FromForm]FilePackage package)
        {

        }
```

注意加上 FromForm 特性，如果没有加上特性那么将会返回客户端 415 Unsupported Media Type 信息

这里的 FilePackage 的定义如下

```csharp
    public class FilePackage
    {
        public IFormFile Package { set; get; }
    }
```

此时从参数里面拿到的 package 属性就是客户端上传的对应的 NuGet 库

修改一下控制器的路径，这样才好假装这是一个 NuGet 服务器

```csharp
    [ApiController]
    [Route("api/v2/[controller]")]
    public class PackageController : ControllerBase
    {
        [HttpPut]
        public async Task<IActionResult> Push([FromForm]FilePackage package)
        {
        	// 代码
        }
    }
```

这样就完成了制作一个假装的 NuGet 上传服务器了

其实如果不从参数里面获取客户端上传的 NuGet 库，还可以通过 HttpContext.Request.Form 拿到，请看代码

```csharp
            var packageFile = package.Package;
            if (packageFile == null)
            {
                packageFile = HttpContext.Request.Form.Files.FirstOrDefault();
            }
```

而将 IFormFile 保存到本地也很简单，请看代码

```csharp
            var file = Path.GetTempFileName();
            using (var stream = new FileStream(file, FileMode.OpenOrCreate))
            {
                await packageFile.CopyToAsync(stream);
            }
```

大概用不到3分钟就能完成一个假装的 NuGet 服务器

细心的小伙伴还发现了刚才的命令行有添加 ApiKey 信息，这个信息可以偷偷在 HttpContext.Request.Headers 拿到

```csharp
             var key = HttpContext.Request.Headers["X-NuGet-ApiKey"];
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/457ebad9ce3895bde7c76ae60bd8c4c4be6f93b4/AluwemjealayCheedeaweabewairhur) 欢迎小伙伴访问

顺便广告一下 [ant-design-blazor](https://github.com/ElderJames/ant-design-blazor ) 这个使用 Blazor 的 ant 界面库，欢迎小伙伴关注

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
