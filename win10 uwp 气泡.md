# win10 uwp 气泡

如果做聊天工具，需要气泡。

本文，如果写一个气泡控件需要如何做

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

<div id="toc"></div>
<!-- csdn -->
## WPF 气泡

先说如何在 WPF 做一个气泡。

可以看到，气泡就是一个和 Grid 差不多的东西，只是有边框，边框是一个气泡

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201755145936.jpg)

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F2017551502.jpg)

如何去写一个外框？

可以新建一个类，继承 Decorator ，就可以啦

现在的难点是如何获得子元素的大小。

可以看到一个气泡是尖的气泡和一个矩形组成

我做了一些修改，先做一个像这样的气泡

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20175515140.jpg)



气泡分为两部分，一个是尖头一个矩形

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20175515350.jpg)

可以看到，尖头大小可以固定，但是矩形必须使用子控件的大小

于是先假如子元素的宽度是100，高度 50 ，这样来画一个气泡。

如何画一个三角？

假设尖头宽度 10 高度 5 ，那么可以看到第一个点是 （0，5） 第二个点是 （5，0） 第三个点是 （10，5）

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201755165744.jpg)

需要知道， WPF 使用的布局不是和以前课本说的一样

但是除了尖头，还需要添加矩形的距离

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20175517242.jpg)

添加的距离是矩形宽度的一半减去5，看到这里一般不会觉得有困难。

于是添加到实际的值，这里矩形宽度为 100 于是最后的值就是 （45，5） ，第二个点是 （50，0） 第三个点是 （55，5）

接下来就是计算矩形的值，矩形的值就是 x=0 y=5 ，宽度 100 高度 50

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20175517557.jpg)

计算出来，就需要画出来。

PathFigure 可以画线，也就是通过他给他三个点就好

把上面的几个点写出来


```csharp
                            new PathFigure
                            {
                                IsClosed = false,
                                StartPoint = new Point(45, 5),
                                Segments = new PathSegmentCollection()
                                {
                                    new LineSegment(new Point(50), 0), true),
                                    new LineSegment(new Point(55, 5), true)
                                }
                            }
```

但需要把三角加到 PathGeometry 才可以显示


```csharp
    
                    Geometry1 = new PathGeometry()
                    {
                        Figures = new PathFigureCollection()
                        {
                            new PathFigure
                            {
                                IsClosed = false,
                                StartPoint = new Point(45, 5),
                                Segments = new PathSegmentCollection()
                                {
                                    new LineSegment(new Point(50), 0), true),
                                    new LineSegment(new Point(55, 5), true)
                                }
                            }
                        }
                    }
```

这样写在界面path，可以看到显示出来三角形，因为没有设置线条，所以没有把鼠标移到三角是看不到的

接着需要画矩形


```csharp
                        Geometry2 = new RectangleGeometry(new Rect(0, 5, 100,
                            50)
                        , 0, 0)
```

但是如何直接把两个显示，看起来是不对的，因为是一个矩形和三角，不是气泡

所以组合一下图形就好


```csharp
                 var cg = new CombinedGeometry
                {
                    Geometry1 ,
                    Geometry2 ,
                    GeometryCombineMode = GeometryCombineMode.Xor
                };
```

但是实际需要获得子元素的大小，也需要显示，那么显示可以先重写 OnRender

从 OnRender 画出的方法很简单


```csharp
                GuidelineSet guideLines = new GuidelineSet();
                drawingContext.PushGuidelineSet(guideLines);
                drawingContext.DrawGeometry(brush, pen, cg);
```
其中的 颜色自己定义，cg就是上面的图形。

但是这样的自定义控件需要设置宽高，如何使用子元素的宽高加上自己的padding？

如果只是重新显示，那么界面是不知道气泡的大小，所以得到的是没显示，为了让气泡可以显示，先给他一个宽高，这样就可以演示。

但是我需要直接就写如何获取子元素的大小，把他作为气泡的大小。

