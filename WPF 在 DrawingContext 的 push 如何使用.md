# WPF 在 DrawingContext 的 push 如何使用

本文告诉大家如何使用 DrawingContext 变换，修改画出的内容。

<!--more-->
<!-- CreateTime:2018/7/15 15:51:00 -->

<!-- csdn -->

如果在一个 DrawingContext 画出一个 DrawingVisual ，如何修改这个 DrawingVisual 的大小，对他进行变换？

简单的方法就是使用 PushTransform 方法，那么如何使用这个方法就是本文要告诉大家的。

先写一个简单的 OnRender ，创建一个类 GearcawralSarBule 继承 FrameworkElement 就可以重写 OnRender 方法，为了让WPF调用 OnRender 方法就需要把 GearcawralSarBule 加入视觉树，最简单加入视觉树的方法就是把他添加到 Grid，下面就是 GearcawralSarBule 类代码和在 xaml 添加他到 Grid 显示

```csharp
    public class GearcawralSarBule : FrameworkElement
    {
        /// <inheritdoc />
        protected override void OnRender(DrawingContext drawingContext)
        {
            base.OnRender(drawingContext);
        }
    }
```

```csharp
        <Grid HorizontalAlignment="Center" VerticalAlignment="Center">
            <local:GearcawralSarBule></local:GearcawralSarBule>
        </Grid>
```

现在运行可以看到界面没有内容，下面来使用一个已有的图片画出来。

```csharp
        public GearcawralSarBule()
        {
            DrawingVisual = new DrawingVisual();
            using (var drawingContext = DrawingVisual.RenderOpen())
            {
                // 本来是想写文字 lindexi 的，但是最后还是画星
                drawingContext.DrawGeometry(Brushes.Black, null, Geometry.Parse("m25,1 6,17h18l-14,11 5,17-15-10-15,10 5-17-14-11h18z"));
            }

            DrawingVisual.CacheMode = new BitmapCache();
        }

        private DrawingVisual DrawingVisual { get; set; }
```

上面代码使用 Geometry.Parse 转换一个图形，大家先猜一下是什么图形。

下面来把上面的 DrawingVisual 画出来

```csharp
        protected override void OnRender(DrawingContext drawingContext)
        {
            drawingContext.DrawDrawing(DrawingVisual.Drawing);
            base.OnRender(drawingContext);
        }
```
那么现在的问题是如何缩放这个画出来的 DrawingVisual ，实际上方法很简单，就是通过 drawingContext 的 push 方法。如果有玩过 ps 就知道，在 ps 有图层，使用 DrawingContext 的 push 方法就是创建一个图层，而且做的变换都是对这个图层做变换，在使用 push 创建图层之后需要使用 pop 把图层画进去。

如对 DrawingVisual 进行变换的代码

```csharp
        protected override void OnRender(DrawingContext drawingContext)
        {
            drawingContext.PushTransform(new ScaleTransform()
            {
                ScaleX = 2,
                ScaleY = 2,
            });
            drawingContext.DrawDrawing(DrawingVisual.Drawing);
            drawingContext.Pop();
            base.OnRender(drawingContext);
        }
```

这时就可以对 DrawingVisual 放大，因为 Transform 可以进行移动、旋转，这里的代码就不告诉大家了

注意使用了 push 需要在画完使用 pop ，不然会出现下面继续对 DrawingVisual 进行画的时候就会发现还是在原先的图层

除了 PushTransform 方法还有很多 push 方法，如 PushClip ，调用这个方法可以裁剪传入的范围。如 PushOpacity 可以设置接下来画的图片的不透明度，如果多次调用 PushOpacity 没有调用 Pop 就会叠加不透明度，如使用下面代码

```csharp
            drawingContext.PushOpacity(0.3);
            drawingContext.PushOpacity(0.3);


            drawingContext.DrawDrawing(DrawingVisual.Drawing);
```

和使用下面代码画出来的图形不透明度相同

```csharp
            drawingContext.PushOpacity(0.09);

            drawingContext.DrawDrawing(DrawingVisual.Drawing);
```

还有一个 PushGuidelineSet 参见：[WPF：基于物理像素的图形绘制 - Aaron Lu - 博客园](http://www.cnblogs.com/AaronLu/archive/2009/11/13/1602332.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
