# C# double 好用的扩展

在很多代码需要使用数学计算，在用到 double 很难直接判断一个值是 0 或者 1 ，判断两个值相等。

本文提供一个数学扩展，让大家可以简单使用到 double 判断

<!--more-->
<!-- CreateTime:2019/3/1 9:19:05 -->


在开始看本文之前，希望大家是知道计算机是如何存放 double 和 double 精度问题原因。如果大家不知道这个的话，会比较难理解为什么需要使用扩展方法来判断。

如果只是想用这个类，请把到文章最后面，复制代码到自己项目。

例如有两个计算出来的 double ，分别是 a 和 b ，如果直接判断相等，那么 Resharper 会不开心，告诉你这个代码可能判断不对。

```csharp
 a == b
```

如果你问 Resharper 建议修改为怎样，他会告诉你，修改为这样

```csharp
 Math.Abs(a-b)<一个很小的数
```

原因就是 double 精度问题，虽然你觉得使用两个相同的方法计算出来的数值在数学计算上是相等的，但是实际上在进行判断的时候判断是不相等。

请注意，只有赋值的 double 才可以进行自带的判断相等，如果是计算拿到的 double ，使用自带的判断相等可能会把两个相同的 double 判断为不相同。

可以看到上面的代码，如果用到很多地方判断两个值就会有很多冗余的代码，而且在 Math.Abs 求绝对值计算性能是比不过判断一个大于 0 的值和一个小于 0 的值做两次判断

一个比较建议的判断两个 double 是不是相等的方法是判断两个值的大小

```csharp
        public static bool IsClose(this double value1, double value2,
            double maximumAbsoluteError = DefaultDoubleAccuracy)
        {
            if (double.IsInfinity(value1) || double.IsInfinity(value2))
            {
                return Equals(value1, value2);
            }

            if (double.IsNaN(value1) || double.IsNaN(value2))
            {
                return false;
            }

            var delta = value1 - value2;

            //return Math.Abs(delta) <= maximumAbsoluteError;

            if (delta > maximumAbsoluteError ||
                delta < -maximumAbsoluteError)
            {
                return false;
            }

            return true;
        }
```

这个方法不敢写判断相等，因为实际上有一些值是在数学上计算不相等，但是在这里判断是相等。

刚刚写了和另一个 double 判断相等，那么如何判断 double 是不是 0？虽然可以直接把 0 作为 double 判断，但是实际上这个判断是不建议的，因为有更好的方法。

在 double 计算，最小的一个单位可以让 1 加上这个值就不等于 1 的就是 `2 * 2^(-53)`，代码把这个这个值变量写为 PositiveMachineEpsilon ，使用这个 PositiveMachineEpsilon 可以判断一个 double 的大小。

判断一个 double 是 0 那么可以通过判断这个值大于 `-PositiveMachineEpsilon` 并且 小于`PositiveMachineEpsilon`

```csharp
        public static bool IsZero(this double value)
        {
            //return Math.Abs(value) <= PositiveMachineEpsilon;

            if (value > PositiveMachineEpsilon ||
                value < -PositiveMachineEpsilon)
            {
                return false;
            }

            return true;
        }
```

那么如何判断 double 是 1 ？如果有仔细看上面的代码，那么很容易就知道如何判断

```csharp
        public static bool IsOne(this double value)
        {
            var delta = value - 1D;

            //return Math.Abs(delta) <= PositiveMachineEpsilon;

            if (delta > PositiveMachineEpsilon ||
                delta < -PositiveMachineEpsilon)
            {
                return false;
            }

            return true;
        }
```

这个代码是从 https://github.com/mathnet/mathnet-numerics/blob/master/src/Numerics/Precision.cs 复制

