我朋友在做一个控件，是显示异常，那么异常多就变为颜色，大概就是下面的图，很简单


![这里写图片描述](http://img.blog.csdn.net/20160804170030148)

![这里写图片描述](http://img.blog.csdn.net/20160804170037101)

![这里写图片描述](http://img.blog.csdn.net/20160804170050257)

![这里写图片描述](http://img.blog.csdn.net/20160804170100710)

![这里写图片描述](http://img.blog.csdn.net/20160804170108085)

![这里写图片描述](http://img.blog.csdn.net/20160804170119132)

![这里写图片描述](http://img.blog.csdn.net/20160804170125736)

![这里写图片描述](http://img.blog.csdn.net/20160804170133148)



首先是一个Ellipse，然后把他的颜色绑定到Int，需要一个转换，UWP的转换和WPF差不多，因为我现在还不会转换，就不多说。

转换很简单，不过我先说如何使用，控件放在[https://github.com/lindexi/UWP/tree/master/uwp/control/RountGradualFigure](https://github.com/lindexi/UWP/tree/master/uwp/control/RountGradualFigure)

首先把控件放在xaml，在后台放个int，然后绑定，接着修改这个int就可以看到颜色从绿到红，使用简单。

```
        <local:RoundFigureGradual N="{x:Bind N,Mode=OneWay}"></local:RoundFigureGradual>
```

转换的代码


```
    public class IntBrushConverter : IValueConverter
    {
        public object Convert(object value, Type targetType, object parameter, string culture)
        {
            byte r = 0, g = 0xff, b = 0;
            int n = (int)value;
            if (n > 0xff)
                return new SolidColorBrush(Colors.Red);
            g -= (byte)n;
            r += (byte)n;
            return new SolidColorBrush(Color.FromArgb(255, r, g, b));
        }

        public object ConvertBack(object value, Type targetType, object parameter, string culture)
        {
            throw new NotImplementedException();
        }
    }
```



## xaml定义常量

我们如何在我们界面定义一个常量，我有很多地方需要用到一个常量，那么我如何定义一个，让修改只有一个，不需要整个界面都在修改。

在WPF我们使用常量可以使用

```
<Page
 xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
 xmlns:sys="clr-namespace:System;assembly=mscorlib"  
 xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml">
   <Page.Resources>
     <sys:Double x:Key="Height">200</sys:Double> 
     <sys:Double x:Key="Width">200</sys:Double>
   </Page.Resources> 
 <Grid> 
  <Rectangle Height="{StaticResource Height}" Width="{StaticResource Width}" Fill="Blue"/> 
 </Grid>
</Page>

```

在UWP那简单，我们在Resource

```
 <x:Double x:Key="Height"> 200 </x:Double>

```

当然需要一个Key，然后一个值，我们可以有

- Boolean

- Int32

- String


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。


