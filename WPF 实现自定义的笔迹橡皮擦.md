# WPF 实现自定义的笔迹橡皮擦

本文来告诉大家使用比较底层的方法来实现 WPF 的笔迹橡皮擦

<!--more-->
<!-- 发布 -->

在 WPF 里面，对于笔迹来说，应该放在 Stroke 类里面，而不是作为点的集合存储。在 Stroke 类里面将作为管理笔迹的类提供笔迹的渲染和橡皮擦等功能。咱下面将从 Stroke 类开始，自己定义笔迹橡皮擦。阅读本文，你将了解如何自定义橡皮擦，如自定义橡皮擦的外观样式，了解如何不依赖 InkCanvas 来实现笔迹的擦除

原本我是想采用 [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html) 的方式来做笔迹的绘制部分的，但是考虑使用上面博客的方法将会让大家需要多了解很多触摸相关的知识，因此我就简单使用 InkCanvas 来做笔迹的绘制。以下只是将 InkCanvas 作为笔迹的绘制，而橡皮擦部分是咱定制的

在 XAML 中添加一个 InkCanvas 的代码很简单，请看代码

```xml
        <InkCanvas x:Name="InkCanvas"></InkCanvas>
```

咱可以从这个 InkCanvas 里面获取当前的笔迹，如下面代码

```csharp
StrokeCollection strokes = InkCanvas.Strokes;
```

这里拿到的 StrokeCollection 是一个集合，这个集合里面包含了多个 Stroke 类，在 WPF 中，一条笔迹就是一个 Stroke 对象。而多个 Stroke 就放在 StrokeCollection 类里面。可以认为是一个笔画就是一个 Stroke 而一个汉子包含了多个笔画，因此一个汉子的笔迹集合就使用 StrokeCollection 表示

通过上面代码就可以拿到 InkCanvas 里面的所有笔迹，接下来就是自定义橡皮擦部分的逻辑

这里的自定义橡皮擦的核心逻辑就是在 InkCanvas 上再放一个 Canvas 容器，在这个 Canvas 容器里面放自定义的橡皮擦的界面。因为这个 Canvas 容器在 InkCanvas 的上方，因此自定义的橡皮擦界面也将会在 InkCanvas 上

在界面里面放一个 Canvas 和一个用 Rectangle 表示的自定义外观的橡皮擦，大家可以使用自己喜欢的控件来代替 Rectangle 控件

```xml
        <InkCanvas x:Name="InkCanvas"></InkCanvas>
        <Canvas x:Name="EraserCanvas" Grid.Row="0" Background="Transparent" Visibility="Collapsed">
            <Rectangle x:Name="EraserShape" HorizontalAlignment="Left" 
                       Width="50" Height="100" Fill="Red" VerticalAlignment="Top">
                <Rectangle.RenderTransform>
                    <TranslateTransform x:Name="TranslateTransform"></TranslateTransform>
                </Rectangle.RenderTransform>
            </Rectangle>
        </Canvas>
```

可以看到在上面代码中，使用了 RenderTransform 来控制自定义的橡皮擦所在的坐标。上面代码有一个细节是需要设置这个自定义橡皮擦就在容器的左上角上，通过 HorizontalAlignment 和 VerticalAlignment 设置。当然了咱因为是放在 Canvas 容器里面，默认就是在左上角上，但是有个好习惯还是不错的。我就怕你抄代码的时候，用的容器和用的控件默认不是在左上角的

在上面代码中，咱默认的 EraserCanvas 是不可见的，而且背景色是透明的。这是为了默认可以在 InkCanvas 上写，而在点击按钮的时候，才设置 EraserCanvas 可见。在 EraserCanvas 设置背景色是透明的，是为了让 EraserCanvas 可以收到命中测试，也就是收到触摸或鼠标消息

在界面添加一个按钮，用于点击按钮的时候进入橡皮擦模式，如下面代码

```xml
        <StackPanel Grid.Row="1">
            <Button Content="进入橡皮擦" Margin="10,10,10,10" Click="Button_OnClick"></Button>
        </StackPanel>
```

现在的整个界面的代码大概如下

```xml
    <Grid>
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>
        <InkCanvas x:Name="InkCanvas"></InkCanvas>
        <Canvas x:Name="EraserCanvas" Grid.Row="0" Background="Transparent" Visibility="Collapsed">
            <Rectangle x:Name="EraserShape" HorizontalAlignment="Left" 
                       Width="50" Height="100" Fill="Red" VerticalAlignment="Top">
                <Rectangle.RenderTransform>
                    <TranslateTransform x:Name="TranslateTransform"></TranslateTransform>
                </Rectangle.RenderTransform>
            </Rectangle>
        </Canvas>
  
        <StackPanel Grid.Row="1">
            <Button Content="进入橡皮擦" Margin="10,10,10,10" Click="Button_OnClick"></Button>
        </StackPanel>
    </Grid>
```

