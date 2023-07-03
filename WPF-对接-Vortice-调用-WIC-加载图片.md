
# WPF 对接 Vortice 调用 WIC 加载图片

本文将告诉大家如何通过 Vortice 库从底层的方式使用 WIC 层加载本地图片文件，解码为 IWICBitmap 图片，然后将 IWICBitmap 图片交给 WPF 进行渲染

<!--more-->


<!-- CreateTime:2023/5/15 8:38:17 -->


<!-- 标签：C#,D2D,DirectX,Vortice,Direct2D, -->
<!-- 博客 -->
<!-- 发布 -->

本文的前置博客：[WPF 对接 Vortice 调用 D2D 使用 IWICBitmap 离屏渲染](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E8%B0%83%E7%94%A8-D2D-%E4%BD%BF%E7%94%A8-IWICBitmap-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html ) [博客园](https://www.cnblogs.com/lindexi/p/16774416.html )

先集中精力到如何通过 WIC 层加载本地文件为图片上，本文的所有代码都可以在本文末尾获取

下面介绍比较通用的做法进行加载图片文件。按照惯例，先创建出 IWICImagingFactory 对象，如以下代码

```csharp
  using Vortice.WIC;

      using var wicImagingFactory = new IWICImagingFactory();
```

通过 `using var` 的方式可以减少大量的 Dispose 代码，将自动在方法结束之前调用 Dispose 方法释放资源。值得一提的是作为非常底层基础的 Vortice 框架，在使用 对接时，各个创建出来的对象资源都需要关注一下，基本都需要自己手工释放

通过 CreateStream 方法从文件创建为 IWICStream 对象，如此即可方便后续的图片加载

```csharp
        var imageFilePath = System.IO.Path.GetFullPath("Image.png");
        using var wicStream = wicImagingFactory.CreateStream(imageFilePath, FileAccess.Read);
```

还请将以上代码的 `imageFilePath` 替换为你实际的图片文件

在获取到 `IWICStream` 之后，想要加载图片，就需要用到图片解码器。解码器本身可以通过 IWICImagingFactory 从 IWICStream 里创建，代码如下

```csharp
        using var decoder = wicImagingFactory.CreateDecoderFromStream(wicStream, DecodeOptions.CacheOnLoad/*参数和 WPF 一样*/);
```

创建出来的解码器自动进行解码，如果解码失败，比如图片文件损坏或者对应图片格式找不到解码器等，都会抛出异常炸掉。异常是对应有 WIC 的错误码信息的，如比较常见的图片损坏的如下异常

```
SharpGen.Runtime.SharpGenException:“HRESULT: [0x88982F50], Module: [Vortice.Direct2D1], ApiCode: [WINCODEC_ERR_COMPONENTNOTFOUND/Componentnotfound], Message: [无法找不到组件。
]”
```

错误码信息可以到 [Codec Error Codes - Win32 apps Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/wic/-wic-codec-error-codes) 官方文档了解更多信息

从解码器获取到解码的 IWICBitmap 图片，可以使用 GetFrame 方法。这个方法需要传入参数，因为解码器将可以解码出图片，对于动态图片可以解析出多张图片出来，对于静态图片只能解析出一张。对于静态图片（区别于 gif 等动态图片）只须取首个，也就是常用的就是传入 0 就好

```csharp
        using var imageFrame = decoder.GetFrame(0);
```

拿到 `IWICBitmapFrameDecode` 类型的 imageFrame 对象即可通过 CreateBitmapFromSource 创建 IWICBitmap 图片

```csharp
        IWICBitmap wicBitmap = wicImagingFactory.CreateBitmapFromSource(imageFrame, BitmapCreateCacheOption.CacheOnLoad);
```

如此即可拿到 IWICBitmap 对象，再通过 [上一篇博客](https://blog.lindexi.com/post/WPF-%E5%AF%B9%E6%8E%A5-Vortice-%E8%B0%83%E7%94%A8-D2D-%E4%BD%BF%E7%94%A8-IWICBitmap-%E7%A6%BB%E5%B1%8F%E6%B8%B2%E6%9F%93.html ) 提供的方法即可在 WPF 显示

以下是加载本地图片作为 IWICBitmap 的所有代码

```csharp
    private static IWICBitmap OpenFileAsWICBitmap()
    {
        using var wicImagingFactory = new IWICImagingFactory();
        var imageFilePath = System.IO.Path.GetFullPath("Image.png");
        using var wicStream = wicImagingFactory.CreateStream(imageFilePath, FileAccess.Read);
        using var decoder = wicImagingFactory.CreateDecoderFromStream(wicStream, DecodeOptions.CacheOnLoad/*参数和 WPF 一样*/);
        // 解码器将可以解码出图片，对于动态图片可以解析出多张图片出来，对于静态图片只能解析出一张
        // 对于静态图片（区别于 gif 等动态图片）只须取首个
        using IWICBitmapFrameDecode imageFrame = decoder.GetFrame(0);

        IWICBitmap wicBitmap = wicImagingFactory.CreateBitmapFromSource(imageFrame, BitmapCreateCacheOption.NoCache);

        return wicBitmap;
    }
```

其中创建 IWICStream 这一步可以省略，通过 CreateDecoderFromFileName 即可从文件创建解码器，更新的代码如下

```csharp
    private static IWICBitmap OpenFileAsWICBitmap()
    {
        using var wicImagingFactory = new IWICImagingFactory();
        var imageFilePath = System.IO.Path.GetFullPath("Image.png");
        using var decoder = wicImagingFactory.CreateDecoderFromFileName(imageFilePath);
        // 解码器将可以解码出图片，对于动态图片可以解析出多张图片出来，对于静态图片只能解析出一张
        // 对于静态图片（区别于 gif 等动态图片）只须取首个
        using IWICBitmapFrameDecode imageFrame = decoder.GetFrame(0);

        IWICBitmap wicBitmap = wicImagingFactory.CreateBitmapFromSource(imageFrame, BitmapCreateCacheOption.NoCache);
        return wicBitmap;
    }
```

只是常用的情况下会考虑一些通用性，才使用 IWICStream 对象

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/11ede4eb7cb224fe0f561de16d86874a4b71cec2/WpfVorticeWicTest) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/11ede4eb7cb224fe0f561de16d86874a4b71cec2/WpfVorticeWicTest) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 11ede4eb7cb224fe0f561de16d86874a4b71cec2
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 11ede4eb7cb224fe0f561de16d86874a4b71cec2
```

获取代码之后，进入 WpfVorticeWicTest 文件夹

更多 DirectX 和 D2D 以及 Vortice 库的博客，请参阅我的 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )

另外，我创建了专门聊 Vortice 的 QQ 群： 622808968 欢迎加入交流技术




<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。