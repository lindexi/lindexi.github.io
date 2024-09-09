# dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件

本文记录我读 WPF 源代码的笔记，本文将介绍在 WPF 底层是如何从 Win32 的消息循环里获取到的 WM_POINTER 消息处理转换作为 Touch 事件的参数

<!--more-->
<!-- CreateTime:2024/09/01 07:15:29 -->
<!-- 置顶1 -->
<!-- 发布 -->
<!-- 博客 -->

由于 WPF 触摸部分会兼顾开启 Pointer 消息和不开启 Pointer 消息，在 WPF 框架里面的逻辑会有部分是兼容逻辑，为了方便大家理解，本文分为两个部分。第一个部分是脱离 WPF 框架，聊聊一个 Win32 程序如何从 Win32 的消息循环获取到的 WM_POINTER 消息处理转换为输入坐标点，以及在触摸下获取触摸信息。第二部分是 WPF 框架是如何安排上这些处理逻辑，如何和 WPF 框架的进行对接

第一部分脱离了 WPF 框架，也就没有了兼容不开启 Pointer 消息的负担，我将使用简单的描述点出关键部分

## 处理 Pointer 消息

在 Win32 应用程序中，大概有三个方式来进行对 Pointer 消息进行处理。我将从简单到复杂和大家讲述这三个方式

方式1:

接收到 WM_POINTER 消息之后，将 wparam 转换为 `pointerId` 参数，调用 GetPointerTouchInfo 方法即可获取到 [POINTER_INFO](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_info) 信息

获取 [POINTER_INFO](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_info) 的 `ptPixelLocationRaw` 字段，即可拿到基于屏幕坐标系的像素点

只需将其转换为窗口坐标系和处理 DPI 即可使用

此方法的最大缺点在于 `ptPixelLocationRaw` 字段拿到的是丢失精度的点，像素为单位。如果在精度稍微高的触摸屏下，将会有明显的锯齿效果

优点在于其获取特别简单

方式2：

依然是接收到 WM_POINTER 消息之后，将 wparam 转换为 `pointerId` 参数，调用 GetPointerTouchInfo 方法即可获取到 [POINTER_INFO](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_info) 信息

只是从获取 [POINTER_INFO](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_info) 的 `ptPixelLocationRaw` 字段换成 `ptHimetricLocationRaw` 字段

使用 `ptHimetricLocationRaw` 字段的优势在于可以获取不丢失精度的信息，但需要额外调用 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 函数获取 `displayRect` 和 `pointerDeviceRect` 信息用于转换坐标点，转换逻辑如以下代码所示

```csharp
            PInvoke.GetPointerDeviceRects(pointerInfo.sourceDevice, &pointerDeviceRect, &displayRect);

            // 如果想要获取比较高精度的触摸点，可以使用 ptHimetricLocationRaw 字段
            // 由于 ptHimetricLocationRaw 采用的是 pointerDeviceRect 坐标系，需要转换到屏幕坐标系
            // 转换方法就是先将 ptHimetricLocationRaw 的 X 坐标，压缩到 [0-1] 范围内，然后乘以 displayRect 的宽度，再加上 displayRect 的 left 值，即得到了屏幕坐标系的 X 坐标。压缩到 [0-1] 范围内的方法就是除以 pointerDeviceRect 的宽度
            // 为什么需要加上 displayRect.left 的值？考虑多屏的情况，屏幕可能是副屏
            // Y 坐标同理
            var point2D = new Point2D(
                pointerInfo.ptHimetricLocationRaw.X / (double) pointerDeviceRect.Width * displayRect.Width +
                displayRect.left,
                pointerInfo.ptHimetricLocationRaw.Y / (double) pointerDeviceRect.Height * displayRect.Height +
                displayRect.top);

            // 获取到的屏幕坐标系的点，需要转换到 WPF 坐标系
            // 转换过程的两个重点：
            // 1. 底层 ClientToScreen 只支持整数类型，直接转换会丢失精度。即使是 WPF 封装的 PointFromScreen 或 PointToScreen 方法也会丢失精度
            // 2. 需要进行 DPI 换算，必须要求 DPI 感知

            // 先测量窗口与屏幕的偏移量，这里直接取 0 0 点即可，因为这里获取到的是虚拟屏幕坐标系，不需要考虑多屏的情况
            var screenTranslate = new Point(0, 0);
            PInvoke.ClientToScreen(new HWND(hwnd), ref screenTranslate);
            // 获取当前的 DPI 值
            var dpi = VisualTreeHelper.GetDpi(this);
            // 先做平移，再做 DPI 换算
            point2D = new Point2D(point2D.X - screenTranslate.X, point2D.Y - screenTranslate.Y);
            point2D = new Point2D(point2D.X / dpi.DpiScaleX, point2D.Y / dpi.DpiScaleY);
```

以上方式2的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/322313ee55d0eeaae7148b24ca279e1df087871e/WPFDemo/DefilireceHowemdalaqu) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/322313ee55d0eeaae7148b24ca279e1df087871e/WPFDemo/DefilireceHowemdalaqu) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 322313ee55d0eeaae7148b24ca279e1df087871e
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 322313ee55d0eeaae7148b24ca279e1df087871e
```

获取代码之后，进入 WPFDemo/DefilireceHowemdalaqu 文件夹，即可获取到源代码

方式2的优点在于可以获取到更高的精度。缺点是相对来说比较复杂，需要多了点点处理

方式3：

此方式会更加复杂，但功能能够更加全面，适合用在要求更高控制的应用里面

先调用 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 方法，获取 HID 描述符上报的对应设备属性，此时可以获取到的是具备完全的 HID 描述符属性的方法，可以包括 [Windows 的 Pen 协议](https://blog.lindexi.com/post/Windows-%E7%9A%84-Pen-%E5%8D%8F%E8%AE%AE.html ) 里面列举的各个属性，如宽度高度旋转角等信息

收到 WM_POINTER 消息时，调用 [GetRawPointerDeviceData](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getrawpointerdevicedata) 获取最原始的触摸信息，再对原始触摸信息进行解析处理

原始触摸信息的解析处理需要先应用获取每个触摸点的数据包长度，再拆数据包。原始触摸信息拿到的是一个二进制数组，这个二进制数组里面可能包含多个触摸点的信息，需要根据数据包长度拆分为多个触摸点信息

解析处理就是除了前面两个分别是属于 X 和 Y 之外，后面的数据就根据 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 方法获取到的触摸描述信息进行套入

此方式的复杂程度比较高，且拿到的是原始的触摸信息，需要做比较多的处理。即使解析到 X 和 Y 坐标点之后，还需要执行坐标的转换，将其转换为屏幕坐标系

这里拿到的 X 和 Y 坐标点是设备坐标系，这里的设备坐标系不是 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 函数获取 的 `pointerDeviceRect` 设备范围坐标系，而是对应 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 方法获取到的描述符的逻辑最大值和最小值的坐标范围

其正确计算方法为从 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 方法获取到的 X 和 Y 描述信息，分别取 [POINTER_DEVICE_PROPERTY](https://learn.microsoft.com/en-us/windows/win32/api/winuser/ns-winuser-pointer_device_property) 的 `logicalMax` 作为最大值范围。分别将 X 和 Y 除以 `logicalMax` 缩放到 `[0,1]` 范围内，再乘以屏幕尺寸即可转换为屏幕坐标系

这里的 屏幕尺寸 是通过 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 函数获取 的 `displayRect` 尺寸

转换为屏幕坐标系之后，就需要再次处理 DPI 和转换为窗口坐标系的才能使用

可以看到方式3相对来说还是比较复杂的，但其优点是可以获取到更多的设备描述信息，获取到输入点的更多信息，如可以计算出触摸宽度对应的物理触摸尺寸面积等信息

对于 WPF 框架来说，自然是选最复杂且功能全强的方法了

## 在 WPF 框架的对接

了解了一个 Win32 应用与 WM_POINTER 消息的对接方式，咱来看看 WPF 具体是如何做的。了解了对接方式之后，阅读 WPF 源代码的方式可以是通过必须调用的方法的引用，找到整个 WPF 的脉络

在开始之前必须说明的是，本文的大部分代码都是有删减的代码，只保留和本文相关的部分。现在 WPF 是完全开源的，基于最友好的 MIT 协议，可以自己拉下来代码进行二次修改发布，想看完全的代码和调试整个过程可以自己从开源地址拉取整个仓库下来，开源地址是： <https://github.com/dotnet/wpf>

本文以下部分仅为开启 WM_POINTER 消息时的 WPF 框架对接的逻辑，如对不开启 WM_POINTER 支持时的逻辑感兴趣，还请参阅 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html )

在 WPF 里面，触摸初始化的故事开始是在 `PointerTabletDeviceCollection.cs` 里面，调用 [GetPointerDevices](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevices ) 方法进行初始化获取设备数量，之后的每个设备都调用 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 方法，获取 HID 描述符上报的对应设备属性，有删减的代码如下

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// Maintains a collection of pointer device information for currently installed pointer devices
    /// </summary>
    internal class PointerTabletDeviceCollection : TabletDeviceCollection
    {
        internal void Refresh()
        {
            ... // 忽略其他代码
                    UnsafeNativeMethods.POINTER_DEVICE_INFO[] deviceInfos
                         = new UnsafeNativeMethods.POINTER_DEVICE_INFO[deviceCount];

                    IsValid = UnsafeNativeMethods.GetPointerDevices(ref deviceCount, deviceInfos);
            ... // 忽略其他代码
        }
    }
}
```

