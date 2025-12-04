# C# 浮点数 double 的 IsFinite IsNormal IsRealNumber 分不清楚

在 dotnet 的 double 或 float 浮点数里有很多个辅助方法用于判断状态，如是否无效值或无穷。这些方法含义不相同，让我比较混淆，于是我就尝试记录一篇博客说清楚这些方法的含义

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

## IsFinite

是否有限数，包括 0 、subnormal次正规数、normal正规数。不包括 NaN 无效数、正负无穷

> normal 和 subnormal 是专业计算机浮点数相关术语，表示正规数和次正规数。正规数就是咱程序猿比较符合直觉的浮点数，大概就是 `xxx.xxx` 格式的数值。细节定义还请自行参阅文档，总之按照你的直觉感觉是正常的非零数值的，就是正规数了
>
> 次正规数是与正规数相关的概念，其引入只是为了防止浮点数在比较接近 0 的时候，快速跌向零而已。当结果非常小，已经小到正常浮点编码的指数再也不能减小时，IEEE 754 不直接把结果变成零，而是进入一个“更小但仍非零”的区间（次正规数）。这样数值从“最小正正常数”到 0 的过程是连续渐进的，而不是突然跳到零。简单科普就到此，在后文会给出更多细节说明

计算结果如下

```
IsFinite 是否有限数：
       1.0 : True
         0 : True
        -0 : True
  4.0E-320 : True
       NaN : False
         ∞ : False
        -∞ : False
   Epsilon : True
    Math.E : True
```

可见只有正负无穷和 NaN 为 false 值

等同于非 NaN 和非无穷

## IsRealNumber

是否实数。其定义是在复数域 `a + bi` 里面，当其中的 `b` 为 0 时的数。复数域是一个数学名词，如果读者没有学到的话，还请跳过这一部分

按照此定义来说，正负无穷都在其定义的实数范围内，自然判断就是 True 值。注： 数学上，正负无穷是否在实数范围是有争议的。仅在上述定义里，正负无穷才在范围内

计算结果如下

```
IsRealNumber 是否实数：
       1.0 : True
         0 : True
        -0 : True
  4.0E-320 : True
       NaN : False
         ∞ : True
        -∞ : True
   Epsilon : True
    Math.E : True
```

可见，只有 NaN 为 false 值，其他测试内容都是返回 true 值

## IsNaN

如方法名，判断传入的是 NaN 无效数，计算结果如下

```
IsNaN：
       1.0 : False
         0 : False
        -0 : False
  4.0E-320 : False
       NaN : True
         ∞ : False
        -∞ : False
   Epsilon : False
    Math.E : False
```

此时就可以看到，当传入 NaN 时，以上三个方法 IsNaN IsRealNumber IsFinite 都能将其区分开来。其实本质含义不相同。请大家根据自己的业务逻辑选择正确的方法

## IsNormal

是否正规数。在开始介绍正规数之前，需要继续聊一下次正规数

### 次正规数

为什么要有次正规数呢？因为如果没有次正规数，很多运算一旦到达最小正（+） 正常数（正规数）再往小，就会直接变 零。这样会造成算法的非连续性：例如一个递减迭代量本来应当继续减小，但受限于浮点数精度从而在数值表示层面突然消失为零

按照 IEEE 754 规范，可知 double 的 64 个 bit 的划分如下：

```
sign(1) | exponent(11) | fraction(52)
```

二进制表示上的具体差别如下：

Normal 正规数：

- exponent 位至少有一个 1（范围 00000000001 到 11111111110）
- fraction 任意（可为 0），但实际计算时隐含 1.xxx
- 最小正 normal 的 exponent 是 00000000001（=1），fraction 为 0；对应值约 2.2250738585072014E-308
- 数值公式：value = (-1)^sign × 2^(exponent - bias) × (1 + fraction/2^52)

Subnormal 次正规数：

- exponent 全 0（00000000000）
- fraction 非 0（否则就是 0）
- 没有隐含 1，精度更低；最小正可表示数是 fraction 仅最低位为 1，对应 double.Epsilon ≈ 4.94065645841247E-324
- 数值公式：value = (-1)^sign × 2^(1 - bias) × (fraction/2^52)

