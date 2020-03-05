# WPF 在 Alt+Tab 隐藏窗口

最近在开发一个 Toast 窗口，因为这个窗口不能在显示之后关闭，因为可能用户会不停让窗口显示，所以只能 Hide 。但是这样会在 切换窗口看到这个窗口，所以我找到了一个方法来让 WPF 窗口不在切换窗口显示。

<!--more-->
<!-- CreateTime:2018/4/15 10:13:40 -->

<!-- csdn -->

现在的 WPF 程序只要设置了不在任务栏显示，而且设置窗口`Visibility="Hidden"`就可以不在切换窗口显示窗口。设置方法可以是在 xaml 添加下面代码

```csharp
ShowInTaskbar="False" Visibility="Hidden"
```

但是如大家见到，如果存在 BitmapCache 和 一个隐藏的窗口，那么就会在锁屏之后软件无法渲染，请看[github](https://github.com/dotnet-campus/wpf-issues/tree/master/BitmapCache) ，所以不要使用这个方法。那么除了这个方法外还有什么方法？

实际上在切换窗口不显示窗口要求窗口是：`WS_EX_TOOLWINDOW` 或其他窗口的子窗口，但是可以看到 Toast 不是其他窗口的子窗口，所以只能设置窗口。

因为只要设置窗口是`WS_EX_TOOLWINDOW`就不会在切换窗口显示，所以需要使用一些特殊的代码。

首先在窗口的 Load 之后拿到窗口句柄，注意不是在 SourceInitialized 之后添加的

```csharp
        public ToastWindow()
        {
            InitializeComponent();

            Loaded += ToastWindow_Loaded;
        }
```

然后在 Load 里面使用隐藏窗口的代码

```csharp
        private void HideAltTab()
        {
            var windowInterop = new WindowInteropHelper(this);
            var exStyle = GetWindowLong(windowInterop.Handle, GWL_EXSTYLE);
            exStyle |= WS_EX_TOOLWINDOW;
            Win32.SetWindowLong(windowInterop.Handle, GWL_EXSTYLE, exStyle);
        }
```

如果你直接复制上面的代码是无法运行的，因为需要写几个函数

第一个函数是  ExtendedWindowStyles 请看下面，实际使用 WS_EX_TOOLWINDOW

<script src="https://gist.github.com/lindexi/21e4e640d53b3dcac3e6a6c69fc09db8.js"></script>

如果看不到上面的代码，请看[ExtendedWindowStyles code from msdn](https://gist.github.com/lindexi/21e4e640d53b3dcac3e6a6c69fc09db8 )

```csharp
#region Window styles

    public enum GetWindowLongFields
    {
        // ...
        GWL_EXSTYLE = (-20),
        // ...
    }

    [DllImport("user32.dll")]
    public static extern IntPtr GetWindowLong(IntPtr hWnd, int nIndex);

    public static IntPtr SetWindowLong(IntPtr hWnd, int nIndex, IntPtr dwNewLong)
    {
        int error = 0;
        IntPtr result = IntPtr.Zero;
        // Win32 SetWindowLong doesn't clear error on success
        SetLastError(0);

        if (IntPtr.Size == 4)
        {
            // use SetWindowLong
            Int32 tempResult = IntSetWindowLong(hWnd, nIndex, IntPtrToInt32(dwNewLong));
            error = Marshal.GetLastWin32Error();
            result = new IntPtr(tempResult);
        }
        else
        {
            // use SetWindowLongPtr
            result = IntSetWindowLongPtr(hWnd, nIndex, dwNewLong);
            error = Marshal.GetLastWin32Error();
        }

        if ((result == IntPtr.Zero) && (error != 0))
        {
            throw new System.ComponentModel.Win32Exception(error);
        }

        return result;
    }

    [DllImport("user32.dll", EntryPoint = "SetWindowLongPtr", SetLastError = true)]
    private static extern IntPtr IntSetWindowLongPtr(IntPtr hWnd, int nIndex, IntPtr dwNewLong);

    [DllImport("user32.dll", EntryPoint = "SetWindowLong", SetLastError = true)]
    private static extern Int32 IntSetWindowLong(IntPtr hWnd, int nIndex, Int32 dwNewLong);

    private static int IntPtrToInt32(IntPtr intPtr)
    {
        return unchecked((int)intPtr.ToInt64());
    }

    [DllImport("kernel32.dll", EntryPoint = "SetLastError")]
    public static extern void SetLastError(int dwErrorCode);
    #endregion
```

参见：https://stackoverflow.com/a/551847/6116637

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
