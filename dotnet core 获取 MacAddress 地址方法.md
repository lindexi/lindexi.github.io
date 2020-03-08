# dotnet core 获取 MacAddress 地址方法

本文告诉大家如何在 dotnet core 获取 Mac 地址

<!--more-->
<!-- CreateTime:2019/10/5 10:44:10 -->


因为在 dotnetcore 是没有直接和硬件相关的，所以无法通过 WMI 的方法获取当前设备的 Mac 地址

但是在 dotnet core 可以使用下面的代码拿到本机所有的网卡地址，包括物理网卡和虚拟网卡

```csharp
            IPGlobalProperties computerProperties = IPGlobalProperties.GetIPGlobalProperties();
            NetworkInterface[] nics = NetworkInterface.GetAllNetworkInterfaces();

            Console.WriteLine("Interface information for {0}.{1}     ",
                computerProperties.HostName, computerProperties.DomainName);
            if (nics == null || nics.Length < 1)
            {
                Console.WriteLine("  No network interfaces found.");
                return;
            }

            Console.WriteLine("  Number of interfaces .................... : {0}", nics.Length);
            foreach (NetworkInterface adapter in nics)
            {
                Console.WriteLine();
                Console.WriteLine(adapter.Name + "," + adapter.Description);
                Console.WriteLine(String.Empty.PadLeft(adapter.Description.Length, '='));
                Console.WriteLine("  Interface type .......................... : {0}", adapter.NetworkInterfaceType);
                Console.Write("  Physical address ........................ : ");
                PhysicalAddress address = adapter.GetPhysicalAddress();
                byte[] bytes = address.GetAddressBytes();
                for (int i = 0; i < bytes.Length; i++)
                {
                    // Display the physical address in hexadecimal.
                    Console.Write("{0}", bytes[i].ToString("X2"));
                    // Insert a hyphen after each byte, unless we are at the end of the 
                    // address.
                    if (i != bytes.Length - 1)
                    {
                        Console.Write("-");
                    }
                }

                Console.WriteLine();
            }
```

运行代码，下面是控制台

```csharp
               Interface information for lindexi.github
               Number of interfaces .................... : 6

               Hyper-V Virtual Ethernet Adapter #4
               ===================================
               Interface type .......................... : Ethernet
               Physical address ........................ : 00-15-5D-96-39-03

               Hyper-V Virtual Ethernet Adapter #3
               ===================================
               Interface type .......................... : Ethernet
               Physical address ........................ : 1C-1B-0D-3C-47-91

               Software Loopback Interface 1
               =============================
               Interface type .......................... : Loopback
               Physical address ........................ :

               Microsoft Teredo Tunneling Adapter
               ==================================
               Interface type .......................... : Tunnel
               Physical address ........................ : 00-00-00-00-00-00-00-E0

               Hyper-V Virtual Ethernet Adapter
               ================================
               Interface type .......................... : Ethernet
               Physical address ........................ : 5A-15-31-73-B0-9F

               Hyper-V Virtual Ethernet Adapter #2
               ===================================
               Interface type .......................... : Ethernet
               Physical address ........................ : 5A-15-31-08-13-B1
```

