
# dotnet 根据基线包版本实现库版本兼容

本文来告诉大家如何根据 基线包版本 的功能来实现自动在构建过程中，告诉开发者，当前版本是否存在不兼容旧版本的变更。其不兼容变更包括二进制中断变更和 API 不兼容变更和源代码中断变更。可以让库开发者花更少的精力在测试兼容性上

<!--more-->


<!-- 发布 -->

今天看到了队长推送的 [.NET 6新特性试用 Nuget包验证](https://mp.weixin.qq.com/s/-J2VQEw9pSYy1tFRdTP2ug) 博客，才回忆起此功能。这个功能是给库和框架开发者使用的，用于处理多版本兼容性问题

## 背景

只有对一个库或框架准备对外发布且长期维护，以及期望给其他开发者使用时，才需要考虑库或框架的兼容性问题。越是开发底层的库，兼容性问题就越加重要。此重要性，只有自己参与开发，踩够坑之后，才能有所体会

换句话说，判断一位开发者是不是库或框架的老司机开发者，可以通过他的兼容性处理上来看出。哈哈，需要说明的是，不是所有老司机开发者都是库或框架开发方向的，这是判断有经验的开发者的充分不必要条件

开始之前，先聊聊什么是兼容性问题。兼容性可以分为以下不兼容变更：

- 源代码中断变更和 API 不兼容变更：简单说 API 不兼容变更，就是更改了开放出去的 API 签名。对于使用了此库或框架的开发者来说，如果更新到新的版本，为了适配变更，就 必须 更改源代码
- 二进制中断变更：尽管是不用更改源代码就能适配新版本，但是如果没有重新构建，提示替换 DLL 文件，那将会在运行程序时挂掉。例如给某个公开的函数加上了一个默认参数，尽管默认参数的添加，在源代码上是可以不做任何变更就可以用上新的版本，然而如果没有重新构建，只是将新版本的 DLL 或 EXE 替换过去，在运行的时候将提示找不到方法
- 行为中断变更：某个行为被更改，执行逻辑和之前不兼容。例如原本一个方法能好好工作，现在调用了，进程就退出了等等

此外，还有更换了底层运行时框架的变更等，但这些就不在本文讨论范围了

更多请参阅官方文档的详细描述： [重大更改和 .NET 库 Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/standard/library-guidance/breaking-changes#source-breaking-change?WT.mc_id=WD-MVP-5003260)

对于使用库或框架的开发者来说，一方面又期望用上新版本的强大功能，另一方面又怕有不兼容的变更，需要花费大量的精力在更新上面。如果库或框架的开发者，可以保持好兼容性，那么升级版本是一个很轻松的事情

对于咱 dotnet 系的大部分库或框架开发者来说，在开发过程中，考虑兼容性是一个必备的选项。那如果真的需要变更 API 了呢？问题也不大，别忘了咱还有版本号规则

## 版本号规则

基本所有 dotnet 系上，正经的库和框架都会遵循约定的版本号规则，从而让开发者在使用任何库的时候，通过版本号都能明确其中的含义，决定自己是否应该升级到最新版本

无异议的版本规则是，版本号由四个部分组成，分为 `主版本号.次版本号.构建号.修订号` 四个部分。其中的 构建号 和 修订号 都可忽略不写。各个部分的含义如下

<!-- 无异议的版本规则是，版本号由 2 到 4 个版本个数组成，格式如 `a.b.c.d` 的记录方式，如 `1.2.3.4` 版本。其中最短的版本号是仅带上前面两个，后面两个不写，不写的默认值是 0 的值。以下是各个版本数字的含义： -->

- 主版本号： major version ， 此版本如有变更，如从 1 升级到 2 的版本，代表着有重大更改。如存在不兼容的 API 或源代码更改，或者机制性，或者行为上的变更。大部分情况下，有主版本的变更就意味着需要在升级完成进行适配的工作
- 次版本号:  minor version，此版本如有变更，代表着有新增的 API 定义或者是较大的但是兼容的修订，如修大 Bug 等，大部分情况下是不需要进行任何的适配工作
- 构建号: build number，此版本如有变更，代表着有小的更改，如修 Bug 等，不改变对外公开的约定的行为。升级新版本不需要进行任何的适配工作
- 修订号: revision，此版本大部分情况是给构建工具链编写的，开发者人类是很少需要变更此。升级到此新版本，无须进行任何适配

此外，有一些库毕竟激进，需要发布预览版本等，可以考虑采用语义版本号的方法，请看 [语义版本号（Semantic Versioning） - walterlv - 博客园](https://www.cnblogs.com/walterlv/p/10236470.html)

通过如上的说明，可以了解到，如果不想刷主版本号，那就要求库或框架保持兼容旧版本。兼容旧版本需要在开发时，投入精力了解是否存在不兼容的更改，然而纯依靠手动去阅读代码了解是否存在不兼容的变更，当然是不靠谱的。本文将告诉大家如何使用 `EnablePackageValidation` 和 `PackageValidationBaselineVersion` 功能，自动让构建工具告诉开发者当前的更改是否存在不兼容的更改，从而更好保持库或框架的兼容

## 使用方法

一如既往的简单，只需要在项目文件上，添加如下代码即可

```xml
    <EnablePackageValidation>true</EnablePackageValidation>
    <PackageValidationBaselineVersion>基于的版本号</PackageValidationBaselineVersion>
```

例如当前是 2.0.0 的版本，期望进行对 1.0.0 包版本的兼容性测试，可以将 PackageValidationBaselineVersion 的值更改为 1.0.0 版本，如下面代码

```xml
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <PackageVersion>2.0.0</PackageVersion>
    <EnablePackageValidation>true</EnablePackageValidation>
    <PackageValidationBaselineVersion>1.0.0</PackageValidationBaselineVersion>
  </PropertyGroup>
```

如此，在存在中断性（也就是不兼容，需要代码适配）变更时，在会在构建时给出提示，同时让构建不通过

## 例子

如何更好的使用此功能，还请让我用一个例子来告诉大家。此例子完全从 [官方文档](https://docs.microsoft.com/zh-cn/dotnet/fundamentals/package-validation/baseline-version-validator) 抄的

在第一个版本时，作为 1.0.2 的版本的 NuGet 包，已对外发布。在进行 1.1.0 版本开发时，期望能做到完全的兼容第一个版本。利用 PackageValidationBaselineVersion 的功能，在 csproj 项目文件上，加上如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <PackageVersion>1.1.0</PackageVersion>
    <EnablePackageValidation>true</EnablePackageValidation>
    <PackageValidationBaselineVersion>1.0.2</PackageValidationBaselineVersion>
  </PropertyGroup>

</Project>
```

通过在 PackageValidationBaselineVersion 执行定了基线包版本为 1.0.2 即可采用此指定的版本进行基线包版本对比。例如几周后，你的任务是为库添加对连接超时的支持，代码的 `Connect` 方法目前如下所示：

```csharp
public static HttpClient Connect(string url)
{
    // ...
}
```

由于连接超时是一个高级配置设置，因此你认为可以添加一个可选参数，更改如下：

```csharp
public static HttpClient Connect(string url, TimeSpan timeout = default)
{
    // ...
}
```

更改之后，构建过程可以正常，但是在打包的时候，将会收到如下提示，打包失败

```
D:\demo>dotnet pack
Microsoft (R) Build Engine version 17.0.0-preview-21460-01+8f208e609 for .NET
Copyright (C) Microsoft Corporation. All rights reserved.

  Determining projects to restore...
  All projects are up-to-date for restore.
  You are using a preview version of .NET. See: https://aka.ms/dotnet-core-preview
  PackageValidationThrough -> D:\demo\bin\Debug\net6.0\PackageValidationThrough.dll
  Successfully created package 'D:\demo\bin\Debug\PackageValidationThrough.2.0.0.nupkg'.
C:\Program Files\dotnet\sdk\6.0.100-rc.1.21463.6\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Compatibility.Common.targets(32,5): error CP0002: Member 'A.B.Connect(string)' exists on [Baseline] lib/net6.0/PackageValidationThrough.dll but not on lib/net6.0/PackageValidationThrough.dll [D:\demo\PackageValidationThrough.csproj]
```

或者中文版本的提示如下

```
用于 .NET 的 Microsoft (R) 生成引擎版本 17.0.0-preview-21501-01+bbcce1dff
版权所有(C) Microsoft Corporation。保留所有权利。

  正在确定要还原的项目…
  所有项目均是最新的，无法还原。
  你正在使用 .NET 的预览版。请查看 https://aka.ms/dotnet-core-preview
  NallcearreyiHernareferkear -> C:\lindexi\NallcearreyiHernareferkear\NallcearreyiHernareferkear\bin\Debug\net6.0\NallcearreyiHernareferkear.dll
  已成功创建包“C:\lindexi\NallcearreyiHernareferkear\NallcearreyiHernareferkear\bin\Debug\NallcearreyiHernareferkear.2.0.0.nupkg”。
C:\Program Files\dotnet\sdk\6.0.100-rc.2.21505.57\Sdks\Microsoft.NET.Sdk\targets\Microsoft.NET.Compatibility.Common.targets(32,5): error CP0002: Member 'NallcearreyiHernareferkear.Foo.Connect(string)' exists on [Baseline] lib/net6.0/NallcearreyiHernareferkear.dll but not on lib/net6.0/NallcearreyiHernareferkear.dll [C:\lindexi\NallcearreyiHernareferkear\NallcearreyiHernareferkear\NallcearreyiHernareferkear.csproj]
```

如此通过打包失败，提示的 CP0002 失败，可以了解到，自己没有做到让当前版本对写入到 PackageValidationBaselineVersion 的兼容。此时要做的事情，要么是废弃掉对 PackageValidationBaselineVersion 的兼容，也就是删除此属性，同时升级主版本号，告诉其他开发者，当前版本存在不兼容。要么是更改 API 定义，更改到兼容

例如以上的代码，虽然加上了一个默认参数，可以实现到源代码兼容。但是大家都知道，这是二进制不兼容的，如果直接替换 DLL 文件，而不经过编译，将会在运行的过程中，因为找不到对应的方法而失败

什么情况下会遇到没有重新构建，只是替换 DLL 文件而已？在于是其他底层库的依赖引用，例如再有另一个库 C 也引用了此，而库 C 打出的 NuGet 包被最终项目所引用。当最终项目升级版本时，由于 Connect 方法被更改，从而让库 C 里面的对应逻辑找不到方法，而在运行时失败

因此为了做到这部分的兼容，可以考虑作为重载的方法更改，更改如下

```csharp
public static HttpClient Connect(string url)
{
    return Connect(url, Timeout.InfiniteTimeSpan);
}

public static HttpClient Connect(string url, TimeSpan timeout)
{
    // ...
}
```

这样进行重新打包，即可看到打包成功，兼容 PackageValidationBaselineVersion 的 1.0.2 版本

## 原理

此功能是依托于 NuGet 包发布而拿到指定版本号规则的，和 [使用基于 Roslyn 的 Microsoft.CodeAnalysis.PublicApiAnalyzers 来追踪项目的 API 改动，帮助保持库的 API 兼容性 - walterlv](https://blog.walterlv.com/post/track-api-changes-using-roslyn-public-api-analyzers.html) 的方法是完全不相同的

本文介绍的方法，是在 PackageValidationBaselineVersion 里面，声明的包版本，在构建过程中，通过 NuGet 去拉取对应的版本，接着通过 DLL 导出类型的对比，从而了解是否存在不兼容的变更

也就是说在 PackageValidationBaselineVersion 里面写入的版本号，要求是可以在 NuGet 源里面（无论是 nuget.org 源，还是你的私有的源，还是你的本机文件夹都可以）拉到对应的版本。由此版本里面的 DLL 执行具体的对比逻辑。这也就要求了此功能只能用在简单的 NuGet 上，对于很多上了黑科技的 NuGet 包是无法执行的。例如使用 [SourceYard](https://github.com/dotnet-campus/SourceYard/) 打包的源代码包

本文介绍的方法，对比[使用基于 Roslyn 的 Microsoft.CodeAnalysis.PublicApiAnalyzers 来追踪项目的 API 改动，帮助保持库的 API 兼容性](https://blog.walterlv.com/post/track-api-changes-using-roslyn-public-api-analyzers.html) 的方法来说，优势在于不需要带上 `PublicAPI.Unshipped.txt` 和 `PublicAPI.Shipped.txt` 文件，此两个文件夹特别好在团队开发时进行冲突，而且需要进行手动管理。但是缺点在于本文介绍的方法功能单一，也依赖 NuGet 包版本

## 代码

本文以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/95692dcaabfb0d143dffa8e31c0a1ad00e7c2e74/NallcearreyiHernareferkear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/95692dcaabfb0d143dffa8e31c0a1ad00e7c2e74/NallcearreyiHernareferkear) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 95692dcaabfb0d143dffa8e31c0a1ad00e7c2e74
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 NallcearreyiHernareferkear 文件夹

## 更多阅读

[听龙华讲公共组件 CBB 建设笔记](https://blog.lindexi.com/post/%E5%90%AC%E9%BE%99%E5%8D%8E%E8%AE%B2%E5%85%AC%E5%85%B1%E7%BB%84%E4%BB%B6-CBB-%E5%BB%BA%E8%AE%BE%E7%AC%94%E8%AE%B0.html)

[创建CBB心得](https://blog.lindexi.com/post/%E5%88%9B%E5%BB%BACBB%E5%BF%83%E5%BE%97.html)

[dotnet CBB 为什么决定推送 Tag 才能打包](https://blog.lindexi.com/post/dotnet-CBB-%E4%B8%BA%E4%BB%80%E4%B9%88%E5%86%B3%E5%AE%9A%E6%8E%A8%E9%80%81-Tag-%E6%89%8D%E8%83%BD%E6%89%93%E5%8C%85.html)

[开源公共组件仓库的更新日志应该如何写](https://blog.lindexi.com/post/%E5%BC%80%E6%BA%90%E5%85%AC%E5%85%B1%E7%BB%84%E4%BB%B6%E4%BB%93%E5%BA%93%E7%9A%84%E6%9B%B4%E6%96%B0%E6%97%A5%E5%BF%97%E5%BA%94%E8%AF%A5%E5%A6%82%E4%BD%95%E5%86%99.html)

[dotnet 使用 Obsolete 特性标记成员过时保持库和框架的兼容性](https://blog.lindexi.com/post/dotnet-%E4%BD%BF%E7%94%A8-Obsolete-%E7%89%B9%E6%80%A7%E6%A0%87%E8%AE%B0%E6%88%90%E5%91%98%E8%BF%87%E6%97%B6%E4%BF%9D%E6%8C%81%E5%BA%93%E5%92%8C%E6%A1%86%E6%9E%B6%E7%9A%84%E5%85%BC%E5%AE%B9%E6%80%A7.html)

[语义版本号（Semantic Versioning） - walterlv - 博客园](https://www.cnblogs.com/walterlv/p/10236470.html)

[使用基于 Roslyn 的 Microsoft.CodeAnalysis.PublicApiAnalyzers 来追踪项目的 API 改动，帮助保持库的 API 兼容性 - walterlv](https://blog.walterlv.com/post/track-api-changes-using-roslyn-public-api-analyzers.html)

[重大更改和 .NET 库 Microsoft Docs](https://docs.microsoft.com/zh-cn/dotnet/standard/library-guidance/breaking-changes#source-breaking-change?WT.mc_id=WD-MVP-5003260)

[Assembly versioning Microsoft Docs](https://docs.microsoft.com/en-us/dotnet/standard/assembly/versioning?WT.mc_id=WD-MVP-5003260)





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。