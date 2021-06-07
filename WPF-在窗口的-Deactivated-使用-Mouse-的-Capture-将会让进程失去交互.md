
# WPF 在窗口的 Deactivated 使用 Mouse 的 Capture 将会让进程失去交互

如果在某个窗口的 Deactivated 事件里面，使用 Mouse.Capture 方法，让这个窗口重新捕获鼠标，那么将会让进程的所有窗口都失去鼠标交互，点击无效，只有在切换到其他进程的窗口之后，才能让窗口继续交互

<!--more-->



<!-- 发布 -->

实现这个坑的逻辑很简单，假定有两个窗口，分别是 MainWindow 和 Window1 两个窗口，在 Window1 的 Deactivated 事件里面，使用 Mouse.Capture 方法，让这个窗口重新捕获鼠标

```csharp
        public Window1()
        {
            InitializeComponent();

            Deactivated += Window1_Deactivated;
        }

        private void Window1_Deactivated(object sender, EventArgs e)
        {
            Mouse.Capture(this);
        }
```

先点击 Window1 激活，然后点击 MainWindow 的空白，切换到 MainWindow 窗口。接着点击任何的窗口或窗口的按钮等，都没有响应

在 WPF 发现鼠标点击失效，或者触摸失效等时，可以全局搜一下 Mouse.Capture 方法，看是否在窗口的 Deactivated 事件里面调用

本来还想聊聊为什么这样做就会失去鼠标焦点的，然而 10 点多了





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。