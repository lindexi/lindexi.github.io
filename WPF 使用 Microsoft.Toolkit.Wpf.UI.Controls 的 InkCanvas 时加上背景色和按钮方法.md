# WPF 使用 Microsoft.Toolkit.Wpf.UI.Controls 的 InkCanvas 时加上背景色和按钮方法

本文来告诉大家如何在 WPF 应用 HOST 了 UWP 的 InkCanvas 控件时，给 InkCanvas 控件设置背景色，加上按钮等业务功能的实现方法

<!--more-->
<!-- CreateTime:2021/8/17 21:19:53 -->


<!-- 发布 -->

在上一篇博客有告诉大家如何在 WPF 里面使用上 UWP 的 InkCanvas 控件，详细请看 [WPF 使用 Microsoft.Toolkit.Wpf.UI.Controls 的 InkCanvas 做高性能笔迹应用](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Microsoft.Toolkit.Wpf.UI.Controls-%E7%9A%84-InkCanvas-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%BA%94%E7%94%A8.html )

如果不想要打包为 MSIX 包，请参阅 [WPF 引用 UWP 控件 不打包为 MSIX 分发的方法](https://blog.lindexi.com/post/WPF-%E5%BC%95%E7%94%A8-UWP-%E6%8E%A7%E4%BB%B6-%E4%B8%8D%E6%89%93%E5%8C%85%E4%B8%BA-MSIX-%E5%88%86%E5%8F%91%E7%9A%84%E6%96%B9%E6%B3%95.html )

在开始之前，需要了解的是 UWP 的 InkCanvas 控件是没有背景色这个属性的，也就是说 UWP 的 InkCanvas 控件需要依靠外层的容器或者背后的元素给的颜色作为背景色。但是在 WPF 里面 HOST 了 UWP 的控件的方式，相当于将 UWP 作为一个窗口嵌入到 WPF 应用里面，这就意味着在 UWP 控件所在的范围，不能使用 WPF 的渲染，在此范围里面的元素都被 UWP 的控件挡住

因此为了给 UWP 的 InkCanvas 控件加上背景色，就需要采用在 WPF 里面 HOST 自定义的 UWP 控件的科技。这部分在官方博客有详细的说明，请参阅 [Host a custom WinRT XAML control in a WPF app using XAML Islands - Windows apps](https://docs.microsoft.com/en-us/windows/apps/desktop/modernize/host-custom-control-with-xaml-islands?WT.mc_id=WD-MVP-5003260 )

大概的做法就是新建两个 UWP 的项目，其中一个是 UWP 的空白应用项目，另一个是 UWP 的控件项目。让 UWP 的空白应用项目作为 UWP 执行入口，用于提供运行的支持。让 UWP 的控件项目作为实际的 UWP 自定义控件编写的项目，咱将在 UWP 的控件项目里面完成所有的自定义逻辑

如何创建项目和如何组织，还请参阅 [官方文档](https://docs.microsoft.com/en-us/windows/apps/desktop/modernize/host-custom-control-with-xaml-islands?WT.mc_id=WD-MVP-5003260 ) 本文这里就不多说了

回到如何给 UWP 的 InkCanvas 控件添加背景色的方法上，在新建的 UWP 控件项目里面，添加一个自定义的控件，如 CustomInkControl.xaml 控件

在这个控件里面的 XAML 添加如下代码

```xml
<UserControl
    x:Class="HinembereneabemWhejurnicelem.XamlIsland.CustomInkControl"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:HinembereneabemWhejurnicelem.XamlIsland"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d"
    d:DesignHeight="300"
    d:DesignWidth="400">

    <Grid Background="LightCoral">
        <InkCanvas x:Name="InkCanvas" Loaded="InkCanvas_OnLoaded"></InkCanvas>
    </Grid>
</UserControl>
```

为了能在鼠标下进行绘制，在 `InkCanvas_OnLoaded` 设置支持鼠标

```csharp
        private void InkCanvas_OnLoaded(object sender, RoutedEventArgs e)
        {
            InkCanvas.InkPresenter.InputDeviceTypes = CoreInputDeviceTypes.Mouse| CoreInputDeviceTypes.Touch;
        }
```

接着如 [官方文档](https://docs.microsoft.com/en-us/windows/apps/desktop/modernize/host-custom-control-with-xaml-islands?WT.mc_id=WD-MVP-5003260 ) 的方法，在 WPF 里面使用刚才创建的控件

```xml
<Window x:Class="LaykearduchuNachairgurharhear.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:LaykearduchuNachairgurharhear"
        xmlns:controls="clr-namespace:Microsoft.Toolkit.Wpf.UI.Controls;assembly=Microsoft.Toolkit.Wpf.UI.Controls"
        xmlns:xaml="clr-namespace:Microsoft.Toolkit.Wpf.UI.XamlHost;assembly=Microsoft.Toolkit.Wpf.UI.XamlHost"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
  <Grid Background="Gray">
    <xaml:WindowsXamlHost InitialTypeName="HinembereneabemWhejurnicelem.XamlIsland.CustomInkControl" ChildChanged="WindowsXamlHost_ChildChanged" />
  </Grid>
</Window>
```

很简单的代码即可完成

以上的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b2aa15343108fa5619bd2605c28085eb3cd6023d/LaykearduchuNachairgurharhear) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/b2aa15343108fa5619bd2605c28085eb3cd6023d/LaykearduchuNachairgurharhear) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin b2aa15343108fa5619bd2605c28085eb3cd6023d
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 LaykearduchuNachairgurharhear 文件夹

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

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
