# WPF 如何画出1像素的线

如何有人告诉你，请你画出1像素的线，是不是觉得很简单，实际上在 WPF 上还是比较难的。

本文告诉大家，如何让画出的线不模糊

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->


画出线的第一个方法，创建一个 Canvas ，添加一个线

界面代码

```csharp
            <Canvas x:Name="Canvas"></Canvas>

```

在后台添加一条线

```csharp
            Line myLine = new Line();

            myLine.Stroke = System.Windows.Media.Brushes.Black;

            myLine.X1 = 100;
            myLine.X2 = 200;  // 150 too far
            myLine.Y1 = 200;
            myLine.Y2 = 200;

            myLine.StrokeThickness = 1;

            Canvas.Children.Add(myLine);
```

那么如何看到线模糊呢？

简单方法是使用 ViewBox 和放大镜，可以看到模糊

在界面添加下面代码

```csharp
    <Viewbox >
            <Canvas x:Name="Canvas"></Canvas>
    </Viewbox>
```

这时拖动窗口可以看到线放大

可以看到线是模糊的，如果想要让线不模糊，可以添加下面的代码

```csharp
        myLine.SnapsToDevicePixels = true;
            myLine.SetValue(RenderOptions.EdgeModeProperty, EdgeMode.Aliased);
```

这个方法是从 [https://stackoverflow.com/q/2879033/6116637][https://stackoverflow.com/q/2879033/6116637]得到，但是无法对于自己的控件

如果自己创建一个控件，那么直接使用 dc.DrawLine 得到不是清晰的

创建一个类自定义控件，添加下面的代码画出线

```csharp
        protected override void OnRender(DrawingContext dc)
        {
           
                dc.DrawLine(_pen,  new Point(10, 10), new Point(310, 10));
            
        }
```

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017720205318.jpg)

可以看到，画出来的线是模糊的，于是看了微软的代码

看了他的矩形是如何画的，看到他画出来的是清晰的，但是复制他的代码到我的控件，画出来不是清晰的

```csharp
        /// <summary>
        /// Render callback.
        /// </summary>
        protected override void OnRender(DrawingContext drawingContext)
        {
            Pen pen = GetPen();
            drawingContext.DrawRoundedRectangle(Fill, pen, _rect, RadiusX, RadiusY);
        }
```

下面代码是我复制他的，但是自己的控件画出来在放大时，线模糊，所以直接复制是无法做到wr的矩形那样

```csharp
       protected override void OnRender(DrawingContext dc)
        {

            dc.DrawRoundedRectangle(null, _pen, new Rect(new Point(10, 10), new Size(100, 100)), 5, 5);
        }
```

在界面画出来wr 的矩形和自定义控件，可以看到，微软的是清晰的

那么是不是wr 做了特殊的东西，到现在还不知道，但是找到了一个方法，可以画出清晰

缩小看到的图片是这样

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017720205458.jpg)

那么放大时就是下面这张图

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017720205544.jpg)

所以需要在放大时，也画一个像素，
这个方法就是本文，所以这是在翻译，只是没有使用对所有的文字翻译，来自工藤大神的方法。

本文使用的方法很简单，第一步

复制方法到一个静态类

```csharp
    public static void DrawSnappedLinesBetweenPoints(this DrawingContext dc,
        Pen pen, double lineThickness, params Point[] points)
    {
        var guidelineSet = new GuidelineSet();
        foreach (var point in points)
        {
            guidelineSet.GuidelinesX.Add(point.X);
            guidelineSet.GuidelinesY.Add(point.Y);
        }
        var half = lineThickness / 2;
        points = points.Select(p => new Point(p.X + half, p.Y + half)).ToArray();
        dc.PushGuidelineSet(guidelineSet);
        for (var i = 0; i < points.Length - 1; i = i + 2)
        {
            dc.DrawLine(pen, points[i], points[i + 1]);
        }
        dc.Pop();
    }
```

然后就可以在自定义控件使用下面的代码

```csharp
      protected override void OnRender(DrawingContext dc)
        {
            dc.DrawSnappedLinesBetweenPoints(_pen,1, new[]
            {
                new Point(10, 10),
                new Point(310, 10),
            });
        }
```

可以看到线是清晰的

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017720201831.jpg)

参见：https://stackoverflow.com/a/45189552/6116637

http://www.nbdtech.com/Blog/archive/2008/11/20/blurred-images-in-wpf.aspx

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。