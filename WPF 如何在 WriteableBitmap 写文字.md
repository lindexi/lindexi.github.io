# WPF 如何在 WriteableBitmap 写文字

最近看到[WPF 使用不安全代码快速从数组转 WriteableBitmap ](https://lindexi.github.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%AE%89%E5%85%A8%E4%BB%A3%E7%A0%81%E5%BF%AB%E9%80%9F%E4%BB%8E%E6%95%B0%E7%BB%84%E8%BD%AC-WriteableBitmap.html )可以快速从数组转 WriteableBitmap 所以就让他画一些元素，但是发现元素有文字就没法了。

本文告诉大家如何在 WriteableBitmap 把文字画上去。

<!--more-->
<!-- CreateTime:2018/12/25 9:13:57 -->

<!-- csdn -->

## 截图

这个方法是从 [WriteableBitmapEx](https://github.com/teichgraf/WriteableBitmapEx/ )看到的，可以在页面创建一个 TextBlock 让他来显示文字，然后使用截图获得文字，把图片画到 WriteableBitmap 就好。

先创建一个对象

```csharp
 var wb = new WriteableBitmap((int) 宽, (int) 高, 96, 96, PixelFormats.Pbgra32, null);
```

然后对文字截图

```csharp
 var rtb = new RenderTargetBitmap(wb.PixelWidth, wb.PixelHeight, wb.DpiX, wb.DpiY, PixelFormats.Pbgra32);
 rtb.Render(txt);
```

然后从截图复制到 WriteableBitmap 可以使用不安全代码

```csharp
   wb.Lock();
   rtb.CopyPixels(new Int32Rect(0,0, rtb.PixelWidth, rtb.PixelHeight), 
   wb.BackBuffer,
   wb.BackBufferStride * wb.PixelHeight,  wb.BackBufferStride);

   wb.AddDirtyRect(new Int32Rect(0, 0, (int)ActualWidth, (int)ActualHeight));
   wb.Unlock();
```

## win form 方法

另一个方法是使用 win form 写文字然后使用 [WPF 使用不安全代码快速从数组转 WriteableBitmap - 林德熙](https://lindexi.github.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%AE%89%E5%85%A8%E4%BB%A3%E7%A0%81%E5%BF%AB%E9%80%9F%E4%BB%8E%E6%95%B0%E7%BB%84%E8%BD%AC-WriteableBitmap.html ) 把文字写到 WriteableBitmap ，这个方法比较简单

```csharp
            var width = 100;
            var height = 50;

            Bitmap bmp = new Bitmap(width, height);
            FontFamily f = new FontFamily("微软雅黑");
            using (Graphics g = Graphics.FromImage(bmp))
            {
                g.DrawString("林德熙", new System.Drawing.Font(f, 10), Brushes.Black, 0, 0);
            }

            WriteableBitmap image = new WriteableBitmap(width, height, 96, 96, PixelFormats.Bgra32, null);
            CopyFrom(image, bmp);
```

上面的代码可能无法直接运行，于是我就给 CopyFrom 代码，代码实际是从[WPF 使用不安全代码快速从数组转 WriteableBitmap - 林德熙](https://lindexi.github.io/lindexi/post/WPF-%E4%BD%BF%E7%94%A8%E4%B8%8D%E5%AE%89%E5%85%A8%E4%BB%A3%E7%A0%81%E5%BF%AB%E9%80%9F%E4%BB%8E%E6%95%B0%E7%BB%84%E8%BD%AC-WriteableBitmap.html ) 复制

```csharp
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
            var bytes = ws * hs * wb.Format.BitsPerPixel / 8;

            var rBitmapData = bitmap.LockBits(new System.Drawing.Rectangle(0, 0, width, height),
                ImageLockMode.ReadOnly, bitmap.PixelFormat);

            wb.Lock();
            unsafe
            {
                System.Buffer.MemoryCopy(rBitmapData.Scan0.ToPointer(), wb.BackBuffer.ToPointer(), bytes, bytes);
            }
            wb.AddDirtyRect(new Int32Rect(0, 0, width, height));
            wb.Unlock();

            bitmap.UnlockBits(rBitmapData);
        }
```

这样运行就可以看到文字，而且这个方法的性能比较好

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F201712151723520171220103729.jpg)

因为我没有设置文字大小和显示的大小，所以看起来文字就没有那么清晰

但是说这个方法的速度比较好，实际也是很差

最近看到一个对 OpenGL 封装的 SharpGL ，感觉还不错，如果需要比较高的速度，那么推荐使用这个库

[SharpGL(Opengl)入门之纹理星球 - BIT祝威 - 博客园](http://www.cnblogs.com/bitzhuwei/archive/2013/05/21/Opengl_Sharpgl_dragtextured_planet_drag_drop.html )

[使用不安全代码将 Bitmap 位图转为 WPF 的 ImageSource 以获得高性能和持续小的内存占用 - walterlv的专栏 - CSDN博客](https://blog.csdn.net/WPwalter/article/details/78619679 )

[基于WriteableBitmap对象类采用擦写内存方式，低开销更新WPF Image UI控件 - Ivan_Whisper的博客 - CSDN博客](https://blog.csdn.net/Ivan_Whisper/article/details/80312586 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 