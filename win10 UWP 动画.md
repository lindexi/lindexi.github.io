# win10 UWP 动画

本文告诉大家如何写同一个简单的动画。

<!--more-->
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