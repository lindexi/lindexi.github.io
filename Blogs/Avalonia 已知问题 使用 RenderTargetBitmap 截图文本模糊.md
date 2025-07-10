---
title: Avalonia 已知问题 使用 RenderTargetBitmap 截图文本模糊
description: 本文记录 Avalonia 的一个已知问题，使用 RenderTargetBitmap 进行截图时，如果顶层子控件没有设置背景色或背景色是透明色，则截图保存出来的图片里面的文本字符串都是模糊的
tags: Avalonia
category: 
---

<!-- 发布 -->
<!-- 博客 -->

此问题能够在 Avalonia 的 11.0-11.3.2 版本复现，更低版本我就没有测试了

出现问题的表现的截图界面图片的文本模糊如下图所示

<!-- ![](image/Avalonia 已知问题 使用 RenderTargetBitmap 截图文本模糊/Avalonia 已知问题 使用 RenderTargetBitmap 截图文本模糊0.png) -->
![](https://img2024.cnblogs.com/blog/1080237/202507/1080237-20250711073032744-1425262102.png)

在 Avalonia 里，使用 RenderTargetBitmap 对控件进行截图，如果控件的顶层子控件没有背景色，则会截图出一张文本是模糊的图片。如果给控件的顶层子控件设置白色作为背景色，则能够截图出正常渲染的清晰文本

此问题已报告给官方，详细请看 https://github.com/AvaloniaUI/Avalonia/issues/19229

现步骤如下：

1. 创建一个空的 Avalonia 程序，在 MainView.axaml 里面放入一个包含“Hello World”内容的 TextBlock 控件，以及两个按钮。两个按钮分别是 TakeSnapshotButton 和 TakeSnapshotWithFixButton 按钮
2. 设置 TakeSnapshotButton 按钮在点击的时候，使用 RenderTargetBitmap 进行截图，且将图片保存到 1.png 文件里
3. 设置 TakeSnapshotWithFixButton 按钮在点击的时候，先设置 RootGrid 的背景色为白色，然后再使用 RenderTargetBitmap 进行截图，且将图片保存到 2.png 文件里

其代码内容如下：

MainView.axaml：

```xml
<UserControl xmlns="https://github.com/avaloniaui"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
             xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
             xmlns:bunaylenerkeLailurlairfea="clr-namespace:KarhelearkuDemkunalhaw"
             mc:Ignorable="d" d:DesignWidth="800" d:DesignHeight="450"
             x:Class="KarhelearkuDemkunalhaw.Views.MainView">
  <Grid x:Name="RootGrid">
      <TextBlock HorizontalAlignment="Center" VerticalAlignment="Center" Text="Hello World"></TextBlock>

      <StackPanel Orientation="Horizontal" HorizontalAlignment="Center" VerticalAlignment="Center" Margin="10,300,10,10">
          <Button x:Name="TakeSnapshotButton" Click="TakeSnapshotButton_OnClick"
                  ToolTip.Tip="Take Snapshot to 1.png file. And you can find the blur text.">Take Snapshot</Button>
          <Button x:Name="TakeSnapshotWithFixButton" Margin="10,0,10,0" Click="TakeSnapshotWithFixButton_OnClick"
                  ToolTip.Tip="Take Snapshot to 2.png file. And you can find the clear text.">Take Snapshot with fix</Button>
      </StackPanel>
  </Grid>
</UserControl>
```

MainView.axaml.cs:

```csharp
using Avalonia.Animation;
using Avalonia.Controls;
using Avalonia.Media;
using Avalonia.Styling;

using System;
using System.IO;
using Avalonia;
using Avalonia.Interactivity;
using Avalonia.Media.Imaging;
using Avalonia.Threading;

namespace KarhelearkuDemkunalhaw.Views;

public partial class MainView : UserControl
{
    public MainView()
    {
        InitializeComponent();
    }

    private void TakeSnapshotButton_OnClick(object? sender, RoutedEventArgs e)
    {
        var mainView = this;

        var renderTargetBitmap =
            new RenderTargetBitmap(new PixelSize((int) mainView.Bounds.Width, (int) mainView.Bounds.Height), new Vector(96, 96));
        renderTargetBitmap.Render(mainView);

        var file = Path.Join(AppContext.BaseDirectory, "1.png");
        renderTargetBitmap.Save(file, 100);
    }

    private void TakeSnapshotWithFixButton_OnClick(object? sender, RoutedEventArgs e)
    {
        RootGrid.Background = Brushes.White;
        Dispatcher.UIThread.InvokeAsync(() =>
        {
            var mainView = this;

            var renderTargetBitmap =
                new RenderTargetBitmap(new PixelSize((int) mainView.Bounds.Width, (int) mainView.Bounds.Height), new Vector(96, 96));
            renderTargetBitmap.Render(mainView);

            var file = Path.Join(AppContext.BaseDirectory, "2.png");
            renderTargetBitmap.Save(file, 100);

            // 截图完成，重新设置到透明色。下次使用 TakeSnapshotButton 按钮截图时，依然文本模糊
            RootGrid.Background = Brushes.Transparent;
        });
    }
}
```

请运行项目，依次点击 TakeSnapshotButton 和 TakeSnapshotWithFixButton 按钮，观察程序生成的 1.png 和 2.png 文件

你可以看见 1.png 里面的文本都是模糊的，如上图所示

而 2.png 里面的文本是清晰的，如下图所示。证明了只要给 RootGrid 设置非透明的背景色，则能够让渲染正常工作

<!-- ![](image/Avalonia 已知问题 使用 RenderTargetBitmap 截图文本模糊/Avalonia 已知问题 使用 RenderTargetBitmap 截图文本模糊1.png) -->
![](https://img2024.cnblogs.com/blog/1080237/202507/1080237-20250711073033225-1137829508.png)

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/86ff9a4bb65e2775fb71d34f34283b68b5ab1605/AvaloniaIDemo/KarhelearkuDemkunalhaw) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/blob/86ff9a4bb65e2775fb71d34f34283b68b5ab1605/AvaloniaIDemo/KarhelearkuDemkunalhaw) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 86ff9a4bb65e2775fb71d34f34283b68b5ab1605
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 86ff9a4bb65e2775fb71d34f34283b68b5ab1605
```

获取代码之后，进入 AvaloniaIDemo/KarhelearkuDemkunalhaw 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )
