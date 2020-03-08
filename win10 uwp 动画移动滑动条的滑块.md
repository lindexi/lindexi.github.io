# win10 uwp 动画移动滑动条的滑块

堆栈网小伙伴问如何点击滑动条的时候，可以通过动画将滑块从原来的坐标移动到用户点击的坐标，同时用户拖动的时候不做动画

在后台代码添加两个事件，一个是按下，一个抬起，通过按下和抬起判断坐标可以知道用户是点击还是拖动。然后用上一个值和当前的值做动画就可以。

<!--more-->
<!-- CreateTime:2019/3/27 10:51:32 -->

<!-- csdn -->

先创建一个项目，添加简单的界面

```csharp
        <Slider x:Name="Slider" />
```

在写的时候发现有三个坑

1. 路由事件的 PointerPressed 会在 Slider 吃了，需要在后台代码添加事件
1. 在 PointerPressed 方法调用之前已经设置了 Slider 的值
1. 动画修改了依赖属性需要修改 EnableDependentAnimation 属性

因为使用的代码很多，我将代码放在了 github 可以下载

## 拿到事件

在 MainPage 的构造函数添加下面代码，在后台写代码可以在控件吃了路由事件还可以拿到事件

```csharp
        public MainPage()
        {
            InitializeComponent();
            Slider.AddHandler(PointerPressedEvent, new PointerEventHandler(Slider_OnPointerPressed), true);
            Slider.AddHandler(PointerReleasedEvent, new PointerEventHandler(Slider_OnPointerReleased), true);
        }
```

注意在后台代码的方法最后一个参数设置为 true 就表示控件吃了路由事件，也会调用方法

## 判断是否点击

如果用户是点击那么才使用动画，在 UWP 没有 PointerClick 事件所以需要自己写

```csharp
        private void Slider_OnPointerPressed(object sender, PointerRoutedEventArgs e)
        {
            var slider = (Slider) sender;

            ClickPoint = e.GetCurrentPoint(slider).Position;
        }

        private Point ClickPoint { set; get; }

        private void Slider_OnPointerReleased(object sender, PointerRoutedEventArgs e)
        {
            var slider = (Slider) sender;

            var point = e.GetCurrentPoint(slider).Position;

            var x = point.X - ClickPoint.X;
            var y = point.Y - ClickPoint.Y;
            var length = x * x + y * y;
            if (length < 10)
            {
                // 开始动画
            }
        }
```

在 `Slider_OnPointerPressed` 拿到用户点击坐标，然后在 `Slider_OnPointerReleased` 判断两个点就可以知道用户是不是拖动

在 UWP 的 Windows.Foundation.Point 没有默认的两个点相减拿到向量的方法，所以我就自己写了一个

## 记录之前的值

在 `Slider_OnPointerPressed` 这些方法拿到的 Slider 的值已经更新了，因为事件是先在 Slider 然后是在 MainPage 里面的方法，在 Slider 里面修改了值，所以需要添加依赖属性用来记录之前的值

```csharp
        public static readonly DependencyProperty ValueProperty = DependencyProperty.Register(
            "Value", typeof(double), typeof(MainPage), new PropertyMetadata(default(double), (s
                , e) =>
            {
                ((MainPage) s)._lastValue = (double) e.OldValue;
            }));

        public double Value
        {
            get => (double) GetValue(ValueProperty);
            set => SetValue(ValueProperty, value);
        }

        private double _lastValue;
```

## 动画

现在知道了用户是不是点击，可以开始做动画

在后台写代码比较不推荐，所以下面我就会在后台写动画。 这个逻辑好像说反了

做动画需要三步

1. 定义 Storyboard 和 Animation 类
1. 通过附加属性绑定 Animation 和元素
1. 播放动画

于是开始第一步

```csharp
            var storyboard = new Storyboard();

            var animation = new DoubleAnimation
            {
                From = _lastValue,
                To = Value,
                Duration = TimeSpan.FromSeconds(2),
                EasingFunction = new CubicEase(),
                EnableDependentAnimation = true
            };
```

注意，需要添加 `EnableDependentAnimation = true` 动画才能播放，这个和 WPF 不相同。为什么需要这个属性，在[DoubleAnimation.EnableDependentAnimation](https://docs.microsoft.com/en-us/uwp/api/windows.ui.xaml.media.animation.doubleanimation.enabledependentanimation )文档里面说到，如果动画修改的是依赖属性，动画需要不断在主线程修改，会降低性能，所以需要用户设置这个属性

第二步开始绑定动画

```csharp
            Storyboard.SetTarget(animation, Slider);
            Storyboard.SetTargetProperty(animation, "Value");
```

第三步开始播放动画

```csharp
            storyboard.BeginTime = TimeSpan.Zero;
            storyboard.Children.Add(animation);

            storyboard.Begin();
```

尝试运行代码，点击一下就可以看到动画

所有代码在[github](https://github.com/lindexi/lindexi_gd/tree/2e89ef71bba336d06ec238e9c0f4c8d893c83bdf/LajeweekallqeFiwigewee)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
