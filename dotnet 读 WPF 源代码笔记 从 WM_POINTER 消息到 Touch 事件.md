# dotnet 读 WPF 源代码笔记 从 WM_POINTER 消息到 Touch 事件

本文记录我读 WPF 源代码的笔记，在 WPF 底层是如何从 Win32 的消息循环获取到的 WM_POINTER 消息处理转换作为 Touch 事件的参数

<!--more-->
<!-- CreateTime:2024/09/01 07:15:29 -->

<!-- 草稿 -->

由于 WPF 触摸部分会兼顾开启 Pointer 消息和不开启 Pointer 消息，为了方便大家理解，本文分为两个部分。第一个部分是脱离 WPF 框架，聊聊一个 Win32 程序如何从 Win32 的消息循环获取到的 WM_POINTER 消息处理转换为输入坐标点，以及在触摸下获取触摸信息。第二部分是 WPF 框架是如何安排上这些处理逻辑，如何和 WPF 框架的进行对接

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

使用 `ptHimetricLocationRaw` 字段的优势在于可以获取不丢失精度的信息，但需要额外调用 [GetPointerDeviceRects](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-getpointerdevicerects) 函数获取 `displayRect` 和 `pointerDeviceRect` 信息用于转换坐标点

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

