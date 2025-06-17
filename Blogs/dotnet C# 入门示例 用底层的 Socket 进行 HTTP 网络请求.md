---
title: dotnet C# 入门示例 用底层的 Socket 进行 HTTP 网络请求
description: 本文属于基础入门博客，将和大家介绍如何在 dotnet C# 代码里面使用底层的 Socket 进行 HTTP 网络请求
tags: dotnet,C#
category: 
---

<!-- CreateTime:2025/06/17 07:05:23 -->

<!-- 发布 -->
<!-- 博客 -->

本文将使用向百度发送 HTTP 和 HTTPS 请求作为示例，来和大家样式如何使用底层的 Socket 进行 HTTP 和 HTTPS 网络请求

本文开始之前，希望大家对基础的网络知识有所了解

回顾进行 HTTP 请求的基本流程。拿到 https://www.baidu.com 之后，需要先经过 DNS 解析，获取到对应的 IP 地址。接着通过 TCP 协议和服务器建立连接，发送 HTTP 请求内容，等待服务器响应内容

尽管通过 Socket 也能通过 DnsEndPoint 进行网络请求，但是为了尽可能展示更多的细节，本文将调用 `Dns.GetHostAddressesAsync` 方法对域名进行解析，获取到对应的 IP 地址。接着通过 `Socket` 类和服务器建立连接，发送 HTTP 请求内容，等待服务器响应内容

调用 `Dns.GetHostAddressesAsync` 对给定的域名进行解析的代码如下。本文内容里面只给出关键代码片段，如需要全部的项目文件，可到本文末尾找到本文所有代码的下载方法

```csharp
// 一个域名可以有多个 IP 地址，利用此特性，可以实现 IP 级域名备份，也能利用此特性实现寻找距离自己最近的 IP 地址
IPAddress[] ipAddresses = await Dns.GetHostAddressesAsync("www.baidu.com");
```

在我当前的网络环境下，能够获取到两个百度的地址。一般而言，取其首个满足条件的 IP 地址即可。有些时候需要禁用 Ip v6 地址的情况，则请大家自行判断和处理，本文不再赘述

额外说明一点，从 `Dns.GetHostAddressesAsync` 里面获取到的 IP 地址是不保证顺序的。从 DNS 提供商的角度上讲，本身也是不保证顺序的，大家可以试试去找一些 CDN 厂商的域名试试看。为什么说去找 CDN 厂商的域名呢？因为 CDN 厂商的域名一般都是有多个 IP 地址的。大家可以试试在 CMD 命令行里面不断输入 `ipconfig /flushdns` 命令来清除 DNS 缓存，然后再调用 `Dns.GetHostAddressesAsync` 获取 IP 地址，测试每次获取到的地址是否顺序相同

为了演示方便，我将遍历百度的所有 IP 地址，依次进行 HTTP 请求。进行 HTTP 请求时，获取到了 IP 地址，就可以开始使用 `Socket` 类进行连接了。在连接之前，先从共享池中租用一个字节数组，为 1MB 大小。网络请求过程中也是可以做到低分配的，合理使用内存池可以减少 GC 压力。代码如下

```csharp
var buffer = ArrayPool<byte>.Shared.Rent(1024 * 1024);

try
{
    foreach (var ipAddress in ipAddresses)
    {
        ... // 忽略其他代码
    }
}
finally
{
    // 别忘了归还共享池中的字节数组
    ArrayPool<byte>.Shared.Return(buffer);
}
```

开始创建 Socket 对象，创建完成之后，调用 `ConnectAsync` 方法进行连接。默认情况下的 HTTP 应该连接的是 80 端口

```csharp
    foreach (var ipAddress in ipAddresses)
    {
        Console.WriteLine($"开始连接 IP:{ipAddress}");

        using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
        await socket.ConnectAsync(ipAddress, 80);

        ... // 忽略其他代码
    }
```

预期能够连接成功，此时就完成了 HTTP 连接中的 TCP 建立过程了。下一步是发送 HTTP 请求内容。这里的请求内容是一个简单的 HTTP GET 请求。在 dotnet 里面封装了 NetworkStream 类，可以简化一些网络通讯代码。可以像一个 Stream 一样进行网络通讯，整个代码写起来还是比较舒服的。咱就在此基础上构建一个 NetworkStream 对象，然后通过 Stream 的方式写入请求内容，注：后面要带两个换行哦。代码如下

```csharp
        using var networkStream = new NetworkStream(socket);

        Console.WriteLine($"连接完成，开始发送请求");

        // 这里的请求内容是一个简单的 HTTP GET 请求，注：后面要带两个换行哦
        ReadOnlySpan<byte> content = """
                                     GET https://www.baidu.com HTTP/1.1
                                     Host: www.baidu.com


                                     """u8;
        content.CopyTo(buffer); // 将请求内容复制到租用的字节数组中。异步请求不能传入 Span 类型，只能传入 Memory 类型。将 Span 转换为 Memory 的方式是先写入到 buffer 中，然后再将其当成 ReadOnlyMemory 或 Memory 类型
        ReadOnlyMemory<byte> writeBuffer = buffer.AsMemory(0, content.Length);
        await networkStream.WriteAsync(writeBuffer);
```

