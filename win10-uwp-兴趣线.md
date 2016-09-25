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