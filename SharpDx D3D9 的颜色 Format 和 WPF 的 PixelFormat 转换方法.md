# SharpDx D3D9 的颜色 Format 和 WPF 的 PixelFormat 转换方法

本文告诉大家在 DirectX 9 的颜色格式 Format 和 WPF 的 PixelFormat 转换方法

<!--more-->
<!-- CreateTime:2021/12/21 20:00:25 -->

<!-- 发布 -->

转换代码如下

```csharp
        private static PixelFormat TranslateFormatToPixelFormat(D3D9.Format format, bool preMultiplied = true)
        {
            return format switch
            {
                D3D9.Format.R8G8B8 => PixelFormats.Bgr24,
                D3D9.Format.A8R8G8B8 => preMultiplied ? PixelFormats.Pbgra32 : PixelFormats.Bgra32,
                D3D9.Format.X8R8G8B8 => PixelFormats.Bgr32,
                //D3D9.Format.R5G6B5 => PixelFormats.Bgr16bpp565,
                //D3D9.Format.X1R5G5B5 => PixelFormats.BGR16bpp555,
                D3D9.Format.P8 => PixelFormats.Indexed8,
                D3D9.Format.L8 => PixelFormats.Gray8,
                D3D9.Format.A2R10G10B10 => PixelFormats.Bgr101010,
                D3D9.Format.A32B32G32R32F => preMultiplied ? PixelFormats.Prgba128Float : PixelFormats.Rgb128Float,
                _ => throw new NotSupportedException(),
            };
        }
```