写入之后，依然可以通过 NetworkStream 读取百度服务器响应的内容，代码如下

```csharp
        // 读取响应内容
        var length = await networkStream.ReadAsync(buffer); // 注： 很多时候，这里都是没有完全读取到完整的响应内容的，可能需要多次读取才能获取完整的响应内容。读取多长的数据需要从返回的 Header 里面获取 Content-Length 字段的值
        var text = Encoding.UTF8.GetString(buffer, 0, length);

        Console.WriteLine($"收到百度的响应内容。内容长度 {length}，内容摘要：{text.Substring(0, Math.Min(50, text.Length))}...");

        Console.WriteLine();
```

以上的代码没有去读取 Content-Length 字段的值，也没有处理一次性读取没有读取完的问题。如果大家想要制作一个合理的 HTTP 请求客户端，则需要自行处理这些问题。本文只是一个简单的示例，演示如何使用底层的 Socket 进行 HTTP 请求，没有处理这些细节

以上代码就完成了 HTTP 的请求了。那么如何进行 HTTPS 的请求呢？其实和 HTTP 的请求是类似的，只不过需要在 NetworkStream 的基础上，使用 SslStream 进行封装。依然是和 HTTP 一样的前置过程，获取 IP 地址，创建 Socket 对象，连接到服务器。相同部分的代码如下，当然别忘了改一下端口号哦

```csharp
        using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
        await socket.ConnectAsync(ipAddress, 443); // 443 是 HTTPS 的默认端口

        using var networkStream = new NetworkStream(socket);
```

完成连接之后，接下来就是其不同的部分。不同的是，连接到服务器之后，需要进行 SSL/TLS 握手，使用 SslStream 包装 NetworkStream，然后进行认证。接下来后续的所有读写操作都将通过 SslStream 进行。代码如下

```csharp
        // 进行 SSL/TLS 握手，使用 SslStream 包装 NetworkStream 然后进行认证
        // 接下来后续的所有读写操作都将通过 SslStream 进行
        using var sslStream = new SslStream(networkStream);
        await sslStream.AuthenticateAsClientAsync("www.baidu.com");
```

在 dotnet 里面，无论是 `NetworkStream` 还是 `SslStream` 都是继承自 `Stream` 类的，所以可以使用相同的方式进行读写操作。接下来就是发送请求内容了，和 HTTP 的请求内容一样，只不过需要使用 SslStream 来发送请求内容。代码如下

```csharp
        // 这里的请求内容是一个简单的 HTTP GET 请求，注：后面要带两个换行哦
        ReadOnlySpan<byte> content = """
                                     GET https://www.baidu.com HTTP/1.1
                                     Host: www.baidu.com


                                     """u8;
        content.CopyTo(buffer); // 将请求内容复制到租用的字节数组中。异步请求不能传入 Span 类型，只能传入 Memory 类型。将 Span 转换为 Memory 的方式是先写入到 buffer 中，然后再将其当成 ReadOnlyMemory 或 Memory 类型
        ReadOnlyMemory<byte> writeBuffer = buffer.AsMemory(0, content.Length);
        await sslStream.WriteAsync(writeBuffer); // 这里要用 SslStream 来发送请求内容
```

可以对比一下 HTTP 请求的代码，只会发现 WriteAsync 的对象从 `NetworkStream` 类型换成 `SslStream` 类型而已

读取百度响应的代码也是类似，代码如下

```csharp
        // 读取响应内容
        var length = await sslStream.ReadAsync(buffer); // 这里要用 SslStream 来读取响应内容
        var text = Encoding.UTF8.GetString(buffer, 0, length);

        Console.WriteLine($"收到百度的响应内容。内容长度 {length}，内容摘要：{text.Substring(0, Math.Min(50, text.Length))}...");
```

至少在 dotnet 的封装基础之下，即使使用十分底层的 Socket 方式进行 HTTP 或 HTTPS 通讯，其代码也是十分简洁的。以上就是本文的全部演示内容了，网络通讯是一个知识量比较庞大的领域，本文只是一个简单的示例，演示如何使用底层的 Socket 进行 HTTP 和 HTTPS 网络请求。希望能够对大家有所帮助

全部的 Program.cs 文件的代码如下

