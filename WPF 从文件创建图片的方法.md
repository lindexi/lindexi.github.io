# WPF 从文件创建图片的方法

本文告诉大家通过 FileStream 创建文件的方法

<!--more-->
<!-- CreateTime:2018/12/27 11:37:46 -->

<!-- csdn -->

如果直接通过文件的 URL 创建，那么可能出现文件被占用的问题，不能比较好做文件的修改，建议通过内存的方式加载

下面是通过内存加载的代码

```csharp
            var bitmapImage = new BitmapImage();
            using (var fileStream = new FileStream("文件路径", FileMode.Open))
            {
                var memoryStream = new MemoryStream();

                fileStream.CopyTo(memoryStream);

                memoryStream.Seek(0, SeekOrigin.Begin);

                bitmapImage.BeginInit();
                bitmapImage.StreamSource = memoryStream;
                bitmapImage.EndInit();
            }

```

通过这个方法加载的图片没有做内存的优化，也就是图片多大，占用的内存就多大

这里存在两个坑，第一个是 memoryStream 在复制之后需要移动到前面，如果没有设置，就会出现下面的代码

```csharp
FileFormatException: 无法对此图像进行解码。该图像头可能已损坏。
```

通过设置 `memoryStream.Seek(0, SeekOrigin.Begin)` 可以解决这个问题，原因是这个流在复制的时候会将指针放在流的最后，但是图片的解析需要将流指针放在最前这样才可以解析

那么此时的 memoryStream 是否可以释放？

如果调用了 `memoryStream.Dispose` 就会显示空白而不是图片

```csharp
            var bitmapImage = new BitmapImage();
            using (var fileStream = new FileStream("E:\\文档\\图片\\2018102016485273.jpg", FileMode.Open))
            {
                var memoryStream = new MemoryStream();

                fileStream.CopyTo(memoryStream);

                memoryStream.Seek(0, SeekOrigin.Begin);

                bitmapImage.BeginInit();
                bitmapImage.StreamSource = memoryStream;
                bitmapImage.EndInit();

                // 下面的代码会让图片显示空
                //memoryStream.Dispose();
            }
```

因为图片需要读取内容，但是内容已经是空的，就没有显示

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
