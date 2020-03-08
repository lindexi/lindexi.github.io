# WPF 一个空的 WPF 程序有多少个窗口

好多小伙伴说 WPF 的程序有五个窗口，但是我尝试使用了 EnumThreadWindows 去获取的时候居然拿到了 10 多个窗口

<!--more-->
<!-- CreateTime:2018/12/18 21:16:40 -->

<!-- csdn -->

在 [WPF 内部的5个窗口之 MediaContextNotificationWindow](https://lindexi.gitee.io/post/WPF-%E5%86%85%E9%83%A8%E7%9A%845%E4%B8%AA%E7%AA%97%E5%8F%A3%E4%B9%8B-MediaContextNotificationWindow.html ) 听说有五个窗口

可以通过 user32 的 EnumThreadWindows 找到一个线程的窗口

```csharp
        delegate bool EnumThreadDelegate(IntPtr hWnd, IntPtr lParam);

        [DllImport("user32.dll")]
        static extern bool EnumThreadWindows(int dwThreadId, EnumThreadDelegate lpfn,
            IntPtr lParam);
```

获取线程的 id 的方法需要先获取进程，在 Loaded 之后尝试获取 WPF 的进程，通过 `Process.GetCurrentProcess()` 可以拿到当前的进程

通过 process.Threads 可以拿到进程的线程，封装为一个方法

```csharp
        delegate bool EnumThreadDelegate(IntPtr hWnd, IntPtr lParam);

        [DllImport("user32.dll")]
        static extern bool EnumThreadWindows(int dwThreadId, EnumThreadDelegate lpfn,
            IntPtr lParam);

        static IEnumerable<IntPtr> EnumerateProcessWindowHandles(Process process)
        {
            var handleList = new List<IntPtr>();

            foreach (ProcessThread thread in process.Threads)
            {
                EnumThreadWindows(thread.Id,
                    (hWnd, lParam) => { handleList.Add(hWnd); return true; }, IntPtr.Zero);
            }

            return handleList;
        }
```

调用 EnumerateProcessWindowHandles 输出进程就可以拿到这个进程内的所有窗口，于是输入当前的 WPF 的进程，获取一下

```csharp
        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var handleList = EnumerateProcessWindowHandles(Process.GetCurrentProcess());
            Debug.WriteLine(handleList.Count());
        }
```

返回的是 14 个窗口，但是如果将代码移动到 WPF 的构造函数，会发现只有两个窗口

```csharp
        public MainWindow()
        {
            var handleList = EnumerateProcessWindowHandles(Process.GetCurrentProcess());
            Debug.WriteLine(handleList.Count());

            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }
```



[WPF 内部的5个窗口之 MediaContextNotificationWindow](https://lindexi.gitee.io/post/WPF-%E5%86%85%E9%83%A8%E7%9A%845%E4%B8%AA%E7%AA%97%E5%8F%A3%E4%B9%8B-MediaContextNotificationWindow.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
