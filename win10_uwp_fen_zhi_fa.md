# win10 uwp 分治法

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

其实我想说Path，因为最近在做一个简单的分治。

算法涉及到了一个平面几何的知识。就是三角形p1p2p3的面积等于以下行列式的二分之一： 
$$
\begin{array}{cccc} 
| x1 & y1 & 1 & | \\
| x2 & y2 & 1 & | &=\\
| x3 & y3 & 1 & | \\
\end{array}
$$


 $$x1*y2+x3*y1+x2*y3-x3*y2-x2*y1-x1*y3  = x1*y2-x2*y1+x3*(y1-y2)+y3*(x2-x1)$$
 
而且当点P3 在射线P1P2的左侧的时候，表达式为正，右侧表达式为负，三点同线的话表达式为0；算法中就利用该几何特性判断一个点在一条线的左侧还是右侧。

参见：http://www.cnblogs.com/Booble/archive/2011/03/10/1980089.html

我们有了一个后台，他可以有很多点，和得到边，我们如何从拿到的`List<Point> point`画出，和拿到的边的点画出

其实我们可以用简单的Path，如何从Path画点

我们可以使用EllipseGeometry

EllipseGeometry是Geometry，看到Geometry大家会看到Path的Data，是的，我们可以使用

```csharp
                Windows.UI.Xaml.Shapes.Path path = new Windows.UI.Xaml.Shapes.Path
                {
                    Data = new EllipseGeometry()
                };
```

为什么画点我会使用EllipseGeometry，因为我就需要一个点作为中心，X的大小和Y的，然后就是点

```csharp
                Windows.UI.Xaml.Shapes.Path path = new Windows.UI.Xaml.Shapes.Path
                {
                    Data = new EllipseGeometry()
                    {
                        Center = point,
                        RadiusX = 5,
                        RadiusY = 5
                    }
                };
```

那么我们需要给点颜色

断句不要弄错，是给 点 ，颜色

实心：Fill = new SolidColorBrush(Colors.Gray)，因为我们可以使用简单Colors，如果需要RBG，那么可以使用

```csharp
                    Fill = new SolidColorBrush(new Color()
                    {
                        R = 0,
                        B = 0,
                        G = 0
                    })
```

然而这样觉得还是不好，我们本来不用十进制

```csharp
                    Fill = new SolidColorBrush(new Color()
                    {
                        R = 0x23,
                        B = 0x54,
                        G = 0xa
                    })
```

博客：blog.csdn.net/lindexi_gd

如果觉得上面代码多：

```csharp
Fill = new SolidColorBrush(Color.FromArgb(0xff,0xff,0xa,0x2))
```

我们这样还是好多，不过垃圾的wr没有给我们string转Color，工藤给我微软的自带可以把string转为Color因为简单，我就没有写，现在想要，找了很久，如果需要可以进： 53078485

我们现在已经弄好画点，但是空心没画

```csharp
Stroke = new SolidColorBrush(Colors.Gray)
```

这样我们就可以画空心和实心

用之前的代码作为我们后台

![这里写图片描述](http://img.blog.csdn.net/20160523191436593)

我们需要连线

连线

```csharp
            n = point.Count;

            PathFigure figures = new PathFigure();

            for (int i = 0; i < n; i++)
            {
                figures.Segments.Add(new LineSegment()
                {
                    Point = point[i]
                });
            }
            figures.Segments.Add(new LineSegment()
            {
                Point = point[0]
            });
            figures.StartPoint = point[0];

            Windows.UI.Xaml.Shapes.Path path_figure = new Path()
            {
                Data = new PathGeometry()
                {
                    Figures = new PathFigureCollection()
                    {
                        figures
                    }
                },
                Stroke = new SolidColorBrush(Colors.Gray)
            };
```


如果觉得这样太快了，我们可以弄个差

```csharp
            PathGeometry path_figure = new PathGeometry();
            for (int i = 0; i < point.Count; i++)
            {
                PathFigure path_segment = new PathFigure()
                {
                    StartPoint = point[i]
                };
                i++;
                LineSegment line = new LineSegment()
                {
                    Point = point[i]
                };
                path_segment.Segments.Add(line);
                path_figure.Figures.Add(path_segment);
            }
```

PathFigure第一个点`StartPoint = point[i]`，LineSegment第二个，`path_segment.Segments.Add(line);`，把`path_segment`放在我们外面定义`path_figure`

这样比第一个会多了`path_segment`，这个变量命名不对，但是我现在不想去改

代码：https://github.com/lindexi/Algorithm

做完我来运行

![这里写图片描述](http://img.blog.csdn.net/20160523191446249)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。