# WPF 已知问题 某些设备上的应用在 WindowChromeWorker 抛出 System.OverflowException 异常

准确来说，这个不算是 WPF 的问题，而是系统等的问题。在某些设备上的使用了 WindowChrome 功能的 WPF 应用，将在运行过程，在 WindowChromeWorker 类里面抛出 System.OverflowException 异常。核心原因是这些设备是 x64 设备，运行的 x64 的 WPF 应用程序，在消息循环里面传入的 lParam 是一个 x64 的指针，但在 WPF 里面使用 ToInt32 方法进行转换，刚好此 x64 的指针超过 int 的范围，从而抛出异常

<!--more-->
<!-- CreateTime:2022/7/15 19:16:49 -->

<!-- 发布 -->

这是一个上古就存在的问题，有人报告说安装了某些驱动就会存在此异常，但是我没有调查到在符合什么情况下就会抛出此异常。此异常的调用堆栈大概如下

```
System.OverflowException
  HResult=0x80131516
  Message=Arithmetic operation resulted in an overflow.
  Source=PresentationFramework
  StackTrace:
   at System.Windows.Shell.WindowChromeWorker._HandleNCHitTest(WM uMsg, IntPtr wParam, IntPtr lParam, Boolean& handled)
   at System.Windows.Shell.WindowChromeWorker._WndProc(IntPtr hwnd, Int32 msg, IntPtr wParam, IntPtr lParam, Boolean& handled)
   at System.Windows.Interop.HwndSource.PublicHooksFilterMessage(IntPtr hwnd, Int32 msg, IntPtr wParam, IntPtr lParam, Boolean& handled)
   at MS.Win32.HwndWrapper.WndProc(IntPtr hwnd, Int32 msg, IntPtr wParam, IntPtr lParam, Boolean& handled)
   at MS.Win32.HwndSubclass.DispatcherCallbackOperation(Object o)
   at System.Windows.Threading.ExceptionWrapper.InternalRealCall(Delegate callback, Object args, Int32 numArgs)
   at System.Windows.Threading.ExceptionWrapper.TryCatchWhen(Object source, Delegate callback, Object args, Int32 numArgs, Delegate catchHandler)
   at System.Windows.Threading.Dispatcher.LegacyInvokeImpl(DispatcherPriority priority, TimeSpan timeout, Delegate method, Object args, Int32 numArgs)
   at MS.Win32.HwndSubclass.SubclassWndProc(IntPtr hwnd, Int32 msg, IntPtr wParam, IntPtr lParam)
```

异常的中文描述是 算术运算导致溢出

通过阅读代码可以了解到是在 `WindowChromeWorker._WndProc` 方法里面接收到 Windows 消息，在 `_HandleNCHitTest` 转换 `lParam` 参数读取时抛出异常，以下是此方法的有删减的代码

```csharp
        private IntPtr _HandleNCHitTest(WM uMsg, IntPtr wParam, IntPtr lParam, out bool handled)
        {
            DpiScale dpi = _window.GetDpi();

            // Let the system know if we consider the mouse to be in our effective non-client area.
            var mousePosScreen = new Point(Utility.GET_X_LPARAM(lParam), Utility.GET_Y_LPARAM(lParam));
            Rect windowPosition = _GetWindowRect();

            // 忽略代码
        }
```

现在的 WPF 是基于最友好的 MIT 协议开源的，所有的源代码都可以从 https://github.com/dotnet/wpf 下载到。如果对以上的 `_HandleNCHitTest` 方法感兴趣，想要阅读完全的代码，还请到官方开源仓库获取

通过以上代码可以看到，使用 Utility 的 GET_X_LPARAM 方法从 `lParam` 参数获取一个点的 x 参数。从业务分析上，实际上 `lParam` 转换的 x 参数是作为屏幕的坐标点，而屏幕的坐标点在 2022 时还是一个 Int16 范围的值。换句话说，如果 `lParam` 转换出一个 Int64 的长度，那一定是不符合预期的

在 `GET_X_LPARAM` 方法里面，将会通过 `ToInt32` 的方法进行转换，且取其低位。在 `GET_Y_LPARAM` 里，将会转换 `lParam` 为 Int32 取高位。而抛出的异常就是 ToInt32 这个方法

如果传入了一个 IntPtr 是一个 long 的值，那将会在 ToInt32 方法抛出 OverflowException 异常

我将此问题报告给 WPF 官方，请看 [System.OverflowException in PresentationFramework.dll in System.Windows.Shell.WindowChromeWorker · Issue #6777 · dotnet/wpf](https://github.com/dotnet/wpf/issues/6777 )

然而有趣的是我就是 WPF 官方开发者，于是我就自己修复了这个问题，请看 [Fix System.OverflowException in WindowChromeWorker._HandleNCHitTest by lindexi · Pull Request #6779 · dotnet/wpf](https://github.com/dotnet/wpf/pull/6779 )

我的修复的方法是转换为 Long 再进行裁剪，这个做法我认为是对的，我也阅读了一些相关的对 NCHitTest 消息处理的博客，例如 [当无边框窗口被子窗口遮挡导致难以调节窗口大小时，可通过处理 NCHITTEST 消息重新支持调节窗口大小 - walterlv](https://blog.walterlv.com/post/handle-nchittest-message-to-support-resize-even-if-window-is-covered-with-child-windows.html ) 也都是如此处理

不过在我的代码还没被合入之前，可以先采用以下代码减少抛出异常

```csharp
      protected override void OnSourceInitialized( EventArgs e )
      {
         base.OnSourceInitialized( e );
         ( (HwndSource)PresentationSource.FromVisual( this ) ).AddHook( HookProc );
      }
      private IntPtr HookProc( IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled )
      {
         if ( msg == 0x0084 /*WM_NCHITTEST*/ )
         {
         	// 如果尝试转换失败，那就吃掉这个消息
            // This prevents a crash in WindowChromeWorker._HandleNCHitTest
            try
            {
               lParam.ToInt32();
            }
            catch ( OverflowException )
            {
               handled = true;
            }
         }
         return IntPtr.Zero;
      }
```

更多的相关链接：

[System.OverflowException in PresentationFramework.dll in System.Windows.Shell.WindowChromeWorker microsoft/dotnet#689](https://github.com/microsoft/dotnet/issues/689)

[OverflowException when converting 64-bit IntPtr to Int32 ControlzEx/ControlzEx#30](https://github.com/ControlzEx/ControlzEx/issues/30)

[Arithmetic operation resulted in an overflow. MahApps/MahApps.Metro#3301](https://github.com/MahApps/MahApps.Metro/issues/3301)

[Exception:算术运算导致溢出。 HandyOrg/HandyControl#886](https://github.com/HandyOrg/HandyControl/issues/886)

[https://developercommunity.visualstudio.com/t/overflow-exception-in-windowchrome/167357](https://developercommunity.visualstudio.com/t/overflow-exception-in-windowchrome/167357)

[https://stackoverflow.com/questions/33287542/what-would-cause-wm-nchittest-lparam-to-overflow-a-32-bit-integer](https://stackoverflow.com/questions/33287542/what-would-cause-wm-nchittest-lparam-to-overflow-a-32-bit-integer)