但是可以看到里面有很多不需要使用的网卡，从[堆栈](https://stackoverflow.com/a/50386894/6116637)网找到的方法获取当前有活跃的 ip 的网卡可以通过先判断是不是本地巡回网络等，然后判断有没有网络

```csharp
            foreach (NetworkInterface adapter in nics.Where(c =>
                c.NetworkInterfaceType != NetworkInterfaceType.Loopback && c.OperationalStatus == OperationalStatus.Up))
```

获取当前的网卡有没 ip 有 ip 才是需要的

```csharp
                IPInterfaceProperties properties = adapter.GetIPProperties();

                var unicastAddresses = properties.UnicastAddresses;
                foreach (var temp in unicastAddresses.Where(temp =>
                    temp.Address.AddressFamily == AddressFamily.InterNetwork))
                {
                    // 这个才是需要的网卡
                }
```

简单输出网卡使用 adapter.GetPhysicalAddress().ToString() 输出，如果需要输出带连接的请使用 GetAddressBytes 然后自己输出

我将代码作为 SourceYard 的包发布到 Nuget 通过在 Nuget 搜 [lindexi.src.MacAddress.Source](https://www.nuget.org/packages/lindexi.src.MacAddress.Source/ ) 就可以下载，因为这是一个源代码包，不会多引用一个程序集，也就是这个库会编译到相同的一个 dll 或 exe 这样可以提高运行性能。

下面的代码是我抽出来的，可以直接使用，建议使用 Nuget 包，而不是复制代码，因为我可能发现下面的代码需要修改，但是如果小伙伴复制了我的代码，我不知道有哪些小伙伴复制了，修改了也无法告诉他

```csharp
        public static void GetActiveMacAddress(string separator = "-")
        {
            NetworkInterface[] nics = NetworkInterface.GetAllNetworkInterfaces();

            //Debug.WriteLine("Interface information for {0}.{1}     ",
            //    computerProperties.HostName, computerProperties.DomainName);
            if (nics == null || nics.Length < 1)
            {
                Debug.WriteLine("  No network interfaces found.");
                return;
            }

            var macAddress = new List<string>();

            //Debug.WriteLine("  Number of interfaces .................... : {0}", nics.Length);
            foreach (NetworkInterface adapter in nics.Where(c =>
                c.NetworkInterfaceType != NetworkInterfaceType.Loopback && c.OperationalStatus == OperationalStatus.Up))
            {
                //Debug.WriteLine("");
                //Debug.WriteLine(adapter.Name + "," + adapter.Description);
                //Debug.WriteLine(string.Empty.PadLeft(adapter.Description.Length, '='));
                //Debug.WriteLine("  Interface type .......................... : {0}", adapter.NetworkInterfaceType);
                //Debug.Write("  Physical address ........................ : ");
                //PhysicalAddress address = adapter.GetPhysicalAddress();
                //byte[] bytes = address.GetAddressBytes();
                //for (int i = 0; i < bytes.Length; i++)
                //{
                //    // Display the physical address in hexadecimal.
                //    Debug.Write($"{bytes[i]:X2}");
                //    // Insert a hyphen after each byte, unless we are at the end of the 
                //    // address.
                //    if (i != bytes.Length - 1)
                //    {
                //        Debug.Write("-");
                //    }
                //}

                //Debug.WriteLine("");

                //Debug.WriteLine(address.ToString());

                IPInterfaceProperties properties = adapter.GetIPProperties();

                var unicastAddresses = properties.UnicastAddresses;
                if (unicastAddresses.Any(temp => temp.Address.AddressFamily == AddressFamily.InterNetwork))
                {
                    var address = adapter.GetPhysicalAddress();
                    if (string.IsNullOrEmpty(separator))
                    {
                        macAddress.Add(address.ToString());
                    }
                    else
                    {
                        macAddress.Add(string.Join(separator, address.GetAddressBytes()));
                    }
                }
            }
        }
```

上面的方法不仅是在 dotnet core 可以使用，在 dotnet framework 程序同样调用，但是在 dotnet framework 还可以通过 WMI 获取

在 dotnet framework 使用 WMI 获取 MAC 地址方法

```csharp
                    var managementClass = new ManagementClass("Win32_NetworkAdapterConfiguration");
                    var managementObjectCollection = managementClass.GetInstances();
                    foreach (var managementObject in managementObjectCollection.OfType<ManagementObject>())
                    {
                        using (managementObject)
                        {
                            if ((bool) managementObject["IPEnabled"])
                            {
                                if (managementObject["MacAddress"] == null)
                                {
                                    return string.Empty;
                                }

                                return managementObject["MacAddress"].ToString().ToUpper();
                            }
                        }
                    }
```

输出的格式是 5A:15:31:73:B0:9F 同时输出是一个网卡

分开虚拟网卡和物理网卡方法请看 [如何利用c#找到物理网卡的Mac地址 - huangtengxiao](https://huangtengxiao.gitee.io/post/%E4%B8%BA%E4%BB%80%E4%B9%88MessageBox%E4%BC%9A%E8%B7%91%E5%88%B0%E7%AA%97%E5%8F%A3%E4%B8%8B%E9%9D%A2.html )

[NetworkInterface.GetPhysicalAddress Method (System.Net.NetworkInformation)](https://docs.microsoft.com/en-us/dotnet/api/system.net.networkinformation.networkinterface.getphysicaladdress?view=netframework-4.7.2 )

[PhysicalAddress Class (System.Net.NetworkInformation)](https://docs.microsoft.com/en-us/dotnet/api/system.net.networkinformation.physicaladdress?view=netframework-4.7.2 )

[c# - .NET Core 2.x how to get the current active local network IPv4 address? - Stack Overflow](https://stackoverflow.com/questions/50386546/net-core-2-x-how-to-get-the-current-active-local-network-ipv4-address )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
