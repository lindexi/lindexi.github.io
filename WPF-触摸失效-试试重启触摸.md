
# WPF 触摸失效 试试重启触摸

在使用一些诡异的系统以及诡异的触摸框的时候，也许会出现 WPF 程序触摸失效，失效的本质原因是 Win32 层应用触摸失效。也许出现的问题是某个窗口设置 TopMost 然后插拔一些触摸设备等，这些行为，如果触摸设备太过诡异，也许就会让 Win32 窗口触摸失效。刚好 WPF 也是一个 Win32 窗口，此时的 WPF 也会触摸失效

<!--more-->


<!-- 发布 -->

这个方法因为过于强，我建议只有你在尝试过其他方法无法修复之后才能使用。本文的方法修复触摸是根据没有什么是重启解决不了的方法修复的，本文的方法将会使用反射调用 WPF 的代码，我仅仅有测试 .NET Framework 4.8 的框架里面的逻辑，这就意味着需要你在运行的设备上安装有 .NET Framework 4.8 框架，但是对于运行的 WPF 没有任何限制，可以使用 .NET Framework 4.5 甚至是 .NET Framework 4.0 的版本。当然，本文方法对于 .NET Core 3.1 和 .NET 5 同样生效

本文的核心逻辑就是调用 WispLogic 的 RegisterHwndForInput 和 UnRegisterHwndForInput 来实现重启触摸，但是没有真的结束触摸线程，因此不够彻底。而我自己基于开源的 WPF 框架也定制了可以从触摸线程都重启的强力版本，当然了，这个版本非开源的版本

在使用本文的方法之前，请确定你对触摸有足够的了解

如果你对触摸的了解很少，那么我推荐你先看以下博客

[WPF 触摸屏应用需要了解的知识](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%B1%8F%E5%BA%94%E7%94%A8%E9%9C%80%E8%A6%81%E4%BA%86%E8%A7%A3%E7%9A%84%E7%9F%A5%E8%AF%86.html )

