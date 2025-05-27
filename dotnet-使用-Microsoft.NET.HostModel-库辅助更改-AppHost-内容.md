
# dotnet 使用 Microsoft.NET.HostModel 库辅助更改 AppHost 内容

本文将和大家介绍如何使用 Microsoft.NET.HostModel 库辅助更改 AppHost 内容

<!--more-->


<!-- CreateTime:2025/05/27 07:07:29 -->

<!-- 发布 -->
<!-- 博客 -->

按照 dotnet 的惯例，先在 NuGet 上安装 Microsoft.NET.HostModel 库，安装之后的 csproj 项目文件内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.HostModel" Version="3.1.16" />
  </ItemGroup>

</Project>
```

提前准备好 AppHost 模版文件，可从 .NET SDK 里面拷贝，也可以从 NuGet 缓存里面找找。如 .NET x64 的在 `C:\Program Files\dotnet\sdk\<SDK VERSION>\AppHostTemplate\apphost.exe` 路径下

比较特殊的是龙芯的版本，要么下一个大 SDK 去 AppHostTemplate 文件夹找找，要么去下 apphost 包。龙芯[旧世界](https://areweloongyet.com/docs/old-and-new-worlds/ )的 dotnet 下载地址： <https://ftp.loongnix.cn/dotnet>

完成准备工作之后，即可调用 HostWriter.CreateAppHost 方法进行创建入口 exe 文件，代码如下

```csharp
using Microsoft.NET.HostModel.AppHost;

var appHost = @"C:\Program Files\dotnet\sdk\9.0.203\AppHostTemplate\apphost.exe";

HostWriter.CreateAppHost
(
    // AppHost 可从 .NET SDK 里面拷贝，也可以从 NuGet 缓存里面找找
    // 如 .NET x64 的在 C:\Program Files\dotnet\sdk\<SDK VERSION>\AppHostTemplate\apphost.exe
    appHostSourceFilePath: appHost,
    // 输出路径，包括指定输出文件名
    appHostDestinationFilePath: "ConsoleApp1.exe",
    // 入口的 DLL 是哪一个，这是相对于 exe 所在的 dll 路径
    appBinaryFilePath: @"Foo\ReagalljaqewhurNiwecearyeja.dll",
    // 是否传入拷贝资源的程序集，如拷贝图标产品信息等等的程序集
    assemblyToCopyResorcesFrom: null,
    // 是否是 GUI 程序。为 false 代表控制台，可以显示控制台内容。为 true 隐藏控制台，为传统的桌面应用程序，如 WinForms 或 WPF 应用
    windowsGraphicalUserInterface: false
);
```

尝试运行以上代码，即可看到输出文件夹多了名为 `ConsoleApp1.exe` 的文件，双击这个 `ConsoleApp1.exe` 的文件，将会尝试加载 `Foo\ReagalljaqewhurNiwecearyeja.dll` 文件作为其入口文件

以上代码里面，允许将入口 DLL 放在 Foo 子文件夹里面。最长路径是 256 个字符

以上代码使用了 `Foo\` 的写法，不适用于在 Linux 环境上，如以下代码，将 AppHost 换成龙芯的，此时需要更换为 `Foo/ReagalljaqewhurNiwecearyeja.dll` 的写法

```csharp
var appHost = @"C:\lindexi\loongarch64\dotnet-apphost-pack-8.0_8.0.14-1_loongarch64\data\usr\share\dotnet\packs\Microsoft.NETCore.App.Host.linux-loongarch64\8.0.14\runtimes\linux-loongarch64\native\apphost";

HostWriter.CreateAppHost
(
    // 龙芯的要自己下，下载地址： https://ftp.loongnix.cn/dotnet/8.0.14/8.0.14-1/deb/dotnet-apphost-pack-8.0_8.0.14-1_loongarch64.deb
    appHostSourceFilePath: appHost,
    // 输出路径，包括指定输出文件名
    appHostDestinationFilePath: "ConsoleApp1",
    // 入口的 DLL 是哪一个，这是相对于 exe 所在的 dll 路径
    appBinaryFilePath: "Foo/ReagalljaqewhurNiwecearyeja.dll",
    // 是否传入拷贝资源的程序集，如拷贝图标产品信息等等的程序集
    assemblyToCopyResorcesFrom: null,
    // 是否是 GUI 程序。为 false 代表控制台，可以显示控制台内容。为 true 隐藏控制台，为传统的桌面应用程序，如 WinForms 或 WPF 应用
    windowsGraphicalUserInterface: false
);
```

如代码注释，以上的 `C:\lindexi\loongarch64\dotnet-apphost-pack-8.0_8.0.14-1_loongarch64\data\usr\share\dotnet\packs\Microsoft.NETCore.App.Host.linux-loongarch64\8.0.14\runtimes\linux-loongarch64\native\apphost` 文件就是从 https://ftp.loongnix.cn/dotnet/8.0.14/8.0.14-1/deb/dotnet-apphost-pack-8.0_8.0.14-1_loongarch64.deb 下载的

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b66056addefd6dc80c944c27563edb42153ee452/Workbench/ReagalljaqewhurNiwecearyeja) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/b66056addefd6dc80c944c27563edb42153ee452/Workbench/ReagalljaqewhurNiwecearyeja) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b66056addefd6dc80c944c27563edb42153ee452
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin b66056addefd6dc80c944c27563edb42153ee452
```

获取代码之后，进入 Workbench/ReagalljaqewhurNiwecearyeja 文件夹，即可获取到源代码

更多 AppHost 内容，请参阅 [dotnet core 应用是如何跑起来的 通过AppHost理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87AppHost%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

如对 dotnet 的启动机制感兴趣，请参阅 [dotnet core 应用是如何跑起来的 通过自己写一个 dotnet host 理解运行过程](https://blog.lindexi.com/post/dotnet-core-%E5%BA%94%E7%94%A8%E6%98%AF%E5%A6%82%E4%BD%95%E8%B7%91%E8%B5%B7%E6%9D%A5%E7%9A%84-%E9%80%9A%E8%BF%87%E8%87%AA%E5%B7%B1%E5%86%99%E4%B8%80%E4%B8%AA-dotnet-host-%E7%90%86%E8%A7%A3%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.html )

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。