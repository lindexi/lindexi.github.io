# dotnet win32 使用 WIC 获取系统编解码器

在 Windows 系统上，有一个很重要的概念是 Windows Imaging Component 也就是 WIC 层，这是专门用来处理多媒体相关的系统组件，特别是用来处理图片相关，包括编码和解码和处理图片。开发者可以扩展 WIC 层的编解码器，从而让系统可以支持更多格式的多媒体文件。本文将告诉大家如何获取当前系统上在 WIC 层安装的图片编解码器，从而了解当前系统支持哪些格式的图片

<!--more-->
<!-- CreateTime:2023/5/12 18:46:48 -->

<!-- 发布 -->
<!-- 博客 -->

为了方便在 dotnet 调用到 WIC 层，本文将安装 stakx 库。这个库是对 WIC 的底层基础封装，代码也不多，大家如果不想安装库，也可以自行去抄代码。详细请看 [dotnet 在 Windows 系统上使用 stakx 的 WIC 库](https://blog.lindexi.com/post/dotnet-%E5%9C%A8-Windows-%E7%B3%BB%E7%BB%9F%E4%B8%8A%E4%BD%BF%E7%94%A8-stakx-%E7%9A%84-WIC-%E5%BA%93.html )

新建 dotnet 6 控制台项目，编辑 csproj 项目文件，替换为如下代码即可完成 stakx 库的安装

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="stakx.WIC" Version="0.1.0" />
  </ItemGroup>
</Project>
```

按照 [dotnet 在 Windows 系统上使用 stakx 的 WIC 库](https://blog.lindexi.com/post/dotnet-%E5%9C%A8-Windows-%E7%B3%BB%E7%BB%9F%E4%B8%8A%E4%BD%BF%E7%94%A8-stakx-%E7%9A%84-WIC-%E5%BA%93.html ) 博客提供的方法，先创建 WICImagingFactory 对象，代码如下

```csharp
        WICImagingFactory factory = new WICImagingFactory();
```

接着通过 CreateComponentEnumerator 获取当前系统的编码器或解码器，如以下代码传入 `WICComponentType.WICDecoder` 即可获取解码器

```csharp
   var componentEnumerator = factory.CreateComponentEnumerator(WICComponentType.WICDecoder, WICComponentEnumerateOptions.WICComponentEnumerateDefault);
```

此 `componentEnumerator` 是一个枚举，可以通过 AsEnumerable 方法转换为 .NET 的枚举，代码如下

```csharp
        foreach (var o in componentEnumerator.AsEnumerable())
        {
            IWICBitmapCodecInfo codecInfo = o as IWICBitmapCodecInfo;
            if (codecInfo != null)
            {
                Console.WriteLine("----------");
                Console.WriteLine($"CLSID: {codecInfo.GetCLSID()}");
                Console.WriteLine(codecInfo.GetFriendlyName());
                Console.WriteLine($"FileExtensions: {string.Join(";", codecInfo.GetFileExtensions())}");
                Console.WriteLine($"MimeType: {string.Join(";", codecInfo.GetMimeTypes())}");
                Console.WriteLine($"Version: {codecInfo.GetVersion()}");
                Console.WriteLine("----------");
            }
        }
```

这就是使用 C# 代码枚举本机所安装的图片解码器的方法

运行代码，输出大概如下

```
----------
CLSID: 6b462062-7cbf-400d-9fdb-813dd10f2778
BMP Decoder
FileExtensions: .bmp;.dib;.rle
MimeType: image/bmp
Version: 1.0.0.0
----------
----------
CLSID: 381dda3c-9ce9-4834-a23e-1f98f8fc52be
GIF Decoder
FileExtensions: .gif
MimeType: image/gif
Version: 1.0.0.0
----------
----------
CLSID: c61bfcdf-2e0f-4aad-a8d7-e06bafebcdfe
ICO Decoder
FileExtensions: .ico;.icon
MimeType: image/ico;image/x-icon
Version: 1.0.0.0
----------
----------
CLSID: 22696b76-881b-48d7-88f0-dc6111ff9f0b
CUR Decoder
FileExtensions: .cur
MimeType:
Version: 1.0.0.0
----------
----------
CLSID: 9456a480-e88b-43ea-9e73-0b2d9b71b1ca
JPEG Decoder
FileExtensions: .jpeg;.jpe;.jpg;.jfif;.exif
MimeType: image/jpeg;image/jpe;image/jpg
Version: 1.0.0.0
----------
----------
CLSID: 389ea17b-5078-4cde-b6ef-25c15175c751
PNG Decoder
FileExtensions: .png
MimeType: image/png
Version: 1.0.0.0
----------
----------
CLSID: b54e85d9-fe23-499f-8b88-6acea713752b
TIFF Decoder
FileExtensions: .tiff;.tif
MimeType: image/tiff;image/tif
Version: 1.0.0.0
----------
----------
CLSID: 981d9411-909e-42a7-8f5d-a747ff052edb
DNG Decoder
FileExtensions: .dng
MimeType: image/DNG
Version: 1.0.0.0
----------
----------
CLSID: a26cec36-234c-4950-ae16-e34aace71d0d
WMPhoto Decoder
FileExtensions: .wdp;.jxr
MimeType: image/vnd.ms-photo
Version: 1.0.0.0
----------
----------
CLSID: 9053699f-a341-429d-9e90-ee437cf80c73
DDS Decoder
FileExtensions: .dds
MimeType: image/vnd.ms-dds
Version: 1.0.0.0
----------
----------
CLSID: e9a4a80a-44fe-4de4-8971-7150b10a5199
Microsoft HEIF Decoder
FileExtensions: .heic;.heif;.hif;.avci;.heics;.heifs;.avcs;.avif;.avifs
MimeType: image/heic;image/heif;image/avci;image/heic-sequence;image/heif-sequence;image/avcs;image/avif;image/avif-sequence
Version: 1.0.0.0
----------
----------
CLSID: 7693e886-51c9-4070-8419-9f70738ec8fa
Microsoft Webp Decoder
FileExtensions: .webp
MimeType: image/webp
Version: 1.0.0.0
----------
----------
CLSID: 41945702-8302-44a6-9445-ac98e8afa086
Microsoft Raw Image Decoder
FileExtensions: .3FR;.ARI;.ARW;.BAY;.CAP;.CR2;.CR3;.CRW;.DCS;.DCR;.DRF;.EIP;.ERF;.FFF;.IIQ;.K25;.KDC;.MEF;.MOS;.MRW;.NEF;.NRW;.ORF;.ORI;.PEF;.PTX;.PXN;.RAF;.RAW;.RW2;.RWL;.SR2;.SRF;.SRW;.X3F;.DNG
MimeType: image/3FR;image/ARI;image/ARW;image/BAY;image/CAP;image/CR2;image/CR3;image/CRW;image/DCS;image/DCR;image/DRF;image/EIP;image/ERF;image/FFF;image/IIQ;image/K25;image/KDC;image/MEF;image/MOS;image/MRW;image/NEF;image/NRW;image/ORF;image/ORI;image/PEF;image/PTX;image/PXN;image/RAF;image/RAW;image/RW2;image/RWL;image/SR2;image/SRF;image/SRW;image/X3F;image/DNG
Version: 10.0.22621.1
----------
----------
CLSID: fc6ceece-aef5-4a23-96ec-5984ffb486d9
Microsoft JPEG-XL Decoder
FileExtensions: .JXL
MimeType: image/JXL
Version: 10.0.22621.1
----------
```

同理的获取解码器可以传入 WICComponentType.WICEncoder 代码，修改之后的代码如下

```csharp
        var componentEnumerator = factory.CreateComponentEnumerator(WICComponentType.WICEncoder, WICComponentEnumerateOptions.WICComponentEnumerateDefault);
```

运行代码可以看到输出大概如下

```
----------
CLSID: 69be8bb4-d66d-47c8-865a-ed1589433782
BMP Encoder
FileExtensions: .bmp;.dib;.rle
MimeType: image/bmp
Version: 1.0.0.0
----------
----------
CLSID: 114f5598-0b22-40a0-86a1-c83ea495adbd
GIF Encoder
FileExtensions: .gif
MimeType: image/gif
Version: 1.0.0.0
----------
----------
CLSID: 1a34f5c1-4a5a-46dc-b644-1f4567e7a676
JPEG Encoder
FileExtensions: .jpeg;.jpe;.jpg;.jfif;.exif
MimeType: image/jpeg;image/jpe;image/jpg
Version: 1.0.0.0
----------
----------
CLSID: 27949969-876a-41d7-9447-568f6a35a4dc
PNG Encoder
FileExtensions: .png
MimeType: image/png
Version: 1.0.0.0
----------
----------
CLSID: 0131be10-2001-4c5f-a9b0-cc88fab64ce8
TIFF Encoder
FileExtensions: .tiff;.tif
MimeType: image/tiff;image/tif
Version: 1.0.0.0
----------
----------
CLSID: ac4ce3cb-e1c1-44cd-8215-5a1665509ec2
WMPhoto Encoder
FileExtensions: .wdp;.jxr
MimeType: image/vnd.ms-photo
Version: 1.0.0.0
----------
----------
CLSID: a61dde94-66ce-4ac1-881b-71680588895e
DDS Encoder
FileExtensions: .dds
MimeType: image/vnd.ms-dds
Version: 1.0.0.0
----------
----------
CLSID: 0dbecec1-9eb3-4860-9c6f-ddbe86634575
Microsoft HEIF Encoder
FileExtensions: .heic;.heif;.hif
MimeType: image/heic;image/heif
Version: 1.0.0.0
----------
----------
CLSID: 0e4ecd3b-1ba6-4636-8198-56c73040964a
Microsoft JPEG-XL Encoder
FileExtensions: .JXL
MimeType: image/JXL
Version: 10.0.22621.1
----------
```

如果同时需要获取编码器和解码器，可以传入 `WICComponentType.WICEncoder | WICComponentType.WICDecoder` 如以下代码

```csharp
        var componentEnumerator = factory.CreateComponentEnumerator(WICComponentType.WICEncoder | WICComponentType.WICDecoder, WICComponentEnumerateOptions.WICComponentEnumerateDefault);
```

本文的代码放在[github](https://github.com/lindexi/lindexi_gd/tree/4cb2d3c557122601447d5b4a1e58587650535bb7/JegejarleRajurnayhajaidee) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/4cb2d3c557122601447d5b4a1e58587650535bb7/JegejarleRajurnayhajaidee) 欢迎访问

可以通过如下方式获取本文的源代码，先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 4cb2d3c557122601447d5b4a1e58587650535bb7
```

以上使用的是 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 4cb2d3c557122601447d5b4a1e58587650535bb7
```

获取代码之后，进入 JegejarleRajurnayhajaidee 文件夹