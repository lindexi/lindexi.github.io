# win10 uwp ImageSourece 和Byte[] 相互转换

<!--more-->

我们先写一个简单的xaml

```
       <Image x:Name="Img" Height="200" Width="200" 
              HorizontalAlignment="Center" Source="Assets/SplashScreen.png" ></Image>
        
        <Button Margin="10,300,10,10" Content="确定" Click="Button_OnClick" ></Button>
   
```

![](http://7xqpl8.com1.z0.glb.clouddn.com/fc7733af-8526-44d2-84b9-99b41ef99f4a2016121293717.jpg)

## 保存WriteableBitmap到文件

        private static async Task SaveWriteableBitmapImageFile(WriteableBitmap image, StorageFile file)
        {
            //BitmapEncoder 存放格式
            Guid bitmapEncoderGuid = BitmapEncoder.JpegEncoderId;
            string filename = file.Name;
            if (filename.EndsWith("jpg"))
            {
                bitmapEncoderGuid = BitmapEncoder.JpegEncoderId;
            }
            else if (filename.EndsWith("png"))
            {
                bitmapEncoderGuid = BitmapEncoder.PngEncoderId;
            }
            else if (filename.EndsWith("bmp"))
            {
                bitmapEncoderGuid = BitmapEncoder.BmpEncoderId;
            }
            else if (filename.EndsWith("tiff"))
            {
                bitmapEncoderGuid = BitmapEncoder.TiffEncoderId;
            }
            else if (filename.EndsWith("gif"))
            {
                bitmapEncoderGuid = BitmapEncoder.GifEncoderId;
            }
            using (IRandomAccessStream stream = await file.OpenAsync(FileAccessMode.ReadWrite, StorageOpenOptions.None))
            {
                BitmapEncoder encoder = await BitmapEncoder.CreateAsync(bitmapEncoderGuid, stream);
                Stream pixelStream = image.PixelBuffer.AsStream();
                byte[] pixels = new byte[pixelStream.Length];
                await pixelStream.ReadAsync(pixels, 0, pixels.Length);
                
                encoder.SetPixelData(BitmapPixelFormat.Bgra8, BitmapAlphaMode.Ignore,
                          (uint)image.PixelWidth,
                          (uint)image.PixelHeight,
                          96.0,
                          96.0,
                          pixels);
                //Windows.Graphics.Imaging.BitmapDecoder decoder = await Windows.Graphics.Imaging.BitmapDecoder.CreateAsync(imgstream);
                //Windows.Graphics.Imaging.PixelDataProvider pxprd = await decoder.GetPixelDataAsync(Windows.Graphics.Imaging.BitmapPixelFormat.Bgra8, Windows.Graphics.Imaging.BitmapAlphaMode.Straight, new Windows.Graphics.Imaging.BitmapTransform(), Windows.Graphics.Imaging.ExifOrientationMode.RespectExifOrientation, Windows.Graphics.Imaging.ColorManagementMode.DoNotColorManage);

                await encoder.FlushAsync();
            }
        }

## 从文件读WriteableBitmap


```
        private static async Task<WriteableBitmap> OpenWriteableBitmapFile(StorageFile file)
        {
            using (IRandomAccessStream stream = await file.OpenAsync(FileAccessMode.Read))
            {
                BitmapDecoder decoder = await BitmapDecoder.CreateAsync(stream);
                WriteableBitmap image = new WriteableBitmap((int)decoder.PixelWidth, (int)decoder.PixelHeight);
                image.SetSource(stream);

                return image;
            }
        }
```

## ImageSource 转byte[]

ImageSource可以是BitmapImage、WriteableBitmap，如果是WriteableBitmap ，那么直接转换

WriteableBitmap  转byte[]

```
bitmap.PixelBuffer.ToArray();
```



## Image 转byte[]

如果我们的ImageSource是BitmapImage,那么我们不能使用上面的办法，直接保存WriteableBitmap ，我们可以使用截图

```
private async Task<string> ToBase64(Image control)
{
    var bitmap = new RenderTargetBitmap();
    await bitmap.RenderAsync(control);
    return await ToBase64(bitmap);
}


```

如果ImageSource是WriteableBitmap ，直接保存

```

//WriteableBitmap 转 byte[]
private async Task<string> ToBase64(WriteableBitmap bitmap)
{
    var bytes = bitmap.PixelBuffer.ToArray();
    return await ToBase64(bytes, (uint)bitmap.PixelWidth, (uint)bitmap.PixelHeight);
}

private async Task<string> ToBase64(StorageFile bitmap)
{
    var stream = await bitmap.OpenAsync(Windows.Storage.FileAccessMode.Read);
    var decoder = await BitmapDecoder.CreateAsync(stream);
    var pixels = await decoder.GetPixelDataAsync();
    var bytes = pixels.DetachPixelData();
    return await ToBase64(bytes, (uint)decoder.PixelWidth, (uint)decoder.PixelHeight, decoder.DpiX, decoder.DpiY);
}

private async Task<string> ToBase64(RenderTargetBitmap bitmap)
{
    var bytes = (await bitmap.GetPixelsAsync()).ToArray();
    return await ToBase64(bytes, (uint)bitmap.PixelWidth, (uint)bitmap.PixelHeight);
}

private async Task<string> ToBase64(byte[] image, uint height, uint width, double dpiX = 96, double dpiY = 96)
{
    // encode image
    var encoded = new InMemoryRandomAccessStream();
    var encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.PngEncoderId, encoded);
    encoder.SetPixelData(BitmapPixelFormat.Bgra8, BitmapAlphaMode.Straight, height, width, dpiX, dpiY, image);
    await encoder.FlushAsync();
    encoded.Seek(0);

    // read bytes
    var bytes = new byte[encoded.Size];
    await encoded.AsStream().ReadAsync(bytes, 0, bytes.Length);

    // create base64
    return Convert.ToBase64String(bytes);
}

private async Task<ImageSource> FromBase64(string base64)
{
    // read stream
    var bytes = Convert.FromBase64String(base64);
    var image = bytes.AsBuffer().AsStream().AsRandomAccessStream();

    // decode image
    var decoder = await BitmapDecoder.CreateAsync(image);
    image.Seek(0);

    // create bitmap
    var output = new WriteableBitmap((int)decoder.PixelHeight, (int)decoder.PixelWidth);
    await output.SetSourceAsync(image);
    return output;
}
```
https://codepaste.net/ijx28i





## BitmapImage 转 WriteableBitmap

我使用http://www.cnblogs.com/cjw1115/p/5164327.html 大神的，直接转`WriteableBitmap bitmap = imageSource as WriteableBitmap;`bitmap为null，于是我在网上继续找。




如果需要保存网络图片到本地，请到[win10 uwp 存放网络图片到本地](http://lindexi.oschina.io/lindexi/post/win10-uwp-存放网络图片到本地/)

参见：http://www.cnblogs.com/cjw1115/p/5164327.html

http://www.cnblogs.com/yuanforprogram/p/4819307.html