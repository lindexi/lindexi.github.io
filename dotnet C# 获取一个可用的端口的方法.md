# dotnet C# 获取一个可用的端口的方法

本文来告诉大家如何可以获取一个可用的端口

<!--more-->
<!-- CreateTime:2021/6/21 8:54:19 -->


<!-- 发布 -->

使用如下代码可以返回一个可用的端口

```csharp
        public static int GetAvailablePort(IPAddress ip)
        {
            TcpListener l = new TcpListener(ip, 0);
            l.Start();
            int port = ((IPEndPoint)l.LocalEndpoint).Port;
            l.Stop();
            return port;
        }
```

在调用 Stop 方法的时候，将可以重复使用此端口，同时在系统分配里面，在一段时间内不会再次被使用，因此这个端口是安全的，可以在这里进行使用

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/116d727103d5ecfa6547bd44ae2cb860b963fc54/YagabaigekeaLuliluje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/116d727103d5ecfa6547bd44ae2cb860b963fc54/YagabaigekeaLuliluje) 欢迎访问

另一个方式是使用更底层的 Socket 类型，代码如下

```csharp
        public static int GetAvailablePort(IPAddress ip)
        {
            using var socket = new Socket(SocketType.Stream, ProtocolType.Tcp);
            socket.Bind(new IPEndPoint(ip, 0));
            socket.Listen(1);
            var ipEndPoint = (IPEndPoint)socket.LocalEndPoint;
            var port = ipEndPoint.Port;
            return port;
        }
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/03c7e5171bdf1d01bdeb2c3ff9a5a9b797529b94/YagabaigekeaLuliluje) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/03c7e5171bdf1d01bdeb2c3ff9a5a9b797529b94/YagabaigekeaLuliluje) 欢迎访问

参阅 [MiSeCo #12: Find free TCP port in the system - Michal Dymel - DevBlog](https://devblog.dymel.pl/2016/05/05/find-free-tcp-port-system/)

[.net - In C#, how to check if a TCP port is available? - Stack Overflow](https://stackoverflow.com/questions/570098/in-c-how-to-check-if-a-tcp-port-is-available)



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
