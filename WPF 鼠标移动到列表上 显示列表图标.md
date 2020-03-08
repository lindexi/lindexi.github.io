# WPF 鼠标移动到列表上 显示列表图标

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->


<div id="toc"></div>

在列表新建一个图标，添加 Visibility

`Visibility="{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=IsMouseOver, Converter={StaticResource BooleanToVisibilityConverter}}"`

我这里用的是 TextBlock ，鼠标移动就会显示

```csharp
        <ListView AlternationCount="10">
            <ListView.ItemTemplate>
                <DataTemplate>
                    <Grid>
                        <Grid.ColumnDefinitions>
                            <ColumnDefinition Width="20"></ColumnDefinition>
                            <ColumnDefinition Width="Auto"></ColumnDefinition>
                            <ColumnDefinition ></ColumnDefinition>
                        </Grid.ColumnDefinitions>
                        <TextBlock Text="{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=(ItemsControl.AlternationIndex),Mode=OneWay,Converter={StaticResource NumberAddOne}}"></TextBlock>
                        <TextBlock Grid.Column="1" Text="点击"
                                   Visibility="{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=IsMouseOver, Converter={StaticResource BooleanToVisibilityConverter}}"></TextBlock>
                        <TextBlock Grid.Column="2" Text="lindexi"></TextBlock>
                    </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>
```


![](http://image.acmx.xyz/%E9%BC%A0%E6%A0%87%E7%A7%BB%E5%8A%A8%E5%88%B0%E5%88%97%E8%A1%A8%E4%B8%8A.gif)

获取当前列表项，使用`{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=(ItemsControl.AlternationIndex),Mode=OneWay,Converter={StaticResource NumberAddOne}}`。注意 AlternationCount 如果没设不会显示


```xml
        <ListView AlternationCount="10">
            <ListViewItem>
                <Grid>
                    <Grid.ColumnDefinitions>
                        <ColumnDefinition Width="20"></ColumnDefinition>
                        <ColumnDefinition ></ColumnDefinition>
                    </Grid.ColumnDefinitions>
                    <TextBlock Text="{Binding RelativeSource={RelativeSource AncestorType=ListBoxItem}, Path=(ItemsControl.AlternationIndex),Mode=OneWay,Converter={StaticResource NumberAddOne}}"></TextBlock>
                    <TextBlock Grid.Column="1" Text="lindexi"></TextBlock>
                </Grid>
            </ListViewItem>
        </ListView>

```


```csharp
    public class NumberAddOne : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, CultureInfo culture)
        {
            var index = (int)value;
            return index + 1;
        }

        public object ConvertBack(object value, Type targetType, object parameter, CultureInfo culture)
        {
            throw new NotImplementedException();
        }
    }
```

为何添加上面转化，这个程序员和客户对于数组的开始是不同的。

![](http://image.acmx.xyz/90d81d04-5aeb-42a3-adc2-8bc3f0d458b1201722210250.jpg)


<!-- for(var i = 57;i>0;i--)
{
    document.getElementById('msg'+i).checked = true; 
} -->

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  