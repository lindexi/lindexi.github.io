---
title: dotnet 对一些 Win32 方法进行 Benchmark 基准性能测试
description: 本文记录对一些 Win32 方法进行 Benchmark 基准性能测试
tags: dotnet
category: 
---

<!-- CreateTime:2025/01/16 07:13:37 -->

<!-- 发布 -->
<!-- 博客 -->

本文非严谨测试，仅在我开发机器进行测试，没有在纯净系统和机器上进行测试

开始之前的说明：

本文使用的是 BenchmarkDotNet 进行测试，没有考虑 AOT 之后的调用性能，仅仅只是 Release 版本的 dotnet 程序的调用而已

数值单位说明：

1,000,000 纳秒（ns） = 1毫秒 (ms)

## GetDoubleClickTime

性能： 396.2 ns

BenchmarkDotNet v0.14.0, Windows 11 (10.0.22631.4391/23H2/2023Update/SunValley3)
13th Gen Intel Core i7-13700K, 1 CPU, 24 logical and 16 physical cores
.NET SDK 9.0.100
  [Host]     : .NET 9.0.0 (9.0.24.52809), X64 RyuJIT AVX2
  DefaultJob : .NET 9.0.0 (9.0.24.52809), X64 RyuJIT AVX2


| Method | Mean     | Error    | StdDev   |
|------- |---------:|---------:|---------:|
| Test   | 396.2 ns | 14.32 ns | 41.53 ns |

测试代码如下

```csharp
public class GetDoubleClickTimeTest
{
    [Benchmark]
    public void Test()
    {
        _ = GetDoubleClickTime();
    }

    public const string LibraryName = "user32";

    /// <summary>
    /// 获取鼠标双击事件两次点击的时间间隔
    /// </summary>
    /// <returns></returns>
    [DllImport(LibraryName)]
    public static extern uint GetDoubleClickTime();
}
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/02ae29a86adb257c285046fc1e27a5539cf8bbb3/Workbench/CofacelceawurDemharawjo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/02ae29a86adb257c285046fc1e27a5539cf8bbb3/Workbench/CofacelceawurDemharawjo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 02ae29a86adb257c285046fc1e27a5539cf8bbb3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 02ae29a86adb257c285046fc1e27a5539cf8bbb3
```

获取代码之后，进入 Workbench/CofacelceawurDemharawjo 文件夹，即可获取到源代码

## GetSystemMetrics

测试内容： 测试获取双击尺寸范围，双击的误差范围

性能：30.09 ns

BenchmarkDotNet v0.14.0, Windows 11 (10.0.22631.4391/23H2/2023Update/SunValley3)
13th Gen Intel Core i7-13700K, 1 CPU, 24 logical and 16 physical cores
.NET SDK 9.0.100
  [Host]     : .NET 9.0.0 (9.0.24.52809), X64 RyuJIT AVX2
  DefaultJob : .NET 9.0.0 (9.0.24.52809), X64 RyuJIT AVX2


| Method | Mean     | Error    | StdDev   |
|------- |---------:|---------:|---------:|
| Test   | 30.09 ns | 0.035 ns | 0.031 ns |


测试代码：

```csharp
public class GetSystemMetricsTest
{
    [Benchmark]
    public (int X, int Y) Test()
    {
        var x = GetSystemMetrics(SystemMetric.SM_CXDOUBLECLK);
        var y = GetSystemMetrics(SystemMetric.SM_CYDOUBLECLK);
        return (x, y);
    }

    public const string LibraryName = "user32";

    /// <summary>
    /// 获取系统度量值
    /// </summary>
    /// <param name="smIndex"></param>
    /// <returns></returns>
    [DllImport(LibraryName)]
    private static extern int GetSystemMetrics(SystemMetric smIndex);

    internal enum SystemMetric
    {
        SM_CXDOUBLECLK = 36, // 0x24
        SM_CYDOUBLECLK = 37, // 0x25
    }
}
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/02ae29a86adb257c285046fc1e27a5539cf8bbb3/Workbench/CofacelceawurDemharawjo) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/02ae29a86adb257c285046fc1e27a5539cf8bbb3/Workbench/CofacelceawurDemharawjo) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 02ae29a86adb257c285046fc1e27a5539cf8bbb3
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 02ae29a86adb257c285046fc1e27a5539cf8bbb3
```

获取代码之后，进入 Workbench/CofacelceawurDemharawjo 文件夹，即可获取到源代码

## 相关博客

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

[dotnet C# 在不同的机器 CPU 型号上的基准性能测试](https://blog.lindexi.com/post/dotnet-C-%E5%9C%A8%E4%B8%8D%E5%90%8C%E7%9A%84%E6%9C%BA%E5%99%A8-CPU-%E5%9E%8B%E5%8F%B7%E4%B8%8A%E7%9A%84%E5%9F%BA%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html )

[C# 标准性能测试](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95.html )

[C# 标准性能测试高级用法](https://blog.lindexi.com/post/C-%E6%A0%87%E5%87%86%E6%80%A7%E8%83%BD%E6%B5%8B%E8%AF%95%E9%AB%98%E7%BA%A7%E7%94%A8%E6%B3%95.html )
