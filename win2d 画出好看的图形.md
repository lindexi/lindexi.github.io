# win2d 画出好看的图形

本文告诉大家，win2d 不需要从零开始做，以前做出来的很多库其实只需要做很小修改就可以做出好看的效果，而且用在 UWP 上。本文修改原先 大神写的 GDI 图形到 win2d 上，而且可以运行起来。

<!--more-->
<!-- CreateTime:2018/9/1 16:25:40 -->


<!-- 标签：uwp,win2d -->


一开始先发一张图片给大家看，本文就是告诉大家如何做出下面这张图的效果。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017623202940.jpg)

本文的算法是学习 山人大大的博客 http://blog.csdn.net/johnsuna/article/details/7981521 ，在他上面做一点修改做出来的。

可以看到他的博客使用的方法就是 GDI ，这是古时候使用的技术，而现在的 UWP 可以在以前的技术上，做一点修改就可以使用。

如果需要使用 win2d ，我希望大家先看这篇[文章](http://lindexi.oschina.io/lindexi/post/win10-uwp-win2d/)，本文不会继续告诉大家如何安装 win2d 。

首先需要了解一个技术，从 point 数组画出来。

如果给了一个 point 数组，那么可以使用这个数组画出形状。

在 win2d 下，可以使用 DrawGeometry 方法画出来。

可以看到 DrawGeometry 的第一个参数是 CanvasGeometry ，第二个参数是可以选的 ，一般使用颜色就好了。

但是 CanvasGeometry 是没有构造方法，所以 UWP 需要使用 CanvasGeometry.CreatePolygon 等方法创建出来。

本文因为需要把  point 数组显示，所以就使用了  CanvasGeometry.CreatePolygon 。这个方法的第一个参数是 ICanvasResourceCreator ，获得 ICanvasResourceCreator 很简单，传入 draw.Device 。

所以代码如下。

```csharp
        private void Canvas_OnDraw(CanvasControl sender, CanvasDrawEventArgs args)
        {
            var draw = args.DrawingSession;
            
            var canvasGeometry = CanvasGeometry.CreatePolygon(draw.Device,
                    point.Select(temp => new Vector2((float) temp.X, (float) temp.Y)).ToArray());
            draw.DrawGeometry(canvasGeometry, Color.FromArgb(255, 100, 100, 100));            
        }
```

那么 point 是如何得到，本文使用 http://blog.csdn.net/johnsuna/article/details/7981521  的方法获得。

```csharp
        public static Point[] CreateStone(Point center, int outerRadius, int innerRadius, int arms)
        {
            int centerX = (int) center.X;
            int centerY = (int) center.Y;
            Point[] points = new Point[arms * 2];
            double offset = Math.PI / 2;
            double arc = 2 * Math.PI / arms;
            double half = arc / 2;
            for (int i = 0; i < arms; i++)
            {
                Random randomOuter = new Random((int) DateTime.Now.Ticks);
                outerRadius = outerRadius -
                              randomOuter.Next((int) (innerRadius * 0.06 * new Random().Next(-20, 20) / 30d),
                                  (int) (innerRadius * 0.08));
                Random randomInner = new Random(Guid.NewGuid().GetHashCode());
                innerRadius = innerRadius +
                              randomInner.Next((int) (innerRadius * 0.02 * new Random().Next(-100, 100) / 150d),
                                  (int) (innerRadius * 0.08));
                if (innerRadius > outerRadius)
                {
                    int temp = outerRadius;
                    outerRadius = innerRadius;
                    innerRadius = temp;
                }
                double angleTemp = arc * randomInner.Next(-5, 5) / 10d;
                var angle = i * arc;
                angle += angleTemp;
                points[i * 2].X = (float) (centerX + Math.Cos(angle - offset) * outerRadius) + 20;
                points[i * 2].Y = (float) (centerY + Math.Sin(angle - offset) * outerRadius) + 20;
                points[i * 2 + 1].X = (float) (centerX + Math.Cos(angle + half - offset) * innerRadius) + 20;
                points[i * 2 + 1].Y = (float) (centerY + Math.Sin(angle + half - offset) * innerRadius) + 20;
            }

            return points;
        }

```

如果需要画出来图形，那么需要两个成员属性

```csharp
        private List<Point[]> _points = new List<Point[]>();

        private Dictionary<string, Stone> _stone;
```

`_points` 就是所有画出的点，`_stone` 就是告诉说，如何画，因为可以传入的数值 角度,长宽,胖瘦,微移间隙 等的不同，可以画出不同的图形，所以需要通过一个属性来告诉如何画

```csharp
    public class Stone
    {
        public static Point[] CreateStone(Point center, int outerRadius, int innerRadius, int arms)
        {
            int centerX = (int) center.X;
            int centerY = (int) center.Y;
            Point[] points = new Point[arms * 2];
            double offset = Math.PI / 2;
            double arc = 2 * Math.PI / arms;
            double half = arc / 2;
            for (int i = 0; i < arms; i++)
            {
                Random randomOuter = new Random((int) DateTime.Now.Ticks);
                outerRadius = outerRadius -
                              randomOuter.Next((int) (innerRadius * 0.06 * new Random().Next(-20, 20) / 30d),
                                  (int) (innerRadius * 0.08));
                Random randomInner = new Random(Guid.NewGuid().GetHashCode());
                innerRadius = innerRadius +
                              randomInner.Next((int) (innerRadius * 0.02 * new Random().Next(-100, 100) / 150d),
                                  (int) (innerRadius * 0.08));
                if (innerRadius > outerRadius)
                {
                    int temp = outerRadius;
                    outerRadius = innerRadius;
                    innerRadius = temp;
                }
                double angleTemp = arc * randomInner.Next(-5, 5) / 10d;
                var angle = i * arc;
                angle += angleTemp;
                points[i * 2].X = (float) (centerX + Math.Cos(angle - offset) * outerRadius) + 20;
                points[i * 2].Y = (float) (centerY + Math.Sin(angle - offset) * outerRadius) + 20;
                points[i * 2 + 1].X = (float) (centerX + Math.Cos(angle + half - offset) * innerRadius) + 20;
                points[i * 2 + 1].Y = (float) (centerY + Math.Sin(angle + half - offset) * innerRadius) + 20;
            }

            return points;
        }

        public Func<float, float, int, int, Point> GetCenter;
        public Func<float, int> GetinnerRadius;
        public Func<float, int> GetouterRadius;
        public int maxCorners = 18;
        public int minCorners = 3;
    }

```

所以在构造使用下面代码

```csharp
        public MainPage()
        {
            InitializeComponent();

            _stone = new Dictionary<string, Stone>()
            {
                {
                    "星", new Stone()
                    {
                        minCorners = 3,
                        maxCorners = 20,
                        GetCenter = (perX, perY, i, j) => new Point((int) (perX * j), (int) (perY * i)),
                        GetouterRadius = perX => (int) (perX * 0.18f),
                        GetinnerRadius = perX => (int) (perX * 0.06f)
                    }
                },
            };
        }

```

在界面弄个按钮

```csharp
        <xaml:CanvasControl x:Name="canvas" Margin="10,10,10,100" Height="600" ClearColor="White" Draw="Canvas_OnDraw"></xaml:CanvasControl>

         <Button Margin="10,600,10,10" Content="星" Click="Button_OnClick"></Button>           
            
```


然后写他的点击代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            string str = ((Button) sender).Content?.ToString();

            _points = new List<Point[]>();

            int width = 500;
            int height = 500;
            int numX = 10;
            int numY = 10;
            float perX = width * 1f / numX;
            float perY = height * 1f / numY;

            int minCorners = _stone[str].minCorners;
            int maxCorners = _stone[str].maxCorners;

            int lastCorners = minCorners;
            Random random = new Random();
            for (int i = 0; i < numX; i++)
            {
                for (int j = 0; j < numY; j++)
                {
                    int corners = random.Next(minCorners, maxCorners);
                    if (Math.Abs(corners - lastCorners) < (maxCorners - minCorners) / 2)
                        corners = RetrievRandomCorners(minCorners, maxCorners);
                    lastCorners = corners;
                    Point point = _stone[str].GetCenter(perX, perY, i, j);
                    int outerRadius = _stone[str].GetouterRadius(perX);
                    int innerRadius = _stone[str].GetinnerRadius(perX);
                    var points = Stone.CreateStone(point, outerRadius,
                        innerRadius, corners);
                    _points.Add(points);
                }
            }

            canvas.Invalidate();
        }

```

尝试跑一下，点击一次按钮就会有不同的图形，是不是感觉世界都是假的。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201762321046.jpg)

是不是觉得这样已经完成？实际上不要忘记很重要一步，win2d在不使用需要自己手动把他从视觉树释放，所以在后台代码页面跳出使用下面代码

```csharp

        private void Page_OnUnloaded(object sender, RoutedEventArgs e)
        {
            canvas.RemoveFromVisualTree();
            canvas = null;
        }
```

感觉这个星期的时间很快，我仔细想了从 2015 年开始 UWP 的研究到现在，实际我一个软件都没做出来，就是在写写博客。如果你觉得有哪个地方不想去看，或者垃圾微软写的不太好的，但你不想去研究的，就可以告诉我，我去研究一下，如果我学会了，我就会写一篇博客，虽然我写出来肯定比不上微软的。

代码：[https://github.com/lindexi/UWP/tree/master/uwp/control/win2d/Dclutterpalan](https://github.com/lindexi/UWP/tree/master/uwp/control/win2d/Dclutterpalan)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  