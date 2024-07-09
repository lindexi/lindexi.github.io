
# dotnet WinUI3 Win2D 翻转图片

本文将告诉大家如何在 WinUI3 里面使用 Win2D 进行图片的翻转，本文的方法也适用于 UWP 框架

<!--more-->


<!-- CreateTime:2024/07/07 07:17:07 -->

<!-- 发布 -->
<!-- 博客 -->

图片的翻转在 Win2D 里面，可以使用 Transform2DEffect 特效来辅助实现，核心逻辑就是通过缩放矩阵当成2D翻转矩阵，将缩放的 X 和 Y 传入负数即可分别实现对应方向的翻转。比如左右水平翻转可将 X 值传入负数，如 -1 表示直接水平翻转

本文接下来将告诉大家一步步进行实现从文件加载图片，再将图片进行翻转在界面显示

在 WinUI3 或 UWP 里面使用 Win2D 需按照 dotnet 的惯例安装 NuGet 库。在 UWP 里面需要安装 [Win2D.uwp](http://www.nuget.org/packages/Win2D.uwp) 库，在 WinUI 3 项目里面需要安装 [Microsoft.Graphics.Win2D](http://www.nuget.org/packages/Microsoft.Graphics.Win2D) 库

对于 WinUI 3 项目，由于使用了 SDK 的 csproj 项目文件代码风格，可以编辑 csproj 项目文件，在 ItemGroup 里面添加如下代码进行快速安装库，代码如下

```xml
    <PackageReference Include="Microsoft.Graphics.Win2D" Version="1.2.0" />
```

编辑之后的 csproj 代码文件大概如下

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net8.0-windows10.0.19041.0</TargetFramework>
    <TargetPlatformMinVersion>10.0.17763.0</TargetPlatformMinVersion>
    <RootNamespace>ChaigelyojeeBifakeljair</RootNamespace>
    <ApplicationManifest>app.manifest</ApplicationManifest>
    <Platforms>x86;x64;ARM64</Platforms>
    <RuntimeIdentifiers Condition="$([MSBuild]::GetTargetFrameworkVersion('$(TargetFramework)')) &gt;= 8">win-x86;win-x64;win-arm64</RuntimeIdentifiers>
    <RuntimeIdentifiers Condition="$([MSBuild]::GetTargetFrameworkVersion('$(TargetFramework)')) &lt; 8">win10-x86;win10-x64;win10-arm64</RuntimeIdentifiers>
    <PublishProfile>win-$(Platform).pubxml</PublishProfile>
    <UseWinUI>true</UseWinUI>
    <EnableMsixTooling>true</EnableMsixTooling>
  </PropertyGroup>

  <ItemGroup>
    <Content Include="Assets\SplashScreen.scale-200.png" />
    <Content Include="Assets\LockScreenLogo.scale-200.png" />
    <Content Include="Assets\Square150x150Logo.scale-200.png" />
    <Content Include="Assets\Square44x44Logo.scale-200.png" />
    <Content Include="Assets\Square44x44Logo.targetsize-24_altform-unplated.png" />
    <Content Include="Assets\StoreLogo.png" />
    <Content Include="Assets\Wide310x150Logo.scale-200.png" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Graphics.Win2D" Version="1.2.0" />
    <PackageReference Include="Microsoft.Windows.SDK.BuildTools" Version="10.0.22621.756" />
    <PackageReference Include="Microsoft.WindowsAppSDK" Version="1.5.240607001" />
    <Manifest Include="$(ApplicationManifest)" />
  </ItemGroup>

  <!--
    Defining the "Msix" ProjectCapability here allows the Single-project MSIX Packaging
    Tools extension to be activated for this project even if the Windows App SDK Nuget
    package has not yet been restored.
  -->
  <ItemGroup Condition="'$(DisableMsixProjectCapabilityAddedByProject)'!='true' and '$(EnableMsixTooling)'=='true'">
    <ProjectCapability Include="Msix" />
  </ItemGroup>

  <!--
    Defining the "HasPackageAndPublishMenuAddedByProject" property here allows the Solution
    Explorer "Package and Publish" context menu entry to be enabled for this project even if
    the Windows App SDK Nuget package has not yet been restored.
  -->
  <PropertyGroup Condition="'$(DisableHasPackageAndPublishMenuAddedByProject)'!='true' and '$(EnableMsixTooling)'=='true'">
    <HasPackageAndPublishMenu>true</HasPackageAndPublishMenu>
  </PropertyGroup>
</Project>
```

本文的示例里面，将编写在 MainWindow.xaml 里面，先添加命名空间 `xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"` 的引用

接着在 xaml 里面添加上 CanvasControl 控件，同时监听 CreateResources 和 Draw 事件，代码如下

```xml
 <canvas:CanvasControl x:Name="Canvas" ClearColor="Black" CreateResources="Canvas_OnCreateResources" Draw="Canvas_OnDraw"/>
```

按照 Win2D 的设计，咱将在 CreateResources 事件里面，进行本地文件的加载作为图片，在 Draw 事件里面进行绘制

为了演示图片翻转，咱需要先有图片。本文这里将读取本地文件作为图片。实现的逻辑放在 `Canvas_OnCreateResources` 方法里面

代码如下

```csharp
    private void Canvas_OnCreateResources(CanvasControl sender, CanvasCreateResourcesEventArgs args)
    {
        var imageFile = @"C:\lindexi\Image\1.png";// 图片地址大家自己替换
        if (!File.Exists(imageFile))
        {
            // 自己换成自己的图片
            Debugger.Break();
        }

        var task = LoadImageAsync();
        args.TrackAsyncAction(task.AsAsyncAction());

        async Task LoadImageAsync()
        {
            CanvasBitmap canvasBitmap = await CanvasBitmap.LoadAsync(sender, imageFile);
            _canvasBitmap = canvasBitmap;
        }
    }

    private CanvasBitmap? _canvasBitmap;
```

请大家将上面代码的 `C:\lindexi\Image\1.png` 路径替换为你自己的本地图片文件的路径

以上代码写了一个名为 LoadImageAsync 的内部方法，这是因为加载图片需要用到异步，需要包装 Task 作为异步任务，再将异步任务通过 TrackAsyncAction 告知给到 Win2D 层。如此即可让 Win2D 等待 LoadImageAsync 完成才完成资源创建逻辑，接着再执行 Draw 绘制。如果没有使用 TrackAsyncAction 方式告知 Win2D 的话，那可能在资源加载完成之前，就会进入到 Draw 绘制导致状态不符合预期

换句话说，直接将 `Canvas_OnCreateResources` 改为 async void 是不可以的，一旦这么做了，那 Win2D 层是无法感知到资源异步加载完成的，也就让 Win2D 层无法知道在何时才是合适的触发渲染

完成图片资源加载逻辑之后，接下来进入到核心的 `Canvas_OnDraw` 方法

图片的翻转在 Win2D 里面，可以使用 Transform2DEffect 特效来辅助实现，核心逻辑就是通过缩放矩阵当成2D翻转矩阵，将缩放的 X 和 Y 传入负数即可分别实现对应方向的翻转。本文以下将演示如何将图片进行水平翻转。还请大家不用担心用到矩阵，本文这里不会直接用到多少矩阵知识，只是简单调用方法而已

先从字段 `_canvasBitmap` 获取 `CanvasBitmap` 类型的对象，保持稳定的话需要判断一次空，防止资源创建步骤出现诡异的事情导致没有创建成功

```csharp
    private void Canvas_OnDraw(CanvasControl sender, CanvasDrawEventArgs args)
    {
        if (_canvasBitmap is { } canvasBitmap)
        {
            ... // 忽略其他代码
        }
    }

    private CanvasBitmap? _canvasBitmap;
```

以上代码只写了 if 为 true 的代码，在实际产品代码里面推荐也加上 else 打上日志或进行其他处理

对图片进行中心点水平翻转，可以使用 Matrix3x2 创建缩放矩阵，在 X 方向传入 -1 表示翻转，在 Y 方向传入 1 表示不变，再传入图片的中心点即可

先获取图片的中心点，代码如下

```csharp
    private void Canvas_OnDraw(CanvasControl sender, CanvasDrawEventArgs args)
    {
        if (_canvasBitmap is { } canvasBitmap)
        {
            var centerX = canvasBitmap.Bounds._width / 2;
            var centerY = canvasBitmap.Bounds._height / 2;
            ... // 忽略其他代码
        }
    }
```

这里的中心点直接取宽度高度的一半是因为图片本身没有平移，且接下来的特效是基于当前图片的坐标系。相当于图片的左上角就是 0 0 点，直接取宽度高度一半就是刚好中心点的值

接下来按照 [win10 uwp win2d 入门 看这一篇就够了](https://blog.lindexi.com/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html ) 和 [win10 uwp win2d 特效](https://blog.lindexi.com/post/win10-uwp-win2d-%E7%89%B9%E6%95%88.html ) 里面提供的方法，创建 Transform2DEffect 特效，代码如下

```csharp
            var transform2DEffect = new Transform2DEffect();
```

将当前的图片作为特效的输入源，代码如下

```csharp
            transform2DEffect.Source = canvasBitmap;
```

接下来对 Transform2DEffect 传入翻转矩阵，即在 X 方向传入 -1 表示翻转，在 Y 方向传入 1 表示不变，再传入图片的中心点，代码如下

```csharp
            var matrix3X2 = Matrix3x2.CreateScale(-1, 1, new Vector2(centerX, centerY));
            transform2DEffect.TransformMatrix = matrix3X2;
```

可以看到本文只是简单调用 Matrix3x2 的 CreateScale 方法，没有使用到多少的矩阵知识

完成特效之后，即可将 Transform2DEffect 传入到 CanvasDrawEventArgs 进行绘制，代码如下

```csharp
            args.DrawingSession.DrawImage(transform2DEffect);
```

如此即可完成图片的左右翻转

有伙伴可能好奇，使用本文的特效方式对图片进行翻转，性能如何呢？答案是性能是特别高的，在 Win2D 里面绝大部分特效对于 GPU 来说时间复杂度都是 O(1) 级，这是什么概念呢，用简单的话说就是 GPU 一口气就能做完，不耗资源的

以上就是本文提供的简单示例代码，在 WinUI 3 或 UWP 里面使用 Win2D 进行翻转图片。本文以上的例子代码只是做水平左右翻转，相信阅读了以上内容的伙伴自己也能实现垂直翻转功能

再近一步，如果图片比较大或比较小，我想要再次缩放图片，让图片刚好在界面里面填充显示，可以如何实现？简单实现那还是使用 Transform2DEffect 特效，在 Win2D 里面可以特效套特效，此时的渲染效率依然是特别高的。在上文，咱使用缩放矩阵当成翻转矩阵，那接下来对图片的缩放就可以将缩放矩阵当成缩放矩阵了

将以上的 `transform2DEffect` 作为下一个 Transform2DEffect 的输入源，在新的 Transform2DEffect 对图片进行缩放，代码如下

```csharp
            var transform2DEffect2 = new Transform2DEffect()
            {
                Source = transform2DEffect,
                TransformMatrix = Matrix3x2.CreateScale((float) (sender.ActualWidth / canvasBitmap.Bounds.Width), (float) (sender.ActualHeight / canvasBitmap.Bounds.Height))
            };
```

最后再将 `transform2DEffect2` 输出到界面，代码如下

```csharp
            args.DrawingSession.DrawImage(transform2DEffect2);
```

以上代码的 `transform2DEffect2` 就是使用当前的 CanvasControl 的实际布局尺寸和图片的尺寸进行比例缩放，如此即可将图片拉伸填充到相同的尺寸。这一点是非常简单的，如果大家想不明白的话，试试拿出纸张和笔画一画，基础初中知识就可以理解

以上代码更多是和大家演示在 Win2D 里面将两个特效进行叠加的写法。在实际应用里面，可以将上述两个 Transform2DEffect 进行合并，因为这两个 Transform2DEffect 都只是进行 TransformMatrix 叠加而已，可以进行更简单的数学计算。接下来部分需要用到点矩阵的知识，即上述的两个特效的叠加和对两个 TransformMatrix 矩阵进行相乘的结果是相同的。此时可以只用一个 Transform2DEffect 然后将本文的两个 Transform2DEffect 的 TransformMatrix 进行相乘即可，如此可以省略一次特效

尽管特效对于 GPU 来说大多都是不费资源的，对 Transform2DEffect 来说，资源使用量就更少了。但是能够通过基础 CPU 进行简单数学计算，还是能做非常多的性能提升的。这里需要额外说明的是性能是相对的，尽管说通过基础 CPU 进行简单数学计算能做非常多的性能提升，但是用人话说就是从一分钱提升到零点一分钱而已，约等于依然还是不用钱

为了更好的进行演示效果，我这里还在界面添加了一个 ToggleButton 用于点击的时候控制是否翻转，界面代码如下

```xml
<ToggleButton HorizontalAlignment="Center" VerticalAlignment="Center" Content="是否翻转" Click="Button_OnClick"/>
```

修改之后的界面代码如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<Window
    x:Class="ChaigelyojeeBifakeljair.MainWindow"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:ChaigelyojeeBifakeljair"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
    mc:Ignorable="d" Closed="MainWindow_OnClosed">

    <Grid>
        <canvas:CanvasControl x:Name="Canvas" ClearColor="Black" CreateResources="Canvas_OnCreateResources" Draw="Canvas_OnDraw"/>

        <ToggleButton HorizontalAlignment="Center" VerticalAlignment="Center" Content="是否翻转" Click="Button_OnClick"/>
    </Grid>
</Window>
```

在点击的时候设置一个字段且让 Win2D 画布进行重新绘制，代码如下

```csharp
    private bool _shouldFlip = false;

    private void Button_OnClick(object sender, RoutedEventArgs e)
    {
        _shouldFlip = (sender as ToggleButton)?.IsChecked is true;
        Canvas.Invalidate();
    }
```

修改一下 `Canvas_OnDraw` 的代码，判断 `_shouldFlip` 字段决定翻转，修改之后的代码如下

```csharp
    private void Canvas_OnDraw(CanvasControl sender, CanvasDrawEventArgs args)
    {
        if (_canvasBitmap is { } canvasBitmap)
        {
            var centerX = canvasBitmap.Bounds._width / 2;
            var centerY = canvasBitmap.Bounds._height / 2;

            var transform2DEffect = new Transform2DEffect();
            transform2DEffect.Source = canvasBitmap;

            var flip = _shouldFlip ? -1 : 1;
            var matrix3X2 = Matrix3x2.CreateScale(flip, 1, new Vector2(centerX, centerY));
            transform2DEffect.TransformMatrix = matrix3X2;

            var transform2DEffect2 = new Transform2DEffect()
            {
                Source = transform2DEffect,
                TransformMatrix = Matrix3x2.CreateScale((float) (sender.ActualWidth / canvasBitmap.Bounds.Width), (float) (sender.ActualHeight / canvasBitmap.Bounds.Height))
            };

            args.DrawingSession.DrawImage(transform2DEffect2);
        }
    }
```

也许有伙伴说如果没有翻转了，那不如将 `transform2DEffect` 去掉好了。没错，但这个特效基本不耗资源，那就跑着咯

最后别忘了在窗口关闭的时候，清理 Win2D 的资源

```csharp
    private void MainWindow_OnClosed(object sender, WindowEventArgs args)
    {
        Canvas.RemoveFromVisualTree();
    }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/bb784a22d576278e2f6dfb878e8c760128e91dad/DirectX/Win2D/ChaigelyojeeBifakeljair) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/bb784a22d576278e2f6dfb878e8c760128e91dad/DirectX/Win2D/ChaigelyojeeBifakeljair) 上，可以使用如下命令行拉取代码

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin bb784a22d576278e2f6dfb878e8c760128e91dad
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin bb784a22d576278e2f6dfb878e8c760128e91dad
```

获取代码之后，进入 DirectX/Win2D/ChaigelyojeeBifakeljair 文件夹，即可获取到源代码

更多 UWP 或 WinUI3 开发教程，以及更多渲染相关教程，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。