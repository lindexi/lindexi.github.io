# win10 UWP 动画

本文告诉大家如何写同一个简单的动画。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<!-- csdn -->

## 动画入门

本文开始写一个简单的动画，只是移动矩形作为本文的例子。

在 UWP 移动元素的动画，可以使用 RenderTransform 移动，然后使用动画修改 RenderTransform 进行动画。关于元素移动，请看 [win10 uwp 拖动控件](http://lindexi.oschina.io/lindexi/post/win10-uwp-%E6%8B%96%E5%8A%A8%E6%8E%A7%E4%BB%B6/)

首先写一个简单的界面，只有一个矩形

```csharp
        <Grid x:Name="Bret">
            <Rectangle x:Name="Rolernd" Width="100" Height="100" Fill="#FFa2a2a2"
                       HorizontalAlignment="Center" VerticalAlignment="Top">
              
            </Rectangle>
        </Grid>
```

然后在矩形加上  RenderTransform ，作为移动

```csharp
        <Grid x:Name="Bret">
            <Rectangle x:Name="Rolernd" Width="100" Height="100" Fill="#FFa2a2a2"
                       HorizontalAlignment="Center" VerticalAlignment="Top">
                <Rectangle.RenderTransform>
                    <TranslateTransform></TranslateTransform>
                </Rectangle.RenderTransform>
            </Rectangle>
        </Grid>
```

但是因为动画不可以直接播放，先加个按钮，点击按钮时播放。

```csharp
        <Button Content="确定" Click="Button_OnClick"></Button>

```

现在界面看起来就是如下

```csharp
        <Grid x:Name="Bret">
            <Rectangle x:Name="Rolernd" Width="100" Height="100" Fill="#FFa2a2a2"
                       HorizontalAlignment="Center" VerticalAlignment="Top">
                <Rectangle.RenderTransform>
                    <TranslateTransform></TranslateTransform>
                </Rectangle.RenderTransform>
            </Rectangle>
        </Grid>
        
        <Button Content="确定" Click="Button_OnClick"></Button>
```

开始写动画。

需要在 Resources 写动画，于是在页面写上动画，这里使用 DoubleAnimation ，注意需要 Storyboard.TargetName 说明动画的元素。因为 RenderTransform 动画有点难，于是我才写他。

```csharp
   <Page.Resources>
        <Storyboard x:Key="Filiberto">
            <DoubleAnimation Storyboard.TargetName="Rolernd" Storyboard.TargetProperty="(Rectangle.RenderTransform).(TranslateTransform.X)"
                             From="0" To="100" Duration="0:0:1">
                
            </DoubleAnimation>
        </Storyboard>
    </Page.Resources>
```

其中的 From 就是修改数值从多少开始，To 就是到多少，后面的属性就是动画时间。写完动画就在按钮写播放，请看代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var s = (Storyboard)Resources["Filiberto"];
            s.Begin();
        }
```

点击F5就是可以看到界面出现矩形，按下按钮就向右走

[win10 uwp 使用动画修改 Grid column 的宽度](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E4%BD%BF%E7%94%A8%E5%8A%A8%E7%94%BB%E4%BF%AE%E6%94%B9-Grid-column-%E7%9A%84%E5%AE%BD%E5%BA%A6/)


http://www.cnblogs.com/KeithWang/archive/2012/03/30/2425588.html

http://www.cnblogs.com/lin277541/p/4882000.html

http://www.cnblogs.com/lin277541/p/5059489.html

http://www.cnblogs.com/lin277541/p/5064899.html

http://www.cnblogs.com/lin277541/p/5068410.html


http://www.cnblogs.com/lin277541/p/5074697.html

http://www.cnblogs.com/lin277541/p/4881188.html


http://www.cnblogs.com/lin277541/p/4876140.html

http://www.cnblogs.com/mantgh/p/4437892.html

http://www.datiancun.net/thread-1586-1-1.html


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。