#win10 uwp 截图 获取屏幕显示界面保存图片

【】

本文主要讲如何保存我们的屏幕显示的，保存为图片，也就是截图，截我们应用显示的。

UWP有一个功能，可以截图，RenderTargetBitmap

我们首先写一个Grid，我们需要给他名字，我这里给他`Stamp`

然后我们可以使用`RenderTargetBitmap`保存我们屏幕Grid显示的

```

            var bitmap = new RenderTargetBitmap();
            StorageFile file = await KnownFolders.PicturesLibrary.CreateFileAsync("1.jpg",
                CreationCollisionOption.GenerateUniqueName);
            await bitmap.RenderAsync(Stamp);
            var buffer = await bitmap.GetPixelsAsync();
            using (IRandomAccessStream stream = await file.OpenAsync(FileAccessMode.ReadWrite))
            {
                var encod = await BitmapEncoder.CreateAsync(
                    BitmapEncoder.JpegEncoderId, stream);
                encod.SetPixelData(BitmapPixelFormat.Bgra8,
                    BitmapAlphaMode.Ignore,
                    (uint)bitmap.PixelWidth,
                    (uint)bitmap.PixelHeight,
                    DisplayInformation.GetForCurrentView().LogicalDpi,
                    DisplayInformation.GetForCurrentView().LogicalDpi,
                    buffer.ToArray()
                   );
                await encod.FlushAsync();
            }

```

` await bitmap.RenderAsync(Stamp);`能把一个UIelement显示转为图片，不过这个图片我们需要用`BitmapEncoder`才可以保存为我们的图片

`BitmapEncoder`可以保存为bmp、jpg、gif、png

保存需要使用`SetPixelData` `BitmapPixelFormat pixelFormat, BitmapAlphaMode alphaMode, System.UInt32 width, System.UInt32 height, System.Double dpiX, System.Double dpiY, [Range(0, int.MaxValue)] System.Byte[] pixels`


我们在Grid放一个Image，然后可以看到，我们的原图

![这里写图片描述](http://img.blog.csdn.net/20160919155040537)

![这里写图片描述](http://img.blog.csdn.net/20160919160057373)

我们把图片一部分不显示




参见：http://www.zmy123.cn/?p=1257