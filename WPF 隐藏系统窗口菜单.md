# WPF 隐藏系统窗口菜单

本文告诉大家如何隐藏系统窗口菜单

<!--more-->
<!-- CreateTime:2019/6/5 17:26:44 -->

<!-- csdn -->

系统的窗口菜单请看下图

<!-- ![](image/WPF 隐藏系统窗口菜单/WPF 隐藏系统窗口菜单0.png) -->

![](http://image.acmx.xyz/lindexi%2F201965171728198)

通过在消息里面钩调一些消息的方式，此方法由 黄滨滨 大佬提供

```csharp
        private IntPtr Hook(IntPtr hwnd, int msg, IntPtr wparam, IntPtr lparam, ref bool handled)
        {
            if (msg == 0x112)
            {
                var param = wparam.ToInt32();
                if (param is 0xf093 // 单击打开菜单
                    || param is 0xf100)//键盘打开菜单
                {
                    handled = true;
                }
            }
            else if (msg == 0xa4)
            {
                var param = wparam.ToInt32();
                if (param == 0x02 // 非图片客户区
                    || param is 0x03)
                {
                    handled = true;
                }
            }

            return IntPtr.Zero;
        }
```

第二个方法是通过设置样式

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

            var windowLong = GetWindowLong(hwnd, GWL_STYLE);
            windowLong &= ~WS_SYSMENU;

            SetWindowLongPtr(hwnd, GWL_STYLE, new IntPtr(windowLong));
        }

        public const int WS_SYSMENU = 0x00080000;

        [DllImport("user32.dll", SetLastError = true)]
        public static extern int GetWindowLong(IntPtr hWnd, int nIndex);

        public const int GWL_STYLE = -16;

        public static IntPtr SetWindowLongPtr(IntPtr hWnd, int nIndex, IntPtr dwNewLong)
        {
            if (Environment.Is64BitProcess)
            {
                return SetWindowLongPtr64(hWnd, nIndex, dwNewLong);
            }

            return new IntPtr(SetWindowLong32(hWnd, nIndex, dwNewLong.ToInt32()));
        }

        [DllImport("user32.dll", EntryPoint = "SetWindowLong")]
        private static extern int SetWindowLong32(IntPtr hWnd, int nIndex, int dwNewLong);

        [DllImport("user32.dll", EntryPoint = "SetWindowLongPtr")]
        private static extern IntPtr SetWindowLongPtr64(IntPtr hWnd, int nIndex, IntPtr dwNewLong);

       
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9d7f8aa1c4aa3bcca4af2bed7cb65ec2e540ad47/HemluchaderJaberkaina) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
