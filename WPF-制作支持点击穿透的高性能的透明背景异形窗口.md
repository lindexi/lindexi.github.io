
# WPF 制作支持点击穿透的高性能的透明背景异形窗口

默认的 WPF 的支持点击穿透的透明背景窗口，是通过 AllowsTransparency 实现的，但是此方法的性能比较低。本文来告诉大家一个高性能的方法，通过此方法制作出来的 WPF 窗口可以获取很高的性能，设置透明和设置窗口不透明之间几乎没有性能差别

<!--more-->


<!-- 发布 -->

本文的方法由 [少珺](https://blog.sdlsj.net/) 小伙伴提供，我只是代为整理博客。本文的方法是基于 [WPF 制作高性能的透明背景异形窗口（使用 WindowChrome 而不要使用 AllowsTransparency=True） - walterlv](https://blog.walterlv.com/post/wpf-transparent-window-without-allows-transparency.html ) 但是 walterlv 大大的方法没有提供可穿透的功能，而本文是提供了全穿透的功能

默认的 WPF 提供的 AllowsTransparency 的方法，这个方法可以适用在让窗口透明的部分能点击穿透，窗口不透明部分点击不穿透。但根据 [WPF 从最底层源代码了解 AllowsTransparency 性能差的原因](https://blog.lindexi.com/post/WPF-%E4%BB%8E%E6%9C%80%E5%BA%95%E5%B1%82%E6%BA%90%E4%BB%A3%E7%A0%81%E4%BA%86%E8%A7%A3-AllowsTransparency-%E6%80%A7%E8%83%BD%E5%B7%AE%E7%9A%84%E5%8E%9F%E5%9B%A0.html ) 可以了解到此方法的性能比较低

本文提供的方法是使用 [WPF 制作高性能的透明背景异形窗口（使用 WindowChrome 而不要使用 AllowsTransparency=True） - walterlv](https://blog.walterlv.com/post/wpf-transparent-window-without-allows-transparency.html )  来实现高性能的，同时通过 WS_EX_TRANSPARENT 设置整个窗口全穿透

因此本文的方法是要么整个窗口透明不穿透，要么就是整个窗口透明穿透。而做不到和 WPF 提供的 AllowsTransparency 的方法让透明的部分支持穿透。但本文的方法的性能特别强

在开始之前，请完全抄袭 [WPF 制作高性能的透明背景异形窗口（使用 WindowChrome 而不要使用 AllowsTransparency=True） - walterlv](https://blog.walterlv.com/post/wpf-transparent-window-without-allows-transparency.html )  这篇博客的内容

接下来给上面的这个方法添加支持全窗口点击穿透功能，因为本文使用到 WS_EX_TRANSPARENT 的方法设置窗口全穿透，此时需要给窗口加上 WS_EX_LAYERED 样式。而在 WPF 中，如果窗口在未设置 AllowsTransparency = true 时，会自动去掉 WS_EX_LAYERED 样式。根据完全开源的 WPF 仓库，可以找到这段逻辑，放在 HwndTarget 类，如下面代码

```csharp
    public class HwndTarget : CompositionTarget
    {
        /// <summary>
        /// The HwndTarget needs to see all windows messages so that
        /// it can appropriately react to them.
        /// </summary>
        internal IntPtr HandleMessage(WindowMessage msg, IntPtr wparam, IntPtr lparam)
        {
            switch (msg)
            {
                	// 忽略其他代码
                case WindowMessage.WM_STYLECHANGING:
                    unsafe
                    {
                        NativeMethods.STYLESTRUCT * styleStruct = (NativeMethods.STYLESTRUCT *) lparam;

                        if ((int)wparam == NativeMethods.GWL_EXSTYLE)
                        {
                        	// 这里的 UsesPerPixelOpacity 属性就是由 AllowsTransparency 决定的
                            if(UsesPerPixelOpacity)
                            {
                                // We need layered composition to accomplish per-pixel opacity.
                                //
                                styleStruct->styleNew |= NativeMethods.WS_EX_LAYERED;
                            }
                            else
                            {
                                // No properties that require layered composition exist.
                                // Make sure the layered bit is off.
                                //
                                // Note: this prevents an external program from making
                                // us system-layered (if we are a top-level window).
                                //
                                // If we are a child window, we still can't stop our
                                // parent from being made system-layered, and we will
                                // end up leaving visual artifacts on the screen under
                                // WindowsXP.
                                //
                                styleStruct->styleNew &= (~NativeMethods.WS_EX_LAYERED);
                            }
                        }
                    }

                    break;
             }
        }
    }
```

为了能够让 WPF 支持在没有设置 AllowsTransparency = true 时也能设置为 WS_EX_LAYERED 样式，就需要使用一点 Hack 的代码，感谢 [少珺](https://blog.sdlsj.net/) 小伙伴找到这个有趣的方法。在 WPF 机制里面，添加 AddHook 执行逻辑是有顺序的，而上面代码的 HandleMessage 其实也是一个消息循环的 Hook 的逻辑。为了让 WPF 支持设置 WS_EX_LAYERED 样式，可以在上面 HwndTarget 的逻辑运行完成之后，运行咱自己的逻辑，再设置一遍。此时因为咱的逻辑在 HwndTarget 之后执行，因此咱的逻辑就覆盖了 HwndTarget 的设置

在窗口的 Loaded 事件里面添加下面代码

```csharp
        private void PerformanceDesktopTransparentWindow_Loaded(object sender, RoutedEventArgs e)
        {
            ((HwndSource)PresentationSource.FromVisual(this)).AddHook((IntPtr hwnd, int msg, IntPtr wParam, IntPtr lParam, ref bool handled) =>
            {
                //想要让窗口透明穿透鼠标和触摸等，需要同时设置 WS_EX_LAYERED 和 WS_EX_TRANSPARENT 样式，
                //确保窗口始终有 WS_EX_LAYERED 这个样式，并在开启穿透时设置 WS_EX_TRANSPARENT 样式
                //但是WPF窗口在未设置 AllowsTransparency = true 时，会自动去掉 WS_EX_LAYERED 样式（在 HwndTarget 类中)，
                //如果设置了 AllowsTransparency = true 将使用WPF内置的低性能的透明实现，
                //所以这里通过 Hook 的方式，在不使用WPF内置的透明实现的情况下，强行保证这个样式存在。
                if (msg == (int)Win32.WM.STYLECHANGING && (long)wParam == (long)Win32.GetWindowLongFields.GWL_EXSTYLE)
                {
                    var styleStruct = (STYLESTRUCT)Marshal.PtrToStructure(lParam, typeof(STYLESTRUCT));
                    styleStruct.styleNew |= (int)Win32.ExtendedWindowStyles.WS_EX_LAYERED;
                    Marshal.StructureToPtr(styleStruct, lParam, false);
                    handled = true;
                }
                return IntPtr.Zero;
            });
        }
```

此时就完成了让窗口设置 WS_EX_LAYERED 这个样式的功能了，以上代码完成之后，在设置窗口是否点击穿透，就可以用上 WS_EX_TRANSPARENT 样式了，如下面代码

```csharp
        /// <summary>
        /// 设置点击穿透到后面透明的窗口
        /// </summary>
        public void SetTransparentHitThrough()
        {
            if (_dwmEnabled)
            {
                Win32.User32.SetWindowLongPtr(_hwnd, Win32.GetWindowLongFields.GWL_EXSTYLE,
                    (IntPtr)(int)((long)Win32.User32.GetWindowLongPtr(_hwnd, Win32.GetWindowLongFields.GWL_EXSTYLE) | (long)Win32.ExtendedWindowStyles.WS_EX_TRANSPARENT));
            }
            else
            {
                Background = Brushes.Transparent;
            }
        }

        /// <summary>
        /// 设置点击命中，不会穿透到后面的窗口
        /// </summary>
        public void SetTransparentNotHitThrough()
        {
            if (_dwmEnabled)
            {
                Win32.User32.SetWindowLongPtr(_hwnd, Win32.GetWindowLongFields.GWL_EXSTYLE,
                    (IntPtr)(int)((long)Win32.User32.GetWindowLongPtr(_hwnd, Win32.GetWindowLongFields.GWL_EXSTYLE) & ~(long)Win32.ExtendedWindowStyles.WS_EX_TRANSPARENT));
            }
            else
            {
                Background = BrushCreator.GetOrCreate("#0100000");
            }
        }
```

通过 WS_EX_TRANSPARENT 样式，就能设置窗口是否全穿透。上面代码用到了我定义的 Win32 的相关方法，这部分代码很多用到了 Enum 枚举的二进制计算方法，因此看起来相对复杂一点

细心的小伙伴会看到，其实我是区分了 `_dwmEnabled` 才决定是否使用 WS_EX_TRANSPARENT 的方式设置透明，原因是 [WPF 制作高性能的透明背景异形窗口（使用 WindowChrome 而不要使用 AllowsTransparency=True） - walterlv](https://blog.walterlv.com/post/wpf-transparent-window-without-allows-transparency.html ) 的方法只支持在有开启 DWM 的模式下才能用上，否则透明部分会显示黑色

判断是否开启 DWM 可以使用 Dwmapi.dll 提供的 DwmIsCompositionEnabled 方法，如下面代码

```csharp
        public static class Dwmapi
        {
            public const string LibraryName = "Dwmapi.dll";

            [DllImport(LibraryName, ExactSpelling = true, PreserveSig = false)]
            [return: MarshalAs(UnmanagedType.Bool)]
            public static extern bool DwmIsCompositionEnabled();
        }
```

在 win7 系统，可以动态更改这个值。但是在 Win10 系统默认都是开启的

如果没有开启 DwmIsCompositionEnabled 那么依然只能使用 AllowsTransparency 的方式设置透明

本文的没有在博客写的代码包括了，如何设置窗口样式以及 win32 方法的定义，这些代码我都放在 [github](https://github.com/lindexi/lindexi_gd/tree/b26274ae/RuhuyagayBemkaijearfear) 欢迎小伙伴访问，这里面包含了所有逻辑，包括博客里面没有放的代码

尽管上面代码有点 Hack 但我已经在尝试在产品级使用了，暂时还没有发现什么锅





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。