
# WPF 基础 2D 图形学知识 求向量旋转角度

求向量的三角函数 sin 或 cos 的值，已知两个点，求两点相连线段角度

<!--more-->


<!-- 发布 -->

在 WPF 或 UWP 中，可以通过两个点的减法获取向量

```csharp
Vector vector = p1 - p2;
```

求向量的三角函数 sin 或 cos 的值，可以使用如下代码

```csharp
    static class VectorExtensions
    {
        /// <summary>
        /// 获取向量的 cos（θ）值
        /// </summary>
        /// <param name="vector"></param>
        /// <returns></returns>
        public static double GetCos(this Vector vector)
            => vector.Y / vector.Length;

        /// <summary>
        /// 获取向量的 sin（θ）值
        /// </summary>
        /// <param name="vector"></param>
        /// <returns></returns>
        public static double GetSin(this Vector vector)
            => vector.X / vector.Length;
    }
```

通过反三角函数可以获取弧度值

```csharp
            var cosθ = vector.GetCos();
            var sinθ = vector.GetSin();

            var 弧度 = Math.Acos(cosθ);
```

从弧度转换角度，可以使用以下方法转换

```csharp
var 角度 = 弧度 / Math.PI * 180;
```

此时比较不推荐使用 tan 这个三角函数，因为也许会出现除以零的问题

更多请看 [WPF 基础 2D 图形学知识](https://blog.lindexi.com/post/WPF-%E5%9F%BA%E7%A1%80-2D-%E5%9B%BE%E5%BD%A2%E5%AD%A6%E7%9F%A5%E8%AF%86.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。