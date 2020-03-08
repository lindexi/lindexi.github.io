# Xamarin Forms 进度条控件

本文翻译：http://xamlnative.com/2016/04/14/xamarin-forms-a-simple-circular-progress-control/ 里面都是胡说的，如果看不懂可以联系邮箱

源代码：https://github.com/billreiss/xamlnative/tree/master/XamarinForms/CircularProgress

最近作者需要做一个简单的圆形的等待控件在一个Xamarin Forms应用，效果可以看

![这里写图片描述](http://img.blog.csdn.net/20160428145540311)
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

看起来很容易做，不知道怎么微软就没有弄个这么好看，微软没有，我们来直接做，看起来这个很简单

原来的进度条是一个线，没有UWP那个ring，我要做一个，可以使用本地控制、自定义渲染器渲染、使用组件里面弄很多我之前做的、到Nuget找，这些都觉得不是我要的。

看到他们没有，我就很高兴，我可以做一个很厉害的，自然这里我是原文的那个，写了Xaml的大神

我首先拿出一个本子，我应该弄矢量图形，在Xamarin原生还没有，我会为每个平台定制渲染，所以他不支持我不能使用，我想到使用图片，矢量图片，既然想要图片我如何让很多图片看起来是一个

![这里写图片描述](http://img.blog.csdn.net/20160428150315349)

我想到简单使用两图，实际对称两图是表示4图，不停覆盖的两个图片表示进度，两个图片颜色不同

![这里写图片描述](http://img.blog.csdn.net/20160428150542049)

![这里写图片描述](http://img.blog.csdn.net/20160428150553877)

图片可以在：https://github.com/billreiss/xamlnative/tree/master/XamarinForms/CircularProgress/CircularProgress/CircularProgress.Droid/Resources/drawable

两个保存格式Png图片，一个图表示0-50%，我们叫第一图“completed”，第二“pending”，颜色深的是第一，进度我们需要一个completed，两个pending，我们先放completed，然后在它上面放pending，在pending对面放pending，第一个图在代码叫“progress1”，第二“background1”，第二个覆盖第一个，第三个pending旋转180，总的一个蓝色圆，这是0%

![这里写图片描述](http://img.blog.csdn.net/20160428151156551)

25%：我们旋转pending第二个，可以让看到下面的图，这个我们覆盖原来的pending因为颜色一样，所以我们就可以看到25%

![这里写图片描述](http://img.blog.csdn.net/20160428151500833)

50%：我们需要改变，两个completed，一个pending，pending覆盖completed，但是只是覆盖一个，他们的层次：

 - completed 
 - pending 
 - completed

可以让pending覆盖右边的completed，超过50%让pending右旋

如果觉得上面说的还是不知道，可以看代码

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Xamarin.Forms;
 
namespace CircularProgress
{
    public class CircularProgressControl : Grid
    {
        View progress1;
        View progress2;
        View background1;
        View background2;
        public CircularProgressControl()
        {
            progress1 = CreateImage("progress_done");
            background1 = CreateImage("progress_pending");
            background2 = CreateImage("progress_pending");
            progress2 = CreateImage("progress_done");
            HandleProgressChanged(1, 0);
        }
 
        private View CreateImage(string v1)
        {
            var img = new Image();
            img.Source = ImageSource.FromFile(v1 + ".png");
            this.Children.Add(img);
            return img;
        }
 
        public static BindableProperty ProgressProperty =
    BindableProperty.Create("Progress", typeof(double), typeof(CircularProgressControl), 0d, propertyChanged: ProgressChanged);
 
        private static void ProgressChanged(BindableObject bindable, object oldValue, object newValue)
        {
            var c = bindable as CircularProgressControl;
            c.HandleProgressChanged(Clamp((double)oldValue, 0, 1), Clamp((double)newValue, 0, 1));
        }
 
        static double Clamp(double value, double min, double max)
        {
            if (value <= max && value >= min) return value;
            else if (value > max) return max;
            else return min;
        }
 
        private void HandleProgressChanged(double oldValue, double p)
        {
            if (p < .5)
            {
                if (oldValue >= .5)
                {
                    // this code is CPU intensive so only do it if we go from >=50% to <50%
                    background1.IsVisible = true;
                    progress2.IsVisible = false;
                    background2.Rotation = 180;
                    progress1.Rotation = 0;
                }
                double rotation = 360 * p;
                background1.Rotation = rotation;
            }
            else
            {
                if (oldValue < .5)
                {
                    // this code is CPU intensive so only do it if we go from <50% to >=50%
                    background1.IsVisible = false;
                    progress2.IsVisible = true;
                    progress1.Rotation = 180;
                }
                double rotation = 360 * p;
                background2.Rotation = rotation;
            }
        }
 
        public double Progress
        {
            get { return (double)this.GetValue(ProgressProperty); }
            set { SetValue(ProgressProperty, value); }
        }
    }
}
```

我们需要把图片放在不同平台的文件夹，ios放在Resources文件夹，Android放在 AndroidResource

我们把控件放MainPage.xaml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://xamarin.com/schemas/2014/forms"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             x:Class="CircularProgress.MainPage"
             xmlns:local="clr-namespace:CircularProgress" BackgroundColor="White">
  <Grid>
    <local:CircularProgressControl x:Name="progressControl" Progress="0" HorizontalOptions="Center" VerticalOptions="Center" WidthRequest="60" HeightRequest="60"/>
  </Grid>
</ContentPage>
```

我们让time进度加0.1每0.02s

```csharp
namespace CircularProgress
{
    public partial class MainPage : ContentPage
    {
        public MainPage()
        {
            InitializeComponent();
            Xamarin.Forms.Device.StartTimer(TimeSpan.FromSeconds(.02), OnTimer);
        }
 
        private bool OnTimer()
        {
            var progress = (progressControl.Progress + .1) ;
            if (progress > 1) progress = 0;
            progressControl.Progress = progress;
            return true;
        }
    }
}
```

不使用自定义渲染，可以在各个平台没有使用厉害的技术覆盖两个图做出从0-100%，可以使用不同角度表示0.001

本文：http://blog.csdn.net/lindexi_gd





