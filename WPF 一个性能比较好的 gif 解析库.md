# WPF 一个性能比较好的 gif 解析库

本文介绍 Magick.NET ，这是 ImageMagick 的 .Net 封装，他支持 100 多种格式的图片，而 gif 也是他支持的。本文告诉大家如何使用这个库播放 gif 。

<!--more-->
<!-- CreateTime:2019/8/30 8:59:45 -->

<!-- 标签：WPF，gif -->
<div id="toc"></div>

先给大家看一下播放下面这个图片需要的内存。

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017%25E5%25B9%25B49%25E6%259C%25881%25E6%2597%25A5%252016.gif)

![](http://image.acmx.xyz/34fdad35-5dfe-a75b-2b4b-8c5e313038e2%2F2017102319153.jpg)

这个库的好处是可以在解析的时候使用比较多的内存，解析完成就不需要那么多的内存。而其他的库解析或播放都需要很多内存。从上面的图看到，解析图片需要使用800M内存，解析完成需要200M内存，有很多资源都被释放。

下面告诉大家如何使用这个库。

使用 Nuget 搜索 Magick.NET 就可以找到。如果看到了很多版本，不要害怕。这个库需要说明是哪个版本，其中Qn就是表示质量，推荐使用Q8，而这个库需要指定cpu，于是就可以找到需要的库。

使用Nuget安装之后就可以使用，例如需要播放 `E:\temp\2017年9月1日 16.gif ` 那么需要写一个Image控件和在后台写很少代码。

```csharp
        <Image x:Name="G"></Image>

```

后台需要使用`collection = new MagickImageCollection(str)`获得文件，然后使用`ToBitmapSource`获得图片。使用之前需要给缓存文件夹`MagickAnyCPU.CacheDirectory = "E:\\temp";`

```csharp
          string str = "E:\\temp\\2017年9月1日 16.gif";

            var image = new List<BitmapSource>();
            using (collection = new MagickImageCollection(str))
            {
                collection.Coalesce();
                foreach (var magickImage in collection)
                {
                    image.Add(magickImage.ToBitmapSource());
                }
            }
```

接下来就是播放图片，使用一个循环播放

```csharp
           Task.Run(async () =>
            {
                while (true)
                {
                    await Dispatcher.InvokeAsync(() =>
                    {
                        G.Source = image[n];
                    });
                    n++;
                    if (n == image.Count)
                    {
                        n = 0;
                    }

                    await Task.Delay(100);
                }
            });
```

运行就可以看到，播放图片。就是上面的截图。但是程序有小问题，就是没有拿到图片播放间隔，这个可以通过 `magickImage.AnimationDelay`拿到，这个数是 1/100 秒，所以一般使用 `*10` 结果是毫秒 。

一般在使用`ToBitmapSource`需要使用`AdaptiveResize`这个方法可以让gif显示播放的图片大小，使用这个函数可以获得比较少的内存。

于是播放 gif 的代码很简单，打开 gif ，解析，播放。

```csharp
          string str = "E:\\temp\\2017年9月1日 16.gif";

            var image = new List<(BitmapSource image,int delay)>();
            using (collection = new MagickImageCollection(str))
            {
                collection.Coalesce();
                foreach (var magickImage in collection)
                {
                    magickImage.AdaptiveResize(100,100);
                    image.Add((magickImage.ToBitmapSource(), magickImage.AnimationDelay * 10));
                }
            }
            Task.Run(async () =>
            {
                while (true)
                {
                    await Dispatcher.InvokeAsync(() =>
                    {
                        G.Source = image[n].image;
                    });
                    n++;
                    if (n == image.Count)
                    {
                        n = 0;
                    }

                    await Task.Delay(image[n].delay);
                }
            });
```

我把最近写的 gif 使用方法写到一个博客，欢迎大家来看这个博客 [WPF 播放 gif](https://lindexi.github.io/lindexi/post/WPF-%E6%92%AD%E6%94%BE-gif.html )

参见：[水印第三版 ~ 变态水印（这次用Magick.NET来实现，附需求分析和源码） - 毒逆天 - 博客园](http://www.cnblogs.com/dunitian/p/5895133.html#undefined )

https://github.com/dlemstra/Magick.NET

其他解析gif 的方法：[【续】WPF支持GIF的各种方法 - CSDN博客](http://blog.csdn.net/gqqnb/article/details/7215408 )

[WPF 如何显示gif - CSDN博客](http://blog.csdn.net/wcc27857285/article/details/52993099 )

[WPF播放GIF控件完整代码 - CSDN博客](http://blog.csdn.net/libby1984/article/details/52535085 )

[WPF中显示GIF图片 - CSDN博客](http://blog.csdn.net/qq_31971935/article/details/49402129 )

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 