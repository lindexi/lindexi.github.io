# win10 uwp 通过 Win2d 完全控制笔迹绘制逻辑

本文来告诉大家如何通过 Win2d 完全控制笔迹绘制逻辑，本文适合用来实现复杂的自定义逻辑，可以完全控制笔迹的行为。包括在书写过程中切换模式，如进行手势擦除切换为橡皮擦模式

<!--more-->
<!-- CreateTime:2021/8/30 20:27:41 -->

<!-- 发布 -->
<!-- 标签：uwp,win2d,笔迹 -->

本文提供的方法适合用来做复杂的自定义，本文的方法的优点也是缺点。优点是啥都可以自己控制，缺点是啥都需要自己控制。需要自己处理笔迹的多笔同步问题，处理笔迹的长笔迹分段问题，处理笔迹的绘制问题，处理动态笔迹切换

本文提供的方法依然可以实现非常高性能的笔迹，比 WPF 最快的笔迹实现还要快，但需要自己处理好各个部分的逻辑，如动态笔迹和静态笔迹，笔迹分段等逻辑。本文提供的方法的性能依然不如只使用默认的 InkCanvas 快

## 界面

在开始之前，请先安装 Win2d 库，可参阅 [win10 uwp win2d 入门 看这一篇就够了](https://blog.lindexi.com/post/win10-uwp-win2d-%E5%85%A5%E9%97%A8-%E7%9C%8B%E8%BF%99%E4%B8%80%E7%AF%87%E5%B0%B1%E5%A4%9F%E4%BA%86.html ) 博客了解如何安装

在 XAML 界面上加上 `xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"` 命名空间，用来导入 Win2d 控件。界面如下

```xml
  <Grid Background="#565656">
    <canvas:CanvasControl x:Name="Canvas" Draw="Canvas_OnDraw"/>
    <InkCanvas x:Name="InkCanvas" />
  </Grid>
```

本文将使用一个 InkCanvas 放在 Win2d 的 CanvasControl 上层，让 InkCanvas 作为快速的事件接收层，让 Win2d 的 CanvasControl 作为实际的绘制层。其实，更好的界面框架是存放两个 Win2d 的 CanvasControl 分别用来存放动态笔迹和静态笔迹。如果自己实现的笔迹没有分动态笔迹和静态笔迹，那么可以忽略，本文为了简洁将不演示动态笔迹和静态笔迹的处理

此时的界面逻辑如下

```xml
<Page
  x:Class="KeanearkallhawDaherenenallyi.MainPage"
  xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
  xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
  xmlns:local="using:KeanearkallhawDaherenenallyi"
  xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
  xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
  mc:Ignorable="d"
  xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
  Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">

  <Grid Background="#565656">
    <canvas:CanvasControl x:Name="Canvas" Draw="Canvas_OnDraw"/>
    <InkCanvas x:Name="InkCanvas" />
  </Grid>
</Page>
```

## 初始化笔迹接收

在构造函数初始化笔迹的接收逻辑，通过 InkCanvas 进行快速的事件接收

```csharp
        private readonly InkSynchronizer _inkSynchronizer;

        public MainPage()
        {
            this.InitializeComponent();
            _inkSynchronizer = InkCanvas.InkPresenter.ActivateCustomDrying();
            InkCanvas.InkPresenter.SetPredefinedConfiguration(InkPresenterPredefinedConfiguration
                .SimpleMultiplePointer);
            InkCanvas.InkPresenter.InputDeviceTypes =
                CoreInputDeviceTypes.Touch | CoreInputDeviceTypes.Pen | CoreInputDeviceTypes.Mouse;
            InkCanvas.InkPresenter.UnprocessedInput.PointerMoved += UnprocessedInput_PointerMoved;
            InkCanvas.InkPresenter.InputProcessingConfiguration.Mode = InkInputProcessingMode.None;
            InkCanvas.InkPresenter.InputProcessingConfiguration.RightDragAction =
                InkInputRightDragAction.LeaveUnprocessed;
        }
```

以上的代码里面，只是监听了 UnprocessedInput 的 PointerMoved 事件，事实上需要监听更多的事件用来了解笔迹的绘制开始和完成逻辑。本文为了方便演示，就不详细写所有逻辑

以上各个部分逻辑的含义，请参阅 [win10 uwp 通过 win2d 画出笔迹](https://blog.lindexi.com/post/win10-uwp-%E9%80%9A%E8%BF%87-win2d-%E7%94%BB%E5%87%BA%E7%AC%94%E8%BF%B9.html )

## 收集笔迹

在 `UnprocessedInput_PointerMoved` 将是本文的核心逻辑，在这里通过事件参数了解到当前是哪个手指或笔触摸，以及通过 InkStrokeBuilder 将输入的点构造笔迹

```csharp
        private void UnprocessedInput_PointerMoved(InkUnprocessedInput sender, Windows.UI.Core.PointerEventArgs args)
        {
            var id = args.CurrentPoint.PointerId;
            // 需要根据 id 分开多个手指

            InkStrokeBuilder.SetDefaultDrawingAttributes(new InkDrawingAttributes()
            {
                Color = Colors.Blue,
                Size = new Size(5, 5)
            });

            _currentPointerList.AddRange(args.GetIntermediatePoints());
            _inkStroke = InkStrokeBuilder.CreateStrokeFromInkPoints(
                _currentPointerList.Select(t => new InkPoint(t.Position, t.Properties.Pressure)), Matrix3x2.Identity);

            Canvas.Invalidate();
        }

        private readonly List<PointerPoint> _currentPointerList = new List<PointerPoint>();
        private InkStroke _inkStroke;

        private InkStrokeBuilder InkStrokeBuilder { get; } = new InkStrokeBuilder();
```

以上代码没有编写的部分是了解当前是由哪个 id 的设备触发的事件，如有多个手指在触摸，那么不同的手指的 id 是不相同的。需要自己创建列表数组进行处理

另外，通过 InkStrokeBuilder 的 CreateStrokeFromInkPoints 创建的 InkStroke 是需要预先在 SetDefaultDrawingAttributes 设置绘制属性的，而不是在创建之后依然可以设置。另外上面代码只使用了一个 InkStroke 字段，实际上需要根据当前是否有多指触摸的需求，使用列表存放多个笔迹

本文以上代码通过 CreateStrokeFromInkPoints 创建是不包含笔迹分段的，也就是说在用户绘制一段长线，将会需要使用较多的计算资源创建笔迹。请在自己的产品逻辑里面，手动分开为多个不同的笔迹段，用来提升性能

上面代码通过调用 CanvasControl 的 Invalidate 让 Win2d 的画布重新绘制。重新绘制会进入 `Canvas_OnDraw` 方法，将在此方法绘制出笔迹

## 绘制笔迹

绘制笔迹的方法十分简单，调用 Win2d 的 DrawInk 方法传入笔迹即可

```csharp
        private void Canvas_OnDraw(CanvasControl sender, CanvasDrawEventArgs args)
        {
            if (_inkStroke != null)
            {
                args.DrawingSession.DrawInk(new []{_inkStroke});
            }
        }
```

为什么在 Win2d 的设计里面，是传入数组？原因是笔迹是需要分段的，多段笔迹可以一起绘制。另外，如果有笔迹分段，那么逻辑上就需要额外的转换为静态笔迹的功能，大概就是将一段连续的多段笔迹合成一段笔迹的过程。建议绘制动态笔迹和静态笔迹放在两个 Win2d 的 CanvasControl 里。这样也能提升笔迹的动态绘制性能，因为笔迹在绘制的时候需要不断调用 Win2d 的刷新，如果此时刷新的是一个只包含很少笔迹的动态笔迹层的画布，那每次刷新的性能就比较好

## 无限漫游

如果需要做无限漫游，可以使用 [CanvasVirtualControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html ) 做一个超级大的画布，同时只画出可见的范围

使用时需要自己转换坐标，可以在 InkStrokeBuilder 的 CreateStrokeFromInkPoints 方法传入缩放和平移的矩阵，此时创建出来的笔迹是包含了变换的

## 代码

本文所有代码放在[github](https://github.com/lindexi/lindexi_gd/tree/d33e26640f0108ae6bb29b90e7b189a14a92d624/KeanearkallhawDaherenenallyi) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/d33e26640f0108ae6bb29b90e7b189a14a92d624/KeanearkallhawDaherenenallyi) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin d33e26640f0108ae6bb29b90e7b189a14a92d624
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 KeanearkallhawDaherenenallyi 文件夹

## 参考

更多笔迹和触摸，请参阅 [触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
