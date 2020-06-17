# WPF 设置 WS_EX_TRANSPARENT 触摸失效

调用 SetWindowLong 方法给 GWL_EXSTYLE 设置 WS_EX_TRANSPARENT 让窗口透明，此时应用程序只能收到鼠标消息但收不到触摸消息

<!--more-->
<!-- CreateTime:6/16/2020 8:46:23 AM -->

<!-- 发布 -->

最简单的 demo 是在 Load 事件添加下面代码

```csharp
        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            IntPtr hwnd = new WindowInteropHelper(this).Handle;
            var extendedStyle = GetWindowLong(hwnd, GWL_EXSTYLE);
            SetWindowLong(hwnd, GWL_EXSTYLE, extendedStyle | WS_EX_TRANSPARENT);
        }
```

此时可以发现窗口的 MouseDown 能收到，但是 TouchDown 事件收不到

最简单的 demo 放在 [github](https://github.com/dotnet-campus/wpf-issues/tree/master/CanNotReceiveTouchMessageWS_EX_TRANSPARENT) 欢迎小伙伴访问

此问题报告为官方，请看 [WPF can not receive the touch message when set WS_EX_TRANSPARENT to window](https://github.com/dotnet/wpf/issues/3145 )

因为小伙伴很少会用到这个方法设置透明，因此此问题影响很小

正规的透明设置方法请看 [WPF 制作高性能的透明背景异形窗口（使用 WindowChrome 而不要使用 AllowsTransparency=True） - walterlv](https://blog.walterlv.com/post/wpf-transparent-window-without-allows-transparency.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
