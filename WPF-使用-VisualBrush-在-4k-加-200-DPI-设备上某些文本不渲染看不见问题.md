
# WPF 使用 VisualBrush 在 4k 加 200 DPI 设备上某些文本不渲染看不见问题

这是我做一个十万点实时刷新的图表控件遇到的问题，做过高性能图表的伙伴大概都知道，此时需要关闭命中测试的功能，无论是控件的还是 Drawing 的，否则计算命中测试的耗时将会让主线程卡住。为了解决此问题，有多个可以选择的方法，在此控件，我选择的是采用 VisualBrush 的方法。将 DrawingVisual 绘制到 VisualBrush 里面，再将 VisualBrush 作为贴图给矩形使用，这样的优势在于可以在命中测试的时候，只处理矩形。矩形命中测试的耗时可以忽略。但是在一些 4k 加百分之 200 的 DPI 缩放设备上，看不到某些 GlyphRun 的内容，本文记录此问题和对应的解决方法

<!--more-->


<!-- CreateTime:2021/11/8 20:18:54 -->


<!-- 发布 -->

前置要求：

- 4k 分辨率屏幕
- 百分之两百 DPI 缩放
- 使用 GlyphRun 直接或间接
- 绘制到 VisualBrush 中

在 WPF 的底层文本绘制都是采用 GlyphRun 绘制，因此可以认定为影响为全部文本，以及对应的文本控件

现象：

有某些文本内容不绘制渲染出来，看不见某些文本内容，但是在相同的 DrawingContext 里面的其他绘制内容，如线条或图片等都可以正常绘制出来

以上的现象包括：

- 在某些设备上，暂时未找到具体影响因素
- 某些文本内容不可见，而不是全部文本内容
- 对整个控件进行 RenderTransform 之后可以让某些文本可见
- 对界面进行刷新，可以让文本可见
- 对界面进行偶数次刷新，文本不可见

开始之前先回答一下为什么会在图表控件里面，将 DrawingContext 的内容放入到 VisualBrush 中。如上文所述，这是因为 DrawingContext 对象是从 DrawingVisual 里面获取的，而 DrawingVisual 的 RenderOpen 返回的是一个带 RenderData 收集器的 DrawingContext 对象，也就是说此对象还远远不是最终被执行 DirectX 渲染的对象，仅仅是收集绘制内容，放入到 RenderData 里面。后续还有在执行默认命中测试的时候，取 RenderData 里面的内容进行计算渲染边距以及命中测试。总之，如果将 DrawingVisual 加入到视觉树里面，那么将会因为存在命中测试等逻辑导致需要执行很多逻辑而降低性能

为了提升性能，提升性能的其中一个方法是减少 CPU 工作量，也就是减少计算逻辑量。此时将 DrawingVisual 放入到 VisualBrush 中，作为 Brush 给一个矩形做填充，这样的优势在于进行命中测试的时候，默认是无视图层的，只会对矩形进行命中测试。刚好矩形命中测试的耗时是基本可以被忽略的，因此也就能极大提升了性能

需要说明的是，默认是可以无视命中测试给 DrawingVisual 带来的性能损耗，因为计算速度还是非常快的。但是在图表控件里面，架不住点的数量很多，尽管命中测试性能足够高，然而点的数量足够多也可以拖住性能

如下是将 DrawingVisual 绘制到 VisualBrush 上，再将 VisualBrush 贴到矩形上的方法，也就是我的图表控件的核心绘制逻辑

```csharp
        private DrawingVisual CreateVisual()
        {
            var dv = new DrawingVisual();
            _visualBase = dv;
            var drawingContext = dv.RenderOpen();

            // 绘制点
            DrawPoints(drawingContext);

            // 绘制线
            DrawLine(drawingContext);

            // 绘制文本
            DrawGlyphRun(drawingContext);

            drawingContext.Close();

            var ret = new DrawingVisual();

            using (var dw = ret.RenderOpen())
            {
                var visualBrush = new VisualBrush(dv)
                {
                    Stretch = Stretch.None,
                };

                dw.DrawRectangle(visualBrush, null, dv.ContentBounds);
                _visualBrush = visualBrush;
            }

            return ret;

            //return dv;
        }
```

将绘制点和绘制线的 DrawingVisual 也就是上文的 dv 创建出来 drawingContext 用来做实际的图表内容绘制收集。而将 dv 作为 VisualBrush 的输入，接着新建一个叫 ret 的 DrawingVisual 对象，在这里面重新绘制出矩形然后用 VisualBrush 做贴图

这样做的优势在于可以利用到 WPF 无视贴图的命中测试的特性，而提升性能

但是带来的问题就是存在某些 GlyphRun 的文本不绘制，在相同的 drawingContext 绘制的点和线是可见的，只有文本看不到

其中最优解决方法是干掉 VisualBrush 而是换成 DrawingBrush 作为贴图，更改之后代码如下

```csharp
        private DrawingVisual CreateTextVisual()
        {
        	// var dv = new DrawingVisual();
            var dv = new DrawingGroup();
            _visualBase = dv;
            var drawingContext = dv.Open();

            // 绘制点
            DrawPoints(drawingContext);

            // 绘制线
            DrawLine(drawingContext);

            // 绘制文本
            DrawGlyphRun(drawingContext);

            drawingContext.Close();

            var ret = new DrawingVisual();

            using (var dw = ret.RenderOpen())
            {
            	// var visualBrush = new VisualBrush(dv)
                 _drawingBrush = new DrawingBrush(dv);

                dw.DrawRectangle(_drawingBrush, null, dv.Bounds);
            }

            return ret;

            //return dv;
        }
```

如上面代码，将 dv 的类型从 DrawingVisual 换成 DrawingGroup 类型，将后续的贴图从 VisualBrush 换成 DrawingBrush 类型。这样就能修复某些文本不显示的问题

为什么 VisualBrush 会让某些文本不更新脏就不显示？和 VisualBrush 的机制有关，在 VisualBrush 里面，要求先将内容渲染为 Bitmap 位图再作为某个元素的贴图层，执行顺序上需要有些复杂。而为什么如此复杂的逻辑会挖坑？表示我追踪了代码也没有发现更本质的问题，而且此问题只有在我的此图表控件才有偶尔复现，在能复现的设备上，每次都能用相同的图表数据进行复现。在能复现的设备上，如果变更了图表的内容，也许就又不复现了

如果将我的图表控件放在 demo 上跑，那也不会有啥锅。我也不知道是不是我的应用层挖的坑。因为我的应用层也充满了各个逗比加诡异的逻辑，因此我也不好说是不是某个有趣的逻辑的锅。此问题只有在使用特定的图表内容（很复杂）再加上放入到我的某个特定的应用里面才能复现，要调试 WPF 层的话，必须加入到我的应用层才能开始调试此问题。因此预计我也不会继续往底层调试，告诉大家具体的原因





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。