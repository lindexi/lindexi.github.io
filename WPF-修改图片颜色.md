
# WPF 修改图片颜色

本文告诉大家如何修改图片的颜色，如去掉图片的蓝色
<!-- 标签：WPF，图片处理 -->

<!--more-->


<!-- csdn -->

在 WPF 可以使用很多图片处理的方法，本文告诉大家的是一个图片处理，可以把处理的图片保存在文件。

在阅读本文，我假设大家是熟悉 WPF 的，至少了解 C# ，也知道图片的格式。

在 WPF 可以使用 ARBG 数组表示图片，本文修改图片颜色的方法就是使用 ARBG 数组的方法修改，修改里面的元素的值。

如我需要去掉图片的蓝色，就可以通过修改 ARBG 数组的元素，设置所有蓝色为 0 ，去掉蓝色。

## 读取图片

首先找到一张好看的图片，放在解决方案

<!-- ![](image/WPF 修改图片颜色/WPF 修改图片颜色0.png) -->

![](http://7xqpl8.com1.z0.glb.clouddn.com/lindexi%2F2018732045136233.jpg)

读取解决方案的图片

```csharp
            var stream = Application.GetResourceStream(new Uri("pack://application:,,,/1.jpg")).Stream;
```

如果找不到图片，就是没有设置图片生成是 Resource 

## 解析文件

创建 WriteableBitmap 需要使用 ImageSource 所以需要先解析

```csharp
// 其他忽略代码
            var bitmapImage = new BitmapImage();
            bitmapImage.BeginInit();
            bitmapImage.StreamSource = stream;
            bitmapImage.EndInit();
```

使用 BitmapImage 解析文件

## 创建图片

在读取图片之后就可以创建图片

```csharp
            var writeableBitmap = new WriteableBitmap(bitmapImage);

```

## 转换图片格式

如果读取到的图片不是 BGRA 的格式，就需要转换图片格式

```csharp
            var formatConvertedBitmap = new FormatConvertedBitmap();

            formatConvertedBitmap.BeginInit();

            formatConvertedBitmap.Source = bitmapImage;

            formatConvertedBitmap.DestinationFormat = PixelFormats.Bgra32;

            formatConvertedBitmap.EndInit();
```

使用这个代码可以把格式转为`PixelFormats.Bgra32`，需要重新创建图片

```csharp
            var stream = Application.GetResourceStream(new Uri("pack://application:,,,/1.jpg")).Stream;

            var bitmapImage = new BitmapImage();
            bitmapImage.BeginInit();
            bitmapImage.StreamSource = stream;
            bitmapImage.EndInit();

            var formatConvertedBitmap = new FormatConvertedBitmap();

            formatConvertedBitmap.BeginInit();

            formatConvertedBitmap.Source = bitmapImage;

            formatConvertedBitmap.DestinationFormat = PixelFormats.Bgra32;

            formatConvertedBitmap.EndInit();

            var writeableBitmap = new WriteableBitmap(formatConvertedBitmap);
```

尝试显示图片，可以看到图片还是很好看

<!-- ![](image/WPF 修改图片颜色/WPF 修改图片颜色1.png) -->

![](http://7xqpl8.com1.z0.glb.clouddn.com/lindexi%2F20187320571094.jpg)

## 读取数组

在图片可以看到图片是使用 BGRA 的格式数组，所以只需要读取图片数组就可以修改图片

读取图片需要使用不安全代码，需要右击项目属性，点击生成，允许不安全代码。

在修改图片之前需要使用 Lock 函数，读取图片的数组长度可以使用这个代码

```csharp
            var length = writeableBitmap.PixelWidth * writeableBitmap.PixelHeight *
                         writeableBitmap.Format.BitsPerPixel / 8;
```

这里知道使用的是 BGRA 也就是一个像素使用 4 个 byte ，一个图片的像素就是`writeableBitmap.PixelWidth * writeableBitmap.PixelHeight` 。这里 `writeableBitmap.Format.BitsPerPixel` 就是拿到一个像素的 bit 数。

转换数组

```csharp
            var backBuffer = (byte*) writeableBitmap.BackBuffer;

```

读取颜色就是从数组拿到值

```csharp
            for (int i = 0; i + 4 < length; i = i + 4)
            {
                var blue = backBuffer[i];
                var green = backBuffer[i + 1];
                var red = backBuffer[i + 2];
                var alpha = backBuffer[i + 3];
            }
```

修改颜色就是修改对应的值然后设置数组，如设置蓝色是 0 去掉蓝色

```csharp
            for (int i = 0; i + 4 < length; i = i + 4)
            {
                var blue = backBuffer[i];
                var green = backBuffer[i + 1];
                var red = backBuffer[i + 2];
                var alpha = backBuffer[i + 3];

                blue = 0;

                backBuffer[i] = blue;
                backBuffer[i + 1] = green;
                backBuffer[i + 2] = red;
                backBuffer[i + 3] = alpha;
            }
```

设置之后需要设置图片显示

```csharp

            writeableBitmap.AddDirtyRect(new Int32Rect(0, 0, writeableBitmap.PixelWidth, writeableBitmap.PixelHeight));
            writeableBitmap.Unlock();
```

所以去掉图片的蓝色可以使用 RemoveBlue 函数，设置蓝色为 0 的方法就是读取蓝色然后修改数组

```csharp
        private unsafe void RemoveBlue(WriteableBitmap writeableBitmap)
        {
            writeableBitmap.Lock();

            var length = writeableBitmap.PixelWidth * writeableBitmap.PixelHeight *
                         writeableBitmap.Format.BitsPerPixel / 8;

            var backBuffer = (byte*) writeableBitmap.BackBuffer;

            for (int i = 0; i + 4 < length; i = i + 4)
            {
                var blue = backBuffer[i];
                var green = backBuffer[i + 1];
                var red = backBuffer[i + 2];
                var alpha = backBuffer[i + 3];

                blue = 0;

                backBuffer[i] = blue;
                backBuffer[i + 1] = green;
                backBuffer[i + 2] = red;
                backBuffer[i + 3] = alpha;
            }

            writeableBitmap.AddDirtyRect(new Int32Rect(0, 0, writeableBitmap.PixelWidth, writeableBitmap.PixelHeight));
            writeableBitmap.Unlock();
        }

```

去掉蓝色的图片

<!-- ![](image/WPF 修改图片颜色/WPF 修改图片颜色2.png) -->

![](http://7xqpl8.com1.z0.glb.clouddn.com/lindexi%2F20187321933857.jpg)

代码：[WPF 修改图片颜色 1.2-CSDN下载](https://download.csdn.net/download/lindexi_gd/10517437 )



参见：

[How to: Convert a BitmapSource to a Different PixelFormat](https://docs.microsoft.com/en-us/dotnet/framework/wpf/graphics-multimedia/how-to-convert-a-bitmapsource-to-a-different-pixelformat )





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。