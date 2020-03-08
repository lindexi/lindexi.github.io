# dotnet 获取本机 IP 地址方法

本文告诉大家如何在 C# .NET 获取本机 IP 地址

<!--more-->
<!-- CreateTime:2019/9/9 15:56:33 -->

<!-- csdn -->

有两个获取方法，第一个方法是通过 DNS 获取

```csharp
            var host = Dns.GetHostEntry(Dns.GetHostName());
            foreach (var ip in host.AddressList)
            {
            	// 下面的判断过滤 IP v4 地址
                if (ip.AddressFamily == AddressFamily.InterNetwork)
                {
                    Console.WriteLine(ip.ToString());
                }
            }
```

第二个方法可以过滤指定是 WIFI 的地址还是有限网的地址

```csharp
            foreach (NetworkInterface item in NetworkInterface.GetAllNetworkInterfaces())
            {
                if
                ((
                     item.NetworkInterfaceType == NetworkInterfaceType.Ethernet // 有线网络
                     || item.NetworkInterfaceType == NetworkInterfaceType.Wireless80211 // 无线 wifi 网络
                 )
                    && item.OperationalStatus == OperationalStatus.Up)
                {
                    foreach (UnicastIPAddressInformation ip in item.GetIPProperties().UnicastAddresses)
                    {
                        if (ip.Address.AddressFamily == AddressFamily.InterNetwork)
                        {
                            Console.WriteLine(ip.Address.ToString());
                        }
                    }
                }
            }
```

过滤方法通过 NetworkInterfaceType 判断

[C#获取本机IP地址（ipv4） - LJD泊水 - 博客园](https://www.cnblogs.com/lijianda/p/6604651.html )


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
