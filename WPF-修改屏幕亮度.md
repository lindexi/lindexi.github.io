
# WPF 修改屏幕亮度

在 WPF 中可以使用 Dxva2 或 GDI 的方法调整屏幕亮度或获取屏幕亮度

<!--more-->


<!-- CreateTime:7/3/2020 2:39:50 PM -->



比较推荐使用 Dxva2 的方法修改亮度，但不是所有的屏幕都支持的。假定某个设备有多个屏幕，此时可以使用 User32 的 MonitorFromWindow 方法获取某个窗口所在的屏幕，此时根据这个屏幕的返回的 GetMonitorBrightness 判断是否支持 Dxva2 的方法

如果 Dxva2 的方法不支持，那么尝试使用 GDI 的方式，下面请让我告诉大家两个方法如何使用

先定义 AdjustScreenByDxva2 类，这个类通过 `dxva2.dll` 的几个方法进行获取或修改屏幕亮度

```csharp
        [DllImport("dxva2.dll")]
        public static extern bool SetMonitorBrightness(IntPtr hMonitor, short brightness);

        [DllImport("dxva2.dll")]
        public static extern bool GetMonitorBrightness(IntPtr hMonitor, ref short pdwMinimumBrightness,
            ref short pdwCurrentBrightness, ref short pdwMaximumBrightness);

        [DllImport("dxva2.dll")]
        public static extern bool GetNumberOfPhysicalMonitorsFromHMONITOR(IntPtr hMonitor,
            ref uint pdwNumberOfPhysicalMonitors);

        [DllImport("dxva2.dll")]
        public static extern bool GetPhysicalMonitorsFromHMONITOR(IntPtr hMonitor,
            uint dwPhysicalMonitorArraySize, [Out] PhysicalMonitor[] pPhysicalMonitorArray);

        [DllImport("user32.dll")]
        public static extern IntPtr MonitorFromWindow([In] IntPtr hwnd, uint dwFlags);
```

这里 PhysicalMonitor 的定义如下

```csharp
    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
    public struct PhysicalMonitor
    {
        public IntPtr hPhysicalMonitor;

        [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 128)]
        public string szPhysicalMonitorDescription;
    }
```


设置屏幕亮度和获取屏幕亮度的方法如下

```csharp
        /// <summary>
        /// 设置屏幕亮度
        /// </summary>
        /// <param name="handle">所在屏幕窗口的句柄</param>
        /// <param name="brightness">亮度</param>
        /// <returns></returns>
        public bool SetBrightness(IntPtr handle, short brightness)
        {
            if (handle == IntPtr.Zero) return false;

            uint pdwNumberOfPhysicalMonitors = uint.MinValue;
            //获取屏幕所在的屏幕设备
            var hMonitor = MonitorFromWindow(handle, (uint) NativeConstantsEnum.MonitorDefaultToPrimary);
            GetNumberOfPhysicalMonitorsFromHMONITOR(hMonitor, ref pdwNumberOfPhysicalMonitors);
            var screen = new PhysicalMonitor[pdwNumberOfPhysicalMonitors];
            GetPhysicalMonitorsFromHMONITOR(hMonitor, pdwNumberOfPhysicalMonitors, screen);
            if (screen.Length <= 0) return false;
            return SetMonitorBrightness(screen[0].hPhysicalMonitor, brightness);
        }

        /// <summary>
        /// 设置屏幕亮度
        /// </summary>
        /// <param name="handle">所在屏幕窗口的句柄</param>
        /// <param name="minBrightness"></param>
        /// <param name="currentBrightness"></param>
        /// <param name="maxBrightness"></param>
        /// <returns></returns>
        public bool GetBrightness(IntPtr handle, ref short minBrightness,
            ref short currentBrightness,
            ref short maxBrightness)
        {
            if (handle == IntPtr.Zero) return false;
            uint pdwNumberOfPhysicalMonitors = uint.MinValue;
            //获取屏幕所在的屏幕设备
            var hMonitor = MonitorFromWindow(handle, (uint) NativeConstantsEnum.MonitorDefaultToPrimary);
            GetNumberOfPhysicalMonitorsFromHMONITOR(hMonitor, ref pdwNumberOfPhysicalMonitors);
            var screen = new PhysicalMonitor[pdwNumberOfPhysicalMonitors];
            GetPhysicalMonitorsFromHMONITOR(hMonitor, pdwNumberOfPhysicalMonitors, screen);
            if (screen.Length <= 0) return false;

            return GetMonitorBrightness(screen[0].hPhysicalMonitor, ref minBrightness,
                ref currentBrightness,
                ref maxBrightness);
        }
```

