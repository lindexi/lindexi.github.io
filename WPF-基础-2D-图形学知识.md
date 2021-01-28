
# WPF 基础 2D 图形学知识

本文收集一些基础的知识，本文的逻辑是在 WPF 框架下实现，有包含了默认的坐标系以及默认类型定义。对于 WPF 系的包括 Xamarin 和 UWP 都适合

<!--more-->



## 运行代码

本文的代码都放在 GitHub 或 Gitee 上，代码都可以下载进行运行。基本的代码都可以使用一句 `dotnet run` 跑起来，当然，前提是你的 dotnet 版本需要足够新

本文代码协议基于 MIT 协议，请放心抄代码

## 根据点集求外接矩形

先看图片，通过给定的点的集合，求这些点的外接矩形

<!-- ![](image/WPF 基础 2D 图形学知识/根据点集求外接矩形.gif) -->

![](http://image.acmx.xyz/lindexi%2F%25E6%25A0%25B9%25E6%258D%25AE%25E7%2582%25B9%25E9%259B%2586%25E6%25B1%2582%25E5%25A4%2596%25E6%258E%25A5%25E7%259F%25A9%25E5%25BD%25A2.gif)

传入的是 `List<Point> pointList` 要求传出的是 Rect 类，实现代码如下

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识0.png) -->