获取到设备之后，将其转换放入到 WPF 定义的 PointerTabletDevice 里面，大概的代码如下

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// Maintains a collection of pointer device information for currently installed pointer devices
    /// </summary>
    internal class PointerTabletDeviceCollection : TabletDeviceCollection
    {
        internal void Refresh()
        {
            ... // 忽略其他代码
                    UnsafeNativeMethods.POINTER_DEVICE_INFO[] deviceInfos
                         = new UnsafeNativeMethods.POINTER_DEVICE_INFO[deviceCount];

                    IsValid = UnsafeNativeMethods.GetPointerDevices(ref deviceCount, deviceInfos);

                    if (IsValid)
                    {
                        foreach (var deviceInfo in deviceInfos)
                        {
                            // Old PenIMC code gets this id via a straight cast from COM pointer address
                            // into an int32.  This does a very similar thing semantically using the pointer
                            // to the tablet from the WM_POINTER stack.  While it may have similar issues
                            // (chopping the upper bits, duplicate ids) we don't use this id internally
                            // and have never received complaints about this in the WISP stack.
                            int id = MS.Win32.NativeMethods.IntPtrToInt32(deviceInfo.device);

                            PointerTabletDeviceInfo ptdi = new PointerTabletDeviceInfo(id, deviceInfo);

                            // Don't add a device that fails initialization.  This means we will try a refresh
                            // next time around if we receive stylus input and the device is not available.
                            // <see cref="HwndPointerInputProvider.UpdateCurrentTabletAndStylus">
                            if (ptdi.TryInitialize())
                            {
                                PointerTabletDevice tablet = new PointerTabletDevice(ptdi);

                                _tabletDeviceMap[tablet.Device] = tablet;
                                TabletDevices.Add(tablet.TabletDevice);
                            }
                        }
                    }
            ... // 忽略其他代码
        }

        /// <summary>
        /// Holds a mapping of TabletDevices from their WM_POINTER device id
        /// </summary>
        private Dictionary<IntPtr, PointerTabletDevice> _tabletDeviceMap = new Dictionary<IntPtr, PointerTabletDevice>();
    }
}

namespace System.Windows.Input
{
    /// <summary>
    ///     Collection of the tablet devices that are available on the machine.
    /// </summary>
    public class TabletDeviceCollection : ICollection, IEnumerable
    {
        internal List<TabletDevice> TabletDevices { get; set; } = new List<TabletDevice>();
    }
}
```

在 PointerTabletDeviceInfo 的 TryInitialize 方法，即 `if (ptdi.TryInitialize())` 这行代码里面，将会调用 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 获取设备属性信息，其代码逻辑如下

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// WM_POINTER specific information about a TabletDevice
    /// </summary>
    internal class PointerTabletDeviceInfo : TabletDeviceInfo
    {
        internal PointerTabletDeviceInfo(int id, UnsafeNativeMethods.POINTER_DEVICE_INFO deviceInfo)
        {
            _deviceInfo = deviceInfo;

            Id = id;
            Name = _deviceInfo.productString;
            PlugAndPlayId = _deviceInfo.productString;
        }

        internal bool TryInitialize()
        {
            ... // 忽略其他代码

            var success = TryInitializeSupportedStylusPointProperties();

            ... // 忽略其他代码

            return success;
        }

        private bool TryInitializeSupportedStylusPointProperties()
        {
            bool success = false;

            ... // 忽略其他代码

            // Retrieve all properties from the WM_POINTER stack
            success = UnsafeNativeMethods.GetPointerDeviceProperties(Device, ref propCount, null);

            if (success)
            {
                success = UnsafeNativeMethods.GetPointerDeviceProperties(Device, ref propCount, SupportedPointerProperties);

                if (success)
                {
                    ... // 执行更具体的初始化逻辑
                }
            }

            ... // 忽略其他代码
        }

        /// <summary>
        /// The specific id for this TabletDevice
        /// </summary>
        internal IntPtr Device { get { return _deviceInfo.device; } }

        /// <summary>
        /// Store the WM_POINTER device information directly
        /// </summary>
        private UnsafeNativeMethods.POINTER_DEVICE_INFO _deviceInfo;
    }
}
```

为什么这里会调用 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 两次？第一次只是拿数量，第二次才是真正的拿值

回顾以上代码，可以看到 PointerTabletDeviceInfo 对象是在 PointerTabletDeviceCollection 的 Refresh 方法里面创建的，如以下代码所示

```csharp
    internal class PointerTabletDeviceCollection : TabletDeviceCollection
    {
        internal void Refresh()
        {
            ... // 忽略其他代码
                    UnsafeNativeMethods.POINTER_DEVICE_INFO[] deviceInfos
                         = new UnsafeNativeMethods.POINTER_DEVICE_INFO[deviceCount];

                    IsValid = UnsafeNativeMethods.GetPointerDevices(ref deviceCount, deviceInfos);
                        foreach (var deviceInfo in deviceInfos)
                        {
                            // Old PenIMC code gets this id via a straight cast from COM pointer address
                            // into an int32.  This does a very similar thing semantically using the pointer
                            // to the tablet from the WM_POINTER stack.  While it may have similar issues
                            // (chopping the upper bits, duplicate ids) we don't use this id internally
                            // and have never received complaints about this in the WISP stack.
                            int id = MS.Win32.NativeMethods.IntPtrToInt32(deviceInfo.device);

                            PointerTabletDeviceInfo ptdi = new PointerTabletDeviceInfo(id, deviceInfo);

                            if (ptdi.TryInitialize())
                            {
                                
                            }
                        }
            ... // 忽略其他代码
        }
    }
```

从 GetPointerDevices 获取到的 `POINTER_DEVICE_INFO` 信息会存放在 `PointerTabletDeviceInfo` 的 `_deviceInfo` 字段里面，如下面代码所示

```csharp
    internal class PointerTabletDeviceInfo : TabletDeviceInfo
    {
        internal PointerTabletDeviceInfo(int id, UnsafeNativeMethods.POINTER_DEVICE_INFO deviceInfo)
        {
            _deviceInfo = deviceInfo;

            Id = id;
        }

        /// <summary>
        /// The specific id for this TabletDevice
        /// </summary>
        internal IntPtr Device { get { return _deviceInfo.device; } }

        /// <summary>
        /// Store the WM_POINTER device information directly
        /// </summary>
        private UnsafeNativeMethods.POINTER_DEVICE_INFO _deviceInfo;
    }
```

调用 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 时，就会将 `POINTER_DEVICE_INFO` 的 `device` 字段作为参数传入，从而获取到 `POINTER_DEVICE_PROPERTY` 结构体列表信息

获取到的 `POINTER_DEVICE_PROPERTY` 结构体信息和 HID 描述符上报的信息非常对应。结构体的定义代码大概如下

```csharp
        /// <summary>
        /// A struct representing the information for a particular pointer property.
        /// These correspond to the raw data from WM_POINTER.
        /// </summary>
        [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
        internal struct POINTER_DEVICE_PROPERTY
        {
            internal Int32 logicalMin;
            internal Int32 logicalMax;
            internal Int32 physicalMin;
            internal Int32 physicalMax;
            internal UInt32 unit;
            internal UInt32 unitExponent;
            internal UInt16 usagePageId;
            internal UInt16 usageId;
        }
```

根据 HID 基础知识可以知道，通过 `usagePageId` 和 `usageId` 即可了解到此设备属性的具体含义。更多请参阅 HID 标准文档： <http://www.usb.org/developers/hidpage/Hut1_12v2.pdf>

在 WPF 使用到的 Pointer 的 `usagePageId` 的只有以下枚举所列举的值

```csharp
        /// <summary>
        ///
        /// WM_POINTER stack must parse out HID spec usage pages
        /// <see cref="http://www.usb.org/developers/hidpage/Hut1_12v2.pdf"/> 
        /// </summary>
        internal enum HidUsagePage
        {
            Undefined = 0x00,
            Generic = 0x01,
            Simulation = 0x02,
            Vr = 0x03,
            Sport = 0x04,
            Game = 0x05,
            Keyboard = 0x07,
            Led = 0x08,
            Button = 0x09,
            Ordinal = 0x0a,
            Telephony = 0x0b,
            Consumer = 0x0c,
            Digitizer = 0x0d,
            Unicode = 0x10,
            Alphanumeric = 0x14,
            BarcodeScanner = 0x8C,
            WeighingDevice = 0x8D,
            MagneticStripeReader = 0x8E,
            CameraControl = 0x90,
            MicrosoftBluetoothHandsfree = 0xfff3,
        }
```

在 WPF 使用到的 Pointer 的 `usageId` 的只有以下枚举所列举的值

```csharp
       /// <summary>
       ///
       /// 
       /// WISP pre-parsed these, WM_POINTER stack must do it itself
       /// 
       /// See Stylus\biblio.txt - 1
       /// <see cref="http://www.usb.org/developers/hidpage/Hut1_12v2.pdf"/> 
       /// </summary>
       internal enum HidUsage
       {
           TipPressure = 0x30,
           X = 0x30,
           BarrelPressure = 0x31,
           Y = 0x31,
           Z = 0x32,
           XTilt = 0x3D,
           YTilt = 0x3E,
           Azimuth = 0x3F,
           Altitude = 0x40,
           Twist = 0x41,
           TipSwitch = 0x42,
           SecondaryTipSwitch = 0x43,
           BarrelSwitch = 0x44,
           TouchConfidence = 0x47,
           Width = 0x48,
           Height = 0x49,
           TransducerSerialNumber = 0x5B,
       }
```

在 WPF 的古老版本里面，约定了使用 GUID 去获取 StylusPointDescription 里面的额外数据信息。为了与此行为兼容，在 WPF 里面就定义了 HidUsagePage 和 HidUsage 与 GUID 的对应关系，实现代码如下

