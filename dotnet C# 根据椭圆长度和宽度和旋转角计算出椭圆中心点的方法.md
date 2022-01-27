# dotnet C# 根据椭圆长度和宽度和旋转角计算出椭圆中心点的方法

本文来告诉大家如何根据椭圆长度和宽度和旋转角计算出椭圆中心点的方法

<!--more-->
<!-- CreateTime:2021/8/28 10:22:57 -->

<!-- 发布 -->

方法很简单，请看代码

```csharp
    /// <summary>
    /// 辅助进行椭圆点计算的类
    /// </summary>
    /// 我觉得这个类应该是框架有带，或现成的方法，但是一时间没找到
    static class EllipseCoordinateHelper
    {
        /// <summary>
        /// 计算椭圆中点坐标
        /// </summary>
        /// <param name="widthRadius"></param>
        /// <param name="heightRadius"></param>
        /// <param name="rotationAngle"></param>
        /// <returns></returns>
        public static (Pixel x, Pixel y) GetEllipseCoordinate(Pixel widthRadius, Pixel heightRadius,
            Degree rotationAngle)
        {
            // 以下为椭圆两个点的计算方法
            // 算法请看 https://astronomy.swin.edu.au/cms/astro/cosmos/E/Ellipse

            var absRotate = Math.Abs(rotationAngle.DoubleValue);
            var rad = Math.Abs(absRotate - 90);
            rad = rad * Math.PI / 180;
            var tan = Math.Tan(rad);

            var a = widthRadius.Value;
            var b = heightRadius.Value;
            var x = Math.Sqrt(1.0 / (1.0 / (a * a) + (tan * tan) / (b * b)));
            var y = x * tan;

            if (rotationAngle.DoubleValue < 0)
            {
                x = -x;
            }

            if (rotationAngle.DoubleValue > -90 && rotationAngle.DoubleValue < 90)
            {
                y = -y;
            }

            x = a + x;
            y = b + y;

            return (new Pixel(x), new Pixel(y));
        }
    }
```

我觉得以上是 WPF 框架有带的，但是一时半会没有找到在哪定义的，因此就自己写了一份

以上的 Pixel 和 Degree 的定义代码在 GitHub 上开源，请看 [Office Open XML 的测量单位](https://blog.lindexi.com/post/Office-Open-XML-%E7%9A%84%E6%B5%8B%E9%87%8F%E5%8D%95%E4%BD%8D.html )

其他计算请参阅 [根据SVG Arc求出其开始角、摆动角和椭圆圆心 - RyzenAdorer - 博客园](https://www.cnblogs.com/ryzen/p/15832672.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
