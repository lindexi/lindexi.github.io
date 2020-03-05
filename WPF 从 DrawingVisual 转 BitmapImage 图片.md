# WPF 从 DrawingVisual 转 BitmapImage 图片

有一些库的设计是需要传入一个 BitmapImage 图片，但是我需要从界面代码创建图片，我没有文件，如何通过 DrawingVisual 画出的控件转换 BitmapImage 传给库？

需要将 DrawingVisual 转为 RenderTargetBitmap 然后将 RenderTargetBitmap 转为 BitmapImage 才可以

<!--more-->
<!-- CreateTime:2018/12/27 11:31:52 -->

<!-- csdn -->

先创建一个 DrawingVisual 在里面绘制一些内容

```csharp
                DrawingVisual drawingVisual = new DrawingVisual();
                DrawingContext drawingContext = drawingVisual.RenderOpen();

                // 画出界面
                
                drawingContext.Close();
```

如在里面写文字

```csharp

                DrawingVisual drawingVisual = new DrawingVisual();
                DrawingContext drawingContext = drawingVisual.RenderOpen();

                drawingContext.DrawText(new FormattedText("欢迎访问我博客 http://lindexi.gitee.io 里面有大量 UWP WPF 博客",
                    CultureInfo.GetCultureInfo("zh-cn"),
                    FlowDirection.LeftToRight,
                    new Typeface("Verdana"),
                    36, System.Windows.Media.Brushes.Black),
                    new System.Windows.Point(200, 116));
                
                drawingContext.Close();
```

写完之后可以将他转换为 RenderTargetBitmap 请看代码

```csharp
                  RenderTargetBitmap bmp = new RenderTargetBitmap(宽度, 高度, 96, 96, PixelFormats.Pbgra32);
                bmp.Render(drawingVisual);
```

需要自己知道截图的宽度和高度才可以，另外这里的 96 是 dpi 的大小

将 DrawingVisual 转 RenderTargetBitmap 就可以通过 PngBitmapEncoder 将 RenderTargetBitmap 转图片

```csharp
                var bitmapImage = new BitmapImage();
                var bitmapEncoder = new PngBitmapEncoder();
                bitmapEncoder.Frames.Add(BitmapFrame.Create(bmp));

                using (var stream = new MemoryStream())
                {
                    bitmapEncoder.Save(stream);
                    stream.Seek(0, SeekOrigin.Begin);

                    bitmapImage.BeginInit();
                    bitmapImage.CacheOption = BitmapCacheOption.OnLoad;
                    bitmapImage.StreamSource = stream;
                    bitmapImage.EndInit();
                }
```

通过这个方法就可以将 DrawingVisual 转 BitmapImage 虽然这个方法的速度比较慢

[WPF 通过 DrawingContext DrawImage 绘制图片](https://lindexi.oschina.io/lindexi/post/WPF-%E9%80%9A%E8%BF%87-DrawingContext-DrawImage-%E7%BB%98%E5%88%B6%E5%9B%BE%E7%89%87.html )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
