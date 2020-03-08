# win10 UWP  蜘蛛网效果

我看见了知乎首页登录背景和[普通的地球人](http://www.cnblogs.com/tsliwei/p/6282183.html)写的博客，发现了个好看的效果。

![](https://ooo.0o0.ooo/2017/01/29/588da9b7530da.gif)

那么我来告诉大家如何做这个效果。

第一步是在 Canvas 画点，第二步是让点移动，第三步是画线

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->


<div id="toc"></div>

## 在 Canvas 画一个点

我们画点可以使用 Ellipse 我们给他宽和高，Fill，就可以画出来。需要加在 Canvas ，可以使用`canvas.Children.Add(ellipse)`

Canvas 一开始的大小是 0 ，需要一开始给他一个足够大的值

``` xml

<Canvas x:Name="P" Width="1000" Height="1000"/>

```

于是给他一个比较大的值，超过外面的Grid就不显示。

添加 一个 Ellipse 就会显示，可以没有指定在哪显示，也就是显示的 X 和 Y。

## 指定添加到 Canvas 的 Element 的位置

我们可以通过几个方法改变控件的位置，在我之前写的拖动控件博客有说到。

现在使用 Canvas，可以使用 Canvas 有的一个方法。

假如有一个 X 和 Y 要设置在控件，那么可以使用

``` csharp

                Canvas.SetLeft(control, X);
                Canvas.SetTop(control, Y);

```

注意，Canvas 是类。

这个方法可以移动控件。

我就是用他移动点。

## 随机移动点

我首先写一个类，Staf。包含显示的 Point 和他的 X，Y，两个方向移动速度。还有移动多久，超过了可以移动的时间，就随机给新移动速度。

``` csharp

    public class Staf
    {
        public UIElement Point { set; get; }

        public double X { set; get; }

        public double Y { set; get; }

        public double Vx { set; get; }

        public double Vy { set; get; }

        public void RandomStaf(Random ran)
        {
            var staf = this;
            _ran = ran;
            staf.Vx = (double)ran.Next(-1000, 1000) / 1000;
            staf.Vy = (double)ran.Next(-1000, 1000) / 1000;
            staf.Time = ran.Next(100);
        }
        private Random _ran;
        public int Time
        {
            set
            {
                _time = value;
                if (value == 0)
                {
                    RandomStaf(_ran);
                }
            }
            get
            {
                return _time;
            }
        }

        private int _time;
    }

```

## 画线

使用两重 foreach ，得到两个点之间距离，如果距离小于我给的一个值，那么就是可以连线

那么距离长的就把连线的宽度变短。

这个做法很简单，可以使用 StrokeThickness 设置线宽度。

```
line.StrokeThickness=最大宽度 * （最大距离-距离）/最大距离
```

线需要多少个点可以确定？这个我就不说啦，确定了两个点是可以连线，于是使用就可以设置线的点。需要知道，点的X和Y是左上角，需要加上画的图形的值才是连在点，不然看起来不是连在点。



## 自动移动

可以使用 DispatcherTimer ，过 0.1 s就移动点和画线。

```csharp

        public MainPage()
        {
            this.InitializeComponent();
            _time = new DispatcherTimer();
            _time.Interval = TimeSpan.FromTicks(500);
            _time.Tick += Time_Tick;            
            _time.Start();
        }

private DispatcherTimer _time
```
Time_Tick就写移动点和线的代码

## 全部代码

```chsarp

<Page
    x:Class="Bsgame.MainPage"
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:local="using:Bsgame"
    xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
    xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
    mc:Ignorable="d">
    
    <Grid Background="{ThemeResource ApplicationPageBackgroundThemeBrush}">
        <Canvas x:Name="P" Width="1000" Height="1000">

        </Canvas>
        <Canvas x:Name="Pw" Width="1000" Height="1000"></Canvas>
    </Grid>
</Page>


using System;
using System.Collections.Generic;
using System.IO;
using System.Linq;
using System.Runtime.InteropServices.WindowsRuntime;
using Windows.Foundation;
using Windows.Foundation.Collections;
using Windows.UI;
using Windows.UI.Xaml;
using Windows.UI.Xaml.Controls;
using Windows.UI.Xaml.Controls.Primitives;
using Windows.UI.Xaml.Data;
using Windows.UI.Xaml.Input;
using Windows.UI.Xaml.Media;
using Windows.UI.Xaml.Media.Animation;
using Windows.UI.Xaml.Navigation;
using Windows.UI.Xaml.Shapes;

namespace Bsgame
{
    /// <summary>
    /// 可用于自身或导航至 Frame 内部的空白页。
    /// </summary>
    public sealed partial class MainPage : Page
    {
        public MainPage()
        {
            this.InitializeComponent();
            _time = new DispatcherTimer();
            _time.Interval = TimeSpan.FromTicks(500);
            _time.Tick += Time_Tick;
            RandomStaf();
            _time.Start();
            _width = Window.Current.Bounds.Width;
            _height = Window.Current.Bounds.Height;//lindexi
        }

        private void RandomStaf()
        {
            const int count = 20;

            for (int i = 0; i < count; i++)
            {
                Staf staf = new Staf();
                staf.X = ran.Next((int)_width);
                staf.Y = ran.Next((int)_height);
                staf.Point = new Ellipse()
                {
                    Height = 10,
                    Width = 10,
                    Fill = new SolidColorBrush(Colors.Gray),
                };
                staf.RandomStaf(ran);
                // ＣＳＤＮ
                _staf.Add(staf);
            }

            foreach (var temp in _staf)
            {
                P.Children.Add(temp.Point);
                //ｌｉｎｄｅｘｉ
            }
        }

        private List<Staf> _staf = new List<Staf>();

        private double _width;
        private double _height;

        private void Time_Tick(object sender, object e)
        {
            foreach (var temp in _staf)
            {
                if (temp.X > _width || temp.Y > _height
                    || temp.X < 0 || temp.Y < 0)
                {
                    temp.X = ran.Next((int)_width);
                    temp.Y = ran.Next((int)_height);
                }//lindexi.oschina.io

                temp.X -= temp.Vx;
                temp.Y -= temp.Vy;

                Canvas.SetLeft(temp.Point, temp.X);
                Canvas.SetTop(temp.Point, temp.Y);

                temp.Time--;
            }
            const double distan = 200;
            Pw.Children.Clear();
            Line line = new Line();
            foreach (var temp in _staf)
            {
                foreach (var p in _staf)
                {
                    line.X1 = temp.X + 5;
                    line.Y1 = temp.Y + 5;
                    line.X2 = p.X + 5;
                    line.Y2 = p.Y + 5;
                    double sqrt = Math.Sqrt(Math.Pow((line.X1 - line.X2), 2) +
                      Math.Pow((line.Y1 - line.Y2), 2));
                    if (sqrt < distan)
                    {
                        line.Stroke = new SolidColorBrush(Colors.Gray);
                        line.StrokeThickness = 5* (distan- sqrt) /distan;
                        Pw.Children.Add(line);
                        line = new Line();
                    }
                }
            }
        }

        private Random ran = new Random();

        private DispatcherTimer _time;
    }

    public class Staf
    {
        public UIElement Point { set; get; }

        public double X { set; get; }

        public double Y { set; get; }

        public double Vx { set; get; }

        public double Vy { set; get; }

        public void RandomStaf(Random ran)
        {
            var staf = this;
            _ran = ran;
            staf.Vx = (double)ran.Next(-1000, 1000) / 1000;
            staf.Vy = (double)ran.Next(-1000, 1000) / 1000;
            staf.Time = ran.Next(100);
        }
        private Random _ran;
        public int Time
        {
            set
            {
                _time = value;
                if (value == 0)
                {
                    RandomStaf(_ran);
                }
            }
            get
            {
                return _time;
            }
        }

        private int _time;
    }
}

```

可以看到性能很差，于是把连线去掉，显示点不显示连接

```csharp
        private void RandomStaf(object sender, object e)
        {
            Storyboard board = new Storyboard();
            board.Duration = new Duration(TimeSpan.FromSeconds(1));
            board.Completed += RandomStaf;
            DoubleAnimationUsingKeyFrames animation;
            foreach (var temp in _staf)
            {
                double f = temp.X;

                temp.X += temp.Vx * 10;
                if (temp.X > _width - 100)
                {
                    temp.X = _width - 100;
                }
                else if (temp.X < 0)
                {
                    temp.X = 0;
                }



                animation = EllPoile(f, temp.X);
                Storyboard.SetTarget(animation, temp.Point);
                Storyboard.SetTargetProperty(animation, "(Canvas.Left)");
                board.Children.Add(animation);

                f = temp.Y;
                temp.Y += temp.Vy * 10;

                if (temp.Y > _height - 100)
                {
                    temp.Y = _height - 100;
                }
                else if (temp.Y < 0)
                {
                    temp.Y = 0;
                }

                animation = EllPoile(f, temp.Y);
                Storyboard.SetTarget(animation, temp.Point);
                Storyboard.SetTargetProperty(animation, "(Canvas.Top)");

                if (temp.X >= _width - 100 || temp.Y >= _height - 100
                      || temp.X <= 0 || temp.Y <= 0)
                {
                    temp.X = ran.Next((int)_width);
                    temp.Y = ran.Next((int)_height);
                }
                board.Children.Add(animation);
                temp.Time -= 10;

                animation = EllPoile(10, 15);
                Storyboard.SetTarget(animation, temp.Point);
                Storyboard.SetTargetProperty(animation, "Height");
                board.Children.Add(animation);

                animation = EllPoile(10, 15);
                Storyboard.SetTarget(animation, temp.Point);
                Storyboard.SetTargetProperty(animation, "Width");
                board.Children.Add(animation);

                animation = new DoubleAnimationUsingKeyFrames();
                EasingDoubleKeyFrame frame = new EasingDoubleKeyFrame();
                frame.KeyTime = KeyTime.FromTimeSpan(TimeSpan.FromSeconds(0));
                frame.Value = 0;
                animation.KeyFrames.Add(frame);

                frame = new EasingDoubleKeyFrame();
                frame.KeyTime = KeyTime.FromTimeSpan(TimeSpan.FromSeconds(0.5));
                frame.Value = 180;
                animation.KeyFrames.Add(frame);

                frame = new EasingDoubleKeyFrame();
                frame.KeyTime = KeyTime.FromTimeSpan(TimeSpan.FromSeconds(1));
                frame.Value = 0;
                animation.KeyFrames.Add(frame);
                Storyboard.SetTarget(animation, temp.Point.RenderTransform);
                Storyboard.SetTargetProperty(animation, "(CompositeTransform.Rotation)");
                board.Children.Add(animation);

            }
            board.Begin();

        }

```




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。