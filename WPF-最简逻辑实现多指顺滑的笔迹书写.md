
# WPF 最简逻辑实现多指顺滑的笔迹书写

只需不到 150 行代码就能实现一个支持多指顺滑的笔迹书写的应用。当然，这个应用除了笔迹书写外，没有其他任何功能。本文将不会使用 InkCanvas 而是使用更底的方法，通过 Stroke 进行绘制

<!--more-->


<!-- CreateTime:2020/8/20 9:46:18 -->


这是我在写测试应用的时候，我想要了解我能用多少行代码实现一个多指顺滑的笔迹书写的核心逻辑。其实在 WPF 下，可以通过 Stroke 类的辅助，不断给 Stroke 添加点的方式，做到绘制出笔迹

绘制笔迹需要给定一个 DrawingAttributes 告诉笔迹的粗细和颜色等

其次需要创建 Stroke 类，在这个类的 StylusPoints 数组里面不断添加点，此时添加的点将会被加入到笔迹里面。在 WPF 的笔迹实际上算法就是将离散的点连接作为一段顺滑的笔迹

那么如何在界面显示出来？在 Stroke 类提供了 Draw 方法，可以绘制到 DrawingContext 里面

根据上面这些内容，咱写一个 StrokeVisual 继承 DrawingVisual 类

```csharp
    /// <summary>
    ///     用于显示笔迹的类
    /// </summary>
    public class StrokeVisual : DrawingVisual
    {

    }
```

第一步就是拿到 DrawingAttributes 的值，可以使用如下代码

```csharp
        /// <summary>
        ///     创建显示笔迹的类
        /// </summary>
        public StrokeVisual() : this(new DrawingAttributes()
        {
            Color = Colors.Red,
            FitToCurve = true,
            Width = 5
        })
        {
        }

        /// <summary>
        ///     创建显示笔迹的类
        /// </summary>
        /// <param name="drawingAttributes"></param>
        public StrokeVisual(DrawingAttributes drawingAttributes)
        {
            _drawingAttributes = drawingAttributes;
        }

        private readonly DrawingAttributes _drawingAttributes;
```

第二步就是实现不断添加点的功能

```csharp
        /// <summary>
        ///     设置或获取显示的笔迹
        /// </summary>
        public Stroke Stroke { set; get; }

        /// <summary>
        ///     在笔迹中添加点
        /// </summary>
        /// <param name="point"></param>
        public void Add(StylusPoint point)
        {
            if (Stroke == null)
            {
                var collection = new StylusPointCollection {point};
                Stroke = new Stroke(collection) {DrawingAttributes = _drawingAttributes};
            }
            else
            {
                Stroke.StylusPoints.Add(point);
            }
        }
```

最后一步是让 Stroke 回执到 DrawingContext 里面。在 StrokeVisual 类，是继承 DrawingVisual 的，所以可以通过调用 RenderOpen 的方法实现

```csharp
        /// <summary>
        ///     重新画出笔迹
        /// </summary>
        public void Redraw()
        {
            using var dc = RenderOpen();
            Stroke.Draw(dc);
        }
```

在拿到一个 Visual 类，也就是 StrokeVisual 可以如何在 WPF 中显示？最简单的方法是加一个自定义的类继承 FrameworkElement 来做，当然，在我自己的工具库里面是有默认实现的，请看代码

```csharp
    public class VisualCanvas : FrameworkElement
    {
        protected override Visual GetVisualChild(int index)
        {
            return Visual;
        }

        protected override int VisualChildrenCount => 1;

        public VisualCanvas(DrawingVisual visual)
        {
            Visual = visual;
            AddVisualChild(visual);
        }

        public DrawingVisual Visual { get; }
    }
```

上面代码需要注意的有一点就是需要添加视觉树，通过 AddVisualChild 方法，否则加入的控件将只会被渲染一次。敲黑板，不在视觉树上的元素将不会持续渲染

接下来就是实现多指了，实现方式是通过 StylusMove 和 StylusUp 事件实现。每一个手指将会对应一个 StrokeVisual 类，因此 StrokeVisual 类只包含一条笔迹

通过 `e.StylusDevice.Id` 可以区分当前触摸的是哪个手指，通过写一个字典就能快速做到分开多个触摸

```csharp
        private Dictionary<int, StrokeVisual> StrokeVisualList { get; } = new Dictionary<int, StrokeVisual>();
```

添加一个辅助方法，通过输入的 Id 返回一个 StrokeVisual 类，如果输入的 Id 不存在，也就是这是第一个按下，此时创建一个新的，同时加入到界面

```csharp
        private StrokeVisual GetStrokeVisual(int id)
        {
            if (StrokeVisualList.TryGetValue(id, out var visual))
            {
                return visual;
            }

            var strokeVisual = new StrokeVisual();
            StrokeVisualList[id] = strokeVisual;
            var visualCanvas = new VisualCanvas(strokeVisual);
            Grid.Children.Add(visualCanvas);

            return strokeVisual;
        }
```

接下来就是在 StylusMove 的事件，拿到触摸点，传入到 StrokeVisual 类

```csharp
        private void MainWindow_StylusMove(object sender, StylusEventArgs e)
        {
            var strokeVisual = GetStrokeVisual(e.StylusDevice.Id);
            var stylusPointCollection = e.GetStylusPoints(this);
            foreach (var stylusPoint in stylusPointCollection)
            {
                strokeVisual.Add(new StylusPoint(stylusPoint.X, stylusPoint.Y));
            }

            strokeVisual.Redraw();
        }
```

