# win10 uwp 装机必备应用 含源代码

zhxilin大神在[文章](http://www.cnblogs.com/zhxilin/p/4819372.html)说到了使用`await Windows.System.Launcher.LaunchUriAsync(new Uri(uri));`打开应用商店

我想到了装机必备的一个软件，就是通过上面的代码来推荐应用给大家

大概界面请看下面

![这里写图片描述](http://img.blog.csdn.net/20151203201502839)

界面不好看求轻喷，毕竟只是这个界面只是告诉大家这个功能如何做

<!--more-->
<!-- CreateTime:2018/8/9 9:07:31 -->


我设计了 MainPage.xaml 拥有两个 Frame 和单例model

从 <https://www.microsoft.com/zh-cn/store/top-free/apps/pc> 得到软件图片，如下面图片就是拿到 QQ 的图片

![这里写图片描述](http://img.blog.csdn.net/20151203201654326)

为了在用户点击的时候可以跳转到商店，可以设置点击的是按钮，按钮Button可以设置Content为Grid所以就可以设置图片和文字，请看下面代码。我特意用 QQ 的图片，文字写了 `搜狐视频` ，点击这个按钮可以跳转到商店

```
                        <Button.Content>
                                <Grid>
                                    <Grid.RowDefinitions>
                                        <RowDefinition Height="auto"/>
                                        <RowDefinition Height="auto"/>
                                    </Grid.RowDefinitions>
                                    <Image Source="ms-appx:///Assets/QQ.png" Grid.Row="0" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                                    <TextBlock Text="搜狐视频" Grid.Row="1" HorizontalAlignment="Center" />
                                </Grid>
                            </Button.Content>
                        </Button>
```

button 设置大小和图片一样，就可以把图片填到button作为按钮的图片

点击按钮通过先获得应用软件 ProductId 这个应用的 id 就是通过商店的链接最后的字符串找到的，如 QQ 的应用链接请看下面，可以看到最后的字符串就是他的 id 通过 这个id 就可以跳转到商店

![这里写图片描述](http://img.blog.csdn.net/20151203201930421)

下面就是跳转到商店的代码

```
            string uri = "ms-windows-store://pdp/?ProductId=9wzdncrfj1ps";
            await Windows.System.Launcher.LaunchUriAsync(new Uri(uri));
```

在按钮写` <Button Click="QQ_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0"/>`就可以点击跳转应用商店

![这里写图片描述](http://img.blog.csdn.net/20151203202121128)

因为这个页面不是写在主页面，在主页面就放了一个 Frame 需要跳转到刚才写的按钮所在页面，例如主页面的是 `chatcommunicationframe` 按钮所在的页面是`chatcommunication` 在页面跳转到QQ页面可以使用下面代码`chatcommunicationframe.Navigate(typeof(chatcommunication));` 在页面跳转不建议使用这个方法，建议使用[MVVM(https://blog.csdn.net/lindexi_gd/article/details/68059121 )来做页面跳转

刚才的代码是写固定的连接，建议差不多的代码使用一个函数来做，请看下面代码

```
        public async Task OpenWindowsapp(string productId)
        {
            string uri = $"ms-windows-store://pdp/?ProductId={productId}";
            await Windows.System.Launcher.LaunchUriAsync(new Uri(uri));
        }
```

可以在点击按钮时调用这个函数

```
        private void Souhu_Click(object sender , RoutedEventArgs e)
        {            
            string productId = "9wzdncrfhvq0";
            _model.OpenWindowsapp(productId );
        }
```

这个软件的界面用到的文件请看下面


 - chatcommunication.xaml
 - movie.xaml
 - model.cs
 - MainPage.xaml

主界面代码

```xaml
<Page
    x:Class="classifyapp.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:classifyapp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Grid.RowDefinitions>
            <RowDefinition Height="auto"/>
            <RowDefinition Height="auto"/>
        </Grid.RowDefinitions>
        <!-- 建议使用 x:Name 而不是  Name ，建议控件的命名使用 ChatcommunicationFrame 而不是第一个字符小写，因为控件是属性 -->
        <Frame Name="chatcommunicationframe" Grid.Row="0" Margin="10,10,10,10"/>
        <Frame Name="movieframe" Grid.Row="1" Margin="10,10,10,10"/>
        
    </Grid>
</Page>

```

chatcommunication.xaml:

```xaml
<Page
    x:Class="classifyapp.view.chatcommunication"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:classifyapp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Border>
            <Border.Background>
                <LinearGradientBrush EndPoint="0.5,1" StartPoint="0.5,0">
                    <GradientStop Color="#FF6369EB" Offset="0"/>
                    <GradientStop Color="#FFFAFBFF" Offset="1"/>
                </LinearGradientBrush>
            </Border.Background>
            <Grid>
            <Grid.RowDefinitions>
                <RowDefinition Height="auto"/>
                <RowDefinition />
            </Grid.RowDefinitions>
            <TextBlock Text="聊天" Grid.Row="0" Margin="10,10,10,10"/>
            <Grid Grid.Row="1">
                <GridView >
                    <Button Click="QQ_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0">
                        <Button.Content>
                            <Image Source="ms-appx:///Assets/QQ.png" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                        </Button.Content>
                    </Button>

                </GridView>
            </Grid>
        </Grid>
      </Border>
    </Grid>
</Page>

```

movie.xaml

```xaml
<Page
    x:Class="classifyapp.view.movie"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:classifyapp"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">

    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Border>
            <Border.Background>
                <LinearGradientBrush EndPoint="0.5,1" StartPoint="0.5,0">
                    <GradientStop Color="#FF6369EB" Offset="0"/>
                    <GradientStop Color="#FFFAFBFF" Offset="1"/>
                </LinearGradientBrush>
            </Border.Background>
            <Grid>
                <Grid.RowDefinitions>
                    <RowDefinition Height="auto"/>
                    <RowDefinition />
                </Grid.RowDefinitions>
                <TextBlock Text="视频" Grid.Row="0" Margin="10,10,10,10"/>
                <Grid Grid.Row="1">
                    <GridView >
                        <Button Click="souhu_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0" >
                            <Button.Content>
                                <Grid>
                                    <Grid.RowDefinitions>
                                        <RowDefinition Height="auto"/>
                                        <RowDefinition Height="auto"/>
                                    </Grid.RowDefinitions>
                                    <Image Source="ms-appx:///Assets/搜狐.png" Grid.Row="0" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                                    <TextBlock Text="搜狐视频" Grid.Row="1" HorizontalAlignment="Center" />
                                </Grid>
                            </Button.Content>
                        </Button>

                        <Button Click="blibli_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0">
                            <Button.Content>
                                <Grid>
                                    <Grid.RowDefinitions>
                                        <RowDefinition Height="auto"/>
                                        <RowDefinition Height="auto"/>
                                    </Grid.RowDefinitions>
                                    <Image Source="ms-appx:///Assets/blibli.png" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                                    <TextBlock Text="搜狐视频" Grid.Row="1" HorizontalAlignment="Center" />
                                </Grid>
                            </Button.Content>
                        </Button>

                        <Button Click="manguo_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0">
                            <Button.Content>
                                <Grid>
                                    <Grid.RowDefinitions>
                                        <RowDefinition Height="auto"/>
                                        <RowDefinition Height="auto"/>
                                    </Grid.RowDefinitions>
                                    <Image Source="ms-appx:///Assets/芒果.png" Width="50" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                                    <TextBlock Text="芒果TV" Grid.Row="1" HorizontalAlignment="Center" />
                                </Grid>
                            </Button.Content>
                        </Button>

                        <Button Click="youku_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0">
                            <Button.Content>
                                <Grid>
                                    <Grid.RowDefinitions>
                                        <RowDefinition Height="auto"/>
                                        <RowDefinition Height="auto"/>
                                    </Grid.RowDefinitions>
                                    <Image Source="ms-appx:///Assets/优酷.png" Width="50" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                                    <TextBlock Text="优酷TV" Grid.Row="1" HorizontalAlignment="Center" />
                                </Grid>
                            </Button.Content>
                        </Button>

                        <Button Click="baofengyingyin_Click" Width="50" Height="50" Margin="10,10,10,10" Padding="0">
                            <Button.Content>
                                <Grid>
                                    <Grid.RowDefinitions>
                                        <RowDefinition Height="auto"/>
                                        <RowDefinition Height="auto"/>
                                    </Grid.RowDefinitions>
                                    <Image Source="ms-appx:///Assets/暴风影音.png" Width="50" ScrollViewer.VerticalScrollBarVisibility="Disabled" />
                                    <TextBlock Text="暴风影音" Grid.Row="1" HorizontalAlignment="Center" />
                                </Grid>
                            </Button.Content>
                        </Button>
                    </GridView>
                </Grid>
            </Grid>
        </Border>
    </Grid>
   
</Page>
```

没有使用比较多的东西，简单单例，按钮，frame，GridView，没有使用 bind，动画。界面
`margin`可以使用`"10"`，我都是使用`"10,10,10,10"`，虽然好多写法可以让代码变少，也不会容易出错。但是本文没有做这么多的东西，因为简单的代码需要很多知识，只是做一个可以看的东西，告诉大家这个软件可以怎么做。

虽然这个应该发布是不会的，但是也有一些想不开的开发者也许就发出来。我这里的代码只是博客用，建议不用直接使用。虽然知道了如何开发，但是一个软件不是只有技术就可以做出来，还需要运营，我没有这么多时间，所以就不想做。

这就是做出来的界面和功能

![这里写图片描述](http://img.blog.csdn.net/20151203205520862)

这个软件需要的技术是很少的，如果要做出一个装机必备的软件，除了上面说道的技术之外，还需要写爬虫，我就不想写这个模块。如果有谁要这个软件的代码，我挂个价格，给我 100 就好。

代码：<https://gitee.com/lindexi/lindexi_gd/tree/master/classifyapp>

参考：https://msdn.microsoft.com/en-us/library/windows/apps/mt228343.aspx

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
