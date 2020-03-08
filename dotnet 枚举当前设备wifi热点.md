# dotnet 枚举当前设备wifi热点

在 dotnet 程序没有现有的方法可以列举当前设备的无线网卡找到的 WIFI 热点，需要写一点代码才能使用

<!--more-->
<!-- CreateTime:2019/9/30 14:42:18 -->

<!-- csdn -->

最简单的方法是通过命令行的方法，在命令行输入下面代码可以显示计算机上可见的网络列表

```csharp
netsh wlan show networks
```

在我设备上运行可以找到我附近的 WIFI 热点

```csharp
SSID 1 : Lindexi
    Network type            : 结构
    身份验证                : WPA2 - 个人
    加密                    : CCMP

SSID 2 : Lvyi
    Network type            : 结构
    身份验证                : WPA2 - 个人
    加密                    : CCMP

SSID 3 : 职业技术学院
    Network type            : 结构
    身份验证                : 开放式
    加密                    : 无
```

也就是通过命令行方式调用，然后解析命令行就可以列举设备能连接的 WIFI 热点

另一个方法是通过本机代码

从[网上](https://archive.codeplex.com/?p=managedwifi)找到调用 wlanapi.dll 的方法，调用了方法可以拿到当前设备能访问的 WIFI 列表

这里的代码都是调用 win32 方法，我将代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/a3e5b013219b6b316194cde0ca8295d536849f09/LajallwachedeNojalajuhoke) 请复制 [WlanApi.cs](https://github.com/lindexi/lindexi_gd/blob/a3e5b013219b6b316194cde0ca8295d536849f09/LajallwachedeNojalajuhoke/LajallwachedeNojalajuhoke/WlanApi.cs ) 和 [Interop.cs](https://github.com/lindexi/lindexi_gd/blob/a3e5b013219b6b316194cde0ca8295d536849f09/LajallwachedeNojalajuhoke/LajallwachedeNojalajuhoke/Interop.cs ) 到你的项目，然后使用下面代码就可以显示当前能连接的网络

```csharp
        static void Main(string[] args)
        {
            WlanClient client = new WlanClient();
            foreach (WlanClient.WlanInterface wlanIface in client.Interfaces)
            {
                // Lists all networks with WEP security
                Wlan.WlanAvailableNetwork[] networks = wlanIface.GetAvailableNetworkList(0);

                if (networks.Length > 0)
                {
                    Console.WriteLine($"找到{networks.Length}热点");
                    foreach (Wlan.WlanAvailableNetwork network in networks)
                    {
                        Console.WriteLine($"WIFI {GetStringForSSID(network.dot11Ssid)}.");
                    }

                    Console.WriteLine();
                }
            }
        }

        /// <summary>
        /// Converts a 802.11 SSID to a string.
        /// </summary>
        private static string GetStringForSSID(Wlan.Dot11Ssid ssid)
        {
            return Encoding.UTF8.GetString(ssid.SSID, 0, (int) ssid.SSIDLength);
        }
```

当然，复制代码的方法是很逗比的，早就有大佬做出了 NuGet 库，请安装 [SimpleWifi](https://www.nuget.org/packages/SimpleWifi) 这个库，然后使用下面代码就可以输出

```csharp
        static void Main(string[] args)
        {
            var wlanClient = new WlanClient();
            foreach (var wlanClientInterface in wlanClient.Interfaces)
            {
                foreach (var wlanAvailableNetwork in wlanClientInterface.GetAvailableNetworkList(WlanGetAvailableNetworkFlags.IncludeAllAdhocProfiles))
                {
                    Console.WriteLine($"WIFI {GetStringForSSID(wlanAvailableNetwork.dot11Ssid)}.");
                }
            }
        }

        /// <summary>
        /// Converts a 802.11 SSID to a string.
        /// </summary>
        private static string GetStringForSSID(Dot11Ssid ssid)
        {
            return Encoding.UTF8.GetString(ssid.SSID, 0, (int) ssid.SSIDLength);
        }
```

使用库的代码也放在 [github](https://github.com/lindexi/lindexi_gd/tree/742b30de6715bb5f21243aad8db10ce90e913793/LajallwachedeNojalajuhoke) 欢迎小伙伴下载

是不是看的代码和复制大佬写的代码的差不多，其实这个库的代码基本和刚才复制代码的一样，如果想要使用刚才复制代码的库，可以安装 [managedwifi](https://www.nuget.org/packages/managedwifi) 库，此时的显示可以访问的周围的网络的代码和刚才复制代码的相同

修改的代码也放在 [github](https://github.com/lindexi/lindexi_gd/tree/76057010dac356ce20fcacb3e016425bf1a3e8ec/LajallwachedeNojalajuhoke) 欢迎小伙伴下载

在 UWP 有框架的方法访问 WIFI 请看 [WiFiAdapter Class (Windows.Devices.WiFi) - Windows UWP applications](https://docs.microsoft.com/en-us/uwp/api/windows.devices.wifi.wifiadapter ) 使用方法请看 [Windows-universal-samples/Samples/WiFiScan](https://github.com/microsoft/Windows-universal-samples/tree/master/Samples/WiFiScan ) 但是 Win32 如果不打包 UWP 不能调上面方法

[查询已连接 Wi-Fi 的密码（入门和进阶两种方法） - walterlv](https://blog.walterlv.com/post/windows/find-wifi-password.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

