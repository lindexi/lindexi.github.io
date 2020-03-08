# WPF 使用不安全代码快速从数组转 WriteableBitmap

本文告诉大家一个快速的方法，直接把数组转 WriteableBitmap 

<!--more-->
<!-- CreateTime:2018/8/10 19:16:53 -->

<!-- csdn -->

先来说下以前的方法，以前使用的是 BitmapSource ，这个方法是大法官方提供的。

```csharp
BitmapSource.Create(LogicalWidth, LogicalHeight, 96, 96, PixelFormats.Bgra32, null,
                    dest,
                    stride);
```

其中 dest 是一个大数组，数据大小为 $$ LogicalWidth*LogicalHeight*4 $$ ，经常在转换的时候出现内存不足异常。假如现在内存占用是 1.5G ，转换的图片大小是 `2000*2000` ，于是几乎一跑就出现内存不足。

为何还有 500 M 内存却出现内存不足？因为图片转换需要的是一段大的连续内存空间。砸桌子，再说一次，图片转换需要一段【大的连续】内存空间，虽然有500M内存，但是连续的空间没有那么多。

这就是以前方法的缺点。

使用不安全代码转换是把数组直接复制到WriteableBitmap，请看[使用不安全代码将 Bitmap 位图转为 WPF 的 ImageSource 以获得高性能和持续小的内存占用 - walterlv](https://walterlv.github.io/post/convert-bitmap-to-imagesource-using-unsafe-method.html )，这里讲了如何从 Bitmap 转 WriteableBitmap ，于是下面只需要把数组转 Bitmap 就可以了。

我在最后会给大家全部代码，所以现在讲原理。

如果已经拿到了数组，知道数组的存放，那么就可以直接把数组复制到 WriteableBitmap 就可以显示。数组的存放就是数组是如何放数据的，是不是还在想，上面的 dest 是一个大数组，他的计算是
$$ LogicalWidth*LogicalHeight*4 $$ 为什么是`*4`，因为存放的数据是 A R B G 一个点需要4个int来放。那么放的顺序是什么？这就是`PixelFormat`指定的类型，可以使用`Bgra32`或者其他的格式，不过指定了格式就需要数组存放和指定一样

因为没有直接从数组转 WriteableBitmap 所以需要先把数组转 Bitmap ，可以使用的方法请看下面

```csharp
                unsafe
                {
                    fixed (int* ptr = _dest)
                    {

                        try
                        {
                            using (Bitmap image = new Bitmap(LogicalWidth, LogicalHeight, LogicalWidth * 4,
                                PixelFormat.Format16bppArgb1555, new IntPtr(ptr)))
                            {
                               
                            }
                        }
                        catch (Exception e)
                        {
                            Console.WriteLine(e);
                        }
                    }
                }
```

Bitmap 的数据类型可以是任意，因为只是把他的数据转换到 WriteableBitmap 所以不需要指定他的数据

获得 Bitmap 就可以把他转 WriteableBitmap ，请看下面的代码

```csharp
                unsafe
                {
                    fixed (int* ptr = _dest)
                    {

                        try
                        {
                            using (Bitmap image = new Bitmap(LogicalWidth, LogicalHeight, LogicalWidth * 4,
                                PixelFormat.Format16bppArgb1555, new IntPtr(ptr)))
                            {
                                CopyFrom(source,image);
                            }
                        }
                        catch (Exception e)
                        {
                            Console.WriteLine(e);
                        }
                    }
                }
```

代码的 CopyFrom 就是[吕毅](https://walterlv.github.io/post/convert-bitmap-to-imagesource-using-unsafe-method.html)提供的方法。

使用这个函数更新，不需要在更新了修改 Image 的 Source 因为会自动更新，用这个方法播放 gif 的性能比找到的[Magick.NET](http://lindexi.github.io/lindexi/post/wpf-%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8-Magick.NET-%E6%92%AD%E6%94%BE-gif-%E5%9B%BE%E7%89%87.html)库的性能都好。

对比一下性能，这时原先的 BitmapSource 方法占用内存

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017119165352.jpg)

这是使用不安全代码占用内存

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171110102410.jpg)

实际跑一张 gif 图的性能

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B411%25E6%259C%258810%25E6%2597%25A5%2520111339.gif)

可以看到这个方法可以节省很多的内存，而且占用的 cpu 很低，因为没有很多gc

但是不要太高兴，因为不安全代码的exception是接不住的，下面请修改一下代码，让他输入错误，于是就出现异常，结果程序就关了。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F20171110102647.jpg)

所以使用这个方法还是很大的坑。

全部的代码：

```csharp
           Application.Current?.Dispatcher.BeginInvoke((Action) (() =>
            {
                unsafe
                {
                    fixed (int* ptr = _dest)
                    {
                       
                        try
                        {
                            using (Bitmap image = new Bitmap(LogicalWidth, LogicalHeight, LogicalWidth * 4,
                                PixelFormat.Format16bppArgb1555, new IntPtr(ptr)))
                            {
                                CopyFrom(_source, image);
                            }
                        }
                        catch (Exception e)
                        {
                            Console.WriteLine(e);
                        }
                    }
                }
            }));


          private void UpdateSource()
        {
            Application.Current?.Dispatcher.Invoke(() =>
            {
                 _source = new WriteableBitmap(LogicalWidth, LogicalHeight, 96, 96,
                        System.Windows.Media.PixelFormats.Bgra32, null);
            });
        }

        public static void CopyFrom(WriteableBitmap wb, Bitmap bitmap)
        {
            if (wb == null)
                throw new ArgumentNullException(nameof(wb));
            if (bitmap == null)
                throw new ArgumentNullException(nameof(bitmap));

            var ws = wb.PixelWidth;
            var hs = wb.PixelHeight;
            var wt = bitmap.Width;
            var ht = bitmap.Height;
            if (ws != wt || hs != ht)
                throw new ArgumentException("暂时只支持相同尺寸图片的复制。");

            var width = ws;
            var height = hs;
            var bytes = ws * hs * wb.Format.BitsPerPixel / 8 ;

            var rBitmapData = bitmap.LockBits(new Rectangle(0, 0, width, height),
                ImageLockMode.ReadOnly, bitmap.PixelFormat);

            wb.Lock();
            unsafe
            {
                Buffer.MemoryCopy(rBitmapData.Scan0.ToPointer(), wb.BackBuffer.ToPointer(), bytes, bytes);
            }
            wb.AddDirtyRect(new Int32Rect(0, 0, width, height));
            wb.Unlock();

            bitmap.UnlockBits(rBitmapData);
        }
```

我把代码给小伙伴看，他说可以直接从数组转 WriteableBitmap ，我使用他的想法，修改了程序，请看代码

```csharp
              unsafe
                {
                    fixed (int* ptr = _dest)
                    {
                        _source.Lock();

                        var bytes = LogicalWidth * LogicalHeight * _source.Format.BitsPerPixel / 8;

                        Buffer.MemoryCopy(ptr,_source.BackBuffer.ToPointer(), bytes, bytes);

                        _source.AddDirtyRect(new Int32Rect(0, 0, LogicalWidth, LogicalHeight));
                        _source.Unlock();
                    }
                }
```

实际上微软已经提供了不安全代码的转换，请看下面代码

```csharp
bitmapImage.WritePixels(new Int32Rect(0, 0, 宽度, 高度), 图片数据, stride, 0)
```

stride 一般就是 `4*宽度` 因为一个像素使用4个byte




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 