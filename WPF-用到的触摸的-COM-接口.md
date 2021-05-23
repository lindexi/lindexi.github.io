
# WPF 用到的触摸的 COM 接口

本文记录 WPF 用到的触摸的 COM 接口

<!--more-->


<!-- 发布 -->

## 消息

用到了 [WM_TABLET_ADDED](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-added?WT.mc_id=WD-MVP-5003260 ) 和 [WM_TABLET_DELETED](https://docs.microsoft.com/en-us/windows/win32/tablet/wm-tablet-deleted?WT.mc_id=WD-MVP-5003260 ) 消息

使用的代码是 `src\Microsoft.DotNet.Wpf\src\PresentationCore\System\Windows\Input\Stylus\Wisp\WispLogic.cs` 的 HandleMessage 方法

```csharp
        internal override void HandleMessage(WindowMessage msg, IntPtr wParam, IntPtr lParam)
        {
            switch (msg)
            {
                // 忽略代码
                case WindowMessage.WM_TABLET_ADDED:
                    OnTabletAdded((uint)NativeMethods.IntPtrToInt32(wParam));
                    break;

                case WindowMessage.WM_TABLET_DELETED:
                    OnTabletRemovedImpl((uint)NativeMethods.IntPtrToInt32(wParam), isInternalCall: true);
                    break;
            }
        }
```

## 接口

包括：


- [ITablet Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itablet?WT.mc_id=WD-MVP-5003260)
- [ITablet2 Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itablet2?WT.mc_id=WD-MVP-5003260)
- [ITablet3 Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itablet3?WT.mc_id=WD-MVP-5003260)
- [ITabletContextP Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletcontextp?WT.mc_id=WD-MVP-5003260)
- [ITabletCursor Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletcursor?WT.mc_id=WD-MVP-5003260)
- [ITabletCursorButton Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletcursorbutton?WT.mc_id=WD-MVP-5003260)
- [ITabletEventSink Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itableteventsink?WT.mc_id=WD-MVP-5003260)
- [ITabletManager Interface](https://docs.microsoft.com/en-us/windows/win32/tablet/itabletmanager?WT.mc_id=WD-MVP-5003260)


基本引用代码在 `src\Microsoft.DotNet.Wpf\src\PenImc\inc\tpcpen.h` 文件，这是 WPF 的 PenImc 层

对此的封装是 `src\Microsoft.DotNet.Wpf\src\PenImc\dll\PimcTablet.cpp` 和 `src\Microsoft.DotNet.Wpf\src\PenImc\dll\PimcManager.cpp` 文件，封装依然作为 COM 方式提供

在框架顶层，在 `src\Microsoft.DotNet.Wpf\src\PresentationCore\System\Windows\Input\Stylus\Wisp\PenImcRcw.cs` 文件通过 COM 方法拿到

请看官方文档 [COM API Used by Windows Presentation Foundation - Win32 apps](https://docs.microsoft.com/en-us/windows/win32/tablet/com-apis-used-by-windows-presentation-foundation?WT.mc_id=WD-MVP-5003260 )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。