![](http://image.acmx.xyz/lindexi%2F2021127194264742.jpg)

在新建矩形的时候，采用了第一个点创建，如果没有传入点，将使用默认的原点

```csharp
        private Rect CreateRect(List<Point> pointList)
        {
            var rect = new Rect(pointList[0], new Size(0, 0));

            for (var i = 1; i < pointList.Count; i++)
            {
                rect.Union(pointList[i]);
            }

            return rect;
        }
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/df873f1e/LeajemhurhoCaiwhemqurhahawwhaw ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/df873f1e/LeajemhurhoCaiwhemqurhahawwhaw ) 欢迎小伙伴访问

## 绘制闭合折线

通过 Polygon 可以根据点集绘制闭合折线

```csharp
            var polygon = new Polygon()
            {
                Points = new PointCollection(PointList),
                Stroke = Brushes.Red,
            };
```

<!-- ![](image/WPF 基础 2D 图形学知识/绘制闭合折线.gif) -->

![](http://image.acmx.xyz/lindexi%2F%25E7%25BB%2598%25E5%2588%25B6%25E9%2597%25AD%25E5%2590%2588%25E6%258A%2598%25E7%25BA%25BF.gif)

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0d947fe2/LeajemhurhoCaiwhemqurhahawwhaw ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0d947fe2/LeajemhurhoCaiwhemqurhahawwhaw ) 欢迎小伙伴访问


## 判断点在几何内

这个做法也叫命中测试，输入是一个 Geometry 和一个点，输出是判断点是否在闭合的 Geometry 几何内。方法是通过 WPF 的 Geometry 的 FillContains 方法，这个方法可以传入点也可以传入另一个 Geometry 用来判断是否在几何内

```csharp
Geometry.FillContains(position)
```

和 FillContains 相对的是 StrokeContains 方法，和 Fill 方法不相同的是，调用 StrokeContains 判断的是在几何的线上，而不是在几何内

我写了一点测试的逻辑，如果鼠标在几何内，那么几何显示灰色

<!-- ![](image/WPF 基础 2D 图形学知识/判断点在几何内.gif) -->

![](http://image.acmx.xyz/lindexi%2F%25E5%2588%25A4%25E6%2596%25AD%25E7%2582%25B9%25E5%259C%25A8%25E5%2587%25A0%25E4%25BD%2595%25E5%2586%2585.gif)

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/5f804a35/LeajemhurhoCaiwhemqurhahawwhaw ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/5f804a35/LeajemhurhoCaiwhemqurhahawwhaw ) 欢迎小伙伴访问

## 给定中心点和宽度高度旋转角度求旋转矩形顶点坐标

如有定义旋转矩形的顶点分别是 A B C D 四个点，在没有进行旋转之前如图

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识1.png) -->

![](http://image.acmx.xyz/lindexi%2F20211272028498690.jpg)

给定中心点 O1 和宽度高度旋转角度弧度表示可以创建旋转矩形，代码逻辑如下

```csharp
    class 旋转矩形
    {
        public 旋转矩形(Point a, Point b, Point c, Point d)
        {
            /*
             *A    B
             *------
             *|    |
             *|    |
             *------
             *C    D
             */
            // 顺序需要是一个逆时针，因此就是 A C D B 的传入

            Polygon = new Point[4]
            {
                a, c, d, b
            };

            A = a;
            B = b;
            C = c;
            D = d;
        }

        public static 旋转矩形 Create旋转矩形(Point position, double width, double height, double rotation)
        {
            var w = width;
            var h = height;

            var ax = -w / 2;
            var ay = -h / 2;
            var bx = w / 2;
            var by = ay;
            var cx = ax;
            var cy = h / 2;
            var dx = bx;
            var dy = cy;

            var a = 已知未旋转的相对矩形中心点的坐标求旋转后的相对于零点的坐标(ax, ay, position, rotation);
            var b = 已知未旋转的相对矩形中心点的坐标求旋转后的相对于零点的坐标(bx, by, position, rotation);
            var c = 已知未旋转的相对矩形中心点的坐标求旋转后的相对于零点的坐标(cx, cy, position, rotation);
            var d = 已知未旋转的相对矩形中心点的坐标求旋转后的相对于零点的坐标(dx, dy, position, rotation);
            return new 旋转矩形(a, b, c, d);
        }

        /// <summary>
        /// 根据未旋转的相对圆角矩形 中心点 的坐标计算旋转后的相对于零点的坐标。
        /// </summary>
        /// <returns>旋转后的相对于零点的坐标</returns>
        private static Point 已知未旋转的相对矩形中心点的坐标求旋转后的相对于零点的坐标(double x, double y, Point position, double 旋转角度弧度)
        {
            var x0 = position.X;
            var y0 = position.Y;
            var θ = 旋转角度弧度;
            return new Point(x * Math.Cos(θ) - y * Math.Sin(θ) + x0, x * Math.Sin(θ) + y * Math.Cos(θ) + y0);
        }

        public Point A { get; }
        public Point B { get; }
        public Point C { get; }
        public Point D { get; }

        public Point[] Polygon { get; }
    }
```

上面代码的 Polygon 仅仅只是用来给界面显示

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/12324b85/LeajemhurhoCaiwhemqurhahawwhaw ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/12324b85/LeajemhurhoCaiwhemqurhahawwhaw ) 欢迎小伙伴访问

## 求旋转矩形命中测试

这是纯数学计算，给定一个旋转矩形，已知这个旋转矩形的各个顶点坐标。以及一个点，求这个点是否在旋转矩形内

定义给定的点是 M 点，而旋转矩形顶点是 A B C D 点。在旋转矩形没有经过旋转的顶点如下

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识1.png) -->

![](http://image.acmx.xyz/lindexi%2F20211272028498690.jpg)

其实在不在 WPF 中，影响都不大，如何判断一个点在旋转后的矩形中，只需要根据公式计算就可以

根据公式可以求出点是否在旋转矩形

```
 (0<AM⋅AB<AB⋅AB)∧(0<AM⋅AC<AC⋅AC)
```

以上逻辑中的 AM 等表示的是向量。在 WPF 中可以使用两个点相减拿到向量。求 AM 的向量就是使用 A 点减去 M 点

```csharp
   var am = A - m;
```

判断代码

```csharp
        public bool Contains(Point point)
        {
            // https://math.stackexchange.com/a/190373/440577
            // (0<AM⋅AB<AB⋅AB)∧(0<AM⋅AC<AC⋅AC)
            var am = A - point;
            var ab = A - B;
            double am_ab = am * ab;
            double ab_ab = ab * ab;

            var ac = A - C;
            double am_ac = am * ac;
            double ac_ac = ac * ac;

            if (am_ab > 0 && am_ab < ab_ab /*(0<AM⋅AB<AB⋅AB)*/
                          && am_ac > 0 && am_ac < ac_ac)
            {
                return true;
            }

            return false;
        }
```

以上数学的证明大概如下，有旋转矩形如下，有点 M 如下

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识2.png) -->

![](http://image.acmx.xyz/lindexi%2F2021127204044398.jpg)

定义的向量如下

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识3.png) -->

![](http://image.acmx.xyz/lindexi%2F20211272042302440.jpg)

简单来说的向量的点乘的含义就是降向量维度，如上面的二维向量的点乘可以拿到一维的标量的值

```csharp
            double am_ab = am * ab;
            double ab_ab = ab * ab;

            double am_ac = am * ac;
            double ac_ac = ac * ac;
```

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识4.png) -->

![](http://image.acmx.xyz/lindexi%2F2021127204705505.jpg)

从图片可以看到所有的向量都从 A 点出发，此时可以将 A 点设置为原点，如果此时的 M 是在矩形外，如认为是在如下图的左边，那么此时向量相乘的值就会是负数，因为相对于 A 作为原点

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识5.png) -->

![](http://image.acmx.xyz/lindexi%2F2021127204956927.jpg)

因此小于零的就不在矩形内，这就是旋转之前水平方向的判断 `0<AM⋅AB` 的依据

而如果 `AB⋅AB` 就表示 AB 的向量长度，也就是说如果 AM 的距离实际上大于 AB 的距离，如点在矩形的右边，那么点也不在矩形内

<!-- ![](image/WPF 基础 2D 图形学知识/WPF 基础 2D 图形学知识6.png) -->

![](http://image.acmx.xyz/lindexi%2F2021127205388240.jpg)

因此旋转之前的水平方向需要满足 `0<AM⋅AB<AB⋅AB` 才可以。而垂直方向也同理，只是将 AB 修改为 AC 两点

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/12324b85/LeajemhurhoCaiwhemqurhahawwhaw ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/12324b85/LeajemhurhoCaiwhemqurhahawwhaw ) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。