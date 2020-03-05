# win10 uwp 萤火虫效果

本文在[Nukepayload2](http://www.cnblogs.com/Nukepayload2/p/uwp_fireflyparticlesys_vbnet.html )指导下，使用他的思想用C#写出来。

本文告诉大家，如何使用 win2d 做出萤火虫效果。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:51 -->

<div id="toc"></div>
<!-- 标签：uwp,win2d -->

## 安装 win2d 

安装win2d的方法请使用 Nuget 下载的方法，参见：[win10 uwp win2d](http://lindexi.oschina.io/lindexi//post/win10-uwp-win2d/ )

![](http://image.acmx.xyz/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F20173262046.jpg)

下面先让大家看一下效果图再告诉大家如何做

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B49%25E6%259C%25881%25E6%2597%25A5%252016.gif)

## 创建界面

界面只需要很简单两句代码，第一句代码是命名引用，第二句代码就是添加 win2d 

```csharp
    xmlns:canvas="using:Microsoft.Graphics.Canvas.UI.Xaml"
        
        <canvas:CanvasAnimatedControl x:Name="canvas" ClearColor="Black" Update="Canvas_OnUpdate" Draw="Canvas_Draw"></canvas:CanvasAnimatedControl>

```

这里为何使用 `CanvasAnimatedControl` 而不是使用 `CanvasControl` ？因为需要进行更新，`CanvasAnimatedControl`提供了一些事件，这些事件可以用来做动画。

## 后台的方法

在`Canvas_OnUpdate`就写更新所有萤火虫的代码，在`Canvas_Draw`就写画出萤火虫的代码。

### 萤火虫

于是开始创建萤火虫的代码，在创建之前，需要一个随机的类，这个类用于控制萤火虫的呼吸和移动，都是随机的。

在指定的范围之内，随机取一个点，这个点作为目的的点。于是当前的值就开始移动向目的的点，移动的过程存在速度。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B49%25E6%259C%25881%25E6%2597%25A5%252016201710891854.jpg)

从这里可以知道，这个类需要下面这些属性

```csharp
        public double Value { get; set; }
        public double To { get; set; }
        public double Dalue { get; set; }

        public double Ma { get; set; }

        public double Mi { get; set; }

        /// <summary>
        /// 加速度
        /// </summary>
        public double Po { get; set; }
```

其中 `Dalue` 就是速度，从 Value 到 To 的速度，这个速度在随时可以被修改。

下面是这个类全部代码

```csharp
    class Ran
    {
        public Ran(double value, double ma, double mi)
        {
            Value = value;
            Ma = ma;
            Mi = mi;
            To = ran.NextDouble() * (Ma - Mi) + Mi;
        }

        public double Value { get; set; }
        public double To { get; set; }
        public double Dalue { get; set; }

        public double Ma { get; set; }

        public double Mi { get; set; }

        public bool EasingFunction { get; set; }

        /// <summary>
        /// 加速度
        /// </summary>
        public double Po { get; set; }

        public void Time(TimeSpan time)
        {

            if (Math.Abs(Dalue) < 0.000001)
            {
                if (Math.Abs(Po) < 0.0001)
                {
                    Dalue = Math.Abs(Value - To) / ran.Next(10, 300);
                }
                else
                {
                    Dalue = Po;
                }
            }
            //减数
            if (EasingFunction && Math.Abs(Value - To) < Dalue*10/*如果接近*/)
            {
                Dalue /= 2;
                if (Math.Abs(Dalue) < 1)
                {
                    Dalue = 1;
                }
            }
            int n = 1;
            if (Value > To)
            {
                n = n * -1;
            }
            Value += n * Dalue * time.TotalSeconds * 2;
            if (n > 0 && Value >= To)
            {
                Value = To;
                To = ran.NextDouble() * (Ma - Mi) + Mi;
                Dalue = 0;
            }
            if (n < 0 && Value <= To)
            {
                Value = To;
                To = ran.NextDouble() * (Ma - Mi) + Mi;
                Dalue = 0;
            }
        }


        private static Random ran = new Random();
    }

```

下面就是主要的类`FireflyParticle`包含了位置和颜色，不同透明度，当然不透明度可以做呼吸效果，于是这些值都需要做随机移动

```csharp
    class FireflyParticle
    {
        public FireflyParticle(Rect bound)
        {
            Point = new Point(ran.Next((int) bound.Width), ran.Next((int) bound.Height));
            _x = new Ran(Point.X, bound.Width, 0)
            {
                EasingFunction = true,
            };
            _y = new Ran(Point.Y, bound.Height, 0)
            {
                EasingFunction = true,
            };
            _radius = new Ran(ran.Next(2, 5), 5, 2)
            {
                Po = 0.71
            };
            Bound = bound;
        }

        public FireflyParticle()
        {
        }

        public void Time(TimeSpan time)
        {
            _radius.Time(time);
            _opColor.Time(time);
            _x.Time(time);
            _y.Time(time);

            Radius = _radius.Value;
            OpColor = _opColor.Value;
            Point = new Point(_x.Value, _y.Value);
        }

        public Point Point { get; set; }

        public Rect Bound
        {
            get { return _bound; }
            set
            {
                _bound = value;
                _x.Ma = value.Width;
                _y.Ma = value.Height;
            }
        }

        public double Radius { get; set; } = 10;
        public Color CenterColor { get; set; } = Color.FromArgb(255, 252, 203, 89);
        public double OpColor { set; get; } = 1;
        private static Random ran = new Random();

        private Ran _radius;
        private Ran _opColor = new Ran(1, 1, 0.001);

        private Ran _x;
        private Ran _y;
        private Rect _bound;
    }

```

看到这，是不是觉得参数存在 `time` 无法理解？这里的使用 time 是为了在性能比较差的电脑得到效果和性能比较好的一样，虽然中途有一些没有显示的，但是计算结果相同，不会出现性能差的电脑，动画速度和性能好的电脑不一样。

### 动画

下面就是更新所有的值，创建属性`FireflyParticle`用于放所有的类，因为很简单，我就不解释。

```csharp
        private List<FireflyParticle> FireflyParticle { set; get; } = new List<FireflyParticle>();

        private void BpyaxxjwkQwknemobzPage_Loaded(object sender, RoutedEventArgs e)
        {
            if (!FireflyParticle.Any())
            {
                Rect bound = new Rect(0, 0, canvas.ActualWidth, canvas.ActualHeight);
                for (int i = 0; i < 100; i++)
                {
                    FireflyParticle.Add(new FireflyParticle(bound));
                }
            }
        }

         private void Canvas_OnUpdate(ICanvasAnimatedControl sender, CanvasAnimatedUpdateEventArgs args)
        {
            foreach (var temp in FireflyParticle)
            {
                temp.Time(args.Timing.ElapsedTime);
            }
        }

```

把所有的值都进行变化，就是在做动画，但是移动距离不能太长，移动的算法在上面的随机类写的，算法很简单，也不是关键，于是在这里就不说了。

## 核心代码

这里的核心就是画出来，如何在 win2d 画出一个点，把这个点高斯模糊。不知道大家知道 PhotoShop ，这里用到了图层，需要自己心中知道是什么东西。现在的图片一般都是很多个图片合成，于是可以把一个点作为一个图层，到时候把这些点合并就是上面给大家看到的图。

如何在 win2d 使用图层，主要的类是`CanvasCommandList`用它就可以做出图层，最好使用`DrawImage`把他弄出来。

```csharp
      private void Canvas_Draw(ICanvasAnimatedControl sender, CanvasAnimatedDrawEventArgs args)
        {
            using (var session = args.DrawingSession)
            {
                 using (var cl = new CanvasCommandList(session))
                 using (var ds = cl.CreateDrawingSession())
                 {
                    //这里就是图层
                    // session.DrawImage(cl); 把 图层画出来

                 }
            }
        }
```

如何对图层做模糊？在win2d有很多效果，先尝试把点画出来，效果图：

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B49%25E6%259C%25881%25E6%2597%25A5%2520161.gif)

