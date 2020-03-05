# WPF 跨线程 UI 的方法

本文告诉大家如何在 WPF 使用多线程的 UI 的方法

在很多的时候都是使用单线程的 UI 但是有时候需要做到一个线程完全处理一个耗时的界面就需要将这个线程作为另一个 UI 线程

<!--more-->
<!-- CreateTime:2018/10/18 10:25:28 -->

<!-- csdn -->

在 WPF 可以使用 VisualTarget 做到多个 UI 线程的绘制，注意这里的 WPF 的渲染线程只有一个，多个 UI 线程无法让渲染的速度加快。如果一个界面有很多的 Visual 那么渲染速度也不会因为添加 UI 线程用的时间比原来少

在 WPF 的 VisualTarget 可以用来连接多个不同的线程的 UI 元素，在使用的时候只需要创建，然后在另一个 UI 线程将创建的元素添加到 RootVisual 就可以

```csharp
           var thread = new Thread(() =>
            {
                _visualTarget = new VisualTarget(xx);

                _visualTarget.RootVisual = 创建的 Visual;
            });
```

创建一个 VisualTarget 需要用到 HostVisual 通过 HostVisual 可以在多个线程连到视觉树，所以创建 HostVisual 需要在主线程

```csharp
public MainWindow()
{
	InitializeComponent();

    var hostVisual = new HostVisual();
          
            var thread = new Thread(() =>
            {
                _visualTarget = new VisualTarget(hostVisual);
                

                _visualTarget.RootVisual = 创建的 Visual;
            });
}
```

这时还需要将 hostVisual 加入视觉树，因为 HostVisual 也是 Visual 最简单将 Visual 加入视觉树的方法是创建一个类继承 UIElement 的方法，请看下面代码

```csharp
    public class DispatcherContainer : UIElement
    {
        /// <inheritdoc />
        protected override Visual GetVisualChild(int index)
        {
            return _hostVisual;
        }

        /// <inheritdoc />
        protected override int VisualChildrenCount => 1;

        private readonly HostVisual _hostVisual = new HostVisual();
    }
```

然后在构造函数添加一个线程用来创建另一个 UI 线程，创建一个 UI 线程的最简单方法是运行 `Dispatcher.Run()` 和设置线程 STA 才可以，注意这里的 `Dispatcher` 是静态类

```csharp
            var thread = new Thread(() =>
            {

                System.Windows.Threading.Dispatcher.Run();
            });

            thread.SetApartmentState(ApartmentState.STA);
            thread.Start();

```

在这个线程里添加 VisualTarget 请看下面

```csharp
            var thread = new Thread(() =>
            {
                _visualTarget = new VisualTarget(_hostVisual);

                _visualTarget.RootVisual = 创建的元素;

                System.Windows.Threading.Dispatcher.Run();
            });

            thread.SetApartmentState(ApartmentState.STA);
            thread.Start();

```

下面创建一个简单的元素在另一个线程

```csharp
            var thread = new Thread(() =>
            {
                _visualTarget = new VisualTarget(_hostVisual);
                DrawingVisual drawingVisual = new DrawingVisual();
                var drawing = drawingVisual.RenderOpen();
                using (drawing)
                {
                    var text = new FormattedText("欢迎访问我博客 http://lindexi.gitee.io 里面有大量 UWP WPF 博客",
                        CultureInfo.CurrentCulture, FlowDirection.LeftToRight,
                        new Typeface(new FontFamily("微软雅黑"), new FontStyle(), FontWeight.FromOpenTypeWeight(1),
                            FontStretch.FromOpenTypeStretch(1)), 20, Brushes.DarkSlateBlue);

                    drawing.DrawText(text, new Point(100, 100));
                }

                var containerVisual = new ContainerVisual();

                containerVisual.Children.Add(drawingVisual);

                _visualTarget.RootVisual = containerVisual;

                System.Windows.Threading.Dispatcher.Run();
            });

            thread.SetApartmentState(ApartmentState.STA);
            thread.Start();

```

这时的 DispatcherContainer 类看起来是这样

```csharp
    public class DispatcherContainer : UIElement
    {
        /// <inheritdoc />
        public DispatcherContainer()
        {
            var thread = new Thread(() =>
            {
                _visualTarget = new VisualTarget(_hostVisual);
                DrawingVisual drawingVisual = new DrawingVisual();
                var drawing = drawingVisual.RenderOpen();
                using (drawing)
                {
                    var text = new FormattedText("欢迎访问我博客 http://lindexi.gitee.io 里面有大量 UWP WPF 博客",
                        CultureInfo.CurrentCulture, FlowDirection.LeftToRight,
                        new Typeface(new FontFamily("微软雅黑"), new FontStyle(), FontWeight.FromOpenTypeWeight(1),
                            FontStretch.FromOpenTypeStretch(1)), 20, Brushes.DarkSlateBlue);

                    drawing.DrawText(text, new Point(100, 100));
                }

                var containerVisual = new ContainerVisual();

                containerVisual.Children.Add(drawingVisual);

                _visualTarget.RootVisual = containerVisual;

                System.Windows.Threading.Dispatcher.Run();
            });

            thread.SetApartmentState(ApartmentState.STA);
            thread.Start();
        }

        /// <inheritdoc />
        protected override Visual GetVisualChild(int index)
        {
            return _hostVisual;
        }

        /// <inheritdoc />
        protected override int VisualChildrenCount => 1;

        private readonly HostVisual _hostVisual = new HostVisual();
        private VisualTarget _visualTarget;
    }

```

为了显示元素，需要添加到界面，打开界面添加下面代码

```csharp
        <local:DispatcherContainer></local:DispatcherContainer>
```

运行可以看到下面界面，这里的文字是在另一个线程绘制，但是也是和主界面在相同的线程渲染

<!-- ![](image/WPF 跨线程 UI 的方法/WPF 跨线程 UI 的方法0.png) -->

![](http://image.acmx.xyz/lindexi%2F2018101893923600)

代码请看 https://github.com/lindexi/UWP/tree/master/wpf/CaitrairSodeyatarFowfurur

更多博客请看 [WPF 同一窗口内的多线程 UI（VisualTarget） - walterlv](https://walterlv.com/post/multi-thread-ui-using-visualtarget-in-wpf.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
