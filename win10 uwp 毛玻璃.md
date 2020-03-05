# win10 uwp 毛玻璃

毛玻璃在UWP很简单，不会和WPF那样伤性能。

本文告诉大家，如何在 UWP 使用 win2d 做毛玻璃。

<div id="toc"></div>

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

<!-- 标签：uwp，win2d -->

毛玻璃可以使用 win2D  方法，也可以使用 Compositor 。

使用 win2d 得到软件内控件毛玻璃，而使用 Compositor 可以获得窗口毛玻璃。

先来说下如何使用 Compositor 做窗口毛玻璃，感觉小伙伴感兴趣的是窗口毛玻璃。

## Compositor 创建毛玻璃

先写最简单的页面，只有一个 Grid， 给他名称 GlassHost，这个控件用于显示毛玻璃


```csharp
            <Grid x:Name="GlassHost"></Grid>

```

然后在构造函数使用InitializeFrostedGlass，这个函数用于在一个控件显示毛玻璃

```csharp
    
        public MainPage()
        {
            InitializeComponent();
            InitializeFrostedGlass(GlassHost);
        }

        private void InitializeFrostedGlass(UIElement glassHost)
        {
            Visual hostVisual = ElementCompositionPreview.GetElementVisual(glassHost);
            Compositor compositor = hostVisual.Compositor;
            var backdropBrush = compositor.CreateHostBackdropBrush();
            var glassVisual = compositor.CreateSpriteVisual();
            glassVisual.Brush = backdropBrush;
            ElementCompositionPreview.SetElementChildVisual(glassHost, glassVisual);
            var bindSizeAnimation = compositor.CreateExpressionAnimation("hostVisual.Size");
            bindSizeAnimation.SetReferenceParameter("hostVisual", hostVisual);
            glassVisual.StartAnimation("Size", bindSizeAnimation);
        }
```

这样就可以看到毛玻璃效果

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017531204634.jpg)

这个代码是从 http://www.jianshu.com/p/3b49fd3d7edb 复制的

大概解释一下， `compositor.CreateHostBackdropBrush()` 获得 创建之前绘制窗口后面视觉效果的区域，然后把他添加到Grid就可以了。

但是模糊的玻璃可以看不到里面控件，于是就把控件放在一个Grid 的最前，这样看起来背景就是毛玻璃


```csharp
      <Grid > 最外层的 Grid 不要设置 BackGround 
        <Grid x:Name="GlassHost"></Grid> 把他放在最前
        <ListView ItemsSource="{x:Bind AvaloniaCol}" IsItemClickEnabled="True" ItemClick="ListViewBase_OnItemClick" >
            <ListView.ItemTemplate>
                <DataTemplate>
                   <Grid Background="#FFFFFF" PointerPressed="UIElement_OnPointerPressed">
                        <TextBlock Text="{Binding}"></TextBlock>
                   </Grid>
                </DataTemplate>
            </ListView.ItemTemplate>
        </ListView>

        <Button Content="添加" Click="ButtonBase_OnClick"></Button> 可以看到按钮，是清晰的
    </Grid>
```

如何去掉标题栏，上面的博客也有说，于是我就不多说啦。

## win2D

下面介绍使用 win2d 做毛玻璃

使用 win2D 方法，需要使用 Nuget 安装，如果速度太慢，推荐使用博客园的镜像

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201753121840.jpg)

这个方法可以获得控件的毛玻璃，但是不可以获得窗口毛玻璃

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E6%25AF%259B%25E7%258E%25BB%25E7%2592%2583.gif)

接下来告诉大家如何做上图的效果。

但是可以看到，上面的图做了其他的，如拖动时显示后面的图片。为了显示最短的代码，让大家知道毛玻璃是如何做的，下面先来做效果。

第一步，获得显示的图片

