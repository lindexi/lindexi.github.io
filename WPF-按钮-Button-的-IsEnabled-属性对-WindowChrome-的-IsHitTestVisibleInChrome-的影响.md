
# WPF 按钮 Button 的 IsEnabled 属性对 WindowChrome 的 IsHitTestVisibleInChrome 的影响

在 WPF 里面，默认标题栏的交互相对复杂，如按钮没有设置 WindowChrome.IsHitTestVisibleInChrome 属性为 True 那按钮是拿不到点击事件的。本文来告诉大家按钮 Button 的 IsEnabled 属性对 WindowChrome 的 IsHitTestVisibleInChrome 的影响

<!--more-->


<!-- CreateTime:2020/12/17 19:37:53 -->

<!-- 发布 -->

在 WPF 中的默认交互是点击标题栏的时候，如果是双击标题栏，那么将会进入最大化窗口。而如果在标题栏放一个按钮，此时按钮默认是没有收到点击的，详细请看 [WPF 非客户区的触摸和鼠标点击响应](https://blog.lindexi.com/post/WPF-%E9%9D%9E%E5%AE%A2%E6%88%B7%E5%8C%BA%E7%9A%84%E8%A7%A6%E6%91%B8%E5%92%8C%E9%BC%A0%E6%A0%87%E7%82%B9%E5%87%BB%E5%93%8D%E5%BA%94.html )

如果想要让按钮能收到点击，需要使用 [WPF 非客户区的触摸和鼠标点击响应](https://blog.lindexi.com/post/WPF-%E9%9D%9E%E5%AE%A2%E6%88%B7%E5%8C%BA%E7%9A%84%E8%A7%A6%E6%91%B8%E5%92%8C%E9%BC%A0%E6%A0%87%E7%82%B9%E5%87%BB%E5%93%8D%E5%BA%94.html ) 的方法，让 WPF 的按钮加上 WindowChrome.IsHitTestVisibleInChrome 属性

此时点击按钮的时候，如果是在标题栏的地方，是让按钮收到点击。此时双击标题栏的按钮不会让窗口最大化，但如果此时的按钮设置 IsEnabled="False"  那么此时双击将依然让窗口最大化

而有趣的是如果在按钮 A 设置了 WindowChrome.IsHitTestVisibleInChrome 属性为 True 同时 IsEnabled="True" 然后在按钮 A 上方再放一个按钮 B 设置 IsEnabled="False" 那么此时双击将依然让窗口最大化

```xml
<Window x:Class="BeehijemwaboHaihafobe.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:BeehijemwaboHaihafobe"
        mc:Ignorable="d"
        Title="MainWindow" WindowStyle="None" Height="450" Width="800">
  <WindowChrome.WindowChrome>
    <WindowChrome CaptionHeight="20" />
  </WindowChrome.WindowChrome>
  <Grid>
    <Grid.ColumnDefinitions>
      <ColumnDefinition />
      <ColumnDefinition />
    </Grid.ColumnDefinitions>
    <Button Grid.ColumnSpan="2" WindowChrome.IsHitTestVisibleInChrome="True" />
    <Button IsEnabled="False" />
  </Grid>
</Window>
```

上面代码运行的时候，将界面分为两列，此时尝试双击两边的标题栏。尽管第一个按钮是覆盖整个窗口大小的，同时设置了 WindowChrome.IsHitTestVisibleInChrome 属性，但是被第二个按钮覆盖的地方，双击标题栏会让窗口全屏

因此在 WPF 中，是否禁用标题栏的默认行为，是根据命中的元素决定的。此时在相同的容器内的其他元素，不会再被路由

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/df12af70d3ca6cb496e67a29ffee3ec52443d354/BeehijemwaboHaihafobe)欢迎小伙伴访问






<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。