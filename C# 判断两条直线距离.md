# C# 判断两条直线距离

本文告诉大家获得两条一般式直线距离。

<!--more-->
<!-- CreateTime:2018/7/31 14:38:13 -->

<!-- 标签：数学，C#，几何 -->
<!-- math -->

一般式的意思就是

$$
Ax+By+C=0
$$

如果有两个直线

$$
A_1x+B_1y+C_1=0 \\
A_2x+B_2y+C_2=0
$$

如何判断两条直线的距离？

如果需要判断两条直线的距离，首先两条直线需要是平行

判断一般式直线平行的方法

$$
A_1B_2-A_2B_1 \approx 0
$$

如果两条直线符合上面公式，可以认为两条直线平行。

对于一般的两条直线，获得距离的公式

$$
d= \frac{ \left| C_1-C_2 \right|}{\sqrt{A^2+B^2}}
$$

但是因为两个直线一般式的 AB 是不相等的，所以需要把两个直线转换相同的 AB

$$
A_1x+B_1y+C_1=0 \\
A_2x\frac{A_1}{A_2}+B_2y\frac{A_1}{A_2}+C_2\frac{A_1}{A_2}=0 \\
A_1x+B_1y+C_2\frac{A_1}{A_2}=0
$$

这时的距离公式是

$$
d= \frac{ \left| C_1-C_2\frac{A_1}{A_2}\right|}{\sqrt{A_1^2+B_1^2}}
$$

但是存在 A 或 B 是 0 ，所以就不能直接使用上面的距离

如果$a=0 ,b \neq 0$ 那么需要修改直线公式

$$
B_1y+C_1=0 \\
B_1y+C_2\frac{B_1}{B_2}=0
$$

这时距离公式

$$
d= \frac{ \left| C_1-C_2\frac{B_1}{B_2}\right|}{B_1}
$$

如果$a\neq0 ,b = 0$ 那么需要修改直线公式

$$
A_1x+C_1=0 \\
A_1x+C_2\frac{A_1}{A_2}=0
$$

这时距离公式

$$
d= \frac{ \left| C_1-C_2\frac{A_1}{A_2}\right|}{A_1}
$$

因为我是在编程，我可以拿到距离平方，这样可以减少开方，我把上面的公式写为代码，代码是C#不过大家可以把他使用其他语言

```csharp
       /// <summary>
        /// 获得两条直线的距离，传入的直线已经是判断平行
        /// </summary>
        /// <param name="otherLine"></param>
        /// <returns></returns>
        public double? GetDistanceWithLineSquare(LineEquation otherLine)
        {
            var aIsZero = A.IsZero();
            var bIsZero = B.IsZero();

            //D=|C1-C2|/sqrt(A^2+B^2)

            // A 是 0 ，但是 B 不是 0
            if (aIsZero && !bIsZero)
            {
                //B1Y+C1=0 B1Y+B1/B2*C2=0
                return Math.Abs(C - B / otherLine.B * otherLine.C) / B*B;
            }

            if (!aIsZero && bIsZero)
            {
                //A1X+C1=0 A1X+A1/A2*C2=0
                return Math.Abs(C - A / otherLine.A * otherLine.C) / A*A;
            }

            if (!aIsZero && !bIsZero)
            {
                return Math.Abs(C - A / otherLine.A * otherLine.C) / (A * A + B * B);
            }

            if (aIsZero && bIsZero)
            {
                return default(double?);
            }
            return default(double?);
        }
```

上面代码的 `A.IsZero()` 就是判断 A 是不是为 0 ，在 C# 很难判断 double 是不是为 0 所以需要这个方法

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
