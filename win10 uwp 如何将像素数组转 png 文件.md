# win10 uwp 如何将像素数组转 png 文件

堆栈的小伙伴好奇他有一个数组，数组里面是 BGRA 的像素，他需要将这个数组转换为 PNG 文件

在 UWP 可以使用 BitmapEncoder 将像素数组加密为文件

<!--more-->
<!-- CreateTime:2019/3/25 8:53:01 -->

<!-- csdn -->

在使用 BitmapEncoder 之前需要要求有像素数组，像素数组的规律有要求，按照 BGRA 按照顺序的数组，同时要求知道像素的原图的像素宽度。因为存放像素数组使用的是一维的数组，如果不知道图片宽度，那么就不知道这个图片的像素是对应数组哪个

通过下面方法可以转换像素数组到文件

```csharp

        private async Task ByteToPng(byte[] byteList, int width, int height, IRandomAccessStream file)
        {
            try
            {
                var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.PngEncoderId, file);
                encoder.SetPixelData(BitmapPixelFormat.Bgra8, BitmapAlphaMode.Ignore, (uint) width, (uint) height, 96,
                    96, byteList);
                await encoder.FlushAsync();
            }
            catch (Exception e)
            {
            }
        }
```
 
这里的 IRandomAccessStream 就是 StorageFile 打开文件

```csharp
        private async Task SaveToFileAsync(byte[] byteList, int width, int height, IStorageFile file)
        {
            using (var stream = (await file.OpenStreamForWriteAsync()).AsRandomAccessStream())
            {
                await ByteToPng(byteList, width, height, stream);
            }
        }
```

通过这个方法，可以传入数组和图片的宽度和高度，保存的文件，就可以将像素数组保存到 png 文件

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  