获取子元素可以通过重写 MeasureOverride 

第一步，测量子元素，通过子元素可以获得高度宽度


```csharp
                    Child.Measure(constraint);
```


定义自己的 padding ，这个值先随意给，表示气泡离元素距离

那么计算得到自己的大小就是 子元素的宽高加上 padding 加上气泡需要的外框

因为对于高度，需要加上气泡的高度 5 才可以，代码很容易就看懂，我就不说啦

```csharp
        protected override Size MeasureOverride(Size constraint)
        {
            Thickness padding = Padding;
            Size result = new Size();
            if (Child != null)
            {
                //测量子控件的大小
                Child.Measure(constraint);

                result.Width = Child.DesiredSize.Width + padding.Left + padding.Right;
                result.Height = Child.DesiredSize.Height + padding.Top + padding.Bottom + 5;
            }
            return result;
        }
```

拿到了子控件的高度，还需要重写自己的布局


```csharp
            protected override Size ArrangeOverride(Size arrangeSize)
        {
            Thickness padding = Padding;
            if (Child != null)
            {
                Size result = new Size();
                Child.Arrange(new Rect(new Point(padding.Left, 5 + padding.Top), Child.DesiredSize));
                result.Width = Child.DesiredSize.Width + padding.Left + padding.Right;
                result.Height = Child.DesiredSize.Height + padding.Top + padding.Bottom + 5;
                return result;
            }
            return arrangeSize;
        }
```
这里出现很多个 5 ，意思就是气泡高度，为了设置气泡高度，所以给他一个属性。


```csharp
         public double HeadHeight
        {
            get
            {
                return (double)GetValue(HeadHeightProperty);
            }
            set
            {
                SetValue(HeadHeightProperty, value);
            }
        }

         /// <summary>
        ///     标识 <see cref="HeadHeight" /> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty HeadHeightProperty = DependencyProperty.Register(
            "HeadHeight", typeof(double), typeof(PeakedAdorner), new PropertyMetadata(5d));
```


现在可以修改一下代码，让他可以自动适应

矩形的宽高可以通过自己的大小计算


```csharp
    Geometry2 = new RectangleGeometry(new Rect(0, HeadHeight, ActualWidth,
                            ActualHeight - HeadHeight)
                        , 0, 0)
```

可以看到 坐标没有变化，有变化的是高度，宽度，可以通过获得自己的大小设置，因为在计算大小已经从子元素加上自己的需要大小，所以得到的大小可以设置

那么现在的 OnRender 可以写为


```csharp
            protected override void OnRender(DrawingContext drawingContext)
        {
           
            
                Pen pen = new Pen();
                pen.Brush = BorderBrush;
                pen.Thickness = BorderThickness;

                var leftpad = (ActualWidth - HeadWidth) / 2;
                var cg = new CombinedGeometry
                {
                    Geometry1 = new PathGeometry()
                    {
                        Figures = new PathFigureCollection()
                        {
                            new PathFigure
                            {
                                IsClosed = false,
                                StartPoint = new Point(leftpad, HeadHeight),
                                Segments = new PathSegmentCollection()
                                {
                                    new LineSegment(new Point(leftpad + (HeadWidth / 2), 0), true),
                                    new LineSegment(new Point(leftpad + HeadWidth, HeadHeight), true)
                                }
                            }
                        }
                    },
                    Geometry2 = new RectangleGeometry(new Rect(0, HeadHeight, ActualWidth,
                            ActualHeight - HeadHeight)
                        , 0, 0),
                    GeometryCombineMode = GeometryCombineMode.Xor
                };


                GuidelineSet guideLines = new GuidelineSet();
                drawingContext.PushGuidelineSet(guideLines);
                drawingContext.DrawGeometry(Background, pen, cg);
            
        }

```

BorderBrush 也是自己设置的 ，BorderThickness 也是，于是继续设置背景色  Background 和其它的如圆角

现在看起来的属性是


