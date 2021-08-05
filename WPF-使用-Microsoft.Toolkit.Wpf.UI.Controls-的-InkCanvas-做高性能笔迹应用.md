
# WPF 使用 Microsoft.Toolkit.Wpf.UI.Controls 的 InkCanvas 做高性能笔迹应用

本文告诉大家如何在 WPF 中应用上 UWP 的笔迹控件，从而实现性能超级高的笔迹应用的方法

<!--more-->


<!-- CreateTime:2021/7/28 16:16:03 -->

<!-- 发布 -->

先新建一个 .NET Core 3.1 的 WPF 应用，当前的方法不支持 .NET Framework 版本。但是请安心，因为带 UWP 控件的应用只能在 Win10 下跑，而 Win10 是不存在 .NET Core 的环境问题的，因此采用 .NET Core 框架将会非常稳

编辑此 WPF 应用的 csproj 项目文件，替换为如下代码

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <UseWPF>true</UseWPF>
    <RuntimeIdentifiers>win10-x64;win-x64;win10-x86;win-x86</RuntimeIdentifiers>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Toolkit.Win32.UI.SDK" Version="6.1.2" />
    <PackageReference Include="Microsoft.Toolkit.Win32.UI.XamlApplication" Version="6.1.3" />
    <PackageReference Include="Microsoft.Toolkit.Wpf.UI.Controls" Version="6.1.2" />
    <PackageReference Include="Microsoft.Toolkit.Wpf.UI.XamlHost" Version="6.1.2" />
    <PackageReference Update="Microsoft.VCRTForwarders.140" Version="1.0.7" />
  </ItemGroup>

</Project>
```

以上的关键在于加上 `RuntimeIdentifiers` 属性和 `Microsoft.VCRTForwarders.140` 的引用，如果没有加上，那么将会看到如下提示

```
  C:\Program Files\dotnet\sdk\5.0.101\Sdks\Microsoft.NET.Sdk\targets\Microsoft.PackageDependencyResolution.targets(241,
5): error NETSDK1047: 资产文件“C:\lindexi\doubi\LaykearduchuNachairgurharhear\obj\project.assets.json”没 有“netcore
app3.1/win-x86”的目标。确保已运行还原，且“netcoreapp3.1”已包含在项目的 TargetFrameworks 中。可能需要在项目 RuntimeIdentifiers 中包括“win-x86”。 [C:\lindexi\doubi\LaykearduchuNachairgurharhear\LaykearduchuNachairgurharhear.csproj]
```

接下来打开 MainWindow.xaml 文件，添加如下代码

```xml
<Window x:Class="LaykearduchuNachairgurharhear.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:LaykearduchuNachairgurharhear"
        xmlns:controls="clr-namespace:Microsoft.Toolkit.Wpf.UI.Controls;assembly=Microsoft.Toolkit.Wpf.UI.Controls"
        mc:Ignorable="d"
    
        Title="MainWindow" Height="450" Width="800">
    <Grid>
      <controls:InkCanvas x:Name="InkCanvas" DockPanel.Dock="Top" Loaded="InkCanvas_Loaded"/>
  </Grid>
</Window>
```

此时就可以在 WPF 应用中添加 UWP 的 InkCanvas 控件了，但是默认此控件是不能写字的。因此咱将在后台代码的 `InkCanvas_Loaded` 设置让笔迹控件可以在鼠标下画出笔迹

```csharp
        private void InkCanvas_Loaded(object sender, RoutedEventArgs e)
        {
            InkCanvas.InkPresenter.InputDeviceTypes = CoreInputDeviceTypes.Mouse;
        }
