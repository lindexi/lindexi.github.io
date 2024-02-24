# Pipelines.Sockets.Unofficial 一个纯托管实现对接 System.IO.Pipelines 的 Sockets 库

本文将和大家介绍 Pipelines.Sockets.Unofficial 这个由纯托管代码实现的，对接了 System.IO.Pipelines 的 Sockets 库。这个库不仅代码性能高，且上层调用的 API 足够简洁

<!--more-->
<!-- CreateTime:2024/2/23 21:06:28 -->

<!-- 发布 -->
<!-- 博客 -->

本文介绍的 Pipelines.Sockets.Unofficial 库是在 GitHub 上使用最友好的 MIT 协议开源的项目，详细请参阅 <https://github.com/mgravell/Pipelines.Sockets.Unofficial>

本文介绍的 Pipelines.Sockets.Unofficial 库是对 Socket 进行足够裸的封装，可以作为在没有 dotnet 官方实现 Socket 与 System.IO.Pipelines 对接之前的替代方式

在 Pipelines.Sockets.Unofficial 库里面同时实现了服务端和客户端的支持，意味着可以使用 Pipelines.Sockets.Unofficial 开启 Socket 服务，也可以使用 Pipelines.Sockets.Unofficial 连接别的现有的 Socket 服务

阅读本文之前，我期望读者已经了解了 System.IO.Pipelines 的作用以及其高性能低内存压力的原理。如尚未了解，我推荐先阅读 [System.IO.Pipelines: High performance IO in .NET - .NET Blog](https://devblogs.microsoft.com/dotnet/system-io-pipelines-high-performance-io-in-net/ ) 和 [System.IO.Pipelines——高性能IO(一) - yswenli - 博客园](https://www.cnblogs.com/yswenli/p/11810317.html ) 博客之后再来阅读本文

本文将通过一个简单的例子用来演示如何使用 Pipelines.Sockets.Unofficial 库，期望从使用的角度上可以让大家看到 Pipelines.Sockets.Unofficial 的优势

按照 dotnet 的惯例，先通过 NuGet 安装 Pipelines.Sockets.Unofficial 库

为了简单方便，本文的例子将只是一个在单个进程之内完成服务端和客户端代码的编写

在 Pipelines.Sockets.Unofficial 里面，默认提供的 SocketServer 是一个抽象类，需要继承实现。继承实现时，需要实现 OnClientConnectedAsync 方法，在这个方法里面将处理与连接进来的客户端的通讯逻辑

```csharp
class FooSocketServer : SocketServer
{
    protected override Task OnClientConnectedAsync(in ClientConnection client)
    {
        Console.WriteLine($"收到客户端 {client.RemoteEndPoint} 连接");
        return DoFooAsync(client);
    }
}
```

以上代码先省略 DoFooAsync 的实现代码，先将整体的框架代码和大家演示，再来完善具体的细节逻辑

开启服务端之前，需要获取好一个可用的端口，本文将使用 [dotnet C# 获取一个可用的端口的方法](https://blog.lindexi.com/post/dotnet-C-%E8%8E%B7%E5%8F%96%E4%B8%80%E4%B8%AA%E5%8F%AF%E7%94%A8%E7%9A%84%E7%AB%AF%E5%8F%A3%E7%9A%84%E6%96%B9%E6%B3%95.html ) 博客提供的方法获取一个可用的端口，代码如下

```csharp
var availablePort = GetAvailablePort(IPAddress.Loopback);

static int GetAvailablePort(IPAddress ip)
{
    using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
    socket.Bind(new IPEndPoint(ip, 0));
    socket.Listen(1);
    var ipEndPoint = (IPEndPoint) socket.LocalEndPoint!;
    var port = ipEndPoint.Port;
    return port;
}
```

获取到可用端口之后，即可创建和开启服务端，如以下代码

```csharp
var availablePort = GetAvailablePort(IPAddress.Loopback);
var endPoint = new IPEndPoint(IPAddress.Loopback, availablePort);

// 创建服务端
using var fooSocketServer = new FooSocketServer();
fooSocketServer.Listen(endPoint);
```

而创建 Pipelines.Sockets.Unofficial 的客户端则是通过 SocketConnection.ConnectAsync 静态方法，连接一个可用终点的方式

```csharp
var socketConnection = await SocketConnection.ConnectAsync(endPoint);
```

以上代码就通过 Pipelines.Sockets.Unofficial 创建服务端，以及对应的使用客户端完成连接的逻辑。如此可以看到 API 相对来说还是设计的很好的

完成了连接之后，接下来就是相互之间的通讯。先来实现服务端的测试代码逻辑，在服务端将实现接收客户端发过来的消息，将客户端发过来的消息打印到控制台，再返回服务端的消息给到客户端，实现的代码逻辑如下

```csharp
class FooSocketServer : SocketServer
{
    protected override Task OnClientConnectedAsync(in ClientConnection client)
    {
        Console.WriteLine($"收到客户端 {client.RemoteEndPoint} 连接");
        return DoFooAsync(client);
    }

    private async Task DoFooAsync(ClientConnection client)
    {
        for (int i = 0; i < int.MaxValue; i++)
        {
            var readResult = await client.Transport.Input.ReadAsync();

            var inputText = Encoding.UTF8.GetString(readResult.Buffer);
            Console.WriteLine($"[服务端] 收到客户端发送的 {inputText}");

            var memory = client.Transport.Output.GetMemory(1024);
            var length = Encoding.UTF8.GetBytes($"{i} 这是来自服务端的消息".AsSpan(), memory.Span);
            client.Transport.Output.Advance(length);
            await client.Transport.Output.FlushAsync();

            // 标记已处理的数据
            client.Transport.Input.AdvanceTo(readResult.Buffer.End);

            if (readResult.IsCompleted)
            {
                break;
            }
        }
    }
}
```

以上的代码将使用到 System.IO.Pipelines 提供的 PipeReader 和 PipeWriter 分别进行读取客户端发送过来的消息以及将服务端的消息发送给到客户端

先使用 `var readResult = await client.Transport.Input.ReadAsync();` 读取到客户端发送过来的一次消息。读取消费完成之后，需要标记已处理的数据，即调用 `client.Transport.Input.AdvanceTo(readResult.Buffer.End);` 告诉 System.IO.Pipelines 框架层本次数据已消费完成，防止重复消费

将服务端的消息发送给到客户端是先通过 `var memory = client.Transport.Output.GetMemory(1024);` 获取到 System.IO.Pipelines 框架提供的一段内存，再使用 `Encoding.UTF8.GetBytes` 将字符串编码写入到这段内存里面。最后分别通过 `client.Transport.Output.Advance(length);` 和 `await client.Transport.Output.FlushAsync();` 告知框架层写入的长度以及将数据刷入发送到客户端

这里有一个细节是根据 [官方文档](https://learn.microsoft.com/en-us/dotnet/api/system.io.pipelines.pipewriter.getmemory?view=dotnet-plat-ext-8.0&viewFallbackFrom=net-9.0 ) 说明，每次都应该使用 GetMemory 获取 `Memory<byte>` 内存，不能进行复用，因为之前的 Memory 对象已经被标记为已写入的数据

以上就是服务端在收到客户端连接时，对客户端执行的处理逻辑

在本文的测试代码里面，将在客户端对服务端进行连接，连接完成之后将立刻发送一条消息给到服务端，且读取服务端的响应

```csharp
// 发送消息
// 从 PipeWriter 里面获取一个 Memory 对象，用来写入数据
Memory<byte> memory = socketConnection.Output.GetMemory(1024);
// 将字符串编码成字节，写入 Memory 对象
var length = Encoding.UTF8.GetBytes("这是来自客户端的消息".AsSpan(), memory.Span);
// 标记已写入的数据的长度
socketConnection.Output.Advance(length);
// 将写入的数据发送出去
await socketConnection.Output.FlushAsync();

// 从 PipeReader 里面读取数据
var readResult = await socketConnection.Input.ReadAsync();
Console.WriteLine($"[客户端] 收到服务端端回复的 {Encoding.UTF8.GetString(readResult.Buffer)}");
// 标记已处理的数据的长度，下次读取的时候会跳过这些数据
socketConnection.Input.AdvanceTo(readResult.Buffer.End);
```

运行代码，预期将可以看到大概如下的控制台输出内容

```
收到客户端 127.0.0.1:29533 连接
[服务端] 收到客户端发送的 这是来自客户端的消息
[客户端] 收到服务端端回复的 0 这是来自服务端的消息
```

再继续编写客户端的逻辑，让客户端可以读取控制台输入内容，将控制台输入内容发送给到服务端，且读取服务端的回复消息

```csharp
while (true)
{
    // 从控制台读取输入，将输入的内容发送给服务端
    var line = Console.ReadLine();
    // 重新从 PipeWriter 里面获取一个 Memory 对象，用来写入数据。不能用之前的 Memory 对象，因为之前的 Memory 对象已经被标记为已写入的数据
    // https://learn.microsoft.com/en-us/dotnet/api/system.io.pipelines.pipewriter.getmemory
    // You must request a new buffer after calling Advance to continue writing more data; you cannot write to a previously acquired buffer.
    memory = socketConnection.Output.GetMemory(1024);
    length = Encoding.UTF8.GetBytes(line.AsSpan(), memory.Span);
    socketConnection.Output.Advance(length);
    var flushResult = await socketConnection.Output.FlushAsync();
    if (flushResult.IsCompleted)
    {
        break;
    }

    readResult = await socketConnection.Input.ReadAsync();
    Console.WriteLine($"[客户端] 收到服务端端回复的 {Encoding.UTF8.GetString(readResult.Buffer)}");
    socketConnection.Input.AdvanceTo(readResult.Buffer.End);
}

Console.Read();
```

完成以上代码，大家可以尝试运行项目，在控制台随意输入内容，测试一下服务端是否能够收到客户端发送的消息，且客户端也能正确收到来自服务端回复的内容

以上的测试代码不仅开放了进程内的访问逻辑，同时也可以测试一下本机内的其他进程的访问，比如使用 [HttpRepl](https://blog.lindexi.com/post/HttpRepl-%E4%BA%92%E6%93%8D%E4%BD%9C%E7%9A%84-RESTful-HTTP-%E6%9C%8D%E5%8A%A1%E8%B0%83%E8%AF%95%E5%91%BD%E4%BB%A4%E8%A1%8C%E5%B7%A5%E5%85%B7.html ) 工具进行访问。测试方法如下

运行测试项目，先在 GetAvailablePort 方法打上断点，记录所获取到的可用端口号，如我这里获取的是 29535 端口。接着继续按下 F5 让 VisualStudio 继续执行代码。在 HttpRepl 使用以下命令进行连接

```
get http://127.0.0.1:29535
```

输入以上命令之后，预期在测试项目里面看到控制台有如下输出内容

```
收到客户端 127.0.0.1:50621 连接
[服务端] 收到客户端发送的 GET / HTTP/1.1
Host: 127.0.0.1:29535
User-Agent: HTTP-REPL
```

通过以上的控制台输出可以了解到 HttpRepl 可以建立连接且测试项目接收到请求的消息。只是因为当前的服务端没有实现 http 协议，导致 HttpRepl 工具无法读取到任何有效的响应信息

如果期望能够让 HttpRepl 工具可以读取到有效的响应信息，则需要改造现有的测试代码，让服务端按照 http 协议返回消息。这部分如果大家感兴趣还请自行试试编写，本文这里就跳过这个细节部分

接下来可以再使用浏览器进行测试，以下是我使用火狐浏览器，在地址栏输入 http://127.0.0.1:29535 时，测试项目控制台的输出内容

```
收到客户端 127.0.0.1:50709 连接
[服务端] 收到客户端发送的 GET / HTTP/1.1
Host: 127.0.0.1:29535
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:122.0) Gecko/20100101 Firefox/122.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
DNT: 1
Connection: keep-alive
Cookie: lang=zh-CN
Upgrade-Insecure-Requests: 1
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
```

如此可以看到使用 Pipelines.Sockets.Unofficial 库做一些简单的测试，或者是作为底层的框架再在此之上封装自己的业务框架还是一个不错的方式 

以下是本文的全部代码

```csharp
using System.Buffers;
using Pipelines.Sockets.Unofficial;

using System;
using System.Net.Sockets;
using System.Net;
using System.Text;

var availablePort = GetAvailablePort(IPAddress.Loopback);
var endPoint = new IPEndPoint(IPAddress.Loopback, availablePort);

// 创建服务端
using var fooSocketServer = new FooSocketServer();
fooSocketServer.Listen(endPoint);

Console.Read();

var socketConnection = await SocketConnection.ConnectAsync(endPoint);

// 发送消息
// 从 PipeWriter 里面获取一个 Memory 对象，用来写入数据
Memory<byte> memory = socketConnection.Output.GetMemory(1024);
// 将字符串编码成字节，写入 Memory 对象
var length = Encoding.UTF8.GetBytes("这是来自客户端的消息".AsSpan(), memory.Span);
// 标记已写入的数据的长度
socketConnection.Output.Advance(length);
// 将写入的数据发送出去
await socketConnection.Output.FlushAsync();

// 从 PipeReader 里面读取数据
var readResult = await socketConnection.Input.ReadAsync();
Console.WriteLine($"[客户端] 收到服务端端回复的 {Encoding.UTF8.GetString(readResult.Buffer)}");
// 标记已处理的数据的长度，下次读取的时候会跳过这些数据
socketConnection.Input.AdvanceTo(readResult.Buffer.End);

while (true)
{
    // 从控制台读取输入，将输入的内容发送给服务端
    var line = Console.ReadLine();
    // 重新从 PipeWriter 里面获取一个 Memory 对象，用来写入数据。不能用之前的 Memory 对象，因为之前的 Memory 对象已经被标记为已写入的数据
    // https://learn.microsoft.com/en-us/dotnet/api/system.io.pipelines.pipewriter.getmemory
    // You must request a new buffer after calling Advance to continue writing more data; you cannot write to a previously acquired buffer.
    memory = socketConnection.Output.GetMemory(1024);
    length = Encoding.UTF8.GetBytes(line.AsSpan(), memory.Span);
    socketConnection.Output.Advance(length);
    var flushResult = await socketConnection.Output.FlushAsync();
    if (flushResult.IsCompleted)
    {
        break;
    }

    readResult = await socketConnection.Input.ReadAsync();
    Console.WriteLine($"[客户端] 收到服务端端回复的 {Encoding.UTF8.GetString(readResult.Buffer)}");
    socketConnection.Input.AdvanceTo(readResult.Buffer.End);
}

Console.Read();

static int GetAvailablePort(IPAddress ip)
{
    using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
    socket.Bind(new IPEndPoint(ip, 0));
    socket.Listen(1);
    var ipEndPoint = (IPEndPoint) socket.LocalEndPoint!;
    var port = ipEndPoint.Port;
    return port;
}

class FooSocketServer : SocketServer
{
    protected override Task OnClientConnectedAsync(in ClientConnection client)
    {
        Console.WriteLine($"收到客户端 {client.RemoteEndPoint} 连接");
        return DoFooAsync(client);
    }

    private async Task DoFooAsync(ClientConnection client)
    {
        for (int i = 0; i < int.MaxValue; i++)
        {
            var readResult = await client.Transport.Input.ReadAsync();

            var inputText = Encoding.UTF8.GetString(readResult.Buffer);
            Console.WriteLine($"[服务端] 收到客户端发送的 {inputText}");

            var memory = client.Transport.Output.GetMemory(1024);
            var length = Encoding.UTF8.GetBytes($"{i} 这是来自服务端的消息".AsSpan(), memory.Span);
            client.Transport.Output.Advance(length);
            await client.Transport.Output.FlushAsync();

            // 标记已处理的数据
            client.Transport.Input.AdvanceTo(readResult.Buffer.End);

            if (readResult.IsCompleted)
            {
                break;
            }
        }
    }
}
```

代码项目放在 [github](https://github.com/lindexi/lindexi_gd/tree/a14d3bce4d0a0e4b932b61b9f5d7d8fd3b9d399a/KarwanufallnalKeajealikaqere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/a14d3bce4d0a0e4b932b61b9f5d7d8fd3b9d399a/KarwanufallnalKeajealikaqere) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin a14d3bce4d0a0e4b932b61b9f5d7d8fd3b9d399a
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin a14d3bce4d0a0e4b932b61b9f5d7d8fd3b9d399a
```

获取代码之后，进入 KarwanufallnalKeajealikaqere 文件夹，即可获取到源代码

参考文档：

[DocsStaging/Pipelines.md at master · davidfowl/DocsStaging](https://github.com/davidfowl/DocsStaging/blob/master/Pipelines.md )

[PipeWriter.GetMemory(Int32) Method (System.IO.Pipelines) Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/api/system.io.pipelines.pipewriter.getmemory?view=dotnet-plat-ext-8.0&viewFallbackFrom=net-9.0 )

[System.IO.Pipelines——高性能IO(一) - yswenli - 博客园](https://www.cnblogs.com/yswenli/p/11810317.html )