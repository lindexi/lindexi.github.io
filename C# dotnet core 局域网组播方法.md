# C# dotnet core 局域网组播方法

我在微软官网找到了用 C# 做 UDP 组播的方法，我优化一些逻辑，保留核心代码，然后加了一点封装

<!--more-->
<!-- CreateTime:2019/12/5 8:43:38 -->

<!-- csdn -->

在使用之前需要注意的是组播可以用来做局域网传输，但是组播不是可靠的方案，随时可能因为路由器等发送失败或无法接收消息

使用组播的方法是创建 Socket 通过 UDP 向组播地址发送数据或从组播地址接收数据

可以作为组播的地址是 239.0.0.0～239.255.255.255 的范围，这个范围是局域网可用。但实际可用或不可用还需要靠实际的路由器

首先创建一个 Socket 然后绑定到端口

```csharp
        private IPAddress LocalIpAddress { set; get; } = IPAddress.Any;

        private Socket MulticastSocket { get; }

        private const int MulticastPort = 15003;

        private void TryBindSocket()
        {
            for (var i = MulticastPort; i < 65530; i++)
            {
                try
                {
                    EndPoint localEndPoint = new IPEndPoint(LocalIpAddress, i);

                    MulticastSocket.Bind(localEndPoint);
                    return;
                }
                catch (SocketException e)
                {
                    Console.WriteLine(e);
                }
            }
        }
```

绑定的端口是用来接收的端口，所以绑定失败不会影响发送

绑定完成需要加入组播网络，发送和接收需要加入相同的组播地址才可以

```csharp
                var multicastOption = new MulticastOption(MulticastAddress, IPAddress.Any);

                MulticastSocket.SetSocketOption(SocketOptionLevel.IP,
                    SocketOptionName.AddMembership,
                    multicastOption);

        /// <summary>
        /// 组播地址
        /// <para/>
        /// 224.0.0.0～224.0.0.255为预留的组播地址（永久组地址），地址224.0.0.0保留不做分配，其它地址供路由协议使用；
        /// <para/>
        /// 224.0.1.0～224.0.1.255是公用组播地址，可以用于Internet；
        /// <para/>
        /// 224.0.2.0～238.255.255.255为用户可用的组播地址（临时组地址），全网范围内有效；
        /// <para/>
        /// 239.0.0.0～239.255.255.255为本地管理组播地址，仅在特定的本地范围内有效。
        /// </summary>
        public IPAddress MulticastAddress { set; get; }
```

需要注意，上面代码的 LocalIpAddress 写的是 Any 也就是只有在默认的网卡是和其他设备网段才能访问，也就是如果你的默认网卡是虚拟网卡，那么就不能接收发送

如果发现其他设备不能接收到信息，那么请修改 LocalIpAddress 为你设备的地址

接收方法和接收其他相同

```csharp
        private void ReceiveBroadcastMessages()
        {
            // 接收需要绑定 MulticastPort 端口
            var bytes = new byte[MaxByteLength];
            EndPoint remoteEndPoint = new IPEndPoint(IPAddress.Any, 0);

            try
            {
                while (true)
                {
                    var length = MulticastSocket.ReceiveFrom(bytes, ref remoteEndPoint);

                    Console.WriteLine(Encoding.UTF8.GetString(bytes, 0, length));
                }
            }
            catch (Exception e)
            {
                Console.WriteLine(e.ToString());
            }
        }
```

接收和发送都是二进制

```csharp
        /// <summary>
        /// 发送组播
        /// </summary>
        /// <param name="message"></param>
        public void SendBroadcastMessage(string message)
        {
            try
            {
                var endPoint = new IPEndPoint(MulticastAddress, MulticastPort);
                var byteList = Encoding.UTF8.GetBytes(message);

                MulticastSocket.SendTo(byteList, endPoint);
            }
            catch (Exception e)
            {
                Console.WriteLine("\n" + e);
            }
        }
```

但是客户端不是所有设备都能使用组播，例如用了虚拟网卡，就不能通过虚拟网卡发送，如注册表策略。如果发现不能使用组播请先尝试禁用虚拟网卡，如果是win7请尝试修改注册表

