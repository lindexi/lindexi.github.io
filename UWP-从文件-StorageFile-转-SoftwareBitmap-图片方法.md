
# UWP 从文件 StorageFile 转 SoftwareBitmap 图片方法

本文告诉大家如何在 UWP 从 文件 StorageFile 转 SoftwareBitmap 图片的方法

<!--more-->


<!-- CreateTime:2022/7/25 8:28:00 -->

<!-- 发布 -->

使用以下三步即可从文件 StorageFile 转 SoftwareBitmap 图片

第一步是读取文件，获取可以随机访问的 IRandomAccessStream 对象。这个对象表示的是一个 Stream 且此 Stream 支持随机访问。随机访问是和顺序访问相对，指的是可以从 Stream 的任意地方开始读写，代码如下

```csharp
            using (IRandomAccessStream stream = await inputFile.OpenAsync(FileAccessMode.Read))
            {
            }
```

第二步是通过 BitmapDecoder 的 CreateAsync 创建出解码器

```csharp
  BitmapDecoder decoder = await BitmapDecoder.CreateAsync(stream);
```

最后是通过解码器获取 SoftwareBitmap 对象

```csharp
var softwareBitmap = await decoder.GetSoftwareBitmapAsync();
```

我封装的代码如下

```csharp
        private static async Task<SoftwareBitmap> StorageFileToSoftwareBitmapAsync(StorageFile inputFile)
        {
            using (IRandomAccessStream stream = await inputFile.OpenAsync(FileAccessMode.Read))
            {
                // Create the decoder from the stream
                BitmapDecoder decoder = await BitmapDecoder.CreateAsync(stream);

                // Get the SoftwareBitmap representation of the file
                var softwareBitmap = await decoder.GetSoftwareBitmapAsync();
                return softwareBitmap;
            }
        }
```

本文代码放在[github](https://github.com/lindexi/lindexi_gd/tree/cca7a541ecffad71371ff89f17108d7d04a9a102/WadeaherkeaLihanececeeneri) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/cca7a541ecffad71371ff89f17108d7d04a9a102/WadeaherkeaLihanececeeneri) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin cca7a541ecffad71371ff89f17108d7d04a9a102
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
```

获取代码之后，进入 WadeaherkeaLihanececeeneri 文件夹




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。