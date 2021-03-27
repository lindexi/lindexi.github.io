
# dotnet 在 Windows 系统上使用 stakx 的 WIC 库

在 Windows 系统上，有一个很重要的概念是 Windows Imaging Component 也就是 WIC 层，这是专门用来处理多媒体相关的系统组件，特别是用来处理图片相关，包括编码和解码和处理图片。默认在 WPF 中就可以使用封装好的 WIC 层，也就是说最好的 WIC 库就是 WPF 框架了。但是如果在 WPF 之外呢，我有一点特别的需求，我想要绕过 WPF 框架，通过纯控制台的方式使用到 WIC 层的逻辑，此时可以使用 stakx 的 WIC 库。当然，最后发现最好的封装依然 WPF 框架，即使是控制台也能使用 WPF 哦

<!--more-->


<!-- 发布 -->

因为我是在寻找 WIC 层的各个方法，本文只是用来记录一个可以使用的库，但最终发现 WPF 才是最好的封装。只要在 Windows 下，无论用不用 WPF 窗口，都能使用 WPF 提供的 WIC 层封装，因为咱如果只用到 WIC 层，那么相当于只是使用 WPF 库封装的方法

在 WPF 中封装的底层原理请看 [dotnet 读 WPF 源代码笔记 WIC 多媒体图片处理通过 WindowsCodecs.dll 实现功能](https://blog.lindexi.com/post/dotnet-%E8%AF%BB-WPF-%E6%BA%90%E4%BB%A3%E7%A0%81%E7%AC%94%E8%AE%B0-WIC-%E5%A4%9A%E5%AA%92%E4%BD%93%E5%9B%BE%E7%89%87%E5%A4%84%E7%90%86%E9%80%9A%E8%BF%87-WindowsCodecs.dll-%E5%AE%9E%E7%8E%B0%E5%8A%9F%E8%83%BD.html )

如果是 .NET Framework 那么引用 WPF 将不会添加任何额外的多余的输出。如果是 .NET Core 下，除非是独立发布，否则也不会有额外的多余的输出。如果是框架依赖发布，那么会添加的额外输出文件也特别少。因此用 WPF 框架没有啥不足的。和 WPF 框架对比，使用 stakx 的 WIC 库没啥优势

回到本文的 stakx 的 WIC 库的使用方法上，在开始之前需要通过 NuGet 安装 stakx.WIC 库，这是在 GitHub 上完全开源的，请看 [https://github.com/stakx/WIC](https://github.com/stakx/WIC) 

或者在 csproj 上添加如下代码

```xml
  <ItemGroup>
    <PackageReference Include="stakx.WIC" Version="0.1.0" />
  </ItemGroup>
```

这个库的入口是 WICImagingFactory 接口，这是一个 COM 定义的接口，因此可以用 new 关键字创建，如下面代码

```csharp
     WICImagingFactory factory = new WICImagingFactory();
```

接口是可以使用 new 的，只要标记了这是 COM 接口就可以

这个库的封装都在 WICImagingFactory 的方法，各个方法的使用方法还请参阅[官方 WIC 文档](https://docs.microsoft.com/en-us/windows/win32/wic/-wic-lh)

以下是一些例子

如获取本机安装的图片解码器，可以用来判断本机是否有 HEIF 解码器

```csharp
        static void Main(string[] args)
        {
            WICImagingFactory factory = new WICImagingFactory();

            foreach (var wicBitmapEncoderInfo in EnumEncoders(factory))
            {
                Console.WriteLine(wicBitmapEncoderInfo.GetFriendlyName());
            }

            // BMP Encoder
            // GIF Encoder
            // JPEG Encoder
            // PNG Encoder
            // TIFF Encoder
            // WMPhoto Encoder
            // DDS Encoder
            // Microsoft HEIF Encoder
        }

        static IEnumerable<IWICBitmapEncoderInfo> EnumEncoders(IWICImagingFactory wic)
        {
            return wic.CreateComponentEnumerator(WICComponentType.WICEncoder, WICComponentEnumerateOptions.WICComponentEnumerateDefault)
                .AsEnumerable()
                .OfType<IWICBitmapEncoderInfo>();
        }
```

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/0129f0e7/HairleakaibaniWawfeahewur ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/0129f0e7/HairleakaibaniWawfeahewur ) 欢迎小伙伴访问

如使用像素的方式快速创建图片

```csharp
            WICImagingFactory factory = new WICImagingFactory();

            var encoderInfo = EnumEncoders(factory)
                .FirstOrDefault(temp => temp.GetFriendlyName() == "PNG Encoder");

            const int width = 256;
            const int height = 256;
            const int bytesPerPixel = 3;// BGR 格式

            var random = new Random();

            if (encoderInfo != null)
            {
                var encoder = factory.CreateEncoder(encoderInfo.GetContainerFormat());

                using (var stream = File.Create("1.png"))
                {
                    encoder.Initialize(stream.AsCOMStream(),WICBitmapEncoderCacheOption.WICBitmapEncoderNoCache);

                    var frame = encoder.CreateNewFrame();
                    frame.Initialize(null);

                    var format = WICPixelFormat.WICPixelFormat24bppBGR;
                    frame.SetPixelFormat(ref format);

                    frame.SetResolution(new Resolution(96, 96));
                    frame.SetSize(width, height);

                    var image = new byte[width * height * bytesPerPixel];

                    for (int i = 0; i < height; i++)
                    {
                        for (int j = 0; j < width; j++)
                        {
                            image[(i * width + j) * bytesPerPixel + 0] = (byte)random.Next(255);
                            image[(i * width + j) * bytesPerPixel + 1] = (byte)random.Next(255);
                            image[(i * width + j) * bytesPerPixel + 2] = (byte)random.Next(255);
                        }
                    }

                    IWICBitmapFrameEncodeExtensions.WritePixels(frame, height, width * bytesPerPixel, image);

                    frame.Commit();
                    encoder.Commit();
                }
            }
```

这是我创建的图片

<!-- ![](image/dotnet 在 Windows 系统上使用 stakx 的 WIC 库/dotnet 在 Windows 系统上使用 stakx 的 WIC 库0.png) -->

![](https://i.loli.net/2021/03/26/Ji4A2eNrz5PVL8g.jpg)

以上代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/dccc43c6/HairleakaibaniWawfeahewur ) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/dccc43c6/HairleakaibaniWawfeahewur ) 欢迎小伙伴访问





<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://licensebuttons.net/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。