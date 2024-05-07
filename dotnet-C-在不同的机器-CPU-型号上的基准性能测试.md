
# dotnet C# 在不同的机器 CPU 型号上的基准性能测试

本文将记录我在多个不同的机器上，在不同的 CPU 型号上，执行相同的我编写的 dotnet 的 Benchmark 的代码，测试不同的 CPU 型号对 C# 系的优化程度。本文非严谨测试，数值只有相对意义

<!--more-->


<!-- 不发布 -->

以下是我的测试结果，对应的测试代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/287e5aee6b79fef16600eb8d76b64cf6e95eada0/BulowukaileFeanayjairwo) 上，可以在本文末尾找到下载代码的方法

我十分推荐你自己拉取代码，在你自己的设备上跑一下，测试其性能。且在开始之前，期望你已经掌握了基础的性能测试知识，避免出现诡异的结论

本文的测试将围绕着尽可能多的覆盖基础 CPU 指令以及基础逻辑行为。基础的 CPU 指令的性能测试已经有许多前辈测试过了，我这里重点测试的是各个 C# 系的上层业务行为下，所调用的多个 CPU 指令的最终性能影响。额外的也覆盖 CPU 缓存，逻辑分支命中，方法参数堆栈传递等的性能。本文的测试重点不在于 C# 系的相同功能的多个不同实现之间的性能对比，重点在于相同的代码在不同的 CPU 型号、内存、系统上的性能差异，正如此需求所述，本文非严谨测试，测试结果的数值只有相对意义

## 数组系

### 英特尔 13th Gen Intel Core i7-13700K

以下是在我开发机上跑的，我开了几百个进程，有比较多干扰，但是问题不大，因为 i7-13700K 依然性能遥遥领先。等后续找个空闲的机器，再跑一次比较准确的性能测试

```

BenchmarkDotNet v0.13.12, Windows 11 (10.0.22631.3447/23H2/2023Update/SunValley3)
13th Gen Intel Core i7-13700K, 1 CPU, 24 logical and 16 physical cores
.NET SDK 8.0.204
  [Host]     : .NET 8.0.4 (8.0.424.16909), X64 RyuJIT AVX2
  Job-AXOZTJ : .NET 8.0.4 (8.0.424.16909), X64 RyuJIT AVX2

RunStrategy=Throughput  

```
| Method                   | ArraySize | Mean             | Error          | StdDev         | Median           | Ratio | RatioSD |
|------------------------- |---------- |-----------------:|---------------:|---------------:|-----------------:|------:|--------:|
| NewArray                 | 10        |         3.873 ns |      0.1146 ns |      0.2417 ns |         3.777 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 10        |        12.234 ns |      0.2815 ns |      0.4382 ns |        12.168 ns |  3.15 |    0.21 |
| GCZeroUninitialized      | 10        |         4.470 ns |      0.1491 ns |      0.4056 ns |         4.354 ns |  1.14 |    0.13 |
| NewArrayWithRandomVisit  | 10        |        12.012 ns |      0.2679 ns |      0.2506 ns |        11.941 ns |  3.09 |    0.18 |
| NewArrayWithOrdinalVisit | 10        |         9.839 ns |      0.3379 ns |      0.9803 ns |         9.635 ns |  2.58 |    0.26 |
|                          |           |                  |                |                |                  |       |         |
| NewArray                 | 100       |        11.875 ns |      0.1932 ns |      0.2444 ns |        11.813 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 100       |        21.980 ns |      0.4524 ns |      0.8931 ns |        21.820 ns |  1.88 |    0.08 |
| GCZeroUninitialized      | 100       |        12.126 ns |      0.2769 ns |      0.5201 ns |        11.953 ns |  1.04 |    0.05 |
| NewArrayWithRandomVisit  | 100       |        47.344 ns |      0.9635 ns |      2.1351 ns |        46.572 ns |  4.03 |    0.24 |
| NewArrayWithOrdinalVisit | 100       |        75.207 ns |      1.4285 ns |      1.3363 ns |        75.364 ns |  6.33 |    0.15 |
|                          |           |                  |                |                |                  |       |         |
| NewArray                 | 1000      |       110.197 ns |      2.1602 ns |      2.0206 ns |       109.619 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 1000      |       116.560 ns |      2.0796 ns |      1.8435 ns |       116.604 ns |  1.06 |    0.03 |
| GCZeroUninitialized      | 1000      |        33.476 ns |      0.5921 ns |      0.5538 ns |        33.643 ns |  0.30 |    0.01 |
| NewArrayWithRandomVisit  | 1000      |       208.835 ns |      4.1962 ns |      8.8512 ns |       205.699 ns |  1.92 |    0.09 |
| NewArrayWithOrdinalVisit | 1000      |       620.850 ns |     11.5406 ns |     10.7951 ns |       619.304 ns |  5.64 |    0.15 |
|                          |           |                  |                |                |                  |       |         |
| NewArray                 | 10000     |       996.853 ns |     21.9389 ns |     61.8790 ns |       970.393 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 10000     |       996.704 ns |     20.8764 ns |     58.5397 ns |       974.900 ns |  1.00 |    0.08 |
| GCZeroUninitialized      | 10000     |        63.200 ns |      1.0544 ns |      0.9863 ns |        63.315 ns |  0.06 |    0.00 |
| NewArrayWithRandomVisit  | 10000     |     1,242.151 ns |     24.2642 ns |     38.4856 ns |     1,233.944 ns |  1.21 |    0.07 |
| NewArrayWithOrdinalVisit | 10000     |     6,068.245 ns |     90.8508 ns |     84.9819 ns |     6,076.727 ns |  5.79 |    0.34 |
|                          |           |                  |                |                |                  |       |         |
| NewArray                 | 100000    |     7,381.046 ns |    137.9635 ns |    147.6194 ns |     7,372.520 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 100000    |     7,214.089 ns |     85.2068 ns |     71.1515 ns |     7,209.220 ns |  0.97 |    0.02 |
| GCZeroUninitialized      | 100000    |     7,347.661 ns |    146.3643 ns |    174.2363 ns |     7,306.838 ns |  1.00 |    0.03 |
| NewArrayWithRandomVisit  | 100000    |     8,456.669 ns |    164.5726 ns |    219.6997 ns |     8,517.366 ns |  1.14 |    0.05 |
| NewArrayWithOrdinalVisit | 100000    |   129,749.709 ns |  2,408.4302 ns |  2,773.5518 ns |   128,963.159 ns | 17.57 |    0.55 |
|                          |           |                  |                |                |                  |       |         |
| NewArray                 | 1000000   |    59,752.036 ns |  1,194.7579 ns |  1,929.3113 ns |    59,414.325 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 1000000   |    60,008.303 ns |  1,188.0164 ns |  1,778.1671 ns |    59,378.000 ns |  1.01 |    0.04 |
| GCZeroUninitialized      | 1000000   |    58,868.279 ns |  1,023.4279 ns |    957.3151 ns |    58,724.731 ns |  0.97 |    0.04 |
| NewArrayWithRandomVisit  | 1000000   |    56,399.609 ns |  1,068.5479 ns |    999.5204 ns |    56,296.948 ns |  0.93 |    0.03 |
| NewArrayWithOrdinalVisit | 1000000   | 1,314,841.960 ns | 26,155.6618 ns | 27,986.2651 ns | 1,313,674.414 ns | 21.92 |    1.00 |


