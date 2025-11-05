---
title: ASP.NET Core 制作一个低资源占用的支持超大文件表单上传的服务
description: 故事的背景是我准备制作一个签名服务，为打包构建之后的产物文件进行签名和对其内容信息进行扫描。在这个过程里面，我需要搭建一个 ASP.NET Core 服务，这个服务要能承载客户端上传的超大文件表单，且预算有限，此服务占用资源要足够低
tags: ASP.NETCore
category: 
---

<!-- 发布 -->
<!-- 博客 -->

上传文件到服务器的经典方法是采用表单上传的方式

在 ASP.NET Core 的默认实现中，无论是直接在参数上写 FromFormAttribute 配合 IFormFile 接收文件，还是通过 `HttpRequest.ReadFormAsync` 方法，对于客户端传入的大文件，都会先缓存到磁盘里面。这也就是为什么会有一些开发者会误认为使用 IFormFile 类型属性时，可以立刻接收到客户端发送过来的文件而在有需要读取时，才开始接收读取的原因

事实上，对于超过缓存大小的表单请求文件，默认的 ASP.NET Core 实现将会先接收客户端的输入数据，将其存放到本地临时文件中。随后再调用业务层的逻辑，构建的 IFormFile 类型读取的内容实际上是从文件读取的。这样的设计的原因是客户端的表单上传可能不是将文件放在末尾，这就意味着只有完全接收了表单，才能知道整个表单包含了哪些内容。比如类似如下的客户端表单上传逻辑：

```csharp
    // 以下是测试代码
    using var httpClient = new HttpClient();

    using var multipartFormDataContent = new MultipartFormDataContent();
    using var fakeLongStream = new FakeLongStream();
    multipartFormDataContent.Add(new StreamContent(fakeLongStream), "TheFile", "FileName.zip");
    multipartFormDataContent.Add(new StringContent("Value1"), "Field1");
    var response = await httpClient.PostAsync($"{url}/PostMultipartForm", multipartFormDataContent);
    response.EnsureSuccessStatusCode();
```

以上的 FakeLongStream 是一个假装是超大文件的 Stream 类型。通过以上代码可见，先是在表单添加了超大文件，随后再添加 Field1 表单内容。这就意味着服务端如果没有完全接收整个表单，则无法列举出整个表单包含的内容。服务端不能无限缓存大表单数据到内存，于是只好先存放到本地磁盘临时文件

可见在此过程里面，整个 ASP.NET Core 的默认实现的服务端，对于超大文件是不能快速响应的。而且也难以预先判断请求合法性，最多只能判断 HEAD 请求头，而不能根据表单读取内容决定是否拒绝响应

对于本文我提及的需求，制作一个签名服务器来说，我本身的服务器性能和磁盘空间都很小。客户端上传的超大文件都会真的超级大，都是按 G 为单位。如果是真等 ASP.NET Core 完全读取表单，缓存到本地文件，随后我再从本地文件读取缓存，计算签名信息，那么这个过程里面不仅占用资源多，且响应速度缓慢。毕竟读写磁盘的速度肯定没有我直接计算签名来得快

好在 ASP.NET Core 从设计上就是自由的，不仅提供了上层的简单方便用法，也提供了底层的基础实现方式。核心官方文档是： <https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads?view=aspnetcore-9.0>

为了方便演示，我这里创建了一个 Mini API 的 ASP.NET Core 项目。且为了简化我的需求，可以认为我只期望对上传的表单文件计算 SHA1 哈希值然后返回给到客户端

先通过 MapPost 映射请求信息，删减后的代码如下

```csharp
WebApplication app = ...
app.Urls.Add(url);

app.MapPost("/PostMultipartForm", async (Microsoft.AspNetCore.Http.HttpContext context) =>
{
    ...
});
```

在此之前，为了让 ASP.NET Core 能够接收超大文件，需要设置无限制请求体大小，其代码如下