为什么使用 Stylus 事件，而不是 Touch 事件？原因有两个，第一个是 Stylus 是触笔，也就是触摸和笔都会进入。第二个是通过 GetStylusPoints 可以拿到密集的点集，此时绘制才能做到顺滑。那么为什么 GetStylusPoints 可以获取比 WM_Touch 更密集的点？原因是 GetStylusPoints 是通过 RealTime Stylus 实时触摸获取的点

最后一步就是在手指抬起的时候，删除字典的对应的值。因此触摸的 Id 是在相同时刻是不同的，但是取值只有0-255也就是最多画 255 画之后，将会存在至少一次 Id 的重复

```csharp
        private void MainWindow_StylusUp(object sender, StylusEventArgs e)
        {
            StrokeVisualList.Remove(e.StylusDevice.Id);
        }
```

这样就实现了一个简单的多指顺滑的笔迹书写，但这不是一个高性能的书写方案。有啥可以做到虐次方案的性能的？有两个点，一个是输入一个是输出。这里的输入就是接收触摸，而输出就是渲染

拿到触摸最快的方法是通过 [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 的 [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html ) 方法拿到触摸点，简单的代码请看 [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )

而渲染部分，请看 [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)

渲染相对复杂，最简单的就是不要让 Stroke 包含太多的点，如果包含很多点，那么分为多个不同的 Stroke 对象，这样每次渲染的内容都不会很多，渲染性能相对比较高

本文的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/4317c4c015365dc65284124aa38fca52c888b70e/KemjawyecawDurbahelal) 欢迎小伙伴访问

全部代码如下

```csharp
    public class MultiTouchInkCanvas : Grid
    {
        public MultiTouchInkCanvas()
        {
            // 只是为了命中测试，设置背景是透明，这样就能收到输入
            Background = Brushes.Transparent;

            HorizontalAlignment = HorizontalAlignment.Stretch;
            VerticalAlignment = VerticalAlignment.Stretch;

            // 使用 StylusMove 事件的速度将会比较慢
            StylusMove += MultiTouchInkCanvas_StylusMove;
            StylusUp += MultiTouchInkCanvas_StylusUp;
        }

        private void MultiTouchInkCanvas_StylusUp(object sender, StylusEventArgs e)
        {
            StrokeVisualList.Remove(e.StylusDevice.Id);
        }

        private void MultiTouchInkCanvas_StylusMove(object sender, StylusEventArgs e)
        {
            var strokeVisual = GetStrokeVisual(e.StylusDevice.Id);
            var stylusPointCollection = e.GetStylusPoints(this);
            foreach (var stylusPoint in stylusPointCollection)
            {
                strokeVisual.Add(new StylusPoint(stylusPoint.X, stylusPoint.Y));
            }

            strokeVisual.Redraw();
        }

        // 其实不使用 Grid 而使用自己定制的 Panel 的性能能更好，但是这里只是给例子而已
        public Grid Grid => this;

        // 如果后续性能优化，使用触摸线程拿到输入，那么记得鼠标和触摸是两个不同线程，不能使用字典
        private Dictionary<int, StrokeVisual> StrokeVisualList { get; } = new Dictionary<int, StrokeVisual>();

        private StrokeVisual GetStrokeVisual(int id)
        {
            if (StrokeVisualList.TryGetValue(id, out var visual))
            {
                return visual;
            }

            var strokeVisual = new StrokeVisual();
            StrokeVisualList[id] = strokeVisual;
            var visualCanvas = new VisualCanvas(strokeVisual);
            Grid.Children.Add(visualCanvas);

            return strokeVisual;
        }
    }


    /// <summary>
    ///     用于显示笔迹的类
    /// </summary>
    public class StrokeVisual : DrawingVisual
    {
        /// <summary>
        ///     创建显示笔迹的类
        /// </summary>
        public StrokeVisual() : this(new DrawingAttributes()
        {
            Color = Colors.Red,
            FitToCurve = true,
            Width = 5
        })
        {
        }

        /// <summary>
        ///     创建显示笔迹的类
        /// </summary>
        /// <param name="drawingAttributes"></param>
        public StrokeVisual(DrawingAttributes drawingAttributes)
        {
            _drawingAttributes = drawingAttributes;
        }

        private readonly DrawingAttributes _drawingAttributes;

        /// <summary>
        ///     设置或获取显示的笔迹
        /// </summary>
        public Stroke Stroke { set; get; }

        /// <summary>
        ///     在笔迹中添加点
        /// </summary>
        /// <param name="point"></param>
        public void Add(StylusPoint point)
        {
            if (Stroke == null)
            {
                var collection = new StylusPointCollection {point};
                Stroke = new Stroke(collection) {DrawingAttributes = _drawingAttributes};
            }
            else
            {
                Stroke.StylusPoints.Add(point);
            }
        }

        /// <summary>
        ///     重新画出笔迹
        /// </summary>
        public void Redraw()
        {
            using var dc = RenderOpen();
            Stroke.Draw(dc);
        }
    }

    public class VisualCanvas : FrameworkElement
    {
        protected override Visual GetVisualChild(int index)
        {
            return Visual;
        }

        protected override int VisualChildrenCount => 1;

        public VisualCanvas(DrawingVisual visual)
        {
            Visual = visual;
            AddVisualChild(visual);
        }

        public DrawingVisual Visual { get; }
    }
```

但是无论如何做，都没有 UWP 的快。除非在 WPF 中上 Composition API [使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html ) 再加上 [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )的方法，使用 [win2d 画出笔迹](https://blog.lindexi.com/post/win10-uwp-%E9%80%9A%E8%BF%87-win2d-%E7%94%BB%E5%87%BA%E7%AC%94%E8%BF%B9.html ) 和 [win2d CanvasVirtualControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html ) 存放绘制的笔迹

更多笔迹相关请看

- [WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html )
- [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)
- [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 
- [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )
- [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )
- [WPF 使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html )
- [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )
- [win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。