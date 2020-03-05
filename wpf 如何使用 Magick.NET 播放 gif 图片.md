# wpf 如何使用 Magick.NET 播放 gif 图片

本文告诉大家使用 Magick.NET 的方法播放 gif 图片。

<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<div id="toc"></div>

<!-- csdn -->
<!-- 标签：WPF，gif -->

最近在做 gif 播放，发现 gif 播放需要很多内存，于是就使用  Magick.NET 播放，但是这个方式也需要很多的内存。播放一张 [uwp 萤火虫](https://lindexi.github.io/lindexi/post/win10-uwp-%E8%90%A4%E7%81%AB%E8%99%AB%E6%95%88%E6%9E%9C.html) 需要 600 M 内存。但是我还是把方法记下。


## 安装 Magick.NET

可以选择的很多，如果只是做测试，那么建议直接使用 AnyCPU 这样就不需要关心在哪里使用。里面的选项 Qn中的n就是表示质量，一般使用 8 就可以啦。

安装的方法建议使用 nuget 下载，nuget 可以使用国内博客园的源，当然现在大法更新了速度，安装也不难。

## 解析 gif

安装完成之后就可以使用，不过使用之前需要先设置缓存`MagickAnyCPU.CacheDirectory`，然后进行解析gif。关于解析参见：[WPF 一个性能比较好的 gif 解析库 - 林德熙](https://lindexi.github.io/lindexi/post/WPF-%E4%B8%80%E4%B8%AA%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83%E5%A5%BD%E7%9A%84-gif-%E8%A7%A3%E6%9E%90%E5%BA%93.html )，这篇文章的解析只能播放常规的 gif ，对于压缩的 gif 是无法进行播放的，如果需要播放压缩后的 gif 那么需要使用 `Coalesce` ，一旦使用了就需要大概800M的内存，虽然很快就gc了。

常规 gif 图是直接把图片存放，对于这个文件，只需要把他分为多个 图片播放出来就好，需要注意就是他的图片时间，多久才继续播放。解析这个格式很简单，还可以使用大法的[wpf GifBitmapDecoder 解析 gif 格式(https://lindexi.github.io/lindexi/post/wpf-GifBitmapDecoder-%E8%A7%A3%E6%9E%90-gif-%E6%A0%BC%E5%BC%8F.html )

压缩的 gif 是把两个图片，判断这张图片有哪些像素和上一张一样，如果存在，就忽略。这个算法可以减少图片的空间。但是解析难度有些大，因为需要获得播放的上一个图片才可以进行解析这一张图片。

本文的解析gif 方法已经在[WPF 一个性能比较好的 gif 解析库 - 林德熙](https://lindexi.github.io/lindexi/post/WPF-%E4%B8%80%E4%B8%AA%E6%80%A7%E8%83%BD%E6%AF%94%E8%BE%83%E5%A5%BD%E7%9A%84-gif-%E8%A7%A3%E6%9E%90%E5%BA%93.html )讲到，下面就是代码。

```csharp
            collection = new MagickImageCollection(File);

```

## 播放 gif

这次播放的方式不是使用 image，而是直接写一个底层的控件播放，请看代码

```csharp
    public class SuxlzHjp : UIElement
    {
        public SuxlzHjp()
        {
            MagickAnyCPU.CacheDirectory = "E:\\temp";
        }

        public void Play()
        {
            if (string.IsNullOrEmpty(CacheDirectory))
            {
                CacheDirectory = Path.Combine(Environment.CurrentDirectory, "temp");
            }
            if (!Directory.Exists(CacheDirectory))
            {
                Directory.CreateDirectory(CacheDirectory);
            }
            MagickAnyCPU.CacheDirectory = CacheDirectory;

            collection = new MagickImageCollection(File);

            int n = 0;
            Task.Run(async () =>
            {
                while (true)
                {
                    if (n == collection.Count)
                    {
                        n = 0;
                    }

                    var t = collection[n];
                    var delay = t.AnimationDelay * 10;

                    await Dispatcher.InvokeAsync(() =>
                    {
                        var width = t.Width;
                        var height = t.Height;
                        RenderTargetBitmap image = new RenderTargetBitmap(width, height, 96, 96, PixelFormats.Pbgra32);
                        image.Render(drawing);
                        using (var drawingContext = drawing.RenderOpen())
                        {
                            drawingContext.DrawImage(image, new Rect(0, 0, width, height));
                            drawingContext.DrawImage(t.ToBitmapSource(),
                                new Rect(t.BoundingBox.X, t.BoundingBox.Y, t.BoundingBox.Width, t.BoundingBox.Height));
                        }
                        InvalidateVisual();
                    });

                    await Task.Delay(delay);
                    n++;
                }
            });
        }

        public static readonly DependencyProperty FileProperty = DependencyProperty.Register(
            nameof(File), typeof(string), typeof(SuxlzHjp), new PropertyMetadata(default(string)));

        public static string CacheDirectory { get; set; }

        public string File
        {
            get => (string) GetValue(FileProperty);
            set => SetValue(FileProperty, value);
        }


        protected override void OnRender(DrawingContext drawingContext)
        {
            drawingContext.DrawDrawing(drawing.Drawing);
            base.OnRender(drawingContext);
        }

        private DrawingVisual drawing = new DrawingVisual();

        private MagickImageCollection collection;
    }

```

可以尝试这个类进行播放，使用方法是设置 File 然后`Play`，可以看到这个方法需要使用的内存有 600M ，还不停gc所以这个方式不是我推荐。

其他播放gif的方法请看[WPF 播放 gif](https://lindexi.github.io/lindexi/post/WPF-%E6%92%AD%E6%94%BE-gif.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  