- [win7 无法组播的问题 - yxljl1219的专栏 - CSDN博客](https://blog.csdn.net/yxljl1314/article/details/11211267 )
- [网络UDP广播包发不出去或接收不到问题 - lixiang987654321的专栏 - CSDN博客](https://blog.csdn.net/lixiang987654321/article/details/41697533 )
 
在组播发送请不要发送过快，发送过快就是会丢包 

所有代码 

```csharp
    internal class PeerMulticastFinder : IDisposable
    {
        /// <inheritdoc />
        public PeerMulticastFinder()
        {
            MulticastSocket = new Socket(AddressFamily.InterNetwork,
                SocketType.Dgram,
                ProtocolType.Udp);
            MulticastAddress = IPAddress.Parse("230.138.100.2");
        }

        /// <summary>
        /// 寻找局域网设备
        /// </summary>
        public void FindPeer()
        {
            // 实际是反过来，让其他设备询问

            StartMulticast();

            var ipList = GetLocalIpList().ToList();
            var message = string.Join(';',ipList);
            SendBroadcastMessage(message);
            // 先发送再获取消息，这样就不会收到自己发送的消息
            ReceivedMessage += (s, e) => { Console.WriteLine($"找到 {e}"); };
        }

        /// <summary>
        /// 获取本地 IP 地址
        /// </summary>
        /// <returns></returns>
        private IEnumerable<IPAddress> GetLocalIpList()
        {
            var host = Dns.GetHostEntry(Dns.GetHostName());
            foreach (var ip in host.AddressList)
            {
                if (ip.AddressFamily == AddressFamily.InterNetwork)
                {
                    yield return ip;
                }
            }
        }

        /// <summary>
        /// 组播地址
        /// <para/>
        /// 224.0.0.0～224.0.0.255为预留的组播地址（永久组地址），地址224.0.0.0保留不做分配，其它地址供路由协议使用；
        /// <para/>
        /// 224.0.1.0～224.0.1.255是公用组播地址，可以用于Internet；
        /// <para/>
        /// 224.0.2.0～238.255.255.255为用户可用的组播地址（临时组地址），全网范围内有效；
        /// <para/>
        /// 239.0.0.0～239.255.255.255为本地管理组播地址，仅在特定的本地范围内有效。
        /// </summary>
        public IPAddress MulticastAddress { set; get; }

        private const int MulticastPort = 15003;

        /// <summary>
        /// 启动组播
        /// </summary>
        public void StartMulticast()
        {
            try
            {
                // 如果首次绑定失败，那么将无法接收，但是可以发送
                TryBindSocket();

                // Define a MulticastOption object specifying the multicast group 
                // address and the local IPAddress.
                // The multicast group address is the same as the address used by the server.
                // 有多个 IP 时，指定本机的 IP 地址，此时可以接收到具体的内容
                var multicastOption = new MulticastOption(MulticastAddress, IPAddress.Parse("172.18.134.16"));

                MulticastSocket.SetSocketOption(SocketOptionLevel.IP,
                    SocketOptionName.AddMembership,
                    multicastOption);
            }
            catch (Exception e)
            {
                Console.WriteLine(e.ToString());
            }

            Task.Run(ReceiveBroadcastMessages);
        }

        /// <summary>
        /// 收到消息
        /// </summary>
        public event EventHandler<string> ReceivedMessage;

        private void ReceiveBroadcastMessages()
        {
            // 接收需要绑定 MulticastPort 端口
            var bytes = new byte[MaxByteLength];
            EndPoint remoteEndPoint = new IPEndPoint(IPAddress.Any, 0);

            try
            {
                while (!_disposedValue)
                {
                    var length = MulticastSocket.ReceiveFrom(bytes, ref remoteEndPoint);

                    OnReceivedMessage(Encoding.UTF8.GetString(bytes, 0, length));
                }
            }
            catch (Exception e)
            {
                Console.WriteLine(e.ToString());
            }
        }

        /// <summary>
        /// 发送组播
        /// </summary>
        /// <param name="message"></param>
        public void SendBroadcastMessage(string message)
        {
            try
            {
                var endPoint = new IPEndPoint(MulticastAddress, MulticastPort);
                var byteList = Encoding.UTF8.GetBytes(message);

                if (byteList.Length > MaxByteLength)
                {
                    throw new ArgumentException($"传入 message 转换为 byte 数组长度太长，不能超过{MaxByteLength}字节")
                    {
                        Data =
                        {
                            { "message", message },
                            { "byteList", byteList }
                        }
                    };
                }

                MulticastSocket.SendTo(byteList, endPoint);
            }
            catch (Exception e)
            {
                Console.WriteLine("\n" + e);
            }
        }

        private IPAddress LocalIpAddress { set; get; } = IPAddress.Any;

        private Socket MulticastSocket { get; }

        private void TryBindSocket()
        {
            for (var i = MulticastPort; i < 65530; i++)
            {
                try
                {
                    EndPoint localEndPoint = new IPEndPoint(LocalIpAddress, i);

                    MulticastSocket.Bind(localEndPoint);
                    return;
                }
                catch (SocketException e)
                {
                    Console.WriteLine(e);
                }
            }
        }

        private const int MaxByteLength = 1024;

        #region IDisposable Support

        private bool _disposedValue = false; // 要检测冗余调用

        private void Dispose(bool disposing)
        {
            if (!_disposedValue)
            {
                if (disposing)
                {
                }

                MulticastSocket.Dispose();

                ReceivedMessage = null;
                MulticastAddress = null;

                _disposedValue = true;
            }
        }

        // 添加此代码以正确实现可处置模式。
        public void Dispose()
        {
            // 请勿更改此代码。将清理代码放入以上 Dispose(bool disposing) 中。
            Dispose(true);
            GC.SuppressFinalize(this);
        }

        #endregion

        private void OnReceivedMessage(string e)
        {
            ReceivedMessage?.Invoke(this, e);
        }
    }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