特殊全 1 指数（11111111111）用于 NaN/Infinity，不在 normal/subnormal 范畴。细节请参阅 [C# dotnet 在内存中的 double 的 NAN 和正负无穷二进制是如何存](https://blog.lindexi.com/post/C-dotnet-%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84-double-%E7%9A%84-NAN-%E5%92%8C%E6%AD%A3%E8%B4%9F%E6%97%A0%E7%A9%B7%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%98%AF%E5%A6%82%E4%BD%95%E5%AD%98.html )

其中 bias（指数偏置）= 1023。

简单认为在 C# .NET 里面，最小正正常数（正规数）约 2.2250738585072014E-308，此后再变小会进入次正规区间，•	次正规区间的最小正值是 double.Epsilon ≈ 4.94E-324

### 计算结果

计算结果如下

```
IsNormal 是否正规数：
       1.0 : True
         0 : False
        -0 : False
  4.0E-320 : False
       NaN : False
         ∞ : False
        -∞ : False
   Epsilon : False
    Math.E : True

IsSubnormal 是否次正规数：
       1.0 : False
         0 : False
        -0 : False
  4.0E-320 : True
       NaN : False
         ∞ : False
        -∞ : False
   Epsilon : True
    Math.E : False
```

可见 IsNormal 和 IsSubnormal 都不包含 0 和无穷和无效数

## FAQ

能否用 IsNormal 判断给定数值是否有效？

不可。如果这里的有效定义是是否 NaN 无效数的反面，请使用对 IsNaN 进行取反

如果是在 UI 框架里面，如 WPF 或 Avalonia 或 MAUI 等框架里，表示一个合法有限的元素尺寸，应该用哪个方法？

取决于业务，绝大部分情况下，应该使用的是 IsFinite 方法。因为 IsFinite 方法等同于非 NaN 和非无穷

什么时候需要判断次正规数？

很少有此需求。我能想到的是为了提升性能，对于很多 CPU 来说，计算次正规数需要进入慢分支，判断进入了次正规数就直接赋零，可以提升一些计算性能

## 代码

以下是我的测试代码

```csharp
Console.WriteLine($"IsNormal 是否正规数：");

foreach (var (name, value) in GetTestValues())
{
    Console.WriteLine($"{name,10} : {double.IsNormal(value)}");
}

Console.WriteLine($"IsSubnormal 是否次正规数：");

foreach (var (name, value) in GetTestValues())
{
    Console.WriteLine($"{name,10} : {double.IsSubnormal(value)}");
}

Console.WriteLine($"IsNaN：");

foreach (var (name, value) in GetTestValues())
{
    Console.WriteLine($"{name,10} : {double.IsNaN(value)}");
}

Console.WriteLine($"IsRealNumber 是否实数：");

foreach (var (name, value) in GetTestValues())
{
    Console.WriteLine($"{name,10} : {double.IsRealNumber(value)}");
}

Console.WriteLine($"IsFinite 是否有限数：");

foreach (var (name, value) in GetTestValues())
{
    Console.WriteLine($"{name,10} : {double.IsFinite(value)}");
}


IEnumerable<(string Name, double Value)> GetTestValues()
{
    yield return ("1.0", 1.0);
    yield return ("0", 0);
    yield return ("-0", double.NegativeZero);
    yield return ("4.0E-320", 4.0E-320);
    yield return ("NaN", double.NaN);
    yield return ("∞", double.PositiveInfinity);
    yield return ("-∞", double.NegativeInfinity);
    yield return ("Epsilon", double.Epsilon);
    yield return ("Math.E", double.E);
}
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/c45802df550bdcb19c3c4fa038bc736b716d5d64/Workbench/CheracefairCawweekewhairjere) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/c45802df550bdcb19c3c4fa038bc736b716d5d64/Workbench/CheracefairCawweekewhairjere) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin c45802df550bdcb19c3c4fa038bc736b716d5d64
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin c45802df550bdcb19c3c4fa038bc736b716d5d64
```

获取代码之后，进入 Workbench/CheracefairCawweekewhairjere 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )