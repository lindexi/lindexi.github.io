# WPF 给应用程序添加水印

我有一个应用，我想要在应用上添加水印，这个水印可以如何做

<!--more-->
<!-- CreateTime:2020/2/9 21:24:55 -->

<!-- 发布 -->

例如我有一个应用，我在主页面添加了功能页面，在功能页面的最上层需要一个水印，这个水印不能被用户点击到，例如我的功能页面是一个用户控件放在页面

```xml
<Window x:Class="NeachecihaDenemceweefai.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:NeachecihaDenemceweefai"
        mc:Ignorable="d"
        Title="MainWindow" Height="300" Width="600">
    <Grid>
        <local:KekalcigarjeyikelRijurjeeyaira></local:KekalcigarjeyikelRijurjeeyaira>
    </Grid>
</Window>

```

可以通过在最顶层元素，也就是上面代码的 Grid 里面添加一个水印控件，如一张图或文字，本文这里使用文字，放在容器最后面，为什么需要放在最后面？因为放在后面的显示在最上层

然后设置水印的命中，请看下面代码

```xml
        <TextBlock Text="林德熙是逗比" Margin="10,10,10,10" IsHitTestVisible="True"></TextBlock>
```

上面代码核心是`IsHitTestVisible="True"`通过这个属性就可以让用户无法点中这个文本

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/38bff1c3053f834795808e310aae9fe5cb7968f0/NeachecihaDenemceweefai) 欢迎小伙伴访问

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