### 兆芯 ZHAOXIN KaiXian KX-U6780A

```

BenchmarkDotNet v0.13.12, UnionTech OS Desktop 20 E
ZHAOXIN KaiXian KX-U6780A2.7GHz (Max: 2.70GHz), 1 CPU, 8 logical and 8 physical cores
.NET SDK 8.0.204
  [Host]     : .NET 8.0.4 (8.0.424.16909), X64 RyuJIT AVX
  Job-YPUGMN : .NET 8.0.4 (8.0.424.16909), X64 RyuJIT AVX

RunStrategy=Throughput  

```

| Method                   | ArraySize | Mean            | Error         | StdDev        | Median          | Ratio | RatioSD |
|------------------------- |---------- |----------------:|--------------:|--------------:|----------------:|------:|--------:|
| NewArray                 | 10        |        40.20 ns |      0.977 ns |      1.491 ns |        39.98 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 10        |       141.12 ns |      2.996 ns |      6.051 ns |       139.67 ns |  3.54 |    0.18 |
| GCZeroUninitialized      | 10        |        48.72 ns |      0.849 ns |      0.663 ns |        48.91 ns |  1.19 |    0.05 |
| NewArrayWithRandomVisit  | 10        |       195.75 ns |      1.082 ns |      0.845 ns |       195.65 ns |  4.77 |    0.16 |
| NewArrayWithOrdinalVisit | 10        |        72.42 ns |      1.513 ns |      2.400 ns |        72.45 ns |  1.80 |    0.08 |
|                          |           |                 |               |               |                 |       |         |
| NewArray                 | 100       |       135.07 ns |      2.892 ns |      6.100 ns |       135.41 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 100       |       228.42 ns |      4.662 ns |     10.135 ns |       228.83 ns |  1.70 |    0.11 |
| GCZeroUninitialized      | 100       |       137.26 ns |      2.939 ns |      5.519 ns |       136.70 ns |  1.02 |    0.06 |
| NewArrayWithRandomVisit  | 100       |       572.02 ns |     11.660 ns |     19.157 ns |       568.34 ns |  4.26 |    0.27 |
| NewArrayWithOrdinalVisit | 100       |       467.29 ns |      9.357 ns |     13.117 ns |       464.49 ns |  3.47 |    0.21 |
|                          |           |                 |               |               |                 |       |         |
| NewArray                 | 1000      |     1,037.70 ns |     20.377 ns |     54.742 ns |     1,031.50 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 1000      |     1,127.93 ns |     22.581 ns |     59.091 ns |     1,125.79 ns |  1.09 |    0.07 |
| GCZeroUninitialized      | 1000      |       653.93 ns |      6.239 ns |      4.871 ns |       652.04 ns |  0.60 |    0.02 |
| NewArrayWithRandomVisit  | 1000      |     2,375.21 ns |     47.088 ns |    100.349 ns |     2,352.11 ns |  2.27 |    0.13 |
| NewArrayWithOrdinalVisit | 1000      |     4,474.90 ns |     87.887 ns |    107.933 ns |     4,453.16 ns |  4.19 |    0.28 |
|                          |           |                 |               |               |                 |       |         |
| NewArray                 | 10000     |     9,586.62 ns |    189.501 ns |    369.608 ns |     9,657.74 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 10000     |     9,767.26 ns |    194.643 ns |    462.590 ns |     9,811.53 ns |  1.02 |    0.07 |
| GCZeroUninitialized      | 10000     |     4,093.63 ns |     80.993 ns |    143.965 ns |     4,026.86 ns |  0.43 |    0.02 |
| NewArrayWithRandomVisit  | 10000     |    13,908.10 ns |    202.573 ns |    169.158 ns |    13,928.15 ns |  1.47 |    0.06 |
| NewArrayWithOrdinalVisit | 10000     |    43,057.16 ns |    854.132 ns |  1,495.943 ns |    42,914.21 ns |  4.50 |    0.25 |
|                          |           |                 |               |               |                 |       |         |
| NewArray                 | 100000    |    63,542.13 ns |    576.256 ns |    510.836 ns |    63,519.28 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 100000    |    66,357.64 ns |  1,312.089 ns |  2,118.779 ns |    66,043.66 ns |  1.03 |    0.03 |
| GCZeroUninitialized      | 100000    |    63,638.29 ns |  1,241.493 ns |  1,477.909 ns |    63,270.73 ns |  1.01 |    0.03 |
| NewArrayWithRandomVisit  | 100000    |    76,609.50 ns |  1,501.442 ns |  1,729.063 ns |    75,958.21 ns |  1.21 |    0.03 |
| NewArrayWithOrdinalVisit | 100000    |   665,286.65 ns |  9,295.620 ns |  7,762.264 ns |   662,915.19 ns | 10.47 |    0.16 |
|                          |           |                 |               |               |                 |       |         |
| NewArray                 | 1000000   |   461,130.99 ns |  9,000.698 ns | 10,004.252 ns |   461,306.23 ns |  1.00 |    0.00 |
| GCZeroInitialized        | 1000000   |   459,810.29 ns |  8,893.401 ns | 10,586.961 ns |   455,791.25 ns |  1.00 |    0.03 |
| GCZeroUninitialized      | 1000000   |   456,245.03 ns |  8,819.606 ns | 12,363.856 ns |   452,252.89 ns |  0.99 |    0.04 |
| NewArrayWithRandomVisit  | 1000000   |   497,132.01 ns |  9,841.562 ns | 12,796.810 ns |   490,990.22 ns |  1.08 |    0.03 |
| NewArrayWithOrdinalVisit | 1000000   | 6,742,537.03 ns | 48,986.470 ns | 38,245.414 ns | 6,732,321.64 ns | 14.51 |    0.31 |



本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/287e5aee6b79fef16600eb8d76b64cf6e95eada0/BulowukaileFeanayjairwo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/287e5aee6b79fef16600eb8d76b64cf6e95eada0/BulowukaileFeanayjairwo) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 287e5aee6b79fef16600eb8d76b64cf6e95eada0
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 287e5aee6b79fef16600eb8d76b64cf6e95eada0
```

获取代码之后，进入 BulowukaileFeanayjairwo 文件夹，即可获取到源代码

## 特别感谢

特别感谢 <https://github.com/mjebrahimi/Performance-Wars-Benchmarking-CSharp> 提供的代码

## 参考文档

[C# 标准性能测试](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html )

[C# 标准性能测试高级用法](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.html )

[dotnet 6 数组拷贝性能对比](https://blog.lindexi.com/post/dotnet-6-%E6%95%B0%E7%BB%84%E6%8B%B7%E8%B4%9D%E6%80%A7%E8%83%BD%E5%AF%B9%E6%AF%94.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。