如果获取 GetBrightness 返回 false 那么此屏幕不支持通过 dxva2 设置亮度

此时可以尝试使用 GDI 的方法，使用 GDI 的方法只支持设置主屏幕，不能根据传入的窗口获取窗口所在的屏幕的方式设置某个屏幕的亮度

```csharp
        [DllImport("gdi32.dll")]
        public static extern bool GetDeviceGammaRamp(IntPtr hDC, ref RAMP lpRamp);

        [DllImport("gdi32.dll")]
        public static extern bool SetDeviceGammaRamp(IntPtr hDC, ref RAMP lpRamp);
```

设置和获取屏幕亮度的方法如下

```csharp
        private double CalAllGammaVal(RAMP ramp)
        {
            return Math.Round(((CalColorGammaVal(ramp.Blue) + CalColorGammaVal(ramp.Red) +
                                CalColorGammaVal(ramp.Green)) / 3), 2);
        }

        private double CalColorGammaVal(ushort[] line)
        {
            var max = line.Max();
            var min = line[0];
            var index = Array.FindIndex(line, n => n == max);
            var gamma = Math.Round((((double) (max - min) / index) / 255), 2);
            return gamma;
        }

        /// <summary>
        /// 读取屏幕亮度
        /// </summary>
        /// <param name="minBrightness"></param>
        /// <param name="currentBrightness"></param>
        /// <param name="maxBrightness"></param>
        /// <returns></returns>
        public bool GetBrightness(ref short minBrightness, ref short currentBrightness,
            ref short maxBrightness)
        {
            var handle = Graphics.FromHwnd(IntPtr.Zero).GetHdc();
            //0-50 亮度变化太小，所以从50开始
            minBrightness = 50;
            maxBrightness = 100;
            var ramp = default(RAMP);
            var deviceGammaRamp = GetDeviceGammaRamp(handle, ref ramp);
            currentBrightness = (short) ((deviceGammaRamp ? CalAllGammaVal(ramp) : 0.5) * 100);
            return deviceGammaRamp;
        }

        /// <summary>
        /// 设置屏幕亮度
        /// </summary>
        /// <param name="brightness">0-100</param>
        /// <returns></returns>
        public bool SetBrightness(short brightness)
        {
            var handle = Graphics.FromHwnd(IntPtr.Zero).GetHdc();
            double value = (double) brightness / 100;
            RAMP ramp = default(RAMP);
            ramp.Red = new ushort[256];
            ramp.Green = new ushort[256];
            ramp.Blue = new ushort[256];

            for (int i = 1; i < 256; i++)
            {
                var tmp = (ushort) (i * 255 * value);
                ramp.Red[i] = ramp.Green[i] = ramp.Blue[i] = Math.Max(ushort.MinValue, Math.Min(ushort.MaxValue, tmp));
            }

            var deviceGammaRamp = SetDeviceGammaRamp(handle, ref ramp);
            return deviceGammaRamp;
        }
```

一个可用的封装如下，只需要使用 AdjustScreenBuilder.CreateAdjustScreen 获取修改屏幕亮度的对象，通过这个对象就可以设置或获取屏幕亮度

