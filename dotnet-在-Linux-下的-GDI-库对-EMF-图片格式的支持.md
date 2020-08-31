
# dotnet 在 Linux 下的 GDI 库对 EMF 图片格式的支持

我想要在 UOS 上支持上古的图片格式，也就是差不多废弃了 20 年的 EMF 和 WMF 增强图形格式，这两个格式十分古老，而在 Windows 下也存在一些不兼容的图片。我在 Windows 下是使用 GDI+ 的方法支持的，可以将 EMF 转 PNG 或 jpg 等格式。而在 UOS 下，因为 GDI+ 是跨平台的，可以使用跨平台的 System.Drawing.Common 库进行转换

<!--more-->


<!-- CreateTime:2020/8/29 8:37:06 -->

<!-- 发布 -->

在哪里可以找到很多 EMF 或 WMF 格式的图片？去 PPT 里面的剪辑版找，安装 Office 2013 的版本，可以在 `Program Files\Microsoft Office\CLIPART\PUB60COR\` 找到一些图片

如将 EMF 或 WMF 转 png 格式图片的代码，在 C# 中可以这样写

```csharp
        public static void ConvertEnhancedMetaFileImage(FileInfo originFile, FileInfo generatedFile, int requestedPixel)
        {
            using var image = Image.FromFile(originFile.FullName);

            var size = GetImageOptimizationSize(new Size(image.Width, image.Height), MaxWidth * MaxHeight,
                requestedPixel);

            var width = size.Width;
            var height = size.Height;

            var resized = new Bitmap(width, height);
            using (var graphics = Graphics.FromImage(resized))
            {
                graphics.CompositingQuality = CompositingQuality.HighSpeed;
                graphics.InterpolationMode = InterpolationMode.HighQualityBicubic;
                graphics.CompositingMode = CompositingMode.SourceCopy;
                graphics.DrawImage(image, 0, 0, width, height);
                resized.Save(generatedFile.FullName, ImageFormat.Png);
            }
        }
```

上面代码的 GetImageOptimizationSize 就是 [dotnet C# 图片等比限制最大和最小大小缩放算法](https://blog.lindexi.com/post/dotnet-C-%E5%9B%BE%E7%89%87%E7%AD%89%E6%AF%94%E9%99%90%E5%88%B6%E6%9C%80%E5%A4%A7%E5%92%8C%E6%9C%80%E5%B0%8F%E5%A4%A7%E5%B0%8F%E7%BC%A9%E6%94%BE%E7%AE%97%E6%B3%95.html) 所使用的方法

就是通过这么简单的逻辑就能实现，上面代码能在 Linux 和 Windows 使用

在使用之前需要使用 NuGet 安装 System.Drawing.Common 库，如果是 SDK 的 csproj 可以添加下面代码安装

```xml
    <ItemGroup>
        <PackageReference Include="System.Drawing.Common" Version="4.7.0" />
    </ItemGroup>
```

而除了 System.Drawing.Common 库之外，其他的库的支持也是很差，如非常有名的 SixLabors.ImageSharp 等

- SixLabors.ImageSharp 在 Windows 下支持，在 Linux 不支持
- Magick.NET 在 Windows 下支持，在 Linux 不支持，详细请看 [Can EMF format pictures be supported on linux · Issue #585 · dlemstra/Magick.NET](https://github.com/dlemstra/Magick.NET/issues/585 )

而在 System.Drawing.Common 库的支持也很弱，大概只有2成左右的命令支持。在 EMF 等格式里面，其实 EMF 可以细分为多个不同的格式，如 EMF 和 EMF+ 等，这个格式核心是通过记录 GDI 和 GDI+ 绘制命令实现图片绘制。因此解析这个图片格式的前提是需要实现超级庞大的 GDI 绘图，这是特别有工作量的

因此在转换的时候，也许你会看到控制台或 VS 输出窗口有这样的输出内容

```
** (process:1209): WARNING **: 17:03:45.698: SelectObject 2, no created object, slot empty.
```

上面输出的 `no created object, slot empty` 就是核心，在 System.Drawing.Common 库的核心是调用 LibGdiPlus 库，这是放在 mono 组织下的一个库，可以大概认为是有微软官方在维护的库

在 LibGdiPlus 库的核心代码里面，可以在 [https://github.com/mono/libgdiplus](https://github.com/mono/libgdiplus) 找到 metafile.c 文件，这是一个用 c 写的库，可以看到有如下代码

```csharp
  switch (context->created.type) 
  {
  case METAOBJECT_TYPE_EMPTY:
    /* if nothing is "created" (and not yet selected into a slot) then we "reselect" the object */
    switch (context->objects [slot].type) 
    {
    case METAOBJECT_TYPE_EMPTY:
      g_warning ("SelectObject %d, no created object, slot empty.", slot);
      break;
    case METAOBJECT_TYPE_PEN:
      context->selected_pen = slot;
      break;
    case METAOBJECT_TYPE_BRUSH:
      context->selected_brush = slot;
      break;
    }
  }
```

这就是上面输出的内容了，也就是这份图片存在不能解析的内容

另外还有其他纯 C 或 C++ 的库，现在是 2020 年，我还没有找到一个支持比较好的库

- [APerricone emf2pdf](https://github.com/APerricone/emf2pdf ) 纯 Windows 下的库
- [wholegroup vector](https://github.com/wholegroup/vector ) 纯 Windows 下的库
- [libemf ECMA-234 Metafile Library ](https://sourceforge.net/projects/libemf/ ) 用来制作 EMF 文件的库
- [pzinovkin emftoimg](https://github.com/pzinovkin/emftoimg ) 支持不到 1 成
- [kakwa libemf2svg](https://github.com/kakwa/libemf2svg ) 转 svg 格式，完成 3 成，对 EMF+ 支持很弱

一些 Linux 上完成度很高的软件

- [LibreOffice - Free Office Suite - Based on OpenOffice - Compatible with Microsoft](https://www.libreoffice.org/ ) 支持 Linux 系统，解析完成度很高，但是软件 100 M 哈。这也是大部分小伙伴给出在 Linux 下命令行转换最好的方法
- [Inkscape](https://inkscape.org/news/2020/05/04/introducing-inkscape-10/ ) 这是一个在 Linux 下能支持的 EMF 格式的软件，一样有 100M 大小，有小伙伴说 Inkscape 解析不如 LibreOffice 但是我测试了 300 份图片，发现差不多

一些反向转换项目

- [LonelyPale Svg2EmfServer](https://github.com/LonelyPale/Svg2EmfServer ) 把 svg 转换成 emf 格式，这是一个 ASP.NET Core 项目





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。