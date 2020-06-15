# win10 uwp Fluent Design System 实践

本文告诉大家我收集的一些 Fluent Design System 设计，希望能给大家一些帮助

需要知道 Fluent Design System 是微软在最近提出的，有  Light、Depth、Motion、Material、Scale 几个理念，Fluent Design System的简称是 FDS。如何设计请看 [Build Amazing Apps with Fluent Design ](https://channel9.msdn.com/Events/Build/2017/B8034 )

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

下面是我从系统收集的界面

## 设置

无边框的设计按钮在这里使用，可以看到无边框的按钮会在之后很多使用，如果大家在设计按钮的时候，试试直接使用。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201812585415.jpg)

无边框按钮请看 [[UWP]使用Reveal - dino.c - 博客园](http://www.cnblogs.com/dino623/p/Reveal.html )，里面的代码直接拿就可以做出无边框的按钮，下面是 dino 大神做出的界面

![](https://images2017.cnblogs.com/blog/38937/201801/38937-20180118201118240-1586298023.gif )

需要知道 dino 大神的按钮使用的设计是 Reveal，Reveal 是光照效果，给应用带来深度的交互。更多关于Reveal，请到[Reveal highlight ](https://docs.microsoft.com/en-us/windows/uwp/design/style/reveal )

## 开始

在开始菜单也使用fds，主要是ListView 使用。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201812591821.jpg)

如何在软件使用毛玻璃，请看 [win10 uwp 毛玻璃 - 林德熙](https://lindexi.oschina.io/lindexi/post/win10-uwp-%E6%AF%9B%E7%8E%BB%E7%92%83.html )

## 创建空白界面

下面来告诉大家如何创建一个空白页面

![](http://image.acmx.xyz/65fb6078-c169-4ce3-cdd9-e35752d07be0%2F20183421123.jpg)

首先创建一个页面，随意的命名，然后在主页面跳转到这个页面，因为这是用于测试的。

```csharp
            var frame = new Frame();

            Content = frame;

            frame.Navigate(typeof(DrowilHuwfevfPage));
```

重写 DrowilHuwfevfPage 的 OnNavigatedTo 设置标题栏

```csharp
            SystemNavigationManager telTtxxskne = SystemNavigationManager.GetForCurrentView();
            telTtxxskne.BackRequested += BackRequested;

            telTtxxskne.AppViewBackButtonVisibility =
                AppViewBackButtonVisibility.Visible;
```

上面的代码就是打开后退按钮，后退按钮的大小大概是高度30，宽度50

```csharp
            CoreApplication.GetCurrentView().TitleBar.ExtendViewIntoTitleBar = true;

```

上面代码是扩展页面到标题栏，现在就不存在标题栏了，可以完全自己定义

```csharp
          var dmbyzkfscDycoue = ApplicationView.GetForCurrentView();

            dmbyzkfscDycoue.TitleBar.BackgroundColor = Colors.Black;

            dmbyzkfscDycoue.TitleBar.ButtonBackgroundColor = Colors.Transparent;
```

上面代码设置最小化按钮的背景，但是`dmbyzkfscDycoue.TitleBar.BackgroundColor`无论设置为什么都没有什么用

然后去到页面，使用毛玻璃

```csharp
    <Grid Background="{ThemeResource SystemControlChromeMediumAcrylicWindowMediumBrush }">
       
    </Grid>
```

这时我找到的颜色，透明度是60%，推荐使用这个作为背景

然后写一个标题

```csharp
    <Grid Background="{ThemeResource SystemControlChromeMediumAcrylicWindowMediumBrush }">
        <Grid.RowDefinitions>
            <RowDefinition Height="30"/>
            <RowDefinition Height="114*"/>
        </Grid.RowDefinitions>
        <Grid Grid.Row="0">
            <TextBlock Margin="50,0,0,0" VerticalAlignment="Center">图床</TextBlock>
        </Grid>
    </Grid>
```

现在的页面就写好了

当然后续 Fluent Design System 也没有后续了，因此本文就这样

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。