# WPF 启动屏幕键盘

在 Windows 的平板模式下才能自动在获取键盘输入焦点时弹出屏幕键盘，但是 Windows 的屏幕键盘做的粗糙，有时候不会自动开启屏幕键盘，此时需要使用代码辅助

<!--more-->
<!-- CreateTime:2020/8/19 14:48:07 -->



如果是非平板模式，以及系统没有检测到触摸，此时不一定能弹出屏幕键盘

在 Win10 版本小于 10.0.14393 时，可以通过启动 TabTip.exe 应用打开屏幕键盘。而在大于等于 10.0.14393 版本需要使用 COM 的方式

先来聊聊如何通过 TabTip.exe 应用打开屏幕键盘

默认的 TabTip.exe 应用将会放在 `Program Files` 文件夹下，可以通过如下代码拿到 `Program Files` 文件夹

```csharp
            var commonFilesPath = Environment.GetFolderPath(Environment.SpecialFolder.CommonProgramFiles);
            //程序集目标平台为X86时，获取到的是x86的Program Files，但TabTip.exe始终在Program Files目录下
            if (commonFilesPath.Contains("Program Files (x86)"))
            {
                commonFilesPath = commonFilesPath.Replace("Program Files (x86)", "Program Files");
            }
```

此时拿到应用路径可以使用下面代码

```csharp
            var tabTipPath = Path.Combine(commonFilesPath, @"microsoft shared\ink\TabTip.exe");
```

启动应用，启动之后需要等待一下，下面代码使用 `Thread.Sleep(50)` 等待，请小伙伴根据需要更改时间或更改为 Task.Delay 等。如果没有后续逻辑依赖键盘，那么可以删除 Thread.Sleep 的代码

```csharp
                var processStartInfo = new ProcessStartInfo
                {
                    FileName = tabTipPath,
                    UseShellExecute = true,
                    CreateNoWindow = true
                };
                Process.Start(processStartInfo);
                //第一次系统软键盘启动时候，需要缓冲一下
                Thread.Sleep(50);
```

如果是 10.0.14393 Windows 10周年纪念版 版本，可以使用 com 的方式启动，在启动之前，可以先判断一下版本号

```csharp
            var minWin10Version = new Version(10, 0, 14393, 0);
            var isNeedCom = Environment.OSVersion.Version >= minWin10Version;
```

注意，默认的 .NET 程序是不会让你获取 Environment.OSVersion 到 win10 的版本，详细请看 [关于C#中Environment.OSVersion判断操作系统及Win10上的问题 - 夏至千秋 - 博客园](https://www.cnblogs.com/chihirosan/p/5139078.html )

通过 COM 只有 Toggle 方法，也就是如果原本是没有开启的，调用将会开启。否则将会关闭

```csharp

  //使用com组件的方式来打开TabTip.exe
  var uiHostNoLaunch = new UIHostNoLaunch();
  // ReSharper disable once SuspiciousTypeConversion.Global
  var tipInvocation = uiHostNoLaunch as ITipInvocation;
  tipInvocation?.Toggle(Win32.User32.GetDesktopWindow());
  Marshal.ReleaseComObject(uiHostNoLaunch);

    [ComImport, Guid("4ce576fa-83dc-4F88-951c-9d0782b4e376")]
    class UIHostNoLaunch
    {
    }

    [ComImport, Guid("37c994e7-432b-4834-a2f7-dce1f13b834b")]
    [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
    interface ITipInvocation
    {
        void Toggle(IntPtr hwnd);
    }

    [DllImport("user32.dll", SetLastError = false)]
    static extern IntPtr GetDesktopWindow();
```


<!-- https://stackoverflow.com/a/48545074/6116637 -->

判断屏幕键盘是否开启，在 10.0.14393 Windows 10周年纪念版之前可以采用如下方法

```csharp
        private static IntPtr FindTipMainWindow()
        {
            return Win32.User32.FindWindow(KeyboardWindowClass, null);
        }

        private const string KeyboardWindowClass = "IPTip_Main_Window";

        private static bool GetIsOpenLegacy()
        {
            var touchHwnd = FindTipMainWindow();

            if (touchHwnd == IntPtr.Zero)
            {
                return false;
            }

            // 这里需要 unchecked 因为返回的是 int 转换为 WindowStyles 需要忽略负号
            var style = (Win32.WindowStyles) Win32.User32.GetWindowLongPtr(touchHwnd,
                (int) Win32.GetWindowLongFields.GWL_STYLE);
            // 如果满足了下面的条件就可以判断显示键盘
            // 由于有的系统在键盘不显示时候只是多返回一个WS_DISABLED这个字段。所以加一个它的判断
            return style.HasFlag(Win32.WindowStyles.WS_CLIPSIBLINGS)
                   && style.HasFlag(Win32.WindowStyles.WS_VISIBLE)
                   && style.HasFlag(Win32.WindowStyles.WS_POPUP)
                   && !style.HasFlag(Win32.WindowStyles.WS_DISABLED);
        }
```

如果是 10.0.14393 需要使用下面代码

```csharp
        private const string WindowParentClass = "ApplicationFrameWindow";
        private const string WindowClass = "Windows.UI.Core.CoreWindow";
        private const string WindowCaption = "Microsoft Text Input Application";

        private static bool? GetIsOpenKeyboardWindow()
        {
            // if there is a top-level window - the keyboard is closed
            var wnd = Win32.User32.FindWindowEx(IntPtr.Zero, IntPtr.Zero, WindowClass, WindowCaption);
            if (wnd != IntPtr.Zero)
                return false;

            var parent = Win32.User32.FindWindowEx(IntPtr.Zero, IntPtr.Zero, WindowParentClass, null);
            if (parent == IntPtr.Zero)
                return null; // no more windows, keyboard state is unknown

            // if it's a child of a WindowParentClass1709 window - the keyboard is open
            wnd = Win32.User32.FindWindowEx(parent, IntPtr.Zero, WindowClass, WindowCaption);
            if (wnd != IntPtr.Zero)
                return true;

            return null;
        }

 // 这是 Win32.User32 的方法
            public const string LibraryName = "user32";

            [DllImport(LibraryName, CharSet = Properties.BuildCharSet)]
            public static extern IntPtr FindWindowEx(IntPtr hwndParent, IntPtr hwndChildAfter, string lpszClass,
                string lpszWindow);

            [DllImport(LibraryName, CharSet = Properties.BuildCharSet)]
            public static extern IntPtr FindWindow(string lpClassName, string lpWindowName);
```

但是在 10.0.18362（1903） 版本，上面判断方法在一些设备上凉凉。可选采用 <https://github.com/microsoft/WindowsAppSDK/discussions/3343> 所述方法，通过 IFrameworkInputPane 的 [Location](https://learn.microsoft.com/en-us/windows/win32/api/shobjidl_core/nf-shobjidl_core-iframeworkinputpane-location) 方法，判断宽度高度为 0 断定屏幕键盘没有打开，如[官方文档](https://learn.microsoft.com/en-us/windows/win32/api/shobjidl_core/nf-shobjidl_core-iframeworkinputpane-location)所述

> `HRESULT Location([out] RECT *prcInputPaneScreenLocation);`
> 
> Parameters:
>
> `[out] prcInputPaneScreenLocation`
>
> A pointer to a RECT structure that, when this method returns successfully, receives the location of the input pane, in screen coordinates.
> If the input pane is not visible, this structure receives an empty rectangle.

关于使用 IFrameworkInputPane 判断屏幕键盘是否打开的详细使用方法请参阅 <https://stackoverflow.com/a/55513524>