进入到咱的后台代码逻辑，在点击按钮的时候，才是进入到核心的逻辑里面

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            EraserCanvas.Visibility = Visibility.Visible;
        }
```

其实就是让 EraserCanvas 可见，因为 EraserCanvas 放在 InkCanvas 上方，如果 EraserCanvas 可见，那么 EraserCanvas 将会吃掉在 InkCanvas 上的交互，如鼠标或触摸，都会命中到 EraserCanvas 上。因此 InkCanvas 就不能接收到消息，也就无法进入书写了

在 EraserCanvas 监听输入的事件，如下面代码监听了鼠标事件。那么即可在进入橡皮擦模式的时候，在 EraserCanvas 可以接收到输入消息触发代码

```csharp
            EraserCanvas.MouseDown += EraserCanvas_MouseDown;
            EraserCanvas.MouseMove += EraserCanvas_MouseMove;
            EraserCanvas.MouseUp += EraserCanvas_MouseUp;
```

在鼠标按下的时候咱开始进入核心的逻辑，请看代码

```csharp
        private IncrementalStrokeHitTester _incrementalStrokeHitTester;
        private bool _isDown;

        private void EraserCanvas_MouseDown(object sender, MouseButtonEventArgs e)
        {
            _isDown = true;

            IncrementalStrokeHitTester incrementalStrokeHitTester =
                InkCanvas.Strokes.GetIncrementalStrokeHitTester(new RectangleStylusShape(EraserShape.ActualWidth,
                    EraserShape.ActualHeight));

            _incrementalStrokeHitTester = incrementalStrokeHitTester;
            _incrementalStrokeHitTester.StrokeHit += IncrementalStrokeHitTester_StrokeHit;
        }
```

在 StrokeCollection 里面有一个方法是 GetIncrementalStrokeHitTester 方法，可以通过这个方法获取这段笔迹的命中测试工具。需要传入的是橡皮擦的形状和大小，可以支持的橡皮擦只有矩形和圆形两个。本文这里使用的是矩形的橡皮擦。如果你需要支持自定义形状的橡皮擦，如三角形等，就需要自己用更底层的方式去实现了，也不在本文范围之内

在获取到 IncrementalStrokeHitTester 工具之后，需要监听他的 StrokeHit 事件，这个事件将会在笔迹被擦到的时候触发，这个事件就是咱的核心逻辑了

在鼠标移动的时候，需要给 IncrementalStrokeHitTester 加上当前的触摸移动的点，请看代码

```csharp
        private void EraserCanvas_MouseMove(object sender, MouseEventArgs e)
        {
            if (_isDown)
            {
                var point = e.GetPosition(this);
                TranslateTransform.X = point.X - EraserShape.ActualWidth / 2;
                TranslateTransform.Y = point.Y - EraserShape.ActualHeight / 2;

                _incrementalStrokeHitTester.AddPoint(point);
            }
        }
```

上面代码有两个功能，一个是移动橡皮擦的外观，另一个是给命中测试工具加上当前的触摸点

在调用 IncrementalStrokeHitTester 的 AddPoint 方法的时候，如果刚好此时命中到了某个笔迹，那么将会触发 StrokeHit 事件

在 StrokeHit 事件里面包含了两个有用的参数，其中一个参数表示的是当前被命中的笔迹是哪个笔迹。另一个是在进行擦除之后新创建的笔迹。也就是说将原有的笔迹，一个笔迹擦为了多个笔迹，当然多个笔迹肯定也包含了零个笔迹

```csharp
        private void IncrementalStrokeHitTester_StrokeHit(object sender, StrokeHitEventArgs e)
        {
            InkCanvas.Strokes.Remove(e.HitStroke);
            InkCanvas.Strokes.Add(e.GetPointEraseResults());
        }
```

上面代码的逻辑就是将被擦到的笔迹删除掉，添加为擦出之后新建的多个笔迹。这样就能实现出笔迹被擦的效果。也就是说笔迹被插不是在原有的笔迹上删除某些点，而是将一条笔迹修改为多条的方式进行擦掉

这样的设计的好处在于撤销重做的功能很好做，因为原有的笔迹是不动的，是通过替换笔迹的形式，因此只需要保存笔迹的对象即可

在鼠标抬起的时候，可以清理一下橡皮擦的逻辑。当然在我的业务里面，抬起鼠标就是等于橡皮擦结束了

```csharp
        private void EraserCanvas_MouseUp(object sender, MouseButtonEventArgs e)
        {
            EraserCanvas.Visibility = Visibility.Collapsed;
            _isDown = false;

            _incrementalStrokeHitTester.EndHitTesting();
            _incrementalStrokeHitTester = null;
        }
