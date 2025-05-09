
# WPF 基于 .NET 5 框架和 .NET 6 的 SDK 进行完全单文件发布

本文来告诉大家如何基于 .NET 5 框架和 .NET 6 SDK 进行完全单文件发布，这是对 WPF 应用程序进行独立发布，生成的是完全单文件的方法

<!--more-->


<!-- CreateTime:2021/8/5 8:57:00 -->


<!-- 发布 -->

在之前的版本，尽管也是基于 .NET 5 框架的 WPF 应用，然而在 .NET 5 的 SDK 下，除非是采用框架依赖的方法，否则大部分应用发布作为单文件将会运行失败。在 .NET 6 的 SDK 下，官方修复了一些文档，对于大部分 WPF 应用程序来说，可以在 .NET 6 的 SDK 下，可以发布为完全的单文件

发布方法是在参数加上 `-p:PublishSingleFile=true` 和 `-p:IncludeNativeLibrariesForSelfExtract=true` 两个参数

```
dotnet publish -r win-x86 -c release -p:PublishSingleFile=true -p:IncludeNativeLibrariesForSelfExtract=true
```

用此方法的要求是，如果在 WPF 的 XAML 或者业务逻辑里面，有用到 Content 的内容都需要进行更改，有使用到当前应用程序所在文件夹附近的其他的文件的逻辑，也需要进行更改。毕竟是单文件发布，也只有一个文件

更改的方法是将原本的读取文件的逻辑，放入到程序集里面，通过程序集读取

当前的 WPF 暂时不支持裁剪的功能，完全单文件无框架依赖发布的空应用有 130M 左右

[Single file application - .NET](https://docs.microsoft.com/en-us/dotnet/core/deploying/single-file?WT.mc_id=WD-MVP-5003260 )

---

更新：

完全单文件发布时，如果带上 `<EnableCompressionInSingleFile>true</EnableCompressionInSingleFile>` 进行压缩，则体积能压到 55MB 左右，大概就是 130MB 的一半以下，符合可执行文件的压缩率

本次测试采用 .NET 9 版本的 WPF 项目，其 csproj 项目文件内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net9.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <UseWPF>true</UseWPF>
    <IncludeNativeLibrariesForSelfExtract>true</IncludeNativeLibrariesForSelfExtract>
    <EnableCompressionInSingleFile>true</EnableCompressionInSingleFile>
  </PropertyGroup>

</Project>
```

发布文件 `FolderProfile.pubxml` 内容如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<!-- https://go.microsoft.com/fwlink/?LinkID=208121. -->
<Project>
  <PropertyGroup>
    <Configuration>Release</Configuration>
    <Platform>Any CPU</Platform>
    <PublishDir>bin\Release\net9.0-windows\publish\win-x86\</PublishDir>
    <PublishProtocol>FileSystem</PublishProtocol>
    <_TargetId>Folder</_TargetId>
    <TargetFramework>net9.0-windows</TargetFramework>
    <RuntimeIdentifier>win-x86</RuntimeIdentifier>
    <SelfContained>true</SelfContained>
    <PublishSingleFile>true</PublishSingleFile>
    <PublishReadyToRun>false</PublishReadyToRun>
  </PropertyGroup>
</Project>
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/200133ccfcdfe10ab71cd16f5c137e4a1c459029/WPFDemo/QahihefaQukukakearqeyi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/200133ccfcdfe10ab71cd16f5c137e4a1c459029/WPFDemo/QahihefaQukukakearqeyi) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 200133ccfcdfe10ab71cd16f5c137e4a1c459029
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 200133ccfcdfe10ab71cd16f5c137e4a1c459029
```

获取代码之后，进入 WPFDemo/QahihefaQukukakearqeyi 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。