#win10 uwp 截图 获取屏幕显示界面保存图片

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

参见：http://www.zmy123.cn/?p=1257