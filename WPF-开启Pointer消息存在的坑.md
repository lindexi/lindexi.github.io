
# WPF 开启Pointer消息存在的坑

本文记录在 WPF 开启 Pointer 消息的坑

<!--more-->


<!-- CreateTime:2019/12/24 14:33:41 -->

<!-- 发布 -->

## 屏幕键盘

启用了Pointer之后，调用Textbox.Focus()，起不来屏幕键盘，必须点在它之上才行，触摸在它之上才行

## 使用屏幕绝对坐标而不是窗口坐标

默认 Pointer 消息是使用屏幕绝对坐标而不是窗口坐标

可能存在获取 Stylus 事件时触摸点不准，此时可以通过获取 Touch 代替，详细请看 [WPF will have a touch offset after trun on the WM_Pointer message · Issue #3360 · dotnet/wpf](https://github.com/dotnet/wpf/issues/3360 ) 此问题应该在 [Fix raw stylus data to support per-monitor DPI by rladuca · Pull Request #2891 · dotnet/wpf](https://github.com/dotnet/wpf/pull/2891 ) 修复

## 开启 Pointer 消息之后无法隐藏触摸反馈点

开启 Pointer 消息之后，调用 `Stylus.IsPressAndHoldEnabled="False"` 无效

![](http://image.acmx.xyz/lindexi%2FWPF%2520can%2520not%2520work%2520well%2520with%2520set%2520IsPressAndHoldEnabled%2520to%2520false%2520when%2520enable%2520pointer%2520message.gif)

在没有开启 Pointer 消息，将会在 System.Windows.Interop.HwndSource 的 Initialize 方法通过判断是否开启 Pointer 消息执行 HwndStylusInputProvider 逻辑

```csharp
            if (StylusLogic.IsStylusAndTouchSupportEnabled)
            {
                // Choose between Wisp and Pointer stacks
                if (StylusLogic.IsPointerStackEnabled)
                {
                	// 开启 Pointer 的逻辑
                    _stylus = new SecurityCriticalDataClass<IStylusInputProvider>(new HwndPointerInputProvider(this));
                }
                else
                {
                    _stylus = new SecurityCriticalDataClass<IStylusInputProvider>(new HwndStylusInputProvider(this));
                }
            }
```

在 HwndStylusInputProvider 将会读取 IsPressAndHoldEnabledProperty 属性，然后使用 WM_TABLET_QUERYSYSTEMGESTURESTATUS 返回 1 的方式告诉系统不显示触摸反馈点。也就是 WPF 隐藏触摸反馈点是通过 [How do I disable the press-and-hold gesture for my window](https://devblogs.microsoft.com/oldnewthing/20170227-00/?p=95585 ) 的方法

如果不设置 `Stylus.IsPressAndHoldEnabled="False"` 也可以自己手动监听消息，在消息 WM_TABLET_QUERYSYSTEMGESTURESTATUS 里面返回 1 就可以告诉系统不显示触摸反馈点

```csharp
       private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            const int WM_TABLET_DEFBASE =0x02C0;
            const int WM_TABLET_QUERYSYSTEMGESTURESTATUS = WM_TABLET_DEFBASE + 12;
            const int WM_TABLET_FLICK = WM_TABLET_DEFBASE + 11;

            if (msg == WM_TABLET_QUERYSYSTEMGESTURESTATUS)
            {
                uint flags = 0;

                flags |= TABLET_PRESSANDHOLD_DISABLED;
                flags |= TABLET_TAPFEEDBACK_DISABLED;
                flags |= TABLET_TOUCHUI_FORCEON;
                flags |= TABLET_TOUCHUI_FORCEOFF;
                flags |= TABLET_FLICKS_DISABLED;

                handled = true;
                return new IntPtr(flags);
            }
            else if (msg == WM_TABLET_FLICK)
            {
                handled = true;
                return new IntPtr(1);
            }

            return IntPtr.Zero;
        }

        private const uint TABLET_PRESSANDHOLD_DISABLED = 0x00000001;
        private const uint TABLET_TAPFEEDBACK_DISABLED = 0x00000008;
        private const uint TABLET_TOUCHUI_FORCEON = 0x00000100;
        private const uint TABLET_TOUCHUI_FORCEOFF = 0x00000200;
        private const uint TABLET_FLICKS_DISABLED = 0x00010000;
```

但如果开启了 Pointer 消息，那么这个机制将会无效，即使依然是手动监听消息，如 [https://github.com/lindexi/lindexi_gd/tree/81b2a63a/KemjawyecawDurbahelal](https://github.com/lindexi/lindexi_gd/tree/81b2a63a/KemjawyecawDurbahelal) 的代码，也是无效的

问题报告给了 WPF 官方，请看 [WPF can not work well with set IsPressAndHoldEnabled to false when enable pointer message · Issue #3379 · dotnet/wpf](https://github.com/dotnet/wpf/issues/3379 ) 但预计不会在 WPF 中修复，原因是这是 Windows 的 WM_Pointer 机制的坑，和 WPF 其实没有关系

另一个解决方法是在关闭系统全局触摸反馈点，关闭方法请看 [3 Ways to Enable or Disable Touch Feedback in Windows 10](https://www.top-password.com/blog/enable-or-disable-touch-feedback-in-windows-10/ )

更多请看

[WPF can not work well with set IsPressAndHoldEnabled to false when enable pointer message · Issue #3379 · dotnet/wpf](https://github.com/dotnet/wpf/issues/3379 )

[How do I disable the press-and-hold gesture for my window](https://devblogs.microsoft.com/oldnewthing/20170227-00/?p=95585 )

[WM_TABLET_QUERYSYSTEMGESTURESTATUS message (Tpcshrd.h)](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-querysystemgesturestatus-message )

[WM_TABLET_FLICK message (Tpcshrd.h)](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-flick-message )

[c# - Calling SetGestureConfig method affects onmousemove override of control - Stack Overflow](https://stackoverflow.com/questions/53274234/calling-setgestureconfig-method-affects-onmousemove-override-of-control )

[SetGestureConfig 函数-中文整理_Augusdi的专栏-CSDN博客](https://blog.csdn.net/Augusdi/article/details/8885396 )

## 更多阅读

[WPF 如何确定应用程序开启了 Pointer 触摸消息的支持](https://blog.lindexi.com/post/WPF-%E5%A6%82%E4%BD%95%E7%A1%AE%E5%AE%9A%E5%BA%94%E7%94%A8%E7%A8%8B%E5%BA%8F%E5%BC%80%E5%90%AF%E4%BA%86-Pointer-%E8%A7%A6%E6%91%B8%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html )

[win10 支持默认把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html) 

[WPF dotnet core 如何开启 Pointer 消息的支持](https://blog.lindexi.com/post/WPF-dotnet-core-%E5%A6%82%E4%BD%95%E5%BC%80%E5%90%AF-Pointer-%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。