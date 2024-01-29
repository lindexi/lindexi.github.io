以往的单元测试都是不能单独作为一个独立的可执行文件跑的，需要在 VisualStudio 或 VSTest 或 dotnet test 里面运行。这就限制了运行单元测试的环境了，有时候开发者可能期望在无 SDK 或开发环境下执行单元测试，这时就可以用到本文介绍的 MSTestRunner 功能，将单元测试制作为独立可执行文件

<!--more-->


<!-- CreateTime:2024/1/27 9:59:41 -->

<!-- 发布 -->
<!-- 博客 -->

将单元测试制作为可执行文件，运行此可执行文件即可运行单元测试。可执行文件可以作为独立框架发布的方式，如此可以在一些纯净的环境里面运行，或者是将单元测试打包分发给测试同事，请测试同事在大批量的设备上进行执行。如此可见，将单元测试制作为可执行文件在许多地方都有用武之地，如：

- 在纯净系统环境运行
- 在大批量设备统一执行
- 允许外置工具调试单元测试执行情况

以上的 “允许外置工具调试单元测试执行情况” 一般指的是现有的性能调试工具，现有的性能调试工具基本都对独立应用程序支持的非常好，通过将单元测试制作为独立可执行文件可以更加方便与现有的性能调试工具进行对接

以下将和大家介绍如何利用 MSTestRunner 的功能，将单元测试制作为独立可执行文件

在开始之前，先建立一个简单的控制台项目。按照 dotnet 的惯例，先安装上 MSTest 这个 NuGet 库，可以编辑 csproj 文件，添加以下代码用来快速安装

```xml
  <ItemGroup>
    <PackageReference Include="MSTest" Version="3.2.0" />
  </ItemGroup>
```

为了让项目最终构建出来的是一个可执行文件，这里需要确保项目的输出类型是 Exe 类型，如下面代码

```xml
    <OutputType>Exe</OutputType>
```

再添加最关键的一句配置属性，即开启 MSTestRunner 的功能。配置此属性之后，且删除项目原本的 Program 文件，即可构建出测试项目的可执行文件

```xml
  <!-- 用 MSTest 测试运行器 -->
  <EnableMSTestRunner>true</EnableMSTestRunner>
```

编辑完成的 csproj 项目文件代码大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <PublishAot>true</PublishAot>
    <InvariantGlobalization>true</InvariantGlobalization>

    <!-- 用 MSTest 测试运行器 -->
    <EnableMSTestRunner>true</EnableMSTestRunner>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="MSTest" Version="3.2.0" />
  </ItemGroup>

</Project>
```

接着咱开始编写一个简单的单元测试用来测试一下此方式的行为

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace BearcenikoriDajebeqehe;

[TestClass]
public class TestClass
{
    [TestMethod]
    public void Foo()
    {
        var a = 1;
        a++;
        Assert.AreEqual(2, a);
    }
}
```

尝试构建项目，然后直接运行 exe 文件，大概就可以看到以下输出内容

```
Passed! - Failed: 0, Passed: 1, Skipped: 0, Total: 1, Duration: 289ms - BearcenikoriDajebeqehe.exe
```

可以使用命令行将其进行独立发布，如下面的命令行代码，将发布在 Linux 上的独立框架的可执行文件

```
dotnet publish -c release -r linux-x64 --self-contained true
```

以上发布内容可以在 Linux 上运行，本文这里通过 [SyncTool](https://github.com/dotnet-campus/dotnetcampus.DotNETBuildSDK/tree/master/SyncTool) 将输出内容同步到我的 UOS（统信国产 Linux 系统）系统上运行，运行结果界面如下图

<!-- ![](image/dotnet 使用 MSTestRunner 将单元测试制作为独立可执行文件/dotnet 使用 MSTestRunner 将单元测试制作为独立可执行文件0.png) -->
![](http://image.acmx.xyz/lindexi%2F20241271024387587.jpg)

这时就体现出这个功能的方便性起来了，原本我的 UOS（统信国产 Linux 系统）系统是不带任何的开发环境的，且在上面的开发体验现在还是不如在 Windows 上熟悉和舒服的。通过将单元测试构建为独立可执行文件，我就可以在 UOS 上只做测试的活，不参与具体的开发。将单元测试构建出来的可执行文件归档起来，通过单元测试可以更好的批量的测试其系统版本之间的行为差异。更多关于国产 UOS 的开发，欢迎加入 810052083 群讨论

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/14c457e0d9933ba10e5eaf3873384bb3b9a0c26d/BearcenikoriDajebeqehe) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/14c457e0d9933ba10e5eaf3873384bb3b9a0c26d/BearcenikoriDajebeqehe) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 14c457e0d9933ba10e5eaf3873384bb3b9a0c26d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 14c457e0d9933ba10e5eaf3873384bb3b9a0c26d
```

获取代码之后，进入 BearcenikoriDajebeqehe 文件夹

如果一个单元测试项目里面包含了多个单元测试方法，在做独立的可执行文件时，期望运行过程中只运行里面的部分方法，可以和原本的 vstest 或 dotnet test 一样添加过滤条件，也就是在执行时添加 `--filter` 参数和对应的条件内容，用法和 dotnet test 的用法相同，只是将命令行前面的 `dotnet test` 换成最终输出的可执行文件

详细请参阅 [Run selected unit tests - .NET Microsoft Learn](https://learn.microsoft.com/en-us/dotnet/core/testing/selective-unit-tests?pivots=mstest )
