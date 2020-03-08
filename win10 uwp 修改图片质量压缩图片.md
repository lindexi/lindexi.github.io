# win10 uwp 修改图片质量压缩图片

本文告诉大家如何在 UWP 通过修改图片的质量减少图片大小，这个方法只支持输出 jpg 文件

<!--more-->
<!-- CreateTime:2019/3/21 15:29:20 -->

<!-- csdn -->

通过创建 BitmapEncoder 的时候指定 BitmapPropertySet 可以设置图片的质量，只有对 JPG 格式才能设置图片质量

图片质量的值是从 0 到 1 其中 1 表示质量最好

```csharp
    var propertySet = new BitmapPropertySet();
    // 图片质量，值范围是 0到1 其中 1 的质量最好
    var qualityValue = new BitmapTypedValue(imageQuality,
        Windows.Foundation.PropertyType.Single);
    propertySet.Add("ImageQuality", qualityValue);
    var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, imageWriteAbleStream,
        propertySet);
```

这里的 imageQuality 就是图片质量，这个需要传入

从一个图片文件压缩图片大小的方法可以这样写，创建一个方法传入原图文件，和需要输出的文件，和图片质量

```csharp
        private async Task<StorageFile> ConvertImageToJpegAsync(StorageFile sourceFile, StorageFile outputFile,
            double imageQuality)
```

先获取图片大小，这样可以知道压缩了多少，对比原图的文件大小和压缩之后的图片大小

```csharp
            var sourceFileProperties = await sourceFile.GetBasicPropertiesAsync();
            var fileSize = sourceFileProperties.Size;
```

获取文件大小更简单的方法是通过 WinRTXamlToolkit 的 StorageItemExtensions.GetSizeAsync 拿到文件大小

读取原图文件，需要先解码原图，然后通过编码的时候修改图片质量

```csharp
            var imageStream = await sourceFile.OpenReadAsync();

```

解码的方法是不需要知道图片的格式

```csharp
                var decoder = await BitmapDecoder.CreateAsync(imageStream);
                var pixelData = await decoder.GetPixelDataAsync();
                var detachedPixelData = pixelData.DetachPixelData();
```

打开输出文件，进行编码

```csharp
                var imageWriteAbleStream = await outputFile.OpenAsync(FileAccessMode.ReadWrite);
```

在创建编码的时候设置图片质量

```csharp
    var propertySet = new BitmapPropertySet();
    // 图片质量，值范围是 0到1 其中 1 的质量最好
    var qualityValue = new BitmapTypedValue(imageQuality,
        Windows.Foundation.PropertyType.Single);
    propertySet.Add("ImageQuality", qualityValue);
    var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, imageWriteAbleStream,
        propertySet);
```

将编码写入到文件

```csharp
    encoder.SetPixelData(decoder.BitmapPixelFormat, decoder.BitmapAlphaMode, decoder.OrientedPixelWidth,
        decoder.OrientedPixelHeight, decoder.DpiX, decoder.DpiY, detachedPixelData);
    await encoder.FlushAsync();
    await imageWriteAbleStream.FlushAsync();
```

拿到压缩只有的文件的大小，对比一下

```csharp
    var jpegImageSize = imageWriteAbleStream.Size;
    // 欢迎访问我博客 https://blog.lindexi.com/ 里面有大量 UWP WPF 博客
    Debug.WriteLine($"压缩之后比压缩前的文件小{fileSize - jpegImageSize}");
```

这个压缩图片的方法的代码虽然看起来很多，但是看起来还是很简单先打开原来的图片文件对原图进行解密然后输出到新的文件

```csharp
        /// <summary>
        /// 将原来的图片转换图片质量和压缩质量
        /// </summary>
        /// <param name="sourceFile">原来的图片</param>
        /// <param name="outputFile">输出的文件</param>
        /// <param name="imageQuality">图片质量，取值范围是 0 到 1 其中 1 的质量最好，这个值设置只对 jpg 图片有效</param>
        /// <returns></returns>
        private async Task<StorageFile> ConvertImageToJpegAsync(StorageFile sourceFile, StorageFile outputFile,
            double imageQuality)
        {
            var sourceFileProperties = await sourceFile.GetBasicPropertiesAsync();
            var fileSize = sourceFileProperties.Size;
            var imageStream = await sourceFile.OpenReadAsync();
            using (imageStream)
            {
                var decoder = await BitmapDecoder.CreateAsync(imageStream);
                var pixelData = await decoder.GetPixelDataAsync();
                var detachedPixelData = pixelData.DetachPixelData();
                var imageWriteAbleStream = await outputFile.OpenAsync(FileAccessMode.ReadWrite);
                using (imageWriteAbleStream)
                {
                    var propertySet = new BitmapPropertySet();
                    // 图片质量，值范围是 0到1 其中 1 的质量最好
                    var qualityValue = new BitmapTypedValue(imageQuality,
                        Windows.Foundation.PropertyType.Single);
                    propertySet.Add("ImageQuality", qualityValue);
                    var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.JpegEncoderId, imageWriteAbleStream,
                        propertySet);
                    //key thing here is to use decoder.OrientedPixelWidth and decoder.OrientedPixelHeight otherwise you will get garbled image on devices on some photos with orientation in metadata
                    encoder.SetPixelData(decoder.BitmapPixelFormat, decoder.BitmapAlphaMode, decoder.OrientedPixelWidth,
                        decoder.OrientedPixelHeight, decoder.DpiX, decoder.DpiY, detachedPixelData);
                    await encoder.FlushAsync();
                    await imageWriteAbleStream.FlushAsync();
                    var jpegImageSize = imageWriteAbleStream.Size;
                    // 欢迎访问我博客 https://blog.lindexi.com/ 里面有大量 UWP WPF 博客
                    Debug.WriteLine($"压缩之后比压缩前的文件小{fileSize - jpegImageSize}");
                }
            }

            return outputFile;
        }

```

于是下面写一个测试的程序

在界面创建一个按钮

```csharp
        <Button Content="压缩图片" HorizontalAlignment="Center" VerticalAlignment="Center" Click="Button_OnClick" />

```

在按钮拿到一个文件，然后在自己的临时文件夹里面创建输出文件，如果真的需要用这个程序压缩图片那么请让用户再选一个文件

```csharp
        private async void Button_OnClick(object sender, RoutedEventArgs e)
        {
            var pick = new FileOpenPicker();
            pick.FileTypeFilter.Add(".jpg");
            var file = await pick.PickSingleFileAsync();

            if (file != null)
            {
                await ConvertImageToJpegAsync(file,
                    await ApplicationData.Current.TemporaryFolder.CreateFileAsync("lindexi"),
                    0.75);
            }
        }
```

现在尝试运行代码，点击界面的按钮，就可以看到点击按钮选择

代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/b396890a0a6d88b41e16f6cff288ba8de09881e0/CetursearhebirLefelembemki)

这个代码参考了[Alex Sorokoletov](https://github.com/alexsorokoletov )的代码

[How to convert image to JPEG and specify quality parameter in UWP C# XAML](https://gist.github.com/alexsorokoletov/71431e403c0fa55f1b4c942845a3c850 )

[BitmapEncoder options reference - Windows UWP applications](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/bitmapencoder-options-reference )

[Create, edit, and save bitmap images - Windows UWP applications](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/imaging )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