```csharp
var builder = WebApplication.CreateSlimBuilder(args);

builder.WebHost.UseKestrel(options =>
{
    // 无限制请求体大小
    // Microsoft.AspNetCore.Server.Kestrel.Core.BadHttpRequestException:“Request body too large. The max request body size is 30000000 bytes.”
    options.Limits.MaxRequestBodySize = null;
});
```

提前了解到将执行表单传输，表单传输需要获取 Boundary 分隔符，利用 MediaTypeHeaderValue 辅助类进行转换，有删减的代码如下

```csharp
app.MapPost("/PostMultipartForm", async (Microsoft.AspNetCore.Http.HttpContext context) =>
{
    var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();

    var request = context.Request;
    var response = context.Response;

    string? contentType = request.ContentType;
    if (contentType is null)
    {
        return;
    }

    MediaTypeHeaderValue mediaTypeHeaderValue = MediaTypeHeaderValue.Parse(contentType);
    var contentTypeBoundary = mediaTypeHeaderValue.Boundary;
    var boundary = HeaderUtilities.RemoveQuotes(contentTypeBoundary).Value!;

    ...
});
```

准备工作完成之后，就可以使用本文用到的核心类 MultipartReader 进行处理。传入 boundary 和 request.Body 给到 MultipartReader 构造函数，即可开始执行读取逻辑，示例代码如下

```csharp
    var boundary = HeaderUtilities.RemoveQuotes(contentTypeBoundary).Value!;
    var multipartReader = new MultipartReader(boundary, request.Body, bufferSize: 1024);
```

读取的方式是写一个无限循环，直到 MultipartReader 的 ReadNextSectionAsync 返回空才退出循环，代码如下

```csharp
    while (true)
    {
        MultipartSection? multipartSection = await multipartReader.ReadNextSectionAsync();
        if (multipartSection == null)
        {
            // 读取完成了
            break;
        }

        ...
    }
```

当前读取到的 MultipartSection 还不能确定表单的类型，不知道是否包含文件。可继续通过 GetContentDispositionHeader 扩展方法服务获取 ContentDispositionHeaderValue 类型，再判断 IsFileDisposition 了解是否传入为文件，代码如下

```csharp
        ContentDispositionHeaderValue? contentDispositionHeaderValue = multipartSection.GetContentDispositionHeader();

        if (contentDispositionHeaderValue is null)
        {
            continue;
        }

        // ContentType=application/octet-stream
        // form-data; name="file"; filename="Input.zip"

        if (contentDispositionHeaderValue.IsFileDisposition())
        {
            FileMultipartSection? fileMultipartSection = multipartSection.AsFileSection();
            if (fileMultipartSection?.FileStream is null)
            {
                continue;
            }
            ...
        }
```

拿到了 FileMultipartSection 即可继续判断 Name 和 FileName 内容，比如说拿到 Foo1 的就来保存文件

示例代码如下，通过如下方式保存文件和使用上层的 IFormFile 的差别在于，如下方式可以直接从网络读取到本地文件，而 IFormFile 是先缓存到本地临时文件再做类似文件读取拷贝到目标本地文件的过程，相对来说如下方式耗费资源更低

```csharp
            // 可在此判断表单的各项内容。如判断是 Foo1 的就保存文件，是 TheFile 的就计算哈希值
            if (fileMultipartSection.Name == "Foo1")
            {
                // 文件
                var fileName = fileMultipartSection.FileName;
                fileName = GetSafeFileName(fileName);
                // 处理文件上传逻辑，例如保存文件
                // 这里简单地将文件保存到临时目录。小心，生产环境中请确保文件名安全，小心被攻击
                var filePath = Path.Join(Path.GetTempPath(), $"Uploaded_{fileName}");
                await using var fileStream = new FileStream(filePath, FileMode.Create, FileAccess.ReadWrite, FileShare.Read,
                    10240,
                    // 确保文件在关闭后被删除，以防止临时文件堆积。此仅仅为演示需求，避免临时文件太多。请根据你的需求决定是否使用此选项
                    FileOptions.DeleteOnClose);
                await fileMultipartSection.FileStream.CopyToAsync(fileStream);

                // 完成文件写入之后，可以通过以下代码，直接读取文件的内容
                fileStream.Position = 0;
                // 此时就可以立刻读取 FileStream 的内容了
                logger.LogInformation($"Received file '{fileName}', saved to '{filePath}'");
            }
```