```csharp
         public static readonly DependencyProperty BackgroundProperty =
            DependencyProperty.Register("Background", typeof(Brush), typeof(PeakedAdorner)
                , new PropertyMetadata(new SolidColorBrush(Color.FromRgb(255, 255, 255))));

        public static readonly DependencyProperty PaddingProperty =
            DependencyProperty.Register("Padding", typeof(Thickness), typeof(PeakedAdorner)
                , new PropertyMetadata(new Thickness(10, 5, 10, 5)));

        public static readonly DependencyProperty BorderBrushProperty =
            DependencyProperty.Register("BorderBrush", typeof(Brush), typeof(PeakedAdorner)
                , new PropertyMetadata(new SolidColorBrush(Colors.Black)));

        public static readonly DependencyProperty BorderThicknessProperty =
            DependencyProperty.Register("BorderThickness", typeof(double), typeof(PeakedAdorner),
                new PropertyMetadata(1d));


        /// <summary>
        ///     标识 <see cref="HeadWidth" /> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty HeadWidthProperty = DependencyProperty.Register(
            "HeadWidth", typeof(double), typeof(PeakedAdorner), new PropertyMetadata(0d));


        /// <summary>
        ///     标识 <see cref="HeadHeight" /> 的依赖项属性。
        /// </summary>
        public static readonly DependencyProperty HeadHeightProperty = DependencyProperty.Register(
            "HeadHeight", typeof(double), typeof(PeakedAdorner), new PropertyMetadata(5d));

        public static readonly DependencyProperty CornerRadiusProperty =
            DependencyProperty.Register("CornerRadius", typeof(CornerRadius)
                , typeof(PeakedAdorner), new PropertyMetadata(new CornerRadius(0)));

        /// <summary>
        ///     背景色
        /// </summary>
        public Brush Background
        {
            get
            {
                return (Brush)GetValue(BackgroundProperty);
            }
            set
            {
                SetValue(BackgroundProperty, value);
            }
        }

        /// <summary>
        ///     内边距
        /// </summary>
        public Thickness Padding
        {
            get
            {
                return (Thickness)GetValue(PaddingProperty);
            }
            set
            {
                SetValue(PaddingProperty, value);
            }
        }

        /// <summary>
        ///     边框颜色
        /// </summary>
        public Brush BorderBrush
        {
            get
            {
                return (Brush)GetValue(BorderBrushProperty);
            }
            set
            {
                SetValue(BorderBrushProperty, value);
            }
        }

        /// <summary>
        ///     边框大小
        /// </summary>
        public double BorderThickness
        {
            get
            {
                return (double)GetValue(BorderThicknessProperty);
            }
            set
            {
                SetValue(BorderThicknessProperty, value);
            }
        }

        /// <summary>
        ///     获取或设置尖角width
        /// </summary>
        public double HeadWidth
        {
            get
            {
                return (double)GetValue(HeadWidthProperty);
            }
            set
            {
                SetValue(HeadWidthProperty, value);
            }
        }

        /// <summary>
        ///     获取或设置尖角
        /// </summary>
        public double HeadHeight
        {
            get
            {
                return (double)GetValue(HeadHeightProperty);
            }
            set
            {
                SetValue(HeadHeightProperty, value);
            }
        }

        /// <summary>
        ///     边框大小
        /// </summary>
        public CornerRadius CornerRadius
        {
            get
            {
                return (CornerRadius)GetValue(CornerRadiusProperty);
            }
            set
            {
                SetValue(CornerRadiusProperty, value);
            }
        }

```

如何使用？


```csharp
            <local:PeakedAdorner CornerRadius="5" Margin="0,0,0.4,-0.2" >
            <TextBlock Text="林德熙"></TextBlock>
        </local:PeakedAdorner>
```

这样就好了，里面的控件可以是任何的，你想要的，如Grid ，textBox

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20175517277.jpg)

现在看起来就是这样

这样就做好气泡，如果需要气泡显示在其他的，那么可以通过自己计算，所有的值需要放在哪