```csharp
    /// <summary>
    /// Double 的扩展
    /// </summary>
    //SOURCE: https://github.com/mathnet/mathnet-numerics/blob/master/src/Numerics/Precision.cs
    //        https://github.com/mathnet/mathnet-numerics/blob/master/src/Numerics/Precision.Equality.cs
    //        http://referencesource.microsoft.com/#WindowsBase/Shared/MS/Internal/DoubleUtil.cs
    //        http://stackoverflow.com/questions/2411392/double-epsilon-for-equality-greater-than-less-than-less-than-or-equal-to-gre
    public static class DoubleExtensions
    {
        /// <summary>
        /// The smallest positive number that when SUBTRACTED from 1D yields a result different from 1D.
        ///
        /// This number has the following properties:
        ///     (1 - NegativeMachineEpsilon) &lt; 1 and
        ///     (1 + NegativeMachineEpsilon) == 1
        /// </summary>
        public static readonly double MeasuredNegativeMachineEpsilon = MeasureNegativeMachineEpsilon();

        /// <summary>
        /// The smallest positive number that when ADDED to 1D yields a result different from 1D.
        ///
        /// This number has the following properties:
        ///     (1 - PositiveDoublePrecision) &lt; 1 and
        ///     (1 + PositiveDoublePrecision) &gt; 1
        /// </summary>
        public static readonly double MeasuredPositiveMachineEpsilon = MeasurePositiveMachineEpsilon();


        /// <summary>
        /// The smallest positive number that when SUBTRACTED from 1D yields a result different from 1D.
        /// The value is derived from 2^(-53) = 1.1102230246251565e-16, where IEEE 754 binary64 &quot;double precision&quot; floating point numbers have a significand precision that utilize 53 bits.
        ///
        /// This number has the following properties:
        ///     (1 - NegativeMachineEpsilon) &lt; 1 and
        ///     (1 + NegativeMachineEpsilon) == 1
        /// </summary>
        public const double NegativeMachineEpsilon = 1.1102230246251565e-16D; //Math.Pow(2, -53);

        /// <summary>
        /// The smallest positive number that when ADDED to 1D yields a result different from 1D.
        /// The value is derived from 2 * 2^(-53) = 2.2204460492503131e-16, where IEEE 754 binary64 &quot;double precision&quot; floating point numbers have a significand precision that utilize 53 bits.
        ///
        /// This number has the following properties:
        ///     (1 - PositiveDoublePrecision) &lt; 1 and
        ///     (1 + PositiveDoublePrecision) &gt; 1
        /// </summary>
        public const double PositiveMachineEpsilon = 2D * NegativeMachineEpsilon;

        public static bool IsClose(this double value1, double value2,
            double maximumAbsoluteError = DefaultDoubleAccuracy)
        {
            if (double.IsInfinity(value1) || double.IsInfinity(value2))
            {
                return Equals(value1, value2);
            }

            if (double.IsNaN(value1) || double.IsNaN(value2))
            {
                return false;
            }

            var delta = value1 - value2;

            //return Math.Abs(delta) <= maximumAbsoluteError;

            if (delta > maximumAbsoluteError ||
                delta < -maximumAbsoluteError)
            {
                return false;
            }

            return true;
        }

        public static bool LessThan(this double value1, double value2)
        {
            return (value1 < value2) && !IsClose(value1, value2);
        }

        public static bool GreaterThan(this double value1, double value2)
        {
            return (value1 > value2) && !IsClose(value1, value2);
        }

        public static bool LessThanOrClose(this double value1, double value2)
        {
            return (value1 < value2) || IsClose(value1, value2);
        }

        public static bool GreaterThanOrClose(this double value1, double value2)
        {
            return (value1 > value2) || IsClose(value1, value2);
        }

        public static bool IsOne(this double value)
        {
            var delta = value - 1D;

            //return Math.Abs(delta) <= PositiveMachineEpsilon;

            if (delta > PositiveMachineEpsilon ||
                delta < -PositiveMachineEpsilon)
            {
                return false;
            }

            return true;
        }

        public static bool IsZero(this double value)
        {
            //return Math.Abs(value) <= PositiveMachineEpsilon;

            if (value > PositiveMachineEpsilon ||
                value < -PositiveMachineEpsilon)
            {
                return false;
            }

            return true;
        }

        /// <summary>
        /// 判断两个 <see cref="T:System.Double" /> 值是否近似相等。
        /// </summary>
        /// <param name="d1">值1。</param>
        /// <param name="d2">值2。</param>
        /// <param name="tolerance">近似容差。</param>
        [PublicAPI]
        public static bool NearlyEquals(double d1, double d2, double tolerance = 1E-05)
        {
            return IsClose(d1, d2, tolerance);
        }

        private static double MeasureNegativeMachineEpsilon()
        {
            var epsilon = 1D;

            do
            {
                var nextEpsilon = epsilon / 2D;

                if (NearlyEquals(1D - nextEpsilon, 1D)) //if nextEpsilon is too small
                {
                    return epsilon;
                }

                epsilon = nextEpsilon;
            } while (true);
        }

        private static double MeasurePositiveMachineEpsilon()
        {
            var epsilon = 1D;

            do
            {
                var nextEpsilon = epsilon / 2D;

                if (NearlyEquals((1D + nextEpsilon), 1D)) //if nextEpsilon is too small
                {
                    return epsilon;
                }

                epsilon = nextEpsilon;
            } while (true);
        }

        private const double DefaultDoubleAccuracy = NegativeMachineEpsilon * 10D;
    }

```

参见：

https://github.com/mathnet/mathnet-numerics/blob/master/src/Numerics/Precision.cs

https://github.com/mathnet/mathnet-numerics/blob/master/src/Numerics/Precision.Equality.cs

http://referencesource.microsoft.com/#WindowsBase/Shared/MS/Internal/DoubleUtil.cs

http://stackoverflow.com/questions/2411392/double-epsilon-for-equality-greater-than-less-than-less-than-or-equal-to-gre

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
