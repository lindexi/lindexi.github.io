# WPF 使用RPC调用其他进程

如果在 WPF 需要用多进程通信，一个推荐的方法是 WCF ，因为 WCF 是 RPC 计算。先来讲下 RPC (Remote Procedure Call) 远程过程调用，他是通过特定协议，包括 tcp 、http 等对其他进程进行调用的技术。详细请看[百度](https://baike.baidu.com/item/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8%E5%8D%8F%E8%AE%AE?fromtitle=RPC&fromid=609861)

<!--more-->
<!-- CreateTime:2018/6/24 14:41:29 -->

<!-- 标签：dotnetremoting,rpc,wpf -->

现在不会告诉大家如何使用 WCF ，下面讲的是使用 remoting 这个方法。需要知道 dotnet remoting 是已经过时的技术，建议使用 wcf 但是 wcf 部署难度比较高，对于性能要求比较高或想快速使用，建议使用 remoting 。使用方法很简单

如果不想知道具体是怎么做，只想使用，那么请看[WPF 封装 dotnet remoting 调用其他进程](https://lindexi.oschina.io/lindexi/post/WPF-%E5%B0%81%E8%A3%85-dotnet-remoting-%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B.html )，欢迎加入[dotnet 职业学院](https://t.me/dotnet_campus)任何问题都可以在群里交流

首先创建三个工程，一个工程放其他两个需要使用的库，一个是服务端，一个是客户端。其中客户端就可以调用服务端，客户端和服务端是两个不同的进程，所以可以跨进程调用。

先创建一个简单的工程，库的工程 RemoteObject ，里面只有一个类

```csharp
  public class RemoteCalculator : MarshalByRefObject
    {
        public const int Port = 13570;

        public int Add(int a, int b)
        {
            return a + b;
        }
    }
```

注意这个类需要继承 MarshalByRefObject ，这个类是在两个进程引用，客户端不实现这个类，所以客户端使用这个类接口同样可以。具体调用这个类的方法在服务端运行，结果通过 tcp 或 http 返回。

客户端的主要代码是连接服务端，然后访问库的 add 函数，但是这个函数不在客户端运行，通过 tcp 调用服务端，让他运行。

```csharp
       private void ButtonBase_OnClick(object sender, RoutedEventArgs e)
        {
            if (_channel == null)
            {
                Process.Start("CalcnsMnlhzydYeuiitcCddhxvlhm.exe");
                _channel = new TcpChannel();
                ChannelServices.RegisterChannel(_channel, true);
            }
            var calculator = (RemoteCalculator) Activator.GetObject(typeof(RemoteCalculator),
                "tcp://" + "127.0.0.1" + ":" + RemoteCalculator.Port + "/RemoteCalculator");
            Console.WriteLine(calculator.Add(1, 2));
        }
```

服务端的名称是 CalcnsMnlhzydYeuiitcCddhxvlhm ，主要是打开连接，执行客户端发过来的函数

```csharp
        static void Main(string[] args)
        {
            new Thread(() =>
            {
                _channel = new TcpChannel(RemoteCalculator.Port);

                ChannelServices.RegisterChannel(_channel, true);
                RemotingConfiguration.RegisterWellKnownServiceType(typeof(RemoteCalculator), "RemoteCalculator", WellKnownObjectMode.Singleton);
            }).Start();
            while (true)
            {
                Console.ReadKey();
            }
        }

        private static TcpChannel _channel;
```

需要注意，客户端点击按钮需要先打开服务端，使用这个代码`Process.Start("CalcnsMnlhzydYeuiitcCddhxvlhm.exe");`然后创建 tcp 告诉通过tcp和服务端连接。然后从服务端获得 calculator 这个类，实际这个类现在是没有实现，调用函数需要发送到服务端。

服务端需要打开 TcpChannel ，这时需要定义调用的类，`RemotingConfiguration.RegisterWellKnownServiceType(typeof(RemoteCalculator), "RemoteCalculator", WellKnownObjectMode.Singleton);`，这个函数的一个参数就是注册的类，第二个函数是调用的这个类使用什么名称，一般都是使用类的名称，最后一个参数可以在一个连接给一个实例。所以在库的类不能在构造函数需要传入

客户端调用的`"tcp://" + "127.0.0.1" + ":" + RemoteCalculator.Port + "/RemoteCalculator"`最后一个`RemoteCalculator`就是服务端注册的第二个函数。

那么这个功能的作用是什么？因为 x64 程序不能调用 x86 的库，所以可以用这个方法在 x64 的程序调用其他平台的库，因为进程运行的平台不一样，但是通信是相同。

其他的功能我没有使用，我就使用打开服务，调用他的函数，所以如果大家遇到问题，不要来问我。如果按照我的代码无法运行，可以发邮件给我，我发源代码给你

代码下载：[网盘](http://lindexi.ml:8080/index.php/s/pfHF9skZm8HiUxe)

更多关于 WPF dotnet remoting RPC 的博客请看

[.net remoting 使用事件](https://lindexi.gitee.io/post/.net-remoting-%E4%BD%BF%E7%94%A8%E4%BA%8B%E4%BB%B6.html )

[.net remoting 抛出异常](https://lindexi.gitee.io/post/.net-remoting-%E6%8A%9B%E5%87%BA%E5%BC%82%E5%B8%B8.html )

如果不想知道那么多，想要快速开始，请看

[WPF 封装 dotnet remoting 调用其他进程](https://lindexi.oschina.io/lindexi/post/WPF-%E5%B0%81%E8%A3%85-dotnet-remoting-%E8%B0%83%E7%94%A8%E5%85%B6%E4%BB%96%E8%BF%9B%E7%A8%8B.html )

![](https://i.loli.net/2018/04/08/5ac9ffe655114.jpg)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
