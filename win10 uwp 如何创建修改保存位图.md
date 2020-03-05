# win10 uwp 如何创建修改保存位图

本文告诉大家如何使用 Softwarebitmap 进行创建、修改保存图片。

<!--more-->
<!-- CreateTime:2018/8/10 19:16:50 -->

<!-- csdn -->

<!-- 标签：win10,uwp -->
<div id="toc"></div>


在 UWP 使用底层的图像渲染就是使用 Softwarebitmap ，这个类提供直接数据修改，可以使用这个类进行软渲染。实际上 Softwarebitmap 和  WriteableBitmap 是差不多的。但是 Softwarebitmap 可以支持 WriteableBitmap 、 Direct3D 和代码修改。通过 Softwarebitmap 可以修改转换不同的像素格式和透明通道，支持低级修改像素。作为一个通用的底层类在很多性能要求比较高的地方用到，如 CapturedFrame、VideoFrame、FaceDetector。下面来告诉大家如何使用。

## 创建 

下面来告诉大家如何读取文件，使用图片数据创建 Softwarebitmap 图片。

首先是需要使用 FileOpenPicker 拿到一张图片，如何读写文件参见：[win10 UWP读写文件](https://blog.csdn.net/lindexi_gd/article/details/49007841 )

因为很简单，下面直接拿到一张 jpg ，当然需要用户点击。下面代码是直接从微软文档复制的，我自己没运行，看起来大家可以直接使用。

```csharp
FileOpenPicker fileOpenPicker = new FileOpenPicker();
fileOpenPicker.SuggestedStartLocation = PickerLocationId.PicturesLibrary;
fileOpenPicker.FileTypeFilter.Add(".jpg");
fileOpenPicker.ViewMode = PickerViewMode.Thumbnail;

var inputFile = await fileOpenPicker.PickSingleFileAsync();

if (inputFile == null)
{
    // The user cancelled the picking operation
    return;
}
```

因为需要拿到文件内容，需要使用 OpenAsync 方法获得随机访问流。随机访问流就是可以在随机的地方进行读写，和他不相同的是顺序流，也就是只能顺序读写。使用  BitmapDecoder.CreateAsync 创建一个图片解析，用来拿到图片

```csharp
SoftwareBitmap softwareBitmap;

using (IRandomAccessStream stream = await inputFile.OpenAsync(FileAccessMode.Read))
{
    // Create the decoder from the stream
    BitmapDecoder decoder = await BitmapDecoder.CreateAsync(stream);

    // Get the SoftwareBitmap representation of the file
    softwareBitmap = await decoder.GetSoftwareBitmapAsync();
}
```

## 保存图片

上面和大家说如何读取文件，现在就可以把刚才读取的图片保存。保存需要用户选择保存在哪

```csharp
          FileSavePicker fileSavePicker = new FileSavePicker();
            fileSavePicker.SuggestedStartLocation = PickerLocationId.PicturesLibrary;
            fileSavePicker.FileTypeChoices.Add("png files", new List<string>() { ".png" });
            fileSavePicker.SuggestedFileName = "image";

            var outputFile = await fileSavePicker.PickSaveFileAsync();

            if (outputFile == null)
            {
                // The user cancelled the picking operation
                return;
            }
```

使用  OpenAsync 方法打开文件，转换随机写入流写入数据。使用  BitmapEncoder.CreateAsync 创建 BitmapEncoder 。创建的函数第一个参数是 GUID 表示需要哪个格式，可以通过 BitmapEncoder 输入，下面代码就是把刚才读取的  jpg 图片转换为 Png 格式。

```csharp
           using (var stream = await outputFile.OpenAsync(FileAccessMode.ReadWrite))
            {
                // 格式 png 通过把上面打开的图片转换
                BitmapEncoder encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.PngEncoderId, stream);

                encoder.SetSoftwareBitmap(_softwareBitmap);
            }
```

这个方法就是把 softwareBitmap 转换为 Stream 的方法

如果在保存需要对图片进行编辑，可以使用 BitmapTransform 进行变换，请看代码

```csharp
                encoder.BitmapTransform.ScaledWidth = 320;
                encoder.BitmapTransform.ScaledHeight = 240;
                // 90度旋转
                encoder.BitmapTransform.Rotation = BitmapRotation.Clockwise90Degrees;
                // 大的图片转换为小的图片，需要使用插值算法，不然会模糊
                encoder.BitmapTransform.InterpolationMode = BitmapInterpolationMode.Fant;
                // 创建新的缩略图
                encoder.IsThumbnailGenerated = true;
```

因为不是所有的文件格式都支持缩略图，如果使用了创建新的图就需要 catch 不支持异常。

在转换图片需要调用  FlushAsync 保存图片。

```csharp
                try
                {
                    await encoder.FlushAsync();
                }
                catch (Exception exception)
                {
                    const int WINCODEC_ERR_UNSUPPORTEDOPERATION = unchecked((int) 0x88982F81);

                    switch (exception.HResult)
                    {
                        case WINCODEC_ERR_UNSUPPORTEDOPERATION:
                            // 如果格式不支持，就会出现这个异常，需要禁止创建缩略图，然后继续保存
                            encoder.IsThumbnailGenerated = false;
                            break;
                        default:
                            throw;
                    }
                }

                if (encoder.IsThumbnailGenerated == false)
                {
                    await encoder.FlushAsync();
                }
```

现在在前台添加两个按钮，一个用于打开文件，另一个用来保存图片

![](http://image.acmx.xyz/lindexi%2F201856104247321.jpg)

随便选一个 jpg 文件，然后保存，可以看到保存了新的格式

![](http://image.acmx.xyz/lindexi%2F2018561044427178.jpg)

在 UWP 可以使用上面的方法修改图片格式

上面代码只是简单使用，在创建 BitmapEncoder 可以传入 BitmapPropertySet 指定图片质量

```csharp
      var propertySet = new Windows.Graphics.Imaging.BitmapPropertySet();
                var qualityValue = new Windows.Graphics.Imaging.BitmapTypedValue(
                    1.0, // Maximum quality
                    Windows.Foundation.PropertyType.Single
                );

                propertySet.Add("ImageQuality", qualityValue);

                // 格式 png 通过把上面打开的图片转换
                BitmapEncoder encoder = await BitmapEncoder.CreateAsync(BitmapEncoder.PngEncoderId, stream, propertySet);
```

其他的值请看 [BitmapEncoder options reference ](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/bitmapencoder-options-reference )

## 在 Image 控件使用

刚才的代码没有显示打开的图片，如果要把 SoftwareBitmap 在 Image 使用，就需要使用 SoftwareBitmapSource 转换，因为 Image 控件只支持 BGRA8 格式而且需要先计算透明值，在转换打开 SoftwareBitmap 静态函数 Convert 让格式在 Image 控件支持。

先在界面创建一个 Image 控件，然后在后台添加代码显示

```csharp
      <Image x:Name="MaixallnayMesejas"></Image>
```

```csharp
            if (_softwareBitmap.BitmapPixelFormat != BitmapPixelFormat.Bgra8 ||
                _softwareBitmap.BitmapAlphaMode == BitmapAlphaMode.Straight)
            {
                _softwareBitmap = SoftwareBitmap.Convert(_softwareBitmap, BitmapPixelFormat.Bgra8,
                    BitmapAlphaMode.Premultiplied);
            }


            var source = new SoftwareBitmapSource();
            await source.SetBitmapAsync(_softwareBitmap);
            MaixallnayMesejas.Source = source;
```

尝试运行一下代码就可以看到显示图片在打开文件。

实际上通过 下面代码可以把 SoftwareBitmap 转 ImageBrush 显示

```csharp
            var imageBrush = new ImageBrush {ImageSource = source};

```

## WriteableBitmap 转换

上面都是读写文件，如果已经使用了 WriteableBitmap 需要把他转换 SoftwareBitmap 可以使用 SoftwareBitmap 的静态函数 SoftwareBitmap.CreateCopyFromBuffer 转换。

```csharp
SoftwareBitmap outputBitmap = SoftwareBitmap.CreateCopyFromBuffer(
    writeableBitmap.PixelBuffer,
    BitmapPixelFormat.Bgra8,
    writeableBitmap.PixelWidth,
    writeableBitmap.PixelHeight
);
```

## 通过读写像素

是不是看到上面的教程感觉这个博客很简单，我就来告诉大家很黑的方法，如果看到这里还没有关闭这个网页，那么现在关闭还是可以，不然我就来和大家说很黑科技的写法。

如果大家直接从 SoftwareBitmap 使用 Resharper 无论怎么点都无法找到读写像素的方法。但是我会告诉大家我自己创建了一个接口，使用这个接口就可以读写。

首先引用`using System.Runtime.InteropServices;`然后创建接口

```csharp
    [ComImport]
    [Guid("5B0D3235-4DBA-4D44-865E-8F1D0E4FD04D")]
    [InterfaceType(ComInterfaceType.InterfaceIsIUnknown)]
    internal unsafe interface IMemoryBufferByteAccess
    {
        void GetBuffer(out byte* buffer, out uint capacity);
    }

```

创建这个接口有什么用，先不告诉大家，因为用了不安全，需要在项目属性，生成，可以使用不安全

![](http://image.acmx.xyz/lindexi%2F201856113191940.jpg)

我来告诉大家如何从代码创建 SoftwareBitmap ，读写像素。

创建一个空白的 SoftwareBitmap 需要设置格式

```csharp
            var softwareBitmap = new SoftwareBitmap(BitmapPixelFormat.Bgra8, 100, 100, BitmapAlphaMode.Straight);
```

使用 LockBuffer 可以拿到 buffer ，使用 buffer.CreateReference 可以拿到指针

```csharp

            using (var buffer = softwareBitmap.LockBuffer(BitmapBufferAccessMode.ReadWrite))
            {
                using (var reference = buffer.CreateReference())
                {
                }
            }
```

然后就是不安全代码，本文的黑科技就是这个代码

```csharp
       using (var buffer = softwareBitmap.LockBuffer(BitmapBufferAccessMode.ReadWrite))
            {
                using (var reference = buffer.CreateReference())
                {
                    unsafe
                    {
                        ((IMemoryBufferByteAccess) reference).GetBuffer(out var dataInBytes, out _);               
                    }
                }
```

把 reference 转换为我自己定义的接口，使用 GetBuffer 拿到数据指针。

这个的原理，本渣在这里不会说。

拿到了 dataInBytes 就是按照 BGRA 的顺序，但是还不知道图片的宽度用了多少个，而且图片如果是分层的，第 n 层是从哪个数据开始。为了知道指针的开始，就使用 BitmapBuffer 的方法

```csharp
  BitmapPlaneDescription bufferLayout = buffer.GetPlaneDescription(0);
```

获取图层数量可以使用`buffer.GetPlaneCount()`，因为第 0 个在这里是有的，所以直接使用

那么图片的宽使用多少个如何拿到，bufferLayout.StartIndex 就是拿到图层开始所在，bufferLayout.Stride 就是一行使用了多少 byte 。所以要访问第 i 行 j 列的像素就可以使用下面的代码

```csharp
dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 0] // B

dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 1] // G

ataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 2] // R

dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 3] // A
```

写入的方式就是直接给一个值，读取的方式就是去拿，方法很简单，下面来写一个渐变

```csharp
      ((IMemoryBufferByteAccess)reference).GetBuffer(out dataInBytes, out capacity);

        // Fill-in the BGRA plane
        BitmapPlaneDescription bufferLayout = buffer.GetPlaneDescription(0);
        for (int i = 0; i < bufferLayout.Height; i++)
        {
            for (int j = 0; j < bufferLayout.Width; j++)
            {

                byte value = (byte)((float)j / bufferLayout.Width * 255);
                dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 0] = value;
                dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 1] = value;
                dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 2] = value;
                dataInBytes[bufferLayout.StartIndex + bufferLayout.Stride * i + 4 * j + 3] = (byte)255;
            }
        }
```

## 转换 CanvasBitmap

使用 `CanvasBitmap.CreateFromSoftwareBitmap ` 可以从 SoftwareBitmap 转换为 CanvasBitmap 

```csharp
var canvasBitmap = CanvasBitmap.CreateFromSoftwareBitmap(device, softwareBitmap);
```

需要注意，如果 SoftwareBitmap 的像素格式比较诡异，那么不一定能创建

<table><tr><th>sourceBitmap's BitmapPixelFormat</th><th>CanvasBitmap's Format</th></tr><tr><td>BitmapPixelFormat.Unknown</td><td>unsupported</td></tr><tr><td>BitmapPixelFormat.Rgba16</td><td>DirectXPixelFormat.R16G16B16A16UIntNormalized</td></tr><tr><td>BitmapPixelFormat.Rgba8</td><td>DirectXPixelFormat.R8G8B8A8UIntNormalized</td></tr><tr><td>BitmapPixelFormat.Gray16</td><td>unsupported</td></tr><tr><td>BitmapPixelFormat.Gray8</td><td>DirectXPixelFormat.A8UIntNormalized</td></tr><tr><td>BitmapPixelFormat.Bgra8</td><td>DirectXPixelFormat.B8G8R8A8UIntNormalized</td></tr><tr><td>BitmapPixelFormat.Nv12</td><td>unsupported</td></tr><tr><td>BitmapPixelFormat.Yuy2</td><td>unsupported</td></tr></table>

参见：[CanvasBitmap.CreateFromSoftwareBitmap Method
](https://microsoft.github.io/Win2D/html/M_Microsoft_Graphics_Canvas_CanvasBitmap_CreateFromSoftwareBitmap.htm
)



[Create, edit, and save bitmap images - UWP app developer ](https://docs.microsoft.com/en-us/windows/uwp/audio-video-camera/imaging )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
