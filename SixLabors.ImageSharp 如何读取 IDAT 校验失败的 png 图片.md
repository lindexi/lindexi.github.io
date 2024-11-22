# SixLabors.ImageSharp 如何读取 IDAT 校验失败的 png 图片

本文记录如何在 SixLabors.ImageSharp 库里面读取 IDAT 校验失败的 png 图片

<!--more-->
<!-- CreateTime:2024/11/21 07:09:35 -->

<!-- 发布 -->
<!-- 博客 -->

以下是我的读取代码

```csharp
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.Formats.Png;
using SixLabors.ImageSharp.Processing;

var file = @"C:\lindexi\Image\Image.png";
using var image = Image.Load(file);
```

运行将会看到如下异常

SixLabors.ImageSharp.InvalidImageContentException:“CRC Error. PNG IDAT chunk is corrupt!”

这是因为我传入的是一张损坏的图片

解决方法就是通过 PngDecoder 解码器，传入 PngCrcChunkHandling.IgnoreAll 枚举，让其忽略即可。修改之后代码如下

```csharp
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.Formats;
using SixLabors.ImageSharp.Formats.Png;
using SixLabors.ImageSharp.Processing;

var file = @"C:\lindexi\Image\Image.png";

using var fileStream = File.OpenRead(file);

var decode = PngDecoder.Instance.Decode(new PngDecoderOptions()
{
    PngCrcChunkHandling = PngCrcChunkHandling.IgnoreAll,
}, fileStream);
```

解码之后即可获取图片的信息，如获取格式和尺寸

```csharp
var decodeSize = decode.Size;
var pixelType = decode.PixelType;
```

SixLabors.ImageSharp 库开源地址： <https://github.com/SixLabors/ImageSharp>

以下是安装了 SixLabors.ImageSharp 库的 csproj 项目文件的代码

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net9.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="SixLabors.ImageSharp" Version="3.1.6" />
  </ItemGroup>

</Project>
```

本文代码放在 [github](https://github.com/lindexi/lindexi_gd/tree/63c9ca8f563ad8e5d4b26bc1a740b10af243abce/Workbench/HegerkalailayiwuCulalajer) 和 [gitee](https://gitee.com/lindexi/lindexi_gd/tree/63c9ca8f563ad8e5d4b26bc1a740b10af243abce/Workbench/HegerkalailayiwuCulalajer) 上，可以使用如下命令行拉取代码。我整个代码仓库比较庞大，使用以下命令行可以进行部分拉取，拉取速度比较快

先创建一个空文件夹，接着使用命令行 cd 命令进入此空文件夹，在命令行里面输入以下代码，即可获取到本文的代码

```
git init
git remote add origin https://gitee.com/lindexi/lindexi_gd.git
git pull origin 63c9ca8f563ad8e5d4b26bc1a740b10af243abce
```

以上使用的是国内的 gitee 的源，如果 gitee 不能访问，请替换为 github 的源。请在命令行继续输入以下代码，将 gitee 源换成 github 源进行拉取代码。如果依然拉取不到代码，可以发邮件向我要代码

```
git remote remove origin
git remote add origin https://github.com/lindexi/lindexi_gd.git
git pull origin 63c9ca8f563ad8e5d4b26bc1a740b10af243abce
```

获取代码之后，进入 Workbench/HegerkalailayiwuCulalajer 文件夹，即可获取到源代码

更多技术博客，请参阅 [博客导航](https://blog.lindexi.com/post/%E5%8D%9A%E5%AE%A2%E5%AF%BC%E8%88%AA.html )