```

现在代码看起来还不多，其实通过如上的代码就是最简单的方式让 WPF 引用 UWP 的笔迹控件了。但是默认此时的应用还不能被运行，因为要用 UWP 的控件就需要做一次桌面打包，此时需要做的是再创建一个打包项目用来辅助打包。打包之后依然可以作为 Win32 的应用分发哈，不需要绑定到应用商店里面

接下来的步骤稍微多一些，好在本文最后放了本次用到的所有的代码，大家可以在本文最后拿到所有代码

新建一个打包项目，接着右击打包项目的应用程序，点击添加引用。添加刚才创建的 WPF 的引用。这里 VisualStudio 将会自动建立启动入口的联系，咱只需要点一下鼠标即可哈

接下来是新建一个测试使用的证书，测试使用的证书用于辅助安装 msix 安装包文件

新建测试证书的方法是双击 Package.appxmanifest 文件，点击打包，点击选择证书，点击创建证书。不需要设置密码，点击确定即可

这样就相当于完全完成了一个最简单的应用了，我推荐大家先完成这个最简单的应用，然后再继续添加自己的功能哈

设置打包应用作为 VisualStudio 启动项目，接着按下 F5 即可执行。更多请看 [VisualStudio 快速设置启动项目](https://blog.lindexi.com/post/VisualStudio-%E5%BF%AB%E9%80%9F%E8%AE%BE%E7%BD%AE%E5%90%AF%E5%8A%A8%E9%A1%B9%E7%9B%AE.html)

如果期望在服务器做自动打包，可以在命令行，进入打包应用的 csproj 所在文件夹，输入下面命令即可自动构建

```
msbuild -restore
```

通过以上命令即可构建出 Debug 版本的 msix 包，默认将会打包应用的 AppPackages 文件夹里面。如果需要构建出发布版本的 release 版本的安装包，请使用以下命令，更多请看 [MSBuild 常用参数](https://blog.lindexi.com/post/MSBuild-%E5%B8%B8%E7%94%A8%E5%8F%82%E6%95%B0.html)

```
msbuild -restore /p:Configuration=Release
```

默认输出的是 msix 包，而因为咱的证书是自己创建的测试证书，因此需要用上 Install.ps1 进行安装。更多请参阅旁加载安装部分的内容

如果做分发此应用的话，推荐使用 [加强版在国内分发 UWP 应用正确方式 通过win32安装UWP应用](https://blog.lindexi.com/post/%E5%8A%A0%E5%BC%BA%E7%89%88%E5%9C%A8%E5%9B%BD%E5%86%85%E5%88%86%E5%8F%91-UWP-%E5%BA%94%E7%94%A8%E6%AD%A3%E7%A1%AE%E6%96%B9%E5%BC%8F-%E9%80%9A%E8%BF%87win32%E5%AE%89%E8%A3%85UWP%E5%BA%94%E7%94%A8.html ) 的方法进行分发，此时就不需要让用户去关注证书问题

以上的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/739665b601d4af8a021fd30960278ed7cbe2b441/LaykearduchuNachairgurharhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/739665b601d4af8a021fd30960278ed7cbe2b441/LaykearduchuNachairgurharhear) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 739665b601d4af8a021fd30960278ed7cbe2b441
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 LaykearduchuNachairgurharhear 文件夹

如果不想走 UWP 安装包，也可以放在 WPF 应用程序上，请看 [WPF 引用 UWP 控件 不打包为 MSIX 分发的方法](https://blog.lindexi.com/post/WPF-%E5%BC%95%E7%94%A8-UWP-%E6%8E%A7%E4%BB%B6-%E4%B8%8D%E6%89%93%E5%8C%85%E4%B8%BA-MSIX-%E5%88%86%E5%8F%91%E7%9A%84%E6%96%B9%E6%B3%95.html)

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html ) 更多笔迹相关请看

- [WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html )
- [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)
- [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 
- [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )
- [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )
- [WPF 使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html )
- [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )
- [win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html )
- [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html)
- [WPF 笔迹触摸点收集工具](https://blog.lindexi.com/post/WPF-%E7%AC%94%E8%BF%B9%E8%A7%A6%E6%91%B8%E7%82%B9%E6%94%B6%E9%9B%86%E5%B7%A5%E5%85%B7.html )
- [WPF 实现自定义的笔迹橡皮擦](https://blog.lindexi.com/post/WPF-%E5%AE%9E%E7%8E%B0%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84%E7%AC%94%E8%BF%B9%E6%A9%A1%E7%9A%AE%E6%93%A6.html )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。