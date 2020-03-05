# wpf GifBitmapDecoder 解析 gif 格式

在网上有很多图片都是gif，那么如何在 wpf 解析 gif？

本文告诉大家如何使用  GifBitmapDecoder 把gif分开为一张一张，获得他的信息。
<!--more-->
<!-- CreateTime:2018/2/13 17:23:03 -->

<!-- csdn -->

<!-- 标签：WPF，gif -->

如果需要把一个 gif 分开，使用的代码很简单

```csharp
            var file = "E:\\林德熙\\测试文件\\2017年9月1日 10.gif";
            var stream = new FileStream(file, FileMode.Open);
            var decoder = new GifBitmapDecoder(stream, BitmapCreateOptions.PreservePixelFormat, BitmapCacheOption.Default);

```

从 decoder 就可以获得每个图片，例如写一个按钮，按一下就切换一个图片。

```csharp
      private int n = 0;

        private void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var t = decoder.Frames[n];
            
            Image.Source = t;
            n++;
            if (n >= decoder.Frames.Count)
            {
                n = 0;
            }
        }
```

按钮点击如上面代码，可以看到 decoder 把 gif 分开很简单，但是如何获得一帧的时间。如果在 wpf 获得 gif 图片间隔，就需要一些特殊方法。

先创建一个类 用于获得 gif 的信息，需要知道，每个gif的里面的图片都有信息。

```csharp
        class FrameInfo
        {
            public TimeSpan Delay { get; set; }
            public FrameDisposalMethod DisposalMethod { get; set; }
            public double Width { get; set; }
            public double Height { get; set; }
            public double Left { get; set; }
            public double Top { get; set; }

            public Rect Rect
            {
                get { return new Rect(Left, Top, Width, Height); }
            }
        }
```

其中 `Delay` 就是两个图片播放的时间，`FrameDisposalMethod`表示两张图片是如何播放，完全替换前一张还是在前一张基础继续显示。

获得 gif 的信息需要使用 `GetQuery` ，这个方法不好用，于是使用下面代码把他转类型

```csharp
        private static T? GetQueryOrNull<T>(this BitmapMetadata metadata, string query)
            where T : struct
        {
            if (metadata.ContainsQuery(query))
            {
                object value = metadata.GetQuery(query);
                if (value != null)
                    return (T) value;
            }
            return null;
        }
```

可以看到 decoder 的每个图片的 `Metadata` 是 `ImageMetadata` ，而且wr也没说它里面有哪些数据。

实际可以使用`BitmapMetadata`获得每个图片信息，因为`Metadata`实际是`BitmapMetadata`，通过`/grctlext/Delay`可以获得两个图片的时间，`/grctlext/Disposal`可以获得两个图片是如何显示，`/imgdesc/Width` 可以获得宽度。于是使用下面函数可以获得图片信息

```csharp
        public static FrameInfo GetFrameInfo(BitmapFrame frame)
        {
            var frameInfo = new FrameInfo
            {
                Delay = TimeSpan.FromMilliseconds(100),
                DisposalMethod = FrameDisposalMethod.Replace,
                Width = frame.PixelWidth,
                Height = frame.PixelHeight,
                Left = 0,
                Top = 0
            };

            BitmapMetadata metadata;
            try
            {
                metadata = frame.Metadata as BitmapMetadata;
                if (metadata != null)
                {
                    const string delayQuery = "/grctlext/Delay";
                    const string disposalQuery = "/grctlext/Disposal";
                    const string widthQuery = "/imgdesc/Width";
                    const string heightQuery = "/imgdesc/Height";
                    const string leftQuery = "/imgdesc/Left";
                    const string topQuery = "/imgdesc/Top";

                    var delay = metadata.GetQueryOrNull<ushort>(delayQuery);
                    if (delay.HasValue)
                        frameInfo.Delay = TimeSpan.FromMilliseconds(10 * delay.Value);

                    var disposal = metadata.GetQueryOrNull<byte>(disposalQuery);
                    if (disposal.HasValue)
                        frameInfo.DisposalMethod = (FrameDisposalMethod) disposal.Value;

                    var width = metadata.GetQueryOrNull<ushort>(widthQuery);
                    if (width.HasValue)
                        frameInfo.Width = width.Value;

                    var height = metadata.GetQueryOrNull<ushort>(heightQuery);
                    if (height.HasValue)
                        frameInfo.Height = height.Value;

                    var left = metadata.GetQueryOrNull<ushort>(leftQuery);
                    if (left.HasValue)
                        frameInfo.Left = left.Value;

                    var top = metadata.GetQueryOrNull<ushort>(topQuery);
                    if (top.HasValue)
                        frameInfo.Top = top.Value;
                }
            }
            catch (NotSupportedException)
            {
            }

            return frameInfo;
        }

```

这个方法实际上性能不好，如果需要一个可以用的gif解析，请看我的博客[WPF 播放 gif](https://lindexi.github.io/lindexi/post/WPF-%E6%92%AD%E6%94%BE-gif.html )

参见： http://www.thomaslevesque.com/2011/03/27/wpf-display-an-animated-gif-image/
	
http://stackoverflow.com/questions/210922/how-do-i-get-an-animated-gif-to-work-in-wpf

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 