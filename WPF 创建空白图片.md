# WPF 创建空白图片

本文告诉大家如何在 WPF 创建空白图片，可以创建1像素图片

<!--more-->
<!-- CreateTime:2020/2/14 20:11:55 -->

<!-- csdn -->

可以使用 BitmapSource 的 Create 方法创建空白图片

```csharp
            // 限制不能创建小于2x2的图片
            const int width = 2;
            const int height = width;
            
            BitmapSource.Create(width, height, 96, 96,
                PixelFormats.Indexed1,
                new BitmapPalette(new List<Color> { Colors.Transparent }),
                new byte[width * height], 1);
```

上面这个方法只有创建 2x2 的图片，而创建1像素图片可以使用下面方法

```csharp
            const int width = 1;
            const int height = width;
            const double dpi = 96;
            // R G B 三个像素
            const int colorLength = 3;

            var image = BitmapSource.Create(width, height, dpi, dpi, PixelFormats.Bgr24, null, new byte[colorLength],
                colorLength);
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。

