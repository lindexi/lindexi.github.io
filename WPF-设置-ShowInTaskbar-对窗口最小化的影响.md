
# WPF 设置 ShowInTaskbar 对窗口最小化的影响

在 WPF 中，如果设置了 ShowInTaskbar 为 False 那么窗口将不会在任务栏显示。此时如果设置窗口最小化，那么窗口将会收起来作为没有任务栏时的显示方法

<!--more-->


<!-- 发布 -->

如下面代码

```xml
<Window x:Class="BekairlilearDujalgereno.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:BekairlilearDujalgereno"
        mc:Ignorable="d" 
        ShowInTaskbar="False"
        Title="MainWindow" Height="450" Width="800">
  <Grid>
    <Button HorizontalAlignment="Center" VerticalAlignment="Center" Content="最小化" Click="Button_OnClick" />
  </Grid>
</Window>
```

通过 `ShowInTaskbar="False"` 设置窗口不在任务栏显示

在点击按钮的时候，设置最小化

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            WindowState = WindowState.Minimized;
        }
```

此时窗口将会到左下角，作为一个只有标题栏的窗口存在，如下图

<!-- ![](image/WPF 设置 ShowInTaskbar 对窗口最小化的影响/WPF 设置 ShowInTaskbar 对窗口最小化的影响0.png) -->

![](http://image.acmx.xyz/lindexi%2F2021318163155400.jpg)

如果不想要这个标题栏窗口，那么除非不要使用最小化，而是使用 `Visibility = Visibility.Collapsed` 或者 Hide 方法，如下面代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            Visibility = Visibility.Collapsed;
            //WindowState = WindowState.Minimized;
        }
```

此时点击按钮就可以让窗口消失，当然，使用 Hide 也是相同的效果

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/9c68faa6/BekairlilearDujalgereno ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/9c68faa6/BekairlilearDujalgereno ) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。