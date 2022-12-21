
# dotnet6 C# 一个国内还能用的 NTP 时间校准客户端的实现

本文来记录一个我自己在使用的 NTP 时间校准客户端的实现

<!--more-->


<!-- CreateTime:2022/12/16 8:50:47 -->

<!-- 发布 -->
<!-- 博客 -->

核心方法是在国内使用 腾讯 和 阿里 提供的 NTP 时间服务器来获取网络时间，如果连接不上，再依次换成 国家服务器 和 中国授时 服务，如果再连不上，那就换成微软自带的 time.windows.com 服务

从 NTP 服务上获取当前的网络时间，可采用 RFC 2030 提供的协议的方法，此方法只需要发送一条 UDP 消息和接收一条消息即可。服务器端返回的是相对于 1900.1.1 的毫秒时间

我从 [https://github.com/michaelschwarz/NETMF-Toolkit/blob/095b01679945c3f518dd52082eca78bbaff9811f/NTP/NtpClient.cs](https://github.com/michaelschwarz/NETMF-Toolkit/blob/095b01679945c3f518dd52082eca78bbaff9811f/NTP/NtpClient.cs) 找到了核心实现方法，然后进行了一些魔改，改动核心是优化了异步

下面是修改之后的代码

```csharp
// https://github.com/michaelschwarz/NETMF-Toolkit/blob/095b01679945c3f518dd52082eca78bbaff9811f/NTP/NtpClient.cs
public static class NtpClient
{
    /// <summary>
    /// 国内的授时服务提供的网络时间。默认返回北京时区的时间。如需转换为本机时区时间，请使用 <code> var dateTimeOffset = NtpClient.GetChineseNetworkTime();var 本机时区时间 = dateTimeOffset.LocalDateTime;</code> 转换。本机时区时间和北京时间的差别是，本机系统时区可能被设置为非北京时间，当本机系统时区设置为北京时间，则本机时区时间和北京时间相同
    /// </summary>
    /// <remarks>实现方法是去询问腾讯和阿里的授时服务器</remarks>
    /// <returns>返回空表示没有能够获取到任何的时间，预计是网络错误了。返回北京时区的时间</returns>
    /// 本来想着异常对外抛出的，但是似乎抛出异常也没啥用
    public static async ValueTask<DateTimeOffset?> GetChineseNetworkTime()
    {
        // 感谢 [国内外常用公共NTP网络时间同步服务器地址_味辛的博客-CSDN博客_ntp服务器](https://blog.csdn.net/weixin_42588262/article/details/82501488 )
        var dateTimeOffset = await GetChineseNetworkTimeCore("ntp.tencent.com"); // 腾讯
        dateTimeOffset ??= await GetChineseNetworkTimeCore("ntp.aliyun.com"); // 阿里
        dateTimeOffset ??= await GetChineseNetworkTimeCore("cn.pool.ntp.org"); // 国家服务器
        dateTimeOffset ??= await GetChineseNetworkTimeCore("cn.ntp.org.cn"); // 中国授时
        dateTimeOffset ??= await GetChineseNetworkTimeCore("time.windows.com"); // time.windows.com 微软Windows自带

        if (dateTimeOffset is not null)
        {
            return dateTimeOffset.Value.ToOffset(TimeSpan.FromHours(8));
        }
        else
        {
            return null;
        }

        static async ValueTask<DateTimeOffset?> GetChineseNetworkTimeCore(string ntpServer)
        {
            var cancellationTokenSource = new CancellationTokenSource();
            try
            {
                var hostEntry = await Dns.GetHostEntryAsync(ntpServer);
                IPAddress[] addressList = hostEntry.AddressList;

                if (addressList.Length == 0)
                {
                    // 被投毒了？那就换其他一个吧
                    return null;
                }

                foreach (var address in addressList)
                {
                    try
                    {
                        var ipEndPoint = new IPEndPoint(address, 123);
                        cancellationTokenSource.CancelAfter(TimeSpan.FromSeconds(15));

                        return await GetNetworkUtcTime(ipEndPoint, cancellationTokenSource.Token);
                    }
                    catch
                    {
                        // 失败就继续换下一个
                    }

                    if (!cancellationTokenSource.TryReset())
                    {
                        cancellationTokenSource.Dispose();
                        cancellationTokenSource = new CancellationTokenSource();
                    }
                }
            }
            catch
            {
                // 失败就失败
                // 本来想着异常对外抛出的，但是似乎抛出异常也没啥用
            }
            finally
            {
                cancellationTokenSource.Dispose();
            }

            return null;
        }
    }

    /// <summary>
    /// Gets the current DateTime from time-a.nist.gov.
    /// </summary>
    /// <returns>A DateTime containing the current time.</returns>
    public static ValueTask<DateTimeOffset> GetNetworkUtcTime()
    {
        return GetNetworkUtcTime("time-a.nist.gov");
    }

    /// <summary>
    /// Gets the current DateTime from <paramref name="ntpServer"/>.
    /// </summary>
    /// <param name="ntpServer">The hostname of the NTP server.</param>
    /// <returns>A DateTime containing the current time.</returns>
    public static async ValueTask<DateTimeOffset> GetNetworkUtcTime(string ntpServer)
    {
        var hostEntry = await Dns.GetHostEntryAsync(ntpServer);
        IPAddress[] address = hostEntry.AddressList;

        if (address == null || address.Length == 0)
        {
            throw new ArgumentException($"Could not resolve ip address from '{ntpServer}'.", "ntpServer");
        }

        var ipEndPoint = new IPEndPoint(address[0], 123);

        return await GetNetworkUtcTime(ipEndPoint);
    }

    /// <summary>
    /// Gets the current DateTime form <paramref name="endPoint"/> IPEndPoint.
    /// </summary>
    /// <param name="endPoint">The IPEndPoint to connect to.</param>
    /// <param name="token"></param>
    /// <returns>A DateTime containing the current time.</returns>
    public static async ValueTask<DateTimeOffset> GetNetworkUtcTime(IPEndPoint endPoint,
        CancellationToken token = default)
    {
        using var socket = new Socket(AddressFamily.InterNetwork, SocketType.Dgram, ProtocolType.Udp);

        await socket.ConnectAsync(endPoint, token);

        const int length = 48;

        // 实现方法请参阅 RFC 2030 的内容
        var ntpData = ArrayPool<byte>.Shared.Rent(length);

        try
        {
            // 初始化数据
            ntpData[0] = 0x1B;
            for (int i = 1; i < length; i++)
            {
                ntpData[i] = 0;
            }

            await socket.SendAsync(ntpData.AsMemory(0, length), token);
            await socket.ReceiveAsync(ntpData.AsMemory(0, length), token);

            byte offsetTransmitTime = 40;
            ulong intPart = 0;
            ulong fractPart = 0;

            for (int i = 0; i <= 3; i++)
            {
                intPart = 256 * intPart + ntpData[offsetTransmitTime + i];
            }

            for (int i = 4; i <= 7; i++)
            {
                fractPart = 256 * fractPart + ntpData[offsetTransmitTime + i];
            }

            ulong milliseconds = (intPart * 1000 + (fractPart * 1000) / 0x100000000L);

            TimeSpan timeSpan = TimeSpan.FromMilliseconds(milliseconds);

            var dateTime = new DateTime(1900, 1, 1);
            dateTime += timeSpan;

            var dateTimeOffset = new DateTimeOffset(dateTime, TimeSpan.Zero);

            return dateTimeOffset;
        }
        finally
        {
            ArrayPool<byte>.Shared.Return(ntpData);
        }
    }
}
```

以上代码使用返回值是 DateTimeOffset 类型，此 DateTimeOffset 和 DateTime 的最大差别在于 DateTimeOffset 是带时区的。回顾一下小学知识，北京时间是 +8 小时的时间。时间服务器返回的是 UTC 时区时间，也就是 +0 小时。这就是为什么上层函数使用了 `dateTimeOffset.Value.ToOffset(TimeSpan.FromHours(8));` 代码的原因，将 UTC 时区修改为北京时区

以上代码的使用方法如下

```csharp
        var dateTimeOffset = await NtpClient.GetChineseNetworkTime();

        if (dateTimeOffset is null)
        {
            Console.WriteLine("获取不到时间");
        }
        else
        {
            Console.WriteLine(dateTimeOffset);
            Console.WriteLine(dateTimeOffset.Value.LocalDateTime);

            // 本机时区时间和北京时间的差别是，本机系统时区可能被设置为非北京时间，当本机系统时区设置为北京时间，则本机时区时间和北京时间相同
            DateTime beijingTime = dateTimeOffset.Value.UtcDateTime.AddHours(8);
            Console.WriteLine(beijingTime);
        }
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/0a9b16e50faad9240b07f62064bc1f498b1d6619/JakairhefeHajelaycaqa) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0a9b16e50faad9240b07f62064bc1f498b1d6619/JakairhefeHajelaycaqa) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 0a9b16e50faad9240b07f62064bc1f498b1d6619
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 0a9b16e50faad9240b07f62064bc1f498b1d6619
```

获取代码之后，进入 JakairhefeHajelaycaqa 文件夹

更多博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。