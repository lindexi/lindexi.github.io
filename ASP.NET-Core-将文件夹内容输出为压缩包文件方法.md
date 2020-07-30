
# ASP.NET Core 将文件夹内容输出为压缩包文件方法

本文主要是告诉大家一个省内存的方法，将整个文件夹的内容作为一个压缩包输出，但是实际上没有申请那么多的内存，也不需要升级创建一个压缩包文件。原理是通过逐个读文件然后按照压缩包格式输出

<!--more-->


<!-- CreateTime:5/25/2020 7:09:13 PM -->

<!-- 发布 -->

在每个请求的方法可以拿到 HttpContext 属性，通过这个属性拿到 Response 属性，在这里可以使用 BodyWriter 属性，在这个属性里面写入的内容将会被客户端下载

而这个属性可以作为 Stream 请看下面代码

```csharp
     using var stream = HttpContext.Response.BodyWriter.AsStream();
```

在 .NET 中可以通过 ZipArchive 将一个文件夹的文件按照压缩文件格式写入，还可以设置压缩的压缩率等，可以设置文件所在文件夹的路径

通过在这个 stream 创建一个 ZipArchive 类，然后在这个类里面创建文件的方法就可以做到不断向客户端发送文件，发送的文件都在一个压缩包里面

```csharp
        /// <summary>
        /// 将一个文件夹的内容读取为 Stream 的压缩包
        /// </summary>
        /// <param name="directory"></param>
        /// <param name="stream"></param>
        public static async Task ReadDirectoryToZipStreamAsync(DirectoryInfo directory, Stream stream)
        {
            var fileList = directory.GetFiles();

            using var zipArchive = new ZipArchive(stream, ZipArchiveMode.Create);
            foreach (var file in fileList)
            {
                var relativePath = file.FullName.Replace(directory.FullName, "");
                if (relativePath.StartsWith("\\") || relativePath.StartsWith("//"))
                {
                    relativePath = relativePath.Substring(1);
                }

                var zipArchiveEntry = zipArchive.CreateEntry(relativePath, CompressionLevel.NoCompression);

                using (var entryStream = zipArchiveEntry.Open())
                {
                    using var toZipStream = file.OpenRead();
                    await toZipStream.CopyToAsync(stream);
                }

                await stream.FlushAsync();
            }
        }
```

上面的代码可以让运行的程序不需要申请和需要传输一样大的内存空间，或者不需要先执行压缩放在本地文件，可以不断读取本地文件然后上传。读取本地文件等都通过 CopyToAsync 自动设置缓存大小。如果不放心 CopyToAsync 方法设置的缓存大小，可以通过重载的方法手动设置缓存的大小

```csharp
      await toZipStream.CopyToAsync(stream, bufferSize: 100);
```

上面的代码设置了文件不要压缩，因为作为文件传输的时候，实际上我的业务是在内网传输，我的磁盘读取速度大概是 20M 一秒，而网络传输是 10M 一秒，也就是此时的压缩其实没什么意义，压缩减少的内容减少的传输时间就和压缩的时间差不多

如果小伙伴需要传输的时候压缩，请设置 zipArchive.CreateEntry 方法

当然此方法的缺点是，也许传输的时候服务器自己读取文件炸了，此时就会传输的文件不对，同时客户端不知道服务器传的对不对，因为压缩的大小没有告诉客户端。如果要告诉客户端压缩后的大小就需要先在服务器端进行压缩。本文的方法设置的是没有压缩率的压缩，大概的大小还可以告诉用户

此方法可以如何使用？在随意一个 Get 方法里面就可以通过 HttpContext 传入 Response 属性

在使用 BodyWriter 写入之前需要先设置 StatusCode 的值

```csharp
            HttpContext.Response.StatusCode = StatusCodes.Status200OK;

            using var stream = HttpContext.Response.BodyWriter.AsStream();
```

假设需要返回的文件夹是 `f:\lindexi\test\` 可以通过下面代码的方式将文件夹输出为压缩包

```csharp
        [HttpGet]
        [Route("{id}")]
        public async Task Get([FromRoute] string id)
        {
            var folder = @"f:\lindexi\test\";
            HttpContext.Response.StatusCode = StatusCodes.Status200OK;

            using var stream = HttpContext.Response.BodyWriter.AsStream();

            await ReadDirectoryToZipStreamAsync(new DirectoryInfo(folder), stream);
        }
```

本地我写了一个 PowerShell 脚本运行

```powershell
For ($i=0; $i -le 100000; $i++) 
{
 (new-object System.Net.WebClient).DownloadFile("http://localhost:5000/File/doubi", "F:\lindexi\zip\2.zip")
} 
```

本地运行这个脚本可以看到内存其实没有 GC 也没有溢出，我运行看到内存大概在 100M 左右

获取的时候会占用一些 CPU 资源，但是很省内存

如果小伙伴有更好的方法欢迎告诉我

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/32e2de954d92cc9fa359ae6eacd327405e156fe4/LarnaceakemLachonanafemhejal)欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。