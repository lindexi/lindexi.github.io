# WPF 让窗口激活作为前台最上层窗口的方法

在 WPF 中，如果想要使用代码控制，让某个窗口作为当前用户的输入的逻辑焦点的窗口，也就是在当前用户活动的窗口的最上层窗口，默认使用 Activate 方法，通过这个方法在大部分设备都可以做到激活窗口

<!--more-->
<!-- CreateTime:4/21/2020 2:15:04 PM -->

<!-- 发布 -->

但是在一些特殊的设备上，使用下面代码调起窗口只是在任务栏闪烁图标，而没有让窗口放在最上层

```csharp
window.Show();
window.Activate();
```

在大部分设备上，通过 Show 和 Activate 组合可以让窗口作为当前用户活动的，即使窗口之前是最小化或隐藏，都可以通过 Show 的方法显示

但是某些设备窗口被盖在其他的窗口的下面，此时的窗口的 window.IsActive 还是 true 但是调用 Activate 不会让窗口放在上层

我在网上看到好多小伙伴调用了 SetForegroundWindow 方法，其实现在 WPF 是开源的，可以看到 Window 的 Activate 方法是这样写

```csharp
        public bool Activate()
        {
            // this call ends up throwing an exception if Activate
            // is not allowed
            VerifyApiSupported();
            VerifyContextAndObjectState();
            VerifyHwndCreateShowState();

            // Adding check for IsCompositionTargetInvalid
            if (IsSourceWindowNull || IsCompositionTargetInvalid)
            {
                return false;
            }

            return UnsafeNativeMethods.SetForegroundWindow(new HandleRef(null, CriticalHandle));
        }
```

源代码请看 [github](https://github.com/dotnet/wpf/blob/d3b4fa3b42e245701bf215794ebf763281cd81a5/src/Microsoft.DotNet.Wpf/src/PresentationFramework/System/Windows/Window.cs#L501-L516)

也就是调用 SetForegroundWindow 和调用 Activate 方法是差不多的，如果调用 Activate 没有用那么应该调用 SetForegroundWindow 也差不多

通过大佬的 [SetForegroundWindow的正确用法 - 子坞 - 博客园](https://www.cnblogs.com/ziwuge/archive/2012/01/06/2315342.html ) 可以了解到，需要按照以下步骤

```
　　　　1.得到窗口句柄FindWindow 
　　　　2.切换键盘输入焦点AttachThreadInput 
　　　　3.显示窗口ShowWindow(有些窗口被最小化/隐藏了) 
　　　　4.更改窗口的Zorder，SetWindowPos使之最上，为了不影响后续窗口的Zorder,改完之后，再还原 
　　　　5.最后SetForegroundWindow 
```

在 WPF 中对应的更改窗口的顺序使用的是 Topmost 属性，同时设置顺序需要做一点小的更改

在 WPF 中通过 [c# - Bring a window to the front in WPF - Stack Overflow](https://stackoverflow.com/questions/257587/bring-a-window-to-the-front-in-wpf ) 可以了解到如何用 AttachThreadInput 方法

整个代码请看下面，具体的 win32 方法我就没有写出来了，请小伙伴自己添加

```csharp
        private static void SetWindowToForegroundWithAttachThreadInput(Window window)
        {
            var interopHelper = new WindowInteropHelper(window);
            // 以下 Win32 方法可以在 https://github.com/kkwpsv/lsjutil/tree/master/Src/Lsj.Util.Win32 找到
            var thisWindowThreadId = Win32.User32.GetWindowThreadProcessId(interopHelper.Handle, IntPtr.Zero);
            var currentForegroundWindow = Win32.User32.GetForegroundWindow();
            var currentForegroundWindowThreadId = Win32.User32.GetWindowThreadProcessId(currentForegroundWindow, IntPtr.Zero);

            // [c# - Bring a window to the front in WPF - Stack Overflow](https://stackoverflow.com/questions/257587/bring-a-window-to-the-front-in-wpf )
            // [SetForegroundWindow的正确用法 - 子坞 - 博客园](https://www.cnblogs.com/ziwuge/archive/2012/01/06/2315342.html )
            /*
               　　1.得到窗口句柄FindWindow 
            　　　　2.切换键盘输入焦点AttachThreadInput 
            　　　　3.显示窗口ShowWindow(有些窗口被最小化/隐藏了) 
            　　　　4.更改窗口的Zorder，SetWindowPos使之最上，为了不影响后续窗口的Zorder,改完之后，再还原 
            　　　　5.最后SetForegroundWindow 
             */
            Win32.User32.AttachThreadInput(currentForegroundWindowThreadId, thisWindowThreadId, true);

            window.Show();
            window.Activate();
            // 去掉和其他线程的输入链接
            Win32.User32.AttachThreadInput(currentForegroundWindowThreadId, thisWindowThreadId, false);

            // 用于踢掉其他的在上层的窗口
            window.Topmost = true;
            window.Topmost = false;

```

我测试了几个原本没有让窗口放在上层的设备，使用上面的代码可以设置，但是我不了解设置上面代码可能的坑是什么

附带 walterlv 的测试工具，可以用来拿到当前的 GetForegroundWindow 是哪个

[walterlv 的工具](https://github.com/walterlv/walterlv.demo/blob/master/Walterlv.Demo.WindowX/Walterlv.Demo.WindowX/Program.cs)

另外少君小伙伴写了一个有趣的库，里面封装了很多 win32 的方法，请看 [kkwpsv lsjutil](https://github.com/kkwpsv/lsjutil/tree/master/Src/Lsj.Util.Win32 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