[浅谈 Windows 桌面端触摸架构演进](https://blog.lindexi.com/post/%E6%B5%85%E8%B0%88-Windows-%E6%A1%8C%E9%9D%A2%E7%AB%AF%E8%A7%A6%E6%91%B8%E6%9E%B6%E6%9E%84%E6%BC%94%E8%BF%9B.html )

[WPF 客户端开发需要知道的触摸失效问题](https://blog.lindexi.com/post/WPF-%E5%AE%A2%E6%88%B7%E7%AB%AF%E5%BC%80%E5%8F%91%E9%9C%80%E8%A6%81%E7%9F%A5%E9%81%93%E7%9A%84%E8%A7%A6%E6%91%B8%E5%A4%B1%E6%95%88%E9%97%AE%E9%A2%98.html )

对于 Win32 应用来说，如果应用的触摸失效了，可以的解决方法是重新注册一次触摸，或者重启应用。而在 WPF 中，没有公开的方法可以让咱重启注册触摸，但是使用非公开的方法可以调用到。关于在 WPF 中的触摸调用细节请看 [WPF 触摸到事件](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E5%88%B0%E4%BA%8B%E4%BB%B6.html ) 和 [WPF 通过 InputManager 模拟调度触摸事件](https://blog.lindexi.com/post/WPF-%E9%80%9A%E8%BF%87-InputManager-%E6%A8%A1%E6%8B%9F%E8%B0%83%E5%BA%A6%E8%A7%A6%E6%91%B8%E4%BA%8B%E4%BB%B6.html )

重启注册触摸的步骤就是先反注册，然后再次注册。分别调用的是 WispLogic 的 UnRegisterHwndForInput 和 RegisterHwndForInput 方法，以下是步骤

在 WPF 中，可以使用下面代码获取 StylusLogic 对象

```csharp
        private object GetStylusLogic()
        {
            TabletDeviceCollection devices = System.Windows.Input.Tablet.TabletDevices;

            if (devices.Count > 0)
            {
                // Get the Type of InputManager.
                Type inputManagerType = typeof(System.Windows.Input.InputManager);

                // Call the StylusLogic method on the InputManager.Current instance.
                object stylusLogic = inputManagerType.InvokeMember("StylusLogic",
                    BindingFlags.GetProperty | BindingFlags.Instance | BindingFlags.NonPublic,
                    null, InputManager.Current, null);

                return stylusLogic;
            }

            return null;
        }
```

在没有开启 Pointer 的情况下，这里的 StylusLogic 就是 WispLogic 对象，因为 WispLogic 的定义如下

```csharp
internal class WispLogic : StylusLogic
{

}
```

在 Win10 中，大多数的触摸失效问题，都可以通过开启 Pointer 消息解决。而在 .NET 5 中，修复了 WPF 使用 WM_Pointer 消息在高 DPI 下的兼容触摸。如何开启 Pointer 消息请看 [WPF dotnet core 如何开启 Pointer 消息的支持](https://blog.lindexi.com/post/WPF-dotnet-core-%E5%A6%82%E4%BD%95%E5%BC%80%E5%90%AF-Pointer-%E6%B6%88%E6%81%AF%E7%9A%84%E6%94%AF%E6%8C%81.html )

在获取到 WispLogic 就可以通过反射调用 RegisterHwndForInput 和 UnRegisterHwndForInput 方法来重启触摸

通过开源的 WPF 代码可以看到两个方法的定义如下

```csharp
 internal void RegisterHwndForInput(InputManager inputManager, PresentationSource inputSource)
 {

 }

 internal void UnRegisterHwndForInput(HwndSource hwndSource)
 {

 }
```

这里的 InputManager 可以使用 InputManager.Current 获取，而 PresentationSource 可以使用 `PresentationSource.FromVisual(this)` 获取，上面的 `this` 需要一个在界面显示的元素

而 HwndSource 可以使用下面代码获取

```csharp
            var windowInteropHelper = new WindowInteropHelper(this);
            var hwndSource = HwndSource.FromHwnd(windowInteropHelper.Handle);
```

先调用 UnRegisterHwndForInput 禁用触摸，然后调用 RegisterHwndForInput 打开触摸

```csharp
            object stylusLogic = GetStylusLogic();

            if (stylusLogic == null)
            {
                return;
            }

            Type inputManagerType = typeof(System.Windows.Input.InputManager);
            var wispLogicType = inputManagerType.Assembly.GetType("System.Windows.Input.StylusWisp.WispLogic");

            var windowInteropHelper = new WindowInteropHelper(this);
            var hwndSource = HwndSource.FromHwnd(windowInteropHelper.Handle);

            var unRegisterHwndForInputMethodInfo = wispLogicType.GetMethod("UnRegisterHwndForInput",
                BindingFlags.Instance | BindingFlags.NonPublic);

            unRegisterHwndForInputMethodInfo.Invoke(stylusLogic, new object[] {hwndSource});


            var registerHwndForInputMethodInfo = wispLogicType.GetMethod("RegisterHwndForInput",
                BindingFlags.Instance | BindingFlags.NonPublic);

            registerHwndForInputMethodInfo.Invoke(stylusLogic, new object[]
            {
                InputManager.Current,
                PresentationSource.FromVisual(this)
            });
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bdf0e7a0/LeekailawnahelNarjailearyaydi ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bdf0e7a0/LeekailawnahelNarjailearyaydi ) 欢迎小伙伴访问

本文的方法不能解决内部逻辑调用问题的触摸失效问题，也不能解决太过诡异的系统的触摸失效问题。本文的重启触摸的方法的执行速度是很慢的

以上方法也是有缺点的，使用了上面方法之后，就不能使用 [高性能 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html ) 的方式。解决 DynamicRenderer 丢失的方法就是重新注册一次 StylusPlugIn 元素

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html ) 更多笔迹相关请看

- [WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html )
- [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)
- [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 
- [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )
- [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )
- [WPF 使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html )
- [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )
- [win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html )
- [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html)
- [WPF 笔迹触摸点收集工具](https://blog.lindexi.com/post/WPF-%E7%AC%94%E8%BF%B9%E8%A7%A6%E6%91%B8%E7%82%B9%E6%94%B6%E9%9B%86%E5%B7%A5%E5%85%B7.html )
- [WPF 实现自定义的笔迹橡皮擦](https://blog.lindexi.com/post/WPF-%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E7%AC%94%E8%BF%B9%E6%A9%A1%E7%9A%AE%E6%93%A6.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。