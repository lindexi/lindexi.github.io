# WPF 获取应用的所有窗口

本文告诉大家如何获取应用内的所有窗口，无论这些窗口有没显示

<!--more-->
<!-- CreateTime:2019/2/11 8:55:31 -->

<!-- csdn -->

在 WPF 可以通过 Application.Current.Windows 列举应用的所有窗口

```csharp
foreach(Window window in Application.Current.Windows ) 
{
    Console.WriteLine(window.Title);
}
```

如果需要获取一个线程的窗口，请看代码

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

[WPF 一个空的 WPF 程序有多少个窗口](https://lindexi.gitee.io/post/WPF-%E4%B8%80%E4%B8%AA%E7%A9%BA%E7%9A%84-WPF-%E7%A8%8B%E5%BA%8F%E6%9C%89%E5%A4%9A%E5%B0%91%E4%B8%AA%E7%AA%97%E5%8F%A3.html )

[WPF 内部的5个窗口之 MediaContextNotificationWindow ](https://lindexi.gitee.io/post/WPF-%E5%86%85%E9%83%A8%E7%9A%845%E4%B8%AA%E7%AA%97%E5%8F%A3%E4%B9%8B-MediaContextNotificationWindow.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
