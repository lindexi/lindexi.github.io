# WPF 如何计算矩形内一个坐标相对另一个矩形的坐标

我在 WPF 中拿到一个矩形里面的一个坐标，在这个矩形里面包含了另一个矩形，我想将这个点转换到另一个矩形里面的坐标。也就是说我拿到一个点，这个点的左上角(0,0)坐标就是矩形1的左上角坐标，而我想要将这个点转换为以矩形2的左上角坐标作为原点的坐标系的坐标

<!--more-->
<!-- CreateTime:2019/11/30 8:28:58 -->

<!-- csdn -->

其实做法就是将矩形2的左上角坐标换算为以矩形1作为原点的坐标，然后将这个点的坐标减去矩形2的左上角就可以计算出当前的点所在矩形2的坐标

定义方法 `private void TranslatePoint(Rect originRect, Rect rect, Point point)` 将点 point 从 originRect 的坐标转换为在矩形 rect 的坐标

如果此时的 originRect 的坐标系和 rect 的坐标系相同，那么有两个方法，第一个方法就是将 rect 转换为 originRect 的坐标系，然后再计算坐标系内的转换。第二个方法时将 point 转换坐标系，让 point 的坐标系和 rect 的坐标系相同

尝试方法一将 rect 转换为 originRect 的坐标系，需要拿到两个矩形之间的向量，也就是将 rect 的左上角减去 originRect 的左上角，将拿到的向量的点作为 rect 的左上角的值

```csharp
        private void TranslatePoint(Rect originRect, Rect rect, Point point)
        {
            var vector = rect.TopLeft - originRect.TopLeft;
            rect = new Rect(vector.X, vector.Y, rect.Width, rect.Height);
        }
```

上面代码就将 rect 转换了坐标系，相当于将 rect 放入了 originRect 矩形

然后进行矩形内的坐标换算，也就是 rect 使用 originRect 的左上角作为原点的坐标系，此时的坐标系和 point 的坐标系相同，也就是计算在相同坐标系的一个点相对于矩形的点

方法通过将点减去矩形的左上角

```csharp
            vector = point - rect.TopLeft;
            point = new Point(vector.X, vector.Y);
```

此时计算的 point 点就是相对于 rect 的点

尝试方法2将 point 转换为和 rect 相同的原点，方法是通过将点加上矩形左上角

```csharp
        private void TranslatePoint(Rect originRect, Rect rect, Point point)
        {
            var x = point.X + originRect.TopLeft.X;
            var y = point.Y + originRect.TopLeft.Y;
            point = new Point(x, y);

        }
```

因为此时的 point 和 rect 在相同坐标系按照上面的方法计算

```csharp
            var vector = point - rect.TopLeft;
            point = new Point(vector.X, vector.Y);
```

两个计算方法如下

```csharp

// 方法1

        private void TranslatePoint(Rect originRect, Rect rect, Point point)
        {
            var vector = rect.TopLeft - originRect.TopLeft;
            rect = new Rect(vector.X, vector.Y, rect.Width, rect.Height);

            vector = point - rect.TopLeft;
            point = new Point(vector.X, vector.Y);
        }

// 方法2

        private void TranslatePoint(Rect originRect, Rect rect, Point point)
        {
            var x = point.X + originRect.TopLeft.X;
            var y = point.Y + originRect.TopLeft.Y;
            point = new Point(x, y);

            var vector = point - rect.TopLeft;
            point = new Point(vector.X, vector.Y);
        }
```

将方法1的计算合并

```csharp
var x = point.X - (rect.TopLeft.X - originRect.TopLeft.X);
var y = point.Y - (rect.TopLeft.Y - originRect.TopLeft.Y);
```

而方法2的计算合并

```csharp
var x = point.X + originRect.TopLeft.X - rect.TopLeft.X;
var y = point.Y + originRect.TopLeft.Y - rect.TopLeft.Y;
```

也就是两个方法的计算是相同

那么假设每个矩形都是左上角都是原点只是因为叠加了矩阵变换才到了当前的坐标，这样就可以应用矩阵计算

开始之前请先复习一下 WPF 的矩阵变换，在 WPF 中变换的矩阵时一个 `3*3` 矩阵，其中最后一列是占坑的不开放修改。矩阵上面的 `M11 M12 M21 M22` 是线性部分，而 `offsetX offsetY` 是平移部分。这里的线性部分指的是旋转和缩放

在 WPF 会将元素的原来的坐标计为 `x y 1` 最后的 1 就是占坑，对元素进行变换就是通过矩阵乘法

```
          | M11     M12     0 |
|x y 1| * | M21     M22     0 |
          | offsetX offsetY 1 |
```

接下来复习一下矩阵的乘法计算，假设有矩阵 a 和 b 按照下面方法计算，要求 a 的列数等于 b 的行数的时候才能相乘，这就是占坑的数的意义

```
ai1 * b1j + ai2 * b2j + ... aik * bkj
```

也就是按照 a 的每一行和 b 的每一列相乘计算

按照这个方法可以计算出矩阵乘法之后的值

```
| x*M11 + y*M21 + 1*offsetX, x*M12 + y*M22 + 1*offsetY, 0*x + 0*y + 1*1 |
```

此时的平移矩阵如下

```
| 1 0 0 |
| 0 1 0 |
| x y 1 |
```

而此时的两个矩形都是左上角作为原点也就是元素的 x 和 y 都是0相当于

```
| 0*1 + 0*0 + 1*x, 0*0 + 0*0 + 1*y, 0*0 + 0*0 + 1*1 |
```

那么将 originRect 里面的点转换坐标出来的做法就是去掉叠加的矩阵，也就是相当于乘以 originRect 的矩阵

```csharp
point * originRectMatrix
```

然后反过来叠加 rect 的矩阵，也就是将 rect 的矩阵乘以 -1 再乘以 point 坐标

```csharp
point * (-1 * rectMatrix)
```

这样通过矩阵就可以计算在 originRect 里面的点相对于另一个矩形坐标

通过矩阵计算可以应用到显卡的计算加速

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
