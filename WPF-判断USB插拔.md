
# WPF 判断USB插拔

本文告诉大家如何在 WPF 在用户插拔 USB 收到消息

<!--more-->


<!-- csdn -->

首先需要在一个窗口重写`OnSourceInitialized`，在这里可以拿到窗口的指针

```csharp
        protected override void OnSourceInitialized(EventArgs e)
        {
            base.OnSourceInitialized(e);

            var hwndSource = PresentationSource.FromVisual(this) as HwndSource;
            hwndSource?.AddHook(new HwndSourceHook(WndProc));
        }
```

在 USB 插拔可以收到 DEVICECHANGE 消息

```csharp
        private IntPtr WndProc(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            if (msg == (int) WM.DEVICECHANGE)
            {
                Debug.WriteLine(DateTime.Now.ToString() + " " + "设备发生插拔\r\n");
            }
            return IntPtr.Zero;
        }
```

这里的 WM.DEVICECHANGE 就是 537 ，关于其他的消息请看[win 消息](https://lindexi.gitee.io/post/win-%E6%B6%88%E6%81%AF.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。