```

上面代码核心是调用 EndHitTesting 清理一下资源，不调用也可以，不会存在内存泄露


全部的后台代码如下

```csharp
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();

            EraserCanvas.MouseDown += EraserCanvas_MouseDown;
            EraserCanvas.MouseMove += EraserCanvas_MouseMove;
            EraserCanvas.MouseUp += EraserCanvas_MouseUp;
        }

        private IncrementalStrokeHitTester _incrementalStrokeHitTester;
        private bool _isDown;

        private void EraserCanvas_MouseDown(object sender, MouseButtonEventArgs e)
        {
            _isDown = true;

            IncrementalStrokeHitTester incrementalStrokeHitTester =
                InkCanvas.Strokes.GetIncrementalStrokeHitTester(new RectangleStylusShape(EraserShape.ActualWidth,
                    EraserShape.ActualHeight));

            _incrementalStrokeHitTester = incrementalStrokeHitTester;
            _incrementalStrokeHitTester.StrokeHit += IncrementalStrokeHitTester_StrokeHit;
        }

        private void EraserCanvas_MouseUp(object sender, MouseButtonEventArgs e)
        {
            EraserCanvas.Visibility = Visibility.Collapsed;
            _isDown = false;

            _incrementalStrokeHitTester.EndHitTesting();
            _incrementalStrokeHitTester = null;
        }

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            EraserCanvas.Visibility = Visibility.Visible;

            TranslateTransform.X = -1000;
            TranslateTransform.Y = -1000;
        }

        private void IncrementalStrokeHitTester_StrokeHit(object sender, StrokeHitEventArgs e)
        {
            InkCanvas.Strokes.Remove(e.HitStroke);
            InkCanvas.Strokes.Add(e.GetPointEraseResults());
        }

        private void EraserCanvas_MouseMove(object sender, MouseEventArgs e)
        {
            if (_isDown)
            {
                var point = e.GetPosition(this);
                TranslateTransform.X = point.X - EraserShape.ActualWidth / 2;
                TranslateTransform.Y = point.Y - EraserShape.ActualHeight / 2;

                _incrementalStrokeHitTester.AddPoint(point);
            }
        }
    }
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0339f56a/BallbujawfemNolahelle ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0339f56a/BallbujawfemNolahelle ) 欢迎小伙伴访问

更多触摸请看 [WPF 触摸相关](https://blog.lindexi.com/post/WPF-%E8%A7%A6%E6%91%B8%E7%9B%B8%E5%85%B3.html ) 更多笔迹相关请看

- [WPF 渲染原理](https://lindexi.gitee.io/post/WPF-%E6%B8%B2%E6%9F%93%E5%8E%9F%E7%90%86.html )
- [高性能笔迹原理](https://blog.lindexi.com/post/%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94%E8%BF%B9%E5%8E%9F%E7%90%86.html)
- [WPF 高性能笔](https://blog.lindexi.com/post/WPF-%E9%AB%98%E6%80%A7%E8%83%BD%E7%AC%94.html ) 
- [WPF 高速书写 StylusPlugIn 原理](https://blog.lindexi.com/post/WPF-%E9%AB%98%E9%80%9F%E4%B9%A6%E5%86%99-StylusPlugIn-%E5%8E%9F%E7%90%86.html )
- [WPF 最小的代码使用 DynamicRenderer 书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E5%B0%8F%E7%9A%84%E4%BB%A3%E7%A0%81%E4%BD%BF%E7%94%A8-DynamicRenderer-%E4%B9%A6%E5%86%99.html )
- [WPF 使用 Composition API 做高性能渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Composition-API-%E5%81%9A%E9%AB%98%E6%80%A7%E8%83%BD%E6%B8%B2%E6%9F%93.html )
- [WPF 使用 Win2d 渲染](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Win2d-%E6%B8%B2%E6%9F%93.html )
- [win10 uwp win2d CanvasVirtualControl 与 CanvasAnimatedControl](https://blog.lindexi.com/post/win10-uwp-win2d-CanvasVirtualControl-%E4%B8%8E-CanvasAnimatedControl.html )
- [WPF 最简逻辑实现多指顺滑的笔迹书写](https://blog.lindexi.com/post/WPF-%E6%9C%80%E7%AE%80%E9%80%BB%E8%BE%91%E5%AE%9E%E7%8E%B0%E5%A4%9A%E6%8C%87%E9%A1%BA%E6%BB%91%E7%9A%84%E7%AC%94%E8%BF%B9%E4%B9%A6%E5%86%99.html)

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
