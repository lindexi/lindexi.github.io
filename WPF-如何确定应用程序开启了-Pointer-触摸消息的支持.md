
# WPF 如何确定应用程序开启了 Pointer 触摸消息的支持

因为 WPF 在开启 Pointer 和没有开启的基础表现几乎相同，因此从业务层很难了解到当前是否开启了 Pointer 消息。本文从开发者的角度，通过 Windows 消息判断当前是否开启 Pointer 支持

<!--more-->


<!-- 发布 -->

在 [win10 支持默认把触摸提升 Pointer 消息](https://blog.lindexi.com/post/win10-%E6%94%AF%E6%8C%81%E9%BB%98%E8%AE%A4%E6%8A%8A%E8%A7%A6%E6%91%B8%E6%8F%90%E5%8D%87-Pointer-%E6%B6%88%E6%81%AF.html) 告诉大家如何在 Win10 下让 WPF 在 .NET 4.7 和以上框架支持 Pointer 消息

那么如何确定这个 WPF 程序我写对了，开启了 Pointer 消息？可以通过监听 Window 消息，如果能收到 Pointer 的消息，那么算开启成功

不需要在用户端判断，用户端只需要判断 运行的系统是 `Windows 10 Creators Update` 1703 10.0.15063 就可以。因此本文更多是给开发端，开发的时候通过此方法可以确定是否开启了 Pointer 消息

在 [WPF 添加窗口消息钩子方法](https://blog.lindexi.com/post/WPF-%E6%B7%BB%E5%8A%A0%E7%AA%97%E5%8F%A3%E6%B6%88%E6%81%AF%E9%92%A9%E5%AD%90%E6%96%B9%E6%B3%95.html) 这篇博客告诉大家如何拿到窗口的消息

在这个基础上，尝试在拿到消息判断是否 Pointer 消息，如果能收到 Pointer 消息，那么证明代码没写错

```csharp
       public MainWindow()
        {
            InitializeComponent();

            SourceInitialized += OnSourceInitialized;
        }

        private void OnSourceInitialized(object sender, EventArgs e)
        {
            var windowInteropHelper = new WindowInteropHelper(this);
            var hwnd = windowInteropHelper.Handle;

            HwndSource source = HwndSource.FromHwnd(hwnd);
            source.AddHook(Hook);
        }

        private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            const int WM_POINTERDOWN = 0x0246;

            if (msg == WM_POINTERDOWN)
            {
                // 开启了 Pointer 消息
                Debugger.Break();
            }


            return IntPtr.Zero;
        }
```

如果能进入 `msg == WM_POINTERDOWN` 那么就是收到 Pointer 消息了





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。