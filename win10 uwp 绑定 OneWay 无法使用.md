# win10 uwp 绑定 OneWay 无法使用

有时候使用绑定的 OneWay 方法无法使用，而使用 TwoWay 的方法就可以使用，但是在调试把 OneWay 做了修改又可以使用，那么请看本文。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->


<!-- csdn -->

这里存在的问题就是，在绑定 OneWay 的时候，存在变量的值是一个绑定，不是具体的值，如果发现代码出现这样的错误。那么请你检查一下绑定的变量是否在其他地方有修改，如果有的话，请不要对他进行修改。

假如有一个简单的界面

```csharp
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition Height="60*"/>
            <RowDefinition Height="121*"/>
        </Grid.RowDefinitions>
      
        <Grid Grid.Row="0">
            <TextBlock x:Name="Barry" Margin="10,10,10,10" Text="0" VerticalAlignment="Center"></TextBlock>
            <Button Margin="10,10,10,10" Content="add" HorizontalAlignment="Right" Click="Button_OnClick"></Button>
        </Grid>

        <Grid Grid.Row="1">
            <TextBlock x:Name="Dagmar" Margin="10,10,10,10" Text="{Binding Text,ElementName=Barry,Mode=OneWay}" VerticalAlignment="Center"></TextBlock>
            <Button Margin="10,10,10,10" Content="Set" HorizontalAlignment="Right" Click="Dagmar_OnClick"></Button>
        </Grid>
    </Grid>

```
后台代码是

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var n = int.Parse(Barry.Text);
            n++;
            Barry.Text = n.ToString();
        }

        private void Dagmar_OnClick(object sender, RoutedEventArgs e)
        {
            Dagmar.Text = "-1";
        }
```

可以看到，在点击第一个按钮时，会增加两个文本，但是点击 Set 按钮时，就会把下面的文本设为-1，之后无论上面的第一个按钮怎么点击，下面的文本都不会跟着变化。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B47%25E6%259C%258824319.gif)

但是可以看到，设置 Mode 是twoway就可以继续绑定，因为在 WPF 有下面的代码，大概就是判断当前是否可以复制，对源进行赋值，如果不可以对源赋值，就清除绑定，给属性赋值。如果可以对源赋值，那么直接对源赋值。实际UWP也一样，但是我没有找到他的源码。

如果使用了属性，遇到oneway错误的问题，需要自己