```csharp
namespace System.Windows.Input
{
    /// <summary>
    /// StylusPointPropertyIds
    /// </summary>
    /// <ExternalAPI/>
    internal static class StylusPointPropertyIds
    {
        /// <summary>
        /// The x-coordinate in the tablet coordinate space.
        /// </summary>
        /// <ExternalAPI/>
        public static readonly Guid X = new Guid(0x598A6A8F, 0x52C0, 0x4BA0, 0x93, 0xAF, 0xAF, 0x35, 0x74, 0x11, 0xA5, 0x61);
        /// <summary>
        /// The y-coordinate in the tablet coordinate space.
        /// </summary>
        /// <ExternalAPI/>
        public static readonly Guid Y = new Guid(0xB53F9F75, 0x04E0, 0x4498, 0xA7, 0xEE, 0xC3, 0x0D, 0xBB, 0x5A, 0x90, 0x11);

        public static readonly Guid Z = ...

        ...

        /// <summary>
        ///
        /// WM_POINTER stack usage preparation based on associations maintained from the legacy WISP based stack
        /// </summary>
        private static Dictionary<HidUsagePage, Dictionary<HidUsage, Guid>> _hidToGuidMap = new Dictionary<HidUsagePage, Dictionary<HidUsage, Guid>>()
        {
            { HidUsagePage.Generic,
                new Dictionary<HidUsage, Guid>()
                {
                    { HidUsage.X, X },
                    { HidUsage.Y, Y },
                    { HidUsage.Z, Z },
                }
            },
            { HidUsagePage.Digitizer,
                new Dictionary<HidUsage, Guid>()
                {
                    { HidUsage.Width, Width },
                    { HidUsage.Height, Height },
                    { HidUsage.TouchConfidence, SystemTouch },
                    { HidUsage.TipPressure, NormalPressure },
                    { HidUsage.BarrelPressure, ButtonPressure },
                    { HidUsage.XTilt, XTiltOrientation },
                    { HidUsage.YTilt, YTiltOrientation },
                    { HidUsage.Azimuth, AzimuthOrientation },
                    { HidUsage.Altitude, AltitudeOrientation },
                    { HidUsage.Twist, TwistOrientation },
                    { HidUsage.TipSwitch, TipButton },
                    { HidUsage.SecondaryTipSwitch, SecondaryTipButton },
                    { HidUsage.BarrelSwitch, BarrelButton },
                    { HidUsage.TransducerSerialNumber, SerialNumber },
                }
            },
        };

        /// <summary>
        /// Retrieves the GUID of the stylus property associated with the usage page and usage ids
        /// within the HID specification.
        /// </summary>
        /// <param name="page">The usage page id of the HID specification</param>
        /// <param name="usage">The usage id of the HID specification</param>
        /// <returns>
        /// If known, the GUID associated with the usagePageId and usageId.
        /// If not known, GUID.Empty
        /// </returns>
        internal static Guid GetKnownGuid(HidUsagePage page, HidUsage usage)
        {
            Guid result = Guid.Empty;

            Dictionary<HidUsage, Guid> pageMap = null;

            if (_hidToGuidMap.TryGetValue(page, out pageMap))
            {
                pageMap.TryGetValue(usage, out result);
            }

            return result;
        }
    }
}
```

通过以上的 `_hidToGuidMap` 的定义关联关系，调用 GetKnownGuid 方法，即可将 `POINTER_DEVICE_PROPERTY` 描述信息关联到 WPF 框架层的定义

具体的对应逻辑如下

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// Contains a WM_POINTER specific functions to parse out stylus property info
    /// </summary>
    internal class PointerStylusPointPropertyInfoHelper
    {
        /// <summary>
        /// Creates WPF property infos from WM_POINTER device properties.  This appropriately maps and converts HID spec
        /// properties found in WM_POINTER to their WPF equivalents.  This is based on code from the WISP implementation
        /// that feeds the legacy WISP based stack.
        /// </summary>
        /// <param name="prop">The pointer property to convert</param>
        /// <returns>The equivalent WPF property info</returns>
        internal static StylusPointPropertyInfo CreatePropertyInfo(UnsafeNativeMethods.POINTER_DEVICE_PROPERTY prop)
        {
            StylusPointPropertyInfo result = null;

            // Get the mapped GUID for the HID usages
            Guid propGuid =
                StylusPointPropertyIds.GetKnownGuid(
                    (StylusPointPropertyIds.HidUsagePage)prop.usagePageId,
                    (StylusPointPropertyIds.HidUsage)prop.usageId);

            if (propGuid != Guid.Empty)
            {
                StylusPointProperty stylusProp = new StylusPointProperty(propGuid, StylusPointPropertyIds.IsKnownButton(propGuid));

                // Set Units
                StylusPointPropertyUnit? unit = StylusPointPropertyUnitHelper.FromPointerUnit(prop.unit);

                // If the parsed unit is invalid, set the default
                if (!unit.HasValue)
                {
                    unit = StylusPointPropertyInfoDefaults.GetStylusPointPropertyInfoDefault(stylusProp).Unit;
                }

                // Set to default resolution
                float resolution = StylusPointPropertyInfoDefaults.GetStylusPointPropertyInfoDefault(stylusProp).Resolution;

                short mappedExponent = 0;

                if (_hidExponentMap.TryGetValue((byte)(prop.unitExponent & HidExponentMask), out mappedExponent))
                {
                    float exponent = (float)Math.Pow(10, mappedExponent);

                    // Guard against divide by zero or negative resolution
                    if (prop.physicalMax - prop.physicalMin > 0)
                    {
                        // Calculated resolution is a scaling factor from logical units into the physical space
                        // at the given exponentiation.
                        resolution =
                            (prop.logicalMax - prop.logicalMin) / ((prop.physicalMax - prop.physicalMin) * exponent);
                    }
                }

                result = new StylusPointPropertyInfo(
                      stylusProp,
                      prop.logicalMin,
                      prop.logicalMax,
                      unit.Value,
                      resolution);
            }

            return result;
        }
    }
}
```

以上的一个小细节点在于对 unit 单位的处理，即 `StylusPointPropertyUnit? unit = StylusPointPropertyUnitHelper.FromPointerUnit(prop.unit);` 这行代码的实现定义，具体实现如下

```csharp
    internal static class StylusPointPropertyUnitHelper
    {
        /// <summary>
        /// Convert WM_POINTER units to WPF units
        /// </summary>
        /// <param name="pointerUnit"></param>
        /// <returns></returns>
        internal static StylusPointPropertyUnit? FromPointerUnit(uint pointerUnit)
        {
            StylusPointPropertyUnit unit = StylusPointPropertyUnit.None;

            _pointerUnitMap.TryGetValue(pointerUnit & UNIT_MASK, out unit);

            return (IsDefined(unit)) ? unit : (StylusPointPropertyUnit?)null;
        }

        /// <summary>
        /// Mapping for WM_POINTER based unit, taken from legacy WISP code
        /// </summary>
        private static Dictionary<uint, StylusPointPropertyUnit> _pointerUnitMap = new Dictionary<uint, StylusPointPropertyUnit>()
        {
            { 1, StylusPointPropertyUnit.Centimeters },
            { 2, StylusPointPropertyUnit.Radians },
            { 3, StylusPointPropertyUnit.Inches },
            { 4, StylusPointPropertyUnit.Degrees },
        };

        /// <summary>
        /// Mask to extract units from raw WM_POINTER data
        /// <see cref="http://www.usb.org/developers/hidpage/Hut1_12v2.pdf"/> 
        /// </summary>
        private const uint UNIT_MASK = 0x000F;
    }
```

这里的单位的作用是什么呢？用于和 `POINTER_DEVICE_PROPERTY` 的物理值做关联对应关系，比如触摸面积 Width 和 Height 的物理尺寸就是通过大概如下算法计算出来的

```csharp
                short mappedExponent = 0;

                if (_hidExponentMap.TryGetValue((byte)(prop.unitExponent & HidExponentMask), out mappedExponent))
                {
                    float exponent = (float)Math.Pow(10, mappedExponent);

                    // Guard against divide by zero or negative resolution
                    if (prop.physicalMax - prop.physicalMin > 0)
                    {
                        // Calculated resolution is a scaling factor from logical units into the physical space
                        // at the given exponentiation.
                        resolution =
                            (prop.logicalMax - prop.logicalMin) / ((prop.physicalMax - prop.physicalMin) * exponent);
                    }
                }

        /// <summary>
        /// Contains the mappings from WM_POINTER exponents to our local supported values.
        /// This mapping is taken from WISP code, see Stylus\Biblio.txt - 4,
        /// as an array of HidExponents.
        /// </summary>
        private static Dictionary<byte, short> _hidExponentMap = new Dictionary<byte, short>()
        {
            { 5, 5 },
            { 6, 6 },
            { 7, 7 },
            { 8, -8 },
            { 9, -7 },
            { 0xA, -6 },
            { 0xB, -5 },
            { 0xC, -4 },
            { 0xD, -3 },
            { 0xE, -2 },
            { 0xF, -1 },
        };
```

通过 resolution 与具体后续收到的触摸点的值进行计算，带上 StylusPointPropertyUnit 单位，这就是触摸设备上报的物理尺寸了

以上 `logicalMax` 和 `logicalMin` 在行业内常被称为逻辑值，以上的 `physicalMax` 和 `physicalMin` 常被称为物理值

<!-- 许多触摸设备厂商，特别是大尺寸的红外框，都默认将逻辑值和物理值设置为相同的值 -->

经过以上的处理之后，即可将 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 拿到的设备属性列表给转换为 WPF 框架对应的定义属性内容

以上过程有一个细节，那就是 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 拿到的设备属性列表的顺序是非常关键的，设备属性列表的顺序和在后续 WM_POINTER 消息拿到的裸数据的顺序是直接对应的

大家可以看到，在开启 Pointer 消息时，触摸模块初始化获取触摸信息是完全通过 Win32 的 WM_POINTER 模块提供的相关方法完成的。这里需要和不开 WM_POINTER 消息的从 COM 获取触摸设备信息区分，和 [dotnet 读 WPF 源代码笔记 插入触摸设备的初始化获取设备信息](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-%E6%8F%92%E5%85%A5%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96%E8%8E%B7%E5%8F%96%E8%AE%BE%E5%A4%87%E4%BF%A1%E6%81%AF.html ) 提供的方法是不相同的

完成上述初始化逻辑之后，接下来看看消息循环收到 WM_POINTER 消息的处理

收到 WM_POINTER 消息时，调用 [GetRawPointerDeviceData](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getrawpointerdevicedata) 获取最原始的触摸信息，再对原始触摸信息进行解析处理

在 WPF 里面，大家都知道，底层的消息循环处理的在 `HwndSource.cs` 里面定义，输入处理部分如下

```csharp
namespace System.Windows.Interop
{
    /// <summary>
    ///     The HwndSource class presents content within a Win32 HWND.
    /// </summary>
    public class HwndSource : PresentationSource, IDisposable, IWin32Window, IKeyboardInputSink
    {
        private IntPtr InputFilterMessage(IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            ... // 忽略其他代码
            // NOTE (alexz): invoke _stylus.FilterMessage before _mouse.FilterMessage
            // to give _stylus a chance to eat mouse message generated by stylus
            if (!_isDisposed && _stylus != null && !handled)
            {
                result = _stylus.Value.FilterMessage(hwnd, message, wParam, lParam, ref handled);
            }
            ... // 忽略其他代码
        }