```csharp
// See https://aka.ms/new-console-template for more information

using System.Buffers;
using System.Net;
using System.Net.Security;
using System.Net.Sockets;
using System.Text;

// 一个域名可以有多个 IP 地址，利用此特性，可以实现 IP 级域名备份，也能利用此特性实现寻找距离自己最近的 IP 地址
IPAddress[] ipAddresses = await Dns.GetHostAddressesAsync("www.baidu.com");

// 从共享池中租用一个字节数组，大小为 1MB
// 网络请求过程中也是可以做到低分配的，合理使用内存池可以减少 GC 压力
var buffer = ArrayPool<byte>.Shared.Rent(1024 * 1024);

try
{
    foreach (var ipAddress in ipAddresses)
    {
        Console.WriteLine($"开始连接 IP:{ipAddress}");

        using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
        await socket.ConnectAsync(ipAddress, 80);

        using var networkStream = new NetworkStream(socket);

        Console.WriteLine($"连接完成，开始发送请求");

        // 这里的请求内容是一个简单的 HTTP GET 请求，注：后面要带两个换行哦
        ReadOnlySpan<byte> content = """
                                     GET https://www.baidu.com HTTP/1.1
                                     Host: www.baidu.com


                                     """u8;
        content.CopyTo(buffer); // 将请求内容复制到租用的字节数组中。异步请求不能传入 Span 类型，只能传入 Memory 类型。将 Span 转换为 Memory 的方式是先写入到 buffer 中，然后再将其当成 ReadOnlyMemory 或 Memory 类型
        ReadOnlyMemory<byte> writeBuffer = buffer.AsMemory(0, content.Length);
        await networkStream.WriteAsync(writeBuffer);

        // 读取响应内容
        var length = await networkStream.ReadAsync(buffer); // 注： 很多时候，这里都是没有完全读取到完整的响应内容的，可能需要多次读取才能获取完整的响应内容。读取多长的数据需要从返回的 Header 里面获取 Content-Length 字段的值
        var text = Encoding.UTF8.GetString(buffer, 0, length);

        Console.WriteLine($"收到百度的响应内容。内容长度 {length}，内容摘要：{text.Substring(0, Math.Min(50, text.Length))}...");

        Console.WriteLine();
    }
}
finally
{
    // 别忘了归还共享池中的字节数组
    ArrayPool<byte>.Shared.Return(buffer);
}

// 以下是进行 https 请求的代码
// 上面的缓存已经被归还了，为了继续使用，就开始重新申请好了
buffer = ArrayPool<byte>.Shared.Rent(1024 * 1024);

try
{
    foreach (var ipAddress in ipAddresses)
    {
        Console.WriteLine($"开始连接 IP:{ipAddress}");

        using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
        await socket.ConnectAsync(ipAddress, 443); // 443 是 HTTPS 的默认端口

        using var networkStream = new NetworkStream(socket);

        Console.WriteLine($"连接完成，开始进行 https 通讯");

        // 进行 SSL/TLS 握手，使用 SslStream 包装 NetworkStream 然后进行认证
        // 接下来后续的所有读写操作都将通过 SslStream 进行
        using var sslStream = new SslStream(networkStream);
        await sslStream.AuthenticateAsClientAsync("www.baidu.com");

        Console.WriteLine($"开始发送请求");

        // 这里的请求内容是一个简单的 HTTP GET 请求，注：后面要带两个换行哦
        ReadOnlySpan<byte> content = """
                                     GET https://www.baidu.com HTTP/1.1
                                     Host: www.baidu.com


                                     """u8;
        content.CopyTo(buffer); // 将请求内容复制到租用的字节数组中。异步请求不能传入 Span 类型，只能传入 Memory 类型。将 Span 转换为 Memory 的方式是先写入到 buffer 中，然后再将其当成 ReadOnlyMemory 或 Memory 类型
        ReadOnlyMemory<byte> writeBuffer = buffer.AsMemory(0, content.Length);
        await sslStream.WriteAsync(writeBuffer); // 这里要用 SslStream 来发送请求内容

        // 读取响应内容
        var length = await sslStream.ReadAsync(buffer); // 这里要用 SslStream 来读取响应内容
        var text = Encoding.UTF8.GetString(buffer, 0, length);

        Console.WriteLine($"收到百度的响应内容。内容长度 {length}，内容摘要：{text.Substring(0, Math.Min(50, text.Length))}...");

        Console.WriteLine();
    }
}
finally
{
    // 别忘了归还共享池中的字节数组
    ArrayPool<byte>.Shared.Return(buffer);
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/cb55daad2c8e5725e7b780261939bf5728e5323a/Workbench/HalaiheakerbarheeLayneberqarke) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/cb55daad2c8e5725e7b780261939bf5728e5323a/Workbench/HalaiheakerbarheeLayneberqarke) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cb55daad2c8e5725e7b780261939bf5728e5323a
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin cb55daad2c8e5725e7b780261939bf5728e5323a
```

获取代码之后，进入 Workbench/HalaiheakerbarheeLayneberqarke 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