参见：[win10 uwp 截图 获取屏幕显示界面保存图片](http://lindexi.oschina.io/lindexi//post/win10-uwp-%E6%88%AA%E5%9B%BE-%E8%8E%B7%E5%8F%96%E5%B1%8F%E5%B9%95%E6%98%BE%E7%A4%BA%E7%95%8C%E9%9D%A2%E4%BF%9D%E5%AD%98%E5%9B%BE%E7%89%87/)

于是在界面显示一个图片，界面的左边就是图片，右边就是毛玻璃。之所以需要获得图片的截图是因为毛玻璃需要输入源，于是界面代码如下


```csharp
         <Grid.ColumnDefinitions>
            <ColumnDefinition Width="*"/>
            <ColumnDefinition Width="*"/>
        </Grid.ColumnDefinitions>
        <Grid Margin="10 10 10 10">
             必须把图片的路径修改为自己工程的路径，需要在工程存在图片
            <Image x:Name="Image" Source="Assets/2017年5月31日 210702.jpg" Stretch="UniformToFill" />
        </Grid>
        <Grid Grid.Column="1" Margin="10 10 10 10">
            <xaml:CanvasControl x:Name="Canvas" CreateResources="Canvas_CreateResources" Draw="Canvas_Draw" />
        </Grid>
```
毛玻璃效果写在 CanvasControl ，
需要对显示截图，把图片做效果。然后把得到的效果显示

但是在什么时候截图？也就是什么时候才是截图最好的时候？

我认为可以在 CreateResources 事件进行截图，请看代码


```csharp
         void Canvas_CreateResources(CanvasControl sender, CanvasCreateResourcesEventArgs args)
        {
            args.TrackAsyncAction(CreateResourcesAsync(sender).AsAsyncAction());
        }

        async Task CreateResourcesAsync(CanvasControl sender)
        {
            // give it a little bit delay to ensure the image is load, ideally you want to Image.ImageOpened event instead
            await Task.Delay(200);  这是等待图片加载，因他发生在控件初始之后，而图片加载发生在图片控件初始的时候，但是图片加载需要时间，所以这里等待一下。我觉得这是比较差的方法

            using (var stream = new InMemoryRandomAccessStream())
            {
                // get the stream from the background image
                var target = new RenderTargetBitmap(); 这就是截图
                await target.RenderAsync(Image);

                var pixelBuffer = await target.GetPixelsAsync();
                var pixels = pixelBuffer.ToArray();

                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.BmpEncoderId, stream);
                encoder.SetPixelData(BitmapPixelFormat.Bgra8, BitmapAlphaMode.Straight, (uint) target.PixelWidth, (uint) target.PixelHeight, 96 如果 dpi 不是96 那么这里需要写实际的，为了简单，我就不写如何获得dpi, 96, pixels);

                await encoder.FlushAsync();
                stream.Seek(0);

                // load the stream into our bitmap
                _bitmap = await CanvasBitmap.LoadAsync(sender, stream);
            }
        }
```

第二步就是把图片进行效果，代码很少


```csharp
         void Canvas_Draw(CanvasControl sender, CanvasDrawEventArgs args)
        {
            using (var session = args.DrawingSession)
            {
                var blur = new GaussianBlurEffect
                {
                    BlurAmount = 50.0f, // increase this to make it more blurry or vise versa.
                    //Optimization = EffectOptimization.Balanced, // default value
                    //BorderMode = EffectBorderMode.Soft // default value
                    Source = _bitmap
                };

                session.DrawImage(blur, new Rect(0, 0, sender.ActualWidth, sender.ActualHeight),
                    new Rect(0, 0, _bitmap.SizeInPixels.Width, _bitmap.SizeInPixels.Height), 0.9f);
            }
        }
```
现在看起来就是

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017531212439.jpg)

如果需要修改模糊，请把 BlurAmount 修改为你想要的

上面的代码就是主要的，接下来就是做上图的效果

首先xaml代码：


```csharp
            <Grid x:Name="ImagePanel2" Width="356" Height="200" Margin="0,0,0,40" VerticalAlignment="Bottom">
            <Image x:Name="Image2" Source="Assets/2017年5月31日 210702.jpg" Stretch="UniformToFill" />
            <Grid x:Name="Overlay" ManipulationMode="TranslateX" ManipulationStarted="Overlay_ManipulationStarted" ManipulationDelta="Overlay_ManipulationDelta" ManipulationCompleted="Overlay_ManipulationCompleted" RenderTransformOrigin="0.5,0.5">
                <Grid.Clip>
                    <RectangleGeometry x:Name="Clip" Rect="0, 0, 356, 200" />
                </Grid.Clip>
                <Rectangle x:Name="WhiteMask" Fill="White" />
                <xaml:CanvasControl x:Name="Canvas" CreateResources="Canvas_CreateResources" Draw="Canvas_Draw" />
            </Grid>
        </Grid>
```

可以看到，这里引用 CanvasControl ，还有很多代码需要写在后面