        private SecurityCriticalDataClass<IStylusInputProvider>        _stylus;
    }
}
```

以上代码的 `_stylus` 就是根据不同的配置参数决定是否使用 Pointer 消息处理的 HwndPointerInputProvider 类型，代码如下

```csharp
namespace System.Windows.Interop
{
    /// <summary>
    ///     The HwndSource class presents content within a Win32 HWND.
    /// </summary>
    public class HwndSource : PresentationSource, IDisposable, IWin32Window, IKeyboardInputSink
    {
        private void Initialize(HwndSourceParameters parameters)
        {
            ... // 忽略其他代码
            if (StylusLogic.IsStylusAndTouchSupportEnabled)
            {
                // Choose between Wisp and Pointer stacks
                if (StylusLogic.IsPointerStackEnabled)
                {
                    _stylus = new SecurityCriticalDataClass<IStylusInputProvider>(new HwndPointerInputProvider(this));
                }
                else
                {
                    _stylus = new SecurityCriticalDataClass<IStylusInputProvider>(new HwndStylusInputProvider(this));
                }
            }
            ... // 忽略其他代码
        }
    }
}
```

在本文这里初始化的是 HwndPointerInputProvider 类型，将会进入到 HwndPointerInputProvider 的 FilterMessage 方法处理输入数据

```csharp
namespace System.Windows.Interop
{
    /// <summary>
    /// Implements an input provider per hwnd for WM_POINTER messages
    /// </summary>
    internal sealed class HwndPointerInputProvider : DispatcherObject, IStylusInputProvider
    {
        /// <summary>
        /// Processes the message loop for the HwndSource, filtering WM_POINTER messages where needed
        /// </summary>
        /// <param name="hwnd">The hwnd the message is for</param>
        /// <param name="msg">The message</param>
        /// <param name="wParam"></param>
        /// <param name="lParam"></param>
        /// <param name="handled">If this has been successfully processed</param>
        /// <returns></returns>
        IntPtr IStylusInputProvider.FilterMessage(IntPtr hwnd, WindowMessage msg, IntPtr wParam, IntPtr lParam, ref bool handled)
        {
            handled = false;

            // Do not process any messages if the stack was disabled via reflection hack
            if (PointerLogic.IsEnabled)
            {
                switch (msg)
                {
                    case WindowMessage.WM_ENABLE:
                        {
                            IsWindowEnabled = MS.Win32.NativeMethods.IntPtrToInt32(wParam) == 1;
                        }
                        break;
                    case WindowMessage.WM_POINTERENTER:
                        {
                            // Enter can be processed as an InRange.  
                            // The MSDN documentation is not correct for InRange (according to feisu)
                            // As such, using enter is the correct way to generate this.  This is also what DirectInk uses.
                            handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.InRange, Environment.TickCount);
                        }
                        break;
                    case WindowMessage.WM_POINTERUPDATE:
                        {
                            handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.Move, Environment.TickCount);
                        }
                        break;
                    case WindowMessage.WM_POINTERDOWN:
                        {
                            handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.Down, Environment.TickCount);
                        }
                        break;
                    case WindowMessage.WM_POINTERUP:
                        {
                            handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.Up, Environment.TickCount);
                        }
                        break;
                    case WindowMessage.WM_POINTERLEAVE:
                        {
                            // Leave can be processed as an OutOfRange.  
                            // The MSDN documentation is not correct for OutOfRange (according to feisu)
                            // As such, using leave is the correct way to generate this.  This is also what DirectInk uses.
                            handled = ProcessMessage(GetPointerId(wParam), RawStylusActions.OutOfRange, Environment.TickCount);
                        }
                        break;
                }
            }

            return IntPtr.Zero;
        }

        ... // 忽略其他代码
    }
}
```

对于收到 Pointer 的按下移动抬起消息，都会进入到 ProcessMessage 方法

进入之前调用的 `GetPointerId(wParam)` 代码的 GetPointerId 方法实现如下

```csharp
        /// <summary>
        /// Extracts the pointer id
        /// </summary>
        /// <param name="wParam">The parameter containing the id</param>
        /// <returns>The pointer id</returns>
        private uint GetPointerId(IntPtr wParam)
        {
            return (uint)MS.Win32.NativeMethods.SignedLOWORD(wParam);
        }

    internal partial class NativeMethods
    {
        public static int SignedLOWORD(IntPtr intPtr)
        {
            return SignedLOWORD(IntPtrToInt32(intPtr));
        }

        public static int IntPtrToInt32(IntPtr intPtr)
        {
            return unchecked((int)intPtr.ToInt64());
        }

        public static int SignedLOWORD(int n)
        {
            int i = (int)(short)(n & 0xFFFF);

            return i;
        }
    }
```

当然了，以上代码简单写就和下面代码差不多

```csharp
            var pointerId = (uint) (ToInt32(wparam) & 0xFFFF);
```

在 WM_POINTER 的设计上，将会源源不断通过消息循环发送指针消息，发送的指针消息里面不直接包含具体的数据信息，而是只将 PointerId 当成 wparam 发送。咱从消息循环里面拿到的只有 PointerId 的值，转换方法如上述代码所示

为什么是这样设计的呢？考虑到现在大部分触摸屏的精度都不低，至少比许多很便宜鼠标的高，这就可能导致应用程序完全无法顶得住每次触摸数据过来都通过消息循环怼进来。在 WM_POINTER 的设计上，只是将 PointerId 通过消息循环发送过来，具体的消息体数据需要使用 [GetPointerInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfo) 方法来获取。这么设计有什么优势？这么设计是用来解决应用卡顿的时候，被堆积消息的问题。假定现在有三个触摸消息进来，第一个触摸消息进来就发送了 Win32 消息给到应用，然而应用等待到系统收集到了三个触摸点消息时，才调用 [GetPointerInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfo) 方法。那此时系统触摸模块就可以很开森的知道了应用处于卡顿状态，即第二个和第三个触摸消息到来时，判断第一个消息还没被应用消费，就不再发送 Win32 消息给到应用。当应用调用 [GetPointerInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfo) 方法时，就直接返回第三个点给到应用，跳过中间第一个和第二个触摸点。同时，使用历史点的概念，将第一个点和第二个点和第三个点给到应用，如果此时应用感兴趣的话

利用如上所述机制，即可实现到当触摸设备产生的触摸消息过快时，不会让应用的消息循环过度忙碌，而是可以让应用有机会一次性拿到过去一段时间内的多个触摸点信息。如此可以提升整体系统的性能，减少应用程序忙碌于处理过往的触摸消息

举一个虚拟的例子，让大家更好的理解这套机制的思想。假定咱在制作一个应用，应用有一个功能，就是有一个矩形元素，这个元素可以响应触摸拖动，可以用触摸拖动矩形元素。这个应用编写的有些离谱，每次拖动的做法就是设置新的坐标点为当前触摸点，但是这个过程需要 15 毫秒，因为中间添加了一些有趣且保密（其实我还没编出来）的算法。当应用跑在一个触摸设备上，这个触摸设备在触摸拖动的过程中，每 10 毫秒将产生一次触摸点信息报告给到系统。假定当前的系统的触摸模块是如实的每次收到设备发送过来的触摸点，都通过 Win32 消息发送给到应用，那将会让应用的消费速度慢于消息的生产速度，这就意味着大家可以明显看到拖动矩形元素时具备很大的延迟感。如拖着拖着才发现矩形元素还在后面慢慢挪动，整体的体验比较糟糕。那如果采用现在的这套玩法呢？应用程序从 Win32 消息收到的是 PointerId 信息，再通过 [GetPointerInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfo) 方法获取触摸点信息，此时获取到的触摸点就是最后一个触摸点，对于咱这个应用来说刚刚好，直接就是响应设置矩形元素坐标为最后一个触摸点的对应坐标。如此即可看到矩形元素飞快跳着走，且由于刚好矩形元素拖动过程为 15 毫秒，小于 16 毫秒，意味着大部分情况下大家看到的是矩形元素平滑的移动，即飞快跳着走在人类看来是一个连续移动的过程

期望通过以上的例子可以让大家了解到微软的“良苦”用心

<!-- DirectInk -->

这里需要额外说明的是 PointerId 和 TouchDevice 等的 Id 是不一样的，在下文将会给出详细的描述

在 WPF 这边，如上面代码所示，收到触摸点信息之后，将会进入到 ProcessMessage 方法，只是这个过程中我感觉有一点小锅的是，时间戳拿的是当前系统时间戳 Environment.TickCount 的值，而不是取 Pointer 消息里面的时间戳内容

继续看一下 ProcessMessage 方法的定义和实现

```csharp
namespace System.Windows.Interop
{
    /// <summary>
    /// Implements an input provider per hwnd for WM_POINTER messages
    /// </summary>
    internal sealed class HwndPointerInputProvider : DispatcherObject, IStylusInputProvider
    {
        /// <summary>
        /// Processes the latest WM_POINTER message and forwards it to the WPF input stack.
        /// </summary>
        /// <param name="pointerId">The id of the pointer message</param>
        /// <param name="action">The stylus action being done</param>
        /// <param name="timestamp">The time (in ticks) the message arrived</param>
        /// <returns>True if successfully processed (handled), false otherwise</returns>
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {
            ... // 忽略其他代码
        }
    }

