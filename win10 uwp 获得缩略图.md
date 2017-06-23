# win10 uwp 获得缩略图

有时候需要获得文件或视频的缩略图。

<!--more-->
<!-- csdn -->

## 文件缩略图

如果有一个文件需要获得缩略图，可以使用 `GetThumbnailAsync` 或 `GetScaledImageAsThumbnailAsync` ，就可以获得。代码请看下面：

```csharp
            FileOpenPicker openPicker = new FileOpenPicker();
            openPicker.FileTypeFilter.Add(".mp4");

            StorageFile file = await openPicker.PickSingleFileAsync();
            var thumbnail =  await file.GetScaledImageAsThumbnailAsync(ThumbnailMode.VideosView);
            BitmapImage bitmapImage = new BitmapImage();
            InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
            await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
            randomAccessStream.Seek(0);
            bitmapImage.SetSource(randomAccessStream);
            Image.Source = bitmapImage;
```

## 视频小图

如果需要获得视频的某一个页面，那么可以使用下面代码

```csharp
            FileOpenPicker openPicker = new FileOpenPicker();
            openPicker.FileTypeFilter.Add(".mp4");

            StorageFile file = await openPicker.PickSingleFileAsync();
            var thumbnail = await GetThumbnailAsync(file);
            BitmapImage bitmapImage = new BitmapImage();
            InMemoryRandomAccessStream randomAccessStream = new InMemoryRandomAccessStream();
            await RandomAccessStream.CopyAsync(thumbnail, randomAccessStream);
            randomAccessStream.Seek(0);
            bitmapImage.SetSource(randomAccessStream);
            Image.Source = bitmapImage;

          
          public async Task<IInputStream> GetThumbnailAsync(StorageFile file)
        {
            var mediaClip = await MediaClip.CreateFromFileAsync(file);
            var mediaComposition = new MediaComposition();
            var time=5000; 获取那一时间的页面
            mediaComposition.Clips.Add(mediaClip);
            return await mediaComposition.GetThumbnailAsync(
                TimeSpan.FromMilliseconds(time), 0, 0, VideoFramePrecision.NearestFrame);
        }
```

这样就可以获得指定时间的页面，因为得到是 IInputStream ，所以需要把他转为 bitmapImage ，这样才可以设置为图片

参见：https://stackoverflow.com/a/37314446/6116637


<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。  