```csharp
         void Canvas_CreateResources(CanvasControl sender, CanvasCreateResourcesEventArgs args)
        {
            args.TrackAsyncAction(CreateResourcesAsync(sender).AsAsyncAction());
        }

        async Task CreateResourcesAsync(CanvasControl sender)
        {
            // give it a little bit delay to ensure the image is load, ideally you want to Image.ImageOpened event instead
            await Task.Delay(200);

            using (var stream = new InMemoryRandomAccessStream())
            {
                // get the stream from the background image
                var target = new RenderTargetBitmap();
                await target.RenderAsync(this.Image2);

                var pixelBuffer = await target.GetPixelsAsync();
                var pixels = pixelBuffer.ToArray();

                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.BmpEncoderId, stream);
                encoder.SetPixelData(BitmapPixelFormat.Bgra8, BitmapAlphaMode.Straight, (uint) target.PixelWidth, (uint) target.PixelHeight, 96, 96, pixels);

                await encoder.FlushAsync();
                stream.Seek(0);

                // load the stream into our bitmap
                _bitmap = await CanvasBitmap.LoadAsync(sender, stream);
            }
        }

        void Canvas_Draw(CanvasControl sender, CanvasDrawEventArgs args)
        {
            using (var session = args.DrawingSession)
            {
                var blur = new GaussianBlurEffect
                {
                    BlurAmount = 50.0f, // increase this to make it more blurry or vise versa.
                    //Optimization = EffectOptimization.Balanced, // default value
                    //BorderMode = EffectBorderMode.Soft // default value
                    Source = _bitmap
                };

                session.DrawImage(blur, new Rect(0, 0, sender.ActualWidth, sender.ActualHeight),
                    new Rect(0, 0, _bitmap.SizeInPixels.Width, _bitmap.SizeInPixels.Height), 0.9f);
            }
        }

        void Overlay_ManipulationStarted(object sender, ManipulationStartedRoutedEventArgs e)
        {
            // 重新设置 _x
            _x = (float) this.ImagePanel2.ActualWidth;
        }

        void Overlay_ManipulationDelta(object sender, ManipulationDeltaRoutedEventArgs e)
        {
            //获得当前的x，用于下面计算
            _x += (float) e.Delta.Translation.X;

            //如果当前的x超过了，或者已经最小
            if (_x > ImagePanel2.ActualWidth || _x < 0)
                return;

            //我们剪辑覆盖，用于显示下面的图片
            Clip.Rect = new Rect(0, 0, _x, this.ImagePanel2.ActualHeight);
        }

        void Overlay_ManipulationCompleted(object sender, ManipulationCompletedRoutedEventArgs e)
        {
            // 重置剪辑显示完整的覆盖
            Clip.Rect = new Rect(0, 0, this.ImagePanel2.ActualWidth, this.ImagePanel2.ActualHeight);
        }
```

上面的代码就是获得图片，把图片使用 GaussianBlurEffect 得到毛玻璃

实际代码做的就是 如下面显示，做出毛玻璃效果，其他代码都是为了做刚才的图


```csharp
        void Canvas_Draw(CanvasControl sender, CanvasDrawEventArgs args)
        {
            using (var session = args.DrawingSession)
            {
                var blur = new GaussianBlurEffect
                {
                    BlurAmount = 50.0f, // increase this to make it more blurry or vise versa.
                    //Optimization = EffectOptimization.Balanced, // default value
                    //BorderMode = EffectBorderMode.Soft // default value
                    Source = _bitmap
                };

                session.DrawImage(blur, new Rect(0, 0, sender.ActualWidth, sender.ActualHeight),
                    new Rect(0, 0, _bitmap.SizeInPixels.Width, _bitmap.SizeInPixels.Height), 0.9f);
            }
        }
```
关于拖动使用裁剪显示后面的图，我就不多说了，实际代码看起来很多，但是不是很难，我就不说拉。


请看下面的效果，这就是不停修改 BlurAmount 得到。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F%25E7%258E%25BB%25E7%2592%25832017%25E5%25B9%25B46%25E6%259C%25881%25E6%2597%25A5%2520205721.gif)

代码很简单，所以我就不说。

## 最简单方法

当然，还有最简单的代码，只需要一句话，请看文档 
[Acrylic material](https://docs.microsoft.com/en-us/windows/uwp/style/acrylic)
因为不知道微软是否还更改，所以我就不写了。

为了说明代码的简单，我需要给个例子，上面那么长的代码，现在只需要一行

```csharp
<Grid Background="{ThemeResource SystemControlAcrylicElementBrush}">
```

关于FDS请看[win10 uwp Fluent Design System 实践](https://lindexi.oschina.io/lindexi/post/win10-uwp-Fluent-Design-System-%E5%AE%9E%E8%B7%B5.html )

参见：https://stackoverflow.com/questions/31987817/how-to-make-frosted-glass-effect-in-windows-10-universal-app

http://microsoft.github.io/Win2D/html/N_Microsoft_Graphics_Canvas_Effects.htm

[（UWP）应用窗口实现毛玻璃效果 - 简书](http://www.jianshu.com/p/3b49fd3d7edb)



<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。