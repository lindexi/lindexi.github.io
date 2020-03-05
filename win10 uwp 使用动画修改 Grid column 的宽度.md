# win10 uwp 使用动画修改 Grid column 的宽度

今天 wurstmitbrot 问如何通过动画修改 Grid 的 column ，虽然 column 是一个依赖属性，可以绑定，但是做出动画还是比较难的。

本文告诉大家如何对 Grid 做动画。

<!--more-->
<!-- CreateTime:2018/8/10 19:17:19 -->


首先发出我做出的效果

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B48%25E6%259C%25883%25E6%2597%25A5%2520091908.gif)

实际上我动画做的是 double ，当然通过 double 进行绑定，可以看到，如果使用绑定需要进行转换，首先写一个转换的代码

```csharp
public class DoubletoGridConvert : IValueConverter
{
    public object Convert(object value, Type targetType, object parameter, string language)
    {
        var n = (double) value;
        return new GridLength(n);
    }

    public object ConvertBack(object value, Type targetType, object parameter, string language)
    {
        throw new NotImplementedException();
    }
}
```

需要两个依赖属性，可以绑定界面，动画。

```csharp
  public static readonly DependencyProperty RcProperty = DependencyProperty.Register(
        "Rc", typeof(double), typeof(MainPage), new PropertyMetadata(100d));

    public double Rc
    {
        get { return (double) GetValue(RcProperty); }
        set { SetValue(RcProperty, value); }
    }

    public static readonly DependencyProperty LcProperty = DependencyProperty.Register(
        "Lc", typeof(double), typeof(MainPage), new PropertyMetadata(500d));

    public double Lc
    {
        get { return (double) GetValue(LcProperty); }
        set { SetValue(LcProperty, value); }
    }
```

然后写一个简单界面，请看代码。

```csharp
      <Grid>
          <Grid.RowDefinitions>
              <RowDefinition Height="{x:Bind Rc,Mode=OneWay,Converter={StaticResource double}}"/>
              <RowDefinition Height="{x:Bind Lc,Mode=OneWay,Converter={StaticResource double}}"/>
          </Grid.RowDefinitions>
          <Grid Background="#FF565656"></Grid>
          <Grid Grid.Row="1" Background="#FFa2a2a2"></Grid>
      </Grid>
      <Button Margin="47,662,0,10" Content="set" Click="Button_OnClick"></Button>
```


点击按钮就可以进行动画。

动画我写在后台，于是会遇到几个问题，如果对于布局的，需要设置`EnableDependentAnimation `如果没有设置，那么动画将不会做什么，这是需要知道的。最近看了 h 神的博客我才知道这个。然后需要知道，一个Storyboard只能设置一个SetTarget到一个对象，所以需要分为多个 Storyboard ，我现在还不知道方法，可以绑定多个。

看起来的按钮点击需要下面的代码。

```csharp
       var storyboard = new Storyboard();
        var animation = new DoubleAnimation();
        Storyboard.SetTargetName(animation, nameof(MainPage));
        Storyboard.SetTarget(animation, this);
        Storyboard.SetTargetProperty(animation,"Rc");
        animation.EnableDependentAnimation = true;
        animation.From = 100;
        animation.To = 200;
        animation.Duration = new Duration(TimeSpan.FromMilliseconds(500));
        storyboard.Children.Add(animation);
        storyboard.Begin();

        storyboard = new Storyboard();
        animation = new DoubleAnimation();
        Storyboard.SetTarget(animation, this);
        Storyboard.SetTargetName(animation,nameof(MainPage));
        Storyboard.SetTargetProperty(animation, nameof(Lc));
        animation.From = 500;
        animation.To = 150;
        animation.Duration = new Duration(TimeSpan.FromMilliseconds(500));
        animation.EnableDependentAnimation = true;
        storyboard.Children.Add(animation);

        storyboard.Begin();
```

上面的代码还需要在动画完成进行设置，因为在配置比较低的机器，可能直接就没动画，所以在这里需要设置。

如果在开发遇到动画的问题，欢迎来问我。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。