需要知道所有的效果都是可以直接画出来，我用的方法很简单，就直接写代码

```csharp
   class GlowEffectGraph : IDisposable
    {
        private MorphologyEffect morphology;
        public GlowEffectGraph()
        {
            Blur.BlurAmount = 10;
            Blur.BorderMode = EffectBorderMode.Soft;

            morphology = new MorphologyEffect()
            {
                Mode = MorphologyEffectMode.Dilate,
                Width = 10,
                Height = 10,
            };

            Blur.Source = morphology;

        }

        public GaussianBlurEffect Blur { get; set; } = new GaussianBlurEffect();

        public void Dispose()
        {
            Blur.Dispose();
            morphology.Dispose();
        }

        public void Setup(ICanvasImage canvas, double amount = 10)
        {
            morphology.Source = canvas;
            amount = Math.Min(amount / 2, 100);
            morphology.Width = (int) Math.Truncate(Math.Floor(amount));
            morphology.Height = (int) Math.Truncate(Math.Floor(amount));
            Blur.BlurAmount = (float) amount;
        }
    }
```

如何要把图层画出来，那么修改`Canvas_Draw`的代码

```csharp
     private void Canvas_Draw(ICanvasAnimatedControl sender, CanvasAnimatedDrawEventArgs args)
        {
            using (var session = args.DrawingSession)
            {
                foreach (var temp in FireflyParticle)
                {
                    using (var cl = new CanvasCommandList(session))
                    using (var ds = cl.CreateDrawingSession())
                    {
                        var c = temp.CenterColor;
                        c.A = (byte) (temp.OpColor * 255);
                        ds.FillCircle((float) temp.Point.X, (float) temp.Point.Y, (float) temp.Radius, c);
                        using (var glow = new GlowEffectGraph())
                        {
                            glow.Setup(cl, temp.Radius);
                            session.DrawImage(glow.Blur);
                        }
                    }
                }
            }
        }
```

<script src='https://gitee.com/lindexi/codes/m90sfzgdwqvk6j81au2y445/widget_preview?title=%E8%90%A4%E7%81%AB%E8%99%AB%E6%95%88%E6%9E%9C+Canvas_Draw+2'></script>

这个效果我放在 [商业游戏](ms-windows-store://pdp/?productid=9pb0286g2ldr) 可以玩一下，代码开源[https://github.com/lindexi/UWP/tree/master/uwp/src/VarietyHiggstGushed](https://github.com/lindexi/UWP/tree/master/uwp/src/VarietyHiggstGushed)

参见：[使用win2d实现萤火虫粒子效果 - Nukepayload2 - 博客园](http://www.cnblogs.com/Nukepayload2/p/uwp_fireflyparticlesys_vbnet.html )



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。