    ... // 忽略其他代码
}
```

在 ProcessMessage 里面将创建 PointerData 对象，这个 PointerData 类型是一个辅助类，在构造函数里面将调用 [GetPointerInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfo) 方法获取指针点信息

```csharp
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {
            bool handled = false;

            // Acquire all pointer data needed
            PointerData data = new PointerData(pointerId);

            ... // 忽略其他代码
        }
```

以下是 PointerData 构造函数的简单定义的有删减的代码

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// Provides a wrapping class that aggregates Pointer data from a pointer event/message
    /// </summary>
    internal class PointerData
    {
        /// <summary>
        /// Queries all needed data from a particular pointer message and stores
        /// it locally.
        /// </summary>
        /// <param name="pointerId">The id of the pointer message</param>
        internal PointerData(uint pointerId)
        {
            if (IsValid = GetPointerInfo(pointerId, ref _info))
            {
                _history = new POINTER_INFO[_info.historyCount];

                // Fill the pointer history
                // If we fail just return a blank history
                if (!GetPointerInfoHistory(pointerId, ref _info.historyCount, _history))
                {
                    _history = Array.Empty<POINTER_INFO>();
                }

                ... // 忽略其他代码
            }
        }

        /// <summary>
        /// Standard pointer information
        /// </summary>
        private POINTER_INFO _info;

        /// <summary>
        /// The full history available for the current pointer (used for coalesced input)
        /// </summary>
        private POINTER_INFO[] _history;

        /// <summary>
        /// If true, we have correctly queried pointer data, false otherwise.
        /// </summary>
        internal bool IsValid { get; private set; } = false;
    }
```

通过上述代码可以看到，开始是调用 [GetPointerInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfo) 方法获取指针点信息。在 WPF 的基础事件里面也是支持历史点的，意图和 Pointer 的设计意图差不多，都是为了解决业务端的消费数据速度问题。于是在 WPF 底层也就立刻调用 [GetPointerInfoHistory](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerinfohistory) 获取历史点信息

对于 Pointer 消息来说，对触摸和触笔有着不同的数据提供分支，分别是 [GetPointerTouchInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointertouchinfo) 方法和 [GetPointerPenInfo](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerpeninfo) 方法

在 PointerData 构造函数里面，也通过判断 `POINTER_INFO` 的 `pointerType` 字段决定调用不同的方法，代码如下

```csharp
            if (IsValid = GetPointerInfo(pointerId, ref _info))
            {
                switch (_info.pointerType)
                {
                    case POINTER_INPUT_TYPE.PT_TOUCH:
                        {
                            // If we have a touch device, pull the touch specific information down
                            IsValid &= GetPointerTouchInfo(pointerId, ref _touchInfo);
                        }
                        break;
                    case POINTER_INPUT_TYPE.PT_PEN:
                        {
                            // Otherwise we have a pen device, so pull down pen specific information
                            IsValid &= GetPointerPenInfo(pointerId, ref _penInfo);
                        }
                        break;
                    default:
                        {
                            // Only process touch or pen messages, do not process mouse or touchpad
                            IsValid = false;
                        }
                        break;
                }
            }
```

对于 WPF 的 HwndPointerInputProvider 模块来说，只处理 PT_TOUCH 和 PT_PEN 消息，即触摸和触笔消息。对于 Mouse 鼠标和 Touchpad 触摸板来说都不走 Pointer 处理，依然是走原来的 Win32 消息。为什么这么设计呢？因为 WPF 里面没有 Pointer 路由事件，在 WPF 里面分开了 Touch 和 Stylus 和 Mouse 事件。就不需要全部都在 Pointer 模块处理了，依然在原来的消息循环里面处理，既减少 Pointer 模块的工作量，也能减少后续从 Pointer 分发到 Touch 和 Stylus 和 Mouse 事件的工作量。原先的模块看起来也跑得很稳，那就一起都不改了

完成 PointerData 的构造函数之后，继续到 HwndPointerInputProvider 的 ProcessMessage 函数里面，在此函数里面判断是 PT_TOUCH 和 PT_PEN 消息，则进行处理

```csharp
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {
            bool handled = false;

            // Acquire all pointer data needed
            PointerData data = new PointerData(pointerId);

            // Only process touch or pen messages, do not process mouse or touchpad
            if (data.IsValid
                && (data.Info.pointerType == UnsafeNativeMethods.POINTER_INPUT_TYPE.PT_TOUCH
                || data.Info.pointerType == UnsafeNativeMethods.POINTER_INPUT_TYPE.PT_PEN))
            {
                ... // 忽略其他代码
            }

            return handled;
        }
```

对于触摸和触笔的处理上，先是执行触摸设备关联。触摸设备关联一个在上层业务的表现就是让当前的指针消息关联上 TouchDevice 的 Id 或 StylusDevice 的 Id 值

关联的方法是通过 [GetPointerCursorId](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointercursorid) 方法先获取 CursorId 的值，再配合对应的输入的 Pointer 的输入设备 `POINTER_INFO` 的 `sourceDevice` 字段，即可与初始化过程中创建的设备相关联，实现代码如下

```csharp
            if (data.IsValid
                && (data.Info.pointerType == UnsafeNativeMethods.POINTER_INPUT_TYPE.PT_TOUCH
                || data.Info.pointerType == UnsafeNativeMethods.POINTER_INPUT_TYPE.PT_PEN))
            {
                uint cursorId = 0;

                if (UnsafeNativeMethods.GetPointerCursorId(pointerId, ref cursorId))
                {
                    IntPtr deviceId = data.Info.sourceDevice;

                    // If we cannot acquire the latest tablet and stylus then wait for the
                    // next message.
                    if (!UpdateCurrentTabletAndStylus(deviceId, cursorId))
                    {
                        return false;
                    }

                     ... // 忽略其他代码
                }

                ... // 忽略其他代码
            }
```

在 WPF 初始化工作里面将输入的 Pointer 的输入设备 `POINTER_INFO` 的 `sourceDevice` 当成 `deviceId` 的概念，即 TabletDevice 的 Id 值。而 `cursorId` 则是对应 StylusDevice 的 Id 值，其更新代码的核心非常简单，如下面代码

```csharp
        /// <summary>
        /// Attempts to update the current stylus and tablet devices for the latest WM_POINTER message.
        /// Will attempt retries if the tablet collection is invalid or does not contain the proper ids.
        /// </summary>
        /// <param name="deviceId">The id of the TabletDevice</param>
        /// <param name="cursorId">The id of the StylusDevice</param>
        /// <returns>True if successfully updated, false otherwise.</returns>
        private bool UpdateCurrentTabletAndStylus(IntPtr deviceId, uint cursorId)
        {
            _currentTabletDevice = tablets?.GetByDeviceId(deviceId);

            _currentStylusDevice = _currentTabletDevice?.GetStylusByCursorId(cursorId);
            
            ... // 忽略其他代码

                if (_currentTabletDevice == null || _currentStylusDevice == null)
                {
                    return false;
                }
            

            return true;
        }
```

对应的 GetByDeviceId 方法的代码如下

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// Maintains a collection of pointer device information for currently installed pointer devices
    /// </summary>
    internal class PointerTabletDeviceCollection : TabletDeviceCollection
    {
        /// <summary>
        /// Holds a mapping of TabletDevices from their WM_POINTER device id
        /// </summary>
        private Dictionary<IntPtr, PointerTabletDevice> _tabletDeviceMap = new Dictionary<IntPtr, PointerTabletDevice>();

         ... // 忽略其他代码

        /// <summary>
        /// Retrieve the TabletDevice associated with the device id
        /// </summary>
        /// <param name="deviceId">The device id</param>
        /// <returns>The TabletDevice associated with the device id</returns>
        internal PointerTabletDevice GetByDeviceId(IntPtr deviceId)
        {
            PointerTabletDevice tablet = null;

            _tabletDeviceMap.TryGetValue(deviceId, out tablet);

            return tablet;
        }
    }
}
```

对应的 GetStylusByCursorId 的代码如下

```csharp
namespace System.Windows.Input.StylusPointer
{  
    /// <summary>
    /// A WM_POINTER based implementation of the TabletDeviceBase class.
    /// </summary>
    internal class PointerTabletDevice : TabletDeviceBase
    {
        /// <summary>
        /// A mapping from StylusDevice id to the actual StylusDevice for quick lookup.
        /// </summary>
        private Dictionary<uint, PointerStylusDevice> _stylusDeviceMap = new Dictionary<uint, PointerStylusDevice>();

