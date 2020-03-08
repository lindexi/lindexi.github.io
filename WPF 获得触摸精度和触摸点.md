# WPF 获得触摸精度和触摸点

本文主要告诉大家如何获得所有的触摸设备的触摸精度和触摸点数。

<!--more-->
<!-- CreateTime:2018/5/4 21:11:51 -->

<!-- csdn -->
<!-- 标签：WPF，触摸 -->

需要通过反射的方法才可以拿到触摸的精度。

使用 Tablet.TabletDevices 可以获得所有的触摸设备，获得触摸点数可以通过下面代码

```csharp
foreach (TabletDevice device in Tablet.TabletDevices)
{
	Console.WriteLine("触摸点数" + device.StylusDevices.Count);//触摸点数
}
```

触摸精度就需要使用反射

```csharp
        var builder = new StringBuilder();
                foreach (TabletDevice device in Tablet.TabletDevices)
                {
                    var deviceProperty = typeof(TabletDevice).GetProperty("TabletDeviceImpl",
                        BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.GetProperty);
                    var deviceImpl = deviceProperty is null ? device : deviceProperty.GetValue(device);
                    var info = deviceImpl.GetType().GetProperty("TabletSize",
                        BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.GetProperty);

                    var tabletSize = (Size) info.GetValue(deviceImpl, null);
                    if (device.Type == TabletDeviceType.Touch)
                    {
                        builder.Append(string.Format("{1}：{2} 点触摸，精度 {3}{0}", Environment.NewLine,
                            device.Name, device.StylusDevices.Count, tabletSize));
                    }
                    else
                    {
                        builder.Append(string.Format("{1}：{2} 个触笔设备，精度 {3}{0}", Environment.NewLine,
                            device.Name, device.StylusDevices.Count, tabletSize));
                    }
                }
```

如果发现设备触摸失效，可以使用这个项目

[ManipulationDemo](https://github.com/walterlv/ManipulationDemo )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
