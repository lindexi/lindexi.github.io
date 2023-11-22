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

如果发现设备触摸失效，可以使用这个项目进行辅助调试

[https://github.com/dotnet-campus/ManipulationDemo](https://github.com/dotnet-campus/ManipulationDemo )