        /// <summary>
        /// Retrieves the StylusDevice associated with the cursor id.
        /// </summary>
        /// <param name="cursorId">The id of the StylusDevice to retrieve</param>
        /// <returns>The StylusDevice associated with the id</returns>
        internal PointerStylusDevice GetStylusByCursorId(uint cursorId)
        {
            PointerStylusDevice stylus = null;
            _stylusDeviceMap.TryGetValue(cursorId, out stylus);
            return stylus;
        }
    }
}
```

通过以上方式即可通过 PointerId 获取的 `cursorId` 进而获取到对应 WPF 里面的设备对象，进而拿到 WPF 里面的设备 Id 号。通过上文的描述也可以看到 PointerId 和 TouchDevice 等的 Id 是不一样的，但是之间有关联关系

调用了 UpdateCurrentTabletAndStylus 的一个副作用就是同步更新了 `_currentTabletDevice` 和 `_currentStylusDevice` 字段的值，后续逻辑即可直接使用这两个字段而不是传参数

完成关联逻辑之后，即进入 GenerateRawStylusData 方法，这个方法是 WPF 获取 Pointer 具体的消息的核心方法，方法签名如下

```csharp
namespace System.Windows.Interop
{
    /// <summary>
    /// Implements an input provider per hwnd for WM_POINTER messages
    /// </summary>
    internal sealed class HwndPointerInputProvider : DispatcherObject, IStylusInputProvider
    {
        /// <summary>
        /// Creates raw stylus data from the raw WM_POINTER properties
        /// </summary>
        /// <param name="pointerData">The current pointer info</param>
        /// <param name="tabletDevice">The current TabletDevice</param>
        /// <returns>An array of raw pointer data</returns>
        private int[] GenerateRawStylusData(PointerData pointerData, PointerTabletDevice tabletDevice)
        {
            ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }
}
```

此 GenerateRawStylusData 被调用是这么写的

```csharp
namespace System.Windows.Interop
{
    /// <summary>
    /// Implements an input provider per hwnd for WM_POINTER messages
    /// </summary>
    internal sealed class HwndPointerInputProvider : DispatcherObject, IStylusInputProvider
    {
        /// <summary>
        /// Processes the latest WM_POINTER message and forwards it to the WPF input stack.
        /// </summary>
        /// <param name="pointerId">The id of the pointer message</param>
        /// <param name="action">The stylus action being done</param>
        /// <param name="timestamp">The time (in ticks) the message arrived</param>
        /// <returns>True if successfully processed (handled), false otherwise</returns>
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {
            PointerData data = new PointerData(pointerId);

            ... // 忽略其他代码
                uint cursorId = 0;
                if (UnsafeNativeMethods.GetPointerCursorId(pointerId, ref cursorId))
                {
                    ... // 忽略其他代码
                    GenerateRawStylusData(data, _currentTabletDevice);
                    ... // 忽略其他代码
                }

        }
        ... // 忽略其他代码
    }
}
```

在 GenerateRawStylusData 方法里面，先通过 PointerTabletDevice 取出支持的 Pointer 的设备属性列表的长度，用于和输入点的信息进行匹配。回忆一下，这部分获取逻辑是在上文介绍到对 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 函数的调用提到的，且也说明了此函数拿到的设备属性列表的顺序是非常关键的，设备属性列表的顺序和在后续 WM_POINTER 消息拿到的裸数据的顺序是直接对应的

```csharp
    /// <summary>
    /// Implements an input provider per hwnd for WM_POINTER messages
    /// </summary>
    internal sealed class HwndPointerInputProvider : DispatcherObject, IStylusInputProvider
    {
        /// <summary>
        /// Creates raw stylus data from the raw WM_POINTER properties
        /// </summary>
        /// <param name="pointerData">The current pointer info</param>
        /// <param name="tabletDevice">The current TabletDevice</param>
        /// <returns>An array of raw pointer data</returns>
        private int[] GenerateRawStylusData(PointerData pointerData, PointerTabletDevice tabletDevice)
        {
            // Since we are copying raw pointer data, we want to use every property supported by this pointer.
            // We may never access some of the unknown (unsupported by WPF) properties, but they should be there
            // for consumption by the developer.
            int pointerPropertyCount = tabletDevice.DeviceInfo.SupportedPointerProperties.Length;

            // The data is as wide as the pointer properties and is per history point
            int[] rawPointerData = new int[pointerPropertyCount * pointerData.Info.historyCount];

            ... // 忽略其他代码
        }

        ... // 忽略其他代码
    }
```

由每个 Pointer 的属性长度配合总共的历史点数量，即可获取到这里面使用到的 `rawPointerData` 数组的长度。这部分代码相信大家很好就理解了

接着就是核心部分，调用 [GetRawPointerDeviceData](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getrawpointerdevicedata) 获取最原始的触摸信息，再对原始触摸信息进行解析处理

```csharp
            int pointerPropertyCount = tabletDevice.DeviceInfo.SupportedPointerProperties.Length;

            // The data is as wide as the pointer properties and is per history point
            int[] rawPointerData = new int[pointerPropertyCount * pointerData.Info.historyCount];

            // Get the raw data formatted to our supported properties
            if (UnsafeNativeMethods.GetRawPointerDeviceData(
                pointerData.Info.pointerId,
                pointerData.Info.historyCount,
                (uint)pointerPropertyCount,
                tabletDevice.DeviceInfo.SupportedPointerProperties,
                rawPointerData))
            {
                ... // 忽略其他代码
            }
```

在 Pointer 的设计里面，历史点 `historyCount` 是包含当前点的，且当前点就是最后一个点。这就是为什么这里只需要传入历史点数量即可，换句话说就是历史点最少包含一个点，那就是当前点

由于 Pointer 获取到的点都是相对于屏幕坐标的，这里需要先偏移一下修改为窗口坐标系，代码如下

```csharp
                // Get the X and Y offsets to translate device coords to the origin of the hwnd
                int originOffsetX, originOffsetY;
                GetOriginOffsetsLogical(out originOffsetX, out originOffsetY);

        private void GetOriginOffsetsLogical(out int originOffsetX, out int originOffsetY)
        {
            Point originScreenCoord = _source.Value.RootVisual.PointToScreen(new Point(0, 0));

            // Use the inverse of our logical tablet to screen matrix to generate tablet coords
            MatrixTransform screenToTablet = new MatrixTransform(_currentTabletDevice.TabletToScreen);
            screenToTablet = (MatrixTransform)screenToTablet.Inverse;

            Point originTabletCoord = originScreenCoord * screenToTablet.Matrix;

            originOffsetX = (int)Math.Round(originTabletCoord.X);
            originOffsetY = (int)Math.Round(originTabletCoord.Y);
        }

        /// <summary>
        /// The HwndSource for WM_POINTER messages
        /// </summary>
        private SecurityCriticalDataClass<HwndSource> _source;
```

这里的 GetOriginOffsetsLogical 的实现逻辑就是去窗口的 0,0 点，看这个点会在屏幕的哪里，从而知道其偏移量。至于添加的 MatrixTransform 矩阵的 TabletToScreen 则在后文的具体转换逻辑会讲到，这里先跳过

获取到相对于窗口的坐标偏移量之后，即可将其叠加给到每个点上，用于将这些点转换为窗口坐标系。但是在此之前还需要将获取到的 `rawPointerData` 进行加工。这一个步骤仅仅只是在 WPF 有需求，仅仅只是为了兼容 WISP 获取到的裸数据的方式。其相差点在于通过 Pointer 获取到的 `rawPointerData` 的二进制数据格式里面，没有带上按钮的支持情况的信息，在 WPF 这边需要重新创建一个数组对 `rawPointerData` 重新排列，确保每个点的数据都加上按钮的信息数据

这部分处理仅只是为了兼容考虑，让后续的 StylusPointCollection 开森而已，咱就跳着看就好了

```csharp
                int numButtons = tabletDevice.DeviceInfo.SupportedPointerProperties.Length - tabletDevice.DeviceInfo.SupportedButtonPropertyIndex;

                int rawDataPointSize = (numButtons > 0) ? pointerPropertyCount - numButtons + 1 : pointerPropertyCount;

                // Instead of a single entry for each button we use one entry for all buttons so reflect that in the raw data size
                data = new int[rawDataPointSize * pointerData.Info.historyCount];

                for (int i = 0, j = rawPointerData.Length - pointerPropertyCount; i < data.Length; i += rawDataPointSize, j -= pointerPropertyCount)
                {
                    Array.Copy(rawPointerData, j, data, i, rawDataPointSize);

                    // Apply offsets from the origin to raw pointer data here
                    data[i + StylusPointDescription.RequiredXIndex] -= originOffsetX;
                    data[i + StylusPointDescription.RequiredYIndex] -= originOffsetY;

                    ... // 忽略其他代码
                }

             ... // 忽略其他代码
            return data;
```

重新拷贝的过程，还将点的坐标更换成窗口坐标系，即以上的 `data[i + StylusPointDescription.RequiredXIndex] -= originOffsetX;` 和 `data[i + StylusPointDescription.RequiredYIndex] -= originOffsetY;` 两个代码

完成获取之后，就将获取到的裸数据给返回了，这就是 GenerateRawStylusData 的内容

在 ProcessMessage 方法里面获取到 GenerateRawStylusData 返回的原始指针信息，即可将其给到 RawStylusInputReport 作为参数，代码如下

```csharp
                    // Generate a raw input to send to the input manager to start the event chain in PointerLogic
                    Int32[] rawData = GenerateRawStylusData(data, _currentTabletDevice);
                    RawStylusInputReport rsir =
                        new RawStylusInputReport(
                            InputMode.Foreground,
                            timestamp,
                            _source.Value,
                            action,
                            () => { return _currentTabletDevice.StylusPointDescription; },
                            _currentTabletDevice.Id,
                            _currentStylusDevice.Id,
                            rawData)
                        {
                            StylusDevice = _currentStylusDevice.StylusDevice,
                        };
```

将创建的 RawStylusInputReport 更新到当前的设备，作为设备的最新的指针信息

```csharp
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {

            PointerData data = new PointerData(pointerId);

             ... // 忽略其他代码

                    _currentStylusDevice.Update(this, _source.Value, data, rsir);
             ... // 忽略其他代码
        }

