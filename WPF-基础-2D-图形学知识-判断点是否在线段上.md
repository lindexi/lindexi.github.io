
# WPF 基础 2D 图形学知识 判断点是否在线段上

在知道一个使用两个点表示的线段，和另一个点，求另一个点是否在线段上

<!--more-->


<!-- CreateTime:2021/3/18 21:05:11 -->

<!-- 发布 -->

本文算法属于通用的算法，可以在 WPF 和 UWP 和 Xamarin 等上运行，基本上所有的 .NET 平台都能执行

如下图，如果点在线段上，那么修改线段颜色

<!-- ![](image/WPF 基础 2D 图形学知识 判断点是否在线段上/WPF 基础 2D 图形学知识 判断点是否在线段上0.gif) -->

![](http://image.acmx.xyz/lindexi%2FWPF%2520%25E5%259F%25BA%25E7%25A1%2580%25202D%2520%25E5%259B%25BE%25E5%25BD%25A2%25E5%25AD%25A6%25E7%259F%25A5%25E8%25AF%2586%2520%25E5%2588%25A4%25E6%2596%25AD%25E7%2582%25B9%25E6%2598%25AF%25E5%2590%25A6%25E5%259C%25A8%25E7%25BA%25BF%25E6%25AE%25B5%25E4%25B8%258A0.gif)

假定有线段的定义如下

```csharp
    public record Line
    {
        public Point APoint { get; init; }

        public Point BPoint { get; init; }
    }
```

以上代码使用了 .NET 5 加 C# 9.0 的新语法

在传入一个点，求这个点是否在线段上，最简单理解的算法是根据两点之间直线距离最短，只需要求 P 点和线段的 AB 两点的距离是否等于 AB 的距离。如果相等，那么证明 P 点在线段 AB 上，代码如下

```csharp
        private static bool CheckIsPointOnLine(Point point, Line line, double epsilon = 0.1)
        {
            // 最简单理解的算法是根据两点之间直线距离最短，只需要求 P 点和线段的 AB 两点的距离是否等于 AB 的距离。如果相等，那么证明 P 点在线段 AB 上
            var ap = point - line.APoint;
            var bp = point - line.BPoint;
            var ab = line.BPoint - line.APoint;

            // 只不过求 Length 内部需要用到一次 Math.Sqrt 性能会比较差
            if (Math.Abs(ap.Length + bp.Length - ab.Length) < epsilon)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
```

不使用 Vector 类，可以替换为如下计算方法。上面代码的 ap 等变量是使用 WPF 的两个点的相减能拿到 Vectore 类，而在 Vectore 类里面有 Length 属性而优化代码的。其实核心计算和下面代码相同。下面代码是 [Tone Škoda](https://stackoverflow.com/users/3572009/tone-%c5%a0koda) 提供的，详细请看 [https://stackoverflow.com/a/56850069/6116637](https://stackoverflow.com/a/56850069/6116637)

```csharp
public static double CalcDistanceBetween2Points(double x1, double y1, double x2, double y2)
{
    return Math.Sqrt(Math.Pow (x1 - x2, 2) + Math.Pow (y1 - y2, 2));
}

public static bool PointLinesOnLine (double x, double y, double x1, double y1, double x2, double y2, double allowedDistanceDifference)
{
    double dist1 = CalcDistanceBetween2Points(x, y, x1, y1);
    double dist2 = CalcDistanceBetween2Points(x, y, x2, y2);
    double dist3 = CalcDistanceBetween2Points(x1, y1, x2, y2);
    return Math.Abs(dist3 - (dist1 + dist2)) <= allowedDistanceDifference;
}
```

以下是另一个方法，以下方法性能比上面一个好

根据点和任意线段端点连接的线段和当前线段斜率相同，同时点在两个端点中间，就可以认为点在线段内

```csharp
(x - x1) / (x2 - x1) = (y - y1) / (y2 - y1)
```

因为乘法性能更高，因此计算方法可以如下

```csharp
 (x - x1) * (y2 - y1) = (y - y1) * (x2 - x1)
 (x - x1) * (y2 - y1) - (y - y1) * (x2 - x1) = 0
```

但是乘法的误差很大，因此还是继续使用除法

另外，需要判断点在两个端点中间

```csharp
             x1 < x < x2, assuming x1 < x2
             y1 < y < y2, assuming y1 < y2
```

以下是代码

```csharp
            if (EqualPoint(point, line.APoint, epsilon) || EqualPoint(point, line.BPoint, epsilon))
            {
                return true;
            }

            // 乘法性能更高，误差大。请试试在返回 true 的时候，看看 crossProduct 的值，可以发现这个值依然很大
            var crossProduct = (point.X - line.APoint.X) * (line.BPoint.Y - line.APoint.Y) -
                               (point.Y - line.APoint.Y) * (line.BPoint.X - line.APoint.X);

            if (Math.Abs((point.X - line.APoint.X) / (line.BPoint.X - line.APoint.X) - (point.Y - line.APoint.Y) / (line.BPoint.Y - line.APoint.Y)) < epsilon)
            {
                var minX = Math.Min(line.APoint.X, line.BPoint.X);
                var maxX = Math.Max(line.APoint.X, line.BPoint.X);

                var minY = Math.Min(line.APoint.Y, line.BPoint.Y);
                var maxY = Math.Max(line.APoint.Y, line.BPoint.Y);

                if (minX < point.X && point.X < maxX && minY < point.Y && point.Y < maxY)
                {
                    return true;
                }
                else
                {
                    return false;
                }
            }
            else
            {
                return false;
            }
```

上面代码的 crossProduct 是不用使用的，只是为了告诉大家，尽管乘法性能比较好，但是误差比较大

当然以上算法有漏洞，在于如果 A 和 B 两个点的 Y 坐标相同或 X 坐标相同的时候，那么以上算法不适合。可以先判断 crossProduct 的值，如果是等于零，那么证明有 A 和 B 两个点的 Y 坐标相同或 X 坐标相同

```csharp
            var crossProduct = (point.X - line.APoint.X) * (line.BPoint.Y - line.APoint.Y) -
                               (point.Y - line.APoint.Y) * (line.BPoint.X - line.APoint.X);
            // 先判断 crossProduct 是否等于 0 可以解决 A 和 B 两个点的 Y 坐标相同或 X 坐标相同的时候，使用除法的坑
            if (crossProduct == 0 || Math.Abs((point.X - line.APoint.X) / (line.BPoint.X - line.APoint.X) - (point.Y - line.APoint.Y) / (line.BPoint.Y - line.APoint.Y)) < epsilon)
            {
                var minX = Math.Min(line.APoint.X, line.BPoint.X);
                var maxX = Math.Max(line.APoint.X, line.BPoint.X);

                var minY = Math.Min(line.APoint.Y, line.BPoint.Y);
                var maxY = Math.Max(line.APoint.Y, line.BPoint.Y);

                if (minX <= point.X && point.X <= maxX && minY <= point.Y && point.Y <= maxY)
                {
                    return true;
                }
                else
                {
                    return false;
                }
            }
            else
            {
                return false;
            }
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/ed61e82f/WokayficeKegayurbu ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/ed61e82f/WokayficeKegayurbu ) 欢迎小伙伴访问

以上方法的计算有些重复，其实加上了 crossProduct 只是为了水平和垂直的线段，其实可以做特殊处理，如下面代码

```csharp
    public static class Math2DExtensions
    {
        public static bool CheckIsPointOnLineSegment(Point point, Line line, double epsilon = 0.1)
        {
            // 以下是另一个方法，以下方法性能比上面一个好

            // 根据点和任意线段端点连接的线段和当前线段斜率相同，同时点在两个端点中间
            // (x - x1) / (x2 - x1) = (y - y1) / (y2 - y1)
            // x1 < x < x2, assuming x1 < x2
            // y1 < y < y2, assuming y1 < y2
            // 但是需要额外处理 X1 == X2 和 Y1 == Y2 的计算

            var minX = Math.Min(line.APoint.X, line.BPoint.X);
            var maxX = Math.Max(line.APoint.X, line.BPoint.X);

            var minY = Math.Min(line.APoint.Y, line.BPoint.Y);
            var maxY = Math.Max(line.APoint.Y, line.BPoint.Y);

            if (!(minX <= point.X) || !(point.X <= maxX) || !(minY <= point.Y) || !(point.Y <= maxY))
            {
                return false;
            }

            // 以下处理水平和垂直线段
            if (Math.Abs(line.APoint.X - line.BPoint.X) < epsilon)
            {
                // 如果 X 坐标是相同，那么只需要判断点的 X 坐标是否相同
                // 因为在上面代码已经判断了 点的 Y 坐标是在线段两个点之内
                return Math.Abs(line.APoint.X - point.X) < epsilon || Math.Abs(line.BPoint.X - point.X) < epsilon;
            }

            if (Math.Abs(line.APoint.Y - line.BPoint.Y) < epsilon)
            {
                return Math.Abs(line.APoint.Y - point.Y) < epsilon || Math.Abs(line.BPoint.Y - point.Y) < epsilon;
            }

            if (Math.Abs((point.X - line.APoint.X) / (line.BPoint.X - line.APoint.X) - (point.Y - line.APoint.Y) / (line.BPoint.Y - line.APoint.Y)) < epsilon)
            {
                return true;
            }
            else
            {
                return false;
            }
        }
    }

    public record Line
    {
        public Point APoint { get; init; }

        public Point BPoint { get; init; }
    }
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/05d0e495/WokayficeKegayurbu ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/05d0e495/WokayficeKegayurbu ) 欢迎小伙伴访问




<!-- 

This is my code which can run in WPF

```csharp

        private static bool CheckIsPointOnLine(Point point, Line line, double epsilon = 0.1)
        {
        	// Thank you Rob Agar
            // (x - x1) / (x2 - x1) = (y - y1) / (y2 - y1)
            // x1 < x < x2, assuming x1 < x2
            // y1 < y < y2, assuming y1 < y2

            if (EqualPoint(point, line.APoint, epsilon) || EqualPoint(point, line.BPoint, epsilon))
            {
                return true;
            }

            if (Math.Abs((point.X - line.APoint.X) / (line.BPoint.X - line.APoint.X) - (point.Y - line.APoint.Y) / (line.BPoint.Y - line.APoint.Y)) < epsilon)
            {
                var minX = Math.Min(line.APoint.X, line.BPoint.X);
                var maxX = Math.Max(line.APoint.X, line.BPoint.X);

                var minY = Math.Min(line.APoint.Y, line.BPoint.Y);
                var maxY = Math.Max(line.APoint.Y, line.BPoint.Y);

                if (minX < point.X && point.X < maxX && minY < point.Y && point.Y < maxY)
                {
                    return true;
                }
                else
                {
                    return false;
                }
            }
            else
            {
                return false;
            }
        }

        private static bool EqualPoint(Point a, Point b, double epsilon = 0.001)
        {
            return Math.Abs(a.X - b.X) < epsilon && Math.Abs(a.Y - b.Y) < epsilon;
        }

    public record Line
    {
        public Point APoint { get; init; }

        public Point BPoint { get; init; }
    }
```

My code is in [github](https://github.com/lindexi/lindexi_gd/tree/1995f3e6/WokayficeKegayurbu ) -->




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。