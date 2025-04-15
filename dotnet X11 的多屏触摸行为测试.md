# dotnet X11 的多屏触摸行为测试

故事的背景是我在给 Avalonia 加上触摸尺寸的支持时，代码审查过程中大佬提出了在多屏上的 X11 行为问题，为此我找了两个触摸屏进行测试 X11 的多屏触摸行为。由于我的设备有限，本文只记录我所测试到的行为

<!--more-->
<!-- CreateTime:2024/07/31 07:19:20 -->

<!-- 发布 -->
<!-- 博客 -->

给 Avalonia 加上触摸尺寸支持的功能的代码： <https://github.com/AvaloniaUI/Avalonia/pull/16498>

## 基础环境

本次测试是我在麒麟 Kylin 系统搭配 [CVT](https://devicehunt.com/view/type/usb/vendor/1FF7) 厂商的双屏进行测试

在我的设备上使用 `cat /etc/.kyinfo` 获取麒麟系统的版本的输出信息如下

```
[dist]
name=Kylin
milestone=Desktop-V10-SP1-General-Release-TSM-lindexi-20230217
arch=arm64
beta=False
time=2023-02-17 19:01:29
```

根据 [定昌电子](https://www.gzdcsmt.com/sys-nd/1193.html ) 记录的文档，这里的 Desktop V10 SP1 General Release 版本就是银河麒麟桌面操作系统V10 SP1版本

运行 `uname -r` 的输出如下

```
>$ uname -r
5.4.18-53sy01-generic
```

在麒麟系统上运行 `cat /etc/debian_version` 获取 debian 版本号，输出信息如下

```
>$ cat /etc/debian_version
bullseye/sid
```

bullseye 是 debian 11 的发布代号，详细请看 <https://www.debian.org/releases/bullseye/>

## 获取基础信息

输入 xrandr 获取屏幕信息，可见内容如下

```
Screen 0: minimum 320 x 200, current 6240 x 2160, maximum 16384 x 16384
HDMI-A-0 disconnected (normal left inverted right x axis y axis)
HDMI-A-1 disconnected (normal left inverted right x axis y axis)
HDMI-A-2 connected primary 3840x2160+0+0 (normal left inverted right x axis y axis) 708mm x 398mm
   3840x2160     60.00*+  50.00    59.94    30.00    25.00    24.00    29.97    23.98
   1920x1200     60.00
   1920x1080     60.00    60.00    50.00    59.94    24.00    23.98
   1600x1200     60.00
   1680x1050     59.88
   1280x1024     60.02
   1440x900      60.00
   1280x960      60.00
   1280x800      59.91
   1280x720      60.00    50.00    59.94
   1024x768      60.00
   800x600       60.32    56.25
   720x576       50.00
   720x480       60.00    59.94
   640x480       60.00    59.94
   720x400       70.08
HDMI-A-3 connected 2400x2160+3840+0 (normal left inverted right x axis y axis) 708mm x 398mm
   2400x2160     59.99*+
   3840x2160     60.00    50.00    59.94    30.00    25.00    24.00    29.97    23.98
   1920x1200     59.99
   1920x1080     60.00    60.00    50.00    59.94    24.00    23.98
   1600x1200     59.99
   1680x1050     59.88
   1280x1024     60.02
   1440x900      59.99
   1200x1080     59.90
   1280x960      60.00
   1280x800      59.91
   1280x720      60.00    50.00    59.94
   1024x768      60.00
   800x600       60.32    56.25
   720x576       50.00
   720x480       60.00    59.94
   640x480       60.00    59.94
   720x400       70.08
```

以上代码的 `Screen 0: minimum 320 x 200, current 6240 x 2160, maximum 16384 x 16384` 中的 `current 6240 x 2160` 就是对应 XDisplayWidth 和 XDisplayHeight 所获取的值，如以下代码所示

```csharp
var display = XOpenDisplay(IntPtr.Zero);
var screen = XDefaultScreen(display);

var xDisplayWidth = XDisplayWidth(display, screen);
var xDisplayHeight = XDisplayHeight(display, screen);

var width = xDisplayWidth;
var height = xDisplayHeight;

Console.WriteLine($"WH={width},{height}");
```

在我的设备上执行以上代码，可以看到如下输出

```
WH=6240,2160
```

由此可见 Screen 的总宽度是多个触摸屏的外接矩形，如下字符图所示

```
   <----- XDisplayWidth ------->
↑┌---------------------------┐
││┌-------┐                │
│││       │                │
 H││ HDMI-1│                │
 E││       │                │
 I││       │     ┌-------┐│
 G│└-------┘     │       ││
 H│                │ HDMI-2││
 T│                │       ││
││                │       ││
││                └-------┘│
↓└---------------------------┘
```



## 触摸宽度高度

使用如下代码查询主设备获取的信息如下

```csharp
    var devices = (XIDeviceInfo*) XIQueryDevice(display,
        (int) XiPredefinedDeviceId.XIAllMasterDevices, out int num);

    XIDeviceInfo? pointerDevice = default;
    for (var c = 0; c < num; c++)
    {
        Console.WriteLine($"XIDeviceInfo [{c}] {devices[c].Deviceid} {devices[c].Use}");

        if (devices[c].Use == XiDeviceType.XIMasterPointer)
        {
            pointerDevice = devices[c];
            break;
        }
    }

    if (pointerDevice != null)
    {
        var multiTouchEventTypes = new List<XiEventType>
        {
            XiEventType.XI_TouchBegin,
            XiEventType.XI_TouchUpdate,
            XiEventType.XI_TouchEnd,

            XiEventType.XI_Motion,
            XiEventType.XI_ButtonPress,
            XiEventType.XI_ButtonRelease,
            XiEventType.XI_Leave,
            XiEventType.XI_Enter,
        };

        XiSelectEvents(display, handle, new Dictionary<int, List<XiEventType>> { [pointerDevice.Value.Deviceid] = multiTouchEventTypes });

        for (int i = 0; i < pointerDevice.Value.NumClasses; i++)
        {
            var xiAnyClassInfo = pointerDevice.Value.Classes[i];
            if (xiAnyClassInfo->Type == XiDeviceClass.XIValuatorClass)
            {
                valuators.Add(*((XIValuatorClassInfo**) pointerDevice.Value.Classes)[i]);
            }
            else if (xiAnyClassInfo->Type == XiDeviceClass.XIScrollClass)
            {
                scrollers.Add(*((XIScrollClassInfo**) pointerDevice.Value.Classes)[i]);
            }
        }

        foreach (var xiValuatorClassInfo in valuators)
        {
            if (xiValuatorClassInfo.Label == touchMajorAtom)
            {
                Console.WriteLine($"TouchMajorAtom Max={xiValuatorClassInfo.Max:0.00}; Min={xiValuatorClassInfo.Min:0.00}; Resolution={xiValuatorClassInfo.Resolution}");
            }
            else if (xiValuatorClassInfo.Label == touchMinorAtom)
            {
                Console.WriteLine($"TouchMinorAtom Max={xiValuatorClassInfo.Max:0.00}; Min={xiValuatorClassInfo.Min:0.00}; Resolution={xiValuatorClassInfo.Resolution}");
            }
        }
    }
```

在我的设备上运行以上代码的输出信息如下

```
TouchMajorAtom Max=18950.00; Min=0.00; Resolution=10000
TouchMinorAtom Max=10660.00; Min=0.00; Resolution=10000
```

以上的 `Max=18950.00` 所获取的值是逻辑值，但刚好在 [CVT](https://devicehunt.com/view/type/usb/vendor/1FF7) 设备上逻辑值和物理值都是相同的最大值最小值，因此可将其直接等同于其物理尺寸，即 1.89 米宽度。这里的 Resolution 是一个比例，计算公式如下

```
TouchMajorAtomMax/Resolution = 18950.00/10000 = 1.895 米
```

<!-- 由于 [CVT](https://devicehunt.com/view/type/usb/vendor/1FF7) 的设备报告的逻辑值和物理值都是相同的最大值最小值，因此以上代码我不确定拿到的是逻辑值还是物理值 -->

通过以上代码也可以看到，我无法直接获取到正确的多屏不同尺寸的设备的多个屏幕的物理尺寸。这是因为我无法直接知道输入的是哪个屏幕以及其比例值

但像素值倒是很好获取到，只需获取到其触摸点报告的 TouchMajor 值，与 TouchMajorXIValuatorClassInfo 的最大值和最小值相比，压缩到 [0-1] 范围内，再乘以屏幕像素，即可获取到其像素值

```
(TouchMajorValuatorValue - TouchMajorXIValuatorClassInfo.Min)/(TouchMajorXIValuatorClassInfo.Max - TouchMajorXIValuatorClassInfo.Min) * Monitor.Width
```

以上伪代码的 `Monitor.Width` 指的是对应屏幕的像素宽度。由于 Min 常是 0 因此在计算中常被忽略

但值得一提的是在 X11 里面，根据 <https://www.kernel.org/doc/html/latest/input/multi-touch-protocol.html> 文档，所获取的是椭圆长轴，将其当成触摸宽度是不准确的

## 校准屏幕

在我的设备上，发现触摸屏的触摸输入和对应的屏幕显示没有对齐，需要根据以下大佬们的博客进行修复

- [Linux处理多触屏的终极解决方案 香风家的火柴盒](https://www.small09.top/posts/210824-linuxtouchscreenfin/ ) 
- [【图形显示】扩展屏模式，触摸点较准不准确_90-touchscreen-map-CSDN博客](https://blog.csdn.net/qq_15770331/article/details/110532735 )

## 具体输入行为测试

我使用了相同的物理面积的物体触摸屏幕，两个屏幕分别是 `3840x2160` 和 `2400x2160` 分辨率

触摸左边 `3840x2160` 屏幕，获取到的 TouchMajorValuatorValue 是大概 100 的值。触摸右边 `2400x2160` 屏幕，获取到的 TouchMajorValuatorValue 是大概 160 的值

分别求像素大小：

- 左边屏幕： `100/18950*3840=20.2638522427`
- 右边屏幕： `160/18950*2400=20.2638522427`


## 代码

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/dedfc0ec3a3c8d04e7bec5276fe5bcaa926fe6e9/X11/DacemciwarjurqofuKearwihiyi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/dedfc0ec3a3c8d04e7bec5276fe5bcaa926fe6e9/X11/DacemciwarjurqofuKearwihiyi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin dedfc0ec3a3c8d04e7bec5276fe5bcaa926fe6e9
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin dedfc0ec3a3c8d04e7bec5276fe5bcaa926fe6e9
```

获取代码之后，进入 X11/DacemciwarjurqofuKearwihiyi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )