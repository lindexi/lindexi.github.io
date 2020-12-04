# WPF 自己封装 Skia 差量绘制控件

使用 Skia 能做到在多个不同的平台使用相同的一套 API 绘制出相同界面效果的图片，可以将图片绘制到应用程序的渲染显示里面。在 WPF 中最稳的方法就是通过 WriteableBitmap 作为承载绘制。本文告诉大家如何封装一个支持差量绘制的控件，默认的绘制方法都是每次都是不保存上次绘制的内容，而且清空画布，重新绘制。这样的绘制方法显然效率不够高

<!--more-->
<!-- CreateTime:2020/9/7 8:57:47 -->



在上一篇博客里面告诉大家如何在 WPF 中使用 Skia 绘制，请看 [WPF 使用 Skia 绘制 WriteableBitmap 图片](https://blog.lindexi.com/post/WPF-%E4%BD%BF%E7%94%A8-Skia-%E7%BB%98%E5%88%B6-WriteableBitmap-%E5%9B%BE%E7%89%87.html)

而这样的绘制方式意味着每次都需要重新绘制画布，而不能在原有上一次绘制的基础上绘制新的内容。其实在 Skia 的 SKSurface 是不需要每次绘制完成就释放，可以保存他的值

只是需要注意和 WriteableBitmap 图片一起使用时，需要在绘制之前调用 Lock 方法，在绘制完成之后调用 Unlock 方法

此时就可以实现在相同的 SKSurface 上重复上次绘制的内容。而如果能了解绘制的界面范围的话，可以使用 WriteableBitmap 的 AddDirtyRect 方法，通过这个方法可以让 WPF 层仅更新指定范围的内容

虽然 Skia 和 WPF 两个的绘制效率都很高，但是在 WriteableBitmap 里面一定存在内存和显存的拷贝，这部分虽然在 DirtyRect 很小的时候几乎不耗性能，但是如果是在 4k 下完全重新绘制，还是稍微有点伤的。只是稍微有点

在使用 WriteableBitmap 作为 Skia 的承载，就需要再来一步，让 WriteableBitmap 在界面绘制。在 WPF 中最简单的绘制 WriteableBitmap 的方法就是使用 Image 控件了

下面写一个继承 Image 控件的 SkiaCanvas 控件

这个控件十分简单，在 Loaded 事件里面将会创建 WriteableBitmap 和 SKSurface 两个字段，请看代码

```csharp
    public class SkiaCanvas : Image
    {
        public SkiaCanvas()
        {
            Loaded += SkiaCanvas_Loaded;
        }

        private void SkiaCanvas_Loaded(object sender, RoutedEventArgs e)
        {
            var writeableBitmap = new WriteableBitmap(PixelWidth, PixelHeight, 96, 96, PixelFormats.Bgra32,
                BitmapPalettes.Halftone256Transparent);

            _writeableBitmap = writeableBitmap;

            var skImageInfo = new SKImageInfo()
            {
                Width = PixelWidth,
                Height = PixelHeight,
                ColorType = SKColorType.Bgra8888,
                AlphaType = SKAlphaType.Premul,
                ColorSpace = SKColorSpace.CreateSrgb()
            };

            SKSurface surface = SKSurface.Create(skImageInfo, writeableBitmap.BackBuffer);
            _skSurface = surface;

            Source = writeableBitmap;
        }

        private WriteableBitmap _writeableBitmap = null!; // 这里的 null! 是 C# 的新语法，是给智能分析用的，表示这个字段在使用的时候不会为空
        private SKSurface _skSurface = null!; // 实际上 null! 的含义是我明确给他一个空值，也就是说如果是空也是预期的

        public int PixelWidth => (int) Width;
        public int PixelHeight => (int) Height;
    }
```

也就是说在使用 SkiaCanvas 控件的时候，需要先设置他的宽度和高度，也不支持后续更改哈

在创建完成了 SKSurface 字段，就可以通过调用他的绘制方法在 WriteableBitmap 上绘制内容。不过在绘制之前需要调用 Lock 等方法，在输入绘制命令完成之后需要调用更新的代码，这部分代码可以封装一个方法

```csharp
        public void Draw(Action<SKCanvas> action)
        {
            Draw(canvas =>
            {
                action(canvas);
                return null;
            });
        }

        public void Draw(Func<SKCanvas, Int32Rect?> draw)
        {
            var writeableBitmap = _writeableBitmap;
            writeableBitmap.Lock();

            var canvas = _skSurface.Canvas;
            var dirtyRect = draw(canvas);
            canvas.Flush();

            dirtyRect ??= new Int32Rect(0, 0, PixelWidth, PixelHeight);

            writeableBitmap.AddDirtyRect(dirtyRect.Value);
            writeableBitmap.Unlock();
        }
```

也就是调用 Draw 方法，传入具体的绘制逻辑就可以完成绘制了。这部分的绘制逻辑有一个优势在于不需要等待绘制时机，随时都可以进行绘制。而 WPF 将会在框架层的绘制命令收集时自动更新和收集。或者换句话说，这里的绘制逻辑有坑在于不能做到对准界面更新

上面这个方法是提供差量更新的，也就是每次绘制的内容都会在上一次画布的基础上继续绘制

下面写一点代码试试，在鼠标划过应用时，绘制出鼠标划过的点，将这些点连为线

如果没有差量更新，也就是需要咱自己去存放记录之前鼠标划过哪些点，在有差量更新的辅助就可以只记录上一次的一个点

在 XAML 代码添加如下代码

```xml
    <Grid MouseMove="UIElement_OnMouseMove">
        <Grid.RowDefinitions>
            <RowDefinition></RowDefinition>
            <RowDefinition Height="Auto"></RowDefinition>
        </Grid.RowDefinitions>
        <local:SkiaCanvas x:Name="Image" Width="1920" Height="1080" Margin="10,10,10,10"></local:SkiaCanvas>
    </Grid>
```

上面代码给 SkiaCanvas 一个固定的宽度和高度，为什么需要给他这个值，在上文告诉了大家

接下来在 UIElement_OnMouseMove 方法，也就是 Grid 容器收到的鼠标划过的事件，将划过的点作为线段在画布中

```csharp
        private void Draw(Action<SKCanvas> action)
        {
            Image.Draw(action);
        }

        private void UIElement_OnMouseMove(object sender, MouseEventArgs e)
        {
            var position = e.GetPosition(this);

            Draw(canvas =>
            {
                using var skPaint = new SKPaint() {Color = new SKColor(0, 0, 0), TextSize = 100};
                canvas.DrawLine(new SKPoint((float) _lastPosition.X, (float) _lastPosition.Y),
                    new SKPoint((float) position.X, (float) position.Y), skPaint);
            });

            _lastPosition = position;
        }

        private Point _lastPosition = new Point(0, 0);
```

可以看到逻辑十分简单，如果我需要让画布重新绘制，可以调用 Clear 方法，请看代码

```csharp
        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            Draw(canvas =>
            {
                canvas.Clear();
                using var skPaint = new SKPaint() {Color = new SKColor(0, 0, 0), TextSize = 100};
                canvas.DrawLine(10, 10, 100, 100, skPaint);
            });
        }
```

因此这个控件就支持重新绘制和差量更新绘制内容的功能

如果每次都能返回具体更新的范围，那么这个控件的绘制效率还是不错的

本文的代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/7b4b746f/ReewheaberekaiNayweelehe) 欢迎小伙伴访问

更多 WPF 渲染请看 [渲染相关](https://lindexi.gitee.io/post/%E6%B8%B2%E6%9F%93.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