值得一说的是 SharpDx 当前官方不维护了，可以选择的代替请看 [SharpDx 的代替项目](https://blog.lindexi.com/post/SharpDx-%E7%9A%84%E4%BB%A3%E6%9B%BF%E9%A1%B9%E7%9B%AE.html )

在 WPF 的 PixelFormat 是和 WIC 层关联的，定义的代码如下

```csharp
        static Guid GetGuidFromFormat(PixelFormatEnum format)
        {
            switch (format)
            {
                case PixelFormatEnum.Default:
                    return WICPixelFormatGUIDs.WICPixelFormatDontCare;

                case PixelFormatEnum.Indexed1:
                    return WICPixelFormatGUIDs.WICPixelFormat1bppIndexed;

                case PixelFormatEnum.Indexed2:
                    return WICPixelFormatGUIDs.WICPixelFormat2bppIndexed;

                case PixelFormatEnum.Indexed4:
                    return WICPixelFormatGUIDs.WICPixelFormat4bppIndexed;

                case PixelFormatEnum.Indexed8:
                    return WICPixelFormatGUIDs.WICPixelFormat8bppIndexed;

                case PixelFormatEnum.BlackWhite:
                    return WICPixelFormatGUIDs.WICPixelFormatBlackWhite;

                case PixelFormatEnum.Gray2:
                    return WICPixelFormatGUIDs.WICPixelFormat2bppGray;

                case PixelFormatEnum.Gray4:
                    return WICPixelFormatGUIDs.WICPixelFormat4bppGray;

                case PixelFormatEnum.Gray8:
                    return WICPixelFormatGUIDs.WICPixelFormat8bppGray;

                case PixelFormatEnum.Bgr555:
                    return WICPixelFormatGUIDs.WICPixelFormat16bppBGR555;

                case PixelFormatEnum.Bgr565:
                    return WICPixelFormatGUIDs.WICPixelFormat16bppBGR565;

                case PixelFormatEnum.Bgr24:
                    return WICPixelFormatGUIDs.WICPixelFormat24bppBGR;

                case PixelFormatEnum.Rgb24:
                    return WICPixelFormatGUIDs.WICPixelFormat24bppRGB;

                case PixelFormatEnum.Bgr101010:
                    return WICPixelFormatGUIDs.WICPixelFormat32bppBGR101010;

                case PixelFormatEnum.Bgr32:
                    return WICPixelFormatGUIDs.WICPixelFormat32bppBGR;

                case PixelFormatEnum.Bgra32:
                    return WICPixelFormatGUIDs.WICPixelFormat32bppBGRA;

                case PixelFormatEnum.Pbgra32:
                    return WICPixelFormatGUIDs.WICPixelFormat32bppPBGRA;

                case PixelFormatEnum.Rgb48:
                    return WICPixelFormatGUIDs.WICPixelFormat48bppRGB;

                case PixelFormatEnum.Rgba64:
                    return WICPixelFormatGUIDs.WICPixelFormat64bppRGBA;

                case PixelFormatEnum.Prgba64:
                    return WICPixelFormatGUIDs.WICPixelFormat64bppPRGBA;

                case PixelFormatEnum.Gray16:
                    return WICPixelFormatGUIDs.WICPixelFormat16bppGray;

                case PixelFormatEnum.Gray32Float:
                    return WICPixelFormatGUIDs.WICPixelFormat32bppGrayFloat;

                case PixelFormatEnum.Rgb128Float:
                    return WICPixelFormatGUIDs.WICPixelFormat128bppRGBFloat;

                case PixelFormatEnum.Rgba128Float:
                    return WICPixelFormatGUIDs.WICPixelFormat128bppRGBAFloat;

                case PixelFormatEnum.Prgba128Float:
                    return WICPixelFormatGUIDs.WICPixelFormat128bppPRGBAFloat;

                case PixelFormatEnum.Cmyk32:
                    return WICPixelFormatGUIDs.WICPixelFormat32bppCMYK;
            }

            throw new System.ArgumentException (SR.Get(SRID.Image_BadPixelFormat, format), "format");
        }
```

对应的逻辑如下

```csharp
    internal static class WICPixelFormatGUIDs
    {
        /* Undefined formats */
        internal static readonly Guid WICPixelFormatDontCare = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x00);

        /* Indexed formats */
        internal static readonly Guid WICPixelFormat1bppIndexed = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x01);
        internal static readonly Guid WICPixelFormat2bppIndexed = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x02);
        internal static readonly Guid WICPixelFormat4bppIndexed = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x03);
        internal static readonly Guid WICPixelFormat8bppIndexed = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x04);

        internal static readonly Guid WICPixelFormatBlackWhite = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x05);
        internal static readonly Guid WICPixelFormat2bppGray = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x06);
        internal static readonly Guid WICPixelFormat4bppGray = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x07);
        internal static readonly Guid WICPixelFormat8bppGray = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x08);

        /* sRGB formats (gamma is approx. 2.2) */
        /* For a full definition, see the sRGB spec */

        /* 16bpp formats */
        internal static readonly Guid WICPixelFormat16bppBGR555 = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x09);
        internal static readonly Guid WICPixelFormat16bppBGR565 = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x0a);
        internal static readonly Guid WICPixelFormat16bppGray   = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x0b);

        /* 24bpp formats */
        internal static readonly Guid WICPixelFormat24bppBGR = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x0c);
        internal static readonly Guid WICPixelFormat24bppRGB = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x0d);

        /* 32bpp format */
        internal static readonly Guid WICPixelFormat32bppBGR  = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x0e);
        internal static readonly Guid WICPixelFormat32bppBGRA = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x0f);
        internal static readonly Guid WICPixelFormat32bppPBGRA = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x10);
        internal static readonly Guid WICPixelFormat32bppGrayFloat  = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x11);

        /* scRGB formats. Gamma is 1.0 */
        /* For a full definition, see the scRGB spec */

        /* 32bpp format */
        internal static readonly Guid WICPixelFormat32bppBGR101010 = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x14);

        /* 48bpp format */
        internal static readonly Guid WICPixelFormat48bppRGB = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x15);

        /* 64bpp format */
        internal static readonly Guid WICPixelFormat64bppRGBA = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x16);
        internal static readonly Guid WICPixelFormat64bppPRGBA = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x17);

         /* Floating point scRGB formats */
        internal static readonly Guid WICPixelFormat128bppRGBAFloat = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x19);
        internal static readonly Guid WICPixelFormat128bppPRGBAFloat = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x1a);
        internal static readonly Guid WICPixelFormat128bppRGBFloat = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x1b);

         /* CMYK formats. */
        internal static readonly Guid WICPixelFormat32bppCMYK = new Guid(0x6fddc324, 0x4e03, 0x4bfe, 0xb1, 0x85, 0x3d, 0x77, 0x76, 0x8d, 0xc9, 0x1c);
    }
```

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。
