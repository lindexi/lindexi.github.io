#win10 uwp 兴趣线

##行间距

我们在ViewModel写一个ObservableCollection，我把它名字叫Str，因为这个是随意的，我们不需要给他他值。

然后在我们的界面，用ListView。

要我们的ListView的Item有和ListView一样的宽度可以简单设置ItemContainerStyle

```
                        <ListView.ItemContainerStyle>
                            <Style TargetType="ListViewItem">
                                <Setter Property="HorizontalContentAlignment"
                                        Value="Stretch" />
                            </Style>
                        </ListView.ItemContainerStyle>

```

这时，我们在我们的模板写Grid的背景为black

```
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

```
                    <Setter Property="VerticalContentAlignment" Value="Stretch"></Setter>


```

![这里写图片描述](http://img.blog.csdn.net/20160925105437116)

全部代码

```
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

##兴趣线

我要想说下兴趣线是什么，这个很多人叫时间轴，UWP时间轴的做法其实就是一个ListView。

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=4c98bf6dbf97f7109aa4716dfb5430d9)

这是我的CSDN博客阅读，虽然界面做的很渣，但是我想说这个左边的线就是我们要做的。

大家可以看到左边的，其实就是两条线和两个圆。

我们先用Rectangle来画我们第一个线，我们需要他就在中间

```
HorizontalAlignment="Center"
```

![](http://jycloud.9uads.com/web/GetObject.aspx?filekey=c536e3bfe4faa3e64a2fb388afd6f5ea)
放在一个Grid的中间就是我们设置水平为center

然后我们在右边放一个Rectangle，如何在Grid放的是一半，我们可以在Grid放一个Grid，使用ColumnDefinitions水平把Grid分左右

```
<Grid.ColumnDefinitions>
             <ColumnDefinition></ColumnDefinition>
             <ColumnDefinition></ColumnDefinition>
 </Grid.ColumnDefinitions>

```

然后把Rectangle放在右边，注意要设置他的高度和宽度

```
<Rectangle Grid.Column="1"/>

```

开始设计我们不知道宽度和高度的值，每次修改都需要改好多个，那么我们如何就修改一个？我们可以使用常亮，也就是我们的Resource

```
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

