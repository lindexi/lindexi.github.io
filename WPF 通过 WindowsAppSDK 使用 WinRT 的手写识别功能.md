# WPF 通过 WindowsAppSDK 使用 WinRT 的手写识别功能

本文告诉大家如何在基于 .NET 6 的 WPF 使用 WinRT 的手写识别功能

<!--more-->
<!-- 发布 -->
<!-- 博客 -->

在开始之前需要先创建 WPF 项目，创建完成之后，可替换 csproj 项目文件为以下代码，用来安装初始化环境

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows10.0.19041</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWPF>true</UseWPF>
    <PlatformTarget>x86</PlatformTarget>
    <RuntimeIdentifiers>win10-x86;win10-x64</RuntimeIdentifiers>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.3.230331000" />
  </ItemGroup>
  
</Project>
```

如果以上代码构建不通过，请参阅 [修复 WPF 安装 WindowsAppSDK 库构建失败 NETSDK1082 和 NETSDK1112 找不到 win10-arm 失败](https://blog.lindexi.com/post/%E4%BF%AE%E5%A4%8D-WPF-%E5%AE%89%E8%A3%85-WindowsAppSDK-%E5%BA%93%E6%9E%84%E5%BB%BA%E5%A4%B1%E8%B4%A5-NETSDK1082-%E5%92%8C-NETSDK1112-%E6%89%BE%E4%B8%8D%E5%88%B0-win10-arm-%E5%A4%B1%E8%B4%A5.html )

另外，还需要你的 VisualStudio 2022 安装对应的负载，如 10.0.19041 负载等，基本上 Visual Studio 告诉你缺哪个就安装哪个

先在 MainWindow.xaml 放入一个 InkCanvas 元素，用来绘制笔迹和创建笔迹对象，代码如下

```xml
    <Grid>
        <InkCanvas x:Name="InkCanvas" StrokeCollected="InkCanvas_OnStrokeCollected"></InkCanvas>
    </Grid>
```

在 `InkCanvas_OnStrokeCollected` 方法里面执行手写识别功能，以下是识别形状的代码逻辑

```csharp
using Windows.Foundation;
using Windows.UI.Input.Inking;
using Windows.UI.Input.Inking.Analysis;

using Point = Windows.Foundation.Point;

    private async void InkCanvas_OnStrokeCollected(object sender, InkCanvasStrokeCollectedEventArgs e)
    {
        var inkStrokeBuilder = new InkStrokeBuilder();
        var inkStroke = inkStrokeBuilder.CreateStroke(e.Stroke.StylusPoints.Select(t => new Point(t.X, t.Y)));
        var inkAnalyzer = new InkAnalyzer();
        inkAnalyzer.AddDataForStroke(inkStroke);
        var result = await inkAnalyzer.AnalyzeAsync();
        foreach (IInkAnalysisNode inkAnalysisNode in inkAnalyzer.AnalysisRoot.FindNodes(InkAnalysisNodeKind.InkDrawing))
        {
            var inkAnalysisInkDrawing = inkAnalysisNode as InkAnalysisInkDrawing;
            var value = inkAnalysisInkDrawing?.DrawingKind;
            if (value == InkAnalysisDrawingKind.Triangle)
            {

            }
        }
    }
```

例如在界面绘制一个三角形，也许就能进入到 `value == InkAnalysisDrawingKind.Triangle` 的判断

以上的 InkAnalyzer 等类型都是 Windows Runtime 提供的类型，也就是需要至少是 Win10 以上版本系统才能支持的功能。如果期望在 Win10 及以下版本使用笔迹识别，可选使用 Microsoft.Ink 组件

本文以上代码放在[github](https://github.com/lindexi/lindexi_gd/tree/55aa84a041b5f9e3446a646662fc079695783e81/WekihairfawKudurnearka) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/55aa84a041b5f9e3446a646662fc079695783e81/WekihairfawKudurnearka) 欢迎访问

可以通过如下方式获取源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 55aa84a041b5f9e3446a646662fc079695783e81
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 55aa84a041b5f9e3446a646662fc079695783e81
```

获取代码之后，进入 WekihairfawKudurnearka 文件夹

更多笔迹相关，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )