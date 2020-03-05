# dot net core 使用 IPC 进程通信

本文告诉大家如何使用dot net core 和其他进程进行通信。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:52 -->

<!-- 标签: dnc , 进程通信，IPC，pipe -->

一般都是使用 WCF 或 remoting 做远程通信，但是 dot net core 不支持 WCF 所以暂时我就只能使用 管道通信。

## 原理

管道通信使用的是 Pipe 需要启动一对服务器和客户端才可以使用。在 NamedPipeServerStream 启动之后可以接受其他 NamedPipeClientStream 连接。

因为现在已经使用了 await 了，所以建议全部都可以写异步，古老的程序员喜欢使用回调，但是现在的程序员还是建议使用 await 比较好，因为比较容易用。

创建的 NamedPipeServerStream 需要告诉管道的命名，和通信方式，通信可以分为单向和双向，大家使用枚举去看一下就可以知道。我来创建一个管道名是`lindexi`，可以双向通信的管道。

```csharp
            var pipe = new NamedPipeServerStream("lindexi", PipeDirection.InOut);

```

上面代码就创建了一个管道，之后需要等待有连接才可以发送数据。

```csharp
            await pipe.WaitForConnectionAsync();
```

等待了连接之后，就可以发送数据，发生的数据使用的是字节发送，所以需要转换编码。

```csharp
            string str = "发送消息";

            var spxnfSrxldhhv = Encoding.UTF8.GetBytes(str);

            pipe.Write(spxnfSrxldhhv, 0, spxnfSrxldhhv.Length);
```

注意，需要指定发送数据的长度和数据，通过这个方法发送是不建议的。

如果需要使用一个比较高级的方法传输，请看文章最后

这时另一个程序就需要下面代码连接

```csharp
            var pipe = new NamedPipeClientStream(".", "lindexi", PipeDirection.InOut, PipeOptions.None);

            pipe.Connect()
```

上面代码使用的 NamedPipeClientStream 需要指定管道的命名才可以找到。

连接之后可以通过这个方式读取数据

```csharp
            var spxnfSrxldhhv = new byte[65535];

            var n = pipe.Read(spxnfSrxldhhv, 0, spxnfSrxldhhv.Length);

            var str = Encoding.UTF8.GetString(spxnfSrxldhhv, 0, n);
```

对于读写数据很多时候就使用 pipe 的读写，写入字节，读出字节。

因为一次读取可能会卡很多时间，所以建议使用异步读。

如果觉得每次发送都需要转 byte 然后进行写，代码很多，可以使用下面的代码

```csharp
            var stream = new StreamWriter(pipe);
            
            stream.Write("发送消息");
            
            stream.Flush();
```

注意不要使用下面的代码

```csharp
            using (var stream = new StreamWriter(pipe))
            {
                stream.Write("发送消息");
            }
```

原因是 using 会关闭 pipe 所以使用之后就不能在写。

## 例子

首先创建两个程序，一个是 WPF 程序 DgvlzKixtdin ，另一个是 dot net core 控制台 HclkvyDanuiag 。接着需要从 DgvlzKixtdin 发送数据，从 HclkvyDanuiag 接收数据。

在 WPF 程序添加一个 TextBlock 和 Button ，点击 Button 就发送消息到 dot net core 程序。收到消息就在 TextBlock 显示。

在 Button 点击的代码写下面代码

```csharp
      private async void HixKkikjgp_OnClick(object sender, RoutedEventArgs e)
        {
            var pipe = new NamedPipeServerStream("lindexi", PipeDirection.InOut);

            await pipe.WaitForConnectionAsync();

            string str = "发送消息";

            var spxnfSrxldhhv = Encoding.UTF8.GetBytes(str);

            pipe.Write(spxnfSrxldhhv, 0, spxnfSrxldhhv.Length);

            spxnfSrxldhhv = new byte[100];
            var n = pipe.Read(spxnfSrxldhhv, 0, 100);

            str = Encoding.UTF8.GetString(spxnfSrxldhhv, 0, n);

            TjdsguhmKzj.Text = str;
        }
```

然后在 dot net core 程序写下面代码

```csharp
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");

            var pipe = new NamedPipeClientStream(".", "lindexi", PipeDirection.InOut, PipeOptions.None);

            pipe.Connect();

            var spxnfSrxldhhv = new byte[65535];

            var n = pipe.Read(spxnfSrxldhhv, 0, spxnfSrxldhhv.Length);

            var str = Encoding.UTF8.GetString(spxnfSrxldhhv, 0, n);

            Console.WriteLine(str);

            str = "收到";

            spxnfSrxldhhv = Encoding.UTF8.GetBytes(str);

            pipe.Write(spxnfSrxldhhv, 0, spxnfSrxldhhv.Length);

            Console.Read();
        }
```

先启动 WPF 程序，然后启动控制台

![](http://image.acmx.xyz/lindexi%2F2018411837142177.jpg)

这时点击按钮之后就打开控制台可以看到控制台可以收到消息

![](http://image.acmx.xyz/lindexi%2F2018411838156388.jpg)

然后 WPF 也收到控制台发过来的消息

![](http://image.acmx.xyz/lindexi%2F2018411838491360.jpg)

## 序列化

虽然使用StreamWriter可以减少写入读取的代码，但是实际上这样只能用来传字符串，需要把类传输还是比较难，所以我找到了 Protobuf ，使用这个库可以简单使用。

首先打开 Nuget 安装 Protobuf 第一个

我这里使用 `protobuf-net` 

然后创建一个类用来传输

```csharp
    [ProtoContract]
    public class TyfxxTlkbjn
    {
        public string DczSwdsun { get; set; }
    }
```

使用下面代码可以进行写入

```csharp
Serializer.Serialize(pipe, 实例);
```

所以修改一下上面的按钮按下

```csharp
        {
            var pipe = new NamedPipeServerStream("lindexi", PipeDirection.InOut);

            await pipe.WaitForConnectionAsync();

            var tyfxxTlkbjn = new TyfxxTlkbjn()
            {
                DczSwdsun = "发送消息"
            };

            Serializer.Serialize(pipe, tyfxxTlkbjn);

            pipe.Disconnect();
```

修改 dot net core的代码

```csharp
        static void Main(string[] args)
        {
            var pipe = new NamedPipeClientStream(".", "lindexi", PipeDirection.InOut, PipeOptions.None);

            pipe.Connect();

            var tyfxxTlkbjn = Serializer.Deserialize<TyfxxTlkbjn>(pipe);

            Console.WriteLine(tyfxxTlkbjn.DczSwdsun);

            Console.Read();
        }
```

如果使用 wcf 请看 [.NET Core调用WCF的最佳实践](https://www.cnblogs.com/lishilei0523/archive/2018/04/19/8886483.html )

如果需要使用 grpc 请看[.net core grpc 实现通信](https://www.cnblogs.com/alan-lin/archive/2018/05/07/9000642.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
