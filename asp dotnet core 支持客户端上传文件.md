# asp dotnet core 支持客户端上传文件

本文告诉大家如何在 asp dotnet core 支持客户端上传文件

<!--more-->
<!-- CreateTime:2019/8/31 16:55:58 -->


新建一个 asp dotnet core 程序，创建一个新的类，用于给客户端上传文件的信息

```csharp
    public class KanajeaLolowge
    {
        public IFormFile File { get; set; }

        public string Sha { get; set; }
    }
```

这个类包含两个信息，一个是 File 文件，另一个是文件校验，这个类可以随意命名，属性也可以随意命名，只要在客户端可以相同

打开一个 controller 添加一个新的函数

```csharp
        [HttpPost("UploadPackage")]
        public async Task<StatusCodeResult> UploadPackage([FromForm]KanajeaLolowge file)
        {
      
            return Ok();
        }
```

在这个函数添加特性 HttpPost 里面的参数就是访问链接，如上面的代码，加入所在的类是这样定义

```csharp
    [Route("api/[controller]")]
    [ApiController]
    public class GairKetemRairsemsController : ControllerBase
```

那么可以通过 `api/GairKetemRairsems/UploadPackage` 访问这个函数

在函数传入的参数使用 `[FromForm]KanajeaLolowge file` 的意思是通过 Post 提交 Form 的方法拿到参数

所以在客户端通过 Form 提交包含 File 和 Sha 信息的 Form 就可以通过链接调用这个方法

请看在客户端如何写

假设已经拿到文件的 FileStream 和计算出文件的 Sha 拿到链接

```csharp
        private static async Task Upload(FileStream fileStream, string sha, string url)

```

在这个方法上传文件，通过 MultipartFormDataContent 构造一个 Form 请看代码

```csharp
            var multipartFormDataContent = new MultipartFormDataContent();
            multipartFormDataContent.Add(new StreamContent(fileStream), "File", fileName: "文件名.png");
            multipartFormDataContent.Add(new StringContent(sha), "Sha");
```

文件通过 StreamContent 传入 Stream 的值，加上的 File 参数是和 asp dotnet core 的上传类对应的属性名，最后的一个 fileName 指的是文件名，因为传入的是 Stream 可能是内存流，这时就无法拿到文件名，需要用户传入

下面的值是传入一个 Key value 的值，这里的 key 是 `"Sha"` 值是 sha 的值

创建一个 HttpClient 上传 Form 代码

```csharp
            var httpClient = new HttpClient();
            await httpClient.PostAsync(url, multipartFormDataContent);
```

这样调用这个 PostAsync 在 Asp dotnet core 就调用 UploadPackage 方法

通过这个方法就可以做到在 asp dotnet core 上传文件

获取文件 Sha 的方法请看下面

```csharp
            var fileStream = fileInfo.OpenRead();
            string fileSha;
            using (var sha = SHA256.Create())
            {
                fileSha = Convert.ToBase64String(sha.ComputeHash(fileStream));

                fileStream.Seek(0, SeekOrigin.Begin);
            }
```

注意需要 fileStream 还原，也就是 `fileStream.Seek(0, SeekOrigin.Begin)` 如果没有添加，那么上传的 Stream 是没有长度

在 asp dotnet core 接收文件，然后校验文件的方法请看下面

```csharp
       [HttpPost("UploadPackage")]
        public async Task<StatusCodeResult> UploadPackage([FromForm]KanajeaLolowge file)
        {
            var fileInfo = new FileInfo("E:\\1.png");

            var fileStream = fileInfo.Open(FileMode.Create, FileAccess.ReadWrite);

            await file.File.CopyToAsync(fileStream);

            fileStream.Seek(0, SeekOrigin.Begin);

            string fileSha;
            using (var sha = SHA256.Create())
            {
                fileSha = Convert.ToBase64String(sha.ComputeHash(fileStream));

                fileStream.Seek(0, SeekOrigin.Begin);
            }

            if (fileSha == file.Sha)
            {
                return Ok();
            }

            return BadRequest();
        }
```

如果需要将文件保存在 ContentRoot 可以通过在 controller 的构造函数添加 IHostingEnvironment 参数，在 IHostingEnvironment 参数拿到 ContentRootPath 值

```csharp
        public GairKetemRairsemsController(IHostingEnvironment hostingEnvironment)
        {
            _context = context;
            _hostingEnvironment = hostingEnvironment;
        }
       
        private readonly IHostingEnvironment _hostingEnvironment;
```

在保存文件可以使用下面代码

```csharp
            var fileInfo = new FileInfo(Path.Combine(_hostingEnvironment.ContentRootPath, "1.png"));

```

[ASP.NET Core文件上传与下载(多种上传方式) - GuZhenYin - 博客园](https://www.cnblogs.com/GuZhenYin/p/8194726.html )

[File uploads in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads?view=aspnetcore-2.2 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
