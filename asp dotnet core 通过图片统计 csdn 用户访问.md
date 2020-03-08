# asp dotnet core 通过图片统计 csdn 用户访问

在 csdn 的访问统计里面，只能用 csdn 提供的访问统计，因为在 csdn 中不支持在博客加上 js 代码，也就是无法使用友盟等工具统计。

通过在 asp dotnet core 创建一个图片链接的方式，将这个链接作为图片放在 csdn 的博客，可以在链接被访问的时候统计用户访问

<!--more-->
<!-- CreateTime:2019/11/29 8:26:58 -->


新建一个 asp dotnet core 项目，在自己的 controler 里面添加一个方法，这个方法的访问链接是 `xx.png` 假装这是一张图片，请看代码

```csharp
    [Route("api/[controller]")]
    [ApiController]
    public class ImageController : ControllerBase
    {
    	[Route("csdn/Image.png")]
        [HttpGet]
        public FileResult GetCSDNImage()
        {
        	// 忽略代码
        }
    }
```

在 csdn 上通过访问我的域名对应的链接就可以调用 GetCSDNImage 方法

```
![] (http://bulleimage.service.walterlv.com/api/image/csdn/image.png )
```

注意路由的命名要清真一点，我本来是用来做广告的，于是命名为广告，然而发现在 csdn 无法被访问到，原因请看 [asp dotnet core 图片在浏览器没访问可能原因](https://blog.lindexi.com/post/asp-dotnet-core-%E5%9B%BE%E7%89%87%E5%9C%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B2%A1%E8%AE%BF%E9%97%AE%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0.html )

通过在 GetCSDNImage 返回一张图片，同时按照这个方法调用的次数就可以用来统计用户的访问了

## 缓存图片

在 GetCSDNImage 需要返回图片才可以用来假装这是一个静态图片的链接

我将一张图片放在输出文件夹，做法就是在解决方案添加一张图片，右击属性设置复制输出到文件夹

在 Startup 的 ConfigureServices 里面添加内存缓存

```csharp
        public void ConfigureServices(IServiceCollection services)
        {
        	// 忽略其他代码
            services.AddMemoryCache(); // 添加这一行代码就可以使用内存缓存

            services.AddControllers()
                .AddNewtonsoftJson();
            // 忽略代码
        }

```

修改 ImageController 在构造注入缓存 IMemoryCache 方法

```csharp
        public ImageController(IMemoryCache memoryCache)
        {
            _cache = memoryCache;
        }

        private IMemoryCache _cache;
```

这样就可以在 GetCSDNImage 方法里面使用缓存了

在使用缓存之前需要读取输出文件夹里面的图片，我添加一个方法用来读取输出文件夹里面的图片。在 asp dotnet core 中，很多都是使用指定静态的文件夹作为静态资源的文件夹，直接通过输出文件夹读取的比较少。但是设置一个静态文件夹是另外的知识，本文就直接通过输出文件夹读取

使用 `Path.GetDirectoryName(Assembly.GetCallingAssembly().Location)` 可以拿到输出文件夹，详细请看[dotnet 获取程序所在路径的方法](https://blog.lindexi.com/post/dotnet-%E8%8E%B7%E5%8F%96%E7%A8%8B%E5%BA%8F%E6%89%80%E5%9C%A8%E8%B7%AF%E5%BE%84%E7%9A%84%E6%96%B9%E6%B3%95.html )

使用下面代码就可以读取图片文件作为字节了，建议只读取一次，解决多线程访问文件的问题

```csharp
        private byte[] GetImage()
        {
            var file = Path.Combine(Path.GetDirectoryName(Assembly.GetCallingAssembly().Location), "Image.png");

            return System.IO.File.ReadAllBytes(file);
        }
```

在 GetCSDNImage 方法里面通过缓存，判断如果缓存里面没有值就从文件读取图片，如果有值就直接从内存返回

如果这段代码是需要我自己写，可能要写几天，因为还存在了多线程访问的问题，如果一开始不存在值，那么就需要创建值，如何作为第一次创建值的时候，刚好多个线程进来，只有一个线程创建等问题。还好微软提供的缓存里面有这样的方法 GetOrCreate 方法，尝试从内存获取，如果获取不到就创建，在这个方法里面第一个参数是传入 key 第二个参数就是传入如何创建的方法。在缓存中就通过 key 来获取或设置值，用法和字典差不多

```csharp
            var file = _cache.GetOrCreate("Image", entry => GetImage());

```

现在拿到了值，可以通过 File 方法返回，注意在返回的时候添加 content type 说这是一张图片

```csharp

        public FileResult GetCSDNImage()
        {     
             var file = _cache.GetOrCreate("Image", entry => GetImage());
      
             return File(file, "image/png");
        }
```

## 统计用户访问

我不会告诉大家如何去创建数据库去存放用户访问的数据，因为这些需要的知识点有些多，本文的统计用户访问只是通过一个简单的静态变量获取，不考虑并发的问题

```csharp

        public FileResult GetCSDNImage()
        {     
        	Count++;

        	Console.WriteLine($"总共有{Count}访问");

             
            // 忽略其他代码
        }

        private static int Count { set; get; }

```

这样就可以完成了统计用户的访问了，同时代码也很少。我还需要用户的 IP 和使用什么浏览器，于是需要添加一点代码

我的网站是通过 frp 让用户访问，需要[从 Frp 获取用户真实 IP 地址](https://blog.lindexi.com/post/asp-dotnet-core-%E4%BB%8E-Frp-%E8%8E%B7%E5%8F%96%E7%94%A8%E6%88%B7%E7%9C%9F%E5%AE%9E-IP-%E5%9C%B0%E5%9D%80.html )用法有点复杂

```csharp
        private static bool TryGetUserIpFromFrp(HttpRequest httpContextRequest, out StringValues ip)
        {
            return httpContextRequest.Headers.TryGetValue("X-Forwarded-For", out ip);
        }
```

上面这个方法就可以从 HttpRequest 拿到从 frp 获取的用户 ip 地址

```csharp
        public FileResult GetCSDNImage()
        {
        	// 忽略代码

            if (TryGetUserIpFromFrp(HttpContext.Request, out var ip))
            {
               
            }

        	// 忽略代码
        }

```

获取用户的浏览器使用 Headers 里面的 `User-Agent` 可以拿到

```csharp
        public FileResult GetCSDNImage()
        {
        	// 忽略代码

            if (HttpContext.Request.Headers.TryGetValue("User-Agent", out var userAgent))
            {
               
            }

        	// 忽略代码
        }
```

将这些值合并输出

```csharp
        [Route("csdn/Image.png")]
        [HttpGet]
        public FileResult GetCSDNImage()
        {
            Count++;

            StringBuilder str = new StringBuilder();
            str.Append(DateTime.Now);
            str.Append(" ");
            str.Append("用户访问 ");

            Console.WriteLine(GetUserId());
            Console.WriteLine("用户id =" + HttpContextAccessor.HttpContext.Request.HttpContext.Session.Id);

            if (TryGetUserIpFromFrp(HttpContextAccessor.HttpContext.Request, out var ip))
            {
                str.Append("用户Ip=");
                str.Append(ip);
                str.Append(" ");
            }

            str.Append($"总共有{Count}访问");

            if (HttpContext.Request.Headers.TryGetValue("User-Agent", out var userAgent))
            {
                str.Append("\r\n");
                str.Append("当前用户浏览器");
                str.Append(userAgent);
            }

            Console.WriteLine(str);

            var file = _cache.GetOrCreate("Image", entry => GetImage());

            return File(file, "image/png");
        }
```

我将这个图片放在 csdn 的博客，运行服务在用户访问的时候可以看到下面的输出

```csharp
2019/5/26 11:39:24 用户访问 用户Ip=58.209.53.254 总共有13访问
当前用户浏览器Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/74.0.3729.157 Safari/537.36
```

如果要分析什么时候的用户访问有多少，就需要添加数据库，这些需要的知识有点多，本文就不告诉大家

代码很简单，都放在 [github](https://github.com/lindexi/lindexi_gd/tree/aec484bd2aa169ed31ef4a90dd462e72c2e32064/FayawlerjiraywereNiharjeechel )

[dotnet 获取程序所在路径的方法](https://blog.lindexi.com/post/dotnet-%E8%8E%B7%E5%8F%96%E7%A8%8B%E5%BA%8F%E6%89%80%E5%9C%A8%E8%B7%AF%E5%BE%84%E7%9A%84%E6%96%B9%E6%B3%95.html )

[asp dotnet core 从 Frp 获取用户真实 IP 地址](https://blog.lindexi.com/post/asp-dotnet-core-%E4%BB%8E-Frp-%E8%8E%B7%E5%8F%96%E7%94%A8%E6%88%B7%E7%9C%9F%E5%AE%9E-IP-%E5%9C%B0%E5%9D%80.html )

[asp dotnet core 图片在浏览器没访问可能原因](https://blog.lindexi.com/post/asp-dotnet-core-%E5%9B%BE%E7%89%87%E5%9C%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B2%A1%E8%AE%BF%E9%97%AE%E5%8F%AF%E8%83%BD%E5%8E%9F%E5%9B%A0.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
