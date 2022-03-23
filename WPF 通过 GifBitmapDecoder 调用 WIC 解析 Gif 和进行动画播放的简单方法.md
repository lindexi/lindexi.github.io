# WPF 通过 GifBitmapDecoder 调用 WIC 解析 Gif 和进行动画播放的简单方法

本文告诉大家如何在 WPF 里，通过 GifBitmapDecoder 调用 WIC 层来解析 GIF 图片，然后采用动画的方式进行播放

<!--more-->
<!-- CreateTime:2022/3/21 16:29:06 -->


<!-- 标签：WPF，gif -->
<!-- 博客 -->
<!-- 发布 -->

在[上一篇](https://blog.lindexi.com/post/wpf-GifBitmapDecoder-%E8%A7%A3%E6%9E%90-gif-%E6%A0%BC%E5%BC%8F.html)博客告诉大家，可以通过 GifBitmapDecoder 调用 WIC 层解析 GIF 图片

使用 WIC 层解析 GIF 图片可以调用系统默认解码器，对 GIF 的支持较好，也能支持很多诡异的格式，而且对这些诡异的图片的行为保持和其他应用相同

本文在[上一篇](https://blog.lindexi.com/post/wpf-GifBitmapDecoder-%E8%A7%A3%E6%9E%90-gif-%E6%A0%BC%E5%BC%8F.html)博客的基础上，告诉大家如何使用动画播放方式，进行播放 GIF 图片

这是一个简单的方式，优势在于使用动画播放，十分简单。缺点在于只能支持简单的 GIF 图片格式，也就是每一帧都是全画的 GIF 文件，如果只是范围更新的，那么效果很差

本文的实现可以从本文最后拿到所有代码，下面来告诉大家这是如何做的。 先创建一个继承 FrameworkElement 类型的 GifImage 类，将在这个类里面播放 GIF 图片

定义 GifSource 依赖属性，在依赖属性变更时，进行初始化逻辑

```csharp
using System;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Media;
using System.Windows.Media.Animation;
using System.Windows.Media.Imaging;

class GifImage : FrameworkElement
{
    public static readonly DependencyProperty GifSourceProperty = DependencyProperty.Register(
        "GifSource", typeof(Uri), typeof(GifImage), new UIPropertyMetadata(default(Uri), GifSourcePropertyChanged));

    public Uri GifSource
    {
        get { return (Uri) GetValue(GifSourceProperty); }
        set { SetValue(GifSourceProperty, value); }
    }

    private static void GifSourcePropertyChanged(DependencyObject sender, DependencyPropertyChangedEventArgs e)
    {
        (sender as GifImage).Initialize();
    }

    private void Initialize()
    {
        // 初始化
    }
}
```

在上面的 Initialize 是本文的核心逻辑，将初始化 GIF 的解析

初始化逻辑采用 GifBitmapDecoder 进行解析，代码如下

```csharp
    private void Initialize()
    {
        _gifDecoder = new GifBitmapDecoder(GifSource, BitmapCreateOptions.PreservePixelFormat, BitmapCacheOption.Default);
    }
    private GifBitmapDecoder _gifDecoder;
```

可以通过 `_gifDecoder.Frames` 拿到 GIF 的多个图片，每个图片信息，都可以通过 `BitmapMetadata` 的 GetQuery 方法获取参数，可以选择的参数有很多，如下

- `/grctlext` 控制信息
- `/grctlext/Disposal` 处置方法，表示如何处理上一张图片，如替换为背景色等
- `/grctlext/TransparencyFlag` 透明色选项
- `/grctlext/Delay` 延迟时间，单位是 10 分之一毫秒
- `/grctlext/TransparentColorIndex` 透明色索引
- `/imgdesc` 图片描述
- `/imgdesc/Left` 当前张图片所在的左上坐标和宽高，这里指的是左值
- `/imgdesc/Top` 当前张图片所在的左上坐标和宽高，这里指的是上值
- `/imgdesc/Width` 当前张图片所在的左上坐标和宽高，这里指的是宽度
- `/imgdesc/Height` 当前张图片所在的左上坐标和宽高，这里指的是高度

其他的还有 `/grctlext/UserInputFlag` `/imgdesc/LocalColorTableFlag` `/imgdesc/InterlaceFlag` `/imgdesc/SortFlag` `/imgdesc/LocalColorTableSize` 等。详细请看 [Native Image Format Metadata Queries - Win32 apps Microsoft Docs](https://docs.microsoft.com/en-us/windows/win32/wic/-wic-native-image-format-metadata-queries#gif-metadata?WT.mc_id=WD-MVP-5003260 )

使用 `/grctlext/Delay` 获取延时时间，根据延时时间创建动画。动画的方式就是修改当前使用第几张图片

```csharp
    private void Initialize()
    {
        _gifDecoder = new GifBitmapDecoder(GifSource, BitmapCreateOptions.PreservePixelFormat, BitmapCacheOption.Default);

        var keyFrames = new Int32KeyFrameCollection();
        TimeSpan last = TimeSpan.Zero;
        for (int i = 0; i < _gifDecoder.Frames.Count; i++)
        {
            var gifDecoderFrame = _gifDecoder.Frames[i];
            var bitmapMetadata = gifDecoderFrame.Metadata as BitmapMetadata;
            var delayTime = bitmapMetadata?.GetQuery("/grctlext/Delay") as ushort?;
            var delay = delayTime ?? 10;
            if (delay == 0)
            {
                delay = 10;
            }
            last += TimeSpan.FromMilliseconds(delay * 10);
            keyFrames.Add(new DiscreteInt32KeyFrame(i, KeyTime.FromTimeSpan(last)));
        }

        _animation = new Int32AnimationUsingKeyFrames()
        {
            KeyFrames = keyFrames,
            RepeatBehavior = RepeatBehavior.Forever,
        };
    }

    private GifBitmapDecoder _gifDecoder;
    private Int32AnimationUsingKeyFrames _animation;
```

添加一个叫播放的函数，调用此函数时，将执行动画

```csharp
    /// <summary>
    /// Starts the animation
    /// </summary>
    public void StartAnimation()
    {
        BeginAnimation(FrameIndexProperty, _animation);
    }

    public static readonly DependencyProperty FrameIndexProperty =
        DependencyProperty.Register("FrameIndex", typeof(int), typeof(GifImage), new FrameworkPropertyMetadata(0, new PropertyChangedCallback(ChangingFrameIndex)));

    static void ChangingFrameIndex(DependencyObject obj, DependencyPropertyChangedEventArgs e)
    {
        var gifImage = obj as GifImage;
        gifImage.ChangingFrameIndex((int) e.NewValue);
    }

    private void ChangingFrameIndex(int index)
    {
        InvalidateVisual();
    }
```

通过动画修改 FrameIndexProperty 从而通过依赖属性修改进入 InvalidateVisual 方法，让框架重新调用 OnRender 方法

```csharp
    protected override void OnRender(DrawingContext drawingContext)
    {
        var gifDecoderFrame = _gifDecoder.Frames[FrameIndex];

        drawingContext.DrawImage(gifDecoderFrame,new Rect(new Size(gifDecoderFrame.PixelWidth, gifDecoderFrame.PixelHeight)));
    }
```

如此即可完成播放

此类型的代码如下

```csharp
class GifImage : FrameworkElement
{
    private bool _isInitialized;
    private GifBitmapDecoder _gifDecoder;
    private Int32AnimationUsingKeyFrames _animation;

    public int FrameIndex
    {
        get { return (int) GetValue(FrameIndexProperty); }
        set { SetValue(FrameIndexProperty, value); }
    }

    private void Initialize()
    {
        _gifDecoder = new GifBitmapDecoder(GifSource, BitmapCreateOptions.PreservePixelFormat, BitmapCacheOption.Default);

        var keyFrames = new Int32KeyFrameCollection();
        TimeSpan last = TimeSpan.Zero;
        for (int i = 0; i < _gifDecoder.Frames.Count; i++)
        {
            var gifDecoderFrame = _gifDecoder.Frames[i];
            var bitmapMetadata = gifDecoderFrame.Metadata as BitmapMetadata;
            var delayTime = bitmapMetadata?.GetQuery("/grctlext/Delay") as ushort?;
            var delay = delayTime ?? 10;
            if (delay == 0)
            {
                delay = 10;
            }
            last += TimeSpan.FromMilliseconds(delay * 10);
            keyFrames.Add(new DiscreteInt32KeyFrame(i, KeyTime.FromTimeSpan(last)));
        }

        _animation = new Int32AnimationUsingKeyFrames()
        {
            KeyFrames = keyFrames,
            RepeatBehavior = RepeatBehavior.Forever,
        };

        _isInitialized = true;
    }

    static GifImage()
    {
        VisibilityProperty.OverrideMetadata(typeof(GifImage),
            new FrameworkPropertyMetadata(VisibilityPropertyChanged));
    }

    private static void VisibilityPropertyChanged(DependencyObject sender, DependencyPropertyChangedEventArgs e)
    {
        if ((Visibility) e.NewValue == Visibility.Visible)
        {
            ((GifImage) sender).StartAnimation();
        }
        else
        {
            ((GifImage) sender).StopAnimation();
        }
    }

    public static readonly DependencyProperty FrameIndexProperty =
        DependencyProperty.Register("FrameIndex", typeof(int), typeof(GifImage), new FrameworkPropertyMetadata(0, new PropertyChangedCallback(ChangingFrameIndex)));

    static void ChangingFrameIndex(DependencyObject obj, DependencyPropertyChangedEventArgs e)
    {
        var gifImage = obj as GifImage;
        gifImage.ChangingFrameIndex((int) e.NewValue);
    }

    private void ChangingFrameIndex(int index)
    {
        InvalidateVisual();
    }

    protected override void OnRender(DrawingContext drawingContext)
    {
        var gifDecoderFrame = _gifDecoder.Frames[FrameIndex];

        drawingContext.DrawImage(gifDecoderFrame,new Rect(new Size(gifDecoderFrame.PixelWidth, gifDecoderFrame.PixelHeight)));
    }

    /// <summary>
    /// Defines whether the animation starts on it's own
    /// </summary>
    public bool AutoStart
    {
        get { return (bool) GetValue(AutoStartProperty); }
        set { SetValue(AutoStartProperty, value); }
    }

    public static readonly DependencyProperty AutoStartProperty =
        DependencyProperty.Register("AutoStart", typeof(bool), typeof(GifImage), new UIPropertyMetadata(false, AutoStartPropertyChanged));

    private static void AutoStartPropertyChanged(DependencyObject sender, DependencyPropertyChangedEventArgs e)
    {
        if ((bool) e.NewValue)
            (sender as GifImage).StartAnimation();
    }

    public static readonly DependencyProperty GifSourceProperty = DependencyProperty.Register(
        "GifSource", typeof(Uri), typeof(GifImage), new UIPropertyMetadata(default(Uri), GifSourcePropertyChanged));

    public Uri GifSource
    {
        get { return (Uri) GetValue(GifSourceProperty); }
        set { SetValue(GifSourceProperty, value); }
    }

    private static void GifSourcePropertyChanged(DependencyObject sender, DependencyPropertyChangedEventArgs e)
    {
        (sender as GifImage).Initialize();
    }

    /// <summary>
    /// Starts the animation
    /// </summary>
    public void StartAnimation()
    {
        if (!_isInitialized)
            this.Initialize();

        BeginAnimation(FrameIndexProperty, _animation);
    }

    /// <summary>
    /// Stops the animation
    /// </summary>
    public void StopAnimation()
    {
        BeginAnimation(FrameIndexProperty, null);
    }
}
```

除此之外的其他播放 GIF 方法，请看：

[WPF 一个性能比较好的 gif 解析库](https://blog.lindexi.com/post/WPF-%E4%B8%80%E4%B8%AA%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83%E5%A5%BD%E7%9A%84-gif-%E8%A7%A3%E6%9E%90%E5%BA%93.html )

[WPF 播放 gif](https://blog.lindexi.com/post/WPF-%E6%92%AD%E6%94%BE-gif.html )

更多请看

[gif 格式](https://blog.lindexi.com/post/gif-%E6%A0%BC%E5%BC%8F.html )

[wpf GifBitmapDecoder 解析 gif 格式](https://blog.lindexi.com/post/wpf-GifBitmapDecoder-%E8%A7%A3%E6%9E%90-gif-%E6%A0%BC%E5%BC%8F.html )



本文以上的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/e11f2ea15fd5107fac4bd4523580587ce7febd56/CairjawworalhulalGeacharkucoha) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/e11f2ea15fd5107fac4bd4523580587ce7febd56/CairjawworalhulalGeacharkucoha) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin e11f2ea15fd5107fac4bd4523580587ce7febd56
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 CairjawworalhulalGeacharkucoha 文件夹

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
