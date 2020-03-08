# WPF 通过 ReadyToRun 提升性能

通过 ReadyToRun 可以在程序集同时包含 IL 和本机代码，可以有效提升软件的启动速度

<!--more-->
<!-- CreateTime:2019/7/2 10:29:55 -->

<!-- csdn -->

在 .NET Core 3.0 Preview 6 发布了 ReadyToRun 的功能，通过这个功能可以提升软件的启动性能，在程序设置在发布的时候使用 ReadyToRun 的特性，将会编译出来 ReadyToRun（R2R）格式的程序集

在这个程序集里面同时包含了 IL 和本机代码，可以做到在启动的时间减少 JIT 编译的时间，减少的这部分启动时间是 JIT 编译代码使用的时间

经过测试，在测试项目上，可以从原来的 1.9 秒的启动时间使用 ReadyToRun 减少为 1.3 秒的时间，同时因为在启动过程不需要 JIT 进行中间编译，可以节省此部分的内存，将内存从 69.1 MB 降低为 55.7 MB 大小

当然在运行性能上的提升同时也会让应用程序大小变大因为当前的应用程序会同时包含 IL 和本机代码，在相同的测试项目上，应用程序的大小从原来的 150MB 变为 156MB 这个大小对于桌面端应用程序几乎是可以被忽略

我认为使用 ReadyToRun 是对桌面端应用程序的极大的提升，这个功能其实已经很古老了，但是一直都没有发布，下面让我告诉大家如何在项目上应用这个白科技提高软件性能

请确定本地的 dotnet sdk 版本大于 .NET Core 3.0 Preview 6 使用以下命令可以知道自己的 sdk 版本

```csharp
dotnet --info
```

在控制台输入上面代码将会输出当前设备安装的版本

```csharp
 Version:   3.0.100-preview6-012264
 Commit:    be3f0c1a03

运行时环境:
 OS Name:     Windows
 OS Version:  10.0.18362
 OS Platform: Windows
 RID:         win10-x64
 Base Path:   C:\Program Files\dotnet\sdk\3.0.100-preview6-012264\
```

如果发现自己的版本比较低请到[官网](https://dotnet.microsoft.com/download/dotnet-core/3.0) 下载最新版本安装

打开或新建一个 WPF 项目，在项目文件里面添加属性 PublishReadyToRun 请看代码

```csharp
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>netcoreapp3.0</TargetFramework>
        <UseWPF>true</UseWPF>
        <PublishReadyToRun>true</PublishReadyToRun>
    </PropertyGroup>

</Project>
```

注意，并非只有 WPF 可以使用这个特性，理论上任何的 dotnet core 程序都可以

在添加了 `<PublishReadyToRun>true</PublishReadyToRun>` 之后可以进行发布，此时发布的时候需要带上确定的运行环境属性，因为这个特性是编译为本地代码需要对每个平台编译为特定的代码

同时 ReadyToRun 只能用于[独立部署](https://docs.microsoft.com/dotnet/core/deploying/) 的应用

请使用下面代码进行发布使用 ReadyToRun 特性的程序

```csharp
dotnet publish -c release  -r win-x64 --self-contained
```

因为 ReadyToRun 是对特性的平台创建特定的代码，所以 `-r` 属性是不能少的。另外因为默认只对独立部署生效，所以独立部署属性可选添加

一个空白的 WPF 程序编译的时候将会让程序文件多了大约 3k 的大小，但是这部分关系不大，因为在 [.NET Core 3.0 Preview 6 ](https://devblogs.microsoft.com/dotnet/announcing-net-core-3-0-preview-6/ ) 还发布了 [Assembly linking](https://aka.ms/dotnet-illink) 功能用于减少应用程序大小

此时的应用程序包含了本机代码，如果需要同时创建本机代码的符号，请在项目文件添加以下代码

```csharp
        <PublishReadyToRunEmitSymbols>true</PublishReadyToRunEmitSymbols>
```

现在的项目文件代码如下

```csharp
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

    <PropertyGroup>
        <OutputType>WinExe</OutputType>
        <TargetFramework>netcoreapp3.0</TargetFramework>
        <UseWPF>true</UseWPF>
        <PublishReadyToRun>true</PublishReadyToRun>
        <PublishReadyToRunEmitSymbols>true</PublishReadyToRunEmitSymbols>
    </PropertyGroup>

</Project>
```

依然使用上面的命令发布

```csharp
dotnet publish -c release  -r win-x64 --self-contained
```

此时在发布的文件夹可以找到比原来多出的 `.ni.pdb` 文件，这就是本机代码的符号文件

关于项目文件的配置请看我的[项目](https://github.com/lindexi/lindexi_gd/tree/36d9e70722f86bc8d03385868a99fc9c7719b504/FuhelerjaihuBuqibeayay) 下载之后可以使用本文方法进行编译

更多请看[官方文档](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/readytorun-overview.md) 

[Announcing .NET Core 3.0 Preview 6 ](https://devblogs.microsoft.com/dotnet/announcing-net-core-3-0-preview-6/ )

[coreclr/readytorun-overview.md at master · dotnet/coreclr](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/readytorun-overview.md )

[coreclr/crossgen.md at master · dotnet/coreclr](https://github.com/dotnet/coreclr/blob/master/Documentation/building/crossgen.md )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
