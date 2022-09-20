
# 使用 Uno Islands 在现有 WPF 里面嵌入 Uno 框架

随着 2022 9 月份 Uno 发布了 4.5 版本，现有的 WPF 应用多了一个新的开发模式，那就是通过 Uno Islands 技术，在现有的 WPF 应用里面嵌入 Uno 应用。通过此方式可以辅助在现有的 WPF 项目里面，部分功能迁入 Uno 项目，或者是某些新开发功能通过 Uno 实现，从而利用 Uno 跨平台的能力，逐个功能点支持跨平台功能。逐个小功能接入的方式，让开发者不需要为一次性迁移一个庞大的项目而烦恼

<!--more-->


<!-- CreateTime:2022/9/19 8:23:29 -->


<!-- 博客 -->
<!-- 发布 -->

本文将尝试写一个非常简单的例子用来尝试在一个空的 WPF 项目上，接入 Uno Islands 技术，核心代码完全来自 Uno 官方，详细请看 [Uno Islands](https://platform.uno/docs/articles/guides/uno-islands.html) 官方文档

在开始之前，先介绍一下 Uno 项目是什么。这是一个支持用 C#+XAML 实现跨平台的 UI 框架，直接对标就是 MAUI 框架。只是 UNO 的主力开发不是微软官方，而是第三方开发者，而且还是特别特别卷的第三方开发者，总体开发进度预计是 MAUI 的 5-10 倍。在 MAUI 还没正式发布，还在进入预览版的时候，这时 UNO 早已发布商业可用版本。在 MAUI 还在打磨的时候，这时 UNO 开始不断发布各种新迭代功能了。说不定后续 UNO 还有被某软收购的可能

总的来说，我认为 UNO 还是比较能打的。而且更加有趣的是 UNO 和 MAUI 之间不是打架的关系，很多开发者都在这两个框架之间跑动。同样的 bug 要修两次，那才有趣

至于好不好用，我推荐大家试试看咯

回到主题，在今年 9 月份新加入的 Uno Islands 技术，让我开始准备在实际的大应用上部分功能接入 Uno 框架。通过 Uno Islands 技术，可以在 WPF 里面划某个矩形范围，让这个范围内的内容使用 Uno 框架进行绘制和交互。为了方便演示，接下来新建一个空白的 WPF 项目，在这个空白的 WPF 项目里面，在主窗口同时放一个 WPF 的控件和一个用来承载 Uno 框架的 UnoXamlHost 控件，以及新建一个共享项目，在共享项目里面存放 Uno 框架所需的代码和编写简单的 UI 界面

新建一个空白的 WPF 项目，采用 dotnet 6 框架，编辑 csproj 项目文件，加上必要的引用

```xml
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Logging" Version="5.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="5.0.0" />
    <PackageReference Include="Uno.WinUI.Skia.Wpf" Version="4.5.9" />
    <PackageReference Include="Uno.WinUI.RemoteControl" Version="4.5.0-dev.453" Condition="'$(Configuration)'=='Debug'" />
    <PackageReference Include="Uno.UI.Adapter.Microsoft.Extensions.Logging" Version="4.5.0-dev.453" />
    <PackageReference Include="Uno.WinUI.XamlHost" Version="4.5.0-dev.453" />
    <PackageReference Include="Uno.WinUI.XamlHost.Skia.Wpf" Version="4.5.0-dev.453" />
  </ItemGroup>
```

接着新建一个叫 TestUnoIslands 的共享项目，这个共享项目里面的文件内容和代码，推荐是从我的测试代码里面抄袭: [https://github.com/lindexi/lindexi_gd/tree/7ddbfed126c37ec07d5d0d94468f5d0551e122f9/TestUnoIslands/TestUnoIslands](https://github.com/lindexi/lindexi_gd/tree/7ddbfed126c37ec07d5d0d94468f5d0551e122f9/TestUnoIslands/TestUnoIslands)

从我的测试代码仓库里面拷贝代码文件的方式可以快速拷贝出一个使用 Uno 框架的项目，这些代码逻辑和官方的[例子](https://github.com/unoplatform/Uno.Samples/tree/master/UI/UnoIslandsSampleApp/UnoIslandsSampleApp.Shared) 代码接近相同。从官方代码仓库里面拷贝例子也不错： [https://github.com/unoplatform/Uno.Samples/tree/master/UI/UnoIslandsSampleApp/UnoIslandsSampleApp.Shared](https://github.com/unoplatform/Uno.Samples/tree/master/UI/UnoIslandsSampleApp/UnoIslandsSampleApp.Shared)

这里的共享项目可以认为是一个现有的使用 Uno 框架的项目，接下来就是在刚才创建的 WPF 项目里面，嵌入这个 Uno 项目的内容

在刚才新建的 WPF 项目里面，添加共享项目的引用，引用刚才创建的共享项目，接着为了解决 Uno 的字体问题，在 WPF 项目里面添加 `uno-fluentui-assets.ttf` 字体，这个字体文件可以从 github 这里下载： [https://github.com/lindexi/lindexi_gd/blob/7ddbfed126c37ec07d5d0d94468f5d0551e122f9/TestUnoIslands/TestUnoIslands.Wpf/Assets/Fonts/uno-fluentui-assets.ttf](https://github.com/lindexi/lindexi_gd/blob/7ddbfed126c37ec07d5d0d94468f5d0551e122f9/TestUnoIslands/TestUnoIslands.Wpf/Assets/Fonts/uno-fluentui-assets.ttf)

添加的 ttf 字体文件放入到 `Assets\Fonts` 文件夹内，同时编辑 WPF 项目的 csproj 文件，添加这个 ttf 文件的引用

```xml
  <ItemGroup>
    <Content Include="Assets\Fonts\uno-fluentui-assets.ttf" />
  </ItemGroup>
```

再编辑 WPF 项目的 csproj 文件，设置对共享项目里的 XAML 文件的引用

```xml
  <ItemGroup>
    <UpToDateCheckInput Include="..\TestUnoIslands\**\*.xaml" />
  </ItemGroup>
```

编辑完成之后的 csproj 项目文件的内容如下

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Logging" Version="5.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="5.0.0" />
    <PackageReference Include="Uno.WinUI.Skia.Wpf" Version="4.5.9" />
    <PackageReference Include="Uno.WinUI.RemoteControl" Version="4.5.0-dev.453" Condition="'$(Configuration)'=='Debug'" />
    <PackageReference Include="Uno.UI.Adapter.Microsoft.Extensions.Logging" Version="4.5.0-dev.453" />
    <PackageReference Include="Uno.WinUI.XamlHost" Version="4.5.0-dev.453" />
    <PackageReference Include="Uno.WinUI.XamlHost.Skia.Wpf" Version="4.5.0-dev.453" />
  </ItemGroup>
  <ItemGroup>
    <UpToDateCheckInput Include="..\TestUnoIslands\**\*.xaml" />
  </ItemGroup>
  <ItemGroup>
    <Content Include="Assets\Fonts\uno-fluentui-assets.ttf" />
  </ItemGroup>
  <Import Project="..\TestUnoIslands\TestUnoIslands.projitems" Label="Shared" />

</Project>
```

接下来打开 WPF 项目的主窗口用来添加对 Uno 项目的引用。开始之前，在 XAML 加上命名空间

```
xmlns:xamlHost="clr-namespace:Uno.UI.XamlHost.Skia.Wpf;assembly=Uno.UI.XamlHost.Skia.Wpf"
```

这是一句话的命名空间引用，官方的文档里面为了格式化，在文档里面换了行

通过添加 Uno Island 即可进行对 Uno 项目的嵌入，添加的代码如下

```xml
<xamlHost:UnoXamlHost InitialTypeName="UnoIslandsSampleApp.MainPage" />
```

使用上和 WinUI 提供的 Xaml Island 几乎相同。如此即可完成嵌入

完全的 XAML 代码如下

```xml
<Window x:Class="TestUnoIslands.Wpf.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:TestUnoIslands.Wpf"
        mc:Ignorable="d"
        xmlns:xamlHost="clr-namespace:Uno.UI.XamlHost.Skia.Wpf;assembly=Uno.UI.XamlHost.Skia.Wpf"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <Grid Margin="20">
            <Grid.RowDefinitions>
                <RowDefinition Height="Auto" />
                <RowDefinition Height="*" />
            </Grid.RowDefinitions>
            <Button x:Name="Button" Click="Button_OnClick">Hello from WPF!</Button>
            <xamlHost:UnoXamlHost Grid.Row="1" 
                                  InitialTypeName="UnoIslandsSampleApp.MainPage" />
        </Grid>
    </Grid>
</Window>
```

尝试运行项目，可以看到在一个 WPF 项目里面嵌入了 Uno 的页面

依然的，这个 Uno Islands 技术存在和 WinFormsHost 技术相同的问题，在此矩形范围内，只允许一个 UI 框架工作。被嵌入 Uno 的范围内，不能再次叠加上 WPF 的控件。但我认为这个问题其实也不大，说不定我想不开，或者是某位大佬行行好，就帮他实现了一个可以作为元素插入的功能哈

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/7ddbfed126c37ec07d5d0d94468f5d0551e122f9/TestUnoIslands) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/7ddbfed126c37ec07d5d0d94468f5d0551e122f9/TestUnoIslands) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 7ddbfed126c37ec07d5d0d94468f5d0551e122f9
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 7ddbfed126c37ec07d5d0d94468f5d0551e122f9
```

获取代码之后，进入 TestUnoIslands 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。