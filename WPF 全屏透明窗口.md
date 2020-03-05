# WPF 全屏透明窗口

本文告诉大家如何在 WPF 做一个全屏的透明窗口，和全屏透明窗口的坑

<!--more-->
<!-- CreateTime:2019/11/27 9:22:19 -->

<!-- csdn -->

使用下面代码就可以作出全屏透明窗口

```csharp
        public MainWindow()
        {
            InitializeComponent();

            AllowsTransparency = true;
            WindowStyle = WindowStyle.None;
            WindowState = WindowState.Maximized;
            Background = Brushes.Transparent;
            Topmost = true;
        }
```

在 WPF 透明窗口会加上背景放到内存，尝试将上面程序在 2K 屏幕运行，将会看到大概需要 70M 内存。如果屏幕分辨率更大，将会发现鼠标移动很慢，请看 [【翻译】关于 WPF 透明窗口的内存占用](https://gandalfliang.github.io/2018/02/16/transparent_4k_window.translate/ )

另外会发现窗口其实和屏幕有一个像素，通过设置 `ResizeMode="NoResize"` 可以解决，因为这一个像素是用来拖动窗口

此外，设置 `ResizeMode="NoResize"` 可以解决：

- [17025 触摸bug](https://blog.lindexi.com/post/win10-17025-%E8%A7%A6%E6%91%B8bug.html )
- 还原窗口再最大化，窗口出现偏移
- 切换屏幕，窗口出现偏移

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
