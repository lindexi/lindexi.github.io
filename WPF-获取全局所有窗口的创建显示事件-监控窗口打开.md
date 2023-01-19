
# WPF 获取全局所有窗口的创建显示事件 监控窗口打开

本文将告诉大家如何在 WPF 里面进行全局监控任意的窗口创建显示打开，可以获取到每个 WPF 窗口的打开的时机。如此可以用来辅助定位问题和输出日志

<!--more-->



<!-- 博客 -->
<!-- 发布 -->

这篇博客是有背景的，老司机告诉我说他的应用不响应鼠标和键盘点击了，于是我和他调查了半天才发现有一个 Dialog 窗口显示出来，导致消息循环被切到了一个 Dialog 窗口里面去了。然而这个 Dialog 窗口是藏起来的，在屏幕上刚好没有能看见他

此时我就好奇了，为什么我的应用就不会存在如此的逗比代码，但是老司机的应用就可能存在如此难以调试的问题。于是我就在自己的应用上写了一个逗比代码，强行弹出一个 Dialog 窗口出来，结果我就发现监控模块告诉了我有一个在白名单之外的窗口弹出了，如果确定这是符合开发预期的，那就需要手动修改白名单。这是一个开发时的辅助机制，用来让开发者不要随便弹出窗口，我又好奇这个监控模块是如何监控到我弹出一个窗口的，学习了监控模块的机制，就写了这个博客

在 WPF 里面，可以通过 EventManager 监听全局的路由事件，刚好窗口创建显示时，将会触发一些路径事件。通过路由事件监听，即可了解是哪个窗口正在准备弹出。于是就可以进行监控窗口创建显示

监听窗口的 SizeChangedEvent 路由事件是比较靠谱的方式，这个有一点点违反开发者的想法，开发者默认想的是使用 LoadedEvent 事件。但是在 WPF 里面做了一些性能优化，如果一个窗口没有 XAML 或者是没有任何代码监听了 Loaded 事件，那将不触发 LoadedEvent 路由事件。因此这里就监听窗口一定会触发的 SizeChangedEvent 事件

代码如下

```csharp
        public App()
        {
            EventManager.RegisterClassHandler(typeof(Window), Window.LoadedEvent, new RoutedEventHandler(Window_OnLoaded));

            EventManager.RegisterClassHandler(typeof(Window), Window.SizeChangedEvent, new RoutedEventHandler(Window_SizeChanged));
        }

        private void Window_OnLoaded(object sender, RoutedEventArgs e)
        {
            // 如果窗口没有 XAML 或者没有监听 Loaded 事件，将不会被触发
        }

        private void Window_SizeChanged(object sender, RoutedEventArgs e)
        {
            // 所有窗口都会触发
        }
```

窗口创建的时候，将会进入 `Window_SizeChanged` 事件。通过 `sender` 参数即可了解是哪个窗口对象被创建，接着就可以方便进行日志或者是提示开发者不要随便创建窗口等

可以自己测试一下代码，例如在 MainWindow 里面再次创建一个空窗口

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            Loaded += MainWindow_Loaded;
        }

        private void MainWindow_Loaded(object sender, RoutedEventArgs e)
        {
            var window = new Window();
            window.Show();
        }
    }
```

运行以上的代码，可以看到 `Window_SizeChanged` 被进入两次，分别是 MainWindow 和在 MainWindow 里面创建显示的窗口。而 `Window_OnLoaded` 只会进入一次，在 MainWindow 准备显示时进入

我推荐在自己的项目里面，在 Debug 下加上此机制，至少输出一下窗口显示的日志，方便了解显示了哪些窗口。如果可以的话，也可以加上白名单机制，如果一个窗口是新写的之类的，可以提示开发者阅读必要的文档之后加入到白名单里面。也可以在事件里面对每个窗口注入一些有趣的逻辑，或者是监听窗口的各个事件，输出更多日志，让开发者可以通过日志了解到当前有哪些窗口依然还在显示

这是另一位大佬写的代码，请看 [https://gist.github.com/mwisnicki/3104963](https://gist.github.com/mwisnicki/3104963)




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。