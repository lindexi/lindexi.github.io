WPF checkbox文字下掉
![这里写图片描述](http://i13.tietuku.cn/5d20587e74850fc8.png)

可以使用

```
<Style TargetType="CheckBox">
            <Setter Property="Margin" Value="10,10,10,10"></Setter>
            <Setter Property="VerticalAlignment" Value="Center" />
            <Setter Property="FontSize" Value="10" />
        </Style>
        <Style TargetType="{x:Type TextBlock}">
            <Setter Property="Margin" Value="1,1,1,1" />
        </Style>
                <CheckBox Grid.Row="0" Grid.Column="1"  >
                    <TextBlock Text="内容" />
                </CheckBox>
```

https://dotblogs.com.tw/v6610688/2014/04/21/xaml_checkbox_content_alignment&prev=searc

博客blog.csdn.net/lindexi_gd