        private SecurityCriticalDataClass<HwndSource> _source;
```

且还加入到 InputManager 的 ProcessInput 里面，进入 WPF 的框架内的消息调度

```csharp
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {

            PointerData data = new PointerData(pointerId);

             ... // 忽略其他代码

                    _currentStylusDevice.Update(this, _source.Value, data, rsir);
                    // Now send the input report
                    InputManager.UnsecureCurrent.ProcessInput(irea);
             ... // 忽略其他代码
        }
```

在进入 InputManager 的 ProcessInput 调度消息之前，先看看 `_currentStylusDevice.Update` 里面的对原始指针信息的解析实现逻辑

在 `_currentStylusDevice.Update` 里面的对原始指针信息的解析实现完全是靠 StylusPointCollection 和 StylusPoint 的构造函数实现的

```csharp
namespace System.Windows.Input.StylusPointer
{
    /// <summary>
    /// A WM_POINTER specific implementation of the StylusDeviceBase.
    /// 
    /// Supports direct access to WM_POINTER structures and basing behavior off of the WM_POINTER data.
    /// </summary>
    internal class PointerStylusDevice : StylusDeviceBase
    {
        /// <summary>
        /// Updates the internal StylusDevice state based on the WM_POINTER input and the formed raw data.
        /// </summary>
        /// <param name="provider">The hwnd associated WM_POINTER provider</param>
        /// <param name="inputSource">The PresentationSource where this message originated</param>
        /// <param name="pointerData">The aggregated pointer data retrieved from the WM_POINTER stack</param>
        /// <param name="rsir">The raw stylus input generated from the pointer data</param>
        internal void Update(HwndPointerInputProvider provider, PresentationSource inputSource,
            PointerData pointerData, RawStylusInputReport rsir)
        {
             ... // 忽略其他代码

            // First get the initial stylus points.  Raw data from pointer input comes in screen coordinates, keep that here since that is what we expect.
            _currentStylusPoints = new StylusPointCollection(rsir.StylusPointDescription, rsir.GetRawPacketData(), GetTabletToElementTransform(null), Matrix.Identity);

             ... // 忽略其他代码
        }
    }
}
```

这里的 `rsir.GetRawPacketData()` 是返回上文提到的 `GenerateRawStylusData` 方法给出的裸数据的拷贝，代码如下

```csharp
    internal class RawStylusInputReport : InputReport
    {
        /// <summary>
        ///     Read-only access to the raw data that was reported.
        /// </summary>
        internal int[] GetRawPacketData()
        {
            if (_data == null)
                return null;
            return (int[])_data.Clone();
        }

        /// <summary>
        /// The raw data for this input report
        /// </summary>
        int[] _data;

        ... // 忽略其他代码
    }
```

这里的 GetTabletToElementTransform 包含了一个核心转换，方法代码如下

```csharp
    internal class PointerStylusDevice : StylusDeviceBase
    {
        /// <summary>
        ///     Returns the transform for converting from tablet to element
        ///     relative coordinates.
        /// </summary>
        internal GeneralTransform GetTabletToElementTransform(IInputElement relativeTo)
        {
            GeneralTransformGroup group = new GeneralTransformGroup();
            Matrix toDevice = _inputSource.Value.CompositionTarget.TransformToDevice;
            toDevice.Invert();
            group.Children.Add(new MatrixTransform(PointerTabletDevice.TabletToScreen * toDevice));
            group.Children.Add(StylusDevice.GetElementTransform(relativeTo));
            return group;
        }

        ... // 忽略其他代码
    }
```

这里面方法存在重点内容，那就是 PointerTabletDevice 的 TabletToScreen 属性的计算方法。这个矩阵的计算需要用到开始初始化过程的 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 函数获取 的 `displayRect` 尺寸，以及 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 获取的 X 和 Y 属性描述信息，属性的定义代码如下

```csharp
        internal Matrix TabletToScreen
        {
            get
            {
                return new Matrix(_tabletInfo.SizeInfo.ScreenSize.Width / _tabletInfo.SizeInfo.TabletSize.Width, 0,
                                   0, _tabletInfo.SizeInfo.ScreenSize.Height / _tabletInfo.SizeInfo.TabletSize.Height,
                                   0, 0);
            }
        }
```

可以看到这是一个用于缩放的 Matrix 对象，正是 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 获取的屏幕尺寸以及 [GetPointerDeviceProperties](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdeviceproperties ) 获取的 X 和 Y 属性描述信息构成的 TabletSize 的比值

回顾一下 `_tabletInfo` 的 SizeInfo 的创建代码，可以看到 TabletSize 完全是由描述符的尺寸决定，代码如下

```csharp
            // 以下代码在 PointerTabletDeviceInfo.cs 文件中
            // private bool TryInitializeSupportedStylusPointProperties()
            SupportedPointerProperties = new UnsafeNativeMethods.POINTER_DEVICE_PROPERTY[propCount];

            success = UnsafeNativeMethods.GetPointerDeviceProperties(Device, ref propCount, SupportedPointerProperties);

            ... // 忽略其他代码

            // private bool TryInitializeDeviceRects()
            var deviceRect = new UnsafeNativeMethods.RECT();
            var displayRect = new UnsafeNativeMethods.RECT();

            success = UnsafeNativeMethods.GetPointerDeviceRects(_deviceInfo.device, ref deviceRect, ref displayRect);

            if (success)
            {
                // We use the max X and Y properties here as this is more readily useful for raw data
                // which is where all conversions come from.
                SizeInfo = new TabletDeviceSizeInfo
                (
                    new Size(SupportedPointerProperties[StylusPointDescription.RequiredXIndex].logicalMax,
                    SupportedPointerProperties[StylusPointDescription.RequiredYIndex].logicalMax),

                    new Size(displayRect.right - displayRect.left, displayRect.bottom - displayRect.top)
                );
            }

    internal struct TabletDeviceSizeInfo
    {
        public Size TabletSize;
        public Size ScreenSize;

        internal TabletDeviceSizeInfo(Size tabletSize, Size screenSize)
        {
            TabletSize = tabletSize;
            ScreenSize = screenSize;
        }
    }
```

如此即可使用 TabletToScreen 属性将收到的基于 Tablet 坐标系的裸指针消息的坐标转换为屏幕坐标，再配合 TransformToDevice 取反即可转换到 WPF 坐标系

在以上代码里面，由于传入 GetTabletToElementTransform 的 `relativeTo` 参数是 null 的值，将导致 `StylusDevice.GetElementTransform(relativeTo)` 返回一个单位矩阵，这就意味着在 GetTabletToElementTransform 方法里面的 `group.Children.Add(StylusDevice.GetElementTransform(relativeTo));` 是多余的，也许后续 WPF 版本这里会被我优化掉

回顾一下 StylusPointCollection 的构造函数参数，有用的参数只有前三个，分别是 `rsir.StylusPointDescription` 传入描述符信息，以及 `rsir.GetRawPacketData()` 返回裸指针数据，以及 `GetTabletToElementTransform(null)` 方法返回转换为 WPF 坐标系的矩阵

```csharp
_currentStylusPoints = new StylusPointCollection(rsir.StylusPointDescription, rsir.GetRawPacketData(), GetTabletToElementTransform(null), Matrix.Identity);
```

那 StylusPointCollection 的最后一个参数，即上述代码传入的 `Matrix.Identity` 有什么用途？其实在 StylusPointCollection 的设计里面，第三个参数和第四个参数是二选一的，且第三个参数的优先级大于第四个参数。即在 StylusPointCollection 底层会判断第三个参数是否有值，如果没有值才会使用第四个参数

在 StylusPointCollection 构造函数里面将会对裸 Pointer 数据进行处理，现在 GetRawPacketData 拿到的裸 Pointer 数据的 int 数组里面的数据排列内容大概如下

```
| X 坐标 | Y 坐标 | 压感（可选）| StylusPointDescription 里面的属性列表一一对应 |
| X 坐标 | Y 坐标 | 压感（可选）| StylusPointDescription 里面的属性列表一一对应 |
| X 坐标 | Y 坐标 | 压感（可选）| StylusPointDescription 里面的属性列表一一对应 |
```

存放的是一个或多个点信息，每个点的信息都是相同的二进制长度，分包非常简单

进入到 StylusPointCollection 的构造函数，看看其代码签名定义

```csharp
namespace System.Windows.Input
{
    public class StylusPointCollection : Collection<StylusPoint>
    {
        internal StylusPointCollection(StylusPointDescription stylusPointDescription, int[] rawPacketData, GeneralTransform tabletToView, Matrix tabletToViewMatrix)
        {
            ... // 忽略其他代码
        }
    }
}
```

在构造函数里面，先调用 StylusPointDescription 的 GetInputArrayLengthPerPoint 方法，获取每个点的二进制长度，代码如下

```csharp
    public class StylusPointCollection : Collection<StylusPoint>
    {
        internal StylusPointCollection(StylusPointDescription stylusPointDescription, int[] rawPacketData, GeneralTransform tabletToView, Matrix tabletToViewMatrix)
        {
            ... // 忽略其他代码
            int lengthPerPoint = stylusPointDescription.GetInputArrayLengthPerPoint();

            ... // 忽略其他代码
        }
    }
```

获取到了一个点的二进制长度，自然就能算出传入的 `rawPacketData` 参数包含多少个点的信息

```csharp
        internal StylusPointCollection(StylusPointDescription stylusPointDescription, int[] rawPacketData, GeneralTransform tabletToView, Matrix tabletToViewMatrix)
        {
            ... // 忽略其他代码
            int lengthPerPoint = stylusPointDescription.GetInputArrayLengthPerPoint();
            int logicalPointCount = rawPacketData.Length / lengthPerPoint;
            Debug.Assert(0 == rawPacketData.Length % lengthPerPoint, "Invalid assumption about packet length, there shouldn't be any remainder");
            ... // 忽略其他代码
        }
```

以上代码的 `Debug.Assert` 就是要确保传入的 `rawPacketData` 是可以被 `lengthPerPoint` 即每个点的二进制长度所整除

完成准备工作之后，接下来就可以将 `rawPacketData` 解出点了，如下面代码所示

```csharp
            int lengthPerPoint = stylusPointDescription.GetInputArrayLengthPerPoint();
            int logicalPointCount = rawPacketData.Length / lengthPerPoint;

