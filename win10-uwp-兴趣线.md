# win10 uwp 兴趣线

本文讲的是如何去做一个时间轴样子的东西但我们放的不一定是时间，可能是我们的时间。我把它放在我的CSDN阅读，我的界面做出来很差，但是应该读者能做出很漂亮的。
<!--more-->
<!-- CreateTime:2019/9/2 12:57:38 -->


<div id="toc"></div>

## 行间距

我们在ViewModel写一个ObservableCollection，我把它名字叫Str，因为这个是随意的，我们不需要给他他值。

然后在我们的界面，用ListView。

要我们的ListView的Item有和ListView一样的宽度可以简单设置ItemContainerStyle

```xml
                        <ListView.ItemContainerStyle>
                            <Style TargetType="ListViewItem">
                                <Setter Property="HorizontalContentAlignment"
                                        Value="Stretch" />
                            </Style>
                        </ListView.ItemContainerStyle>

```

这时，我们在我们的模板写Grid的背景为black

```xml
           <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid Background="Black">
                        <TextBlock Text="123"></TextBlock>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>

```

可以看到行间隔，UWP行间距其实是我们没有设置垂直，因为开始是Center

![这里写图片描述](http://img.blog.csdn.net/20160925105158895)

我们可以使用设置他和宽度一样，其实这里我说错，是水平布局

```xml
                    <Setter Property="VerticalContentAlignment" Value="Stretch"></Setter>


```

![这里写图片描述](http://img.blog.csdn.net/20160925105437116)

全部代码

```xml
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <ListView ItemsSource="{x:Bind View.Str}">
            <ListView.ItemContainerStyle>
                <Style TargetType="ListViewItem">
                    <Setter Property="HorizontalContentAlignment"
                                        Value="Stretch" />
                    <Setter Property="VerticalContentAlignment" Value="Stretch"></Setter>
                </Style>
            </ListView.ItemContainerStyle>
            
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid Background="Black">
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
    </Grid>

```

## 兴趣线

我要想说下兴趣线是什么，这个很多人叫时间轴，UWP时间轴的做法其实就是一个ListView。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=4c98bf6dbf97f7109aa4716dfb5430d9)

这是我的CSDN博客阅读，虽然界面做的很渣，但是我想说这个左边的线就是我们要做的。

大家可以看到左边的，其实就是两条线和两个圆。

我们先用Rectangle来画我们第一个线，我们需要他就在中间

```csharp
HorizontalAlignment="Center"
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=c536e3bfe4faa3e64a2fb388afd6f5ea)
放在一个Grid的中间就是我们设置水平为center

然后我们在右边放一个Rectangle，如何在Grid放的是一半，我们可以在Grid放一个Grid，使用ColumnDefinitions水平把Grid分左右

```xml
<Grid.ColumnDefinitions>
             <ColumnDefinition></ColumnDefinition>
             <ColumnDefinition></ColumnDefinition>
 </Grid.ColumnDefinitions>

```

然后把Rectangle放在右边，注意要设置他的高度和宽度

```xml
<Rectangle Grid.Column="1"/>

```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=bf9139aa65691dcc99c5f915fd48f762)

开始设计我们不知道宽度和高度的值，每次修改都需要改好多个，那么我们如何就修改一个？我们可以使用常亮，也就是我们的Resource

```xml
                        <ListView.Resources>
                            <!--<x:Double x:Key="LeftListWidth">100</x:Double>-->
                            <GridLength x:Key="LeftListWidth" >100</GridLength>
                            <x:Double x:Key="RectangleWidth">6</x:Double>
                            <SolidColorBrush x:Key="RectangleColor" Color="#FFDA3E3E"></SolidColorBrush>
                            <!--<SolidColorBrush x:Key="VerticalRectangleColor"  ></SolidColorBrush>-->
                            <x:Double x:Key="EllipseWidth">30</x:Double>
                        </ListView.Resources>

```

我首先是定义了左边的宽度，也就是放圆圈的那个Grid宽度，然后定义Rectangle的宽度，作为垂直的Rectangle就是他宽度，水平的就是他高度。

然后定义它的颜色，定义了Ellipse的宽度。

画完了线我们需要画圆

在Grid放一个Grid，然后画一个圆，注意这个圆Stroke为颜色，然后Fill背景颜色

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=8854fb16de8fc8bc02990fe9c84763d0)

这样就可以让后面的Rectangle被圆不看
![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=c54ef276e7bc53dd6f5fc40df7bd4be2)

然后我们需要在我们的圆再一个小的

```xml
      <Ellipse Width="10" Height="10"
               Fill="{StaticResource RectangleColor}"></Ellipse>


```

这样就是我们的画法

全部代码

```xml
<Grid Margin="0,0,0,0">
                                        <Rectangle Margin="0,0,0,0"
                                                   Width="{StaticResource RectangleWidth}"
                                                   Fill="{StaticResource RectangleColor}" 
                                                   VerticalAlignment="Stretch"
                                                   HorizontalAlignment="Center"></Rectangle>
                                        <Grid >
                                            <Grid.ColumnDefinitions>
                                                <ColumnDefinition></ColumnDefinition>
                                                <ColumnDefinition></ColumnDefinition>
                                            </Grid.ColumnDefinitions>
                                            <Rectangle Grid.Column="1"
                                                       Fill="{StaticResource RectangleColor}"
                                                       Height="{StaticResource RectangleWidth}"></Rectangle>
                                        </Grid>
                                        <Grid Width="{StaticResource EllipseWidth}" Height="{StaticResource EllipseWidth}">
                                            <Ellipse Stroke="{StaticResource RectangleColor}" StrokeThickness="6"
                                                     Fill="White"></Ellipse>
                                            <Ellipse Width="10" Height="10"
                                                     Fill="{StaticResource RectangleColor}"></Ellipse>
                                        </Grid>
                                    </Grid>

```

做完左边，就去做右边，右边其实就是一个Border里面一个TextBlock，当然里面最好把TextBlock换Grid，注意Margin，这样就好啦。

源代码：https://github.com/lindexi/csdn-uwp

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