以上示例代码里面设置了 FileOptions.DeleteOnClose 选项，仅仅只是为了演示，作用是确保文件在关闭之后自动删除，防止堆积测试文件

以上示例代码用到的 GetSafeFileName 方法我将在后文给出，详细请参阅 [C# 不能用于文件名的字符](https://blog.lindexi.com/post/C-%E4%B8%8D%E8%83%BD%E7%94%A8%E4%BA%8E%E6%96%87%E4%BB%B6%E5%90%8D%E7%9A%84%E5%AD%97%E7%AC%A6.html )

通过以上方式写文件还有一个优势是可以在 CopyToAsync 完成之后，立刻设置 Position 为 0 从而从零读取文件，立刻就能读取。整个过程发生在内存中的缓存占用非常低

按照本文的需求，是给文件做签名，连将文件写入磁盘的消耗都可以不用。对比 IFormFile 先缓存到本地临时文件的方式，如下直接读取立刻处理的方式可以做到更低的资源占用。如下方式基本上磁盘和内存都能做到非常平稳和非常低的水平。代码如下

```csharp
            if (fileMultipartSection.Name == "Foo1")
            {
                ...
            }
            else if (fileMultipartSection.Name == "TheFile")
            {
                using var sha1 = SHA1.Create();
                var hashByteList = await sha1.ComputeHashAsync(fileMultipartSection.FileStream);
                var hashString = Convert.ToHexString(hashByteList);
                logger.LogInformation($"Received file '{fileMultipartSection.FileName}', SHA1: {hashString}");
                await using var streamWriter = new StreamWriter(response.Body, leaveOpen: true);
                await streamWriter.WriteLineAsync($"Received file '{fileMultipartSection.FileName}', SHA1: {hashString}");
            }
```

如以上代码所示，此时直接 SHA1 计算从网络传输获取的数据，无需碰触磁盘读写。消耗的内存集中在缓存里面，缓存内存固定大小，总体损耗很低

在此方式里面，依然可以读取到普通的表单内容，如以下代码所示

```csharp
        if (contentDispositionHeaderValue.IsFileDisposition())
        {
            ...
        }
        else
        {
            // 普通表单字段
            var formMultipartSection = multipartSection.AsFormDataSection();
            if (formMultipartSection is null)
            {
                continue;
            }

            var name = formMultipartSection.Name;
            var value = await formMultipartSection.GetValueAsync();

            logger.LogInformation($"Received form field '{name}': {value}");
        }
```

看到这里，相信大家也就理解为什么那么多 CDN 厂商或 OSS 厂商都要求将文件放在表单末尾，这样可以方便他们的服务读取表单的开始就可以进行足够的校验，而不是接收了一个超大文件之后才能读取到可以校验的表单信息

完全的示例代码如下

```csharp
using Microsoft.AspNetCore.WebUtilities;

using System.IO;
using System.Net;
using System.Net.Mime;
using System.Net.Sockets;
using System.Security.Cryptography;
using System.Text;
using Microsoft.Net.Http.Headers;

var port = GetAvailablePort(IPAddress.Loopback);
var url = $"http://127.0.0.1:{port}";

_ = Task.Run(async () =>
{
    // 以下是测试代码
    using var httpClient = new HttpClient();

    using var multipartFormDataContent = new MultipartFormDataContent();
    using var fakeLongStream = new FakeLongStream();
    multipartFormDataContent.Add(new StreamContent(fakeLongStream), "TheFile", "FileName.zip");
    multipartFormDataContent.Add(new StringContent("Value1"), "Field1");
    var response = await httpClient.PostAsync($"{url}/PostMultipartForm", multipartFormDataContent);
    response.EnsureSuccessStatusCode();
    var responseContent = await response.Content.ReadAsStringAsync();
    Console.WriteLine($"{responseContent}");
});

var builder = WebApplication.CreateSlimBuilder(args);

builder.WebHost.UseKestrel(options =>
{
    // 无限制请求体大小
    // Microsoft.AspNetCore.Server.Kestrel.Core.BadHttpRequestException:“Request body too large. The max request body size is 30000000 bytes.”
    options.Limits.MaxRequestBodySize = null;
});

WebApplication app = builder.Build();
app.Urls.Add(url);

app.MapPost("/PostMultipartForm", async (Microsoft.AspNetCore.Http.HttpContext context) =>
{
    var logger = context.RequestServices.GetRequiredService<ILogger<Program>>();

    var request = context.Request;
    var response = context.Response;

    string? contentType = request.ContentType;
    if (contentType is null)
    {
        return;
    }

    MediaTypeHeaderValue mediaTypeHeaderValue = MediaTypeHeaderValue.Parse(contentType);
    var contentTypeBoundary = mediaTypeHeaderValue.Boundary;
    var boundary = HeaderUtilities.RemoveQuotes(contentTypeBoundary).Value!;
    var multipartReader = new MultipartReader(boundary, request.Body, 1024);

    await response.StartAsync();

    while (true)
    {
        MultipartSection? multipartSection = await multipartReader.ReadNextSectionAsync();
        if (multipartSection == null)
        {
            // 读取完成了
            break;
        }

        ContentDispositionHeaderValue? contentDispositionHeaderValue = multipartSection.GetContentDispositionHeader();

        if (contentDispositionHeaderValue is null)
        {
            continue;
        }

        // ContentType=application/octet-stream
        // form-data; name="file"; filename="Input.zip"

        if (contentDispositionHeaderValue.IsFileDisposition())
        {
            var fileMultipartSection = multipartSection.AsFileSection();
            if (fileMultipartSection?.FileStream is null)
            {
                continue;
            }

            // 可在此判断表单的各项内容。如判断是 Foo1 的就保存文件，是 TheFile 的就计算哈希值
            if (fileMultipartSection.Name == "Foo1")
            {
                // 文件
                var fileName = fileMultipartSection.FileName;
                fileName = GetSafeFileName(fileName);
                // 处理文件上传逻辑，例如保存文件
                // 这里简单地将文件保存到临时目录。小心，生产环境中请确保文件名安全，小心被攻击
                var filePath = Path.Join(Path.GetTempPath(), $"Uploaded_{fileName}");
                await using var fileStream = new FileStream(filePath, FileMode.Create, FileAccess.ReadWrite, FileShare.Read,
                    10240,
                    // 确保文件在关闭后被删除，以防止临时文件堆积。此仅仅为演示需求，避免临时文件太多。请根据你的需求决定是否使用此选项
                    FileOptions.DeleteOnClose);
                await fileMultipartSection.FileStream.CopyToAsync(fileStream);

                // 完成文件写入之后，可以通过以下代码，直接读取文件的内容
                fileStream.Position = 0;
                // 此时就可以立刻读取 FileStream 的内容了
                logger.LogInformation($"Received file '{fileName}', saved to '{filePath}'");
            }
            else if (fileMultipartSection.Name == "TheFile")
            {
                using var sha1 = SHA1.Create();
                var hashByteList = await sha1.ComputeHashAsync(fileMultipartSection.FileStream);
                var hashString = Convert.ToHexString(hashByteList);
                logger.LogInformation($"Received file '{fileMultipartSection.FileName}', SHA1: {hashString}");
                await using var streamWriter = new StreamWriter(response.Body, leaveOpen: true);
                await streamWriter.WriteLineAsync($"Received file '{fileMultipartSection.FileName}', SHA1: {hashString}");
            }
        }
        else
        {
            // 普通表单字段
            var formMultipartSection = multipartSection.AsFormDataSection();
            if (formMultipartSection is null)
            {
                continue;
            }

            var name = formMultipartSection.Name;
            var value = await formMultipartSection.GetValueAsync();

            logger.LogInformation($"Received form field '{name}': {value}");
        }
    }


    await response.CompleteAsync();
});

app.Run();

static string GetSafeFileName(string arbitraryString)
{
    var invalidChars = System.IO.Path.GetInvalidFileNameChars();
    var replaceIndex = arbitraryString.IndexOfAny(invalidChars, 0);
    if (replaceIndex == -1) return arbitraryString;

    var r = new StringBuilder();
    var i = 0;

    do
    {
        r.Append(arbitraryString, i, replaceIndex - i);

        switch (arbitraryString[replaceIndex])
        {
            case '"':
                r.Append("''");
                break;
            case '<':
                r.Append('\u02c2'); // '˂' (modifier letter left arrowhead)
                break;
            case '>':
                r.Append('\u02c3'); // '˃' (modifier letter right arrowhead)
                break;
            case '|':
                r.Append('\u2223'); // '∣' (divides)
                break;
            case ':':
                r.Append('-');
                break;
            case '*':
                r.Append('\u2217'); // '∗' (asterisk operator)
                break;
            case '\\':
            case '/':
                r.Append('\u2044'); // '⁄' (fraction slash)
                break;
            case '\0':
            case '\f':
            case '?':
                break;
            case '\t':
            case '\n':
            case '\r':
            case '\v':
                r.Append(' ');
                break;
            default:
                r.Append('_');
                break;
        }

        i = replaceIndex + 1;
        replaceIndex = arbitraryString.IndexOfAny(invalidChars, i);
    } while (replaceIndex != -1);

    r.Append(arbitraryString, i, arbitraryString.Length - i);

    return r.ToString();
}

static int GetAvailablePort(IPAddress ip)
{
    using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
    socket.Bind(new IPEndPoint(ip, 0));
    socket.Listen(1);
    var ipEndPoint = (IPEndPoint) socket.LocalEndPoint!;
    var port = ipEndPoint.Port;
    return port;
}

class FakeLongStream : Stream
{
    public override void Flush()
    {
        throw new NotImplementedException();
    }

    public override int Read(byte[] buffer, int offset, int count)
    {
        if (Position == Length)
        {
            return 0;
        }

        Position += count;

        Random.Shared.NextBytes(buffer.AsSpan(offset, count));

        if (Position < Length)
        {
            return count;
        }

        var result = (int) (Length - (Position - count));
        Position = Length;
        return result;
    }

    public override long Seek(long offset, SeekOrigin origin)
    {
        throw new NotImplementedException();
    }

    public override void SetLength(long value)
    {
        throw new NotImplementedException();
    }

    public override void Write(byte[] buffer, int offset, int count)
    {
        throw new NotImplementedException();
    }

    public override bool CanRead => true;
    public override bool CanSeek => false;
    public override bool CanWrite => false;
    public override long Length => int.MaxValue / 2;
    public override long Position { get; set; }
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/6b9f5ce59f0159e8e87c073db57ab62e12adecc5/Workbench/NujawfeafuKeekenercekiji) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/6b9f5ce59f0159e8e87c073db57ab62e12adecc5/Workbench/NujawfeafuKeekenercekiji) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 6b9f5ce59f0159e8e87c073db57ab62e12adecc5
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 6b9f5ce59f0159e8e87c073db57ab62e12adecc5
```

获取代码之后，进入 Workbench/NujawfeafuKeekenercekiji 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

参考文档：

- https://github.com/dotnet/aspnetcore/issues/58233
- https://learn.microsoft.com/en-us/aspnet/core/mvc/models/file-uploads?view=aspnetcore-9.0
