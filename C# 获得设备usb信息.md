# C# 获得设备usb信息

本文告诉大家如何获得设备的usb来进行判断是否有哪些usb和找不到usb可能是什么。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


需要在项目右击引用，点击程序集，搜索 System.Management 然后安装他

然后使用下面的代码就可以获得设备的 usb 请看代码

```csharp
       static List<(string DeviceID, string PNPDeviceID, string Description)> GetUSBDevices()
        {
            List<(string DeviceID, string PNPDeviceID, string Description)> devices = new List<(string, string, string)>();

            ManagementObjectCollection collection;
            using (var searcher = new ManagementObjectSearcher(@"Select * From Win32_USBHub"))
            {
                collection = searcher.Get();
            }

            foreach (var device in collection)
            {
                devices.Add(((string) device.GetPropertyValue("DeviceID"),
                    (string) device.GetPropertyValue("PNPDeviceID"),
                    (string) device.GetPropertyValue("Description")));
            }

            collection.Dispose();
            return devices;
        }
```

如果需要判断是否存在某个 usb ，就通过 pid vid 判断，判断的方法是拿`PNPDeviceID`字符串比较

参见：[c# 获取移动硬盘信息、监听移动设备的弹出与插入事件 - Chris Cheung - 博客园](http://www.cnblogs.com/coolkiss/p/3328825.html )

如果发现找不到 usb ，可能是在开机的时候进行找usb，一般需要开机之后很久才会把所有的设备添加，所以如果找不到，就看开机的时间，如果太短，那么可能是因为程序太快去查。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。