```csharp
using System;
using System.Windows;
using System.Windows.Interop;


    public class AdjustScreenBuilder
    {
        /// <summary>
        /// 创建一个调整屏幕亮度的对象
        /// </summary>
        /// <returns>如果返回空，则代表此屏幕不支持</returns>
        [CanBeNull]
        public static IAdjustScreen CreateAdjustScreen(Window window)
        {
            if (window == null) throw new ArgumentNullException(nameof(window));
            //第一次尝试使用Dxva2方式
            AdjustScreenByDxva2 adjustScreenByDxva2 = new AdjustScreenByDxva2();
            var handle = new WindowInteropHelper(window).Handle;
            short min = 0;
            short current = 0;
            short max = 0;
            var adjustScreenByDxva2Value = adjustScreenByDxva2.GetBrightness(handle, ref min, ref current, ref max);
            if (adjustScreenByDxva2Value)
                return adjustScreenByDxva2;
            //如果不满足，则尝试使用Gdi32方式
            else
            {
                var adjustScreenByGdi32 = new AdjustScreenByGdi32();
                var adjustScreenByGdi32Value = adjustScreenByGdi32.GetBrightness(handle, ref min, ref current, ref max);
                if (adjustScreenByGdi32Value)
                    return adjustScreenByGdi32;
            }

            return null;
        }
    }

    public class AdjustScreenByDxva2 : IAdjustScreen
    {
        [DllImport("dxva2.dll")]
        public static extern bool SetMonitorBrightness(IntPtr hMonitor, short brightness);

        [DllImport("dxva2.dll")]
        public static extern bool GetMonitorBrightness(IntPtr hMonitor, ref short pdwMinimumBrightness,
            ref short pdwCurrentBrightness, ref short pdwMaximumBrightness);

        [DllImport("dxva2.dll")]
        public static extern bool GetNumberOfPhysicalMonitorsFromHMONITOR(IntPtr hMonitor,
            ref uint pdwNumberOfPhysicalMonitors);

        [DllImport("dxva2.dll")]
        public static extern bool GetPhysicalMonitorsFromHMONITOR(IntPtr hMonitor,
            uint dwPhysicalMonitorArraySize, [Out] PhysicalMonitor[] pPhysicalMonitorArray);

        [DllImport("user32.dll")]
        public static extern IntPtr MonitorFromWindow([In] IntPtr hwnd, uint dwFlags);

        /// <summary>
        /// internal类型不要修改，不希望外界构造。可通过<see cref="AdjustScreenBuilder.CreateAdjustScreen"/>创建
        /// 由于调整屏幕亮度有多种方案，不同的屏幕适配不同的方案。所以不需要外界做过多判断。
        /// </summary>
        internal AdjustScreenByDxva2()
        {
        }

        /// <summary>
        /// 设置屏幕亮度
        /// </summary>
        /// <param name="handle">所在屏幕窗口的句柄</param>
        /// <param name="brightness">亮度</param>
        /// <returns></returns>
        public bool SetBrightness(IntPtr handle, short brightness)
        {
            if (handle == IntPtr.Zero) return false;

            uint pdwNumberOfPhysicalMonitors = uint.MinValue;
            //获取屏幕所在的屏幕设备
            var hMonitor = MonitorFromWindow(handle, (uint) NativeConstantsEnum.MonitorDefaultToPrimary);
            GetNumberOfPhysicalMonitorsFromHMONITOR(hMonitor, ref pdwNumberOfPhysicalMonitors);
            var screen = new PhysicalMonitor[pdwNumberOfPhysicalMonitors];
            GetPhysicalMonitorsFromHMONITOR(hMonitor, pdwNumberOfPhysicalMonitors, screen);
            if (screen.Length <= 0) return false;
            return SetMonitorBrightness(screen[0].hPhysicalMonitor, brightness);
        }

        /// <summary>
        /// 设置屏幕亮度
        /// </summary>
        /// <param name="handle">所在屏幕窗口的句柄</param>
        /// <param name="minBrightness"></param>
        /// <param name="currentBrightness"></param>
        /// <param name="maxBrightness"></param>
        /// <returns></returns>
        public bool GetBrightness(IntPtr handle, ref short minBrightness,
            ref short currentBrightness,
            ref short maxBrightness)
        {
            if (handle == IntPtr.Zero) return false;
            uint pdwNumberOfPhysicalMonitors = uint.MinValue;
            //获取屏幕所在的屏幕设备
            var hMonitor = MonitorFromWindow(handle, (uint) NativeConstantsEnum.MonitorDefaultToPrimary);
            GetNumberOfPhysicalMonitorsFromHMONITOR(hMonitor, ref pdwNumberOfPhysicalMonitors);
            var screen = new PhysicalMonitor[pdwNumberOfPhysicalMonitors];
            GetPhysicalMonitorsFromHMONITOR(hMonitor, pdwNumberOfPhysicalMonitors, screen);
            if (screen.Length <= 0) return false;

            return GetMonitorBrightness(screen[0].hPhysicalMonitor, ref minBrightness,
                ref currentBrightness,
                ref maxBrightness);
        }
    }

    public class AdjustScreenByGdi32 : IAdjustScreen
    {
        [DllImport("gdi32.dll")]
        public static extern bool GetDeviceGammaRamp(IntPtr hDC, ref RAMP lpRamp);

        [DllImport("gdi32.dll")]
        public static extern bool SetDeviceGammaRamp(IntPtr hDC, ref RAMP lpRamp);

        private double CalAllGammaVal(RAMP ramp)
        {
            return Math.Round(((CalColorGammaVal(ramp.Blue) + CalColorGammaVal(ramp.Red) +
                                CalColorGammaVal(ramp.Green)) / 3), 2);
        }

        /// <summary>
        /// internal类型不要修改，不希望外界构造。可通过<see cref="AdjustScreenBuilder.CreateAdjustScreen"/>创建
        /// 由于调整屏幕亮度有多种方案，不同的屏幕适配不同的方案。所以不需要外界做过多判断。
        /// </summary>
        internal AdjustScreenByGdi32()
        {
        }

        private double CalColorGammaVal(ushort[] line)
        {
            var max = line.Max();
            var min = line[0];
            var index = Array.FindIndex(line, n => n == max);
            var gamma = Math.Round((((double) (max - min) / index) / 255), 2);
            return gamma;
        }

        /// <summary>
        /// 读取屏幕亮度
        /// </summary>
        /// <param name="handle"></param>
        /// <param name="minBrightness"></param>
        /// <param name="currentBrightness"></param>
        /// <param name="maxBrightness"></param>
        /// <returns></returns>
        public bool GetBrightness(IntPtr handle, ref short minBrightness, ref short currentBrightness,
            ref short maxBrightness)
        {
            handle = Graphics.FromHwnd(IntPtr.Zero).GetHdc();
            //0-50 亮度变化太小，所以从50开始
            minBrightness = 50;
            maxBrightness = 100;
            var ramp = default(RAMP);
            var deviceGammaRamp = GetDeviceGammaRamp(handle, ref ramp);
            currentBrightness = (short) ((deviceGammaRamp ? CalAllGammaVal(ramp) : 0.5) * 100);
            return deviceGammaRamp;
        }

        /// <summary>
        /// 设置屏幕亮度
        /// </summary>
        /// <param name="handle"></param>
        /// <param name="brightness"></param>
        /// <returns></returns>
        public bool SetBrightness(IntPtr handle, short brightness)
        {
            handle = Graphics.FromHwnd(IntPtr.Zero).GetHdc();
            double value = (double) brightness / 100;
            RAMP ramp = default(RAMP);
            ramp.Red = new ushort[256];
            ramp.Green = new ushort[256];
            ramp.Blue = new ushort[256];

            for (int i = 1; i < 256; i++)
            {
                var tmp = (ushort) (i * 255 * value);
                ramp.Red[i] = ramp.Green[i] = ramp.Blue[i] = Math.Max(ushort.MinValue, Math.Min(ushort.MaxValue, tmp));
            }

            var deviceGammaRamp = SetDeviceGammaRamp(handle, ref ramp);
            return deviceGammaRamp;
        }
    }

    public interface IAdjustScreen
    {
        bool GetBrightness(IntPtr handle, ref short minBrightness,
            ref short currentBrightness,
            ref short maxBrightness);

        bool SetBrightness(IntPtr handle, short brightness);
    }

    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi)]
    public struct RAMP
    {
        [MarshalAs(UnmanagedType.ByValArray, SizeConst = 256)]
        public UInt16[] Red;

        [MarshalAs(UnmanagedType.ByValArray, SizeConst = 256)]
        public UInt16[] Green;

        [MarshalAs(UnmanagedType.ByValArray, SizeConst = 256)]
        public UInt16[] Blue;
    }

    [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Unicode)]
    public struct PhysicalMonitor
    {
        public IntPtr hPhysicalMonitor;

        [MarshalAs(UnmanagedType.ByValTStr, SizeConst = 128)]
        public string szPhysicalMonitorDescription;
    }

    public enum NativeConstantsEnum
    {
        MonitorDefaultToNull,
        MonitorDefaultToPrimary,
        MonitorDefaultToNearest
    }
```





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。