            for (int count = 0, i = 0; count < logicalPointCount; count++, i += lengthPerPoint)
            {
                //first, determine the x, y values by xf-ing them
                Point p = new Point(rawPacketData[i], rawPacketData[i + 1]);

                ... // 忽略其他代码

                int startIndex = 2;

                ... // 忽略其他代码

                int[] data = null;
                int dataLength = lengthPerPoint - startIndex;
                if (dataLength > 0)
                {
                    //copy the rest of the data
                    var rawArrayStartIndex = i + startIndex;
                    data = rawPacketData.AsSpan(rawArrayStartIndex, dataLength).ToArray();
                }

                StylusPoint newPoint = new StylusPoint(p.X, p.Y, StylusPoint.DefaultPressure, _stylusPointDescription, data, false, false);

                ... // 忽略其他代码

                ((List<StylusPoint>)this.Items).Add(newPoint);
            }
```

以上代码忽略的部分包含了一些细节，如对 Point 的坐标转换，使用 `Point p = new Point(rawPacketData[i], rawPacketData[i + 1]);` 拿到的点的坐标是属于 Tablet 坐标，需要使用传入的参数转换为 WPF 坐标，如下面代码所示

```csharp
        internal StylusPointCollection(StylusPointDescription stylusPointDescription, int[] rawPacketData, GeneralTransform tabletToView, Matrix tabletToViewMatrix)
        {
                ... // 忽略其他代码

                Point p = new Point(rawPacketData[i], rawPacketData[i + 1]);
                if (tabletToView != null)
                {
                    tabletToView.TryTransform(p, out p);
                }
                else
                {
                    p = tabletToViewMatrix.Transform(p);
                }

                ... // 忽略其他代码
        }
```

通过以上的代码就可以看到 StylusPointCollection 构造函数使用了第三个或第四个参数作为变换，如果第三个参数存在则优先使用第三个参数

其他处理的逻辑就是对压感的额外处理，压感作为 StylusPoint 的一个明确参数，需要额外判断处理

```csharp
                int startIndex = 2; // X 和 Y 占用了两个元素
                bool containsTruePressure = stylusPointDescription.ContainsTruePressure;
                if (containsTruePressure)
                {
                    // 如果有压感的话，压感也需要多占一个元素
                    //don't copy pressure in the int[] for extra data
                    startIndex++;
                }

                StylusPoint newPoint = new StylusPoint(p.X, p.Y, StylusPoint.DefaultPressure, _stylusPointDescription, data, false, false);
                if (containsTruePressure)
                {
                    // 压感必定是第三个元素，有压感则更新压感
                    //use the algorithm to set pressure in StylusPoint
                    int pressure = rawPacketData[i + 2];
                    newPoint.SetPropertyValue(StylusPointProperties.NormalPressure, pressure);
                }
```

如此即可解包 `| X 坐标 | Y 坐标 | 压感（可选）| StylusPointDescription 里面的属性列表一一对应 |` 里面前三个元素，其中压感是可选的。后续的 `StylusPointDescription 里面的属性列表一一对应` 部分需要重新创建 data 数组传入到各个 StylusPoint 里面，代码如下

```csharp
                int[] data = null;
                int dataLength = lengthPerPoint - startIndex;
                if (dataLength > 0)
                {
                    //copy the rest of the data
                    var rawArrayStartIndex = i + startIndex;
                    data = rawPacketData.AsSpan(rawArrayStartIndex, dataLength).ToArray();
                }
```

后续对 StylusPoint 获取属性时，即可通过描述信息获取，描述信息获取到值的方式就是取以上代码传入的 `data` 二进制数组的对应下标的元素，比如触摸点的宽度或高度信息

完成转换为 StylusPointCollection 之后，即可使用 `InputManager.UnsecureCurrent.ProcessInput` 方法将裸输入信息调度到 WPF 输入管理器

```csharp
        private bool ProcessMessage(uint pointerId, RawStylusActions action, int timestamp)
        {
             ... // 忽略其他代码
                    InputReportEventArgs irea = new InputReportEventArgs(_currentStylusDevice.StylusDevice, rsir)
                    {
                        RoutedEvent = InputManager.PreviewInputReportEvent,
                    };

                    // Now send the input report
                    InputManager.UnsecureCurrent.ProcessInput(irea);
             ... // 忽略其他代码
        }
```

进入到 ProcessInput 里面将会走标准的路由事件机制，通过路由机制触发 Touch 或 Stylus 事件，接下来的逻辑看一下调用堆栈即可，和其他的输入事件逻辑差不多

```
>   Lindexi.dll!Lindexi.MainWindow.MainWindow_TouchDown(object sender, System.Windows.Input.TouchEventArgs e)
    PresentationCore.dll!System.Windows.RoutedEventArgs.InvokeHandler(System.Delegate handler, object target)
    PresentationCore.dll!System.Windows.EventRoute.InvokeHandlersImpl(object source, System.Windows.RoutedEventArgs args, bool reRaised) 
    PresentationCore.dll!System.Windows.UIElement.RaiseEventImpl(System.Windows.DependencyObject sender, System.Windows.RoutedEventArgs args)
    PresentationCore.dll!System.Windows.UIElement.RaiseTrustedEvent(System.Windows.RoutedEventArgs args) 
    PresentationCore.dll!System.Windows.Input.InputManager.ProcessStagingArea()
    PresentationCore.dll!System.Windows.Input.TouchDevice.RaiseTouchDown() 
    PresentationCore.dll!System.Windows.Input.TouchDevice.ReportDown() 
    PresentationCore.dll!System.Windows.Input.StylusTouchDeviceBase.OnDown() 
    PresentationCore.dll!System.Windows.Input.StylusPointer.PointerLogic.PromoteMainDownToTouch(System.Windows.Input.StylusPointer.PointerStylusDevice stylusDevice, System.Windows.Input.StagingAreaInputItem stagingItem)
    PresentationCore.dll!System.Windows.Input.InputManager.RaiseProcessInputEventHandlers(System.Tuple<System.Windows.Input.ProcessInputEventHandler, System.Delegate[]> postProcessInput, System.Windows.Input.ProcessInputEventArgs processInputEventArgs) 
    PresentationCore.dll!System.Windows.Input.InputManager.ProcessStagingArea()
    PresentationCore.dll!System.Windows.Interop.HwndPointerInputProvider.ProcessMessage(uint pointerId, System.Windows.Input.RawStylusActions action, int timestamp) 
    PresentationCore.dll!System.Windows.Interop.HwndPointerInputProvider.System.Windows.Interop.IStylusInputProvider.FilterMessage(nint hwnd, MS.Internal.Interop.WindowMessage msg, nint wParam, nint lParam, ref bool handled) 
    PresentationCore.dll!System.Windows.Interop.HwndSource.InputFilterMessage(nint hwnd, int msg, nint wParam, nint lParam, ref bool handled)
```

由于我跑的是 Release 版本的 WPF 导致了有一些函数被内联，如从 `HwndPointerInputProvider.ProcessMessage` 到 `InputManager.ProcessStagingArea` 中间就少了 `InputManager.ProcessInput` 函数，完全的无函数内联的堆栈应该如下

```csharp
    PresentationCore.dll!System.Windows.Input.InputManager.ProcessStagingArea()
    PresentationCore.dll!System.Windows.Input.InputManager.ProcessInput()
    PresentationCore.dll!System.Windows.Interop.HwndPointerInputProvider.ProcessMessage(uint pointerId, System.Windows.Input.RawStylusActions action, int timestamp)
```

如下面代码是 ProcessInput 函数的代码

```csharp
    public sealed class InputManager : DispatcherObject
    {
        public bool ProcessInput(InputEventArgs input)
        {
            ... // 忽略其他代码
            PushMarker();
            PushInput(input, null);
            RequestContinueProcessingStagingArea();

            bool handled = ProcessStagingArea();
            return handled;
        }
    }
```

进入到 ProcessStagingArea 方法会执行具体的调度逻辑，用上述触摸按下的堆栈作为例子，将会进入到 PointerLogic 的 PostProcessInput 方法里面，由 PostProcessInput 方法调用到 PromoteMainToOther 再到 PromoteMainToTouch 最后到 PromoteMainDownToTouch 方法。只不过中间的几个方法被内联了，直接从堆栈上看就是从 RaiseProcessInputEventHandlers 到 PromoteMainDownToTouch 方法，堆栈如下

```
PresentationCore.dll!System.Windows.Input.StylusPointer.PointerLogic.PromoteMainDownToTouch(...)
PresentationCore.dll!System.Windows.Input.InputManager.RaiseProcessInputEventHandlers(...)
```

核心触发按下的代码就在 PromoteMainDownToTouch 里，其代码大概如下

```csharp
        private void PromoteMainDownToTouch(PointerStylusDevice stylusDevice, StagingAreaInputItem stagingItem)
        {
            PointerTouchDevice touchDevice = stylusDevice.TouchDevice;

            ... // 忽略其他代码

            touchDevice.OnActivate();
            touchDevice.OnDown();
        }
```

从上文可以知道，在 HwndPointerInputProvider 的 ProcessMessage 里面调用了 `_currentStylusDevice.Update` 方法时，就将输入的数据存放到 PointerStylusDevice 里面

后续的逻辑就和 [WPF 模拟触摸设备](https://blog.lindexi.com/post/WPF-%E6%A8%A1%E6%8B%9F%E8%A7%A6%E6%91%B8%E8%AE%BE%E5%A4%87.html ) 提到的使用方法差不多，只是数据提供源是从 PointerStylusDevice 提供。如果大家对进入到 InputManager 的后续逻辑感兴趣，可参考 [WPF 通过 InputManager 模拟调度触摸事件](https://blog.lindexi.com/post/WPF-%E9%80%9A%E8%BF%87-InputManager-%E6%A8%A1%E6%8B%9F%E8%B0%83%E5%BA%A6%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6.html ) 提供的方法自己跑一下

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )